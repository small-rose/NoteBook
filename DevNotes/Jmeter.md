---
layout: default
title: Jmeter
parent: DevNotes
nav_order: 6
---

# Jmeter
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Jmeter 基础
----------------------------------------------------

### 安装

环境-->系统变量

JMETER_HOME=D:\apache-jmeter-5.4.1

CLASSPATH=%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar; %JMETER_HOME%\lib\jorphan.jar; 

系统变量path后面加上%JMETER_HOME%\bin


###  命令压测：
```bash
jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]
```
HEAP="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"

参数说明

- -h ： –help -> prints usage information and exit
- -n ： –nongui -> 非图形模式允许；
- -t ： –testfile <argument> -> jmeter 测试脚本文件(.jmx) 路径，默认是当前路径；
- -l ： –logfile <argument> 运行结果保存路径（.jtl或.csv）这里后缀可以是jtl或csv，但文件内容格式必须要是csv格式 ;
- -e ： 在脚本运行结束后生成html报告
- -o ： 保存html报告的路径, 此文件夹必须为空或者不存在。注意：每次启动命令之前，文件夹内容必须和 jtl 文件一起清空

示例 1：
```bash
jmeter -n -t Pay-Station-postApplication.jmx -l postApplication.jtl
```
示例 2：
```bash
jmeter -n -t /opt/Pay-Station-postApplication.jmx -l /opt/postApplication.jtl -e -o /opt/test
```

打开已经生成的日志

基本命令格式：
jmeter -g -o
样例：
```bash
jmeter -g D:\apache-jmeter-3.0\bin\testLogFile -o ./output 

jmeter -g D:\Jemeter-work\postapplication_tps_ctg_800.jtl  -o  D:\Jemeter-work\postapplication_tps_ctg_80

jmeter -g D:\Jemeter-work\takeapplication_tps_2000.jtl  -o  D:\Jemeter-work\takeapplication_tps_2000

jmeter -g D:\Jemeter-work\postapplication_tps_2000.jtl  -o  D:\Jemeter-work\postapplication_tps_2000

jmeter -g D:\Jemeter-work\postapplication_tps_2006.jtl  -o  D:\Jemeter-work\postapplication_tps_2006

```


### 测试技巧

插件管理插件：jmeter-plugins-manager-1.6.jar
放 `D:\apache-jmeter-5.4.1\lib\ext` 目录


#### 普通线程组

https://www.cnblogs.com/hjhsysu/p/9189897.html

线程数：n

Ramp-Up Period：T （有人称之为启动时间，有人说是准备时长，看个人喜好）

循环次数：a  

若每个循环运行时间是 t

当时间到 S = (T- T/n)时，最后一个线程启动，若要使所有线程同时运作，则需要在最后一个线程启动的时候第一个线程仍未关闭，为达到这个要求，需满足 a·t > S及a > S/t

每一个个线程运行时间既是R = a·t(此处的a是大于S/t的某一值)，则第一个线程在时间点为R 的时候停止，整个测试理论运行时间则是 ：S + R = (1-1/n)·T + a·t

1、运行数次，确定单个请求实际平均执行时间

t = 0.045 (秒)
假设 T=10

2、如果线程数是n = 100，得到 S = (T- T/n) = 99.9 ，也就是说，从第一个线程启动到第100秒的时候，最后一个线程开始启动，若需要在最后一个线程启动的时候第一个线程仍未关闭，则需要满足 a·t > S ，已知S = 99.9, t = 0.045，得到 a > 2220 。



#### 并发线程组

阶梯式增加并发

Concurrency Thread Group参数讲解

