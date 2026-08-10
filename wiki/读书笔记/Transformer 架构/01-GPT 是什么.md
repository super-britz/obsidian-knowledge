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

> [!abstract] 核心问题
> GPT 如何发展而来，Transformer 为什么会成为大语言模型的基础？

## 本章结论

- Transformer 提供了适合并行训练和规模化扩展的序列建模架构。
- GPT 将 Transformer 用于生成式预训练语言模型，通过预测下一个 Token 学习语言规律。
- ChatGPT 不只是一个 GPT 模型，还包含指令对齐、安全策略、上下文管理和产品交互等系统能力。
- AI Agent 在模型之外增加工具、状态、工作流和验证，使 AI 从生成回答进一步走向执行任务。

## 知识框架

### 1. 关键演进

```text
RNN / LSTM
  -> 2017 Transformer
  -> 2018 GPT-1
  -> 大规模预训练
  -> InstructGPT / ChatGPT
  -> 多模态、推理模型与 Agent
```

这条路线的重点不是背诵所有年份和人物，而是理解模型能力如何随架构、数据、算力、训练方式和产品系统共同演进。

### 2. RNN、LSTM 与 Transformer

**RNN**

- 按顺序处理 Token。
- 每一步将之前的信息压缩为隐藏状态，再传给下一步。
- 序列较长时，早期信息可能逐渐丢失。

**LSTM**

- 是带有门控记忆机制的特殊 RNN。
- 通过遗忘门、输入门和输出门控制信息的保留与使用。
- 改善了长期依赖问题，但仍然需要按顺序计算。

**Transformer**

- 使用 Attention 直接建立不同位置之间的关系。
- 训练时可以并行处理序列中的多个位置。
- 更容易扩展到大量数据、参数和计算资源。

```text
RNN / LSTM：把过去压缩到一份不断传递的状态中
Transformer：通过 Attention 直接寻找上下文中的相关位置
```

### 3. Attention 的直觉

Attention（注意力机制）让模型在处理一个 Token 时，判断上下文中其他 Token 与它有多相关，并按照相关程度汇总信息。

例如：

```text
小明把书递给小红，因为她需要复习。
```

模型处理“她”时，需要更多关注“小红”，而不是平均对待句子中的所有词。Attention 会为不同位置计算相关性权重，再把重要位置的信息更多地组合到当前表示中。

可以暂时把它理解为：

```text
当前 Token
  -> 查看上下文中的其他 Token
  -> 计算相关程度
  -> 将权重转换为比例
  -> 按比例汇总相关信息
  -> 得到包含上下文的新表示
```

Attention 不是人类意识中的“注意”，也不代表模型真正理解了句子；它是一种根据相关性加权组合信息的计算机制。

**与 RNN / LSTM 的区别**

```text
RNN / LSTM：信息需要沿序列一步步传递
Attention：相关位置之间可以直接建立联系
```

当 Query、Key、Value 都来自同一个序列时，这种机制称为 Self-Attention。具体如何计算相关性，将在后续章节继续学习。

### 4. Transformer、GPT、ChatGPT 与 Agent

| 层次 | 含义 |
| --- | --- |
| Transformer | 一种神经网络架构 |
| GPT | 通常采用 Decoder-only Transformer 的生成式预训练模型系列 |
| ChatGPT | 围绕 GPT 类模型构建的对话产品 |
| Agent | 模型与工具、状态、工作流、权限和验证组成的任务系统 |

### 5. GPT 的含义

GPT 是 **Generative Pre-trained Transformer**：

- **Generative**：生成新的内容。
- **Pre-trained**：先在大量数据上进行预训练。
- **Transformer**：使用 Transformer 架构。

### 6. 能力演进的主要因素

```text
模型架构
  + 数据
  + 算力
  + 规模化预训练
  + 指令微调与偏好对齐
  + 产品和工具系统
```

GPT 的发展不能简单归因于模型参数变多，以上因素共同影响最终能力。

## 面试问题与答案

### 1. GPT 和 Transformer 有什么区别？

Transformer 是一种通用神经网络架构；GPT 是基于 Transformer 构建的生成式预训练模型系列，通常采用 Decoder-only 架构并通过预测下一个 Token 生成内容。

### 2. Transformer 相比 RNN 和 LSTM 有什么优势？

- Transformer 在训练时可以并行处理序列中的多个位置，而 RNN 和 LSTM 依赖前一步的状态。
- Attention 让不同位置能够直接建立联系，更容易建模长距离关系。
- Transformer 更适合利用大量数据和计算资源进行规模化训练。

需要注意：自回归语言模型在推理生成时通常仍需逐个生成 Token。

### 3. ChatGPT 和 GPT 是一回事吗？

不是。GPT 是底层模型系列，ChatGPT 是基于模型构建的对话产品，还包含指令对齐、安全策略、上下文管理和交互系统等能力。

### 4. 为什么 Transformer 适合训练大模型？

Transformer 具有较好的训练并行性和扩展性，可以有效利用大量训练数据与计算资源；Attention 也让模型更容易学习上下文中不同位置之间的关系。

### 5. LLM 和 Agent 有什么区别？

LLM 主要负责理解、推理和生成；Agent 在模型之外加入工具调用、状态管理、任务规划、执行流程、权限控制和结果验证。

### 6. GPT 的能力为什么不只是来自“把模型做大”？

模型规模只是因素之一。模型能力还取决于架构、训练数据、计算资源、预训练目标、指令微调、偏好对齐以及外部工具和产品系统。

## 我的疑问与澄清

### RNN 和 LSTM 是什么？（已解决）

- RNN 按顺序处理数据，并将隐藏状态传给下一步。
- LSTM 通过门控机制改善长期记忆。
- Transformer 使用 Attention，更容易并行训练和处理长距离关系。

### Attention 是什么？（已解决）

- Attention 让模型判断上下文中不同 Token 与当前 Token 的相关程度。
- 模型按照相关性权重汇总信息，得到包含上下文的新表示。
- Attention 是一种计算机制，不等同于人类的注意力或真正的语义理解。
- 目前先掌握直觉，具体的 Query、Key、Value 和数学计算留到后续章节。

### 阅读后仍未解决的问题

- 当前暂无；后续阅读时只记录自己确实没有理解的问题。

## 后续学习问题

以下问题由本章引出，将在后续章节继续学习，不代表当前已经存在理解障碍：

- Attention 具体如何计算两个 Token 的相关性？
- Query、Key、Value 分别承担什么作用？
- GPT 为什么需要逐个生成 Token？

## 复习检查

- [ ] 不看笔记，说明 Transformer、GPT、ChatGPT 与 Agent 的区别。
- [ ] 解释 RNN、LSTM 与 Transformer 处理序列方式的差异。
- [ ] 用自己的话解释 Attention 如何汇总上下文信息。
- [ ] 用一分钟说明从 Transformer 到 ChatGPT 再到 Agent 的演进。
- [ ] 独立回答本页的六个面试问题。

## 关联

- 读书入口：[[wiki/读书笔记/Transformer 架构/00-索引|《Transformer 架构：从直觉到实现》读书笔记]]
- 已有课程：[[wiki/直播课/课程笔记/40-大语言模型原理|40 大语言模型原理]]
- Agent 延伸：[[wiki/直播课/课程笔记/43-Agent 与 Skills 编排|43 Agent 与 Skills 编排]]

## 来源

- [[raw/links/2026-08-10-Transformer 架构：从直觉到实现|Transformer 架构：从直觉到实现]]
