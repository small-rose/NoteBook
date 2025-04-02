---
layout: default
title: Js Function
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 14
---

 Here are JavaScript function examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、function 简介


## 1、含义

**函数**：是一般是由事件驱动的，为了实现特定功能的，可以重复调用的一段可以执行的代码块。

## 2、分类

内置函数：字符串，数学函数，数组函数等，js 系统给我们提供好的函数。

自定义函数：根据需要定义的函数。

## 3、函数的类型

- 无参无返回值
- 无参有返回值
- 有参无返回值
- 有参有返回值

基本语法

```
function fn(){
    return 1 ;
}
```

示例：

```html
<script>
function noArgsReturn() {
    console.log("我是无参无返回值函数");
}

function noReturn(a) {
    console.log("我是有参无返回值函数，收到参数" +a);
}

function noArgs() {
    console.log("我是无参有返回值函数");
    return result;
}

function add(a, b) {  
    console.log("我是有参有返回值函数");
    return a + b;
}
</script>
```

return 语句将终止当前函数并返回当前函数的值，return 后面的语句不执行。


return 后面可以跟一个 value ，可以是 Js 中的任何数据类型，数字，字符串，函数，对象等，当然也可是再返回一个函数。


```html
<script>
function getObj() {
    console.log("我是无参有返回值函数");
    return {
        "name": "xiaocai"
    };
}

function getFunc(a, b) {  
    console.log("我是有参有返回值函数");
    return function() {
       return a - b ;
    };
}
</script>
```

**boolean类型的返回**

- return true; 返回正确的处理结果。
- return false; 返回错误的处理结果，终止处理。return false 只在当前函数有效，不会影响其他外部函数的执行。
- return ; 把控制权返回给页面。(返回 undefined)


## 4、void 的使用


`javascript:void(0);` 该操作符指定要计算一个表达式但是不返回值。

`void()` 仅仅是代表不返回任何值，但是括号内的表达式还是要运行: 

```javascript
void(alert("running!"));
```

`href="#"`与 `href="javascript:void(0)"`的区别

`#` 包含了一个位置信息，默认的锚是#top 也就是网页的上端。

`javascript:void(0)`, 仅仅表示一个死链接。


##  5、匿名函数


匿名函数：在 JavaScript 中，当把函数当做数据使用时，可以不设置名字。

函数必须要先声明后使用。如果后声明（函数）会自动进行`函数提升`。

```html
<script>
var sum = function(a, b) {
  return a + b;
}

console.log(sum(1,2));

console.log(sub(10, 8));
var sub = function(a, b) {
  return a - b;
}

</script>
```

##  6、回调函数

什么是回调函数？

当一个函数作为另一个函数的参数时，作为参数的函数被称之为回调函数。


```html
<script>
var obt =document.querySelector("button");
//回调函数
obt.onmouseover = function(){ 
    alert("回调函数被触发！");
}

//延时器：按照指定的时间之后，执行函数表达式
setTimeout(function(){
    console.log("2秒之和你会见到我！");
},2000)


var arr =["red","blue","green"];

arr.forEach(function(v, i, arr){
    console.log(v, i, arr);
});
</script>
```

## 7、函数的自调用(沙箱)

function 前面加上一些操作符，这样 js 引擎在解析的时候就不会把它当成是函数声明了。


```html
<script>
//匿名函数
var f1 = function(){
    console.log("立即执行1" +111);
}();


(function(){
    console.log("立即执行2" + 222);
}());

(function(){
    console.log("立即执行3" + 333);
})();

!function(){
     console.log("立即执行4" + 444);
 }();

//函数的声明
+function f1(){
    console.log("立即执行5" + 555);
}();


</script>
```

沙箱：就是与外界隔绝的一个环境，外界无法修改该环境内的任何信息（沙箱内的东西单独属于一个世界）。

JS 中的沙箱模式：还是通过函数来实现的

（1）沙箱模式的基本模型：自调用函数

```
(function(){
var a = 123; //外面并不能访问到这个 a
})();
```
（2）沙箱里面的变量对外面没有影响

（3）为什么要使用立即执行函数表达式 IIFE（其实就是匿名函数）来写沙箱：因为立即 IIFE 不会在外界暴露任何的全局变量，但是又可以形成一个封闭的空间，刚好可以实现沙箱模式。

（4）在沙箱中将所有变量的定义放在最上面，然后中间就放一些逻辑代码。

（5）js 中沙箱模式的实现原理：函数可以构建作用域，上级作用域不能直接访问下级作用域中的数据。


```html
<script>
// r 是形式参数
(function(r){
    var area = Math.PI * r * r;
    console.log(area);
})(5);
// 5 是实际参数
</script>
```

## 8、Function 构造函数（函数声明）


