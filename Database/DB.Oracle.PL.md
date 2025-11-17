---
layout: default
title: ORACLE PL
nav_order: 22
parent: Database
---

# ORACLE  Procedure & Package & PL/SQL
{: .no_toc }

## TABLE of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## oracle PL 专栏

Oracle 在线学习：https://livesql.oracle.com

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

### 取常用日期

```sql
--本月初-日期
SELECT TRUNC(SYSDATE,'MM')  AS FIRST_DAY_OF_MONTH FROM DUAL;
--本月末-日期
SELECT TRUNC(LAST_DAY(SYSDATE)) AS LAST_DAY_OF_MONTH FROM DUAL;
--上月初-日期
SELECT TRUNC(TRUNC(SYSDATE,'MM')-1, 'MM')  AS FIRST_DAY_OF_LAST_MONTH FROM DUAL;
--上月末-日期
SELECT LAST_DAY(TRUNC(SYSDATE,'MM')- 1) AS LAST_DAY_OF_LAST_MONTH FROM DUAL;
--上年初-日期
SELECT TRUNC(ADD_MONTHS(SYSDATE,-12),'YYYY') AS FIRST_DAY_OF_LAST_YEAR FROM DUAL;
--上年末-日期
SELECT LAST_DAY(ADD_MONTHS(TRUNC(SYSDATE,'YEAR'), -1)) AS LAST_DAY_OF_LAST_YEAR FROM DUAL;

--前年初-日期
SELECT TRUNC(ADD_MONTHS(TRUNC(SYSDATE, 'YEAR'), -24), 'YEAR') AS FIRST_DAY_OF_TWO_YEARS_AGO FROM DUAL;
--前年末-日期
SELECT LAST_DAY(ADD_MONTHS(TRUNC(SYSDATE, 'YEAR'), -13)) AS LAST_DAY_OF_TWO_YEARS_AGO FROM DUAL;
```


### oralce 获取异常的栈信息

```sql
DECLARE
  excep   EXCEPTION;
  pragma exception_init(excep, -20001);
  v_count number(16,2);
BEGIN
  -- 你的代码逻辑，可能会引发异常
  select 1/0 into v_count from dual ;
  raise_application_error(-20001,'测试抛出异常');  
EXCEPTION
  WHEN excep THEN
    DBMS_OUTPUT.PUT_LINE('Exception Message: ' || SQLERRM);
    DBMS_OUTPUT.PUT_LINE('Exception Stack:');
    DBMS_OUTPUT.PUT_LINE(DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
END;
/
```


## 存储过程案例



### FOR UPDATE NOWAIT

```sql
-- 缴费计划补分期
CREATE OR REPLACE PROCEDURE DO_ex_MODIFY_BYPLAN(IN_SUBCOMPANY IN AMS_MIRROR_DETAIL_TD.SUBCOMPANY%TYPE,
                                                start_key IN number,
                                                end_key IN number) IS
    cursor v_plan_modify(in_subcompany varchar2,start_key number,end_key number) is
        select id, subcompany, policyno, endorseno, currencycode, businessattr
        from ams_plan_modify_td t 
        where t.status = '0'
          and t.subcompany = in_subcompany
          and t.id between start_key and end_key 
          for update nowait;  
BEGIN
    v_index := 0; v_count := 0; v_sumcount := 0;
    for P_REC in v_plan_modify(in_subcompany, start_key, end_key)
        loop
            BEGIN
                -- DO SOME ELSE
                
                UPDATE AMS_PLAN_MODIFY_TD T
                SET T.STATUS           = '2',
                    T.LASTOPDATE       = SYSDATE,
                    T.HIBERNATEVERSION = T.HIBERNATEVERSION + 1
                WHERE current of v_plan_modify;
            EXCEPTION
                WHEN OTHERS THEN ROLLBACK;
            END;
        END LOOP;
    commit;
EXCEPTION
    WHEN OTHERS THEN ROLLBACK;
    AMS_ERRORLOG_PKG.LOG_ERROR();
END;
``` 


### 缓存Map数据

> 数据量比较小的可以这样缓存，减少反复查询。

（1）基于 `TYPE` 使用  `TABLE OF ... INDEX BY VARCHAR2 ` 模式的Map, 索引 key 和 值 value 绑定。

使用场景：适合类似Java的Map缓存模式, 如果险种/部门类的值需要反复查询时, 缓存可以减少查询次数。



案例一 

