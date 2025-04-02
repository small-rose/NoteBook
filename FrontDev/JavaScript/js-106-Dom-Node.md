---
layout: default
title: Js Dom Node
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 15
---

 Here are JavaScript examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、Node(节点)概述

DOM 是文档对象模型的简称。它的基本思想是：

把结构化文档解析成一系列的节点，再由这些节点组成一个树状结构（DOM Tree）。

所有的节点和最终的树状结构，都有规范的对外接口，以达到使用编程语言操作文档的目的（比如增删内容）。


## 1、什么是节点 Node

node 是 DOM 的最小组成单位，一个文档的树形结构就是由各种不同类型的节点组成。

对于 HTML 文档，node 主要有以下六种类型：

| 节点 | 名称 | 含义 |
|------|-----|-----|
| Document | 文档node | 整个文档（window.document) |
| DocumentType | 文档类型node | 文档的类型（比如<!DOCTYPE htmI>）| 
| Element  | 元素node  |  HTML元素（比如`<body>`、`<a>`等)| 
| Attribute | 属性node |  HTML元素的属性（比如class="right"）| 
| Text | 文本node | HTML文档中出现的文本 | 
|  DocumentFragment | 文档碎片node | 文档的片段 | 


## 2、Node 节点与节点属性

（1）通用属性

- nodeName：获取节点名称
- nodeType：获取节点类型

常见的名称和类型

| 类型 | nodeName | nodeType |
|------|---------|----------|
| DOCUMENT_NODE | #document | 9 |
| ELEMENT_NODE | 大写的HTML元素名 |1 |
| ATTRIBUTE_NODE | 等同于Attr.name| 2|
| TEXT_NODE | #ext | 3|
|DOCUMENT_FRAGMENT_NODE | #document-fragment| 11|
|DOCUMENT_TYPE_NODE| 等同于DocumentType.name|10|


（2） 获取 node 的相关节点-兄弟节点

- ownerDocument 返回当前节点元素的顶层对象-document
- nextSibling 获取下一个兄弟节点元素，包括文本节点和元素节点
- nextElementSibling 获取下一个兄弟节点元素，会跳过所有的非元素节点（如文本节点和注释节点）
- previousSibling 获取上一个兄弟节点元素，包括文本节点和元素节点
- previousElementSibling：获取下一个兄弟节点元素，会跳过所有的非元素节点（如文本节点和注释节点）


nextSibling 和 nextElementSibling 的区别

- nextSibling 属性返回当前元素在其父元素的子节点列表中的下一个节点。这个“下一个”是基于节点在DOM树中的物理位置来确定的，而不是基于文档流（即，它不考虑节点是否为元素节点）。因此，nextSibling 可能会返回一个文本节点（如果当前元素的后面紧跟着一个文本节点），或者如果当前元素是最后一个子元素，它可能会返回 null。

- nextElementSibling 属性则只返回当前元素在其父元素的子元素列表中的下一个元素节点。与 nextSibling 不同，nextElementSibling 会跳过所有非元素节点（如文本节点和注释节点），只返回下一个元素节点。如果没有下一个元素节点，它将返回 null。

> previousSibling 和 previousElementSibling 同理。

```html
<div id="parent">
	<span id="first">First</span>
	<!-- This is a comment -->
	<span id="second">Second</span>
</div>
<script>
    var firstSpan = document.getElementById('first');
	var nextSibling = firstSpan.nextSibling; // 可能是注释节点，或者在某些浏览器中可能是文本节点（空白字符）
	var nextElementSibling = firstSpan.nextElementSibling; // 是id为'second'的<span>元素
	
	console.log(nextSibling); // 输出可能是注释节点或者null（取决于浏览器如何处理空白字符）
	console.log(nextElementSibling); // 输出是<span id="second">Second</span>
</script>
```

> 由于 nextElementSibling 只会返回元素节点，因此当你知道你想要的是下一个兄弟元素时，使用它会更加方便和直观。然而，如果你需要处理所有类型的节点（包括文本节点和注释节点），那么你应该使用 nextSibling。但请注意，在使用 nextSibling 时，你可能需要添加额外的检查来确保你正在处理的是期望的节点类型。


（3）node 的相关节点-父节点

- parentNode：获取父节点元素
- parentElement：获取父节点元素


```html
<div>块元素</div>
<ul>
    <li>菜单1</li>
    <li>菜单2</li>
    <li class="li3">菜单3</li>
    <li>菜单4</li>
</ul>
<button>获取节点</button>
<script>
var obt = document.querySelectorAll("button");
var div = document.querySelector("div");
var li3 = document.querySelector(".li3");
//设置样式
obt[0].onclick = function(){
    console.log(li3.parentNode);// ul...
    console.log(div.parentElement);// body ...
}
</script>
```

（4）node 内容属性

- innerHTML：设置或者是返回 html 标签直接的内容
- innerText：设置或者返回文本内容。HTML 标签不能解析，会原样输出。
- textContent：设置或者返回文本内容。
- nodeValue：返回节点的值

