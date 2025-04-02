---
layout: default
title: Js Dom
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 16
---

 Here are Dom examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


Dom
{: .label .label-blue }

# 一、Dom

## 1、什么是Dom

DOM （Document Object Model），文档对象模型，DOM 可以以一种独立于平台和语言的方式访问和修改 html 文档的内容和结构。

在 HTML DOM (Document Object Model) 中, 每一个元素都是 节点: 

- 文档是一个文档节点。
- 所有的 HTML 元素都是元素节点。
- 所有 HTML 属性都是属性节点。
- 文本插入到 HTML 元素是文本节点。
- 注释是注释节点。


## 2、Document 对象

Document 对象是 HTML 文档的根节点。

Document 对象使我们可以从脚本中对 HTML 页面中的所有元素进行访问。

Document 对象是 Window 对象的一部分，可通过 `window.document` 属性对其进行访问。




# 二、Document 节点和属性



## 1、获取文档内部的某个节点

- doctype   文档类型
- documentElement  文档结构
- head  文档头
- body  文档体

```html
<html>
<head>
    <title>文档头节点</title>
</head>
<body>
    <h2>文档body节点</h2>
</body>
</html>
<script>
let doctype = window.document.doctype;
let doc = document.documentElement;
let head = document.head;
let body = document.body;

console.log(doctype);
console.log(doc);
console.log(head);
console.log(body);
</script>
```

## 2、获取文档指定信息

文档属性
- documentURI 返回当前的网址
- URL 返回当前的url
- domain 返回当前的域名
- lastModified 返回当前的最新修改时间
- location 网址跳转
- title 返回当前文档的标题
- readyState 返回当前文档的状态。
    - loading : 加载 HTML 代码阶段（尚未完成解析），
    - interactive : 加载外部资源阶段。
    - complete : 全部加载完成是 complete。

```html
<html>
<head>
    <title>文档头节点</title>
</head>
<body>
    <h2>文档body节点</h2>
</body>
</html>
<script>
console.log(document.readyState);//Loading

let documentURI = document.documentURI;
let url = document.URL;
let domain = document.domain;
let lastModified = document.lastModified;
let title = document.title;

console.log(documentURI);
console.log(url);
console.log(domain);
console.log(lastModified);
console.log(title);
console.log(lastModified);
console.log(lastModified);

//location: 跳转网址
location.assign('https://docs.zhangxiaocai.cn');
window.location = 'https://docs.zhangxiaocai.cn';
location.href = 'https://docs.zhangxiaocai.cn';

//异步加载完成
setInterval(function(){
    if(document.readyState == "complete"){
         console.log("加载完成！");
    }
}, 1000);
</script>
```


## 3、获取文档内部特定节点的集合

- anchors 返回所有锚点名称集合
- forms  返回所有`<form />`表单节点集合
- images  返回所有 `<img />`节点集合
- links  返回所有 `<link />`节点集合
- scripts 返回所有 `<script />`节点集合


```html
<html>
<head>
    <title>文档头节点</title>
</head>
<body>
    <h2>文档body节点</h2>
</body>
</html>
<script>
let anchors = document.anchors;
let forms = document.forms;
let images = document.images;
let links = document.links;
let scripts = document.scripts;

console.log(anchors);
console.log(document.forms);
console.log(images);
console.log(links);
console.log(scripts);
</script>
```

 
# 三、获取元素节点

重要
{: .label .label-blue }
 
 
常用方法
 
- getElementById():通过标签的 id 属性获取元素。

- getElementsByTagName():通过标签名来获取元素。（返回数组）

- getElementsByName():通过标签的 name 属性获取元素。（返回数组）

- getElementsByClassName():通过标签的 class 属性来获取元素。（数组），有浏览器兼容性，主要是 ie8 以下。

- querySelector():通过 css 选择器来获取元素。

- querySelectorAll():通过 css 选择器来获取元素(数组)。


**getElement 和 querySelector 的区别**

query 选择符选出来的元素及元素数组是静态的。

getElement 这种方法选出的元素是动态的。

静态的就是说选出的所有元素的数组，不会随着文档操作而改变。

在使用的时候 getElement 这种方法性能比较好，query 选择符则比较方便。


