* flume、zookeeper、kafka、HBASE、Spark Streaming

* 整合Flume、kafka、Spark Streaming打造通用的流处理平台基础
* Spark Streaming项目实战
* 数据处理结果可视化
* 拓展
* 环境参数
  * Linux：CentOS（6.4）
  * JDK：1.8
  * Hadoop版本：CDH（5.7）
  * Scala：2.11.8
  * Spark版本：2.2.0

```shell
vim /etc/hosts
	192.168.0.226 hadoop000
	192.168.0.226 localhost

# /home/hadoop目录下创建如下目录
# app 存放所有的软件安装包
# data 存放测试数据
# lib 存放开发的jar
# software 存放软件安装包
# source 存放框架源码

# 2.4.6
https://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-2.4.6/spark-2.4.6-bin-hadoop2.7.tgz
# scala 2.13.1
# 所有hadoop生态的软件下载地址
http://archive.cloudera.com/cdh5/cdh/5/
```

![image-20200607154954111](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607154954111.png)

![image-20200607155020421](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607155020421.png)

## 功能实现

* 今天到现在为止实战课程的访问量
* 今天到现在为止从搜索引擎引流过来的实战课程的访问量

## 项目流程

* 需求分析、产生、采集、清洗、统计分析、入库、可视化

  ![image-20200604200115171](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200604200115171.png)

## 可视化效果

* 使用SpringBoot整合Echarts实现
* 阿里云DataV数据可视化框架实现

## 初识实时流处理

* ==[Spark git 上的example案例一定要看](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/HdfsWordCount.scala)==

![image-20200607154337692](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607154337692.png)

### 业务状况分析

* 需求：统计主站每个（指定）课程访问的客户端、地域信息分布
  * 地域：ip转换   Spark SQL项目实战
  * 客户端：useragent获取   Hadoop基础课程
  * ==> 如上两个操作：采用离线（Spark/MapReduce）的方式进行统计
* 实现步骤
  * 课程编号、ip信息、useragent
  * 进行相应的统计分析操作：MapReduce/Spark
* 项目架构
  * 日志收集：Flume
  * 离线分析：MapReduce/Spark
  * 统计结果图形化展示
* 问题
  * 小时级别
  * 10分钟
  * 5分钟
  * 1分钟
  * 秒级别
* 如何解决？ =》 实时流处理框架

### 实时流处理产生背景

* 时效性高
* 数据量大

### 实时流处理概述

* 实时计算，响应时间短，延时低
* 流式计算
* 实时流计算

### 离线计算与实时计算对比

* 数据来源
  * 离线：HDFS历史数据，数据量比较大
  * 实时：消息队列（Kafka），实时新增/修改记录过来的某一笔数据
* 处理过程
  * 离线：MapReduce：map + reduce
  * 实时：Spark（DStream/SS）
* 处理速度
  * 离线：慢
  * 实时：快速
* 进程
  * 离线：启动 + 销毁
  * 实时：7 * 24

### 实时流处理框架对比

* Apache Storm
  * 实时
* Apache Spark Streaming
  * 微批

### 实时流处理架构与技术选型

* `web/app` -> `WebServer(/var/log/access.log)` -> `Flume` -> `Kafka` -> `Spark/Storm` -> `RDBMS/NoSQL` -> `可视化展示` 

![image-20200607160739323](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607160739323.png)

### 实时流处理在企业中的应用

* 电信行业
* 电商行业

## 日志收集框架Flume

### 业务现状分析

![image-20200607161914335](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607161914335.png)

* WebServer/ApplicationServer分散在各个机器上
* 想大数据平台Hadoop进行统计分析
* 日志如何收集到Hadoop平台上
* 解决方案及存在的问题
* 容错、负载均衡、高延时、压缩

### Flume概述

* 是一个分布式、高可靠、高可用的服务，用于分布式的海量日志的高效收集、聚合、移动系统

* 把日志从A搬运到B

  ![image-20200607162401693](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607162401693.png)

* 设计目标
  
  * 可靠性、扩展性、管理性

### Flume架构及核心组件

![image-20200607163055628](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607163055628.png)

* Source：收集
* Channel：聚集
* Sink：输出

![image-20200607163409448](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607163409448.png)

![image-20200607163905991](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607163905991.png)

### Flume环境部署

```shell
# jdk1.8
# # 下载、解压、配置环境变量
tar -zxvf flume-ng-1.6.0-cdh5.7.0.tar.gz -C /home/app
vi /etc/profile
	export FLUME_HOME=/home/woody/app/apache-flume-1.9.0-bin
	export PATH=$PATH:$FLUME_HOME/bin
source /etc/profile
```

* 拷贝一份`flume-env.sh`
  * ` cp flume-env.sh.template flume-env.sh`
  * `export JAVA_HOME=xxxx`

* 检测是否安装成功

  * `./bin/flume-ng version`

### Flume实战

### 需求：从指定网络端口采集数据输出到控制台

```shell
# 使用Flume的关键就是写配置文件
1. 配置Source
2. 配置Channel
3. 配置Sink
4. 把以上三个组件串起来

a1: agent名称
r1: source名称
k1: sink的名称
c1: channel的名称

# conf/example.conf
# Name the component on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
# 服务器的hostname
a1.sources.r1.bind = hadoop000 
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

* 启动agent

```shell
flume-ng agent \
-n $agent_name \
-c conf \
-f conf/flume-conf.properties.conf

flume-ng agent \
--name a1 \
--conf $FLUME_HOME/conf
--conf-file $FLUME_HOME/conf/example_1.conf
-Dflume.root.logger=INFO,console

flume-ng agent -n a1 -c $FLUME_HOME/conf -f $FLUME_HOME/conf/example_1.conf -Dflume.root.logger=INFO,console
```

```shell
# 服务器上
telnet master 44444
```

![image-20200607173400674](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607173400674.png)

![image-20200607173440333](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607173440333.png)

### 需求：监控一个文件实时采集新增的数据输出到控制台

[官网](http://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html#exec-source)

```shell
# Agent选型：exec source + memory channel + logger sink
# conf/example_1.conf
# Name the component on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
# 先创建日志
a1.sources.r1.command = tail -F /home/hadoop/data/data.log
a1.sources.r1.shell = /bin/sh -c

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

```shell
# 启动
flume-ng agent -n a1 -c $FLUME_HOME/conf -f $FLUME_HOME/conf/example_2.conf -Dflume.root.logger=INFO,console

[root@master bussines_flume]# echo hello >> data.log
[root@master bussines_flume]# echo world >> data.log
[root@master bussines_flume]# echo welcome >> data.log
[root@master bussines_flume]# echo flume >> data.log
```

![image-20200607175115973](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607175115973.png)

### 需求：将A服务器上的日志实时采集到B服务器

 ![image-20200607182635904](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607182635904.png)

![image-20200607180638744](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607180638744.png)

```shell
# 技术选型
exec source + memory channel + avro sink
avro source + memory channel + logger sink

# conf/exec-memory-avro.conf
exec-memory-avro.sources = exec-source
exec-memory-avro.sinks = avro-sink
exec-memory-avro.channels = memory-channel

exec-memory-avro.sources.exec-source.type = exec
exec-memory-avro.sources.exec-source.command = tail -F /home/hadoop/data/data.log
exec-memory-avro.sources.exec-source.shell = /bin/sh -c

exec-memory-avro.sinks.avro-sink.type = avro
exec-memory-avro.sinks.avro-sink.hostname = hadoop000
exec-memory-avro.sinks.avro-sink.port = 44444

exec-memory-avro.channels.memory-channel.type = memory

exec-memory-avro.sources.exec-source.channels = memory-channel
exec-memory-avro.sinks.avro-sink.channel = memory-channel


# conf/avro-memory-logger.conf
avro-memory-logger.sources = avro-source
avro-memory-logger.sinks = logger-sink
avro-memory-logger.channels = memory-channel

avro-memory-logger.sources.avro-source.type = avro
avro-memory-logger.sources.avro-source.bind = hadoop000
avro-memory-logger.sources.avro-source.port = 44444

avro-memory-logger.sinks.logger-sink.type = logger
avro-memory-logger.channels.memory-channel.type = memory

avro-memory-logger.sources.avro-source.channels = memory-channel
avro-memory-logger.sinks.logger-sink.channel = memory-channel
```

```shell
# 先启动 avro-memory-logger
flume-ng agent -n avro-memory-logger -c $FLUME_HOME/conf -f $FLUME_HOME/conf/avro-memory-logger.conf -Dflume.root.logger=INFO,console

# 再启动 exec-memory-avro
flume-ng agent -n exec-memory-avro -c $FLUME_HOME/conf -f $FLUME_HOME/conf/exec-memory-avro.conf -Dflume.root.logger=INFO,console
```

```shell
[root@master bussines_flume]# echo hello spark >> data.log
[root@master bussines_flume]# echo hello hadoop >> data.log
[root@master bussines_flume]# echo hello flume >> data.log
```

## 消息队列Kafka

### kafka概述

### Kafka架构及核心概念

![image-20200607192854578](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200607192854578.png)

* producer：生产者
* consumer：消费者
* broker：一个kafka
* topic：主题

### Kafka部署及使用

