---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary"
---

# React：Error Boundaries

- **机构：** React
- **URL：** https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- **访问日期：** 2026-08-24
- **资料类型：** 官方参考文档

## 来源摘要

React 官方文档说明，渲染期间发生错误时，默认会从屏幕移除相关 UI；Error Boundary 可以包围一部分组件树，在后代组件渲染失败时显示替代界面，并通过 `componentDidCatch` 等入口记录错误。

官方同时明确列出其捕获边界：它不捕获事件处理器、服务端渲染、Boundary 自身抛出的错误，以及通常的异步回调错误。文档还建议按“在哪里显示错误信息有意义”决定粒度，而不是给每个组件都套一层 Boundary。

## 使用边界

- Error Boundary 是 React 渲染树中的故障隔离机制，不是所有 JavaScript、网络或业务错误的统一处理器。
- 显示 fallback 只证明崩溃 UI 已被替换，不证明数据、副作用或依赖已经恢复。
- Boundary 的放置仍需由独立有用的界面单元、状态归属和恢复方式决定。
