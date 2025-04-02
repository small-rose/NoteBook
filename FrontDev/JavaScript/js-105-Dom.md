---
layout: default
title: Js onEnvent
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 15
---

 Here are JavaScript examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、事件概述

## 1、什么是事件

事件就是在文档中或者在浏览器窗口中通过某些动作触发。

比如，单击，鼠标经过，键盘按下等。事件通常和函数结合使用。

事件的作用：
- (1)各个元素之间可以借助事件来进行交互
- (2)用户和页面之间也可以通过事件来交互
- (3)后端和页面之间也可以通过事件来交互（减缓服务器的压力）Onsubmit(提交事件和表单一起使用)

## 2、事件分类

（1）鼠标事件

onclick、ondblclick、onmouseover、onmouseout、onmousedown、onmou
seup、onmousemove

（2）HTML 事件

onload、onscoll、onsubmit、onchange、onfoucs(获取焦点)，onblur(失去焦点)

（3）键盘事件

- onkeydown: 键盘按下时触发
- onkeypress:键盘按下并松开的瞬间触发
- onkeyup: 键盘抬起时触发

# 二、事件的使用（重点）

## 1、html 事件

绑定操作发生在 HTML 代码中的事件，称为 HTML 事件。

语法：
```
on+事件=`函数();函数();函数();......`
```
HTML 事件的移除方式：
```
元素.setAttribute('on+事件名'，null);
```
HTML 事件缺陷：耦合性太强了，修改一处另一处也要修改。

当函数没有加载成功时，用户去触发事件，则会报错。

```html
<body>
<button onclick="f1();f2();">单击</button>
<script>
var obt=document.querySelector("button");
//事件清除的方法1
//obt.onclick = null;

//事件清除的方法2
//obt.setAttribute("onclick", null);

//事件清除的方法3
obt.removeAttribute("onclick");
function f1(){console.log("f1");}
function f2(){console.log("f2");}
</script>
</body>
```
## 2、DOM0 级事件

在 js 脚本中，直接通过【on+事件名】方式绑定的事件称为是 **DOM0 级事件**。

基本语法：
```
元素.on+事件名 = function(){ };

元素[on+事件名] = function(){ };
```
DOM0 级事件的移除方式：
```
元素. on+事件名=null;
```

**DOM0 事件调用函数的几种方式**：

- （1）`<input type=”button” onclick=”函数名”>`
- （2）节点.事件=function(){}
- （3）节点.事件=函数名

```html
<body>
<!-- 方式1  -->
<button onclick="f1();f2();">单击</button>
<script>
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
```

## 3、DOM2 级事件

在 js 脚本中，通过 addEventListener 函数绑定的事件称为是 DOM2 级事件。

语法：
```
元素.addEventListener(type,listener,useCapture)
```
- type:事件类型。【没有 on！没有 on！没有 on！】
- listener:监听函数，绑定的函数
- useCapture:是否使用捕获机制。如果不写，默认值为 false
    - false:冒泡机制
    - true：捕获机制
注意:DOM2 级事件可以绑定多个函数，执行顺序按照函数书写的顺
序。

{: .tips }
>注意:DOM2 级事件可以绑定多个函数，执行顺序按照函数书写的顺序。

DOM2 级事件的移除方式:

```
node.removeEventListener(type,外部函数名,useCapture)
```

**事件流**

事件流：多个**嵌套的标记**或者包含的标记**拥有相同的事件**，其中一个元素的事件触发，同时影响其他元素同类型的事件的触发，我们称之为“事件流”。

事件流语法
方式 1：
```
对象.addEventListener(“事件类型”,”函数”,“事件流”); 
// 针对非 ie 浏览器的，事件类型：不加 on
```
方式 2：
```
对象.attachEvent(“事件类型”,”函数”,“事件流”); 
//针对 ie 浏览器的,事件类型：加 on
```

事件流有两种：

- 1、冒泡型（事件由内到外），默认值：false
- 2、捕捉型（事件由外到内）值：true


**冒泡**

什么是事件冒泡?

事件由具体某个元素（子节点）逐级向上传播（父节点，比如html）

**捕捉**

什么是事件捕捉？

事件由某个具体的元素（父节点）逐级向下传播（子节点）。


