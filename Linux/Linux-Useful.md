---
layout: default
title: Linux Command
has_children: false
parent: Linux
nav_order: 2
---


# Linux
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Linux 环境相关

## lscpu

查看CPU架构信息的标准工具，包括CPU型号

```bash
lscpu
```

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

##  ll ls

列出文件 按文件时间先后排序显示
```bash
# 列出文件 按文件时间先后排序显示
ll -lrt
```

列出符合文件关键字的文件 按时间先后排序显示
```bash
# 列出符合文件关键字的文件 按时间先后排序显示
ll P_AMS*.txt  -lrt
```

统计kjhs*.txt文件个数
```bash
# 统计kjhs*.txt文件个数
ls kjhs*.txt | wc -l
```
统计使用 find（更可靠，支持递归）

```bash
# 统计当前目录下的 .txt 文件数量：
find . -maxdepth 1 -type f -name "*.txt" | wc -l
```
    -maxdepth 1：仅当前目录（不包括子目录）
    -type f：只统计文件（排除目录）
    -name "*.txt"：匹配 .txt 后缀的文件



获取目录中符合关键字文件的最大的文件
```bash
ls -S | head -1

# 获取所有 kjhs开头的 txt 文件， 按文件大小倒排序，取前3
ll -h kjhs*.txt -S | head -3
```

获取目录中符合关键字文件的最新的文件

```bash
# 如果文件名包含日期（如 log_2023-10-01.txt, log_2023-10-02.txt），可以按日期排序：
ls log_*.txt | sort -t '_' -k 2 | tail -1
```
    -t '_'：以 _ 分隔
    -k 2：按第 2 列（日期部分）排序
    tail -1：取最新的文件

ls 可能会遇到文件名包含空格或特殊字符的问题，更安全的方式是 find + sort：
```
find . -maxdepth 1 -type f -name "kjhs_*.txt" | sort -t '_' -k 2 -n | tail -1
```
    -maxdepth 1：只在当前目录查找
    -type f：只匹配文件（排除目录）
    -name "file_*.txt"：匹配文件名模式


计算文件大小
```bash
find . -maxdepth 1 -type f -name "*.txt" -exec du -cb {} + | grep total | awk '{print $1}'
```
    -exec du -cb {} +：计算所有匹配文件的总大小
    grep total：提取 du 输出的 total 行
    awk '{print $1}'：仅显示总字节数


**计算文件大小**
```bash
find . -maxdepth 1 -type f -name "*.txt" -printf "%s\n" | awk '{sum += $1} END {print "文件数:", NR, "总大小:", sum, "字节"}'
```

    -printf "%s\n"：输出每个文件的大小（字节）
    awk 计算总和：
        sum += $1：累加文件大小
        NR：文件数量（行数）
        END：最终输出结果
        

统计 .txt 文件的总大小（人类可读格式）
```bash
du -ch *.txt | grep total
```
    du -ch：计算总大小并显示 total
    grep total：提取总大小行
        



## curl

安装

```bash
yum install curl
```

curl 是一个非常强大的命令行工具，用于发送请求到服务器并获取响应。

参数：

