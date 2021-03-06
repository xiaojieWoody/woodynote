# Spark简介

* 是一个快速且通用的集群计算平台
* Spark是快速的
  * Spark扩充了流行的Mapreduce计算模型
  * Spark是基于内存的计算
* Spark是通用的
  * Spark的设计容纳了其他分布式系统拥有的功能
  * 批处理，迭代式计算，交互查询和流处理等
  * 优点：降低了维护成本
* Spark是高度开放的
  * Spark提供了Python，Java，Scala，SQL的API和丰富的内置库
  * Spark和其他的大数据工具整合的很好，包括hadoop，kafka等

## Spark的组件

* Spark包括多个紧密集成的组件

  ![image-20200602094320729](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602094320729.png)

* Spark Core：

  * 包含Spark的基本功能，包含任务调度，内存管理，容错机制等
  * 内部定义了RDDs（弹性分布式数据集）
  * 提供了很多APIs来创建和操作这些RDDs
  * 应用场景，为其他组件提供底层的服务

* Spark SQL：

  * 是Spark处理结构化数据的库，就像Hive SQL、MySQL一样
  * 应用场景，企业中用来做报表统计

* Spark Streaming

  * 是实时数据流处理组件，类似Storm
  * Spark Streaming提供了API来操作实时流数据
  * 应用场景，企业中用来从Kafka接收数据做实时统计

* Mlib

  * 一个包含通用机器学习功能的包，Machine learning lib
  * 包含分类，聚类，回归等，还包括模型评估，和数据导入
  * MLlib提供的上面这些方法，都支持集群上的横向扩展
  * 应用场景，机器学习

* Graphx：

  * 是处理图的库（例如，社交网络图），并进行图的并行计算
  * 像Spark Streaming，Spark SQL一样，它也继承了RDD API
  * 它提供了各种图的操作，和常用的图算法，例如PangeRank算法
  * 应用场景，图计算

* Cluster Managers

  * 就是集群管理，Spark自带一个集群管理是单独调度器
  * 常见集群管理包括Hadoop YARN，Apache Mesos

* 紧密集成的优点

  * Spark底层优化了，基于Spark底层的组件，也得到了相应的优化
  * 紧密集成，节省了各个组件组合使用时的部署、测试等时间
  * 向Spark增加新的组件时，其他组件，可立刻享受新组件的功能

* Hadoop应用场景

  * 离线处理
  * 对时效性要求不高

* Spark应用场景

  * 时效性要求高的场景
  * 机器学习等领域

* 比较

  * Doug Cutting的观点
    * 这是生态系统，每个组件都有其作用，各司其职即可
    * Spark不具有HDFS的存储能力，要借助HDFS等持久化数据
    * 大数据将会孕育出更多的新技术

# Spark的安装

## Spark的运行环境

* Spark是Scala写的，运行在JVM上，所以运行环境Java 7+
* 如果使用Python API，需要安装Python2.6+或Python 3.4+
* Spark 1.6.2 - Scala 2.10
* Spark 2.0.0 - Scala 2.11

## Spark下载

* http://spark.apache.org/downloads.html
* 搭Spark不需要Hadoop，如有Hadoop集群，可下载相应的版本
* 解压

## Spark目录

* bin包含用来和Spark交互的可执行文件，如Spark shell
* core，Streaming，python，...包含主要组件的源代码
* examples包含一些单机Spark job，可以研究和运行这些例子

## Spark的shell

* Spark的shell能够处理分布在集群上的数据
* Spark把数据加载到节点的内存中，因此分布式处理可在秒级完成
* 快速迭代式计算，实时查询、分析一般能够在shells中完成
* Spark提供了Python shells和Scala shells

## Python Shell

```shell
# bin目录下执行
./pyspark
```

## Scala Shell

```shell
# bin目录下执行
./spark-shell
```

* 修改日志级别

  ```shell
  cd conf
  cp log4j.properties.template log4j.properties
  vi log4j.properties
  	log4j.rootCategory=WARN,console
  # cd bin
  ./spark-shell
  ```

* 例子

  ```shell
  [root@instance-0zsp0b3k-3 testfile]# pwd
  /data/spark_dev/testfile
  [root@instance-0zsp0b3k-3 testfile]# cat helloSpark
  hello Spark
  hello World
  hello Spark !
  [root@instance-0zsp0b3k-3 testfile]#
  ```

  ```scala
  val lines = sc.textFile("../../testfile/helloSpark")
  lines.count()
  lines.first()
  ```

# Spark开发环境搭建

* idea安装scala、sbt插件

* 新建scala项目，选择SBT

  ```shell
  # /xxx/.sbt目录下创建repositories文件
  [repositories]
  local
  huaweicloud-maven: https://repo.huaweicloud.com/repository/maven/
  maven-central: https://repo1.maven.org/maven2/
  huaweicloud-ivy: https://repo.huaweicloud.com/repository/ivy/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
  ```

