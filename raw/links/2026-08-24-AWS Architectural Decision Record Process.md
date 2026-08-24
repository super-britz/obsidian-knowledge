---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html"
---

# AWS：Architectural Decision Record Process

- **机构：** Amazon Web Services
- **URL：** https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html
- **访问日期：** 2026-08-24
- **资料类型：** Prescriptive Guidance

## 来源摘要

AWS 将 ADR 定义为描述软件架构重要选择、上下文和后果的文档，并把多份 ADR 组成的集合称为 Decision Log。适合记录的范围包括系统结构、质量属性、依赖、公开接口以及库、框架和流程等构建技术。

其建议流程包含 Owner、Proposed、Review、Accepted 或 Rejected 等状态。Accepted 或 Rejected 后的记录作为历史保持不变；新认识要求改变决定时创建新 ADR，并把旧记录标记为 Superseded。代码评审可以引用 ADR 检查实现是否违背已接受决定。

## 使用边界

- 这是通用项目指南，不要求照搬具体会议时长、审批人数或状态名称。
- “Accepted”表示团队已经授权某项决定，不表示该决定的业务或运行结果已经验证成功。
- ADR 的不可变性针对历史语义；错误链接、排版等非语义修正是否允许，应由团队规则明确。
