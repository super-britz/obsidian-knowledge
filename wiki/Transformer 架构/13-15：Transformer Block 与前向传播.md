---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [13, 14, 15]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
---

# 第 13～15 章：Transformer Block 与前向传播

> [!abstract] 对 Decoder-Only 语言模型而言，模型维护一条贯穿所有 Block 的 residual stream。Attention 和 FFN 不必重建完整表示，而是分别计算“应该补充什么”的更新量，再通过残差连接写回主通道。多个参数独立的 Block 依次更新表示后，LM Head 才把 `d_model` 维隐藏状态转换成词表 logits。

> [!info] 阅读粒度
> 本页把 residual stream、Block 布局、输入位置表示与 LM Head 保留在同一条完整前向链中，因此标为“深入机制”；只需快速复述时，阅读摘要、“先抓住一条主线”和“完整前向流程”即可。

## 先明确本页范围

本页描述的是 GPT 风格的 **Decoder-Only、因果语言模型**，不是所有 Transformer 变体的统一结构：

- Attention 使用因果 Mask，只允许当前位置读取自己和之前的 Token；
- 没有 Encoder，也没有 Encoder-Decoder Cross-Attention；
- 不同模型可能采用可学习位置嵌入、RoPE、RMSNorm、不同激活函数或不同残差布局。

还要区分系统边界：

```text
文字 → Token ID：Tokenizer，通常在模型外部完成
Token ID → logits：模型的一次前向传播
logits → 下一个 Token：解码过程
```

## 先抓住一条主线

把 `residual stream` 想成一块从输入一直传到输出的**共享白板**：

1. Token Embedding 写下每个 Token 的初始内容表示，并通过相加或旋转等机制注入位置信息。
2. Attention 读取白板，在因果 Mask 约束下计算“还应该从前文取回什么信息”，再把增量加回白板。
3. FFN 读取更新后的白板，计算“当前 Token 的信息还应该怎样加工”，再把增量加回白板。
4. 下一个 Block 继续读取和更新同一条信息通道。
5. Final Norm 整理最终表示，LM Head 为词表中的每个 Token 计算分数。

因此，一个 Block 最值得记住的不是模块名称，而是：

> **读取当前表示 → 计算增量 → 加回主通道。**

## 完整前向流程

```text
Token ID [B, T]
→ Token Embedding [B, T, d_model]
→ 注入位置信息
→ 初始 residual stream X₀
→ 重复 N 个 Transformer Block
     ├─ Norm → Causal Attention → Dropout → 残差相加
     └─ Norm → FFN              → Dropout → 残差相加
→ Final Norm [B, T, d_model]
→ LM Head
→ 词表 logits [B, T, vocab_size]
```

以常见的 Pre-Norm Block 为例，一层可以写成：

```text
A  = Attention(Norm(X), causal_mask)
X' = X + Dropout(A)

F  = FFN(Norm(X'))
Y  = X' + Dropout(F)
```

这里的 `Y` 会成为下一个 Block 的输入。**重复 N 次指结构重复，不代表权重共享**；标准堆叠中，每个 Block 通常有自己独立的 Attention、FFN 和 Norm 参数。

## 关键理解

### 1. Block 的任务不是替换 `X`，而是给 `X` 增加修正量

如果没有残差连接，子层需要直接产出下一层所需的完整表示；有了 `X + F(X)`，子层只需学习“在已有信息上补充什么”。

- `X` 保留进入子层前已经积累的信息；
- `F(X)` 是 Attention 或 FFN 新算出的增量；
- `X + F(X)` 把旧信息和新信息合并后继续传递。

在反向传播时，加法还提供了一条直接梯度路径。对 `Y = X + F(X)`：

```text
∂Y/∂X = I + ∂F(X)/∂X
```

即使经过子层的梯度很小，恒等项 `I` 仍给梯度保留直接路径。因此残差连接同时影响：

