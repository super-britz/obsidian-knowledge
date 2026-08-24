---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.openpolicyagent.org/docs"
---

# Open Policy Agent：Policy as Code

- **机构：** Open Policy Agent
- **概览：** https://www.openpolicyagent.org/docs
- **集成说明：** https://www.openpolicyagent.org/docs/integration
- **访问日期：** 2026-08-24
- **资料类型：** 官方概念与集成文档

## 来源摘要

OPA 是通用策略引擎，使用声明式策略和结构化输入产生决策。其集成模型把策略的决策责任与业务系统中的执行责任分开：应用或工具提交输入，OPA 根据策略和数据返回结果，调用方再决定怎样执行；管理接口还可以分发策略、查看状态和记录决策。

这套模型说明一条可执行规则至少涉及策略、输入、决策点和执行点，且决策结果需要留下可审计证据。它可以运行在服务、API Gateway、CI/CD 等不同位置，但具体位置取决于延迟、可用性和信任边界。

## 使用边界

- OPA 能执行已经形式化的策略，不能发现正确的业务边界或替代 ADR 中的上下文与权衡。
- 把策略决策放到远程链路会新增延迟和可用性依赖；是否本地嵌入、旁路运行或集中调用必须按风险选择。
- 策略输入和决策日志可能含有身份、权限或配置等敏感信息，需要最小化、脱敏和访问控制。
