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

```shell
http://archive.cloudera.com/cdh5/cdh/5/
Apache Hadoop 2.6.0-cdh5.15.1
http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.1.tar.gz
```

![image-20200120212243936](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120212243936.png)

![image-20200120213408832](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120213408832.png)

![image-20200120213256412](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120213256412.png)

```shell
export JAVA_HOME=/home/hadoop/app/jdk1.8.0_191
export PATH=$JAVA_HOME/bin:$PATH
```

![image-20200120213829237](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120213829237.png)

![image-20200120220555663](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120220555663.png)

![image-20200120224817787](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120224817787.png)

```shell
# http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.1/hadoop-project-dist/hadoop-common/SingleCluster.html
vi /home/hadoop/app/hadoop-2.6.0-cdh5.15.1/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_161

etc/hadoop/core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>

etc/hadoop/hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>


[hadoop@hadoop000 app]$ mkdir tmp
[hadoop@hadoop000 app]$ pwd
/home/hadoop/app/tmp

vi hdfs-site.xml
<property>
	<name>hadoop.tmp.dir</name>
	<value>/home/hadoop/app/tmp</value>
</property>

vi slaves 
hadoop000

# 启动HDFS：第一次启动时要格式化
hdfs namenode -format
# Storage directory /home/hadoop/app/tmp/dfs/name has been successfully formatted
```

![image-20200120222014164](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120222014164.png)

```shell
# 注意权限，谁上传谁解压谁配置谁执行-hadoop用户
start-dfs.sh
[hadoop@hadoop000 hadoop]$ jps
6451 DataNode
6331 NameNode
6715 Jps
6605 SecondaryNameNode

# 访问
http://192.168.0.200:50070/
# jps ok，但是浏览器不ok，查看防火墙是否关闭
```

![image-20200120224924359](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120224924359.png)

![image-20200120230152767](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200120230152767.png)

```shell
# 递归查看
hadoop fs -ls -R /test/dir
http://192.168.0.200:50070/explorer.html#/
```

```shell
# 问题
Exception in thread "main" java.net.ConnectException: Call From Jie.local/127.0.0.1 to 192.168.0.200:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
# 解决
netstat -tpnl   #检查主节点9000端口是否打开，且允许远程访问
vi hdfs-site.xml
		<property>
        <name>dfs.namenode.rpc-bind-host</name>
        <value>0.0.0.0</value>
    </property>
# 重启虚拟机
# 解决思路
1. 上官网查看配置
https://hadoop.apache.org/docs/r3.1.2/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
# dfs.namenode.rpc-bind-host
# This is useful for making the name node listen on all interfaces by setting it to 0.0.0.0
```

![image-20200126175737953](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200126175737953.png)

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.net.URI;

public class HDFSApp {
    public static void main(String[] args) throws Exception{
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.0.200:9000"), configuration, "hadoop");
        Path path = new Path("/hdfsapi/test1");
        boolean result = fileSystem.mkdirs(path);
        System.out.println(result);
    }
}
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.util.Progressable;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.net.URI;

public class HDFSTest {

    public static final String HDFS_PATH = "hdfs://192.168.0.200:9000";
    FileSystem fileSystem = null;
    Configuration configuration = null;

    @Before
    public void setUp() throws Exception {

        System.out.println("---------setUp-----------");

        configuration = new Configuration();
        // 设置副本数
        configuration.set("dfs.replication", "1");

        fileSystem = FileSystem.get(new URI(HDFS_PATH), configuration, "hadoop");


    }

    /**
     * 创建HDFS文件夹
     * @throws Exception
     */
    @Test
    public void mkdir() throws Exception {
        fileSystem.mkdirs(new Path("/20200126"));
    }

    /**
     * 查看HDFS内容
     * @throws Exception
     */
    @Test
    public void text() throws Exception {
        FSDataInputStream in = fileSystem.open(new Path("/cdh_version.properties"));
        IOUtils.copyBytes(in, System.out, 1024);
    }

    @Test
    public void create() throws Exception {
        FSDataOutputStream out = fileSystem.create(new Path("/hdfsapi/test/a.txt"));
        out.writeUTF("hello woody");
        out.flush();
        out.close();
    }

    @Test
    public void testReplication() {
        System.out.println(configuration.get("dfs.replication"));
    }

