# Tensorflow简介与环境搭建

## Tensorflow是什么

* Google的开源软件库
  * 采取数据流图，用于数值计算
  * 支持多种平台-CPU、GPU、移动设备
  * 最初用于深度学习，变得越来越通用
* 数据流图
  * 节点-处理数据
  * 线-节点间的输入输出关系
  * 线上运输张量
  * 节点被分配到各种计算设备上运行

![image-20200702164414454](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702164414454.png)

* 特性
  * 高度的灵活性
  * 真正的可移植性
  * 产品和科研结合
  * 自动求微分
  * 多语言支持
  * 性能最优化

## Tensorflow1.0

* 主要特性
  * XLA——Accelerate Linear Algebra
    * 提升训练速度58倍
    * 可以在移动设备运行
  * 引入更高级别的API——tf.layers / tf.metrics / tf.losses / tf.keras
  * Tensorflow调试器
  * 支持docker镜像，引入tensorflow serving服务

* 架构

![image-20200702165211852](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702165211852.png)

## Tensorflow2.0

* 主要特性
  * 使用tf.keras和eager mode进行更加简单的模型构建
  * 鲁棒的跨平台模型部署
  * 强大的研究实验
  * 清除不推荐使用的API和减少重复来简化API

* 架构

![image-20200702165434337](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702165434337.png)

* 简化的模型开发流程
  * 使用tf.data加载数据
  * 使用tf.keras构建模型，也可以使用premade estimator来验证模型
    * 使用tensorflow hub进行迁移学习
  * 使用eager mode进行运行和调试
  * 使用分发策略来进行分布式训练
  * 导出到SavedModel
  * 使用Tensorflow Serve、Tensorflow Lite、Tensorflow.js部署模型

* 强大的跨平台能力
  * Tensorflow服务	
    * 直接通过HTTP/REST或GRPC/协议缓冲区
  * Tensorflow Lite——可部署在Android、iOS和嵌入式系统上
  * Tensorflow.js——在javascript中部署模型
  * 其他语言
    * C、Java、GO、C#、Rust、Julia、R等

* 强大的研究实验
  * Keras功能API和子类API，允许创建复杂的拓扑结构
  * 自定义训练逻辑，使用tf.GradientTape和tf.custom_gradient进行更细粒度的控制
  * 低层API自始至终可以与高层结合使用，完全的可定制
  * 高级扩展：Ragged Tensors、Tensor2Tensor等

## Tensorflow vs PyTorch

* 入门时间
  * Tensorflow1.*
    * 静态图
    * 学习额外概念：图、会话、变量、占位符等
    * 写样板代码
  * Tensorflow2.0
    * 动态图
    * Eager mode避免1.0缺点，直接集成在python中
  * Pytorch
    * 动态图
    * Numpy的扩展，直接集成在python中
    * ![image-20200702170229777](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702170229777.png)
* 图创建和调试
  * Tensorflow1.*
    * 静态图，难以调试，学习tfdbg调试
  * Tensorflow2.0 和 PyTorch
    * 动态图，python自带调试的工具
* 全面性
  * Pytorch缺少
    * 沿维翻转张量(np.flip，np.flipud，np.fliplr)
    * 检查无穷与非数值张量(np.is_nan，np.is_inf)
    * 快速傅立叶变换(np.fft)
  * 随着时间变化，越来越接近
* 序列化和部署
  * Tensorflow支持更加广泛
    * 图保存为protocol buffer
    * 跨语言
    * 跨平台
  * Pytorch支持比较简单

![image-20200702170651599](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702170651599.png)

![image-20200702170718881](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702170718881.png)

![image-20200702170813268](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702170813268.png)

## tensorflow环境配置

* 本地配置
  * Virtualenv安装——www.tensorflow.org/install/pip
  * GPU版环境配置
    * https://blog.csdn.net/u014595019/article/details/53732015
    * 安装显卡驱动->Cuda安装->Cudnn安装
    * Tensorflow安装

* docker安装

* 云配置

  * 规格统一，节省自己的机器
  * 有直接配置好环境的镜像
  * Google Cloud配置——送300刀免费体验
  * Amazon云配置
  * 实战
    * 从0配置
    * 从镜像配置

* ubuntu18.04

  ```shell
  sudo apt-get install python3
  # sudo apt-get install python
  python3
  # sudo apt-get install software-properties-common
  # sudo apt-add-repository universe
  sudo apt-get update
  sudo apt-get install python3-pip
  # 安装virtualenv
  sudo pip3 install -U virtualenv
  ```

  ```shell
  # ~ 目录
  mkdir environment
  cd environment
  # 配置python虚拟环境
  virtualenv --system-site-packages -p pyhton3 ./tf_py3
  # 激活python虚拟环境
  source tf_py3/bin/activate
  # 退出python虚拟环境
  deactivate
  # (tf_py3)...
  python3
  # 安装tensorflow
  pip install tensorflow -i https://pypi.tuna.tsinghua.edu.cn/simple
  # 指定版本
  pip install tensorflow==2.2.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
  # 验证tensorflow版本
  python
  >>> import tensorflow as tf
  >>> print(tf.__version__)
  # （tf_py3）安装jupyter
  pip install numpy matplotlib sklearn pandas jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple
  # 启动jupyter
  jupyter notebook
  # 配置jupyter
  # environment上层目录
  ls -a            # 没有.jupyter配置
  .ssh environment .profile .python_history .keras .gnupg .cache .bashrc .bash_logout
  # 生成jupyter配置文件
  jupyter notebook --generate-config
  ls -a          # 有.jupyter配置
  cd .jupyter/
  ls
  vim jupyter_notebook_config.py
  	## This is an application.
  	c = get_config()
  	c.NotebookApp.ip = '*'
  	c.NotebookApp.open_browser = False
  	c.NotebookApp.port = 6006
  	# with hostnames configured in local_hostnames
  	c.NotebookApp.allow_remote_access = True
  # environment同级目录创建 workspace
  cd workspace
  jupyter notebook
  # http://xxxx:6006/?token=xxxx
  # 第一次登陆页面，直接在页面的Token下输入token的值，然后设置New Password
  # jupyter的密码：woody
  ```

* 重启服务器后

  ```python
  # 激活python环境
  source environment/tf_py3/bin/activate
  # 切换虚拟环境：直接source另一个虚拟环境即可
  # 运行jupyter notebook
  cd workspace
  jupyter notebook
  ```

* 安装GPU版本
  * ubuntu18.04 LTS
  * GPU1
  * https://tensorflow.google.cn/install/gpu

![image-20200702175757726](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702175757726.png)

![image-20200702175911428](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702175911428.png)

```shell
# 无法定位软件包：cuda问题解决方法
apt-get install nvidia-cuda-dev nvidia-cuda-toolkit
```

![image-20200702180123897](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180123897.png)

![image-20200702180347357](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180347357.png)

![image-20200702180411236](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180411236.png)

![image-20200702180501487](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180501487.png)

![image-20200702180642724](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180642724.png)

![image-20200702180709678](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180709678.png)

![image-20200702180850315](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180850315.png)

![image-20200702180948394](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702180948394.png)

![image-20200702181452256](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702181452256.png)

![image-20200702181619620](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200702181619620.png)

