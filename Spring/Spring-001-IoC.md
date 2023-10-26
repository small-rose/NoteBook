---
layout: default
title: Spring IOC
parent: Spring
nav_order: 10
---


## Spring IOC

IoC的概念：对象控制权由对象本身转向容器，由容器根据配置文件或注解去创建实例和各个实例之间的依赖关系。
IoC的核心：IoC的核心是bean工厂，在Spring中，bean工厂创建的各个实例称作bean 。 

Spring 最核心的功能就是IOC了，动态注入，让一个对象的创建不再使用`new`关键字，就可以自动的生产对象，本质是利用java里的反射原理，在运行时动态调用对象的构造方法，完成对象的创建。

Spring就是在运行时，跟根据对应XML配置或相关注解注解来动态的创建对象，和调用对象里的方法的 。  

> `org.springframework.beans`包和`org.springframework.context`包是Spring Framework的IoC容器的基础。

### IoC 容器接口 <!-- {docsify-ignore} -->

`org.springframework.context.ApplicationContext`接口表示`Spring IoC容器`，并负责实例化，配置和组装Bean。`IoC容器`通过读取配置元数据获取有关实例化、配置和组装哪些对象的指令。配置元数据用XML、Java注释或Java代码表示。

这里术语中说的配置元数据就包括我们经常说的bean，当然还包括一些其他内部组件。

官方术语听着陌生，以下就统一简称Bean。


>可以使用`ApplicationContext`接口的实现`ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`的实例，加载少量XML配置来声明性地启用对这些其他元数据格式的支持，从而指示容器将Java注释或代码用作元数据格式。
如配置了开启注解模式的配置`<context:annotation-config/>`后可以直接使用注解了。


这是一个传统容器示例：

初始化容器:

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml", "services.xml")
```

对应的配置：
```xml
    <bean id="user" class="com.xiaocai.beans.UserBean">
        <property name="name" value="zhangxiaocai"></property>
    </bean>
```

使用容器：

`ApplicationContext`是高级工厂的接口，该工厂能够维护不同bean及其依赖关系的注册表。通过使用方法`T getBean(String name，Class <T> requiredType)`，可以检索bean的实例。

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

// retrieve configured instance
UserBean service = context.getBean("user", UserBean.class);

// use configured instance
String name = service.getName();
```

但是现在更多的是基于注解开发了。

 
## 1.1 基于注解的容器配置

> Spring 2.5 引入了对基于注解的配置元数据的支持。

现在spring的配置方式基本上都会涵盖3种：`XML方式`、 `基于注解配置`和`基于Java配置`，要么三种方式都支持，要么支持部分方式。原来的XML方式就不再列举了。这是整理的一些常用的注解：

- `@Required`：适用于bean属性setter方法。
- `@Autowired`：一般应用于构造方法，可以应用于字段，也可用于传统的setter方法，还可以用于具有任意名称和多个参数的方法。支持与构造函数混合使用。官方推荐使用于构造方法。
- `@Primary`：优先符注解。注册多个相同类型的Bean时，`@Primary`指示优先考虑特定的bean。
- `@Qualifier`：修饰符注解。多个相同的类型的Bean使用过程使用修饰符。支持使用泛型的方式进行自动识别修饰。

- `@Value`：一般用于注入外部化属性，就是我们常说的属性注入、数据绑定。
- `@Scope("singleton")`：作用域注解。
- `@Lazy`： 延迟加载。

大多是JSR330注解。至于具体哪些是JSR330，哪些是JSR250，不算特别重要吧？

> Spring 3.0 开始一些新特性。

- `@Component`：通用型组件注解。会隐式调用构造方法。`@Repository`，`@Service`，`@Controller`是`@Component`的特殊化，其中 `@Repository` 注解支持自动转换异常。


- `@DependsOn`：依赖型注解，启动和停止的顺序。
 
其他的一些：

- `@RestController`: 组合注解，相当于 `@Controller` + `@ResponseBody`组合。
- `@SessionScope`：解扩展，等同于`@Scope(WebApplicationContext.SCOPE_SESSION)` 只是做一个包装来简化使用。
- `@ComponentScan`：组件扫描注解。用于自动指定检测组件。

