---
layout: default
title: ORACLE
nav_order: 20
parent: Database
---

# ORACLE
{: .no_toc }

## TABLE of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## oracle 

Oracle 在线学习：https://livesql.oracle.com

## 基础语法

### Oracle新建数据库（新用户）

1.创建用户

```sql
-- username：新用户名的用户名
-- password: 新用户的密码也可以不创建新用户，而仍然用以前的用户，如：继续利用scott用户2
create user user_name identified by "user_password"
default tablespace tbs_name
temporary tablespace temp profile DEFAULT;
```

2.创建表空间：    
```sql
create tablespace tablespacename datafile 'd:\data.dbf' size xxxm;
```
- tablespacename：表空间的名字    
- d:\data.dbf'：表空间的存储位置    
- xxx表空间的大小，m单位为兆(M)

3.将空间分配给用户:
  
```sql
alert user username default tablespace tablespacename;
```
将名字为tablespacename的表空间分配给username 

4.给用户授权：   

一次授权

```sql
grant create session,create table,unlimited tablespace to username;
```

分别授权

```sql
grant connect to user_name;
grant create indextype to user_name;
grant create job to user_name;
grant create materialized view to user_name;
grant create procedure to user_name;
grant create public synonym to user_name;
grant create sequence to user_name;
grant create session to user_name;
grant create TABLE to user_name;
grant create trigger to user_name;
grant create type to user_name;
grant create view to user_name;
grant unlimited tablespace to user_name;
ALTER user user_name quota unlimited on tbs_name;

grant debug any procedure to user_name;
grant debug connect session to user_name;
```

5.然后再以创建的用户登录，登录之后创建表即可。

```sql
conn username/password;
```


### rename 重命名：

（1）表的重命名

```sql
ALTER TABLE old_table_name rename to new_table_name
```

说明：ALTER TABLE 表名 rename to 新表名

示例：

```sql
ALTER TABLE sf_user rename to t_user;
```


（2）索引重命名

```sql
-- 索引重命名约束
alter index idx_custseq  rename to idx_custseq_new;
```

（2）约束重命名

```sql
-- 重命名主键约束名
alter table A_ALAN_TEST rename constraint pk_un_idx_custseq to pk_txun_idx_custseq;
```


### 字段补偿

（1）**增加字段语法**：

```sql
ALTER TABLE table_name add (column datatype [default value][null/not null],….);
```

说明：ALTER TABLE 表名 add (字段名 字段类型 默认值 是否为空);

示例：

```sql
ALTER TABLE t_users add (user_image blob);

ALTER TABLE t_users add (userName varchar2(30) default '空' not null);
```



（2）**修改字段的语法：**

- modify 改列类型或长度、非空。
- rename 改名字。

```sql
ALTER TABLE table_name modify (column datatype [default value][null/not null],….); 
```

说明：ALTER TABLE 表名 modify (字段名 字段类型 默认值 是否为空);

示例：

```sql
ALTER TABLE t_users MODIFY (BILLCODE number(4));
```


```sql
ALTER TABLE table_name modify (column datatype [default value][null/not null],….); 
```

说明：ALTER TABLE 表名 MODIFY (字段名 字段类型 默认值 是否为空);


rename 语法格式
```
ALTER TABLE t_users RENAME COLUMN old_column TO new_column ;
```

说明：ALTER TABLE 表名 RENAME COLUMN 旧字段 TO 新字段 ;


示例：

```sql
ALTER TABLE table_name RENAME column product_id  TO product_num; 
```



高级玩法：


```sql
-- 搜索用户下的某个字段，都存在于哪张表（t.OWNER指定用户，大写；t.COLUMN_NAME要搜索的字段）
SELECT distinct t1.TABLE_NAME,t.COLUMN_NAME
FROM DBA_TAB_COLS t,
     DBA_TABLES t1
WHERE t.TABLE_NAME = t1.TABLE_NAME
  and t.OWNER = 'PAYMT' -- 用户
  and t.COLUMN_NAME = 'VEHKIND' --列表
order by t1.TABLE_NAME,t.COLUMN_NAME;

-- 拼接出字段修改类型修改sql（t.OWNER指定用户，大写；t.COLUMN_NAME要搜索的字段；字段类型及长度需要修改为自己想要的）
SELECT 'ALTER TABLE ' || t.TABLE_NAME || ' modify ' || t.COLUMN_NAME || ' varchar2(30);'
FROM DBA_TAB_COLS t,
     DBA_TABLES t1
WHERE t.TABLE_NAME = t1.TABLE_NAME
  and t.OWNER = 'PAYMT' -- 用户
  and t.COLUMN_NAME = 'VEHKIND' --列表
order by t1.TABLE_NAME,t.COLUMN_NAME;
```
 

（3）**删除字段的语法：**

```sql
ALTER TABLE table_name DROP (COLUMN column_name);
```

说明：ALTER TABLE 表名 DROP COLUMN 字段名;

示例：

```sql
ALTER TABLE t_users DROP COLUMN user_image;
```

 

（4）**字段的重命名：**

```sql
-- 其中：column是关键字
ALTER TABLE table_name  rename  column  old_cloumn_name to  new_cloumn_name  
```

说明：ALTER TABLE 表名 rename  column 列名 to 新列名  （其中：column是关键字）

示例：

```sql
ALTER TABLE t_users rename column  nlcke_name  to  nick_name;
```

（4）**索引的补偿：**

无主键表查询

```sql
--没有主键的表
select table_name
  from user_tables a
 where not exists (select *
          from user_constraints b
         where b.constraint_type = 'P'
           and a.table_name = b.table_name)
           
           
--SYS开头的       
select table_name
  from user_tables a
 where not exists (select *
          from user_constraints b
         where b.constraint_type = 'P'
           and a.table_name = b.table_name)
and table_name  LIKE 'SYS%' 
```

主键属性： = 唯一索引 + 非空约束 （NOT NUL）

主键属性：普通索引 + 唯一约束 + NOT NULL约束

唯一索引属性：普通索引 + 唯一约束

**最佳实践：**

- （1）主键用唯一索引+主键约束两步骤来创建，可直接变更为唯一索引

- （2）唯一索引用普通索引+唯一约束两步骤来创建，可以直接变更为普通索引


(A) 针对没有索引、没有主键的非空字段

可以直接添加设置主键。

```sql
-- 需保证 bussid 字段非空
alter table TEST01 add constraint PK_TEST01 primary key (bussId);
```

(B) 针对已经是唯一索引的非空字段

可以删除重建主键。

```sql
-- 已是唯一
create unique  index un_idx_custseq on A_ALAN_TEST(CUSTSEQ) tablespace ams_index ;

-- 删除重建主键
drop index idx_custseq ;
alter table TEST01 add constraint PK_AT_CUSTSEQ primary key (CUSTSEQ);
```

```sql
drop index idx_custseq ;
create unique index pk_un_idx_custseq on A_ALAN_TEST(CUSTSEQ) tablespace AMS_INDEX  parallel 32;
alter index  pk_un_idx_custseq noparallel;
-- 直接添加主键约束
alter table  A_ALAN_TEST add constraint pk_un_idx_custseq primary key (CUSTSEQ) using index pk_un_idx_custseq;
```

(B) 针对已是普通一索引的非空字段

只能选择删除索引重建主键。

```sql
drop index idx_custseq ;
alter table TEST01 add constraint PK_TEST01 primary key (bussId);
```
 
针对数据量较大，如上亿条记录。可进行并发添加索引


```sql
-- 【存在索引时删除】重建主键
drop index idx_custseq ;
create unique index pk_un_idx_custseq on A_ALAN_TEST(CUSTSEQ) tablespace AMS_INDEX  parallel 32;
alter index  pk_un_idx_custseq noparallel;

alter table  A_ALAN_TEST add constraint pk_un_idx_custseq primary key (CUSTSEQ) using index pk_un_idx_custseq;

```

