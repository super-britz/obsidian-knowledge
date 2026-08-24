---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.sei.cmu.edu/library/architecture-tradeoff-analysis-method-collection/"
---

# CMU SEI：Architecture Tradeoff Analysis Method

- **机构：** Carnegie Mellon University Software Engineering Institute
- **URL：** https://www.sei.cmu.edu/library/architecture-tradeoff-analysis-method-collection/
- **访问日期：** 2026-08-24
- **资料类型：** 方法集合与研究报告入口

## 来源摘要

ATAM 用业务驱动因素、质量属性目标、场景和候选架构方法分析软件架构。它关注的不是某个方案是否拥有最多优点，而是性能、可修改性、安全、可用性等质量属性如何相互影响，并识别 Risk、Non-risk、Sensitivity Point 与 Tradeoff Point。

典型流程会先明确业务驱动与架构，再建立并排序质量属性场景，用高优先级场景分析架构方法，最后汇总风险主题。场景在这里既是需求澄清工具，也是评估架构选择的测试输入。

## 使用边界

- 完整 ATAM 是需要多类 Stakeholder 和专门评估过程的方法，不应机械套用到每个日常技术选择。
- 小团队可以复用“业务目标 → 质量场景 → 候选方案 → 风险与权衡”的推理骨架，但不能把轻量讨论声称为正式 ATAM 评估。
- 该方法主要暴露设计风险和权衡，生产结果仍需在实现和运行后继续验证。
