---
layout: default
title: Singleton
parent: Design Patterns
has_children: false
nav_order: 11
---

Here are java   Singleton Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是单例模式？

在 设计层面 保证某个类在应用运行生命周期范围内只能存在一个对象实例。


## 单例模式 8 种写法

### 饿汉式 2种

### 1、饿汉式之静态常量；   

生产级别可以使用。

```java
package com.xiaocai.designs.single;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 1 -饿汉式 1 ---静态常量
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo01 {

    //TODO 1、构造器私有化
    private SingletonDemo01(){

    }
    //TODO 2、创建静态常量实例
    private  final  static SingletonDemo01 instance = new SingletonDemo01();

    //TODO 3、提供公有静态方法返回实例对象
    public static SingletonDemo01 getInstance(){
        return instance ;
    }
    
    public static void main(String[] args) {
        System.out.println("饿汉式1--静态常量----可用");
        SingletonDemo01 instance01 = SingletonDemo01.getInstance();
        SingletonDemo01 instance02 = SingletonDemo01.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());

    }
}


```
### 2、饿汉式之静态代码块

【生产级可以使用】

```java
package com.xiaocai.designs.single;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 2 - 饿汉式 2 ---静态代码块写法
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo02 {

    //TODO 1、构造器私有化
    private SingletonDemo02(){

    }
    //TODO 2、创建静态变量实例
    private   static SingletonDemo02 instance ;

    //TODO 3、静态代码块中进行实例化
    static {
        instance = new SingletonDemo02();
    }
    
    //TODO 4、提供公有静态方法返回实例对象
    public static SingletonDemo02 getInstance(){

        return instance ;
    }

    public static void main(String[] args) {
        System.out.println("饿汉式2--静态代码块写法----可用");
        SingletonDemo02 instance01 = SingletonDemo02.getInstance();
        SingletonDemo02 instance02 = SingletonDemo02.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());
    }
}
```
### 懒汉式 2种

### 3、懒汉式之延迟加载【非线程安全】  
SingletonDemo03
起到了LazyLoading 延迟加载的效果，只能在单线程下加载。

多线程下，线程不安全。

实际开发中不能使用。

```java
package com.xiaocai.designs.single;

import java.util.concurrent.*;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 3 - 懒汉式 1 --- 线程不安全写法
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo03 {

    //TODO 1、构造器私有化
    private SingletonDemo03(){

    }
    //TODO 2、创建静态变量实例
    private   static SingletonDemo03 instance ;


    //TODO 3、提供公有静态方法返回实例对象 ----使用时才创建对象
    public static SingletonDemo03 getInstance(){
        if(instance == null){

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            instance = new SingletonDemo03();
        }
        return instance ;
    }



    public static void main(String[] args) {
        System.out.println("懒汉式1--线程不安全式写法----");
        /*
        SingletonDemo03 instance01 = SingletonDemo03.getInstance();
        SingletonDemo03 instance02 = SingletonDemo03.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());
*/
        /*
        Callable<SingletonDemo03> callable = new Callable<SingletonDemo03>() {
            @Override
            public SingletonDemo03 call() throws Exception {
                return SingletonDemo03.getInstance();
            }
        };
        */

        Callable<SingletonDemo03> callable = () -> SingletonDemo03.getInstance();

        ExecutorService service1 = Executors.newFixedThreadPool(2);

        Future<SingletonDemo03> future1 = service1.submit(callable);
        Future<SingletonDemo03> future2 = service1.submit(callable);

        try {
            SingletonDemo03 singletonDemo03_t1 = future1.get();
            SingletonDemo03 singletonDemo03_t2 = future2.get();

            System.out.println(singletonDemo03_t1.equals(singletonDemo03_t2));
            System.out.println(singletonDemo03_t1.hashCode());
            System.out.println(singletonDemo03_t2.hashCode());


        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }finally {
            service1.shutdown();
        }
    }
}
```