### 删除数据

（1）删除数据，没有事务，无法回滚。

```sql
truncate TABLE table_name ;
```

（2）删除数据，有事务，可以回滚。

```sql
delete FROM table_name ;
```

（3）删除表和数据。

```sql
drop TABLE table_name ;
```

（3）删除重复数据。

```sql
--查询id重复数据
select * from TB_TEST TD where ID in 
(select id from TB_TEST t group by t.id having count(t.id) >1 ) order by accountbatchid;

-- 删除重复数据，删除前务必确认所有字段一致，非所有字段一致，慎用
delete from TB_TEST where ID in (
select ID from TB_TEST group by ID having count(ID) >1) 
and rowid not in (select min(rowid) from TB_TEST group by ID having count(*)>1)
```

一、重复记录根据单个字段来判断

 
1、首先，查找表中多余的重复记录，重复记录是根据单个字段（USER_CODE）来判断

```sql
select * from AMS_REPEATE_TEST where USER_CODE in(
select USER_CODE from AMS_REPEATE_TEST group by USER_CODE having count(USER_CODE) >1)
```
 

2、删除表中多余的重复记录，重复记录是根据单个字段（USER_CODE）来判断，只留有rowid最小的记录

```sql
delete from AMS_REPEATE_TEST where (USER_CODE) in (
select USER_CODE from AMS_REPEATE_TEST group by USER_CODE having count(USER_CODE) >1) 
and rowid not in (select min(rowid) from AMS_REPEATE_TEST group by USER_CODE having count(*)>1)
```

二、重复记录根据多个字段来判断


1、查找表中多余的重复记录（多个字段）

 
```sql
select * from AMS_REPEATE_TEST a where (a.USER_CODE,a.USER_NAME) in(
select USER_CODE,USER_NAME from AMS_REPEATE_TEST group by USER_CODE,USER_NAME having count(*) > 1)
```

2、删除表中多余的重复记录（多个字段），只留有rowid最小的记录

```sql
delete from AMS_REPEATE_TEST a where (a.USER_CODE,a.USER_NAME) in (
select USER_CODE,USER_NAME from AMS_REPEATE_TEST group by USER_CODE,USER_NAME having count(*) > 1) 
and rowid not in (
select min(rowid) from AMS_REPEATE_TEST group by USER_CODE,USER_NAME having count(*)>1)
```

 
3、查找表中多余的重复记录（多个字段），不包含rowid最小的记录

```sql
select * from AMS_REPEATE_TEST a where (a.USER_CODE,a.USER_NAME) in (
select USER_CODE,USER_NAME from AMS_REPEATE_TEST group by USER_CODE,USER_NAME having count(*) > 1) 
and rowid not in (select min(rowid) from AMS_REPEATE_TEST group by USER_CODE,USER_NAME having count(*)>1)
```


### 自增分区：

(1)在DDL中添加

将时间按日或按月分区

```sql
CREATE TABLE TEST_TB
(
ID NUMBER(20) NOT NULL,
REMARK VARCHAR2(100),
SUBCOMPANY VARCHAR2(100),
CREATETIME DATE
)
PARTITION BY RANGE(CREATETIME) INTERVAL(NUMTODSINTERVAL(1,'DAY'))
SUBPARTITION BY LIST(SUBCOMPANY)
(
    PARTITION PART_101 VALUES LESS THAN(TO_DATE('2022-10-01','yyyy-mm-dd'))
    (
        SUBPARTITION P1_SUB1 VALUES ('1010100') TABLESPACE AMS_DATA,
        SUBPARTITION P1_SUB2 VALUES ('1020100') TABLESPACE AMS_DATA,
        SUBPARTITION P1_SUBN VALUES DEFAULT TABLESPACE AMS_DATA
    )
);
```

(2)补偿添加

```sql
-- 自增分区间隔1天
ALTER TABLE TEST_TB SET INTERVAL(NUMTODSINTERVAL(1,'DAY'))

-- 自增分区间隔1月
ALTER TABLE TEST_TB SET INTERVAL(NUMTOYMINTERVAL(1,'MONTH'))
```

### 检测中文乱码

可用函数

- `ASCIISTR` 函数可以使中文变成 '\xxxx' ;
- `UNISTR` 函数可以反转回中文。

```sql
SELECT asciistr('安诚') FROM  dual ;

SELECT unistr('\5B89\8BDA') FROM  dual ;
```

检测指定的字段是否含有中文乱码:

```sql
SELECT column_name FROM table_name WHERE asciistr('column_name') like '%??%' or asciistr('column_name') like '%\FFFD%' ;
```

检测指定表或指定库是否包含中文乱码，并使用DBMS输出到控制台：

```sql
create or replace procedure check_zh_cn is
    v_table_name  varchar2(50) ;
    v_table_column  varchar2(50) ;
    v_sql  varchar2(1000) ;
    v_count  int(10) := 0;
    v_index  int(10) := 0;
    -- 查询所有的表
    cursor v_table_names is
        SELECT * FROM USER_TABLES U;
begin
    for v_table_data in v_table_names loop
      begin
        v_table_name := v_table_data.TABLE_NAME ;
        -- 否则查询出表的列数据
        for v_table_column_data in (SELECT * FROM user_tab_columns WHERE Table_Name=v_table_name ORDER BY COLUMN_ID ASC ) loop

            begin
            v_table_column := v_table_column_data.COLUMN_NAME ;
            if  instr(v_table_column_data.DATA_TYPE, 'VARCHAR') > 0 then
                -- 判断乱码
                --v_sql := 'SELECT count(1) FROM  :v_table_name WHERE asciistr(:v_table_column) like ''%??%'' or asciistr(:v_table_column) like ''%\FFFD%'' ';
                --execute immediate v_sql into v_count  using v_table_name, v_table_column,v_table_column;
                v_sql := 'SELECT count(1) FROM '|| v_table_name || ' WHERE asciistr('|| v_table_column ||') like ''%??%'' or asciistr('|| v_table_column ||') like ''%\FFFD%'' ';
                execute immediate v_sql into v_count;
                if v_count >0 then
                   -- L_RESULT[v_index] = v_table_name || '.' || v_table_column ;
                   v_index := v_index+1;
                   DBMS_OUTPUT.PUT_LINE(v_index ||' table.column is ' || v_table_name || '.' || v_table_column);
                end if;
            end if;
            end;
        end loop;
      end;
    end loop ;
end;
```

也可以将有乱码的表和字段输出到一张表里：

```sql
create  or replace procedure check_zh_cn_to_table is
    v_table_name  varchar2(50) ;
    v_table_column  varchar2(50) ;
    v_sql  varchar2(1000) ;
    v_count  int(10) := 0;
    v_index  int(10) := 0;
    r_table_name varchar2(20) := 'CHECK_ZHCN_RESULT';
    d_sql varchar2(500) := 'drop TABLE '||r_table_name ;
    c_sql varchar2(500) := 'create TABLE '||r_table_name||' (table_name varchar2(100), column_name  varchar2(100))';
    i_sql varchar2(500) := 'insert into '||r_table_name||' (table_name, column_name) values (:c1, :c2)';
    -- 查询所有的表
    cursor v_table_names is
        SELECT * FROM USER_TABLES U;

begin
    -- 结果表
    SELECT count(1) into v_count FROM USER_TABLES WHERE TABLE_NAME = r_table_name;
    IF v_count > 0 then
        execute immediate d_sql ;
        execute immediate c_sql ;
    else
        -- DBMS_OUTPUT.PUT_LINE(c_sql);
        execute immediate c_sql ;
    end if;
    for v_table_data in v_table_names loop
      begin
        v_table_name := v_table_data.TABLE_NAME ;
        -- 否则查询出表的列数据
        for v_table_column_data in (SELECT * FROM user_tab_columns WHERE Table_Name=v_table_name ORDER BY COLUMN_ID ASC ) loop

            begin
            v_table_column := v_table_column_data.COLUMN_NAME ;
            if  instr(v_table_column_data.DATA_TYPE, 'VARCHAR') > 0 then
                -- 判断乱码
                 v_sql := 'SELECT count(1) FROM '|| v_table_name || ' WHERE asciistr('|| v_table_column ||') like ''%??%'' or asciistr('|| v_table_column ||') like ''%\FFFD%'' ';
                execute immediate v_sql into v_count;
                if v_count >0 then
                    v_index := v_index+1;
                    DBMS_OUTPUT.PUT_LINE(v_index ||' table.column is ' || v_table_name || '.' || v_table_column);
                    execute immediate i_sql using v_table_name, v_table_column;
                    commit ;
                end if;
            end if;
            end;
        end loop;
      end;
    end loop ;
end;

```

