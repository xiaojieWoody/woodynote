# MapReduce简介

* 一种大规模数据处理的编程模型

![image-20200101202112977](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101202112977.png)

## MapReduce计算场景

* 数据查找
  * 分布式Grep
* Web访问日志分析
  * 词频统计
  * 网站PV UV统计
  * Top K问题
* 倒排索引
  * 建立搜索引擎索引
* 分布式排序

## MapReduce优点和缺点

* 特点
  * 模型简单
    * Map + Reduce
  * 高伸缩性
    * 支持横向扩展
  * 灵活
    * 结构化和非结构化数据
  * 速度快
    * 高吞吐离线处理数据
  * 并行处理
    * 编程模型天然支持并行处理
  * 容错能力强
* 缺点
  * 流式数据-MapReduce处理模型就决定了需要静态数据
  * 实时计算-不适合低延时数据处理，需要毫秒级别响应
  * 复杂算法-例如SVM支持向量机
  * 迭代计算-例如斐波拉契数列

# MapReduce编程模型

## MapReduce编程模型

* 如何统计一个文本中单词的出现次数？

  * Bash命令实现

    ```shell
    tr -s """\n"
    sort file
    uniq -c
    cat wordcount.txt|tr -s """\n"|sort|uniq -c
    ```

  * 单机版实现

    * 使用HashMap统计

  * 如果数据量极大如何在分布式的机器上计算？

    * MapReduce使用了分治思想简化了计算处理模型分为两步
      * Map阶段
        * 获得输入数据
        * 对输入数据进行转换并输出
      * Reduce阶段
        * 对输出结果进行聚合计算

## MapReduce-Map

* 对输入数据集的每个值都执行函数以创建新的结果集合

* 例如

  * 输入数据[1,2,3,4,5,6,7,8,9,10]
  * 定义Map方法需要执行的变换为f(x)=sin(x)
  * 则输出结果为[0.84,0.91,0.14,-0.76,-0.96,-0.28,0.66,0.99,0.41,-0.54]

* 形式化的表达

  * each <key,value> in input

  * map<key,value> to <intermediate key, intermediate value>

    ![image-20200101204520213](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101204520213.png)

## MapReduce-Reduce

* 对Map输出的结果进行聚合，输出一个或者多个聚合的结果

* 例如

  * Map的输出结果为[0.84,0.91,0.14,-0.76,-0.96,-0.28,0.66,0.99,0.41,-0.54]
  * 使用求和+作为聚合方法
  * 则输出结果为[1.41]

* 形式化的表达

  * each <intermediate key, List<intermediate value>> in input
  * reduce<reduce key, reduce value>

  ![image-20200101204932224](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101204932224.png)

* ==相同的key会被划分到同一个Reduce上==

  ![image-20200101205051345](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101205051345.png)

  ![image-20200101205141775](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101205141775.png)

## Map数据输入

* Map阶段由一定的数量的Map Task组成
* 文件分片
  * 输入数据会被split切分为多份
  * HDFS默认Block大小
    * Hadoop 1.0 = 64MB
    * Hadoop 2.0 = 128MB
  * 默认将文件解析为<key,value>对的实现是TextInputFormat
    * key为偏移量
    * value为每一行内容
* 因此有多少个Map Task任务？
  * 一个split一个Map Task，默认情况下一个block对应一个split
  * 例如一个文件大小为10TB，block大小设置为128MB，则一共会有81920个Map Task任务（10 * 1024 * 1024 / 128 = 81920）

## Reduce数据输入

* Partitioner决定了哪个Reduce会接收到Map输出的<key, value>对
* 在Hadoop中默认的Partitioner实现为HashPartitioner
* 计算公式
  * Abs(Hash(key)) mod NR 其中NR 等于Reduce Task数目
* Partitioner可以自定义
* 例如
  * 有3个Reduce Task
  * 那么Partitioner会返回0～2

## MapReduce-Shuffle

![image-20200101210251801](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101210251801.png)

![image-20200101210534669](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101210534669.png)

## Shuffle

* 为何需要shuffle
  * Reduce阶段的数据来源于不同的Map
* Shuffle由Map端和Reduce端组成
* ==Shuffle的核心机制==
  * 数据分区 + 排序
* Map端
  * 对Map输出结果进行spill(溢写)
