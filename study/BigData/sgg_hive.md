# Hive基本概念

## 是什么？

* 由Facebook开源用于解决海量结构化日志的数据统计

* 是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能

* 本质是：将HQL转化成MapReduce程序

  ![image-20191126203811002](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126203811002.png)

  * Hive处理的数据存储在HDFS
  * Hive分析数据底层的实现是MapReduce
  * 执行程序运行在Yarn上

## 优缺点

* 优点
  * 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）
  * 避免了去写MapReduce，减少开发人员的学习成本
  * Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合
  * Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高
  * Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数
* 缺点
  * Hive的HQL表达能力有限
    * 迭代式算法无法表达
    * 数据挖掘方面不擅长，由于MapReduce数据处理流程的限制，效率更高的算法却无法实现
  * Hive的效率比较低
    * Hive自动生成的MapReduce作业，通常情况下不够智能化
    * Hive调优比较困难，粒度较粗

## Hive架构原理

1. 用户接口：Client
   * CLI（command-line interface）、JDBC/ODBC(jdbc访问hive)、WEBUI（浏览器访问hive）
2. 元数据：Metastore
   * 元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等
   * 默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore
3. Hadoop
   * 使用HDFS进行存储，使用MapReduce进行计算
4. 驱动器：Driver
   1. 解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误
   2. 编译器（Physical Plan）：将AST编译生成逻辑执行计划
   3. 优化器（Query Optimizer）：对逻辑执行计划进行优化
   4. 执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark

![image-20191126204404866](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126204404866.png)

* Hive通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的Driver，结合元数据(MetaStore)，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口

## Hive和数据库比较

* 查询语言
  * 由于SQL被广泛的应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL。熟悉SQL开发的开发者可以很方便的使用Hive进行开发
* 数据存储位置
  * Hive 是建立在 Hadoop 之上的，所有 Hive 的数据都是存储在 HDFS 中的。而数据库则可以将数据保存在块设备或者本地文件系统中
* 数据更新
  * 由于Hive是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，Hive中不建议对数据的改写，所有的数据都是在加载的时候确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO … VALUES 添加数据，使用 UPDATE … SET修改数据
* 执行
  * Hive中大多数查询的执行是通过 Hadoop 提供的 MapReduce 来实现。而数据库通常有自己的执行引擎
* 执行延迟
  * Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致 Hive 执行延迟高的因素是 MapReduce框架。由于MapReduce 本身具有较高的延迟，因此在利用MapReduce 执行Hive查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive的并行计算显然能体现出优势
* 可扩展性
  * 由于Hive是建立在Hadoop之上的，因此Hive的可扩展性是和Hadoop的可扩展性是一致的（世界上最大的Hadoop 集群在 Yahoo!，2009年的规模在4000 台节点左右）。而数据库由于 ACID 语义的严格限制，扩展行非常有限。目前最先进的并行数据库 Oracle 在理论上的扩展能力也只有100台左右
* 数据规模
  * 由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小

# Hive安装部署

## Hive安装及配置

```shell
#1. 把apache-hive-1.2.1-bin.tar.gz上传到linux的/opt/software目录下
#2. 解压apache-hive-1.2.1-bin.tar.gz到/opt/module/目录下面
[atguigu@hadoop102 software]$ tar -zxvf apache-hive-1.2.1-bin.tar.gz -C /opt/module/
#3. 修改apache-hive-1.2.1-bin.tar.gz的名称为hive
[atguigu@hadoop102 module]$ mv apache-hive-1.2.1-bin/ hive
#4. 修改/opt/module/hive/conf目录下的hive-env.sh.template名称为hive-env.sh
[atguigu@hadoop102 conf]$ mv hive-env.sh.template hive-env.sh
#5. 配置hive-env.sh文件
#a 配置HADOOP_HOME路径
export HADOOP_HOME=/opt/module/hadoop-2.7.2
#b 配置HIVE_CONF_DIR路径
export HIVE_CONF_DIR=/opt/module/hive/conf
```

