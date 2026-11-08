---
layout: default
title: Chain of Responsibility
parent: Design Patterns
has_children: false
nav_order: 23
---

Here are java Chain of Responsibility Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是代理模式？

责任链模式（Chain of Responsibility Pattern）

责任链模式是一种行为型设计模式，它允许多个对象都有机会处理请求，将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。

核心思想是：将请求的发送者和接收者解耦，让多个接收者都有机会处理请求，并且可以动态地组合处理顺序。

> 生活中的类比：公司内部的报销审批流程。员工提交报销单，首先由组长审批，组长权限不够则转给经理，经理不行再转给总监，总监不行再转给CEO。每个审批人要么处理请求，要么将其传递给下一个审批人。

考虑一个在线客服系统：用户提交一个工单，系统需要根据工单类型（技术问题、账单问题、投诉建议）分配给不同的处理人员。

如果使用硬编码的 if-else 或 switch-case，当处理规则变化时（例如新增一个处理环节），需要修改核心代码，违反开闭原则。

责任链模式将每个处理者封装成一个独立的节点，通过链式结构动态组合，可以灵活地增删处理者或调整顺序。

## 模式结构

```
┌──────────────────┐
│     Handler      │    （抽象处理者）
│ - successor      │
│ + handleRequest()│
└────────┬─────────┘
         ▲
         │ 继承
┌────────┴─────────┐
│ ConcreteHandler1 │
│ + handleRequest()│
└──────────────────┘
```

核心角色：

- Handler（抽象处理者）：定义一个处理请求的接口，并持有对下一个处理者的引用。
- ConcreteHandler（具体处理者）：处理它所负责的请求，如果可以处理则处理，否则将请求转发给后继者。
- Client（客户端）：负责组装链，并向链首发送请求。

 
  
##  业务案例：报销审批系统

业务案例：报销审批系统

假设一家公司有三级审批权限：组长（≤1000元）、经理（≤5000元）、总监（＞5000元）。超过总监权限的需要CEO审批（这里简化到总监为止）。

1.抽象处理者

```java
public abstract class Approver {
    protected Approver nextApprover;  // 下一个审批人
    protected String name;

    public Approver(String name) {
        this.name = name;
    }

    public void setNextApprover(Approver nextApprover) {
        this.nextApprover = nextApprover;
    }

    // 模板方法：处理请求
    public abstract void processRequest(PurchaseRequest request);
}
```

2. 具体处理者
```java
// 请求对象
public class PurchaseRequest {
    private double amount;
    private String purpose;

    public PurchaseRequest(double amount, String purpose) {
        this.amount = amount;
        this.purpose = purpose;
    }

    public double getAmount() { return amount; }
    public String getPurpose() { return purpose; }
}

// 组长：处理 ≤1000 元的请求
public class GroupLeader extends Approver {
    public GroupLeader(String name) {
        super(name);
    }

    @Override
    public void processRequest(PurchaseRequest request) {
        if (request.getAmount() <= 1000) {
            System.out.println(name + "（组长）审批通过：" + request.getPurpose() + "，金额：" + request.getAmount());
        } else if (nextApprover != null) {
            System.out.println(name + "（组长）权限不足，转交上级");
            nextApprover.processRequest(request);
        }
    }
}

// 经理：处理 ≤5000 元的请求
public class Manager extends Approver {
    public Manager(String name) {
        super(name);
    }

    @Override
    public void processRequest(PurchaseRequest request) {
        if (request.getAmount() <= 5000) {
            System.out.println(name + "（经理）审批通过：" + request.getPurpose() + "，金额：" + request.getAmount());
        } else if (nextApprover != null) {
            System.out.println(name + "（经理）权限不足，转交上级");
            nextApprover.processRequest(request);
        }
    }
}

// 总监：处理 >5000 元的请求
public class Director extends Approver {
    public Director(String name) {
        super(name);
    }

    @Override
    public void processRequest(PurchaseRequest request) {
        if (request.getAmount() > 5000) {
            System.out.println(name + "（总监）审批通过：" + request.getPurpose() + "，金额：" + request.getAmount());
        } else {
            // 理论上不会到这里，但保留传递逻辑
            if (nextApprover != null) {
                nextApprover.processRequest(request);
            }
        }
    }
}
```

