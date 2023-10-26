---
layout: default
title: SpringBoot Custom Annotation
parent: SpringBoot
nav_order: 180
---


## 基于SpringBoot的自定义注解

## 一、Java注解

Java 定义的注解分三类。

（1）普通注解。
（2）元注解。
（3）自定义注解。

### 1、普通注解

普通注解在Java.lang 中有3个：

- `@Override`：检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。

- `@Deprecated`：标记过时方法。若某类或某方法加上该注解之后，表示此方法或类不再建议使用，调用时也会出现删除线，但并不代表不能用，只是说，不推荐使用，因为还有更好的方法可以调用。

- `@SuppressWarnings`：指示编译器去忽略注解中声明的警告。

### 2、元注解

元注解就是用来修饰注解的注解。通俗的理解为为了开发人员方便开发自定义注解的工具型注解，简单说的就类似JDK工具包作用。

在java.lang.annotation中有4个元注解：

- `@Retention`：用来说明注解的存活时间，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。保存方式有3种：

    - RetentionPolicy.SOURCE - 标记的注释仅保留在源级别中，并由编译器忽略

    - RetentionPolicy.CLASS - 标记的注释在编译时由编译器保留，但Java虚拟机（JVM）会忽略

    - RetentionPolicy.RUNTIME - 标记的注释由JVM保留，因此运行时环境可以使用它

- `@Documented`：标记这些注解是否包含在用户文档中，注释表明，无论何时使用指定的注释，都应使用Javadoc工具记录这些元素。（默认情况下，注释不包含在Javadoc中）

- `@Target`：标记这个注解应该是哪种 Java 对象范围，所修饰的对象范围有：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数），支持的元素类型有：

    - ElementType.TYPE 可以应用于类的任何元素

    - ElementType.FIELD 可以应用于字段或属性

    - ElementType.METHOD 可以应用于方法级注释

    - ElementType.PARAMETER 可以应用于方法的参数

    - ElementType.CONSTRUCTOR 可以应用于构造函数

    - ElementType.LOCAL_VARIABLE 可以应用于局部变量

    - ElementType.ANNOTATION_TYPE 可以应用于注释类型

    - ElementType.PACKAGE 可以应用于包声明

    - ElementType.TYPE_PARAMETER 可以应用于类型参数

    - lementType.TYPE_USER 可以应用任何类型名称

- `@Inherited`：此注释仅适用于类声明。 类继承关系中，子类会继承父类使用的注解中被`@Inherited`修饰的注解；接口继承关系中，子接口不会继承父接口中的任何注解，不管父接口中使用的注解有没有被`@Inherited`修饰；类实现接口关系中，类实现接口时不会继承任何接口中定义的注解。

```java
    @Inherited
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Person {
    }
     
    @Person
    public class Student {
    }
     
    public class ZhangSan extends Student{
    }
```    
注解`@Person`被`@Inherited`修饰，`Student`被 `@Person`修饰，ZhangSan继承Student（ZhangSan上又无其他注解），那么ZhangSan就会拥有Person这个注解。

### 3、新增注解

Java7之后增加的3个注解：

- `@SafeVarargs`： Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。

- `@FunctionalInterface`：Java 8 开始支持，标识一个匿名函数或函数式接口。

- `@Repeatable`：Java 8 开始支持，标识某注解可以在同一个声明上使用多次。


### 3、自定义注解

开发者自己开发的注解。


## 二、写个自定义注解

### 1、需求

还记得上个月面试中遇到一个问题，怎么让一个接口限制在1分钟内访问200次。

我记得我的回答是用缓存，配合步长计数器。整体回答自己感觉都答的不是很好。

所以干嘛不写个注解来解决这个问题呢？这样减少代码与接口耦合，又可以一次编写多处可用，使用前面提到过的BaseController 的通用处理方法，在需要的时候调用父类，或者借由钩子方法进行前置和后置操作，再配合缓存一样可以实现这个功能。

先用注解的方式实现，后面再用这个BaseController的方式实现。


### 2、分析与设计

要自定义注解，同时要可以灵活的配置每个接口在自己的时间窗口期内限制指定的次数，不同的接口限定方式肯定是不一样的，同时也可以给默认值。