- `@Profile`：profile环境注解。profile属性表达式允许表达更复杂的profile逻辑。
- '@Conditional':条件注解。

## 1.2 基于Java Config的容器配置

Java规范的注解，典型特征是包路径一般以`javax`开头。

- `@Configuration`：配置组件注解，是一个特殊的`@Component`。相当于xml里的`<beans/>`标签。
- `@Bean`：注册组件。相当于xml里的`<bean/>`标签。

- `@PropertySource`


- `@Resource`：JSR-250注解，功能类似`@Autowired`。

- `@Inject`: JSR330注解。功能类似`@Autowired`。
- `@Named` 和 `@ManagedBean`:功能类似`@Component`。
- `@PostConstruct`和`@PreDestroy`：JSR-250 的java注解，生命周期交互相关的注解。jdk9分离，jdk11移除。

- `@Singleton`: 相当于`@Scope("singleton")`。

- ~~`@javax.inject.Qualifier`~~ / ~~`@Named`~~：类似`@Qualifier`。


- `@Lookup`：方法查找
- `@Import`：导入组件的注解。


说明：

(1)`@Autowired`是Spring自带的，`@Inject`是JSR330规范实现的，`@Resource`是JSR250规范实现的，导包不同。

(2)`@Autowired`、`@Inject`用法基本一样，不同的是`@Autowired`有一个request属性。

(3)`@Autowired`、`@Inject`是默认按照类型匹配的，`@Resource`是按照名称匹配的。

(4)`@Autowired`如果需要按照名称匹配需要和`@Qualifier`一起使用，`@Inject`和`@Name`一起使用。






## 2. bean 相关

`spring ioc容器`管理一个或多个bean。这些bean是提供给容器的配置元数据创建的。

在容器本身中，这些bean定义被表示为BeanDefinition对象，一般包含以下元数据：

- 包限定类名：通常是定义的bean的实际实现类。
- Bean行为配置元素，说明Bean在容器中的行为（范围、生命周期回调等等）。
- 对bean执行其工作所需的其他bean的引用。这些引用也称为协作者或依赖项。
- 在新创建的对象中设置的其他配置设置，例如，池的大小限制或要在管理连接池的bean中使用的连接数。


## 2.1 Bean 注册

Bean 注册有以下几种方式

- 使用XML的`<bean/>`标签
- 使用 `@Bean` 注解
- 使用 `@ComponentScan` 组件扫描
- 使用 `@Import` 导入组件
- 使用 FactoryBean 接口注册

### XML标签注册

-  1️⃣  传统的XML容器注册Bean：

使用`<bean/>`直接放到容器中。

```xml
    <bean id="user" class="com.xiaocai.beans.UserBean">
        <property name="name" value="zhangxiaocai"></property>
    </bean>
```
使用时取出：

```java
// 返回 IOC 容器，基于 XML配置，传入配置文件的位置
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
User user = (User) applicationContext.getBean("user");
```

### @Bean 注解注册

- 2️⃣  使用`@Bean`注解注册

```java
@Configuration
public class AppConfig {
	@Bean
	public UserBean user(){
		return new UserBean();
	}
}
```
配置组件扫描来识别组件注解：

```xml
<context:component-scan base-package=""></context:component-scan>
```
`base-package`指定了扫描的路径。路径下所有被`@Controller`、`@Service`、`@Repository`和`@Component`注解标注的类都会被纳入IOC容器中。



### 扫描组件注册

- 3️⃣  扫描组件注册Bean

使用`@ComponentScan`添加到`@Configuration`类中，其中`basePackages`属性描述组件包路径：

组件Service：

```JAVA
@Component
public class UserService  {
    
}
```

配置扫描：

```java
@Configuration
@ComponentScan(basePackages = "com.xiaocai.seivice")
public class AppConfig  {
    
}
```

`@ComponentScan`支持组件扫描过滤：


```java
@Configuration
@ComponentScan(basePackages = "com.xiaocai",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*test.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    
}
```

includeFilters 指定需要扫描的过滤条件。 excludeFilters 指定排除扫描的过滤条件。


### @Import导入组件

- 4️⃣ 使用`@Import`注解向容器导入组件

