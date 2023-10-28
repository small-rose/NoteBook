---
layout: default
title: SpringBoot Logs
parent: SpringBoot
nav_order: 120
---


## SpringBoot 日志学习

### 一、 日志框架了解

Log4j、Log4j2
Logback
Slf4j
JCL (Jakarta Commons Logging)，也叫 Apache Common logging
JUL (java.util.logging)

其中，JCL（Commons Logging） 和 Slf4j 是属于日志门面。日志门面是一种软件设计模式，也叫正面模式或外观模式。门面对外提供统一的抽象层接口，真正的日志实现子系统只要实现抽象接口即可。调用者可以使用统一的调用API，不需要的关心子系统是如何实现，实现解耦。如果真正的实现子系统可以根据需要进行更换或升级，而不修改原有调用代码。

整理表格1：

| 日志门面（抽象层，本身无日志实现） | 日志实现                                         |
| ---------------------------------- | ------------------------------------------------ |
| Jakarta Commons Logging， Slf4j    | log4j，logback，log4j2，Jul（Java Util Logging） |

在实际使用时：门面 + 实现 。

整理表格2：

| 名称                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| Jakarta Commons Logging / Commons Logging | Apache项目，提供一套日志接口，属于日志门面。                 |
| Slf4j                                     | 类似Commons Logging 也是日志接口，属于日志门面。较优于前者。 |
| Logback                                   | 日志记录组件的实现。与Slf4j是同一作者，高兼容，必须配合Slf4j使用。 |
| Log4j                                     | 日志记录组件工具实现，日志实现框架之一。                     |
| Log4j2                                    | Log4j的升级版，且不兼容前者。                                |
| Jul（Java Util Logging）                  | java1.4 之后的官方日志实现。                                 |
| jboss-logging                             | 日志的抽象层。                                               |

### 二、常见 Commons Logging 集成

现在slf4j的优势出来之后，相对的就会逐渐代替旧的 `Commons Logging` 的集成方式，但是学习了解不可少，万一你遇到了项目里就是用了呢，你能否看得懂项目里的配置日志集成吗？

| 集成组合                       | 所需Jar包                                                    |
| ------------------------------ | ------------------------------------------------------------ |
| commons-logging 与 jdk-logging | commons-logging.jar                                          |
| commons-logging 与 Log4j       | commons-logging.jar <br/>log4j 基本包                        |
| commons-logging 与 Log4j2      | commons-logging.jar <br/>log4j-api<br/>log4j-core<br/>log4j-jcl (集成转换适配) |
| commons-logging 与 logback     | logback-core <br/>logback-classic <br/>slf4j-api <br/>       |

对于 `commons-logging` 与 `logback` 的组合，我表示存疑，logback 属于slf4j阵营，这个组合待考...

### 三、Slf4j 使用与集成

#### 1、基本使用

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

#### 2、Slf4j 集成关系图

Slf4j 官网

![Slf4g集成关系图](../Assets/images/springboot-images/concrete-bindings.png)

将写成表格：

| Slf4j 集成           | 集成所需jar                                                  |
| -------------------- | ------------------------------------------------------------ |
| Slf4j 与 logback     | slf4j-api.jar<br/>logback-core.jar <br/>logback-classic.jar  |
| Slf4j 与 log4j       | slf4j-api.jar<br/>log4j.jar<br/>slf4j-log4j12.jar （slf4j 和 log4j 的桥接） |
| Slf4j 与 jdk logging | slf4j-api.jar<br/>slf4j-jdkxxx.jar（slf4j 和 jdk logging 的桥接） |
| Slf4j 与 Simple      | slf4j-api.jar<br/>slf4j-simple.jar                           |
| Slf4j 与 Nop         | slf4j-api.jar<br/>slf4j-nop.jar （slf4j 和 NOP 的桥接）      |

使用 Slf4j 之后，每个日志实现框架，需要自己本身的配置文件。

#### 3、Slf4j 桥接遗留API

在实际使用中，引入不同的组件，组件自身会携带日志组件，如果需要统一使用某个日志门面如 Slf4j ，就需要把其他的日志组件重定向到我们想使用的日志门面Slf4j管理。然后 Slf4j 会根据绑定器，使用具体的日志实现框架。

官方桥接策略图：

![ Slf4j 桥接遗留适配图](../Assets/images/springboot-images/legacy.png)

Slf4j 自带了几个桥接模板，可以使Log4j，JCL和Java.util.logging的API重定向 Slf4j。

| 桥接适配包               | 作用                                               |
| ------------------------ | -------------------------------------------------- |
| log4j-over-slf4j-xxx.jar | 将 Log4j c重定向到 Slf4j                           |
| jcl-over-slf4j-xxx.jar   | 将 Commons Logging 里 Simple Logger 重定向到 Slf4j |
| jul-to-slf4j-xxx.jar     | 将 Java Uitl Logging 重定向到 Slf4j                |

**使用 Slf4j 桥接时注意避免冲突和死循环问题：**

（1）log4j-over-slf4j-xxx.jar 和 slf4j-log4j.jar 同时存在，slf4j-log4j12.jar 的存在会将所有日志调用委托给log4j。但由于同时由于log4j-over-slf4j.jar的存在，会将所有对log4j api的调用委托给相应等值的slf4j,所以log4j-over-slf4j.jar和slf4j-log4j12.jar同时存在会形成死循环。

