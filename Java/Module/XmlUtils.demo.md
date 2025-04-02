---
layout: default
title: Java  XmlUtils
parent: Java
nav_order: 93
---

Here are async Export code experience .
{: .fs-6 .fw-300 }


## Here are XmlUtils code demo .
{: .no_toc .text-delta }

1. TOC
{:toc}


## XmlUtils


```java

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Unmarshaller;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

public class XmlUtils{

    /**
        setFeature 方法用于配置 DocumentBuilderFactory 的特性，以控制 XML 解析器的行为。通过设置特定的特性，可以启用或禁用某些功能，从而提高安全性。以下是禁用外部实体解析的特性及其原因：
        1. http://xml.org/sax/features/external-general-entities：
        作用：控制是否允许解析外部通用实体。
        设置为 false：禁用外部通用实体的解析，防止攻击者通过 XML 文件引用外部资源。
        http://xml.org/sax/features/external-parameter-entities：
        作用：控制是否允许解析外部参数实体。
        设置为 false：禁用外部参数实体的解析，进一步防止外部实体的攻击。
        http://apache.org/xml/features/disallow-doctype-decl：
        作用：控制是否允许文档类型声明（DOCTYPE）。
        设置为 true：禁用 DOCTYPE 声明，防止攻击者利用 DOCTYPE 声明来引入外部实体。
    **/
    public static <T> T xmlToBean(String xmlStr, Class<T> beanClass) {
       try {
           StringReader stringReader = new StringReader(xmlStr);
           JAXBContext jaxbContext = JAXBContext.newInstance(beanClass);
        
           // 创建 Unmarshaller 并禁用外部实体
           Unmarshaller unmarshaller = jaxbContext.createUnmarshaller();
           DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
           dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
           dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
           dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
           
           return (T) unmarshaller.unmarshal(stringReader);
       } catch (JAXBException e) {
           System.out.println("XML 转换为 Bean 时发生错误：" + e.getMessage());
           return null; // 返回 null 表示转换失败
       }
    }

    public static String beanToXml(Object bean) {
       try {
           JAXBContext jaxbContext = JAXBContext.newInstance(bean.getClass());
           Marshaller marshaller = jaxbContext.createMarshaller();
           
           // 设置格式化输出
           marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);
           
           // 使用 StringWriter 将 Bean 转换为 XML 字符串
           StringWriter writer = new StringWriter();
           marshaller.marshal(bean, writer);
           
           return writer.toString(); // 返回 XML 字符串
       } catch (JAXBException e) {
           System.out.println("Bean 转换为 XML 时发生错误：" + e.getMessage());
           return null; // 返回 null 表示转换失败
       }
    }

}
```