```java
@Configuration
@Import({HelloWorldService.class})
public class AppConfig {
	
}
```
打印一下组件名字：

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
String[] beanNames = context.getBeanDefinitionNames();
Arrays.stream(beanNames).forEach(System.out::println);
```
使用该种方式注册bean，bean的id默认为Bean的全类名。

<hr/>

### @Import扩展

**@Import扩展**：关于导入组件的方式有三个：

(1) 直接使用`@Import`注解。

(2) `ImportSelector`接口。适合一次性导入较多组件，可以使用`ImportSelector`来实现。示例如下，也可以参考[SpringBoot使用 @Import 实现模块组件装配](pages/后端/SpringBoot/SpringBoot-AutoConfig?id=实现方式二：接口编程)

(3) `ImportBeanDefinitionRegistrar`接口。


**使用`ImportSelector`接口配合`@Import`注解导入多个组件**

`ImportSelector`接口源码：

```java
public interface ImportSelector {

    /**
     * Select and return the names of which class(es) should be imported based on
     * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
     */
     String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

`ImportSelector`是一个接口，包含一个`selectImports`方法，方法返回类的全类名数组，即需要导入到IOC容器中组件的全类名数组，入参数是AnnotationMetadata类型，通过这个参数我们可以获取到使用`ImportSelector`的实现类的全部注解信息。

（1）首先要(直接或间接)实现`ImportSelector`接口。

```java
/**
 * @author Xiaocai.Zhang
 */
public class HelloImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{
        	HelloWorldService.class.getName(),
        	HelloWorldTestService.class.getName()
        };
        // return new String[]{
        // "com.xiaocai.base.demo.auto.HelloWorldService",
        // "com.xiaocai.base.demo.auto.HelloWorldTestService" };
    }
}
```
（2）然后在配置类的`@Import`注解上使用`HelloImportSelector`来把这2个组件导入到IOC容器中：

```java
@Configuration
@Import({HelloImportSelector.class})
public class AppConfig {
	
}
```

（3）验证测试

```java
ApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
String[] beanNames = context.getBeanDefinitionNames();
Arrays.stream(beanNames).forEach(System.out::println)
```



**使用`ImportBeanDefinitionRegistrar`接口配合`@Import`注解导入多个组件**

`ImportBeanDefinitionRegistrar`接口源码：

```java
public interface ImportBeanDefinitionRegistrar {
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);
}
```

接口中包含一个`registerBeanDefinitions()`方法，该方法包含两个入参：

 (a) AnnotationMetadata：可以通过它获取到类的注解信息；

