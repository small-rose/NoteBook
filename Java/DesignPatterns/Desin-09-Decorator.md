---
layout: default
title: Decorator Pattern
parent: Design Patterns
has_children: false
nav_order: 19
---

Here are java Decorator Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是装饰器模式？

装饰器模式（Decorator Pattern）

装饰器模式是一种结构型设计模式，它允许动态地给一个对象添加额外的职责，同时不改变其结构。相比于继承，装饰器提供了更灵活的扩展方式——你可以像叠包装一样，一层层地给核心对象添加新功能。

> 生活中的类比：买一杯原味奶茶（核心对象），你可以选择加珍珠、加椰果、加布丁……每加一种配料就是一次装饰。最终你得到的是被多层装饰包裹的奶茶，但它的本质仍然是奶茶（同一接口）。

考虑一个咖啡店的点单系统：基础饮品有浓缩咖啡、美式咖啡、拿铁等，顾客可以选择加牛奶、加糖、加奶油、加巧克力酱等配料。

如果使用继承来实现，每种组合都需要一个子类，会导致类爆炸（2种饮品 × 3种配料 = 8个类，随着配料增加呈指数增长）。

装饰器模式通过组合的方式，将配料作为装饰器层层包裹核心饮品，每个装饰器只负责添加自己的功能，然后调用下一个组件。

## 模式结构

```
┌──────────────────┐
│   Component      │    （抽象组件）
│ + cost()         │
│ + description()  │
└────────┬─────────┘
         ▲
         │ 实现
┌────────┴─────────┐    ┌──────────────────────┐
│ ConcreteComponent│    │      Decorator       │   （抽象装饰器）
│ + cost()         │    │ - component          │
│ + description()  │    │ + cost()             │
└──────────────────┘    │ + description()      │
                        └──────────┬───────────┘
                                   ▲
                                   │ 继承
                        ┌──────────┴───────────┐
                        │ ConcreteDecoratorA   │
                        │ + cost()             │
                        │ + description()      │
                        └──────────────────────┘
```

核心角色：

- Component（抽象组件）：定义对象接口，可以给这些对象动态添加职责。
- ConcreteComponent（具体组件）：定义了一个具体的对象，可以给它添加职责。
- Decorator（抽象装饰器）：持有一个 Component 引用，并实现与 Component 一致的接口。
- ConcreteDecorator（具体装饰器）：负责给组件添加额外的功能。


  
## 案例1 咖啡点单系统

业务场景：咖啡点单系统

1. 抽象组件
```java
public abstract class Beverage {
    protected String description = "Unknown Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}
```

2. 具体组件（核心饮品）
```java
public class Espresso extends Beverage {
    public Espresso() {
        description = "浓缩咖啡";
    }

    @Override
    public double cost() {
        return 25.0;
    }
}

public class Americano extends Beverage {
    public Americano() {
        description = "美式咖啡";
    }

    @Override
    public double cost() {
        return 20.0;
    }
}

public class Latte extends Beverage {
    public Latte() {
        description = "拿铁咖啡";
    }

    @Override
    public double cost() {
        return 28.0;
    }
}
```

3. 抽象装饰器
```java
public abstract class CondimentDecorator extends Beverage {
    protected Beverage beverage;

    public CondimentDecorator(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public abstract String getDescription();
}
```
4. 具体装饰器（配料）
```java
// 加牛奶
public class Milk extends CondimentDecorator {
    public Milk(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + " + 牛奶";
    }

    @Override
    public double cost() {
        return beverage.cost() + 5.0;
    }
}

// 加糖
public class Sugar extends CondimentDecorator {
    public Sugar(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + " + 糖";
    }

    @Override
    public double cost() {
        return beverage.cost() + 2.0;
    }
}

// 加奶油
public class Whip extends CondimentDecorator {
    public Whip(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + " + 奶油";
    }

    @Override
    public double cost() {
        return beverage.cost() + 6.0;
    }
}

// 加巧克力酱
public class Chocolate extends CondimentDecorator {
    public Chocolate(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + " + 巧克力酱";
    }

    @Override
    public double cost() {
        return beverage.cost() + 8.0;
    }
}
```
5. 客户端使用
```java
public class CoffeeShop {
    public static void main(String[] args) {
        // 一杯浓缩咖啡
        Beverage espresso = new Espresso();
        System.out.println(espresso.getDescription() + " ￥" + espresso.cost());

        // 一杯美式咖啡加牛奶和糖
        Beverage americano = new Americano();
        americano = new Milk(americano);
        americano = new Sugar(americano);
        System.out.println(americano.getDescription() + " ￥" + americano.cost());

        // 一杯拿铁加奶油和巧克力酱
        Beverage latte = new Latte();
        latte = new Whip(latte);
        latte = new Chocolate(latte);
        System.out.println(latte.getDescription() + " ￥" + latte.cost());
    }
}
```

