---
type: 链接来源
status: 已记录
created: 2026-08-17
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC15%E7%AB%A0-Swarm%E6%A8%A1%E5%BC%8F/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 15 章 Swarm 模式

- **原文标题：** 第 15 章：Swarm 模式
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC15%E7%AB%A0-Swarm%E6%A8%A1%E5%BC%8F/
- **访问日期：** 2026-08-17
- **资料类型：** 在线书籍章节页

## 来源摘要

本章讨论当任务结构无法在启动前固定、执行中需要新增或重分配工作、Agent 需要互相协作且用户需要实时介入时，如何从静态 DAG 转向 Swarm。原文将 Swarm 定义为 Lead 驱动的事件循环：Lead 负责初始规划、创建或复用 Worker、按 idle/completed/checkpoint/human_input 等事件调整计划、检查质量并在结束时综合；Worker 则各自运行受控的 ReAct 循环。

原文区分两种协作通道：Workspace 面向共享发现的一对多广播，P2P Mailbox 面向请求、委派和通知的一对一通信。Worker 完成局部工作后进入 idle，Lead 可读取产物、决定接受、重试、重新分配或关闭，从而避免每项新任务都重新创建 Agent。用户输入也作为事件进入 Lead 循环，使运行中的计划可以被人类纠偏。

章节以纯去中心化协作在强耦合任务上难以收敛为例，强调群体协作需要全局协调与可靠验证。原文同时提示要限制 Agent 数量和 Token 预算、检测无实质产出的循环，并将 Workspace 视为临时协作区而不是权威持久数据库。

## 使用边界

- Swarm 的价值来自动态调整和协作，不来自 Agent 数量；任务图稳定、依赖明确时，DAG 的可预测性和较低协调成本通常更合适。
- Worker 的“完成”声明和 Lead 的文件阅读都不是事实正确性证明；关键结论、代码和副作用仍需测试、来源校验、策略与审批。
- Workspace 与 P2P 消息需要范围、权限、版本、保留期和冲突规则；共享信息若没有治理，会放大噪声、过期数据与提示注入风险。