 (b) BeanDefinitionRegistry：Bean定义注册器，包含了一些和Bean有关的方法：

BeanDefinitionRegistry 接口相关方法：

```java
public interface BeanDefinitionRegistry extends AliasRegistry {
    void registerBeanDefinition(String var1, BeanDefinition var2) throws BeanDefinitionStoreException;

    void removeBeanDefinition(String var1) throws NoSuchBeanDefinitionException;

    BeanDefinition getBeanDefinition(String var1) throws NoSuchBeanDefinitionException;

    boolean containsBeanDefinition(String var1);

    String[] getBeanDefinitionNames();

    int getBeanDefinitionCount();

    boolean isBeanNameInUse(String var1);
}
``` 
借助`BeanDefinitionRegistry`接口的`registerBeanDefinition()`方法来往IOC容器中注册Bean。该方法含有两个入参，第一个为需要注册的Bean名称（Id）,第二个参数为Bean的定义信息，可以使用他的实现类`RootBeanDefinition`来完成

自定义一个bean，这里还是用之前的`HelloWorldService`类。

自定义一个`ImportBeanDefinitionRegistrar`接口的实现类`MyHelloImportBeanDefinitionRegistrar`：

```java
/**
 * @author Xiaocai.Zhang
 */
public class MyHelloImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, 
    											BeanDefinitionRegistry registry) {
        final String beanName = "helloWorldService";
        boolean contain = registry.containsBeanDefinition(beanName);
        if (!contain) {
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(HelloWorldService.class);
            registry.registerBeanDefinition(beanName, rootBeanDefinition);
        }
    }
}
```
逻辑本身不复杂，先通过`BeanDefinitionRegistry`的`containsBeanDefinition()`方法判断IOC容器中是否包含了名称为`helloWorldService`的组件，如果没有，则手动通过`BeanDefinitionRegistry`的`registerBeanDefinition()`方法注册一个。

<hr/>


### 工厂接口注册

- 5️⃣  使用FactoryBean注册组件

可以通过实现`FactoryBean`接口来注册组件。

定义 bean：

```java
public class Hello{

}
```
创建FactoryBean的实现类`HelloFactoryBean`:

```java
public class HelloFactoryBean implements FactoryBean<Hello> {
    @Override
    public Hello getObject() {
        return new Hello();
    }

    @Override
    public Class<?> getObjectType() {
        return Hello.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```
`getObject()`返回需要注册的组件对象，`getObjectType()`返回需要注册的组件类型，`isSingleton()`指明该组件是否为单例。如果bean是多例的话，每次从容器中获取该组件都会调用其`getObject()`方法。

然后注册组件工厂：

```java
@Bean
public HelloFactoryBean helloFactoryBean() {
    return new HelloFactoryBean();
}
```

获取 bean：

```java
ApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
Object hello = context.getBean("&helloFactoryBean");
System.out.println(hello.getClass());
```

输出：
```
class com.xiaocai.beans.HelloFactoryBean
```

> 注意获取bean的时候前面有个`&`前缀。


## 2.2 Bean 命名

每个bean都有一个或多个标识符。这些标识符在承载bean的容器中必须是唯一的。一个bean通常只有一个标识符。但是，如果它需要一个以上的，额外的可以被视为别名。

在基于XML的配置元数据中，可以使用id属性或name属性来指定bean标识符。id属性允许您指定一个id。

如果要为bean引入其他别名，也可以在name属性中指定它们，用逗号（，）、分号（；）或空格分隔。

如果不显式地提供名称或id，容器将为该bean生成一个唯一的名称。但是，如果想通过使用ref元素或服务定位器样式的查找按名称引用该bean，则必须提供名称。



**命名规范**

惯例是在命名bean时使用标准Java约定作为实例字段名。也就是说，bean名称以一个小写字母开头，然后用驼峰大小写。这些名称的示例包括`accountManager`、`accountService`、`userDao`、`loginController`等。一致地命名bean使配置更易于阅读和理解。


### Bean 别名 <!-- {docsify-ignore} -->

XML中配置别名：

```xml
<alias name="fromName" alias="toName"/>
```

`@Bean` 也可实现别名使用。

## 2.4 Bean 实例化

bean定义本质上是创建一个或多个对象。

**实例化配置**

基于XML的配置元数据，则指定要在<bean/>元素的`class`属性中实例化的对象的类型。

- 构造函数实例化

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
```

- 使用静态工厂方法实例化:

下面的bean定义指定通过调用工厂方法创建bean。定义没有指定返回对象的类型（类），只指定包含工厂方法的类。在本例中，<b>`createInstance()`方法必须是静态方法</b>:

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

对应的类:

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```


- 使用实例工厂方法实例化

与通过静态工厂方法实例化类似，使用实例工厂方法的实例化从容器调用现有bean的<b>非静态方法</b>来创建新的bean


配置类

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```
对应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含多个工厂方法，如：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

对应定义多个工厂方法类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```
>工厂bean本身可以通过依赖注入（DI）进行管理和配置。




## 3 依赖

## 3.1  依赖注入

依赖注入，Dependency Injection，也就是常说的DI。

依赖注入（DI）是一个过程，通过该过程，对象只通过构造函数参数、工厂方法的参数或在对象实例构造或从工厂方法返回后在对象实例上设置的属性来定义其依赖项。然后容器在创建bean时注入这些依赖项。而这个过程基本上是bean本身的逆过程（因此称为控制反转），它通过直接构造类或服务定位器模式来控制依赖项的实例化或位置。


### 基于构造方法的依赖注入

基于构造函数的DI是通过容器调用一个构造函数来实现的，每个参数都代表一个依赖项。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
```

- ##### 根据构造方法类型的注入 <!-- {docsify-ignore} -->


```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```


- ##### 根据构造方法索引的注入 <!-- {docsify-ignore} -->

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

- ###### 根据构造方法参数名注入 <!-- {docsify-ignore} -->

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

### 基于Set方法的依赖注入

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

## 3.2 依赖配置详细

```xml
<bean>
	<property>
		<value/>
		<ref/>
		<list/>
	</property>		
</bean>
```	
常用标签：bean | ref | idref | list | set | map | props | value | null

略

## 3.3 depends-on

如果一个bean是另一个bean的依赖项，那一般会将一个bean被设置为另一个bean的属性。通常，在XML中配置bean的时候使用`<ref/>`元素来实现依赖注入。但是，有时bean之间的依赖关系不那么直接。例如，需要触发类中的静态初始值设定项，例如数据库驱动程序注册。`depends-on`属性可以显式地强制一个或多个bean在初始化使用此元素的bean之前初始化。

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

> depends-on属性可以指定初始化时间依赖项，并且仅在单例bean的情况下使用。相应的销毁时间也存在依赖关系。首先销毁与给定bean定义依赖关系的依赖bean，然后销毁给定bean本身。因此，`depends-on`还可以控制停机顺序

对应的注解是`@DependsOn`。

## 3.4 Lazy-init-bean

默认情况下，`ApplicationContext`实现在初始化过程中预先地创建和配置所有单例bean。但是一个延迟初始化的bean告诉IoC容器在第一次被请求时创建，而不是在启动时创建一个bean实例。

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

全部延迟创建：

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

对应的注解是`@Lazy`。

## 3.5 方法注入

现象描述：

在大多数应用程序场景中，容器中的大多数bean都是单例的。当一个单例bean需要与另一个单例的 bean 协作，或者一个非单例 bean需要与另一个非单例的 bean 协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。
但是当bean的生命周期不同时，就会出现一个问题。假设单例bean A需要使用非单例（原型）bean B，可能是在A的每个方法调用上。容器只创建一次单例bean A，因此只有一次机会设置属性。容器不能在每次需要bean B的时候都向bean A提供bean B的新实例。

解决办法：

放弃控制反转，使BeanA实现ApplicationContextAware接口，可以感知容器，在每次Bean A需要时对容器进行`getBean("B")`调用，请求（通常是新的）BeanB实例。

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
### @Lookup查找方法注入

官方术语：Lookup Method Injection

Lookup方法注入是容器重写容器托管bean上的方法并返回容器中另一个命名bean的查找结果的能力。查找通常针对一个原型bean。

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
如果方法是抽象的，则动态生成的子类实现该方法。否则，动态生成的子类重写原始类中定义的具体方法。

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

也可以使用注解模式：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```
习惯上，可以依赖目标bean根据lookup方法声明的返回类型进行解析

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

### 任意方法替换

一种不如查找方法注入有用的方法注入形式是，可以用另一种方法实现替换托管bean中的任意方法。

对于基于XML的配置元数据，对于已部署的bean，可以使用替换的方法元素将现有的方法实现替换为另一个方法实现。

比如下面这个类中，我们想重写`computeValue()` 方法：

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```
实现`org.springframework.beans.factory.support.MethodReplacer`接口提供了新的方法定义:

```java
/**
 * method reimplement to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

配置原始类并指定方法重写:

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

对应的注解`@Replace`

## 4 Bean 作用域

**作用域类型**

- Singleton
- Prototype
- Request
- session
- applicaiton
- websocket

**作用域使用**

XML使用

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

注解使用`@Scope`

```java
@Configuration
public class GlobalConfig {

    @Scope("singleton")
    public HelloWorldService helloWorldService(){
        return new HelloWorldService();
    }
}
```

**自定义作用域**

1、实现`org.springframework.beans.factory.config.Scope`接口。

`Scope`接口有四种方法可以从作用域中获取对象，从作用域中删除对象，然后销毁它们。

```java
public interface Scope {
    Object get(String var1, ObjectFactory<?> var2);

    @Nullable
    Object remove(String var1);

    void registerDestructionCallback(String var1, Runnable var2);

    @Nullable
    Object resolveContextualObject(String var1);

    @Nullable
    String getConversationId();
}
```

2、让Spring 容器发现，或注册到容器。

```java
void registerScope(String scopeName, Scope scope);
```



## 5 定制 Bean

Spring框架提供了许多接口，可以用来定制bean的性质

- 生命周期回调。
- `ApplicationContextAware`和`BeanNameAware`。
- 其他感知接口。


### 生命周期回调

　　为了与容器对bean生命周期的管理进行交互，可以实现Spring `InitializingBean`和`DisposableBean`接口。容器为`InitializingBean`接口调用`afterPropertiesSet()`，为`DisposableBean`接口调用`destroy()`，让bean在初始化和销毁bean时可以自定义执行某些操作。

**初始化回调方式：**

 1️⃣  使用<bean/> 标签的属性`init-method`

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

2️⃣ 实现`InitializingBean`接口:

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```
3️⃣ 使用`@PostConstruct`注解：

```java
/**
 * @author Xiaocai.Zhang
 */
public class MyHelloBean {


