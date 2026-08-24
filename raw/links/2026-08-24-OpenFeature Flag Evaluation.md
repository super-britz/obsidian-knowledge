---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://openfeature.dev/specification/sections/flag-evaluation/"
---

# OpenFeature：Flag Evaluation

- **机构：** OpenFeature
- **URL：** https://openfeature.dev/specification/sections/flag-evaluation/
- **访问日期：** 2026-08-24
- **资料类型：** 官方规范

## 来源摘要

OpenFeature 的 Flag Evaluation 规范把一次求值描述为：客户端提供类型化 Flag Key、同类型默认值，以及可选的 Evaluation Context 与求值选项；提供方返回解析后的值。详细结果还可以携带 Variant、Reason、Error Code 与 Error Message，供诊断和观测使用。

规范要求异常求值返回调用方声明的默认值，并在详细结果中报告错误，而不是把求值异常直接抛给业务调用方。类型不匹配、Flag 不存在、提供方未就绪或一般错误都属于需要显式处理的解析结果。

## 使用边界

- 规范统一的是求值接口和错误语义，不定义哪个默认值对当前业务、数据版本或安全边界是正确的。
- Evaluation Context 可能包含身份和属性；采集、传输与记录仍需遵守最小化和隐私规则。
- Feature Flag 决定代码走哪条路径，不是权限控制；服务端仍必须独立执行授权和业务不变量。
