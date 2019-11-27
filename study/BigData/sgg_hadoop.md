# 大数据

## 是什么

* 指无法在一定时间范围内用常规软件工具进行捕捉、管理和处理的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化能力的海量、高增长率和多样化的信息资产

* 主要解决，海量数据的存储和海量数据分析计算问题

* 特点

  * 大量
  * 高速
  * 多样：结构化数据和非结构化数据
  * 低价值密度

* 应用场景

  * 物流仓储：大数据分析系统助力商家精细化运营、提升销量、节约成本
  * 零售：分析用户消费习惯，为用户购买商品提供方便，从而提升商品销量
  * 旅游：深度结合大数据能力与旅游行业需求，共建旅游产业智慧管理、智慧服务和智慧营销的未来
  * 商品广告推荐：给用户推荐喜欢的商品
  * 保险、金融、房产、人工智能

  ![image-20191127140221846](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127140221846.png)

# Hadoop

## 简介

* 是一个分布式系统基础架构
* 主要解决，海量数据的存储和海量数据的分析计算问题
* 广义上来说，通常指Hadoop生态圈
* 优势
  * 高可用性
    * Hadoop底层维护多个数据副本，所以即使Hadoop某个计算元素或存储出现故障，也不会导致数据的丢失
  * 高扩展性
    * 在集群间分配任务数据，可方便的扩展数以千计的节点
  * 高效性
    * 在MapReduce的思想下，Hadoop是并行工作的，以加快任务处理速度
  * 高容错性
    * 能够自动将失败的任务重新分配
* ![image-20191127141434422](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127141434422.png)

## HDFS

* NameNode（nn）：存储文件的元数据，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限）、以及每个文件的块列表和块所在的DataNode等
* DataNode（dn）：在本地文件系统存储文件块数据，以及块数据的校验和
* Secondary NameNode（2nn）：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照

## YARN

![image-20191127143223570](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127143223570.png)

1. ResourceManager（RM）主要作用如下
   * 处理客户端请求
   * 监控NodeManager
   * 启动或监控ApplicationMaster
   * 资源的分配与调度
2. NodeManager（NM）主要作用如下
   * 管理单个节点上的资源
   * 处理来自ResourceManager的命令
   * 处理来自ApplicationMaster的命令
3. ApplicationMaster（AM）作用如下
   * 负责数据的切分
   * 为应用程序申请资源并分配给内部的任务
   * 任务的监控与容错
4. Container
   * 是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等

## MapReduce

* MapReduce将计算过程分为两个阶段：Map和Reduce
  * Map阶段并行处理输入数据
  * Reduce阶段对Map结果进行汇总

## 大数据技术生态体系

![image-20191127144323534](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127144323534.png)

* Sqoop：Sqoop是一款开源的工具，主要用于在Hadoop、Hive与传统的数据库(MySql)间进行数据的传递，可以将一个关系型数据库（例如 ：MySQL，Oracle 等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中
* Flume：Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力
* Kafka：Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：
  * 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能
  * 高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息
  * 支持通过Kafka服务器和消费机集群来分区消息
  * 支持Hadoop并行数据加载
* Storm：Storm用于“连续计算”，对数据流做连续查询，在计算时就将结果以流的形式输出给用户
* Spark：Spark是当前最流行的开源大数据内存计算框架。可以基于Hadoop上存储的大数据进行计算
* Oozie：Oozie是一个管理Hadoop作业（job）的工作流程调度管理系统
* Hbase：HBase是一个分布式的、面向列的开源数据库。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库
* Hive：Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析
* R语言：R是用于统计分析、绘图的语言和操作环境。R是属于GNU系统的一个自由、免费、源代码开放的软件，它是一个用于统计计算和统计制图的优秀工具
* Mahout：Apache Mahout是个可扩展的机器学习和数据挖掘库
* ZooKeeper：Zookeeper是Google的Chubby一个开源的实现。它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、 分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户

