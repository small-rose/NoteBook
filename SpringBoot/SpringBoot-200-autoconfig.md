---
layout: default
title: SpringBoot AutoConfig
parent: SpringBoot
nav_order: 99
---

## SpringBoot  自动装配 <!-- {docsify-ignore} -->

### 什么是SpringBoot

SpringBoot 自动装配。

Springboot主要思想是遵循"约定优于配置"的设计，使用注解对一些常规的配置项做默认配置，减少使用XML配置，让你的项目结构快速搭建运行起来。

Springboot针对主流常用的框架技术将其封装成starter组件，现在引入框架只要引入一个starter，你就可以使用这个框架，只需少量的配置甚至是不需要任何配置就可以实现快速集成。可以让开发者关注业务，减少技术开发成本。


### 自动装配原理


Spring Boot中的自动装配常用技术：

1、Spring 模式注解装配。 如`@AutoConfiguration`，`@Configration`。

2、Spring `@Enable`模块装配。通过`@EnableXxx`注解灵活开启某些模块组件，如`@EnableAutoConfiguration`开启默认组件的自动化装配。

3、Spring 条件装配装。通过`@ConditionalOnXxx`注解进行条件判断是否加载相关组件。

4、Spring 工厂加载机制。通过读取`META-INF/spring.factories`加载对应的自动配置类。


###  模式注解装配

#### 模式注解

`stereotype annotation` 一般翻译为模式注解，样板注解。

什么叫模式注解？

比如Spring中常见的模式注解有`@Component`，`@Service`，`@Repository`，`@Controller` ，后面3个是`@Component`的派生注解，凡是被`@Component`标注的类都会被Spring扫描并纳入到IOC容器中。那么后面三个注解就有`@Component`相同的作用。

`@Component` 定义：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}

```

再看看其中一个派生注解`@Controller`的定义：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```
可以发现 `@Controller`注解是被`@Component` 修饰的，确实是从`@Component`派生出来的模式注解，在开发中我们使用过很多次，肯定是能够被扫描到IOC容器中的。

还有`@RestController`, `@RequestMapping`， `@ResponseBody`等之类的。

>模式注解，可以理解为就是这个注解专门用来处理解决某个事情的。让我联想到设计模式的单一职责原则。注解的本质不也是一个类吗？只是这个类比较特殊，可以对其他元素进行注解修饰。

#### 模式注解装配 

那么在SpringBoot中，也有不少派生注解，如`@SpringBootApplication`，`@Configuration`，`@AutoConfiguration` 注解。比如`SpringBootApplication` 注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {

}
```
正好被`@SpringBootConfiguration` 修饰，正好可以看看：

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}

```
正好被`@Configuration` 修饰，正好可以看看：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";

    boolean proxyBeanMethods() default true;
}

```

发现了，竟然也是被`@Component`修饰，而且还很有层次：


```
└─----- @Component
      └─----- @Configuration
            └─----- @SpringBootConfiguration
                  └─----- @SpringBootApplication
```
这一层一层的像不像楼梯一样可爱？ 🤣 


这些都是派出来的注解。


### @Enable模块驱动

什么是模块驱动？

找个例子怎么判断呢？在逐渐转向面向注解开发之后，模块驱动典型特征是**以`@Enable`开头的注解**。

比如Spring 中 开启web模块的`@EnableWebMvc`，开启事物管理的`@EnableTransactionManagement`，开启缓存模块的`@EnableCaching`，开启异步执行的`@EnableAsync`，开启响应编程的`@EnableWebFlux`等。

比如SpringBoot里的自动装配`@EnableAutoConfiguration`，开启属性绑定的`@EnableConfigurationProperties`,

模块驱动其实就是功能组件开发模块化过程，需要使用的时候通过一个注解开启模块的使用，不使用的时候不加载模块相关组件和配置。

>模块驱动可以简化组件使用时的配置步骤，实现按需装配，典型的可插拔式使用，同时屏蔽组件集合装配的细节。

Srping实现`@Enable`模块驱动的方法大致分为两类: **注解驱动** 和 **接口编程**。

#### 实现方式一：注解驱动

注解驱动的实现`@Enable`模块驱动，可以参考现有的`@EnableWebMvc`或`EnableScheduling`：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import({SchedulingConfiguration.class})
@Documented
public @interface EnableScheduling {
}
```
比普通的注解多了一个`@Import({SchedulingConfiguration.class})`，使用`@Import`注解导入了一个`SchedulingConfiguration`的配置类。

