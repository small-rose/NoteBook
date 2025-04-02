---
layout: default
title: Js Base
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 10
---

 Here are JavaScript examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、Javascript 简介

## js 的前世今生

JavaScript 一种直译式脚本语言，是一种动态类型、弱类型、基于原型的语言，最早是在 HTML 网页上使用，用来给 HTML 网页增加动态功能。

动态：在运行时确定数据类型。变量使用之前不需要类型声明，通常变量的类型是被赋值的那个值的类型。

弱类：计算时可以不同类型之间对使用者透明地隐式转换，即使类型不正确，也能通过隐式转换来得到正确的类型。

原型：新对象继承对象（作为模版），将自身的属性共享给新对象，模版对象称为原型。这样新对象实例化后不但可以享有自己创建时和运行时定义的属性，而且可以享有原型对象的属性。

脚本语言：不需要编译器编译。

> 在 1995 年 时 ， 由 Netscape 公 司 的 布 兰 登 · 艾 奇 （ Brendan Eich，1961 年～），JavaScript 的发明人，在网景导航者浏览器（Navigator）上首次设计实现而成。
>
>由于网景公司希望能在静态 HTML 页面上添加一些动态效果，于是叫Brendan Eich 这哥们在两周之内设计出了 JavaScript 语言。你没看错，这哥们只用了 10 天时间。
>

为什么起名叫 JavaScript？

原因是当时 Java 语言非常红火，所以网景公司希望借 Java 的名气来推广，但事实上 JavaScript 除了语法上有点像 Java，其他部分基本上没啥关系。

Netscape 在最初将其脚本语言命名为 LiveScript，后来 Netscape 在与 Sun 合作之后将其改名为 JavaScript。

Java Script 的三个主要组成部分是：ECMAScript(核心)，DOM（文档对象模型），BOM（浏览器对象模型）。


## 2、js 的特点


1.是一种解释性脚本语言（代码不进行预编译）。

2.主要用来向 HTML（标准通用标记语言下的一个应用）页面添加交互行为。

3.可以直接嵌入 HTML 页面，但写成单独的 js 文件有利于结构和行为的分离。

4. 跨平台特性，在绝大多数浏览器的支持下， 可以在多种平台下运行（如Windows 、Linux、Mac 、Android、iOS 等

5.它是**单线程编程**语言。
 

## 3、js 的应用

表单验证、购物车、放大镜

ajax  异步交互


## 4、ECMAScript 和 JavaScript 的关系



1996 年 11 月，JavaScript 的创造者 Netscape 公司，决定将 JavaScript 提交给标准化组织ECMA，希望这种语言能够成为国际标准。

因为网景开发了 JavaScript ， 一年后微软模仿 JavaScript 开发了JScript，为了让 JavaScript 成为全球标准，几个公司联合 ECMA（EuropeanComputer Manufacturers Association ）组织定制了 JavaScript 语言的标 准 ， 被称为 ECMAScript 标准。

简单说来就是， ECMAScript 是一种语言标准 ， 而 JavaScript 是网景公司对 ECMAScript 标准的一种实现。

那为什么不直接把 JavaScript 定为标准呢？

因为 JavaScript 是网景的注册商标。不过大多数时候，我们还是用 JavaScript 这个词。

如果遇到 ECMAScript 这个词，简单把它替换为 JavaScript 就行了。

由于 JavaScript 的标准——ECMAScript 在不断发展，最新版 ECMAScript 6 标准（简称ES6，有时也被称为 ES2015）已经在 2015 年 6 月正式发布了，所以，讲到 JavaScript 的版本，实际上就是说它实现了 ECMAScript 标准的哪个版本。


## 5、Javascript 发展历史

1995 年 12 月 4 日 Netscape 公司与 Sun 公司联合发布了 JavaScript 语言。
1996 年 03 月 Navigator 2.0 浏览器正式内置了 JavaScript 脚本语言。
1997 年 07 月 ECMAScript 1.0 发布。
1998 年 06 月 ECMAScript 2.0 版发布。
1999 年 12 月 ECMAScript 3.0 版发布，成为 JavaScript 的通行标准，得到了广泛支持。
2007 年 10 月 ECMAScript 4.0 版草案发布
2009 年 12 月 ECMAScript 5.0 版正式发布
2015 年 06 月 ECMAScript 6 正式发布


