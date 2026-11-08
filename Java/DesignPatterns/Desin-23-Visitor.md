---
layout: default
title: Visitor Pattern
parent: Design Patterns
has_children: false
nav_order: 33
---

Here are java Visitor Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是模板方法模式？

访问者模式（Visitor Pattern）

访问者模式是一种行为型设计模式，它表示一个作用于某对象结构中各元素的操作，让你可以在不修改元素类的前提下，定义作用于这些元素的新操作。

核心思想是：把“数据结构”和“数据操作”解耦——元素类只负责存数据和接收访问，所有易变的操作逻辑集中到独立的“访问者”中。

>  生活中的类比：公司年会审计。财务、HR、安全员分别来“访问”同一批员工数据。员工（元素）不需要知道审计规则，只需把自己交给访问者；财务算工资、HR查考勤、安全员查违规，各自带着自己的逻辑来“访问”一遍。

访问者模式的关键机制是双分派（Double Dispatch）：

1.第一次分派：element.accept(visitor)，根据元素运行时类型确定调用哪个 accept。

2.第二次分派：accept 内部调用 visitor.visit(this)，根据 this 的编译时类型匹配到对应的重载 visit 方法。

Java 本身只支持单分派，访问者模式靠“元素回调访问者 + 方法重载”模拟出双分派。

## 模式结构

```
┌──────────────────┐          ┌──────────────────────┐
│    Visitor       │          │      Element         │
│ + visit(Book)    │◄──────── │ + accept(Visitor)    │
│ + visit(Electronic)         └──────────────────────┘
└────────┬─────────┘                       ▲
         ▲                                 │ 实现
         │ 实现                    ┌────────┴─────────┐
┌────────┴─────────┐      ┌───────┴──────┐   ┌───────┴──────┐
│ PriceVisitor     │      │    Book      │   │  Electronic  │
│ ReportVisitor    │      │ accept(v)    │   │ accept(v)    │
└──────────────────┘      └──────────────┘   └──────────────┘
```

核心角色：

- Visitor（抽象访问者）：为每种具体元素声明一个 visit 方法（方法重载）。
- ConcreteVisitor（具体访问者）：实现每个 visit 方法，定义对某种元素的具体操作。
- Element（抽象元素）：声明 accept(Visitor) 方法。
- ConcreteElement（具体元素）：实现 accept，在内部调用 visitor.visit(this)。
- ObjectStructure（对象结构）：维护元素集合，提供遍历并让访问者依次访问。

##  业务案例：电商购物车多维度计算

购物车里有多种商品（图书、电子产品）。我们需要支持多种操作：

- 计算总价（图书打折、电子含税）
- 生成订单小票
- 库存预警检查

如果把这些逻辑都塞进 Book、Electronic 类里，类会越写越臃肿；用访问者模式，把每种操作做成独立访问者。

1. 抽象访问者
```java
public interface ShoppingCartVisitor {
    void visit(Book book);
    void visit(Electronic electronic);
}
```

2. 抽象元素
```java
public interface Product {
    void accept(ShoppingCartVisitor visitor);
}
```

3. 具体元素
```java
// 图书
public class Book implements Product {
    private String name;
    private double price;
    private String author;

    public Book(String name, double price, String author) {
        this.name = name;
        this.price = price;
        this.author = author;
    }

    public double getPrice() { return price; }
    public String getName() { return name; }
    public String getAuthor() { return author; }

    @Override
    public void accept(ShoppingCartVisitor visitor) {
        visitor.visit(this);  // 双分派第二步
    }
}

// 电子产品
public class Electronic implements Product {
    private String name;
    private double price;
    private int warrantyYears;
    private int stock;

    public Electronic(String name, double price, int warrantyYears, int stock) {
        this.name = name;
        this.price = price;
        this.warrantyYears = warrantyYears;
        this.stock = stock;
    }

    public double getPrice() { return price; }
    public String getName() { return name; }
    public int getWarrantyYears() { return warrantyYears; }
    public int getStock() { return stock; }

    @Override
    public void accept(ShoppingCartVisitor visitor) {
        visitor.visit(this);
    }
}
```

4. 具体访问者：价格计算
```java
public class PriceCalculatorVisitor implements ShoppingCartVisitor {
    private double total = 0.0;

    @Override
    public void visit(Book book) {
        // 图书打9折
        double finalPrice = book.getPrice() * 0.9;
        total += finalPrice;
        System.out.println("图书《" + book.getName() + "》折后价：" + finalPrice);
    }

    @Override
    public void visit(Electronic electronic) {
        // 电子产品加13%税
        double finalPrice = electronic.getPrice() * 1.13;
        total += finalPrice;
        System.out.println("电子产品" + electronic.getName() + "含税价：" + finalPrice);
    }

    public double getTotal() {
        return total;
    }
}
```

5. 具体访问者：订单小票生成
```java
public class OrderReceiptVisitor implements ShoppingCartVisitor {
    private StringBuilder receipt = new StringBuilder("==== 订单小票 ====\n");

    @Override
    public void visit(Book book) {
        receipt.append("📚 图书：").append(book.getName())
               .append("（作者：").append(book.getAuthor()).append("）\n");
    }

    @Override
    public void visit(Electronic electronic) {
        receipt.append("🔌 电子产品：").append(electronic.getName())
               .append("（保修：").append(electronic.getWarrantyYears()).append("年）\n");
    }

    public String getReceipt() {
        return receipt.toString();
    }
}
```

