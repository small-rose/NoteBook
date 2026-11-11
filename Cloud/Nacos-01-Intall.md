---
layout: default
title: Nacos 3.X dev 
parent: Cloud
nav_order: 11
---


## 一、环境前置要求

|组件 |要求|
|----|----|
|JDK |17 及以上（Nacos 3.x 强制，推荐 JDK 21）|
|系统 |Windows 10/11 64 位，管理员 CMD|
|内存 |开发机建议 ≥ 2C 4G|
|MySQL |5.6.5+（推荐 8.0，持久化用）|
|路径 |不要含中文、空格、特殊字符|

>    ⚠️ Nacos 3.x 默认开启 Client / Admin / Console 鉴权，首次启动会要求生成 token 密钥

## 二、Nacos 3.3.0 核心概念梳理

一些核心概念如下表

|维度	|取值	|决定方式	|说明|
|-------|-------|--------|-----|
|部署模式（拓扑）|	standalone / cluster	|启动参数 -m	|单机 vs 集群，决定一致性协议与高可用形态|
|部署形态（加载模块）|	merged / server / console	|启动参数 -d 或 -Dnacos.deployment.type	|merged=引擎+UI 一体（默认）；server=只有核心引擎无控制台；console=只有 UI|
|功能模式（能力集）|	all / config / naming / microservice / ai	|启动参数 -f	|microservice=仅 Config+Naming，不加载 AI 模块，本地开发最省资源|
|存储|	Derby（内嵌）/ MySQL（外置）	|conf/application.properties	|单机默认 Derby；集群或需持久化走 MySQL|

>四个维度正交组合，互相不绑定。比如可以：standalone + merged + microservice + derby，也可以 cluster + server + all + mysql。


### 1、部署模式（Deployment Mode）

|模式  |启动参数  |存储 |用途|
|-----|--------|-----|----|
|Standalone 单机| -m standalone |默认 Derby |本地开发、调试|
|Cluster 集群 | 默认（无参数） |必须外置 MySQL |生产、高可用|

> 不写 -m standalone 会按集群模式启动，没配 cluster.conf 和外置库会直接失败

### 2、部署形态 

- merged 
- server 
- console 

三种形态的部署端口差异

|形态|	8080(UI)	|8848(HTTP)	|9848(gRPC)	|9849(同步)|
|---|----|---|---|---|
|merged	|✅	|✅	|✅	|✅|
|server	|❌|	✅	|✅	|✅|
|console	|✅	|❌	|❌	|❌|

>本地开发用 merged 最省心，一个进程全有。

#### 2.1. 本地开发「最省资源」推荐组合

启动命令：
```bat
startup.cmd -m standalone -d merged -f microservice
```
- 单机拓扑
- 引擎+控制台一体
- 只加载配置中心+注册中心，不加载 MCP/Agent/Skill 等 AI 模块
- 存储走默认 Derby，零外部依赖

#### 2.2. 纯后端引擎（无 UI，CI / 被远程连）

启动命令：
```bat
startup.cmd -m standalone -d server -f microservice
```
- 不启 8080 控制台
- 只提供 8848/9848 接口
- 适合“本机跑引擎，浏览器连别的机器控制台”或自动化测试

#### 2.3. 只要控制台 UI（引擎在别处）

启动命令：
```bat
startup.cmd -m standalone -d console
```

- 只起 UI，连远端 server
- 一般配合 Nacos Console 指向外部引擎用，本地少用

#### 2.4. 全量模式（含 AI Registry）

启动命令：
```bat
startup.cmd -m standalone -d merged -f all
```

- 加载 config / naming / ai（MCP、Prompt、Skill、Agent）
- 内存占用明显更高，本地笔记本慎开

#### 2.5 集群 + 外置 MySQL（生产形态）

生产形态，本地可用 3 个端口模拟 3 节点。

(1)准备3份解压目录
```text
D:\nacos\node1
D:\nacos\node2
D:\nacos\node3
```
(2)每个节点 conf/cluster.conf

```text
127.0.0.1:8848
127.0.0.1:8858
127.0.0.1:8868
```
(3)每个节点 application.properties 差异项

node1
```text
server.port=8848
nacos.plugin.datasource.db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?
```
node2
```text
server.port=8858
nacos.plugin.datasource.db.url.1=jdbc:mysql://127.0.0.1:3306/nacos_config?
```
node3
```text
server.port=8868
nacos.plugin.datasource.db.url.2=jdbc:mysql://127.0.0.1:3306/nacos_config?
```
> （都连同一个 MySQL 库，token/identity 配置完全一致）

(4) 分别启动
```bat
cd D:\nacos\node1\bin && startup.cmd
cd D:\nacos\node2\bin && startup.cmd
cd D:\nacos\node3\bin && shutdown.cmd
```

注意：
```
每个节点都不加 -m standalone，靠 cluster.conf 识别

startup.cmd -d server -f microservice
```
> 不添加 -m 便默认走集群模式，靠读取conf/cluster.conf文件识别集群


