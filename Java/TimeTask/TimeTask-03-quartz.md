---
layout: default
title: Quartz
parent: Java TimeTask
grand_parent: Java
nav_order: 30
---


> 本文主要记录学习Quartz的相关内容。
  整理于2020-06-25，最后更新2020-07-18。



### 一、Quartz简述

关于调度的实现方式主要有三种：

（1）Java自带的Api有个Timer类和TimerTask类。

（2）Spring自带的Spring Task调度工具。

（3）Quartz（读阔子）开源框架，功能强大，使用起来稍显复杂。



> Quartz是一个完全由java编写的开源作业调度框架，由OpenSymphony组织开源出来。作业调度其实就是按照程序的设定，根据定义的执行时间频次表达去执行某个作业任务。

之前说过一个定时任务的基本组件有：执行的任务类、触发任务的触发器、管理任务的任务调度器。

那么Quartz同样有这三个基本的组件，但是除此之外还有更丰富的组件。在`Quartz-2.0`之后增加了构造器模式创建示例，监听器的集中管理接口等内容，所以有些地方在写法上稍有不同。




### 二、Quartz的基本组件

#### 1、Scheduler调度器

`Scheduler`在使用之前需要实例化。一般通过调度工厂`SchedulerFactory`来创建一个实例。

```java
//得到一个默认的调度工厂
SchedulerFactory schedulerFactory = new StdSchedulerFactory
Scheduler sched = schedFact.getScheduler();
```

`scheduler`实例化后，可以启动(start)、暂停(stand-by)、停止(shutdown)、还可以设置自定义相关监听。

【注意，此处只是列出一些常见方法，实际使用先后顺序】

```java
scheduler.scheduleJob(jobDetail, simpleTrigger);//添加调度任务

scheduler.start();  //启动
scheduler.standby(); //暂停
scheduler.shutdown();//关闭

scheduler.addJobListener(new UpJobListener()); //添加自定义任务执行监听
scheduler.addTriggerListener(new UpTriggerListener()); //添加自定义触发器监听
scheduler.addSchedulerListener(new UpSchedulerListener()); //添加自定义调度器监听
```

​	需要注意的是：scheduler被停止后，除非重新实例化，否则不能重新启动；只有当scheduler启动后，即使处于暂停状态也不行，trigger才会被触发（job才能被执行）。

​       Quartz-2.0之后的监听器调整为使用监听管理的`ListenerManager`接口进行集中管理的。使用添加或移除都需要先拿到`ListenerManager`之后再进行操作，后面会有具体示例。

#### 2、Job作业接口

Job接口只有一个execute方法，类似线程的run方法，用来执行业务作业逻辑。

业务作业实现了Job接口之后就可以由调度器Schedule进行管理执行。

```java
package com.smallrose.web.app.quartz.job;

import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class TestJob implements Job {

	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		JobDataMap dataMap = context.getJobDetail().getJobDataMap();
        System.out.println(" job executed " + dataMap);
	}

}
```



#### 3、JobDetail作业实例

`JobDetail`用于定义作业的实例（Job实例），为Job实例提供属性设置，用来存储特定Job实例的状态信息，调度器Schedule需要借助`JobDetail`对象来添加Job实例。

`JobDetail`的重要属性有：

​	`name`---- 任务名称

​	`group` ---- 任务所在组名称

​	`jobClass` ---- 任务执行类，也就是实现了`Job`接口的业务作业类。

​	`jobDataMap` ---- 任务的状态数据，其实就是一些相关参数，可以动态传入。

```java
Map<String,Object> map = new HashMap<Srting,Object>();
JobDetail JobDetail = new JobDetail(jobName, jobGroupName, jobClass);
JobDetail.getJobDataMap().put("data", map);
// 该数据可以通过Job中的JobDataMap dataMap = context.getJobDetail().getJobDataMap();来进行参数传递值
```



#### 4、Trigger触发器

Trigger是用来定义执行业务作业的计划组件。

Trigger的类型主要有4种：

​	（1）`SimpleTrigger`：从某一个时间开始，以一定的时间间隔来执行任务。它主要有两个属性，`repeatInterval` 重复的时间间隔；`repeatCount` 重复的次数，实际上执行的次数是n+1,因为在`startTime`的时候会执行一次。

​	（2） `CronTrigger `：适合于复杂的任务，使用`cron`表达式来定义执行规则。

