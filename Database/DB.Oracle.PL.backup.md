---
layout: default
title: ORACLE PL BACK UP
nav_order: 23
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

Oracle 在线数据库：[https://freesql.com/worksheet?tutorial=creating-tables-databases-for-developers-SQru0F#module4](https://freesql.com/worksheet?tutorial=creating-tables-databases-for-developers-SQru0F#module4)

## PL/SQL 相关

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

### 通用的归档案例

#### 错误日志表:

```sql
-- 错误日志记录
create table AMS_ERROR_LOG
(
    OWNER    VARCHAR2(30)                                          not null,
    INFO     VARCHAR2(4000),
    SQLCODE  NUMBER,
    SQLERRM  VARCHAR2(4000),
    TRACE    VARCHAR2(4000),
    LOGDATE  DATE                                                  not null,
    PROCNAME VARCHAR2(100)                                         not null,
    KEYWORD1 VARCHAR2(300),
    KEYWORD2 VARCHAR2(300),
    KEYWORD3 VARCHAR2(300),
    KEYWORD4 VARCHAR2(300),
    createtime date default sysdate,
    lastopdate date default sysdate,
    hibernateversion VARCHAR2(300),
    ERRORID  NUMBER default "BPJYDATA"."SEQ_ERRORPOLICY"."NEXTVAL" not null,
    constraint PK_ERRORID primary key (ERRORID)
);
comment on table AMS_ERROR_LOG is '错误日志表';
comment on column AMS_ERROR_LOG.OWNER is '所有者';
comment on column AMS_ERROR_LOG.INFO is 'some error information';
comment on column AMS_ERROR_LOG.SQLCODE is 'sql error code';
comment on column AMS_ERROR_LOG.SQLERRM is 'sql error message';
comment on column AMS_ERROR_LOG.TRACE is 'some error trace';
comment on column AMS_ERROR_LOG.LOGDATE is '错误日志写表时间';
comment on column AMS_ERROR_LOG.PROCNAME is '过程名';
comment on column AMS_ERROR_LOG.KEYWORD1 is '关键字1,记录业务表的主键，放表定位错误';
comment on column AMS_ERROR_LOG.KEYWORD2 is '关键字2,同上';
comment on column AMS_ERROR_LOG.KEYWORD3 is '关键字3,同上';
comment on column AMS_ERROR_LOG.KEYWORD4 is '关键字4,同上';
comment on column AMS_ERROR_LOG.createtime is '创建时间';
comment on column AMS_ERROR_LOG.lastopdate is '更新时间';
comment on column AMS_ERROR_LOG.hibernateversion is '版本号';
comment on column AMS_ERROR_LOG.ERRORID is '主键id';

create index IDX_ERRORLOG1 on AMS_ERROR_LOG (LOGDATE, PROCNAME);
```

####  错误日志存过包:
```sql
create or replace package ams_errorlog_pkg is
  /*
  * 自定义全局异常号
  */
  e_null_value constant binary_integer := -20001; /*空值*/
  null_value exception;
  pragma exception_init(null_value, -20001);
  e_invalid_value constant binary_integer := -20002; /*无效值*/
  invalid_value exception;
  pragma exception_init(invalid_value, -20002);
  e_invalid_rowcnt constant binary_integer := -20003; /*错误行数*/
  invalid_rowcnt exception;
  pragma exception_init(invalid_rowcnt, -20003);
  e_invalid_rule constant binary_integer := -20004; /*校验规则失败*/
  invalid_rule exception;
  pragma exception_init(invalid_rule, -20004);
  e_split_error constant binary_integer := -20005; /*拆分错误*/
  split_error exception;
  pragma exception_init(split_error, -20005);
  e_update_error constant binary_integer := -20006; /*更新失败*/
  update_error exception;
  pragma exception_init(update_error, -20006);
  e_system_error constant binary_integer := -20007; /*无法处理的错误*/
  system_error exception;
  pragma exception_init(system_error, -20007);
  e_no_data_found constant binary_integer := -20100; /*获取不到值,*/
  no_data_found exception;
  pragma exception_init(no_data_found, -20100);

  bulk_errors exception;
  pragma exception_init(bulk_errors, -24381); /*传输数据块错误*/
  /*
   * text="错误日志,dbms_utility.format_error_backtrace适用于oracle10g"
   */
  procedure log_error(procname_in in mm_error_log.procname%type,
                      keyword1_in in mm_error_log.keyword1%type default null,
                      keyword2_in in mm_error_log.keyword2%type default null,
                      keyword3_in in mm_error_log.keyword3%type default null,
                      keyword4_in in mm_error_log.keyword4%type default null,
                      info_in     in mm_error_log.info%type default null,
                      sqlcode_in  in mm_error_log.sqlcode%type default sqlcode,
                      sqlerrm_in  in mm_error_log.sqlerrm%type default sqlerrm,
                      trace_in    in mm_error_log.trace%type default dbms_utility.format_error_backtrace);

end ams_errorlog_pkg;
/

create or replace package body ams_errorlog_pkg is
  -- purpose : 记录错误日志
  procedure log_error(procname_in in mm_error_log.procname%type,
                      keyword1_in in mm_error_log.keyword1%type default null,
                      keyword2_in in mm_error_log.keyword2%type default null,
                      keyword3_in in mm_error_log.keyword3%type default null,
                      keyword4_in in mm_error_log.keyword4%type default null,
                      info_in     in mm_error_log.info%type default null,
                      sqlcode_in  in mm_error_log.sqlcode%type default sqlcode,
                      sqlerrm_in  in mm_error_log.sqlerrm%type default sqlerrm,
                      trace_in    in mm_error_log.trace%type default dbms_utility.format_error_backtrace) is
    pragma autonomous_transaction; /*自治事务*/
  begin
    insert into mm_error_log(owner, info, sqlcode, sqlerrm, trace, logdate, procname, keyword1, keyword2, keyword3, keyword4)
    values
      (user, info_in, sqlcode_in, sqlerrm_in, trace_in,
       sysdate, upper(procname_in),
       keyword1_in, keyword2_in, keyword3_in, keyword4_in, seq_errorpolicy.nextval);
    commit;
  exception
    when others then
      rollback;
  end;

end ams_errorlog_pkg;
/
```

####  设计归档表

```sql

drop table ams_backup_td;
create table ams_backup_td
(
    id               number                              not null,
    backtype         varchar2(2)                         not null,
    originaltable    varchar2(100)                       not null,
    idxtable         varchar2(40),
    idxcolumns       varchar2(40),
    backuptable      varchar2(100)                       not null,
    backupdesc       varchar2(500)             ,
    condition1       varchar2(4000)                      not null,
    bakcolumn        varchar2(30),
    subcompany       varchar2(400),
    ifvalid          varchar2(2)    default '1'          not null,
    status           varchar2(2)    default '1'          not null,
    errormsg         varchar2(4000) default 'OK'         not null,
    batchsize        number         default 10000,
    exestarttime     date           default sysdate      not null,
    exeendtime       date           default sysdate + 30 not null,
    exenexttime      varchar2(512)                       not null,
    createtime       date           default sysdate      not null,
    lasopdate        date           default sysdate      not null,
    modifydesc       varchar2(1000)      ,
    hibernateversion number         default 1            not null,
    constraint pk_backupconfig_td primary key (id)
);
comment on table ams_backup_td is '备份配置表';
comment on column ams_backup_td.id is '主键';
comment on column ams_backup_td.backtype is '备份类型 0-删除备份表数据; 1-单表归档,执行时默认按时间, 2-多表归档默认有表数据关联,按关联键的批次规范,3-按分公司归档';
comment on column ams_backup_td.originaltable is '原始表';
comment on column ams_backup_td.idxtable is '备份索引表';
comment on column ams_backup_td.idxcolumns is '备份索引列,一般是主键，多个以逗号分割';
comment on column ams_backup_td.backuptable is '备份表';
comment on column ams_backup_td.backupdesc is '备份说明';
comment on column ams_backup_td.condition1 is '检查最小可备份数据';
comment on column ams_backup_td.bakcolumn is '检查最小可备份数据使用的列,一般是备份维度,当按单表或分公司归档时，是时间字段createtime,opdate之类, 按批次归档时是关联批次字段batchno,matchid之类';
comment on column ams_backup_td.subcompany is '分公司集合多个以英文逗号连接如1010100,2010100';
comment on column ams_backup_td.ifvalid is '启用或停用,1-启用;2-停用';
comment on column ams_backup_td.status is '运行状态 0-停止,1-待执行,2-执行中,3-成功,4-失败';
comment on column ams_backup_td.errormsg is '失败的原因';
comment on column ams_backup_td.batchsize is '批次大小默认10000';
comment on column ams_backup_td.exestarttime is '可执行区间始';
comment on column ams_backup_td.exeendtime is '可执行区间终';
comment on column ams_backup_td.exenexttime is '执行频率';
comment on column ams_backup_td.createtime is '创建时间';
comment on column ams_backup_td.lasopdate is '最后执行时间';
comment on column ams_backup_td.modifydesc is '修改说明';
comment on column ams_backup_td.hibernateversion is '版本号';

```

####  备份 存储过程包

包头

```sql
create or replace package ams_backup_pkg is
    /**
      * @author
      * @date 2025.11.26
     */
    -- 备份表删除
    procedure do_backup_delete(v_back_up ams_backup_td%rowtype);
    -- 按单表备份
    procedure do_backup_sigle(v_back_up ams_backup_td%rowtype);
    -- 按关联表备份
    procedure do_backup_mutil(v_back_up ams_backup_td%rowtype);
    -- 按分公司分区备份
    procedure do_backup_subcompany(v_subcompany varchar2, v_back_up ams_backup_td%rowtype);
    -- 备份入口
    procedure do_backup;
end ams_backup_pkg;
```

包体

```sql
create or replace package body ams_backup_pkg is
    type back_up_type is table of ams_backup_td%rowtype index by pls_integer;
    type varchar2_list is table of varchar2(1000);

    function split_string(p_str in varchar2, p_delimiter in varchar2 default ',')
        return varchar2_list
        is
        v_result varchar2_list := varchar2_list();
    begin
        select regexp_substr(p_str, '[^' || p_delimiter || ']+', 1, level) bulk collect
        into v_result
        from dual
        connect by level <= regexp_count(p_str, '[^' || p_delimiter || ']+');
        return v_result;
    end split_string;
    -- 备份表删除
    procedure do_backup_delete(v_back_up ams_backup_td%rowtype) is
        v_del_sql varchar2(1000);
    begin
        v_del_sql := 'delete from ' || v_back_up.originaltable || v_back_up.condition1 || ' and rownum <= ' ||
                     v_back_up.batchsize;
        execute immediate v_del_sql;
        commit;
    end do_backup_delete;
    -- 按单表备份
    procedure do_backup_sigle(v_back_up ams_backup_td%rowtype) is
        v_opdate        date; --可归档数据的归档最小日期
        v_sub_sql       varchar2(1000);
        v_count_sql     varchar2(2000); --统计分公司最小归档日的数据量
        v_tmp_sql       varchar2(2000);
        v_archive_count number ;
        v_batch_size    number    := 10000;
        v_max_min       number    := 30; --最长执行时间限制
        v_size          number ; --单日总次数
        v_do_count      number ; --执行次数累计
        v_cond          varchar2(2000);
        v_start_time    timestamp := systimestamp;--开始时间
        v_current_time  timestamp; --当前时间
        v_time_exceeded boolean   := false;
    begin

        --取可归档区间的最小日期
        v_sub_sql := 'select min(trunc( ' || v_back_up.bakcolumn || ')) from ' || v_back_up.originaltable || ' '|| v_back_up.condition1;
        --取可归档区间最小日期 获取最小opdate
        --dbms_output.put_line('v_sub_sql : ' || v_sub_sql );
        execute immediate v_sub_sql into v_opdate;
        if v_opdate is null then
            return;
        end if;
        --归档基本条件［数据状态 status 或其他条件都可以写在 condition1 里面 ］
        v_cond := v_back_up.condition1 || 'and ' || v_back_up.bakcolumn || ' between :v_opdate and :v_opdate + 1 ';
        v_count_sql := ' select count(1) from ' || v_back_up.originaltable || ' ' || v_cond;
        --dbms_output.put_line('v_count_sql : ' || v_count_sql );
        execute immediate v_count_sql into v_archive_count using v_opdate, v_opdate;
        --获取最小opdate +分公司+数据状态 的数据量
        if v_archive_count = 0 then
            return;
        end if;
        if v_back_up.batchsize is not null then
            v_batch_size := v_back_up.batchsize ;
        end if;
        --dbms_output.put_line('v_archive_count = ' ||v_archive_count || ', v_batch_size = ' || v_batch_size || ' ,v_opdate = '||v_opdate );
        --计算本次归档次数,分页归档
        v_size := ceil(v_archive_count / v_batch_size);
        v_do_count := 0;
        while (v_do_count < v_size and v_time_exceeded = false)
            loop
                --检查执行时间
                v_current_time := systimestamp;
                if extract(minute from (v_current_time - v_start_time)) +
                   extract(hour from (v_current_time - v_start_time)) * 60 > v_max_min then
                    --检查是否超过 30分钟
                    v_time_exceeded := true; -- 设置超时标志
                    exit; --退出循环
                end if;

                -- 把主键插入 table_idx 索引表
                v_tmp_sql := 'insert into ' || v_back_up.idxtable || '(' || v_back_up.idxcolumns || ') '
                    || ' select ' || v_back_up.idxcolumns || ' from  ' || v_back_up.originaltable || ' '
                    || v_cond
                    || ' and rownum <= ' || v_batch_size;
                --dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql );
                execute immediate v_tmp_sql using v_opdate, v_opdate;

                v_tmp_sql := 'insert into ' || v_back_up.backuptable || ' select * from ' || v_back_up.originaltable
                    || ' where (' || v_back_up.idxcolumns || ') in (select ' || v_back_up.idxcolumns || ' from ' ||
                             v_back_up.idxtable || ')';
                --插入bak 归档表
                --dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql);
                execute immediate v_tmp_sql;

                v_tmp_sql := 'delete from ' || v_back_up.originaltable || ' where (' || v_back_up.idxcolumns ||
                             ') in (select ' || v_back_up.idxcolumns || ' from ' || v_back_up.idxtable || ')';
                --根据索引表 删除原表数据
                --dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql);
                execute immediate v_tmp_sql;

                --清空本次执行索引表
                v_tmp_sql := 'delete from ' || v_back_up.idxtable || ' where 1 = 1 ';
                --dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql );
                execute immediate v_tmp_sql;
                commit;
                v_do_count := v_do_count + 1;
            end loop;
    exception
        when others then
            rollback;
            mm_errorlog_pkg.log_error(procname_in => 'ams_backup_pkg',
                                      keyword1_in => 'do_backup_sigle',
                                      keyword3_in => 'backup id = ' || v_back_up.id || ',opdate =' ||
                                                     to_char(v_opdate, 'yyyy-mm-dd'),
                                      info_in => substr(sqlerrm, 1, 2000));
            raise_application_error(-20023, '单表备份执行错误'||substr(sqlerrm, 1, 1800));


    end do_backup_sigle;
    -- 按关联表备份
    procedure do_backup_mutil(v_back_up ams_backup_td%rowtype) is
        v_opdate        date; --可归档数据的归档最小日期
        v_sub_sql       varchar2(1000);
        v_batch_sql     varchar2(2000); --统计分公司最小归档日的数据量
        v_batchno_list  varchar2_list ;
        v_batch_size    number    := 10000;
        v_max_min       number    := 30; --最长执行时间限制
        v_do_count      number ; --执行次数累计
        v_cond          varchar2(2000);
        v_org_table     varchar2_list;
        v_bak_table     varchar2_list;
        v_batch_column     varchar2_list;
        v_start_time    timestamp := systimestamp;--开始时间
        v_current_time  timestamp; --当前时间
        v_time_exceeded boolean   := false;
    begin
        v_org_table := split_string(v_back_up.originaltable, ',');
        v_bak_table := split_string(v_back_up.backuptable, ',');
        v_batch_column := split_string(v_back_up.batchcolumn, ',');
        if v_org_table.count > 0 and v_org_table.count != v_bak_table.count and v_org_table.count != v_batch_column.count then
            raise_application_error(-20023, '多表备份配置错误,源表数量和备份表数据不一致!');
        end if;
        --取可归档区间的最小日期
        v_sub_sql := 'select min(trunc( ' || v_back_up.bakcolumn || ')) from ' || v_org_table(1) || ' '|| v_back_up.condition1;
        dbms_output.put_line('v_sub_sql : ' || v_sub_sql );
        execute immediate v_sub_sql into v_opdate;
        if v_opdate is null then
            return;
        end if;
        if v_back_up.batchsize is not null then
            v_batch_size := v_back_up.batchsize ;
        end if;

        --归档基本条件［ 多表关联时，主表必须要配置在第一个位置］
        v_cond := v_back_up.condition1 || 'and ' || v_back_up.bakcolumn || ' between :v_opdate and :v_opdate + 1 '|| ' and rownum <= '|| v_batch_size;
        v_batch_sql := ' select ' || v_batch_column(1) || ' from ' || v_org_table(1) || ' ' || v_cond;
        dbms_output.put_line('v_batch_sql : ' || v_batch_sql );
        execute immediate v_batch_sql bulk collect into v_batchno_list using v_opdate, v_opdate;
        --获取最小opdate 的数据量
        if v_batchno_list.count = 0 then
            return;
        end if;
        dbms_output.put_line('v_batchno_list count = ' ||v_batchno_list.COUNT || ', v_batch_size = ' || v_batch_size || ' ,v_opdate = '||TO_CHAR(v_opdate,'yyyy-MM-dd') );
        --计算本次归档次数,分页归档
        v_do_count := 1;
        while (v_do_count <= v_batchno_list.count and v_time_exceeded = false)
            loop
                begin
                    savepoint v_batch_tmp;
                    --检查执行时间
                    v_current_time := systimestamp;
                    if extract(minute from (v_current_time - v_start_time)) +
                       extract(hour from (v_current_time - v_start_time)) * 60 > v_max_min then
                        --检查是否超过 30分钟
                        v_time_exceeded := true; -- 设置超时标志
                        exit; --退出循环
                    end if;
                    for j in 1..v_org_table.count
                        loop
                            v_batch_sql := 'insert into ' || v_bak_table(j) || ' select * from ' || v_org_table(j) ||
                                           ' where ' || v_batch_column(j) || ' = :batchno ';
                            dbms_output.put_line('ii v_batch_sql : ' || v_batch_sql );
                            execute immediate v_batch_sql using v_batchno_list(v_do_count);
                        end loop;
                    for k in 1..v_org_table.count
                        loop
                            v_batch_sql := 'delete from ' || v_org_table(k) || ' where ' || v_batch_column(k) ||
                                           ' = :batchno ';
                            dbms_output.put_line('dd v_batch_sql : ' || v_batch_sql );
                            execute immediate v_batch_sql using v_batchno_list(v_do_count);
                        end loop;
                    commit;
                    v_do_count := v_do_count + 1;
                exception
                    when others then
                        rollback to v_batch_tmp;
                        mm_errorlog_pkg.log_error(procname_in => 'ams_backup_pkg',
                                                  keyword1_in => 'do_backup_mutil',
                                                  keyword3_in => 'opdate =' || to_char(v_opdate, 'yyyy-mm-dd')|| ',batchcolumn='||v_back_up.batchcolumn,
                                                  info_in => substr(sqlerrm, 1, 2000));
                        raise_application_error(-20023, '多表关联Loop备份执行错误'||substr(sqlerrm, 1, 1800));
                end;
            end loop;
    end do_backup_mutil;
    -- 按分公司备份
    procedure do_backup_subcompany(v_subcompany varchar2, v_back_up ams_backup_td%rowtype) is
        v_opdate        date; --可归档数据的归档最小日期
        v_sub_sql       varchar2(1000);
        v_count_sql     varchar2(2000); --统计分公司最小归档日的数据量
        v_tmp_sql       varchar2(2000);
        v_archive_count number ;
        v_max_min       number    := 30; --最长执行时间限制
        v_batch_size    number    := 10000;
        v_size          number ; --单日总次数
        v_do_count      number ; --执行次数累计
        v_guid          varchar2(100); --并发支持
        v_cond          varchar2(2000);
        v_start_time    timestamp := systimestamp;--开始时间
        v_current_time  timestamp; --当前时间
        v_time_exceeded boolean   := false;
    begin
        --取可归档区间的最小日期
        v_sub_sql := 'select min( ' || v_back_up.bakcolumn || ') from ' || v_back_up.originaltable ||' '||
                     v_back_up.condition1;
        dbms_output.put_line('v_sub_sql : ' || v_sub_sql );
        execute immediate v_sub_sql into v_opdate using v_subcompany;
        if v_opdate is null then
            return;
        end if;
        --归档基本条件［数据状态 status 或其他条件都可以写在 condition1 里面 ］
        v_cond := v_back_up.condition1 || 'and ' || v_back_up.bakcolumn || ' between :v_opdate and :v_opdate + 1 ';
        v_count_sql := ' select count(1) from ' || v_back_up.originaltable || ' ' || v_cond;
        dbms_output.put_line('v_count_sql : ' || v_count_sql );
        execute immediate v_count_sql into v_archive_count using v_subcompany, v_opdate, v_opdate;
        --获取最小opdate +分公司+数据状态 的数据量
        if v_archive_count = 0 then
            return;
        end if;
        if v_back_up.batchsize is not null then
            v_batch_size := v_back_up.batchsize ;
        end if;
        dbms_output.put_line('v_archive_count = ' ||v_archive_count || ', v_batch_size = ' || v_batch_size || ' ,v_opdate = '||TO_CHAR(v_opdate,'yyyy-MM-dd') );
        --计算本次归档次数,分页归档
        v_size := ceil(v_archive_count / v_batch_size);
        v_do_count := 0;
        while (v_do_count < v_size and v_time_exceeded = false)
            loop
                begin
                    savepoint t_point_page;
                    --检查执行时间
                    v_current_time := systimestamp;
                    if extract(minute from (v_current_time - v_start_time)) +
                       extract(hour from (v_current_time - v_start_time)) * 60 > v_max_min then
                        --检查是否超过 30分钟
                        v_time_exceeded := true; -- 设置超时标志
                        exit; --退出循环
                    end if;
                    v_guid := v_subcompany || sys_guid();
                    -- 把主键插入 table_idx 索引表
                    v_tmp_sql := 'insert into ' || v_back_up.idxtable || '(' || v_back_up.idxcolumns || ', guid ) '
                        || ' select ' || v_back_up.idxcolumns || ',''' || v_guid || ''' from  ' || v_back_up.originaltable || ' '
                        || v_cond
                        || ' and rownum <= ' || v_batch_size;
                    dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql );
                    execute immediate v_tmp_sql using v_subcompany, v_opdate, v_opdate;

                    v_tmp_sql := 'insert into ' || v_back_up.backuptable || ' select * from ' || v_back_up.originaltable
                        || ' where (' || v_back_up.idxcolumns || ') in (select ' || v_back_up.idxcolumns || ' from ' ||
                                 v_back_up.idxtable || ' where  guid = :v_guid )';
                    --插入bak 归档表
                    dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql);
                    execute immediate v_tmp_sql using  v_guid;

                    v_tmp_sql := 'delete from ' || v_back_up.originaltable || ' where (' || v_back_up.idxcolumns ||
                                 ') in (select ' || v_back_up.idxcolumns || ' from ' || v_back_up.idxtable ||
                                 ' where  guid = :v_guid)';
                    --根据索引表 删除原表数据
                    dbms_output.put_line('v_tmp_sql : ' || v_tmp_sql);
                    execute immediate v_tmp_sql using v_guid;

                    --清空本次执行索引表
                    v_tmp_sql := 'delete from ' || v_back_up.idxtable || ' where  guid = :v_guid';
                    dbms_output.put_line('v_tmp_sql :' || v_tmp_sql );
                    execute immediate v_tmp_sql using v_guid;
                    commit;
                    v_do_count := v_do_count + 1;
                exception
                    when others then
                        rollback to t_point_page;

                        mm_errorlog_pkg.log_error(procname_in => 'ams_backup_pkg',
                                                  keyword1_in => 'do_backup_subcomany',
                                                  keyword3_in => 'v_subcompany=' || v_subcompany || ',opdate =' ||
                                                                 to_char(v_opdate, 'yyyy-mm-dd'),
                                                  info_in => substr(sqlerrm, 1, 2000));
                        raise_application_error(-20023, '按分公司分区备份执行错误'||substr(sqlerrm, 1, 1800));
                end;
            end loop;
    end do_backup_subcompany;
    -- 执行执行完成
    procedure do_backup_over(v_back_up ams_backup_td%rowtype) is
        v_end_sql varchar2(1000);
        nextdate  varchar2(100);
        v_error   varchar2(2000);
    begin
        select exenexttime into nextdate from ams_backup_td where id = v_back_up.id;
        v_end_sql := 'update  ams_backup_td t set t.hibernateversion=1, t.status = ''3'', t.errormsg=''success'', t.exestarttime =' ||
                     nextdate || ', t.exeendtime = ' || nextdate || '+30 '
            || ' where t.status = ''2'' and t.id= :id ';
        execute immediate v_end_sql using v_back_up.id;
        commit;
    exception
        when others then
            rollback;
            v_error := substr(sqlerrm, 1, 2000);
            update ams_backup_td t
            set t.status   = '4', t.hibernateversion = t.hibernateversion+1,
                t.errormsg = v_error
            where t.status = '2'
              and t.id = v_back_up.id;
            commit;
    end do_backup_over;
    -- 程序入口
    procedure do_backup is
        v_back_up_list    back_up_type ;
        v_subcompany_list varchar2_list := varchar2_list();
        v_error   varchar2(2000);
    begin
        select * bulk collect
        into v_back_up_list
        from (select *
              from ams_backup_td t
              where t.ifvalid = '1'
                and t.status = '1'
                --and (t.status in ('1','3') or (t.status ='4' and t.hibernateversion < 10))
                and sysdate between t.exestarttime and t.exeendtime
              order by t.lasopdate)
        where rownum <= 10;
        dbms_output.put_line('v_back_up_list.count : ' || v_back_up_list.count );
        for i in 1..v_back_up_list.count loop
            begin
                -- 支持并发
                update ams_backup_td t
                set t.status   = '2',
                    t.errormsg ='running'
                where t.status = '1'
                  and t.id = v_back_up_list(i).id;
                if sql%rowcount = 0 then
                    continue ;
                end if;
                commit;

                if v_back_up_list(i).backtype = 0 then
                    -- 删除
                    do_backup_delete(v_back_up_list(i));
                elsif v_back_up_list(i).backtype = 1 then
                    -- 单表备份
                    do_backup_sigle(v_back_up_list(i));
                elsif v_back_up_list(i).backtype = 2 then
                    -- 多表关联批次号备份
                    do_backup_mutil(v_back_up_list(i));
                elsif v_back_up_list(i).backtype = 3 then
                    -- 按分公司的单表备份
                    v_subcompany_list := split_string(v_back_up_list(i).subcompany, ',');
                    if v_subcompany_list.count = 0 then
                        continue ;
                    end if;
                    for s in 1..v_subcompany_list.count
                        loop
                            -- 按处理分公司
                            dbms_output.put_line('s_subcompany : ' || v_subcompany_list(s) );
                            do_backup_subcompany(v_subcompany_list(s), v_back_up_list(i));
                        end loop;
                end if;
                -- 处理结束
                do_backup_over(v_back_up_list(i));
            exception when others then
                v_error := substr(sqlerrm, 1, 2000);
                update ams_backup_td t
                set t.status   = '4', t.hibernateversion = t.hibernateversion + 1,
                    t.errormsg = v_error
                where t.status = '2'
                  and t.id = v_back_up_list(i).id;
                commit;
            end;
        end loop;
    exception
        when others then
            mm_errorlog_pkg.log_error(procname_in => 'ams_backup_pkg',
                                      keyword1_in => 'do_backup',
                                      keyword3_in => 'do_backup occurred error',
                                      info_in => substr(sqlerrm, 1, 2000));
    end do_backup;

end ams_backup_pkg;

```

####  测试案例


#### 案例1，单表备份

```

-------------------单表备份

-- 源表
create table t_test_sigle_td
(
    id number default zzy_test_01.nextval not null,
    name varchar2(200) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_sigle_id primary key (id)
);
create index idx_sigle_createtime on t_test_sigle_td(createtime);

--备份表
create table t_test_sigle_td_bak
(
    id number default zzy_test_01.nextval not null,
    name varchar2(200) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_sigle_bak_id primary key (id)
);
create index idx_sigle_createtime_bak on t_test_sigle_td_bak(createtime);

--索引表
create table t_test_sigle_td_idx(
    id number not null primary key
);

```



模拟数据

```
begin
    delete from t_test_sigle_td where 1=1 ;
    for i in 1..200000 loop
        insert into t_test_sigle_td(id, name, createtime, lastopdate)
        values (i, i||'test', sysdate-(i/365), sysdate-(i/365));
        commit ;
    end loop;
end;
```


```

-- 加配置


select count(1) from t_test_sigle_td where  createtime < add_months(trunc(sysdate-7),-6);
select min(trunc(createtime)), count(1) from t_test_sigle_td where  createtime < add_months(trunc(sysdate-7),-6);

--单表备份案例
insert into ams_backup_td (id, backtype, originaltable, idxtable, idxcolumns, backuptable, backupdesc, condition1, bakcolumn, subcompany, exenexttime)
values (1, 1, 't_test_sigle_td', 't_test_sigle_td_idx','id','t_test_sigle_td_bak','单表备份t_test_sigle_td表半年前数据','where createtime < add_months(trunc(sysdate-7),-6) ', 'createtime', null,'sysdate+1/24/60');


select * from ams_backup_td ;
```


执行测试

```
begin
    -- 测试修正状态 1 ，直接更新可执行
    update ams_backup_td t set t.status='1', t.exestarttime = sysdate-1/24/60 where id =1;
    ams_backup_pkg.do_backup;
end;

```


排错

```
SELECT * FROM MM_ERROR_LOG T ORDER BY T.LOGDATE DESC ;
select t.errormsg, t.* from ams_backup_td t;
```


检查结果

```

/*
可进行多轮测试，检查是否按日备份
t_test_sigle_td,199511
t_test_sigle_td_bak,489

t_test_sigle_td,199146
t_test_sigle_td_bak,854
*/

select 't_test_sigle_td' as tt, count(1) from t_test_sigle_td
union all
select 't_test_sigle_td_bak' as tt, count(1) from t_test_sigle_td_bak
union all
select 't_test_sigle_td_idx' as tt, count(1) from t_test_sigle_td_idx ;
```


#### 案例2  关联表备份

```
------------------ 关联式备份
create sequence zzy_test_01 minvalue 1000 maxvalue 99999999999999 start with 1000 ;

create table t_test_td
(
    id number default zzy_test_01.nextval not null,
    name varchar2(200) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_id primary key (id)
);
create index idx_createtime on t_test_td(createtime);
create table t_test_td_bak
(
    id number not null,
    name varchar2(200) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_id_bak primary key (id)
);
create index idx_createtime_bak on t_test_td_bak(createtime);

create table t_test_detail_td
(
    detailid number default zzy_test_01.nextval not null,
    mainid number not null,
    name varchar2(100) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_id_detail primary key (detailid)
);
create index idx_mainid_test on t_test_detail_td(mainid);

create table t_test_detail_td_bak
(
    detailid number  not null,
    mainid number not null,
    name varchar2(100) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_id_detail_bak primary key (detailid)
);
create index idx_mainid_test_bak on t_test_detail_td_bak(mainid);

begin
    delete from t_test_detail_td ;
    delete from t_test_td ;
    for i in 1001..10000 loop
        for x in 1..100 loop
        insert into t_test_detail_td(mainid, name, createtime, lastopdate)
        values (i, i||'test'||x, sysdate-i, sysdate-1);
        end loop ;
        insert into t_test_td(id, name, createtime, lastopdate)
        values (i, i||'test', sysdate-(i/365), sysdate-(i/365));
        commit ;
    end loop;
end;

```


模拟数据

```
begin
    delete from t_test_detail_td ;
    delete from t_test_td ;
    for i in 1001..10000 loop
        for x in 1..100 loop
        insert into t_test_detail_td(mainid, name, createtime, lastopdate)
        values (i, i||'test'||x, sysdate-i, sysdate-1);
        end loop ;
        insert into t_test_td(id, name, createtime, lastopdate)
        values (i, i||'test', sysdate-(i/365), sysdate-(i/365));
        commit ;
    end loop;
end;
```


加配置

```

select trunc( createtime), count(1) from t_test_td where createtime < trunc(sysdate-7)
group by trunc( createtime) ;

select count(1) from t_test_td where createtime < trunc(sysdate-7) and createtime between date'2025-11-13'  and date'2025-11-13'+1  and rownum <= 10 ;
 

-- 关联表备份案例
insert into ams_backup_td (id, backtype, originaltable, idxtable, idxcolumns, backuptable, backupdesc, condition1, bakcolumn, batchcolumn, subcompany, exenexttime)
values (2, 2, 't_test_td,t_test_detail_td', '','','t_test_td_bak,t_test_detail_td_bak','关联表备份7天前数据','where createtime < trunc(sysdate-7) ', 'createtime', 'id,mainid',null,'sysdate+10/24/60');

select * from ams_backup_td ;
```

执行测试

```
begin
    -- 测试修正状态 1 ，直接更新可执行
    update ams_backup_td t set t.status='1', t.exestarttime = sysdate-1/24/60 where id =2;
    ams_backup_pkg.do_backup;
end;

```

排错

```
SELECT * FROM MM_ERROR_LOG T ORDER BY T.LOGDATE DESC ;
select t.errormsg, t.* from ams_backup_td t;
```


检查结果

```
/*
t_test_td,8341
t_test_td_bak,659
t_test_detail_td,834100
t_test_detail_td_bak,65900

t_test_td,7976
t_test_td_bak,1024
t_test_detail_td,797600
t_test_detail_td_bak,102400

*/
select 't_test_td' as tt, count(1) from t_test_td
union all
select 't_test_td_bak' as tt, count(1) from t_test_td_bak
union all
select 't_test_detail_td' as tt, count(1) from t_test_detail_td
union all
select 't_test_detail_td_bak' as tt, count(1) from t_test_detail_td_bak;
```


#### 案例3  分区表备份

建表

```

create sequence seq_t_sub_test_td_id minvalue 1000 maxvalue 99999999999999 start with 1000 ;
drop table t_sub_test_td ;

create table t_sub_test_td (
    id number  not null,
    subcompany varchar2(10) not null,
    name varchar2(100) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_id_sub_test_id primary key (id,subcompany)
) partition by list ( subcompany ) (
  partition subcompany_1010100 values ('1010100'),
  partition subcompany_1020100 values ('1020100'),
  partition subcompany_2010100 values ('2010100'),
  partition subcompany_2020100 values ('2020100'),
  partition subcompany_3020100 values ('3020100'),
  partition subcompany_3040100 values ('3040100'),
  partition subcompany_4010100 values ('4010100'),
  partition subcompany_4020100 values ('4020100'),
  partition subcompany_5010100 values ('5010100'),
  partition subcompany_5020100 values ('5020100'),
  partition subcompany_6010100 values ('6010100'),
  partition subcompany_6020100 values ('6020100')
);

-- 索引表
create table t_sub_test_td_idx
(
    id number  not null,
    subcompany number not null,
    guid number not null,
    constraint pk_t_sub_test_td_idx primary key(id,subcompany)
);
-- 备份表
create table t_sub_test_td_bak(
    id number  not null,
    subcompany number not null,
    name varchar2(100) default 'test',
    createtime date default sysdate,
    lastopdate date default sysdate,
    constraint pk_id_sub_test_bak primary key (id,subcompany)
) partition by list ( subcompany ) (
  partition subcompany_1010100 values ('1010100'),
  partition subcompany_1020100 values ('1020100'),
  partition subcompany_2010100 values ('2010100'),
  partition subcompany_2020100 values ('2020100'),
  partition subcompany_3020100 values ('3020100'),
  partition subcompany_3040100 values ('3040100'),
  partition subcompany_4010100 values ('4010100'),
  partition subcompany_4020100 values ('4020100'),
  partition subcompany_5010100 values ('5010100'),
  partition subcompany_5020100 values ('5020100'),
  partition subcompany_6010100 values ('6010100'),
  partition subcompany_6020100 values ('6020100')
);

```


模拟数据


```
begin
    for rec in (select PARTITION_NAME from USER_TAB_PARTITIONS where TABLE_NAME = 'T_SUB_TEST_TD' ) loop
        for i in 1..50000 loop
        insert into t_sub_test_td (id,subcompany,name, createtime)
        values (seq_t_sub_test_td_id.nextval, substr(rec.PARTITION_NAME, 12), rec.PARTITION_NAME, sysdate - (i/365));
        end loop;
        commit ;
    end loop;
end;

select subcompany, count(*) from t_sub_test_td group by subcompany;
```


加配置

```
select min(createtime), count(*) from t_sub_test_td where subcompany = :subcompany and createtime < add_months(trunc(sysdate-7),-2);


-- 分区表备份案例
insert into ams_backup_td (id, backtype, originaltable, idxtable, idxcolumns, backuptable, backupdesc, condition1, bakcolumn, batchcolumn, subcompany, exenexttime)
values (3, 3, 't_sub_test_td', 't_sub_test_td_idx','id, subcompany','t_sub_test_td_bak','分区表备份三年前数据','where subcompany = :subcompany and createtime < add_months(trunc(sysdate-7),-2) ', 'createtime', '','1010100,1020100,2010100,2020100', 'sysdate+10/24/60');
insert into ams_backup_td (id, backtype, originaltable, idxtable, idxcolumns, backuptable, backupdesc, condition1, bakcolumn, batchcolumn, subcompany, exenexttime)
values (4, 3, 't_sub_test_td', 't_sub_test_td_idx','id, subcompany','t_sub_test_td_bak','分区表备份三年前数据','where subcompany = :subcompany and createtime < add_months(trunc(sysdate-7),-2) ', 'createtime', '','3020100,3040100,4010100,4020100', 'sysdate+10/24/60');
insert into ams_backup_td (id, backtype, originaltable, idxtable, idxcolumns, backuptable, backupdesc, condition1, bakcolumn, batchcolumn, subcompany, exenexttime)
values (5, 3, 't_sub_test_td', 't_sub_test_td_idx','id, subcompany','t_sub_test_td_bak','分区表备份三年前数据','where subcompany = :subcompany and createtime < add_months(trunc(sysdate-7),-2) ', 'createtime', '','5010100,5020100,6010100,6020100', 'sysdate+10/24/60');


```


执行测试

```
begin
    -- 测试修正状态 1 ，直接更新可执行
    --update ams_backup_td t set t.status='1', t.exestarttime = sysdate + 1/24 where id in (1,2);
    update ams_backup_td t set t.status='1', t.exestarttime = sysdate - 1/24 where id in (3,4,5);
    ams_backup_pkg.do_backup;
end;
```


排错

```
SELECT * FROM MM_ERROR_LOG T ORDER BY T.LOGDATE DESC ;
select t.errormsg, t.* from ams_backup_td t;
```

执行结果


```

/*
t_sub_test_td,600000
t_sub_test_td_bak,0

t_sub_test_td,595608
t_sub_test_td_bak,4392

--看看分公司
t_sub_test_td,2010100,49634
t_sub_test_td,3020100,49634

t_sub_test_td_bak,1010100,366
t_sub_test_td_bak,2010100,366

*/
select 't_sub_test_td' as tt, count(1) from t_sub_test_td
union all
select 't_sub_test_td_bak' as tt, count(1) from t_sub_test_td_bak ;

(select 't_sub_test_td' as tt, subcompany, count(1) from t_sub_test_td group by subcompany)
union all
(select 't_sub_test_td_bak' as tt, subcompany, count(1) from t_sub_test_td_bak group by subcompany);
```


