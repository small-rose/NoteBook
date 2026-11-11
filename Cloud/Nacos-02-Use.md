---
layout: default
title: Nacos 3.X use 
parent: Cloud
nav_order: 12
---


# Nacos 3.3.0  控制台使用指南：注册中心与配置中心实操

一、登录控制台

打开浏览器访问 http://localhost:8080

首次启动会进入初始化密码页面，请设置 nacos 用户的密码（至少8位）

后续登录：用户名 nacos，密码为你设置的密码

> 如果跳过初始化页面，默认密码也是 nacos，但强烈建议修改。

## 基本认识-三维隔离心智

Nacos 里所有资源都挂在一条坐标下，先记住这个公式：

```
Namespace（环境） → Group（业务域） → Data ID / Service（具体对象）
```

|层级	|通常用来隔离什么	|新用户怎么用|
|-------|---------------|----------|
|Namespace |环境 / 租户（dev / test / prod）	|一环境一个空间，用 ID 不用名称|
|Group	|业务域 / 模块（order / user / pay）	|同环境内区分系统，别拿它当环境用|
|Data ID	|一份具体配置	|{服务名}-{profile}.yaml|
|Service	|一个注册服务	|服务发现视角下的逻辑名|

> ⚠️ 最常见事故：只用默认 public 空间，dev 改了配置把 prod 带崩；或者 config.namespace 和 discovery.namespace 不一致，服务注册在一个空间、配置读另一个空间。


### 在控制台建命名空间（环境隔离底座）

打开 http://localhost:8080 → 登录（首次启动已初始化 nacos 密码）

左侧 > 命名空间 → 新建命名空间

按下面规范建三个：

|显示名	|命名空间 ID（自定义，固定不变）	|说明|
|----|---|---|
|开发环境	|ns-dev	|开发随便改，读写放开|
|测试环境	|ns-test	|联调用，禁删|
|生产环境	|ns-prod	|只读，变更走审批/CI|

> 💡 ID 一旦定下别改，客户端配置写死 ID。显示名可以随便改，但 ID 是身份。

建完之后，控制台顶部会出现命名空间切换器，配置列表、服务列表都按空间隔离。

## 配置中心：在控制台建配置（带隔离视角）

1.切到 ns-dev 命名空间

左上角命名空间下拉 → 选 开发环境（ns-dev）

2.新建配置

配置管理 → 配置列表 → ＋

|字段	|值	|说明|
|----|---|----|
|Data ID	|demo-service-dev.yaml	|服务名 + 环境 + 后缀|
|Group	|DEMO_GROUP	|业务域分组，不用 DEFAULT_GROUP|

格式	YAML 内容如下：	

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/demo_dev
    username: root
    password: devPass

my:
  feature:
    switch: true
    timeout: 3000
```
点 发布。

3.共享配置怎么放

比如所有服务都要 Redis：

- Data ID：shared-redis.yaml
- Group：SHARED_GROUP
- 内容：redis 连接信息

后面客户端用 extension-configs 额外引入。

4.历史版本与回滚（重点）

每次发布都会留版本。改错配置时：
- 点配置行 历史版本
- 找到上一版 → 回滚

>  千万别把 Nacos 当记事本直接改，重要变更前先看一眼当前值再动手。

## 注册中心：在控制台看服务（带隔离视角）

1.服务注册到哪个空间，就在哪看

服务管理 → 服务列表，左上角同样要切到 ns-dev。

- 没切对空间 = 看不到服务（不是没注册上）
- 服务名、Group、集群、实例都在空间内独立

2.服务详情里能做什么

进入某个服务（如 demo-service）：

- 实例列表：IP、端口、权重、健康状态、上次心跳
- 上下线：手动摘流量，不改代码
- 编辑权重：灰度先放 10% 流量
- 元数据：给实例打 label，比如 version=1.2.0、zone=hangzhou
- 订阅者：看谁在调我

3.临时实例 vs 持久实例

- 默认临时实例：客户端掉线 30s 自动剔除
- 持久实例：注册信息落存储，适合核心基础设施

开发阶段不用管，默认就行。

## 配置案例

- Spring Boot 3.5

1.基本依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2.bootstrap.yml（隔离核心配置）

```yaml
spring:
  application:
    name: demo-service
  profiles:
    active: dev
  cloud:
    nacos:
      username: nacos
      password: 你设的密码

      # ---- 配置中心 ----
      config:
        server-addr: 127.0.0.1:8848
        namespace: ns-dev        # ← 命名空间 ID，不是“开发环境”这几个字
        group: DEMO_GROUP
        file-extension: yaml
        # 共享配置
        extension-configs:
          - data-id: shared-redis.yaml
            group: SHARED_GROUP
            refresh: true

      # ---- 注册中心 ----
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: ns-dev         # ← 必须和 config.namespace 一致
        group: DEMO_GROUP
