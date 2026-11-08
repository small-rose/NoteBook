---
layout: default
title: Command Pattern
parent: Design Patterns
has_children: false
nav_order: 24
---

Here are java Command Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是代理模式？

命令模式（Command Pattern）

命令模式是一种行为型设计模式，它将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化、对请求排队或记录请求日志，以及支持可撤销的操作。

核心思想是：将“发出请求的对象”与“执行请求的对象”解耦，请求本身变成一个独立的对象，包含了执行所需的所有信息。

> 生活中的类比：餐厅点餐。顾客（客户端）把订单（命令对象）交给服务员（调用者），服务员把订单传给厨师（接收者）。顾客不需要知道厨师是谁、怎么做菜，订单本身携带了所有信息（菜品、桌号、备注）。

考虑一个在线客服系统：用户提交一个工单，系统需要根据工单类型（技术问题、账单问题、投诉建议）分配给不同的处理人员。

如果使用硬编码的 if-else 或 switch-case，当处理规则变化时（例如新增一个处理环节），需要修改核心代码，违反开闭原则。

责任链模式将每个处理者封装成一个独立的节点，通过链式结构动态组合，可以灵活地增删处理者或调整顺序。

## 模式结构

```
┌──────────────────┐
│    Command       │    （抽象命令）
│ + execute()      │
│ + undo()         │
└────────┬─────────┘
         ▲
         │ 实现
┌────────┴─────────┐
│ ConcreteCommand  │    （具体命令）
│ - receiver       │
│ + execute()      │
│ + undo()         │
└──────────────────┘
         │ 使用
┌────────▼─────────┐
│    Receiver      │    （接收者：实际执行者）
│ + action()       │
└──────────────────┘
         ▲
         │ 调用
┌────────┴─────────┐
│    Invoker       │    （调用者/触发者）
│ + setCommand()   │
│ + pressButton()  │
└──────────────────┘
```

核心角色：

- Command（抽象命令）：声明执行操作的接口。
- ConcreteCommand（具体命令）：绑定一个接收者对象，调用接收者的相应操作来实现 execute()。
- Receiver（接收者）：知道如何实施和执行一个请求相关的操作。
- Invoker（调用者）：要求命令对象执行请求，通常持有命令对象并在适当时候调用 execute()。
- Client（客户端）：创建具体命令对象并设置其接收者。

  
##  业务案例：智能家居遥控器

业务案例：智能家居遥控器

假设你有一个智能家居系统，遥控器上有多个按钮，每个按钮可以控制不同的设备（灯、空调、电视），并且支持撤销操作和宏命令（一键离家模式：关灯、关空调、关电视）。

1.抽象命令
```java
public interface Command {
    void execute();
    void undo();
}
```

2. 接收者（设备）
```java
// 灯
public class Light {
    private String location;

    public Light(String location) {
        this.location = location;
    }

    public void on() {
        System.out.println(location + " 灯已打开");
    }

    public void off() {
        System.out.println(location + " 灯已关闭");
    }
}

// 空调
public class AirConditioner {
    private String location;

    public AirConditioner(String location) {
        this.location = location;
    }

    public void turnOn() {
        System.out.println(location + " 空调已开启，温度设为26℃");
    }

    public void turnOff() {
        System.out.println(location + " 空调已关闭");
    }
}

// 电视
public class TV {
    private String location;

    public TV(String location) {
        this.location = location;
    }

    public void turnOn() {
        System.out.println(location + " 电视已打开");
    }

    public void turnOff() {
        System.out.println(location + " 电视已关闭");
    }
}
```