```sql
-- 索引key 映射 value
DECLARE
  TYPE NAME_MAP IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(10);
  NAMECACHE NAME_MAP;
  CODE VARCHAR2(10);
  NAME VARCHAR2(100);
BEGIN
  -- 查询表并缓存到关联数组中
  FOR REC IN (SELECT  T.UNITCODE,  T.UNITNAME  FROM  T_UNIT_TC T ) LOOP
    NAMECACHE(REC.UNITCODE) := REC.UNITNAME;
  END LOOP;

  -- 根据 CODE 直接获取缓存中的 NAME
  CODE := '001';
  NAME := NAMECACHE(CODE);
  DBMS_OUTPUT.PUT_LINE('名称: ' || NAME);
END;
```

案例二

```sql
-- 索引key 映射对象
DECLARE
    TYPE UNIT_MAP IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(10);
    G_UNITNAME_CACHE UNIT_MAP;
    OUT_UNIT         T_UNIT_TC%ROWTYPE;
    OUT_NAME         VARCHAR2(100);

    -- 不需要每次都查询
    PROCEDURE GET_UNIT_NAME_CACHE(V_CODE VARCHAR2, OUT_UNIT VARCHAR2) IS
        V_TMP_CODE varchar2(20);
    BEGIN
        V_TMP_CODE := nvl(V_CODE, '空')
        if G_UNITNAME_CACHE.EXISTS(V_TMP_CODE) then
            OUT_UNIT := G_UNITNAME_CACHE(V_TMP_CODE);
        ELSE
            BEGIN
                SELECT *
                INTO G_UNITNAME_CACHE(V_TMP_CODE)
                FROM T_UNIT_TC T
                WHERE T.UNITCODE = V_TMP_CODE
                  AND ROWNUM <= 1;
            EXCEPTION
                WHEN OTHERS THEN
                    G_UNITNAME_CACHE(V_TMP_CODE) := V_TMP_CODE
                --RAISE_APPLICATION_ERROR('-20999', V_CODE||'找不到部门名称');
            end;
            OUT_UNIT := G_UNITNAME_CACHE(V_TMP_CODE);
        END IF;
    END;
BEGIN
    -- 查询表并缓存到关联数组中
    FOR REC IN (SELECT T.UNITCODE, T.USERNAME FROM T_USER_TC T )
        LOOP
            -- 根据 unitcode 获取 unit 对象
            GET_UNIT_NAME_CACHE(REC.UNITCODE, OUT_UNIT);
            DBMS_OUTPUT.PUT_LINE('unit名称: ' || OUT_UNIT.UNITNAME);
        END LOOP;
END;
```


（2）基于 `TYPE` 使用  `RECORD` 缓存行记录模式的Map, 真的是一个Map 的实现，使用自定义的PUT 和 GET 填充集合。 

使用场景：适合类似Java的Map缓存模式, 如果险种/部门类的值需要反复查询时, 缓存可以减少查询次数。

```sql
DECLARE
  -- 定义一个类型，用于表示键和值
  TYPE pair IS RECORD (
    key VARCHAR2(100),
    value VARCHAR2(100)
  );

  -- 声明一个基于上述类型的嵌套表
  TYPE key_value_pairs IS TABLE OF pair;

  -- 实例化嵌套表
  my_map key_value_pairs := key_value_pairs();

  -- 添加键值对
  PROCEDURE put(key VARCHAR2, value VARCHAR2) IS
  BEGIN
    my_map.EXTEND;
    my_map(my_map.LAST).key := key;
    my_map(my_map.LAST).value := value;
  END;

  -- 获取键对应的值, 这种需要循环查找
  FUNCTION get(key VARCHAR2) RETURN VARCHAR2 IS
    value VARCHAR2(100);
  BEGIN
    FOR i IN 1..my_map.COUNT LOOP
      IF my_map(i).key = key THEN
        value := my_map(i).value;
        EXIT;
      END IF;
    END LOOP;
    RETURN value;
  END;
BEGIN
  -- 添加数据到映射
  put('name', 'John Doe');
  put('age', '30');

  -- 获取映射中的数据
  DBMS_OUTPUT.PUT_LINE('Name: ' || get('name'));
  DBMS_OUTPUT.PUT_LINE('Age: ' || get('age'));
END;
```


### 缓存 List 数据

（1）基于 `TYPE` 使用  `TABLE OF ... INDEX BY PLS_INTEGER ` 带下标的 List 集合, 索引 index 和 值 对象 绑定，使用 `BULK COLLECT`填充集合。