（1）针对单个接口
比如默认情况下一个接口，只能在1分钟内访问100次，首先需要记录时间，其次需要有个计数器，进行计数，如果在1分钟内超过100次，并在第101次访问的时候，就限制访问，最好给个友好提示。每个接口第一次访问触发限制条件，脱离限制的时间窗口，接口自己的设定值限制要自动失效，访问的数据也要失效或归零。这里可以采用配置的方式注入，不过既然是写注解，最好是利用注解特性，采用注解属性赋值。


（2）针对多个接口
针对多个不同接口执行不同的限制规则，接口A可以限制在30秒内访问10次，接口B在1分钟内访问60次，接口之间不要相互干扰。这个可以利用注解的属性，进行分别赋值。


（3）注解通用性

注解的通用性，意味着限制逻辑不能和业务代码写在一起。

**思路一：**使用拦截器的特性对Restful 接口进行拦截验证。

**思路二：**使用AOP的方式进行拦截。

既然可以使用AOP的方式零入侵是不是可以考虑做成一个组件呢？

**思路三：**完成之后可以将注解开发成独立的starter组件。


## 三、自定义注解实现


### 1、准备工作

这里将自定义注解取个名字，因为要用不同的思路实现，注解的名字就不写一样的了。

准备工作，因为想让数据自动失效，同时又减少耦合依赖，就不使用第三方缓存中间件，搞个属于自己的缓存管理器，这里借鉴已有的缓存管理器，并做一些细节调整：

#### 1.1、环境搭建

创建springboot工程，因为要测试Restful接口，需要引入`spring-boot-starter-web`，然后准备个controller用来测试:


```java
package com.xiaocai.base.demo.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Xiaocai.Zhang
 */
@Slf4j
@RestController
@RequestMapping("/api")
public class ApiController extends BaseController {

}
```
这里的BaseController 非必须，可以没有。



#### 1.2、缓存管理

