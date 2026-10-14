---
layout: default
title:  RESTful-01
parent: NET
grand_parent: Java
nav_order: 12
---


Here are about Java RESTful about .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }




# 01 · REST 基础与 HTTP 必备

> 阶段：入门
> 本篇目标：理解 REST 到底是什么、掌握 HTTP 方法与状态码的语义、跑通第一个 Spring Boot RESTful API。

---

## 1. REST 是什么

REST（**RE**presentational **S**tate **T**ransfer，表述性状态转移）由 Roy Fielding 在 2000 年的博士论文中提出。它不是框架、不是协议，而是一种**架构风格（Architectural Style）**——描述「怎样设计分布式系统才是好的」。

一句话理解 REST：

> **把网络上的一切看作「资源」，用统一的接口对资源做增删改查，资源的状态在客户端与服务端之间转移。**

- 「资源」：一篇文章、一个用户、一张订单……都是一种资源，用 URL 标识。
- 「表述（Representation）」：资源的具体表现形式，如 JSON、XML。
- 「状态转移」：客户端通过 HTTP 方法改变服务端资源的状态。

## 2. REST 的六大约束

Fielding 定义了 6 条约束，全部满足才是 RESTful 系统：

| # | 约束 | 含义 | 工程落地 |
|---|------|------|---------|
| 1 | **客户端-服务器（Client-Server）** | 前后端职责分离，各自独立演化 | 前端 SPA/APP + 后端纯 API |
| 2 | **无状态（Stateless）** | 服务端不保存会话，每次请求携带全部信息 | 鉴权用 Token/JWT，不用 HttpSession |
| 3 | **可缓存（Cacheable）** | 响应必须标明是否可缓存 | 返回 `Cache-Control`、`ETag` |
| 4 | **统一接口（Uniform Interface）** | 核心！资源用 URL 标识、HTTP 方法操作、自描述消息、HATEOAS | 见第 3 节 |
| 5 | **分层系统（Layered System）** | 客户端不需要知道服务端后面还有什么 | 网关、负载均衡、微服务分层 |
| 6 | **按需代码（Code-On-Demand，可选）** | 服务端可下发脚本 | JSONP（已过时）、JS Bundle |

其中**统一接口**是 REST 区别于 RPC 风格的最本质特征。

## 3. HTTP 必备知识

### 3.1 方法语义（动词）

| HTTP 方法 | 语义 | 幂等 | 安全 | 典型用法 |
|-----------|------|------|------|---------|
| GET | 获取资源 | ✅ | ✅ | `GET /books/1` |
| POST | 创建资源 / 执行子操作 | ❌ | ❌ | `POST /books` |
| PUT | 全量替换 | ✅ | ❌ | `PUT /books/1` |
| PATCH | 局部更新 | ❌（规范上） | ❌ | `PATCH /books/1` |
| DELETE | 删除资源 | ✅ | ❌ | `DELETE /books/1` |

**关键概念**：
- **安全**：不改变服务端状态（GET、HEAD）。
- **幂等**：执行一次和执行多次效果相同（GET、PUT、DELETE、HEAD 幂等；POST 不幂等）。幂等性在弱网重试、网关重放场景下至关重要。

### 3.2 状态码（结果的语言）

| 范围 | 含义 | 常用 |
|------|------|------|
| 2xx | 成功 | `200 OK`、`201 Created`、`204 No Content` |
| 3xx | 重定向 | `301` 永久移动、`304 Not Modified`（缓存） |
| 4xx | 客户端错误 | `400 Bad Request`、`401 Unauthorized`（未认证）、`403 Forbidden`（无权限）、`404 Not Found`、`405 Method Not Allowed`、`409 Conflict`（冲突）、`422 Unprocessable Entity`（语义错误）、`429 Too Many Requests` |
| 5xx | 服务端错误 | `500 Internal Server Error`、`502 Bad Gateway`、`503 Service Unavailable` |

> **新手最常犯的错**：查询失败返回 200 + `code: 404`。HTTP 状态码本身就是给客户端程序读的，应当正确使用（Spring 中返回 `ResponseEntity` 或抛异常交由 `@ExceptionHandler` 映射）。

### 3.3 常用 Headers

| Header | 方向 | 作用 |
|--------|------|------|
| `Content-Type` | 请求/响应 | 消息体类型，如 `application/json` |
| `Accept` | 请求 | 客户端期望的响应格式 |
| `Authorization` | 请求 | 携带凭证，如 `Bearer <token>` |
| `Location` | 响应 | 201 时告知新资源的 URL |
| `Cache-Control` | 响应 | 缓存策略，如 `max-age=60` |
| `ETag` | 响应 | 资源版本指纹，配合 `If-None-Match` 做条件请求 |

## 4. REST vs RPC vs SOAP