使用场景：适合使用下标, 需要判断集合大小的场景, 一般缓存数据时需要限制大小。

```sql
DECLARE
    TYPE DAILYAUDITNO_TYPE IS TABLE OF AMS_DAILYNUM_TD%ROWTYPE INDEX BY PLS_INTEGER;
    V_DAILYAUDITNO_LIST DAILYAUDITNO_TYPE;
BEGIN
    SELECT * BULK COLLECT
    INTO V_DAILYAUDITNO_LIST
    FROM AMS_DAILYNUM_TD T
    WHERE ROWNUM <= 10;

    dbms_output.put_line('count ' || V_DAILYAUDITNO_LIST.COUNT);
    FOR i IN 1..V_DAILYAUDITNO_LIST.COUNT
        LOOP
            dbms_output.put_line('i ' || i || ', element.subcompany ' || V_DAILYAUDITNO_LIST(i).subcompany);
            dbms_output.put_line('i ' || i || ', element.dailyauditno ' || V_DAILYAUDITNO_LIST(i).dailyauditno);
        END LOOP;
END;
```


### 驼峰转换函数


```sql
-- 版本一
CREATE OR REPLACE FUNCTION FN_CAMELCASE(P_FIELD_NAME IN VARCHAR2) RETURN VARCHAR2 IS
    V_CAMELCASE VARCHAR2(32767);
    V_NEXT_UPPERCASE BOOLEAN := FALSE;
BEGIN
    V_CAMELCASE := LOWER(P_FIELD_NAME);

    FOR I IN 1..LENGTH(V_CAMELCASE) LOOP
        IF SUBSTR(V_CAMELCASE, I, 1) = '_' THEN
            V_NEXT_UPPERCASE := TRUE;
        ELSIF V_NEXT_UPPERCASE THEN
            V_CAMELCASE := SUBSTR(V_CAMELCASE, 1, I - 2) || INITCAP(SUBSTR(V_CAMELCASE, I, 1)) || SUBSTR(V_CAMELCASE, I + 1);
            V_NEXT_UPPERCASE := FALSE;
        END IF;
    END LOOP;

    RETURN V_CAMELCASE;
END;
/
```

```sql
-- 版本2 AI 生成
CREATE OR REPLACE FUNCTION FN_CAMELCASE_SIMPLE(P_FIELD_NAME IN VARCHAR2) RETURN VARCHAR2 IS
BEGIN
    -- 1. 将下划线替换为空格，使每个部分成为独立的“单词”
    -- 2. 使用INITCAP将每个单词的首字母大写
    -- 3. 使用REPLACE移除所有空格，将单词连接起来
    -- 4. 使用LOWER和SUBSTR确保第一个字符为小写（实现小驼峰）
    RETURN LOWER(SUBSTR(REPLACE(INITCAP(REPLACE(P_FIELD_NAME, '_', ' ')), ' ', ''), 1, 1)) ||
           SUBSTR(REPLACE(INITCAP(REPLACE(P_FIELD_NAME, '_', ' ')), ' ', ''), 2);
END;
```

demo

```sql
SELECT FN_CAMELCASE('_hello_word_hi_camel_case') FROM DUAL;
SELECT FN_CAMELCASE_SIMPLE('_hello_word_hi_camel_case') FROM DUAL;
```


### 生成 JavaBean

```sql
SELECT
    'private ' ||
    (CASE
        WHEN DATA_TYPE = 'VARCHAR2' THEN 'String'
        WHEN DATA_TYPE = 'CHAR' THEN 'String'
        WHEN DATA_TYPE = 'NUMBER' AND DATA_SCALE > 0 THEN 'Double'
        WHEN DATA_TYPE = 'NUMBER' THEN 'int'
        WHEN DATA_TYPE = 'DATE' THEN 'Date'
        WHEN DATA_TYPE = 'TIMESTAMP' THEN 'Date'
        -- 可以根据需要添加其他数据类型的映射
        ELSE 'Object' -- 对于未明确映射的类型，使用Object
    END) ||
    ' ' ||
    LOWER(COLUMN_NAME) ||
    ';' ||
    (CASE WHEN COMMENTS IS NOT NULL THEN ' // ' || COMMENTS ELSE '' END) AS java_field
FROM
    (SELECT
        s.COLUMN_NAME,
        s.DATA_TYPE,
        s.DATA_SCALE, -- 用于判断NUMBER类型是整数还是小数
        t.COMMENTS
    FROM
        USER_TAB_COLUMNS s
    INNER JOIN
        USER_COL_COMMENTS t
    ON
        s.TABLE_NAME = t.TABLE_NAME
        AND s.COLUMN_NAME = t.COLUMN_NAME
    WHERE
        s.TABLE_NAME = UPPER('T_FILE_TABLE_MAPPING_TC') -- 请将 your_table_name 替换为您的实际表名
    ORDER BY
        s.COLUMN_ID -- 按表中列的顺序输出
    );
```