|参数|描述|
|----|-----|
|   -a/--append | 上传文件时，附加到目标文件 | 
|    -A/--user-agent `<string>` | 设置用户代理发送给服务器 | 
|    - anyauth  | 可以使用“任何”身份验证方法 |
|    -b/--cookie <name=string/file>  | cookie字符串或文件读取位置|
|    - basic  | 使用HTTP基本验证|
|    -B/--use-ascii  | 使用ASCII /文本传输|
|  -c/--cookie-jar `<file>` |操作结束后把cookie写入到这个文件中|
|    -C/--continue-at `<offset>` |断点续转|
|   -d/--data `<data>` |HTTP POST方式传送数据|
|   --data-ascii `<data>` |以ascii的方式post数据|
|   --data-binary `<data>` |以二进制的方式post数据|
|   --negotiate |使用HTTP身份验证|
|   --digest |使用数字身份验证|
|   --disable-eprt |禁止使用EPRT或LPRT|
|   --disable-epsv |禁止使用EPSV|
|   -D/--dump-header `<file>` |把header信息写入到该文件中|
|   --egd-file `<file>` |为随机数据(SSL)设置EGD socket路径|
|  --tcp-nodelay |使用TCP_NODELAY选项|
|   -e/--referer |来源网址|
|   -E/--cert <cert[:passwd]> |客户端证书文件和密码(SSL)|
|   --cert-type `<type>` |证书文件类型(DER/PEM/ENG) (SSL)|
|   --key `<key>` |私钥文件名 (SSL)|
|   --key-type `<type>` |私钥文件类型(DER/PEM/ENG) (SSL)|
|   --pass `<pass>` |私钥密码(SSL)|
|   --engine `<eng>` |加密引擎使用(SSL). "--engine list" for list|
|   --cacert `<file>` |CA证书(SSL)|
|   --capath `<directory>`  | CA目录(made using c_rehash) to verify peer against (SSL)|
|   --ciphers `<list>`  | SSL密码|
|   --compressed  | 要求返回是压缩的形势(using deflate or gzip)|
|   --connect-timeout `<seconds>`  | 设置最大请求时间|
|   --create-dirs  | 建立本地目录的目录层次结构||
|   --crlf  | 上传是把LF转变成CRLF
|   -f/--fail  | 连接失败时不显示http错误|
|   --ftp-create-dirs  | 如果远程目录不存在，创建远程目录|
|   --ftp-method [multicwd/nocwd/singlecwd]  | 控制CWD的使用|
|   --ftp-pasv  | 使用PASV/EPSV 代替端口 | 
|   --ftp-skip-pasv-ip  | 使用PASV的时候,忽略该IP地址 | 
|   --ftp-ssl  | 尝试用SSL/TLS 来进行ftp数据传输 | 
|   --ftp-ssl-reqd  | 要求用SSL/TLS 来进行ftp数据传输 | 
|   -F/--form <name=content>  | 模拟http表单提交数据 | 
|   -form-string <name=string>  | 模拟http表单提交数据 | 
|   -g/--globoff  | 禁用网址序列和范围使用{}和[] | 
|   -G/--get  | 以get的方式来发送数据 | 
|   -h/--help  | 帮助 | 
|   -H/--header <line> | 自定义头信息传递给服务器 | 
|   --ignore-content-length  | 忽略的HTTP头信息的长度 | 
|   -i/--include  | 输出时包括protocol头信息 | 
|   -I/--head  | 只显示文档信息 | 
|   -j/--junk-session-cookies 从文件中读取略session cookie | 
|   –interface<interface> |指定网络接口/地址使用 | 
|   - krb4 <级别> |启用与指定的安全级别krb4 | 
|   -j/--junk-session-cookies |读取文件进忽略session cookie | 
|   --interface <interface> |使用指定网络接口/地址|
|   --krb4 <level> |使用指定安全级别的krb4|
|   -k/--insecure |允许不使用证书到SSL站点|
|   -K/--config |指定的配置文件读取|
|   -l/--list-only |列出ftp目录下的文件名称|
|   --limit-rate <rate> |设置传输速度|
|   --local-port<NUM> |强制使用本地端口号|
|   -m/--max-time `<seconds>`  | 设置最大传输时间|
|   --max-redirs `<num>`  | 设置最大读取的目录数|
|   --max-filesize `<bytes>`  | 设置最大下载的文件总量|
|   -M/--manual |显示全手动|
|   -n/--netrc |从netrc文件中读取用户名和密码 | 
|  --netrc-optional |使用.netrc 或者URL来覆盖-n | 
|   --ntlm | 使用HTTP NTLM 身份验证 | 
|   -N/--no-buffer  | 禁用缓冲输出 | 
|   -o/--output  | 把输出写到该文件中 | 
|   -O/--remote-name  | 把输出写到该文件中，保留远程文件的文件名 | 
|   -p/--proxytunnel  | 使用HTTP代理 | 
|   --proxy-anyauth  | 选择任一代理身份验证方法 | 
|   --proxy-basic  | 在代理上使用基本身份验证 | 
|   --proxy-digest  | 在代理上使用数字身份验证 | 
|   --proxy-ntlm  | 在代理上使用ntlm身份验证 | 
|   -P/--ftp-port `<address>`  | 使用端口地址，而不是使用PASV | 
|   -Q/--quote `<cmd>` | 文件传输前，发送命令到服务器 | 
|   -r/--range `<range>`  | 检索来自HTTP/1.1或FTP服务器字节范围 | 
|   --range-file  | 读取（SSL）的随机文件 | 
|   -R/--remote-time  | 在本地生成文件时，保留远程文件时间 | 
|   --retry `<num>`  | 传输出现问题时，重试的次数 | 
|   --retry-delay `<seconds>`  | 传输出现问题时，设置重试间隔时间 | 
|   --retry-max-time `<seconds>`  | 传输出现问题时，设置最大重试时间 | 
|   -s/--silent  | 静音模式。不输出任何东西 | 
|   -S/--show-error  | 显示错误 | 
|   --socks4 <host[:port]>  | 用socks4代理给定主机和端口 | 
|   --socks5 <host[:port]>  | 用socks5代理给定主机和端口 | 
|   --stderr <file>  |   | 
|   -t/--telnet-option <OPT=val>  | Telnet选项设置 | 
|   --trace `<file>`  | 对指定文件进行debug | 
|   --trace-ascii `<file>` Like - | -跟踪但没有hex输出 | 
|   --trace-time  |  跟踪/详细输出时，添加时间戳 | 
|   -T/--upload-file `<file>` | 上传文件 | 
|   --url <URL>  | Spet URL to work with | 
|   -u/--user <user[:password]>  | 设置服务器的用户和密码 | 
|   -U/--proxy-user <user[:password]>  | 设置代理用户名和密码 | 
|   -v/--verbose  |   | 
|   -V/--version  | 显示版本信息
|   -w/--write-out [format]  | 什么输出完成后 | 
|   -x/--proxy <host[:port]>  | 在给定的端口上使用HTTP代理 | 
|   -X/--request `<command>` | 指定什么命令 | 
|   -y/--speed-time  | 放弃限速所要的时间。默认为30 | 
|   -Y/--speed-limit  | 停止传输速度的限制，速度时间'秒 | 
|   -z/--time-cond  | 传送时间设置 | 
|   -0/--http1.0  | 使用HTTP 1.0 | 
|   -1/--tlsv1  | 使用TLSv1（SSL） | 
|   -2/--sslv2  | 使用SSLv2的（SSL） | 
|   -3/--sslv3  | 使用的SSLv3（SSL） | 
|   --3p-quote like -Q  | for the source URL for 3rd party transfer | 
|   --3p-url  | 使用url，进行第三方传送 | 
|   --3p-user  | 使用用户名和密码，进行第三方传送 | 
|   -4/--ipv4  | 使用IP4 | 
|   -6/--ipv6  | 使用IP6 | 
|   -#/--progress-bar  | 用进度条显示当前的传送状态 | 

