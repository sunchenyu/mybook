# Lambda表达式和函数式编程

## 概念

什么是Lambda表达式

Lambda表达式，也可以称作闭包，它是推动Jdk8发布的最重要的特性。

在此之前，jdk是不允许将函数作为参数进行传递的，即不支持函数式编程。这使其在很多地方，代码过于啰嗦。因此，在Jdk8引入Lambda后，代码变得更加简洁，更加清晰了。

## 基本语法

基本语法: (parameters) -> expression 或 (parameters) ->{ statements; }

完整的Lambda表达式包含三部分组成：参数列表，箭头，声明语句。

```
可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。
```

## 函数式接口

要想使用Lambda，必然需要一个函数式接口。

那么，什么是函数式接口呢？

函数式接口，Functional Interface，必须有且仅有一个抽象方法的接口，对默认方法的个数没有限制（Jdk8 允许在接口中有默认实现），同时函数式接口用@FunctionalInterface来标注。函数式接口可以友好的支持Lambda。

## 常见示例

比如Runnable接口，可以进行如下定义

```
public class F {
    public static void main(String[] args) throws Exception {
        Runnable runnable = () -> {
            System.out.println("打印数据");
        };
    }
}
```

示例

```
import java.util.function.BiPredicate;

public class F {
    public static void main(String[] args) throws Exception {
        BiPredicate<String, String> biPredicate = (a, b) -> {
            return Integer.parseInt(a) > Integer.parseInt(b);
        };
    }
}
```

## Jdk8 自带的函数式接口

位于java.util.function包下

整理如下

四大核心函数式接口

| 函数式接口           | 方法                | 参数类型 | 返回类型    | 作用                        |
| --------------- | ----------------- | ---- | ------- | ------------------------- |
| Consumer 消费型接口  | void accept(T t)  | T    | void    | 对T类型的参数进行操作，无返回值          |
| Supplier 供给型接口  | T get()           | 无    | T       | 无参，返回T类型的结果               |
| Function 函数型接口  | R apply(T t)      | T    | R       | 对T类型参数进行操作，并返回R类型的结果      |
| Predicate 断言型接口 | boolean test(T t) | T    | boolean | 确定T类型参数是否满足某约束，并返回boolean |

其他函数式接口

| 函数式接口                                         | 方法                                       | 参数类型              | 返回类型          | 作用                         |
| --------------------------------------------- | ---------------------------------------- | ----------------- | ------------- | -------------------------- |
| BiFunction                                    | R apply(T t, U u)                        | T, U              | R             | 对 T，U 类型的参数进行操作，并返回R类型的结果  |
| UnaryOperator (Function 子接口)                  | T apply(T t)                             | T                 | T             | 对 T类型的参数进行一元运算，并返回R对象的结果   |
| BinaryOperato (BiFunction 子接口) )              | T apply(T t1, T t2                       | T, T              | T             | 对T类型的参数进行二元运算，并返回T类型的结果    |
| BiConsumer                                    | void accept(T t, U u)                    | T, U              | void          | 对T，U参数执行操作，无返回值            |
| ToIntFunction ToLongFunction ToDoubleFunction | int(long、double) applyAsDouble(T value); | T                 | intlongdouble | 计算 int 、 long 、double值的函数  |
| IntFunction LongFunction DoubleFunction       | R apply(int(long,double) value)          | int, long, double | R             | 参数分别为int、long、double 类型的函数 |

