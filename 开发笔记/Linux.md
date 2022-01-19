Linux 环境相关

### grep

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

### wget
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

3、如果安装了还是不能使用
就先卸载再重新安装

```bash
yum remove wget
yum install wget -y
```

### Ruby 安装

在线安装

```bash
# CentOS, Fedora, 或 RHEL 系统
sudo yum install ruby   

# Debian 或 Ubuntu 系统
sudo apt-get install ruby-full    
```
离线安装

下载安装包 http://www.ruby-lang.org/en/downloads/ 之后上传，或者使用wget下载。

```bash
wget -c  https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.0.tar.gz
tar -xvzf ruby-3.1.0.tar.gz  
cd ruby-3.1.0

./configure
make
sudo make install

ruby -v
```

### gem 安装

Linux ubuntu 安装gem

```bash
wget -c http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz

tar xzvf rubygems-1.8.24.tgz

cd rubygems-1.8.24

ruby setup.rb
```

列出安装源 
```bash
gem sources -l
```
    http://gems.github.com/
    http://rubygems.org/
    https://gems.ruby-china.org

添加安装源 
```
gem sources -a XXX
```

```bash
gem source -a https://gems.ruby-china.org

gem source -a http://gems.github.com/

gem source -a http://rubygems.org/
```

```bash
#删除安装源
gem sources -r XXX

#更新安装源缓存
gem sources -u
 

升级/更新

```bash
#更新 gem 本身
gem update --system

#更新所有程序包
gem update

使用 gem 安装**pod **

```bash
sudo gem install cocoapods
```

### screen 安装

```bash
yum  -y install screen
```

命令模板:

```
screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]
```

参数说明
```
-A 　将所有的视窗都调整为目前终端机的大小。
-d <作业名称> 　将指定的screen作业离线。
-h <行数> 　指定视窗的缓冲区行数。
-m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。
-r <作业名称> 　恢复离线的screen作业。
-R 　先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业。
-s 　指定建立新视窗时，所要执行的shell。
-S <作业名称> 　指定screen作业的名称。
-v 　显示版本信息。
-x 　恢复之前离线的screen作业。
-ls或--list 　显示目前所有的screen作业。
-wipe 　检查目前所有的screen作业，并删除已经无法使用的screen作业。
```
常用命令

```bash
screen -S yourname  # 新建一个叫yourname的session
screen -ls          # 列出当前所有的session
screen -r yourname  # 回到yourname这个session
screen -d yourname  # 远程detach某个session
screen -d -r yourname # 结束当前session并回到yourname这个session
```

linux screen 命令详解 : https://www.cnblogs.com/mchina/archive/2013/01/30/2880680.html


### tree 安装

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

### netstat 安装

---------------------------
centos7默认没netstat命令，需要安装

```bash
yum install net-tools 
```



### fuser命令

------------------------------
fuser命令需要安装
```bash
yum install psmisc 
```



### rzsz

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


### 系統监控 

```bash
yum install sysstat
```



### Firawalld

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



### 查看端口

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



### 查看进程

```bash
ps -ef | grep  xxx
ps aux | grep  xxx

ps -ef | grep $APP_NAME | grep -v grep |awk '{print $2}'
```

### 基础环境

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

```
vi /etc/profile
vi  .bash
```

在配置文件末尾添加

```
export M2_HOME=/usr/local/maven/apache-maven-3.8.2
export JAVA_HOME=/usr/local/java/jdk1.8.0_212
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:${M2_HOME}/bin:$PATH
```

### nginx

在线安装

```bash
yum install gcc-c++       
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

cd /usr/local
wget http://nginx.org/download/nginx-1.20.1.tar.gz
tar -zxvf  nginx-1.20.1.tar.gz
./configure && make && make install

# 这是 nginx 软连接
ln -s /usr/local/nginx/sbin/nginx  /usr/local/bin/nginx
nginx
ps -ef|grep nginx
```

离线安装

```bash


```

### tomcat 线程数

上线前可以压测一下，调整参数配置，查看tomcat线程数命令如下：

获取tomcat进程pid ：ps -ef|grep tomcat

统计该tomcat进程内的线程个数 ：ps -Lf 29295 |wc -l


### 应用重启脚本

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



### v2Ray

参考： https://hissin.cn/zheteng/795.html

https://github.com/Loyalsoldier/v2ray-rules-dat

https://yearliny.com/v2ray-complete-tutorial/
https://www.linodovultr.com/post/resolve-v2ray-after-install-can-not-connect.html?replytocom=69
https://github.com/search?q=v2ray
https://fx.tmioe.com/654.html
https://www.ddayh.com/1136.html

https://ivu4e.com/toolbox/newwork/2019-10-16/259.html

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
            "id": "98ee3c11-fe5b-4e57-a2ca-882b10c7b402", // ID
            "alterId": 66
          }
        ]
      }
    },
    {
      "port": 444, // SS 协议服务端监听端口
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-128-gcm", // 加密方式
        "password": "small.CAI" //密码
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
