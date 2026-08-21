---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://fetch.spec.whatwg.org/"
---

# WHATWG Fetch Standard

- **机构：** WHATWG
- **URL：** https://fetch.spec.whatwg.org/
- **访问日期：** 2026-08-21
- **资料类型：** Web Living Standard

## 来源摘要

Fetch Standard 分开建模网络错误与带 HTTP 状态的响应。规范示例也分别处理 `network error` 和响应状态不是 `ok status` 的情况；因此得到一个 Fetch 响应不等于业务成功，而网络失败也没有提供一个可直接当成业务拒绝使用的 HTTP 错误体。

对前端错误模型而言，请求层至少要分别保留：是否获得响应、HTTP 状态、响应体能否解析、响应体是否符合契约。之后才能由具体用例判断是重试、提示、刷新、降级，还是进入“写操作结果未知”。

## 使用边界

- Fetch 规范定义浏览器网络获取机制，不负责解释某个 HTTP 状态对应的业务含义。
- 浏览器暴露的网络错误可能合并离线、连接、CORS 或策略阻断等多种原因；应用不能伪造自己实际上并不知道的根因。
- 请求重放是否安全取决于具体操作的幂等性、操作身份和服务端状态，而不是 Fetch 异常的名称。
