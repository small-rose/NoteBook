---
layout: default
title:  RESTful Tutorial
parent: NET
grand_parent: Java
nav_order: 11
---

Here are about Java gRPC about .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }



## RESTful 完全教程：从入门到精通

> 技术主线：**Java 17 + Spring Boot 3.x**
> 目标读者：零基础 → 精通全阶段
> 每篇结构：概念讲解 → 正反例对比 → 完整可运行代码 → 小结 → 自测题

## 目录

| 篇章 | 阶段 | 内容概要 |
|------|------|---------|
| [01 · REST 基础与 HTTP 必备](./01-REST基础与HTTP必备.md) | 入门 | REST 由来与六大约束、HTTP 方法/状态码/Headers、REST vs RPC/SOAP、动手写第一个 Spring Boot API |
| [02 · 资源设计与接口规范](./02-资源设计与接口规范.md) | 进阶 | 资源命名与 URI 设计正反例、CRUD 映射表、嵌套资源、版本管理、分页/排序/过滤 |
| [03 · 请求响应与代码实现](./03-请求响应与代码实现.md) | 进阶 | DTO/VO/Entity 分层、Jakarta 参数校验、统一响应体、全局异常处理、内容协商、HATEOAS |
| [04 · 架构进阶与工程化](./04-架构进阶与工程化.md) | 高级 | JWT 鉴权、接口幂等性、缓存与 ETag、限流、OpenAPI 文档自动化、MockMvc/cURL 测试 |
| [05 · 精通：设计模式与治理](./05-精通-设计模式与治理.md) | 精通 | API 治理与平台化、API 网关、REST 常见坑与最佳实践、GraphQL/gRPC 协作、面试高频题 |

## 学习路线建议

```
入门（01）──→ 进阶（02 → 03）──→ 高级（04）──→ 精通（05）
  │                │                 │
  │                │                 └── 建议边学边做：给自己的项目加鉴权/文档/测试
  │                └── 进阶两篇建议配合阅读：02 管设计，03 管落地
  └── 有 HTTP 基础可速读，重点看 REST 六大约束
```

## 环境准备

- JDK 17+
- Maven 3.8+
- IDE：IntelliJ IDEA / VS Code
- 测试工具：cURL、Apifox/Postman

## 示例工程说明

各篇中的代码均基于同一个「图书管理 API」示例演进：

- 01 篇：最小可运行的 CRUD 接口（内存存储）
- 02 篇：规范化 URI 与资源设计
- 03 篇：加入 DTO、校验、统一响应、全局异常
- 04 篇：加入 JWT 鉴权、ETag、OpenAPI、测试
- 05 篇：讨论该工程演进为生产级服务的治理要点

每篇代码自成一体，单独复制即可运行；跨篇演进时会注明「相比上一篇的改动点」。
