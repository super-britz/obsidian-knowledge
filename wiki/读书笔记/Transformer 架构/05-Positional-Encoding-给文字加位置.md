---
type: 读书笔记
status: 学习中
created: 2026-08-10
updated: 2026-08-10
book: Transformer 架构：从直觉到实现
chapter: 5
topics: [Positional Encoding, Embedding, Attention, Transformer]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 5 章：Positional Encoding - 给文字加位置

> [!abstract] 一句话理解
> Embedding 表示 Token“是什么”，位置编码表示 Token“在哪里”。两者合起来，Transformer 才能理解顺序。

## 核心结论

- Embedding 只记录内容，不记录位置。
- Transformer 需要额外加入位置编码，才能区分不同的顺序。
- 最基本的做法是：`内容向量 + 位置向量 = Transformer 的输入`。

## 主流程：给内容补上顺序

```text
文字
  ↓ Tokenization（分词 / 编码）
Token
  ↓ Embedding（嵌入）
内容向量：这个 Token 是什么
  +
位置向量：这个 Token 在哪里
  ↓
完整输入
  ↓ Transformer
结合上下文理解
```

这一章只需要先记住一条主线：

```text
是什么 + 在哪里 → Transformer 才能理解顺序
```

## 核心概念

### 1. Embedding 解决了什么问题

第四章把 Token 转成了向量。这个向量主要表示 Token 的内容，但同一个 Token 出现在不同位置时，查到的向量仍然相同。

所以，Embedding 能回答：

> 这个 Token 是什么？

但它不能单独回答：

> 这个 Token 在第几个位置？

### 2. 位置编码解决了什么问题

位置编码给每个位置一个不同的向量，让模型知道：

- Token 的先后顺序；
- Token 之间的大致距离；
- 相同 Token 出现在不同位置时，输入表示应该不同。

Attention（注意力机制）本身不会自动知道顺序，所以需要先把位置放进输入中。

### 3. 为什么使用正弦和余弦

原始 Transformer 使用 Sinusoidal Positional Encoding（正弦位置编码）：

- 偶数维使用 `sin`；
- 奇数维使用 `cos`；
- 不同维度使用不同变化速度。

这样，每个位置都会形成一个不同的模式，而且数值始终在 `[-1, 1]`。相比直接加上 `1、2、3……`，它不会随着位置变大而不断放大，也不会让所有维度沿同一个方向变化。

先理解到这里即可：**不同位置有不同模式，模型学习如何使用这些模式。**

### 4. 内容和位置如何合成

```text
Embedding E [seq_len, d_model]
Position  P [seq_len, d_model]
              ↓ 相加
Input     [seq_len, d_model]
```

- `seq_len`（sequence length，序列长度）：有多少个 Token；
- `d_model`（model dimension，模型维度）：每个 Token 向量有多宽。

相加后，数字的值变了，但向量的长度没有变，所以仍然可以直接送入 Transformer。

混合后的信息已经够模型使用，因为模型不需要把“内容”和“位置”重新拆开。它只需要根据整个向量的变化，同时判断 Token 的内容和顺序。位置编码有规律，后续层可以学习如何利用这种规律。

选择相加还有一个实际好处：维度保持不变。如果改用拼接，向量会变宽，后面通常还要再压回原来的维度，计算也会增加。

> [!note]- 补充：原始论文的缩放
> 原始 Transformer 论文写成 `Input = √d_model × E + P`。这是实现细节，当前阶段先记住“内容向量和位置向量相加”即可。

### 5. 固定编码和可学习编码

位置向量有两种来源：

| 方式 | 含义 |
| --- | --- |
| 固定正弦编码 | 用公式计算位置，不作为模型参数更新 |
| 可学习位置编码 | 把位置向量当作参数，训练时一起更新 |

两者都在做同一件事：给模型提供位置线索。区别只是位置向量是由公式产生，还是由训练学出来。

> [!note]- 先不用掌握的后续术语
> RoPE（Rotary Position Embedding，旋转位置嵌入）和 ALiBi（Attention with Linear Biases，带线性偏置的注意力）是后续的位置编码方案。它们不再完全按照本章的“输入端直接相加”来处理位置，留到后面的演进章节再学。

## 术语速查

| 英文 / 缩写 | 中文 | 先记住什么 |
| --- | --- | --- |
| Token | 文本单元 | 文字被切分后的单位 |
| Embedding | 嵌入 | 表示 Token 的内容 |
| Positional Encoding | 位置编码 | 表示 Token 位于哪里 |
| Attention | 注意力机制 | 结合上下文信息 |
| `seq_len`（sequence length） | 序列长度 | Token 的数量 |
| `d_model`（model dimension） | 模型维度 | 每个向量的宽度 |

## 复习问题

> 以下问题是根据本章概念整理的自测题，不是原文提供的面试题。

1. Embedding 和位置编码分别表示什么？
2. 为什么 Transformer 需要额外加入位置编码？
3. `内容向量 + 位置向量` 之后，得到了什么？
4. 固定正弦编码和可学习位置编码有什么区别？

## 复习检查

- [ ] 能说清“Embedding 表示是什么，位置编码表示在哪里”。
- [ ] 能复述“内容向量 + 位置向量 → Transformer 输入”。
- [ ] 能解释正弦位置编码只是给不同位置生成不同模式。
- [ ] 能区分固定位置编码和可学习位置编码。
- [ ] 暂时不要求背诵正弦公式、RoPE 和 ALiBi 的细节。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/04-Tokenization-文字如何变成数字|第 4 章：Tokenization - 文字如何变成数字]]
- 前置全景：[[wiki/读书笔记/Transformer 架构/03-Transformer 全景图|第 3 章：Transformer 全景图]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 下一章：[[wiki/读书笔记/Transformer 架构/06-LayerNorm与Softmax-数字的缩放与概率化|第 6 章：LayerNorm 与 Softmax - 数字的缩放与概率化]]

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第五章原文：https://waylandz.com/llm-transformer-book/第05章-Positional-Encoding-给文字加位置/