得到的元素不是需要很麻烦的多次 getElementByXxx ,尽量使用 getElementByXxx 因为他快些。

得到的元素需要很麻烦的多次 getElementByXxx 组合才能得到的话使用 querySelector 方便。



```html

<ul>
    <li>菜单1</li>
    <li>菜单2</li>
    <li>菜单3</li>
</ul>

<script>

//var oul = document.getELementsByTagName("uL")[0];
//var oLi = document.getELementsByTagName("Li");//数组

var oul = document.querySelectorAll("ul")[0];
var oLi = document.querySelectorAll("li");//数组

for(var i = 0 ; i < oLi.length; i++){
    var oliNew = document.createElement("li");
    oliNew.innerText = "新菜单"+i;
    oul.appendChild(oliNew);//给uL添加Li元素
}
</script>
```

创建 li 元素， getElement 创建元素 ， 会造成死循环，而使用 querySelecotor 创建的 li 元素是可以的。


# 四、创建页面元素（重点）

重要
{: .label .label-blue }

## 1、常用方法

- createElement()：创建元素节点
- createTextNode()：创建文本节点
- createAttribute()：创建属性

```html
<body>
</body>
<script>
//创建p元素
var op = document.createElement("p");

//创建文本节点
var otxt = document.createTextNode("黄河远上白云间，一片孤城万仞山。");

//给p元素添加文本内容
op.appendChild(otxt);

//创建一个styLe属性
var ostyle = document.createAttribute("style");

ostyle.value = "font-size:60px;color:red;font-weight:bold;font-style:italic;";
//给p元素添加styLe样式属性
op.setAttributeNode(ostyle);

//将p元素添加body元素中
document.body.appendChild(op);
</script>
```

案例

```html
<html>
<head lang="en">
<meta charset="UTF-8">
<title>dom的操作</title>
<style>
li:hover{ background:tomato!important;}
</style>
</head>
<script>

var arr = ["首页","百度","华为","京东","淘宝"];
var oul = document.createElement("ul");//创建uL元素
var ostyle = document.createAttribute("style");//创建属性
ostyle.value = "list-style:none";
oul.setAttributeNode(ostyle);//给uL添加styLe样式属性

for(var i = 0; i < arr.length; i++){
    var oLi = document.createElement("li");//创建Li元素
    oLi.innerHTML = arr[i];//给元素添加内容
    oLi.style.cssText = "float:left;color:snow;width:100px;height:39px;line-height: 40px";
    oul.appendChild(oLi);//将Li添加到uL中
}
document.body.appendChild(oul);//将uL添加到body中

</script>
```

# 五、操作页面元素属性

操作标签节点的属性

语法：
```
setAttribute('属性名','属性值')：给节点元素设置属性
getAttribute('属性名')：获取属性的值
removeAttribute('属性名')：删除属性
```

操作 style 属性

- style 对象的 cssText 属性

style 属性其他方法，style 对象提供了三个方法来读写行内 css 规则：

- setProperty(propertyName,value)：设置某个 CSS 属性。
- getPropertyValue(propertyName)：读取某个 CSS 属性的值。
- removeProperty(propertyName)：删除某个 CSS 属性。


```html
<div>这是一个快元素</div>
<button>设置属性</button>
<button>获取属性</button>
<button>删除属性</button>
<script>

var odiv = document.querySelector("div");
var obt = document.querySelectorAll("button");
odiv.setAttribute("align", "center");//设置属性并赋值

obt[0].onclick = function(){
    odiv.style.setProperty("width", "300px");
    odiv.style.setProperty("height", "300px");
    odiv.style.setProperty("background","green");
}

// 获取属性
obt[1].onclick = function(){
    console.log(odiv.style.getPropertyValue("width")) ;
    console.log(odiv.style.getPropertyValue("background")) ;
}

//删除属性
obt[2].onclick = function(){
    odiv.style.removeProperty("background");
}
</script>
```

setAttribute 和 createAttribute 的区别

- setAttribute 设置属性并赋值。
- createAttribute 创建一个指定名字的属性，可以自定义，是节点没有的。
- setAttributeNode 给元素添加一个属性节点。

