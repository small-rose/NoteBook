---
layout: default
title: SB3-CXF-JAX-WS
parent: SpringBoot3
has_children: false
nav_order: 1003
---

Spring Boot 3 WebService 开发指南 - CXF JAX-WS 实战教程 (boot3-004-jaxws) .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }



## WebService 发展历程

WebService 技术起源于 2000 年初，是 SOAP 协议和 WSDL 标准的推动下发展起来的。早期 WebService 主要用于企业级应用程序之间的系统集成，随着微服务架构的兴起，WebService 逐渐演变为一种标准的服务暴露和调用方式。

**关键里程碑：**
- 2000 年：WSDL 1.1 发布
- 2003 年：SOAP 1.2 发布
- 2004 年：WS-I 规范发布
- 2006 年：JAX-WS 2.0 发布
- 2011 年：Apache CXF 2.0 发布
- 2019 年：Jakarta EE 8，支持 Jakarta XML Web Services

## WebService 组件对比分析

### 1. REST vs WebService 对比

| 维度 | REST | WebService |
|------|------|------------| 
| **通信协议** | HTTP/HTTPS | HTTP/HTTPS + SOAP |
| **数据格式** | JSON、XML | XML (SOAP Envelope) |
| **描述方式** | 基于 URI | WSDL 契约 |
| **状态管理** | 无状态 | 支持复杂消息处理 |
| **安全性** | OAuth2、JWT | WS-Security、SSL |
| **性能** | 更高 | 较低 |
| **开发成本** | 较低 | 较高 |
| **适用场景** | 轻量级微服务 | 企业级系统集成 |

### 2. WebService 主流框架对比

| 框架 | 维护者 | 技术栈 | 适用场景 |
|--------|----------|------------|------------|
| **Apache CXF** | Apache 基金会 | Java/Kotlin | 企业级、混合协议支持 |
| **Spring WS** | Spring | Java/Kotlin | Spring 生态集成 |
| **JAX-WS** | Java 平台 | Java | 官方标准 |
| **SOAP4J** | OpenSource | Java | 旧项目迁移 |

### 3. 当前应用现状

- **金融行业**：银行系统对账、支付处理
- **制造业**：MES 系统集成、ERP 系统对接
- **医疗行业**：电子病历、医疗保险系统
- **政府项目**：电子政务系统互联互通

## Apache CXF 发展历程

### 1. 萌芽期 (2004-2008)
- **2004 年**：Apache CXF 项目启动
- **2006 年**：首次发布 2.0.0 版本
- **2008 年**：正式进入 Apache 孵化器

### 2. 成长期 (2009-2015)
- **2011 年**：CXF 2.7.0 发布，支持 Spring Integration
- **2013 年**：CXF 3.0.0 发布，支持 WebSocket
- **2015 年**：CXF 3.3.0 发布，支持互操作性

### 3. 成熟期 (2016-2024)
- **2018 年**：CXF 3.3.0，Spring Boot 2.0 支持
- **2021 年**：CXF 3.5.0，Kubernetes 原生支持
- **2024 年**：CXF 4.1.3，支持 Spring Boot 3

**当前流行版本：** `4.1.6` (Spring Boot 3.5 配套)

### Spring Boot 版本与 CXF 版本的配套关系

| Spring Boot 版本 | CXF 版本 | `cxf-spring-boot-starter-jaxws` 版本 | 备注 |
|---------------|----------|------------------------------|-------|
| **Spring Boot 1.5** | 3.1.x | `cxf-spring-boot-starter-jaxws:3.2.0` | 基于 CXF 3.1.x |
| **Spring Boot 2.0** | 3.2.x | `cxf-spring-boot-starter-jaxws:3.2.0` | 适配 Spring Boot 2.0 |
| **Spring Boot 2.5** | 3.4.x | `cxf-spring-boot-starter-jaxws:3.4.0` | 适配 Spring Boot 2.5 |
| **Spring Boot 3.0** | 3.5.x | `cxf-spring-boot-starter-jaxws:3.5.0` | 适配 Spring Boot 3.0 |
| **Spring Boot 3.1** | 3.5.x | `cxf-spring-boot-starter-jaxws:3.5.1` | 适配 Spring Boot 3.1 |
| **Spring Boot 3.5** | 4.1.x | `cxf-spring-boot-starter-jaxws:4.1.6` | 适配 Spring Boot 3.5 |

**版本升级规则：**
- 每 major 版本升级，通常需要适配 Spring Boot 版本
- 保持 **1.x minor 版本** 进行功能增强和 bug 修复
- 仅在有重大 breaking changes 时才进行 **major 版本** 升级

**当前推荐版本组合：**
```gradle
implementation 'org.springframework.boot:spring-boot-starter:3.5.14'
implementation 'org.apache.cxf:cxf-spring-boot-starter-jaxws:4.1.6'
```

## WSDL 契约文件详解

### 1. WSDL 结构

WSDL (Web Services Description Language) 包含四个核心部分：

```xml
<definitions xmlns="http://schemas.xmlsoap.org/wsdl/"
             targetNamespace="http://example.com/service">
    <!-- 1. Types: 数据类型定义 -->
    <types>
        <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                    targetNamespace="http://example.com/types">
            <xsd:element name="GetUserInfoRequest">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="userId" type="xsd:string"/>
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
            <xsd:element name="GetUserInfoResponse">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="userInfo" type="tns:UserInfo"/>
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
            <!-- 其他类型定义... -->
        </xsd:schema>
    </types>

    <!-- 2. Message: 消息结构 -->
    <message name="GetUserInfoRequest">
        <part name="parameters" element="tns:GetUserInfoRequest"/>
    </message>
    <message name="GetUserInfoResponse">
        <part name="parameters" element="tns:GetUserInfoResponse"/>
    </message>

    <!-- 3. PortType: 服务接口 -->
    <portType name="CommonService">
        <operation name="getUserInfo">
            <input message="tns:GetUserInfoRequest"/>
            <output message="tns:GetUserInfoResponse"/>
        </operation>
        <!-- 其他操作... -->
    </portType>

    <!-- 4. Binding & Service: 服务实现 -->
    <binding name="CommonServiceSOAP" type="tns:CommonService">
        <soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
        <operation name="getUserInfo">
            <soap:operation soapAction=""/>
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
    </binding>

    <service name="CommonService">
        <port name="CommonServicePort" binding="tns:CommonServiceSOAP">
            <soap:address location="http://localhost:8080/services/common"/>
        </port>
    </service>
</definitions>
```

### 2. SOAP 绑定风格详解（rpc/encoded vs rpc/literal vs document/literal）

#### 2.1 三种绑定风格对比

| 绑定风格 | 说明 | CXF 支持 | 使用场景 |
|---------|------|----------|----------|
| **rpc/encoded** | RPC 风格，编码类型 | ❌ 不支持 | 旧系统（Axis 1.x） |
| **rpc/literal** | RPC 风格，字面量类型 | ✅ 支持 | 部分旧系统 |
| **document/literal** | 文档风格，字面量类型 | ✅ 支持（推荐） | 现代 WebService |

#### 2.2 三种风格的 SOAP 消息格式对比

**rpc/encoded 格式（CXF 不支持）：**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:enc="http://schemas.xmlsoap.org/soap/encoding/"
                  xmlns:typ="http://example.com/types">
  <soapenv:Body>
    <typ:getUserInfo>
      <userId xsi:type="enc:string">user001</userId>  <!-- 带类型编码 -->
    </typ:getUserInfo>
  </soapenv:Body>
</soapenv:Envelope>
```

**rpc/literal 格式（CXF 支持）：**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:typ="http://example.com/types">
  <soapenv:Body>
    <typ:getUserInfo>
      <userId>user001</userId>  <!-- 无类型编码 -->
    </typ:getUserInfo>
  </soapenv:Body>
</soapenv:Envelope>
```

**document/literal 格式（CXF 推荐）：**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:typ="http://example.com/types">
  <soapenv:Body>
    <typ:getUserInfoRequest>  <!-- 元素名称由 WSDL 定义 -->
      <typ:userId>user001</typ:userId>
    </typ:getUserInfoRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

#### 2.3 WSDL 中绑定风格的定义方式

**rpc/encoded（旧系统，CXF 不支持）：**

```xml
<binding name="CommonServiceBinding" type="tns:CommonService">
  <soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
  <!-- 注意：没有 use="literal"，默认为 encoded -->
  <operation name="getUserInfo">
    <soap:operation soapAction="getUserInfo"/>
    <input>
      <soap:body use="encoded"
                 encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
    </input>
    <output>
      <soap:body use="encoded"
                 encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
    </output>
  </operation>
</binding>
```

**rpc/literal（CXF 支持）：**

```xml
<binding name="CommonServiceBinding" type="tns:CommonService">
  <soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
  <operation name="getUserInfo">
    <soap:operation soapAction="getUserInfo"/>
    <input>
      <soap:body use="literal"/>  <!-- 明确声明 use="literal" -->
    </input>
    <output>
      <soap:body use="literal"/>
    </output>
  </operation>
</binding>
```

**document/literal（CXF 推荐）：**

