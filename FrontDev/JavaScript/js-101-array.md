---
layout: default
title: Js Array
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 11
---

 Here are JavaScript Array examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、Array 简介

## 数组

数组就是按照一定的顺序排列的一组值，每个值都有自己的编号，编号(下标)从 0 开始，整个数组用可以用[]表示。

定义数组的方式：

方式 1：中括号[]
```
arr = [数组的值 1，数组的值 2，数组的值 3,....];
```

方式 2：使用 new Array()

本质上，数组是对象类型的一种特殊表现形式。因此创建的时候我们可以使用 new 方式来创建。

typeof 运算符会返回数组的类型是 object。
```
arr = new Array(数组的值 1，数组的值 2，数组的值 3,....);
```

## 2、数组分类

- 按维度分：分一维数组，二维数组等。

- 按类型分：索引数组和关联数组。



## 3、数组特点

数组就是一组数据（数字，字符串，对象）类型的集合，简单来说数组就是一种容器。

1. 数组内的数据可以是任意的类型。

2. 数组下标从 0 开始。

3. 数组的长度就是数组元素的个数(length)。

4. 数组下标的范围是 0-length-1。

 
# 二、数组的使用

## 1、数组的地址传递
 
普通变量是值传递,其中的一个改变不会影响到另外一个值。因为各自都有自己的地址。

数组是地址传递：其中的一个改变，会影响另一个的改变。因为他们共用一个地址。

[]:相当于（new Array）在堆区开辟了一个内存空间。


## 2、数组的遍历

遍历：循环访问数组元素的值。



```html
<html>
<body>
<div class="div1">测试一下</div>
<button>单击</button>
<div class="div2"></div>
<script type="text/javascript">

var fruits = ["cucomber","peach","plum"];
var i = 0;
while(i<fruits.length){
    console.log(fruits[i]);
    i++;
} 

var person = [];
person["name"] = "刘秀";
person["sex"] = "猛男";
person["age"] = 65;
// i 数组的下标
for(var i in person){
    console.log(i,person[i]);
}

// val 数组元素的值
for(var val of fruits){
    console.log(val);
}

//keys()：返回数组的下标
for(var k of fruits.keys()){
    console.log(k);
}

//values()：返回数组元素的值
for(var v of fruits.values()){
    console.log(v);
}


//entries()：返回数组元素的键名和键值
// v 包含索引和值 [0, "cucomber"]
for(var v of fruits.entries()){
    console.log(v);
}

</script>
</body>
</html>
```

## 3、数组的基本操作


- push(): 向数组的尾部添加数据
- unshift(): 向数组的头部添加数据
- pop(): 删除尾部的数据
- shift()：删除头部的数据



```html
<script>
var fruits = ["cucomber","peach","plum"];
console.log(fruits.push("apple","orange"));//向后添加数据-返回值是5
console.log(fruits.unshift("straw","berry"));//向前添加数据-返回值是7
console.log(fruits);//删除前
console.log(fruits.pop());//删除最后一个数组元素
console.log(fruits.shift());//删除第一个数组元素
console.log(fruits);//删除后
</script>
```


splice(from,howmany,items): 添加删除数据。
- From:从哪个下标开始添加还是删除
- Howmany:删除或者添加几个数组元素
- Items:添加的数据或者是修改的数据。

1.删除数组元素：

```html
<script>
var fruits = ["cucomber","peach","plum","apple","orange"];
console.log(fruits);// 操作前 "cucomber","peach","plum","apple","orange"
console.log(fruits.splice(1,2));//返回删除的数组 "peach","plum"
console.log(fruits);//操作后  "cucomber", "apple","orange"
</script>
```

2.删除数组元素并替换数组元素


```html
<script>
var fruits = ["cucomber","peach","plum","apple","orange"];
console.log(fruits);// 操作前 "cucomber","peach","plum","apple","orange"
console.log(fruits.splice(1,2, "grape","pineapple"));//返回删除的数组 "peach","plum"
console.log(fruits);//操作后  "cucomber", "grape","pineapple", "apple","orange"
</script>
```

3.添加数组元素

```html
<script>
var fruits = ["cucomber","peach","plum","apple","orange"];
console.log(fruits);// 操作前 "cucomber","peach","plum","apple","orange"
console.log(fruits.splice(1,0, "grape","pineapple"));//返回删除的数组 "peach","plum"
console.log(fruits);//操作后  "cucomber", "grape","pineapple","peach","plum", "apple","orange"
</script>
```


## 4、数组的方法


length:数组的长度

length 是一个可写属性。

如果设置 length 长度小于数组本身长度，那么多余元素舍弃。

如果设置 length 长度大于数组本身长度，那么缺少元素用空位补齐。

如果设置 length 长度不是合法数值，那么会报错 `Invalid array length` 。


```html
<html>
<body>
<script type="text/javascript">
var fruits = ["cucomber","peach","plum","apple","orange"];
console.log(fruits.length);// 5

fruits.length = 7;
console.log(fruits);// 有2个空位

</script>
</body>
</html>
```

数组的空位

当数组的某个位置是空元素，即两个逗号之间没有任何值，我们称该数组存在空位(hole)。

```javascript
var countryNameArr = ['China','','Japan'];
countryNameArr.length //3
```

**数组的方法**


