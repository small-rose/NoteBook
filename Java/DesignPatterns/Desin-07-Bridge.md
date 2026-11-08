---
layout: default
title: Bridge Pattern
parent: Design Patterns
has_children: false
nav_order: 17
---

Here are java Bridge Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是桥接模式？

桥接模式（Bridge Pattern）

桥接模式是一种结构型设计模式，它将抽象部分与实现部分分离，使它们都可以独立地变化。

核心思想是：用组合关系代替继承关系，从而避免在多个维度上扩展时导致的“类爆炸”。

考虑一个消息通知系统：消息可以分为普通消息、紧急消息、定时消息；发送渠道可以是邮件、短信、App推送。

如果用继承来实现：

```
           Message
      /         |       \
NormalMsg  UrgentMsg  TimerMsg
  /    \     /    \     /    \
Email  SMS  Email  SMS  Email  SMS
```

当消息类型有 M 种，发送渠道有 N 种时，需要 M × N 个子类。如果再增加一种消息类型或渠道，类数量会迅速膨胀。

桥接模式将这两个维度拆分成两条独立的继承链，通过组合将它们连接起来，从而大大减少类的数量。

## 模式结构

```
┌──────────────────────┐          ┌──────────────────────┐
│     Abstraction      │   关联    │    Implementor       │
│ (抽象化角色)           │─────────▶│ (实现化角色)           │
│ + operation()        │          │ + operationImpl()    │
└─────────┬────────────┘          └──────────┬───────────┘
          ▲                                   ▲
          │ 继承                              │ 继承
┌─────────┴────────────┐          ┌──────────┴───────────┐
│ RefinedAbstraction   │          │ ConcreteImplementor  │
│ (扩充抽象化)           │          │ (具体实现化)           │
│ + operation()        │          │ + operationImpl()    │
└──────────────────────┘          └──────────────────────┘
```

核心角色：

- Abstraction（抽象化）：定义抽象类的接口，维护一个对 Implementor 的引用。
- RefinedAbstraction（扩充抽象化）：扩展 Abstraction，改变或完善父类的抽象逻辑。
- Implementor（实现化）：定义实现类的接口，通常提供比 Abstraction 更底层的操作。
- ConcreteImplementor（具体实现化）：实现 Implementor 接口，给出具体实现。

  
## 案例1 对象适配器模型

业务场景：多通道消息通知系统

1. 实现化角色（发送渠道接口）
```java
// 消息发送渠道
public interface MessageSender {
    void send(String content);
}
```
2. 具体实现化（各渠道实现）
```java
public class EmailSender implements MessageSender {
    @Override
    public void send(String content) {
        System.out.println("发送邮件：" + content);
    }
}

public class SmsSender implements MessageSender {
    @Override
    public void send(String content) {
        System.out.println("发送短信：" + content);
    }
}

public class AppPushSender implements MessageSender {
    @Override
    public void send(String content) {
        System.out.println("App推送：" + content);
    }
}
```

3. 抽象化角色（消息抽象）
```java
public abstract class Message {
    protected MessageSender sender;

    public Message(MessageSender sender) {
        this.sender = sender;
    }

    public abstract void send(String content);
}
```

4. 扩充抽象化（不同类型消息）
```java
// 普通消息
public class NormalMessage extends Message {
    public NormalMessage(MessageSender sender) {
        super(sender);
    }

    @Override
    public void send(String content) {
        sender.send("[普通]" + content);
    }
}

// 紧急消息
public class UrgentMessage extends Message {
    public UrgentMessage(MessageSender sender) {
        super(sender);
    }

    @Override
    public void send(String content) {
        sender.send("[紧急!!]" + content);
    }
}

// 定时消息（示例：延迟发送）
public class TimedMessage extends Message {
    private LocalDateTime scheduleTime;

    public TimedMessage(MessageSender sender, LocalDateTime scheduleTime) {
        super(sender);
        this.scheduleTime = scheduleTime;
    }

    @Override
    public void send(String content) {
        System.out.println("定时发送时间：" + scheduleTime);
        sender.send("[定时]" + content);
    }
}
```
5.客户端调用：
```java
public class NotificationService {
    public static void main(String[] args) {
        // 组合一：普通消息 + 邮件
        Message normalEmail = new NormalMessage(new EmailSender());
        normalEmail.send("您的订单已发货");

        // 组合二：紧急消息 + 短信
        Message urgentSms = new UrgentMessage(new SmsSender());
        urgentSms.send("账户异常登录，请及时处理");

        // 组合三：定时消息 + App推送
        Message timedPush = new TimedMessage(
                new AppPushSender(),
                LocalDateTime.now().plusHours(1)
        );
        timedPush.send("明日天气提醒");
    }
}
```

