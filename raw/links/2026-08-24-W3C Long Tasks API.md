---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.w3.org/TR/longtasks-1/"
---

# W3C：Long Tasks API

- **机构：** W3C Web Performance Working Group
- **URL：** https://www.w3.org/TR/longtasks-1/
- **访问日期：** 2026-08-24
- **资料类型：** Working Draft

## 来源摘要

Long Tasks API 用于发现长期占用 UI Thread、阻碍输入响应等关键工作的任务。当前草案把超过 50 毫秒的 Event Loop Task 及其随后 Microtask Checkpoint、渲染更新步骤等列为 Long Task，并通过 Performance Timeline 暴露开始时间、持续时间与受同源策略限制的归因信息。

规范还说明，脚本、Layout 等工作以及第三方 Frame 都可能造成主线程阻塞；出于跨源隐私和安全考虑，浏览器不能总是暴露精确责任代码。

## 使用边界

- 本文档是仍可能变化的 Working Draft；50 毫秒用于检测主线程风险，不等于所有用户操作的完整性能预算。
- 没有 Long Task 不保证任务完成快，网络、服务端、多个较短任务和业务等待仍可能造成延迟。
- 归因受跨源与浏览器支持限制，第三方脚本问题需要结合网络、Long Animation Frame、Event Timing 和业务事件分析。
