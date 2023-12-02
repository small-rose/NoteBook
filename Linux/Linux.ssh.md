---
layout: default
title: Linux ssh
has_children: false
parent: Linux
nav_order: 2
---


# Linux - ssh 证书登录
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## ssh 证书登录



## Linux 多个主机质检免密登录

### 节点信息

| IP | 节点名称 | 系统 |
|-|-|-|
| 192.168.147.140 | k3s-server1 | CentOS7.9 |
| 192.168.147.141 | k3s-server2 | CentOS7.9 |
| 192.168.147.142 | k3s-server3 | CentOS7.9 |


### 1. 生产免密的公私钥对

在三个节点分别执行ssh-keygen 命令生成公私钥对。

```bash
ssh-keygen -t rsa
```
文件名默认是id_rsa和id_rsa.pub

root账户路径在`/root/.ssh/`

普通账户路径在`/home/username/.ssh/`


### 2. 将公钥集中到一台


把另外2个节点的公钥复制到server1来，本次需要使用root密码

```bash
scp  root@192.168.147.141:/root/.ssh/id_rsa.pub  /root/.ssh/id_rsa_141.pub
scp  root@192.168.147.142:/root/.ssh/id_rsa.pub  /root/.ssh/id_rsa_142.pub
```

### 3、配置免密

查看/root/.ssh/是否有文件`authorized_keys`，如果没有则创建一个,如果有就直接追加内容。

将server1 的 id_rsa.pub 追加到 authorized_keys 文件中：
```bash
cat  /root/.ssh/id_rsa.pub >> ./authorized_keys
```

将server2 和 server3 的 id_rsa.pub 也追加到 authorized_keys 文件中：

```bash
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
cat /root/.ssh/id_rsa_141.pub >> /root/.ssh/authorized_keys
cat /root/.ssh/id_rsa_142.pub >> /root/.ssh/authorized_keys
```

验证server1 的 `authorized_keys` 文件配置是否成功追加3个ssh-rsa的配置：

```bash
cat /root/.ssh/authorized_keys
```

将配置分发其他机器：

server 2 、server 3 分别执行

此时已不需要root密码了。

```bash
scp root@192.168.147.140:/root/.ssh/authorized_keys  /root/.ssh/
```
### 3、免密使用

```bash
ssh root@192.168.147.140
ssh root@k3s-server1
ssh root@192.168.147.141
ssh root@192.168.147.142
```

后续验证可以相互ssh不要密码了。