## 推荐系统架构

![image-20191127144949632](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127144949632.png)

## 搭建环境

## 运行模式

### 本地模式

* 官方Grep案例

  ```shell
  [atguigu@hadoop101 hadoop-2.7.2]$ mkdir input
  [atguigu@hadoop101 hadoop-2.7.2]$ cp etc/hadoop/*.xml input
  # 执行share目录下的MapReduce程序
  [atguigu@hadoop101 hadoop-2.7.2]$ bin/hadoop jar
  share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
  # 查看输出结果
  [atguigu@hadoop101 hadoop-2.7.2]$ cat output/*
  ```

* 官方WordCount案例

  ```shell
  [atguigu@hadoop101 hadoop-2.7.2]$ mkdir wcinput
  [atguigu@hadoop101 hadoop-2.7.2]$ cd wcinput
  [atguigu@hadoop101 wcinput]$ touch wc.input
  [atguigu@hadoop101 wcinput]$ vi wc.input
  hadoop yarn
  hadoop mapreduce
  atguigu
  atguigu
  # 执行程序
  [atguigu@hadoop101 hadoop-2.7.2]$ hadoop jar
   share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount wcinput wcoutput
  # 查看结果
  [atguigu@hadoop101 hadoop-2.7.2]$ cat wcoutput/part-r-00000
  ```

### 伪分布式模式

* 启动HDFS并运行MapReduce程序

  1. 配置集群

     ```shell
     #1. 配置：hadoop-env.sh
     # Linux系统中获取JDK的安装路径
     [atguigu@ hadoop101 ~]# echo $JAVA_HOME
     /opt/module/jdk1.8.0_144
     # 修改JAVA_HOME 路径
     export JAVA_HOME=/opt/module/jdk1.8.0_144
     #2. 配置：core-site.xml
     ```

     ```xml
     <!-- 指定HDFS中NameNode的地址 -->
     <property>
     <name>fs.defaultFS</name>
         <value>hdfs://hadoop101:9000</value>
     </property>
     <!-- 指定Hadoop运行时产生文件的存储目录 -->
     <property>
     	<name>hadoop.tmp.dir</name>
     	<value>/opt/module/hadoop-2.7.2/data/tmp</value>
     </property>
     ```

     ```xml
     <!--3. 配置：hdfs-site.xml-->
     <!-- 指定HDFS副本的数量 -->
     <property>
     	<name>dfs.replication</name>
     	<value>1</value>
     </property>
     ```

  2. 启动集群

     ```shell
     #1. 格式化NameNode（第一次启动时格式化，以后就不要总格式化）
     # 格式化NameNode，会产生新的集群id,导致NameNode和DataNode的集群id不一致，集群找不到已往数据。所以，格式NameNode时，一定要先删除data数据和log日志，然后再格式化NameNode
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs namenode -format
     #2. 启动NameNode
     [atguigu@hadoop101 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
     #3. 启动DataNode
     [atguigu@hadoop101 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start datanode
     ```

  3. 查看集群

     ```shell
     #1. 查看是否启动成功
     [atguigu@hadoop101 hadoop-2.7.2]$ jps
     13586 NameNode
     13668 DataNode
     13786 Jps
     #2. web端查看HDFS文件系统
     http://hadoop101:50070
     ```

  4. 查看产生的Log日志

     ```shell
     # 当前目录：/opt/module/hadoop-2.7.2/logs
     [atguigu@hadoop101 logs]# cat hadoop-atguigu-datanode-hadoop101.log
     ```

  5. 操作集群

     ```shell
     # 在HDFS文件系统上创建一个input文件夹
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -mkdir -p /user/atguigu/input
     # 将测试文件内容上传到文件系统上
     [atguigu@hadoop101 hadoop-2.7.2]$bin/hdfs dfs -put wcinput/wc.input
       /user/atguigu/input/
     # 查看上传的文件是否正确
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -ls  /user/atguigu/input/
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -cat  /user/atguigu/ input/wc.input
     # 运行MapReduce程序
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hadoop jar
     share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/atguigu/input/ /user/atguigu/output
     # 查看输出结果
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -cat /user/atguigu/output/*
     ```

     ![image-20191127151046847](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127151046847.png)

     ```shell
     # 将测试文件内容下载到本地
     [atguigu@hadoop101 hadoop-2.7.2]$ hdfs dfs -get /user/atguigu/output/part-r-00000 ./wcoutput/
     # 删除输出结果
     [atguigu@hadoop101 hadoop-2.7.2]$ hdfs dfs -rm -r /user/atguigu/output
     ```

