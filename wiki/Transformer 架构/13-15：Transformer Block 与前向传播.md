---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [13, 14, 15]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
---

# 第 13～15 章：一个 Token 怎样穿过 Transformer

> [!abstract] Transformer Block 不是每层都把 Token 表示推倒重来，而是在同一条信息主干上不断追加修正：Attention 补充“应该从前文取回什么”，FFN 补充“这些信息还要怎样加工”。多层更新完成后，LM Head 才把内部表示变成词表中每个候选 Token 的分数。

> [!info] 放回完整生成链
> - **位置：** Tokenization 和 Embedding 之后，选择下一个 Token 之前；它是语言模型的主体。
> - **输入 → 输出：** Token ID → 每个位置对整个词表的 logits。
> - **前端类比：** 可以把隐藏状态暂时看成贯穿中间件链的 `state`，每个 Block 读取并追加更新；这只是数据流类比，不表示模型在执行手写业务规则。

## 先看完整流程

本页只讨论 GPT 一类 **Decoder-Only 因果语言模型**：当前位置能读取自己和前文，不能读取未来 Token。

```text
Token ID
→ Embedding：编号变成向量
→ 注入位置信息
→ Block 1：Attention + FFN
→ Block 2：Attention + FFN
→ ……
→ Final Norm
→ LM Head
→ 词表 logits
```

一次前向传播只计算 logits。训练会拿 logits 计算 Loss 并更新参数；推理会从最后位置的 logits 选出下一个 Token。前向、训练和生成不是同一件事。

## 一个 Block 到底做什么

最值得记住的心智模型是：

```text
读取当前表示
→ 计算一份更新量
→ 加回原表示
```

常见的 Pre-Norm Block 可以简化为：

```text
X' = X  + Attention(Norm(X))
Y  = X' + FFN(Norm(X'))
```

| 组件 | 解决的问题 | 写回的信息 |
| --- | --- | --- |
| Attention | 当前 Token 需要参考哪些前文 | 从其他位置汇总的上下文 |
| FFN | 汇总后的信息怎样继续加工 | 当前 Token 的非线性变换结果 |
| Residual | 深层网络怎样保留已有信息 | 将更新量叠加到原表示 |
| Norm | 多层连续计算怎样保持稳定 | 调整子层读取到的数值尺度 |

多个 Block 结构相似，但标准模型中通常各有一套参数。层数增加，表示会经历更多轮上下文读取和加工，并不是反复运行完全相同的规则。

## 为什么残差连接很重要

把隐藏状态想成一块持续传递的共享白板：

- Attention 不必重写白板，只写入它从前文找到的补充信息；
- FFN 再写入加工后的补充信息；
- 原来的内容沿 `X` 这条路径继续保留；
- 训练时，梯度也多了一条直接穿过加法的路径。

因此残差连接既帮助保留前向信息，也降低深层网络的训练难度。它不保证信息永远不丢失，只是给信息和梯度提供更直接的通道。

## 最后怎样变成下一个 Token

Block 一直维护固定宽度的隐藏状态，LM Head 才把它投影到词表大小：

```text
隐藏状态 [B, T, d_model]
→ LM Head
→ logits [B, T, vocab_size]
```

生成时通常只使用最后一个有效位置的 logits，再通过 Greedy、Temperature、Top-K 或 Top-P 选出下一个 Token。训练时，交叉熵通常直接接收 logits，不需要手动先做 Softmax。

读代码时只需先追两条 shape 规则：

1. Attention 和 FFN 内部可以拆头或扩宽，但写回残差主干前必须恢复成 `d_model`。
2. LM Head 才把最后一维从 `d_model` 变成 `vocab_size`。

## 哪些细节先不用背

- **Pre-Norm / Post-Norm：** 区别是 Norm 放在残差加法前还是后；现代模型常见 Pre-Norm 或 RMSNorm 变体，查具体模型配置即可。
- **Dropout：** 训练时随机扰动激活，推理时关闭；它不会永久删除参数，而且有些现代 LLM 将其设为零。
- **位置注入：** 绝对位置向量可以在入口相加，RoPE 通常在 Attention 内旋转 Q、K；不能把所有方案都理解成 `Embedding + Position`。
- **Weight tying：** LM Head 与 Token Embedding 可以共享权重，但不是理解前向主线的前提。

这些细节影响架构复现和训练，不影响第一次理解“隐藏状态经过多层增量更新后变成词表分数”。

## 做项目时记住

1. **应用开发者通常不需要实现 Block。** 需要理解的是模型输出仍是概率分数，事实、权限和动作结果要由外部系统验证。
2. **阅读模型代码先追主干。** 找到 `[B, T, d_model]` 在哪里被读取、更新和写回，再看多头拆分与 FFN 扩宽。
3. **不要混淆三个边界。** `forward` 计算 logits；训练还需要 Loss、反向传播和优化器；生成还需要解码循环。

## 别误解

- **“Attention 就是完整的 Transformer。”** Block 还依赖 FFN、残差和 Norm，多层之后还需要 LM Head。
- **“每层都会替换掉上一层的信息。”** 子层主要计算增量，再通过残差加回当前表示。
- **“N 层就是同一个 Block 循环 N 次。”** 通常只是结构重复，每层参数独立。
- **“LM Head 输出的最高分就是事实。”** logits 表示模型在当前上下文中的偏好，不是真实世界的验证结果。

## 复习

1. Attention、FFN、Residual 和 Norm 分别解决什么问题？
2. 为什么一个 Block 可以理解成“计算更新量并加回主干”？
3. 为什么 LM Head 之前保持 `d_model`，之后变成 `vocab_size`？
4. 如果接口返回了一个高概率工具参数，应用为什么仍要校验？

## 来源与下一步

- **来源事实：** 第 13 章介绍残差、Norm 与 Dropout；第 14 章介绍位置表示怎样进入模型；第 15 章串联 Decoder-Only 输入、Block 堆叠、Final Norm 和 LM Head。
- **机制推导：** Block 可以统一理解为“读取 residual stream、计算增量、写回 residual stream”，LM Head 是隐藏空间到词表空间的边界。
- **删减说明：** 省略残差梯度公式、Dropout 数学推导和模型配置举例；需要复现架构时再查原始章节。
- **下一主题：** [[16-17：训练、推理与参数更新|训练、推理与参数更新]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
