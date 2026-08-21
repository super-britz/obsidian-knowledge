---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://martinfowler.com/eaaCatalog/repository.html"
---

# Fowler Catalog：Repository Pattern

- **模式作者：** Edward Hieatt、Rob Mee
- **收录：** Martin Fowler 的 Patterns of Enterprise Application Architecture 目录
- **URL：** https://martinfowler.com/eaaCatalog/repository.html
- **访问日期：** 2026-08-21
- **资料类型：** 企业应用架构模式目录

## 来源摘要

Repository 模式在领域层和数据映射层之间提供类似集合的领域对象访问接口。它集中查询构造和持久化映射，使调用方像操作内存中的领域对象集合一样查询、添加或移除对象，同时保持领域与数据访问之间的单向依赖。

原条目指出，当系统具有复杂领域模型、较多领域类或繁重查询时，这层抽象更有价值，因为它可以隔离数据库细节并减少重复查询逻辑。

## 使用边界

- Repository 有领域集合与持久化语义，不是任意 HTTP 请求封装的通用名称。
- 前端可以让 Repository 由 HTTP、IndexedDB 或组合数据源实现，但必须保留权威来源、缓存和一致性语义。
- 简单命令式外部能力使用 Gateway 或特定 Port 往往比伪造集合语义更清楚。
