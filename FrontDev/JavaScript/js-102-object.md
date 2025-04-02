---
layout: default
title: Js Object
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 12
---

 Here are JavaScript Object examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、Object 简介


## 1、js 对象

在 JavaScript 中的所有事物都是**对象**：字符串 （ new String）、布尔(new Boolean())、数值(Number)、数组(Array)、函数(Function)等。

**类**：就是具有相同的属性和方法的集合。人类，动物类，家电类等。

对象：就类中的一个具体的实物。


> 人类-具体某一个人（张三）。动物类-(一个具体的动物-东北虎)，家电类-（具体一个比如说彩色电视。）

js 中我们包含哪些对象呢？

- （1）内置对象（本地对象）：Math 对象，Number 对象，Date 对象等，组件已经内置定义好的。
- （2）宿主对象：dom（文档对象模型）,bom(浏览器对象)。
- （3）自定义对象：我们自己定义和开发的对象。
 

## 2、对象的创建与构成


对象是 JavaScript 的核心概念，也是最重要的数据类型。JavaScript 的所有数据都可以被视为对象。

JavaScript 允许自定义对象。

对象（object）是大括号定义的无序的数据集合，由键值对构成，键名，键名与键值之间用冒号分隔，大括号末尾要使用分号表示对象定义结束。


基本语法：

```javascript
var obj = { key : value };
```
上面代码定义了一个对象，它被赋值给变量 obj。

- key 是“键名”
- value 是“键值”

如果对象内部包含多个键值对，每个键值对之间用逗号分隔。最后一个键值对末尾不用加逗号。

```javascript
var obj = {key1:value1,key2:value2};
```

对象创建方式

- a.直接使用大括号创建对象
- b.使用 `new` 命令生成一个 Object 对象的实例
- c.使用 `Object.create` 方法创建对象

````html
<script type="text/javascript">
var obj1 = {};

let obj2 = new Object();

let obj3 = Object.create(null);

//给对象添加了属性
obj3.name = "唐三藏";
obj3.sex = "男";
obj3.from = "洛阳";

//添加方法
obj3.fly = function(){
    console.log("我会念经！")
};
obj3.fly();
console.log(obj3);
</script>
````

Object 是在 javascript 中一个被我们经常使用的类型，（和Java一样）Js 中的所有对象都是继承自 Object 对象的



## 3、对象的键名和键值

**键名**

键名也被称为属性(property)，对象的所有属性都是字符串，所以加不加引号都可以。

```javascript
var obj = { 'key' : value };
```
但是，如果属性不符合标识符的条件(比如第一个字符为数字，或者含有空格或运算符)，则必须加上引号。

```javascript
var obj = {
    '1p': "Hello World",
    'h w': "Hello World",
    'p+q': "Hello World"
};
```
上面对象的三个属性，都不符合标识名的条件，所以**必须加上引号**。

> JavaScript 的保留字可以不加引号直接当作对象的属性。

**键值**

键值是属性所对应的具体的值。javascript 的对象的键值可以是任何数据类型。

```javascript
let Jackson = {
    name: "Jackson",
    age: 18,
    sex: "male",
    ability: eat(); //eat()表示函数
};
```

如果一个属性的值(ability)为函数，通常把这个属性称为**方法**。

```javascript
//json
var obj = {
    "name":"孙悟空",
    "sex":"石",
    "from":"花果山",
    '3w':"www.sunwukong.com",
    'a#b':'123',
    f1:function(){
        alert("ok!");
    }
}
    //方法
    console.log(obj.f1());
```


# 二、对象的引用

## 1、对象属性的读取和设置
 
读取对象的属性,有两种方法:

一种是使用点运算符`.`,还有一种是使用方括号运算符`[]`。

需要注意的是，使用方括号读取对象属性的时候需要加引号点运算符用来为对象的属性写入值。

`[]` 的使用总结说明：

- （1）可以使用一个变量存储对象的属性，`.`是不能使用的。
- （2）可以使用纯数字的方式来访问，`.`是不能使用的。

```javascript

```javascript
//json
var obj = {
    "name":"孙悟空",
    "sex":"石",
    "from":"花果山",
    '3w':"www.swk.com",
    'a#b':'123',
    '555':'12345',
    f1:function(){
        alert("ok!");
    }
}
var char = "from";
console.log(obj.char);//undefined
console.log(obj[char]);//花果山

