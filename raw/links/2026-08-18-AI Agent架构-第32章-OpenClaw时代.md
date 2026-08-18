---
type: 链接来源
status: 已记录
created: 2026-08-18
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC32%E7%AB%A0-OpenClaw%E6%97%B6%E4%BB%A3/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 32 章 OpenClaw 时代

- **原文标题：** 第 32 章：OpenClaw 时代
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC32%E7%AB%A0-OpenClaw%E6%97%B6%E4%BB%A3/
- **访问日期：** 2026-08-18
- **资料类型：** 在线书籍章节页

## 来源摘要

本章将“OpenClaw 时代”概括为一类本地 Agent 架构：Agent 在用户机器上持续运行，通过文件、Shell、浏览器或 GUI 工具完成任务，并要求全过程可审计、可中断、可扩展。其价值在于可访问用户本地状态和语义化 GUI，同时代价是 Agent 获得了真实系统副作用，权限边界必须主动构建。

原章以 Agent Harness 作为执行骨架：每轮先由 LLM 生成工具请求；工具调用经历串行准入（去重、缓存、权限、审批、Pre-hook）、并行执行和串行收尾（Post-hook、审计、循环检测、写入上下文）。长任务通过受限摘要压缩上下文、进度检查点和“文本伪装工具调用”纠正机制保持收敛。

本地界面操作优先使用 Accessibility Tree 等语义树，以稳定引用 ID 操作元素；坐标点击与截图验证只作为通用但脆弱的兜底。浏览器应使用独立配置文件，隔离用户 Cookie 与历史。Hooks 在 PreToolUse、PostToolUse、SessionStart、Stop 等事件运行，PreToolUse 可通过约定退出码拒绝调用；Hook 命令需要受限路径、超时和输出大小边界。

章节的权限引擎按硬封锁、配置拒绝、复合命令拆解、配置允许、用户审批五层处理，并为文件、网络等工具增加专项边界。循环检测采用重复调用、无进展等互补信号，按提醒或强制停止处理。原章还讨论本地工具执行与远程 LLM 推理的分离，以及网络失败后的有界重试和优雅退出。

## 使用边界

- 本地优先提升可见性和对本地资源的访问，不自动带来安全；已登录会话、私钥和内网访问同样扩大了错误或被诱导操作的影响面。
- Accessibility Tree 比坐标稳定，但并非所有应用完整暴露语义；语义引用失效后仍需重新观察和验证，不能盲目复用。
- Hook 是可编程控制面，不是安全旁路；Hook 自身也具有执行权限，应按来源、路径、超时、输出和审计受控。
