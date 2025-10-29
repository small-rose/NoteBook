---
layout: default
title: Java jvm 
parent: Java
has_children: false
nav_order: 100
---


 Here are commonly used Java examples .
{: .fs-6 .fw-300 }




# jvm 命令与参数

## 1、基础命令 jps

**查看java程的pid及基本信息**

```
jps -l
```


**查看进程pid及main方法参数**

```
jps -m
```

执行结果：
```
15332 sun.tools.jps.Jps -m -l
16628 com.small.rose.demo.DbDemoApplication
12748 org.jetbrains.idea.maven.server.RemoteMavenServer36
```

**查看pid及JVM参数**

```
jps -v
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

如：

```
jstat -gcutil <pid> [间隔时间] [统计次数]
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


**使用率百分比 gcutil**

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

**GC原因 gccause‌**

```bash
jstat -gccause‌ pid  3000 5
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

**元空间使用情况**

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


## 2、GC参数 jmap

jmap把进程内存使用情况dump到文件中

```bash
jmap -dump:format=b,file=dumpFileName.hrof pid

# live参数表示需要抓取目前在生命周期内的内存对象，也就是GC收不走的对象
jmap -dump:live,format=b,file=/applog/dump.hrof pid 
```

  
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




## 3、 GC分析

Full GC是JVM垃圾回收中最重要的事件之一，分析Full GC日志可以帮助识别内存问题、性能瓶颈和优化机会。以下是分析Full GC日志的详细方法：

### 3.1. Full GC日志的基本结构

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

3.1.1 触发原因

 - ​​Allocation Failure​​：年轻代空间不足
 - ​​Metadata GC Threshold​​：元空间不足
 - ​​System.gc()​​：显式调用
 - ​​Ergonomics​​：JVM自适应机制触发


**Full GC常见原因及诊断方法​​**

|原因分类| 具体场景 | 诊断方法 | 关键指标 |
|-----|--------|--------|-------|
|老年代空间不足​​ | 大对象直接分配/对象晋升过快 | jmap -histo:live <pid> | O列接近100% |
|​​Metaspace耗尽​​ | 动态类加载过多 | jstat -gcmetacapacity <pid> | M列接近100% |
| ​​System.gc()调用​​ |代码或三方库触发 | jcmd <pid> VM.log what=gc | 查看GC原因字段 |
​​分配失败担保​​ |Young GC后Survivor放不下 | -XX:+PrintTenuringDistribution | 晋升年龄异常 |
​​堆外内存不足​​ |Direct Buffer或Native内存耗尽 |jcmd <pid> VM.native_memory | Native内存使用量 |


3.1.2 各区域内存变化

```
[PSYoungGen: 1024K->0K(2048K)]  # 年轻代: 回收前->回收后(总容量)
[ParOldGen: 4096K->4096K(8192K)] # 老年代: 回收前->回收后(总容量)
5120K->4096K(10240K)            # 堆总量: 回收前->回收后(总容量)
[Metaspace: 2560K->2560K(1056768K)] # 元空间
```

3.1.3 时间信息

0.123456 secs：暂停时间（秒）

3. 重点分析指标

3.1 回收效率

- 老年代回收量​​：ParOldGen: 4096K->4096K表示没有回收任何对象
- 堆总量变化​​：5120K->4096K表示回收了1024K

3.2 内存使用率

- 回收后老年代使用率：4096K/8192K = 50%
- 元空间使用率：2560K/1056768K ≈ 0.24%

3.3 时间消耗

Full GC时间超过1秒通常需要关注。

频繁Full GC（如每分钟多次）是严重问题。

4. 常见问题诊断

4.1 内存泄漏迹象

- 老年代使用量持续增长
- 每次Full GC后老年代回收量很少
- 最终导致OutOfMemoryError

4.2 配置不当

- 年轻代过小导致过早晋升
- 堆总量不足
- 元空间未设置上限

4.3 性能问题

- Full GC频率过高
- 单次Full GC时间过长
- 系统吞吐量下降


#### 添加JVM参数获取完整GC日志

```
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps 
-XX:+PrintGCCause 
-Xloggc:/path/to/gc.log
```

#### 使用工具分析


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

mat与JDK版本对应关系

- 分析堆内存
- Memory Analyzer 1.14 及更高版本 JDK17及以上
- Memory Analyzer 1.12 及更高版本 JDK11及以上
- Memory Analyzer 1.8 至 1.11 需要 Java 1.8 VM 或更高版本的 VM 才能运行
- 最新版本：[https://eclipse.dev/mat/download/](https://eclipse.dev/mat/download/)
- 历史版本：[https://eclipse.dev/mat/download/previous/](https://eclipse.dev/mat/download/previous/)