## Hadoop集群配置

```shell
#1. 必须启动hdfs和yarn
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
#2. 在HDFS上创建/tmp和/user/hive/warehouse两个目录并修改他们的同组权限可写
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir /tmp
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir -p /user/hive/warehouse
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /tmp
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /user/hive/warehouse
```

## Hive基本操作

```shell
#1. 启动hive
[atguigu@hadoop102 hive]$ bin/hive
#2. 查看数据库
hive> show databases;
#3. 打开默认数据库
hive> use default;
#4. 显示default数据库中的表
hive> show tables;
#5. 创建一张表
hive> create table student(id int, name string);
#6. 显示数据库中有几张表
hive> show tables;
#7. 查看表的结构
hive> desc student;
#8. 向表中插入数据
hive> insert into student values(1000,"ss");
#9. 查询表中数据
hive> select * from student;
#10. 退出hive
hive> quit;
```

* 说明：（查看hive在hdfs中的结构）
  * 数据库：在hdfs中表现为${hive.metastore.warehouse.dir}目录下一个文件夹
  * 表：在hdfs中表现所属db目录下一个文件夹，文件夹中存放该表中的具体数据

## 将本地文件导入hive案例

* 需求

  * 将本地/opt/module/datas/student.txt这个目录下的数据导入到hive的student(id int, name string)表中

* 数据准备

  ```shell
  # 在/opt/module/datas这个目录下准备数据
  #1. 在/opt/module/目录下创建datas
  [atguigu@hadoop102 module]$ mkdir datas
  #2. 在/opt/module/datas/目录下创建student.txt文件并添加数据，注意以tab键间隔
  [atguigu@hadoop102 datas]$ touch student.txt
  [atguigu@hadoop102 datas]$ vi student.txt
  1001	zhangshan
  1002	lishi
  1003	zhaoliu
  ```

* Hive实际操作

  ```shell
  #1. 启动hive
  [atguigu@hadoop102 hive]$ bin/hive
  #2. 显示数据库
  hive> show databases;
  #3. 使用default数据库
  hive> use default;
  #4. 显示default数据库中的表
  hive> show tables;
  #5. 删除已创建的student表
  hive> drop table student;
  #6. 创建student表, 并声明文件分隔符’\t’
  hive> create table student(id int, name string) ROW FORMAT DELIMITED FIELDS TERMINATED
   BY '\t';
  #7. 加载/opt/module/datas/student.txt 文件到student数据库表中
  hive> load data local inpath '/opt/module/datas/student.txt' into table student;
  #8. Hive查询结果
  hive> select * from student;
  OK
  1001	zhangshan
  1002	lishi
  1003	zhaoliu
  Time taken: 0.266 seconds, Fetched: 3 row(s)
  ```

* 遇到的问题

  ```shell
  # 再打开一个客户端窗口启动hive，会产生java.sql.SQLException异常
  Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException:
   Unable to instantiate
   org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
          at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:522)
          at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:677)
          at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
          at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
          at 
  # 原因是，Metastore默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore;        
  ```

## MySQL安装

```shell
# 查看mysql是否安装，如果安装了，卸载mysql
[root@hadoop102 桌面]# rpm -qa|grep mysql
mysql-libs-5.1.73-7.el6.x86_64
[root@hadoop102 桌面]# rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64s
# 解压mysql-libs.zip文件到当前目录
[root@hadoop102 software]# unzip mysql-libs.zip
[root@hadoop102 software]# ls
mysql-libs.zip
mysql-libs
# 进入到mysql-libs文件夹下
[root@hadoop102 mysql-libs]# ll
总用量 76048
-rw-r--r--. 1 root root 18509960 3月  26 2015 MySQL-client-5.6.24-1.el6.x86_64.rpm
-rw-r--r--. 1 root root  3575135 12月  1 2013 mysql-connector-java-5.1.27.tar.gz
-rw-r--r--. 1 root root 55782196 3月  26 2015 MySQL-server-5.6.24-1.el6.x86_64.rpm
# 安装MySQL服务端
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
# 查看产生的随机密码
[root@hadoop102 mysql-libs]# cat /root/.mysql_secret
OEXaQuS8IWkG19Xs
# 查看mysql状态
[root@hadoop102 mysql-libs]# service mysql status
# 启动mysql
[root@hadoop102 mysql-libs]# service mysql start
# 安装Mysql客户端
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
# 连接mysql
[root@hadoop102 mysql-libs]# mysql -uroot -pOEXaQuS8IWkG19Xs
# 修改密码
mysql>SET PASSWORD=PASSWORD('000000');
# 退出mysql
mysql>exit
```

