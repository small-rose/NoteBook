---
layout: default
title: Observer Pattern
parent: Design Patterns
has_children: false
nav_order: 29
---

Here are java Observer Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是观察者模式？

观察者模式（Observer Pattern）

观察者模式是一种行为型设计模式，它定义了对象之间的一种一对多的依赖关系，使得当一个对象（主题）的状态发生改变时，所有依赖于它的对象（观察者）都会得到通知并被自动更新。

核心思想是：“发布-订阅”——主题（Subject）发布事件，观察者（Observer）订阅事件，两者解耦。

>生活中的类比：微信公众号。你关注了一个公众号（订阅），当公众号发布新文章时（状态变化），所有粉丝都会收到推送通知。你不需要天天去刷公众号有没有更新，公众号也不需要知道你具体是谁。

一个天气预报系统：气象站（数据源）采集温度、湿度、气压等数据，需要实时更新到多个显示面板（当前天气面板、统计面板、预报面板）。
如果不使用观察者模式，你可能需要让气象站直接调用每个面板的更新方法，这样：

- 新增一个面板需要修改气象站的代码，违反开闭原则。
- 气象站需要知道所有面板的具体类型，耦合度高。
- 面板的更新频率、时机难以统一控制。

观察者模式将气象站作为主题，面板作为观察者。气象站只负责通知所有已注册的观察者，而观察者自行决定如何处理数据。

## 模式结构

```
┌──────────────────┐          ┌──────────────────────┐
│    Subject       │          │     Observer         │
│ + attach(o)      │◄────────▶│ + update(data)       │
│ + detach(o)      │          └──────────────────────┘
│ + notifyObservers│                     ▲
└────────┬─────────┘                     │ 实现
         ▲                               │
         │ 实现                           │
┌────────┴─────────┐          ┌──────────┴───────────┐
│ ConcreteSubject  │          │ ConcreteObserver     │
│ - observers      │          │ + update(data)       │
│ + getState()     │          └──────────────────────┘
│ + setState()     │
└──────────────────┘
```

核心角色：

- Subject（主题/被观察者）：维护观察者列表，提供注册、注销和通知接口。
- ConcreteSubject（具体主题）：存储感兴趣的状态，当其状态改变时通知所有观察者。
- Observer（观察者）：定义更新接口，当收到主题通知时调用。
- ConcreteObserver（具体观察者）：实现更新接口，以保持自身状态与主题一致。

##  业务案例：气象站与显示面板

实现一个气象站系统，采集天气数据，并实时更新三个显示面板：当前状况面板、统计数据面板、天气预报面板。

1. 观察者接口
```java
public interface Observer {
    void update(float temperature, float humidity, float pressure);
}
```

2. 主题接口
```java
public interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}
```

3. 具体主题（气象站）
```java
import java.util.ArrayList;
import java.util.List;

public class WeatherData implements Subject {
    private List<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }

    // 当数据更新时调用此方法
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }

    private void measurementsChanged() {
        notifyObservers();
    }
}
```

