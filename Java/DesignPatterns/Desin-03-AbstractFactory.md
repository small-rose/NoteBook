---
layout: default
title: Abstract Factory
parent: Design Patterns
has_children: false
nav_order: 13
---

Here are java   Singleton Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是工厂方法模式？

抽象工厂模式（Abstract Factory Pattern）

抽象工厂模式是一种创建型设计模式，它提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们的具体类。
它与工厂方法模式的区别在于：工厂方法针对单个产品等级结构，而抽象工厂针对多个产品等级结构组成的“产品族”。


假设你是一家大型家电制造商，旗下有 海尔 和 美的 两个品牌。每个品牌都需要生产三种产品：空调、冰箱、洗衣机。
如果你用工厂方法模式，你需要为每个品牌的每种产品都创建一个工厂（海尔空调工厂、海尔冰箱工厂……），导致工厂类爆炸。
更重要的是，客户端必须确保同一品牌的产品一起使用（不能把海尔的空调和美的的冰箱混搭）。
抽象工厂模式通过将同一品牌的所有产品封装到一个工厂中来解决这个问题。

## 模式结构

```
        ┌──────────────────────┐      create A         ┌──────────────────┐
        │    AbstractFactory   │──────────────────────▶│ AbstractProductA │
        │ + createProductA()   │                       └────────┬─────────┘
        │ + createProductB()   │                                │
        │ + createProductC()   │                                │ 实现
        └─────────┬────────────┘                      ┌────────▼─────────┐
                  │ 继承                               │ ConcreteProductA1│
        ┌─────────▼────────────┐                      │ ConcreteProductA2│
        │ ConcreteFactory1     │                      └──────────────────┘
        │ + createProductA()   │
        │ + createProductB()   │      create B         ┌──────────────────┐
        │ + createProductC()   │──────────────────────▶│ AbstractProductB │
        └──────────────────────┘                       └────────┬─────────┘
                                                                │ 实现
                                                        ┌────────▼─────────┐
                                                        │ ConcreteProductB1│
                                                        │ ConcreteProductB2│
                                                        └──────────────────┘
```

核心角色：

- AbstractProduct：抽象产品，定义一类产品的接口（如空调、冰箱）。
- ConcreteProduct：具体产品，属于某个产品族（如海尔空调、美的冰箱）。
- AbstractFactory：抽象工厂，声明创建各个抽象产品的方法。
- ConcreteFactory：具体工厂，实现创建某个产品族的所有具体产品。


## 案例1

业务场景：多品牌家电生产

1. 抽象产品接口
```java
// 空调
public interface AirConditioner {
    void cool();
}

// 冰箱
public interface Refrigerator {
    void freeze();
}

// 洗衣机
public interface Washer {
    void wash();
}
```

2. 具体产品（海尔家族）
```java
public class HaierAirConditioner implements AirConditioner {
    @Override
    public void cool() {
        System.out.println("海尔空调制冷，静音节能");
    }
}

public class HaierRefrigerator implements Refrigerator {
    @Override
    public void freeze() {
        System.out.println("海尔冰箱风冷无霜");
    }
}

public class HaierWasher implements Washer {
    @Override
    public void wash() {
        System.out.println("海尔洗衣机滚筒变频");
    }
}
```
3. 具体产品（美的家族）
```java
public class MideaAirConditioner implements AirConditioner {
    @Override
    public void cool() {
        System.out.println("美的空调制冷，智能控温");
    }
}

public class MideaRefrigerator implements Refrigerator {
    @Override
    public void freeze() {
        System.out.println("美的高端冰箱，双循环系统");
    }
}

public class MideaWasher implements Washer {
    @Override
    public void wash() {
        System.out.println("美的洗衣机，DD直驱电机");
    }
}
```
4.抽象工厂