* Reduce端
  * 拷贝Map端输出结果到本地
  * 对拷贝的数据进行归并排序

## Shuffle Map端

![image-20200101211015609](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101211015609.png)

![image-20200101211059703](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101211059703.png)

## Shuffle Reduce端

* Map端完成之后会暴露一个Http Server供Reduce端获取数据

* Reduce启动拷贝线程从各个Map端拷贝结果

  * 有大量的网络I/O开销

* 一边拷贝一边进行Merge操作（归并排序）

  ![image-20200101211444159](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101211444159.png)

## Combiner

* Map端本地Reducer
* ==合并了Map端输出数据 =》 减少Http Traffic==
* Combiner可以自定义
* 例如Word Count中
  * 对同一个Map输出的相同的key，直接对其value进行reduce
* 可以使用Combiner的前提
  * 满足结合律：求最大值、求和
  * 不适用场景：计算平均数
* Combiner在Map什么阶段发生？
  * Shuffle中的溢写文件时

## Combiner Example

![image-20200101211941739](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101211941739.png)

# MapReduce应用API介绍

## MapReduce Java API

* 基于Hadoop 3.0.0版本

* 新版本API均在包org.apache.hadoop.mapreduce下面

* 编写MapReduce程序的核心

  * 继承Hadoop提供的Mapper类并实现其中的map方法

    ```java
    public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>
    ```

  * 继承Hadoop提供的Reducer类并实现其中的reduce方法

    ```java
    // KEYIN为map的KEYOUY、VALUEIN为map的VALUEOUT
    public class Reducer<KEYIN,VALUEIN, KEYOUT,VALUEOUT>
    ```

## Word Count Example

![image-20200101212430878](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101212430878.png)

## Word Count Mapper实现

![image-20200101212529981](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101212529981.png)

## Word Count Reducer实现

![image-20200101212627496](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101212627496.png)

## WordCount Main方法实现

![image-20200101212721286](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101212721286.png)

## WordCount 编译和执行

* 通过mvn compile package将code打包成jar
* 将example数据上传至HDFS
  * hadoop fs -mkdir /user/cdh/wordcount/input
  * hadoop fs -put example.txt /user/cdh/wordcount/input
* 使用hadoop jar命令执行
  * hadoop jar命令格式:hadoop jar <job jar file path> <main class> <arg1> <arg2> ...
  * hadoop jar example.jar wordcount /user/cdh/wordcount/input /user/cdh/wordcount/output

```shell
# 查看目录
hadoop fs -ls /
# 递归创建目录
hadoop fs -mkdir -p /user/hadoop/mapreduce/demo/input
# 文件put到文件夹中
hadoop fs -put /home/hadoop/software/temp/geci.txt /user/hadoop/mapreduce/demo/input
# 查看文件内容
hadoop fs -cat /user/hadoop/mapreduce/demo/input/*
# mapreduce不允许把结果输出到一个已经存在的文件上去
# 删除文件
hadoop fs -rm -r /user/hadoop/mapreduce/demo/output
# 歌词文件存入/user/hadoop/mapreduce/demo/input中
/user/hadoop/mapreduce/demo/input/geci.txt
# 运行任务   
hadoop jar /home/hadoop/software/temp/metriccount_mr-1.0-SNAPSHOT-jar-with-dependencies.jar com.gupao.bigdata.mapreduce.WordCount /user/hadoop/mapreduce/demo/input /user/hadoop/mapreduce/demo/output
# 查看结果
hadoop fs -cat /user/hadoop/mapreduce/demo/output/*
```