* MySql中user表中主机配置

  * 配置只要是root用户+密码，在任何主机上都能登录MySQL数据库

    ```shell
    #1. 进入mysql
    [root@hadoop102 mysql-libs]# mysql -uroot -p000000
    #2. 显示数据库
    mysql>show databases;
    #3. 使用mysql数据库
    mysql>use mysql;
    #4. 展示mysql数据库中的所有表
    mysql>show tables;
    #5. 展示User表的结构
    mysql>desc user;
    #6. 查询user表
    mysql>select User, Host, Password from user;
    #7. 修改user表，把Host表内容改为%
    mysql>update user set host='%' where host='localhost';
    #8. 删除root用户的其他host
    mysql>delete from user where Host='hadoop102';
    mysql>delete from user where Host='127.0.0.1';
    mysql>delete from user where Host='::1';
    #9. 刷新
    mysql>flush privileges;
    #10. 退出
    mysql>quit;
    ```

## Hive元数据配置到MySQL

1. 驱动拷贝

   ```shell
   #1. 在/opt/software/mysql-libs目录下解压mysql-connector-java-5.1.27.tar.gz驱动包
   [root@hadoop102 mysql-libs]# tar -zxvf mysql-connector-java-5.1.27.tar.gz
   #2. 拷贝/opt/software/mysql-libs/mysql-connector-java-5.1.27目录下的mysql-connector-java-5.1.27-bin.jar到/opt/module/hive/lib/
   [root@hadoop102 mysql-connector-java-5.1.27]# cp mysql-connector-java-5.1.27-bin.jar
    /opt/module/hive/lib/
   ```

2. 配置Metastore到MySQL

   ```shell
   #1. 在/opt/module/hive/conf目录下创建一个hive-site.xml
   [atguigu@hadoop102 conf]$ touch hive-site.xml
   [atguigu@hadoop102 conf]$ vi hive-site.xml
   #2. 根据官方文档配置参数，拷贝数据到hive-site.xml文件中
   https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin
   #3. 配置完毕后，如果启动hive异常，可以重新启动虚拟机。（重启后，别忘了启动hadoop集群）
   ```

   ```xml
   <?xml version="1.0"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
   	<property>
   	  <name>javax.jdo.option.ConnectionURL</name>
   	  <value>jdbc:mysql://hadoop102:3306/metastore?createDatabaseIfNotExist=true</value>
   	  <description>JDBC connect string for a JDBC metastore</description>
   	</property>
   
   	<property>
   	  <name>javax.jdo.option.ConnectionDriverName</name>
   	  <value>com.mysql.jdbc.Driver</value>
   	  <description>Driver class name for a JDBC metastore</description>
   	</property>
   
   	<property>
   	  <name>javax.jdo.option.ConnectionUserName</name>
   	  <value>root</value>
   	  <description>username to use against metastore database</description>
   	</property>
   
   	<property>
   	  <name>javax.jdo.option.ConnectionPassword</name>
   	  <value>000000</value>
   	  <description>password to use against metastore database</description>
   	</property>
   </configuration>
   ```

