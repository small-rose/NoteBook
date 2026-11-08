---
layout: default
title: Flyweight Pattern
parent: Design Patterns
has_children: false
nav_order: 21
---

Here are java Flyweight Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是享元模式？

享元模式（Flyweight Pattern）

享元模式是一种结构型设计模式，它通过共享大量细粒度的对象来减少内存占用和提高性能。

核心思想是：将对象的状态分为内部状态（Intrinsic State）和外部状态（Extrinsic State）。内部状态是对象可共享的、不变的部分；外部状态依赖于上下文，由客户端在使用时传入，不存储在享元对象中。

考虑一个文本编辑器：每个字符都是一个对象，如果一篇文档有10万个字符，就需要创建10万个字符对象。

每个字符对象包含字体、字号、颜色、字形等信息，内存消耗巨大。

但实际上，很多字符的格式是相同的（例如所有“A”都用Times New Roman 12pt黑色）。

享元模式将字符的格式信息作为内部状态共享，每个唯一的格式组合只创建一个对象，而字符的位置、所属段落等作为外部状态由客户端维护。

## 模式结构

```
┌──────────────────┐          ┌─────────────────────┐
│    Flyweight     │          │   FlyweightFactory  │
│ + operation(ext) │          │ + getFlyweight(key) │
└────────┬─────────┘          └──────────┬──────────┘
         ▲                               │ 管理
┌────────┴─────────┐          ┌──────────▼──────────┐
│ ConcreteFlyweight│          │  享元对象池           │
│ - intrinsicState │          │(Map<key, Flyweight>)│
│ + operation(ext) │          └─────────────────────┘
└──────────────────┘
```

核心角色：

- Flyweight（抽象享元）：声明一个接口，接受外部状态作为参数。
- ConcreteFlyweight（具体享元）：实现 Flyweight 接口，存储内部状态。
- FlyweightFactory（享元工厂）：创建和管理享元对象，确保共享。当客户端请求一个享元时，工厂检查是否已存在，若存在则返回现有对象，否则创建新对象。
- Client（客户端）：维护外部状态，在调用享元方法时传入。

  
## 案例1 棋牌游戏中的棋子

业务场景：公司组织架构与薪资统计

假设你正在开发一个围棋游戏。棋盘上有361个交叉点，每个点上可以放置黑子或白子。

如果每下一颗棋子就创建一个对象，一盘棋可能有几百个棋子对象。但所有黑子的外观完全相同（圆形、黑色），所有白子也完全相同。

因此，可以将颜色作为内部状态共享，而坐标位置作为外部状态。
 
1. 抽象享元
```java
public interface ChessPiece {
    void place(int x, int y);  // 显示棋子，x,y是外部状态
}
```

2. 具体享元
```java
public class ConcreteChessPiece implements ChessPiece {
    private String color;  // 内部状态：黑色或白色

    public ConcreteChessPiece(String color) {
        this.color = color;
        // 模拟加载棋子图像等耗时操作
        System.out.println("创建" + color + "棋子对象");
    }

    @Override
    public void place(int x, int y) {
        System.out.println(color + "棋子放在(" + x + "," + y + ")");
    }
}
```

3. 享元工厂
```java
import java.util.HashMap;
import java.util.Map;

public class ChessPieceFactory {
    private static final Map<String, ChessPiece> pool = new HashMap<>();

    public static ChessPiece getChessPiece(String color) {
        ChessPiece piece = pool.get(color);
        if (piece == null) {
            piece = new ConcreteChessPiece(color);
            pool.put(color, piece);
        }
        return piece;
    }

    public static int getPoolSize() {
        return pool.size();
    }
}
```
4. 客户端（棋盘控制器）
```java
public class GoGame {
    public static void main(String[] args) {
        // 下棋过程中，始终只使用两个享元对象（黑子和白子）
        ChessPiece blackPiece = ChessPieceFactory.getChessPiece("黑色");
        ChessPiece whitePiece = ChessPieceFactory.getChessPiece("白色");

        // 第一手：黑子落在(3,3)
        blackPiece.place(3, 3);

        // 第二手：白子落在(16,16)
        whitePiece.place(16, 16);

        // 第三手：黑子落在(4,4)
        blackPiece.place(4, 4);

        // 第四手：白子落在(15,15)
        whitePiece.place(15, 15);

        System.out.println("享元池中对象数：" + ChessPieceFactory.getPoolSize());
        // 输出：享元池中对象数：2
    }
}
```

