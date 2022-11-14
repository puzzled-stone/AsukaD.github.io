---
title: 为什么Lambda表达式中的变量需要是Final或是Effectively Final
date: 2022-10-04 23:10:32
tags: 
    - Java
    - Lambda
---


#### 1.什么是lambda表达式？

我们先来看一下代码

```java
public class Anno {

    public static Function<Integer, Integer> incrementer(final int step) {
        return (Integer i) -> i + step;
    }

    public static Function<Integer, Integer> annoIncrementer(final int step) {
        return new Function<Integer, Integer>() {
            @Override
            public Integer apply(Integer i) {
                return i + step;
            }
        };
    }

}
```

注意到，Anno类中提供了两个方法incrementer和annoIncrementer，两个方法是等效的。这里我们可以认为lambda其实是一种匿名类。

而我们创建lambda表达式其实就是创建了一个实现Function接口的类的实例，参考以下代码输出：

```java
public static void main(String[] args) {
        Function<Integer, Integer> inc = Anno.incrementer(1);
        log.info(inc.getClass().getName());
        log.info(inc.getClass().getSuperclass().getName());
        log.info("{}", inc.getClass().getInterfaces().length);
        log.info("{}", inc.getClass().getInterfaces()[0]);
}
```

现在，回想一下，Java创建对象是存放在哪里的？没错，堆上（暂不需考虑逃逸分析高级特性）。

#### 2.实例变量&成员变量&局部变量

Java中变量有三种，实例变量、成员变量和局部变量。

那么这三种变量有什么区别呢？从存放位置思考一下🤔

> 没错，实例变量和成员变量都是存放在堆上，只有局部变量存放在栈中。

#### 3.捕获lambda中的变量

现在，对于这段代码当lambda表达式返回之后，还能继续使用吗？

对于实例变量和成员变量，同样和lambda表达式的实例存放在堆上，自然没有问题。而对于局部变量，方法结束，栈中的数据会被清理。此时，lambda实例要如何继续使用局部变量呢？

```java
public static Function<Integer, Integer> incrementer(final int step) {
        return (Integer i) -> i + step;
}
```

Java中采用的方式为复制。

那么，即然是复制，如果变量一直在改变。我们复制到的值还正确吗？

答案是否定的。JVM复制的值是一个冻结值，是无法变化的。所以，这里就必须要求局部变量必须是final或effectively final（只赋值一次）的。



---

参考文档：

https://www.baeldung.com/java-lambda-effectively-final-local-variables

https://www.javacodegeeks.com/2021/12/lambda-and-final-variables.html