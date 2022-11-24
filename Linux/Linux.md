---
layout: default
title: Linux
has_children: true
nav_order: 11
---

# Linux
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Linux 环境相关

## grep

参数：
```
参数：

    -a 或 --text : 不要忽略二进制的数据。
    -A<显示行数> 或 --after-context=<显示行数> : 除了显示符合范本样式的那一列之外，并显示该行之后的内容。
    -b 或 --byte-offset : 在显示符合样式的那一行之前，标示出该行第一个字符的编号。
    -B<显示行数> 或 --before-context=<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前的内容。
    -c 或 --count : 计算符合样式的列数。
    -C<显示行数> 或 --context=<显示行数>或-<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前后的内容。
    -d <动作> 或 --directories=<动作> : 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
    -e<范本样式> 或 --regexp=<范本样式> : 指定字符串做为查找文件内容的样式。
    -E 或 --extended-regexp : 将样式为延伸的正则表达式来使用。
    -f<规则文件> 或 --file=<规则文件> : 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。
    -F 或 --fixed-regexp : 将样式视为固定字符串的列表。
    -G 或 --basic-regexp : 将样式视为普通的表示法来使用。
    -h 或 --no-filename : 在显示符合样式的那一行之前，不标示该行所属的文件名称。
    -H 或 --with-filename : 在显示符合样式的那一行之前，表示该行所属的文件名称。
    -i 或 --ignore-case : 忽略字符大小写的差别。
    -l 或 --file-with-matches : 列出文件内容符合指定的样式的文件名称。
    -L 或 --files-without-match : 列出文件内容不符合指定的样式的文件名称。
    -n 或 --line-number : 在显示符合样式的那一行之前，标示出该行的列数编号。
    -o 或 --only-matching : 只显示匹配PATTERN 部分。
    -q 或 --quiet或--silent : 不显示任何信息。
    -r 或 --recursive : 此参数的效果和指定"-d recurse"参数相同。
    -s 或 --no-messages : 不显示错误信息。
    -v 或 --invert-match : 显示不包含匹配文本的所有行。
    -V 或 --version : 显示版本信息。
    -w 或 --word-regexp : 只显示全字符合的列。
    -x --line-regexp : 只显示全列符合的列。
    -y : 此参数的效果和指定"-i"参数相同。 
```
常用实例：

我喜欢配合cat使用

```bash

## 查文件中指定含有指定关键字的内容
cat /home/test/app/logs/paystationlog/paystation.log | grep  17ACIC01220000000002871371146E

## 查文件中指定含有指定关键字的内容，并多显示有关键字内容附加前面10行
cat /home/test/app/logs/paystationlog/paystation.log | grep -B 10  17ACIC01220000000002871371146E

## 查文件中指定含有指定关键字的内容，并多显示有关键字内容附近后面10行
cat /home/test/app/logs/paystationlog/paystation.log | grep -A 10  17ACIC01220000000002871371146E

## 查文件中指定含有指定关键字的内容，并多显示有关键字内容附近前面及后面10行，多20行
cat /home/test/app/logs/paystationlog/paystation.log | grep -B 10 -A 10  17ACIC01220000000002871371146E
cat /home/test/app/logs/paystationlog/paystation.log | grep -C 10  17ACIC01220000000002871371146E
```



## 固定IP配置

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33 
```

```text
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV5_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=9cf4f495-18f1-4c56-80b5-d433c5273d37
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.80.81
GATEWAY=192.168.80.1
IPV6_PRIVACY=no
PREFIX=24
IPV4_FAILURE_FATAL=no

REERDNS=no
DNS1=192.168.80.1
DNS2=8.8.8.8
DNS3=223.6.6.6
DNS4=114.114.114.114
```

这个是网络配置参数：
 - BOOTPROTO=static 静态IP
 - BOOTPROTO=dhcp 动态IP
 - BOOTPROTO=none 无（不指定）

通常情况下是dhcp或者static,固定IP可以为 static或者none
 - ONBOOT ：是否开机

重启网络即可

```bash
service network restart
```


备份原始官方源

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

换源

```bash
# CentOS 6

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

