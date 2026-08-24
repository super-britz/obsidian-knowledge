---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.rfc-editor.org/rfc/rfc10017.html"
---

# RFC 10017：OAuth 2.0 for Browser-Based Applications

- **机构：** IETF / RFC Editor
- **URL：** https://www.rfc-editor.org/rfc/rfc10017.html
- **访问日期：** 2026-08-24
- **资料类型：** Best Current Practice（RFC）
- **版本状态：** RFC 10017，BCP 212，2026 年 8 月发布

## 来源摘要

RFC 10017 在浏览器 OAuth 语境下描述了一种 BFF：服务端组件作为机密 OAuth Client 完成授权职责，把访问令牌和刷新令牌关联到基于 Cookie 的会话中，浏览器不直接获得这些令牌；BFF 再代表前端向 Resource Server 发起请求。

规范同时说明，令牌不暴露给浏览器并不意味着浏览器中的恶意代码无法借用户会话发起操作。Cookie 会话必须实施 CSRF 防御；可动态代理请求时，BFF 必须只允许显式批准的目标主机和路径，否则可能造成未授权访问或令牌泄漏。

## 使用边界

- 这是面向浏览器 OAuth Client 的安全最佳实践，不是所有广义数据聚合 BFF 的通用定义。
- BFF 降低令牌直接泄漏与新令牌获取风险，不会消除 XSS、CSRF、Client Hijacking、越权、代理滥用和会话保护责任。
- RFC 仍需要结合具体身份提供商、部署域、资源服务器和威胁模型落地，不能只根据架构名称推断系统已经安全。
