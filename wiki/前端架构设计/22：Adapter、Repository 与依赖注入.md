---
type: 概念建模
status: 可用
created: 2026-08-21
updated: 2026-08-21
sources:
  - "raw/live/11-前端架构设计指南⑤.txt"
  - "raw/links/2026-08-21-Cockburn Hexagonal Architecture.md"
  - "raw/links/2026-08-21-Fowler Dependency Injection.md"
  - "raw/links/2026-08-21-Fowler Repository Pattern.md"
related:
  - "wiki/前端架构设计/00：索引.md"
  - "wiki/前端架构设计/11：让依赖指向稳定规则.md"
  - "wiki/前端架构设计/13：可信边界与副作用隔离.md"
  - "wiki/前端架构设计/20：Feature-based 模块与公共 API.md"
  - "wiki/前端架构设计/21：状态机与异步流程.md"
---

# Adapter、Repository 与依赖注入

> [!abstract] 先用一句话说清楚
> Port 让核心用自己的语言声明“我需要什么能力”，Adapter 把具体 UI、HTTP、存储或 SDK 翻译成这种能力，Repository 是具有领域集合语义的数据访问 Port，依赖注入则从外部把选定实现交给核心。它们共同解决的是变化传播和对象组装，不是为了给每次 `fetch` 多套几层文件。

## 从“外部变化不应改写核心规则”推导

假设提交订单的用例直接依赖请求库、URL、服务端 DTO 和供应商错误：

    SubmitOrder
    → Axios / fetch 调用方式
    → POST /v2/orders
    → ServerOrderDTO、AxiosError

更换请求库、接口版本或测试环境时，稳定的下单规则也被迫修改。把调用移动到 `OrderService` 仍不一定有用；如果它只是逐项转发 URL、DTO 和错误类型，外部协议仍然控制着核心的理解方式。

真正需要倒置的是**谁定义协作语言、谁选择具体实现**：

    核心用例定义所需能力 Port
    → 外部 Adapter 实现并翻译该能力
    → Composition Root 选择实现并注入
    → 核心只在 Port 的业务语义真正改变时修改

这把 [[11：让依赖指向稳定规则|依赖方向原则]] 落成可执行连接，同时把 [[13：可信边界与副作用隔离|外部转换与副作用]] 集中到可识别位置。

## 六个概念各自负责什么

| 概念 | 责任 | 不等于什么 |
| --- | --- | --- |
| Port | 定义核心与外界一场有目的的对话 | 第三方 API 的方法清单 |
| Adapter | 在 Port 与具体协议、设备或 SDK 间双向转换 | 只改一个函数名的 Wrapper |
| Repository | 用集合式领域语义访问持久化对象 | 通用 `get/post/put` Client |
| DI | 从外部把依赖显式交给使用者 | DI Container |
| Container | 自动注册、解析和管理对象图 | 正确架构的前提 |
| Composition Root | 集中选择实现、配置和生命周期 | 业务代码到处 `resolve()` |

IoC 是更宽泛的“控制权交给外部协调者”；DI 只是在对象如何获得依赖这件事上实现 IoC。核心主动调用全局 Service Locator 也能隐藏具体实现，但依赖不再能从函数或构造签名直接看出，因此通常优先显式注入。

## 源码方向与运行方向并不相同

一次真实执行仍然是核心调用外部实现：

    运行时：SubmitOrder → HttpOrderAdapter → HTTP Server

但源码依赖可以指向核心契约：

    SubmitOrder       → OrderPort
    HttpOrderAdapter  → OrderPort
    CompositionRoot   → SubmitOrder + HttpOrderAdapter + Config

Composition Root 被允许同时知道稳定契约和易变实现，因为它的唯一责任就是完成选择。核心若自己 `new HttpOrderAdapter()`、读取供应商配置或调用 `container.resolve()`，创建责任又泄漏回使用者。

Port 也不必总是一个 `interface` 文件。函数参数、对象类型或回调都能形成契约；关键证据是核心源码不再引用外部实现的类型、配置和协议。