## 6、主要浏览器内核和引擎

一个完整的浏览器包含浏览器内核和浏览器的外壳(shell)。浏览器核心——内核分成两部分：渲染引
擎和 js 引擎。
浏览器内核主要指的是 浏览器 的渲染引擎，2013 年以前， 代表 有Trident（IE），Gecko（firefox），Webkit（Safari chrome 等）以及 Presto（opera)。

2013 年以后，谷歌开始研发 blink 引擎，chrome 28 以后开始使用，而 opera 则放弃了自主研发的 Presto 引擎，投入谷歌怀抱，和谷歌一起研发 blink 引擎，国内各种 chrome 系的浏览器（360、UC、QQ、2345 等等）也纷纷放弃 webkit，投入 blink 的怀抱。


1、IE 浏览器内核：Trident 内核，也是俗称的 IE 内核；

2、Chrome 浏览器内核：统称为 Chromium 内核或 Chrome 内核，以前是 Webkit 内核，现在是 Blink 内核；

3、Firefox 浏览器内核：Gecko 内核，俗称 Firefox 内核；

4、Safari 浏览器内核：Webkit 内核；

5、Opera 浏览器内核：最初是自己的 Presto 内核，后来是 Webkit，现在是 Blink 内核；

6、360 浏览器、猎豹浏览器内核：IE+Chrome 双内核；

7、搜狗、遨游、QQ 浏览器内核：Trident（兼容模式）+Webkit（高速模式）；

8、百度浏览器、世界之窗内核：IE 内核；

9、2345 浏览器内核：以前是 IE 内核，现在也是 IE+Chrome 双内核；



# 二、js 的语法结构

## 1、js 引入

- 1、内部方式
- 2、外部结构
- 3、行内方式

```html
<html>
<body>
<!-- 行内方式 -->
<button onclick="f1();f2();">单击</button>

<!-- 外部部方式 -->
<script src="https://code.jquery.com/jquery-2.2.4.js"
			  integrity="sha256-iT6Q9iMJYuQiMWNd9lDyBUStIq/8PuOW33aOqmvFpqI="
			  crossorigin="anonymous"></script>

<!-- 内部方式 -->
<script type="text/javascript">
var obt=document.querySelector("button");
// 方式2 直接写函数
obt.onclick = function(){
    f1();
}

// 方式3 直接写函数名
//obt.onclick = f1 ;

function f1(){console.log("f1");}
function f2(){console.log("f2");}
</script>
</body>
</html>
``` 
 
## 2、js 输出

4种输出方式

```html
<html>
<body>
<div class="div1">测试一下</div>
<button>单击</button>
<div class="div2"></div>
<script type="text/javascript">
    // 方式1: document.write
    document.write("<div style='color:red'>测试一下红色！</div>");
    // 方式2: window.alert
    window.alert('test');
    // 方式3: innerHTML 
    //获取按钮元素
    var obt = document.querySelector("button");
    //获取div2元素
    var odiv = document.querySelector(".div2");
    obt.onclick = function(){
        odiv.innerHTML = "我来了！";
    }
    // 方式4: console.log 
    console.log("我在浏览器控制台");

</script>
</body>
</html>
```

## 3、js 的语法规则


(1) 分号是语句结束的标志，分号不是必须的,我们不建议这样做，严格来说，语句要加上分号。

(2) js 会忽略多个空格和换行。

(3) 字符集

- utf-8:统一国际编码，兼容各个国家的语言
- gb2312/gbk:简体中文编码
- big5:繁体中文编码

(4) 变量区分大小写。

(5) 注释 。

html 注释：
```
<!--注释内容-->
```
js 、css 的注释：
```
/*内容*/
```

## 4、Javascript 变量

一般固定值称为**字面量**，如 3.14。给变量赋值时，等号右边都可以认为是字面量。英语叫做 literals，有些书上叫做直接量。

