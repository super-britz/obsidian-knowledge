---
type: 链接来源
status: 已记录
created: 2026-08-19
source_url: "https://waylandz.com/ai-agent-book/%E7%AC%AC39%E7%AB%A0-Prompt-Cache%E7%A8%B3%E5%AE%9A%E6%80%A7/"
parent_source: "raw/links/2026-08-17-AI Agent架构从单体到企业级多智能体.md"
---

# AI Agent 架构：第 39 章 Prompt Cache 稳定性

- **原文标题：** 第 39 章：Prompt Cache 稳定性
- **URL：** https://waylandz.com/ai-agent-book/%E7%AC%AC39%E7%AB%A0-Prompt-Cache%E7%A8%B3%E5%AE%9A%E6%80%A7/
- **访问日期：** 2026-08-19
- **资料类型：** 在线书籍章节页

## 来源摘要

本章将 Prompt Cache 定义为 Request Builder 的序列化契约：Provider 复用逐字节相同的前缀，而非语义相同的对象；第一个不同字节即终止复用。Map 遍历顺序、工具 Schema 排序、空值编码、时间戳、历史截断边界或压缩重写都可能造成逻辑等价但 Wire 字节不同的请求。系统必须测试实际发送的请求表示、实施确定性排序与规范编码，并将易变值放在稳定缓存边界之后。

有限的 Cache Breakpoint 应按共享范围和变化频率分配给跨用户稳定的系统规则、完整且确定排序的工具 Schema、滚动历史前缀和单 Session 稳定上下文；Provider 的插槽上限意味着新增逻辑区域需合并或移动既有边界，而非无限添加标记。执行期授权不应通过过滤或重排 Schema 表达，否则会破坏稳定工具前缀；延迟发现、模型可见 Schema 和执行授权是三套不同机制。

需要派生 Turn 后操作或推测请求时，应从已 Dispatch 的真实 Request Artifact Fork，只追加明确差异，保持工具数组等共享前缀只读；从高层状态重新构建或在 Fork 后继续修改模型参数、工具和预算都会引入缓存漂移。缓存调试需记录有意重写前后的哈希及其原因，并与请求日志、Cache Read/Creation、成本、延迟、调用次数和任务成功率一起分析；命中率本身不是产品指标。

## 使用边界

- 缓存优化不能改变权限、正确性或用户可见行为；先定义稳定接口，再评估是否值得复用。
- Breakpoint 数量、Marker 语义和 Provider 缓存布局是实现相关约束；可迁移原则是“按共享范围布置稳定前缀、将漂移后移、以 Wire 字节验证”。
- `cache_source` 等来源字段可用于成本归因，不应假定它们必然控制 TTL 或其他 Provider 策略。
