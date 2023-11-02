---
layout: default
title: Java Concurrent 
parent: Java
has_children: true
nav_order: 50
---


Here are about Java Concurrent .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }


---

# 并发编程

## 一、JUC 是什么

在 Java 5.0 提供了 `java.util.concurrent`(简称JUC)包，该包中增加了在并发编程中常用的工具类，用于定义类似于线程的自定义子系统，包括线程池、异步IO 和轻量级任务框架;还提供了设计用于多线程上下文中
的 Collection 实现，多线程交互、锁等相关API。

可以称呼为并发编程，也可以叫多线程了，只能说不是很规范。

```
java.util.concurrent  // 通常在并发编程中很实用的程序类。
java.util.concurrent.atomic  // 一个小型工具包，支持单个变量上的无锁线程安全编程。
java.util.concurrent.locks // 接口和类提供了一个框架，用于锁定和等待与内置同步和监视器不同的条件。
```



- `Atomic` : `AtomicInteger`，`AtomicLong`  原子操作类。

- `Locks` : `Lock`， `Condition`， `ReadWriteLock`  可重入读写锁。

- `Collections` : `Queue`， `ConcurrentMap` 并发集合。

- `Executer` : `Future`， `Callable`， `Executor` 线程执行池，异步Future等。

- `Tools` : `CountDownLatch`， `CyclicBarrier`， `Semaphore` 减数器，等待器，信号量。


[Java 8 API](https://www.matools.com/api/java8)



## 二、进程与线程

**进程(process)**

含义：程序的一次执行过程，或是正在运行的一个程序。

>进程作为资源分配的单位，系统在运行时会为每个进程分配不同的内存区域

**线程(thread)**

含义：进程可进一步细化为线程，是一个程序内部的一条执行路径。

>线程作为CPU调度和执行的单位，每个线程拥独立的运行栈和程序计数器(pc)，线程切换的开销小。

进程可以细化为多个线程。

>在JVM运行时数据区，每个线程，拥有自己独立的：栈、程序计数器。多个线程，共享同一个进程中的结构：方法区、堆。


## 三、并行与并发

**单核CPU与多核CPU**

单核CPU，其实是一种假的多线程，因为在一个时间单元内，也只能执行一个线程的任务。例如：虽然有多车道，但是收费站只有一个工作人员在收费，只有收了费才能通过，那么CPU就好比收费人员。如果某个人不想交钱，那么收费人员可以把他“挂起”（晾着他，等他想通了，准备好了钱，再去收费。）但是因为CPU时间单元特别短，因此感觉不出来。

如果是多核CPU，才能更好的发挥多线程的效率。（现在的服务器都是大多是多核的）

一个Java应用程序java.exe，其实至少三个线程：main()主线程，gc()垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。

**并行与并发**

**并行：**多个CPU同时执行多个任务。比如：多个人同时做不同的事。

**并发：**一个CPU(采用时间片)同时执行多个任务。比如：秒杀、多个人做同一件事。 单核CPU模拟出来多条线程来执行，天下武功，唯快不破，快速交替。

Java 可以直接获取CPU核数：

```java
System.out.println(Runtime.getRuntime().availableProcessors());
```

>由此可见，并发编程与多线程不可分离。


## 四、JAVA程序线程

#### 常见线程 <!-- {docsify-ignore} -->

一个Java程序一般至少有2个线程。

一个是启动线程，即`main()`方法的启动入口;
另一个是开发者创建的线程。

但是JVM启动后，实际上至少有一个非守护线程，即`main线程`。

- Finalizer 线程：GC守护线程。
- RMI 线程：Java 自带的远程方法调用线程。
- Monitor 线程：是一个守护线程，负责监听一些操作，在main线程组中。


#### 守护线程 <!-- {docsify-ignore} -->

设置守护线程：`public final void setDaemon(boolean on)`

将某个线程设置成守护线程，传递参数为 `true` 即可。

当主线程死亡后，守护线程会跟着死亡。一般守护线程可以帮助做一些辅助性的东西，如"心跳检测"、"缓存清除"等等。