### 3. 功能模式（Capability）
   
Nacos 同时承担两个角色，可单独用也可一起用：

- 配置中心（Config）：mysql-schema.sql 里的 config_* 表
- 注册中心（Naming/Discovery）：service_*、instance_* 表

只启动作为**配置中心**：
```bat
:: 只配置中心
startup.cmd -m standalone -d merged -f config
```

只启动作为**注册中心**：
```bat
:: 只注册中心
startup.cmd -m standalone -d merged -f naming
```

### 4. 存储模式

|存储 |说明 |适用|
|---|----|----|
|Derby（内嵌） |文件型数据库，单机可用 |本地开发|
|MySQL（外置） |多节点共享、持久化 |测试/生产必选|

> Nacos 3.3 数据源配置键已升级为插件式：nacos.plugin.datasource.*

#### 4.1. Derby（默认，什么都不用配）
   
- -f microservice 默认就用内嵌 Derby，数据落 data/derby。
- 限制：不能集群、不能多进程共享。

#### 4.2. MySQL 外置（推荐测试/联调）

4.2.1 建库并初始化表

MySQL里先建库：

```sql
CREATE DATABASE nacos_config DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci;
USE nacos_config;
```
再执行 conf/mysql-schema.sql，再启动：


4.2.2. 改 conf/application.properties

```properties
nacos.plugin.datasource-dialect.type=mysql
nacos.plugin.datasource.db.num=1
nacos.plugin.datasource.db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&serverTimezone=UTC
nacos.plugin.datasource.db.user=root
nacos.plugin.datasource.db.password=rootpass

# 3.x 鉴权必须
nacos.core.auth.enabled=true
nacos.core.auth.console.enabled=true
nacos.plugin.auth.type=nacos

# token 密钥：Base64，解码后 ≥32 字节，各节点必须一致
nacos.core.auth.plugin.nacos.token.secret.key=自定义Base64(≥32字节)

# 节点身份
nacos.core.auth.server.identity.key=serverIdentity
nacos.core.auth.server.identity.value=自定义值
```

```bat
startup.cmd -m standalone -d merged -f microservice
```

>哪怕服务端是 -f microservice，配置中心和注册中心都可用；只有用到 AI Registry 时才需要 -f all + 对应 SDK。

### 5. 端口矩阵（3.x 很重要）

|端口 |偏移 |作用|
|----|----|----|
|8080 |独立 |控制台 UI（3.x 新分离）|
|8848 |0 |HTTP OpenAPI / Admin API|
|9848 |+1000 |客户端 gRPC|
|9849 |+1001 |服务端 gRPC 同步|
|7848 |-1000 |Raft/Jraft 通信|

访问控制台用 http://localhost:8080 ，不是老教程里的 8848/nacos


本地开发方案：

```bat
:: 一键启动：单机 + 一体部署 + 仅微服务模块 + Derby
cd D:\nacos\nacos-3.3.0\bin
startup.cmd -m standalone -d merged -f microservice
```
- 控制台：http://localhost:8080
- 需要持久化再切 MySQL，需要 AI 能力再改 -f all。


**组合速查表**

|场景	|命令|
|------|-----|
|本地最简	|startup.cmd -m standalone -d merged -f microservice |
|本地全量	|startup.cmd -m standalone -d merged -f all |
|无 UI 引擎	|startup.cmd -m standalone -d server -f microservice |
|仅配置	|startup.cmd -m standalone -d merged -f config |
|仅注册	|startup.cmd -m standalone -d merged -f naming |
|集群	|startup.cmd -d server -f microservice（配 cluster.conf + MySQL）|

## 三、下载与安装

### 3.1 解压安装

官网下载：https://github.com/alibaba/nacos/releases/tag/3.3.0

选：nacos-server-3.3.0.zip

解压到：
> D:\nacos\nacos-3.3.0

目录结构关注两个：

> bin\        ← startup.cmd / shutdown.cmd
> conf\       ← application.properties / cluster.conf / mysql-schema.sql

### 3.2 docker安装

```bat
docker run --name nacos-standalone ^
  -e MODE=standalone ^
  -e NACOS_AUTH_TOKEN=自定义Base64 ^
  -e NACOS_AUTH_IDENTITY_KEY=serverIdentity ^
  -e NACOS_AUTH_IDENTITY_VALUE=自定义值 ^
  -p 8080:8080 -p 8848:8848 -p 9848:9848 ^
  -d nacos/nacos-server:v3.3.0
```
MySQL 外挂用 `-v` 挂 `application.properties` 。

## 访问凭据配置

### 首次启动：控制台强制设置管理员密码

从 Nacos 3.0 开始，控制台默认启用鉴权。首次启动后访问 http://localhost:8080，会跳转到初始化页面，要求设置 nacos 管理员密码：

- 输入新密码（至少 8 位，建议包含字母数字特殊字符）
- 确认密码

点击提交。密码修改后，后续登录都用新密码。

> ⚠️ 如果跳过此步骤（例如通过 API 直接调用），Nacos 会使用内置默认密码 nacos，但这极不安全，生产环境必须修改。


