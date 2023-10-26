---
layout: default
title: SpringBoot Config
parent: SpringBoot
nav_order: 160
---


## SpringBoot 配置文件相关

## 一、Profile 机制

Profile 其实就是环境配置切换的过程。

在实际的项目开发中，经常需要不同的环境配置，如开发时使用的开发环境数据库，一般可以数据或表可以直接操作修改，但是不能直接连生产环境的数据库，测试的时候测试人员使用测试数据库，而上线时就需要连生产环境的数据库。这就要求项目能够切换环境配置。

spring boot项目中，`application.properties `称为全局配置文件，实际上可以引入不同环境的配置文件，如：`application-dev.properties`，`application-prod.properties`，`application-test.properties`，

通过指定在全局配置文件`application.properties`中指定：`spring.profiles.active`的值切换配置文件，比如：

```properties
spring.profiles.active=xxx
```

就可加载`application-xxx.properties`配置。

当然，这里是列举的properties 文件，yml 后缀同理。

## 二、激活profile方式

（1）在全局配置文件中激活

```properties
spring.profiles.active=xxx
```

（2）在启动项目时使用命令行激活

```bash
java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev
```

（3）使用虚拟机参数激活

```properties
-Dspring.profiles.active=dev
```

## 三、配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件：

- file:./config/
- file:./
- classpath:/config/
- classpath:/

SpringBoot会从这四个位置全部加载主配置文件，**优先级由高到底，高优先级的配置（相同属性）会覆盖低优先级的配置，不同属性互补配置**；

总结：重复配置会覆盖，不同配置可补充。

## 四、外部配置加载顺序

Spring Boot 支持多种外部配置方式如下，**优先级从高到低**：

**1.命令行参数**

所有的配置都可以在命令行上进行指定

java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc

多个配置用空格分开； `--配置项=值`



2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的 `random.*` 属性值



**由jar包外向jar包内进行寻找；**

==**优先加载带profile**==

**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**



==**再来加载不带profile**==

**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**



10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源；

[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)



参考资料：尚硅谷学习视频，官方文档等