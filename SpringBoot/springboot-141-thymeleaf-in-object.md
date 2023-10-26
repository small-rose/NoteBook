---
layout: default
title: Thymeleaf Object
parent: SpringBoot
nav_order: 141
---

## Thymeleaf  内置对象

Thymeleaf 的内置对象主要有三类

- 请求/会话属性的Web上下文命名空间对象
- Web 上下文对象
- 内置工具对象



## 一. 请求/会话属性等

请求/会话属性等的Web上下文命名空间对象。如下

### #ctx

`org.thymeleaf.context.IContext ` API提供

```
${#ctx.locale}
${#ctx.variableNames}

```

`class org.thymeleaf.context.IWebContext` API提供

```
${#ctx.request}
${#ctx.response}
${#ctx.session}
${#ctx.servletContext}

```

### #locale

直接访问java.util.Locale与当前请求关联。

```
${#locale}
```

### param

不是上下文对象，而是作为变量添加到上下文中的映射，不使用#来访问它们。param类似名称空间。

`org.thymeleaf.context.WebRequestParamsVariablesMap` API提供

```
${param.foo}
${param.fooArray[0]}
${param.size()}
${param.isEmpty()}
${param.containsKey('foo')}

```

### session

`org.thymeleaf.context.WebSessionVariablesMap` 提供API

```
${session.foo} // Retrieves the session atttribute 'foo'
${session.size()}
${session.isEmpty()}
${session.containsKey('foo')}
```

### application

用于检索 `application/servlet`上下文属性。

```
${application.foo} // Retrieves the ServletContext atttribute 'foo'
${application.size()}
${application.isEmpty()}
${application.containsKey('foo')}
```



## 二. Web 上下文对象

Web 上下文对象。如下：

### #request :

直接访问 `javax.servlet.http.HttpServletRequest` 对象与当前请求关联

```
${#request.getAttribute('foo')}
${#request.getParameter('foo')}
${#request.getContextPath()}
${#request.getRequestName()}

```

### #session :

直接访问 `javax.servlet.http.HttpSession` 对象与当前请求关联

```
${#session.getAttribute('foo')}
${#session.id}
${#session.lastAccessedTime}

```

### #servletContext :

直接访问 `javax.servlet.ServletContext` 对象与当前请求关联

```
${#servletContext.getAttribute('foo')}
${#servletContext.contextPath}

```



## 三. 内置工具对象

### 1. Execution Info

`#execInfo` ：提供有关在Thymeleaf 内部正在处理的模板的有用信息标准表达式。

由 `org.thymeleaf.expression.ExecutionInfo` 提供的API

```
/*
* Return the name and mode of the 'leaf' template. This means the template
* from where the events being processed were parsed. So if this piece of
* code is not in the root template "A" but on a fragment being inserted
* into "A" from another template called "B", this will return "B" as a
* name, and B's mode as template mode.
*/
${#execInfo.templateName}
${#execInfo.templateMode}
/*
* Return the name and mode of the 'root' template. This means the template
* that the template engine was originally asked to process. So if this
* piece of code is not in the root template "A" but on a fragment being
* inserted into "A" from another template called "B", this will still
* return "A" and A's template mode.
*/
${#execInfo.processedTemplateName}
${#execInfo.processedTemplateMode}
/*
* Return the stacks (actually, List<String> or List<TemplateMode>) of
* templates being processed. The first element will be the
* 'processedTemplate' (the root one), the last one will be the 'leaf'
* template, and in the middle all the fragments inserted in nested
* manner to reach the leaf from the root will appear.
*/
${#execInfo.templateNames}
${#execInfo.templateModes}
/*
* Return the stack of templates being processed similarly (and in the
* same order) to 'templateNames' and 'templateModes', but returning
* a List<TemplateData> with the full template metadata.
*/
${#execInfo.templateStack}
```

### 2. Messages

`#message` ：在变量表达式中获取外部化消息的工具方法。也是使用 `#{ }` 语法。

由 `org.thymeleaf.expression.Messages` 提供API。    