### 4、懒汉式之----【同步方法】
SingletonDemo04  ---- 同步代码块在多线程下也是线程不安全的。
```java
package com.xiaocai.designs.single;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 4 - 懒汉式 2 --- 线程安全写法 -- synchronized 同步方法
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo04 {

    //TODO 1、构造器私有化
    private SingletonDemo04(){

    }
    //TODO 2、创建静态变量实例
    private   static SingletonDemo04 instance ;


    //TODO 3、提供公有静态方法返回实例对象 ----使用时才创建对象,并且使用  synchronized 同步方法，解决线程安全问题
    public static synchronized SingletonDemo04 getInstance(){
        if(instance == null){
            instance = new SingletonDemo04();
        }
        return instance ;
    }

    public static void main(String[] args) {
        System.out.println("懒汉式2--线程安全式写法----");
        SingletonDemo04 instance01 = SingletonDemo04.getInstance();
        SingletonDemo04 instance02 = SingletonDemo04.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());
    }
}
```
synchronized 同步方法 造成了效率低  SingletonDemo05
```java
package com.xiaocai.designs.single;

import java.util.concurrent.*;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 5 - 懒汉式 3  --- 线程不安全写法 -- synchronized 同步代码块
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo05 {

    //TODO 1、构造器私有化
    private SingletonDemo05(){

    }
    //TODO 2、创建静态变量实例
    private  static  SingletonDemo05 instance ;


    //TODO 3、提供公有静态方法返回实例对象 ----  synchronized
    public static  SingletonDemo05 getInstance(){

        if(instance == null){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized(SingletonDemo05.class){

                instance = new SingletonDemo05();
            }
        }
        return instance ;
    }



    public static void main(String[] args) {
        System.out.println("懒汉式3--synchronized 同步代码块，反而线程不安全----");
        /*
        SingletonDemo05 instance01 = SingletonDemo05.getInstance();
        SingletonDemo05 instance02 = SingletonDemo05.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());
*/

        Callable<SingletonDemo05> callable = () -> SingletonDemo05.getInstance();

        ExecutorService service1 = Executors.newFixedThreadPool(2);

        Future<SingletonDemo05> future1 = service1.submit(callable);
        Future<SingletonDemo05> future2 = service1.submit(callable);

        try {
            SingletonDemo05 singletonDemo05_t1 = future1.get();
            SingletonDemo05 singletonDemo05_t2 = future2.get();

            System.out.println(singletonDemo05_t1.equals(singletonDemo05_t2));
            System.out.println(singletonDemo05_t1.hashCode());
            System.out.println(singletonDemo05_t2.hashCode());


        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }finally {
            service1.shutdown();
        }
    }
}
```

### 5、懒汉式之【双重检测锁】 
SingletonDemo06
解决延迟加载问题，解决线程安全问题，但是如果使用了反射，依旧可以破坏单例
```java
package com.xiaocai.designs.single;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 6 - 懒汉式 4 --- 双重检测锁----
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo06 {

    //TODO 1、构造器私有化
    private SingletonDemo06(){

    }
    //TODO 2、创建静态变量实例 使用 volatile 关键字，保证单例的原子操作
    private  static  volatile SingletonDemo06 instance ;


    //TODO 3、提供公有静态方法返回实例对象 ----  synchronized
    public static SingletonDemo06 getInstance(){

        if(instance == null){

            synchronized(SingletonDemo06.class){
                if (instance == null){
                    instance = new SingletonDemo06();
                }
            }
        }
        return instance ;
    }



    public static void main(String[] args) {
        System.out.println("懒汉式4--双重检测锁----正常测试---");
        SingletonDemo06 instance01 = SingletonDemo06.getInstance();
        SingletonDemo06 instance02 = SingletonDemo06.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());


        try {
            System.out.println("懒汉式4--双重检测锁----反射测试，破坏单例模式---");
            Constructor<SingletonDemo06> constructor = SingletonDemo06.class.getDeclaredConstructor(null);
            constructor.setAccessible(true);
            SingletonDemo06 singletonDemo06A = constructor.newInstance();
            SingletonDemo06 singletonDemo06B = constructor.newInstance();

            System.out.println("使用反射实例化 " + singletonDemo06A.equals(singletonDemo06B));
            System.out.println("singletonDemo06A " + singletonDemo06A.hashCode());
            System.out.println("使用反射实例化singletonDemo02 " + singletonDemo06B.hashCode());

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

### 6、懒汉式之【三重检测锁】  
SingletonDemo07
可以防止反射破坏单例【推荐使用】

```java
package com.xiaocai.designs.single;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 7 - 懒汉式 5 --- 双重检测锁的升级版---三重检测锁
 * @author: 张小菜
 * @date: 2020/10/18 21:10
 * @version: v1.0
 */
