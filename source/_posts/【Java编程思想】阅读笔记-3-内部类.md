---
title: 【Java编程思想】阅读笔记(3)--内部类
comments: true
excerpt: 《on Java8》（Thinking in Java第六版）阅读笔记(2)
date: 2020-10-14 16:14:30
tags:
  - Java
categories:
  - 编程
  - 服务端
---

# 第十一章  内部类

内部类是一种非常有用的特性，因为它允许把一些逻辑相关的类组织在一起，并控制位于内部的类的可见性。

## 闭包与回调

```
闭包的概念：闭包（closure）是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。
```

内部类是面向对象的闭包，它不仅包含外部类对象（创建内部类的作用域）的信息，还自动拥有一个指向此外部类对象的引用，在此作用域内，内部类有权操作所有的成员，包括 private 成员。

可以看出内部类是面向对象的闭包，因为它不仅包含外部类对象（创建内部类的作用域）的信息，还自动拥有一个指向此外部类对象的引用，在此作用域内，内部类有权操作所有的成员，包括 private 成员。

```
// innerclasses/Callbacks.java
// Using inner classes for callbacks
// {java innerclasses.Callbacks}
package innerclasses;
interface Incrementable {
    void increment();
}
// Very simple to just implement the interface:
class Callee1 implements Incrementable {
    private int i = 0;
    @Override
    public void increment() {
        i++;
        System.out.println(i);
    }
}
class MyIncrement {
    public void increment() {
        System.out.println("Other operation");
    }
    static void f(MyIncrement mi) { mi.increment(); }
}
// If your class must implement increment() in
// some other way, you must use an inner class:
class Callee2 extends MyIncrement {
    private int i = 0;
    @Override
    public void increment() {
        super.increment();
        i++;
        System.out.println(i);
    }
    private class Closure implements Incrementable {
        @Override
        public void increment() {
            // Specify outer-class method, otherwise
            // you'll get an infinite recursion:
            Callee2.this.increment();
        }
    }
    Incrementable getCallbackReference() {
        return new Closure();
    }
}
class Caller {
    private Incrementable callbackReference;
    Caller(Incrementable cbh) {
        callbackReference = cbh;
    }
    void go() { callbackReference.increment(); }
}
public class Callbacks {
    public static void main(String[] args) {
        Callee1 c1 = new Callee1();
        Callee2 c2 = new Callee2();
        MyIncrement.f(c2);
        Caller caller1 = new Caller(c1);
        Caller caller2 =
                new Caller(c2.getCallbackReference());
        caller1.go();
        caller1.go();
        caller2.go();
        caller2.go();
    }
}
```

输出：

```
Other operation
1
1
2
Other operation
2
Other operation
3
```

就代码而言，Callee1 是更简单的解决方式。Callee2 继承自 MyIncrement，后者已经有了一个不同的 increment() 方法，并且与 Incrementable 接口期望的 increment() 方法完全不相关。所以如果 Callee2 继承了 MyIncrement，就不能为了 Incrementable 的用途而覆盖 increment() 方法，于是只能使用内部类独立地实现 Incrementable。

内部类 Closure 实现了 Incrementable，以提供一个返回 Callee2 的 **“钩子”（hook）** -而且是一个安全的钩子。无论谁获得此 Incrementable 的引用，都只能调用 increment()，除此之外没有其他功能。

**回调的价值在于它的灵活性：可以在运行时动态地决定需要调用什么方法。**

## 内部类与控制框架

应用程序框架（application framework）就是被设计用以解决某类特定问题的一个类或一组类。要运用某个应用程序框架，通常是继承一个或多个类，并覆盖某些方法。在覆盖后的方法中，编写代码定制应用程序框架提供的通用解决方案，以解决你的特定问题。

控制框架是一类特殊的应用程序框架，它用来解决响应事件的需求。主要用来响应事件的系统被称作事件驱动系统。应用程序设计中常见的问题之一是图形用户接口（GUI），它几乎完全是事件驱动的系统。

框架实现选择内部类（比如经典框架：GreenhouseControls），原因：

1. 控制框架的完整实现是由单个的类创建的，从而使得实现的细节被封装了起来。内部类用来表示解决问题所必需的各种不同的 action()。
2. 内部类能够很容易地访问外部类的任意成员，所以可以避免这种实现变得笨拙。如果没有这种能力，代码将变得令人讨厌，以至于你肯定会选择别的方法。

## 内部类标识符

由于编译后每个类都会产生一个**.class** 文件，其中包含了如何创建该类型的对象的全部信息（此信息产生一个"meta-class"，叫做 Class 对象）。

内部类也必须生成一个 **.class** 文件以包含它们的 Class 对象信息。这些类文件的命名有严格的规则：外部类的名字，加上“ **$"，再加上内部类的名字。例如，LocalInnerClass.java** 生成的 .class 文件包括：

```
Counter.class
LocalInnerClass$1.class
LocalInnerClass$LocalCounter.class
LocalInnerClass.class
```

如果内部类是匿名的，编译器会简单地产生一个数字作为其标识符。如果内部类是嵌套在别的内部类之中，只需直接将它们的名字加在其外部类标识符与“ **$** ”的后面。

虽然这种命名格式简单而直接，但它还是很健壮的，足以应对绝大多数情况。因为这是 java 的标准命名方式，所以产生的文件自动都是平台无关的。（注意，为了保证你的内部类能起作用，Java 编译器会尽可能地转换它们。）