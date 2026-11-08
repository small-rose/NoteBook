---
layout: default
title: Mediator Pattern
parent: Design Patterns
has_children: false
nav_order: 27
---

Here are java Mediator Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是中介者模式？

中介者模式（Mediator Pattern）

中介者模式是一种行为型设计模式，它用一个中介对象来封装一组对象之间的交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，并且可以独立地改变它们之间的交互。

核心思想是：“多对多”的交互简化为“多对一”的交互，所有同事对象只与中介者通信，由中介者协调它们的行为。

>  生活中的类比：机场塔台调度系统。飞机（同事对象）之间不直接通信，而是全部与塔台（中介者）联系。塔台负责协调飞机的起飞、降落、滑行路线，避免碰撞。如果没有塔台，每架飞机都要与其他所有飞机通信，极度复杂且危险。

考虑一个聊天室系统：多个用户之间可以互相发送消息。如果每个用户都持有对其他所有用户的引用，那么：

- 新增一个用户需要修改所有现有用户的代码（添加引用）。
- 用户之间的交互逻辑分散在各个用户类中，难以维护。
- 系统耦合度极高。

中介者模式引入一个聊天室（中介者），所有用户只与聊天室通信。 用户发送消息给聊天室，聊天室负责将消息广播给其他用户。这样用户之间不再直接耦合。


## 模式结构

```
┌──────────────────┐          ┌──────────────────────┐
│    Mediator      │          │     Colleague        │
│ + notify(sender, │◄────────▶│ + setMediator(m)     │
│   event)         │          │ + receive(msg)       │
└────────┬─────────┘          └──────────────────────┘
         ▲                               ▲
         │ 实现                           │ 继承
┌────────┴─────────┐          ┌──────────┴───────────┐
│ ConcreteMediator │          │ ConcreteColleague    │
│ + notify(...)    │          │ + send(msg)          │
│ - colleagues     │          │ + receive(msg)       │
└──────────────────┘          └──────────────────────┘
```

核心角色：
- Mediator（抽象中介者）：定义通信接口，用于各同事对象之间的通信。
- ConcreteMediator（具体中介者）：实现中介者接口，协调各同事对象的交互，维护对同事对象的引用。
- Colleague（抽象同事类）：所有同事类的基类，持有对中介者的引用。
- ConcreteColleague（具体同事类）：每个同事类只知道自己的行为，与其他同事的交互委托给中介者。


##  业务案例：智能家居控制中心

假设你有智能家居系统，包含灯、空调、窗帘、报警器等多个设备。当检测到有人入侵时，需要：报警器响、灯全亮、空调关闭、窗帘拉开。如果设备之间直接通信，每个设备都需要知道其他设备的接口，耦合严重。

我们用中介者模式实现一个家居控制中心。

1. 抽象同事类
```java
public abstract class SmartDevice {
    protected Mediator mediator;
    protected String name;

    public SmartDevice(String name, Mediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }

    public abstract void receiveSignal(String event);

    public String getName() {
        return name;
    }
}
```

2. 具体同事类
```java
// 灯
public class Light extends SmartDevice {
    private boolean isOn = false;

    public Light(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void turnOn() {
        isOn = true;
        System.out.println(name + " 灯已打开");
    }

    public void turnOff() {
        isOn = false;
        System.out.println(name + " 灯已关闭");
    }

    @Override
    public void receiveSignal(String event) {
        switch (event) {
            case "INTRUSION":
                turnOn();
                break;
            case "GOODBYE":
                turnOff();
                break;
            default:
                System.out.println(name + " 收到未知信号：" + event);
        }
    }
}

// 空调
public class AirConditioner extends SmartDevice {
    private boolean isOn = false;

    public AirConditioner(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void turnOn() {
        isOn = true;
        System.out.println(name + " 空调已开启");
    }

    public void turnOff() {
        isOn = false;
        System.out.println(name + " 空调已关闭");
    }

    @Override
    public void receiveSignal(String event) {
        switch (event) {
            case "INTRUSION":
                turnOff();
                break;
            case "ARRIVAL":
                turnOn();
                break;
            default:
                System.out.println(name + " 收到未知信号：" + event);
        }
    }
}

// 窗帘
public class Curtain extends SmartDevice {
    private boolean isOpen = false;

    public Curtain(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void open() {
        isOpen = true;
        System.out.println(name + " 窗帘已打开");
    }

    public void close() {
        isOpen = false;
        System.out.println(name + " 窗帘已关闭");
    }

    @Override
    public void receiveSignal(String event) {
        switch (event) {
            case "INTRUSION":
                open();
                break;
            case "GOODBYE":
                close();
                break;
            default:
                System.out.println(name + " 收到未知信号：" + event);
        }
    }
}

// 报警器
public class Alarm extends SmartDevice {
    public Alarm(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void sound() {
        System.out.println(name + " 报警器响起！呜哇呜哇～");
    }

    public void silence() {
        System.out.println(name + " 报警器静音");
    }

    @Override
    public void receiveSignal(String event) {
        switch (event) {
            case "INTRUSION":
                sound();
                break;
            case "ALARM_OFF":
                silence();
                break;
            default:
                System.out.println(name + " 收到未知信号：" + event);
        }
    }
}
```

3. 抽象中介者
```java
public interface Mediator {
    void registerDevice(SmartDevice device);
    void notify(SmartDevice sender, String event);
}
```

