---
layout: default
title: Java-Timer
parent: Java TimeTask
grand_parent: Java
nav_order: 10
---



## 定时任务之Java定时任务API

> 本来老早就准备整理的，后面先去整理Spring Task 和 Quartz 的，然后又去搞其他的了。我也有点犹豫要不要整理这个，毕竟现在基本不怎用这个API了。为了知识树的完整性，还是整理一下吧，作为知识面了解一下。而且以前在项目中还使用了。

## 一、定时任务简述

关于调度的实现方式主要有三种：

（1）Java自带的Api有个Timer类和TimerTask类。

（2）Spring自带的Spring Task调度工具。

（3）Quartz（读阔子）开源框架，功能强大，使用起来稍显复杂。

本文主要介绍第二种。

其他两种请参考：《Java定时任务》《Quartz使用》

在开始之前需要说明是一个定时任务的几个组件：

（A）执行的任务类；

（B）触发任务的触发器；

（C）管理任务的任务调度器。

本次整理的是Java自带的任务调度API。

Java中实现定时任务的API，比较古老了，jdk3之后才有的。现在用的非常非常少了。通过`Timer`类实现任务调度，任务执行则可以继承`TimerTask`接口或者直接使用匿名内部类。

基本组件

（1）任务调度管理的`Timer`类，

（2）任务执行接口`TimerTask`接口。

看到这里发现，不是三个常用的吗？怎么没有触发调度，是的，没有专门的触发调度。都是`schedule`里面实现的。

## 二、常见API

`Timer`调度类的：

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `void schedule(TimerTask task , long delay)`                 | 调度一个task，经过delay(ms)后开始进行调度，仅仅调度一次。    |
| `void schedule(TimerTask task, Date firstTime)`              | 在指定的时间点`firstTime`上调度一次。                        |
| `void schedule(TimerTask task, long delay, long period)`     | 调度一个task，在delay（ms）后开始调度，每次调度完后，最少等待period（ms）后才开始调度。 |
| `void schedule(TimerTask task, Date firstTime, long period)` | 调度一个task，在firstTime时第一次调度，调度完后，最少等待period（ms）后才开始调度。 |
| `void scheduleAtFixedRate(TimerTask task, long delay, long period)` | 调度一个task，在delay(ms)后开始调度，然后每经过period(ms)再次调度 。 |
| `void scheduleAtFixedRate(TimerTask task, Date firstTime, long period)` | 调度一个task，在`firstTime`时第一次调度，调度完后，最少等待period（ms）后才开始调度。 |

**主要参数**

`task`：这是被调度的任务。

`firstTime`：这是首次在该任务将被执行。

`delay`：这是以毫秒为单位的延迟之前的任务执行。

`period`：这是在连续执行任务之间的毫秒的时间。 



关于schedule 和 scheduleAtFixedRate 区别。

（1）schedule方法 和 scheduleAtFixedRate 方法，大部分情况都基本一样的。

（2） 如果指定的计划执行时间scheduledExecutionTime<= systemCurrentTime，则task会被立即执行。scheduledExecutionTime不会因为某一个task的过度执行而改变。 

（2）schedule注重保持间隔时间的稳定， 在计算下一次执行的时间的时候，是通过当前时间（在任务执行前得到） + 时间片，而**scheduleAtFixedRate**方法是通过当前需要执行的时间（也就是计算出现在应该执行的时间）+ 时间片。前者是运行的实际时间，而后者是理论时间点。

例如：schedule时间片是5s，那么理论上会在5、10、15、20这些时间片被调度，但是如果由于某些原因导致第8s才被第一次调度，那么schedule方法计算出来的下一次时间应该是第13s而不是第10s，这样有可能下次就越到20s后而被少调度一次或多次。  **scheduleAtFixedRate**方法就是每次理论计算出下一次需要调度的时间用以排序，若第8s被调度，那么计算出应该是第10s，所以它距离当前时间是2s，那么再调度队列排序中，会被优先调度，那么就**尽量减少漏掉调度**的情况。 



## 三、运行原理

