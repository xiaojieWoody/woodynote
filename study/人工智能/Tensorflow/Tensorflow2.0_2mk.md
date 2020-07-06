# Tensorflow keras实战

## keras是什么

* 基于python的高级神经网络API
* 以Tensorflow、CNTK或者Theano为后端运行，，keras必须有后端才可以运行
  * 后端可以切换，现在多用tensorflow
* 极方便于快速实验，帮助用户以最少的时间验证自己的想法

## Tensorflow-keras是什么

* Tensorflow对keras API规范的实现
* 相对于以tensorflow为后端的keras，Tensorflow-keras与Tensorflow结合更加紧密
* 实现在tf.keras空间下

## Tf-keras和keras联系

* 基于同一套API
  * keras程序可以通过改导入方式轻松转为tf.keras程序
  * 反之可能不成立，因为tf.keras有其他特性
* 相同的JSON和HDF5模型序列化格式和语义

## Tf-keras和keras区别

* Tf.keras全面支持eager mode
  * 只是用keras.Sequential和keras.Model时没影响
  * 自定义Model内部运算逻辑的时候会有影响
    * Tf底层API可以使用keras的model.fit等抽象
    * 适用于研究人员
  * Tf.keras支持基于tf.data的模型训练
  * Tf.keras支持TPU训练
  * Tf.keras支持tf.distribution中的分布式策略
  * 其他特性
    * Tf.keras可以与Tensorflow中的estimator集成
    * Tf.keras可以保存为SavedModel

## 如何选择

* 如果想用tf.keras的任何一个特性，那么选tf.keras
* 如果后端互换性很重要，那么选keras
* 如果都不重要，那随便

## 分类问题与回归问题

* 分类问题预测的是类别，模型的输出是概率分布
  * 三分类问题输出例子：[0.2,0.7,0.1]
* 回归问题预测的是值，模型的输出是一个实数值

## 目标函数

* 为什么需要目标函数？
  * 参数是逐步调整的
  * 目标函数可以帮助衡量模型的好坏
    * Model A：[0.1, 0.4, 0.5]
    * Model B：[0.1, 0.2, 0.7]

* 分类问题

  * 需要衡量目标类别与当前预测的差距

    * 三分类问题输出例子：[0.2, 0.7, 0.1]
    * 三分类真实类别：2 -> one_hot -> [0, 0, 1]

  * One-hot编码，把正整数变为向量表达

    * 生成一个长度不小于正整数的向量，只有正整数的位置处于1，其余位置都为0

  * 平方差损失

    ![image-20200702234745901](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702234745901.png)

  * 交叉熵损失

    ![image-20200702234801708](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702234801708.png)

  * 平方差损失举例

    * 预测值：[0.2, 0.7, 0.1]
    * 真实值：[0, 0, 1]
    * 损失函数值：[(0 - 0)^2 + (0.7 - 0)^2 + (0.1 - 1)^2] * 0.5 = 0.65

* 回归问题

  * 预测值与真实值的差距
  * 平方差损失
  * 绝对值损失

* 模型的训练就是调整参数，使得目标函数逐渐变小的过程

## 实战