```html
<body>
<!-- 方式1  -->
<button onclick="f1();f2();">单击</button>
<script>
var obt=document.querySelector("button");
var outer = document.querySelector("#outer");
var inner = document.querySelector("#inner");
var core = documcnt.qucrySelector("#core");
// true : 表示事件捕捉由外向内
outer.addEventListener("click", function(){
    alert("outer")
}, true);
inner.addEventListener("click", function(){
    alert("inner")
} ,true);
core.addEventListener("click", function(){
    alert("core")
},true);
</script>
</body>
```

如何阻止冒泡

- stopPropagation(); 针对非 ie 浏览器阻止冒泡,阻止事件的派发
- cancelBubble=true; 针对 ie 浏览器


```html
<body>
<!-- 方式1  -->
<button onclick="f1();f2();">单击</button>
<script>
var outer = document. querySelector("#outer");
var inner = document.querySeleetor("#inner");
var core = document.querySelector("#core");

core.onclick = function(){
    alert("core!")
    event.stopPropagation();//方法1-阻止冒泡event：是js的内置事件对象
    event.cancelBubble=true; //方法2-阻此泡
}
inner.onclick = function(){
    alert("inner!")
    event.stopPropagation();//方法1-阻止冒泡
    event.cancelBubble = true; //方法2-阻止冒泡 
}
outer.onclick = function(){
    alert("outer!");
}

</script>
</body>
```

如何阻止默认行为？

