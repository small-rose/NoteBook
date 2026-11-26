---
layout: default
title: SpringBoot upgrate
parent: SpringBoot
nav_order: 601
---

Here are SpringBoot upgrade experience .
{: .fs-6 .fw-300 }


## Here are SpringBoot upgrade experience .
{: .no_toc .text-delta }

1. TOC
{:toc}

# Spring Boot 2.x → 3.5.x → 4.x 迁移对照手册


> 生成时间：2026-11-26

---

## 一. 主要基线版本

| 维度 | Spring Boot 2.7.x | Spring Boot 3.5.x | Spring Boot 4.0.x                       |
|---|---|---|-----------------------------------------|
| Java 基线 | 8 | 17 | 17/21/25（推荐最新 LTS）                      |
| Spring Framework | 5.3.x | 6.2.x | 7.x                                     |
| Jakarta EE | Java EE 8（`javax.*`） | EE 10（`jakarta.*`） | EE 11                                   |
| Servlet | 4.0 | 6.0 | **6.1**                                 |
| 内嵌服务器 | Tomcat 9 / Jetty / Undertow | Tomcat 10.1 / Jetty 12 | Tomcat 11 / Jetty 12.1（**Undertow 移除**） |
| Jackson | 2.x | 2.x | **3.x（默认，保留 2.x 弃用支持）**                 |
| Kotlin（若使用） | — | — | **≥ 2.2**                               |
| GraalVM native-image（若使用） | — | — | **≥ 25**                                |

来源：
- Spring Boot 4.0 官方发布博客：https://spring.io/blog/2025/11/20/spring-boot-4-0-0-available-now/
- Spring Framework 7.0 GA 官方博客：https://spring.io/blog/2025/11/13/spring-framework-7-0-general-availability
- Spring Boot 4.0 官方迁移指南：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide

**一句话定性**：
- **2.x → 3.5.x = 换底座**（Java 17 + `javax→jakarta` + Spring Framework 6 + Hibernate 6）
- **3.5.x → 4.x = 换结构**（模块化拆分 + Jackson 3 + Spring Framework 7 + Jakarta EE 11 + Servlet 6.1）

---

## 二. Spring Boot 3.5.x 相比 2.x 的重要变化

### 2.1 硬门槛与破坏性变更

> Spring Boot 3.0 迁移指南官方口径

- **Java 17 起**：Spring Boot 3.0 起最低 JDK 17
- **`javax.*` → `jakarta.*`**：Servlet / JPA / Validation / Annotation 全部包名替换
- **Micrometer 指标导出属性前缀迁移**：`management.metrics.export.*` → `management.<tool>.metrics.export.*`（例：`management.metrics.export.prometheus` → `management.prometheus.metrics.export`）
- **Spring Data 属性前缀收拢**：
    - Cassandra：`spring.data.cassandra.*` → `spring.cassandra.*`
    - Redis：`spring.redis.*` → `spring.data.redis.*`（Redis 自动配置现在要求 classpath 有 Spring Data）
- **Flyway 默认升到 9.0**，`FlywayConfigurationCustomizer` 调用时机调整（Callback / JavaMigration bean 先注册再 customizer）
- **Liquibase 默认 4.17.x**，官方提示部分用户遇到问题可自行覆写版本
- **Hibernate 6.1** 默认，JPQL / 方言 / 自定义 SQL 函数需要回归
- **MongoDB 健康检查**：`buildInfo` 换成 `isMaster`，返回值从 `version` 变 `maxWireVersion`

官方源：Spring Boot 3.0 Migration Guide（GitHub Wiki 官方）
https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide

> ⚠️ 迁移前必做：`mvn dependency:tree | grep javax` 扫干净所有传递依赖里残留的 `javax.*`。

### 2.2 Spring Boot 3.5 这一代自身的新增与收紧

**新增能力**（Spring Boot 3.5.0 官方发布博客 + 3.5 Release Notes 官方）：
- `@ServletRegistration` / `@FilterRegistration` 注解，替代 `ServletRegistrationBean` / `FilterRegistrationBean`
- `spring.config.import=env:MY_VAR` 从单个环境变量加载多行属性（properties / yaml 均可）
- 结构化日志堆栈可配：`logging.structured.json.stacktrace.*`
- WebClient 全局属性化配置（超时、重定向）+ 新增 `ClientHttpConnectorBuilder` 接口
- Service Connections 客户端 SSL 支持：Cassandra / Couchbase / Elasticsearch / Kafka / MongoDB / RabbitMQ / Redis
- OpenTelemetry 原生识别 `OTEL_RESOURCE_ATTRIBUTES` / `OTEL_SERVICE_NAME`；`service.namespace` 从 `spring.application.group` 读取；`service.group` 弃用
- `TaskDecorator` bean 自动应用到 `taskScheduler`；`spring.task.execution.mode=force` 可在已有 `Executor` bean 时仍自动配 `AsyncTaskExecutor`
- Spring Batch：`JobParametersConverter` 可自定义；新增 `spring.batch.jdbc.validate-transaction-state`
- Apache Pulsar 客户端默认版本 3.3.x → **4.0.x LTS**

