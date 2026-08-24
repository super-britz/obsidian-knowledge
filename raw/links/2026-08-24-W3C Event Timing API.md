---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.w3.org/TR/event-timing/"
---

# W3C：Event Timing API

- **机构：** W3C Web Performance Working Group
- **URL：** https://www.w3.org/TR/event-timing/
- **访问日期：** 2026-08-24
- **资料类型：** Working Draft

## 来源摘要

Event Timing API 为受支持的用户输入暴露事件时间，并用 `interactionId` 把同一手势产生的多个底层事件归为一次 Interaction。规范指出，这组机制可用于定义 Interaction to Next Paint，观察从用户输入到浏览器下一次呈现反馈的响应体验。

API 只覆盖规范列出的可信输入事件，并受浏览器实现、Duration Threshold、采样和隐私约束。它提供交互阶段的时间证据，不解释业务操作最终是否成功。

## 使用边界

- INP 衡量可观察交互反馈，不等于接口完成时间、领域状态已提交或整个用户任务完成。
- 第一次输入、最慢交互和特定核心动作可能具有不同业务重要性，需要保留动作语义而不只聚合为一个页面数字。
- Event Timing 是诊断与真实用户监测的一部分，仍需和 Long Task、网络 Trace、版本及业务完成事件关联。
