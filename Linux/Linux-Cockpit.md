---
layout: default
title: Cockpit Linux
nav_order: 0
parent: Linux
---



# Linux - Cockpit-Project
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Linux Cockpit

>很多时候，我们有一堆服务器来运行很多应用，并且这些服务器可能在不同服务商，但是我们又渴望统一管理。
>这个时候，我们就需要一个管理软件来管理这些服务器。试用了一下cockpit感觉还不错，
>软件官网是：https://cockpit-project.org/

### 基本服务安装

```bash
# 安装服务
yum install cockpit
# 启动服务
systemctl start cockpit.service
#设置开机启动
systemctl enable --now cockpit.socket
# 开启防火墙
firewall-cmd --add-service=cockpit --permanent
```

安装完成之后访问即可，默认端口为 **9090** 。https://192.168.80.81:9090/

使用机器的账户密码登录即可。

### 支持管理 docker

```bash
yum install cockpit-docker -y
```

### 支持管理 podman

```bash
yum install cockpit-podman -y
```


### 支持管理 磁盘

```bash
yum install -y cockpit-storaged
```

### 支持管理多台主机

```bash
yum install -y cockpit-dashboard
```

目标服务器也需要安装cockpit），而连接的方式可以通过账号密码或者直接ssh秘钥免密登录。

如果我们想让服务器A不用输入密码就可以连接服务器B，那么我们只需要在服务器A上执行下面的命令

```bash
# 需要三次回车
ssh-keygen -t rsa
# 需要输入服务器B的密码
ssh-copy-id -i ~/.ssh/id_rsa 服务器B的IP
```

当我们可以免密登录的时候，我们就可以在cockpit的管理面板直接添加而不需要填写密码。

### 支持证书管理

```bash
yum install -y cockpit-certificates
```
cockpit-certificates


### 更多插件

https://cockpit-project.org/applications.html

