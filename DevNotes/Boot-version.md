---
layout: default
title: Version Mix
parent: DevNotes
nav_order: 0
---

## 开发常用版本搭配
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }


<div class="my-right-toc" markdown="1">
1. TOC
{:toc}
</div>

## 一、Spring Boot 主版本对照表

>纵向：一行看完一代


| Spring Boot | JDK 支持范围 | 推荐 JDK* | 最低 Maven | Maven 推荐* | 最低 Gradle | Gradle 推荐* | Spring Framework | Hibernate ORM | Jakarta EE 9+ | 内嵌容器 |
|---|---|---|---|---|---|---|---|---|---|---|
| **1.5.x** | 6–8 | 8 | 3.2+ | 3.3.9 | 3.x | 4.x | 4.3.x | 5.0.x | ❌ Java EE 7 | Tomcat 8.5 |
| **2.0.x** | 8–9 | 8 | 3.5+ | 3.5.4 | 4.4 | 4.10 | 5.0.x | 5.2.x | ❌ Java EE 8 | Tomcat 8.5 / 9.0 |
| **2.1.x** | 8–12 | 8 | 3.5+ | 3.5.4 | 4.4 | 4.10 | 5.1.x | 5.3.x | ❌ Java EE 8 | Tomcat 8.5 / 9.0 |
| **2.2.x** | 8–15 | 8 或 11 | 3.5+ | 3.6.3 | 5.6.x | 6.x | 5.2.x | 5.4.x | ❌ Java EE 8 | Tomcat 9.0 |
| **2.3.x** | 8–15 | 8 或 11 | 3.5+ | 3.6.3 | 5.6.x | 6.x | 5.2.x | 5.4.x | ❌ Java EE 8 | Tomcat 9.0 |
| **2.4.x** | 8–16 | 11 | 3.3+（2.4.0–2.4.3）<br>3.5+（2.4.4+） | 3.6.3 | 6.3 | 6.9 | 5.3.x | 5.4.x | ❌ Java EE 8 | Tomcat 9.0 |
| **2.5.x** | 8–18 | 11 | 3.5+ | 3.6.3 | 6.8 | 6.9 | 5.3.x | 5.4.x | ❌ Java EE 8 | Tomcat 9.0 |
| **2.6.x** | 8–19 | 11 或 17 | 3.5+ | 3.8.x | 6.8 | 7.6 | 5.3.x | 5.6.x | ❌ Java EE 8 | Tomcat 9.0 |
| **2.7.x** | 8–21 | 11 或 17 | 3.5+ | 3.8.x | 6.8 | 7.6 | 5.3.x | 5.6.x | ❌ Java EE 8 | Tomcat 9.0 |
| **3.0.x** | 17–19 | 17 | 3.5+ | 3.9.x | 7.5 | 7.6 | 6.0.x | 6.1.x | ✅ Jakarta EE 10 | Tomcat 10.1 |
| **3.1.x** | 17–20 | 17 | 3.5+ | 3.9.x | 7.5 | 7.6 | 6.0.x | 6.2.x | ✅ Jakarta EE 10 | Tomcat 10.1 |
| **3.2.x** | 17–21 | 21 | 3.6.3+ | 3.9.x | 7.5 | 8.x | 6.1.x | 6.3 → 6.4（3.2.1 起） | ✅ Jakarta EE 10 | Tomcat 10.1 |
| **3.3.x** | 17–23 | 21 | 3.6.3+ | 3.9.x | 7.5 | 8.x | 6.1.x | 6.5.x | ✅ Jakarta EE 10 | Tomcat 10.1 |
| **3.4.x** | 17–24 | 21 | 3.6.3+ | 3.9.x | 7.6.4 或 8.4 | 8.7+ | 6.2.x | 6.6.x | ✅ Jakarta EE 10 | Tomcat 10.1 |
| **3.5.x** | 17–25 | 21 或 25 | 3.6.3+ | 3.9.x | 7.6.4 或 8.4 | 8.14 | 6.2.x | 6.6.x | ✅ Jakarta EE 10 | Tomcat 10.1 |
| **4.0.x** | 17–25 | 25 | 3.6.3+ | 3.9.x | 8.14 | 8.14 | 7.0.x | 7.1 → 7.2（4.0.1 起） | ✅ Jakarta EE 11 | Tomcat 11.0 / Jetty 12.1 |
| **4.1.x** | 17–26 | 25 | 3.6.3+ | 3.9.x | 8.14 | 8.14 / 9.x | 7.0.x | 7.2.x | ✅ Jakarta EE 11 | Tomcat 11.0 / Jetty 12.1 |
| **4.2.x** | 17–27 | 25 | 3.6.3+ | 3.9.x | 8.14 | 8.14 / 9.x | 7.0.x | 7.4.x | ✅ Jakarta EE 11 | Tomcat 11.0 / Jetty 12.1 |