//console.log(obj.555); //不行
console.log(obj[555]);
```

`.` 的使用的总结说明: 

 - 点（.）运算符可以将 js 的关键字（var,if 等）作为属性来访问。


```javascript
//json
var obj = {
    "name":"孙悟空",
    "sex":"石",
    "from":"花果山",
    '3w':"www.swk.com",
    'a#b':'123',
    '555':'12345',
    f1:function(){
        alert("ok!");
    }
}

obj.var = "var";
obj.if="如果";
console.log(obj.if);
//console.log(obj[if]); 不行
```



## 2、对象属性的操作

常见如下：

- Object.keys() 获取对象所有属性
- Object.values() 获取对象所有的值
- Object.entries() 获取对象所有的键值对
- delete 删除一个属性
- in 检查对象是否包含一个属性（true，false）
- for in 遍历对象所有属性
- for of 遍历对象所有属性


```javascript
//json
var obj = {
    "name":"孙悟空",
    "sex":"石",
    "from":"花果山",
    '3w':"www.swk.com",
    'a#b':'123',
    '555':'12345',
    f1:function(){
        alert("ok!");
    }
}

//object.keys():对象的键名
for(var x of Object.keys(obj)){
    console.log(x, obj[x]);
}

//object.values()：对象的键值
for(var x of Object.values(obj)){
    console.log(x);
}

//object.entries()：对象的键值对
for(var x of Object.entries(obj)){
    console.log(x);
}

delete obj[555];//删除属性
// console.log(obj);

console.log("name"in obj);//true
console.log("address" in obj);//false
```

for 遍历

```javascript
//json
var obj = {
    "name":"孙悟空",
    "sex":"石",
    "from":"花果山",
    '3w':"www.swk.com",
    'a#b':'123',
    '555':'12345',
    f1:function(){
        alert("ok!");
    }
}

// x 是键，是属性
for(var x in obj){
    console.log(x, obj[x]);
}

// x 是键，是属性, 其他情况参考前一个案例
for(var x of Object.keys(obj)){
    console.log(x, obj[x]);
}
```

## 3、对象排序

数组排序:

```javascript
sort(function(a,b){
    return a - b;
});
```

按年龄排序，年龄一样时，按工资排序

```javascript
var person1 = {name:"小米", age:25, salary:9000};
var person2 = {name:"小明", age:21, salary:8000};
var person3 = {name:"小汤", age:20, salary:6000};
var person4 = {name:"小丽", age:22, salary:15000};
var person5 = {name:"小小", age:22, salary:17000}; 

var arr = [person1, person2, person3, person4, person5];
arr.sort(function(a,b){
    //如果年龄相等按薪水排序
    if (a.age == b.age) {
        return (a.salary-b.salary);
    }
    //默认按年龄排序
    return a.age - b.age;
});

console.log(arr);
```

# 常用内置对象

## Math 对象

Math 对象是数学对象

Math 对象的属性：PI（圆周率）

Math 对象的方法：
- random(): 随机函数(0-1)
- floor(): 向下取整
- ceil(): 向上取整
- round(): 四省五入取整
- pow(): 求一个数的幂数
- max(): 求最大值
- min(): 求最小值
- abs(): 绝对值
- sqrt(): 求平方根

```html
<script>
console.log(Math.PI);//圆周率

console.log(Math.floor(6.2));//6 向下取整
console.log(Math.floor(6.7));//6

console.log(Math.ceil(6.2));//7 7/向上取整
console.log(Math.ceil(6.7));//7

console.log(Math.round(6.2));//6 四舍五入取整
console.log(Math.round(6.7));//7

console.log(Math.round(6.2));//6 四舍五入取整
console.log(Math.round(6.7));//7

console.log(Math.pow(5,3));//125求一个数的幂数

console.log(Math.max(92,56,15,65,73,50));//最大数

console.log(Math.min(92,56,15,65,73,50));//最小数

console.log(Math.abs(-95));//绝对值