```java
@Configuration(
    proxyBeanMethods = false
)
@Role(2)
public class SchedulingConfiguration {
    public SchedulingConfiguration() {
    }

    @Bean(
        name = {"org.springframework.context.annotation.internalScheduledAnnotationProcessor"}
    )
    @Role(2)
    public ScheduledAnnotationBeanPostProcessor scheduledAnnotationProcessor() {
        return new ScheduledAnnotationBeanPostProcessor();
    }
}
```
可以看到里面使用`@Bean`的方式注入了一个Bean。

有人就要说了这个玩意怎么就能加入到容器呢？

```java
public class ScheduledAnnotationBeanPostProcessor implements ScheduledTaskHolder, 
	MergedBeanDefinitionPostProcessor, DestructionAwareBeanPostProcessor, Ordered, 
	EmbeddedValueResolverAware, BeanNameAware, BeanFactoryAware, 
	ApplicationContextAware, SmartInitializingSingleton, 
	ApplicationListener<ContextRefreshedEvent>, DisposableBean {

}
```
这个名字很长的玩意`ScheduledAnnotationBeanPostProcessor` 实现了那么多接口，乍一看是很吓人啊~ 不过不要担心，里面有一堆熟悉spring的接口呀`BeanNameAware`,
`BeanFactoryAware`，`ApplicationContextAware`，`SmartInitializingSingleton`。不说别的，就凭它实现了Spring的一系列接口，不能加入到容器，我吃了它。


> 基于注解驱动实现`@Enable`模块驱动，本质就是通过`@Import`注解来导入一个对应配置类，来实现将相应的模块组件注册到IOC容器中，那么这个模块对应的功能也就可以让程序使用。


#### 注解驱动示例

##### 1、先定义个`Bean` <!-- {docsify-ignore} -->

```java
package com.xiaocai.base.demo.auto;

/**
 * @author Xiaocai.Zhang
 */
@Data
public class HelloWorldService {

    private String name ;
    
    public HelloWorldService(String name){
        this.name = name;
    }

    public HelloWorldService(){
        this("helloWorld");
    }
}
```
这里两个构造方法，一个可以自定义名字，一个无参数的给默认名字。

##### 2、 定义一个配置类 <!-- {docsify-ignore} -->

因为要使用`@Import`导入配置类，那就定义一个属于自己的配置类

```java
package com.xiaocai.base.demo.auto;

@Configuration
public class HelloWorldConfiguration {

    @Bean
    public HelloWorldService helloWorldService(){
        return new HelloWorldService("XiaoCaiService");
    }
}
```

##### 3、定义`@Enable`注解驱动  <!-- {docsify-ignore} -->

定义对应的`@Enable`注解驱动`@EnableHelloWorld`

```java
/**
 * @author Xiaocai.Zhang
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(HelloWorldConfiguration.class)
public @interface EnableHelloWorld {

    
}
```
##### 4、测试验证 <!-- {docsify-ignore} -->

```java
package com.xiaocai.base.demo.auto;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @author Zongyuan.Zhang
 */
@EnableHelloWorld
public class HelloWordTest {

    public static void main(String[] args) {

        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(HelloWordTest.class);
        context.refresh();
        HelloWorldService helloWorld = context.getBean("helloWorldService", HelloWorldService.class);
        System.out.println(helloWorld.getName());
    }
}
```

输出
```
XiaoCaiService
```

**结论：**使用`@EnableHelloWorld`通过导入配置`HelloWorldConfiguration`成功注册`HelloWorldService`组件。


##### 关于`@Import`注解  <!-- {docsify-ignore} -->

`@Import`注解是Spring 注册组件的其中一种方式。

一般如果是个普通的类，可以直接导入，导入的名字默认为全类名

