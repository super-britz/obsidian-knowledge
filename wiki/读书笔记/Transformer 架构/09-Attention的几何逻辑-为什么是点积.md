---
type: 读书笔记
status: 学习中
created: 2026-08-11
updated: 2026-08-11
book: Transformer 架构：从直觉到实现
chapter: 9
topics: [Attention, 点积, Self-Attention, Cross-Attention, Mask, Softmax]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 9 章：Attention 的几何逻辑 - 为什么是点积

> [!abstract] 一句话理解
> **Attention 就像“带权重的搜索”：当前 Token 先找自己需要的信息，再按相关程度从其他 Token 取回信息。**

> [!tip] 学习方式
> 目前先不用记向量夹角、矩阵维度和方差推导。先记住：**找谁 → 分配多少注意力 → 取回什么信息**。

## 核心结论

- Attention 让一个 Token 可以直接读取其他允许访问的 Token。
- `QKᵀ` 用来计算“当前 Token 应该关注谁”。
- `Mask` 决定哪些位置不能读取，`Softmax` 把匹配分数变成注意力权重。
- 最后用这些权重混合 `V`，得到更新后的信息。
- Attention 权重表示信息如何流动，不等于模型完整的理解过程。

## 主流程：把 Attention 当成一次信息检索

```text
Token 表示
  ↓
提出需求 Q，准备标签 K 和内容 V
  ↓
Q 找 K：计算匹配分数
  ↓
屏蔽不能看的位置
  ↓
把分数变成注意力权重
  ↓
按权重取回并混合 V
  ↓
得到新的 Token 表示
```

在 GPT 中，当前位置只能读取自己和前面的 Token，不能读取未来 Token。

## 用“图书馆找书”理解 Q、K、V

把 Attention 想成你在图书馆找资料：

| Attention | 图书馆类比 | 作用 |
| --- | --- | --- |
| **Query（Q，查询）** | 你想找的内容 | 说明“我现在需要什么” |
| **Key（K，键）** | 每本书的标签和关键词 | 说明“这本书可能有什么” |
| **`QKᵀ`** | 需求和书籍标签的匹配程度 | 判断应该关注哪本书 |
| **Softmax** | 按匹配程度分配注意力比例 | 决定每本书贡献多少 |
| **Value（V，值）** | 每本书的实际内容 | 提供真正要取回的信息 |
| **输出** | 按比例混合多本书的内容 | 形成当前 Token 的新信息 |

所以可以这样记：

> **Q 决定想找什么，K 决定匹配谁，V 决定取回什么。**

这只是帮助理解信息流，不需要把它当成真实的搜索系统。

## 核心概念

### 1. 为什么需要 Attention

一个 Token 的含义，往往要结合其他 Token 才能确定。

可以把 Token 想成列表中的一条数据：当前数据需要查看其他相关数据，再把有用信息合并回来。

- **RNN（Recurrent Neural Network，循环神经网络）**：像接力传话，信息一级一级传递；
- **Attention**：像直接查询允许访问的其他数据，信息路径更短，也更容易并行处理。

因此，Attention 更适合处理相距较远的 Token 之间的关系。

### 2. `QKᵀ` 只负责“打分”

`QKᵀ` 的结果是匹配分数，不是概率：

```text
QKᵀ → 哪些位置更值得关注
```

矩阵乘法的好处是：可以一次计算很多 Query 和 Key 的匹配分数，而不是逐个计算。

### 3. 分数如何变成权重

原始分数不能直接当成百分比，所以要经过：

```text
匹配分数 → 缩放 → Mask → Softmax → 注意力权重
```

各步骤只需先记住作用：

- **Scale（缩放）**：防止某个分数大得过头，导致其他位置几乎没有机会；
- **Mask（掩码）**：禁止读取不允许的位置，例如 GPT 不能读取未来；
- **Softmax**：把分数转换成一组总和为 `1` 的权重。

### 4. 为什么最后要混合 `V`

得到权重后，Attention 不会只选一个 Token，而是按权重读取多个 Token 的内容：

- 相关程度高的 Token，贡献更多；
- 相关程度低的 Token，贡献更少；
- 混合后的结果，作为当前 Token 的新表示。

这就是 Attention 的“信息交换”。

### 5. Attention 热力图怎么看

热力图可以理解成“谁在看谁”：

- 行：正在发起查询的 Query；
- 列：被读取的 Key；
- 颜色越亮：这一行分配到的权重越高；
- GPT 中未来位置被 Mask 后，权重为 `0`。

它只能说明某一层、某一个 head 的信息路由，不能单独解释模型为什么得到最终答案。

### 6. Self-Attention 与 Cross-Attention

继续用“找资料”理解：

- **Self-Attention（自注意力）**：在同一份资料中查找。Q、K、V 来自同一个序列，GPT 使用这种方式；
- **Cross-Attention（交叉注意力）**：一份资料提出查询，另一份资料提供标签和内容。Encoder-Decoder 模型常用这种方式。

## 只需要记住的公式

```text
Attention(Q, K, V) = softmax(QKᵀ / √d_k + M) × V
```

把公式翻译成中文就是：

```text
匹配 → 稳定分数 → 屏蔽位置 → 变成权重 → 混合信息
```

现在不需要推导 `√d_k` 的数学来源。只要知道它是为了让分数不要过大，避免注意力权重过度集中即可。

## 术语速查

| 英文 / 缩写 | 中文 | 先记住什么 |
| --- | --- | --- |
| Attention | 注意力机制 | 按权重读取并混合信息 |
| Query（Q） | 查询 | 当前 Token 想找什么 |
| Key（K） | 键 | 用来匹配查询的标签 |
| Value（V） | 值 | 真正被取回的内容 |
| Dot Product | 点积 | 计算匹配分数 |
| Attention Weight | 注意力权重 | 决定每个位置贡献多少 |
| Scale | 缩放 | 控制分数大小 |
| Mask | 掩码 | 屏蔽不能读取的位置 |
| Softmax | 归一化函数 | 把分数变成总和为 1 的权重 |
| Self-Attention | 自注意力 | Q、K、V 来自同一序列 |
| Cross-Attention | 交叉注意力 | Q 与 K、V 来自不同序列 |

## 复习问题

> 以下问题是根据本章内容整理的自测题，不是原文提供的面试题。

1. Attention 想解决什么问题？
2. 用图书馆类比时，Q、K、V 分别是什么？
3. `QKᵀ`、Mask、Softmax、混合 `V` 分别做什么？
4. GPT 为什么不能读取未来 Token？
5. Self-Attention 和 Cross-Attention 有什么区别？
6. 为什么不能只看热力图判断模型的完整理解？

## 复习检查

- [ ] 能用“图书馆找书”说清 Q、K、V。
- [ ] 能复述 `QKᵀ → 缩放 → Mask → Softmax → 混合 V`。
- [ ] 能说明 Mask 为什么阻止 GPT 读取未来。
- [ ] 能区分 Self-Attention 和 Cross-Attention。
- [ ] 能说清注意力权重只是信息路由，不是完整解释。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/08-线性变换的几何意义-矩阵乘法的本质|第 8 章：线性变换的几何意义 - 矩阵乘法的本质]]
- 前置章节：[[wiki/读书笔记/Transformer 架构/06-LayerNorm与Softmax-数字的缩放与概率化|第 6 章：LayerNorm 与 Softmax - 数字的缩放与概率化]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 下一章：第 10 章：QKV 到底是什么（待整理）

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第九章原文：https://waylandz.com/llm-transformer-book/第09章-Attention的几何逻辑-为什么是点积/