- Spring Boot 3.x 起，不再支持 JDK 8/11，最低要求 JDK 17
- Spring Boot 3.x 全面迁移到 Jakarta EE 9+，包名从 javax.servlet → jakarta.servlet
- Spring Boot 2.x：Maven ≥ 3.5.0 ; Spring Boot 3.x：Maven ≥ 3.6.3（官方强烈建议 3.8+）


**配套 Spring Cloud 列车号（关键锚点）**

| Spring Boot | Spring Cloud |
|---|---|
| 1.5.x | Edgware（末班） |
| 2.0.x – 2.2.x | Finchley / Greenwich |
| 2.3.x – 2.4.x | Hoxton |
| 2.5.x – 2.7.x | 2020.0.x（Ilford）– 2021.0.x（Jubilee） |
| 3.0.x – 3.2.x | 2022.0.x（Kilburn）– 2023.0.x |
| 3.4.x | 2024.0.x |
| 3.5.x | 2025.0.x（Northfields） |
| 4.0.x – 4.1.x | 2025.1.x（Oakwood） |

---

## 二、MyBatis 兼容表

| mybatis-spring-boot-starter | MyBatis-Spring | 适配 Spring Boot | JDK | 状态 |
|---|---|---|---|---|
| 4.0 | 4.0 | 4.0+ | 17+ | 当前 |
| 3.0 | 3.0 | 3.2 – 3.5 | 17+ | 当前 |
| 2.3 | 2.1 | 2.7 | 8+ | 当前 |
| 2.2 | 2.0（≥ 2.0.6） | 2.5 – 2.7 | 8+ | EOL |
| 2.1 | 2.0（≥ 2.0.6） | 2.1 – 2.4 | 8+ | EOL |
| 2.0 | 2.0 | 2.0 / 2.1 | 8+ | EOL |
| 1.3 | 1.3 | 1.5 | 6+ | EOL |

**这张表暴露的一个坑**：3.0 那一行覆盖 Boot 3.2–3.5，而 Boot 3.0 / 3.1 在官方 starter 矩阵里**没有对应版本**。跑 3.0/3.1 的项目只能退用 2.3.x 并自行验证，或者干脆升到 3.2+。

**实践建议**：MyBatis 版本必须在 `<properties>` 里**显式声明**，不要指望 BOM 托管，并在升级 Boot 时同步对照上表核对。

> mybatis-spring-boot-starter 由 MyBatis 社区维护，不在 spring-boot-dependencies BOM 的托管清单里——BOM 中不存在 mybatis.version 属性。因此官方从未为任何一代 Spring Boot 指定过 MyBatis 版本。本文第二节的 MyBatis 表取自 MyBatis 官方 starter 文档自己声明的兼容矩阵，这是目前唯一权威来源。


## 三、代际注意项

| 代际 | 注意项 |
|---|---|
| **1.5.x** | 2019 年已 EOL；Java 9 未正式支持；末班 Spring Cloud 为 Edgware；Actuator 端点默认不设安全，需手动加固；自动配置仍走 `spring.factories` |
| **2.0 – 2.1** | 2.0 是首个 JDK 8 基线 + WebFlux 代际，HikariCP 成默认连接池；2.1 默认关闭 bean 定义覆盖（`spring.main.allow-bean-definition-overriding`）；**2.0 最高只到 Java 9，Java 11 上跑 2.0 会出问题**，需先升 2.1 |
| **2.2 – 2.3** | 2.2 起 `spring.factories` 增加条件过滤，自定义 starter 需补 `META-INF/spring-autoconfigure-metadata.properties`；2.3 引入分层镜像（`spring-boot-layertools`），Dockerfile 需同步调整；H2 与 JUnit 5 版本变动较大 |
| **2.4 – 2.5** | **2.4 是破坏性的一代**：`application.yml` 配置树合并规则变更、禁用旧的 `spring.profiles.include` 语义、激活改由 `spring.config.activate.on-profile` 承担；2.4.0–2.4.3 只到 Java 15，2.4.4+ 才支持 16；Gradle 下限从 5.6.x 抬到 6.3 |
| **2.6 – 2.7** | 2.6 默认**禁止循环依赖**（`spring.main.allow-circular-references=false`），老项目升级最常炸的一条；2.7 是 2.x 终版与跳板，`AutoConfiguration.imports` 替代 `spring.factories`（旧机制仍兼容）；2.7.x 商业支持延至 **2029-06** |
| **3.0 – 3.1** | **三重硬门槛同时到来**：Java 17 + `javax.*` → `jakarta.*` + Spring Framework 6；Spring Security 6 移除 `WebSecurityConfigurerAdapter`，须改为 `SecurityFilterChain` Bean 写法；`@ConstructorBinding` 语义变化；Spring Cloud 必须配 2022.0.x（Kilburn），Netflix 组件曾因未适配 Jakarta EE 缺席首发版本 |
| **3.2 – 3.3** | 3.2 引入 `RestClient`、虚拟线程支持（需 Java 21）；Spring Data 换到 2023.x 列车号，Boot 与 Cloud 版本须成对升；Gradle 下限仍是 7.5（到 3.4 才抬到 7.6.4 / 8.4） |
| **3.4** | 3.4.0 官方页曾误写「Java 16 起」，实为 **Java 17**（后续补丁已修正），不要照抄旧文档；`MockitoTestExecutionListener` 在此代标记废弃，为 4.x 移除铺路 |
| **3.5** | 3.x 终版；**OSS 支持已于 2026-06-30 结束**，商业支持到 2032-06；目前是想停在 3.x 的新项目唯一推荐选择；升 4.x 的过渡成本在 3.x 各代中最低 |
| **4.0 – 4.2** | 见第四节专项 |

