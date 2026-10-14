---
layout: default
title:  RESTful-04
parent: NET
grand_parent: Java
nav_order: 15
---


Here are about Java RESTful about .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }

# 04 · 架构进阶与工程化

> 阶段：高级
> 本篇目标：给 API 加上生产级能力——JWT 鉴权、接口幂等性、缓存与 ETag、限流、OpenAPI 自动文档、MockMvc/cURL 接口测试。

---

## 1. JWT 无状态鉴权

REST 要求无状态，因此不用 Session，用 Token。JWT（JSON Web Token）是主流方案。

### 1.1 依赖

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
```

### 1.2 JWT 工具类

```java
package com.example.demo.auth;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;

@Component
public class JwtUtil {

    private final SecretKey key;
    private final long expireMillis;

    public JwtUtil(@Value("${jwt.secret}") String secret,
                   @Value("${jwt.expire-hours:24}") long hours) {
        this.key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
        this.expireMillis = hours * 3600_000L;
    }

    public String generateToken(String username) {
        Date now = new Date();
        return Jwts.builder()
                .subject(username)
                .issuedAt(now)
                .expiration(new Date(now.getTime() + expireMillis))
                .signWith(key)
                .compact();
    }

    public String parseUsername(String token) {
        return Jwts.parser()
                .verifyWith(key)
                .build()
                .parseSignedClaims(token)
                .getPayload()
                .getSubject();
    }
}
```

### 1.3 认证过滤器

```java
package com.example.demo.auth;

import jakarta.servlet.*;
import jakarta.servlet.http.*;
;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;

    public JwtAuthFilter(JwtUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            try {
                String username = jwtUtil.parseUsername(token);
                request.setAttribute("currentUser", username);
            } catch (Exception e) {
                response.setStatus(401);
                response.setContentType("application/json");
                response.getWriter().write("{\"message\":\"无效或过期的 Token\"}");
                return;
            }
        }
        chain.doFilter(request, response);
    }
}
```

### 1.4 登录接口 + 保护写操作

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    private final JwtUtil jwtUtil;

    public AuthController(JwtUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    @PostMapping("/login")
    public Map<String, String> login(@RequestBody LoginRequest req) {
        // 生产环境：查库校验密码哈希（如 BCrypt）
        if ("admin".equals(req.getUsername()) && "123456".equals(req.getPassword())) {
            return Map.of("token", jwtUtil.generateToken(req.getUsername()));
        }
        throw new org.springframework.web.server.ResponseStatusException(401, "用户名或密码错误");
    }

    public record LoginRequest(String username, String password) {}
}
```

在写操作上开启鉴权（示意）：

```java
@PostMapping
public ResponseEntity<BookResponse> create(@Valid @RequestBody BookCreateRequest req,
                                           HttpServletRequest http) {
    String user = (String) http.getAttribute("currentUser");
    if (user == null) {
        throw new org.springframework.web.server.ResponseStatusException(401, "请先登录");
    }
    // ...
}
```

> 实际项目用 Spring Security 配置 `SecurityFilterChain`，按 URL 粒度声明 `/books` GET 公开、写操作需认证。注意区分：**401 = 没认证；403 = 认证了但权限不够**。

### 1.5 JWT 使用要点

| 要点 | 说明 |
|------|------|
| 密钥 | 用足够长的随机串（≥32 字节），放配置中心，**严禁硬编码进代码仓库** |
| 过期时间 | Access Token 短（30min~2h），配 Refresh Token 机制 |
| 敏感信息 | 不要把密码等敏感信息放 claims（JWT 可被解码看 payload） |
| 撤销 | JWT 无状态难以吊销；需要踢人时用黑名单（Redis）或短 Token + 刷新 |

## 2. 接口幂等性

弱网下客户端会重试，非幂等接口（POST 创建订单、支付）重试会造成重复下单。

### 2.1 方案：Idempotency-Key 请求头

```
POST /orders
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

服务端逻辑：
1. 收到请求，先在 Redis `SETNX idem:{key}`。
2. 若 key 已存在 → 说明是重试，直接返回**首次的结果**（从缓存取回响应）。
3. 若不存在 → 正常处理，处理完把响应存入 `idem:{key}` 并设 TTL（如 24h）。

示意实现：

```java
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrder(
        @RequestHeader("Idempotency-Key") String idemKey,
        @RequestBody OrderCreateRequest req) {

    String cacheKey = "idem:" + idemKey;
    Boolean first = redisTemplate.opsForValue()
            .setIfAbsent(cacheKey, "PROCESSING", Duration.ofHours(24));

    if (Boolean.FALSE.equals(first)) {
        // 重试请求：返回之前缓存的结果
        Object cached = redisTemplate.opsForValue().get(cacheKey + ":resp");
        if (cached != null) {
            return ResponseEntity.ok((OrderResponse) cached);
        }
        return ResponseEntity.status(409).body(/* 冲突说明 */);
    }

    OrderResponse result = orderService.create(req);
    redisTemplate.opsForValue().set(cacheKey + ":resp", result, Duration.ofHours(24));
    return ResponseEntity.status(201).body(result);
}
```

## 3. 缓存与 ETag（条件请求）

### 3.1 服务端开启 ETag

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {}
}

// 简单方式：Spring MVC 自带 ShallowEtagHeaderFilter
@Bean
public Filter shallowEtagFilter() {
    return new ShallowEtagHeaderFilter();
}
```

