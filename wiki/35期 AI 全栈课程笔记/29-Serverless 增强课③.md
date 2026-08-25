---
type: 课程笔记
status: 已整理
created: 2026-07-28
updated: 2026-07-28
session_no: 29
title: Serverless 增强课③
module: Web3、区块链与 Serverless
topics: [AWS Lambda, Cloud Run, Cloudflare Workers, Vercel, S3, CDN, 消息队列, ARM64]
sources:
  - "raw/live/29-Serverless增强课③.txt"
---

# 29 Serverless 增强课③

> [!abstract] 核心问题
> 面对函数、容器、边缘与托管平台，如何建立 Serverless 选型框架？

## 本节结论

- Serverless 是计算、存储、队列、网关、监控和边缘网络的组合，不等于单独使用云函数。
- 函数、托管容器、边缘运行时和全栈托管平台的执行模型不同，适用场景也不同。
- SSR 架构通常需要动静分离：动态请求进入计算层，静态资源进入对象存储与 CDN。
- 平台选型应比较运行时限制、冷启动、网络、数据服务、观测、成本与团队能力。
- ARM64 可能改善部分工作负载的性价比，但必须先验证依赖兼容性、性能和回滚。

## 知识框架

### 1. Serverless 是一组托管能力

```text
入口：DNS / CDN / WAF / API Gateway
计算：Function / Container / Edge Runtime
数据：Database / Object Storage / Cache
异步：Queue / Event Bus / Scheduler
治理：Logs / Metrics / Tracing / IAM / Budget
```

完整架构要说明这些组件如何协作，而不是只画一个函数图标。

### 2. 常见计算平台的定位

| 类型 | 课程中的代表 | 更适合关注的问题 |
| --- | --- | --- |
| 函数计算 | AWS Lambda | 事件驱动、短任务、弹性与冷启动 |
| 托管容器 | Google Cloud Run | 容器兼容、服务进程与运行时自由度 |
| 边缘运行时 | Cloudflare Workers | 全球分发、Web API 与运行时限制 |
| 全栈托管 | Vercel | 前端框架集成、预览与部署体验 |

平台能力会持续变化，实际决策应以当前项目验证为准。

### 3. SSR 的动静分离

```text
HTML / API 动态请求 -> 计算平台
JS / CSS / 图片等静态资源 -> 对象存储 -> CDN
日志与指标 -> 观测平台
```

动态与静态资源具有不同的更新频率、缓存方式和扩缩容需求。

### 4. 选型问题比产品名称更重要

- 请求持续时间和并发模式是什么？
- 是否需要长连接、后台任务或特定系统依赖？
- 数据库与外部服务位于哪里？
- 冷启动是否影响核心路径？
- 区域、隐私与数据合规有什么要求？
- 团队能否监控、排障和控制费用？

### 5. ARM64 迁移是一次兼容性实验

迁移前应检查原生依赖、构建镜像、性能基准、内存使用和回滚。不能只依据标价或理论性能直接切换生产环境。

## 实践任务

**建议产出：**

- 为一个 SSR 应用完成 Serverless 平台选型记录。

**完成标准：**

- [ ] 画出入口、计算、静态资源、数据和观测组件。
- [ ] 对比函数、容器、边缘和全栈托管方案。
- [ ] 记录关键限制、成本假设和验证结果。
- [ ] ARM64 方案通过依赖、性能和回滚测试。

## 复习检查

- [ ] 为什么 Serverless 不等于云函数？
- [ ] SSR 应用为什么通常需要动静分离？
- [ ] ARM64 迁移前必须验证哪些问题？

## 关联

- 上一节：[[28-Serverless 增强课②]]
- 下一节：[[30-阶段一收尾-区块链原理]]

## 来源

- [[raw/live/29-Serverless增强课③|29-Serverless 增强课③原始文字稿]]
