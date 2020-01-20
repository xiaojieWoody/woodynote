# Hive概述

## MapReduce的局限性

* 学习曲线陡峭，需Java基础
* 开发效率不高
* 发布和更新比较麻烦，不适合业务快速变化的场景

## 开源数据仓库工具

* 开发目的
  * 解决海量结构化的日志数据统计问题
* 声明式编程
  * 告诉框架你想要的是什么，让框架想出如何去做——SQL

## Hive环境选择

![image-20200111221731825](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111221731825.png)

```shell
􏰗􏱛􏱐􏰉[1] 下载链接 https://www.cloudera.com/downloads/quickstart_vms/5-13.html 
[2] https://github.com/big-data-europe/docker-hive
[3] 如果会用􏰊􏰆􏰚􏰐Docker Compose软件，更推荐􏱝􏰅􏰒􏰥􏰣􏱘Docker方式􏰛􏰨
[4] https://help.aliyun.com/document_detail/28129.html
[5] https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/emr-hive.html 
[6] https://docs.microsoft.com/zh-cn/azure/hdinsight/hadoop/hdinsight-use-hive
```

## 大数据整体架构

![image-20200111111339188](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111111339188.png)

## Hive架构

![image-20200111111418712](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111111418712.png)

## Hive数据流

![image-20200111111546000](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111111546000.png)

## Hive Server Web UI

* Hive 2.0 后增加了Web UI，方便查看Hive server2的状态
* hive-site.xml中增加配置
  * hive.server2.webui.host=0.0.0.0
  * hive.server2.webui.port=10002
* 可以查看Hive Server启动时的配置

## Hive Server Web UI Screen

![image-20200111111913435](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111111913435.png)

## Hive的用途

![image-20200111112007207](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111112007207.png)

# Hive使用场景

## 网站的访问日志分析

![image-20200111112332632](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111112332632.png)

## Apache HTTP服务器的访问日志

![image-20200111112638495](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111112638495.png)

![image-20200111113549789](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111113549789.png)

## 最流行的区域

![image-20200111113637743](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111113637743.png)

## Hive数据生命周期

![image-20200111203855195](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111203855195.png)

# Hive客户端详解

## 客户端分类

* 按协议分
  * JDBC客户端
    * Hue（Web）
    * squirrel-sql
    * Beeline CLI
  * ODBC客户端
    * Tableau Desktop
  * Thrift客户端（更底层）
    * Hive CLI
* 按使用方式
  * CLI客户端
  * 程序库或包
* 常见认证方式
  * None
  * Kerberos
  * LDAP（eg.Active Directory）

## Beeline CLI

* JDBC Client
* 一般工作在远程模式下，可以和Hive Server2不在同一台机器上
* 启动方式
  * beeline -u jdbc:hive2://<hive-server2>:10000/default -n <user> -p <pwd>
  * 前提是HiveServer2的认证方式是None（默认值）
* 更多启动参数
  * 执行SQL文件：beeline ... -f xxx.hql
  * 执行SQL：beeline ... -e 'show tables'
  * 设置hiveconf变量：beeline ... --hiveconf property=value --hiveconf ...
  * 设置hivevar变量（可替换sql文件里的变量）：beeline ... --hivevar var1=value1
* 退出
  * !quit（或!q）

## Beeline代码分析

* 常见使用问题
  * 连不上HiveServer
    * telnet检查端口是否是通的
  * 如何检查Hive默认设置
    * /etc/hive/conf
  * 开启无认证时，没有提供用户名而变成匿名用户，导致执行tez任务权限不够
  * 如何打印更多日志
