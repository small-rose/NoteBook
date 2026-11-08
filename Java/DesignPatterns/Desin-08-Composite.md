---
layout: default
title: Composite Pattern
parent: Design Patterns
has_children: false
nav_order: 18
---

Here are java Composite Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是组合模式？

组合模式（Composite Pattern）

组合模式是一种结构型设计模式，它将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得客户端对单个对象和组合对象的使用具有一致性（即透明性）。

通俗地说：文件夹里有文件和子文件夹，子文件夹里又有文件和子文件夹……无论你操作的是一个文件还是一个文件夹，你都可以用相同的方式去处理它们（比如双击打开、右键删除）。

一个公司组织架构系统：公司有部门，部门下有小组，小组下有员工。你需要计算整个公司的总薪资、总人数，或者遍历所有员工发送通知。

如果不使用组合模式，你需要分别处理“部门”（容器）和“员工”（叶子）， 客户端代码中充斥着 if (obj instanceof Department) ... else if (obj instanceof Employee) ...。

组合模式通过定义一个统一的接口（Component），让容器和叶子都实现它，客户端就可以递归地、一致地处理整个树结构。

## 模式结构

```
┌────────────────────────┐
│      Component         │   （抽象构件）
│ + operation()          │
│ + add(Component)       │   （可选，叶子不支持时抛出异常）
│ + remove(Component)    │
│ + getChild(int)        │
└───────────┬────────────┘
            ▲
            │ 继承
   ┌────────┴─────────┐
   │                  │
┌──┴──────────┐   ┌───┴───────────┐
│   Leaf      │   │   Composite   │   （树枝构件）
│ +operation()│   │ + operation() │
└─────────────┘   │ + add()       │
                  │ + remove()    │
                  │ + getChild()  │
                  └───────────────┘
```

核心角色：

- Component（抽象构件）：为叶子和容器声明公共接口，可以包含默认实现（如添加、删除子节点的方法）。
- Leaf（叶子构件）：叶子节点，没有子节点，实现 operation()。
- Composite（容器构件）：包含子节点的容器，实现 operation() 时会遍历所有子节点并调用它们的 operation()。


  
## 案例1 对象适配器模型

业务场景：公司组织架构与薪资统计

1. 抽象构件
```java
public abstract class OrganizationComponent {
    protected String name;

    public OrganizationComponent(String name) {
        this.name = name;
    }

    // 公共方法
    public abstract double getSalary();       // 获取该节点总薪资
    public abstract void print();             // 打印组织结构

    // 容器特有方法（提供默认实现，叶子继承后会抛异常或空实现）
    public void add(OrganizationComponent component) {
        throw new UnsupportedOperationException("不支持添加子节点");
    }

    public void remove(OrganizationComponent component) {
        throw new UnsupportedOperationException("不支持移除子节点");
    }

    public OrganizationComponent getChild(int index) {
        throw new UnsupportedOperationException("不支持获取子节点");
    }
}
```
2. 叶子构件（员工）
```java
public class Employee extends OrganizationComponent {
    private double salary;

    public Employee(String name, double salary) {
        super(name);
        this.salary = salary;
    }

    @Override
    public double getSalary() {
        return salary;
    }

    @Override
    public void print() {
        System.out.println("  员工：" + name + "，薪资：" + salary);
    }
}
```

3. 容器构件（部门/小组）
```java
import java.util.ArrayList;
import java.util.List;

public class Department extends OrganizationComponent {
    private List<OrganizationComponent> children = new ArrayList<>();

    public Department(String name) {
        super(name);
    }

    @Override
    public void add(OrganizationComponent component) {
        children.add(component);
    }

    @Override
    public void remove(OrganizationComponent component) {
        children.remove(component);
    }

    @Override
    public OrganizationComponent getChild(int index) {
        return children.get(index);
    }

    @Override
    public double getSalary() {
        // 递归计算所有子节点的薪资总和
        double total = 0;
        for (OrganizationComponent child : children) {
            total += child.getSalary();
        }
        return total;
    }

    @Override
    public void print() {
        System.out.println("部门：" + name + "，总薪资：" + getSalary());
        for (OrganizationComponent child : children) {
            child.print();  // 递归打印
        }
    }
}
```

4. 客户端构建树并统一操作
```java
public class CompanyClient {
    public static void main(String[] args) {
        // 构建顶层公司
        Department company = new Department("总公司");

        // 研发部
        Department devDept = new Department("研发部");
        devDept.add(new Employee("张三（开发）", 15000));
        devDept.add(new Employee("李四（测试）", 12000));

        // 研发部下设前端组和后端组
        Department frontendGroup = new Department("前端组");
        frontendGroup.add(new Employee("王五（前端）", 14000));
        frontendGroup.add(new Employee("赵六（前端）", 13000));

        Department backendGroup = new Department("后端组");
        backendGroup.add(new Employee("孙七（后端）", 16000));
        backendGroup.add(new Employee("周八（后端）", 17000));

        devDept.add(frontendGroup);
        devDept.add(backendGroup);

        // 市场部
        Department marketDept = new Department("市场部");
        marketDept.add(new Employee("吴九（市场）", 11000));
        marketDept.add(new Employee("郑十（市场）", 10000));

        // 组装到公司
        company.add(devDept);
        company.add(marketDept);

        // 统一操作：打印整个组织架构
        System.out.println("=== 公司组织架构 ===");
        company.print();

        // 统一操作：计算总薪资
        System.out.println("\n公司总薪资：" + company.getSalary());

        // 也可以只操作某个部门
        System.out.println("\n=== 研发部薪资 ===");
        System.out.println("研发部总薪资：" + devDept.getSalary());
    }
}
```