官方源：
- Spring Boot 3.5.0 官方发布博客：https://spring.io/blog/2025/05/22/spring-boot-3-5-0-available-now
- Spring Boot 3.5 Release Notes（GitHub Wiki 官方）：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes

**破坏性收紧**（3.5 相对 3.4，官方 Release Notes）：
- 3.3 中标记要在 3.5 移除的 `@Deprecated` 类/方法/属性**已全部删除**
- `FilterRegistrationBean` 对空 `DispatcherType` 集合的行为调整为等同 `setDispatcherTypes(null)`

官方源：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes

### 2.3 Spring Cloud 在 3.x 线的对齐

Spring Cloud 官方历史兼容表（Release Train ↔ Spring Boot）：

| Spring Cloud 列车 | 兼容 Spring Boot |
|---|---|
| **2025.0（Northfields）** | **3.5.x** |
| 2024.0（Moorgate） | 3.4.x |
| 2023.0（Leyton） | 3.3.x / 3.2.x |
| 2022.0（Kilburn） | 3.1.x / 3.0.x |
| 2021.0（Jubilee） | 2.7.x / 2.6.x |
| 2020.0（Ilford） | 2.5.x / 2.4.x |
| Hoxton | 2.3.x / 2.2.x |

子项目版本（2025.0 Northfields 线上）：commons 4.3.x、config 4.3.x、gateway 4.3.x、openfeign 4.3.x、circuitbreaker 3.3.x、stream 4.3.x、bus 4.3.x、kubernetes 3.3.x、task 3.3.x、vault 4.3.x、zookeeper 4.3.x、function 4.3.x。

- **Spring Cloud Sleuth 已下线**：Jubilee（3.1.x）之后不再随列车维护，链路追踪走 **Micrometer Tracing**

官方源：Spring Cloud 官方项目页与历史版本表：https://spring.io/spring-cloud

### 2.4 流程引擎在 3.x 线

**Flowable**（官方文档明确对照表）：

| Flowable | Spring Boot |
|---|---|
| 3.14.x | 2.7.x |
| 3.16.x | 3.3.x |
| 3.17.x | **3.5.x** |
| 2025.1.x | **3.5.x** |
| 2025.2.x | 4.0.x |

官方源：Flowable 官方文档 Spring Boot 版本对照：https://documentation.flowable.com/latest/admin/release-notes/security-vulnerabilities

**Camunda 7**（官方 Spring Boot 集成兼容页）：
- 7.13.x 起 Camunda Platform 与 Spring Boot Starter 共享版本号
- 2.3.x 线支持 Spring Boot 2.2.x / 2.3.x.RELEASE

官方源：https://docs.camunda.org/manual/7.13/user-guide/spring-boot-integration/version-compatibility

> ⚠️ **Camunda 8 / Spring Zeebe SDK 对 Spring Boot 3.5 的官方兼容原始页本轮未取到**，需以 Camunda Docs 站 "Spring Zeebe" 章节 / Maven Central 制品页为准，本文不补造地址。

### 2.5 常用工具组件在 3.x 线

**MyBatis**（官方 spring-boot-starter 兼容表）：

| mybatis-spring-boot-starter | MyBatis-Spring | Spring Boot | Java |
|---|---|---|---|
| 4.0 | 4.0 | 4.0+ | 17+ |
| **3.0** | **3.0** | **3.2 – 3.5** | **17+** |
| 2.3 | 2.1 | 2.7 | 8+ |
| 2.2（EOL） | 2.0 | 2.5 – 2.7 | 8+ |

官方源：https://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

**MyBatis-Plus**（官方安装页）：
- Spring Boot 2 → `com.baomidou:mybatis-plus-boot-starter`
- Spring Boot 3 → `com.baomidou:mybatis-plus-spring-boot3-starter`
- 当前版本 3.5.18

官方源：https://baomidou.com/en/getting-started/install/

**springdoc-openapi**：⚠️ 未取到官方原始页中针对 Spring Boot 3.5 的明确兼容声明段落（官网首页可访问但正文未被检索抽取），请以 https://springdoc.org 官方文档 Release Notes 为准。

