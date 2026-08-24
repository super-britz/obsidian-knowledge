---
type: 概念建模
status: 可用
created: 2026-08-24
updated: 2026-08-24
sources:
  - "raw/live/08-前端架构设计指南③.txt"
  - "raw/links/2026-08-24-Google Monorepo Paper.md"
  - "raw/links/2026-08-24-pnpm Workspace.md"
  - "raw/links/2026-08-24-Nx Affected Project Graph.md"
  - "raw/links/2026-08-24-Bazel Remote Caching.md"
  - "raw/links/2026-08-21-Node.js Package Entry Points.md"
  - "raw/links/2026-08-21-Nx Enforce Module Boundaries.md"
  - "raw/links/2026-08-21-GitLab Code Owners.md"
  - "raw/links/2026-08-21-Team Topologies Key Concepts.md"
related:
  - "wiki/前端架构设计/00：索引.md"
  - "wiki/前端架构设计/10：按变化原因建立边界.md"
  - "wiki/前端架构设计/11：让依赖指向稳定规则.md"
  - "wiki/前端架构设计/15：可逆决策与演进边界.md"
  - "wiki/前端架构设计/16：组织所有权与系统边界.md"
  - "wiki/前端架构设计/20：Feature-based 模块与公共 API.md"
  - "wiki/前端架构设计/25：缓存与数据一致性.md"
  - "wiki/前端架构设计/27：渐进迁移与兼容窗口.md"
  - "wiki/前端架构设计/32：BFF 与客户端场景边界.md"
---

# Monorepo 与代码协作边界

> [!abstract] 先用一句话说清楚
> Monorepo 是把多个可独立描述的项目放进同一个版本控制仓库，用统一源码快照、依赖图和自动化降低跨项目变更的发现、联调与验证成本；它没有消除耦合，而是把一部分跨仓版本协调搬成同仓依赖治理。只有高频联合变化带来的收益，大于仓库规模、权限、CI、发布和平台维护成本时，它才是合理的协作形态。

## 为什么 Monorepo 属于 L5

[[20：Feature-based 模块与公共 API|Feature-based 模块与公共 API]]已经说明怎样用目录、入口和依赖规则建立代码边界；Workspace、Package Manager、Task Runner、Remote Cache 也都是可以分别理解的机制。

Monorepo 属于 L5，是因为一个可运行的 Monorepo 会同时组合：

- 多个应用与 Package；
- 逻辑模块、公共 API 与依赖方向；
- 统一或分层的工具配置；
- Project Graph 与 Task Graph；
- Affected CI、并行执行和构建缓存；
- Package 版本、应用发布与兼容迁移；
- 路径所有权、评审权限和平台责任。

只运行 `pnpm init` 或把几个目录移动到同一 Git 仓库，只得到了一种文件摆放。只有上述边界与反馈共同成立，仓库形态才真正改变团队的协作成本。

## 第一性原理：优化的是“一次变化”的总成本

用户需求不会因为代码分散在几个仓库就停止跨边界。一次真实变化可能经过：

    发现受影响项目
    → 理解契约和所有者
    → 修改生产者与消费者
    → 在本地组合正确版本
    → 验证直接和间接影响
    → 产生 Package 或应用产物
    → 按兼容顺序发布
    → 观察并恢复失败

可以用一个定性模型思考：

    一次变化总成本
    ≈ 发现成本
      + 修改成本
      + 跨边界协调成本
      + 验证成本
      + 交付成本
      + 失败恢复成本

Polyrepo 把边界做成多个版本控制与发布单元。它能带来清楚的访问隔离、较小仓库和独立工具链，但跨仓变化需要：

    仓库 A 先修改并发布新版本
    → 仓库 B、C 分别升级
    → 多个 PR 与流水线各自排队
    → 在兼容窗口内处理版本组合

Monorepo 让生产者和可控消费者在同一源码快照中修改和验证：

    修改 Package A
    → 依赖图找到 B、C
    → 一个变更集同时更新
    → CI 验证受影响闭包

它减少的是**已有跨项目变化的协调摩擦**，不是所有工程成本。仓库变大后，依赖发现、验证范围、构建缓存、根配置治理和权限本身又成为新的公共基础设施。

## 先把七种边界分开

“都在一个仓库”最容易让人误以为所有东西都应该一起变化。实际上至少有七种正交边界：

