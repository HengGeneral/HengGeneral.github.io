---
layout: post
title: Java最佳实践之hashCode与equals
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java最佳实践之hashcode和equals"
---

Java中, Object类的hashCode()方法可以返回该实例的hashcode, 这有利于对Map类容器的广泛应用提供支持。
理想情况下, 基于hash散列的容器比list等容器能提供更有效率的插入和查询操作, 因此在java对象中直接支持hash就变得很重要。

### equals
Object类有两种方法可以用来对相应实例进行标识: equals()和hashCode()。
一般来说，如果Override其中一个方法，最好也要Override另一个，因为它们之间至关重要的某种关系必须维持:
如果两个对象是equals，它们的hashCode()方法必须返回相同的值。

Object类的equals()方法和hashCode()方法, 定义如下:

```
    public boolean equals(Object var1) {
        return this == var1;
    }
    
    public native int hashCode();
```

实际上, Object类中的hashCode()返回的是该实例内存地址映射的一个32-bit的整数。
因此, 可能会存在不同的对象返回相同的hashCode。
另外, 如果Override了hashCode()方法, 可以使用System.identityHashCode()方法获取hashCode的默认值。

Object类的equals()方法判断两个对象是否是同一个对象, 这感觉比较严格。在某些场景下, 会希望放宽等价性的定义, 如下:

```
	public boolean equals(Object obj){
		if(obj == null || !(obj instanceof AuthCookie)){
			return false;
		}
		AuthCookie that = (AuthCookie)obj;
		return StringUtils.equals(uid, that.uid);
	}
```

### 为什么需要hashCode()?为什么需要覆盖equals()?

为什么Object类需要hashCode()方法？

hashCode()方法纯粹用于提高效率。
Java平台设计人员预计到了典型Java应用程序中Map相关类的重要性--如Hashtable, HashMap 和 HashSet, 同时在使用 equals() 与集合中的对象进行比较时, 效率低且计算昂贵。
因此, 在Object类中添加hashCode()方法使得所有Java对象都能够运用于Map相关类的容器中。

为什么需要覆盖equals()方法?

如果Integer类不覆盖equals()方法, 情况又将如何? 如果我们从未在HashMap或其它相关的集合中使用Integer作为关键字的话，什么也不会发生。
但是，如果我们在HashMap中使用这类Integer对象作为关键字，我们将不能够可靠地检索相关的值。除非我们在HashMap的get()调用中使用与put()调用时的同一个对象。
不用说，这种方法极不方便, 而且错误频频。

### hashCode()和equals()方法的要求

hashCode()方法的要求:

1. 当在equals()方法中涉及到的变量值没有更改时, 同一对象多次调用hashCode()方法, 须要返回相同的整数值;
2. 当两个对象使用equals()方法等价时, 那么它们的hashCode()也须返回相同的整数值;
3. 不需要两个对象equals()方法后不等价时, hashCode()也返回不同的整数值(当然, 如果能更好)。

equals() 方法的要求:

1. 自反性: 对于任意非null的x, x.equals(x)须返回true;
2. 对称性: 对于任意非null的x和y, 当y.equals(x)返回true, 则x.equals(y)也须返回true;
3. 传递性: 对于任意非null的x,y和z, 当x.equals(y), y.equals(z)都返回true时, 则x.equals(z)也须返回true;
4. 一致性: 对于任意非null的x和y, 当在equals()方法中涉及到的变量值没有更改时, 多次调用x.equals(y)须返回同样的结果;
5. 对于任意非null的x, x.equals(null)须返回false.

### 对象的相等性

上述Object类对equals()和hashCode()的要求规范, 很容易满足。
当决定是否以及如何覆盖equals()方法时, 除了满足上述条件外，还要求其它考虑。

在值不可修改的对象中，如Integer, equals()方法选择就相当明显--相等性应基于基本对象状态的相等性。
对于Integer下，对象的唯一状态是基本的整数值。

对于值可以修改的对象来说，答案并不总是如此清楚。 equals()和 hashCode()是否应基于对象的标识(如默认值)或对象的状态(如Integer和String)？
没有简单的答案--它取决于类以后如何使用。

如果对象的hashCode()值可以基于其状态进行更改,
那么当使用这类对象作为key时必须注意，确保在用于key时，其状态不做更改。
因为所有hash-based的集合类都基于一假设: 当对象的hashCode()用于key时,该值不会改变。

### 参考文献:

1. https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode()
2. https://muhammadkhojaye.blogspot.tw/
3. https://www.ibm.com/developerworks/library/j-jtp05273/
