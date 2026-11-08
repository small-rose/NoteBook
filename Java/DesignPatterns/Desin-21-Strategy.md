---
layout: default
title: Strategy Pattern
parent: Design Patterns
has_children: false
nav_order: 31
---

Here are java Strategy Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是策略模式？

策略模式（Strategy Pattern）

策略模式是一种行为型设计模式，它定义了一系列算法，将每一个算法封装起来，并使它们可以互相替换。策略模式让算法的变化独立于使用算法的客户。

核心思想是：“将算法的定义和使用分离”——客户端根据需求选择不同的策略对象，而无需关心策略的具体实现。

>生活中的类比：出行方式的选择。你要从家去机场，可以选择坐地铁、打车、骑共享单车、自驾。每种方式（策略）都能把你送到机场，但费用、时间、舒适度不同。你根据实际情况（预算、时间、天气）选择一种策略，而“去机场”这个行为本身不变。


一个电商系统的订单价格计算：不同用户等级（普通、VIP、超级VIP）享受不同的折扣，而且节假日可能有满减活动，还有优惠券叠加。

如果直接在订单类中用 if-else 实现所有折扣逻辑，代码会变得臃肿且难以维护。新增一种促销方式就要修改订单类，违反开闭原则。

策略模式将每种折扣算法封装成独立的策略类，订单类只需持有一个策略引用，运行时动态切换。

## 模式结构

```
┌──────────────────┐
│    Context       │    （上下文）
│ - strategy       │
│ + setStrategy(s) │
│ + executeStrategy│
└────────┬─────────┘
         │ 委托
         ▼
┌──────────────────┐
│    Strategy      │    （抽象策略）
│ + calculate(p)   │
└────────┬─────────┘
         ▲
         │ 实现
┌────────┴─────────┐    ┌──────────────────┐
│ ConcreteStrategyA│    │ ConcreteStrategyB│
│ + calculate(p)   │    │ + calculate(p)   │
└──────────────────┘    └──────────────────┘
```

核心角色：

- Context（上下文）：持有一个策略对象的引用，可以定义让客户端访问策略的接口。上下文将请求委托给当前策略对象执行。
- Strategy（抽象策略）：定义所有支持的算法的公共接口。
- ConcreteStrategy（具体策略）：实现策略接口，提供具体的算法实现。


##  业务案例：电商促销折扣计算

实现一个订单系统，支持多种折扣策略：无折扣、满减、百分比折扣、VIP 双倍积分折扣。客户端可以根据促销活动动态选择策略。

1. 抽象策略
```java
public interface DiscountStrategy {
    /**
     * 计算折后价格
     * @param originalPrice 原价
     * @return 折后价格
     */
    double applyDiscount(double originalPrice);
}
```

2. 具体策略
```java
// 无折扣
public class NoDiscountStrategy implements DiscountStrategy {
    @Override
    public double applyDiscount(double originalPrice) {
        return originalPrice;
    }
}

// 满减策略：满200减30
public class FullReductionStrategy implements DiscountStrategy {
    private double threshold = 200.0;
    private double reduction = 30.0;

    @Override
    public double applyDiscount(double originalPrice) {
        if (originalPrice >= threshold) {
            return originalPrice - reduction;
        }
        return originalPrice;
    }
}

// 百分比折扣：全场9折
public class PercentageDiscountStrategy implements DiscountStrategy {
    private double discountRate = 0.9;  // 9折

    public PercentageDiscountStrategy(double discountRate) {
        this.discountRate = discountRate;
    }

    @Override
    public double applyDiscount(double originalPrice) {
        return originalPrice * discountRate;
    }
}

// VIP会员折扣：8折 + 额外积分（积分不在价格中体现，此处简化）
public class VIPDiscountStrategy implements DiscountStrategy {
    @Override
    public double applyDiscount(double originalPrice) {
        return originalPrice * 0.8;
    }
}
```

3. 上下文（订单类）
```java
public class Order {
    private double totalAmount;
    private DiscountStrategy discountStrategy;

    public Order(double totalAmount) {
        this.totalAmount = totalAmount;
        // 默认无折扣
        this.discountStrategy = new NoDiscountStrategy();
    }

    public void setDiscountStrategy(DiscountStrategy strategy) {
        this.discountStrategy = strategy;
    }

    // 计算最终应付金额
    public double calculateFinalAmount() {
        return discountStrategy.applyDiscount(totalAmount);
    }

    // 打印订单信息
    public void printOrder() {
        System.out.println("订单原价：" + totalAmount);
        System.out.println("折扣策略：" + discountStrategy.getClass().getSimpleName());
        System.out.println("应付金额：" + calculateFinalAmount());
        System.out.println("---------------------");
    }
}
```

