---
layout: default
title: MySQL
nav_order: 1
parent: Database
---

# MySQL
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

MYSQL URL参数
--------------

  | 参数名称        |  参数说明      |       
 | ----------- | ----------- |                  
 | user          |              数据库用户名（用于连接数据库）
 | passWord      |              用户密码（用于连接数据库）
 | useUnicode    |              是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为true 
 | autoReconnect          |     当数据库连接异常中断时，是否自动重新连接？
 | autoReconnectForPools  |     是否使用针对数据库连接池的重连策略
 | maxReconnects          |     autoReconnect设置为true时，重试连接的次数 
 | failOverReadOnly       |     自动重连成功后，连接是否设置为只读？

对应中文环境，通常mysql连接URL可以设置为：
```text
jdbc:mysql://localhost:3306/test?user=root&password=&useUnicode=true&characterEncoding=utf8&autoReconnect=true&failOverReadOnly=false
```

在使用数据库连接池的情况下，最好设置如下两个参数：
```text
autoReconnect=true&failOverReadOnly=false
```


需要注意的是，在xml配置文件中，url中的&符号需要转义成&amp;。比如在tomcat的server.xml中配置数据库连接池时，mysql jdbc url样例如下：
```text
jdbc:mysql://localhost:3306/test?user=root&amp;password=&amp;useUnicode=true&amp;characterEncoding=utf8&amp;autoReconnect=true
```

使用连接池

```text
autoReconnect=true&failOverReadOnly=false 
```


MySQL 改密码策略，改秘密
----------------------------

修改：密码最小长度策略
```
set global validate_password_length=0;
```
修改：密码强度检查等级策略，0/LOW、1/MEDIUM、2/STRONG
```
set global validate_password_policy=0;
```

**记得密码改密码 MYSQL5.7**

（1）直接改

```sql
mysqladmin -uroot -pshapolang password 123456
```
（2）登录之后改
```SQL
--登录
mysql -uroot -p
--切換mysql
use mysql;
--设置新密码。
update mysql.user set authentication_string=password('新密码') where user='用户名' and Host ='localhost'; 
--刷新权限。
flush privileges; 
```


**忘记密码改密码 MYSQL5.7**

（1）进行免密登录

在mysql的配置文件中找到[mysqld]然后添加 `skip-grant-tables`,然后直接`mysql -uroot -p` 免密登录。

（2）修改密码

``` SQL
set password for 'root'@'localhost' = password('123456');
## -------------------------------------------------------
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
## --------------------------------------------------------
flush privileges;
```


重新登录进行验证.

**忘记密码改密码 MYSQL 8**

（1）进行免密登录

在mysql的配置文件中找到[mysqld]然后添加 `skip-grant-tables`,然后直接`mysql -uroot -p` 免密登录。

（2）修改密码

```SQL
use mysql; 

--将字段置为空
update user set authentication_string='' where user='root';

--修改密码为root
ALTER user 'root'@'localhost' IDENTIFIED BY 'root';
-- 刷新权限
flush privileges;
```
   

MySQL无密码登录

Windows

1.打开命令窗口cmd，输入命令：`net stop mysql`，停止MySQL服务，

2.开启跳过密码验证登录的MySQL服务

输入命令  

```text
mysqld --console --skip-grant-tables --shared-memory 
```
        
3.新开cmd 窗口 
```text
mysql -uroot -p
```

Linux


MySQL查表和注释
----------------------------


```sql
SELECT table_name 表名,TABLE_COMMENT 表注释
FROM INFORMATION_SCHEMA.TABLES
WHERE table_schema = 'sfdev01' 
AND table_name IN ( SELECT table_name FROM information_schema.tables WHERE table_schema='sfdev01')
AND table_name IN ( 'mm_batchinfo_ti','mm_batchinfo_td','mm_policy_legal','mm_accperiod_td');
```



MySQL查表字段注释
----------------------------

