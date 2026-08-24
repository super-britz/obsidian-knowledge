---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets"
---

# GitHub：Rulesets 的评估、执行与绕过

- **机构：** GitHub Docs
- **概览：** https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets
- **创建与执行状态：** https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository
- **可用规则：** https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets
- **访问日期：** 2026-08-24
- **资料类型：** 官方产品文档

## 来源摘要

GitHub Rulesets 可以对目标 Branch 或 Tag 应用规则，并以 `Evaluate`、`Active` 或 `Disabled` 状态管理执行：Evaluate 只记录如果启用规则会通过还是失败，不阻断贡献；Active 才实际执行；Disabled 既不执行也不评估。Required Status Checks 可以要求指定检查通过后才能合并。

Rulesets 还支持限定 Bypass 的角色、团队或 App，并可要求通过 Pull Request 留下审计路径。这展示了自动约束从观察到阻断、从普通路径到受控例外的一种具体实现。

## 使用边界

- 这是 GitHub 的仓库治理能力，具体状态、许可和可用范围会随产品版本与套餐变化。
- Required Check 只证明某个检查报告为通过；若输入、规则或检查实现错误，Gate 仍可能给出错误结论。
- Bypass 是应急与治理能力，不应成为长期逃避规则的默认路径；例外仍需限定范围、理由、责任人和期限。
