---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://nx.dev/docs/features/enforce-module-boundaries"
---

# Nx Enforce Module Boundaries

- **机构：** Nx 官方文档
- **URL：** https://nx.dev/docs/features/enforce-module-boundaries
- **访问日期：** 2026-08-21
- **资料类型：** 官方工程工具文档

## 来源摘要

Nx 支持为项目添加标签并声明依赖约束。JavaScript/TypeScript 项目可以通过 ESLint 规则检查源码导入和 `package.json` 依赖；完整项目图还可以通过 Nx 的 Conformance 能力检查，从而把允许的依赖关系变成自动化约束。

这些规则能够阻止已声明的越界依赖，但规则本身不会发现正确的业务边界。标签、允许方向和例外仍需由团队根据真实变化流与所有权设计。

## 使用边界

- 这是 Nx 工作区的具体实现，不是建立模块边界的前置条件；普通项目也可使用其他 lint、依赖图或构建规则。
- 自动检查只能执行已形式化的规则，错误的标签或方向会把错误架构固化下来。
- Conformance 等具体能力可能受产品版本或许可限制，采用前应重新核对官方文档。