```java
/**
 * @author Xiaocai.Zhang
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(HelloWorldService.class)
public @interface EnableHelloWorld {

    
}
```
但是这种方式每次只能导入一个类，我要导入一堆组件怎么办？ 当然了`@Import`注解本身就支持多个类导入，但是一个一个的写，如果导入比较多还是很麻烦，Spring 已经想好了。




#### 实现方式二：接口编程

接口编程方式实现模块驱动，相对于注解驱动，该种方式稍微复杂一些，参考现有的`@EnableCaching`示例：


```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({CachingConfigurationSelector.class})
public @interface EnableCaching {
    boolean proxyTargetClass() default false;

    AdviceMode mode() default AdviceMode.PROXY;

    int order() default 2147483647;
}
```

比普通的注解多了一个`@Import({CachingConfigurationSelector.class})`，使用`@Import`注解导入了一个`CachingConfigurationSelector`的类，虽然不知道这个类是干嘛的，但是可以向上找看看：

```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {

}

// 父级接口
public abstract class AdviceModeImportSelector<A extends Annotation> implements ImportSelector{

}
```
从代码可以看到，该类间接实现了`ImportSelector`接口。

那`ImportSelector`接口又是干嘛的？

刚才留了一个问题`@Import`要导入一堆组件怎么办？这个两个都有个Import关键字会不会有关系呢？

```java
public interface ImportSelector {

    /**
     * Select and return the names of which class(es) should be imported based on
     * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
     */
     String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```
`ImportSelector`接口包含一个`selectImports()`方法，方法返回类的全类名数组，即需要导入到IOC容器中组件的全类名数组，包含一个`AnnotationMetadata`类型入参，通过这个参数我们可以获取到使用`ImportSelector`的类的全部注解信息。

也就说是`@Import({CachingConfigurationSelector.class})` 这个东东其实是`CachingConfigurationSelector`类借助父类`AdviceModeImportSelector`实现了`ImportSelector`接口的特征，从而导入了一个组件全类名的数组，向IOC容器注册组件。

> 接口编程实现@Enable模块驱动的本质是：通过@Import来导入`ImportSelector`接口的实现类，该实现类里可以定义需要注册到IOC容器中的组件，以此实现相应模块对应组件的注册。

多说一句：

>`@Imprt`注解除了传一个`@Configuration`类外，同时也支持传`ImportSelector`或者`ImportBeanDefinitionRegistrar`的实现类。


#### 接口编程示例

这次就简单粗暴点吧~

##### 1、自定义组件  <!-- {docsify-ignore} -->

我这里定义三个简单的类：HelloZhang，HelloXiao，HelloCai，分别返回一个字符串组成一个名字：

```java
/**
 * @author Xiaocai.Zhang
 */
@Data
public class HelloZhang {

    private String name ;

    public HelloZhang(String zhang) {
        this.name = zhang;
    }

    public HelloZhang(){
        this("Zhang");
    }
}

/**
 * @author Xiaocai.Zhang
 */
@Data
public class HelloXiao {
    private String name ;

    public HelloXiao(String xiao) {
        this.name = xiao;
    }

    public  HelloXiao(){
        this("xiao");
    }
}

/**
 * @author Xiaocai.Zhang
 */
@Data
public class HelloCai {
    private String name ;

    public  HelloCai(){
        this("Cai");
    }
    public HelloCai(String cai) {
        this.name = cai ;
    }
}
```
##### 2、定义`ImportSelector`组件   <!-- {docsify-ignore} -->


自定义一个`ImportSelector`实现类`MyImportSelector`：

```java
/**
 * @author Xiaocai.Zhang
 */
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return  new String[]{
                HelloZhang.class.getName(),
                HelloXiao.class.getName(),
                HelloCai.class.getName()
        };
    }

    @Override
    public Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

##### 3、定义模块驱动   <!-- {docsify-ignore} -->

```java
/**
 * @author Xiaocai.Zhang
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({MyImportSelector.class})
public @interface EnableHello {
}
```


##### 4、测试验证  <!-- {docsify-ignore} -->

```java
/**
 * @author Zongyuan.Zhang
 */
@EnableHello
public class HelloTest {

