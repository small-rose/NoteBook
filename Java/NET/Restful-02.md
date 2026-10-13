---
layout: default
title:  RESTful
parent: NET
grand_parent: Java
nav_order: 13
---


Here are about Java RESTful about .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }


# 02 · 资源设计与接口规范

> 阶段：进阶
> 本篇目标：学会正确的资源命名与 URI 设计，掌握 CRUD 映射、嵌套资源、版本管理、分页/排序/过滤的规范做法。

---

## 1. 设计思维的转变：从「动作」到「资源」

RPC 思维：接口是「动作」——`/getUser`、`/createOrder`、`/deleteBook`。
REST 思维：接口操作的是「资源」——动词由 HTTP 方法承担，URL 里只留名词。

**判断标准**：URL 里如果出现了动词（get/create/add/update/delete/save/load/submit……），基本可以断定设计错了。

### 正反例对照

| 反例（RPC 风格） | 正例（REST 风格） | 说明 |
|-----------------|------------------|------|
| `GET /getBookList` | `GET /books` | 查询集合 |
| `POST /createBook` | `POST /books` | 创建 |
| `POST /updateBook` | `PUT /books/{id}` | 更新 |
| `GET /deleteBook?id=1` | `DELETE /books/1` | 删除；**GET 做删除是大忌**（预取、爬虫会误删） |
| `GET /getUserOrder?userId=3` | `GET /users/3/orders` | 用路径表达从属关系 |

## 2. 资源命名规范

1. **用名词复数**：`/books`、`/users`、`/orders`（即使只有 1 条也用复数，保持一致性）。
2. **全小写，短横线连接**：`/user-profiles`（避免驼峰和下划线混用；推荐一种风格坚持到底）。
3. **层级用 `/` 表达**：`/departments/{deptId}/employees/{empId}`。
4. **不要在 URL 末尾加动词或文件扩展名**：`/books.json` 在纯 API 中没必要（除非做内容协商兼容）。
5. **路径层级不超过 3 层**：层级太深说明资源抽象有问题，考虑扁平化 + 查询参数。

## 3. CRUD 完整映射表

以「图书 `/books`、单本书 `/books/{id}`」为例：

| 操作 | 方法 + URL | 请求体 | 成功响应 | 失败 |
|------|-----------|--------|---------|------|
| 列表查询 | `GET /books` | - | `200` + 数组 | - |
| 详情查询 | `GET /books/{id}` | - | `200` + 对象 | `404` |
| 创建 | `POST /books` | JSON | `201` + 对象 + `Location` | `400/422` |
| 全量更新 | `PUT /books/{id}` | JSON（完整字段） | `200` + 对象 | `404`、`400` |
| 局部更新 | `PATCH /books/{id}` | JSON（部分字段） | `200` + 对象 | `404`、`400` |
| 删除 | `DELETE /books/{id}` | - | `204` | `404` |
| 批量删除 | `DELETE /books?ids=1,2,3` | - | `204` 或 `207` | - |

### PUT vs PATCH

- `PUT`：**全量替换**。客户端必须提交完整对象；缺失字段通常置空。
- `PATCH`：**局部更新**。只提交要改的字段，如 `{"price": 59.9}`。

实际工程中，很多团队简化为「只提供 PUT 做合并更新」——这在严格 REST 里是不准确的，但可接受；更好的做法是两者都提供，PATCH 用 JSON Merge Patch（`Content-Type: application/merge-patch+json`）。

### 批量操作

REST 对批量操作没有完美答案，常见折中：

```
DELETE /books?ids=1,2,3          # 用查询参数（简洁，推荐）
POST /books/batch-delete         # 子资源动作（见第 4 节）
```

## 4. 嵌套资源与子资源

### 4.1 表达从属关系

```
GET /users/3/orders            # 用户 3 的订单列表
GET /users/3/orders/1001       # 用户 3 的某个订单（1001 属于用户 3）
```

### 4.2 慎用深层嵌套

```
反例：GET /users/3/orders/1001/items/5/reviews/99
```

问题：
1. 耦合了从属链——如果 review 其实和 user 无关呢？
2. 破坏资源的独立性。

**替代方案**：引用扁平化，用查询参数表达过滤：

```
GET /reviews?orderId=1001
```

### 4.3 「动作」型接口怎么办

REST 不擅长表达动作（如「发布」「审核」）。两种主流做法：

**方案 A（推荐）：把动作转化为资源状态**

```
# 错误：POST /books/1/publish
# 正确：通过 PATCH 修改状态字段
PATCH /books/1
{"status": "PUBLISHED"}
```

**方案 B：动作为子资源（可接受的折中）**

当动作无法自然建模为状态时（如「结算购物车」「导出报表」），可以用：

```
POST /books/1/publish
POST /reports/export
```

