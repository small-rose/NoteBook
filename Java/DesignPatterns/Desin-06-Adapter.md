---
layout: default
title: Adapter Pattern
parent: Design Patterns
has_children: false
nav_order: 16
---

Here are java Adapter Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是适配器模式？

适配器模式（Adapter Pattern）

适配器模式是一种结构型设计模式，它允许将一个类的接口转换成客户端期望的另一个接口，从而使原本因接口不兼容而无法一起工作的类能够协同工作。

通俗地说，适配器就像一个电源转换插头——你从国外带回一个电器，它的插头形状与本国的插座不匹配，这时需要一个转换插头来适配。

## 模式结构

```
┌─────────────────┐      客户端期望的接口        ┌──────────────────┐
│     Target      │◀──────────────────────────│     Client       │
│ + request()     │                           └──────────────────┘
└────────┬────────┘
         ▲ 实现/继承
┌────────┴────────┐
│    Adapter      │
│ + request()     │
└────────┬────────┘
         │ 委派/调用
┌────────▼────────┐
│    Adaptee      │        （被适配者，已有类）
│ + specificReq() │
└─────────────────┘
```

核心角色：

- Target（目标接口）：客户端期望的接口。
- Adaptee（被适配者）：已经存在的、需要被适配的类。
- Adapter（适配器）：将 Adaptee 的接口转换为 Target 接口。
- Client（客户端）：通过 Target 接口与 Adapter 交互。

有两种实现方式：

- 类适配器：通过继承 Adaptee 并实现 Target 接口（Java 不支持多重继承，较少用）。
- 对象适配器：通过组合（持有 Adaptee 的引用）并实现 Target 接口（更常用）。
- 接口适配器模式：也叫默认适配器模式，使用一个抽象类先完成接口全部方法的空实现，在使用时，利用匿名内部类的方式去选择性的实现某个方法。

默认适配器结构：

```text
┌──────────────────┐
│    Target        │    （目标接口，包含多个方法）
│ + methodA()      │
│ + methodB()      │
│ + methodC()      │
│ + methodD()      │
└────────┬─────────┘
         ▲ 实现（抽象类提供空实现）
┌────────┴─────────┐
│ DefaultAdapter   │    （缺省适配器，抽象类）
│ + methodA() {}   │    // 默认空实现
│ + methodB() {}   │
│ + methodC() {}   │
│ + methodD() {}   │
└────────┬─────────┘
         ▲ 继承（子类选择性覆盖）
┌────────┴─────────┐
│ ConcreteService  │    （具体业务类）
│ + methodA()      │    // 只覆盖需要的方法
└──────────────────┘
```

## 案例1 对象适配器模型

业务场景：电源适配器，220V电源输出，但是手机只能接受5v，需要一个适配器中转

被适配的类：220V电压
```java
package com.xiaocai.designs.adapter.objectadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/20 23:17
 * @version: v1.0
 */
public class Voltage220V_Obj {

    private final int voltage = 220;

    public int output220V(){
        System.out.println("输入电压："+voltage +"V");
        return voltage;
    }
}
```

手机：适配器的使用者
```java
package com.xiaocai.designs.adapter.objectadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/20 23:16
 * @version: v1.0
 */
public class Phone_obj {

    public void charging(VoltageObjectAdapter voltageObjectAdapter){
        if(voltageObjectAdapter.output5V()==5){
            System.out.println("电压为 5V, 可以充电....");
        }else{
            System.out.println("电压不稳，不能充电...");
        }
    }
}
```
适配接口：
```java
public interface IVoltage5V_Obj {

    public int output5V();
}
```
电压适配器：
```java
package com.xiaocai.designs.adapter.objectadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：   ---- 对象适配器--- 关系从继承变为聚合，
 * @author: 张小菜
 * @date: 2020/11/20 23:16
 * @version: v1.0
 */
public class VoltageObjectAdapter implements IVoltage5V_Obj{

    private Voltage220V_Obj voltage220V ; // 聚合方式，合成-聚合原则

    public VoltageObjectAdapter(Voltage220V_Obj voltage220V){
        this.voltage220V = voltage220V ;
    }

    @Override
    public int output5V() {
        int dest = 0 ;
        if (null == voltage220V){
            System.out.println("对象配置，执行转换失败...");
        }else{
            int out = voltage220V.output220V();
            dest = out / 44 ;
            System.out.println("对象配置，执行转换...");
        }

        return dest;
    }
}
```
客户端调用：
```java
package com.xiaocai.designs.adapter.objectadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/11/20 23:18
 * @version: v1.0
 */
public class Client_Obj {

    public static void main(String[] args) {
        Phone_obj phone = new Phone_obj();
        phone.charging(new VoltageObjectAdapter(new Voltage220V_Obj()));
    }
}
```

