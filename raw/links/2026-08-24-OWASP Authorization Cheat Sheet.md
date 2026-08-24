---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html"
---

# OWASP：Authorization Cheat Sheet

- **机构：** OWASP Cheat Sheet Series
- **URL：** https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- **访问日期：** 2026-08-24
- **资料类型：** 官方安全指南

## 来源摘要

指南区分 Authentication 与 Authorization，并建议默认拒绝、最小权限，以及对每一次请求执行权限检查。请求来自普通导航、AJAX、服务端渲染还是其他入口，都不能成为跳过对象级和操作级授权的理由。

指南还指出，静态资源是否需要访问控制取决于其内容；仅隐藏链接、按钮或难猜的标识符不能保护底层资源。授权规则需要日志和单元、集成测试支持，避免某条替代入口绕过检查。

## 使用边界

- OWASP 提供安全原则，不决定具体项目采用 Session、Token、RBAC、ABAC 或其他实现。
- 服务端渲染能在可信执行环境读取数据，但不会自动赋予正确权限；每个请求仍需按主体、对象与动作校验。
- 页面缓存和静态产物也是数据副本，含私有内容时必须纳入授权、隔离和删除策略。