Function 是一个构造器， 能创建 Function 对象 ，即 JavaScript 中每个函数实际上都是 Function 对象。

函数构造器的语法：
```
new Function ([arg1[, arg2[, ...argN]],] functionBody)
```
说明：
- arg1、arg2 等为构造器的参数，
- functionBody 为方法体。注意：参数必须用引号包围！

实例：

```javascript
var plus = new Function("a","b","return a+b");
var result = plus(1,2);//调用，返回 3

```
上述方法创建 function 等同于普通函数声明方法：

```javascript
function plus(a,b){return a+b;};
```

```javascript
var add = new Function(
'x',
'y',
'return x + y');
```

不同方式创建函数

```javascript

//字面量方式创建函数
var fun =function(){
    console.log(100)
}

//函数声明方式创建函数
function fn(){
    console.log(200);
}
//创建Funtion类型的对象var函数名=newFunction（‘参数'，"`函数体）
var f = new Function('a','console.log(a)');
f(2);//以函数方式调用

var f = new Function('a','b','var c = a+b ; console.log(c); return c;');
f(2);//以函数方式调用
```

## 8、Function 与 function 的区别

**Function 与 function 的区别**

- Function 是一个功能完整的对象，作为 JS 的内置对象之一 。 

- function 只是一个关键字，用来创建一个普通函数或对象的构造函数。

- JS 的普通函数都是 Function 对象的实例，所以函数本身也是一个对象，就像 var 一样，只不过这个对象具有可调用特征而已。



## 9、递归函数

递归函数：函数自己调用自己

```javascript
var n = 1;

function f1(){
    console.log(n);
    n++;
    if(n>80){
        return 0;//调用终止
    }
    f1();
}

f1();
```

## 10、默认值和函数参数

Es6 的参数默认值

```javascript
function log(x, y="lisi"){
    console.log(x,y);
}
log("hello");//hello,Lisi
log("hello","zhangsan");//heLLo,zhansan
```

Es5 的参数默认值

```javascript
function log(x, y){
    y = y || "lisi";
    console.log(x,y);
}
log("hello");//hello,Lisi
log("hello","zhangsan");//heLLo,zhansan
```

参数是一个函数

```javascript
//参数是函数
function f1(fn){
    console.log("我是函数f1");
    fn()
}