```sql
SELECT
    c.table_schema 库名,
    c.table_name 表名,
    c.COLUMN_NAME 字段名,
    c.column_comment 字段说明,
    c.column_type 字段类型,
    c.column_key 约束,
       c.COLUMN_DEFAULT 默认值,
       c.*
FROM information_schema.columns c
WHERE c.table_schema = 'bp_dev'
AND c.table_name = 'mm_match_dest_td'
-- AND c.COLUMN_DEFAULT is not null ;
```

MySQL注释补偿
---------------------

```sql

ALTER TABLE table_name COMMENT '新的表注释';

# 只修改注释
ALTER TABLE table_name MODIFY COLUMN field_name COMMENT '修改后的字段注释';

# 修改字段类型也修改注释
ALTER TABLE table_name MODIFY COLUMN field_name varchar(30) COMMENT '修改后的字段注释';
```


MySQL字段补偿
----------------------------

table_name 就是实际的表名； field_name 就是实际的字段名。

**添加字段**

```sql
# 追加新的字段信息
ALTER TABLE  table_name ADD  field_name  varchar(20) COMMENT '字段注释';
```

**修改字段**

(1) 不修改名称 使用modify

```sql
# 只改类型和注释
ALTER TABLE  table_name  MODIFY field_name varchar(20) NOT NULL COMMENT '用户名';
```

(2) 修改名称 使用change  格式 是 change 要修改的名称 新名称 ...

```sql
# 将已有名字改成新名字
ALTER TABLE table_name CHANGE old_field_name new_field_name varchar(20) NOT NULL COMMENT '用户名';
```

**删除字段**

```sql
# 删除某个字段
ALTER  TABLE  table_name  DROP COLUMN field_name;
# 删除多个字段
ALTER  TABLE  table_name  DROP COLUMN field_name1, DROP COLUMN field_name2;
```



MySQL 取某张表的下一个自增值
----------------------------

```sql
select AUTO_INCREMENT from INFORMATION_SCHEMA.TABLES    
where TABLE_NAME='your_table_name'  
```



MySQL 日期处理
----------------------------

```sql
# 日期转字符串
SELECT DATE_FORMAT(CURRENT_DATE,'%Y%m%d');

# 字符串转日期
SELECT STR_TO_DATE(DATE_FORMAT(CURRENT_DATE,'%Y%m%d'),'%Y%m%d');

# MySQL不支持年月这种
SELECT STR_TO_DATE(DATE_FORMAT(CURRENT_DATE,'%Y%m'),'%Y%m');
```



MySQL检查是否自动提交事务
----------------------------

（navicat 、idea 2020 已验证，其他工具应该也类似）

```sql
show variables like 'autocommit';

# 开启自动提交事务
set autocommit=on;

# 关闭自动提交事务
set autocommit=off;
```

查表
```sql
show create table bp_test.t_log;
```



Navicat SQL的保存位置
----------------------------

```txt
C:\Users\{username}\Documents\Navicat\MySQL\servers\
```

在mysql中使用show collation指令可以查看到mysql所支持的所有COLLATE

```sql
show collation;

SHOW VARIABLES LIKE '%character_set%';

SHOW CHARACTER SET;
```



MySQL索引重复率查询
----------------------------

```sql
SELECT
t.TABLE_SCHEMA,t.TABLE_NAME,INDEX_NAME, CARDINALITY,
TABLE_ROWS, CARDINALITY/TABLE_ROWS AS SELECTIVITY
FROM
information_schema.TABLES t,
(
SELECT table_schema,table_name,index_name,cardinality
FROM information_schema.STATISTICS
WHERE (table_schema,table_name,index_name,seq_in_index) IN (
SELECT table_schema,table_name,index_name,MAX(seq_in_index)
FROM information_schema.STATISTICS
GROUP BY table_schema , table_name , index_name )
) s
WHERE
t.table_schema = s.table_schema
AND t.table_name = s.table_name AND t.table_rows != 0
AND t.table_schema = 'bp_dev'
AND t.TABLE_NAME='T_APPLICATIONS'
ORDER BY SELECTIVITY; 
```