`Timer`调度器源码中，主要是一个任务队列，和一个`TimerThread `线程，`TimerThread `继承了`Thread ` 类，用来调度我们需要执行的任务。

```java
/**
     * The timer task queue.  This data structure is shared with the timer
     * thread.  The timer produces tasks, via its various schedule calls,
     * and the timer thread consumes, executing timer tasks as appropriate,
     * and removing them from the queue when they're obsolete.
     */
    private final TaskQueue queue = new TaskQueue();

    /**
     * The timer thread.
     */
    private final TimerThread thread = new TimerThread(queue);

    /**
     * This object causes the timer's task execution thread to exit
     * gracefully when there are no live references to the Timer object and no
     * tasks in the timer queue.  It is used in preference to a finalizer on
     * Timer as such a finalizer would be susceptible to a subclass's
     * finalizer forgetting to call it.
     */
    private final Object threadReaper = new Object() {
        protected void finalize() throws Throwable {
            synchronized(queue) {
                thread.newTasksMayBeScheduled = false;
                queue.notify(); // In case queue is empty.
            }
        }
    };
```

`hreadReaper`，它是Object类型，只是重写了`finalize`方法而已，是为了垃圾回收的时候，将相应的信息回收掉，做GC的回补，也就是当`timer`线程由于某种原因死掉了，而未被`cancel`，里面的队列中的信息需要清空掉， 

`Timer`调度的核心方法是：

```java
private void sched(TimerTask task, long time, long period) {
        if (time < 0)
            throw new IllegalArgumentException("Illegal execution time.");

        // Constrain value of period sufficiently to prevent numeric
        // overflow while still being effectively infinitely large.
        if (Math.abs(period) > (Long.MAX_VALUE >> 1))
            period >>= 1;

        synchronized(queue) {
            if (!thread.newTasksMayBeScheduled)
                throw new IllegalStateException("Timer already cancelled.");

            synchronized(task.lock) {
                if (task.state != TimerTask.VIRGIN)
                    throw new IllegalStateException(
                        "Task already scheduled or cancelled");
                task.nextExecutionTime = time;
                task.period = period;
                task.state = TimerTask.SCHEDULED;
            }

            queue.add(task);
            if (queue.getMin() == task)
                queue.notify();
        }
    }
```

`queue` 在做这个操作的时候，发生了同步，所以在`timer`级别，这个是线程安全的，最后将`task`相关的参数赋值，主要包含`nextExecutionTime`（下一次执行时间），`period`（时间片），`state`（状态），然后将它放入`queue`队列中，做一次`notify`操作 。

`TaskQueue`任务队列中，则是有一个`TimerTask`数组用来存放我需要调度执行的任务，及一个 size 。

```java
private TimerTask[] queue = new TimerTask[128];
private int size = 0;
```

一个`Timer`只要内部的task个数不超过128是不会造成扩容的；内部  提供了add(TimerTask tt)、size()、getMin()、get(int)、removeMin()、quickRemove(int i)、 rescheduleMin(long  newTime)、isEmpty()、clear()、fixUp()、fixDown()、heapify()； 

## 四、实际使用

定义一个任务类：

```java
package xiaocai.cn.timetask;

import java.time.LocalDateTime;
import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;

public class DemoTask extends TimerTask{

	@Override
	public void run() {
		 try{
			 System.out.println(" execute time : " +LocalDateTime.now());
			 Thread.sleep(6000);
		 }catch(Exception e){
		 }
	}

	public static void main(String[] args) {
 
		Timer timer = new Timer();
		Date startDate = new Date();
		timer.schedule(new DemoTask(), startDate , 5 * 1000);
        // 立即执行，每5s重复一次
	}
}
```

` Thread.sleep(3000);` 时执行结果： 

```txt
execute end time - 2020-07-18T20:00:39.836
execute end time - 2020-07-18T20:00:44.817
execute end time - 2020-07-18T20:00:49.817
execute end time - 2020-07-18T20:00:54.816
execute end time - 2020-07-18T20:00:59.817
```

` Thread.sleep(10000);` 时执行结果：