# CentOS 7

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo    

# CentOS 8

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

刷新缓存

```bash
yum clean all  && yum makecache
```

更新系统

```bash
yum update
```


## wget
---------------------------

-bash: wget: command not found的两种解决方法
wget 时提示 -bash:wget command not found,很明显没有安装wget软件包。一般linux最小化安装时，wget不会默认被安装。

可以通过以下两种方法来安装：

1、rpm 安装

rpm 下载源地址：http://mirrors.163.com/centos/6.2/os/x86_64/Packages/

下载wget的RPM包：http://mirrors.163.com/centos/6.2/os/x86_64/Packages/wget-1.12-1.4.el6.x86_64.rpm

rpm ivh wget-1.12-1.4.el6.x86_64.rpm 安装即可。

如果客户端用的是SecureCRT,linux下没装rzsz 包时，rz无法上传文件怎么办？我想到的是安装另一个SSH客户端：SSH Secure Shell。然后传到服务器上安装，这个比较费劲，所以推荐用第二种方法，不过如果yum包也没有安装的话，那就只能用这种方法了。

2、yum安装

```bash
yum -y install wget
```
3、离线安装

gcc安装

m4 安装

- 下载: [http://mirrors.kernel.org/gnu/m4/m4-1.4.13.tar.gz](http://mirrors.kernel.org/gnu/m4/m4-1.4.13.tar.gz)
- 其他版本: [http://mirrors.kernel.org/gnu/m4/](http://mirrors.kernel.org/gnu/m4/)

```bash
cd m4-1.4.13
# 配置编译环境
./configure –prefix=/usr/local
# 编译并安装
make && make install
```
[未完待续]


## tree 安装

官网：http://mama.indstate.edu/users/ice/tree/

方法一，yum安装

```bash
yum install tree
```

方法二，源码安装

1.下载安装包，地址：http://mama.indstate.edu/users/ice/tree/


```bash
wget  http://mama.indstate.edu/users/ice/tree/src/tree-1.8.0.tgz
```

2.解压安装

```bash
# 解压
tar -zxvf tree-1.8.0.tgz

# 进入
cd tree-1.7.0

# 安装文件
make install

　　　　　　　　
# 使用
tree
tree --version

# 显示目录深度3
tree -L 3
# 只显示目录不显示文件
tree -d -L 3
```


tree命令行参数：

```
-a 显示所有文件和目录。
-A 使用ASNI绘图字符显示树状图而非以ASCII字符组合。
-C 在文件和目录清单加上色彩，便于区分各种类型。
-d 显示目录名称而非内容。
-D 列出文件或目录的更改时间。
-f 在每个文件或目录之前，显示完整的相对路径名称。
-F 在执行文件，目录，Socket，符号连接，管道名称名称，各自加上"*","/","=","@","|"号。
-g 列出文件或目录的所属群组名称，没有对应的名称时，则显示群组识别码。
-i 不以阶梯状列出文件或目录名称。
-I 不显示符合范本样式的文件或目录名称。
-l 如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录。
-n 不在文件和目录清单加上色彩。
-N 直接列出文件和目录名称，包括控制字符。
-p 列出权限标示。
-P 只显示符合范本样式的文件或目录名称。
-q 用"?"号取代控制字符，列出文件和目录名称。
-s 列出文件或目录大小。
-t 用文件和目录的更改时间排序。
-u 列出文件或目录的拥有者名称，没有对应的名称时，则显示用户识别码。
-x 将范围局限在现行的文件系统中，若指定目录下的某些子目录，其存放于另一个文件系统上，则将该子目录予以排除在寻找范围外。
```

## netstat 安装

---------------------------
centos7默认没netstat命令，需要安装

```bash
yum install net-tools 
```



## fuser命令

------------------------------

用指定的文件或者文件系统显示进程进程号，默认情况下每一个文件名后会跟着一个字母来表示类型

fuser命令需要安装
```bash
yum install psmisc 
```



## rzsz

---------------------------

lrzsz是一个unix通信套件提供的X，Y，和ZModem文件传输协议,可以用在windows与linux 系统之间的文件传输，体积小速度快。官网入口：https://ohse.de/uwe/software/lrzsz.html

1、在线安装

```bash
yum -y install lrzsz
```

上传命令
```bash
rz 
```

下载命令

```bash
sz log_web.log
```

2.离线安装

```bash
# 下载安装包 
wget http://down1.chinaunix.net/distfiles/lrzsz-0.12.20.tar.gz 
tar -zxvf lrzsz-0.12.20.tar.gz 
cd lrzsz-0.12.20 
# 编译 
./configure –prefix=/usr/local/lrzsz 
make 
make install 
# 建立软连接 
ln -s /usr/local/lrzsz/bin/lrz /usr/bin/rz 
ln -s /usr/local/lrzsz/bin/lsz /usr/bin/sz
```


## 系統监控 

项目地址：https://github.com/sysstat/sysstat

安装

```bash
yum install sysstat
```
启用


```shell
systemctl enable sysstat  
systemctl start sysstat  
```


## Firawalld

1、查看firewall服务状态

```bash
systemctl status firewalld
```

2、查看firewall的状态
```bash
firewall-cmd --state
```

3、开启、重启、关闭、firewalld.service服务

```bash
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
```

```bash
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp

# 开放80端口
firewall-cmd --permanent --add-port=80/tcp

# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload

firewall-cmd --list-ports
```

D:\idea-Work\XianDai-TrunkNew\cm-web\web\pages

参数解释

1、firwall-cmd：是Linux提供的操作firewall的一个工具；

2、--permanent：表示设置为持久；

3、--add-port：标识添加的端口；



## 查看端口

用于查看某一端口的占用情况

```bash
lsof -i:端口号 ，
```

比如查看8000端口使用情况:

```bash
lsof -i:8000
```

通过PID查看端口号：
```bash
netstat -anop|grep pid 
```



## 查看进程

```bash
ps -ef | grep  xxx
ps aux | grep  xxx

ps -ef | grep $APP_NAME | grep -v grep |awk '{print $2}'
```

## 空间清理

1.查看空间占用

```bash
df
# 显示G单位
df -h 
#显示 M 单位
df -m
```

2.定位最大文件目录

```bash
#进入根目录。
cd / 

#寻找当前目录，哪个文件夹占用空间最大
du -h --max-depth=1 
```
3.定位最大文件

```bash
# 将文件以从大到小顺序展现
ls –lhS 
```

4.确认文件未被占用

```bash
#确认删除文件是否被占用
/usr/sbin/lsof|grep deleted 
```
可用根据pid 杀死占用程序，或者正常暂时停止占用程序。



## 基础环境

jdk 和 maven

在线下：
```
mkdir -p /usr/local/maven/
cd /usr/local/maven/
wget https://dlcdn.apache.org/maven/maven-3/3.8.2/binaries/apache-maven-3.8.2-bin.tar.gz
```

上传 jdk 包 和 maven 包。
jdk解压到 /usr/local/java/
maven 解压到  /usr/local/maven/

修改全局配置:
```bash
cp /etc/profile /etc/profile.bak
vi /etc/profile
vi  .bash
```

在配置文件末尾添加

```text
export M2_HOME=/usr/local/maven/apache-maven-3.8.2
export JAVA_HOME=/usr/local/java/jdk1.8.0_212
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:${M2_HOME}/bin:$PATH
```

 

## tomcat 线程数

上线前可以压测一下，调整参数配置，查看tomcat线程数命令如下：

获取tomcat进程pid ：
```bash
ps -ef|grep tomcat
```
统计该tomcat进程内的线程个数 ：

```bash
ps -Lf 29295 |wc -l
```



## 应用重启脚本

```bash
#!/bin/sh
APP_NAME=bp-paystation-springboot-yace.jar
pid=`ps -ef | grep $APP_NAME | grep -v grep |awk '{print $2}'`
if [ $pid ]; then
    echo :App  is  running pid=$pid
    kill -9 $pid
fi

JMX_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=7969  -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.168.10.184"

nohup java ${JMX_OPTS} -jar $APP_NAME --server.port=8086 > log_paystation_yace.log 2>&1 &
```

## 安全加固

给web代理服务器补充header头配置，消除X-Frame-Options头缺失、HTTP Content-Security-Policy头缺失、X-Content-Type-Options头缺失三个风险。

1、apache 服务器，在 `http.conf` 文件末尾增加

```text
Header always append X-Frame-Options SAMEORIGIN
Header set X-Content-Type-Options  nosniff
Header set Content-Security-Policy "worker-src 'self';"
```

2、nginx 服务器，在 `nginx.conf` 文件对应的 http{} 块或 server{}块里末尾增加

```text
add_header  X-Frame-Options SAMEORIGIN;
add_header  X-Content-Type-Options  nosniff;
add_header  Content-Security-Policy "worker-src 'self';"
#启用XSS保护，并在检查到XSS攻击时，停止渲染页面
add_header  X-XSS-Protection "1; mode=block";
```
X-Frame-Options 有三个值:

- DENY 表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。
- SAMEORIGIN 表示该页面可以在相同域名页面的 frame 中展示。
- ALLOW-FROM uri 表示该页面可以在指定来源的 frame 中展示。

`X-Content-Type-Options  nosniff`则 script 和 styleSheet 元素会拒绝包含错误的 MIME 类型的响应。这是一种安全功能，有助于防止基于 MIME 类型混淆的攻击,通过设置"X-Content-Type-Options: nosniff"响应标头，对 script 和 styleSheet 在执行是通过MIME 类型来过滤掉不安全的文件.

spring boot支持EnableWebSecurity 这个anotation来设置不全的安全策略

```java
import com.alibaba.spring.websecurity.DefaultWebSecurityConfigurer;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.header.writers.frameoptions.WhiteListedAllowFromStrategy;
import org.springframework.security.web.header.writers.frameoptions.XFrameOptionsHeaderWriter;
 
import java.util.Arrays;
 
@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends DefaultWebSecurityConfigurer {
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
       //disable 默认策略。 这一句不能省。 
        http.headers().frameOptions().disable();
       //新增新的策略。 
        http.headers().addHeaderWriter(new XFrameOptionsHeaderWriter(
                new WhiteListedAllowFromStrategy(
                        Arrays.asList("http://itaobops.aliexpress.com", "https://cpp.alibaba-inc.com",
                                "https://pre-cpp.alibaba-inc.com"))));
    }
}
```
上面是支持ALLOW-FROM uri的设置方式。
```java
@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends DefaultWebSecurityConfigurer {
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
        http.headers().frameOptions().sameOrigin();
 
    }
}
```
去除x-frame-options header配置：
```java
@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends DefaultWebSecurityConfigurer {
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
        http.headers().frameOptions().disable();
    }
}
```



## v2Ray

参考： 
- https://hissin.cn/zheteng/795.html
- https://github.com/Loyalsoldier/v2ray-rules-dat
- https://yearliny.com/v2ray-complete-tutorial/
- https://www.linodovultr.com/post/resolve-v2ray-after-install-can-not-connect.html?replytocom=69
- https://github.com/search?q=v2ray
- https://fx.tmioe.com/654.html
- https://www.ddayh.com/1136.html
- https://ivu4e.com/toolbox/newwork/2019-10-16/259.html

```bash
yum -y install curl

bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)

Downloading V2Ray archive: https://github.com/v2fly/v2ray-core/releases/download/v4.40.1/v2ray-linux-64.zip

ExecStart=/usr/local/bin/v2ray -config /usr/local/etc/v2ray/config.json

systemctl enable v2ray

systemctl start v2ray
```

```bash

yum -y install wget 

cd /opt

wget https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh

wget https://github.com/v2fly/v2ray-core/releases/download/v4.40.1/v2ray-linux-64.zip

wget https://github.com/v2fly/v2ray-core/releases/download/v4.40.1/v2ray-linux-64.zip.dgst


chmod +x install-release.sh


```
V2Ray 配置文件路径：/etc/v2ray/config.json

V2ray 常用管理命令
```bash
v2ray info     # 查看 V2Ray 配置信息
v2ray config   # 修改 V2Ray 配置
v2ray link     # 生成 V2Ray 配置文件链接
v2ray infolink # 生成 V2Ray 配置信息链接
v2ray qr       # 生成 V2Ray 配置二维码链接
v2ray ss       # 修改 Shadowsocks 配置
v2ray ssinfo   # 查看 Shadowsocks 配置信息
v2ray ssqr     # 生成 Shadowsocks 配置二维码链接
v2ray status   # 查看 V2Ray 运行状态
v2ray start    # 启动 V2Ray
v2ray stop     # 停止 V2Ray
v2ray restart  # 重启 V2Ray
v2ray log         #  查看 V2Ray 运行日志
v2ray update      #  更新 V2Ray
v2ray update.sh   # 更新 V2Ray 管理脚本
v2ray uninstall   # 卸载 V2Ray 
```

配置

```bash
# 检测
/usr/local/bin/v2ray --test  --config /usr/local/etc/v2ray/config.json

netstat -apn | grep v2ray
```
(1)生成ID

原先的脚本，会自动配置ID和生成config.json文件，现在这个脚本，需要自己配置config.json文件，首先获取用户ID：

运用指令：
```bash
cat /proc/sys/kernel/random/uuid
```
创建一个用户 id ，

并记住这个id号；

98ee3c11-fe5b-4e57-a2ca-882b10c7b402

(2)配置文件

配置文件路径为/usr/local/etc/v2ray/config.json 可以使用vi指令打开文本进行配置。

直接复制我上面的配置即可使用，id就是上面第二步获取的用户id，此时端口是2088 ID是 98ee3c11-fe5b-4e57-a2ca-882b10c7b402  


```JSON
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "98ee3c11-fe5b-4e57-a2ca-882b10c7b402", 
            "alterId": 66
          }
        ]
      }
    },
    {
      "port": 444,  
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-128-gcm",  
        "password": "small.CAI"  
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

## vm centos 扩容


1.查看磁盘空间占用
df -m

2.查看新磁盘的分区信息

fdisk -l

3.对新加的磁盘进行分区操作（选择新增的磁盘），并回车

fdisk  /dev/sda 

```
[root@localhost ~]# fdisk -l

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

磁盘 /dev/mapper/centos-root：29.0 GB, 28982640640 字节，56606720 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

[root@localhost ~]# fdisk  /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：
```


4.输入“p”命令，回车，查看已分区的数量。

```
[root@localhost ~]# fdisk -l

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

磁盘 /dev/mapper/centos-root：29.0 GB, 28982640640 字节，56606720 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

[root@localhost ~]# fdisk  /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：p

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

命令(输入 m 获取帮助)：
```

5.接着输入“n”命令，回车，新增一个分区。

```

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

命令(输入 m 获取帮助)：n
```

6.接着，在终端命令窗口中输入“p”命令,回车，在打开的分区号命中使用默认的分区号，并回车。

```
磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
分区号 (3,4，默认 3)：3

```

7、接着，在显示的起始扇区直接敲回车键，在弹出的Last后面中，直接敲回车键即可进入下一步（在如图画框位置直接使用回车即可进入下一步）。

```
磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
分区号 (3,4，默认 3)：3
起始 扇区 (62914560-83886079，默认为 62914560)：
将使用默认值 62914560
Last 扇区, +扇区 or +size{K,M,G} (62914560-83886079，默认为 83886079)：
将使用默认值 83886079
分区 3 已设置为 Linux 类型，大小设为 10 GiB

命令(输入 m 获取帮助)：
```

 8、接着，继续在终端命令窗口中输入“p”回车，查看当前分区的信息，在显示的信息中就可以看到已经多了一个新的分区了，大小就是刚设置的大小。

 ```
 命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
分区号 (3,4，默认 3)：3
起始 扇区 (62914560-83886079，默认为 62914560)：
将使用默认值 62914560
Last 扇区, +扇区 or +size{K,M,G} (62914560-83886079，默认为 83886079)：
将使用默认值 83886079
分区 3 已设置为 Linux 类型，大小设为 10 GiB

命令(输入 m 获取帮助)：p

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM
/dev/sda3        62914560    83886079    10485760   83  Linux
```

9、接着，在终端命令中输入“w”命令，回车，写入磁盘信息并保存。

```
磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000bf0ed

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM
/dev/sda3        62914560    83886079    10485760   83  Linux

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。
[root@localhost ~]# 
```

10、接着，在终端命令窗口中，输入“# reboot”回车，重启虚拟机格式化新建分区。

11、接着，等待系统自动重启完成后，打开终端命令，在命令窗口中输入“# vgdisplay”，查看磁盘卷组名。

```
[root@localhost ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <29.00 GiB
  PE Size               4.00 MiB
  Total PE              7423
  Alloc PE / Size       7422 / 28.99 GiB
  Free  PE / Size       1 / 4.00 MiB
  VG UUID               BM9XkO-sfdB-26Fd-Z6DM-HzoA-0ofX-fobcyD
```
  
12、接着，在终端命令窗口中输入“# pvcreate  /dev/sda3”命令并回车，初始化刚建立的分区。

13、接着，在终端命令窗口中输入“# vgextend centos /dev/sda3”命令并回车，把刚初始化的分区加入到虚拟卷组名中（命令：# vgextend 虚拟卷组名 新增的分区）。

```
[root@localhost ~]# pvcreate  /dev/sda3
  Physical volume "/dev/sda3" successfully created.
[root@localhost ~]# vgextend centos /dev/sda3
  Volume group "centos" successfully extended
[root@localhost ~]# 
```

14、接着，继续在终端命令窗口中输入“# vgdisplay”命令，查看卷组的详细信息，在详细信息中可以看到刚增加的10G空间还是处于空闲的状态。

```
[root@localhost ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               38.99 GiB
  PE Size               4.00 MiB
  Total PE              9982
  Alloc PE / Size       7422 / 28.99 GiB
  Free  PE / Size       2560 / 10.00 GiB
  VG UUID               BM9XkO-sfdB-26Fd-Z6DM-HzoA-0ofX-fobcyD
 
```

15、接着，继续在终端命令窗口中输入“# df -h”命令并回车，查看并记下需要扩展的文件系统名（小编这边查看到的系统名是：/dev/mapper/centos-root，不同版本查看到的应该有所区别，大家应该注意区分）。 

```
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 1.9G     0  1.9G    0% /dev
tmpfs                    1.9G     0  1.9G    0% /dev/shm
tmpfs                    1.9G   13M  1.9G    1% /run
tmpfs                    1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/mapper/centos-root   27G   23G  4.8G   83% /
/dev/sda1               1014M  235M  780M   24% /boot
tmpfs                    378M     0  378M    0% /run/user/0
tmpfs                    378M  8.0K  378M    1% /run/user/42
overlay                   27G   23G  4.8G   83% /var/lib/docker/overlay2/b20aee46c1a8b5fd4deddcd1a107d99cedb67b9c5d520207929280be120b5033/merged
overlay                   27G   23G  4.8G   83% /var/lib/docker/overlay2/43444c6f450068b13ab2958fc016ea0f44516f6ac9d50758f2ae2733dee78675/merged
overlay                   27G   23G  4.8G   83% /var/lib/docker/overlay2/8b954061b16d1fcf454f5d7f0dd51694f7548dd2433fbe90c7e6b7859b122628/merged
```


16、接着，继续在终端命令窗口中输入“# lvextend -L +9G /dev/mapper/centos-root”命令并回车，扩容已有的卷组容量（注意：如果您扩容的是10G，这里10G就不能全部扩展，只能扩展比10G小的容量，不然系统会报错导致扩容失败，小编这里只扩容9G。命令：# lvextend -L +需要扩展的容量 需要扩展的文件系统名，需要注意命令中区分字母的大小写）。

  ```
  [root@localhost ~]# lvextend -L +9.5G /dev/mapper/centos-root
  Size of logical volume centos/root changed from 26.99 GiB (6910 extents) to 36.49 GiB (9342 extents).
  Logical volume centos/root successfully resized.
[root@localhost ~]# 
```


17、接着，继续在终端命令窗口中输入“# pvdisplay”命令并回车，查看当前的卷组，在显示的信息中卷组已经扩容成功了，接下来需要将文件系统也扩容。

```
[root@localhost ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <29.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              7423
  Free PE               0
  Allocated PE          7423
  PV UUID               Khhlnt-KvTn-Qm2G-l5OW-2Gax-RNAe-W5wTBi
   
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               centos
  PV Size               10.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               128
  Allocated PE          2431
  PV UUID               azWOrX-wu5g-nBCW-AnfL-zVHW-f6Zw-2m7c85
   
 ```
   
18、接着，继续在终端命令窗口中输入` cat /etc/fstab | grep centos-root` 命令并回车，查看文件系统的格式。

```
[root@localhost ~]# cat /etc/fstab | grep centos-root
/dev/mapper/centos-root /                       xfs     defaults        0 0
[root@localhost ~]# 
```
19、接着，在上一步已经看到了文件系统的格式了，就需要使用相应的命令来扩展磁盘空间，在终端命令中输入` xfs_growfs /dev/mapper/centos-root` 命令并回车（命令：# xfs_growfs 文件系统名，不同的文件系统要用不同的命令，不然会导致报错）。   

 ```
[root@localhost ~]# xfs_growfs /dev/mapper/centos-root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1768960 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=7075840, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=3455, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 7075840 to 9566208
[root@localhost ~]# 
 ```
20、接着，等待系统自动扩容完成，完成后，在终端命令窗口中输入“# df -h”命令并回车，查看磁盘大小是否扩容成功，在显示的磁盘信息中，可以看到磁盘扩容成功了。

  ```
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 1.9G     0  1.9G    0% /dev
tmpfs                    1.9G     0  1.9G    0% /dev/shm
tmpfs                    1.9G   13M  1.9G    1% /run
tmpfs                    1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/mapper/centos-root   37G   23G   15G   61% /
/dev/sda1               1014M  235M  780M   24% /boot
tmpfs                    378M     0  378M    0% /run/user/0
tmpfs                    378M   12K  378M    1% /run/user/42
overlay                   37G   23G   15G   61% /var/lib/docker/overlay2/b20aee46c1a8b5fd4deddcd1a107d99cedb67b9c5d520207929280be120b5033/merged
overlay                   37G   23G   15G   61% /var/lib/docker/overlay2/43444c6f450068b13ab2958fc016ea0f44516f6ac9d50758f2ae2733dee78675/merged
overlay                   37G   23G   15G   61% /var/lib/docker/overlay2/8b954061b16d1fcf454f5d7f0dd51694f7548dd2433fbe90c7e6b7859b122628/merged
[root@localhost ~]# 
  ```

内容来源：[vm centos 扩容](https://www.cnblogs.com/leiblog/p/13560070.html) 