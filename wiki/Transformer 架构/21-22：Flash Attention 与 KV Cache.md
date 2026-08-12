---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [21, 22]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-PyTorch-Module状态与Buffer.md"
---

# 第 21～22 章：Flash Attention 与 KV Cache

> [!abstract] Flash Attention 和 KV Cache 都能让 Attention 更快，但解决的是两个不同问题：Flash Attention 优化“一次 Attention 内部怎样搬运和保存中间结果”，KV Cache 优化“连续生成多个 Token 时哪些历史计算不必重做”。前者减少临时显存和 HBM 读写，后者用持续占用的缓存换取 Decode 计算量下降。

> [!info] 阅读粒度
> 本页用同一张性能地图比较两种容易混淆的优化，因此标为“深入机制”；快速复述时，阅读摘要、对比表、“Prefill 与 Decode”和“别误解”即可。

## 先区分两个优化方向

```text
一次 Attention 调用内部
Q、K、V → scores → Softmax → output
问题：是否必须把完整 T×T 中间矩阵写入显存？
方案：Flash Attention

多次自回归 Decode 之间
Prompt → Token₁ → Token₂ → Token₃
问题：历史 Token 的 K、V 是否每一步都要重算？
方案：KV Cache
```

| 对比 | Flash Attention | KV Cache |
| --- | --- | --- |
| 优化范围 | 单次 Attention 内部 | 多个生成步骤之间 |
| 避免的浪费 | 大型中间矩阵的显存读写 | 历史 K、V 的重复计算 |
| 主要收益 | 降低临时显存、提高内核效率 | 降低 Decode 计算和单 Token 延迟 |
| 主要代价 | 依赖适配的内核、形状与硬件 | 每个请求持续占用 KV 显存 |
| 是否改变模型语义 | 数学上仍是精确 Attention | 复用同一组历史 K、V；与完整前缀重算应在数值容差内一致 |

两者可以同时使用，并不是二选一。

## Flash Attention：不物化完整 Attention 矩阵

标准 Attention 的概念流程是：

```text
Q @ Kᵀ
→ scores [B, H, T, T]
→ Mask + Softmax
→ probabilities [B, H, T, T]
→ probabilities @ V
→ output
```

朴素实现可能把 `scores` 和 Softmax 结果写入 HBM，再读回来执行下一步。序列变长时，`T × T` 中间矩阵不仅占用显存，还产生大量显存读写。

Flash Attention 的核心是 **Tiling + Online Softmax**：

1. 把 Q、K、V 切成能放入片上高速存储的小块；
2. 每次只计算一个 Q 块与一个 K/V 块；
3. 用 Online Softmax 维护每行的运行最大值、指数和与加权输出；
4. 在最大值更新时重新缩放之前的累积结果；
5. 最终直接写出 Attention 输出，不把完整 `T × T` 概率矩阵长期存入 HBM。

可以把每行需要维护的状态简化为：

```text
m：截至当前块的最大 score
l：经过数值稳定缩放后的 exp 总和
o：经过相同缩放的加权 V 累积结果
```

因为 Softmax 可以分块累积，所以改变计算顺序后仍能得到数学上等价的精确 Attention；但浮点运算顺序不同，不能要求与朴素实现逐 bit 完全一致。

### Flash Attention 改变了什么

- 减少 HBM 与片上存储之间的数据搬运；
- 避免保存完整 Attention 概率矩阵，降低中间激活显存；
- 反向传播时可重新计算部分块内中间结果，用额外计算换取更少存储；
- 长序列、训练和 Prefill 阶段通常更容易体现收益。

### Flash Attention 没改变什么

- Dense Attention 的主要乘加运算仍随序列长度近似二次增长；
- 模型参数量没有减少；
- 它不是稀疏 Attention，也不是低秩近似；
- 调用高层 API 不保证一定命中 Flash 内核，实际后端取决于硬件、dtype、形状、Mask、Dropout 和框架支持。

工程上优先使用框架提供的融合 Attention API，例如 PyTorch 的 `scaled_dot_product_attention`，让运行时选择可用后端；若性能重要，应通过 profiler 或后端日志确认实际选择，而不是只根据函数名判断。

## KV Cache：只计算新 Token 的 K、V

在第 `t` 个 Decode 步骤中，新 Token 的 Query 需要关注从位置 `1` 到 `t` 的所有 Key 和 Value：

