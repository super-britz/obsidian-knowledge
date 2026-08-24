---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html"
---

# AWS：Transactional Outbox Pattern

- **机构：** AWS Prescriptive Guidance
- **URL：** https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html
- **访问日期：** 2026-08-24
- **资料类型：** 官方架构模式文档

## 来源摘要

文档把“双写”问题描述为：一个操作既要修改数据库，又要向消息系统或另一个服务发送事件时，两个独立写入无法共享原子事务，任一步骤失败都可能让业务状态与事件不一致。

Transactional Outbox 让业务数据变更与待发送事件在同一个数据库事务中持久化，再由独立 Relay 将 Outbox 记录发送到目标系统。这样可以避免“业务提交但事件丢失”或“事件发出但业务回滚”的窗口。Relay 仍可能重复发送，因此消费者需要幂等；事件顺序对业务重要时也必须显式维护。

## 使用边界

- Outbox 解决的是本地事务与跨系统通知之间的可靠交接，不会让数据库和外部系统形成一个同步原子事务。
- Relay 延迟、重复投递、积压、顺序、清理和监控都会成为新的运行责任。
- 若双写发生在同一事务边界内，可以直接使用事务；若目标不支持消息中继，还需评估 CDC、对账或补偿等替代机制。
