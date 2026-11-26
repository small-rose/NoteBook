---
layout: default
title: SB3-OSGi
parent: SpringBoot3
has_children: false
nav_order: 1007
last_modified_date: 2026-11-27
---

Here are the logging system changes and practices on SpringBoot3 .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }

# Spring Boot 3.5 多模块 OSGi 热插拔组件实战指南

> Spring Boot 3.5 + Apache Felix 热插拔组件

---

## 一、什么是OSGi

OSGi（Open Services Gateway Initiative）是一个面向 Java 的动态模块化系统。

它定义了模块化应用的开发、部署和管理标准，允许在运行时动态地安装、启动、停止、更新和卸载模块（称为 Bundle）。

OSGi 联盟最初成立于1999年，旨在为嵌入式设备和网关提供服务平台，后来广泛应用于企业级应用、IDE（如 Eclipse）、应用服务器等领域。

核心思想：将应用程序拆分为多个独立的、可热插拔的模块，每个模块拥有自己的类加载器，严格管理依赖和可见性，从而避免传统的类路径混乱和版本冲突问题。

## 二、OSGi 运行机制与原理

### 2.1 模块化层（Module Layer）

Bundle：OSGi 的基本模块单元，本质上是一个包含 MANIFEST.MF 文件的 JAR 包。MANIFEST.MF 描述了该 Bundle 的元数据，包括：

- Bundle-SymbolicName：唯一标识符
- Bundle-Version：版本号
- Import-Package：需要导入的外部包（及其版本范围）
- Export-Package：对外导出的包
- Bundle-Activator：激活器类（实现 BundleActivator 接口）

类加载机制：每个 Bundle 拥有独立的类加载器，遵循 委托模型：

- 首先在自己的私有包中查找
- 其次在显式导入的包中查找（由框架协调）
- 最后委派给父类加载器（如 JDK 核心类）

### 2.2 生命周期层（Lifecycle Layer）

OSGi 定义了 Bundle 的六个状态转换：

```text
INSTALLED → RESOLVED → STARTING → ACTIVE
                                      ↓
                                    STOPPING → RESOLVED
                                      ↓
                                   UNINSTALLED
```

- INSTALLED：Bundle 已安装但未解析依赖
- RESOLVED：依赖满足，可启动
- STARTING：正在执行 BundleActivator.start() 方法
- ACTIVE：正常运行
- STOPPING：正在执行 BundleActivator.stop() 方法
- UNINSTALLED：已卸载

生命周期管理 API（Bundle 接口）提供了 start()、stop()、update()、uninstall() 等方法，这些操作可以在运行时动态执行，从而实现热插拔。

### 2.3 服务层（Service Layer）

OSGi 服务模型是基于 `发布-发现-绑定` 模式的轻量级服务注册表：

- 服务注册：Bundle 通过 BundleContext.registerService() 将自己的服务对象注册到框架
- 服务发现：其他 Bundle 通过 BundleContext.getServiceReference() 或 ServiceTracker 获取服务引用
- 服务监听：使用 ServiceListener 或 ServiceTracker 动态跟踪服务的注册与注销
- 服务属性：可以为服务附加元数据（如版本、区域等），用于筛选

### 2.4 安全层（Security Layer）

基于 Java 安全模型，通过 Permission 控制 Bundle 对包、服务、资源的访问权限。开发环境中通常不启用。


## 三、常见应用场景

|场景 |说明|
|---|---|
|插件化架构 | IDE（Eclipse）、富客户端应用，允许第三方开发者贡献插件|
|微服务降级/热更新 | 在不重启主进程的情况下更新业务模块，实现零停机维护|
|物联网网关 | 动态加载不同设备协议驱动|
|企业级应用模块化 | 大型单体应用拆分为可独立部署的 Bundle，降低耦合|
|动态规则引擎 | 热加载决策规则、算法插件|
|多租户定制 | 根据不同租户需求加载不同功能 Bundle|


## 四、实战案例

Spring Boot 3.5 + Apache Felix 热插拔组件

### 4.1 案例目标

- 构建一个 Spring Boot 3.5 宿主应用，内嵌 Apache Felix OSGi 框架
- 提供一个简单的 GreetingService 接口
- 开发一个 OSGi Bundle 插件，实现该接口并注册为 OSGi 服务
- 宿主应用通过 REST API 动态管理 Bundle（安装/启动/停止/卸载）
- 提供另一个 REST 端点调用 OSGi 服务，验证热插拔效果

### 4.2 技术栈

- Spring Boot: 3.5.0（假设已发布，实际可使用 3.4.x 替代，原理相同）
- Apache Felix: 7.0.5
- Maven: 3.9+
- JDK: 17+

### 4.3 项目结构

多模块Maven父子工程。