```java
package com.xiaocai.base.demo.cache;

import lombok.extern.slf4j.Slf4j;

import java.lang.ref.SoftReference;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 学习 Java缓存管理器
 * 1, 按照JSR-107规范实现
 * 2，使用 ConcurrentHashMap 结构存储数据，并且 value 值使用 SoftReference 包装。
 * 3，缓存定义超时时间，开启后台线程定时扫描清理超时的数据
 * 本类非原创，参考其他博客文章修改而成
 * @author Xiaocai.Zhang
 */
@Slf4j
public class MyCacheManager {

    /**
     * 定义多长时间描扫一次过期的数据，默认为5秒
     */
    private static final long CLEAN_UP_PERIOD_IN_SEC = 5;
 
    /**
     * 使用Map数据结构存储缓存数据，key是字符串类型，value是软引用类型，当内存确实不够用的时候，jvm会回收对应的value引用对象
     */
    private final ConcurrentHashMap<String, SoftReference<Cache>> cacheMap = new ConcurrentHashMap<>();

    public MyCacheManager(){
        this(CLEAN_UP_PERIOD_IN_SEC);
    }

    public MyCacheManager(long period){
        long realPeriod = period > CLEAN_UP_PERIOD_IN_SEC ? period : CLEAN_UP_PERIOD_IN_SEC ;
        // 定义一个定时任务，清理已过期的缓存数据
        Thread cleanerThread = new Thread(() -> {
            while(!Thread.currentThread().isInterrupted()){
                try {
                    Thread.sleep(realPeriod * 1000); // 每5秒清理缓存一次
                    for(Map.Entry<String, SoftReference<Cache>> entry : cacheMap.entrySet()){
                        Cache cache = entry.getValue().get();
                        if(cache.isExpiry()){
                            // 注释 log.warn("the entry of  key {}  is expiry ！" , entry.getKey());
                            System.out.println(realPeriod + " sec. is over, the entry of  key "+entry.getKey()+"  is expiry ！"  );
                            cacheMap.remove(entry.getKey());
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    Thread.currentThread().interrupt();
                }
            }
        });
        cleanerThread.setDaemon(true);
        cleanerThread.start();
    }

    /**
     * 清除所有缓存
     */
    public void clear(){
        cacheMap.clear();
    }

    /**
     * 返回缓存管理器里面缓存的元素的个数
     * @return
     */
    public long size(){
        if(cacheMap.isEmpty()){
            return 0L;
        }
        Set<Map.Entry<String, SoftReference<Cache>>> entrySet = cacheMap.entrySet();
        long count = 0L;
        for(Map.Entry<String, SoftReference<Cache>> entry : entrySet){
            SoftReference<Cache> value = entry.getValue();
            Cache cache = value.get();
            if(!cache.isExpiry()){
                count++;
            }
        }
        return count;
    }

    /**
     * 增加或更新一个缓存数据
     * @param key
     * @param value
     * @param expiryTime -1表示不超期，单位为毫秒
     * @return 返回null或者旧的缓存值
     */
    public Object put(String key, Object value, long expiryTime){
        if(key == null){
            return null;
        }
        if(value == null){
            return extractObjVal(cacheMap.remove(key));
        }
        if(expiryTime <= 0){
            expiryTime = -1;
        }else{
            expiryTime = System.currentTimeMillis() + expiryTime;
        }
        return extractObjVal(cacheMap.put(key, new SoftReference<>(new Cache(value, expiryTime))));
    }

    /**
     * 删除缓存
     * @param key
     * @return
     */
    public Object remove(String key){
        if(key == null){
            return null;
        }
        return extractObjVal(cacheMap.remove(key));
    }

    /**
     * 从缓存中获取值，没有找到返回null，找到但已超期也返回null。
     * @param key
     * @return
     */
    public Object get(String key){
        if(key == null){
            return null;
        }
        SoftReference<Cache> softReference = cacheMap.get(key);
        if(softReference == null){
            return null;
        }else{
            Cache cache = softReference.get();
            if(cache.isExpiry()){
                // 访问的时候被动删除超期的元素
                cacheMap.remove(key);
                return null;
            }else{
                return cache.getValue();
            }
        }
    }

    /**
     * 抽取cache对象里面的缓存值
     * @param cacheSoftReference
     * @return
     */
    private Object extractObjVal(SoftReference<Cache> cacheSoftReference){
        return cacheSoftReference == null ? null : cacheSoftReference.get().getValue();
    }

    private static class Cache{
        private Object value; // 缓存数据对象
        private long expiryTime; // 超时时间，以时间戳的形式代表，-1表示不超期

        public Cache(Object value, long expiryTime){
            this.value = value;
            this.expiryTime = expiryTime;
        }

        /**
         * 判断缓存是否已经超期
         * @return true - 已超期，false - 未超期
         */
        public boolean isExpiry(){
            if(expiryTime == -1){
                return false;
            }
            return System.currentTimeMillis() > expiryTime;
        }

        public Object getValue() {
            return value;
        }

        public void setValue(Object value) {
            this.value = value;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        MyCacheManager cacheManager = new MyCacheManager();

        cacheManager.put("1", "jerry", 3 * 1000);
        System.out.println("get 1 emp :" + cacheManager.get("1"));
        Thread.sleep(5000);
        System.out.println("get 1 emp :" + cacheManager.get("1"));
        System.out.println("-------");

        cacheManager.put("2", "tom", 6 * 1000);

        System.out.println("get 2 emp :" + cacheManager.get("2"));
        Thread.sleep(5000);
        System.out.println("get 2 emp :" + cacheManager.get("2"));
        System.out.println("-------");


        cacheManager.put("3", "jack", 8 * 1000);
        System.out.println("get 3 emp :" + cacheManager.get("3"));
        Thread.sleep(5000);
        System.out.println("get 3 emp :" + cacheManager.get("3"));
    }
}
```
这个缓存管理器可以设置缓存数据的有效时间，并且时间到了之后就会自动过期。

#### 1.3、统一返回格式

包装使用统一的返回格式Json

