---
type: 链接来源
status: 已记录
created: 2026-08-18
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC27%E7%AB%A0-Deep-Research/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 27 章 Deep Research

- **原文标题：** 第 27 章：Deep Research
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC27%E7%AB%A0-Deep-Research/
- **访问日期：** 2026-08-18
- **资料类型：** 在线书籍章节页

## 来源摘要

本章将 Deep Research 定义为多轮的 Plan → Search → Evaluate → Verify → Synthesize 研究过程，而非一次搜索后拼接摘要。研究先按任务类型与维度制定计划；每轮根据已知信息、空白维度和冲突调整查询；最终输出结构化报告及可追溯引用。

原章比较单 Agent 强推理与 Lead/Sub-agent 协作两条路线。多 Agent 以任务契约划分探索维度，子 Agent 将完整发现存储、只回传摘要和引用，降低主 Agent 上下文压力；但协调和 Token 成本更高，效果提升不能简单归因于“Agent 数量”。

章节强调研究质量来自实际访问并核对 URL、跟踪结论与来源的对应关系、优先一手和高质量来源、呈现来源冲突，以及按信息时效、实体主场语言和地域选择检索策略。停止条件应同时有覆盖率等软标准和最大轮数、页面数、预算等硬上限；长任务应保留检查点，允许单个搜索或页面失败后改换路径继续。

## 使用边界

- 引用只能证明报告可追溯，不能自动证明结论为真；来源质量、时效、适用范围和冲突仍需在报告中显式呈现。
- 覆盖率评估是研究完整性的近似信号，不是客观真理；它需要结合问题范围、时间预算和人工判断，避免将“达到阈值”误称为“没有遗漏”。
- 多 Agent 适合可并行的复杂维度探索；简单事实问题通常不值得承担协调、压缩、上下文传递和额外 Token 的成本。
