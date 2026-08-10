---
type: 读书笔记
status: 学习中
created: 2026-08-10
updated: 2026-08-10
book: Transformer 架构：从直觉到实现
chapter: 4
topics: [Tokenization, Token, BPE, Embedding, Context Length]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 4 章：Tokenization - 文字如何变成数字

> [!abstract] 一句话理解
> Tokenization 把文字转换为 Token ID，Embedding 把 Token ID 转换为初始向量，Transformer 再结合上下文加工这些向量。

## 核心结论

- Tokenization 负责“文字 → Token → Token ID”。
- Token ID 只是词表中的编号，不是语义向量。
- Embedding 通过查表把 Token ID 转换为向量。
- Context Length 约束 Token 序列的长度。
- Transformer Block 负责把初始向量加工成上下文表示。

## 主流程：从文字到向量

```text
原始文字
  ↓ Tokenization（分词 / 编码）
Token 序列
  ↓ Vocabulary Lookup（词表查找）
Token ID 序列
  ↓ Embedding Lookup（嵌入查表）
初始向量矩阵
  ↓ Transformer Blocks（Transformer 模块）
上下文表示
```

可以概括为：

> **先切分，再编号；先查表得到初始向量，再由 Transformer 结合上下文。**

## 核心概念

### 1. Tokenization（分词 / 编码）

Tokenization 把连续文字切成模型能够处理的基本单位。

这些单位叫 **Token（文本单元）**，可能是字符、词、词片段或字节片段，不固定等于一个字或一个单词。

**BPE（Byte Pair Encoding，字节对编码）** 是常见的 Tokenization 方法：

- 合并常见片段，减少 Token 数量；
- 将罕见文字拆成更小的字节片段；
- 让有限词表能够覆盖更多文字。

不同 Tokenizer（分词器）对同一段文字的切分结果可能不同。

### 2. Vocabulary（词表）与 Token ID

Vocabulary（词表）保存 Token 以及对应的编号。

**Token ID（Token Identifier，Token 标识符 / 编号）** 是 Token 在词表中的数字索引：

```text
Token 序列 → Token ID 序列
```

Token ID 只是查表用的行号，不是语义向量，也不是模型对文字的最终理解。

### 3. Context Length（上下文长度）

Context Length 是模型一次能够处理的 Token 数量预算，通常由输入和输出共同使用。

如果 Tokenization 后得到 `n` 个 Token，需要满足：

```text
n ≤ Context Length
```

因此，Context Length 约束的是 Token ID 序列的长度，而不是原始文字的字数。

### 4. Embedding（嵌入）

Embedding 把离散的 Token ID 转换为连续向量。模型内部有一张 **Embedding Lookup Table（嵌入查找表）**：

```text
表的形状 = [vocab_size, d_model]
```

- `vocab_size`（Vocabulary Size，词表大小）：表的行数；
- `d_model`（model dimension，模型维度）：每行向量的维度；
- Token ID：要查找的行号。

如果 Token ID 序列长度为 `n`：

```text
Token ID 序列 [n]
        ↓ 查 Embedding 表
初始向量矩阵 [n, d_model]
```

### 5. 初始向量与上下文表示

Embedding 查到的是 Token 的**初始向量**，还没有充分结合当前句子的上下文。

Transformer Block 会通过 Attention（注意力）等计算，让每个 Token 结合其他 Token 的信息，形成上下文表示：

```text
初始向量 → Attention 等上下文计算 → 上下文表示
```

### 6. 参数量

```text
Embedding 参数量 = vocab_size × d_model
```

Embedding 参数量只是模型总参数量的一部分。通常所说的“多少亿参数”还包括 Attention、FFN、输出层等可训练参数；如果输入 Embedding 和输出词表投影共享权重，这部分只统计一次。

## 术语速查

| 英文 / 缩写 | 中文 | 在流程中的位置 |
| --- | --- | --- |
| Tokenization | 分词 / 编码 | 文字 → Token |
| Tokenizer | 分词器 | 执行 Tokenization |
| Token | 文本单元 | Tokenization 的输出 |
| Vocabulary | 词表 | 保存 Token 与编号的对应关系 |
| Token ID（Token Identifier） | Token 编号 | Token → 数字编号 |
| BPE（Byte Pair Encoding） | 字节对编码 | 一种 Tokenization 方法 |
| Context Length | 上下文长度 | 限制 Token 数量 |
| Embedding | 嵌入 | Token ID → 向量 |
| Embedding Lookup Table | 嵌入查找表 | 保存 Token 向量 |
| `vocab_size`（Vocabulary Size） | 词表大小 | 查找表的行数 |
| `d_model`（model dimension） | 模型维度 | 每个向量的维度 |
| Attention | 注意力 | 结合上下文信息 |
| Transformer Block | Transformer 模块 | 生成上下文表示 |

## 复习问题

> 以下问题是根据本章概念整理的自测题，不是原文提供的面试题。

1. Tokenization、Token ID 和 Embedding 分别完成哪一步？
2. 为什么 Token ID 只是编号，而不是语义向量？
3. Context Length 约束的是哪一部分数据？
4. Embedding 查找表的形状 `[vocab_size, d_model]` 分别表示什么？
5. Embedding 参数量和模型总参数量有什么区别？

## 复习检查

- [ ] 能复述“文字 → Token → Token ID → 初始向量 → 上下文表示”。
- [ ] 能区分 Tokenization、Token ID 和 Embedding。
- [ ] 能解释 Context Length 的作用。
- [ ] 能说出 Embedding 表的形状及参数量公式。
- [ ] 能区分初始向量和上下文表示。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/03-Transformer 全景图|第 3 章：Transformer 全景图]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 下一章：Positional Encoding（位置编码）

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第四章原文：https://waylandz.com/llm-transformer-book/第04章-Tokenization-文字如何变成数字/