## 开发第一个Spark程序

* 配置ssh无密登陆
  * ssh-keygen
  * .ssh目录下 cat xxx_rsa.pub > authorized_keys
    * 如果authorized_keys不存在，则touch authorized_keys
  * chmod 600 authorized_keys

* WordCount

  * 创建一个Spark Context

  * 加载数据

  * 把每一行分割成单词

  * 转换成pairs并且计数

    ```shell
    [root@instance-0zsp0b3k-3 testfile]# cat helloSpark2.txt
    hello spark
    hello world
    hello !
    [root@instance-0zsp0b3k-3 testfile]# pwd
    /data/spark_dev/testfile
    ```

    ```shell
    # build.sbt
    name := "my_scala"
    version := "0.1"
    # 注意版本，版本不一致会有各种问题
    # scalaVersion := "2.11.12"
    scalaVersion := "2.10.5"
    libraryDependencies ++= Seq(
    #  "org.apache.spark" %% "spark-core" % "2.4.0"
   "org.apache.spark" %% "spark-core" % "1.6.2"
    )
    ```
    
    ```scala
    import org.apache.spark.{SparkConf, SparkContext}
    
    object WordCount {
      def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setAppName("wordcount")
        val sc = new SparkContext(conf)
    		// 需处理文件
        val input = sc.textFile("/data/spark_dev/testfile/helloSpark2.txt")
        val lines = input.flatMap(line=>line.split(" "))
        val count = lines.map(word=>(word,1)).reduceByKey{case (x,y)=>x+y}
    		// 结果存放目录
        val output = count.saveAsTextFile("/data/spark_dev/testfile/helloSpark2Res")
      }
    }
    ```
    
  * 打包

    * `File - Project Structure - Artifacts - + - JAR - From modules with dependencies`

      ![image-20200602143932007](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602143932007.png)

    * 如果提示已存在，则将项目中`META-INF`下的`MANIFEST.MF`删除

    * `Build - Build Artifacts..`，在`out`目录下会生成项目`jar`包

  * 启动master

    ```shell
    # 启动master
    ./sbin/start-master.sh
    # jps 查看启动
    jps
    [root@instance-0zsp0b3k-3 sbin]# jps
    12054 Jps
    2842 Master
    
    # http://106.13.82.229:8080/
    spark://instance-0zsp0b3k-3:7077
    ```

    ![image-20200602132025568](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602132025568.png)

  * 启动worker

    ```shell
    ./bin/spark-class org.apache.spark.deploy.worker.Worker spark://instance-0zsp0b3k-3:7077
    # jps 查看启动
    [root@instance-0zsp0b3k-3 /]# jps
    14852 Jps
    14378 Worker
    2842 Master
    ```

  * 提交作业

    ```shell
    # 上传作业jar包
    # 提交作业
    ./bin/spark-submit --master spark://instance-0zsp0b3k-3:7077 --class WordCount /data/spark_dev/my_scala.jar
    ```

  * 结果

    ```shell
    # 结果生成目录 helloSpark2Res
    [root@instance-0zsp0b3k-3 helloSpark2Res]# cat part-00000
    (hello,3)
    (world,1)
    [root@instance-0zsp0b3k-3 helloSpark2Res]# cat part-00001
    (spark,1)
    (!,1)
    ```

# RDDs

## 介绍

### Driver Program：

* 包含程序的`main()`方法，RDDs的定义和操作

* 它管理很多节点，我们称作`executors`

  ![image-20200602144920581](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602144920581.png)

### SparkContext：

* `Driver programs`通过`SparkContext`对象访问`Spark`
* `SparkContext`对象代表和一个集群的连接
* 在`Shell`中`SparkContext`自动创建好了，就是sc

### RDDs

* `Resilient distributed datasets`（弹性分布式数据集，简写`RDDs`）

* 这些`RDDs`，并行的分布式在整个集群中

* `RDDs`是`Spark`分发数据和计算的基础抽象类

  ```shell
  # lines 相当于RDDs
  scala> val lines = sc.textFile("/data/spark_dev/testfile/helloSpark.txt")
  ```

* 一个`RDD`是一个不可改变的分布式集合对象
* `Spark`中，所有的计算都是通过`RDDs`的创建、转换、操作完成的
* 一个`RDD`内部由许多`partitions`（分片）组成

### 分片

* 每个分片包括一部分数据，`partitions`可在集群不同节点上计算
* 分片是`Spark`并行处理的单元，`Spark`顺序的，并行的处理分片

### RDDs的创建方法

* 把一个存在的集合传给`SparkContext`的`parallelize()`方法，测试用

  ```scala
  // 第1个参数：待并行化处理的集合，第2个参数：分区个数
  val rdd = sc.parallelize(Array(1,2,2,4),4)
  rdd.count()
  rdd.foreach(println)
  ```