* 代码位置
  * /usr/bin/beeline -> /usr/lib/hive/bin/ext/beeline.sh
  * 入口类：org.apache.hive.beeline.Beeline
    1. Beeline beeline = new Beeline() 添加配置 options.addOption 
       * -d driver class; -u jdbc url; -n username; -p password; -u authType等等
       * 通过BeeLineCommandCompeleter加载所有的命令（可以通过beeline help查看所有的命令 例如 !conect !q等等），这样就可以用到jline的tab补全等特性，历史信息
    2. beeLine.begin(args, inputStream)
       * initArgs(args)初始化输入的参数 --> 通过BeeLineParser进行解析。（解析-hivevar -hiveconf -d -u等等）此时如果url不为空，此时执行!connect操作
  * execute函数：处理命令输入
  * Dispatch函数
    1. 以!开头的命令
    2. 不以!开头的命令
  * Command:execute()执行SQL然后创建一个日志线程去获取server的日志（通过client FetchResults获取），当执行语句结束的时候，中断日志线程

## Hive CLI

* 最早的CLI工具和Hive Server(1)一起出现
* 主要为了操作Hive Server（1）和提供SQL查询CLI
* 随着Hive Server2的出现，已被Beeline所替代
* 最新版的Hive版本中，可以设置环境变量export USE_DEPRECATED_CLI=false来强制使用新版HiveCLI（底层和Beeline一样）
* 启动方式
  * 命令行输入hive
* 退出
  * exit;

## CLI安装

1. 先确定Hadoop的版本和Hive的版本，这里以Hadoop2.8 Hive 2.3为例, 事先安装好JDK，确定JAVA_HOME已设置
2. 下载Hadoop包： wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz
3. 解压后进入目录：export HADOOP_HOME=$(pwd)
4. 下载Hive包：wget http://mirror.bit.edu.cn/apache/hive/hive-2.3.4/apache-hive-2.3.4-bin.tar.gz
5. 解压后进入目录: export HIVE_HOME=$(pwd)
6. 把hive bin目录下的文件加入PATH： export PATH=$HIVE_HOME/bin:$PATH
7. 使用beeline -u jdbc:hive2://<hive-server-ip>:10000 -n hive -p 连接到Hive Server

## Java Client（无认证）

![image-20200111161841409](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111161841409.png)

## Python Client（无认证）

![image-20200111161929265](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111161929265.png)

# Hive编程介绍

## Hive DDL

* Hive CLI

  * 早期的命令行工具
  * 打开方式，需要配置
    * hive

  ```sql
  -- hive
  hive> show databases;
  hive> show tables;
  ```

* ==Beeline CLI==

  * 在配置好的Hive Client环境中，执行beeline命令进入
  * 打开方式
    * beeline -u jdbc:hive2://localhost:10000 -n hive -p hive

* CLI 工具等价于MySQL的客户端工具mysql命令

* 创建表

  ```sql
  CREATE TABLE [database_name.]my_table_name(
  	dummy_column STRING COMMENT 'xxx',
    another_column STRING
  )
  -- [database_name.]    -->   dafault
  ```

## 创建表

![image-20200111204258822](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111204258822.png)

![image-20200111162545406](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111162545406.png)

![image-20200111162621728](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111162621728.png)

* RegexSerDe可以处理有规则的非csv类文本，还有个常用的JSON序列化类是org.openx.data.jsonserde.JsonSerDe,可以处理一行一个JSON串的文件

## 字段类型

* 基本

![image-20200111162725906](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111162725906.png)

* 复杂

  ![image-20200111162812811](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111162812811.png)

## DESC指令

![image-20200111204414968](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111204414968.png)

## 分区表

![image-20200111204453942](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111204453942.png)

* 作用
  * 避免全表扫描
  * 提升查询性能
  * 方便数据按分区写入（覆盖）
  * 方便管理

## 数据写入分区表

![image-20200111204859004](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111204859004.png)

## 写分区表设置参数

![image-20200111204935799](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111204935799.png)

## 分区表演示

1. 创建分区表

   ```sql
   DROP TABLE IF EXISTS partitioned_access_logs;
   CREATE TABLE partitioned_access_logs (
       ip STRING,
       request_time STRING,
       method STRING,
       url STRING,
       http_version STRING,
       code1 STRING,
       code2 STRING,
       dash STRING,
       user_agent STRING,
       `timestamp` int)
   PARTITIONED BY (request_date STRING)
   STORED AS PARQUET
   ;
   ```

