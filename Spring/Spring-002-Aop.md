---
layout: default
title: Spring AOP
parent: Spring
grand_parent: Java
nav_order: 11
---


## Spring 归纳 

> 官网：https://spring.io/ <br/>
文档直达：[Spring 文档](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)

一般，Spring 是 `Spring Framework` 的简称。

Spring 是干嘛的？


`Spring`可以让对象与对象（模块与模块）之间的关系不在通过硬编码关联，而是通过配置类或注解标记进行管理。
`Spring`是一个容器。

## Spring AOP 主要应用场景

Authentication 权限、

Caching  缓存、

Context passing 内容传递、

Error handling 错误处理、

Lazy loading 懒加载、

Debugging  调试、

logging, tracing, profiling and monitoring 记录跟踪优化校准、

Performance  optimization 性能优化、

Persistence 持久化、

Resource pooling 资源池、

Synchronization  同步、

Transactions 事务。



## Spring AOP


AOP全称：Aspect-Oriented Programming，面向切面编程。

AOP 也是Spring 非常实用的核心组件。

AOP可以为某一类对象进行控制，实现方法增加，在调用该类对象的具体方法的前后去调用其他需要使用模块，从而达到对原有对象功能扩充目的。

Spring AOP是基于代理的框架，支持2种代理模式：
 - 使用JDK动态代理。JDK动态代理内置于JDK中。

 - CGLIB为给定的目标对象创建代理的动态代理。CGLIB是一个通用的开源类定义库，打包在`spring-core`包中。

如果要代理的目标对象至少实现了一个接口，则使用JDK动态代理。目标类型实现的所有接口都是代理的。如果目标对象没有实现任何接口，则会创建一个CGLIB代理。

强制使用CGLIB注意事项：
（1）对于CGLIB，final方法不能被通知，因为它们不能在运行时生成的子类中被重写
（2）Spring 4.0 之后，代理对象的构造函数不再被调用两次，因为CGLIB代理实例是通过objensis创建的。只有用户的JVM不允许构造函数绕过时，才会看到来自Spring的AOP支持的双重调用和相应的调试日志条目。

开启强制cglib：

```xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```
>`proxy-target-class="true"`可以用在`<tx:annotation-driven/>`，`<aop:aspectj-autoproxy/>`, `<aop:config/>`三个标签上开启强制CGLIB动态代理。

如果使用@AspectJ自动代理支持：
```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

### AOP启用与声明

1. **AOP @Aspectj的启用**

```xml
<aop:aspectj-autoproxy/>
```
或者：

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

2. **AOP切面声明**


AOP切面声明方式有两种：`XML方式` 和 `注解方式`。

- XML方式：
```xml
<bean id="myLogAspect" class="com.xiaocai.MyLogAspect">
    <!-- configure properties of the aspect here -->
</bean>
```

- 注解声明:

```java
@Aspect
public class MyLogAspect {

}
```


### 切入点支持

**切入点声明表达式支持：**

- `execution` ：用于匹配方法执行连接点。指定的连接点范围包或类。
- `within`：使用`within(类型表达式)`。匹配指定类型内的方法执行；
- `this`：常用于绑定形式。使用`this(类型全限定名)`匹配当前AOP代理对象类型的执行方法，注意是AOP代理对象的类型匹配，这样就可能包括引入接口方法也可以匹配；注意this中使用的表达式必须是类型全限定名，不支持通配符；
- `target`：使用`target(类型全限定名)`。匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；注意target中使用的表达式必须是类型全限定名，不支持通配符；
- `args`：使用`args(参数类型列表)`。匹配当前执行的方法传入的参数为指定类型的执行方法；注意是匹配传入的参数类型，不是匹配方法签名的参数类型；参数类型列表中的参数必须是类型全限定名，通配符不支持。

- `@target`：使用`@target(全限定类型名称的注解)`。匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解。注解类型也必须是全限定类型名；
- `@args`：使用`@args(全限定类型名称的注解)`。匹配当前执行的方法传入的参数持有指定注解的执行；注解类型也必须是全限定类型名；
- `@within`：使用`@within(全限定类型名称的注解)`。匹配所有持有指定注解类型内的方法；注解类型也必须是全限定类型名。
- `@annotation`：使用`@annotation(全限定类型名称的注解)`。匹配当前执行方法持有指定注解的方法；注解类型也必须是全限定类型名。

- `bean`：匹配特定命名的bean或者带通配符的一组bean。 Spring aop环境特有的。


**类型匹配语法符号：**

 - `*` ：匹配任何数量字符；
 - `..`：匹配任何数量字符的重复，如在类型模式中匹配任意数量子包，在方法参数模式中匹配任意数量参数。
 - `+` ：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。


**找点栗子说明：**

- **bean 的示例**

先看`bean`，比如将指定的bean作为切入点，参数可以是bean的id或者名字。

```java
    @Pointcut("bean(userService)")
    public void beanPoint(){

    };
