---
layout: post
title: HBase数据模型
tags:  [HBase]
categories: [NoSQL]
author: liheng
excerpt: "hbase data model"
---

### 数据模型

#### 表(table)

和 MySQL 一样, HBase的一个表有很多行组成。

#### 行(row)

HBase 中的一行包含一个rowkey和多个列的值组成。
同时, row是按字典序存储的。因此, rowkey的设计就显得很重要。

#### 列(column)

HBase中的一列由列簇(column family)和 列描述符(column qualifier)表示, 它们之间以:分隔。

#### 列簇(column Family)

列簇就是一些列的集合。
在物理上，一个列簇的成员在文件系统上都是存储在一起。
每个列簇都会包含很多存储属性, 比如值是否进行缓存、数据是否压缩以及rowkey是否已经编码等。
表中的每一行都包含相同的列簇, 尽管该行在该列簇没有存放任何东西。

#### 列描述符(column qualifier)

列描述符加在列簇之后, 用于給相应的数据片段提供索引。
给定一个列簇content, 列描述符可能是content:html 或 content:pdf。
尽管列簇在表创建过程中固定好了, 但是列描述符是可变的, 并且不同行的列描述符可能不同。

#### 单元(cell)

一个单元由行、列簇、列描述符、值以及相应的时间戳(表示值的版本)构成。

#### 时间戳(timestamp)

时间戳是伴随着值而写入到单元的, 它也是值的版本的标识。

###

在HBase中, 空的单元并不占存储空间, 这也是HBase稀疏的原因。



### 参考文献:
1. http://hbase.apache.org/book.html#datamodel