```xml
<binding name="CommonServiceBinding" type="tns:CommonService">
  <soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
  <operation name="getUserInfo">
    <soap:operation soapAction="getUserInfo"/>
    <input>
      <soap:body use="literal"/>
    </input>
    <output>
      <soap:body use="literal"/>
    </output>
  </operation>
</binding>
```

#### 2.4 如何检查你的 WSDL 绑定风格

```bash
# 方法一：使用 curl 下载后检查
curl -s "http://xxxx/xxx/?wsdl" | grep -i "soap:binding"

# 方法二：使用 grep 直接检查
grep -i "style=" wsdl文件
grep -i "use=" wsdl文件

# 方法三：使用 xmllint 格式化后查看
xmllint --format wsdl文件 | grep -A5 "binding"
```

**判断规则：**
- 如果看到 `use="encoded"` → **rpc/encoded**（CXF 不支持）
- 如果看到 `style="rpc"` 且 `use="literal"` → **rpc/literal**（CXF 支持）
- 如果看到 `style="document"` 且 `use="literal"` → **document/literal**（CXF 推荐）

#### 2.5 解决 rpc/encoded 问题的方案

> ⚠️ **重要警告：关于手动修改 WSDL 的严重后果**
>
> **如果你无法修改服务端 WSDL，绝对不要手动修改客户端 WSDL 的绑定风格！**
>
> 手动修改 WSDL 绑定风格会导致以下严重问题：
> 1. **客户端与服务端消息格式不匹配**：客户端发送 rpc/literal 格式，服务端期望 rpc/encoded 格式
> 2. **运行时调用失败**：SOAP 请求/响应解析错误，数据丢失或格式错误
> 3. **难以排查问题**：错误信息不明确，调试困难
> 4. **生产环境事故**：可能导致系统不可用
>
> **正确做法**：当服务端 WSDL 为 rpc/encoded 且无法修改时，**必须联系服务端维护者修改 WSDL**。现代工具（CXF、wsimport 4.x）均不支持 rpc/encoded。

**方案一：联系服务端维护者修改 WSDL（推荐）**

```bash
# 请服务端将 use="encoded" 改为 use="literal"
# 修改后，使用 CXF 或 wsimport 生成客户端代码
wsimport -keep -verbose http://localhost:8080/services/common?wsdl
```

**方案二：手动修改 WSDL 风格（⚠️ 仅当服务端可修改时使用）**

> ⚠️ **再次强调：只有在你能同时修改服务端 WSDL 时才能使用此方案！**
> 
> 如果服务端不可修改，使用此方案会导致客户端与服务端不兼容，调用必然失败。

```xml
<!-- 修改前（rpc/encoded）-->
<soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
<input>
  <soap:body use="encoded"
             encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
</input>

<!-- 修改后（rpc/literal）-->
<soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
<input>
  <soap:body use="literal"/>
</input>

<!-- 或者修改后（document/literal，推荐）-->
<soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
<input>
  <soap:body use="literal"/>
</input>
```

**方案三：下载 WSDL 到本地处理后生成代码**

> ⚠️ **警告：此方案中的"手动修改 WSDL"仅在服务端可修改时才有效！**
>
> 如果服务端不可修改，下载到本地后不要修改绑定风格，直接用 wsimport 生成代码。

```bash
# 1. 下载 WSDL 到本地
curl -o src/main/wsdl/commonService.wsdl "http://xxxx/xxx/?wsdl"

# 2. 手动修改 WSDL 中的绑定风格
#    将 use="encoded" 改为 use="literal"
#    或者将 style="rpc" 改为 style="document"

# 3. 使用修改后的本地 WSDL 生成代码
wsdl2java -client -d src/main/java src/main/wsdl/commonService.wsdl
```

#### 2.6 各框架对绑定风格的支持情况

| 框架 | rpc/encoded | rpc/literal | document/literal | document/wrapped |
|------|-------------|-------------|------------------|------------------|
| **Apache CXF** | ❌ | ✅ | ✅ | ✅ |
| **Axis 1.x** | ✅ | ✅ | ✅ | ❌ |
| **Axis 2.x** | ❌ | ✅ | ✅ | ✅ |
| **JAX-WS (wsimport 4.x)** | ❌ | ✅ | ✅ | ✅ |
| **Spring WS** | ❌ | ✅ | ✅ | ✅ |

> ⚠️ **重要**：wsimport 4.x（Jakarta EE 10）已**不支持** rpc/encoded。旧版本（如 2.x）可能支持，但不推荐使用。

#### 2.7 最佳实践建议

> ⚠️ **特别重要：服务端不可修改时的正确做法**
>
> **如果你无法修改服务端 WSDL（这是大多数情况），请遵循以下规则：**
>
> 1. **不要手动修改 WSDL 绑定风格**：这会导致客户端与服务端不兼容
> 2. **wsimport 和 CXF 都不支持 rpc/encoded**：现代工具已摒弃此过时协议
> 3. **联系服务端维护者**：让他们将 WSDL 改为 rpc/literal 或 document/literal
> 4. **如果服务端确实不可改**：考虑使用 Axis 1.x（老旧但唯一支持 rpc/encoded）或手动拼 SOAP 消息

**详细方案：**

1. **新项目首选**：document/literal + wrapped（最标准，兼容性最好）
2. **旧系统迁移**：先用 wsimport 生成代码，再逐步迁移到 CXF
3. **服务端可控**：修改 WSDL 绑定风格为 rpc/literal 或 document/literal
4. **服务端不可控（最常见情况）**：
   ```
   # ✅ 正确做法：联系服务端维护者修改 WSDL
   # ❌ 错误做法：wsimport / CXF 都无法处理 rpc/encoded
   ```

#### 2.8 wsimport Gradle 完整配置教程（Java 17+ 环境）

> ⚠️ **重要说明**：Java 17 已移除 wsimport 命令行工具，需要通过 `com.sun.xml.ws:jaxws-tools` 库来使用。

##### 2.8.1 完整 build.gradle 配置

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.14'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.example'
version = '1.0.0'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

// ============================================
// wsimport 任务配置（用于生成 WebService 客户端代码）
// ============================================

// 1. 定义 wsimport 工具依赖配置
configurations {
    wsimportTools
}

// 2. 添加 jaxws-tools 依赖（提供 wsimport 功能）
dependencies {
    wsimportTools 'com.sun.xml.ws:jaxws-tools:4.0.3'
    wsimportTools 'com.sun.xml.ws:jaxws-rt:4.0.3'
    wsimportTools 'jakarta.xml.ws:jakarta.xml.ws-api:4.0.2'
    wsimportTools 'jakarta.xml.bind:jakarta.xml.bind-api:4.0.2'
    wsimportTools 'org.glassfish.jaxb:jaxb-runtime:4.0.5'
}

// 3. 将 wsimport 生成的源码目录加入 source set
sourceSets {
    main {
        java {
            srcDir layout.buildDirectory.dir('generated/sources/wsimport')
        }
    }
}

// 4. wsimport 代码生成任务
tasks.register('wsimportGenerate', JavaExec) {
    group = 'build'
    description = '使用 wsimport 根据 WSDL 生成 Java 客户端代码'

    // 使用 jaxws-tools 中的 wsimport
    classpath = configurations.wsimportTools
    mainClass = 'com.sun.tools.ws.WsImport'

    // 输出目录
    def outputDir = layout.buildDirectory.dir('generated/sources/wsimport').get().asFile

    args = [
        '-keep',                          // 保留生成的 Java 源文件
        '-d', outputDir.absolutePath,     // 输出目录
        '-p', 'com.example.webservice.stub.wsimport',  // 包名
        '-encoding', 'UTF-8',             // 编码
        '-verbose',                        // 详细输出
        file('src/main/wsdl/commonService.wsdl').absolutePath  // WSDL 文件路径
    ]

    // JVM 参数（Java 17+ 需要）
    jvmArgs = [
        '--add-opens', 'java.base/java.lang=ALL-UNNAMED',
        '--add-opens', 'java.base/java.util=ALL-UNNAMED',
        '--add-opens', 'java.base/java.io=ALL-UNNAMED',
        '--add-opens', 'java.xml/com.sun.org.apache.xerces.internal.xni.parser=ALL-UNNAMED'
    ]

    // 确保输出目录存在
    doFirst {
        outputDir.mkdirs()
    }
}

// 5. 编译前自动跑 wsimport 任务
compileJava.dependsOn wsimportGenerate

// 6. wsimport 生成的代码打包成独立 JAR
tasks.register('wsimportStubJar', Jar) {
    dependsOn compileJava
    archiveBaseName = 'commonservice-wsimport'
    archiveVersion = ''
    from(layout.buildDirectory.dir('classes/java/main')) {
        include 'com/example/webservice/stub/wsimport/**/*.class'
    }
}
```

##### 2.8.2 执行命令

```bash
# 生成客户端代码
gradle wsimportGenerate

# 打包成独立 JAR
gradle wsimportStubJar

