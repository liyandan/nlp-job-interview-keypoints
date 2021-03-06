# TextCNN

## 问题1：CNN相关知识

### 1.1 如何理解CNN的稀疏连接与权值共享？

参见：[https://www.cnblogs.com/LittleHann/p/6792511.html\#\_label3\_1\_2\_0](https://www.cnblogs.com/LittleHann/p/6792511.html#_label3_1_2_0)

**1. 稀疏交互（sparse interactions）-局部感受野\(local receptive fields\)**

卷积网络具有稀疏交互\(sparse interactions\)\(也叫做稀疏连接\(sparse connectivity\)或者稀疏权重\(sparse weights\)\)的特征。这里的稀疏是相对full connected NN网络而言的。

这是通过使**核的大小远小于输入的大小**来达到的。

稀疏交互还有另外一种称呼，叫局部感知野。

回想一下传统的全连接\(full-connected\) DNN，每一层的神经元都和上一层以及下一层的所有神经元建立了连接。

但是CNN不这样做，我们以图像为例，CNN只会将隐藏层\(CNN的hidden layer就是一层滤波器层/卷积层\)和图像上相同大小的一个区域进行连接。

这个输入图像的区域被称为隐藏神经元的局部感受野，它是输入像素上的一个小窗口。局部感受野中的每个神经元（个数由感受野大小而定），单独学习一个权重w，同时整个卷积核也学习一个总的偏置b。下图展示了DNN网络和CNN网络在参数个数上的对比：

![](https://raw.githubusercontent.com/anxiang1836/FigureBed/master/img/20200130165901.png)

局部感受野像一台扫描仪一样，每次按照一定的跨距\(可以是1、2..\)从左到右，从上到下移动。

笔者思考：CNN使用参数共享机制，这决定了CNN中每层中的一个卷积核只能捕获一种或者一类特征（因为卷积模版是固定的），所以一般情况下，CNN网络的每层都需要设置多个卷积Filter。

**2. 参数共享：--共享权重和偏置**

我们把深度神经网络看成是由大量简单线性函数和非线性函数组合而成的复合函数，则参数共享\(parameter sharing\)是指在一个模型的多个函数中使用相同的参数。

在传统的神经网络中，当计算一层的输出时，权重矩阵的每一个元素只使用一次，当它乘以输入的一个元素后就再也不会用到了。作为参数共享的同义词，我们可以说一个网络含有绑定的权重\(tied weights\)，因为用于一个输入的权重也会被绑定在其他的权重上。

在卷积神经网络中，核的每一个元素都作用在输入的每一位置上\(是否考虑边界像素取决于对边界决策的设计\)。卷积运算中的参数共享保证了我们只需要学习一个参数集合，而不是对于每一位置都需要学习一个单独的参数集合。

这虽然没有改变前向传播的运行时间\(仍然是 O\(k × n\)\)，但它显著地把模型的存储需求降低至k个参数，并且k通常要比m小很多个数量级。因为m和n通常有着大致相同的大小（例如图像方阵），k在实际中相对于 m × n 是很小的。因此，卷积在存储需求和统计效率方面极大地优于稠密矩阵的乘法运算。

**共享权重和偏置**指的是图像的所有局部感受野都使用同一份权重和偏置。

例如对一张 28  _28 的图像，使用 5_  5 的局部感受野，步距1，且该CNN隐层只设置一个卷积核Filter。则该CNN隐藏层总共由 （28 - 5 + 1） _\(28 - 5 + 1\) = 576 个感知野隐藏神经元组成（不考虑边缘padding，在实际项目中往往会加上边缘padding，这样卷积后张量维度不变），这很好理解，因为卷积核是逐行从左到右扫描过去的，当然。每个神经元都共享同一份w向量（5_  5个像素点）和偏置b。

在不考虑边缘padding的情况下，经过这样的卷积后，原本 28  _28 维度的输入，就会得到 24_  24 维度的输出。我们接下来来看看每个元素的计算过程，也即每个感知野神经元的计算公式。

以 5 \* 5 感受野为例，对第（k，j）个感知野隐藏神经元，输出为：

![](https://raw.githubusercontent.com/anxiang1836/FigureBed/master/img/20200130165958.png)

这里σ是神经元的激活函数（例如可以是sigmoid或者ReLU）；b是偏置的共享值；Wl,m是5\*5的权重数组；a是和这个感知野相同大小的输入矩阵\(对于第一层来说，就是像素强度值本身，如果不是第一层就是上一层激活函数的输出\)。

从公式中可以看到几点：

1. 对于同一个卷积核Filter来说，所有（k，j）感知野隐藏神经元都共享同一份参数w和偏置b；
2. 如果将卷积核的感知野看成是”响应信号“，则每个感知野神经元的计算累加公式可以看成是整个”输入激发信号（这里就是感知野大小的输入矩阵）“和”响应信号（卷积感知野）“的累加结果，和信号里的卷积概念上是一致的。

在我们这个例子中，上面的公式，重复 576 次（（28 - 5 + 1）\* \(28 - 5 + 1\) = 576），就得到一个特征图谱，即一个卷积核。

**一个卷积滤波器就是一个特征映射。**在CNN卷积网络中，我们把从输入层到隐藏层的映射称为一个特征映射，每个卷积核可以生产一个特征映射。我们把定义特征映射的权重称为共享权重，把定义特征映射的偏置称为共享偏置。共享权重和偏置合称为卷积核或滤波器。

值得注意的是，一个特征映射代表了一种“视角”或者一种“模式”，为了完成图像细节识别和提取需要多个特征映射：

![](https://raw.githubusercontent.com/anxiang1836/FigureBed/master/img/20200130170244.png)

在上图中，有3个特征映射，每个特征映射定义为一个5\*5共享权重和单个共享偏置的集合，其结果是网络能够检测3种不同的特征。

笔者思考：由于采用了参数共享机制，CNN具备处理可变大小输入的能力。这点上和RNN是类似的。但是所不同的是，RNN通过改进后的LSTM遗忘门来实现记忆模式的更新和丢弃，所以RNN更擅长处理序列数据，CNN则需要在每层网络中使用多个卷积Filter来实现多模式的同时捕获，所有CNN更擅长处理局部区块特征模式。

### 1.2 池化层（pooling）的反向传播是怎么实现的？

参见：[https://blog.csdn.net/hellocsz/article/details/88096446](https://blog.csdn.net/hellocsz/article/details/88096446)

![](https://raw.githubusercontent.com/anxiang1836/FigureBed/master/img/20200130170358.png)

### 1.3 什么是残差网络，它解决了什么问题？

参见：[https://zhuanlan.zhihu.com/p/80226180](https://zhuanlan.zhihu.com/p/80226180)

（待补充）

## TextCNN相关知识

参见：[https://www.jianshu.com/p/f69e8a306862](https://www.jianshu.com/p/f69e8a306862)

