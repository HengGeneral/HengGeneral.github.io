---
layout: post
title: Java-基本数据类型
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java基本类型"
---
### Java基本类型

Java基本类型是Java语言预定义的, 并由保留的关键字来命名。
基本类型与引用类型最大的区别, 我认为就是它并不和其它类型的变量并不share states, 即改变某个基本类型变量的值,不会对其他变量造成影响。
Java语言支持8种基本类型, 这8种类型分为三大类: 布尔型(boolean), 字符型(char)和数值型(byte, short, int, long, float, double)。

然而, 这8种基本类型的变量并不是对象, 在jdk1.5之前, 这些基本类型的变量并不能直接运用在对于面向对象的操作中, 需要进行包装成相应的包装类(wrapper class)。
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


***

**注意**: 当基本类型的变量作为函数局部变量使用时, 编译器不会对其赋予默认值。此时, 若没有赋予值就使用的话, 将会出现编译时错误。

***

### 字符

#### 整型字符(Integer Literals)

以字符L或者l结尾的整型字符为long literals, 否则则为int literals。

byte, short, int, long型变量的值可以根据int literals创建; 当long型变量的值超过了int的范围, 则需要使用long literals创建。

#### 浮点字符(Floating-Point Literals)

以字符F或者f结尾的浮点型字符为float literals, 否则则为double literals 其中包括以D或者d结尾的。

#### 下划线(Underscore)
在jdk1.7及其后的版本中, 任意数量的下划线(_)可以出现在数值常量的各个位之间。这种特性, 可以用于数字的分组以提高代码的可读性。

```
long creditCardNumber = 1234_5678_9012_3456L;
long socialSecurityNumber = 999_99_9999L;
float pi =  3.14_15F;
long hexBytes = 0xFF_EC_DE_5E;
long hexWords = 0xCAFE_BABE;
long maxLong = 0x7fff_ffff_ffff_ffffL;
byte nybbles = 0b0010_0101;
long bytes = 0b11010010_01101001_10010100_10010010;
int intNumber = 123__456;
```

只能将下划线放置在数字之间, 下列地方则不可放置:

1. number的起始处;
2. 小数点前后;
3. F/f/D/d这类后缀之前;

例子如下:

```
// Invalid: cannot put underscores
// adjacent to a decimal point
float pi1 = 3_.1415F;
// Invalid: cannot put underscores 
// adjacent to a decimal point
float pi2 = 3._1415F;
// Invalid: cannot put underscores 
// prior to an L suffix
long socialSecurityNumber1 = 999_99_9999_L;

// OK (decimal literal)
int x1 = 5_2;
// Invalid: cannot put underscores
// At the end of a literal
int x2 = 52_;
// OK (decimal literal)
int x3 = 5_______2;

// Invalid: cannot put underscores
// in the 0x radix prefix
int x4 = 0_x52;
// Invalid: cannot put underscores
// at the beginning of a number
int x5 = 0x_52;
// OK (hexadecimal literal)
int x6 = 0x5_2; 
// Invalid: cannot put underscores
// at the end of a number
int x7 = 0x52_;
```

### 基本类型的转换

Java 语言是一种强类型的语言。强类型的语言有以下几个要求：

1. 声明时必须有类型: 要求声明变量时必须声明类型，且只能在声明以后才能使用;
2. 赋值时类型必须一致: 值的类型必须和变量类型完全一致, 且变量类型不能改变;
3. 运算时类型必须一致: 参与运算的数据类型必须一致才能运算。

然而, 在平时的编程场景中, 可能会存在一些表达式的类型不符合上下文的场景。
这就需要一种新的语法来适应这种需要，这个语法就是数据类型转换。

Java 语言中的基本数据类型转换分为以下两种：

1. 自动类型转换([Widening Primitive Conversion][Widening_Primitive_Conversion]): 编译器自动完成类型转换，不需要在程序中编写代码。
2. 强制类型转换([Narrowing Primitive Conversion][Narrowing_Primitive_Conversion]): 强制编译器进行类型转换，必须在程序中编写代码。

#### 自动类型转换

自动类型转换，也称隐式类型转换，是指不需要书写代码，由系统自动完成的类型转换。由于实际开发中这样的类型转换很多，所以 Java 语言在设计时，没有为该操作设计语法，而是由 JVM 自动完成。
转换规则：从存储范围小的类型到存储范围大的类型。
具体规则为：byte → short(char) → int → long → float → double
也就是说 byte 类型的变量可以自动转换为 short 类型，示例代码：

```
byte b = 10;
short sh = b;
```

这里在给sh赋值时，JVM首先将b的值转换成short类型然后再赋值给sh。
当然，在类型转换的时候也可以跳跃，就是byte也可以自动转换为int类型的。

#### 强制类型转换

强制类型转换，也称显式类型转换，是指必须书写代码才能完成的类型转换。该类类型转换很可能存在值差异过大或者精度的损失。
转换规则: 从存储范围大的类型到存储范围小的类型。
具体规则: double → float → long → int → short(char) → byte
语法格式: (转换到的类型)需要转换的值, 实例代码如下:

```
double d = 3.14;
int i = (int) d;
```

[Widening_Primitive_Conversion]:http://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.2
[Narrowing_Primitive_Conversion]:http://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.3

### 参考文献
1. https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
2. https://www.oschina.net/question/127301_37313
3. http://alexyyek.github.io/2014/12/29/wrapperClass/
4. http://blog.csdn.net/caohaicheng/article/details/38016193
5. https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html
6. http://www.javaworld.com/article/2150208/java-language/a-case-for-keeping-primitives-in-java.html
7. https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.2.3
