---
layout: default
title: Facade Pattern
parent: Design Patterns
has_children: false
nav_order: 20
---

Here are java Facade Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是外观模式？

外观模式（Facade Pattern）

外观模式是一种结构型设计模式，它为子系统中的一组接口提供一个统一的高层接口，使得子系统更容易使用。

通俗地说，外观模式就是“一键操作”——你不需要了解空调、电视、音响各自怎么开关，只需要按一下“影院模式”按钮，系统自动帮你完成所有操作。

在一个复杂的系统中，通常包含多个相互协作的子系统或模块。客户端如果直接与这些子系统交互，需要了解每个子系统的接口、调用顺序、依赖关系，导致客户端代码臃肿、耦合度高、难以维护。

外观模式通过引入一个外观类，将子系统的复杂操作封装成一个简单的接口，客户端只需与外观类交互，从而降低耦合、简化使用。

## 模式结构

```
┌─────────────────────────────────────────────────┐
│                   Client                        │
└───────────────────────┬─────────────────────────┘
                        │ 使用
┌───────────────────────▼─────────────────────────┐
│                  Facade                         │
│  + simpleOperation()                            │
└───────┬────────────────────┬────────────────────┘
        │ 委派                │ 委派
┌───────▼──────┐    ┌────────▼───────┐    ┌────────▼───────┐
│ SubsystemA   │    │  SubsystemB    │    │  SubsystemC    │
│+ operationA()│    │ + operationB() │    │ + operationC() │
└──────────────┘    └────────────────┘    └────────────────┘
```

核心角色：

- Facade（外观）：知道哪些子系统类负责处理请求，将客户端请求委派给适当的子系统对象。
- Subsystem（子系统）：实现子系统的功能，处理外观类指派的任务。子系统不知道外观的存在（即没有对外观的引用）。
- Client（客户端）：只与外观交互，不直接调用子系统。

  
## 案例1 电商下单流程

业务场景：公司组织架构与薪资统计

假设一个电商平台的下单流程涉及多个子系统：

- 库存系统：检查商品库存并锁定库存。
- 支付系统：处理扣款。
- 物流系统：创建配送单。
- 通知系统：发送短信/邮件通知。
- 积分系统：更新用户积分。

如果没有外观模式，客户端需要依次调用这五个子系统，并处理它们之间的依赖和异常：
```java
// 不使用外观：客户端代码混乱
public void placeOrder(Order order) {
    InventorySystem inventory = new InventorySystem();
    PaymentSystem payment = new PaymentSystem();
    LogisticsSystem logistics = new LogisticsSystem();
    NotificationSystem notification = new NotificationSystem();
    PointsSystem points = new PointsSystem();

    if (!inventory.checkStock(order.getItems())) {
        throw new RuntimeException("库存不足");
    }
    inventory.lockStock(order.getItems());

    if (!payment.debit(order.getTotalAmount())) {
        inventory.unlockStock(order.getItems()); // 回滚
        throw new RuntimeException("支付失败");
    }

    String trackingNo = logistics.createShipment(order);
    notification.sendOrderConfirmation(order.getUserId(), trackingNo);
    points.addPoints(order.getUserId(), order.getTotalAmount() * 0.01);
}
```
问题：

- 客户端需要了解所有子系统的接口和调用顺序。
- 异常处理复杂（支付失败需要回滚库存）。
- 子系统替换或升级时，客户端代码也需要修改。

使用外观模式重构

1. 子系统类（简化示例）
```java
public class InventorySystem {
    public boolean checkStock(List<Item> items) {
        System.out.println("检查库存...");
        return true;
    }
    public void lockStock(List<Item> items) {
        System.out.println("锁定库存");
    }
    public void unlockStock(List<Item> items) {
        System.out.println("解锁库存");
    }
}

public class PaymentSystem {
    public boolean debit(double amount) {
        System.out.println("扣款：" + amount);
        return true;
    }
    public void refund(double amount) {
        System.out.println("退款：" + amount);
    }
}

public class LogisticsSystem {
    public String createShipment(Order order) {
        System.out.println("创建物流单");
        return "SF1234567890";
    }
}

public class NotificationSystem {
    public void sendOrderConfirmation(long userId, String trackingNo) {
        System.out.println("发送订单确认通知给用户" + userId + "，运单号：" + trackingNo);
    }
}

public class PointsSystem {
    public void addPoints(long userId, double points) {
        System.out.println("为用户" + userId + "增加积分：" + points);
    }
}
```

