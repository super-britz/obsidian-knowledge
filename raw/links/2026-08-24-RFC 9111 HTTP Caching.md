---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.rfc-editor.org/rfc/rfc9111.html"
---

# RFC 9111：HTTP Caching

- **机构：** IETF / RFC Editor
- **URL：** https://www.rfc-editor.org/rfc/rfc9111.html
- **访问日期：** 2026-08-24
- **资料类型：** Internet Standard

## 来源摘要

RFC 9111 定义 HTTP Cache 以及控制缓存行为的 Header。Cache Key 至少由请求方法与目标 URI 构成；当响应包含 `Vary` 时，缓存还必须比较被列出的请求 Header，不能只因 URL 相同就复用响应。

规范把存储响应区分为 fresh 与 stale。Freshness Lifetime 可以来自 `s-maxage`、`max-age` 或 `Expires`；Fresh 响应可以不联系源站直接复用。Stale 响应通常需要通过条件请求重新校验，除非协议或约定明确允许返回旧值。`no-cache` 允许存储但要求复用前成功校验，`no-store` 才表示不要存储。

对成功的 PUT、POST、DELETE 等不安全请求，经过该请求路径的缓存必须使目标 URI 的已存响应失效；规范同时指出，这不能保证所有相关响应在全局都被失效。

## 使用边界

- Fresh 只表示缓存策略允许当前副本不经校验复用，不证明源站事实在此期间绝对没有变化。
- HTTP 规范无法知道商品列表、统计聚合等业务依赖图；一个写请求还影响哪些其他 Query，需要应用层明确。
- 缓存淘汰可能早于过期时间；Freshness Lifetime 不是存储保留期限或命中保证。
