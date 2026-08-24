---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.w3.org/TR/trace-context/"
---

# W3C Trace Context

- **机构：** W3C
- **URL：** https://www.w3.org/TR/trace-context/
- **访问日期：** 2026-08-24
- **资料类型：** W3C Recommendation

## 来源摘要

Trace Context 规定 `traceparent` 与 `tracestate` HTTP Header，使不同追踪实现可以传播同一分布式执行上下文。`traceparent` 以通用格式标识当前请求在追踪图中的位置，包含版本、Trace ID、Parent ID 与 Flags；`tracestate` 可携带供应商相关的追踪信息。

规范要求参与传播的系统正确处理这些 Header，并专门讨论隐私风险：Trace Context 字段的目的只是追踪关联，不应包含个人身份信息或其他敏感信息。

## 使用边界

- Trace ID 标识一次执行路径，不等同于业务 Operation ID、幂等键、用户身份或订单号。
- 追踪上下文跨越信任边界时仍需校验、限制和隐私评估，不能把任意上游字段原样当成可信业务信息。
- 采样或链路中断可能造成不完整 Trace；关联能力不等于每次执行都有完整记录。
