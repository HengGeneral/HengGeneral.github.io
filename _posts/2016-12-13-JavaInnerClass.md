---
layout: post
title: Java-内部类
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "内部类"
---

##嵌套类

Java 允许你在一个类内部定义另外一个类, 这样的类称作嵌套类, 如下:
`class OuterClass {
    ...
    class NestedClass {
        ...
    }
}`

---
**专业术语**:嵌套类分为两类: 静态的和非静态的。 通常, 静态的嵌套类被称作静态嵌套类, 非静态的嵌套类称作内部类。
---

`class OuterClass {
    ...
    static class StaticNestedClass {
        ...
    }
    class InnerClass {
        ...
    }
}`
内部类属于包含它的外部类(the enclosing class)的一个成员, 并且具有外部类其它成员的访问权限, 即使这些成员声明为private。静态嵌套类没有访问包含它的外部类成员的访问权限。作为外部类的成员, 嵌套类可以声明为private, public, protected。


##为什么需要试用嵌套类?

比较令人信服的原因如下:
*<strong>程序逻辑上, 该类仅出现在另外一个类的内部</strong>: 如果一个类仅对另外一个类有用, 把它潜入到另外那个类当中就显得合乎情理, 同时也使当前package更加精简。
*<strong>封装</strong>: 给定两个top-level的类, A和B, 其中B需要访问A的成员变量(private)。通过将B放入A内部, 那么B就可以访问A的成员变量, 同时该成员变量也可以声明为private。除此之外, B还可以隐身。
*<strong>可读性和可维护性</strong>: 嵌套类放在使用到该内的地方会让程序可读性更好。

##静态嵌套类

和类的成员一样, 静态内部类属于它的外部类。同时和静态方法一样, 静态嵌套类不可直接使用外部类中的实例变量和实例方法: 该嵌套类仅能通过外部类的实例访问它们。

***
**Note**: 静态内部类与外部类(包括其它类)的实例交互与其它top-level一样的。事实上, 静态内部类表现的行为像是该package下另一个的top-level类。
***

可以通过外部封装的类名访问它的静态嵌套类:
  `OuterClass.StaticNestedClass`

例如, 创建静态嵌套类的实例的方法如下:
	`OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();`

##内部类

和实例的成员(变量和方法)一样, 内部类也属于它的外部类的实例。同时, 又具有该实例的变量和方法的访问权限。又因为该内部类属于外部类的某个实例, 所以它不可定义任何静态的成员。

内部类的某个实例存在于外部类的实例中, 如下:
`   class OuterClass {
        ...
        class InnerClass {
            ...
        }
}`

内部类的实例存在于且仅存在于它的外部类的实例, 同时又具有外部类的实例变量和实例方法的访问权限。
为了实例化某个内部类, 需要首先实例它的外部类。 然后, 使用如下的语法:
    `OuterClass.InnerClass innerObject = outerObject.new InnerClass();`

另外, 有两种比较特别的内部类: 局部内部类(local class) 和 匿名内部类(anonymous classes)。

##遮蔽(Shadowing)
如果内部类的成员变量或成员方法的某个名称 和 外部类的其它某个声明的名称一样, 那么外部类的那个名称将会被遮蔽, 则不可以仅通过该名称访问被遮蔽的声明。如下:

`public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}`

输出为:
	`x = 23
	this.x = 1
	ShadowTest.this.x = 0`

该例子展示了名称为x的三种变量: 外部类 ShadowTest 的成员变量, 内部类 FirstLevel 的成员变量 和 方法 methodInFirstLevel 的参数。methodInFirstLevel 的参数x 遮蔽了 内部类 FirstLevel 的成员变量。因而, 当你使用x的时候, 其实用的x是 methodInFirstLevel 的参数。为了使用内部类 FirstLevel 的成员变量, 需要使用关键词 **this**, 如下:
    `System.out.println("this.x = " + this.x);`
为了使用外部类的成员变量, 需要用到外部类的名称。比如, 可以通过下面的语句在方法 methodInFirstLevel 中访问外部类ShadowTest的成员变量:
    `System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);`


##序列化

内部类(包含局部内部类和匿名内部类)的序列化, 是强烈不鼓励的。当Java compiler编译某些结构,如内部类, 它会合成结构, 而这些结构在源码(classes, methods, fields, and other constructs) 中都不会有相应的构造。以至于Java compiler有一些JVM没有的新语言特性。然而, 这种合成的结构会随着不同的Java compiler而不同, 即.class文件的差异性。因此在不同的JRE环境中, 序列化和反序列化可能会有兼容性问题。内部类编译时合成的结构详情参见于[章节][ISPONMP]。

[ISPONMP]: https://docs.oracle.com/javase/tutorial/reflect/member/methodparameterreflection.html#implcit_and_synthetic