| 边界 | 回答的问题 | Monorepo 是否自动统一 |
| --- | --- | --- |
| Repository | 哪些文件共享一次 Commit、历史和根策略？ | 是 |
| Project / Package | 哪些源码、依赖和任务构成一个可识别项目？ | 否，需要显式声明 |
| Module / Public API | 外部允许依赖哪些能力？ | 否，需要入口与规则 |
| Build Task | 哪些输入产生哪个输出，怎样排序与缓存？ | 否，需要 Task Graph |
| Release | 哪些产物使用同一版本、Changelog 和发布节奏？ | 否，可固定或独立版本 |
| Deploy / Runtime | 哪些应用能独立部署、扩容、失败和回滚？ | 否 |
| Team / Access | 谁能看、改、审批、发布并处理事故？ | 否 |

因此：

- Monorepo 不等于 Monolith；
- 一个仓库可以包含多个独立部署的服务和前端；
- 一个 Package 可以只在仓库内消费，也可以发布给外部仓库；
- 同一次 Commit 可以构建多个版本和多个部署单元；
- 同一个团队也可以拥有多个仓库。

仓库只是协作与源码事务边界。业务、模块、发布和运行边界是否合理，仍要分别证明。

## Monorepo 不消除耦合，只改变耦合的支付位置

如果修改设计 Token 时三个应用都必须变化，这种共同变化已经存在。Polyrepo 用版本发布和升级排期支付它；Monorepo 用同仓修改、依赖图和集中验证支付它。

反过来，把本来无关的两个系统放入一个仓库，不会产生有意义的复用，却会让它们共享：

- 根 Lockfile 和工具升级；
- CI 队列与缓存基础设施；
- 仓库权限与合并策略；
- 根配置错误和全局脚本的爆炸半径；
- 开发者获取、搜索和理解仓库的成本。

Monorepo 甚至可能放大隐藏耦合：

    所有源码都近在眼前
    → 直接 Deep Import 或读取别人的内部类型很容易
    → 内部细节变成事实契约
    → 任何重构都影响大量消费者

因此真正目标是：

> 让应该一起变化的代码容易一起改，同时让不应一起变化的代码仍被公共 API、依赖方向、所有权和发布协议隔开。

## 用变化频率与隔离要求判断

选择仓库形态前，先看两个主要变量：

1. **联合变化频率：** 最近的真实需求中，有多少需要同时修改多个项目？
2. **强隔离要求：** 是否存在不同访问权限、法规、客户、供应商、技术栈或生命周期？

| 联合变化 | 隔离要求 | 更可能的方向 |
| --- | --- | --- |
| 高频 | 较低 | Monorepo 更可能减少版本等待和重复验证 |
| 低频 | 较高 | Polyrepo 更自然地表达权限与自治 |
| 高频 | 较高 | 不能靠仓库二选一解决；考虑领域分组仓库、版本化契约和自动同步 |
| 低频 | 较低 | 两者都可，选择当前团队最简单、最熟悉的形态 |

还要把规模能力放入判断：

    Monorepo 净价值
    ≈ 联合变化频率
      × 每次跨仓协调可节省成本
      - 依赖治理、CI、缓存、权限和平台维护成本

Google Research 的案例把 Monorepo描述为共同源码事实来源，同时明确讨论了使超大仓库可工作的专门系统与工作流。值得迁移的是“收益必须与工程系统一起评估”，不是“Google 使用所以所有团队都应该使用”。

## 一个可治理仓库先有 Project，再有共享

常见目录只是起点：

```text
apps/
  customer-web/
  admin-web/
  mini-program/

packages/
  design-system/
  domain-contracts/
  api-client/

tooling/
  eslint-config/
  tsconfig/
  build-presets/
```

每个 Project 至少应该声明：

| 契约 | 要回答什么 |
| --- | --- |
| 职责 | 它拥有哪类变化，不拥有什么？ |
| Public API | 消费者允许从哪些入口使用？ |
| Dependencies | 直接依赖谁，允许指向哪些层？ |
| Owner | 谁理解、评审和修复它？ |
| Tasks | 怎样 Build、Test、Lint、Typecheck？ |
| Outputs | 哪些产物可以缓存、发布或部署？ |
| Release | 仅随源码消费、独立版本还是固定版本组？ |
| Runtime | 被编进应用、独立运行还是只在构建期使用？ |

`apps/` 与 `packages/` 不必成为强制命名；重要的是应用负责装配和交付，Package 提供边界清楚的能力，而不是依赖文件夹看起来整齐。