# 一步到位（编译 + 打包）
gradle build
```

##### 2.8.3 生成的文件结构

```
build/
├── generated/
│   └── sources/
│       └── wsimport/
│           └── com/example/webservice/stub/wsimport/
│               ├── CommonService.java
│               ├── CommonServiceImplService.java
│               ├── GetUserInfo.java
│               ├── GetUserInfoResponse.java
│               ├── ObjectFactory.java
│               ├── ProcessComplexData.java
│               ├── ProcessComplexDataResponse.java
│               ├── SendSms.java
│               ├── SendSmsResponse.java
│               ├── SyncData.java
│               ├── SyncDataResponse.java
│               ├── UserVo.java
│               ├── WsRequestVo.java
│               ├── WsResponseVo.java
│               └── package-info.java
└── libs/
    └── commonservice-wsimport.jar  ← 独立 JAR 包
```

##### 2.8.4 常见问题解决

**问题 1：Could not find method wsimportTools()**
```groovy
// 解决方案：必须先在 configurations 中定义 wsimportTools
configurations {
    wsimportTools
}
```

**问题 2：Could not resolve all files for configuration**
```groovy
// 解决方案：检查依赖版本是否正确
dependencies {
    wsimportTools 'com.sun.xml.ws:jaxws-tools:4.0.3'  // 不要加 :jar 后缀
}
```

**问题 3：Could not find main class**
```groovy
// 解决方案：使用正确的主类名
mainClass = 'com.sun.tools.ws.WsImport'  // 不是 com.sun.tools.ws.wscompile.WsimportTool
```

##### 2.8.5 wsimport vs wsdl2java 对比

| 特性 | wsimport (4.x) | wsdl2java (CXF) | Axis 1.x |
|------|----------------|-----------------|----------|
| **rpc/encoded 支持** | ❌ 不支持 | ❌ 不支持 | ✅ 支持 |
| **rpc/literal 支持** | ✅ 支持 | ✅ 支持 | ✅ 支持 |
| **document/literal 支持** | ✅ 支持 | ✅ 支持 | ✅ 支持 |
| **Java 17+ 支持** | ⚠️ 需要 jaxws-tools 库 | ✅ 原生支持 | ❌ 可能有问题 |
| **Spring Boot 3 集成** | ⚠️ 需要自定义配置 | ✅ 有官方插件 | ❌ 不兼容 |
| **代码质量** | 一般 | 更好 | 较差 |
| **维护状态** | 已废弃 | 活跃维护 | 已废弃 |

**结论**：
- **rpc/encoded 格式** → 联系服务端维护者修改 WSDL（现代工具已不支持）
- **rpc/literal 或 document/literal** → 使用 **wsdl2java (CXF)**

##### 2.8.6 多个 WSDL 生成多个独立 JAR 的配置

> **场景**：项目中有多个 WSDL 需要分别生成独立的 JAR 包

**方式一：动态任务（推荐）**

使用 `ext` 定义 WSDL 列表，动态创建任务：

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.14'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.example'
version = '1.0.0'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

// ============================================
// wsimport 任务配置（支持多个 WSDL）
// ============================================

// 1. 定义 WSDL 列表：key = 输出 JAR 名称，value = [WSDL 文件, 包名]
ext {
    wsdlServices = [
        'commonService': [file('src/main/wsdl/commonService.wsdl'), 'com.example.stub.common'],
        'userService':  [file('src/main/wsdl/userService.wsdl'),  'com.example.stub.user'],
        'orderService': [file('src/main/wsdl/orderService.wsdl'), 'com.example.stub.order'],
    ]
}

// 2. 定义 wsimport 工具依赖配置
configurations {
    wsimportTools
}

// 3. 添加 jaxws-tools 依赖
dependencies {
    wsimportTools 'com.sun.xml.ws:jaxws-tools:4.0.3'
    wsimportTools 'com.sun.xml.ws:jaxws-rt:4.0.3'
    wsimportTools 'jakarta.xml.ws:jakarta.xml.ws-api:4.0.2'
    wsimportTools 'jakarta.xml.bind:jakarta.xml.bind-api:4.0.2'
    wsimportTools 'org.glassfish.jaxb:jaxb-runtime:4.0.5'
}

// 4. 将所有 wsimport 生成的源码目录加入 source set
sourceSets {
    main {
        java {
            srcDir layout.buildDirectory.dir('generated/sources/wsimport')
        }
    }
}

// 5. 为每个 WSDL 动态创建 generate + stubJar 任务
wsdlServices.each { name, wsdlConfig ->
    def wsdlFile = wsdlConfig[0]
    def packageName = wsdlConfig[1]

    // 代码生成任务
    tasks.register("wsimportGenerate_${name}", JavaExec) {
        group = 'build'
        description = "wsimport: 从 ${name}.wsdl 生成 Java 代码"

        classpath = configurations.wsimportTools
        mainClass = 'com.sun.tools.ws.WsImport'

        def outputDir = layout.buildDirectory.dir("generated/sources/wsimport/${name}").get().asFile

        args = [
            '-keep',
            '-d', outputDir.absolutePath,
            '-p', packageName,
            '-encoding', 'UTF-8',
            '-verbose',
            wsdlFile.absolutePath
        ]

        jvmArgs = [
            '--add-opens', 'java.base/java.lang=ALL-UNNAMED',
            '--add-opens', 'java.base/java.util=ALL-UNNAMED',
            '--add-opens', 'java.base/java.io=ALL-UNNAMED',
            '--add-opens', 'java.xml/com.sun.org.apache.xerces.internal.xni.parser=ALL-UNNAMED'
        ]

        doFirst {
            outputDir.mkdirs()
        }
    }

    // 独立 JAR 打包任务
    tasks.register("wsimportStubJar_${name}", Jar) {
        dependsOn compileJava
        dependsOn "wsimportGenerate_${name}"

        archiveBaseName = "${name}-wsimport"
        archiveVersion = ''

        from(layout.buildDirectory.dir('classes/java/main')) {
            include "${packageName.replace('.', '/')}/**/*.class"
        }
    }
}

// 6. 总任务：生成所有
tasks.register('wsimportGenerateAll') {
    group = 'build'
    description = 'wsimport: 生成所有 WSDL 的客户端代码'
    wsdlServices.each { name, _ ->
        dependsOn "wsimportGenerate_${name}"
    }
}

// 7. 总任务：打包所有
tasks.register('wsimportStubJarAll') {
    group = 'build'
    description = 'wsimport: 打包所有 WSDL 的独立 JAR'
    wsdlServices.each { name, _ ->
        dependsOn "wsimportStubJar_${name}"
    }
}

// 8. 编译前自动跑所有 wsimport 生成任务
compileJava.dependsOn wsimportGenerateAll
```

**执行命令**：

```bash
# 生成所有 WSDL 的代码
gradle wsimportGenerateAll

# 打包所有独立 JAR
gradle wsimportStubJarAll

# 只生成/打包单个 WSDL
gradle wsimportGenerate_commonService
gradle wsimportStubJar_commonService

# 一步到位
gradle build
```

**生成的 JAR 文件**：

```
build/libs/
├── commonService-wsimport.jar
├── userService-wsimport.jar
└── orderService-wsimport.jar
```

---

**方式二：手动定义任务（简单场景）**

```groovy
// 为每个 WSDL 手动定义任务
tasks.register('wsimportGenerate_commonService', JavaExec) {
    classpath = configurations.wsimportTools
    mainClass = 'com.sun.tools.ws.WsImport'
    def outputDir = layout.buildDirectory.dir('generated/sources/wsimport/common').get().asFile
    args = ['-keep', '-d', outputDir.absolutePath, '-p', 'com.example.stub.common',
            '-encoding', 'UTF-8', file('src/main/wsdl/commonService.wsdl').absolutePath]
    jvmArgs = ['--add-opens', 'java.base/java.lang=ALL-UNNAMED']
    doFirst { outputDir.mkdirs() }
}

tasks.register('wsimportGenerate_userService', JavaExec) {
    classpath = configurations.wsimportTools
    mainClass = 'com.sun.tools.ws.WsImport'
    def outputDir = layout.buildDirectory.dir('generated/sources/wsimport/user').get().asFile
    args = ['-keep', '-d', outputDir.absolutePath, '-p', 'com.example.stub.user',
            '-encoding', 'UTF-8', file('src/main/wsdl/userService.wsdl').absolutePath]
    jvmArgs = ['--add-opens', 'java.base/java.lang=ALL-UNNAMED']
    doFirst { outputDir.mkdirs() }
}

// 以此类推...
```

> **建议**：3 个以上 WSDL 时使用「方式一」动态任务，少于 3 个可使用「方式二」手动定义。

---

### 3. WSDL 生成流程

```
Java 接口 (SEI) + Annotation
    ↓
CXF 插件解析 @WebService
    ↓
生成 WSDL 文件
    ↓
发布到 Spring Boot 应用
    ↓
客户端自动生成代理
```

### 4. CXF 注解体系

| 注解 | 使用场景 | 示例 |
|---------|-----------|-------|
| `@WebService` | 服务端 SEI 接口 | `@WebService(serviceName = "CommonService")` |
| `@WebMethod` | 公开方法 | `@WebMethod(operationName = "getUserInfo")` |
| `@WebParam` | 参数映射 | `@WebParam(name = "userId") String userId` |
| `@WebResult` | 返回值映射 | `@WebResult(name = "userInfo") UserInfo userInfo` |
| `@SOAPBinding` | SOAP 绑定 | `@SOAPBinding(style = Style.DOCUMENT)` |
| `@HandlerChain` | SOAP 处理器 | `@HandlerChain(file = "/WEB-INF/laf.xml")` |

