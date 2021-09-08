

## oracle 



Oracle 在线学习：https://livesql.oracle.com

### 基础语法

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

```sql
alter table table_name modify (column datatype [default value][null/not null],….); 
```

说明：alter table 表名 modify (字段名 字段类型 默认值 是否为空);

示例：

```sql
alter table t_users modify (BILLCODE number(4));
```

高级玩法：


```SQL
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
alter table table_name drop (column column_name);
```

说明：alter table 表名 drop column 字段名;

示例：

```
alter table t_users drop column user_image;
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



### 常用查询



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

#### 查看空表-函数：

```sql
select sysdate from dual;
-- 等价于mysql
select CURRENT_TIMESTAMP ;
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

```SQL
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

#### 账号被锁定的解决办法

错误场景：当使用sqlplus进行登录时报错：ORA-28000 账号被锁定。
错误原因：由于oracle 11g 在默认在default概要文件中设置了密码最大错误次数为10，“FAILED_LOGIN_ATTEMPTS=10”，密码错误的次数超过10次，账号就会被锁定。
解决方案：

使用管理员账户SYSTEM 登录。

1.查看用户使用的概要文件名，一般为DEFAULT
	

```SQL
 select username,profile from dba_users;
```

2.查看概要文件中设置的密码错误后限制的登录次数
	
```SQL
select * from dba_profiles where profile='DEFAULT' and resource_name='FAILED_LOGIN_ATTEMPTS';
```

3.如图，将10次（默认）改为不受限制，改动后立即生效

```SQL
alter profile default limit failed_login_attempts unlimited;　
```

4.检查已经被锁定的用户
	
```SQL
select username,account_status from dba_users;
```

如图，账号的状态大致被分为：
- OPEN(正常)，
- LOCKED（通过SQL语句进行的锁定），
- LOCKED(TIMED)（超过最大错误登录次数被动锁定），
- EXPIRED或者EXPIRED(GRACE)（密码过期状态），
- EXPIRED & LOCKED(TIMED)（密码过期并超过了限制次数被锁定）等。

解锁被锁定的账户
```SQL
alter user user_name account unlock;
```