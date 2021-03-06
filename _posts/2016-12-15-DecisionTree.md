---
layout: post
title: 决策树
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning"
---
## 介绍

决策树学习是一种近似估计离散函数(discrete-valued function)的方法, 经过学习的函数就代表了一棵决策树。
另外, 该决策树表示为 if-then 语句的集合时, 也能更易于人们理解, 下图是一个根据Outlook, Temperature等
属性是否打高尔夫球的决策树:
![ID3算法](/images/ml/decisionTree/IFELSE.png)

目前,这些学习方法在归纳推理算法中极其流行, 同时也广泛地应用于[疾病检测][DESEASEDIGNOSE]和[信贷风险评估][RISKLOAN]等领域。

## 决策树的表示方法

决策树将每个实体(instance)沿着树根到叶子节点进行判断, 这样就可以得到该实体的分类。
例如, 一个例子如下:

<Outlook = Sunny, Temperature = Hot, Humidity = High, Wind = Strong>


决策树种的每个结点表示了实体某种属性的判断, 结点的每个分支则对应了该属性的一个可能的取值。
我们根据上述决策树(最左分支)就能预测该实体不会打高尔夫球了。 

## 适合决策树学习的问题

决策树学习方法通常适合运用于具有如下特点的问题中:

1. 实例都是以key-value形式的。
2. 目标函数的结果是离散的。比如true/false输出, 或者更多的输出结果, 甚至是实值函数(functions with real-valued outputs), 尽管这种情况并不常见。
3. 训练集可能含有错误数据。决策树学习方法对错误数据具有鲁棒性,这里的错误数据包含训练集数据的分类和属性的错误值。
4. 训练集中某些属性的值可能缺失。

## 算法

大部分决策树算法都是为了构建一个自上而下的, 贪心搜索的决策树。那么问题是, 哪个属性应该放在决策树的根节点进行检验?

为了回答这个问题, 需要找到训练集中最优的属性作为根结点, 该结点的每个分支是基于该属性不同的取值, 然后对该结点的各个子结点用相应
的方法。


### 选择哪个属性作为最优的分类器?

首先选择一种统计特性--信息增益(information gain), 去度量各属性作为分类器的效果。


### 熵(Entropy)

为了更准确地定义信息增益(information gain), 信息论中的熵被用来表示训练集中数据的不确定性程度。
给定集合S, 该集合包含两种分类结果, yes 和 no, 那么S的熵则是:

$$ Entroy(S) = - p_{y} \log_{2} p_{y} - p_{n} \log_{2} p_{n} $$

其中, $$p_{y}$$ 表示 S中yes的占比, $$p_{n}$$ 表示 S 中no的占比。

在信息论中, 熵的一个解释是 根据信息的概率分布对信息编码所需要的最短平均编码长度, 这里的解释可以看[知乎-達聞西][ZHIHULIANWENXI]的回答。

更一般化一些, 如果目标属性有 c 个不同的取值, 那么集合S的熵则定义为:

$$ Entroy(S) = \sum_{i=0}^{c} - p_{i} \log_{2} p_{i}$$

其中, $$p_{i}$$ 表示 S中值取第i类的占比。这里之所以使用底为2的log函数, 是因为熵是编码长度所占bit的度量。

### 信息增益(information gain)分析

既然已经给定熵作为信息不确定程度的度量, 那么我们可以定义一种度量方式, 来度量某个属性在训练集中作为分类器的效果,
它就是信息增益(information gain), 即采用某个属性作为分类器后, 集合中的不确定性程度(熵)减少的期望值。

给定集合S 和 某个属性A, 信息增益Gain(S, A)则被定义为:

$$ Gain(S, A) = Entropy(S) - \sum_{v \in Values(A)} \frac{|S_{v}|}{|S|}Entropy(S_{v}) $$

其中, Values(A) 表示A属性所有取值构成的集合, $$S_{v}$$ 表示S的集合中属性A取值v 所构成的子集
(i.e., $$S_{v}$$ = { s $$\in$$ S|A(s) = v})。信息增益Gain(S, A) 就表示当知道属性A的值, 对S中
的任意成员进行编码时可以节省的bit数目。

在构造决策树的过程中, ID3算法使用信息增益精确地度量, 以在每一步选择当前情况下最优的属性。ID3 算法描述如下:
![ID3算法](/images/ml/decisionTree/ID3.png)

## 算法实现

1. 使用决策树来分析眼镜问题, [代码][LENCODE]

[ZHIHULIANWENXI]: https://www.zhihu.com/question/22178202
[DESEASEDIGNOSE]: https://sites.google.com/a/lclark.edu/drake/courses/ai/decision-trees-heart-disease
[RISKLOAN]: http://courses.media.mit.edu/2008fall/mas622j/Projects/CharlieCocoErnestoMatt/decision_trees/
[LENCODE]: https://github.com/HengGeneral/machineLearning/tree/master/decisionTree

## 参考文献:

1. https://www.cs.princeton.edu/courses/archive/spring07/cos424/papers/mitchell-dectrees.pdf
2. http://leijun00.github.io/2014/09/decision-tree/