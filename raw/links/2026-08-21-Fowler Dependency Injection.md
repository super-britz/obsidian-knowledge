---
type: 链接来源
status: 已记录
created: 2026-08-21
source_url: "https://martinfowler.com/articles/injection.html"
---

# Fowler：Dependency Injection

- **作者：** Martin Fowler
- **URL：** https://martinfowler.com/articles/injection.html
- **访问日期：** 2026-08-21
- **资料类型：** 设计模式原始文章

## 来源摘要

Martin Fowler 将 Dependency Injection 描述为：由独立的组装者把接口所需的具体实现交给使用组件，而不是让组件自行实例化实现。文章区分构造注入、Setter 注入和接口注入，并将 Service Locator 作为另一种解除具体实现依赖的方式进行比较。

DI 让依赖可以从构造或注入位置直接看到；Service Locator 则要求组件主动查找服务，因此组件仍依赖 Locator。文章强调，比选择具体容器或注入形式更重要的是把服务配置与服务使用分开。

## 使用边界

- 原文示例使用 Java 与早期 IoC Container，机制不要求前端项目引入容器或类体系。
- 普通函数参数、工厂函数和构造参数都可以完成显式注入。
- DI 只负责提供实现，不会自动产生正确的 Port、生命周期或依赖方向。
