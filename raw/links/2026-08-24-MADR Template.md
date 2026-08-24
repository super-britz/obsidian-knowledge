---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://adr.github.io/madr/"
---

# MADR：Markdown Architectural Decision Records

- **项目：** MADR
- **URL：** https://adr.github.io/madr/
- **访问日期：** 2026-08-24
- **资料类型：** 开源 ADR 模板与说明

## 来源摘要

MADR 用 Markdown 结构化记录有架构意义的单项决定。完整模板在基础的 Context、Decision 和 Consequences 之外，还提供 Decision Drivers、Considered Options、各方案 Pros / Cons、Decision Outcome、Confirmation、Decision Makers、Consulted 和 Informed 等字段。

其中 Confirmation 用来说明如何通过设计或代码评审、测试等方法确认实现与决定一致；More Information 可以记录证据、复审条件和关联决策。项目建议让记录易于编写、版本化，并可放在与项目文档相邻的 `decisions/` 目录。

## 使用边界

- 完整模板中的许多字段是可选的；低风险决定不必为了填满模板制造无价值文本。
- Confirmation 主要检查实现是否符合决定，不能替代对用户结果、运行质量和副作用的验证。
- 目录、编号和分类方式属于团队元决策，不是 MADR 强制的架构规则。
