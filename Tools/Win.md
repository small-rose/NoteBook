---
layout: default
title: Windows
nav_order: 2
parent: Tools
---

# Windows
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## windows 端口占用

```text
netstat -aon | findstr :80
```

找到对应的PID,根据PID找到对应的服务名称

```text
tasklist|findstr “12824”

tasklist /fi “PID eq 12824”
```

## windows 端口转发

windows 添加端口转发
```text
netsh interface portproxy add v4tov4 listenaddress=127.0.0.1 listenport=9000 connectaddress=37.220.51.222 connectport=1521
```

windows 移除端口转发

```text
netsh interface portproxy delete v4tov4 listenaddress=127.0.0.1 listenport=9000
```

## win10 SSH

安装open ssh服务

启动ssh服务

```
net start sshd
```

查看登录用户
```
net user
```

停止ssh服务

```
net stop sshd
```
也可以在"服务"里操作。


## nslookup

nslookup 全称:name server lookup。

它有两种模式:交互 & 非交互，进入交互模式在命令行界面直接输入nslookup按回车，非交互模式则是后面跟上查询的域名或者IP地址按回车。一般来说，非交互模式适用于简单的单次查询，若需要多次查询，则交互模式更加适合.

RR (Resource Records) – 来自WIKI百科以及计算机网络: 自顶向下(7th)

资源记录（RR）是包含了下列字段的4元组：
(Name, Value, Type, TTL)

- 主机记录（A记录）：RFC 1035 定义，A记录是用于名称解析的重要记录，提供标准的主机名到IP的地址映射。
- 别名记录（CNAME记录）: RFC 1035 定义，向查询的主机提供主机名对应的规范主机名。
- 域名服务器记录（NS记录） ：用来指定该域名由哪个DNS服务器来进行解析。 您注册域名时，总有默认的DNS服务器，每个注册的域名都是由一个DNS域名服务器来进行解析的，DNS服务器NS记录地址一般以以下的形式出现： ns1.domain.com、ns2.domain.com等。 简单的说，NS记录返回域中主机IP地址的权威DNS服务器的主机名。
- 邮件交换记录（MX记录）：返回别名为Name对应的邮件服务器的规范主机名。

![Resource Records](../Assets/images/resource_records.png)