2. 往分区表写入数据

   ```sql
   set hive.exec.dynamic.partition.mode=nonstrict;
   
   INSERT OVERWRITE TABLE partitioned_access_logs 
   PARTITION (request_date)
   SELECT ip, request_time, method, url, http_version, code1, code2, dash, user_agent, `timestamp`, to_date(request_time) as request_date
   FROM pq_access_logs
   ;
   ```

   * 默认分区：__HIVE_DEFAULT_PARTITION__， 没有匹配上的记录会放在这个分区

3. 观察分区表物理存储结构

   ```shell
   hdfs dfs -ls /user/hive/warehouse/partitioned_access_logs
   ```

## 表分桶

![image-20200111205101348](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111205101348.png)

## 写入需要注意的地方

![image-20200111205138715](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111205138715.png)

## Sort的用处

![image-20200111205204610](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111205204610.png)

## 分桶表Demo

* 创建日志分桶表
  * 按IP地址的第一段分桶
  
    ```sql
    DROP TABLE IF EXISTS bucketed_access_logs;
    CREATE TABLE bucketed_access_logs (
        first_ip_addr INT,
        request_time STRING)
    CLUSTERED BY (first_ip_addr) 
    SORTED BY (request_time) 
    INTO 10 BUCKETS
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    STORED AS TEXTFILE
    ;
    
    ！如果DISTRIBUTE BY和SORT BY不写，则需要设置hive参数 (2.0后不用，默认为true)
    SET hive.enforce.sorting = true;
    SET hive.enforce.bucketing = true;
    
    INSERT OVERWRITE TABLE bucketed_access_logs 
    SELECT cast(split(ip, '\\.')[0] as int) as first_ip_addr, request_time
    FROM pq_access_logs
    --DISTRIBUTE BY first_ip_addr
    --SORT BY request_time
    ;
    ```
  
  * 桶内按request_time排序
  
* 观察分桶表的物理存储结构

  ```shell
  hdfs dfs -ls /user/hive/warehouse/bucketed_access_logs/
  # 猜猜有几个文件？
  
  hdfs dfs -cat /user/hive/warehouse/bucketed_access_logs/000000_0 | head
  
  hdfs dfs -cat /user/hive/warehouse/bucketed_access_logs/000001_0 | head
  
  hdfs dfs -cat /user/hive/warehouse/bucketed_access_logs/000009_0 | head
  
  # 能看出分桶的规则吗？
  ```

## 表的文件存储格式(File Format)

![image-20200111162903998](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111162903998.png)

## Record Columnar File（RCFile，列式存储）

![image-20200111205409495](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111205409495.png)

## 列式存储之压缩

![image-20200111205445366](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111205445366.png)

## 两种最常用列式存储格式

![image-20200113222033399](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200113222033399.png)

## ORC表演示

* 创建ORC表
  * 启用压缩
* 经验
  * 时间和空间的权衡
  * 依据实际测试

## 演示 - ORC表的压缩

1. 新建一张访问日志的ORC表，插入数据时启用压缩

   ```sql
   DROP TABLE IF EXISTS compressed_access_logs;
   CREATE TABLE compressed_access_logs (
       ip STRING,
       request_time STRING,
       method STRING,
       url STRING,
       http_version STRING,
       code1 STRING,
       code2 STRING,
       dash STRING,
       user_agent STRING,
       `timestamp` int)
   STORED AS ORC
   TBLPROPERTIES ("orc.compression"="SNAPPY");
   
   --SET hive.exec.compress.intermediate=true;
   --SET mapreduce.map.output.compress=true;
   
   INSERT OVERWRITE TABLE compressed_access_logs
   SELECT * FROM pq_access_logs;
   
   describe formatted compressed_access_logs;
   ```