console.log(Math.sqrt(8));//平方根
</script>
```

取一个范围的随机数: 公式**`parseInt((max-min+1)*Math.random()+min)`**

```javascript
//30-80的随机数
// parseInt((max-min+1)*Math.random()+min)
console.log(parseInt((80-30+1)*Math.random()+30));
```

## Date 对象

Date 对象是 JavaScript 提供的日期和时间的操作接口。

在 JavaScript 内部，所有日期和时间都储存为一个整数。

这个整数是当前时间距离 1970 年 1 月 1 日 00:00:00 的毫秒数，正负的范围为基准时间前后各 1 亿天

```html
<script>
console.log(Date(),typeof Date());// 日期时间对象   string
console.log(new Date(),typeof new Date());//日期时间对象  object 
</script>
```


**Date 函数**

Date 对象是一个构造函数，对它使用 new 命令，会返回一个Date 对象的实例。


一些其他合法的日期字符串写法：

- new Date(datestring)

```javascript
new Date("2024-9-15")
new Date('2024/9/15')
new Date("2024-FEB-15")
new Date("FEB, 15, 2024")
new Date("FEB 15, 2024")
new Date("Feberuary, 15, 2024")
new Date("Feberuary 15, 2024")
new Date("15, Feberuary, 2024")
//Sun Jan 06 2024 00:00:00 GMT+0800 (中国标准时间)
```

**Date 的方法**

- getTime():获取距离 1970 年 1 月 1 日的毫秒数
- getYear():获取年份（距离 1900 的年数）
- getFullYear()：获取全年（4 位数）
- getMonth()：获取月份（0-11）
- getDate()：获取日期
- getDay()：获取星期几（0-6）,0：星期日，6：星期六
- getHours()：获取小时（0-23）
- getMinutes()：获取分钟（0-59）
- getSeconds()：获取秒数（0-59）
- toLocaleString()：获取当地的日期和时间
- toLocaleDateString()：获取当地的日期
- toLocaleTimeString():获取当地的时间

```html
<script>
var date = new Date(); //创建一个日期时间对象

console.log(date.getYear());//获取年份
console.log(date.getFullYear()）;//获取4位数的年份
console.log(date.getMonth()+1);//获取月份（0-11)
console.log(date.getDate());//获取日期
console.log(date.getDay()));//获取星期（0-6)
console.log(date.getHours());//获取小时
console.log(date.getMinutes());//获取分钟
console.log(date.getSeconds());//获取秒数
console.log(date.toLocaleString());//获取当地的日期和时间
console.log(date.toLocaleDateString());//获取当地的日期
console.log(date.toLocaleTimeString());//获取当地的时间
</script>
```

显示当前的日期时间和星期

```html
<script>
var date = new Date();//创建一个日期时间对象
var year = date.getFullYear();//获取年
var month = date.getMonth()+1;//获取月
var mydate = date.getDate();//获取日期
var day = date.getDay();//获取星期(0-6)

switch(day){
    case 0 :
        myday="星期日";break;
    case 1:myday = "星期一";break;
    case 2:myday = "星期二";break;
    case 3:myday = "星期三";break;
    case 4:myday = "星期四";break;
    case 5:myday = "星期五";break;
    default: myday="星期六";
}

console.log("今天是："+year+"年"+month+"月"+mydate+"日"+"&nbsp;&nbsp;"+myday);
</script>
```

距离新中国成立多少年


```html
<script>
var minutes = 1000*60 ;
var hours = minutes*60 ;
var days = hours*24 ;
var years = days*365 ;
var d = new Date();
var t = d-new Date("1949/10/01");
var y = t/years ;
console.log("It's been: " + y + " years since 1970/01/01!")
</script>
```

产品剩余几天几小时几分几秒

```html
倒计时：<div></div>
<script>
var odiv = document.querySelector("div");//获取div元素
//剩余的天数，小时，分钟，秒数
var day,hour,minute,second;
var date = new Date(); //获取当前的日期
var fdate = new Date("2034-9-13");//获取未来的日期
var time =(fdate-date)/1000;//获取的是秒数

setInterval(function(){
    //21.58953
    day = Math.floor(time/60/60/24);//获取剩余的天数
    //hour = 总的小时数－ 21天的小时数
    hour =Math.floor(time/60/60)-day*24;//获取剩余的小时数
    minute = Math.floor(time/60)- day*24*60 - hour * 60;//获取剩余的分钟数
    second = Math.floor(time)- day*24*60*60 - hour * 60 * 60 - minute*60;//获取剩余的
    
    odiv.innerHTML ="days:"+day+"hours:"+hour+"minutes: "+minute+"seconds:"+second;
    time--;
},1000);
</script>
```