- **前向信息流**：保留已有表示；
- **反向梯度流**：减少深层网络训练困难；
- **学习目标**：让子层学习增量，而非重建完整表示。

### 2. Attention 和 FFN 都在写 residual stream，但写入内容不同

- **Causal Attention** 负责 Token 之间的信息交换：当前位置应该从允许访问的前文位置取回什么；
- **FFN** 负责分别加工每个 Token 已经汇总好的信息。

可以简化为：

```text
Attention：跨 Token 收集信息
FFN：在单个 Token 内加工信息
残差连接：把两种更新依次积累起来
```

### 3. Pre-Norm 和 Post-Norm 的关键差异是 Norm 在残差加法哪一侧

常见的简化写法是：

```text
Pre-Norm： X_next = X + F(Norm(X))
Post-Norm：X_next = Norm(X + F(X))
```

Pre-Norm 中，子层读取规范化后的表示，但残差分支直接保留 `X`；Post-Norm 则在子层输出与残差相加后再做 Norm。

所以对 Pre-Norm 要区分：

- **子层读取的是**规范化后的表示；
- **残差分支保留的是**进入该子层前的原表示。

Pre-Norm 通常更容易训练深层网络，但它不是 Transformer 的唯一布局；具体模型还可能使用 RMSNorm 或其他变体。

### 4. Dropout 扰动的是更新分支，不是永久删除参数

图中常见的数据流是：

```text
X + Dropout(F(Norm(X)))
```

Dropout 位于 Attention / FFN 输出之后、残差相加之前。训练时，若丢弃概率为 `p`，常见的 inverted dropout 可写成：

```text
mask_i ~ Bernoulli(1 - p)
output_i = input_i × mask_i / (1 - p)
```

除以 `1-p` 是为了让训练时输出的期望尺度与推理时大致一致。推理时 Dropout 关闭，也不会修改或删除模型权重。

Dropout 率不是 Transformer 的固定常数；有些模型会设得很低或直接设为零。

### 5. “Token 信息 + 位置信息”只适用于可加位置表示

在可加位置表示中，Token Embedding 和位置向量都使用 `d_model` 维，因此可以逐元素相加：

```text
初始表示 = Token Embedding + Position Embedding
[B, T, d_model] + [T, d_model]
→ [B, T, d_model]
```

相加的主要工程优势是保持 residual stream 宽度不变；若直接拼接，最后一维会变成 `2 × d_model`，后续投影矩阵和计算量都要随之改变。

但不能把“位置表示”一概理解成入口处相加：

- 固定正弦位置编码和可学习绝对位置嵌入通常可以直接相加；
- RoPE 通常在 Attention 内部旋转 Q、K，而不是加到初始 residual stream；
- 无论采用哪种方式，内容和位置信号的数值尺度都需要协调，避免一方压过另一方。

### 6. LM Head 改变最后一维，logits 之后才进入训练或解码

Block 堆叠始终维护 `d_model` 宽度，LM Head 才把隐藏状态映射到词表维度：

```text
hidden states: [B, T, d_model]
LM Head 权重:  [vocab_size, d_model]
logits:        [B, T, vocab_size]
```

LM Head 的每一行可以理解为一个词表 Token 的输出方向。很多模型会让 LM Head 与 Token Embedding **共享权重**（weight tying），但这是一种常见设计，不是所有模型的硬性要求。

logits 是未归一化分数：

- 训练时，交叉熵函数通常可以直接接收 logits，不需要先手动 Softmax；
- 生成时，通常只取最后一个有效位置的 logits，再执行 Greedy、Temperature、Top-K 或 Top-P 等解码；
- 一次 `model(input)` 只完成当前输入的前向计算，不会自动更新参数，也不会自动生成完整回答。

## 用形状检查完整流程

