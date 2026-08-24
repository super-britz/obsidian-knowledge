---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://docs.github.com/en/rest/about-the-rest-api/breaking-changes"
---

# GitHub：REST API Breaking Changes

- **机构：** GitHub
- **URL：** https://docs.github.com/en/rest/about-the-rest-api/breaking-changes
- **访问日期：** 2026-08-24
- **资料类型：** 官方 API 文档

## 来源摘要

GitHub 文档说明，删除或重命名操作、参数或响应字段，新增必填参数，改变参数或响应字段类型，以及改变认证方式等，都属于需要新 API 版本的 Breaking Change。

在 GitHub 明示的 REST 契约中，增加新的资源、操作、可选参数、可选请求头、响应字段或响应头等被视为 Non-breaking Change。客户端应只依赖已经承诺的契约，并通过显式 API Version 选择支持范围，而不是把某次响应快照当成永不变化的封闭结构。

## 使用边界

- “增加字段属于兼容变更”只在消费者能够忽略未知字段的契约下成立；严格反序列化器、封闭枚举和业务穷举仍可能被新增值破坏。
- GitHub 的版本政策是一个具体 API 契约，不等于所有组织必须采用相同版本周期或兼容分类。
- API 语法兼容不能证明业务语义不变；默认值、权限、排序和副作用变化仍需契约测试。