```java
package com.xiaocai.base.demo.common.message;

import lombok.Getter;
import lombok.Setter;

/**
 * @author Xiaocai.Zhang
 */
@Setter
@Getter
public class CommonResult<T> {

    private Boolean success;

    private String msg;

    private T data;

    public CommonResult(boolean success) {
        this.success = success;
    }
    public CommonResult(boolean success, String msg) {
        this.success = success;
        this.msg = msg ;
    }
    public CommonResult(T data) {
        this.success = Boolean.TRUE;
        this.data = data;
    }
    public CommonResult(boolean success, String msg, T data) {
        this.success = success;
        this.msg = msg ;
        this.data = data;
    }
    public static <T> CommonResult<T> ok(){
        return new CommonResult(Boolean.TRUE);
    }
    public static <T> CommonResult<T> ok(String msg){
        return new CommonResult(Boolean.TRUE, msg);
    }

    public static <T> CommonResult<T> ok(String msg, T data){
        return new CommonResult(Boolean.TRUE, msg, data);
    }

    public static <T> CommonResult<T> ok(T data){
        return new CommonResult(data);
    }

    public static <T> CommonResult<T> fail(){
        return new CommonResult(Boolean.FALSE);
    }
    public static <T> CommonResult<T> fail(String msg){
        return new CommonResult(Boolean.FALSE, msg);
    }
}
```

#### 1.4、测试工具

浏览器或者PostMan 均可。


### 2、配合拦截器使用

#### 2.1、先定义注解：

```java 
package com.xiaocai.base.demo.annotation;

import java.lang.annotation.*;

/**
 * @author Xiaocai.Zhang
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface ApiChecked {
    /**
     * 用来区分api的名字
     * @return String
     */
    String name() default "default";
    /**
     * 表示当前api开启限制标记
     * @return boolean
     */
    boolean limit() default true;
    /**
     * 表示当前api限制的时间窗口期，单位是秒
     * @return boolean
     */
    long second() default 60;
    /**
     * 表示当前api限制的时间窗口期内允许访问次数
     * @return boolean
     */
    long totalCount() default 100;
    /**
     * 表示当前api限制之后是否开启延迟等待
     * @return boolean
     */
    boolean waitEnable() default false;
    /**
     * 表示当前api限制之后开启延迟等待的限制时间，单位是秒
     * @return long
     */
    long limitWaiting() default 3;
}

```

#### 2.2、利用拦截器解析注解

这里是经过多个修改调整之后最终代码：

