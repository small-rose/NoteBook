---
layout: default
title: SpringBoot IOC
parent: SpringBoot
nav_order: 150
---


# SpringBoot注解IoC



## 一、Ioc简介

`IoC`是一种通过描述来生成或者获取对象的技术。在Spring 中把每个需要管理的对象称为Spring Bean，简称Bean，Spring 管理这些Bean的容器，被称为Spring IoC容器（简称IoC容器）。

2个基本作用：

（1）通过描述管理Bean，包括发布和获取Bean；

（2）通过描述完成Bean之间的依赖关系。



`Spring IoC `容器其实就是管理Bean的容器。Spring定义中，要求所有的 `IoC` 容器都需要实现 `BeanFactory` 接口，它是一个顶级容器接口。

`BeanFactory` 接口主要代码：

```java
package org . springframework.beans.factory;

public interface BeanFactory {
    // 前缀 用来区分是获取FactoryBean还是FactoryBean的createBean创建的实例.如果&开始则获取FactoryBean;否则获取createBean创建的实例.
    String FACTORY BEAN PREFIX = " &" ;
    //多个getBean方法
    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException; 

    <T> T getBean(Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

    <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);

    <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

    //是否包含某个Bean
	boolean containsBean(String name);
	//Bean 是否为单例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	//Bean 是否为原型
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	//是否类型匹配
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
 
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
 	
    //获取Bean的类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	 //获取Bean的别名
	String[] getAliases(String name);
}
```

源码中有多个 `getBean()` 的方法，有按类型获取Bean，有按名称获取Bean，Spring IoC容器允许按类型或者名称获取Bean。

`isSingleton()` 方法判断Bean 在Spring IoC中是否为单例。**在Spring IoC中，默认情况下，Bean 都是以单例存在的，即使用 `getBean()` 方法返回的都是同一个对象。**

`isPrototype()` 方法与`isSingleton()` 方法刚好相反，如果它返回`true`， 那么使用 `getBean()` 方法获取Bean的时候，Spring IoC容器就会创建一个新的Bean返回。

Spring IoC 容器布局：

![spring IoC 接口](/images/spring/springioc.svg)

`ApplicationContext` 接口通过继承上级接口进而继承 `BeanFactory` 接口，还在`BeanFactory` 的基础上扩展了消息国际化接口（MessageSource）、环境可配置接口（EnvironmentCapable）、应用事件发布接口（ApplicationEventPublisher）和资源模式解析接口（ResourcePatternResolver），功能更强大。



XML相关的`IoC` 容器装配的方式主要是使用`<bean>` 标签，通过解析`bean`标签的描述来管理`bean`及相关依赖。

在SpringBoot 中主要是通过注解来装配`Bean` 到`Spring IoC`容器中，基于注解的`IoC` 容器为`AnnotationConfigApplicationContext`。

示例：

一个普通的POJO

```java
package com.smallrose.web.app.model;

public class User {

	private String id;
	private String userName;
	private String note;
 	// getter setter  tostring
}
```

SpringBoot 的配置：

```java
package com.smallrose.web.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.smallrose.web.app.model.User;

@Configuration
public class AppConfig {

	@Bean(name="user")
	private User initUser() {
		User u = new User();
		u.setId("123");
		u.setUserName("zhangxiaocai.cn");
		u.setNote("小菜");
		return u;
	}
}
```

`@Configuration` ：表示一个Java配置文件，类似Spring的xml配置文件。Spring的容器会根据它来生成IoC容器去装配Bean；

`@Bean`：表示将 `initeUser()` 方法返回的POJO装配到IoC容器中，它的属性name 定义这个Bean的名称，如果没有配置name属性，则将方法名称`“initUser”`作为Bean的名称保存到Srping IoC 容器中。

然后就可以使用`AnnotationConfigApplicationContext`来构建自己的IoC容器：

```java
package com.smallrose.web.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.smallrose.web.app.model.User;

public class IoCTest {
	
	private static Logger log = LoggerFactory.getLogger(IoCTest.class);

	public static void main(String[] args) {
		ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		User user = ctx.getBean(User.class);
		System.out.println(user);
	}
}
```

