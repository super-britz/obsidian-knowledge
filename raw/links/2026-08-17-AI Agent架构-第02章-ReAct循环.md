---
type: 链接来源
status: 已记录
created: 2026-08-17
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC02%E7%AB%A0-ReAct%E5%BE%AA%E7%8E%AF/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 2 章 ReAct 循环

- **原文标题：** 第 2 章：ReAct 循环
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC02%E7%AB%A0-ReAct%E5%BE%AA%E7%8E%AF/
- **访问日期：** 2026-08-17
- **资料类型：** 在线书籍章节页

## 来源摘要

本章把 ReAct 定义为 Reason、Act、Observe 的交织循环：模型基于当前目标和已有观察决定下一步，调用工具或向用户追问，记录客观的执行结果，再以结果驱动下一轮决策。它的价值不是保证模型正确，而是让系统从静态猜测转向获取证据、修正路径和保留可追溯过程。

章节用竞品定价调查和 API 500 错误排查说明：只有在每步行动后读取真实结果，模型才会获得原本不在上下文中的信息。Reason 负责识别缺口和选择下一步，Act 负责执行一个可验证动作，Observe 只记录事实并将其送回下一轮判断。

原文列出任务完成、结果收敛、最大迭代、用户中断、预算耗尽与超时等终止条件；其中预算和超时属于硬护栏。生产配置还讨论 MaxIterations、MinIterations、ObservationWindow、Token 预算、提前停止和“必须有工具证据才能完成”等机制，用来应对无限循环、过早收工、上下文膨胀和思考行动脱节。

## 使用边界

- ReAct 是跨框架的设计模式；Shannon、LangChain、LangGraph 等具体 API 仅是实现示例，不构成稳定定义。
- 章节给出的具体参数数值属于实践建议，必须结合任务风险、工具延迟、模型能力和成本预算验证。
- 工具成功不等于任务完成；ReAct 也不等于生产可用，仍需外部验收、权限控制、预算、超时、审计与恢复机制。
