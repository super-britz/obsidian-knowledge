---
type: 概念建模
status: 可用
created: 2026-08-21
updated: 2026-08-21
sources:
  - "raw/live/10-前端架构设计指南④.txt"
  - "raw/links/2026-08-21-TypeScript Erased Types.md"
  - "raw/links/2026-08-21-OWASP Input Validation Cheat Sheet.md"
  - "raw/links/2026-08-21-JSON Schema 2020-12.md"
  - "raw/links/2026-08-21-RFC 9457 Problem Details.md"
  - "raw/links/2026-08-21-WHATWG Fetch Standard.md"
related:
  - "wiki/前端架构设计/00：索引.md"
  - "wiki/前端架构设计/13：可信边界与副作用隔离.md"
  - "wiki/前端架构设计/21：状态机与异步流程.md"
  - "wiki/前端架构设计/22：Adapter、Repository 与依赖注入.md"
  - "wiki/前端架构设计/24：失败边界、恢复与可观测性.md"
---

# 运行时 Schema 与错误模型

> [!abstract] 先用一句话说清楚
> Runtime Schema 在外部值进入系统时执行一次可观察的准入判断，把 `unknown` 转成满足边界条件的内部值；错误模型则保留那些会改变责任和恢复动作的失败事实。前者不是再写一遍 TypeScript 类型，后者也不是给所有异常统一配一条 `message`。

## 从“程序无法继承外部保证”推导

TypeScript 能检查当前代码怎样使用一个值，但类型会在编译后擦除。网络响应、URL、缓存、跨窗口消息和第三方 SDK 到达运行时后，真实值只由外部世界决定：

    外部字节或 JavaScript unknown
    → 反序列化：能否读成 JSON 等数据
    → 结构 Schema：形状、类型、必需字段和局部约束是否成立
    → 规范化与协议映射：单位、日期、命名和缺省值转成内部语义
    → 业务、权限、版本与并发判断
    → 当前上下文可使用的内部模型

直接写 `response as Order` 只是让编译器停止追问。错误通常会在更远处以“无法读取属性”表现，根因与故障位置分离。Runtime Schema 的价值，就是在信任刚要增加的位置产生明确证据：**这个具体值通过了哪些检查，或者在哪个路径因为什么规则被拒绝。**

但只得到“失败了”仍不够：

    不同失败全部压成 Error(message)
    → 身份、责任和远端执行事实丢失
    → 状态机不知道应回到编辑、重新登录、刷新、等待还是查询结果
    → UI 和重试库只能猜测

因此 Schema 负责建立值的边界，错误模型负责保留失败之后仍需决策的信息。

## Schema 只证明它声明的那一层

| 机制 | 证明什么 | 不能证明什么 |
| --- | --- | --- |
| TypeScript 类型 | 受控源码中的使用关系在编译期一致 | 运行时值真实符合声明 |
| Runtime Schema | 当前值满足已执行的结构、格式和范围规则 | 业务允许、主体有权、数据仍然新鲜 |
| 领域规则 | 当前上下文中的业务不变量成立 | 外部操作最终成功 |
| 认证与授权 | 主体身份及对目标动作的权限 | 响应结构正确或写入没有竞争 |

Schema 本质上是一个可执行分类器：

```ts
type SchemaIssue = {
  path: ReadonlyArray<string | number>;
  code: 'required' | 'type' | 'range' | 'unknownVariant';
};

type DecodeResult<T> =
  | { ok: true; value: T }
  | { ok: false; issues: SchemaIssue[] };

declare function decodeOrder(input: unknown): DecodeResult<OrderDTO>;
```

它的成功输出应是经过筛选和转换的新值，而不是继续传播原始对象。失败输出保留机器可判断的路径和代码；面向用户的本地化文案在展示层生成，遥测则只保存脱敏后的诊断信息。

JSON Schema 是可序列化契约的一种实现，不等于“Runtime Schema”这个更一般的机制。使用 JSON Schema 时还要固定方言、词汇表和验证器能力；例如 `format` 在不同配置下可能只是 Annotation，而不是强制断言。契约存在不代表所有运行环境真的执行了同一规则。

## 同一份数据不应该只有一个 Schema

表单、传输对象和领域模型看起来字段相似，但有效集合和变化原因不同：

| 边界 | 为什么它的有效集合不同 |
| --- | --- |
| 表单草稿 | 允许空值、字符串数字和输入中的临时不完整状态 |
| 提交命令 | 必须满足本次操作所需的完整输入 |
| HTTP 响应 DTO | 服从服务端协议、兼容窗口和序列化格式 |
| 领域模型 | 已规范化，并满足内部代码依赖的不变量 |
| 本地缓存 | 还要识别历史版本、迁移状态和过期数据 |

强行复用一个 Schema，会让“用户还没输完”变成领域非法，或让后端 DTO 的可选字段污染内部模型。可以共享稳定的字段片段，但每个信任边界应拥有最终准入规则和转换责任。

