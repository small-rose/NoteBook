---
layout: default
title: Jvm Params
parent: DevNotes
nav_order: 9
---


# Jvm parameters
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Jvm 相关

##  jps 命令

jps(Java Virtual Machine Process Status Tool) 

> 是java提供的一个显示当前所有java进程pid的命令，适合在linux/unix平台上简单察看当前java进程的一些简单情况。
很多人都是用过unix系统里的ps命令，这个命令主要是用来显示当前系统的进程情况，有哪些进程以及进程id。 
jps 也是一样，它的作用是显示当前系统的java进程情况及进程id。

**命令格式**

```bash
jps [-l][-m][-v][-q][-V]
```

**命令参数**

- -help
- -l 输出应用程序main class的完整package名或者应用程序的jar文件完整路径名.
- -v 输出传递给JVM的参数
- -m 查看进程pid及main方法参数
- -q 仅显示pid
- -V 查看pid及JVM参数

```bash
[root@VM-0-3-centos ~]# jps -l
20822 small-admin.jar
32522 small-rose-js.jar
17070 sun.tools.jps.Jps
[root@VM-0-3-centos ~]# jps -v
20822 jar -Dserver.port=8001
17112 Jps -Denv.class.path=.:/usr/jdk1.8/jre/lib/rt.jar:/usr/jdk1.8/lib/dt.jar:/usr/jdk1.8/lib/tools.jar -Dapplication.home=/usr/jdk1.8 -Xms8m
32522 jar -Dserver.port=8088
```

##  jstack 命令

使用jstack可查看指定进程（pid）的堆栈信息，用以分析线程情况：

- NEW：未启动的。不会出现在Dump中。
- RUNNABLE：在虚拟机内执行的。
- BLOCKED：受阻塞并等待监视器锁。
- WATING：无限期等待另一个线程执行特定操作。
- TIMED_WATING：有时限的等待另一个线程的特定操作。
- TERMINATED：已退出的。 

**命令格式**

```bash
jstack [-l][-m][-F] pid
```
**命令参数**

- -h 帮助
- -l：长列表，打印锁的附加信息；
- -m：打印java和native c/c++框架的所有栈信息；
- -F：没有响应的时候强制打印栈信息；

将线程信息输出到文件

```bash
jstack -l 20822 >> 1.txt
```

## jmap 命令 与 jhat 命令

查看堆内存使用情况：

- `jmap -heap pid`  通过可查看堆内存的配置情况及使用情况
- `jmap -histo pid`  统计对象的创建数量
- `jmap -dump:format=b,file=heapDump pid` 生成dump文件与jhat配合使用
- `jhat -port xxxx heapDump` 浏览器访问localhost:xxxx即可查看dump

示例：

```bash
[root@VM-0-3-centos ~]# jmap -heap 20822
Attaching to process ID 20822, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.212-b10

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 482344960 (460.0MB)
   NewSize                  = 10485760 (10.0MB)
   MaxNewSize               = 160759808 (153.3125MB)
   OldSize                  = 20971520 (20.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 56688640 (54.0625MB)
   used     = 19885608 (18.964393615722656MB)
   free     = 36803032 (35.098106384277344MB)
   35.078647150469656% used
Eden Space:
   capacity = 50397184 (48.0625MB)
   used     = 14152632 (13.497001647949219MB)
   free     = 36244552 (34.56549835205078MB)
   28.082188084159622% used
From Space:
   capacity = 6291456 (6.0MB)
   used     = 5732976 (5.4673919677734375MB)
   free     = 558480 (0.5326080322265625MB)
   91.12319946289062% used
To Space:
   capacity = 6291456 (6.0MB)
   used     = 0 (0.0MB)
   free     = 6291456 (6.0MB)
   0.0% used
tenured generation:
   capacity = 125743104 (119.91796875MB)
   used     = 81101696 (77.3446044921875MB)
   free     = 44641408 (42.5733642578125MB)
   64.49792745692042% used

50389 interned Strings occupying 5094968 bytes.
```

生成heapDump文件，默认生成在当前执行命令的目录

```bash
 jmap -dump:format=b,file=heapDump 20822
```
再通过jhat -port 8099 heapDump 启动一个监听程序查看文件：

访问 `http://localhost:8099` 查看dump文件。



## 在Windows、Linux中查看Xmx和Xms默认值的大小

- windows

```bash
java -XX:+PrintFlagsFinal -version | findstr /i "HeapSize"
```

- linux

```bash
java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize'
```

## 在Linux中查看运行中Java程序的Xmx和Xms值

获取应用进程的PID

```bash
ps -ef | grep java 
jsp -l
```

然后根据PID查看当前设置值

```bash
sudo jcmd PID VM.flags
```
