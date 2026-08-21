---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html"
---

# TypeScript Erased Types

- **机构：** TypeScript 官方文档
- **URL：** https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html
- **访问日期：** 2026-08-21
- **资料类型：** 官方语言手册

## 来源摘要

TypeScript 是静态类型检查器。编译器完成检查后会擦除类型，生成的 JavaScript 不携带这些类型信息；类型系统本身不会在运行时根据声明改变程序行为。

因此，为网络响应、URL、本地存储或第三方消息写出 TypeScript 类型，只能描述开发期预期，不能证明运行时值符合该类型。外部值仍需要实际解析和校验。

## 使用边界

- TypeScript 仍能显著减少受控代码内部的类型错误；本页结论不是否定静态类型的价值。
- 类、枚举等少数构造可能有运行时产物，但普通类型声明和类型断言不能替代运行时校验。
- 具体 Schema 工具选型不属于本来源。
