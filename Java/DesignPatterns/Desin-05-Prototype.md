---
layout: default
title: Prototype Pattern
parent: Design Patterns
has_children: false
nav_order: 15
---

Here are java   Prototype Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是原型模式？

原型模式是一种创建型设计模式，它通过复制一个已有对象（称为原型） 来创建新对象，而不是通过 new 关键字调用构造器。

核心思想是：用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。
> 当创建一个对象需要很多步骤（或很多参数）时，建造者模式让你一步步地构造，最后一步才产出成品，并且可以灵活组合。

考虑一个场景：你有一个复杂的对象，它的构造过程涉及大量的数据库查询、网络调用或复杂的初始化逻辑。如果用 new 每次都从头构建，性能开销巨大。
另外，有些对象的状态在运行时才能确定（如用户配置的报表模板），你希望基于一个现有的实例快速生成另一个相似但略有不同的实例。

原型模式通过克隆来解决这些问题——不需要知道对象的具体类，也不需要重复执行昂贵的构造过程。

## 模式结构

```
        ┌──────────────────┐
        │    Prototype     │    （抽象原型）
        │ + clone()        │
        └────────┬─────────┘
                 │ 实现
        ┌────────▼──────────┐
        │ ConcretePrototype │   （具体原型）
        │ + clone()         │
        └───────────────────┘
```

核心角色：

- Prototype：声明克隆自身的接口（通常是 clone() 方法）。
- ConcretePrototype：实现克隆方法，返回自身的一个副本。
- Client：通过调用原型对象的 clone() 方法来创建新对象。

## Java 中的原型模式

Java 提供了 Cloneable 接口（标记接口）和 Object.clone() 方法作为原型的原生支持。
但是需要注意：
- clone() 是 protected 方法，必须在子类中重写并公开。
- 默认的 clone() 是浅拷贝（只复制基本类型和引用地址，不复制引用指向的对象）。
- 如果需要深拷贝，必须手动重写 clone() 方法。


## 案例1
业务场景：工作流审批表单模板

假设你有一个 OA 系统，里面有各种审批表单：请假单、报销单、出差单。

每种表单都有固定的结构（标题、申请人、部门、审批节点列表），但每次发起审批时，需要填入具体的内容（请假天数、报销金额等）。

如果每次都从数据库读取模板并重新组装，效率低下。更好的做法是：预先创建好模板原型，使用时克隆一份再填充内容。

抽象原型：支付接口
```java
public abstract class FormPrototype implements Cloneable {
    protected String title;
    protected String applicant;
    protected String department;
    protected List<String> approvalNodes;

    // 克隆方法
    @Override
    public FormPrototype clone() {
        try {
            // 浅拷贝
            return (FormPrototype) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Clone failed", e);
        }
    }

    // 抽象方法：子类实现具体的填写逻辑
    public abstract void fillContent(Map<String, Object> params);

    // getters/setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    // ... 其他 getter/setter
}
```

2. 具体原型：请假单

```java
public class LeaveForm extends FormPrototype {
    public LeaveForm() {
        this.title = "请假申请单";
        this.approvalNodes = new ArrayList<>(Arrays.asList("直属主管", "HR"));
    }

    @Override
    public void fillContent(Map<String, Object> params) {
        this.applicant = (String) params.get("applicant");
        this.department = (String) params.get("department");
        // 可以进一步填充请假天数等
    }
}
```
3. 具体原型：报销单
```java
public class ExpenseForm extends FormPrototype {
    public ExpenseForm() {
        this.title = "费用报销单";
        this.approvalNodes = new ArrayList<>(Arrays.asList("部门经理", "财务", "总经理"));
    }

    @Override
    public void fillContent(Map<String, Object> params) {
        this.applicant = (String) params.get("applicant");
        this.department = (String) params.get("department");
    }
}
```
4. 原型管理器（可选）
```java
public class FormPrototypeManager {
    private static Map<String, FormPrototype> prototypes = new HashMap<>();

    static {
        prototypes.put("leave", new LeaveForm());
        prototypes.put("expense", new ExpenseForm());
    }

    public static FormPrototype getForm(String type) {
        return prototypes.get(type).clone();
    }
}
```
 
