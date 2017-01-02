---
layout: post
title: 朴素贝叶斯
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning-native bayes"
---
### 贝叶斯定理

贝叶斯定理的数学表达式,如下:

$$ P(A\mid B)={\frac{P(B\mid A)P(A)}{P(B)}}$$

其中,

$$P(A)$$ 和 $$P(B)$$ - 事件A 和 事件B发生的概率(不用关心彼此是否发生),

$$P(A\mid B)$$ - 条件概率, 在事件B发生的条件下, A事件发生的概率

$$P(B\mid A)$$ - 条件概率, 在事件A发生的条件下, B事件发生的概率

### 贝叶斯决策思想

假如有一个数据集, 它由两类数据组成, 数据分布如下图所示:

 ![Bayes-Decision-Theroy](/images/ml/nativeBayes/bayes-decision-introducion.png)

现在用 p1(x, y) 表示数据结点(x, y)属于类别1的概率, p2(x, y) 表示数据结点(x, y)属于类别2的概率。
那么对于一个新数据结点(x, y), 可以用下面的规则来判断它的类别:

1.  如果p1(x, y) > p2(x, y), 那么属于类别1;
2.  如果p1(x, y) < p2(x, y), 那么属于类别2.

即选择高概率所对应的类别。

上述问题可以抽象化为, 给定一个具有n个属性的实例, 表示为$${\mathbf{x}} = (x_{1},\dots , x_{n})$$,
其中$$x_{i}$$ 表示第$$i_{th}$$个属性的取值, 该实例属于$$C_{k}$$分类下的概率记为:

$$p(C_{k}\vert x_{1}, \dots, x_{n})$$

其中, k为分类的个数。

根据Bayes' theorem, 上述条件概率可以写成:
 
$$p(C_{k}\mid \mathbf {x}) = {\frac{p(C_{k}) p(\mathbf {x} \mid C_{k})} {p(\mathbf {x})}}$$

实际上, 没有必要对n个属性的所有取值进行组合枚举, 同时它的取值分布和n个属性有关系, 与分类没有关系。
在各个属性给定的情况下, $$p(\mathbf {x})}$$ 为常数。而分子则是$$\mathbf{x}$$ 和 $$C_{k}$$的联合概率, 表示为:

$$p(C_{k},x_{1},\dots ,x_{n})$$

根据[Chain rules][ChainRules], 可以写为:

$$p(C_{k},x_{1},\dots ,x_{n}) = p(x_{1},\dots ,x_{n},C_{k})$$
$$                            = p(x_{1}\mid x_{2},\dots ,x_{n},C_{k})p(x_{2},\dots ,x_{n},C_{k})$$
$$                            = p(x_{1}\mid x_{2},\dots ,x_{n},C_{k})p(x_{2}\mid x_{3},\dots ,x_{n},C_{k})p(x_{3},\dots ,x_{n},C_{k})$$
$$                            = p(x_{1}\mid x_{2},\dots ,x_{n},C_{k})p(x_{2}\mid x_{3},\dots ,x_{n},C_{k})\dots p(x_{n-1}\mid x_{n},C_{k})p(x_{n}\mid C_{k})p(C_{k})$$

另外, 朴素(naive) 假定 各个属性之间相互独立, 则有

$$p(C_{k},x_{1},\dots ,x_{n}) = p(x_{1}\mid C_{k})\ p(x_{2}\mid C_{k})\dots\ p(x_{3}\mid C_{k})p(C_{k})$$

因此, 计算$$p(C_{k}\mid x_{1},\dots ,x_{n})$$ 可以看成计算 $$p(C_{k})\prod _{{i=1}}^{n}p(x_{i}\vert C_{k})$$

[ChainRules]: https://en.wikipedia.org/wiki/Chain_rule_(probability)

## 参考文献:
1. https://en.wikipedia.org/wiki/Bayes'_theorem
2. 《机器学习实战》


