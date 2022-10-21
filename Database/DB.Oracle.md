---
layout: default
title: ORACLE
nav_order: 2
parent: Database
---

# ORACLE
{: .no_toc }

## Table of contents
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
grant create table to user_name;
grant create trigger to user_name;
grant create type to user_name;
grant create view to user_name;
grant unlimited tablespace to user_name;
alter user user_name quota unlimited on tbs_name;

grant debug any procedure to user_name;
grant debug connect session to user_name;
```

5.然后再以创建的用户登录，登录之后创建表即可。

```sql
conn username/password;
```


#### 表的重命名：

```sql
alter table old_table_name rename to new_table_name
```

说明：alter table 表名 rename to 新表名

示例：

```sql
alter table sf_user rename to t_user;
```



#### 字段补偿

（1）**增加字段语法**：

```sql
alter table table_name add (column datatype [default value][null/not null],….);
```

说明：alter table 表名 add (字段名 字段类型 默认值 是否为空);

示例：

```sql
alter table t_users add (user_image blob);

alter table t_users add (userName varchar2(30) default '空' not null);
```



（2）**修改字段的语法：**

- modify 改列类型或长度、非空。
- rename 改名字。

```sql
alter table table_name modify (column datatype [default value][null/not null],….); 
```

说明：alter table 表名 modify (字段名 字段类型 默认值 是否为空);

示例：

```sql
ALTER TABLE t_users MODIFY (BILLCODE number(4));
```


```sql
alter table table_name modify (column datatype [default value][null/not null],….); 
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
select distinct t1.TABLE_NAME,t.COLUMN_NAME
from DBA_TAB_COLS t,
     DBA_TABLES t1
where t.TABLE_NAME = t1.TABLE_NAME
  and t.OWNER = 'PAYMT' -- 用户
  and t.COLUMN_NAME = 'VEHKIND' --列表
order by t1.TABLE_NAME,t.COLUMN_NAME;

-- 拼接出字段修改类型修改sql（t.OWNER指定用户，大写；t.COLUMN_NAME要搜索的字段；字段类型及长度需要修改为自己想要的）
select 'alter table ' || t.TABLE_NAME || ' modify ' || t.COLUMN_NAME || ' varchar2(30);'
from DBA_TAB_COLS t,
     DBA_TABLES t1
where t.TABLE_NAME = t1.TABLE_NAME
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
alter table table_name  rename  column  old_cloumn_name to  new_cloumn_name  
```

说明：alter table 表名 rename  column 列名 to 新列名  （其中：column是关键字）

示例：

```sql
alter table t_users rename column  nlcke_name  to  nick_name;
```

#### 删除数据

（1）删除数据，没有事务，无法回滚。

```sql
truncate table table_name ;
```

（2）删除数据，有事务，可以回滚。

```sql
delete from table_name ;
```

（3）删除表和数据。

```sql
drop table table_name ;
```

#### 检测中文乱码

可用函数

- `ASCIISTR` 函数可以使中文变成 '\xxxx' ;
- `UNISTR` 函数可以反转回中文。

```sql
select asciistr('安诚') from  dual ;

select unistr('\5B89\8BDA') from  dual ;
```

检测指定的字段是否含有中文乱码:

```sql
select column_name from table_name where asciistr('column_name') like '%??%' or asciistr('column_name') like '%\FFFD%' ;
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
        for v_table_column_data in (select * from user_tab_columns where Table_Name=v_table_name ORDER BY COLUMN_ID ASC ) loop

            begin
            v_table_column := v_table_column_data.COLUMN_NAME ;
            if  instr(v_table_column_data.DATA_TYPE, 'VARCHAR') > 0 then
                -- 判断乱码
                --v_sql := 'select count(1) from  :v_table_name where asciistr(:v_table_column) like ''%??%'' or asciistr(:v_table_column) like ''%\FFFD%'' ';
                --execute immediate v_sql into v_count  using v_table_name, v_table_column,v_table_column;
                v_sql := 'select count(1) from '|| v_table_name || ' where asciistr('|| v_table_column ||') like ''%??%'' or asciistr('|| v_table_column ||') like ''%\FFFD%'' ';
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
    d_sql varchar2(500) := 'drop table '||r_table_name ;
    c_sql varchar2(500) := 'create table '||r_table_name||' (table_name varchar2(100), column_name  varchar2(100))';
    i_sql varchar2(500) := 'insert into '||r_table_name||' (table_name, column_name) values (:c1, :c2)';
    -- 查询所有的表
    cursor v_table_names is
        SELECT * FROM USER_TABLES U;