输出结果：

```txt
15:06:34.640 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'appConfig'
15:06:34.656 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'user'
15:06:34.792 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'spring.liveBeansView.mbeanDomain' in PropertySource 'systemProperties' with value of type String
User [id=123, userName=zhangxiaocai.cn, note=小菜]

```



## 二、Bean 的装配

Spring 允许通过XML或Java配置文件装配Bean，但 SpringBoot 是基于注解的方式。

### 1、扫描装配Bean

前面的例子是使用注解`@Bean` 注入到SpringIoC 容器，实际项目中，肯定会有很多个Bean，每个都这么写，岂不是要累死人。

Spring 还允许进行扫描装配Bean 到 IoC 容器，对于扫描装配而言使用的注解是`@Component`和`@ComponentScan`。

`@Component` ：用来标明哪个类被扫描进入`Spring IoC` 容器。

`@ComponentScan` ：用来标明采用何种策略去扫描装配Bean。

实例：

在刚才的 `config`目录下建一个POJO 如，可以将刚才的`User`移过来，为了区分我新建一个类：`Employee` ：

```java
package com.smallrose.web.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component("emp")
public class Employee {

	@Value("123")
	private Long id;
	@Value("zhangxiaocai.cn")
	private String empName;
	@Value("note_xiaocai")
	private String empNote;
	
	//getter setter tostring
}
```

使用`@Component` 注解表示该类将会被`Spring IoC`容器扫描装备，其中`“emp”`表示作为Bean的名称，也可以不配置这个名称字符串，IoC 容器会把类名第一个字母作为小写其他不变作为Bean的名称放入到IoC容器中；

`@Value` 注解在学习YML语法的时候遇到过，是用来绑定属性注入参数，也可以直接赋值。

修改`AppConfig`类：

```java
package com.smallrose.web.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class AppConfig {
 
}
```

这里添加了`@ComponentScan` 注解，它会进行扫描，默认情况下它只会扫描所在注解类`AppConfig`所在的当前包和其子包。为什么要把`Employee`建在`config` 目录下就是这个原因。

运行测试类：

```java
package com.smallrose.web.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.smallrose.web.app.model.User;

public class IoCTest {
	
	private static Logger log = LoggerFactory.getLogger(IoCTest.class);

	public static void main(String[] args) {
		ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		Employee emp = ctx.getBean(Employee.class);
		System.out.println(emp);
	}
}
```

运行结果：

```txt
16:07:46.227 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'appConfig'
16:07:46.238 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'emp'
Employee [id=123, empName=zhangxiaocai.cn, empNote=note_xiaocai]
```

找到注解类`ComponentScan`源码：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)//在一个类中可重复定义
public @interface ComponentScan {
 	//** 定义扫描的包
	@AliasFor("basePackages")
	String[] value() default {};
 	//** 定义扫描的包
	@AliasFor("value")
	String[] basePackages() default {};
 	//** 定义扫描的类
	Class<?>[] basePackageClasses() default {};
 	// Bean name 生成器
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

 	// 作用域解析器
	Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;
 	// 作用域代理模式
	ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

	 // 资源匹配模式
	String resourcePattern() default ClassPathScanningCandidateComponentProvider.DEFAULT_RESOURCE_PATTERN;
 	//是否启用默认的过滤器
	boolean useDefaultFilters() default true;
 	//** 当满足过滤条件时扫描
	Filter[] includeFilters() default {};
	//** 当不满足过滤条件时扫描
	Filter[] excludeFilters() default {};
    // ** 是否启用延迟加载
	boolean lazyInit() default false;
 