```java
package com.xiaocai.base.demo.intercepter;

import com.alibaba.fastjson.JSONObject;
import com.xiaocai.base.demo.annotation.ApiChecked;
import com.xiaocai.base.demo.cache.MyCacheManager;
import com.xiaocai.base.demo.common.message.CommonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.util.HashMap;

/**
 * @author Xiaocai.Zhang
 */
@Slf4j
@Component
public class ApiCheckedInterceptor implements HandlerInterceptor {

    private final String DEFAULT_NAME = "default";
    private final String TIME_KEY = "limitTime";
    private  final String TOTAL_KEY = "limitTotal";
    private final String WAIT_KEY = "limitWait";
    /**
     * 自定义缓存器，按API分别缓存
     */
    private static final HashMap<String, MyCacheManager> cache = new HashMap<>();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info(" into interceptor ---");
        // 进行注解解析并作相关限制验证
        if (!apiChecked(handler, response)){
            return false;
        }
        return true;
    }

    /**
     * 拦截器方式使用注解，解析注解参数
     * @param handler
     * @param response
     * @return
     * @throws Exception
     */
    private boolean apiChecked(Object handler, HttpServletResponse response) throws Exception{
        // 反射获取方法上的LoginRequred注解
        HandlerMethod handlerMethod = (HandlerMethod)handler;

        ApiChecked interfaceChecked = handlerMethod.getMethod().getAnnotation(ApiChecked.class);
        if(interfaceChecked == null){
            log.info("preHandle interfaceChecked is null ");
            return true;
        }
        // 注解验证无限制直接放行
        if (interfaceChecked.limit()){
            String nameKey = interfaceChecked.name();
            nameKey = DEFAULT_NAME.equals(nameKey) ?  handlerMethod.getMethod().getName() : nameKey ;
            long total = interfaceChecked.totalCount();
            long sec = interfaceChecked.second();
            long wait = interfaceChecked.limitWaiting();
            log.info("total = {}, sec = {}, wait = {}" , total, sec, wait);
            MyCacheManager cacheManager = cache.get(nameKey) ;
            if ( null == cacheManager){
                cacheManager = new MyCacheManager(sec);
                cache.put(nameKey, cacheManager);
                log.info("init cacheManager for api [ {} ], and the period is {} ", nameKey, sec);
            }
            if ( null != cacheManager.get(WAIT_KEY)){
                responseMsg(response, "the [ "+nameKey+" ] API limit, please visit it later wait "+wait+" sec");
                return false ;
            }
            long expiryTime = sec * 1000 ;
            if (null == cacheManager.get(TIME_KEY)){
                LocalDateTime newLimit = LocalDateTime.now().plus(sec, ChronoUnit.SECONDS);
                cacheManager.put(TIME_KEY, newLimit, expiryTime);
                cacheManager.put(TOTAL_KEY, total, expiryTime);
                log.debug("--put cacheManager -- key = {} , value = {}", TIME_KEY, newLimit);
                log.debug("--put cacheManager -- key = {} , value = {}", TOTAL_KEY, total);
            }

            LocalDateTime limitTime = (LocalDateTime) cacheManager.get(TIME_KEY);
            log.debug("limit limitTime -- key = {} , value = {}", TIME_KEY, limitTime);
            long newTotal = (long)cacheManager.get(TOTAL_KEY);
            log.debug("limit newTotal -- key = {} , value = {}", TOTAL_KEY, newTotal);

            long waitTime = wait * 1000 ;
            if (limitTime.isBefore(LocalDateTime.now()) ){
                if (interfaceChecked.waitEnable()) cacheManager.put(WAIT_KEY, wait, waitTime);
                responseMsg(response, "the [ "+nameKey+" ] API limit, The time window has reached the limit time of expiry time");
                return false ;
            }
            if ( newTotal <= 0){
                if (interfaceChecked.waitEnable()) cacheManager.put(WAIT_KEY, wait, waitTime);
                responseMsg(response, "the [ "+nameKey+" ] API limit, The time window has reached the limit number of times ");
                return false ;
            }
            newTotal = newTotal -1 ;
            cacheManager.put(TOTAL_KEY, newTotal, expiryTime);
            log.debug("--put new total value -- key : {} , value {}", TOTAL_KEY, newTotal);
        }
        return true ;
    }


    /**
     * 拦截限制返回
     * @param response
     * @param msg
     * @throws IOException
     */
    private void responseMsg(HttpServletResponse response, String msg) throws IOException {
        response.setCharacterEncoding( "UTF-8");
        response.setContentType( "application/json; charset=utf-8");
        PrintWriter out = null ;
        try{
            CommonResult<Object> result = CommonResult.fail(msg);
            String res = JSONObject.toJSONString(result);
            out = response.getWriter();
            out.append(res);
        } catch (Exception e){
            e.printStackTrace();
            response.sendError(500);
        }
    }
}
```

#### 2.3、注册拦截器

```java
package com.xiaocai.base.demo.config;

import com.xiaocai.base.demo.intercepter.ApiCheckedInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @author Xiaocai.Zhang
 */
@Configuration
public class InterceptorTrainConfig implements WebMvcConfigurer {

    /**
     * 拦截器注册类 InterceptorRegistry 加入自己的拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new ApiCheckedInterceptor()).addPathPatterns("/api/**");
    }
}

```

#### 2.4、添加注解使用

```JAVA
package com.xiaocai.base.demo.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Xiaocai.Zhang
 */
@Slf4j
@RestController
@RequestMapping("/api")
public class ApiController  {



    @RequestMapping("/check")
    @ApiChecked(second = 10, totalCount = 3)
    public CommonResult checkUser() {
        log.info("--@InterfaceChecked  normal  do business ...");
        return CommonResult.ok("@InterfaceChecked is normal, please watch console log.");
    }

    @RequestMapping("/check2")
    @ApiChecked(second = 15, totalCount = 5, limit = false)
    public CommonResult checkUser2() {
        log.info("--@InterfaceChecked limit false  business ...");
        return CommonResult.ok("@InterfaceChecked limit is false, please watch console log.");
    }
}    
```
比如 `checkUser()` 添加了 `@ApiChecked` 注解，并且设置了注解参数 second = 10, totalCount = 3，表示10秒内，只能访问3次，10秒内超过3次就会被拦截器拦截。


