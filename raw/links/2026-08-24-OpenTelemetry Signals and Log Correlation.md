---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://opentelemetry.io/docs/concepts/signals/"
---

# OpenTelemetry：Signals 与 Log Correlation

- **机构：** OpenTelemetry
- **Signals：** https://opentelemetry.io/docs/concepts/signals/
- **Log Correlation：** https://opentelemetry.io/docs/specs/otel/logs/#log-correlation
- **访问日期：** 2026-08-24
- **资料类型：** 官方概念与规范文档

## 来源摘要

OpenTelemetry 将 Traces、Metrics、Logs 和 Baggage 区分为不同遥测信号：Trace 描述请求经过应用的路径，Metric 是运行时测量，Log 是事件记录，Baggage 是随信号传递的上下文。不同信号从不同角度描述同一系统活动。

其日志规范说明，日志可以按时间、执行上下文和资源来源与其他遥测关联；在 LogRecord 中携带 TraceId 与 SpanId，可以把同一执行上下文中的日志和追踪直接关联，也能连接参与一次请求的不同系统组件。

## 使用边界

- 遥测信号提供观察材料，不会自动给出根因；字段、采样、关联和查询仍需围绕真实诊断问题设计。
- 浏览器关闭、离线、拦截或上报链路自身失败会造成客户端证据缺失，不能把“没有日志”直接解释为“没有失败”。
- 记录能力必须受隐私、敏感数据、成本和保留期限约束。