4. 客户端
```java
public class ShoppingCart {
    public static void main(String[] args) {
        // 创建一个订单，原价500元
        Order order = new Order(500.0);

        // 平时：无折扣
        System.out.println("【日常购买】");
        order.setDiscountStrategy(new NoDiscountStrategy());
        order.printOrder();

        // 国庆节：全场9折
        System.out.println("【国庆促销：全场9折】");
        order.setDiscountStrategy(new PercentageDiscountStrategy(0.9));
        order.printOrder();

        // 双十一：满200减30
        System.out.println("【双十一：满200减30】");
        order.setDiscountStrategy(new FullReductionStrategy());
        order.printOrder();

        // VIP会员：8折
        System.out.println("【VIP会员专享】");
        order.setDiscountStrategy(new VIPDiscountStrategy());
        order.printOrder();

        // 动态组合：VIP会员 + 满减（策略模式也支持组合，此处展示策略切换）
        System.out.println("【VIP会员叠加满减：先VIP折扣再满减】");
        // 使用组合策略（如果需要，可以定义CompositeStrategy）
        double afterVIP = new VIPDiscountStrategy().applyDiscount(500.0);
        Order tempOrder = new Order(afterVIP);
        tempOrder.setDiscountStrategy(new FullReductionStrategy());
        System.out.println("VIP折后价：" + afterVIP);
        System.out.println("再满减后：" + tempOrder.calculateFinalAmount());
    }
}
```
输出：
```
【日常购买】
订单原价：500.0
折扣策略：NoDiscountStrategy
应付金额：500.0
---------------------
【国庆促销：全场9折】
订单原价：500.0
折扣策略：PercentageDiscountStrategy
应付金额：450.0
---------------------
【双十一：满200减30】
订单原价：500.0
折扣策略：FullReductionStrategy
应付金额：470.0
---------------------
【VIP会员专享】
订单原价：500.0
折扣策略：VIPDiscountStrategy
应付金额：400.0
---------------------
【VIP会员叠加满减：先VIP折扣再满减】
VIP折后价：400.0
再满减后：370.0
```

核心价值：

- Order 类不需要知道任何折扣算法的细节，只需调用 discountStrategy.applyDiscount()。
- 新增一种促销（如“第二件半价”）只需新建一个策略类，无需修改订单类。
- 运行时可以动态切换策略，甚至组合策略（通过包装或组合模式）。


## 优缺点

优点：

- 符合开闭原则：新增策略无需修改上下文和其他策略。
- 消除条件分支：将复杂的 if-else 或 switch 替换为策略对象的委托。
- 提高复用性：策略类可以独立测试，并在不同上下文中复用。
- 支持运行时切换：客户端可以在运行时动态改变对象的行为。

缺点：

- 类数量增多：每个策略一个类，策略较多时类数量膨胀。
- 客户端必须了解策略：客户端需要知道有哪些策略可用，并选择合适的策略（增加了客户端的负担）。
- 策略之间可能互相依赖：某些策略可能需要其他策略的结果（如组合策略），需要额外设计。

## 适用场景

- 一个系统需要在多种算法中选择一种，且算法可以互相替换。
- 算法经常变化，或者未来可能增加新的算法。
- 算法包含大量条件逻辑，导致代码难以维护。
- 希望将算法的定义和使用分离，以便独立测试和复用。



## 策略模式 vs 状态模式

|对比项	|状态模式	|策略模式|
|-------|----------|---------|
|目的	|根据状态改变行为（状态转换）	|封装算法，使其可互换|
|状态管理	|状态对象通常会持有上下文的引用，并负责切换到下一个状态	|策略对象通常不持有上下文引用，也不负责切换策略|
|行为数量	|同一操作在不同状态下行为不同	|同一操作在不同策略下行为不同|
|典型例子	|售货机状态、订单状态	|排序算法、支付方式|

> 两者结构相似，但意图不同：状态模式强调状态转换，策略模式强调算法替换。

侧重点
- 工厂模式：创建对象，重点是“创建什么”。
- 策略模式：行为封装，重点是“怎么执行”。

> 两者常结合使用：工厂模式创建策略对象，策略模式执行算法。


## 现实应用

| 领域	 | 示例                                                  |
|------|-----------------------------------------------------|
| Java 集合排序 |Collections.sort(list, comparator) 中的 Comparator 就是策略模式，不同的比较策略决定排序结果。 |
| Spring ResourceLoader |根据资源路径前缀（classpath:/、file:/、http://）选择不同的资源加载策略。|
| 支付系统|不同支付方式（支付宝、微信、银联）封装为策略，客户端选择支付策略。|
| 压缩工具|ZIP、GZIP、7z 等不同压缩算法封装为策略。|
| 验证框架 |Hibernate Validator 中的校验器（如 @NotNull、@Size）可看作策略。 |
| 路由算法 |导航软件根据路况、距离、时间选择不同的路线规划策略。 |

## 总结：

策略模式的核心是“算法家族化，可替换”。它让你能够将算法的定义和使用解耦，使系统更容易扩展和维护。

在实际开发中，只要遇到“根据不同情况执行不同逻辑”且逻辑可能变化的场景，就可以考虑策略模式。

记住一句话：“条条大路通罗马，选哪条路你自己定，我只管走。”