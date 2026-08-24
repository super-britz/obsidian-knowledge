---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://pnpm.io/workspaces"
---

# pnpm：Workspace

- **机构：** pnpm
- **URL：** https://pnpm.io/workspaces
- **访问日期：** 2026-08-24
- **资料类型：** 官方工具文档

## 来源摘要

pnpm Workspace 可以在一个仓库中联合管理多个项目。`workspace:` 协议要求依赖解析到本地 Workspace Package；当指定的本地版本不存在时安装失败，避免悄悄改从 Registry 获取另一个版本。Package 发布时，pnpm 会把 Workspace 依赖转换为普通版本范围，使其能被仓库外消费者安装。

文档同时指出，多 Package 的版本与发布本身较复杂，pnpm 不内置完整版本管理方案；Workspace 依赖出现循环时，脚本执行顺序也不能得到可靠的拓扑保证。

## 使用边界

- Workspace 解决本地 Package 发现、安装和链接，不负责决定正确业务边界、公共 API、任务图、版本策略或部署顺序。
- 本地源码可以直接联调，不等于发布产物、`exports`、依赖声明和外部安装路径已经验证。
- 一个共享 Lockfile 能统一解析快照，但不表示所有 Package 必须同版本或同时发布。
