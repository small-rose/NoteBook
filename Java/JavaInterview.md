---
layout: default
title: Java Interview
parent: Java
nav_order: 11999
---


## 2020年10月Java面试题

### 2020年10月23日整理

1、说说枚举和单例

2、怎么理解死锁

3、对jvm的理解

4、SpringCloud 技术体系

5、HashMap在1.8里和1.7有什么不同

6、线程池怎么创建，有哪几个参数，分别是什么作用

7、main 方法里启动了两个线程，怎么让两个线程都执行完，再继续执行main线程

8、web容器相关，tomcat启动有哪几个端口，分别是什么作用

9、@Transaction有没有可能会失效？如果事务方法是private还会生效吗

10、redis 设计一下库存怎么做

11、redis分布式锁怎么实现

12、redis 基本数据结构有哪些

13、maven 的打包是什么命令？package和install 作用是什么，怎么部署包到远程仓库？

14、git常用命令有哪些？怎么合并分支

15、spring cloud config怎么用

16、nginx 反向代理怎么配置

17、nginx访问路径文件夹没有权限怎么办？修改组的权限命令是什么



### 2020年10月28日整理


1、sql优化除了SQL语句和索引还可以做哪些？

2、现在有个业务，需要同时满足几种不同的条件才能走下去，并且几种条件会经常变化，你怎么设计？

3、web.xml 可以配置哪些东西

4、web应用启动时组件的加载先后顺序

5、jvm调优使用的参数有哪些，分别是什么意思

6、说下springboot 自动化配置加载

7、说说Spring bean的生命周期

8、innodb有哪些特点

9、Mysql 索引原理，外键索引

10、数据库建表需要注意哪些事情？

11、Spring cloud的Feign 的做什么的，怎么实现的

12、Ribbon 负载均衡策略有哪些？轮询算法的逻辑是怎么样？

13、线程池的几个参数和作用，怎么判断是不是核心线程

14、synchronized 锁膨胀过程，底层原理

15、类加载过程是怎样的？

16、双亲委派模型是怎样，怎么破坏的

17、什么是粗化锁



### 2020年10月30日整理

1、spring mvc 请求到响应的过程

2、HashMap在1.8里和1.7有什么不同

3、字符串比较

4、Mysql 索引原理及分类，BD2 索引有哪些？

5、HashTable 和 ConcurrentHashMap 区别

6、单节点应用，按钮点击后端只能查询一次，后端怎么处理



### 2020年11月

1、execute和submit的区别

> execute和submit都属于线程池的方法，execute只能提交Runnable类型的任务，而submit既能提交Runnable类型任务也能提交Callable类型任务。
>
> execute会直接抛出任务执行时的异常，submit会吃掉异常，可通过Future的get方法将任务执行时的异常重新抛出。
>
> execute所属顶层接口是Executor,submit所属顶层接口是`ExecutorService`，实现类`ThreadPoolExecutor`重写了execute方法,抽象类`AbstractExecutorService`重写了submit方法。

2、JVM运行时内存布局，各个区大小

3、常见的OOM，SOF以及预防措施

4、CAP理论-BASE理论

5、设计原则和设计模式，设计模式的使用

6、分布式事务

7、Redis使用 MQ使用

8、手写单例

9、为什么HashMap的加载因子是0.75

10、为什么volatile 不能保证原子性？

11、ReentrantLock 加锁原理（AQS+CAS）

12、聚簇索引和非聚簇索引

13、你的优势是什么？你最擅长什么？

