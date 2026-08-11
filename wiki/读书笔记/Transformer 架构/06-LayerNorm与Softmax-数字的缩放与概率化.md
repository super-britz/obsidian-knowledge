---
type: 读书笔记
status: 学习中
created: 2026-08-10
updated: 2026-08-11
book: Transformer 架构：从直觉到实现
chapter: 6
topics: [LayerNorm, RMSNorm, Softmax, logits, Temperature, Transformer]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 6 章：LayerNorm 与 Softmax - 数字的缩放与概率化

> [!abstract] 一句话理解
> LayerNorm / RMSNorm 负责把中间数字拉回稳定尺度；Softmax 负责把分数变成概率。

## 核心结论

- **LayerNorm / RMSNorm：稳定向量。**
- **Softmax：生成概率。**
- 训练时通常直接使用 `logits`，不需要手动 Softmax；推理采样时才需要把 logits 转成概率。

## 主流程：从向量到预测

```text
Token 向量
  ↓
Transformer Block
  ├─ LayerNorm / RMSNorm：稳定数字
  ├─ Attention / FFN：继续计算
  └─ Softmax：把 Attention 分数变成权重
  ↓
最终隐藏向量
  ↓ LM Head（语言模型头）
logits（候选 Token 的分数）
  ├─ 训练：Cross-Entropy（交叉熵）
  ├─ 贪心生成：argmax 取最高分
  └─ 随机采样：Softmax + Temperature → 概率
```

> LayerNorm 的具体位置会因架构不同而变化，但都在 Attention、FFN 附近发挥稳定作用。

## 核心概念

### 1. LayerNorm：稳定每个 Token 的向量

神经网络不断做乘法和加法，数字可能逐渐变得过大或过小。LayerNorm 对**每个 Token 的特征维度**单独计算，把它们拉回稳定尺度。

```text
计算均值和方差
  ↓
减去均值、除以标准差
  ↓
用 γ 和 β 做可学习调整
```

公式：

```text
y = (x - μ) / √(σ² + ε) × γ + β
```

- `μ`：均值；`σ²`：方差；`ε`：防止除以 0 的小常数；
- `γ`（gamma）和 `β`（beta）：允许模型重新放大、缩小或平移结果。

注意：均值约为 0、方差约为 1，只描述 `γ`、`β` 调整之前的标准化结果。

### 2. RMSNorm：LayerNorm 的近亲

RMSNorm 也用于稳定向量尺度，但通常不减去均值，而是使用均方根进行缩放，通常也没有 `β`。

只需要先记住：

```text
LayerNorm：中心化 + 缩放
RMSNorm：主要做缩放
```

### 3. Softmax：把分数变成概率

`logits` 是模型的原始分数，不要求为正，也不要求总和为 1。Softmax 将它们变成概率分布：

```text
P(i) = e^zᵢ / Σⱼ e^zⱼ
```

结果满足：

- 每项在 0 到 1 之间；
- 所有项相加等于 1；
- 分数越高，通常得到的概率越高。

Softmax 主要出现在两个地方：

- **Attention 内部**：注意力分数 → 注意力权重；
- **推理采样**：词表 logits → 下一个 Token 的概率。

训练时，交叉熵通常直接接收 logits；如果只是选择最高分，也可以直接 `argmax(logits)`，不必先 Softmax。

> 实现中常先计算 `Softmax(z - max(z))`，结果不变，但可以避免指数过大而溢出。

### 4. Temperature：控制采样的集中程度

```text
Softmax(logits / T)
```

- `T < 1`：概率更集中，输出更保守；
- `T = 1`：标准 Softmax；
- `T > 1`：概率更平滑，输出更多样。

Temperature 只改变推理时如何采样，不改变模型已经学到的内容。

### 5. 两者的区别

|  | LayerNorm / RMSNorm | Softmax |
| --- | --- | --- |
| 处理对象 | Token 向量 | 一组分数 |
| 作用 | 稳定数字尺度 | 转成概率分布 |
| 结果 | 不要求总和为 1 | 总和为 1 |
| 常见位置 | Attention、FFN 附近 | Attention 内部、推理采样 |

## 术语速查

| 英文 / 缩写 | 中文 | 先记住什么 |
| --- | --- | --- |
| Layer Normalization / LayerNorm | 层归一化 | 稳定 Token 向量 |
| Root Mean Square Normalization / RMSNorm | 均方根归一化 | 主要缩放向量 |
| Softmax | 概率化 | 把分数变成概率 |
| Logits | 未归一化分数 | 模型对候选 Token 的打分 |
| LM Head | 语言模型头 | 把隐藏向量映射成 logits |
| Temperature | 温度 | 控制采样的保守或多样 |
| Cross-Entropy | 交叉熵 | 训练时计算预测误差 |

## 复习问题

> 以下问题是根据本章内容整理的自测题，不是原文提供的面试题。

1. LayerNorm / RMSNorm 和 Softmax 分别做什么？
2. LayerNorm 为什么要使用 `γ` 和 `β`？
3. logits、Softmax 和概率是什么关系？
4. 训练、贪心生成、随机采样时，Softmax 分别如何使用？
5. Temperature 变小或变大有什么影响？

## 复习检查

- [ ] 能说清“LayerNorm 稳定数字，Softmax 生成概率”。
- [ ] 能复述 LayerNorm 的基本步骤和 `γ`、`β` 的作用。
- [ ] 能区分 logits、概率和注意力权重。
- [ ] 能解释训练、贪心生成和随机采样的区别。
- [ ] 能说出 Temperature 对采样的影响。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/05-Positional-Encoding-给文字加位置|第 5 章：Positional Encoding - 给文字加位置]]
- 前置全景：[[wiki/读书笔记/Transformer 架构/03-Transformer 全景图|第 3 章：Transformer 全景图]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 下一章：神经网络层（Feed Forward Network，前馈神经网络）

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第六章原文：https://waylandz.com/llm-transformer-book/第06章-LayerNorm与Softmax-数字的缩放与概率化/
