---
type: source-note
title: PyTorch Module 状态、Autograd 与 Buffer
author: PyTorch
url: https://docs.pytorch.org/docs/main/notes/autograd.html
accessed: 2026-08-12
status: 已链接
---

# PyTorch Module 状态、Autograd 与 Buffer

这组官方文档用于核验 Transformer 最小实现中的训练 / 推理模式、梯度开关、Buffer 持久化和融合 Attention API 行为。

## 来源

- Autograd 与 Evaluation Mode：https://docs.pytorch.org/docs/main/notes/autograd.html
- `torch.nn.Module` 与 `register_buffer`：https://docs.pytorch.org/docs/stable/generated/torch.nn.Module.html
- `scaled_dot_product_attention`：https://docs.pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention

## 使用边界

- `model.eval()` 只切换依赖训练状态的模块行为，不是关闭梯度的机制；梯度记录由 `requires_grad`、`no_grad()` 或 `inference_mode()` 等机制控制。
- Buffer 默认会进入 `state_dict`；可由配置重建的固定 Mask 可以使用 `persistent=False`，是否持久化取决于模型恢复契约。
- `scaled_dot_product_attention` 的可用后端和参数行为可能随 PyTorch 版本变化；实现时应以目标版本官方文档和运行时验证为准。

## 本地解读

- [[wiki/Transformer 架构/16-17：训练、推理与参数更新|训练、推理与参数更新]]
- [[wiki/Transformer 架构/18-20：最小 Transformer 实现|最小 Transformer 实现]]
- [[wiki/Transformer 架构/21-22：Flash Attention 与 KV Cache|Flash Attention 与 KV Cache]]
