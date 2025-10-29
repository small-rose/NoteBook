---
layout: default
title: Java jvm 
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

jvm大小为：
= 年轻代 + 老年代
=Ec + S0c + S1c + Oc

默认情况下：年轻代 1/3 ，老年代 2/3 .
若 -Xmx=4g ,则 年轻代 4096/3 = 1563.3 老年代： 4096/3*2 =2731



 
**元空间使用情况**

```
jstat -gcmetacapacity <pid>
```


## 2、GC参数 jmap

jmap把进程内存使用情况dump到文件中

```
jmap -dump:format=b,file=dumpFileName.hrof pid
```


在启动命令中捕获内存溢出时生成堆转储文件：

java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./heapdump.hprof -jar app.jar

- `-XX:+HeapDumpOnOutOfMemoryError`：发生 OutOfMemoryError 时生成堆转储。
- `-XX:HeapDumpPath=./heapdump.hprof`：设置堆转储文件的保存路径。
  

## 3、内存大小

查默认垃圾回收器

```
# openJdk 查看GC参数
java -XX:+PrintFlagsFinal -version | grep "Use.*GC"

# 查看GC日志确认压缩行为
java -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseMaximumCompactionOnSystemGC -version
```


## 4、 GC分析

Full GC是JVM垃圾回收中最重要的事件之一，分析Full GC日志可以帮助识别内存问题、性能瓶颈和优化机会。以下是分析Full GC日志的详细方法：

### 4.1. Full GC日志的基本结构

典型的Full GC日志示例（G1 GC为例）：

``
[Full GC (Allocation Failure) 
[PSYoungGen: 1024K->0K(2048K)] 
[ParOldGen: 4096K->4096K(8192K)] 
5120K->4096K(10240K), 
[Metaspace: 2560K->2560K(1056768K)], 
  0.123456 secs]
``

2. 关键信息解析

2.1 触发原因

 - ​​Allocation Failure​​：年轻代空间不足
 - ​​Metadata GC Threshold​​：元空间不足
 - ​​System.gc()​​：显式调用
 - ​​Ergonomics​​：JVM自适应机制触发

2.2 各区域内存变化

```
[PSYoungGen: 1024K->0K(2048K)]  # 年轻代: 回收前->回收后(总容量)
[ParOldGen: 4096K->4096K(8192K)] # 老年代: 回收前->回收后(总容量)
5120K->4096K(10240K)            # 堆总量: 回收前->回收后(总容量)
[Metaspace: 2560K->2560K(1056768K)] # 元空间
```

2.3 时间信息

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