    @PostConstruct
    public void loadCache(){
        
    }
}
```


**销毁方法回调：**

 1️⃣  使用<bean/> 标签的属性`destroy-method`

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="destroy"/>
```

```java
public class ExampleBean {

    public void destroy() {
        // do some destroy work
    }
}
```

2️⃣  实现`DisposableBean`接口:

```java
/**
 * @author Xiaocai.Zhang
 */
@Component
public class MyHelloBean implements DisposableBean {
    /**
     * 自定义销毁方法
     * @throws Exception
     */
    @Override
    public void destroy() throws Exception {
        
    }
}
```

3️⃣  使用`@PreDestroy`注解：

```java
/**
 * @author Xiaocai.Zhang
 */
public class MyHelloBean {


    @PreDestroy
    public void clearCache(){
        
    }
}
```


### 组合生命周期使用

1、实现`InitializingBean`和`DisposableBean`回调接口。

2、自定义`init()`方法 和 `destroy()`方法。

3、`@PostConstruct`和`@PreDestroy`注解。


**优先级：**

针对同一个bean配置的多个生命周期机制，不同的初始化方法调用顺序先后如下：

（1）优先调用被`@PostConstruct`注解修饰的方法。
（2）`DisposableBean`回调接口定义的`afterPropertiesSet()`方法。
（3）自定义配置的`init()`方法。

销毁方法同理:

（1）优先调用被`@PreDestroy`注解修饰的方法。
（2）`DisposableBean`回调接口定义的`destroy()`方法。
（3）自定义配置的`destroy()`方法。


### 生命周期启停

生命周期接口为具有自己生命周期要求的任何对象定义基本方法：

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```
任何Spring管理的对象都可以实现该生命周期接口。然后，当`ApplicationContext`本身接收到启动和停止信号（例如，对于运行时的停止/重新启动方案），容器将这些调用作用到该上下文中定义的所有生命周期实现，通过委托给LifecycleProcessor来实现：

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

`LifecycleProcessor`本身就是对生命周期接口的扩展。它还添加了两个其他方法来响应正在刷新和关闭的上下文。

> 注意，`org.springframework.context.Lifecycle`接口是用于显式启动和停止通知的普通协议，并不意味着在上下文刷新时自动启动。为了对特定bean的自动启动（包括启动阶段）进行细粒度的控制，可以考虑改为实现`org.springframework.context.SmartLifecycle`接口。

　　启动和关闭调用的顺序很重要，如果两个对象之间存在"依赖"关系，则依赖方在其依赖项之后启动，在其依赖项之前停止。有时，直接依赖关系是不明确，于是有了`SmartLifecycle`接口。

```java
public interface Phased {

