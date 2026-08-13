---
type: 主题笔记
status: 已整理
reading_mode: 深入机制
source_chapters: [18, 19, 20]
created: 2026-08-12
updated: 2026-08-12
sources:
  - "raw/links/2026-08-11-Transformer架构从直觉到实现.md"
  - "raw/links/2026-08-12-PyTorch-Module状态与Buffer.md"
---

# 第 18～20 章：用最小代码验证 Transformer

> [!abstract] 这三章的作用不是教你从零训练生产级大模型，而是把前面的概念装进一个能运行的小项目：`model.py` 负责算 logits，`train.py` 负责更新参数，`inference.py` 负责加载参数并循环生成。重点是看懂三者怎样通过 Tokenizer、配置、shape 和 checkpoint 连接起来。

> [!tip] 前端开发者可以跳过吗？
> 如果你只调用模型 API，可以先跳过本页；如果你想理解模型服务为什么会出现词表不匹配、加载失败、乱码或生成越来越慢，本页值得快速读一遍。教学代码用于验证机制，不等于生产训练框架。

## 项目闭环

```text
训练文本 → Tokenizer → x / y Batch
→ model.py 得到 logits
→ train.py 计算 Loss、更新参数
→ 保存 checkpoint

Prompt → 同一个 Tokenizer
→ inference.py 重建模型、加载 checkpoint
→ 循环预测 Token
→ decode 成文本
```

| 文件 | 主要输入 | 负责什么 | 主要输出 |
| --- | --- | --- | --- |
| `model.py` | Token ID | 定义 Transformer 和一次前向传播 | logits |
| `train.py` | 训练文本、目标 Token | 计算 Loss、反向传播、更新并保存 | checkpoint |
| `inference.py` | Prompt、checkpoint | 恢复同一个模型并循环解码 | 文本 |

## 四份共享契约

三份代码能配合，不是因为文件名正确，而是因为它们遵守同一组契约：

1. **Tokenizer：** 相同文本必须得到相同 Token ID，词表和特殊 Token 也要一致。
2. **模型配置：** `vocab_size`、`d_model`、层数、头数、位置方案等必须兼容。
3. **Shape：** 主干维度保持一致，最后才映射到词表。
4. **Checkpoint：** 参数名和 shape 必须匹配；恢复训练还要保存优化器、步数和随机状态。

```text
Token IDs                    [B, T]
隐藏状态                     [B, T, d_model]
Attention 分数               [B, heads, T, T]
LM Head logits               [B, T, vocab_size]
```

只保存 `state_dict` 不等于保存完整模型：它不包含 Python 类、Tokenizer 和全部配置。推理前必须先按兼容配置重建结构，再加载张量。

## `forward()` 和 `generate()` 的区别

```text
forward：当前序列 → 一次 logits 计算

generate：循环调用 forward
          → 取最后位置 logits
          → 选一个 Token
          → 拼回输入
```

`generate()` 是模型外层的控制流程，不是新的神经网络层。这个最小版本通常没有 KV Cache，所以每生成一个 Token 都会重复计算历史；它能帮助理解机制，但速度不能代表现代推理引擎。

## 最值得保留的三个测试

### 1. Shape 与边界测试

断言输出是 `[B, T, vocab_size]`，Loss 是有限值；同时测试序列超长、Token ID 越界以及 `d_model` 无法整除头数时能否尽早报错。

### 2. 单 Batch 过拟合

固定一个很小的 Batch 反复训练，Loss 应明显下降。否则优先检查目标是否错开一位、因果 Mask 方向、梯度和优化器是否有效。这是验证训练链路能否学习的快速实验，不用于评价泛化能力。

### 3. 保存—加载一致性

同一输入在保存前和重新加载后，logits 应在数值容差内一致。它能一次验证模型配置、参数名和加载流程。

## 教学代码不能直接拿去生产

最小实现通常省略：

- 高效的多头投影、Flash Attention 和 KV Cache；
- 混合精度、分布式训练、梯度累积与容错；
- 完整验证集、日志、最佳模型保存和异常监控；
- EOS、停止字符串、批处理和服务调度；
- 数据治理、权限、安全和模型评测。

因此，“代码能跑”只说明主链路连通，不说明模型训练正确、能泛化或适合部署。

## 做项目时记住

1. **应用层故障也可能来自模型契约。** 乱码、Token 越界和加载失败通常不是调 Temperature 能解决的。
2. **推理检查点和恢复训练检查点不同。** 前者保证可重建并生成；后者还要尽量恢复原优化轨迹。
3. **先测试再优化。** Shape、单 Batch 过拟合、保存—加载一致性通过后，再加入性能功能。

## 别误解

- **“会写最小 Transformer 就能训练大模型。”** 生产系统还需要数据、并行、稳定性、评测和大量计算资源。
- **“`state_dict` 就是完整模型。”** 它只是张量集合，必须与代码、配置和 Tokenizer 配套。
- **“训练 Loss 下降就说明效果好。”** 还需要独立验证集和任务评测。
- **“生成越来越慢是前端流式渲染导致的。”** 也可能是没有 KV Cache 而反复计算历史。

## 复习

1. 三份代码通过哪四类契约连接？
2. `forward()` 与 `generate()` 分别处在哪个层次？
3. 为什么保存—加载一致性测试很重要？
4. 为什么教学实现的性能不能代表生产推理服务？

## 来源与下一步

- **前置概念：** [[13-15：Transformer Block 与前向传播|Transformer Block 与前向传播]]、[[16-17：训练、推理与参数更新|训练、推理与参数更新]]。
- **来源事实：** 第 18～20 章分别实现模型、训练和推理代码；PyTorch 补充来源说明参数、Buffer、自动求导和运行模式的边界。
- **机制推导：** 三份文件通过 Tokenizer、配置、张量 shape 和 checkpoint schema 构成同一系统契约。
- **删减说明：** 省略逐模块代码、Buffer 持久化和训练恢复字段清单；需要实际实现时再查原始章节与框架文档。
- **下一主题：** [[21-22：Flash Attention 与 KV Cache|Flash Attention 与 KV Cache]]。
- **来源：** [[raw/links/2026-08-11-Transformer架构从直觉到实现|Transformer 架构：从直觉到实现]]、[[raw/links/2026-08-12-PyTorch-Module状态与Buffer|PyTorch Module 状态、Autograd 与 Buffer]]。