3. 客户端组装链并发送请求
```java
public class Client {
    public static void main(String[] args) {
        // 创建审批人
        Approver groupLeader = new GroupLeader("张三");
        Approver manager = new Manager("李四");
        Approver director = new Director("王五");

        // 组装责任链：组长 → 经理 → 总监
        groupLeader.setNextApprover(manager);
        manager.setNextApprover(director);

        // 创建几个报销请求
        PurchaseRequest req1 = new PurchaseRequest(500, "购买办公用品");
        PurchaseRequest req2 = new PurchaseRequest(3000, "团建活动经费");
        PurchaseRequest req3 = new PurchaseRequest(8000, "服务器采购");

        // 从链首开始处理
        groupLeader.processRequest(req1);
        System.out.println("---");
        groupLeader.processRequest(req2);
        System.out.println("---");
        groupLeader.processRequest(req3);
    }
}
```

输出：
```
张三（组长）审批通过：购买办公用品，金额：500.0
        ---
张三（组长）权限不足，转交上级
李四（经理）审批通过：团建活动经费，金额：3000.0
        ---
张三（组长）权限不足，转交上级
李四（经理）权限不足，转交上级
王五（总监）审批通过：服务器采购，金额：8000.0
```

核心价值：

- 客户端只需向链首发送请求，无需知道谁最终处理。
- 新增审批层级（如增加“副总裁”）只需新建一个类并插入链中，无需修改现有处理者。
- 可以动态调整审批顺序（如临时绕过某环节）。

## 两种变体：纯责任链 vs 不纯责任链

|类型	|特点	|示例|
|-------|------|----|
|纯责任链 |	一个请求要么被某个处理者处理，要么不被处理（到达链尾仍未处理则丢弃）	|上述报销审批案例|
|不纯责任链 |	请求可以被多个处理者处理，每个处理者都可以拦截并处理，然后继续传递（类似拦截器链）	|Java Servlet Filter 链|

## 优缺点

优点：

- 降低耦合度：请求发送者与接收者解耦，发送者无需知道谁在处理。
- 增强灵活性：可以动态地新增、删除、调整处理者顺序。
- 符合开闭原则：新增处理者无需修改现有代码。
- 简化对象：每个处理者只需关注自己负责的请求范围，职责单一。

缺点：

- 请求可能未被处理：如果链尾没有兜底处理者，请求可能被丢弃（可通过添加默认处理者解决）。
- 性能问题：链过长时，请求需要遍历整个链，影响效率。
- 调试困难：请求在链中流转，调用栈较深，定位问题相对复杂。


## 适用场景

- 多个对象可以处理同一请求，但具体由哪个对象处理在运行时决定。
- 需要动态指定处理者集合，且处理顺序可能变化。
- 希望在不明确接收者的情况下，向多个对象中的一个提交请求。
- 典型的应用：审批流程、异常处理、事件冒泡、拦截器链。

## 责任链模式 vs 装饰器模式

|对比项	|责任链模式	|装饰器模式|
|-------|----------|----------|
|目的	|将请求沿着链传递，直到有人处理	|动态添加功能，层层包裹|
|处理方式	|每个节点决定是否处理或继续传递	|每个节点都执行自己的功能，然后调用下一个|
|链的方向	|单向传递，可能中断	|始终执行完所有装饰器|
|典型例子	|审批流程、Filter 链	|Java I/O 流 |


## 现实应用

|领域	| 示例                                                                   |
|------|----------------------------------------------------------------------|
|Java Servlet Filter | 	FilterChain 是典型的不纯责任链，每个 Filter 可以处理请求并决定是否继续传递。        |
|Spring MVC 拦截器 | 	HandlerInterceptor 组成拦截器链，preHandle/postHandle/afterCompletion 形成处理链。 |
|Netty 管道 | ChannelPipeline 中的 ChannelHandler 组成责任链，入站/出站事件沿链传播。     |
|日志框架 | Logback 的 Logger 继承机制：子 Logger 未设置级别时，向父 Logger 传递日志请求。  |
|异常处理 |Java 中的 catch 块链：异常从最近的 catch 开始匹配，逐级向上传递。    |
|UI 事件冒泡 |JavaScript 中 DOM 事件的冒泡阶段，事件从目标元素向上传播到 document。 |

## 总结：

责任链模式的核心是“链式传递，谁来处理？” 它将请求的发送者和接收者解耦，让多个对象都有机会处理请求，并且可以灵活地组合处理顺序。

在实际开发中，Spring MVC 的拦截器链、Servlet 的 Filter 链都是责任链模式的经典应用。

记住一句话：“把请求扔进链里，总有人会接住它。”