- 数字字面量 25,98.23， Var num = 92

- 字符串字面量，如: ‘123’，“Hello”

- 常量: 值不能改变的。

- 变量: 值可以被改变的。变量是用来存储数据的。那存储什么种类数据呢？声明变量使用关键字：var(variable) 你给它赋什么类型的值，那么这个变量就是什么数据类型。
    - 统一的声明关键字，类型由值决定。var、let、const
    - 单独声明
    - 多个变量声明，中间用逗号隔开。
    - 相同变量允许重复声明。
    - 遗漏声明会提示为定义。
    - 显式声明和隐式声明
    - 隐式声明的变量可以删除
    
内存中堆区和栈区： 一般基本类型在栈区，object 类型在堆区。

```html
<html>
<body>
<script type="text/javascript">
    var a = 25; 
    let str1 = "abc", str2="def"; 
    let str = 'Hello'; 
    const word = 3.14;
    
    a = 100;
    str2 = 'Hello world'; 
    var str = 303 ; //重复声明

    // JS 中变量声明分显式声明和隐式声明。
    var name1 = 'xiaocai';//显示声明
    name2 = 'xiaocai';// 隐式声明 （ 为全局对象（window）的一个属性）
    delete name2 ;

    function fun1(){
        console.log(scope);//全局变量。undefined
        var scope1 = "局部变量"; //变量提升的问题
        console.log(scope);//局部变量
    }
    fun1();

     var scope = "全局变量";
     function fun2(){
         console.log(scope);//全局变量
         scope = "局部变量"; // 隐式声明
         console.log(scope);//局部变量
     }
     fun2();   
</script>
</body>
</html>
```

> 局部变量的提升仅限于变量声明，即 var 关键字声明的变量。如果使用 let 或 const 声明的变量，则不会发生提升，这被称为“暂时性死区”（temporal dead zone）。

Const 也是块级作用域。

const 声明的常量必须初始化，而 let 声明的变量不用。

const 实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。

对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。

但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const 只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制。


**变量的命名**

变量是由字母，数字，下划线，$组成，但第一个字符必须是字母或者是下划线，$开头。

> $:不建议使用，它可能会和其他的框架语法冲突，或者是函数的名字冲突。

（1）JavaScript 语言的标识符对大小写敏感，所以 a 和 A 是两个不同的标识符。

（2）首字母可以是任意字母以及美元符号和下划线。剩余可以是任意字母，美元符号，下划线和数字。

（3）不能使用 javascript 中的关键字(保留字)来命名变量

（4）中文也可以声明变量，不建议使用它。


**let 和 var 的区别**

var： variable,它是可变的。

let： 块作用域。

```html
<html>
<body>
<script type="text/javascript">
     function fun3(){
         let  scope = "局部变量"; // 隐式声明
         console.log(scope);//局部变量
     }
     fun3(); 
    console.log(scope);//全局变量  
</script>
</body>
</html>
```

**使用var和不使用var的区别(全局变量/局部变量)**

1、在function内部， 加var的是局部变量， 不加var的则是全局变量；

2、在function外部， 不管有没有使用var声明变量，都是全局变量。



## 5、JavaScript 严格模式(use strict)

声明关键字：`"use strict";`

使用严格模式的变量一定要先声明，后使用。

变量提升：变量一定要先声明后使用，如果先使用后声明，js 的内部机制自然使变量提升。



# 三、Js数据类型


- 基本数据类型
    - 数值
    - 字符串
    - 布尔。
    - null
    - undefined
    
- 引用类型 object
    - array
    - function
    - 日期
    
增加了一个基本数据类型：Symbol，Symbol 是 ES6 引入了一种新的原始数据类型，表示独一无二的值。

打印变量的数据类型：typeof

请注意：
- NaN 的数据类型是 number。 NaN: Not a Number: 不是一个数值。
- 数组(Array)的数据类型是 object
- 日期(Date)的数据类型为 object
- null 的数据类型是 object
- 未定义变量的数据类型为 undefined
    