4. 具体中介者（家居控制中心）
```java
import java.util.ArrayList;
import java.util.List;

public class HomeControlCenter implements Mediator {
    private List<SmartDevice> devices = new ArrayList<>();

    @Override
    public void registerDevice(SmartDevice device) {
        devices.add(device);
    }

    @Override
    public void notify(SmartDevice sender, String event) {
        System.out.println("\n【控制中心】接收到来自 " + sender.getName() + " 的信号：" + event);
        // 根据事件类型通知所有相关设备
        for (SmartDevice device : devices) {
            if (device != sender) {  // 不通知发送者自己
                device.receiveSignal(event);
            }
        }
    }

    // 模拟触发事件
    public void triggerEvent(String event) {
        System.out.println("\n========== 触发事件：" + event + " ==========");
        for (SmartDevice device : devices) {
            device.receiveSignal(event);
        }
    }
}
```
5. 客户端
```java
public class SmartHomeClient {
    public static void main(String[] args) {
        // 创建中介者
        HomeControlCenter controlCenter = new HomeControlCenter();

        // 创建设备并注册到中介者
        Light livingRoomLight = new Light("客厅灯", controlCenter);
        Light bedroomLight = new Light("卧室灯", controlCenter);
        AirConditioner ac = new AirConditioner("客厅空调", controlCenter);
        Curtain curtain = new Curtain("客厅窗帘", controlCenter);
        Alarm alarm = new Alarm("大门报警器", controlCenter);

        controlCenter.registerDevice(livingRoomLight);
        controlCenter.registerDevice(bedroomLight);
        controlCenter.registerDevice(ac);
        controlCenter.registerDevice(curtain);
        controlCenter.registerDevice(alarm);

        // 模拟入侵事件（可由传感器触发）
        controlCenter.triggerEvent("INTRUSION");

        // 模拟主人到家
        System.out.println("\n\n========== 触发事件：ARRIVAL ==========");
        controlCenter.triggerEvent("ARRIVAL");

        // 模拟主人离开
        System.out.println("\n\n========== 触发事件：GOODBYE ==========");
        controlCenter.triggerEvent("GOODBYE");
    }
}
```
输出：
```
========== 触发事件：INTRUSION ==========
客厅灯 灯已打开
卧室灯 灯已打开
客厅空调 空调已关闭
客厅窗帘 窗帘已打开
大门报警器 报警器响起！呜哇呜哇～

========== 触发事件：ARRIVAL ==========
客厅灯 收到未知信号：ARRIVAL
卧室灯 收到未知信号：ARRIVAL
客厅空调 空调已开启
客厅窗帘 收到未知信号：ARRIVAL
大门报警器 收到未知信号：ARRIVAL

========== 触发事件：GOODBYE ==========
客厅灯 灯已关闭
卧室灯 灯已关闭
客厅空调 收到未知信号：GOODBYE
客厅窗帘 窗帘已关闭
大门报警器 收到未知信号：GOODBYE
```

核心价值：

- 所有设备只与 HomeControlCenter 交互，彼此不知道对方的存在。
- 新增设备（如加湿器）只需实现 SmartDevice 并注册到控制中心，无需修改其他设备。
- 改变事件处理逻辑只需修改中介者或设备自身的 receiveSignal 方法。


## 优缺点

优点：

- 降低耦合：同事类之间不再直接引用，改为与中介者通信。
- 简化对象交互：将多对多的交互简化为一对多的交互，更易理解和管理。
- 集中控制：交互逻辑集中在中介者中，便于统一修改和维护。
- 易于扩展：新增同事类只需注册到中介者，无需修改现有同事类。

缺点：

- 中介者可能变得臃肿：如果交互逻辑复杂，中介者会承担过多职责，变成“上帝对象”。
- 增加系统复杂度：引入中介者增加了额外的抽象层，对于简单交互可能过度设计。
- 中介者故障影响全局：中介者是通信枢纽，一旦出现问题，整个系统可能瘫痪。

## 适用场景

- 一组对象之间通信方式复杂，导致相互依赖、结构混乱。
- 一个对象引用了很多其他对象，导致难以复用。
- 想要定制分布在多个类之间的行为，而又不想生成太多子类。
- 典型的应用：聊天室、消息中间件、GUI 控件协调、航空管制。



## 中介者模式 vs 观察者模式

| 对比项	  | 中介者模式 | 观察者模式      |
|-------|-----------------|------------|
| 通信方向	 | 多对一（同事→中介者→同事）| 一对多（主题→观察者） |
| 目的	   |减少同事之间的耦合   | 解耦主题和观察者     |
| 角色	   | 中介者集中控制交互 |观察者被动接收通知   |
| 典型例子  |聊天室、塔台调度 | 事件监听、发布订阅  |

## 现实应用

|领域	| 示例                                                          |
|-----|-------------------------------------------------------------|
|聊天室 |用户通过聊天室服务器（中介者）收发消息，用户之间不直接通信。    |
|消息队列/中间件 |Kafka、RabbitMQ 作为中介者，生产者和消费者通过它异步通信。 |
|GUI 框架 |Java Swing 中的 ActionListener 可以看作中介者，协调按钮和文本框等组件。|
|MVC 架构|Controller 充当 Model 和 View 的中介者，协调数据更新和界面刷新。|
|工作流引擎 |Activiti、Flowable 作为中介者协调任务、网关、事件之间的流转。 |
|微服务编排 |服务编排层（如 Kubernetes 的 API Server）作为中介者协调各微服务。 |

## 总结：

中介者模式的核心是“集中管控，分散执行”——将复杂的网状交互简化为星型交互，所有通信经由中介者转发。

它特别适合需要协调多个对象的场景，能显著降低耦合度。但要注意避免中介者过于臃肿。

记住一句话：“不要让大家互相喊话，找一个话务员，所有人只对他说话。”