输出：
```
=== 公司组织架构 ===
部门：总公司，总薪资：125000.0
  部门：研发部，总薪资：105000.0
    员工：张三（开发），薪资：15000.0
    员工：李四（测试），薪资：12000.0
    部门：前端组，总薪资：27000.0
      员工：王五（前端），薪资：14000.0
      员工：赵六（前端），薪资：13000.0
    部门：后端组，总薪资：33000.0
      员工：孙七（后端），薪资：16000.0
      员工：周八（后端），薪资：17000.0
  部门：市场部，总薪资：21000.0
    员工：吴九（市场），薪资：11000.0
    员工：郑十（市场），薪资：10000.0

公司总薪资：125000.0

=== 研发部薪资 ===
研发部总薪资：105000.0
```

>关键点：
> 客户端调用 print() 或 getSalary() 时，完全不需要区分对象是 Employee 还是 Department，统一通过 OrganizationComponent 接口操作。这就是组合模式的威力。

 
##  优缺点

优点：
- 清晰定义层次结构：树形结构直观反映“部分-整体”关系。
- 客户端一致对待：无需区分叶子与容器，简化客户端代码。
- 易于扩展：新增一种叶子或容器类型，只要实现 Component 接口即可，不影响现有代码。
- 递归处理：非常适合递归遍历、求和、查找等操作。

缺点：
- 设计过于泛化：如果系统层次结构不深，强行使用组合模式会增加不必要的复杂度。
- 违反接口隔离原则：透明模式下，叶子类继承了它不需要的管理子节点方法（尽管是空实现或抛异常）。
- 递归性能开销：对于极深的树结构，递归调用可能带来性能问题。

## 两种模式：透明模式 vs 安全模式

|模式	|特点	|优缺点|
|-------|-------|-----|
|透明模式（本例采用）	|在 Component 中声明所有管理子节点的方法（add/remove/getChild），叶子类继承后抛出异常或空实现	|客户端完全透明，但叶子类有不安全的方法（调用会抛异常）|
|安全模式|	Component 只声明公共操作，add/remove 等方法只在 Composite 中定义	|更安全，但客户端需要 instanceof 判断才能调用容器方法，牺牲了透明性|

>实际开发中，透明模式更常用，因为客户端代码更简洁，只需捕获可能的异常或提前检查。

## 适用场景

- 需要表示对象的部分-整体层次结构（如文件系统、组织架构、菜单、UI 组件树）。
- 希望客户端忽略组合对象与单个对象的差异，以统一方式处理。
- 树形结构中的节点可以有子节点，也可以没有，且操作具有递归性质。



## 比较 组合模式 vs 装饰器模式

| 对比项	| 组合模式	| 装饰器模式|
|------|-----------------|-----|
| 目的	| 表示部分-整体层次，统一对待叶子与容器	| 动态地为对象添加额外功能|
| 结构	| 树形结构，容器包含子节点	| 链式结构，装饰器包裹被装饰者|
| 关系	| 容器与叶子是“has-a”关系	| 装饰器与被装饰者是“is-a”关系（通过继承接口）|
| 典型例子	| 文件系统、组织架构	| Java I/O 流（BufferedInputStream 包装 FileInputStream）|



## 实际应用

| 领域   | 应用方式       |
|------|-----------------|
| 日志框架 |SLF4J 是抽象化，Logback、Log4j、JUL 是实现化。用户面向 SLF4J 编程，底层日志实现可以热切换。  |
| JDBC 驱动 | java.sql.DriverManager 是抽象化，不同数据库的 Driver 是实现化。应用程序通过统一的 JDBC 接口操作数据库，底层驱动可以随时切换（MySQL、Oracle、PostgreSQL）。 |
| 图形绘制 | 形状（圆形、矩形）是抽象化，绘制引擎（DirectX、OpenGL、Vulkan）是实现化。两者独立变化。 |
| UI 控件 |窗口（Button、TextField）是抽象化，操作系统渲染（Windows、Mac、Linux）是实现化。  |
| 消息队列 | 消息生产者/消费者是抽象化，底层消息中间件（Kafka、RocketMQ、RabbitMQ）是实现化。     |


## 总结：

组合模式的核心是让叶子对象和容器对象具有相同的接口，从而构建出统一的树形结构。

它特别适合处理递归层次的场景，能让客户端代码变得极其简洁。

记住一句话：“文件夹和文件都能双击打开，这就是组合模式。”