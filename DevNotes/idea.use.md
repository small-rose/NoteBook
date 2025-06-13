---
layout: default
title: IDEA About
parent: DevNotes
nav_order: 2
---

# Idea
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


## 护眼色


护眼色

绿豆沙色能有效的减轻长时间用电脑的用眼疲劳！

色调：85，饱和度：123，亮度：205；

RGB颜色红：199，绿：237，蓝：204；

十六进制颜色：#C7EDCC  或用 #CCE8CF


<table  width="200px">
<tr>
<td style="background-color: #C7EDCC" width="40px" height="40px">#C7EDCC</td>
<td style="background-color: #CCE8CF" width="40px" height="40px">#CCE8CF</td>
<td style="background-color: #FAF9DE" width="40px" height="40px">#FAF9DE</td>
<td style="background-color: #FFF2E2" width="40px" height="40px">#FFF2E2</td>
<td style="background-color: #FDE6E0" width="40px" height="40px">#FDE6E0</td>
</tr>
<tr>
<td style="background-color: #E3EDCD" width="40px" height="40px">#E3EDCD</td>
<td style="background-color: #DCE2F1" width="40px" height="40px">#DCE2F1</td>
<td style="background-color: #E9EBFE" width="40px" height="40px">#E9EBFE</td>
<td style="background-color: #EAEAEF" width="40px" height="40px">#EAEAEF</td>
<td style="background-color: #B7E8BD" width="40px" height="40px">#B7E8BD</td>
</tr>
<tr>
<td style="background-color: #19CAAD" width="40px" height="40px">#19CAAD</td>
<td style="background-color: #8CC7B5" width="40px" height="40px">#8CC7B5</td>
<td style="background-color: #A0EEE1" width="40px" height="40px">#A0EEE1</td>
<td style="background-color: #BEE7E9" width="40px" height="40px">#BEE7E9</td>
<td style="background-color: #BEEDC7" width="40px" height="40px">#BEEDC7</td>
</tr>
<tr>
<td style="background-color: #D6D5B7" width="40px" height="40px">#D6D5B7</td>
<td style="background-color: #D1BA74" width="40px" height="40px">#D1BA74</td>
<td style="background-color: #E6CEAC" width="40px" height="40px">#E6CEAC</td>
<td style="background-color: #ECAD9E" width="40px" height="40px">#ECAD9E</td>
<td style="background-color: #F4606C" width="40px" height="40px">#F4606C</td>

</tr>
</table>



其他几种电脑窗口视力保护色：

```
银河白    #FFFFFF    RGB(255, 255, 255)   

杏仁黄    #FAF9DE    RGB(250, 249, 222)

秋叶褐    #FFF2E2    RGB(255, 242, 226)

胭脂红    #FDE6E0    RGB(253, 230, 224)

青草绿    #E3EDCD    RGB(227, 237, 205)

海天蓝    #DCE2F1    RGB(220, 226, 241)

葛巾紫    #E9EBFE    RGB(233, 235, 254)

极光灰    #EAEAEF    RGB(234, 234, 239)
```

**IDEA设置背景图片** 

路径：File->Settings->Appearance&Behavior->Appearance

**idea编辑区** 

手动设置护眼绿的RGB颜色

路径：File->Settings->Editor->Color Scheme->General (右侧- Text - Default text) 

 **idea项目区** 

File ->Settings -> Appearance & Behavior  -> File Colors  ，


选择+号 选择Project  file

console背景修改

  编辑代码区的BGC:setting-->editor-->color Scheme -->console Colors-->右侧Console-->background
  
## Idea字符集乱码

VM Options :  -Dfile.encoding=URF-8

## Idea自动导包

setting -> Editor -> Auto Import

- 勾选 Add unambiguous imports on the fly
- 勾选 Optimize imports on the fly

## Idea禁止导*

setting -> Editor -> Code Style –> Java –> Imports

- 勾选 Use single class import
- Class count to use import with *  :  99999
- Names count to use static import with * : 99999


## Idea内存使用优化

开启idea底下的工具栏 memory indicator 的展示。

修改VM参数： 

针对当前用户的修改
> Help -> Edit Custom VM Options....

针对当idea的修改
> JetBrains\IntelliJ IDEA 2020.1\bin\idea64.exe.vmoptions

