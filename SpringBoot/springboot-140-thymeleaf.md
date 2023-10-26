---
layout: default
title: SpringBoot Thymeleaf
parent: SpringBoot
nav_order: 140
---

## SpringBoot Thymeleaf  语法学习

## 一、基本使用

常见的模板引擎有JSP、Velocity、Freemarker、Thymeleaf等

SpringBoot 官方推荐的是 **Thymeleaf** 

Thtmeleaf 在 SpringBoot 中使用只要引入

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
          	2.1.6
		</dependency>
```

也可以自己指定切换thymeleaf版本：

```xml
<properties>
		<thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
		<!-- 布局功能的支持程序  thymeleaf3主程序  layout2以上版本 -->
		<!-- thymeleaf2   layout1-->
		<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
</properties>
```

Thymeleaf 配置类是

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
  	//
```

可以自定义的配置：

```properties
spring.thymeleaf.prefix=classpath:/views/
spring.thymeleaf.suffix=.html
spring.thymeleaf.mode=HTML5
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.servlet.content-type=text/html  
spring.thymeleaf.cache=false
```

缓存在开发阶段可以设置 `false`。idea 中可以使用 Ctrl + F9 快速编译 thymeleaf 语法页面。

当然也使用使用 Java 方式配置：

```java
public class GTVGApplication {
    ...
    private final TemplateEngine templateEngine;
    ...
    public GTVGApplication(final ServletContext servletContext) {
        super();
        ServletContextTemplateResolver templateResolver =
        new ServletContextTemplateResolver(servletContext);
        // HTML is the default mode, but we set it anyway for better understanding of code
        templateResolver.setTemplateMode(TemplateMode.HTML);
        // This will convert "home" to "/WEB-INF/templates/home.html"
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        // Template cache TTL=1h. If not set, entries would be cached until expelled
        templateResolver.setCacheTTLMs(Long.valueOf(3600000L));
        // Cache is set to true by default. Set to false if you want templates to
        // be automatically updated when modified.
        templateResolver.setCacheable(true);
        this.templateEngine = new TemplateEngine();
        this.templateEngine.setTemplateResolver(templateResolver);
        ...
    }
}
```

默认只要把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染。

## 二、基本语法

Thymeleaf 支持6种常见模板模式：

2种标记模板模式：HTML和XML;

3种文本模板模式：TEXT、JAVASCRIPT和CSS

1种无操作模板模式：RAW

1、命名空间

导入thymeleaf的名称空间，在编写的时候会有提示。

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2、基本使用示例

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>This is my Page</h1>
    <!--th:text 将div里面的文本内容设置为取到的值 -->
    <div th:text="${hello}">Hello World</div>
</body>
</html>
```

3、基本语法规则

`th:*`，可以来替换原生属性的值，其中 `"*"` 表示任意html属性。

如常用的 `th:text` 改变或替换当前元素里面的文本内容。



## 三、语法规则与优先级

语法优先级数字越小，优先级越高。

| 优先级 | 基本特征       | 常见属性                                              | 类似语法 JSP/JSTL/JS等                           |
| :----: | -------------- | ----------------------------------------------------- | ------------------------------------------------ |
|   1    | 片段包含       | `th:insert`<br/>`th:replace`                          | `jsp:include`                                    |
|   2    | 片段遍历       | `th:each`                                             | `c:forEach `                                      |
|   3    | 条件判断       | `th:if`<br/>`th:unless`<br/>`th:switch`<br/>`th:case` | `c:if`                                           |
|   4    | 变量声明       | `th:object`<br/>`th:with`                             | `c:set`                                          |
|   5    | 一般属性修改   | `th:attr`<br/>`th:attrprepend` <br/>`th:attrappend`   | `类似js的中attr()`<br/> `append()`、`appendTo()` |
|   a    | 特定属性修改   | `th:value` <br/>`th:href` <br/>`th:src`               | `类似js的中attr()`<br/>`val()`                   |
|   7    | 修改标签体内容 | `th:text` <br/>`th:utext`                             | c:out                                            |
|   8    | 片段声明       | `th:fragment`                                         | -                                                |
|   9    | 片段移除       | `th:remove`                                           | -                                                |

简单示例：

```java
model.addAttribute("hello","<h2>Hello Thyneleaf ! </h2>")
```

页面使用：

```html
<div th:text="${hello}">Hello World</div>
<div th:utext="${hello}">Hello World</div>
```

最终展示效果区别：

`th:text` ：转义特殊字符，内容是什么原样输出什么，浏览器不进行解析。

`th:utext` ：不转义特殊字符，有标签时，浏览器会识别解析 html 标签。

 

## 四、标准的表达式语法

常见或常用表达式学习。

### 1. 简单表达式

- Variable Expressions： 获取变量值的表达式：`${...} `

- Selection Variable Expressions：选择表达式： `*{...} `

- Message Expressions： 获取国际化的表达式： `#{...} `

- Link URL Expressions: 定义URL的表达式：`@{...} `

- Fragment Expressions: 引用代码片段表达式：`~{...}`

  
#### 1） 获取变量值的表达式 

- 获取对象的属性、调用方法；

- 也可以使用内置的基本对象；

简单示例如：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>This is my Page</h1>
    <div th:text="${hello}">Hello World</div>
    <!-- 实际等价于  ctx.getVariable("today"); -->
    <p th:text="${session.user.userName}"></p>
    <!-- 实际等价于  ctx.getVariable("session").get("user")).getUserName(); -->