### 归档案例

```sql
CREATE OR REPLACE PACKAGE AMS_DATA_ARCHIVE_PKG AS
    -- AMS_APPLICATIONS 持续归档
    PROCEDURE DO_ARCHIVE_APPLICATIONS(V_SUBCOMPANY IN VARCHAR2);
END AMS_DATA_ARCHIVE_PKG;
/
CREATE OR REPLACE PACKAGE BODY AMS_DATA_ARCHIVE_PKG IS
    PROCEDURE DO_ARCHIVE_APPLICATIONS(V_SUBCOMPANY IN VARCHAR2) IS
        V_SUB_OPDATE    DATE;           --对应分公司的可归档最小日期
        V_OPDATE        DATE;           --可归档数据的归档最小日期
        V_SQL           VARCHAR2(1000);
        V_SUB_SQL       VARCHAR2(1000);
        V_COUNT_SQL     VARCHAR2(2000); --统计分公司最小归档日的数据量
        V_TMP_SQL       VARCHAR2(2000);
        V_ARCHIVE_COUNT NUMBER ;
        V_BATCH_SIZE    NUMBER    := 10000;
        V_SIZE          NUMBER ;        --单日总次数
        V_DO_COUNT      NUMBER ;        --执行次数累计
        V_GUID          VARCHAR2(200);  --并发区分
        V_COND          VARCHAR2(100);
        V_START_TIME    TIMESTAMP := SYSTIMESTAMP;--开始时间
        V_CURRENT_TIME  TIMESTAMP;      --当前时间
        V_TIME_EXCEEDED BOOLEAN   := FALSE;

    BEGIN
        --取三年前可归档区间的分公司最小日期
        V_SUB_SQL := 'SELECT MIN(OPDATE) FROM AMS_APPLICATIONS WHERE SUBCOMPANY = :SUBCOMPANY AND STATUS IN (''C'', ''G'') AND OPDATE <= ADD_MONTHS(SYSDATE, -36)';
        --取可归档区间最小日期
        V_SQL := 'SELECT MIN(OPDATE) FROM AMS_APPLICATIONS WHERE STATUS IN (''C'', ''G'') AND OPDATE <= ADD_MONTHS(SYSDATE, -36) ';
        --获取最小OPDATE
        --DBMS_OUTPUT.PUT_LINE('V_SQL : ' || V_SQL );
        EXECUTE IMMEDIATE V_SUB_SQL INTO V_SUB_OPDATE USING V_SUBCOMPANY;
        EXECUTE IMMEDIATE V_SQL INTO V_OPDATE;
        IF V_SUB_OPDATE IS NULL OR V_OPDATE IS NULL THEN
            RETURN;
        END IF;
        IF V_SUB_OPDATE > V_OPDATE THEN
            --防止一直归一家分公司，强制最早时间优先归档
            RETURN;
        END IF;

        --归档基本条件［分公司 SUBCOMPANY，数据状态 STATUS，日期 OPDATE ］
        V_COND := ' WHERE SUBCOMPANY = :SUBCOMPANY AND OPDATE = :OPDATE AND STATUS IN (''C'',''G'') ';
        V_COUNT_SQL := ' SELECT COUNT(1) FROM AMS_APPLICATIONS ' || V_COND;
        EXECUTE IMMEDIATE V_COUNT_SQL INTO V_ARCHIVE_COUNT USING V_SUBCOMPANY, V_SUB_OPDATE;
        --获取最小OPDATE +分公司+数据状态 的数据量
        IF V_ARCHIVE_COUNT = 0 THEN
            RETURN;
        END IF;

        --计算本次归档次数,分页归档
        V_SIZE := CEIL(V_ARCHIVE_COUNT / V_BATCH_SIZE);
        V_DO_COUNT := 0;
        WHILE (V_DO_COUNT < V_SIZE AND V_TIME_EXCEEDED = FALSE)
            LOOP
                --检查执行时间
                V_CURRENT_TIME := SYSTIMESTAMP;
                IF EXTRACT(MINUTE FROM (V_CURRENT_TIME - V_START_TIME)) +
                   EXTRACT(HOUR FROM (V_CURRENT_TIME - V_START_TIME))*60 > 30 THEN
                    --检查是否超过 30分钟
                    V_TIME_EXCEEDED := TRUE; -- 设置超时标志
                    EXIT; --退出循环
                END IF;

                V_GUID := V_SUBCOMPANY || SYS_GUID() || TO_CHAR(V_SUB_OPDATE, 'YYYY-MM-DD');
                -- APPLYNO, SUBCOMPANY 是主键
                V_TMP_SQL := 'INSERT INTO AMS_APPLICATIONS_IDX ( APPLYNO, SUBCOMPANY, GUID ) '
                    || ' SELECT APPLYNO, SUBCOMPANY,' || V_GUID || 'FROM AMS_APPLICATIONS '
                    || V_COND 
                    || ' AND ROWNUM <= ' || V_BATCH_SIZE ||' ORDER BY APPLYNO  ';

                --插入AMS_APPLICATIONS_IDX 索引表
                --DBMS_OUTPUT.PUT_LINE('V_TMP_SQL : ' || V_TMP_SQL );
                EXECUTE IMMEDIATE V_TMP_SQL USING V_SUBCOMPANY, V_SUB_OPDATE;

                V_TMP_SQL := 'INSERT INTO AMS_APPLICATIONS_BAK SELECT * FROM AMS_APPLICATIONS ' 
                    || V_COND
                    || ' AND ( APPLYNO, SUBCOMPANY ) IN (SELECT APPLYNO, SUBCOMPANY FROM AMS_APPLICATIONS_IDX WHERE GUID = :GUID )';
                --插入BAK 归档表
                EXECUTE IMMEDIATE V_TMP_SQL USING V_SUBCOMPANY, V_SUB_OPDATE, V_GUID;

                
                V_TMP_SQL := 'DELETE FROM AMS_APPLICATIONS WHERE ( APPLYNO, SUBCOMPANY ) IN (SELECT APPLYNO, SUBCOMPANY FROM AMS_APPLICATIONS_IDX WHERE GUID = :GUID )' ;
                --从主表中删除数据
                EXECUTE IMMEDIATE V_TMP_SQL USING V_GUID;

                --DBMS_OUTPUT.PUT_LINE('V_TMP_SQL : ' || V_TMP_SQL);
                --清空本次执行临时表
                V_TMP_SQL := 'DELETE FROM AMS_APPLICATIONS_IDX WHERE GUID = :GUID ';
                -- DBMS_OUTPUT.PUT_LINE('V_TMP_SQL :' || V_TMP_SQL );
                EXECUTE IMMEDIATE V_TMP_SQL USING V_GUID;
                COMMIT;
                V_DO_COUNT := V_DO_COUNT + 1;
            END LOOP;
    EXCEPTION
        WHEN OTHERS THEN
            ROLLBACK;
            AMS_ERRORLOG_PKG.LOG_ERROR(PROCNAME_IN => 'AMS_DATA_ARCHIVE_PKG',
                                       KEYWORD1_IN => 'DO_ARCHIVE_APPLICATIONS',
                                       KEYWORD3_IN => TO_CHAR(V_OPDATE, 'YYYY-MM-DD'),
                                       INFO_IN => 'V_GUID =' || V_GUID || SUBSTR(SQLERRM, 1, 2000));

    
    END DO_ARCHIVE_APPLICATIONS;

END AMS_DATA_ARCHIVE_PKG;
```


### 统计PL耗时

```sql
DECLARE
    V_START_TIME TIMESTAMP;
    V_END_TIME TIMESTAMP;
    V_ELAPSED_MS NUMBER;
BEGIN
    -- 第一次执行（冷启动）
    V_START_TIME := SYSTIMESTAMP;
    DBMS_OUTPUT.PUT_LINE('开始性能测试');
    DBMS_OUTPUT.PUT_LINE('========================================');

    -- DO YOUR BUSS
    AMS_PM_TRANSFER_PKG.GET_DEPARTMENTNAME(P_DEPT_CODE, V_RESULT);

    V_END_TIME := SYSTIMESTAMP;
    V_ELAPSED_MS := (CAST(V_END_TIME AS DATE) - CAST(V_START_TIME AS DATE)) * 24 * 60 * 60 * 1000;

    DBMS_OUTPUT.PUT_LINE('耗时: ' || ROUND(V_ELAPSED_MS, 3) || ' 毫秒');
    DBMS_OUTPUT.PUT_LINE('========================================');
END;
```