---
title: Object Detection
toc: true
tags:
  - GBD
categories:
  - 深度学习
date: 2017-05-19 11:16:12
---

最近两个月忙着找实习和做实验写论文，一直没更新博客，今天上午听了港中文大学[欧阳万里](http://www.ee.cuhk.edu.hk/~wlouyang/)老师关于目标检测的讲座，在思路上很有的启发，结合自己的理解记一下笔记。

<!--more-->

在使用深度学习这个工具的时候，我们不能将它当成一个黑箱去使用，我们要通过可视化具体层或者神经元的局部响应去挖掘图像中视觉信息的相关关系，通过可视化可以使得我们了解模型内部的参数学习情况，比如人脸向左看和向右看会有不同的响应map，但这些是在标签中体现不出来的，通过分析模型的内部情况，从而指引我们设计针对这一问题域的网络结构，还有一点是通过实验证明，我们不需要纠结于寻找全局最优点的完美主义陷阱，通过不同的初始化值，网络最终会收敛到不同的局部最小值点，但是实验发现不同的局部最小值情况下的模型的性能相差并不大，大都符合我们的性能需求。

> 在利用深度学习的工具时候，最重要的是深入地分析问题域，我们从输入层、隐含层和输出层三个方面

### **输入结构化建模**

比如在ImageNet的detection任务中，各个种类的样本分布并不是均匀分布的，成现长尾分布，比如人的样本数目有2000多个，然而狮子只有几个，那么我们对于训练样本中的这种情况可以根据`视觉信息的相似性`进行训练样本的分组，可以看做是构建了一个基于视觉相似性的决策树，比如我们把牛、羊、够、狮子、豹子等类分为一组，这样我们可以先判断一个样本属于哪个组然后再进行细分类。

> 通过这种分组操作，我们可以让不同类之间共享相似的视觉信息特征

### **特征结构化建模**

论文[Crafting GBD-Net for Object Detection](https://arxiv.org/abs/1610.02579)介绍了如何利用图像本身的上下文进行目标检测，通过不同的候选区域的上下文信息来改进设计网络结构，比如是全连接结构、局部连接结构还是通过控制门进行特定的连接，论文中采用了根据上下文信息采用控制门来决定是否将特征传递到下一层，如图所示：

![](/img/Object-Detection/context.jpg)

- 首先通过在候选区域内选择不同的尺寸进行预测

![](/img/Object-Detection/gate.jpg)

- 在网络结构的优化方面，引入了信息传递控制门，如果上下文和候选区域属于一类那么就将特征传递到下一层，否则丢弃。

![](/img/Object-Detection/gate1.jpg)

在上图的例子中，我们选择`兔耳朵`进行预测，那么上下文信息属于同一类那么就会传递到下一层，否则如果是人类，就不将特征传递到下一层

### **检测器**
    
![](/img/Object-Detection/detector.jpg)

当我们做一些监测任务时，我们可以先在低层用一些局部的检测器来检测局部特征，在高层通过组合不同的低层检测器来提取特征，这样就解决了在整个GroundTruth上有局部遮挡或者形变的情况，相当于先利用局部特征进行小分类，再到上下文中去做大的分类。