-  Target Concurrency：目标并发（线程数）
-  Ramp Up Time：启动时间；若设置 1 min，则目标线程在1 imn内全部启动
-  Ramp-Up Steps Count：阶梯次数；若设置 6 ，则目标线程在 1min 内分六次阶梯加压（启动线程）；每次启动的线程数 = 目标线程数 / 阶梯次数 = 60 / 6 = 10
-  Hold Target Rate Time：持续负载运行时间；若设置 2 ，则启动完所有线程后，持续负载运行 2 min，然后再结束
-  Time Unit：时间单位（分钟或者秒）
-  Thread Iterations Limit：线程迭代次数限制（循环次数）；默认为空，理解成永远，如果运行时间到达Ramp Up Time + Hold Target Rate Time，则停止运行线程【不建议设置该值】
-  Log Threads Status into File：将线程状态记录到文件中（将线程启动和线程停止事件保存为日志文件）；

特别注意点

-  Target Concurrency只是个期望值，实际不一定可以达到这个并发数，得看上面的配置【电脑性能、网络、内存、CPU等因素都会影响最终并发线程数】
-  Jmeter会根据Target Concurrency的值和当前处于活动状态的线程数来判断当前并发线程数是否达到了Target Concurrency；若没有，则会不断启动线程，尽力让并发线程数达到Target Concurrency的值

Concurrency Thread Group和Stepping Thread Group的区别
官方说法

- Stepping Thread Group不提供设置启动延迟时间，阶梯增压过渡时间，阶梯释放过渡时间，但Concurrency Thread Group提供
- Stepping Thread Group可以阶梯释放线程，而Concurrency Thread Group是瞬时释放（具体看下面介绍）
- Stepping Thread Group设置了需要启动多少个线程就会严格执行，Concurrency Thread Group会尽力启动线程达到Target Concurrency值


通俗点理解

- Stepping Thread Group 是手动场景：测试过程，按照设定好的步骤执行
- Concurrency Thread Group 是目标场景：达到某个目标运行场景，测试过程不可控，动态变化



生成

```bash
jmeter -g D:\Jemeter-work\postapplication_tps_ctg_800.jtl  -o  D:\Jemeter-work\postapplication_tps_ctg_80

jmeter -g D:\Jemeter-work\takeapplication_tps_1000.jtl -o  D:\Jemeter-work\takeapplication_tps_1000

jmeter -g D:\Jemeter-work\takeapplication_tps_2003.jtl -o  D:\Jemeter-work\takeapplication_tps_2003

jmeter -g D:\Jemeter-work\consumeApplication_tps_2000.jtl -o  D:\Jemeter-work\consumeApplication_tps_2000
```


### 报错

Jmeter高并发压测时 java.net.SocketException: Connection reset 报错解决方案

新建txt,保存以下脚本修改后缀为reg文件，编辑值如下，保存后双击执行；重启电脑，再次压测即不会出现报错
解析中值为10进制，下方脚本已全转换为16进制:
```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\]
"MaxUserPort"=dword:fffe
"TcpTimedWaitDelay"=dword:1e
"TcpNumConnections"=dword:fffffe
"MaxFreeTcbs"=dword:7D0
"MaxHashTableSize"=dword:10000
```


相关值解析
   - MaxUserPort：最大动态端口数（Default = 5000, Max = 65534）
   - （Default = 5000, Max = 65534）
   - TcpTimedWaitDelay：TCP等待延迟时间（30）
   - TcpNumConnections：TCP连接数
   - MaxFreeTcbs：最大TCP控制块（1000-2000）
   - MaxHashTableSize：最大TCB Hash table数量（64-65536）


## JMeter压力测试

### 0、知识准备

概念：

吞吐率（Requests per second）
   概念：服务器并发处理能力的量化描述，单位是`reqs/s`（每秒处理请求数），指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。

计算公式：总请求数 / 处理完成这些请求数所花费的时间，即
   Request per second = Complete requests / Time taken for tests

并发连接数（The number of concurrent connections）
   概念：某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

并发用户数（The number of concurrent users，Concurrency Level）
   概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

用户平均请求等待时间（Time per request）
   计算公式：处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即
   Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