```text
-Xms1024m
-Xmx2048m
-XX:MaxMetaspaceSize=512m
-XX:ReservedCodeCacheSize=256m
-XX:+UseG1GC
-XX:SoftRefLRUPolicyMSPerMB=50
-ea
-XX:CICompilerCount=2
-Dsun.io.useCanonPrefixCache=false
-Djdk.http.auth.tunneling.disabledSchemes=""
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Djdk.attach.allowAttachSelf=true
-Dkotlinx.coroutines.debug=off
-Djdk.module.illegalAccess.silent=true
-Dfile.encoding=UTF-8
```

## 常用的快捷键

用于在IDEA中查看类结构和方法结构：

查看类结构

    Ctrl + H（或Cmd + H在Mac上）：这个快捷键用于查看当前类的层次结构，包括它的父类、实现的接口以及继承关系等。这是理解类在项目中位置和作用的重要工具。
    Alt + 7：在IDEA中，你也可以通过按下Alt键和数字7的组合键来打开Structure视图，该视图展示了当前类的字段、方法、内部类等结构信息。
    Ctrl + F12：这个快捷键可以显示当前文件的结构，允许你快速跳转到文件中的任何方法、字段或属性。

查看方法结构

    Alt + 7 打开Structure视图后，你可以在其中看到当前类的所有方法，并且可以通过点击来跳转到具体的方法实现。
    Ctrl + B（或Cmd + B在Mac上）：这个快捷键用于跳转到方法或变量的声明处。如果你想查看某个方法的具体实现，特别是当它在接口中被声明时，这个快捷键非常有用。
    Ctrl + Alt + B：这个快捷键更进一步，它允许你跳转到方法的具体实现处，而不是仅仅停留在声明处。这对于理解接口的具体实现非常有帮助。

其他相关快捷键

    Ctrl + N：根据输入的类名快速查找并打开类文件。
    Ctrl + F：在当前文件中查找文本。
    Ctrl + Shift + F：在项目中查找文本，可以指定搜索范围。
    Alt + Insert：自动生成代码，如getter/setter方法、构造函数等。
    Ctrl + Alt + L：格式化代码，使代码更加整洁易读。


## idea好用的插件

IDE Eval Reset : 试用插件

MavenHelper  ： Maven依赖检查

GitToolBox  ： Git插件可以显示行的git变化版本信息

MybatisX  : Mybatis 插件

Mybatis-Log-For-Free ：Mybatis 打印SQL插件

jclasslib bytecode viewer ： jclasslib is a bytecode viewer for Java class files

generateO2O : 2个对象快速赋值生成插件

background Image plus ：设置背景图插件

generateAllSetter：set赋值





## Live template 使用教程

相关内容 custom postfix template 插件

settings-live template

1、新建Group

2、新建Live Template

3、命名快捷方式tag

4、编写快捷方式模板

5、声明使用范围

6、可选项。绑定函数

日志打印

命名：logm

模板：

```txt
log.debug("$METHOD_NAME$::$PLACE_HOLDERS$",$ARGUMENTS$);
```

变量：

METHOD_NAME 

```txt
# 打印方法名称
methodName()
```

PLACE_HOLDERS 变量

```groovy
groovyScript("_1.collect { it + ' = [{}]'}.join(', ') ", methodParameters())
```

ARGUMENTS 变量

```groovy
groovyScript( 
"def result=''; 
def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); 
for(i = 0; i < params.size(); i++) {
	result+='JSON.toJSONString(' + params[i] + ((i < params.size() - 1) ? '), ' : ')')
};
 return result" , methodParameters())
```


另外一套

PLACE_HOLDERS 变量

```groovy
groovyScript("def params = _2.collect {'【'+it+' = {}】'}.join(', '); return '\"' + _1 + '() called with parameters => ' + (params.empty  ? '' : params) + '\"'", methodName(), methodParameters())
```

ARGUMENTS 变量

```groovy
groovyScript("def params = _1.collect {it}.join(', '); return   (params.empty  ? '' : params) ",  methodParameters())
```

-------------------------------------------------

打印方法参数

命名：logm

模板：
```
log.info($content$,$params$);
```

变量：

content变量：
```groovy
groovyScript(
"def params = _2.collect {'【'+it+' = {}】'}.join(', '); 
return '\"' + _1 + '() called with parameters => ' + (params.empty  ? '' : params) + '\"'", methodName(), methodParameters())
```

params变量：
```groovy
groovyScript(
"def params = _1.collect {it}.join(', '); 
return   (params.empty  ? '' : params) ",  methodParameters())
```

-------------------------------------------------

打印方法返回值

命名： logr

模板：

```
log.info("$METHOD_NAME$() returned: " +  $result$);
```

变量：

```
METHOD_NAME 变量:    methodName()
result 变量:     variableOfType(methodReturnType())
```

-------------------------------------------------


