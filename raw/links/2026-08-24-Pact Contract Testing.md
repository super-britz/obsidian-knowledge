---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://docs.pact.io/getting_started/how_pact_works"
---

# Pact：Consumer-driven Contract Testing

- **机构：** Pact
- **URL：** https://docs.pact.io/getting_started/how_pact_works
- **访问日期：** 2026-08-24
- **资料类型：** 官方概念文档

## 来源摘要

Pact 把调用方称为 Consumer，把提供功能或数据的一方称为 Provider。一个 Pact 由双方交互组成：对 HTTP 而言包含预期请求，以及 Consumer 真正依赖的最小响应；Consumer Test 生成契约，Provider Verification 再向 Provider 重放请求并核对实际响应。

这种方式可以在不同时启动完整系统的情况下，持续检查 Provider 是否仍满足已知 Consumer 的交互要求。它把“公共契约保持兼容”的一部分意图转成可在交付链执行的反馈。

## 使用边界

- Contract Test 只覆盖已经表达的交互和 Consumer 依赖，不证明 API 的全部行为、业务正确性、安全性或性能。
- Mock 使用不当、Provider State 不真实或契约版本未映射到实际部署版本，都会制造错误信心。
- 不是所有共享 Schema 都需要 Pact；同进程模块、简单单体或由其他兼容性检查充分覆盖的边界应选择更小机制。
