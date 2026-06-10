---
layout: default
title: SB3-Logging
parent: SpringBoot3
has_children: false
nav_order: 1003
---

Here are the logging system changes and practices on SpringBoot3 .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }



## 目录

1. [Spring Boot 3 日志体系核心变更概览](#spring-boot-3-日志体系核心变更概览)
2. [SLF4J 2.0 迁移关键变化](#slf4j-20-迁移关键变化)
3. [Logback 1.4+ 完整配置（默认日志实现）](#logback-14-完整配置默认日志实现)
4. [Log4j2 2.19+ 迁移与配置](#log4j2-219-迁移与配置)
5. [Spring Boot 3 原生配置属性与结构化日志](#spring-boot-3-原生配置属性与结构化日志)
6. [多环境/Profile/条件化日志配置实战](#多环境profile条件化日志配置实战)
7. [GraalVM Native Image 日志适配](#graalvm-native-image-日志适配)
8. [Spring Boot 2.x → 3.x 日志迁移清单](#spring-boot-2x--3x-日志迁移清单)

---

# Spring Boot 3 日志体系核心变更概览

## Spring Boot 3 日志体系架构

Spring Boot 3 是基于 **Jakarta EE 9+** 和 **Java 17+** 构建的第一个大版本。日志体系基座升级如下：

| 组件 | Spring Boot 2.x | Spring Boot 3.x | 关键变更 |
|------|----------------|----------------|----------|
| Java Baseline | Java 8 / 11 | Java 17+ | 模块化、密封类、record |
| SLF4J | 1.7.x | 2.0.x | 新增 fluent API、SPI 重构 |
| Logback | 1.2.x | 1.4.x / 1.5.x | Jakarta EE 9+ 兼容、性能优化 |
| Log4j2 | 2.17.x | 2.20+ | 多线程改进、Garbage-Free 增强 |
| Servlet | javax.servlet | jakarta.servlet | 包名整体迁移 |
| Observability | 无内置 | Actuator + Micrometer Tracing | 结构化日志、分布式追踪 |

## 版本对应关系速查表

```xml
<!-- Spring Boot 3.x 默认引入的日志依赖链 -->
spring-boot-starter / spring-boot-starter-web
  └── spring-boot-starter-logging
       ├── logback-classic 1.4.x     (SLF4J 2.0 实现)
       ├── logback-core 1.4.x
       ├── slf4j-api 2.0.x           (日志门面)
       └── log4j-to-slf4j            (Log4j → SLF4J 桥接)
```

### Maven 依赖版本速查

| Spring Boot 3.x 版本 | SLF4J | Logback | Log4j2 |
|---------------------|-------|---------|--------|
| 3.0.x | 2.0.6+ | 1.4.8+ | 2.19.0+ |
| 3.1.x | 2.0.9+ | 1.4.11+ | 2.20.0+ |
| 3.2.x | 2.0.11+ | 1.4.14+ | 2.21.1+ |
| 3.3.x | 2.0.12+ | 1.5.6+ | 2.23.0+ |

## 从 Spring Boot 2.x 升级后必须注意的突破性变更

```
1. javax.* → jakarta.* 包名变更
   影响：Logback 配置文件若引用了例如 ch.qos.logback.classic 下的
   javax.servlet 相关类，需要更新为 jakarta.servlet。

2. SLF4J 2.0 服务提供者接口（SPI）重构
   影响：自定义 LoggingSystem 实现需要适配新 SPI。

3. 移除了 Spring Boot 2.x 部分已废弃的日志属性
   影响：logging.pattern.rolling-file-clean-history-on-start
   等属性已移除，需使用新版配置。

4. Logback 1.4+ 默认启用 ReconfigureOnChange
   影响：生产环境下自动扫描间隔从 60 秒改为了默认启用，
   可通过 <configuration scan="false"> 关闭。
```

---

# SLF4J 2.0 迁移关键变化

## Fluent Logging API（SLF4J 2.0 核心亮点）

SLF4J 2.0 引入了一套全新的 **fluent（流式）API**，支持延迟求值和链式调用：

```java
// SLF4J 1.7 传统写法
logger.debug("Processing order: {} for user: {}", orderId, userId);

// SLF4J 2.0 Fluent API
logger.atDebug()
      .setMessage("Processing order: {} for user: {}")
      .addArgument(orderId)
      .addArgument(userId)
      .log();

// 延迟求值（Lambda）
logger.atDebug()
      .setMessage("Processing complex data: {}")
      .addArgument(() -> expensiveOperation())  // 只有DEBUG级别才执行
      .log();

// 结构化键值对
logger.atInfo()
      .setMessage("User login")
      .addKeyValue("userId", userId)
      .addKeyValue("ip", request.getRemoteAddr())
      .addKeyValue("timestamp", System.currentTimeMillis())
      .log();
```

## LoggingEvent 构建器

```java
// 带异常的流式日志
logger.atError()
      .setMessage("Failed to process payment for order: {}")
      .addArgument(orderId)
      .setCause(exception)
      .log();

// 动态日志级别
logger.atLevel(Level.WARN)
      .setMessage("Disk usage: {}%")
      .addArgument(diskUsage)
      .log();
```

## Marker 增强

```java
// SLF4J 2.0 Marker 支持更丰富的操作
Marker audit = MarkerFactory.getMarker("AUDIT");

logger.atInfo()
      .addMarker(audit)
      .setMessage("Sensitive data access: {}")
      .addArgument(dataId)
      .log();
```

## SPI 重构对框架集成的影响

SLF4J 2.0 将 `SLF4JServiceProvider` 重构为基于 `java.util.ServiceLoader` 的 SPI 机制：

```java
// SLF4J 2.0 自定义 LoggingSystem 实现（适配 Spring Boot 3）
public class CustomLoggingSystem extends LogbackLoggingSystem {

    @Override
    protected void loadDefaults(LogbackConfigurator configurator,
                                org.springframework.core.env.Environment environment) {
        // SLF4J 2.0 要求显式调用 SLF4JServiceProvider
        if (!isSlf4j24OrLater()) {
            SLF4JServiceProvider provider = LoggerFactory.getILoggerFactory();
            // 适配逻辑...
        }
        super.loadDefaults(configurator, environment);
    }
}
```

## 桥接包更新（JCL → SLF4J 2.0）

```xml
<!-- Spring Boot 3 不再强制排除 commons-logging，但依然推荐统一门面 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>  <!-- 已升级到 2.0.x -->
</dependency>
```

---

# Logback 1.4+ 完整配置（默认日志实现）

## 基础配置示例

在 `src/main/resources/logback-spring.xml` 中：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="30 seconds">

    <!-- Spring Boot 3 推荐使用 springProperty 加载 boot 配置 -->
    <springProperty scope="context" name="APP_NAME"
                    source="spring.application.name" defaultValue="unknown"/>
    <springProperty scope="context" name="LOG_HOME"
                    source="logging.file.path" defaultValue="./logs"/>

    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 文件滚动输出 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${APP_NAME}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- JSON 格式化输出（Logback 1.4+ 内置支持） -->
    <appender name="JSON_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${APP_NAME}.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}.%d{yyyy-MM-dd}.%i.json</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.JsonEncoder"/>
    </appender>

    <!-- Async 异步输出（Logback 1.4+ 性能提升） -->
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="FILE"/>
        <queueSize>1024</queueSize>
        <neverBlock>true</neverBlock>
        <discardingThreshold>0</discardingThreshold>
        <maxFlushTime>5000</maxFlushTime>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_FILE"/>
    </root>

</configuration>
```

## Logback 1.4+ 新增特性配置

### JsonEncoder（测试时可用，生产推荐 logstash-encoder）

```xml
<!-- logback-spring.xml -->
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="ch.qos.logback.classic.encoder.JsonEncoder">
        <!-- 可自定义字段映射 -->
        <timestamp>timestamp</timestamp>
        <version>version</version>
        <logger>loggerName</logger>
        <thread>threadName</thread>
        <level>level</level>
        <message>message</message>
    </encoder>
</appender>
```

### Logback 1.5+ 新增：条件化配置基于 Java 版本

```xml
<!-- 自动适配不同 Java 版本的日志格式 -->
<if condition='property("java.version").startsWith("17")'>
    <then>
        <appender name="JFR" class="ch.qos.logback.classic.jfr.JFRAppender"/>
    </then>
</if>
```

## 性能优化配置（Garbage-Free Logging）

```xml
<configuration>
    <!-- Logback 1.4+ Garbage-Free 模式 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <immediateFlush>false</immediateFlush>  <!-- Logback 1.4+ 新增 -->
        </encoder>
    </appender>

    <!-- 异步 Appender 调优 -->
    <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="FILE"/>
        <queueSize>2048</queueSize>
        <neverBlock>true</neverBlock>
        <discardingThreshold>0</discardingThreshold>
        <!-- Logback 1.4+ 新增：指定阻塞策略 -->
        <maxFlushTime>3000</maxFlushTime>
        <includeCallerData>false</includeCallerData>
    </appender>
</configuration>
```

## application.yml 配置中的日志属性（推荐简配方案）

不编写 `logback-spring.xml`，完全通过 application.yml 控制：

```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
    org.springframework.web: WARN
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: ${LOG_HOME:-./logs}/${spring.application.name:-app}.log
    max-size: 100MB
    max-history: 30
    total-size-cap: 3GB
  logback:
    rolling:
      file-name-pattern: ${LOG_HOME:-./logs}/${spring.application.name:-app}.%d{yyyy-MM-dd}.%i.log
  # Spring Boot 3 新增：结构化日志
  structured:
    format: logstash  # logstash | ecs | gelf
```

---

# Log4j2 2.19+ 迁移与配置

## 切换至 Log4j2

Spring Boot 3 不再内置 Log4j2 桥接，需要主动排除默认 Logback：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- 排除默认的 Logback -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 引入 Log4j2 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

<!-- Log4j2 2.20+ 异步日志依赖（可选，显著提升性能） -->
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.4</version>
</dependency>
```

## Log4j2 XML 完整配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- log4j2-spring.xml - Spring Boot 3 推荐命名 -->
<Configuration status="WARN"
               monitorInterval="30"
               name="SpringBoot3Log4j2">
    <!-- Spring Boot 属性绑定 -->
    <SpringProperty name="APP_NAME" source="spring.application.name" defaultValue="app"/>
    <SpringProperty name="LOG_HOME" source="logging.file.path" defaultValue="./logs"/>

    <!-- 控制台输出 -->
    <Appenders>
        <Console name="CONSOLE" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %c{1.} - %msg%n"/>
            <ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
        </Console>

        <!-- 滚动文件 -->
        <RollingFile name="FILE"
                     fileName="${LOG_HOME}/${APP_NAME}.log"
                     filePattern="${LOG_HOME}/${APP_NAME}.%d{yyyy-MM-dd}.%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %c{1.} - %msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>

        <!-- Log4j2 2.19+ JSON 格式化（Garbage-Free） -->
        <RollingFile name="JSON_FILE"
                     fileName="${LOG_HOME}/${APP_NAME}.json"
                     filePattern="${LOG_HOME}/${APP_NAME}.%d{yyyy-MM-dd}.%i.json.gz">
            <JsonLayout complete="false" compact="true"
                        properties="true"
                        locationInfo="false">
                <KeyValuePair key="service" value="${APP_NAME}"/>
                <KeyValuePair key="environment" value="${spring:profiles.active:-default}"/>
            </JsonLayout>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>

        <!-- Log4j2 2.20+ Routing Appender（按日志级别分流） -->
        <Routing name="ROUTING">
            <Routes pattern="$${level:-INFO}">
                <Route ref="CONSOLE" key="DEBUG"/>
                <Route ref="FILE" key="INFO"/>
                <Route ref="FILE" key="WARN"/>
                <Route ref="ERROR_FILE" key="ERROR">
                    <RollingFile name="ERROR_FILE"
                                 fileName="${LOG_HOME}/${APP_NAME}.error.log"
                                 filePattern="${LOG_HOME}/${APP_NAME}.error.%d{yyyy-MM-dd}.%i.log.gz">
                        <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %c{1.} - %msg%n"/>
                        <Policies>
                            <SizeBasedTriggeringPolicy size="50 MB"/>
                        </Policies>
                    </RollingFile>
                </Route>
            </Routes>
        </Routing>

        <!-- 异步 Appender（依赖 Disruptor） -->
        <Async name="ASYNC_FILE" bufferSize="1024" blocking="false">
            <AppenderRef ref="FILE"/>
        </Async>
    </Appenders>

    <Loggers>
        <!-- Spring Boot 3 默认 Logger -->
        <Logger name="org.springframework" level="INFO"/>
        <Logger name="org.springframework.boot" level="INFO"/>

        <!-- Hibernate SQL 日志 -->
        <Logger name="org.hibernate.SQL" level="DEBUG" additivity="false">
            <AppenderRef ref="CONSOLE"/>
        </Logger>
        <Logger name="org.hibernate.type.descriptor.sql.BasicBinder"
                level="TRACE" additivity="false">
            <AppenderRef ref="CONSOLE"/>
        </Logger>

        <!-- 业务包 -->
        <Logger name="com.example" level="DEBUG"/>

        <!-- Root 配置 -->
        <Root level="INFO">
            <AppenderRef ref="CONSOLE"/>
            <AppenderRef ref="ASYNC_FILE"/>
        </Root>
    </Loggers>
</Configuration>
```

## Log4j2 YAML 配置（Log4j2 2.19+ 支持）

```yaml
# log4j2-spring.yml
Configuration:
  status: WARN
  monitorInterval: 30

  Properties:
    Property:
      - name: APP_NAME
        value: "${spring:spring.application.name:-app}"
      - name: LOG_HOME
        value: "${sys:LOG_HOME:-./logs}"

  Appenders:
    Console:
      name: CONSOLE
      target: SYSTEM_OUT
      PatternLayout:
        pattern: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %c{1.} - %msg%n"

    RollingFile:
      - name: FILE
        fileName: "${LOG_HOME}/${APP_NAME}.log"
        filePattern: "${LOG_HOME}/${APP_NAME}.%d{yyyy-MM-dd}.%i.log.gz"
        PatternLayout:
          pattern: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %c{1.} - %msg%n"
        Policies:
          TimeBasedTriggeringPolicy:
            interval: 1
            modulate: true
          SizeBasedTriggeringPolicy:
            size: 100MB
        DefaultRolloverStrategy:
          max: 30

  Loggers:
    Logger:
      - name: org.springframework
        level: INFO
      - name: com.example
        level: DEBUG
    Root:
      level: INFO
      AppenderRef:
        - ref: CONSOLE
        - ref: FILE
```

## Log4j2 2.19+ Garbage-Free 配置

```yaml
# application.yml
logging:
  level:
    root: INFO
  log4j2:
    config:
      # Log4j2 2.19+ Garbage-Free 模式
      garbage-free:
        enabled: true
        # 线程上下文支持
        thread-context-map: false
        # 字符串构建器复用
        encoder:
          text-encoder: org.apache.logging.log4j.core.layout.TextEncoderHelper
```

---

# Spring Boot 3 原生配置属性与结构化日志

## 完整的 application.yml 日志配置

```yaml
spring:
  application:
    name: my-service

logging:
  # 日志级别配置
  level:
    root: INFO
    com.example: DEBUG
    # 可动态调整
    org.springframework.web.servlet: WARN
    org.hibernate.SQL: DEBUG
    org.hibernate.orm.jdbc.bind: TRACE

  # 日志格式
  pattern:
    console: "%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd'T'HH:mm:ss.SSSXXX}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"
    file: "%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd'T'HH:mm:ss.SSSXXX}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"
    dateformat: yyyy-MM-dd'T'HH:mm:ss.SSSXXX
    level: "%5p"

  # 日志文件配置（Spring Boot 3 新增 name 统一管理）
  file:
    name: ${spring.application.name:-app}.log
    path: ./logs
    max-size: 100MB
    max-history: 30
    total-size-cap: 3GB
    # Spring Boot 3 新增：文件权限
    append: true

  # 日志分组（Spring Boot 2.3+ / 3）
  group:
    web: org.springframework.web, org.springframework.web.servlet
    sql: org.hibernate.SQL, org.springframework.jdbc
    business: com.example.service, com.example.controller

  # 结构化日志（Spring Boot 3.2+ 新增）
  structured:
    # logstash / ecs / gelf 三种格式
    format: logstash
    # 自定义字段
    logstash:
      custom-fields:
        service: ${spring.application.name}
        environment: ${spring.profiles.active:default}

  # 日志系统属性
  logback:
    rolling:
      file-name-pattern: ${logging.file.path:-./logs}/${spring.application.name:-app}.%d{yyyy-MM-dd}.%i.log
      clean-history-on-start: false

  # 异常输出
  exception-conversion-word: "%wEx"
  register-shutdown-hook: true
```

## 结构化日志（Spring Boot 3.2+ 核心特性）

Spring Boot 3.2 正式引入了内置的结构化日志支持，无需第三方依赖即可输出 JSON 格式日志。

### 三种内置格式

```yaml
# ECS (Elastic Common Schema) - 推荐 ELK 场景
logging:
  structured:
    format: ecs

# Logstash 格式 - 兼容现有 Logstash 管道
logging:
  structured:
    format: logstash

# GELF (Graylog Extended Log Format)
logging:
  structured:
    format: gelf
```

### ECS 格式输出示例

```json
{
  "@timestamp": "2026-06-10T10:30:00.123+08:00",
  "log.level": "INFO",
  "message": "Application started",
  "service.name": "my-service",
  "process.thread.name": "main",
  "log.logger": "com.example.Application",
  "service.environment": "production"
}
```

### Logstash 格式自定义字段

```yaml
logging:
  structured:
    format: logstash
    logstash:
      custom-fields:
        service: my-service
        region: ap-east-1
        team: backend
```

### 结构化日志与 Micrometer Tracing 集成

Spring Boot 3 的 **Micrometer Tracing** 会自动将 TraceId / SpanId 注入结构化日志：

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 1.0
    baggage:
      enabled: true

logging:
  structured:
    format: ecs  # trace.id / span.id 自动写入 JSON
```

输出示例：
```json
{
  "@timestamp": "2026-06-10T10:30:01.456+08:00",
  "log.level": "INFO",
  "message": "Payment processed",
  "trace.id": "a1b2c3d4e5f6g7h8",
  "span.id": "i9j0k1l2m3n4o5p6",
  "transaction.id": "abc123",
  "service.name": "payment-service"
}
```

## 配置属性完整速查表

| 属性 | 作用域 | 默认值 | 说明 |
|------|--------|--------|------|
| `logging.level.*` | ALL | INFO | 日志级别，支持包/类粒度 |
| `logging.group.*` | ALL | — | 日志级别分组，方便批量配置 |
| `logging.file.name` | ALL | — | 日志文件名（3.x 推荐，替代 file） |
| `logging.file.path` | ALL | — | 日志目录路径 |
| `logging.file.max-size` | ALL | 10MB | 单个文件上限 |
| `logging.file.max-history` | ALL | 7 | 保留天数 |
| `logging.file.total-size-cap` | ALL | 0 (不限) | 总日志大小上限 |
| `logging.pattern.console` | ALL | — | 控制台输出格式 |
| `logging.pattern.file` | ALL | — | 文件输出格式 |
| `logging.pattern.dateformat` | ALL | yyyy-MM-dd HH:mm:ss.SSS | 日期格式 |
| `logging.pattern.level` | ALL | %5p | 级别格式 |
| `logging.logback.rolling.*` | Logback | — | Logback 滚动策略 |
| `logging.structured.format` | 3.2+ | — | 结构化格式(ecs/logstash/gelf) |
| `logging.structured.*` | 3.2+ | — | 结构化日志自定义配置 |
| `logging.threshold.console` | 3.2+ | TRACE | 控制台最低输出级别 |
| `logging.exception-conversion-word` | ALL | %wEx | 异常转换 |
| `logging.register-shutdown-hook` | ALL | true | 关闭时清理日志资源 |

---

# 多环境/Profile/条件化日志配置实战

## 使用 logback-spring.xml 中的 `<springProfile>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <springProperty scope="context" name="APP_NAME"
                    source="spring.application.name" defaultValue="app"/>

    <!-- 开发环境：控制台详细输出 + 颜色 -->
    <springProfile name="dev, local">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %green([%thread]) %highlight(%-5level) %cyan(%logger{36}) - %msg%n</pattern>
            </encoder>
        </appender>

        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <!-- 生产环境：异步 + JSON + 性能模式 -->
    <springProfile name="prod, staging">
        <!-- 先输出到文件，再异步 -->
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${LOG_HOME:-/var/log}/${APP_NAME}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <fileNamePattern>${LOG_HOME:-/var/log}/${APP_NAME}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
                <maxFileSize>200MB</maxFileSize>
                <maxHistory>90</maxHistory>
                <totalSizeCap>10GB</totalSizeCap>
            </rollingPolicy>
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>

        <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
            <appender-ref ref="FILE"/>
            <queueSize>2048</queueSize>
            <neverBlock>true</neverBlock>
        </appender>

        <root level="INFO">
            <appender-ref ref="ASYNC_FILE"/>
        </root>
    </springProfile>

    <!-- 测试环境：只输出到文件，减少 IO 干扰 -->
    <springProfile name="test">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>./target/test-logs/${APP_NAME}.log</file>
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>

        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>

</configuration>
```

## 使用 application-{profile}.yml 多文件配置

### application-dev.yml
```yaml
logging:
  level:
    root: DEBUG
    org.springframework: INFO
    org.springframework.web: DEBUG
  pattern:
    console: "%clr(%d{HH:mm:ss.SSS}){faint} %clr(%-5level) %clr(%logger{36}){cyan} %msg%n"
  file:
    path: ./logs/dev
```

### application-prod.yml
```yaml
logging:
  level:
    root: INFO
    org.springframework: WARN
  file:
    name: /var/log/app/my-service.log
    max-size: 200MB
    max-history: 90
    total-size-cap: 10GB
  pattern:
    file: "%d{yyyy-MM-dd'T'HH:mm:ss.SSSZ} [%thread] %-5level %logger{36} - %msg%n"
  structured:
    format: logstash
    logstash:
      custom-fields:
        service: my-service
        environment: production
```

## 运行时动态调整日志级别（Actuator）

```yaml
# 启用日志级别端点
management:
  endpoints:
    web:
      exposure:
        include: loggers
```

```bash
# 查看当前日志级别
curl http://localhost:8080/actuator/loggers

# 查看指定包
curl http://localhost:8080/actuator/loggers/com.example

# 动态调整（无需重启）
curl -X POST http://localhost:8080/actuator/loggers/com.example.service \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'

# 重置为默认级别
curl -X POST http://localhost:8080/actuator/loggers/com.example.service \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": null}'
```

---

# GraalVM Native Image 日志适配

## 已知限制与解决方案

GraalVM Native Image 在编译时会进行 **静态分析**，日志框架的反射调用需要通过配置文件提前注册。

### 自动配置（推荐）

Spring Boot 3.x + GraalVM Native 时，使用 `org.springframework.aot.hint` 自动生成反射配置：

```java
import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;

public class LoggingRuntimeHints implements RuntimeHintsRegistrar {

    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        // Logback 反射需要
        hints.reflection()
              .registerType(ch.qos.logback.classic.Logger.class,
                            hint -> hint.withMembers())
              .registerType(ch.qos.logback.core.ConsoleAppender.class,
                            hint -> hint.withMembers())
              .registerType(ch.qos.logback.core.rolling.RollingFileAppender.class,
                            hint -> hint.withMembers());
    }
}
```

### resource-config.json 手动注册

```json
{
  "resources": [
    { "pattern": "\\Qlogback-spring.xml\\E" },
    { "pattern": "\\Qlogback.xml\\E" },
    { "pattern": "\\Qlog4j2-spring.xml\\E" },
    { "pattern": "\\Qlog4j2.xml\\E" }
  ]
}
```

### Logback 在 Native Image 下的最小配置

```xml
<!-- logback-spring.xml - Native Image 兼容版本 -->
<configuration>
    <!-- 避免使用 groovy 配置 -->
    <!-- 避免使用复杂的条件化 if/then -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

---

# Spring Boot 2.x → 3.x 日志迁移清单

## 渐进式迁移对照表

| 检查项 | Spring Boot 2.x | Spring Boot 3.x | 迁移操作 |
|--------|----------------|----------------|----------|
| Java 版本 | 8 / 11 | 17+ | 升级 JDK，利用 record/密封类简化代码 |
| javax.servlet | javax.* | jakarta.* | 全局替换包名，Logback 配置中若有引用需更新 |
| SLF4J API | 1.7 | 2.0 | 兼容（无需改代码），可选升级到 Fluent API |
| Logback | 1.2.x | 1.4.x+ | 配置基本兼容，测试 `scan` 行为（1.4 默认启用） |
| Log4j2 | 2.17.x | 2.20+ | 配置兼容，推荐添加 `Disruptor` 开启异步 |
| `logging.file` | ✅ 支持 | ❌ 废弃 | 替换为 `logging.file.name` |
| `logging.path` | ✅ 支持 | ❌ 废弃 | 替换为 `logging.file.path` |
| `logging.pattern.rolling-file-clean-history-on-start` | ✅ 支持 | ❌ 移除 | 移除该配置，Logback 配置 `cleanHistoryOnStart` |
| 结构化日志 logging.structured | ❌ | ✅ 3.2+ | 新增功能，可选使用 |
| Micrometer Tracing 集成日志 | ❌ | ✅ | 启用 management.tracing 即可 |
| Actuator loggers 端点 | ✅ | ✅ | 兼容，路径不变 |

## 升级检查清单

```bash
# 1. 检查 javax 引用
grep -r "javax\." src/ --include="*.java" --include="*.xml" --include="*.properties"

# 2. 检查废弃的日志属性
grep -r "logging\.file$" src/main/resources/application*.yml
grep -r "logging\.path$" src/main/resources/application*.yml

# 3. 检查 Logback 配置中是否引用了 javax.servlet
grep -r "javax" src/main/resources/logback*.xml

# 4. 确认日志依赖树
mvn dependency:tree -Dincludes=*logging*
mvn dependency:tree -Dincludes=*slf4j*
mvn dependency:tree -Dincludes=*logback*
```

## 常见迁移问题

### 问题 1：SLF4J 2.0 的 ClassCastException

```bash
# 错误现象
java.lang.ClassCastException: class ch.qos.logback.classic.Logger
  cannot be cast to class org.slf4j.Logger
```

**原因**：classpath 中存在多个版本的 `slf4j-api.jar`。

**排查**：
```bash
# 查看冲突版本
mvn dependency:tree -Dincludes=org.slf4j:slf4j-api
```

**解决**：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

### 问题 2：Logback 1.4+ ReconfigureOnChange 导致频繁 I/O

```xml
<!-- 关闭自动热重载，生产环境推荐关闭 -->
<configuration scan="false">
    <!-- ... -->
</configuration>
<!-- 或使用长间隔 -->
<configuration scan="true" scanPeriod="300 seconds">
    <!-- ... -->
</configuration>
```

### 问题 3：Log4j2 AsyncLogger 在 Spring Boot 3 中死锁

**解决方案**：
```xml
<!-- log4j2-spring.xml 中关闭 asyncLogger -->
<Configuration status="WARN" monitorInterval="30">
    <AsyncLogger name="com.example" level="DEBUG" includeLocation="false">
        <AppenderRef ref="FILE"/>
    </AsyncLogger>
    <!-- 其他配置 -->
</Configuration>
```

---

## 参考资料

- [Spring Boot 3.0 Release Notes - Logging](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes#logging)
- [Spring Boot 3.2 Structured Logging](https://spring.io/blog/2023/11/09/structured-logging-in-spring-boot-3-2)
- [SLF4J 2.0 Manual](https://www.slf4j.org/manual.html)
- [Logback 1.4.x Documentation](https://logback.qos.ch/manual/)
- [Log4j 2.x Garbage-Free](https://logging.apache.org/log4j/2.x/manual/garbagefree.html)
- [GraalVM Native Image Logging](https://www.graalvm.org/latest/reference-manual/native-image/logging/)

---

*最后更新时间：2026-06-10*