## Apache CXF 工作原理

### 1. 服务端流程

```
1. SEI 接口 + 实现类 (CommonServiceImpl)
2. 启动 Spring Boot 应用
3. Spring 自动发现 @WebService SEI
4. CXF 注册服务端点 (Server)
5. 通过 HTTP 端口接受 SOAP 请求
6. 执行业务逻辑，生成 SOAP 响应
```

### 2. 客户端流程

```
1. 通过 WSDL 获取服务描述
2. 运行 wsdl2java 工具生成客户端代理
3. Spring 依赖注入代理到服务调用层
4. 客户端调用代理方法，发送 HTTP 请求
5. 接收并反序列化 SOAP 响应
```

### 3. 核心组件

| 组件 | 角色 | 典型用法 |
|----------|--------|------------|
| **Server** | 服务端点实现 | `JaxWsServerFactoryBean` |
| **Bus** | CXF 消息处理管道 | Spring 自动注入 |
| **Binding** | HTTP/SOAP 协议处理 | `SoapBindingConfiguration` |
| **Invoker** | 方法调用桥接 | `MethodInvoker`
| **Dispatch** | 请求分发 | `DispatchServlet`

## 完整使用案例

### 1. 服务端 (SIB) - 发布 WebService

#### 1.1 建立服务端项目结构

```
project-root/
├── src/main/java/com/example/service/
│   ├── CommonService.java              ← SEI 接口
│   └── impl/CommonServiceImpl.java     ← 实现类
├── src/main/resources/application.yml   ← Spring 配置
├── src/main/java/com/example/config/   ← CXF 配置类
└── build.gradle                        ← 依赖和插件配置
```

#### 1.2 CommonService.java (SEI)

```java
package com.example.service;

import jakarta.jws.WebMethod;
import jakarta.jws.WebParam;
import jakarta.jws.WebService;
import jakarta.jws.soap.SOAPBinding;

/**
 * CommonService SEI - WebService 服务端点接口
 * 
 * @WebService 注解标记这是一个 WebService 接口
 * @SOAPBinding 定义 SOAP 协议绑定
 */
@WebService(
    serviceName = "CommonService",
    portName = "CommonServicePort",
    targetNamespace = "http://example.com/service",
    endpointInterface = "com.example.service.CommonService"
)
@SOAPBinding(style = SOAPBinding.Style.DOCUMENT, use = SOAPBinding.Use.LITERAL)
public interface CommonService {

    /**
     * 获取用户信息
     * 
     * @WebMethod 标记这是一个可操作的方法
     * @WebParam 指定参数在 SOAP 消息中的映射
     */
    @WebMethod(operationName = "getUserInfo")
    String getUserInfo(
        @WebParam(name = "userId", partName = "userId") String userId
    );

    /**
     * 发送短信
     */
    @WebMethod
    String sendSms(
        @WebParam(name = "phone", partName = "phone") String phone,
        @WebParam(name = "message", partName = "message") String message
    );

    /**
     * 同步数据
     */
    @WebMethod
    boolean syncData(
        @WebParam(name = "data", partName = "data") String data
    );
}
```

#### 1.3 CommonServiceImpl.java (实现类)

```java
package com.example.service.impl;

import com.example.model.UserInfo;
import com.example.service.CommonService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

/**
 * CommonServiceImpl - WebService 的具体实现
 * 
 * 通过 @Service 注入到 Spring 容器中
 * 实现 SEI 接口中的所有方法
 */
@Service
public class CommonServiceImpl implements CommonService {

    private static final Logger logger = LoggerFactory.getLogger(CommonServiceImpl.class);

    @Override
    public String getUserInfo(String userId) {
        logger.info("获取用户信息：userId={}", userId);
        
        // 模拟查询用户
        UserInfo userInfo = new UserInfo(
            userId,
            "张三",
            "zhangsan@example.com",
            "13800000000"
        );
        
        // 转换为 XML 格式返回
        return "<userInfo>" +
               "<userId>" + userInfo.getUserId() + "</userId>" +
               "<name>" + userInfo.getName() + "</name>" +
               "<email>" + userInfo.getEmail() + "</email>" +
               "<phone>" + userInfo.getPhone() + "</phone>" +
               "</userInfo>";
    }

    @Override
    public String sendSms(String phone, String message) {
        logger.info("发送短信：phone={}, message={}", phone, message);
        
        // 模拟短信发送
        String messageId = "MSG_" + System.currentTimeMillis();
        return "<response>" +
               "<success>true</success>" +
               "<messageId>" + messageId + "</messageId>" +
               "<phone>" + phone + "</phone>" +
               "<content>" + message + "</content>" +
               "</response>";
    }

    @Override
    public boolean syncData(String data) {
        logger.info("同步数据：data={}", data);
        
        // 模拟数据同步
        try {
            // 这里可以解析 data 并持久化
            logger.debug("数据同步成功：{}", data);
            return true;
        } catch (Exception e) {
            logger.error("数据同步失败：{}", data, e);
            return false;
        }
    }
}
```

#### 1.4 CommonServiceConfig.java (CXF 配置)

```java
package com.example.config;

import com.example.service.CommonService;
import org.apache.cxf.binding.corba.types.CorbaType;
import org.apache.cxf.jaxws.JaxWsServerFactoryBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * CommonServiceConfig - CXF WebService 配置
 * 
 * 负责配置和发布 CommonService WebService 服务端点
 */
@Configuration
public class CommonServiceConfig {

    @Value("${example.service.url:http://localhost:8080/services/common}")
    private String serviceUrl;

    @Value("${example.service.port:8080}")
    private int servicePort;

    /**
     * 创建并发布 CommonService WebService
     * 
     * @param commonServiceImpl 实现类 bean
     * @return WebService Server 对象
     */
    @Bean
    public JaxWsServerFactoryBean commonServiceServer(CommonService commonServiceImpl) {
        JaxWsServerFactoryBean factoryBean = new JaxWsServerFactoryBean();
        
        // 设置服务实现类
        factoryBean.setServiceClass(CommonService.class);
        factoryBean.setAddress(serviceUrl);
        factoryBean.setBeanId("commonService");
        
        // 设置实现类
        factoryBean.setServiceBean(commonServiceImpl);
        
        // 配置 Spring Bus
        factoryBean.setBus(org.apache.cxf.BusFactory.getDefaultBus());
        
        // 启动服务端点
        return factoryBean;
    }
}
```

#### 1.5 application.yml (Spring 配置)

```yaml
# application.yml - Spring Boot 配置

server:
  port: 8080
  servlet:
    context-path: /

spring:
  application:
    name: spring-boot3-cxf-webservice
  jackson:
    property-naming-strategy: SNAKE_CASE
    time-zone: GMT+8
  profiles:
    active: dev

# CXF WebService 配置
example:
  service:
    url: http://localhost:8080/services/common
    port: 8080
    timeout:
      connect: 5000
      read: 10000

# Spring Boot Actuator 配置
management:
  endpoints:
    web:
      base-path: /actuator
    jmx:
      enabled: false
  endpoint:
    health:
      show-details: always
      enabled: true

# 日志配置
logging:
  level:
    com.example: DEBUG
    org.apache.cxf: INFO
    org.springframework.web: INFO
```

#### 1.6 UserInfo.java (数据模型)

```java
package com.example.model;

import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlRootElement;

/**
 * UserInfo - XML/JSON 通用数据模型
 * 
 * 使用 Jackson 注解，实现 XML 和 JSON 互转
 */
@JacksonXmlRootElement(localName = "userInfo")
public class UserInfo {

    private String userId;
    private String name;
    private String email;
    private String phone;

    // 构造函数
    public UserInfo() {}

    public UserInfo(String userId, String name, String email, String phone) {
        this.userId = userId;
        this.name = name;
        this.email = email;
        this.phone = phone;
    }

    // Getter 和 Setter 方法
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }

    @Override
    public String toString() {
        return "UserInfo{" +
               "userId='" + userId + \"'" +
               ", name='" + name + \"'" +
               ", email='" + email + \"'" +
               ", phone='" + phone + \"'" +
               '}';
    }
}
```

#### 1.7 完整 build.gradle 配置

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.14'
    id 'io.spring.dependency-management' version '1.1.7'
    // CXF 代码生成插件
    id 'io.mateo.cxf-codegen' version '2.5.0'
}

// 基本项目配置
group = 'com.example'
version = '1.0.0'
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

