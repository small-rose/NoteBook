---
layout: default
title: OceanBase 
nav_order: 60
parent: Database
---

# OceanBase Learn
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

--- 

### OceanBase v3.2.3.3


## OB 兼容问题

0、通用问题

经测试 OB 不支持 `$$PLSQL_UNIT` 

解决办法：

(1）人工修改

(2）导出包体脚本后可执行批量替换。

1、Sql-20221208-01-AMS_BASE_DB_ACCT_PKG

```
for i in 1 .. C_BASE_DB_ACCT_DEL.count() loop
```

去掉count后面去掉()改为

```
for i in 1 .. C_BASE_DB_ACCT_DEL.count loop
```


2、Sql-20221208-02-AMS_POLICYINSURANCEMATCH_PKG

无效的字符：

將中文字符修改成英文。

3、Sql-20221208-03-AMS_POM_GATHER_TD_NEW_PKG(i-V_MONTH)

去掉括号改为i-V_MONTH

4、sql-20221208-04-mm_mirror_pkg_usabnogroup

约4217行

ROUND((V_REMAINS)*v_rate,2)

改为


## OB优化指导


1、OB库表的数据接近10亿条，必须要做表分区。


2、多台机器部署涉及到分布式操作时。

可以不使用OB负载均衡.

比如根据业务特征，表关联密切，人工管理表到指定机器节点，以减少高频操作的分布式跨机器执行，尽可能SQL执行效率（不是一定可以提高，具体还是要看业务特征。）

也可以根据业务特征，进行统一的表分区。进行表组绑定。让相同的分区字段的表分区在某些节点上，

比如上海分公司在节点1，江苏、浙江在节点2.

如果要建表组，必须是（1、分区表2、分区表的分区数一致）才。


3、OB不建议使用二级分区表。

二级分区表调整为一级分区表，需要自己手工整理出SQL脚本，压测环境和生产环境执行这种SQL脚本。

对索引变动影响

查询条件带上分区条件的时候，自动道指定分区检索。可以适当创建联合索引，一般使用本地索引。

- LOCAL索引：分区有多少个，就有多少份LOCAL索引，其实是分区内的索引
- GLOBAL索引： 一个索引，对所有的分区数据进行排序。涉及的跨分区排序的时候，可以使用。

二级分区表调整为一级分区表后，表分区不一样，对导数据有没有影响？
数据影响小


4、其他兼容问题

针对Oracle模式兼容Oracle问题。



需要sys租户下查看
oceanbase.__all_virtual_table表中的 index_type 字段判断索引的类型，其中字段值和索引类型如下
```
  INDEX_TYPE_NORMAL_LOCAL = 1, // 局部普通索引
  INDEX_TYPE_UNIQUE_LOCAL = 2, // 局部唯一索引
  INDEX_TYPE_NORMAL_GLOBAL = 3, // 全局普通索引
  INDEX_TYPE_UNIQUE_GLOBAL = 4, // 全局唯一索引
  INDEX_TYPE_PRIMARY = 5,
  INDEX_TYPE_DOMAIN_CTXCAT = 6, // 全文索引
  INDEX_TYPE_NORMAL_GLOBAL_LOCAL_STORAGE = 7, // 全局索引，局部存储
  INDEX_TYPE_UNIQUE_GLOBAL_LOCAL_STORAGE = 8, // 全局索引，局部存储
```
其中INDEX_TYPE_NORMAL_GLOBAL和INDEX_TYPE_UNIQUE_GLOBAL是全局索引，其他类型都为局部索引。

对于非分区表，OB内部对建索引做了一个优化，对外表现是GLOBAL索引，实质上是创建的LOCAL索引，对应的TYPE就是INDEX_TYPE_NORMAL_GLOBAL_LOCAL_STORAGE，index_type=3和index_typ=4是真实的全局索引，索引有自己独立的分区。


## OB 内存挤占

### 内存管理机制与挤占根源

OceanBase 将内存从逻辑上划分为几个关键部分，其动态平衡是挤占发生的根源：

 - 不可动态伸缩的内存：主要指 MemStore，用于缓存数据库的增量写操作数据（如插入、更新）。当数据达到一定规模，会通过转储/合并操作持久化到磁盘，从而释放内存。这是写入操作消耗内存的主要部分。
 
 - 可动态伸缩的内存：主要指 KV Cache，包含数据块缓存、索引缓存等多种缓存，目的是加速数据读取。设计上，这部分内存是“可挤占”的，即当系统其他部分（主要是MemStore）需要内存时，可以通过 wash（洗出）机制淘汰掉一些缓存内容，将内存让出。

 - 其他内存组件：包括SQL执行时算子（如排序、哈希连接）使用的工作区内存、执行计划缓存等。
 
这种设计在理想情况下非常高效。但在特定压力下，平衡会被打破，导致内存挤占，主要发生在以下三个层面：

 - 租户内存超限：单个业务租户的内存使用量超过其配置的上限。
 
 - 服务器内存耗尽：整个OBServer进程的内存使用量达到配置的memory_limit。
 
 - 物理内存耗尽：服务器操作系统的物理内存被耗尽。
 
###  🔍 主要挤占场景分析

内存挤占通常由以下一种或多种情况共同引发：

####  1. 租户内存超限

这是最常见的情况，通常与数据写入和MemStore相关。

写入过快，转储不及：当业务进行批量数据导入、大规模更新或删除时，MemStore内存会快速上涨。

如果写入速度持续超过转储/合并到磁盘的速度，MemStore占用的内存就无法及时释放，最终耗尽租户内存，导致后续写入失败。

非MemStore模块占用过高：有时，MemStore本身占用并不高，但租户内存仍被耗尽。

这通常是因为其他内存模块消耗了大量内存，挤占了MemStore应有的空间。

常见原因包括：

 - 大查询：需要大量内存的SQL算子（如排序HASH JOIN、分组MERGE GROUP BY）可能瞬间申请大块工作区内存。
 
 - 复杂执行计划：生成复杂的查询执行计划（如涉及CTE优化、复杂索引条件抽取）可能消耗大量内存。
 
 - 内存泄漏：极少数情况下，特定模块可能存在缺陷，导致内存无法在操作结束后正常释放，造成内存泄漏。
 
 - KV Cache洗出失败：当租户内存不足时，理论上KV Cache应该被洗出以释放空间。但如果缓存的数据正被活跃事务引用（ref_count不为2），就可能无法被洗出，导致内存回收失败，加剧内存紧张。
 
#### 2. 服务器内存耗尽

这通常与系统级的配置和管理有关。
500租户超卖：500租户是一个特殊的内部租户，许多系统内部操作和缓存（如Schema缓存）使用其内存。如果多个业务租户的规格总和配置过高（即“超卖”），或者500租户自身的内存被某些系统操作（如日志归档任务ArcClogTask）大量占用，就可能导致整个OBServer进程的内存达到上限
。
系统缓存未设上限：OBServer会使用一些缓存来提升性能（如memory_chunk_cache_size）。如果这些缓存没有设置合理的上限，可能会占用大量内存，导致可用于租户的内存减少
。
#### 3. 物理内存耗尽

这通常是由于基础设施层面的资源规划不足或部署不当造成的。

内存超卖：在云环境或虚拟化环境中，可能会超卖物理内存。如果一台机器上部署了多个OBServer实例，或者OBServer的memory_limit参数设置得过高，接近机器物理内存，当其他系统进程也需要内存时，就可能触发操作系统的OOM Killer，强制杀死OBServer进程以保护系统。

###  🛠️ 如何排查与缓解？

当出现内存相关的告警或错误（如-4013）时，可以遵循以下思路：

确定类型：首先查看OBServer日志（搜索[OOPS]），确定内存爆的具体类型（例如TENANT_HOLD_REACH_LIMIT或PHYSICAL_MEMORY_EXHAUST）。

定位热点：根据类型，进一步分析内存元信息。例如，对于租户内存爆，可以查看该租户下哪个CTX_ID（内存上下文）占用最高，然后再分析该CTX_ID下哪个具体的内存模块（mod）是“罪魁祸首”。

针对性行动：

 - 针对写入过快：考虑对大批量写入操作进行限流或分批次进行，给转储/合并留出时间。也可以评估是否适当调大租户内存规格（治本）或调整触发转储的阈值（freeze_trigger_percentage）。

 - 针对大查询：优化消耗内存过大的SQL语句，避免不必要的复杂操作。可以调整工作区内存参数（ob_sql_work_area_percentage），但更关键的是优化SQL本身。

 - 针对500租户或系统缓存：检查租户资源规划是否合理，避免超卖。检查并合理设置系统缓存的上限。

 - 针对物理内存耗尽：需要从基础设施层面解决，确保分配给OBServer的内存上限合理，并考虑为操作系统和其他进程预留足够的内存


##  大批量数据批处理优化

Ocean Base 数据库基础设施和系统级配置已经相对稳定，不会轻易调整。

而且最常见的内存挤占一般发生在数据写入和MemStore.相关区域，属于应用层影响范围，因此在应用层使用过程中必须十分注意：

OceanBase更适合短（耗时短）、平（操作简单）、快（见效快）的执行处理。因为存储过程要以此为目标，向短、平、快的方向靠近。

**注意点1**：使用存储过程处理数据时避免长时间占用不结束执行，同时结合数据设置的超时时间限制。

**注意点2**：针对大批量数据处理，需要才有分批次的方式进行，给MemStore.转储留出时间。也需要尽快执行完毕，快速释放占用资源，如单次批量时间控制在半小时以内。

**注意点3**：嵌套FOR 循环使用游标是不允许的，即使在 FOR 循环中执行commit提交语句，存储过程DML相关语句使用的内存均不会被释放，必须整个存储过程调用块全部结束才能释放占用的内存。如使用while或loop进行替换


历史总结的大批量数据处理方式：

（1）对大批量数据，如已经产出稳定的主键，可使用主键ID分段处理，提前预算要处理的数据批次进行存储到任务中间表，执行时依次将每组的ID按10万条1组的结果进行执行，每执行1次处理10万数据。可参考共保摊赔案例AMS_MIRROR_NEWREP_PKG.p_daily_rep

（2）针对大批量数据同时处理步骤较多的场景，无法使用主库ID分段，则可以进行分步处理，将步骤标记和必要参数进行配置存储，每次执行时读取必要参数和步骤标记，根据步骤对应执行步骤逻辑，当前步骤执执行完毕，更新步骤标记。

（3）针对多表的大批量数据，如出现不同分公司部分数据量差异极大时，针对不可变数据如保单号进行HASH分组，均衡数据，单次大批量数据无法处理。

（4）以上方案也可适当组合，最终目标向短平块靠齐.


## queuing表

变成queuing表
```
ALTER TABLE AMS_APPLICATIONS_IDX TABLE MODE = 'queuing';
```

## 表组

一.表组相关命令

```sql
show tablegroups;--查询所有表组

show tablegroups where tablegroup_name='TG_1';--查询表组名为TG_1的表组信息

SHOW TABLEGROUPS WHERE Tablegroup_name='TG_1';--查询表组TG_1的分区信息

DROP TABLEGROUP tblgroup1;--删除表组

ALTER TABLE fbs1 SET TABLEGROUP '';--把fbsl移出表组

```

二、建模拟一级分区表，建分区表组。

结论：分区表组和分区表组内的分区表分区结构必须要统一，分区键名称可以不一样

1.新建分区表

```sql
create table fbs1
(
SUBCOMPANY varchar2(10),
fgs varchar2(20),
classcode varchar2(20)
)
 partition by list("SUBCOMPANY")
(partition SUBCOMPANY_1010100 values  ('1010100'),
partition SUBCOMPANY_1020100 values  ('1020100'),
partition SUBCOMPANY_OTHER values  (DEFAULT));

create table fbs2
(
SUBCOMPANY varchar2(10),
fgs varchar2(20),
risktype varchar2(20)
)
 partition by list("SUBCOMPANY")
(partition SUBCOMPANY_1010100 values  ('1010100'),
partition SUBCOMPANY_1020100 values  ('1020100'),
partition SUBCOMPANY_OTHER values  (DEFAULT));

create table fbs4
(
SUBCOMPANY varchar2(10),
fgs varchar2(20),
risktype1 varchar2(20)
)
 partition by list("SUBCOMPANY")
(partition SUBCOMPANY_1010100 values  ('1010100'),
partition SUBCOMPANY_1020100 values  ('1020100'),
partition SUBCOMPANY_OTHER values  (DEFAULT));

create table fbs3
(
FGSDM varchar2(10),
fgs varchar2(20),
risktype varchar2(20)
)
 partition by list("FGSDM")
(partition FGSDM_1010100 values  ('1010100'),
partition FGSDM_1020100 values  ('1020100'),
partition FGSDM_OTHER values  (DEFAULT));

create table fbs5
(
FGSDM varchar2(10),
fgs varchar2(20),
risktype1 varchar2(20)
)
 partition by list("FGSDM")
(partition FGSDM_1010100 values  ('1010100'),
partition FGSDM_1020100 values  ('1020100'),
partition FGSDM_1030100 values  ('1030100'));
```

2.新建分区表组

```sql
CREATE TABLEGROUP tg_1 PARTITION BY LIST 1  
(PARTITION SUBCOMPANY_1010100 VALUES ('1010100'), 
PARTITION SUBCOMPANY_1020100 VALUES ('1020100'),
partition SUBCOMPANY_OTHER values  (DEFAULT));
```


3.给分区表组添加分区表
```sql

ALTER TABLE fbs4 TABLEGROUP = tg_1;--单分区表加入表组的方法。不报错

ALTER TABLEGROUP tg_1 ADD fbs1,fbs2;--多个分区表加入表组的方法。不报错

ALTER TABLE fbs3 TABLEGROUP = tg_1;--分区键名称不一样，但是结构内容一样加上表组。不报错

ALTER TABLE fbs5 TABLEGROUP = tg_1;--分区键名称不一样，结构不一样。会报错， table and tablegroup use different partition options not allowed

```
{: .tips }
> 建表可以没有默认分区。没有默认分区的时候可以添加新的分区，有默认分区的时候无法添加新的分区。

## 配置查询

```sql
show global variables like '%timeout%'
```

|variable_name|value|
|-------------|-----|
|connect_timeout  | 10 |
|interactive_timeout |2880 |
|net_read_timeout   |30 |
|net_write_timeout  |60 |
|ob_pl_block_timeout  |3216672000000000 |
|ob_qeury_timeout      |100000000000 |
|ob_trx_idle_timeout   |120000000000 |
|ob_trx_lock_timeout   | -1 |
|ob_trx_timeout   |100000000000 |
|wait_timeout   |86400 |

```sql
set global ob_query_timeout = 100000000000 ;
set global ob_trx_timeout = 100000000000 ;
set global ob_trx_idle_timeout = 1200000000000 ;

```

查租户工作空间内存

```sql
show variables  like  '%area%'   申宇16:47
```


## 查询权限

```sql
SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE = 'AMSAPP';
SELECT * FROM DBA_TAB_PRIVS WHERE GRANTEE = 'AMSAPP';
SELECT * FROM DBA_ROLE_PRIVS WHERE GRANTEE ='AMSAPP'
```

## 对象授权


```sql
--查询对象授权
SELECT * FROM DBA_TAB_PRIVS WHERE TABLE_NAME LIKE 'SEQ_EMAILPROCEDURETD%';

--表/视图赋权
GRANT SELECT,INSERT, UPDATE, DELETE ON AMS.AMS_INVOICE_TD TO AMSAPP;
CREATE OR REPLACE SYNONYM AMSAPP.AMS_INVOICE_TD FOR AMS.AMS_INVOICE_TD;

--序列赋权
GRANT SELECT ON AMS.SEQ_INVOICEDETAIL TO AMSAPP;
CREATE OR REPLACE SYNONYM AMSAPP.SEQ_INVOICEDETAIL FOR AMS.SEQ_INVOICEDETAIL;

-- 函数/存过/包赋权
GRANT EXECUTE ON AMS.CONFIRMATION_LETTER_GENERATE TO AMSAPP;
```

## 查视图

```sql
select * from v$version;

select * from gv$sql_audit;
```
QUERY_SQL 实际执行的SQL语句
PLAN_TYPE 执行计划类型：
- 1：本地执行计划（Local）
- 2：远程执行计划（Remote）
- 3：分布式执行计划（Distribute）

[gv$sql_audit视图字段说明](https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000356239)


## 查数据库最大连接数

在oceanbase 库 用sys 用户  show  proxyconfig 或者在ocp上能看到 ,

obproxy 最大连接数一般是 8000  - 16000 。

## 快速生成DDL

快速生成CREATE-TABLE-DDL语句,查DBA视图，如果要查USER视图全局替换DBA_为USER_即可。

```sql
SELECT 'CREATE TABLE ' || T.TABLE_NAME || ' ('
FROM DBA_TABLES T
WHERE T.TABLE_NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD'
UNION ALL
SELECT C.COLUMN_NAME || ' ' || C.DATA_TYPE || ' ' || C.NULLABLE || ' DEFAULT ' || C.DATA_DEFAULT || ','
FROM (
         SELECT TC.TABLE_NAME,
                TC.COLUMN_NAME,
                CASE
                    WHEN TC.DATA_TYPE = 'DATE' THEN TC.DATA_TYPE
                    WHEN TC.DATA_TYPE = 'NUMBER' THEN (CASE
                                                           WHEN TC.DATA_PRECISION IS NOT NULL
                                                               THEN TC.DATA_TYPE || '(' || TC.DATA_PRECISION || ',' || TC.DATA_SCALE || ')'
                                                           ELSE TC.DATA_TYPE END)
                    WHEN TC.CHARACTER_SET_NAME IS NOT NULL THEN TC.DATA_TYPE || '(' || TC.CHAR_LENGTH || ')'
                    ELSE TC.DATA_TYPE || '(' || TC.DATA_LENGTH || ')' END DATA_TYPE,
                CASE TC.NULLABLE WHEN 'N' THEN ' NOT NULL ' END           NULLABLE,
                TC.DATA_DEFAULT
         FROM DBA_TAB_COLUMNS TC
         WHERE TC.TABLE_NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD'
         ORDER BY TC.TABLE_NAME, TC.COLUMN_ID) C
UNION ALL
SELECT 'CONSTRAINT ' || P.CONSTRAINT_NAME || ' PRIMARY KEY (' || P.COLUMN_NAME || ')'
FROM (
         SELECT CU.TABLE_NAME,
                CU.CONSTRAINT_NAME,
                LISTAGG(CU.COLUMN_NAME, ',') WITHIN GROUP (ORDER BY CU.TABLE_NAME,CU.COLUMN_NAME) AS COLUMN_NAME
         FROM DBA_CONS_COLUMNS CU INNER JOIN DBA_CONSTRAINTS AU
         ON CU.CONSTRAINT_NAME = AU.CONSTRAINT_NAME
         WHERE CU.OWNER='AMS' AND AU.CONSTRAINT_TYPE = 'P' AND CU.TABLE_NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD'
         GROUP BY CU.TABLE_NAME, CU.CONSTRAINT_NAME) P
UNION ALL
SELECT ')'
FROM DUAL
UNION ALL
SELECT 'PARTITION BY LIST (' || PK.COLUMN_NAME || ') '
FROM (
         SELECT PK.NAME, PK.COLUMN_NAME, PK.OBJECT_TYPE, PK.COLUMN_POSITION
         FROM DBA_PART_KEY_COLUMNS PK
         WHERE PK.NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD') PK
UNION ALL
SELECT CASE P.PARTITION_NAME
           WHEN 'SUBCOMPANY_0' THEN '(PARTITION ' || P.PARTITION_NAME || ' VALUES (' || P.HIGH_VALUE || '),'
           WHEN 'SUBCOMPANY_WU' THEN 'PARTITION ' || P.PARTITION_NAME || ' VALUES (' || P.HIGH_VALUE || '));'
           ELSE 'PARTITION ' || P.PARTITION_NAME || ' VALUES (' || P.HIGH_VALUE || '),' END
FROM (SELECT PT.TABLE_NAME, PT.PARTITION_NAME, PT.HIGH_VALUE, PT.PARTITION_POSITION
      FROM DBA_TAB_PARTITIONS PT
      WHERE PT.TABLE_NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD'
      ORDER BY PT.PARTITION_POSITION) P
UNION ALL
SELECT 'COMMENT ON TABLE ' || T.TABLE_NAME || ' IS ''' || T.COMMENTS || ''';'
FROM DBA_TAB_COMMENTS T
WHERE T.TABLE_NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD'
UNION ALL
SELECT 'COMMENT ON COLUMN ' || CC.TABLE_NAME || '.' || CC.COLUMN_NAME || ' IS ''' || CC.COMMENTS || ''';'
FROM (SELECT T.TABLE_NAME, T.COLUMN_NAME, C.COMMENTS
      FROM DBA_TAB_COLUMNS T
               INNER JOIN DBA_COL_COMMENTS C
                          ON T.TABLE_NAME = C.TABLE_NAME AND T.COLUMN_NAME = C.COLUMN_NAME
      WHERE T.TABLE_NAME = 'AMS_ACCOUNTIMPDATA_DETAIL_TD'
      ORDER BY T.COLUMN_ID) CC
```

## OB常见兼容问题

## OB 临时表问题与应对方案

一、存在的问题

1、临时表在存过中响应时间缓慢。

问题原因：

（1）临时表是会话级别的。当会话结束时，或者定义为 ON COMMIT DELETE ROWS 的临时表在执行 COMMIT 时，会对临时表进行数据清理操作。数据清理默认一次最多删除 1000 行，使用循环删除的方式直至临时表中所有数据被清空。如果临时表中的数据量很大，则清理临时表的耗时会比较久。
（2）由于不同session对临时表访问无法共享计划，如果全局临时表存在在PL对象中，每个session都需要编译一次这个PL可能导致性能和稳定性问题。
（3）临时表在程序中使用随着临时表的数据量增多，会导致响应时间变慢。

2、临时表引起登录时间变慢甚至夯住问题。

问题现象:

集群中存在业务租大量使用临时表（CGTT）的情况，业务租户登录很慢，甚至无法登录，OBProxy 日志中 COM_LOGIN 耗时很久或报 -4152 错误。

问题原因：

临时表中的数据只对本 session 可见，其生命周期随着 session 断开而终止。在 OceanBase 数据库 V3.2.4 BP5（oceanbase-3.2.4.5-105000012023081513）之前，由于 session id 可能存在复用的情况，会在登陆时对当前 session id 的数据进行检查，如果存在则需要额外进行一次清理。当同样的 session id 曾经执行过大量的临时表时，清理动作耗时较久，会导致登录缓慢甚至无法登录的问题。

2、临时表数据无法清理。

问题原因：

全局临时表在PL业务块中穿插自治事务时，自治事务内部的提交改变了一个控制临时是否提交清理的标记，最终导致临时表数据无法清理。

3、分区临时表 频繁 truncate 分区表不回收。

现象：DSG同步巨大延迟，DML/DDL执行变慢

问题原因：

truncate 分区动作不会触发schema回收，频繁执行后，刷新schema会越来越慢。

schema历史保留七天，每次ddl都会产生一个或多个schema变更，ddl较多时schema也会变多，OB处理ddl数据时构造指定版本schema时需要回溯的数据也会变多，耗时就会变长，schema回收可以减少历史schema的数量，schema保留时间时间会影响我们回溯schema过程的效率，但也不能太短，如果链路延迟时间超过了schema保留时间，就有可能会导致链路中断且无法恢复。

目前大部分环境schema保留时间为7天。


二、应对方案：

1、临时表数据量比较大引起的变慢

应对方案：建议如果临时表据量比较大，建议换成实体表，通过 truncate 来进行数据清理。

已知影响版本:   V2.2.x、V3.1.x、V3.2.x、V4.0.x、V4.1.x、V4.2.x

2、临时表引起登录时间变慢甚至夯住问题

应对方案：尽量避免使用临时表，建议改造成普通表后新增唯一标识字段来实现session之间隔离的功能。

解决方法: 升级到 OceanBase 数据库 V3.2.4 BP5（oceanbase-3.2.4.5-105000012023081513）。

在 OceanBase 数据库 V3.2.4 BP5（oceanbase-3.2.4.5-105000012023081513）之前，尽量避免使用临时表，使用普通表来代替。

影响版本: OceanBase 数据库 V3.2.4 BP5（oceanbase-3.2.4.5-105000012023081513）之前的版本 


3、临时表数据无法清理。

应对方案：在自治事务后面，执行一次临时表DML动作（可以是无意义动作）。

发现问题版本  V3.2.3.3 bp8

4、分区临时表频繁truncate 分区问题

应对方案：需要定时对该表进行整体truncate的操作以触发schema回收，可以减少历史schema的数量。

发现问题版本  V3.2.3.3 bp10


## OB 超时参数

```sql
--查超时 相关参数
show global variables Like'%timeout%';

--查 连接 相关参数
show global variables Like'%connections%';

--查 undo 参数
show global variables Like'%undo%';
```

```
VARIABLE_NAME       VALUE
connect_timeout     10
interactive_timeout 28800
net_read_timeout    30
net_write_timeout   60
ob_pl_block_timeout 3216672000000000
ob_query_timeout    21600000000
ob_trx_idle_timeout 21600000000
ob_trx_lock_timeout 6000000
ob_trx_timeout      21600000000
wait_timeout        86400


-- 历史版本保留1小时
undo_retention      3600 
```

## OB 查默认事务

show variables '%transaction%';

```
VARIABLE_NAME       VALUE
transaction_isolation    READ-COMMITTED
transaction_read_only    OFF
```


## OB PL

一次性获取函数索引

```SQL
-- 将索引表达式转成字符串
CREATE OR REPLACE FUNCTION  INDEX_COLUMN_EXPRESSION(
    IN_TABLE_NAME VARCHAR2,
    IN_INDEX_NAME VARCHAR2,
    IN_COLUMN_POSITION VARCHAR2
)
RETURN VARCHAR AS
    TEXT_STR VARCHAR2(32767);
    SQL_STR VARCHAR2(2000);
BEGIN
    SQL_STR = 'SELECT COLUMN_EXPRESSION
        FROM USER_IND_EXPRESSIONS T WHERE T.TABLE_NAME ='''|| IN_TABLE_NAME||'''
        AND T.INDEX_NAME = '''|| IN_INDEX_NAME||'''
        AND T.COLUMN_POSITION = '''|| IN_COLUMN_POSITION||''' ';
    EXECUTE IMMEDIATE SQL_STR INTO TEXT_STR;
    TEXT_STR := SUBSTR(TEXT_STR, 1,32767);
    RETURN TEXT_STR;
END;
/
```

带上OWNER账户查询

```SQL
-- 将索引表达式转成字符串
CREATE OR REPLACE FUNCTION  INDEX_COLUMN_EXPRESSION(
    IN_TABLE_OWNER VARCHAR2,
    IN_TABLE_NAME VARCHAR2,
    IN_INDEX_NAME VARCHAR2,
    IN_COLUMN_POSITION VARCHAR2
)
RETURN VARCHAR AS
    TEXT_STR VARCHAR2(32767);
    SQL_STR VARCHAR2(2000);
BEGIN
    SQL_STR = 'SELECT COLUMN_EXPRESSION
        FROM USER_IND_EXPRESSIONS T WHERE T.TABLE_NAME ='''|| IN_TABLE_NAME||'''
        AND T.INDEX_OWNER = '''|| IN_TABLE_OWNER||'''
        AND T.INDEX_NAME = '''|| IN_INDEX_NAME||'''
        AND T.COLUMN_POSITION = '''|| IN_COLUMN_POSITION||''' ';
    EXECUTE IMMEDIATE SQL_STR INTO TEXT_STR;
    TEXT_STR := SUBSTR(TEXT_STR, 1,32767);
    RETURN TEXT_STR;
END;
/
```

LONG_TO_CHAR 视图数据类型转换

```SQL
-- ORACLE LONG 转 CHAR 函数
CREATE OR REPLACE FUNCTION LONG_TO_CHAR(
    IN_COLUMN_NAME VARCHAR2,
    IN_OWNER VARCHAR2,
    IN_INDEX_NAME VARCHAR2,
    IN_CONDITION VARCHAR2
)
    RETURN VARCHAR AS
    TEXT_STR VARCHAR2(32767);
    SQL_STR  VARCHAR2(2000);
BEGIN
    SQL_STR := 'SELECT ' || IN_COLUMN_NAME || ' FROM ' || IN_OWNER || '.' || IN_INDEX_NAME ||
               ' T WHERE 1=1 ' || IN_CONDITION;
    EXECUTE IMMEDIATE SQL_STR INTO TEXT_STR;
    TEXT_STR := SUBSTR(TEXT_STR, 1, 32767);
    RETURN TEXT_STR;
END;
/
```