```txt
 execute time : 2020-07-18T18:18:47.416
 execute time : 2020-07-18T18:18:57.417
 execute time : 2020-07-18T18:19:07.418
 execute time : 2020-07-18T18:19:17.419
 execute time : 2020-07-18T18:19:27.419
 execute time : 2020-07-18T18:19:37.420
```

方法模拟的执行时间是3秒， 设置的间隔参数是5秒。调度是每5秒执行一次，按照自己设定的间隔来执行。

方法模拟的执行时间是10秒， 可是我们设置的间隔是5秒。调度是每10秒执行一次。在第一次执行之后，后续间隔仍然是10s，没有在18分52秒、19分02秒时执行。 可见`schedule`注重调度周期稳定。

也可以配合日历使用

```java
public static void main(String[] args) {
 
	Timer timer = new Timer();
    DemoTask demo = new DemoTask()
	Date startDate = new Date();
    // 立即执行，每5s重复一次
	timer.schedule(demo, startDate , 5 * 1000);
        
    // 配合日历
    Calendar cal = Calendar.getInstance();
	cal.set(Calendar.MINUTE, 30);
	timer.schedule(demo, cal.getTime() , 2000);
}
```

再试试`scheduleAtFixedRate`方法，修改一下`main`方法:

```java
package xiaocai.cn.timetask;

import java.time.LocalDateTime;
import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;

public class DemoTask extends TimerTask{

	@Override
	public void run() {
		 try{	 
			 Thread.sleep(6000);
		 }catch(Exception e){
		 }
        System.out.println(" execute time : " +LocalDateTime.now());
	}
    
    public static void main(String[] args) throws ParseException {
        final SimpleDateFormat dateFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        Date startDate = dateFormatter.parse("2020-07-18 18:30:00");  

        Timer timer = new Timer();

        timer.scheduleAtFixedRate(new DemoTask(),startDate , 5 * 1000);
	}
 }
```

`Thread.sleep(3000);` 时执行结果： 

```txt
 execute end time - 2020-07-18T20:06:37.458
 execute end time - 2020-07-18T20:06:40.458
 execute end time - 2020-07-18T20:06:43.459
 execute end time - 2020-07-18T20:06:46.459
 execute end time - 2020-07-18T20:06:49.460
 execute end time - 2020-07-18T20:06:52.460
```

`Thread.sleep(10000);` 时执行结果： 

```txt
 execute end time - 2020-07-18T20:09:14.068
 execute end time - 2020-07-18T20:09:24.069
 execute end time - 2020-07-18T20:09:34.069
 execute end time - 2020-07-18T20:09:44.070
 execute end time - 2020-07-18T20:09:54.071
 execute end time - 2020-07-18T20:10:04.071
```

上面的运行结果可以看到，当模拟执行时间10s大于设置间隔时间5s时，按照实际执行间隔立即执行。当模拟执行时间3s小于设置间隔时间5s时，没有等到5s就执行了，没有按照自己设定的间隔执行。难道是我把计划执行时间设置成历史时间的原因？将

```java
Date startDate = dateFormatter.parse("2020-07-18 18:30:00");
```

修改成未来时间

```java
Date startDate = dateFormatter.parse("2020-07-18 20:16:00");
```

执行结果如下：

```txt
 execute end time - 2020-07-18T20:16:03.052
 execute end time - 2020-07-18T20:16:08
 execute end time - 2020-07-18T20:16:13.001
 execute end time - 2020-07-18T20:16:18.001
 execute end time - 2020-07-18T20:16:23.001
 execute end time - 2020-07-18T20:16:28
 execute end time - 2020-07-18T20:16:33.001
 execute end time - 2020-07-18T20:16:38.001
```

果然是，计划执行时间是历史时间时，设定的间隔会失效。计划执行时间是未来时间时，间隔生效了。

<br/>

**相关文章**

[定时任务之Java定时API](a516f8e8.html)
[定时任务之Spring Task](5a5ab620.html)
[定时任务之quartz](6ddb3c14.html)


<br/>
