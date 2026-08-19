---
type: 链接来源
status: 已记录
created: 2026-08-19
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC38%E7%AB%A0-Deferred-Tool-Loading%E4%B8%8ETool-Search/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 38 章 Deferred Tool Loading 与 Tool Search

- **原文标题：** 第 38 章：Deferred Tool Loading 与 Tool Search
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC38%E7%AB%A0-Deferred-Tool-Loading%E4%B8%8ETool-Search/
- **访问日期：** 2026-08-19
- **资料类型：** 在线书籍章节页

## 来源摘要

本章处理工具 Schema 目录占据 Prompt 的问题。是否延迟工具不应按工具数量判断，而要按序列化 Schema 的 token 成本做保守预算：少数复杂工具可能比大量简单工具更贵。进入 Deferred Mode 的条件包括总 Schema 预算超限，以及冷集合中出现“昂贵但极少使用”的类别；后者即使未超预算，也能减少每个冷启动会话的默认负担。

并非越多延迟越好。开场高频工具不应延迟，因为模型首次检索会把额外往返直接加到用户的第一条回复；隐式记忆未命中后的兜底工具也可能需要常驻，避免模型在没有 Schema 时猜测旧参数。常驻在稳定、可缓存的 system 前缀中的 Schema，成本会被会话内多轮使用摊薄，因此不能只看单次体量。

模型检索过的工具进入 Session 级 Warm Set；每个会话从冷集合开始，且缓存键必须基于 Schema 内容指纹而非工具名称，以应对 MCP Server 的参数变更。Deferred Loading 只是隐藏模型看到的 Schema、引导它通过 tool_search 发现能力，不是权限控制；危险工具的允许与拒绝仍须在执行时强制。工具目录中的名称和描述必须足够表达用途，否则模型无法形成有效检索查询。

## 使用边界

- 延迟加载的收益是减少默认 Prompt 负担，代价是首次发现某工具时多一次检索往返；应按真实冷启动与开场路径测量，而非凭直觉分类。
- Warm Set 优化的是同一 Session 内的后续轮次，不能掩盖每个新会话仍从冷启动开始的体验成本。
- 公开源码中的具体预算、类别和延迟数字属于有日期的实现快照；可迁移原则是“按 token 计量、按会话工作集缓存、按内容失效、检索与授权分离”。
