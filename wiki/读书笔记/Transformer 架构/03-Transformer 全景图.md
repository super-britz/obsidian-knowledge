---
type: 读书笔记
status: 学习中
created: 2026-08-10
updated: 2026-08-10
book: Transformer 架构：从直觉到实现
chapter: 3
topics: [Transformer, Tokenization, Embedding, Positional Encoding, Attention, LayerNorm, FFN, Softmax]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 3 章：Transformer 全景图

> [!abstract] 一句话理解
> Transformer 把文字转换为带位置信息的向量，经过多层 Transformer Block 加工，再输出下一个 Token 的概率分布。

## 核心结论

- 输入端负责把文字变成带位置信息的向量。
- Transformer Block 的核心是 Attention 和 FFN，并配合残差连接与 LayerNorm。
- 输出端把隐藏状态转换为词表上的分数和概率，得到下一个 Token 的预测。

## 主流程：从文字到下一个 Token

![[raw/assets/Transformer 架构/ch03-01-simplified-flow.svg]]

```text
文字
  ↓ Tokenization + Embedding + Positional Encoding
带位置信息的向量
  ↓ N 层 Transformer Blocks
上下文表示
  ↓ Linear
logits（原始分数）
  ↓ Softmax
Token 概率分布
  ↓
下一个 Token
```

## 核心概念

### 1. 输入处理

输入文字先经过：

1. **Tokenization（分词 / 编码）**：文字 → Token ID；
2. **Embedding（嵌入）**：Token ID → 初始向量；
3. **Positional Encoding（位置编码）**：把位置信息加入向量。

Transformer 需要位置编码，是因为 Attention 本身主要处理 Token 之间的关系，不会自动知道顺序。

### 2. Transformer Block

![[raw/assets/Transformer 架构/ch03-02-standard-architecture.jpg]]

每个 Transformer Block 主要包含两部分：

- **Attention（注意力）**：让 Token 获取上下文中与自己相关的信息；
- **FFN（Feed-Forward Network，前馈网络）**：进一步变换和加工信息。

同时配合两个辅助结构：

- **Residual Connection（残差连接）**：保留并传递原始信息；
- **LayerNorm（Layer Normalization，层归一化）**：稳定网络中的数值。

```text
Attention：看上下文
FFN：加工信息
Residual：保留信息
LayerNorm：稳定数值
```

同样的 Block 结构会重复 N 层，但每一层通常使用不同的参数。

### 3. 输出预测

经过多层 Block 后，模型得到上下文表示，再依次经过：

```text
Linear（线性层）
  ↓
logits（原始分数）
  ↓
Softmax（概率化）
  ↓
词表上的 Token 概率分布
```

最终根据概率分布选择或采样下一个 Token。

### 4. Mask（掩码）

自回归语言模型需要使用 Causal Mask（因果掩码）遮住未来位置，防止模型在预测当前 Token 时提前看到答案。

## 术语速查

| 英文 / 缩写 | 中文 | 在流程中的作用 |
| --- | --- | --- |
| Tokenization | 分词 / 编码 | 文字 → Token ID |
| Embedding | 嵌入 | Token ID → 向量 |
| Positional Encoding | 位置编码 | 添加位置信息 |
| Attention | 注意力 | 获取上下文相关信息 |
| Multi-Head Attention | 多头注意力 | 从多个表示空间处理关系 |
| FFN（Feed-Forward Network） | 前馈网络 | 进一步加工信息 |
| Residual Connection | 残差连接 | 保留和传递原始信息 |
| LayerNorm（Layer Normalization） | 层归一化 | 稳定数值 |
| Linear | 线性层 | 隐藏状态 → 词表分数 |
| logits | 原始分数 | Softmax 前的分数 |
| Softmax | 概率化 | 分数 → 概率 |
| Mask | 掩码 | 限制可见位置 |

## 复习问题

> 以下问题是根据本章概念整理的自测题，不是原文提供的面试题。

1. Transformer 从文字到下一个 Token 的完整流程是什么？
2. Embedding 和 Positional Encoding 分别解决什么问题？
3. Transformer Block 中 Attention 和 FFN 的分工是什么？
4. Residual Connection 和 LayerNorm 分别有什么作用？
5. 为什么自回归语言模型需要 Causal Mask？

## 复习检查

- [ ] 能复述“输入处理 → N 层 Transformer Block → 输出预测”。
- [ ] 能说明 Attention、FFN、残差连接和 LayerNorm 的分工。
- [ ] 能解释位置编码和 Causal Mask 的作用。
- [ ] 能区分 logits 和 Softmax 后的概率。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/02-大模型的本质|第 2 章：大模型的本质]]
- 下一章：[[wiki/读书笔记/Transformer 架构/04-Tokenization-文字如何变成数字|第 4 章：Tokenization - 文字如何变成数字]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 已有课程：[[wiki/直播课/课程笔记/40-大语言模型原理|40 大语言模型原理]]

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第三章原文：https://waylandz.com/llm-transformer-book/第03章-Transformer全景图/
