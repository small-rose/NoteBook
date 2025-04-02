---
layout: default
title: Js String
parent: JavaScript
grand_parent: Front-end Dev
nav_order: 13
---

 Here are JavaScript String examples .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


# 一、string 简介


字符串就是用单引号或者双引号包裹起来的，零个或多个排列在一起的字符。

例如：'javascript', "", "311" , '9-11a$', "xiao_cai"

嵌套：字符串可以嵌套。

在单引号包裹的字符串内部，应该使用双引号进行嵌套。

在双引号包裹的字符串内部，应该使用单引号进行嵌套。

例如："I am 'xiaocai'", 'are u "kidding" me'


# 二、字符串的使用

## 1、字符串换行

字符串换行需要使用反斜杠(\)

```javascript
var x = "Hello \
World!";

```


## 2、length 属性


````html
<script type="text/javascript">
var str = "I am 'xiaocai'";
 
console.log(str.length);
</script>
````


## 3、字符索引


**[ ]**方法：在字符串后面接中括号，中括号中写数字。能够访问到字符串中的每个字符。

索引一次只能索引一个字符，如果需要多个则需要用+连接符。

索引从 0 开始，0 表示第一个字符。
 
```javascript
var str = "I am 'xiaocai'";
for(var i = 0; i<str.length;i++){
    console.log(str[i]);
}
```


## 4、模板字符串

模板字符串（template string）是增强版的字符串，用反引号（`）标识。

模板字符串中嵌入变量，需要将变量名写在`${}`之中。

```javascript
var str = `i like ${name} `;
var name = "China!";
console.log(str);
```



## 5、转义字符


常见转义字符:
- `\'` 单引号
- `\"` 双引号
- `\\` 反斜杠
- `\n` 换行
- `\r` 光标到首行
- `\t` tab(制表符)



# 三、字符串对象


javascript 中有字符串类型 string 类型，我们也知道这种基本类型的变量的创建方式。

但 javascript 中还提供了另外一种字符串的声明方式，这种方式叫字符串对象。

使用 new 关键字将字符串定义为一个对象 `new String()`;

```html
<script>
var str = "abc";
var txt = new String("abc");
console.log(typeof str);//string
console.log(typeof txt);//object
console.log(str == txt);
console.log(str === txt);
</script>
```


# 四、字符串常用方法

## 1、常用方法：

- （1）charAt（number）：返回当前指定位置的字符
- （2）charCodeAt(number)：返回当前指定位置的字符 ascii 码值
- （3）concat:连接字符串
- （4）substring(start,end):截取字符串(从哪里开始到哪里结束,end：不包含 end))
- （5）substr(start,length):截取字符串（从哪里开始取多长的字符）
- （6）slice(start,end):截取字符串(end：不包含 end)

- （7）indexOf(str,offset)：返回当前查找字符串在整个字符串中的首次位置，如果没有返回-1。Offset:从哪里开始查找
- （8）lastIndexOf:倒过来查找

- （9）trim():去掉字符串两端的空格
- （10）toUpperCase 和 toLowerCase:大小写转换
- （11）match:返回一个指定字符串的数组
- （12）search:返回位置
- （13）replace:替换字符串
- （14）split:字符串切割，返回数组



Es6 新增的方法
- includes()：返回布尔值，表示是否找到了参数字符串。
- startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
- endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。
- repeat 方法返回一个新字符串，表示将原字符串重复 n 次。

前三个方法都支持第二个参数，表示开始搜索的位置。

S2017 引入了字符串补全长度的功能。

如果某个字符串不够指定长度，会在头部或尾部补全。

padStart()用于头部补全，padEnd()用于尾部补全。

padStart()和 padEnd()一共接受两个参数，第一个参数是字符串补全生效的最大长度，第二个参数是用来补全的字符串。


## 1、chartAt()

charAt(index):返回的是具体的字符。

index : 就是字符串的位置（它是一个数字）, 位置也称索引，索引从 0 开始。

charCodeAt（index）返回的是字符对应的 Unicode 编码（ascii 编码值）

如：A:65 a:97 0:48

