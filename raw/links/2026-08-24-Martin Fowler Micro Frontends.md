---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://martinfowler.com/articles/micro-frontends.html"
---

# Martin Fowler：Micro Frontends

- **作者：** Cam Jackson
- **站点：** MartinFowler.com
- **URL：** https://martinfowler.com/articles/micro-frontends.html
- **访问日期：** 2026-08-24
- **资料类型：** 基础架构文章

## 来源摘要

文章将微前端定义为把可独立交付的前端应用组合成一个更大整体的架构风格，重点是围绕业务能力形成垂直切片，让团队拥有从开发、测试到生产发布的完整责任。它列举了服务端模板、构建时 Package、iframe、运行时 JavaScript 和 Web Components 等集成方式，并把独立部署而非某种工具作为关键能力。

文章同时讨论跨应用通信、全局 CSS、共享组件、测试、Payload、环境差异和治理复杂度。共享依赖可以减少重复下载，却会重新引入版本协调；跨应用频繁通信和共享领域模型也会削弱原本想获得的自治。

## 使用边界

- 这是基于实践经验的基础文章，不是浏览器标准，也不要求所有大型前端采用微前端。
- 文中的页面与团队切分是示例；真实边界仍应由稳定业务变化、交付阻塞和所有权证据决定。
- 文章发布于 2019 年，具体加载工具和浏览器能力需要结合当前官方文档复核，但独立交付、显式契约和成本权衡仍可作为分析框架。
