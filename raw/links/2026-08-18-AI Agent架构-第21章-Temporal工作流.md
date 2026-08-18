---
type: 链接来源
status: 已记录
created: 2026-08-18
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC21%E7%AB%A0-Temporal%E5%B7%A5%E4%BD%9C%E6%B5%81/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 21 章 Temporal 工作流

- **原文标题：** 第 21 章：Temporal 工作流
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC21%E7%AB%A0-Temporal%E5%B7%A5%E4%BD%9C%E6%B5%81/
- **访问日期：** 2026-08-18
- **资料类型：** 在线书籍章节页

## 来源摘要

本章介绍 Temporal 如何以事件历史持久化长时工作流：Workflow 只负责确定性的编排逻辑，Activity 负责搜索、LLM 调用等可产生副作用的实际工作。Worker 或进程崩溃后，Workflow 重放已记录的历史，已完成 Activity 的结果从历史中恢复，从而从可靠检查点继续。

原文强调 Workflow 不应直接调用时间、随机数、网络、环境变量或普通并发原语；这些不确定性应使用 Temporal 提供的工作流 API 或放进 Activity。工作流代码变更必须通过 `GetVersion` 进行版本门控，保证新代码不会使仍在运行的旧历史重放失败。信号用于改变运行中的行为，查询只读取状态。

章节还覆盖活动/总体/心跳超时、优先级队列、并行 Future、子工作流、取消、事件历史膨胀与时间旅行调试。原文建议大型结果只传递引用，长循环通过 Continue-As-New 控制历史长度，并通过事件历史定位和重放故障。

## 使用边界

- Temporal 提供持久化编排与重放，不会让外部副作用自动具备幂等性或跨服务原子性；写操作仍需幂等键、去重、补偿或人工处理策略。
- 确定性约束适用于 Workflow 逻辑而非 Activity；将任意逻辑都放进 Activity 会失去可观察的编排语义，将副作用塞进 Workflow 则会破坏重放。
- 工作流事件历史是持久化执行记录，不是长期业务数据仓库；大对象、敏感数据和无界循环应外置、控制或轮换。
