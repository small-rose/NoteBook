---
layout: default
title:  RESTful
parent: NET
grand_parent: Java
nav_order: 14
---


Here are about Java RESTful about .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }

# 03 · 请求响应与代码实现

> 阶段：进阶
> 本篇目标：把 01 篇的「能跑」升级为「工程级」——DTO 分层、参数校验、统一响应体、全局异常处理、分页实现、内容协商与 HATEOAS。

本篇在 01 篇示例基础上演进，最终形成一个结构清晰的 Controller 落地范式。

---

## 1. 分层架构：Entity ≠ DTO ≠ VO

直接把数据库实体返回给前端是常见事故源：

1. 泄漏内部字段（密码哈希、逻辑删除标记、自增主键规律）。
2. 数据库改字段 = API 破坏性变更。
3. 循环引用导致 JSON 序列化死循环（如 User ↔ Order）。

**标准分层**：

| 层 | 名称 | 作用 |
|----|------|------|
| Entity | 实体 | 对应数据库表，仅供持久层 |
| DTO | Data Transfer Object | 接收请求体（入参），字段按需暴露 |
| VO | View Object | 返回给前端的视图模型，字段按需裁剪 |
| BO/Domain | 可选 | 业务逻辑层对象，中大型项目才需要 |

> 简单项目可让 DTO 兼任入参和出参；中大型项目务必分开。

## 2. 参数校验（Jakarta Validation）

Spring Boot 3 内置 Jakarta Bean Validation。

### 2.1 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 2.2 入参 DTO 带注解

```java
package com.example.demo.book.dto;

import jakarta.validation.constraints.*;

public class BookCreateRequest {

    @NotBlank(message = "书名不能为空")
    @Size(max = 100, message = "书名最长 100 字符")
    private String title;

    @NotBlank(message = "作者不能为空")
    private String author;

    @NotNull(message = "价格不能为空")
    @Positive(message = "价格必须为正数")
    private Double price;

    // getter / setter 略
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
}
```

### 2.3 Controller 上开启校验

```java
@PostMapping
public ResponseEntity<BookResponse> create(@Valid @RequestBody BookCreateRequest request) {
    // ...
}
```

校验失败时 Spring 默认抛 `MethodArgumentNotValidException`——由全局异常处理器统一接管（见第 4 节）。

常用注解速查：

| 注解 | 含义 |
|------|------|
| `@NotNull` / `@NotBlank` / `@NotEmpty` | 非空（对象/字符串空白/集合） |
| `@Size(max/min)` | 长度或元素个数 |
| `@Min/@Max` | 数值范围 |
| `@Pattern(regexp=...)` | 正则（如手机号、ISBN） |
| `@Email` | 邮箱格式 |
| `@Positive` | 正数 |

## 3. 统一响应体

两种流派：

**流派 A：HTTP 状态码如实 + 纯资源 JSON（推荐，国际主流）**

```json
// 成功 200
{ "id": 1, "title": "图解HTTP", "price": 49.0 }

// 失败 422
{ "timestamp": "2026-10-13T10:00:00Z", "status": 422,
  "errors": [{ "field": "price", "message": "价格必须为正数" }] }
```

**流派 B：统一包装 code/data/msg（国内常见）**

```json
{ "code": 0, "message": "ok", "data": { "id": 1, "title": "图解HTTP" } }
```

- 流派 A 更符合 HTTP 语义，前端可用拦截器统一处理 401/403/500。
- 流派 B 对「全站前后端同构的错误码体系」更友好。

**务实建议**：对外开放 API 用 A；内部中台若已有 code 体系可用 B，但**二选一，全公司统一**。本教程采用流派 A。

### 3.1 VO 与响应

```java
package com.example.demo.book.vo;

public class BookResponse {

    private Long id;
    private String title;
    private String author;
    private Double price;

    public BookResponse(Long id, String title, String author, Double price) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.price = price;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public Double getPrice() { return price; }
}
```

## 4. 全局异常处理

用 `@RestControllerAdvice` 把异常统一映射为 HTTP 状态码 + JSON 错误体。