## 一个前端下单边界

核心先用业务语言声明需要的能力：

```ts
type OrderPort = {
  submit(command: SubmitOrderCommand): Promise<SubmitOrderReceipt>;
  findByOperationId(id: OperationId): Promise<OrderStatus>;
};

type Dependencies = { orders: OrderPort };

export const makeSubmitOrder = ({ orders }: Dependencies) =>
  async (command: SubmitOrderCommand): Promise<SubmitOrderReceipt> => {
    if (command.items.length === 0) return { kind: 'rejected', reason: 'empty' };
    return orders.submit(command);
  };
```

HTTP Adapter 再负责把内部命令翻译成请求，并把外部回执转回内部结果：

```ts
const orders = makeHttpOrderAdapter({ request: fetchJson, baseUrl: config.api });
export const submitOrder = makeSubmitOrder({ orders });
```

Adapter 内部应承担 DTO 映射、外部响应解析、供应商错误转换以及 Operation ID 的协议传递；运行时 Schema 和错误分类将在下一页展开。它不能偷偷替核心决定写请求是否重试，也不能把 `Response`、`AxiosError` 或原始 DTO 重新暴露给用例，否则边界只是搬家。

在输入方向，Router、UI 事件处理器或测试驱动器也可以是 Adapter：它们把点击、URL 或测试步骤转换为应用用例调用。Adapter 不只存在于“数据库那一侧”。

## Repository 只在集合语义成立时使用

Repository 的核心直觉是：调用方像面对一个领域对象集合，而不关心对象实际来自 HTTP、IndexedDB 还是其他存储：

    OrderRepository
      getById(orderId)
      findPending(customerId)
      save(order)

它适合复杂领域对象、重复查询规则或需要隔离数据映射的场景。支付扣款、文件选择、发送埋点等命令式能力没有自然集合语义，使用 `PaymentPort`、`FilePickerPort`、`AnalyticsPort` 通常更诚实。

以下命名通常只是 Client 或 Gateway，而非 Repository：

    api.get(url)
    api.post(url, body)
    repository.request(method, path, data)

前端 Repository 即使聚合网络、缓存和本地存储，也不能抹掉 [[12：状态所有权与合法转换|数据权威与副本规则]]。调用方仍需知道返回的是权威事实、允许陈旧的缓存，还是离线草稿；“统一数据源”不能以丢失一致性语义为代价。

## 注入位置同时决定生命周期

依赖注入不只影响测试，也决定对象何时创建、共享和释放：

| 生命周期 | 适合什么 | 主要风险 |
| --- | --- | --- |
| 每次调用 | 无状态、创建便宜的临时对象 | 重复创建与连接浪费 |
| Feature / 页面 | 只服务当前交互流程的状态 | 卸载清理和重新进入 |
| 应用 / 会话 | 配置、认证客户端、会话缓存 | 切换用户后残留旧状态 |
| SSR 请求 | 与一次服务端渲染绑定的数据 | 错用全局单例造成跨用户泄漏 |

最小实现优先使用函数参数、工厂或构造注入；对象图变大、生命周期和环境变体反复出现后，再考虑 Container。无论是否有容器，浏览器启动入口、Provider 边界或 SSR 请求入口都应成为清楚的 Composition Root，并负责销毁订阅、连接和有状态资源。

## 新增成本与可观察信号

Port 会增加命名和跳转，Adapter 会增加转换代码，多个实现可能把契约拉成“最低共同能力”，替身也可能逐渐偏离真实系统。只有外部变化、独立测试、环境替换或生命周期控制带来的收益超过这些间接成本时，边界才值得存在。

