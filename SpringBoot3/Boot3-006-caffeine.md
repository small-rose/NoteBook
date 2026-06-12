---
layout: default
title: SB3-caffeine
parent: SpringBoot3
has_children: false
nav_order: 1006
last_modified_date: 2026-06-10
---

Here are the logging system changes and practices on SpringBoot3 .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }

# Caffeine 高级指南

> Spring Boot 3 + Java 17 环境下的 Caffeine 缓存全面教程

---

## 目录

1. [Caffeine 简介](#1-caffeine-简介)
2. [快速入门](#2-快速入门)
3. [缓存配置详解](#3-缓存配置详解)
4. [核心 API 使用](#4-核心-api-使用)
5. [Spring Cache 集成](#5-spring-cache-集成)
6. [多级缓存实战](#6-多级缓存实战)
7. [高级应用场景](#7-高级应用场景)
8. [CXF WebService 客户端缓存](#8-cxf-webservice-客户端缓存)
9. [监控与运维](#9-监控与运维)
10. [最佳实践](#10-最佳实践)
11. [完整实战案例](#11-完整实战案例)
12. [附录：常用配置速查](#附录常用配置速查)

---

## 1. Caffeine 简介

### 1.1 什么是 Caffeine

Caffeine 是一个基于 Java 8 的高性能本地缓存库，由 Google Guava Cache 的作者 **Ben Manes** 开发。它提供了接近 **100% 的命中率**，并且在读写性能上远超 Guava Cache。

### 1.2 为什么选择 Caffeine

| 特性 | Caffeine | Guava Cache | EhCache |
|---|---|---|---|
| **淘汰算法** | W-TinyLFU | LRU | LRU/FIFO |
| **读写性能** | 极高 | 高 | 中等 |
| **并发模型** | 无锁（CAS） | 分段锁 | 锁 |
| **过期策略** | 写入/访问/自定义 | 写入/访问 | 写入/访问 |
| **异步支持** | AsyncLoadingCache | 不支持 | 不支持 |
| **Spring 集成** | 原生支持 | 需手动适配 | 需要 EhCache 适配 |
| **统计功能** | 内置 | 需手动开启 | 需配置 |

### 1.3 W-TinyLFU 淘汰算法

Caffeine 使用 **W-TinyLFU**（Window Tiny Least Frequently Used）算法，这是 LRU 和 LFU 的混合方案：

- **Window LRU**：小窗口，用于短期缓存
- **TinyLFU**：频率统计，用于长期缓存
- **Admission**：新条目先进入 Window，命中一定次数后才进入 Main

```
┌─────────────────────────────────────────────────┐
│                    Caffeine                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │ Window   │───>│ Admission│───>│   Main   │  │
│  │   LRU    │    │ TinyLFU  │    │ Segmented│  │
│  │ (小容量) │    │ (频率统计)│    │   LRU    │  │
│  └──────────┘    └──────────┘    └──────────┘  │
└─────────────────────────────────────────────────┘
```

---

## 2. 快速入门

### 2.1 添加依赖

**build.gradle**
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.14'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.small.rose'
version = '1.0.0'
description = 'Demo project for Spring Boot'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

// 使用项目属性动态设置名称
jar {
    archiveFileName = "${project.name}-${project.version}.jar"
    // 详见 README
    archiveClassifier = ''
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    // 核心依赖
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'com.github.ben-manes.caffeine:caffeine:3.2.0'
}
```

**Maven**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 2.2 Spring Boot 配置

**application.yml**
```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=600s,recordStats
```

`spring.cache.caffeine.spec` 语法说明（对应 `CaffeineSpec`）：

| 参数 | 示例 | 说明 |
|---|---|---|
| `maximumSize` | `maximumSize=1000` | 最大条目数 |
| `maximumWeight` | `maximumWeight=10000` | 最大权重（需同时配置 `weigher`） |
| `expireAfterWrite` | `expireAfterWrite=600s` | 写入后过期，支持 `s`(秒) `m`(分) `h`(时) `d`(天) |
| `expireAfterAccess` | `expireAfterAccess=300s` | 访问后过期 |
| `initialCapacity` | `initialCapacity=100` | 初始容量 |
| `recordStats` | `recordStats` | 开启统计（仅此参数无值） |

> 若需为不同缓存设置不同规格，需自定义 `CaffeineCacheManager`（见 5.8）。

### 2.3 启用缓存

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class DemoBoot3Application {
    public static void main(String[] args) {
        SpringApplication.run(DemoBoot3Application.class, args);
    }
}
```

### 2.4 基本使用

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Cacheable(value = "user", key = "#userId")
    public UserInfo getUserById(String userId) {
        return userRepository.findById(userId).orElse(null);
    }
}
```

---

## 3. 缓存配置详解

### 3.1 容量策略

#### maximumSize - 最大条目数

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;

public class CapacityDemo {

    Cache<String, String> maxSize = Caffeine.newBuilder()
            .maximumSize(1000)
            .build();

    Cache<String, String> maxWeight = Caffeine.newBuilder()
            .maximumWeight(10000)
            .weigher((key, value) -> value.length())
            .build();
}
```

### 3.2 过期策略

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.Expiry;
import java.util.concurrent.TimeUnit;

public class ExpiryDemo {

    Cache<String, String> afterWrite = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();

    Cache<String, String> afterAccess = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterAccess(5, TimeUnit.MINUTES)
            .build();

    Cache<String, String> afterCreate = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterCreate(15, TimeUnit.MINUTES)
            .build();

    Cache<String, String> customExpiry = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfter(new Expiry<String, String>() {
                @Override
                public long expireAfterCreate(String key, String value, long currentTime) {
                    return key.startsWith("vip")
                            ? TimeUnit.HOURS.toNanos(1)
                            : TimeUnit.MINUTES.toNanos(10);
                }

                @Override
                public long expireAfterUpdate(String key, String value,
                        long currentTime, long currentDuration) {
                    return currentDuration;
                }

                @Override
                public long expireAfterRead(String key, String value,
                        long currentTime, long currentDuration) {
                    return currentDuration;
                }
            })
            .build();
}
```

### 3.3 刷新策略

#### refreshAfterWrite - 写入后刷新

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.LoadingCache;
import java.util.concurrent.TimeUnit;

public class RefreshDemo {

    LoadingCache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .refreshAfterWrite(5, TimeUnit.MINUTES)
            .build(key -> loadFromDatabase(key));

    private String loadFromDatabase(String key) {
        return "value-" + key;
    }
}
```

**刷新 vs 过期：**
- **过期**：条目被移除，下次访问需要重新加载
- **刷新**：异步重新加载，旧值仍然可用（不阻塞读取）

### 3.4 移除监听

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;

public class RemovalListenerDemo {

    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .removalListener((key, value, cause) -> {
                switch (cause) {
                    case EXPLICIT:
                    case SIZE:
                    case EXPIRED:
                    case COLLECTED:
                    case REPLACED:
                }
            })
            .build();
}
```

### 3.5 定时清理

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.Scheduler;
import java.util.concurrent.TimeUnit;

public class SchedulerDemo {

    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .scheduler(Scheduler.systemScheduler())
            .build();
}
```

### 3.6 CaffeineConfig 5 个 Bean 详解

项目中 `CaffeineConfig.java` 定义了 5 个缓存 Bean，覆盖常用场景：

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.LoadingCache;
import com.github.benmanes.caffeine.cache.Scheduler;
import com.small.rose.demo.caffeine.entity.UserInfo;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.concurrent.TimeUnit;

@Configuration
public class CaffeineConfig {

    // ① userCache — 最大 500 条，10 分钟过期，开启统计
    @Bean("userCache")
    public Cache<String, UserInfo> userCache() {
        return Caffeine.newBuilder()
                .maximumSize(500)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats()
                .build();
    }

    // ② userLoadingCache — get(key) 未命中时自动加载，5 分钟异步刷新
    @Bean("userLoadingCache")
    public LoadingCache<String, UserInfo> userLoadingCache() {
        return Caffeine.newBuilder()
                .maximumSize(500)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .refreshAfterWrite(5, TimeUnit.MINUTES)
                .recordStats()
                .build(key -> new UserInfo(key, "auto-loaded-" + key, 25, "remark", "系统自动加载"));
    }

    // ③ orderCache — 30 秒极短过期，适合高频变化数据
    @Bean("orderCache")
    public Cache<String, String> orderCache() {
        return Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(30, TimeUnit.SECONDS)
                .recordStats()
                .build();
    }

    // ④ statsCache — 5 分钟无访问则过期，适合 Session 类数据
    @Bean("statsCache")
    public Cache<String, String> statsCache() {
        return Caffeine.newBuilder()
                .maximumSize(200)
                .expireAfterAccess(5, TimeUnit.MINUTES)
                .recordStats()
                .build();
    }

    // ⑤ scheduledCleanCache — 系统级定时清理，避免惰性删除堆积
    @Bean("scheduledCleanCache")
    public Cache<String, String> scheduledCleanCache() {
        return Caffeine.newBuilder()
                .maximumSize(500)
                .expireAfterWrite(1, TimeUnit.MINUTES)
                .scheduler(Scheduler.systemScheduler())
                .recordStats()
                .build();
    }
}
```

### 3.7 EvictionListener vs RemovalListener

| 特性 | RemovalListener | EvictionListener (Caffeine 3.x+) |
|---|---|---|
| **触发时机** | 所有移除（手动、淘汰、过期） | 仅淘汰（因容量/过期/Size） |
| **手动 invalidate** | ✅ 触发 | ❌ 不触发 |
| **可访问值** | `(key, value, cause)` | `(key, value, cause)` |
| **典型用途** | 日志、资源关闭 | 缓存命中统计、降级通知 |

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.RemovalCause;

public class ListenerComparison {

    Cache<String, String> removalExample = Caffeine.newBuilder()
            .maximumSize(1000)
            .removalListener((key, value, cause) -> {
                if (cause == RemovalCause.EXPLICIT) {
                    System.out.println("手动移除: " + key);
                } else if (cause == RemovalCause.SIZE) {
                    System.out.println("容量淘汰: " + key);
                }
            })
            .build();

    Cache<String, String> evictionExample = Caffeine.newBuilder()
            .maximumSize(1000)
            .evictionListener((key, value, cause) -> {
                System.out.println("淘汰: " + key);
            })
            .build();
}
```

---

## 4. 核心 API 使用

### 4.1 基本 CRUD

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;

public class BasicCrudDemo {

    public void crudExample() {
        Cache<String, UserInfo> cache = Caffeine.newBuilder()
                .maximumSize(500)
                .build();

        cache.put("user1", new UserInfo("user1", "张三", 28));
        UserInfo user = cache.getIfPresent("user1");
        user = cache.get("user1", key -> loadFromDatabase(key));
        cache.invalidate("user1");
        cache.invalidateAll();
        long size = cache.estimatedSize();
    }

    private UserInfo loadFromDatabase(String key) {
        return null;
    }
}
```

### 4.2 同步加载缓存（LoadingCache）

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.LoadingCache;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeUnit;

public class LoadingCacheDemo {

    private final LoadingCache<String, UserInfo> cache = Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .refreshAfterWrite(5, TimeUnit.MINUTES)
            .build(key -> {
                System.out.println("加载用户: " + key);
                return userRepository.findById(key).orElse(null);
            });

    public void demo() {
        UserInfo user = cache.get("user1");
        Map<String, UserInfo> users = cache.getAll(List.of("user1", "user2", "user3"));
    }
}
```

### 4.3 异步加载缓存（AsyncLoadingCache）

```java
import com.github.benmanes.caffeine.cache.AsyncLoadingCache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

public class AsyncLoadingCacheDemo {

    private final AsyncLoadingCache<String, UserInfo> asyncCache = Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .buildAsync(key -> CompletableFuture.supplyAsync(() ->
                    userRepository.findById(key).orElse(null)));

    public void demo() throws Exception {
        CompletableFuture<UserInfo> future = asyncCache.get("user1");
        UserInfo user = future.get();

        CompletableFuture<Map<String, UserInfo>> allFuture =
                asyncCache.getAll(List.of("user1", "user2"));
        Map<String, UserInfo> users = allFuture.get();
    }
}
```

### 4.4 批量操作

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.List;
import java.util.Map;

public class BatchOpsDemo {

    public void batchExample() {
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(100)
                .build();

        cache.put("a", "1");
        cache.put("b", "2");

        Map<String, String> present = cache.getAllPresent(List.of("a", "b", "c"));
        Map<String, String> all = cache.asMap();
    }
}
```

### 4.5 写入方式对比

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.Map;
import java.util.concurrent.CompletableFuture;

public class WritePatternsDemo {

    public void writeExamples() {
        Cache<String, String> cache = Caffeine.newBuilder().build();

        cache.put("key", "value");

        String value = cache.get("key", k -> "loaded-" + k);

        Map<String, String> all = cache.getAll(List.of("a", "b"), this::loadBatch);
    }

    private Map<String, String> loadBatch(Iterable<? extends String> keys) {
        return Map.of();
    }
}
```

### 4.6 弱引用 / 软引用（防止 OOM）

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.awt.image.BufferedImage;

public class ReferenceDemo {

    Cache<String, BufferedImage> weakCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .weakValues()
            .build();

    Cache<String, String> softCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .softValues()
            .build();
}
```

| 方法 | 回收时机 | 说明 |
|---|---|---|
| `weakKeys()` | GC 时 | key 无强引用即回收 |
| `weakValues()` | GC 时 | value 无强引用即回收 |
| `softValues()` | OOM 前 | value 在内存不足时回收 |

> **注意**：弱/软引用模式下，`RemovalListener` 不会在 GC 回收时立即触发。

### 4.7 Ticker（时间模拟 — 单元测试利器）

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.Ticker;
import java.util.concurrent.TimeUnit;

public class TickerDemo {

    public void tickerExample() {
        Ticker ticker = new Ticker() {
            private long nanos = 0;
            @Override public long read() { return nanos; }
            void advance(long n) { nanos += n; }
        };

        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(5)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .ticker(ticker::read)
                .build();
    }
}
```

> 结合 `FakeTicker`（Caffeine 测试工具）可精确控制时间，用于验证过期策略。

---

## 5. Spring Cache 集成

### 5.1 @Cacheable

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Cacheable(
        value = "user",
        key = "#userId",
        condition = "#userId != null",
        unless = "#result == null || #result.age < 18"
    )
    public UserInfo getUserById(String userId) {
        return userRepository.findById(userId).orElse(null);
    }
}
```

### 5.2 @CachePut

```java
import org.springframework.cache.annotation.CachePut;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @CachePut(value = "user", key = "#userId")
    public UserInfo updateUser(String userId, UserInfo user) {
        return userRepository.save(user);
    }
}
```

### 5.3 @CacheEvict

```java
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @CacheEvict(value = "user", key = "#userId")
    public void deleteUser(String userId) {
        userRepository.deleteById(userId);
    }

    @CacheEvict(value = "user", allEntries = true)
    public void clearAllCache() {
    }
}
```

### 5.4 @Caching 组合注解

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Caching;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private static final Logger log = LoggerFactory.getLogger(UserService.class);

    @Caching(
        evict = {
            @CacheEvict(value = "user", key = "#userId")
        },
        put = {
            @CachePut(value = "user", key = "'user2'", condition = "#userId == 'user1'")
        })
    public void transferData(String userId) {
        log.info(">>> 数据迁移操作: {}", userId);
    }
}
```

### 5.5 @CacheConfig 类级别配置

```java
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
@CacheConfig(cacheNames = "user")
public class UserService {

    @Cacheable(key = "#userId")
    public UserInfo getUserById(String userId) {
        return userRepository.findById(userId).orElse(null);
    }

    @CacheEvict(key = "#userId")
    public void deleteUser(String userId) {
        userRepository.deleteById(userId);
    }
}
```

### 5.6 自定义 KeyGenerator

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CacheKeyConfig {

    @Bean("customKeyGenerator")
    public KeyGenerator keyGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getSimpleName());
            sb.append(":");
            sb.append(method.getName());
            sb.append(":");
            for (Object param : params) {
                sb.append(param.toString()).append("_");
            }
            return sb.toString();
        };
    }
}

// 使用
@Service
public class UserService {

    @Cacheable(keyGenerator = "customKeyGenerator")
    public UserInfo getUser(String userId, String type) {
        return userRepository.findById(userId).orElse(null);
    }
}
```

### 5.7 条件缓存

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Cacheable(value = "user", key = "#userId + '_vip'", condition = "#username.contains('VIP')")
    public UserInfo getVipUser(String userId, String username) {
        return DB.get(userId);
    }

    @Cacheable(value = "user", unless = "#result == null")
    public UserInfo getUserOrNull(String userId) {
        return DB.get(userId);
    }
}
```

### 5.8 CaffeineCacheManager 原理与自定义

#### 原理

Spring Cache 通过 `CacheManager` 管理缓存实例。当配置 `spring.cache.type=caffeine` 时，Spring Boot 自动装配 `CaffeineCacheManager`：

```java
import com.github.benmanes.caffeine.cache.Cache;
import org.springframework.cache.caffeine.CaffeineCacheManager;

public class CacheManagerPrinciple {

    public void autoCreate() {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCacheSpecification("maximumSize=1000,expireAfterWrite=600s");
        Cache cache = manager.getCache("user");
    }
}
```

#### 自定义 — 多规格缓存

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Configuration
public class CustomCacheManagerConfig {

    @Primary
    @Bean("caffeineCacheManager")
    public CacheManager caffeineCacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCaffeine(Caffeine.newBuilder()
                .maximumSize(500)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats());
        manager.setCacheNames(Set.of("user", "order", "product"));
        return manager;
    }

    @Bean("shortCacheManager")
    public CacheManager shortCacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCaffeine(Caffeine.newBuilder()
                .maximumSize(100)
                .expireAfterWrite(30, TimeUnit.SECONDS));
        return manager;
    }
}
```

> 同一项目可共存多个 `CacheManager`，通过 `@Cacheable(cacheManager = "...")` 选择。

---

## 6. 多级缓存实战

### 6.1 架构设计

```
┌─────────────────────────────────────────────────┐
│                    应用层                         │
│  ┌──────────────┐  ┌──────────────┐             │
│  │  Controller  │  │   Service    │             │
│  └──────┬───────┘  └──────┬───────┘             │
│         │                 │                      │
│  ┌──────▼─────────────────▼───────┐             │
│  │      MultiLevelCacheService   │             │
│  └──────┬─────────────────┬───────┘             │
│         │                 │                      │
│  ┌──────▼───────┐  ┌──────▼───────┐             │
│  │   Caffeine   │  │    Redis     │             │
│  │  (本地缓存)   │  │ (分布式缓存) │             │
│  └──────┬───────┘  └──────┬───────┘             │
│         │                 │                      │
│  ┌──────▼─────────────────▼───────┐             │
│  │          数据库 (MySQL)        │             │
│  └───────────────────────────────┘             │
└─────────────────────────────────────────────────┘
```

### 6.2 读写流程

**读取流程：**
1. 先查 Caffeine 本地缓存 → 命中则返回
2. 再查 Redis 分布式缓存 → 命中则回填 Caffeine 并返回
3. 最后查数据库 → 回填两级缓存并返回

**写入流程：**
1. 更新数据库
2. 清除 Caffeine 本地缓存
3. 清除 Redis 分布式缓存

### 6.3 完整代码

```java
import com.github.benmanes.caffeine.cache.Cache;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;
import java.util.concurrent.TimeUnit;

@Service
public class MultiLevelCacheService {

    private static final Logger log = LoggerFactory.getLogger(MultiLevelCacheService.class);
    private final Cache<String, String> localCache;
    private final StringRedisTemplate redisTemplate;

    private static final String REDIS_PREFIX = "multi:cache:";
    private static final long REDIS_EXPIRE_MINUTES = 30;

    public MultiLevelCacheService(
            @Qualifier("statsCache") Cache<String, String> localCache,
            StringRedisTemplate redisTemplate) {
        this.localCache = localCache;
        this.redisTemplate = redisTemplate;
    }

    public String get(String key) {
        String value = localCache.getIfPresent(key);
        if (value != null) {
            log.info("命中 Caffeine: {}", key);
            return value;
        }

        String redisKey = REDIS_PREFIX + key;
        value = redisTemplate.opsForValue().get(redisKey);
        if (value != null) {
            log.info("命中 Redis: {}", key);
            localCache.put(key, value);
            return value;
        }

        log.info("查询数据库: {}", key);
        value = loadFromDatabase(key);

        if (value != null) {
            localCache.put(key, value);
            redisTemplate.opsForValue().set(redisKey, value, REDIS_EXPIRE_MINUTES, TimeUnit.MINUTES);
        }

        return value;
    }

    public void put(String key, String value) {
        saveToDatabase(key, value);
        localCache.invalidate(key);
        redisTemplate.delete(REDIS_PREFIX + key);
    }

    private String loadFromDatabase(String key) {
        // 模拟数据库查询
        return null;
    }

    private void saveToDatabase(String key, String value) {
        // 模拟数据库写入
    }
}
```

---

## 7. 高级应用场景

### 7.1 缓存预热

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class CacheWarmUp implements CommandLineRunner {

    private static final Logger log = LoggerFactory.getLogger(CacheWarmUp.class);

    @Autowired
    private UserService userService;

    @Override
    public void run(String... args) {
        log.info(">>> 开始缓存预热...");

        userService.getUserById("user1");
        userService.getUserById("user2");
        userService.getUserById("user3");

        log.info(">>> 缓存预热完成");
    }
}
```

### 7.2 缓存击穿防护

**方案 1：sync = true（推荐）**

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Cacheable(value = "user", key = "#userId", sync = true)
    public UserInfo getUserById(String userId) {
        return userRepository.findById(userId).orElse(null);
    }
}
```

**`sync=true` 底层原理**：
- `@Cacheable(sync=true)` 映射到 Caffeine 的 `get(key, callable)` 原子操作
- 内部使用 `ConcurrentHashMap.computeIfAbsent` 保证同一 key 仅一个线程执行加载
- 这与 `synchronized` 不同：粒度是 **key 级别**而非方法级别，不同 key 的请求可并发
- 配合 `LoadingCache` 或 `AsyncLoadingCache` 效果更佳

**方案 2：手动加锁**

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.concurrent.locks.ReentrantLock;

public class ManualLockGuard {

    private final Cache<String, UserInfo> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();

    private final Cache<String, ReentrantLock> lockCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .build();

    public UserInfo getUserById(String userId) {
        UserInfo user = cache.getIfPresent(userId);
        if (user != null) {
            return user;
        }

        ReentrantLock lock = lockCache.get(userId, k -> new ReentrantLock());
        lock.lock();
        try {
            user = cache.getIfPresent(userId);
            if (user != null) {
                return user;
            }
            user = userRepository.findById(userId).orElse(null);
            if (user != null) {
                cache.put(userId, user);
            }
            return user;
        } finally {
            lock.unlock();
        }
    }
}
```

**方案 3：LoadingCache（自动防护）**

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.LoadingCache;

public class LoadingCacheGuard {

    private final LoadingCache<String, UserInfo> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build(key -> userRepository.findById(key).orElse(null));
}
```

### 7.3 缓存穿透防护

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    // 空值缓存：结果为 null 时不缓存
    @Cacheable(value = "user", key = "#userId", unless = "#result == null")
    public UserInfo getUserById(String userId) {
        return userRepository.findById(userId).orElse(null);
    }

    // Bloom Filter 前置判断
    @Cacheable(value = "user", key = "#userId", condition = "@bloomFilter.mightContain(#userId)")
    public UserInfo getWithBloomFilter(String userId) {
        return userRepository.findById(userId).orElse(null);
    }
}
```

### 7.4 缓存雪崩防护

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.Expiry;
import java.time.Duration;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;

public class AvalancheGuard {

    private Duration getRandomExpire() {
        long base = 10;
        long random = ThreadLocalRandom.current().nextLong(5);
        return Duration.ofMinutes(base + random);
    }

    public void buildWithRandomExpiry() {
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfter(new Expiry<String, String>() {
                    @Override
                    public long expireAfterCreate(String key, String value, long currentTime) {
                        return getRandomExpire().toNanos();
                    }
                    @Override
                    public long expireAfterUpdate(String key, String value,
                            long currentTime, long currentDuration) {
                        return currentDuration;
                    }
                    @Override
                    public long expireAfterRead(String key, String value,
                            long currentTime, long currentDuration) {
                        return currentDuration;
                    }
                })
                .build();
    }
}
```

---

## 8. CXF WebService 客户端缓存

### 8.1 问题场景

在微服务架构中，Web Service 客户端的创建是昂贵的操作：
- 需要解析 WSDL
- 创建代理对象
- 配置 HTTP 连接
- 建立 TCP 连接

如果每次调用都重新创建客户端，会导致：
- 性能下降
- 资源浪费
- 连接数耗尽

### 8.2 CxfCaffWebServiceUtil 实现

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class CxfCaffWebServiceUtil {

    private static final Logger log = LoggerFactory.getLogger(CxfCaffWebServiceUtil.class);

    // 动态客户端缓存：30 分钟过期，最大 200 个
    private final Cache<String, Client> dynClientCache = Caffeine.newBuilder()
            .maximumSize(200)
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .removalListener((key, client, cause) -> {
                if (client != null) {
                    try {
                        client.close();
                    } catch (Exception ignore) {}
                }
                log.info("客户端缓存移除: key={}, cause={}", key, cause);
            })
            .recordStats()
            .build();

    // 代理客户端缓存：60 分钟过期
    private final Cache<String, Object> proxyCache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(60, TimeUnit.MINUTES)
            .build();

    // 失败计数：5 分钟过期
    private final Cache<String, AtomicInteger> failCounter = Caffeine.newBuilder()
            .expireAfterWrite(5, TimeUnit.MINUTES)
            .build();

    // 获取或创建动态客户端
    private Client getOrCreateDyn(String key, String wsdlUrl, Object... args) {
        return dynClientCache.get(key, k -> {
            Client client = getClientFactory().createClient(wsdlUrl);
            // 配置超时...
            return client;
        });
    }

    // 获取代理客户端
    public <T> T getProxy(String wsdlUrl, Class<T> serviceClass) {
        String key = proxyCacheKey(wsdlUrl, serviceClass);
        T proxy = (T) proxyCache.getIfPresent(key);
        if (proxy != null) {
            return proxy;
        }
        // 创建代理...
        proxyCache.put(key, proxy);
        return proxy;
    }
}
```

### 8.3 对比原版手写 LRU

| 特性 | 原版 CxfWebServiceUtil | CxfCaffWebServiceUtil |
|---|---|---|
| **并发安全** | ReentrantReadWriteLock | Caffeine 内部 CAS |
| **性能** | 锁竞争 | 无锁并发 |
| **淘汰算法** | LRU（固定顺序） | W-TinyLFU（频率感知） |
| **过期策略** | 无 | expireAfterWrite |
| **统计功能** | 无 | Cache.stats() |
| **异步加载** | 不支持 | AsyncLoadingCache |
| **代码量** | ~300 行 | ~200 行 |

---

## 9. 监控与运维

### 9.1 CacheStats 统计

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.stats.CacheStats;

public class CacheStatsDemo {

    public void statsExample() {
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(1000)
                .recordStats()
                .build();

        // 获取统计信息
        CacheStats stats = cache.stats();
        System.out.println("命中次数: " + stats.hitCount());
        System.out.println("未命中次数: " + stats.missCount());
        System.out.println("命中率: " + stats.hitRate());
        System.out.println("加载次数: " + stats.loadCount());
        System.out.println("加载成功次数: " + stats.loadSuccessCount());
        System.out.println("加载失败次数: " + stats.loadFailureCount());
        System.out.println("淘汰次数: " + stats.evictionCount());
    }
}
```

### 9.2 Spring Boot Actuator 集成

**build.gradle**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

**application.yml**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: caches,cache
  endpoint:
    caches:
      enabled: true
    cache:
      enabled: true
```

**访问监控端点**
```bash
# 查看所有缓存
curl http://localhost:31003/boot3-demo/actuator/caches

# 查看指定缓存
curl http://localhost:31003/boot3-demo/actuator/caches/user

# 清空缓存
curl -X POST http://localhost:31003/boot3-demo/actuator/caches/user
```

### 9.3 Micrometer 指标集成

Caffeine 原生支持 Micrometer（Spring Boot 3 默认引入），可直接接入 Prometheus/Grafana：

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.binder.cache.CaffeineCacheMetrics;

public class MicrometerDemo {

    private final MeterRegistry meterRegistry;

    public MicrometerDemo(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void registerMetrics() {
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(1000)
                .recordStats()
                .build();

        // 注册到 Micrometer（集成到 Actuator /metrics）
        CaffeineCacheMetrics.monitor(meterRegistry, cache, "my-cache");
    }
}
```

**应用配置文件（Prometheus 接入）**：
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    tags:
      application: ${spring.application.name}
```

**Micrometer 自动采集的指标**：

| 指标名 | 类型 | 说明 |
|---|---|---|
| `cache.gets` | Counter | 查询次数（tag: `result=hit/miss`） |
| `cache.puts` | Counter | 写入次数 |
| `cache.evictions` | Counter | 淘汰次数（tag: `cause=size/weight/expired`） |
| `cache.load.duration` | Timer | 加载耗时 |
| `cache.size` | Gauge | 当前大小 |

> 确保 `Cache` 实例调用了 `.recordStats()`，否则指标为 0。

### 9.4 运维建议

1. **监控命中率**：命中率低于 80% 时需要调整缓存策略
2. **监控淘汰数**：淘汰数过高说明容量不足
3. **定期清理**：使用 `scheduler` 定时清理过期条目
4. **日志记录**：使用 `RemovalListener` 记录移除事件
5. **告警规则**：命中率突降 >10% / 淘汰数突增 >100% 时告警

---

## 10. 最佳实践

### 10.1 容量规划

| 场景 | 建议容量 | 过期时间 |
|---|---|---|
| 用户信息 | 1000-5000 | 10-30 分钟 |
| 配置信息 | 100-500 | 1-24 小时 |
| 菜单权限 | 500-2000 | 30-60 分钟 |
| 数据字典 | 1000-5000 | 1-24 小时 |
| 热点数据 | 5000-10000 | 5-15 分钟 |

### 10.2 过期时间设置

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.concurrent.TimeUnit;

public class ExpiryPolicyDemo {

    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
}
```

### 10.3 常见陷阱

#### 陷阱 1：缓存对象是引用

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    // 错误：缓存的是引用，外部修改会影响缓存
    @Cacheable("user")
    public User getUser(String id) {
        return userRepository.findById(id);
    }

    public void wrongUsage() {
        User user = getUser("1");
        user.setName("修改");  // 会影响缓存中的值
    }

    // 正确：返回副本或不可变对象
    @Cacheable("user")
    public UserDTO getUserSafe(String id) {
        User user = userRepository.findById(id);
        return new UserDTO(user);  // 返回新对象
    }
}
```

#### 陷阱 2：循环调用

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class CircularService {

    // 错误：A 调用 B，B 调用 A
    @Cacheable("a")
    public void methodA() {
        methodB();  // B 的缓存会失效
    }

    @Cacheable("b")
    public void methodB() {
        methodA();  // A 的缓存会失效
    }
}
```

#### 陷阱 3：非 public 方法

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    // 错误：Spring AOP 无法代理 private/protected 方法
    @Cacheable("user")
    private UserInfo getUser(String id) {  // 不会生效
        return userRepository.findById(id);
    }
}
```

#### 陷阱 4：void 方法上的 @CacheEvict 与 @Transactional 顺序

```java
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    // 注意：@CacheEvict 默认在方法执行后生效（afterInvocation=true）
    // 若同时有 @Transactional，缓存操作在事务提交前执行
    @Transactional
    @CacheEvict(value = "user", key = "#id")
    public void deleteUser(String id) {
        userRepository.deleteById(id);
        // 此时缓存已清除，但事务可能还未提交
        // 若事务回滚，缓存已丢失 → 数据不一致
    }

    // 解决：使用 beforeInvocation=true（方法执行前清缓存）
    // 或使用 CacheEvict 的事务同步模式
    @Transactional
    @CacheEvict(value = "user", key = "#id", beforeInvocation = true)
    public void deleteUser(String id) {
        userRepository.deleteById(id);
    }
}
```

### 10.4 调优建议

1. **开启统计**：`.recordStats()` 监控命中率
2. **合理设置容量**：避免 OOM
3. **使用 refreshAfterWrite**：避免缓存击穿
4. **异步加载**：使用 AsyncLoadingCache 提升性能
5. **监控 JVM 内存**：定期检查缓存占用

---

## 11. 完整实战案例

### 11.1 项目结构

```
src/main/java/com/small/rose/demo/caffeine/
├── config/
│   └── CaffeineConfig.java              # 缓存配置
├── entity/
│   └── UserInfo.java                    # 示例实体
├── service/
│   ├── UserService.java                 # Spring Cache 示例
│   └── MultiLevelCacheService.java      # 多级缓存
├── listener/
│   └── CacheEventDemo.java              # 移除监听
├── async/
│   └── AsyncCacheDemo.java              # 异步缓存
├── controller/
│   └── CacheDemoController.java         # REST 接口
└── util/
    └── CxfCaffWebServiceUtil.java       # CXF 客户端缓存
```

### 11.2 测试类

```
src/test/java/com/small/rose/demo/caffeine/
├── CaffeineBasicTest.java               # 基础 API 测试
├── CaffeineSpringCacheTest.java         # Spring Cache 测试
├── CaffeineMultiLevelTest.java          # 多级缓存测试
├── CaffeineListenerTest.java            # 监听器测试
├── CaffeineAsyncTest.java               # 异步缓存测试
└── CxfCaffWebServiceUtilTest.java       # CXF 工具测试
```

### 11.3 REST 接口

| 接口 | 方法 | 说明 |
|---|---|---|
| `/caffeine/user/{userId}` | GET | 查询用户（@Cacheable） |
| `/caffeine/users` | GET | 查询所有用户 |
| `/caffeine/user/{userId}` | PUT | 更新用户（@CachePut） |
| `/caffeine/user/{userId}` | DELETE | 删除用户（@CacheEvict） |
| `/caffeine/cache/clear` | DELETE | 清空所有缓存（@CacheEvict allEntries） |
| `/caffeine/cache/user/{userId}` | GET | 手动查看 Caffeine 缓存 |
| `/caffeine/cache/user` | POST | 手动写入 Caffeine 缓存 |
| `/caffeine/cache/user/{userId}` | DELETE | 手动移除 Caffeine 缓存 |
| `/caffeine/cache/stats` | GET | 查看缓存统计 |
| `/caffeine/cache/event/demo` | GET | 演示移除事件 |
| `/caffeine/async/{key}` | GET | 异步获取缓存 |
| `/caffeine/async/{key}/process` | GET | 异步获取并处理（thenApply） |

### 11.4 运行测试

```bash
# 运行所有 Caffeine 测试
./gradlew test --tests "com.small.rose.demo.caffeine.*"

# 运行单个测试类
./gradlew test --tests "com.small.rose.demo.caffeine.CaffeineBasicTest"

# 运行单个测试方法
./gradlew test --tests "com.small.rose.demo.caffeine.CaffeineBasicTest.testBasicOperations"
```

---

## 附录：常用配置速查

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.github.benmanes.caffeine.cache.Scheduler;
import java.util.concurrent.TimeUnit;
import static java.util.concurrent.TimeUnit.MINUTES;

public class ConfigReference {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(1000)                    // 最大容量
            .maximumWeight(10000)                 // 最大权重
            .weigher((k, v) -> v.length())        // 权重计算器
            .expireAfterWrite(10, MINUTES)        // 写入后过期
            .expireAfterAccess(5, MINUTES)        // 访问后过期
            .refreshAfterWrite(5, MINUTES)        // 写入后刷新
            .recordStats()                        // 开启统计
            .scheduler(Scheduler.systemScheduler()) // 定时清理
            .removalListener((k, v, c) -> {})     // 移除监听
            .evictionListener((k, v, c) -> {})    // 淘汰监听（Caffeine 3.x+）
            .weakValues()                         // 弱引用值（GC 可回收）
            .softValues()                         // 软引用值（OOM 前回收）
            .ticker(ticker::read)                 // 自定义时间源（测试）
            .initialCapacity(100)                 // 初始容量
            .build();                             // 构建
}
```

---

**文档版本**: Caffeine 

**适用环境**: Spring Boot 3.5.x + Java 17 + Caffeine 3.2.x  

*最后更新时间：2026-06-16*
