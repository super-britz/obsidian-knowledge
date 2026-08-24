---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html"
---

# OWASP：Server-Side Request Forgery Prevention Cheat Sheet

- **机构：** OWASP Cheat Sheet Series
- **URL：** https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- **访问日期：** 2026-08-24
- **资料类型：** 官方安全指南

## 来源摘要

指南解释了 SSRF 的基本风险：攻击者诱导一个有内部网络访问能力的服务端应用代为请求非预期目标，从而把服务端变成访问内部系统或云元数据服务的代理。

当业务只需要访问已知系统时，指南推荐在应用层和网络层进行纵深防御，校验输入格式，并将目标 IP、域名和协议限制在明确允许列表内。域名解析、重定向和编码差异都可能绕过过于简单的字符串判断，因此不能只依赖一个正则或拒绝列表。

## 使用边界

- BFF 若只调用代码中固定配置的下游，风险面小于接受任意目标 URL 的通用代理，但仍要保护配置、重定向、DNS 与网络出口。
- 允许列表必须覆盖协议、主机、端口、路径和解析结果中的实际风险，具体实现取决于网络与运行环境。
- SSRF 防护不替代下游的身份认证、对象级授权和最小权限。
