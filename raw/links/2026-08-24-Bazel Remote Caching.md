---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://bazel.build/remote/caching"
---

# Bazel：Remote Caching

- **机构：** Bazel
- **URL：** https://bazel.build/remote/caching
- **访问日期：** 2026-08-24
- **资料类型：** 官方构建文档

## 来源摘要

Bazel Remote Cache 允许开发者与 CI 共享已经完成的 Build Action 输出。每个 Action 显式声明输入、输出名称、命令行与环境变量；系统根据 Action Hash 查找结果，并通过 Content-addressable Store 保存输出内容。

官方文档把安全复用建立在 Build Reproducibility 上：相同已声明输入必须产生可复用结果。Remote Cache 还需要独立的存储后端、认证、读写策略和清理，因此它不仅是本地临时目录的网络版本。

## 使用边界

- 未声明的文件、时间、网络、环境变量或工具链会让 Hash 不能代表真实输入，可能产生错误 Cache Hit。
- 远程缓存保存构建输出和日志，应控制读写权限、敏感内容与不可信写入，不能只关注命中率。
- Cache 只能减少重复计算；错误依赖图、过大的任务单元和本身缓慢的首次构建仍需单独治理。