```shell
[hadoop@master temp]$ hadoop jar /home/hadoop/software/temp/metriccount_mr-1.0-SNAPSHOT-jar-with-dependencies.jar com.gupao.bigdata.mapreduce.WordCount /user/hadoop/mapreduce/demo/input /user/hadoop/mapreduce/demo/output
2020-01-09 22:26:57,436 INFO client.RMProxy: Connecting to ResourceManager at master/192.168.0.201:8040
2020-01-09 22:26:58,219 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2020-01-09 22:26:58,307 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1578579390795_0001
2020-01-09 22:26:59,542 INFO input.FileInputFormat: Total input files to process : 1
2020-01-09 22:26:59,663 INFO mapreduce.JobSubmitter: number of splits:1
2020-01-09 22:26:59,706 INFO Configuration.deprecation: yarn.resourcemanager.system-metrics-publisher.enabled is deprecated. Instead, use yarn.system-metrics-publisher.enabled
2020-01-09 22:26:59,967 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1578579390795_0001
2020-01-09 22:26:59,968 INFO mapreduce.JobSubmitter: Executing with tokens: []
2020-01-09 22:27:00,258 INFO conf.Configuration: resource-types.xml not found
2020-01-09 22:27:00,258 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2020-01-09 22:27:00,264 WARN mapred.YARNRunner: Configuration yarn.app.mapreduce.am.resource.memory-mb=512Mi is overriding the yarn.app.mapreduce.am.resource.mb=1536 configuration
2020-01-09 22:27:01,171 INFO impl.YarnClientImpl: Submitted application application_1578579390795_0001
2020-01-09 22:27:01,299 INFO mapreduce.Job: The url to track the job: http://master:8088/proxy/application_1578579390795_0001/
2020-01-09 22:27:01,299 INFO mapreduce.Job: Running job: job_1578579390795_0001
2020-01-09 22:27:15,646 INFO mapreduce.Job: Job job_1578579390795_0001 running in uber mode : false
2020-01-09 22:27:15,647 INFO mapreduce.Job:  map 0% reduce 0%
2020-01-09 22:27:26,797 INFO mapreduce.Job:  map 100% reduce 0%
2020-01-09 22:27:33,872 INFO mapreduce.Job:  map 100% reduce 100%
2020-01-09 22:27:34,904 INFO mapreduce.Job: Job job_1578579390795_0001 completed successfully
2020-01-09 22:27:35,017 INFO mapreduce.Job: Counters: 53
	File System Counters
		FILE: Number of bytes read=1122
		FILE: Number of bytes written=432407
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=850
		HDFS: Number of bytes written=721
		HDFS: Number of read operations=8
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=8488
		Total time spent by all reduces in occupied slots (ms)=4348
		Total time spent by all map tasks (ms)=8488
		Total time spent by all reduce tasks (ms)=4348
		Total vcore-milliseconds taken by all map tasks=8488
		Total vcore-milliseconds taken by all reduce tasks=4348
		Total megabyte-milliseconds taken by all map tasks=4345856
		Total megabyte-milliseconds taken by all reduce tasks=2226176
	Map-Reduce Framework
		Map input records=25
		Map output records=147
		Map output bytes=1309
		Map output materialized bytes=1122
		Input split bytes=125
		Combine input records=147
		Combine output records=99
		Reduce input groups=99
		Reduce shuffle bytes=1122
		Reduce input records=99
		Reduce output records=99
		Spilled Records=198
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=212
		CPU time spent (ms)=1260
		Physical memory (bytes) snapshot=328028160
		Virtual memory (bytes) snapshot=4621082624
		Total committed heap usage (bytes)=144384000
		Peak Map Physical memory (bytes)=212119552
		Peak Map Virtual memory (bytes)=2305949696
		Peak Reduce Physical memory (bytes)=115908608
		Peak Reduce Virtual memory (bytes)=2315132928
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=725
	File Output Format Counters
		Bytes Written=721
[hadoop@master temp]$
```

```java
And	6
But	1
Hold	1
If	2
In	1
It's	1
Look	1
Lord	1
No	1
So	1
That	1
There's	1
When	1
With	1
You	2
a	4
afraid	1
alone	2
an	1
and	1
answer	1
anyone	1
are	2
aside	1
away	2
be	3
can	2
carry	1
cast	1
comes	1
disappear	1
don't	2
dreams	1
emptiness	1
face	1
fears	1
feel	1
felt	1
finally	1
find	2
follow	1
for	1
gone	1
hand	1
hard	1
have	1
heart	1
hero	3
hold	1
hope	1
if	1
in	1
inside	2
into	1
is	1
know	2
knows	1
let	1
lies	1
like	1
long	1
look	1
love	1
melt	1
of	1
on	1
on,	1
one	1
out	1
reach	1
reaches	1
road	1
search	1
see	1
sorrow	1
soul	1
strength	1
strong	1
survive	1
tear	1
that	1
the	6
them	1
then	1
there	1
time,	1
to	4
tomorrow	1
truth	1
way	1
what	1
when	1
will	3
within	1
world	1
you	14
you'll	2
your	3
yourself	1
```