3. 多窗口启动hive测试

   ```shell
   #1. 先启动MySQL
   [atguigu@hadoop102 mysql-libs]$ mysql -uroot -p000000
   # 查看有几个数据库
   mysql> show databases;
   #2. 再次打开多个窗口，分别启动hive
   [atguigu@hadoop102 hive]$ bin/hive
   #3. 启动hive后，回到MySQL窗口查看数据库，显示增加了metastore数据库
   mysql> show databases;
   ```

## HiveJDBC访问

```shell
#1. 启动hiveserver2服务
[atguigu@hadoop102 hive]$ bin/hiveserver2
#2. 启动beeline
[atguigu@hadoop102 hive]$ bin/beeline
#3. 连接hiveserver2
beeline> !connect jdbc:hive2://hadoop102:10000（回车）
Connecting to jdbc:hive2://hadoop102:10000
Enter username for jdbc:hive2://hadoop102:10000: atguigu（回车）
Enter password for jdbc:hive2://hadoop102:10000: （直接回车）
Connected to: Apache Hive (version 1.2.1)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://hadoop102:10000> show databases;
+----------------+--+
| database_name  |
+----------------+--+
| default        |
| hive_db2       |
+----------------+--+
```

## Hive常用交互命令

```shell
[atguigu@hadoop102 hive]$ bin/hive -help
#1. “-e”不进入hive的交互窗口执行sql语句
[atguigu@hadoop102 hive]$ bin/hive -e "select id from student;"
#2. “-f”执行脚本中sql语句
#2.1 在/opt/module/datas目录下创建hivef.sql文件
[atguigu@hadoop102 datas]$ touch hivef.sql
select *from student;
#2.2 执行文件中的sql语句
[atguigu@hadoop102 hive]$ bin/hive -f /opt/module/datas/hivef.sql
#2.3 执行文件中的sql语句并将结果写入文件中
[atguigu@hadoop102 hive]$ bin/hive -f /opt/module/datas/hivef.sql  > /opt/module/datas/hive_result.txt
```

* Hive其他命令操作

  ```shell
  #1. 退出hive窗口
  hive(default)>exit;            # exit:先隐性提交数据，再退出；
  hive(default)>quit;            # quit:不提交数据，退出；
  #2. 在hive cli命令窗口中如何查看hdfs文件系统
  hive(default)>dfs -ls /;
  #3. 在hive cli命令窗口中如何查看本地文件系统
  hive(default)>! ls /opt/module/datas;
  #4. 查看在hive中输入的所有历史命令
  #4.1 进入到当前用户的根目录/root或/home/atguigu
  #4.2 查看. hivehistory文件
  [atguigu@hadoop102 ~]$ cat .hivehistory
  ```

## Hive常见属性配置

* Hive数据仓库位置配置

  ```shell
  #1. Default数据仓库的最原始位置是在hdfs上的：/user/hive/warehouse路径下
  #2. 在仓库目录下，没有对默认的数据库default创建文件夹。如果某张表属于default数据库，直接在数据仓库目录下创建一个文件夹
  #3. 修改default数据仓库原始位置（将hive-default.xml.template如下配置信息拷贝到hive-site.xml文件中）
  <property>
  	<name>hive.metastore.warehouse.dir</name>
  	<value>/user/hive/warehouse</value>
  	<description>location of default database for the warehouse</description>
  </property>
  #配置同组用户有执行权限
  bin/hdfs dfs -chmod g+w /user/hive/warehouse
  ```

* 查询后信息显示配置

  ```shell
  #1. 在hive-site.xml文件中添加如下配置信息，就可以实现显示当前数据库，以及查询表的头信息配置
  <property>
  	<name>hive.cli.print.header</name>
  	<value>true</value>
  </property>
  <property>
  	<name>hive.cli.print.current.db</name>
  	<value>true</value>
  </property>
  #2. 重新启动hive，对比配置前后差异
  select * from student						# 配置前
  select * from db_hive.student		# 配置后
  ```

