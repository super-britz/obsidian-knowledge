---
type: 读书笔记
status: 学习中
created: 2026-08-11
updated: 2026-08-11
book: Transformer 架构：从直觉到实现
chapter: 7
topics: [神经网络, FFN, 激活函数, Transformer]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 7 章：神经网络层 - 不需要懂也能理解 Transformer

> [!abstract] 一句话理解
> **Attention 负责让 Token 互相参考，FFN 负责把每个 Token 单独加工。**

## 核心结论

- Token 进入 Transformer 后，已经是一串数字。
- **Attention**：让一个 Token 参考其他 Token。
- **FFN**：只处理当前 Token，不直接看其他 Token。
- FFN 的形状变化是：`d_model → d_ff → d_model`。
- Attention 和 FFN 反复执行，最后由 LM Head 预测下一个 Token。

## 主流程：Transformer Block 如何加工 Token

```text
Token 向量
  ↓
Attention：互相参考
  ↓
FFN：各自加工
  ↓
重复多个 Transformer Block
  ↓
LM Head：预测下一个 Token
```

实际 Block 中还会配合 Norm（归一化）和残差连接，帮助计算稳定。第七章真正要理解的是：

> **Attention 改善 Token 之间的关系，FFN 改善每个 Token 自己的表示。**

## 核心概念

### 1. 神经网络是什么

先把神经网络理解成一个“数字加工器”：

```text
输入数字 → 带参数的变换 → 输出数字
```

参数是在训练中学到的数字。神经网络不需要手写每条规则，而是通过训练学会如何变换输入。

### 2. FFN 做什么

FFN（Feed Forward Network，前馈网络）是 Transformer 中的神经网络层。

它对每个 Token 单独处理：

```text
一个 Token 的向量
  ↓ 变宽：d_model → d_ff
  ↓ 加工
  ↓ 变回：d_ff → d_model
```

因此：

- Token 的数量不变；
- 每个 Token 内部的数字会改变；
- 输出仍然可以接回下一层 Transformer。

### 3. 为什么中间要变宽

中间变宽，是为了给模型更多空间组合信息；最后变回 `d_model`，是为了保持统一的输入、输出宽度。

只记住：

```text
向量暂时变宽 → 处理更多信息 → 恢复原宽度
```

`d_ff` 不一定等于 `4 × d_model`，具体由模型结构决定。

### 4. Attention 和 FFN 的分工

| 组件 | 作用 | 处理范围 |
| --- | --- | --- |
| Attention（注意力机制） | 让 Token 互相参考 | 多个 Token 之间 |
| FFN（前馈网络） | 重新加工 Token 表示 | 每个 Token 自己 |

这是本章最重要的区别：

> **Attention 看别人，FFN 改自己。**

### 5. 激活函数是什么

激活函数（Activation Function）让网络具备处理复杂模式的能力。

如果只有矩阵乘法，多层变换仍然比较简单；加入激活函数后，模型才能学习非线性关系。

常见名称：

- ReLU（Rectified Linear Unit，修正线性单元）；
- GELU（Gaussian Error Linear Unit，高斯误差线性单元）；
- SiLU（Sigmoid Linear Unit，Sigmoid 线性单元）；
- SwiGLU（Swish Gated Linear Unit，带门控的线性单元）。

当前阶段只需要知道它们都是激活或门控方式，不必背公式。

### 6. 参数主要在哪里

参数就是训练中学到的数字，主要分布在：

| 组件 | 参数作用 |
| --- | --- |
| Embedding（嵌入） | 保存 Token 的初始向量 |
| Attention | 学习 Q、K、V、O 的投影 |
| FFN | 学习 Token 的非线性变换 |
| Norm | 调整数字的尺度 |
| LM Head | 把隐藏向量映射回词表 |

本章给出的典型配置中，FFN 每层参数量约为 Attention 的 2 倍。这个结论只用来建立直觉，不需要背具体模型参数。

> [!note]- 进阶：神经网络的公式
> 可以写成：`输出 = 激活函数(输入 × 权重 + 偏置)`。现在只要知道“输入经过带参数的变换”即可。

## 术语速查

| 英文 / 缩写 | 中文 | 先记住什么 |
| --- | --- | --- |
| Neural Network | 神经网络 | 数字加工器 |
| Attention | 注意力机制 | 让 Token 互相参考 |
| FFN（Feed Forward Network） | 前馈网络 | 逐个 Token 加工 |
| `d_model`（model dimension） | 模型维度 | Token 向量的正常宽度 |
| `d_ff`（feed-forward dimension） | 前馈层维度 | FFN 中间的宽度 |
| Activation Function | 激活函数 | 增加非线性 |
| LM Head（Language Model Head） | 语言模型头 | 预测下一个 Token |

## 复习问题

> 以下问题是根据本章内容整理的自测题，不是原文提供的面试题。

1. Attention 和 FFN 分别负责什么？
2. FFN 为什么要先变宽，再变回原来的宽度？
3. FFN 改变的是 Token 数量，还是 Token 向量中的数字？
4. LM Head 在流程最后负责什么？

## 复习检查

- [ ] 能说清“Attention 看别人，FFN 改自己”。
- [ ] 能复述 `d_model → d_ff → d_model` 的含义。
- [ ] 能说出 FFN 不改变 Token 数量。
- [ ] 能区分 FFN 和最后的 LM Head。

## 关联

- 上一章：[[wiki/读书笔记/Transformer 架构/06-LayerNorm与Softmax-数字的缩放与概率化|第 6 章：LayerNorm 与 Softmax - 数字的缩放与概率化]]
- 前置全景：[[wiki/读书笔记/Transformer 架构/03-Transformer 全景图|第 3 章：Transformer 全景图]]
- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 下一章：[[wiki/读书笔记/Transformer 架构/08-线性变换的几何意义-矩阵乘法的本质|第 8 章：线性变换的几何意义 - 矩阵乘法的本质]]

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第七章原文：https://waylandz.com/llm-transformer-book/第07章-神经网络层-不需要懂也能理解Transformer/