```
/*
* Obtain externalized messages. Can receive a single key, a key plus arguments,
* or an array/list/set of keys (in which case it will return an array/list/set of
* externalized messages).
* If a message is not found, a default message (like '??msgKey??') is returned.
*/
${#messages.msg('msgKey')}
${#messages.msg('msgKey', param1)}
${#messages.msg('msgKey', param1, param2)}
${#messages.msg('msgKey', param1, param2, param3)}
${#messages.msgWithParams('msgKey', new Object[] {param1, param2, param3, param4})}
${#messages.arrayMsg(messageKeyArray)}
${#messages.listMsg(messageKeyList)}
${#messages.setMsg(messageKeySet)}
/*
* Obtain externalized messages or null. Null is returned instead of a default
* message if a message for the specified key is not found.
*/
${#messages.msgOrNull('msgKey')}
${#messages.msgOrNull('msgKey', param1)}
${#messages.msgOrNull('msgKey', param1, param2)}
${#messages.msgOrNull('msgKey', param1, param2, param3)}
${#messages.msgOrNullWithParams('msgKey', new Object[] {param1, param2, param3, param4})}
${#messages.arrayMsgOrNull(messageKeyArray)}
${#messages.listMsgOrNull(messageKeyList)}
${#messages.setMsgOrNull(messageKeySet)}
```

### 3. URI/URL

`#uris`：用于在 Thymeleaf 标准表达式中进行URI/URL操作的工具对象。

比如:URL参数中有特殊字符可以进行转义或取消转义，中文乱码可以进行编码转码。

由 `org.thymeleaf.expression.Uris` 提供API。

```
/*
* Escape/Unescape as a URI/URL path
*/
${#uris.escapePath(uri)}
${#uris.escapePath(uri, encoding)}
${#uris.unescapePath(uri)}
${#uris.unescapePath(uri, encoding)}
/*
* Escape/Unescape as a URI/URL path segment (between '/' symbols)
*/
${#uris.escapePathSegment(uri)}
${#uris.escapePathSegment(uri, encoding)}
${#uris.unescapePathSegment(uri)}
${#uris.unescapePathSegment(uri, encoding)}
/*
* Escape/Unescape as a Fragment Identifier (#frag)
*/
${#uris.escapeFragmentId(uri)}
${#uris.escapeFragmentId(uri, encoding)}
${#uris.unescapeFragmentId(uri)}
${#uris.unescapeFragmentId(uri, encoding)}
/*
* Escape/Unescape as a Query Parameter (?var=value)
*/
${#uris.escapeQueryParam(uri)}
${#uris.escapeQueryParam(uri, encoding)}
${#uris.unescapeQueryParam(uri)}
${#uris.unescapeQueryParam(uri, encoding)}
```

### 4. Conversions

`#conversions`：允许在模板的任何点执行转换服务。

由 `org.thymeleaf.expression.Conversions` 提供API。

```
/*
* Execute the desired conversion of the 'object' value into the
* specified class.
*/
${#conversions.convert(object, 'java.util.TimeZone')}
${#conversions.convert(object, targetClass)}
```

### 5. Dates

`#dates` :  其实就和 `java.util.Date`对象的工具方法类似。

由 `org.thymeleaf.expression.Dates` 提供API。

```
/*
* 格式化成 标准区域格式时间
* 支持 arrays, lists or sets 操作
*/
${#dates.format(date)}
${#dates.arrayFormat(datesArray)}
${#dates.listFormat(datesList)}
${#dates.setFormat(datesSet)}
/*
* 格式化成  ISO8601时间
* 支持 arrays, lists or sets 操作
*/
${#dates.formatISO(date)}
${#dates.arrayFormatISO(datesArray)}
${#dates.listFormatISO(datesList)}
${#dates.setFormatISO(datesSet)}
/*
* 格式化日期时间成自定义的格式
* 支持 arrays, lists or sets 操作
*/
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}
/*
* 获取日期属性
* 支持 arrays, lists or sets 操作
*/
${#dates.day(date)} // also arrayDay(...), listDay(...), etc.
${#dates.month(date)} // also arrayMonth(...), listMonth(...), etc.
${#dates.monthName(date)} // also arrayMonthName(...), listMonthName(...), etc.
${#dates.monthNameShort(date)} // also arrayMonthNameShort(...), listMonthNameShort(...), etc.
${#dates.year(date)} // also arrayYear(...), listYear(...), etc.
${#dates.dayOfWeek(date)} // also arrayDayOfWeek(...), listDayOfWeek(...), etc.
${#dates.dayOfWeekName(date)} // also arrayDayOfWeekName(...), listDayOfWeekName(...), etc.
${#dates.dayOfWeekNameShort(date)} // also arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...), etc.
${#dates.hour(date)} // also arrayHour(...), listHour(...), etc.
${#dates.minute(date)} // also arrayMinute(...), listMinute(...), etc.
${#dates.second(date)} // also arraySecond(...), listSecond(...), etc.
${#dates.millisecond(date)} // also arrayMillisecond(...), listMillisecond(...), etc.
/*
* 使用自带组件创建日期 (java.util.Date) 对象
*/
${#dates.create(year,month,day)}
${#dates.create(year,month,day,hour,minute)}
${#dates.create(year,month,day,hour,minute,second)}
${#dates.create(year,month,day,hour,minute,second,millisecond)}
/*
* 使用自带组件创建当前日期或时间 (java.util.Date)
*/
${#dates.createNow()}
${#dates.createNowForTimeZone()}
/*
* 使用自带组件创建当前日期 (java.util.Date)，时间是00:00
*/
${#dates.createToday()}
${#dates.createTodayForTimeZone()}
```

