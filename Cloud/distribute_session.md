---
layout: default
title: 分布式会话
parent: Cloud
nav_order: 2
---

# Distribute Global Session
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
   {:toc}


## 分布式会话解决方案

分布式环境中，Session 共享的核心问题是：用户的多次请求可能落在不同的服务器上，如何让每台服务器都能识别出同一个用户会话。下面梳理主流方案，按复杂度从低到高排列，并根据场景选则合适的方案。

### 1.粘性 Session（Sticky Session）

原理：负载均衡器根据用户 IP 或 Cookie 将同一用户的请求始终转发到同一台后端服务器。

实现方式：Nginx **ip_hash**、HAProxy **stick-table**、AWS ALB 的粘性会话。

优点：
- 零改造，应用无需感知分布式。
- 无额外存储开销。

缺点：
- 单点故障：服务器宕机后该用户 Session 丢失。
- 弹性伸缩困难：扩缩容会导致哈希重分布，部分用户 Session 失效。
- 负载不均：IP 哈希可能导致热点。

适用场景：小型应用、临时过渡方案、对 Session 持久性要求不高的场景。


### 2.Session 复制（Session Replication）

原理：每台服务器都保存全量 Session，通过组播或点对点协议同步变更（如 Tomcat 的 DeltaManager）。

优点：
- 应用无侵入。
-任意服务器都可处理请求。

缺点：
- 网络带宽浪费严重（所有 Session 全量复制）。
- 服务器数量增多时性能急剧下降（O(n²) 同步）。
- 存在短暂不一致窗口。

适用场景：小规模集群（≤5 台）、对实时性要求不高的遗留系统。

### 3.集中式 Session 存储（主流方案）

原理：将 Session 数据抽离到独立的中间件（Redis、Memcached、数据库），所有应用服务器统一读写。

#### 3.1 Redis + Spring Session（Java 生态首选）
实现： 
（1）引入 spring-session-data-redis 依赖。
（2）配置 Redis 连接。
（3）替换 HttpSession 为 Redis 后端存储。
（4）序列化方式可选 JDK、JSON（推荐 JSON 以跨语言）。

优点：

- 高性能（Redis 纯内存）。
- 天然支持过期、持久化、集群。
- 应用无感知，API 与本地 Session 完全一致。
- 支持 WebSocket、Header、Cookie 等多种 Session 映射方式。

缺点：

- 引入 Redis 运维成本。
- 网络延迟略高于本地内存（通常 <1ms）。
- 需考虑 Redis 宕机时的降级策略（如本地二级缓存 + 哨兵/集群）。

伪代码示例（Spring Boot 配置）：

```yaml
spring:
  session:
    store-type: redis
  redis:
    host: redis-cluster-ip
    port: 6379
```

#### 3.2 Memcached 或其他 KV 存储

原理相似，但 Memcached 不支持持久化和复杂数据结构，已逐渐被 Redis 取代。

#### 3.3 数据库存储（如 MySQL）

优点：不需要额外组件。

缺点：性能差（磁盘 IO），不适合高并发。

适用：极低并发、对一致性要求极高的管理后台。

### 4.无状态 Token（现代微服务架构）

原理：服务端不保存 Session，用户认证信息加密存储在客户端（Cookie、LocalStorage、Header），每次请求携带。最常见的是 JWT（JSON Web Token）。

工作流：
(1)用户登录 → 服务端签发 JWT（含用户 ID、角色、过期时间，用密钥签名）。

(2)客户端存储 JWT（通常放在 HTTP Header Authorization: Bearer <token>）。

(3)后续请求携带 JWT → 服务端验签后解析用户信息。

优点：

- 完全无状态，水平扩展零成本。
- 跨语言、跨平台。
- 减少服务端存储压力。
- 天然支持移动端、前后端分离。

缺点：

- Token 泄露后无法主动失效（除非维护黑名单或缩短有效期）。
- Payload 大小有限（Base64 编码后约几百字节），不适合存大量数据。
- 续期需要额外机制（Refresh Token）。

适用场景：RESTful API、微服务、前后端分离、移动 App。

增强方案：

- 短期 Access Token（15分钟）+ 长期 Refresh Token（7天）。
- 结合 Redis 黑名单实现即时吊销。


### 5.混合方案（企业常见）

典型组合：

- 前端应用：使用 JWT 做身份认证（无状态）。
- 后端业务：将部分临时数据（如验证码、购物车）存入 Redis 共享，用 Session ID 关联。
- 敏感操作：配合 OAuth2、CAS 单点登录。

优点：兼顾性能与灵活性。

### 6.方案对比

|方案	|一致性	|性能	|扩展性	|运维复杂度	|适用规模|
|-------|-------|-------|--------|----------|------|
|粘性 Session	|弱（单点）	|高	|差	|低	|小型|
|Session 复制	|最终一致	|中	|差	|中	|中小型|
|集中式 Redis	|强（取决于 Redis）	|高	|好	|中	|大型|
|无状态  JWT	|无状态	|高	|极好	|低	|任意规模 |


### 7。注意事项

- Session ID 生成：必须足够随机（如 UUID、SecureRandom），防止暴力猜测。
- HTTPS 传输：防止中间人窃取 Cookie/Token。
- Cookie 属性：设置 HttpOnly、SameSite=Lax、Secure。
- JWT 签名：使用 RS256/ES256 非对称算法，私钥妥善保管。
- Token 吊销：关键操作（密码修改、账号封禁）需使旧 Token 失效，可通过 Redis 黑名单或短有效期实现。