    //定义过滤器
	@Retention(RetentionPolicy.RUNTIME)
	@Target({})
	@interface Filter {
 		// 过滤器类型，可以按注解类型或者正则表达式等过滤
		FilterType type() default FilterType.ANNOTATION;
 		// 定义过滤的类
		@AliasFor("classes")
		Class<?>[] value() default {};
 		// 定义过滤的类
		@AliasFor("value")
		Class<?>[] classes() default {};
   		// 匹配方式
		String[] pattern() default {};

	}
}
```

带`“**”`的是最常用的配置项。

`basePackages`： 定义扫描的包名，没有定义时只会扫描当前包和子包下的路径；

`basePackagesClasses`：定义扫描的类；

`includeFilters`：定义满足过滤器（Filter）条件的 `Bean`才去扫描

`excludeFilters`：排除过滤器（Filter）条件的 `Bean`才去扫描，二者都需要通过一个`@Filter`去定义，它有一个type类型，这里可以定义为注解或者正则等类型。`classes`定义注解类，`pattern`定义正则式类。

按照前面的实例，`User`类不在`config`目录，可以将`AppConfig`中的注解修改为：

```java
@ComponentScan("com.smallrose.web.app.*")
```

或者

```java
@ComponentScan(basePackages={"com.smallrose.web.app.model"})
```

或者

```java
@ComponentScan(basePackageClass={User.class})
```

三种方式都可以使`IoC` 容器去扫描`User` 类，而包名可以采用正则式去匹配。

但有些时候需求是扫描一些包，将一些Bean 装配到SpringIoC 容器中，而不想加载这个包里的某些Bean。比如在包`com.smallrose.web.app` 下可能有`service`包，在`service`包下有个`UserService`类，一般使用`@Service` 注解标注（该标注注入了`@Component`，在默认情况下会被Spring 扫描装配到 `IoC`容器），类如下：

```java
package com.smallrose.web.app.service;

import org.springframework.stereotype.Service;

import com.smallrose.web.app.model.User;

@Service
public class UserService {

	public void printUser(User user){
		System.out.println(user);
	}
}
```

如果正常启动，那么这个类是会被扫描到`Spring IoC` 容器中。假设不想装配这个类呢？

则需要把`AppConfig` 扫描策略修改为：

```java
@ComponentScan(basePackages = "com.smallrose.web.app.*",
	excludeFilters = {@Filter(classes = {Service. class})})
```

记得把`User`类添加`@Component`注解：

```java
package com.smallrose.web.app.model;

import org.springframework.stereotype.Component;

@Component
public class User {

	private String id;
	private String userName;
	private String note;
 	// getter setter toString
	
}
```

运行测试类：

```java
package com.smallrose.web.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.smallrose.web.app.model.User;
import com.smallrose.web.app.service.UserService;

public class IoCTest {
	
	private static Logger log = LoggerFactory.getLogger(IoCTest.class);

	public static void main(String[] args) {
		ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		User user = ctx.getBean(User.class);
		System.out.println(user);
		
		UserService userservice = ctx.getBean(UserService.class);
		System.out.println(" userservice "+ userservice);
	}
}
```

运行结果：

```txt
17:18:43.019 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'appConfig'
17:18:43.030 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'user'
User [id=null, userName=null, note=null]
Exception in thread "main" org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.smallrose.web.app.service.UserService' available
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:352)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:343)
	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1127)
	at com.smallrose.web.config.IoCTest.main(IoCTest.java:20)
```

可以看到`User`类已经装配了，只是属性默认`null`值，而`UserService`则提示没有定义这个`Bean`，说明`UserService` 类根本没有进行装配。在加入了`excludeFilters`的配置，使标注了`@Service`的类将不被`IoC`容器扫描注入，这样就可以把`UserService` 类排除出 `Spring IoC`中了。

在`@SpringBootApplication`注解中也注入了`@ComponentScan`，源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
//自定义排除扫描类
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
 	// 通过类型排除自动类型类
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};
 	// 通过名称排除自动类型类
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};
 	// 定义扫描包
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

 	// 定义被扫描的类
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

    // Bean 的名称生成器
	@AliasFor(annotation = ComponentScan.class, attribute = "nameGenerator")
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;
 	
    //是否生成代理bean方法
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;
}
```

`exclude` 和 `excludeName` 两个方法是对于其内部的自动配置类才会生效。如果要自己排除其他类，可以加入`@ComponentScan`达到目录，如上面的扫描`User`而不扫描`UserService`可以修改启动配置文件：

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.smallrose.web.app.*",
	excludeFilters = {@Filter(classes = {Service. class})})