### `shared/` 不是垃圾桶

两段代码相似，只能证明它们当前长得像，不能证明未来会因同一原因变化。抽取共享 Package 的条件更接近：

    多个消费者需要相同语义
    ∧ 由同一所有者维护
    ∧ 变化与兼容策略一致
    ∧ 公共 API 比复制更稳定

设计 Token、协议 Schema 和稳定基础组件通常较容易满足；某两个页面当前相似的业务状态、路由和请求编排未必满足。少量重复有时比错误共享更可逆。

### 可见不等于可依赖

同仓源码可以被搜索和阅读，但调用方只应通过 Package 公开入口依赖。Node.js `exports`、Lint Rule 或 Project Tag 可以限制 Deep Import 和错误方向；依赖还要在各自 Manifest 中直接声明，不能因为根目录恰好能解析就形成幽灵依赖。

pnpm 的 `workspace:` 协议可以保证某项依赖必须解析到当前 Workspace Package，并在本地版本不存在时失败。它解决的是依赖解析确定性，不会自动设计正确 API，也不会证明发布后的外部安装路径可用。

循环依赖尤其要警惕：

    A 依赖 B，B 又依赖 A
    → 谁先构建、谁拥有规则、谁能独立发布都不清楚

pnpm 文档也说明循环 Workspace 依赖会破坏可靠的拓扑执行顺序。工具可以报出环，真正修复仍要重新确定编排层、稳定规则或所有权。

## Project Graph 决定变化传播

一个 Monorepo 的可管理性取决于它能否回答：

> 修改这个节点，哪些直接和间接消费者可能被破坏？

假设依赖方向是：

```text
customer-web ─┐
admin-web ────┼→ api-client → domain-contracts
mini-program ─┘
```

改变 `domain-contracts` 时，受影响集合不是它的下游依赖，而是沿反向依赖边找到所有消费者。改变 `customer-web` 内部页面，则不应触发其他应用验证。

完整控制链是：

    Git Changed Files
    → 文件属于哪些 Project
    → Project Graph 求消费者闭包
    → Task Graph 加上任务前置关系
    → 对闭包运行 Build / Test / Lint / Typecheck
    → 复用输入相同的缓存结果

Nx 官方把 Project Graph 与 Task Graph 分开，并用 Git 变化与依赖图计算 Affected Projects。这个模型可迁移到任何工具：源码依赖回答“谁可能受影响”，任务依赖回答“验证与构建按什么顺序运行”。

### Affected 不是按目录猜

只运行改动目录的测试会漏掉消费者；只要一个共享 Package 变化，其所有相关消费者都属于影响闭包。根 TypeScript 配置、构建插件或 Lockfile 改变也可能影响全仓。

正确的 Affected Pruning 依赖：

- Project 与文件归属完整；
- Static、Dynamic、Generated Dependencies 都被建模；
- 根配置、环境、工具链与 Lockfile 被标为全局或定向输入；
- CI 使用正确 Base / Head，失败期间不会跳过尚未验证的 Commit；
- 不确定时选择扩大验证，而不是乐观漏判。

因此可以用分层验证：

    PR：快速 Affected Checks
    → Main：合并后关键集成与契约检查
    → 周期或发布前：全量、跨版本与高风险验证

Graph 用于减少已知无关工作，不是证明所有未运行测试都绝对无关。

## Build Cache 是正确性协议

仓库变大后，重复构建相同 Package 会浪费大量时间。Remote Cache 只有在以下关系成立时才安全：

    Cache Key
    = Source
      + Direct / Transitive Dependencies
      + Config
      + Environment
      + Toolchain
      + Command

    相同完整输入
    → 可复用相同输出

Bazel 文档把 Build 拆为显式声明输入、输出、命令与环境的 Action，并在构建可复现时共享输出。若任务实际读取了未声明环境变量、当前时间、网络响应或工作区外文件，Cache Key 就不能代表真实输入：

    Hash 相同
    但隐藏输入不同
    → 错误 Cache Hit
    → CI 可能发布与源码不匹配的旧产物

所以缓存不是“开一个远端服务”：

- 任务要尽可能 Hermetic 和可复现；
- 输出目录必须明确，不能把源码或密钥混入；
- 开发机与 CI 的 Runtime、工具链差异要进入 Key；
- 不可信分支的写权限、缓存投毒和敏感日志需要隔离；
- 命中率要和错误复用、冷构建时间一起观察。

