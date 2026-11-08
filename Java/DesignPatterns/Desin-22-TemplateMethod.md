---
layout: default
title: Template Method
parent: Design Patterns
has_children: false
nav_order: 32
---

Here are java Template Method Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是模板方法模式？

模板方法模式（Template Method Pattern）

模板方法模式是一种行为型设计模式，它在一个方法中定义了一个算法的骨架，而将一些步骤的实现延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些特定步骤。

核心思想是：“父类定义流程，子类实现细节”——将不变的通用逻辑放在父类的模板方法中，将可变的具体步骤留给子类去覆盖。

>生活中的类比：制作一杯咖啡或茶。基本流程是一样的：烧水 → 冲泡 → 倒入杯子 → 加调料。但具体步骤不同：咖啡是研磨咖啡豆并用热水冲泡，茶是放入茶叶并用热水浸泡；调料方面咖啡加糖和奶，茶加柠檬或蜂蜜。这个“制作热饮”的算法骨架就是模板方法，子类（咖啡、茶）各自实现具体步骤。


一个电商系统的订单价格计算：不同用户等级（普通、VIP、超级VIP）享受不同的折扣，而且节假日可能有满减活动，还有优惠券叠加。

如果直接在订单类中用 if-else 实现所有折扣逻辑，代码会变得臃肿且难以维护。新增一种促销方式就要修改订单类，违反开闭原则。

策略模式将每种折扣算法封装成独立的策略类，订单类只需持有一个策略引用，运行时动态切换。

## 模式结构

```
┌─────────────────────────────┐
│      AbstractClass          │
│ + templateMethod()          │  ← 模板方法（final，定义算法骨架）
│ + step1()                   │  ← 抽象方法（子类必须实现）
│ + step2()                   │  ← 具体方法（可选，可被子类覆盖）
│ + hook()                    │  ← 钩子方法（可选，默认空实现）
└─────────────┬───────────────┘
              ▲
              │ 继承
┌─────────────┴───────────────┐
│      ConcreteClass          │
│ + step1()                   │
│ + step2()                   │
│ + hook()                    │  ← 可选覆盖
└─────────────────────────────┘
```

核心角色：

- AbstractClass（抽象类）：定义模板方法和基本方法（抽象方法、具体方法、钩子方法）。模板方法调用这些基本方法来构成算法骨架。
- ConcreteClass（具体子类）：实现抽象方法，也可覆盖钩子方法，从而改变算法的某些步骤。

基本方法类型：

- 抽象方法：子类必须实现。
- 具体方法：父类已实现，子类可选择覆盖或不覆盖。
- 钩子方法：默认空实现或返回固定值，子类可选择性地覆盖以影响模板方法的执行流程（如在某个步骤前后加入条件判断）。



##  业务案例：数据导出器

假设有一个报表系统，需要支持导出为 PDF、Excel、CSV 三种格式。导出的通用流程是：

(1) 查询数据
(2) 格式化数据
(3) 写入文件
(4) 发送通知

其中“查询数据”和“发送通知”对所有格式都一样，“格式化数据”和“写入文件”因格式而异。我们可以用模板方法模式来设计。

1. 抽象类（定义模板方法）
```java
public abstract class DataExporter {

    // 模板方法：定义导出算法的骨架
    public final void export() {
        // 1. 查询数据（通用）
        List<String> data = fetchData();

        // 2. 格式化数据（子类实现）
        String formatted = formatData(data);

        // 3. 写入文件（子类实现）
        writeToFile(formatted);

        // 4. 钩子方法：是否需要发送通知
        if (needNotification()) {
            sendNotification();
        }
    }

    // 具体方法：查询数据（所有子类共用）
    private List<String> fetchData() {
        System.out.println("从数据库查询原始数据...");
        return Arrays.asList("姓名,年龄,城市", "张三,28,北京", "李四,32,上海");
    }

    // 抽象方法：格式化数据（子类必须实现）
    protected abstract String formatData(List<String> rawData);

    // 抽象方法：写入文件（子类必须实现）
    protected abstract void writeToFile(String content);

    // 钩子方法：是否需要发送通知（默认true，子类可覆盖）
    protected boolean needNotification() {
        return true;
    }

    // 具体方法：发送通知（通用）
    private void sendNotification() {
        System.out.println("发送邮件通知：导出完成");
    }
}
```

2. 具体子类
```java
// PDF 导出器
public class PdfExporter extends DataExporter {
    @Override
    protected String formatData(List<String> rawData) {
        System.out.println("将数据格式化为 PDF 格式...");
        // 模拟PDF格式化
        StringBuilder sb = new StringBuilder();
        sb.append("%%PDF-1.4\n");
        for (String line : rawData) {
            sb.append("(").append(line).append(")\n");
        }
        return sb.toString();
    }

    @Override
    protected void writeToFile(String content) {
        System.out.println("将 PDF 内容写入 report.pdf");
        // 实际写入文件逻辑
    }

    @Override
    protected boolean needNotification() {
        return false;  // PDF导出不需要通知
    }
}

// Excel 导出器
public class ExcelExporter extends DataExporter {
    @Override
    protected String formatData(List<String> rawData) {
        System.out.println("将数据格式化为 Excel 格式...");
        // 模拟Excel格式化（CSV-like）
        StringBuilder sb = new StringBuilder();
        for (String line : rawData) {
            sb.append(line).append("\n");
        }
        return sb.toString();
    }

    @Override
    protected void writeToFile(String content) {
        System.out.println("将 Excel 内容写入 report.xlsx");
    }
}

// CSV 导出器
public class CsvExporter extends DataExporter {
    @Override
    protected String formatData(List<String> rawData) {
        System.out.println("数据已经是 CSV 格式，无需转换");
        StringBuilder sb = new StringBuilder();
        for (String line : rawData) {
            sb.append(line).append("\n");
        }
        return sb.toString();
    }

    @Override
    protected void writeToFile(String content) {
        System.out.println("将 CSV 内容写入 report.csv");
    }
}
```

