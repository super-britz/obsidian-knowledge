---
type: 课程笔记
status: 已整理
created: 2026-07-28
updated: 2026-07-28
session_no: 28
title: Serverless 增强课②
module: Web3、区块链与 Serverless
topics: [CSR, SSR, Streaming SSR, TTFB, TTLB, Hydration, Code Splitting, Serverless 治理]
sources:
  - "raw/live/28-Serverless增强课②.txt"
---

# 28 Serverless 增强课②

> [!abstract] 核心问题
> 如何用请求时间线判断 CSR、SSR 与 Streaming SSR 的性能，并评估 Serverless 的组织落地？

## 本节结论

- CSR、SSR 和 Streaming SSR 的差异应放在完整时间线上比较，不能只看首个 HTML 是否返回。
- SSR 可能更早提供内容，但服务端取数与渲染也可能增加 TTFB。
- Streaming SSR 的价值是先发送可用 Shell，再逐步发送等待中的内容，不是自动消除所有性能瓶颈。
- SSR 的代码分割需要在服务端收集本次渲染使用的资源，并把对应 JS 与 CSS 注入响应。
- Serverless 落地同时改变运行平台、权限、成本和团队职责，因此也是组织治理问题。

## 知识框架

### 1. 用统一时间线比较渲染方式

```text
请求发出
  -> 收到首字节
  -> 收完响应
  -> 内容可见
  -> 客户端代码加载
  -> Hydration
  -> 页面可交互
```

只观察其中一个节点，容易把网络、服务端计算、资源下载和客户端执行混在一起。

### 2. TTFB 与 TTLB 回答不同问题

- TTFB：从请求开始到收到响应第一个字节。
- TTLB：从请求开始到接收完整响应。

Streaming 可能提前首批内容，但完整响应和交互准备仍取决于后续数据、资源与 Hydration。

### 3. 三种渲染路径

| 模式 | 主要等待位置 | 典型优势 | 主要代价 |
| --- | --- | --- | --- |
| CSR | 浏览器加载与执行 | 服务端简单、交互逻辑集中 | 首屏依赖 JS |
| SSR | 服务端完成取数与渲染 | 初始 HTML 含内容 | 服务端计算与 Hydration |
| Streaming SSR | 服务端分批输出 | 不必等待全部内容 | 错误、资源与状态管理更复杂 |

选择应基于页面内容、缓存、个性化程度和运行环境。

### 4. SSR 代码分割需要资源清单

服务端渲染某个路由时，需要知道实际使用了哪些 Chunk，并将对应的脚本、样式和预加载信息写入 HTML。否则服务端有内容，客户端却可能缺少完成 Hydration 的资源。

### 5. 技术推进要回答组织问题

- 谁拥有生产部署权限？
- 事故由谁监控和响应？
- 成本如何归属和限制？
- 安全与合规由谁审批？
- 现有运维和后端职责如何变化？
- 新平台失败后如何回滚？

Serverless 方案只有进入这些约束后，才算可落地架构。

## 实践任务

**建议产出：**

- 绘制 CSR、SSR 和 Streaming SSR 的性能时间线，并编写 Serverless 落地检查表。

**完成标准：**

- [ ] 时间线区分首字节、完整响应、内容可见和可交互。
- [ ] 使用实际测量数据，而不是仅凭渲染模式判断快慢。
- [ ] SSR 版本能正确注入当前路由所需资源。
- [ ] 检查表覆盖权限、费用、监控、责任和回滚。

## 复习检查

- [ ] 为什么 SSR 可能改善内容可见性，却增加 TTFB？
- [ ] Streaming 改变的是响应中的哪个等待关系？
- [ ] 为什么 Serverless 推进不是纯技术选型？

## 关联

- 上一节：[[27-走进区块链的世界⑧]]
- 下一节：[[29-Serverless 增强课③]]
- 相关课程：[[23-Serverless 增强课①]]

## 来源

- [[raw/live/28-Serverless增强课②|28-Serverless 增强课②原始文字稿]]