```
bean还支持通配符（`*`）和逻辑符 `&&`, ` || `，` ! `。

如匹配所有的Service切入：
```java
    @Pointcut("bean(*Service)")
    public void beanPoint(){

    };
```
如匹配所有非Dao切入：
```java
    @Pointcut("bean(!loginDao)")
    public void beanNotLoginDaoPoint(){

    }
```

- `execution` 示例

    - 匹配service 包下的任意无参的方法切入：
```java
@Pointcut("execution(* cn.xiaocai.*.service+.*())")
private void anyNoParamMethodUnderPackage() {} 
```
    - 匹配 service 包下的任意只有一个参数的方法执行切入：
```java
@Pointcut("execution(* cn.xiaocai.*.service.*(*))")
private void anyOneParamMethodUnderPackage() {} 
```
    - 匹配 service 包下的不限定参数的任意方法执行切入：
```java
@Pointcut("execution(* cn.xiaocai.*.service.*(..))")
private void anyMethodUnderPackage() {} 
```
    - 匹配 service 包下的限定只有一个参数且参数类型为`java.util.Date`的方法执行切入：
```java
@Pointcut("execution(* cn.xiaocai.*.service.*(java.util.Date))")
private void givenParamaterType() {} 
```
    - 匹配 service 包下的不限定参数的任意方法且抛出`IllegalArgumentException`和`ArrayIndexOutOfBoundsException`异常执行切入：
```java
@Pointcut("execution(* cn.xiaocai.*.service.*(..)) throws IllegalArgumentException, ArrayIndexOutOfBoundsException")
private void anyPackageWithException() {} 
```
    - 匹配 任意路径下只有一个参数且参数声明（持有）`@Param`注解的任意方法执行切入：
```java
@Pointcut("execution(* *(@Param *)")
private void oneParamAnnotation() {} 
```
    - 匹配所有被`@MyLog`注解修饰的任意方法：
```java
@Pointcut("@cn.xiaocai.annotation.MyLog * *(..)")
private void myLogAnnotation() {} 
```
    - 匹配任何被`@MyLog`和 `RequestMapping`注解修饰的方法：
```java
@Pointcut("@cn.xiaocai.annotation.MyLog @org.springframework.web.bind.annotation.RequestMapping  * *(..)")
private void manyAnnotation() {} 
```
    - 匹配任何被`@BizEmail`或`BizWarn`注解修饰的方法：
```java
@Pointcut("@(cn.xiaocai.annotation.BizEmail || cn.xiaocai.annotation.BizWarn) * *(..)")
private void manyAnnotation() {} 
```
    - 匹配 任意路径下参数声明`@MyAnnotation`注解并且参数类型上也有`@MyAnnotation`注解的任意方法：
```java
@Pointcut("* *(@MyAnnotation (@cn.xiaocai.annotation.MyAnnotation *), @cn.xiaocai.annotation.MyAnnotation (@cn.javass..MyAnnotation *))")
private void manyParamsAnnotation() {} 
```

    - 任意公共方法执行切入：
```java
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} 
```

    - 任意update开头的方法切入：
```java
@Pointcut("execution(* update*(..))")
private void anyUpdateMethodOperation() {} 
```

    - AccountService接口的任意方法切入:
```java
@Pointcut("execution(* com.xyz.service.AccountService.*(..))")
private void givenInterfaceOperation() {} 
```

    - `com.xyz.service`包及子包下的任意方法切入:
```java
@Pointcut("execution(* com.xyz.service.*.*(..))")
private void givenPackageOperation() {} 
```
    - `execution` 组合引用匹配的示例：
```java
@Aspect
public class CombinePointcuts{

