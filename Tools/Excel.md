---
layout: default
title: Excel
nav_order: 1
parent: Tools
---

# Excel
{: .no_toc }

Record some excel skills that can be used for development, such as functions.
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Excel 常用函数

判断字段类型

### IF函数

```text
=IF(COUNTIF(B2,"*number*")>0,"BigDecimal",
IF(COUNTIF(B2,"*varchar*")>0,"String",
IF(COUNTIF(B2,"*datetime*")>0,"Date",
IF(COUNTIF(B2,"*decimal*")>0,"BigDecimal","不能识别的类型"))))
```


### 驼峰命名

```text
=LOWER(LEFT((SUBSTITUTE((PROPER(A2)),"_","")),1))&RIGHT((SUBSTITUTE((PROPER(A2)),"_","")),LEN((SUBSTITUTE((PROPER(A2)),"_","")))-1)
```


### 截取函数

Excel的截取函数有很多，最常用的有三种，即Left、Right、Mid。

```text
=LEFT(text,[num_chars])
=Right(text,[num_chars])
=MID(text【提取字符的源数据】, start_num【开始位置】, num_chars【截取长度】)
```

“num_chars”参数是大于等于0的整数，省略默认为1。

示例： Mid（F2,7,8）

F2是给定的Excel工作簿中身份证号所在单元格，身份证号从第7位是出生年月日的开始数据，年月日一共8位数据。

### 间接引用函数 INDIRECT

(1)引用单元格数据数值
    
(2)配合名称管理器，引用名称管理器的名字地址，得到下拉框数据

(3)多个名称管理器组合成联动下拉框。