服务器平均请求等待时间（Time per request: across all concurrent requests）
   计算公式：处理完成所有请求数所花费的时间 / 总请求数，即
   Time taken for / testsComplete requests
   可以看到，它是吞吐率的倒数。
   同时，它也=用户平均请求等待时间/并发用户数，即
   Time per request / Concurrency Level

压力测试中的指标

**TPS**

TPS 即Transactions Per Second的缩写，每秒处理的事务数目。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程**(完整处理，即客户端发起请求到得到响应)**。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数，最终利用这些信息作出的评估分。一个事务可能对应多个请求，可以参考下数据库的事务操作。
**QPS**

QPS 即Queries Per Second的缩写，每秒能处理查询数目(完整处理，即客户端发起请求到得到响应)。是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。
我们从它的英文全名可以得出它是查询意思，原来在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。对应fetches/sec，即每秒的响应请求数。 虽然名义上是查询的意思，但实际上，现在习惯于对单一接口服务的处理能力用QPS进行表述（即使它并不是查询操作）。

**平均处理时间（RT）**

RT：响应时间，处理一次请求所需要的平均处理时间。
我们一般还会关注90%请求的的平均处理时间，因为可能因网络情况出现极端情况。

**并发用户数(并发量)**

每秒对待测试接口发起请求的用户数量。

**换算关系**

QPS = 并发数/平均响应时间

并发量 = QPS * 平均响应时间



比如3000个用户(并发量)同时访问待测试接口，在用户端统计，3000个用户平均得到响应的时间为1188.538 ms。所以QPS=3000/1.188538 s= 2524.11 q/s。
我们就可以这样描述本次测试，在3000个并发量的情况下，QPS为2524.11，平均响应事件为1188.538 ms

**TPS和QPS的区别**

这个问题开始，我认为这两者应该是同一个东西,但在知乎上看到他们的英文名，现在我认为：
QPS 每秒能处理查询数目，但现在一般也用于单服务接口每秒能处理请求数。
TPS 每秒处理的事务数目，如果完成该事务仅为单个服务接口，我们也可以认为它就是QPS。

PS:还有一个RPS的的概念 request per second 。每秒请求数，在一定条件下和QPS 和TPS类似。 

 一个系统的吞度量（承压能力）与request对CPU的消耗、外部接口、IO等等紧密关联。

单个reqeust 对CPU消耗越高，外部系统接口、IO影响速度越慢，系统吞吐能力越低，反之越高。

**吞吐量要素：**

系统吞吐量几个重要参数：QPS（TPS）、并发数、响应时间

一个系统吞吐量通常由QPS（TPS）、并发数两个因素决定，每套系统这两个值都有一个相对极限值，在应用场景访问压力下，只要某一项达到系统最高值，系统的吞吐量就上不去了，如果压力继续增大，系统的吞吐量反而会下降，原因是系统超负荷工作，上下文切换、内存等等其它消耗导致系统性能下降。

决定系统响应时间要素

我们做项目要排计划，可以多人同时并发做多项任务，也可以一个人或者多个人串行工作，始终会有一条关键路径，这条路径就是项目的工期。

系统一次调用的响应时间跟项目计划一样，也有一条关键路径，这个关键路径是就是系统影响时间；

关键路径是有CPU运算、IO、外部系统响应等等组成。

**系统吞吐量评估：**

我们在做系统设计的时候就需要考虑CPU运算、IO、外部系统响应因素造成的影响以及对系统性能的初步预估。

而通常境况下，我们面对需求，我们评估出来的出来QPS、并发数之外，还有另外一个维度：日PV。

通过观察系统的访问日志发现，在用户量很大的情况下，各个时间周期内的同一时间段的访问流量几乎一样。比如工作日的每天早上。只要能拿到日流量图和QPS我们就可以推算日流量。

通常的技术方法：

1. 找出系统的最高TPS和日PV，这两个要素有相对比较稳定的关系（除了放假、季节性因素影响之外）