# MapReduce源代码解析

## Hadoop Mapper定义

```java
public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {
	public abstract class Context implements MapContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> { }
	protected void setup(Context context) throws IOException, InterruptedException {
		// NOTHING
	}
	protected void map(KEYIN key, VALUEIN value,Context context) throws IOException, InterruptedException {
		context.write((KEYOUT) key, (VALUEOUT) value); 
  }
	protected void cleanup(Context context) throws IOException, InterruptedException { 
  	// NOTHING
	}
	public void run(Context context) throws IOException, InterruptedException { 			
    setup(context);
		while (context.nextKeyValue()) {
			map(context.getCurrentKey(), context.getCurrentValue(), context); 
    }
		cleanup(context); 
  }
}
```

## Hadoop Reducer定义

```java
public class Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {
	public abstract class Context implements ReduceContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> { }
	protected void setup(Context context) throws IOException, InterruptedException {
		// NOTHING
	}
	protected void reduce(KEYIN key, Iterable<VALUEIN> values, Context context ) throws IOException, InterruptedException {
		for(VALUEIN value: values) {
			context.write((KEYOUT) key, (VALUEOUT) value); 
    }
	}
	protected void cleanup(Context context) throws IOException, InterruptedException { 
    // NOTHING
	}
	public void run(Context context) throws IOException, InterruptedException { 		
    setup(context);
		while (context.nextKey()) {
			reduce(context.getCurrentKey(), context.getValues(), context); 
    }
		cleanup(context); 
  }
}
```

## Hadoop Partitioner定义和默认实现

* Partitioner抽象类定义

  ```java
  public abstract class Partitioner<KEY, VALUE> {
    public abstract int getPartition(KEY key, VALUE value, int numPartitions);
  }
  ```

* 默认HashPartitioner实现

  ```java
  public class HashPartitioner<K, V> extends Partitioner<K, V> {
    public int getPartition(K key, V value, int numReduceTasks) {
      return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
    }
  }
  ```

## Hadoop InputFormat定义和默认实现

![image-20200101213807193](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101213807193.png)

## 与InputFormat有关的类

![image-20200101213854075](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101213854075.png)

## TextInputFormat处理流程

![image-20200101213948459](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101213948459.png)

# MapReduce执行机制

## MapReduce on Yarn

![image-20200101214107417](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101214107417.png)

## MapReduce容错性

* Case 1. 如果Task运行失败
  * Map Task失败
    * MRAppMaster重启Map Task，Map Task没有依赖性
  * Reduce Task失败
    * MRAppMaster重启Reduce Task，Map Task的输出保存在磁盘上
  * 同一个Task运行多次失败(默认4次)则本次作业失败
* Case 2. 如果Task所在的Node节点挂了
  * 在另外一个节点上重启所有在挂掉节点上曾经运行过的任务
* Case 3. 如果Task运行缓慢
  * 通常由于硬件损坏、软件Bug或者配置错误导致
  * 单个task运行缓慢会显著影响整体作业运行时间
  * 解决方案:推测执行
    * 在另外一个节点上启动相同的任务，谁先完成就kill掉另外一个节点 上的任务
  * 无法启动推测执行的情况:写入数据库

## MapReduce数据本地性问题

* 在集群中网络资源是一种稀缺资源
* 文件在HDFS上存储在不同的DataNode节点上
* 如果Map Task任务从远程机器上拷贝数据会消耗大量的网络带宽

![image-20200101214920366](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101214920366.png)

![image-20200101215033248](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101215033248.png)

# 案例介绍

## 用户行为分析-PV/UV计算

* 计算思路

  ![image-20200101215221392](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101215221392.png)

