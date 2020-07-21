# 什么是深度学习

* 机器学习是人工智能的一个分支，深度学习是机器学习的一个分支

* 人工智能的简洁定义如下：努力将通常由人类完成的智力任务自动化

* 机器学习系统是训练出来的，而不是明确地用程序编写出来的

* 给定包含预期结果的示例，机器学习将会发现执行一项数据处理任务的规则

* 进行机器学习的三要素

  * 输入数据点
  * 预期输出的示例
  * 衡量算法效果好坏的方法

* 机器学习和深度学习的核心问题在于有意义地变换数据。

* 机器学习中的学习指的是，寻找更好数据表示的自动搜索过程

* 所有机器学习算法都包括自动寻找这样一种变换:这种变换可以根据任务将数据转化为更加有用的表示

  * 机器学习算法在寻找这些变换时通常没有什么创造性，而仅仅是遍历一组预先定义好的操作，这组操作叫作假设空间(hypothesis space)

* 机器学习的技术定义

  * 在预先定义好的可能性空间中，利用反馈信号的指引来寻找输入数据的有用表示

* 深度学习

  * 是机器学习的一个分支领域：它是从数据中学习表示的一种新方法，强调从连续的层(layer)中进行学习，这些层对应于越来越有意义的表示
  * “深度学习”中的“深度”是指一系列连续的表示层。数据模型中 包含多少层，这被称为模型的深度(depth)
  * 现代深度学习 通常包含数十个甚至上百个连续的表示层，这些表示层全都是从训练数据中自动学习的
  * 在深度学习中，这些分层表示几乎总是通过叫作神经网络(neural network)的模型来学习 得到的
    * 神经网络的结构是逐层堆叠
  * 深度学习是从数据中学习表示的一种数学框架
  * 可以将深度网络看作多级信息蒸馏操作：信息穿过连续的过滤器，其纯度越来越高(即对任务的帮助越来越大)
  * 深度学习的技术定义
    * 学习数据表示的多级方法

* 深度学习的工作原理

  * 机器学习是将输入(比如图像)映射到目标(比如标签“猫”)，这一过程是通过观察许多输入和目标的示例来完成的

  * 深度神经网络通过一系列简单的数据变换(层)来实现这种输入到目标的映射，而这些数据变换都是通过观察示例学习到的

  * 神经网络中每层对输入数据所做的具体操作保存在该层的**权重**(weight)中，其本质是一 串数字

    * 每层实现的变换由其权重来参数化
    * 在这种语境下，学习的意思是为神经网络的所有层找到一组权重值，使得该网络能够将每个示例输入与其目标正确地一一对应
    * 一个深度神经网络可能包含数千万个参数。找到所有参数的正确取值可能是一项非常艰巨的任务，特别是考虑到修改某个参数值将会影响其他所有参数的行为

    ![image-20200708145422117](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708145422117.png)

    * 神经网络**损失函数**(loss function)（目标函数）的任务，衡量该输出与预期值之间的距离

      * 损失函数的输入是网络预测值与真实目标值(即希望网络输出的结果)，然后计算一个距离值，衡量该网络在这个示例上的效果好坏

        ![image-20200708145636610](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708145636610.png)

    * 深度学习的基本技巧是利用这个距离值作为反馈信号来对权重值进行微调，以降低当前示例对应的损失值

      * 这种调节由**优化器**来完成，它实现了反向传播(backpropagation)算法，这是深度学习的核心算法

        ![image-20200708145859436](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708145859436.png)

    * 训练循环

      * 一开始对神经网络的权重随机赋值，因此网络只是实现了一系列随机变换。其输出结果自然也和理想值相去甚远，相应地，损失值也很高。但随着网络处理的示例越来越多，权重值也在向正确的方向逐步微调，损失值也逐渐降低
      * 将这种循环重 复足够多的次数(通常对数千个示例进行数十次迭代)，得到的权重值可以使损失函数最小。具 有最小损失的网络，其输出值与目标值尽可能地接近，这就是训练好的网络。再次强调，这是 一个简单的机制，一旦具有足够大的规模，将会产生魔法般的效果

* 概率建模

  * 是统计学原理在数据分析中的应用，是最早的机器学习形式之一，至今仍在广泛使用。其中最有名的算法之一就是朴素贝叶斯算法
    * 朴素贝叶斯是一类基于应用贝叶斯定理的机器学习分类器，它假设输入数据的特征都是独 立的
    *  logistic 回归模型，logreg 是一种分类算法，而不是回归算法，它既简单又通用， 至今仍然很有用。面对一个数据集，数据科学家通常会首先尝试使用这个算法，以便初步熟悉手头的分类任务

