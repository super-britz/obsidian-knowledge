---
type: 主题笔记
status: 已整理
source_chapters: [13, 14, 15]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
---

# 第 13～15 章：Transformer Block 与前向传播

> [!abstract] Transformer 不会让每个子层重新生成一份全新的表示，而是维护一条贯穿所有 Block 的主信息通道。Attention 和 FFN 每次只计算一份“应该补充什么”的更新量，再通过残差连接写回这条通道。多个 Block 反复更新后，LM Head 才把最终表示转换成词表 logits。

## 先抓住一条主线

把 `residual stream` 想成一块从输入一直传到输出的**共享白板**：

1. Token Embedding 和位置表示先写下初始信息。
2. Attention 读取白板，计算“还应该从其他 Token 取回什么信息”，再把结果加回白板。
3. FFN 读取更新后的白板，计算“当前 Token 的信息还应该怎样加工”，再把结果加回白板。
4. 下一个 Block 继续在同一块白板上读和写。
5. 最后的 LM Head 根据白板上的最终表示，为词表中的每个 Token 打分。

因此，一个 Block 最值得记住的不是模块名称，而是：

> **读取当前表示 → 计算增量 → 加回主通道。**

## 完整前向流程

```text
Token ID
→ Token Embedding + 位置表示
→ 初始 residual stream X₀
→ 重复 N 个 Transformer Block
     ├─ Attention：读取 X，写回上下文增量
     └─ FFN：读取 X，写回逐 Token 加工增量
→ Final Norm
→ LM Head
→ 词表 logits
```

以常见的 Pre-Norm Block 为例，一层可以写成：

```text
A  = Attention(Norm(X))
X' = X + Dropout(A)

F  = FFN(Norm(X'))
Y  = X' + Dropout(F)
```

这里的 `Y` 会成为下一个 Block 的输入。图中的 Dropout 只在训练时启用，并且有些模型会把 Dropout 率设得很低或直接设为零。

## 关键理解

### 1. Block 的任务不是替换 `X`，而是给 `X` 增加修正量

如果没有残差连接，子层需要直接产出下一层所需的完整表示；有了 `X + F(X)`，子层只需学习“在已有信息上补充什么”。

- `X` 保留进入子层前已经积累的信息。
- `F(X)` 是 Attention 或 FFN 新算出的增量。
- `X + F(X)` 把旧信息和新信息合并后继续传递。

所以残差连接不是“绕过这一层，让这一层没用”，而是把每个子层从**重建全部信息**改成**增量更新信息**。

### 2. Attention 和 FFN 都在写 residual stream，但写入的内容不同

- **Attention** 负责 Token 之间的信息交换：当前 Token 应该从前文哪些位置取回什么。
- **FFN** 负责分别加工每个 Token 已经汇总好的信息。

可以简化为：

```text
Attention：跨 Token 收集信息
FFN：在单个 Token 内加工信息
残差连接：把两种更新依次积累起来
```

### 3. Pre-Norm 是“先整理读数，再计算更新”，不是把主通道清空

在 Pre-Norm 中，`Norm(X)` 被送入 Attention 或 FFN，但残差分支仍直接保留原来的 `X`。Norm 的作用是给子层提供较稳定的输入尺度；子层算出的结果随后再加回主通道。

因此要区分：

- **子层读取的是**规范化后的表示；
- **残差连接保留的是**进入该子层前的原表示。

Pre-Norm 是常见布局，但不是 Transformer 的唯一布局。

### 4. 输入表示必须先变成统一宽度，才能进入这条主通道

在采用可加位置表示的设计中，Token Embedding 和位置表示都使用 `d_model` 维，因此可以逐元素相加：

```text
初始表示 = Token 信息 + 位置信息
```

相加后形状仍是 `[B, T, d_model]`，可以直接进入后续 Block。若改成拼接，最后一维会变宽，后续投影矩阵和计算量也要随之改变。

### 5. 前向传播到 logits 为止，不等于模型已经学习或已经选出答案

- **前向传播**：根据当前参数，从输入算出 logits。
- **训练**：还要计算 loss、反向传播梯度，再由优化器更新参数。
- **生成**：还要从最后位置的 logits 中选择或采样下一个 Token。

因此，`model(input)` 本身既不会自动更新参数，也不等于已经完成整段文本生成。

## 用形状检查这条主线

Block 外部最稳定的形状是 residual stream：

```text
[B, T, d_model] → Transformer Block → [B, T, d_model]
```

但子层内部会暂时变形：

- Attention 分数常见为 `[B, heads, T, T]`；
- 每个 Attention Head 会使用拆分后的 head 维度；
- FFN 中间层通常会扩宽，再投影回 `d_model`。

这些内部结果最终都必须回到 `[B, T, d_model]`，才能与残差分支相加并交给下一个 Block。

## 做项目时记住

1. **读模型代码先追 residual stream。** 先找 `[B, T, d_model]` 在哪里被读取、在哪里加回，再进入 Attention 头拆分和 FFN 扩宽细节。
2. **看到一个子层就问三个问题：** 它读取什么、计算什么增量、增量写回哪里。
3. **排查形状错误先看残差相加处。** 相加两边的形状必须兼容；内部可以变形，写回前必须恢复到主通道宽度。
4. **不要混淆前向与参数更新。** `forward` 只计算结果，`backward` 计算梯度，`optimizer.step()` 才修改参数。

## 别误解

- **“每经过一层，旧表示就被新表示替换。”** 更准确的理解是：旧表示沿残差分支保留，子层计算的增量被加到它上面。
- **“残差连接只是为了防止梯度消失。”** 它也改变了子层要学习的问题：从生成完整表示变成学习增量更新。
- **“Dropout 在推理时也会随机丢信息。”** 正常推理模式会关闭 Dropout。
- **“logits 就是概率或最终 Token。”** logits 是未归一化分数，还需要 Softmax 或其他解码步骤才能用于选择 Token。

## 复习

1. 为什么说 Transformer Block 学习的是对 residual stream 的“增量更新”？
2. Attention 和 FFN 分别向 residual stream 写入什么类型的信息？
3. 在 Pre-Norm 中，子层读取的表示和残差分支保留的表示有什么区别？
4. 从 logits 到参数更新或下一个 Token，中间还缺少哪些步骤？

## 来源与下一步

- **来源事实：** 第 13 章讲残差、Dropout 与 Pre-Norm；第 14 章讨论词嵌入和位置表示的相加；第 15 章将输入、Block 堆叠、Final Norm、LM Head、训练与生成的前向边界串成完整流程。
- **本页推论：** 阅读 Transformer 实现时，把 Block 统一理解为“读取 residual stream、计算增量、写回 residual stream”，比把 Attention、Norm、Dropout 和 FFN 当成互不相关的模块更容易迁移到不同模型代码。
- **下一主题：** [[16-17：训练与推理|第 16～17 章：训练与推理]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
