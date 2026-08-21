---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html"
---

# OWASP Authorization Cheat Sheet

- **机构：** OWASP Cheat Sheet Series
- **URL：** https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- **访问日期：** 2026-08-21
- **资料类型：** 官方安全实践指南

## 来源摘要

该指南区分身份认证与授权：确认请求者是谁，不代表其有权访问任意资源或执行任意动作。权限应默认拒绝，并针对每次请求及具体对象进行校验。

客户端访问控制可以改善界面体验，但容易被绕过，不能成为授予或拒绝资源访问的决定性控制；授权检查应位于服务端、网关或相应受信任的 Serverless 执行边界。

## 使用边界

- 授权模型需要结合业务资源、主体、动作、关系和环境条件设计，不能只靠隐藏按钮或角色名称。
- 自动化测试可以发现部分问题，但不能替代威胁建模、安全评审和必要的人工测试。
- 本页只记录与前端可信边界相关的原则，不展开具体 RBAC、ABAC 或 ReBAC 选型。