### 6. Calendar

`#calendar` ：类似 `#dates` ，但是是 `java.util.Calendar` 对象。

由 `class org.thymeleaf.expression.Calendars` 提供API。

```
/*
* Format calendar with the standard locale format
* Also works with arrays, lists or sets
*/
${#calendars.format(cal)}
${#calendars.arrayFormat(calArray)}
${#calendars.listFormat(calList)}
${#calendars.setFormat(calSet)}
/*
* Format calendar with the ISO8601 format
* Also works with arrays, lists or sets
*/
${#calendars.formatISO(cal)}
${#calendars.arrayFormatISO(calArray)}
${#calendars.listFormatISO(calList)}
${#calendars.setFormatISO(calSet)}
/*
* Format calendar with the specified pattern
* Also works with arrays, lists or sets
*/
${#calendars.format(cal, 'dd/MMM/yyyy HH:mm')}
${#calendars.arrayFormat(calArray, 'dd/MMM/yyyy HH:mm')}
${#calendars.listFormat(calList, 'dd/MMM/yyyy HH:mm')}
${#calendars.setFormat(calSet, 'dd/MMM/yyyy HH:mm')}
/*
* 获取日历属性
* Also works with arrays, lists or sets
*/
${#calendars.day(date)} // also arrayDay(...), listDay(...), etc.
${#calendars.month(date)} // also arrayMonth(...), listMonth(...), etc.
${#calendars.monthName(date)} // also arrayMonthName(...), listMonthName(...), etc.
${#calendars.monthNameShort(date)} // also arrayMonthNameShort(...), listMonthNameShort(...), etc.
${#calendars.year(date)} // also arrayYear(...), listYear(...), etc.
${#calendars.dayOfWeek(date)} // also arrayDayOfWeek(...), listDayOfWeek(...), etc.
${#calendars.dayOfWeekName(date)} // also arrayDayOfWeekName(...), listDayOfWeekName(...), etc.
${#calendars.dayOfWeekNameShort(date)} // also arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...), etc.
${#calendars.hour(date)} // also arrayHour(...), listHour(...), etc.
${#calendars.minute(date)} // also arrayMinute(...), listMinute(...), etc.
${#calendars.second(date)} // also arraySecond(...), listSecond(...), etc.
${#calendars.millisecond(date)} // also arrayMillisecond(...), listMillisecond(...), etc.
/*
* Create calendar (java.util.Calendar) objects from its components
*/
${#calendars.create(year,month,day)}
${#calendars.create(year,month,day)}
${#calendars.create(year,month,day,hour,minute)}
${#calendars.create(year,month,day,hour,minute,second)}
${#calendars.create(year,month,day,hour,minute,second,millisecond)}
${#calendars.createForTimeZone(year,month,day,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,millisecond,timeZone)}
/*
* 创建当前日期和时间的日历对象(java.util.Calendar) 
*/
${#calendars.createNow()}
${#calendars.createNowForTimeZone()}
/*
*   创建当前日期 (time set to 00:00)的日历对象(java.util.Calendar) 
*/
${#calendars.createToday()}
${#calendars.createTodayForTimeZone()}
```

### 7. Numbers

`#numbers` ：数字对象的工具对象方法。