</body>
</html>
```

##### (a) 常见取变量的写法

访问属性可以使用 `.` 点号

```
${person.father.name}
${user.department.departmenrName}
```

访问属性可以使用 `[ ]` 号

```
${person['father']['name']}
${user['department']['departmenrName']}
```

对象是 Map 类型，可以使用 `[ ]` 和 `.` 混用

```
${countriesByCode.ES}
${personsByName['Stephen Zucchini'].age}
```

带索引的数组或集合可以使用下标索引访问：

```
${personsArray[0].name}
```

也可以调用方法，允许带参数：

```
${person.createCompleteName()}
${person.createCompleteNameWithSeparator('-')}
```



##### (b) 常见内置基本对象：

| 内置对象        | 说明                                                 |
| --------------- | ---------------------------------------------------- |
| #ctx            | 容器对象/上下文对象，也可 `#root` ,`#vars`,推荐`#ctx` |
| #vars           | 容器里的变量                                         |
| #locale         | 容器理的区域对象                                     |
| #request        | Web容器里的HttpServletRequest对象                    |
| #response       | Web容器里的HttpServletResponse对象                   |
| #session        | Web容器里的HttpSession对象                           |
| #servletContext | Web容器里的ServletContext对象                        |

内置对象的详细内容比较多一次整理不完，会单独整理一篇内置对象。

内置对象参考[《Thymeleaf 内置对象》](44478111.html)

#### （c）内置的工具对象

`#execInfo` : 有关模板运行时的一些信息
`#messages` : 在变量表达式中获取国际化信息的方法，也可以使用`#{…}`语法.
`#uris` : 对URI或URL进行转义相关的方法
`#conversions` : 如果有写相关的转换，用来执行配置的转换服务的方法.
`#dates` : java.util.Date 日期对象格式化，组件提取的方法
`#calendars` : 类似于 #dates , 但是是 java.util.Calendar 对象的.
`#numbers` : 数字格式化相关的方法.
`#strings` : 字符串相关方法: 包含, 以什么开始/结尾, 追加, 去空格等等.
`#objects` : 对象通用方法.
`#bools` : boolean 运算方法
`#arrays` : 数组相关方法.
`#lists` : list集合相关的方法.
`#sets` :  set集合相关的方法.
`#maps` : map集合相关的方法.
`#aggregates` : 在数组或集合上创建聚合的方法.
`#ids` : 处理可能重复的id属性的方法.例如，作为迭代的结果.

内置对象的详细内容比较多一次整理不完，会单独整理一篇内置对象。

内置对象参考[《Thymeleaf 内置对象》](44478111.html)

#### 2）选择变量表达式

选择表达式语法：`*{...} ` 和 `*{...} `

 `*{...} ` 一般配合 `th:object` 使用

```html
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Email: <span th:text="*{email}">Saturn</span>.</p>
</div>
```

等价于：

```html
<div>
    <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
    <p>Email: <span th:text="${session.user.email}">Saturn</span>.</p>
</div>
```

也支持混合使用:

```html
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
    <p>Email: <span th:text="*{email}">Saturn</span>.</p>
</div>
```

也可以使用 `#object` 表达式取值: 

```html
<div th:object="${session.user}"> 
    <p>Name: <span th:text="${#object.firstName}">Sebastian</span>.</p> 
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p> 
    <p>Email: <span th:text="*{email}">Saturn</span>.</p> 
</div    
```

如果没有执行对象选择，`$` 和 `*` 语法是等价的:

```html
<div>
    <p>Name: <span th:text="*{session.user.name}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{session.user.surname}">Pepper</span>.</p>
    <p>Email: <span th:text="*{session.user.email}">Saturn</span>.</p>
</div>
```

#### 3）获取国际化的表达式

获取国际化的表达式语法： `#{...}`

比如在国际化里配置好
配置英文
```
home.welcome=Welcome
```
或者中文
```
home.welcome=欢迎登录
```
使用则使用表达式获取即可
```html
<p th:utext="#{home.welcome}">Welcome to our site!</p>
```

也可以添加占位符参数进行动态支持：

`java.text.MessageFormat` 参数支持。

`java.text.*` 动态消息格式化类可以格式化数字或日期。

```
home.welcome=Welcome, {0}!
```

使用时直接传入参数即可

```html
<p th:utext="#{home.welcome(${session.user.name})}">
	Welcome to our site, guest!
</p>
```

参数也可以从变量获取

```html
<p th:utext="#{${welcomeMsgKey}(${session.user.name})}">
	Welcome to our site, guest!
</p>
```

#### 4）定义URL的表达式：

定义URL的表达式语法：`@{...} `

支持绝对路径或相对路径。
相对路径包括：
- 页面相对路径：user/login.html
- 上下文相对路径：/emps?id=2
- 服务相对路径：~、billing/processInvoice
- 相对URL： //code.jquery.com/jquery-2.0.3.min.js    

>表达式的语法真正处理及其到将要输出的url的转换由实现 `org.thymeleaf.linkbuilder`注册到正在使用 `ITemplateEngine` 对象。默认情况下，该接口的一个实现是类的注册`org.thymeleaf.linkbuilder.StandardLinkBuilder`，对于脱机（非web）和web场景都可以支持基于Servlet API。其他场景（如与非ServletAPI web框架的集成）可能需要特定的链接生成器接口的实现。

使用：`th:href`

