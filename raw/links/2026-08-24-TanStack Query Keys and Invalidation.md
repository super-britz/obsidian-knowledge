---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://tanstack.com/query/latest/docs/framework/react/guides/query-keys"
---

# TanStack Query：Query Keys、Invalidation 与 Optimistic Updates

- **机构：** TanStack Query
- **Query Keys：** https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- **Query Invalidation：** https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- **Optimistic Updates：** https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- **访问日期：** 2026-08-24
- **资料类型：** 官方 v5 文档

## 来源摘要

TanStack Query v5 根据 Query Key 管理查询缓存。官方要求 Key 在顶层使用数组、能够序列化并唯一描述查询数据；Query Function 依赖且会改变结果的变量应进入 Key。

调用 `invalidateQueries` 会把匹配 Query 标记为 stale，覆盖原有 `staleTime`；当前正在渲染的 Query 还会在后台重新获取。失效因此不是直接证明数据已经更新，而是改变本地副本的可信状态并触发后续协调。

官方乐观更新指南给出两类做法：只在 UI 中显示待确认变量，或直接更新 Cache。直接更新时需要处理进行中的 Refetch、保存旧快照、失败回滚，并在结束后重新失效或获取权威数据。

## 使用边界

- 这些行为是 TanStack Query v5 的具体实现，不代表所有查询库具有完全相同的默认值和 API。
- Query Key 正确只解决本地副本身份；服务端权限、响应 `Vary`、CDN Key 和多租户隔离仍需各层独立保证。
- 回滚到旧快照可能覆盖更新的并发结果；复杂并发仍需要版本、操作身份和权威重取。
