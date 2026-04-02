---
layout: default
title: @EnableConfigurationProperties
parent: SpringBoot3
has_children: false
nav_order: 81
---


Here are webservices of CXF used examples on springboot3 .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }



## 目录

1. [概述](#概述)
2. [Spring Boot 1.x 时代](#spring-boot-1x-时代)
3. [Spring Boot 2.2 重大变革](#spring-boot-22-重大变革)
4. [Spring Boot 2.3+ 回归严谨](#spring-boot-23-回归严谨)
5. [Spring Boot 3.x 当前最佳实践](#spring-boot-3x-当前最佳实践)
6. [版本对比总表](#版本对比总表)
7. [常见问题与解决方案](#常见问题与解决方案)
8. [最佳实践总结](#最佳实践总结)

---


## Spring Boot @EnableConfigurationProperties 版本演进与使用指南

## 概述

`@ConfigurationProperties` 是 Spring Boot 中用于类型安全配置绑定的核心注解。然而，**仅仅添加 `@ConfigurationProperties` 注解并不会让类自动成为 Spring Bean**。

在不同版本的 Spring Boot 中，配置属性类的注册机制经历了多次演变：

| 版本 | 注册方式 | 默认行为 |
|------|----------|----------|
| 1.x | `@EnableConfigurationProperties` 或 `@Component` | 需显式声明 |
| 2.2 | `@ConfigurationPropertiesScan` | 自动扫描（默认开启） |
| 2.3+ | `@ConfigurationPropertiesScan` | 需显式声明（默认关闭） |
| 3.x | `@ConfigurationPropertiesScan` | 需显式声明（默认关闭） |

---

## Spring Boot 1.x 时代

### 核心机制

在 Spring Boot 1.x 中，`@ConfigurationProperties` 注解只是告诉 Spring 如何绑定配置属性，但**不会自动注册为 Bean**。

必须通过以下方式之一显式注册：

### 方式一：@EnableConfigurationProperties（推荐）

```java
// 配置属性类（不需要 @Component）
@ConfigurationProperties(prefix = "mail")
public class MailProperties {
    private String hostName;
    private int port;
    private String from;
    
    // getters and setters
}

// 在配置类上开启
@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class MailAutoConfiguration {
    
    @Bean
    public MailService mailService(MailProperties properties) {
        return new MailService(properties.getHostName(), 
                              properties.getPort(), 
                              properties.getFrom());
    }
}
```

### 方式二：@Component 注解

```java
@Component
@ConfigurationProperties(prefix = "mail")
public class MailProperties {
    private String hostName;
    private int port;
    private String from;
    
    // getters and setters
}
```

**注意**：这种方式会让配置类被组件扫描，可能导致与 `@EnableConfigurationProperties` 方式产生重复 Bean。

### 方式三：@Bean 方法

```java
@Configuration
public class MailConfiguration {
    
    @Bean
    @ConfigurationProperties(prefix = "mail")
    public MailProperties mailProperties() {
        return new MailProperties();
    }
}
```

### 1.x 版本的局限性

1. **必须显式声明**：每个配置类都需要在某个地方注册
2. **容易遗漏**：忘记注册会导致配置不生效，且无明显错误提示
3. **组件扫描污染**：使用 `@Component` 会让配置类参与组件扫描

---

## Spring Boot 2.2 重大变革

### 新增 @ConfigurationPropertiesScan

Spring Boot 2.2 引入了 `@ConfigurationPropertiesScan` 注解，实现了**自动扫描并注册** `@ConfigurationProperties` 类。

### 核心变化

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

在 2.2 版本中，**只要配置类在 `@SpringBootApplication` 扫描的包路径下**，就会被自动发现和注册。

### 使用示例

```java
// 配置属性类（不需要任何额外注解）
@ConfigurationProperties(prefix = "mail")
public class MailProperties {
    private String hostName;
    private int port;
    private String from;
    
    // getters and setters
}

// 启动类（2.2 默认开启扫描）
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 可以直接注入使用
@Service
public class MailService {
    private final MailProperties mailProperties;
    
    public MailService(MailProperties mailProperties) {  // 自动注入
        this.mailProperties = mailProperties;
    }
}
```

### 自定义扫描路径

如果配置类不在启动类所在包下，可以使用 `@ConfigurationPropertiesScan` 指定扫描路径：

```java
@SpringBootApplication
@ConfigurationPropertiesScan("com.example.config")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2.2 的问题

2.2 的自动扫描导致了一个严重问题：**无法条件化注册配置属性类**。

```java
// 这种模式在 2.2 中会失败
@Configuration
@ConditionalOnProperty(prefix = "mail", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(MailProperties.class)  // 条件不满足时不会注册
public class MailAutoConfiguration {
    // ...
}

// 但由于自动扫描，MailProperties 总是会被注册
// 导致 @ConditionalOnProperty 失效
```

---

## Spring Boot 2.3+ 回归严谨

### 默认关闭自动扫描

从 Spring Boot 2.3 开始，**默认不再自动扫描 `@ConfigurationProperties` 类**，恢复了 2.1 的行为。

### 必须显式声明

要启用自动扫描，必须显式添加 `@ConfigurationPropertiesScan`：

```java
@SpringBootApplication
@ConfigurationPropertiesScan  // 必须显式声明
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 构造器绑定支持

Spring Boot 2.2+ 引入了构造器绑定（Constructor Binding），需要显式启用：

```java
@ConfigurationProperties(prefix = "mail")
@ConstructorBinding  // 2.2+ 新增
public record MailProperties(
    String hostName,
    int port,
    String from
) {}
```

**注意**：使用构造器绑定时，必须通过 `@EnableConfigurationProperties` 或 `@ConfigurationPropertiesScan` 显式注册。

### 2.3+ 版本对比

| 版本 | 默认扫描 | 构造器绑定 | 条件注册支持 |
|------|----------|------------|--------------|
| 2.2.0 | ✅ 开启 | ✅ 支持 | ❌ 不支持 |
| 2.2.x | ✅ 开启 | ✅ 支持 | ❌ 不支持 |
| 2.3.0+ | ❌ 关闭 | ✅ 支持 | ✅ 支持 |

---

## Spring Boot 3.x 当前最佳实践

### 当前推荐方式

Spring Boot 3.x 延续了 2.3+ 的设计，**默认不自动扫描**，推荐显式声明。

### 方式一：@ConfigurationPropertiesScan（推荐）

```java
@SpringBootApplication
@ConfigurationPropertiesScan  // 显式开启扫描
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 配置类
@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(
    String host,
    int port,
    String username,
    String password
) {}
```

### 方式二：@EnableConfigurationProperties

```java
@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class MailAutoConfiguration {
    // ...
}

@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(
    String host,
    int port,
    String username,
    String password
) {}
```

### 方式三：@Bean 方法

```java
@Configuration
public class MailConfiguration {
    
    @Bean
    @ConfigurationProperties(prefix = "app.mail")
    public MailProperties mailProperties() {
        return new MailProperties();
    }
}
```

### Spring Boot 3.x 新特性

#### 不可变配置绑定

```java
@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(
    String host,        // 必须在构造器中赋值
    int port,
    String username,
    String password
) {}
```

#### 嵌套配置绑定

```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    String name,
    MailProperties mail
) {}

public record MailProperties(
    String host,
    int port
) {}
```

#### 响应式支持

```java
@ConfigurationProperties(prefix = "app")
public class ReactiveMailProperties {
    private final String host;
    private final int port;
    
    // 响应式场景下的配置绑定
}
```

---

## 版本对比总表

| 特性 | 1.x | 2.0-2.1 | 2.2 | 2.3+ | 3.x |
|------|-----|---------|-----|------|-----|
| 默认扫描 | ❌ | ❌ | ✅ | ❌ | ❌ |
| 需要显式声明 | ✅ | ✅ | ❌ | ✅ | ✅ |
| 构造器绑定 | ❌ | ❌ | ✅ | ✅ | ✅ |
| 条件注册 | ✅ | ✅ | ❌ | ✅ | ✅ |
| `@ConfigurationPropertiesScan` | ❌ | ❌ | ✅ | ✅ | ✅ |
| `@EnableConfigurationProperties` | ✅ | ✅ | ✅ | ✅ | ✅ |

### 版本迁移指南

#### 从 1.x/2.0-2.1 升级到 2.2+

```java
// 旧方式（1.x/2.0-2.1）
@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class MailAutoConfiguration {
}

// 新方式（2.2+）
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application {
}
```

#### 从 2.2 升级到 2.3+/3.x

```java
// 2.2（自动扫描，无需额外注解）
@SpringBootApplication
public class Application {
}

// 2.3+/3.x（需要显式声明）
@SpringBootApplication
@ConfigurationPropertiesScan  // 新增
public class Application {
}
```

---

## 常见问题与解决方案

### 问题一：配置类不生效

**错误信息**：
```
No constructor binding found for @ConfigurationProperties class
```

**原因**：配置类没有被注册为 Bean。

**解决方案**：

```java
// 方案 1：添加 @ConfigurationPropertiesScan
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application {
}

// 方案 2：添加 @EnableConfigurationProperties
@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class Config {
}

// 方案 3：添加 @Component（不推荐）
@Component
@ConfigurationProperties(prefix = "mail")
public class MailProperties {
}
```

### 问题二：条件注册失效

**场景**：
```java
@Configuration
@ConditionalOnProperty(prefix = "mail", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(MailProperties.class)
public class MailAutoConfiguration {
}
```

**问题**：即使 `mail.enabled=false`，`MailProperties` 仍然被注册。

**原因**：2.2 的自动扫描导致配置类总是被注册。

**解决方案**（2.3+）：

```java
// 不要使用 @ConfigurationPropertiesScan，改用 @EnableConfigurationProperties
@Configuration
@ConditionalOnProperty(prefix = "mail", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(MailProperties.class)
public class MailAutoConfiguration {
}
```

### 问题三：构造器绑定不生效

**错误信息**：
```
Not registered via @EnableConfigurationProperties, marked as Spring component, 
or scanned via @ConfigurationPropertiesScan
```

**原因**：使用构造器绑定时，必须显式注册。

**解决方案**：

```java
// 必须显式注册
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application {
}

@ConfigurationProperties(prefix = "mail")
@ConstructorBinding  // 或使用 record（自动启用）
public record MailProperties(String host, int port) {
}
```

### 问题四：多环境配置冲突

**场景**：不同环境使用不同的配置类。

**解决方案**：

```java
// 使用 @Profile 隔离
@Configuration
@Profile("dev")
@EnableConfigurationProperties(DevMailProperties.class)
public class DevMailConfig {
}

@Configuration
@Profile("prod")
@EnableConfigurationProperties(ProdMailProperties.class)
public class ProdMailConfig {
}
```

---

## 最佳实践总结

### 1. Spring Boot 3.x 推荐用法

```java
// 启动类
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 配置类（使用 record 实现不可变）
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    String name,
    MailProperties mail,
    DatabaseProperties database
) {}

public record MailProperties(String host, int port) {}
public record DatabaseProperties(String url, String username) {}
```

### 2. 条件化配置

```java
// 需要条件化注册时，使用 @EnableConfigurationProperties
@Configuration
@ConditionalOnProperty(prefix = "app.mail", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(MailProperties.class)
public class MailAutoConfiguration {
    
    @Bean
    public MailService mailService(MailProperties properties) {
        return new MailService(properties);
    }
}
```

### 3. 配置类设计原则

```java
// ✅ 推荐：不可变、使用 record
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    String name,
    @DefaultValue("8080") int port
) {}

// ❌ 不推荐：可变、使用 @Component
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private int port;
    // setters...
}
```

### 4. 测试配置

```java
@SpringBootTest
@TestPropertySource(locations = "classpath:test-config.properties")
class ApplicationTests {
    
    @Autowired
    private AppProperties appProperties;
    
    @Test
    void contextLoads() {
        assertThat(appProperties).isNotNull();
    }
}
```

### 5. 版本选择建议

| 场景 | 推荐版本 | 原因 |
|------|----------|------|
| 新项目 | 3.x | 最新特性、长期支持 |
| 维护旧项目 | 2.7 | LTS、兼容性好 |
| 需要自动扫描 | 2.2 | 默认开启 |
| 需要条件注册 | 2.3+ | 支持条件化 |

---

## 参考资料

- [Spring Boot 官方文档 - Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)
- [Spring Boot 2.2 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2.0-Release-Notes)
- [GitHub Issue #12602 - Add support for ConfigurationProperties scanning](https://github.com/spring-projects/spring-boot/issues/12602)
- [GitHub Issue #18674 - Enabling configuration properties scanning by default](https://github.com/spring-projects/spring-boot/issues/18674)

---

*最后更新时间：2026-06-08*
