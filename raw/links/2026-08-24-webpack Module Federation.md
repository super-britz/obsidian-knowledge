---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://webpack.js.org/concepts/module-federation/"
---

# webpack：Module Federation

- **项目：** webpack
- **概念文档：** https://webpack.js.org/concepts/module-federation/
- **插件配置：** https://webpack.js.org/plugins/module-federation-plugin/
- **访问日期：** 2026-08-24
- **资料类型：** 官方概念与配置文档

## 来源摘要

Module Federation 让多个独立 Build 在运行时组成一个应用。Host 可以异步加载 Remote Container 暴露的模块；Container 通过初始化共享作用域并提供模块工厂完成运行时连接，因此远程模块会进入额外的 Chunk Loading 路径。

`shared` 可以在多个 Build 间协商依赖；`requiredVersion`、`singleton` 和 `strictVersion` 分别表达版本范围、单实例需求和不兼容时是否拒绝。官方文档特别指出，React 等依赖全局内部状态的库通常需要单实例，但这种共享也会把运行时版本兼容变成跨 Build 契约。

## 使用边界

- Module Federation 是远程模块加载与依赖共享机制，不等于已经建立微前端的业务、团队和交付边界。
- 能加载任意 Remote 不代表任意版本组合都兼容；公开模块、共享依赖和 Host 生命周期仍需版本策略与故障兜底。
- Remote 加载处于页面关键路径，必须同时评估网络、缓存、失败、安全与用户性能。
