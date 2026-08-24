---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://research.google/pubs/why-google-stores-billions-of-lines-of-code-in-a-single-repository/"
---

# Google Research：Why Google Stores Billions of Lines of Code in a Single Repository

- **机构：** Google Research
- **URL：** https://research.google/pubs/why-google-stores-billions-of-lines-of-code-in-a-single-repository/
- **访问日期：** 2026-08-24
- **资料类型：** 官方研究论文入口

## 来源摘要

论文把 Google 的 Monolithic Repository 描述为数万名开发者共同使用的单一事实来源，并同时讨论该源码管理模型的优势、权衡，以及让超大仓库保持可用所需的系统与工作流。

该案例的重要事实不是“规模越大越应该 Monorepo”，而是仓库策略和配套工程能力不可分离：统一源码视图能降低部分发现、复用和跨项目修改成本；超大代码量、开发者数量与提交频率又要求专门的版本控制、代码搜索、构建、测试和变更治理基础设施。

## 使用边界

- Google 使用的是为自身规模建设的专用源码与工程系统，不能据此推断普通 Git 仓库会自动获得相同扩展能力。
- 单一事实来源只统一了源码快照，不会自动形成正确模块、公共 API、团队所有权或运行时隔离。
- Google 的组织可见性和权限条件具有特殊性；存在严格代码保密、客户隔离或外部协作者边界时需要单独评估。
