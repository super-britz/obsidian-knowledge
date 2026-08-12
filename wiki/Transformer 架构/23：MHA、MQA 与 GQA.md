---
type: 章节笔记
status: 已整理
chapter: 23
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-MQA与GQA论文.md"
---

# MHA、MQA 与 GQA：用多少组 K/V 才合适

> [!abstract] MHA、MQA、GQA 保留多个 Query Head，但让不同数量的 Query Head 共享 K、V。共享越多，KV Cache 和 Decode 内存带宽越省；同时 K/V 表示容量越受约束。GQA 位于 MHA 与 MQA 之间，目标是在质量和推理效率之间取一个可验证的折中。

## 先看同一条结构轴

设 Query Head 数为 `Hq`，KV Head 数为 `Hkv`，要求 `Hq % Hkv == 0`：

| 结构 | `Hkv` | 每组共享 K/V 的 Query Head | 主要取舍 |
| --- | ---: | ---: | --- |
| MHA | `Hq` | 1 | K/V 表示容量最大，Cache 最大 |
| GQA | `1 < Hkv < Hq` | `Hq / Hkv` | 在容量与推理成本间折中 |
| MQA | `1` | `Hq` | Cache 最小，共享约束最强 |

例如 `Hq=8、Hkv=2` 时，Q0～Q3 共享 KV0，Q4～Q7 共享 KV1。各 Query Head 仍有独立的 Q 投影和注意力权重，并没有合并成一个 Head。

## 为什么减少 KV Head 能加速 Decode

每层 KV Cache 的近似容量为：

```text
bytes ≈ 2 × batch × tokens × layers × Hkv × head_dim × bytes_per_element
```

所以在其他维度相同时，Cache 大小与 `Hkv` 线性相关。Decode 每一步要读取不断增长的历史 K/V；减少 KV Head 不仅降低单请求显存，也减少内存带宽压力，可能容纳更长上下文或更高并发。

但它没有改变两个事实：新 Query 仍要关注历史位置，Dense Attention 的位置配对仍随上下文增长；端到端速度还受模型权重、内核、Batch、调度和硬件限制。因此 Cache 缩小 4 倍，不代表吞吐必然提高 4 倍。

## 共享 K/V 的约束与表现

- **能力约束：** MHA 允许每个 Query Head 使用独立 K/V 表示；MQA 只保留一组，可能损失质量；GQA 保留若干组，不能保证在所有模型上都无损。
- **实现约束：** 高效内核应直接把 Query Head 映射到所属 KV Head。为了教学而显式复制 K/V 会把张量扩回 `Hq`，可能抵消内存收益。
- **迁移约束：** 已训练的 MHA 检查点与 GQA 的 K/V 投影形状不同。GQA 论文先对组内 K/V Head 做平均池化，再继续训练；只改 `num_key_value_heads` 不是可靠转换。
- **可观察表现：** `Hkv` 减少后，KV Cache 实际占用和 Decode 带宽通常下降；质量、TTFT、TPOT 和吞吐变化则必须分别测量。

## 做项目时记住

1. **先读实际配置。** 比较 `num_attention_heads` 与 `num_key_value_heads`，不要只凭模型家族名称猜 MHA、GQA 或 MQA。
2. **把选择当成多目标实验。** 在目标模型、上下文长度、并发和硬件上，同时比较验证质量、TPOT、吞吐、峰值显存和 KV Cache 占用。
3. **验证实现语义。** 对同一模型比较 Cached / Uncached 的逐步 logits；检查 `Hq % Hkv == 0`、Head 映射、位置、Mask，以及实际是否命中支持 GQA 的高效内核。

## 别误解

- **“MQA 只有一个 Attention Head。”** 它仍有多个 Query Head，只是共享一组 K/V Head。
- **“GQA 的 KV Head 越少越好。”** 更少通常更省 Cache，但共享约束更强，质量与硬件并行效率不一定单调改善。
- **“把 MHA 配置中的 KV Head 数改小就完成转换。”** 权重形状和语义都已改变，需要明确的初始化、继续训练和重新评估。

## 复习

1. 为什么 MQA 保留多个 Query Head，却能显著缩小 KV Cache？
2. 如果 `Hq=32、Hkv=8`，每组有几个 Query Head，它们共享什么、不共享什么？
3. 为什么选择 GQA 不能只看 Cache 缩小比例，还要测质量和目标硬件？

## 来源与下一步

- **前置概念：** [[09-12：Attention|Attention]]、[[21-22：Flash Attention 与 KV Cache|Flash Attention 与 KV Cache]]。
- **来源事实：** 第 23 章将 MHA、GQA、MQA 统一为 `Hq / Hkv` 结构族；MQA 与 GQA 论文分别给出共享 K/V 的解码动机，以及从 MHA 检查点初始化并继续训练 GQA 的实验方案。
- **机制推导：** `Hkv` 同时控制 K/V 投影容量、KV Cache 大小和 Decode 读取量，因此它是质量、显存与带宽之间的架构旋钮。
- **工程建议：** 读取真实配置，在目标工作负载上同时验证 logits 语义、质量和性能；不要把论文单一实验或 Cache 比例当成部署保证。
- **下一主题：** [[24-25：长上下文|第 24～25 章：长上下文]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-MQA与GQA论文|MQA 与 GQA 原始论文]]。