	// 匹配任意public方法作为切入点
	@Pointcut("execution(public * *(..))")
	private void anyPublicOperation() {} 

	//匹配com.xyz.myapp.trading 包下的方法执行作为切入点。方法执行在交易模块中，则匹配切入
	@Pointcut("within(com.xyz.myapp.trading..*)")
	private void inTrading() {
		
	} 

	// 组合引用以上两者使用，匹配交易模块中的任意public方法
	@Pointcut("anyPublicOperation() && inTrading()")
	private void tradingOperation() {}
}
```
> 最好的做法是使用较小的命名组件构建更复杂的切入点表达式。


- `within` 示例
    - 指定包下的类任意连接点：
```java
@Pointcut("within(com.xyz.service.*)")
private void givenService() {}
```
    - 匹配指定包及子包下的BizPointcutService类型及子类型的任意方法：
```java
@Pointcut("within(com.xyz.service.BizPointcutService+)")
private void givenServiceAndSub() {}
```
    - 匹配目标对象的声明类型持有`@TTransactional`注解：
```java
@Pointcut("@within(org.springframework.transaction.annotation.Transactional)")
private void anyTargethasAnnotation() {}  
```


- `this` 示例
    - Aop代理实现AccountService接口的任何连接点
```java
@Pointcut("this(com.xyz.service.AccountService)")
private void givenServiceProxyPoint() {}
```
    - 声明注解的代理连接
```java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.CommonPointcuts.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }
}
```
>这里使用`@DeclareParents`引入`UsageTracked`接口声明，就是引入接口使用接口的方法。而所有的Bean都实现了这个`UsageTracked`接口。每次调用公共切入的`businessService()` 方法时会自动找到对应的bean。  


- `target` 示例

所有实现了AccountService接口的实现类的目标对象切入连接点(针对接口)：

```java
@Pointcut("target(com.xyz.service.AccountService)")
private void anyTargetInterfaceImpl() {}  
```
特定实现类的目标对象切入连接点(针对具体的类)：

```java
@Pointcut("target(com.xyz.service.impl.AccountServiceImpl)")
private void anyTargetImplClass() {}  
```

- `args` 示例

>`args` 匹配当前执行的方法传入的参数为指定类型的执行方法；注意是匹配传入的参数类型，不是匹配方法签名的参数类型；参数类型列表中的参数必须是类型全限定名，通配符不支持；args属于动态切入点，这种切入点开销非常大，非特殊情况最好不要使用；。

匹配只有一个参数且参数类型为 `java.io.Serializable`方法切入：

```java
@Pointcut("args(java.io.Serializable)")
private void oneParamsForSerialized() {}  
```

匹配第一个参数的类型为 `java.io.Serializable`，后面跟任意个数任意类型的参数的方法切入：

```java
@Pointcut("args(java.io.Serializable,..)")
private void anyParamsForMany() {}  
```

- `@target` 示例

目标对象持有`@Transactional`注解的

```java
@Pointcut("@target(org.springframework.transaction.annotation.Transactional)")
private void anyTargethasAnnotation() {}  
```

- `@annotation` 示例 

持有`@Transactional`注解的执行方法：

```java
@Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
private void anyMethodHasAnnotation() {}  
```

 - 共享公共切入点的示例：


```java
package com.xyz.myapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class CommonPointcuts {

    /**
     * 切入方法在 com.xyz.myapp.web 包及子包下的类定义的方法
     */
    @Pointcut("within(com.xyz.myapp.web..*)")
    public void inWebLayer() {}

    /**
     * 切入方法在 com.xyz.myapp.service 包及子包下的类定义的方法
     */
    @Pointcut("within(com.xyz.myapp.service..*)")
    public void inServiceLayer() {}

    /**
     * 切入点在 com.xyz.myapp.dao 包及子包下的类定义的方法
     */
    @Pointcut("within(com.xyz.myapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * 切入方法在 业务执行方法 com.xyz.myapp..service 包及子包的实现类中,接口声明模式
     * 相当于bean(*Service) 
     */
    @Pointcut("execution(* com.xyz.myapp..service.*.*(..))")
    public void businessService() {}

    /**
     *
    * 切入方法在 业务执行方法 com.xyz.myapp.dao 包及子包的实现类中，接口声明模式
     */
    @Pointcut("execution(* com.xyz.myapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

### 切入通知

**通知声明**

通知与切入点表达式相关联，并在与切入点匹配的方法执行之前、之后或周围运行。切入点表达式可以是对命名切入点的简单引用，也可以是就地声明的切入点表达式。

几种通知形式：

- `@Before`：前置通知。
- `@After`：后置通知、最终通知。
- `@AfterReturning`：返回后通知。
- `@AfterThrowing`：抛出异常通知。目标方法抛出异常时执行。
- `@Around`：环绕通知。

#### @Before

前置通知

- 引用已经命名的切入点：
```java
@Aspect
public class MyLogAspect {

	@Pointcut("@annotation(com.xiaocai.base.demo.annotation.MyLog)")
    public void logPoint(){};

    @Before(value = "logPoint()")
    public void logBefore(){
         
    }
}
```

- 就地声明切入点:
```java
@Aspect
public class MyLogAspect {

    @Before(value = "@annotation(com.xiaocai.base.demo.annotation.MyLog)")
    public void logBefore(){
    }
}
```
其他同理。


#### @After

类似`try-catch-finally`结构里的`finally`块。

```java
@Aspect
public class MyLogAspect {
	@Pointcut("@annotation(com.xiaocai.base.demo.annotation.MyLog)")
    public void logPoint(){};

    @After(value = "logPoint()")
    public void logAfter(){
        log.info("--log After ");

    }
}
```


#### @AfterReturning 返回后通知

`@AfterReturning`的返回参数绑定到通知参数上。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class MyLogAspect {

    @AfterReturning(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
}
```
> 该通知只能从连接点（用户声明的目标方法）本身接收异常。



#### @Around 环绕通知

- 基本示例

```java
@Aspect
public class MyLogAspect {

	@Pointcut("@annotation(com.xiaocai.base.demo.annotation.MyLog)")
    public void logPoint(){};

	@Around(value = "logPoint()")
    public Object logAround(ProceedingJoinPoint joinPoint){
        log.info("--log  Around  Before --");
        // 获取方法名称
        String methodName = joinPoint.getSignature().getName();
        // 获取入参
        Object[] param = joinPoint.getArgs();
        log.info("--methodName {},  param {}", methodName, param);
        // 继续执行方法
        Object retVal = 
        try {
            retVal = joinPoint.proceed();

        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        log.info("--log  Around  After --");
        return retVal ;
    }
}
```
>环绕通知的第一个参数必须是`ProceedingJoinPoint`类型。返回类型可以声明为`void`，`proceed()`方法最多只能调用一次，可以不调用。

- 访问当前连接点
比较实用的方法：
  - getArgs（）：返回方法参数。
  - getThis（）：返回代理对象。
  - getTarget（）：返回目标对象。
  - getSignature（）：返回所建议的方法的说明。
  - toString（）：打印建议的方法的有用描述。

`JoinPoint`接口源码：

```java
package org.aspectj.lang;  
import org.aspectj.lang.reflect.SourceLocation;  
public interface JoinPoint {  
    String toString();         //连接点所在位置的相关信息  
    String toShortString();     //返回连接点的缩写字符串表示形式。
    String toLongString();     //返回连接点的扩展字符串表示形式
    Object getThis();         //返回AOP代理对象 ，返回当前正在执行的对象。
    Object getTarget();       //返回目标对象  
    Object[] getArgs();       //返回被通知方法参数列表  
    Signature getSignature();  //返回当前连接点签名  
    SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置  
    String getKind();        //连接点类型  
    StaticPart getStaticPart(); //返回一个封装此连接点的静态部分的对象
} 
```

#### 通知参数传递

使用`args`表达式进行参数绑定。

```java
@Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```
或者：
```java
@Pointcut("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```
> `args(account,..)` 的作用有两个：一是只匹配那些方法至少接受一个参数的方法执行；二是通过Account参数使通知可以使用实际的Account对象。

代理对象（this）、目标对象（target）和注释（@within、@target、@annotation和@args）都可以以类似的方式绑定。

比如我们自己定义一个注解，然后传递注解:

我的注解：

```java
/**
 * @author Xiaocai.Zhang
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface LimitChecked {
	String name() default "";
}
```

切面与通知：

```java
/**
 * @author Xiaocai.Zhang
 */
@Slf4j
@Aspect
@Component
public class LimitCheckedAspect {
	@Pointcut("@annotation(com.xiaocai.base.demo.annotation.LimitChecked)")
    public void checkedPoint(){};

    @Before(value = "checkedPoint() && @annotation(limitChecked)")
    public void checkBefore(LimitChecked limitChecked){
        String nameKey = limitChecked.name();
    }

    @Around(value = "checkedPoint() && @annotation(limitChecked)")
    public CommonResult checkAround(ProceedingJoinPoint joinPoint, LimitChecked limitChecked){
    	String nameKey = limitChecked.name();
    }
}
```
这样在前置通知或环绕通知都可以拿到对应注解的属性值。

#### 通知参数与泛型

Spring Aop 可以处理类声明和方法参数中使用的泛型。

声明接口:

```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```
通知使用：

```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```
#### 通知参数命名

> 如果用户已明确指定参数名，则使用指定的参数名。通知和`@Pointcut`注解都有一个可选的`argNames`属性，可以使用它来指定带注释方法的参数名。

```java
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code and bean
}
```

>如果第一个参数是JoinPoint、ProceedingJoinPoint或JoinPoint.StaticPart类型，则可以从`argNames`属性的值中省略参数的名称。

```java
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(JoinPoint jp, Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code, bean, and jp
}
```

不需要收集连接点上下文的，可以省略`argNames`属性。

```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod()")
public void audit(JoinPoint jp) {
    // ... use jp
}
```

#### 多个切点或切面执行先后

（1）实现 `org.springframework.core.Ordered` 接口
（2）使用`@Order`注解。
（3）数字越小，优先级越高。


在切入点添加注解：

```java
    @Order(1) // Order 代表优先级，数字越小优先级越高
    @Pointcut("@annotation(com.xiaocai.base.demo.annotation.LimitChecked)")
    public void checkedPoint(){};
```
在通知添加注解：

一般是针对同一个切入点出现多个切面或多个通知操作时。

```java
	@Order(1) 
    @Before(value = "logPoint()")
    public void logBefore(){
        log.info("--log before ");

    }
```


## 切面实例化模型

默认情况下，应用程序上下文中每个切面都有一个实例。AspectJ称之为单例实例化模型。
Spring 支持 AspectJ的 `perthis` 和 `pertarget` 的实例化。

```java
@Aspect("perthis(com.xyz.myapp.CommonPointcuts.businessService())")
public class MyAspect {

    private int someState;

    @Before("com.xyz.myapp.CommonPointcuts.businessService()")
    public void recordServiceUsage() {
        // ...
    }
}
```
`perthis`子句的作用是为执行`businessService()`的每个唯一服务对象创建一个切面实例。每个唯一的对象会在与切入点表达式匹配的连接点处绑定到该对象。

切面实例是在服务对象第一次调用方法时创建的，当服务对象不在生命周期范围内时，切面也无法进入声明周期范围。所以在创建切面实例之前，其中的任何通知都不会执行，但是只要创建了切面实例，其中声明的通知就可以在匹配的连接点上执行，当然前提是服务对象与之相关联了。

`pertarget`实例化模型的工作方式与`perthis`完全相同，但它在匹配的连接点为每个唯一的目标对象创建一个方面实例。

> 总结：perthis 和 pertarget 都可以进行切面实例化，但是`perthis`是单例模式的实例化，而`pertarget`是原型模式的实例化。


## AOP Demo

业务执行有时会由于并发问题而失败（例如，死锁失败者）。

如果重试该操作，则下次尝试可能会成功，对于这种适合重试的业务服务，可以定义一个切面去实现重试操作：


```java
@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }
}
```

该切面实现了有序接口`Ordered`，以便可以将切面的优先级设置为高于事务通知，因为每次重试时都需要一个新的事务。`maxRetries`和`order`属性都由Spring配置。主要重试操作逻辑发生在`doconcurrentooperation()`方法中。请注意，这里将重试逻辑应用于每个`businessService()`方法。 如果由于悲观的`LockingFailureException`而失败，我们将重试，知道达到我们设置的重试次数。

对应的配置：

```xml
<aop:aspectj-autoproxy/>

<bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
    <property name="maxRetries" value="3"/>
    <property name="order" value="100"/>
</bean>
```

要优化切面使其只重试幂等操作，定义一个幂等注解

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```
然后使用`@Idempotent`注解来注释服务操作的实现。

优化匹配连接点：

```java
@Around("com.xyz.myapp.CommonPointcuts.businessService() && " +
        "@annotation(com.xyz.myapp.service.Idempotent)")
public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
    // ...
}
```

## XML AOP

XML模式用的少，暂时就先不整理了。

## Spring中使用AspectJ进行加载时织入

加载时织入，LoadTimeWeaving，简称LTW。

AOP通过为目标类织入切面的方式，实现对目标类功能的增强。按切面被织如到目标类中的时间划分，主要有3种:

- 运行时织入
这是最常见的，比如在运行期通过为目标类生成动态代理的方式实现AOP就属于运行期织入，这也是Spring AOP中的默认实现,Spring也提供了两种创建动态代理的方式：自带的JDK针对接口的动态代理和使用CGLib动态创建子类的方式创建动态代理。

- 编译时织入
使用特殊的编译器在编译期将切面织入目标类,这种比较少见,因为需要特殊的编译器的支持。

- 加载时织入 

类加载时织入通过字节码编辑技术在类加载期将切面织入目标类中，它的核心思想是在目标类的class文件被JVM加载前,通过自定义类加载器或者类文件转换器将横切逻辑织入到目标类的class文件中,然后将修改后class文件交给JVM加载。这种织入方式可以简称为LTW(LoadTimeWeaving)。


Spring的LTW支持中的关键组件是`LoadTimeWeaver`接口（在`org.springframework.instrument.classloading`包）。`LoadTimeWeaver`接口源码：

```java
package org.springframework.instrument.classloading;

import java.lang.instrument.ClassFileTransformer;

public interface LoadTimeWeaver {
    void addTransformer(ClassFileTransformer var1);

    ClassLoader getInstrumentableClassLoader();

    ClassLoader getThrowawayClassLoader();
}
```

基本原理：

`Spring LTW`通过读取classpath下`META-INF/aop.xml`文件，获取切面类和要被切面织入的目标类的相关信息，再通过`LoadTimeWeaver`在`ClassLoader`加载类文件时将切面织入目标类中。

Spring中可以通过`LoadTimeWeaver`将Spring提供的`ClassFileTransformer`注册到ClassLoader类加载器中。在类加载时，注册的`ClassFileTransformer`读取类路径下`META-INF/aop.xml`文件中定义的切面类和目标类信息，在目标类的class文件真正被VM加载前织入切面信息，生成新的Class文件字节码，然后交给VM加载，然后创建目标类的实例，实现AOP功能。


spring支持需要2个包。

- spring-aop.jar
- aspectjweaver.jar
- spring-instrument.jar（如果使用Spring提供的代理来启用检测）


官方Demo

1、首先声明切入点和切面

```java
package foo;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.util.StopWatch;
import org.springframework.core.annotation.Order;

@Aspect
public class ProfilingAspect {

    @Around("methodsToBeProfiled()")
    public Object profile(ProceedingJoinPoint pjp) throws Throwable {
        StopWatch sw = new StopWatch(getClass().getSimpleName());
        try {
            sw.start(pjp.getSignature().getName());
            return pjp.proceed();
        } finally {
            sw.stop();
            System.out.println(sw.prettyPrint());
        }
    }

    @Pointcut("execution(public * foo..*.*(..))")
    public void methodsToBeProfiled(){}
}
```

2、创建创一个`META-INF/aop.xml`文件

创建创`META-INF/aop.xml`文件用于通知`AspectJ weaver`将我们的切面`ProfilingAspect`编织到类中。

`aop.xml` 配置：

```xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "https://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>
    <weaver>
        <!-- 要织入切面的目标类 这里使用的是包，也可以明确指定某个类 foo.StubEntitlementCalculationService  -->
        <include within="foo.*"/>
    </weaver>

    <aspects>
        <!-- 要织入的切面 -->
        <aspect name="foo.ProfilingAspect"/>
    </aspects>
</aspectj>
```
3、开启LTW支持

Spring 的XML配置开启LTW.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 要织入的模板 service -->
    <bean id="entitlementCalculationService"
            class="foo.StubEntitlementCalculationService"/>

    <!-- 开启LTW支持 -->
    <context:load-time-weaver/>
</beans>
```

如果是在 SpringBoot 中可以使用`@EnableLoadTimeWeaving`注解开启LTW支持。

```java
@Configuration
@ComponentScan("com.xiaocai.ltw.*")
@EnableLoadTimeWeaving(aspectjWeaving=AUTODETECT)
public class LtwAopConfig{
}
```

4、测试LTW

```JAVA
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService =
                (EntitlementCalculationService) ctx.getBean("entitlementCalculationService");

        // 验证切面织入
        entitlementCalculationService.calculateEntitlement();
    }
}
```
5、使用代理开启

可以直接使用java代理来执行main方法：

```bash
java -javaagent:C:/projects/foo/lib/global/spring-instrument.jar foo.Main
```

或者配置JVM运行参数：
```
-javaagent:C:/projects/foo/lib/global/spring-instrument.jar
```


如果是 springboot 可以引入对应的jar包组件配置

```xml
		<plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <agent>
                    ${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar
                </agent>
                <agent>
                    ${settings.localRepository}/org/springframework/spring-instrument/${spring.version}/spring-instrument-${spring.version}.jar
                </agent>
            </configuration>
        </plugin>
```

对于独立的java应用程序，spring提供了一个`InstrumentationLoadTimeWeaver`，需要一个通用的JVM代理`spring-instrument.jar`。由`@EnableLoadTimeWeaving`和 `<context:load-time-weaver/>`的配置完成自动检测。同时必须配置jvm参数：

```
-javaagent:/path/to/spring-instrument.jar
```


在上面的配置中，如果要指定特定的`LoadTimeWeaver`可以进行配置。

- 使用Java配置：

指定特定的`LoadTimeWeaver`需要实现`LoadTimeWeavingConfigurer`接口并重写`getLoadTimeWeaver()`方法

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig implements LoadTimeWeavingConfigurer {

    @Override
    public LoadTimeWeaver getLoadTimeWeaver() {
        return new ReflectiveLoadTimeWeaver();
    }
}
```
- 使用XML配置：

直接配置对应的`weaver-class`即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:load-time-weaver
            weaver-class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>

</beans>
```