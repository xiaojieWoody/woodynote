# todo

```shell
# day01
# 完成简单rpc小demo，可看当节视频，着重rpctest模块中的DoitUtil.java
# 看下代码，迭代器这块

# day02
# 安装四台虚拟机，并且都安装HDSF
```

# Java基础

## 序列化

* 序列化就是把Java对象编码成一串二进制的过程

  * 作用：将对象放进文件存储、或者在网络上传输

* 反序列化就是一个解码的过程

* 序列化没有固定的标准，有各种各样序列化的方法

  * jdk自带，标记：Serializable

    ```java
    User u = new User(“1”,18)
    ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(“d:/u.obj”))
    // 序列化
    out.writeObject(u)
    // 反序列化
    ObjectInputStream in = new ObjectInputStream(new FileInputStream(“d:/u.obj”))
    Object obj = in.readObject();
    ```

  *  自定义序列化机制：

    1. 将对象变成逗号连接的字符串再编码 

       ```java
       User u = new User(“zs”,18)
       // 序列化
       String strU = u.toString();  //  zs,18
       FileOutputStream out = new FileOutputStream(“d:/u.obj”);
       out.write(strU.getBytes());
       
       // 反序列化
       BufferedReader br = new BufferedReader(new FileReader(“d:/u.obj”));
       String line = br.readLine();
       String[] split = line.split(“,”)
       User u = new User();
       u.setName(split[0])
       u.setAge(Integer.parseInt(split[1]))
       ```

    2. 将对象变成json字符串再编码

       ```java
       // 序列化
       User u = new User(“zs”,18);
       Gson gson = new Gson();
       String json = gson.toJson(u);    // {“name”:”zs”,”age”:18}
       FileOutputStream out = new FileOutputStream(“d:/u.obj”);
       out.write(json.getBytes());
       
       // 反序列化
       BufferedReader br = new BufferedReader(new FileReader(“d:/u.obj”));
       String json = br.readLine();
       Gson gson = new Gson();
       User u = gson.fromJson(json,User.class)
       ```

    3. 将对象中的每一个属性按照它原来的类型进行逐个编码

       * DataOutputStream、DataInputStream——可以对基本数据类型按类型机制编解码

         ```java
         // 序列化
         User u = new User(“zs”,18);
         DataOutputStream dout = new DataOutputStream(new FileOutputStream(“d:/u.obj”))
         dout.writeUTF(u.getName());
         dout.writeInt(u.getAge());
         
         // 反序列化
         DataInputStream  din = new DataInputStream(new FileInputStream(“d:/u.obj”));
         User u = new User();
         u.setName(din.readUTF());
         u.setAge(din.readInt());
         ```

  * Hadoop内部也有一套自己的序列化框架：提供了一个接口Writable
    * 类只要实现了Writable接口，hadoop内部就能用自己的序列化机制去序列化这个类的对象
  * 跨平台的序列化机制：
    * avro、thrift

## 服务概念

* 就是一个业务功能，变成可以通过网络进行请求

* 核心要素

  * 必须有一个能接收网络请求的服务端
  * 要有一个业务功能类
  * 要有一套请求方和被请求方约定好的协议
  * 要有一个支持该协议的客户端

* 自定义socket服务

  * 服务端：应该有一个绑定特定端口的服务器程序
  * 客户端：知道服务端的地址和端口就可以请求
  * 双方应该约定通信协议

  ```java
   // 接收请求的服务端
  ServerSocket ss = new ServerSocket(6666);
  while(true){
  	Socekt sc = ss.accept();  // 等待客户端请求，并为请求建立通信连接
  	InputStream in = c.getInputStream();  // 构造一个读取连接中发送过来的数据的流
    byte[] b = new byte[1024];
  	in.read(b);   // 读取发送过来的请求的数据
  	// 根据请求的数据去调用相应的业务类的业务方法，并拿到业务处理结果
  	// 将处理结果通过socket的输出流向客户端返回；
  }
  ```

* web服务（restApi）：

  * 本质上还是socket通信；

  * 关键点在于，通信的协议是使用了一套全球通用的规范协议：HTTP

  * 所以，服务端需要能够解析HTTP请求，能够按这种协议封装响应

  * 客户端需要能够封装HTTP协议的请求，能够解析HTTP协议的响应；

  * HTTP请求协议头：

    ```http
    GET / HTTP/1.1
    Host: localhost:8080
    Connection: keep-alive
    Cache-Control: max-age=0
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
    Accept-Encoding: gzip, deflate, br
    Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
    Cookie: Hm_lvt_b393d153aeb26b46e9431fabaf0f6190=1535981983,1536045585
    ```

  * HTTP响应协议头

    ```http
    HTTP/1.1 200 OK --响应行   
    Server: Apache-Coyote/1.1 --响应头（key-vaule）   
    Content-Length: 24   
    Date: Fri, 30 Jan 2015 01:54:57 GMT   
    --一个空行   
    <html>………</html> --响应正文/实体内容
    ```

  * 数据库服务：比如MySql，它就是一个服务端程序，监听3306端口，接收JDBC协议规范的请求

  * RPC（远程过程调用）服务：

    * 这种服务本质上也是socket通信
    * 不过在使用形式上，客户端有一个特点：看起来就像在调用本地的方法；

## 进程与线程

* 进程：操作系统级别的，程序运行空间（启动一个程序，就是一个进程；再起一个，又是一个进程）
* 线程：程序内部的执行空间（线程栈）；

## 文件格式

* 文件格式，在最底层看来，都是一样的：二进制
* 上升一层（编码）：就可以有不同的编码
* 上升一层（数据组织结构）：数据组织结构不同
* SequenceFile  —>  kv存储，Hadoop
* ParquetFile  —>  列式存储 ，apache

## 迭代器

* 迭代器的意义
  * 让使用者可以不了解数据存储的细节的情况下，也能方便地取到数据
* 迭代器有一个通用的接口： Iterator
  * 还可以再封装一层，实现另一个通用的接口：Iterable
  * Iterable类可以通过iterator()方法获取一个Iterator对象，进行迭代
  * 而且可以直接使用增强for循环进行迭代

## 作业题

* 请开发一套分布式系统

  1. 客户端可以将文件传入你的系统；
  2. 客户端传入的文件，可能存在任一一台机器上
  3. 客户端将来还能通过你的系统取到完整的文件

  ![image-20191126103927714](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126103927714.png)

# 大数据导论

## 什么是大数据

