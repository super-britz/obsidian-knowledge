---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [21, 22]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-PyTorch-Module状态与Buffer.md"
---

# 第 21～22 章：Flash Attention 与 KV Cache

> [!abstract] 两者都让 Attention 更快，但消除的是不同浪费：Flash Attention 减少一次 Attention 内部的大型中间矩阵和显存搬运；KV Cache 在逐 Token 生成时保存历史 K/V，避免每轮重新计算整段前缀。前者省临时内存，后者用持久缓存换 Decode 速度。

> [!info] 放回用户请求链路
> - **长 Prompt 迟迟不出首字：** 主要发生在 Prefill，Flash Attention 可能改善计算和临时显存。
> - **首字出来后生成很慢：** 主要发生在 Decode，KV Cache 是基础优化。
> - **并发一高就显存不足：** 每个请求的 Cache 都在增长，需要 GQA、Cache 管理和调度。

## 先区分两个优化

| 对比 | Flash Attention | KV Cache |
| --- | --- | --- |
| 优化范围 | 一次 Attention 内部 | 多个 Decode 步骤之间 |
| 避免的浪费 | `T × T` 中间结果的存储和读写 | 历史 Token 的 K/V 重复计算 |
| 主要收益 | 降低临时显存和内存 IO | 降低每个新 Token 的计算量 |
| 主要代价 | 依赖硬件和融合内核 | 每个请求持续占用显存 |
| 是否可同时使用 | 可以 | 可以 |

不要因为名称都含 Attention，就把它们理解成竞争方案。

## Flash Attention 做了什么

普通 Attention 的概念流程是：

```text
Q @ Kᵀ → 完整分数矩阵 → Softmax → 完整权重矩阵 → @ V
```

序列长度为 `T` 时，中间矩阵有 `T × T` 个位置。朴素实现频繁把它写入显存再读回，数据搬运可能成为主要瓶颈。

Flash Attention 把 Q/K/V 分块放进更快的片上存储，用 Online Softmax 累积结果，最后直接写出输出。它仍然计算精确的 Dense Attention，只是改变计算和搬运顺序。

它没有做到：

- 没把 Dense Attention 的二次算术变成线性；
- 没减少模型参数；
- 没解决跨生成步骤的重复计算；
- 调用某个 API 也不保证一定命中 Flash 内核，仍要看硬件、dtype、shape 和框架。

## KV Cache 做了什么

生成第 `t` 个 Token 时，新 Query 需要读取之前所有 Token 的 K/V。模型参数和历史输入没有改变时，历史 K/V 也不会改变，因此可以保存并复用：

```text
Prefill：Prompt → 一次算出历史 K/V → 保存
Decode：新 Token → 只算新的 Q/K/V → K/V 追加到 Cache
         → 新 Q 读取全部历史 K/V → 生成下一个 Token
```

历史 Q 不需要缓存，因为过去位置的输出已经算完；当前步骤只需要新的 Query 去读取历史 K/V。

KV Cache 没让长上下文变成常数时间：新 Query 仍要读取越来越长的历史 K/V。它还会随上下文、层数、KV Head、并发和数据精度增长，最终可能成为服务并发的主要显存限制。

## Prefill 和 Decode 要分开测

```text
Prefill：处理整个 Prompt → 建立 Cache → 影响首 Token 延迟
Decode：每轮生成一个 Token → 读取 Cache → 影响后续生成速度
```

产品和服务至少分别记录：

- **TTFT：** 请求开始到第一个 Token；
- **TPOT / Token/s：** 首 Token 后的生成速度；
- **Throughput：** 整个服务单位时间处理量；
- **显存：** 权重、临时工作区和 KV Cache 各占多少。

前端流式渲染只能改善“用户何时看到内容”，不会降低模型 TTFT 或 TPOT。UI 仍应支持取消、超时、断线和不完整 Markdown。

## 做项目时记住

1. **先定位瓶颈。** 长 Prompt OOM 或 TTFT 高，检查 Prefill 与 Attention 内核；后续生成慢或并发低，检查 Cache 与内存带宽。
2. **Cache 是请求状态。** 历史 Token、位置或 Mask 改变后，受影响部分不能盲目复用。
3. **托管 API 不需要猜内部内核。** 记录真实延迟、吞吐、价格和失败率，用端到端结果选模型。
4. **自建服务必须做数值对照。** Flash 与普通实现、Cached 与 Uncached 的 logits 应在合理浮点容差内一致。

## 别误解

- **“Flash Attention 把复杂度从 `O(T²)` 降到 `O(T)`。”** 它主要减少中间存储和 IO，Dense Attention 的配对计算仍近似二次。
- **“KV Cache 后每个 Token 都是 `O(1)`。”** 新 Query 仍要读取全部历史 K/V。
- **“KV Cache 保存所有历史激活。”** 通常只保存每层的历史 K/V。
- **“两项都开就不会 OOM。”** Flash 减少临时内存，KV Cache 却随请求和上下文增长。

## 复习

1. Flash Attention 和 KV Cache 分别消除什么浪费？
2. 为什么 Flash Attention 省临时内存，却没有消除 Dense Attention 的二次计算？
3. 为什么 KV Cache 能加速 Decode，同时又限制并发？
4. 前端怎样区分模型首 Token 慢和后续生成慢？

## 来源与下一步

- **前置概念：** [[09-12：Attention|Attention]]、[[16-17：训练、推理与参数更新|训练与推理]]。
- **来源事实：** 第 21 章介绍 Tiling、Online Softmax 和 Flash Attention 的 IO 优化；第 22 章介绍自回归重复计算、KV Cache 及 Prefill / Decode。
- **机制推导：** 两者分别优化单次 Attention 和多次 Decode，资源方向并不相同。
- **删减说明：** 省略 Online Softmax 状态推导、完整 Cache 显存公式和基准测试清单；自建推理引擎时再查原始章节。
- **下一主题：** [[23：MHA、MQA 与 GQA|MHA、MQA 与 GQA]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-PyTorch-Module状态与Buffer|PyTorch Module 状态、Autograd 与 Buffer]]。