    public static void main(String[] args) {

        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(HelloTest.class);
        context.refresh();


        HelloZhang helloZhang = context.getBean("com.xiaocai.base.demo.auto.HelloZhang", HelloZhang.class);
        HelloXiao helloXiao = context.getBean("com.xiaocai.base.demo.auto.HelloXiao", HelloXiao.class);
        HelloCai helloCai = context.getBean("com.xiaocai.base.demo.auto.HelloCai", HelloCai.class);
        System.out.println(helloZhang.getName()+" "+helloXiao.getName()+" "+ helloCai.getName());
    }
}
```
输出：

```
Zhang xiao Cai
```


### 工厂加载机制

Spring 工厂加载机制的实现类为`SpringFactoriesLoader`。

```java
public final class SpringFactoriesLoader {

public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

static final Map<ClassLoader, Map<String, List<String>>> cache 
				 = new ConcurrentReferenceHashMap();

}
```

该类的`loadSpringFactories()`方法会读取`spring-boot-autoconfigure-2.4.0.RELEASE.jar`里的`META-INF/spring.factories`这个配置文件，并将读到的配置加载到自己的静态变量`cache`中。

`spring.factories` 部分组件：

```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
```


当启动类被`@EnableAutoConfiguration`标注后，上面所以配置在`org.springframework.boot.autoconfigure.EnableAutoConfiguration`中的所有类Spring都会去扫描，看是否可以纳入到IOC容器中进行管理。

比如我们查看`org.springframework.boot.autoconfigure.aop.AopAutoConfiguration`的源码：

 

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {

}
```
可看到该类上标注了一些注解，其中`@Configuration`为模式注解，`ConditionalOnClass`为条件装配，`@EnableConfigurationProperties`为注解式模块装配。


### 工厂加载组件示例

前面已经写过的就不重复写了，原来的写好的一些组件加一些初始化打印:

（1）组件类`HelloWorldService`:

```java
/**
 * @author Xiaocai.Zhang
 */
@Data
public class HelloWorldService {

    private String name ;

    public HelloWorldService(String name){
        this.name = name;
    }

    public HelloWorldService(){
        this("helloWorld");
    }
}
```

（2）组件配置类`HelloWorldConfiguration`:

```java
/**
 * @author Xiaocai.Zhang
 */
@Configuration
public class HelloWorldConfiguration {
    public HelloWorldConfiguration(){
        System.out.println(" init HelloWorldConfiguration");
    }
    @Bean
    public HelloWorldService helloWorldService(){
        System.out.println(" init helloWorldService");

        return new HelloWorldService("XiaoCaiService");
    }
}

```
（2）模块驱动`@EnableHelloWorld`:

```java
/**
 * @author Xiaocai.Zhang
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(HelloWorldConfiguration.class)
public @interface EnableHelloWorld {


}
```

##### 1、写个自己的自动配置类  <!-- {docsify-ignore} -->

```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Configuration;

/**
 * @author Xiaocai.Zhang
 */
@Configuration
@EnableHelloWorld
@ConditionalOnProperty(prefix = "helloWorld",name = "enabled", havingValue = "true")
public class MyHelloWorldConfiguration {

    public MyHelloWorldConfiguration(){
        System.out.println(" init MyHelloWorldConfiguration");
    }
}
```

##### 2、创建spring.factories文件  <!-- {docsify-ignore} -->

然后在`resources`目录下新建`META-INF`目录，并创建`spring.factories`文件：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.xiaocai.base.demo.auto.MyHelloWorldConfiguration
```

##### 3、添加配置  <!-- {docsify-ignore} -->

在配置文件`application.properties`中添加

```properties
helloWorld.enabled=true
```

##### 4、测试一下  <!-- {docsify-ignore} -->

```java
/**
 * @author Xiaocai.Zhang
 */
@EnableAutoConfiguration
public class MyHelloWorldCfgTest {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(MyHelloWorldCfgTest.class)
                .web(WebApplicationType.NONE)
                .run(args);
        HelloWorldService hello = context.getBean("helloWorldService", HelloWorldService.class);
        System.out.println("name : " + hello.getName());
        context.close();
    }

}
```