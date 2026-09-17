---
type: source-note
title: commit-and-tag-version
author: absolute-version
url: https://github.com/absolute-version/commit-and-tag-version
accessed: 2026-09-11
status: 已链接
---

# commit-and-tag-version

`commit-and-tag-version` 是停止维护的 `standard-version` 的社区延续，用 Conventional Commits 和 SemVer 在本地完成版本号计算、版本文件更新、CHANGELOG 生成、release commit 与 Git tag 创建。

## 来源

- 项目与 README：https://github.com/absolute-version/commit-and-tag-version
- 版本记录：https://github.com/absolute-version/commit-and-tag-version/blob/master/CHANGELOG.md
- v13 迁移说明：https://github.com/absolute-version/commit-and-tag-version/blob/master/MIGRATION.md
- 原项目：https://github.com/conventional-changelog/standard-version
- Conventional Commits：https://www.conventionalcommits.org/zh-hans/v1.0.0/
- Semantic Versioning：https://semver.org/lang/zh-CN/

## 来源摘要

截至 2026-09-11，项目当前版本为 `13.2.0`。当前版本要求 Node.js 22 或更高，并已转为纯 ESM；CLI 用户通常不受 ESM 变化影响，但编程式调用、自定义 preset 和自定义 CHANGELOG writer 需要按迁移文档检查兼容性。

工具读取版本文件或最近 Git tag，根据上次发布后的 Conventional Commits 计算下一个版本，然后更新版本文件和 CHANGELOG，创建 release commit 与 tag。它不会自动 push、创建 GitHub Release 或发布 npm 包。

相对 `standard-version@9.5.0`，该分支持续升级 conventional-changelog 依赖，增加 Maven、Gradle、.NET、YAML、OpenAPI 与 Poetry 等版本文件支持，并补充 `--config`、`--noBumpWhenEmptyChanges` 等能力。

## 本地解读

机制、使用流程和工程边界整理见 [[wiki/commit-and-tag-version-基于提交语义的本地发布流水线|commit-and-tag-version：基于提交语义的本地发布流水线]]。
