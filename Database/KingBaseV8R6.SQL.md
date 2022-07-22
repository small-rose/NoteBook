---
layout: default
title: KingBase V8R6
nav_order: 1
parent: Database
---

# KingBase V8R6 SQL
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

KingBase 查表和注释 
-----------------


```sql
SELECT col.COLUMN_NAME, col.data_type, col.UDT_NAME, d.DESCRIPTION, col.IS_NULLABLE,s.CONTYPE,col.table_schema,col.ordinal_position
FROM information_schema.COLUMNS col
         JOIN SYS_CLASS c ON c.RELNAME = col.TABLE_NAME
         LEFT JOIN SYS_DESCRIPTION d ON d.OBJOID = c.OID AND d.OBJSUBID = col.ORDINAL_POSITION
         LEFT JOIN sys_constraint s on c.OID = s.conrelid and col.ORDINAL_POSITION=ANY(conkey::int[])
WHERE
     col.table_catalog = 'bp_dev'
     and col.table_schema = 'public'
  AND  col.TABLE_NAME = 'mm_bad_debts';
```

KingBase 查表字段注释 
--------------------


```sql
SELECT col.COLUMN_NAME, col.data_type, col.UDT_NAME, d.DESCRIPTION, col.IS_NULLABLE,s.CONTYPE,col.table_schema,col.ordinal_position
FROM information_schema.COLUMNS col
         JOIN SYS_CLASS c ON c.RELNAME = col.TABLE_NAME
         LEFT JOIN SYS_DESCRIPTION d ON d.OBJOID = c.OID AND d.OBJSUBID = col.ORDINAL_POSITION
         LEFT JOIN sys_constraint s on c.OID = s.conrelid and col.ORDINAL_POSITION=ANY(conkey::int[])
WHERE
     col.table_catalog = 'bp_dev'
     and col.table_schema = 'public'
  AND  col.TABLE_NAME = 'mm_bad_debts';
```
