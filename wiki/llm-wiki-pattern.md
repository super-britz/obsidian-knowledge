---
type: 概念
status: 草稿
created: 2026-06-04
updated: 2026-06-04
sources:
  - raw/karpathy-llm-wiki.md
---

# LLM Wiki 模式

## 核心观点

LLM Wiki 的核心不是把资料上传给模型做一次性问答，而是让 LLM 持续维护一个可以积累的 Markdown wiki。每次加入新资料时，LLM 都要把新信息整合进已有页面、更新索引、补充交叉引用，并记录知识库如何演化。

## 三层结构

### 原始来源层

`raw/` 保存原始资料，是知识库的事实来源。LLM 可以读取它，但默认不改写它。文章、论文、书摘、截图、图片和数据文件都属于这一层。

### Wiki 层

`wiki/` 保存经过整理的长期知识页面。这里的页面由 LLM 维护，包括概念页、实体页、综述、比较、问题回答和综合分析。

### 维护 Schema

`AGENTS.md` 是这个仓库的维护 schema。它告诉后续的 Codex 或其他 LLM agent：目录怎么用、页面怎么写、摄取怎么执行、查询怎么回答、健康检查怎么做。

## 三种操作

### 摄取

把新资料加入 `raw/` 后，让 LLM 读取并整合它。一次摄取通常会创建或更新多个 wiki 页面，并更新 `wiki/index.md` 和 `wiki/log.md`。

### 查询

针对知识库提问时，LLM 应先读 `wiki/index.md`，再读相关 wiki 页面和必要原始来源。高价值回答可以沉淀为新的 wiki 页面。

### 健康检查

周期性检查知识库健康状况，包括矛盾、过期说法、孤立页面、缺失链接、重要概念缺页、索引未更新等问题。

## 为什么适合 Obsidian

Obsidian 适合浏览和编辑 Markdown wiki：可以使用 wiki links、图谱视图、本地附件和 git 版本管理。人在 Obsidian 中阅读和判断方向，LLM 负责整理、链接、更新和记录。

## 当前实现选择

这个 vault 采用最小实现：

- `raw/` 保存来源和附件。
- `wiki/index.md` 保存内容索引。
- `wiki/log.md` 保存追加式日志。
- `wiki/*.md` 保存长期知识页。
- `AGENTS.md` 保存维护规则。

暂不默认加入 PARA、Dataview、Marp 或本地搜索工具；这些可以在知识库规模变大后再添加。

## 相关链接

- [[raw/karpathy-llm-wiki|Karpathy: LLM Wiki]]
- [[index]]
- [[log]]