要点：**必须用 POST**（有副作用，不能用 GET）；这类接口占整体比例应很低，超过 20% 说明你的建模偏向 RPC。

## 5. 版本管理

API 一定会变，必须提前规划版本。三种主流方案：

| 方案 | 示例 | 优点 | 缺点 |
|------|------|------|------|
| URL 路径 | `GET /v1/books` | 直观、易路由、易网关分流 | 不够「纯粹」（版本不是资源） |
| Accept 头 | `Accept: application/vnd.book.v1+json` | URL 干净、业界正统 | 调试麻烦、CDN/网关支持差 |
| 查询参数 | `GET /books?version=1` | 简单 | 缓存不友好，不推荐 |

**工程建议**：用 **URL 路径 `/v1/`**，务实优先。大版本不兼容时升 v2；小改动不升版本。

```
GET /v1/books
GET /v2/books          # v2 中 price 拆成 {amount, currency}
```

Spring Boot 中的落地方式：

```java
@RestController
@RequestMapping("/v1/books")
public class BookControllerV1 {
    // ...
}

@RestController
@RequestMapping("/v2/books")
public class BookControllerV2 {
    // v2 返回结构升级，v1 保持不变
}
```

## 6. 查询参数：分页、排序、过滤、搜索

`GET /books` 返回全部数据在生产上不可接受。统一采用以下查询参数约定（可全公司规范化）：

| 参数 | 含义 | 示例 |
|------|------|------|
| `page` | 页码（从 0 或 1 开始，团队统一） | `?page=0` |
| `size` | 每页条数（设上限，如 ≤100） | `?size=20` |
| `sort` | 排序字段，方向可选 | `?sort=price,desc` |
| 过滤字段 | 直接以字段名为参数名 | `?author=周志明&minPrice=50` |
| `search`/`q` | 模糊搜索 | `?q=Java` |

示例：

```
GET /books?page=0&size=10&sort=price,desc&author=周志明
```

### 分页响应格式

不要只返回数组，要返回**分页元数据**：

```json
{
  "content": [ /* 本页数据 */ ],
  "page": 0,
  "size": 10,
  "totalElements": 137,
  "totalPages": 14,
  "first": true,
  "last": false
}
```

> Spring Data 的 `Page<T>` 序列化后即为此结构；自研也可仿照。03 篇会给出完整实现。

### 部分参数不合法时

```
GET /books?page=-1        → 400 Bad Request（而不是悄悄纠正）
GET /books?size=10000     → 400（超出上限）
GET /books?sort=hack      → 400（字段不允许排序）
```

## 7. 状态码速查（进阶版）

结合本篇场景补充几个容易混淆的：

| 状态码 | 场景 |
|--------|------|
| `400` | 参数格式错误（JSON 解析失败、类型不匹配） |
| `401` | 未登录/Token 失效 |
| `403` | 已登录但无权限（如改别人的订单） |
| `404` | 资源不存在（id 对应的书没有） |
| `405` | 方法不允许（对 `/books` 用 PUT） |
| `409` | 资源冲突（创建重名书、并发修改冲突） |
| `415` | Content-Type 不支持（发了 XML 但服务端只收 JSON） |
| `422` | 语义/业务校验失败（price 为负数） |
| `429` | 触发限流 |

`400` vs `422`：语法错用 400，语义错用 422。团队若觉得难区分，统一用 400 也可，但要全团队一致。

## 8. 设计评审 Checklist

新接口评审时逐项过：

- [ ] URL 是名词、复数、小写短横线风格？
- [ ] 动词只出现在 HTTP 方法里？
- [ ] 方法的幂等性/安全性符合语义？
- [ ] 状态码如实（201/204/404/409…）？
- [ ] 列表接口有分页？分页参数有上限？
- [ ] 需要鉴权的接口说明了权限点？
- [ ] 有无版本规划？破坏性变更是否升版本？
- [ ] 响应字段是否有冗余/泄漏内部实现（如直接暴露 DB 主键、密码字段）？

## 9. 本篇小结

1. URL 只写名词，动词交给 HTTP 方法。
2. 嵌套最多两层，深层用查询参数扁平化。
3. 动作优先建模为资源状态变化。
4. 版本用 `/v1/` 路径，务实优先。
5. 列表必分页，分页响应带元数据。

## 10. 自测题

1. `POST /books/1/delete` 和 `DELETE /books/1` 哪个对？为什么？
2. 用查询参数实现「按作者过滤 + 按价格倒序 + 第 2 页每页 10 条」的完整 URL。
3. 什么情况下 `PATCH` 比 `PUT` 更合适？举一个真实业务例子。
4. 为什么 `GET /deleteBook?id=1` 特别危险？（提示：想想浏览器预取和爬虫）
