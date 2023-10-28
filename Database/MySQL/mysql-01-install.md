---
layout: default
title: MySQL Install
parent: MySQL
grand_parent: Database
nav_order: 81
permalink: docs/Database/MySQL.Install
---

## MySQL 在线安装与卸载 <!-- {docsify-ignore} -->

### MySQL 在线安装

准备使用yum安装，如果yum命令使用正常，可以跳过。

更新yum软件包

```bash
yum check-update  
```

更新软件

```
yum update
```

 yum update 更新报错解决

当在CentOS 7中使用yum update更新软件时，可能会遇到如下错误信息：

```shell
Error: Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
```

解决这个问题需要安装deltarpm，可以先通过如下命令查找该包的包名：

```shell
yum provides '*/applydeltarpm'
```

然后用如下命令安装即可解决：

```shell
yum install deltarpm
```

1. 下载 MySQL yum包

```
wget http:``//repo.mysql.com/mysql57-community-release-el7-10.noarch.rpm
```

 

2.安装MySQL源

```
rpm -Uvh mysql57-community-release-el7-10.noarch.rpm
```

 

3.安装MySQL服务端,需要等待一些时间

```
yum install -y mysql-community-server
```

 

4.启动MySQL

```
systemctl start mysqld.service
```

 

5.检查是否启动成功

```
systemctl status mysqld.service
```

 

6.获取临时密码，MySQL5.7为root用户随机生成了一个密码

```
grep 'temporary password' /var/log/mysqld.log 
```

![img](data:image/png;base64,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)

 

 

7.通过临时密码登录MySQL，进行修改密码操作

```
mysql -uroot -p
```

使用临时密码登录后，不能进行其他的操作，否则会报错，这时候我们进行修改密码操作

 

8.因为MySQL的密码规则需要很复杂，我们一般自己设置的不会设置成这样，所以我们全局修改一下

```
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
```

这时候我们就可以自己设置自己的密码

 

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'yourpassword';
```

 

9.授权其他机器远程登录

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'yourpassword' WITH GRANT OPTION;
 
FLUSH PRIVILEGES;
```

 

10.开启开机自启动

先退出mysql命令行，然后输入以下命令

```
systemctl enable mysqld
systemctl daemon-reload
```

 

11.设置MySQL的字符集为UTF-8，令其支持中文

```
vim /etc/my.cnf
```

改成如下,然后保存

 

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
 
[mysql]
default-character-set=utf8
 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
default-storage-engine=INNODB
character_set_server=utf8
 
symbolic-links=0
 
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

 

12.重启一下MySQL,令配置生效

```
service mysqld restart
```

 

13.防火墙开放3306端口

```
firewall-cmd --state
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

 

### MySQL 在线安装的卸载

1、删除MySQL组件的命令

```bash
yum remove  mysql mysql-server mysql-libs mysql-server;
```

2、查找跟MySQL相关的文件及文件夹

```bash
find / -name mysql
```

3、删除MySQL文件及文件夹

```bash
rm -rf /var/lib/mysql
```



4、查找已安装的MySQL组件

```bash
rpm -qa|grep -i mysql
```

5、RPM方式卸载已安装组件

```bash
rpm -ev mysql57-community-release-el7-10
```



## MySQL 解压安装  <!-- {docsify-ignore} -->

卸载系统自带的Mariadb

```bash
[root@zhangxiaocai ~]# rpm -qa|grep mariadb
mariadb-libs-5.5.44-2.el7.centos.x86_64
[root@zhangxiaocai ~]# rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64
```

删除etc目录下的my.cnf文件

```bash
[root@zhangxiaocai ~]# rm /etc/my.cnf
rm: cannot remove ?etc/my.cnf? No such file or directory
```
检查mysql是否存在

```bash
[root@zhangxiaocai ~]# rpm -qa | grep mysql
[root@zhangxiaocai ~]# 
```
检查mysql组和用户是否存在，如无创建

```bash
[root@zhangxiaocai ~]# cat /etc/group | grep mysql 

[root@zhangxiaocai ~]#  cat /etc/passwd | grep mysql
```

创建mysql用户组，并设置密码

```bash
#创建mysql用户组
[root@zhangxiaocai ~]# groupadd mysql