`org.thymeleaf.expression.Numbers` 提供API。

 integer 类型格式化

```
/*
* Set minimum integer digits.
* Also works with arrays, lists or sets
*/
${#numbers.formatInteger(num,3)}
${#numbers.arrayFormatInteger(numArray,3)}
${#numbers.listFormatInteger(numList,3)}
${#numbers.setFormatInteger(numSet,3)}
/*
* Set minimum integer digits and thousands separator:
* 'POINT', 'COMMA', 'WHITESPACE', 'NONE' or 'DEFAULT' (by locale).
* Also works with arrays, lists or sets
*/
${#numbers.formatInteger(num,3,'POINT')}
${#numbers.arrayFormatInteger(numArray,3,'POINT')}
${#numbers.listFormatInteger(numList,3,'POINT')}
${#numbers.setFormatInteger(numSet,3,'POINT')}
```

decimal   类型格式化

````
/*
* Set minimum integer digits and (exact) decimal digits.
* Also works with arrays, lists or sets
*/
${#numbers.formatDecimal(num,3,2)}
${#numbers.arrayFormatDecimal(numArray,3,2)}
${#numbers.listFormatDecimal(numList,3,2)}
${#numbers.setFormatDecimal(numSet,3,2)}
/*
* Set minimum integer digits and (exact) decimal digits, and also decimal separator.
* Also works with arrays, lists or sets
*/
${#numbers.formatDecimal(num,3,2,'COMMA')}
${#numbers.arrayFormatDecimal(numArray,3,2,'COMMA')}
${#numbers.listFormatDecimal(numList,3,2,'COMMA')}
${#numbers.setFormatDecimal(numSet,3,2,'COMMA')}
/*
* Set minimum integer digits and (exact) decimal digits, and also thousands and
* decimal separator.
* Also works with arrays, lists or sets
*/
${#numbers.formatDecimal(num,3,'POINT',2,'COMMA')}
${#numbers.arrayFormatDecimal(numArray,3,'POINT',2,'COMMA')}
${#numbers.listFormatDecimal(numList,3,'POINT',2,'COMMA')}
${#numbers.setFormatDecimal(numSet,3,'POINT',2,'COMMA')}
````

货币格式化：

```
${#numbers.formatCurrency(num)}
${#numbers.arrayFormatCurrency(numArray)}
${#numbers.listFormatCurrency(numList)}
${#numbers.setFormatCurrency(numSet)}
```

百分百格式化：

```
${#numbers.formatPercent(num)}
${#numbers.arrayFormatPercent(numArray)}
${#numbers.listFormatPercent(numList)}
${#numbers.setFormatPercent(numSet)}
```

设置最小整数位数和（精确）小数位数。

```
${#numbers.formatPercent(num, 3, 2)}
${#numbers.arrayFormatPercent(numArray, 3, 2)}
${#numbers.listFormatPercent(numList, 3, 2)}
${#numbers.setFormatPercent(numSet, 3, 2)}
```

创建一个从x到y整数序列（数组）

```
${#numbers.sequence(from,to)}
${#numbers.sequence(from,to,step)}
```

### 8. Strings

`#strings` : 字符串对象实用工具对象。

由 `org.thymeleaf.expression.Strings` 提供API。

Null安全的转字符串：

```
${#strings.toString(obj)} // also array*, list* and set*
```

判断是否为空

```
/*
* Check whether a String is empty (or null). Performs a trim() operation before check
* Also works with arrays, lists or sets
*/
${#strings.isEmpty(name)}
${#strings.arrayIsEmpty(nameArr)}
${#strings.listIsEmpty(nameList)}
${#strings.setIsEmpty(nameSet)}
```

带默认值的判空

```
/*
* Perform an 'isEmpty()' check on a string and return it if false, defaulting to
* another specified string if true.
* Also works with arrays, lists or sets
*/
${#strings.defaultString(text,default)}
${#strings.arrayDefaultString(textArr,default)}
${#strings.listDefaultString(textList,default)}
${#strings.setDefaultString(textSet,default)}
```

判断片段是否包含指定字符串

```
/*
* Check whether a fragment is contained in a String
* Also works with arrays, lists or sets
*/
${#strings.contains(name,'ez')} // also array*, list* and set*
${#strings.containsIgnoreCase(name,'ez')} // also array*, list* and set*
```

判断以某字符串开始或结束：

```
/*
* Check whether a String starts or ends with a fragment
* Also works with arrays, lists or sets
*/
${#strings.startsWith(name,'Don')} // also array*, list* and set*
${#strings.endsWith(name,endingFragment)} // also array*, list* and set*
```

字符串截取相关：

```
/*
* Substring-related operations
* Also works with arrays, lists or sets
*/
${#strings.indexOf(name,frag)} // also array*, list* and set*
${#strings.substring(name,3,5)} // also array*, list* and set*
${#strings.substringAfter(name,prefix)} // also array*, list* and set*
${#strings.substringBefore(name,suffix)} // also array*, list* and set*
${#strings.replace(name,'las','ler')} // also array*, list* and set*
```

字符串追加：

```
/*
* Append and prepend
* Also works with arrays, lists or sets
*/
${#strings.prepend(str,prefix)} // also array*, list* and set*
${#strings.append(str,suffix)} // also array*, list* and set*
```

字符串大小写转换

```
/*
* Change case
* Also works with arrays, lists or sets
*/
${#strings.toUpperCase(name)} // also array*, list* and set*
${#strings.toLowerCase(name)} // also array*, list* and set*
```

字符串分割或连接

```
/*
* Split and join
*/
${#strings.arrayJoin(namesArray,',')}
${#strings.listJoin(namesList,',')}
${#strings.setJoin(namesSet,',')}
${#strings.arraySplit(namesStr,',')} // returns String[]
${#strings.listSplit(namesStr,',')} // returns List<String>
${#strings.setSplit(namesStr,',')} // returns Set<String>
```

去空格：

```
/*
* Trim
* Also works with arrays, lists or sets
*/
${#strings.trim(str)} // also array*, list* and set*
```

计算字符串长度：

```
/*
* Compute length
* Also works with arrays, lists or sets
*/
${#strings.length(str)} // also array*, list* and set*
```

缩写文本使其最大大小为n。如果文本更大，它将被剪辑并以“…”结尾

```
${#strings.abbreviate(str,10)} // also array*, list* and set*
```

将字符串第一个字符转换为大写或小写

```
${#strings.capitalize(str)} // also array*, list* and set*
${#strings.unCapitalize(str)} // also array*, list* and set*
```

将字符串中每个单词的第一个字符转换为大写

```
${#strings.capitalizeWords(str)} // also array*, list* and set*
${#strings.capitalizeWords(str,delimiters)} // also array*, list* and set*
```

字符串转义

```
${#strings.escapeXml(str)} // also array*, list* and set*
${#strings.escapeJava(str)} // also array*, list* and set*
${#strings.escapeJavaScript(str)} // also array*, list* and set*
Page 98 of 106${#strings.escapeJavaScript(str)} // also array*, list* and set*
${#strings.unescapeJava(str)} // also array*, list* and set*
${#strings.unescapeJavaScript(str)} // also array*, list* and set*
```

Null-Safe 的比较和串联 

```
${#strings.equals(first, second)}
${#strings.equalsIgnoreCase(first, second)}
${#strings.concat(values...)}
${#strings.concatReplaceNulls(nullValue, values...)}
```

随机数

```
${#strings.randomAlphanumeric(count)}
```

### 9.Objects

`#objects` ：对象的常用工具方法。

`org.thymeleaf.expression.Objects` 提供API。

```
/*
* Return obj if it is not null, and default otherwise
* Also works with arrays, lists or sets
*/
${#objects.nullSafe(obj,default)}
${#objects.arrayNullSafe(objArray,default)}
${#objects.listNullSafe(objList,default)}
${#objects.setNullSafe(objSet,default)}
```

### 10. Booleans

`#booleans` ：布尔运算的实用方法。

`org.thymeleaf.expression.Bool` 提供API。

```
/*
* Evaluate a condition in the same way that it would be evaluated in a th:if tag
* (see conditional evaluation chapter afterwards).
* Also works with arrays, lists or sets
*/
${#bools.isTrue(obj)}
${#bools.arrayIsTrue(objArray)}
${#bools.listIsTrue(objList)}
${#bools.setIsTrue(objSet)}
/*
* Evaluate with negation
* Also works with arrays, lists or sets
*/
${#bools.isFalse(cond)}
${#bools.arrayIsFalse(condArray)}
${#bools.listIsFalse(condList)}
${#bools.setIsFalse(condSet)}
/*
* Evaluate and apply AND operator
* Receive an array, a list or a set as parameter
*/
${#bools.arrayAnd(condArray)}
${#bools.listAnd(condList)}
${#bools.setAnd(condSet)}
/*
* Evaluate and apply OR operator
* Receive an array, a list or a set as parameter
*/
${#bools.arrayOr(condArray)}
${#bools.listOr(condList)}
${#bools.setOr(condSet)}Arrays

```

### 11. Arrays

`#arrays` ： 数组工具对象。

`org.thymeleaf.expression.Arrays`  提供API。

```
/*
* Converts to array, trying to infer array component class.
* Note that if resulting array is empty, or if the elements
* of the target object are not all of the same class,
* this method will return Object[].
*/
${#arrays.toArray(object)}
/*
* Convert to arrays of the specified component class.
*/
${#arrays.toStringArray(object)}
${#arrays.toIntegerArray(object)}
${#arrays.toLongArray(object)}
${#arrays.toDoubleArray(object)}
${#arrays.toFloatArray(object)}
${#arrays.toBooleanArray(object)}
/*
* Compute length
*/
${#arrays.length(array)}
/*
* Check whether array is empty
*/
${#arrays.isEmpty(array)}
/*
* Check if element or elements are contained in array
*/
${#arrays.contains(array, element)}
${#arrays.containsAll(array, elements)}
```

### 12. Lists

`#Lists` ： List 集合工具对象。

`org.thymeleaf.expression.Lists`  提供API。

```
/*
* Converts to list
*/
${#lists.toList(object)}
/*
* Compute size
*/
${#lists.size(list)}
/*
* Check whether list is empty
*/
${#lists.isEmpty(list)}
/*
* Check if element or elements are contained in list
*/
${#lists.contains(list, element)}
${#lists.containsAll(list, elements)}
/*
* Sort a copy of the given list. The members of the list must implement
* comparable or you must define a comparator.
*/
${#lists.sort(list)}
${#lists.sort(list, comparator)}
```

### 13. Sets

`#Sets` ： Sets 集合工具对象。

`org.thymeleaf.expression.Sets`  提供API。

```
/*
* Converts to set
*/
${#sets.toSet(object)}
/*
* Compute size
*/
${#sets.size(set)}
/*
* Check whether set is empty
*/
${#sets.isEmpty(set)}
/*
* Check if element or elements are contained in set
*/
${#sets.contains(set, element)}
${#sets.containsAll(set, elements)}
```

### 14. Maps

`#Maps` ： Maps 集合工具对象。

`org.thymeleaf.expression.Maps`  提供API。

```
/*
* Compute size
*/
${#maps.size(map)}
/*
* Check whether map is empty
*/
${#maps.isEmpty(map)}
/*
* Check if key/s or value/s are contained in maps
*/
${#maps.containsKey(map, key)}
${#maps.containsAllKeys(map, keys)}
${#maps.containsValue(map, value)}
${#maps.containsAllValues(map, value)}
```

### 15. Aggregates

`#Aggregates` ： 在数组或集合上创建聚合的实用方法。

`org.thymeleaf.expression.Aggregates`  提供聚合API。

```
/*
* Compute sum. Returns null if array or collection is empty
*/
${#aggregates.sum(array)}
${#aggregates.sum(collection)}
/*
* Compute average. Returns null if array or collection is empty
*/
${#aggregates.avg(array)}
${#aggregates.avg(collection)}
```

### 16. Ids

`#Ids`：处理可能重复的id属性的实用方法（例如，作为迭代的结果）。

`org.thymeleaf.expression.Aggregates.Ids`  提供API。

```
/*
* Normally used in th:id attributes, for appending a counter to the id attribute value
* so that it remains unique even when involved in an iteration process.
*/
${#ids.seq('someId')}
/*
* Normally used in th:for attributes in <label> tags, so that these labels can refer to Ids
* generated by means if the #ids.seq(...) function.
* *
Depending on whether the <label> goes before or after the element with the #ids.seq(...)
* function, the "next" (label goes before "seq") or the "prev" function (label goes after
* "seq") function should be called.
*/
${#ids.next('someId')}
${#ids.prev('someId')}
```