（2）jul-to-slf4j-xxx.jar 和 slf4j-jdkxxx.jar 同时存在，slf4j-jdk14.jar的存在会将所有日志调用委托给jdk的log。但由于同时jul-to-slf4j.jar的存在，会将所有对jul api的调用委托给相应等值的slf4j，所以jul-to-slf4j.jar和slf4j-jdk14.jar同时存在会形成死循环。

**如何让系统中所有的日志都统一到 Slf4j ？**

- 1、将系统中其他日志框架先排除出去；
- 2、用中间包来替换原有的日志框架；
- 3、导入slf4j其他的实现

更多内容参考 [Slf4j 官网](https://www.slf4j.org)

### 四、SpringBoot 日志使用

#### 1、SpringBoot 日志

（1）基本依赖

SpringBoot的启动器

```xml
 <dependency>
	 <groupId>org.springframework.boot</groupId>
	 <artifactId>spring-boot-starter</artifactId>
 </dependency>
```

SpringBoot移动器的依赖中使用 日志功能；

```xml
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
	</dependency>
```

SpringBoot 底层依赖关系：

![Slf4g集成关系图](../Assets/images/springboot-images/springboot-log.png)

SpringBoot底层也是使用 `slf4j` + `logback` 的方式进行日志记录。

SpringBoot也把其他的日志都替换成了slf4j，使用了桥接中间包替换了原来框架的日志。

如果要引入其他框架，一定要把这个框架的默认日志依赖移除掉，比如：

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<exclusions>
		<exclusion>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

SpringBoot能自动适配所有的日志，而且底层使用 slf4j + logback 的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可。

（2）logback

logback 主要有三个模块：

 - logback-core：基础日志功能。
 - logback-classic：log4j的升级，对日志门面 Slf4j API 进行具体实现。
 - logback-access：用于Servlet 容器进行基础、提供网络访问日志功能。

logback 主要包含三个组成部分：

 - Logger - 日志记录器
 - Appenders - 输出控制器
 - Layouts - 日志输出格式管理



#### 2、SpringBoot 日志配置

##### （1）默认日志

SpringBoot默认帮配置好了日志，默认的日志级别是 `info `

日志的级别由低到高：  **trace < debug < info < warn < error**

```java
//记录器
	Logger logger = LoggerFactory.getLogger(getClass());
	@Test
	public void contextLoads() {
		//System.out.println();

		//可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
		logger.trace("这是trace日志...");
		logger.debug("这是debug日志...");
		//SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别；root级别
		logger.info("这是info日志...");
		logger.warn("这是warn日志...");
		logger.error("这是error日志...");
	}
```

日志输出格式：

`%d` 表示日期时间，
`%thread` - 表示线程名，
`%-5level` - 级别从左显示5个字符宽度
`%logger{50}` - 表示logger名字最长50个字符，否则按照句点分割。 
`%msg` -  日志消息，
`%n` - 是换行符

如：
```txt
%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

SpringBoot修改日志的默认配置

```properties
logging.level.com.smallrose=trace


#logging.path=
# 不指定路径在当前项目下生成springboot.log日志
# 可以指定完整的路径；
#logging.file=G:/springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

#  在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

**logging.file 和 logging.path :**

| logging.file | logging.path | Example  | Description                        |
| ------------ | ------------ | -------- | ---------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                     |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件               |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

`logging.file` 和  `logging.path` 二者选其一即可。如果都配置了，`logging.file` 会生效。



##### （2）指定配置

类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了；

**logback-spring.xml**：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能，也是官方推荐的方式。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>

<springProfile name="dev, staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```
使用示例如：

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
```
如果使用logback.xml作为日志配置文件，还要使用profile功能，会报错 ：`no applicable action for [springProfile]`

#### 3、日志框架切换

可以按照 Slf4j 桥接遗留适配图 ，进行相关的切换；

(1) `slf4j+log4j `的方式 :

使用 `slf4j+log4j ` 的方式需要排除 `logback` 和 自身携带的 log4j到slf4j 的桥接包 `log4j-over-slf4j`，同时要引入

观察 slf4j 集成整理的表格里 Slf4j 与 log4j 集成，需要 slf4j-api.jar 、log4j.jar 以及slf4j 和 log4j 的桥接包 slf4j-log4j12.jar ，springboot 默认已有前者两者，需要引入桥接包 `slf4j-log4j12`

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

切换为log4j2 ：

springboot 支持的日志框架:`spring-boot-starter-log4j2 ` 和 `spring-boot-starter-logging` ，后者为默认，若要使用前者则，直接排除后者即可。 [官网说明](https://docs.spring.io/spring-boot/docs/2.2.8.RELEASE/reference/html/using-spring-boot.html#using-boot) 。

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```



参考文章：

https://segmentfault.com/a/1190000017909301 

硅谷springboot 日志相关视频教程

[springboot 官网](https://docs.spring.io/spring-boot/docs/2.2.8.RELEASE/reference/html/using-spring-boot.html#using-boot) 