# 基础

* 作用

  * 主要用于在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递，可以将一个关系型数据库（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中

* 原理

  * 将导入或导出命令翻译成mapreduce程序来实现
  * 在翻译出的mapreduce中主要是对inputformat和outputformat进行定制

* 安装

  ```shell
  # 解压安装包到指定目录
  $ tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
  # 修改配置文件（sqoop根目录下的conf目录中）
  #1. 重命名配置文件
  $ mv sqoop-env-template.sh sqoop-env.sh
  #2. 修改配置文件sqoop-env.sh
  export HADOOP_COMMON_HOME=/opt/module/hadoop-2.7.2
  export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.2
  export HIVE_HOME=/opt/module/hive
  export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.10
  export ZOOCFGDIR=/opt/module/zookeeper-3.4.10
  export HBASE_HOME=/opt/module/hbase
  #3. 拷贝jdbc驱动到sqoop的lib目录下
  $ cp mysql-connector-java-5.1.27-bin.jar /opt/module/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/lib/
  #4. 通过某一个command来验证sqoop配置是否正确
  $ bin/sqoop help
  ```

* 测试是否能连接数据库

  ```shell
  bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 000000
  ```

# 导入数据

* 从非大数据集群（RDBMS）向大数据集群（HDFS，HIVE，HBASE）中传输数据

## RDBMS到HDFS

1. 确定Mysql服务开启正常

2. 确定Mysql服务开启正常

   ```sql
   mysql -uroot -p000000
   mysql> create database company;
   mysql> create table company.staff(id int(4) primary key not null auto_increment, name varchar(255), sex varchar(255));
   mysql> insert into company.staff(name, sex) values('Thomas', 'Male');
   mysql> insert into company.staff(name, sex) values('Catalina', 'FeMale')
   ```

3. 导入数据

   * 全部导入

     ```shell
     bin/sqoop import \
     --connect jdbc:mysql://hadoop102:3306/company \
     --username root \
     --password 000000 \
     --table staff \
     --target-dir /user/company \
     --delete-target-dir \
     --num-mappers 1 \
     --fields-terminated-by "\t"
     ```

   * 查询导入

     ```shell
     bin/sqoop import \
     --connect jdbc:mysql://hadoop102:3306/company \
     --username root \
     --password 000000 \
     --target-dir /user/company \
     --delete-target-dir \
     --num-mappers 1 \
     --fields-terminated-by "\t" \
     --query 'select name,sex from staff where id <=1 and $CONDITIONS;'
     ```

     * 提示：must contain '$CONDITIONS' in WHERE clause
       * 如果query后使用的是双引号，则$CONDITIONS前必须加转移符，防止shell识别为自己的变量

   * 导入指定列

     ```shell
     bin/sqoop import \
     --connect jdbc:mysql://hadoop102:3306/company \
     --username root \
     --password 000000 \
     --target-dir /user/company \
     --delete-target-dir \
     --num-mappers 1 \
     --fields-terminated-by "\t" \
     --columns id,sex \
     --table staff
     ```

     * 提示：columns中如果涉及到多列，用逗号分隔，分隔时不要添加空格

   * 使用sqoop关键字筛选查询导入数据

     ```shell
     $ bin/sqoop import \
     --connect jdbc:mysql://hadoop102:3306/company \
     --username root \
     --password 000000 \
     --target-dir /user/company \
     --delete-target-dir \
     --num-mappers 1 \
     --fields-terminated-by "\t" \
     --table staff \
     --where "id=1"
     ```

## RDBMS到Hive

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-table staff_hive
```

* 提示：该过程分为两步，第一步将数据导入到HDFS，第二步将导入到HDFS的数据迁移到Hive仓库，第一步默认的临时目录是/user/atguigu/表名

##  RDBMS到Hbase

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--columns "id,name,sex" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_company" \
--num-mappers 1 \
--split-by id
```

* 提示：sqoop1.4.6只支持HBase1.0.1之前的版本的自动创建HBase表的功能

* 解决方案：手动创建HBase表

  ```sql
  hbase> create 'hbase_company','info'
  ```

  * 在HBase中scan这张表得到如下内容

    ```sql
    hbase> scan 'hbase_company'
    ```