public class SpringiocApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringiocApplication.class, args);
	}

}
```

### 2、自定义第三方Bean

有时候需要引入第三方的`Bean`，比如`DataSource`，使用`@Bean`注解即可。

示例：

引入DBCP数据源

```xml
<dependency>
    <groupid>org.apache.commons</groupid>
    <artifactid>commons-dbcp2</artifactid>
</dependency>
<dependency>
    <groupid>mysql</groupid>
    <artifactid>mysql-connector-java</artifactid>
</dependency>
```

然后在`AppConfig.java` 中添加到IoC容器即可：

```java
@Bean(name="dataSource")
public DateSource getDataSource(){
	Properties props = new Properties();
    props.setProperty("driver","com.mysql.jdbc,Driver");
    props.setProperty("url","jdbc:mysql://localhost:3306/test");
    props.setProperty("username","root");
    props.setProperty("password","123456");
    DateSource dataSource = null;
    try{
        dataSource = BasicDataSourceFactory.creatDataSource(props);
    }catch(Exception e){
    }
    return dataSource;
}
```

## 三、依赖注入

### 1、@Autowired 注解

Spring 中常用注解之一。注入机制最基本的一条是根据属性的类型（by type）找到对应的`Bean` 进行注入。在Ioc的顶级接口`BeanFactory` 中就有`getBean()` 方法获取对应的`Bean` ,`getBean()` 支持根据类型或根据名称获取方式。

比较常见的就不举例了。在常见的操作里，一个接口一个实现类，实现类使用`@Service` 标注，然后使用接口注入的方式进行使用。

这里举个不一样的例子，模拟调用支付的，请求只调用支付接口，接口分布被用微信支付和支付宝支付实现。

接口：

```java
package com.smallrose.web.app.service;

public interface PayServiceI {

	public void pay();
}
```

微信支付实现类：

```java
package com.smallrose.web.app.service.impl;

import org.springframework.stereotype.Service;
import com.smallrose.web.app.service.PayServiceI;

@Service
public class WxPayImpl implements PayServiceI{

	@Override
	public void pay() {
		System.out.println("使用微信支付....");
	}
}
```

支付宝支付实现类：

```java
package com.smallrose.web.app.service.impl;

import org.springframework.stereotype.Service;
import com.smallrose.web.app.service.PayServiceI;
@Service
public class AliPayImpl implements PayServiceI{

	@Override
	public void pay() {
		System.out.println("使用支付宝支付....");
	}

}
```

在controller 中注入调用：

```java
package com.smallrose.web.app.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import com.smallrose.web.app.service.PayServiceI;

@Controller
public class PayController {

	@Autowired
	private PayServiceI payservice ;
	
	@GetMapping("/pay")
	public void toPay(HttpServletRequest request){
		payservice.pay();	
	}
}
```

在示例中可以看到，输入值是一个支付操作，但是却有两种支付方式，`SpringIoC` 注入怎么办呢？启动程序发现会有以下错误信息：

```txt
Description:

Field payservice in com.smallrose.web.app.controller.PayController required a single bean, but 2 were found:
	- aliPayImpl: defined in file [D:\dev-toos\sts-bundle\stsWork\SpringBootLearn\target\classes\com\smallrose\web\app\service\impl\AliPayImpl.class]
	- wxPayImpl: defined in file [D:\dev-toos\sts-bundle\stsWork\SpringBootLearn\target\classes\com\smallrose\web\app\service\impl\WxPayImpl.class]
```

程序注入需要一个Bean 但是现在发现了两个匹配的Bean，所以就不知道到底使用哪一个了，怎么办呢？

将属性名称改成对应的实现类名称

```java
package com.smallrose.web.app.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import com.smallrose.web.app.service.PayServiceI;

@Controller
public class PayController {

	@Autowired
	private PayServiceI wxPayImpl ;
	
