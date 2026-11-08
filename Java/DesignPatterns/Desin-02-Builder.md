---
layout: default
title: Builder Pattern
parent: Design Patterns
has_children: false
nav_order: 12
---

Here are java  Builder Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是建造者模式？

建造者模式（Builder Pattern）

建造者模式也叫生成器模式。

建造者模式是一种创建型设计模式，它将一个复杂对象的构造过程与它的表示分离，使得同样的构建过程可以创建不同的表示。

> 当创建一个对象需要很多步骤（或很多参数）时，建造者模式让你一步步地构造，最后一步才产出成品，并且可以灵活组合。

建造者模式：
一个模式

核心角色：
- Product 产品角色：一个具体的产品对象。
- Builder 抽象建造者： 创建一个Product 对象的各个部件指定的接口/ 抽象类
- ConcreateBuilder : 具体的建造者，实现接口，构造和装配各个配件
- Director 指挥者 ： 构建一个使用 Builder 接口的对象，主要是创建一个复杂对象，
  - 作用（1）隔离客户对对象的生产过程。
  - 作用（2）复杂控制产品对象的生产过程。

 
## 案例1-买电脑

组装电脑，不同用途的电脑cpu和gpu不同，如打游戏需要好的gpu，办公电脑可以不需要gpu。

```java
public class Computer {
    private String cpu;
    private String gpu;
    private int ram;       // GB
    private int storage;   // GB
    private boolean hasBluetooth;

    // 私有构造器，只有 Builder 能调用
    private Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.gpu = builder.gpu;
        this.ram = builder.ram;
        this.storage = builder.storage;
        this.hasBluetooth = builder.hasBluetooth;
    }

    // 静态内部 Builder
    public static class Builder {
        private String cpu;           // 必填
        private String gpu;           // 可选
        private int ram = 8;          // 默认值
        private int storage = 256;    // 默认值
        private boolean hasBluetooth = false;

        public Builder(String cpu) {  // 必填参数通过构造器传入
            this.cpu = cpu;
        }

        public Builder gpu(String val) { this.gpu = val; return this; }
        public Builder ram(int val) { this.ram = val; return this; }
        public Builder storage(int val) { this.storage = val; return this; }
        public Builder bluetooth(boolean val) { this.hasBluetooth = val; return this; }

        public Computer build() {
            // 可以在 build() 中做参数校验
            if (cpu == null) throw new IllegalStateException("CPU must not be null");
            return new Computer(this);
        }
    }
}
```
客户端调用：

```java
Computer gamingPC = new Computer.Builder("Intel i9")
        .gpu("RTX 4090")
        .ram(32)
        .storage(1024)
        .bluetooth(true)
        .build();

Computer officePC = new Computer.Builder("Intel i5")
        .ram(16)
        .build();   // 只设置了必填和部分可选
```
优点一目了然：

- 链式调用，可读性强
- 参数有默认值，可选参数不强制
- 对象构造完成后不可变（没有 setter）
- 构建时可以做参数校验

## 案例2 - 建房子

房子：

```java
package com.xiaocai.designs.builder;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/10/21 11:42
 * @version: v1.0
 */
public class ProductHouse {

    // 产品的公共属性
    private String houseBasic ;//地基
    private String houseWalls ;//墙面
    private String houseWindos ;//窗户
    private String houseRoofed ;//屋顶


    public String getHouseBasic() {
        return houseBasic;
    }

    public void setHouseBasic(String houseBasic) {
        this.houseBasic = houseBasic;
    }

    public String getHouseWalls() {
        return houseWalls;
    }

    public void setHouseWalls(String houseWalls) {
        this.houseWalls = houseWalls;
    }

    public String getHouseWindos() {
        return houseWindos;
    }

    public void setHouseWindos(String houseWindos) {
        this.houseWindos = houseWindos;
    }

    public String getHouseRoofed() {
        return houseRoofed;
    }

    public void setHouseRoofed(String houseRoofed) {
        this.houseRoofed = houseRoofed;
    }

    @Override
    public String toString() {
        return "ProductHouse{" +
                "houseBasic='" + houseBasic + '\'' +
                ", houseWalls='" + houseWalls + '\'' +
                ", houseWindos='" + houseWindos + '\'' +
                ", houseRoofed='" + houseRoofed + '\'' +
                '}';
    }
}
```
包工头：
```java
package com.xiaocai.designs.builder;

/**
 * @description: TODO 功能角色说明： 指挥者
 * TODO 描述：  可以根据需要自己定义建造流程。
 * @author: 张小菜
 * @date: 2020/10/21 13:18
 * @version: v1.0
 */
public class HouseDirector {

    private HouseBuilder houseBuilder = null ;

    public HouseDirector(HouseBuilder houseBuilder) {
        this.houseBuilder = houseBuilder;
    }

    public HouseBuilder getHouseBuilder() {
        return houseBuilder;
    }

    public void setHouseBuilder(HouseBuilder houseBuilder) {
        this.houseBuilder = houseBuilder;
    }

    //TODO  真正建房子的步骤交给指挥者
    public ProductHouse constructHouse(){
        houseBuilder.buildBasic();
        houseBuilder.buildWalls();
        houseBuilder.buildRoofed();
        houseBuilder.buildWindows();
        return houseBuilder.build();
    }
}
```