* Hive运行日志信息配置

  ```shell
  #1. Hive的log默认存放在/tmp/atguigu/hive.log目录下（当前用户名下）
  #2. 修改hive的log存放日志到/opt/module/hive/logs
  #2.1 修改/opt/module/hive/conf/hive-log4j.properties.template文件名称为hive-log4j.properties
  [atguigu@hadoop102 conf]$ pwd
  /opt/module/hive/conf
  [atguigu@hadoop102 conf]$ mv hive-log4j.properties.template hive-log4j.properties
  #2.2 在hive-log4j.properties文件中修改log存放位置
  hive.log.dir=/opt/module/hive/logs
  ```

* 参数配置方式

  ```shell
  #1. 查看当前所有的配置信息
  hive>set;
  #2. 参数的配置三种方式
  #2.1 配置文件方式
  默认配置文件：hive-default.xml 
  用户自定义配置文件：hive-site.xml
  # 注意：用户自定义配置会覆盖默认配置。另外，Hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，Hive的配置会覆盖Hadoop的配置。配置文件的设定对本机启动的所有Hive进程都有效
  #2.2 命令行参数方式
  #启动Hive时，可以在命令行添加-hiveconf param=value来设定参数
  #例如
  [atguigu@hadoop103 hive]$ bin/hive -hiveconf mapred.reduce.tasks=10;
  # 注意：仅对本次hive启动有效
  # 查看参数设置
  hive (default)> set mapred.reduce.tasks;
  #3. 参数声明方式
  # 可以在HQL中使用SET关键字设定参数
  # 例如：
  hive (default)> set mapred.reduce.tasks=100;
  # 注意：仅对本次hive启动有效
  # 查看参数设置
  hive (default)> set mapred.reduce.tasks;
  # 上述三种设定方式的优先级依次递增。即配置文件<命令行参数<参数声明。注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些参数的读取在会话建立以前已经完成了
  ```

# Hive数据类型

## 基本数据类型

![image-20191126214603146](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126214603146.png)

* 对于Hive的String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数

## 集合数据类型

![image-20191126214726200](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126214726200.png)

* Hive有三种复杂数据类型ARRAY、MAP 和 STRUCT。ARRAY和MAP与Java中的Array和Map类似，而STRUCT与C语言中的Struct类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套

* 案例实操

  1. 假设某表有如下一行，用JSON格式来表示其数据结构。在Hive下访问的格式为

  ```json
  {
      "name": "songsong",
      "friends": ["bingbing" , "lili"] ,       //列表Array, 
      "children": {                            //键值Map,
          "xiao song": 18 ,
          "xiaoxiao song": 19
      }
      "address": {                            //结构Struct,
          "street": "hui long guan" ,
          "city": "beijing" 
      }
  }
  ```

  2. 基于上述数据结构，在Hive里创建对应的表，并导入数据

  ```shell
  # 创建本地测试文件test.txt
  songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
  yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
  # 注意：MAP，STRUCT和ARRAY里的元素间关系都可以用同一个字符表示，这里用“_”
  ```

  3. Hive上创建测试表test

  ```sql
  create table test(
  name string,
  friends array<string>,
  children map<string, int>,
  address struct<street:string, city:string>
  )
  row format delimited fields terminated by ','     
  collection items terminated by '_'
  map keys terminated by ':'
  lines terminated by '\n';
  ```

  > row format delimited fields terminated by ',' 	  -- 列分隔符
  >
  > collection items terminated by '_' 						 --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)
  >
  > map keys terminated by ':'									  -- MAP中的key与value的分隔符
  >
  > map keys terminated by ':'				                      -- MAP中的key与value的分隔符

  4. 导入文本数据到测试表

  ```shell
  hive (default)> load data local inpath ‘/opt/module/datas/test.txt’into table test
  ```

  5. 访问三种集合列里的数据，以下分别是ARRAY，MAP，STRUCT的访问方式

  ```shell
  hive (default)> select friends[1],children['xiao song'],address.city from test
  where name="songsong";
  OK
  _c0     _c1     city
  lili    18      beijing
  Time taken: 0.076 seconds, Fetched: 1 row(s)
  ```

## 类型转换