常用实例：

## 1、常用请求

```bash
## 1、测试访问 get 请求
curl http://docs.zhangxiaocai.cn
curl https://docs.zhangxiaocai.cn

## get 请求获得的内容写到文件
curl -o home.html http://docs.zhangxiaocai.cn
curl http://docs.zhangxiaocai.cn > index.html

# POST 请求带参数
curl -X POST -d "param1=value1&param2=value2" http://localhost:8080/resource

# POST 请求带指定格式的参数 json
curl -X POST -H "Content-Type: application/json" -d '{"key1":"value1", "key2":"value2"}' http://localhost:8080/resource

# PUT 请求
curl -X PUT -d "param1=value1&param2=value2" http://localhost:8080/resource

# DELETE 请求
curl -X DELETE http://localhost:8080/resource

# 带请求头的请求
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" http://localhost:8080/resource
```
## 2、下载文件 

```bash
get 请求下载文件  -O  （大写O）
curl -O https://cdn.jsdelivr.net/gh/small-rose/small-rose.github.io/favicon.ico

# 断点续传下载
curl -C -O http://xxx.com/download/2025/something.tar.gz

# 分段下载
curl -r 0-100 -o file.part1 http://localhost:8080/resource/download/big_file.zip
curl -r 101-200 -o file.part1 http://localhost:8080/resource/download/big_file.zip
curl -r 201-300 -o file.part1 http://localhost:8080/resource/download/big_file.zip
cat file.part* > big_file.zip

#下载显示进度条
curl -# -O http://localhost:8080/resource/download/big_file.zip
```

## 3、模拟登录

```bash
# 3，模拟表单信息，模拟登录，保存cookie信息
curl -c ./cookie_c.txt -F log=admin -F pwd=****** http://localhost:8080/login

# 4，模拟表单信息，模拟登录，保存头信息
curl -D ./cookie_D.txt -F log=admin -F pwd=****** http://localhost:8080/login
```

> -c(小写)产生的cookie和-D里面的cookie是不一样的。

## 4、其他

```bash
# -v 显示详细信息
curl -v http://example.com

# --header：允许你添加额外的 HTTP 头部
curl --header "X-My-Header: 123" http://example.com。

# --data-urlencode：用于发送 URL 编码的数据
curl --data-urlencode "comment=hello world" http://example.com

# --cookie：发送 cookie
curl --cookie "name=value" http://example.com

# --include 或 -i：包括 HTTP 头在输出中 
curl -i http://example.com。

# -e 伪造来源
url -e http://baidu.com http://localhost:8080/login

# --user-agent 添加代理
curl --user-agent "Mozilla/5.0" http://localhost:8080/login

# –referer
curl –referer http://www.baidu.com https://proxy.mimvp.com

# 使用--limit-rate限制curl的下载速度：
curl URL –limit-rate 50k
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

**查系统空间占用**

```bash
df
# 显示G单位
df -h 
#显示 M 单位
df -m
```

**查当前目录下每个文件夹空间占用大小**

```bash
du -sh */
```



2.定位最大文件目录

```bash
#进入根目录。
cd / 

#寻找当前目录，哪个文件夹占用空间最大
du -h --max-depth=1 

```

追踪某个目录大小

```bash
cd /

#也可以直接追踪某个目录大小
du -sh  /applog
```


找整个机器下所有大于100M的文件

```bash
#进入根目录。
cd / 

#查找整个机器下所有大于100M的文件并显示出完整的路径
find / -xdev -size +100M -exec ls -l {} \;
```

以G为单位列出目录大小

```bash
cd /
 
# 查找大文件
du -h | grep [0-9]G 
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


5.其他

若是大文件均清除后磁盘空间仍未释放，则还有一个可能，存在僵尸进程，就是系统文件删除后还存在进程活着的情况。

可通过命令：

```bash
lsof |grep delete
```
语句查看对应进程号，再使用kill杀掉对应进程即可
```
kill -9 进程号
```


## 空间清理

1.统计文件数量

查询当前目录下有多少个pdf文件

```bash
find . -type f -name "*.pdf" | wc -l
```

