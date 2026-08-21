---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://datatracker.ietf.org/doc/html/rfc9457"
---

# RFC 9457：Problem Details for HTTP APIs

- **机构：** IETF
- **URL：** https://datatracker.ietf.org/doc/html/rfc9457
- **访问日期：** 2026-08-21
- **资料类型：** RFC（Proposed Standard）

## 来源摘要

RFC 9457 为 HTTP API 定义 `application/problem+json` 等问题详情格式。JSON 对象可包含 `type`、`status`、`title`、`detail`、`instance` 及问题类型特有的扩展成员。

其中 `type` 是问题类型的主要机器标识；`title` 和 `detail` 是面向人的说明，消费者不应解析 `detail` 来驱动程序行为。`status` 是方便消费者使用的提示值，实际 HTTP 状态仍必须正确。扩展成员可保存字段问题、余额或其他机器可用信息，客户端必须忽略不认识的扩展，以允许契约演进。

## 使用边界

- RFC 9457 统一的是 HTTP 错误载荷，不自动定义某个业务领域的错误分类、状态转换或重试安全性。
- Problem Details 本身仍是不可信外部输入，客户端需要检查媒体类型、结构和已知问题类型。
- 错误详情可能包含敏感信息；服务端与客户端仍需分别执行披露控制、脱敏和日志访问控制。
