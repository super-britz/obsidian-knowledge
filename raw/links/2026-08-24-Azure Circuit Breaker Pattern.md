---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker"
---

# Azure Architecture Center：Circuit Breaker Pattern

- **机构：** Microsoft Azure Architecture Center
- **URL：** https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker
- **访问日期：** 2026-08-24
- **资料类型：** 官方架构模式文档

## 来源摘要

Circuit Breaker 通过 Closed、Open 和 Half-Open 状态控制对易失败依赖的调用。Closed 时正常放行并观察近期失败；达到阈值后进入 Open，快速拒绝后续调用；等待恢复窗口后进入 Half-Open，只放行有限探测，再根据结果回到 Closed 或 Open。

官方文档将它与 Retry 区分：Retry 假定暂时故障最终可能成功，Circuit Breaker 则阻止继续调用当前很可能失败的依赖。文档也强调阈值、恢复节奏、并发、资源粒度、错误类型和可观测性都需要设计；粒度过粗可能把健康分片一起阻断，恢复探测过快或过慢也会造成新的失败。

## 使用边界

- 熔断器是带共享状态和阈值的资源保护机制，不替代业务错误处理、超时或重复安全契约。
- 对本地内存等简单资源，或已有平台机制充分治理的场景，额外熔断可能只有复杂度。
- Open 时返回缓存或默认值仍要明确新鲜度、功能缺失和用户可采取的动作。