* 启动YARN并运行MapReduce程序

  1. 配置集群

     ```shell
     # 配置yarn-env.sh，配置一下JAVA_HOME
     export JAVA_HOME=/opt/module/jdk1.8.0_144
     ```

     ```xml
     <!--配置yarn-site.xml-->
     <!-- Reducer获取数据的方式 -->
     <property>
      		<name>yarn.nodemanager.aux-services</name>
      		<value>mapreduce_shuffle</value>
     </property>
     
     <!-- 指定YARN的ResourceManager的地址 -->
     <property>
     <name>yarn.resourcemanager.hostname</name>
     <value>hadoop101</value>
     </property>
     ```

     ```shell
     # 配置 mapred-env.sh，配置一下JAVA_HOME
     export JAVA_HOME=/opt/module/jdk1.8.0_144
     # 配置： (对mapred-site.xml.template重新命名为) mapred-site.xml
     [atguigu@hadoop101 hadoop]$ mv mapred-site.xml.template mapred-site.xml
     [atguigu@hadoop101 hadoop]$ vi mapred-site.xml
     ```

     ```xml
     <!-- 指定MR运行在YARN上 -->
     <property>
     		<name>mapreduce.framework.name</name>
     		<value>yarn</value>
     </property>
     ```

  2. 启动集群

     ```shell
     # 启动前必须保证NameNode和DataNode已经启动
     # 启动ResourceManager
     [atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh start resourcemanager
     # 启动NodeManager
     [atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager
     ```

  3. 集群操作

     ```shell
     # YARN的浏览器页面查看
     http://hadoop101:8088/cluster
     # 删除文件系统上的output文件
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -rm -R /user/atguigu/output
     # 执行MapReduce程序
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hadoop jar
      share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/atguigu/input  /user/atguigu/output
     # 查看运行结果
     [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -cat /user/atguigu/output/*
     ```

* 配置历史服务器

  * 为了查看程序的历史运行情况，需要配置一下历史服务器

    ```shell
    [atguigu@hadoop101 hadoop]$ vi mapred-site.xml
    # 在该文件里面增加如下配置
    ```

    ```xml
    <!-- 历史服务器端地址 -->
    <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop101:10020</value>
    </property>
    <!-- 历史服务器web端地址 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop101:19888</value>
    </property>
    ```

    ```shell
    # 启动历史服务器
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh start historyserver
    # 查看历史服务器是否启动
    [atguigu@hadoop101 hadoop-2.7.2]$ jps
    # 查看JobHistory
    http://hadoop101:19888/jobhistory
    ```