```html
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>
<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>
<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

注意事项：

这里需要注意的是：

- `th:href` 是一个修饰符属性：一旦处理，它将计算要使用的链接URL并将该值设置为`<a>` 标记的href属性。

- 我们可以对URL参数使用表达式（如orderId=${o.id}）。所需的URLparameter编码操作也将自动执行。

- 如果需要多个参数，这些参数将用逗号隔开：`@{/order/process(execId=${execId}, execType='FAST')}`

- URL路径中也允许变量模板：`@{/order/{orderId}/details(orderId=${orderId})}`

- 以 `/` 开头的相对URL将自动以应用程序上下文名称作为前缀。例如：`/order/details` 是页面查看源代码可以看到是 `http://localhost:8080/thymeleaf/order/details`

- 如果 cookies 未启用或尚不知道，则可能会在相对URL中添加“；jsessionid=…”后缀，以便会话被保留。这叫做URL重写，Thymeleaf允许使用 `response.encodeURL(...)` 的机制对每个URL的Servlet API 进行自定义重写。

- `th:href` 属性允许我们（可选地）在模板中有一个工作的静态 `href`属性，以便当直接打开模板链接以进行原型设计时，浏览器仍然可以导航模板链接。默认`href` 和 `th:href` 可以同时存在，即可以设置默认值。

允许URL通过运算获得：
```html
<a th:href="@{${url}(orderId=${o.id})}">view</a>
<a th:href="@{'/details/'+${user.login}(orderId=${o.id})}">view</a>
```

简单示例：

```html
<p>Please select an option</p>
<ol>
    <li><a href="product/list.html" th:href="@{/product/list}">Product List</a></li>
    <li><a href="order/list.html" th:href="@{/order/list}">Order List</a></li>
    <li><a href="subscribe.html" th:href="@{/subscribe}">Subscribe to our Newsletter</a></li>
    <li><a href="userprofile.html" th:href="@{/userprofile}">See User Profile</a></li>
</ol>
```

 服务器根相对URL

还可以使用其他语法来创建相对于服务器根目录（而不是相对于上下文根目录）的url，以便进行链接同一服务器中的不同上下文。这些url将被指定为 `@{~/path/to/something}`



#### 5）片段表达式

引用代码片段的片段引用表达式语法：`~{...}`

片段表达式是表示标记片段并在模板中移动它们的一种简单方法。这允许我们重复使用相同的代码片段，将它们作为参数传递给其他模板。

代码片段一般配合布局使用，比如头部，菜单，脚部等可以公共使用的代码片段。

##### (a) 小示例

如：在 `/template/footer.html` 定义一个片段

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <div th:fragment="copy">
    	&copy; 2020  Small-Rose 
    </div>
</body>
</html>
```

可以在  `home.html` 使用 `th:insert` 或 `th:replace` 或 `th:include` 引用片段：

使用  `th:insert` :

```html
<body>
...
	<div th:insert="~{footer :: copy}"></div>
</body>
```

对于非复杂的表达式等价于

```html
<body>
...
	<div th:insert="footer :: copy"></div>
</body>
```

##### (b) 片段规范语法

规范语法有三种：

- `"~{templatename::selector}"` 引入名为 `templatename` ，指定标记选择器的模板片段。注意可以是一个片段名，标记选择器可以指定一些内容。
- `"~{templatename::selector}"` 引入名为 `templatename` 的模板片段。
- `~{::selector}"` 或 `"~{this::selector}" ` 引入匹配选择器的同一模板中的片段，如果找不到则会遍历整个模板目录，直到选择器找到匹配。

引入方式都是全功能的，支持选择表达式。

比如定义多个：

```html
<div th:fragment="adminFooter">
	&copy; 2020 Welcome to site , Admin ~
</div>

<div th:fragment="normaluser">
	&copy; 2020  Welcome to Site 
</div>
```

对应的使用：

```html
<div th:insert="footer :: (${user.isAdmin}? #{footer.admin} : #{footer.normaluser})"></div>
```

判断用户是否是Admin用户，如果是admin，则引入 footer 模板页面中取到的国际化值( `#{footer.admin}`)对应的代码片段。

如不使用`th:fragment` 定义示例：

```html
<div id="copy-section">
	&copy; 2020 Small-Rose
</div>
```

对应的引入为：

```html
<body>
...
<	div th:insert="~{footer :: #copy-section}"></div>
</body>
```

其实就是使用了css 选择器，这里使用的 id 选择器

##### (c) 三种片段规范区别