2. 和原来不启用压缩的Parquet表进行比对

   * 大小

   * 原始TXT是38 MB

     ```shell
     hdfs dfs -ls /user/hive/warehouse/pq_access_logs/
     ```

   * Parquet无压缩: 4,158,592 (4.1 MB)

     ```shell
     hdfs dfs -ls /user/hive/warehouse/compressed_access_logs/
     ```

   * Orc压缩后: 1,074,404 (1.0 MB)
   * 压缩比: 约等于5:2  （4:1 - Parquet Raw: ORC Compressed)
   * 注意： 数据备份时建议启用压缩，数据读多的情况下，启用压缩不一定能带来查询性能提升。
   * --Parquet压缩后: 1604560

## 建表知识总结

![image-20200113222000174](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200113222000174.png)

## 其他表相关操作

* 删除表：DROP TABLE <TableName>;
* 列出当前数据库下所有表 
  * SHOW TABLES
* 查看建表语句
  * SHOW CREATE TABLE <TableName>
* 查看表的结构
  * DESC [FORMATTED] <TableName>
* 查看分区表的分区
  * SHOW PARTITIONS <TableName>

## Hive QL

* 查询：SELECT
* 过滤：WHERE
* 分组：GROUP BY
* 分组过滤：HAVING
* TOP：LIMIT
* 排序：ORDER BY，SORT BY
* 聚合函数：SUM()，MIN()，AVG()，MAX()
* 条件语句：CASE WHEN，IF
* 分区函数：Partition By

## Hive JOIN

![image-20200111205850317](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111205850317.png)

## Hive DML

![image-20200111210135127](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111210135127.png)

## 数据导入

![image-20200111163851469](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111163851469.png)

## 数据导出

![image-20200111163923258](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111163923258.png)

## 演示:日志表创建和数据导入

1. 将日志文件传到HDFS

   ```shell
   hdfs dfs -mkdir /user/hive/warehouse/original_access_logs_0104
   hdfs dfs -put access.log /user/hive/warehouse/original_access_logs_0104
   # 检查文件是否已正确拷贝
   hdfs dfs -ls /user/hive/warehouse/original_access_logs_0104
   ```

2. 建立Hive外部表对应于日志文件

   ```sql
   DROP TABLE IF EXISTS original_access_logs;
   CREATE EXTERNAL TABLE original_access_logs (
       ip STRING,
       request_time STRING,
       method STRING,
       url STRING,
       http_version STRING,
       code1 STRING,
       code2 STRING,
       dash STRING,
       user_agent STRING)
   ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe'
   WITH SERDEPROPERTIES (
       'input.regex' = '([^ ]*) - - \\[([^\\]]*)\\] "([^\ ]*) ([^\ ]*) ([^\ ]*)" (\\d*) (\\d*) "([^"]*)" "([^"]*)"',
       'output.format.string' = "%1$$s %2$$s %3$$s %4$$s %5$$s %6$$s %7$$s %8$$s %9$$s")
   LOCATION '/user/hive/warehouse/original_access_logs_0104';
   ```

3. 将TEXT表转换为PARQUET表

   ```sql
   DROP TABLE IF EXISTS pq_access_logs;
   CREATE TABLE pq_access_logs (
       ip STRING,
       request_time STRING,
       method STRING,
       url STRING,
       http_version STRING,
       code1 STRING,
       code2 STRING,
       dash STRING,
       user_agent STRING,
       `timestamp` int)
   STORED AS PARQUET;
   
   #ADD JAR /opt/cloudera/parcels/CDH/lib/hive/lib/hive-contrib.jar;
   
   INSERT OVERWRITE TABLE pq_access_logs
   SELECT 
     ip,
     from_unixtime(unix_timestamp(request_time, 'dd/MMM/yyyy:HH:mm:ss z'), 'yyyy-MM-dd HH:mm:ss z'),
     method,
     url,
     http_version,
     code1,
     code2,
     dash,
     user_agent,
     unix_timestamp(request_time, 'dd/MMM/yyyy:HH:mm:ss z')
   FROM original_access_logs;
   ```