* 大数据是对海量数据进行处理的技术体系的通称
* 大数据技术体系的鲜明特点（更快处理更多数据）：
  * 跟传统技术最大的区别：大数据技术体系都是用分布式并行处理来设计的
* 大数据技术体系有大量的现成的框架可用：
  * 海量文件分布式存储：HDFS
  * 海量数据分布式运算：MAPREDUCE /SPARK/STORM/FLINK.....
  * 海量数据分布式数据库：HBASE
  * 海量数据分布式消息缓存系统：KAFKA
  * 这些框架的共同特征是：分布式处理

## 大数据场景

* 各行业（移动运营商、电力、交通、互联网、医疗…..）在信息化时代，产生了日益增多的海量数据，然后，各行业可以对这些数据进行挖掘、分析，获取数据价值，而现有的mysql、oracle等传统数据库，无法存储如此大量的数据，而现有的java单机程序，无法对如此海量的数据进行快速运算处理，所以，在此需求出现的情况下，业界产生了一整套应对海量数据存储、运算的技术体系——大数据技术
* 电商：分析用户浏览、购物行为，以用于智能推荐等
* 社交类：分析用户属性、社交关系等，用于智能推荐好友，兴趣圈...，挖掘人群社交规律
* 地图类：路线推荐、耗费时间估算。。。。
* 金融类：分析用户的金融属性（风险、信用），以用于更好地开展金融业务
* 电信类：流量数据分析，以用于更好地开展电信业务；还可以出售数据

## 大数据核心技术

* 分布式存储
* 分布式运算
* 分布式：利用大量机器来协同工作，实现存储、运算
* ==大数据开发：最锻炼核心编程思想和熟练度==
  * ==因为：框架相对较少，编程的领域很集中：数据处理==

## 大数据的学习路线和前景规划

* Linux
* hadoop生态体系（mapreduce hdfs yarn ；  hive hbase flume sqoop ……）
* spark生态体系（spark core，  spark streaming ， spark sql ， spark graphx ， spark mllib ， kafka）
* flink实时计算的新秀
* 项目：广告、通信运营、web流量运营、电商推荐、新闻推荐；
* kylin、clickHouse  —>  OLAP框架

## Hadoop简介

* 应用情况
  * HADOOP中的HDFS基本上是大数据存储中的首选技术
  * HADOOP中的mapreduce基本上已经被淘汰；替代者：spark 、flink
  * HADOOP中的YARN，基本上是大数据系统中最通用的资源调度平台
  * 上层：HIVE依然是大数据体系中的数据仓库建设工具首选
  * HBASE在一些大型企业中还是有不少应用
* 内部核心组件
  * 分布式存储： HDFS
  * 分布式运算： MAPREDUCE
  * 资源调度系统：YARN
* 外围组件（子项目）
  * HIVE   HBASE  FLUME(数据采集)  SQOOP（数据迁移）

# HDFS

## 简介

* hadoop distributed File System

* HDFS文件系统：能够利用大量机器分布式存储海量的文件，但是又需要对外提供一个整体的感觉
  * 有统一的目录结构：/
  * 文件不以整体进行物理存储（目的：便于并行计算）
* 是一个分布式文件系统（整个系统中有多种角色<namenode、datanode、客户端、secondary namenode>，共同协作完成文件系统的功能）
* 功能：提供一个目录结构，顶层目录为：/
* 可以：创建文件夹、删除文件或文件夹、重命名文件、列出文件夹下的文件（涉及元数据操作）、保存文件、读取文件等（涉及元数据操作、文件块读写）
* 特点
  * 可以存储海量的文件，如果容量不够，添加服务器（data node）即可
  * 文件被分散存储在若干台datanode服务器上（存储目录中）
  * 一个文件也可能被切分成多个文件块（block块）分散存储在若干台datanode服务器
  * 每一个文件（文件块）在整个集群中，可以存储多个副本
  * （一个文件存几个副本、一个文件按多大来切块，是由客户端决定？）
* hdfs的运作机制：
  * 客户端存入的文件，一方面由datanode存储文件内容（block），另一方面由namenode记录文件的块信息（？块，？副本，在哪些dn上）

## 核心设计思想

* 系统中有一个namenode服务器，用来维护一个统一的虚拟目录结构，并记录每一个文件的元数据（文件名、文件总大小、文件分了几个块，每个块在哪些机器上，块的ID……..）
* 系统中有大量的datanode服务器，用来存储用户的文件的物理的块！并帮用户读、存；

![image-20191126143139112](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126143139112.png)

## 集群搭建

* hdfs的配置项，[官网配置文件地址](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/core-default.xml)

  ```shell
  #1. namenode的地址：    主机名：端口号
  dfs.namenode.rpc-address   nn1.hdp:9000    #RPC请求端口
  dfs.namenode.http-address  0.0.0.0:50070   #web请求端口
  #2. datanode的文件块存储路径
  dfs.datanode.data.dir    /opt/hdpdata/data/
  #3. namenode的元数据序列化文件存储路径
  dfs.namenode.name.dir    /opt/hdpdata/name/
  ```

