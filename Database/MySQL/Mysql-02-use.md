---
layout: default
title: MySQL Use
parent: MySQL
grand_parent: Database
nav_order: 82
---


# MySQL Use
{: .no_toc }


## 创建数据库

### root 账户创建

> 登录mysql
> mysql -hlocalhost -pyourpass


```sql
CREATE DATABASE mydatabase
CHARACTER SET utf8mb4
COLLATE utf8mb4_general_ci;
```

防止重复创建

```sql
CREATE DATABASE IF NOT EXISTS mydatabase 
       CHARACTER SET utf8mb4
COLLATE utf8mb4_general_ci ;
```

### mysqladmin 创建

使用 mysqladmin 创建数据库 mysqladmin 是 MySQL 提供的一个用于执行管理任务的命令行工具。

通过 mysqladmin，你可以执行各种数据库管理操作，包括创建数据库。

使用 mysqladmin 创建数据库的基本语法：

```bash
mysqladmin -hlocalhost -pyourpass create your_database
```

参数含义：

    -u 参数用于指定 MySQL 用户名。
    -p 参数表示需要输入密码。
    create 是执行的操作，表示创建数据库。
    your_database 是要创建的数据库的名称。

> 指定字符集和排序规则，可以使用 -default-character-set 和 -default-collation 参数：

```bash
mysqladmin -u your_username -p create your_database \
  --default-character-set=utf8mb4 \
  --default-collation=utf8mb4_general_ci
```

使用普通用户，你可能需要特定的权限来创建或者删除 MySQL 数据库。

可以使用 root 用户登录，root 用户拥有最高权限，可以使用 mysql mysqladmin 命令来创建数据库。

其他：

使用 mysqladmin 连接到 MySQL 服务器执行其他管理任务，例如查看服务器状态、重启服务器等，可以使用以下形式的命令：
```bash
mysqladmin -u your_username -p your_command
```

在这里，your_command 是你希望执行的具体管理命令。

例如，要查看 MySQL 服务器的状态，可以使用：
```bash
mysqladmin -u your_username -p status
```

这将要求你输入密码，并显示有关服务器状态的信息。

## MySQL 用户设置


### 创建用户

要创建一个新用户，你可以使用以下 SQL 命令：

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

   - username：用户名。
   - host：指定用户可以从哪些主机连接。例如，localhost 仅允许本地连接，% 允许从任何主机连接。
   - password：用户的密码。

实例
```sql
CREATE USER 'bp_demo'@'localhost' IDENTIFIED BY 'password123';
```

### 授权权限

创建用户后，你需要授予他们访问权限，使用 GRANT 命令来授予权限：

```sql
GRANT privileges ON your_schema_name.* TO 'user_name'@'host';
```
   - privileges：所需的权限，如 ALL PRIVILEGES、SELECT、INSERT、UPDATE、DELETE 等。
   - database_name.*：表示对某个数据库或表授予权限。database_name.* 表示对整个数据库的所有表授予权限，database_name.table_name 表示对指定的表授予权限。
   - TO 'username'@'host'：指定授予权限的用户和主机。

实例

```sql
GRANT ALL PRIVILEGES ON bp_demo.* TO 'bp_user'@'localhost';
```

> All或者All privileges代表权限列表中除Grant option权限之外的所有权限

```sql
show privileges ;
```


### 刷新权限

授予或撤销权限后，需要刷新权限使更改生效：

```sql
FLUSH PRIVILEGES;
```

看看  ALL PRIVILEGES 有哪些：
```sql
show privileges ;
```

更细化的授权：

```sql
GRANT  Alter routine, Execute,
SELECT, INSERT, UPDATE, delete, Alter,
create, -- To create new databases and tables
Create routine, -- To use CREATE FUNCTION/PROCEDURE
Create temporary tables  -- To use CREATE TEMPORARY TABLE
ON bp_demo.* TO 'bp_demo_user'@'%' IDENTIFIED BY 'small.rose@2025' 
       -- with grant option ;
```

### 查看用户权限

要查看特定用户的权限，可以使用以下命令：
```sql
SHOW GRANTS FOR 'username'@'host';
```


实例
```sql
SHOW GRANTS FOR 'bp_user'@'localhost';
```




### 撤销权限

要撤销用户的权限，使用 REVOKE 命令：


```sql
REVOKE privileges ON database_name.* FROM 'username'@'host';
```
实例

```sql
REVOKE ALL PRIVILEGES ON bp_demo.* FROM 'bp_user'@'localhost';
```