### 核心鉴权配置项（application.properties）

无论是否使用控制台，服务端必须配置以下三项，否则客户端无法通过鉴权：

```properties

# ---------- Token 密钥（必须自定义） ----------
# 生成方式见下方
nacos.core.auth.plugin.nacos.token.secret.key=VGhpcyBpcyBhIHNhbXBsZSBrZXkgZm9yIG5hY29z...

# ---------- 节点身份标识（集群时各节点必须一致） ----------
nacos.core.auth.server.identity.key=serverIdentity
nacos.core.auth.server.identity.value=custom_value
```
>重要：该密钥值（nacos.core.auth.plugin.nacos.token.secret.key）用于 JWT Token 签名，长度必须 ≥32 字节（Base64 解码后），且所有节点保持一致。不要使用官方文档中的示例值。

### 控制台用户管理

登录控制台后，可以在 权限控制 → 用户管理  中：
- 修改 nacos 用户密码
- 创建新用户（如 admin、ops）
- 为用户分配角色（ROLE_ADMIN 或自定义）

这些用户信息存储在 users 表中（Derby 或 MySQL），重启后保留。



## 拍错和常见坑：

- 中文路径 -> startup.cmd 解析异常
- 8848 被占 -> 改 server.port
- Derby 锁文件 -> 删 data/derby 重启
- 客户端报 403 Forbidden (未配置 username/password) -> 客户端没配用户名密码,补上凭据
- MySQL 驱动 -> Nacos 3.3 自带 8.x 驱动，连 MySQL 8 不用额外放 jar
- 控制台 401 / 登录失败 -> 重置密码或统一密钥
- 集群节点间无法同步 -> server.identity.key/value 不一致,确保所有节点相同
- 修改 Token 密钥后服务不可用 -> JWT 缓存失效, 重启所有节点

## 安全最佳实践

安全最佳实践（开发环境也建议遵循）

- 永远不要使用默认的 SecretKey 示例值
- Token 密钥定期更换（生产每季度一次）
- 控制台密码强度足够，不与系统密码复用
- 客户端凭据通过环境变量或密钥管理服务注入，不写死在代码仓库


## 客户端应用案例

客户端只关心“有没有 8848 + 鉴权”，不管服务端是 merged/server/console，也不管功能模式是 microservice 还是 all。

- Spring Boot 3.5.x
- jdk17+
- Spring Cloud 2025.0.x
- Spring Cloud Alibaba 2025.0.0.0
- Nacos Server 3.3.0（兼容 3.0.3 客户端）

父级POM 依赖

```xml
<properties>
    <java.version>21</java.version>
    <spring-boot.version>3.5.6</spring-boot.version>
    <spring-cloud.version>2025.0.0</spring-cloud.version>
    <spring-cloud-alibaba.version>2025.0.0.0</spring-cloud-alibaba.version>
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
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>${spring-cloud-alibaba.version}</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
</dependencies>
</dependencyManagement>
```

业务模块
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- 注册中心 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!-- 配置中心 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
</dependencies>
```
> 不需要单独引 nacos-client，starter 已带适配版。

所有客户端（SDK、OpenAPI）都必须提供用户名和密码，否则返回 403。

环境变量方式（推荐 CI/CD 不硬编码）
```bat
set NACOS_USERNAME=nacos
set NACOS_PASSWORD=your_password
```

bootstrap.yml
```
spring:
  application:
    name: demo-service
  profiles:
    active: dev
  cloud:
    nacos:
      username: ${NACOS_USERNAME:nacos}
      password: ${NACOS_PASSWORD:你设的管理员密码}
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: ${NACOS_NAMESPACE:public}
        group: DEFAULT_GROUP
      config:
        server-addr: 127.0.0.1:8848
        namespace: ${NACOS_NAMESPACE:public}
        group: DEFAULT_GROUP
        file-extension: yaml
        # 支持多配置拉取
        extension-configs:
          - data-id: common.yaml
            group: DEFAULT_GROUP
            refresh: true
```
> [Nacos 3.3 客户端默认要鉴权](https://nacos-group.github.io/docs/next/ecology/use-nacos-with-spring-boot3/)，必须设 username/password，否则拉配置 403 

{.tips}
> 注意：username 和 password 放在 spring.cloud.nacos 顶层，会被 discovery 和 config 共用。

application.yml

```yaml
server:
  port: 8081

spring:
  config:
    import:
      - nacos:demo-service-dev.yaml?group=DEFAULT_GROUP&refreshEnabled=true

my:
  feature:
    switch: true
```

OpenAPI 调用（curl / Postman）
```
# 第一步：获取 access_token
curl -X POST 'http://127.0.0.1:8848/nacos/v1/auth/login' \
  -d 'username=nacos&password=your_password'

# 返回 {"accessToken":"eyJ...","tokenTtl":18000,"globalAdmin":true}

# 第二步：携带 token 调用其他 API
curl 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=my-service&accessToken=eyJ...'
```