* 配置日志的聚集

  * 日志聚集概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上

  * 日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试

  * 注意：开启日志聚集功能，需要重新启动NodeManager 、ResourceManager和HistoryManager

    ```shell
    # 1.配置yarn-site.xml
    [atguigu@hadoop101 hadoop]$ vi yarn-site.xml
    # 在该文件里面增加如下配置
    ```

    ```xml
    <!-- 日志聚集功能使能 -->
    <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
    </property>
    
    <!-- 日志保留时间设置7天 -->
    <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
    </property>
    ```

    ```shell
    # 2.关闭NodeManager 、ResourceManager和HistoryServer
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop resourcemanager
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh stop historyserver
    # 3.启动NodeManager 、ResourceManager和HistoryServer
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh start resourcemanager
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager
    [atguigu@hadoop101 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh start historyserver
    # 4.删除HDFS上已经存在的输出文件
    [atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -rm -R /user/atguigu/output
    # 5.执行WordCount程序
    [atguigu@hadoop101 hadoop-2.7.2]$ hadoop jar
     share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/atguigu/input /user/atguigu/output
    # 6.查看日志
    http://hadoop101:19888/jobhistory
    ```

* 配置文件说明

  * Hadoop配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值

    1. 默认配置文件

       ```shell
       # 要获取的默认文件							文件存放在Hadoop的jar包中的位置
       core-default.xml						hadoop-common-2.7.2.jar/ core-default.xml
       hdfs-default.xml						hadoop-hdfs-2.7.2.jar/ hdfs-default.xml
       yarn-default.xml						hadoop-yarn-common-2.7.2.jar/yarn-default.xml
       mapred-default.xml					hadoop-mapreduce-client-core-2.7.2.jar/ mapred-default.xml
       ```

    2. 自定义配置文件
       * core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml四个配置文件存放在$HADOOP_HOME/etc/hadoop这个路径上，用户可以根据项目需求重新进行修改配置

### ==完全分布式模式==

1. 准备3台客户机（关闭防火墙、静态ip、主机名称）、安装JDK、配置环境变量

   ```shell
   # 服务器间数据拷贝
   [atguigu@hadoop101 /]$ scp -r /opt/module  root@hadoop102:/opt/module
   # 注意：拷贝过来的/opt/module目录，别忘了在hadoop102、hadoop103、hadoop104上修改所有文件的，所有者和所有者组。sudo chown atguigu:atguigu -R /opt/module
   [atguigu@hadoop101 ~]$ sudo scp /etc/profile root@hadoop102:/etc/profile
   # 注意：拷贝过来的配置文件别忘了source一下/etc/profile
   ```

   * rsync 远程同步工具

     * rsync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点
     * rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去

     ```shell
     # 基本语法
     rsync    -av       $pdir/$fname              $user@hadoop$host:$pdir/$fname
     命令     选项参数    要拷贝的文件路径/名称         目的用户@主机:目的路径/名称
     # -a		归档拷贝
     # -v		显示复制过程
     [atguigu@hadoop101 opt]$ rsync -av /opt/software/ hadoop102:/opt/software
     ```

     * 需求：循环复制文件到所有节点的相同目录下

       ```shell
       # 说明：在/home/atguigu/bin这个目录下存放的脚本，atguigu用户可以在系统任何地方直接执行
       [atguigu@hadoop102 ~]$ mkdir bin
       [atguigu@hadoop102 ~]$ cd bin/
       [atguigu@hadoop102 bin]$ touch xsync
       [atguigu@hadoop102 bin]$ vi xsync
       #!/bin/bash
       #1 获取输入参数个数，如果没有参数，直接退出
       pcount=$#
       if ((pcount==0)); then
       echo no args;
       exit;
       fi
       
       #2 获取文件名称
       p1=$1
       fname=`basename $p1`
       echo fname=$fname
       
       #3 获取上级目录到绝对路径
       pdir=`cd -P $(dirname $p1); pwd`
       echo pdir=$pdir
       
       #4 获取当前用户名称
       user=`whoami`
       
       #5 循环
       for((host=103; host<105; host++)); do
               echo ------------------- hadoop$host --------------
               rsync -av $pdir/$fname $user@hadoop$host:$pdir
       done
       
       # 修改脚本 xsync 具有执行权限
       [atguigu@hadoop102 bin]$ chmod 777 xsync
       # 调用脚本形式：xsync 文件名称
       [atguigu@hadoop102 bin]$ xsync /home/atguigu/bin
       # 注意：如果将xsync放到/home/atguigu/bin目录下仍然不能实现全局使用，可以将xsync移动到/usr/local/bin目录下
       ```