* 加载外部数据集

  ```scala
  val rddText = sc.textFile("helloSpark.txt")
  ```

## Scala的基础知识

### Scala的变量声明

* 在Scala中创建变量的时候，必须使用`val`或`var`

* `val`，变量值不可修改，一旦分配不能重新指向别的值

* `var`，分配后，可以指向类型相同的值

* 举例

  ```scala
  val lines = sc.textFile("helloSpark.txt")
  lines = sc.textFile("helloSpark2.txt")
  var lines2 = sc.textFile("helloSpark.txt")
  lines2 = sc.textFile("helloSpark2.txt")
  ```

### Scala的匿名函数和类型推断

* 定义一个匿名函数，接收一个参数`line`，使用`line`这个`String`类型变量上的`contains`方法，并且返回结果
* `line`的类型不需指定，能够推断出来

```scala
val lines2 = lines.filter(line => line.contains("world"))
lines2.foreach(println)
```

## Transformation

* 转换
* 从之前的`RDD`构建一个新的`RDD`，像`map()`和`filter()`

### map()

* `map()`接收函数，把函数应用到RDD的每一个元素，返回新的RDD

  ```scala
  val lines = sc.parallelize(Array("hello","spark","hello","world","!"))
  lines.foreach(println)
  val lines2 = lines.map(word => (word,1))
  lines.foreach(println)
  (hello,1)
  (spark,1)
  (hello,1)
  (world,1)
  (hello,1)
  (!,1)
  ```

### filter()

* `filter()`接收函数，返回只包含满足`filter()`函数的元素的新`RDD`

  ```scala
  val lines3 = lines.filter(word => word.contains("hello")) 
  lines3.foreach(println)
  ```

### flatMap()

* `flatMap()`

  * 对每个输入元素，输出多个输出元素
  * `flat`压扁的意思，将`RDD`中元素压扁后返回一个新的`RDD`

  ```scala
  val lines = inputs.flatMap(line => line.split(" "))
  lines.foreach(println)
  hello
  spark
  hello
  world
  hello
  !
  ```

### 集合运算

* `RDDs`支持数学集合的计算，例如并集，交集计算

  ```scala
  val rdd1 = sc.parallelize(Array("coffe","coffe","panda","monkey","tea"))
  val rdd2 = sc.parallelize(Array("coffe","monkey","kitty"))
  val rdd_distinct = rdd1.distinct()
  rdd_distinct.foreach(println)
  val rdd_union = rdd1.union(rdd2)
  rdd_union.foreach(println)
  // 交集
  val rdd_inter = rdd1.intersection(rdd2)
  rdd_inter.foreach(println)
  // 差
  val rdd_sub = rdd1.subtract(rdd2)
  rdd_sub.foreach(println)
  ```

## Action 

* 在`RDD`上计算出来一个结果

* 把结果返回给`driver program`或保存在文件系统，`count()`，`save`

  ![image-20200602155829589](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602155829589.png)

  ![image-20200602155845044](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602155845044.png)

### reduce()

* 接收一个函数，作用在`RDD`两个类型相同的元素上，返回新元素

* 可以实现，`RDD`中元素的累加，计数，和其它类型的聚集操作

  ```scala
  val rdd = sc.parallelize(Array(1,2,3,3))
  rdd.collect()
  rdd.reduce((x,y) => x + y)
  ```

### collect()

* 遍历整个`RDD`，向`driver program` 返回`RDD`的内容
* 需要单机内存能够容纳下（因为数据要拷贝给`driver`，测试使用）
* 大数据的时候，使用`saveAsTextFile() action`等

### take(n)

* 返回`RDD`的`n`个元素（同时尝试访问最少的`partitions`）
* 返回结果是无序的，测试使用

### top(n)

* 排序（根据`RDD`中的数据的比较器）

### foreach()

* 计算`RDD`中的每个元素，但不返回到本地
* 可以配合`println()`友好的打印出数据

## RDDs的特性

### RDDs的血统关系图

* `Spark`维护着`RDDs`之间的依赖关系和创建关系，叫做血统关系图

* `Spark`使用血统关系图来计算每个`RDD`的需求和恢复丢失的数据

  ![image-20200602161951233](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602161951233.png)

### 延迟计算(Lazy Evaluation)：

* `Spark`对`RDDs`的计算是，他们第一次使用`action`操作的时候
* 这种方式在处理大数据的时候特别有用，可以减少数据的传输
* `Spark`内部记录`metadata`表名`transformations`操作已经被响应了
* 加载数据也是延迟计算，数据只有在必要的时候，才会被加载进去

### RDD.persist()

* 默认每次在`RDDs`上面进行`action`操作时，`Spark`都重新计算`RDDs`

