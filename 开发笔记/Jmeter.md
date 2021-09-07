Jmeter 
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
