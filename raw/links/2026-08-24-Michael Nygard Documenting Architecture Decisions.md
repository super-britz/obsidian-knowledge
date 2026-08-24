---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions"
---

# Michael Nygard：Documenting Architecture Decisions

- **作者：** Michael Nygard
- **URL：** https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- **访问日期：** 2026-08-24
- **资料类型：** 原始方法文章

## 来源摘要

文章从“项目成员会遗忘旧决策的动机”出发，提出用小而独立的 Architecture Decision Record 保存架构上重要的单项决定。架构重要性包括对结构、质量属性、依赖、接口或构建方式产生持续影响。

最小 ADR 包含 Title、Context、Decision、Status 与 Consequences。Context 应中性记录当时存在且彼此拉扯的力量；Decision 用主动句说明团队将做什么；Consequences 同时记录正面、负面和中性后果。旧决定被改变时不删除，而是标记为 Superseded 并指向替代记录。

## 使用边界

- 文章原始模板追求一项决策一到两页，重点是未来读者能理解当时为什么这样决定，不是编写完整设计说明书。
- ADR 保存历史理由，不自动证明实现符合决定，也不自动证明决定改善了生产结果。
- 文章发表于 2011 年，具体状态、目录和审批流程应由团队按当前协作方式选择。