```text
Token IDs                       [B, T]
Token / Position 表示           [B, T, d_model]
Transformer Block 输入与输出    [B, T, d_model]
Attention 分数                  [B, heads, T, T]
FFN 中间表示                    [B, T, d_ff]
Final Norm                      [B, T, d_model]
LM Head 输出 logits             [B, T, vocab_size]
生成时最后有效位置              [B, vocab_size]
```

检查代码时应抓住两个形状边界：

1. Attention 和 FFN 内部可以拆头、扩宽或变形，但写回 residual stream 前必须恢复到 `d_model`；
2. 只有 LM Head 会把最后一维从 `d_model` 明确映射为 `vocab_size`。

## 做项目时记住

1. **先确认架构范围。** Decoder-Only、Encoder-Only 和 Encoder-Decoder 的 Block 与 Mask 不完全相同。
2. **读模型代码先追 residual stream。** 找 `[B, T, d_model]` 在哪里被读取、在哪里加回，再进入头拆分和 FFN 扩宽细节。
3. **看到一个子层就问三个问题：** 它读取什么、计算什么增量、增量写回哪里。
4. **不要把层数理解成循环复用同一组参数。** 先检查代码是 `ModuleList` 的独立 Block，还是显式做了权重共享。
5. **排查输出错误时检查 LM Head 边界。** 确认最终维度是 `vocab_size`，训练目标的 Token ID 范围与词表一致。
6. **不要混淆前向、训练和解码。** `forward` 计算 logits，训练还需要 loss / backward / optimizer，生成还需要解码和下一轮输入。

## 别误解

- **“每经过一层，旧表示就被新表示替换。”** 旧表示沿残差分支保留，子层计算的增量被加到它上面。
- **“残差连接只是为了防止梯度消失。”** 它还保留前向信息，并把子层目标变成增量更新。
- **“Dropout 会在模型中永久删除一些神经元或权重。”** 它只在训练期对当次前向的激活做随机掩码。
- **“所有位置编码都与 Token Embedding 相加。”** RoPE 等方法在 Attention 内部注入位置关系。
- **“N 个 Block 就是同一个 Block 重复调用 N 次。”** 标准堆叠通常结构相同但参数独立。
- **“LM Head 后必须先 Softmax，才能计算训练 loss。”** 常见交叉熵实现直接接收 logits，内部完成所需计算。

## 哪些来源内容没有展开

下面内容在来源章节中出现，但不是本页主线的必要部分：

- 各公开模型的具体 Dropout 率和参数配置，具有模型与版本依赖；
- 缩放残差、门控残差等变体，适合在架构变体主题中讨论；
- GPT-2 Small 的参数量占比示例，适合理解规模分布，但不影响前向传播机制；
- Attention 热力图、具体 Token 示例和完整 PyTorch 实现，分别由 Attention 笔记和第 18～20 章承接。

## 复习

1. 为什么说 Transformer Block 学习的是对 residual stream 的增量更新？
2. Pre-Norm 与 Post-Norm 的残差公式有什么差异？
3. 为什么训练期 Dropout 要除以 `1-p`，推理时又要关闭？
4. 可加位置嵌入与 RoPE 分别在什么位置注入位置信息？
5. 为什么 Block 输入输出保持 `d_model`，LM Head 输出却是 `vocab_size`？
6. “堆叠 N 个 Block”为什么通常不等于共享同一组参数？

## 来源与下一步

- **来源事实：** 第 13 章讲残差梯度路径、Dropout、Pre-Norm / Post-Norm 和训练稳定性；第 14 章讨论 Token Embedding 与位置表示的结合方式及位置编码变体；第 15 章串联 Decoder-Only 输入、Block 堆叠、Final Norm、LM Head、词表输出和训练边界。
- **机制推导：** 阅读 Transformer 实现时，可把 Block 统一理解为“读取 residual stream、计算增量、写回 residual stream”，再把 LM Head 看成从隐藏空间到词表空间的独立边界。
- **下一主题：** [[16-17：训练、推理与参数更新|第 16～17 章：训练、推理与参数更新]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
