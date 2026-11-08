---
layout: default
title: Factory Method
parent: Design Patterns
has_children: false
nav_order: 14
---

Here are java Factory Method Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是工厂方法模式？

工厂方法模式（Factory Method Pattern）

工厂方法模式是一种创建型设计模式，它定义了一个用于创建对象的接口（或抽象类），但让子类决定实例化哪一个具体类。这样，对象的创建延迟到了子类，使得系统在不修改现有代码的情况下引入新的产品类型。

> 当创建一个对象需要很多步骤（或很多参数）时，建造者模式让你一步步地构造，最后一步才产出成品，并且可以灵活组合。

假设你有一个日志系统，需要输出到控制台和文件。如果直接在业务代码中用 new ConsoleLogger() 或 new FileLogger()，一旦要增加新的日志类型（如数据库日志），就必须修改所有创建日志的地方，违反了开闭原则（对扩展开放，对修改关闭）。
工厂方法模式通过引入创建者的层次结构来解决这个问题：每个具体创建者负责创建一种具体产品，客户端只需依赖抽象的创建者接口。


## 模式结构

```
┌─────────────────┐     build     ┌──────────────────┐
│   Creator       │──────────────▶│   Product        │
│ (abstract)      │               │ (interface/abs)  │
│ +factoryMethod()│               └────────┬─────────┘
│ +someOperation()│                        │
└────────┬────────┘                        │
         │ 继承                             │ 实现
┌────────▼────────┐               ┌────────▼─────────┐
│ ConcreteCreator │               │ ConcreteProduct  │
│ +factoryMethod()│──────────────▶│                  │
└─────────────────┘               └──────────────────┘
```

核心角色：

- Product（抽象产品）：定义产品的接口。
- ConcreteProduct（具体产品）：实现 Product 接口的具体类。
- Creator（抽象创建者）：声明工厂方法 factoryMethod()，该方法返回一个 Product 对象。Creator 也可以包含依赖于 Product 的业务逻辑（模板方法）。
- ConcreteCreator（具体创建者）：重写 factoryMethod()，返回一个具体的 ConcreteProduct 实例。


## 案例1
业务场景：多渠道支付网关

假设你正在开发一个电商平台的支付模块。平台需要支持多种支付方式：支付宝、微信支付、银行卡支付。

每种支付方式的创建逻辑、参数校验、签名算法都不同，而且未来很可能还会接入新的支付渠道（如 Apple Pay、PayPal）

不使用设计模式：
```java
// 客户端直接 new 具体支付对象
public class PaymentService {
    public void pay(String channel, double amount) {
        if ("alipay".equals(channel)) {
            AlipayPayment alipay = new AlipayPayment();
            alipay.pay(amount);
        } else if ("wechat".equals(channel)) {
            WechatPayment wechat = new WechatPayment();
            wechat.pay(amount);
        } else if ("card".equals(channel)) {
            CardPayment card = new CardPayment();
            card.pay(amount);
        }
        // 新增渠道时需要修改这里，违反开闭原则
    }
}
```
痛点：
- 每次新增支付渠道都要修改 PaymentService，容易引发回归 bug。
- 不同支付方式的初始化逻辑（获取商户号、设置回调 URL、加载证书）散落在各处。
- 单元测试困难，无法轻松 mock 未实现的渠道。

用工厂方法模式重构:

抽象产品：支付接口
```java
public interface Payment {
    Result pay(double amount);
    RefundResult refund(String orderId);
}
```

具体产品：各种支付实现

```java
public class AlipayPayment implements Payment {
    private String appId;
    private String privateKey;

    public AlipayPayment() {
        // 从配置中心加载支付宝专用配置
        this.appId = ConfigLoader.get("alipay.appId");
        this.privateKey = ConfigLoader.get("alipay.privateKey");
    }

    @Override
    public Result pay(double amount) {
        // 调用支付宝 SDK，构建签名等
        System.out.println("支付宝支付：" + amount + "元");
        return Result.success();
    }

    @Override
    public RefundResult refund(String orderId) {
        // ...
        return null;
    }
}
```

```java
public class WechatPayment implements Payment {
    private String mchId;
    private String apiKey;

    public WechatPayment() {
        this.mchId = ConfigLoader.get("wechat.mchId");
        this.apiKey = ConfigLoader.get("wechat.apiKey");
    }

    @Override
    public Result pay(double amount) {
        System.out.println("微信支付：" + amount + "元");
        return Result.success();
    }
    // ...
}
```

```java
public class CardPayment implements Payment {
    private String merchantId;
    public CardPayment() { /* 加载银联配置 */ }
    // ...
}
```

抽象创建者：支付工厂

```java
public abstract class PaymentFactory {
    // 工厂方法：子类实现具体的创建逻辑
    public abstract Payment createPayment();

    // 可选：模板方法，统一执行支付流程
    public Result executePay(double amount) {
        Payment payment = createPayment();
        // 可以在这里添加统一的前置处理（如风控检查、日志记录）
        return payment.pay(amount);
    }
}
```
具体创建者：每个渠道一个工厂

```java
public class AlipayPaymentFactory extends PaymentFactory {
    @Override
    public Payment createPayment() {
        return new AlipayPayment();
    }
}
```

```java
public class WechatPaymentFactory extends PaymentFactory {
    @Override
    public Payment createPayment() {
        return new WechatPayment();
    }
}
```

```java
public class CardPaymentFactory extends PaymentFactory {
    @Override
    public Payment createPayment() {
        return new CardPayment();
    }
}
```

客户端使用（通常配合策略模式或配置中心）
```java
public class OrderController {
    // 根据用户选择的支付方式，获取对应的工厂
    public PaymentFactory getFactory(String channel) {
        switch (channel) {
            case "alipay":  return new AlipayPaymentFactory();
            case "wechat":  return new WechatPaymentFactory();
            case "card":    return new CardPaymentFactory();
            default: throw new IllegalArgumentException("Unknown channel");
        }
    }

    public void checkout(String channel, double amount) {
        PaymentFactory factory = getFactory(channel);
        Result result = factory.executePay(amount);
        // 处理结果...
    }
}
```

如果新增一个支付渠道（如 Apple Pay）
你只需要：

1、新建 ApplePayPayment 实现 Payment 接口。

2、新建 ApplePayPaymentFactory 继承 PaymentFactory。

3、在 getFactory 中添加一个分支（或改为注册表模式彻底消除 switch）。

完全不修改已有的 AlipayPayment、WechatPayment、CardPayment 等类的代码。这就是开闭原则的体现。


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
- 符合开闭原则，新增产品无需修改现有代码。
- 将对象的创建和使用分离，降低耦合。
- 客户端只需要知道抽象产品和抽象工厂，不关心具体类名。

缺点：
- 每增加一个产品就需要增加一个具体工厂类，导致类数量翻倍。
- 增加了系统的抽象性和理解难度。


## 适用场景

一个类不知道它所需要的对象的类。
一个类希望由它的子类来指定所创建的对象。
将创建对象的逻辑委托给多个工厂子类中的一个，且客户端需要灵活切换。

## 实际应用

- Java 标准库：java.util.Collection 的 iterator() 方法就是一个工厂方法，返回不同类型的 Iterator。
- Spring Framework：BeanFactory 的 getBean() 本质上是一个工厂方法，根据名称或类型创建 bean 实例。
- JDBC：DriverManager.getConnection() 使用工厂方法模式创建不同数据库的连接。
