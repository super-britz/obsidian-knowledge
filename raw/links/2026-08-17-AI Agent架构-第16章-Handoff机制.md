---
type: 链接来源
status: 已记录
created: 2026-08-17
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC16%E7%AB%A0-Handoff%E6%9C%BA%E5%88%B6/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 16 章 Handoff 机制

- **原文标题：** 第 16 章：Handoff 机制
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC16%E7%AB%A0-Handoff%E6%9C%BA%E5%88%B6/
- **访问日期：** 2026-08-17
- **资料类型：** 在线书籍章节页

## 来源摘要

本章讨论 Agent 间如何传递数据与状态，并按协作复杂度划分三层机制：线性或固定依赖可把 `previous_results` / `dependency_results` 注入下游上下文；多个 Agent 在运行中共享发现时可使用按 Topic 组织的 Workspace，并以序列号增量读取；只有出现请求、提议、接受、委派等双向协商时，才使用 P2P Mailbox 消息协议。

原文建议在任务分解阶段声明 `Produces` 与 `Consumes`，使编排器能理解数据流、调度依赖并在启动前检查“消费的 Topic 是否存在生产者”。依赖等待需要超时和增量检查，避免无限等待；Workspace 与 Mailbox 的数据应限制大小、设置保留期，并使用工作流时间而非非确定性系统时间，以支持可靠重放。

章节强调按需选择交接复杂度：简单链式任务不应为“完整架构”引入 Workspace 或 P2P；Workspace 更适合一对多的共享发现，P2P 更适合双向协调。大内容应传递外部存储引用而非整段数据，主题名称也应标准化以避免生产者和消费者错配。

## 使用边界

- Handoff 解决的是产物和状态的交接，不证明内容正确、来源可信或接收者有权继续执行；这些仍需验证、授权和验收。
- 生产者集合检查只能发现缺失生产者，不能保证生产者会成功写出可消费、未过期且语义正确的产物。
- Workspace 与 P2P 是运行时协作介质，不应取代权威业务记录；敏感数据、跨租户可见性、保留和删除规则必须由独立存储与治理层处理。