```text
Q_t @ [K_1, K_2, ..., K_t]ᵀ
→ Attention 权重
→ 加权 [V_1, V_2, ..., V_t]
```

历史 Token 的 K、V 在参数不变时不会改变，因此可以按层缓存：

```text
Prefill：计算 Prompt 的全部 K、V，并用 Prompt 最后位置的 logits 采样 Token₁
Decode 1：输入 Token₁，只算它的 Q₁、K₁、V₁，追加 K₁、V₁，再采样 Token₂
Decode 2：输入 Token₂，只算它的 Q₂、K₂、V₂，追加 K₂、V₂，再采样 Token₃
……
```

不需要缓存历史 Q，因为过去位置的输出已经算完；当前步骤只需要新 Query 去读取全部历史 K、V。

KV Cache 省掉的是历史 Token 的 K/V 投影和旧前缀的重复前向计算，但它没有让新 Query “不用看历史”：每个新 Token 仍需与缓存中的全部 Key 做匹配。因此：

- 单步 Decode 的 Attention 工作量仍随已缓存长度线性增加；
- 整段生成的 Attention 工作量仍会随生成长度近似二次增长；
- KV Cache 不是把长序列生成变成常数时间。

## KV Cache 的显存公式

对于常见实现，KV Cache 的近似容量为：

```text
bytes
≈ 2
× batch_size
× cached_tokens
× num_layers
× num_kv_heads
× head_dim
× bytes_per_element
```

其中：

- `2` 表示 K 和 V；
- `cached_tokens` 包含 Prompt 和已生成 Token；
- MHA 通常有 `num_kv_heads = num_query_heads`；
- MQA / GQA 会减少 `num_kv_heads`，因此能直接减少 KV Cache；
- 量化 Cache 会减少 `bytes_per_element`，但可能增加量化开销和误差。

这个公式解释了三个系统现象：

1. 长上下文会线性增加单请求缓存；
2. 每个并发请求通常需要独立缓存，因此 KV 显存会限制并发；
3. 降低 KV Head 数量、精度或保留长度，都是直接的内存优化方向。

## Prefill 与 Decode 的瓶颈不同

```text
Prefill
→ 输入多个 Prompt Token
→ 并行构建各层 KV Cache
→ 通常更偏计算密集
→ 主要影响首 Token 延迟

Decode
→ 每轮只有少量新 Token
→ 读取模型权重和不断增长的 KV Cache
→ 通常更偏内存带宽受限
→ 主要影响后续每 Token 延迟
```

因此不能只说“模型每秒生成多少 Token”，还应至少区分：

- **TTFT（Time to First Token）**：从请求进入到第一个输出 Token；
- **TPOT（Time per Output Token）**：首 Token 之后，每个新 Token 的平均时间；
- **Throughput**：单位时间内整个服务处理的 Token 或请求数量；
- **Peak / Reserved Memory**：权重、临时激活和 KV Cache 各自占多少显存。

Flash Attention 通常对长序列 Attention、训练和 Prefill 的临时内存及执行效率更关键；KV Cache 则是标准自回归 Decode 的基础优化。Decode 阶段是否还能从 Flash 类内核明显获益，要看 Query 长度、Batch、缓存布局和具体内核，不能直接套用 Prefill 基准。

## 两者的内存方向恰好相反

这是最容易忽略的关系：

```text
Flash Attention
→ 减少单次 Attention 的临时中间内存

KV Cache
→ 增加跨 Decode 步骤持久保存的请求状态
```

所以开启 Flash Attention 不代表长对话就不会 OOM；当并发数、上下文长度或 KV Head 数增长时，KV Cache 仍可能成为主要显存占用。

反过来，开启 KV Cache 也不代表单次 Prefill 的 Attention 中间内存已经优化。生产系统通常需要同时管理：

- 模型权重；
- Prefill / Attention 临时工作区；
- 每个请求的 KV Cache；
- Batch 调度和内存碎片。

## 做项目时记住