对未知字段采取严格拒绝还是兼容忽略，也不是工具偏好：接收命令时严格白名单有助于暴露拼写和意外输入；消费可演进响应时通常可以忽略新增字段，但只投影已知字段进入内部。如果出现未知判别值，而程序不知道对应行为，就应当失败或降级，不能假装它属于已有分支。

## 错误分类由恢复动作反推

错误类别不是越细越好。最小错误模型只保留那些会改变以下决策的差异：

    谁负责处理
    + 用户能做什么
    + 程序能否自动恢复
    + 状态机进入哪里
    + 需要留下什么诊断证据

| 稳定语义 | 已知事实 | 合理动作 |
| --- | --- | --- |
| `invalidInput` | 操作尚未按有效输入执行 | 标出字段，允许用户修改 |
| `domainRejected` | 输入被理解，但业务条件拒绝 | 解释原因或提供替代选择 |
| `unauthenticated` / `forbidden` | 身份缺失或没有权限 | 重新认证，或停止并申请权限 |
| `conflict` | 依据的版本或状态已经过期 | 获取权威状态，再合并或重做决定 |
| `rateLimited` / `unavailable` | 当前不能完成，但可能稍后恢复 | 遵守等待提示；只有操作安全时才重试 |
| `outcomeUnknown` | 写操作可能已发生，当前无法证明结果 | 优先用 Operation ID 查询；仅在同身份重放已被契约保证安全时重试 |
| `contractViolation` | 外部响应不能转换成内部承诺 | 安全降级，记录契约版本和追踪 ID |
| `unexpected` | 内部不变量或代码假设被破坏 | 由失败边界兜底并交给工程诊断 |

同一个底层网络错误，对读取可能映射为 `unavailable`，对已经发出的写操作却可能是 `outcomeUnknown`。所以 Adapter 应先保留“有没有响应、HTTP 状态、问题类型、Retry-After、操作身份、响应 Schema 是否通过”等事实，再由用例按业务语义映射；不能由全局拦截器看到异常名称就统一重试。

RFC 9457 的 Problem Details 提供了 HTTP 错误载荷的通用外形。其中 `type` 是主要机器标识，`detail` 是面向人的单次说明，不应被字符串解析为控制逻辑。外部 `type` 仍应在 Adapter 中映射成本地稳定错误，而不是让组件依赖供应商错误码。

## 一个提交操作的边界结果

`throw` 还是返回 Result 只是控制流选择，不能代替语义分类。预期会发生且调用者能够处理的结果，通常用判别联合更容易穷尽状态转换：

```ts
type SubmitOrderResult =
  | { kind: 'accepted'; orderId: string }
  | { kind: 'invalidInput'; issues: SchemaIssue[] }
  | { kind: 'rejected'; reason: 'outOfStock' | 'limitExceeded' }
  | { kind: 'conflict'; currentVersion: string }
  | { kind: 'rateLimited'; retryAfterMs?: number }
  | { kind: 'outcomeUnknown'; operationId: string }
  | { kind: 'contractViolation'; traceId?: string };
```

边界处理顺序可以复述为：

    Fetch 网络层事实
    → 检查 HTTP 状态和媒体类型
    → 把响应体当 unknown 解析
    → 成功体或 Problem Details 分别通过 Schema
    → Adapter 映射外部协议
    → 用例生成本地 Result
    → 状态机选择 editing、reauthenticating、reconciling、waiting 或 failed

响应成功但成功体 Schema 失败，应是 `contractViolation`，不是让用户修改表单；用户输入 Schema 失败才是 `invalidInput`。同一校验机制落在哪个边界，决定了错误责任。

## 兼容性、性能与新增成本

Schema 本身也是版本化代码。字段改名、必需字段增加、判别联合新增成员、默认值变化和日期解析策略都可能改变被接受的值集合。发布时应验证旧生产样本、新旧客户端组合、缓存历史版本和未知字段策略，而不是只测试一个最新成功样例。

运行时校验还会增加包体、CPU、Schema 维护和错误映射成本。最经济的位置通常是信任入口：值成功转换一次后，内部模块共享已建立的不变量；若每次渲染和每个内部函数都重复校验，往往说明信任边界或状态所有权仍不清楚。

错误模型也有过度设计风险。把数据库驱动、HTTP Client 和供应商的每个错误码都暴露给业务，会让调用方依赖易变细节；全部压成 `unknown` 又失去恢复能力。判断标准始终是：这一差异是否会改变合法状态转换、用户动作、自动恢复或故障归属。

## 可观察的错误信号

