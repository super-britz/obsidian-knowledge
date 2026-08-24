---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://docs.aws.amazon.com/wellarchitected/2023-04-10/framework/perf_select_network_location.html"
---

# AWS：按网络要求选择工作负载位置

- **机构：** AWS Well-Architected Framework
- **URL：** https://docs.aws.amazon.com/wellarchitected/2023-04-10/framework/perf_select_network_location.html
- **访问日期：** 2026-08-24
- **资料类型：** 官方架构指南

## 来源摘要

指南说明，工作负载所在位置会影响用户访问时的网络延迟与吞吐。选择区域或边缘位置时，应同时考虑用户在哪里、权威数据在哪里，以及安全、合规和数据驻留等约束；对数据密集型应用，计算靠近数据可以减少反复跨区传输。

CDN 和边缘服务可以把可复用内容放到更接近用户的位置，或优化通往源站的网络路径。边缘计算还适合执行延迟敏感且不必频繁往返远端状态的轻量操作。

## 使用边界

- 文档中的 Region、Local Zone、CloudFront 等是 AWS 具体产品；本页只迁移“用户、计算与数据之间的网络距离共同决定端到端延迟”这一原则。
- 计算靠近用户不等于完整请求更快；若每一步仍需跨洲访问权威数据库，边缘层可能增加往返。
- 多区域和边缘部署会新增数据复制、缓存失效、运行时差异、合规与故障排查责任。