```sql
     
-- 撤销用户授予其他用户的特定权限的GRANT OPTION
REVOKE GRANT OPTION ON bp_demo.* FROM 'username'@'host';

--撤销用户username将任何权限授予其他用户的GRANT OPTION 权限
REVOKE GRANT OPTION ON bp_demo.table FROM 'username'@'host';

-- 移除用户授予其他用户的所有权限的GRANT OPTION
REVOKE ALL PRIVILEGES ON *.* FROM 'username'@'host' WITH GRANT OPTION;

-- 移除GRANT OPTION
REVOKE GRANT OPTION ON *.* FROM 'username'@'host';

-- 重新授予其他需要的权限（如果需要）
GRANT SELECT, INSERT ON database.table TO 'username'@'host';
```



### 删除用户

如果需要删除用户，可以使用以下命令：

```sql
DROP USER 'username'@'host';
```

实例

```sql
DROP USER 'bp_user'@'localhost';
```

### 修改用户密码

要修改用户的密码，可以使用 ALTER USER 命令：
```sql
ALTER USER 'username'@'host' IDENTIFIED BY 'new_password';
```

实例
```sql
ALTER USER 'bp_user'@'localhost' IDENTIFIED BY 'newpassword456';
```

### 修改用户主机

要更改用户的主机（即允许从哪些主机连接），可以先删除用户，再重新创建一个新的用户。

实例
```sql
-- 删除旧用户
DROP USER 'bp_user'@'localhost';

-- 重新创建用户并指定新的主机
CREATE USER 'bp_user'@'%' IDENTIFIED BY 'password123';
```


###  创建用户时指定权限

在创建用户时，也可以同时授予权限（在 MySQL 8.0.16 及更高版本）：

```sql
CREATE USER 'bp_user'@'localhost' IDENTIFIED BY 'password123' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON test_db.* TO 'bp_user'@'localhost';
```

## 其他命令

列出所有可用的数据库：
```sql
SHOW DATABASES;
```

选择要使用的数据库：

```sql
USE your_database;
```
列出所选数据库中的所有表：

```sql
SHOW TABLES;
```

## /etc/my.cnf 文件配置

一般情况下，不需要修改该配置文件，该文件默认配置如下：

```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

[mysql.server]
user=mysql
basedir=/var/lib

[safe_mysqld]
err-log=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

1. 基本设置

- basedir: MySQL 服务器的基本安装目录。
- datadir: 存储 MySQL 数据文件的位置。
- socket: MySQL 服务器的 Unix 套接字文件路径。
- pid-file: 存储当前运行的 MySQL 服务器进程 ID 的文件路径。
- port: MySQL 服务器监听的端口号，默认是 3306。

2. 服务器选项

 - bind-address: 指定 MySQL 服务器监听的 IP 地址，可以是 IP 地址或主机名。
 - server-id: 在复制配置中，为每个 MySQL 服务器设置一个唯一的标识符。
 - default-storage-engine: 默认的存储引擎，例如 InnoDB 或 MyISAM。
 - max_connections: 服务器可以同时维持的最大连接数。
 - thread_cache_size: 线程缓存的大小，用于提高新连接的启动速度。
 - query_cache_size: 查询缓存的大小，用于提高相同查询的效率。
 - default-character-set: 默认的字符集。
 - collation-server: 服务器的默认排序规则。

3. 性能调优

 - innodb_buffer_pool_size: InnoDB 存储引擎的缓冲池大小，这是 InnoDB 性能调优中最重要的参数之一。
 - key_buffer_size: MyISAM 存储引擎的键缓冲区大小。
 - table_open_cache: 可以同时打开的表的缓存数量。
 - thread_concurrency: 允许同时运行的线程数。

4. 安全设置

 - skip-networking: 禁止 MySQL 服务器监听网络连接，仅允许本地连接。
 - skip-grant-tables: 以无需密码的方式启动 MySQL 服务器，通常用于恢复忘记的 root 密码，但这是一个安全风险。
 - auth_native_password=1: 启用 MySQL 5.7 及以上版本的原生密码认证。

5. 日志设置

 - log_error: 错误日志文件的路径。
 - general_log: 记录所有客户端连接和查询的日志。
 - slow_query_log: 记录执行时间超过特定阈值的慢查询。
 - log_queries_not_using_indexes: 记录未使用索引的查询。

6. 复制设置

 - master_host 和 master_user: 主服务器的地址和复制用户。
 - master_password: 复制用户的密码。
 - master_log_file 和 master_log_pos: 用于复制的日志文件和位置。