在OLTP的应用场景下，创建的索引是要求高选择性的。若CARDINALITY / TABLE_ROWS小于10%（经验值），那么表示数据重复率较高，通常需要考虑是否有必要创建该索引。该语句运行的结果如下所示，列SELECTIVITY表示的就是选择性：

| TABLE_SCHEMA | TABLE_NAME | INDEX_NAME | CARDINALITY | TABLE_ROWS | SELECTIVITY |
| ---- | ---- | ---- | ---- | ---- | ---- |
|bp_dev	| t_applications	| MER_ORDERID	| 1	| 1380272	| 0.0000 |
|bp_dev	| t_applications	| FK_B_APPLIC2	| 3	| 1380272	| 0.0000 |
|bp_dev	| t_applications	| PRIMARY	| 1380272	| 1380272	| 1.0000 |
|bp_dev	| t_applications	| IDX_T_APPLICATIONS_01	| 1380272	| 1380272 | 1.0000|



MySQL查锁
----------------------------

```sql
SELECT  * FROM  INNODB_LOCKS;

SELECT * FROM INNODB_LOCK_WAITS;

SELECT  * FROM  INNODB_TRX;

SELECT * FROM `PROCESSLIST` WHERE ID IN (SELECT TRX_MYSQL_THREAD_ID FROM INNODB_TRX )

#显示当前正在执行的mysql连接
show processlist;
```



MySQL的show status和show variables
----------------------------
status && variables
- 凡是参数的开头字母是大写的参数，都需要使用show status命令来查看该参数的具体值是多少，用另外一个命令查看该参数输出内为空。
- 凡是参数的开头字母是小写的参数，都需要使用show variables命令来查该参数的具体值是多，用另外一个命令查看该参数输出内为空。
区别

**show status**

status查看的参数值是由MySQL自己统计计算得到的。它是MySQL服务运行状态具体的量化体现。都是不可以修改的，也就是不能通过set xxx=yyy;的方式来改变它的值的。这种参数大多数以**大写英文字母**开头。

**show variables**

variables查看的参数是MySQL服务的配置参数，在启动MySQL服务的时候，是可以修改具体的参数值来达到对MySQL进行动态配置的目的，通常配置在MySQL的my.cnf配置文件中。这些参数中，有些动态的参数可以通过set xxx=yyy;的方式来动态修改。这种参数大多数以小写的英文字母开头。


> 命令带有关键词global，上面的两个命令不带有global，这个我们应该都知道，带有global关键字的命令是查看全局参数的值，而不带global关键字的命令是只查看当前session级别的参数的值。
> 如: show global status like '%xxx%';
>     show global variables like '%xxx%';

show status  查看所有状态参数
- Threads_connected 当前的连接数，
- Connections 试图连接到（不管是否成功）MYSQL服务器的连接总数， 
- Max_used_connections 服务器启动后已经同时使用过的连接最大数量（并发）。

```sql
# 查连接相关状态
show  status like '%connect%';
# 查连接相关参数
show variables like '%connect%';


# 最大已使用连接数
show global status like 'Max_used_connections';
# 查最大连接数参数
show variables like 'max_connections';
show global variables like 'max_connections';
# 修改值
set global max_connections = 200 ;



# 查线程相关状态
show status like 'Threads%';
# 查线程相关配置
show  variables like '%thread%';


#交互式连接超时时间(mysql工具、mysqldump等)
show variables like 'interactive_timeout';

# 非交互式连接超时时间，默认的连接mysqlapi程序,jdbc连接数据库等
show variables like 'wait_timeout';


#显示mysql的其他状态
mysqladmin -uroot -p -hlocalhost extended-status
```