#### 2.5、注解测试

启动工程访问验证。我的端口号是8805。

连续点击四次 `http://localhost:8805/api/check`

前三次可以正常返回

```json
{
    "success":true,
    "msg":"@InterfaceChecked is normal, please watch console log.",
    "data":null
}
```
第四次会出现限制信息：

```json
{
	"success":false,
    "msg":"the [ checkUser ] API limit, The time window has reached the limit number of times "
}
```
等待几秒之后，又可以继续访问了。

限制的时间窗口期是 second 秒，等待的时间是从第一次访问时开始触发计时计数，在 second 秒范围内的访问会进行计数，超过访问次数会被限制。 在 second 秒范围外，即超过 second 秒后将重新访问会重新触发计时计数的限制条件。



也可以查看对应的控制台日志打印信息。


### 3、配合AOP使用

需要引入aop的包

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

#### 3.1、先定义注解：

为了和拦截器方式区分，我就换了一个注解名字，并且里面加了其他标记，都是调整完善之后的：

```java
package com.xiaocai.base.demo.annotation;

import java.lang.annotation.*;

/**
 * @author Xiaocai.Zhang
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface LimitChecked {
    /**
     * 为当前限制方法取个名字，默认是方法名
     * @return
     */
    String name() default "";

    /**
     * 方法限制标记，默认开启限制
     * @return
     */
    boolean limit() default true;

    /**
     * 方法限制一个时间窗口期，单位是秒
     * @return
     */
    long second() default 0;
    /**
     * 方法限制一个时间窗口期内限定次数
     * @return
     */
    long totalCount() default 0;
    /**
     * 是否开启限定后的等待延迟
     * @return
     */
    boolean waitEnable() default false;
    /**
     * 启限定后的等待延迟时间，仅在 waitEnable = true 的时候生效
     * @return
     */
    long limitWaiting() default 5;

    /**
     * 是否调用成功才计算次数
     * @return
     */
    boolean onlySuccessCount() default false;
}

```

#### 3.2、APO处理