- preventDefault()
- returnValue = false; 

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="uTF-8">
<title></title>
<style></style>
</head>
<body>
<a href="http://www.baidu.com">百度</a>
<button>单击</button>
<form action="reg.php" method="post" name="form1">
用户名：<input type="text" name="username"/><br/>
密码：<input type="password" name="pwd"/><br/>
<input type="submit" value="提交表单"/>
</form>
<script>
var oa = document.querySelector("a");
var oform = document.querySelector("form");
var username = document. getElementsByName(username)[0];
oform.onsubmit = function(){
    if(username.value == ""){
        alert("用户名不能为空！");
        document.forml.uscrname.focus();//获取焦点
        event.preventDefault();//阻止默认提交
    }
    console.log("密码是："+document["form1"]["pwd"]["value"]);
}
oa.addEventListener("click",function(e){
    //event.preventDefault();/阻止默认链接-方法1
    e.preventDefault();
    e.returnValue=false ;//阻比默认链接-方法2
}
</script>
</body>
</html>
``` 


## 4、DOM0 和 DOM2 的区别

DOM0 和 DOM2 的区别

- DOM0 如果你写了多个事件，只应用最后一个！

- Dom2 如果您写了多个事件，它会都应用。

[DOM0, DOM1, DOM2, DOM3事件的区别与详解](https://blog.51cto.com/knifeedge/5010718)


# 三、鼠标事件 

语法：
```
元素.on+鼠标事件名称 = 调用函数
```
例如：
```
d1.ondblclick = function () { console.log('这是 d1');}
```

鼠标事件指与鼠标相关的事件，具体的事件主要有以下10个事件：

- (1)  click：按下鼠标时触发
- (2)  dblclick：在同一个元素上双击鼠标时触发
- (3)  mousedown：按下鼠标键时触发
- (4)  mouseup：释放按下的鼠标键时触发
- (5)  mousemove：当鼠标在节点内部移动时触发。当鼠标持续移动时，该事件会连触发。
- (6)  mouseenter：鼠标进入一个节点时触发，进入子节点不会触发这个事件(不冒泡)
- (7)  mouseleave：鼠标离开一个节点时触发，离开父节点不会触发这个事件(不冒泡)
- (8)  mouseover：鼠标进入一个节点时触发，进入子节点会再一次触发这个事件(冒泡)
- (9)  mouseout：鼠标离开一个节点时触发，离开父节点也会触发这个事件(冒泡)
- (10)  wheel：滚动鼠标的滚轮时触发

```html
<style>
    .box1{ width: 100px; height: 100px; background-color: rebeccapurple;}
    .box2{ width: 100px; height: 100px; background-color: green;}
    .box3{ width: 100px; height: 100px; background-color: blue;}
    .box4{ width: 100px; height: 100px; background-color: blanchedalmond;}
    .box5{ width: 100px; height: 100px; background-color: orange;}
</style>
<button id="btn1">鼠标单击</button>
<button id="btn2">鼠标双击</button>
<button id="btn3">鼠标按下</button>
<button id="btn4">鼠标抬起</button>

<button id="btn5" class="box1">mousemove</button>
<button id="btn6" class="box2">mouseenter</button>
<button id="btn7" class="box3">mouseleave</button>
<button id="btn8" class="box4">mouseover</button>
<button id="btn9" class="box5">mouseout</button>
<script>
var btn1 = document.getElementById("btn1");
var btn2 = document.getElementById("btn2");
var btn3 = document.getElementById("btn3");
var btn4 = document.getElementById("btn4");

btn1.onclick = function(){
    console.log("单击事件");
}
btn2.ondblclick = function(){
    console.log("双击事件");
}
btn3.onmousedown = function(){
    console.log("鼠标按下");
}
btn4.onmouseup = function(){
    console.log("鼠标抬起");
}
btn5.onmousemove = function(){
    console.log("鼠标移动了");
}
btn6.onmouseenter = function(){
    console.log("鼠标enter进入了");
}
btn7.onmouseleave = function(){
    console.log("鼠标leave离开了");
}
btn8.mouseover = function(){
    console.log("鼠标over进入了");
}
btn9.mouseout = function(){
    console.log("鼠标out离开了");
}
</script>                                   
```


# 四、IE 事件

## 1、事件绑定

在 js 脚本中，通过 attachEvent 函数绑定事件
语法：
```
元素.attachEvent(type,listener)
```
- type:事件类型。【有 on！有 on！有 on！】
- listener:监听函数，绑定的函数
> 注意:如果绑定多个函数，按照函数书写的倒叙执行。

## 2、事件解除

IE 下 DOM2 级事件的移除方式：
```
元素.detachEvent(type,listener)
```
ps：匿名函数无法被移除。

## 3、事件兼容

由于【IE 浏览器中的事件绑定】和【非 IE 浏览器中的事件绑定】方式方法都有所不同。所以单一的某种函数都不能完美解决不同浏览器下的方法绑定问题。

```html
<script>
//封装一个函数，让各个湖览器都能给元素添加事件
//element：元素，type：事件类型，method：函数方法
var obt = document.querySelector(button"):
function addEvent(element, tvpe,method){
    if(element. addEventListener) {
        element.addEventListener (type, method)
    } else if(element.attachEvent) {
        element.attachEvent ("on"+type, method) ;
    }else {
        element['on'+type] = method ;//obt[onclick]= function(){}
    }
}

function show(){
    alert("我是show方法!");
}

addEvent(obt,"click", show);
</script>
```


# 五、表单事件

## （1）表单文档事件

焦点：js 当前正在和用户发生交互的节点称为焦点。可以类比为人类目光汇聚的地方。

语法：获得焦点和失去焦点事件既可以使用 DOM1 绑定也可以使用 DOM2 绑定。

>获得焦点和失去焦点事件均不支持事件冒泡。


```html
<body>
<form action="">
用户名：<input type="text" class="username"/><br>
手机：<input type="text"/><br>
<input type="submit" value="提交"/>
</form>
<script>
var username = document.querySelector(".username");
//onfocus：获取焦点事件
username.onfocus = function(){
    this. style.cssText = "border:1px solid yellow;background:grey:"
}
//onblur：失去焦点事件
username.onblur = function(){
    this. style.cssText = "";
}
</script>
</body>
```

## （2）oninput 事件和 onchange 事件

onchange事件: 元素发生变化的时候，就会触发。

oninput事件: 当给元素输入内容的时候，就会触发。


区别：
- onchange:当失去焦点的时候才会触发此事件。
- oninput:当输入内容的时候，会立即触发。



```html
<style>
.userfalse{ color: red; }
.usertrue{ color: green;}
</style>
<body>
<form action="">
用户名：<input type="text” class="username"/><span class="usershow"></span><br>
手机：<input type="text"/><br>
<input type="submit" value="提交"/>
</form>
<script>
var username = document.querySelector(".username");
var usershow = document.querySelector(".usershow");
//oninput：内容变化
username.oninput = function(){
   console.log(" oninput 事件")
    if(username.value.length <5 || username.value.length>20){
        usershow.innerHTML ="错误，请输入5-20个字符！";
        //usershow.className = "userfalse";
        usershow.setAttribute("class","userfalse");
    }
    else {
        usershow.innerHTML ="正确";
        usershow.setAttribute("class","usertrue");
    }
    console.log("触发了oninput事件!");
}

//onchange：元素发生变化触发此事件
username.onchange = function(){
   console.log(" onchange 事件")
}
</script>
</body>
```

# 六、键盘事件

键盘事件是指当用户在操作键盘的时候会自动被触发的事件。

通常有以下三种：

- (1) onkeydown:用户按下任意键都可以触发这个事件。如果按住不放，事件会被连续触发。

- (2) onkeypress:用户按下任意键都可以触发这个事件(功能键除外)。如果按住不放，事件会被连续触发。

- (3) onkeyup: 用户释放按键时触发。

> ： 键盘事件一般绑定在需要用户输入的元素上( 例如 input)，但是由于键盘事件默认采用事件冒泡机制，因此将键盘事件直接绑定在 body 之上也是允许的。

**键盘属性**

key 和 keyCode 属性

- Key:具体是哪一个键
- keyCode:返回 keydown 何 keyup 事件发生的时候按键的代码，以及
- keypress 事件的 Unicode 字符(ascii 码值)。 如: A:65,a:97,0:48,空格键：32.

```html
<style>
.box{ width:100px; height: 100px; background: red; position: relative; left: 50%; bottom:-350px:}
@keyframes fly { 100%{ transform: translateY(200px) ; } }
</style>
<body>
<div class="box"></div>
<script>
var box = document.querySelector(". box") ;
document.onkeydown = function(e){
switch(keyCode)
	case 37:
		box.style.left = box.offsetLeft + 25+"px"; break;
	case 38:
		box.style.top = box.offsetTop +  20 +"px"; break;
	case 39:
        box.style.left = box.offsetLeft + 25+"px"; break;
	case 40:
        box.style.top = box.offsetTop + 20 +"px"; break;
	case 32:
		var bomp = document.createElement("div");
		box.appendChild(bomp);
		bomp.style.width = "10px";
		bomp.style.height = "30px";
		bomp.style.background = "green";
        bomp.style.position = "absolute";
		bomp.style.left = "46px";
		bomp.style.animation ="fly 2s";
		break;
</script>
```

# 七、滚动事件

## 回到页面顶部

(1) 锚点方式

```html
<body style="height:2000px;">
<div id="topAnchor"></div>
<a href="#topAnchor" style="position:fixed;right:0;bottom:0">回到顶部
</a>
</body>
```

示例:

```html
<html lang="en">
<head >
<meta charset="UTF-8">
<title></title>
<style>
html { height:2000px;} 
a { position:fixed; right:0; bottom:0;}
</style>
</head>
<body>
<div id="top">这是最顶端</div>
<a href="#top">回到顶部</a>
</body>
</html>
```

(2) scrollTop 属性


scrollTop 属性表示被隐藏在内容区域上方的像素数（文档距离顶部的距离）。

元素未滚动时，scrollTop 的值为 0，如果元素被垂直滚动了，scrollTop 的值大于 0，且表示元素上方不可见内容的像素宽度。

由于 scrollTop 是可写的，可以利用 scrollTop 来实现回到顶部的功能。


```html
<body style="height:2000px;">
<button id="test" style="position:fixed;right:0;bottom:0;"> 回到顶部</button>
<script>
test.onclick = function(){
    document.body.scrollTop = 0;
    document.documentElement.scrollTop = 0;
}
</script>
</body>
```


```html
<html lang="en">
<head >
<meta charset="UTF-8">
<title></title>
<style>
html { height:2000px;} 
button { position:fixed; right:0; bottom:0;}
</style>
</head>
<body>
<div id=top">这是最顶端</div>
<button>回到顶部</button>
<script>
    var obt = document.querySelector("button");
    // obt.addEventListener("click", back);
    
    obt.onclick = function()(
        back();
    }
    function back(){
        document.documentElement.scrollTop = 0;
        //document. body. scrollTop = 0;
    }
</script>
</body>
</html>
```

（3）滚动事件

onscroll：滚动事件，onscroll 事件会在【文档】或【元素】发生滚动操作时触发。

```
window.onscroll = function(){
}
```


```html
<html lang="en">
<head>
<meta charset="UTF-8">
<title></title>
<style>
html{width:2000px; height:2000px;}
</style>
</head>
<body>
<script>
window.onscroll = function(){
	console.log("onscroll被触发了！");
}
</script>
</body>
</html>
```