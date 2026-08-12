---
type: source-note
title: Transformer 架构：从直觉到实现
author: Wayland Zhang
url: https://waylandz.com/llm-transformer-book/
accessed: 2026-08-11
updated: 2026-08-12
status: 已链接
---

# Transformer 架构：从直觉到实现

这是一套以直觉、可视化和最小实现为主线的 Transformer 学习资料。当前知识库以它作为 Transformer 专题笔记的外部事实来源之一。

## 来源

- 书籍首页：https://waylandz.com/llm-transformer-book/
- 第 1 章：GPT 是什么：https://waylandz.com/llm-transformer-book/%E7%AC%AC01%E7%AB%A0-GPT%E6%98%AF%E4%BB%80%E4%B9%88-LLM%E5%8F%91%E5%B1%95%E7%AE%80%E5%8F%B2%E4%B8%8E%E6%A0%B8%E5%BF%83%E6%80%9D%E6%83%B3/
- 第 2 章：大模型的本质：https://waylandz.com/llm-transformer-book/%E7%AC%AC02%E7%AB%A0-%E5%A4%A7%E6%A8%A1%E5%9E%8B%E7%9A%84%E6%9C%AC%E8%B4%A8-%E5%B0%B1%E6%98%AF%E4%B8%A4%E4%B8%AA%E6%96%87%E4%BB%B6/
- 第 3 章：Transformer 全景图：https://waylandz.com/llm-transformer-book/%E7%AC%AC03%E7%AB%A0-Transformer%E5%85%A8%E6%99%AF%E5%9B%BE/
- 第 4 章：Tokenization：https://waylandz.com/llm-transformer-book/第04章-Tokenization-文字如何变成数字/
- 第 5 章：Positional Encoding：https://waylandz.com/llm-transformer-book/第05章-Positional-Encoding-给文字加位置/
- 第 6 章：LayerNorm 与 Softmax：https://waylandz.com/llm-transformer-book/第06章-LayerNorm与Softmax-数字的缩放与概率化/
- 第 7 章：神经网络层：https://waylandz.com/llm-transformer-book/第07章-神经网络层-不需要懂也能理解Transformer/
- 第 8 章：线性变换的几何意义：https://waylandz.com/llm-transformer-book/第08章-线性变换的几何意义-矩阵乘法的本质/
- 第 9 章：Attention 的几何逻辑：https://waylandz.com/llm-transformer-book/第09章-Attention的几何逻辑-为什么是点积/
- 第 10 章：QKV 到底是什么：https://waylandz.com/llm-transformer-book/第10章-QKV到底是什么-Attention的三个主角/
- 第 11 章：Multi-Head Attention：https://waylandz.com/llm-transformer-book/第11章-Multi-Head-Attention-多视角理解/
- 第 12 章：QKV 输出的本质：https://waylandz.com/llm-transformer-book/第12章-QKV输出的本质/
- 第 13 章：残差连接与 Dropout：https://waylandz.com/llm-transformer-book/第13章-残差连接与Dropout-训练稳定的秘密/
- 第 14 章：词嵌入与位置信息：https://waylandz.com/llm-transformer-book/第14章-词嵌入与位置信息的深层逻辑-为什么相加而不是拼接/
- 第 15 章：完整前向传播：https://waylandz.com/llm-transformer-book/第15章-Transformer完整前向传播-从输入到输出/
- 第 16 章：训练与推理的异同：https://waylandz.com/llm-transformer-book/第16章-训练与推理的异同-为什么推理要一个字一个字生成/
- 第 17 章：学习率的理解：https://waylandz.com/llm-transformer-book/第17章-学习率的理解-训练稳定的关键/
- 第 18 章：手写 Model.py：https://waylandz.com/llm-transformer-book/第18章-手写Model.py-模型定义
- 第 19 章：手写 Train.py：https://waylandz.com/llm-transformer-book/第19章-手写Train.py-训练循环
- 第 20 章：手写 Inference.py：https://waylandz.com/llm-transformer-book/第20章-手写Inference.py-推理逻辑
- 第 21 章：Flash Attention：https://waylandz.com/llm-transformer-book/第21章-Flash-Attention-内存优化原理/
- 第 22 章：KV Cache：https://waylandz.com/llm-transformer-book/第22章-KV-Cache-推理加速/

## 使用限制

- 本页只记录来源和访问范围，不复制书籍正文。
- `wiki/` 中的页面只提炼可追溯的概念、机制、边界与工程决策；不以本页替代原书阅读。
- 书中涉及具体公司、模型、人物或产品的时效性信息，需要时以对应官方资料再次核验。

## 本地解读

- 专题学习入口：[[wiki/Transformer 架构/00：索引|Transformer 架构学习索引]]
- 已整理概念：[[wiki/Transformer 架构/01-03：LLM 与 Transformer|Transformer 总览]]