2. 通过压力测试或者经验预估，得出最高TPS，然后跟进1的关系，计算出系统最高的日吞吐量。B2B中文和淘宝面对的客户群不一样，这两个客户群的网络行为不应用，他们之间的TPS和PV关系比例也不一样。



### 1、安装

环境-->系统变量

`JMETER_HOME` = `D:\apache-jmeter-5.4.1`

`CLASSPATH` = `%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar; %JMETER_HOME%\lib\jorphan.jar; `

系统变量 `path` 后面加上`%JMETER_HOME%\bin`



### 2、安装插件

有些插件可以图形化展示数据，所以看着比较直观，还有一些特殊的线程组，监听器，处理器，元件等。比如需要使用阶梯式加压就有可以使用并发线程组 ` Concurrency Thread Group` 。

插件管理插件：`jmeter-plugins-manager-1.6.jar`
放 `D:\apache-jmeter-5.4.1\lib\ext` 目录

安装插件管理工具之后可以根据需要安装插件即可。

双击`jmeter.bat`重启`JMeter`，找到"options"--"Plugins Manager"，点开之后就有可以选择的插件了，勾选需要安装的插件，然后点击右下角应用选择并重启（ "Apply Changes and Restart JMeter"）的按钮。

如果不习惯英文，可以在"options"--"Choose Language"--"Chinese(Simplified)"，就可以显示中文了。

### 3、编辑测试脚本

双击`jmeter.bat` 可以启动`JMeter GUI` 界面，方便编辑脚本。

点击测试计划，右键--添加--线程（模拟用户）



#### 3.1 普通线程组

https://www.cnblogs.com/hjhsysu/p/9189897.html

线程数：n

Ramp-Up Period：T （有人称之为启动时间，有人说是准备时长，看个人喜好）

循环次数：a  

若每个循环运行时间是 t

当时间到 S = (T- T/n)时，最后一个线程启动，若要使所有线程同时运作，则需要在最后一个线程启动的时候第一个线程仍未关闭，为达到这个要求，需满足 a·t > S及a > S/t

每一个个线程运行时间既是R = a·t(此处的a是大于S/t的某一值)，则第一个线程在时间点为R 的时候停止，整个测试理论运行时间则是 ：S + R = (1-1/n)·T + a·t

1、运行数次，确定单个请求实际平均执行时间

t = 0.045 (秒)
假设 T=10

2、如果线程数是n = 100，得到 S = (T- T/n) = 99.9 ，也就是说，从第一个线程启动到第100秒的时候，最后一个线程开始启动，若需要在最后一个线程启动的时候第一个线程仍未关闭，则需要满足 a·t > S ，已知S = 99.9, t = 0.045，得到 a > 2220 。



#### 3.2 并发线程组

阶梯式增加并发

Concurrency Thread Group参数讲解

-  Target Concurrency：目标并发（线程数）
-  Ramp Up Time：启动时间；若设置 1 min，则目标线程在1 min内全部启动
-  Ramp-Up Steps Count：阶梯次数；若设置 6 ，则目标线程在 1 min 内分六次阶梯加压（启动线程）；每次启动的线程数 = 目标线程数 / 阶梯次数 = 60 / 6 = 10
-  Hold Target Rate Time：持续负载运行时间；若设置 2 ，则启动完所有线程后，持续负载运行 2 min，然后再结束
-  Time Unit：时间单位（分钟或者秒）
-  Thread Iterations Limit：线程迭代次数限制（循环次数）；默认为空，理解成永远，如果运行时间到达Ramp Up Time + Hold Target Rate Time，则停止运行线程【不建议设置该值】
-  Log Threads Status into File：将线程状态记录到文件中（将线程启动和线程停止事件保存为日志文件）；

特别注意点

-  Target Concurrency只是个期望值，实际不一定可以达到这个并发数，得看上面的配置【电脑性能、网络、内存、CPU等因素都会影响最终并发线程数】
-  Jmeter会根据Target Concurrency的值和当前处于活动状态的线程数来判断当前并发线程数是否达到了Target Concurrency；若没有，则会不断启动线程，尽力让并发线程数达到Target Concurrency的值

