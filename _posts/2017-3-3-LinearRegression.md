---
layout: post
title: 线性回归
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning"
---
## 介绍

首先看什么是学习（learning）？一个成语就可概括：举一反三。
此处以高考为例，高考的题目在上考场前我们未必做过，但在高中三年我们做过很多很多题目，懂解题方法，因此考场上面对陌生问题也可以算出答案。
机器学习的思路也类似：我们能不能利用一些训练数据（已经做过的题），使机器能够利用它们（解题方法）分析未知数据（高考的题目）？[来源知乎-王丰][whatIsLearning]

### 基本概念

监督学习(Supervised Learning): Given the "right answer" for the example in the data; 否则就是非监督学习(Unsupervised Learning)。

监督学习的学习过程是: 给定测试样本集, 我们的目标是学习到一个函数h: x->y, 同时y是x的一个比较好的预测值。具体流程如下图所示:

![supervise-learning-process](/images/ml/linearRegression/machine-learning-process.png)

回归问题(Regression Problem): Predict real-value output; 否则则是。 

### Cost Function

如何评价上述学习到的hypothesis function, 我们引入了cost function。
下图中所列出的cost function称作"Squared error function" 或 "Mean squared error"。
1/2 是为了再涉及到梯度下降(导数)的时候为了方便。具体如下图所示:

![cost-function](/images/ml/linearRegression/cost-function.png)

当只有个输入时, 模型如下图所示:

![single-parameter-cost-function](/images/ml/linearRegression/single.png)

同时我们能画出J(θ)在一定场景下的曲线。从图中可以找到极值点θ, 使得J(θ)取值最小。

![single-variable-cost-funcion-picture](/images/ml/linearRegression/single-variable-cost-funcion-picture.png)

当对于两个变量时, J(θ)的曲面如下。从图中看出, 存在极值点(θ1, θ2)使得J(θ)取值最小。

![dual-parameter-cost-function](/images/ml/linearRegression/dual-parameter-cost-function.png)

### Parameter Learning


[whatIsLearning]: https://www.zhihu.com/question/23194489/answer/25028661

## 参考文献:

1. 