---
layout: default
title: Mongodb-CURD
nav_order: 5
parent: Database
---

# Mongodb-CURD
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

--- 

## 一、查询入门

### 1、基础概念

下表介绍了各种SQL术语和概念以及相应的MongoDB术语和概念。

| SQL术语/概念                                       | MongoDB术语/概念                                             |
| -------------------------------------------------- | ------------------------------------------------------------ |
| database                                           | [database](https://docs.mongodb.com/manual/reference/glossary/#term-database) |
| table                                              | [collection](https://docs.mongodb.com/manual/reference/glossary/#term-collection) |
| row                                                | [document](https://docs.mongodb.com/manual/reference/glossary/#term-document) or [BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson) document |
| column                                             | [field](https://docs.mongodb.com/manual/reference/glossary/#term-field) |
| index                                              | [index](https://docs.mongodb.com/manual/reference/glossary/#term-index) |
| table joins                                        | [$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup), 嵌入文档 |
| primary key （指定任何唯一的列或列组合作为主键。） | [primary key](https://docs.mongodb.com/manual/reference/glossary/#term-primary-key) （在MongoDB中，主键自动设置为_id字段。） |

MySQL中的许多概念在MongoDB中具有相近的类比。

| MySQL           | MongoDB                    |
| --------------- | -------------------------- |
| 库 database     | 库 database                |
| 表 table        | 集合 collection            |
| 行 row          | 文档 document 或 bson 文档 |
| 列 column       | 字段 field                 |
| 表连接 joins    | 嵌入文档或链接             |
| 索引 index      | 索引 index                 |
| 主键 可以自定义 | 主键 _id                   |

数据库对于的服务也是类似的：

下表展示了一些数据库可执行文件和相应的MongoDB可执行文件。这个表格并不是详尽无遗的。

|                 | MongoDB                                                      | MySQL  | Oracle  | Informix  | DB2        |
| --------------- | ------------------------------------------------------------ | ------ | ------- | --------- | ---------- |
| Database Server | [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) | mysqld | oracle  | IDS       | DB2 Server |
| Database Client | [mongo](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) | mysql  | sqlplus | DB-Access | DB2 Client |

### 2、入门示例

下表展示了各种SQL语句和相应的MongoDB语句。表中的例子假设以下条件:

- SQL示例假设有一个名为**users**的表。
- MongoDB示例假设一个名为**users**的集合，它包含以下原型的文档:

```json
{ 
       _id: ObjectId("509a8fb2f3f4948bd2f983a0"),
       user_id: "abc123",
       age: 55,
       status: 'A'
 }
```



##### 创建和修改

下表展示了与表级操作相关的各种SQL语句以及相应的MongoDB语句。

| SQL Schema语句                                               | MongoDB Schema语句                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **CREATE** **TABLE** users(    id MEDIUMINT **NOT** **NULL**        AUTO_INCREMENT,    user_id Varchar(30),    age Number,    status char(1),    **PRIMARY** **KEY** (id) ) | 隐式创建的第一个[`insertOne()`](https://docs.mongodb.com/master/reference/method/db.collection.insertOne/#db.collection.insertOne)或[`insertMany()`](https://docs.mongodb.com/master/reference/method/db.collection.insertMany/#db.collection.insertMany)操作。如果没有指定**_id**字段，则会自动添加主键_id。 db.users.insertOne( {    user_id: "abc123",    age: 55,    status: "A" } ) 但是，也可以显式地创建一个集合: db.createCollection("users") |
| **ALTER** **TABLE** users **ADD** join_date DATETIME         | 集合不描述或不强制其文件结构； 即在集合级别没有结构上的更改。 但是，在文档级别，[updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)操作可以使用[$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set)运算符将字段添加到现有文档中。 db.users.updateMany(    { },    { $set: { join_date: **new** Date() } } ) |
| **ALTER** **TABLE** users **DROP** **COLUMN** join_date      | 集合不描述或不强制其文件结构； 即在集合级别没有结构上的更改。 但是，在文档级别，[updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)操作可以使用[$unset](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset)运算符将字段添加到现有文档中。 db.users.updateMany(    { },    { $unset: { "join_date": "" } } ) |
| **CREATE** **INDEX** idx_user_id_asc **ON** users(user_id)   | db.users.createIndex( { user_id: 1 } )                       |
| **CREATE** **INDEX**       idx_user_id_asc_age_desc **ON** users(user_id, age **DESC**) | db.users.createIndex( { user_id: 1, age: -1 } )              |
| **DROP** **TABLE** users                                     | db.users.drop()                                              |

##### 写入记录

下表显示了与将记录插入表和相应的MongoDB语句有关的各种SQL语句。

| SQL INSERT语句                                               | **MongoDB insertOne() Statements**                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **INSERT** **INTO** users(user_id, age,  status) **VALUES** ("bcd001",        45,        "A") | db.users.insertOne(   { user_id: "bcd001", age: 45, status: "A" } ) |

相关文档：

- [`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) 

- [`Insert Documents`](https://docs.mongodb.com/manual/tutorial/insert-documents/) 
- [`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) 
- [`Databases and Collections`](https://docs.mongodb.com/manual/core/databases-and-collections/) 
- [`Documents `](https://docs.mongodb.com/manual/core/document/)



##### 查询/选择

下表展示了与从表中读取记录相关的各种SQL语句以及相应的MongoDB语句。

> **注意**
>
> 除非通过投影明确排除，否则[[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法始终在返回的文档中包含**_id**字段。 下面的某些SQL查询可能包含一个**_id**字段来反映这一点，即使该字段未包含在相应的[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)查询中也是如此。

| SQL SELECT 语句                                              | MongoDB find() 语句                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| SELECT  FROM users                                           | db.users.find()                                              |
| **SELECT** id,       user_id,       status **FROM** users    | db.users.find(     { },     { user_id: 1, status: 1 } )      |
| **SELECT** user_id, status **FROM** users                    | db.users.find(     { },     { user_id: 1, status: 1, _id: 0 } ) |
| **SELECT**  * **FROM** **users **WHERE status = "A"          | db.users.find(     { status: "A" } )                         |
| **SELECT** user_id, status **FROM** users **WHERE** status = "A" | db.users.find(     { status: "A" },     { user_id: 1, status: 1, _id: 0 } ) |
| **SELECT**  * **FROM** *users* **WHERE** status != "A"       | db.users.find(     { status: { $ne: "A" } } )                |
| **SELECT**  * **FROM** **users** **WHERE** status = "A"  AND age = 50 | db.users.find(     { status: "A",       age: 50 } )          |
| **SELECT**  * **FROM** *users* **WHERE** status = "A" **OR** age = 50 | db.users.find(     { $or: [ { status: "A" } , { age: 50 } ] } ) |
| **SELECT**  * **FROM** *users* **WHERE** age > 25            | db.users.find(     { age: { $gt: 25 } } )                    |
| **SELECT**  * **FROM** *users* **WHERE** age < 25            | db.users.find(    { age: { $lt: 25 } } )                     |
| **SELECT**  * **FROM** *users* **WHERE** *age > 25* **AND**   age <= 50 | db.users.find(    { age: { $gt: 25, $lte: 50 } } )           |
| **SELECT**  * **FROM** *users* **WHERE** *user_id like* "%bc%" | db.users.find( { user*id: /bc/ } )* *_or* db.users.find( { user_id: { $regex: /bc/ } } ) |
| **SELECT**  * **FROM** *users* **WHERE** *user_id like* "bc%" | db.users.find( { user*id: /^bc/ } )* *_or* db.users.find( { user_id: { $regex: /^bc/ } } ) |
| **SELECT**  * **FROM** *users* **WHERE** *status = "A"* **ORDER** **BY** *user_id* *ASC* | db.users.find( { status: "A" } ).sort( { user_id: 1 } )      |
| **SELECT**  * **FROM** *users* **WHERE** *status = "A"* **ORDER** **BY** *user_id* *DESC* | db.users.find( { status: "A" } ).sort( { user_id: -1 } )     |
| **SELECT** **COUNT**(*) **FROM** users                       | db.users.count() *or* db.users.find().count()                |
| **SELECT** **COUNT**(user_id) **FROM** users                 | db.users.count( { user*id: { $exists:* **true** *} } )* *_or* db.users.find( { user_id: { $exists: **true** } } ).count() |
| **SELECT** **COUNT**(*)* **FROM** *users* **WHERE** age > 30 | db.users.count( { age: { $gt: 30 } } ) *or* db.users.find( { age: { $gt: 30 } } ).count() |
| **SELECT** **DISTINCT**(status) **FROM** users               | db.users.aggregate( [ { $group : { _id : "$status" } } ] ) or, for distinct value sets that do not exceed the [BSON size limit](https://docs.mongodb.com/manual/reference/limits/#limit-bson-document-size) db.users.distinct( "status" ) |
| **SELECT**  * **FROM** *users* **LIMIT** 1                   | db.users.findOne() *or* db.users.find().limit(1)             |
| **SELECT**  * **FROM** *users* **LIMIT** 5 SKIP 10           | db.users.find().limit(5).skip(10)                            |
| **EXPLAIN** **SELECT**  **FROM** *users* **WHERE** status = "A" | db.users.find( { status: "A" } ).explain()                   |

相关文档：

- [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) 

- [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) 

- [`db.collection.findOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/#db.collection.findOne) 

- [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/) 
- [Query and Projection Operators](https://docs.mongodb.com/manual/reference/operator/query/) 
- [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/) 

##### 更新记录

下表显示了与更新表中的现有记录和相应的MongoDB语句有关的各种SQL语句。

| **SQL Update Statements**                                    | **MongoDB updateMany() Statements**                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **UPDATE** users **SET** status = "C" **WHERE** age > 25     | db.users.updateMany(   { age: { $gt: 25 } },   { $set: { status: "C" } } ) |
| **UPDATE** users **SET** age = age + 3 **WHERE** status = "A" | db.users.updateMany(       { status: "A" } ,       { $inc: { age: 3 } } ) |

相关文档：

- [Update Documents](https://docs.mongodb.com/manual/tutorial/update-documents/)
- [Update Operators](https://docs.mongodb.com/manual/reference/operator/update/)
- [db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)
- [db.collection.replaceOne()](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)

##### 查询指定的列

下表展示了与从表中读取记录相关的各种SQL语句以及相应的MongoDB语句。

> **注意**
>
> 除非通过投影明确排除，否则[[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法始终在返回的文档中包含**_id**字段。 下面的某些SQL查询可能包含一个**_id**字段来反映这一点，即使该字段未包含在相应的[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)查询中也是如此。

| SQL SELECT 语句                                              | MongoDB find() 语句                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **SELECT**  * **FROM** users                                 | db.users.find()                                              |
| **SELECT** id,       user_id,       status **FROM** users    | db.users.find(     { },     { user_id: 1, status: 1 } )      |
| **SELECT** user_id, status **FROM** users                    | db.users.find(     { },     { user_id: 1, status: 1, _id: 0 } ) |
| **SELECT**  * **FROM** *users* **WHERE** status = "A"        | db.users.find(     { status: "A" } )                         |
| **SELECT** user_id, status **FROM** users **WHERE** status = "A" | db.users.find(     { status: "A" },     { user_id: 1, status: 1, _id: 0 } ) |
| **SELECT**  * **FROM** *users* **WHERE** status != "A"       | db.users.find(     { status: { $ne: "A" } } )                |
| **SELECT**  * **FROM** *users* **WHERE** *status = "A"* **AND** age = 50 | db.users.find(     { status: "A",       age: 50 } )          |
| **SELECT**  * **FROM** *users* **WHERE** *status = "A"* **OR** age = 50 | db.users.find(     { $or: [ { status: "A" } , { age: 50 } ] } ) |
| **SELECT**  * **FROM** *users* **WHERE** age > 25            | db.users.find(     { age: { $gt: 25 } } )                    |
| **SELECT**  * **FROM** *users* **WHERE** age < 25            | db.users.find(    { age: { $lt: 25 } } )                     |
| **SELECT**  * **FROM** *users* **WHERE** *age > 25*  **AND**   age <= 50 | db.users.find(    { age: { $gt: 25, $lte: 50 } } )           |
| **SELECT**  * **FROM** *users* **WHERE** *user_id* **like** "%bc%" | db.users.find( { user*id: /bc/ } )* *_or* db.users.find( { user_id: { $regex: /bc/ } } ) |
| **SELECT**  * **FROM** *users* **WHERE** *user_id* **like** "bc%" | db.users.find( { user*id: /^bc/ } )* *_or* db.users.find( { user_id: { $regex: /^bc/ } } ) |
| **SELECT**  * **FROM** *users* **WHERE** *status = "A"* **ORDER** **BY** *user_id* **ASC** | db.users.find( { status: "A" } ).sort( { user_id: 1 } )      |
| **SELECT**  * **FROM** *users* **WHERE** *status = "A"* **ORDER** **BY** *user_id* *DESC* | db.users.find( { status: "A" } ).sort( { user_id: -1 } )     |
| **SELECT** **COUNT**(\*)  **FROM** users                     | db.users.count() *or* db.users.find().count()                |
| **SELECT** **COUNT**(user_id) **FROM** users                 | db.users.count( { user*id: { $exists:* ***true\*** *} } )* *_or* db.users.find( { user_id: { $exists: **true** } } ).count() |
| **SELECT** **COUNT** **(****)*  **FROM**  *users* **WHERE** age > 30 | db.users.count( { age: { $gt: 30 } } ) *or* db.users.find( { age: { $gt: 30 } } ).count() |
| **SELECT** **DISTINCT**(status) **FROM** users               | db.users.aggregate( [ { $group : { _id : "$status" } } ] ) or, for distinct value sets that do not exceed the [BSON size limit](https://docs.mongodb.com/manual/reference/limits/#limit-bson-document-size)  db.users.distinct( "status" ) |
| **SELECT**  * **FROM** *users* **LIMIT** 1                   | db.users.findOne() *or* db.users.find().limit(1)             |
| **SELECT**  * **FROM** *users* **LIMIT** 5 SKIP 10           | db.users.find().limit(5).skip(10)                            |
| **EXPLAIN** **SELECT**  * **FROM** *users* **WHERE** status = "A" | db.users.find( { status: "A" } ).explain()                   |

相关文档

- [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
- [Query and Projection Operators](https://docs.mongodb.com/manual/reference/operator/query/)
- [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/)

##### 删除记录

下表显示了与从表中删除记录和相应的MongoDB语句有关的各种SQL语句。

| **SQL Delete Statements**                        | **MongoDB deleteMany() Statements**    |
| ------------------------------------------------ | -------------------------------------- |
| **DELETE** **FROM** users **WHERE** status = "D" | db.users.deleteMany( { status: "D" } ) |
| **DELETE** **FROM** users                        | db.users.deleteMany({})                |

获得更多信息，请参见：[db.collection.deleteMany()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany).

另看：

- [Delete Documents](https://docs.mongodb.com/manual/tutorial/remove-documents/) 
- [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne) 



## 二、查询与映射运算符

- 查询选择器
- 映射运算符
- 文档直达：[查询与映射运算符文档](https://docs.mongodb.com/v4.0/reference/operator/query/) 

### 1、常见函数

#### （1）count()

统计条数。

```SQL
select count(*) from users; 

db.users.count(); 
```



#### （2）distinct()

去重。

```SQL
select distinct (name) from users; 

db.users.distinct('name');
```

#### 

#### （3）limit()

```SQL
select * from users limit 10; 

db.users.find().limit(10););
```



#### （3）sort()

```SQL
select * from users order by  opdate  desc; 

db.users.find()..sort({
"opdate":-1
});
```

排序函数，类似`order by`  1是升序，-1降序。



#### （4）查特定列

```SQL
select name, skills from users;

db.users.find({}, {'name' : 1, 'age' : 1}); 
```

补充说明： 第一个{} 放where条件 第二个{} 指定那些列显示和不显示 （0表示不显示 1表示显示)



### 2、查询选择器



#### （1）比较

有关不同BSON类型值的比较，请参见指定的BSON比较顺序。

 **$eq**  

选择：等于某个值

```SQL
{ <field>: { $eq: <value> } }
```

示例：

查询年龄等于18岁的用户：

```SQL
db.Users.find({ age : {$eq:18} })
```

等价sql：

```SQL
SELECT * FROM USERS WHERE age = 18;
```



**$gt** 

选择：大于某个值

```SQL
{field: {$gt: value} }
```

示例：

查询年龄大于18岁的用户：

```SQL
db.Users.find({ age : {$gt:18} })
```

等价sql：

```SQL
SELECT * FROM USERS WHERE age > 18;
```



**$gte**

 选择：大于等于某个值

```SQL
{field: {$gte: value} }
```

示例：

查询年龄大于或等于18岁的用户：

```SQL
db.Users.find({ age : {$gte:18} })
```

等价sql：

```SQL
SELECT * FROM USERS WHERE age >= 18;
```



**$lt**

选择：小于某个值

```SQL
{field: {$lt: value} }
```

示例：

查询年龄小于22岁的用户：

```SQL
db.Users.find({ age : {$lt:18} })
```

等价sql：

```SQL
SELECT * FROM USERS WHERE age < 22;
```



**$lte**

选择：小于等于某个值

```SQL
{ field: { $lte: value} }
```

示例：

查询年龄小于22岁的用户：

```SQL
db.Users.find({ age : {$lte:22} })
```

等价sql：

```SQL
SELECT * FROM USERS WHERE age <= 22;
```



**$ne**

```SQL
{field: {$ne: value} }
```

选择：

field不等于某个值
field不存在

示例：

查询用户状态不等于1的用户

```
db.Users.find({ status : {$ne: 1} })
```

等价sql：

```SQL
SELECT * FROM USERS WHERE status != 1;
```





**$in**

```
{ field: { $in: [<value1>, <value2>, ... <valueN> ] } }
```

选择：

（1）filed是一个值，并且等于value1 or value2 。
（2）field是个数组，并且filed中至少存在某个值等于value1 or value2 。

示例：

查询指定邮箱的用户：

```SQL
db.Users.find({ email : {$in: ['small-rose@qq.com','xiaocai@qq.com']} })
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE email in ('small-rose@qq.com','xiaocai@qq.com');
```



**$nin**

```
{ field: { $nin: [ <value1>, <value2> ... <valueN> ]} }
```

选择：
（1）field值不存在于数组中
（2）field不存在

 

示例：

查询不含有某几个邮箱的用户：

```SQL
db.Users.find({ email : {$nin: ['small-rose@qq.com','xiaocai@qq.com']} })
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE email NOT IN ('small-rose@qq.com','xiaocai@qq.com');
```



#### （2）逻辑

**$and**

```SQL
{ $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }
```

含义：选择同时匹配所有expression的数据,如果expression1为false，则mongodb不会继续后续的判断

示例：

查询年龄是20，状态为1的用户：

```SQL
db.Users.find({ $and : [ { age : 20}, { status : 1} ]});
```

如果找不到年龄为20的，status为1不作判断。

等价SQL：

```SQL
SELECT * FROM USERS WHERE age =20 and status =1;
```



**$not**

```SQL
{ field: { $not: { <operator-expression> } } }
```

含义：选择所有不符合operator-expression的数据。

示例：

查询所有状态不为1 或者status字段不存在的用户：

```SQL
db.Users.find({ $not : [{ status : 1} ]});
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE status !=1;
```



**$or**

```SQL
{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
```

含义：选择至少满足一个expression的数据。

示例：

查询所有状态为1或者状态为3的用户：

```SQL
db.Users.find({ $or : [{ status : 1},{ status : 3} ]});
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE status =1 or status =3 ;
```



**$nor**

```SQL
{ $nor: [ { <expression1> }, { <expression2> }, ... { <expressionN> } ] }
```

含义：选择同时不满足所有expression的数据。

示例：

查询所有状态为1或者状态为3的用户：

```SQL
db.Users.find({ $or : [{ status : 1},{ status : 3},{ age : 22} ]});
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE status =1 or status =3 or age = 22;
```



#### （3）元素

**$exists**

```SQL
{ field: { $exists: <boolean> } }
```

含义：选选择filed是否存在的数据，$exists: true代表存在，false代表不存在。

示例：

查询所有邮箱没有值的用户：

```SQL
db.Users.find({ email: { $exists : false} });
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE email is null ;
```



**$type**

```SQL
{ field: { $type: <BSON type> } }
```

含义：选择filed是某个具体类型的实例的数据，可以写具体type或者对应数字。

具体BSON type可以看文档：[Bson Type]()  

　　选择字段值为指定的BSON数据类型的文档.<BSON type>使用下面类型对应的编号：

| 类型                    | 类型                   | 编号 |
| ----------------------- | ---------------------- | ---- |
| Double                  | 双精度                 | 1    |
| String                  | 字符串                 | 2    |
| Object                  | 对象                   | 3    |
| Array                   | 数组                   | 4    |
| Binary data             | 二进制对象             | 5    |
| Object id               | 对象id                 | 7    |
| Boolean                 | 布尔值                 | 8    |
| Date                    | 日期                   | 9    |
| Null                    | 未定义                 | 10   |
| Regular Expression      | 正则表达式             | 11   |
| JavaScript              | JavaScript代码         | 13   |
| Symbol                  | 符号                   | 14   |
| JavaScript (with scope) | JavaScript代码(带范围) | 15   |
| 32-bit integer          | 32 位整数              | 16   |
| Timestamp               | 时间戳                 | 17   |
| 64-bit integer          | 64 位整数              | 18   |
| Min key                 | 最小键                 | 255  |
| Max key                 | 最大键                 | 127  |

　　如果文档的键值是一个数组。那么$type将对数组里面的元素进行类型匹配而不是键值数组本身。



示例：

```json
{ "_id" : 1, "name" : "Jack", "age": 18, "tags": ['student','child'] }
{ "_id" : 2, "name" : "Lucy", "age": 22, "tags": ['teacher','foods'] }
{ "_id" : 2, "name" : "Lucy", "age": 22, "tags": 'teacher' }
{ "_id" : 2, "name" : "Lucy", "age": 22 }
```

查询所有标签的数据类型是数组的的用户：

```SQL
db.user.find( { tags: { $type : 4 } } )

//如果想检查键值的类型是否为数组类型，使用$where操作符
db.user.find( { $where : "Array.isArray(this.tags)" } )
```

 



#### （4）评估

**$expr**

```SQL
{ $expr: { <expression> } }
```

含义：选择filed符合聚合表达式的数据。

聚合表达式官方参考：[聚合表达式管道参考](https://docs.mongodb.com/v4.0/meta/aggregation-quick-reference/#aggregation-expressions) 

示例：

查询所有spent值大于budget值的所有数据：

数据：

```
{ "_id" : 1, "category" : "food", "budget": 400, "spent": 450 }
{ "_id" : 2, "category" : "drinks", "budget": 100, "spent": 150 }
{ "_id" : 3, "category" : "clothes", "budget": 100, "spent": 50 }
{ "_id" : 4, "category" : "misc", "budget": 500, "spent": 300 }
{ "_id" : 5, "category" : "travel", "budget": 200, "spent": 650 }
```

查询语句

```SQL
db.monthlyBudget.find( { $expr: { $gt: [ "$spent" , "$budget" ] } } )
```

结果：

```
{ "_id" : 1, "category" : "food", "budget" : 400, "spent" : 450 }
{ "_id" : 2, "category" : "drinks", "budget" : 100, "spent" : 150 }
{ "_id" : 5, "category" : "travel", "budget" : 200, "spent" : 650 }
```

等价伪SQL：

```SQL
SELECT * FROM monthlyBudget WHERE spent > budget ;
```



| Name                                                         | Description                                    |
| ------------------------------------------------------------ | ---------------------------------------------- |
| [`$expr`](https://docs.mongodb.com/v4.0/reference/operator/query/expr/#op._S_expr) | 允许在查询语言中使用聚合表达式。               |
| [`$jsonSchema`](https://docs.mongodb.com/v4.0/reference/operator/query/jsonSchema/#op._S_jsonSchema) | 根据给定的JSON Schema验证文档。                |
| [`$mod`](https://docs.mongodb.com/v4.0/reference/operator/query/mod/#op._S_mod) | 对字段的值执行模运算并选择具有指定结果的文档。 |
| [`$regex`](https://docs.mongodb.com/v4.0/reference/operator/query/regex/#op._S_regex) | 选择值与指定的正则表达式匹配的文档。           |
| [`$text`](https://docs.mongodb.com/v4.0/reference/operator/query/text/#op._S_text) | 执行文本搜索。                                 |
| [`$where`](https://docs.mongodb.com/v4.0/reference/operator/query/where/#op._S_where) | 匹配满足JavaScript表达式的文档。               |



#### （5）数组

> **注意**
>
> 有关特定运算符的详细信息，包括语法和示例，请单击特定运算符以转到其参考页。

| 名称             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| [`$all`]()       | 匹配包含查询中指定的所有元素的数组。                         |
| [`$elemMatch`]() | 如果array字段中的元素符合所有指定`$elemMatch`条件，则选择文档。 |
| [`$size`]()      | 如果数组字段为指定大小，则选择文档。                         |

示例：

```json
{ "_id" : 1, "name" : "Jack", "age": 18, "tags": ['student','child'] }
{ "_id" : 2, "name" : "Lucy", "age": 22, "tags": ['teacher','foods'] }
{ "_id" : 2, "name" : "Lucy", "age": 22, "tags": ['python','mysql','java'] }
{ "_id" : 2, "name" : "Lucy", "age": 21, "tags": ['java','python']  }
```

查询所有标签的数据类型是数组的的用户：

```SQL
//tags 中必须同时包含java 和 python 
db.users.find({'tags' : {'$all' : ['java','python']}})

//tags数组长度为3的数据，注意 $size不能与$lt等组合使用
db.users.find({'tags' : {'$size' : 3}}) 
```

 







#### （6）地理

6.1 查询选择器

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$geoIntersects`](https://docs.mongodb.com/v4.0/reference/operator/query/geoIntersects/#op._S_geoIntersects) | 选择与GeoJSON几何形状相交的几何形状。2dsphere索引支持 `$geoIntersects`。 |
| [`$geoWithin`](https://docs.mongodb.com/v4.0/reference/operator/query/geoWithin/#op._S_geoWithin) | 选择边界GeoJSON几何内的几何。2dsphere和2D指标支持 `$geoWithin`。 |
| [`$near`](https://docs.mongodb.com/v4.0/reference/operator/query/near/#op._S_near) | 返回点附近的地理空间对象。需要地理空间索引。2dsphere和2D指标支持 `$near`。 |
| [`$nearSphere`](https://docs.mongodb.com/v4.0/reference/operator/query/nearSphere/#op._S_nearSphere) | 返回球体上某个点附近的地理空间对象。需要地理空间索引。2dsphere和2D指标支持 `$nearSphere`。 |



6.2 几何说明符

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$box`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 使用传统坐标对来指定一个矩形框进行 `$geoWithin`查询。所述2D指数支持 `$box`。 |
| [`$center`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 使用平面几何时，使用旧坐标对指定圆以进行`$geoWithin`查询。所述2D指数支持`$center`。 |
| [`$centerSphere`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 使用球形几何图形时，使用传统坐标对或GeoJSON格式指定一个圆 用于`$geoWithin`查询。2dsphere和 2D指标支持`$centerSphere`。 |
| [`$geometry`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 为地理空间查询运算符指定GeoJSON格式的几何。                  |
| [`$maxDistance`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 指定最大距离以限制`$near` 和`$nearSphere`查询的结果。2dsphere和2D指标支持 `$maxDistance`。 |
| [`$minDistance`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 指定最小距离以限制`$near` 和`$nearSphere`查询的结果。`2dsphere`仅用于索引。 |
| [`$polygon`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 指定用于`$geoWithin`查询的旧坐标对的多边形。2d索引支持`$center`。 |
| [`$uniqueDocs`](https://docs.mongoing.com/can-kao/yun-suan-fu/query-and-projection-operators/geospatial-query-operators) | 不推荐使用。修改`$geoWithin`和`$near`查询以确保即使文档多次匹配查询，查询也只返回一次文档。 |



#### （7）按位运算

> **注意**
>
> 有关特定运算符的详细信息，包括语法和示例，请单击特定运算符以转到其参考页。

| 名称                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| [`$bitsAllClear`]() | 匹配数字或二进制值，其中一组位的*所有*值均为`0`。            |
| [`$bitsAllSet`]()   | 匹配数字或二进制值，其中一组位的*所有*值均为`1`。            |
| [`$bitsAnyClear`]() | 匹配数值或二进制值，在这些数值或二进制值中，一组位的位置中*任何*位的值为`0`。 |
| [`$bitsAnySet`]()   | 匹配数值或二进制值，在这些数值或二进制值中，一组位的位置中*任何*位的值为`1`。 |



#### （8）评论选

| Name                                                         | Description                          |
| ------------------------------------------------------------ | ------------------------------------ |
| [`$comment`](https://docs.mongodb.com/v4.0/reference/operator/query/comment/#op._S_comment) | Adds a comment to a query predicate. |



### 3、映射运算符

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$`](https://docs.mongodb.com/v4.0/reference/operator/projection/positional/#proj._S_) | 匹配数组中与查询条件匹配的第一个元素。                       |
| [`$elemMatch`](https://docs.mongodb.com/v4.0/reference/operator/projection/elemMatch/#proj._S_elemMatch) | Projects the first element in an array that matches the specified [`$elemMatch`](https://docs.mongodb.com/v4.0/reference/operator/projection/elemMatch/#proj._S_elemMatch) condition. |
| [`$meta`](https://docs.mongodb.com/v4.0/reference/operator/projection/meta/#proj._S_meta) | Projects the document’s score assigned during [`$text`](https://docs.mongodb.com/v4.0/reference/operator/query/text/#op._S_text) operation. |
| [`$slice`](https://docs.mongodb.com/v4.0/reference/operator/projection/slice/#proj._S_slice) | Limits the number of elements projected from an array. Supports skip and limit slices. |







参考资料：

https://mongoing.com/docs/core/query-plans.html

https://mongoing.com/archives/docs/mongodb

https://docs.mongodb.com/v4.0/reference/operator/query/

## 