* 安装详细步骤

  1. 准备4台linux机器

     * 装jdk

       ```shell
       # 上传jdk安装包并解压
       # 修改linux的环境变量：/etc/profile
       export JAVA_HOME=/opt/jdk/jdk1.8.0_60/
       export PATH=$PATH:$JAVA_HOME/bin
       ```

     * 配置好IP地址(`vi /etc/sysconfig/network-scripts/ifcfg-eth0`)

     * 主机名(`vi /etc/sysconfig/network`)

     * 域名映射(`vi /etc/hosts`)

     * 关闭防火墙(`service iptables stop;  chkconfig iptables off`)

       ```shell
       nn1.hdp   192.168.33.91
       dn1.hdp   192.168.33.92
       dn2.hdp   192.168.33.93
       dn3.hdp   192.168.33.94
       # 互相是否ping通
       ```

  2. 上传hadoop安装包并解压

     * 创建一个安装大数据软件的目录：`mkdir /usr/local/bgdt`
     * `tar -zxf hadoop-2.8.5.tar.gz -C /usr/local/bgdt/`

  3. 修改配置文件 ${HADOOP_HOME}/etc/hadoop/ 下

     * hadoop-env.sh  配置jdk HOME

       ```shell
       # which java     #/opt/jdk/java/..  
       vi  hadoop-env.sh
       export JAVA_HOME= /opt/jdk/
       ```

     * hdfs-site.xml  配置上文中所述的3个核心参数

       ```xml
       <configuration>
       	<property>
       		<name>dfs.namenode.rpc-address</name>
       		<value>nn1.hdp:9000</value>
       	</property>
       	<property>
       		<name>dfs.namenode.name.dir</name>
       		<value>/opt/hdpdata/name</value>
       	</property>
         <property>
       		<name>dfs.datanode.data.dir</name>
       		<value>/opt/hdpdata/data</value>
       	</property>
       	<property>
         	<name>dfs.namenode.secondary.http-address</name>
       	  <value>dn1.hdp:50090</value>
       	</property>
       </configuration>
       ```

  4. 复制配置好的安装包到其他机器

     * 某台机器配置好后，可以直接将这个安装包发送给其他几台机器去安装

     * 为了加快传输速度，可将安装包内的doc文件删除

       ```shell
       #hadoop-2.8.5目录下
       cd share/
       ll
       rm -rf doc
       ```

       ```shell
       scp -r  ./bgdt    dn1:$PWD
       scp -r  ./bgdt    dn2:$PWD
       scp -r  ./bgdt    dn3:$PWD
       ```

     * 复制文件

       ```shell
       for i in {1..3}; do scp /etc/profile dn${i}.hdp:/etc/;done
       ```

       ![image-20191126000913342](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126000913342.png)

  5. 启动HDSF

     * 先执行jps命令，将其他线程干掉，kill -9 xxx

     * 在nn1.hdp上启动namenode：

       * 第一次启动namenode，需要初始化它的元数据存储目录：`bin/hadoop namenode -format`
       * 启动：`sbin/hadoop-daemon.sh start namenode`

     * 在另外3台机器上分别启动datanode

       * `sbin/hadoop-daemon.sh start datanode`

     * 执行jps查看是否启动成功，出现 NameNode或DataNode则成功

       ```shell
       # 查看启动端口
       netstat -nltp|grep 27910
       # 外部访问
       虚拟主机ip:50070
       ```

       ![image-20191126001249384](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126001249384.png)

       * 失败则查看日志

         ```shell
         # hadoop-2.8.5目录
         ll
         cd logs
         ll
         less hadoop-root-namenode-nn1.hdp.log
         ```

       * 失败后重新启动

         ```shell
         # 启动失败，删除启动目录
         rm -rf /opt/hdpdata/data/
         # 重新启动
         [root@dn2 hadoop-2.8.5]# sbin/hadoop-daemon.sh start datanode
         ```

       * 修改clusterID

         ```shell
         hadoop-daemon.sh stop namenode
         cd /opt/hdpdata/name
         ll
         cd current/
         ll
         vi VERSION
         # 修改ClusterId
         ```

         ![image-20191126001900307](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126001900307.png)

  6. 启动完成后，可以用浏览器来访问namenode或datanode

     * namenode:    http://nn1.hdp:50070
     * datanode:   http://dn1.hdp:50075

  7. 一台一台手动起，太麻烦，可以通过shell脚本自动去启动整个集群

     ```shell
     #!/bin/bash
     ssh nn1.hdp "/usr/local/bgdt/hadoop-2.8.5/sbin/hadoop-daemon.sh start namenode"
     for i in {1..3}
     do
     ssh dn${i}.hdp "/usr/local/bgdt/hadoop-2.8.5/sbin/hadoop-daemon.sh start datanode"
     done
     ```

     * 当然，HADOOP软件中已经有写好的自动启动脚本:

       ```shell
       # 启动脚本
       sbin/start-dfs.sh        
       # 停止脚本
       sbin/stop-dfs.sh
       # 只不过这个脚本，不光会起namenode，datanode，还会起一个secondary namenode进程
       #而且，这个脚本需要知道我们想在哪里启动datanode： ${HADOOP_HOME}/etc/hadoop/slaves
       # 要用这个脚本，需要配置nn1.hdp 到nn1.hdp / dn1.hdp / dn2.hdp / dn3.hdp的免密登陆
       ```

  8. 配置环境变量

     ```shell
     echo $PATH
     pwd
     vi /etc/profile
     	export JAVA_HOME=/opt/jdk
     	export HADOOP_HOME=/usr/local/bgdt/hadoop-2.8.5
     	export PATH=.:$PATH:$HADOOP_HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/sbin
     source /etc/profile
     # 例子
     # 添加可执行权限
     chmod +x helloworld.sh
     # 当前目录 helloworld.sh就会直接执行，因为在环境变量中配置了在当前目录下查找并执行(.)
     ```

     ```shell
     ssh-copy-id nn1.hdp
     ```

     ![image-20191126002907877](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126002907877.png)

  * 安装常见问题

    ```shell
    #主机名：不能带下划线 “_” 
    症状：启动失败，报错：there is not valid host:port 无效的主机名和端口号
    # 输入hadoop命令，报错：command not found
    原因是：linux查找命令执行的搜索机制,可以把hadoop命令所在的目录配入PATH环境变量,或者指定路径来执行这个命令
    # 第一次启动前忘了初始化namenode的元数据目录
    第一次启动前忘了初始化namenode的元数据目录,storage directory does not exist or is not accessible
    # 防火墙忘了关
    症状：网页无法连接,或者namenode不识别datanode
    # 启动datanode失败
    datanode上的clusterID与namenode上的clusterID不一致
    原因：namenode再次被format后，生成了一个新的clusterID，导致与原来的datanode上的clusterID不一致
    ```

  ```xml
  <!--hadoop配置-->
  ------core-site.xml----------
  <property>
  	<name>fs.defaultFS</name>
  	<value>hdfs://nn1.hdp:9000/</value>
  </property>
  ------hdfs-site.xml----------
  <property>
  	<name>dfs.namenode.rpc.address</name>
  	<value>nn1.hdp:9000</value>
  </property>
  <property>
  	<name>dfs.namenode.name.dir</name>
  	<value>/opt/hdpdata/name/</value>
  </property>
  <property>
  	<name>dfs.datanode.data.dir</name>
  	<value>/opt/hdpdata/data/</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>dn1:50090</value>
  </property>
  ------yarn-site.xml----------
  <property>
  	<name>yarn.resourcemanager.hostname</name>
  	<value>nn1.hdp</value>
  </property>
  <property>
  	<name>yarn.nodemanager.aux-services</name>
  	<value>mapreduce_shuffle</value>
  </property>
  ------mapred-site.xml----------
  <property>
  	<name>mapredcue.framework.name</name>
  	<value>yarn</value>
  </property>
  ```

  