**Lombok / MapStruct / Hutool / Guava / Knife4j**：⚠️ 未取到这些库针对 Spring Boot 3.5 的官方兼容声明原始页，属编译期/注解处理器层面，通常以各库 release notes 为准，本文不做断言。

---

## 三. Spring Boot 4.x 相比 3.x 的重要变化

### 3.1 系统要求抬升（Spring Boot 4.0 迁移指南官方口径）

- **Java 17 起**（鼓励最新 LTS）
- **Kotlin ≥ 2.2**（若使用）
- **GraalVM native-image ≥ 25**（若使用）
- **Jakarta EE 11**、**Servlet 6.1 基线**
- **必须 Spring Framework 7.x**

官方源：Spring Boot 4.0 Migration Guide：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide

Spring Framework 7.0 官方 GA 博客补充基线：
- Java 25 为一等公民，保留 Java 17 基线
- Jakarta EE 11：Servlet 6.1 / JPA 3.2 / Bean Validation 3.1
- **JSpecify 全量空安全**
- **Jackson 3.0 支持（保留 Jackson 2.x 弃用支持）**
- Kotlin 2.2、JUnit 6.0
- 新特性：编程式 Bean 注册、核心弹性（Resilience）、JmsClient、API Versioning、HTTP Interface Client 配置、RestTestClient

官方源：https://spring.io/blog/2025/11/13/spring-framework-7-0-general-availability

### 3.2 模块化重构（本代最大结构变化）

官方迁移指南原文要点：
- Spring Boot 4.0 采用**全新模块化设计**，从「少量大 jar」改为「更小、更聚焦的模块」
- 用 starter POM 的项目大部分依赖自动对齐
- **测试基础设施也已模块化**，需要逐个 review 测试依赖
- **原本没有 starter 的技术现在都有了专属 starter**，且每个 starter 配一个 `-test` 伴生 starter
- 提供 `spring-boot-autoconfigure-classic` 作为过渡模块，接近 Boot 3 的非模块化体验

官方源：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide

官方发布博客补充：
- 完整模块化代码库，提供更小更聚焦的 jar
- 新增 RESTful **API Versioning** 与 **HTTP Service Client** 支持

官方源：https://spring.io/blog/2025/11/20/spring-boot-4-0-0-available-now

**starter 改名 / 拆分**（官方迁移指南点名）：
- `spring-boot-starter-aop` → **`spring-boot-starter-aspectj`**
- `spring-boot-tx` 模块 → **`spring-boot-transaction`**
- 可观测模块改名：`spring-boot-metrics` → `spring-boot-micrometer-metrics`、`spring-boot-observation` → `spring-boot-micrometer-observation`、`spring-boot-tracing` → `spring-boot-micrometer-tracing`；根包 `org.springframework.boot.metrics` → `org.springframework.boot.micrometer`
- 新增 `spring-boot-starter-opentelemetry`（OTLP 导出 metrics + traces，自动配 OTel SDK）

官方源：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide

### 3.3 官方移除项（Spring Boot 4.0 迁移指南逐条列出）

- **Undertow**（starter + 内嵌支持全删，因未兼容 Servlet 6.1）
- **Spring Pulsar Reactive**（自动配置与依赖管理移除）
- **内嵌可执行 Uber Jar 启动脚本**（Unix fully-executable）
- **Spring Session Hazelcast**（转 Hazelcast 团队自维护）
- **Spring Session MongoDB**（转 MongoDB 团队自维护）
- **Spock 集成**（Spock 尚未支持 Groovy 5）
- 3.x 线所有 `@Deprecated` 类/方法/属性**一次性删除**

官方源：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide

### 3.4 配置属性重命名（Spring Boot 4.0 Configuration Changelog 官方）

