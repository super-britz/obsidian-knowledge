---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://single-spa.js.org/docs/recommended-setup/"
---

# single-spa：Recommended Setup

- **项目：** single-spa
- **URL：** https://single-spa.js.org/docs/recommended-setup/
- **访问日期：** 2026-08-24
- **资料类型：** 官方架构与部署指南

## 来源摘要

single-spa 官方建议使用浏览器模块与 Import Maps 组织可独立开发和部署的应用，并优先按路由划分较粗粒度应用，减少同屏应用之间的通信。每个应用通过公共入口暴露能力，Root Config 负责装配，部署流程更新线上模块映射。

指南提醒不要把所有依赖都共享：大型框架只加载一次可以节省资源，但共享依赖通常需要协调升级；小型依赖允许重复，反而能保留自治。它也指出，如果两个微前端频繁传递 UI State，应考虑合并；全局 Store 会让所有应用依赖相同状态形状和 Action，从而破坏独立部署。

## 使用边界

- 这些建议服务于 single-spa 的运行模型，不是所有微前端实现必须照搬的通用标准。
- “独立仓库”是该项目推荐的工作流，不代表 Monorepo 无法实现独立构建与部署；仓库和运行时边界仍是正交维度。
- Import Maps、Utility Modules 和 Root Config 只是落地机制，不能替代业务边界、版本契约、故障隔离和团队责任设计。
