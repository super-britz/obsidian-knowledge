---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [30]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-MoE论文与技术报告.md"
---

# 第 30 章：Mixture of Experts——用条件计算扩展参数容量

> [!abstract] 稀疏 MoE 通常把 Transformer Block 的单个 FFN 换成多个独立 FFN，由 Router 为每个 Token 选择少数专家并组合输出。它把“模型可容纳多少参数”与“每个 Token 执行多少专家计算”部分解耦，却不会同比减少权重存储、通信或端到端延迟。

> [!info] 阅读粒度
> 本页把路由机制、训练约束和分布式执行放在同一条 Token 调度链中，因此标为“深入机制”；快速阅读时先看 MoE 层流程、四本资源账和部署决策。

## MoE 改的是 FFN 路径，不是整个 Transformer

标准 Decoder Block 的 Attention 和 FFN 都对每个 Token 执行。常见 Sparse MoE 保留 Attention、Norm 和 Residual，只把部分或全部 Dense FFN 替换为多个 FFN 专家：

```text
Token hidden state x
→ Attention 等共享层
→ Router 为 x 计算专家分数
→ 选择 Top-k 专家并把 Token 分发过去
→ 专家各自执行 FFN
→ 按路由权重合并专家输出
→ 写回 residual stream
```

因此条件计算只发生在 MoE 子层。总参数会随专家数显著增长，但每个 Token 仍需执行共享层、Router 和 k 个专家。总参数、激活参数、FLOPs、显存与速度是五个不同指标。

## Router、Top-k 与专家输出

设 Router 为 Token 表示 `x` 产生 N 个分数 `s_i(x)`，再经 Softmax、Sigmoid 或具体架构定义的归一化得到门控值。选中集合记为 `TopK(x)`：

```text
y = Σ g_i(x) · Expert_i(x),  i ∈ TopK(x)
```

不同 Token、不同层可以选择不同专家；同一个 Token 在后续层也会重新路由。Top-1 只执行一个专家，计算和通信量较小；Top-2 或更大的 k 允许多个专家共同贡献，但增加专家计算和分发量。Switch Transformer 使用 Top-1，GShard 与 Mixtral 使用过 Top-2，足以说明 Top-k 是架构选择，不是固定升级顺序。

Router 与专家通过最终任务 Loss 联合训练。未被某个 Token 选中的专家通常不会收到该 Token 的专家路径梯度；Top-k 的离散选择、门控权重和辅助目标共同决定 Router 学到的分区。专家可能出现统计上的专门化，但它们更像隐空间中的条件子网络，不能默认是可命名的“数学专家”或“中文专家”。

## 容量与负载均衡是一项调度契约

若大量 Token 同时路由到少数专家，会出现三个问题：热门专家形成延迟尾部，冷门专家训练不足，跨设备缓冲区和通信容量溢出。常见系统用容量因子为每个专家预留近似上限：

```text
expert_capacity ≈ capacity_factor × tokens × k / num_experts
```

具体实现可能对超额 Token 丢弃专家分支、经残差旁路、送往次选专家或动态扩容，语义必须写进实现契约。Capacity Factor 越大，丢弃风险越低，但 Padding、内存和计算浪费可能上升。

训练时常加入负载均衡辅助损失，让实际选中比例与 Router 概率质量不过度集中；Router Z-loss 可抑制 Logit 过大。辅助损失过强又可能为了“均匀”牺牲有用路由。DeepSeek-V3 则报告了无辅助损失的偏置调节策略，同时保留序列级均衡约束，说明负载均衡不是只有一个公式。

应同时监控每层的专家 Token 数、路由概率、溢出 / 丢弃率、熵、Top-k 重合度和每设备执行时间。只看平均负载会隐藏少数 Batch 的热点和 P99 延迟。

## 四本资源账不能混算

| 资源 | MoE 的影响 | 常见误判 |
| --- | --- | --- |
| 参数容量 | 多个专家增加可学习权重 | 总参数大就必然能力强 |
| 每 Token 计算 | 只执行 k 个路由专家，但共享层仍执行 | 激活 13B 等于 Dense 13B FLOPs |
| 权重内存 | 推理服务总体仍需保存或加载全部专家，可跨设备分片 | 只需为激活专家准备显存 |
| 通信与调度 | Expert Parallel 常需 All-to-All 分发 Token 和收回输出 | FLOPs 降低就必然同比提速 |

“激活参数”本身也依赖统计口径：是否计算 Embedding、Attention、共享专家和 LM Head，厂商与论文可能不同。比较模型时应优先使用同一后端实测的 Prefill / Decode 延迟、吞吐、峰值显存、网络流量和每 Token 成本。

## 专家并行为什么会成为系统瓶颈

当专家分布在多张设备上时，每张设备先持有一批 Token，Router 再决定它们实际需要前往哪个专家：

```text
各设备上的 Token
→ All-to-All：按专家重新分桶并跨设备发送
→ 本地专家计算
→ All-to-All：把结果送回原 Token 所在设备
→ 恢复原序列顺序
```

Dense Tensor Parallel 主要切分同一矩阵；Expert Parallel 则动态搬运 Token。性能取决于网络拓扑、Token 数、专家放置、分桶粒度、负载尾部及计算—通信重叠。小 Batch 或逐 Token Decode 时，每个专家得到的 Token 很少，矩阵乘法利用率和通信摊销可能变差；大 Batch 提高专家批量，却可能增加排队延迟。