	@GetMapping("/pay")
	public void toPay(HttpServletRequest request){
		wxPayImpl.pay();	
	}
}
```

将属性名称从 payservice 改成 wxPayImp 为什么可以？

**因为`@Autowired ` 首先根据类型找对应的Bean，如果对应的类型的Bean 不是唯一的，那么会根据其属性名称和Bean 的名称进行匹配。如果匹配得上就会使用该Bean，如果还无法匹配，就会抛出异常。**

注意，`@Autowired ` 是一个默认必须找到对应Bean的注解，如果不能确定其标注属性一定会存在并且允许这个被标注的属性为`null`，那么可以配置`@Autowired` 的属性`required` 为`false`。

```java
@Autowired(required=false)
```

另外，`@Autowired `除了可以标注属性外，还可以标注方法，如setPayservice方法：

```java
@Autowired
public void setPayservice(Payservice payservice){
	this.payservice = payservice;
}
```

### 2、消除歧义

在上面，支付方式有两种的时候，为了使`@Autowired ` 能继续使用，将属性名称 payservice 改成 wxPayImp 。

产生注入失败的问题根本是按类型查找时，发现了多个匹配Bean，造成IoC 容器注入的困扰，这个问题成为歧义性问题。

如果不修改属性名称，消除歧义的办法有哪些呢？

（1）使用 `@Primary` 注解，它是一个修改优先权的注解。当有微信支付和支付宝支付的时候，假设这个是使用微信支付，那么只需要在微信支付的类上加入 `@Primary` 注解即可。

```java
package com.smallrose.web.app.service.impl;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Service;

import com.smallrose.web.app.service.PayServiceI;

@Service
@Primary
public class WxPayImpl implements PayServiceI{

	@Override
	public void pay() {
		System.out.println("使用微信支付....");
	}
}
```

 `@Primary` 注解会告诉spring IoC 容器，放发现多个相同类型的Bean 时，请优先使用我进行注入。

这样通过优先级变换使得IoC 容器知道那个具体的实例满足依赖注入。

但是， `@Primary` 注解可以使用在多个类上，微信支付和支付宝支付都添加了该注解，那么IoC容器还是无法区分采用哪个Bean的实例进行注入。那么可以使用`@Quelifier`注解来灵活实现注入。它将和`@Autowired `注解 组合在一起，通过类型和名称一起找到Bean。**因为Bean名称在Spring IoC容器中是唯一标识。**这样就可以消除歧义性。

```java
package com.smallrose.web.app.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import com.smallrose.web.app.service.PayServiceI;

@Controller
public class PayController {

	@Autowired
	@Qualifier("wxPayImpl")
	private PayServiceI payService ;
	
	@GetMapping("/pay")
	public void toPay(HttpServletRequest request){
		payService.pay();
	}
}
```

### 3、带参数的构造方法类装配

上面都是不带参数的构造方法下实现的依赖注入。但有时候有些累只有带参数的构造方法，可以使用`@Autowired`

 注解对构造方法的参数进行注入。

```java
@Component
public class PersonBean {
    
	private PayServiceI payService ; 
    
