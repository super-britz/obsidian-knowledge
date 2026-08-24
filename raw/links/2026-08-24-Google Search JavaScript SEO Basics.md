---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics"
---

# Google Search：JavaScript SEO 基础

- **机构：** Google Search Central
- **URL：** https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics
- **访问日期：** 2026-08-24
- **资料类型：** 官方搜索文档

## 来源摘要

Google 文档把 JavaScript 页面的处理拆为 Crawling、Rendering 与 Indexing。Googlebot 可以执行 JavaScript，但页面可能先进入渲染队列，且并非所有机器人都具备相同能力；服务端生成或预渲染因此仍可能改善用户与爬虫取得内容的路径。

文档同时要求页面使用有意义的 HTTP Status，提供可发现的 URL、标题、描述与 Canonical 等元信息，并对 JavaScript 生成内容进行实际检查。客户端路由若把不存在页面一律返回 200，可能形成 Soft 404；仅以“Google 能执行 JavaScript”不能证明内容必然被正确发现和索引。

## 使用边界

- 这些规则描述 Google Search，不代表所有搜索引擎、社交预览机器人或企业爬虫都具备相同渲染能力。
- SSR 或 SSG 只改善内容和协议语义的可获得性，不保证收录、排名或业务流量。
- 登录后私有页面通常不以公开索引为目标，不能为了 SEO 把受保护内容变成公共响应。