4. 具体观察者（显示面板）
```java
// 当前状况面板
public class CurrentConditionsDisplay implements Observer {
    private float temperature;
    private float humidity;

    @Override
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    public void display() {
        System.out.println("当前状况：温度 " + temperature + "°C，湿度 " + humidity + "%");
    }
}

// 统计数据面板（显示最高/最低/平均温度）
public class StatisticsDisplay implements Observer {
    private float maxTemp = Float.MIN_VALUE;
    private float minTemp = Float.MAX_VALUE;
    private float tempSum = 0.0f;
    private int numReadings = 0;

    @Override
    public void update(float temperature, float humidity, float pressure) {
        tempSum += temperature;
        numReadings++;
        if (temperature > maxTemp) maxTemp = temperature;
        if (temperature < minTemp) minTemp = temperature;
        display();
    }

    public void display() {
        System.out.println("气温统计：最高 " + maxTemp + "°C，最低 " + minTemp +
                "°C，平均 " + (tempSum / numReadings) + "°C");
    }
}

// 天气预报面板（简单预测：温度上升->炎热，下降->凉爽）
public class ForecastDisplay implements Observer {
    private float currentPressure = 29.92f;
    private float lastPressure;

    @Override
    public void update(float temperature, float humidity, float pressure) {
        lastPressure = currentPressure;
        currentPressure = pressure;
        display();
    }

    public void display() {
        System.out.print("天气预报：");
        if (currentPressure > lastPressure) {
            System.out.println("天气正在好转（气压上升）");
        } else if (currentPressure < lastPressure) {
            System.out.println("小心降雨（气压下降）");
        } else {
            System.out.println("天气稳定");
        }
    }
}
```
5. 客户端
```java
public class WeatherStation {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();

        // 创建观察者并注册
        CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay();
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay();
        ForecastDisplay forecastDisplay = new ForecastDisplay();

        weatherData.registerObserver(currentDisplay);
        weatherData.registerObserver(statisticsDisplay);
        weatherData.registerObserver(forecastDisplay);

        // 模拟新的气象数据
        System.out.println("=== 第一次测量 ===");
        weatherData.setMeasurements(25.0f, 65.0f, 1013.0f);

        System.out.println("\n=== 第二次测量 ===");
        weatherData.setMeasurements(27.0f, 70.0f, 1010.0f);

        System.out.println("\n=== 第三次测量（移除统计面板） ===");
        weatherData.removeObserver(statisticsDisplay);
        weatherData.setMeasurements(22.0f, 80.0f, 1015.0f);
    }
}
```
输出：
```
=== 第一次测量 ===
当前状况：温度 25.0°C，湿度 65.0%
气温统计：最高 25.0°C，最低 25.0°C，平均 25.0°C
天气预报：天气稳定

=== 第二次测量 ===
当前状况：温度 27.0°C，湿度 70.0%
气温统计：最高 27.0°C，最低 25.0°C，平均 26.0°C
天气预报：小心降雨（气压下降）

=== 第三次测量（移除统计面板） ===
当前状况：温度 22.0°C，湿度 80.0%
天气预报：天气正在好转（气压上升）
```

核心价值：

- 气象站（主题）不知道也不关心有哪些显示面板（观察者），只需调用 notifyObservers()。
- 新增显示面板只需实现 Observer 接口并注册，无需修改气象站代码。
- 可以动态添加或移除观察者（如第三次测量移除了统计面板）。

## 优缺点

优点：

- 抽象耦合：主题只知道观察者实现了某个接口，不需要知道具体类。
- 支持广播通信：主题自动通知所有已注册的观察者。
- 符合开闭原则：新增观察者无需修改主题代码。
- 支持动态关联：可以在运行时添加或移除观察者。


缺点：

- 可能导致意外更新：观察者收到通知时不知道其他观察者的存在，可能产生连锁反应。
- 性能问题：如果观察者很多或者更新操作耗时，通知过程可能成为瓶颈。
- 循环依赖风险：如果观察者和主题互相观察，可能导致死循环。
- 通知顺序不可控：观察者收到的通知顺序取决于注册顺序，可能引发不一致。


## 适用场景

- 一个对象状态改变需要同时更新其他对象，且对象数量不确定。
- 一个对象需要通知其他对象，但又不希望与这些对象紧密耦合。
- 需要实现事件驱动架构（如 GUI 事件处理、消息系统）。
- 典型的应用：MVC 架构中的 Model-View 同步、事件监听器、发布-订阅系统。

## 观察者模式 vs 发布-订阅模式

|对比项	|观察者模式	|发布-订阅模式|
|-------|----------|---------|
|耦合度	|主题直接持有观察者引用（强引用）	|通过消息代理（Broker）中转，完全解耦|
|通信方式	|同步（通常直接调用 update）	|异步（通过消息队列）|
|典型实现	|Java 的 Observable/Observer	|Kafka、Redis Pub/Sub|
|适用场景	|进程内、实时性要求高	|分布式、跨进程、解耦要求更高|


## 现实应用

| 领域	 | 示例                                                     |
|-----------|--------------------------------------------------------|
| Java 事件监听 |ActionListener、MouseListener 等都是观察者模式的应用。 |
| Spring 事件机制 |ApplicationEvent + ApplicationListener 实现松耦合的事件驱动。|
| RxJava / Reactor |响应式编程中的 Observable 和 Observer 是观察者模式的进化版。|
| Android 广播 |BroadcastReceiver 接收系统或应用发出的广播。|
| 消息队列      |Kafka、RabbitMQ 的生产者-消费者模型本质上是观察者模式。 |
| 前端框架      |Vue.js 的响应式系统（数据变化自动更新视图）基于观察者模式。 |

## 总结：

观察者模式是实现事件驱动编程的基石，它让对象之间既能保持松耦合，又能实现高效的状态同步。

无论是 GUI 中的按钮点击、MVC 中的模型更新，还是分布式系统中的消息通知，观察者模式都扮演着核心角色。

记住一句话：“我不找你，但我变了就会告诉你；你想知道，就提前告诉我。”