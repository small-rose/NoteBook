---
layout: default
title: MySQL Command
parent: MySQL
grand_parent: Database
nav_order: 86
---


## Mysql 常用命令

> 虽然现在已经有很多界面化的连接工具可以直接鼠标操作。但是若环境不允许远程连接怎么呢？所以常用命令还是有用武之地的。整理一下。

### 一、连接命令

```bash
## mysql终端连接
mysql  [ -h  <hostname | ip> ] -u <username> [-P <port>] -p [<userpass>]
```

参数含义：

**-h:** 主机名，表示连接的目标数据库主机名或者IP地址。

**-u:** 用户名，表示连接的目标数据库的用户名。

**-P:** 端口，表示连接的目标数据库的端口，不写默认是3306，非默认须确定端口号。

**-p:** 表示连接的目标数据库的密码，-p后面可以直接输入密码，也可以不写执行命令的时候输入密码，下文同理。

关于 `-u`和`username`是否一定要连在一起，并没有找到答案。

### 二、基础查询命令

执行登录之后使用。

```mysql
-- 列出全部已有数据库
show databases;

-- 创建新数据库
create database <database_name>; 

-- 创建指定数据库
drop database <database_name>;

-- 打开/使用指定数据库
use <database_name>;

-- 列出打开的数据库所有的表名
show tables;

-- 查看数据库指定的表结构
desc <table_name>;

-- 删除表
drop table <table_name>;
```

### 三、数据库备份

#### 1、直接备份库

（A）备份一个库

```bash
## 备份指定主机某个库
mysqldump -h host_name -P port -u user_name -p user_pass –database database_name > bak_file_name.sql

## 备份本地数据库，执行时输密码
mysqldump -u user_name –p –database database_name  >  bak_file_name.suffix
```

实例如：

```bash
## 备份192.168.66.82上的数据库DBCM
mysqldump -h 192.168.66.82 -u root –p123456  –database dbcm  >  dbcm_20200617.sql

## 备份数据库DBCM，执行时输密码
mysqldump -u root –p  –database dbcm  >  dbcm_20200617.sql
```

（B）备份多个库:

```bash
## 备份指定主机里的多个库
mysqldump -h hostname -u username -p user_pass –databases db_name1 db_name2 db_name3 > more_dbname_file.suffix
```

如备份系统自带库：

```bash
## 备份192.168.66.82上的数据库自带系统库
mysqldump -h 192.168.66.82 -u root –p123456  –databases information_schema mysql > sys_bak.sql
```

说明：后续列举就不再分别列远程与本地了，区别只是`-h host_name`及`-P host_port`，统一用本地列举。

#### 2、直接备份库的表

备份指定库的指定表 

```bash
mysqldump -u user_name –p –database database_name  table_name1  table_name2 >  bak_fileName.suffix
```

例如：

```bash
##备份数据库DBCM里的loginfo表
mysqldump -u root –p –database dbcm loginfo  >  dbcm_loginfo_20200617.sql
## 多张表
mysqldump -u root –p –database dbcm loginfo user >  dbcm_log_user_20200617.sql
```

#### 3、压缩备份

```bash
mysqldump -h host_name -u user_name -p user_pass -database db_name | gzip > backup_file.sql.gz
```

#### 4、只备份数据库表结构

```bash
mysqldump  –no-data -u user_name -p user_pass –databases db_name1 db_name2 db_name3 > db_table_bak.sql
```

#### 5、备份所有数据库

```bash
mysqldump –all-databases -u user_name -p user_pass > all_db.sql
```

#### 6、带删除的备份

备份`MySQL`数据库为带删除表的格式，能够让该备份覆盖已有数据库而不需要手动删除原有数据库

通俗的说就是`CREATE TABLE table_name `之前有`DROP TABLE IF EXISTS table_name`的动作。mysql默认备份就支持。

```bash
mysqldump --add-drop-table -u username -p password -database databasename > backup_file.sql
```

#### 7、binlog增量备份

增量备份是使用`mysql`的`binlog`日志作为记录新增的变化。

`--master-data `：

在`mysqldump`命令中使用`--master-data=2`，会记录`binlog`文件和`position`的信息。

`--single-transaction`：

`--single-transaction`会将隔离级别设置成`repeatable-commited`

首先进行全备

```
mysqldump -u username -p password --single-transaction --master-data=2 db_name > backup_all_file.sql
```

备份文件中会有

`-- CHANGE MASTER TO MASTER_LOG_FILE='bin-log.000002', MASTER_LOG_POS=xxx;`

