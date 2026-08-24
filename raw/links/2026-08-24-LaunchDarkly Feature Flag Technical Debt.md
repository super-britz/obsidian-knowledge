---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://launchdarkly.com/docs/guides/flags/technical-debt"
---

# LaunchDarkly：管理 Feature Flag 技术债

- **机构：** LaunchDarkly
- **URL：** https://launchdarkly.com/docs/guides/flags/technical-debt
- **访问日期：** 2026-08-24
- **资料类型：** 官方产品工程指南

## 来源摘要

指南区分临时 Flag 与永久 Flag。Release、Experiment、Interoperability 等 Flag 通常服务于一次有限迁移；Entitlement、Operational 等 Flag 可能长期存在，但仍要有明确用途和所有者。

指南把 Flag 视为有生命周期的代码资产：创建时声明目的，发布期间观察，达到稳定终态后先删除代码中的旧分支和求值调用，再归档配置。仅把流量调到 100% 或长期关闭并不等于完成清理；遗留 Flag 会增加分支组合、测试范围和理解成本。清理责任应进入交付完成条件，Flag 的作用域也应尽量受限。

## 使用边界

- 临时与永久的分类、生命周期阶段和具体工具操作来自 LaunchDarkly 的产品模型，可迁移的是所有权、退出条件和债务控制思想。
- 删除 Flag 前需要证明仍在运行的旧代码不再求值它；只看控制台的当前流量比例不够。
- 永久 Flag 也不是无需治理，仍要检查权限、默认值、故障降级和配置变更审计。