| 维度 | REST | RPC（gRPC/JSON-RPC） | SOAP |
|------|------|---------------------|------|
| 风格 | 面向资源 | 面向动作（调用方法） | 面向消息 |
| 载体 | HTTP + JSON 为主 | 二进制（gRPC/Protobuf）或 JSON | XML |
| 可读性 | 高，URL 即语义 | 中 | 低 |
| 性能 | 中 | 高（二进制 + 长连接） | 低 |
| 适用 | 对外开放 API、Web/移动端 | 内部微服务高性能调用 | 企业级、强规范、遗留系统 |

**结论**：对外 API 用 REST；内部高性能调用用 gRPC；不要用 SOAP（除非维护遗留系统）。三者不互斥，一个公司常常「对内 gRPC、对外 REST」。

## 5. 动手：第一个 Spring Boot RESTful API

下面实现一个「图书管理」最小示例：内存 `Map` 存储，支持增删改查。

### 5.1 创建工程

用 [start.spring.io](https://start.spring.io) 生成：Java 17、Spring Boot 3.2.x，依赖只勾选 **Spring Web**。

`pom.xml` 核心依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.5</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 5.2 领域对象

```java
package com.example.demo.book;

public class Book {

    private Long id;
    private String title;
    private String author;
    private Double price;

    public Book() {}

    public Book(Long id, String title, String author, Double price) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.price = price;
    }

    // getter / setter 略，实际工程请用 Lombok @Data
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
}
```

### 5.3 内存仓库

```java
package com.example.demo.book;

import org.springframework.stereotype.Repository;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class BookRepository {

    private final Map<Long, Book> store = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(0);

    public List<Book> findAll() {
        return new ArrayList<>(store.values());
    }

    public Optional<Book> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    public Book save(Book book) {
        long id = idGenerator.incrementAndGet();
        book.setId(id);
        store.put(id, book);
        return book;
    }

    public Book update(Long id, Book book) {
        book.setId(id);
        store.put(id, book);
        return book;
    }

    public void deleteById(Long id) {
        store.remove(id);
    }
}
```

### 5.4 Controller（核心）

```java
package com.example.demo.book;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;

import java.net.URI;
import java.util.List;

@RestController
@RequestMapping("/books")
public class BookController {

    private final BookRepository repository;

    public BookController(BookRepository repository) {
        this.repository = repository;
    }

    // 查询全部
    @GetMapping
    public List<Book> list() {
        return repository.findAll();
    }

    // 查询单个
    @GetMapping("/{id}")
    public ResponseEntity<Book> get(@PathVariable Long id) {
        return repository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // 创建
    @PostMapping
    public ResponseEntity<Book> create(@RequestBody Book book) {
        Book created = repository.save(book);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();
        return ResponseEntity.created(location).body(created);
    }

    // 全量更新
    @PutMapping("/{id}")
    public ResponseEntity<Book> replace(@PathVariable Long id, @RequestBody Book book) {
        if (repository.findById(id).isEmpty()) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(repository.update(id, book));
    }

    // 删除
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        if (repository.findById(id).isEmpty()) {
            return ResponseEntity.notFound().build();
        }
        repository.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 5.5 启动类

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 5.6 运行与验证

启动后用 cURL 验证：

```bash
# 创建
curl -X POST http://localhost:8080/books \
  -H "Content-Type: application/json" \
  -d '{"title":"深入理解Java虚拟机","author":"周志明","price":129.0}'

# 响应（注意 201 Created 和 Location 头）
# {"id":1,"title":"深入理解Java虚拟机","author":"周志明","price":129.0}

# 查询单个
curl http://localhost:8080/books/1

# 查询全部
curl http://localhost:8080/books

# 更新
curl -X PUT http://localhost:8080/books/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"图解HTTP","author":"上野宣","price":49.0}'

# 删除（成功返回 204 No Content）
curl -X DELETE -i http://localhost:8080/books/1
```

### 5.7 示例代码中已体现的 REST 规范

| 规范 | 代码体现 |
|------|---------|
| URL 标识资源 | `/books`、`/books/{id}` |
| 动词交给 HTTP 方法 | `@GetMapping` / `@PostMapping` / … |
| 201 + Location | `ResponseEntity.created(location)` |
| 404 处理 | `ResponseEntity.notFound()` |
| 204 无响应体 | `noContent().build()` |

> 这个示例还有不少「不规范」之处（如 PUT 不存在的资源直接更新、没有参数校验），02、03 篇会逐步完善。

## 6. 本篇小结

1. REST 是架构风格，核心是「资源 + 统一接口」。
2. 记住六大约束，工程上重点落地：**无状态、可缓存、统一接口**。
3. HTTP 方法要记语义和幂等性；状态码要如实反映结果。
4. REST ≠ 「用 GET/POST」，URL 里写动词（`/getBook?id=1`）不是 RESTful。

## 7. 自测题

1. 为什么说「无状态」约束能让服务端水平扩展更容易？
2. `POST /books/1/publish` 这种写法违反 REST 吗？（提示：见 02 篇「子资源/动作」一节）
3. GET 请求需要幂等吗？幂等对客户端重试有什么意义？
4. 把本篇示例的 `update` 改为：资源不存在时返回 404。