f1(function(){
    console.log("我是实际参数函数");
});
```

## 11、高阶函数

高阶函数英文叫 Higher-order function。

那么什么是高阶函数？

JavaScript 的函数其实都指向某个变量。

既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。




# 二、函数的作用域和作用域链

## 1、作用域


作用域（scope）就是变量和函数的可访问范围，或者说变量或函数起作用的区域。

```javascript
//f1函数是全局作用域
function f1(){
    console.log("我是f1");
    //f2是局部作用域：只能在函数内部调用
    function f2(){
        console.log("我是f2");
}
f1();
f2();//报错
```


## 2、作用域分类

Javascript 只有两种作用域

（1）全局作用域

全局作用域：变量和函数在整个程序中一直存在，所有地方都可以读取。

全局变量(global variable)：在函数外部声明的变量，它可以在函数内部读取。

那么什么情况下声明为全局变量呢，多个函数共同使用。

（2）局部作用域

局部作用域：变量和函数只在函数内部存在。在函数外部变量失效。

和作用域与之对应的，javascript 中有两种变量：

局部变量(local variable)：在函数内部定义的变量，外部无法读取。

在函数中，参数也是局部变量。

函数也有作用域，声明在外部的函数，可以任意位置调用，声明在函数内部的函数，一般只能在内部调用。

局部变量会在函数运行以后被删除。

全局变量会在页面关闭后被删除。


**变量加 var 和不加 var 的区别**


1.在函数作用域内加 var 定义的变量是局部变量，不加 var 定义的就成了"全局变量"。

2.在全局作用域下，使用 var 定义的变量不可以 delete,没有 var 定义的变量可以 delete。也就说明隐式全局变量严格来说不是真正的变量，而是全局对象(window)的属性，因为属性可以通过 delete 删除，而变量不可以。

3.使用 var 定义的变量不赋值时会有一个默认初始值：undefined，而不使用 var 定义的变量在 alert()时浏览器会给出错误信息：a is not defined  。

4.在 ECMASceipt5 的'use strict'模式下，如果变量没有使用 var 定义，会报错，


# 三、函数总结


（1）不能在非函数的代码块中声明函数。


（2）name 属性和 length 属性。

```javascript
//name：函数的名称，Length：函数参数的个数
function func(a,b){
    console.log(func.name,func.length);
}
func(2,3);
```

（3）变量和函数的提升。

（4）在 JavaScript 世界中，函数拥有一切传统函数的使用方式（声明和调用），而且可以做到像简单值一样赋值、传参、返回，这样的函数也称之为第一级函数（First-class Function）。JavaScript 中的函数还充当了类的构造函数的作用，同时又是一个 Function 类的实例(instance)。这样的多重身份让 JavaScript 的函数变得非常重要。

（5）arguments 对象

```javascript
//arguments[0]:形式参数1，arguments[1]：形式参数2，arguments[2]：形式参数3
function sum(){
    return arguments[0] + arguments[1] + arguments[2];
}
console.log(sum(2,5,7));
```

（6）值传递和地址传递

**值传递**

```html
<script>
//函数的值传递，形式参数的改变不会影响到实际参数
var num = 100 ;
console.log("调用之前："+num);//100
function change(num){
    num = 888;
    console.log("函数中输出："+num);//888
}
change(num); // num实际参数
console.log("调用之后："+num);//100
</script>
```

**地址传递**

```html
<script>
//地址传递：实际参数和形式参数公用一个地址，形式参数的改变会影响到实际参数
var arr = ["red","green","blue"];
console.log("调用之前：arr'red'green","blue");

function change(val){
    val[0] = "yellow";
    console.log("函数中输出：" + val);//"yelLow","green","blue"
}
change(arr);
console.log("调用之后："+arr);//"yellow","green","blue"
</script>
```

（7）函数的同名参数

```javascript
//同名参数：使用最后一个值
function f1(a,a){
    console.log(a);//undefined
}
f1(2);
```

（8）eval 函数

`eval("字符串")`:可以计算字符串的值

```javascript
console.log(eval("3*6+7"));
```

（9）instanceof 类型检测

```javascript
function person(){
    this.name = "halon";
    this.age = 23;
}

var p1 = new person();
console.log(p1 instanceof person);//true
```
（10）javascript 垃圾回收机制。

对于其他语言来说，如 C,C++,需要开发者手动的来跟踪并管理内存。

Javascript 具有自动垃圾回收机制，会定期进行垃圾回收。

其原理就是找出那些不在被使用的变量，然后释放其所占有的内存。


# 四、闭包

## 1、什么是闭包

闭包是解决什么问题的？

```html
<script>
//在函数外部访问函数内部的局部变量
function f1(){
    var a = 100 ; 
}
f1();
alert(a);//报错
</script>
```

> 在函数外部无法访问函数内部的局部变量

闭包：能够读取其他函数（外部函数）变量的函数（内部函数）就是闭包。


## 2、闭包的分类



（1）函数闭包

```html
<script>
//闭包：能够访问其他函数（f2)变量的函数（f1）就是闭包
function f1(){
    var n = 99; //对于f2这个函数来说，n就是全局变量
    function f2(){
        n++; 
        return n;
    }
    return f2
};
var result = f1();

console.log(result());//100
console.log(result());//101
console.log(result());//102
</script>
```

没有返回值时:

```html
<script>
function f1(){
    var n = 99;
    function f2(){
        n++;
        console.log(n);
    }
    f2();
}

f1();//100
f1();//100
f1();//100
</script>
```

（2）对象闭包

```html
<script>

//对象闭包
var name = "the window" ;
//定对象
var obj = { 
    name:"myobject",
    getName: function(){
        return function(){
            return this.name;//this指向window
        }
    }
}
// 我们调用obj.getName这个方法，我们将它赋值一个变量re，这个变量re在全局作用域，
//那么re这个变量就属性window对象，所以这个this指向window对象
var result = obj.getName();
console.log(result());//the window

</script>
```



## 3、闭包的特点和作用

闭包的特点：

- 特点 1：函数内部包含一个函数
- 特点 2：内部函数可以使用外部函数的变量


闭包的作用：

- 一个就是可以读取函数内部的变量。
- 另一个就是让这些变量的值始终保持在内存中。


闭包的问题

**内存泄漏**

内存泄漏（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。


## 4、闭包的实际应用

没有使用闭包

```html
<ul>
<li>第一行</li>
<li>第二行</li>
<li>第三行</li>
</ul>
<script>
var oLi = document.querySelectorAll("li");//获取Li元素-是一个数组
for( let i = 0; i<oLi.length; i++){
    oLi[i].index = i;
    oLi[i].onmouseover = function(){
        oLi[i].style.background = "yellow";
    }
    oLi[i].onmouseout = function(){
        oLi[i].style.background = "";
    }
}
</script>
```

使用闭包

```html
<ul>
<li>第一行</li>
<li>第二行</li>
<li>第三行</li>
</ul>
<script>
var oLi = document.querySelectorAll("li");//获取Li元素-是一个数组
for( let i = 0; i<oLi.length; i++){
    oLi[i].onmouseover = showBgColor(i,"blue");
    oLi[i].onmouseout = showBgColor(i,"");
    
    function showBgColor(i,color){
        return function(){
            oLi[i].style.background = color;
        }
    }
}
</script>
```