begin
    -- 结果表
    SELECT count(1) into v_count FROM USER_TABLES where TABLE_NAME = r_table_name;
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
        for v_table_column_data in (select * from user_tab_columns where Table_Name=v_table_name ORDER BY COLUMN_ID ASC ) loop

            begin
            v_table_column := v_table_column_data.COLUMN_NAME ;
            if  instr(v_table_column_data.DATA_TYPE, 'VARCHAR') > 0 then
                -- 判断乱码
                 v_sql := 'select count(1) from '|| v_table_name || ' where asciistr('|| v_table_column ||') like ''%??%'' or asciistr('|| v_table_column ||') like ''%\FFFD%'' ';
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

### 常用查询

#### 查询数据库全部序列

 
--查看当前用户的所有序列 PAYMT 为数据库名称

```sql
select SEQUENCE_OWNER,SEQUENCE_NAME from dba_sequences where sequence_owner='PAYMT';
```

#### 查看数据库版本：

```sql
select * from v$version;
```

查询结果：

| BANNER | CON_ID |
| ------ | ------ |
|Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production | 0 |
|PL/SQL Release 12.2.0.1.0 - Production | 0 |
|CORE	12.2.0.1.0	Production | 0 |
|TNS for 64-bit Windows: Version 12.2.0.1.0 - Production| 0 |
|NLSRTL Version 12.2.0.1.0 - Production | 0 |


#### 查询表-字段-注释

查表DDL最后更新时间

```sql
select uat.table_name as tableName,
    (select last_ddl_time from user_objects where  OBJECT_TYPE='TABLE' AND object_name = uat.table_name ) as lasDdlTime
from user_all_tables uat ;
```

查用户所有的表:

```sql
-- 查指定用户的所有表
SELECT TABLE_NAME FROM DBA_TABLES WHERE OWNER='用户名' ;
```

```sql
-- 查用户所有的表
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
select * from user_tab_columns where Table_Name='T_APPLICATIONS' ORDER BY COLUMN_ID ASC;
-- 查（全部用户）表名全部的列
select * from all_tab_columns where Table_Name='T_APPLICATIONS';
-- 查（全部用户，含系统表）表名全部的列
select * from dba_tab_columns where Table_Name='T_APPLICATIONS';
```

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
select* from all_indexes where table_name='T_APPLICATIONS';

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

#### 查询表约束

查找表约束：`select constraint_name from user_cons_columns a where a.table_name='tablename';`

增加表约束：`alter table tablename add constraint pk_name primary key(column);`

删除表约束：`alter table tablename drop constraint constraint_name `----（SYS_C002715）;

修改表约束：1）禁用表主键：`alter table tablename disable primary key;`

　　　　　　2）启用表主键：`alter table tablename enable primary key;`

　　　　　　3）重命名表主键：`alter table tablename rename constraint pk_id to new_pk_id;`

```sql
--查外键约束
select * from user_cons_columns cl where cl.constraint_name = 外键名称

--查当前schema的约束
select table_name,constraint_name,constraint_type from user_constraints
where table_name='大写的表名'

--查所有的约束
select table_name,constraint_name,constraint_type from dba_constraints
where table_name='大写的表名'


-- 查约束
SELECT constraint_name, table_name, r_owner, r_constraint_name
FROM all_constraints
WHERE table_name = 'table_name' and owner = 'owner_name';

--另一个视图ALL_CONS_COLUMNS也包括组成表上约束列的信息。
```

生成启动、禁用约束的SQL

```sql
--查询出所有表的唯一键约束的,生成禁用sql
select 'alter table ' || table_name || ' disable constraint '||constraint_name||';'
from user_constraints where constraint_type='U';
```

如下：

