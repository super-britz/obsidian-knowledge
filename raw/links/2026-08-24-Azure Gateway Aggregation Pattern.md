---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://learn.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation"
---

# Azure Architecture Center：Gateway Aggregation Pattern

- **机构：** Microsoft Azure Architecture Center
- **URL：** https://learn.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation
- **访问日期：** 2026-08-24
- **资料类型：** 官方架构模式文档

## 来源摘要

文档将 Gateway Aggregation 描述为：客户端只发出一个请求，聚合层再调用多个后端系统、组合结果并返回，以减少高延迟网络中的往返和客户端编排工作。适用场景是一个用户操作确实依赖多个后端；若只是反复调用同一个服务，批量接口可能更简单。

聚合层也可能成为单点故障和瓶颈，并把一个客户端请求放大为多个内部调用。文档要求考虑容量、负载测试、超时、重试、熔断、部分结果、缓存和分布式追踪；复杂领域逻辑或长流程编排不应塞进轻量网关策略，而应进入专门服务。

## 使用边界

- 减少客户端请求数不等于一定降低端到端延迟；结果仍由内部关键路径、额外跳转和排队决定。
- 返回部分数据是否有意义必须由具体用户任务定义，不能把缺失字段静默伪装成完整成功。
- Gateway Aggregation 是一种聚合机制；它可以被 BFF 使用，但并不自动形成客户端所有权边界。
