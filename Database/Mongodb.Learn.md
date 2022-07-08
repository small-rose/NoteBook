---
layout: default
title: Mongodb Learn
nav_order: 6
parent: Database
---

# Mongodb Learn
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

--- 

## mongodb查询入门

 一、查询入门

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

