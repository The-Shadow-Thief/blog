---
title: Gradient Descent
date: 2016-09-05 22:46:04
toc: true
tags:
  - 梯度下降法
categories:
  - cs231n
---

在`SVM and Softmax`那篇文章中, 我们知道模型最主要的两个模块, `评分函数`和`代价函数`, 我们通过`代价函数`来表征真实值与模型预测值之间的差异, 当然我们只知道和真实值相差多少是不够的, 我们还要不断的更改模型参数$W$, 使得差异越来越小, 这个过程就是最优化, 这篇文章介绍最优化方法中广泛使用的`梯度下降法` 

<!--more-->

> 预备知识请看, [从线性模型到神经网络](http://simtalk.cn/2016/08/23/%E4%BB%8E%E7%BA%BF%E6%80%A7%E6%A8%A1%E5%9E%8B%E5%88%B0%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/) 中的`梯度下降法`部分

### **损失函数的可视化分析**

在最优化方法中, 我们优化的目标函数是`代价函数`, 优化的变量是参数集$W$, 根据微分求极值的思想, 如果我们可以可视化代价函数的`单调性`, 就可以很容易发现代价函数的最小值在哪里, 但是参数集$W$通常处于高位空间中, 那么只能在参数集中的一个维度或者两个维度方向上做切面, 我们可以随机生成一个$W$, 将其中的一个参数(一维)或者两个参数(二维)作为变量, 其余的参数固定不变, 那么就可以可视化损失函数了, 如图所示

![](/img/Gradient-Descent/svm1d.png)

- `一维可视化` : 首先我们随机生成一个参数集$W$, 为了使得$W$在某一个方向上变化, 我们随机选定一个方向$W_1$, 通过改变$a$的值$ W+aW_1 $使得参数集在某一维度上变化, 这样就可以看到损失函数$L(W + a W_1)$的变化曲线

![](/img/Gradient-Descent/svm_one.jpg)

`PS:` 蓝色部分是低损失值区域，红色部分是高损失值区域, 单一样本

- `二维可视化`: 同样的道理, 我们随机生成一个参数集$W$, 为了使得$W$在某两个方向上变化, 我们随机选定两个方向$W_1,W_2$, 通过$ W+aW_1+bW_2 $使得参数集在某两个维度上变化, 这样就可以看到损失函数$L(W + a W_1+bW_2)$的变化曲线

![](/img/Gradient-Descent/svm_all.jpg)

- 上图表示的是多个样本的损失值的总体平均值, 因为损失函数的分段线性结构, 所以上图是很多的分段线性结构的平均值

> 什么是损失函数的分段线性结构?

$$ L\_i = \sum\_{j\neq y\_i} \left[ \max(0, w\_j^Tx\_i - w\_{y\_i}^Tx\_i + 1) \right] $$

通过上式可知, 每个样本的数据损失值是以参数集$W$的线性函数的总和, 举个例子

- 有3个特征维度为一维的样本点$\\{ (x_0),(x_1),(x_2) \\}$, 数据集有3个类别, 那么无正则项的SVM损失值计算过程如下:

$$
\begin{align}
L_0 = & \max(0, w_1^Tx_0 - w_0^Tx_0 + 1) + \max(0, w_2^Tx_0 - w_0^Tx_0 + 1) \\\\
L_1 = & \max(0, w_0^Tx_1 - w_1^Tx_1 + 1) + \max(0, w_2^Tx_1 - w_1^Tx_1 + 1) \\\\
L_2 = & \max(0, w_0^Tx_2 - w_2^Tx_2 + 1) + \max(0, w_1^Tx_2 - w_2^Tx_2 + 1) \\\\
L = & (L_0 + L_1 + L_2)/3
\end{align} 
$$

由于是一维的,$x_i$和$w_j$都是单个值, 不是向量, 由于$max$函数是分段的, 且与0比较的是线性函数$wx+1$的形式, 如下图所示,

![](/img/Gradient-Descent/svmbowl.png)

- 上图是从一个维度的方向上进行可视化的损失函数, x轴代表某个$w_j$, y轴是损失值, 代价函数中的数据损失部分是由多个分段线性损失组合(本质上就是不同的HingeLoss函数)而成, 完整的SVM代价函数的数据损失部分就是将这个形状扩展到高维空间 

`PS`: 

1. 根据SVM的损失函数的形状可以判定是一个凸函数, 但是我们将评分函数$f(x)$变为神经网络那么目标函数就不是凸函数了, 而是类似凹凸不平的复杂地形的形状

2. 由于在梯度下降法中有微分运算, $max$函数在拐点处是不可导的, 梯度没有定义, 我们用`次梯度`的概念来代替`梯度`的概念

### **最优化策略**

既然代价函数可以评价某个参数集$W$的好坏程度, 那么我们最优化的目标就是找到能够最小化损失函数的参数集$W$

1. **初始化参数集** : 随机搜索, 即将$W$随机赋值

2. **迭代优化** : 在随机参数的基础上进行模型参数集的迭代更新, 使得模型的损失值更低
 
**更新参数策略** : 

当满足什么条件时, 我们才进行参数的更新, 然后如何更新呢?

1. 随机本地搜索 : 我们在随机参数集$W$的基础上, 加上一个随机的扰动$\delta W$然后新的参数集$W + \delta W$如果可以使得损失函数值更小, 那么就更新参数, 否则不更新参数

2. 在`随机本地搜索`中, 通过随机项我们试图找到一个可以使得损失函数下降的方向, 其实不用随机寻找, 在数学上`梯度`的定义就是函数$f(x)$增长最快的方向(最陡峭), 我们可以通过计算损失函数的梯度来找到下降最快的方向(最陡峭)

在一维函数中，斜率是函数在某一点的瞬时变化率。梯度是函数的斜率的一般化表达，它不是一个值，而是一个向量。一维空间的斜率如下

$$\frac{df(x)}{dx} = \lim_{h\ \to 0} \frac{f(x + h) - f(x)}{h}$$

`PS` : 在高维空间中, 梯度是各个维度的斜率组成的向量, 函数有多个参数的时候，在某个方向上的导数为偏导数

### **梯度的计算**

**数值梯度法 : 计算速度慢且不精确**

    ```python
    def eval_numerical_gradient(f, x):
      """ 
      a naive implementation of numerical gradient of f at x 
      - f should be a function that takes a single argument
      - x is the point (numpy array) to evaluate the gradient at
      """ 
    
      fx = f(x) # evaluate function value at original point
      grad = np.zeros(x.shape)
      h = 0.00001
    
      # iterate over all indexes in x
      it = np.nditer(x, flags=['multi_index'], op_flags=['readwrite'])
      while not it.finished:
    
        # evaluate function at x+h
        ix = it.multi_index
        old_value = x[ix]
        x[ix] = old_value + h # increment by h
        fxh = f(x) # evalute f(x + h)
        x[ix] = old_value # restore to previous value (very important!)
    
        # compute the partial derivative
        grad[ix] = (fxh - fx) / h # the slope
        it.iternext() # step to next dimension
    
      return grad
    ```

   **注意**: 
 
   - 在数学公式中，h的取值是趋近于0的，然而在实际中，用一个很小的数值（比如例子中的1e-5）就足够了。
   - 实际中用中心差值公式（centered difference formula）: $\frac{df(x)}{dx} =\frac{[f(x+h) - f(x-h)]}{ 2 h}$
   - **在梯度负方向上更新：在上面的代码中，为了计算$W_{new}$，要注意我们是向着梯度$df$的负方向去更新，这是因为我们希望损失函数值是降低而不是升高。**
   - `学习率`的影响, 参见[从线性模型到神经网络](http://simtalk.cn/2016/08/23/%E4%BB%8E%E7%BA%BF%E6%80%A7%E6%A8%A1%E5%9E%8B%E5%88%B0%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/#梯度下降法) 中的`梯度下降法`部分
   - 效率问题: 计算数值梯度的复杂性和参数的量线性相关, 在模型中有30730个参数，所以损失函数每走一步就需要计算30731次损失函数的梯度, 神经网络很容易就有上千万的参数，显然这个策略不适合大规模数据，我们需要更好的策略。
 
 
2. **微分分析计算梯度 : 速度快且精确**

使用有限差值近似计算梯度比较简单，但缺点是近似计算（对于h = 0.00001是选取了一个很小的数值，但真正的梯度定义中h趋向0的极限），且耗费计算资源太多, 但是利用微分来分析，能得到计算梯度的公式（不是近似），用公式计算梯度速度很快，唯一不好的就是实现的时候容易出错。

我们拿$x_i$点来对SVM损失函数来进行举例:

$$ L\_i = \sum_{j\neq y\_i} \left[ \max(0, w\_j^Tx\_i - w\_{y\_i}^Tx\_i + \Delta) \right] $$

对$w\_{y\_i}$进行微分得到,

$$\nabla\_{w\_{y\_i}} L\_i = - \left( \sum\_{j\neq y\_i} \mathbb{1}(w\_j^Tx\_i - w\_{y\_i}^Tx\_i + \Delta > 0) \right) x\_i$$

- 如果括号中的条件为真，那么函数值为1，如果为假，则函数值为0。
- 代码实现的时候比较简单：只需要计算没有满足边界值的分类的数量（由于对损失函数产生了贡献），然后乘以x_i就是梯度了。

**注意**, 上个式子中是对$w\_{y\_i}$进行求导, 得到的是对正确分类所对应的参数集$W$的行进行求梯度, 其余的行$(j \neq y_i)$需要对$w_j$求导, 他们的梯度为

$$\nabla\_{w\_j} L\_i = \mathbb{1}(w\_j^Tx\_i - w\_{y\_i}^Tx\_i + \Delta > 0) x\_i$$

- 可以将每一行理解为一个线性函数, 我们要优化这个线性函数的参数向量

### **梯度下降**

```python
while True:
  weights_grad = evaluate_gradient(loss_fun, data, weights)
  weights += - step_size * weights_grad 
   perform parameter update
```

- 到目前为止，梯度下降是对神经网络的损失函数最优化中最常用的方法。

**小批量数据梯度下降（Mini-batch gradient descent）：**

在大规模数据的训练中, 训练数据可达百万数量级, 这样更新参数太笨重了, 常用的方法就是利用训练集中的一小部分数据来更新一个参数, 这样做基于一个假设: `数据具有相关性`, 实际情况中，数据集肯定不会包含重复图像，那么小批量数据的梯度就是对整个数据集梯度的一个近似。因此，在实践中通过计算小批量数据的梯度可以实现更快速地收敛，并以此来进行更频繁的参数更新。

```
# Vanilla Minibatch Gradient Descent
while True:
  data_batch = sample_training_data(data, 256) # sample 256 examples
  weights_grad = evaluate_gradient(loss_fun, data_batch, weights)
  weights += - step_size * weights_grad 
  # perform parameter update
```

- 小批量数据策略有个极端情况，那就是每个批量中只有1个数据样本，这种策略被称为`随机梯度下降（Stochastic Gradient Descent 简称SGD）`, SGD在技术上是指每次使用1个数据来计算梯度，实际上通常使用SGD来指代小批量数据梯度下降

- 小批量数据的大小是一个超参数，但是一般并不需要通过交叉验证来调参, 它一般由存储器的限制来决定的，或者干脆设置为同样大小，比如32，64，128等, 之所以使用2的指数，是因为在实际中许多向量化操作实现的时候，如果输入数据量是2的倍数，那么运算更快。

### **模型数据流**

![](/img/Gradient-Descent/dataflow.jpeg)

- 将参数集$W$和样本$x_i$输入`评分函数`$f(x_i,W)$得到评分(class scores)

- $W$计算得到`正则损失(regularization loss)`, 真实值$y_i$和`评分(class scores)`通过相应的损失函数计算得到`数据损失(data loss)`

- `模型总代价`$L$ = `正则损失` + `数据损失`

> **特别鸣谢** : 文中部分内容引自知乎专栏 [智能单元](https://zhuanlan.zhihu.com/p/21387326?refer=intelligentunit), 感谢杜客团队的翻译