输出：
```
发送邮件：[普通]您的订单已发货
发送短信：[紧急!!]账户异常登录，请及时处理
定时发送时间：2026-11-09T10:30
App推送：[定时]明日天气提醒
```

类数量对比：
- 继承方案：3种消息 × 3种渠道 = 9个类
- 桥接方案：3种消息 + 3种渠道 = 6个类（加上抽象类共8个）
- 当维度增加到4×4时，继承需要16个类，桥接只需要8个类，优势更明显。

 
##  优缺点

优点：
- 分离抽象与实现：两者可以独立扩展，互不影响。
- 符合开闭原则：新增消息类型或新增渠道都不需要修改现有代码。
- 避免类爆炸：将 M × N 的继承关系降为 M + N 的组合关系。
- 实现细节对客户端透明：客户端只需关心抽象接口，无需了解具体实现。


缺点：
- 增加系统复杂度：需要额外设计抽象层和实现层，对初学者不易理解。
- 要求正确识别两个独立变化的维度：如果维度划分不当，反而会增加维护成本。


## 适用场景

- 一个类存在两个（或多个）独立变化的维度，且这两个维度都需要扩展。
- 不希望使用继承，或继承会导致类数量急剧增长。
- 抽象化角色和实现化角色可以独立扩展，且它们的改变不会相互影响。
- 需要在运行时动态切换实现（例如根据配置选择不同的数据库驱动）。


## 实际应用

| 领域   | 应用方式       |
|------|-----------------|
| 日志框架 |SLF4J 是抽象化，Logback、Log4j、JUL 是实现化。用户面向 SLF4J 编程，底层日志实现可以热切换。  |
| JDBC 驱动 | java.sql.DriverManager 是抽象化，不同数据库的 Driver 是实现化。应用程序通过统一的 JDBC 接口操作数据库，底层驱动可以随时切换（MySQL、Oracle、PostgreSQL）。 |
| 图形绘制 | 形状（圆形、矩形）是抽象化，绘制引擎（DirectX、OpenGL、Vulkan）是实现化。两者独立变化。 |
| UI 控件 |窗口（Button、TextField）是抽象化，操作系统渲染（Windows、Mac、Linux）是实现化。  |
| 消息队列 | 消息生产者/消费者是抽象化，底层消息中间件（Kafka、RocketMQ、RabbitMQ）是实现化。     |

## 比较 桥接模式 vs 适配器模式

|对比项 |桥接模式 |适配器模式|
|------|------|---------|
|设计目的 |将抽象与实现分离，使它们可以独立变化 |将不兼容的接口转换为客户端期望的接口|
|使用时机| 设计阶段，预见两个维度会独立变化| 后期集成，已有类接口不匹配|
|关系方向| 抽象部分持有实现部分的引用 |适配器包装被适配者|
|典型例子 |JDBC 驱动、日志门面  | 电源转换插头、SLF4J 桥接器|

## 总结：

桥接模式的核心是用组合代替继承，将“抽象”和“实现”两个维度解耦，让它们各自独立演化。

它特别适合处理多维度变化的系统，能够显著减少类的数量，提高代码的可扩展性和可维护性。

记住一句口诀：“继承是静态的，组合是动态的；桥接让变化各走各路。”