## 常用查询

### 查版本

```sql
select * from V$VERSION  ;
```
查询结果：

| BANNER | CON_ID |
| ------ | ------ |
|Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production | 0 |
|PL/SQL Release 12.2.0.1.0 - Production | 0 |
|CORE	12.2.0.1.0	Production | 0 |
|TNS for 64-bit Windows: Version 12.2.0.1.0 - Production| 0 |
|NLSRTL Version 12.2.0.1.0 - Production | 0 |


### 查询数据库全部序列

 
--查看当前用户的所有序列 PAYMT 为数据库名称

```sql
SELECT SEQUENCE_OWNER,SEQUENCE_NAME FROM dba_sequences WHERE sequence_owner='PAYMT';
```


### 查询表-字段-注释

查表DDL最后更新时间

```sql
SELECT uat.table_name as tableName,
    (SELECT last_ddl_time FROM user_objects WHERE  OBJECT_TYPE='TABLE' AND object_name = uat.table_name ) as lasDdlTime
FROM user_all_tables uat ;
```

查用户所有的表:

```sql
-- 查指定用户的所有表(dba权限)
SELECT TABLE_NAME FROM DBA_TABLES WHERE OWNER='用户名' ;
```

```sql
-- 查用户所有的表(dba权限)
SELECT TABLE_NAME FROM DBA_TABLES WHERE OWNER='BPJYDATA';

-- 当前用户拥有的表
SELECT U.TABLE_NAME,U.* FROM USER_TABLES U;
-- 所有用户的表
SELECT TABLE_NAME FROM ALL_TABLES;
-- 包括系统表
SELECT TABLE_NAME FROM DBA_TABLES;

```

```
-- 描述当前用户有访问权限的所有对象
-- ALL_OBJECTS describes all objects accessible to the current user.
-- 描述了数据库中的所有对象
-- DBA_OBJECTS     describes all objects in the database.
-- 描述了当前用户所拥有的所有对象
-- USER_OBJECTS    describes all objects owned by the current user.
```


查表对应的注释:

```sql
-- 查表对应的注释
SELECT distinct
        t1.TABLE_NAME,t.COMMENTS
FROM DBA_TAB_COMMENTS t,
     DBA_TABLES t1
WHERE t.TABLE_NAME = t1.TABLE_NAME
  AND t.OWNER = 'BPJYDATA' AND T1.OWNER='BPJYDATA'
  --AND t.TABLE_NAME = 'T_APPLICATIONS'
;
```

查表对应的主键:

```sql
-- 查表对应的主键
SELECT
       a.TABLE_NAME AS 表名,
       a.constraint_name AS 主键名称,
       a.column_name AS 列名,
       b.constraint_type
FROM user_cons_columns a, user_constraints b
WHERE a.constraint_name = b.constraint_name
AND b.constraint_type = 'P'
AND a.OWNER ='BPJYDATA' AND a.TABLE_NAME IN (
    SELECT TABLE_NAME FROM DBA_TABLES WHERE OWNER='BPJYDATA'
    ) ;
```

查表对应的列:

```sql
-- 查（当前用户）表对应的列
SELECT * FROM user_tab_columns WHERE Table_Name='T_APPLICATIONS' ORDER BY COLUMN_ID ASC;
-- 查（全部用户）表名全部的列
SELECT * FROM all_tab_columns WHERE Table_Name='T_APPLICATIONS';
-- 查（全部用户，含系统表）表名全部的列
SELECT * FROM dba_tab_columns WHERE Table_Name='T_APPLICATIONS';
```


不含有管理员的表

```sql
-- 查表对应的列
SELECT UTC.COLUMN_NAME as columnName, --列名
       UTC.DATA_TYPE,
       CASE UTC.DATA_TYPE WHEN 'DATE' THEN  UTC.DATA_TYPE
            ELSE    UTC.DATA_TYPE || '(' || UTC.DATA_LENGTH || ')'
       END columnType,  -- 类型
       UCC.COMMENTS as columnComment,  -- 表字段的注释
       UTC.DATA_DEFAULT as dataDefault,  -- 默认值
        UTC.NULLABLE as nullable
 FROM USER_TAB_COLUMNS  UTC ,
     USER_COL_COMMENTS UCC
WHERE UTC.TABLE_NAME =UCC.TABLE_NAME
  AND UTC.COLUMN_NAME = UCC.COLUMN_NAME
  AND UTC.TABLE_NAME= 'RISK_SURVEY_PARAM'
  ORDER BY UTC.COLUMN_ID ASC
```

含有管理员的表

```sql
-- 查表对应的列
SELECT UTC.COLUMN_NAME as columnName, --列名
       CASE UTC.DATA_TYPE WHEN 'DATE' THEN  UTC.DATA_TYPE
            ELSE    UTC.DATA_TYPE || '(' || UTC.DATA_LENGTH || ')'
       END columnType,  -- 类型
       UCC.COMMENTS as columnComment,  -- 表字段的注释
       UTC.DATA_DEFAULT as dataDefault,  -- 默认值
        UTC.NULLABLE as nullable ,  -- 是否为空
       UTC.DEFAULT_ON_NULL   
 FROM USER_TAB_COLUMNS  UTC ,
     DBA_TABLES T,
     USER_COL_COMMENTS UCC
WHERE UTC.TABLE_NAME = T.TABLE_NAME AND T.TABLE_NAME=UCC.TABLE_NAME
  AND UTC.COLUMN_NAME = UCC.COLUMN_NAME
  AND T.OWNER= 'PAYMT'    
  AND UTC.TABLE_NAME= 'T_APPLICATIONS'  
  ORDER BY UTC.COLUMN_ID ASC

```


查表对应的注释:

```sql
SELECT* FROM all_indexes WHERE table_name='T_APPLICATIONS';

-- 查表对应的注释
SELECT UIC.INDEX_NAME, TC.COLUMN_NAME, UIC.*, TC.* 
FROM USER_IND_COLUMNS UIC,
DBA_TAB_COLS TC
WHERE TC.TABLE_NAME = UIC.TABLE_NAME
  AND TC.COLUMN_NAME = UIC.COLUMN_NAME
  AND TC.OWNER = 'PAYMT'
  AND TC.TABLE_NAME='T_APPLICATIONS'
ORDER BY UIC.INDEX_NAME, UIC.COLUMN_POSITION ;
```

### 查询表约束

查找表约束：`SELECT constraint_name FROM user_cons_columns a WHERE a.table_name='tablename';`

增加表约束：`ALTER TABLE tablename ADD CONSTRAINT pk_name primary key(column);`

删除表约束：`ALTER TABLE tablename DROP CONSTRAINT constraint_name `----（SYS_C002715）;

修改表约束：1）禁用表主键：`ALTER TABLE tablename DISABLE primary key;`

　　　　　　2）启用表主键：`ALTER TABLE tablename ENABLE primary key;`

　　　　　　3）重命名表主键：`ALTER TABLE tablename RENAME CONSTRAINT pk_id TO new_pk_id;`