---

## 四、Spring Boot 4.x 专项注意项

4.x 是架构级重构，升级前必须逐条核对：

1.**Undertow 被彻底移除**。它未实现 Servlet 6.1，Spring Framework 7 也删掉了 Undertow 专属的 WebSocket / WebFlux 类。使用 `spring-boot-starter-undertow` 的项目必须**先在 3.x 迁到 Tomcat / Jetty / Reactor Netty**，否则升上去直接构建失败；`server.undertow.*` 全部配置项需在新容器上重做。

2.**Jackson 3 成为默认**。包名从 `com.fasterxml.jackson` 迁到 `tools.jackson`（`jackson-annotations` 例外，仍留 `com.fasterxml.jackson.annotation`，所以 DTO 上的注解不用改）。`@JsonComponent` → `@JacksonComponent`，`@JsonMixin` → `@JacksonMixin`，`Jackson2ObjectMapperBuilderCustomizer` → `JsonMapperBuilderCustomizer`。**声明 `ObjectMapper` Bean 不再替换自动配置的 mapper**，需改声明 `JsonMapper`。`spring.jackson.read.*` / `write.*` 移到 `spring.jackson.json.read.*` / `write.*`。Jackson 2 依赖仍保留但已进废弃通道，可用 `spring.jackson.use-jackson2-defaults=true` 平滑过渡。

3.**模块被拆碎**。`spring-boot-autoconfigure` 拆成多个 `spring-boot-{module}`，自动配置类包名迁移（如 `@EntityScan` 移到 `org.springframework.boot.persistence.autoconfigure`）。**Flyway / Liquibase 现在必须显式引 `spring-boot-starter-flyway` / `spring-boot-starter-liquibase`**，否则启动时不执行数据库迁移。过渡期可用 `spring-boot-starter-classic` / `spring-boot-starter-test-classic` 先编译通过，再逐个收敛到精确 starter。

4.**starter 改名**（旧名在 4.0 仍可用但已废弃）：`spring-boot-starter-web` → `-webmvc`、`-web-services` → `-webservices`、`-oauth2-client` → `-security-oauth2-client`、`-oauth2-resource-server` → `-security-oauth2-resource-server`、`-aop` → `-aspectj`。

5.**测试基础设施大改**。`@MockBean` / `@SpyBean` **已删除**，改用 `@MockitoBean` / `@MockitoSpyBean`；`@SpringBootTest` 不再自动提供 `MockMvc`（需 `@AutoConfigureMockMvc`），也不再提供 `TestRestTemplate` / `WebClient`（需 `@AutoConfigureRestTestClient` 或迁到新的 `RestTestClient`）；`MockitoTestExecutionListener` 已移除，`@Mock` / `@Captor` 字段需显式挂 `MockitoExtension`。

6.**Jakarta EE 11**：Servlet 6.1、JPA 3.2、Validation 3.1、WebSocket 2.2；JPA 4.0 / 4.1 分别对应 Spring Data 2025.1 / 2026.0。

7.**Kotlin 基线抬到 2.x**（Boot 4 要求 2.2+，Boot 3.0 时代是 1.7+），Groovy 需同步升级，老版本会因编译器 API 变更而失败。

8.**Lombok 需 1.18.30+**，低版本的注解处理器会冲突；Hutool、Guava 等工具库老版本存在反射调用失效问题，建议一并升级。

9.**GraalVM 原生镜像从实验转生产级**：Boot 4.0 要求 GraalVM Community 25 + Native Build Tools 0.11.3+（4.0.10 对应 0.11.5）。

10.**Spring Cloud 必须对齐**：4.0 / 4.1 配 2025.1.x（Oakwood），3.5 配 2025.0.x（Northfields），不可混搭。