4. 统计最多访问的5个IP
   1. 注意观察Hive Job拆分成Map Reduce Jon并执行
   2. 如何查看Hive MR Job执行的日志（通过JobID到Yarn的Resource Manager里查看）

   ```sql
   select ip, count(*) cnt
   from pq_access_logs
   group by ip
   order by cnt desc
   limit 5
   ```

# Hive函数介绍

## Hive操作符

* 内置操作符（Operators）

  ```shell
  =,!=,<,>,IS NULL,...	# 关系型操作符
  +,-,*,/, ...          # 算术型操作符
  AND,OR,IN,...         # 逻辑型操作符
  # 不支持NOT IN
  ```

## Hive函数分类

* 内置标准函数
  * 定义：org/apache/hadoop/hive/ql/exec/FunctionRegistry.java
  * 数学函数
  * 日期函数
  * 类型转换函数
  * 条件函数
  * 字符函数
* 内置聚合函数
* 内置表生成函数
* 自定义函数

## Hive函数

![image-20200111165257427](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111165257427.png)

* 内置函数（Functions）
  * math:round,oor,ceil,exp,log,...
  * date:to_date,from_unixtimestamp,year,...
  * conditional:if,isnull,case,coalesce,...
  * string:char,concat,lower,trim,repeat,...
* 内置聚合函数（Aggregate Functions）
  * count，sum，min，max，corr，...
* 内置表生成函数（Table-generating functions）
  * explode，posexplode，parse_url_tuple，...

## 各种函数的区别

![image-20200111170629406](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111170629406.png)

## 自定义函数开发

![image-20200111170713081](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111170713081.png)

## UDF实现

```java
import org.apache.hadoop.hive.ql.exec.UDF;
/**
* 自定义Hive函数，需要继承org.apache.hadoop.hive.ql.exec.UDF
* 并覆写 evaluate方法
*/
public class HelloStringExt extends UDF{
	public String evaluate(String strVal){
		return "Hello " + strVal; 
  } 
}
```

## 自定义函数部署和加载

![image-20200111180031370](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180031370.png)

## 表生成函数UDTF例子

![image-20200111180105257](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180105257.png)

## 窗口分析函数（Windowing and Analytics Functions）

![image-20200111180207396](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180207396.png)

## 窗口分析函数解析

![image-20200111180244063](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180244063.png)

## 分区表函数举例

![image-20200111180322906](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180322906.png)

## 函数总结

* 能解释UDF，UDAF，UDTF和PTF，知道如何使用它们
* 知道如何查看函数的文档
* 知道如何使用自定义函数

# 课后作业

* 将网站的访问日志写入Hive，然后
  * 统计访问量最多的10个URL
  * 统计网站两天内每天的PV和UV（IP + UserAgent的组合可以确定一个唯一的用户）
  * 统计网站访问最多的5个落地页（每个会话的第一次访问称为落地页）

## 演示:函数使用

1. 普通函数应用
   * 关联IP国家列表统计出访问最多的5个国家
2. 分区函数应用
   * 会话分析
     * 切分原则：同一个用户两次请求间隔时间超过20分钟

## 会话分析

![image-20200111180722420](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180722420.png)

## 会话分析逻辑

![image-20200111180756986](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111180756986.png)

# 总结

* Hive的基本架构

* Hive的应用场景之Web日志处理

* Hive SQL的基本语法

* Hive函数的使用方法

# 课后作业

* 将网站的访问日志导入Hive，然后
  1. 统计访问量最多的10个URL
  2. 统计网站两天内每天的PV和UV
     * IP + UserAgent的组合可以确定一个唯一的用户
  3. 统计网站访问最多的5个落地页
     * 每个会话的第一次访问称为落地页

# Hive组成介绍

## Hive整体架构

![image-20200111211409623](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111211409623.png)

## Hive数据流

![image-20200111211441877](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111211441877.png)

## Hive Server Web UI