Concurrency Thread Group和Stepping Thread Group的区别
官方说法

- Stepping Thread Group不提供设置启动延迟时间，阶梯增压过渡时间，阶梯释放过渡时间，但Concurrency Thread Group提供
- Stepping Thread Group可以阶梯释放线程，而Concurrency Thread Group是瞬时释放（具体看下面介绍）
- Stepping Thread Group设置了需要启动多少个线程就会严格执行，Concurrency Thread Group会尽力启动线程达到Target Concurrency值


通俗点理解

- Stepping Thread Group 是手动场景：测试过程，按照设定好的步骤执行
- Concurrency Thread Group 是目标场景：达到某个目标运行场景，测试过程不可控，动态变化



#### 3.3 取样器

选好线程组之后，就可以继续右键添加取样器，比较常见的是HTTP请求。

然后就是在基本里填写：协议、IP、端口、请求、路径、参数等，我测的时候用XML，放消息体数据；在基本里填写：客户端实现 HttpClient4，超时时间一般10-60s，可以自己根据实际情况调整。

如果HTTP头需要设置，右键添加配置元件--**HTTP信息头管理器**，添加对应的键值对即可。

压力测试需要大量请求，如果需要可变参数，可以在线程组下添加"配置元件"--"CVS Data Set Config"，然后可以读取文件里准备好的参数，没有严格限制文件格式，可以是cvs、txt

比如：

```
Filename 文件名称：D:/Jemeter-work/crpt_no_result/crpt_no_20210607.txt
File encoding 文件编码：（根据需要设置，可以不填，一般为UTF-8）
Variable Names(comma-delimited)变量名：applyno (这里相当于给要取的参数取个变量名)
忽略首行：False
分隔符：,
是否允许带引号？:  False
遇到文件结束符再次循环?：True
遇到文件结束符停止线程?：False
线程共享模式： 所有线程
```

然后在请求使用变量的地方获取变量参数即可，比如我`PAY_APP_NO`节点需要动态取`applyno`

```xml
<?xml version="1.0" encoding="utf-8"?>
<PACKET TYPE="REQUEST"> 
  <HEAD> 
    <TRAN_CODE>10</TRAN_CODE>  
    <USER>E2088101568349711</USER>
    <PASSWORD>123456</PASSWORD> 
  </HEAD>  
  <BODY> 
    <BASE> 
      <BANK_CODE>00000001</BANK_CODE>  
      <INSURE_ID>00000001</INSURE_ID>  
      <MIDNO>105370663006008</MIDNO>  
      <TIDNO>0078</TIDNO>  
      <BK_ACCT_DATE>20150827</BK_ACCT_DATE>  
      <BK_ACCT_TIME>135610</BK_ACCT_TIME>  
      <BK_SERIAL>348303</BK_SERIAL>  
      <BK_TRAN_CHNL>POS</BK_TRAN_CHNL>  
      <PAY_APP_NO>${applyno}</PAY_APP_NO>  
      <CHECKCODE>5736</CHECKCODE> 
    </BASE> 
  </BODY> 
</PACKET>
```

这里`crpt_no_20210607.txt`的内容：

```txt
5780051517784064
5780051408732160
5780051299680256
5780051182239744
```

我这里只有一列，如果有多列，变量名可以定义多个，使用的时候根据需要取值即可。



#### 3.4 提取器

 提取器属于处理器比较多，处理器一般分前置处理器和后置处理器。

这里使用到提取器，是因为后一次请求执行参数依赖前一次的执行结果，比较常见的有2种提取器。

- JSON 提取器
- 正则表达式提取器

JSON提取器



**正则表达式提取器**

点击我们使用的取样器，点击右键添加--后置处理器--正则表达式提取器

设置：

要检查的响应字段：主体或body，可以根据需要选择。

引用名字：applyno

