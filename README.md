# Obsidian 知识库

这是一个用于维护个人知识库的 Obsidian vault，由 LLM 持续整理 wiki 页面。

这个 vault 的初始结构来自 Andrej Karpathy 的 LLM Wiki 模式：

- 来源：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- 核心想法：保持原始资料不可变，让 LLM 维护一个可持续积累的 Markdown wiki，并用 schema 文件定义维护规则。

## 目录结构

```text
raw/        原始资料、链接说明、剪藏、文件和本地附件。
wiki/       由 LLM 维护的 Markdown wiki 页面。
AGENTS.md   Codex 和其他 agent 的维护规则。
```

特殊 wiki 文件：

- `wiki/index.md`：按内容组织的索引。
- `wiki/log.md`：按时间追加的活动日志。

## 基本流程

1. 把资料放入 `raw/`。
2. 让 Codex 摄取这份资料。
3. 检查 `wiki/` 中生成或更新的页面。
4. 在 Obsidian 中浏览链接、图谱视图和历史记录。

人负责选择资料和提出问题；LLM 负责维护摘要、链接、索引条目和日志。
