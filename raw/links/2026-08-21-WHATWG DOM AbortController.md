---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://dom.spec.whatwg.org/#aborting-ongoing-activities"
---

# WHATWG DOM：AbortController 与 AbortSignal

- **机构：** WHATWG
- **URL：** https://dom.spec.whatwg.org/#aborting-ongoing-activities
- **访问日期：** 2026-08-21
- **资料类型：** Web 平台标准

## 来源摘要

DOM Standard 说明 Promise 本身没有内建中止机制。需要支持中止语义的 API 可以接收 `AbortSignal`，观察其状态，并在 `AbortController.abort()` 改变信号后执行自身的中止步骤；规范鼓励相关 API 用信号的原因拒绝尚未完成的 Promise。

因此 `AbortController` 提供的是协作式中止协议：控制方发出信号，实际工作是否以及怎样停止，由接收信号的 API 实现。

## 使用边界

- 中止本地等待或网络活动不等于撤销服务端已经执行的业务副作用。
- 并非所有异步 API 都接受或正确传播 `AbortSignal`，需要核对具体 API 契约。
- 中止可能与完成事件竞态，因此调用方仍需用请求身份判断迟到结果能否生效。