```html
<script> 
var num = 0b01110110;//二进制
var num2 = 0o56; //八进制
var num3 = 0xfab23562;//十六进制
var num4 = 98; //十进制
console.log(num4.toString(2)); //十进制转换二进制
console.log(num4.toString(8));// 十进制转换/进制
console.log(num4.toString(16));//十进制转换十六进制
console.log(parseInt(num,10));//二进制转换十进制
console.log(parseInt(num2,10)); ///(进制转换十进制
console.log(parseInt(num3,10));//十六进制转换十进制一
//最大值：MAX_VALUE
//最小值：MIN_VALUE
console.log(Number.MAX_VALUE);//数值类型的最大值
console.log(Number.MIN_VALUE);//数值类型的最小值
//浮点数不能比较
console.log(0.08 == 1-0.92);
console.log(1-0.92);
console.log("a"*9); // NaN
console.log(NaN == NaN); // false
console.log(NaN === NaN); // false
console.log(2/0);//Infinity
</script>
```
> c 语言中：
>
>  int(整数), float(单精度浮点型), double(双精度浮点型)，char(字符), string(字符串)。
>
>  二进制(binary)：0b101010101
>
>  八进制：0o2535
>
>  十六进制：0x69852
> 
> 进制转换:
>  - toSring():十进制转换其他的进制
>  - parseInt():其他的进制转换十进制


**布尔类型（boolean）**

true(真),false（假），都是小写的。TRUE,FALSE,True,False:这些都不正确。

**字符串类型(string)**

字符串：加单引号或者是双引号。需要原样输出引号的需要带转义符"\\"


```html
<script> 
var str = "我已经是\"王者\"了";
console.log(str); // 我已经是"王者"了
console.log("7"==7);//true
console.log("7"===7);//false

// 引用类型
let num = new Number(303);
let str = new String('hello');
var arr = [1,2,3,4,5];
var person = {
    name : 'xiaocai',
    nice : true
};
console.log(typeof person);//object
</script>
```

**undefined** 和  **null**

null 和 undefined 的区别

- 1.类型不相等
- 2.强制类型转换值不一样
- 3.undefined 和 null 在值上看一样，但是属于不同的类型。

```html
<script> 
// 类型区别
console.log(typeof undefined);//undefined
console.log(typeof null);//object

// 强转
var rel = Number (undefined) ;//NaN
var re2 = Number(null) ;//0
console.log(rel);//NaN
console.log(re2;//0


//比较
console.log(undefined == null);//true
console.log(undefined === null);//false
</script>
```


# 四、Js 运算符

## 1、基本运算符


`+`，`-`，`*`，`/`，`%（求余数）`，`++`，`--`，`**（求幂数-es7 新增）`

‌‌i++和‌++i的主要区别在于赋值顺序和返回值。‌

- ‌赋值顺序‌：
   - ‌i++（后缀自增运算符）‌：先将变量的当前值返回，然后将变量的值增加1。
   - ‌++i（前缀自增运算符）‌：先将变量的值增加1，然后将增加后的值返回。

- ‌返回值‌：
   - i++返回变量增加前的值。
   - ++i返回变量增加后的值。

示例说明

假设变量 i 的初始值为 5 ：
- 使用 i++ 后，i 的值变为 6，但表达式的结果为 5（因为先返回了 i 的原始值，然后 i 才自增）。
- 使用 ++i 后，i 的值变为 6，表达式的结果也为 6（因为先自增，然后返回新的值）。

在循环中的表现

在循环中，i++ 和 ++i 的表现也有所不同：

    使用 i++ 时，循环的判断条件使用的是变量增加前的值。
    使用 ++i 时，循环的判断条件使用的是变量增加后的值。

```html
<script> 
var num1 = 1;
var num2 = 2;
var num3 = 3;
var num4 = 4;

console.log(num1 + num2);
console.log(num2 - num1);
console.log(num2 * num3);
console.log(num4 / num2);

console.log("num4 % num3 = " + (num4 % num3));
console.log(num2 - num1);
console.log(num2 * num3);
console.log(num4 / num2);

var i = 5;

// ++ 在前
console.log(i++); //5
console.log(i); //6
console.log(++i); //7
console.log(i);  //7

for(var i = 0; i < 3; ) {
   i++
   console.log(i); 
}
for(var i = 0; i < 3; ) {
    ++i
   console.log(i);
}
</script>
```
 