---

## 五、选型建议

### 5.1 新项目

| 情况 | 建议 |
|---|---|
| **首选** | Spring Boot **4.2 + JDK 25**（LTS） |
| **依赖生态未跟上时**（Spring Cloud Alibaba、部分国产中间件、老 SDK 尚无 Jakarta EE 11 版本） | 退到 **3.5 + JDK 21**。注意 3.5 的 OSS 支持已于 2026-06-30 结束，安全补丁需商业订阅——这是一个需要明确记入技术债的决策 |
| **务必避开** | 3.0 – 3.4（已有更新代际且 3.5 迁移成本很低）、2.x 全线（Java 8 + javax，生态加速断裂） |

### 5.2 存量系统升级路径

**核心原则：不跨代跳。**

```
1.5 → 2.7 → 3.x → 4.x
```

- **1.5 → 2.7**：主要是 Java 6/7 → 8 与依赖清理，相对机械。
- **2.7 → 3.x 是最贵的一跳**：Java 8 → 17、`javax.*` → `jakarta.*`、Spring Security 6 配置重写三件事叠加。建议**单独立为一个迭代周期**，不要和业务大改并行。
    - 包名迁移可用 OpenRewrite 的 `UpgradeSpringBoot_3_0` recipe 批量处理；
    - 但**依赖树必须人工过一遍**——第三方库没有 Jakarta 兼容版本会成为硬阻塞，这类问题机器解决不了，只能换库或等社区。
- **3.5 → 4.x**：改动集中且明确（第四节那 10 条），配合 `spring-boot-starter-classic` 做过渡，可在一个迭代内完成。**这也是建议先把 2.x 项目推到 3.5 而非直接冲 4.x 的原因**——3.5 是唯一能让你把兼容性问题分批消化的落点。

### 5.3 构建工具

| 场景 | Maven | Gradle |
|---|---|---|
| Boot 4.x | 3.9.x | 8.14 或 9.x |
| Boot 3.x | 3.9.x | 8.4+ / 7.6.4+（3.2 及以前 7.5 即可） |
| Boot 2.7 | 3.8.x / 3.9.x | 7.6 |
| Boot 2.0 – 2.3 | 3.5.4 – 3.6.3 | 4.10 – 6.x |

Maven 3.6.3 及之前已停止维护，且大量新插件要求 3.6.3+，一律提到 3.9.x 最省心。

### 5.4 MyBatis

版本**必须显式声明**在 `<properties>` 中，并对照第二节表格核对 starter 与 Boot 的对应关系。特别注意 Boot 3.0 / 3.1 无官方匹配的 starter 版本。

### 5.5 一个实用的自查命令

升级前后各跑一次，把第三方依赖版本差异落到纸面，比对着文档猜准得多：

```bash
mvn dependency:tree -DoutputFile=deps-before.txt
./gradlew dependencies --configuration runtimeClasspath > deps-before.txt
```

重点 grep 这几类：`undertow`、`jackson`、`javax.`、`jakarta.`、`hibernate`、`mybatis`。


## 六、IntelliJ IDEA 版本兼容JDK、SpringBoot环境版本

|IDEA 版本 |支持的 Spring Boot |支持的 JDK|建议场景|
|-----|--------------|-----|-----|
|IDEA 2024.2+ |Spring Boot 2.6 ~ 3.3|JDK 8 ~ 23|最新开发（推荐）|
|IDEA 2023.3 |Spring Boot 2.5 ~ 3.2 |JDK 8 ~ 21|稳定主力|
|IDEA 2022.3 |Spring Boot 2.4 ~ 3.0 |JDK 8 ~ 19|老项目维护|
|IDEA < 2022.1 |不支持 Spring Boot 3.x |不支持 JDK 19+ |❌ 避免用于新项目|

> 使用 IDEA 2023.3 或 2024.x 开发 Spring Boot 3.x 项目，对 Jakarta EE、虚拟线程（JDK 21）等新特性支持更好。

## 七、常见组合

A : 已有文档项目

|组件	|版本|
|-------|----|
|Spring Boot	|2.7.18（LTS，支持到 2025.11）|
|JDK	|17（LTS）|
|Maven	|3.8.6|
|IntelliJ IDEA	|2023.3+|

>社区成熟、依赖库兼容性好、无需迁移 Jakarta, servlet api <= 2.x

B : 新项目（推荐）

| 组件	           |版本|
|----------------|----|
| Spring Boot	   |3.3.0|
| JDK	           |21（LTS，支持虚拟线程）|
| Maven	         |3.9.6+|
| IntelliJ IDEA  |2024.1+ |

> 享受最新特性（如 AOT 编译、GraalVM 原生 镜像 ）、长期支持、性能更好。