---
layout: post
title: Java 集合框架Set
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java 集合框架"
---

### Set

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

![ListTree](/images/java/mapSet.png)

#### Hashtable



### 参考文献
1. http://stackoverflow.com/questions/2444359/what-is-the-difference-between-a-hashmap-and-a-treemap
2. 