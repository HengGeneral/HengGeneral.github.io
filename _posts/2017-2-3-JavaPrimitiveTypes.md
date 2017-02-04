---
layout: post
title: Java-基本数据类型
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java基本类型"
---
### Java基本类型

A primitive type is predefined by the language and is named by a reserved keyword.
Java基本类型是Java语言预定义的, 并由保留的关键字来命名。
基本类型与引用类型最大的区别, 我认为就是它并不和其它类型的变量并不share states, 即改变某个基本类型变量的值,不会对其他变量造成影响。
Java语言支持8种基本类型, 这8种类型分为三大类: 布尔型(boolean), 字符型(char)和数值型(byte, short, int, long, float, double)。

然后, 这8种基本类型的变量并不是对象, 在jdk1.5之前, 这些基本类型的变量并不能直接运用在对于面向对象的操作中, 需要进行包装成相应的包装类(wrapper class)。
为了简化这个操作, java设计者引入了autoboxing和unboxing。同时, 该操作由编译器自动完成。

这8种基本类型,以及相应的包装类如下:

#### boolean
wrapper class: Boolean
default value: false

#### char
bits: 16 (Character.SIZE)
wrapper class: Character.class
default value: '\u0000'

#### byte
bits: 8 (Byte.SIZE)
wrapper class: Byte.class
default value: 0

#### short
bits: 16 (Short.SIZE)
wrapper class: Short.class
default value: 0

#### int
bits: 32 (Integer.SIZE)
wrapper class: Integer.class
default value: 0

#### long
bits: 64 (Long.SIZE)
wrapper class: Long.class
default value: 0

#### float
bits: 32 (Float.SIZE)
wrapper class: Float.class
default value: 0.0f

#### double
bits: 64 (Double.SIZE)
wrapper class: Double.class
default value: 0.0

包装类的继承关系如下:
![继承关系](/images/java/primitive_types.png)

### 基本类型的转换

Java 语言是一种强类型的语言。强类型的语言有以下几个要求：

1. 声明时必须有类型: 要求声明变量时必须声明类型，且只能在声明以后才能使用;
2. 赋值时类型必须一致: 值的类型必须和变量类型完全一致, 且变量类型不能改变;
3. 运算时类型必须一致: 参与运算的数据类型必须一致才能运算。

然而, 在平时的编程场景中, 可能会存在一些表达式的类型不符合上下文的场景。
这就需要一种新的语法来适应这种需要，这个语法就是数据类型转换。

Java 语言中的数据类型转换有两种：

1. 自动类型转换: 编译器自动完成类型转换，不需要在程序中编写代码。
2. 强制类型转换: 强制编译器进行类型转换，必须在程序中编写代码。

下面来具体介绍两种类型转换的规则、适用场合以及使用时需要注意的问题。

#### 自动类型转换

自动类型转换，也称隐式类型转换，是指不需要书写代码，由系统自动完成的类型转换。由于实际开发中这样的类型转换很多，所以 Java 语言在设计时，没有为该操作设计语法，而是由 JVM 自动完成。
转换规则：从存储范围小的类型到存储范围大的类型。
具体规则为：byte→short(char)→int→long→float→double
也就是说 byte 类型的变量可以自动转换为 short 类型，示例代码：

```
byte b = 10;
short sh = b;
```

这里在给sh赋值时，JVM首先将b的值转换成short类型然后再赋值给sh。
当然，在类型转换的时候也可以跳跃，就是byte也可以自动转换为int类型的。
注意问题：在整数之间进行类型转换的时候数值不会发生变化，但是当将整数类型特别是比较大的整数类型转换成小数类型的时候，由于存储范围和精度的不同，可能
会存在数值发生变化。

#### 强制类型转换

强制类型转换，也称显式类型转换，是指必须书写代码才能完成的类型转换。该类类型转换很可能存在值差异过大或者精度的损失。
转换规则: 从存储范围大的类型到存储范围小的类型。
具体规则: double→float→long→int→short(char)→byte
语法格式: (转换到的类型)需要转换的值, 实例代码如下:

```
double d = 3.14;
int i = (int) d;
```

### 参考文献
1. https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
2. https://www.oschina.net/question/127301_37313
3. http://alexyyek.github.io/2014/12/29/wrapperClass/
4. http://blog.csdn.net/caohaicheng/article/details/38016193
5. https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html
6. http://www.javaworld.com/article/2150208/java-language/a-case-for-keeping-primitives-in-java.html
7. https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.2.3
