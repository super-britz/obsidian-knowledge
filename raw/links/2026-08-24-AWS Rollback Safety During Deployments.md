---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://builder.aws.com/content/3F04j2yRAAMBuPSPs50xwXZqg01/ensuring-rollback-safety-during-deployments"
---

# AWS：Ensuring rollback safety during deployments

- **机构：** AWS Builder Center
- **URL：** https://builder.aws.com/content/3F04j2yRAAMBuPSPs50xwXZqg01/ensuring-rollback-safety-during-deployments
- **访问日期：** 2026-08-24
- **资料类型：** 官方工程文章

## 来源摘要

文章指出，分布式系统的部署过程通常会让旧版本和新版本同时运行。若新版本把共享数据、消息或序列化状态写成旧版本无法读取的格式，代码即使能回滚，旧版本也可能因遇到新数据而失败。

文章建议把不兼容变更拆成 Prepare 与 Activate 两阶段：先让所有读取方同时理解旧格式和新格式，但继续写旧格式；确认所有读取方升级并经过观察期后，才让写入方产生新格式。新格式一旦进入共享状态，安全回滚点通常只能退到仍兼容两种格式的 Prepare 版本，而不是更早的旧版本。删除旧读取能力前，还要完成历史数据回填并确认旧格式不再使用。

文章还建议为格式显式记录版本，按版本产生指标，并在测试中覆盖升级、降级和新旧版本共存，而不只测试两个孤立稳定版本。

## 使用边界

- 两阶段部署解决的是新旧程序与共享格式的兼容顺序，不会自动撤销已经产生的业务数据和外部副作用。
- “所有读取方已经升级”必须由实例、客户端、队列消费者或缓存使用情况的可观测证据证明，不能只依据发布任务显示成功。
- 对长期离线客户端或不可控第三方消费者，兼容窗口可能需要明确支持期限和版本路由，而不能假设可以同步升级。