3. 具体命令
```java
// 开灯命令
public class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }

    @Override
    public void undo() {
        light.off();
    }
}

// 关灯命令
public class LightOffCommand implements Command {
    private Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.off();
    }

    @Override
    public void undo() {
        light.on();
    }
}

// 开空调命令
public class ACOnCommand implements Command {
    private AirConditioner ac;

    public ACOnCommand(AirConditioner ac) {
        this.ac = ac;
    }

    @Override
    public void execute() {
        ac.turnOn();
    }

    @Override
    public void undo() {
        ac.turnOff();
    }
}

// 关空调命令
public class ACOffCommand implements Command {
    private AirConditioner ac;

    public ACOffCommand(AirConditioner ac) {
        this.ac = ac;
    }

    @Override
    public void execute() {
        ac.turnOff();
    }

    @Override
    public void undo() {
        ac.turnOn();
    }
}
```

```java
// 宏命令：一键离家模式（关灯、关空调、关电视）
import java.util.ArrayList;
import java.util.List;

public class MacroCommand implements Command {
    private List<Command> commands;

    public MacroCommand(List<Command> commands) {
        this.commands = commands;
    }

    @Override
    public void execute() {
        for (Command cmd : commands) {
            cmd.execute();
        }
    }

    @Override
    public void undo() {
        // 逆序撤销
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
}
```

4. 调用者（遥控器）
```java
public class RemoteControl {
    private Command[] onCommands;
    private Command[] offCommands;
    private Command lastCommand;  // 用于撤销

    private static final int SLOT_COUNT = 3;

    public RemoteControl() {
        onCommands = new Command[SLOT_COUNT];
        offCommands = new Command[SLOT_COUNT];

        // 初始化空命令（避免null判断）
        NoCommand noCommand = new NoCommand();
        for (int i = 0; i < SLOT_COUNT; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        lastCommand = noCommand;
    }

    // 设置某个槽位的开/关命令
    public void setCommand(int slot, Command onCmd, Command offCmd) {
        onCommands[slot] = onCmd;
        offCommands[slot] = offCmd;
    }

    // 按下开按钮
    public void pressOnButton(int slot) {
        onCommands[slot].execute();
        lastCommand = onCommands[slot];
    }

    // 按下关按钮
    public void pressOffButton(int slot) {
        offCommands[slot].execute();
        lastCommand = offCommands[slot];
    }

    // 撤销按钮
    public void pressUndoButton() {
        System.out.print("【撤销】");
        lastCommand.undo();
    }
}

// 空命令（Null Object模式）
public class NoCommand implements Command {
    @Override
    public void execute() {}
    @Override
    public void undo() {}
}
```
5. 客户端
```java
public class SmartHomeClient {
    public static void main(String[] args) {
        // 创建设备
        Light livingRoomLight = new Light("客厅");
        Light kitchenLight = new Light("厨房");
        AirConditioner bedroomAC = new AirConditioner("卧室");
        TV tv = new TV("客厅");

        // 创建命令
        Command livingRoomLightOn = new LightOnCommand(livingRoomLight);
        Command livingRoomLightOff = new LightOffCommand(livingRoomLight);
        Command kitchenLightOn = new LightOnCommand(kitchenLight);
        Command kitchenLightOff = new LightOffCommand(kitchenLight);
        Command acOn = new ACOnCommand(bedroomAC);
        Command acOff = new ACOffCommand(bedroomAC);

        // 创建遥控器并分配按钮
        RemoteControl remote = new RemoteControl();
        remote.setCommand(0, livingRoomLightOn, livingRoomLightOff);
        remote.setCommand(1, kitchenLightOn, kitchenLightOff);
        remote.setCommand(2, acOn, acOff);

        // 使用遥控器
        System.out.println("=== 正常操作 ===");
        remote.pressOnButton(0);   // 打开客厅灯
        remote.pressOnButton(2);   // 打开卧室空调
        remote.pressOffButton(1);  // 关闭厨房灯

        System.out.println("\n=== 撤销上次操作 ===");
        remote.pressUndoButton();  // 撤销关厨房灯 -> 打开厨房灯

        System.out.println("\n=== 一键离家模式（宏命令） ===");
        List<Command> leaveHomeCommands = new ArrayList<>();
        leaveHomeCommands.add(new LightOffCommand(livingRoomLight));
        leaveHomeCommands.add(new LightOffCommand(kitchenLight));
        leaveHomeCommands.add(new ACOffCommand(bedroomAC));
        // 假设电视也有命令...
        Command leaveHomeMacro = new MacroCommand(leaveHomeCommands);
        leaveHomeMacro.execute();

        System.out.println("\n=== 撤销离家模式 ===");
        leaveHomeMacro.undo();
    }
}
```
输出：
```
=== 正常操作 ===
客厅 灯已打开
卧室 空调已开启，温度设为26℃
厨房 灯已关闭

=== 撤销上次操作 ===
【撤销】厨房 灯已打开

=== 一键离家模式（宏命令） ===
客厅 灯已关闭
厨房 灯已关闭
卧室 空调已关闭

=== 撤销离家模式 ===
卧室 空调已开启，温度设为26℃
厨房 灯已打开
客厅 灯已打开
```