> 通过组合（持有 Adaptee 的引用）并实现 Target 接口。

## 案例2 类适配器模型

> 通过继承 Adaptee 并实现 Target 接口（Java 不支持多重继承，较少用）。

被适配的类：
```java
package com.xiaocai.designs.adapter.classadapter;

/**
 * @description: TODO 功能角色说明：  被适配的类
 * TODO 描述：   -- 类适配器模式--被适配的类,可以输出220V 电压
 * @author: 张小菜
 * @date: 2020/11/20 22:54
 * @version: v1.0
 */
public class Voltage220V {

    private final int voltage = 220;

    public int output220V(){
        System.out.println("输入电压："+voltage +"V");
        return voltage;
    }
}
```
适配器的使用者:
```java
package com.xiaocai.designs.adapter.classadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：-- 类适配器模式-- 适配器的使用者
 * @author: 张小菜
 * @date: 2020/11/20 22:55
 * @version: v1.0
 */
public class Phone {

    public void charging(IVoltage5V iVoltage5V){
        if(iVoltage5V.output5V()==5){
            System.out.println("电压为 5V, 可以充电....");
        }else{
            System.out.println("电压不稳，不能充电...");
        }
    }
}
```
适配接口：
```java
package com.xiaocai.designs.adapter.classadapter;

/**
 *  适配接口
 */
public interface IVoltage5V {

    public int output5V();
}
```
适配过程：
```java
package com.xiaocai.designs.adapter.classadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：  -- 适配器类
 * @author: 张小菜
 * @date: 2020/11/20 22:54
 * @version: v1.0
 */
public class VoltageAdapter extends Voltage220V implements IVoltage5V{
    
    @Override
    public int output5V() {
        int out = output220V();
        int dest = out / 44 ;
        System.out.println("执行配置转换...");
        return dest;
    }
}
```
客户端：
```java
package com.xiaocai.designs.adapter.classadapter;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：  -- 类适配器调用：
 *      优势： 可以重写，适配度灵活
 *      不足： 类继承带来的耦合度
 * @author: 张小菜
 * @date: 2020/11/20 23:02
 * @version: v1.0
 */
public class Client {

    public static void main(String[] args) {
        Phone phone = new Phone();
        phone.charging(new VoltageAdapter());
    }
}
```

## 案例3 接口适配器

> 接口适配器通过提供一个抽象类，对该接口的所有方法给出默认的空实现（或默认行为），然后客户端只需要继承这个抽象类，并选择性覆盖自己需要的方法即可。

假设你有一个文件处理接口，定义了多种文件操作：上传、下载、删除、预览、分享。但在大多数业务场景中，客户端只需要其中一两个功能。

1. 目标接口（方法太多）
```java
public interface FileHandler {
    void upload(String path);
    void download(String path);
    void delete(String path);
    void preview(String path);
    void share(String path, String user);
}
```
2. 缺省适配器（抽象类，空实现所有方法）
```java
public abstract class FileHandlerAdapter implements FileHandler {
    @Override
    public void upload(String path) {
        // 默认空实现
    }

    @Override
    public void download(String path) {
        // 默认空实现
    }

    @Override
    public void delete(String path) {
        // 默认空实现
    }

    @Override
    public void preview(String path) {
        // 默认空实现
    }

    @Override
    public void share(String path, String user) {
        // 默认空实现
    }
}
```
3. 具体业务类（只覆盖需要的方法）
```java
// 图片上传服务：只需要上传和预览
public class ImageUploadService extends FileHandlerAdapter {
    @Override
    public void upload(String path) {
        System.out.println("上传图片到：" + path);
    }

    @Override
    public void preview(String path) {
        System.out.println("预览图片：" + path);
    }
}
```

