---
type: 链接来源
status: 已记录
created: 2026-08-24
source_url: "https://www.rfc-editor.org/rfc/rfc9110.html"
---

# RFC 9110：Validators 与 Conditional Requests

- **机构：** IETF / RFC Editor
- **URL：** https://www.rfc-editor.org/rfc/rfc9110.html
- **访问日期：** 2026-08-24
- **资料类型：** Internet Standard

## 来源摘要

RFC 9110 定义 ETag、Last-Modified 等 Validator 以及条件请求。客户端用 `If-None-Match` 携带已有表示的 Entity Tag 发起 GET 时，若当前表示仍匹配，服务端可以返回 304 Not Modified；若不匹配则返回新的表示，从而减少没有变化时的响应体传输。

`If-Match` 则可以把写操作限定为“当前表示仍匹配指定 Entity Tag 时才执行”，用于避免客户端基于过期版本覆盖其他更新；前置条件失败通常返回 412 Precondition Failed。Validator 因而既可以服务缓存校验，也可以参与乐观并发控制，但两种用途需要按具体方法和比较规则配置。

## 使用边界

- 304 仍经历请求、网络往返和服务端校验，只是通常避免重新传输完整表示。
- ETag 标识的是某个资源表示版本，不等同于业务 Operation ID、用户身份或永久内容哈希。
- 条件写只能保护采用同一版本协议的入口，不能代替领域约束、权限或跨资源事务。