是指备份后所有的更改将会保存到新文件bin-log.000002二进制文件中。

### 四、数据库还原

#### 1、直接还原

```bash
mysql -h hostname -u user_name -p user_pass database_name < backup_file.sql
```

#### 2、压缩还原

```bash
gunzip < backup_file.sql.gz | mysql -u user_pass -p password database_name
```

#### 3、source导入

使用source命令，需要使用命令登录mysql，用use进入到某个数据库后执行

```mysql
source d:\backup_file.sql
```

#### 4、数据库转移

```bash
mysqldump -u user_name -p user_pass db_name | mysql –host=*.*.*.* -C db_name
```

#### 5、增量还原/恢复

首先导入全备数据，source亦可。

```bash
mysql -h host_name -u user_name -p user_pass  < backup_all_file.sql
```

然后，恢复bin-log.000002

```bash
mysqlbinlog bin-log.000002 | mysql -h host_name -u user_name -p user_pass
```

如果还有后续新增同理恢复。

恢复部分，恢复到某个操作的前面一个position点。

控制`binlog`的区间的参数有：

```mysql
--start-position  # 开始点 
--stop-position # 结束点
--start-date  # 开始时间 
--stop-date  # 结束时间
```

找到需要操的恢复点，后进行恢复

```bash
mysqlbinlog mysql-bin.000003 --stop-position=308 |mysql -h host_name -u user_name -p user_pass 
```

找到需要的恢复时间区间，进行恢复指定时间区间数据

 ```bash
mysqlbinlog mysql-bin.000003 --start-datetime='2019-12-01 00:00:00' --stop-datetime='2019-12-31 23:59:59' |mysql -h host_name -u user_name -p user_pass
 ```

查看`binlog`内容，可以将`binlog`内容写入到临时文件查看：

```
mysqlbinlog --no-defaults --database=db   --base64-output=decode-rows -v --start-datetime='2019-12-01 00:00:00' --stop-datetime='2019-12-31 23:59:59'  mysql-bin.000003 >  binlog003.sql
```

### 五、授权管理

经常见到或自己设置使用的授权长这样：

```mysql
##给指定root用户在本机登录连接及操作所有Schema的所有表的权限
grant all privileges on *.* to 'root'@'localhost' identified by 'user_pass';
## 刷新权限设置
flush privileges;
```

实际使用的时候可能会进行一些更细节的授权限制，那么怎么办呢？

#### 1、查看用户权限

```mysql
##查看当前登录用户权限
show grants;
## 查看其他 MySQL 用户权限,注意user_name和localhost要加引号
show grants for user_name@localhost;
```



#### 2、基本授权命令

```mysql
grant <privileges>  on schema.table to user_name@host_name identified by user_pass
```

**privileges** ：包括常见的查询、插入、更新、删除操作外，还包括创建表、索引、视图、存储过程、函数等权限。使用时关键字 “privileges” 可以省略。

**schema.table** ：可以细节到指定库指定表，`*.*` 表示所有的库下所有的表。

**user_name@host_name identified by user_pass**：表示允许哪个用户使用指定密码从哪个主机登录；常见的写法如下：（`identified by user_pass`实际使用时记得加上）

| 写法                   | 含义                                                 |
| ---------------------- | ---------------------------------------------------- |
| 'root'@'localhost'     | 允许root用户从本机连接登录                           |
| 'root'@'%'             | 允许root用户从任意机器连接登录                       |
| 'root'@'192.168.0.221' | 允许root用户从`IP`为` 192.168.0.221`的机器连接登录   |
| 'root'@'192.168.0.%'   | 允许root用户从`IP`网段为` 192.168.0.*`的机器连接登录 |

> 不写@选项时，效果与加@'%'是一样。
>
> '%'从名义上包括任何主机，据说有些版本'%'不包括`localhost`，则需要单独对`@'localhost'`进行赋值。

（1）给普通用户授权

普通用户一般可以授权基本数据的查询、写入、修改、删除操作，修改、删除视实际要求情况授权。

`bus_dbname` ：表示业务数据库。`common_user`：普通用户连接账户。

```mysql
## 分别授权
grant select on bus_dbname.* to common_user@'%'
grant insert on bus_dbname.* to common_user@'%'
grant update on bus_dbname.* to common_user@'%'
grant delete on bus_dbname.* to common_user@'%'

## 合并操作，使用英文逗号分隔
grant select, insert, update, delete on bus_dbname.* to common_user@'%'
```

（2）给高权用户授权

