---
layout: default
title: Interpreter Pattern
parent: Design Patterns
has_children: false
nav_order: 25
---

Here are java Interpreter Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是解释器模式？

解释器模式（Interpreter Pattern）

解释器模式是一种行为型设计模式，它给定一个语言，定义其文法的表示，并定义一个解释器，该解释器使用该表示来解释语言中的句子。

核心思想是：将一个问题分解为抽象语法树（AST）上的表达式，每个表达式节点对应文法中的一个规则，通过递归组合来求解。

> 生活中的类比：翻译官。你说法语（源语言），翻译官按照法语语法规则（文法）解析你的话，然后用英语表达出来（解释执行）。不同的句子结构对应不同的翻译规则。

考虑一个在线客服系统：用户提交一个工单，系统需要根据工单类型（技术问题、账单问题、投诉建议）分配给不同的处理人员。

如果使用硬编码的 if-else 或 switch-case，当处理规则变化时（例如新增一个处理环节），需要修改核心代码，违反开闭原则。

责任链模式将每个处理者封装成一个独立的节点，通过链式结构动态组合，可以灵活地增删处理者或调整顺序。

## 模式结构

```
┌──────────────────┐
│   AbstractExpr   │    （抽象表达式）
│ + interpret(ctx) │
└────────┬─────────┘
         ▲
         │ 继承
┌────────┴─────────┐    ┌──────────────────┐
│ TerminalExpr     │    │ NonterminalExpr  │   （非终结符表达式）
│ + interpret(ctx) │    │ - left, right    │
└──────────────────┘    │ + interpret(ctx) │
                        └──────────────────┘
```

核心角色：

- AbstractExpression（抽象表达式）：声明一个抽象的解释操作，所有表达式节点都实现它。
- TerminalExpression（终结符表达式）：文法中的最小单元，不可再分（如数字、变量名）。
- NonterminalExpression（非终结符表达式）：文法中的组合规则，包含对其他表达式的引用（如加法、乘法）。
- Context（上下文）：包含解释器需要的全局信息（如变量值映射）。
- Client（客户端）：构建抽象语法树，调用解释操作。


  
##  业务案例：自定义规则引擎（促销折扣计算）

假设电商平台需要根据用户条件计算折扣：规则由“与”、“或”、“比较”等表达式组成。例如：

(userLevel == "VIP" && amount > 200) || isNewUser

我们需要解析这样的条件表达式并计算出是否符合折扣条件。

1. 抽象表达式
```java
public interface Expression {
    boolean interpret(Context ctx);
}
```

2. 终结符表达式（变量、常量、比较）
```java
// 变量表达式（从上下文中取值）
public class VariableExpression implements Expression {
    private String name;

    public VariableExpression(String name) {
        this.name = name;
    }

    @Override
    public boolean interpret(Context ctx) {
        return ctx.getVariable(name);
    }
}

// 等于比较（例如 userLevel == "VIP"）
public class EqualsExpression implements Expression {
    private String variableName;
    private String expectedValue;

    public EqualsExpression(String variableName, String expectedValue) {
        this.variableName = variableName;
        this.expectedValue = expectedValue;
    }

    @Override
    public boolean interpret(Context ctx) {
        String actualValue = ctx.getStringVariable(variableName);
        return expectedValue.equals(actualValue);
    }
}

// 大于比较（例如 amount > 200）
public class GreaterThanExpression implements Expression {
    private String variableName;
    private double threshold;

    public GreaterThanExpression(String variableName, double threshold) {
        this.variableName = variableName;
        this.threshold = threshold;
    }

    @Override
    public boolean interpret(Context ctx) {
        double value = ctx.getDoubleVariable(variableName);
        return value > threshold;
    }
}
```

3. 非终结符表达式（逻辑运算）
```java
// 与运算
public class AndExpression implements Expression {
    private Expression left;
    private Expression right;

    public AndExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public boolean interpret(Context ctx) {
        return left.interpret(ctx) && right.interpret(ctx);
    }
}

// 或运算
public class OrExpression implements Expression {
    private Expression left;
    private Expression right;

    public OrExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public boolean interpret(Context ctx) {
        return left.interpret(ctx) || right.interpret(ctx);
    }
}

// 非运算
public class NotExpression implements Expression {
    private Expression expr;

    public NotExpression(Expression expr) {
        this.expr = expr;
    }

    @Override
    public boolean interpret(Context ctx) {
        return !expr.interpret(ctx);
    }
}
```