2. 外观类
```java
public class OrderFacade {
    private InventorySystem inventory;
    private PaymentSystem payment;
    private LogisticsSystem logistics;
    private NotificationSystem notification;
    private PointsSystem points;

    public OrderFacade() {
        this.inventory = new InventorySystem();
        this.payment = new PaymentSystem();
        this.logistics = new LogisticsSystem();
        this.notification = new NotificationSystem();
        this.points = new PointsSystem();
    }

    /**
     * 统一的下单接口
     */
    public void placeOrder(Order order) {
        // 1. 检查并锁定库存
        if (!inventory.checkStock(order.getItems())) {
            throw new RuntimeException("库存不足");
        }
        inventory.lockStock(order.getItems());

        try {
            // 2. 支付
            if (!payment.debit(order.getTotalAmount())) {
                throw new RuntimeException("支付失败");
            }

            // 3. 创建物流
            String trackingNo = logistics.createShipment(order);

            // 4. 发送通知
            notification.sendOrderConfirmation(order.getUserId(), trackingNo);

            // 5. 增加积分
            points.addPoints(order.getUserId(), order.getTotalAmount() * 0.01);

        } catch (Exception e) {
            // 发生异常时回滚库存和支付
            inventory.unlockStock(order.getItems());
            payment.refund(order.getTotalAmount());
            throw e;
        }
    }
}
```

3. 客户端操作
```java
public class Client {
    public static void main(String[] args) {
        OrderFacade facade = new OrderFacade();
        Order order = new Order(/* ... */);
        facade.placeOrder(order);  // 一行搞定
    }
}
```

改进效果：

- 客户端只需调用 facade.placeOrder()，无需关心内部细节。
- 事务性操作（异常回滚）集中在外观类中，避免客户端遗漏。
- 子系统替换（如更换支付网关）只需修改外观类，客户端不受影响。

 
##  优缺点

优点：
- 简化客户端使用：隐藏子系统的复杂性，提供简单接口。
- 降低耦合：客户端与子系统解耦，子系统的变化不会影响客户端。
- 提高安全性：可以限制客户端对子系统的直接访问，只暴露必要功能。
- 利于分层：外观可以作为系统分层之间的入口点，例如业务层与数据层之间加一个外观。


缺点：
- 不符合开闭原则：如果子系统新增功能，可能需要修改外观类（除非外观设计得足够灵活）。
- 可能成为上帝对象：外观类如果承担过多职责，会变得臃肿，违反单一职责原则。
- 过度使用会隐藏有用接口：如果客户端确实需要子系统的某些高级功能，外观可能成为障碍。

## 适用场景

- 为复杂子系统提供简单入口：如编译器（词法分析、语法分析、代码生成）对外提供 compile() 方法。
- 系统分层：每一层通过外观向上层提供服务，隔离内部实现。
- 与第三方库集成：将第三方库的复杂 API 封装成简单接口，降低替换成本。
- 遗留系统现代化：为老旧系统提供新接口，逐步替换内部实现而不影响客户端。


## 比较 外观模式 vs 中介者模式

| 对比项	 | 	外观模式	| 中介者模式              |
|----|----------------|--------------------|
| 目的 | 为子系统提供统一接口，简化客户端使用	| 协调多个对象之间的交互，降低它们之间的耦合 |
| 方向 | 单向：客户端 → 外观 → 子系统	| 双向：同事对象之间通过中介者通信   |
| 关系 | 子系统之间可以互相调用，外观只做封装	| 同事对象之间不直接通信，全部通过中介者 |
| 典型例子 |JdbcTemplate、智能家居一键模式	|聊天室、飞机塔台调度|                   



## 实际应用

| 领域   | 应用方式  |
|------|-------------------------------------------------------------------|
| Java EE | javax.faces.context.FacesContext 封装了 JSF 生命周期中的多个子系统。   |
| Spring 框架| JdbcTemplate 是对 JDBC 子系统的外观，隐藏了连接管理、异常处理、资源释放等细节。  |
| Tomcat | org.apache.catalina.startup.Bootstrap 是 Tomcat 启动的外观，内部协调多个容器和组件。 |
| 操作系统 | 系统调用接口（如 POSIX）是硬件和内核子系统的外观。   |
| 智能家居 | “回家模式”一键开启灯光、空调、窗帘，就是外观模式。   |


## 总结：

外观模式的核心是“简化”——为复杂的子系统提供一个干净、简单的入口。它不增加新功能，而是封装混乱，让客户端专注于业务逻辑。

在实际项目中，几乎每个系统都会有意无意地使用外观模式（例如 Service 层对 DAO 层的封装）。

记住一句话：“你不需要知道汽车引擎如何点火、变速箱如何换挡，你只需要转动钥匙——这就是外观。”