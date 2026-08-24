---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends"
---

# Azure Architecture Center：Backends for Frontends Pattern

- **机构：** Microsoft Azure Architecture Center
- **URL：** https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends
- **访问日期：** 2026-08-24
- **资料类型：** 官方架构模式文档

## 来源摘要

文档把 BFF 描述为位于前端接口和后端服务之间、只处理特定前端需求的服务层。不同前端确有不同能力、数据形状或发布诉求时，可以分别定制后端，并让前端团队管理相应 BFF；这样能减少多个客户端争用同一个通用后端所产生的协调瓶颈。

文档同时强调新增服务的生命周期、部署、维护、安全、额外网络跳转和代码重复成本。多个界面请求相同或相似、或系统只有一个界面时，这个模式可能并不合适；通用鉴权、监控、路由等横切能力也不应无条件重复到每个 BFF。

## 使用边界

- “一个界面一个 BFF”是用于说明客户端差异的模式表达，不应机械落实成每个页面、设备型号或前端仓库一个服务。
- 是否拆分仍要依据真实请求差异、团队所有权、可靠性目标和运维能力判断。
- 文档提到 GraphQL 可能降低独立 BFF 的必要性；这只说明数据裁剪机制可能重叠，不代表 GraphQL 自动解决所有权、安全、失败恢复和部署责任。
