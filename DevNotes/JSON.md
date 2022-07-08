---
layout: default
title: JSON-fastjson
parent: DevNotes
nav_order: 6
---

# JSON fastjson
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## JSON 相关


### fastjson 

依赖包

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.4</version>
        </dependency>
```

### @JSONField

使用 `@JSONField` 可以让JSON字符串里的key和Bean的属性不一致时，映射赋值到对应的属性上。

```java
@Data
public class MyBean {

    @JSONField(name = "FLAG")
    private String status ;

    @JSONField(name = "RETURN_MSG")
    private String returnMsg ;
}
```

使用将JSON字符串解析成对应的Bean:

```java
public class ParseTest {

    public static void main(String[] args) {

        String data = "{\"FLAG\":\"0\",\"RETURN_MSG\":\"SUCCESS\"}";
        MyBean request = JSON.parseObject(data, MyBean.class);
        System.out.println(request);
    }
}
```

输出结果：
```
MyBean(status=0, returnMsg=SUCCESS)
```
Bean转成对应的JSON字符串:
```java
public class FormatTest {

    public static void main(String[] args) {

        MyBean bean = new MyBean();
        bean.setStatus("1");
        bean.setReturnMsg("你的参数没有输入");
        String data = JSONObject.toJSONString(bean);
        System.out.println(data);
    }
}
```
输出结果：
```
{"FLAG":"1","RETURN_MSG":"你的参数没有输入"}
```


### fastjson 排序 

（1）可以通过ordinal指定字段的顺序。这个特性需要1.1.42以上版本。

```java
@Data
public class Rules {

    @JSONField(ordinal = 1)
    private String id ;

    @JSONField(ordinal = 2)
    private String door ;
    
    @JSONField(ordinal = 3)
    private Category category ;
    
    @JSONField(ordinal = 4)
    private CategoryPages categoryPages;
    
    @JSONField(ordinal = 5)
    private LinkGroup linkGroup ;

    @JSONField(ordinal = 6)
    private LinkGroupPages linkGroupPages ;

    @JSONField(ordinal = 7)
    private PicLink picLink ;
}

```
 

（2）可以通过name的值大小指定字段的顺序。

```java
@Data
public class Rules {

    @JSONField(name="a")
    private String id ;

    @JSONField(name="b")
    private String door ;
    
    @JSONField(name="c")
    private Category category ;
    
    @JSONField(name="d")
    private CategoryPages categoryPages;
    
    @JSONField(name="e")
    private LinkGroup linkGroup ;

    @JSONField(name="f")
    private LinkGroupPages linkGroupPages ;

    @JSONField(name="g")
    private PicLink picLink ;
}

```
