---
layout: post
title: 逻辑回归
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning"
---
### 逻辑回归

 ![Why-We-Need-LR](/images/ml/logisticRegression/WhyNeedLogisticRegression.png)

从上述图中可以看出, 使用线性回归对上述问题并不能很好的进行分类, 这时我们我们引入了逻辑回归(遇到分类问题)。

逻辑回归的基本模型如下, 其中我们引入了sigmoid函数(a.k.a, logistic函数), 即上述的g(z)。

 ![LR-Hypothesis-Model](/images/ml/logisticRegression/LR-Hypothesis-Model.png)
 
其中, 对于h(θ)的解释如下:

 ![logistic-model-interpret](/images/ml/logisticRegression/logistic-model-interpret.png)
 
至于为什么会选择sigmoid函数, 以及h(θ)解释为y=1的概率请看[这里][why-sigmoid-function]。

### 决策边界（Decision Boundary)

Decision Boundary就是能够将所有数据点进行很好地分类的边界, 该边界可以有h(θ)求得。以下两张例子分别属于linear decision boundary和non-linear decision boundary:

 ![linear-decision-bound](/images/ml/logisticRegression/linear-decision-bound.png)
 
 ![non-linear-decision-bound](/images/ml/logisticRegression/non-linear-decision-bound.png)

### Cost Function

如果和线性回归一样使用如下作为cost function, 得到的函数并不是convex, 如下图所示:

 ![LinearR-Cost-LogisticR](/images/ml/logisticRegression/lineraRegression-costFunction-Logistic.png)

原因则是线性回归的数据我们认为是正态分布的, 而(二分)逻辑回归的数据是二项分布的, 因此线性回归的cost函数并不适合逻辑回归。[详情][why-sigmoid-function]
基于极大似然估计, 逻辑回归得到了一个新的cost function, 如下:

 ![Logistic-CostFunction](/images/ml/logisticRegression/Logistic-CostFunction.png)

然后, 使用梯度下降方法即可求出最优的θ, 具体如下:

 ![logistic-gradient](/images/ml/logisticRegression/logistic-gradient.png)

### multi-class Regression

所谓one-vs-all method就是将二项分类的方法应用到多项分类中。
比如现在有k类, 那么就将其中一类作为positive，另（k-1）合起来作为negative, 按照二项分类的方式就可以得到k个分类器。
然后, 求出对新的样例, 利用这k个分类器就可以得到在这k个分类下各自的概率,取概率大的分类为这个样例的分类。模型如下:

![multiclass-lr](/images/ml/logisticRegression/multiclass-lr.png)

### 过拟合(over-fitting)

给定一个假设空间H，一个假设h属于H，如果存在其他的假设h’属于H,使得在训练样例上h的错误率比h’小，但在整个实例分布上h’比h的错误率小，那么就说假设h过度拟合训练数据。

![over-fitting](/images/ml/logisticRegression/over-fitting.png)

如何解决该问题呢? 有两个方法：

1. 减少feature个数（人工定义留多少个feature、算法选取这些feature）

2. regularization（留下所有的feature，但对于部分feature定义其parameter非常小）

![solve-over-fitting](/images/ml/logisticRegression/solve-over-fitting.png)

### regularization

![regulation](/images/ml/logisticRegression/regulation.png)

![logisticRegressionCost](/images/ml/logisticRegression/logisticRegressionCost.png)

### 矩阵计算(normal equation)

![linearNormCost](/images/ml/logisticRegression/linearNormCost.png)

![logistic-regression-regulation-cost](/images/ml/logisticRegression/logistic-regression-regulation-cost.png)


[why-sigmoid-function]: http://www.hanlongfei.com/机器学习/2015/08/05/mle

## 参考文献:

1. https://www.cs.princeton.edu/courses/archive/spring07/cos424/papers/mitchell-dectrees.pdf
2. http://leijun00.github.io/2014/09/decision-tree/
3. http://www.hanlongfei.com/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/2015/08/05/mle