如开发人员，`DBA`等，允许建表结构，删表，改表结构，改外键，建视图，临时表，索引，存储过程等。

`bus_dbname` ：表示业务数据库。`dev_userr`：开发或管理用户连接账户。

```mysql
## 授权dev_user用户 在MySQL的数据库bus_dbname中建表操作 
grant create on bus_dbname.* to dev_user@'192.168.0.%';
## 授权 改表
grant alter on bus_dbname.* to dev_user@'192.168.0.%';
## 授权 删表
grant drop on bus_dbname.* to dev_user@'192.168.0.%';

## 授权 外键权限。
grant references on bus_dbname.* to dev_user@'192.168.0.%';

## 授权 临时表权限。
grant create temporary tables on bus_dbname.* to dev_user@'192.168.0.%';
## 授权 索引权限。
grant index on bus_dbname.* to dev_user@'192.168.0.%';

## 授权 视图创建。
grant create view on bus_dbname.* to dev_user@'192.168.0.%';
## 授权 视图查看代码。
grant show view on bus_dbname.* to dev_user@'192.168.0.%';

##给普通 DBA/dev_user授权
grant all privileges on bus_dbname.* to dev_user@'localhost'
##可以省略privileges
grant all on bus_dbname.* to dev_user@'localhost' 

##其实是给dev_user授权所有库和所以表，一般可以给DBA
grant all on *.* to dev_user@'localhost'
```

关于存储过程和函数的使用关键字：`create routine `。

授权`create  routine`表示可以创建`procedure` 和` function` 。

如果用户创建了`procedure`或`function`那么`mysql `会自动赋予该用户对`procedure` 或 `function` 的`alter routine`和`execute` 权限。

```mysql
grant create routine on bus_dbname.* to dev_user@'192.168.0.%'; 
```

创建存储过程：

```mysql
delimiter $
create procedure sp_hello_mysql()
begin
    select 'hello mysql';
end 
$
```

再去查询授权

```mysql
show grants;
```

就可以看到用户`alter routine`和`execute` 权限。

（3）授权分层

`grant`的授权可以授权给整个`mysql`库，也可以指定某个库，指定某个库的某个表，指定表的列，指定视图、指定函数、指定存储过程等

```mysql
## 授权 MySQL中的所有数据库下的所有表，也就是完整的全部权限
grant all on *.* to 'root'@'localhost'; 

## 授权user_name 给某个指定的数据库的查询权限
grant select  on bus_dbname.* to user_name@localhost; 

## 授权 给某个指定的数据库的指定表select, insert, update, delete权限
grant select, insert, update, delete on bus_dbname.table_name to user_name@localhost; 

## 授权 给某个指定的数据库的指定表查询指定列的权限
grant select(id, name, rank) on bus_dbname.table_name to user_name@localhost;

## 授权 指定的存储过程执行权限
grant execute on procedure bus_dbname.pr_procname to user_name@localhost;
## 授权 指定的函数执行权限
grant execute on function bus_dbname.fn_funcname to user_name@localhost;
```



#### 3、撤销授权命令

撤销授权使用`revoke`命令。

`revoke` 和`grant` 的语法相似，需要把关键字 `to` 换成 `from`,命令格式：

```mysql
## 授权dev_user用户在本地登录后的 操作所有库所有表的所有权限
grant all on *.* to dev_user@localhost ;
## 撤销dev_user用户在本地登录后的 操作所有库所有表的所有权限
revoke all on *.* from dev_user@localhost;
```



#### 4、注意事项

（1）执行`grant`,` revoke` 操作权限后，相关用户，重新连接` MySQL` 数据库后，权限才能生效。

（2）用户被授予了某个权限，那么默认情况下，该用户是不能把这个权限授予给其他人的.。

可以使用`WITH GRANT OPTION`这个子句来让该用户可以将权限再授予给其他人。

`WITH GRANT OPTION`：表示允许权限传递。用于对象授权。区别于 用于系统权限授权的`with admin option`。

还可以通过直接授予GRANT OPTION权限来达到这个效果。

```mysql
##使用 WITH GRANT OPTION允许权限传递
grant select, insert, update, delete on bus_dbname.table_name to user_name@localhost WITH  GRANT OPTION; 

## 直接授权GRANT OPTION
grant select,grant option on bus_dbname.table_name to user_name@localhost ; 
```

（3）权限传递`WITH  GRANT OPTION`用户的权限被收回时，其他被传递的相同权限自动收回。

（4）权限信息用user、db、host、`tables_priv`和`columns_priv`表被存储在`mysql`数据库中





