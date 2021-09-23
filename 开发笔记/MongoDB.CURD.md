## 一、查询与映射运算符

- 查询选择器
- 映射运算符

### 1、查询选择器

### 2、映射运算符

#### （1）比较运行符

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



#### （2）逻辑运算符

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



#### （3）元素运算符

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

示例：

查询所有邮箱没有值的用户：

```SQL
db.Users.find({ email: { $exists : false} });
```

等价SQL：

```SQL
SELECT * FROM USERS WHERE email is null ;
```



具体BSON type可以看文档：[Bson Type](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.mongodb.com%2Fmanual%2Freference%2Foperator%2Fquery%2Ftype%2F%23document-type-available-types)  





参考资料：https://juejin.cn/post/6844904136094253064



https://mongoing.com/docs/core/query-plans.html



https://mongoing.com/archives/docs/mongodb%e5%88%9d%e5%ad%a6%e8%80%85%e6%95%99%e7%a8%8b/%e4%bd%bf%e7%94%a8find%ef%bc%88%ef%bc%89%e5%92%8c%e7%a4%ba%e4%be%8b%e7%9a%84mongodb%e6%9f%a5%e8%af%a2%e6%96%87%e6%a1%a3


## 二、查询与映射运算符