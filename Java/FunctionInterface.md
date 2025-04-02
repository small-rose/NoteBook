---
layout: default
title: Func Interface
parent: Java
nav_order: 92
---

Here are java 8 Functional Interface experience .
{: .fs-6 .fw-300 }


## Here are function interface demo .
{: .no_toc .text-delta }

1. TOC
{:toc}


## Functional Interface

## 什么是函数式接口


什么是Functional Interface


```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface{

}
```

interface做注解的注解类型，被定义成java语言规则
- 一个被它注解的接口只能有一个抽象方法，有两种例外
  - 第一是接口允许有实现的方法，这种实现的方法是用default关键字来标记的(java反射中java.lang.reflect.Method#isDefault()方法用来判断是否是default方法)
  - 第二如果声明的方法和java.lang.Object中的某个方法一样，它可以不当做未实现的方法，不违背这个原则: 一个被它注解的接口只能有一个抽象方法, 比如: java public interface Comparator<T> { int compare(T o1, T o2); boolean equals(Object obj); } 
  
- 如果一个类型被这个注解修饰，那么编译器会要求这个类型必须满足如下条件: 
   - 这个类型必须是一个interface，而不是其他的注解类型、枚举enum或者类class
   - 这个类型必须满足function interface的所有要求，如你个包含两个抽象方法的接口增加这个注解，会有编译错误。
- 编译器会自动把满足function interface要求的接口自动识别为function interface。
 



## 内置函数式接口

内置的函数接口都是 `rt.jar` 中的 `java.util.function` 包下。

常见的有4大类：

消费型接口: Consumer< T> void accept(T t)有参数，无返回值的抽象方法；

```
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

供给型接口: Supplier < T> T get() 无参有返回值的抽象方法；

以stream().collect(Collector<? super T, A, R> collector)为例:比如:
```
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

断定型接口: Predicate<T> boolean test(T t):有参，但是返回值类型是固定的boolean

比如: steam().filter()中参数就是PredicatePredicate<String> predicate = (s) -> s.length() > 0;

```
predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

函数型接口: Function<T,R> R apply(T t)有参有返回值的抽象方法；

比如: steam().map() 中参数就是Function<? super T, ? extends R>；reduce()中参数BinaryOperator<T> (ps: BinaryOperator<T> extends BiFunction<T,T,T>)

```
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123");     // "123"
```

其他接口主要差别是参数类型、参数个数、有无返回值或返回值类型差异。

## 自定义函数式接口

自定义（无参无返回）函数式接口

```java
@FunctionalInterface
public interface IMyInterface {
    void study();
}

public class TestIMyInterface {
    public static void main(String[] args) {
        IMyInterface iMyInterface = () -> System.out.println("I like study");
        iMyInterface.study();
    }
}
``` 

## 函数式接口应用

