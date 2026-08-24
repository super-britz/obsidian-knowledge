---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.thoughtworks.com/en-au/insights/articles/fitness-function-driven-development"
---

# Thoughtworks：Fitness Function-Driven Development

- **机构：** Thoughtworks
- **URL：** https://www.thoughtworks.com/en-au/insights/articles/fitness-function-driven-development
- **访问日期：** 2026-08-24
- **资料类型：** 工程实践文章

## 来源摘要

文章把 Architecture Fitness Function 描述为衡量架构接近某个目标程度的反馈机制。它从 Stakeholder 真正重视的 Resilience、Operability、Stability 等质量出发，尝试用有客观含义的指标、测试或检查持续发现架构漂移。

Fitness Function 可以位于代码、构建流水线、环境验证或生产监控中，覆盖结构、日志与追踪、性能、韧性、安全、合规和可运维性。其价值在于让关键架构意图在变化发生时得到持续反馈，而不是只在事后由人工回忆。

## 使用边界

- 能自动化的通常只是架构质量的一部分；可用性、团队协作和复杂用户结果仍需要定性与生产证据。
- 指标和阈值本身也会老化并产生维护成本，错误代理可能制造僵化或诱导局部优化。
- Fitness Function 检查架构特征，不替代 ADR 对上下文、选项、权衡和退出条件的说明。
