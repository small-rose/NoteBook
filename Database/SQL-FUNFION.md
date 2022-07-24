---
layout: default
title: SQL-FUNCTION
nav_order: 100
parent: Database
---

# SQL-FUNCTION
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

--- 

SQL使用类比
-------------------------

## 带上序号的分页
### MYSQL

第一页
```sql
select * from (select d.*,@rownum := @rownum + 1 as num from (
    select * from mm_subcompany_tc
    ) d, (SELECT @rownum := 0) myTempTable LIMIT 10 ) TT where num >= 1 ;
```

第二页
```sql
select * from (select d.*,@rownum := @rownum + 1 as num from (
    select * from mm_subcompany_tc
    ) d, (SELECT @rownum := 0) myTempTable LIMIT 20 ) TT where num >= 11 ;
``` 


### ORACLE 

KingBase 的Oracle模式。

第一页
 ```sql
select * from (select d.*,rownum as num from (
      select * from mm_subcompany_tc
    ) d where rownum <= 10 )where num >= 1 ;
```

第二页
```sql
select * from (select d.*,rownum as num from (
      select * from mm_subcompany_tc
    ) d where rownum <= 20 )where num >= 11 ;
```
### DB2
 ```sql
SELECT *FROM (
    SELECT row_number () OVER (ORDER BY CC_BRAND.BRAND_CODE) AS rown,
        MS.* FROM mm_subcompany_tc MS ) AS A
WHERE A.ROWN >= 1 AND A.ROWN <= 10; 
```

第二页
```sql
SELECT *FROM (
    SELECT row_number () OVER (ORDER BY CC_BRAND.BRAND_CODE) AS rown,
        MS.* FROM mm_subcompany_tc MS ) AS A
WHERE A.ROWN >= 11 AND A.ROWN <= 20; 
```
  
## 函数使用

### 1.判断列字段是否包含关键字.

判断列的值里是否包含 BP2101 关键字。

MYSQL:

```sql
select locate('BP2101',powerrole) > 0 from t_test;
```
Oracle
```sql
select instr(powerrole, 'BP2101') > 0 from t_test;
```
kingBase:
```sql
select strpos(powerrole,'BP2101')>0  from t_test ;
```