​	（3）`CalendarIntervalTrigger`：类似于`SimpleTrigger`，指定从某一个时间开始，以一定的时间间隔执行的任务。但是`CalendarIntervalTrigger`执行任务的时间间隔比`SimpleTrigger`要丰富，它支持的间隔单位有秒，分钟，小时，天，月，年，星期。它的主要两个属性，`interval`执行间隔；`intervalUnit `执行间隔的单位（秒，分钟，小时，天，月，年，星期）

相较于`SimpleTrigger`有两个优势：

1、更方便，比如每隔1小时执行，不用自己去计算1小时等于多少毫秒。
2、支持不是固定长度的间隔，比如间隔为月和年。但劣势是精度只能到秒。

​	（4）`DailyTimeIntervalTrigger`：指定每天的某个时间段内，以一定的时间间隔执行任务。并且它可以支持指定星期。

它适合的任务类似于：指定每天9:00 至 18:00，每隔70秒执行一次，并且只要周一至周五执行。

它的属性有`startTimeOfDay `每天开始时间；`endTimeOfDay `每天结束时间；`daysOfWeek `需要执行的星期；`interval` 执行间隔；`intervalUnit `执行间隔的单位（秒，分钟，小时，天，月，年，星期）；`repeatCount `重复次数



Trigger公共属性主要有：

`jobKey`属性，当触发器触发时被执行的job定位；

`startTime`属性：设置Trigger第一次触发的时间；

`endTime`属性：表示Trigger失效的时间点。

优先级(priority)、错过触发(misfire)、日历(calendar)等

​	优先级（priority）

>Quartz线程池的工作线程太少或者触发器较多时，Quartz可能没有足够的资源同时触发所有的trigger；这种情况下，你可能希望控制哪些trigger优先使用Quartz的工作线程，要达到该目的，可以在trigger上设置priority属性。
>
>注意：只有同时触发的trigger之间才会比较优先级。比如：11:59触发的trigger总是在12:00触发的trigger之前执行。
>
>注意：如果trigger是可恢复的，在恢复后再调度时，优先级与原trigger是一样的。

```java
SimpleTrigger simpleTrigger = new SimpleTrigger(triggerName, triggerGroupName);
simpleTrigger.setStartTime(new Date());

// set the interval, how often the job should run (10 seconds here) 
simpleTrigger.setRepeatInterval(3000); //执行间隔毫秒
    
simpleTrigger.setRepeatCount(10);//重复执行次数（不含第一次执行）

//结束时间
simpleTrigger.setEndTime(new Date(ctime + 60000L));
// 设置触发器的优先级 默认是 5
simpleTrigger.setPriority(10);
```



​	错过触发（misfire）

>如果scheduler关闭了（系统崩溃），或者Quartz线程池中没有可用的线程来执行job（任务时间过长），此时持久性的trigger就会错过(miss)其触发时间，即错过触发(misfire)。
>
>不同类型的trigger，有不同的misfire机制。它们默认都使用“智能机制(smart policy)”，即根据trigger的类型和配置动态调整行为。当scheduler启动的时候，查询所有错过触发(misfire)的持久性trigger。然后根据它们各自的misfire机制更新trigger的信息。



​	日历（Calendar）

> Quartz的`org.quartz.Calendar`对象(不是`java.util.Calendar`对象)可以在定义和存储trigger的时候与trigger进行关联。Calendar用于从trigger的调度计划中排除时间段。比如，可以创建一个trigger，每个工作日的上午9:30执行，然后增加一个Calendar，排除掉所有节假日。
>
> 任何实现了Calendar接口的可序列化对象都可以作为Calendar对象。一个Trigger可以和多个Calendar关联，以便排除或包含某些时间点。

```java
package org.quartz;

public interface Calendar {
  //参数是毫秒单位的时间戳
  public boolean isTimeIncluded(long timeStamp);
  public long getNextIncludedTime(long timeStamp);

}
```

Quartz提供的`org.quartz.impl.HolidayCalendar`类可以很方便地实现排除整个日期。

```java
HolidayCalendar cal = new HolidayCalendar();
cal.addExcludedDate( someDate );
cal.addExcludedDate( someOtherDate );

scheduler.addCalendar("myHolidaysName", cal, false);


Trigger t = newTrigger()
    .withIdentity("myTriggerName")
    .forJob("myJobName")
    .withSchedule(dailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
    .modifiedByCalendar("myHolidaysName") // but not on holidays
    .build();
```

只要的日历类有：