### 4.1 统一错误体

```java
package com.example.demo.common;

import java.time.Instant;
import java.util.List;

public class ApiError {

    private Instant timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private List<FieldErrorDetail> errors;

    public ApiError(int status, String error, String message, String path) {
        this.timestamp = Instant.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
        this.errors = List.of();
    }

    // getter / setter 略
    public Instant getTimestamp() { return timestamp; }
    public int getStatus() { return status; }
    public String getError() { return error; }
    public String getMessage() { return message; }
    public String getPath() { return path; }
    public List<FieldErrorDetail> getErrors() { return errors; }
    public void setErrors(List<FieldErrorDetail> errors) { this.errors = errors; }

    public record FieldErrorDetail(String field, String message) {}
}
```

### 4.2 全局处理器

```java
package com.example.demo.common;

import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // 业务：资源不存在
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(ResourceNotFoundException ex,
                                                   HttpServletRequest request) {
        ApiError body = new ApiError(404, "Not Found", ex.getMessage(),
                request.getRequestURI());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
    }

    // 校验失败：422 + 字段明细
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiError> handleValidation(MethodArgumentNotValidException ex,
                                                     HttpServletRequest request) {
        List<ApiError.FieldErrorDetail> details = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(fe -> new ApiError.FieldErrorDetail(fe.getField(), fe.getDefaultMessage()))
                .toList();
        ApiError body = new ApiError(422, "Unprocessable Entity",
                "参数校验失败", request.getRequestURI());
        body.setErrors(details);
        return ResponseEntity.status(HttpStatus.UNPROCESSABLE_ENTITY).body(body);
    }

    // 参数类型错误（如 id 传了 "abc"）
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ApiError> handleTypeMismatch(MethodArgumentTypeMismatchException ex,
                                                       HttpServletRequest request) {
        ApiError body = new ApiError(400, "Bad Request",
                "参数类型错误: " + ex.getName(), request.getRequestURI());
        return ResponseEntity.badRequest().body(body);
    }

    // 兜底：500
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleOther(Exception ex, HttpServletRequest request) {
        ApiError body = new ApiError(500, "Internal Server Error",
                "服务器内部错误", request.getRequestURI());
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(body);
    }
}
```

> 生产上兜底 handler 应记录日志但**不把堆栈返回给客户端**。

### 4.3 自定义业务异常

```java
package com.example.demo.common;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

## 5. 完整 Controller（整合版）

```java
package com.example.demo.book;

import com.example.demo.book.dto.BookCreateRequest;
import com.example.demo.book.vo.BookResponse;
import com.example.demo.common.ResourceNotFoundException;
import jakarta.validation.Valid;
import org.springframework.data.domain.*;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;

import java.net.URI;

