---
layout: default
title: Multi-Thread
parent: Java Concurrent
grand_parent: Java
nav_order: 10
---


# 多线程

线程创建方式：

- 继承 `Thread` 线程类。

- 实现 `Runnable` 接口。

- 实现 `Callable` 接口。

### 1、继承 `Thread` 线程类方式

>继承Thread类，重写Thread类的`run()`方法并将此线程执行的操作声明在`run()`方法中，调用的时候，创建Thread类的子类的对象，通过此对象调用`start()`方法启动线程。

基本类：

```java
package com.xiaocai.juc.threads;

public class ThreadStyle01 extends Thread{

    public ThreadStyle01(String name){
        super(name);
    }

    @Override
    public void run() {
        System.out.println("这里写线程任务实现");
    }
}
```
基本类：

```java
package com.xiaocai.juc.threads;
/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/12/24 15:58
 * @version: v1.0
 */
public class ThreadStyle01Test {

    public static void main(String[] args) {
        Thread thread = new ThreadStyle01("thread-style-01");
        thread.start();
    }
}
```

### 2、实现 `Runnable` 接口方式

>创建一个实现`Runnable`接口的实现类，实现类去实现Runnable中的抽象`run()`方法。调用的时候，创建实现类的对象并将此对象作为参数传递到Thread类的构造器中来创建Thread类的对象，通过Thread类的对象调用`start()`方法启动线程。


（1）外部式

```java
package com.xiaocai.juc.threads;

public class ThreadStyle02 implements Runnable{
    private String name ;

    public ThreadStyle02(String name){
        this.name = name ;
    }

    @Override
    public void run() {
        System.out.println("这里写线程任务实现");
    }
}
```

调用类：

```java
package com.xiaocai.juc.threads;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/12/24 16:02
 * @version: v1.0
 */
public class ThreadStyle02Test {

    public static void main(String[] args) {
        ThreadStyle02 run01 = new ThreadStyle02( "thread-style-02");
        Thread thread = new Thread(run01);
        thread.start();
    }
}
```

（2）匿名内部类

```java
package com.xiaocai.juc.threads;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/12/24 16:15
 * @version: v1.0
 */
public class ThreadStyle03 {

    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("这里写线程任务实现");
            }
        });
        thread.start();
    }
}
```

（3）Lamda表达式

```java
package com.xiaocai.juc.threads;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/12/24 16:18
 * @version: v1.0
 */
public class ThreadStyle04 {

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
                System.out.println("这里写线程任务实现");
        });
        thread.start();
    }
}
```

### 3、实现 `Callable` 接口

>`Callable<V>` 提供返回数据，根据需要返回不同类型V。

```java
package com.xiaocai.juc.threads;

import java.util.concurrent.Callable;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/12/24 16:30
 * @version: v1.0
 */
public class ThreadStyle05 implements Callable<String> {
    
    @Override
    public String call() throws Exception {
        System.out.println("这里写线程任务实现");
        return "hello world";
    }
}
```

基本调用：

```java
package com.xiaocai.juc.threads;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/12/24 16:30
 * @version: v1.0
 */
public class ThreadStyle05Test {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Callable<String> tc = new ThreadStyle05();
        FutureTask<String> task = new FutureTask<String>(tc);
        new Thread(task).start();
        String result = task.get();
        System.out.println(result);
    }
}
```	

输出结果：

```
这里写线程任务实现
hello world
```

# Thread 和 Runnable 关系

关系：

```java
public class Thread implements Runnable{

}
```
相同点：两种方式都需要重写`run()`方法，将线程要执行的逻辑声明在`run()`中。

>实际开发中一般优先选择**实现`Runnable`接口的方式**。因为**实现的方式没有类的单继承性的局限性，实现的方式更适合来处理多个线程有共享数据的情况。** 


（1）在多线程的设计之中，使用了代理设计模式的结构，用户自定义的线程主体只是负责项目核心功能的实现，而所有的辅助实现全部交由（代理类）`Thread`类来处理。

（2）在进行Thread启动多线程的时候调用的是`start()`方法，而后找到的是`run()`方法，但通过Thread类的构造方法传递了一个Runnable接口对象的时候，那么该接口对象将被Thread类中的target属性所保存，`start()`方法执行的时候会调用Thread类中的`run()`方法。而这个`run()`方法去调用实现了Runnable接口的那个类所重写过run()方法，进而执行相应的逻辑。

（3）多线程开发的本质实际上是在于多个线程可以进行同一资源的抢占，那么Thread主要描述的是线程，而资源的描述是通过Runnable完成的。如下图所示：


# Thread 源码

**Runnable 接口源码**

```java
@FunctionalInterface
public interface Runnable {

	public abstract void run();
}
```


**Thread类的部分源码**

