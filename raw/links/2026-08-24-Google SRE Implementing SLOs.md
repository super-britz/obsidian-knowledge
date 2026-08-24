---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://sre.google/workbook/implementing-slos/"
---

# Google SRE：Implementing SLOs

- **机构：** Google SRE
- **URL：** https://sre.google/workbook/implementing-slos/
- **访问日期：** 2026-08-24
- **资料类型：** 官方工程实践章节

## 来源摘要

该章节把 SLO 定义为服务可靠性的目标水平，并强调指标只有进入共同认可的决策流程才“有牙齿”。要使用 Error Budget，产品、开发和运行责任方需要认可 SLO 可达且适合用户，并用书面 Policy 说明预算耗尽时采取什么动作、由谁执行以及发生争议时怎样升级。

文档还要求记录指标实现、阈值理由、审批与复审时间，并持续迭代 SLI、SLO 和 Policy。由此可见，运行时测量本身不是约束；只有“信号 → 阈值 → 动作 → 责任人 → 复审”闭合，指标才会影响系统演进。

## 使用边界

- SLO 与 Error Budget 主要治理用户可感知的可靠性，不替代源码依赖、类型、契约和安全等更早期检查。
- 错误 SLI、样本偏差或不合理目标会驱动错误动作；阈值必须来自产品风险、用户结果和可达能力，而不是照抄示例。
- 冻结发布只是可能动作之一，不是所有团队、故障来源和业务阶段都适用的通用规则。