| 现象 | 可能的问题 |
| --- | --- |
| 网络、URL 或缓存数据直接 `as DomainType` | 静态声明代替了运行时准入 |
| Schema 验证后仍传播原始对象 | 边界可以被后续代码绕过 |
| 表单、DTO、领域和缓存共用一个 Schema | 不同生命周期被错误耦合 |
| 代码解析 `error.message` 决定流程 | 人类文案被误当成稳定机器标识 |
| 所有错误都进入同一个 `isError` | 状态机无法选择不同恢复路径 |
| 任何 Fetch 异常都自动重放 POST | 网络事实被误解为副作用未发生 |
| 响应 Schema 失败却显示“请修改表单” | 错误责任归给了错误主体 |
| 日志记录完整非法载荷和 Token | 可诊断性制造了隐私与安全风险 |
| 内部每层重复验证同一对象 | 信任边界不清，持续支付重复成本 |

## 做项目时怎么验证

1. 枚举真实输入入口：HTTP、URL、Storage、消息、实时推送、配置和第三方 SDK，并默认标为 `unknown`。
2. 为每个入口定义最小 Schema、方言或库版本、未知字段策略，以及成功后的内部投影和规范化。
3. 按“用户动作—自动恢复—状态转换—责任主体”列矩阵，只保留会改变这些决策的错误类别。
4. 在 Adapter 中映射 HTTP 状态、稳定问题类型和结构化扩展；不解析 `message`、`title` 或 `detail` 驱动逻辑。
5. 把错误结果接入 [[21：状态机与异步流程|状态机]]，尤其测试冲突、限流、写结果未知和成功体契约损坏。
6. 用缺字段、错类型、超范围、未知判别值、新增字段、旧缓存和非 JSON 响应做契约测试。
7. 检查遥测只保留 Schema 版本、问题代码、字段路径和关联 ID 等必要证据，并对原始值和身份信息脱敏。
8. 若某个错误类别从不改变处理动作，合并它；若一个通用错误触发多处猜测和字符串判断，再拆分它。

## 别误解

- **前后端共用类型就不需要 Runtime Schema。** 共享类型减少开发期漂移，不能验证已经到达运行时的值。
- **Schema 通过就代表数据可信。** 它只证明已声明且已执行的规则，不证明授权、业务状态和副作用结果。
- **一个实体应该只有一个 Schema。** 表单、协议、领域和缓存处在不同信任边界，有不同有效集合。
- **HTTP 状态码就是完整错误模型。** 状态提供协议级分类，业务恢复还需要稳定问题身份、操作语义和上下文。
- **异常都应该抛出，或都应该返回 Result。** 控制流形式不是关键；关键是边界是否保留了调用者做正确决定所需的事实。

## 复习

1. 为什么 TypeScript 类型不能证明网络响应在运行时有效？
2. 反序列化、结构 Schema、规范化、业务规则和授权各自证明什么？
3. 为什么表单、HTTP DTO、领域模型和缓存不应强行共用同一 Schema？
4. 为什么同一个网络失败对读取可能是 `unavailable`，对写入却可能是 `outcomeUnknown`？
5. 如何判断两个错误应该合并，还是必须保留为不同类别？

## 来源与下一步

- **来源事实：** TypeScript 官方文档说明类型会在编译后擦除；OWASP 区分结构或语法校验与业务语义校验，并要求不可信输入尽早校验；JSON Schema 2020-12 将 Core 与 Validation 分开，并允许方言和词汇表影响断言行为；RFC 9457 规定 Problem Details 的机器标识、面向人说明和扩展成员；Fetch Standard 分开处理网络错误与非成功 HTTP 状态；课程材料强调类型安全 API 不替代运行时校验、权限与业务规则。
- **本页推论：** 将 Runtime Schema 建模为“unknown → 结构证据 → 规范化内部值”的可执行信任转换，并从责任、用户动作、自动恢复、状态转换和诊断需要反推最小错误模型。
- **工程建议：** 只在真实信任入口验证一次，按边界分别拥有 Schema；让 Adapter 保留协议事实并映射到本地判别联合，用契约样本、兼容窗口和状态转换测试验证，而不是依赖类型断言、错误文案或全局重试。
- **下一步：** [[24：失败边界、恢复与可观测性|失败边界、恢复与可观测性]]，继续说明单次错误已经被正确分类后，怎样限制影响范围、组织恢复并留下可定位证据。
- **来源：** [[raw/links/2026-08-21-TypeScript Erased Types|TypeScript Erased Types]]、[[raw/links/2026-08-21-OWASP Input Validation Cheat Sheet|OWASP Input Validation Cheat Sheet]]、[[raw/links/2026-08-21-JSON Schema 2020-12|JSON Schema 2020-12]]、[[raw/links/2026-08-21-RFC 9457 Problem Details|RFC 9457：Problem Details for HTTP APIs]]、[[raw/links/2026-08-21-WHATWG Fetch Standard|WHATWG Fetch Standard]]、[[wiki/直播课/课程笔记/10-前端架构设计指南④|前端架构设计指南④]]
