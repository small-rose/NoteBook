---
layout: default
title: Spring Batch
parent: Spring
nav_order: 13
---

## spring-batch 


> 分类说明，Spring batch 属于sping框架，而非springboot，但因为现在多使用springboot开发，并且本文案例也是基于springboot的，所以就放到SpringBoot类目下。


> Spring Batch是处理大量数据操作的一个框架，主要用来读取大量数据，然后进行一定的处理后，按照指定的形式回写或者输出。



任务存储仓库可以是关系型数据库MySQL，非关系型数据库MongoDB或者直接存储在内存中

## 一、基本原理

### 1、主要原理

![spring-batch 基本原理图](https://notes.zhangxiaocai.cn/images/spring-batch/spring-batch-01.png)

Spring Batch里最基本的单元就是任务Job，一个Job由若干个步骤Step组成。任务启动器Job Launcher负责运行Job，任务存储仓库Job Repository存储着Job的执行状态，参数和日志等信息。

Job处理任务又可以分为三大类：数据读取Item Reader、数据中间处理Item Processor和数据输出Item Writer。

### 2、 @EnableBatchProcessing

开启批量处理的模块化注册组件，配置的核心接口是BatchConfigurer。开启之后默认装载一下Bean组件：

- `JobRepository`：bean名称 `jobRepository`

- `JobLauncher`：bean名称 `jobLauncher`

- `JobRegistry`：bean名称 `jobRegistry`

- `PlatformTransactionManager`：bean名称 `transactionManager`

- `JobBuilderFactory`：bean名称`jobBuilders`

- `StepBuilderFactory`：bean名称`stepBuilders`

开启之后，可以使用提供的构建器工厂来配置作业。

```java
@Configuration
@EnableBatchProcessing
@Import(DataSourceConfiguration.class)
public class AppConfig {

    @Autowired
    private JobBuilderFactory jobs;

    @Autowired
    private StepBuilderFactory steps;

    @Bean
    public Job job(@Qualifier("step1") Step step1, @Qualifier("step2") Step step2) {
        return jobs.get("myJob").start(step1).next(step2).build();
    }

    @Bean
    protected Step step1(ItemReader<Person> reader,
                         ItemProcessor<Person, Person> processor,
                         ItemWriter<Person> writer) {
        return steps.get("step1")
            .<Person, Person> chunk(10)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
    }

    @Bean
    protected Step step2(Tasklet tasklet) {
        return steps.get("step2")
            .tasklet(tasklet)
            .build();
    }
}
```

## 二、核心组件

Spring Batch框架7个主要组件

- 1）**Job**： 即作业，也就是在使用过程中真处理业务数据的批处理作业。 Job 是封装整个批处理过程的实体。

- 2）**Step**：即步骤，包括：ItemReader->ItemProcessor->ItemWriter

- 3）JobLauncher：用来启动Job的接口

- 4）JobRepository：用来注册Job容器，设置数据库相关属性。

- 5）ItemReader：用来读取数据，做实体类与数据字段之间的映射。比如读取csv文件中的人员数据，之后对应实体person的字段做mapper

- 6）ItemProcessor：用来处理数据的接口，同时可以做数据校验（设置校验器，使用JSR-303(hibernate-validator)注解），比如将中文性别男/女，转为M/F。同时校验年龄字段是否符合要求等

- 7）ItemWriter：用来输出数据的接口，设置数据库源。编写预处理SQL插入语句

以上七个组成部分中，Job和Step是spring batch执行批处理任务最为核心的两个概念。

### 1、Job 作业组件

job是作为运行的基本单位，它内部由step组成。job本质上可以看成step的一个容器。
一个job可以按照指定的逻辑顺序组合step，并提供了我们给所有step设置相同属性的方法，例如一些事件监听，跳过策略等。
Job实现类主要有两种类型的job，一个是simplejob，另一个是flowjob。

![](/images/spring-batch/job-code.jpg)

在Spring Batch中，作业Job只是Step实例的容器。它组合了逻辑上属于流程的多个Step步骤，并允许配置所有步骤全局的属性，例如可重新启动性。作业配置包含：

- 作业的简单名称。

- 步骤实例的定义和顺序。

- 作业是否可重新启动。

关于Job作业的配置称为作业配置。作业配置常见模式：

**Java 模式的作业配置：**

```java
    @Bean
    public Job multiStepJob() {
        return jobBuilderFactory.get("multiStepJob")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }
```

**XML 模式的作业配置：**

```xml
<job id="multiStepJob">
    <step id="step1" next="step2"/>
    <step id="step2" next="step3"/>
    <step id="step3"/>
</job>
```

#### 1.1、JobInstance 作业实例

![](/images/spring-batch/job-heirarchy.png)

JobInstance是Job的更加底层的一个抽象，是指逻辑作业运行的概念。

> 比如现在有一个在每天结束时运行一次的批处理作业，例如上图中的 "EndOfDay" 作业。有一个"EndOfDay"作业，但是该作业的每个单独运行都必须分别进行跟踪。对于此作业，每天有一个逻辑JobInstance。例如，有1月1日运行，1月2日运行，依此类推。如果1月1日运行第一次失败并在第二天再次运行，则仍是1月1日运行。 （通常，这也与它正在处理的数据相对应，这意味着1月1日运行处理1月1日的数据）。因此，每个JobInstance可以有多个执行，并且在给定时间只能运行一个与特定Job对应并标识JobParameters的JobInstance。

官方的说明通俗的理解其实就是，我虽然定义了一个Job作业（相当于作业模板），但是会多次执行，每一次执行都要单独跟踪记录，不然怎么知道每次的执行情况呢？所以每次执行都会产生一个`JobInstance`，可以理解为运行记录，这个记录对应着 `batch_job_instance` 表（默认batch开头，可以自定义配置）中对应Job的记录。

JobInstance 作业实例接口，有两个方法，一个是返回Job作业实例的id，另一个是返回Job作业名称。

```java
public interface JobInstance {
    long getInstanceId();

    String getJobName();
}
```

#### 1.2、JobParameters

JobParameters 即作业运行参数。

同一个Job作业，每天运行一次的话，那么每天都有一个jobIntsance作业实例记录，但他们的Job作业模板定义都是一样的，那么就需要一个用区别一个job作业模板执行时的不同jobinstance的标记，spring batch中提供的用来标识一个jobinstance的东西就是JobParameters 。

JobParameters对象包含一组用于启动批处理作业的参数，它可以在运行期间用于识别或甚至用作参考数据。通常运行时间，就可以作为一个JobParameters。

![](/images/spring-batch/job-stereotypes-parameters.png)

啊哦！原来是这样，不过，我偏不信邪，我就不用运行时间作为参数标记，我偏要设置一个固定的 JobParameters。

先定义一个简单的Job作业执行模板：

```java
package cn.xiaocai.batch.single;
 
/**
 * import 省略
 * @author Xiaocai.Zhang
 */
@Slf4j
@Component
@AllArgsConstructor
public class SingleJobSingleStepDemo {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    /**
     *  以Bean的形式放入ioc，本实例仅作demo使用，实际请放到单独的config中，其他同理，不再单独说明
     * @return
     */
    @Bean
    public Job singleStepJobDemo() {
        return jobBuilderFactory.get("singleStepJobDemo")
                .start(singleStep())
                .build();
    }

    private Step singleStep() {
        return stepBuilderFactory.get("step")
                .tasklet((contribution, chunkContext) -> {
                    log.info("----this is my first job ....");
                    return RepeatStatus.FINISHED;
                }).build();
    }
}
```
为了不让这个Job启动工程就执行，先将默认的工程启动立即执行`spring.batch.job.enabled`配置设置为`false`

```yml
server:
  port: 8806
#-------------------
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboot_batch?serverTimezone=GMT%2b8&useSSL=true&Unicode=true&characterEncoding=utf8&autoReconnectForPools=true&allowMultiQueries=true&rewriteBatchedStatements=true
    username: root
    password: 123456

# 默认是启动时执行一次，设置为false，表示启动不执行，可以通过 web等其他请求调用方式执行
  batch:
    job:
      enabled: false  #是否自动执行定义的Job，默认是
      names:  # 启动是要执行的job
    initialize-schema: never # 是否初始化Spring Batch的数据库，ALWAYS,EMBEDDED,NEVER
    schema:    # 默认有对应的sql脚本
    table-prefix: batch_  # 设置Spring Batch的数据库表的前缀

#-------------------
scheduled:
  enabled: false

```

为了模拟真实场景测试，我写个`controller`来模拟真实调用：

```java
    @ApiOperation("不变参数重复调用测试接口")
    @GetMapping("/single/noParameters")
    public String noParameters() throws Exception {
        JobParameters jobParameters = new JobParametersBuilder().addString("testKey","testValue")
                .toJobParameters();
        jobLauncher.run(singleStepJobDemo, jobParameters);
        return "The noParameters job is proceed.";
    }
```

我这里偏要给一个永远固定的 JobParameters，看他又能把我咋样？
```java
 JobParameters jobParameters = new JobParametersBuilder().addString("testKey","testValue")
                .toJobParameters();
```

启动工程访问测试API: `http://localhost:8806/job/single/noParameters`

第1次执行结果：
```txt
2020-12-09 15:52:48.105  INFO 15500 --- [nio-8806-exec-8] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=singleStepJobDemo]] launched with the following parameters: [{testKey=testValue}]
2020-12-09 15:52:48.122  INFO 15500 --- [nio-8806-exec-8] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step]
2020-12-09 15:52:48.131  INFO 15500 --- [nio-8806-exec-8] c.x.b.single.SingleJobSingleStepDemo     : ----this is my first job ....
2020-12-09 15:52:48.138  INFO 15500 --- [nio-8806-exec-8] o.s.batch.core.step.AbstractStep         : Step: [step] executed in 16ms
```
顺利执行了。

第2次执行结果：

```txt
2020-12-09 15:52:56.200 ERROR 15500 --- [nio-8806-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException: A job instance already exists and is complete for parameters={testKey=testValue}.  If you want to run this job again, change the parameters.] with root cause

org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException: A job instance already exists and is complete for parameters={testKey=testValue}.  If you want to run this job again, change the parameters.
    at org.springframework.batch.core.repository.support.SimpleJobRepository.createJobExecution(SimpleJobRepository.java:132) ~[spring-batch-core-4.3.0.jar:4.3.0]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_212]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_212]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_212]
    at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_212]
    at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344) ~[spring-aop-5.3.1.jar:5.3.1]
```
可以看到 `SimpleJobRepository`的`createJobExecution(String jobName, JobParameters jobParameters)`方法会抛出错误信息：`JobInstanceAlreadyCompleteException: A job instance already exists and is complete for parameters={testKey=testValue}.  If you want to run this job again, change the parameters.`

**结论**
（1）JobInstance = Job + jobParameters 标识。

（2）jobParameters 作业参数 搭配 Job 作业模板执行才能产生一个 JobInstance 。 

（3）而 Job 作业模板基本上一开始就定义好的，所以可以通过Jobparameter来操作正确的JobInstance，在通俗的意义上理解将Jobparameter 作业参数 和JobInstance 作业实例看成一一对应的。

> 注意：并非所有作业参数都需要用于标识JobInstance。默认情况下，会使用JobParameters来标识JobInstance。但是，spring batch 框架还是允许提交带有不影响JobInstance标识的参数的作业。 


#### 1.3、JobExecution 作业执行

JobExecution 作业执行是指一次尝试运行作业的技术概念。执行的最终结果可能是失败，也可能是成功，但与给定执行相对应的JobInstance不会被视为完成，除非作业执行成功完成。

以前面描述的EndOfDay作业为例，考虑01-01-2017的JobInstance在第一次运行时失败。如果使用与第一次运行（01-01-2017）相同的标识作业参数再次运行，则会创建一个新的JobExecution。但是，仍然只有一个JobInstance。

Job定义了什么是作业及其执行方式，执行模板。

JobInstance是一个纯粹的组织对象，用于将执行分组在一起，主要是为了实现正确的重启语义。

JobExecution是运行期间实际发生情况的主要存储机制，它包含许多必须控制和持久化的属性。

JobExecution 的重要属性

| 属性   | 含义 |
|--------|--------|
| Status   |  BatchStatus对象，指示执行的状态。在运行时，它是 BatchStatus#STARTED。如果失败，则为BatchStatus#FAILED。如果成功完成，那就是BatchStatus#COMPLETED      |
| startTime   | 一个java.util.Date代表当执行开始时的当前系统时间。如果作业尚未开始，则此字段为空。     |
| endTime     | 一个java.util.Date代表当执行完成后，无论它是否是成功的当前系统时间。如果作业尚未完成，则该字段为空。|
| exitStatus  | ExitStatus，说明运行的结果。这是最重要的，因为它包含返回给调用方的退出代码。 如果作业尚未完成，则该字段为空。 |
| createTime   | java.util.Date表示当当前系统时间JobExecution最早持续。作业可能尚未启动（因此没有启动时间），但是它始终具有createTime，这是管理作业级别的框架所需的 ExecutionContexts。 |
| lastUpdated   | java.util.Date 代表上一次JobExecution持续存在的时间。如果作业尚未开始，则此字段为空。 |
| executionContext   |  执行上下文 包含在两次执行之间需要保留的所有用户数据。|
| failureExceptions   |  执行Job。时遇到的异常列表。如果在失败时遇到多个异常，这些功能将非常有用Job。|




### 2、Step 作业组件

Step 即作业执行步骤。Step是一个域对象，每一个Step对象都封装了批处理作业的一个独立的或者连续的阶段。因此，每个Job都由一个或多个Step步骤组成。Step步骤包含定义和控制实际批处理所需的所有信息。与Job作业一样，Step步骤具有单独的StepExecution，与唯一的JobExecution相关。

> 任何特定的内容Step都是由编写Job的开发人员自行决定。 一个step可以非常简单也可以非常复杂。 例如，一个step的功能是将文件中的数据加载到数据库中，那么基于现在spring batch的支持则几乎不需要写代码。 更复杂的step可能具有复杂的业务逻辑，这些逻辑作为Step处理的一部分。

![](/images/spring-batch/jobHeirarchyWithSteps.png)

#### 2.1、StepExecution
StepExecution表示一次执行Step的尝试, 每次运行一个Step时都会创建一个新的StepExecution，类似于JobExecution。 但是，某个步骤可能由于其之前的步骤失败而无法执行。 只有在Step实际启动时才会创建StepExecution。

一次step执行的实例由StepExecution类的对象表示。 每个StepExecution都包含对其相应步骤的引用以及JobExecution和事务相关的数据，例如提交和回滚计数以及开始和结束时间。 此外，每个步骤执行都包含一个ExecutionContext，其中包含开发人员需要在批处理运行中保留的任何数据，例如重新启动所需的统计信息或状态信息。


StepExecution 的重要属性

| 属性 | 含义 |
|--------|--------|
| Status | BatchStatus对象，指示执行的状态。在运行时，状态为BatchStatus.STARTED。如果失败，则状态为BatchStatus.FAILED。如果成功完成，则状态为BatchStatus.COMPLETED。       |
| startTime | 一个java.util.Date代表当执行开始时的当前系统时间。如果步骤尚未开始，则此字段为空。       |
| endTime       | 一个java.util.Date代表当执行完成后，无论它是否是成功的当前系统时间。如果步骤尚未退出，则此字段为空。      |
| exitStatus    | ExitStatus指示执行的结果。这是最重要的，因为它包含返回给调用方的退出代码。如果作业尚未退出，则此字段为空。      |
| executionContext | 执行上下文 包含在两次执行之间需要保留的所有用户数据。       |
| readCount   |   已成功读取的条目数。     |
| writeCount      | 已成功写入的条目数。       |
| commitCount     | 为此执行已提交的事务数    |
| rollbackCount   | 由Step所控制的业务交易已回滚的次数。       |
| readSkipCount   | read失败次数导致条目被跳过。       |
| processSkipCount| process失败次数导致条目被跳过。       |
| filterCount     | 已被ItemProcessor"过滤"的条目数。  |
| writeSkipCount  | write失败次数导致条目被跳过。       |


### 3、ExecutionContext 执行上下文

ExecutionContext表示键-值对的集合，这些键-值对由框架进行持久化和控制，以便允许开发人员在一个地方存储持久状态，该持久状态的范围仅限于StepExecution对象或JobExecution对象。

官方说这个东东和 Quartz 的 JobDataMap 非常相似。

扯一堆术语，说白了就是有个可以用来存放执行过程中的记录数据、环境数据的Map集合，而这个Map集合会在你每次提交数据的时候，包存在数据库`BATCH_STEP_EXECUTION_CONTEXT` 表里面。万一执行出错或者中断，下次启动执行的时候还可以接着用。

最佳用法示例是促进重新启动。以将文件内容加载到数据库为例，在处理各个行时，框架会定期在提交点持久保存ExecutionContext。这样做可以让ItemReader存储其状态，以防在运行过程中发生致命错误，甚至断电。 需要做的就是将当前读取的行数放入上下文中：

比如在某个step中，记录下当前读取的行数。

```java
ExecutionContext ecStep = stepExecution.getExecutionContext();
ecStep.putLong(getKey(LINES_READ_COUNT), reader.getPosition());
```
获取上下文

可以根据StepExecution对象或JobExecution对象获取对应的执行上下文：

```java
ExecutionContext ecStep = stepExecution.getExecutionContext();
ecStep.gutLong(getKey(LINES_READ_COUNT), reader.getPosition());

ExecutionContext ecJob = jobExecution.getExecutionContext();
ecJob.gutLong(getKey(LINES_READ_COUNT), reader.getPosition());
```
ecStep 和 ecJob 是两个不同的ExecutionContext。范围为step的step执行上下文环境的一个保存在step中的每个提交点，而范围为job的一个上下文保存在每个step中。


### 4、JobLauncher 作业启动器

```java
public interface JobLauncher {

public JobExecution run(Job job, JobParameters jobParameters)
            throws JobExecutionAlreadyRunningException, JobRestartException,
                   JobInstanceAlreadyCompleteException, JobParametersInvalidException;
}
```
执行Job时指定了JobParameters和对应的Job，并返回对应的JobExecution。


### 5、JobRepository 作业仓库

JobRepository是上述所有构造型的持久性机制。它为JobLauncher，Job和Step实现提供CRUD操作。首次启动Job时，会从存储库中获取JobExecution，并在执行过程中将StepExecution和JobExecution实现传递到存储库中，以实现持久性。


Java配置当使用`@EnableBatchProcessing`注解，spring batch 就会自动提供一个 `JobRepository`作为开箱即用自动配置的组件之一。




### 6、ItemReader 条目

ItemReader是一个读数据的抽象，它的功能是为每一个Step提供数据输入。 当ItemReader以及读完所有数据时，它会返回null来告诉后续操作数据已经读完。Spring Batch为ItemReader提供了非常多的有用的实现类。

![](/images/spring-batch/all-itemReader.png)

ItemReader支持的读入的数据源也是非常丰富的，包括各种类型的数据库，文件，数据流，消息中间件等等。几乎涵盖了我们的所有场景。

#### 6.1 内置ItemReader

- JdbcPagingItemReader：jdbc分页从数据库读取数据
- HibernateItemReader：使用Hibernate读取数据库
- JpaPagingItemReader: jpa 分页读
- StaxEventItemReader：从XML文件中读取数据
- FlatFileItemReader：从CVS表格文件中读取数据
- MultiResourceItemReader：从多个文件读取数据
- ListItemReader：从List读取数据
- JsonItemReader：从Josn读取数据
- GsonJsonObjectItemReader：从Josn读取数据
- JasksonJsonObjectItemReader：从Josn读取数据
- MongoItemReader：从MongoDB数据库读取数据
- JmsItemReader：从Jms消息读取数据
- AmqpItemReader：从mq 读取数据
- KafkaItemReader：从Kafka读取数据
- Neo4jItemReader：从图形数据库读取数据


相关文档：[关于ItemReader的文档](https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/index-single.html#readersAndWriters)

#### 6.2 ItemReader 示例

简单的读数据库数据示例:

```java
/**
     * 从数据库读取数据
     * @return
     * @throws Exception
     */
    public ItemReader<UserVO> dataSourceItemReader() throws Exception {
        JdbcPagingItemReader<UserVO> reader = new JdbcPagingItemReader<>();
        // 设置数据源
        reader.setDataSource(dataSource);
        // 每次取多少条记录
        reader.setFetchSize(5);
        // 设置每页数据量
        reader.setPageSize(5);

        // 指定sql查询语句 select id,name,age,gender from user
        MySqlPagingQueryProvider provider = new MySqlPagingQueryProvider();
        provider.setSelectClause("id,name,age,gender"); //设置查询字段
        provider.setFromClause("from user"); // 设置从哪张表查询

        // 将读取到的数据转换为UserVO对象
        reader.setRowMapper((resultSet, rowNum) -> {
            UserVO data = new UserVO();
            data.setId(resultSet.getString(1));
            data.setName(resultSet.getString(2)); // 读取第一个字段，类型为String
            data.setAge(resultSet.getInt(3));
            data.setGender(resultSet.getString(4));
            return data;
        });

        Map<String, Order> sort = new HashMap<>(1);
        sort.put("id", Order.ASCENDING);
        provider.setSortKeys(sort); // 设置排序,通过id 升序

        reader.setQueryProvider(provider);

        // 设置namedParameterJdbcTemplate等属性
        reader.afterPropertiesSet();
        return reader;
    }
```

也可以定制自己的ItemReader，只要实现 `ItemReader<T>` 接口即可：

```java
package org.springframework.batch.item;

import org.springframework.lang.Nullable;

public interface ItemReader<T> {
    @Nullable
    T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;
}
```
示例：

```java
package cn.xiaocai.batch.itemrw.reader;
 
import org.springframework.batch.item.ItemReader;
import java.util.Iterator;
import java.util.List;

/**
 *  自己定义 自己的 ItemReader
 * @author Xiaocai.Zhang
 */
public class SimpleDemoItemReader implements ItemReader<String> {

    private Iterator<String> iterator;

    public SimpleDemoItemReader(List<String> data) {
        this.iterator = data.iterator();
    }

    @Override
    public String read()  {
        return iterator.hasNext() ? iterator.next() : null;
    }
}

```




### 7、ItemProcessor 条目处理器

ItemProcessor是表示条目业务处理的抽象。

当ItemReader读取一个条目，而ItemWriter写入一个条目时，ItemProcessor提供了一个访问点来转换或应用其他业务处理。


#### 7.1 内置处理器Processor

- FunctionItemProcessor

- ClassifierCompositeItemProcessor

- CompositeItemProcessor

- PassThroughItemProcessor

- ScriptItemProcessor

- BeanValidatingItemProcessor

- ValidatingItemProcessor



#### 7.2 自定义处理器Processor

**实现方式一：完全的自定义。实现 `ItemProcessor` 接口**

如果在处理项目时确定该条目无效，则返回null表示不应将该项目写出。

```java
package cn.xiaocai.batch.itemrw.validator;

import cn.xiaocai.batch.VO.UserVO;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.util.StringUtils;

/**
 * @author Xiaocai.Zhang
 */
public class DataFilterItemProcessor implements ItemProcessor<UserVO, UserVO> {

    @Override
    public UserVO process(UserVO userVO) throws Exception {
        // 返回null，会过滤掉这条数据,条件是ID为空或者
        return ( !StringUtils.hasLength(userVO.getId()) || !StringUtils.hasLength(userVO.getName())) ? null : userVO;
    }
}
```

**实现方式二：自定义继承已有内置的`ItemProcessor`类**


定义一个导入用户数据的处理，要将男女转换成1和2。


```java
package cn.xiaocai.batch.itemrw.processor;

import cn.xiaocai.batch.VO.UserVO;
import org.springframework.batch.item.validator.ValidatingItemProcessor;
import org.springframework.batch.item.validator.ValidationException;
import org.springframework.stereotype.Component;

/**
 * @author Xiaocai.Zhang
 */
@Component
public class CsvItemProcessor extends ValidatingItemProcessor<UserVO> {

    @Override
    public UserVO process(UserVO item) throws ValidationException {
        // 需执行super.proces:(item)才会调用自定义校验器。
        super.process(item);
        // 对数据做简单的处理，若性别为男，则数据转换成1，其余转换成2.
        if(item.getGender().equals("男")) {
            item.setGender("1");
        }else {
            item.setGender("2");
        }
        return item;
    }
}
```


### 8、ItemWriter 条目写

与ItemReader相对应的，ItemWriter就是一个写数据的抽象，它是为每一个step提供数据写出的功能。写的单位是可以配置的，我们可以一次写一条数据，也可以一次写一个chunk的数据。ItemWriter对于读入的数据是不能做任何操作的。


Spring Batch为ItemWriter也提供了非常多的有用的实现类，当然我们也可以去实现自己的writer功能。

![](/images/spring-batch/all-itemWriter.png)

跟ItemReader一样，ItemWriter支持的写入入的数据源也是非常丰富的，包括各种类型的数据库，文件，数据流，消息中间件等等。几乎涵盖了我们的所有场景。

#### 7.1 内置ItemWriter

- SimpleMailMessageItemWriter、MimeMessageItemWriter：写邮件相关
- JdbcBatchItemWriter：jdbc批量写入数据库
- HibernateItemWriter：使用Hibernate写入数据库
- JpaItemWriter: jpa 写入数据库
- StaxEventItemWriter：写入XML文件
- FlatFileItemWriter：写入CVS文件
- MultiResourceItemWriter：写入多个文件
- ListItemWriter：写入List数据
- JsonFileItemWriter：写入Josn文件
- JmsItemWriter：Jms消息写入
- AmqpItemWriter：写入标准应用消息队列
- KafkaItemWriter：写入Kafka数据
- MongoItemWriter：写入MongoDB数据库
- Neo4jItemWriter：写入图形数据库读
- AvroItemWriter：序列化相关读取数据

相关文档：[关于ItemWriter的文档](https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/index-single.html#readersAndWriters)

#### 7.2 ItemWriter 的示例

简单的写文件示例：

```java
    /**
     *  写文件，写成字符串
     * @return
     * @throws Exception
     */
    public FlatFileItemWriter<UserVO> fileItemWriter() throws Exception {
        FlatFileItemWriter<UserVO> writer = new FlatFileItemWriter<>();

        FileSystemResource file = new FileSystemResource("E:/spring-batch/user_info");
        Path path = Paths.get(file.getPath());
        if (!Files.exists(path)) {
            Files.createFile(path);
        }
        writer.setResource(file); // 设置目标文件路径

        // 把读到的每个UserVO对象转换为JSON字符串
        LineAggregator<UserVO> aggregator = item -> {
            try {
                // 这里可以自定义拼接规则
                ObjectMapper mapper = new ObjectMapper();
                return mapper.writeValueAsString(item);
            } catch (JsonProcessingException e) {
                e.printStackTrace();
            }
            return "";
        };

        writer.setLineAggregator(aggregator);
        writer.afterPropertiesSet();
        return writer;
    }
```

### 9、补充 

#### （1）chunk

一次batch的任务可能会有很多的数据读写操作，因此一条一条的处理并向数据库提交的话效率不会很高，因此spring batch提供了chunk这个概念，我们可以设定一个chunk size，spring batch 将一条一条梳理处理之后，先不提交到数据库，只有当处理的数据数量达到chunk size设定的值得时候，才一起去commit到数据库。

![](/images/spring-batch/step-chunk.png)

#### （2）skip


```java
    private Step rwStep() throws Exception {
        return stepBuilderFactory.get("step")
                .listener(myJobListener)
                .<UserVO,UserVO>chunk(2)
                .reader(fileReader())
                .processor(processor())
                .writer(dataSourceItemWriter())
                .faultTolerant()
                .skipLimit(1)
                .skip(Exception.class)
                .noSkip(FileNotFoundException.class)
                .build();
    }
```

skipLimit方法的意思是我们可以设定一个我们允许的这个step可以跳过的异常数量，假如我们设定为10，则当这个step运行时，只要出现的异常数目不超过10，整个step都不会fail。注意，若不设定skipLimit，则其默认值是0.

skip方法我们可以指定我们可以跳过的异常，因为有些异常的出现，我们是可以忽略的。

noSkip方法的意思则是指出现这个异常我们不想跳过，也就是从skip的所以exception当中排除这个exception，从上面的例子来说，也就是跳过所有除FileNotFoundException的exception。那么对于这个step来说，FileNotFoundException就是一个fatal的exception，抛出这个exception的时候step就会直接fail


## 三、重要API

#### 1、数据上下文

- ExecutionContext 执行上下文。 本质是个Map，可以持久化到数据库，持久状态的范围仅限于StepExecution对象或JobExecution对象。

- JobContext：可以获取 JobExecution、存取用户临时数据、 Job实例、Job执行实例、退出状态、批量处理状态、异常等。

- JobExecution：可以通过 JobContext 获取。可以获取 JobExecutionContext 作业执行上下文对象 和 JobParameters 作业参数对象。

- StepContext：可以获取 StepExecution、存取用户临时数据、 持久化用户数据、步骤执行实例、退出状态、批量处理状态、异常等。

- StepExecution：可以通过 StepContext 获取。可以获取 JobExecution 作业执行对象、StepExecutionContext 作业执行上下文对象。

- ChunkContext： 可以获取 StepContext上下文。


### 2、监听器

- JobExecutionListener 作业执行监听器。实现方式2种：
  - 实现 JobExecutionListener 接口，重写`beforeJob()` 和 `afterJob()` 方法。
  - 使用`@BeforeJob` 和 `@AfterJob` 修饰对应的方法。

- StepExecutionListener 步骤执行监听器。实现方式2种：
  - 实现 StepExecutionListener 接口，重写`beforeStep()` 和 `afterStep()` 方法。
  - 使用`@BeforeStep` 和 `@AfterStep` 修饰对应的方法。

- ItemReadListener 读数据项监听器。实现方式2种：
  - 实现 ItemReadListener 接口，重写`beforeRead()` 和 `afterRead()` 以及 `onReadError()` 方法。
  - 使用`@BeforeRead`、 `@AfterRead`、`@OnReadError` 修饰对应的方法。


- ItemWriteListener 写数据项监听器。实现方式2种：
  - 实现 ItemWriteListener 接口，重写`beforeWrite()` 和 `afterWrite()` 以及 `onWriteError()` 方法。
  - 使用`@BeforeWrite`、 `@AfterWrite`、`@OnWriteError` 修饰对应的方法。


- ItemProcessListener 处理数据项监听器。实现方式2种：
  - 实现 ItemProcessListener 接口，重写`beforeProcess()` 和 `afterProcess()`以及 `onProcessError()` 方法。
  - 使用`@BeforeProcess`、 `@AfterProcess`、`@OnProcessError` 修饰对应的方法。

- ChunkListener 处理数据块项监听器。实现方式2种：
  - 实现 ChunkListener 接口，重写`beforeChunk()` 和 `afterChunk()`以及 `AfterChunkError()` 方法。
  - 使用`@BeforeChunk`、 `@AfterChunk`、`@AfterChunkError` 修饰对应的方法。

- SkipListener 跳过数据的监听器。实现方式2种：
  - 实现 SkipListener 接口，重写`onSkipInRead()` 和 `onSkipInWrite()`以及 `onSkipInProcess()` 方法。
  - 使用`@OnSkipInRead`、 `@OnSkipInWrite`、`@OnSkipInProcess` 修饰对应的方法。


实例：

```java
package cn.xiaocai.batch.listener;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.BatchStatus;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobExecutionListener;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.annotation.BeforeJob;
import org.springframework.batch.core.scope.context.JobContext;
import org.springframework.batch.core.scope.context.StepContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.time.Duration;
import java.time.LocalDateTime;

/**
 * 自定义 Job作业执行 的监听器
 * @author Xiaocai.Zhang
 */
@Slf4j
@Component
public class MyJobExecutionListener implements JobExecutionListener {

    private LocalDateTime startTime;
    private LocalDateTime endTime;

    private JobContext jobContext;
    StepContext stepContext;

    @Override
    public void beforeJob(JobExecution jobExecution) {
        startTime = LocalDateTime.now();
        log.info("start time {}", startTime);
        JobParameters jobParameters = jobExecution.getJobParameters();
        log.info("jobParameters {}", jobParameters);

    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        endTime = LocalDateTime.now();
        log.info("start time {}", endTime);
        log.info("elapsed time: {} s", Duration.between(startTime, endTime).getSeconds());
        if (BatchStatus.COMPLETED.equals(jobExecution.getStatus()) ) {
            //job success
            log.info("job success !");
        }
        else if (BatchStatus.FAILED.equals(jobExecution.getStatus())) {
            //job failure
            log.info("job failure !");
        }

    }
}
```


## 四、使用案例

### 1、引入依赖

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>
```

### 2、添加配置

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboot_batch?serverTimezone=GMT%2b8&useSSL=true&Unicode=true&characterEncoding=utf8&autoReconnectForPools=true&allowMultiQueries=true&rewriteBatchedStatements=true
    username: root
    password: 123456


# 默认是启动时执行一次，设置为false，表示启动不执行，可以通过 web等其他请求调用方式执行
  batch:
    job:
      enabled: false  #是否自动执行定义的Job，默认是
      names:  # 启动是要执行的job
    initialize-schema: never # 是否初始化Spring Batch的数据库，ALWAYS,EMBEDDED,NEVER
    schema:   # 默认有对应的sql脚本，建议手工执行脚本
    table-prefix: TASK_  # 设置Spring Batch的数据库表的前缀,默认是TASK_
```

### 3、数据库准备

这里使用的是MySQL脚本。其他脚本请到spring-batch-core 包找一下就有。

我的工程里也有，路径在：`resource/sql/batch_init.sql`

Schema是 `springboot_batch`，添加待会要导入的 uer表

```SQL
CREATE TABLE USER (
    ID BIGINT PRIMARY KEY AUTO_INCREMENT NOT NULL,
    NAME VARCHAR(10) NOT NULL,
    AGE SMALLINT NULL,
    GENDER CHAR(1)
) ENGINE=InnoDB;

```

### 4、读写示例

比如完整将文件数据导入数据库的示例：

```java
package cn.xiaocai.batch.itemrw.job;
 
/**
 * 一个完整读写的例子，导入省略
 * @author Xiaocai.Zhang
 */
@Slf4j
@Component
@AllArgsConstructor
public class ItemRwJobDemo {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final CsvItemProcessor csvBeanValidator;
    private final MyJobListener myJobListener;

    private final DataSource dataSource;

    /**
     *  以Bean的形式放入ioc，工程启动就会执行，仅demo使用
     * @return
     */
    @Bean
    public Job rwDemoJob() throws Exception {

        return jobBuilderFactory.get("rwDemoJob")
                .start(rwStep())
                .build();
    }

    /**
     *  这里是把读写处理都放一个步骤
     * @return
     */
    private Step rwStep() throws Exception {
        return stepBuilderFactory.get("step")
                .listener(myJobListener)
                .<UserVO,UserVO>chunk(2)
                .reader(fileReader())
                .processor(processor())
                .writer(dataSourceItemWriter())
                .build();
    }

    /**
     * 读取数据
     * @return
     */
    public ItemReader<UserVO> fileReader(){
        // 使用FlatFileItemReader去读cvs文件，一行即一条数据
        FlatFileItemReader<UserVO> reader = new FlatFileItemReader<>();
        // 设置文件处在路径
        reader.setResource(new ClassPathResource("user.csv"));
        // entity与csv数据做映射
        reader.setLineMapper(new DefaultLineMapper<UserVO>() {
            {
                setLineTokenizer(new DelimitedLineTokenizer() {
                    {
                        // 列映射
                        setNames(new String[]{"id", "name", "age", "gender"});
                    }
                });
                setFieldSetMapper(new BeanWrapperFieldSetMapper<UserVO>() {
                    {
                        setTargetType(UserVO.class);
                    }
                });
            }
        });
        return reader;
    }

    /**
     *  处理过程
     * @return
     */
    private ItemProcessor<UserVO, UserVO> processor(){
        CsvItemProcessor processor = new CsvItemProcessor();
        processor.setValidator((Validator<UserVO>) csvBeanValidator);
        return processor;
    }

    /**
     * 写入数据库,需要自己建表
     * @return
     */
    private ItemWriter<UserVO> dataSourceItemWriter() {
        // 写入数据库
        JdbcBatchItemWriter<UserVO> writer = new JdbcBatchItemWriter<>();
        // 设置写入的数据源
        writer.setDataSource(dataSource);

        String sql = "insert into USER(id,name,age,gender) values (:id,:name,:age,:gender)";
        // 设置插入sql脚本
        writer.setSql(sql);

        // 映射UserVO对象属性到占位符中的属性
        BeanPropertyItemSqlParameterSourceProvider<UserVO> provider = new BeanPropertyItemSqlParameterSourceProvider<>();
        writer.setItemSqlParameterSourceProvider(provider);

        writer.afterPropertiesSet(); // 设置一些额外属性
        return writer;
    }


}
```


诸多示例，不再一一列举，找示例可以参考我的工程代码。

### 5、案例代码

#### 1、Demo code

工程地址：[spring-batch-demo](https://github.com/small-rose/springboot-batch-demo)

#### 2、工程代码保函示例：

#### 2.1 ItemReader 相关示例

- 读数据库数据的Job示例
- 读文件数据的Job示例
- 读多个文件数据的Job示例
- 读XML文件数据的Job示例
- 读Json文件数据的Job示例
- 按自定义规则读取数据的Job示例

#### 2.2 Processor 处理器相关示例

- 现接口方式自定义数据过滤的Processor
- 继承的方式实现自定义的Processor

#### 2.3 ItemWriter 相关示例

- 写入数据到数据库的Job示例
- 写入数据到文件的Job示例
- 写入数据到Json文件的Job示例
- 写入数据到多个文件的Job示例
- 写入数据到XML文件的Job示例

#### 2.4 Job 相关示例

- 单个Step的Job示例
- 多个Step的Job示例
- 带状态判断的多个Step的Job示例
- 带决策器的Job示例
- 带监听器的Job示例
- 使用Flow组装Step的Job示例
- 并行执行Job使用的Job示例
- 嵌套执行Job使用的Job示例
- 配合Scheduled调度器启动Job的示例
- Restful接口调用启动Job的示例



## 五、XML 版配置

尽管现在基本上都是面向注解开发，XML配置用的少了，但是保不齐会有一些老项目使用，学习了解一下也是不错的。


### 1、命名空间支持

```xml
<beans:beans xmlns="http://www.springframework.org/schema/batch"
xmlns:beans="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="
   http://www.springframework.org/schema/beans
   https://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/batch
   https://www.springframework.org/schema/batch/spring-batch.xsd">

<!-- 配置 jobRepository-->
<job-repository id="jobRepository"/>

<job id="ioSampleJob">
    <step id="step1">
        <tasklet>
            <chunk reader="itemReader" writer="itemWriter" commit-interval="2"/>
        </tasklet>
    </step>
</job>

</beans:beans>
```
命名空间下面就不再全部粘贴了。

### 2、配置Job

```xml
<job id="footballJob" >
    <step id="playerload"          parent="s1" next="gameLoad"/>
    <step id="gameLoad"            parent="s2" next="playerSummarization"/>
    <step id="playerSummarization" parent="s3"/>
</job>
```

等价于

```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .end()
                     .build();
}
```
#### 2.1 可重启配置

如果当一个job不能被restart的时候,只需要将 restartable 属性设置成 'false'.一个job instance和job parameter相关，每次在启动job时传入的参数的值相同，可以认为同一个job instance. 我们可以将运行job的当前时间作为一个参数传入,这样子每次启动都是不同的job instance.

```xml
<job id="footballJob" restartable="false">
    <step id="playerload"          parent="s1" next="gameLoad"/>
    <step id="gameLoad"            parent="s2" next="playerSummarization"/>
    <step id="playerSummarization" parent="s3"/>
</job>
```

#### 2.2 配置监听器

```xml
<job id="footballJob">
    <step id="playerload"          parent="s1" next="gameLoad"/>
    <step id="gameLoad"            parent="s2" next="playerSummarization"/>
    <step id="playerSummarization" parent="s3"/>
    <listeners>
        <listener ref="sampleListener"/>
    </listeners>
</job>
```

注意的afterJob是，无论方法是成功还是失败，都将调用该方法Job。如果需要确定成功或失败，则可以从中获取JobExecution判断：

```java
public void afterJob(JobExecution jobExecution){
    if (jobExecution.getStatus() == BatchStatus.COMPLETED ) {
        //job success
    }
    else if (jobExecution.getStatus() == BatchStatus.FAILED) {
        //job failure
    }
}
```
#### 2.3 继承Job

```xml
<job id="baseJob" abstract="true">
    <listeners>
        <listener ref="listenerOne"/>
    <listeners>
</job>

<job id="job1" parent="baseJob">
    <step id="step1" parent="standaloneStep"/>

    <listeners merge="true">
        <listener ref="listenerTwo"/>
    <listeners>
</job>
```
此时的 job1 就有两个监听器了。


在XML名称空间中声明的作业或使用的任何子类 `AbstractJob`可以在运行时声明作业参数的验证器。例如，当需要断言一个作业使用其所有必填参数启动时，此功能很有用。有一个 `DefaultJobParametersValidator`可用于约束简单的强制性和可选参数的组合，对于更复杂的约束，需要自己实现接口。

```xml
<job id="job1" parent="baseJob3">
    <step id="step1" parent="standaloneStep"/>
    <validator ref="parametersValidator"/>
</job>
```

### 3、JobRepostory

```xml
<job-repository id="jobRepository"
    data-source="dataSource"
    transaction-manager="transactionManager"
    isolation-level-for-create="SERIALIZABLE"
    table-prefix="BATCH_"
    max-varchar-length="1000"/>
```

配置中，除了ID，其他的如果未设置，将使用上面显示的默认值。`max-varchar-length` 的默认值是2500。

#### 3.1、JobRepostory事务配置

如果使用名称空间或提供的名称空间FactoryBean，则会在JobRepostory操作时自动创建事务建议。


```xml
<job-repository id="jobRepository" isolation-level-for-create="REPEATABLE_READ" />
``                

使用aop的方式配置事物：

​```xml
<aop:config>
    <aop:advisor
           pointcut="execution(* org.springframework.batch.core..*Repository+.*(..))"/>
    <advice-ref = "txAdvice" />
</aop:config>

<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>
```

#### 3.2、改表前缀

```xml
<job-repository id="jobRepository"   table-prefix="SYSTEM.TEST_" />
```

#### 3.3、内存数据库

```xml
<bean id="jobRepository"
  class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
```

新的版本已经废弃了。


### 4、JobLauncher

#### 4.1、同步作业

![JobLauncher 同步作业](/images/spring-batch/job-launcher-sequence-sync.png)

```xml
<bean id="jobLauncher"
      class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository" />
</bean>
```

#### 4.2、异步作业

![JobLauncher 异步作业](/images/spring-batch/job-launcher-sequence-async.png)

```xml
<bean id="jobLauncher"
      class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository" />
    <property name="taskExecutor">
        <bean class="org.springframework.core.task.SimpleAsyncTaskExecutor" />
    </property>
</bean>
```

spring 的`TaskExecutor` 接口的任何实现都可以用来控制如何异步执行作业。


东西还说非常多的，直接留个API吧：[spring-batch-XML](https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/index-single.html#xmlReadingWriting)



 