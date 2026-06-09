---
layout: default
title: SB3-CXF
parent: SpringBoot3
has_children: false
nav_order: 1003
---


 Here are webservices of CXF used examples on springboot3 .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }


### 一 Apache CXF

Apache CXF 是 Apache 软件基金会的一个开源 Web 服务框架，由 Celtix 和 XFire 合并而成。

它主要用于构建和开发 SOAP 和 RESTful 风格的 Web 服务，并可与 Spring 等框架无缝集成。

核心功能与特点

多协议支持：全面支持 SOAP、RESTful HTTP、XML/HTTP 等多种通信协议。

规范支持：全面支持 JAX-WS（用于 SOAP）和 JAX-RS（用于 REST）规范。

开发模式灵活：支持代码优先（Code First，从 Java 类生成服务）和 WSDL 优先（WSDL First，从 WSDL 定义生成 Java 代码）两种模式。

数据绑定：内置 XML 和 JSON 数据绑定，并支持 WS-Security（服务安全认证）、WS-Addressing 等高级特性。

拦截器机制：提供强大的拦截器（Interceptor）功能，允许在请求处理的前后阶段进行自定义逻辑干预（如日志记录、安全校验等）。

官网：[https://cxf.apache.org/](https://cxf.apache.org/)



### SEI和SIB 


在 Web 服务（特别是 Java 的 JAX-WS 规范与 Apache CXF 框架）中，SEI 是 Service Endpoint Interface（服务端点接口） 的缩写, SIB（Service Implementation Bean，服务实现豆/类） 


核心作用与概念映射

分离契约与实现：SEI 负责定义业务方法的契约（方法名、参数、返回值），而具体的业务逻辑由 SIB （服务实现豆/类） 来编写。

对应 WSDL 元素：在 WSDL（Web 服务描述语言）契约中，SEI 直接映射为 wsdl:portType 元素，而 SEI 中定义的方法则映射为 wsdl:operation 元素。


两种主要的开发模式

（1）代码优先（Java-First / Bottom-Up）：你先编写一个 Java 接口（SEI）并加上 @WebService 注解，Apache CXF 会根据这个接口自动生成 WSDL 契约文件。（可理解为服务端的开发。）

（2）契约优先（WSDL-First / Top-Down）：你已经有了现成的 WSDL 文件，利用 Apache CXF 的 wsdl2java 工具，可以自动为你生成对应的 Java SEI 接口和相关的实体类。（可理解为客户端的开发）





### 使用案例


在springboot 3.5.14中使用

```
    implementation("org.apache.cxf:cxf-spring-boot-starter-jaxws:4.1.6")
    implementation('jakarta.activation:jakarta.activation-api:2.1.4')
    implementation('jakarta.xml.bind:jakarta.xml.bind-api:4.0.4')
    implementation('org.glassfish.jaxb:jaxb-runtime:4.0.8')
    implementation('org.glassfish.jaxb:jaxb-core:4.0.6')
    implementation('org.glassfish.jaxb:jaxb-xjc:4.0.6')
    implementation('org.glassfish.jaxb:txw2:4.0.6')
```

作为客户端案例里的依赖如下：

```
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.5.14'
	id 'io.spring.dependency-management' version '1.1.7'
    // 1. 引入专门用于执行 wsdl2java 的 Gradle 插件
    id 'io.mateo.cxf-codegen' version '2.5.0'
}

import io.mateo.cxf.codegen.wsdl2java.Wsdl2Java

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
    archiveClassifier = ''
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories  {
	mavenLocal()
	mavenCentral()
    //maven = {
        //allowInsecureProtocol = true
        //url  "http://192.168.10.138:9090/nexus/content/repositories/fnd-release"
    //}
    //maven = {
        //allowInsecureProtocol = true
        //url = "http://192.168.10.138:9090/nexus/content/repositories/thirdpart"
    //}
}

// 全局拦截老旧日志组件，防止与spring-jcL冲突
configurations.all{
    exclude group: 'commons-logging', module: 'commons-logging'
    //可选：拦截已废弃的javax包，强制走jakarta
    exclude group: 'javax.servlet', module: 'javax.servlet-api'
}


dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-actuator'
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-data-redis'
	implementation 'org.springframework.boot:spring-boot-starter-jdbc'
	implementation 'org.springframework.boot:spring-boot-starter-mail'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-validation'
    //implementation 'jakarta.persistence:jakarta.persistence-api:3.1.0'
    implementation ('org.springframework.boot:spring-boot-starter-web')
    implementation ('org.springframework.boot:spring-boot-starter-tomcat')
	implementation 'org.springframework.session:spring-session-data-redis'
    implementation 'org.springframework.session:spring-session-jdbc'
    implementation("com.alibaba:druid-spring-boot-3-starter:1.2.28")// 需要排除漏洞组件

    // 与boot3兼容的swagger-ui
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.17")
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	implementation('com.oracle.database.jdbc:ojdbc11')
    implementation('com.h2database:h2')
	annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'


	implementation("org.jsoup:jsoup:1.21.2")
	// Selenium WebDriver -->
	implementation 'org.seleniumhq.selenium:selenium-java:4.38.0'
	implementation 'io.github.bonigarcia:webdrivermanager:5.6.0'
	implementation 'commons-io:commons-io:2.11.0'
	implementation 'com.alibaba:fastjson:1.2.79'

	implementation("cn.hutool:hutool-core:5.8.42")
    implementation("org.apache.cxf:cxf-spring-boot-starter-jaxws:4.1.6")
    implementation('jakarta.activation:jakarta.activation-api:2.1.4')
    implementation('jakarta.xml.bind:jakarta.xml.bind-api:4.0.4')
    implementation('org.glassfish.jaxb:jaxb-runtime:4.0.8')
    implementation('org.glassfish.jaxb:jaxb-core:4.0.6')
    implementation('org.glassfish.jaxb:jaxb-xjc:4.0.6')
    implementation('org.glassfish.jaxb:txw2:4.0.6')

	/*implementation files(
	        // boot2 支持的宝蓝德
			'libs/bes-actuator-spring-boot-2.x-starter-9.5.2.017.jar',
			'libs/bes-lite-spring-boot-2.x-starter-9.5.2.017.jar',
			'libs/bes-websocket-9.5.2.017.jar'
	)*/
    //宝兰德引l用包宝兰德永久License：bes.Lic.txt切记上生产需要替换
    //implementation('com.bes.besstarter:bes-lite-spring-boot-starter:11.5.0.007') ;
    //implementation('com.bes.besstarter:bes-actuator-spring-boot-starter:11.5.0.0o7')

}

tasks.named('test') {
	useJUnitPlatform()
}

tasks.withType(JavaCompile) {
	options.encoding = 'UTF-8'
}

// ✅ 插件自动把生成目录加入 main source set，无需手写 sourceSets 块
tasks.register('generateCommonService', Wsdl2Java) {
    group = 'build'
    description = '根据 WSDL 生成 Java 客户端代码（CXF wsdl2java）'

    toolOptions {
        // ✅ WSDL 已落盘为 src/main/wsdl/commonService.wsdl，构建不再依赖运行中的服务
        //    注意：wsdl 属性是 String，不是 File，必须用 .absolutePath 转一次
        wsdl = file('src/main/wsdl/commonService.wsdl').absolutePath
        // 目标包名（注意是 packageNames 复数 + List；不是 -p 的单数 String）
        // ✅ 放到 stub 子包，避免与手写的 com.small.rose.demo.modules.webservices.CommonService（服务端 SEI）冲突
        packageNames = ['com.small.rose.demo.modules.webservices.stub']
        // 输出目录
        outputDir = layout.buildDirectory.dir('generated/sources/cxf/WS').get().asFile
        // -client 放 extraArgs（插件没有为它设强类型属性）
        //extraArgs.addAll(['-client'])
        extraArgs.addAll(['-encoding', 'UTF-8'])
    }

    // Wsdl2Java 继承 JavaExec，JVM 参数用 allJvmArgs
    allJvmArgs = [
        '--add-opens',  'java.base/java.lang=ALL-UNNAMED',
        '--add-opens',  'java.base/java.util=ALL-UNNAMED',
        '--add-opens',  'java.base/java.io=ALL-UNNAMED',
        '--add-opens',  'java.base/java.net=ALL-UNNAMED',
        '--add-opens',  'java.xml/javax.xml.namespace=ALL-UNNAMED',
        '--add-exports','java.xml/com.sun.org.apache.xerces.internal.xni.parser=ALL-UNNAMED',
        '--add-exports','java.xml/com.sun.org.apache.xerces.internal.util=ALL-UNNAMED'
    ]
}

// ✅ 编译前自动跑所有 Wsdl2Java 任务（用 withType 兜底，未来加多个 WSDL 也无须再改）
compileJava.dependsOn tasks.withType(Wsdl2Java)
```

其中注册 generateCommonService 是为了根据WSDL 契约文件 生成SEI 作为客户端来调用服务端代码。



问题一：多 WSDL 配置

两种写法，按需选：

> 这2种写法都是可以根据WSDL生成SEI文件作为编译文件输出到编译目录，在最终生成jar的时候会跟随编译目录打进jar包里。

写法 A：逐个注册（简单直接，适合 2-3 个）
```
tasks.register('generateCommonService', Wsdl2Java) {
    toolOptions {
        wsdl = file('src/main/wsdl/commonService.wsdl').absolutePath
        packageNames = ['com.small.rose.demo.modules.webservices.stub.common']
        outputDir = layout.buildDirectory.dir('generated/sources/cxf/CommonService').get().asFile
        extraArgs.addAll(['-encoding', 'UTF-8'])
    }
}

tasks.register('generateOrderService', Wsdl2Java) {
    toolOptions {
        wsdl = file('src/main/wsdl/orderService.wsdl').absolutePath
        packageNames = ['com.small.rose.demo.modules.webservices.stub.order']
        outputDir = layout.buildDirectory.dir('generated/sources/cxf/OrderService').get().asFile
        extraArgs.addAll(['-encoding', 'UTF-8'])
    }
}

compileJava.dependsOn tasks.withType(Wsdl2Java)
```

写法 B：Map + 循环（适合 N 个，统一管理）
```
def services = [
    common: [file: 'commonService.wsdl', pkg: 'com.small.rose.demo.modules.webservices.stub.common'],
    order:  [file: 'orderService.wsdl',  pkg: 'com.small.rose.demo.modules.webservices.stub.order'],
]

services.each { name, cfg ->
    tasks.register("generate${name.capitalize()}Service", Wsdl2Java) {
        toolOptions {
            wsdl = file("src/main/wsdl/${cfg.file}").absolutePath
            packageNames = [cfg.pkg]
            outputDir = layout.buildDirectory.dir("generated/sources/cxf/${name.capitalize()}").get().asFile
            extraArgs.addAll(['-encoding', 'UTF-8'])
        }
    }
}

compileJava.dependsOn tasks.withType(Wsdl2Java)
```

问题二：独立 JAR 包

> 独立jar包的意思是独立管理。

方案 A：独立子模块（最标准，推荐）

```
project-root/
├── build.gradle                  ← 主应用
├── settings.gradle               ← 声明子模块
├── src/main/java/                ← 业务代码
│
└── stubs/                        ← 独立模块目录
    ├── common-service/
    │   ├── build.gradle          ← 只含 WSDL 生成
    │   └── src/main/wsdl/
    │       └── commonService.wsdl
    └── order-service/
        ├── build.gradle
        └── src/main/wsdl/
            └── orderService.wsdl
```

settings.gradle 加入：

```
include 'stubs:common-service'
include 'stubs:order-service'
```

stubs/common-service/build.gradle：
```
plugins {
    id 'java-library'             // 对外暴露 API，而非可执行 JAR
    id 'io.mateo.cxf-codegen' version '2.5.0'
}

tasks.register('generate', Wsdl2Java) {
    toolOptions {
        wsdl = file('src/main/wsdl/commonService.wsdl').absolutePath
        packageNames = ['com.small.rose.demo.modules.webservices.stub.common']
        outputDir = layout.buildDirectory.dir('generated/sources/cxf').get().asFile
        extraArgs.addAll(['-encoding', 'UTF-8'])
    }
}
compileJava.dependsOn tasks.withType(Wsdl2Java)

// 只声明编译期依赖（生成的代码只用 JAXB 注解，运行时不需要 CXF）
dependencies {
    compileOnly 'jakarta.xml.bind:jakarta.xml.bind-api:4.0.4'
    compileOnly 'jakarta.xml.ws:jaxws-api:4.0.0'
}
```

主项目 build.gradle 依赖：

```
dependencies {
    implementation project(':stubs:common-service')  // 自动依赖 JAR
    implementation project(':stubs:order-service')
    implementation 'org.apache.cxf:cxf-spring-boot-starter-jaxws:4.1.6'
}
```

最终产出 JAR：
```
stubs/common-service/build/libs/common-service-1.0.0.jar
```
这种jar 只含 SEI 接口 + DTO 类，不混任何业务代码。可以独立发版、共享给其他项目。


方案 B：同一模块内抽 source set（不想拆项目）

不拆模块，但在一个 build.gradle 里分 source set：
```
// 1. 新建 source set 放生成代码
sourceSets {
    stub {
        java {
            srcDir layout.buildDirectory.dir('generated/sources/cxf')
            srcDir 'src/stub/java'
        }
    }
}

// 2. 生成代码的编译期依赖
dependencies {
    stubCompileOnly 'jakarta.xml.bind:jakarta.xml.bind-api:4.0.4'
    stubCompileOnly 'jakarta.xml.ws:jaxws-api:4.0.0'
}

// 3. 单独 task 打 stub JAR（只含生成的类）
task stubJar(type: Jar) {
    archiveBaseName = 'common-service-stub'
    from sourceSets.stub.output
}

// 4. 主项目编译时依赖 stub output
dependencies {
    implementation sourceSets.stub.output
}

// 5. 生成任务：写到 stub source set 的位置
tasks.register('generateCommonService', Wsdl2Java) {
    toolOptions {
        wsdl = file('src/main/wsdl/commonService.wsdl').absolutePath
        packageNames = ['com.small.rose.demo.modules.webservices.stub']
        outputDir = sourceSets.stub.java.srcDirs.first().parentFile
        extraArgs.addAll(['-encoding', 'UTF-8'])
    }
}
compileStubJava.dependsOn tasks.withType(Wsdl2Java)
compileJava.dependsOn compileStubJava
```

执行 gradle stubJar 即可单独打 stub JAR。

两方案对比

|  x | 独立子模块（方案A）	| 同模块 source set（方案B） |
|----|---------------|-------------------------|
|隔离程度|	✅ 完全隔离，独立仓库都行|	⚠同一仓库，依赖编译顺序|
|发版	|✅ 单独发版、单独版本号	|⚠和主项目同版本 |
|依赖管理	|✅ 显式依赖，IDE 感知	|⚠source set 输出需要手动管 |
|多人协作	|✅ stubs/ 可独立 PR	|⚠改 build.gradle 影响所有人 |
|复杂度	|中等（多一个 settings）	|高（source set + 手动管 JAR） |

生产推荐方案 A。你的场景（1-3 个 WSDL、Java 17、Spring Boot 3）完全适用。


当然对于模块化的东西我们作为调用方其实大多数都不会修改，我们更希望有一个固定的jar来减少源码编译的风险，只想简单的打个jar

只需要在现有 build.gradle 加一个 Jar 任务，从已编译的类里只提取生成的 stub 目录。

做法:

在 build.gradle 已有 generateCommonService 任务之后，加：
```
task stubJar(type: Jar) {
    archiveBaseName = 'commonservice'
    from(layout.buildDirectory.dir('classes/java/main')) {
        include '**/webservices/stub/**'   // 只取生成的桩类
    }
}
```
执行：
```
gradle stubJar
```

> SEI的桩文件一旦确认好，就不要反复修改。

如果需要打多个

```
def stubJars = [
    common: [baseName: 'commonservice', pkg: 'com/small/rose/demo/modules/webservices/stub/common'],
    order:  [baseName: 'orderservice',  pkg: 'com/small/rose/demo/modules/webservices/stub/order'],
]

stubJars.each { name, cfg ->
    tasks.register("${name}StubJar", Jar) {
        dependsOn compileJava
        archiveBaseName = cfg.baseName
        archiveVersion = ''
        from(layout.buildDirectory.dir('classes/java/main')) {
            include "${cfg.pkg}/*.class"
        }
    }
}
```

执行：
```
gradle commonStubJar orderStubJar
```

产出：build/libs/commonservice.jar、build/libs/orderservice.jar