* [zookeeper-3.4.5+cdh5.7.6](https://docs.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh_package_tarball_57.html)

```shell
# zookeeper
解压、配置环境变量
export ZK_HOME=/data/hadoop/app/zookeeper-3.4.5-cdh5.7.6
export PATH=$PATH:$ZK_HOME/bin

#conf
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
dataDir=/data/hadoop/app/tmp/zk

# 启动
./bin/zkServer.sh start
jps
./bin/zkCli.sh
ls /
```

* [kafka_2.12-2.5.0.tgz](https://mirrors.bfsu.edu.cn/apache/kafka/2.5.0/kafka_2.12-2.5.0.tgz)
* 单节点单Broker部署及使用

```shell
# kafka
解压、配置环境变量
export KAFKA_HOME=/data/hadoop/app/kafka-2.5.0-src
export PATH=$PATH:$KAFKA_HOME/bin

# vim /config/server.properties
broker.id=0
listeners=PLAINTEXT://:9092
host.name=hadoop000
log.dirs=/data/hadoop/app/tmp/kafka
zookeeper.connect=hadoop000

# 启动kafka
./bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties
jps
jps -m
# 创建topic
kafka-topics.sh --create --zookeeper hadoop000:2181 --replication-factor 1 --partitions 1 --topic hello_topic
# 查看所有topic
kafka-topics.sh --list --zookeeper hadoop000:2181
# 发送消息
kafka-console-producer.sh --broker-list hadoop000:9092 --topic hello_topic
# 消费消息
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello_topic --from-beginning
# --from-beginning 从头开始消费
# 查看所有topic的详细信息
kafka-topics.sh --describe --zookeeper hadoop:2181
# 查看指定topic的详细信息
kafka-topics.sh --describe --zookeeper hadoop:2181 --topic hello_topic
```

* 单节点多Broker部署及使用

  ```shell
  cp config/server.properties config/server-1.properties
  cp config/server.properties config/server-2.properties
  cp config/server.properties config/server-3.properties
  
  config/server-1.properties:
      broker.id=1
      listeners=PLAINTEXT://:9093
      log.dirs=/tmp/kafka-logs-1
   
  config/server-2.properties:
      broker.id=2
      listeners=PLAINTEXT://:9094
      log.dirs=/tmp/kafka-logs-2
      
  config/server-3.properties:
      broker.id=3
      listeners=PLAINTEXT://:9095
      log.dirs=/tmp/kafka-logs-3
      
  # 启动
  bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server-1.properties &
  bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server-2.properties &
  bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server-3.properties &
  
  jps -m
  
  # 创建topic
  kafka-topics.sh --create --zookeeper master:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
  # 查看所有topic
  kafka-topics.sh --describe --zookeeper master:2181
  # 发送消息
  kafka-console-producer.sh --broker-list master:9093,master:9094,master:9095 --topic my-replicated-topic
  # 消费消息
  # kafka-console-consumer.sh --zookeeper master:2181 --topic my-replicated-topic
  kafka-console-consumer.sh --bootstrap-server master:9093 --topic my-replicated-topic
  ```

* 多节点多Broker部署及使用

### Kafka容错性测试

```shell
# 查看所有topic
kafka-topics.sh --describe --zookeeper master:2181
Topic: my-replicated-topic	PartitionCount: 1	ReplicationFactor: 3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
# leader 为3
jps -m
[root@master bin]# jps -m
29489 Kafka /data/hadoop/app/kafka_2.12-2.5.0/config/server-1.properties
30290 Kafka /data/hadoop/app/kafka_2.12-2.5.0/config/server-3.properties
21476 QuorumPeerMain /data/hadoop/app/zookeeper-3.4.5-cdh5.7.6/bin/../conf/zoo.cfg
29892 Kafka /data/hadoop/app/kafka_2.12-2.5.0/config/server-2.properties
32742 ConsoleProducer --broker-list master:9093,master:9094,master:9095 --topic my-replicated-topic
2651 ConsoleConsumer --bootstrap-server master:9093 --topic my-replicated-topic --from-beginning
4109 Jps -m
# 把2干掉
kill -9 29892
jps -m
# 发送和消费消息
# 查看详细信息
kafka-topics.sh --describe --zookeeper master:2181
Topic: my-replicated-topic	PartitionCount: 1	ReplicationFactor: 3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 3	Replicas: 3,1,2	Isr: 3,1
# 把3（master）干掉	
kill -9 30290
# 查看详细信息
kafka-topics.sh --describe --zookeeper master:2181
Topic: my-replicated-topic	PartitionCount: 1	ReplicationFactor: 3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 1	Replicas: 3,1,2	Isr: 1
# 只要有一个节点正常，就不影响使用	
```

### Kafka API编程

```shell
https://kafka.apache.org/25/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html

# java.net.UnknownHostException: master
# 本地配置
vim /etc/hosts
192.168.0.221 master localhost
```

```java
public class KafkaTest {

    public static final String TOPIC = "hello_topic";
    public static final String BROKER_LIST = "192.168.0.221:9092";
    public static final String GROUP_ID = "test";

    public static void main(String[] args) {
        kafkaProducer();
        kafkaConsumer();
    }

    public static void kafkaProducer() {
        Properties props = new Properties();
        // Kafka服务端的主机名和端口号
        props.put("bootstrap.servers", BROKER_LIST);

        // 指定consumer group
        props.put("group.id", GROUP_ID);

        // 等待所有副本节点的应答
        props.put("acks", "all");
        // 消息发送最大尝试次数
        props.put("retries", 3);
        // 一批消息处理大小
        props.put("batch.size", 16384);
        // 请求延时
        props.put("linger.ms", 1);
        // 发送缓存区内存大小
        props.put("buffer.memory", 33554432);
        // key序列化
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // value序列化
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        Producer<String, String> producer = new KafkaProducer<>(props);
        for (int i = 0; i < 10; i++) {
            producer.send(new ProducerRecord<String, String>(TOPIC, Integer.toString(i), "_hello world-" + i));
        }
        producer.close();
    }

    public static void kafkaConsumer() {
        Properties props = new Properties();
        // 定义kakfa 服务的地址，不需要将所有broker指定上
        props.put("bootstrap.servers", KafkaProperties.BROKER_LIST);
        // 指定consumer group
        props.put("group.id", KafkaProperties.GROUP_ID);
        // 是否自动确认offset
        props.put("enable.auto.commit", "true");
        // 自动确认offset的时间间隔
        props.put("auto.commit.interval.ms", "1000");
        // key的序列化类
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        // value的序列化类
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        // 定义consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        // 消费者订阅的topic, 可同时订阅多个
        consumer.subscribe(Arrays.asList(TOPIC));
        while (true) {
            // 读取数据，读取超时时间为100ms
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
            }
        }
    }
}
```

### Kafka实战

* 整合Flume和Kafka完成实时数据采集

  ![image-20200610204225219](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200610204225219.png)

```properties
# avro-memory-kafka.conf
avro-memory-kafka.sources = avro-source
avro-memory-kafka.sinks = kafka-sink
avro-memory-kafka.channels = memory-channel

avro-memory-kafka.sources.avro-source.type = avro
avro-memory-kafka.sources.avro-source.bind = hadoop000
avro-memory-kafka.sources.avro-source.port = 44444
# 看官网 kafka-sink这节
avro-memory-kafka.sinks.kafka-sink.type = org.apache.flume.sink.kafka.KafkaSink
avro-memory-kafka.sinks.kafka-sink.brokerList = hadoop000:9092
avro-memory-kafka.sinks.kafka-sink.topic = hello_topic
avro-memory-kafka.sinks.kafka-sink.batchSize = 5
avro-memory-kafka.sinks.kafka-sink.requiredAcks = 1

avro-memory-kafka.channels.memory-channel.type = memory

avro-memory-kafka.sources.avro-source.channels = memory-channel
avro-memory-kafka.sinks.kafka-sink.channel = memory-channel
```

```shell
# 先启动flume
flume-ng agent -n avro-memory-kafka -c $FLUME_HOME/conf -f $FLUME_HOME/conf/avro-memory-kafka.conf -Dflume.root.logger=INFO,console

flume-ng agent --name exec-memory-avro --conf $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/exec-memory-avro.conf -Dflume.root.logger=INFO,console

# 启动kafka
kafka-server-start.sh $KAFKA_HOME/config/server.properties
# 启动Kafka消费者
kafka-console-consumer.sh --bootstrap-server master:9092 --topic hello_topic

# 造数据
# /data/hadoop/app/log/bussines_flume/data.log
echo hellospark1 >> data.log
echo hellospark2 >> data.log
echo hellospark3 >> data.log
```

## 实战环境搭建

* jdk、zookeeper

* scala安装

  ```shell
  # 下载、解压、配置环境变量、检查是否安装成功
  vim /etc/profile
  export SCALA_HOME=/data/hadoop/app/scala-2.13.1
  export PATH=$PATH:$SCALA_HOME/bin
  source /etc/profile
  scala
  ```

* maven安装

  ```shell
  # 下载、解压、配置环境变量、检查是否安装成功
  vim /etc/profile
  export MAVEN_HOME=/data/hadoop/app/apache-maven-3.6.3
  export PATH=$PATH:$MAVEN_HOME/bin
  source /etc/profile
  mvn -version
  
  /conf/setting.xml
  <localRepository>/data/hadoop/app/mvn_repo</localRepository>
  <mirror>  
  	<id>alimaven</id>  
  	<name>aliyun maven</name>  
  	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
  	<mirrorOf>central</mirrorOf>          
  </mirror>
  ```

* hadoop环境搭建

  ```shell
  # 下载、解压、配置环境变量、检查是否安装成功、配置文件修改
  # http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0.tar.gz
  # http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0/
  # http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.1/hadoop-project-dist/hadoop-common/SingleCluster.html
  
  hadoop-2.6.5版本
  
  ssh-keygen -t rsa
  cd ~/.ssh
  cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
  
  # 解压、cd etc/hadoop
  # hadoop-env.sh
  	export JAVA_HOME=/soft/jdk/jdk1.8.0_60
  # core-site.xml
  	<configuration>
    	<property>
    		<name>fs.defaultFS</name>
   			<value>hdfs://master:8020</value>
    		<name>hadoop.tmp.dir</name>
    		<value>/data/hadoop/app/tmp</value>
      </property>
      <property>
        <name>fs.default.name</name>
        <value>hdfs://master:8020</value>
      </property>
  </configuration>
  # hdfs-site.xml
  	<property>
  		<name>dfs.replication</name>
  		<value>1</value>
  	</property>
  # slaves
  master
  
  bin/hdfs namenode -format
  
  export HADOOP_HOME=/data/hadoop/app/hadoop-2.6.5
  export PATH=$PATH:$HADOOP_HOME/bin
  
  sbin/start-dfs.sh
  
  [root@master hadoop-2.6.5]# jps
  14240 DataNode
  14400 SecondaryNameNode
  24474 Jps
  14110 NameNode
  
  # 访问
  http://192.168.0.221:50070/
  
  # 搭建yarn
  etc/hadoop
  cp mapred-site.xml.template mapred-site.xml
  vim mapred-site.xml
  	<property>
  		<name>mapreduce.framework.name</name>
  		<value>yarn</value>
  	</property>
  	
  vim yarn-site.xml
  	<property>
  		<name>yarn.nodemanager.aux-services</name>
  		<value>mapreduce_shuffle</value>
  	</property>	
  	
  sbin/start-yarn.sh
  jps
  [root@master sbin]# jps
  14240 DataNode
  14400 SecondaryNameNode
  26248 Jps
  26073 NodeManager
  25933 ResourceManager
  14110 NameNode
  # 访问
  http://192.168.0.221:8088/
  
  # 测试 hadoop
  /sbin
  hadoop fs -ls /
  hadoop fs -mkdir /data
  ls
  hadoop fs -ls /data
  hadoop fs -put xxx.sh /data/
  hadoop fs -ls /data
  hadoop fs -text xxx.sh
  # 测试
  cd hadoop-2.6.5/share/hadoop/mapreduce
  hadoop jar hadoop-mapreduce-examples-2.6xxxx.jar pi 2 3
  # 刷新 http://192.168.0.221:8088/  
  # FINISHED 
  ```
  
* HBash搭建

  ```shell
  http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.7.0.tar.gz
  # 下载、解压、环境变量
  export HBASE_HOME=/data/hadoop/app/hbase-1.2.0-cdh5.7.0
  export PATH=$PATH:$HBASE_HOME/bin
  
  conf/
  vim hbase-env.sh
  export JAVA_HOME=/soft/jdk/jdk1.8.0_60
  export HBASE_MANAGES_ZK=false
  
  vim hbase-site.xml
  	<property>
  		<name>hbase.rootdir</name>
  		<value>hdfs://master:8020/hbase</value>
  	</property>
  	<property>
  		<name>hbase.cluster.distributed</name>
  		<value>true</value>
  	</property>
  	<property>
  		<name>hbase.zookeeper.quorum</name>
  		<value>master:2181</value>
  	</property>
  	
  vim reginservers
  master
  
  cd $ZOOKEEPER_HOME/bin
  ./zkServer.shs start
  
  # 启动
  /bin/start-hbase.sh
  jps
  # 访问
  # http://192.168.0.221:60010
  http://192.168.0.221:16010
  jps
  
  /bin
  ./hbase shell
  version
  status
  create 'member','info','address'
  list
  describe 'member'
  ```

* spark安装

  ```shell
  # 下载源码（[编译]）、解压、配置环境变量、检测是否安装成功
  # 官网，download，package type:Source Code
  wget https://mirrors.bfsu.edu.cn/apache/spark/spark-2.4.6/spark-2.4.6.tgz
  
  export SPARK_HOME=/data/hadoop/app/spark-2.4.6-bin-hadoop2.6
  export PATH=$PATH:$SPARK_HOME/bin
  
  bin/
  ./spark-shell --master local[2] 
  ```

* 实战环境搭建
  * 使用IDEA整合Maven搭建Spark Streaming开发环境
  *  pom.xml中添加对应的依赖

## Spark Streaming入门

### 概述

![image-20200613173906437](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613173906437.png)

![image-20200613174305019](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174305019.png)

* 将不同的数据源的数据经过Spark Streaming处理之后将结果输出到外部文件系统
* 特点
  * 低延时
  * 能从错误中高效的恢复：fault-tolerant
  * 能够运行在成百上千的节点
  * 能够将批处理、机器学习、图计算等子框架和Spark Streaming综合起来使用
* 安装Spark即可使用Spark Streaming
* one stack to rule them all 一站式

![image-20200613174355735](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174355735.png)

### 应用场景

![image-20200613174527514](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174527514.png)



### 集成Spark生态系统的使用

![image-20200613174628443](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174628443.png)

![image-20200613174705958](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174705958.png)

![image-20200613174756402](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174756402.png)

![image-20200613174834046](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174834046.png)

### 发展史

![image-20200613174923656](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613174923656.png)

### 从词频统计功能着手入门

* spark-submit执行（生产）

  ```shell
  #https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/NetworkWordCount.scala
  # 执行 (未安装，则先安装yum install -y nc)
  nc -lk 9999
  # bin/
  ./spark-submit --master local[2] --class org.apache.spark.examples.streaming.NetworkWordCount --name NetworkWordCount /data/hadoop/app/spark-2.4.6-bin-hadoop2.6/examples/jars/spark-examples_2.11-2.4.6.jar master 9999
  # 输入
  [root@master bin]# nc -lk 9999
  a a a a a a d d b
  ```

* spark-shell执行（测试）

  ```shell
  # bin/
  ./spark-shell --master local[2]
  
  # 执行
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  val ssc = new StreamingContext(sc, Seconds(1))
  val lines = ssc.socketTextStream("master", 9999)
  val words = lines.flatMap(_.split(" "))
  val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
  wordCounts.print()
  ssc.start()
  ssc.awaitTermination()
  
  # 输入
  [root@master bin]# nc -lk 9999
  a a a a a a d d b
  ```

### 工作原理

* 粗粒度
  * Spark Streaming接收到实时数据流，把数据按照指定时间段切成一片片小的数据块，然后把小的数据块传给Spark Engine处理
* 细粒度
  * ![image-20200613181857724](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613181857724.png)
  * Spark应用程序运行在java端，有Streaming Context、SparkContext
  * 启动Receiver，接收Inout Stream 切分成blocks存入内存、副本
  * 把block的信息告诉StreamingContext，再启动jobs

## Spark Streaming进阶

###核心概念

* StreamingContext

  ```scala
  import org.apache.spark.streaming._
  
  val sc = ...                // existing SparkContext
  val ssc = new StreamingContext(sc, Seconds(1))
  ```

  ```scala
  def this(sparkContext: SparkContext, batchDuration: Duration) = {
    this(sparkContext, null, batchDuration)
  }
  def this(conf: SparkConf, batchDuration: Duration) = {
    this(StreamingContext.createNewSparkContext(conf), null, batchDuration)
  }
  // batch interval 可以根据你的应用程序需求的延迟要求以及集群可用的资源情况来设置
  // 一旦StreamingContext定义好之后，就可以做一些事情
  ```

  ![image-20200613184508616](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613184508616.png)

* DStreams

  ![image-20200613184744119](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613184744119.png)

  * Internally, a DStream is represented by a continuous series of RDDs
  * Each RDD in a DStream contains data from a certain interval
  * 对DStream操作算子，比如map/flatMap，其实底层会被翻译为对DStream中的每个RDD都做相同的操作，因为一个DStream是由不同批次的RDD所构成的

* Input DStreams and Receivers

  ![image-20200613191844070](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613191844070.png)

  * Every input DStream (except file stream, discussed later in this section) is associated with a **Receiver** ([Scala doc](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.receiver.Receiver), [Java doc](http://spark.apache.org/docs/latest/api/java/org/apache/spark/streaming/receiver/Receiver.html)) object which receives the data from a source and stores it in Spark’s memory for processing.

### Transformations

* Transformations on DStreams
  * Similar to that of RDDs, transformations allow the data from the input DStream to be modified. DStreams support many of the transformations available on normal Spark RDD’s

### Output Operations

* Output Operations on DStreams
  * Output operations allow DStream’s data to be pushed out to external systems like a database or a file systems. Since the output operations actually allow the transformed data to be consumed by external systems, they trigger the actual execution of all the DStream transformations (similar to actions for RDDs)

### 案例实战

* SparkStreaming处理Socket数据

  ![image-20200613214121554](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200613214121554.png)

  ```shell
  # 待解决
  Exception in thread "main" java.lang.NoClassDefFoundError: scala/Cloneable
  ```

  ```scala
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  /**
    * Spark Streaming处理Socket数据
    * 测试： nc
    */
  object NetworkWordCount {
    def main(args: Array[String]): Unit = {
  //    val sparkConf = new SparkConf().setMaster("local").setAppName("NetworkWordCount")
      val sparkConf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
  
      /**
        * 创建StreamingContext需要两个参数：SparkConf和batch interval
        */
      val ssc = new StreamingContext(sparkConf, Seconds(2))
  
  //    val lines = ssc.socketTextStream("localhost", 6789)
      val lines = ssc.socketTextStream("192.168.0.221", 6789)
  
      val result = lines.flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_)
  
      result.print()
  
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

* Spark Streaming 处理HDFS文件数据

  ![image-20200614093028773](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614093028773.png)

  ```scala
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  /**
    * 使用Spark Streaming处理文件系统(local/hdfs)的数据
    */
  object FileWordCount {
    def main(args: Array[String]): Unit = {
  
      val sparkConf = new SparkConf().setMaster("local").setAppName("FileWordCount")
      val ssc = new StreamingContext(sparkConf, Seconds(5))
  
      // 本地测试，先启动，在创建文件
      // cd /Users/dingyuanjie/work/tmp
      // vim test.log
      // a a b b b c
      // cp test.log ss/
      val lines = ssc.textFileStream("file:///Users/dingyuanjie/work/tmp/ss/")
      val result = lines.flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_)
      result.print()
  
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

### 带状态的算子

* UpdateStateByKey

  ```scala
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  /**
    * 使用Spark Streaming完成有状态统计
    */
  object StatefulWordCount {
    def main(args: Array[String]): Unit = {
      val sparkConf = new SparkConf().setAppName("StatefulWordCount").setMaster("local[2]")
      val ssc = new StreamingContext(sparkConf, Seconds(5))
  
      // 如果使用了stateful的算子，必须要设置checkpoint
      // 在生产环境中，建议大家把checkpoint设置到HDFS的某个文件夹中
      ssc.checkpoint(".")
  
  //    val lines = ssc.socketTextStream("localhost", 6789)
      val lines = ssc.socketTextStream("192.168.0.221", 6789)
  
      val result = lines.flatMap(_.split(" ")).map((_,1))
      val state = result.updateStateByKey[Int](updateFunction _)
  
      state.print()
  
      ssc.start()
      ssc.awaitTermination()
    }
  
    /**
      * 把当前的数据去更新已有的或者是老的数据
      * @param currentValues  当前的
      * @param preValues  老的
      * @return
      */
    def updateFunction(currentValues: Seq[Int], preValues: Option[Int]): Option[Int] = {
      val current = currentValues.sum
      val pre = preValues.getOrElse(0)
  
      Some(current + pre)
    }
  }
  ```

### 实战

* 计算到目前为止累计出现的单词个数写入到MySQL中

  * 使用Spark Streaming进行统计分析

  * Spark Streaming统计结果写入到MySQL

    ```shell
    create database imooc_spark;
    use imooc_spark;
    create table wordcount(
    	word varchar(50) default null,
    	wordcount int(10) default null
    );
    # 通过该sql将统计结果写入MySQL
    "insert into wordcount(word, wordcount) values('" + record._1 + "'," + record._2 + ")"
    # 存在的问题：
    1） 对于已有的数据做更新，而是所有的数据均为insert
    		改进思路：
    			a) 在插入数据前先判断单词是否存在，如果存在就update，不存在则insert
    			b) 工作中：HBase / Redis
    2) 每个rdd的partition创建connection，建议改成连接池·			
    ```

    ```scala
    import org.apache.spark.SparkConf
    import org.apache.spark.streaming.{Seconds, StreamingContext}
    
    /**
      * 使用Spark Streaming完成词频统计，并将结果写入到MySQL数据库中
      */
    object ForeachRDDApp {
      def main(args: Array[String]): Unit = {
        val sparkConf = new SparkConf().setAppName("ForeachRDDApp").setMaster("local[2]")
        val ssc = new StreamingContext(sparkConf, Seconds(5))
        val lines = ssc.socketTextStream("localhost", 6789)
        val result = lines.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
        //result.print()  //此处仅仅是将统计结果输出到控制台
        //TODO... 将结果写入到MySQL
    //        result.foreachRDD(rdd =>{
    //          val connection = createConnection()  // executed at the driver
    //          rdd.foreach { record =>
    //            val sql = "insert into wordcount(word, wordcount) values('"+record._1 + "'," + record._2 +")"
    //            connection.createStatement().execute(sql)
    //          }
    //        })
        result.print()
        result.foreachRDD(rdd => {
          rdd.foreachPartition(partitionOfRecords => {
            val connection = createConnection()
            partitionOfRecords.foreach(record => {
              val sql = "insert into wordcount(word, wordcount) values('" + record._1 + "'," + record._2 + ")"
              connection.createStatement().execute(sql)
            })
            connection.close()
          })
        })
        ssc.start()
        ssc.awaitTermination()
      }
    
      /**
        * 获取MySQL的连接
        */
      def createConnection() = {
        Class.forName("com.mysql.cj.jdbc.Driver")
        DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/imooc_spark", "root", "IamMySQL0108")
      }
    }
    ```

### 基于window的统计

![image-20200614113106295](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614113106295.png)

```shell
# window：定时的进行一个时间段内的数据处理
window length：窗口的长度
sliding interval：窗口的间隔

这2个参数和我们的batch size有关系：倍数

每隔多久计算某个范围内的数据：每隔10秒计算前10分钟的wc
==》 每隔sliding interval统计前window length的值
```

### 实战

* 黑名单过滤

  * transform算子的使用

  * Spark Streaming整合RDD进行操作

    ```shell
    # 访问日志 ==> DStream
    20180808,zs
    20180808,ls
    20180808,ww
    		==> (zs: 20180808,zs)(ls: 20180808,ls)(ww: 20180808,ww)
    # 黑名单列表 ==> RDD
    zs
    ls
    		==> (zs: true)(ls:true)
    ==> 20180808,ww
    
    leftjoin
    (zs:[<20180808,zs>,<true>])  x
    (ls:[<20180808,ls>,<true>])  x
    (ww:[<20180808,ww>,<false>]) => tuple 1
    ```

    ```scala
    import org.apache.spark.SparkConf
    import org.apache.spark.streaming.{Seconds, StreamingContext}
    
    /**
      * 黑名单过滤
      */
    object TransformApp {
      def main(args: Array[String]): Unit = {
        val sparkConf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
        /**
          * 创建StreamingContext需要两个参数：SparkConf和batch interval
          */
        val ssc = new StreamingContext(sparkConf, Seconds(5))
        /**
          * 构建黑名单
          */
        val blacks = List("zs", "ls")
        val blacksRDD = ssc.sparkContext.parallelize(blacks).map(x => (x, true))
    
        val lines = ssc.socketTextStream("localhost", 6789)
        val clicklog = lines.map(x => (x.split(",")(1), x)).transform(rdd => {
          rdd.leftOuterJoin(blacksRDD)
            .filter(x=> x._2._2.getOrElse(false) != true)
            .map(x=>x._2._1)
        })
    
        clicklog.print()
    
        ssc.start()
        ssc.awaitTermination()
      }
    }
    ```

    ```scala
    > nc -lk 6789
    20180808,zs
    20180808,ls
    20180808,ww
    ```

* SparkStreaming整合Spark SQL实战

  ```scala
  import org.apache.spark.SparkConf
  import org.apache.spark.rdd.RDD
  import org.apache.spark.sql.SparkSession
  import org.apache.spark.streaming.{Seconds, StreamingContext, Time}
  /**
    * Spark Streaming整合Spark SQL完成词频统计操作
    */
  object SqlNetworkWordCount {
  
    def main(args: Array[String]): Unit = {
      val sparkConf = new SparkConf().setAppName("ForeachRDDApp").setMaster("local[2]")
      val ssc = new StreamingContext(sparkConf, Seconds(5))
      val lines = ssc.socketTextStream("localhost", 6789)
      val words = lines.flatMap(_.split(" "))
  
      // Convert RDDs of the words DStream to DataFrame and run SQL query
      words.foreachRDD { (rdd: RDD[String], time: Time) =>
        val spark = SparkSessionSingleton.getInstance(rdd.sparkContext.getConf)
        import spark.implicits._
        // Convert RDD[String] to RDD[case class] to DataFrame
        val wordsDataFrame = rdd.map(w => Record(w)).toDF()
        // Creates a temporary view using the DataFrame
        wordsDataFrame.createOrReplaceTempView("words")
  
        // Do word count on table using SQL and print it
        val wordCountsDataFrame =
          spark.sql("select word, count(*) as total from words group by word")
        println(s"========= $time =========")
        wordCountsDataFrame.show()
      }
      ssc.start()
      ssc.awaitTermination()
    }
  
    /** Case class for converting RDD to DataFrame */
    case class Record(word: String)
  
    /** Lazily instantiated singleton instance of SparkSession */
    object SparkSessionSingleton {
      @transient  private var instance: SparkSession = _
      def getInstance(sparkConf: SparkConf): SparkSession = {
        if (instance == null) {
          instance = SparkSession
            .builder
            .config(sparkConf)
            .getOrCreate()
        }
        instance
      }
    }
  }
  ```

  ```scala
  > nc -lk 6789
  a a a a b b b
  a a a a c c c
  d d d d e e e 
  ```

## SparkStreaming 整合Flume

###   实战一

* Flume-style Push-bashed Approach

![image-20200614134556558](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614134556558.png)

```properties
# Configure Flume agent to send data to an Avro sink by having the following in the configuration file.
agent.sinks = avroSink
agent.sinks.avroSink.type = avro
agent.sinks.avroSink.channel = memoryChannel
agent.sinks.avroSink.hostname = <chosen machine's hostname>
agent.sinks.avroSink.port = <chosen port on the machine>
```

```shell
cd $FLUME_HOME/conf
cp exec-memory-avro.conf flume_push_streaming.conf
vim flume_push_streaming.conf

simple-agent.sources = netcat-source
simple-agent.sinks = avro-sink
simple-agent.channels = memory-channel

simple-agent.sources.netcat-source.type = netcat
# 本地测试，则需将服务器IP改成本地IP 192.168.0.221
simple-agent.sources.netcat-source.bind = master
simple-agent.sources.netcat-source.port = 44444

simple-agent.sinks.avro-sink.type = avro
simple-agent.sinks.avro-sink.hostname = master
simple-agent.sinks.avro-sink.port = 41414

simple-agent.channels.memory-channel.type = memory

simple-agent.sources.netcat-source.channels = memory-channel
simple-agent.sinks.avro-sink.channel = memory-channel
```

```shell
# 测试 IDEA 配置
Program arguments: 0.0.0.0 41414
# 启动SparkStreaming作业

# 启动flume agent
flume-ng agent --name simple-agent --conf $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/flume_push_streaming.conf -Dflume.root.logger=INFO,console &

# 通过telnet服务器上44444输入数据，观察IDEA控制台的输出
telnet 192.168.0.221 44444
```

```shell
# 打包到生产 mvn clean package
# sparktrain-1.0.jar 上传到服务器
# 服务器上执行  要下载许多依赖包，todo：考虑先将依赖包上传上去
spark-submit --class com.imooc.spark.FlumePushWordCount --master local[2] --packages org.apache.spark:spark-streaming-flume_2.11:2.2.0 /data/hadoop/app/test_jar/sparktrain-1.0.jar master 41414
# spark-submit --class com.imooc.spark.FlumePushWordCount --master local[2] --jars /data/hadoop/app/test_jar/spark-streaming-flume-assembly_2.11-2.2.0.jar /data/hadoop/app/test_jar/sparktrain-1.0.jar master 41414
# 启动flume agent
flume-ng agent --name simple-agent --conf $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/flume_push_streaming.conf -Dflume.root.logger=INFO,console &
# 通过telnet服务器上44444输入数据，观察IDEA控制台的输出
telnet 192.168.0.221 44444
```

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.flume.FlumeUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * Spark Streaming整合Flume的第一种方式
  */
object FlumePushWordCount {
  def main(args: Array[String]): Unit = {
    if(args.length != 2) {
      System.err.println("Usage: FlumePushWordCount <hostname> <port>")
      System.exit(1)
    }
    val Array(hostname, port) = args
    // 打包到生产上
    val sparkConf = new SparkConf() //.setMaster("local[2]").setAppName("FlumePushWordCount")
    // 本地测试
//    val sparkConf = new SparkConf().setMaster("local[2]").setAppName("FlumePushWordCount")
    val ssc = new StreamingContext(sparkConf, Seconds(5))

    //TODO... 如何使用SparkStreaming整合Flume
    val flumeStream = FlumeUtils.createStream(ssc, hostname, port.toInt)

    flumeStream.map(x=> new String(x.event.getBody.array()).trim)
      .flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()

    ssc.start()
    ssc.awaitTermination()
  }
}
```

### 实战二

* Pull-based Approach using a Custom Sink

* 工作中常用

  ![image-20200614145227594](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614145227594.png)

  ```shell
  # Configuration file: On that machine, configure Flume agent to send data to an Avro sink by having the following in the configuration file.
  agent.sinks = spark
  agent.sinks.spark.type = org.apache.spark.streaming.flume.sink.SparkSink
  agent.sinks.spark.hostname = <hostname of the local machine>
  agent.sinks.spark.port = <port to listen on for connection from Spark>
  agent.sinks.spark.channel = memoryChannel
  ```

  ```shell
  cd $FLUME_HOME/conf
  cp flume_push_streaming.conf flume_pull_streaming.conf
  vim flume_pull_streaming.conf
  
  simple-agent.sources = netcat-source
  simple-agent.sinks = spark-sink
  simple-agent.channels = memory-channel
  
  simple-agent.sources.netcat-source.type = netcat
  # 本地测试，则需将服务器IP改成本地IP 192.168.0.221
  simple-agent.sources.netcat-source.bind = master
  simple-agent.sources.netcat-source.port = 44444
  
  simple-agent.sinks.spark-sink.type = org.apache.spark.streaming.flume.sink.SparkSink
  simple-agent.sinks.spark-sink.hostname = master
  simple-agent.sinks.spark-sink.port = 41414
  
  simple-agent.channels.memory-channel.type = memory
  
  simple-agent.sources.netcat-source.channels = memory-channel
  simple-agent.sinks.spark-sink.channel = memory-channel
  ```

  ```scala
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.flume.FlumeUtils
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  /**
    * Spark Streaming整合Flume的第二种方式
    */
  object FlumePullWordCount {
  
    def main(args: Array[String]): Unit = {
  
      if(args.length != 2) {
        System.err.println("Usage: FlumePullWordCount <hostname> <port>")
        System.exit(1)
      }
  
      val Array(hostname, port) = args
  
      val sparkConf = new SparkConf() //.setMaster("local[2]").setAppName("FlumePullWordCount")
      val ssc = new StreamingContext(sparkConf, Seconds(5))
  
      //TODO... 如何使用SparkStreaming整合Flume
      val flumeStream = FlumeUtils.createPollingStream(ssc, hostname, port.toInt)
  
      flumeStream.map(x=> new String(x.event.getBody.array()).trim)
        .flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()
  
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

  ```shell
  # 本地测试
  # 先启动flume，再启动Spark Streaming应用程序
  flume-ng agent --name simple-agent --conf $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/flume_pull_streaming.conf -Dflume.root.logger=INFO,console
  
  telnet 192.168.0.221 44444
  
  # IDEA
  Program arguments: 0.0.0.0 41414
  ```

  ```shell
  # 服务器测试
  # 打包到生产 mvn clean package
  # sparktrain-1.0.jar 上传到服务器
  # 启动flume agent
  flume-ng agent --name simple-agent --conf $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/flume_push_streaming.conf -Dflume.root.logger=INFO,console &
  # 服务器上执行  要下载许多依赖包，todo：考虑先将依赖包上传上去
  spark-submit --class com.imooc.spark.FlumePullWordCount --master local[2] --packages org.apache.spark:spark-streaming-flume_2.12:2.4.6 /data/hadoop/app/test_jar/sparktrain-1.0.jar master 41414
  # 通过telnet服务器上44444输入数据，观察IDEA控制台的输出
  telnet 192.168.0.221 44444
  ```

## Spark Streaming 集成Kafka

![image-20200614154504037](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614154504037.png)

## 实战一：Receiver-based

![image-20200614162259285](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614162259285.png)

```shell
# 启动zk、kafka
./zkServer.sh start
kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
jps
# 创建topic
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka_streaming_topic
./kafka-topics.sh --list --zookeeper localhost:2181
# 通过控制台验证kafka生产和消费消息
./kafka-console-producer.sh --broker-list localhost:9092 --topic kafka_streaming_topic
# ./kafka-console-consumer.sh -zookeeper localhost:2181 --topic kafka_streaming_topic
./kafka-console-consumer.sh --bootstrap-server master:9092 --topic kafka_streaming_topic
```

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * Spark Streaming对接Kafka的方式一
  */
object KafkaReceiverWordCount {
  def main(args: Array[String]): Unit = {
    if(args.length != 4) {
      System.err.println("Usage: KafkaReceiverWordCount <zkQuorum> <group> <topics> <numThreads>")
    }

    val Array(zkQuorum, group, topics, numThreads) = args

    // 服务器
//    val sparkConf = new SparkConf() //.setAppName("KafkaReceiverWordCount")
//      //.setMaster("local[2]")
    // 本地测试
    // 参数设置  Program arguments: master:2181 test kafka_streaming_topic 1
      val sparkConf = new SparkConf() .setAppName("KafkaReceiverWordCount").setMaster("local[2]")

    val ssc = new StreamingContext(sparkConf, Seconds(5))

    val topicMap = topics.split(",").map((_, numThreads.toInt)).toMap

    // TODO... Spark Streaming如何对接Kafka
    val messages = KafkaUtils.createStream(ssc, zkQuorum, group,topicMap)

    // TODO... 自己去测试为什么要取第二个
    messages.map(_._2).flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()

    ssc.start()
    ssc.awaitTermination()
  }
}
```

```shell
# 生产
# maven 打包、上传
spark-submit --class com.imooc.spark.KafkaReceiverWordCount --master local[2] --name KafkaReceiverWordCount --packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.2.0 /data/hadoop/app/test_jar/sparktrain-1.0.jar master:2181 test kafka_streaming_topic 1

# http://192.168.0.221:4040/
```

## 实战二：Direct Approach

* 工作中

![image-20200614162324237](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200614162324237.png)

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}
// spark-streaming-kafka-0-8_2.11与kafka0.9.0.0不匹配造成的。（不支持0.9.0.0的broker）
// 将kafka版本换为0.8.2.1，不报错可正常运行（此时恢复com.imooc.kafka，并使用 import _root_.kafka.serializer.StringDecoder语句)
import _root_.kafka.serializer.StringDecoder
/**
  * Spark Streaming对接Kafka的方式二
  */
object KafkaDirectWordCount {

  def main(args: Array[String]): Unit = {

    if(args.length != 2) {
      System.err.println("Usage: KafkaDirectWordCount <brokers> <topics>")
      System.exit(1)
    }

    val Array(brokers, topics) = args

//    val sparkConf = new SparkConf() //.setAppName("KafkaReceiverWordCount")
//      //.setMaster("local[2]")
    // master:9092 kafka_streaming_topic
    // 本地测试，kafka producer端输入数据，idea控制台查看结果
    val sparkConf = new SparkConf().setAppName("KafkaReceiverWordCount").setMaster("local[2]")

    val ssc = new StreamingContext(sparkConf, Seconds(5))

    val topicsSet = topics.split(",").toSet
    val kafkaParams = Map[String,String]("metadata.broker.list"-> brokers)

    // TODO... Spark Streaming如何对接Kafka
    val messages = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](
    ssc,kafkaParams,topicsSet
    )

    // TODO... 自己去测试为什么要取第二个
    messages.map(_._2).flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()

    ssc.start()
    ssc.awaitTermination()
  }
}
```

```shell
# 服务器
# maven打包上传
spark-submit --class com.imooc.spark.KafkaDirectWordCount --master local[2] --name KafkaDirectWordCount --packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.2.0 /data/hadoop/app/test_jar/sparktrain-1.0.jar master:9092 test kafka_streaming_topic 1

# http://192.168.0.221:4040/
```

## ==基于Spark Streaming&Flume&Kafka打造通用流处理平台==

![image-20200615203446718](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200615203446718.png)

### 整合日志输出到Flume

```properties
# streaming.conf
agent1.sources=avro-source
agent1.channels=logger-channel
agent1.sinks=log-sink
# define source 
agent1.sources.avro-source.type=avro
agent1.sources.avro-source.bind=0.0.0.0
agent1.sources.avro-source.port=41414
# define channel
agent1.channels.logger-channel.type=memory
# define sink
agent1.sinks.log-sink.type=logger

agent1.sources.avro-source.channels=logger-channel
agent1.sinks.log-sink.channel=logger-channel

# 启动flume
flume-ng agent -n agent1 -c $FLUME_HOME/conf -f $FLUME_HOME/conf/streaming.conf -Dflume.root.logger=INFO,console
```

```properties
# log4j.properties
log4j.rootLogger=INFO,stdout,flume

log4j.appender.flume = org.apache.flume.clients.log4jappender.Log4jAppender
log4j.appender.flume.Hostname = hadoop000
log4j.appender.flume.Port = 41414
log4j.appender.flume.UnsafeMode = true

<dependency>
	<groupId>org.apache.flume.flume-ng-clients</groupId>
	<artifactId>flume-ng-log4jappender</artifactId>
	<version>1.6.0</version>
</dependency>
```

```java
// 先启动flume，再启动logger产生日志
/**
 * 模拟日志产生
 */
public class LoggerGenerator {
    private static Logger logger = Logger.getLogger(LoggerGenerator.class.getName());
    public static void main(String[] args) throws Exception{
        int index = 0;
        while(true) {
            Thread.sleep(1000);
            logger.info("value : " + index++);
        }
    }
}
```

### 整合Flume到Kafka

```shell
# 启动ZK、Kafka
cd $ZK_HOME/bin
./zkServer.sh start
cd $KAFKA_HOME/bin
./kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
jps
./kafka-topics.sh --list --zookeeper master:2181
./kafka-topics.sh --create --zookeeper master:2181 --replication-factor 1 --partitions 1 --topic streamingtopic
```

```properties
# streaming2.conf
agent1.sources=avro-source
agent1.channels=logger-channel
agent1.sinks=kafka-sink
# define source
agent1.sources.avro-source.type=avro
agent1.sources.avro-source.bind=0.0.0.0
agent1.sources.avro-source.port=41414
# define channel
agent1.channels.logger-channel.type=memory
# define sink
agent1.sinks.kafka-sink.type=org.apache.flume.sink.kafka.KafkaSink
agent1.sinks.kafka-sink.topic=streamingtopic
agent1.sinks.kafka-sink.brokerList=master:9092
agent1.sinks.kafka-sink.requiredAcks=1
agent1.sinks.kafka-sink.batchSize=20

agent1.sources.avro-source.channels=logger-channel
agent1.sinks.kafka-sink.channel=logger-channel

# 启动flume
flume-ng agent -n agent1 -c $FLUME_HOME/conf -f $FLUME_HOME/conf/streaming2.conf -Dflume.root.logger=INFO,console
# 启动kafka消费者
./kafka-console-consumer.sh --bootstrap-server master:9092 --topic streamingtopic
# 启动log4j日志，打印日志，Kafka consumer就会消费日志消息
```

### 整合Kafka到Spark Streaming

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * Spark Streaming对接Kafka
  把数据从kafka中取出来，在SparkStreaming中消费
  */
object KafkaStreamingApp {

  def main(args: Array[String]): Unit = {

    if(args.length != 4) {
      System.err.println("Usage: KafkaStreamingApp <zkQuorum> <group> <topics> <numThreads>")
    }
		// 本地测试，Program arguments: master:2181 test streamingtopic 1
    val Array(zkQuorum, group, topics, numThreads) = args

    val sparkConf = new SparkConf().setAppName("KafkaReceiverWordCount")
      .setMaster("local[2]")

    val ssc = new StreamingContext(sparkConf, Seconds(5))

    val topicMap = topics.split(",").map((_, numThreads.toInt)).toMap

    // TODO... Spark Streaming如何对接Kafka
    val messages = KafkaUtils.createStream(ssc, zkQuorum, group,topicMap)

    // TODO... 自己去测试为什么要取第二个
    messages.map(_._2).count().print()

    ssc.start()
    ssc.awaitTermination()
  }
}
```

### Spark Streaming对接收到的数据进行处理

* 现在是在本地进行测试的，在IDEA中运行LoggerGenerator，然后使用Flume、Kafka以及Spark Streaming进行处理操作
* 在生产上肯定不是这么干，怎么干呢？
  1. 打包jar，执行LoggerGenerator类
  2. Flume、Kafka和测试时是一样的
  3. Spark Streaming的代码也是需要打成jar包，然后使用Spark-submit的方式进行提交到环境上执行，可以根据实际情况选择运行模式：local/yarn/standalone/mesos
* 在生产上，整个流处理的流程都一样的，区别在于业务逻辑的复杂性

## 项目实战

### 需求说明

* 今天到现在为止实战课程的访问量
* 今天到现在为止从搜索引擎引流过来的实战课程的访问量

### 互联网访问日志概述

* 为什么要记录用户访问行为日志

  * 网站页面的访问量
  * 网站粘性
  * 推荐

* 用户行为日志内容

  ![image-20200620224501429](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200620224501429.png)

* 用户行为日志分析的意义

  * 网站的眼睛
  * 网站的神经
  * 网站的大脑

* 使用Python脚本实时产生数据

  * Python实时日志产生器开发

    ```python 
    # coding=UTF-8
    
    import random
    import time
    
    url_paths = [
      "class/301.html",
      "class/385.html",
      "class/357.html",
      "class/335.html",
      "class/131.html",
      "class/130.html",
      "learn/821",
      "course/list"
    ]
    
    ip_slices = [132,156,124,10,29,167,143,187,30,46,55,63,72,87,98,168]
    
    http_referers = [
      "https://www.baidu.com/s?wd={query}",
      "https://www.sogou.com/web?query={query}",
      "https://cn.bing.com/search?q={query}",
      "https://search.yahoo.com/search?p={query}"
    ]
    
    search_keyword = [
      "Spark SQL实战",
      "Hadoop基础",
      "Storm实战",
      "Spark Streaming实战",
      "大数据面试"
    ]
    
    status_codes = ["200", "404", "500"]
    
    def sample_url():
      return random.sample(url_paths, 1)[0]
    
    def sample_ip():
      slice = random.sample(ip_slices, 4)
      return ".".join([str(item) for item in slice])
    
    def sample_referer():
      if random.uniform(0, 1) > 0.2:
        return "_"
      
      refer_str = random.sample(http_referers, 1)
      query_str = random.sample(search_keyword, 1)
      return refer_str[0].format(query=query_str[0])
    
    def sample_status_code():
      return random.sample(status_codes, 1)[0]
    
    def generate_log(count = 10):
      time_str = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
      
      f = open("/woody/data/project/logs/access.log", "w+")
      # f = open("/Users/dingyuanjie/dev_env/test/python_test/logs/access.log", "w+")
      
      while count >= 1:
        query_log = "{ip}\t{local_time}\t\"GET /{url} HTTP/1.1\"\t{status_code}\t{referer}".format(url=sample_url(),ip=sample_ip(), referer=sample_referer(), status_code=sample_status_code(), local_time=time_str)
        print(query_log)
        f.write(query_log + "\n")
        count = count - 1
    
    if __name__ == '__main__':
      generate_log(5)
    ```

    ```shell
    87.46.187.98	2020-06-21 11:07:04	"GET /class/131.html HTTP/1.1"	404	https://cn.bing.com/search?q=Storm实战
    55.187.98.46	2020-06-21 11:07:04	"GET /course/list HTTP/1.1"	404	_
    10.167.143.46	2020-06-21 11:07:04	"GET /class/131.html HTTP/1.1"	500	_
    124.87.156.168	2020-06-21 11:07:04	"GET /class/301.html HTTP/1.1"	500	https://search.yahoo.com/search?p=Hadoop基础
    187.167.98.87	2020-06-21 11:07:04	"GET /course/list HTTP/1.1"	200	https://cn.bing.com/search?q=Hadoop基础
    ```

    ```shell
    # https://tool.lu/crontab
    	每分钟执行一次：*/1 * * * *
    # log_generator.sh
    python3 /woody/data/project/py_access_log.py
    
    chmod u+x log_generator.sh
    
    # 执行
    crontab -e 
    	*/1 * * * * /woody/data/project/log_generator.sh
    ```

### 功能开发及本地运行

* 打通Flume&Kafka&Spark Streaming线路

  ```properties
  # 对接python日志产生器输出的日志到Flume
  streaming_project.conf
  # 选型 : access.log  ===> 控制台输出
  	exec
  	memory
  	logger
  	
  flume-ng agent -n exec-memory-logger -c $FLUME_HOME/conf -f /woody/data/project/streaming_project.conf -Dflume.root.logger=INFO,console	
  
  # streaming_project.conf
  exec-memory-logger.sources = exec-source
  exec-memory-logger.sinks = logger-sink
  exec-memory-logger.channels = memory-channel
  
  exec-memory-logger.sources.exec-source.type = exec
  exec-memory-logger.sources.exec-source.command = tail -F /woody/data/project/logs/access.log
  exec-memory-logger.sources.exec-source.shell = /bin/sh -c
  
  exec-memory-logger.channels.memory-channel.type = memory
  
  exec-memory-logger.sinks.logger-sink.type = logger
  
  exec-memory-logger.sources.exec-source.channels = memory-channel
  exec-memory-logger.sinks.logger-sink.channel = memory-channel
  ```

* 日志 -> flume -> Kafka

  ```shell
  # 启动ZK
  ./zkServer.sh start
  # 启动Kafka Server
  kafka-server-start.sh $KAFKA_HOME/config/server.properties
  # 修改Flume配置文件使得flume sink数据到KafKa
  
  # 创建topic
  kafka-topics.sh --create --zookeeper hadoop000:2181 --replication-factor 1 --partitions 1 --topic streamingtopic
  # 查看所有topic
  kafka-topics.sh --list --zookeeper hadoop000:2181
  kafka-topics.sh --describe --zookeeper hadoop000:2181
  # 发送消息
  # kafka-console-producer.sh --broker-list hadoop000:9092 --topic hello_topic
  # 消费消息,正常情况是阻塞在这等待消费
  kafka-console-consumer.sh --bootstrap-server woody:9092 --topic streamingtopic
  
  # 1 partitions have leader brokers without a matching listener, including
  # 错误原因是因为创建了2个partition
  # Generally one consumer should mapped with one partition
  
  # 再启动flume(注意 -n的名字 和配置中的名称要对应)
  flume-ng agent -n exec-memory-kafka -c $FLUME_HOME/conf -f /woody/data/project/streaming_project2.conf -Dflume.root.logger=INFO,console
  
  # streaming_project2.conf
  exec-memory-kafka.sources = exec-source
  exec-memory-kafka.sinks = kafka-sink
  exec-memory-kafka.channels = memory-channel
  
  exec-memory-kafka.sources.exec-source.type = exec
  exec-memory-kafka.sources.exec-source.command = tail -F /woody/data/project/logs/access.log
  exec-memory-kafka.sources.exec-source.shell = /bin/sh -c
  
  exec-memory-kafka.channels.memory-channel.type = memory
  
  exec-memory-kafka.sinks.kafka-sink.type = org.apache.flume.sink.kafka.KafkaSink
  exec-memory-kafka.sinks.kafka-sink.brokerList = woody:9092
  exec-memory-kafka.sinks.kafka-sink.topic = streamingtopic
  exec-memory-kafka.sinks.kafka-sink.batchSize = 5
  exec-memory-kafka.sinks.kafka-sink.requiredAcks = 1
  
  exec-memory-kafka.sources.exec-source.channels = memory-channel
  exec-memory-kafka.sinks.kafka-sink.channel = memory-channel
  ```

* 数据清洗

  ![image-20200621182315139](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200621182315139.png)

  ```scala
  # 启动定时任务产生日志-启动zk/Kafka消费者-启动flume
  // 数据清洗操作：从原始日志中取出我们所需要的字段信息就可以了
  // 数据清洗结果类似如下：
  ClickLog(124.156.98.187,20200621184601,131,500,_)
  ClickLog(87.132.55.72,20200621184601,357,404,_)
  ClickLog(187.98.55.143,20200621184601,131,500,_)
  ClickLog(143.29.46.87,20200621184601,335,404,_)
  ClickLog(87.156.63.46,20200621184601,301,500,_)
  ClickLog(143.29.124.46,20200621184601,385,404,_)
  ClickLog(55.143.124.30,20200621184601,130,404,_)
  ClickLog(124.187.72.167,20200621184601,130,200,_)
  ClickLog(10.156.29.167,20200621184601,335,500,_)
  ClickLog(143.46.168.72,20200621184601,385,200,https://www.sogou.com/web?query=Spark Streaming实战)
  
  // 到数据清洗完为止，日志中只包含了实战课程的日志
  ```

  ```scala
  /**
    * 使用Spark Streaming处理Kafka过来的数据
    */
  object ImoocStatStreamingApp {
  
    def main(args: Array[String]): Unit = {
  
      if (args.length != 4) {
        println("Usage: ImoocStatStreamingApp <zkQuorum> <group> <topics> <numThreads>")
        System.exit(1)
      }
  
      val Array(zkQuorum, groupId, topics, numThreads) = args
  
      // 本地测试
      val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp").setMaster("local[5]")
      val ssc = new StreamingContext(sparkConf, Seconds(60))
      val topicMap = topics.split(",").map((_, numThreads.toInt)).toMap
      val messages = KafkaUtils.createStream(ssc, zkQuorum, groupId, topicMap)
  
      // Program arguments: woody:2181 test streamingtopic_1 1
      // 注意配置本机的hosts  vim /etc/hosts
      // 测试步骤一：测试数据接收
      //messages.map(_._2).count().print
  
      // 测试步骤二：数据清洗
      val logs = messages.map(_._2)
      val cleanData = logs.map(line => {
        print("------")
        print(line.toString)
        val infos = line.split("\t")
  
        // infos(2) = "GET /class/130.html HTTP/1.1"
        // url = /class/130.html
        print(infos.toString)
        val url = infos(2).split(" ")(1)
        var courseId = 0
  
        // 把实战课程的课程编号拿到了
        if (url.startsWith("/class")) {
          val courseIdHTML = url.split("/")(2)
          courseId = courseIdHTML.substring(0, courseIdHTML.lastIndexOf(".")).toInt
        }
  
        ClickLog(infos(0), DateUtils.parseToMinute(infos(1)), courseId, infos(3).toInt, infos(4))
      }).filter(clicklog => clicklog.courseId != 0)
  
      cleanData.print()
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

* 功能：统计今天到现在为止实战课程的访问量

  ```shell
  # yyyyMMdd courseid
  # 使用数据库来进行存储我们的统计结果
  # 	Spark Streaming 把统计结果写入到数据库里面
  #   可视化前端根据：yyyyMMdd 把数据库里面的统计结果展示出来
  # HBase：NoSQL
  #		一个API就能搞定，非常方便
  #   20171111 + 1 =》 click_count + 下一个批次的统计结果
  #   本次课程为什么要选择HBase的一个原因所在
  # 前提：HDFS、Zookeeper、HBase
  cd $HADOOP_HOME/sbin
  ./start-dfs.sh
  cd $ZOOKEEPER_HOME/bin
  ./zkServer.shs start
  cd $HBASE_HOME/bin
  ./start-hbase.sh
  
  ./hbase shell
  	list
  	create 'imooc_course_clickcount', 'info'
  	list
  	desc 'imooc_course_clickcount'
  	scan 'imooc_course_clickcount'
  	
  jps -m
  # HBase表设计
  # 	创建表：create 'imooc_course_clickcount' 'info'
  #   Rowkey设计：day_courseid
  ```

* 如何使用Scala来操作HBase

  ```scala
  /**
    * 实战课程点击数实体类
    * @param day_course  对应的就是HBase中的rowkey，20171111_1
    * @param click_count 对应的20171111_1的访问总数
    */
  case class CourseClickCount(day_course:String, click_count:Long)
  ```

  ```scala
  /**
    * 实战课程点击数-数据访问层
    */
  object CourseClickCountDAO {
  
    val tableName = "imooc_course_clickcount"
    val cf = "info"
    val qualifer = "click_count"
  
  
    /**
      * 保存数据到HBase
      * @param list  CourseClickCount集合
      */
    def save(list: ListBuffer[CourseClickCount]): Unit = {
  
      val table = HBaseUtils.getInstance().getTable(tableName)
  
      for(ele <- list) {
        table.incrementColumnValue(Bytes.toBytes(ele.day_course),
          Bytes.toBytes(cf),
          Bytes.toBytes(qualifer),
          ele.click_count)
      }
    }
  
    /**
      * 根据rowkey查询值
      */
    def count(day_course: String):Long = {
      val table = HBaseUtils.getInstance().getTable(tableName)
  
      val get = new Get(Bytes.toBytes(day_course))
      val value = table.get(get).getValue(cf.getBytes, qualifer.getBytes)
  
      if(value == null) {
        0L
      }else{
        Bytes.toLong(value)
      }
    }
  
    def main(args: Array[String]): Unit = {
      val list = new ListBuffer[CourseClickCount]
      list.append(CourseClickCount("20171111_8",8))
      list.append(CourseClickCount("20171111_9",9))
      list.append(CourseClickCount("20171111_1",100))
  
      save(list)
  
      println(count("20171111_8") + " : " + count("20171111_9")+ " : " + count("20171111_1"))
    }
  }
  ```

  ```java
  package com.imooc.spark.project.utils;
  
  import org.apache.hadoop.conf.Configuration;
  import org.apache.hadoop.hbase.client.HBaseAdmin;
  import org.apache.hadoop.hbase.client.HTable;
  import org.apache.hadoop.hbase.client.Put;
  import org.apache.hadoop.hbase.util.Bytes;
  
  import java.io.IOException;
  
  /**
   * HBase操作工具类：Java工具类建议采用单例模式封装
   */
  public class HBaseUtils {
  
  
      HBaseAdmin admin = null;
      Configuration configuration = null;
  
  
      /**
       * 私有改造方法
       */
      private HBaseUtils(){
          configuration = new Configuration();
          configuration.set("hbase.zookeeper.quorum", "hadoop000:2181");
          configuration.set("hbase.rootdir", "hdfs://hadoop000:8020/hbase");
  
          try {
              admin = new HBaseAdmin(configuration);
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      private static HBaseUtils instance = null;
  
      public  static synchronized HBaseUtils getInstance() {
          if(null == instance) {
              instance = new HBaseUtils();
          }
          return instance;
      }
  
  
      /**
       * 根据表名获取到HTable实例
       */
      public HTable getTable(String tableName) {
  
          HTable table = null;
  
          try {
              table = new HTable(configuration, tableName);
          } catch (IOException e) {
              e.printStackTrace();
          }
  
          return table;
      }
  
      /**
       * 添加一条记录到HBase表
       * @param tableName HBase表名
       * @param rowkey  HBase表的rowkey
       * @param cf HBase表的columnfamily
       * @param column HBase表的列
       * @param value  写入HBase表的值
       */
      public void put(String tableName, String rowkey, String cf, String column, String value) {
          HTable table = getTable(tableName);
  
          Put put = new Put(Bytes.toBytes(rowkey));
          put.add(Bytes.toBytes(cf), Bytes.toBytes(column), Bytes.toBytes(value));
  
          try {
              table.put(put);
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      public static void main(String[] args) {
  
          //HTable table = HBaseUtils.getInstance().getTable("imooc_course_clickcount");
          //System.out.println(table.getName().getNameAsString());
  
          String tableName = "imooc_course_clickcount" ;
          String rowkey = "20171111_88";
          String cf = "info" ;
          String column = "click_count";
          String value = "2";
  
          HBaseUtils.getInstance().put(tableName, rowkey, cf, column, value);
      }
  }
  ```

  ```scala
  package com.imooc.spark.project.spark
  
  import com.imooc.spark.project.dao.{CourseClickCountDAO, CourseSearchClickCountDAO}
  import com.imooc.spark.project.domain.{ClickLog, CourseClickCount, CourseSearchClickCount}
  import com.imooc.spark.project.utils.DateUtils
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.kafka.KafkaUtils
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  import scala.collection.mutable.ListBuffer
  
  /**
    * 使用Spark Streaming处理Kafka过来的数据
    */
  object ImoocStatStreamingApp {
  
    def main(args: Array[String]): Unit = {
  
      if (args.length != 4) {
        println("Usage: ImoocStatStreamingApp <zkQuorum> <group> <topics> <numThreads>")
        System.exit(1)
      }
  
      val Array(zkQuorum, groupId, topics, numThreads) = args
  
      // 本地测试
      val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp").setMaster("local[5]")
  
  //    val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp") //.setMaster("local[5]")
      val ssc = new StreamingContext(sparkConf, Seconds(60))
  
      val topicMap = topics.split(",").map((_, numThreads.toInt)).toMap
  
      val messages = KafkaUtils.createStream(ssc, zkQuorum, groupId, topicMap)
  
      // Program arguments: woody:2181 test streamingtopic_1 1
      // 注意配置本机的hosts  vim /etc/hosts
      // 测试步骤一：测试数据接收
      //messages.map(_._2).count().print
  
      // 测试步骤二：数据清洗
      // # 定时任务产生日志、zk/kafka、flume都已起来
      // hbase中执行 hbase(main):015:0> scan 'imooc_course_clickcount'  查看清洗后存入的数据
      val logs = messages.map(_._2)
      val cleanData = logs.map(line => {
        print("------")
        print(line.toString)
        val infos = line.split("\t")
  
        // infos(2) = "GET /class/130.html HTTP/1.1"
        // url = /class/130.html
        print(infos.toString)
        val url = infos(2).split(" ")(1)
        var courseId = 0
  
        // 把实战课程的课程编号拿到了
        if (url.startsWith("/class")) {
          val courseIdHTML = url.split("/")(2)
          courseId = courseIdHTML.substring(0, courseIdHTML.lastIndexOf(".")).toInt
        }
  
        ClickLog(infos(0), DateUtils.parseToMinute(infos(1)), courseId, infos(3).toInt, infos(4))
      }).filter(clicklog => clicklog.courseId != 0)
  
      cleanData.print()
  //    // 测试步骤三：统计今天到现在为止实战课程的访问量
      cleanData.map(x => {
        // HBase rowkey设计： 20171111_88
        (x.time.substring(0, 8) + "_" + x.courseId, 1)
      }).reduceByKey(_ + _).foreachRDD(rdd => {
        rdd.foreachPartition(partitionRecords => {
          val list = new ListBuffer[CourseClickCount]
  
          partitionRecords.foreach(pair => {
            list.append(CourseClickCount(pair._1, pair._2))
          })
  
          CourseClickCountDAO.save(list)
        })
      })
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

  ![image-20200621193618310](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200621193618310.png)

* 功能：统计今天到现在为止从搜索引擎引流过来的实战课程的访问量

  ```shell
  # 功能一+ 从搜索引擎引流过来的
  # HBase表设计
  create 'imooc_course_search_clickcount','info'
  scan 'imooc_course_search_clickcount'
  # 清除数据
  # truncate 'imooc_course_search_clickcount'
  # rowKey设计：根据业务需求设计
  
  ```

  ```scala
  package com.imooc.spark.project.dao
  
  import com.imooc.spark.project.domain.{CourseClickCount, CourseSearchClickCount}
  import com.imooc.spark.project.utils.HBaseUtils
  import org.apache.hadoop.hbase.client.Get
  import org.apache.hadoop.hbase.util.Bytes
  import scala.collection.mutable.ListBuffer
  
  /**
    * 从搜索引擎过来的实战课程点击数-数据访问层
    */
  object CourseSearchClickCountDAO {
  
    val tableName = "imooc_course_search_clickcount"
    val cf = "info"
    val qualifer = "click_count"
  
    /**
      * 保存数据到HBase
      * @param list  CourseSearchClickCount集合
      */
    def save(list: ListBuffer[CourseSearchClickCount]): Unit = {
      val table = HBaseUtils.getInstance().getTable(tableName)
      for(ele <- list) {
        table.incrementColumnValue(Bytes.toBytes(ele.day_search_course),
          Bytes.toBytes(cf),
          Bytes.toBytes(qualifer),
          ele.click_count)
      }
    }
  
    /**
      * 根据rowkey查询值
      */
    def count(day_search_course: String):Long = {
      val table = HBaseUtils.getInstance().getTable(tableName)
  
      val get = new Get(Bytes.toBytes(day_search_course))
      val value = table.get(get).getValue(cf.getBytes, qualifer.getBytes)
  
      if(value == null) {
        0L
      }else{
        Bytes.toLong(value)
      }
    }
  
    def main(args: Array[String]): Unit = {
      val list = new ListBuffer[CourseSearchClickCount]
      list.append(CourseSearchClickCount("20171111_www.baidu.com_8",8))
      list.append(CourseSearchClickCount("20171111_cn.bing.com_9",9))
      save(list)
      println(count("20171111_www.baidu.com_8") + " : " + count("20171111_cn.bing.com_9"))
    }
  }
  ```

  ![image-20200621194347963](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200621194347963.png)

  ```scala
  import com.imooc.spark.project.dao.{CourseClickCountDAO, CourseSearchClickCountDAO}
  import com.imooc.spark.project.domain.{ClickLog, CourseClickCount, CourseSearchClickCount}
  import com.imooc.spark.project.utils.DateUtils
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.kafka.KafkaUtils
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  import scala.collection.mutable.ListBuffer
  
  /**
    * 使用Spark Streaming处理Kafka过来的数据
    */
  object ImoocStatStreamingApp {
  
    def main(args: Array[String]): Unit = {
  
      if (args.length != 4) {
        println("Usage: ImoocStatStreamingApp <zkQuorum> <group> <topics> <numThreads>")
        System.exit(1)
      }
  
      val Array(zkQuorum, groupId, topics, numThreads) = args
  
      // 本地测试
      val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp").setMaster("local[5]")
  
  //    val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp") //.setMaster("local[5]")
      val ssc = new StreamingContext(sparkConf, Seconds(60))
  
      val topicMap = topics.split(",").map((_, numThreads.toInt)).toMap
  
      val messages = KafkaUtils.createStream(ssc, zkQuorum, groupId, topicMap)
  
      // Program arguments: woody:2181 test streamingtopic_1 1
      // 注意配置本机的hosts  vim /etc/hosts
      // 测试步骤一：测试数据接收
      //messages.map(_._2).count().print
  
      // 测试步骤二：数据清洗
      val logs = messages.map(_._2)
      val cleanData = logs.map(line => {
        print("------")
        print(line.toString)
        val infos = line.split("\t")
  
        // infos(2) = "GET /class/130.html HTTP/1.1"
        // url = /class/130.html
        print(infos.toString)
        val url = infos(2).split(" ")(1)
        var courseId = 0
        // 把实战课程的课程编号拿到了
        if (url.startsWith("/class")) {
          val courseIdHTML = url.split("/")(2)
          courseId = courseIdHTML.substring(0, courseIdHTML.lastIndexOf(".")).toInt
        }
        ClickLog(infos(0), DateUtils.parseToMinute(infos(1)), courseId, infos(3).toInt, infos(4))
      }).filter(clicklog => clicklog.courseId != 0)
      cleanData.print()
  
  //    // 测试步骤三：统计今天到现在为止实战课程的访问量
      cleanData.map(x => {
        // HBase rowkey设计： 20171111_88
        (x.time.substring(0, 8) + "_" + x.courseId, 1)
      }).reduceByKey(_ + _).foreachRDD(rdd => {
        rdd.foreachPartition(partitionRecords => {
          val list = new ListBuffer[CourseClickCount]
          partitionRecords.foreach(pair => {
            list.append(CourseClickCount(pair._1, pair._2))
          })
          CourseClickCountDAO.save(list)
        })
      })
  
      // 测试步骤四：统计从搜索引擎过来的今天到现在为止实战课程的访问量
      cleanData.map(x => {
        /**
          * https://www.sogou.com/web?query=Spark SQL实战
          * ==>
          * https:/www.sogou.com/web?query=Spark SQL实战
          */
        val referer = x.referer.replaceAll("//", "/")
        val splits = referer.split("/")
        var host = ""
        if(splits.length > 2) {
          host = splits(1)
        }
  
        (host, x.courseId, x.time)
      }).filter(_._1 != "").map(x => {
        (x._3.substring(0,8) + "_" + x._1 + "_" + x._2 , 1)
      }).reduceByKey(_ + _).foreachRDD(rdd => {
        rdd.foreachPartition(partitionRecords => {
          val list = new ListBuffer[CourseSearchClickCount]
          partitionRecords.foreach(pair => {
            list.append(CourseSearchClickCount(pair._1, pair._2))
          })
          CourseSearchClickCountDAO.save(list)
        })
      })
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

  ![image-20200621194900126](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200621194900126.png)

### 生产环境运行

* 编译打包`mvn clean package -DskipTests`

  ```scala
  // 本地测试
  //	val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp").setMaster("local[5]")
  // 生产
  	val sparkConf = new SparkConf().setAppName("ImoocStatStreamingApp") //.setMaster("local[5]")
  ```

  ```xml
  <build>
  <!--需注释掉-->
  <!--
  <sourceDirectory>src/main/scala</sourceDirectory>
  <testSourceDirectory>src/test/scala</testSourceDirectory>
  -->
  ```

  ```shell
  scp /Users/dingyuanjie/code/sparktrain/target/sparktrain-1.0.jar root@192.168.0.226:/woody/lib
  ```

* 运行

  ```shell
  cd $SPARK_HOME/sbin
  ./spark-submit --master local[5] \
  --class com.imooc.spark.project.spark.ImoocStatStreamingApp \
  /woody/lib/sparktrain-1.0.jar \
  woody:2181 test streamingtopic_1 1
  ```

  ```shell
  # 报错
  Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/spark/streaming/kafka/KafkaUtils$
  	at com.imooc.spark.project.spark.ImoocStatStreamingApp$.main(ImoocStatStreamingApp.scala:34)
  	at com.imooc.spark.project.spark.ImoocStatStreamingApp.main(ImoocStatStreamingApp.scala)
  	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  	at java.lang.reflect.Method.invoke(Method.java:498)
  	at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
  	at org.apache.spark.deploy.SparkSubmit.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:845)
  	at org.apache.spark.deploy.SparkSubmit.doRunMain$1(SparkSubmit.scala:161)
  	at org.apache.spark.deploy.SparkSubmit.submit(SparkSubmit.scala:184)
  	at org.apache.spark.deploy.SparkSubmit.doSubmit(SparkSubmit.scala:86)
  	at org.apache.spark.deploy.SparkSubmit$$anon$2.doSubmit(SparkSubmit.scala:920)
  	at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:929)
  	at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
  Caused by: java.lang.ClassNotFoundException: org.apache.spark.streaming.kafka.KafkaUtils$
  	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
  	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
  	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
  	... 14 more
  # 报错
  java.lang.NoClassDefFoundError: org/apache/hadoop/hbase/client/HBaseAdmin
  	at com.imooc.spark.project.utils.HBaseUtils.<init>(HBaseUtils.java:32)
  	at com.imooc.spark.project.utils.HBaseUtils.getInstance(HBaseUtils.java:42)
  	at com.imooc.spark.project.dao.CourseClickCountDAO$.save(CourseClickCountDAO.scala:26)
  	at com.imooc.spark.project.spark.ImoocStatStreamingApp$$anonfun$main$4$$anonfun$apply$1.apply(ImoocStatStreamingApp.scala:80)
  	at com.imooc.spark.project.spark.ImoocStatStreamingApp$$anonfun$main$4$$anonfun$apply$1.apply(ImoocStatStreamingApp.scala:73)
  	at org.apache.spark.rdd.RDD$$anonfun$foreachPartition$1$$anonfun$apply$28.apply(RDD.scala:980)
  	at org.apache.spark.rdd.RDD$$anonfun$foreachPartition$1$$anonfun$apply$28.apply(RDD.scala:980)
  	at org.apache.spark.SparkContext$$anonfun$runJob$5.apply(SparkContext.scala:2101)
  	at org.apache.spark.SparkContext$$anonfun$runJob$5.apply(SparkContext.scala:2101)
  	at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:90)
  	at org.apache.spark.scheduler.Task.run(Task.scala:123)
  	at org.apache.spark.executor.Executor$TaskRunner$$anonfun$10.apply(Executor.scala:408)
  	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1360)
  	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:414)
  	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  	at java.lang.Thread.run(Thread.java:748)
  ```

  ```shell
  # 解决
  ./spark-submit --master local[5] \
  --class com.imooc.spark.project.spark.ImoocStatStreamingApp \
  --packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.2.0 \
  --jars $(echo /woody/app/hbase-1.4.13/lib/*.jar | tr '' ',') \
  /woody/lib/sparktrain-1.0.jar \
  woody:2181 test streamingtopic_1 1
  ```

  ```shell
  # 提交作业时，注意事项  （需在--class后面）
  1）--packages的使用
  2) --jars的使用
  ```

## 可视化

![image-20200621202054831](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200621202054831.png)

* SpringBoot构建Web项目

* 使用Echarts构建静态数据可视化

* 使用Echarts构建动态数据可视化

  ```java
  // SpringBoot 整合Echarts动态获取HBase的数据
  1）动态的传递进去当天的时间
    	a)在代码中写死
    	b)查询昨天的、前天的
    			在页面中放一个时间插件(JQuery插件)，默认只读取当天的数据
  2）自动刷新展示图
    	每隔多久发送一个请求去刷新当前的数据供展示
    
  统计慕课网当天实战课程从搜索引擎过来的点击量
    	数据已经在HBase中有的
    	自己通过Echarts整合SpringBoot方式自己来实现
  ```

* 阿里云DataV数据可视化





