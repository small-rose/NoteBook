---
layout: default
title: Java jvm 
parent: Java
has_children: false
nav_order: 100
---


# Java JVM About
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


# jvm 命令与参数

## 1、基础命令 jps

查看java程的pid及基本信息

```
jps -l
```

查看进程pid及main方法参数

```
jps -m
```


查看pid及JVM参数

```
jps -v
```


执行结果：
```
[root@small-rose]$ jps  -m -l
15332 sun.tools.jps.Jps -m -l
16628 com.small.rose.demo.DbDemoApplication
12748 org.jetbrains.idea.maven.server.RemoteMavenServer36
```


## 2、GC参数 jstat

是Java虚拟机自带的统计监控工具，主要用于实时监测JVM内存管理和垃圾回收状态。本文将详解其发音规则及核心命令jstat -gc的使用方法。

### 2.1命令语法  jstat -gc 

```
jstat [options] <vmid> [interval[s|ms] [count>]
```

参数说明：
- options‌：指定你想要查看的统计信息类型。
- ‌vmid‌：虚拟机的唯一标识，可以是本地或远程虚拟机的进程ID或全局唯一标识符（例如，在Solaris上可以使用lsof -a -p <pid>来找到JVM进程的标识符）。
- ‌interval[s|ms]‌：两次统计之间的间隔时间，可选。
- ‌count‌：你想要获取统计的次数，可选。

常用选项[options‌]

  -  ‌-class‌：显示类加载、卸载数量以及总空间等信息。
  -  ‌-gc‌：显示与GC相关的堆行为统计数据，包括年轻代、老年代、永久代（或元空间）的大小和使用情况。
  -  ‌-gccapacity‌：显示各代的容量（年轻代、老年代、永久代或元空间）以及使用情况。
  -  ‌-gcutil‌：显示GC已使用空间占当前空间量的百分比，以及各代的内存使用情况。
  -  ‌-gccause‌：显示上一次或当前GC事件的原因。
  -  ‌-compiler‌：显示JIT编译器的状态信息。
  -  ‌-printcompilation‌：显示JVM编译方法的统计信息。


### 2.2  jstat -gc 

如：

```
jstat -gc <pid> [间隔时间] [统计次数]
```

```bash
[root@small-rose]$ jstat -gc 16628 2000 5
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
19456.0 16896.0  0.0   16888.3 246784.0 29230.5   208384.0   23549.3   76632.0 72578.1 10368.0 9609.8      9    0.052   3      0.132    0.184
19456.0 16896.0  0.0   16888.3 246784.0 30310.7   208384.0   23549.3   76632.0 72578.1 10368.0 9609.8      9    0.052   3      0.132    0.184
19456.0 16896.0  0.0   16888.3 246784.0 30310.7   208384.0   23549.3   76632.0 72578.1 10368.0 9609.8      9    0.052   3      0.132    0.184
19456.0 16896.0  0.0   16888.3 246784.0 30310.7   208384.0   23549.3   76632.0 72578.1 10368.0 9609.8      9    0.052   3      0.132    0.184
19456.0 16896.0  0.0   16888.3 246784.0 31391.0   208384.0   23549.3   76632.0 72578.1 10368.0 9609.8      9    0.052   3      0.132    0.184
```

列的含义：
```
S0C：年轻代中第一个Survivor区的容量，单位为KB。
S1C：年轻代中第二个Survivor区的容量，单位为KB。
S0U：年轻代中第一个Survivor区已使用大小，单位为KB。
S1U：年轻代中第二个Survivor区已使用大小，单位为KB。
EC：年轻代中Eden区的容量，单位为KB。
EU：年轻代中Eden区已使用大小，单位为KB。
OC：老年代的容量，单位为KB。
OU：老年代已使用大小，单位为KB。
MC：元空间的容量，单位为KB。
MU：元空间已使用大小，单位为KB。
CCSC：压缩类的容量，单位为KB。
CCSU：压缩类已使用大小，单位为KB。
YGC：Young GC的次数。
YGCT：Young GC所用的时间。
FGC：Full GC的次数。
FGCT：Full GC的所用的时间。
GCT：GC总耗时。
```
> C和U 容量（Capacity）和使用量（Used）如果是次数 C 是count的简写

jvm大小为 = 年轻代 + 老年代 = Ec + S0c + S1c + Oc

默认情况下：年轻代 1/3 ，老年代 2/3 .
若 -Xmx=4g ,则 年轻代 4096/3 = 1563.3 老年代： 4096/3*2 =2731

