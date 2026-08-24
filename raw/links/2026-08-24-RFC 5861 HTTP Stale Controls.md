---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.rfc-editor.org/rfc/rfc5861.html"
---

# RFC 5861：HTTP Stale Controls

- **机构：** RFC Editor
- **URL：** https://www.rfc-editor.org/rfc/rfc5861.html
- **访问日期：** 2026-08-24
- **资料类型：** Informational RFC

## 来源摘要

RFC 5861 定义 `stale-while-revalidate` 与 `stale-if-error` 两个 Cache-Control 扩展。前者允许缓存进入 stale 后在指定秒数内先返回旧响应，同时在后台尝试重新校验，以隐藏等待延迟；后者允许在源站错误或网络故障时，于指定范围内用旧响应代替硬失败。

这些指令给旧副本的使用增加了明确窗口，而不是把 stale 重新解释为 fresh。窗口结束或条件不满足后，缓存仍应回到正常校验或失败行为。

## 使用边界

- 用旧值换可用性必须由业务容忍度决定；权限、价格、库存和交易结果不应仅因技术上可返回就使用旧值。
- `stale-while-revalidate` 只有在窗口内出现请求并触发校验时才可能完成后台更新。
- 延长旧值窗口会降低等待，但也扩大用户观察到过期事实的最长时间。