public class SingletonDemo07 {

    //  添加单例存活标记
    private static boolean xiaocai = false ;

    //TODO 1、构造器私有化，并使用
    private SingletonDemo07(){
        synchronized(SingletonDemo07.class){
            if (xiaocai == false){

                xiaocai = true ;
            }else{
                throw new RuntimeException("不要使用反射破坏单例");
            }
        }
    }

    //TODO 2、创建静态变量实例 使用 volatile 关键字，保证单例的原子操作
    private  static  volatile SingletonDemo07 instance ;


    //TODO 3、提供公有静态方法返回实例对象 ----  synchronized
    public static SingletonDemo07 getInstance(){

        if(instance == null){

            synchronized(SingletonDemo07.class){
                if (instance == null){
                    instance = new SingletonDemo07();
                    xiaocai = true ;
                }
            }
        }
        return instance ;
    }



    public static void main(String[] args) {
        System.out.println("懒汉式5--三重检测锁----正常测试（不优先使用）---");
        SingletonDemo07 instance01 = SingletonDemo07.getInstance();
        SingletonDemo07 instance02 = SingletonDemo07.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());


        try {
            System.out.println("懒汉式5--三重检测锁----反射测试，破坏单例模式---");
            // 将文件反编译，找到 单例存活标记
            Field xiaocai = SingletonDemo07.class.getDeclaredField("xiaocai");
            xiaocai.setAccessible(true);

            Constructor<SingletonDemo07> constructor = SingletonDemo07.class.getDeclaredConstructor(null);
            constructor.setAccessible(true);
            SingletonDemo07 singletonDemo06A = constructor.newInstance();
            SingletonDemo07 singletonDemo06B = constructor.newInstance();

            xiaocai.set(xiaocai, false);

            System.out.println("使用反射实例化 " + singletonDemo06A.equals(singletonDemo06B));
            System.out.println("singletonDemo06A " + singletonDemo06A.hashCode());
            System.out.println("使用反射实例化singletonDemo02 " + singletonDemo06B.hashCode());

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
}
```

### 7、静态内部类式

利用类装载机制保证初始化只有一个县城；

类的静态属性只会在第一次加载的时候初始化，利用 JVM保证线程的安全性。在类进行初始化时，别的线程无法进入。

避免线程不安全，实现了延迟加载，效率高。【推荐使用】

```java
package com.xiaocai.designs.single;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 8 - 使用静态内部类模式 ---利用静态内部类的装载机制，使 JVM 来保证安全
 * @author: 张小菜
 * @date: 2020/10/18 22:13
 * @version: v1.0
 */
public class SingletonDemo08 {

    // 构造器私有
    private SingletonDemo08(){

    }

    // 公有方法获取
    public static  synchronized  SingletonDemo08 getInstance(){

        return SingleInstance.SINGLETON_INSTANCE;
    }

    private static  class SingleInstance{

        private static  final SingletonDemo08 SINGLETON_INSTANCE = new SingletonDemo08();
    }


    public static void main(String[] args) {
        System.out.println("单例模式 8 ---静态内部类----正常测试（推荐使用）---");
        SingletonDemo08 instance01 = SingletonDemo08.getInstance();
        SingletonDemo08 instance02 = SingletonDemo08.getInstance();
        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());
    }
}
```
### 8、枚举式单例
从根源上保证单例【推荐使用】
避免多线程问题，防止反序列化重新创建对象。
```java
package com.xiaocai.designs.single;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 单例模式 9 - 使用枚举 --推荐使用
 * @author: 张小菜
 * @date: 2020/10/18 22:13
 * @version: v1.0
 */
public enum SingletonDemo09Enum {

    INSTANCE ;

    public void sayOK(){
        System.out.println("hello world ");
    }


    public static void main(String[] args) {

        System.out.println("单例模式 9 --- 枚举类----正常测试（推荐使用）---");
        SingletonDemo09Enum instance01 = SingletonDemo09Enum.INSTANCE;
        SingletonDemo09Enum instance02 = SingletonDemo09Enum.INSTANCE;

        System.out.println(instance01.equals(instance02));
        System.out.println(instance01.hashCode());
        System.out.println(instance01.hashCode());
    }
}
```