| 日历类          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| HolidayCalendar | 指定特定的日期，比如20200613。精度到天。                     |
| DailyCalendar   | 指定每天的时间段（rangeStartingTime, rangeEndingTime)，格式是HH:MM[:SS[:mmm]]。也就是最大精度可以到毫秒。 |
| WeeklyCalendar  | 指定每星期的星期几，可选值比如为java.util.Calendar.SUNDAY。精度是天。 |
| MonthlyCalendar | 指定每月的几号。可选值为1-31。精度是天                       |
| AnnualCalendar  | 指定每年的哪一天。使用方式如上例。精度是天。                 |
| CronCalendar    | 指定`Cron`表达式。精度取决于`Cron`表达式，也就是最大精度可以到秒。 |



#### 5、JobBuilder构造器

[注意：版本较低的可能没有构造器方式，即可能没有`JobBuilder`、`TriggerBuilder`等Builder模式]

`JobBuilder`也是用于定义/构建`JobDetail`实例，定义作业的实例。`JobBuilder`的属性和`JobDetail`的属性基本一致，这样得以在build()方法里面，完成对`JobDetail`进行创建。是一个构造器风格的API。

```java
JobDetail job = newJob(MyJob.class)
             .withIdentity("myJob")
             .build();
             
Trigger trigger = newTrigger() 
             .withIdentity(triggerKey("myTrigger", "myTriggerGroup"))
             .withSchedule(simpleSchedule()
             .withIntervalInHours(1)
             .repeatForever())
             .startAt(futureDate(10, MINUTES))
             .build();
         
scheduler.scheduleJob(job, trigger);
```

#### 6、TriggerBuilder构造器

`TriggerBuilder`  用于定义/构建触发器实例。是一个构造器风格的API。

```java
JobDetail job = newJob(TestJob.class)
             .withIdentity("myJob")
             .build();
             
Trigger trigger = newTrigger() 
             .withIdentity(triggerKey("myTrigger", "myTriggerGroup"))
             .withSchedule(simpleSchedule()
             .withIntervalInHours(1)
             .repeatForever())
             .startAt(futureDate(10, MINUTES))
             .build();
         
scheduler.scheduleJob(job, trigger);
```



### 三、触发器使用

​	前面学习了不同的Trigger，常用2种是：`SimpleTrigger`和 `CronTrigger `。

#### 1、SimpleTrigger

简易触发器的使用场景是：在具体的时间点执行一次，或者在具体的时间点执行并且以指定的间隔重复执行若干次。

`SimpleTrigger`的属性包括：开始时间、结束时间、重复次数以及重复的间隔。

重复次数，可以是0、正整数，以及常量`SimpleTrigger.REPEAT_INDEFINITELY`。

重复的间隔，必须是0，或者long型的正数，表示毫秒。注意:，如果重复间隔为0，trigger将会以重复次数并发执行(或者以scheduler可以处理的近似并发数)。

endTime属性的值会覆盖设置重复次数的属性值；比如，你可以创建一个trigger，在终止时间之前每隔10秒执行一次，你不需要去计算在开始时间和终止时间之间的重复次数，只需要设置终止时间并将重复次数设置为REPEAT_INDEFINITELY(当然，你也可以将重复次数设置为一个很大的值，并保证该值比trigger在终止时间之前实际触发的次数要大即可)。

##### 构建SimpleTrigger

`SimpleTrigger`实例通过`TriggerBuilder`设置主要的属性，通过`SimpleScheduleBuilder`设置与`SimpleTrigger`相关的属性。要使用这些builder的静态方法，需要静态导入：

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
import static org.quartz.DateBuilder.*:
```

##### 常见的触发实例：

如：指定时间开始触发，不重复：

```java
Scheduler scheduler = schedulerFactory.getScheduler();
		 
Date someDate = new Date().from(Instant.now().plusSeconds(30));
		
SimpleTrigger simpleTrigger =  (SimpleTrigger)newTrigger()
        .withIdentity("trigger1", "group1")
        .startAt(someDate)        // some Date 
        .forJob("job1", "group1")  // identify job with name, group strings
        .build();
```

指定时间触发，每隔10秒执行一次，重复10次：

```java
simpleTrigger =  (SimpleTrigger)newTrigger()
        .withIdentity("trigger2", "group1")
        .startAt(myTimeToStartFiring)  // if a start time is not given (if this line were omitted), "now" is implied
        .withSchedule(simpleSchedule()
        .withIntervalInSeconds(10)
         .withRepeatCount(10)) // note that 10 repeats will give a total of 11 firings
        .forJob(myJob) // identify job with handle to its JobDetail itself             
        .build();
