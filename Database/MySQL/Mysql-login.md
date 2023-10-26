---
layout: default
title: MySQL Login
nav_order: 8
parent: MySQL About
grand_parent: Database
permalink: docs/Database/MySQL.Denied
---


## Linux下MySQL的Access denied for user

> 使用环境：Centos7.4 ，Mysql5.7

### 1、root不能在版本地登录

**问题描述：**

在linux命令行用mysql -uroot -ppasswaord 登录却报了这么个错：

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)



**解决方法：**

（1）首先要确定登录密码正确，如果密码不对也会出现这个错误。（可以看应用里连接配置确认）

（2）如果密码正确之后还是不能连接：

​	（a）编辑`/etc/my.cnf`的mysqld下 添加`skip-grant-tables`

​	（b）重启MySQL服务，根据版本选择。

```bash
service mysqld restart
systemctl restart mysqld.service
```

​	（c）直接mysql -uroot 登录

​	（d）修改MySQL密码

```sql
# 5.7之前
update user set password=password('your_password') where user='root' and host='localhost';
# 5.7之后
update user set authentication_string=password('your_password') where user='root' and host='localhost';

flush privileges;
```

​	（e）注释掉`skip-grant-table`、重启MySQL服务。再次登录验证即可。



### 2、用户不能在非本地登录

**问题描述：**

一般出现在客户端连接mysql服务的时候：

mysql access denied for user root@ip useing password



**解决方法：**

（1）首先确保密码正确

（2）如果密码正确还是不能连接，说明缺少授权。

​	（a）mysql -uroot 在服务端登录

​	（b）查询用户登录的授权状态

```sql
use mysql;
select host,user from user;
```

​	（c）确认用户在指定IP没有授权，就授权

```sql
-- 适用于开发账户，授权root在任意主机登录，拥有全部schema的全部表的全部权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'your_password' WITH GRANT OPTION;
-- 刷新权限，刷新之后客户端需要重新连接方可生效
FLUSH   PRIVILEGES;
```

​	（d）客户端重新连接即可。

<font color="red" >特别说明：</font>如果对权限管理比较严格，可以按IP授权，也可以按网段授权，可以授权不同的权限级别，可以授权不同操作权限等相关操作，可以参考关于Mysql授权的的内容在《Mysql常用命令》中的授权部分。