2. 安装Hadoop

3. 配置环境变量

4. 配置集群

   * 集群部署规划

     ![image-20191127155138713](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127155138713.png)

     1. 核心配置文件

        ```shell
        [atguigu@hadoop102 hadoop]$ vi core-site.xml
        ```

        ```xml
        <!-- 指定HDFS中NameNode的地址 -->
        <property>
        		<name>fs.defaultFS</name>
              <value>hdfs://hadoop102:9000</value>
        </property>
        
        <!-- 指定Hadoop运行时产生文件的存储目录 -->
        <property>
        		<name>hadoop.tmp.dir</name>
        		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
        </property>
        ```

     2. HDFS配置文件

        ```shell
        [atguigu@hadoop102 hadoop]$ vi hadoop-env.sh
        export JAVA_HOME=/opt/module/jdk1.8.0_144
        [atguigu@hadoop102 hadoop]$ vi hdfs-site.xml
        ```

        ```xml
        <property>
        		<name>dfs.replication</name>
        		<value>3</value>
        </property>
        
        <!-- 指定Hadoop辅助名称节点主机配置 -->
        <property>
              <name>dfs.namenode.secondary.http-address</name>
              <value>hadoop104:50090</value>
        </property>
        ```

     3. YARN配置文件

        ```shell
        [atguigu@hadoop102 hadoop]$ vi yarn-env.sh
        export JAVA_HOME=/opt/module/jdk1.8.0_144
        [atguigu@hadoop102 hadoop]$ vi yarn-site.xml
        ```

        ```xml
        <!-- Reducer获取数据的方式 -->
        <property>
        		<name>yarn.nodemanager.aux-services</name>
        		<value>mapreduce_shuffle</value>
        </property>
        
        <!-- 指定YARN的ResourceManager的地址 -->
        <property>
        		<name>yarn.resourcemanager.hostname</name>
        		<value>hadoop103</value>
        </property>
        ```

     4. MapReduce配置文件

        ```shell
        [atguigu@hadoop102 hadoop]$ vi mapred-env.sh
        export JAVA_HOME=/opt/module/jdk1.8.0_144
        [atguigu@hadoop102 hadoop]$ cp mapred-site.xml.template mapred-site.xml
        [atguigu@hadoop102 hadoop]$ vi mapred-site.xml
        ```

        ```xml
        <!-- 指定MR运行在Yarn上 -->
        <property>
        		<name>mapreduce.framework.name</name>
        		<value>yarn</value>
        </property>
        ```

   * 在集群上分发配置好的Hadoop配置文件

     ```shell
     [atguigu@hadoop102 hadoop]$ xsync /opt/module/hadoop-2.7.2/
     ```

   * 查看文件分发情况

     ```shell
     [atguigu@hadoop103 hadoop]$ cat /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml
     ```

5. 单点启动

   ```shell
   # 如果集群是第一次启动，需要格式化NameNode
   [atguigu@hadoop102 hadoop-2.7.2]$ hdfs namenode -format
   # 在hadoop102上启动NameNode
   [atguigu@hadoop102 hadoop-2.7.2]$ hadoop-daemon.sh start namenode
   [atguigu@hadoop102 hadoop-2.7.2]$ jps
   3461 NameNode
   # 在hadoop102、hadoop103以及hadoop104上分别启动DataNode
   [atguigu@hadoop102 hadoop-2.7.2]$ hadoop-daemon.sh start datanode
   [atguigu@hadoop102 hadoop-2.7.2]$ jps
   3461 NameNode
   3608 Jps
   3561 DataNode
   [atguigu@hadoop103 hadoop-2.7.2]$ hadoop-daemon.sh start datanode
   [atguigu@hadoop103 hadoop-2.7.2]$ jps
   3190 DataNode
   3279 Jps
   [atguigu@hadoop104 hadoop-2.7.2]$ hadoop-daemon.sh start datanode
   [atguigu@hadoop104 hadoop-2.7.2]$ jps
   3237 Jps
   3163 DataNode
   ```

