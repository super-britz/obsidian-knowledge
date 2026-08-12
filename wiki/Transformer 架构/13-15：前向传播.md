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

> [!abstract] 一个 Transformer Block 不只是 Attention 和 FFN 的顺序连接。它用残差连接保留原信息，用 Norm 稳定子层输入，用可选 Dropout 做训练期正则化；许多 Block 堆叠后，再把最终隐藏状态映射成词表 logits。

## 先看完整流程

```text
Token ID
→ Token Embedding + 位置表示
→ residual stream X
→ Norm → Attention →（训练时可选 Dropout）→ + X
→ Norm → FFN       →（训练时可选 Dropout）→ + X
→ 重复 N 个 Block → Final Norm
→ LM Head → 词表 logits
→ 训练：所有位置算 loss / 生成：通常取最后位置选下一个 Token
```

最重要的是：**Block 入口和出口的 residual stream 宽度通常保持 `d_model`，但内部 Attention、头拆分和 FFN 会出现不同形状。**

## 关键理解

- **残差连接就是把子层输出加回原输入。** `X + F(X)` 给信息和梯度保留一条直接路径；某层即使暂时学不到有用更新，也不会轻易破坏已有表示。
- **Pre-Norm 是常见布局。** 先对输入做 Norm，再进入 Attention / FFN，最后把子层更新加回 residual stream。它是现代 decoder-only LLM 的常见起点，但不是唯一架构答案。
- **Dropout 只在训练时随机置零部分激活。** 它用于减轻过拟合；推理时关闭，不会永久删除权重。很多现代大模型的 Dropout 率也可能很低或为零。
- **Token 信息和位置表示通常相加。** 两者维度相同，相加能保留 `d_model` 宽度，避免拼接后让后续矩阵变宽、计算变贵；模型会学习怎样同时解读内容与顺序。
- **前向传播只产生 logits，不修改参数。** 训练还要经过 loss、反向传播和优化器更新；生成则通常只用最后位置的 logits 选择或采样下一个 Token。

## 做项目时记住

1. **读模型代码先追 residual stream。** 关注 `[B, T, d_model]` 如何进出每个 Block，再看内部的头拆分和 FFN 扩宽，最不容易迷路。
2. **不要混淆训练与推理。** `model(input)` 只是前向计算；只有 `loss.backward()` 加 `optimizer.step()` 才会更新参数。
3. **排查形状错误时别假设全程形状不变。** Block 外部宽度通常固定，但 Attention 分数会是 `[B, heads, T, T]`，FFN 中间层会比 `d_model` 更宽。

## 别误解

- **“残差连接等于简单绕过某层，所以该层没用。”** 它让子层学习对原表示的增量更新，而不是强迫每层从头重建全部信息。
- **“Dropout 在推理时也会随机丢信息。”** 正常推理模式会关闭 Dropout。
- **“logits 就是概率。”** logits 是未归一化分数；需要概率或采样时才进一步使用 Softmax 等解码步骤。

## 复习

1. 为什么 Block 中要把 Attention / FFN 的结果加回 residual stream？
2. Token Embedding 和位置表示为什么常相加而不是拼接？
3. 从输入 Token 到生成下一个 Token，前向计算、反向传播和优化器各自负责什么？

## 来源与下一步

- **来源事实：** 第 13 章讲残差、Dropout 与 Pre-Norm；第 14 章讨论词嵌入和位置表示的相加；第 15 章将输入、Block 堆叠、Final Norm、LM Head、训练与生成串成完整前向流程。
- **本页推论：** 对应用工程而言，能追踪 residual stream、区分内部形状并分清前向 / 更新边界，比背具体模型参数量更重要。
- **下一主题：** 第 16～17 章：训练与自回归推理。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