# 导出数据

* 从大数据集群（HDFS，HIVE，HBASE）向非大数据集群（RDBMS）中传输数据

## HIVE/HDFS到RDBMS

```shell
$ bin/sqoop export \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--export-dir /user/hive/warehouse/staff_hive \
--input-fields-terminated-by "\t"
```

* 提示：Mysql中如果表不存在，不会自动创建

## 脚本打包

* 使用opt格式的文件打包sqoop命令，然后执行

1. 使用opt格式的文件打包sqoop命令，然后执行

   ```shell
   $ mkdir opt
   $ touch opt/job_HDFS2RDBMS.opt
   ```

2. 编写sqoop脚本

   ```shell
   $ vi opt/job_HDFS2RDBMS.opt
   
   export
   --connect
   jdbc:mysql://hadoop102:3306/company
   --username
   root
   --password
   000000
   --table
   staff
   --num-mappers
   1
   --export-dir
   /user/hive/warehouse/staff_hive
   --input-fields-terminated-by
   "\t"
   ```

3. 执行该脚本

   ```shell
   $ bin/sqoop --options-file opt/job_HDFS2RDBMS.opt
   ```

# 常用命令

* import

  > ImportTool，将数据导入到集群

* export

  > ExportTool，将集群数据导出

* codegen

  > CodeGenTool，获取数据库中某张表数据生成Java并打包Jar

* create-hive-table

  > CreateHiveTableTool，创建Hive表

* eval

  > EvalSqlTool，查看SQL执行结果

* import-all-tables

  > ImportAllTablesTool，导入某个数据库下所有表到HDFS中

* job

  > JobTool，用来生成一个sqoop的任务，生成后，该任务并不执行，除非使用命令执行该任务

* list-databases

  > ListDatabasesTool，列出所有数据库名

* list-tables

  > ListTablesTool，列出某个数据库下所有表

* merge

  > MergeTool，将HDFS中不同目录下面的数据合在一起，并存放在指定的目录中

* metastore

  > MetastoreTool，记录sqoop job的元数据信息，如果不启动metastore实例，则默认的元数据存储目录为：~/.sqoop，如果要更改存储目录，可以在配置文件sqoop-site.xml中进行更改

* help

  > HelpTool，打印sqoop帮助信息

* version

  > VersionTool，打印sqoop版本信息

# 参数

## 公用参数

### 数据库连接

```shell
# 连接关系型数据库的URL
--connect
# 指定要使用的连接管理类
--connection-manager
# Hadoop根目录
--driver
# 打印帮助信息
--help
# 连接数据库的密码
--password
# 连接数据库的用户名
--username
# 在控制台打印出详细信息
--verbose
```

### import

```shell
# 给字段值前加上指定的字符
--enclosed-by <char>
# 对字段中的双引号加转义符
--escaped-by <char>
# 设定每个字段是以什么符号作为结束，默认为逗号
--fields-terminated-by <char>
# 设定每行记录之间的分隔符，默认是\n
--lines-terminated-by <char>
# Mysql默认的分隔符设置，字段之间以逗号分隔，行之间以\n分隔，默认转义符是\，字段值以单引号包裹
--mysql-delimiters
# 给带有双引号或单引号的字段值前后加上指定字符。
--optionally-enclosed-by <char>
```

### export

```shell
# 对字段值前后加上指定字符
--input-enclosed-by <char>
# 对含有转移符的字段做转义处理
--input-escaped-by <char>
# 字段之间的分隔符
--input-fields-terminated-by <char>
# 行之间的分隔符
--input-lines-terminated-by <char>
# 给带有双引号或单引号的字段前后加上指定字符
--input-optionally-enclosed-by <char>
```

### hive

```shell
# 用自定义的字符串替换掉数据中的\r\n和\013 \010等字符
--hive-delims-replacement <arg>
# 在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符
--hive-drop-import-delims
# 生成hive表时，可以更改生成字段的数据类型
--map-column-hive <arg>
# 创建分区，后面直接跟分区名，分区字段的默认类型为string
--hive-partition-key
# 导入数据时，指定某个分区的值
--hive-partition-value <v>
# hive的安装目录，可以通过该参数覆盖之前默认配置的目录
--hive-home <dir>
# 将数据从关系数据库中导入到hive表中
--hive-import
# 覆盖掉在hive表中已经存在的数据
--hive-overwrite
# 默认是false，即，如果目标表已经存在了，那么创建任务失败。
--create-hive-table
# 后面接要创建的hive表,默认使用MySQL的表名
--hive-table
# 指定关系数据库的表名
--table
```