输出
```text
创建黑色棋子对象
创建白色棋子对象
黑色棋子放在(3,3)
白色棋子放在(16,16)
黑色棋子放在(4,4)
白色棋子放在(15,15)
享元池中对象数：2
```

效果：即使下了1000手，也只创建了2个棋子对象，大幅节省内存。每个棋子的位置（外部状态）由客户端在调用 place() 时传入。

## 内部状态 vs 外部状态

|状态类型 |特点| 存储位置| 示例|
|-------|---|--------|-----|
|内部状态| 可共享、不变、与上下文无关 |享元对象内部 |棋子颜色、字符字体格式|
|外部状态 |不可共享、随上下文变化 |客户端维护，调用时传入 |棋子坐标、字符在文档中的位置  |
|设计关键：必须正确区分哪些状态可以共享，哪些不能。错误地将可变状态放入内部状态会导致线程安全问题。|

##  优缺点

优点：
- 大幅减少内存占用：尤其当大量对象具有相同内部状态时，效果显著。
- 提升性能：减少对象创建次数，降低GC压力。
- 集中管理：享元工厂可以控制对象的创建策略（如懒加载、预加载）。

缺点：
- 增加系统复杂度：需要分离内部状态和外部状态，并设计工厂管理。
- 时间换空间：每次调用时需要传递外部状态，可能增加CPU开销。
- 线程安全问题：如果享元对象本身是可变的，需要同步控制（通常享元应设计为不可变）。


## 适用场景

- 系统中有大量相似对象，且这些对象的大部分状态可以外部化。
- 内存资源紧张，需要优化对象存储。
- 对象创建成本较高（如加载图片、建立数据库连接）。
- 需要缓存对象池，避免重复创建。



## 比较 享元模式 vs 单例模式 vs 对象池

|模式	|核心	|对象数量	|用途|
|------|-------|--------|------|
|享元模式 |	共享不可变状态，外部状态分离	|有限个（按内部状态分类）	|减少内存占用|
|单例模式 |	全局唯一实例	|1个	|控制实例数量|
|对象池 |	复用对象，避免频繁创建/销毁	|固定大小池	|提升创建/销毁性能（如线程池）|

>注意：享元模式强调共享相同内部状态的对象，而对象池强调的是复用对象实例（即使状态不同也要重置后复用）。

## 实际应用

| 领域   | 应用方式  |
|------|-----------------------------------------------------------------|
| Java 标准库 |String 常量池：相同字面量的字符串共享同一个对象。Integer.valueOf(-128~127) 返回缓存对象。  |
| 文本编辑器| 每个字符的字体、颜色等格式作为内部状态共享，位置作为外部状态。。  |
| 游戏开发 | 粒子系统中的粒子纹理、子弹模型共享，位置、速度、角度作为外部状态。 |
| GUI 控件 |Swing 中 Icon 对象可以被多个按钮共享，按钮的位置和文本是外部状态。   |
| 数据库连接池 | 连接对象本身是享元（共享），但连接的当前数据库、事务状态等是外部状态（由客户端管理）。   |
| 浏览器渲染 | CSS 样式计算中，相同的样式规则（如 .red{color:red}）只存储一份，应用于多个DOM元素。   |


## 总结：

享元模式的核心是“共享不变，分离变化”。它将对象的可变部分剥离出去，让不可变部分被大量复用，从而在保证功能的前提下极大节省内存。

在实际开发中，Java 的 String 常量池和 Integer 缓存是最经典的享元应用。

记住一句话：“同样的东西，只造一次；不同的场景，传参就行。”