```sql
--查外键约束
SELECT * FROM user_cons_columns cl WHERE cl.constraint_name = 外键名称

--查当前schema的约束
SELECT table_name,constraint_name,constraint_type FROM user_constraints
WHERE table_name='大写的表名'

--查所有的约束
SELECT table_name,constraint_name,constraint_type FROM dba_constraints
WHERE table_name='大写的表名'


-- 查约束
SELECT constraint_name, table_name, r_owner, r_constraint_name
FROM all_constraints
WHERE table_name = 'table_name' and owner = 'owner_name';

--另一个视图ALL_CONS_COLUMNS也包括组成表上约束列的信息。
```

生成启动、禁用约束的SQL

```sql
--查询出所有表的唯一键约束的,生成禁用sql
SELECT 'ALTER TABLE ' || table_name || ' disable constraint '||constraint_name||';'
FROM user_constraints WHERE constraint_type='U';
```

如下：

```sql
ALTER TABLE MM_PERMISSIONS_TD DISABLE CONSTRAINT MM_PERMISSIONS_TD_UNIQUE ;
ALTER TABLE MM_PLAN_TD DISABLE CONSTRAINT U_PLAN_TD;
ALTER TABLE MM_PREPAYINFO_TD DISABLE CONSTRAINT U_PREPAYINFO_TD_01;
ALTER TABLE MM_PREPAYMENTIN_TD DISABLE CONSTRAINT U_PREPAYMENTID_TD_01;
```


```sql
--查询出所有表的唯一键约束的,生成启用sql
SELECT 'ALTER TABLE ' || table_name || ' ENABLE CONSTRAINT '||constraint_name||';'
FROM user_constraints WHERE constraint_type='U';
```

如下：
```sql
ALTER TABLE SYENTINFO2 enable constraint UQ_SY20180806;
ALTER TABLE WEB_INFO enable constraint UQWEBINFO201807211136;
ALTER TABLE ENT_INFO_WUHAN enable constraint SYS_C0024733;
```

```sql
--查询所有表的所有外键 
SELECT 'ALTER TABLE ' || table_name || ' enable constraint '||constraint_name||';'
FROM user_constraints WHERE constraint_type='R';
```

### 查询表索引
 

```sql

-- 查用户所有的所有
SELECT * FROM USER_INDEXES WHERE table_name = '大写的表名'

-- 查所有的所有
SELECT* FROM all_indexes WHERE table_name='T_APPLICATIONS';


-- 查表的索引信息
SELECT UIC.INDEX_NAME as indexName,  -- 索引名称
       TC.COLUMN_NAME AS columnName, -- 列名称
       CASE TC.DATA_TYPE WHEN 'DATE' THEN  TC.DATA_TYPE
            ELSE    TC.DATA_TYPE || '(' || TC.DATA_LENGTH || ')'
        END columnType ,  -- 列类型
       TC.*
FROM USER_IND_COLUMNS UIC,
DBA_TAB_COLS TC
WHERE TC.TABLE_NAME = UIC.TABLE_NAME
  AND TC.COLUMN_NAME = UIC.COLUMN_NAME
  AND TC.OWNER = 'PAYMT'
  AND TC.TABLE_NAME='T_CENTRALPAYMENTINFO_TD'
ORDER BY UIC.INDEX_NAME, UIC.COLUMN_POSITION ;
```

### 查视图

```sql
SELECT * FROM all_views WHERE OWNER='PAYMT'
ORDER BY VIEW_NAME
```


### 查看空表-函数：

```sql
SELECT sysdate FROM dual;
-- 等价于mysql
SELECT CURRENT_TIMESTAMP ;
```

### TO_CHAR()函数

 （1）用作日期转换：

to_char(date,'格式');

```sql
SELECT TO_DATE('2021-09-13', 'YYYY-MM-DD') FROM DUAL ;
SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD') FROM DUAL ;

SELECT to_char(now(),'Day, HH12:MI:SS')  FROM DUAL ;   -- 'Tuesday , 05:39:18'
SELECT to_char(now(),'FMDay, HH12:MI:SS')  FROM DUAL ;   -- 'Tuesday, 05:39:18'
```

（2）处理数字：

to_char(number,'格式');

```sql
SELECT to_char(-0.1,'99.99')  FROM DUAL ;   --  ' -.10'
SELECT to_char(-0.1,'FM9.99')  FROM DUAL ;   --  '-.1'
SELECT to_char(0.1,'0.9')  FROM DUAL ;   --  ' 0.1'
SELECT to_char(12,'9990999.9')  FROM DUAL ;   --  ' 0012.0'
SELECT to_char(12,'FM9990999.9')  FROM DUAL ;   --  '0012'
SELECT to_char(485,'999')  FROM DUAL ;   --  ' 485'
SELECT to_char(-485,'999')  FROM DUAL ;   --  '-485'
SELECT to_char(485,'9 9 9')  FROM DUAL ;   --  ' 4 8 5'
SELECT to_char(1485,'9,999')  FROM DUAL ;   --  ' 1,485'
SELECT to_char(1485,'9G999')  FROM DUAL ;   --  ' 1 485'
SELECT to_char(148.5,'999.999')  FROM DUAL ;   --  ' 148.500'
SELECT to_char(148.5,'999D999')  FROM DUAL ;   --  ' 148,500'
SELECT to_char(3148.5,'9G999D999')  FROM DUAL ;   --  ' 3 148,500'
SELECT to_char(-485,'999S')  FROM DUAL ;   --  '485-'
SELECT to_char(-485,'999MI')  FROM DUAL ;   --  '485-'
SELECT to_char(485,'999MI')  FROM DUAL ;   --  '485'
SELECT to_char(485,'PL999')  FROM DUAL ;   --  '+485'
SELECT to_char(485,'SG999')  FROM DUAL ;   --  '+485'
SELECT to_char(-485,'SG999')  FROM DUAL ;   --  '-485'
SELECT to_char(-485,'9SG99')  FROM DUAL ;   --  '4-85'
SELECT to_char(-485,'999PR')  FROM DUAL ;   --  '<485>'
SELECT to_char(485,'L999')  FROM DUAL ;   --  'DM 485
SELECT to_char(485,'RN')  FROM DUAL ;   --  ' CDLXXXV'
SELECT to_char(485,'FMRN')  FROM DUAL ;   --  'CDLXXXV'
SELECT to_char(5.2,'FMRN')  FROM DUAL ;   --  V
SELECT to_char(482,'999th')  FROM DUAL ;   --  ' 482nd'
SELECT to_char(485, '"Good number:"999')  FROM DUAL ;   --  'Good number: 485'
SELECT to_char(485.8,'"Pre-decimal:"999" Post-decimal:" .999')  FROM DUAL ;   --  'Pre-decimal: 485 Post-decimal: .800'
SELECT to_char(12,'99V999')  FROM DUAL ;   --  ' 12000'
SELECT to_char(12.4,'99V999')  FROM DUAL ;   --  ' 12400'
SELECT to_char(12.45, '99V9')  FROM DUAL ;  --  ' 125'
SELECT TO_CHAR(4567, '$99,99') FROM DUAL ;  --   $45,67
```


（4）用于进制转换：将10进制转换为16进制；

例子：
```sql
SELECT TO_CHAR(123,'XXX') FROM DUAL ;

SELECT TO_CHAR(4567, 'XXXX') FROM DUAL ;
```

### decode 函数

decode(条件,值1,返回值1,值2,返回值2,...值n,返回值n,缺省值)

该函数的含义如下：

```
IF 条件=值1 THEN
　　　　RETURN(返回值1)
ELSIF 条件=值2 THEN
　　　　RETURN(返回值2)
　　　　......
ELSIF 条件=值n THEN
　　　　RETURN(返回值n)
ELSE
　　　　RETURN(缺省值)
END IF
```

### concat 函数

concat(字符串1, 字符串2)

该函数的含义如下：

将字符串1和字符串2拼接到一起。


## PL/SQL 相关