## 命令参数

### import

* 将关系型数据库中的数据导入到HDFS（包括Hive，HBase）中，如果导入的是Hive，那么当Hive中没有对应表时，则自动创建

* 导入数据到hive中

  ```shell
  $ bin/sqoop import \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --table staff \
  --hive-import
  ```

* 增量导入数据到hive中，mode=append

  ```shell
  # append导入：
  $ bin/sqoop import \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --table staff \
  --num-mappers 1 \
  --fields-terminated-by "\t" \
  --target-dir /user/hive/warehouse/staff_hive \
  --check-column id \
  --incremental append \
  --last-value 3
  ```

  * 尖叫提示：append不能与--hive-等参数同时使用（Append mode for hive imports is not yet supported. Please remove the parameter --append-mode）

* 增量导入数据到hdfs中，mode=lastmodified

  1. 先在mysql中建表并插入几条数据：

     ```sql
     mysql> create table company.staff_timestamp(id int(4), name varchar(255), sex varchar(255), last_modified timestamp DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP);
     mysql> insert into company.staff_timestamp (id, name, sex) values(1, 'AAA', 'female');
     mysql> insert into company.staff_timestamp (id, name, sex) values(2, 'BBB', 'female');
     ```

  2. 先导入一部分数据

     ```shell
     $ bin/sqoop import \
     --connect jdbc:mysql://hadoop102:3306/company \
     --username root \
     --password 000000 \
     --table staff_timestamp \
     --delete-target-dir \
     --m 1
     ```

  3. 再增量导入一部分数据

     ```shell
     mysql> insert into company.staff_timestamp (id, name, sex) values(3, 'CCC', 'female');
     $ bin/sqoop import \
     --connect jdbc:mysql://hadoop102:3306/company \
     --username root \
     --password 000000 \
     --table staff_timestamp \
     --check-column last_modified \
     --incremental lastmodified \
     --last-value "2017-09-28 22:20:38" \
     --m 1 \
     --append
     ```

     * 尖叫提示：使用lastmodified方式导入数据要指定增量数据是要--append（追加）还是要--merge-key（合并）
     * 尖叫提示：last-value指定的值是会包含于增量导入的数据中

* 参数

  ```shell
  # 将数据追加到HDFS中已存在的DataSet中，如果使用该参数，sqoop会把数据先导入到临时文件目录，再合并。
  --append
  # 将数据导入到一个Avro数据文件中
  --as-avrodatafile
  # 将数据导入到一个sequence文件中
  --as-sequencefile
  # 将数据导入到一个普通文本文件中
  --as-textfile
  # 边界查询，导入的数据为该参数的值（一条sql语句）所执行的结果区间内的数据
  --boundary-query <statement>
  # 指定要导入的字段
  --columns <col1, col2, col3>
  # 直接导入模式，使用的是关系数据库自带的导入导出工具，以便加快导入导出过程
  --direct
  # 在使用上面direct直接导入的基础上，对导入的流按字节分块，即达到该阈值就产生一个新的文件
  --direct-split-size
  # 设定大对象数据类型的最大值
  --inline-lob-limit
  # 启动N个map来并行导入数据，默认4个。
  --m或–num-mappers
  # 将查询结果的数据导入，使用时必须伴随参--target-dir，--hive-table，如果查询中有where条件，则条件后必须加上$CONDITIONS关键字
  --query或--e <statement>
  # 按照某一列来切分表的工作单元，不能与--autoreset-to-one-mapper连用（请参考官方文档）
  --split-by <column-name>
  # 关系数据库的表名
  --table <table-name>
  # 指定HDFS路径
  --target-dir <dir>
  # 与--target-dir <dir>参数不能同时使用，导入数据到HDFS时指定的目录
  --warehouse-dir <dir>
  # 从关系数据库导入数据时的查询条件
  --where
  # 允许压缩
  --z或--compress
  # 指定hadoop压缩编码类，默认为gzip(Use Hadoop codec default gzip)
  --compression-codec
  # string类型的列如果null，替换为指定字符串
  --null-string <null-string>
  # 非string类型的列如果null，替换为指定字符串
  --null-non-string <null-string>
  # 作为增量导入判断的列名
  --check-column <col>
  # mode：append或lastmodified
  --incremental <mode>
  # 指定某一个值，用于标记增量导入的位置
  --last-value <value>
  ```

