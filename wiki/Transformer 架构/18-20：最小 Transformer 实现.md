---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [18, 19, 20]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-PyTorch-Module状态与Buffer.md"
---

# 第 18～20 章：最小 Decoder-Only Transformer 实现

> [!abstract] 一个最小语言模型项目不是只有 `Model` 类，而是三份代码共同形成闭环：`model.py` 把 Token 映射成 logits，`train.py` 用下一个 Token 目标更新参数，`inference.py` 用相同配置恢复参数并循环生成。真正需要理解的是三者共享的形状、词表、配置和检查点契约，而不是逐行背代码。

> [!info] 阅读粒度
> 本页以三份代码共享的系统契约为主题，因此标为“深入机制”；快速复述时，阅读摘要、“项目闭环”、职责表和“来源代码的教学边界”即可。

## 先看项目闭环

```text
训练文本
→ Tokenizer
→ Token ID 序列
→ 随机截取 x / y Batch
→ model(x, y)
→ logits + loss
→ backward + optimizer.step
→ checkpoint

Prompt
→ 同一个 Tokenizer
→ 重建同一种 Model
→ load_state_dict(checkpoint)
→ model.generate
→ Token ID
→ decode 为文本
```

三份文件的职责可以先记成：

| 文件 | 输入 | 核心职责 | 输出 |
| --- | --- | --- | --- |
| `model.py` | Token IDs、可选 targets | 定义参数、前向传播和最小生成循环 | logits、可选 loss、生成序列 |
| `train.py` | 训练文本、超参数 | 构造 Batch、评估、反向传播、更新与保存 | 训练后的 checkpoint |
| `inference.py` | checkpoint、Prompt | 恢复模型、关闭训练行为、执行解码 | 生成文本 |

## `model.py`：把概念变成可组合模块

最小实现通常按下面的依赖关系组织：

```text
Attention Head
→ Multi-Head Attention

FFN + Multi-Head Attention + Norm + Residual
→ Transformer Block

Token Embedding + Position + N × Block + Final Norm + LM Head
→ Model
```

关键不是类名，而是每层都遵守形状契约：

```text
Token IDs                         [B, T]
Embedding / residual stream       [B, T, d_model]
单头 Q、K、V                      [B, T, head_size]
Attention 分数                    [B, heads, T, T]
FFN 中间层                        [B, T, d_ff]
LM Head logits                    [B, T, vocab_size]
```

实现前至少检查：

```text
d_model % num_heads == 0
T <= context_length
0 <= token_id < vocab_size
```

### 为什么 Causal Mask 应注册为 Buffer

因果 Mask 不是需要优化器更新的参数，但通常需要随模型一起移动设备，因此适合注册为 Buffer：

```python
self.register_buffer("causal_mask", mask, persistent=False)
```

`persistent=False` 表示它不进入 `state_dict`，适合可由配置重新构造的固定 Mask；若某个 Buffer 确实属于必须保存的模型状态，再使用默认的持久 Buffer。同理，固定位置编码如果不会变化，也更适合在初始化时构造为 Buffer，而不是每次 `forward()` 都重新计算；是否持久化应按恢复契约决定。

### `forward()` 和 `generate()` 不是同一个层次

```text
forward：对当前输入做一次并行前向计算
         [B, T] → [B, T, vocab_size]

generate：反复调用 forward
          取最后位置 logits → 采样一个 Token → 拼回输入
```

`generate()` 是控制流程，不是新的神经网络层。没有 KV Cache 时，每生成一个 Token 都会重新计算截断后上下文中的全部位置。

## `train.py`：把连续 Token 流变成监督信号

训练数据通常是一条连续 Token 序列。随机选择起点后，输入与目标错开一位：

```text
原始：A B C D E F
x：   A B C D E
y：   B C D E F
```

模型一次输出所有位置的 logits：

```text
logits  [B, T, vocab_size]
targets [B, T]
```

计算交叉熵时，常把 Batch 和序列维展平：

```text
logits  → [B × T, vocab_size]
targets → [B × T]
```

一次普通训练迭代是：

```python
xb, yb = get_batch("train")
logits, loss = model(xb, yb)
optimizer.zero_grad(set_to_none=True)
loss.backward()
optimizer.step()
```

这段代码能更新参数，但还不能单独证明模型训练可靠。至少需要：

- 独立的训练集和验证集；
- `model.eval()` 与 `no_grad()` 下的周期性验证；
- 同时观察训练 loss 与验证 loss；
- 保存可恢复的检查点，而不只是最终权重文件。

## `inference.py`：恢复的是同一个模型契约

`state_dict` 只保存张量值，不会自动恢复 Python 类和架构定义。加载时必须先用兼容配置重建模型：

```python
checkpoint = torch.load(path, map_location=device)
model = Model(checkpoint["model_config"])
model.load_state_dict(checkpoint["model_state_dict"])
model.to(device)
model.eval()
```

然后在 `torch.inference_mode()` 下生成：

```python
with torch.inference_mode():
    output_ids = model.generate(prompt_ids)
```

下面几项必须和训练时一致：

- Tokenizer 及其词表；
- `vocab_size`、`d_model`、层数、头数和上下文长度；
- 位置表示类型、Norm、激活函数和参数命名；
- 特殊 Token 的 ID 和停止规则。

只要其中一项不一致，就可能出现无法加载、形状不匹配、Token 越界或生成乱码。

## 检查点不只是模型权重

如果检查点只用于推理，通常至少需要：

```text
model_state_dict
model_config
Tokenizer 标识或文件
特殊 Token 配置
```

如果还要从中断处继续训练，还应保存：

```text
optimizer_state_dict
scheduler_state_dict
当前 step / epoch
随机数状态
混合精度 scaler 状态（如使用）
数据进度或采样器状态（按需）
```

