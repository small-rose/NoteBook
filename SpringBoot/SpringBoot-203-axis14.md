---
layout: default
title: SpringBoot2+axis1.4
parent: SpringBoot
nav_order: 300
---


Here are SpringBoot session experience .
{: .fs-6 .fw-300 }


##   Here are axis 1.4 using demo experience .
{: .no_toc .text-delta }

1. TOC
{:toc}


# Spring Boot 2.5 + Axis 1.4 发布 WebService 服务端完整教程

## 目录

1. [版本搭配说明](#1-版本搭配说明)
2. [项目结构概览](#2-项目结构概览)
3. [完整构建配置](#3-完整构建配置)
4. [三种消息模式详解](#4-三种消息模式详解)
5. [核心配置与安全](#5-核心配置与安全)
6. [实现案例代码](#6-实现案例代码)
7. [Security Patches（安全补丁）](#7-security-patches安全补丁)
8. [构建与运行](#8-构建与运行)
9. [客户端调用示例](#9-客户端调用示例)

---

## 1. 版本搭配说明

| 组件 | 版本 | 说明 |
|------|------|------|
| Spring Boot | **2.5.13** | 最后一个兼容 Java 8 的稳定版本线 |
| Axis | **1.4-patched** | 1.4 基线 + 手工 CVE 补丁 |
| Java | **1.8 (JDK 8)** | 长期支持，与 Axis 1.4 完全兼容 |
| Gradle | **7.3.3** (wrapper) | 兼容 JDK 8 且支持 Spring Boot 2.5 的最低版本 |
| javax.xml.rpc-api | 1.1.2 | Axis 的 JAX-RPC 依赖 |
| wsdl4j | 1.6.3 | WSDL 解析 |
| commons-discovery | 0.5 | Axis 动态发现机制 |

> **注意**：Axis 1.4 官方仅发布到 Java 1.4 编译的 jar，**不兼容 JDK 17+**（无法反射处理 module 限制）。生产环境建议使用 JDK 8 或 JDK 11。

---

## 2. 项目结构概览

```
demo-boot2-axis/
├── build.gradle                    # 构建脚本
├── settings.gradle                 # 项目设置
├── gradle.properties               # Gradle 属性
├── gradlew.bat                     # Gradle Wrapper (Windows)
├── patch-axis.bat                  # 重新构建 patched jar 脚本
├── lib/
│   └── axis-1.4-patched.jar        # flatDir 回退 jar
├── src/main/
│   ├── java/com/small/rose/demo/
│   │   ├── DemoBoot2AxisApplication.java
│   │   ├── config/
│   │   │   ├── AxisConfig.java             # Axis Servlet + Filter 注册
│   │   │   ├── AxisSecurityFilter.java     # 安全过滤器
│   │   │   └── AxisStartupLogger.java      # 启动日志
│   │   └── webservice/
│   │       ├── HelloWebService.java        # RPC/encoded 接口
│   │       ├── HelloWebServiceImpl.java    # RPC/encoded 实现
│   │       ├── HelloRpcLiteralService.java # RPC/literal 接口
│   │       ├── HelloRpcLiteralServiceImpl.java
│   │       ├── HelloDocLiteralService.java # Document/literal 接口
│   │       ├── HelloDocLiteralServiceImpl.java
│   │       ├── User.java                   # 公共模型
│   │       └── doclit/                     # Document/literal 包装类
│   │           ├── SayHelloRequest.java
│   │           ├── SayHelloResponse.java
│   │           ├── GetUserRequest.java
│   │           └── GetUserResponse.java
│   └── resources/
│       ├── application.yml
│       ├── server-config.wsdd              # Axis 部署描述符
│       └── axis-patch/                     # 补丁源码
└── test/
    └── java/.../DemoBoot2AxisApplicationTests.java
```

---

## 3. 完整构建配置

### 3.1 build.gradle

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.5.13'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
}

group = 'com.small.rose.demo'
version = '1.0.0'
sourceCompatibility = '1.8'

tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}

repositories {
    mavenCentral()
    mavenLocal()
    flatDir { dirs 'lib' }          // flatDir 回退
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // 使用 patched 版本
    implementation ('org.apache.axis:axis:1.4-patched') {
        force = true
    }
    implementation 'javax.xml.rpc:javax.xml.rpc-api:1.1.2'
    implementation 'commons-logging:commons-logging:1.2'
    implementation 'commons-discovery:commons-discovery:0.5'
    implementation 'wsdl4j:wsdl4j:1.6.3'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}
```

> **关键点**：`axis:1.4-patched` 通过 `mavenLocal()` 或 `flatDir` 解析（详见第 7 节）。

### 3.2 server-config.wsdd

这是 Axis 的**服务端部署描述符**，等价于传统 Servlet 项目中的 `WEB-INF/server-config.wsdd`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<deployment xmlns="http://xml.apache.org/axis/wsdd/"
    xmlns:java="http://xml.apache.org/axis/wsdd/providers/java">

    <handler type="java:org.apache.axis.handlers.http.URLMapper" name="URLMapper"/>

    <!-- RPC/encoded -->
    <service name="HelloWebService" provider="java:RPC" style="rpc" use="encoded">
        <parameter name="className" value="com.small.rose.demo.webservice.HelloWebServiceImpl"/>
        <parameter name="allowedMethods" value="sayHello getUser"/>
    </service>

    <!-- RPC/literal -->
    <service name="HelloRpcLiteralService" provider="java:RPC" style="rpc" use="literal">
        <parameter name="className" value="com.small.rose.demo.webservice.HelloRpcLiteralServiceImpl"/>
        <parameter name="allowedMethods" value="sayHello getUser"/>
    </service>

    <!-- Document/literal -->
    <service name="HelloDocLiteralService" provider="java:RPC" style="document" use="literal">
        <parameter name="className" value="com.small.rose.demo.webservice.HelloDocLiteralServiceImpl"/>
        <parameter name="allowedMethods" value="sayHello getUser"/>
    </service>

    <transport name="http">
        <requestFlow>
            <handler type="URLMapper"/>
        </requestFlow>
    </transport>
</deployment>
```

配置说明：

| 属性 | 含义 | 取值 |
|------|------|------|
| `style` | 消息风格 | `rpc` / `document` / `message` |
| `use` | 编码风格 | `encoded` / `literal` |
| `provider` | 服务提供者 | `java:RPC` 表示 Java POJO |
| `allowedMethods` | **允许暴露的方法** | 逗号分隔，**不要写 `*`** |

> **security**：`allowedMethods="*"` 会暴露 Axis 内部全部方法（含 `AdminServlet`），必须显式列出。

### 3.3 application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: demo-boot2-axis
```

---

## 4. 三种消息模式详解

Axis 支持 WSDL 定义的 `style` x `use` 组合，常见三种。

### 4.1 RPC/encoded（style="rpc", use="encoded"）

**特点**：
- 消息体遵循 SOAP RPC 约定：方法名作为包装元素，参数按顺序映射
- 使用 SOAP **Section 5 编码规则**（`soapenc:Array`、`soapenc:Struct`）
- Axis 1.4 的**默认模式**

**SOAP 请求示例**：
```xml
<soap:Body>
    <ns1:sayHello xmlns:ns1="urn:HelloWebService">
        <name xsi:type="xsd:string">World</name>
    </ns1:sayHello>
</soap:Body>
```

**优点**：简单直接，接口 = Java 方法的 1:1 映射。

**缺点**：
- SOAP Section 5 编码**不是 WS-I BP 合规的**
- 跨平台互操作性差（.NET 3.5+ 已放弃支持）
- 无法通过 XML Schema 严格校验

**适用场景**：遗留系统、内部服务、快速原型。

### 4.2 RPC/literal（style="rpc", use="literal"）

**特点**：
- 消息体仍然是 RPC 包装（方法名作为包装元素）
- 但参数使用 **Literal** XML（遵循 XML Schema，不使用 SOAP 编码规则）

**SOAP 请求示例**：
```xml
<soap:Body>
    <ns1:sayHello xmlns:ns1="http://service.example.com/">
        <name>World</name>
    </ns1:sayHello>
</soap:Body>
```

**优点**：
- 符合 WS-I Basic Profile，**跨平台性好**
- 参数可被 XML Schema 严格校验
- 大多数现代语言（Java、C#、Python、Node.js）都能良好支持

**缺点**：
- 仍然受 RPC 包装约束（方法名在消息体中）

**适用场景**：与其他语言系统对接、遵循 WS-I 标准。

### 4.3 Document/literal（style="document", use="literal"）

**特点**：
- **没有 RPC 包装**，直接传输 XML 文档
- 方法通常只接收**一个请求对象**，返回**一个响应对象**
- 消息结构完全由 XML Schema 定义

**SOAP 请求示例**：
```xml
<soap:Body>
    <sayHelloRequest>
        <name>World</name>
    </sayHelloRequest>
</soap:Body>
```

**优点**：
- **最佳互操作性**，是 WebService 的"现代"标准
- 消息通过 XSD 严格验证，支持 `xsd:choice`、`xsd:sequence` 等复杂约束
- 可以用工具先生成 XML Schema，再生成代码（Contract-First）
- **适合复杂业务对象**的传输

**缺点**：
- 需要额外编写请求/响应的包装类
- Java 接口与方法签名的直观性稍差

**适用场景**：第三方对接、复杂业务、跨语言系统、正式生产环境。

### 4.4 对比总结

| 维度 | RPC/encoded | RPC/literal | Document/literal |
|------|-------------|-------------|------------------|
| WS-I BP 合规 | ❌ | ✅ | ✅ |
| 跨平台互操作 | ❌ | ✅ | ✅ |
| 消息可校验 | ❌ | ✅ | ✅ |
| 接口直观性 | ✅ | ✅ | ⚠️ 需包装类 |
| Axis 默认 | ✅ | ❌ | ❌ |
| 使用场景 | 遗留内部系统 | 跨语言对接 | **推荐的生产模式** |

---

## 5. 核心配置与安全

### 5.1 AxisConfig（AxisServlet 注册）

Spring Boot 嵌入式 Tomcat 没有 `WEB-INF/` 目录，Axis 默认加载 `server-config.wsdd` 的路径不可用。需要在 `@PostConstruct` 中将 WSDD 解压到临时目录。

```java
@Configuration
public class AxisConfig {

    @PostConstruct
    public void initAxisConfig() throws IOException {
        // 解压 server-config.wsdd 到临时目录
        Path tempDir = Files.createTempDirectory("axis-");
        Path wsddFile = tempDir.resolve("server-config.wsdd");
        Path workDir = tempDir.resolve("work");
        Files.createDirectories(workDir);

        try (InputStream is = new ClassPathResource("server-config.wsdd").getInputStream()) {
            Files.copy(is, wsddFile, StandardCopyOption.REPLACE_EXISTING);
        }

        // 设置 Axis 系统属性
        System.setProperty("axis.config", wsddFile.toAbsolutePath().toString());
        System.setProperty("axis.workDir", workDir.toAbsolutePath().toString());
    }

    @Bean
    public ServletRegistrationBean<AxisServlet> axisServlet() {
        ServletRegistrationBean<AxisServlet> bean = new ServletRegistrationBean<>();
        bean.setServlet(new AxisServlet());
        bean.addUrlMappings("/services/*");
        return bean;
    }

    @Bean
    public FilterRegistrationBean<AxisSecurityFilter> axisSecurityFilter() {
        FilterRegistrationBean<AxisSecurityFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new AxisSecurityFilter());
        bean.addUrlPatterns("/services/*");
        bean.setOrder(1);
        return bean;
    }
}
```

### 5.2 AxisSecurityFilter（禁用 AdminServlet）

Axis 1.4 自带的 `AdminServlet` 没有鉴权，默认可以通过 `/servlet/AdminServlet` 访问，暴露了运行时修改服务的能力。这是**已知安全风险**。

```java
public class AxisSecurityFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String uri = httpRequest.getRequestURI();

        // 拦截所有 Axis 管理路径
        if (uri.contains("AdminServlet") || uri.contains("/admin/") || uri.contains("/Admin")) {
            ((HttpServletResponse) response).sendError(HttpServletResponse.SC_FORBIDDEN,
                    "Access to Axis admin endpoints is forbidden");
            return;
        }

        chain.doFilter(request, response);
    }
}
```

### 5.3 Input Validation 输入校验

在服务实现中添加基本的 XSS 防护和参数校验：

```java
public class HelloWebServiceImpl implements HelloWebService {

    @Override
    public String sayHello(String name) {
        if (name == null || name.trim().isEmpty()) {
            return "Hello, Guest!";
        }
        // XSS 过滤
        String safeName = name.replaceAll("[<>&\"'\\\\]", "");
        return "Hello, " + safeName + "!";
    }

    @Override
    public User getUser(Long id) {
        if (id == null || id < 0) {
            return new User(0L, "Unknown", "unknown@example.com");
        }
        return new User(id, "User-" + id, "user" + id + "@example.com");
    }
}
```

### 5.4 AxisStartupLogger（启动日志）

```java
@Component
public class AxisStartupLogger {

    private static final Logger log = LoggerFactory.getLogger(AxisStartupLogger.class);

    @Value("${server.port:8080}")
    private int port;

    @EventListener(ApplicationReadyEvent.class)
    public void onApplicationReady() {
        String base = "http://localhost:" + port;

        log.info("=======================================");
        log.info("Axis WebService started");
        log.info("=======================================");
        log.info("(1) RPC/encoded      : {}/services/HelloWebService?wsdl", base);
        log.info("(2) RPC/literal      : {}/services/HelloRpcLiteralService?wsdl", base);
        log.info("(3) Document/literal : {}/services/HelloDocLiteralService?wsdl", base);
        log.info("=======================================");
    }
}
```

---

## 6. 实现案例代码

### 6.1 RPC/encoded 模式

接口 — 方法签名直观，参数平铺：

```java
public interface HelloWebService {
    String sayHello(String name);
    User getUser(Long id);
}
```

实现 — 继承接口，**不需要任何 Axis 注解**：

```java
public class HelloWebServiceImpl implements HelloWebService {
    @Override
    public String sayHello(String name) {
        if (name == null || name.trim().isEmpty()) {
            return "Hello, Guest! Welcome to Axis WebService.";
        }
        String safeName = name.replaceAll("[<>&\"'\\\\]", "");
        return "Hello, " + safeName + "! Welcome to Axis WebService.";
    }

    @Override
    public User getUser(Long id) {
        if (id == null || id < 0) {
            return new User(0L, "Unknown", "unknown@example.com");
        }
        return new User(id, "User-" + id, "user" + id + "@example.com");
    }
}
```

### 6.2 RPC/literal 模式

接口与 RPC/encoded 完全一致，差异仅在 `server-config.wsdd` 中的 `use="literal"`：

```java
public interface HelloRpcLiteralService {
    String sayHello(String name);
    User getUser(Long id);
}
```

### 6.3 Document/literal 模式

**这是推荐的生产模式** —— 每个操作需要请求/响应的包装类：

```java
// 请求包装
public class SayHelloRequest implements Serializable {
    private String name;
    // getter/setter...
}

// 响应包装
public class SayHelloResponse implements Serializable {
    private String message;
    // getter/setter...
}

// 请求包装
public class GetUserRequest implements Serializable {
    private Long id;
    // getter/setter...
}

// 响应包装
public class GetUserResponse implements Serializable {
    private User user;
    // getter/setter...
}
```

接口 — 参数不再是平铺的，而是包装对象：

```java
public interface HelloDocLiteralService {
    SayHelloResponse sayHello(SayHelloRequest request);
    GetUserResponse getUser(GetUserRequest request);
}
```

实现：

```java
public class HelloDocLiteralServiceImpl implements HelloDocLiteralService {

    @Override
    public SayHelloResponse sayHello(SayHelloRequest request) {
        if (request == null || request.getName() == null || request.getName().trim().isEmpty()) {
            return new SayHelloResponse("Hello, Guest! Welcome to Axis Document/Literal WebService.");
        }
        String safeName = request.getName().replaceAll("[<>&\"'\\\\]", "");
        return new SayHelloResponse("Hello, " + safeName + "! Welcome to Axis Document/Literal WebService.");
    }

    @Override
    public GetUserResponse getUser(GetUserRequest request) {
        if (request == null || request.getId() == null || request.getId() < 0) {
            return new GetUserResponse(new User(0L, "Unknown", "unknown@example.com"));
        }
        Long id = request.getId();
        return new GetUserResponse(new User(id, "User-" + id, "user" + id + "@example.com"));
    }
}
```

### 6.4 公共模型 User

所有模式共享的 `User.java`，实现 `Serializable`：

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    private Long id;
    private String name;
    private String email;

    public User() {}
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    // getter/setter...
}
```

---

## 7. Security Patches（安全补丁）

### 7.1 为什么要打补丁？

Apache Axis 1.4 自 2006 年发布后**官方不再更新**，存在以下已知漏洞：

| CVE | 漏洞类型 | 受影响的类 | 风险 |
|-----|---------|-----------|------|
| **CVE-2023-40743** | JNDI 注入 | `ServiceFactory.java` | 攻击者可利用 `getService()` 中的 JNDI 查找执行远程代码 |
| **CVE-2012-5784** | SSL 主机名未验证 | `JSSESocketFactory.java` | MITM 攻击（中间人攻击） |
| **CVE-2014-3596** | SSL 主机名验证不完整 | `JSSESocketFactory.java` | 通配符证书校验绕过 |

### 7.2 补丁实现

**CVE-2023-40743 补丁**（`ServiceFactory.java`）：

在 `getService()` 方法中，获取 JNDI name 后检查危险协议：

```
private static boolean isUnsupportedJndiProtocol(String name) {
    if (name == null) return false;
    String upper = name.toUpperCase();
    return upper.contains("LDAP")
        || upper.contains("RMI")
        || upper.contains("JMS")
        || upper.contains("JMX")
        || upper.contains("JRMP")
        || upper.contains("DNS")
        || upper.contains("IIOP")
        || upper.contains("CORBANAME");
}
```

**CVE-2012-5784 / CVE-2014-3596 补丁**（`JSSESocketFactory.java`）：

在 `create()` 方法结尾添加主机名校验：

```java
verifyHostName(host, (SSLSocket) sslSocket);
```

实现了完整的 `verifyHostName()` 逻辑，包括：
- 比对 Subject CN
- 遍历 Subject Alternative Names (DNS entries)
- 通配符证书校验
- `.com` / `.cn` 等二级国家代码通配符限制

### 7.3 补丁构建

项目提供了 `patch-axis.bat` 一键构建脚本：

```batch
patch-axis.bat
```

流程：下载官方 `axis-1.4.jar` → 替换补丁源码 → 编译 → 打包 → 安装到本地 Maven 仓库。

输出物：
- 安装到 `~/.m2/repository/org/apache/axis/axis/1.4-patched/`
- 同时复制到 `lib/axis-1.4-patched.jar` 作为 flatDir 回退

> 具体详见[SpringBoot-204-axis-patched]()

---

## 8. 构建与运行

### 8.1 前置要求

- JDK 1.8（必须，JDK 17+ 不兼容 Axis 1.4）
- 环境变量 `JAVA_HOME` 指向 JDK 8

### 8.2 构建

```batch
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_341
gradlew.bat build
```

### 8.3 运行

```batch
gradlew.bat bootRun
```

启动后日志输出：

```
=======================================
Axis WebService started
=======================================
(1) RPC/encoded      : http://localhost:8080/services/HelloWebService?wsdl
(2) RPC/literal      : http://localhost:8080/services/HelloRpcLiteralService?wsdl
(3) Document/literal : http://localhost:8080/services/HelloDocLiteralService?wsdl
=======================================
```

### 8.4 验证

打开浏览器访问 WSDL 地址：
- `http://localhost:8080/services/HelloWebService?wsdl`
- `http://localhost:8080/services/HelloRpcLiteralService?wsdl`
- `http://localhost:8080/services/HelloDocLiteralService?wsdl`

---

## 9. 客户端调用示例

### 9.1 使用 Axis 原生客户端（RPC/encoded）

```
Service service = new Service();
Call call = (Call) service.createCall();
call.setTargetEndpointAddress("http://localhost:8080/services/HelloWebService");
call.setOperationName(new QName("example", "sayHello"));

String result = (String) call.invoke(new Object[] { "World" });
System.out.println(result);  // Hello, World! ...
```

### 9.2 使用 SOAP 请求（HTTP Client 直接调用）

```
POST /services/HelloDocLiteralService HTTP/1.1
Content-Type: text/xml; charset=utf-8
SOAPAction: "sayHello"

```
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <sayHelloRequest>
            <name>World</name>
        </sayHelloRequest>
    </soap:Body>
</soap:Envelope>
```

---

## 附录：常见问题

**Q: 为什么用 patched 版本包名还是 `org.apache.axis`？**
A: 补丁只改了两个类的实现逻辑，未改包名，因此完全兼容所有依赖 `org.apache.axis` 的代码。

**Q: 可以在同一项目中同时部署三种模式吗？**
A: 可以。每个服务在 `server-config.wsdd` 中声明自己的 `style` 和 `use`，Axis 引擎会为每个请求独立处理。

**Q: `allowedMethods` 设为 `*` 有什么风险？**
A: 会暴露 Axis 内部所有方法，包括 `AdminServlet` 的管理操作（如动态取消部署服务），应始终显式列出。
 