## 访问方式及命令行常用操作命令

* 访问方式

  * namenode 提供的WEB页面（web console）

  * **命令行客户端**

    * 所谓的命令行客户端，是hadoop软件中自带的一个客户端程序，通过命令的形式来使用客户端命令是：(hadoop)  hdfs dfs

      * 客户端程序改在哪里运行呢？
      * 客户端程序可以在任何机器上运行（集群内的机器、集群外的机器、linux、windows、mac），前提是，那个机器上安装了hadoop软件

    * 注意

      * hadoop的命令行客户端，可以访问各种文件系统：本地文件系统、HDFS文件系统、腾讯的分布式文件系统

      * 当你直接  hdfs  dfs  -ls  /，它就不知道你要访问的是哪一个文件系统了！！！默认访问的是本机的本地文件系统！

      * 如果要访问一个指定的文件系统，需要在目标路径前加上这个文件系统的URI：

        * http://www.taobao.com:80/abc.html
        * jdbc:mysql://xx:3306/db1
        * hdfs://nn1.hdp:9000/

      * 如果要让hdfs客户端默认就访问我们想指定的文件系统，那么就在core-site.xml配置文件中，覆盖一个参数：

        * fs.defaultFS = file:///     改成    fs.defaultFS = hdfs://nn1.hdp:9000/

          ```xml
          <!--core-site.xml-->
          <property>
            <name>fs.defaultFS</name>
            <value>hdfs://nn1.hdp:9000/</value>
          </property>
          ```

* 文件操作命令

  ```shell
  #文件夹：
  #创建文件夹：
  hdfs dfs -mkdir /abc
  hdfs dfs -mkdir -p /xx/yy
  #删除文件夹
  hdfs dfs -rm -r /xx
  #上传文件：
  hdfs dfs -put /本地路径  /hdfs路径
  #下载文件：下载到了本地的当前路径下
  hdfs dfs -get /abc/hadoop-2.8.5.tar.gz
  #移动文件
  hdfs dfs -mv /abc/hadoop-2.8.5.tar.gz  /
  #拷贝文件：
  hdfs  dfs  -cp  /hadoop-2.8.5.tar.gz  /abc/hdp.tar.gz
  #读文件内容：
  hdfs dfs -cat /a.txt
  hdfs dfs -tail /a.txt
  hdfs dfs -tail -f /a.txt
  #追加内容到已存在的文件：
  hdfs dfs -appendToFile ./2.txt /a.txt
  #下载多个文件并在本地合并成一个文件：
  hdfs dfs -getmerge /*.txt ./big.txt
  #修改指定文件的副本数量：
  hdfs dfs -setrep 1 /a.txt
  # hdfs dfs客户端的命令大全：
  ```

* HDFS运行机制的观察
  1. 如果强行进入datanode的数据存储目录，删掉一个block块副本
     * 发现：过一段时间后，namenode会指派拥有这个block副本的一台danode，将这个副本复制给另一台datanode，以恢复副本数量！
  2. 如果将一个副本数量为3的文件，通过 setrep 命令将副本数量降低为2
     * 发现：namenode会指派拥有这个文件的block的某些datanode，删除掉每一个block的一个副本
  3. 客户端上传到hdfs中的文件，确实被分块存储，而且是存在不同datanode服务器的data存储目录中
  4. 如果文件的某个block副本全部丢失，namenode会发现，并在web console上汇报
     * 解决办法： hdfs  dfs  -rm  /这个坏了的文件
* 常用运维命令
  * namenode的安全模式管理：
    * namenode如果发现丢失的block数量超过0.1%时，会自动进入安全模式
  * namenode也会自动退出安全模式；
    * 如果namenode自动退出安全模式没有成功，可以用运维命令，强行退出安全模式：
    * 强行进入安全模式：`hdfs dfsadmin -safemode enter`
    * 强行退出安全模式：`hdfs dfsadmin -safemode leave`

## HDFS客户端API编程

```java
// 构造一个参数对象
Configuration conf = new Configuration();
conf.set("fs.defaultFS", "hdfs://nn1.hdp:9000/");
conf.setInt("dfs.blocksize", 134217728);
conf.setInt("dfs.replication", 3);
// 通过设置当前JVM系统环境变量来指明客户身份
System.setProperty("HADOOP_USER_NAME", "root");
// 获取HDFS写好的客户端对象
FileSystem fs = FileSystem.get(conf);
// 调这个客户端对象的方法上传文件
fs.copyFromLocalFile(new Path("d:/dataser.obj"), new Path("/abc/"));
// 关闭客户端对象
fs.close();
```

* 文件、文件夹操作

  ```java
  // 上传：
  fs.copyFromLocalFile(new Path("d:/dataser.obj"), new Path("/abc/"));
  
  // 下载
  // 调用本地库来写本地文件的方式
  fs.copyToLocalFile(new Path("/abc/dataser.obj"), new Path("e:/"));
  // 调用java原生api来写本地文件的方式
  fs.copyToLocalFile(false, new Path("/abc/dataser.obj"), new Path("e:/"), true);
  // 关闭客户端对象
  fs.close();
  /**
  *hdfs客户端在往本地操作系统写文件时，会调用本地库，而如果这个程序在windows上，就需要调针对windows编译出来的本地库，需要：
  1.在windows上解压一份hadoop安装包
  2.将这个安装包路径配置到windows的环境变量中：HADOOP_HOME
  3.将老师给的根据windows编译出来的bin中所有文件，覆盖掉你解压的hadoop安装目录中的bin的文件；
  4.重启eclipse
  **/
  
  // 删除
  // 参数1：目标文件路径  参数2：是否递归删除
  fs.delete(new Path("/abc"), true);
  fs.close();
  
  // 判断是否存在
  boolean exists = fs.exists(new Path("/duanwangye/hive.2"));
  System.out.println(exists);
  fs.close();
  
  // 判断路径类型
  boolean directory = fs.isDirectory(new Path("/duanwangye"));
  System.out.println(directory);
  boolean file = fs.isFile(new Path("/duanwangye"));
  System.out.println(file);
  fs.close();
  
  // 移动或改名
  fs.rename(new Path("/hive.2"), new Path("/duanwangye/hive.2"));
  fs.close();
  
  // 查看目录信息
  // 只返回文件信息
  RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
  // 返回文件和文件夹信息
  FileStatus[] ls = fs.listStatus(new Path("/"));
  ```