* 如果想重复利用一个`RDD`，可以使用`RDD.persist()`

* `unpersist()`方法从缓存中移除

* 例子-`persist()`

  ![image-20200602162513539](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602162513539.png)

## KeyValue对RDDs

### 创建KeyValue对RDDs

* 使用`map()`函数，返回`key/value`对

* 例如，包含数行数据的`RDD`，把每行数据的第一个单词作为`keys`

  ```scala
  val rdd = sc.textFile("/data/spark_dev/helloSpark.txt")
  rdd.foreach(println)
  val rdd2 = rdd.map(line => (line.split(" ")(0), line))
  rdd2.foreach(println)
  ```

  ![image-20200602163102768](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602163102768.png)

  ```scala
  val rdd3 = sc.parallelize(Array((1,2),(3,4),(3,6)))
  rdd3.foreach(println)
  val rdd4 = rdd3.reduceByKey((x,y) => x + y)
  rdd4.foreach(println)
  (1,2)
  (3,10)
  val rdd5 = rdd3.groupByKey()
  rdd5.foreach(println)
  (1,CompactBuffer(2))
  (3,CompactBuffer(4, 6))
  ```

  ![image-20200602163516846](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200602163516846.png)

### combineByKey()

* `combineByKey`是`Spark`中一个比较核心的高级函数，其他一些高阶键值对函数底层都是用它实现的。诸如 `groupByKey`,`reduceByKey`等等

* `createCombiner`、`mergeValue`、`mergeCombiners`、`partitioner`

* 遍历`partition`中的元素，元素的`key`，要么之前见过的，要么不是

* 如果是新元素，使用我们提供的`createCombiner()`函数

  * `createCombiner: V => C` ，这个函数把当前的值作为参数，此时可以对其做些附加操作(类型转换)并把它返回 (这一步类似于初始化操作)

* 如果是这个`partition`中已经存在的key，就会使用`mergeValue()`函数

  * `mergeValue: (C, V) => C`，该函数把元素V合并到之前的元素C(createCombiner)上 (这个操作在每个分区内进行)

* 合计每个`partition`的结果的时候，使用`mergeCombiners()`函数

  * `mergeCombiners: (C, C) => C`，该函数把2个元素C合并 (这个操作在不同分区间进行)

* 例子：求平均值

  ```scala
  val scores = sc.parallelize(Array(("jack",80.0),("jack",90.0),("jack",85.0),("mike",85.0),("mike",92.0),("mike",90.0)))
  scores.foreach(println)
  val score2 = scores.combineByKey(score => (1,score),(c1:(Int,Double),newScore) => (c1._1+1,c1._2+newScore),(c1:(Int,Double),c2:(Int,Double)) => (c1._1+c2._1,c1._2+c2._2))
  score2.foreach(println)
  (jack,(3,255.0))
  (mike,(3,267.0))
  val average = scores2.map{case(name,(num,score)) => (name,score/num)}
  average.foreach(println)
  (mike,89.0)
  (jake,85.0)
  ```

  ```scala
  // 使用combineByKey来求解平均数
  val initialScores = Array(("Fred", 88.0), ("Fred", 95.0), ("Fred", 91.0), ("Wilma", 93.0), ("Wilma", 95.0), ("Wilma", 98.0))
  val d1 = sc.parallelize(initialScores)
  type MVType = (Int, Double) //定义一个元组类型(科目计数器,分数)
  d1.combineByKey(
    score => (1, score),
    (c1: MVType, newScore) => (c1._1 + 1, c1._2 + newScore),
    (c1: MVType, c2: MVType) => (c1._1 + c2._1, c1._2 + c2._2)
  ).map { case (name, (num, socre)) => (name, socre / num) }.collect
  
  // res1: Array[(String, Double)] = Array((Wilma,95.33333333333333), (Fred,91.33333333333333))
  
  // score => (1, score)，我们把分数作为参数,并返回了附加的元组类型。 以"Fred"为列，当前其分数为88.0 =>(1,88.0)  1表示当前科目的计数器，此时只有一个科目
  // (c1: MVType, newScore) => (c1._1 + 1, c1._2 + newScore)，注意这里的c1就是createCombiner初始化得到的(1,88.0)。在一个分区内，我们又碰到了"Fred"的一个新的分数91.0。当然我们要把之前的科目分数和当前的分数加起来即c1._2 + newScore,然后把科目计算器加1即c1._1 + 1
  // (c1: MVType, c2: MVType) => (c1._1 + c2._1, c1._2 + c2._2)，注意"Fred"可能是个学霸,他选修的科目可能过多而分散在不同的分区中。所有的分区都进行mergeValue后,接下来就是对分区间进行合并了,分区间科目数和科目数相加分数和分数相加就得到了总分和总科目数
  ```
  
  