    @Test
    public void testRename() throws Exception {
        Path oldPath = new Path("/hdfsapi/test/b.txt");
        Path newPath = new Path("/hdfsapi/test/a.txt");
        boolean result = fileSystem.rename(oldPath, newPath);
        System.out.println(result);
    }

    @Test
    public void copyFromLocalFile() throws Exception {
        Path src = new Path("/Users/dingyuanjie/Downloads/镇魂.pdf");
        Path dst = new Path("/hdfsapi/test");
        fileSystem.copyFromLocalFile(src, dst);
    }

    @Test
    public void copyFromBigLocalFile() throws Exception {
        InputStream in = new BufferedInputStream(new FileInputStream(new File("/User/test/bigFile.data")));

        FSDataOutputStream out = fileSystem.create(new Path("/hdfsapi/test/"), new Progressable() {
            public void progress() {
                System.out.print(".");
            }
        });

        IOUtils.copyBytes(in, out, 4096);
    }

    @Test
    public void copyToLocalFile() throws Exception {
        Path src = new Path("/hdfsapi/test/hello.txt");
        Path dst = new Path("/User/dyj/data/");
        fileSystem.copyToLocalFile(src, dst);
    }

    /**
     * 查看目标文件夹下的所有文件
     */
    @Test
    public void listFiles() throws Exception {
        FileStatus[] statuses = fileSystem.listStatus(new Path("/hdfsapi/test"));
        for(FileStatus file : statuses) {
            String isDir = file.isDirectory() ? "文件夹" : "文件";
            String permission = file.getPermission().toString();
            short replication = file.getReplication();
            long len = file.getLen();
            String path = file.getPath().toString();

            System.out.println(isDir + "\t" + permission + "\t"
                    + replication + "\t" + len + "\t" + path);
        }
    }

    /**
     * 递归查看
     * @throws Exception
     */
    @Test
    public void listFilesRecursive() throws Exception {
        RemoteIterator<LocatedFileStatus> files = fileSystem.listFiles(new Path("/hdfsapi/test"), true);
        while (files.hasNext()) {
            LocatedFileStatus file = files.next();
            String isDir = file.isDirectory() ? "文件夹" : "文件";
            String permission = file.getPermission().toString();
            short replication = file.getReplication();
            long len = file.getLen();
            String path = file.getPath().toString();

            System.out.println(isDir + "\t" + permission + "\t"
                    + replication + "\t" + len + "\t" + path);
        }
    }

    /**
     * 查看文件块信息
     */
    @Test
    public void getFileBlockLocations() throws Exception {
        FileStatus fileStatus = fileSystem.getFileStatus(new Path("/hdfsapi/test/jdk.tar.gz"));
        BlockLocation[] blocks = fileSystem.getFileBlockLocations(fileStatus, 0, fileStatus.getLen());
        for(BlockLocation block : blocks) {
            for(String name : block.getNames()) {
                System.out.println(name + ":" + block.getOffset() + ":" + block.getLength());
            }
        }
    }

    @Test
    public void delete() throws Exception {
        boolean result = fileSystem.delete(new Path("/hdfsapi/test"), true);
        System.out.println(result);
    }

    @After
    public void tearDown() {
        configuration = null;
        fileSystem = null;
        System.out.println("---------tearDown-----------");
    }
}
```

![image-20200126180753469](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200126180753469.png)

![image-20200126180921619](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200126180921619.png)

```shell
hadoop fs -put /home/hadoop/app/hello.txt /hdfsapi/test/
hadoop fs -ls -R /hdfsapi
hadoop fs -rm -r /hdfsapi/test/hello.txt

hadoop fs -cat /hdfsapi/test/hello.txt
hadoop fs -text /hdfsapi/test/hello.txt
	hello world welcome
	hello welcome
```

![image-20200126225448771](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200126225448771.png)

![image-20200126230157627](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200126230157627.png)

![image-20200126231045603](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200126231045603.png)

# MapReduce

* 分布式计算框架

## 概述

* 优点：海量数据离线处理&易开发&易运行 
* 缺点：实时流式计算

![image-20200127121445704](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200127121445704.png)

![image-20200127122029666](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200127122029666.png)

![image-20200127125403843](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200127125403843.png)

![image-20200127125523446](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200127125523446.png)