```java
public class Thread implements Runnable {
	// some code ...

	// Thread类中的target属性，就是我们要执行的目标线程
	private Runnable target; 
 
	
	//构造方法的代码从上往下顺序执行
	public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    
	private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize) {
        init(g, target, name, stackSize, null, true);
    }
    
	private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
    }

    /* 由于这里是通过继承Thread类来实现的线程，那么target这个东西就是Null。
     * 但是因为你继承了Runnable接口并且重写了run()，所以最终还是调用子类的run()
     * */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
 
}
```
# Thread 构造方法

 - 创建线程对象Thread，默认有一个线程名，以Thread-开头，从0开始计数，如"Thread-0、Thread-1、Thread-2 …"。

 - 如果没有传递Runnable或者没有覆写Thread的run方法，该Thread不会调用任何方法。

 - 如果传递Runnable接口的实例或者覆写run方法，则会执行该方法的逻辑单元（逻辑代码）。

 - 如果构造线程对象时，未传入ThreadGroup，Thread会默认获取父线程的ThreadGroup作为该线程的ThreadGroup，此时子线程和父线程会在同一个ThreadGroup中。

 - stackSize可以提高线程栈的深度，放更多栈帧，但是会减少能创建的线程数目。

 - stackSize默认是0，如果是0，代表着被忽略，该参数会被JNI函数调用，但是注意某些平台可能会失效，可以通过"-Xss10m"设置。


# Thread 线程启动

`Thread` 类 `start()` 方法源码：

```java
    public synchronized void start() {
   
	    if (threadStatus != 0)
	        throw new IllegalThreadStateException();//①

	   
	    group.add(this);

	    boolean started = false;
	    try {
	        start0();
	        started = true;
	    } finally {
	        try {
	            if (!started) {
	                group.threadStartFailed(this);
	            }
	        } catch (Throwable ignore) {
	            /* do nothing. If start0 threw a Throwable then
	              it will be passed up the call stack */
	        }
	    }
	}

	private native void start0();
```

当多次调用start()，会抛出`IllegalThreadStateException` 异常。也就是每一个线程类的对象只允许启动一次，如果重复启动则就抛出此异常。


![](../Assets/images/juc/thread-start.png) 



# Thread 核心逻辑


1、如果`Thread`类的构造方法传递了一个`Runnable`接口对象

①那么该接口对象将被`Thread`类中的`target`属性所保存。

②在`start()`方法执行的时候会调用`Thread`类中的`run()`方法。因为`target`不为`null`， `target.run()`就去调用实现`Runnable`接口的子类重写的`run()`。

2、如果`Thread`类的构造方传没传`Runnable`接口对象

①`Thread`类中的`target`属性保存的就是`null`。

②在`start()`方法执行的时候会调用`Thread`类中的`run()`方法。因为`target`为`null`，只能去调用继承`Thread`的子类所重写的`run()`。


# ThreadLocal 

`Thread`类中还有一个比较经典的`ThreadLocal`类。

>`ThreadLocal`提供了线程内存储变量的能力，这些变量不同之处在于每一个线程读取的变量是对应的互相独立的。通过get和set方法就可以得到当前线程对应的值。


`Thread`类是一个线程内部的存储类，可以在指定线程内存储数据，数据存储以后，只有指定线程可以得到存储数据。其实就是我们常说的线程数据隔离。


`ThreadLocal`的静态内部类`ThreadLocalMap`为每个`Thread`都维护了一个数组`table`，`ThreadLocal`确定了一个数组下标，而这个下标就是`value`存储的对应位置。


# 线程生命周期

Java 源码中枚举的生命周期

![](../Assets/images/juc/thread-life.png) 


线程状态变化图

![](../Assets/images/juc/thread-status.png) 

> 注意：图中 wait到 runnable状态的转换中，join实际上是Thread类的方法，但这里写成了Object。

1、线程创建之后它将处于 `NEW`（新建） 状态，调用 `start()` 方法后开始运行，线程这时候处于 `READY`（可运行） 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 `RUNNING`（运行） 状态。

2、操作系统隐藏 Java 虚拟机（JVM）中的 `READY` 和 `RUNNING` 状态，它只能看到 `RUNNABLE` 状态，所以 Java 系统一般将这两个状态统称为 `RUNNABLE`（运行中） 状态 。

3、调用线程的`sleep()`方法，会进入`Blocked`状态。 `sleep()`结束之后，`Blocked`状态首先回到的是Runnable状态中的`Ready`（也就是可运行状态，但并未运行）。只有拿到了cpu的时间片才会进入`Runnable`中的`Running`状态。


获取当前存活的线程数：`public int activeCount()`
获取当前线程组的线程的集合：`public int enumerate(Thread[] list)`
 