- `spring.codec.*` → `spring.http.codecs.*`（`log-request-details`、`max-in-memory-size`）
- `spring.dao.exceptiontranslation.enabled` → `spring.persistence.exceptiontranslation.enabled`
- MongoDB 一批：`spring.data.mongodb.*` → `spring.mongodb.*`（`host` / `port` / `database` / `username` / `password` / `uri` / `replica-set-name` / `additional-hosts` / `authentication-database` / `protocol` / `ssl.bundle` → `ssl.bundles` / `ssl.enabled` / `uuid-representation` → `representation.uuid`）
- `spring.session.redis.*` → `spring.session.data.redis.*`（`cleanup-cron` / `configure-action` / `flush-mode` / `namespace` / `repository-type` / `save-mode`）
- `spring.session.hazelcast.*`、`spring.session.mongodb.*` 整体移除
- `spring.flyway.clean-on-validation-error` 移除（Flyway 10.18 弃用、11.0 删除）
- Flyway Oracle/SQLServer 属性改层级：`oracle-kerberos-cache-file` → `oracle.kerberos-cache-file`、`oracle-sqlplus` → `oracle.sqlplus`、`oracle-sqlplus-warn` → `oracle.sqlplus-warn`、`oracle-wallet-location` → `oracle.wallet-location`、`sql-server-kerberos-login-file` → `sqlserver.kerberos-login-file`
- `spring.freemarker.enabled`、`spring.thymeleaf.enabled` 移除（不再需要）
- `spring.test.mockmvc.webclient.enabled` → `spring.test.mockmvc.htmlunit.webclient.enabled`
- `spring.test.mockmvc.webdriver.enabled` → `spring.test.mockmvc.htmlunit.webdriver.enabled`
- `spring.test.observability.auto-configure` → 被 `spring.test.metrics.export` + `spring.test.tracing.export` 取代
- 所有 `server.undertow.*` 属性移除

官方源：Spring Boot 4.0 Configuration Changelog：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Configuration-Changelog

> 迁移期务必挂 `spring-boot-properties-migrator`（runtime scope），启动即报废弃属性，**迁完记得摘掉**。

### 3.5 Jackson 2 → Jackson 3

Spring Framework 7.0 官方 GA 博客原文：**"Support for Jackson 3.0 (with deprecated support for Jackson 2.x retained)"**

官方源：https://spring.io/blog/2025/11/13/spring-framework-7-0-general-availability

> ⚠️ Jackson 3 具体的包名迁移（`com.fasterxml.jackson` → `tools.jackson`）、`JsonMapper` 不可变化、`@JsonComponent` → `@JacksonComponent`、`spring.jackson.use-jackson2-defaults` 过渡开关等**细粒度行为**，本轮未取到 Spring 官方原始页逐条陈述，落地前请以 docs.spring.io 的 Spring Boot 4.0 Reference 与 Jackson 3 官方迁移说明为准，本文不代为断言。

### 3.6 Spring Cloud 对齐（官方博客明确口径）

Spring Cloud 2025.1.1（Oakwood）官方发布博客原文：
- **Spring Cloud 2025.0.0 与 Spring Boot 4.0.1 及之后 4.0.x 不兼容**（因 Spring Boot 变更）
- **2025.1.x 列车引入对 Spring Boot 4.0.1+ 的兼容**
- 子项目版本跳到 5.0.1：Cloud Function / Task / Kubernetes / Commons / OpenFeign / Zookeeper
- BOM：`org.springframework.cloud:spring-cloud-dependencies:2025.1.1`

官方源：https://spring.io/blog/2026/01/29/spring-cloud-2025-1-1-aka-oakwood-has-been-released

### 3.7 流程引擎对齐

**Flowable**（官方对照表）：**2025.2.x ↔ Spring Boot 4.0.x**

官方源：https://documentation.flowable.com/latest/admin/release-notes/security-vulnerabilities

**Camunda 8 / Camunda 7 对 Boot 4 的专用 artifact**：⚠️ 本轮未取到 Camunda 官方原始页对 `camunda-spring-boot-4-starter` / `camunda-bpm-spring-boot-starter-4` 的明确声明，需以 Camunda Docs 与 Maven Central 制品页为准，本文不补造地址。

（Camunda 7 官方历史兼容页只覆盖到 2.3.x 线：https://docs.camunda.org/manual/7.13/user-guide/spring-boot-integration/version-compatibility）

### 3.8 常用工具组件在 Boot 4 的状态

**MyBatis**（官方兼容表）：`mybatis-spring-boot-starter` **4.0** ↔ MyBatis-Spring 4.0 ↔ Spring Boot **4.0+** ↔ Java 17+

官方源：https://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

**MyBatis-Plus**（官方安装页）：**自 3.5.13 起**提供 `com.baomidou:mybatis-plus-spring-boot4-starter`（当前 3.5.18）

官方源：https://baomidou.com/en/getting-started/install/

**springdoc-openapi 对 Boot 4**：⚠️ 官网 https://springdoc.org 本轮正文未被检索抽取到 Boot 4 兼容声明，需以官方 Release Notes 为准，本文不断言。

**Lombok / MapStruct / Hutool / Guava / Knife4j 对 Boot 4**：⚠️ 未取到官方原始兼容声明，按各库 release notes 核验。

---

## 四. 升级路径（按存量形态）

### 4.1 还在 2.x