    public PersonBean(@Autowired @Qualifier("wxPayImpl") PayServiceI payService){
    	this.payService=payService;
    }
}
```

## 四、生命周期

 IoC 容器装配和销毁Bean的过程，即Bean 的生命周期过程，大致分为Bean定义、Bean的初始化、Bean的生存期、Bean的销毁4个部分。

**Bean定义过程：**

（1）`Spring` 通过我们的配置，如`@ComponentScan`定义的扫描路径去找到带有`@Component`的类，这个过程就是资源定位的过程。

（2）找到资源之后，就开始解析，并且将定义的信息保存起来（保存到BeanDefinition的实例中）。（注意，此时没有初始化Bean，也没有Bean实例）

（3）接着会把`Bean 定义`发布到`Spring IoC`容器中。（注意，此时也只有Bean定义，没有初始化Bean，没有Bean实例）

完成3步只是资源定位并将Bean的定义发布到 `IoC` 容器的过程，还没有`Bean` 实例的生成，更没有完成依赖注入。

在默认情况下，Spring 会继续去完成Bean 的实例化和依赖注入，这样从 `IoC`容器中就可以得到一个依赖注入完成的Bean。

Spring Bean 的初始化流程：

```txt
资源定位       ——>       Bean定义          ——>     发布Bean定义    ——>    实例化   ——>  依赖注入（DI）
@ComponentScan    保存到BeanDefinition的实例     IoC容器装载Bean定义    创建Bean实例    @Autowired注入
```

`ComponentScan` 中还有一个配置项 lazyInit，只可以配置 Boolean 值，且默认值为 false ，也就是 默认不进行延迟初始化，因此在默认的情况下 Spring 会对 Bean 进行实例化和依赖注入对应的属性值。    

使用的话在配置类 AppConfig 的`＠ComponentScan` 中加入 lazylnit 配置，如下面的代码： 

```java
@ComponentScan(basePackages = "com.smallrose.web.app.*", lazyinit = true)    
```

这样`IoC`容器不会在发布Bean定义后马上完成实例化和依赖注入，而是在使用Bean的时候进行实例化和依赖注入。



![Spring Bean 的生命周期]()



（1）这些接口和方法，在没有注释说明的情况下的流程节点都是针对单个Bean，但是`BeanPostProcessor`是针对所有Bean。

（2）即使定义了`ApplicationContextAware` 接口，有时并不会调用，这要根据`IoC`容器来决定。`Spring IoC`容器的最低要求是实现`BeanFactory`接口，而不是实现`ApplicationContext` 接口。对于没有实现`ApplicationContextAware` 接口的容器，在生命周期对于的`ApplicationContextAware` 定义的方法也是不会被调用的，只有实现了`ApplicationContext` 接口的容器，才好在生命周期调用`ApplicationContextAware` 所定义的`setApplicationContext`方法。

## 五、条件装配Bean

有时因为某些客观因素会使一些Bean无法进行初始化，如数据库连接池的配置中漏掉了一下配置会造成数据源不能连接上，这时`IoC`容器如果还进行数据源装配，系统将会抛出异常，导致应用无法继续，这时希望`IoC`容器不去装配数据源。

为了处理这种场景，Spring 提供了 `@Conditional`注解来实现，同时它需要配合另外一个`Condition`接口来完成对应的功能。(`org.springframework .context.annotation.Condition`)

如之前配置数据源：

```java
package com.smallrose.web.config;

import java.util.Properties;

import javax.sql.DataSource;

import org.apache.commons.dbcp2.BasicDataSourceFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

import com.smallrose.web.condition.DatabaseConditional;

@Configuration
public class AppConfig {

	@Bean(name="dataSource",destroyMethod="close")
	@Conditional(DatabaseConditional.class)
	public DataSource getDataSource(
			@Value("${database.driverName}") String driver,
			@Value("${database. url}")  String url ,
			@Value("${database.username}") String username ,
			@Value("${database. password }") String password
			){
		Properties props = new Properties();
	    props.setProperty("driver",driver);
	    props.setProperty("url",url);
	    props.setProperty("username",username);
	    props.setProperty("password",password);
	    DataSource dataSource = null;
	    try{
	        dataSource = BasicDataSourceFactory.createDataSource(props);
	    }catch(Exception e){
	    }
	    return dataSource;
	}
}
```

添加`@Conditional`注解，并且配置类`DatabaseConditional`，这个类必须现实`Condition`接口。对于`Condition`接口则要求实现`matches`方法：

```java
package com.smallrose.web.condition;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class DatabaseConditional implements Condition{
	/**
	 * 数据库装配条件
	 * @param context 条件上下文
	 * @param metadata 注释类型的元数据
	 * @return true 装配 Bean ，否则不装配
	 */
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		//取出环境配置
		Environment evn = context.getEnvironment();
		// 判断属性文件是否存在对应的数据库配置
		return evn.containsProperty("database.driverName") 
				&& evn.containsProperty("database.url") 
				&& evn.containsProperty("database.userName")
				&& evn.containsProperty("database.password");
	}
}
```

matches 方法首先读取其上下文环境 ， 然后判定是否已经配置了对应的数据库信息。这样，当这 些都己经配置好后则返回 true。这个时候 Spring 会装配数据库连接池的 Bean ，否则是不装配的。    

## 六、Bean的作用域

在`BeanFactory`接口中，有`isSingleton` 和 `isPrototype` 两个方法。其中， `isSingleton` 方法如果返回 `true` ，则 Bean 在 loC 容器中以单例存在，这也是 `Spring IoC`容器的默认值 ；如果 `isPrototype` 方法返回 `true`，则 当我们每次获取 Bean 的时候， `IoC` 容器都会创建一个新的 Bean，这显然存在很大的不同，这便是 Spring Bean 的作用域的问题。 

在一般容器中，Bean 都会存在单例（Singleton）和原型（Prototype）两种作用域。而Web 容器，则存在页面（page）、请求（ request ）、会话 （ session ）和应用（ Application ）4种作用域。对于页面（Page）是针对JSP当前页面而言，spring 无法支持。

Bean的作用域

| 作用域类型      | 使用范围       | 描述                                                         |
| --------------- | -------------- | ------------------------------------------------------------ |
| **singleton**   | 所有spring应用 | 默认值，IoC容器值存在单例                                    |
| **prototype**   | 所有spring应用 | 每当从IoC容器取出一个Bean，就会创建一个新的 Bean             |
| **session**     | spring web应用 | HTTP会话                                                     |
| **application** | spring web应用 | Web 工程生命周期                                             |
| request         | spring web应用 | Web 工程单词请求（request）                                  |
| globalSession   | spring web应用 | 在一个全局的HTTP Session，一个Bean定义对应一个示例。<br/>实践中基本不使用 |

常用的是加粗的4种，对于application 作用域，完全可以使用单例来代替。

**单例 （ Singleton）和原型（ prototype ）的区别**

不进行任何配置默认就是单例。单例就是每次获取的Bean实例都他相同的一个Bean。

如果要进行原型配置，则在需要配置的Bean上添加作用域注解`@Scope`

````
@Scope(ConfiguarebleBeanFactory.SCOPE_PROTOTYPE)
````

示例：

```java
package com.smallrose.web.config;