这和 [[25：缓存与数据一致性|数据缓存]]遵循同一原理：Build Output 也是由输入推导出的副本；Key、版本、权威重算与清理决定它能否被信任。

## 同一 Commit 不等于同时发布

Monorepo 可以在一个 Commit 中同时修改 Package 与消费者，保证源码快照内部一致。但交付链仍有多个状态：

    Source Commit
    → Package Build Artifact
    → Registry Version
    → Application Artifact
    → Environment Deployment
    → 用户实际运行版本

这些状态不会原子切换。

### 三种常见版本关系

| 关系 | 适用条件 | 主要责任 |
| --- | --- | --- |
| 源码同快照消费 | 所有消费者都在仓库内构建，不对外发布 | Commit 可重现、Affected 验证与部署兼容 |
| Package 独立版本 | 消费者需要分别升级或存在仓库外使用方 | SemVer、Changelog、弃用期、发布顺序 |
| 固定版本组 | 一组 Package 对外形成一个产品面，独立组合没有价值 | 共同版本、共同 Release Gate 和更大发布范围 |

一个共享 Lockfile 统一某次依赖解析结果，不代表所有 Package 必须同版本；Package 在同一仓库，也不代表必须同一天发布。pnpm 本身也明确把多 Package 的版本管理视为独立问题。

### 原子改源码仍要兼容运行版本

假设同一 PR 同时修改 `domain-contracts`、Web 和小程序：

- Web 可以今天部署；
- 小程序审核后才发布；
- 用户设备还会长期运行旧版本；
- 服务端也可能滚动发布。

因此 API、事件和持久数据仍要遵循 [[27：渐进迁移与兼容窗口|渐进兼容]]。Monorepo 缩短了源码协作链，不能让分布式运行状态原子切换。

对发布到 Registry 的 Package，还要在隔离环境验证实际 Tarball、`exports`、声明依赖和类型产物；Workspace 本地链接成功可能隐藏“忘记打包文件”或“依赖未声明”等发布问题。

## 所有权与访问边界决定仓库能否成立

仓库统一后，根工具、Lockfile、CI 和共享 Package 会成为所有团队的公共依赖。所有权至少分三层：

| 范围 | 主要责任 |
| --- | --- |
| 业务 Project | 业务语义、Public API、测试、发布与运行结果 |
| 共享 Package | 消费者契约、兼容性、文档、版本和支持 |
| Workspace Platform | 根配置、Dependency Policy、Task Graph、Cache、CI 与迁移工具 |

`CODEOWNERS` 可以把路径映射到评审人并执行审批，但不能代替上下文、发布权限和故障响应。根配置也不能变成“所有人都能改、没人负责”的公共区域。

另一方面，单仓通常意味着更宽的代码可见范围。若项目受客户合同、外部供应商、出口管制或敏感算法限制，依靠目录和 Review 规则不一定足以表达访问隔离。仓库策略必须先满足最小权限与审计要求，不能为了本地联调方便扩大读取面。

治理也不能走向另一个极端。若一个普通业务改动要获得所有共享团队批准，Monorepo 只是把跨仓等待搬进一个巨大的 PR。稳定公共 API、自动约束和明确变更类别，应让大多数日常变化由 Project Owner 闭环。

## 一个多端项目怎样推导

假设产品包含：

```text
apps/customer-web
apps/admin-web
apps/mini-program
packages/domain-contracts
packages/api-client
packages/design-system
tooling/build-presets
```

不能因为三个端都使用 TypeScript，就把所有业务代码提取为共享 Package。逐项判断：

| 候选共享项 | 更合理的判断 |
| --- | --- |
| API Schema 与错误映射 | 多端消费同一协议，适合共享稳定契约或生成源 |
| Design Token 与基础组件 | 视觉语义和所有权相同才共享；平台差异通过适配层处理 |
| 页面 Store 与路由状态 | 生命周期和平台交互不同，通常留在各应用 |
| 核心领域计算 | 不依赖平台且业务规则确实相同，可提取并用契约测试保护 |
| 构建配置 | 相同规则可复用 Preset，但保留应用必要覆盖，避免万能 Root Config |

一次 API 字段演进可以：

    服务端先兼容旧、新协议
    → 同一 PR 更新 domain-contracts 与 api-client
    → Project Graph 找到三个消费者
    → Affected Tests 验证各端映射
    → 三个应用按自己的发布节奏上线
    → 旧客户端退出后再删除兼容字段