    int getPhase();
}

public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

　　`Phased`接口定义了一个返回int的g`etPhase()`方法，得到一个阶段的数值，姑且叫阶段值。

　　启动时，阶段值最小的对象优先启动。停止时，阶段值最大的对象优先启动。

　　所以一个实现`SmartLifecycle`接口并返回其`getPhase()`方法的整数最小值会是第一个启动，最后一个停止。反过来看，阶段值`Integer.MAX_VALUE`值表该对象应在最后启动并最先停止（可能是因为它依赖于其他正在运行的进程）。

　　在考虑阶段值时，其他没有实现`SmartLifecycle`接口的"正常"生命周期对象的默认阶段值是0。因此，任何负阶段值都表示对象应该在这些标准组件之前启动，也会在它们之后停止。

　　`LifecycleProcessor`接口的默认实现`DefaultLifecycleProcessor`类，该类将等待每个阶段中的一组对象调用该回调。默认生命周期处理器`DefaultLifecycleProcessor`，每个阶段刷新时间是30s。

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

`LifecycleProcessor`接口定义了用于正在刷新和正在关闭上下文的回调方法，正在关闭方法`onClose()`执行的过程就和显式的调用`stop()`方法一样。

另一方面，"refresh"回调启用了`smartlifecycle`组件的其他特性，当上下文刷新时，这个`onRefresh()`回调就会被调用。这时，默认生命周期处理器`DefaultLifecycleProcessor`将检查每个`SmartLifecycle对象的isAutoStartup()`方法返回的布尔值。如果是`true`，该对象开始启动，不需要显示调用`start()`方法。

### ApplicationContextAware

`ApplicationContextAware` 和 `BeanNameAware` 。

#### ApplicationContextAware  <!-- {docsify-ignore} -->

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```
当`ApplicationContext`创建一个实现`org.springframework.context.ApplicationContextAware`接口的实例时，该实例提供了对该应用程序上下文的引用。通过改种方式
bean可以通过`ApplicationContext`接口或通过将引用强制转换为该接口的已知子类，以编程方式操作创建它们的`ApplicationContext`，比如`ConfigurableApplicationContext`，它公开了附加功能。

我们在使用的时候一般可以通过构造方法或setter方法为构造函数参数或setter方法参数提供ApplicationContext类型的依赖关系。

我们常用的`@Autowired`也是获取对`ApplicationContext`的引用的另一种方法。


自定义工具类，从`ApplicationContext`获取需要的Bean:


```java
@Component
public class SpringContextUtils  implements ApplicationContextAware {