因此要区分：

- **推理检查点**：目标是可重建并生成；
- **恢复训练检查点**：目标是尽量继续原来的优化轨迹。

## 来源代码的教学边界

第 18～20 章提供的是帮助理解机制的最小实现，不应直接当作生产训练框架。需要特别注意：

1. **词表大小不能只取训练数据中最大的 Token ID。** 使用现成 Tokenizer 时，Prompt 可能出现训练集没出现过但属于合法词表的更大 ID；应使用 Tokenizer 的完整 `vocab_size`，或训练一套与模型绑定的小词表。
2. **设备不应永久写死在模型配置里。** 在模块内部创建张量时，优先跟随输入张量的 `device`，或使用 Buffer；加载检查点时使用 `map_location` 并由运行环境决定设备。
3. **固定位置编码不应每次前向重新构造。** 可预先计算并注册为 Buffer，减少重复工作并避免设备不一致。
4. **只保存模型参数不足以可靠恢复训练。** 还需要优化器、调度器、步数和随机状态。
5. **最小生成循环没有停止语义。** 如果不检查 EOS、停止字符串或最大上下文，只能依赖 `max_new_tokens` 强制结束。
6. **没有 KV Cache。** 每一步 Decode 都重复计算历史上下文，序列越长越慢。
7. **物理拆分多个 Attention Head 便于教学，但效率较低。** 实际实现通常一次投影后 reshape 成多头，利用批量矩阵计算。

## 最值得先做的三个验证

### 1. 形状与越界测试

```text
随机 Token IDs [B, T]
→ forward
→ 断言 logits.shape == [B, T, vocab_size]
→ 断言 loss 是有限标量
```

同时测试：`T > context_length`、非法 Token ID、`d_model` 不能整除头数时是否尽早报错。

### 2. 单 Batch 过拟合测试

固定一个很小的 Batch，反复训练。如果实现正确且模型容量足够，loss 应明显下降。若完全不下降，优先检查：

- x / y 是否正确错开一位；
- 因果 Mask 是否方向相反；
- 参数是否进入优化器；
- 梯度是否为零、NaN 或没有生成；
- 训练前向是否误包在 `no_grad()` / `inference_mode()` 中，或参数的 `requires_grad` 是否被关闭；`eval()` 本身不会关闭梯度，不应把它当作参数完全不更新的原因。

### 3. 保存—加载一致性测试

在 `eval()` 和无梯度模式下：

```text
同一输入
→ 保存前 logits
→ 保存并重新加载
→ 加载后 logits
→ 两者应在数值容差内一致
```

这个测试能同时验证模型配置、参数名、权重文件和加载流程是否匹配。

## 做项目时记住

1. **先固定接口，再扩展功能。** 先保证 `forward(idx, targets=None)`、检查点结构和 Tokenizer 契约稳定，再加入 AMP、梯度累积、分布式训练或 KV Cache。
2. **教学代码优先可读，生产代码优先可恢复和可观测。** 真正训练前要增加日志、验证、最佳模型保存、异常检测、资源预算和失败恢复。
3. **生成质量问题不只看采样参数。** 重复、乱码或不连贯也可能来自训练不足、数据质量、词表不一致、上下文截断或检查点错误。

## 别误解

- **“`model.py` 能运行，就说明训练代码正确。”** 前向形状正确不代表目标错位、梯度更新、验证和检查点流程正确。
- **“保存 `state_dict` 就保存了整个模型。”** 它不包含 Python 类定义、Tokenizer 和全部训练状态。
- **“训练 loss 下降就表示模型泛化更好。”** 还必须观察独立验证集；训练下降而验证上升通常意味着过拟合或分布差异。
- **“Temperature 和 Top-K 能修复模型知识不足。”** 它们只改变解码分布，不能补回模型没有学到的模式。
- **“这个最小实现就是现代 LLM 的完整训练系统。”** 它省略了大量性能、数值稳定、并行、数据治理和可靠性机制。

## 复习

1. `model.py`、`train.py` 和 `inference.py` 通过哪些配置与张量契约连接起来？
2. 为什么 `generate()` 必须循环调用 `forward()`，而训练可以一次计算多个位置？
3. 为什么使用现成 Tokenizer 时不能把训练数据中的最大 Token ID 当作完整词表大小？
4. 推理检查点与可恢复训练检查点分别必须保存什么？
5. 如果单 Batch 都无法过拟合，应该优先排查哪些环节？
6. 为什么保存—加载后的 logits 一致性是重要测试？

## 来源与下一步

- **前置概念：** [[13-15：Transformer Block 与前向传播|Transformer Block 与前向传播]]、[[16-17：训练、推理与参数更新|训练、推理与参数更新]]。
- **来源事实：** 第 18 章实现 FFN、因果 Attention、Multi-Head Attention、Pre-Norm Block、完整模型与生成循环；第 19 章实现 Token 化、随机 Batch、交叉熵、验证、AdamW、训练循环和检查点；第 20 章实现模型恢复、Prompt 编码、自回归采样、解码及常见生成问题诊断。
- **机制推导：** 三份文件通过 Tokenizer、模型配置、张量形状和 checkpoint schema 构成同一个系统契约；任一侧不一致都会在加载、训练或生成阶段暴露。
- **工程建议：** 来源代码适合概念验证；真正训练或部署前，应补齐完整词表、设备可移植性、恢复训练状态、停止条件、测试、日志与性能优化。
- **下一主题：** [[21-22：Flash Attention 与 KV Cache|第 21～22 章：Flash Attention 与 KV Cache]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-PyTorch-Module状态与Buffer|PyTorch Module 状态、Autograd 与 Buffer]]。
