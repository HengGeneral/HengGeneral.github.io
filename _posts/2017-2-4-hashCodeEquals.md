---
layout: post
title: Java最佳实践之hashCode与equals
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java最佳实践之hashcode和equals"
---

Java中, Object类的hashCode()方法可以返回该实例的hashcode, 这有利于对Map类容器的广泛应用提供支持。
理想情况下, 基于hash-based的容器比list等容器能提供更有效率的插入和查询操作, 因此在java对象中直接支持hash就变得很重要。

### equals
Object类有两种方法可以用来对相应实例进行标识: equals()和hashCode()。
一般来说，如果Override其中一个方法，最好也要Override另一个，因为它们之间至关重要的某种关系必须维持:
**如果两个对象是equals，它们的hashCode()方法必须返回相同的值。**

为什么equals()和hashCode()需要遵守上述约定呢?

假如我们违背上述约定, 就会有这样的问题: 假如两个对象equals, 但hashCode()方法返回不同的值, 会有什么影响呢? 我们来看一个例子, 如下:

```
import com.alibaba.fastjson.JSON;

import java.util.HashSet;
import java.util.Set;

public class User {
    private Integer id;

    private String name;

    public User() {

    }

    public User(Integer id, String name) {
        this.id = id;
        this.name = name;
    }

    public Integer getId() {
        return id;
    }

    public User setId(Integer id) {
        this.id = id;
        return this;
    }

    public String getName() {
        return name;
    }

    public User setName(String name) {
        this.name = name;
        return this;
    }

    @Override
    public boolean equals(Object user) {
        if (user == null) return false;
        if (user == this) return true;
        if (user instanceof User) {
            return ((User) user).getId().equals(this.getId());
        } else {
            return false;
        }
    }

    @Override
    public int hashCode() {
        return name.hashCode();
    }

    public static void main(String[] args) {
        User user1 = new User(1, "liheng");
        User user2 = new User(1, "lihengAlias");
        
        //output: true
        System.out.println(user1.equals(user2));

        Set<User> userSet = new HashSet<>();
        userSet.add(user1);
        userSet.add(user2);
        
        //output: [{"name":"liheng","id":1},{"name":"lihengAlias","id":1}]
        System.out.println(JSON.toJSON(userSet));
    }
}
```

从上述打印结果可以看出, user1和user2是等价的(user1和user2是同一用户, 只不过user2名称用了别名)。
当放入hashSet的时候, 还是把他们当成了不同的用户, 这与hash-based的容器的初衷不相符合, hashCode只是为了快速插入和检索, 不应该对插入和检索的结果产生影响。

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
Java平台设计人员预计到了典型Java应用程序中Map相关类的重要性--如Hashtable, HashMap 和 HashSet, 同时在使用 equals() 与集合中的对象逐一进行比较时, 效率低且计算昂贵。
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

### 如何生成一个合适的hashCode()方法

一个好的哈希函数对于不等的对象趋向于产生不等的哈希值。
理想情况下，一个哈希函数应该将实例集合，均匀地散列在所有可能的哈希值上。
要取得这样的目标是非常困难的。幸运的是不难取得一个公平的近似。下面是简单的流程：