客户端使用
```java
public class Client {
    public static void main(String[] args) {
        // 用户发起请假申请
        FormPrototype leaveForm = FormPrototypeManager.getForm("leave");
        Map<String, Object> params = new HashMap<>();
        params.put("applicant", "张三");
        params.put("department", "研发部");
        leaveForm.fillContent(params);

        System.out.println(leaveForm.getTitle());       // 请假申请单
        System.out.println(leaveForm.getApplicant());   // 张三

        // 另一用户发起报销
        FormPrototype expenseForm = FormPrototypeManager.getForm("expense");
        // ... 填充不同内容
    }
}
```
优势：

无需每次从数据库加载模板定义，直接克隆内存中的原型，速度快。
每个克隆出来的对象都是独立的，修改其中一个不影响原型和其他克隆体。
 

其他业务型案例速览

|业务领域  |工厂方法产品  |工厂类|
|--------|-----------|------|
|通知服务 |短信通知、邮件通知、App Push  |SmsNotifierFactory、EmailNotifierFactory|
|优惠券发放 |满减券、折扣券、免邮券 |FullReductionCouponFactory、DiscountCouponFactory|
|文件导出 |Excel导出、PDF导出、CSV导出 |ExcelExporterFactory、PdfExporterFactory|
|第三方登录 |微信登录、QQ登录、微博登录 |WechatAuthFactory、QQAuthFactory|
|数据清洗 |去除空格、脱敏、格式校验 |TrimProcessorFactory、DesensitizeProcessorFactory|

>总结：
> 
> 工厂方法模式在业务系统中的核心价值是将“创建哪种对象”的决策权交给子类，让主业务流程不依赖具体实现。当你的业务规则需要支持多种同类但不同实现的“渠道”时，它就是最佳选择。

##  优缺点

优点：
- 性能提升：避免重复执行昂贵的构造过程（如数据库查询、网络调用）。
- 简化对象创建：客户端不需要知道对象的具体类，只需知道原型接口。
- 动态添加/移除原型：可以在运行时注册新的原型，无需修改代码。
- 保存对象状态：可以随时保存当前状态并克隆出相同状态的新对象（如撤销操作）。


缺点：
- 深拷贝实现复杂：如果对象引用链很深，需要递归处理所有引用对象。
- 需要实现 Cloneable：Java 中必须显式实现 Cloneable 接口，否则抛异常。
- 循环引用问题：如果对象之间存在循环引用，深拷贝可能导致无限递归。



## 适用场景

- 对象创建成本高昂（如需要加载配置文件、建立网络连接、查询数据库）。
- 对象状态只有少数几种变体，且变体之间差异不大（如不同角色的用户权限模板）。
- 需要避免构造器的复杂逻辑，直接基于现有实例复制。
- 运行时动态指定创建对象的类型，且该类型可能未知或频繁变化。


## 实际应用

|领域  |应用方式|
|-----|--------|
|Java 标准库 | Object.clone() 本身就是原型模式的实现。|
|Spring 框架 | Bean 的作用域 prototype：每次请求都创建一个新实例，但 Spring 通过反射 newInstance 实现，并非克隆；不过可以通过配置 @Scope(value="prototype") 达到类似效果。
|游戏开发| 创建大量相似的敌人、子弹、粒子效果，通过克隆原型避免重复加载资源。|
|图形编辑器| 复制一个图形对象（如矩形、圆形），保持其样式属性不变，只需修改位置。|
|缓存系统| 从缓存中取出一个原型对象，克隆后修改部分属性再使用，避免破坏缓存中的原始对象。|
|ORM 框架| Hibernate 的一级缓存中，从数据库查询出的对象可以作为原型，用于快速创建脱管对象。|

总结：
原型模式最适合那些构造过程昂贵、状态可变且需要频繁创建相似对象的场景。在 Java 中，利用 Cloneable 和 clone() 可以实现浅拷贝，但深拷贝通常需要额外的工作。


## 浅拷贝 vs 深拷贝

浅拷贝（默认 clone()）
- 基本类型：复制值。
- 引用类型：复制引用地址，新旧对象共享同一个引用对象。

问题：在上面的例子中，approvalNodes 是一个 ArrayList，浅拷贝后所有克隆体共享同一个审批节点列表。如果修改其中一个的审批节点，会影响所有其他克隆体。

深拷贝的实现方式：
- 序列化/反序列化（实现 Serializable，然后通过 ObjectOutputStream 和 ObjectInputStream）。
- JSON 序列化（如 Jackson 的 readValue(writeValueAsString(...), ...)）。
- 手动递归拷贝。

