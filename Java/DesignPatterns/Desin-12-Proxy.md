---
layout: default
title: Proxy Pattern
parent: Design Patterns
has_children: false
nav_order: 22
---

Here are java Proxy Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是代理模式？

代理模式（Proxy Pattern）

代理模式是一种结构型设计模式，它为另一个对象提供一个替身或占位符，以控制对这个对象的访问。

代理对象在客户端和目标对象之间起到中介作用，可以在不修改目标对象的情况下，增加额外的功能（如权限校验、日志、延迟加载、远程调用等）。

## 模式结构

```
┌──────────────┐         ┌──────────────────┐
│   Subject    │         │     Proxy        │
│ + request()  │◄────────│ - realSubject    │
└──────┬───────┘         │ + request()      │
       ▲                 └────────┬─────────┘
       │ 实现                      │ 委派
┌──────┴───────┐         ┌────────▼─────────┐
│ RealSubject  │         │  preProcess()    │
│ + request()  │         │  postProcess()   │
└──────────────┘         └──────────────────┘
```

核心角色：

- Subject（抽象主题）：定义真实对象和代理对象的共同接口。
- RealSubject（真实主题）：真正的业务逻辑对象。
- Proxy（代理）：持有对 RealSubject 的引用，控制对其访问，并可附加额外操作。

三种常见的代理实现方式：
- 静态代理
- JDK 动态代理
- CGLIB 动态代理。
  
##  静态代理

原理：在编译期就确定了代理类，由程序员手动编写一个与目标类实现相同接口的代理类。代理类在编译后生成 .class 文件。

业务案例：用户权限控制

假设有一个 UserService 接口，真实实现 UserServiceImpl 提供用户查询功能。现在需要对所有查询操作进行权限校验——只有管理员才能查询用户详情。
 
 
1. 抽象主题
```java
public interface UserService {
    User getUserById(Long id);
}
```

2. 真实主题
```java
public class UserServiceImpl implements UserService {
    @Override
    public User getUserById(Long id) {
        System.out.println("从数据库查询用户：" + id);
        return new User(id, "张三");
    }
}
```

3. 静态代理类
```java
public class UserServiceStaticProxy implements UserService {
    private UserService target;
    private String currentUserRole;

    public UserServiceStaticProxy(UserService target, String role) {
        this.target = target;
        this.currentUserRole = role;
    }

    @Override
    public User getUserById(Long id) {
        // 前置增强：权限校验
        if (!"ADMIN".equals(currentUserRole)) {
            throw new SecurityException("无权访问用户信息");
        }
        System.out.println("权限校验通过");

        // 调用真实方法
        User user = target.getUserById(id);

        // 后置增强：记录日志
        System.out.println("查询用户日志记录：" + id);
        return user;
    }
}
```
4. 客户端（棋盘控制器）
```java
public class Client {
    public static void main(String[] args) {
        UserService realService = new UserServiceImpl();
        UserService proxy = new UserServiceStaticProxy(realService, "ADMIN");
        User user = proxy.getUserById(1L);
    }
}
```

### 优缺点

|优点 | 缺点|
|-----|----|
|实现简单，容易理解 |每个目标类都需要一个代理类，类数量膨胀|
|可以在不修改目标类的前提下扩展功能 |如果接口发生变化，代理类也必须同步修改|
|编译期确定，运行效率高 |不适合代理大量类|

### 适用场景

- 代理类数量少且稳定，不易变化。
- 对性能要求极高，不希望引入反射开销。

### 现实应用

- 早期 Java RMI 的 Stub 和 Skeleton。
- 自定义 AOP 切面，但只针对少量类。


##  JDK 动态代理

原理：利用 Java 的 java.lang.reflect.Proxy 类和 InvocationHandler 接口，在运行时动态生成代理对象。

要求目标类必须实现一个或多个接口。代理对象在内存中生成，不写入磁盘。

业务案例：统一日志记录

假设系统中有多个 Service（UserService、OrderService），都需要在执行每个方法前后打印日志。

用 JDK 动态代理可以一次性为所有实现接口的 Service 添加日志。

**InvocationHandler 实现**

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class LoggingHandler implements InvocationHandler {
    private Object target;

    public LoggingHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("[日志] 开始调用方法：" + method.getName());
        long start = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long elapsed = System.currentTimeMillis() - start;
        System.out.println("[日志] 方法结束：" + method.getName() + "，耗时：" + elapsed + "ms");
        return result;
    }

    // 工厂方法：创建代理对象
    @SuppressWarnings("unchecked")
    public static <T> T createProxy(T target) {
        return (T) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new LoggingHandler(target)
        );
    }
}
```

使用
```java
public class Client {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        UserService proxy = LoggingHandler.createProxy(userService);
        proxy.getUserById(1L);