```javascript
var str = "Welcome to zxc";
console.log(str.charAt(2));//返回指定位置的字符
console.log(str.charCodeAt(1));//返回指定位置的字符的ascii码值
```



## 2、concat()

字符串连接。

```javascript
var str = "Welcome to ";
var txt = "China!";
var num = 1860;
console.log(str + txt + num);
console.log(str.concat(txt,"-Beijing","-shanghai","-shenzhen"));
``` 

## 3、字符串截取


```html
<script>
let str = "zhang xiao cai, ni hao ya !";

console.log(str.substr(0,5));
console.log(str.substring(5, 5));
console.log(str.search("n"));

</script>
```

## 4、字符串搜索


```html
<script>
let str = " zhang xiao cai , ni hao ya ! ";

console.log(str.indexOf("cai"));
console.log(str.lastIndexOf("h"));
</script>
```


## 5、其他


```html
<script>
let str = " Zhang Xiao Cai, 你 好 呀! ";

console.log(str.trim());

console.log(str.toUpperCase());
console.log(str.toLowerCase());

console.log(str.split(" "));
console.log(str.replace("呀!","吗?"));
</script>
```
 

## 2、字符串 Base64 编码


Base64 本身是一种加密方式，可以将任意字符转成可打印字符。

有时需要以文本格式传递二进制数据，那么也可以使用 Base64 编码。

而我们使用这种编码方法，主要不是为了加密，而是为了不出现特殊字符，简化程序的处理。

javascript 中字符串提供了两个有关 Base64 编码的方法：

- btoa()：字符串或二进制值转为 Base64 编码。
- atob()：Base64 编码转为原来的编码。
- encodeURIComponent()：要将非 ASCII 码字符转为 Base64 编码
- decodeURIComponent()：将转码后的内容转为非 ASCII 内容

```html
<script>
var str = "He11o,xiaocai !";
var txt = "#$%&*)><zxc@pc12138？";
var result = btoa(str);
console.log(result); //给字符串加密
console.log(atob(result));//解密

var enTxt = encodeURIComponent(txt);
console.log(enTxt);
console.log(decodeURIComponent(enTxt));
</script>
```

# 实例

## 打字机效果

```html
<div id="div1"></div>
<div id="div2" ></div>
<script>
var str1 = "十里平湖霜满天,寸寸青丝愁华年。";
var div1 = document.getElementById("div1");
var n = setInterval(function(){
    //截取字符串
    div1.innerHTML = str1.substring(0,n)
    n++;
    if(n>str1.length){
        n=0;
    }
},1000);


var str2="对月形单望相护，只羡鸳鸯不羡仙。";
var div2 = document.getElementById("div2");
var i = 0 ;
setInterval(function(){
	div2.innerHTML += str2[i]
	i++;
	if(i>str2.length){
		i=0;
		div2.innerHTML = "";
	}
},1000);
</script>
```

## 抽奖效果(动画)


```html
<!DOCTYPE html>
抽奖开始显示：<div></div>
<button>开始抽奖</button><button>停止抽奖</button><input type="text"/>

<script>
//获取div元素
var odiv = document.querySelector("div");
//获取按钮元素
var obt = document.querySelectorAll("button");
//获取文本框元素
var oinput = document.querySelector("input");

//定义一个数组
var prize =["一等奖-ZT非凡大师","二等奖-P70","三等奖-oppo-r17","四等奖-小米14","五等奖-瓜子一包"];
//定时器对象
var timer = null;


//开始抽奖按钮单击事件
obt[0].onclick = function(){
	clearInterval(timer);//先停止动画
	timer = setInterval(function(){
		//随机显示抽奖结果在div元素上
		odiv.innerHTML = prize[rnd()];
	},90);
	console.log("开始抽奖");
};

//随机显示数组的下标
function rnd(){
    return Math.floor(Math.random()*prize.length);
};
//停止抽奖按钮单击事件
obt[1].onclick = function(){
	clearInterval(timer);
	//将抽奖结果（odiv.innerHTML）存储到文本框中
	oinput.value = odiv.innerHTML;
	console.log("停止抽奖");
};

</script>
</html>
```



