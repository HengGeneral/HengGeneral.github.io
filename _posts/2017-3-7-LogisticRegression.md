---
layout: post
title: 逻辑回归
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning"
---
## 介绍

#### 为什么上述问题线性回归无法解决, 需要引入逻辑回归?

 ![Why-We-Need-LR](/images/ml/logisticRegression/WhyNeedLogisticRegression.png)
 
#### hypothesis

 ![LR-Hypothesis-Model](/images/ml/logisticRegression/LR-Hypothesis-Model.png)

#### 决策边界（Decision Boundary)

The decision boundary is the line that separates the area where y = 0 and where y = 1. It is created by our hypothesis function.

#### Cost Function

如果和线性回归一样使用如下作为cost function, 得到的函数并不是convex, 如下图所示:
 ![LinearR-Cost-LogisticR](/images/ml/logisticRegression/lineraRegression-costFunction-Logistic.png)

因此, 又构造了一个新的cost function, 如下:
 ![Logistic-CostFunction](/images/ml/logisticRegression/Logistic-CostFunction.png)

#### multi-class Regression
 ![multiclass-lr](/images/ml/logisticRegression/multiclass-lr.png)

#### 过拟合(over-fitting)
![over-fitting](/images/ml/logisticRegression/over-fitting.png)

如何解决该问题呢?
![solve-over-fitting](/images/ml/logisticRegression/solve-over-fitting.png)

#### regularization

![regulation](/images/ml/logisticRegression/regulation.png)

![logisticRegressionCost](/images/ml/logisticRegression/logisticRegressionCost.png)

![linearNormCost](/images/ml/logisticRegression/linearNormCost.png)

![logistic-regression-regulation-cost](/images/ml/logisticRegression/logistic-regression-regulation-cost.png)


## 参考文献:

1. https://www.cs.princeton.edu/courses/archive/spring07/cos424/papers/mitchell-dectrees.pdf
2. http://leijun00.github.io/2014/09/decision-tree/