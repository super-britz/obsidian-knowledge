---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element"
---

# WHATWG HTML：iframe 与 Sandbox

- **机构：** WHATWG
- **URL：** https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element
- **访问日期：** 2026-08-24
- **资料类型：** HTML Living Standard

## 来源摘要

`iframe` 会创建子级 Navigable 和独立 Document；`sandbox` 属性默认施加一组额外限制，再由 `allow-*` Token 有选择地恢复脚本、表单、弹窗、导航或真实 Origin 等能力。规范建议把潜在不可信内容放在独立域名，并谨慎配置 Sandbox。

规范特别提醒：当嵌入页与宿主页同源，同时允许 `allow-scripts` 和 `allow-same-origin` 时，嵌入脚本可能移除 Sandbox 并在重新加载后逃逸限制。动态改变 Sandbox 也会让实际权限难以推理。

## 使用边界

- iframe 提供的是可配置的 Document、Origin 和能力边界，不是无条件安全容器。
- 更强隔离会增加路由、尺寸、焦点、无障碍、会话和通信成本；是否值得取决于威胁模型和集成需求。
- 即使浏览器隔离了脚本权限，多个区域仍共享用户设备、网络和整体产品体验。
