---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://html.spec.whatwg.org/multipage/webappapis.html#import-maps"
---

# WHATWG HTML：Import Maps

- **机构：** WHATWG
- **URL：** https://html.spec.whatwg.org/multipage/webappapis.html#import-maps
- **访问日期：** 2026-08-24
- **资料类型：** HTML Living Standard

## 来源摘要

HTML 标准把 Import Map 定义为包含 `imports`、`scopes` 和 `integrity` 等映射的结构，用于将 JavaScript Module Specifier 解析为 URL。解析会按适用 Scope、最具体的 Specifier 和 Prefix 选择映射，找不到有效结果时抛出错误。

Import Map 因而可以把稳定模块名指向具体部署资产，也可以按 Scope 控制不同解析结果。它负责浏览器模块解析，不负责决定业务边界、部署审批、版本兼容或失败恢复。

## 使用边界

- 更新 Import Map 可以改变后续模块解析结果，但已经解析和执行的模块不会因此自动热切换为新版本。
- 映射文件是运行时组合的控制面，需要自身的发布原子性、缓存、回滚、权限和观测策略。
- `integrity` 能力与具体浏览器、加载链和部署实现有关，落地时应重新核对兼容性。