* 用户行为分析-PV计算

  ```java
  public static class PVMapper extends Mapper<Object, Text, Text, IntWritable> { 
    private final static IntWritable one = new IntWritable(1);
  	private Text word = new Text();
  	public void map(Object key, Text value, Context context) throws IOException, InterruptedException { 
      String[] rawLogFields = value.toString().split("\t");
  		String accessURL = rawLogFields[1];
  		MultiMap<String> values = new MultiMap<String>();
  		UrlEncoded.decodeTo(accessURL, values, "UTF-8"); 
      String productId = values.getValue("product_id", 0); 
      if (StringUtils.isNumeric(productId)) {
  			word.set(productId);
  			context.write(word, one); 
      }
  	} 
  }
  ```

  ```java
  public static class PVReducer extends Reducer<Text, IntWritable, Text, IntWritable> { 		private IntWritable result = new IntWritable();
  	public void reduce(Text key, Iterable<IntWritable> values,
  Context context) throws IOException, InterruptedException {
  		int sum = 0;
  		for (IntWritable val : values) {
  			sum += val.get(); 
      }
  		result.set(sum);
  		context.write(key, result); 
    }
  }
  ```

* 用户行为分析-UV计算

  ```java
  public static class UVDistinctMapper extends Mapper<Object, Text, Text, NullWritable> {
  	private Text word = new Text();
  	public void map(Object key, Text value, Context context) throws IOException, 	InterruptedException { 
      String[] rawLogFields = value.toString().split("\t");
  		// Get productId
  		String accessURL = rawLogFields[1];
  		MultiMap<String> values = new MultiMap<String>(); 
      UrlEncoded.decodeTo(accessURL, values, "UTF-8"); 
      String productId = values.getValue("product_id", 0); 
      // Get userId
  		String cookie = rawLogFields[3];
  		String userId = cookie.contains("uid=") ? cookie.split("=")[1] : null;
  		if (StringUtils.isNumeric(productId) && StringUtils.isNotEmpty(userId)) {
  			word.set(productId + "\t" + userId);
  			context.write(word, NullWritable.get()); }
  		} 
  }
  ```

  ```java
  public static class UVDistinctReducer extends Reducer<Text, NullWritable, Text, NullWritable> {
  	public void reduce(Text key, Iterable<NullWritable> values,
  Context context) throws IOException, InterruptedException {
  		context.write(key, NullWritable.get()); 
    }
  }
  ```

  ```java
  public static class UVCountMapper extends Mapper<Object, Text, Text, IntWritable> { 		
    private final static IntWritable one = new IntWritable(1);
  	private Text word = new Text();
  	public void map(Object key, Text value, Context context) throws IOException, InterruptedException { 
      String[] dataFields = value.toString().split("\t");
  		word.set(dataFields[0]);
  		context.write(word, one);
  	} 
  }
  ```

  ```java
  public static class UVCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> { 
    private IntWritable result = new IntWritable();
  	public void reduce(Text key, Iterable<IntWritable> values,
  Context context) throws IOException, InterruptedException {
  		int sum = 0;
  		for (IntWritable val : values) {
  			sum += val.get(); 
      }
  		result.set(sum);
  		context.write(key, result); 
    }
  }
  ```

# 调优参数

## Map Task和Reduce Task数目调整

* Map Task数目

  * Map读取文件时，通过InputFormat计算分割文件

  * split大小由以下三个参数决定

    * dfs.blocksize HDFS Block大小

    * mapreduce.input.fileinputformat.split.minsize 划分最小字节数

    * mapreduce.input.fileinputformat.split.maxsize 划分最大字节数

    * 计算公式

      ```java
      protected long computeSplitSize(long blockSize, long minSize, long maxSize) { 	
        return Math.max(minSize, Math.min(maxSize, blockSize));
      }
      ```

* Reduce Task数目
  * 默认每个作业Reduce Task数目可以通过mapreduce.job.reduces控制
  * 在每个作业中也可以通过Job.setNumReduceTasks(Int number)进行控制

## 容错参数调整

![image-20200101220238167](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101220238167.png)

## 推测执行参数调整

![image-20200101220321288](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101220321288.png)

## 内存参数调整

![image-20200101220356207](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101220356207.png)

# 课后作业

* Word Count Example运行
* 阅读MapReduce TextInputFormat源代码
* 实现PV和UV计算
  * 使用maven构建项目
  * 实战了解Mapper、Reducer
  * 打包提交CDH集群运行并输出结果
* 实现TopK(例如:最热门的N个商品)
  * 思路
  * 完整的工程
* 思考如何解决数据倾斜问题?
* 如何使用MR实现矩阵相乘?

