---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://react.dev/reference/react-dom/client/hydrateRoot"
---

# React：hydrateRoot

- **机构：** React
- **URL：** https://react.dev/reference/react-dom/client/hydrateRoot
- **访问日期：** 2026-08-24
- **资料类型：** 官方参考文档

## 来源摘要

React 官方文档将 Hydration 描述为：客户端把组件逻辑附着到服务端已经生成的 HTML 上，使初始 HTML 快照变成可交互应用。`hydrateRoot` 期望客户端首次渲染结果与服务端输出一致；开发环境会提示不匹配，但 React 不保证修补所有属性差异。

文档列出的常见不匹配来源包括：服务端与客户端使用不同数据、在渲染逻辑中读取浏览器专属 API，以及根据 `window` 是否存在直接产生不同输出。不匹配最好情况会增加额外工作，严重时事件处理器可能附着到错误元素。

## 使用边界

- 页面已经显示服务端 HTML，不表示事件处理器和客户端状态已经就绪；“可见”与“可交互”必须分开衡量。
- `hydrateRoot` 是 React 的具体机制；其他框架可能采用不同名称或选择性激活方式，但仍需处理静态输出与客户端行为之间的交接。
- 抑制 Hydration 警告只是逃生口，不能替代修复数据、环境和版本不一致。
