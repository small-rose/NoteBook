---
layout: default
title: Java -jar 
parent: Java
has_children: false
nav_order: 99
---



# Java -jar params
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


# jar包启动命令与参数

## 1、基础命令

```
java -jar appdemo.jar
```
## 2、进阶参数

### 2.1  -XX参数

标准选择（Standard Options）这些是 JVM 的所有实现都支持的最常用的选项。及该类型参数是标准参数，所有JVM实现都支持。

例如以-XX开头的配置参数
```
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnable
-XX:+UseFastAccessorMethods
```

`-XX`: 参数大致分为三类，根据其格式区分：

- 布尔类型选项：用于启用或禁用某个特性。
    - 格式：**`-XX:+<Option>`** 或 **`-XX:-<Option>`**
    - **`+` 表示启用该选项, `-` 表示禁用该选项**
    
- 键值对选项： 用于设置某个参数的具体值。
    - 格式：**`-XX:<Option>=<Value>`**

- 诊断选项：用于调试和诊断 JVM 的行为。
    - 格式：类似于布尔类型和键值对，但通常需要额外启用 **`-XX:+UnlockDiagnosticVMOptions`**
    
布尔类型参数

| 参数 | 说明 |默认值|
|------|-----|-----|
| `-XX:+UseG1GC` |	启用 G1 垃圾回收器。|	否|
| `-XX:+UseParallelGC` |启用并行垃圾回收器（Parallel GC）。	|是 |
| `-XX:+UseConcMarkSweepGC` |启用 CMS 垃圾回收器（Concurrent Mark-Sweep）。	|否 |
| `-XX:-UseAdaptiveSizePolicy` |禁用自适应内存分配策略（与垃圾回收器有关）。	|否 |
| `-XX:+PrintGCDetails` |输出详细的 GC 日志信息。	|否 |
| `-XX:+PrintGCDateStamps` |输出 GC 日志时添加时间戳。	|否 |
| `-XX:+HeapDumpOnOutOfMemoryError` |当发生 OutOfMemoryError 时生成堆转储文件。	|否 |

键值对类型参数

| 参数 | 说明 |默认值|
|------|-----|-----|
| `-XX:MaxHeapSize=<size>`	|设置堆的最大大小。例如 `-XX:MaxHeapSize=512m` 或 `-XX:MaxHeapSize=2g`。	|系统自动计算 |
| `-XX:InitialHeapSize=<size>`	|设置堆的初始大小。	|系统自动计算|
| `-XX:MaxMetaspaceSize=<size>`	|设置元空间的最大大小（仅适用于 Java 8 及以上）。	|无限（受系统限制） |
| `-XX:ThreadStackSize=<size>`	|设置每个线程的堆栈大小。	|与操作系统相关 |
| `-XX:NewRatio=<value>`	|设置新生代和老年代内存比例。例如，`-XX:NewRatio=2` 表示新生代是老年代的 1/2。	|2|
| `-XX:SurvivorRatio=<value>`	|设置 Eden 区和 Survivor 区的比例。例如，`-XX:SurvivorRatio=8` 表示 Eden 是 Survivor 的 8 倍。	|8 |
| `-XX:MaxTenuringThreshold=<value>`	|设置对象从新生代晋升到老年代所需的最大年龄。	|15 |

诊断和调试参数

诊断参数需要配合 -XX:+UnlockDiagnosticVMOptions 使用。例如：
```
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal -version
```

| 参数 | 说明 |默认值|
|------|-----|-----|
| `-XX:+UnlockExperimentalVMOptions` |	启用实验性参数。|	使用时需谨慎|
| `-XX:+PrintFlagsFinal` | 打印 JVM 参数的最终值，包括默认值和用户设置值。	|调试常用|
| `-XX:+TraceClassLoading`	| 打印类加载的详细信息。	|调试类加载问题 |
| `-XX:+TraceClassUnloading` | 打印类卸载的详细信息。	|调试类卸载问题 |
| `-XX:+LogCompilation`	| 输出 JIT 编译相关日志，通常与 -XX:+UnlockDiagnosticVMOptions 一起使用。	|性能调优|
| `-XX:+PrintGCApplicationStoppedTime` | 打印 GC 导致的应用停止的时间。 |GC 调优|

