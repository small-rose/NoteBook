---
layout: default
title: SpringBoot session
parent: SpringBoot
nav_order: 300
---


Here are SpringBoot session experience .
{: .fs-6 .fw-300 }


##   Here are session using demo experience .
{: .no_toc .text-delta }

1. TOC
{:toc}



# session



##  session 配置

服务器session超时的配置

springboot 1.X 单位默认是秒

```
server.session.timeout=1800
```

springboot 2.x 单位变成了 Duration
```
server.servlet.session.timeout=
```

编码设置
```
int timeout = 1800 ;
session.setMaxInactiveInterval(timeout);
```

获取session 超时时间

```
int sessionTimeout = session.getServletContext().getSessionTimeout();
```


Duration 类

Duration转换字符串方式: 
   - 默认为正，负以-开头，紧接着P，（字母不区分大小写）
   - D : 天 
   - T : 天和小时之间的分隔符 
   - H : 小时 
   - M : 分钟 
   - S : 秒 

每个单位都必须是数字，且时分秒顺序不能乱。

例如: PT20M，就是设置为20分钟，PT8h，就是设置为8小时，


```properties
# 超时时间24h
server.servlet.session.timeout=PT24h
# 分钟
#server.servlet.session.timeout=1440
``` 

实在看不明白就使用代码转了再输出:

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;
 
public class DurationToString {
    public static void main(String[] args) {
        // 创建一个持续3天6小时的时长
        Duration duration = Duration.ofDays(3).plusHours(6);
 
        // 转换为字符串
        String durationString = duration.toString();
 
        // 输出结果
        System.out.println(durationString); // 输出格式例如: P3DT6H
    }
}
```

## session 共享的方式

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
	</dependency>
	<dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-core</artifactId>
	</dependency>
	<dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-jdbc</artifactId>
	</dependency>
</dependencies>
```



# session 扩展

> 内容来源：https://zhuanlan.zhihu.com/p/139581725

## session 是什么

Servlet 规范中的内置对象, 属于客户端与服务器的会话对象。

> Session经常被翻译为会话，其本来的含义是指有始有终的一系列动作/消息。比如打电话时，从拿起电话拨号到挂断电话这中间的一系列过程可以称之为一个Session。而在网络中，Session是指从一个浏览器窗口打开再到关闭的这个期间。 互联网应用层协议基本都是基于 HTTP 和 HTTPS 协议的，它们的本身都是无状态的， 也就是只负责网络的请求和响应。 我们只需要告诉服务器我们需要什么，服务器就会给我们返回相应的资源。 如果没有额外处理的话，服务器并不知道发起请求的人是谁，也无法根据请求者是谁来给你展现和你相关的内容了。 HTTP 协议之所以一开始被设计成这样，还是有一些历史原因的，当时的互联网多用于学术交流，只用于文章信息的展现，远没有像现在这么丰富多彩。所以在当时的背景下， HTTP 协议被设计成这样，其实也是很符合它的场景的。 但随着互联网应用越来越广泛，应用的形式也变得越来越多，我们的 Web 应用已不只限于提供简单的信息展现了，还需要用户能够与服务器进行交互，比如能够登录，可以在留言回复、可以进行购物、社交等。 这就需要 HTTP 协议能够记录用户的状态，而这个状态就可以由HttpSession来进行保存。


## session 工作原理


Client(浏览器)第一次发送请求的时候，Web Container(Tomcat、Jetty等服务器容器)会生成唯一的Session ID(这个Session ID包括随机数+时间+JVM ID)，并将其返回给Client(在Web Container返回给Client的Response中)，但是Web Container上的这个HttpSession是临时的。

接下来Client在每次发送请求给服务器时，都会将Session ID发送给Web Container，这样Web Container就能很容易区分出是哪个Client。

Web Container会使用这个Session ID，找到对应的HttpSession，并将这个Request与这个HttpSession联系起来。 


如：

当用户第一次访问Servlet时，服务器端会给用户客户端创建一个独立的Session；

该Session会有一个Session ID(JSESSIONID),格式如：JSESSIONID=7F149950097E7B5B41B390436497CD21，其中JSESSIONID是固定的。而这个Session ID在响应浏览器的时候会被存储到Cookie中,从而被保存到浏览器中；

而后面的value值对应的则是给该客户端新创建的session的ID；

当用户再一次访问Servlet时,请求C都会携带着Cookie中的SessionID去访问；

服务器会根据这个Session ID去查看是否有对应的Session对象；