建房子作业流程:
```java
package com.xiaocai.designs.builder;

/**
 * @description: TODO 功能角色说明： 建造者接口或抽象类
 * TODO 描述： 本类为 建造者抽象类,需要组合 House
 * @author: 张小菜
 * @date: 2020/10/21 12:47
 * @version: v1.0
 */
public abstract class HouseBuilder {

    //TODO  将产品组合进抽象类
    protected  ProductHouse productHouse = new ProductHouse();

    // 建造房子的流程
    abstract void buildBasic(); // 打地基
    abstract void buildWalls(); // 建墙面
    abstract void buildWindows(); // 安装窗户
    abstract void buildRoofed();  //  盖顶


    protected ProductHouse build() {
       return productHouse ;
    }
}
```
高级房子：
```java
package com.xiaocai.designs.builder;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：具体的建造者实现，本类代表高级房子建造者实现
 * @author: 张小菜
 * @date: 2020/10/21 13:05
 * @version: v1.0
 */
public class HighBuilding extends HouseBuilder{
    @Override
    void buildBasic() {
        productHouse.setHouseBasic("20米");
        System.out.println("高级房子打地基："+productHouse.getHouseBasic());
    }

    @Override
    void buildWalls() {
        productHouse.setHouseWalls("20cm");
        System.out.println("高级房子墙面厚度："+productHouse.getHouseWalls());
    }

    @Override
    void buildWindows() {
        productHouse.setHouseWindos("智能百叶窗");
        System.out.println("高级房子安装窗户是："+productHouse.getHouseWindos());
    }

    @Override
    void buildRoofed() {
        productHouse.setHouseRoofed("防晒玻璃瓦");
        System.out.println("高级房子建房顶："+productHouse.getHouseRoofed());
    }
}
```
普通房子：
```java
package com.xiaocai.designs.builder;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述： 具体的建造者实现，本类代表普通房子建造者实现。 产品在抽象类中，建造过程和 产品解耦。
 * @author: 张小菜
 * @date: 2020/10/21 13:05
 * @version: v1.0
 */
public class CommonBuilding extends HouseBuilder{

    @Override
    void buildBasic() {
        productHouse.setHouseBasic("5米");
        System.out.println("普通房子打地基："+productHouse.getHouseBasic());
    }

    @Override
    void buildWalls() {
        productHouse.setHouseWalls("10cm");
        System.out.println("普通房子墙面厚度："+productHouse.getHouseWalls());

    }

    @Override
    void buildWindows() {
        productHouse.setHouseWindos("推拉窗");
        System.out.println("普通房子安装窗户是："+productHouse.getHouseWindos());
    }

    @Override
    void buildRoofed() {
        productHouse.setHouseRoofed("盖瓦");
        System.out.println("普通房子建房顶："+productHouse.getHouseRoofed());

    }

    // 普通房子的建造流程
    @Override
    public ProductHouse build() {
        return productHouse ;
    }
}
```

开建：
```java
package com.xiaocai.designs.builder;

/**
 * @description: TODO 功能角色说明：
 * TODO 描述：
 * @author: 张小菜
 * @date: 2020/10/21 13:21
 * @version: v1.0
 */
public class Client {

    public static void main(String[] args) {

        // 普通房子
        CommonBuilding commonBuilding = new CommonBuilding();
        // 创建指挥者
        HouseDirector houseDirector = new HouseDirector(commonBuilding);
        // 完成 盖房子
        ProductHouse productHouse = houseDirector.constructHouse();
        System.out.println(productHouse);

        System.out.println("------------------------");

        // 高级房子
        HighBuilding highBuilding = new HighBuilding();
        // 重置建造者（实现）
        houseDirector.setHouseBuilder(highBuilding);
        // 完成 盖房子
        ProductHouse productHouse02 = houseDirector.constructHouse();
        System.out.println(productHouse02);
        
    }
}
```
可以看出，建房子的流程是标准的，但是建什么样的房子可以实现HouseBuilder接口进行组合定制。


## 建造者模式的变体：Lombok @Builder

在实际项目中，最常用的其实是 Lombok 的 @Builder 注解，自动生成上述样板代码：

```java
import lombok.Builder;
import lombok.ToString;

@Builder
@ToString
public class User {
    private String username;   // 必填
    private String email;
    private int age;
    private String phone;
}

// 使用
User user = User.builder()
        .username("admin")
        .email("admin@example.com")
        .age(30)
        .build();
```
> 注意：Lombok 的 @Builder 会把所有字段设为可选，如果需要某些字段必填，可以结合 @NonNull 或在 build() 中手动校验。



## 适用场景

| 场景           | 说明                                        |
|--------------|-------------------------------------------|
| 对象有大量可选参数    | 构造器参数过多，setter 模式破坏不变性                    |
| 构造过程需要分步骤    | 例如案例2建房子，如组装披萨：先加面团 → 加酱料 → 加芝士 → 加配料     |
| 需要生成不同表示的对象  | 同一个构建过程，不同的 ConcreteBuilder 产出不同的 Product |
| 希望对象创建后不可变   | 不暴露 setter，所有属性通过 Builder 在构造时确定          |


## 与工厂模式的区别

| 模式 |  关注点 | 典型场景 |
|-----|---------|--------|
| 简单工厂 / 工厂方法 | 创建哪一种产品（类型选择） |根据参数返回不同子类对象|
| 建造者  |如何一步步创建一个复杂对象（组装过程）| 对象有很多可选部件，构造步骤固定|

> 两者不互斥，可以结合使用：工厂决定用哪个 Builder，Builder 负责具体组装。

## 核心价值

建造者模式的核心价值在于：

- 分离复杂对象的构造与表示，让客户端代码简洁。
- 支持不可变对象，线程安全。
- 参数校验集中化，错误尽早暴露。
- 链式调用，代码流畅自然。


在日常 Java 开发中，遇到参数超过 4 个的构造器，或者需要构建一个多步骤、多配置的对象时，第一时间想到建造者模式。