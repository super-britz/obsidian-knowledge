---
type: 链接来源
status: 已记录
created: 2026-08-18
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC33%E7%AB%A0-Building-on-the-Harness-ShanClaw/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 33 章 Building on the Harness — Kocoro

- **原文标题：** 第 33 章：Building on the Harness — Kocoro
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC33%E7%AB%A0-Building-on-the-Harness-ShanClaw/
- **访问日期：** 2026-08-18
- **资料类型：** 在线书籍章节页

## 来源摘要

本章从单次 Agent Loop 进入持久运行时：身份、Session 状态、Memory、权限、集成、自动化和验证都必须成为独立契约，且 TUI、Daemon、Schedule 与 MCP 不能绕过共同的权限和执行核心。文章将 Kocoro 公开引擎、CLI 和 Daemon 作为参考，并明确原生 Desktop 产品与私有实现不属于公开资料范围。

Named Agent 组合指令、模型策略、工具/MCP 范围、Session、Memory 与可选 Skills；配置应遵循最小权限、最终值来源可见和切换 Agent 时重建范围，避免意外继承。工作上下文、可恢复 Session 与跨会话 Memory 应分离；压缩有损，持久事实需要来源、时间、作用域、去重、冲突处理和隐私过滤。

Daemon 接收渠道消息后要验证来源、选择 Agent 与 Session、通过共享 Harness 执行、支持审批/Pending 状态和流式事件，并定义取消、超时、重启与重复投递的终态。渠道元数据和工具输出都属于不可信数据，不能自行扩大权限或变成系统指令。MCP Client/Server、Schedule、Watcher 和 Heartbeat 同样要遵守共同的权限、审计、幂等、防重叠和禁用机制；自动触发通常权限更少。

章节将发布质量定义为持续、安全地完成任务：使用工具/权限/隔离/取消等契约测试、合成 Golden Trace 回放、故障注入、隔离 Canary 或 Shadow Run、受监控发布与回滚演练。公开资料只可使用公开命令、概念、通用模式与合成示例，不能泄露私有源码、生产拓扑、客户数据或可重建原系统的细节。

## 使用边界

- Named Agent 是缩小作用域的配置单元，不是天然隔离边界；共享 Harness、权限与数据范围仍必须在运行时强制执行。
- Golden Trace 用于回归语义和安全决策，不应要求模型逐字复现；轨迹必须合成、脱敏且不携带客户或生产资产。
- 自动化触发提高覆盖和响应速度，却扩大了无人值守副作用；每种触发器都要定义去重、重叠、失败可见性和关闭路径。
