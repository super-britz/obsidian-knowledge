---
type: 概念
status: 稳定
created: 2026-09-11
updated: 2026-09-11
sources:
  - raw/links/2026-09-11-commit-and-tag-version.md
---

# commit-and-tag-version：基于提交语义的本地发布流水线

## 核心结论

`commit-and-tag-version` 不是远程发布平台，而是一个本地状态转换器：它把 Git 提交中的变更语义转换为一个可审查的版本发布状态。

```text
当前版本 + 上次 tag 后的提交 + 发布配置
                  ↓
             计算下一个 SemVer
                  ↓
版本文件 + CHANGELOG + release commit + Git tag
```

它解决的根本问题不是“怎样执行一条发版命令”，而是让版本号、变更记录、代码快照和 Git tag 指向同一个事实。它不会自动 push、创建 GitHub Release 或发布 npm 包，这些属于后续交付阶段。

## 第一性原理：为什么需要它

一个可追踪的软件版本至少要同时回答四个问题：

1. **版本身份**：这次发布叫什么，例如 `v1.4.0`。
2. **代码边界**：这个版本对应哪个不可变 Git 快照。
3. **变化原因**：相对上个版本增加、修复或破坏了什么。
4. **机器状态**：`package.json` 等版本文件是否与 tag 一致。

手工维护时，这四份状态可以独立修改，因此容易出现“版本文件是 `1.4.0`，tag 是 `v1.3.1`，CHANGELOG 又漏了一项”的漂移。`commit-and-tag-version` 把它们收敛到一次确定性操作中：

```text
Conventional Commits 提供变化语义
→ SemVer 把变化语义映射为兼容性等级
→ 工具原子化地产生版本文件、说明、提交和 tag
→ 人检查后再决定是否推送和发布
```

因此，真正的前置条件不是安装工具，而是团队先把提交信息当成发布数据维护。

## 版本怎样计算

默认规则来自 Conventional Commits：

| 提交 | 含义 | 版本变化 |
| --- | --- | --- |
| `fix: 修复登录失败` | 向后兼容的缺陷修复 | patch：`1.2.3 → 1.2.4` |
| `feat: 增加短信登录` | 向后兼容的新能力 | minor：`1.2.3 → 1.3.0` |
| `feat!: 重构登录接口` | 不兼容变化 | major：`1.2.3 → 2.0.0` |
| `docs:`、`test:`、`chore:` | 通常不代表用户可感知发布 | 不应单独决定版本 |

一次发布包含多个提交时，取兼容性影响最大的等级。历史默认行为可能在没有有效发布提交时仍给出 patch；需要“无有效变化就不发版”时，应开启 `noBumpWhenEmptyChanges`。

## 最小可用配置

当前 `13.x` 要求 Node.js 22 或更高。对普通 Node.js 项目，先安装本地开发依赖：

```bash
npm install --save-dev commit-and-tag-version
```

在 `package.json` 中建立三个入口：

```json
{
  "scripts": {
    "release": "commit-and-tag-version --noBumpWhenEmptyChanges",
    "release:dry": "commit-and-tag-version --dry-run --noBumpWhenEmptyChanges",
    "release:first": "commit-and-tag-version --first-release"
  },
  "commit-and-tag-version": {
    "releaseCommitMessageFormat": "chore(release): {{currentTag}}"
  }
}
```

把工具装在项目本地而不是全局，可以让版本和配置随仓库固定，减少不同开发者机器上的行为差异。

## 标准发布流程

### 1. 开发阶段维护提交语义

```bash
git commit -m "feat(search): 增加关键词高亮"
git commit -m "fix(search): 修复空结果状态"
```

### 2. 发布前预演

```bash
npm run release:dry
```

预演用于检查下一个版本和 CHANGELOG，不修改文件、不提交、不打 tag。正式发布前还应保证测试通过、工作区干净，并确认当前分支和远端状态正确。

### 3. 在本地生成发布状态

```bash
npm run release
```

工具随后更新版本文件和 `CHANGELOG.md`，创建 release commit 与类似 `v1.3.0` 的 tag。

### 4. 人工检查并交付远端

```bash
git show --stat HEAD
git tag --points-at HEAD
git push --follow-tags origin main
```

如果项目还需要 GitHub Release、npm publish 或部署，应由后续 CI/CD 或人工步骤完成。将“生成发布状态”和“把状态交付到远端”分开，可以在 push 前保留最后一个审查与回滚点。

## 首次发布、指定版本与预发布

如果版本文件已经写好首次版本，例如 `1.0.0`：

```bash
npm run release:first
```

强制指定准确版本或升级等级：

```bash
npm run release -- --release-as 1.0.0
npm run release -- --release-as minor
```

创建预发布版本：

```bash
npm run release -- --prerelease alpha
```

可能产生 `1.3.0-alpha.0`。这些参数是对自动决策的显式覆盖，不应成为日常替代提交语义的手段。

## 相对 standard-version 改变了什么

`commit-and-tag-version` 延续了 `standard-version` 的核心流水线，主要改变发生在边界层：

- **维护性**：继续升级 Node.js 与 conventional-changelog 生态，修复 tag、prerelease 和版本文件处理问题。
- **文件适配**：内置支持 Maven、Gradle、`.csproj`、YAML、OpenAPI 和 Poetry，不再只围绕 Node.js 版本文件。
- **控制能力**：增加自定义配置路径、无有效变化时不升级等选项。
- **现代运行时**：当前版本要求 Node.js 22，并采用纯 ESM。
- **深度定制接口**：非默认 preset 需要单独安装；自定义 writer 模板需要使用现代 render function。

没有改变的是发布哲学：它仍然是人工触发、本地生成、允许审查的发布工具，而不是合并代码后自动把产物推到生产环境的无人值守系统。

## 适用边界

适合使用：

- 项目有明确的 SemVer 版本和发布物。
- 团队愿意维护 Conventional Commits。
- 希望本地审查版本、CHANGELOG、commit 和 tag 后再 push。
- 需要兼容 Node.js 之外的常见版本文件。

不应优先使用：

- 希望 GitHub 自动维护 Release PR：考虑 `release-please`。
- 希望合并后自动发布 npm 和生成远端 Release：考虑 `semantic-release`。
- 仓库只是没有独立发布物的内容集合：Git 历史本身可能已经足够，不必为了形式增加版本工具。
- 无法升级到 Node.js 22：需要先规划运行时升级，或评估兼容的旧主版本，不能直接安装当前最新版。

## 对本知识库的工程判断

这个 Obsidian 仓库的主要产物是持续演化的 Markdown 知识，而不是提供兼容性承诺的软件包、CLI 或 API。它当前没有必须同步维护的包版本和发布物，因此没有必要仅为了“看起来规范”而安装 `commit-and-tag-version`。

这里应保存的是可迁移的发布机制知识；如果将来知识库开始发布带版本的公开快照、主题包或插件，再引入该工具才有明确收益。

## 来源

- [[raw/links/2026-09-11-commit-and-tag-version|commit-and-tag-version 官方资料说明]]