```java
// 文档管理服务：只需要下载和删除
public class DocumentManageService extends FileHandlerAdapter {
    @Override
    public void download(String path) {
        System.out.println("下载文档：" + path);
    }

    @Override
    public void delete(String path) {
        System.out.println("删除文档：" + path);
    }
}
```
4. 客户端使用
```java
public class Client {
    public static void main(String[] args) {
        FileHandler imageHandler = new ImageUploadService();
        imageHandler.upload("/images/photo.jpg");   // 上传图片到：/images/photo.jpg
        imageHandler.preview("/images/photo.jpg");  // 预览图片：/images/photo.jpg
        // 其他方法调用不会有任何副作用（空实现）

        FileHandler docHandler = new DocumentManageService();
        docHandler.download("/docs/report.pdf");    // 下载文档：/docs/report.pdf
        docHandler.delete("/docs/report.pdf");      // 删除文档：/docs/report.pdf
    }
}
```
如果不使用接口适配器，ImageUploadService 必须实现 FileHandler 的全部五个方法，其中三个方法只能写空方法体，造成代码冗余。


##  优缺点

优点：
- 开闭原则：无需修改现有代码即可引入新功能（只需新增适配器）。
- 单一职责：将接口转换的逻辑集中到适配器中，客户端和被适配者各司其职。
- 复用性：可以让任何类与现有系统协作，即使它们的接口不匹配。

缺点：
- 增加系统复杂度：需要额外引入适配器类，尤其是当需要适配多个接口时。
- 过度使用：如果系统中有大量适配器，可能说明接口设计本身存在问题，应该考虑重构而非适配。

## 适用场景

- 使用第三方库或旧系统：它们的接口与你的系统不兼容，且无法修改。
- 统一多个相似但不同接口的类：例如不同的日志框架（Log4j、Logback、JUL）通过 SLF4J 适配器统一。
- 重用现有类：现有类的功能满足需求，但接口不符合预期。
- 接口转换：需要将一种数据格式转换为另一种（如 XML 转 JSON）。



## 实际应用

| 领域    | 应用方式       |
|-------|-----------------|
| 日志框架  | SLF4J 是典型的适配器模式，它为 Log4j、Logback、JUL 等提供了统一的门面接口，底层通过适配器桥接。   |
| Spring 框架 | HandlerAdapter 用于适配不同类型的处理器（Controller、HttpRequestHandler 等），让 DispatcherServlet 统一调用。 |
| JDBC 驱动 | JDBC 定义了统一接口（Connection、Statement、ResultSet），各个数据库厂商提供各自的适配器实现。  |
| GUI 工具包 | AWT 的 ComponentAdapter 将底层操作系统的事件转换为 Java 事件。 |
| 缓存系统  | JCache（JSR-107）定义了标准缓存接口，各种缓存实现（Ehcache、Caffeine）通过适配器接入。  |
| 电源适配器 | 现实世界中的插头转换器就是适配器模式的完美类比。     |


## 总结：

适配器模式的核心是“让不兼容的接口协同工作”，它通过一层包装，将现有类的接口转换成客户端期望的形式。在系统集成、第三方库接入、接口统一等场景中非常实用。记住一个生活类比：出国旅行时带的转换插头，就是适配器模式的最佳写照。

接口适配器（缺省适配器）是适配器模式的一种特殊形式，专门用于简化对庞大接口的实现。它通过一个抽象类提供默认的空实现，让客户端只需关注自己真正需要的方法。在 Java 的 GUI 事件处理、Netty 等框架中广泛使用。
