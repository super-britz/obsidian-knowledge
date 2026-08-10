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
> Transformer 把文字变成向量，经过多层 Block 加工，再预测下一个 Token。

## 先记住主流程

![[raw/assets/Transformer 架构/ch03-01-simplified-flow.svg]]

整个过程只有三步：

1. **输入处理**：文字 → Token ID → Embedding（词嵌入）+ Positional Encoding（位置编码）。
2. **核心加工**：经过 N 层 Transformer Block。
3. **输出预测**：Linear（线性层）→ logits（原始分数）→ Softmax（概率）→ 下一个 Token。

## 一个 Block 做什么

![[raw/assets/Transformer 架构/ch03-02-standard-architecture.jpg]]

从下向上看，每个 Block 主要做两件事：

- **Attention（注意力）**：让每个 Token 查找上下文中与自己有关的信息。
- **FFN（前馈神经网络）**：进一步处理 Attention 得到的信息。

同时使用两个辅助结构：

- **Residual Connection（残差连接）**：保留原始信息。
- **LayerNorm（层归一化）**：让数值更稳定。

> [!important] Block 的核心
> Attention 负责“看上下文”，FFN 负责“加工信息”。同样的结构重复 N 层，但每层通常使用不同参数。

## 术语速查

| 英文 | 中文 | 作用 |
| --- | --- | --- |
| Tokenization | 分词/编码 | 文字 → 数字 |
| Embedding | 嵌入 | 数字 → 向量 |
| Positional Encoding | 位置编码 | 添加位置信息 |
| Multi-Head Attention | 多头注意力 | 理解词之间的关系 |
| Layer Norm | 层归一化 | 稳定数值范围 |
| Feed Forward | 前馈网络 | 信息处理 |
| Residual Connection | 残差连接 | 跳跃连接 |
| Softmax | 概率转换 | 转换为概率 |

> [!note]- 展开查看详细架构图
> ![[raw/assets/Transformer 架构/ch03-03-detailed-architecture.webp]]
>
> 左侧展开了 MHA（Multi-Head Attention，多头注意力）的内部计算。本章先理解主流程，不必记住内部公式。

## 面试速答

### 1. Transformer 的整体流程是什么？

文字先转换为带位置信息的向量，再经过 N 层 Transformer Block，最后由 Linear 和 Softmax 得到下一个 Token 的概率。

### 2. Transformer Block 的核心是什么？

核心是 Attention（注意力）和 FFN（前馈神经网络），并配合残差连接与层归一化。

### 3. 为什么需要位置编码和 Mask（掩码）？

位置编码告诉模型词语顺序；Mask 遮住未来 Token，防止模型提前看到答案。

## 复习检查

- [ ] 能复述“输入处理 → N 层 Block → 输出预测”。
- [ ] 能说清 Attention 和 FFN 的分工。
- [ ] 能解释位置编码和 Mask 的作用。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/02-大模型的本质|第 2 章：大模型的本质]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 已有课程：[[wiki/直播课/课程笔记/40-大语言模型原理|40 大语言模型原理]]

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第三章原文：https://waylandz.com/llm-transformer-book/第03章-Transformer全景图/