// 依赖管理
dependencies {
    // Spring Boot 启动器
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    
    // CXF WebService
    implementation 'org.apache.cxf:cxf-spring-boot-starter-jaxws:4.1.6'
    
    // 数据类型支持
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.18.4'
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-json:2.18.4'
    
    // 开发工具
    compileOnly 'org.projectlombok:lombok:1.18.36'
    annotationProcessor 'org.projectlombok:lombok:1.18.36'
    
    // 测试
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// CXF wsdl2java 生成配置
tasks.withType(Wsdl2Java) {
    wsdl = file("src/main/wsdl/commonService.wsdl").absolutePath
    packageNames = ['com.example.webservice.stub']
    outputDir = layout.buildDirectory.dir('generated/sources/cxf').get().asFile
    extraArgs.addAll(['-encoding', 'UTF-8', '-client'])
}

compileJava.dependsOn tasks.withType(Wsdl2Java)
```

### 2. 客户端 (SEI) - 调用 WebService

#### 2.1 SEI 客户端 5 种调用方式大全

##### 方式一：JDK 原生代码实现调用

**必要条件：**
1. WebService 服务端发布地址
2. WebService 服务发布的接口类
3. 参数和返回值实体类

**核心原理：** 使用 JDK 原生的 `URL`、`URLConnection` 手工构建 SOAP 请求，通过 SAX/DOM 解析 XML 响应。

**适用场景：** 纯 Java 环境、没有第三方库、需要最小依赖的场景。

**优点：** 不依赖任何第三方库，JDK 自带。
**缺点：** 开发复杂度极高，需要手工处理 SOAP 协议细节。

**完整代码案例：**

```java
package com.example.client;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;

/**
 * 方式一：JDK 原生 WebService 客户端
 * 
 * 使用 JDK 自带的 URL、URLConnection 手工构建 SOAP 请求
 * 通过 DOM 解析 XML 响应
 */
public class JdkNativeWebServiceClient {

    private static final String SOAP_NAMESPACE = "http://schemas.xmlsoap.org/soap/envelope/";
    private static final String SERVICE_NAMESPACE = "http://webservices.modules.demo.rose.small.com/";

    /**
     * 调用 getUserInfo 方法
     * 
     * @param serviceUrl 服务端发布地址
     * @param userId 用户ID
     * @return 用户信息 XML 字符串
     */
    public static String getUserInfo(String serviceUrl, String userId) throws Exception {
        // 1. 构建 SOAP 请求 XML
        String soapRequest = buildSoapRequest("getUserInfo",
            "<ns2:getUserInfo xmlns:ns2=\"" + SERVICE_NAMESPACE + "\">" +
            "    <arg0>" + userId + "</arg0>" +
            "</ns2:getUserInfo>"
        );

        // 2. 发送 HTTP POST 请求
        String soapResponse = sendSoapRequest(serviceUrl, soapRequest);

        // 3. 解析 SOAP 响应
        return parseSoapResponse(soapResponse, "userInfo");
    }

    /**
     * 调用 sendSms 方法
     */
    public static String sendSms(String serviceUrl, String phone, String message) throws Exception {
        String soapRequest = buildSoapRequest("sendSms",
            "<ns2:sendSms xmlns:ns2=\"" + SERVICE_NAMESPACE + "\">" +
            "    <arg0>" + phone + "</arg0>" +
            "    <arg1>" + message + "</arg1>" +
            "</ns2:sendSms>"
        );

        String soapResponse = sendSoapRequest(serviceUrl, soapRequest);
        return parseSoapResponse(soapResponse, "sendSmsResponse");
    }

    /**
     * 构建 SOAP 请求 XML
     */
    private static String buildSoapRequest(String operationName, String bodyContent) {
        return "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
               "<soapenv:Envelope xmlns:soapenv=\"" + SOAP_NAMESPACE + "\">" +
               "  <soapenv:Header/>" +
               "  <soapenv:Body>" +
               bodyContent +
               "  </soapenv:Body>" +
               "</soapenv:Envelope>";
    }

    /**
     * 发送 SOAP HTTP 请求
     */
    private static String sendSoapRequest(String serviceUrl, String soapRequest) throws Exception {
        URL url = new URL(serviceUrl);
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();

        // 设置 HTTP 属性
        connection.setRequestMethod("POST");
        connection.setRequestProperty("Content-Type", "text/xml; charset=UTF-8");
        connection.setRequestProperty("SOAPAction", "");
        connection.setDoOutput(true);
        connection.setConnectTimeout(5000);
        connection.setReadTimeout(10000);

        // 写入 SOAP 请求体
        try (OutputStream os = connection.getOutputStream()) {
            byte[] input = soapRequest.getBytes(StandardCharsets.UTF_8);
            os.write(input, 0, input.length);
        }

        // 读取响应
        StringBuilder response = new StringBuilder();
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(connection.getInputStream(), StandardCharsets.UTF_8))) {
            String line;
            while ((line = br.readLine()) != null) {
                response.append(line);
            }
        }

        return response.toString();
    }

    /**
     * 解析 SOAP 响应 XML
     */
    private static String parseSoapResponse(String soapResponse, String elementName) throws Exception {
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        factory.setNamespaceAware(true);
        DocumentBuilder builder = factory.newDocumentBuilder();
        Document document = builder.parse(new java.io.ByteArrayInputStream(
            soapResponse.getBytes(StandardCharsets.UTF_8)));

        // 提取目标元素内容
        NodeList nodeList = document.getElementsByTagNameNS("*", elementName);
        if (nodeList.getLength() > 0) {
            return nodeList.item(0).getTextContent();
        }
        return null;
    }

    /**
     * 测试方法
     */
    public static void main(String[] args) throws Exception {
        String serviceUrl = "http://localhost:8080/services/commonService";

        // 测试 getUserInfo
        String userInfo = getUserInfo(serviceUrl, "user001");
        System.out.println("用户信息：" + userInfo);

        // 测试 sendSms
        String smsResult = sendSms(serviceUrl, "13800000000", "测试短信");
        System.out.println("短信结果：" + smsResult);
    }
}
```

---

##### 方式二：wsimport/wsdl2java 命令或组件生成客户端代码

**必要条件：**
1. WebService 服务端发布地址
2. WSDL 契约文件（可从发布地址获取）

**核心原理：** 使用 `wsimport`（JDK 自带）或 `wsdl2java`（CXF 工具）从 WSDL 自动生成 Java 客户端代理代码。

**适用场景：** 标准 WebService 开发、需要类型安全、需要提前准备客户端代码。

**优点：** 类型安全、IDE 支持好、代码可维护。
**缺点：** 需要提前生成代码、WSDL 变更时需要重新生成。

###### 2.1.2.1 Spring Boot 2.5 环境

**A. 命令行模式（wsimport - JDK 自带）**

```bash
# 基本用法：生成客户端代码
wsimport -keep -verbose http://localhost:8080/services/common?wsdl

# 指定输出目录
wsimport -keep -d src/main/java http://localhost:8080/services/common?wsdl

# 生成带包名的代码
wsimport -keep -p com.example.webservice.stub -d src/main/java http://localhost:8080/services/common?wsdl

# 详细输出（调试用）
wsimport -keep -verbose -debug http://localhost:8080/services/common?wsdl

# 禁用验证（跳过 WSDL 校验）
wsimport -keep -Xnocompile http://localhost:8080/services/common?wsdl
```

**B. Maven 插件模式（jaxws-maven-plugin）**

```xml
<!-- pom.xml -->
<plugins>
    <plugin>
        <groupId>org.jvnet.jax-ws-commons</groupId>
        <artifactId>jaxws-maven-plugin</artifactId>
        <version>2.5.0</version>
        <executions>
            <execution>
                <goals>
                    <goal>wsimport</goal>
                </goals>
                <configuration>
                    <wsdlUrls>
                        <wsdlUrl>http://localhost:8080/services/common?wsdl</wsdlUrl>
                    </wsdlUrls>
                    <packageName>com.example.webservice.stub</packageName>
                    <sourceDestDir>src/main/java</sourceDestDir>
                    <verbose>true</verbose>
                    <keep>true</keep>
                </configuration>
            </execution>
        </executions>
        <dependencies>
            <dependency>
                <groupId>com.sun.xml.ws</groupId>
                <artifactId>jaxws-tools</artifactId>
                <version>2.3.1</version>
            </dependency>
        </dependencies>
    </plugin>
</plugins>

<!-- 执行命令 -->
<!-- mvn jaxws:wsimport -->
```

**C. Gradle 插件模式（jax-ws-tools）**

```groovy
// build.gradle
buildscript {
    repositories {
        mavenCentral()
        maven { url 'https://plugins.gradle.org/m2/' }
    }
    dependencies {
        classpath 'com.sun.xml.ws:jaxws-tools:2.3.1'
    }
}

task wsImport(type: JavaExec) {
    classpath = sourceSets.main.compileClasspath
    mainClass = 'com.sun.tools.ws.wscompile.WsimportTool'
    args = [
        '-keep',
        '-d', 'build/generated/sources/jaxws',
        '-p', 'com.example.webservice.stub',
        'http://localhost:8080/services/common?wsdl'
    ]
}

// 执行命令
// gradle wsImport
```

###### 2.1.2.2 Spring Boot 3.5 环境

**A. 命令行模式（wsdl2java - Apache CXF）**

```bash
# 基本用法：生成客户端代码
wsdl2java -client -d src/main/java http://localhost:8080/services/common?wsdl

# 指定包名
wsdl2java -p com.example.webservice.stub -d src/main/java http://localhost:8080/services/common?wsdl

# 生成服务器端和客户端代码
wsdl2java -server -client -d src/main/java http://localhost:8080/services/common?wsdl

# 使用 UTF-8 编码
wsdl2java -encoding UTF-8 -d src/main/java http://localhost:8080/services/common?wsdl

# 详细输出
wsdl2java -verbose -d src/main/java http://localhost:8080/services/common?wsdl

