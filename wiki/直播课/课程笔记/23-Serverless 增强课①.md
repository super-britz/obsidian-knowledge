---
type: 课程笔记
status: 已整理
created: 2026-07-28
updated: 2026-07-28
session_no: 23
title: Serverless 增强课①
module: Web3、区块链与 Serverless
topics: [CSR, SSR, Streaming SSR, React, RSC, TTFB, Hydration, 服务端安全]
sources:
  - "raw/live/23-Serverless增强课①.txt"
---

# 23 Serverless 增强课①

> [!abstract] 核心问题
> SSR 与 Streaming SSR 改变了什么请求链路，又给前端带来了哪些服务端责任？

## 本节结论

- SSR 在服务端生成初始 HTML，有助于首屏内容可见性和爬虫读取，但不保证所有性能指标都更好。
- Streaming SSR 可以先发送页面 Shell，再逐步发送等待数据或代码的部分。
- 浏览器收到 HTML 后仍需加载客户端代码并 Hydrate，页面可见与可交互不是同一时刻。
- Node Stream 与 Web Stream 环境使用不同的 React 服务端渲染接口。
- RSC、SSR 和全栈框架把代码移到服务端后，密钥、输入校验和依赖漏洞也进入前端团队责任范围。

## 知识框架

### 1. CSR 与 SSR 的响应差异

```text
CSR: HTML 壳 -> 下载 JS -> 执行 -> 请求数据 -> 渲染
SSR: 请求 -> 服务端取数与渲染 -> HTML -> Hydrate
```

SSR 可以更早提供内容，但服务端取数和渲染也可能增加首字节等待。

### 2. Streaming SSR 拆开发送时机

```text
请求
  -> 生成 Shell
  -> 发送首批 HTML
  -> 等待异步内容
  -> 持续发送片段
  -> 客户端完成 Hydration
```

收益来自不必等待全部内容就开始传输和渲染。

### 3. React 服务端渲染接口

- `renderToString`：一次生成字符串，适合简单或兼容场景。
- `renderToPipeableStream`：面向 Node.js Stream。
- `renderToReadableStream`：面向 Web Streams 运行时。

生产实现还要处理错误、超时、取消、状态序列化和静态资源注入。

### 4. 可见不等于可交互

服务端 HTML 到达后，按钮可能已经可见，但在 Hydration 完成前未必能响应。架构需要同时观察 TTFB、内容绘制和交互准备时间。

### 5. 服务端边界需要重新审查

- 绝不能把密钥序列化到客户端。
- 所有外部输入都要在服务端验证。
- Server Action、RSC 和 API 权限必须独立检查。
- 日志不得记录敏感请求数据。
- 服务端依赖和运行时需要持续更新。

## 实践任务

**建议产出：**

- 实现同一页面的 CSR、传统 SSR 与 Streaming SSR 对照实验。

**完成标准：**

- [ ] 记录首字节、首屏内容和可交互时间。
- [ ] Streaming 版本能先返回 Shell。
- [ ] 错误和超时不会留下半个不可用页面。
- [ ] 检查服务端密钥与输入验证边界。

## 复习检查

- [ ] SSR 为什么可能改善内容可见性，却增加 TTFB？
- [ ] Streaming SSR 与 Hydration 分别负责什么？
- [ ] RSC 为什么扩大了前端的服务端安全责任？

## 关联

- 上一节：[[22-走进区块链的世界④]]
- 下一节：[[24-走进区块链的世界⑤]]

## 来源

- [[raw/live/23-Serverless增强课①|23-Serverless 增强课①原始文字稿]]