* 隐式类型转换规则如下

  1. 任何整数类型都可以隐式地转换为一个范围更广的类型，如TINYINT可以转换成INT，INT可以转换成BIGINT
  2. 所有整数类型、FLOAT和STRING类型都可以隐式地转换成DOUBLE
  3. TINYINT、SMALLINT、INT都可以转换为FLOAT
  4. BOOLEAN类型不可以转换为任何其它的类型

* 可以使用CAST操作显示进行数据类型转换

  * 例如CAST('1' AS INT)将把字符串'1' 转换成整数1；如果强制类型转换失败，如执行CAST('X' AS INT)，表达式返回空值 NULL

    ```sql
    0: jdbc:hive2://hadoop102:10000> select '1'+2, cast('1'as int) + 2;
    +------+------+--+
    | _c0  | _c1  |
    +------+------+--+
    | 3.0  | 3    |
    +------+------+--+
    ```

# DDL数据定义

## 创建数据库

```shell
#1. 创建一个数据库，数据库在HDFS上的默认存储路径是/user/hive/warehouse/*.db
hive (default)> create database db_hive;
#2. 避免要创建的数据库已经存在错误，增加if not exists判断。（标准写法）
hive (default)> create database db_hive;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Database db_hive already exists
hive (default)> create database if not exists db_hive;
#3. 创建一个数据库，指定数据库在HDFS上存放的位置
hive (default)> create database db_hive2 location '/db_hive2.db';
```

## 查询数据库

```shell
# 显示数据库
hive> show databases;
# 过滤显示查询的数据库
hive> show databases like 'db_hive*';
# 显示数据库信息
hive> desc database db_hive;
# 显示数据库详细信息，extended
hive> desc database extended db_hive;
# 切换当前数据库
hive (default)> use db_hive;
```

## 修改数据库

* 用户可以使用ALTER DATABASE命令为某个数据库的DBPROPERTIES设置键-值对属性值，来描述这个数据库的属性信息。数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置

  ```sql
  hive (default)> alter database db_hive set dbproperties('createtime'='20170830');
  -- 在hive中查看修改结果
  hive> desc database extended db_hive;
  db_name comment location        owner_name      owner_type      parameters
  db_hive         hdfs://hadoop102:8020/user/hive/warehouse/db_hive.db    atguigu USER    {createtime=20170830}
  ```

## 删除数据库

```shell
# 删除空数据库
hive>drop database db_hive2;
# 如果删除的数据库不存在，最好采用 if exists判断数据库是否存在
hive> drop database db_hive;
FAILED: SemanticException [Error 10072]: Database does not exist: db_hive
hive> drop database if exists db_hive2;
# 如果数据库不为空，可以采用cascade命令，强制删除
hive> drop database db_hive;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database db_hive is not empty. One or more tables exist.)
hive> drop database db_hive cascade;
```

## 创建表

