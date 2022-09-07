---
layout: default
title: Docker Use
parent: DevNotes
nav_order: 5
---

# Docker Use
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## 固定IP配置

```shell script
vi /etc/sysconfig/network-scripts/ifcfg-ens33 
```

```text
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
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


docker UI
-----------------------

## Portainer.io

**搜索**

```bash
docker search portainer
```

**下载**

```shell script
docker pull portainer/portainer　
```
　　 
**启动**

```shell script
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --name prtainer-zzy portainer/portainer
```

参数说明：

```bash
docker run -d 　　　　　　　　# 后台运行容器
　　-p 9000:9000 　　　　　　 # 默认9000端口，映射到宿主机，通过本地地址访问
　　--name prtainer-test 　　　# 指定容器名
　　--restart=always 　　　　  # 设置自动启动
    -v /opt/portainer:/data     # 保存portainer数据到宿主机
    -v /var/run/docker.sock:/var/run/docker.sock 　　# 单机方式必须指定
　　portainer/portainer　
```

**访问**

```
http://192.168.80.81:9000/#!/home
admin
1qaz2wsx
```

## Mysql 安装

**下载**

```shell script
docker pull mysql：5.7.39
```
　　
**启动**

```shell script
docker run --name mysql01 -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.39
```

解析：

```text
--name mysql-sr                                #  对容器的命名
-d                                             # 后台运行
-p 3310:3306                                   #对外暴露端口号3310
-v /home/mysql/conf:/etc/mysql/conf.d     #配置文件挂载到当前宿主机的/home/mysql/conf
-v /home/mysql/data:/var/lib/mysql            #数据挂载到当前宿主机的 /home/mysql/data
-e MYSQL_ROOT_PASSWORD=123456    #设置mysql的root用户的密码是：·123456
```

也可以界面部署

ENV:

```text
MYSQL_ROOT_PASSWORD   12345678
```

## oracle_11g

在 dockerHub 上可以搜 `oracle` 或 `oracle_11g`

使用阿里镜像

```shell script
docker pull  docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```
使用 `docker images` 查看镜像


启动容器

默认启动容器的方式
```shell script
docker run -d -it -p 1521:1521 --name oracle11g --restart=always registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

持久化启动的方式

```shell script
docker run -d -it -p 1521:1521 --name oracle11g --restart=always --mount source=oracle_vol,target=/home/oracle/app/oracle/oradata registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

查看启动的线程 

```shell script
netstat -tulnp
```
可以看到1521端口的映射。

容器内环境配置

进入容器 
```shell script
docker exec -it oracle11g bash
```

切换到 root 用户 su root，密码为 helowin
```shell script
su root
```

docker容器配置环境变量不是在 /etc/profile 中，容器启动不会走这个文件。

可以将环境变量的配置设置在 /home/oracle/.bashrc 文件下，这样可以省略掉软连接的创建 `ln -s $ORACLE_HOME/bin/sqlplus /usr/bin`

编辑环境变量 
```shell script
vi /home/oracle/.bashrc
```

在文件最后加入以下命令

```text
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2

export ORACLE_SID=helowin

export PATH=$ORACLE_HOME/bin:$PATH
```


wq 保存并退出。然后使用 

```shell script
source /home/oracle/.bashrc 
```
刷新环境变量，并使之生效

进入 oracle 命令行

使用 

```shell script
sqlplus /nolog 
```
进入oracle命令行

登录oracle

```shell script
conn / as sysdba
```
 
如果直接使用默认的 root 用户登录，会报登录失败。这里必须使用 su - oracle 命令，将当前用户切换到 oracle，然后在执行登录命令

ORA-12514, TNS:listener does not currently know of service requested in connect descriptor

这个错误是由于数据库名用错了

```shell script
su - oracle

sqlplus /nolog

conn / as sysdba
```

```oraclesqlplus
select instance_name from v$instance;

show user;
```
使用上述命令查出来的，就是所有可用的 “数据库名” 和 “用户名”


> 阿里的这个镜像，所有的密码都是统一的 **helowin**

system用户具有DBA权限，但是没有SYSDBA权限。平常一般用该帐号管理数据库。
而sys用户是Oracle数据库中权限最高的帐号，具有“SYSDBA”和“SYSOPER”权限，一般不允许从外部登录

使用system连接测试：

```yml
url: jdbc:oracle:thin:@localhost:1521:ORCL
username: system
password: helowin
```
实际使用过程中，可以创建用户，不同的用户表是相互隔离的。


**配置防火墙**

防火墙要允许 1521 端口，外部的数据库管理工具才能连的上


```bash
#打开防火墙
systemctl start firewalld
service firewalld status
#查询端口状态
firewall-cmd --query-port=1521/tcp

#永久性开放端口
firewall-cmd --permanent --zone=public --add-port=1521/tcp

#重启防火墙
firewall-cmd --reload

firewall-cmd --query-port=1521/tcp
```

## rabbitMQ

**下载**

```shell script
docker pull rabbitmq:3.10.7-management
```

**启动**

```shell script
docker run -di --name rabbitmq -p 5672:5672 -p 15672:15672  rabbitmq:3.10.7-management
```

```shell script
docker run -d --name rabbitmq3.9.3 -p 5672:5672 -p 15672:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname  -e RABBITMQ_DEFAULT_VHOST=/  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.10.7-management
```

如果不想特别说明带上版本信息 management 也可以直接拉取最新的，然后启动容器：

```bash
docekr pull rabbitmq
docker run -d --hostname my-rabbitmq --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq
```

然后可以进入容器后执行以下命令启动界面程序：
```bash
rabbitmq-plugins enable rabbitmq_management
```
访问界面程序

```text
http://192.168.80.81:15672/#/
guest/guest
```
验证启动服务

```bash
curl -v  127.0.0.1:5672
```

https://www.modb.pro/db/392483

https://blog.csdn.net/weixin_39609953/article/details/110235188