核心价值：

- 解耦：遥控器（Invoker）不知道灯、空调的具体实现，只知道 Command 接口。
- 可撤销：通过保存最后执行的命令，轻松实现撤销。
- 宏命令：将多个命令组合成一个复合命令，实现一键场景。
- 可扩展：新增设备（如窗帘、音响）只需添加对应的命令类，无需修改遥控器。

## 优缺点

优点：

- 解耦请求发送者和接收者：调用者无需知道接收者的具体类型。
- 支持撤销/重做：命令对象可以存储状态，方便实现撤销操作。
- 支持命令队列和日志：可以将命令对象持久化或排队，实现延迟执行或事务日志。
- 易于扩展：新增命令无需修改现有代码，符合开闭原则。
- 支持组合命令：通过宏命令可以将多个命令组合成一个复合命令。

缺点：

- 类数量增多：每个具体操作都需要一个命令类，可能导致系统类膨胀。
- 增加系统复杂度：引入了额外的抽象层，对于简单场景可能过度设计。

## 适用场景

- 需要将请求参数化，以便在不同时间、不同上下文中执行。
- 需要支持撤销/重做操作（如文本编辑器、绘图软件）。
- 需要支持事务或日志：将命令记录到日志中，崩溃后可恢复。
- 需要实现任务队列或线程池：将命令对象放入队列，由工作线程执行。
- 需要支持宏命令：将一系列操作组合成一个原子操作。

## 命令模式 vs 策略模式

|对比项	|命令模式	|策略模式|
|-------|----------|----------|
|目的	|将请求封装为对象，支持排队、撤销、日志	|封装算法，使它们可以互相替换|
|关注点	|执行一个操作（可能带接收者）|算法的选择和替换|
|状态	|通常包含接收者，可能保存执行状态	|通常是无状态的算法|
|典型例子	|遥控器按钮、撤销操作	|排序算法、支付策略 |


## 现实应用

|领域	| 示例                                                                 |
|-----|--------------------------------------------------------------------|
|Spring 框架 |JdbcTemplate 的回调接口（如 PreparedStatementCallback）可看作命令模式。      |
|文本编辑器 | 每个编辑操作（输入、删除、粘贴）都是一个命令，支持 Ctrl+Z 撤销。 |
|游戏开发 | 玩家操作（移动、攻击）封装为命令，支持回放、录制、网络同步。 |
|分布式系统 | 命令模式用于实现远程调用（RPC）中的请求封装。 |
|消息队列 |消息本身就是一个命令对象，消费者从队列中取出并执行。   |
|Java Swing/AWT |Action 接口和 AbstractAction 类，将按钮点击等事件封装为命令。 |

## 总结：

命令模式的核心是“把请求变成对象”，从而获得面向对象的所有好处：参数化、排队、记录、撤销。

它在需要解耦调用者和执行者、支持撤销/重做、实现宏命令的场景中非常强大。

记住一句话：“将请求封装成命令对象，让调用者只管发令，执行者只管干活，中间的命令对象掌控一切。”