1. **先定位瓶颈再选优化。** 长 Prompt OOM 或 Prefill 慢，优先检查 Attention 内核和临时激活；Decode 慢或并发低，优先检查 KV Cache 大小、布局和内存带宽。
2. **缓存必须和位置及 Mask 同步。** 追加新 KV 时，位置编号、Attention Mask 和有效长度必须一致；如果历史 Token 被修改，受影响位置之后的 Cache 不能直接复用。
3. **手算 KV Cache 预算。** 在部署前用 `batch × tokens × layers × kv_heads × head_dim × dtype` 估算，并为框架工作区和碎片保留余量。
4. **验证实际内核。** 融合 API 可能回退到其他实现；以 profiler、峰值显存和端到端延迟为准。
5. **把单请求延迟与服务吞吐分开。** Continuous Batching、Paged KV Cache 等主要改善资源利用率和并发，不等于每个请求的每一步计算都更少。

## 最值得做的三个验证

### 1. Flash 与基线数值对照

在相同 Q、K、V、Mask 和 Dropout 配置下，比较融合实现与数学基线：

```text
输出误差在 dtype 对应容差内
梯度误差在可接受容差内
峰值显存下降
端到端时间在目标形状上改善
```

不能只验证“小张量更快”；实际收益高度依赖序列长度、Head 维度、Batch 和硬件。

### 2. Cached 与 Uncached logits 对照

固定模型和输入，分别使用完整前缀重算与 KV Cache 增量 Decode。每一步最后位置的 logits 应在数值容差内一致。

若不一致，优先检查：

- Cache 拼接维度和层对应关系；
- `cache_position` / Position ID；
- 因果 Mask 与 Padding Mask；
- Batch 中不同序列的有效长度；
- Beam Search 或请求重排后 Cache 是否同步重排。

### 3. 分阶段性能测量

分别测量不同 Prompt 长度、生成长度和并发下的：

```text
TTFT
TPOT
吞吐量
峰值显存
KV Cache 实际占用
```

只有这样才能判断优化改善的是 Prefill、Decode、单请求延迟还是总体吞吐。

## 别误解

- **“Flash Attention 把 Dense Attention 的计算复杂度从 O(T²) 降成 O(T)。”** 它主要降低中间存储和内存 IO，Dense Attention 的主要算术工作仍近似二次增长。
- **“Flash Attention 是近似算法，所以会牺牲模型质量。”** 它重新排序精确 Attention 的计算；浮点舍入可能不同，但不是稀疏近似。
- **“有 KV Cache 后，每个新 Token 都是 O(1)。”** 新 Query 仍要读取并匹配全部历史 K，Cache 长度越长，单步工作越多。
- **“KV Cache 会保存模型已经生成的所有中间激活。”** 通常只保存各层历史 K、V，不保存完整 Attention 矩阵或所有层激活。
- **“训练也应该开启 KV Cache。”** Teacher Forcing 可并行处理完整序列，训练还需要反向传播激活；自回归 Cache 主要服务推理。
- **“开启两项优化后就不会显存不足。”** Flash 减少临时内存，KV Cache 却会随并发和上下文增长。

## 复习

1. Flash Attention 和 KV Cache 分别消除了哪一种重复或搬运？
2. Flash Attention 为什么能分块计算 Softmax，而不保存完整 `T × T` 概率矩阵？
3. 为什么 KV Cache 只缓存 K、V，不需要缓存历史 Q？
4. 为什么 KV Cache 能减少重复计算，却不能让单步 Decode 变成 O(1)？
5. `num_kv_heads`、上下文长度和并发怎样共同决定 Cache 显存？
6. 为什么 Flash Attention 与 KV Cache 可以同时开启，却可能分别改善不同指标？

## 来源与下一步

- **前置概念：** [[09-12：Attention|Attention]]、[[16-17：训练、推理与参数更新|训练、推理与参数更新]]、[[18-20：最小 Transformer 实现|最小 Transformer 实现]]。
- **来源事实：** 第 21 章解释 GPU 内存层级、Tiling、Online Softmax、Flash Attention 的 IO 优化及使用边界；第 22 章解释自回归重复计算、KV Cache、Prefill / Decode、缓存显存公式和后续压缩方向。
- **机制推导：** Flash Attention 优化单次 Attention 的临时 IO，KV Cache 优化多次 Decode 之间的重复计算；两者共同决定长上下文推理中的临时内存、持久状态、延迟与吞吐。
- **工程补充：** 性能验收应拆分 TTFT、TPOT、吞吐和显存，并通过数值对照确认融合内核与 Cache 没有破坏 Attention 语义。
- **下一主题：** [[23：MHA、MQA 与 GQA|第 23 章：MHA、MQA 与 GQA]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-PyTorch-Module状态与Buffer|PyTorch Module 状态、Autograd 与 Buffer]]。