## 深拷贝案例：


```text
package com.xiaocai.designs.prototype.deepclone;

import java.io.Serializable;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/10/21 11:55
 * @version: v1.0
 */
public class DeepCloneableTarget implements Serializable ,Cloneable {

    private String cloneName ;
    private String cloneClass ;

    // 构造器
    public DeepCloneableTarget(String cloneName, String cloneClass) {
        this.cloneName = cloneName;
        this.cloneClass = cloneClass;
    }

    // TODO 因为此类的属性都是string，可以使用默认的 clone 方法完成
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    @Override
    public String toString() {
        return "DeepCloneableTarget{" +
                "cloneName='" + cloneName + '\'' +
                ", cloneClass=" + cloneClass +
                '}';
    }
}
```

原型对象实现 Serializable, Cloneable 接口。
```text
package com.xiaocai.designs.prototype.deepclone;

import java.io.*;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/10/21 11:58
 * @version: v1.0
 */
public class DeepPrototype implements Serializable, Cloneable {

    private String name ; //String 属性
    public DeepCloneableTarget deepCloneableTarget ;// 引用类型

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public DeepCloneableTarget getDeepCloneableTarget() {
        return deepCloneableTarget;
    }

    public void setDeepCloneableTarget(DeepCloneableTarget deepCloneableTarget) {
        this.deepCloneableTarget = deepCloneableTarget;
    }

    public DeepPrototype() {
        super();
    }

    @Override
    public String toString() {
        return "DeepPrototype{" +
                "name='" + name + '\'' +
                ", deepCloneableTarget=" + deepCloneableTarget +
                '}';
    }

    //TODO 深拷贝---方式一 使用 clone 方法
    @Override
    protected Object clone()  {

        Object deep = null ;
        DeepPrototype deepPrototype = null;
        try {
            //TODO 先完成属性（ 基本类型和String ）的克隆
            deep = super.clone();
            //TODO 针对引用类型的属性，进行单独处理
            deepPrototype = (DeepPrototype) deep;
            deepPrototype.deepCloneableTarget = (DeepCloneableTarget) deepCloneableTarget.clone();
        }catch (CloneNotSupportedException e){
            e.printStackTrace();
        }


        return deepPrototype;
    }


    //TODO 深拷贝---方式一 使用 序列化 和反序列化

    public   Object serializedDeepClone(){
        DeepPrototype deepPrototype = null;
        ByteArrayOutputStream bos = null;
        ObjectOutputStream oos  = null ;
        ByteArrayInputStream bis = null ;
        ObjectInputStream ois = null;
        try {
            // 序列号操作
            bos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            // 反序列化
            bis = new ByteArrayInputStream(bos.toByteArray()); //重新读出来
            ois = new ObjectInputStream(bis);
            deepPrototype = (DeepPrototype) ois.readObject();

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                bos.close();
                oos.close();
                bis.close();
                ois.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return deepPrototype;
    }

}

```

调用
```text
package com.xiaocai.designs.prototype.deepclone;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/10/21 12:05
 * @version: v1.0
 */
public class Client {

    public static void main(String[] args) throws CloneNotSupportedException {
        
        DeepPrototype prototype = new DeepPrototype();
        prototype.setName("张三");
        DeepCloneableTarget target = new DeepCloneableTarget("李四", "李四的类");
        prototype.setDeepCloneableTarget(target);

        // TODO 方式一 完成 深拷贝
        DeepPrototype copy = (DeepPrototype)prototype.clone();
        System.out.println("prototype : " + prototype + "copy.refrence hashcode : " + prototype.getDeepCloneableTarget().hashCode());
        System.out.println("copy : " + copy + "copy.refrence hashcode : " + copy.getDeepCloneableTarget().hashCode());

        System.out.println("-------------------------------------------");

        // TODO 方式二 完成 深拷贝
        DeepPrototype copy2 = (DeepPrototype)prototype.serializedDeepClone();
        System.out.println("prototype : " + prototype + "copy.refrence hashcode : " + prototype.getDeepCloneableTarget().hashCode());
        System.out.println("copy2 : " + copy2 + " copy2.refrence hashcode : " + copy2.getDeepCloneableTarget().hashCode());
    }
}
```