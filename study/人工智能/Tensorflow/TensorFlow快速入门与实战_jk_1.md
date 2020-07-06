##==14、15==

# TensorFlow初印象

![image-20200703163602766](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703163602766.png)

## 应用场景

* 围棋、 天文学、智慧（无人）银行、行人检测、人脸检测、行为识别、身份证自动输入与人脸图像比较、OCR+自动化审核

## 落地应用

* Google神经网络机器翻译（GNMT）
* 减少数据中心能源消耗
* 监控奶牛状况
* 疾病辅助诊断

# TensorFlow初接触 

![image-20200703170512248](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703170512248.png)

![image-20200703170553484](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703170553484.png)

```python
root@woody-VirtualBox:~# cd environment/
root@woody-VirtualBox:~/environment# ls
tf_py3
# 激活python虚拟环境
root@woody-VirtualBox:~/environment# source tf_py3/bin/activate
(tf_py3) root@woody-VirtualBox:~/environment#
```

```python
# python 2.7
import tensorflow as tf
hello = tf.constant("Hello Tensorflow")
sess = tf.Session()
print(sess.run(hello))
```

```shell
root@woody-VirtualBox:~# cd workspace/
root@woody-VirtualBox:~/workspace# jupyter notebook
```

* 交互环境中使用tensorflow

![image-20200703172851288](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703172851288.png)



* 在容器中使用Tensorflow

  ![image-20200703174556543](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703174556543.png)

  ```shell
  docker pull tensorflow/tensorflow:latest-jupyter
  # docker pull tensorflow/tensorflow:latest-gpu
  # docker pull tensorflow/tensorflow
  
  [root@woody demo]# docker run -it -p 8888:8888 -v $(pwd):/tf/notebooks tensorflow/tensorflow:latest-jupyter
  # woody
  ```

# TensorFlow基本概念解析

## TensorFlow模块与架构介绍

![image-20200703180545350](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703180545350.png)

![image-20200703180722159](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703180722159.png)

## TensorFlow数据流图介绍

![image-20200703181322146](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703181322146.png)

![image-20200703181427586](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703181427586.png)

![image-20200703182730874](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200703182730874.png)

## 张量（Tensor）是什么

![image-20200704153507801](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704153507801.png)

![image-20200704153644216](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704153644216.png)

![image-20200704153803045](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704153803045.png)

![image-20200704154127786](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704154127786.png)

![image-20200704154323177](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704154323177.png)

![image-20200704160955181](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704160955181.png)

![image-20200704161023950](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704161023950.png)

![image-20200704161055547](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704161055547.png)

## 变量（Variable）是什么

![image-20200704161449957](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704161449957.png)

* W、b 是变量（模型参数）

  ```python
  import tensorflow as tf
  # 创建变量
  w = tf.Variable(<initial-value>, name=<optional-name>)
  # 将变量作为操作的输入
  y = tf.matmul(w, ...another variable or tensor...)
  z = tf.sigmoid(w + y)
  # 使用assign 或 assign_xxx 方法重新给变量赋值
  w.assign(w + 1.0)
  w.assign_add(1.0)
  ```

![image-20200704162038824](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704162038824.png)

* 此处tensorflow版本为1.12

![image-20200704162640756](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704162640756.png)

![image-20200704162731317](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704162731317.png)

![image-20200704163104138](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704163104138.png)

![image-20200704163444987](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704163444987.png)

![image-20200704163324529](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704163324529.png)

## 操作（Operation）是什么

![image-20200704163707733](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704163707733.png)

![image-20200704163756560](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704163756560.png)

![image-20200704163930906](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704163930906.png)

![image-20200704164552205](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704164552205.png)

![image-20200704164445953](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704164445953.png)

## 会话（Session）是什么

![image-20200704164815402](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704164815402.png)

![image-20200704165053654](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704165053654.png)

![image-20200704165229831](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704165229831.png)

![image-20200704165529373](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704165529373.png)

![image-20200704165707346](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704165707346.png)

![image-20200704165729521](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704165729521.png)

## 优化器（Optimizer）是什么

![image-20200704170006153](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170006153.png)

![image-20200704170236891](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170236891.png)

![image-20200704170307743](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170307743.png)

![image-20200704170442659](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170442659.png)

![image-20200704170608524](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170608524.png)

![](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170846273.png)

![image-20200704170935128](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704170935128.png)

![image-20200704171248336](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704171248336.png)

![image-20200704171328103](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200704171328103.png)