线程相关状态参数含义：
- Threads_cached:一共缓存过多少连接，如果在mysql服务器的配置文件中设置了thread_cache_size参数，当前客户端断开之后，服务器处理此客户的线程将会缓存起来，以备下一个连接进来的客户而不是立即销毁（前提是缓存数量未达到上限），可以通过命令：set global thread_cache_size=50 设置或者在 my.cnf中配置
- Threads_connected: 代表当前已经有多少个连接(sleep+query)
- Threads_created:历史总共创建过多少个连接
- Threads_running:代表有几个连接正处于"工作"状态，也是目前的并发数 

关于连接数

对于mysql服务器最大连接数值的设置范围比较理想的是：服务器响应的最大连接数值占服务器上限连接数值的比例值在10%以上，如果在10%以下，说明mysql服务器最大连接上限值设置过高.
```text
Max_used_connections / max_connections * 100% （理想值≈ 85%）
```
如果max_used_connections跟max_connections相同 那么就是max_connections设置过低或者超过服务器负载上限了，低于10%则设置过大


MySQL查表使用空间情况
----------------------------

```sql
select table_name, truncate(sum(data_length)/1024/1024,2) as data_size_MB,
truncate(sum(index_length)/1024/1024,2) as index_size_MB
from information_schema.tables where table_schema = 'bp_test'
group by table_name
order by data_size_MB desc; 
```


MySQL数据库备份
--------------------------

1、直接备份库

语法：

（A）备份一个库

```bash
## 备份指定主机某个库
mysqldump -h host_name -P port -u user_name -p user_pass –database database_name > bak_file_name.sql

## 备份本地数据库，执行时输密码
mysqldump -u user_name –p –database database_name  >  bak_file_name.suffix
```

实例：

```bash
## 备份192.168.66.82上的数据库DBCM
mysqldump -h 192.168.66.82 -u root –p123456  –database dbcm  >  dbcm_20200617.sql

## 备份数据库DBCM，执行时输密码
mysqldump -u root –p  –database dbcm  >  dbcm_20200617.sql
```

（B）备份多个库

```bash
## 备份指定主机里的多个库
mysqldump -h hostname -u username -p user_pass –databases db_name1 db_name2 db_name3 > more_dbname_file.suffix
```

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

3、压缩备份

```BASH
mysqldump -h host_name -u user_name -p user_pass -database db_name | gzip > backup_file.sql.gz
```

4、只备份数据库表结构

```bash
mysqldump  –no-data -u user_name -p user_pass –databases db_name1 db_name2 db_name3 > db_table_bak.sql
```

5、备份所有数据库
```bash
mysqldump –all-databases -u user_name -p user_pass > all_db.sql
```

MySQL常见函数
---------------------

```sql
-- MySQL 取日期所在月的第一天
select DATE_ADD(curdate(),interval -day(curdate())+1 day);
-- MySQL 取日期所在月的最后一天
select last_day(curdate()) from dual;
```




MySQL 存储过程
---------------------