```html
<div id="div1">这是第1个块元素</div>
<div id="div2">这是第2个块元素</div>
<div id="div3">这是第3个块元素</div>
<div id="div4">这是第4个块元素</div>
<p>这是一段文字</p>
<button>设置值</button>
<button>返回值</button>
<script>
var obt = document.querySelectorAll("button");
var divAll = document.querySelectorAll("div");
var odiv1 = document.getElementById("div1") ;
var odiv2 = document.getElementById("div2") ;
var odiv3 = document.getElementById("div3") ;
var odiv4 = document.getElementById("div4") ;
var op = document.querySelector("p");

//设置样式
obt[0].onclick = function(){
    odiv1.innerHTML = "<a href='http://doc.zhangxiaocai.cn' >NoteBook</a>";
    odiv2.innerText = "<a href='http://doc.zhangxiaocai.cn' >NoteBook InnerText</a>";
    odiv3.textContent = "<p>NoteBook textContent</p>";
    odiv4.nodeValue = "<span> zhangxiaocai nodeValue </span>";
}
obt[1].onclick = function(){
    console.log(odiv1.innerHTML);
    console.log(odiv2.innerText);
    console.log(odiv3.textContent);
    console.log(odiv4.nodeValue);
    console.log(divAll.firstChild.nodeValue);
    console.log(op.childNodes[0]);
    console.log(op.childNodes[0].childNodes[0].nodeValue);
}
</script>
```


（5）获取当前 node 子节点相关属性

- firstChild 获取第一个子节点
- lastChild 获取最后一个子节点
- firstElementChild : 获取第一个子节点, 没有空白和换行
- lastElementChild : 获取最后一个子节点, 没有空白和换行
- childNodes：获取子节点（数组）。childNodes 可以获取元素节点和文本节点。
- Children：获取子节点（数组）。children 只能获取元素节点，不能获取文本节点。


```html

<divclass="box">
    <ul>
        <h2>标题</h2>
        <li>搜狐<a href="http://www.doc.zhangxiaocai.cn">隐藏</a></li>
        <li>雅虎<a href="javascript:void(0)">隐藏</li>
        <li>腾讯<a href="javascript:void(0)">隐藏</li>
        <li>华为<a href="javascript:void(0)">隐藏</li>
        <li>百度<a href="javascript:void(0)">隐藏</li>
        <p>11</p>
        <p>22</p>
    </ul>
</div>
<button> 获取子节点 </button>
<script>
var oh2 = document.querySelector("h2") ;
var oul = document.querySelector("ul") ;
var obtn = document.querySelector("button") ;


obtn.onclick = function() {
    console.log(oul.firstChild);//这个有空白和换行节点
    console.log(oul.firstElementChild) ;//直接获取子节点，没有空白的和换行节点
    console.log(oul.lastChild.previousSibling);//获取的是#text文本节点 （换行)
    console.log(oul.childNodes[0].nextSibling);
    console.log(oul.childNodes[0].nextSibling.childNodes[0].nodeValue);
    console.log(oul.children[1]); //获取子节点，没有空白节点
}

</script>
```

# 二、Node 的方法(重点)


## 1、节点的增删改查


- appendChild()：向后添加（追加）子节点
- cloneNode()：克隆子节点
- insertBefore(newNode, oldNode)：向前添加子节点
- removeChild()：删除子节点
- replaceChild(newNode, oldNode)：修改子节点
- hasChildNodes():判断是否有子节点（true,false）。空白节点也是子节点。
- contains()：判断是否包含指定的子节点（true,false）。
- isEqualNode():判断两个节点是否相等


```html
<body>
<div class="box">
    <p class="p2">这是第2段<span>行内元素</span></p>
    <p>这是第3段</p>
</div>
<div class="box2">

</div>
<button>克隆子节点</button>
<button>添加子节点</button>
<button>移除子节点</button>
<button>修改子节点</button>
<button>判断子节点</button>
</body>
<script>
var odiv = document.querySelector("div");//获取div元素
var op = document.createElement("p");//创建一个p元素
var otxt = document.createTextNode("这是一段大佬的介绍,第1段"); //创建文本
op.appendChild(otxt); //给p元素添加文本内容
odiv.appendChild(op); //将P元素添加到div元素中


var obt =document. querySelector("button");
obt[0].onclick = function() {
    odiv.appendChild(p.cloneNode(true));//将克隆的第二段添加到div元素中
}
console.log(p.cloneNode(true));// true: 表示所有的子节点都会

var p2 = document.querySelector(".p2");//创建一个p元素

obt[1].onclick = function() {
    odiv.insertBefore(op, p2); //在p2前面添加节点
}

obt[2].onclick = function() {
    odiv.removeChild(p2.nextElementSibling);
}

obt[3].onclick = function() {
    var op = document.createElement("p");//创建一个p元素
    var otxt = document.createTextNode("这是张小菜的介绍"); //创建文本
    op.appendChild(otxt);
    odiv.removeChild(op, p2);// 
}
var box = document.querySelector(".box");//获取div元素
var box2 = document.querySelector(".box2");//获取div元素

obt[4].onclick = function() {
    if (box.hasChildNodes()){
        console.log("box 有子节点");
    }else{
        console.log("box 没有子节点");
    }  
    if (box.contains(box2)){    
        console.log("yes");
    }else{    
        console.log("no");
    }
    console.log(box.isEqualNode(box2));
}

</script>
```