开启后，Spring 会计算响应体哈希放入 `ETag` 头。

### 3.2 客户端条件请求

```
GET /books/1

响应：
ETag: "abc123"

──────────────────────────
GET /books/1
If-None-Match: "abc123"

响应：304 Not Modified（无响应体，节省带宽）
```

### 3.3 Cache-Control

```java
@GetMapping("/{id}")
@CacheControl(maxAge = 60)  // 或在配置中全局设置
public BookResponse get(@PathVariable Long id) { /* ... */ }
```

| 头 | 作用 |
|----|------|
| `Cache-Control: max-age=60` | 60 秒内可直接用本地缓存 |
| `ETag` / `If-None-Match` | 精确校验资源是否变化 |
| `Last-Modified` / `If-Modified-Since` | 按时间校验 |

## 4. 限流

防止被刷爆。常见算法：令牌桶、滑动窗口。

### 4.1 用 Bucket4j + Redis 做简单限流

```java
@Aspect
@Component
public class RateLimitAspect {

    private final StringRedisTemplate redis;

    public RateLimitAspect(StringRedisTemplate redis) {
        this.redis = redis;
    }

    @Around("@annotation(rateLimit)")
    public Object limit(ProceedingJoinPoint pjp, RateLimit rateLimit) throws Throwable {
        String key = "rl:" + pjp.getSignature().toShortString();
        Long count = redis.opsForValue().increment(key);
        if (count != null && count == 1L) {
            redis.expire(key, rateLimit.seconds(), TimeUnit.SECONDS);
        }
        if (count != null && count > rateLimit.limit()) {
            throw new ResponseStatusException(429, "请求过于频繁，请稍后再试");
        }
        return pjp.proceed();
    }
}
```

自定义注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    int limit() default 60;
    long seconds() default 60;
}
```

使用：

```java
@RateLimit(limit = 10, seconds = 60)
@PostMapping("/auth/login")
public Map<String, String> login(...) { /* ... */ }
```

限流粒度建议：登录接口按 IP + 用户名；普通读接口按用户/IP；写接口从严。

## 5. OpenAPI 自动文档（springdoc）

手写文档必然过期，用注解自动生成并托管 Swagger UI。

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI bookApi() {
        return new OpenAPI()
                .info(new Info()
                        .title("图书管理 API")
                        .version("1.0")
                        .description("RESTful 教程示例工程"));
    }
}
```

Controller 注解示例：

```java
@Tag(name = "图书管理", description = "图书的增删改查")
@RestController
@RequestMapping("/books")
public class BookController {

    @Operation(summary = "分页查询图书", description = "支持按价格排序")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "成功"),
            @ApiResponse(responseCode = "400", description = "参数错误")
    })
    @GetMapping
    public Page<BookResponse> list(/* ... */) { /* ... */ }

    @Operation(summary = "创建图书")
    @io.swagger.v3.oas.annotations.parameters.RequestBody(
            required = true,
            description = "图书信息，price 必须为正数")
    @PostMapping
    public ResponseEntity<BookResponse> create(@Valid @RequestBody BookCreateRequest req) { /* ... */ }
}
```

启动后访问：

- Swagger UI：`http://localhost:8080/swagger-ui.html`
- OpenAPI JSON：`http://localhost:8080/v3/api-docs`

## 6. 接口测试

### 6.1 cURL 冒烟测试

```bash
# 登录获取 token
TOKEN=$(curl -s -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"123456"}' | jq -r .token)

# 带 token 创建
curl -X POST http://localhost:8080/books \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"RESTful最佳实践","author":"某作者","price":66}'
```

### 6.2 MockMvc 自动化测试

