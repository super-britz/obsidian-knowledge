---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://alistair.cockburn.us/hexagonal-architecture"
---

# Cockburn：Hexagonal Architecture

- **作者：** Alistair Cockburn
- **URL：** https://alistair.cockburn.us/hexagonal-architecture
- **访问日期：** 2026-08-21
- **资料类型：** Ports and Adapters 原始文章

## 来源摘要

Alistair Cockburn 在 2005 年的原始文章中把业务逻辑与 UI、数据库等外部实体纠缠视为同一类问题。应用通过 Port 与外部交互，Port 的协议由双方对话的目的决定；每种具体外部设备或技术由 Adapter 在该协议与自身信号之间转换。

同一个 Port 可以连接 GUI、自动化测试、HTTP 调用、SQL、平面文件或内存替身等不同 Adapter，使应用能够脱离最终 UI、数据库和运行设备进行开发与测试。六边形只是强调应用内部与外部的非对称关系，不表示必须存在六个边或固定层数。

## 使用边界

- Ports and Adapters 是应用边界模型，不要求为每个函数或类都创建 Port。
- Port 表达一场有目的的对话，不只是一个语法上的 TypeScript Interface。
- 原文示例来自面向对象与数据库场景；迁移到前端时需根据真实 UI、网络、存储和浏览器边界调整。