如果有就拿出来使用;如果没有就创建一个Session(相当于用户第一次访问)。

**session 超时**

由于会有越来越多的用户访问服务器，因此Session也会越来越多。

为了防止内存溢出，服务器会把长时间内没有活跃的Session从内存中删除，而这个时间就是Session的超时时间。

如果超过了超时时间没访问过服务器，Session就自动失效了。

这个就是开始设置session超时时间的必要性。


## session特点:

- Session数据保存在服务器端；
- Session中可以保存任意类型的数据；
- Session默认的生命周期是30分钟，可以手动设置更长或更短的时间。


## HttpSession生命周期


### HttpSession 创建

（1）对于JSP而言: 是否浏览器访问服务端的任何一个JSP，服务器都会立即创建一个HttpSession对象呢？ 不一定。 　　①.若当前的JSP或Servlet，是客户端访问当前WEB应用的第一个资源，且JSP的page指令中的session属性为false时，服务器是不会为JSP创建HttpSession对象的； 　　②.若当前JSP不是客户端
访问的WEB应用的第一个资源，且其他页面已经创建了一个HttpSession对象，则服务器也不会为当前JSP创建一个新的HttpSession对象，而是会把和当前会话关联的那个HttpSession对象返回给当前的JSP页面。

（2）对于Servlet而言:若Servlet是客户端访问的第一个WEB应用资源，只有调用了request.getSession()或request.getSession(true) 才会创建HttpSession对象。

### HttpSession 销毁

（1）直接调用HttpSession的invalidate()方法，会使HttpSession失效；

（2）服务器卸载了当前Web应用;

（3）超出了HttpSession的过期时间。 


```
#代码中设置session过期时间的方式 
session.setMaxInactiveInterval(5);
```

```
#web.xml中设置session过期时间的方式 
<session-config> 
     <session-timeout>30</session-timeout>
</session-config>  
```


## Cookie机制

### cookie 是什么

Cookie翻译成中文是甜饼的意思，其实就是一个小型的文本文件，用来保存一些简单的信息(浏览器对Cookie的内存大小是有限制的)。Cookie由服务器端生成，并且会发送给 User-Agent (一般是浏览器)，服务器一般会告诉浏览器设置一下Cookie，然后浏览器会自动将该 Cookie 以 key/value 的格式保存到浏览器的某个目录下；等到下次请求同一网站时，浏览器会自动通过请求头发送该Cookie给服务器，前提是浏览器设置了启用Cookie功能。

### 为什么要有Cookie
   
Web应用程序是使用HTTP协议来传输数据的，而HTTP协议是无状态的协议，也就是说一旦数据交换完毕，客户端与服务器端的连接就会关闭，等再次交换数据就需要建立新的连接，这就意味着服务器无法从连接上跟踪会话。比如我们登陆一个网站的时候，会提醒你要不要记住账户和密码，这样下次来你就不用再次输入账号密码了，这就是Cookie的作用。当我们再次访问的时候，服务器会直接根据我们的Cookie来获取上一次取过的东西。


### Cookie 的特点

- Cookie 的过期时间
- Cookie要满足同源策略
- Cookie内存大小受限制
- Cookie的安全性

（1）Cookie 的过期时间

每次发送请求的时候，都会根据domain来设置相应的Cookie。Cookie有永久的，也有临时的，每个浏览器都有自己的Cookie，我们可以通过设置expires、max-age来设置保存日期，如果不设置的话默认是临时存储，也就是说关闭浏览器后Cookie就会消失。

（2）Cookie 的过期时间

虽然网站http://news.baidu.com与http://www.baidu.com同属于Baidu，但是域名却不一样，也就是说这两者之间是不能互相操作彼此Cookie的。只有域名和path都必须一样，才能相互访问彼此的Cooki。但是需要注意不同浏览器对path访问规定是不一样的，对于chrome，path必须为当前目录，设置为其他目录无效，当前页面只能访问当前目录的Cookie`。

（3）Cookie内存大小受限制

Cookie有个数和大小的限制，大小一般是4k，但是不同的浏览器，具体的Cookie大小也是不同的。

- Firefox和Safari允许Cookie多达4097个字节，包括名(name)、值(value)和等号;
- Opera允许Cookie多达4096个字节，包括名(name)、值(value)和等号;
- Internet Explorer允许Cookie多达4095个字节，包括名(name)、值(value)和等号。

（4）Cookie的安全性

Cookie是保存在浏览器本地的，是可以被修改的，所以敏感的数据不要放在Cookie里。