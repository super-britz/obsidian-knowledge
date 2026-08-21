---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html"
---

# OWASP Input Validation Cheat Sheet

- **机构：** OWASP Cheat Sheet Series
- **URL：** https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- **访问日期：** 2026-08-21
- **资料类型：** 官方安全实践指南

## 来源摘要

该指南说明输入校验应尽早发生在外部数据进入系统的位置，并同时覆盖结构或语法正确性与具体业务语境中的语义正确性。潜在不可信来源不只包括互联网客户端，也包括供应商、合作伙伴和其他后端数据源。

客户端 JavaScript 校验可以改善用户体验，但能够被绕过；承担安全责任的输入仍需在服务端进入应用功能前进行校验。

## 使用边界

- 输入校验是纵深防御的一部分，不单独替代输出编码、参数化查询、授权和其他专项安全控制。
- 具体字段、文件、富文本和协议需要结合相应威胁模型及 OWASP 专项指南设计。
- 本页只记录与可信边界相关的原则，不复制完整检查清单。
