---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://redux.js.org/style-guide/#treat-reducers-as-state-machines"
---

# Redux Style Guide：Treat Reducers as State Machines

- **机构：** Redux 官方文档
- **URL：** https://redux.js.org/style-guide/#treat-reducers-as-state-machines
- **访问日期：** 2026-08-21
- **资料类型：** 官方状态管理风格指南

## 来源摘要

Redux 风格指南建议把 Reducer 当作状态机：新状态应由“当前状态 + 收到的 Action”共同决定，而不是只看 Action 后无条件更新。例如，请求成功事件只有在当前确实处于加载状态时才应生效。

指南以 `idle`、`loading`、`success`、`failure` 等有限状态为例，并说明 TypeScript 可用可辨识联合让不同状态只携带各自合法的数据，从而减少不可能组合。

## 使用边界

- 这是 Redux 的实现建议，但“当前状态 + 事件决定合法转换”的机制不依赖 Redux。
- 只有状态枚举仍不能处理乱序响应；事件还需要携带请求或操作身份。
- 状态机不会自动保证副作用幂等、接口契约正确或服务端事实一致。