#创建一个用户名为mysql的用户并加入mysql用户组
[root@zhangxiaocai ~]# useradd -g mysql mysql

#制定password 这里设置为111111
[root@zhangxiaocai ~]# passwd mysql
Changing password for user mysql.
New password: 
BAD PASSWORD: The password is a palindrome
Retype new password: 
passwd: all authentication tokens updated successfully.
```

一般安装到 `/usr/local` 目录，这里记录安装到`/var`

```bash
# 解压
[root@zhangxiaocai var]# tar -zxvf mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz 

# 重命名文件夹
[root@zhangxiaocai var]# mv mysql-5.7.19-linux-glibc2.12-x86_64/ mysql57

```bash
#更改所属的组和用户
[root@zhangxiaocai var]# chown -R mysql mysql57/

[root@zhangxiaocai var]# chgrp -R mysql mysql57/

[root@zhangxiaocai var]# cd mysql57/
# 创建数据目录
[root@zhangxiaocai mysql57]# mkdir data
# 授与用户权限权
[root@zhangxiaocai mysql57]# chown -R mysql:mysql data
```

在etc下新建配置文件my.cnf，并在该文件内添加以下配置

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
skip-name-resolve
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=/var/mysql57
# 设置mysql数据库的数据的存放目录
datadir=/var/mysql57/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 
lower_case_table_names=1
max_allowed_packet=16M
```

安装和初始化


```bash
[root@zhangxiaocai mysql57]# bin/mysql_install_db --user=mysql --basedir=/var/mysql57/ --datadir=/var/mysql57/data/
2017-04-17 17:40:02 [WARNING] mysql_install_db is deprecated. Please consider switching to mysqld --initialize
2017-04-17 17:40:05 [WARNING] The bootstrap log isn't empty:
2017-04-17 17:40:05 [WARNING] 2017-04-17T09:40:02.728710Z 0 [Warning] --bootstrap is deprecated. Please consider using --initialize instead
2017-04-17T09:40:02.729161Z 0 [Warning] Changed limits: max_open_files: 1024 (requested 5000)
2017-04-17T09:40:02.729167Z 0 [Warning] Changed limits: table_open_cache: 407 (requested 2000)
```

赋权限

```bash
# 复制进程服务
[root@zhangxiaocai mysql57]# cp ./support-files/mysql.server /etc/init.d/mysqld

# 赋权限
[root@zhangxiaocai mysql57]# chown 777 /etc/my.cnf 

# 允许执行
[root@zhangxiaocai mysql57]# chmod +x /etc/init.d/mysqld
```

重启mysql服务

```bash
[root@zhangxiaocai mysql57]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```

#设置开机启动

```bash
[root@zhangxiaocai mysql57]# chkconfig --level 35 mysqld on
[root@zhangxiaocai mysql57]# chkconfig --list mysqld

[root@zhangxiaocai mysql57]# chmod +x /etc/rc.d/init.d/mysqld

[root@zhangxiaocai mysql57]# chkconfig --add mysqld
[root@zhangxiaocai mysql57]# chkconfig --list mysqld

[root@zhangxiaocai mysql57]# service mysqld status
 SUCCESS! MySQL running (4475)
```

```bash
vi etc/profile/
```

```
export PATH=$PATH:/var/mysql57/bin
```

让配置生效

```bash
[root@zhangxiaocai mysql57]# source /etc/profile
```

获得初始密码

```bash
[root@zhangxiaocai bin]# cat /root/.mysql_secret  
# Password set for user 'root@localhost' at 2020-04-17 17:40:02 
p8Uhdk*1j
```
 

修改密码

```bash
[root@zhangxiaocai bin]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.18

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set PASSWORD = PASSWORD('111111');
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

添加远程访问权限


```bash
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select host,user from user;
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | root      |
| localhost | mysql.sys |
+-----------+-----------+
2 rows in set (0.00 sec)
```

```
create user 'xxx'@'%' identified by '123';  
这里 @‘%’ 表示在任何主机都可以登录
```
 
重启生效

```bash
/bin/systemctl restart  mysql.service

[root@zhangxiaocai bin]# /etc/init.d/mysqld restart 
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```