输出：
```text
浓缩咖啡 ￥25.0
美式咖啡 + 牛奶 + 糖 ￥27.0
拿铁咖啡 + 奶油 + 巧克力酱 ￥42.0
```

核心价值：

- 每种配料是一个独立的装饰器类，可以自由组合。
- 新增一种配料只需新增一个装饰器类，无需修改现有代码。
- 客户端可以像堆叠积木一样任意组合，且结果仍然是一个 Beverage 对象。


 
##  优缺点

优点：
- 比继承更灵活：继承是静态的（编译期决定），装饰器是动态的（运行期组合）。
- 避免类爆炸：M 种核心组件 × N 种装饰器，最多只需 M+N 个类，而不是 M×N 个。
- 符合开闭原则：无需修改现有代码即可扩展功能。
- 单一职责：每个装饰器只负责一项功能，职责清晰。

缺点：
- 增加系统复杂度：产生许多小类，可能使系统难以理解。
- 多层装饰调试困难：嵌套调用栈较深时，排查问题较为麻烦。
- 可能导致对象类型混淆：如果客户端依赖具体类型，装饰后可能无法直接识别。


## 适用场景

- 需要动态地、透明地给单个对象添加职责，且这些职责可以撤销。
- 不能或不希望通过继承扩展功能（例如类被 final 修饰，或继承会导致子类数量过多）。
- 需要给对象叠加多个功能，且功能的组合方式多变。
- 经典例子：Java I/O 流中的 FilterInputStream 及其子类。



## 比较 装饰器模式 vs 代理模式

| 对比项	 | 	装饰器模式	| 代理模式              |
|----|---------------|--------------------|
| 目的 | 动态添加新功能	| 控制对目标对象的访问 |
| 关系 | 装饰器与组件是“is-a”关系（实现相同接口）	| 代理与真实对象是“has-a”关系（持有引用）  |
| 使用者视角 | 客户端看到的是增强后的组件	|客户端看到的是代理对象，可能不知道真实对象存在 |
| 典型例子 |Java I/O 流	|Spring AOP 代理、远程代理|                   

一句话区分：装饰器增强功能，代理控制访问。

## 实际应用

| 领域   | 示例                                                 |
|------|----------------------------------------------------------------------------------------------------|
| Java I/O | jBufferedInputStream 装饰 FileInputStream，DataInputStream 装饰 BufferedInputStream，层层嵌套提供缓冲、数据类型读取等功能。 |
| Spring 框架| TransactionAwareCacheDecorator 装饰缓存操作，添加事务感知功能。      |
| Web 过滤器 | Servlet 中的 Filter 链本质上是一种装饰器模式，每个 Filter 可以装饰请求/响应对象。    |
| 日志框架 | Logback 的 Appender 可以用 AsyncAppender 装饰，添加异步写入功能。      |
| Python 装饰器 | Python 语言原生支持装饰器语法（@decorator），常用于添加日志、计时、权限校验。   |


## 总结：

装饰器模式的核心是用组合替代继承，动态叠加功能。它让你像搭积木一样给核心对象添加各种能力，而且每一块积木（装饰器）都是独立可复用的。

在 Java I/O 流中，你每天使用的 new BufferedReader(new InputStreamReader(System.in)) 就是最经典的装饰器应用。

记住一句话：“继承是静态的加法，装饰是动态的叠加。”