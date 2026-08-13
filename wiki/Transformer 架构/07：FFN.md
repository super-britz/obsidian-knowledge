---
type: 章节笔记
status: 已整理
chapter: 7
created: 2026-08-11
updated: 2026-08-13
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
---

# FFN：拿到上下文后，怎样加工当前 Token

> [!abstract] Attention 负责让 Token 从上下文取得信息；FFN 再分别加工每个 Token 当前的向量。它通过“扩宽 → 非线性选择 → 缩回”生成更新量，然后加回原表示。

> [!info] 放回完整生成链
> FFN 位于每个 Transformer Block 内。输入和输出都是一个 Token 的隐藏向量；它不直接读取其他位置，但输入已经经过 Attention，因此包含上下文。

## 作用是什么

可以先记成：

```text
Attention：从其他 Token 取得什么信息
FFN：拿到这些信息后，当前 Token 应怎样改变
```

例如当前 Token 是 `bank`，Attention 读到前文的 `river`。FFN 可以增强“河岸”相关特征、削弱“银行”相关特征。模型实际处理的是训练得到的向量，这只是帮助理解的例子。

## 怎么做

```text
Token 向量
→ 扩宽：产生更多候选特征
→ 选择和变化：让当前有用的特征通过
→ 缩回：组合成与原向量同维度的更新量
→ 通过残差连接加回原表示
```

模型先展开更多可能的特征，在当前语境下筛选有用部分，再把它们组合成对当前 Token 表示的修改。具体矩阵和维度不影响理解 FFN 的作用，可以在阅读模型代码时再学习。

## 关键理解

- **FFN 不是跨 Token 通信。** 位置之间的信息交流主要由 Attention 完成。
- **FFN 提供逐 Token 的特征加工。** 没有它，模型仍有 Attention，但会缺少这条重要的加工路径。
- **先扩宽再缩回是容量设计。** 更大的中间空间允许模型形成更多潜在特征，再组合回 residual stream。
- **FFN 往往占大量参数和计算。** 但不能因此说模型知识只存在于 FFN。

## 做项目时记住

1. 调用模型 API 时不需要实现 FFN，只需知道模型成本不只来自 Attention。
2. 本地选型时，FFN 的大小和类型会影响参数量、显存和速度。
3. [[30：Mixture of Experts|MoE]] 通常就是把一个 FFN 换成多个专家 FFN，再让 Router 为每个 Token 选择少数专家。

## 别误解

- **“FFN 不读取其他位置，所以不使用上下文。”** 它的输入已经包含 Attention 汇总的上下文。
- **“FFN 负责输出下一个 Token。”** FFN 只更新隐藏状态；最终词表打分由 LM Head 完成。
- **“Attention 是完整的 Transformer。”** FFN、残差、Norm 和输出层同样属于完整生成链。

## 来源与下一步

- **来源事实：** 第 7 章说明 FFN 由矩阵变换和非线性组成，并对每个 Token 位置分别执行。
- **机制推导：** FFN 可理解为“扩展潜在特征、按输入选择、再组合成更新量”。
- **下一步：** [[08：线性变换|线性变换]]、[[09-12：Attention|Attention]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