### 2.3  jstat -gcutil 

使用率百分比 gcutil

```bash
jstat -gcutil pid  3000 5
```

每隔3000毫秒获取一次结果，累计获取5次，内存各个区域使用率。

执行结果：
```
[root@small-rose]$ jstat -gcutil 16628  3000 5
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00  99.95  48.95  11.30  94.71  92.69      9    0.052     3    0.132    0.184
  0.00  99.95  49.39  11.30  94.71  92.69      9    0.052     3    0.132    0.184
  0.00  99.95  49.39  11.30  94.71  92.69      9    0.052     3    0.132    0.184
  0.00  99.95  49.39  11.30  94.71  92.69      9    0.052     3    0.132    0.184
  0.00  99.95  49.83  11.30  94.71  92.69      9    0.052     3    0.132    0.184
```

### 2.4  jstat -gccause‌ 

实时查上次GC原因 gccause‌

```bash
jstat -gccause‌ pid  3000 5
```
每隔3000毫秒获取一次结果，累计获取5次，内存各个区域使用率。

执行结果：
```
[root@small-rose]$ jstat -gccause 16628 3000 3
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
 95.31   0.00  41.88  11.30  94.92  92.69     10    0.059     3    0.132    0.191 Allocation Failure   No GC
 95.31   0.00  41.88  11.30  94.92  92.69     10    0.059     3    0.132    0.191 Allocation Failure   No GC
 95.31   0.00  42.85  11.30  94.92  92.69     10    0.059     3    0.132    0.191 Allocation Failure   No GC

```

### 2.5 jstat -gcmetacapacity

**查元空间使用情况**

```
jstat -gcmetacapacity <pid>
```
执行结果

```bash
[root@small-rose]$ jstat -gcmetacapacity 16628  
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT
       0.0  1116160.0    76632.0        0.0  1048576.0    10368.0     9     3    0.132    0.184

```
单位默认是KB:
 - MCMN​​ : Min metaspace capacity 最小容量（通常为0）。
 - MCMX :​​ Max metaspace capacity  1116160/1024 = 1090M 。
 - ​​MC​​ : Current metaspace capacity 当前使用量 76632/1024 = 74M。
 - CCSMX​​ Compressed class space max 压缩类空间最大。
 - CCSC​​ : Compressed class space capacity 压缩类空间当前。


## 3、GC参数 jmap


jmap（JVM Memory Map）：一方面是获取dump文件（堆转储快照文件，二进制文件），还可以获取目标Java进程的内存相关信息，包括Java堆各区域的使用情况、堆中对象的统计信息、类加载信息等。

可以在控制台中输入命令“jmap -help”查阅jmap的具体使用方式和一些标准选项命令参数。

查看命令帮助:

```bash
jmap -h
```

执行结果

```
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

### 3.1 jmap -dump


jmap把进程内存使用情况dump到文件中,

```bash
jmap -dump:format=b,file=dumpFileName.hrof pid

# live参数表示需要抓取目前在生命周期内的内存对象，也就是GC收不走的对象
jmap -dump:live,format=b,file=/applog/dump.hrof pid 
```

分析工具： 

- jvisualvm : 一般在JDK目录 **C:\Program Files\Java\jdk1.8.0_341\bin\jvisualvm.exe**
- Memory Analyzer 

mat与JDK版本对应关系

- 分析堆内存
- Memory Analyzer 1.14 及更高版本 JDK17及以上
- Memory Analyzer 1.12 及更高版本 JDK11及以上
- Memory Analyzer 1.8 至 1.11 需要 Java 1.8 VM 或更高版本的 VM 才能运行
- 最新版本：[https://eclipse.dev/mat/download/](https://eclipse.dev/mat/download/)
- 历史版本：[https://eclipse.dev/mat/download/previous/](https://eclipse.dev/mat/download/previous/)


### 3.2 jmap -heap
  
jmap 查询试试内存情况： 

```
jmap -heap pid
```
执行结果:

```bash
[root@small-rose]$ jmap -heap 16628
Attaching to process ID 16628, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.341-b10