@RestController
@RequestMapping("/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    // 分页查询
    @GetMapping
    public Page<BookResponse> list(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id,asc") String[] sort) {

        Sort sortOrder = Sort.by(
                Sort.Order.by(sort[0]),
                sort.length > 1 && "desc".equalsIgnoreCase(sort[1])
                        ? Sort.Direction.DESC : Sort.Direction.ASC);
        Pageable pageable = PageRequest.of(page, size, sortOrder);

        return bookService.findPage(pageable).map(BookResponse::from);
    }

    @GetMapping("/{id}")
    public BookResponse get(@PathVariable Long id) {
        return bookService.findById(id)
                .map(BookResponse::from)
                .orElseThrow(() -> new ResourceNotFoundException("图书不存在: " + id));
    }

    @PostMapping
    public ResponseEntity<BookResponse> create(@Valid @RequestBody BookCreateRequest req) {
        Book created = bookService.create(req);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}").buildAndExpand(created.getId()).toUri();
        return ResponseEntity.created(location).body(BookResponse.from(created));
    }

    @PatchMapping("/{id}")
    public BookResponse patch(@PathVariable Long id,
                              @RequestBody BookCreateRequest req) {
        return BookResponse.from(bookService.patch(id, req));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        bookService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### VO 映射方法

```java
package com.example.demo.book.vo;

public class BookResponse {

    // ...字段同前...

    public static BookResponse from(com.example.demo.book.Book entity) {
        return new BookResponse(entity.getId(), entity.getTitle(),
                entity.getAuthor(), entity.getPrice());
    }
}
```

### Service 层（PATCH 合并逻辑）

```java
package com.example.demo.book;

import com.example.demo.book.dto.BookCreateRequest;
import com.example.demo.common.ResourceNotFoundException;
import org.springframework.data.domain.*;
import org.springframework.stereotype.Service;

@Service
public class BookService {

    private final BookRepository repository;

    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public Page<Book> findPage(Pageable pageable) {
        return repository.findAll(pageable);
    }

    public java.util.Optional<Book> findById(Long id) {
        return repository.findById(id);
    }

    public Book create(BookCreateRequest req) {
        Book book = new Book(null, req.getTitle(), req.getAuthor(), req.getPrice());
        return repository.save(book);
    }

    public Book patch(Long id, BookCreateRequest req) {
        Book book = repository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("图书不存在: " + id));
        if (req.getTitle() != null) book.setTitle(req.getTitle());
        if (req.getAuthor() != null) book.setAuthor(req.getAuthor());
        if (req.getPrice() != null) book.setPrice(req.getPrice());
        return repository.save(book);
    }

    public void delete(Long id) {
        if (repository.findById(id).isEmpty()) {
            throw new ResourceNotFoundException("图书不存在: " + id);
        }
        repository.deleteById(id);
    }
}
```

> 上例中的 Repository 若使用 Spring Data JPA，只需继承 `JpaRepository<Book, Long>` 即可自动获得 `findAll(Pageable)` 分页能力。为保持示例简单，沿用内存 Map 版时可自行实现分页切分。

## 6. 内容协商

Spring 默认支持按 `Accept` 头返回 JSON/XML：

```
GET /books/1
Accept: application/json     → 返回 JSON
Accept: application/xml      → 返回 XML（需引入 jackson-dataformat-xml）
```

```xml
<!-- 需要 XML 支持时加入 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

多数现代前后端项目只保留 JSON 即可（`spring.mvc.produce-media-type` 或直接不引 XML 依赖）。

## 7. HATEOAS 入门

HATEOAS（Hypermedia as the Engine of Application State）是 REST 约束中最难落地的一条：响应中包含**超链接**，客户端通过链接发现可用操作。

```json
{
  "id": 1,
  "title": "图解HTTP",
  "links": [
    { "rel": "self",  "href": "http://localhost:8080/books/1" },
    { "rel": "orders","href": "http://localhost:8080/books/1/orders" }
  ]
}
```

Spring HATEOAS 示例：

```java
@GetMapping("/{id}")
public EntityModel<BookResponse> get(@PathVariable Long id) {
    BookResponse book = /* ... */;
    return EntityModel.of(book,
            linkTo(methodOn(BookController.class).get(id)).withSelfRel(),
            linkTo(methodOn(BookController.class).list(0, 10, null)).withRel("books"));
}
```

**工程现实**：HATEOAS 理论优美，但移动端/前端实践中很少真正消费超链接。除非你在做「对外开放的 HATEAPI 平台」，否则了解即可，不必强上。

## 8. 本篇小结

1. Entity 永远不出 Controller，入参 DTO、出参 VO 分离。
2. 校验用 `@Valid` + 注解，失败交给全局 handler 返回 422。
3. 全局异常处理是「统一错误语义」的唯一正确落点。
4. 分页必带元数据；排序字段要白名单化。
5. HATEOAS 了解即可，务实优先。

## 9. 自测题

1. 为什么不能直接把 JPA Entity 返回给前端？至少说出 3 个风险。
2. `@NotBlank` 和 `@NotNull` 的区别是什么？分别用在什么字段上？
3. 写一个 `@ExceptionHandler` 处理 `HttpRequestMethodNotSupportedException`，应返回什么状态码？
4. PATCH 接口里 `req.getPrice() == null` 时应如何处理？（保持原值 vs 置空，两种语义如何设计？）
