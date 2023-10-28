---
layout: default
title: MySQL V8 Setup
parent: MySQL
grand_parent: Database
nav_order: 85
---



## MySQL 8 解压安装记录

下载zip包，解压：`C:\devtools\mysql-8.0.22-winx64`

### 1. 创建配置文件

在目录下创建一个名为`my.ini`的文件：

```bash
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8

[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
#basedir = C:\\devtools\\mysql-8.0.22-winx64\\
# 设置mysql数据库的数据的存放目录
#datadir = C:\\devtools\\mysql-8.0.22-winx64\\data
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 创建模式
sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

### 2. 配置环境变量

（1）新建系统变量：

变量名：`MYSQL_HOME`

变量值：`C:\devtools\mysql-8.0.22-winx64`

（2）Path 追加

```bash
%MYSQL_HOME%\bin;
```


### 3. 执行初始化

以管理员运行CMD，安装mysql服务，进入`C:\devtools\mysql-8.0.22-winx64\bin` 目录：

```bash
mysqld --initialize
```
找到 data 目录下有个 `xxx.err` 文件，xxx一般是你的计算机名称，可以在计算机/此电脑，右键属性可以看到你的计算机名称。

打开改文件找到对应的密码：

```txt
2020-11-19T07:08:46.118531Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-11-19T07:08:47.275550Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-11-19T07:08:49.076756Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2020-11-19T07:10:24.084477Z 0 [System] [MY-010116] [Server] C:\devtools\mysql-8.0.22-winx64\bin\mysqld.exe (mysqld 8.0.22) starting as process 6500
2020-11-19T07:10:24.086846
```
我的提示是初始密码为空，如果不为空会记录临时密码。


### 4. 服务安装

以管理员运行CMD，安装mysql服务，进入`C:\devtools\mysql-8.0.22-winx64\bin` 目录：

```bash
mysqld.exe -install
```
```txt
Service successfully installed
```
安装之后可以进行服务启停：

```bash
net start mysql
net stop mysql
```

启动服务就可以登录了。


### 5.修改密码：

使用临时密码或者空密码登录：
```bash
mysql -u root -p 
```
登录之后修改密码：

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY '123456';

flush privileges;
```

### 6.验证修改密码

```bash
exit
```
使用新密码登录验证：

```bash
mysql -u root -p 123456
```