using thread-local object allocation.
Parallel GC with 11 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 1073741824 (1024.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 357564416 (341.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 252706816 (241.0MB)
   used     = 246545648 (235.12425231933594MB)
   free     = 6161168 (5.8757476806640625MB)
   97.56193042296097% used
From Space:
   capacity = 13631488 (13.0MB)
   used     = 0 (0.0MB)
   free     = 13631488 (13.0MB)
   0.0% used
To Space:
   capacity = 17301504 (16.5MB)
   used     = 0 (0.0MB)
   free     = 17301504 (16.5MB)
   0.0% used
PS Old Generation
   capacity = 213385216 (203.5MB)
   used     = 23309272 (22.229454040527344MB)
   free     = 190075944 (181.27054595947266MB)
   10.923564639079776% used

26187 interned Strings occupying 2375328 bytes.
```

### 3.3 jmap -histo

查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象

```bash
jmap -histo[:live] pid
```

执行结果

```bash
[root@small-rose]$ jmap -histo:live 16628 | more
 num     #instances         #bytes  class name
----------------------------------------------
   1:         63361        5991328  [C
   2:         14406        1607792  java.lang.Class
   3:         62950        1510800  java.lang.String
   4:         13814        1215632  java.lang.reflect.Method
   5:         37815        1210080  java.util.concurrent.ConcurrentHashMap$Node
   6:         10956         637288  [Ljava.lang.Object;
   7:          3482         583976  [B
   8:         12087         386784  java.util.HashMap$Node
   9:          4493         376560  [Ljava.util.HashMap$Node;
  10:          7272         359272  [I
  11:          8655         346200  java.util.LinkedHashMap$Entry
  12:         21588         345408  java.lang.Object
  13:           192         315176  [Ljava.util.concurrent.ConcurrentHashMap$Node;
  14:          5519         309064  java.util.LinkedHashMap
  15:          9546         215024  [Ljava.lang.Class;
  16:          5636         135264  org.springframework.core.MethodClassKey
```

## 3、GC参数 jcmd

JVM诊断命令行工具，主要用于监控Java进程、执行线程分析、内存管理和性能调优‌。

```
[root@small-rose]$ jcmd -h
Usage: jcmd <pid | main class> <command ...|PerfCounter.print|-f file>
   or: jcmd -l
   or: jcmd -h

  command must be a valid jcmd command for the selected jvm.
  Use the command "help" to see which commands are available.
  If the pid is 0, commands will be sent to all Java processes.
  The main class argument will be used to match (either partially
  or fully) the class used to start Java.
  If no options are given, lists Java processes (same as -p).

  PerfCounter.print display the counters exposed by this process
  -f  read and execute commands from the file
  -l  list JVM processes on the local machine
  -h  this help
```

jcmd核心功能与使用场景

 - 进程管理‌：通过 `jcmd -l`列出当前所有Java进程信息。‌‌
 - ‌线程分析‌：使用 `jcmd <PID> Thread.print` 生成线程转储，定位死锁或高负载问题。‌‌
 - ‌内存监控‌：`jcmd <PID> GC.heap_dump` 导出堆内存快照，`jcmd <PID> VM.native_memory`显示本地内存分配。‌
 
性能调优应用
 
 - ‌GC分析‌：通过GC.class_histogram统计类实例分布，GC.run手动触发垃圾回收。‌‌1‌
 - ‌日志管理‌：利用GC.rotate_log循环记录GC日志文件。‌‌
 - ‌实时采样‌：配合JMC工具实现飞行记录器数据采集。
     
### 3.1 GC参数 jcmd -l


列出当前所有Java进程信息,类似 jps -l

```bash
[root@small-rose]$  jcmd -l
23712 sun.tools.jcmd.JCmd -l
16628 com.small.rose.demo.DbDemoApplication
12748 org.jetbrains.idea.maven.server.RemoteMavenServer36

```
### 3.1 GC参数 jcmd Thread.print

生成线程转储，定位死锁或高负载问题。‌‌

```bash
[root@small-rose]$  jcmd 16628  Thread.print
16628:
2025-10-29 17:12:02
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.341-b10 mixed mode):

"DestroyJavaVM" #43 prio=5 os_prio=0 tid=0x0000029c88f51800 nid=0x6278 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"http-nio-8081-Acceptor" #42 daemon prio=5 os_prio=0 tid=0x0000029c88f4f000 nid=0x5de4 runnable [0x000000b7ba3fe000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
        at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:424)
        at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:252)
        - locked <0x00000000fbb1a508> (a java.lang.Object)
        at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:546)
        at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:79)
        at org.apache.tomcat.util.net.Acceptor.run(Acceptor.java:129)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-Poller" #41 daemon prio=5 os_prio=0 tid=0x0000029c88f50800 nid=0x1e04 runnable [0x000000b7ba2fe000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll0(Native Method)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll(WindowsSelectorImpl.java:296)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.access$400(WindowsSelectorImpl.java:278)
        at sun.nio.ch.WindowsSelectorImpl.doSelect(WindowsSelectorImpl.java:159)
        at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
        - locked <0x00000000fbb1ad58> (a sun.nio.ch.Util$3)
        - locked <0x00000000fbb1ad48> (a java.util.Collections$UnmodifiableSet)
        - locked <0x00000000fbb1abd8> (a sun.nio.ch.WindowsSelectorImpl)
        at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
        at org.apache.tomcat.util.net.NioEndpoint$Poller.run(NioEndpoint.java:807)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-10" #40 daemon prio=5 os_prio=0 tid=0x0000029c88f50000 nid=0x5d5c waiting on condition [0x000000b7ba1fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-9" #39 daemon prio=5 os_prio=0 tid=0x0000029c88f4a800 nid=0x25d4 waiting on condition [0x000000b7ba0ff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-8" #38 daemon prio=5 os_prio=0 tid=0x0000029c88f4d800 nid=0x37ec waiting on condition [0x000000b7b9fff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-7" #37 daemon prio=5 os_prio=0 tid=0x0000029c88f4d000 nid=0x11a0 waiting on condition [0x000000b7b9eff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-6" #36 daemon prio=5 os_prio=0 tid=0x0000029c88f4c000 nid=0x6204 waiting on condition [0x000000b7b9dfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-5" #35 daemon prio=5 os_prio=0 tid=0x0000029c88f4a000 nid=0x5a08 waiting on condition [0x000000b7b9cfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-4" #34 daemon prio=5 os_prio=0 tid=0x0000029c88f4b800 nid=0x1750 waiting on condition [0x000000b7b9bfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-3" #33 daemon prio=5 os_prio=0 tid=0x0000029c88f48800 nid=0x4d0c waiting on condition [0x000000b7b9aff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-2" #32 daemon prio=5 os_prio=0 tid=0x0000029c88f49000 nid=0x5418 waiting on condition [0x000000b7b99ff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"http-nio-8081-exec-1" #31 daemon prio=5 os_prio=0 tid=0x0000029c88f47800 nid=0x2c08 waiting on condition [0x000000b7b97fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000fbb18340> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:146)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1114)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1176)
        at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"Live Reload Server" #30 daemon prio=5 os_prio=0 tid=0x0000029c88f47000 nid=0x6a1c runnable [0x000000b7b98fe000]
   java.lang.Thread.State: RUNNABLE
        at java.net.DualStackPlainSocketImpl.accept0(Native Method)
        at java.net.DualStackPlainSocketImpl.socketAccept(DualStackPlainSocketImpl.java:127)
        at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:535)
        at java.net.PlainSocketImpl.accept(PlainSocketImpl.java:189)
        - locked <0x00000000fb803d68> (a java.net.SocksSocketImpl)
        at java.net.ServerSocket.implAccept(ServerSocket.java:545)
        at java.net.ServerSocket.accept(ServerSocket.java:513)
        at org.springframework.boot.devtools.livereload.LiveReloadServer.acceptConnections(LiveReloadServer.java:145)
        at org.springframework.boot.devtools.livereload.LiveReloadServer$$Lambda$1066/825352379.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:750)

"File Watcher" #28 daemon prio=5 os_prio=0 tid=0x0000029c88f45800 nid=0x3f38 waiting on condition [0x000000b7b96ff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at org.springframework.boot.devtools.filewatch.FileSystemWatcher$Watcher.scan(FileSystemWatcher.java:279)
        at org.springframework.boot.devtools.filewatch.FileSystemWatcher$Watcher.run(FileSystemWatcher.java:263)
        at java.lang.Thread.run(Thread.java:750)

"lettuce-timer-3-1" #27 daemon prio=5 os_prio=0 tid=0x0000029c88f44800 nid=0x6668 waiting on condition [0x000000b7b95ff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at io.netty.util.HashedWheelTimer$Worker.waitForNextTick(HashedWheelTimer.java:600)
        at io.netty.util.HashedWheelTimer$Worker.run(HashedWheelTimer.java:496)
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        at java.lang.Thread.run(Thread.java:750)

"Druid-ConnectionPool-Destroy-1314142852" #26 daemon prio=5 os_prio=0 tid=0x0000029c88f44000 nid=0x6ab4 waiting on condition [0x000000b7b94ff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at com.alibaba.druid.pool.DruidDataSource$DestroyConnectionThread.run(DruidDataSource.java:2887)

"Druid-ConnectionPool-Create-1314142852" #25 daemon prio=5 os_prio=0 tid=0x0000029c88f46000 nid=0x1598 waiting on condition [0x000000b7b93ff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000c0a99c30> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2788)

"OracleTimeoutPollingThread" #24 daemon prio=10 os_prio=2 tid=0x0000029c88f43000 nid=0x29f4 waiting on condition [0x000000b7b92ff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at oracle.jdbc.driver.OracleTimeoutPollingThread.run(OracleTimeoutPollingThread.java:150)

"container-0" #23 prio=5 os_prio=0 tid=0x0000029c868d8000 nid=0xd6c waiting on condition [0x000000b7b90fe000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at org.apache.catalina.core.StandardServer.await(StandardServer.java:563)
        at org.springframework.boot.web.embedded.tomcat.TomcatWebServer$1.run(TomcatWebServer.java:197)

"Catalina-utility-2" #22 prio=1 os_prio=-2 tid=0x0000029c868db000 nid=0x4900 waiting on condition [0x000000b7b8fff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000c0cd74a0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1088)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"Catalina-utility-1" #21 prio=1 os_prio=-2 tid=0x0000029c868da800 nid=0xa5c waiting on condition [0x000000b7b8eff000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000c0cd74a0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:750)

"Service Thread" #13 daemon prio=9 os_prio=0 tid=0x0000029c85d6a000 nid=0x1e38 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #12 daemon prio=9 os_prio=2 tid=0x0000029c85ce4800 nid=0x4eec waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #11 daemon prio=9 os_prio=2 tid=0x0000029c85ce1800 nid=0x2900 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #10 daemon prio=9 os_prio=2 tid=0x0000029c85cdb000 nid=0x4374 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #9 daemon prio=9 os_prio=2 tid=0x0000029c85c69800 nid=0x6ab0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Command Reader" #8 daemon prio=10 os_prio=0 tid=0x0000029c857ae000 nid=0x56d4 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Event Helper Thread" #7 daemon prio=10 os_prio=0 tid=0x0000029c857a9000 nid=0x5ae8 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Transport Listener: dt_socket" #6 daemon prio=10 os_prio=0 tid=0x0000029c83069000 nid=0x5df8 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x0000029c83059800 nid=0x364c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x0000029c8579e000 nid=0x4cb4 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x0000029c8303d800 nid=0x2988 in Object.wait() [0x000000b7b80ff000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000c001b9e8> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:150)
        - locked <0x00000000c001b9e8> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:171)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x0000029c83030000 nid=0x4cd0 in Object.wait() [0x000000b7b7fff000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000c00240e0> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000c00240e0> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x0000029c83007800 nid=0x5b04 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x0000029cef161800 nid=0x69b0 runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x0000029cef163000 nid=0x65f0 runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x0000029cef165000 nid=0x5668 runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x0000029cef166800 nid=0x5894 runnable

"GC task thread#4 (ParallelGC)" os_prio=0 tid=0x0000029cef168800 nid=0x4338 runnable

"GC task thread#5 (ParallelGC)" os_prio=0 tid=0x0000029cef169800 nid=0x6b34 runnable

"GC task thread#6 (ParallelGC)" os_prio=0 tid=0x0000029cef16c800 nid=0x68d0 runnable

"GC task thread#7 (ParallelGC)" os_prio=0 tid=0x0000029cef16d800 nid=0x234c runnable

"GC task thread#8 (ParallelGC)" os_prio=0 tid=0x0000029cef16e800 nid=0x6228 runnable

"GC task thread#9 (ParallelGC)" os_prio=0 tid=0x0000029cef171800 nid=0x57c runnable

"GC task thread#10 (ParallelGC)" os_prio=0 tid=0x0000029cef172800 nid=0x1098 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x0000029c85e0d000 nid=0x32f4 waiting on condition

JNI global references: 33451

```

### 3.1 GC参数 jcmd 

```bash
#查看类加载情况
jcmd <pid> VM.classloader_stats
```


```bash
#检查重复类
jcmd <pid> VM.find_class_by_name java.lang.Object
```

## 4、 jstack

jstack主要用来查看某个Java进程内的线程堆栈信息。


```bash
jstack [option] pid
jstack [option] executable core
jstack [option] [server-id@]remote-hostname-or-ip
```


```bash
[root@small-rose]$  jstack -h
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message

```


- `-l long listings`，会打印出额外的锁信息，在发生死锁时可以用 `jstack -l pid来`观察锁持有情况
- `-m mixed mode`，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）

### 4、1 jstack 定位线程慢的原因

(1) 找出应用进程pid，如jps -l 没有就使用ps -ef | grep java。

(2) 找出pid进程内最耗费CPU的线程，可以使用`ps -Lfp pid`或者`ps -mp pid -o THREAD, tid, time`或者`top -Hp pid` .

(3) 查看TIME列就是各个Java线程耗费的CPU时间，CPU时间最长的是线程ID为 2877的线程

(4) 将对于线程ID转换为 16进制

(5) 使用jstack定位线程和原因  jstack 21711 | grep 0xB3D



## 5、 GC分析

Full GC是JVM垃圾回收中最重要的事件之一，分析Full GC日志可以帮助识别内存问题、性能瓶颈和优化机会。以下是分析Full GC日志的详细方法：

**Full GC常见原因及诊断方法​​**

|原因分类| 具体场景 | 诊断方法 | 关键指标 |
|-----|--------|--------|-------|
|老年代空间不足​​ | 大对象直接分配/对象晋升过快 | jmap -histo:live <pid> | O列接近100% |
|​​Metaspace耗尽​​ | 动态类加载过多 | jstat -gcmetacapacity <pid> | M列接近100% |
| ​​System.gc()调用​​ |代码或三方库触发 | jcmd <pid> VM.log what=gc | 查看GC原因字段 |
​​分配失败担保​​ |Young GC后Survivor放不下 | -XX:+PrintTenuringDistribution | 晋升年龄异常 |
​​堆外内存不足​​ |Direct Buffer或Native内存耗尽 |jcmd <pid> VM.native_memory | Native内存使用量 |


### 5.1. Full GC日志的基本结构

典型的Full GC日志示例（G1 GC为例）：

``
[Full GC (Allocation Failure) 
[PSYoungGen: 1024K->0K(2048K)] 
[ParOldGen: 4096K->4096K(8192K)] 
5120K->4096K(10240K), 
[Metaspace: 2560K->2560K(1056768K)], 
  0.123456 secs]
``

关键信息解析

5.1.1 触发原因

 - ​​Allocation Failure​​：年轻代空间不足
 - ​​Metadata GC Threshold​​：元空间不足
 - ​​System.gc()​​：显式调用
 - ​​Ergonomics​​：JVM自适应机制触发
 
5.1.2 各区域内存变化

```
[PSYoungGen: 1024K->0K(2048K)]  # 年轻代: 回收前->回收后(总容量)
[ParOldGen: 4096K->4096K(8192K)] # 老年代: 回收前->回收后(总容量)
5120K->4096K(10240K)            # 堆总量: 回收前->回收后(总容量)
[Metaspace: 2560K->2560K(1056768K)] # 元空间
```

5.1.3 时间信息

0.123456 secs：暂停时间（秒）

5.2  重点分析指标

5.2.1 回收效率

- 老年代回收量​​：ParOldGen: 4096K->4096K表示没有回收任何对象
- 堆总量变化​​：5120K->4096K表示回收了1024K

5.2.2 内存使用率

- 回收后老年代使用率：4096K/8192K = 50%
- 元空间使用率：2560K/1056768K ≈ 0.24%

5.2.3 时间消耗

Full GC时间超过1秒通常需要关注。

频繁Full GC（如每分钟多次）是严重问题。

### 6. 常见问题诊断

6.1 内存泄漏迹象

- 老年代使用量持续增长
- 每次Full GC后老年代回收量很少
- 最终导致OutOfMemoryError

6.2 配置不当

- 年轻代过小导致过早晋升
- 堆总量不足
- 元空间未设置上限

6.3 性能问题

- Full GC频率过高
- 单次Full GC时间过长
- 系统吞吐量下降

### 7 开启GC日志

#### 7。1 添加JVM参数获取完整GC日志

```
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps 
-XX:+PrintGCCause 
-Xloggc:/path/to/gc.log
```

#### 7。2 使用工具分析


```bash
java -jar gcviewer.jar gc.log

# 发生Full GC后立即dump堆
jmap -dump:live,format=b,file=heap.hprof <pid>

# 使用MAT分析大对象
java -jar mat/ParseHeapDump.sh heap.hprof
```


GCViewer下载地址: 

- 分析GC日志
- GCViewer [https://github.com/chewiebug/GCViewer](https://github.com/chewiebug/GCViewer)