    private static ApplicationContext applicationContext = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (SpringContextUtils.applicationContext == null) {
            SpringContextUtils.applicationContext = applicationContext;
        }
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }


    public static Object getBean(String name) {
        return getApplicationContext().getBean(name);
    }


    public static <T> T getBean(Class<T> clazz) {
        return getApplicationContext().getBean(clazz);
    }


    public static <T> T getBean(String name, Class<T> clazz) {
        return getApplicationContext().getBean(name, clazz);
    }
}
```

### BeanNameAware

当`ApplicationContext`创建实现`org.springframework.beans.factory.BeanNameAware`接口的类时，则该类提供了对该类关联对象定义的名称定义的引用。

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```
在填充普通bean属性之后，但在初始化回调（如initializegbean、afterPropertiesSet或自定义init方法）之前调用回调。


## 容器扩展Bean

一般可以通过集成已有接口的实现来扩展spring `IoC容器`。


### 实现`BeanPostProcessor`接口

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

### 实现`BeanFactoryPostProcessor`接口

BeanFactoryPostProcessor：Bean工厂后置处理器。

这个接口的语义与`BeanPostProcessor`相似，但有一个主要区别：`BeanFactoryPostProcessor`可以操作bean配置元数据。`Spring IoC容器`允许`BeanFactoryPostProcessor`读取配置元数据，并在容器实例化(除`BeanFactoryPostProcessor`实例之外）任何bean之前对bean进行更改。

配置多个`BeanFactoryPostProcessor`实例时，可以通过设置`order`属性来控制运行先后顺序。

`BeanFactoryPostProcessor`在`ApplicationContext`中声明时自动运行，以便将更改应用到定义容器的配置元数据。


### 使用FactoryBean接口

实现`org.springframework.beans.factory.FactoryBean`接口，使实现类本身变身成工厂。

`FactoryBean`接口是`Spring IoC容器`实例化逻辑的一个可插拔点。

如果需要Java代码方式初始化（不是XML），那么可以创建自己的`FactoryBean`，在该类中编写复杂的初始化，然后将自定义的FactoryBean插入容器中。

> 注意获取bean的时候前面有个`&`前缀。

参考示例 [FactoryBean 示例](pages/后端/Spring/Spring-01?id=工厂接口注册)


## 环境抽象

### Profiles

`@Profile` 的使用。

如：使用`@Profile("development")`标记加载开发环境Bean，使用`@Profile("production")`加载生产环境相关Bean。

激活profile:

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

命令激活：

```bash
java -jar  -Dspring.profiles.active="profile1,profile2"
```

### 属性加载优先级

对于常见的`StandardServletEnvironment`，完整的层次结构如下，优先级由高到低：

（1）ServletConfig参数（例如，对于DispatcherServlet上下文）

（2）ServletContext参数（如web.xml文件上下文参数项）

（3）JNDI环境变量(java:comp/env/条目）

（4）JVM参数属性（-D个命令行参数）

（5）JVM系统环境（操作系统环境变量）


### @PropertySource

```properties
testbean.name=myTestBea
```
使用

```java
@Configuration
@PropertySource("classpath:/resource/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

## ApplicationContext其他功能

### 容器事件

内置事件：

- ContextRefreshedEvent：上下文刷新事件

- ContextStartedEvent：上下文启动事件

- ContextStoppedEvent：上下文停止事件

- ContextClosedEvent：上下文关闭事件

- RequestHandledEvent：请求处理事件

- ServletRequestHandledEvent：servlet请求处理事件。

自定义应用事件：

自定义需要实现`ApplicationEvent`接口：

```java
/**
 * DemoEvent(自定义事件)
 * @Description(描述) : spring的event为bean和bean之间的消息通信提供了支持，可以用一个bean监听当前的bean所发送的事件
 * @author Xiaocai.Zhang
 */
public class DemoEvent extends ApplicationEvent {

    private static final long serialVersionUID = 1L;

    private String msg;

    public void sysLog() {
        System.out.println(msg);
    }

    public DemoEvent(Object source,String msg) {
        super(source);
        this.setMsg(msg);
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```
实现事件监听器:
```java
/**
 * 事件监听器
 * 实现ApplicationListener并制定监听的事件类型
 * @author Zongyuan.Zhang
 */
@Component
public class DemoListener implements ApplicationListener<DemoEvent> {
    /***
     * 对消息进行接受处理
     */
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMsg();
        System.out.println("DemoListener接收到了DemoPublisher发布的消息："+msg);
    }
}
```
不过，从Spring 4.2 开始可以使用注解`@EventListener`实现事件监听：

```java
/**
 * 事件监听器
 * 实现ApplicationListener并制定监听的事件类型
 * @author Zongyuan.Zhang
 */
@Component
public class DemoListener  {
    /***
     * 对消息进行接受处理
     */
    @EventListener({DemoEvent.class})
    public void myEventListener(DemoEvent event) {
        String msg = event.getMsg();
        System.out.println("DemoListener接收到了DemoPublisher发布的消息："+msg);
    }
}
```
事件发布：
```java
/** 事件发布类
 * @author Zongyuan.Zhang
 */
@Component
public class DemoPublisher {
    @Autowired
    ApplicationContext context;

    public void published() {
        DemoEvent event = new DemoEvent(this, "发布成功！");
        System.out.println("发部event："+event);
        context.publishEvent(event);
    }
}
```

`@EventListener`注解其他特性：

（1）可以监听内置的事件:

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```
（2）多个监听事件现有顺序使用`@Order(22)`注解即可，数字小优先。

（3）支持复杂的`SpEL`表达式

```java
@EventListener(condition = "#blEvent.msg == 'myEvent'")
public void processBlockedListEvent(BlockedListEvent blockedListEvent) {
    // notify appropriate parties via notificationAddress...
}
```
`SpEL`表达式支持的元数据有：

（a）Event，如`#root.event`，`event`

（b）参数数组，如 `#root.args`，`args`，`args[0]`

（c）参数名称，如 `#blEvent`，`#a0`，`#p0`




### 异步监听


```java
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent is processed in a separate thread
}
```

### 普通事件

比如实体创建事件：`EntityCreatedEvent<T>`，通过泛型定义结构：

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```
大部分事件都是这种结构，当然也可以实现`ResolvableTypeProvider`接口操作超出运行时环境的提供的功能。

如：
```java
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

## Web 应用上下文注册

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```


## 2.Spring AOP


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