## 2、比较（关系）运算符

`>`,`<`,`>=`,`<=`,`!=`,`!==（不全等）`,`==`,`===(全等：值和类型都相等)` 。


示例:

```html
<script> 
var num1 = 1;
var num2 = 2;
var num3 = 3;
var num4 = 5;
var str4 = "5";
var str5 = "5";

console.log(num1 > num2);
console.log(num2 >= num1);
console.log(num2 < num3);
console.log(num4 <= num2);

console.log("num4 != str4 = " + (num4 != str4));
console.log("num4 !== str4 = " + (num4 !== str4));
console.log(num4 == str4);
console.log(str4 == str5);
console.log(num4 === str4);
console.log(str4 === str5);
</script>
```


## 3、逻辑运算符

- `||` : 逻辑或，有一个为 true, 则结果为true 。
- `&&` : 逻辑或，有一个为 false, 则结果为 false 。

示例:

```html
<script>
console.log(10 && 'js');//js
console.log(0 && 'abc');//0
console.log(10 || 'js');//10
console.log(0 || 'abc');//abc

//怎样输出：true 和 false
console.log(Boolean(0) && Boolean('abc'));

</script>
```
逻辑短路:

- 对于逻辑与(`&&`）运算符第一个表达式为假就会发生短路,即逻辑`&&`后面的表达式不会执行运算。
- 对于逻辑与(`||`）运算符第一个表达式为真就会发生短路,即逻辑`||`后面的表达式不会执行运算。

```html
<script>
var score = 98 ;
var a = 5,   b = 7;

//faLse：短路，对于逻辑与(&&）运算符第一个表达式为假就会发生短路
// 后面的表达式不会运算
console.log(Boolean(null) && ++a);
//true:短路，对于逻辑或(ll)运算符第一个表达式为真就会发生短路
// 后面的表达式不会运算
console.log(score>=90 || ++b);
console.log("a="+a);//a = 5
console.log("b="+b);//b = 7
</script>
```


## 4、赋值运算符

`=`,`+=`,`-=`,`*=`,`/=`,`%=`

`=`: 赋值

`==`：比较（等于）

`===`：比较（全等）

```html
<script>
var score = 98 ;
var a = 5,   b = 7;
a += 2; // a = a +2 
b -= 3; // b = b - 3

console.log("a="+a);
console.log("b="+b);

a *= 2; // a = a * 2 
b /= 3; // b = b / 3

console.log("a="+a);
console.log("b="+b);

b %= 3; // b = b % 3

console.log("a="+a);
console.log("b="+b);
</script>
```

## 5、条件运算符（三目运算符）

语法：

```
表达式 1?表达式 2:表达式 3
```
解释：如果表达式 1 为真(true),计算表达式 2 的值，如果为假(false),计算表达式 3 的值。


## 6、等性运算符

- Null==undefined //true
- Null===undefined //false
- true == 1;
- false ==0;
- NaN == NaN //false
- NaN !== NaN //true


## 7、运算符的优先级

下表按照从高到低优先级列出，相同优先级从左到右运算。


| 运算符 | 描述  |
|-------|------|
| .[]()  | 字段访问。数组下标函数调用以及表达式分组 |
| ++ -- - ~ ! delete new typeof void | 一元运算符。返回数据类型对象创建未定义值 |
| * / % | 乘法、除法、取模 |
| + - + | 加法、减法。字符串连接 |
| <<  >>  >>> | 移位 |
| < <+ > >= instanceof | 小于，小于等于，大于，大于等于 |
| ==  !=  ===  !==  | 等于、不等于、严格相等。非严格相等 |
| & | 按位与 |
| ^ | 按位异或 | 
| \|  |  按位或 |
| && | 逻辑与 |
| **\\****\\**  | 逻辑或 |
| ?:   | 条件运算 |
| =  op=  |  赋值运算赋值 |
| , | 多重求值 | 


## 8、类型转换

Number():转化成数值。只有纯数字的才能转换得到真实数字。

布尔类型转换为 Number：true 转换为 1、false 转换为 0。

未定义类型转换为 Number：underfind 转换为 NaN。

空类型转换为 Number：null 转换为 0。

```html
<script type="application/javascript">
console.log(Number("123"));  //123
console.log(Number("95a97"));//NaN
console.log(Number(null));   //0
console.log(Number(undefined));//NaN
console.log(Number(true));   //1
console.log(Number(false));  //0
</script>
```

String()：转化成字符串


```html
<script type="application/javascript">
console.log(typeof String(123),String(123));//string 123
console.log(typeof String(true),String(true)); // string true
console.log(String(false));//false
console.log(String(null));//null
console.log(String(undefined));//undefined
</script>
```

Boolean()：转化成布尔


```html
<script type="application/javascript">
console.log("NaN="+Boolean(NaN));//false
console.log("0="+Boolean(0));//false
console.log("null="+Boolean(null));//false
console.log("undefined="+Boolean(undefined));//false
console.log(Boolean(""));//false
console.log(Boolean(''));//false

</script>
```

**小结**
- （1） NaN , 0 ,"" , '' ,undefined, null 转换为：false。
- （2）正数，负数都是:true
- （3）只要不是空字符串都是：true


parseInt()：将字符串转化为整数


```html
<script type="application/javascript">
var input= parseInt(prompt("请输入一个数："));
console.log(typeof input,input);
console.log(typeof parseInt("92"),parseInt("92"));//number 92
</script>
```

parseFloat()：将字符串转化为浮点数


```html
<script type="application/javascript">
var input = parseFloat(prompt("请输入一个数："));
console.log(typeof input,input);
console.log(typeof parseFloat("26.22395"),parseFloat("26.22395"));//number 92
</script>
```

# 五、Js 分支与循环

- 顺序单语句
- 分支语句
    - 单分支语句  `if(表达式1){语句;}`
    - 多分支语句  `if(表达式1){ 语句; }else if(表达式2){ 语句; }else{ 语句; }`
    - Switch 语句 `switch(表达式){case 表达式 1: 语句 1; break;case 表达式 2: 语句 2; break;case 表达式 n: 语句 n; break;default: 语句 n;}`
- 循环语句
    - for 循环
    - while() 循环  `while(条件判断表达式){ 语句； 循环变化值 }`
    - do{ }while() 循环
- 嵌套语句

if 示例

```html
<script type="application/javascript">
    var age  = 18;
    if(age>=18){
        console.log("您是成年人！");
    }else{
        console.log("您还未成年！");
    }
</script>
```

switch 示例

```html
<script type="application/javascript">
var score = prompt("请输入一个学生的成绩：");
switch(true){
    case score>=90:
        console.log("优");break;
    case score>=80:
        console.log("良");break;
    case score>=70 :
        console.log("中");break;
    case score>=60:
        console.log("差");break;
    default:
        console.log("不及格");
}
</script>
```


循环示例

```html
<script type="application/javascript">
// for 循环
for(var i=1;i<=10;i+=2){
    console.log(i);
}

// for in 循环
var arr = ['1', 'x', "red", "green"] ;
for(i in arr){
    console.log(i+","+arr[i]);
}
var person = {
    username:"特朗普"
    age:72,
    from: "加州"
}
for(var i in person){
    console.log(i+","+person[i]);
}


//第一步：循环的初始值
var  x = 1; 
//第二步：循环判断的条件
while(x<=10){
    console.log(x+".您好！");
    //第三步：循环的变化值
    x++; 
}


var y=1; 
do {
    console.log("第 y = " + y +" 次循环");
    i++;
}while(i<=10);|



// 打印乘法口诀表
//外层控制行数
for(var i= 1;i<=9;i++){
    //内循环控制每行有几列
    for(var j = 1;j<=i;j++){
        document.write(j+"&times;"+i+"="+j*i+"&nbsp;&nbsp;") ;
    }
    document.write("<br />");
}
</script>
```

break : 结束循环

continue : 结束循环的某次执行，继续下次执行。