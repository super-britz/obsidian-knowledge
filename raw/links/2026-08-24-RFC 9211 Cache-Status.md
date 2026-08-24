---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.rfc-editor.org/rfc/rfc9211.html"
---

# RFC 9211：Cache-Status

- **机构：** IETF / RFC Editor
- **URL：** https://www.rfc-editor.org/rfc/rfc9211.html
- **访问日期：** 2026-08-24
- **资料类型：** Standards Track RFC

## 来源摘要

RFC 9211 定义标准化的 `Cache-Status` 响应 Header，用来解释一层或多层 HTTP Cache 怎样处理本次请求。每个成员可以表示 hit，或用 fwd 说明请求为何被转发，例如 URI Miss、Vary Miss、Stale 或 Bypass；还可携带剩余 Freshness Lifetime、是否存储、是否合并请求等信息。

多个缓存可以保留既有成员并追加自己的处理结果，从而呈现从源站到用户的缓存链路。规范同时指出，暴露命中状态、Cache Key 或内部细节可能帮助攻击者推测用户活动或实施 Cache Poisoning，应按授权限制信息。

## 使用边界

- `Cache-Status` 解释 HTTP Cache 行为，不会自动覆盖前端 Query Cache、Service Worker Cache 或 IndexedDB 副本。
- Cache Hit 只说明请求由缓存满足，不能单独证明缓存键、数据隔离和业务新鲜度正确。
- 生产环境是否暴露 Key、主机名和 Detail，需要经过安全评估；必要时只对授权诊断请求提供。
