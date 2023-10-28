---
layout: default
title: MySQL Explain
parent: MySQL
grand_parent: Database
nav_order: 84
permalink: docs/Database/MySQL.Explain
---


## MySQL Explain

## 一、性能影响因素

性能分析相关影响因素。

### 1、MySQL 查询优化器

Query Optimizer 是MySQL 的 Server 中专门负责优化Select 语句的优化器模块，主要通过计算分析系统手机到的统计信息，为客户端请求的Query 提供它认为最优的执行计划。

程序计算分析的最优数据检索方式，和DBA认为的最优方式可能不同。

当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是Select 并转发给MySQL Query Optimizer时，MySQL Query Optimizer首先会对整条Query 进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析Query 中的Hint 信息（如果有），看显示Hint信息是否可以完全确定该Query 的执行计划。如果没有Hint 或 Hint 信息还不足以完全确定执行计划，则会读所取涉及对象的统计信息，根据Query 进行写相应的计算分析，然后再得出最后的执行计划。

### 2、MySQL 常见瓶颈

- CPU ：CPU饱和，一般发生在数据装入内存或从磁盘读取数据时。
- IO ：磁盘IO瓶颈发生在装入数据原大于内存容量时。
- 服务器硬件性能瓶颈。

### 3、SQL与索引

程序问题，SQL问题，索引问题

explain针对SQL/索引，explain这个命令来查看一个这些SQL语句的执行计划



## 二、查询执行计划

### 1、什么是查询执行计划

MySQL 官方内置了 EXPLAIN 关键字可以模拟优化器执行SQL查询语句，从而知晓MYSQL如果处理SQL语句，分析出查询语句或是表解耦股的性能瓶颈问题。

### 2、查询执行计划作用

（1）可以知道表的读取顺序；

（2）可以知道数据读取操作的操作类型；

（3）可以知道哪些索引可以使用，哪些索引被实际使用

（4）可以知道表之间的引用

（5）可以知道每张表有多少行被优化器查询


### 3、查询执行计划使用

```txt
explain + SQL语句
```
示例：
```SQL

```

执行计划包含的信息有10列，分别是<mark>`id`</mark>、`select_type`、`table`、<mark>`type`</mark>、`possible_keys`、<mark>`key`</mark>、`key_len`、`ref`、<mark>`rows`</mark>、<mark>`Extra`</mark> 。

**主要含义：**

- **`id`**： SELECT选择识别符。是SELECT的查询序列号。数字大优先级高。
- **`select_type`**：表示查询的类型。
- **`table`**：输出结果集的表。
- **`partitions`**：匹配的分区。
- **`type`**：表示表的连接类型。
- **`possible_keys`**：表示查询时，可能使用的索引。
- **`key`**：表示实际使用的索引。
- **`key_len`**：索引字段的长度。
- **`ref`**：列与索引的比较。
- **`rows`**：扫描出的行数(估算的行数)。
- **`filtered`**：按表条件过滤的行百分比。
- **`Extra`**：执行情况的描述和说明。

## 三、执行计划信息

### 1、id

select 查询的序列化，包含一组数字，表示查询中执行select字句或操作表的顺序。

主要有分2种情况：

- id 相同，执行顺序由上至下；

- 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行；

- 混合情况，参考以上规则。

### 2、select_type

 描述查询中每个select子句的类型。区分子查询、联合查询等复杂查询。

（1）**SIMPLE**（简单SELECT，不使用UNION或子查询等）

（2）**PRIMARY**（子查询中最外层查询，查询中若包含任何复杂的子部分，最外层的select被标记为PRIMARY）

（3）**SUBQUERY**（在select或where 列表中包含的子查询）

（4）**DERIVED**（在FROM列表中包含的子查询被标记为DERIVED衍生，MySQL会递归执行这些子查询，把结果放入临时表）

（5）**UNION**（UNION中的第二个或后面的SELECT语句）

（6）**UNION RESULT**（从UNION表获取结果的select）

（7）DEPENDENT UNION（UNION中的第二个或后面的SELECT语句，取决于外面的查询）

（8）DEPENDENT SUBQUERY（子查询中的第一个SELECT，依赖于外部查询）