3. 客户端
```java
public class ExportClient {
    public static void main(String[] args) {
        System.out.println("=== 导出 PDF ===");
        DataExporter pdfExporter = new PdfExporter();
        pdfExporter.export();

        System.out.println("\n=== 导出 Excel ===");
        DataExporter excelExporter = new ExcelExporter();
        excelExporter.export();

        System.out.println("\n=== 导出 CSV ===");
        DataExporter csvExporter = new CsvExporter();
        csvExporter.export();
    }
}
```

输出：
```
=== 导出 PDF ===
从数据库查询原始数据...
将数据格式化为 PDF 格式...
将 PDF 内容写入 report.pdf

=== 导出 Excel ===
从数据库查询原始数据...
将数据格式化为 Excel 格式...
将 Excel 内容写入 report.xlsx
发送邮件通知：导出完成

=== 导出 CSV ===
从数据库查询原始数据...
数据已经是 CSV 格式，无需转换
将 CSV 内容写入 report.csv
发送邮件通知：导出完成
```

核心价值：

- 导出算法的骨架（查询→格式化→写入→通知）在父类中定义且不可修改（final），保证了流程的一致性。
- 子类只需关注自己独有的步骤（格式化和写入），无需重复编写通用逻辑。
- 钩子方法 needNotification() 让 PDF 导出器可以跳过通知步骤，体现了模板方法的灵活性。

## 优缺点

优点：

- 复用代码：将不变的部分放在父类，避免子类重复编写。
- 符合开闭原则：新增导出格式只需新建子类，无需修改父类或已有子类。
- 控制反转：父类调用子类的方法（好莱坞原则：“别打电话给我们，我们会打给你”），子类只能实现细节，不能改变整体流程。
- 提供扩展点：钩子方法让子类可以有选择地干预流程。

缺点：

- 继承带来的限制：子类必须继承父类，如果父类有变化会影响所有子类（继承的脆弱性）。
- 增加类的数量：每个具体实现都需要一个子类。
- 模板方法可能变得庞大：如果算法步骤很多，模板方法会很长，难以阅读。

## 适用场景

- 多个子类有相同的算法骨架，但某些步骤的实现不同。
- 希望控制子类扩展，防止子类改变算法整体结构（将模板方法声明为 final）。
- 需要提取公共行为到父类，避免代码重复。
- 典型的应用：框架中的回调方法、JUnit 测试生命周期、Spring 中的 JdbcTemplate。

## 模板方法模式 vs 策略模式

|对比项	|模板方法模式	|策略模式|
|-------|-----------|-------|
|关系	|继承（父类定义骨架，子类实现步骤）	|组合（上下文持有策略对象，委托执行）|
|复用粒度	|算法整体结构复用	|单个算法复用|
|变化点	|算法中的某些步骤	|整个算法|
|控制权	|父类控制流程（好莱坞原则）	|客户端控制策略选择|
|典型例子	|JUnit 生命周期	|Collections.sort() 的比较器|


## 模板方法模式 vs 工厂方法模式

- 工厂方法模式：专注于创建对象，子类决定创建哪个具体类。
- 模板方法模式：专注于算法流程，子类决定如何实现某些步骤。

> 两者可以结合使用：模板方法中的某个步骤可以通过工厂方法来创建所需的对象。

## 现实应用

| 领域	 | 示例                                                  |
|------|-----------------------------------------------------|
| Java 集合排序 |Collections.sort(list, comparator) 中的 Comparator 就是策略模式，不同的比较策略决定排序结果。 |
| Spring ResourceLoader |根据资源路径前缀（classpath:/、file:/、http://）选择不同的资源加载策略。|
| 支付系统|不同支付方式（支付宝、微信、银联）封装为策略，客户端选择支付策略。|
| 压缩工具|ZIP、GZIP、7z 等不同压缩算法封装为策略。|
| 验证框架 |Hibernate Validator 中的校验器（如 @NotNull、@Size）可看作策略。 |
| 路由算法 |导航软件根据路况、距离、时间选择不同的路线规划策略。 |

## 总结：

模板方法模式是“框架思维”的典型体现——父类写好剧本，子类扮演角色。它通过继承实现了代码复用和流程控制，在框架设计中无处不在。

当你发现多个类有相似的执行流程但细节不同时，就可以考虑使用模板方法模式。

记住一句话：“流程我来定，细节你说了算。”