6. 具体访问者：库存预警
```java
public class StockCheckVisitor implements ShoppingCartVisitor {
    @Override
    public void visit(Book book) {
        // 图书不检查库存
    }

    @Override
    public void visit(Electronic electronic) {
        if (electronic.getStock() < 5) {
            System.out.println("⚠️ 库存预警：" + electronic.getName() + " 仅剩 " + electronic.getStock() + " 件");
        }
    }
}
```
7. 对象结构（购物车）
```java
import java.util.ArrayList;
import java.util.List;

public class ShoppingCart {
    private List<Product> products = new ArrayList<>();

    public void addProduct(Product product) {
        products.add(product);
    }

    public void accept(ShoppingCartVisitor visitor) {
        for (Product product : products) {
            product.accept(visitor);
        }
    }
}
```
8. 客户端
```java
public class StoreClient {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(new Book("设计模式", 89.0, "GoF"));
        cart.addProduct(new Electronic("智能手机", 4999.0, 2, 3));
        cart.addProduct(new Electronic("笔记本电脑", 7999.0, 3, 10));

        System.out.println("=== 价格计算 ===");
        PriceCalculatorVisitor priceVisitor = new PriceCalculatorVisitor();
        cart.accept(priceVisitor);
        System.out.println("总价：" + priceVisitor.getTotal());

        System.out.println("\n=== 订单小票 ===");
        OrderReceiptVisitor receiptVisitor = new OrderReceiptVisitor();
        cart.accept(receiptVisitor);
        System.out.println(receiptVisitor.getReceipt());

        System.out.println("=== 库存检查 ===");
        StockCheckVisitor stockVisitor = new StockCheckVisitor();
        cart.accept(stockVisitor);
    }
}
```
输出：
```
=== 价格计算 ===
图书《设计模式》折后价：80.1
电子产品智能手机含税价：5648.87
电子产品笔记本电脑含税价：9038.87
总价：14767.84

=== 订单小票 ===
==== 订单小票 ====
📚 图书：设计模式（作者：GoF）
🔌 电子产品：智能手机（保修：2年）
🔌 电子产品：笔记本电脑（保修：3年）

=== 库存检查 ===
⚠️ 库存预警：智能手机 仅剩 3 件
```

核心价值：

- Book、Electronic 完全不用关心“怎么算钱、怎么打印、怎么查库存”。
- 新增一种操作（比如“满减计算”“重量统计”）只需新增一个 Visitor 实现类，元素类一行都不用改。
- 访问者可以在遍历过程中累积状态（如总价、小票文本），非常适合聚合计算。

## 优缺点

优点：

- 符合开闭原则（对操作开放）：新增操作只需新增访问者类，不碰元素类。
- 单一职责：把一类操作集中到一个访问者中，元素类只管数据。
- 易于积累状态：访问者可以边遍历边汇总结果（统计、报表、校验）。
- 避免元素类膨胀：否则每个元素都要塞 calcPrice()、printReceipt()、checkStock() 等方法。

缺点：

- 新增元素类型很痛苦：每加一种新商品（如 Clothing），所有访问者接口和实现都要改，违反开闭原则。
- 破坏封装：访问者通常需要读取元素内部状态，元素不得不暴露 getter。
- 理解成本较高：双分派、accept/visit 来回调用，新手容易绕晕。
- 适合面较窄：只在“结构稳定、操作多变”时才划算。


## 适用场景

- 对象结构稳定，但操作经常变化（元素类型不怎么加，但算法/功能频繁加）。
- 需要对一组异构元素做多种不相关操作（统计、导出、校验、渲染）。
- 不想让操作逻辑污染元素类（域模型保持干净）。
- 需要在遍历过程中累积状态（报表、计数、聚合）。

反例：如果商品类型本身经常新增（今天加生鲜、明天加虚拟商品），别用访问者，改用策略+分派更合适。

## 访问者模式 vs 其他模式


|对比项	|访问者模式	|策略模式	|迭代器模式|
|-----|------|-----------|-------------|
|关注点	|对稳定结构做多种操作	|替换整个算法	|遍历集合元素|
|扩展维度	|加操作容易，加元素难	|加算法容易	|加遍历方式容易|
|双分派	|是	|否	|否|
|典型例子	|AST 处理、购物车多维度计算	|折扣策略、排序算法	|List/Set 遍历|

## 实现注意点

- Visitor 接口用重载不用重写：visit(Book) 和 visit(Electronic) 是重载，靠 this 的编译时类型分发。
- 元素类型稳定是前提：如果元素经常变，访问者模式维护成本会爆炸。
- Java 8+ 可用默认方法：在 Visitor 接口给 visit 提供默认实现，缓解新增元素时全量改接口的痛点。
- 访问者可以组合：多个访问者嵌套使用（先算价再生成小票），也可以通过装饰器增强。


## 现实应用

| 领域	 | 示例                                                  |
|-----|-----------------------------------------------------|
| 编译器 |抽象语法树（AST）节点稳定，但要做类型检查、代码生成、优化、格式化——每种操作一个 Visitor。 |
| 文档处理 |文档由标题、段落、图片、表格组成，导出为 Markdown / HTML / PDF 各自是一个访问者。|
| 报表系统|销售数据固定，但要做月度报表、区域对比、利润分析等多种统计视角。|
| 文件系统|文件/目录结构稳定，操作有复制、删除、压缩、杀毒、配额统计。|
| GUI 框架 |组件树遍历：绘制、事件分发、无障碍检查、布局计算。 |
| 企业级中间件 |对请求链路做审计、埋点、权限校验，不同访问者负责不同横切逻辑。|

## 总结：

访问者模式是“数据稳如磐石，操作如流水”场景下的最优解：它把易变的操作从稳定的数据结构中抽出来，用双分派精准路由到对应逻辑。

代价是对“新增元素”不友好，所以选型时要先问自己一句：未来是加操作多，还是加数据类型多？

如果是前者，访问者模式非常值得用；如果是后者，趁早换方案。

记住一句话：“元素只说‘我在这’，访问者决定‘拿你干什么’。”