```java
package com.example.demo.book;

import com.example.demo.book.dto.BookCreateRequest;
import com.example.demo.common.GlobalExceptionHandler;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.context.annotation.Import;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.util.Optional;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BookController.class)
@Import(GlobalExceptionHandler.class)
class BookControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BookService bookService;

    @Test
    void 创建图书_应返回201和Location头() throws Exception {
        Book created = new Book(1L, "图解HTTP", "上野宣", 49.0);
        when(bookService.create(any(BookCreateRequest.class))).thenReturn(created);

        mockMvc.perform(post("/books")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"title\":\"图解HTTP\",\"author\":\"上野宣\",\"price\":49}"))
                .andExpect(status().isCreated())
                .andExpect(header().string("Location", "/books/1"))
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.title").value("图解HTTP"));
    }

    @Test
    void 查询不存在的书_应返回404() throws Exception {
        when(bookService.findById(999L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/books/999"))
                .andExpect(status().isNotFound());
    }

    @Test
    void 创建图书_price为负_应返回422() throws Exception {
        mockMvc.perform(post("/books")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"title\":\"x\",\"author\":\"y\",\"price\":-1}"))
                .andExpect(status().isUnprocessableEntity())
                .andExpect(jsonPath("$.errors[0].field").value("price"));
    }
}
```

测试覆盖建议：
- 每个接口至少：正常路径 + 边界 + 404/400/401 异常路径。
- 状态码、关键响应头（Location/ETag）、关键字段都要断言。

## 7. 受限网络环境：只允许 GET/POST 怎么办

**先澄清一个常见误解**：Nginx 本身并不禁止 PUT/DELETE/PATCH，它默认支持全部 HTTP 方法。只有你主动写了 `limit_except GET POST { deny all; }` 才会拦。

真正拦你的是中间层：**企业内网代理、防火墙、部分 WAF/网关、老式负载均衡器**。银行、政企、运营商内网里只放行 GET/POST 的情况确实很常见。这种环境下 PUT/DELETE/PATCH 根本到不了你的服务。

### 7.1 方案一：Method Override 头（业界通用）

用 POST 打头，带一个标准约定头声明真实方法：

```
POST /books/1
X-HTTP-Method-Override: PUT
Content-Type: application/json

{"title":"图解HTTP","price":49}
```

服务端读头转发到对应处理逻辑。这是 HTTP 社区广泛约定俗成的做法（虽然 RFC 未正式定义，但 de-facto 标准）。

### 7.2 方案二：`_method` 表单参数（Spring 内置支持）

```
POST /books/1
Content-Type: application/x-www-form-urlencoded

_method=PUT&title=图解HTTP&price=49
```

Spring Boot 自带 `HiddenHttpMethodFilter`，在配置里打开即可：

```properties
# application.properties
spring.mvc.hiddenmethod.filter.enabled=true
```

它会把带 `_method=PUT` 的 POST 请求包装成 `HttpServletRequest` 后的 PUT，Controller 里的 `@PutMapping` 照常命中。

### 7.3 方案三：全站 POST 化（务实但妥协）

有些团队干脆约定：所有操作都走 POST，靠 URL 区分：

```
POST /books/query     # 查询
POST /books/create    # 创建
POST /books/update    # 更新
POST /books/delete    # 删除
```

这是彻底的 RPC 风格，**不推荐**——丢掉了 HTTP 语义、缓存能力（GET 可缓存）和幂等性提示。但在极端受限且无法加 override 头的环境里，是可接受的兜底。

### 7.4 如何选

| 环境 | 建议 |
|------|------|
| 公网 / 现代架构 | 直接用标准方法，别被带偏 |
| 企业内网有代理但可加头 | 方案一 `X-HTTP-Method-Override` |
| Spring 项目且表单提交为主 | 方案二 `_method`（零代码） |
| 代理连自定义头都剥 | 方案三 POST 化，但在文档里注明 |

> 无论选哪种，**内部服务之间（网关后面）仍用标准方法**，override 只发生在边界网关/接入层做转换，避免语义在系统内部扩散。

## 8. 本篇小结

1. JWT 让鉴权无状态化，注意密钥管理与 Token 撤销问题。
2. 写接口（创建/支付）必须考虑幂等，用 Idempotency-Key。
3. ETag + Cache-Control 减少重复传输；限流保护服务。
4. OpenAPI 注解自动生成文档，杜绝文档过期。
5. MockMvc 让接口规范进 CI，防止回归。
6. 受限网络下用 Method Override 兜底，别让基础设施限制污染内部设计。

## 9. 自测题

1. 401 和 403 分别在什么场景返回？举两个业务例子。
2. 为什么 JWT 不适合直接放「需要即时撤销」的权限（如封号）？给出至少一种解决方案。
3. Idempotency-Key 机制中，TTL 设 24h 和 1h 各有什么权衡？
4. 用 MockMvc 写一个「未带 token 访问创建接口应返回 401」的测试。