Monorepo 在这里降低源码发现和联调成本；版本兼容仍由协议和发布系统负责。若把三个端的页面状态也强行共享，短期少写代码，长期每个平台需求都会扩散到其他端。

## 迁移到 Monorepo 应从一条变化流开始

不要因为“统一技术栈”就一次搬入所有仓库。更可验证的顺序是：

1. 统计最近一段时间跨仓需求、重复配置、版本等待和联调故障；
2. 选择一组联合变化频繁、访问权限相容的项目作为 Pilot；
3. 迁入时保留原有业务、Owner、Release 和 Runtime 边界；
4. 声明 Project、Public API、直接依赖、任务输入输出与禁止方向；
5. 先建立可靠全量验证，再用 Graph 逐步缩小 Affected 范围；
6. 让 Build 可复现后再启用本地和 Remote Cache；
7. 分开设计内部源码消费、Package 发布和应用部署；
8. 记录兼容、回退与重新拆仓的代价，按交付证据决定是否扩大。

Monorepo 的退出成本也不是零：拆仓要恢复版本历史、Package 发布、权限、CI、依赖更新与兼容流程。因此它仍属于 [[15：可逆决策与演进边界|需要按证据渐进采用的结构决策]]。

## 做项目时怎么验证

1. 选最近十次跨项目变化，记录涉及仓库数、PR 数、等待时间、版本组合、失败与返工。
2. 画出当前源码依赖、发布依赖和团队交接图，区分真实联合变化与仅仅代码相似。
3. 为每个 Project 声明职责、入口、Owner、任务、产物、版本、部署和故障责任。
4. 用 Deep Import、未声明依赖、循环、错误方向和超大公共 Package 检查边界是否真实。
5. 从已知变更计算 Affected Closure，故意修改共享配置、Lockfile、生成器和环境输入，验证不会漏跑。
6. 比较 Cold Build、Warm Build、PR P50/P95、Queue Time、Affected Ratio 和 Remote Cache 命中，同时注入错误 Cache 与不可信写入。
7. 在临时空目录安装发布 Package，验证 Tarball、类型、`exports`、直接依赖与消费者升级路径。
8. 分别部署不同应用版本，验证同一 Commit 的消费者也能在新旧运行版本共存时保持兼容。
9. 审计仓库读取、路径评审、根配置修改、发布和缓存权限，确认满足最小权限。
10. 持续比较跨项目 Lead Time、变更失败率和平台维护投入；收益不足时停止扩大或重新分组。

## 可观察的错误信号

| 现象 | 更可能缺少的边界或机制 |
| --- | --- |
| 每个 PR 都运行全仓所有任务 | Project / Task Graph 或输入声明不足 |
| Affected CI 很快但线上经常漏回归 | 依赖图存在隐藏输入或动态依赖 |
| 修改一个 `shared` Package 影响所有应用 | 共享职责过宽，边界按“可复用”而非变化原因建立 |
| 大量跨 Package Deep Import | Public API 和自动约束失效 |
| 本地联调成功，发布 Package 后缺文件 | Workspace 链接掩盖了真实发布产物 |
| 根 Lockfile 或配置频繁阻塞全仓 | 全局输入爆炸半径缺少治理 |
| Remote Cache 偶发产出旧结果 | Hash 漏掉环境、工具链或生成输入 |
| 同一 Commit 的两个应用上线后仍不兼容 | 把源码原子性误认为部署原子性 |
| 普通改动需要许多 Code Owner 审批 | 所有权范围过宽或公共 API 不稳定 |
| 某团队不应看到的代码进入同仓 | Repository 访问边界与组织安全约束冲突 |
| 平台团队每天修根脚本和 CI | 仓库规模超过当前工程系统能力 |

## 别误解