正常表达式：`<PAY_APP_NO>(.*?)</PAY_APP_NO>`

模板：$1$

匹配数字：1

缺省值：就是取不到的时候给个默认值



配置完正则表达式提取器之后，后一个取样器的请求参数里就可以直接使用 `${applyno}`获取刚才返回的结果。



#### 3.5 Bean Shell

##### 3.5.1 基本概念

官网:[http://www.BeanShell.org/](http://www.beanshell.org/)

- BeanShell是一种完全符合Java语法规范的脚本语言,并且又拥有自己的一些语法和方法;
- BeanShell是一种松散类型的脚本语言(这点和JS类似);
- BeanShell是用Java写成的,一个小型的、免费的、可以下载的、嵌入式的Java源代码解释器,具有对象脚本语言特性,非常精简的解释器jar文件大小为175k。
- BeanShell执行标准Java语句和表达式,另外包括一些脚本命令和语法。

##### 3.5.2 **Jmeter的Bean Shell**

- 定时器：　　BeanShell Timer
- 前置处理器：BeanShell PreProcessor
- 采样器：　　BeanShell Sampler
- 后置处理器：BeanShell PostProcessor
- 断言：　　　BeanShell断言
- 监听器：　　BeanShell Listener

##### 3.5.3 Bean Shell常用内置变量

　　 JMeter在它的BeanShell中内置了变量，用户可以通过这些变量与JMeter进行交互，其中主要的变量及其使用方法如下:

- **log**：写入信息到jmeber.log文件，使用方法：log.info(“This is log info!”);
- **ctx**：该变量引用了当前线程的上下文，使用方法可参考：[org.apache.jmeter.threads.JMeterContext](http://jmeter.apache.org/api/org/apache/jmeter/threads/JMeterContext.html)。
- **vars** - (JMeterVariables)：操作jmeter变量，这个变量实际引用了JMeter线程中的局部变量容器（本质上是Map），它是测试用例与BeanShell交互的桥梁，常用方法：

　　　　a) vars.get(String key)：从jmeter中获得变量值

　　　　b) vars.put(String key，String value)：数据存到jmeter变量中

　　　　更多方法可参考：[org.apache.jmeter.threads.JMeterVariables](http://jmeter.apache.org/api/org/apache/jmeter/threads/JMeterVariables.html)

- **props** - (JMeterProperties - class  java.util.Properties)：操作jmeter属性，该变量引用了JMeter的配置信息，可以获取Jmeter的属性，它的使用方法与vars类似，但是只能put进去String类型的值，而不能是一个对象。对应于java.util.Properties。 

　　　　a) props.get("START.HMS");　　注：START.HMS为属性名，在文件jmeter.properties中定义 

　　　　b) props.put("PROP1","1234"); 

- **prev** - (SampleResult)：获取前面的sample返回的信息，常用方法：

　　　　a) getResponseDataAsString()：获取响应信息

　　　　b) getResponseCode() ：获取响应code