```

5分钟以后开始触发，仅执行一次：

```java
simpleTrigger =  (SimpleTrigger)newTrigger()
        .withIdentity("trigger3", "group1")
        .startAt(futureDate(5, IntervalUnit.MINUTE)) // use DateBuilder to create a date in the future
        .forJob(myJobKey) // identify job with its JobKey
        .build();
```

立即触发，每个5分钟执行一次，直到22:00：

```java
simpleTrigger =  (SimpleTrigger)newTrigger()
        .withIdentity("trigger4", "group1")
        .withSchedule(simpleSchedule()
            .withIntervalInMinutes(5)
            .repeatForever())
        .endAt(dateOf(22, 0, 0))
        .build();
```

在下一小时整点触发，每个2小时执行一次，一直重复：

```java
simpleTrigger =  (SimpleTrigger)newTrigger()
        .withIdentity("trigger5") // 没有定义组名，"trigger5"进入默认的group
        .startAt(evenHourDate(null)) //整点触发 (minutes and seconds zero ("00:00"))
        .withSchedule(simpleSchedule()
        .withIntervalInHours(2)
        .repeatForever())//一直重复
        // note that in this example, 'forJob(..)' is not called which is valid 
        // if the trigger is passed to the scheduler along with the job  
        .build();

scheduler.scheduleJob(trigger, job);
```

还有一些没有列举的方法，在实际使用中可以点开代码看。

> TriggerBuilder(以及Quartz的其它builder)会为那些没有被显式设置的属性选择合理的默认值。比如：如果你没有调用*withIdentity(..)*方法，TriggerBuilder会为trigger生成一个随机的名称；如果没有调用*startAt(..)*方法，则默认使用当前时间，即trigger立即生效。



**`SimpleTrigger` Misfire策略：**

所有的trigger都有一个Trigger.MISFIRE_INSTRUCTION_SMART_POLICY策略可以使用，该策略也是所有trigger的默认策略。对于`SimpleTrigger`，这个“智能机制”将根据触发器实例的状态和配置来决定其行为。具体如下：

如果Repeat Count=0：只执行一次　

​	 instruction selected = MISFIRE_INSTRUCTION_FIRE_NOW;

如果Repeat Count=REPEAT_INDEFINITELY：无限次执行

​	instruction selected = MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT;

如果Repeat Count>0：执行多次（有限）

　　instruction selected = MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT;



`SimpleTrigger`常见策略：

| 常见策略                                                 | 策略说明                                               |
| -------------------------------------------------------- | ------------------------------------------------------ |
| MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY                | 忽略所有的超时状态，按照触发器的策略执行。             |
| MISFIRE_INSTRUCTION_FIRE_NOW                             | 立刻执行。对于不会重复执行的任务，这是默认的处理策略。 |
| MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT | 在下一个激活点执行，且超时期内错过的执行机会作废。     |
| MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_COUNT  | 立即执行，且超时期内错过的执行机会作废。               |
| MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT  | 在下一个激活点执行，并重复到指定的次数。               |
| MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_COUNT   | 立即执行，并重复到指定的次数。                         |
| MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY                | 忽略所有的超时状态，按照触发器的策略执行。             |



#### 2、CronTrigger 

`Cron`表达式触发器的使用场景是：

基于时间表，在复杂的时间点，复杂的执行周期执行一次或多次。

例如：“每周一到周五上午8:00”或“每月最后一个星期五凌晨1:30”。 

 `CronTrigger `类似于在给定的时间触发作业，和Linux的“`cron-like`”调度定义相似。“`Cron Expression`”字符串的格式记录在`CronExpression`类中。

##### Cron Expressions

`Cron-Expressions`用于配置`CronTrigger`的实例。`Cron Expressions`是由七个子表达式组成的字符串，用于描述日程表的各个细节。这些子表达式用空格分隔，七个子表达式依次为：` 秒 分 时 日 月 周 年`

| 位置 | 说明         | 允许值 | 特殊值                             |
| ---- | ------------ | ------ | ---------------------------------- |
| 1    | Seconds      | 0-59   | `,`  `-` `*`  `/`                  |
| 2    | Minutes      | 0-59   | `,`  `-` `*`  `/`                  |
| 3    | Hours        | 0-23   | `,`  `-` `*`  `/`                  |
| 4    | Day-of-Month | 1-31   | `,`  `-` `*`  `/` `?`  `L` `W` `C` |
| 5    | Month        | 1-12   | `,`  `-` `*`  `/`                  |
| 6    | Day-of-Week  | 1-7    | `,`  `-` `*`  `/` `?` `L` `C` `#`  |
| 7    | Year (可选)  |        | `,`  `-` `*`  `/`                  |