### PLSQL Developer解决中文乱码问题

1.查服务端字符集编码

```sql
SELECT userenv('language') FROM dual;
```

结果

```
    userenv('language')
1	SIMPLIFIED CHINESE_CHINA.AL32UTF8
```

2.执行语句 `SELECT * FROM V$NLS_PARAMETERS` 查看第一行中PARAMETER项中为NLS_LANGUAGE 对应的VALUE项中是否和第一步得到的值一样。

```sql
SELECT * FROM V$NLS_PARAMETERS 
```

结果

```
    PARAMETER              VALUE               CON_IN  
1	NLS_LANGUAGE 	       SIMPLIFIED CHINESE	0
2	NLS_TERRITORY	       CHINA	            0
3	NLS_CURRENCY	       ￥	               0
4	NLS_ISO_CURRENCY	       CHINA	        0
5	NLS_NUMERIC_CHARACTERS	   .,	            0
6	NLS_CALENDAR	       GREGORIAN	        0
7	NLS_DATE_FORMAT	       DD-MON-RR	        0
8	NLS_DATE_LANGUAGE	   SIMPLIFIED CHINESE	0
9	NLS_CHARACTERSET	   AL32UTF8         	0
10	NLS_SORT	           BINARY	            0
11	NLS_TIME_FORMAT	       HH.MI.SSXFF AM	    0
12	NLS_TIMESTAMP_FORMAT	DD-MON-RR HH.MI.SSXFF AM	    0
13	NLS_TIME_TZ_FORMAT	    HH.MI.SSXFF AM TZR	            0
14	NLS_TIMESTAMP_TZ_FORMAT	DD-MON-RR HH.MI.SSXFF AM TZR	0
15	NLS_DUAL_CURRENCY	    ￥	               0
16	NLS_NCHAR_CHARACTERSET	AL16UTF16	        0
17	NLS_COMP	            BINARY	            0
18	NLS_LENGTH_SEMANTICS	BYTE	            0
19	NLS_NCHAR_CONV_EXCP	    FALSE	            0
```



如果不是，需要设置环境变量.PLSQL客户端使用的编码和服务器端编码不一致,插入中文时就会出现乱码.

**NLS_LANGUAGE_NLS_TERRITORY.NLS_CHARACTERSET**

3.设置环境变量计算机->属性->高级系统设置->环境变量->新建

设置变量名：`NLS_LANG`,
变量值：`SIMPLIFIED CHINESE_CHINA.AL32UTF8`

4.重启PL/SQL





### 存储过程

```sql
-- 尚未测试
create or replace PROCEDURE ZZY(demo in ats_back_ti % rowtype) is
   v_back_ti  ats_back_ti % rowtype ;
begin
   v_back_ti := demo;
   v_back_ti.d
   dbms_output.put_line(v_back_ti.id);
   dbms_output.put_line(v_back_ti.TRANSCODE);
end 
```


```sql
-- 尚未测试
create or replace procedure test_dblink(out_cursor out int)  Authid Current_User  as

begin
      --  test db link 数据库 连接
     execute immediate 'create database link dblink1 
     connect to  用户名 identified by "密码"
     using ''192.168.1.100:1521/orcl'' ';

   --    open out_cursor for 'SELECT * FROM A@dblink1';
  

  -- 返回个数值
    out_cursor :=0;
    execute immediate 'SELECT count(*) FROM A@dblink1'  into out_cursor ;
    
    -- 取消连接
    execute immediate 'drop database link dblink1';
    
    return ;
    
end  ;
```


## 元数据查询

ORACLE 数据字典视图的种类分别为：USER,ALL 和 DBA.
- USER_* ：有关用户所拥有的对象信息，即用户自己创建的对象信息。
- ALL_* ：有关用户可以访问的对象的信息，即用户自己创建的对象的信息加上其他用户创建的对象但该用户有权访问的信息。
- DBA_* ：有关整个数据库中对象的信息，需要DBA权限。

> *可以为TABLES,INDEXES,OBJECTS,USERS等。

### 1、查用户

查看当前登录用户名、创建时间、默认表空间、临时表空间：
```sql
SELECT * FROM user_users;
```
查询全部的用户信息（信息有限，不能查看默认表空间等）：

```sql
SELECT * FROM all_users;
```

查看全部的用户(信息全，需要dba角色)：
```sql
SELECT * FROM dba_users;
```

查看用户是否以dba身份登录：
```sql
SELECT SYS_CONTEXT('BVIS','ISDBA') FROM dual;
```
显示 TRUE/FALSE


### 2、查角色

查看所有的角色（需要已授权dba角色）：
```sql
SELECT * FROM dba_roles;
```
查看当前用户拥有的角色：
```sql
SELECT * FROM user_role_privs;
```
查看哪些用户有sysdba或sysoper系统权限(需要dba角色)：
```sql
SELECT * FROM V$PWFILE_USERS
```
查看角色的授予情况：
```sql
SELECT * FROM dba_role_privs;
```
### 3、查权限

查看所有的系统权限：
```sql
SELECT * FROM system_privilege_map;
```
查看直接授予给当前用户的系统权限：
```sql
SELECT * FROM user_sys_privs;
```
查看当前用户可以使用的权限信息（将解析拥有的角色对应的权限）：
```sql
SELECT * FROM session_privs
```
查看已经授予所有用户的系统权限信息（需要dba角色）：
```sql
SELECT * FROM dba_sys_privs
```

查看当前用户的对象权限：
```sql
SELECT * FROM user_tab_privs;
```
查看所有授予的对象权限（需要dba角色）：
```sql
SELECT * FROM dba_tab_privs;
```

### 4、查登录数据库

查看当前登录的数据库名：
```sql
SELECT SYS_CONTEXT('USERENV','DB_NAME') FROM dual;
```

查看当前登录的实例名称：
```sql
SELECT SYS_CONTEXT('USERENV','INSTANCE_NAME') FROM dual;
```

### 5、查数据库表

#### 5.1 查常规表

查看当前用户的所有表：
```sql
SELECT * FROM user_tables;
```

查看当前用户的表的表结构：
```sql
SELECT *
  FROM USER_TAB_COLUMNS T, USER_COL_COMMENTS C
 WHERE T.TABLE_NAME = C.TABLE_NAME
   AND T.COLUMN_NAME = C.COLUMN_NAME
   AND T.TABLE_NAME = UPPER('T_APPLICATION')
```

查主键
```sql
-- 查看所有主键
SELECT CU.* FROM USER_CONS_COLUMNS CU, USER_CONSTRAINTS AU
WHERE CU.CONSTRAINT_NAME = AU.CONSTRAINT_NAME AND AU.CONSTRAINT_TYPE = 'P' ;

-- 分组查主键
SELECT CU.TABLE_NAME ,CU.CONSTRAINT_NAME ,LISTAGG(CU.COLUMN_NAME, ',')WITHIN GROUP(ORDER BY CU.TABLE_NAME)
FROM USER_CONS_COLUMNS CU, USER_CONSTRAINTS AU 
WHERE CU.CONSTRAINT_NAME = AU.CONSTRAINT_NAME AND AU.CONSTRAINT_TYPE = 'P'
GROUP BY  CU.TABLE_NAME,CU.CONSTRAINT_NAME
ORDER BY CU.TABLE_NAME, CU.CONSTRAINT_NAME;
```

#### 5.2 临时表查询

1、查看当前用户下的表是否为临时表：
```sql
-- 其中的 duration（持续时间）为 null 表示非临时表，SYS$SESSION 表示会话临时表，SYS$TRANSACTION 表示事务临时表
select * from user_tables where duration is not null;

-- 查询当前用户所有的临时表
SELECT * FROM USER_TABLES WHERE TEMPORARY = ‘Y’ ;
-- 查询全部用户的所有的临时表
SELECT * FROM DBA_TABLES WHERE TEMPORARY = ‘Y’;
```


2、临时表删除

