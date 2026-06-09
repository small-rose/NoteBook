---
layout: default
title: @EnableConfigurationProperties
parent: SpringBoot3
has_children: false
nav_order: 82
---


Here are webservices of CXF used examples on springboot3 .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }


# Spring Session 版本演进与使用指南

## 目录

1. [概述](#概述)
2. [Spring Boot 1.5.x 时代](#spring-boot-15x-时代)
3. [Spring Boot 2.5.x 重大变革](#spring-boot-25x-重大变革)
4. [Spring Boot 3.5.x 当前最佳实践](#spring-boot-35x-当前最佳实践)
5. [版本对比总表](#版本对比总表)
6. [存储类型详解](#存储类型详解)
7. [JDBC Schema DDL](#jdbc-schema-ddl)
8. [常见问题与解决方案](#常见问题与解决方案)
9. [迁移指南](#迁移指南)

---

## 概述

Spring Session 提供了一套管理用户会话信息的基础设施，支持将 HttpSession 存储到 Redis、JDBC、Hazelcast、MongoDB 等外部存储，实现会话共享和分布式部署。

在不同版本的 Spring Boot 中，Spring Session 的配置方式经历了多次演变：

| 版本 | 配置方式 | 自动化程度 |
|------|----------|------------|
| 1.5.x | `@Enable*HttpSession` 显式声明 | 需手动配置 |
| 2.5.x | 依赖自动检测 + 属性配置 | 高度自动化 |
| 3.5.x | 完全自动配置 + Jakarta EE | 完全自动化 |

---

## Spring Boot 1.5.x 时代

### 核心机制

Spring Boot 1.5.x 配合 Spring Session 1.x，需要**显式使用注解**启用 Session 存储。

### Redis 存储配置

#### 依赖配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

#### 配置类

```java
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class RedisSessionConfig {
    
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory();
    }
}
```

#### application.properties

```properties
# Session 配置
spring.session.store-type=redis
server.servlet.session.timeout=1800

# Redis 配置（1.5.x 使用 spring.redis.*）
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
```

### JDBC 存储配置

#### 配置类

```java
@Configuration
@EnableJdbcHttpSession
public class JdbcSessionConfig {
    
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/session_db")
            .username("root")
            .password("password")
            .build();
    }
}
```

#### application.properties

```properties
spring.session.store-type=jdbc
spring.session.jdbc.initialize-schema=always
spring.session.jdbc.table-name=SPRING_SESSION
```

### 1.5.x 的局限性

1. **必须显式声明注解**：`@EnableRedisHttpSession` 或 `@EnableJdbcHttpSession`
2. **模块未拆分**：所有 SessionRepository 实现在 `spring-session` 包中
3. **配置繁琐**：需要手动配置 ConnectionFactory、DataSource 等
4. **无自动检测**：不能根据 classpath 自动选择存储类型

---

## Spring Boot 2.5.x 重大变革

### 核心变化

Spring Boot 2.x 配合 Spring Session 2.x，实现了**高度自动化配置**。

### 模块拆分

Spring Session 2.0 将项目拆分为多个模块：

```
spring-session-core          # 核心 API
spring-session-data-redis    # Redis 实现
spring-session-jdbc          # JDBC 实现
spring-session-hazelcast     # Hazelcast 实现
spring-session-data-mongodb  # MongoDB 实现
```

### 自动配置机制

Spring Boot 2.x 自动配置类：`SessionRepositoryAutoConfiguration`

```java
@AutoConfiguration
@ConditionalOnClass({SessionRepository.class, RedisOperationsSessionRepository.class})
@ConditionalOnMissingBean(SessionRepository.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@EnableConfigurationProperties(SessionProperties.class)
public class RedisHttpSessionConfiguration {
    // 自动创建 springSessionRepositoryFilter
}
```

### Redis 存储配置

#### 依赖配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

#### application.properties

```properties
# 无需任何配置类，只需添加依赖
spring.session.store-type=redis

# Redis 配置（2.x 使用 spring.data.redis.*）
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=

# Session 配置
spring.session.timeout=1800s
spring.session.redis.flush-mode=on-save
spring.session.redis.namespace=spring:session
```

### JDBC 存储配置

#### 依赖配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-jdbc</artifactId>
    </dependency>
</dependencies>
```

#### application.properties

```properties
spring.session.store-type=jdbc

# JDBC 配置
spring.datasource.url=jdbc:mysql://localhost:3306/session_db
spring.datasource.username=root
spring.datasource.password=password

# Schema 初始化
spring.session.jdbc.initialize-schema=embedded
spring.session.jdbc.table-name=SPRING_SESSION
```

### Hazelcast 存储配置

#### 依赖配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-hazelcast</artifactId>
    </dependency>
    <dependency>
        <groupId>com.hazelcast</groupId>
        <artifactId>hazelcast</artifactId>
    </dependency>
</dependencies>
```

#### 配置类

```java
@Configuration
@EnableHazelcastHttpSession
public class HazelcastSessionConfig {
    
    @Bean
    public HazelcastInstance hazelcastInstance() {
        Config config = new Config();
        config.getNetworkConfig().setPort(5701);
        return Hazelcast.newHazelcastInstance(config);
    }
}
```

### 2.5.x 的优势

1. **自动检测存储类型**：只需添加依赖，无需注解
2. **模块化设计**：按需引入依赖
3. **配置简化**：通过 `spring.session.*` 属性统一配置
4. **Schema 自动初始化**：JDBC 存储自动创建表结构

---

## Spring Boot 3.5.x 当前最佳实践

### 核心变化

Spring Boot 3.x 配合 Spring Session 3.x，主要变化：

1. **Jakarta EE 迁移**：`javax.servlet` → `jakarta.servlet`
2. **完全自动配置**：无需 `@Enable*HttpSession` 注解
3. **响应式支持**：WebFlux 会话管理
4. **MongoDB 支持**：新增 MongoDB 存储选项

### Redis 存储配置

#### 依赖配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

#### application.yml

```yaml
spring:
  session:
    store-type: redis
    timeout: 30m
    redis:
      flush-mode: on-save
      namespace: spring:session
      repository-type: indexed  # 或 default
  data:
    redis:
      host: localhost
      port: 6379
      password: ${REDIS_PASSWORD:}
```

#### Redis Session Repository 选择

Spring Session 3.x 提供两种 Redis 存储实现：

```
// 默认：RedisSessionRepository（推荐）
// 基于哈希存储，性能更高
spring.session.redis.repository-type=default
```

```
// 可选：RedisIndexedSessionRepository
// 支持按属性查询，但性能稍低
spring.session.redis.repository-type=indexed
```

### JDBC 存储配置

#### 依赖配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-jdbc</artifactId>
    </dependency>
</dependencies>
```

#### application.yml

```yaml
spring:
  session:
    store-type: jdbc
    timeout: 30m
    jdbc:
      initialize-schema: embedded
      table-name: SPRING_SESSION
  datasource:
    url: jdbc:mysql://localhost:3306/session_db
    username: root
    password: ${DB_PASSWORD:}
```

### 响应式会话配置

Spring Boot 3.x 支持 WebFlux 响应式会话：

```yaml
spring:
  session:
    store-type: redis
```

```java
@Configuration
@EnableRedisWebSession
public class ReactiveSessionConfig {
    // 无需额外配置，自动使用响应式会话存储
}
```

### 自定义 Session 序列化

```java
@Configuration
public class SessionConfig {
    
    @Bean
    public RedisSerializer<Object> springSessionRedisSerializer() {
        return new GenericJackson2JsonRedisSerializer();
    }
}
```

### Session 事件监听

```java
@Component
public class SessionEventListener {
    
    @EventListener
    public void onSessionCreated(SessionCreatedEvent event) {
        System.out.println("Session created: " + event.getSessionId());
    }
    
    @EventListener
    public void onSessionDestroyed(SessionDestroyedEvent event) {
        System.out.println("Session destroyed: " + event.getSessionId());
    }
}
```

---

## 版本对比总表

| 特性 | 1.5.x | 2.5.x | 3.5.x |
|------|-------|-------|-------|
| Spring Session 版本 | 1.x | 2.x | 3.x |
| Servlet API | javax.servlet | javax.servlet | jakarta.servlet |
| 配置方式 | `@Enable*HttpSession` | 自动检测 | 完全自动 |
| 模块拆分 | ❌ | ✅ | ✅ |
| store-type 配置 | 必须 | 推荐 | 可选 |
| Redis 配置前缀 | `spring.redis.*` | `spring.data.redis.*` | `spring.data.redis.*` |
| Schema 自动初始化 | 手动 | ✅ | ✅ |
| 响应式支持 | ❌ | 部分 | ✅ |
| RedisIndexedRepository | ❌ | ✅ | ✅ |
| Session 事件 | ✅ | ✅ | ✅ |

---

## 存储类型详解

### 存储类型选择

| 存储类型 | 适用场景 | 依赖包 |
|----------|----------|--------|
| Redis | 分布式会话、高性能 | `spring-session-data-redis` |
| JDBC | 关系型数据库、事务支持 | `spring-session-jdbc` |

```yaml
# 指定存储类型
spring:
  session:
    store-type: redis  # 或 jdbc
```

---

## JDBC Schema DDL

Spring Session JDBC 使用两张表存储会话数据：

| 表名 | 用途 |
|------|------|
| `SPRING_SESSION` | 存储会话基本信息 |
| `SPRING_SESSION_ATTRIBUTES` | 存储会话属性 |

### 版本差异

| 版本 | 主键 | 新增列 | 外键关联 | 状态 |
|------|------|--------|----------|------|
| 1.x | `SESSION_ID` | 无 | `SESSION_ID` | ❌ 已废弃 |
| 2.x | `PRIMARY_ID` | `EXPIRY_TIME` | `SESSION_PRIMARY_ID` | ✅ 推荐 |
| 3.x | `PRIMARY_ID` | `EXPIRY_TIME` | `SESSION_PRIMARY_ID` | ✅ 推荐 |

**⚠️ 重要**：
- **以下 DDL 适用于 2.x/3.x 版本**
- 1.x 版本的表结构已废弃，不建议在新项目中使用
- 从 1.x 升级到 2.x/3.x 需要执行 DDL 迁移，详见[迁移指南](#jdbc-模式迁移)

---

### 1.x 旧表结构（已废弃）

```sql
-- Spring Session 1.x DDL（不推荐使用）
CREATE TABLE SPRING_SESSION (
    SESSION_ID CHAR(36) NOT NULL,          -- 主键是 SESSION_ID
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (SESSION_ID)
) ENGINE=InnoDB;

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
    SESSION_ID CHAR(36) NOT NULL,          -- 外键关联 SESSION_ID
    ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
    ATTRIBUTE_BYTES BLOB NOT NULL,
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_ID)
        REFERENCES SPRING_SESSION(SESSION_ID) ON DELETE CASCADE
) ENGINE=InnoDB;
```

---

### 2.x/3.x 新表结构（推荐）

#### MySQL DDL

```sql
-- SPRING_SESSION 表（会话基本信息）
CREATE TABLE SPRING_SESSION (
    PRIMARY_ID CHAR(36) NOT NULL,
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    EXPIRY_TIME BIGINT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

-- 索引
CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

-- SPRING_SESSION_ATTRIBUTES 表（会话属性）
CREATE TABLE SPRING_SESSION_ATTRIBUTES (
    SESSION_PRIMARY_ID CHAR(36) NOT NULL,
    ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
    ATTRIBUTE_BYTES BLOB NOT NULL,
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID)
        REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```

### Oracle DDL

```sql
CREATE TABLE SPRING_SESSION (
    PRIMARY_ID CHAR(36) NOT NULL,
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME NUMBER(19) NOT NULL,
    LAST_ACCESS_TIME NUMBER(19) NOT NULL,
    MAX_INACTIVE_INTERVAL NUMBER(10) NOT NULL,
    EXPIRY_TIME NUMBER(19) NOT NULL,
    PRINCIPAL_NAME VARCHAR2(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
);

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
    SESSION_PRIMARY_ID CHAR(36) NOT NULL,
    ATTRIBUTE_NAME VARCHAR2(200) NOT NULL,
    ATTRIBUTE_BYTES BLOB NOT NULL,
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID)
        REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
);
```

### PostgreSQL DDL

```sql
CREATE TABLE SPRING_SESSION (
    PRIMARY_ID CHAR(36) NOT NULL,
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    EXPIRY_TIME BIGINT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
);

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
    SESSION_PRIMARY_ID CHAR(36) NOT NULL,
    ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
    ATTRIBUTE_BYTES BYTEA NOT NULL,
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID)
        REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
);
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `PRIMARY_ID` | CHAR(36) | 主键（UUID） |
| `SESSION_ID` | CHAR(36) | 会话 ID（唯一） |
| `CREATION_TIME` | BIGINT | 创建时间（毫秒时间戳） |
| `LAST_ACCESS_TIME` | BIGINT | 最后访问时间（毫秒时间戳） |
| `MAX_INACTIVE_INTERVAL` | INT | 最大非活动间隔（秒） |
| `EXPIRY_TIME` | BIGINT | 过期时间（毫秒时间戳） |
| `PRINCIPAL_NAME` | VARCHAR(100) | 主体名称（用户标识） |
| `SESSION_PRIMARY_ID` | CHAR(36) | 外键关联 SPRING_SESSION.PRIMARY_ID |
| `ATTRIBUTE_NAME` | VARCHAR(200) | 属性名称 |
| `ATTRIBUTE_BYTES` | BLOB/BYTEA | 属性值（序列化后的字节数组） |

### Schema 初始化配置

```yaml
spring:
  session:
    jdbc:
      # Schema 初始化模式
      # always: 每次启动都初始化
      # embedded: 仅嵌入式数据库（H2/HSQL）
      # never: 从不初始化
      initialize-schema: always

      # 自定义表名
      table-name: SPRING_SESSION

      # 自定义 Schema 文件路径
      schema: classpath:org/springframework/session/jdbc/schema-@@platform@@.sql
```

### 自定义表名

```java
@Configuration
@EnableJdbcHttpSession(tableName = "MY_SESSION")
public class SessionConfig {
    // 会创建 MY_SESSION 和 MY_SESSION_ATTRIBUTES 两张表
}
```

---

## 常见问题与解决方案

### 问题一：Session 不生效

**错误信息**：Session 数据没有持久化到外部存储

**原因**：缺少 `@Enable*HttpSession` 注解（1.5.x）或依赖未正确引入

**解决方案**：

```xml
<!-- 2.x/3.x 只需添加依赖 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

### 问题二：Redis 连接失败

**错误信息**：`Unable to connect to Redis`

**解决方案**：

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: ${REDIS_PASSWORD:}
      timeout: 5000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
```

### 问题三：JDBC Schema 初始化失败

**错误信息**：`Table "SPRING_SESSION" not found`

**解决方案**：

```yaml
spring:
  session:
    jdbc:
      initialize-schema: always  # 或 embedded
```

### 问题四：多实例 Session 不同步

**原因**：各实例连接不同的 Redis 实例

**解决方案**：确保所有实例连接同一个 Redis 集群

```yaml
spring:
  data:
    redis:
      cluster:
        nodes:
          - redis1:6379
          - redis2:6379
          - redis3:6379
```

### 问题五：Session 超时设置不生效

**解决方案**：

```yaml
spring:
  session:
    timeout: 30m  # 使用 Duration 格式
    # 或
    timeout: 1800  # 秒（1.5.x/2.x）
```

---

## 迁移指南

### Redis 模式迁移

#### 从 1.5.x 迁移到 2.5.x

##### 1. 更新依赖

```xml
<!-- 旧依赖 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- 新依赖（模块拆分） -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

##### 2. 移除注解

```java
// 旧代码（1.5.x 必须）
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SessionConfig {
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory();
    }
}

// 新代码（2.5.x 自动配置，删除此类即可）
// 无需任何配置类
```

##### 3. 更新配置属性

```properties
# 旧配置（1.5.x）
spring.redis.host=localhost
spring.redis.port=6379
spring.session.store-type=redis
```

```properties
# 新配置（2.5.x）
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.session.store-type=redis
```

#### 从 2.5.x 迁移到 3.5.x

##### 1. Jakarta EE 迁移

```java
// 旧代码（2.5.x 使用 javax）
import javax.servlet.http.HttpSession;
import javax.servlet.http.HttpServletRequest;

// 新代码（3.5.x 使用 jakarta）
import jakarta.servlet.http.HttpSession;
import jakarta.servlet.http.HttpServletRequest;
```

##### 2. 更新依赖版本

```xml
<!-- 版本由 Spring Boot BOM 管理，无需指定版本号 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

##### 3. 配置变化

```yaml
# 2.5.x 配置
spring:
  session:
    store-type: redis
    timeout: 1800s
  data:
    redis:
      host: localhost

# 3.5.x 配置（基本相同，无需修改）
spring:
  session:
    store-type: redis
    timeout: 30m  # 支持 Duration 格式
  data:
    redis:
      host: localhost
```

##### 4. Redis Repository 选择（3.x 新增）

```yaml
# 3.x 新增 Redis Session Repository 选择
spring:
  session:
    redis:
      # 默认：RedisSessionRepository（推荐，性能更高）
      repository-type: default
      # 可选：RedisIndexedSessionRepository（支持按属性查询）
      repository-type: indexed
```

---

### JDBC 模式迁移

#### 从 1.5.x 迁移到 2.5.x

##### 1. 更新依赖

```xml
<!-- 旧依赖 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>

<!-- 新依赖（模块拆分） -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-jdbc</artifactId>
</dependency>
```

##### 2. 移除注解

```java
// 旧代码（1.5.x 必须）
@Configuration
@EnableJdbcHttpSession(tableName = "SPRING_SESSION")
public class SessionConfig {
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/session_db")
            .username("root")
            .password("password")
            .build();
    }
}

// 新代码（2.5.x 自动配置，保留 DataSource 配置即可）
@Configuration
public class DataSourceConfig {
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/session_db")
            .username("root")
            .password("password")
            .build();
    }
}
```

##### 3. 更新配置属性

```properties
# 旧配置（1.5.x）
spring.session.store-type=jdbc

# 新配置（2.5.x）
spring.session.store-type=jdbc
spring.session.jdbc.initialize-schema=embedded  # 新增：自动初始化 Schema
spring.session.jdbc.table-name=SPRING_SESSION   # 新增：自定义表名
```

##### 4. Schema 初始化

2.5.x 新增自动 Schema 初始化功能：

```yaml
spring:
  session:
    jdbc:
      # 初始化模式
      # always: 每次启动都初始化
      # embedded: 仅嵌入式数据库（H2/HSQL）
      # never: 从不初始化
      initialize-schema: always
```

**注意**：1.5.x 需要手动执行 DDL，2.5.x 可自动初始化。

#### 从 2.5.x 迁移到 3.5.x

##### 1. Jakarta EE 迁移

```java
// 旧代码（2.5.x 使用 javax）
import javax.servlet.http.HttpSession;

// 新代码（3.5.x 使用 jakarta）
import jakarta.servlet.http.HttpSession;
```

##### 2. 依赖变化

```xml
<!-- 依赖名称不变，版本由 BOM 管理 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-jdbc</artifactId>
</dependency>
```

##### 3. Schema DDL 变化

**重要**：Spring Session 1.x 到 2.x/3.x 的表结构**有重大变化**，需要执行 DDL 迁移。

```sql
-- 1.x 旧表结构（主键是 SESSION_ID）
CREATE TABLE SPRING_SESSION (
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (SESSION_ID)
);

-- 2.x/3.x 新表结构（主键改为 PRIMARY_ID，新增 EXPIRY_TIME）
CREATE TABLE SPRING_SESSION (
    PRIMARY_ID CHAR(36) NOT NULL,
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    EXPIRY_TIME BIGINT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
);
```

**DDL 迁移脚本（MySQL）**：

```sql
-- 步骤 1：备份旧数据（可选）
CREATE TABLE SPRING_SESSION_BACKUP AS SELECT * FROM SPRING_SESSION;

-- 步骤 2：删除旧表（注意：会丢失所有 Session 数据）
DROP TABLE SPRING_SESSION_ATTRIBUTES;
DROP TABLE SPRING_SESSION;

-- 步骤 3：创建新表
CREATE TABLE SPRING_SESSION (
    PRIMARY_ID CHAR(36) NOT NULL,
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    EXPIRY_TIME BIGINT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
    SESSION_PRIMARY_ID CHAR(36) NOT NULL,
    ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
    ATTRIBUTE_BYTES BLOB NOT NULL,
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
    CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID)
        REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```

**注意**：生产环境建议在低峰期执行，并提前通知用户重新登录。

##### 4. 配置变化

```yaml
# 2.5.x 配置
spring:
  session:
    store-type: jdbc
    jdbc:
      initialize-schema: always
      table-name: SPRING_SESSION

# 3.5.x 配置（相同，无需修改）
spring:
  session:
    store-type: jdbc
    jdbc:
      initialize-schema: always
      table-name: SPRING_SESSION
```

##### 5. Schema 路径变化（3.x）

```yaml
# 2.5.x Schema 路径
spring.session.jdbc.schema=classpath:org/springframework/session/jdbc/schema-@@platform@@.sql

# 3.5.x Schema 路径（相同）
spring.session.jdbc.schema=classpath:org/springframework/session/jdbc/schema-@@platform@@.sql
```

---

### 迁移注意事项

#### 数据兼容性

| 存储类型 | 1.5.x → 2.5.x | 2.5.x → 3.5.x |
|----------|----------------|----------------|
| Redis | ✅ 数据兼容 | ✅ 数据兼容 |
| JDBC | ⚠️ DDL 不兼容，需迁移 | ✅ DDL 相同 |

#### 序列化兼容性

```java
// Redis 序列化格式在版本间可能变化
// 建议：升级后清空旧 Session 数据，或使用 JSON 序列化

@Configuration
public class SessionConfig {
    @Bean
    public RedisSerializer<Object> springSessionRedisSerializer() {
        // 使用 JSON 序列化，兼容性更好
        return new GenericJackson2JsonRedisSerializer();
    }
}
```

#### 降级方案

如果升级后出现问题，可快速回退：

```yaml
# 临时允许旧版本 Session 格式（Redis）
spring:
  session:
    redis:
      flush-mode: on-save
```

---

## 最佳实践总结

### 1. 选择合适的存储类型

```yaml
# 分布式环境推荐 Redis
spring:
  session:
    store-type: redis
```

### 2. 会话超时设置

```yaml
spring:
  session:
    timeout: 30m  # 生产环境建议 30 分钟
```

### 3. Redis 高可用配置

```yaml
spring:
  data:
    redis:
      sentinel:
        master: mymaster
        nodes:
          - sentinel1:26379
          - sentinel2:26379
          - sentinel3:26379
```

### 4. 序列化优化

```java
@Configuration
public class SessionConfig {
    
    @Bean
    public RedisSerializer<Object> springSessionRedisSerializer() {
        return new GenericJackson2JsonRedisSerializer();
    }
}
```

### 5. 监控与告警

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,sessions
  endpoint:
    sessions:
      enabled: true
```

---

## 参考资料

- [Spring Session 官方文档](https://docs.spring.io/spring-session/reference/)
- [Spring Boot Spring Session 自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.spring-session)
- [Spring Session GitHub 仓库](https://github.com/spring-projects/spring-session)

---

*最后更新时间：2026-06-09*