```java
package com.xiaocai.base.demo.aspect;

import com.alibaba.fastjson.JSONObject;
import com.xiaocai.base.demo.annotation.LimitChecked;
import com.xiaocai.base.demo.cache.MyCacheManager;
import com.xiaocai.base.demo.common.message.CommonResult;
import com.xiaocai.base.demo.exception.BizCommonException;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.util.HashMap;


/**
 * @author Xiaocai.Zhang
 */
@Slf4j
@Aspect
@Component
public class LimitCheckedAspect {

    private final String DEFAULT_NAME = "";
    private final String TIME_KEY = "limitTime";
    private final String TOTAL_KEY = "limitTotal";
    private final String WAIT_KEY = "limitWait";
    private static final HashMap<String, MyCacheManager> cache = new HashMap<>();

    @Order(1) // Order 代表优先级，数字越小优先级越高
    @Pointcut("@annotation(com.xiaocai.base.demo.annotation.LimitChecked)")
    public void checkedPoint(){};

    @Before(value = "checkedPoint()")
    public void checkBefore(){

        log.info("--checked before ");
    }

    @After(value = "checkedPoint()")
    public void checkAfter(){
        log.info("--checked After ");
    }

    @Around(value = "checkedPoint() && @annotation(limitChecked)")
    public CommonResult checkAround(ProceedingJoinPoint joinPoint, LimitChecked limitChecked){
        CommonResult commonResult = null;
        log.info("--checked before Around   ---");
        // 获取方法名称
        String methodName = joinPoint.getSignature().getName();
        // 获取入参
        Object[] param = joinPoint.getArgs();
        log.debug("methodName {},  param {}", methodName, param);

        // 注解拦截处理
        String nameKey = limitChecked.name();
        nameKey = DEFAULT_NAME.equals(nameKey) ?  methodName : nameKey ;
        long sec = limitChecked.second();
        // 注解验证无限制直接放行
        if (limitChecked.limit()){
            MyCacheManager cacheManager = cache.get(nameKey) ;
            long total = limitChecked.totalCount();
            long wait = limitChecked.limitWaiting();
            log.info("@LimitChecked params list: nameKey = {} , total = {},  sec = {},  wait = {}", nameKey, total, sec, wait);

            if ( null == cacheManager){
                cacheManager = new MyCacheManager(sec);
                cache.put(nameKey, cacheManager);
                log.info("Init cacheManager for api [ {} ], and the period is {} ", nameKey, sec);
            }
            if (limitChecked.waitEnable() && null != cache.get(nameKey).get(WAIT_KEY)){
                log.info("get wait value : {}", cache.get(nameKey).get(WAIT_KEY));
                return CommonResult.fail("the [ "+nameKey+" ] API limit, please call it after wait "+wait+" sec") ;
            }
            long expiryTime = sec * 1000 ;
            if (null == cache.get(nameKey).get(TIME_KEY)){
                LocalDateTime newLimit = LocalDateTime.now().plus(sec, ChronoUnit.SECONDS);
                cache.get(nameKey).put(TIME_KEY, newLimit, expiryTime);
                cache.get(nameKey).put(TOTAL_KEY, total, expiryTime);
                log.debug("put value -- key : {} , value {} ", TIME_KEY, newLimit);
                log.debug("put value -- key : {} , value {} ", TOTAL_KEY, total);
            }

            LocalDateTime limitTime = (LocalDateTime) cache.get(nameKey).get(TIME_KEY);
            long newTotal = (long)cache.get(nameKey).get(TOTAL_KEY);
            log.debug("limit newTotal -- key : {} , value {} ", TOTAL_KEY, newTotal);

            long waitTime = wait * 1000 ;
            // 其实该种情况理论上不会出现，因为 limitTime 在 sec 之后就会过期，如果限制时间过期，其实不会比拿出比现在小的时间
            if (limitTime.isBefore(LocalDateTime.now()) ){
                Object nothing = limitChecked.waitEnable() ? cache.get(nameKey).put(WAIT_KEY, wait, waitTime) : null;
                return CommonResult.fail("the [ "+nameKey+" ] API limit, The time window has reached the limit time of expiry time") ;
            }
            if ( newTotal <= 0 ){
                if (limitChecked.waitEnable()){
                    cache.get(nameKey).put(WAIT_KEY, wait, waitTime);
                    log.info("wait {} ", wait);
                }
                return CommonResult.fail("the [ "+nameKey+" ] API limit, The time window has reached the limit number of times ["+total+"]") ;
            }
            if (!limitChecked.onlySuccessCount()){
                newTotal = newTotal - 1 ;
                cache.get(nameKey).put(TOTAL_KEY, newTotal, expiryTime);
                log.debug("--put new total value -- key : {} , value {}", TOTAL_KEY, newTotal);
            }
        }
        // 继续执行请求方法
        try {
            commonResult = (CommonResult) joinPoint.proceed();
        }catch (BizCommonException e) {
            return CommonResult.fail("api occurred exception ...");
        } catch (Throwable throwable) {
            throwable.printStackTrace();

        }

        log.info("--checked after Around   ---");

        // 注解限制生效前提下，开启接口调用成功才计数
        if (limitChecked.limit() && limitChecked.onlySuccessCount()){
            long newTotal = (long)cache.get(nameKey).get(TOTAL_KEY);
            newTotal = newTotal - 1 ;
            cache.get(nameKey).put(TOTAL_KEY, newTotal, sec * 1000);
            log.debug("--put new total value -- key : {} , value {}", TOTAL_KEY, newTotal);
        }

        return  commonResult == null ? CommonResult.fail("API Error "): commonResult ;
    }

    @AfterReturning(value = "checkedPoint()", returning = "result")
    public void checkAfterReturn(CommonResult result){
        log.info("--checked after return   ---");
        log.info("result : {}", JSONObject.toJSONString(result));
    }


    @AfterThrowing(value = "checkedPoint()", throwing="throwable")
    public CommonResult afterThrow(Throwable throwable){
        log.info("--checked  afterThrow ");
        return CommonResult.fail("AfterThrowing api occurred exception ...");
    }

}
```