（1）事务临时表提交或者回滚后，只是删除其中的数据，表结构仍然还在，会话临时表也是一样，会话结束后，数据删除了，当表结构还在。

（2）如果会话临时表的会话没有结束，则无法删除此临时表，事务临时表也是同理，也只能在未被使用时才能删除。


### 6、查视图

查看当前用户的所有视图：
```sql
SELECT * FROM user_views;
```

### 7、查存储过程

```sql
create or replace procedure DEMO_PROC
(
--定义输入、输出参数--
num_A in number ,
num_B in number,
numType in number,
num_C out number
)
as
--定义变量--
 -- numCount integer;
 -- numStr varchar(20);  
begin   
     --判断计算类型--
     if numType=1 then
        num_C := num_A + num_B;
     elsif numType=2 then
        num_C := num_A - num_B;
     elsif numType=3 then
        num_C := num_A * num_B; 
     elsif numType=4 then
        num_C := num_A / num_B; 
     else
     --其它处理
       dbms_output.put_line('其它处理');
     end if;
end;
-- 调用存储过程
declare num_C number;
begin
   --调用存储过程---
   DEMO_PROC(3,4,3,num_C);
   dbms_output.put_line('输出结果：'|| num_C );
end;
```

查看当前用户创建的所有存储过程：

```sql
SELECT * FROM user_procedures;
```
查看存储过程定义语句：

```sql
SELECT * FROM user_source WHERE NAME ='DEMO_PROC' ORDER BY line;
```

### 8、查序列


```sql
-- 创建一个序列
create sequence t_common_seq start with 1 increment by 1;
SELECT * FROM user_sequences;
```

### 9、查触发器

查看创建的触发器：
```sql
SELECT * FROM user_triggers;
```

### 10、查函数

创建函数

```sql
CREATE OR REPLACE FUNCTION FUNC_ADD (
    x int,
    y int
) RETURN int IS
    res int;
BEGIN
    return x+y;
END;
```

调用函数

```sql
DECLARE results NUMBER;
BEGIN
results := FUNC_ADD(2,3);
DBMS_OUTPUT.PUT_LINE('输出: 2+3='||results);
END;
```

查看函数：

```sql
SELECT * FROM USER_OBJECTS WHERE OBJECT_TYPE='FUNCTION';
SELECT * FROM user_source WHERE TYPE='FUNCTION'; 

```

### 11、查索引


查看当前用户的索引：

```sql
SELECT * FROM USER_INDEXES;

-- 索引与列
SELECT * FROM USER_IND_COLUMNS ;

-- 函数式索引表达式
SELECT * FROM USER_IND_EXPRESSIONS ;


-- 分组查索引
SELECT T.TABLE_NAME,T.INDEX_NAME,
LISTAGG(T.COLUMN_NAME ||' ' || T.DESCEND, ',')WITHIN GROUP(ORDER BY T.TABLE_NAME, T.COLUMN_POSITION) AS COLUMN_NAME
,I.INDEX_TYPE, I.UNIQUENESS
FROM USER_IND_COLUMNS T JOIN USER_INDEXES I ON  T.INDEX_NAME = I.INDEX_NAME
AND T.INDEX_NAME = I.INDEX_NAME AND T.TABLE_NAME = I.TABLE_NAME
--AND I.TABLE_NAME = 'AMS_ACCOUNTMATCH_TD'
GROUP BY  T.TABLE_NAME,T.INDEX_NAME ,I.INDEX_TYPE, I.UNIQUENESS  ORDER BY T.TABLE_NAME,T.INDEX_NAME ;

```

函数式索引的表达式查询

问题 函数式索引表达式查询的时候如果group by 的话  USER_IND_EXPRESSIONS 的类型不能转换，需要编写函数 处理 `USER_IND_EXPRESSIONS` 的类型。

```sql
CREATE OR REPLACE FUNCTION INDEX_COLUMN_EXPRESSION(
   IN_TABLE_NAME  VARCHAR2,
   IN_INDEX_NAME  VARCHAR2,
   IN_COLUMN_POSITION  VARCHAR2
)
RETURN VARCHAR AS
  TEXT_C1 VARCHAR2(32767);
  SQL_CUR VARCHAR2(2000);
BEGIN
 SQL_CUR := ' SELECT COLUMN_EXPRESSION '||
             ' FROM USER_IND_EXPRESSIONS TC '||
             ' WHERE 1=1 '||
             ' AND TC.TABLE_NAME='''||IN_TABLE_NAME||''' '||
             ' AND TC.INDEX_NAME='''||IN_INDEX_NAME||''' '||
             ' AND TC.COLUMN_POSITION='''||IN_COLUMN_POSITION||''' '
             ;
 --DBMS_OUTPUT.PUT_LINE (SQL_CUR);
 EXECUTE IMMEDIATE SQL_CUR INTO TEXT_C1;
 TEXT_C1 := SUBSTR(TEXT_C1, 1, 4000);
 RETURN TEXT_C1;
END;
```
使用函数
```sql
SELECT T.TABLE_NAME, T.INDEX_NAME,
       LISTAGG(COLUMN_EXPRESSION ||' ASC',',')WITHIN GROUP(ORDER BY T.TABLE_NAME, T.INDEX_NAME) AS COLUMN_NAME
FROM (SELECT E.TABLE_NAME,E.INDEX_NAME,INDEX_COLUMN_EXPRESSION(E.TABLE_NAME, E.INDEX_NAME,E.COLUMN_POSITION) AS COLUMN_EXPRESSION
    FROM  USER_IND_EXPRESSIONS E) T
GROUP BY T.TABLE_NAME, T.INDEX_NAME;
```

改函数不同通用，只针对特定的表才可以使用。也可以尝试做成通用函数

```sql

CREATE OR REPLACE FUNCTION LONG_TO_CHAR(
IN_COLUMN VARCHAR2,
IN_OWNER VARCHAR,
IN_TABLE_NAME VARCHAR,
IN_CONDITION VARCHAR2
)
RETURN VARCHAR AS
TEXT_C1 VARCHAR2(32767);
SQL_CUR VARCHAR2(2000);

BEGIN
  SQL_CUR := 'SELECT '||IN_COLUMN||' FROM '||IN_OWNER||'.'||IN_TABLE_NAME||' WHERE 1=1 AND ' || IN_CONDITION;
  DBMS_OUTPUT.PUT_LINE (SQL_CUR);
  EXECUTE IMMEDIATE SQL_CUR INTO TEXT_C1;

  TEXT_C1 := SUBSTR(TEXT_C1, 1, 4000);
  RETURN TEXT_C1;
END;
```

如我们需要处理表 SYS.USER_IND_EXPRESSIONS 的 COLUMN_EXPRESSION 字段类型，可如下使用：
表 AMS_ACCOUNTIMPDATA_DETAIL_TI 有函数式索引：

```sql
CREATE INDEX "AMSDB01"."IDX_ACCOUNTIMPDATA_DETAIL_TI04" ON "AMSDB01"."AMS_ACCOUNTIMPDATA_DETAIL_TI" (TRUNC("TRADEDATE") ASC, "PAYMENTNO" ASC, "ACCOUNTNO" ASC, "CREDITAMOUNT" ASC);
```

使用时依次传递参数即可，最后一个 IN_CONDITION 参数可以动态化传查询对应表的补充条件，这样可以更加通用。
```sql
-- 使用 LONG_TO_CHAR 函數
SELECT LONG_TO_CHAR('COLUMN_EXPRESSION', 'SYS','USER_IND_EXPRESSIONS',
    ' TABLE_NAME = ''AMS_ACCOUNTIMPDATA_DETAIL_TI'' AND INDEX_NAME = ''IDX_ACCOUNTIMPDATA_DETAIL_TI04'' '
) AS COLUMN_EXPRESSION FROM DUAL  ;
```
### 12、查集合

