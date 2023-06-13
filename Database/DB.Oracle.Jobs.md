---
layout: default
title: ORACLE Jobs
nav_order: 21
parent: Database
---

# ORACLE Jobs
{: .no_toc }

## TABLE of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## oracle 

Oracle 在线学习：https://livesql.oracle.com


### Oracle DBA_JOBS



### Oracle Schedule

在11g中，Oracle提供了一个新建的Scheduler特性，帮助将作业实现自动化。它还可以帮助你控制资源的利用与并可以将数据库中的作业按优先顺序执行。传统的dbms_jobs的一个限制是它只能调度基于PL/SQL的作业，不能用来调度操作系统的可执行文件或脚本。



dbms_scheduler.create_job

授权

```sql
grant scheduler_admin  to ams
grant manage scheduler to ams
grant  create job  to ams
```

create job系统权限允许做如下工作：

创建作业（job）、进度表（schedule）、程序（program）、链（chain）和事件（event）。

#### 1、使用procedure创建调度任务

```sql
begin

    dbms_scheduler.create_job(job_name            => 'xxxx',
                            job_type            => 'STORED_PROCEDURE',
                            job_action          => 'xxxx',
                            start_date          => to_date('04-08-2018 04:50:15', 'dd-mm-yyyy hh24:mi:ss'),
                            repeat_interval     => 'Freq=Daily;Interval=1;ByHour=4;ByMinute=50;',
                            end_date            => null,
                            job_class           => 'DEFAULT_JOB_CLASS',
                            enabled             => true,
                            auto_drop           => false,
                            comments            => 'xxxxxxx');
end;
```
参数说明：

 -   job_name： 任务名称
 -   job_type：有三种类型，PL/SQL Block、Stored procedure、Executable
 -   job_action：根据job_type的不同，有不同的含义
      -  如果job_type指定的是存储过程，就需要指定存储过程的名字;
      -  如果job_type指定的是PL/SQL块，就需要输入完整的PL/SQL代码;
      -  如果job_type指定的外部程序，就需要输入script的名称或者操作系统的指令名

 -   start_date：开始时间
 -   repeat_interval：运行的时间间隔，上面例子是每天23点运行一次
 -   end_date：到期时间
 -   enabled：创建后自动激活
 -   auto_drop：默认true,即当job执行完毕都到期是否直接删除job

#### 2、常用的写法

```sql
--删除job
 begin
     dbms_scheduler.drop_job('xxxx');
end;

--禁用
 begin
     dbms_scheduler.disable('xxxx');
end;

    --启动
 begin
     dbms_scheduler.enable('xxxx');
 end;


exec dbms_scheduler.enable('demo_job');  --启用jobs  
     
exec dbms_scheduler.disable('demo_job');  --禁用jobs  
     
exec dbms_scheduler.run_job('demo_job');  --执行jobs  
     
exec dbms_scheduler.stop_job('demo_job');  --停止jobs  
     
exec dbms_scheduler.drop_job('demo_job');  --删除jobs 
```

#### 3、常用频率

1.每周5的时候运行，以下3条实现功能一样

    REPEAT_INTERVAL => 'FREQ=DAILY; BYDAY=FRI';

    REPEAT_INTERVAL => 'FREQ=WEEKLY; BYDAY=FRI';

    REPEAT_INTERVAL => 'FREQ=YEARLY; BYDAY=FRI';

2.每隔一周运行一次，仅在周5运行

    REPEAT_INTERVAL => 'FREQ=WEEKLY; INTERVAL=2; BYDAY=FRI’;

3.每月最后一天运行

    REPEAT_INTERVAL => 'FREQ=MONTHLY; BYMONTHDAY=-1'

4.在3月10日运行

    REPEAT_INTERVAL => 'FREQ=YEARLY; BYMONTH=MAR; BYMONTHDAY=10’;

    REPEAT_INTERVAL => 'FREQ=YEARLY; BYDATE=0310';

5.每10隔天运行

    REPEAT_INTERVAL => 'FREQ=DAILY; INTERVAL=10’;

6.每天的下午4、5、6点时运行

    REPEAT_INTERVAL => 'FREQ=DAILY; BYHOUR=16,17,18’;

7.每月29日运行

    REPEAT_INTERVAL => 'FREQ=MONTHLY; BYMONTHDAY=29’;

8.每年的最后一个周5运行

    REPEAT_INTERVAL => 'FREQ=YEARLY; BYDAY=-1FRI’;

9.每隔50个小时运行

    REPEAT_INTERVAL => 'FREQ=HOURLY; INTERVAL=50’;

使用plsql块创建job

```sql
 --创建作业程序(pragram)
begin
     sys.dbms_scheduler.create_program(
     program_name=>'demo_pragram',
     program_type=>'PLSQL_BLOCK',
     program_action=>'BEGIN p_insSysdate;END;',
     enabled=>TRUE,
     comments=>'test pragram');
end;
/

--创建job
begin
     sys.dbms_scheduler.create_job(
     job_name=>'demo_job',
     program_name=>'demo_pragram',
     schedule_name=>'demo_scheduler',
     job_class=>'DEFAULT_JOB_CLASS',
     enabled=>true,
     auto_drop=>true,
     comments=>'test job');
end;
 /

--执行job
exec sys.dbms_scheduler.run_job('demo_job');
```

shell 调度

```bash
vi test.sh 

#!/bin/bash
source /home/oracle/.bash_profile
/bin/date


BEGIN
  dbms_scheduler.create_job(job_name     => 'JOB_TEST.sh',
                            job_type => 'EXECUTABLE',
                            job_action => '/home/oracle/test.sh',
                            enabled      => TRUE,
                            auto_drop => FALSE); --->值为TRUE用于激活JOB 
END;
/
```


相关表的查询
     
````sql

select job_name,state from dba_scheduler_jobs;

select owner,job_name, status,error#,cpu_used,additional_info from dba_scheduler_job_run_details where error#>0;

```