```
/*
 * step 1使用的非零初始值会对step 2的计算的hash值有影响。
 * 若设为0, 全部的hash值都不会受到初始值得影响, 容易造成hash碰撞 
 */
Step 1. 存储一些非零常量值，例如17，存储在变量名为result的int变量中; 

Step 2. 对于对象中每一个有意义的字段f（equals方法考虑的每个字段），按以下做法去做：

    a. 为这个字段计算一个int型的哈希码c：
        i. 如果这个字段是一个boolean，计算(f ? 1 : 0);
        ii. 如果这个字段是一个byte，char，short或int，计算(int) f;
        iii. 如果这个字段是一个long，计算(int)(f^(f>>>32));
        iv. 如果这个字段是一个float，计算Float.floatToIntBits(f);
        v. 如果这个字段是一个double，计算Double.doubleToLongBits(f)，然后对结果long进行2.a.iii处理;
        vi. 如果这个字段是一个对象引用并且这个类的equals方法通过递归调用equals方法来比较这个字段，
             那么对这个字段递归的调用hashCode方法。如果需要更复杂的比较，为这个字段计算一个“标准
             表示”然后在标准表示上调用hashCode方法。如果字段值为null，返回0(或一些其它常量，但0是传统表示);
        vii. 如果字段是一个数组，将它每一个元素看做是一个单独的字段。也就是说，通过递归的应用这些规则为每一个
              有效元素计算一个哈希值，并结合这些值对每一个用步骤2.b处理。如果数组的每个元素都是有意义的，
              你可以用JDK 1.5中的Arrays.hashCode方法。
        
    /*
     * 该步骤中乘积使结果依赖于字段的顺序，如果这个类有多个相似的字段会取得一个更好的哈希函数。
     * 若String哈希函数忽略了乘积，所有的anagram将有相同的哈希码。
     * 同时, 选择值31是因为它是一个奇素数。如果它是偶数并且乘积溢出，会损失信息，因为与2想乘等价于位移运算。
     * 使用一个素数的优势不是那么明显，但习惯上都使用素数。
     * 31的一个很好的特性是乘积可以用位移和减法运算替换, 从而取得更好的性能：31 * i == (i << 5) - i。
     * 现代的虚拟机能自动进行排序的优化
     */
    b. 结合步骤2.a计算的哈希码c得到结果如下：result = 31 * result + c:
       i. 返回结果;
       ii. 当你完成了hashCode方法的编写后，问一下自己相等的对象是否有相同的哈希码。写单元测试来验证你的直觉！如果相等的实例有不等的哈希码弄明白为什么并修正这个问题。
 
```

hashCode()方法[例子][hashCode1], 如下：

```
    @Override public int hashCode() {
        int result = 17;
        result = 31 * result + areaCode;
        result = 31 * result + prefix;
        result = 31 * result + lineNumber;
        return result;
    }
```

Integer类的hashCode()方法, 如下:

```
    public static int hashCode(int value) {
        return value;
    }
```

String类的hashCode()方法, 如下:

```
    public int hashCode() {
        int var1 = this.hash;
        if(var1 == 0 && this.value.length > 0) {
            char[] var2 = this.value;

            for(int var3 = 0; var3 < this.value.length; ++var3) {
                var1 = 31 * var1 + var2[var3];
            }

            this.hash = var1;
        }

        return var1;
    }
```

HashMap类的查询操作源码, 如下:
```
 /**
     * Implements Map.get and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab;
        Node<K,V> first, e;
        int n;
        K k;
        //先查看该key被分配到哪个槽里
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
            //先检查头结点
            if (**first.hash == hash** &&
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
                
            //若头部结点不是, 则依次遍历后续结点   
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (**e.hash == hash** &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

其中, 在检索的过程中, 都会先判断hash值是否相等, 若相等才会判断是否是同一结点或是等价结点。
hash值得计算方法如下, 从代码中看出, 若equals的对象不具有相同的hashCode()值, HashMap将无法正常使用。

```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = **key.hashCode())** ^ (h >>> 16);
    }
```

[hashCode1]: http://noahsnail.com/2016/12/02/2016-12-2-Effective%20Java%202.0_%E4%B8%AD%E8%8B%B1%E6%96%87%E5%AF%B9%E7%85%A7_Item%209/

### 参考文献:

1. https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode()
2. https://muhammadkhojaye.blogspot.tw/
3. https://www.ibm.com/developerworks/library/j-jtp05273/
4. http://www.ebooksbucket.com/uploads/itprogramming/java/Effective_Java_2nd_Edition.pdf
5. http://noahsnail.com/2016/12/02/2016-12-2-Effective%20Java%202.0_%E4%B8%AD%E8%8B%B1%E6%96%87%E5%AF%B9%E7%85%A7_Item%209/