* 文件内容读写

  ```java
  // 读文件内容
  FSDataInputStream in = fs.open(new Path("/qingshu.txt"));
  // 写内容到文件
  //创建一个新文件写入内容
  FSDataOutputStream hdfsOut1 = fs.create(new Path("/jiaocheng.txt"));
  //往一个已存在的文件中追加内容
  FSDataOutputStream hdfsOut2 = fs.append(new Path("/qingshu.txt"));
  ```

## HDFS客户端编程案例(1)

1. 需求

![image-20191126150759980](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126150759980.png)

2. 设计
   1. 客户端程序应该位于web服务器上（因为要读取服务器本地磁盘上的日志文件）
   2. 采用定时采集的策略（每隔1小时采集一次）
   3. 采集前应该先探测一下日志目录中需要采集的文件
   4. 然后将这一批待采集文件移动到一个临时目录（待上传目录）
   5. 然后读取待上传目录中的日志文件，逐个文件上传到HDFS
   6. 然后将上传完的这一批文件，移动到一个备份目录
   7. 定期清理过期的备份日志

## HDFS客户端编程案例(2)

1. 需求

   ![image-20191126152236804](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126152236804.png)

## HDFS核心运行机制

* NameNode元数据管理

  1. namenode的元数据存储位置

     * 有两个地方：
       * 内存，元数据对象（树）
       * 磁盘，fsimage

  2. checkpoint操作：secondarynamenode会定期从namenode上下载一批edits操作日志文件，然后在自己的本地将fsimage和这一批新edits进行状态合并，生成一个新版的fsimage，然后再上传给namenode；

     ![image-20191126152632552](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126152632552.png)

* 客户端读数据流程

  ![image-20191126152720857](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126152720857.png)

* 客户端写数据流程

  ![image-20191126152746903](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126152746903.png)

# MapReduce

* 是一个用于开发分布式计算程序的框架

## 基本工作机制：

* 框架内有几个核心程序：
  * JobSubmitter、MRAppMaster、MapTask、ReduceTask

## 架构

![image-20191126153520063](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126153520063.png)

## maptask和reducetask工作流程

![image-20191126153843919](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126153843919.png)

## **MAP REDUCE 入门程序示例**

* 需求：wordcount

* 整体逻辑流程

  ![image-20191126154015707](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126154015707.png)

## **MAP REDUCE 程序运行方式**

* mapreduce运行的主要流程
  * 客户端job对象，将我们的程序jar包，发给yarn，然后再向yarn中resourcemanager申请一个app容器（容器位于nodemanager服务器中）
  * 然后，job对象会往这台nodemanager发送一条启动MRAppMaster程序的shell命令；
  * 然后这个NodeManager的指定容器中，就运行起MRAppMaster
  * MRAppmaster启动后，会总控mapreduce程序的运行
* 运行步骤
  1. 将程序jar包上传到任意一台linux机器
  2. 运行jar包中的job客户端主类，`hadoop jar  wc.jar  day04.mr.wc.WcJobSubmitter`

## YARN集群安装

1. 修改yarn的配置文件；

   ```xml
   <!--yarn-site.xml-->
   <property>
   	<name>yarn.resourcemanager.hostname</name>
   	<value>nn1.hdp</value>
   </property>
   <property>
   	<name>yarn.nodemanager.aux-services</name>
   	<value>mapreduce_shuffle</value>
   </property>
   ```

2. 启动yarn的各个服务进程

   ```shell
   # 逐个进程手动启动
   yarn-daemon.sh start resourcemanager
   yarn-daemon.sh start nodemanager
   # 一键启动脚本（配slaves）：
   start-yarn.sh
   stop-yarn.sh
   ```

3. 启动后，可以查看resourcemanager提供的web console

   * `http://nn1.hdp:8088/cluster`

## MAP REDUCE 程序运行方式

## MAP REDUCE 编程案例（2）

* 需求：针对移动公司的流量使用日志，统计出每一个手机用户所耗费的总上行流量，总下行流量，总流量

* 设计：

  * map ：  key--》 手机号；  value--》流量数据对象
  * reduce ： 对相同key的一组value数据进行累加

* **涉及到知识点：**自定义类型需要实现hadoop的序列化接口Writable

  ![image-20191126155007261](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126155007261.png)

  ```java
  //  mr框架在反序列化这个类的对象时要调用的方法
  @Override
  public void readFields(DataInput in) throws IOException {
    this.upFlow = in.readLong();
    this.dFlow = in.readLong();
    this.sumFlow = in.readLong();
  
  }
  // mr框架在序列化这个类的对象时要调用的方法
  @Override
  public void write(DataOutput out) throws IOException {
    out.writeLong(upFlow);
    out.writeLong(dFlow);
    out.writeLong(sumFlow);
  }
  ```

## MAP REDUCE 编程案例（3）

* 数据：电影评分数据 rating.json

* 需求

  1. 统计每一个用户的评分的综合

     * 设计：map：  key--> uid  ;  value-->评分

  2. 统计每一个用户评分最高的20部电影的评分信息

     * 设计：

       * map：
         * key  --> uid  ;
         * key  --> uid  ;
       * reduce
         * 拿到相同uid的一组rateBean后，排序，返回前20条

     * 涉及到一个知识点

       * reduceTask端提供的迭代器，每次迭代返回的是同一个对象（只是被set了不同的值）

       * 所以，如果你要通过迭代器把一组数据取出来存入一个集合的话，你不能直接把迭代器返回的对象放入集合

         ![image-20191126155959164](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126155959164.png)