# 从本地 WSDL 文件生成
wsdl2java -client -d src/main/java src/main/wsdl/commonService.wsdl
```

**B. Maven 插件模式（cxf-codegen-plugin）**

```xml
<!-- pom.xml -->
<plugins>
    <plugin>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-codegen-plugin</artifactId>
        <version>3.5.1</version>
        <executions>
            <execution>
                <id>generate-sources</id>
                <phase>generate-sources</phase>
                <goals>
                    <goal>wsdl2java</goal>
                </goals>
                <configuration>
                    <wsdlOptions>
                        <wsdlOption>
                            <wsdl>http://localhost:8080/services/common?wsdl</wsdl>
                            <packagename>com.example.webservice.stub</packagename>
                            <wsdlLocation>classpath:wsdl/commonService.wsdl</wsdlLocation>
                            <extraargs>
                                <extraarg>-client</extraarg>
                                <extraarg>-encoding</extraarg>
                                <extraarg>UTF-8</extraarg>
                            </extraargs>
                        </wsdlOption>
                    </wsdlOptions>
                </configuration>
            </execution>
        </executions>
        <dependencies>
            <dependency>
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-rt-frontend-jaxws</artifactId>
                <version>3.5.1</version>
            </dependency>
        </dependencies>
    </plugin>
</plugins>

<!-- 执行命令 -->
<!-- mvn cxf-codegen:wsdl2java -->
```

**C. Gradle 插件模式（io.mateo.cxf-codegen）**

```groovy
// build.gradle (Spring Boot 3.5 + CXF 4.1.x)
plugins {
    id 'java'
    id 'io.mateo.cxf-codegen' version '2.5.0'
}

import io.mateo.cxf.codegen.wsdl2java.Wsdl2Java

tasks.register('generateCommonService', Wsdl2Java) {
    group = 'build'
    description = '根据 WSDL 生成 Java 客户端代码（CXF wsdl2java）'
    toolOptions {
        wsdl = file('src/main/wsdl/commonService.wsdl').absolutePath
        packageNames = ['com.example.webservice.stub']
        outputDir = layout.buildDirectory.dir('generated/sources/cxf').get().asFile
        extraArgs.addAll(['-encoding', 'UTF-8', '-client'])
    }
}

compileJava.dependsOn tasks.withType(Wsdl2Java)

// 执行命令
// gradle generateCommonService
// 或直接 gradle build（自动触发）
```

**D. Gradle 插件模式（Spring Boot 2.5 + CXF 3.x）**

```groovy
// build.gradle (Spring Boot 2.5 + CXF 3.x)
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.apache.cxf:cxf-tools-wsdlto-java:3.4.0'
    }
}

task wsdl2java(type: JavaExec) {
    classpath = sourceSets.main.runtimeClasspath
    mainClass = 'org.apache.cxf.tools.wsdlto.WSDLToJava'
    args = [
        '-d', 'build/generated/sources/cxf',
        '-p', 'com.example.webservice.stub',
        '-client',
        '-encoding', 'UTF-8',
        'http://localhost:8080/services/common?wsdl'
    ]
}

compileJava.dependsOn wsdl2java

// 执行命令
// gradle wsdl2java
```

---

##### 方式三：动态调用（运行时动态生成客户端）

**必要条件：**
1. WebService 服务端发布地址
2. 调用方法使用的参数和返回值
3. 如果参数或返回值是实体类，需要根据指定的 namespace 创建实体类

**核心原理：** 使用 `JaxWsDynamicClientFactory` 在运行时动态创建客户端代理，无需提前生成代码。

**适用场景：** 临时测试、快速开发、不需要类型安全的场景。

**优点：** 无需生成代码、运行时动态创建、简单快捷。
**缺点：** 每次调用都需要创建代理、没有类型安全。

**完整代码案例（Spring Boot 3.5 环境）：**

```java
package com.example.client.dynamic;

import org.apache.cxf.jaxws.JaxWsDynamicClientFactory;
import org.apache.cxf.endpoint.Client;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

/**
 * 方式三：动态调用客户端
 * 
 * 使用 JaxWsDynamicClientFactory 在运行时动态创建客户端代理
 * 无需提前生成代码，适合临时测试和快速开发
 */
@Service
public class DynamicWebServiceClient {

    @Value("${example.service.url:http://localhost:8080/services/common}")
    private String serviceUrl;

    @Value("${example.service.namespace:http://webservices.modules.demo.rose.small.com/}")
    private String serviceNamespace;

    @Value("${example.service.interface:com.example.webservice.stub.common.CommonService}")
    private String serviceInterface;

    /**
     * 创建动态客户端
     */
    public Client createDynamicClient() {
        JaxWsDynamicClientFactory factory = JaxWsDynamicClientFactory.newInstance();
        return factory.createClient(serviceUrl + "?wsdl", serviceInterface, null);
    }

    /**
     * 调用方法（通用方法）
     * 
     * @param methodName 方法名
     * @param args 参数
     * @return 返回结果
     */
    public Object[] invokeMethod(String methodName, Object... args) throws Exception {
        Client client = createDynamicClient();
        try {
            return client.invoke(methodName, args);
        } finally {
            client.close();
        }
    }

    /**
     * 获取用户信息
     */
    public String getUserInfo(String userId) throws Exception {
        Object[] result = invokeMethod("getUserInfo", userId);
        return result.length > 0 ? (String) result[0] : null;
    }

    /**
     * 发送短信
     */
    public String sendSms(String phone, String message) throws Exception {
        Object[] result = invokeMethod("sendSms", phone, message);
        return result.length > 0 ? (String) result[0] : null;
    }

    /**
     * 同步数据
     */
    public boolean syncData(String data) throws Exception {
        Object[] result = invokeMethod("syncData", data);
        return result.length > 0 && (Boolean) result[0];
    }

    /**
     * 测试方法
     */
    public void testAllMethods() throws Exception {
        System.out.println("=== 动态调用测试 ===");

        // 测试 getUserInfo
        String userInfo = getUserInfo("user001");
        System.out.println("用户信息：" + userInfo);

        // 测试 sendSms
        String smsResult = sendSms("13800000000", "动态调用测试");
        System.out.println("短信结果：" + smsResult);

        // 测试 syncData
        boolean syncResult = syncData("动态同步数据");
        System.out.println("同步结果：" + syncResult);
    }
}
```

**测试类：**

```java
package com.example.client.dynamic;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DynamicWebServiceClientTest {

    @Autowired
    private DynamicWebServiceClient dynamicClient;

    @Test
    void testGetUserInfo() throws Exception {
        String result = dynamicClient.getUserInfo("user001");
        System.out.println("用户信息：" + result);
    }

    @Test
    void testSendSms() throws Exception {
        String result = dynamicClient.sendSms("13800000000", "测试短信");
        System.out.println("短信结果：" + result);
    }

    @Test
    void testAllMethods() throws Exception {
        dynamicClient.testAllMethods();
    }
}
```

---

##### 方式四：代理工厂方式（JAX-WS Static Proxy）

**必要条件：**
1. WebService 服务端发布地址
2. WebService 服务发布的接口类
3. 参数和返回值实体类

**核心原理：** 使用 `JaxWsProxyFactoryBean` 创建静态代理对象，通过代理对象调用 WebService 方法。

**适用场景：** 生产环境、常规调用、需要类型安全和性能的场景。

**优点：** 类型安全、性能好、Spring 容器管理、生产级稳定性。
**缺点：** 需要提前准备接口类、需要生成客户端代码。

**完整代码案例（Spring Boot 3.5 环境）：**

```java
package com.example.client.config;

import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 方式四：代理工厂客户端配置
 * 
 * 使用 JaxWsProxyFactoryBean 创建静态代理对象
 * 生产环境推荐，类型安全、性能好
 */
@Configuration
public class ProxyFactoryClientConfig {

    @Value("${example.service.url:http://localhost:8080/services/common}")
    private String serviceUrl;

    @Value("${example.service.connect-timeout:5000}")
    private int connectTimeout;

    @Value("${example.service.receive-timeout:10000}")
    private int receiveTimeout;

    /**
     * 创建 CommonService 代理对象
     */
    @Bean
    public com.example.webservice.stub.common.CommonService commonService() {
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();

        // 设置服务地址
        factory.setAddress(serviceUrl);

        // 设置服务接口
        factory.setServiceClass(com.example.webservice.stub.common.CommonService.class);

        // 设置超时配置
        factory.setReceiveTimeout(receiveTimeout);
        factory.setConnectionTimeout(connectTimeout);

        // 创建代理对象
        return (com.example.webservice.stub.common.CommonService) factory.create();
    }

    /**
     * 创建第二个 WebService 代理（如果需要多个）
     */
    @Bean
    public com.example.webservice.stub.order.OrderService orderService() {
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();
        factory.setAddress("http://localhost:8080/services/orderService");
        factory.setServiceClass(com.example.webservice.stub.order.OrderService.class);
        factory.setReceiveTimeout(receiveTimeout);
        factory.setConnectionTimeout(connectTimeout);
        return (com.example.webservice.stub.order.OrderService) factory.create();
    }
}
```

**调用示例：**

```java
package com.example.client.service;

import com.example.webservice.stub.common.CommonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 方式四：代理工厂调用示例
 */