| 可观察现象 | 可能的问题 |
| --- | --- |
| 业务用例直接引用 SDK 类型或 HTTP 错误 | 外部协议已经进入核心 |
| 替换请求库需要修改多个 Feature | 缺少局部 Adapter |
| `IUserService` 与 `UserService` 方法完全一一对应 | 为形式而抽象，没有转换责任 |
| 所有模块都调用 `container.resolve()` | Service Locator 隐藏了真实依赖 |
| 每处调用都自行创建 Adapter | 配置、身份和生命周期没有集中 |
| 一个 `Repository` 包含所有后端接口 | 边界按技术入口而非领域能力划分 |
| Fake 测试通过但真实 Adapter 行为不同 | 缺少契约测试和边界集成测试 |
| 全局单例保存用户或请求状态 | 注入生命周期越过了所有权边界 |

## 做项目时怎么验证

1. 选择一个已经造成测试困难或技术变化传播的外部依赖，不从目录模板开始。
2. 用调用方语言写出最小 Port，保留核心真正需要判断的取消、失败和一致性语义。
3. 让 Adapter 完成外部参数、DTO、错误和协议身份的转换，不把供应商类型重新导出。
4. 在 Composition Root 注入真实实现，并明确实例是调用级、Feature 级、会话级还是请求级。
5. 用轻量 Fake 测试核心规则；用相同契约案例验证真实 Adapter，再保留少量边界集成测试。
6. 模拟更换 HTTP Client、存储或第三方 SDK；核心用例和测试不应改变。
7. 若新增层长期只是原样转发、没有隔离任何变化，就合并它，避免为假想替换点持续付费。

## 别误解

- **Adapter 就是把 `fetch` 放进 Service。** 只有外部协议被转换为内部语义，并且不再向外泄漏时，才形成适配边界。
- **Repository 是所有数据请求的标准名字。** 它强调领域对象集合；命令式能力应使用符合真实语义的 Port。
- **依赖注入必须使用类和容器。** 普通函数参数已经能显式提供依赖，Container 只是对象图工具。
- **为了测试必须 Mock 每个模块。** 应替换真正的外部 Port，并通过契约测试防止 Fake 与真实实现分叉。
- **抽象越多越容易替换。** 只有契约由稳定需求定义且转换责任清楚时，间接层才减少变化传播。

## 复习

1. 为什么把 HTTP 调用移动到 `Service` 不一定隔离了变化？
2. Port、Adapter、Repository、DI 和 Composition Root 分别负责什么？
3. 为什么 `PaymentPort` 通常比 `PaymentRepository` 更准确？
4. 全局 Service Locator 与显式注入在依赖可见性上有什么差别？
5. 怎样用“替换测试”判断一个抽象是否真正有价值？

## 来源与下一步

- **来源事实：** Cockburn 的 Ports and Adapters 原文用内部—外部边界统一解释 UI、测试、数据库和外部服务的替换，并让 Adapter 在 Port API 与具体设备之间转换；Fowler 将 DI 描述为由外部组装者提供实现，并强调配置与使用分离；Repository 模式原条目把它定义为领域层与数据映射层之间的集合式对象访问接口；课程材料使用容器集中注入 Service、Controller 并管理生命周期。
- **本页推论：** 将 Port、Adapter、Repository、DI、Container 和 Composition Root 放回“契约定义—协议转换—实现选择—生命周期”链条，并把 Repository 限定为特定数据访问语义，而非所有请求的统一后缀。
- **工程建议：** 优先使用调用方定义的最小 Port、显式函数或构造注入和集中 Composition Root；只有真实对象图复杂度出现后才引入 Container，并用替换测试、契约测试和生命周期测试验证收益。
- **下一步：** [[23：运行时 Schema 与错误模型|运行时 Schema 与错误模型]]，继续说明 Adapter 收到不可信外部值后，怎样把结构解析、语义错误和恢复决策转成稳定内部契约。
- **来源：** [[raw/links/2026-08-21-Cockburn Hexagonal Architecture|Cockburn：Hexagonal Architecture]]、[[raw/links/2026-08-21-Fowler Dependency Injection|Fowler：Dependency Injection]]、[[raw/links/2026-08-21-Fowler Repository Pattern|Fowler Catalog：Repository Pattern]]、[[wiki/直播课/课程笔记/11-前端架构设计指南⑤|前端架构设计指南⑤]]