部署需要联合设计 Data、Tensor、Pipeline、Expert Parallel，尽量把高频通信限制在高速互联域，并通过专家复制、路由约束或连续批处理平衡热点。理论稀疏度只是性能上限，不是部署结果。

## Mixtral 与 DeepSeek 说明了哪些设计空间

Mixtral 8x7B 在每层放置 8 个 FFN 专家，每个 Token 选 2 个；技术报告给出约 47B 总参数、约 13B 激活参数和 32K 训练上下文，并在其评测集合中与 Llama 2 70B 等模型比较。这证明了该具体模型的质量—计算取舍，不证明所有 13B 激活 MoE 都等价或优于 70B Dense。

DeepSeek 系列展示了另一组组合：DeepSeekMoE 使用更细粒度的路由专家和共享专家，让通用路径始终参与、路由路径承担差异化计算；MLA 压缩 KV Cache，解决的是 Attention 推理状态而非 MoE 路由本身。DeepSeek-V3 报告 671B 总参数、每 Token 激活 37B，并使用辅助损失之外的均衡策略。

这些组件可以组合，但优化对象不同：MoE 扩展条件参数容量，MLA 压缩 KV 状态，量化降低权重表示成本。不能把整套模型的收益全部归因于 MoE，也不能把报告的 2.788M H800 GPU 小时直接转换为完整研发美元成本或与未披露模型对比。

## 怎样判断该用 Dense 还是 MoE

```text
需要在相近每 Token 计算下继续扩展模型容量？
├─ 否：Dense 更简单，先优化模型、量化和推理内核
└─ 是：是否有高速互联、专家并行和路由可观测性？
   ├─ 否：MoE 的通信与运维成本可能抵消计算收益
   └─ 是：用目标 Batch / 长度做 Dense 与 MoE 端到端对照
```

MoE 更适合大规模、多任务训练，以及能够汇聚大 Batch、分摊专家权重和高速通信的服务集群。单机显存紧张、边缘部署、小 Batch 极低延迟或运维能力有限时，Dense + 量化往往更直接。MoE 和 [[26-27：LoRA、QLoRA 与量化|量化]] 可以组合，但所有专家仍需保存，量化误差、专家调度和内核兼容必须重新验证。

## 最值得做的验证

1. **数学正确性：** 小模型对比稠密计算与 Dispatch / Gather 实现，验证门控归一化、Token 顺序、Padding、梯度和溢出路径。
2. **路由健康度：** 按层、专家、数据域和时间统计负载、熵、溢出率、死专家及热点专家，不能只看全局均值。
3. **质量消融：** 在相同数据和近似训练 FLOPs 下比较 Dense、不同专家数、Top-k、共享专家与均衡系数，检查专家数带来的收益来源。
4. **分布式性能：** 分开测 Router、Dispatch、All-to-All、Expert GEMM、Combine 时间，以及网络字节量、P50 / P99 和设备空闲率。
5. **Artifact 契约：** 记录专家数、Top-k、容量策略、共享专家、Router dtype、并行拓扑、量化配置和后端版本，做保存—加载及单卡 / 多卡 Logits 对照。

## 别误解

- **“12B 激活参数就是一台 12B Dense 模型的成本。”** 两者共享层、专家形状、内存访问、通信和算子效率不同。
- **“每次只用两个专家，所以只需加载两个专家。”** 不同 Token 会选择不同专家，服务总体仍需让全部专家可访问。
- **“专家会自然变成可解释的学科专家。”** 路由专门化可能存在，但通常是分布式、层相关且难以稳定命名的模式。
- **“负载越均匀，模型越好。”** 均衡有利于硬件利用率，过强约束可能阻止有用的非均匀分工。
- **“FLOPs 少就一定延迟低。”** All-to-All、动态分桶、小批量 GEMM 和热点专家都可能成为瓶颈。
- **“某模型的性能和成本都来自 MoE。”** 数据、训练预算、Attention、精度、内核与评测同样影响结果。

## 复习

1. Sparse MoE 把 Transformer Block 的哪一部分换成了条件计算？
2. 为什么总参数、激活参数、FLOPs、显存和端到端速度不能互相替代？
3. 容量因子和负载均衡损失分别解决什么，又引入什么代价？
4. Expert Parallel 为什么需要两次动态 Token 通信？
5. 哪些工作负载更可能从 MoE 获益，哪些更适合 Dense？

## 来源与下一步

- **前置概念：** [[07：FFN|FFN]]、[[13-15：Transformer Block 与前向传播|Transformer Block]]、[[21-22：Flash Attention 与 KV Cache|推理资源]]、[[26-27：LoRA、QLoRA 与量化|量化部署]]。
- **来源事实：** 第 30 章介绍 Router、Experts、Top-k、负载均衡、Mixtral 和 DeepSeek-V3；原始论文与技术报告用于核验稀疏门控、专家并行、公开配置和实验边界。
- **机制推导：** MoE 用动态路由把参数容量和每 Token 专家计算部分解耦，但权重驻留、Token 通信、负载尾部与算子效率形成新的系统瓶颈。
- **工程建议：** 只在目标工作负载和集群拓扑上比较 Dense 与 MoE；用路由健康、端到端质量、P50 / P99、显存和网络指标联合验收。
- **下一主题：** [[31：推理模型与测试时计算|推理模型与测试时计算]]、[[32：状态空间模型与混合架构|状态空间模型与混合架构]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-MoE论文与技术报告|MoE 原始论文与技术报告]]。
