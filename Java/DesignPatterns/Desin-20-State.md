---
layout: default
title: State Pattern
parent: Design Patterns
has_children: false
nav_order: 30
---

Here are java State Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是状态模式？

状态模式（State Pattern）

状态模式是一种行为型设计模式，它允许一个对象在其内部状态改变时改变它的行为，看起来就像修改了它的类。

核心思想是：将状态相关的行为封装到独立的状态类中，并将对象的行为委托给当前状态对象，从而避免大量的 if-else 或 switch-case 分支判断。

>生活中的类比：电梯。电梯有多种状态：停止、运行、开门、关门、超载、维修。在不同状态下，同样的操作（比如按楼层按钮）会产生不同的行为——停止时可以开门，运行时不能开门。如果将所有这些行为逻辑写在电梯类中，会有大量条件判断；状态模式将每种状态封装成一个类，电梯类只需持有当前状态对象并委托给它。


考虑一个自动售货机：它有多种状态（待付款、已付款、出货中、售罄、故障），每种状态下对用户操作（投币、选择商品、退款）的反应不同。

如果用 if-else 实现，每增加一种状态或操作，都需要修改核心类，代码迅速膨胀且难以维护。

状态模式将每个状态封装成独立类，售货机类只需维护一个当前状态引用，所有行为委托给状态对象处理。新增状态只需新增一个类，无需修改现有代码。

## 模式结构

```
┌──────────────────┐
│    Context       │    （上下文/环境）
│ - state: State   │
│ + request()      │
│ + setState(s)    │
└────────┬─────────┘
         │ 委托
         ▼
┌──────────────────┐
│      State       │    （抽象状态）
│ + handle(context)│
└────────┬─────────┘
         ▲
         │ 实现
┌────────┴─────────┐    ┌──────────────────┐
│ ConcreteStateA   │    │ ConcreteStateB   │
│ + handle(context)│    │ + handle(context)│
└──────────────────┘    └──────────────────┘
```

核心角色：

- Context（上下文）：持有当前状态对象的引用，并将请求委托给状态对象处理。上下文可以向客户端暴露切换状态的方法。
- State（抽象状态）：定义所有具体状态的公共接口，封装与上下文相关的一个行为。
- ConcreteState（具体状态）：实现状态接口，定义在该状态下上下文应表现的行为。每个具体状态通常还会负责切换到下一个合适的状态。


##  业务案例：自动售货机

实现一个简单的自动售货机，有四种状态：待付款（NoCoin）、已投币（HasCoin）、出售（Sold）、售罄（SoldOut）。用户操作：投币、退币、转动曲柄（购买）。

1. 观察者接口
```java
public interface State {
    void insertCoin();       // 投币
    void ejectCoin();        // 退币
    void turnCrank();        // 转动曲柄
    void dispense();         // 出货
}
```

2. 上下文（自动售货机）
```java
public class VendingMachine {
    private State noCoinState;
    private State hasCoinState;
    private State soldState;
    private State soldOutState;

    private State currentState;
    private int count;  // 剩余商品数量

    public VendingMachine(int count) {
        noCoinState = new NoCoinState(this);
        hasCoinState = new HasCoinState(this);
        soldState = new SoldState(this);
        soldOutState = new SoldOutState(this);

        this.count = count;
        if (count > 0) {
            currentState = noCoinState;
        } else {
            currentState = soldOutState;
        }
    }

    public void setState(State state) {
        this.currentState = state;
    }

    public State getNoCoinState() { return noCoinState; }
    public State getHasCoinState() { return hasCoinState; }
    public State getSoldState() { return soldState; }
    public State getSoldOutState() { return soldOutState; }

    public int getCount() { return count; }

    public void releaseProduct() {
        if (count > 0) {
            count--;
            System.out.println("出货成功，剩余 " + count + " 件商品");
        }
    }

    // 对外暴露的用户操作
    public void insertCoin() {
        currentState.insertCoin();
    }

    public void ejectCoin() {
        currentState.ejectCoin();
    }

    public void turnCrank() {
        currentState.turnCrank();
        currentState.dispense();
    }
}
```

3. 具体状态
```java
// 待付款状态（无硬币）
public class NoCoinState implements State {
    private VendingMachine machine;

    public NoCoinState(VendingMachine machine) {
        this.machine = machine;
    }

    @Override
    public void insertCoin() {
        System.out.println("投币成功");
        machine.setState(machine.getHasCoinState());
    }

    @Override
    public void ejectCoin() {
        System.out.println("没有硬币可退还");
    }

    @Override
    public void turnCrank() {
        System.out.println("请先投币");
    }

    @Override
    public void dispense() {
        System.out.println("请先投币");
    }
}

// 已投币状态
public class HasCoinState implements State {
    private VendingMachine machine;

    public HasCoinState(VendingMachine machine) {
        this.machine = machine;
    }

    @Override
    public void insertCoin() {
        System.out.println("已投币，请不要重复投币");
    }

    @Override
    public void ejectCoin() {
        System.out.println("退还硬币");
        machine.setState(machine.getNoCoinState());
    }

    @Override
    public void turnCrank() {
        System.out.println("转动曲柄，准备出货");
        machine.setState(machine.getSoldState());
    }

    @Override
    public void dispense() {
        System.out.println("请先转动曲柄");
    }
}

// 出售状态（正在出货）
public class SoldState implements State {
    private VendingMachine machine;

    public SoldState(VendingMachine machine) {
        this.machine = machine;
    }

    @Override
    public void insertCoin() {
        System.out.println("正在出货，请稍后再投币");
    }

    @Override
    public void ejectCoin() {
        System.out.println("正在出货，无法退币");
    }

    @Override
    public void turnCrank() {
        System.out.println("正在出货，请勿重复转动曲柄");
    }

    @Override
    public void dispense() {
        machine.releaseProduct();
        if (machine.getCount() > 0) {
            machine.setState(machine.getNoCoinState());
        } else {
            System.out.println("商品已售罄");
            machine.setState(machine.getSoldOutState());
        }
    }
}

// 售罄状态
public class SoldOutState implements State {
    private VendingMachine machine;

    public SoldOutState(VendingMachine machine) {
        this.machine = machine;
    }

    @Override
    public void insertCoin() {
        System.out.println("商品已售罄，无法购买");
    }

    @Override
    public void ejectCoin() {
        System.out.println("没有硬币可退还");
    }

    @Override
    public void turnCrank() {
        System.out.println("商品已售罄");
    }

    @Override
    public void dispense() {
        System.out.println("商品已售罄");
    }
}
```