`th:insert` 和 `th:replace`和 `th:include`(从3.0开始不推荐）之间有什么区别？

- `th:insert` 最简单的使用，直接将引入的代码片段插入引用处的标签。
- `th:replace` 直接将引入的代码片段替换引用处的标签。
- `th:include` 有点类似 `th:insert` ，但只是插入引用代码片段的内容部分。     

来个示例：

```html
<footer th:fragment="copy">
	&copy; 2020 Small-Rose
</footer>
```

不同的引入方式，假如使用 `<div>` 引入：

```html
<body>
...
    <div th:insert="footer :: copy"></div>
    <div th:replace="footer :: copy"></div>
    <div th:include="footer :: copy"></div>
</body>
```

最终效果：

```html
<body>
...
    
    <div>
        <footer>
            &copy; 2020 Small-Rose
        </footer>
    </div>

    <footer>
        &copy; 2020 Small-Rose
    </footer>

    <div>
        &copy; 2020 Small-Rose
    </div>
</body>
```

##### (d）可参数化片段签名 

参数化签名可以使模板片段创建一个类似函数的机制，用`th:fragment`定义的片段可以指定一组参数。

```
<div th:fragment="frag (onevar,twovar)">
	<p th:text="${onevar} + ' - ' + ${twovar}">...</p>
</div
```

在引入代码片段的地方传入参数 `th:insert` 或 `th:replace` 皆可：

```html
<div th:replace="::frag (${value1},${value2})">...</div>
```

该种方式需要注意参数顺序，如果不想为参数顺序烦恼，可以使用第一种方式：

```html
<div th:replace="::frag (onevar=${value1},twovar=${value2})">...</div>
```

**特别注意**

如果在定义的时候不指定模板，则只能使用第二种方式传入参数。

定义片段：

```html
<div th:fragment="frag">
...
</div>
```

引入使用只能第二种方式：

```html
<div th:replace="::frag (onevar=${value1},twovar=${value2})">
```

等价于`th:replace` 和 `th:with` 的组合：

```html
<div th:replace="::frag" th:with="onevar=${value1},twovar=${value2}">
```

##### (e）模板内断言

`th:assert` 对于模板内断言 `th:assert` 属性可以指定一个逗号分隔的表达式列表，只有当每个表达式的值都是true的时候，对应的代码片段将会被激活，否则会引发异常。

片段签名添加参数验证：

```html
<header th:fragment="contentheader(title)" th:assert="${!#strings.isEmpty(title)}">...</header>
```

##### (f）灵活布局

片段中title和links变量的用法，在`/template/base.html` ：

```html
<head th:fragment="common_header(title,links)">
    <title th:replace="${title}">The awesome application</title>
    <!-- Common styles and scripts -->
    <link rel="stylesheet" type="text/css" media="all" th:href="@{/css/awesomeapp.css}">
    <link rel="shortcut icon" th:href="@{/images/favicon.ico}">
    <script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>
    <!--/* Per-page placeholder for additional links */-->
    <th:block th:replace="${links}" />
</head>
```

使用

```html
<head th:replace="base :: common_header(~{::title},~{::link})">
    <title>Awesome - Main</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
    <link rel="stylesheet" th:href="@{/themes/smoothness/jquery-ui.css}">
</head>
```

这里的引入是使用的选择器方式引入，结果将使用调用模板中的实际 `<title>` 和 `<link>` 标记作为`title`和链接变量，使我们的片段在插入过程中被自定义：

```html
<head>
    <title>Awesome - Main</title>
    <!-- Common styles and scripts -->
    <link rel="stylesheet" type="text/css" media="all" href="/awe/css/awesomeapp.css">
    <link rel="shortcut icon" href="/awe/images/favicon.ico">
    <script type="text/javascript" src="/awe/sh/scripts/codebase.js"></script>
    <link rel="stylesheet" href="/awe/css/bootstrap.min.css">
    <link rel="stylesheet" href="/awe/themes/smoothness/jquery-ui.css">
</head>
```

还有一些空片段使用、无操作片段使用参考[Thymeleaf 官方文档](https://www.thymeleaf.org/documentation.html)。

### 2. 字面量表达式

1）文本字面量

```html
<p>
Now you are looking at a <span th:text="'working web application'">template file</span>.
</p
```

2）数字字面量

```html
<p>The year is <span th:text="2013">1492</span>.</p>
<p>In two years, it will be <span th:text="2013 + 2">1494</span>.</p>
```

3）布尔字面量

```html
<div th:if="${user.isAdmin()} == false">
```

4）null字面量

```html
<div th:if="${variable.something} == null">
```

3）文字标记

数字、布尔和空文字实际上是文字标记的一种特殊情况。

文字标记允许在标准表达式中进行一点简化。它们的工作原理与文本字面量完全相同，但它们只允许字母`A-Z` 和 `A-Z`、数字 `0-9`、方括号`[ ]`、点 `.`，连字符`-` 和下划线`_` 。**没有空格，没有逗号等**。

```html
<div th:class="content">...</div>
```

代替

```html
<div th:class="'content'">...</div>
```

### 3. 文本操作

`+` 使用：

```html
<p><span th:text="'The name of the user is ' + ${user.name}"></p
```

`|` 使用：

```html
<span th:text="|Welcome to our application, ${user.name}!|">
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

组合使用

```html
<span th:text="${onevar} + ' ' + |${twovar}, ${threevar}|"></span>
```

**注意：**在 `|  |` 文本替换中只允许变量/消息表达式（`${…}`，`*{…}`，`#{…}`）。不允许其他文字标记（`"…"`）、布尔/数字标记、条件表达式等。

### 4. 数学运算

常见的：**`+` , `-` , `*` , `/` 和 `%`**

文本别名运算符：`div`（`/`），  `mod`（`%`）

### 5.逻辑运算

常见的运算符：`and` ，`or` , `|` ，`not`

### 6. 比较运算

gt ( > ), lt ( < ), ge ( >= ), le ( <= ), not ( ! ). Also eq ( == ), neq / ne ( != ).    

| 比较运算   | 符号         |
| ---------- | ------------ |
| 大于       | `gt` ， `>` |
| 小于       | `lt` ， `<`  |
| 大于或等于 | `ge` ，`>=`  |
| 小于或等于 | `le` ， `<=` |
| 非         | `not` ， `!` |
| 等于       | `eq` ， `==` |
| 不等于     | `neq` ，`!=` |

### 6. 条件表达式

常见的条件表达式基本格式：

- `condition ? then : else`
- `condition ? then`
- `condition ? ( condition2 ? then2 : else2 ): else`

```html
<tr th:class="${row.even}? 'even' : 'odd'">
...
</tr>
```

条件表达式的三个部分都可以使用：

- 变量表达式 `${ }`， `*{ }` ，
- 消息表达式 `#{ }` ，
- 链接表达式 `@{ }` ，
- 字面量表达式 `' '`。

### 7. 默认表达式

默认表达式是条件表达式没有 `then` 部分。

基本语法：`condition ?: else`

嵌套语法：`condition ?: ( condition ? then : else )`

```html
<div th:object="${session.user}">
...
	<p>Age: <span th:text="*{age}?: '(no age specified)'">27</span>.</p>
</div>
```

等价于：

```html
<p>Age: <span th:text="*{age != null}? *{age} : '(no age specified)'">27</span>.</p>
```

嵌套示例：

```html
<p>
Name:
<span th:text="*{firstName}?: (*{admin}? 'Admin' : #{default.username})">Sebastian</span>
</p>
```

### 8. 无操作标记

无操作标记使用下划线符号 `_` 

```html
<span th:text="${user.name} ?: _">no user authenticated</span>
```

## 五、属性设置

### 1、通用属性设置

基本语法：`th:attr`

如，动态设置表单的 `action` 值、根据国际化的值显示按钮文字：

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
<fieldset>
<input type="text" name="email" />
<input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
</fieldset>
</form>
```

也可以设置标签没有的自定义属性:

```html
<input type="input" value="someone" th:attr="love=you"/>
```

还可以设置多个属性：

```html
<img src="../../images/gtvglogo.png"
th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

### 2、特殊属性设置

比如上面使用 `th:attr` 设置 `action` 属性和 `value` 属性：

```html
<input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
```

也可以使用`th:action`  和 `th:value` 设置 ：

```html
<form action="subscribe.html" th:action="@{/subscribe}">
<fieldset>
<input type="text" name="email" />
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
</fieldset>
</form>
```

HTML5 常见特殊属性：

| th:abbr  | th:accesskey | th:alt |
|:-------|:-------|:-------|
|th:autocomplete | th:accept | th:action |
| th:archive | th:axis | th:accept-charset |
| th:align | th:audio | th:background |
| th:bgcolor | th:cellspacing | th:cite |
| th:codebase | th:colspan | th:contenteditable |
| th:border | th:challenge | th:class |
| th:codetype | th:compact | th:contextmenu |
| th:cellpadding | th:charset | th:classid |
| th:cols |  th:content | th:data |
| th:datetime |  th:dropzone |  th:form  |
| th:formmethod |  th:frame | th:dir |
|  th:enctype |  th:formaction |  th:formtarget |
|  th:frameborder | th:draggable | th:for |
| th:formenctype |  th:fragment |  th:headers |
| th:height |  th:hreflang |  th:icon |
|  th:keytype |  th:lang  | th:low |
| th:high |  th:hspace |  th:id |
|  th:kind |  th:list |  th:manifest|
| th:href |  th:http-equiv |  th:inline |
|  th:label  |  th:longdesc |  th:marginheight |
|  th:low |  th:manifest |  th:marginheight|
| th:marginwidth |  th:media |  th:name |
|  th:onbeforeprint |  th:oncanplay | th:max |
|  th:method |  th:onabort |  th:onbeforeunload |
|  th:oncanplaythrough | th:maxlength | th:min|
|   th:onafterprint | th:onblur |  th:onchange |
| th:onclick |  th:ondrag |  th:ondragleave |
| th:ondrop |  th:onended |  th:onformchange |
| th:oncontextmenu |  th:ondragend |  th:ondragover |
| th:ondurationchange |  th:onerror |  th:onforminput |
| th:ondblclick  |  th:ondragenter |  th:ondragstart |
| th:onemptied |  th:onfocus |  th:onhashchange |
| th:oninput |  th:onkeypress |  th:onloadeddata |
| th:onmessage |  th:onmouseout | th:oninvalid |
| th:onkeyup |  th:onloadedmetadata |  th:onmousedown |
| th:onmouseover | th:onkeydown | th:onload |
| th:onloadstart |  th:onmousemove|  th:onmouseup |
| th:onmousewheel |  th:onpause |  th:onpopstate |
| th:onreadystatechange |  th:onresize | th:onoffline |
| th:onplay |  th:onprogress |  th:onredo |
| th:onscroll | th:ononline | th:onplaying |
| th:onratechange | th:onreset | th:onseeked |
| th:onseeking |  th:onstalled |  th:onsuspend |
| th:onunload |  th:optimum |  th:poster |
| th:onselect |  th:onstorage |  th:ontimeupdate |
| th:onvolumechange |  th:pattern |  th:preload |
|  th:onshow | th:onsubmit | th:onundo |
| th:onwaiting | th:placeholder | th:radiogroup |
| th:rel |  th:rowspan |  th:scheme |
| th:size |  th:spellcheck | th:rev |
|  th:rules |  th:scope |  th:sizes |
|   th:src | th:rows|  th:sandbox |
|  th:scrolling | th:span | th:srclang |
| th:standby |  th:style |  th:target |
|  th:usemap |  th:vspace  | th:xmlbase |
| th:start |  th:summary |  th:title |
|  th:value |  th:width |  th:xmllang |
| th:step | th:tabindex | th:type |
|  th:valuetype | th:wrap | th:xmlspace |

 ### 3、一次设置多个属性值

有两个相当特殊的属性称为 `th:alt-title` 和 `th:lang-xmllang` ，可用于设置两个属性：

- `th:alt-title` 可以设置 `alt` 和 `title` 属性 . 
- `th:lang-xmllang` 可以设置  `lang`  和  `xml:lang` 属性 . 

```html
<img src="../../images/gtvglogo.png"
th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

等价于

```html
<img src="../../images/gtvglogo.png"
th:src="@{/images/gtvglogo.png}" th:title="#{logo}" th:alt="#{logo}" />
```

等价于

```html
<img src="../../images/gtvglogo.png"
th:src="@{/images/gtvglogo.png}" th:alt-title="#{logo}" />
```

### 4、追加属性

`th:attrappend` ：表示在后面追加 。

`th:attrprepend` ：表示在前面追加 。  

比较常见的是修改 `CSS` 样式：

```html
<input type="button" value="Do it!" class="btn" th:attrappend="class=${' ' + cssStyle}" />
```

假如 cssStyle 的值是 success ，最终效果如下：

```html
<input type="button" value="Do it!" class="btn success" />
```

标准方言中还有两个特定的附加属性：`th:classappend` 和 `th:styleappend` 属性，用于向元素添加 `CSS` 类或样式片段，而不覆盖现已经有的代码。

```html
<tr th:each="prod : ${prods}" class="row" th:classappend="${prodStat.odd}? 'odd'">
```

满足条件之后就会变成：

```html
<tr th:each="prod : ${prods}" class="row odd" >
```

### 5、固定值布尔属性

比如 `checked` 属性:

```html
<input type="checkbox" name="option2" checked /> <!-- HTML -->
<input type="checkbox" name="option1" checked="checked" /> <!-- XHTML -->
```

标准方言包含允许您通过计算条件来设置这些属性的属性，这样如果表达式求值为true，则属性将设置为其固定值，如果求值为false，则不会设置该属性。如：

```html
<input type="checkbox" name="active" th:checked="${user.active}" />
```

 标准方言中存在以下固定值布尔属性：

| th:async   | th:autofocus | th:autoplay       |
| ---------- | ------------ | ----------------- |
| th:checked | th:default   | th:formnovalidate |
| th:loop |  th:nowrap | th:controls |
| th:defer |  th:hidden |  th:multiple |
| th:open | th:declare |  th:disabled |
| th:ismap |  th:novalidate | th:pubdate |
| th:readonly |  th:scoped | th:required |
| th:seamless | th:reversed | th:selected |

###  6、任意属性设置

Thymeleaf提供了一个默认的属性处理器，允许我们设置任何属性的值，即使没有特定的 `th：*` 处理器是用标准方言定义的。如：

```html
<span th:whatever="${user.name}">...</span>
```

解析后：

```html
<span whatever="John Apricot">...</span>
```

### 7、H5属性元素支持

`data-{prefix}-{name}` 语法是在HTML5中编写自定义属性的标准方法，无需开发人员可以使用任何名称空间的名称，有些类似 `th:*` 的用法。

比如：

```html
<table>
<tr data-th-each="user : ${users}">
<td data-th-text="${user.login}">...</td>
<td data-th-text="${user.name}">...</td>
</tr>
</table>
```

## 六、迭代

### 1、基本迭代

基本语法：`th:each` 

示例：

```html
<body>
<h1>Product list</h1>
<table>
    <tr>
        <th>NAME</th>
        <th>PRICE</th>
        <th>IN STOCK</th>
    </tr>
    <tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
<p>
	<a href="../home.html" th:href="@{/}">Return to home</a>
</p>
</body>
```

- 将 `${prods}` 称为迭代表达式或被迭代变量。
- 将 `prod` 称为迭代变量或简称iter变量。
- 注意 `prod` 迭代变量的作用域是`<tr>`元素，这意味着它可用于内部标记，如 `<td>` ，同时也将意味着迭代会产生多个 `<tr>` 元素。
- 可以用于迭代的对象不止`List`, 还有：
  - （ A ）任何实现了`java.util.Iterable` 或 `java.util.Enumeration` 或  `java.util.Iterator`(值将在迭代器返回时使用，不需要在内存中缓存所有值) 或 `java.util.Map`（ Map中迭代变量会是 `java.util.Map.Entry` 类）。
  - （ B ）可以是数组。
  - （ C ）任何其他对象都将被视为包含对象本身的单值列表

### 2、迭代时状态

当使用 `th:each` 时，Thymeleaf 提供了一种用于跟踪迭代状态的机制：状态变量。

状态变量在th:each属性中定义，并包含以下数据：

- 当前迭代索引，从0开始。`index` 属性。 

- 当前迭代索引，从1开始。`count` 属性。 

- 迭代变量中元素的总数。`size` 属性。 

- 每次迭代的iter变量。`current` 属性。 

- 当前迭代是偶数还是奇数。这些是 even /odd 布尔属性。 

- 当前迭代是否是第一个迭代。first 布尔属性。 

- 当前迭代是否是最后一个迭代。last 布尔属性。

示例：
```html
<table>
    <tr>
        <th>NAME</th>
        <th>PRICE</th>
        <th>IN STOCK</th>
    </tr>
    <tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```

​	状态变量（在本例中是 `iterStat`）是通过在 `iter`后面写入其名称在th:each属性中定义的变量本身，用逗号分隔。与iter变量一样，status变量的作用域也是由包含 `th:each` 属性的标记定义的代码。 如果我们不写也可以，Thymeleaf 也能识别：

```html
<table>
    <tr>
        <th>NAME</th>
        <th>PRICE</th>
        <th>IN STOCK</th>
    </tr>
        <tr th:each="prod : ${prods}" th:class="${prodStat.odd}? 'odd'">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```

### 3、延迟加载

有时，我们可能希望优化数据集合的检索（例如从数据库），以便只有当集合真的要被使用时才会被检索。实际上，这是一种可以应用于任何数据块的东西，但是可能指定了内存集合的大小，检索要迭代的集合是这种情况下最常见的情况。

为了支持这一点，Thymeleaf 提供了一种延迟加载上下文变量的机制。上下文变量实现 `ILazyContextVariable` 接口（很可能是通过扩展其 `LazyContextVariable` 默认值来实现）实现将在执行时解决。例如：

```java
context.setVariable(
"users",
new LazyContextVariable<List<User>>() {
@Override
protected List<User> loadValue() {
return databaseRepository.findAllUsers();
}
});
```

代码中，可以在不知道其懒散性的情况下使用此变量，为了保证是否执行，可以加个条件：

```html
<ul th:if="${condition}">
	<li th:each="u : ${users}" th:text="${u.name}">user name</li>
</ul>
```

## 七、条件表达式

### 1、简单条件表达式

`if` 和  `unless` 。

示例：

```html
<table>
    <tr>
        <th>NAME</th>
        <th>PRICE</th>
        <th>IN STOCK</th>
        <th>COMMENTS</th>
    </tr>
    <tr th:each="prod : ${prods}" th:class="${prodStat.odd}? 'odd'">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
        <td>
            <span th:text="${#lists.size(prod.comments)}">2</span> comment/s
            <a href="comments.html"
            th:href="@{/product/comments(prodId=${prod.id})}"
            th:if="${not #lists.isEmpty(prod.comments)}">view</a>
        </td>
    </tr>
</table>
```

可以看到 `<a>`标签被解析的条件是`prod.comments` 不能为空。

注意，`th:if` 属性不仅计算布尔条件。它的能力远不止于此，而且将根据以下规则将指定的表达式求值为 `true`：

（1）if 表达式值不为 null 的情况运算结果是`true`: 

- If 表达式值是个布尔值并且是true. 
- If 表达式值是个不为 0 的数字。
- If 表达式值是个不为 `'0'` 的字符。
- If 表达式值是个字符，但不是 “false”, “off” or “no” 
- If 表达式值不是布尔值、不是数字、不是字符、也不是字符串。

（2）if 表达式值为 null ，那么运算结果是 `false`  。    

`th:if` 的逆属性 `th:unless` 也可以使用。

```html
 <a href="comments.html"
            th:href="@{/product/comments(prodId=${prod.id})}"
            th:unless="${#lists.isEmpty(prod.comments)}">view</a>
```

### 2、切换选择语句

还有一种方法可以使用Java中的`switch` 结构有条件地显示内容：`th:switch` / `th:case` 属性集。

示例：

````html
<div th:switch="${user.role}">
    <p th:case="'admin'">User is an administrator</p>
    <p th:case="#{roles.manager}">User is a manager</p>
</div>
````

请注意，只要一个 `th:case` 属性被运算为 `true` ，则在同一个`switch` 中其他每一个 `th:case`属性运算结果为 `false` 。

万一所有的 `case` 都不中怎么办？可以设置默认选项呀~ 默认选项指定为`th:case="*"`：

```html
<div th:switch="${user.role}">
    <p th:case="'admin'">User is an administrator</p>
    <p th:case="#{roles.manager}">User is a manager</p>
    <p th:case="*">User is some other thing</p>
</div>
```

## 八、内联

### 1、表达式内联

表达式内联、也叫内联写法、行间写法。反正都是翻译的怎么叫都行，其实就是直接在HTML文本中写入表达式，不配合标签来使用。

示例：

```html
<p>Hello, [[${session.user.name}]]!</p
```

等价于：

```html
<p>Hello, <span th:text="${session.user.name}">Sebastian</span>!</p>
```

`[[...]]` 和 `[(...)]` 的区别类似 `th:text` 和 `th:utext`    

`[[...]]` 对应着 `th:text` ：转义特殊字符，内容是什么原样输出什么，浏览器不进行解析。

`[(...)]`对应着 `th:utext` ：不转义特殊字符，有标签时，浏览器会识别解析 html 标签。

注意：这个内联写法可以使用 `th:inline="none"` 禁用：

```html
<p th:inline="none">A double array looks like this: [[1, 2, 3], [4, 5]]!</p>
```

解析结果：

```html
<p>A double array looks like this: [[1, 2, 3], [4, 5]]!</p>
```



### 2、文本内联 

Textual template modes 文本模板模式

这个特征在页面开发中感觉使用并不是特别多，如果以后遇到再查API 进行补充学习。



### 3、JS脚本内联 

就是允许在`<javascript>` 脚本中使用 Thymeleaf 。

1) 基本使用

使用 `th:inline="javascript"` 来激活使用：

```javascript
<script th:inline="javascript">
...
var username = [[${session.user.name}]];
...
</script>
```

最终结果：

```javascript
<script th:inline="javascript">
...
var username = "Sebastian \"Fruity\" Applejuice";
...
</script>
```

需要注意：

（1）JavaScript 内联不仅输出所需的文本，还将其用引号括起来，并且JavaScript 转义其内容，以便将表达式结果作为格式良好的JavaScript文本输出。

（2）因为我们 `${session.user.name}` 输出，表达式为转义，如果使用双括号表达式：[[${session.user.name}]] ，表达式的结果就是非转义。就像这样：

```html
<script th:inline="javascript">
...
var username = [(${session.user.name})];
...
</script>
```

最终结果：

```html
<script th:inline="javascript">
...
var username = Sebastian "Fruity" Applejuice;
...
</script>
```

明显这样是一种错误的JS格式，浏览器肯定会报错。但是输出为转义的内容可能是我们需要的，比如通过附加内联表达式来实现脚本的一部分内容，也是一个比较不错的工具。

JavaScript内联机制的智能性远不止是应用特定于JavaScript的转义并将表达式结果作为有效文本输出。

例如，我们可以在JavaScript注释中包装（转义）内联表达式，例如：

```html
<script th:inline="javascript">
...
var username = /*[[${session.user.name}]]*/ "Gertrud Kiwifruit";
...
</script>
```

而Thymeleaf将忽略我们在注释之后和分号之前写的所有内容（在本例中 `'Gertrud Kiwifruit'`），因此执行此操作的结果将与不使用包装时完全相同：

```html
<script th:inline="javascript">
...
var username = "Sebastian \"Fruity\" Applejuice";
...
</script>
```

注意：这是如何有效的JavaScript代码。当你在静态模式下打开你的模板文件时，它将完美地执行方式（不在服务器上执行）。所以我们这里有一种方法来制作JavaScript自然模板！

2） 高级内联求值和JavaScript序列化:

关于JavaScript内联，需要注意的一点是，这个表达式计算是智能的，不局限于字符串。Thymeleaf将正确地用JavaScript语法编写以下类型的对象：

`Strings`、`Numbers` 、`Booleans`、 `Arrays`、 `Collections`、 `Maps`、 `Beans` (对象要求有getter和setter方法)    

示例：

```html
<script th:inline="javascript">
...
var user = /*[[${session.user}]]*/ null;
...
</script>
```

`${session.user}` 将会运算 user 对象，运行结果：

```html
<script th:inline="javascript">
...
var user = {"age":null,"firstName":"John","lastName":"Apricot",
"name":"John Apricot","nationality":"Antarctica"};
...
</script>
```

​    	JavaScript序列化的方法是通过 ` org.thymeleaf.standard.serializer.IStandardJavaScriptSerializer` 接口，可以在模板引擎中使用的`StandardDialect`的实例。 这个JS序列化机制的默认实现将在类路径中查找`Jackson`库，如果存在，将使用它。如果没有，它将应用一个内置的序列化机制来满足大多数场景和产生类似的结果（但灵活性较差） 



### 4、CSS样式内联

就是允许在`<style>` 样式脚本中使用 Thymeleaf 。

使用 `th:inline="css"` 来激活使用：

```html
<style th:inline="css">
...
</style>
```

比如我们有两个字符串变量：

```css
classname = 'main elems'
align = 'center
```

就可以直接使用：

```html
<style th:inline="css">
.[[${classname}]] {
text-align: [[${align}]];
}
</style>
```

最终解析结果：

```html
<style th:inline="css">
.main\ elems {
	text-align: center;
}
</style>
```

注意：

​	CSS内联也具有一些智能，就像JavaScript一样像 `[[${classname}]]` 这样的表达式将作为CSS标识符转义。这就是为什么上面的 `classname='main elems'`在代码片段中变成了`main\ elems`。

高级功能：CSS自然模板。

​	与前面对JavaScript的解释相同，CSS内联也允许 `<style>`标记工作静态和动态都可以，例如通过在注释中包装内联表达式作为CSS自然模板。

```html
<style th:inline="css">
.main\ elems {
text-align: /*[[${align}]]*/ left;
}
</style>
```

结果是：

```html
<style th:inline="css">
.main\ elems {
text-align: center;
}
</style>
```

## 九、注释和块

### 1、标准HTML/XML注释

基本语法：`<!--  ...   -->`

### 2、解析器级注释块

解析器级注释块是在Thymeleaf解析模板时将其从模板中简单删除的代码。

基本语法：`<!--/* and */-->`

```html
<!--/* This code will be removed at Thymeleaf parsing time! */-->
```

### 3、Thymeleaf原型仅注释块

当模板静态地（例如，作为一个原型）打开时，Thymeleaf允许定义标记为注释的特殊注释块，但是在执行模板时，Thymeleaf认为是正常的标记。

示例：

```html
<span>hello!</span>
<!--/*/
<div th:text="${...}">
...
</div>
/*/-->
<span>goodbye!</span>
```

Thymeleaf的解析系统将简单地删除 `<！--/*  */-->` 标记，但不包括其内容。因此没有注释。因此，在执行模板时，Thymeleaf将实际看到：

```html
<span>hello!</span>
<div th:text="${...}">
...
</div>
<span>goodbye!</span>
```

与解析器级别的注释块一样，这个特性与方言无关。

### 4、合成 th:block 标记

Thymeleaf标准方言中唯一的元素处理器（不是属性）是`th:block`。`th:block` 仅仅是一个属性容器，允许模板开发人员指定他们想要的任何属性。

Thymeleaf将执行这些属性，然后简单地使块（而不是其内容）消失。因此，例如，在创建每个元素需要多个`<tr>`的迭代表时，它可能很有用：

```html
<table>
<th:block th:each="user : ${users}">
    <tr>
        <td th:text="${user.login}">...</td>
        <td th:text="${user.name}">...</td>
    </tr>
    <tr>
    	<td colspan="2" th:text="${user.address}">...</td>
    </tr>
</th:block>
</table>
```

尤其是与原型仅注释块结合使用时非常有用：

```html
<table>
<!--/*/ <th:block th:each="user : ${users}"> /*/-->
<tr>
<td th:text="${user.login}">...</td>
<td th:text="${user.name}">...</td>
</tr>
<tr>
<td colspan="2" th:text="${user.address}">...</td>
</tr>
<!--/*/ </th:block> /*/-->
</table>
```

注意：

​	这个解决方案允许模板是有效的HTML（不需要在`<table>`中添加禁止的`<div>`块），以及作为原型在浏览器中静态打开时仍然可以正常工作！ 

