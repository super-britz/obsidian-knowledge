---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://react.dev/reference/react-dom/static/prerender"
---

# React：Static Prerender

- **机构：** React
- **URL：** https://react.dev/reference/react-dom/static/prerender
- **访问日期：** 2026-08-24
- **资料类型：** 官方参考文档

## 来源摘要

React 的 `prerender` API 用于生成静态 HTML。它会等待渲染所需的数据准备完成，再返回完整静态输出，官方将其定位为 Static Site Generation；若要在数据逐步到达时发送内容，应使用 Streaming SSR API。

静态输出可以不包含客户端启动脚本，成为纯 HTML；需要交互时，也可以加入脚本并在浏览器调用 `hydrateRoot`。因此“提前生成 HTML”与“是否向浏览器发送并执行 React”是两个独立选择。

## 使用边界

- React API 证明的是静态生成机制，不规定产物必须在何时构建、怎样发布、多久重新生成或如何失效。
- 静态 HTML 的新鲜度取决于构建、再生成和缓存协议，不能从“静态”推断“永远最新”。
- 动态路径数量、构建所需数据和失败恢复会决定静态生成的时间与运维成本。
