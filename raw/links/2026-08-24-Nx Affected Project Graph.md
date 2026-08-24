---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://nx.dev/docs/features/ci-features/affected"
---

# Nx：Affected 与 Project Graph

- **机构：** Nx
- **URL：** https://nx.dev/docs/features/ci-features/affected
- **补充资料：** https://nx.dev/docs/features/explore-graph
- **访问日期：** 2026-08-24
- **资料类型：** 官方工程工具文档

## 来源摘要

Nx 把 Workspace 描述为多个 Project 构成的 Project Graph，并另外构造包含任务依赖与执行顺序的 Task Graph。`affected` 机制结合 Git 变更范围与 Project Graph，找出直接改变的项目及其下游依赖者，只对这部分运行指定任务。

文档指出，Lockfile 改变默认会把全部项目标为受影响，以避免依赖解析变化被漏判；“只验证受影响项目”因此依赖图和输入建模的完整性，不是简单按改动目录过滤。

## 使用边界

- Project Graph、Task Graph 与 `affected` 是 Nx 的具体实现；其他工具可以采用不同配置或算法实现相同原则。
- 图中没有声明的动态依赖、环境变量、生成文件和外部服务可能造成错误漏判，需要把全局输入与例外显式纳入。
- Affected Pruning 降低日常 CI 成本，不应替代关键契约测试、周期性全量验证或发布后的运行证据。