import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class BeanTest {

}
```

测试：

```java
package com.smallrose.web.test;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.smallrose.web.config.AppConfig;
import com.smallrose.web.config.BeanTest;

public class IoCTest {

	private static Logger log = LoggerFactory.getLogger(IoCTest.class);

	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		BeanTest bean1 = ctx.getBean(BeanTest.class);
		BeanTest bean2 = ctx.getBean(BeanTest.class);
		System.out.println(bean1==bean2);
	}
}
```

测试结果：false

注释掉`@Scope` 结果为 true 。

ConfigurableBeanFactory 只能提供 单 例 （ SCOPE_SINGLETON ）和 原 型 （ SCOPE_ PROTOTYPE ） 两种作用域供选择 ， 如果是在 SpringMVC 环境中，还可以使用 WebApp l icationContext 去定 义其他作用域 ， 如请求（ SCOPE REQUEST ）、 会话 （ SCOPE_SESSION ） 和应用 （ SCOPE APPLICATION ） 。     

如：

```java
package com.smallrose.web.config;

import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope(WebApplicationContxt.SCOPE_REQUEST)
public class BeanTest {

}
```

这样同一个请求范围内去获取这个Bean的时候，只会共用同一个Bean，第二次请求就会产生新的Bean。



## 七、使用 @Profile

实际开发中，常常是多环境切换。每套环境可能一些上下文配置不一样、数据库连接不一样等。Spring 提供了 Profile 机制，可以在不同环境之间切换。

复习一下配置文件的方式的话之间使用以下格式：

`application-{profile}.properties` 或`application-{profile}.yml`

然后在 `application.properties` 或`application.yml` 中激活Profile使用机制：

如：

````
spring.profiles.active=dev
````

这里主要学习的是注解方式。



在 Spring 中存在两个 参数可以提供给配置，以修改启动 Profile 机制， 一个是`spring.profiles.active` ， 另一个是 `spring profiles.default` 。在这两个属性都没有配置的情况下 ， Spring 将不会启 动 Profile 机制，这就意味着被`＠Profile` 标注的 Bean 将不会被 Spring 装配到 `IoC` 容器中 。 Spring是先判定是否存在`spring.profiles.active`配置后再去查找 `spring profiles.default` 配置的，所以`spring.profiles.active`的优先级要大于 `spring profiles.default` 。

在Java 启动项目时，如果不在配置文件激活也可以使用以下配置也可以启动Profile 机制：

```
JAVA_OPTS="-Dspring.profiles.active=dev"
```

如果在全局配置文件和profile配置文件有相同配置，根据Spring Boot规则，如果配置了profile激活，那么`application- {profile} . properties` 文件去代替原来默认的 `app lication.properties` 文件，相同属性会覆盖，不同属性则合并。

比如开发环境，对日志打印使用较多，但生产环境较少。

## 八、XML引入Bean

SpringBoot 建议使用注解和扫描配置Bean，但同样支持XML配置Bean。

在SpringBoot 中使用XML 对Bean 进行配置，需要使用`@ImportResource`注解引入对应的 XML 文件，用以价值Bean。 有些框架（如Dubbo等）是基于Spring XML 方式开发，这时就需要引入XML 方式来实现配置。

比如有个想加载的Bean，即使是个普通的POJO也行，然后建个`spring-other.xml`文件，接着配置Bean。

在java 配置文件中直接载入：

```java
package com.smallrose.web.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;