```

> 🔑 铁律：config.namespace 和 discovery.namespace 必须相同，否则“服务起来了但读不到配置”或“配置对了但服务互相看不见”。


3.Data ID 自动推导规则

Spring Cloud Alibaba 默认会拼两个 Data ID：

```text
1. demo-service.yaml          （不带 profile，共享基配置）
2. demo-service-dev.yaml      （带 profile，环境专属）
```

所以控制台里至少要建 demo-service-dev.yaml，否则只加载到空配置。

4.动态刷新

```java
@RestController
@RefreshScope
public class FeatureController {

    @Value("${my.feature.switch:false}")
    private Boolean switchOn;

    @GetMapping("/switch")
    public Boolean status() {
        return switchOn;
    }
}
```

控制台改 my.feature.switch → 发布 → 不用重启，接口返回值立刻变。

> 注意：
> 
> 纯 @Value 必须加 @RefreshScope；
> 
>用 @ConfigurationProperties 的 Bean 只要被容器管理也会自动刷新。

5.切环境只改一个地方

本地起 test 环境：

```bat
set SPRING_PROFILES_ACTIVE=test
set NACOS_NAMESPACE=ns-test
```
bootstrap.yml 里改成：

```yaml
profiles:
  active: ${SPRING_PROFILES_ACTIVE:dev}
# ...
namespace: ${NACOS_NAMESPACE:ns-dev}
```

这样一份代码，三个环境各走各的空间。

## 权限隔离（开发期就可以养成的习惯）

Nacos 3.x 默认开鉴权，RBAC 模型是 用户 → 角色 → 权限（资源+操作）。

推荐角色划分

|角色	|绑定空间	|权限|
|---|----|----|
|dev-rw	|ns-dev	|READ/WRITE|
|test-rw	|ns-test	|READ/WRITE（禁 DELETE）|
|prod-ro	|ns-prod	|READ only|
|prod-release	|ns-prod	|WRITE（只给 CI 流水线/运维）|

WRITE（只给 CI 流水线/运维）
操作路径：控制台 权限管理 → 用户管理 / 角色管理 / 权限管理

- 别用 nacos 管理员账号日常操作
- 每人一个账号，出事能审计到具体人


## 新用户最容易踩的坑（对照自查）

|现象	|根因	|解决|
|---|---|---|
|控制台看不到服务	|命名空间切错	|左上角切到对应 ns|
|配置不生效	|Data ID 拼错 / profile 不对	|检查 服务名-环境.yaml|
|改了配置不刷新	|没加 @RefreshScope 或没引 config 依赖	|补注解和依赖|
|服务能注册但调不通	|config/discovery namespace 不一致	|两个 namespace 写同一个 ID
|客户端 403	没|配 username/password	|bootstrap 里补凭据|
|误改生产配置	|全挤在 public 空间	|一环境一 namespace，prod 只读|
|namespace 用名字不用 ID	|名字改了就失联	|永远用 ID|

## 命名规范参考

```text
Namespace:
  ns-dev / ns-test / ns-prod

Group（按业务域）:
  USER_GROUP / ORDER_GROUP / PAY_GROUP / SHARED_GROUP

Data ID:
  {服务名}.yaml                # 基础配置
  {服务名}-{profile}.yaml      # 环境配置
  shared-{中间件}.yaml         # 跨服务共享

Service 名:
  直接用 spring.application.name，别带环境后缀
  （环境靠 namespace 区分，不靠服务名）
```