6. 配置ssh无密登录

   ![image-20191127155910632](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127155910632.png)

   ```shell
   # 生成公钥和私钥
   [atguigu@hadoop102 .ssh]$ ssh-keygen -t rsa
   # 然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）
   # 将公钥拷贝到要免密登录的目标机器上
   [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop102
   [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop103
   [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop104
   # 注意：
   #还需要在hadoop102上采用root账号，配置一下无密登录到hadoop102、hadoop103、hadoop104；
   #还需要在hadoop103上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。
   ```

7. 群起并测试集群

   1. 配置slaves

      ```shell
      /opt/module/hadoop-2.7.2/etc/hadoop/slaves
      [atguigu@hadoop102 hadoop]$ vi slaves
      # 增加，注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行
      hadoop102
      hadoop103
      hadoop104
      # 同步所有节点配置文件
      [atguigu@hadoop102 hadoop]$ xsync slaves
      ```

   2. 启动集群

      ```shell
      # 如果集群是第一次启动，需要格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除data和log数据）
      [atguigu@hadoop102 hadoop-2.7.2]$ bin/hdfs namenode -format
      # 启动HDFS
      [atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
      [atguigu@hadoop102 hadoop-2.7.2]$ jps
      4166 NameNode
      4482 Jps
      4263 DataNode
      [atguigu@hadoop103 hadoop-2.7.2]$ jps
      3218 DataNode
      3288 Jps
      [atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
      [atguigu@hadoop102 hadoop-2.7.2]$ jps
      4166 NameNode
      4482 Jps
      4263 DataNode
      [atguigu@hadoop103 hadoop-2.7.2]$ jps
      3218 DataNode
      3288 Jps
      ```

   3. 启动YARN

      ```shell
      [atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
      # 注意：NameNode和ResourceManger如果不是同一台机器，不能在NameNode上启动 YARN，应该在ResouceManager所在的机器上启动YARN
      ```

   4. Web端查看SecondaryNameNode

      ```shell
      # 浏览器中输入：http://hadoop104:50090/status.html
      # 查看SecondaryNameNode信息
      ```

* 集群基本测试

  1. 上传文件到集群

     ```shell
     # 上传小文件
     [atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfs -mkdir -p /user/atguigu/input
     [atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfs -put wcinput/wc.input /user/atguigu/input
     # 上传大文件
     [atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -put
      /opt/software/hadoop-2.7.2.tar.gz  /user/atguigu/input
     ```

  2. 上传文件后查看文件存放在什么位置

     ```shell
     # 查看HDFS文件存储路径
     [atguigu@hadoop102 subdir0]$ pwd
     /opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/BP-938951106-192.168.10.107-1495462844069/current/finalized/subdir0/subdir0
     # 查看HDFS在磁盘存储文件内容
     [atguigu@hadoop102 subdir0]$ cat blk_1073741825
     hadoop yarn
     hadoop mapreduce 
     atguigu
     atguigu
     ```

  3. 拼接

     ```shell
     [atguigu@hadoop102 subdir0]$ cat blk_1073741836>>tmp.file
     [atguigu@hadoop102 subdir0]$ cat blk_1073741837>>tmp.file
     [atguigu@hadoop102 subdir0]$ tar -zxvf tmp.file
     ```

  4. 下载

     ```shell
     [atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -get
      /user/atguigu/input/hadoop-2.7.2.tar.gz ./
     ```