* Keras搭建分类模型

  ![image-20200703135835116](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703135835116.png)

  ![image-20200703145651594](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703145651594.png)

  ```python
  import matplotlib as mpl
  import matplotlib.pyplot as plt
  %matplotlib inline
  import numpy as np
  import sklearn
  import pandas as pd
  import os
  import sys
  import time
  import tensorflow as tf
  
  from tensorflow import keras
  
  print(tf.__version__)
  print(sys.version_info)
  for module in mpl, np, pd, sklearn, tf, keras:
      print(module.__name__, module.__version__)
  ```

  ![image-20200703145551512](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703145551512.png)

  ```python
  fashion_mnist = keras.datasets.fashion_mnist
  (x_train_all, y_train_all),(x_test, y_test) = fashion_mnist.load_data()
  x_valid, x_train = x_train_all[:5000], x_train_all[5000:]
  y_valid, y_train = y_train_all[:5000], y_train_all[5000:]
  
  print(x_valid.shape, y_valid.shape)
  print(x_train.shape, y_train.shape)
  print(x_test.shape, y_test.shape)
  ```

  ![image-20200703145446872](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703145446872.png)

  ```python
  def show_single_image(img_arr):
      plt.imshow(img_arr, cmap="binary")
      plt.show()
  show_single_image(x_train[0]) 
  ```

  ![image-20200703150033666](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703150033666.png)

  ```python
  def show_imgs(n_rows, n_cols, x_data, y_data, class_names):
      assert len(x_data) == len(y_data)
      assert n_rows * n_cols < len(x_data)
      plt.figure(figsize = (n_cols * 1.4, n_rows * 1.6))
      for row in range(n_rows):
          for col in range(n_cols):
              index = n_cols * row + col
              plt.subplot(n_rows, ncols, index + 1)
              plt.imshow(x_data[index], cmap="binary",
                        interpolation = 'nearest')
              plt.axis('off')
              plt.title(class_names[y_data[index]])
      plt.show()
      
  class_names = ['T-shirt', 'Trouser', 'Pullover', 'Dress',
                'Coat', 'Sandal', 'Shirt', 'Sneaker',
                'Bag', 'Ankle boot']
  show_imgs(3, 5, x_train, y_train, class_names)
  ```

  ```python
  # 构建模型
  # tf.keras.models.Sequential()
  """
  model = keras.models.Sequential()
  model.add(keras.layers.Flatten(input_shape=[28, 28]))
  model.add(keras.layers.Dense(300, activation="relu"))
  model.add(keras.layers.Dense(100, activation="relu"))
  model.add(keras.layers.Dense(10, activation="softmax"))
  """
  model = keras.models.Sequential([
      keras.layers.Flatten(input_shape=[28, 28]),
      keras.layers.Dense(300, activation="relu"),
      keras.layers.Dense(100, activation="relu"),
      keras.layers.Dense(10, activation="softmax")
  ])
  
  
  # relu: y = max(0, x)
  # softmax: 将向量变成概率分布. x = [x1, x2, x3],
  #          y = [e^x1/sum, e^x2/sum, e^x3/sum], sum = e^x1 + e^x2 + e^x3
  # reason for sparse: y -> index. y->one_hot->[]
  model.compile(loss="sparse_categorical_crossentropy",
               optimizer = "sgd",
               metrics = ["accuracy"])
  ```

  ![image-20200703151248602](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703151248602.png)

  ![image-20200703151400447](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703151400447.png)

  ```python
  history = model.fit(x_train, y_train, epochs=10,
                     validation_data=(x_valid, y_valid))
  ```

  ![image-20200703151942197](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703151942197.png)

  ![image-20200703152011038](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703152011038.png)

  ![image-20200703152337394](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703152337394.png)

  ```python
  def plot_learning_curves(history):
      pd.DataFrame(history.history).plot(figsize=(8, 5))
      plt.grid(True)
      plt.gca().set_ylim(0, 1)
      plt.show()
      
  plot_learning_curves(history)   
  ```

  ![image-20200703153131401](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703153131401.png)

  * 数据归一化

    ```python
    import matplotlib as mpl
    import matplotlib.pyplot as plt
    %matplotlib inline
    import numpy as np
    import sklearn
    import pandas as pd
    import os
    import sys
    import time
    import tensorflow as tf
    
    from tensorflow import keras
    
    print(tf.__version__)
    print(sys.version_info)
    for module in mpl, np, pd, sklearn, tf, keras:
        print(module.__name__, module.__version__)
    ```

    ```python
    fashion_mnist = keras.datasets.fashion_mnist
    (x_train_all, y_train_all),(x_test, y_test) = fashion_mnist.load_data()
    x_valid, x_train = x_train_all[:5000], x_train_all[5000:]
    y_valid, y_train = y_train_all[:5000], y_train_all[5000:]
    
    print(x_valid.shape, y_valid.shape)
    print(x_train.shape, y_train.shape)
    print(x_test.shape, y_test.shape)
    ```

    ```python
    print(np.max(x_train), np.min(x_train))
    ```

    ```python
    # x = (x - u) / std
    from sklearn.preprocessing import StandardScaler
    scaler = StandardScaler()
    # x_train: [None, 28, 28] -> [None, 784]
    x_train_scaled = scaler.fit_transform(
        x_train.astype(np.float32).reshape(-1, 1)).reshape(-1, 28, 28)
    x_valid_scaled = scaler.transform(
        x_valid.astype(np.float32).reshape(-1, 1)).reshape(-1, 28, 28)
    x_test_scaled=scaler.transform(
        x_test.astype(np.float32).reshape(-1, 1)).reshape(-1, 28, 28)    
    ```

    ```python
    print(np.max(x_train_scaled), np.min(x_train_scaled))
    # 结果 2.0231433 -0.8105136
    ```

    ```python
    # tf.keras.models.Sequential()
    """
    model = keras.models.Sequential()
    model.add(keras.layers.Flatten(input_shape=[28, 28]))
    model.add(keras.layers.Dense(300, activation="relu"))
    model.add(keras.layers.Dense(100, activation="relu"))
    model.add(keras.layers.Dense(10, activation="softmax"))
    """
    model = keras.models.Sequential([
        keras.layers.Flatten(input_shape=[28, 28]),
        keras.layers.Dense(300, activation="relu"),
        keras.layers.Dense(100, activation="relu"),
        keras.layers.Dense(10, activation="softmax")
    ])
    
    
    # relu: y = max(0, x)
    # softmax: 将向量变成概率分布. x = [x1, x2, x3],
    #          y = [e^x1/sum, e^x2/sum, e^x3/sum], sum = e^x1 + e^x2 + e^x3
    # reason for sparse: y -> index. y->one_hot->[]
    model.compile(loss="sparse_categorical_crossentropy",
                 optimizer = "sgd",
                 metrics = ["accuracy"])
    ```

    ```python
    history = model.fit(x_train_scaled, y_train, epochs=10,
                       validation_data=(x_valid_scaled, y_valid))
    ```

    ![image-20200703160439554](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703160439554.png)

* Keras回调函数

# 18

* Keras搭建回归模型

# Tensorflow基础API使用

# Tensorflow dataset使用

# Tensorflow Estimator使用与tf1.0

# 卷积神经网络

# 循环神经网络

# Tensorflow分布式

# Tensorflow模型保存与部署

# 机器翻译