---
type: 课程笔记
status: 已整理
created: 2026-07-28
updated: 2026-07-28
session_no: 15
title: 企业 AIGC 工程提效第二季③
module: 企业 AIGC 工程提效
topics: [Agent, Skills, Tool Wrapper, Generator, Reviewer, Interviewer, Pipeline, Git worktree]
sources:
  - "raw/live/15-企业AIGC工程提效第二季③.txt"
---

# 15 企业 AIGC 工程提效第二季③

> [!abstract] 核心问题
> 如何把团队经验设计成 Agent 可按需调用、组合和验证的 Skills？

## 本节结论

- Agent 是执行与编排主体，Skill 是可复用的能力模块，Tool 是访问外部系统的接口。
- Skill 的价值不在文件格式，而在清楚定义触发条件、输入、步骤、产物和失败边界。
- 团队规范应拆成按需加载的 Skills，避免所有知识都塞进一个巨大系统提示。
- 生成、评审、访谈和流水线是不同能力模式，应根据任务目标选择。
- Worktree 和 Subagent 可以支持并行，但只有写入范围彼此独立时才真正安全。

## 知识框架

### 1. Agent、Skill 与 Tool 的关系

```text
Agent
  -> 选择并执行 Skill
  -> Skill 按流程调用 Tool
  -> Tool 访问代码、浏览器、API 或应用
```

Agent 决定做什么，Skill 提供怎样做的组织经验，Tool 提供实际动作。

### 2. 五类常见 Skill 模式

| 模式 | 主要用途 |
| --- | --- |
| Tool Wrapper | 封装 API、库或内部平台的正确用法 |
| Generator | 按稳定模板生成代码、配置或文档 |
| Reviewer | 根据清单、规则和优先级检查产物 |
| Interviewer | 执行前主动提问，补齐缺失约束 |
| Pipeline | 强制复杂任务按阶段推进并设置检查点 |

一个 Skill 可以组合多种模式，但职责应保持可理解。

### 3. Skill 需要明确契约

至少写清：

- 何时触发，何时不适用。
- 需要读取哪些上下文。
- 按什么顺序执行。
- 可以调用哪些工具。
- 输出格式和完成标准。
- 遇到歧义、失败或高风险操作时怎样处理。

### 4. 公司级 Skills 沉淀组织经验

适合模块化的内容包括代码规范、架构审查、安全检查、发布流程、故障复盘和特定业务领域规则。敏感规则还需要访问控制和版本治理。

### 5. 并行执行依赖隔离

不同 Agent 在不同 Git worktree 中工作，可以隔离文件状态；但任务依赖、公共接口和最终集成仍需协调。不要把相互依赖的修改伪装成并行任务。

## 实践任务

**建议产出：**

- 为团队编写一个 Reviewer Skill 和一个 Pipeline Skill。

**完成标准：**

- [ ] Skill 有明确触发条件、输入和输出。
- [ ] Reviewer 使用可执行的检查清单并区分严重程度。
- [ ] Pipeline 包含需求、实现、验证和人工审批节点。
- [ ] 用一个真实任务验证 Skill 能否重复得到稳定产物。

## 复习检查

- [ ] Agent、Skill 和 Tool 的职责有何不同？
- [ ] 什么情况下应使用 Interviewer 模式？
- [ ] 为什么并行 Agent 仍需要集成与审批？

## 关联

- 上一节：[[14-企业 AIGC 工程提效第二季②]]
- 下一节：[[16-企业 AIGC 工程提效第二季（终）]]
- 相关主题：[[AI 工程化-上下文工程与知识复利]]

## 来源

- [[raw/live/15-企业AIGC工程提效第二季③|15-企业 AIGC 工程提效第二季③原始文字稿]]