* 集群启动/停止方式总结

  ```shell
  # 各个服务组件逐一启动/停止
  # 1. 分别启动/停止HDFS组件
  hadoop-daemon.sh  start / stop  namenode / datanode / secondarynamenode
  # 2. 启动/停止YARN
  yarn-daemon.sh  start / stop  resourcemanager / nodemanager
  
  # 各个模块分开启动/停止（配置ssh是前提）常用
  #1. 整体启动/停止HDFS
  start-dfs.sh   /  stop-dfs.sh
  #2. 整体启动/停止YARN
  start-yarn.sh  /  stop-yarn.sh
  ```

* 集群时间同步

## Hadoop编译源码

* 配置CentOS能连接外网。Linux虚拟机ping [www.baidu.com](http://www.baidu.com) 是畅通的

  * 注意：采用root角色编译，减少文件夹权限出现问题

* jar包准备(hadoop源码、JDK8、maven、ant 、protobuf)

  （1）hadoop-2.7.2-src.tar.gz

  （2）jdk-8u144-linux-x64.tar.gz

  （3）apache-ant-1.9.9-bin.tar.gz（build工具，打包用的）

  （4）apache-maven-3.0.5-bin.tar.gz

  （5）protobuf-2.5.0.tar.gz（序列化的框架）

* Jar包安装，所有操作必须在root用户下完成

  * 配置jdk、maven、ant

  * 安装  glibc-headers 和  g++  命令如下

    ```shell
    [root@hadoop101 apache-ant-1.9.9]# yum install glibc-headers
    [root@hadoop101 apache-ant-1.9.9]# yum install gcc-c++
    ```

  * 安装make和cmake

    ```shell
    [root@hadoop101 apache-ant-1.9.9]# yum install make
    [root@hadoop101 apache-ant-1.9.9]# yum install cmake
    ```

  * 解压protobuf ，进入到解压后protobuf主目录，/opt/module/protobuf-2.5.0，然后相继执行命令

    ```shell
    [root@hadoop101 software]# tar -zxvf protobuf-2.5.0.tar.gz -C /opt/module/
    [root@hadoop101 opt]# cd /opt/module/protobuf-2.5.0/
    [root@hadoop101 protobuf-2.5.0]#./configure 
    [root@hadoop101 protobuf-2.5.0]# make 
    [root@hadoop101 protobuf-2.5.0]# make check 
    [root@hadoop101 protobuf-2.5.0]# make install 
    [root@hadoop101 protobuf-2.5.0]# ldconfig 
    [root@hadoop101 hadoop-dist]# vi /etc/profile
    #LD_LIBRARY_PATH
    export LD_LIBRARY_PATH=/opt/module/protobuf-2.5.0
    export PATH=$PATH:$LD_LIBRARY_PATH
    
    [root@hadoop101 software]#source /etc/profile
    # 验证命令：protoc --version
    ```

  * 安装openssl库

    ```shell
    [root@hadoop101 software]#yum install openssl-devel
    ```

  * 安装 ncurses-devel库

    ```shell
    [root@hadoop101 software]#yum install ncurses-devel
    ```

  * 到此，编译工具安装基本完成

* 编译源码

  ```shell
  #1. 解压源码到/opt/目录
  [root@hadoop101 software]# tar -zxvf hadoop-2.7.2-src.tar.gz -C /opt/
  #2. 进入到hadoop源码主目录
  [root@hadoop101 hadoop-2.7.2-src]# pwd
  /opt/hadoop-2.7.2-src
  #3. 通过maven执行编译命令
  [root@hadoop101 hadoop-2.7.2-src]#mvn package -Pdist,native -DskipTests -Dtar
  # 等待时间30分钟左右，最终成功是全部SUCCESS
  #4. 成功的64位hadoop包在/opt/hadoop-2.7.2-src/hadoop-dist/target下
  [root@hadoop101 target]# pwd
  /opt/hadoop-2.7.2-src/hadoop-dist/target
  ```

* 编译源码过程中常见的问题及解决方案