* 练习需求

  * 统计每一个用户的平均评分

  * 统计最热门的前20部电影

    * 求全局聚合topn运算，reducetask开1个，然后在这个reducetask中，通过reduce方法，把每一组数据的聚合结果放入一个成员集合中，然后最后在cleanup中从成员集合中取topn
    * 涉及知识点：cleanup方法
    * // reduceTask在处理完自己的数据后，最后调用一次cleanup方法

  * 统计总评分最高的前20部电影

  * 请用mr实现之前写过的求共同好友

    ```shell
    A:B,C,D,F,E,O
    B:A,C,E,K
    C:F,A,D,I
    D:A,E,F,L
    E:B,C,D,M,L
    F:A,B,C,D,E,O,M
    G:A,C,D,E,F
    H:A,C,D,E,O
    I:A,O
    J:B,O
    K:A,C,D
    L:D,E,F
    M:E,F,G
    O:A,H,I,
    ```

    * 求出，有哪些人两两之间有共同好友，以及共同好友都是谁：
      * A-B	C,E
      * D-E    L

  * 创建索引-两个mr job来实现

    * 第一个job
      * map：读原始数据， 输出 “单词-文件名”的组合作为key，1作为value
      * reduce: 按组累加
    * 第二个job
      * map：读job1的输出结果，输出 单词作为key,  “文件名-->次数” 作为value
      * reduce： 按组拼接字符串

  * Join算法

    * 需求

      ```shell
      1.有order数据
      oid,uid,pid,number
      o1,u1,p1,20
      o2,u1,p3,10
      
      2.有user数据
      uid,name,addr,phone
      u1,张飞,长坂坡,13888889999
      
      3.求：把每一条订单数据中的用户信息补全；
      o1,u1,张飞，长坂坡，13888889999,p1,20
      
      等价于sql中的join:
      select o.oid,o.uid,u.name,u.addr,u.phone,o.pid,o.number
      from order o join user u on o.uid=u.uid;
      ```

    * 设计

      * map： 输出uid作key，其他信息做value
      * reduce： 拿到uid相同的一组数据，分辨出来哪条是user数据，哪些条是order数据，然后拼接输出；

    * 涉及的知识点

      1. 在map中如何获知当前正在处理的数据所属的文件：

         ```java
         FileSplit  split = (FileSplit) context.getInputSplit();
         String fileName =  split.getPath.getName();
         ```

      2. maptask在正式调用map方法处理数据之前，会调用一次setup()方法

      3. maptask在处理完所有数据之后，会调用一次cleanup()方法

         <img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126163020179.png" alt="image-20191126163020179" style="zoom:25%;" />

## MAP REDUCE 编程案例（4）

* 需求：有一堆文本文件，a.txt b.txt ....，请输出去重的单词

* 需求2

  ![image-20191126160401163](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126160401163.png)

## MAP REDUCE运行模式详解

* 两种模式

1. Yarn分布式模式

   * 总控：MRAppMaster

     ![image-20191126160812580](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126160812580.png)

2. 本地运行模式

   * 单机程序：LocalJobRunner

     <img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126160906045.png" alt="image-20191126160906045" style="zoom:25%;" />

* 提交方式

  * 决定mr程序究竟以yarn模式运行，还是以local模式运行，取决于一个提交参数：

    `mapreduce.framework.name =local`

  * 这个参数，可以

    * 在mr程序代码中修改`conf.set(“mapreduce.framework.name”,”yarn”);`

    * 或者，在提交job的机器上修改配置文件`mapred-site.xml`

      ```xml
      <property>
      	<name>mapreduce.framework.name</name>
      	<value>yarn</value>
      </property
      ```

  1. windows提交到本地运行

     ```shell
     # 需要准备windows中的hadoop_home
     # 在环境变量中配置 HADOOP_HOME
     # 在PATH变量中配置  HADOOP_HOME下的bin目录
     # bin目录中是经过windows平台编译的文件
     ```

  2. windows提交到yarn分布式运行(仅供了解)

     ```java
     System.setProperty("HADOOP_USER_NAME", "root");
     Configuration conf = new Configuration();
     conf.set("mapreduce.framework.name", "yarn");
     conf.set("yarn.resourcemanager.hostname", "nn1.hdp");
     conf.set("fs.defaultFS", "hdfs://nn1.hdp:9000/");
     // 表示本次是跨平台提交job，需要按linux的方言拼接shell命令
     conf.set("mapreduce.app-submission.cross-platform", "true");
     
     Job job = Job.getInstance(conf);
     //job.setJarByClass(UserMoiveTopn.class);
     job.setJar("d:/topn.jar");
     ```

  3. Linux提交到本地运行
  4. **Linux提交到yarn分布式运行**

## MAP REDUCE高级编程案例-分组topn高效实现

![image-20191126163139906](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126163139906.png)

* 核心要点：
  * 利用mr框架的排序机制来帮我们排序
    * 把自定义的Bean放入map输出的key中
    * Bean就得实现WritableComparable接口
  * 自己控制数据分区规则
    * 自定义Partitioner的子类，覆盖getPartition方法
  * 自己控制分组规则GroupingComparator
    * 自定义WritableComparator的子类，覆盖compare(WritableComparable a,WritableComparable b)

## MAP REDUCE 编程模型总结

* 分组聚合

  ![image-20191126163634363](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126163634363.png)

* 全局聚合

  ![image-20191126163844690](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126163844690.png)

  * 总结，mapreduce编程模型就是一个分布式分组聚合运算模型

## MAP REDUCE原理1：并行度机制

1. maptask并行度

   * maptask的并行度，**由job客户端来计算**得出
   * **计算逻辑**封装在FileInputFormat抽象类中.**getSplits()方法**中
   * 具体逻辑
     1. 探测输入目录中有哪些文件(/wc/input/a.txt 200M  /wc/input/b.txt 100M  /wc/input/c.txt 130M)
     2. 遍历每一个文件：
        * -->切分任务片
        * -->如果这个文件的剩余长度>128M*1.1,则划分一个任务片FileSplit{path,startoff,length}；

   * 上述文件的任务分片结果如下

     * FileSplit[]
       * split0: /wc/input/a.txt  		0   		128M
       * split1: /wc/input/a.txt 	 128M	    72M
       * split2: /wc/input/b.txt 	     0   	    100M
       * split3: /wc/input/c.txt 	      0   	    130M

   * 思考1

     * 为什么计算任务分片的时候，要以128M作为计算基准？

       * 因为hdfs中的文件在物理上是分块存储的，所以，切片以block块大小作为基准，mr程序的读数据时，就不用大量地跨datanode节点读取

       * 如果用户非要自己决定任务分片的大小，还是有办法的，通过修改参数来实现

         ```xml
         <property>
           <name>mapreduce.input.fileinputformat.split.minsize</name>
           <value>0</value>
         </property>
         ```

       * 可以将minsize调大，大于blocksize时，切片大小就以你的minsize为准了

   * 思考2

     * 如果待处理的是大量小文件，则通过上述默认并行度计算机制，会导致大量的maptask启动，每个maptask处理的数据量又特别小，效率极低
     * 如何解决
       1. 提前把小文件进行合并
       2. 重写FileInputFormat的getSplit()方法，改变切片计算逻辑，把多个小文件分到一个任务片
       3. hadoop已经有了一个子类实现我们（2）中的想法：CombineTextInputFormat

