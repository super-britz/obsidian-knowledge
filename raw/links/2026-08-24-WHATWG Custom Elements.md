---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://html.spec.whatwg.org/multipage/custom-elements.html"
---

# WHATWG HTML：Custom Elements

- **机构：** WHATWG
- **URL：** https://html.spec.whatwg.org/multipage/custom-elements.html
- **访问日期：** 2026-08-24
- **资料类型：** HTML Living Standard

## 来源摘要

HTML 标准定义了 Custom Element 的注册、升级和生命周期回调，包括元素连接、断开、移动、属性变化以及表单相关生命周期。自定义元素名称具有明确语法，并通过 Custom Element Registry 与构造器关联。

生命周期回调可能被调用多次，例如元素重新连接时会再次触发 `connectedCallback`，因此初始化与清理必须明确。Custom Elements 为不同实现提供浏览器原生的元素契约，但标准本身不赋予独立部署、CSS 隔离、JavaScript 权限隔离或团队自治。

## 使用边界

- Custom Element 可以作为微前端的挂载接口，也可以只实现普通组件；不能从标签形式反推系统架构。
- Shadow Root 可以与 Custom Element 组合，但两者不是同一个概念，是否使用样式封装需要单独决定。
- Attributes、Properties、Events、Lifecycle 和错误语义仍要作为可演进公共契约设计。