```sql
alter table MM_PERMISSIONS_TD disable constraint MM_PERMISSIONS_TD_UNIQUE ;
alter table MM_PLAN_TD disable constraint U_PLAN_TD;
alter table MM_PREPAYINFO_TD disable constraint U_PREPAYINFO_TD_01;
alter table MM_PREPAYMENTIN_TD disable constraint U_PREPAYMENTID_TD_01;
```


```sql
--查询出所有表的唯一键约束的,生成启用sql
select 'alter table ' || table_name || ' enable constraint '||constraint_name||';'
from user_constraints where constraint_type='U';
```

如下：
```sql
alter table SYENTINFO2 enable constraint UQ_SY20180806;
alter table WEB_INFO enable constraint UQWEBINFO201807211136;
alter table ENT_INFO_WUHAN enable constraint SYS_C0024733;
```

```sql
--查询所有表的所有外键 
select 'alter table ' || table_name || ' enable constraint '||constraint_name||';'
from user_constraints where constraint_type='R';
```

#### 查询表索引


```sql

-- 查用户所有的所有
select * from USER_INDEXES where table_name = '大写的表名'

-- 查所有的所有
select* from all_indexes where table_name='T_APPLICATIONS';


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

#### 查视图

```sql
select * from all_views WHERE OWNER='PAYMT'
ORDER BY VIEW_NAME
```


#### 查看空表-函数：

```sql
select sysdate from dual;
-- 等价于mysql
select CURRENT_TIMESTAMP ;
```

#### TO_CHAR()函数

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

#### decode 函数

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

#### concat 函数

concat(字符串1, 字符串2)

该函数的含义如下：

将字符串1和字符串2拼接到一起。


### PL/SQL 相关

#### PLSQL Developer解决中文乱码问题

1.查服务端字符集编码

```sql
select userenv('language') from dual;
```

结果

```
    userenv('language')
1	SIMPLIFIED CHINESE_CHINA.AL32UTF8
```

2.执行语句 `select * from V$NLS_PARAMETERS` 查看第一行中PARAMETER项中为NLS_LANGUAGE 对应的VALUE项中是否和第一步得到的值一样。

```sql
select * from V$NLS_PARAMETERS 
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





#### 存储过程

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
     using ''192.168.1.100:1521/orcl''
     ';

   

   --    open out_cursor for 'select * from A@dblink1';
  

  -- 返回个数值
    out_cursor :=0;
    execute immediate 'select count(*) from A@dblink1'  into out_cursor ;
    
    -- 取消连接
    execute immediate 'drop database link dblink1';
    
    return ;
    
end  ;
```



### 常见错误

#### 锁账户

账号被锁定的解决办法

错误场景：当使用sqlplus进行登录时报错：ORA-28000 账号被锁定。
错误原因：由于oracle 11g 在默认在default概要文件中设置了密码最大错误次数为10，“FAILED_LOGIN_ATTEMPTS=10”，密码错误的次数超过10次，账号就会被锁定。
解决方案：

使用管理员账户SYSTEM 登录。

1.查看用户使用的概要文件名，一般为DEFAULT
	

```sql
 select username,profile from dba_users;
```

2.查看概要文件中设置的密码错误后限制的登录次数
	
```sql
select * from dba_profiles where profile='DEFAULT' and resource_name='FAILED_LOGIN_ATTEMPTS';
```

3.如图，将10次（默认）改为不受限制，改动后立即生效

```sql
alter profile default limit failed_login_attempts unlimited;　
```

4.检查已经被锁定的用户
	
```sql
select username,account_status from dba_users;
```

如图，账号的状态大致被分为：
- OPEN(正常)，
- LOCKED（通过SQL语句进行的锁定），
- LOCKED(TIMED)（超过最大错误登录次数被动锁定），
- EXPIRED或者EXPIRED(GRACE)（密码过期状态），
- EXPIRED & LOCKED(TIMED)（密码过期并超过了限制次数被锁定）等。

解锁被锁定的账户
```sql
alter user user_name account unlock;
```


#### 锁表

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
select b.owner,b.object_name,a.session_id,a.locked_mode from v$locked_object a,dba_objects b where b.object_id = a.object_id;
```

查看那个用户那个进程照成死锁

```sql
select b.username,b.sid,b.serial#,logon_time from v$locked_object a,v$session b where a.session_id = b.sid order by b.logon_time;
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
alter system kill session'210,11562';
```