### export

* 从HDFS（包括Hive和HBase）中将数据导出到关系型数据库中

* 命令

  ```shell
  $ bin/sqoop export \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --table staff \
  --export-dir /user/company \
  --input-fields-terminated-by "\t" \
  --num-mappers 1
  ```

* 参数

  ```shell
  # 利用数据库自带的导入导出工具，以便于提高效率
  --direct
  # 存放数据的HDFS的源目录
  --export-dir <dir>
  # 启动N个map来并行导入数据，默认4个
  -m或--num-mappers <n>
  # 指定导出到哪个RDBMS中的表
  --table <table-name>
  # 对某一列的字段进行更新操作
  --update-key <col-name>
  # updateonly	allowinsert(默认)
  --update-mode <mode>
  # 请参考import该类似参数说明
  --input-null-string <null-string>
  # 请参考import该类似参数说明
  --input-null-non-string <null-string>
  # 创建一张临时表，用于存放所有事务的结果，然后将所有事务结果一次性导入到目标表中，防止错误。
  --staging-table <staging-table-name>
  # 如果--staging-table参数非空，则可以在导出操作执行前，清空临时事务结果表
  --clear-staging-table
  ```

### codegen

* 将关系型数据库中的表映射为一个Java类，在该类中有各列对应的各个字段

  ```shell
  $ bin/sqoop codegen \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --table staff \
  --bindir /home/admin/Desktop/staff \
  --class-name Staff \
  --fields-terminated-by "\t"
  ```

  ```shell
  # 指定生成的Java文件、编译成的class文件及将生成文件打包为jar的文件输出路径
  --bindir <dir>
  # 设定生成的Java文件指定的名称
  --class-name <name>
  # 生成Java文件存放的路径
  --outdir <dir>
  # 包名，如com.z，就会生成com和z两级目录
  --package-name <name>
  # 在生成的Java文件中，可以将null字符串或者不存在的字符串设置为想要设定的值（例如空字符串）
  --input-null-non-string <null-str>
  # 将null字符串替换成想要替换的值（一般与5同时使用）
  --input-null-string <null-str>
  # 数据库字段在生成的Java文件中会映射成各种属性，且默认的数据类型与数据库类型保持对应关系。该参数可以改变默认类型，例如：--map-column-java id=long, name=String
  --map-column-java <arg>
  # 在生成Java文件时，可以将不存在或者null的字符串设置为其他值
  --null-non-string <null-str>
  # 在生成Java文件时，将null字符串设置为其他值（一般与8同时使用）
  --null-string <null-str>
  # 对应关系数据库中的表名，生成的Java文件中的各个属性与该表的各个字段一一对应
  --table <table-name>
  ```

### create-hive-table

* 生成与关系数据库表结构对应的hive表结构

  ```shell
  $ bin/sqoop create-hive-table \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --table staff \
  --hive-table hive_staff
  ```

  ```shell
  # Hive的安装目录，可以通过该参数覆盖掉默认的Hive目录
  --hive-home <dir>
  # 覆盖掉在Hive表中已经存在的数据
  --hive-overwrite
  # 默认是false，如果目标表已经存在了，那么创建任务会失败
  --create-hive-table
  # 后面接要创建的hive表
  --hive-table
  # 指定关系数据库的表名
  --table
  ```

### eval

* 可以快速的使用SQL语句对关系型数据库进行操作，经常用于在import数据之前，了解一下SQL语句是否正确，数据是否正常，并可以将结果显示在控制台

  ```shell
  $ bin/sqoop eval \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --query "SELECT * FROM staff"
  ```

  ```shell
  # 后跟查询的SQL语句
  --query或--e
  ```

### import-all-tables