```code
spring-boot-osgi-demo/
├── pom.xml                      # 父 POM
├── common-api/                  # 公共接口模块（打包为普通 jar，供宿主和 Bundle 引用）
│   ├── pom.xml
│   └── src/main/java/com/example/api/
│       └── GreetingService.java
├── host-app/                    # Spring Boot 宿主应用
│   ├── pom.xml
│   └── src/main/java/com/example/host/
│       ├── HostApplication.java
│       ├── config/FelixConfig.java
│       ├── controller/BundleController.java
│       └── controller/GreetController.java
└── greeting-plugin/             # OSGi Bundle 插件
    ├── pom.xml
    └── src/main/java/com/example/plugin/
        ├── Activator.java
        └── GreetingServiceImpl.java
```

### 4.4 完整代码

4.4.1 父 POM (pom.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-boot-osgi-demo</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>common-api</module>
        <module>host-app</module>
        <module>greeting-plugin</module>
    </modules>

    <properties>
        <java.version>17</java.version>
        <spring-boot.version>3.5.0</spring-boot.version>
        <felix.version>7.0.5</felix.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- 公共接口模块 -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>common-api</artifactId>
                <version>${project.version}</version>
            </dependency>
            <!-- Apache Felix -->
            <dependency>
                <groupId>org.apache.felix</groupId>
                <artifactId>org.apache.felix.framework</artifactId>
                <version>${felix.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.felix</groupId>
                <artifactId>org.apache.felix.main</artifactId>
                <version>${felix.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

4.4.2 公共接口模块 (common-api)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-boot-osgi-demo</artifactId>
        <groupId>com.example</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>common-api</artifactId>
    <packaging>jar</packaging>
</project>
```

GreetingService.java

```java
package com.example.api;

public interface GreetingService {
    String greet(String name);
}
```

4.4.3 宿主应用模块 (host-app)

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-boot-osgi-demo</artifactId>
        <groupId>com.example</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>host-app</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.felix</groupId>
            <artifactId>org.apache.felix.framework</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.felix</groupId>
            <artifactId>org.apache.felix.main</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

HostApplication.java

```java
package com.example.host;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HostApplication {
    public static void main(String[] args) {
        SpringApplication.run(HostApplication.class, args);
    }
}
```

FelixConfig.java（OSGi 框架配置与生命周期管理）

```java
package com.example.host.config;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.apache.felix.framework.Felix;
import org.apache.felix.main.AutoProcessor;
import org.osgi.framework.BundleContext;
import org.osgi.framework.Constants;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class FelixConfig {

    private static final Logger log = LoggerFactory.getLogger(FelixConfig.class);
    private Felix felix;

    @PostConstruct
    public void init() throws Exception {
        Map<String, Object> configMap = new HashMap<>();
        configMap.put(Constants.FRAMEWORK_STORAGE, "felix-cache");
        configMap.put(AutoProcessor.AUTO_DEPLOY_DIR_PROPERTY, "deploy");
        configMap.put(AutoProcessor.AUTO_DEPLOY_ACTION_PROPERTY,
                AutoProcessor.AUTO_DEPLOY_INSTALL | AutoProcessor.AUTO_DEPLOY_START);

        felix = new Felix(configMap);
        felix.init();
        felix.start();
        log.info("Felix OSGi framework started successfully.");
    }

    @PreDestroy
    public void destroy() throws Exception {
        if (felix != null) {
            felix.stop();
            felix.waitForStop(5000);
            log.info("Felix OSGi framework stopped.");
        }
    }

    @Bean
    public BundleContext bundleContext() {
        return felix.getBundleContext();
    }
}
```

BundleController.java（Bundle 管理 REST API）

```java
package com.example.host.controller;

import org.osgi.framework.Bundle;
import org.osgi.framework.BundleContext;
import org.osgi.framework.BundleException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/osgi")
public class BundleController {

    @Autowired
    private BundleContext bundleContext;

    @GetMapping("/bundles")
    public List<String> listBundles() {
        return Arrays.stream(bundleContext.getBundles())
                .map(b -> b.getBundleId() + " - " + b.getSymbolicName() + " [" + stateToString(b.getState()) + "]")
                .collect(Collectors.toList());
    }

    @PostMapping("/install")
    public String installBundle(@RequestParam("file") MultipartFile file) throws IOException, BundleException {
        java.io.File tmpFile = java.io.File.createTempFile("bundle", ".jar");
        try (FileOutputStream fos = new FileOutputStream(tmpFile)) {
            fos.write(file.getBytes());
        }
        Bundle bundle = bundleContext.installBundle("file:" + tmpFile.getAbsolutePath());
        bundle.start();
        return "Installed bundle: " + bundle.getSymbolicName() + " (ID: " + bundle.getBundleId() + ")";
    }

    @PostMapping("/start/{bundleId}")
    public String startBundle(@PathVariable long bundleId) throws BundleException {
        Bundle bundle = bundleContext.getBundle(bundleId);
        if (bundle == null) return "Bundle not found";
        bundle.start();
        return "Started: " + bundle.getSymbolicName();
    }

    @PostMapping("/stop/{bundleId}")
    public String stopBundle(@PathVariable long bundleId) throws BundleException {
        Bundle bundle = bundleContext.getBundle(bundleId);
        if (bundle == null) return "Bundle not found";
        bundle.stop();
        return "Stopped: " + bundle.getSymbolicName();
    }

    @DeleteMapping("/uninstall/{bundleId}")
    public String uninstallBundle(@PathVariable long bundleId) throws BundleException {
        Bundle bundle = bundleContext.getBundle(bundleId);
        if (bundle == null) return "Bundle not found";
        bundle.uninstall();
        return "Uninstalled: " + bundle.getSymbolicName();
    }

    private String stateToString(int state) {
        return switch (state) {
            case Bundle.UNINSTALLED -> "UNINSTALLED";
            case Bundle.INSTALLED -> "INSTALLED";
            case Bundle.RESOLVED -> "RESOLVED";
            case Bundle.STARTING -> "STARTING";
            case Bundle.STOPPING -> "STOPPING";
            case Bundle.ACTIVE -> "ACTIVE";
            default -> "UNKNOWN";
        };
    }
}
```

GreetController.java（调用 OSGi 服务）

```java
package com.example.host.controller;

import com.example.api.GreetingService;
import org.osgi.framework.BundleContext;
import org.osgi.framework.ServiceReference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetController {

    @Autowired
    private BundleContext bundleContext;

    @GetMapping("/greet")
    public String greet(@RequestParam(defaultValue = "World") String name) {
        ServiceReference<GreetingService> ref =
                bundleContext.getServiceReference(GreetingService.class);
        if (ref == null) {
            return "No GreetingService available. Please install the plugin bundle first.";
        }
        GreetingService service = bundleContext.getService(ref);
        try {
            return service.greet(name);
        } finally {
            bundleContext.ungetService(ref);
        }
    }
}
```

4.4.4 OSGi Bundle 插件模块 (greeting-plugin)

pom.xml（注意打包方式为 bundle）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-boot-osgi-demo</artifactId>
        <groupId>com.example</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>greeting-plugin</artifactId>
    <packaging>bundle</packaging>

    <dependencies>
        <dependency>
            <groupId>org.apache.felix</groupId>
            <artifactId>org.osgi.core</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-api</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.felix</groupId>
                <artifactId>maven-bundle-plugin</artifactId>
                <version>5.1.9</version>
                <extensions>true</extensions>
                <configuration>
                    <instructions>
                        <Bundle-SymbolicName>${project.groupId}.${project.artifactId}</Bundle-SymbolicName>
                        <Bundle-Activator>com.example.plugin.Activator</Bundle-Activator>
                        <Import-Package>com.example.api, org.osgi.framework</Import-Package>
                        <Export-Package></Export-Package>
                    </instructions>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

GreetingServiceImpl.java

```java
package com.example.plugin;

import com.example.api.GreetingService;

public class GreetingServiceImpl implements GreetingService {
    @Override
    public String greet(String name) {
        return "Hello from OSGi Plugin! Welcome, " + name + ".";
    }
}
```

Activator.java

```java
package com.example.plugin;

import com.example.api.GreetingService;
import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.osgi.framework.ServiceRegistration;

public class Activator implements BundleActivator {

    private ServiceRegistration<?> registration;

    @Override
    public void start(BundleContext context) throws Exception {
        GreetingService service = new GreetingServiceImpl();
        registration = context.registerService(
                GreetingService.class.getName(),
                service,
                null
        );
        System.out.println("GreetingPlugin started and service registered.");
    }

    @Override
    public void stop(BundleContext context) throws Exception {
        if (registration != null) {
            registration.unregister();
        }
        System.out.println("GreetingPlugin stopped.");
    }
}
```

### 4.5 测试热插拔

4.5.1 查看当前 Bundle 列表

使用 GET 方法调用 http://localhost:8080/osgi/bundles

返回系统 Bundle 和宿主自身。

4.5.2 调用 Greet 接口（无插件）

使用 GET 方法调用 http://localhost:8080/greet?name=Tom

返回：No GreetingService available. Please install the plugin bundle first.

4.5.3 安装插件 Bundle

curl -F "file=@../greeting-plugin/target/greeting-plugin-1.0.0.jar" \

http://localhost:8080/osgi/install

返回：Installed bundle: com.example.greeting-plugin (ID: 5)

4.5.4 再次调用 Greet 接口

GET http://localhost:8080/greet?name=Tom

返回：Hello from OSGi Plugin! Welcome, Tom.

4.5.5 停止插件

POST http://localhost:8080/osgi/stop/5

再次调用 Greet → 返回无服务。

4.5.6 卸载插件

DELETE http://localhost:8080/osgi/uninstall/5

彻底移除。

## 五. 安全问题

5.1 OSGi 安全依赖

OSGi 安全层的实现依赖 java.lang.SecurityManager + AccessController + Permission 体系。而 SecurityManager 的命运是这样的：

|JDK 版本 |状态 |影响|
|----|----|-----|
|≤ 16 |可用	|OSGi 安全层正常工作|
|17	|JEP 411 标记为 deprecated for removal	|启动时打印警告，功能仍在|
|18–23 |弃用持续，逐步收紧	|需显式 -Djava.security.manager=allow|
|24+ |JEP 486 永久禁用	|System.setSecurityManager() 直接抛 UnsupportedOperationException，连 allow 都不认|

Spring Boot 3.5 基线是 JDK 17，实际项目普遍跑 21/25。也就是说：

- JDK 17/21 上：理论上还能开，但要背弃用警告，且随时会炸

- JDK 24+ 上：开了也没用，ConditionalPermissionAdmin 服务还在，但底层没有 SecurityManager 执行检查，权限判定等于空转

Felix 7.0.5 的 security 代码路径同样依赖这套东西，Equinox 那边已经公开宣布放弃 SecurityManager 支持。

历史jdk版本场景：

- 电信网关、车载 OSGi 平台（BMW 的 JOBS、部分 Android Auto 方案）
- 需要加载第三方不可信 Bundle 的插件市场
- 早期 Eclipse RCP 的企业客户端

共同点：插件来源不可信，必须防住“插件 A 反射读插件 B 的私有字段”“插件偷偷读 ~/.ssh/id_rsa”这类越权。

而绝大多数企业内部插件平台，插件都是自家团队写的，威胁模型完全不同——这时候开安全层属于典型的“安全表演”，成本极高收益为零。


5.2 OSGi 安全层要解决的两件事

A: Bundle 间越权访问 → 靠模块化层（仍然有效）

Import-Package / Export-Package 的可见性约束不依赖 SecurityManager，是类加载器层面强制的。这才是 OSGi 最有价值的部分：

```xml
<!-- 只导出必要包，其余一律不导出 -->
<Export-Package>com.example.api</Export-Package>
<Private-Package>com.example.plugin.internal.*</Private-Package>
```

插件拿不到没导出的类就不可见。

B: 插件恶意行为（读文件、开 socket、执行命令）→ 靠进程/容器隔离

|手段|说明|
|---|---|
|独立进程 + 容器 | 插件跑在 sidecar 容器里，只暴露 gRPC/HTTP 接口，天然沙箱|
|Bundle 签名校验 | 安装前验 SHA-256 + 证书链，只信任自家签发的 jar|
|白名单 ClassLoader | 自定义 ClassLoader 拦截 Runtime.exec、FileInputStream 等敏感类的加载|
|Seccomp / gVisor | 容器层限制 syscall，比 JVM 内沙箱可靠得多|


5.3 其他建议

- FelixConfig 里不要碰 `org.osgi.framework.security` 相关属性，设了在 JDK 24 上会启动失败。
- installBundle 接口必须加校验，这是你这个案例真正的安全缺口——现在任何人 POST 一个 jar 就能在宿主 JVM 里跑任意代码：

```java
private static final Set<String> ALLOWED_SIGNERS = Set.of("CN=YourCompany, O=YourOrg");

private void verifyBundleSignature(File jar) {
    // 校验 jar 签名证书链，不在白名单直接拒绝
    // 或用 SHA-256 摘要比对预置清单
}
```
- 生产部署建议：宿主只做 Bundle 生命周期编排，真正执行业务逻辑的 Bundle 放到独立进程，通过 IPC 通信。这样插件崩了、被投毒了，宿主不受影响。


## 六、总结

- OSGi 提供了一种成熟的 Java 模块化方案，支持动态热插拔。
- 通过 Apache Felix 内嵌到 Spring Boot 应用中，可以轻松实现插件化架构。
- 本案例展示了完整的 Maven 多模块项目结构，宿主与 Bundle 通过公共接口模块解耦。
- 实际生产环境中，还可以结合 OSGi 服务追踪器（ServiceTracker）实现更优雅的服务动态绑定，或使用声明式服务（DS）简化 Bundle 开发。

扩展建议：

- 将 common-api 单独发布到仓库，供第三方开发者编写插件。
- 使用 Spring 的 @EventListener 监听 OSGi 服务事件，实现自动注入。
- 考虑使用 Blueprint 或 Declarative Services 进一步简化开发。
