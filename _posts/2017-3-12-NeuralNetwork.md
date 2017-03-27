---
layout: post
title: 神经网络
tags:  [机器学习]
categories: [机器学习]
author: liheng
excerpt: "Machine Learning"
---
### 介绍

在之前讨论的机器学习问题中，主要针对Regression做了分析，其中采用梯度下降法或矩阵计算进行参数更新。
然而其可行性基于假设参数不多，如果参数多起来或者模型复杂起来怎么办呢？
比如下图中这个例子：从100*100个pixels中选出所有$$x_{i}x_{j}$$作为logistic regression的一个参数，那么总共就有$$5*10^{7}$$个feature，即x有这么多维。
因此使用回归中用到的梯度下降时, 就需要同时更新$$5*10^{7}$$个θ; 若使用矩阵计算, 内存中放入这么大的矩阵也特别耗费计算资源。

 ![Why-We-Need-NN-1](/images/ml/neuralNetwork/motivition1.png)
 
 ![Why-We-Need-NN-2](/images/ml/neuralNetwork/motivition2.png)

### 神经网络的表示形式

 ![neuralNetwork-model](/images/ml/neuralNetwork/neuralNetwork-model.png)
 
通过以上这种神经元的传递方式（input->activation->output）来计算h(x), 我们称做Forward propagation(向前传递)。具体的计算方式如下:
 
 ![neuralNetwork-model-relation](/images/ml/neuralNetwork/neuralNetwork-model-relation.png)
 
### cost function
 
给定m个训练样本，每个样本包含一组输入x和相应的输出y，L表示神经网络层数，Sl表示每层的neuron个数(SL表示输出层神经元个数), 具体定义如下:

 ![neutral-network-definition](/images/ml/neuralNetwork/neutral-network-definition.png)

和逻辑回归类似, 神经网络的cost function定义如下:

 ![neural-network-cost](/images/ml/neuralNetwork/neural-network-cost.png)
 
### Backpropagation Algorithm

前面我们已经讲了cost function的形式，下面我们需要的就是最小化J(Θ), 如下:
   
![neural-network-cost-target](/images/ml/neuralNetwork/neutral-network-cost-target.png)

以梯度下降法为代表的很多优化算法可以用来解决上述的最小化问题。
在优化过程中，这些算法通常会用到 J(Θ) 对 Θ 中各元素的偏导数（Θ 是一个矩阵）, 反向传播算法就是用来计算这些偏导数的[推导][bp-reference]。

![bp-theta](/images/ml/neuralNetwork/bp-theta.png)

具体的算法如下:

![bp-algorithm](/images/ml/neuralNetwork/bp-algorithm.png)

### Backpropagation intuition

![bp-intuition](/images/ml/neuralNetwork/bp-intuition.png)

### Gradient Checking
    
神经网络中计算起来可能会有bug，那我们怎么知道咱们算法的实现对不对呢？
于是, 我们就可以使用gradient checking，通过check梯度判断我们的code有没有问题？思路如下：

![gradient-checking-idea](/images/ml/neuralNetwork/gradient-checking-idea.png)

按照上述思路, 对于每个参数的求导公式如下图所示：

![gradient-checking-process](/images/ml/neuralNetwork/gradient-checking-process.png)

最后判断两种计算方式的值是否很相似:

![gradient-checking-equal](/images/ml/neuralNetwork/gradient-checking-equal.png)

### Random Initialization
        
对于参数θ的initialization问题，我们之前采用全部赋0的方法。但使用该方法会存在如下问题:

![why-need-random](/images/ml/neuralNetwork/why-need-random.png)

为了解决该问题, 我们采用了如下的方法:

![random-initialization](/images/ml/neuralNetwork/random-initialization.png)

### Put it together

#### 选择神经网络模型

![nn-architect](/images/ml/neuralNetwork/nn-architect.png)

#### 计算过程

![nn-process-1](/images/ml/neuralNetwork/nn-process.png)

![nn-process-1](/images/ml/neuralNetwork/nn-process-2.png)

[bp-reference]: https://my.oschina.net/findbill/blog/529001

## 参考文献:

1. https://my.oschina.net/findbill/blog/529001
2. http://blog.csdn.net/abcjennifer/article/details/7758797
3. https://www.coursera.org/learn/machine-learning/home/week/4
4. https://www.coursera.org/learn/machine-learning/home/week/5