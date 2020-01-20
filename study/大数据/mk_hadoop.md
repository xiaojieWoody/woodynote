# Hadoop

* reliable、scalable、distributed computing
* 提供分布式的存储（一个文件被拆分成很多个块，并且以副本的方式存储在各个节点中）和计算
* 是一个分布式的系统基础架构：用户可以在不了解分布式底层细节的情况下进行使用
  * 分布式文件系统：HDFS实现将文件分布式存储在和很多的服务器上
  * 分布式计算框架：MapReduce实现在很多机器上分布式并行计算
  * 分布式资源调度框架：YARN实现集群资源管理以及作业的调度

## HDFS

* 分布式文件系统

* 源自于Google的GFS论文，论文发表于2003年10月
* HDFS是GFS的克隆版
* HDFS特点：扩展性&容错性&海量数据存储
* 将文件切分成指定大小的数据块并以多副本的存储在多个机器上（默认3副本）
* 数据切分、多副本、容错等操作对用户是透明的

![image-20200115221535579](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200115221535579.png)

## MapReduce

* 分布式计算框架
* 源自于Google的MapReduce论文，论文发表于2004年12月
* MapReduce是Google MapReduce的克隆版
* MapReduce特点：扩展性&容错性&海量数据离线处理

![image-20200115221734526](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200115221734526.png)

## YARN

* 资源调度系统 Yet Another Resource Negotiator
* 负责整个集群资源的管理和调度
* YARN特点：扩展性&容错性&多框架资源统一调度

![image-20200115222021475](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200115222021475.png)

## 优势

* 高可靠性
  * 数据存储：数据块多副本
  * 数据计算：重新调度作业计算
* 高扩展性
  * 存储/计算资源不够时，可以横向的线性扩展机器
  * 一个集群中可以包括数以千计的节点

## 狭义Hadoop VS 广义Hadoop

* 狭义的Hadoop：是一个适合大数据分布式存储（HDFS）、分布式计算（MapReduce）和资源调度（YARN）的平台
* 广义的Hadoop：指的是Hadoop生态系统，Hadoop生态系统是一个很庞大的概念，Hadoop是其中最重要最基础的一个部分；生态系统中的每一子系统只解决某一个特定的问题域（甚至可能很窄），不搞统一型的一个全能系统，而是小而精的多个小系统

![image-20200115222725188](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200115222725188.png)

## 版本选择

* 常用的Hadoop发行版
  * Apache
    * 优点：纯开源
    * 缺点：不同版本/不同框架之间整合 jar冲突。。。吐血
  * CDH：https://www.cloudera.com/
    * 优点：cm（cloudera manager），通过页面一键安装各种框架、升级、impala
    * 缺点：cm不开源、与社区版本有些许出入
  * Hortonworks：HDP 企业发布自己的数据平台可以直接基于页面框架进行改造
    * 优点：原装Hadoop、纯开源、支持tez
    * 缺点：企业级安全不开源

# HDFS

## 概述 

* 分布式的文件系统，能够横跨N个机器
* commodity hardware
* fault-tolerant容错
* high throughput
* large data sets

## 前提和设计目标

* Hardware Failure 硬件错误
  * 每个机器只存储文件的部分数据，blocksize=128M
  * block存放在不同的机器上的，由于容错，HDFS默认采用3副本机制
* Streaming Data Access 流式数据访问
  * The emphasis is on high throughout of data access rather than low latency of data access
* Large Data Sets 大规模数据集
* Moving Computation is Cheaper than Moving Data

## 架构

1. NameNode（master）and DataNodes（slave）
2. master / slave的架构
3. NN：
   * the file system namespace
   * regulates access to files by clients
4. DN：storage
5. HDFS expose a file system namespace and allows user data
6. a file is split into one or more blocks
   * blocksize：128M
   * 150M拆成2个block
7. blocks are stored in a set of DataNodes
   * 为什么？容错
8. NameNode executes file system namespace operations：CRUD
9. detrmines the mapping of blocks to DataNodes
   * a.txt	150M	blocksize=128M
   * a.txt    拆分成2个block 一个是block1：128M 另一个block2：22M
   * block1存放在哪个DN？block2存放在哪个DN？
     * a.txt
       * block1：128M，192.168.199.1
       * block2：22M，192.168.199.2
     * get a.txt
     * 这个过程对于用户来说是不感知的

3.4