- 1.concat(arr1,arr2,....):连接多个数组
- 2.push() 向数组的尾部添加数据
- 3.pop()  删除尾部的数据
- 4.shift()  删除头部的数据
- 5.unshift() 向数组的头部添加数据
- 6.join():给数组添加一个分隔符并将数组转化为字符串
- 7.reverse()：倒叙输出数组元素
- 8.slice(start,end)：数组的截取
- 9.splice():添加修改删除数组元素
- 10.sort()：数组的排序。按照字符（a-z）的顺序排序。
- 11.map()：循环遍历数组，有返回值
- 12.forEach()：循环遍历数组，没有返回值
  - 语法：`forEach(function(val,index,arr){  })`
  - val:数组元素的值
  - index:下标
  - arr:数组本身
- 13.filter():找到符合条件的所有元素
- 14.find():找到符合条件的第一个元素
- 15.findIndex():找到符合条件的第一个元素的下标
- 16.some()：只要有一个符合条件的就返回 true
- 17.every()：只要有一个不符合条件的就返回 false
- 18.includes():判断是否包含指定的数组元素，有就是 true,没有就是 false 。
- 19.reduce():数组元素的计算（从左到右）
- 20.reduceRight():数组元素的计算（从右到左）
- 21.indexOf()：返回指定数组元素的首次出现的下标
- 22.lastIndexOf()：返回指定数组元素的最后一次出现的下标



join & reverse & slice 示例

```html
<script>
// join
var fruits =["cucomber","peach","plum","apple","orange"];
console.log(fruits);
console.log(fruits.join("-"));//给数组添加分隔符并将数组转化为字符串

// reverse 转至
console.log(fruits.reverse());//倒叙输出数组
// slice 截取
console.log(fruits.slice(1,4));//截取数组元素-不包含最后一个下标
</script>
```

sort 排序示例

```html
<html>
<body>
<script type="text/javascript">
var fruits =["cucomber","peach","plum","apple","orange"];
console.log(fruits);//排序前
console.log(fruits.sort());//排序后

var score = [98,85,99,78,65,23,83,100,7];
console.log(score);
//自定义一个排序函数：//冒泡排序法：两个相邻的数进行比较，如果前面的数比后面的数大就交换
function paixu(a,b){
    if(a>b){
        return 1;
    }else if(a<b){
        return -1;
    } else{
        return 0;
    }
}
console.log(score.sort(paixu));
</script>
</body>
</html>
```

foreach & map 示例

```html
<script>
var fruits =["cucomber","peach","plum","apple","orange"];
//循环遍历数组，它没有返回值-undefined 
var re = fruits.forEach(function(val){
    return val;
});
console.log("forEach:"+re);

//循环遍历数组，它有返回值
var re2 = fruits.map(function(val){
    return "I want eat "+val;
}
console.log("map:"+re2);

//循环遍历数组
fruits.map(function(val){
    console.log("newmap:"+val);
});

var score = [98,85,99,78,65,23,83,100,7];
//map：有返回值，返回一个新的数组
var re = score.map(function(val){
    return val*2 ;
});
console.log(re);

//数组元素的值 val  下标 index   arr数组本身
fruits.forEach(function(val,index,arr){
    console.log(val,index,arr);
});
</script>
```

find & filter & findIndex 示例

```html
<script>
//find:找到符合条件的第一个数组元素
var re= score.find(function(val,index,arr){
    return val>85;
});
console.log(re);
//filter：找到符合条件的所有数组元素
var re2 = score.filter(function(val,index,arr){
    return val>85;
});

//findIndex:找到符合条件的第一个数组元素的下标
var re3 = score.findIndex(function(val,index,arr){
    return val>85;
});
console.log(re3);

</script>
```

some & every & includes 示例

```html
<script>
var score=[85,99,78,65,23,83,100,7,98];
//只要有一个符合条件的就返回true
var re= score.some(function(val,index,arr){
    return val>85;
});
console.log(re);
//只要有一个不符合条件的就返回faLse
var re2=score.every(function(val,index,arr){
    return val>85;
});
console.log(re2);

var users = ["jack","tom","peter"];
//判断是否包含指定的数组元素，有就是true，没有就是faLse
var re = users.includes("xiaocai");
console.log(re);//false

</script>
```

reduce 示例

```html
<script>
var score=[1,2,3,4,5];
//数组元素的计算（从左到右）
// total：总计，vaL：数组元素的值
var re = score.reduce(function(total,val){
    return total + val;
});
console.log(re);

//数组元素的计算（（从右到左）
var re2 = score.reduceRight(function(total,val){
    return total - val;
});
console.log(re2);
</script>
```

indexOf lastIndexOf

```html
<script>
var fruits = ["cucomber","peach","plum","apple","peach","orange"];
console.log(fruits.indexOf("plum"));//2
console.log(fruits.lastIndexOf("peach"));//4
</script>
```

# 三、二维数组

## 1、定义
如果数组的元素还是数组，那么我们就称外层数组是一个二维数组。
 
语法：
```
var arr = [[item1,item2],[item3,item4]];
```

示例

```html
<script>
var arr = [[1,2],[3,4]];
arr[0][1] = 2;
arr[1][0] = 3; 
```

## 2、二维数组的遍历

```html
<script>

var person = [
["Jack", "Tome", "Jerry", "Lucy"]
["张三", "李四", "王五", "赵六"]
["诸葛亮", "赵云", "关羽", "张飞"]
];

console.log(person[1][2]);

for (var i = 0 ; i < person.length ; i++){
    for (var j= 0; j < person[i].length ; j++){
         console.log(person[i][j]);
    }
}
</script>
```

## 3、一些排序算法