- **Monorepo 就是一个大项目。** 它应包含多个显式 Project；没有边界只会成为 Monolith Source Tree。
- **代码放在一起就更容易复用。** 可见性也更容易产生错误依赖；共享必须有共同语义、Owner 和公共 API。
- **一个 Lockfile 就代表所有 Package 同版本。** Lockfile记录解析快照，版本与发布策略仍可独立。
- **一个 Commit 能同时改完，所以能同时上线。** Commit 只保证源码快照，构建、发布、部署和用户升级都有独立时间线。
- **Affected CI 自动保证不漏测。** 它只对已建模依赖可靠；隐藏输入必须进入 Graph 或扩大验证。
- **Remote Cache 只是性能优化。** 错误 Key 会复用错误产物，读写权限也属于供应链边界。
- **Monorepo 天然适合微服务或微前端。** 仓库与 Runtime 是正交维度，运行自治要另行设计。
- **重复代码都应该提取到 `shared`。** 错误共享会把原本独立的变化绑在一起，少量重复有时成本更低。
- **大厂采用就说明规模越大越应该单仓。** 大规模 Monorepo 的收益依赖专门版本控制、构建、搜索和治理投入。

## 复习

1. Monorepo 优化的是一次变化链中的哪些成本，又新增了哪些公共成本？
2. Repository、Package、Release、Deploy 和 Team 为什么不能视为同一边界？
3. 为什么 Monorepo 不消除语义耦合，只改变协调成本的支付位置？
4. 什么样的代码相似性不足以证明应该抽取共享 Package？
5. Project Graph 与 Task Graph 分别回答什么问题？
6. 为什么 Affected CI 必须计算消费者闭包，而不能只看改动目录？
7. Remote Cache Key 漏掉环境或工具链会发生什么？
8. 为什么同一 Commit 修改所有消费者，仍然需要运行时兼容窗口？
9. 哪些访问和组织条件会让 Polyrepo 更合理？
10. 应用、共享 Package 与 Workspace Platform 分别由谁承担什么责任？

## 来源与下一步

- **来源事实：** 课程材料展示多应用、UI、API、环境与基础设施 Package 组成的 Workspace，并提到 pnpm Workspace、本地联调与私有 Registry；Google Research 将其 Monolithic Repository 描述为大量开发者的共同源码事实来源，同时讨论配套系统、工作流与权衡；pnpm 文档说明 Workspace、`workspace:` 本地解析、发布转换、循环依赖和版本管理边界；Nx 文档区分 Project Graph 与 Task Graph，并用 Git 变化和依赖图计算 Affected Projects；Bazel 文档说明可复现 Build Action 如何通过声明输入与 Hash 安全复用 Remote Cache；Node.js、Nx 与 GitLab 文档分别支持 Package 入口、依赖约束和路径所有权。
- **机制推导：** 将仓库策略建模为跨项目变化协调成本的重新分配；分离 Repository、Project、Module、Task、Release、Deploy 与 Team 七种边界；用联合变化频率与强隔离要求判断仓库形态，并把 CI Pruning、Build Cache、Package 版本和运行兼容放回同一交付链。
- **工程建议：** 从真实跨仓变化记录选择 Pilot，保留原有业务与发布边界；先声明 Project、Public API、Graph、Owner 和可复现任务，再增加 Affected 与 Remote Cache；分别验证本地源码、发布 Package 和多版本部署，不把同仓便利当成解耦证据。
- **待项目验证：** 是否采用 Monorepo、哪些项目进入同仓、共享哪些 Package、Affected 与 Cache 能缩小多少 CI、权限和发布怎样分层，都必须由联合变化记录、隔离要求、仓库规模与平台维护成本决定。
- **下一步：** [[32：BFF 与客户端场景边界|BFF 与客户端场景边界]]，分析为什么客户端页面经常需要不同的数据形状，以及把聚合、裁剪与协议适配移到服务端后会新增哪些责任。
- **来源：** [[raw/live/08-前端架构设计指南③.txt|08 前端架构设计指南③原始文字稿]]、[[raw/links/2026-08-24-Google Monorepo Paper|Google Research：Why Google Stores Billions of Lines of Code in a Single Repository]]、[[raw/links/2026-08-24-pnpm Workspace|pnpm：Workspace]]、[[raw/links/2026-08-24-Nx Affected Project Graph|Nx：Affected 与 Project Graph]]、[[raw/links/2026-08-24-Bazel Remote Caching|Bazel：Remote Caching]]、[[raw/links/2026-08-21-Node.js Package Entry Points|Node.js Package Entry Points]]、[[raw/links/2026-08-21-Nx Enforce Module Boundaries|Nx Enforce Module Boundaries]]、[[raw/links/2026-08-21-GitLab Code Owners|GitLab Code Owners]]、[[raw/links/2026-08-21-Team Topologies Key Concepts|Team Topologies：Key Concepts]]
