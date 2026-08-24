---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://react.dev/reference/react-dom/server/renderToPipeableStream"
---

# React：Streaming Server Rendering

- **机构：** React
- **URL：** https://react.dev/reference/react-dom/server/renderToPipeableStream
- **访问日期：** 2026-08-24
- **资料类型：** 官方参考文档

## 来源摘要

`renderToPipeableStream` 将 React 树渲染为 Node.js 可写入的 HTML Stream。服务端可以在 Shell 准备后先发送页面骨架，再以 `Suspense` Boundary 为单位，在数据到达后继续发送其余 HTML；客户端仍需 Hydration 才能让服务端 HTML 具备完整交互。

官方文档区分 Shell 内外的失败：Shell 生成失败时可以在尚未开始流式响应前返回替代 HTML；Boundary 内的后续区域失败时，服务端可先发送 fallback，并让客户端尝试恢复。文档也指出，响应开始发送后便不能再任意修改 HTTP 状态码。

## 使用边界

- 流式传输改变的是独立区域的等待顺序，不会让慢数据源、浏览器脚本执行或 Hydration 本身变快。
- `Suspense` Boundary 同时影响呈现顺序和失败恢复，粒度需要贴合用户可理解的页面区域。
- 代理、网关或平台若缓冲响应，应用层产生 Stream 也不一定能让用户提前收到内容；需要端到端验证。