* 核方法

  * 是一组分类算法，其中最有名的就是支持向量机(SVM）
    * SVM 的目标是通过在属于两个不同类别的两组数据点之间找到良好决策边界来解决分类问题
      * 决策边界可以看作一条直线或一个平面，将训练数据 划分为两块空间，分别对应于两个类别。对于新数据点的分类，你只需判断它位于决策边界的 哪一侧
    * SVM 通过两步来寻找决策边界
      * 将数据映射到一个新的高维表示，这时决策边界可以用一个超平面来表示
      * 尽量让超平面与每个类别最近的数据点之间的距离最大化，从而计算出良好决策边界(分 割超平面)，这一步叫作间隔最大化(maximizing the margin)。这样决策边界可以很好 地推广到训练数据集之外的新样本
      * SVM 很难扩展到大型数据集，并且在图像分类等感知问题上的效果也不好

* 决策树、随机森林与梯度提升机

  * 决策树
    * 是类似于流程图的结构，可以对输入数据点进行分类或根据给定输入来预测输出值
  * 随机森林算法
    * 引入了一种健壮且实用的决策树学习方法，即 首先构建许多决策树，然后将它们的输出集成在一起
  * 梯度提升机
    * 与随机森林类似，也是将弱预测模型(通常是决策树)集成的机器学习技术，使用了梯度提升方法，通过迭代地训练新模型来专门解决 之前模型的弱点，从而改进任何机器学习模型的效果

* 深度学习的变革性在于，模型可以在同一时间共同学习所有表示层，而不是依次连续学习(这被称为贪婪学习)。通过共同的特征学习，一旦模型修改某个内部特征，所有依赖于该特征的其他特征都会相应地自动调节适应，无须人为干预。一切都由单一反馈信号来监督:模型中的每一处 变化都是为了最终目标服务。这种方法比贪婪地叠加浅层模型更加强大，因为它可以通过将复杂、 抽象的表示拆解为很多个中间空间(层)来学习这些表示，每个中间空间仅仅是前一个空间的简单变换

* 深度学习从数据中进行学习时有两个基本特征

  * 通过渐进的、逐层的方式形成越来 越复杂的表示
  * 对中间这些渐进的表示共同进行学习，每一层的变化都需要同时考虑上下两层的需要

* 要想在如今的应用机器学习中取得成功，应该熟悉这两种技术

  * 梯度提升机，用于浅层学习问题
  * 深度学习，用于感知问题

* 三种技术力量在推动着机器学习的进步

  * 硬件
  * 数据集和基准
  * 算法上的改进

* 深度学习重要性质

  * 简单
    * 深度学习不需要特征工程，它将复杂的、不稳定的、工程量很大的流程替换为简 单的、端到端的可训练模型，这些模型通常只用到五六种不同的张量运算
  * 可扩展
    * 深度学习非常适合在 GPU 或 TPU 上并行计算，因此可以充分利用摩尔定律。此外，深度学习模型通过对小批量数据进行迭代来训练，因此可以在任意大小的数据集上进行训练。(唯一的瓶颈是可用的并行计算能力，而由于摩尔定律，这一限制会越来越小。)
  * 多功能与可复用
    * 与之前的许多机器学习方法不同，深度学习模型无须从头开始就可以在附加数据上进行训练，因此可用于连续在线学习，这对于大型生产模型而言是非常重要的特性

# 神经网络的数学基础

## 案例：对手写数字进行分类

```python
# 加载数据集
import keras
from keras.datasets import mnist

# 训练集: 模型将从这些数据中进行 学习
# 测试集: 对模型进行测试
(train_images, train_labels), (test_images, test_labels) = mnist.load_data()
# 图像被编码为 Numpy 数组，而标签是数字数组，取值范围为 0~9。图像和标签一一对应

train_images.shape
# (60000, 28, 28)
len(train_labels)
# 60000
train_labels
# array([5, 0, 4, ..., 5, 6, 8], dtype=uint8)

test_images.shape
# (10000, 28, 28)
len(test_labels)
# 10000
test_labels
# array([7, 2, 1, ..., 4, 5, 6], dtype=uint8)
```

```python
# 网络架构
# 首先，将训练数据(train_images 和 train_labels)输入神经网络
# 其次，网络学习将图像和标签关联在一起
# 最后，网络对 test_images 生成预测， 而我们将验证这些预测与 test_labels 中的标签是否匹配
from keras import models
from keras import layers
network = models.Sequential()
# 神经网络的核心组件是层(layer)：是一种数据处理模块，可以将它看成数据过滤器。进去一些数据，出来的数据变得更加有用
# 具体来说，层从输入数据中提取表示——我们期望这种表示有助于解决手头的问题
# 大多数深度学习都是将简单的层链接起来，从而实现渐进式的数据蒸馏(data distillation)
# 深度学习模型就像是数据处理的筛子，包含一系列越来越精细的数据过滤器(即层)
# Dense 层，它们是密集连接(也叫全连接)的神经层
network.add(layers.Dense(512, activation='relu', input_shape=(28 * 28,)))
# 一个10路 softmax 层，它将返回一个由10个概率值(总和为 1)组成的数组。 每个概率值表示当前数字图像属于10个数字类别中某一个的概率
network.add(layers.Dense(10, activation='softmax'))
# 网络包含两个 Dense 层，每层都对输入数据进行一些简单的张量运算，这些运算都包含权重张量。权重张量是该层的属性，里面保存了网络所学到的知识
```

```python
# 要想训练网络，还需要选择编译(compile)步骤的三个参数
network.compile(
    # 优化器：基于训练数据和损失函数来更新网络的机制
  	# 梯度下降的具体方法由第一个参数给定，即 rmsprop 优化器
    optimizer='rmsprop', 
    # 损失函数：网络如何衡量在训练数据上的性能，即网络如何朝着正确的方向前进
  	# 损失函数，是用于学习权重张量的反馈信号，在训练阶段应使它最小化
    # 减小损失是通过小批量随机梯度下降来实现的
    loss='categorical_crossentropy',
    # 在训练和测试过程中需要监控的指标：例如精度，即正确分类的图像所占的比例
    metrics=['accuracy']
)
```

```python
# 在开始训练之前，将对数据进行预处理，将其变换为网络要求的形状，并缩放到所有值都在 [0, 1] 区间。
# 比如，之前训练图像保存在一个 uint8 类型的数组中，其形状为 (60000, 28, 28)，取值区间为[0, 255]。
# 需要将其变换为一个float32数组，其形 状为 (60000, 28 * 28)，取值范围为 0~1
train_images = train_images.reshape((60000, 28 * 28))
train_images = train_images.astype('float32') / 255
test_images = test_images.reshape((10000, 28 * 28)) 
test_images = test_images.astype('float32') / 255
# 输入图像保存在 float32 格式的 Numpy 张量中，形状分别为 (60000, 784)(训练数据)和 (10000, 784)(测试数据)
```

```python
# 对标签进行分类编码
from keras.utils import to_categorical

train_labels = to_categorical(train_labels) 
test_labels = to_categorical(test_labels)
```

```python
# 准备开始训练网络
# 在训练数据上拟合(fit)模型
# 训练循环
# 在调用 fit 时，网络开始在训练数据上进行迭代(每个小批量包含 128 个样本)，共迭代 5 次[在所有训练数据上迭代一次叫作一个轮次(epoch)]。在每次迭代 过程中，网络会计算批量损失相对于权重的梯度，并相应地更新权重。5 轮之后，网络进行了 2345 次梯度更新(每轮 469 次)，网络损失值将变得足够小，使得网络能够以很高的精度对手 写数字进行分类
network.fit(train_images, train_labels, epochs=5, batch_size=128)
# Epoch 1/5
# 60000/60000 [=============================] - 9s - loss: 0.2524 - acc: 0.9273 
# Epoch 2/5
# 51328/60000 [=======================>.....] - ETA: 1s - loss: 0.1035 - acc: 0.9692

# 训练过程中显示了两个数字:一个是网络在训练数据上的损失(loss)，另一个是网络在训练数据上的精度(acc)。
# 在训练数据上达到了 0.989(98.9%)的精度
```

```python
# 检查一下模型在测试集上的性能
test_loss, test_acc = network.evaluate(test_images, test_labels)
print('test_acc:', test_acc)
test_acc: 0.9785
    
# 测试集精度为 97.8%，比训练集精度低不少
# 训练精度和测试精度之间的这种差距是过拟合(overfit)造成的
# 过拟合是指机器学习模型在新数据上的性能往往比在训练数据上要差
```

* 张量(输入网络的数据存储对象)
* 张量运算(层的组成要素)
* 梯度下降(可以让网络从训练样本中进行学习)

## 神经网络的数据表示

* 一般来说，当前所有机器学习系统都使用张量作为基本数据结构

* 张量

  * 是一个数据容器。它包含的数据几乎总是数值数据，因此它是数字的容器

  * 矩阵是二维张量

  * 张量是矩阵向任意维度的推广。注意， 张量的维度(dimension)通常叫作轴(axis)

  * 三个关键属性来定义

    * 轴的个数(阶)
      * 例如，3D 张量有 3 个轴，矩阵有 2 个轴
    * 形状
      * shape，这是一个整数元组，表示张量沿每个轴的维度大小(元素个数)
    * 数据类型
      * dtype，张量的类型可以是 float32、uint8、float64 等。在极少数情况下，可能会遇到字符 (char)张量
      * Numpy(以及大多数其他库)中不存在字符串张量，因为张量存储在预先分配的连续内存段中，而字符串的长度是可变的，无法用这种方式存储

    ```python
    from keras.datasets import mnist
    
    (train_images, train_labels), (test_images, test_labels) = mnist.load_data()
    # 给出张量 train_images 的轴的个数，即 ndim 属性
    print(train_images.ndim) # 3
    # 形状
    print(train_images.shape)  # (60000, 28, 28)
    # 数据类型，即 dtype 属性。
    print(train_images.dtype)  # uint8
    # 这里 train_images 是一个由 8 位整数组成的 3D 张量
    # 更确切地说，它是60000个矩阵组成的数组，每个矩阵由28×28个整数组成。每个这样的矩阵都是一张灰度图像，元素取值范围为 0~255
    
    # 显示第 4 个数字
    digit = train_images[4]                # 选择沿着第一个轴的特定数字
    import matplotlib.pyplot as plt 
    plt.imshow(digit, cmap=plt.cm.binary) 
    plt.show()
    ```

* 标量

  * 0D 张量，仅包含一个数字的张量叫作标量(scalar）

  * 在 Numpy 中，一个 float32 或 float64 的数字就是一个标量张量(或标量数组)

  * 可以用 ndim 属性来查看一个 Numpy 张量的轴的个数。标量张量有 0 个轴(ndim == 0)。张量轴的个数也叫作阶(rank)

    ```python
    >>> import numpy as np
    >>> x = np.array(12)
    >>> x
    array(12)
    >>> x.ndim 0
    # 形状为()
    ```

* 向量

  * 1D张量，数字组成的数组叫作向量(vector)。一维张量只有一个轴

    ```python
    >>> x = np.array([12, 3, 6, 14, 7]) 
    >>> x
    array([12, 3, 6, 14, 7])
    >>> x.ndim
    1
    # 形状为(5,)
    ```

  * 向量有 5 个元素，所以被称为 5D 向量
    * 5D 向量只有一个轴，沿着轴有 5 个维度，而 5D 张量有 5 个轴(沿着每个轴可能有任意个维度)
    * 维度 (dimensionality)可以表示沿着某个轴上的元素个数(比如 5D 向量)，也可以表示张量中轴的个数(比如 5D 张量)

* 矩阵

  * 2D 张量，向量组成的数组叫作矩阵(matrix)。矩阵有 2 个轴(行和列)。可以将矩阵直观地理解为数字组成的矩形网格

    * 第一个轴上的元素叫作行(row)，第二个轴上的元素叫作列(column)

    ```python
    >>> x = np.array([
      					[5, 78, 2, 34, 0],
      					[6, 79, 3, 35, 1],
    						[7, 80, 4, 36, 2]])
    >>> x.ndim 2
    # [5, 78, 2, 34, 0] 是 x 的第一行
    # [5, 6, 7] 是第一列
    # 形状为(3, 5)
    ```

* 3D 张量与更高维张量

  * 将多个矩阵组合成一个新的数组，可以得到一个 3D 张量。可以将其直观地理解为数字 组成的立方体

    ```python
    >>> x = np.array([[[5, 78, 2, 34, 0], 
                       [6, 79, 3, 35, 1],
                       [7, 80, 4, 36, 2]], 
                      [[5, 78, 2, 34, 0], 
                       [6, 79, 3, 35, 1],
    									 [7, 80, 4, 36, 2]], 
                      [[5, 78, 2, 34, 0], 
                       [6, 79, 3, 35, 1],
    								   [7, 80, 4, 36, 2]]])
    >>> x.ndim
    3
    # 形状为 (3, 3, 5)
    ```

  * 将多个 3D 张量组合成一个数组，可以创建一个 4D 张量，以此类推
  * 深度学习处理的一般 是 0D 到 4D 的张量，但处理视频数据时可能会遇到 5D 张量

* Numpy中操作张量

  * NumPy(Numerical Python) 是 Python 语言的一个扩展程序库，支持大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库

  * 使用语法 train_images[i] 来选择沿着第一个轴的特定数字

  * 选择张量的特定元素叫作张量切片

  *  Numpy 数组上的张量切片运算

    ```python
    # 选择第 10~100 个数字(不包括第 100 个)，并将其放在形状为 (90, 28,28) 的数组中
    >>> my_slice = train_images[10:100] 
    >>> print(my_slice.shape)
    (90, 28, 28)
    # 等同于
    >>> my_slice = train_images[10:100, :, :]           # : 等同于选择整个轴
    >>> my_slice.shape
    (90, 28, 28)
    >>> my_slice = train_images[10:100, 0:28, 0:28] 
    >>> my_slice.shape
    (90, 28, 28)
    # 一般来说，可以沿着每个张量轴在任意两个索引之间进行选择
    # 可以在所有图 像的右下角选出 14 像素×14 像素的区域
    my_slice = train_images[:, 14:, 14:]
    # 可以使用负数索引，它表示与当前轴终点的相对位置
    # 可以在图像中心裁剪出 14 像素×14 像素的区域
    my_slice = train_images[:, 7:-7, 7:-7]
    ```

* 数据批量的概念

  * 通常来说，深度学习中所有数据张量的第一个轴(0 轴，因为索引从 0 开始)都是样本轴(samples axis，有时也叫样本维度)

  * 此外，深度学习模型不会同时处理整个数据集，而是将数据拆分成小批量

    ```python
    # 是 MNIST 数据集的一个批量，批量大小为 128
    batch = train_images[:128]
    # 然后是下一个批量
    batch = train_images[128:256]
    # 然后是第 n 个批量
    batch = train_images[128 * n:128 * (n + 1)]
    ```

  * 对于这种批量张量，第一个轴(0 轴)叫作批量轴(batch axis)或批量维度(batch dimension)

* 现实世界中的数据张量

  * 向量数据

    * 2D 张量，形状为 (samples, features)
    * 这是最常见的数据，对于这种数据集，每个数据点都被编码为一个向量，因此一个数据批 量就被编码为 2D 张量(即向量组成的数组)，其中第一个轴是样本轴，第二个轴是特征轴
    * 人口统计数据集，其中包括每个人的年龄、邮编和收入。每个人可以表示为包含 3 个值的向量，而整个数据集包含 100 000 个人，因此可以存储在形状为 (100000, 3) 的 2D张量中

  * 时间序列数据或序列数据

    * 3D 张量，形状为 (samples, timesteps, features)。根据惯例，时间轴始终是第 2 个轴(索引为 1 的轴)

    * 当时间(或序列顺序)对于数据很重要时，应该将数据存储在带有时间轴的 3D 张量中

    *  每个样本可以被编码为一个向量序列(即 2D 张量)，因此一个数据批量就被编码为一个 3D 张量

    * 股票价格数据集。每一分钟，我们将股票的当前价格、前一分钟的最高价格和前一分钟 的最低价格保存下来。因此每分钟被编码为一个 3D 向量，整个交易日被编码为一个形 状为 (390, 3) 的 2D 张量(一个交易日有 390 分钟)，而 250 天的数据则可以保存在一个形状为 (250, 390, 3) 的 3D 张量中。这里每个样本是一天的股票数据

      ![image-20200708171201228](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708171201228.png)

  * 图像

    * 4D 张量，形状为 (samples, height, width, channels) 或 (samples, channels,height, width)

    * 通常具有三个维度:高度、宽度和颜色深度

    * 虽然灰度图像(比如 MNIST 数字图像) 只有一个颜色通道，因此可以保存在 2D 张量中，但按照惯例，图像张量始终都是 3D 张量，灰度图像的彩色通道只有一维

    * 因此，如果图像大小为 256×256，那么 128 张灰度图像组成的批量可以保存在一个形状为 (128, 256, 256, 1) 的张量中。而128张彩色图像组成的批量则可以保存在一个形状为 (128, 256, 256, 3) 的张量中

      ![image-20200708171123626](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708171123626.png)

    * 图像张量的形状有两种约定:通道在后(channels-last)的约定(在 TensorFlow 中使用)和 通道在前(channels-first)的约定(在 Theano 中使用)。Google 的 TensorFlow 机器学习框架将 颜色深度轴放在最后:(samples, height, width, color_depth)

  * 视频

    * 5D 张量，形状为 (samples, frames, height, width, channels) 或 (samples,frames, channels, height, width)
    * 视频可以看作一系列帧， 每一帧都是一张彩色图像。由于每一帧都可以保存在一个形状为 (height, width, color_ depth) 的 3D 张量中，因此一系列帧可以保存在一个形状为 (frames, height, width, color_depth) 的 4D 张量中，而不同视频组成的批量则可以保存在一个 5D 张量中，其形状为 (samples, frames, height, width, color_depth)
    * 举个例子，一个以每秒 4 帧采样的 60 秒 YouTube 视频片段，视频尺寸为 144×256，这个 视频共有 240 帧。4 个这样的视频片段组成的批量将保存在形状为 (4, 240, 144, 256, 3) 的张量中。总共有 106 168 320 个值!如果张量的数据类型(dtype)是 float32，每个值都是 32 位，那么这个张量共有 405MB。好大!你在现实生活中遇到的视频要小得多，因为它们不以 float32 格式存储，而且通常被大大压缩，比如 MPEG 格式

## 神经网络的齿轮：张量运算

* 深度神经网络学到的所有变换可以简化为数值数据张量上的一些张量运算，例如加上张量、乘以张量等

* 通过叠加 Dense 层来构建网络

  ```python
  # Keras 层的实例
  # 这个层可以理解为一个函数，输入一个 2D 张量，返回另一个 2D 张量
  keras.layers.Dense(512, activation='relu')
  # 这个函数如下所示(其中 W 是一个 2D 张量，b 是一个向量，二者都是该层的 属性)
  output = relu(dot(W, input) + b)
  # 输入张量和张量 W 之间的点积运算(dot)、
  # 得到的 2D 张量与向量 b 之间的加法运算(+)、最后的 relu 运算。relu(x) 是 max(x, 0)
  ```

* 逐元素运算

  * relu 运算和加法都是逐元素(element-wise)的运算，即该运算独立地应用于张量中的每个元素，也就是说，这些运算非常适合大规模并行实现

    ```python
    # 对逐元素 relu 运算的简单实现
    def naive_relu(x):
    	assert len(x.shape) == 2          # x 是一个 Numpy 的 2D 张量
    	x = x.copy()                      # 避免覆盖输入张量 
    	for i in range(x.shape[0]):
    		for j in range(x.shape[1]): 
     	   x[i, j] = max(x[i, j], 0)
    	return x
    # 加法采用同样的实现方法
    def naive_add(x, y):
    	assert len(x.shape) == 2          # x和y是 Numpy 的 2D 张量
      assert x.shape == y.shape
    
      x = x.copy()                      # 避免覆盖输入张量
    	for i in range(x.shape[0]):
    		for j in range(x.shape[1]):
          x[i, j] += y[i, j]
    	return x
    
    # 在实践中处理 Numpy 数组时，这些运算都是优化好的 Numpy 内置函数
    # 在 Numpy 中可以直接进行下列逐元素运算，速度非常快
    import numpy as np
    
    z=x+y                      # 逐元素的相加
    z = np.maximum(z, 0.)      # 逐元素的 relu
    ```

* 广播

  * 如果将两个形状不同的张量相加，如果没有歧义的话，较小的张量会被广播(broadcast)，以匹配较大张量的形状
    * 向较小的张量添加轴(叫作广播轴)，使其ndim与较大的张量相同
    * 将较小的张量沿着新轴重复，使其形状与较大的张量相同

  ```python
  def naive_add_matrix_and_vector(x, y):
    assert len(x.shape) == 2               # x 是一个 Numpy 的 2D 张量
    assert len(y.shape) == 1               # y 是一个 Numpy 向量
    assert x.shape[1] == y.shape[0]
  	x = x.copy()                           # 避免覆盖输入张量
  	for i in range(x.shape[0]):
  		for j in range(x.shape[1]):
        x[i, j] += y[j]
  	return x
  # 如果一个张量的形状是 (a, b, ... n, n+1, ... m)，另一个张量的形状是 (n, n+1, ... m)，那么通常可以利用广播对它们做两个张量之间的逐元素运算。广播操作会自动应用 于从a到n-1的轴
  
  # 利用广播将逐元素的 maximum 运算应用于两个形状不同的张量
  import numpy as np
  x = np.random.random((64, 3, 32, 10)) # x 是形状为 (64, 3, 32, 10) 的随机张量 
  y = np.random.random((32, 10))        # y 是形状为 (32, 10) 的随机张量
  z = np.maximum(x, y)                  # 输出 z 的形状是 (64, 3, 32, 10)，与 x 相同
  ```

* 张量点积

  * 点积运算，也叫张量积，是最常见也最有用的张量运算。与逐元素的运算不同，它将输入张量的元素合并在一起

  * 在 Numpy、Keras、Theano 和 TensorFlow 中，都是用 * 实现逐元素乘积。TensorFlow 中的 点积使用了不同的语法，但在 Numpy 和 Keras 中，都是用标准的 dot 运算符来实现点积

    ```python
    import numpy as np 
    z = np.dot(x, y)
    ```

  * 数学符号中的点(.)表示点积运算

    ```python
    z=x.y
    # 从数学的角度来看，点积运算做了什么
    # 首先看一下两个向量 x 和 y 的点积
    def naive_vector_dot(x, y):
    	assert len(x.shape) == 1
    	assert len(y.shape) == 1
    	assert x.shape[0] == y.shape[0]
    
    	z = 0.
    	for i in range(x.shape[0]):
    		z += x[i] * y[i] 
    	return z
    ```

  * 两个向量之间的点积是一个标量，而且只有元素个数相同的向量之间才能做点积

  * 可以对一个矩阵 x 和一个向量 y 做点积，返回值是一个向量，其中每个元素是 y 和 x 的每一行之间的点积

    ```python
    import numpy as np
    
    def naive_matrix_vector_dot(x, y): 
      assert len(x.shape) == 2           # x 是一个 Numpy 矩阵
      assert len(y.shape) == 1           # y 是一个 Numpy 向量
      assert x.shape[1] == y.shape[0]
      
    	z = np.zeros(x.shape[0])           # 这个运算返回一个全是 0 的向量,其形状与 x.shape[0] 相同
      for i in range(x.shape[0]):
    		for j in range(x.shape[1]): 
          z[i] += x[i, j] * y[j]
    	return z
    ```

  * 如果两个张量中有一个的 ndim 大于 1，那么 dot 运算就不再是对称的，也就是说， dot(x, y) 不等于 dot(y, x)。

  * 点积可以推广到具有任意个轴的张量。最常见的应用可能就是两个矩阵之间的点积

  * 对于两个矩阵 x 和 y，当且仅当 x.shape[1] == y.shape[0] 时，你才可以对它们做点积(dot(x, y))。得到的结果是一个形状为 (x.shape[0], y.shape[1]) 的矩阵，其元素为 x 的行与 y 的列之间的点积

    ![image-20200708181125123](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708181125123.png)

    ![image-20200708181142885](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708181142885.png)

  * 更高维的张量做点积，只要其形状匹配遵循与前面 2D 张量相同的原则

    ```python
    (a, b, c, d) . (d,) -> (a, b, c)
    (a, b, c, d) . (d, e) -> (a, b, c, e)
    ```

* 张量变形

  * 张量变形是指改变张量的行和列，以得到想要的形状。变形后的张量的元素总个数与初始张量相同

    ```python
    train_images = train_images.reshape((60000, 28 * 28))
    >>> x = np.array([[0., 1.], 
                      [2., 3.],
    									[4., 5.]]) 
    >>> print(x.shape)
    (3, 2)
    >>> x = x.reshape((6, 1)) 
    >>> x
    array([[ 0.],
    			 [ 1.], 
    			 [ 2.], 
    			 [ 3.], 
           [ 4.], 
           [ 5.]])
    >>> x = x.reshape((2, 3))
    >>> x
    array([[ 0., 1., 2.],
    			 [ 3., 4., 5.]])
    ```

  * 一种特殊的张量变形是转置(transposition)。对矩阵做转置是指将行和列互换， 使 x[i, :] 变为 x[:, i]

    ```python
    >>> x = np.zeros((300, 20))
    >>> x = np.transpose(x) 
    >>> print(x.shape)
    (20, 300)
    ```

* 张量运算的几何解释
  * 对于张量运算所操作的张量，其元素可以被解释为某种几何空间内点的坐标，因此所有的张量运算都有几何解释
  * 通常来说，仿射变换、旋转、缩放等基本的几何操作都可以表示为张量运算
  * 举个例子，要将 一个二维向量旋转 theta 角，可以通过与一个 2×2 矩阵做点积来实现，这个矩阵为 R = [u, v]，其 中 u 和 v 都是平面向量:u = [cos(theta), sin(theta)]，v = [-sin(theta), cos(theta)]

* 深度学习的几何解释
  * 神经网络完全由一系列张量运算组成，而这些张量运算都只是输入数据的几何 变换
  * 可以将神经网络解释为高维空间中非常复杂的几何变换，这种变换可以通过许多简单的步骤来实现
  * 对于三维的情况，这个思维图像是很有用
    * 想象有两张彩纸:一张红色，一张蓝色，将其中一张纸放在另一张上。现在将两张纸一起揉成小球。这个皱巴巴的纸球就是你的输入数 据，每张纸对应于分类问题中的一个类别。神经网络(或者任何机器学习模型)要做的就是找 到可以让纸球恢复平整的变换，从而能够再次让两个类别明确可分。通过深度学习，这一过程 可以用三维空间中一系列简单的变换来实现，比如你用手指对纸球做的变换，每次做一个动作
    * 让纸球恢复平整就是机器学习的内容:为复杂的、高度折叠的数据流形找到简洁的表示
    * 深度学习特别擅长这一点
      * 它将复杂的几何变换逐步分解 为一长串基本的几何变换，这与人类展开纸球所采取的策略大致相同。深度网络的每一层都通过变换使数据解开一点点——许多层堆叠在一起，可以实现非常复杂的解开过程

## 神经网络的“引擎”：基于梯度的优化

```python
# 对输入数据进行变换
output = relu(dot(W, input) + b)
```

* W 和 b 都是张量，均为该层的属性。它们被称为该层的权重(weight)或 可训练参数(trainable parameter)，分别对应 kernel 和 bias 属性。这些权重包含网络从观察 训练数据中学到的信息

* 一开始，这些权重矩阵取较小的随机值，这一步叫作随机初始化

* 当然，W 和 b 都是随机的，relu(dot(W, input) + b) 肯定不会得到任何有用的表示。虽然得到的表示是没有意义的，但这是一个起点。下一步则是根据反馈信号逐渐调节这些权重

* 这个逐渐调节的过程叫作训练，也就是机器学习中的学习

* 训练循环具体过程如下：必要时一直重复这些 步骤

  1. 抽取训练样本x和对应目标y组成的数据批量
  2. 在 x 上运行网络[这一步叫作前向传播(forward pass)]，得到预测值 y_pred
  3. 计算网络在这批数据上的损失，用于衡量y_pred和y之间的距离
  4. 更新网络的所有权重，使网络在这批数据上的损失略微下降

* 最终得到的网络在训练数据上的损失非常小，即预测值 y_pred 和预期目标 y 之间的距离非常小

* 第一步看起来非常简单，只是输入 / 输出(I/O)的代码

* 第二步和第三步仅仅是一些张量运算的应用

* 难点在于第四步:更新网络的权重。考虑网络中某个权重系数，你怎么知道这个系数应该增大还是减小，以及变化多少?

  * 一种简单的解决方案是，保持网络中其他权重不变，只考虑某个标量系数，让其尝试不同的取值。对于网络中的所有系数都要重复这一过程。但这种方法是非常低效的，因为对每个系数(系数很多，通常有上千个，有时甚至多达上百万个)都需要计算两次前向传播(计算代价很大)
  * 一种更好的方法是利用网络中所有运算都是可微(differentiable)的这一事实，计算损失相对于网络系数的梯度(gradient)，然后向梯度的反方向改变系数，从而使损失降低

* 什么是导数

  * f(x + epsilon_x) = y + a * epsilon_x
  * 只有在 x 足够接近 p 时，这个线性近似才有效
  * 斜率 a 被称为 f 在 p 点的导数
    * 如果 a 是负的，说明 x 在 p 点附近的微小变 化将导致 f(x) 减小;如果 a 是正的，那么 x 的微小变化将导致 f(x) 增大。 此外，a 的绝对值(导数大小)表示增大或减小的速度快慢
  * 对于每个可微函数 f(x)(可微的意思是“可以被求导”。例如，光滑的连续函数可以被求导)， 都存在一个导数函数 f'(x)，将 x 的值映射为 f 在该点的局部线性近似的斜率
    * 例如，cos(x) 的导数是 -sin(x)，f(x) = a * x 的导数是 f'(x) = a，等等
    * 如果想要将 x 改变一个小因子 epsilon_x，目的是将 f(x) 最小化，并且知道 f 的导数， 那么问题解决了:导数完全描述了改变 x 后 f(x) 会如何变化。如果希望减小 f(x) 的值，只 需将 x 沿着导数的反方向移动一小步

* 张量运算的导数：梯度

  * 梯度(gradient)是张量运算的导数。多元函数 是以张量作为输入的函数
  * 对于一个函数 f(x)，可以通过将 x 向导数的反方向移动一小步来减小 f(x) 的值。同样，对于张量的函数 f(W)，也可以通过将 W 向梯度的反方向移动来减小 f(W)

* 随机梯度下降

  * 函数的最小值是导数为 0 的点

  * 将这一方法应用于神经网络，就是用解析法求出最小损失函数对应的所有权重值

  * 可以通过对方程 gradient(f)(W) = 0 求解 W 来实现这一方法

  * 沿着梯度的反方向更新权重，损失每次都会变小一点

    1. 抽取训练样本x和对应目标y组成的数据批量
    2. 在x上运行网络，得到预测值y_pred
    3. 计算网络在这批数据上的损失，用于衡量y_pred和y之间的距离
    4. 计算损失相对于网络参数的梯度[一次反向传播(backward pass)]
    5. 将参数沿着梯度的反方向移动一点，比如 W -= step * gradient，从而使这批数据上的损失减小一点。

  * 该方法叫作小批量随机梯度下降（小批量 SGD）

    * 随机(stochastic)是指每批数据都是随机抽取的

      ![image-20200708200342095](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708200342095.png)

      * 直观上来看，为 step 因子选取合适的值是很重要的。如果取值太小，则沿着 曲线的下降需要很多次迭代，而且可能会陷入局部极小点。如果取值太大，则更新权重值之后 可能会出现在曲线上完全随机的位置

      * 注意，小批量 SGD 算法的一个变体是每次迭代时只抽取一个样本和目标，而不是抽取一批数据。这叫作真 SGD(有别于小批量 SGD)。还有另一种极端，每一次迭代都在所有数据上 运行，这叫作批量 SGD。这样做的话，每次更新都更加准确，但计算代价也高得多。这两个极 端之间的有效折中则是选择合理的批量大小

      * 神经网络的每一个权重参数都是空间中的一个自由维度，网络中可能包含数万个甚至上百万个参数维度

        ![image-20200708200451032](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708200451032.png)

      * SGD 还有多种变体，其区别在于计算下一次权重更新时还要考虑上一次权重更新， 而不是仅仅考虑当前梯度值，比如带动量的 SGD、Adagrad、RMSProp 等变体。这些变体被称 为优化方法(optimization method)或优化器(optimizer)

        * 动量解决了 SGD 的两个问题:收敛速度和局部极小点

        ![image-20200708200611775](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708200611775.png)

      * 局部极小点附近，向左移动和向右移动都会导致损失值增大。如果使用小学习率的 SGD 进行优化，那么优化过程可能会陷入局部极小点，导致无法找到全局最小点

      * 使用动量方法可以避免这样的问题

        * 将优化过程想象成一个小球从损失函数曲线上滚下来。如果小球的动量足够大，那么它不会卡在峡谷里，最终会到达全局最小点。

        * 动量方法的实现过程是每一步都移动小球，不仅要考虑当 前的斜率值(当前的加速度)，还要考虑当前的速度(来自于之前的加速度)。这在实践中的是指， 更新参数 w 不仅要考虑当前的梯度值，还要考虑上一次的参数更新

          ```python
          past_velocity = 0.
          momentum = 0.1    # 不变的动量因子 
          while loss > 0.01:  # 优化循环
          	w, loss, gradient = get_current_parameters()
          	velocity = past_velocity * momentum - learning_rate * gradient 
            w = w + momentum * velocity - learning_rate * gradient 
            past_velocity = velocity
          	update_parameter(w)
          ```

* 链式求导:反向传播算法

  * 假设函数是可微的，因此可以明确计算其导数。在实践中，神经网络函数包含许多连接在一起的张量运算，每个运算都有简单的、已知的导数。例如，下面这个网络f包含 3个张量运算a、b和c，还有 3 个权重矩阵W1、W2和W3
  * f(W1, W2, W3) = a(W1, b(W2, c(W3)))
  * 根据微积分的知识，这种函数链可以利用下面这个恒等式进行求导，它称为链式法则(chain rule)：(f(g(x)))' = f'(g(x)) * g'(x)
  * 将链式法则应用于神经网络梯度值的计算，得到的算法叫作反向传播
  * 反向传播从最终损失值开始，从最顶层反向作用至最底层，利用链式法则计算每个参数对损失值的贡献大小
  * 现在以及未来数年，人们将使用能够进行符号微分(symbolic differentiation)的现代框架来实现神经网络，比如 TensorFlow
    * 给定一个运算链，并且已知每个运算的导数，这些框架就可以利用链式法则来计算这个运算链的梯度函数，将网络参数值映射为梯度值
    * 对于这样的函数，反向传播就简化为调用这个梯度函数。由于符号微分的出现，无须手动实现反向传播算法
    * 只需充分理解基于梯度的优化方法的工作原理

## 小结

* 学习是指找到一组模型参数，使得在给定的训练数据样本和对应目标值上的损失函数最小化
* 学习的过程：随机选取包含数据样本及其目标值的批量，并计算批量损失相对于网络参 数的梯度。随后将网络参数沿着梯度的反方向稍稍移动(移动距离由学习率指定)
* 整个学习过程之所以能够实现，是因为神经网络是一系列可微分的张量运算，因此可以利用求导的链式法则来得到梯度函数，这个函数将当前参数和当前数据批量映射为一个 梯度值
* 损失是在训练过程中需要最小化的量，因此，它应该能够衡量当前任务是否已成功解决
* 优化器是使用损失梯度更新参数的具体方式，比如 RMSProp 优化器、带动量的随机梯度下降(SGD)等

# 神经网络入门

## 神经网络剖析

* 层，多个层组合成网络(或模型)

* 输入数据和相应的目标

* 损失函数，即用于学习的反馈信号

* 优化器，决定学习过程如何进行

  ![image-20200708202425111](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708202425111.png)

  * ==多个层链接在一起组成了网络，将输入数据映射为预测值。然后损失函数将这些预测值与目标进行比较，得到损失值，用于衡量网络预 测值与预期结果的匹配程度。优化器使用这个损失值来更新网络的权重==

* 层：深度学习的基础组件

  * 神经网络的基本数据结构是层

  * ==层是一个数据处理模块，将一个或多个输入张量转换为一个或多个输出张量==

  * 有些层是无状态的，但大多数的层是有状态的， 即层的权重

    * 权重是利用随机梯度下降学到的一个或多个张量，其中包含网络的知识

  * 不同的张量格式与不同的数据处理类型需要用到不同的层

    * 简单的向量数据保存在 形状为 (samples, features) 的 2D 张量中，通常用密集连接层[densely connected layer，也 叫全连接层(fully connected layer)或密集层(dense layer)，对应于 Keras 的 Dense 类]来处理
    * 序列数据保存在形状为 (samples, timesteps, features) 的 3D 张量中，通常用循环层(recurrent layer，比如 Keras 的 LSTM 层)来处理
    * 图像数据保存在 4D 张量中，通常用二维卷积层(Keras 的 Conv2D)来处理

  * 可以将层看作深度学习的乐高积木，Keras 等框架则将这种比喻具体化

    * 在 Keras 中，构建深度学习模型就是将相互兼容的多个层拼接在一起，以建立有用的数据变换流程

    * 这里层兼容性(layer compatibility)具体指的是每一层只接受特定形状的输入张量，并返回特定形状的输出张量

      ```python
      from keras import models
      from keras import layers
      
      # 创建了一个层，只接受第一个维度大小为 784 的 2D 张量(第 0 轴是批量维度，其大小没有指定，因此可以任意取值)作为输入。这个层将返回一个张量，第一个维度的大小变成了 32。
      # 这个层后面只能连接一个接受 32 维向量作为输入的层
      # 使用 Keras 时，无须担心兼容性，因为向模型中添加的层都会自动匹配输入层的形状
      model = models.Sequential()
      model.add(layers.Dense(32, input_shape=(784,)))    # 有 32 个输出单元的密集层
      # 第二层没有输入形状(input_shape)的参数，相反，它可以自动推导出输入形状等 于上一层的输出形状
      model.add(layers.Dense(32))
      ```

* 模型：层构成的网络
  * ==深度学习模型是层构成的有向无环图==
    * 最常见的例子就是层的线性堆叠，将单一输入映射 为单一输出
  * 一些常见的网络拓扑结构
    * 双分支(two-branch)网络
    * 多头(multihead)网络
    * Inception 模块
  * 网络的拓扑结构定义了一个假设空间
    * 机器学习 的定义:“在预先定义好的可能性空间中，利用反馈信号的指引来寻找输入数据的有用表示。”
    * 选定了网络拓扑结构，意味着将可能性空间(假设空间)限定为一系列特定的张量运算，将输入数据映射为输出数据。然后，需要为这些张量运算的权重张量找到一组合适的值
* 损失函数与优化器:配置学习过程的关键
  * 网络架构的参数
    * 损失函数(目标函数)——在训练过程中需要将其最小化。它能够衡量当前任务是否已成功完成
    * 优化器——决定如何基于损失函数对网络进行更新。它执行的是随机梯度下降(SGD)的某个变体
  * 具有多个输出的神经网络可能具有多个损失函数(每个输出对应一个损失函数)。但是，梯度下降过程必须基于单个标量损失值。因此，对于具有多个损失函数的网络，需要将所有损失函数取平均，变为一个标量值
  * 选择正确的目标函数对解决问题是非常重要的。网络的目的是使损失尽可能最小化，因此， 如果目标函数与成功完成当前任务不完全相关，那么网络最终得到的结果可能会不符合你的预期
  * 对于分类、回归、序列预测等常见问题，可以遵循一些简单的指导原则来选择正确的损失函数
    * 对于二分类问题，可以使用二元交叉熵损失函数
    * 对于多分类问题，可以用分类交叉熵损失函数
    * 对于回归问题，可以用均方误差损失函数
    * 对于序列学习问题，可以用联结主义时序分类损失函数
    * 只有在面对真正全新的研究问题时，才需要自主开发目标函数

## Keras简介

* Keras 是一个 Python 深度学习框架，可以方便地定 义和训练几乎所有类型的深度学习模型。最开始是为研究人员开发的，其目的在于快速 实验
* 重要特性
  * 相同的代码可以在 CPU 或 GPU 上无缝切换运行
  * 具有用户友好的 API，便于快速开发深度学习模型的原型
  * 内置支持卷积网络(用于计算机视觉)、循环网络(用于序列处理)以及二者的任意组合
  * 支持任意网络架构:多输入或多输出模型、层共享、模型共享等。
    * Keras能够构建任意深度学习模型，无论是生成式对抗网络还是神经图灵机
* Keras 是一个模型级(model-level)的库，为开发深度学习模型提供了高层次的构建模块。 它不处理张量操作、求微分等低层次的运算。相反，它依赖于一个专门的、高度优化的张量库来完成这些运算，这个张量库就是 Keras 的后端引擎
  * Keras 有三个后端实现:TensorFlow 后端、 Theano 后端和微软认知工具包(CNTK，Microsoft cognitive toolkit)后端
  * 通过 TensorFlow(或 Theano、CNTK)，Keras 可以在 CPU 和 GPU 上无缝运行。在 CPU 上运行 时，TensorFlow 本身封装了一个低层次的张量运算库，叫作 Eigen;在 GPU 上运行时，TensorFlow 封装了一个高度优化的深度学习运算库，叫作 NVIDIA CUDA 深度神经网络库(cuDNN)。
* 使用Keras开发：概述
  * 典型的 Keras 工作流程
    1. 定义训练数据:输入张量和目标张量
    2. 定义层组成的网络(或模型)，将输入映射到目标
       * 定义模型有两种方法：一种是使用 Sequential 类(仅用于层的线性堆叠，这是目前最常见的网络架构)，另一种是函数式 API(functional API，用于层组成的有向无环图，可以构建任意形式的架构)
       * 利用函数式 API，可以操纵模型处理的数据张量，并将层应用于这个张量，就好像这些层是函数一样
    3. 配置学习过程（编译）:指定模型使用的优化器和损失函数，以及训练过程中想要监控的指标
    4. 调用模型的fit方法在训练数据上进行迭代

## 电影评论分类：二分类问题

*  根据电影评论的文字内容将其划分为正面或负面

* 加载imdb数据集

  ```python
  from keras.datasets import imdb
  # 参数 num_words=10000 的意思是仅保留训练数据中前 10 000 个最常出现的单词
  # train_data 和 test_data 这两个变量都是评论组成的列表，每条评论又是单词索引组成的列表(表示一系列单词)
  # train_labels 和 test_labels 都是 0 和 1 组成的列表，其中 0 代表负面(negative)，1 代表正面(positive)
  (train_data, train_labels), (test_data, test_labels) = imdb.load_data( num_words=10000)
  >>> train_data[0]
  [1, 14, 22, 16, ..., 178, 32]
  >>> train_labels[0]
  1
  # 由于限定为前 10 000 个最常见的单词，单词索引都不会超过 10 000
  >>> max([max(sequence) for sequence in train_data]) 
  9999
  # 可以将某条评论迅速解码为英文单词
  word_index = imdb.get_word_index() # word_index 是一个将单词映射为整数索引的字典 
  reverse_word_index = dict(
    # 键值颠倒，将整数索引映射为单词
  	[(value, key) for (key, value) in word_index.items()]) 
  decoded_review = ' '.join(
    # 将评论解码。注意，索引减去了 3，因为 0、1、2是为“padding”(填充)、“start of sequence”(序列开始)、“unknown”(未知词)分别保留的索引
    [reverse_word_index.get(i - 3, '?') for i in train_data[0]])
  ```

* 准备数据

  * 不能将整数序列直接输入神经网络，需要将列表转换为张量，转换方法有以下两种

    * 填充列表，使其具有相同的长度，再将列表转换成形状为 (samples, word_indices) 的整数张量，然后网络第一层使用能处理这种整数张量的层
    * 对列表进行 one-hot 编码，将其转换为 0 和 1 组成的向量
      * 举个例子，序列 [3, 5] 将会被转换为 10 000 维向量，只有索引为 3 和 5 的元素是 1，其余元素都是 0。然后网络第一层可以用 Dense 层，它能够处理浮点数向量数据

    ```python
    # 将整数序列编码为二进制矩阵
    import numpy as np
    def vectorize_sequences(sequences, dimension=10000): 
      # 创建一个形状为 (len(sequences), dimension) 的零矩阵
      results = np.zeros((len(sequences), dimension))
      for i, sequence in enumerate(sequences):
        # 将 results[i] 的指定索引设为 1
        results[i, sequence] = 1.
    	return results
    # 将训练数据向量化
    x_train = vectorize_sequences(train_data) 
    # 将测试数据向量化
    x_test = vectorize_sequences(test_data)
    
    # 样本
    >>> x_train[0]
    array([ 0., 1., 1., ..., 0., 0., 0.])
    # 将标签向量化
    y_train = np.asarray(train_labels).astype('float32') 
    y_test = np.asarray(test_labels).astype('float32')
    
    # 现在可以将数据输入到神经网络中
    ```

* 构建网络

  * 输入数据是向量，而标签是标量(1 和 0)，这是会遇到的最简单的情况

    * 有一类网络在这种问题上表现很好，就是带有 relu 激活的全连接层(Dense)的简单堆叠，比如 Dense(16, activation='relu')

      * 传入 Dense 层的参数(16)是该层隐藏单元的个数。一个隐藏单元(hidden unit)是该层表示空间的一个维度
      * 每个带有 relu 激活的 Dense 层都实现了下列张量运算
        * output = relu(dot(W, input) + b)
        * 16 个隐藏单元对应的权重矩阵 W 的形状为 (input_dimension, 16)，与 W 做点积相当于将输入数据投影到 16 维表示空间中(然后再加上偏置向量 b 并应用 relu 运算)。
        * 可以将表示空间的维度直观地理解为“网络学习内部表示时所拥有的自由度”。隐藏单元越多(即更高维的表示空间)，网络越能够学到更加复杂的表示，但网络的计算代价也变得更大，而且可能会导致学到不好的模式(这种模式会提高训练数据上的性能，但不会提高测试数据上的性能)
      * 对于这种 Dense 层的堆叠，需要确定以下两个关键架构
        * 网络有多少层
        * 每层有多少个隐藏单元

      ![image-20200708230111099](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200708230111099.png)

    ```python
    from keras import models
    from keras import layers
    
    model = models.Sequential()
    # 两个中间层，每层都有 16 个隐藏单元
    # 中间层使用 relu 作为激活函数
    # relu(整流线性单元)函数将所有负值归零
    model.add(layers.Dense(16, activation='relu', input_shape=(10000,))) 
    model.add(layers.Dense(16, activation='relu')) 
    # 第三层输出一个标量，预测当前评论的情感
    # 最后一层使用 sigmoid 激活以输出一个 0~1 范围内的概率值(表示样本的目标值等于 1 的可能性，即评论为正面的可能性)
    # sigmoid 函数则将任意值“压缩”到 [0, 1] 区间内，其输出值可以看作概率值
    model.add(layers.Dense(1, activation='sigmoid'))
    ```

  * 什么是激活函数?为什么要使用激活函数?
    * 如果没有 relu 等激活函数(也叫非线性)，Dense 层将只包含两个线性运算——点积和加法：output = dot(W, input) + b
    * 这样 Dense 层就只能学习输入数据的线性变换(仿射变换):该层的假设空间是从输 入数据到 16 位空间所有可能的线性变换集合。这种假设空间非常有限，无法利用多个表示层的优势，因为多个线性层堆叠实现的仍是线性运算，添加层数并不会扩展假设空间
    * 为了得到更丰富的假设空间，从而充分利用多层表示的优势，你需要添加非线性或激活函数。relu 是深度学习中最常用的激活函数，但还有许多其他函数可选，它们都有类似 的奇怪名称，比如 prelu、elu 等
  * 面对的是一个二分类问题，网络输出是一 个概率值(网络最后一层使用 sigmoid 激活函数，仅包含一个单元)，那么最好使用 binary_ crossentropy(二元交叉熵)损失
    
    * 对于输出概率值的模型，交叉熵(crossentropy)往往是最好的选择。交叉熵是来自于信息论领域的概念，用于衡量概率分布之间的距离，在这个例子中就 是真实分布与预测值之间的距离

* 编译模型

  ```python
  model.compile(
    # Keras内置
    optimizer='rmsprop', 
    loss='binary_ crossentropy',
  	metrics=['accuracy'])
  
  # 希望配置自定义优化器的 参数，或者传入自定义的损失函数或指标函数。前者可通过向 optimizer 参数传入一个优化器 类实例来实现;后者可通过向 loss 和 metrics 参数传入函数对象来实现
  from keras import optimizers
  from keras import losses 
  from keras import metrics
  
  model.compile(
    # 配置优化器
    optimizer=optimizers.RMSprop(lr=0.001),
    # 使用自定义的损失和指标
    loss=losses.binary_crossentropy,
  	metrics=[metrics.binary_accuracy])
  ```

* 验证方法

  ```python
  # 留出验证集
  x_val = x_train[:10000]
  partial_x_train = x_train[10000:]
  y_val = y_train[:10000] 
  partial_y_train = y_train[10000:]
  
  # 训练模型
  model.compile(
    optimizer='rmsprop', 
    loss='binary_crossentropy',
  	metrics=['acc'])
  history = model.fit(
    partial_x_train, 
    partial_y_train,
    # 将模型训练 20 个轮次(即对 x_train 和 y_train 两 个张量中的所有样本进行 20 次迭代)
  	epochs=20,
    # 使用 512 个样本组成的小批量
  	batch_size=512,       
    # 监控在留出的 10 000 个样本上的损失 和精度
    # 可以通过将验证数据传入 validation_data 参数来完成
    validation_data=(x_val, y_val))
  
  # 调用 model.fit() 返回了一个 History 对象。这个对象有一个成员 history，它是一个字典，包含训练过程中的所有数据
  >>> history_dict = history.history
  >>> history_dict.keys()
  dict_keys(['val_acc', 'acc', 'val_loss', 'loss'])
  
  # 使用 Matplotlib 在同一张图上绘制训练损失和验证损失，以及训练精度和验证精度
  import matplotlib.pyplot as plt
  
  history_dict = history.history 
  loss_values = history_dict['loss'] 
  val_loss_values = history_dict['val_loss']
  epochs = range(1, len(loss_values) + 1)
  
  # 绘制训练损失和验证损失
  # 'bo' 表示蓝色圆点
  plt.plot(epochs, loss_values, 'bo', label='Training loss')    
  # 'b' 表示蓝色实线
  plt.plot(epochs, val_loss_values, 'b', label='Validation loss') 
  plt.title('Training and validation loss')
  plt.xlabel('Epochs')
  plt.ylabel('Loss') 
  plt.legend()
  plt.show()
  
  # 绘制训练精度和验证精度
  plt.clf()              # 清空图像
  acc = history_dict['acc'] 
  val_acc = history_dict['val_acc']
  plt.plot(epochs, acc, 'bo', label='Training acc') 
  plt.plot(epochs, val_acc, 'b', label='Validation acc') 
  plt.title('Training and validation accuracy') 
  plt.xlabel('Epochs')
  plt.ylabel('Accuracy') 
  plt.legend()
  plt.show()
  
  # 模型在训练数据上的表现越来越好， 但在前所未见的数据上不一定表现得越来越好
  # 过拟合(overfit)：在第二轮之后，对训练数据过度优化，最终学到的表示仅针对于训练数据，无法泛化到训练集之外的数据
  ```

![image-20200709101713032](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709101713032.png)

![image-20200709101728883](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709101728883.png)

* 从头开始重新训练一个模型，epochs=4

  ```python
  # 从头开始训练一个新的网络，训练 4 轮，然后在测试数据上评估模型
  model = models.Sequential()
  model.add(layers.Dense(16, activation='relu', input_shape=(10000,)))
  model.add(layers.Dense(16, activation='relu')) 
  model.add(layers.Dense(1, activation='sigmoid'))
  model.compile(optimizer='rmsprop', loss='binary_crossentropy',
  metrics=['accuracy'])
  model.fit(x_train, y_train, epochs=4, batch_size=512) 
  results = model.evaluate(x_test, y_test)
  # 最终结果如下所示
  >>> results
  [0.2929924130630493, 0.88327999999999995]
  # 这种相当简单的方法得到了 88% 的精度。利用最先进的方法，应该能够得到接近 95% 的 精度
  ```

* 使用训练好的网络在新数据上生成预测结果

  * 训练好网络之后，希望将其用于实践。可以用 predict 方法来得到评论为正面的可能性大小

    ```python
    >>> model.predict(x_test) array([[ 0.98006207]
    [ 0.99758697]
    [ 0.99975556]
    ...,
    [ 0.82167041]
    [ 0.02885115]
    [ 0.65371346]], dtype=float32)
    ```

* 进一步的实验
  * 前面使用了两个隐藏层。可以尝试使用一个或三个隐藏层，然后观察对验证精度和测试精度的影响
  * 尝试使用更多或更少的隐藏单元，比如 32 个、64 个等
  * 尝试使用 mse 损失函数代替 binary_crossentropy
  * 尝试使用 tanh 激活(这种激活在神经网络早期非常流行)代替 relu
* 小结
  * 通常需要对原始数据进行大量预处理，以便将其转换为张量输入到神经网络中
    * 单词序列可以编码为二进制向量，但也有其他编码方式
  * 带有 relu 激活的 Dense 层堆叠，可以解决很多种问题(包括情感分类)，可能会经常用到这种模型
  * 对于二分类问题(两个输出类别)，网络的最后一层应该是只有一个单元并使用 sigmoid激活的 Dense 层，网络输出应该是 0~1 范围内的标量，表示概率值
  * 对于二分类问题的 sigmoid 标量输出，应该使用 binary_crossentropy 损失函数
  * 无论你的问题是什么，rmsprop 优化器通常都是足够好的选择
  * 随着神经网络在训练数据上的表现越来越好，模型最终会过拟合，并在前所未见的数据上得到越来越差的结果。一定要一直监控模型在训练集之外的数据上的性能

## 新闻分类：多分类问题

* 单标签、多分类：每个数据点只能划分到一个类别

* 多标签、多分类：如果每个数据点可以划分到多个类别(主题)

* 数据集

  ```python
  # 加载路透社数据集
  from keras.datasets import reuters
  
  (train_data, train_labels), (test_data, test_labels) = reuters.load_data( num_words=10000)
  >>> len(train_data) 
  8982
  >>> len(test_data) 
  2246
  # 每个样本都是一个整数列表(表示单词索引)
  >>> train_data[10]
  [1, 245, 273, 207, 156, 53, 74, 160, 26, 14, 46, 296, 26, 39, 74, 2979, 3554, 14, 46, 4689, 4329, 86, 61, 3499, 4795, 14, 61, 451, 4329, 17, 12]
  # 可将索引解码为单词
  word_index = reuters.get_word_index()
  reverse_word_index = dict([(value, key) for (key, value) in word_index.items()]) 
  # 索引减去了 3，因为 0、1、2 是为“padding”(填充)、“start of sequence”(序列开始)、“unknown”(未知词)分别保留的索引
  decoded_newswire = ' '.join([reverse_word_index.get(i - 3, '?') for i in train_data[0]])
  
  # 样本对应的标签是一个 0~45 范围内的整数，即话题索引编号
  >>> train_labels[10] 
  3
  ```

* 准备数据

  ```python
  # 将数据向量化
  # 编码数据
  
  import numpy as np
  
  def vectorize_sequences(sequences, dimension=10000): 
    results = np.zeros((len(sequences), dimension)) 
    for i, sequence in enumerate(sequences):
      results[i, sequence] = 1.
  	return results
  
  # 将训练数据向量化
  x_train = vectorize_sequences(train_data)     
  # 将测试数据向量化
  x_test = vectorize_sequences(test_data)
  
  # 将标签向量化有两种方法
  # 可以将标签列表转换为整数张量，或者使用 one-hot 编码
  # one-hot 编码是分类数据广泛使用的一种格式，也叫分类编码
  # 标签的 one-hot 编码就是将每个标签表示为全零向量， 只有标签索引对应的元素为 1
  def to_one_hot(labels, dimension=46):
    results = np.zeros((len(labels), dimension)) 
    for i, label in enumerate(labels):
      results[i, label] = 1. 
  	return results
  
  # 将训练标签向量化
  one_hot_train_labels = to_one_hot(train_labels)
  # 将测试标签向量化
  one_hot_test_labels = to_one_hot(test_labels)
  # Keras 内置方法可以实现这个操作
  from keras.utils.np_utils import to_categorical
  
  one_hot_train_labels = to_categorical(train_labels)
  one_hot_test_labels = to_categorical(test_labels)
  ```

* 构建网络

  *  Dense 层的堆叠，每层只能访问上一层输出的信息。如果某一层丢失了与 分类问题相关的一些信息，那么这些信息无法被后面的层找回，也就是说，每一层都可能成为 信息瓶颈。维度较小的层可能成为信息瓶颈，永久地丢失相关信息

    ```python
    # 将使用维度更大的层，包含 64 个单元
    # 模型定义
    from keras import models
    from keras import layers
    
    model = models.Sequential()
    model.add(layers.Dense(64, activation='relu', input_shape=(10000,))) 
    model.add(layers.Dense(64, activation='relu')) 
    # 网络的最后一层是大小为 46 的 Dense 层。这意味着，对于每个输入样本，网络都会输出一个 46 维向量。这个向量的每个元素(即每个维度)代表不同的输出类别
    # 最后一层使用了 softmax 激活，网络将输出在 46 个不同输出类别上的概率分布——对于每一个输入样本，网络都会输出一个 46 维向量，其中 output[i] 是样本属于第 i 个类别的概率。46 个概率的总和为 1。
    model.add(layers.Dense(46, activation='softmax'))
    
    # 编译模型
    # 最好的损失函数是 categorical_crossentropy(分类交叉熵)。它用于 衡量两个概率分布之间的距离，这里两个概率分布分别是网络输出的概率分布和标签的真实分布。通过将这两个分布的距离最小化，训练网络可使输出结果尽可能接近真实标签
    model.compile(
      optimizer='rmsprop', 
      loss='categorical_crossentropy',
      metrics=['accuracy'])
    ```

* 验证方法

  ```python
  # 留出验证集
  x_val = x_train[:1000]
  partial_x_train = x_train[1000:]
  
  y_val = one_hot_train_labels[:1000] 
  partial_y_train = one_hot_train_labels[1000:]
  
  # 训练模型
  history = model.fit(
    partial_x_train, 
    partial_y_train,
    epochs=20,
    batch_size=512, 
    validation_data=(x_val, y_val))
  ```

  ![image-20200709101840714](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709101840714.png)

  ![image-20200709101853795](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709101853795.png)

  ```python
  # 绘制训练损失和验证损失
  import matplotlib.pyplot as plt
  
  loss = history.history['loss'] 
  val_loss = history.history['val_loss']
  epochs = range(1, len(loss) + 1)
  plt.plot(epochs, loss, 'bo', label='Training loss') 
  plt.plot(epochs, val_loss, 'b', label='Validation loss') 
  plt.title('Training and validation loss')
  plt.xlabel('Epochs')
  plt.ylabel('Loss') 
  plt.legend()
  plt.show()
  
  # 绘制训练精度和验证精度
  plt.clf()                          # 清空图像
  acc = history.history['acc']
  val_acc = history.history['val_acc']
  plt.plot(epochs, acc, 'bo', label='Training acc') 
  plt.plot(epochs, val_acc, 'b', label='Validation acc') 
  plt.title('Training and validation accuracy') 
  plt.xlabel('Epochs')
  plt.ylabel('Accuracy') plt.legend()
  plt.show()
  ```

  * 网络在训练 9 轮后开始过拟合。从头开始训练一个新网络，共 9 个轮次（epochs=9,），然后在测试 集上评估模型

    ```python
    model = models.Sequential()
    model.add(layers.Dense(64, activation='relu', input_shape=(10000,))) 
    model.add(layers.Dense(64, activation='relu')) 
    model.add(layers.Dense(46, activation='softmax'))
    model.compile(
      optimizer='rmsprop', 
      loss='categorical_crossentropy',
    	metrics=['accuracy']) 
    model.fit(
      partial_x_train,
    	partial_y_train,
    	epochs=9,
    	batch_size=512, 
      validation_data=(x_val, y_val))
    results = model.evaluate(x_test, one_hot_test_labels)
    
    # 最终结果如下。
    >>> results
    [0.9565213431445807, 0.79697239536954589]
    # 这种方法可以得到约 80% 的精度。对于平衡的二分类问题，完全随机的分类器能够得到 50% 的精度。但在这个例子中，完全随机的精度约为 19%，所以上述结果相当不错，至少和随 机的基准比起来还不错
    >>> import copy
    >>> test_labels_copy = copy.copy(test_labels)
    >>> np.random.shuffle(test_labels_copy)
    >>> hits_array = np.array(test_labels) == np.array(test_labels_copy) 
    >>> float(np.sum(hits_array)) / len(test_labels) 
    0.18655387355298308
    ```

* 在新数据上生成预测结果

  * 模型实例的 predict 方法返回了在 46 个主题上的概率分布。我们对所有测试数据生成主题预测

    ```python
    # predictions中的每个元素的都是长度为46的向量
    predictions = model.predict(x_test)
    >>> predictions[0].shape
    (46,)
    # 这个向量的所有元素总和为 1
    >>> np.sum(predictions[0]) 
    1.0
    # 最大的元素就是预测类别，即概率最大的类别
    >>> np.argmax(predictions[0]) 
    4
    ```

* 处理标签和损失的另一种方法

  ```python
  # 另一种编码标签的方法，就是将其转换为整数张量，唯一需要改变的是损失函数的选择
  y_train = np.array(train_labels)
  y_test = np.array(test_labels)
  
  # 使用的损失函数categorical_crossentropy，标签应该遵循分类编码
  # 对于整数标签，应该使用sparse_categorical_crossentropy
  model.compile(optimizer='rmsprop', loss='sparse_categorical_crossentropy', metrics=['acc'])
  # 新的损失函数在数学上与 categorical_crossentropy 完全相同，二者只是接口不同
  ```

* 中间层维度足够大的重要性

  * 最终输出是 46 维的，因此中间层的隐藏单元个数不应该比 46 小太多。现在来看一下，如果中间层的维度远远小于 46(比如 4 维)，造成了信息瓶颈，那么会发生什么?

    ```python
    # 具有信息瓶颈的模型
    ...
    model.add(layers.Dense(4, activation='relu'))
    ...
    # 现在网络的验证精度最大约为 71%，比前面下降了 8%。导致这一下降的主要原因在于，试图将大量信息(这些信息足够恢复 46 个类别的分割超平面)压缩到维度很小的中间空间。网 络能够将大部分必要信息塞入这个四维表示中，但并不是全部信息
    ```

* 进一步试验

  * 尝试使用更多或更少的隐藏单元，比如 32 个、128 个等
  * 前面使用了两个隐藏层，现在尝试使用一个或三个隐藏层

* 小结

  * 如果要对 *N* 个类别的数据点进行分类，网络的最后一层应该是大小为 *N* 的 Dense 层
  * 对于单标签、多分类问题，网络的最后一层应该使用 softmax 激活，这样可以输出在 N个输出类别上的概率分布
  * 这种问题的损失函数几乎总是应该使用分类交叉熵。它将网络输出的概率分布与目标的真实分布之间的距离最小化
  * 处理多分类问题的标签有两种方法
    * 通过分类编码(也叫 one-hot 编码)对标签进行编码，然后使用categorical_ crossentropy 作为损失函数
    * 将标签编码为整数，然后使用 sparse_categorical_crossentropy 损失函数

  * 如果需要将数据划分到许多类别中，应该避免使用太小的中间层，以免在网络中造成信息瓶颈

## 预测房价：回归问题

* 分类问题，其目标是预测输入数据点所对应的单一离散的标签

* 回归问题，它预测一个连续值而不是离散的标签

* 数据集

  ```python
  from keras.datasets import boston_housing
  
  (train_data, train_targets), (test_data, test_targets) = boston_housing.load_data()
  # 有 404 个训练样本和 102 个测试样本，每个样本都有 13 个数值特征，比如 人均犯罪率、每个住宅的平均房间数、高速公路可达性等
  >>> train_data.shape (404, 13)
  >>> test_data.shape (102, 13)
  # 目标是房屋价格的中位数，单位是千美元。
  >>> train_targets
  array([ 15.2, 42.3, 50. ... 19.4, 19.4, 29.1])
  ```

* 准备数据

  * 取值范围不同的数据，普遍采用的最佳实践是对每 个特征做标准化，即对于输入数据的每个特征(输入数据矩阵中的列)，减去特征平均值，再除 以标准差，这样得到的特征平均值为 0，标准差为 1。用 Numpy 可以很容易实现标准化

    ```python
    # 数据标准化
    mean = train_data.mean(axis=0) 
    train_data -= mean
    std = train_data.std(axis=0) 
    train_data /= std
    test_data -= mean 
    test_data /= std
    # 用于测试数据标准化的均值和标准差都是在训练数据上计算得到的。在工作流程中，不能使用在测试数据上计算得到的任何结果，即使是像数据标准化这么简单的事情也不行
    ```

* 构建网络

  * 由于样本数量很少，将使用一个非常小的网络，其中包含两个隐藏层，每层有 64 个单元。一般来说，训练数据越少，过拟合会越严重，而较小的网络可以降低过拟合

    ```python
    # 模型定义
    from keras import models
    from keras import layers
    def build_model():
      model = models.Sequential() 
      # 因为需要将同一个模型多次实例化， 所以用一个函数来构建模型
      model.add(layers.Dense(64, activation='relu',input_shape=(train_data.shape[1],)))
      model.add(layers.Dense(64, activation='relu'))
      model.add(layers.Dense(1))
      # 编译网络用的是 mse 损失函数，即均方误差,预测值与 目标值之差的平方。这是回归问题常用的损失函数
      # 在训练过程中还监控一个新指标:平均绝对误差。它是预测值与目标值之差的绝对值
      # 比如，如果这个问题的 MAE 等于 0.5，就表示你预测的房价与实际价 格平均相差 500 美元
      model.compile(optimizer='rmsprop', loss='mse', metrics=['mae']) 
      return model
    # 网络的最后一层只有一个单元，没有激活，是一个线性层。这是标量回归(标量回归是预 测单一连续值的回归)的典型设置。添加激活函数将会限制输出范围。例如，如果向最后一层 添加 sigmoid 激活函数，网络只能学会预测 0~1 范围内的值。这里最后一层是纯线性的，所以 网络可以学会预测任意范围内的值
    ```

* 利用 *K* 折验证来验证方法

  * 验证集的划分方式可能会造成验证分数上有很大的方差，这样就无法对模型进行可靠的评估

    * 在这种情况下，最佳做法是使用 *K* 折交叉验证。这种方法将可用数据划分为 *K* 个分区(*K* 通常取 4 或 5)，实例化 *K* 个相同的模型，将每个模型在 *K*-1 个分区上训练，并在剩 下的一个分区上进行评估。模型的验证分数等于 *K* 个验证分数的平均值

    ![image-20200709105506064](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709105506064.png)

    * K折验证

      ![image-20200709105702803](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709105702803.png)

      * 设置 num_epochs = 100，运行结果如下

        ```python
        >>> all_scores
        [2.588258957792037, 3.1289568449719116, 3.1856116051248984, 3.0763342615401386] 
        >>> np.mean(all_scores)
        2.9947904173572462
        ```

      * 每次运行模型得到的验证分数有很大差异，从 2.6 到 3.2 不等。平均分数(3.0)是比单一 分数更可靠的指标——这就是 *K* 折交叉验证的关键。在这个例子中，预测的房价与实际价格平 均相差 3000 美元，考虑到实际价格范围在 10 000~50 000 美元，这一差别还是很大的。
      * 让训练时间更长一点，达到 500 个轮次。为了记录模型在每轮的表现，我们需要修改 训练循环，以保存每轮的验证分数记录

    * 保存每折的验证结果

      ![image-20200709105845470](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709105845470.png)

      ```python
      # 计算所有轮次中的 K 折验证分数平均值
      average_mae_history = [
        np.mean([x[i] for x in all_mae_histories]) for i in range(num_epochs)]
      ```

      ```python
      # 绘制验证分数
      import matplotlib.pyplot as plt
      plt.plot(range(1, len(average_mae_history) + 1), average_mae_history) 
      plt.xlabel('Epochs')
      plt.ylabel('Validation MAE')
      plt.show()
      ```

      ![image-20200709110012702](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709110012702.png)

      * 因为纵轴的范围较大，且数据方差相对较大，所以难以看清这张图的规律。重新绘 制一张图。
        * 删除前 10 个数据点，因为它们的取值范围与曲线上的其他点不同
        * 将每个数据点替换为前面数据点的指数移动平均值，以得到光滑的曲线

      * 绘制验证分数(删除前 10 个数据点)

      ![image-20200709110139742](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709110139742.png)

      ![image-20200709110225823](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709110225823.png)

      * 验证 MAE 在 80 轮后不再显著降低，之后就开始过拟合
      * 完成模型调参之后(除了轮数，还可以调节隐藏层大小)，可以使用最佳参数在所有训练数据上训练最终的生产模型，然后观察模型在测试集上的性能

* 训练最终模型

  ```python
  model = build_model() 
  # 一个全新的编译好的模型 
  model.fit(train_data, train_targets,epochs=80, batch_size=16, verbose=0)
  # 在所有训练数据上训练模型
  test_mse_score, test_mae_score = model.evaluate(test_data, test_targets)
  # 最终结果如下
  >>> test_mae_score 2.5532484335057877
  # 预测的房价还是和实际价格相差约 2550 美元
  ```

* 小结
  * 回归问题使用的损失函数与分类问题不同。回归常用的损失函数是均方误差(MSE)
  * 同样，回归问题使用的评估指标也与分类问题不同。显而易见，精度的概念不适用于回归问题。常见的回归指标是平均绝对误差(MAE)
  * 如果输入数据的特征具有不同的取值范围，应该先进行预处理，对每个特征单独进行缩放
  * 如果可用的数据很少，使用 *K* 折验证可以可靠地评估模型
  * 如果可用的训练数据很少，最好使用隐藏层较少(通常只有一到两个)的小型网络，以避免严重的过拟合

* 本章小结
  * 在将原始数据输入神经网络之前，通常需要对其进行预处理
  * 如果数据特征具有不同的取值范围，那么需要进行预处理，将每个特征单独缩放
  * 随着训练的进行，神经网络最终会过拟合，并在前所未见的数据上得到更差的结果
  * 如果训练数据不是很多，应该使用只有一两个隐藏层的小型网络，以避免严重的过拟合
  * 如果数据被分为多个类别，那么中间层过小可能会导致信息瓶颈
  * 回归问题使用的损失函数和评估指标都与分类问题不同
  * 如果要处理的数据很少，*K* 折验证有助于可靠地评估模型

# 机器学习基础

## 机器学习的四个分支

* 二分类问题、多分类问题和标量回归问题。这三者都是监督学习(supervised learning)的例子，其目标是学习训练输入与训 练目标之间的关系
* 监督学习
  * 是目前最常见的机器学习类型。给定一组样本(通常由人工标注)，它可以学会将输入数据映射到已知目标[也叫标注(annotation)]
  * 深度学习应用：光学字符识别、语音识别、 图像分类和语言翻译
  * 监督学习主要包括分类和回归，但还有更多的奇特变体
    * 序列生成(sequence generation)。给定一张图像，预测描述图像的文字
    * 语法树预测(syntax tree prediction)。给定一个句子，预测其分解生成的语法树
    * 目标检测(object detection)。给定一张图像，在图中特定目标的周围画一个边界框
    * 图像分割(image segmentation)。给定一张图像，在特定物体上画一个像素级的掩模(mask)。
* 无监督学习
  * 无监督学习是指在没有目标的情况下寻找输入数据的有趣变换，其目的在于数据可视化、 数据压缩、数据去噪或更好地理解数据中的相关性。无监督学习是数据分析的必备技能，在解决监督学习问题之前，为了更好地了解数据集，它通常是一个必要步骤。降维(dimensionality reduction)和聚类(clustering)都是众所周知的无监督学习方法
* 自监督学习
  * 是没有 人工标注的标签的监督学习，可以将它看作没有人类参与的监督学习。标签仍然存在(因为 总要有什么东西来监督学习过程)，但它们是从输入数据中生成的，通常是使用启发式算法生成的
  * 举个例子，自编码器(autoencoder)是有名的自监督学习的例子，其生成的目标就是未经修改的输入。同样，给定视频中过去的帧来预测下一帧，或者给定文本中前面的词来预测下一个词， 都是自监督学习的例子
* 强化学习
  * 在强化学习中，智能体(agent)接收有关其环境的信息，并学会选择使某种奖励最大化的行动
  * 目前，强化学习主要集中在研究领域，除游戏外还没有取得实践上的重大成功。但是，我们期待强化学习未来能够实现越来越多的实际应用:自动驾驶汽车、机器人、资源管理、教育等
* 分类和回归术语表
  * 样本(sample)或输入(input)：进入模型的数据点
  * 预测(prediction)或输出(output)：从模型出来的结果
  * 目标(target)：真实值。对于外部数据源，理想情况下，模型应该能够预测出目标
  * 预测误差(prediction error)或损失值(loss value)：模型预测与目标之间的距离
  * 类别(class)：分类问题中供选择的一组标签。例如，对猫狗图像进行分类时，“狗”和“猫”就是两个类别
  * 标签(label)：分类问题中类别标注的具体例子。比如，如果1234号图像被标注为包含类别“狗”，那么“狗”就是 1234 号图像的标签
  * 真值(ground-truth)或标注(annotation)：数据集的所有目标，通常由人工收集
  * 二分类(binary classification)：一种分类任务，每个输入样本都应被划分到两个互斥的类别中
  * 多分类(multiclass classification)：一种分类任务，每个输入样本都应被划分到两个以上的类别中，比如手写数字分类。
  * 多标签分类(multilabel classification)：一种分类任务，每个输入样本都可以分配多个标签
  * 标量回归(scalar regression)：目标是连续标量值的任务。预测房价就是一个很好的例子，不同的目标价格形成一个连续的空间
  * 向量回归(vector regression)：目标是一组连续值(比如一个连续向量)的任务。如果对多个值(比如图像边界框的坐标)进行回归，那就是向量回归
  * 小批量(mini-batch)或批量(batch)：模型同时处理的一小部分样本(样本数通常 为 8~128)。样本数通常取 2 的幂，这样便于 GPU 上的内存分配。训练时，小批量用来为模型权重计算一次梯度下降更新

## 评估机器学习模型

* 过拟合，随着训练的进行，模型在训练数据上的性能始终在提高，但在前所未见的 数据上的性能则不再变化或者开始下降

* 机器学习的目的是得到可以泛化(generalize)的模型，即在前所未见的数据上表现很好的模型，而过拟合则是核心难点

* 训练集、验证集和测试集

  * 为什么不是两个集合:一个训练集和一个测试集?
    * 原因在于开发模型时总是需要调节模型配置，比如选择层数或每层大小[这叫作模型的超参数(hyperparameter)，以便与模型参数(即权重)区分开]。这个调节过程需要使用模型在验 证数据上的性能作为反馈信号这个调节过程本质上就是一种学习：在某个参数空间中寻找良好的模型配置。因此，如果基于模型在验证集上的性能来调节模型配置，会很快导致模型在验证集上过拟合，即使你并没有在验证集上直接训练模型也会如此
    * 造成这一现象的关键在于信息泄露
      * 每次基于模型在验证集上的性能来调节模型超参数，都会有一些关于验证数据的信息泄露到模型中。如果对每个参数只调节一次， 那么泄露的信息很少，验证集仍然可以可靠地评估模型。但如果多次重复这一过程(运行一次实验，在验证集上评估，然后据此修改模型)，那么将会有越来越多的关于验证集的信息泄露到模型中
    * 最后，得到的模型在验证集上的性能非常好(人为造成的)，因为这正是你优化的目的。 你关心的是模型在全新数据上的性能，而不是在验证数据上的性能，因此你需要使用一个完全 不同的、前所未见的数据集来评估模型，它就是测试集。你的模型一定不能读取与测试集有关 的任何信息，既使间接读取也不行。如果基于测试集性能来调节模型，那么对泛化能力的衡量 是不准确的

* 将数据划分为训练集、验证集和测试集可能看起来很简单，但如果可用数据很少，还有几 种高级方法可以派上用场。先来介绍三种经典的评估方法:简单的留出验证、*K* 折验证， 以及带有打乱数据的重复 *K* 折验证

  * 简单的留出验证

    * 留出一定比例的数据作为测试集。在剩余的数据上训练模型，然后在测试集上评估模型

    * 为了防止信息泄露，不能基于测试集来调节模型，所以还应该保留一个验证集

      ![image-20200709112818619](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709112818619.png)

      ![image-20200709112853149](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709112853149.png)

      * 这是最简单的评估方法，但有一个缺点：如果可用的数据很少，那么可能验证集和测试集 包含的样本就太少，从而无法在统计学上代表数据。这个问题很容易发现：如果在划分数据前 进行不同的随机打乱，最终得到的模型性能差别很大，那么就存在这个问题。接下来会介绍 *K* 折 验证与重复的 *K* 折验证，它们是解决这一问题的两种方法。

  * K折验证

    * 将数据划分为大小相同的 *K* 个分区。对于每个分区 i，在剩 余的 *K*-1 个分区上训练模型，然后在分区 i 上评估模型。最终分数等于 *K* 个分数的平均值

      * 对 于不同的训练集 - 测试集划分，如果模型性能的变化很大，那么这种方法很有用。与留出验证 一样，这种方法也需要独立的验证集进行模型校正

        ![image-20200709113050814](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709113050814.png)

        ![image-20200709113120658](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709113120658.png)

  * 带有打乱数据的重复 *K* 折验证
    * 如果可用的数据相对较少，而你又需要尽可能精确地评估模型
    * 具体做法是多次使用 *K* 折验证，在每次将数据划分为 *K* 个分区之前都先将数据打乱。 最终分数是每次 *K* 折验证分数的平均值。注意，这种方法一共要训练和评估 *P*×*K* 个模型(*P* 是重复次数)，计算代价很大。

* 评估模型的注意事项
  * 数据代表性，希望训练集和测试集都能够代表当前数据。在将数据划分为训练集和测试集 之前，通常应该随机打乱数据
  * 时间箭头，如果想要根据过去预测未来，那么在划分数据前你不应该随机打乱数据，因为这么做会造成时间泄露：你的模型将在未来数据上得到有效训练。在这种情况下，你应该始终确保测试集 中所有数据的时间都晚于训练集数据
  * 数据冗余，如果数据中的某些数据点出现了两次(这在现实中的数据里十分常见)，那么打乱数据并划分成训练集和验证集会导致训练集和验证集之间的数据冗余。从效果上来看，你是在部分训练数据上评估模型，这是极其糟糕的!一 定要确保训练集和验证集之间没有交集。

## 数据预处理、特征工程和特征学习

* 神经网络的数据预处理

  * ==数据预处理的目的是使原始数据更适于用神经网络处理，包括向量化、标准化、处理缺失值和特征提取==

  * 向量化

    * 神经网络的所有输入和目标都必须是浮点数张量(在特定情况下可以是整数张量)。无论 处理什么数据(声音、图像还是文本)，都必须首先将其转换为张量，这一步叫作数据向量化
    * 例如开始时文本都表示为整数列表(代 表单词序列)，然后用 one-hot 编码将其转换为 float32 格式的张量

  * 值标准化

    * 为了让网络的学习变得更容易，输入数据应该具有以下特征

      * 取值较小：大部分值都应该在 0~1 范围内
      * 同质性(homogenous)：所有特征的取值都应该在大致相同的范围内

    * 更严格的标准化方法也很常见，而且很有用，虽然不一定总是必需的

      * 将每个特征分别标准化，使其平均值为 0

      * 将每个特征分别标准化，使其标准差为 1

        ```python
        # 这对于 Numpy 数组很容易实现
        # 假设 x 是一个形状为 (samples, features) 的二维矩阵
        x -= x.mean(axis=0)
        x /= x.std(axis=0)
        ```

  * 处理缺失值

    * 一般来说，对于神经网络，将缺失值设置为 0 是安全的，只要 0 不是一个有意义的值。网 络能够从数据中学到 0 意味着缺失数据，并且会忽略这个值
    * 注意，如果测试数据中可能有缺失值，而网络是在没有缺失值的数据上训练的，那么网络 不可能学会忽略缺失值。在这种情况下，你应该人为生成一些有缺失项的训练样本:多次复制 一些训练样本，然后删除测试数据中可能缺失的某些特征

* 特征工程

  * 是指将数据输入模型之前，利用你自己关于数据和机器学习算法(这里指神经网络)的知识对数据进行硬编码的变换(不是模型学到的)，以改善模型的效果。多数情况下，一个机器学习模型无法从完全任意的数据中进行学习。呈现给模型的数据应该便于模型进行学习
  * 特征工程的本质：用更简单的方式表述问题，从而使问题变得更容易。它通常需要深入理解问题
  * 对于现代深度学习，大部分特征工程都是不需要的，因为神经网络能够从原始 数据中自动提取有用的特征，但是
    * 良好的特征仍然可以让你用更少的资源更优雅地解决问题
      * 例如，使用卷积神经网络来读取钟面上的时间是非常可笑的
    * 良好的特征可以让你用更少的数据解决问题。深度学习模型自主学习特征的能力依赖于 大量的训练数据。如果只有很少的样本，那么特征的信息价值就变得非常重要

## 过拟合和欠拟合

* 模型在留出验证数据上的性能总是在几轮后达到最高点，然后开始下降。也就是说，模型很快就在训练数据上开始过 拟合。过拟合存在于所有机器学习问题中。学会如何处理过拟合对掌握机器学习至关重要

* ==机器学习的根本问题是优化和泛化之间的对立==

  * 优化(optimization)是指调节模型以在训练数据上得到最佳性能(即机器学习中的学习)
  * 泛化(generalization)是指训练好的模型在 前所未见的数据上的性能好坏
  * 机器学习的目的当然是得到良好的泛化，但无法控制泛化，只能基于训练数据调节模型

* 训练开始时，优化和泛化是相关的

  * 训练数据上的损失越小，测试数据上的损失也越小。这时的模型是欠拟合(underfit)的，即仍有改进的空间，网络还没有对训练数据中所有相关模 式建模
  * 但在训练数据上迭代一定次数之后，泛化不再提高，验证指标先是不变，然后开始变差， 即模型开始过拟合。这时模型开始学习仅和训练数据有关的模式，但这种模式对新数据来说是 错误的或无关紧要的
  * 为了防止模型从训练数据中学到错误或无关紧要的模式，最优解决方法是获取更多的训练数据。模型的训练数据越多，泛化能力自然也越好。如果无法获取更多数据，次优解决方法是调节模型允许存储的信息量，或对模型允许存储的信息加以约束。如果一个网络只能记住几个模式，那么优化过程会迫使模型集中学习最重要的模式，这样更可能得到良好的泛化。
  * 这种降低过拟合的方法叫作正则化

* 减小网络大小

  * 防止过拟合的最简单的方法就是减小模型大小，即减少模型中可学习参数的个数(这由层数和每层的单元个数决定)。在深度学习中，模型中可学习参数的个数通常被称为模型的容量

  * 深度学习模型通常都很擅长拟合训练数据，但真正的挑战在于泛化，而不是拟合

  * 如果网络的记忆资源有限，则无法轻松学会这种映射。因此，为了让损失最小化， 网络必须学会对目标具有很强预测能力的压缩表示，这也正是我们感兴趣的数据表示

  * 使用的模型应该具有足够多的参数，以防欠拟合，即模型应避免记忆资源不足。在容 量过大与容量不足之间要找到一个折中

  * 必须评估一系列不同的网络架构(当然是在验证集上评估，而不是在测试集上)，以便为数据找到最佳的模型大小。 要找到合适的模型大小，一般的工作流程是开始时选择相对较少的层和参数，然后逐渐增加层的大小或增加新层，直到这种增加对验证损失的影响变得很小

    ```python
    # 电影评论分类的网络上试一下
    # 原始模型
    from keras import models
    from keras import layers
    model = models.Sequential()
    model.add(layers.Dense(16, activation='relu', input_shape=(10000,))) 
    model.add(layers.Dense(16, activation='relu')) 
    model.add(layers.Dense(1, activation='sigmoid'))
    # 容量更小的模型
    model = models.Sequential()
    model.add(layers.Dense(4, activation='relu', input_shape=(10000,))) 
    model.add(layers.Dense(4, activation='relu')) 
    model.add(layers.Dense(1, activation='sigmoid'))
    ```

    ![image-20200709115208386](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709115208386.png)

    * 更小的验证损失对应更好的模型

    ```python 
    # 容量更大的模型
    model = models.Sequential()
    model.add(layers.Dense(512, activation='relu', input_shape=(10000,))) 
    model.add(layers.Dense(512, activation='relu')) 
    model.add(layers.Dense(1, activation='sigmoid'))
    ```

    ![image-20200709115340893](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709115340893.png)

    * 更大的网络只过了一轮就开始过拟合，过拟合也更严重。其验证损失的波动也更大

    * 更大网络的训练损失很快就接近于零。 网络的容量越大，它拟合训练数据(即得到很小的训练损失)的速度就越快，但也更容易过拟合(导致训练损失和验证损失有很大差异)

      ![image-20200709115443685](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709115443685.png)

* 添加权重正则化

  * 奥卡姆剃刀(Occam’s razor)原理：如果一件事情有两种解释，那么最可能正确的解释就是最简单的那个，即假设更少的那个

    * 给定一些训练数据和一种网络架构，很多组权重值(即很多模型)都可以解释这些数据。简单模型比复杂模型更不容易过拟合

    * 这里的简单模型(simple model)是指参数值分布的熵更小的模型(或参数更少的模型

    * 因此，一种常见的降低过拟合的方法就是强制让模型权重只能取较小的值，从而限制模型的复杂度，这使得权重值的分布更加规则(regular)。这种方法叫作权重正则化。其实现方法是向网络损失函数中添加与较大权重值相关的成本

    * 这个成本有两种形式

      * L1 正则化(L1 regularization):添加的成本与权重系数的绝对值[权重的 L1 范数(norm)] 成正比
      * L2 正则化(L2 regularization):添加的成本与权重系数的平方(权重的 L2 范数)成正比。 神经网络的 L2 正则化也叫权重衰减(weight decay)。不要被不同的名称搞混，权重衰减 与 L2 正则化在数学上是完全相同的

    * 在 Keras 中，添加权重正则化的方法是向层传递权重正则化项实例(weight regularizer instance)作为关键字参数

      ```python
      # 向模型添加 L2 权重正则化
      from keras import regularizers
      
      model = models.Sequential()
      model.add(layers.Dense(
        16, 
        kernel_regularizer=regularizers.l2(0.001),
        activation='relu', 
        input_shape=(10000,))) 
      model.add(layers.Dense(
        16, 
        kernel_regularizer=regularizers.l2(0.001),
        activation='relu'))
      model.add(layers.Dense(1, activation='sigmoid'))
      # l2(0.001)的意思是该层权重矩阵的每个系数都会使网络总损失增加0.001 * weight_ coefficient_value。注意，由于这个惩罚项只在训练时添加，所以这个网络的训练损失会 比测试损失大很多
      ```

      ![image-20200709131641748](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709131641748.png)

      * 即使两个模型的参数个数相同，具有 L2 正则化的模型(圆点)比参考模型(十字)更不容易过拟合

      ```python
      # 还可以用 Keras 中以下这些权重正则化项来代替 L2 正则化
      # Keras 中不同的权重正则化项
      from keras import regularizers
      # L1 正则化
      regularizers.l1(0.001) 
      # 同时做 L1 和 L2 正则化
      regularizers.l1_l2(l1=0.001, l2=0.001)
      ```

* 添加 dropout 正则化

  * dropout 是神经网络最有效也最常用的正则化方法之一

  * 对某一层使用 dropout，就是在训练过程中随机将该层的一些输出特征舍 弃(设置为 0)

    * 假设在训练过程中，某一层对给定输入样本的返回值应该是向量 [0.2, 0.5, 1.3, 0.8, 1.1]。使用 dropout 后，这个向量会有几个随机的元素变成 0，比如 [0, 0.5, 1.3, 0, 1.1]

  * dropout 比率(dropout rate)是被设为 0 的特征所占的比例，通常在 0.2~0.5 范围内。测试时没有单元被舍弃，而该层的输出值需要按 dropout 比率缩小，因为这时比训练时 有更多的单元被激活，需要加以平衡

    ```python
    # 假设有一个包含某层输出的Numpy矩阵layer_output，其形状为(batch_size, features)。训练时，随机将矩阵中一部分值设为 0
    # 训练时，舍弃 50%的输出单元
    layer_output *= np.random.randint(0, high=2, size=layer_output.shape)
    # 测试时，我们将输出按 dropout 比率缩小。这里我们乘以 0.5(因为前面舍弃了一半的单元)
    # 测试时
    layer_output *= 0.5 
    # 注意，为了实现这一过程，还可以让两个运算都在训练时进行，而测试时输出保持不变。 这通常也是实践中的实现方式
    layer_output *= np.random.randint(0, high=2, size=layer_output.shape)      # 训练时
    layer_output /= 0.5      # 注意，是成比例放大而不是成比例缩小
    ```

    ![image-20200709132243030](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709132243030.png)

  * 在每个样本中随机删除不同的部分神经元，可以阻止它们的阴谋，因此可以降低过拟合。其核心思想是在层的输出值中引入噪声， 打破不显著的偶然模式(Hinton 称之为阴谋)。如果没有噪声的话，网络将会记住这些偶然模式。
  * 在 Keras 中，你可以通过 Dropout 层向网络中引入 dropout，dropout 将被应用于前面一层 的输出
    
  * model.add(layers.Dropout(0.5))
  
  ```python
  # 向 IMDB 网络中添加两个 Dropout 层，来看一下它们降低过拟合的效果
  model = models.Sequential()
  model.add(layers.Dense(16, activation='relu', input_shape=(10000,))) 
  model.add(layers.Dropout(0.5))
  model.add(layers.Dense(16, activation='relu')) 
  model.add(layers.Dropout(0.5))
  model.add(layers.Dense(1, activation='sigmoid'))
```
  
![image-20200709133054787](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709133054787.png)
  
  * 防止神经网络过拟合的常用方法包括
    * 获取更多的训练数据
    * 减小网络容量
    * 添加权重正则化
    * 添加 dropout

## 机器学习的通用工作流程

* 定义问题，收集数据集

  * 首先，必须定义所面对的问题
    * 你的输入数据是什么?你要预测什么?
      * 只有拥有可用的训练数据，才能学习预测某件事情。数据可用性通常是这一阶段的限制因素
    * 面对的是什么类型的问题?
      * 是二分类问题、多分类问题、标量回归问题、向量回归问题， 还是多分类、多标签问题?或者是其他问题，比如聚类、生成或强化学习?确定问题类型有助于你选择模型架构、损失函数等
  * 只有明确了输入、输出以及所使用的数据，才能进入下一阶段。注意在这一阶段所做的假设
    * 假设输出是可以根据输入进行预测的
    * 假设可用数据包含足够多的信息，足以学习输入和输出之间的关系
  * 机器学习只能用来记忆训练数据中存在的模式
    * 只能识别出曾经见过的东西。 在过去的数据上训练机器学习来预测未来，这里存在一个假设，就是未来的规律与过去相同。 但事实往往并非如此

* 选择衡量成功的指标

  * 精度?准确 率(precision)和召回率(recall)?客户保留率?
  * 衡量成功的指标将指引选择损失函数，即 模型要优化什么。它应该直接与的目标(如业务成功)保持一致
  * 对于平衡分类问题(每个类别的可能性相同)，精度和接收者操作特征曲线下面积是常用的指标
  * 对于类别不平衡的问题，可以使用准确率和召回率
  * 对于排序问题或多标签分类，可以使用平均准确率均值
  * 自定义衡量成功的指标也很常见

* 确定评估方法

  * 一旦明确了目标，必须确定如何衡量当前的进展，三种常见的评估方法
    *   留出验证集。数据量很大时可以采用这种方法。
    *   *K* 折交叉验证。如果留出验证的样本量太少，无法保证可靠性，那么应该选择这种方法。
    *   重复的 *K* 折验证。如果可用的数据很少，同时模型评估又需要非常准确，那么应该使用这种方法。
  * 只需选择三者之一。大多数情况下，第一种方法足以满足要求

* 准备数据

  * 一旦知道了要训练什么、要优化什么以及评估方法，那么就几乎已经准备好训练模型了
  *  但首先应该将数据格式化，使其可以输入到机器学习模型中
    * 如前所述，应该将数据格式化为张量
    * 这些张量的取值通常应该缩放为较小的值，比如在 [-1, 1] 区间或 [0, 1] 区间。 
    * 如果不同的特征具有不同的取值范围(异质数据)，那么应该做数据标准化。 
    * 可能需要做特征工程，尤其是对于小数据问题
  * 准备好输入数据和目标数据的张量后，你就可以开始训练模型了

* 开发比基准更好的模型

  * 这一阶段的目标是获得统计功效，即开发一个小型模型，它能够打败纯随机的基准

    * 在 MNIST 数字分类的例子中，任何精度大于 0.1 的模型都可以说 具有统计功效;在 IMDB 的例子中，任何精度大于 0.5 的模型都可以说具有统计功效

  * 不一定总是能获得统计功效。如果你尝试了多种合理架构之后仍然无法打败随机基准， 那么原因可能是问题的答案并不在输入数据中。要记住你所做的两个假设

    * 假设输出是可以根据输入进行预测的
    * 假设可用的数据包含足够多的信息，足以学习输入和输出之间的关系。

  * 这些假设很可能是错误的，这样的话需要从头重新开始

  * 如果一切顺利，还需要选择三个关键参数来构建第一个工作模型

    * 最后一层的激活。它对网络输出进行有效的限制。例如，IMDB 分类的例子在最后一层 使用了 sigmoid，回归的例子在最后一层没有使用激活，等等。
    * 损失函数。它应该匹配你要解决的问题的类型。例如，IMDB 的例子使用binary_ crossentropy、回归的例子使用 mse，等等。
    *  优化配置。你要使用哪种优化器?学习率是多少?大多数情况下，使用 rmsprop 及其 默认的学习率是稳妥的

  * 关于损失函数的选择，需要注意，直接优化衡量问题成功的指标不一定总是可行的。有时难以将指标转化为损失函数，要知道，损失函数需要在只有小批量数据时即可计算(理想情况 下，只有一个数据点时，损失函数应该也是可计算的)，而且还必须是可微的(否则无法用反向 传播来训练网络)

    ![image-20200709134830065](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200709134830065.png)

  * 扩大模型规模:开发过拟合的模型

    * 一旦得到了具有统计功效的模型，问题就变成了:模型是否足够强大?它是否具有足够多 的层和参数来对问题进行建模?
      * 例如，只有单个隐藏层且只有两个单元的网络，在 MNIST 问题 上具有统计功效，但并不足以很好地解决问题
    * 请记住，机器学习中无处不在的对立是优化和泛化的对立，理想的模型是刚好在欠拟合和过拟合的界线上，在容量不足和容量过大的界线上。 为了找到这条界线，你必须穿过它
    * 要搞清楚需要多大的模型，就必须开发一个过拟合的模型，这很简单。
      1. 添加更多的层
      2. 让每一层变得更大
      3. 训练更多的轮次
    * 要始终监控训练损失和验证损失，以及你所关心的指标的训练值和验证值。如果你发现模型在验证数据上的性能开始下降，那么就出现了过拟合

* 模型正则化与调节超参数
  * 尽可能地接近理想模型，既不过拟合也不欠拟合
  * 这一步是最费时间的：你将不断地调节模型、训练、在验证数据上评估(这里不是测试数据)、 再次调节模型，然后重复这一过程，直到模型达到最佳性能。你应该尝试以下几项
    * 添加 dropout
    * 尝试不同的架构:增加或减少层数。
    * 添加 L1 和 / 或 L2 正则化。
    * 尝试不同的超参数(比如每层的单元个数或优化器的学习率)，以找到最佳配置。
    * (可选)反复做特征工程:添加新特征或删除没有信息量的特征
* 一旦开发出令人满意的模型配置，就可以在所有可用数据(训练数据 + 验证数据)上训 练最终的生产模型，然后在测试集上最后评估一次。如果测试集上的性能比验证集上差很多， 那么这可能意味着你的验证流程不可靠，或者你在调节模型参数时在验证数据上出现了过拟合。 在这种情况下，你可能需要换用更加可靠的评估方法，比如重复的 *K* 折验证

* 小结
  * 定义问题与要训练的数据。收集这些数据，有需要的话用标签来标注数据。
  * 选择衡量问题成功的指标。你要在验证数据上监控哪些指标?
  * 确定评估方法:留出验证? *K* 折验证?你应该将哪一部分数据用于验证?
  * 开发第一个比基准更好的模型，即一个具有统计功效的模型。
  * 开发过拟合的模型。
  * 基于模型在验证数据上的性能来进行模型正则化与调节超参数。许多机器学习研究往往只关注这一步，但你一定要牢记整个工作流程