#### 3.3、添加注解使用

```java
package com.xiaocai.base.demo.controller;

import com.xiaocai.base.demo.annotation.ApiChecked;
import com.xiaocai.base.demo.annotation.LimitChecked;
import com.xiaocai.base.demo.annotation.MyLog;
import com.xiaocai.base.demo.common.base.BaseController;
import com.xiaocai.base.demo.common.base.BaseService;
import com.xiaocai.base.demo.common.message.CommonResult;
import com.xiaocai.base.demo.exception.BizCommonException;
import com.xiaocai.base.demo.exception.ErrorCode;
import com.xiaocai.base.demo.service.base.UserBsService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Xiaocai.Zhang
 */
@Slf4j
@RestController
@RequestMapping("/api")
public class ApiController {

    @RequestMapping("/check")
    @ApiChecked(second = 10, totalCount = 3)
    public CommonResult checkUser() {
        log.info("--@InterfaceChecked  normal  do business ...");
        return CommonResult.ok("@InterfaceChecked is normal, please watch console log.");
    }
    @RequestMapping("/check2")
    @ApiChecked(second = 15, totalCount = 5, limit = false)
    public CommonResult checkUser2() {
        log.info("--@InterfaceChecked limit false  business ...");
        return CommonResult.ok("@InterfaceChecked limit is false, please watch console log.");
    }

    @RequestMapping(value = {"/check3", "limitNormal"})
    @LimitChecked(second = 5, totalCount = 2)
    public CommonResult check3() {
        log.info("--limit normal do business ...");
        return CommonResult.ok("@LimitChecked limit is normal (true), please watch console log.");
    }

    @RequestMapping(value = {"/check4", "/limitFalse"})
    @LimitChecked(limit = false, second = 8, totalCount = 3)
    public CommonResult check4() {
        log.info("--limit false 该接口不进行限制 do business ...");
        return CommonResult.ok("@LimitChecked limit is false, please watch console log.");
    }

    @RequestMapping(value = {"/check5", "/limitWait"})
    @LimitChecked(second = 6, totalCount = 2, waitEnable = true, limitWaiting = 10)
    public CommonResult check5() {
        log.info("--limit normal  waitEnable do business ...");
        return CommonResult.ok("@LimitChecked waitEnable is  true, please watch console log.");
    }


    int temp = 0 ;
    @RequestMapping(value = {"/check6", "/limitSuccess"})
    @LimitChecked(second = 30, totalCount = 3, onlySuccessCount = true)
    public CommonResult check6() {
        temp++;
        log.info("--onlySuccessCount do business ...");
        if ( 2 == temp ){
            throw new BizCommonException(ErrorCode.SYSTEM_SERVICE_ERROR_CODE);
        }
        return CommonResult.ok("@LimitChecked onlySuccessCount is true , please watch console log.");
    }

    @RequestMapping("/checkLog")
    @MyLog
    public CommonResult checkLog() {
        log.info("--checkLog do business ...");
        return CommonResult.ok("@MyLog to checkLog, please watch console log.");
    }
}
```

#### 2.5、注解测试

启动工程访问验证。我的端口号是8805。

连续点击四次 `http://localhost:8805/api/check3`

原理同上。

前三次可以正常返回

```json
{
    "success":true,
    "msg":"@LimitChecked limit is normal (true), please watch console log.",
    "data":null
}
```
第四次会出现限制信息：

```json
{
    "success":false,
    "msg":"the [ check3 ] API limit, The time window has reached the limit number of times [2]",
    "data":null
}
```
对应的日志：

也可以查看对应的控制台日志打印信息。

其他几个API可以修改注解属性自行测试。

示例工程地址：[demo-project-base](https://github.com/small-rose/demo-project-base)

### 4、starter组件

到此，拦截器模式和AOP模式全部完成。如果在进行调整可以将注解开发成独立的starter组件，实现和springboot项目组件化。在自己需要的项目中使用。

关于starter的内容也不少，单独放一篇。

自定义starter组件开发。




