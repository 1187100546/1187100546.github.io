---
title: Lenet-5卷积神经网络结构
date: 2020-01-14 21:03:21
tags:
	- CNN
	- Deep Learning
	- 神经网络
categories: 深度学习
mathjax: true
---

最近在写大创论文，涉及到神经网络结构分析，记录一下（第一次写期刊论文是真不会呀，作图工具都不知道，第一个想到的居然是PPT.....)

<!--more-->

## Lenet-5基本结构

![lXGMTI.png](https://s2.ax1x.com/2020/01/15/lXGMTI.png)

 LeNet-5共有7层，不包含输入，每层都包含可训练参数（连接权重）。输入图像为32*32大小。这要比[Mnist数据库](http://yann.lecun.com/exdb/mnist/)（一个公认的手写数据库）中最大的字母还大。这样做的原因是希望潜在的明显特征如笔画断电或角点能够出现在最高层特征监测子感受野的中心。 

### 卷积层 

卷积层采用的都是5x5大小的卷积核/过滤器，且卷积核每次滑动一个像素，一个特征图谱使用同一个卷积核. 每个上层节点的值乘以连接上的参数，把这些乘积及一个偏置参数相加得到一个和，把该和输入激活函数，激活函数的输出即是下一层节点的值 。

### 池化层 

下抽样层采用的是2x2的输入域，即上一层的4个节点作为下一层1个节点的输入，且输入域不重叠，即每次滑动2个像素，下抽样节点的结构如下：      

![lXJFBj.png](https://s2.ax1x.com/2020/01/15/lXJFBj.png)



``上图显示的是一种池化方式，但是lenet-5采用的是最大池化，直白点说就是四个格子取最大的值。``

得出的值乘以一个参数加上一个偏置参数作为激活函数的输入，激活函数的输出即是下一层节点的值。 

## LeNet-5第一层：卷积层C1

输入图片：32×32

卷积核大小：5×5

卷积核种类：6

输出featuremap大小：28*28 （32-5+1）

神经元数量：28×28×6

可训练参数：（5×5+1）×6（每个滤波器5*5=25个unit参数和一个bias参数，一共6个滤波器）

连接数：（5×5+1）×6×28×28

C1层是卷积层，形成6个特征图谱。卷积的输入区域大小是5x5，每个特征图谱内参数共享，即每个特征图谱内只使用一个共同卷积核，卷积核有5x5个连接参数加上1个偏置共26个参数。卷积区域每次滑动一个像素，这样卷积层形成的每个特征图谱大小是(32-5)/1+1=28x28。C1层共有26x6=156个训练参数，有(5x5+1)x28x28x6=122304个连接。

##  LeNet-5第二层：池化层S2

输入：28×28

采样区域：2×2

采样方式：4个输入相加，乘以一个可训练参数，再加上一个可训练偏置。结果通过sigmoid

采样种类：6

输出featureMap大小：14×14（28/2）

神经元数量：14×14×6

可训练参数：2×6（和的权+偏置）

连接数：（2×2+1）×6×14×14

S2中每个特征图的大小是C1中特征图大小的1/4

 S2层是一个下采样层。C1层的6个28x28的特征图谱分别进行以2x2为单位的下抽样得到6个14x14（（28-2）/2+1）的图。每个特征图谱使用一个下抽样核。5（S2中的每个像素都与C1中的2∗2个像素和1个偏置相连接）x14x14x6=5880个连接。 

为什么是下采样？利用图像局部相关性的原理，对图像进行子抽样，可以减少数据处理量同时保留有用信息 。

有6个14×14的特征图。特征图中的每个单元与C1中相对应特征图的2×2邻域相连接。S2层每个单元的4个输入相加，乘以一个可训练参数，再加上一个可训练偏置。结果通过sigmoid函数计算。可训练系数和偏置控制着sigmoid函数的非线性程度。如果系数比较小，那么运算近似于线性运算，亚采样相当于模糊图像。如果系数比较大，根据偏置的大小亚采样可以被看成是有噪声的“或”运算或者有噪声的“与”运算。每个单元的2×2感受野并不重叠，因此S2中每个特征图的大小是C1中特征图大小的1/4（行和列各1/2）。S2层有12个可训练参数和5880个连接。

![lXYDQU.jpg](https://s2.ax1x.com/2020/01/15/lXYDQU.jpg)

图：卷积和子采样过程：卷积过程包括：用一个可训练的滤波器f<sub>x</sub>去卷积一个输入的图像（第一阶段是输入的图像，后面的阶段就是卷积特征map了），然后加一个偏置b<sub>x</sub>，得到卷积层Cx。子采样过程包括：每邻域四个像素求和变为一个像素，然后通过标量W<sub>x+1</sub>加权，再增加偏置b<sub>x+1</sub>，然后通过一个sigmoid激活函数，产生一个大概缩小四倍的特征映射图S<sub>x+1</sub>。

所以从一个平面到下一个平面的映射可以看作是作卷积运算，S-层可看作是模糊滤波器，起到二次特征提取的作用。隐层与隐层之间空间分辨率递减，而每层所含的平面数递增，这样可用于检测更多的特征信息。

## LeNet-5第三层：卷积层C3

输入：S2中所有6个或者几个特征map组合

卷积核大小：5×5

卷积核种类：16

输出featureMap大小：10×10

C3中的每个特征map是连接到S2中的所有6个或者几个特征map的，表示本层的特征map是上一层提取到的特征map的不同组合

存在的一个方式是：C3的前6个特征图以S2中3个相邻的特征图子集为输入。接下来6个特征图以S2中4个相邻特征图子集为输入。然后的3个以不相邻的4个特征图子集为输入。最后一个将S2中所有特征图为输入。

则：可训练参数：6×（3×25+1）+6×（4×25+1）+3×（4×25+1）+（25×6+1）=1516

连接数：10×10×1516=151600

 C3层是一个卷积层，卷积和和C1相同，不同的是C3的每个节点与S2中的多个图相连。  每个图与S2层的连接的方式如下表 所示。C3与S2中前3个图相连的卷积结构见下图.这种不对称的组合连接的方式有利于提取多种组合特征。 

![lXtRBQ.png](https://s2.ax1x.com/2020/01/15/lXtRBQ.png)

##  LeNet-5第四层：池化层S4

输入：10×10

采样区域：2×2

采样方式：4个输入相加，乘以一个可训练参数，再加上一个可训练偏置。结果通过sigmoid

采样种类：16

输出featureMap大小：5×5（10/2）

神经元数量：5×5×16=400

可训练参数：2×16=32（和的权+偏置）

连接数：16×（2×2+1）×5×5=2000

S4中每个特征图的大小是C3中特征图大小的1/4

S4层是一个下采样层，由16个5*5大小的特征图构成。特征图中的每个单元与C3中相应特征图的2*2邻域相连接，跟C1和S2之间的连接一样。S4层有32个可训练参数（每个特征图1个因子和一个偏置）和2000个连接。 

##  LeNet-5第五层：全连接层C5 

输入：S4层的全部16个单元特征map（与s4全相连）

卷积核大小：5×5

卷积核种类：120

输出featureMap大小：1×1（5-5+1）

可训练参数/连接：120×（16×5×5+1）=48120

C5层是一个全连接层。由于S4层的16个图的大小为5x5，与卷积核的大小相同，所以卷积后形成的图的大小为1x1。这里形成120个卷积结果。每个都与上一层的16个图相连。所以共有(5x5x16+1)x120 = 48120个参数，同样有48120个连接。 

##  LeNet-5第六层：全连接层F6

输入：c5 120维向量

计算方式：计算输入向量和权重向量之间的点积，再加上一个偏置，结果通过sigmoid函数

可训练参数:84*(120+1)=10164

F6层有84个单元（之所以选这个数字的原因来自于输出层的设计），与C5层全相连。有10164个可训练参数。如同经典神经网络，F6层计算输入向量和权重向量之间的点积，再加上一个偏置。然后将其传递给sigmoid函数产生单元i的一个状态。 

## LeNet-5第七层：全连接层Output

输出层由欧式径向基函数（Euclidean Radial Basis Function）单元组成，每类一个单元，每个有84个输入。换句话说，每个输出RBF单元计算输入向量和参数向量之间的欧式距离。输入离参数向量越远，RBF输出的越大。一个RBF输出可以被理解为衡量输入模式和与RBF相关联类的一个模型的匹配程度的惩罚项。用概率术语来说，RBF输出可以被理解为F6层配置空间的高斯分布的负log-likelihood。给定一个输入模式，损失函数应能使得F6的配置与RBF参数向量（即模式的期望分类）足够接近。这些单元的参数是人工选取并保持固定的（至少初始时候如此）。这些参数向量的成分被设为-1或1。虽然这些参数可以以-1和1等概率的方式任选，或者构成一个纠错码，但是被设计成一个相应字符类的7*12大小（即84）的格式化图片。这种表示对识别单独的数字不是很有用，但是对识别可打印ASCII集中的字符串很有用。

使用这种分布编码而非更常用的“1 of N”编码用于产生输出的另一个原因是，当类别比较大的时候，非分布编码的效果比较差。原因是大多数时间非分布编码的输出必须为0。这使得用sigmoid单元很难实现。另一个原因是分类器不仅用于识别字母，也用于拒绝非字母。使用分布编码的RBF更适合该目标。因为与sigmoid不同，他们在输入空间的较好限制的区域内兴奋，而非典型模式更容易落到外边。RBF参数向量起着F6层目标向量的角色。需要指出这些向量的成分是+1或-1，这正好在F6 sigmoid的范围内，因此可以防止sigmoid函数饱和。实际上，+1和-1是sigmoid函数的最大弯曲的点处。这使得F6单元运行在最大非线性范围内。必须避免sigmoid函数的饱和，因为这将会导致损失函数较慢的收敛和病态问题。

## 训练过程

这个的话建议看一些机器学习的网课，比如网易云课堂上吴恩达的课。

## 一维Lenet-5

 https://github.com/1187100546/CNN_LeNet-5_onedimension 

**参考链接**

 https://blog.csdn.net/zrh_CSDN/article/details/81267873 

 https://www.jianshu.com/p/ce609f9b5910 

 https://blog.csdn.net/qiaofangjie/article/details/16826849 

 http://blog.csdn.net/zouxy09/article/details/9993371 

 http://blog.csdn.net/celerychen2009/article/details/8973218 


