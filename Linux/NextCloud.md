---
layout: default
title: NextCloud Install
nav_order: 30
parent: Linux
---

# NextCloud Install
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## NextCloud 

可以搭建自己的个人云盘。

官网： https://nextcloud.com/

镜像： https://hub.docker.com/_/nextcloud


## Linux 安装

(暂无)

## Docker 安装

### 前置准备

#### 1、docker 环境

（必须）

参考docker篇

#### 2、docker redis 

（可选）

参考docker use redis

```bash

```

#### 3、docekr mysql

（可选）

参考docker use mysql 篇。这里只写安装之后需要做的事情。

创建出`nextcloud`需要使用的数据库。登录容器。

```bash
docker exec -it mysqlsr /bin/bash

mysql -uroot -pjWhEMfMwZDTJjNP@23
```

创建数据库并授权。

```sql
-- 新建数据库
create database nextcloud  default charset utf8mb4 COLLATE utf8mb4_general_ci;

-- 新建用户
create user 'nextcloud'@'%' identified by 'small@2023';

-- 授权全部权限
grant all privileges on nextcloud.* to 'nextcloud'@'%';

-- 因为要建表，就用这个授权吧
GRANT all privileges on nextcloud.* to 'nextcloud'@'%' IDENTIFIED BY 'jWhEMfMwZDTJjNP@23' WITH GRANT OPTION;
```


### 安装 docker nextcloud


docker 安装 next cloud ：

```bash
docker run -d --name nextcloud \
    -v /opt/docker/nextcloud/data:/var/www/html \
    --link mysqlsr:mysql \
    --link small-redis:redis \
    --restart=always \
    -p 8801:80 nextcloud
```

访问: http://IP:8801 安装配置管理员账户和数据库配置。

默认使用SQLLITE。

如果换自己的数据库在这个安装界面配置数据库。

{: .tips }
> 提示：云服务需要在安全组开放对象的端口。建议使用域名偶nginx的80端口转发。


**问题1**
>
> 分享文件的下载地址变localhost解决
```
vi /var/www/html/config/config.php
'overwrite.cli.url' => '你的域名或者IP'
```

**问题2：**
>
> nextcloud 错误  Error while trying to create admin user: Failed to connect to the database: An exception occurred in the driver: SQLSTATE[HY000] [2002] No such file or directory 

IP地址写mysql容器内部的地址试试。


其他参考

https://mrchou.com/internet/build-nextcloud-easily-by-docker.html