```sql
DELIMITER $$
CREATE OR REPLACE TYPE ty_str_split IS TABLE OF VARCHAR2 (4000);
$$
```

查看当前用户的集合：

```sql
SELECT * FROM USER_OBJECTS WHERE OBJECT_TYPE='TYPE';
SELECT * FROM user_source WHERE TYPE='TYPE'; 
```
使用，定义一个函数 fn_split

```sql
CREATE OR REPLACE FUNCTION fn_split (p_str IN VARCHAR2, p_delimiter IN VARCHAR2)
    RETURN ty_str_split PIPELINED
IS
    j INT := 0;
    i INT := 1;
    len INT := 0;
    len1 INT := 0;
    str VARCHAR2 (4000);
BEGIN
    len := LENGTH (p_str);
    len1 := LENGTH (p_delimiter);

    WHILE j < len
    LOOP
        j := INSTR (p_str, p_delimiter, i);

        IF j = 0
        THEN
            j := len;
            str := SUBSTR (p_str, i);
            PIPE ROW (str);

            IF i >= len
            THEN
                EXIT;
            END IF;
        ELSE
            str := SUBSTR (p_str, i, j - i);
            i := j + len1;
            PIPE ROW (str);
        END IF;
    END LOOP;

    RETURN;
END fn_split;
```

不使用表函数实现：
```sql
CREATE OR REPLACE FUNCTION FN_SPLIT_STR (P_STR IN VARCHAR2, P_DELIMITER IN VARCHAR2)
    RETURN TY_STR_SPLIT
IS
    L_RESULT TY_STR_SPLIT := TY_STR_SPLIT();
    L_START INTEGER := 1;
    L_DELIMITER_LENTH INTEGER := LENGTH(P_DELIMITER);
    L_SUBSTR INTEGER := 0;
BEGIN
    IF P_STR IS NULL OR P_DELIMITER IS NULL THEN
        RETURN L_RESULT;
    END IF;
    WHILE  L_START <= LENGTH(P_STR) LOOP
        L_SUBSTR := INSTR(P_STR, P_DELIMITER, L_START);
        IF L_SUBSTR = 0 THEN
            L_RESULT.EXTEND(1);
            L_RESULT(L_RESULT.LAST) := SUBSTR(P_STR, L_START);
            L_START := LENGTH(P_STR) + 1 ;
        ELSE
            L_RESULT.EXTEND(1);
            L_RESULT(L_RESULT.LAST) := SUBSTR(P_STR, L_START,  L_SUBSTR - L_START);
             L_START := L_SUBSTR + L_DELIMITER_LENTH;
        END IF;
    END LOOP;

    RETURN L_RESULT;
END FN_SPLIT_STR;
```

调用函数得到集合：
```sql
select fn_split('111,222', ',')  from dual;
```

查询结果就是集合，类似 `List<Object>`

集合对象在存过中的使用示例：


```sql
create or replace package TYPE_TEST_DEMO is
    procedure init ;
    procedure exe_test ;
end TYPE_TEST_DEMO ;
/
create or replace package body TYPE_TEST_DEMO is

    TYPE T_VALUE2S IS RECORD(
        value0 VARCHAR2(100),
        value1 VARCHAR2(100));
    TYPE T_VALUE2S_F IS TABLE OF T_VALUE2S INDEX BY BINARY_INTEGER;
    C_COLLECTTION T_VALUE2S_F ;    
        
    procedure exe_qy_out(i_sql in varchar2, o_value2s out T_VALUE2S_F) is
    begin
      --dbms_output.put_line(i_sql);
      EXECUTE IMMEDIATE i_sql bulk collect
      into o_value2s;
    end exe_qy_out;
        
    procedure init is
       v_sql varchar2(1000);
    begin
        v_sql := 'SELECT PARAM_KEY, PARAM_VALUE  FROM AMS_TASK_PROCEDURE_PARAM_TD ';
        
        exe_QY_out(v_sql, C_COLLECTTION);
        for i in 1 .. C_COLLECTTION.count() LOOP
            DBMS_OUTPUT.PUT_LINE(C_COLLECTTION(i).value0 || C_COLLECTTION(i).value1 );
        end loop;
    end init;

    procedure exe_test is
    begin
        init;
    end exe_test;
   
end TYPE_TEST_DEMO ;
/



create or replace package  small_type_demo is
    procedure TEST_EXE is
end;
/
create or replace package body small_type_demo is
    procedure TEST_EXE is
        v_tmp_count1 number; 
        v_tmp_count2 number; 
        type m_rules is record(
          classgrpcode  varchar2(10), --险种代码
          classgrpcodename  varchar2(100)); --险种大类

        type t_rules is table of m_rules index by binary_integer; --存放险种代码的集合
        v_rules t_rules;
        
    begin
        SELECT classgrpcode,classgrpcodename bulk collect
          into v_rules  FROM AMS_CLASSGRP_TC ;
        
        v_tmp_count1 := v_rules.count;
        DBMS_OUTPUT.PUT_LINE('v_tmp_count:' || v_tmp_count1);  
        v_tmp_count2 := v_rules.count();
        DBMS_OUTPUT.PUT_LINE('v_tmp_count():' || v_tmp_count2);
        for i in 1 .. v_tmp_count2 loop
          --读取集合内容
          DBMS_OUTPUT.PUT_LINE('classgrpcode:' || v_rules(i).classgrpcode || 
                                      ' classgrpcode:' || v_rules(i).classgrpcodename);

        end loop ;
    end TEST_EXE;
end small_type_demo;

```

### 13 查表的数据量

执行SQL 更新oracle记录的最新数据:
```sql
CREATE OR REPLACE PROCEDURE ALL_TABLE_STATISTICS_COUNT
IS
V_SQL VARCHAR2(1000) DEFAULT '';
BEGIN

FOR RS IN ( SELECT T.TABLE_NAME FROM USER_TABLES T )LOOP
V_SQL :='ANALYZE TABLE '||RS.TABLE_NAME||' COMPUTE STATISTICS';
EXECUTE IMMEDIATE V_SQL;
COMMIT;
END LOOP;
EXCEPTION
WHEN OTHERS THEN
DBMS_OUTPUT.PUT_LINE('ERRM ALL_TABLE_STATISTICS_COUNT:' || SQLERRM);
END;
/
BEGIN
  ALL_TABLE_STATISTICS_COUNT ;
END ;
/
DROP PROCEDURE ALL_TABLE_STATISTICS_COUNT ;
```

查询表记录

```sql
SELECT T.TABLE_NAME,T.NUM_ROWS FROM USER_TABLES T
```


### 14、查询存过

执行SQL 查询存过、存过包、函数等:

```sql
-- 查询包含特定字符串的存过
SELECT NAME , LINE, TEXT FROM USER_SOURCE WHERE NAME = 'MM_MIRROR_PKG_SS'
AND (TEXT like '%$$plsql_unit%' or TEXT like '%$$PLSQL_UNIT%');

-- 查询需要处理行号集
SELECT NAME , LISTAGG(LINE, ',')WITHIN GROUP(ORDER BY NAME) AS LINE
FROM USER_SOURCE WHERE 1=1 -- AND NAME = 'AMS_HQ_CS'
AND (TEXT like '%$$plsql_unit%' or TEXT like '%$$PLSQL_UNIT%')
GROUP BY  NAME

```

### 15 查表分区

```sql
select tablespace_name, sum(bytes)/1024/1024 from dba_data_files group by tablespace_name;
```



## Oracle 分组统计，按照天、月份周和自然周、月、季度和年

1.按天


```sql
select to_char(t.STARTDATE+15/24, 'YYYY-MM-DD') as 天,sum(1) as 数量
from HOLIDAY t
group by to_char(t.STARTDATE+15/24, 'YYYY-MM-DD') 
ORDER by 天 NULLS  LAST;

select trunc(t.STARTDATE, 'DD') as 天,sum(1) as 数量
from HOLIDAY t
group by trunc(t.STARTDATE, 'DD')
ORDER by 天 NULLS  LAST;
```