`Cron`表达式对特殊字符的大小写不敏感，对代表星期的缩写英文大小写也不敏感。

特殊符号说明：

| 符号 | 说明 |
| ---- | ---- |
|**星号(\*)**|可用在所有字段中，表示对应时间域的每一个时刻，例如，* 在分钟字段时，表示“每分钟”；|
|**问号（?）**|该字符只在日期和星期字段中使用，它通常指定为“无意义的值”，相当于占位符；|
|**减号(-)**|表达一个范围，如在小时字段中使用“10-12”，则表示从10到12点，即10,11,12；|
|**逗号(,)**|表达一个列表值，如在星期字段中使用“MON,WED,FRI”，则表示星期一，星期三和星期五；|
|**斜杠(/)**|x/y表达一个等步长序列，x为起始值，y为增量步长值。如在分钟字段中使用0/15，则表示为0,15,30和45秒，而5/15在分钟字段中表示5,20,35,50，你也可以使用*/y，它等同于0/y；|
|**L** &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|该字符只在日期和星期字段中使用，代表“Last”的意思，但它在两个字段中意思不同。<br/>L在日期字段中，表示这个月份的最后一天，如一月的31号，非闰年二月的28号；<br/>如果L用在星期中，则表示星期六，等同于7。<br/>但是，如果L出现在星期字段里，而且在前面有一个数值X，则表示“这个月的最后X天”，例如，6L表示该月的最后星期五；|
|**W**|该字符只能出现在日期字段里，是对前导日期的修饰，表示离该日期最近的工作日。<br/>例如15W表示离该月15号最近的工作日，如果该月15号是星期六，则匹配14号星期五；如果15日是星期日，则匹配16号星期一；如果15号是星期二，那结果就是15号星期二。但必须注意关联的匹配日期不能够跨月，如你指定1W，如果1号是星期六，结果匹配的是3号星期一，而非上个月最后的那天。W字符串只能指定单一日期，而不能指定日期范围；|
|**LW组合**|在日期字段可以组合使用LW，它的意思是当月的最后一个工作日；|
|**井号(#) **|该字符只能在星期字段中使用，表示当月某个工作日。如6#3表示当月的第三个星期五(6表示星期五，#3表示当前的第三个)，而4#5表示当月的第五个星期三，假设当月没有第五个星期三，忽略不触发；|
|**C **|该字符只在日期和星期字段中使用，代表“Calendar”的意思。它的意思是计划所关联的日期，如果日期没有被关联，则相当于日历中所有日期。例如5C在日期字段中就相当于日历5日以后的第一天。1C在星期字段中相当于星期日后的第一天。|

##### 常见表达式示例：

| 表达式                       | 含义                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `"0 0 12 * * ?"`             | 每天12点触发执行                                             |
| `"0 15 10 ? * *"`            | 每天10点15分触发执行                                         |
| `"0 15 10 * * ?"`            | 每天10点15分触发执行                                         |
| `"0 15 10 * * ? *"`          | 每天10点15分触发执行                                         |
| `"0 15 10 * * ? 2020"`       | 在2020年内，每天10点15分触发执行                             |
| `"0 * 14 * * ?"`             | 每天从下午2点开始到下午2点59分结束，每分钟触发执行一次       |
| `"0 0/5 14 * * ?"`           | 每天从下午2点到下午2点55分，每5分钟触发一次                  |
| `"0 0/5 14,18 * * ?"`        | 每天从下午2点开始到下午2点55分每5分钟触发一次<br/>从下午6点开始到下午6点55分结束，每5分钟触发一次 |
| `"0 0-5 14 * * ?"`           | 每天从下午2点开始到下午2点05分结束，每分钟触发一次           |
| `"0 10,44 14 ? 3 WED"`       | （每年）三月每周三下午2:10和2:44触发                         |
| `"0 15 10 ? * MON-FRI"`      | 每周一、二、三、四、五上午10:15触发                          |
| `"0 15 10 15 * ?"`           | 每月15日上午10:15触发                                        |
| `"0 15 10 L * ?"`            | 每月最后一天上午10:15触发                                    |
| `"0 15 10 * ？ 6L"`          | 每月最后一个星期五上午10:15触发                              |
| `"0 15 10 ? * 6L"`           | 每月最后一个星期五上午10:15触发                              |
| `"0 15 10 ? * 6L 2012-2015"` | 在 2012, 2013, 2014,2015期间，每月最后一个星期五上午10:15触发 |
| `"0 15 10 ? * 6#3"`          | 每月第三个星期五上午10:15触发执行                            |



##### 构建CronTriggers

`CronTrigger`实例使用`TriggerBuilder`（用于触发器的主要属性）和`CronScheduleBuilder`（对于`CronTrigger`特定的属性）构建。要以`DSL`风格使用这些构建器，请使用静态导入：

```
import static org.quartz.TriggerBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.DateBuilder.*:
```

建立一个触发器，每隔一分钟，每天上午8点至下午5点之间：

```
CronTrigger cronTrigger = newTrigger()
				.withIdentity("trigger3", "triggerGroupName")
			    .withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
			    .forJob("myJobName", "groupName").build();
```

建立一个触发器，将在上午10:42每天发射：

```
CronTrigger cronTrigger = newTrigger()
    .withIdentity("trigger3", "triggerGroupName")
    .withSchedule(dailyAtHourAndMinute(10, 42))
    .forJob(myJobKey)
    .build();
```

或者：

```
CronTrigger cronTrigger = newTrigger()
    .withIdentity("trigger3", "triggerGroupName")
    .withSchedule(cronSchedule("0 42 10 * * ?"))
    .forJob(myJobKey)
    .build();
```

建立一个带时区的触发器，将在星期三上午10:42在TimeZone（系统默认值）之外触发：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "triggerGroupName")
    .withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42))
    .forJob(myJobKey)
    .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
    .build();