* Hive 2.0后增加了Web UI，方便查看Hive server 2的状态
* hive-site.xml中增加配置
  * hive.server2.webui.host = 0.0.0.0
  *  hive.server2.webui.port = 10002
* 可以查看Hive Server启动时的配置

## Hive Server Web UI Screen

![image-20200111211742667](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111211742667.png)

# Hive Metastore详解

## 初始化megastore

* Metastore支持三种存储方式

  * 本地derby (只支持单一会话, 仅限测试) 
  * 本地数据库, 如mysql, postgres
  * 远程数据库 (推荐生产使用) 

* 本地元存储和远程元存储的区别是：

  * 本地元存储不需要单独起metastore服务，用的是跟hive在同一个进程里的metastore服务。

  * 远程元存储需要单独起metastore服务，然后每个客户端都在配置文件里配置连接到该metastore服

    务。远程元存储的metastore服务和hive运行在不同的进程里。

## Metastore配置

![image-20200111212006664](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212006664.png)

## 启动megastore服务

* **初始化Schema**
  * 先在mysql上创建好hive用户和hive数据库,并授权hive用户对hive数据库的所有权限
    * GRANT ALL ON hive.* TO 'hive'@'%';
  * 然后执行schema tool的init schema命令
    * /usr/lib/hive/bin/schematool --dbType mysql --initSchema
  * 在命令行执行/usr/lib/hive/bin/hive --service metastore

## **Metastore Schema**

![image-20200111212319041](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212319041.png)

## **客户端连接MetaStore**

![image-20200111212425864](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212425864.png)

# Hive查询性能优化

* 使用Join优化特性
  1. Hash Join
  2. (Sorted)Merge Join
  3. Nested Loop

* 2 > 1 > 3

## Join优化

![image-20200111212638010](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212638010.png)

![image-20200111212702938](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212702938.png)

![image-20200111212719809](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212719809.png)

![image-20200111212739912](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212739912.png)

![image-20200111212758416](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212758416.png)

## SMB Join演示

* 用户表关联订单表
* 统计每个用户的总消费金额
* 查看执行计划

## 中间临时数据的压缩

![image-20200111212912285](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111212912285.png)

## 查询的性能优化

* 使用Join优化特性
* IO优化
  * 减少读取数据量 -> 使用分区表
  * 使用列式存储
  * 启用中间临时数据压缩
* 增加并行度
  * 设置Mapper和Reducer的数量
  * 输入文件的数量和大小与Mapper的数量有关

# Apache Tez介绍

## Tez引擎介绍

![image-20200111213117564](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213117564.png)

## Tez组成

![image-20200111213148563](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213148563.png)

## Tez优化举例

![image-20200111213308272](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213308272.png)

![image-20200111213329244](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213329244.png)

## Tez UI

![image-20200111213357261](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213357261.png)

## Tez UI Demo

![image-20200111213426577](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213426577.png)

# 经验分享

## 知识点梳理

![image-20200111213514715](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213514715.png)

## 常用面试题

* Hive Metastore服务的作用
* Hive有几种执行引擎及其特点
* Order By和Sort By的区别
* Parquet或ORC文件格式的特点
*  Hive性能调优的基本方法
*  Hive建表时的[ROW FORMAT row_format]和[STORED AS file_format]的作用
*  Hive建表时InputFormat、OutputFormat与SerDe三者关系

## 课后作业

* 将网站的访问日志导入Hive，然后
  
  * 建立按访问时间的小时来分割的分区表
* 将视频网站的video表（3.3MB）和user表（36.5MB）的数据导入Hive，建立分桶表，然后统计
  * 找出用户中上传视频最多的10个用户的所有视频
  * 统计出含有最多视频的前10个类型
  * 筛选出每个类别中评分最高的前5个视频

* Video表的格式如下（原文件是tsv格式）:

  ![image-20200111213746683](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213746683.png)

* User表的格式如下（原文件是tsv格式）:

  ![image-20200111213808637](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200111213808637.png)



