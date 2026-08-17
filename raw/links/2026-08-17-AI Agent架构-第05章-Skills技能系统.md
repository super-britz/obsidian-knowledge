---
type: 链接来源
status: 已记录
created: 2026-08-17
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC05%E7%AB%A0-Skills%E6%8A%80%E8%83%BD%E7%B3%BB%E7%BB%9F/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 5 章 Skills 技能系统

- **原文标题：** 第 5 章：Skills 技能系统
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC05%E7%AB%A0-Skills%E6%8A%80%E8%83%BD%E7%B3%BB%E7%BB%9F/
- **访问日期：** 2026-08-17
- **资料类型：** 在线书籍章节页

## 来源摘要

本章把技能系统定义为将 System Prompt、工具白名单和参数约束打包成可复用配置。它解决同一个通用 Agent 面对代码审查、研究、写作等不同任务时，角色指令、可用工具和成本边界不匹配的问题。Shannon 的 Presets 是代码级配置；Agent Skills 则以包含 SKILL.md 的目录作为文件级、可跨平台分发的技能单元。

章节进一步说明渐进式披露：启动时只加载 Skill 的名称和 description；任务匹配时才加载完整指令；引用资料、示例和脚本只在实际需要时加载。它用按需加载缓解大量 MCP 工具定义和领域资料占满上下文的问题。Skills 与 MCP 互补：MCP 连接外部能力，Skills 编码完成具体任务时的工作流知识与调用约束。

原文还讨论了模板变量、运行时动态增强、Vendor Adapter、子代理，以及 System Prompt 含糊、工具权限过宽、缺少参数约束与降级策略等常见问题。对副作用较大的技能，可通过调用控制限制仅由用户触发；但具体前端字段、生态支持范围与文件格式应以各平台当前文档为准。

## 使用边界

- Skill 是任务配置、工作流知识和上下文装载单元，不是绕过工具授权或策略治理的安全边界。
- 框架 Preset 与跨平台 Agent Skills 的格式不同；本页只提炼其共同的机制，不将某个厂商字段视为通用标准。
- Skills 不取代 MCP：前者定义“在什么任务中、怎样组合使用能力”，后者提供“如何连接和调用外部能力”。