常见场景下的 -XX: 参数使用

(1) 设置 JVM 堆大小
 
设置 JVM 的堆初始大小和最大大小：
```
java -XX:InitialHeapSize=512m -XX:MaxHeapSize=2g -jar app.jar
```
- 初始堆大小：512 MB。
- 最大堆大小：2 GB。

(2) 启用 G1 垃圾回收器

使用 G1 垃圾回收器并设置最大暂停时间目标：
```
java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -jar app.jar
```

- `-XX:+UseG1GC` ：启用 G1 垃圾回收器。
- `-XX:MaxGCPauseMillis=200`：目标最大暂停时间为 200 毫秒。

(3) 打印 GC 日志

调试 GC 时输出详细日志：
```
java -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log -jar app.jar
```
- `-XX:+PrintGCDetails`：打印详细的 GC 信息。
- `-XX:+PrintGCDateStamps`：添加时间戳。
- `-Xloggc:gc.log`：将 GC 日志输出到文件 gc.log。

(4) 调试类加载

输出类加载和卸载信息：
```
java -XX:+TraceClassLoading -XX:+TraceClassUnloading -jar app.jar
```
- `-XX:+TraceClassLoading`：打印类加载的详细信息。
- `-XX:+TraceClassUnloading`：打印类卸载的详细信息。

(5) 在 OOM 时生成堆转储

捕获内存溢出时生成堆转储文件：

java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./heapdump.hprof -jar app.jar

- `-XX:+HeapDumpOnOutOfMemoryError`：发生 OutOfMemoryError 时生成堆转储。
- `-XX:HeapDumpPath=./heapdump.hprof`：设置堆转储文件的保存路径。

(6) 如何查看支持的 `-XX:` 参数 ? 

运行以下命令查看所有 JVM 参数及其默认值：
```
java -XX:+PrintFlagsFinal -version
java -XX:+PrintFlagsFinal -version | grep "G1"
```

列出所有支持的 JVM 参数：
```
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsInitial -version
```

{: .tips }
> 注意事项
> - 稳定性：某些 -XX: 参数是实验性或诊断参数，可能在不同版本的 JVM 中行为不同，需谨慎使用。
> - 兼容性：参数在不同的 JVM 实现（如 Oracle JDK 和 OpenJDK）中可能略有差异。
> - 调优慎重：在生产环境中调整 -XX: 参数时，应先在测试环境中充分验证。
    
### 2.2  -X参数

非标准选择（Non-Standard Options）这些选项是特定于 Java HotSpot 虚拟机的通用选项。


例如以-X开头的配置参数
```
-Xmx256m
-Xms256m
-Xmn768m
-Xss256k
```

### 2.3  -D参数

设置系统属性值；

-D属性名称=属性值

```
-DpropName=propValue
```
`-DpropName=propValue`的形式携带，**要放在-jar参数前面**

>  -D=：设置系统属性。

实例：

```bash
java -DpropName=test -DprocessType=1 -jar appdemo.jar
```
代码取值：
```
System.getProperty("propName");
System.getProperty("processType");
```

比如使用 -D设置外部配置文件路径

```bash
-Dspring.config.location=/app/conf/
```

> 如果 属性值 是一个带有空格的字符串，那么用引号将其括起来
  例如 -Dfoo = " foo bar"


### 2.2  `--`参数

springboot的方式，`--key=value`方式

```bash
java -jar appdemo.jar --propName=test
```
代码取值：

spring的 `@value("${propName}")`


比如 
Java 9 引入的模块系统（JPMS）​默认严格隔离模块，导致以下问题：
- ​传统类路径代码无法访问 Java 标准库的内部 API​
  > 例如：旧代码直接通过反射访问 java.lang.reflect.Proxy 的私有方法，或第三方库调用 sun.misc.Unsafe。
- 未模块化的第三方库冲突​
  > 如果第三方库未适配模块化，其代码可能依赖 Java 标准库的非公开接口。