* 可以将RDBMS中的所有表导入到HDFS中，每一个表都对应一个HDFS目录

  ```shell
  $ bin/sqoop import-all-tables \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --warehouse-dir /all_tables
  ```

  ```shell
  # 这些参数的含义均和import对应的含义一致
  --as-avrodatafile
  --as-sequencefile
  --as-textfile
  --direct
  --direct-split-size <n>
  --inline-lob-limit <n>
  --m或—num-mappers <n>
  --warehouse-dir <dir>
  -z或--compress
  --compression-codec
  ```

### job

* 用来生成一个sqoop任务，生成后不会立即执行，需要手动执行

  ```shell
  $ bin/sqoop job \
   --create myjob -- import-all-tables \
   --connect jdbc:mysql://hadoop102:3306/company \
   --username root \
   --password 000000
  $ bin/sqoop job \
  --list
  $ bin/sqoop job \
  --exec myjob
  ```
  * 尖叫提示：注意import-all-tables和它左边的--之间有一个空格
  * 尖叫提示：如果需要连接metastore，则--meta-connect jdbc:hsqldb:hsql://linux01:16000/sqoop

  ```shell
  # 创建job参数
  --create <job-id>
  # 删除一个job
  --delete <job-id>
  # 执行一个job
  --exec <job-id>
  # 显示job帮助
  --help
  # 显示job列表
  --list
  # 用来连接metastore服务
  --meta-connect <jdbc-uri>
  # 显示一个job的信息
  --show <job-id>
  # 打印命令运行时的详细信息
  --verbose
  ```

  * 尖叫提示：在执行一个job时，如果需要手动输入数据库密码，可以做如下优化

  ```xml
  <property>
  	<name>sqoop.metastore.client.record.password</name>
  	<value>true</value>
  	<description>If true, allow saved passwords in the metastore.</description>
  </property>
  ```

### list-databases

```shell
$ bin/sqoop list-databases \
--connect jdbc:mysql://hadoop102:3306/ \
--username root \
--password 000000
```

### list-tables

```shell
$ bin/sqoop list-tables \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000
```

### merge

* 将HDFS中不同目录下面的数据合并在一起并放入指定目录中

  ```shell
  # 数据环境
  new_staff
  1       AAA     male
  2       BBB     male
  3       CCC     male
  4       DDD     male
  old_staff
  1       AAA     female
  2       CCC     female
  3       BBB     female
  6       DDD     female
  ```

  * 尖叫提示：上边数据的列之间的分隔符应该为\t，行与行之间的分割符为\n，如果直接复制，请检查之

  ```shell
  # 创建JavaBean
  $ bin/sqoop codegen \
  --connect jdbc:mysql://hadoop102:3306/company \
  --username root \
  --password 000000 \
  --table staff \
  --bindir /home/admin/Desktop/staff \
  --class-name Staff \
  --fields-terminated-by "\t"
  
  # 开始合并
  $ bin/sqoop merge \
  --new-data /test/new/ \
  --onto /test/old/ \
  --target-dir /test/merged \
  --jar-file /home/admin/Desktop/staff/Staff.jar \
  --class-name Staff \
  --merge-key id
  
  # 结果
  1	AAA	MALE
  2	BBB	MALE
  3	CCC	MALE
  4	DDD	MALE
  6	DDD	FEMALE
  ```

  ```shell
  # HDFS 待合并的数据目录，合并后在新的数据集中保留
  --new-data <path>
  # HDFS合并后，重复的部分在新的数据集中被覆盖
  --onto <path>
  # 合并键，一般是主键ID
  --merge-key <col>
  # 合并时引入的jar包，该jar包是通过Codegen工具生成的jar包
  --jar-file <file>
  # 对应的表名或对象名，该class类是包含在jar包中的
  --class-name <class>
  # 合并后的数据在HDFS里存放的目录
  --target-dir <path>
  ```

### metastore

* 记录了Sqoop job的元数据信息，如果不启动该服务，那么默认job元数据的存储目录为~/.sqoop，可在sqoop-site.xml中修改

  ```shell
  # 启动sqoop的metastore服务
  $ bin/sqoop metastore
  # 关闭metastore
  --shutdown
  ```

  





