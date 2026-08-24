---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://cheatsheetseries.owasp.org/cheatsheets/Third_Party_Javascript_Management_Cheat_Sheet.html"
---

# OWASP：Third Party JavaScript Management Cheat Sheet

- **机构：** OWASP Cheat Sheet Series
- **URL：** https://cheatsheetseries.owasp.org/cheatsheets/Third_Party_Javascript_Management_Cheat_Sheet.html
- **访问日期：** 2026-08-24
- **资料类型：** 官方安全指南

## 来源摘要

指南指出，把外部 JavaScript 加载进页面会带来失去变更控制、在用户环境执行任意代码和泄露敏感信息等风险。它讨论了服务端数据转发、Sandbox、Subresource Integrity、依赖更新和 iframe 隔离等防护方向。

微前端 Remote 即使来自组织内部，也同样属于独立发布后在宿主页执行的代码。其团队、构建系统、CDN 或发布凭据一旦被攻陷，影响可能传播到宿主页能够访问的 DOM、Storage、Session 和数据。

## 使用边界

- 原指南主要面向第三方脚本；将其风险模型迁移到内部 Remote 是本知识页的机制推导，不是原文对微前端的直接结论。
- SRI、CSP、Sandbox 和依赖扫描是纵深防御，不能替代来源信任、最小权限、发布审批和事故响应。
- 动态更新与固定内容 Hash 之间存在交付权衡，具体校验方案必须与 Module Loader、缓存和回滚设计共同验证。
