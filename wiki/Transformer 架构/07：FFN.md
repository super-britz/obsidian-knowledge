---
type: 章节笔记
status: 已整理
chapter: 7
created: 2026-08-11
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
---

# FFN（Feed-Forward Network，前馈网络）：模型怎样加工每个 Token 的信息

> [!abstract] Attention 负责让一个 Token 从其他位置拿到相关信息；前馈网络（FFN）负责把拿到的信息在“当前 Token 自己的向量里”继续加工。它通常先把向量扩宽、经过非线性变化、再缩回原来的大小。

> [!info] 放回完整生成链
> - **位置：** 每个 Transformer Block 内，通常位于 Attention 子层之后。
> - **输入 → 输出：** 每个 Token 的隐藏向量 → 相同维度的更新量。
> - **为什么需要：** Attention 主要负责跨 Token 汇总信息；FFN 再对汇总结果做非线性加工。缺少 FFN，模型只有信息搬运，表达能力会明显受限。

## 先看流程

```text
某个 Token 的当前向量
→ 线性层：扩宽到更大的中间空间
→ 激活函数：加入非线性变化
→ 线性层：缩回 d_model
→ 得到加工后的该 Token 向量
```

在一个 Transformer Block 中，可以先记成：

```text
Attention：不同 Token 之间交换、汇总信息
→ FFN：每个 Token 分别加工自己的信息
```

## 关键理解

- **神经网络的核心是“带参数的矩阵变换 + 非线性”。** 参数通过训练调整，让网络从数据中学出有用模式，而不是由人手写所有规则。
- **FFN 对每个位置使用同一套规则，但不直接让位置之间交流。** Token 之间的信息交流主要由 Attention 完成；FFN 是逐 Token 的处理器。
- **先扩宽再缩回是常见做法。** `d_model → d_ff → d_model` 给网络一个更大的中间空间做变化；具体扩宽倍数因模型而异。
- **FFN 往往占大量参数和计算。** 在许多稠密 Transformer 中，它的参数量可能超过 Attention；但不能因此简单认为“知识只在 FFN 里”，模型行为来自多个组件共同作用。

## 做项目时记住

1. **读模型配置时，不只看 Attention。** `d_model`、`intermediate_size / d_ff`、激活函数和 FFN 类型会明显影响参数量、显存和速度。
2. **理解职责分工。** Attention 更像“跨位置找资料”，FFN 更像“拿到资料后在当前 Token 内加工”；这有助于理解后续 Block、MoE 和推理性能。
3. **做应用通常不需要手写 FFN。** 更重要的是理解它为什么存在、为什么模型大小和推理成本不只由 Attention 决定。

## 别误解

- **“神经网络就是模仿人脑。”** 它受生物神经元启发，但本质是数学模型，不应把它当作大脑的直接复制品。
- **“FFN 就是模型输出下一个 Token 的地方。”** FFN 只是 Block 内部组件；最终词表打分由 LM Head 和解码步骤完成。
- **“Attention 是 Transformer 唯一重要的部分。”** Attention 很关键，但 FFN、Embedding、Norm、残差和输出层共同决定模型能力与成本。

## 复习

1. Attention 和 FFN 分别负责什么，为什么两者需要配合？
2. 为什么 FFN 常先扩宽向量、再缩回原始维度？
3. 为什么评估本地模型资源时，不能只关注 Attention 的结构？

## 来源与下一步

- **来源事实：** 第 7 章将神经网络概括为矩阵变换与非线性的组合，并说明 Transformer Block 中的 FFN 会对每个 Token 表示进行逐位置变换；章节还比较了常见 FFN 结构与参数分布。
- **本页推论：** 理解“Attention 负责跨位置路由，FFN 负责逐位置加工”足以支撑当前的 Transformer 与应用工程学习；更细的矩阵和激活函数推导可按需要再学。
- **下一步：** [[08：线性变换|线性变换]]、[[09-12：Attention|Attention]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
