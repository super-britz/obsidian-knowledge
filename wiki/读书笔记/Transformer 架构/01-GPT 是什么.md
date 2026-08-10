---
type: 读书笔记
status: 学习中
created: 2026-08-10
updated: 2026-08-10
book: Transformer 架构：从直觉到实现
chapter: 1
topics: [GPT, Transformer, Attention, RNN, LSTM, ChatGPT, Agent]
sources:
  - raw/links/2026-08-10-Transformer 架构：从直觉到实现.md
---

# 第 1 章：GPT 是什么

> [!abstract] 一句话理解
> Transformer 是底层架构，GPT 是基于 Transformer 的生成式预训练模型，ChatGPT 是围绕模型构建的对话产品，Agent 则在模型之外增加工具和执行系统。

## 核心结论

- Transformer 通过 Attention 建立序列中不同位置之间的关系，并支持较好的训练并行性。
- GPT（Generative Pre-trained Transformer）通过预测下一个 Token 学习语言规律并生成内容。
- ChatGPT 不等于 GPT 模型本身，还包含指令对齐、安全策略、上下文管理和产品交互。
- Agent 在模型之外增加工具、状态、工作流、权限和验证，使模型能够参与任务执行。

## 主流程：从序列模型到 Agent

```text
RNN / LSTM
  ↓
Transformer
  ↓
GPT
  ↓
ChatGPT
  ↓
Agent
```

这条路线不是简单的产品更名，而是能力边界逐步扩大：

```text
序列建模 → 生成式预训练 → 对话产品 → 工具与任务执行
```

## 核心概念

### 1. RNN、LSTM 与 Transformer

- **RNN（Recurrent Neural Network，循环神经网络）**：按顺序处理 Token，用隐藏状态传递历史信息。
- **LSTM（Long Short-Term Memory，长短期记忆网络）**：通过门控机制控制信息的保留和遗忘，改善 RNN 的长期依赖问题。
- **Transformer**：使用 Attention 直接建立不同位置之间的关系，训练时可以并行处理多个位置，更适合规模化训练。

核心区别：

```text
RNN / LSTM：依靠沿序列传递的隐藏状态
Transformer：通过 Attention 直接建立位置之间的联系
```

### 2. Attention（注意力机制）

Attention 让模型根据相关性对上下文信息进行加权汇总：

```text
当前 Token
  ↓ 查看上下文中的其他 Token
计算相关程度
  ↓
按权重汇总信息
  ↓
得到包含上下文的新表示
```

Attention 是一种计算机制，不等于人类意识中的注意，也不自动代表真正的语义理解。

当 Query、Key、Value 都来自同一个序列时，称为 **Self-Attention（自注意力）**。

### 3. Transformer、GPT、ChatGPT 与 Agent

| 名称 | 所在层次 | 作用 |
| --- | --- | --- |
| Transformer | 神经网络架构 | 提供序列建模和上下文计算结构 |
| GPT | 模型系列 | 基于 Transformer 进行生成式预训练 |
| ChatGPT | 对话产品 | 在模型上增加对齐、交互和产品能力 |
| Agent | 任务系统 | 在模型之外增加工具、状态、流程和验证 |

### 4. GPT 的三个组成部分

GPT 是 **Generative Pre-trained Transformer**：

- **Generative（生成式）**：生成新的 Token 序列；
- **Pre-trained（预训练）**：先在大量数据上学习语言规律；
- **Transformer**：采用 Transformer 架构。

### 5. 能力演进的共同因素

模型能力不只由参数规模决定，还受到以下因素共同影响：

```text
模型架构 + 数据 + 算力 + 预训练
        + 指令微调与偏好对齐
        + 产品和工具系统
```

## 术语速查

| 英文 / 缩写 | 中文 | 含义 |
| --- | --- | --- |
| RNN | 循环神经网络 | 按顺序传递隐藏状态的序列模型 |
| LSTM | 长短期记忆网络 | 带门控记忆机制的 RNN |
| Attention | 注意力 | 按相关性加权汇总上下文信息 |
| Self-Attention | 自注意力 | Q、K、V 来自同一序列的注意力 |
| Transformer | Transformer 架构 | 基于 Attention 的神经网络架构 |
| GPT | Generative Pre-trained Transformer | 生成式预训练 Transformer 模型系列 |
| ChatGPT | ChatGPT 对话产品 | 围绕 GPT 类模型构建的对话系统 |
| Agent | 智能体 / 任务系统 | 模型与工具、状态、流程、验证的组合 |

## 复习问题

> 以下问题是根据本章概念整理的自测题，不是原文提供的面试题。

1. Transformer、GPT、ChatGPT 和 Agent 分别处于什么层次？
2. RNN / LSTM 与 Transformer 处理序列的方式有什么不同？
3. Attention 的输入、计算和输出分别可以怎样理解？
4. GPT 这三个字母分别代表什么？
5. 为什么 GPT 的能力不能只归因于参数规模？

## 复习检查

- [ ] 能画出“RNN / LSTM → Transformer → GPT → ChatGPT → Agent”的演进关系。
- [ ] 能区分架构、模型、产品和任务系统。
- [ ] 能解释 Attention 如何汇总上下文信息。
- [ ] 能说出 GPT 的英文全称及三个组成部分。

## 关联

- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 下一章：[[wiki/读书笔记/Transformer 架构/02-大模型的本质|第 2 章：大模型的本质]]
- 已有课程：[[wiki/直播课/课程笔记/40-大语言模型原理|40 大语言模型原理]]
- Agent 延伸：[[wiki/直播课/课程笔记/43-Agent 与 Skills 编排|43 Agent 与 Skills 编排]]

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
- 第一章原文：https://waylandz.com/llm-transformer-book/第01章-GPT是什么-LLM发展简史与核心思想/
