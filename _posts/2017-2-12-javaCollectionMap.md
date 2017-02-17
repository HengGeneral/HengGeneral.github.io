---
layout: post
title: Java 集合框架Map
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java 集合框架"
---

### Map

关联表, 用于存储key和value的映射。一个 map 不能包含重复的keys, 同时每个key最多对应一个value。

***

注意: 当使用可变得对象作为keys千万需要小心。当某对象作为map的key时, 当该对象中equals方法涉及到的变量发生修改时,
该map的行为将变得不可预测。另外, 当使用该map的某个value或者key为该map自己时, 尤其要小心, 同时 hashCode() 方法
和 equals() 方法的行为也无法预测。

***

Map中的某些方法在遍历map时, 如果遇到map中直接或者间接遇到self-referential实例时, 将会抛出异常,
比如clone(), equals(), hashCode() 和 toString()方法。

接口中的方法如下:

```
    int size();
    boolean isEmpty();
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    V get(Object key);
    V put(K key, V value);
    V remove(Object key);
    void putAll(Map<? extends K, ? extends V> m);
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();
    ...
```

常用的子类有HashMap, TreeMap 和 HashTable等, 如图所示:

![ListTree](/images/java/mapTree.png)


#### HashMap


#### TreeMap


#### HashTable


### 参考文献:
1. 