（9）UNCACHEABLE SUBQUERY（一个子查询的结果不能被缓存，必须重新评估外链接的第一行）



### 3、table

显示这一步所访问数据库中表名称（显示这一行的数据是关于哪张表的），有时不是真实的表名字，可能是简称，例如上面的e，d，也可能是第几步执行的结果的简称。

### 4、type

对表访问方式，表示MySQL在表中找到所需行的方式，又称“访问类型”。

常用的类型有： **ALL、index、range、 ref、eq_ref、const、system、NULL**

性能好差：**system > const > eq_ref > ref > range> index > ALL **

- **` system`**：  system是`const`类型的特例，当查询的表只有一行的情况下，使用`system`。平时不会出现。
- **` const `**：表示通过索引一次就找到了数据，const 用于比较 primary key 主键索引 或者 unique 唯一索引。因为只匹配一行数据，所以很快，如将主键置于where 条件列表中，MySQL就能将该查询转换为一个常量。
- **`eq_ref`**： 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用`primary key`或者 `unique key`作为关联条件分解语句。
- **`ref`**： 非唯一的索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行。它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。

- **`range`**：只检索给定范围的行，使用一个索引来选择行，key列显示使用哪个所以。一般就是在where语句中出现了 `between`、 `<`、 `>`、`in` 等范围查询时，这个范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。

- **`index`**： Full Index Scan，index与ALL区别为index类型只遍历索引树。通常比ALL快，因为索引文件比数据文件小。虽然index 和 all 都是读全表，但是index 是从索引读取的，而all 是从硬盘读取的。

- **`ALL`**：Full Table Scan， MySQL将遍历全表数据以找到匹配的行。
- **`NULL`**：  MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

 

### 5、possible_keys

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该说了将被列出，但不一定被查询实际使用。

需要和`key`配合查看。

### 6、key

需要和`possible_keys`配合查看。

被查询实际使用的索引。如果为NULL表示没有使用索引，可能是没有索引或索引失效。

查询中如果使用了覆盖索引，则该索引仅出现在key列表中。

覆盖索引：select 后的列表和索引一一对应了。



### 7、key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，不损失精确性的情况下，长度越短越好 。key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。



### 8、ref

**列与索引的比较，表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值**

显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引列上的值。

### 9、rows

 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。



### 10、Extra

该列包含MySQL解决查询的详细信息，主要有：

（1）**`Using filesort`**：说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行当Query中包含 order by 操作，而且无法利用索引完成的排序操作称为"文件内排序"。【警告指标】

（2）**`Using temporary`**：表示MySQL需要使用临时表来存储结果集，常见于排序 `group by` 和分组查询 `order by`。【危险指标】

（3）**`Using index`**：表示响应的select操作中使用了覆盖索引（Covering Index）,避免访问了表的数据行，效率不错。如果同时出现`using where` 表明索引被用来执行索引键值的查找；如果没有同时出现`using where`，表明索引用来读取数据而非执行查找动作。

> 索引覆盖Covering Index：
>
> 就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件。即：查询列要被所建的索引覆盖。

（4）**`Using where`**：不用读取表中所有信息，仅通过索引就可以获取所需数据，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤。

（5）**`Using join buffer`**：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，收货码使用连接缓存，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

（6）**`Impossible where`**：这个值强调了where语句会导致没有符合条件的行（通过收集统计信息不可能存在结果）。

（7）**`Select tables optimized away`**：在没有Group By子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行。

（8）**`distinct`**：优化distinct操作，在找到第一匹配的元组后停止找同样值的动作。

（9）**`No tables used`**：Query语句中使用from dual 或不含任何from子句

**注意事宜**

• EXPLAIN没有描述关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
• EXPLAIN不考虑各种Cache
• EXPLAIN不能显示MySQL在执行查询时所作的优化工作
• 部分统计信息是估算的，并非精确值
• EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划。



学习参考资料：

- B站视频[MySQL高级](https://www.bilibili.com/video/BV1KW411u7vy?p=21) 

- [博客资料原地址](https://www.cnblogs.com/tufujie/p/9413852.html) 

- 网络资料