4. 上下文（保存变量值）
```java
import java.util.HashMap;
import java.util.Map;

public class Context {
    private Map<String, Object> variables = new HashMap<>();

    public void setVariable(String name, Object value) {
        variables.put(name, value);
    }

    public boolean getVariable(String name) {
        Object val = variables.get(name);
        if (val instanceof Boolean) {
            return (Boolean) val;
        }
        throw new RuntimeException("Variable not found or not boolean: " + name);
    }

    public String getStringVariable(String name) {
        return (String) variables.get(name);
    }

    public double getDoubleVariable(String name) {
        return ((Number) variables.get(name)).doubleValue();
    }
}
```
5. 客户端：构建语法树并解释
```java
public class DiscountRuleEngine {
    public static void main(String[] args) {
        // 构建规则：(userLevel == "VIP" && amount > 200) || isNewUser

        // 终结符
        Expression userLevelEqualsVIP = new EqualsExpression("userLevel", "VIP");
        Expression amountGreater200 = new GreaterThanExpression("amount", 200);
        Expression isNewUser = new VariableExpression("isNewUser");

        // 非终结符组合
        Expression vipAndHighAmount = new AndExpression(userLevelEqualsVIP, amountGreater200);
        Expression rule = new OrExpression(vipAndHighAmount, isNewUser);

        // 准备上下文（用户数据）
        Context ctx = new Context();
        ctx.setVariable("userLevel", "VIP");
        ctx.setVariable("amount", 350.0);
        ctx.setVariable("isNewUser", false);

        // 解释执行
        boolean eligible = rule.interpret(ctx);
        System.out.println("是否符合折扣条件：" + eligible);  // true (VIP且金额>200)

        // 另一个用户
        Context ctx2 = new Context();
        ctx2.setVariable("userLevel", "Normal");
        ctx2.setVariable("amount", 50.0);
        ctx2.setVariable("isNewUser", true);
        boolean eligible2 = rule.interpret(ctx2);
        System.out.println("是否符合折扣条件：" + eligible2);  // true (新用户)
    }
}
```
输出：
```
是否符合折扣条件：true
是否符合折扣条件：true
```

核心价值：

- 规则的定义和解释分离，规则本身是对象，可以动态组合、持久化、传输。
- 新增一种运算符（如 XOR）只需新增一个非终结符表达式类。
- 相比硬编码的 if-else，更加灵活、可维护。


## 优缺点

优点：

- 易于改变和扩展文法：新增表达式类即可扩展语言。
- 实现简单：每个文法规则对应一个类，结构清晰。
- 支持复杂语法：通过递归组合可以表示任意复杂的表达式。

缺点：

- 类数量膨胀：每条文法规则对应一个类，文法复杂时类数量剧增。
- 执行效率低：递归解释可能导致大量方法调用，性能较差（尤其在深度嵌套时）。
- 维护困难：对于复杂文法，语法树的构建和维护变得复杂。

## 适用场景

- 需要解释和执行一种简单的语言或表达式，且文法相对固定。
- 文法比较简单，且效率不是首要考虑因素。
- 需要实现自定义的领域特定语言（DSL），如规则引擎、查询语言、配置文件解析。
- 语法树容易构建，且节点数量可控。

不适用：对于复杂文法（如完整的编程语言），应使用专门的解析器生成器（如 ANTLR、JavaCC）而非手写解释器模式。

## 解释器模式 vs 策略模式

|对比项	| 解释器模式	            |策略模式|
|-------|-------------------|----------|
|目的	| 解释语言中的句子（文法）|封装算法，使它们可以互相替换|
|关注点	| 递归树状结构（AST）    |算法的选择和替换|
|状态	| 每个文法规则一个类 |通常是无状态的算法|
|典型例子	| 数学表达式计算 |排序算法、支付策略 |


## 现实应用

|领域	| 示例                                                              |
|-----|-----------------------------------------------------------------|
|正则表达式 |正则表达式引擎内部使用解释器模式解析模式串并匹配文本。    |
|SQL 解析 |许多数据库的 SQL 解析器将 SQL 语句解析为抽象语法树，然后解释执行。 |
|数学表达式计算 |计算器应用（如 2*(3+4)）使用解释器模式解析和求值。 |
|Spring SpEL |Spring 表达式语言（SpEL）内部使用解释器模式解析 #{bean.property} 等表达式。|
|规则引擎 |Drools、EasyRules 等规则引擎将规则（如 when ... then ...）解析为表达式树并执行。 |
|模板引擎 |FreeMarker、Thymeleaf 将模板中的 ${...} 表达式解析为 AST 并解释执行。 |

## 总结：

解释器模式适合文法简单、变化不频繁的 DSL 实现场景。它通过将每条文法规则建模为一个类，利用递归组合来求解问题。

虽然类数量较多且性能不高，但对于特定领域的规则引擎、表达式计算等场景，它提供了极高的灵活性和可扩展性。

记住一句话：“把语言规则写成类，用递归搭成树，解释器沿着树求值。”