```java
public interface HomeApplianceFactory {
    AirConditioner createAirConditioner();
    Refrigerator createRefrigerator();
    Washer createWasher();
}
```
5.具体工厂
```java
public class HaierFactory implements HomeApplianceFactory {
    @Override
    public AirConditioner createAirConditioner() {
        return new HaierAirConditioner();
    }

    @Override
    public Refrigerator createRefrigerator() {
        return new HaierRefrigerator();
    }

    @Override
    public Washer createWasher() {
        return new HaierWasher();
    }
}

public class MideaFactory implements HomeApplianceFactory {
    @Override
    public AirConditioner createAirConditioner() {
        return new MideaAirConditioner();
    }

    @Override
    public Refrigerator createRefrigerator() {
        return new MideaRefrigerator();
    }

    @Override
    public Washer createWasher() {
        return new MideaWasher();
    }
}
```

客户端使用（装修公司）

```java
public class DecorationCompany {
    private HomeApplianceFactory factory;

    public DecorationCompany(HomeApplianceFactory factory) {
        this.factory = factory;  // 由配置文件或用户选择决定
    }

    public void equipHome() {
        AirConditioner ac = factory.createAirConditioner();
        Refrigerator fridge = factory.createRefrigerator();
        Washer washer = factory.createWasher();

        ac.cool();
        fridge.freeze();
        washer.wash();
        System.out.println("--- 同一品牌家电安装完毕 ---");
    }

    public static void main(String[] args) {
        // 用户选择了海尔
        DecorationCompany haierHome = new DecorationCompany(new HaierFactory());
        haierHome.equipHome();

        // 用户选择了美的
        DecorationCompany mideaHome = new DecorationCompany(new MideaFactory());
        mideaHome.equipHome();
    }
}
```


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
最终输出
```text
海尔空调制冷，静音节能
海尔冰箱风冷无霜
海尔洗衣机滚筒变频
--- 同一品牌家电安装完毕 ---
美的空调制冷，智能控温
美的高端冰箱，双循环系统
美的洗衣机，DD直驱电机
--- 同一品牌家电安装完毕 ---
```

关键点：客户端只依赖 HomeApplianceFactory 接口，切换品牌只需换一个工厂对象，不会出现跨品牌混搭。

 
##  优缺点

优点：
 - 保证产品族的一致性：同一工厂创建的产品天然属于同一系列，避免混搭问题。
 - 符合开闭原则：增加新的产品族（如格力）只需新增具体工厂和具体产品，无需修改现有代码。
 - 分离具体类：客户端只使用抽象接口，不关心具体产品是如何创建的。


缺点：
 - 扩展产品等级结构困难：如果需要增加新的产品类型（如烤箱），必须修改抽象工厂接口，进而修改所有具体工厂类（违背开闭原则）。
 - 类数量膨胀：每个产品族都需要一套具体产品类和具体工厂类，系统复杂度较高。



## 适用场景

- 系统需要独立于产品的创建、组合和表示。
- 系统需要配置多个产品族中的某一个，且产品族之间存在约束（必须一起使用）。
- 强调相关产品对象的协同使用，且不希望客户端混搭。
- 产品等级结构相对稳定，而产品族经常变化。


## 实际应用

|领域 |抽象工厂 |产品族 |产品等级|
|-----|-------|-------|-------|
|跨平台 GUI | Qt / Java Swing | Windows 风格、Mac 风格、Linux 风格 | 按钮、文本框、菜单|
|数据库访问 |DAO 工厂 |MySQL 工厂、Oracle 工厂、PostgreSQL 工厂 |Connection、Statement、ResultSet
|云服务商 |云资源工厂| AWS 工厂、Azure 工厂、阿里云工厂| 虚拟机、存储、数据库|
|游戏引擎| 关卡元素工厂 |森林主题、沙漠主题、城堡主题 | 树木、石头、敌人|
|报表系统 | 报表样式工厂| 简约风格、商务风格、科技风格| 图表、表格、页眉|

## 总结

抽象工厂模式是工厂方法模式的升级版，专门用于创建一组相关对象。它保证了客户端始终使用同一产品族的对象，避免了不兼容的组合。当你的系统存在多个产品族，且产品族内部的成员需要协同工作时，抽象工厂就是最合适的选择。
