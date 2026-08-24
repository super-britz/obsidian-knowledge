---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://html.spec.whatwg.org/multipage/web-messaging.html#crossDocumentMessages"
---

# WHATWG HTML：Cross-document Messaging

- **机构：** WHATWG
- **URL：** https://html.spec.whatwg.org/multipage/web-messaging.html#crossDocumentMessages
- **访问日期：** 2026-08-24
- **资料类型：** HTML Living Standard

## 来源摘要

HTML 标准定义了通过 `postMessage` 在不同 Document 间传递结构化消息的机制。发送方可以指定 `targetOrigin`；接收方应检查 `MessageEvent.origin`，并在信任来源后继续校验消息数据的预期格式。

规范提醒，不应对含敏感信息的消息使用通配 `targetOrigin`，否则无法保证只交付给预期接收方；接收任意来源消息还可能被利用放大计算或网络请求，因此需要限制来源、Schema、能力和频率。

## 使用边界

- `postMessage` 只提供跨 Document 传输通道，不自动提供业务事件 Schema、请求响应关联、幂等或授权。
- Origin 正确不代表消息语义可信；来源页自身若被攻陷，仍可能发送恶意数据。
- 事件总线和消息封装不能消除耦合，只会改变耦合的可见形式。
