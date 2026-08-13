---
type: 主题笔记
status: 已整理
source_chapters: [21, 22]
created: 2026-08-12
updated: 2026-08-13
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-PyTorch-Module状态与Buffer.md"
---

# Flash Attention 与 KV Cache：为什么能让生成更快

> [!abstract] 两者都在优化 Attention，但解决不同浪费。Flash Attention 减少一次 Attention 计算中的内存搬运；KV Cache 保存已经算过的历史信息，避免生成每个新 Token 时反复重算前文。

## 先区分两种慢

```text
处理整段 Prompt
→ 等待第一个 Token
→ 继续逐 Token 生成
```

- **第一个 Token 等很久：** 模型要先处理整段 Prompt。Flash Attention 主要优化这一阶段的 Attention 计算。
- **后面的 Token 一个个生成很慢：** 模型每次都要读取历史。KV Cache 主要减少这里的重复计算。

两者可以同时使用，并不是互相替代的方案。

## Flash Attention 做了什么

普通 Attention 会产生很大的中间结果，并在 GPU 内存中反复读写。序列越长，中间数据越大，搬运数据可能比计算本身更浪费时间。

Flash Attention 会把计算拆成较小的块，在更快的片上存储中完成更多步骤，减少大型中间结果和显存读写。

它改变的是计算方式，不是 Attention 的含义。模型仍然在比较 Token 之间的关系，也没有因此获得更多知识。

## KV Cache 做了什么

生成下一个 Token 时，前文没有改变，所以历史 Token 的 K 和 V 也不需要重复计算。

```text
首次处理 Prompt
→ 算出历史 K/V 并保存

生成新 Token
→ 只计算新 Token 的信息
→ 读取已保存的历史 K/V
→ 把新的 K/V 追加到缓存
```

这就是 KV Cache：用内存保存历史计算结果，换取后续生成速度。

## 代价是什么

- Flash Attention 依赖框架、硬件和高效实现，不是写一个参数就一定生效。
- KV Cache 会随着对话长度和并发请求增加，占用越来越多显存。
- Cache 只能复用没有变化的历史；Prompt、位置或 Mask 改变后，相关部分需要重算。
- 即使使用 Cache，新 Token 仍要读取越来越长的历史，所以长对话不会变成固定成本。

## 做项目时记住

1. 分开测“多久看到第一个 Token”和“之后生成得多快”。
2. 长 Prompt 慢时先减少无关上下文；技术优化不能替代内容裁剪。
3. 调用托管 API 时不必猜内部内核，直接比较真实延迟、费用和回答质量。
4. 自建服务需要同时关注模型权重、临时内存和随请求增长的 KV Cache。

## 别误解

- **“Flash Attention 会减少模型参数。”** 它优化 Attention 的执行方式，不改变参数量。
- **“KV Cache 会让每个新 Token 都是固定成本。”** 历史越长，需要读取的缓存仍越多。
- **“KV Cache 越多越好。”** 它提高速度，也会限制可同时服务的请求数。
- **“页面流式显示能让模型算得更快。”** 流式只让用户更早看到已经生成的内容。

## 一句话复习

> Flash Attention 减少一次计算中的内存搬运；KV Cache 保存历史 K/V，减少多次生成之间的重复计算。

## 来源与下一步

- **来源事实：** 第 21 章介绍 Flash Attention 的分块与内存优化；第 22 章介绍自回归生成中的重复计算和 KV Cache。
- **机制推导：** 两者分别优化单次 Attention 和连续生成，资源取舍不同。
- **下一步：** [[23：MHA、MQA 与 GQA|MHA、MQA 与 GQA]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]。