        OrderService orderService = new OrderServiceImpl();
        OrderService orderProxy = LoggingHandler.createProxy(orderService);
        orderProxy.createOrder(1001);
    }
}
```
### 优缺点

|优点 | 缺点 |
|----|------|
|无需手动编写代理类，减少代码量 |只能代理接口，不能代理类|
|可扩展性强，一个 InvocationHandler 可代理多个目标| 性能略低于静态代理（有反射开销）|
|符合开闭原则，新增功能只需修改 handler| 目标类没有实现接口时无法使用|

### 适用场景

- 目标类面向接口编程。
- 需要为大量类添加相同的横切逻辑（如日志、事务、权限）。

### 现实应用

- Spring AOP：默认使用 JDK 动态代理（当目标类实现了接口时）。
- MyBatis：Mapper 接口的动态代理实现。
- Java RMI：远程对象的动态 Stub。



## CGLIB 动态代理

原理： CGLIB（Code Generation Library）通过字节码增强技术（ASM）在运行时生成目标类的子类作为代理对象。

因此它可以代理没有实现接口的类，但不能代理 final 类或 final 方法。

业务案例：对普通类进行事务管理

假设有一个 UserDao 类（没有实现接口），需要在其所有数据库操作方法前后添加事务控制。

**CGLIB 代理实现**

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class TransactionInterceptor implements MethodInterceptor {
    private Object target;

    public TransactionInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("开启事务");
        try {
            Object result = proxy.invoke(target, args);  // 调用真实方法
            System.out.println("提交事务");
            return result;
        } catch (Exception e) {
            System.out.println("回滚事务");
            throw e;
        }
    }

    // 工厂方法
    @SuppressWarnings("unchecked")
    public static <T> T createProxy(T target) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(new TransactionInterceptor(target));
        return (T) enhancer.create();
    }
}
```
被代理类（无接口）
```java
public class UserDao {
    public void save(User user) {
        System.out.println("插入用户：" + user.getName());
    }
    public void update(User user) {
        System.out.println("更新用户：" + user.getId());
    }
}
```
客户端使用
```java
public class Client {
    public static void main(String[] args) {
        UserDao dao = new UserDao();
        UserDao proxy = TransactionInterceptor.createProxy(dao);
        proxy.save(new User(1L, "张三"));
        proxy.update(new User(1L, "李四"));
    }
}
```

### 优缺点

|优点 | 缺点|
|----|----|
|可以代理普通类（无需接口） |不能代理 final 类或 final 方法|
|性能优于 JDK 动态代理（CGLIB 使用 FastClass 机制，反射调用少） |需要额外引入 CGLIB 依赖（Spring 已内置）|
|生成的代理类更高效 |创建代理对象时较慢（字节码生成）|

### 适用场景

- 目标类没有实现接口。
- 需要对类的方法进行增强，且该类不是 final 的。

### 现实应用

- Spring AOP：当目标类没有实现接口时，自动切换到 CGLIB 代理。
- Hibernate：延迟加载时对实体类生成 CGLIB 代理。
- EasyMock / Mockito：部分 Mock 框架使用 CGLIB 生成模拟对象。


## 三种代理对比总结

|特性	|静态代理	|JDK 动态代理	|CGLIB 动态代理|
|-------|----------|------|-----------|
|代理生成时机	|编译期	|运行期	|运行期|
|是否需要接口	|是	|必须	|否|
|代理方式	|手动编码实现接口	|反射 + Proxy	|字节码生成子类|
|性能（调用）	|最高	|中等（反射）	|较高（FastClass）|
|可维护性	|差（类膨胀）	|好	|好|
|限制	|每个类需独立代理	|目标必须有接口	|不能代理 final 类/方法|

>注意：享元模式强调共享相同内部状态的对象，而对象池强调的是复用对象实例（即使状态不同也要重置后复用）。

## 代理模式通用优缺点


**优点**

- 职责清晰：代理类负责控制访问，真实类专注业务逻辑。
- 扩展性强：可以在不修改真实类的情况下增加新功能（如日志、缓存、鉴权）。
- 智能控制：可实现延迟加载、访问控制、远程通信等高级功能。

**缺点**

- 增加系统复杂度：引入代理层，可能使调用链路变长。
- 响应变慢：代理层增加额外处理（如反射、字节码增强）可能影响性能。

## 代理模式通用适用场景

- 远程代理 ：为一个位于不同地址空间的对象提供本地代表（如 RPC、WebService）。
- 虚拟代理 ： 延迟创建开销大的对象，直到真正需要时才加载（如图片懒加载）。
- 保护代理 ： 控制对敏感对象的访问权限（如权限校验）。
- 智能引用 ： 在访问对象时附加额外操作（如引用计数、日志记录）。

## 现实应用

|领域	|示例|
|-------|----|
|Spring AOP |	基于 JDK 动态代理或 CGLIB 实现声明式事务、日志、安全等。|
|MyBatis |	Mapper 接口通过 JDK 动态代理生成实现类。|
|Java RMI |	远程对象通过 Stub（代理）进行网络通信。|
|Hibernate |	延迟加载时使用 CGLIB 代理实体类。|
|Android Retrofit |	通过动态代理将 HTTP API 接口转化为网络请求。|
|Nginx 反向代理 |	作为 Web 服务器的代理，实现负载均衡、缓存、安全过滤。|

## 总结：

代理模式的核心是间接访问，通过代理对象在客户端和真实对象之间插入一层，从而实现控制、增强或延迟。

在实际开发中，JDK 动态代理和 CGLIB 动态代理是主流选择，前者适用于接口场景，后者适用于类场景。

Spring AOP 会根据目标类是否实现接口自动选择最优方案。

记住一句话：“想给对象加功能又不想改它？找个代理就行了。”