@Configuration
@ComponentScan("com.smallrose.web.app.*")
@ImportResource(value= {"classpath:spring-other.xml"})
public class AppConfig {

 
}
```

## 九、Spring EL

为了装配Bean 更加灵活，Spring 提供了表达式语言 Spring EL 。主要作用如下：

### （1）读取属性文件

如：

```java
@Value("${database.driverName}")
String driver ;
```

`@Value` 中的`${ }` 代表占位符，它会去读上下文的属性值装配到属性中。

更多的属性与JavaBean 绑定参考 [YAML 语法](bb845937.html)

### （2）调用方法

如记录一个Bean 初始化时间：

```java
@Value("#{T(System).currentTimeMillis()}")
private Long initTime = null ;
```

这里采用 `#{ }`代表启用 Spring 表达式，它将具有运算的功能 ； `T(...)`代表的是引入类；

System是`java.lang.*`包的类，这是Java 默认加载的包，因此可以不写全限定名，如果是其他的包，则需要写出全限定名才能引用类；`currentTimeMillis` 是它的静态（ static ）方法，也就是我们调用一次`System.currentTimeMillis() `方法来为这个属性赋值。

###  （3）给属性直接赋值。

如：

```java
@Value("#{'使用spring EL 赋值字符串'}") //字符串赋值
private String str = null;

@Value("#{9.3E3}") //科学计数法赋值
private double d ;

@Value("#{3.14}") // 浮点数赋值
private float pi ;
```

还可以获取其他Spring Bean 的属性来给当前 Bean 属性赋值，如：

```java
@Value("#{beanName.str}")
private String otherBeanProp = null;
```

注意 ，这里的 beanName 是 Spring IoC 容器 Bean 的名称 。 str 是其属性，代表引用对应的 Bean 的属性给当前属性赋值。如果想把这个属性的字母全部变大写还可以这样：

```java
@Value("#{beanName.str?.toUpperCase()}")
private String otherBeanProp = null ;
```

注意这里的 Spring EL。这里引用由属性后跟着是一个`？`，这个符号 的含义是判断这个属性是否为空。如果不为空才会去执行 toUppercase 方法，进而把引用到的属性转换为大写，赋予当前属性。

### （4）其他运算

可以使用 Spring EL 进行一定的运算。如：

```java
//数学运算
@Value("#{1+2}")
private int addsum ; 

//浮点比较运算
@Value("#{beanName.pi == 3.14f}")
private int piFlag ; 

//字符串比较运算
@Value("#{ beanName.str eq 'Spring Boot' }")
pri.vate boolean strFlag ;

// 字符串连接
@Value("#{beanName.str + ' 连接字符串 '}"）
private String strApp = null ;

//＃三元运算
@Value("#{beanName.d > 1000 ？ '大于' ：'小于'}" ）
private String resultDesc = null ;
```

Spring EL 能够支持的运算还有很多，其中等值比较如果是数字型的可 以使用`==`比较符，如果是字符串型的可以使用 `eq` 比较符。当然 ， Spring EL 的内容远不止这些，只 是其他表达式的使用率没有那么高    











​     