　　　　更多方法可参考：[org.apache.jmeter.samplers.SampleResult](http://jmeter.apache.org/api/org/apache/jmeter/samplers/SampleResult.html)

- sampler - (Sampler)：gives access to the current sample

##### 3.5.4 Bean Shell 使用

**场景1**，我们需要将取样器执行的结果保存到文件

（1）点击取样器，点击右键添加--后置处理器--正则表达式提取器。这里使用3.4里的取样器，取applyno，保存起来。

（2）点击我们使用的取样器，点击右键添加--后置处理器--Bean Shell PostProcessor后置处理程序

填写脚本：

```java
FileWriter writer = new FileWriter("D:\\Jemeter-work\\crpt_no_result\\crpt_no_20210607.txt", true);
BufferedWriter out = new BufferedWriter(writer);
out.write(vars.get("applyno")+"\n");
out.close();
writer.close();
```

（3）测试请求，如果看到对应的文件里有取到返回数据就说明正常。

**场景2**，我们某个取样器请求时需要指定规则生成的ID进行请求取样。

（1）使用开发工具写好ID生成的工具类，打包成jar，

（2）把jar包放到jmeter目录`D:\apache-jmeter-5.4.1\lib\ext`目录下

（3）重启 Jmeter，点击我们使用的取样器，点击右键添加--后置处理器--Bean Shell PreProcessor前置处理程序，添加脚本

```
import com.xiaocai.utils.IdGenerator;

String id = new IdGenerator().generate();
log.info("----自定义生成ID----" + id);
//把值保存到jmeter变量idparam中
vars.put("idparam",id);
```

#### 3.6 监听器

监听器比较常用的是查看结果树、聚合报告，如果安装的插件还可以选中查看TPS曲线的`jp@gc - Transactions per Second` 、查看活动线程时序图`jp@gc - Active Threads Over Time`、查看响应时间的时序图`jp@gc - Response Times Over Time`.

选中线程组--右键--监听器，选中需要的监听器就可以，一般不需要设置，如果要保存执行结果，可以保存指定一下文件路径即可，格式一般`jtl`、`cvs`均可。



#### 3.7 断言

jmeter断言常用有两种，一种是响应断言，一种是响应时间断言，如果响应内容不满足断言的配置，则认为这次的请求是失败的。

- 响应断言：判断响应内容是否包含指定的字符信息，用于判断api接口返回内容是否正确。

- 响应时间断言：判断响应时间，是否超过预期的时间，用于判断api接口返回时间是否超过预期。

 断言配置

（1）点击取样器，点击右键添加--断言--响应断言

正常返回code值为200，如果接口返回code值不是200表示接口异常，为了测试，这里修改为接口返回code值不为999则表示访问失败。

（2）点击取样器，点击右键添加--断言--响应时间断言

设置断言持续时间为1ms。

正常返回需要30ms,如果返回超过1ms，表示请求接口失败。

现在配置了2个断言，发起http请求，由于返回内容code值不为999，以及访问时间超过1毫秒，所以认为访问失败。

#### 3.8 变量参数化配置

不管是多个线程组还是多个取样器，有时候需要修改IP，或者访问接口的时候，就需要修改，如果多出使用就可以进行变量参数化，每次修改改一个地方就可以。

选择线程组，右键添加--配置元件--用户定义的变量，比如定义IP，端口，接口路径，字符集等一些公共的变量均可以在用户定义变量里定义。

比如：

| 名称    | 值             |
| ------- | -------------- |
| ip      | 192.168.10.184 |
| port    | 8086           |
| charset | uft-8          |

在使用的地方：${ip}，${port}，${charset}直接取值使用即可。



###  4、命令压测

> 使用命令压测的前提是要编辑好测试脚本 `.jmx` 文件。

```bash
jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]
```
HEAP="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"

参数说明

- -h ： –help -> prints usage information and exit
- -n ： –nongui -> 非图形模式允许；
- -t ： –testfile <argument> -> jmeter 测试脚本文件(.jmx) 路径，默认是当前路径；
- -l ： –logfile <argument> 运行结果保存路径（.jtl或.csv）这里后缀可以是jtl或csv，但文件内容格式必须要是csv格式 ;
- -e ： 在脚本运行结束后生成html报告
- -o ： 保存html报告的路径, 此文件夹必须为空或者不存在。注意：每次启动命令之前，文件夹内容必须和 jtl 文件一起清空

示例 1：
```bash
jmeter -n -t Pay-Station-postApplication.jmx -l postApplication.jtl
```
示例 2：
```bash
jmeter -n -t /opt/Pay-Station-postApplication.jmx -l /opt/postApplication.jtl -e -o /opt/test
```

### 5、生成报告

打开已经生成的日志

基本命令格式：

```bash
jmeter -g -o
```

样例

```cmd
jmeter -g D:\apache-jmeter-3.0\bin\testLogFile -o ./output 

jmeter -g D:\Jemeter-work\postapplication_tps_ctg_800.jtl  -o  D:\Jemeter-work\postapplication_tps_ctg_80

jmeter -g D:\Jemeter-work\postapplication_tps_ctg_800.jtl  -o  D:\Jemeter-work\postapplication_tps_ctg_80

jmeter -g D:\Jemeter-work\takeapplication_tps_1000.jtl -o  D:\Jemeter-work\takeapplication_tps_1000
```



### 6、分布式压测

在工作中使用jmeter做大并发压力测试的场景下，单机受限内存、CPU、网络IO，会出现服务器压力还没有上去，但是压测服务器已经由于模拟的压力太大死机了。为了让jmeter工具提供更强大的负载能力，jmeter提供了多台机器同时产生负载的机制。

基本架构：由一台Server控制机，指挥多个agent压力机执行，并且收集压测结果。

**分布式环境压力服务器要求：**
 　需要server（控制机）和agent（压力机），agent搭建在linux（centos 6.5）服务器环境下，server搭建在windows（server 2012）环境下。
 　压力测试瓶颈大都在带宽上面，需要保证压力机的带宽要比服务器的带宽高，不然压力上不去。
 　需要保证agent和server都在一个网络中，且在多网卡环境需要保证启动的网卡都在一个网段。
 　需要保证server和agent之间的时间同步。
 　关闭防火墙。

#### 6.1 环境搭建

windows部署 

Server机一般会用windows，基本安装前面有。

（1）部署jdk环境,配置path变量
 （2）直接去官网下载最新的二进制源码包即可。
 （3）解压jmeter到指定目录，设置path变量，安装完成之后，在命令行运行jmeter命令，如果可以正常启动jmeter，说明环境配置ok。

 Linux部署jmeter

（1）下载安装

 ```bash
wget http://mirrors.tuna.tsinghua.edu.cn/apache//jmeter/binaries/apache-jmeter-5.4.1.zip
unzip apache-jmeter-5.4.1.zip -d /usr/local/
cd /usr/local/
ln -s apache-jmeter-5.4.1/ jmeter
 ```

（2）配置启动脚本

```bash
　　#!/bin/bash
　　# chkconfig: 345 26 74
　　# description: jmeter agent
　　myip=`ifconfig eth0 |awk '/inet addr/{gsub(/addr:/,"");print $2}'`
　　cmd="/usr/local/jmeter/bin/jmeter-server -Djava.rmi.server.hostname=$myip"
　　start(){
　　  $cmd &
　　}
　　 
　　stop(){
　　    jmeter_pid=`ps aux | grep jmeter-server | grep -v grep | awk '{print $2}'`
　　    for pid in $jmeter_pid;do
　　    kill -9 $pid
　　    done
　　}
　　 
　　act=$1
　　case $act in
　　 'start')
　　   start;;
　　 'stop')
　　   stop;;
　　 'restart')
　　   stop
　　   sleep 2
　　   start;;
　　  *)
　　   echo '[start|stop|restart]';;
　　esac
```

3）启动jmeter agent服务，验证是否监听1099端口

```bash
/etc/init.d/jmeter-agent start
netstat -lntp | grep 1099
```

#### 6.2 分布式环境配置

（1）确保server和agnet安装正确。
（2）Agent启动，并监听1099端口。
（3）在server机器的jmeter安装目录下bin目录下，找到properties文件，修改远程主机选项，添加3个agent服务器的地址。

```properties
remote_hosts=192.168.10.184:1099,192.168.10.185:1099,192.168.10.186:1099
```

 （4）启动jmeter server，多网卡模式需要指定IP地址启动
 　`jmeter -Djava.rmi.server.hostname=192.168.10.61`
 （5）验证分布式环境是否搭建成功

​		5.1）、jmeter启动之后在运行--远程启动中,会出现添加的远程主机列表

​		5.2）、使用jmeter 的server机器发起一次请求

待续...