@Service
public class ProxyFactoryClientService {

    @Autowired
    private CommonService commonService;

    /**
     * 获取用户信息
     */
    public String getUserInfo(String userId) {
        try {
            return commonService.getUserInfo(userId);
        } catch (Exception e) {
            throw new RuntimeException("调用 getUserInfo 失败", e);
        }
    }

    /**
     * 发送短信
     */
    public String sendSms(String phone, String message) {
        try {
            return commonService.sendSms(phone, message);
        } catch (Exception e) {
            throw new RuntimeException("调用 sendSms 失败", e);
        }
    }

    /**
     * 同步数据
     */
    public boolean syncData(String data) {
        try {
            return commonService.syncData(data);
        } catch (Exception e) {
            throw new RuntimeException("调用 syncData 失败", e);
        }
    }
}
```

**测试类：**

```java
package com.example.client;

import com.example.client.config.ProxyFactoryClientConfig;
import com.example.client.service.ProxyFactoryClientService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(classes = ProxyFactoryClientConfig.class)
class ProxyFactoryClientTest {

    @Autowired
    private ProxyFactoryClientService clientService;

    @Test
    void testGetUserInfo() {
        String result = clientService.getUserInfo("user001");
        System.out.println("用户信息：" + result);
    }

    @Test
    void testSendSms() {
        String result = clientService.sendSms("13800000000", "代理工厂测试");
        System.out.println("短信结果：" + result);
    }

    @Test
    void testSyncData() {
        boolean result = clientService.syncData("代理工厂同步数据");
        System.out.println("同步结果：" + result);
    }
}
```

---

##### 方式五：HttpClient + SoapUI 方式

**必要条件：**
1. WebService 服务端发布地址
2. SoapUI 工具（获取请求 XML 格式）
3. HttpClient 库（发送 HTTP 请求）
4. XML 解析库（解析响应）

**核心原理：** 使用 SoapUI 获取 SOAP 请求 XML 格式，通过 HttpClient 发送请求，手工解析 XML 响应。

**适用场景：** 手工测试、调试、需要精确控制 SOAP 消息格式的场景。

**优点：** 灵活、可视化调试（SoapUI）、精确控制消息格式。
**缺点：** 开发复杂度高、需要手工解析 XML、依赖第三方工具。

**完整代码案例（Spring Boot 3.5 环境）：**

```java
package com.example.client.http;

import org.apache.hc.client5.http.classic.methods.HttpPost;
import org.apache.hc.client5.http.entity.classic.StringEntity;
import org.apache.hc.client5.http.impl.classic.CloseableHttpClient;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.apache.hc.core5.http.ContentType;
import org.apache.hc.core5.http.io.entity.EntityUtils;
import org.apache.hc.core5.http.io.support.ClassicRequestBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.NodeList;
import java.io.ByteArrayInputStream;

/**
 * 方式五：HttpClient + SoapUI 方式
 * 
 * 使用 Apache HttpClient 发送 SOAP 请求
 * 需要手工构建 SOAP XML 格式
 */
@Service
public class HttpClientSoapClient {

    @Value("${example.service.url:http://localhost:8080/services/common}")
    private String serviceUrl;

    @Value("${example.service.namespace:http://webservices.modules.demo.rose.small.com/}")
    private String serviceNamespace;

    /**
     * 使用 SoapUI 获取的请求格式调用 getUserInfo
     * 
     * SoapUI 中获取的请求格式示例：
     * <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
     *                   xmlns:ser="http://webservices.modules.demo.rose.small.com/">
     *   <soapenv:Header/>
     *   <soapenv:Body>
     *     <ser:getUserInfo>
     *       <arg0>user001</arg0>
     *     </ser:getUserInfo>
     *   </soapenv:Body>
     * </soapenv:Envelope>
     */
    public String getUserInfo(String userId) throws Exception {
        // 构建 SOAP 请求（从 SoapUI 复制的格式）
        String soapRequest = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
            "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" " +
            "xmlns:ser=\"" + serviceNamespace + "\">" +
            "  <soapenv:Header/>" +
            "  <soapenv:Body>" +
            "    <ser:getUserInfo>" +
            "      <arg0>" + userId + "</arg0>" +
            "    </ser:getUserInfo>" +
            "  </soapenv:Body>" +
            "</soapenv:Envelope>";

        // 发送请求
        String soapResponse = sendSoapRequest(soapRequest, "getUserInfo");

        // 解析响应
        return parseResponse(soapResponse, "userInfo");
    }

    /**
     * 使用 SoapUI 获取的请求格式调用 sendSms
     */
    public String sendSms(String phone, String message) throws Exception {
        String soapRequest = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
            "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" " +
            "xmlns:ser=\"" + serviceNamespace + "\">" +
            "  <soapenv:Header/>" +
            "  <soapenv:Body>" +
            "    <ser:sendSms>" +
            "      <arg0>" + phone + "</arg0>" +
            "      <arg1>" + message + "</arg1>" +
            "    </ser:sendSms>" +
            "  </soapenv:Body>" +
            "</soapenv:Envelope>";

        String soapResponse = sendSoapRequest(soapRequest, "sendSms");
        return parseResponse(soapResponse, "sendSmsResponse");
    }

    /**
     * 使用 SoapUI 获取的请求格式调用 syncData
     */
    public boolean syncData(String data) throws Exception {
        String soapRequest = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
            "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" " +
            "xmlns:ser=\"" + serviceNamespace + "\">" +
            "  <soapenv:Header/>" +
            "  <soapenv:Body>" +
            "    <ser:syncData>" +
            "      <arg0>" + data + "</arg0>" +
            "    </ser:syncData>" +
            "  </soapenv:Body>" +
            "</soapenv:Envelope>";

        String soapResponse = sendSoapRequest(soapRequest, "syncData");
        String result = parseResponse(soapResponse, "syncDataResponse");
        return "true".equalsIgnoreCase(result);
    }

    /**
     * 发送 SOAP 请求
     */
    private String sendSoapRequest(String soapRequest, String soapAction) throws Exception {
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            HttpPost httpPost = new HttpPost(serviceUrl);
            httpPost.setHeader("Content-Type", "text/xml; charset=UTF-8");
            httpPost.setHeader("SOAPAction", soapAction);
            httpPost.setEntity(new StringEntity(soapRequest, ContentType.create("text/xml", "UTF-8")));

            return httpClient.execute(httpPost, response -> {
                int statusCode = response.getCode();
                if (statusCode != 200) {
                    throw new RuntimeException("SOAP 请求失败，状态码：" + statusCode);
                }
                return EntityUtils.toString(response.getEntity(), "UTF-8");
            });
        }
    }

    /**
     * 解析 SOAP 响应
     */
    private String parseResponse(String soapResponse, String elementName) throws Exception {
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        factory.setNamespaceAware(true);
        DocumentBuilder builder = factory.newDocumentBuilder();
        Document document = builder.parse(new ByteArrayInputStream(soapResponse.getBytes("UTF-8")));

        NodeList nodeList = document.getElementsByTagNameNS("*", elementName);
        if (nodeList.getLength() > 0) {
            return nodeList.item(0).getTextContent();
        }
        return null;
    }
}
```

**测试类：**

```java
package com.example.client.http;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class HttpClientSoapClientTest {

    @Autowired
    private HttpClientSoapClient httpClient;

    @Test
    void testGetUserInfo() throws Exception {
        String result = httpClient.getUserInfo("user001");
        System.out.println("用户信息：" + result);
    }

    @Test
    void testSendSms() throws Exception {
        String result = httpClient.sendSms("13800000000", "HttpClient测试");
        System.out.println("短信结果：" + result);
    }

    @Test
    void testSyncData() throws Exception {
        boolean result = httpClient.syncData("HttpClient同步数据");
        System.out.println("同步结果：" + result);
    }
}
```

**SoapUI 使用步骤：**

```
1. 下载并安装 SoapUI（https://www.soapui.org/）
2. 新建项目，输入 WSDL 地址：http://localhost:8080/services/common?wsdl
3. 双击接口方法，查看请求 XML 格式
4. 复制请求 XML 到 Java 代码中
5. 修改参数值，发送请求
6. 查看响应 XML，解析结果
```

---

##### 🔍 5 种方式差异总结

| 方式 | 技术栈 | 必要条件 | 开发复杂度 | 类型安全 | 性能 | 生产推荐 |
|------|--------|----------|------------|----------|------|----------|
| **方式一：JDK 原生** | URLConnection、SAX/DOM | 发布地址、接口类、实体类 | 极高 | 差 | 一般 | ❌ |
| **方式二：wsimport/wsdl2java** | JDK/CXF 代码生成工具 | 发布地址、WSDL 契约 | 中等 | 优秀 | 优秀 | ✅ |
| **方式三：动态调用** | CXF 动态客户端工厂 | 发布地址、方法参数 | 低 | 差 | 一般 | ❌ |
| **方式四：代理工厂** | CXF 静态代理工厂 | 发布地址、接口类、实体类 | 中等 | 优秀 | 优秀 | ✅ |
| **方式五：HttpClient+SoapUI** | HttpClient、SoapUI、XML 解析 | 发布地址、SoapUI、HttpClient | 高 | 差 | 一般 | ⚠️ |

##### 📊 按使用场景推荐

| 场景 | 推荐方式 | 原因 |
|------|----------|------|
| **生产环境常规调用** | 方式四（代理工厂） | 类型安全、性能好、Spring 管理 |
| **开发环境快速测试** | 方式三（动态调用） | 无需生成代码、简单快捷 |
| **标准 WebService 项目** | 方式二（wsimport/wsdl2java） | 标准化、类型安全 |
| **手工测试和调试** | 方式五（HttpClient+SoapUI） | 可视化调试、精确控制 |
| **最小依赖环境** | 方式一（JDK 原生） | 不依赖第三方库 |

##### ⚠️ 两种分类对比

| 分类 | 提出者 | 侧重点 | 技术栈覆盖 |
|------|--------|--------|------------|
| **分类 A（5 种方式）** | 用户 | 技术手段分类 | 涵盖原生API、代码生成、第三方工具等 |
| **分类 B（5 种方式）** | 教程 | 基于 Apache CXF | 完全基于 CXF 生态 |

**重叠部分：**
- 方式三（动态调用） ≈ 分类 B 的方式一（动态代理）
- 方式四（代理工厂） ≈ 分类 B 的方式二（静态代理）

**差异部分：**
- 分类 A 包含：JDK 原生、wsimport/wsdl2java、HttpClient+SoapUI
- 分类 B 包含：WebServiceTemplate、POJO 客户端、拦截器+回调

##### 💡 最佳实践建议

1. **新项目首选**：方式四（代理工厂） + 方式二（wsimport/wsdl2java）
2. **现有项目维护**：根据已有技术栈选择
3. **特殊需求**：方式五（HttpClient+SoapUI）用于手工调试
4. **快速验证**：方式三（动态调用）用于临时测试

---

#### 2.2 客户端项目结构

```
client-project/
├── src/main/java/com/example/client/
│   ├── WebServiceClient.java          ← 客户端启动类
│   ├── WebServiceTest.java           ← 客户端测试类
│   └── config/WebServiceConfig.java   ← CXF 客户端配置
├── src/main/resources/application.yml   ← Spring 配置
└── build.gradle                        ← 依赖配置
```

#### 2.2 WebServiceClient.java (客户端启动类)

```java
package com.example.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * WebServiceClient - CXF WebService 客户端启动类
 * 
 * 启动 Spring Boot 应用，初始化 WebService 客户端
 */
