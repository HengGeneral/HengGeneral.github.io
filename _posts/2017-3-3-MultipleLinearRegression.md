---
layout: post
title: 多变量线性回归
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning"
---
## 介绍

### 基本概念

输出由多维输入决定的线性回归称为多变量线性回归，如下图所示, 其中Price为输出，前面四维为输入：

![multi-linear-regression-example](/images/ml/multiLinearRegression/multi-linear-regression-example.png)

多变量线性回归的一个常用的hypothesis为

$$h_{\theta} = x_{0} + {\theta}_{1}{x}_{1} + {\theta}_{2}{x}_{2} + ... + {\theta}_{n}{x}_{n}$$

具体如下:

![multi-linear-regression-model](/images/ml/multiLinearRegression/multi-variable-linear-regression-model.png)

### 梯度下降

多变量线性回归中cost function的梯度下降算法如下:

![multi-linear-regression-gradient](/images/ml/multiLinearRegression/multi-linear-regression-gradient.png)

多变量线性回归和单变量线性回归对比如下:

![multi-linear-regression-gradient](/images/ml/multiLinearRegression/multi-linear-regression-gradient.png)

### 归一化

我们可以对输入进行归一化来提高梯度下降的速度。若θ在位于小区间上的输入参数进行梯度下降, 下降速度会很快; 若θ在位于比较大区间上的输入参数进行梯度下降, 下降速度会比较慢。

![not-same-range](/images/ml/multiLinearRegression/not-same-range.png)

有两种方法可以通过是输入参数的范围变得均匀, 进而改善这种效果: feature scaling 和 normalization。

#### Feature Scaling

![feature-scaling](/images/ml/multiLinearRegression/feature-scaling.png)

#### Normalization

![normalization](/images/ml/multiLinearRegression/normalization.png)

### Learning Rate

梯度下降算法中另一关键点就是机器学习率的设计：设计准则是保证每一步迭代后都保证能使cost function下降。具体如下:

![learning-rate-object](/images/ml/multiLinearRegression/gradient-rate-object.png)

在α选取的时候,过大或者过小都会产生不好的现象。

![learning-rate-effect](/images/ml/multiLinearRegression/learning-rate-effect.png)

如何选取α?

![learning-rate-experience](/images/ml/multiLinearRegression/learning-rete-experience.png)

### Features and Polynomial Regression
 
![polynomial-example](/images/ml/multiLinearRegression/multi-linear-regression-example.png)

### Normal Equation

![normal-equation-example](/images/ml/multiLinearRegression/normal-equation-example.png)

![normal-equation](/images/ml/multiLinearRegression/normal-equation.png)

### Normal Equation Noninvertibility

对于有m个样本，每个拥有n个feature的一个训练集，有X是m×(n+1)的矩阵，$$X^{T}X$$是(n+1)×(n+1)的方阵。

那么对于参数θ的计算就出现了一个问题，如果|$$X^{T}X$$|=0,即$$X^{T}X$$不可求逆矩阵怎么办？1.冗余feature的删除; 2. m<=n的情况，feature过多。

![deal-non-invert](/images/ml/multiLinearRegression/deal-non-invert.png)

当然也可以使用octave, 在遇到不可逆的情况下也会返回正确的答案。

![pinv-invert-matrix](/images/ml/multiLinearRegression/pinv-invert-matrix.png)


### 参考文献:

1. https://www.coursera.org/learn/machine-learning/home/week/2
2. http://blog.csdn.net/abcjennifer/article/details/7700772