2. reducetask并行度

   * 并行度通常都是由用户根据数据量自行设定
   * job.setNumReduceTasks(1);

## MAP REDUCE原理2：mr内部运行流程

* mapreduce程序是一个分布式程序，每一个mapreduce程序中都有如下角色

  1. YarnChild（MapTask）：工作进程
  2. YarnChild（ReduceTask）：工作进程
  3. MRAppMaster：控制进程

* 每一个mapreduce程序中，有且仅有1个MRAppMaster

* 可能有多个MapTask，多个ReduceTask

  ![mapreduce程序内部数据处理流程--九阴真经](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/mapreduce程序内部数据处理流程--九阴真经.png)

## MAP REDUCE原理3：mr程序启动调度机制

* 前置知识，思考一个问题

  * **你能不能从一台机器上运行一个程序，在这个程序中再去启动另一台机器上的另一个程序**

  ![mapreduce分布式启动运行全流程--九阳神功](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/mapreduce分布式启动运行全流程--九阳神功.png)

## MAP REDUCE知识点扩充

1. InputFormat/OutputFormat

   * [参考资料：CombineTextInputFormat](https://blog.csdn.net/wawmg/article/details/17095125)

   * 1.SequenceFile文件格式案例

     * SequenceFile简介

       * 是hadoop中设计的一种对象序列化文件格式
       * 有一个文件头： SEQ 表示这是一个sequenceFile文件，key的类型全名   value的类型全名
       * 文件中的内容组织形式为： 连续的KEY-VALUE对象序列化二进制

     * 输出sequenceFile

       ```java
       // 想把最终输出结果设置成sequenceFile
       job.setOutputFormatClass(SequenceFileOutputFormat.class);
       ```

     * 读取sequenceFile

       ```java
       //在main方法中：
       job.setInputFormatClass(SequenceFileInputFormat.class);
       //然后，在自定义Mapper中，要注意KEYIN,VALUEIN这两个泛型应该跟sequenceFile文件中的key，value类型对应
       //并且map方法中得到的key-value就是从sequenceFile中读到的一对key-value
       ```

   * 2.小文件输入处理

     * 如果面对的是大量小文件，默认情况下，使用TextFileInputFormat里面的getSplits()方法，会算出大量的mapTask任务片，导致资源浪费，效率低下

     * 解决办法

       1. 提前合并小文件，再做业务计算

       2. 还可以使用CombineTextInputFormat组件来做输入，它能将多个小文件规划为一个任务切片

          ```java
          job.setInputFormatClass(CombineTextInputFormat.class);
          ```

     * [参考资料：CombineTextInputFormat](https://blog.csdn.net/wawmg/article/details/17095125)

   * 3.自定义OutputFormat案例

     * 需求：

       * 现有大量移动用户的上网浏览记录，需要对这些浏览记录进行信息增强
       * 将增强成功的日志写入一个结果目录
       * 将增强不成功的url写入另一个结果目录

       ```shell
       # MySQL小知识补充
       # mysql服务器默认情况下是不允许用户从远程机器连接的，如果需要远程连接，那可以在mysql服务器上用命令行客户端登录进去，用mysql命令进行远程连接授权
       mysql> grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
       Query OK, 0 rows affected (0.00 sec)
       mysql> flush privileges;
       Query OK, 0 rows affected (0.00 sec)
       mysql>
       ```

     * 关键点

       * 自定义了一个OutputFormat，如何在mapreduce程序中访问mysql数据库

       * 流程设计

         ![image-20191126174706719](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126174706719.png)

   * 4.MultipleOutputs组件（多路输出器）

     ![image-20191126174900600](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126174900600.png)

     ![image-20191126174924035](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126174924035.png)

2. 输出压缩

   1. map输出压缩

      ```xml
      <!--mapred-site.xml-->
      <property>
        <name>mapreduce.map.output.compress</name>
        <value>false</value>
      </property>
      <property>
        <name>mapreduce.map.output.compress.codec</name>
        <value>org.apache.hadoop.io.compress.DefaultCodec</value>
      </property>
      ```

      * 还可以在job提交器的main方法中配：

        ```java
        conf.set(“mapreduce.map.output.compress”,”true”)
        ```

   2. reduce输出压缩

      ```xml
      <!--mapred-site.xml-->
      <property>
        <name>mapreduce.output.fileoutputformat.compress</name>
        <value>false</value>
      </property>
      
      <property>
        <name>mapreduce.output.fileoutputformat.compress.type</name>
        <value>RECORD</value>
        <description>If the job outputs are to compressed as SequenceFiles, how should
                     they be compressed? Should be one of NONE, RECORD or BLOCK.
        </description>
      </property>
      
      <property>
        <name>mapreduce.output.fileoutputformat.compress.codec</name>
        <value>org.apache.hadoop.io.compress.DefaultCodec</value>
      </property>
      ```

   * 还可以在job提交器的main方法中配

     ```java
     conf.set(“mapreduce.output.fileoutputformat.compress”,”true”)
     ```

3. Combiner组件

   * 本质：

     * 是maptask对自己的局部数据进行局部聚合时使用的工具类

     * maptask对局部数据聚合时的行为规则  与   reducetask对全局数据进行聚合时的行为规则完全一致

     * 所以，这个Combiner组件的规范依然是继承Reducer类

     * 通常，我们直接把写好的XXReducer设置为maptask所要用的combiner

     * 当然，你如果觉得写代码很爽，你可以再写一个专门Combiner

     * 写好后，要告诉maptask，做局部聚合

       ```java
       // 告诉maptask需要做局部聚合，而且告诉maptask，做局部聚合所用的工具是这个WcCombiner类
       job.setCombinerClass(WcReducer.class);
       ```

4. 配置文件传递

   * 如果需要从外部传入参数到mapreduce内部去使用，则可以通过conf对象

   * 通过客户端将各类参数封装到conf对象

   * conf对象会被job客户端写成xml文件，提交到yarn的程序提交资源路径中

   * maptask或者reducetask在启动时会读取到容器中的这些配置参数文件，并加载，然后放入context

   * 在map方法或者reduce方法等内部程序中，可以通过context获取到之前设置的参数

     ![image-20191126180915623](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126180915623.png)

5. 分布式cache

   * 场景：改造join示例

   * 核心思想：让每一个maptask本地持有一份user.txt，以便于maptask读取order数据后，直接就join完成，不用再发给reducetask分组聚合。可以消除数据倾斜的不良影响

     ![image-20191126181028579](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126181028579.png)

   * 实现机制：利用mr框架中的分布式cache来将user.txt作为创建容器的程序资源

     ```java
     // 将user数据作为程序资源提交到yarn的资源目录中，以便于进入容器
     job.addCacheFile(new URI("hdfs://nn1.hdp:9000/cachetmp/user.txt"));
     ```

   * MR框架分布式cache的实质：

     * 就是帮你将程序运行时所需要的一些额外资源，发到每一个task容器中去；比如，可以把一些额外依赖的jar包发到容器的classpath中

       ```java
       job.addArchiveToClassPath(new Path("/jars/fastjson.jar"));
       ```

     * Spark框架中，类似的机制叫做广播变量

6. 全局计数器

   * 如果需要在MR程序中进行一些全局计数，那么MRAppMaster提供了这个全局计数器，所有maptask或者reducetask都可以往MRAppMaster提供的全局计数器中进行全局累加

   * 使用方法

     ![image-20191126181240594](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126181240594.png)

   * 查看计数器结果：客户端控制台的输出中会打印出所有计数器的值

     ![image-20191126181321613](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126181321613.png)

7. MAP REDUCE / YARN内存参数

   * [vmem参考资料](https://blog.yoodb.com/sugarliny/article/detail/1307)

   1. MRAPpMaster容器的内存、cpu资源参数

   ```xml
   <!--MRApplicationMaster-->
   <property>
     <name>yarn.app.mapreduce.am.resource.mb</name>
     <value>1536</value>
     <description>The amount of memory the MR AppMaster needs.</description>
   </property>
   
   <property>
     <name>yarn.app.mapreduce.am.resource.cpu-vcores</name>
     <value>1</value>
     <description>
         The number of virtual CPU cores the MR AppMaster needs.
     </description>
   </property>
   ```

   * 最好是在mapreduce的job客户端中通过代码设置

   ```java
   conf.set("yarn.app.mapreduce.am.resource.mb", "2048");
   conf.set("yarn.app.mapreduce.am.resource.cpu-vcores", "2");
   ```

   2. maptask或reducetask容器的内存、cpu大小

   ```xml
   <property>
     <name>mapreduce.map.memory.mb</name>
     <value>1024</value>
     <description>The amount of memory to request from the scheduler for each
     map task.
     </description>
   </property>
   
   <property>
     <name>mapreduce.map.cpu.vcores</name>
     <value>1</value>
     <description>The number of virtual cores to request from the scheduler for
     each map task.
     </description>
   </property>
   
   <property>
     <name>mapreduce.reduce.memory.mb</name>
     <value>1024</value>
     <description>The amount of memory to request from the scheduler for each
     reduce task.
     </description>
   </property>
   
   <property>
     <name>mapreduce.reduce.cpu.vcores</name>
     <value>1</value>
     <description>The number of virtual cores to request from the scheduler for
     each reduce task.
     </description>
   </property>
   ```

   3. yarn中的资源总量配置

      * yarn中一个nodemanager机器上，最多可以提供多少内存用来创建容器：

        * 以下参数都需要配置在yarn-site.xml中，并重启yarn后才能生效；

        * 新版本中，nodemanager的资源总量，有两种方式来配置：

          ```shell
          # 1.自动探测
          yarn.nodemanager.resource.memory-mb = -1   // 内存容量
          yarn.nodemanager.resource.cpu-vcores = -1     // cpu核数
          yarn.nodemanager.resource.detect-hardware-capabilities = true
          # 2.手动指定
          yarn.nodemanager.resource.memory-mb = 2048
          yarn.nodemanager.resource.cpu-vcores = 4
          nodemanger创建一个容器的最大内存上限：
          yarn.scheduler.maximum-allocation-mb = 8192
          nodemanger创建一个容器的最小内存下限：
          yarn.scheduler.minimum-allocation-mb = 1024
          最小值相当于一个基本申请单元，如果你申请一个1.5g，那么真实分配的就是2g，是最小值的整数倍
          # 3.虚拟内存的配置
          nodemanager的虚拟内存分配参数：
          nodemanger对每一个容器能使用的虚拟内存数，有限制：
          yarn.nodemanager.vmem-pmem-ratio = 2.1    //  就是容器的物理内存的2.1倍
          yarn.nodemanager.vmem-check-enabled = true   // 虚拟内存使用量的监控开关，如果关闭，则不检查虚拟内存使用是否超标；
          ```

        * [vmem参考资料](https://blog.yoodb.com/sugarliny/article/detail/1307)

8. historyserver配置

   * jobhistoryserver可以将yarn上运行过的程序相关记录信息持久化地保存下来（hdfs）:

   ```xml
   <!--mapred-site.xml-->
   <property>
     <name>mapreduce.jobhistory.address</name>
     <value>nn1.hdp:10020</value>
     <description>MapReduce JobHistory Server IPC host:port</description>
   </property>
   
   <property>
     <name>mapreduce.jobhistory.webapp.address</name>
     <value>nn1.hdp:19888</value>
     <description>MapReduce JobHistory Server Web UI host:port</description>
   </property>
   
   <property>
     <name>mapreduce.jobhistory.webapp.https.address</name>
     <value>nn1.hdp:19890</value>
     <description>
       The https address the MapReduce JobHistory Server WebApp is on.
     </description>
   </property>
   
   <property>
     <name>mapreduce.jobhistory.admin.address</name>
     <value>nn1.hdp:10033</value>
     <description>The address of the History server admin interface.</description>
   </property>
   ```

   ```xml
   <!--yarn-site.xml-->
   <property>
     <name>yarn.log-aggregation-enable</name>
     <value>true</value>
     <description>The address of the History server admin interface.</description>
   </property>
   ```

   * 启动jobhistoryserver
     * `mr-jobhistory-daemon.sh start historyserver`

9. yarn中运行的job信息查看

   ```shell
   hadoop  job  -status job_id
   hadoop  job  -kill job_id
   ```

10. hdfs运维补充

    * 如何动态扩容？
      * 加一台datanode服务器，同步配置文件，然后启动datanode服务进程，即已经扩容，然后在slaves中列入这台新增datanode服务器，以便于下次可以一键启动
    * 如果namenode的元数据文件（fsimage、edits）损坏，导致namenode启动失败，将secondarynamenode的元数据目录中的所有文件，拷贝覆盖掉namenode的元数据目录
    * 有一个更可靠的解决方案： 将HDFS配置成HA（高可用）模式

# Hadoop的HA集群搭建