```

或者：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "triggerGroupName")
    .withSchedule(cronSchedule("0 42 10 ? * WED"))
    .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
    .forJob(myJobKey)
    .build();
```

##### CronTrigger Misfire说明



##### CronTrigger常见策略：

| 策略名称                                  | 策略说明                                                     |
| ----------------------------------------- | ------------------------------------------------------------ |
| MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY | 忽略所有的超时状态，按照触发器的策略执行。                   |
| MISFIRE_INSTRUCTION_FIRE_ONCE_NOW         | 立刻执行一次，然后就按照正常的计划执行。                     |
| MISFIRE_INSTRUCTION_DO_NOTHING            | 目前不执行，然后就按照正常的计划执行。这意味着如果下次执行时间超过了end time，实际上就没有执行机会了。 |

所有的trigger都有一个Trigger.MISFIRE_INSTRUCTION_SMART_POLICY策略可以使用，该策略也是所有trigger的默认策略。对于`CronTrigger`，该“智能机制”默认选择`MISFIRE_INSTRUCTION_FIRE_ONCE_NOW`以指导其行为。

>TriggerBuilder(以及Quartz的其它builder)会为那些没有被显式设置的属性选择合理的默认值。比如：如果你没有调用*withIdentity(..)*方法，TriggerBuilder会为trigger生成一个随机的名称；如果没有调用*startAt(..)*方法，则默认使用当前时间，即trigger立即生效。



### 四、监听器使用



**Listeners**(监听器)是实现监听接口创建的对象，用于根据调度程序中发生的事件执行操作。

比较常用的监听器： 

#### 1. 触发器监听和任务执行监听

TriggerListeners接收到与触发器（trigger）相关的事件，JobListeners 接收与jobs相关的事件。

与触发相关的事件包括：触发器触发，触发失灵，并触发完成（触发器关闭的作业完成）。

`org.quartz.TriggerListener`接口

```java
public interface TriggerListener {

    public String getName();

    public void triggerFired(Trigger trigger, JobExecutionContext context);

    public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);

    public void triggerMisfired(Trigger trigger);

    public void triggerComplete(Trigger trigger, JobExecutionContext context,
            int triggerInstructionCode);
}
```

job相关事件包括：job即将执行的通知，以及job完成执行时的通知。

org.quartz.JobListener接口

```java
public interface JobListener {

    public String getName();

    public void jobToBeExecuted(JobExecutionContext context);

    public void jobExecutionVetoed(JobExecutionContext context);

    public void jobWasExecuted(JobExecutionContext context,
            JobExecutionException jobException);

}
```