1. 先到 **2.7.18**（2.x 末版），清空所有 deprecation 警告
2. JDK 升 17 或 21，`javax.*` → `jakarta.*` 用 OpenRewrite 或 IDE 批量替换
3. `mvn dependency:tree | grep javax` 确认无残留传递依赖
4. 加 `spring-boot-properties-migrator` 跑一轮配置诊断
5. 升到 **3.5.x 最新补丁**，Spring Cloud 同步 **2025.0.x Northfields**
6. 稳定后再评估上 4.x（**不建议 2.x 直跳 4.x**）

### 4.2 在 3.0~3.4

1. 先升 **3.5.x 最新补丁**，deprecation 清零（官方迁移指南明确要求「先到最新 3.5.x」再上 4.0）
2. Spring Cloud 切 **2025.0.x** 过渡
3. 再上 **4.0.x**，Spring Cloud 切 **2025.1.1+**（Oakwood；官方明确 2025.0 与 Boot 4.0.1+ 不兼容 ）

官方源：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide ；https://spring.io/blog/2026/01/29/spring-cloud-2025-1-1-aka-oakwood-has-been-released

### 4.3 新项目

直接 **Spring Boot 4.x + Java 21 或 25 + Spring Cloud 2025.1.x**，别再起 3.x 新项目。

---

## 五. 升级前核验清单

- [ ] `mvn dependency:tree` 扫全部 `javax.*` 残留（2→3 必做）
- [ ] 自研 starter / 自动配置类：`spring.factories` → `AutoConfiguration.imports`；4.x 还要跟模块化坐标
- [ ] Flyway / Liquibase：4.x 属性层级已改（见 §2.4），`clean-on-validation-error` 移除
- [ ] `spring-boot-starter-aop` → `spring-boot-starter-aspectj`；`spring-boot-tx` → `spring-boot-transaction`
- [ ] 可观测模块包名 `org.springframework.boot.metrics` → `org.springframework.boot.micrometer`
- [ ] Undertow / Pulsar Reactive / Spring Session Hazelcast&Mongo / Spock —— 用到就得找替代
- [ ] Spring Cloud BOM 强绑定：3.5↔2025.0.x，4.0.1+↔2025.1.x
- [ ] Flowable 升 2025.2.x（对应 Boot 4.0.x）
- [ ] MyBatis starter 4.0；MyBatis-Plus 换 `mybatis-plus-spring-boot4-starter`（3.5.13+）
- [ ] Kotlin ≥ 2.2、GraalVM native-image ≥ 25（若使用）
- [ ] `spring-boot-properties-migrator` 跑完**记得移除**

---

## 六. 官方原始网页地址总表

**Spring Boot / Spring Framework**
- Spring Boot 3.0 Migration Guide：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide
- Spring Boot 3.5.0 发布博客：https://spring.io/blog/2025/05/22/spring-boot-3-5-0-available-now
- Spring Boot 3.5 Release Notes：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes
- Spring Boot 4.0.0 发布博客：https://spring.io/blog/2025/11/20/spring-boot-4-0-0-available-now
- Spring Boot 4.0 Migration Guide：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide
- Spring Boot 4.0 Configuration Changelog：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Configuration-Changelog
- Spring Framework 7.0 GA 博客：https://spring.io/blog/2025/11/13/spring-framework-7-0-general-availability

**Spring Cloud**
- Spring Cloud 官方项目页（含 Release Train 兼容表）：https://spring.io/spring-cloud
- Spring Cloud 2025.1.1 Oakwood 发布博客：https://spring.io/blog/2026/01/29/spring-cloud-2025-1-1-aka-oakwood-has-been-released

**流程引擎**
- Flowable 官方 Spring Boot 版本对照：https://documentation.flowable.com/latest/admin/release-notes/security-vulnerabilities
- Camunda 7 官方 Spring Boot 兼容页：https://docs.camunda.org/manual/7.13/user-guide/spring-boot-integration/version-compatibility
- Camunda 8 / Spring Zeebe SDK 官方兼容页：⚠️ 未取到原始 URL

**常用工具**
- MyBatis spring-boot-starter 官方兼容表：https://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/
- MyBatis-Plus 官方安装页：https://baomidou.com/en/getting-started/install/
- springdoc-openapi 官网：https://springdoc.org （Boot 4 兼容声明未取到正文）

> 凡本文标注「⚠️ 未取到官方原始页」的条目（Camunda 8 SDK、springdoc Boot 4、Lombok / MapStruct / Hutool / Guava / Knife4j、Jackson 3 细粒度迁移项），请以各库官方 release notes 与 docs.spring.io 对应版本 Reference 为准，本文不代为断言。