@SpringBootApplication
public class WebServiceClient {

    public static void main(String[] args) {
        System.out.println("正在启动 CXF WebService 客户端...");
        System.out.println("服务地址：http://localhost:8080/services/common");
        System.out.println("WSDL 地址：http://localhost:8080/services/common?wsdl");
        
        SpringApplication.run(WebServiceClient.class, args);
        
        System.out.println("CXF WebService 客户端启动成功！");
        System.out.println("\n可用 CLI 命令：");
        System.out.println("  curl http://localhost:8080/actuator/health");
        System.out.println("  curl \"http://localhost:8080/services/common\" -X POST -H \"Content-Type: text/xml\" -d '<soapenv:Envelope ...>'");
    }
}
```

#### 2.3 WebServiceConfig.java (客户端配置)

```java
package com.example.client.config;

import com.example.webservice.stub.common.CommonService;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * WebServiceConfig - CXF WebService 客户端配置
 * 
 * 负责配置和创建 CommonService 的代理对象
 */
@Configuration
public class WebServiceConfig {

    @Value("${example.service.url:http://localhost:8080/services/common}")
    private String serviceUrl;

    /**
     * 创建 CommonService 代理对象
     * 
     @Bean 注解确保代理对象作为 Spring Bean 管理
     * JaxWsProxyFactoryBean 创建 WebService 客户端代理
     */
    @Bean
    public CommonService commonService() {
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();
        
        // 设置服务地址
        factory.setAddress(serviceUrl);
        
        // 设置服务接口
        factory.setServiceClass(CommonService.class);
        
        // 设置代理超时配置
        factory.setReceiveTimeout(10000);
        factory.setConnectionTimeout(5000);
        
        // 创建代理对象
        return factory.create();
    }
}
```

#### 2.4 WebServiceTest.java (客户端测试)

```java
package com.example.client;

import com.example.client.config.WebServiceConfig;
import com.example.webservice.stub.common.CommonService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

import static org.junit.jupiter.api.Assertions.*;

/**
 * WebServiceTest - CXF WebService 客户端测试类
 * 
 * 演示 WebService 的各种调用方式
 */
@SpringBootTest(classes = {WebServiceClient.class, WebServiceConfig.class})
@ActiveProfiles("test")
class WebServiceTest {

    @Autowired
    private CommonService commonService;

    @Test
    void testGetUserInfo() {
        // 方式 1：基本调用
        String result = commonService.getUserInfo("user001");
        
        assertNotNull(result);
        assertTrue(result.contains("userId"));
        assertTrue(result.contains("张三"));
        
        System.out.println("=== 获取用户信息 ===");
        System.out.println("返回结果：" + result);
    }

    @Test
    void testSendSms() {
        // 方式 2：参数验证
        String result = commonService.sendSms("13800000000", "测试短信内容");
        
        assertNotNull(result);
        assertTrue(result.contains("success"));
        assertTrue(result.contains("messageId"));
        
        System.out.println("=== 发送短信 ===");
        System.out.println("返回结果：" + result);
    }

    @Test
    void testSyncData() {
        // 方式 3：异常处理
        boolean result = commonService.syncData("同步数据内容");
        
        assertTrue(result);
        
        System.out.println("=== 同步数据 ===");
        System.out.println("同步结果：" + result);
    }

    @Test
    void testComplexOperation() {
        // 方式 4：链式调用
        String userInfo = commonService.getUserInfo("user002");
        String smsResult = commonService.sendSms("13900000000", "链式测试");
        boolean syncResult = commonService.syncData("链式数据");
        
        assertAll("复杂操作",
            () -> assertNotNull(userInfo),
            () -> assertNotNull(smsResult),
            () -> assertTrue(syncResult)
        );
        
        System.out.println("=== 复杂操作 ===");
        System.out.println("用户信息：" + userInfo);
        System.out.println("短信结果：" + smsResult);
        System.out.println("同步结果：" + syncResult);
    }
}
```

#### 2.5 客户端 build.gradle

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.14'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.example'
version = '1.0.0'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

dependencies {
    // Spring Boot 启动器
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // CXF WebService 客户端依赖
    implementation 'org.apache.cxf:cxf-spring-boot-starter-jaxws:4.1.6'
    
    // Jackson 数据转换
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.18.4'
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-json:2.18.4'
    
    // 测试
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

#### 2.6 application.yml (客户端配置)

```yaml
# application.yml - Spring Boot 客户端配置

server:
  port: 18080
  servlet:
    context-path: /

spring:
  application:
    name: spring-boot3-cxf-webservice-client
  jackson:
    property-naming-strategy: SNAKE_CASE
    time-zone: GMT+8
  profiles:
    active: test

# CXF WebService 客户端配置
example:
  service:
    url: http://localhost:8080/services/common
    timeout:
      connect: 5000
      read: 10000

# Spring Boot Actuator 配置
management:
  endpoints:
    web:
      base-path: /actuator
    jmx:
      enabled: false
  endpoint:
    health:
      show-details: always

# 日志配置
logging:
  level:
    com.example: DEBUG
    org.apache.cxf: INFO
    org.springframework.web: INFO
```

## 总结

### 1. CXF JAX-WS 在 Spring Boot 3 中的优势

1. **标准规范**：符合 SOAP 1.1/1.2 和 WSDL 1.1 标准
2. **企业级支持**：完整的 Spring 生态集成
3. **双向契约**：WSDL 驱动代码生成
4. **协议复用**：支持 SOAP、HTTP、JAXB 等
5. **成熟生态**：丰富的工具和中间件

### 2. 应用场景建议

| 场景 | 推荐方案 | 原因 |
|----------|---------------|-----|
| **企业系统集成** | CXF JAX-WS | 标准协议，成熟生态 |
| **轻量级微服务** | Spring MVC/REST | 性能更高，开发成本低 |
| **混合协议系统** | CXF + Spring MVC | 灵活多样，支持多种协议 |
| **大数据处理系统** | CXF + Kafka | 支持复杂消息处理 |

### 3. 最佳实践

1. **服务设计**：确保接口设计符合 WSDL 标准
2. **版本管理**：为 WebService 版本化管理
3. **错误处理**：实现 SOAP 故障处理策略
4. **安全配置**：使用 WS-Security、SSL 等安全机制
5. **监控与日志**：启用 Spring Boot Actuator，监控 WebService 调用

### 4. 学习路径

1. **入门**：学习 WSDL、SOAP、JAX-WS 基础知识
2. **实践**：按照教程搭建完整案例
3. **扩展**：学习高级特性（拦截器、HandlerChain 等）
4. **生产**：学习 CI/CD、监控、部署等实战经验

Apache CXF + Spring Boot 3 的 WebService 开发为企业级系统集成提供了强大的支持。通过 WSDL 驱动的方式，可以确保接口的一致性和互操作性。希望大家能够熟练掌握，并在实际项目中应用。祝大家学习愉快！🎉