#### 2. 自定义监听器：

要创建一个自定义的listener，只需创建一个实现`org.quartz.TriggerListener`或`org.quartz.JobListener`接口的对象。

如自定义一个触发相关监听器：

```ava
package com.smallrose.web.app.quartz.listener;

import org.quartz.JobExecutionContext;
import org.quartz.Trigger;
import org.quartz.Trigger.CompletedExecutionInstruction;
import org.quartz.TriggerListener;

public class TestTriggerListener implements TriggerListener {

	@Override
	public String getName() {
		return null;
	}

	@Override
	public void triggerFired(Trigger trigger, JobExecutionContext context) {

	}

	@Override
	public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context) {
		return false;
	}

	@Override
	public void triggerMisfired(Trigger trigger) {

	}

	@Override
	public void triggerComplete(Trigger trigger, JobExecutionContext context,
			CompletedExecutionInstruction triggerInstructionCode) {

	}
}
```

如自定义一个任务相关监听器：

```java
package com.smallrose.web.app.quartz.listener;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.quartz.JobListener;

public class TestJobListener implements JobListener {

	@Override
	public String getName() {
		return null;
	}

	@Override
	public void jobToBeExecuted(JobExecutionContext context) {
        
	}

	@Override
	public void jobExecutionVetoed(JobExecutionContext context) {
        
	}

	@Override
	public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException) {
        
	}
}
```

#### 3. 监听器的使用

添加监听使用示例：

```java
//旧版
scheduler.addJobListener(new UpJobListener()); //添加自定义任务执行监听
scheduler.addTriggerListener(new UpTriggerListener()); //添加自定义触发器监听

//新版 since 2.0
scheduler.getListenerManager().addJobListener(new UpJobListener()); //添加任务执行监听
scheduler.getListenerManager().addTriggerListener(new UpJobListener()); //添加任务执行触发器监听
```

移除监听使用示例：

```java
scheduler.removeJobListener(new UpJobListener()); //移除自定义任务执行监听
scheduler.removeTriggerListener(new UpTriggerListener()); //移除自定义触发器监听
```

```
package org.quartz;

import java.util.List;

/**
 * 
 * @author jhouse
 * @since 2.0 - previously listeners were managed directly on the Scheduler interface.
 */
public interface ListenerManager {

    public void addJobListener(JobListener jobListener);

    public void addJobListener(JobListener jobListener, Matcher<JobKey> matcher);

    public void addJobListener(JobListener jobListener, Matcher<JobKey> ... matchers);

    public void addJobListener(JobListener jobListener, List<Matcher<JobKey>> matchers);
 
    public boolean addJobListenerMatcher(String listenerName, Matcher<JobKey> matcher);

    public boolean removeJobListenerMatcher(String listenerName, Matcher<JobKey> matcher);

    public boolean setJobListenerMatchers(String listenerName, List<Matcher<JobKey>> matchers);

    public List<Matcher<JobKey>> getJobListenerMatchers(String listenerName);
 
    public boolean removeJobListener(String name);
 
    public JobListener getJobListener(String name);
 
    public void addTriggerListener(TriggerListener triggerListener);
    
 
    public void addTriggerListener(TriggerListener triggerListener, Matcher<TriggerKey> matcher);
    
   
    public void addTriggerListener(TriggerListener triggerListener, Matcher<TriggerKey> ... matchers);

 
    public void addTriggerListener(TriggerListener triggerListener, List<Matcher<TriggerKey>> matchers);
 
    public boolean addTriggerListenerMatcher(String listenerName, Matcher<TriggerKey> matcher);

 
    public boolean removeTriggerListenerMatcher(String listenerName, Matcher<TriggerKey> matcher);

 
    public boolean setTriggerListenerMatchers(String listenerName, List<Matcher<TriggerKey>> matchers);
 
    public List<Matcher<TriggerKey>> getTriggerListenerMatchers( String listenerName);

 
    public boolean removeTriggerListener(String name);

 
    public List<TriggerListener> getTriggerListeners();
 
    public TriggerListener getTriggerListener(String name);
 
    public void addSchedulerListener(SchedulerListener schedulerListener);
 
    public boolean removeSchedulerListener(SchedulerListener schedulerListener);

 
    public List<SchedulerListener> getSchedulerListeners();

}
```



#### 4. 调度监听器

