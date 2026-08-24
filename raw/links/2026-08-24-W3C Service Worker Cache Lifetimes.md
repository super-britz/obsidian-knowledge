---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.w3.org/TR/service-workers/#cache-lifetimes"
---

# W3C Service Workers：Cache Lifetimes

- **机构：** W3C
- **URL：** https://www.w3.org/TR/service-workers/#cache-lifetimes
- **访问日期：** 2026-08-24
- **资料类型：** 官方规范草案

## 来源摘要

Service Workers 规范说明，Cache API 由脚本完全管理，并与浏览器 HTTP Cache 相互独立。Cache 中的条目不会自动更新，也不会自动过期；Service Worker 脚本升级本身也不会让既有 Cache 自动消失。

规范因此建议按名称给 Cache 做版本管理，并确保只有能够安全解释该版本内容的 Service Worker 使用它。CacheStorage 提供 open、match、delete 等接口，但更新、淘汰和旧版本清理由应用作者负责。

## 使用边界

- Cache API 提供存取机制，不提供应用数据的新鲜度、冲突合并或服务端同步协议。
- 持久化离线副本会增加版本迁移、配额、隐私、退出登录清理和删除传播责任。
- HTTP Cache Header 不会自动治理脚本写入的 Cache API 条目；两层需要分别设计和验证。