1. 建表语法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```

2. 字段解释说明

   1. CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常

   2. EXTERNAL关键字可以让用户创建一个外部表，在建表的同时可以指定一个指向实际数据的路径（LOCATION），在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据

   3. COMMENT：为表和列添加注释

   4. PARTITIONED BY创建分区表

   5. CLUSTERED BY创建分桶表

   6. SORTED BY不常用，对桶中的一个或多个列另外排序

   7. ROW FORMAT

      ```sql
      DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]
              [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char] 
         | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
      ```

      * 用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者ROW FORMAT DELIMITED，将会使用自带的SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的SerDe，Hive通过SerDe确定表的具体的列的数据
      * SerDe是Serialize/Deserilize的简称， hive使用Serde进行行对象的序列与反序列化

   8. STORED AS指定存储文件类型

      * 常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）
      * 如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE

   9. LOCATION ：指定表在HDFS上的存储位置

   10. AS：后跟查询语句，根据查询结果创建表

   11. LIKE允许用户复制现有的表结构，但是不复制数据

* 管理表

  * 理论

    * 默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive会（或多或少地）控制着数据的生命周期。Hive默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。	当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据

  * 案例实操

    ```sql
    --1. 普通表
    create table if not exists student2(
    id int, name string
    )
    row format delimited fields terminated by '\t'
    stored as textfile
    location '/user/hive/warehouse/student2';
    --2. 根据查询结果创建表（查询的结果会添加到新创建的表中）
    create table if not exists student3 as select id, name from student;
    --3. 根据已经存在的表结构创建表
    create table if not exists student4 like student;
    --4. 查询表的类型
    hive (default)> desc formatted student2;
    Table Type:             MANAGED_TABLE  
    ```

* 外部表

  * 理论

    * 因为表是外部表，所以Hive并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉

  * 管理表和外部表的使用场景

    * 每天将收集到的网站日志定期流入HDFS文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表

  * 案例实操

    ```shell
    # 分别创建部门和员工外部表，并向表中导入数据
    #（1）上传数据到HDFS
    hive (default)> dfs -mkdir /student;
    hive (default)> dfs -put /opt/module/datas/student.txt /student;
    # (2) 建表语句
    # 创建外部表
    hive (default)> create external table stu_external(
    id int, 
    name string) 
    row format delimited fields terminated by '\t' 
    location '/student';
    #（3）查看创建的表
    hive (default)> select * from stu_external;
    #（4）查看表格式化数据
    hive (default)> desc formatted dept;
    Table Type:             EXTERNAL_TABLE
    #（5）删除外部表
    hive (default)> drop table stu_external;
    # 外部表删除后，hdfs中的数据还在，但是metadata中stu_external的元数据已被删除
    ```

* 管理表与外部表的互相转换

  ```shell
  #1. 查询表的类型
  hive (default)> desc formatted student2;
  Table Type:             MANAGED_TABLE
  #2. 修改内部表student2为外部表
  alter table student2 set tblproperties('EXTERNAL'='TRUE');
  #3. 查询表的类型
  hive (default)> desc formatted student2;
  Table Type:             EXTERNAL_TABLE
  #4. 修改外部表student2为内部表
  alter table student2 set tblproperties('EXTERNAL'='FALSE');
  #5. 查询表的类型
  hive (default)> desc formatted student2;
  Table Type:             MANAGED_TABLE
  # 注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！
  ```

## 分区表

* 分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。Hive中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多

* 分区表基本操作

  1.  引入分区表（需要根据日期对日志进行管理）

     ```shell
     /user/hive/warehouse/log_partition/20170702/20170702.log
     /user/hive/warehouse/log_partition/20170703/20170703.log
     /user/hive/warehouse/log_partition/20170704/20170704.log
     ```

  2. 创建分区表语法

     ```shell
     hive (default)> create table dept_partition(
     deptno int, dname string, loc string
     )
     partitioned by (month string)
     row format delimited fields terminated by '\t';
     #注意：分区字段不能是表中已经存在的数据，可以将分区字段看作表的伪列
     ```

  3. 加载数据到分区表中

     ```shell
     hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201709');
     hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201708');
     hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201707’);
     # 注意：分区表加载数据时，必须指定分区
     ```

  ![image-20191126224601149](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191126224601149.png)

  4. 查询分区表中数据

     ```shell
     # 单分区查询
     hive (default)> select * from dept_partition where month='201709';
     # 多分区联合查询
     hive (default)> select * from dept_partition where month='201709'
                   union
                   select * from dept_partition where month='201708'
                   union
                   select * from dept_partition where month='201707';
     
     _u3.deptno      _u3.dname       _u3.loc _u3.month
     10      ACCOUNTING      NEW YORK        201707
     10      ACCOUNTING      NEW YORK        201708
     10      ACCOUNTING      NEW YORK        201709
     20      RESEARCH        DALLAS  201707
     20      RESEARCH        DALLAS  201708
     20      RESEARCH        DALLAS  201709
     30      SALES   CHICAGO 201707
     30      SALES   CHICAGO 201708
     30      SALES   CHICAGO 201709
     40      OPERATIONS      BOSTON  201707
     40      OPERATIONS      BOSTON  201708
     40      OPERATIONS      BOSTON  201709
     ```

  5. 增加分区

     ```shell
     # 创建单个分区
     hive (default)> alter table dept_partition add partition(month='201706') ;
     # 同时创建多个分区
     hive (default)> alter table dept_partition add partition(month='201705') partition(month='201704');
     ```

  6. 删除分区

     ```shell
     # 删除单个分区
     hive (default)> alter table dept_partition drop partition (month='201704');
     # 同时删除多个分区
     hive (default)> alter table dept_partition drop partition (month='201705'), partition (month='201706');
     ```

  7. 查看分区表有多少分区

     ```shell
     hive> show partitions dept_partition;
     ```

  8. 查看分区表结构

     ```shell
     hive> desc formatted dept_partition;
     # Partition Information          
     # col_name              data_type               comment             
     month                   string   
     ```

* 分区表注意事项

  ```shell
  #1. 创建二级分区表
  hive (default)> create table dept_partition2(
                 deptno int, dname string, loc string
                 )
                 partitioned by (month string, day string)
                 row format delimited fields terminated by '\t';
  #2. 正常的加载数据
  #2.1 加载数据到二级分区表中
  hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table
   default.dept_partition2 partition(month='201709', day='13');
  #2.2 查询分区数据
  hive (default)> select * from dept_partition2 where month='201709' and day='13';
  #3. 把数据直接上传到分区目录上，让分区表和数据产生关联的三种方式
  #方式一：上传数据后修复
  # 上传数据
  hive (default)> dfs -mkdir -p
   /user/hive/warehouse/dept_partition2/month=201709/day=12;
  hive (default)> dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=12;
  # 查询数据（查询不到刚上传的数据）
  hive (default)> select * from dept_partition2 where month='201709' and day='12';
  # 执行修复命令
  hive> msck repair table dept_partition2;
  # 再次查询数据
  hive (default)> select * from dept_partition2 where month='201709' and day='12';
  # 方式二：上传数据后添加分区
  # 上传数据
  hive (default)> dfs -mkdir -p
   /user/hive/warehouse/dept_partition2/month=201709/day=11;
  hive (default)> dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=11;
  # 执行添加分区
  hive (default)> alter table dept_partition2 add partition(month='201709',
   day='11');
  # 查询数据
  hive (default)> select * from dept_partition2 where month='201709' and day='11';
  # 方式三：创建文件夹后load数据到分区
  # 创建目录
  hive (default)> dfs -mkdir -p
   /user/hive/warehouse/dept_partition2/month=201709/day=10;
   # 上传数据
   hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table
   dept_partition2 partition(month='201709',day='10');
   # 查询数据
   hive (default)> select * from dept_partition2 where month='201709' and day='10';
  ```

## 修改表

```shell
# 重命名表
# 语法
ALTER TABLE table_name RENAME TO new_table_name
# 实操案例
hive (default)> alter table dept_partition2 rename to dept_partition3;
# 增加、修改和删除表分区
# 增加/修改/替换列信息
# 语法
# 更新列
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
# 增加和替换列
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
# 注：ADD是代表新增一字段，字段位置在所有列后面(partition列前)，REPLACE则是表示替换表中所有字段
```

* 实操案例

  ```shell
  #1. 查询表结构
  hive> desc dept_partition;
  #2. 添加列
  hive (default)> alter table dept_partition add columns(deptdesc string);
  #3. 查询表结构
  hive> desc dept_partition;
  #4. 更新列
  hive (default)> alter table dept_partition change column deptdesc desc int;
  #5. 查询表结构
  hive> desc dept_partition;
  #6. 替换列
  hive (default)> alter table dept_partition replace columns(deptno string, dname
   string, loc string);
  #7. 查询表结构
  hive> desc dept_partition;
  ```

## 删除表

```shell
hive (default)> drop table dept_partition;
```

# DML 数据操作