4. 客户端
```java
public class VendingMachineDemo {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine(2);

        System.out.println("=== 正常购买流程 ===");
        machine.insertCoin();
        machine.turnCrank();

        System.out.println("\n=== 再次购买 ===");
        machine.insertCoin();
        machine.turnCrank();

        System.out.println("\n=== 售罄后尝试投币 ===");
        machine.insertCoin();
        machine.turnCrank();
    }
}
```
输出：
```
=== 正常购买流程 ===
投币成功
转动曲柄，准备出货
出货成功，剩余 1 件商品

=== 再次购买 ===
投币成功
转动曲柄，准备出货
出货成功，剩余 0 件商品
商品已售罄

=== 售罄后尝试投币 ===
商品已售罄，无法购买
商品已售罄
```

核心价值：

- 每个状态类只关注自己在当前状态下的行为逻辑，职责单一。
- 新增状态（如“故障状态”）只需新增一个类并实现接口，无需修改现有状态类。
- 状态转换逻辑分散在各个状态类中（如 SoldState 出货后根据库存决定转到 NoCoinState 还是 SoldOutState），避免了集中的 if-else。


## 优缺点

优点：

- 消除庞大的条件分支：将状态判断逻辑分散到各个状态类中，避免 if-else 或 switch-case 泛滥。
- 符合开闭原则：新增状态只需新增类，无需修改上下文或其他状态。
- 提高内聚性：与特定状态相关的行为都集中在同一个类中。
- 状态转换显式化：每个状态类明确知道自己可以转换到哪些状态，转换逻辑清晰。

缺点：

- 类数量增多：每个状态都需要一个类，状态较多时类数量膨胀。
- 增加系统复杂度：需要额外维护状态类之间的转换关系。
- 上下文与状态类耦合：上下文需要知道所有状态类的存在（通常通过构造方法传入或工厂创建）。

## 适用场景

- 对象的行为依赖于其状态，并且状态改变时行为必须随之改变。
- 操作中包含大量与状态相关的条件分支（if-else 或 switch），且分支数量可能增长。
- 状态转换规则复杂，且转换逻辑需要显式管理。
- 典型的应用：工作流引擎、游戏角色状态（行走/攻击/受伤/死亡）、TCP 连接状态、订单状态流转。


## 状态模式 vs 策略模式

|对比项	|状态模式	|策略模式|
|-------|----------|---------|
|目的	|根据状态改变行为（状态转换）	|封装算法，使其可互换|
|状态管理	|状态对象通常会持有上下文的引用，并负责切换到下一个状态	|策略对象通常不持有上下文引用，也不负责切换策略|
|行为数量	|同一操作在不同状态下行为不同	|同一操作在不同策略下行为不同|
|典型例子	|售货机状态、订单状态	|排序算法、支付方式|

> 两者结构相似，但意图不同：状态模式强调状态转换，策略模式强调算法替换。

## 实现要点

- 状态对象可以是单例：如果状态类没有自己的实例变量，可以设计为单例以减少对象创建开销。
- 状态转换可以由上下文触发，也可以由状态对象触发：通常由状态对象在 handle() 方法中调用 context.setState() 来完成转换。
- 避免状态爆炸：如果状态数量过多（几十个），可以考虑使用状态表（二维矩阵）代替状态类，但会牺牲一定的可读性。

## 现实应用

| 领域	 | 示例                                                   |
|--------|------------------------------------------------------|
| Java 线程状态 |线程有 NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED 六种状态，不同状态下 start()、sleep()、wait() 等操作行为不同。 |
| 游戏开发 |游戏角色的状态（站立、跑步、跳跃、攻击、受伤、死亡），每个状态对应不同的动画和物理行为。|
| 网络协议|TCP 连接的状态机（CLOSED、LISTEN、SYN_SENT、ESTABLISHED 等），不同状态下对数据包的处理不同。|
| 订单系统|订单状态（待支付、已支付、已发货、已收货、已取消、已退款），每个状态下可执行的操作不同。|
| Spring Web Flow  |Spring 的状态机框架用于定义业务流程的状态转换。 |
| 电梯控制系统   |电梯的状态（停止、运行、开门、关门、超载、检修），不同状态下对按钮的响应不同。 |

## 总结：

状态模式的核心是“将状态变成对象”，让对象的行为随着内部状态的改变而改变。

它非常适合处理具有复杂状态转换的业务逻辑，能够显著提升代码的可维护性和扩展性。

记住一句话：“对象像变色龙，状态一变，行为就变。”