SchedulerListeners非常类似于TriggerListeners和JobListeners，除了它们在Scheduler本身中接收到事件的通知 ，不一定与特定触发器（trigger）或job相关的事件。

与计划程序相关的事件包括：添加job/触发器，删除job/触发器，调度程序中的严重错误，关闭调度程序的通知等。

`org.quartz.SchedulerListener`接口

```java
public interface SchedulerListener {  

    public void jobScheduled(Trigger trigger);  

    public void jobUnscheduled(String triggerName, String triggerGroup);  

    public void triggerFinalized(Trigger trigger);  

    public void triggersPaused(String triggerName, String triggerGroup);  

    public void triggersResumed(String triggerName, String triggerGroup);  

    public void jobsPaused(String jobName, String jobGroup);  

    public void jobsResumed(String jobName, String jobGroup);  

    public void schedulerError(String msg, SchedulerException cause);  

    public void schedulerStarted();  

    public void schedulerInStandbyMode();  

    public void schedulerShutdown();  

    public void schedulingDataCleared();  
}  
```

调度监听器注册到调度器的监听管理器中，调度监听器实际上可以是实现org.quartz.SchedulerListener接口的任何对象。

添加调度监听器

```java
//旧版
scheduler.addSchedulerListener(new UpSchedulerListener()); //添加自定义调度器监听
//新版 since 2.0
scheduler.getListenerManager().addSchedulerListener(mySchedListener);
```

移除调度监听器

```java
//旧版
scheduler.removeSchedulerListener(new UpSchedulerListener()); //移除自定义调度器监听
//新版 since 2.0
scheduler.getListenerManager().removeSchedulerListener(mySchedListener);
```

### 五、Job Stores

JobStore负责跟踪您提供给调度程序的所有“工作数据”：jobs，triggers，日历等。 

`JobStore`其实就是定时任务的相关数据存储。

支持存储方式有：

（1）RAMJobStore 内存定时数据存储。

（2）JDBC JobStore 通过JDBC将其所有数据保存在数据库中。

（3）TerracottaJobStore TerracottaJobStore用于在集群服务器内存储调度信息（Jobs，Triggers和calendars） 

#### RAMJobStore

RAMJobStore是使用最简单的JobStore，配置简单，性能最高的（在CPU时间方面）。RAMJobStore以其明显的方式获取其名称，它将其所有数据保存在RAM中。缺点是当您的应用程序结束（或崩溃）时，所有调度信息都将丢失  - 这意味着RAMJobStore无法履行作业和triggers上的“非易失性”设置。

使用RAMJobStore ，只需将类名称org.quartz.simpl.RAMJobStore指定为用于配置石英的JobStore类属性：

配置Quartz以使用RAMJobStore

```
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

其实在没有JDBC的情况下，默认就是RAMJobStore。

#### JDBC JobStore

JDBCJobStore几乎支持与任何数据库一起使用，已被广泛应用于Oracle，PostgreSQL，MySQL，MS SQLServer，HSQLDB和DB2。 

数据库中各表的含义：

|表名|表存储信息说明|
|------|--------------------|
|qrtz_blob_triggers　| 这张表示存储自己定义的trigger，不是quartz自己提供的trigger |
|	qrtz_calendars　　| 存储Calendar |
|　　qrtz_cron_triggers　|  存储cronTrigger |
|　　qrtz_fired_triggers　　| 存储触发的Tirgger |
|　　qrtz_job_details　　| 存储JobDetail |
|　　qrtz_locks　　　　| 存储程序中非悲观锁的信息 |
|　　qrtz_paused_trigger_grps　　| 存储已暂停的Trigger组信息 |
|　　qrtz_scheduler_state　　| 存储集群中note实例信息，quartz会定时读取该表的信息判断集群中 每个实例的当前状态 |
|　　qrtz_simple_triggers　　| 存储simpleTrigger的信息|
|　　qrtz_simprop_triggers　　| 储存CalendarIntervalTrigger和DailyTimeIntervalTrigger的信息|
|　　qrtz_triggers　　| 存储已配置的trigger信息|

### 六、其他

后续再补充

<br/>





本文整理参考以下前辈链接，放个链接，以示感谢：

 - 参考文章1：https://blog.csdn.net/chengqiuming/java/article/details/84144534
 - 参考文章2：https://www.cnblogs.com/kyleinjava/p/10432168.html
 - 参考文章3：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/
 - 参考文章4：https://www.w3cschool.cn/quartz_doc/








