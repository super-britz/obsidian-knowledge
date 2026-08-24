---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://drafts.csswg.org/css-shadow-1/"
---

# CSSWG：CSS Shadow Module Level 1

- **机构：** CSS Working Group
- **URL：** https://drafts.csswg.org/css-shadow-1/
- **访问日期：** 2026-08-24
- **资料类型：** Editor's Draft（工作草案）

## 来源摘要

CSS Shadow Module 定义 CSS 如何与 Shadow DOM 的封装树交互。Shadow Tree 把组件内部子树与页面其余部分分开，可以减少选择器意外越界和全局样式误作用；同时，`:host`、Parts 和 Custom Properties 等机制允许组件有控制地向外暴露样式接口。

该规范仍是工作草案。它描述的是 DOM 与 CSS 名称、选择器和样式作用范围，不是 JavaScript 执行、网络权限、CPU、内存或浏览器安全主体的隔离机制。

## 使用边界

- Shadow DOM 适合减少组件内部 DOM 与样式实现泄漏，但不能被当作恶意代码沙箱。
- 宿主页与 Shadow Tree 仍处于同一个页面运行环境；脚本信任、主线程占用和网络能力需要另行治理。
- 暴露 Parts、Custom Properties 或宿主样式入口时，它们也会成为需要兼容的公共契约。