2.按周

```sql
select to_char(next_day(t.STARTDATE+15/24 - 7,2),'YYYY-MM-DD') AS 周,sum(1) as 数量 
from HOLIDAY t 
group by to_char(next_day(t.STARTDATE+15/24 - 7,2),'YYYY-MM-DD')ORDER BY 周;


-- 按自然周统计 
select to_char(t.STARTDATE,'iw') AS 周,sum(1) as 数量
from HOLIDAY t
group by to_char(t.STARTDATE,'iw')
ORDER BY 周;
```

3.按自然月

```sql
select to_char(t.STARTDATE,'YYYY-MM') as 月份,sum(1) as 数量
from HOLIDAY t
GROUP BY  to_char(t.STARTDATE,'YYYY-MM')
ORDER BY 月份;
```

4.按季度

```sql
select to_char(t.STARTDATE,'q') 季度,sum(1) as 数量
from HOLIDAY t
group by to_char(t.STARTDATE,'q')
ORDER BY 季度 NULLS  LAST;
```
    

5.按年

```sql
select to_char(t.STARTDATE,'yyyy') AS 年度,sum(1) as 数量
from HOLIDAY t
group by to_char(t.STARTDATE,'yyyy')
ORDER BY 年度;
```


## 常见错误 

### 锁账户

账号被锁定的解决办法

错误场景：当使用sqlplus进行登录时报错：ORA-28000 账号被锁定。
错误原因：由于oracle 11g 在默认在default概要文件中设置了密码最大错误次数为10，“FAILED_LOGIN_ATTEMPTS=10”，密码错误的次数超过10次，账号就会被锁定。
解决方案：

使用管理员账户SYSTEM 登录。

1.查看用户使用的概要文件名，一般为DEFAULT
	

```sql
 SELECT username,profile FROM dba_users;
```

2.查看概要文件中设置的密码错误后限制的登录次数
	
```sql
SELECT * FROM dba_profiles WHERE profile='DEFAULT' and resource_name='FAILED_LOGIN_ATTEMPTS';
```

3.如图，将10次（默认）改为不受限制，改动后立即生效

```sql
ALTER profile default limit failed_login_attempts unlimited;　
```

4.检查已经被锁定的用户
	
```sql
SELECT username,account_status FROM dba_users;
```

如图，账号的状态大致被分为：
- OPEN(正常)，
- LOCKED（通过SQL语句进行的锁定），
- LOCKED(TIMED)（超过最大错误登录次数被动锁定），
- EXPIRED或者EXPIRED(GRACE)（密码过期状态），
- EXPIRED & LOCKED(TIMED)（密码过期并超过了限制次数被锁定）等。

解锁被锁定的账户
```sql
ALTER user user_name account unlock;
```


### 锁表

查看被锁的表和解锁

 
以下几个为相关表
```sql
SELECT * FROM v$lock;
SELECT * FROM v$sqlarea;
SELECT * FROM v$session;
SELECT * FROM v$process ;
SELECT * FROM v$locked_object;
SELECT * FROM all_objects;
SELECT * FROM v$session_wait;
```

查看被锁的表

```sql
SELECT b.owner,b.object_name,a.session_id,a.locked_mode FROM v$locked_object a,dba_objects b WHERE b.object_id = a.object_id;
```

查看那个用户那个进程照成死锁

```sql
SELECT b.username,b.sid,b.serial#,logon_time FROM v$locked_object a,v$session b WHERE a.session_id = b.sid order by b.logon_time;
```

查看连接的进程

```sql
SELECT sid, serial#, username, osuser FROM v$session;
```

3.查出锁定表的sid, serial#,os_user_name, machine_name, terminal，锁的type,mode

```sql
SELECT s.sid, s.serial#, s.username, s.schemaname, s.osuser, s.process, s.machine,
s.terminal, s.logon_time, l.type
FROM v$session s, v$lock l
WHERE s.sid = l.sid
AND s.username IS NOT NULL
ORDER BY sid;
```

这个语句将查找到数据库中所有的DML语句产生的锁，还可以发现，
任何DML语句其实产生了两个锁，一个是表锁，一个是行锁。

杀掉进程 sid,serial#
```sql
ALTER system kill session'210,11562';
```


oracle 分区


```sql
-- 查询数据库中不同用户的分区表的数目
select owner,count(1) from dba_tables where partitioned='YES' group By owner; 
 
-- 查询数据库中用户的分区表
select * from dba_tables where partitioned='YES'  and owner='数据库用户名' ; 

--查询数据库中 该用户下的对应表的分区字段
select * from dba_part_key_columns where name='表名' and owner ='数据库用户名';  
 
 ----查看该数据库中 所有用户的 所有分区表的和对应分区字段
SELECT * FROM all_PART_KEY_COLUMNS;
 
 
SELECT * FROM all_PART_KEY_COLUMNS t where  t.owner='数据库用户名'  and  t.name  in（select table_name from dba_tables where partitioned='YES'  and owner='数据库用户名' ）; 
--查询数据库中，该用户下对应的分区表的表名 和分区表所对应的分区字段

```

生成insert into select 

```sql
SELECT 'INSERT INTO ' || X.TABLE_NAME ||'_NEW (' || COLUMN_NAMES||') SELECT ' || COLUMN_NAMES ||' FROM ' || X.TABLE_NAME ||'_OLD'
AS MY_SQL
FROM
(SELECT T.TABLE_NAME,LISTAGG(T.COLUMN_NAME , ',')WITHIN GROUP(ORDER BY T.COLUMN_ID)  AS COLUMN_NAMES
FROM USER_TAB_COLUMNS T WHERE T.TABLE_NAME = 'AMS_BAD_DEBTS_LIST_TD'
GROUP BY T.TABLE_NAME ) X ;
```

表存在则删除操作

```sql
declare
    table_name varchar(100) := 'tb_test';
    num number;
begin
    select count(1) into num from user_tables where table_name = upper(table_name) ;
    if num > 0 then,
        execute immediate 'drop table '||table_name ;
    end if;
end;
```


分区表与非分区表查询

```sql
--查询用户分区表
SELECT table_name FROM dba_tab_partitions 
WHERE table_owner='CFMS' AND table_name not like 'BIN$%' GROUP BY table_name
```

```sql
--查询用户非分区表
SELECT ut.* FROM user_tables ut WHERE ut.TABLE_NAME NOT IN(
   SELECT table_name FROM dba_tab_partitions WHERE table_owner ='CFMS' 
AND table_name NOT LIKE 'BIN$%' GROUP BY table_name
)
```


```sql
--查询分区表各分区占用空间大小（二级分区的统计，存在二级分区字段名称截取（不叫branchcode的需要调整截取长度），单表查询）
SELECT 
   b.segment_name,b.partition_name,sum(b.BYTES)/1024/1024
FROM
   (select
       ds.segment_name,substr(ds.partition_name,0,10) partition_name,ds.segment_type,ds.tablespace_name,ds.bytes
   FROM
      dba_segments ds where segment_name = 'tableName'
) b
group by 
   b.segment_name,b.partition_name
order by
   b.segment_name,b.partition_name;
```

```sql

--查询分区表个人去占用空间大小（一级分区查询，针对存在二级分区的统计会有问题）
SELECT 
b.partition_name,sum(b.BYTES)/1024/1024 
FROM 
b.owner='CFMS'
and a.table_name=b.segment_name
and a.partition_name=b.partition_name
and b.segment_name IN(
   SELECT table_name FROM dba_tab_partitions WHERE table_owner='CFMS' and table_name NOT LIKE 'BIN$%' GROUP BY table_name
)
GROUP BY
   b.partition_name
ORDER BY b.partition_name;
```

查表空间与大小

```sql
select tablespace_name, sum(bytes)/1024/1024 from dba_data_files group by tablespace_name;
```
