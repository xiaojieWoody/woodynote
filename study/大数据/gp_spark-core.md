# Spark简介

## RDD

* 弹性分布式数据集RDD
* Spark将数据缓存在分布式内存中
* 如何实现？RDD
  * Spark的核心
  * 分布式内存抽象
  * 提供了一个高度受限的共享内存模型
  * 逻辑上集中但是物理上是存储在集群的多台机器上

* 属性和特点
  * 只读
    * 通过HDFS或者其他持久化系统创建RDD
    * 通过transformation将父RDD转化得到新的RDD
    * RDD上保存着前后之间的依赖关系
  * Partition
    * 基本组成单位，RDD在逻辑上按照Partition分块
      * Partition0、Partition1、Partition2...
    * 分布在各个节点上
    * 分片数量决定并行计算的粒度
    * RDD中保存如何计算每一个分区的函数
  * 容错
    * 失败自动创建
    * 如果发生部分分区数据丢失，可以通过依赖关系重新计算

![image-20200607093553450](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607093553450.png)

![image-20200607093651721](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607093651721.png)



# Scala编程基础

![image-20200607093452840](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607093452840.png)

# Spark体系结构和源代码解析

# Spark编程模型

## 核心思想

* 函数式编程
  * 数据映射
  * 变量不可变
  * 没有副作用
  * 一等公民

![image-20200607094024157](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607094024157.png)

* 对于RDD有四种类型的算子

  * Create
    * SparkContext.textFile()
    * SparkContext.parallelize()
  * Transformation
    * 作用于一个或者多个RDD，输出转换后的RDD
    * 例如：map、filter、groupBy
  * Action
    * 会触发Spark提交作业，并将结果返回Driver Program
    * 例如：reduce，countByKey
  * Cache
    * cache缓存
    * persist持久化

* 惰性运算：遇到Action时才会真正的执行

  ```scala
  // Example
  val lines = sc.textFile("/hdfs/parth")
  lines.filter(x => x.contains("ERROR")).count()
  // filter属于transformation算子
  // count属于Action算子
  ```

* 运行Spark方式
  * CDH集群上运行Spark-Shell
    * 在shell中输入spark-shell --master yarn-client
  * 使用Zeppelin
    * sudo docker run -p 8080:8080 --rm --name zeppelin apache/zeppelin:0.8.1
    * https://zeppelin.apache.org
  * 使用Spark-Submit递交作业
* Spark API文档
  * [官方文档](https://spark.apache.org/docs/latest)
* cdh2.4.0

## Value类型 Transformation算子

* Transformation算子分类
  * 输入分区与输出分区一对一：map、flatMap、mapPartitions
  * 输入分区与输出分区多对一：union、cartesian
  * 输入分区与输出分区多对多：groupBy
  * 输出分区是输入分区的子集：filter、distinct、subtract、sample、intersection
  * Cache类型：cache、persist

* ![image-20200607095317978](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607095317978.png)
* ![image-20200607095909772](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607095909772.png)
* ![image-20200607100045823](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607100045823.png)
* ![image-20200607095539333](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607095539333.png)
* ![image-20200607100103278](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607100103278.png)
* ![image-20200607100239245](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607100239245.png)

* ![image-20200607100337426](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607100337426.png)
* ![image-20200607100433879](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607100433879.png)

## Key-Value类型 Transformation算子

* 分类

  * 输入分区与输出分区一对一：mapValues
  * 对单个RDD聚集：groupByKey、reduceByKey、combineByKey、partitionBy、aggregateByKey
  * 对两个RDD聚集：cogroup
  * 连接：join、leftOuterJoin、rightOuterJoin

* ![image-20200607100838032](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607100838032.png)

  ![image-20200607101022730](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607101022730.png)

  

* ![image-20200607101054609](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607101054609.png)

  

  ![image-20200607101230280](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607101230280.png)

* ![image-20200607101346713](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607101346713.png)

  ![image-20200607101438729](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607101438729.png)

  * 两个函数

    * 初始值(0,0)
    * 第一个是累加动作
      * `(acc,value) => (acc._1 + value,acc._2+1)` 
    * 第二个是关于每个分区怎么合并
      * `(acc1,acc2) => (acc1._1 + acc2._1,acc1._2 + acc2._2)`

    ![image-20200607102150216](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607102150216.png)

* ![image-20200607102213245](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607102213245.png)

## Action算子分类

* 分类
  * 无输出：foreach
  * 操作HDFS：saveAsTextFile、saveAsObjectFile
  * 统计类：count、countByKey、countByValue
  * 集合类：collect、take、takeOrdered、top
  * 聚合类：reduce、fold、aggregate
* ![image-20200607102352160](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607102352160.png)
  * countByKey：查看key的比例是否不均衡，数据是否倾斜
* ![image-20200607102625245](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607102625245.png)
* ![image-20200607111254094](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607111254094.png)
* ![image-20200607111440086](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607111440086.png)
* ![image-20200607111541872](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607111541872.png)

# Spark内存模型

## Yarn资源调度过程

![image-20200607111623013](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607111623013.png)

## Spark内存结构

![image-20200607111856606](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607111856606.png)

![image-20200607112554964](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607112554964.png)

![image-20200607112657864](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607112657864.png)



# Spark案例介绍

![image-20200607112824755](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607112824755.png)

![image-20200607112916246](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607112916246.png)

![image-20200607145942133](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607145942133.png)

![image-20200607150054043](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150054043.png)

![image-20200607113146492](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607113146492.png)

![image-20200607113119037](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607113119037.png)

![image-20200607150139461](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150139461.png)

![image-20200607150236434](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150236434.png)

![image-20200607150247913](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150247913.png)

![image-20200607150441098](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150441098.png)

![image-20200607150413049](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150413049.png)

![image-20200607150521182](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150521182.png)

![image-20200607150538443](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150538443.png)

![image-20200607150555420](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150555420.png)

![image-20200607150617597](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150617597.png)

![image-20200607150745922](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150745922.png)

![image-20200607150801318](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150801318.png)

![image-20200607150850153](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150850153.png)

![image-20200607150914162](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150914162.png)

![image-20200607150941955](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607150941955.png)

![image-20200607151017404](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607151017404.png)

![image-20200607151112229](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607151112229.png)



![image-20200607151210369](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607151210369.png)

