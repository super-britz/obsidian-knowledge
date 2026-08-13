---
type: 章节笔记
status: 已整理
chapter: 23
created: 2026-08-12
updated: 2026-08-13
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-MQA与GQA论文.md"
---

# MHA、MQA 与 GQA：怎样减少 KV Cache

> [!abstract] 三者都是多头 Attention，区别在于多个 Query Head 是否共享 K 和 V。共享越多，需要保存和读取的 KV Cache 越少，但模型可用的 K/V 表示也更受限制。

## 为什么会出现三种结构

多头 Attention 会从多个角度读取上下文。传统 MHA 为每个 Query Head 准备独立的 K 和 V；长对话生成时，这些 K/V 都要进入 KV Cache，显存和读取成本很高。

于是出现了两种共享方案：

```text
MHA：每个 Query Head 使用自己的 K/V
GQA：一组 Query Head 共享一组 K/V
MQA：所有 Query Head 共享一组 K/V
```

Query Head 仍然有多个。变化的只是它们共享多少组 K/V。

## 各自的取舍

- **MHA：** K/V 表示最丰富，但 KV Cache 最大。
- **MQA：** KV Cache 最小，生成时读取的数据更少，但共享约束最强。
- **GQA：** 保留多组 K/V，在模型能力和推理成本之间折中。

减少 K/V 主要改善长上下文生成的显存和带宽压力，不会让 Attention 不再读取历史，也不保证速度按相同比例提升。

## 为什么应用开发者通常不用选择

MHA、MQA 和 GQA 是模型训练时确定的内部结构。调用托管模型 API 时，不能通过 Prompt 改变它，只需比较：

- 回答质量；
- 第一个 Token 延迟；
- 后续生成速度；
- 长对话稳定性；
- 价格和并发限制。

只有在选择或部署本地模型时，才需要查看模型配置和推理框架是否正确支持其 KV Head 结构。

## 别误解

- **“MQA 只有一个 Attention Head。”** 它仍有多个 Query Head，只共享 K/V。
- **“共享越多一定越好。”** Cache 更小，但表示能力可能受到影响。
- **“把配置数字改小就能把 MHA 变成 GQA。”** 已训练权重的结构不同，不能只改配置。
- **“GQA 解决了所有长上下文问题。”** 它只减少 KV Cache，不保证远处信息能被可靠使用。

## 一句话复习

> MHA 不共享 K/V，MQA 全部共享，GQA 分组共享；共享越多越省 KV Cache，但需要重新权衡模型质量。

## 来源与下一步

- **来源事实：** 第 23 章及相关论文介绍 MHA、MQA、GQA 的 K/V 共享方式与推理动机。
- **机制推导：** KV 共享是模型能力、显存和生成带宽之间的架构取舍。
- **下一步：** [[24-25：长上下文|长上下文总览]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-MQA与GQA论文|MQA 与 GQA 原始论文]]。