而使用 `--add-opens` 会临时打破这种隔离，允许未命名模块访问指定包。
```
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.io=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
--add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.baSe/jaVa.util=ALL-UNNAMED
--add-opens=jaVa.baSe/jaVa.math=ALL-UNNAMED
--add-opens=java.base/java.net=ALL-UNNAMED
--add-opens=java.base/java.text=ALL-UNNAMED
--add-opens=java.baSe/sun.Security.util=ALL-UNNAMED
```


### 2.3 参数直接跟在命令后面

```bash
java -jar appdemo.jar processType=1 processType2=2
```
代码取值: 参数就是jar包里主启动类中main方法的args参数，按顺序来(空格区分参数个数)

## 3、命令写法

第1种：

```bash
java -jar appdemo.jar
```
第1种写法：用这种方法启动后，不能继续执行其它命令了，如果想要继续执行其它命令，需要退出当前命令运行窗口，会打断jar的运行，打断一般用ctrl+c。

第2种：

```bash
java -jar demo.jar &
```
第2种在第1种方式的基础上在命令的结尾增加了&，**&表达的意思是在后台运行。**

**这种方式可以避免打断后程序停止运行的问题，但是如果关闭当前窗口后程序会停止运行。**

第3种：
```bash
nohup java -jar demo.jar &
```

第3种在第2种方式的基础上，在命令的最前面增加了nohup。

**nohup是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行。**

加了nohup后，即使关掉命令窗口，后台程序demo.jar也会一直执行。

第4种
```bash
nohup java -jar demo.jar >1.txt &
```

第4种在第3种的基础上，在后面增加了 `>1.txt`，意思是将nohup java -jar demo.jar的运行内容重定向输出到`1.txt`文件中，即输出内容不打印到当前窗口上，而是输出到1.txt文件中。

第3种没有加`>1.txt`，它的输出重定向到nohup.out文件中，nohup.out也就是nohup命令的默认输出文件， 文件位于`$HOME/nohup.out` 文件中,比如用root执行，就会输出到`/root/nohup.out`。

第5种
```bash
nohup java -jar demo.jar >/dev/null 2>&1 &
```

这里说下jar后面这串符号的意义 `>/dev/null 2>&1 &` 
>   代表重定向到哪里，例如：echo "123" > /home/123.txt
    /dev/null 代表空设备文件
    2> 表示stderr标准错误
    & 表示等同于的意思，2>&1，表示2的输出重定向等同于1
    1 表示stdout标准输出，系统默认值是1，所以">/dev/null"等同于 "1>/dev/null"
    最后一个&表示在后台运行。

这里再补充说下这几个数字代表的含义：
> 0 标准输入（一般是键盘）
  1 标准输出（一般是显示屏，是用户终端控制台）
  2 标准错误（错误信息输出）
  /dev/null ：首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。一般项目中定义中输出运行日志到指定地址，这样的话，就不需要再单独输出nohup.out文件，这种情况可以考虑使用这种。

我们可以把它写成一个脚本，不用每次都写一遍。新建start.sh，根据我上传的demo.jar,输出到1.txt中，具体脚本如下：

```bash
nohup  java  -Xms512m -Xmx1024m -jar -Dfile.encoding=UTF-8 demo.jar --spring.profiles.active=prod >/dev/null 2>&1 &
```

可以看到，上面的命令中我使用了Xms、Xmx、Dfile.encoding、spring.profiles.active等参数，那么java -jar可以添加什么参数，各自又能实现什么样的效果呢，且看下文：

> -Xms 指定jvm运行最小运行堆内存，默认为物理内存1/64，用法 ：-Xmx512m 注意：Xmx和512m中间不用添加空格
  -Xmx 指定jvm运行最大运行堆内存，认物理内存1/4，用法： -Xmx1024m 注意：Xmx和1024m中间不用添加空格
  --server.port 指定jar运行的port端口，用法：--server.port=8085
  --spring.profiles.active=pro 指定运行的配置文件、环境，用法：--spring.profiles.active=prod



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