原来是根据 [oracle检测中文乱码](https://docs.zhangxiaocai.cn/Database/DB.Oracle.html#%E6%A3%80%E6%B5%8B%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81)
适配mysql，发现函数不适用，不好判断，就当一个demo了

```sql

drop procedure if exists check_zh_cn ;

create procedure check_zh_cn()
begin

    DECLARE v_table_name  varchar(50) ;
    DECLARE v_table_column  varchar(50) ;
    DECLARE v_sql  varchar(1000) ;
    DECLARE v_count  int(10) default 0;
    DECLARE v_index  int(10) default 0;
    DECLARE isFlag INT DEFAULT TRUE;
    -- 查询所有的表
    DECLARE v_table_names cursor for
        SELECT table_name FROM information_schema.tables WHERE table_schema='bp_demo';
    -- 查询某张表的所有列
    DECLARE v_table_columns cursor for
        select v_table_column FROM information_schema.columns c WHERE c.table_schema = 'bp_demo' AND c.table_name = v_table_name order by ORDINAL_POSITION asc ;

    -- #游标中的内容循环执行完后将 isFlag 设置为flase
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET isFlag = FALSE;
    -- 打开声明的游标
    open v_table_names ;
    loop_label:loop
        -- 取出游标中的变量,
        fetch v_table_names into v_table_name ;
        select v_table_name ;
        while isFlag  is TRUE DO
            -- 每次循环开始时重新对num进行赋值
            -- set @num = 0;
            -- 开启第二个游标
            open v_table_columns;
            inner_loop: LOOP
                fetch v_table_columns into v_table_column;
                -- 打印过程
                -- select num;
                IF isFlag THEN
                    LEAVE inner_loop;
                END IF;
                select v_table_name,v_table_column ;

            END LOOP;
            close v_table_columns;
            -- 第一个循环结束时done=1
            -- 故需手动设置done=0,否则外层循环仅会执行一次
            SET isFlag = 0;
        end while;
    end loop;
end;

```
两个存过均需要重新验证

```sql
create
    definer = root@`37.220.51.%` procedure check_zh_cn()
begin

    DECLARE v_table_name  varchar(50) ;
    DECLARE v_table_column  varchar(50) ;
    DECLARE v_sql  varchar(1000) ;
    DECLARE v_count  int(10) default 0;
    DECLARE v_index  int(10) default 0;
    DECLARE isFlag INT DEFAULT TRUE;

    declare r_table_name varchar(20) default 'CHECK_ZHCN_RESULT';
    -- 查询所有的表
    DECLARE v_table_names cursor for
        SELECT table_name FROM information_schema.tables WHERE table_schema='hrbyq';
    -- 查询某张表的所有列
    DECLARE v_table_columns cursor for
        select COLUMN_NAME FROM information_schema.columns c WHERE c.table_schema = 'hrbyq' AND c.table_name = v_table_name order by ORDINAL_POSITION asc ;

     -- #游标中的内容循环执行完后将 isFlag 设置为flase
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET isFlag = FALSE;

    set @d_sql = concat('drop table ',r_table_name);
    set @c_sql = concat(concat('create table ',r_table_name),' (table_name varchar(100), column_name  varchar(100))');
    PREPARE d_sql from  @d_sql ;
    PREPARE c_sql from @c_sql;

    -- 结果表
    SELECT count(1) into v_count FROM information_schema.TABLES where TABLE_NAME = r_table_name;
    IF v_count > 0 then
        execute  d_sql ;
        execute  c_sql ;
    else
        execute  c_sql ;
    end IF;

    -- 打开声明的游标
    open v_table_names ;
    loop_label:loop
        -- 取出游标中的变量,
        fetch v_table_names into v_table_name ;
        insert into procedure_log select concat('tableName is ', v_table_name);
        while isFlag  is TRUE DO
            -- 每次循环开始时重新对num进行赋值
            -- set @num = 0;
            -- 开启第二个游标
            open v_table_columns;
            inner_loop: LOOP
                fetch v_table_columns into v_table_column;
                -- 打印过程
                -- select num;
                IF isFlag THEN
                    LEAVE inner_loop;
                END IF;
                set @v_sql = concat('select count(1) into @count from ', v_table_name , ' where hex(convert(',v_table_column ,' using ucs2) )  like ''%FFFD%'' ');
                insert into procedure_log select concat('@v_sql is ', @v_sql);
                prepare v_sql from @v_sql;
                execute  v_sql  ;
                if @count >0 then
                        set v_index := v_index+1;
                        set @i_sql = concat('insert into ',r_table_name,' (table_name, column_name) values (',v_table_name,',', v_table_column,')');
                        insert into procedure_log select concat('@i_sql is ', @i_sql);
                        PREPARE i_sql from @i_sql;
                        execute  i_sql ;
                        DEALLOCATE PREPARE v_sql;
                        commit ;
                end if;

            END LOOP;
            close v_table_columns;
            -- 第一个循环结束时done=1
            -- 故需手动设置done=0,否则外层循环仅会执行一次
            SET isFlag = 0;
        end while;
    end loop;
end;

```

