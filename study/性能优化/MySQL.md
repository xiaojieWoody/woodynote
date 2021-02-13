![image-20210117122043028](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117122043028.png)

# 数据库优化的必要性

避免网站页面出现访问错误

​	数据库连接timeout产生页面5xx错误

​	慢查询造成页面无法加载

​	阻塞造成数据无法提交

增加数据库的稳定性

​	很多数据库问题都是由于低效的查询引起的

​	随着时间的推移，系统变得极其臃肿，数据库中的数据量越来越大，数据检索越来越困难，对整个系统带来的资源消耗也就越来越大，系统越发不稳定

优化用户体验

​	流畅的页面访问速度

​	良好的网站功能体验

# mysql数据库优化层面

==2-3视频==

成本：低 -> 高

效果：

![image-20210117122900841](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117122900841.png)

商业需求

​	不合理需求造成资源投入产出比过低

​	无用功能堆积使系统过度复杂影响整体性能

系统架构

​	数据库中存放的数据都是合适在数据库中存放的吗？

​	是否合理的利用了应用层Cache机制？

​	数据层实现都是最精简的吗？

SQL及索引

​	根据需求写出良好的SQL，并创建有效的索引，实现某一种需求可以多种写法，就要选择一种效率最高的写法，这个时候就要了解sql优化

​	sql优化的目的之一就是减少中间结果集，降低物理IO（如何优化select t1.id,t2.name from t1,t2 where t1.pid=t2.id;）

数据库表结构

​	根据数据库的范式，设计表结构，表结构设计的好坏直接关系到SQL语句的复杂度

​	适当的将表进行拆分，原本需要做join的查询只需要一张单表查询就可以了

系统配置

​	大多数运行在Linux机器上，如tcp连接数的限制、打开文件数的限制、安全性的限制，因此要对这些配置进行相应的优化

硬件配置优化

​	数据库主机的IO性能是需要最优先考虑的一个因素

​	数据库主机和普通的应用程序服务器相比，资源要相对集中很多，单台主机上所需要进行的计算量自然也就比较多，所以数据库主机的CPU处理能力也是一个重要因素

​	数据库主机的网络设备（一般指网卡等）的性能也可能会成为系统的瓶颈

# SQL及索引优化

mysql安装

先移除mariadb数据库：yum remove mariadb-libs.x86_64

mkdir /etc/mysql

cd /etc/mysql

// https://dev.mysql.com/downloads/repo/yum/

wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

添加到本地：yum localinstall mysql80-community-release-el7-3.noarch.rpm

正式安装：yum install mysql-community-server

启动测试：service mysqld start、service mysqld status

查看默认密码并且登陆：

​	查看：cat /var/log/mysqld.log | grep password

​	登陆：mysql -uroot -p [密码]

修改密码：

set global validate_password.policy=0;

set global validate_password.length=1;

ALTER USER "root"@"localhost" IDENTIFIED BY "123456";

查看数据库的版本：select @@version;

示例数据：

​	https://dev.mysql.com/doc/index-other.html

​	sakila database

​	source /xxx/xxx.sql

PowerDesign：逆向导出数据库物理模型

问题SQL筛查步骤：

​	检查慢查日志是否开启：show variables like 'slow_query_log';

​	检查慢日志路径：show variables like '%slow_query_log%';

​	show variables like '%slow_query%';

​	开启慢日志：set global slow_query_log=on;

​	慢日志判断标准（默认查询时间大于10s的sql语句）：show variables like 'long_query_time';

​	set global long_query_time=1;

​	慢日志测试：select sleep(12);

​	为了方便测试，所有查询都记录进慢日志：

​		show variables like '%log%';

​		设置开启即可：set global log_queries_not_using_indexes=on;

储存过程

```sql
create database test;
use test;
create table t1(id int,name varchar(25));

-- 存储过程
DROP PROCEDURE IF EXISTS pro_t1;
delimiter $$
create procedure pro_t1()
begin
declare i int;
set i=0;
while i<100000 do
				insert into t1 (id,name)
				values(i,CONCAT('smartan',i));
set i=i+1;
end while;
end
$$
delimiter ;
call pro_t1();
```

Jmeter压测：

​	mysql-connector-java-8.0.22.jar添加到JMeter的lib目录中

​	设置语言：option-language-Chinese

​	右键-添加-线程-线程组，线程数10

​	线程组-右键-添加-配置元件-JDBC Connection Configuration

​		pool：test

​		max connections：10

​		database url：jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC&characterEncoding=utf-8

​		jdbc driver class：com.mysql.jdbc.Driver

​		username：root

​		password：123456

​	线程组-右键-添加-取样器-JDBC Request：

​		JDBC Connection Configuration：test

​		Update Statement

​		update t1 set name='smartan1';	

​	线程组-右键-添加-监听器-查看结果树、聚合报告、用表格查看结果

​	运行

监听慢日志：

​	tail -f /var/lib/mysql/2016775328d8-slow.log

```shell
root@2016775328d8:/# tail -f /var/lib/mysql/2016775328d8-slow.log
# Time: 2021-01-17T09:04:35.469507Z
# User@Host: root[root] @ localhost []  Id:    37
# Query_time: 12.005003  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1610874275;
select sleep(12);
# Time: 2021-01-17T09:04:45.539247Z
# User@Host: root[root] @ localhost []  Id:    37
# Query_time: 3.316245  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1610874285;
select sleep(12);
```

MySQL慢查日志的存储格式解析：

​	第一行，SQL查询执行的时间

​	第二行，执行SQL查询的连接信息，用户和连接IP

​	第三行，记录了一些比较有用的信息，如下解析：

​		Query_time，这条SQL执行的时间，越长则越慢

​		Lock_time，在MySQL服务器阶段（不是在存储引擎阶段）等待表锁时间

​		Rows_sent，查询返回的行数

​		Rows_examined，查询检查的行数，越长就当然越费时间	

​	第四行，设置时间戳，没有实际意义，只是和第一行对应执行时间

​	第五行及后买呢所有行（第二个#Time:之前），执行的sql语句记录信息，因为sql可能会很长

# MySQL慢查询日志分析工具

## mysqldumpslow

如果开启了慢查询日志，就会产生大量的数据，然后就可以通过对日志的分析，生成分析报表，通过报表进行优化

用法：执行mysqldumpslow --help查看详细用法

注意：在mysql数据库所在的服务器上，而不是在mysql>命令行中

mysqldumpslow --help

![image-20210117153618230](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117153618230.png)

mysqldumpslow -v /var/lib/mysql/2016775328d8-slow.log

按照count排序：mysqldumpslow -s c /var/lib/mysql/2016775328d8-slow.log

按照时间排序：mysqldumpslow -s t /var/lib/mysql/2016775328d8-slow.log

按照平均时间排序：mysqldumpslow -s at /var/lib/mysql/2016775328d8-slow.log

查看慢查询日志的前10个：mysqldumpslow  -t 10  /var/lib/mysql/2016775328d8-slow.log

mysqldumpslow -s at -t  /var/lib/mysql/2016775328d8-slow.log

mysqldumpslow -s t -t 5  /var/lib/mysql/2016775328d8-slow.log

mysqldumpslow -s r  /var/lib/mysql/2016775328d8-slow.log

mysqldumpslow -s ar  /var/lib/mysql/2016775328d8-slow.log

优缺点：

​	是最常用的工具，通过安装mysql进行附带安装，但是该工具统计的结果比较少，对优化所提供的信息还是比较少，比如cpu，io等信息都没有

# 利用pt-query-digest利器查找三大类有问题的SQL

pt-query-digest是用于分析mysql慢查询的第一个第三方工具，它可以分析binlog、General log、slowlog，也可以通过SHOWPROCESSLIST或者通过tcpdump抓取的MySQL协议数据来进行分析。可以把分析结果输出到文件中，分析过程是先对查询语句的条件进行参数化，然后对参数化以后的查询进行分组统计，统计出各查询的执行时间、次数、占比等，可以借助分析结果找出问题进行优化

pt-query-digest本质是perl脚本，所以首先安装perl模块

​	yum install -y perl-CPAN perl-Time-HiRes

快速安装：

​	centos：

​		wget https://www.percona.com/downloads/percona-toolkit/3.2.0/binary/redhat/7/x86_64/percona-toolkit-3.2.0-1.2l7.x86_64.rpm 

​		yum localinstall -y percona-toolkit-3.2.0-1.el7.x86_64.rpm

​	ubuntu：

​		apt-get install percona-toolkit -y

检查是否安装完成：

​	执行 pt-query-digest --help

常用命令详解：

​	查看服务器信息：pt-summary

​	查看磁盘开销使用信息：pt-diskstats

​	查看mysql数据库信息：pt-mysql-summary --user=root --password=123456

​	分析慢查询日志（重点）：pt-query-digest /var/lib/mysql/2016775328d8-slow.log

![image-20210117162010082](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117162010082.png)

pt-query-digest --limit=100% /var/lib/mysql/2016775328d8-slow.log

![image-20210117162606212](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117162606212.png)

![image-20210117162812317](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117162812317.png)

总揽信息 - 具体每个查询信息

​	查找mysql的从库和同步状态：pt-slave-find --host=localhost --user=root --password=123456

​	查看mysql的死锁信息(把死锁信息存入表中)：pt-deadlock-logger --run-time=10 --interval=3 --create-dest-table --dest D=test,t=deadlocks u=root,p=123456

![image-20210117164849966](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117164849966.png)

​		第一个客户端：

​			use test;

​			show tables;

​			select * from deadlocks;

​			set autocommit=0;

​			select * from t1 where id=1 for update;

​		第二个客户端：

​			use test;

​			set autocommit=0;

​			select * from t2 where id=1 for update;

​		第一个客户端：

​			select * from t2 where id=1 for update;

​	从慢查询日志中分析索引使用情况：pt-index-usage --user=root --password=123456 --host=localhost /var/lib/mysql/2016775328d8-slow.log

​	从慢查找数据库表中重复的索引：pt-duplicate-key-checker --host=localhost --user=root --password=123456

​	查看mysql表和文件的当前活动IO开销（不要在高峰时用）：pt-ioprofile

​	查看不同mysql配置文件的差异（集群常用，双方都生效的变量）：pt-config-diff /etc/my.cnf /root/my_master.cnf

![image-20210117165519063](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117165519063.png)

​	pt-find查找mysql表和执行命令，示例如下：

​		查找数据库里大于1M的表：pt-find --user=root --password=123456 --tablesize +1M

​		查看表和索引大小并排序：pt-find --user=root --password=123456 --printf "%T\t%D.%N\n" | sort -rn

​	pt-kill杀掉符合标准的mysql进程，示例：

​		显示查询时间大于3秒的查询：pt-kill --user=root --password=123456 --busy-time 3 --print

​		kill掉大于3秒的查询：pt-kill --user=root --password=123456 --busy-time 3 --kill	![image-20210117170340471](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117170340471.png)

​	查看mysql授权（集群常用，授权复制），示例如下：

​		pt-show-grants --user=root --password=123456

​		pt-show-grants --user=root --password=123456 --separate --revoke

​	验证数据库复制的完整性（集群常用，主从复制后检验），示例：pt-table-checksum --user=root --password=123456

## 三大类有问题的SQL

### 查询次数多且每次查询占用时间长的SQL

通常为pt-query-digest分析的前几个查询，该工具可以很清楚的看出每个SQL执行的次数及百分比等信息，执行的次数多，占比比较大的SQL

pt-query-digest /var/lib/mysql/2016775328d8-slow.log

![image-20210117180402486](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117180402486.png)

### IO大的SQL

注意pt-query-digest分析中的Rows examine项，扫描的行数越多，IO越大

### 未命中索引的SQL

pt-query-digest分析中的Rows examine和Rows Send的对比。说明该SQL的索引命中率不高，对于这种SQL，要重点进行关注

# 通过explain分析SQL执行计划

SQL的执行计划反映出了SQL的执行效率，在执行的SQL前面加上explain即可

![image-20210117181153205](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117181153205.png)

id列

​	数字越大越先执行，如果数字一样大，那么就从上往下依次执行，id列为null就表示这是一个结果集，不需要使用它来进行查询

![image-20210117181716403](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117181716403.png)

select_type列

​	simple：表示不需要union操作或者不包含子查询的简单select查询，有连接查询时，外层的查询为simple，且只有一个

​	primary：一个需要union操作或者含有子查询的select，位于最外层的查询，select_type即为primary，且只有一个

![image-20210117185242160](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117185242160.png)

​	union：union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union	

​	union result：包含union的结果集，在union和union all语句中，因为它不需要参与查询，所以id字段为null

​	dependent union：与union一样，出现在union或union all语句中，但是这个查询要受到外部查询的影响

![image-20210117194346822](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117194346822.png)

​	subquery：除了from子句中包含的子查询外，其他地方出现的子查询都可能是subquery

![image-20210117194608659](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117194608659.png)

​	dependent subquery：与dependent union类似，表示这个subquery的查询要受到外部表查询的影响

![image-20210117195224976](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117195224976.png)

​	derived：from字句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select

![image-20210117195452513](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117195452513.png)

​	materialization：物化通过将子查询结果作为一个临时表来加快查询执行速度，正常来说是常驻内存，下次查询会再次引用临时表

table列

​	显示的查询表名，如果查询使用了别名，那么这里显示的是别名，如果不涉及对数据表的操作，那么这显示为null，如果显示为尖括号括起来的<derived N>就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。如果是尖括号括起来的<union M,N>，与<derived N>类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集

partitions

**type列**

​	system：表中只有一行数据或者是空表，且只能用于myisam和memory表，如果是Innodb引擎表，type列在这个情况通常都是all或者index	

​		show create table t4;

​		alter table t4 engine=myisam;

​	const：使用唯一索引或者主键，返回记录一定是1行记录的等值where条件时，通常type是const，其他数据库也叫做唯一索引扫描

​		alter table t4 engine=innodb;

​		alter table t4 add index idx_t4_id(id);

​		drop index idx_t4_id on t4;

​		alter table t4 add unique(id); 

​	eq_ref：出现在要连接多个表的查询计划中，驱动表循环获取数据，这行数据是第二个表的主键或者唯一索引，作为条件查询只返回一条数据，且必须为not null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref

​			驱动表 left join 被驱动表，相当于for循环，两层for

![image-20210117202300520](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117202300520.png)

​	ref：不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现

![image-20210117221101382](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210117221101382.png)

​	fulltext：全文索引检索，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引

​			create fulltext index ft_idx_t3_name_nickname on t3(name,nickname) with parser ngram;      // ngram解析器

​			explain select * from t3 where match(name,nickname) against('性能 棒');

![image-20210121082826749](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121082826749.png)

![image-20210121083017195](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083017195.png)

value为2个字符，改为一个字符：vi /etc/my.cnf   ngram_token_size=1	service mysqld restart

需删除重建

![image-20210121083342473](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083342473.png)

![image-20210121083421647](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083421647.png)

ref_or_null：与ref方法类似，只是增加了null值的比较，实际用的不多

![image-20210121083522159](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083522159.png)

![image-20210121083556083](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083556083.png)

![image-20210121083631338](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083631338.png)

==4.4==

![image-20210121083716119](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083716119.png)

![image-20210121083835379](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210121083835379.png)

​	all：全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录

​	type列总结：

​		依次性能从好到差：system，const，eq_ref，ref，fulltext，ref_or_null，unique_subquery，index_subquery，range，index_merge，index，ALL，除了all之外，其他的type都可以使用到索引，除了index_merge之外，其他的type只可以用到一个索引。一般来说，好的sql查询至少达到range级别，最好能达到ref

possible_keys：查询可能使用到的索引

key列：查询真正使用到的索引，select_type为index_merge时，这里可能出现两个以上的索引，其他的select_type这里只会出现一个

key_len：用于处理查询的索引长度，如果是单列索引，那就是整个索引长度，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列不会计算进去。留意下这个列的值，算一下你的多列索引总长度就知道有没有使用到所有的列了。另外，key_len只计算where条件用到的索引长度，而排序和分组就算用到了索引，也不会计算到key_len中

**ref列**：如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

rows列：这里是执行计划中估算的扫描行数，不是精确值

filtered列：使用explain extended时会出现这个列，5.7之后的版本默认就有这个字段，不需要使用explain extended了。这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数

Extra列：

​	no tables userd：不带from字句的查询或者From dual查询，例如explain select sleep(1)\G    explain select 1 from dual\G

​	NULL：查询的列末被索引覆盖，并且where筛选条件是索引的前导列，意味着用到了索引，但是部分字段未被索引覆盖，必须通过“回表”来实现，不是纯碎地用到了索引，也不是完全没用到索引	![image-20210131173316196](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131173316196.png)

​	using index：查询时不需要回表查询，直接通过索引就可以获取查询的数据

​	Using where：查询的列未被索引覆盖，where筛选条件非索引的前导列

![image-20210131173723003](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131173723003.png)

​	Using where Using index：查询的列被索引覆盖，并且where筛选条件是索引列之一但是不是索引的前导列，意味着无法直接通过索引查找来查询到符合条件的数据

![image-20210131174405897](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131174405897.png)

![image-20210131174436703](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131174436703.png)

​	Using index condition：与Using where类似，查询的列不完全被索引覆盖，where条件中是一个前导列的范围

![image-20210131174650925](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131174650925.png)

![image-20210131174722072](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131174722072.png)

​	using temporary：表示使用了临时表存储中间结果。临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来

![image-20210131175417330](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131175417330.png)

​	using filesort：mysql会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。此时mysql会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行信息。这种情况下一般也是要考虑使用索引来优化的

![image-20210131175535239](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131175535239.png)

​	using intersect：表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集

![image-20210131180106287](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131180106287.png)

​	using union：表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集

![image-20210131180440919](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210131180440919.png)

​	using sort_union和using sort_intersection：用and和or查询信息量大时，先查询主键，然后进行排序合并后返回结果集

​	firstmatch（tb_name）：5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个

​	loosescan(m..n)：5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个

# 慢查询优化思路及案例

慢查询的优化思路

​	优化更需要优化的SQL，优先高并发场景的SQL

​		每小时10000次，每次20个IO 最后只要18个io，每小时节约20000次io（优先优化）

​		每小时10次，每次20000个io，2000个io，2000 * 10 = 20000

​	定位优化对象的性能瓶颈，IO、CPU、网络带宽

​	明确的优化目标

​	从explain执行计划入手

​	**永远用小结果集驱动大的结果集**  

​	**尽可能在索引中完成排序**

​	**只取出自己需要的列，不要用select ***

​	**仅使用最有效的过滤条件**

​	**尽可能避免复杂的join和子查询**

​	**小心使用order by，group by，distinct语句（都需要排序）**

​	**合理设计并利用索引**

## **永远用小结果集驱动大的结果集**

join操作表小于百万级别

驱动表的定义：

​	当进行多表连接查询时，[驱动表]的定义为：

​		1）指定了联结条件时，满足查询条件的记录行数少的表为[驱动表]

​		2）未指定联接条件时，行数少的表为[驱动表]

mysql关联查询的概念

​	MySQL表关联的算法是Nest Loop Join，是通过驱动表的结果集作为循环基础数据，然后一条一条地通过该结果集中的数据作为过滤条件到下一个表中查询数据，最后合并结果

left join、right join、inner join

尽可能减少join语句中的循环总次数

优先优化内层循环

保证join语句中被驱动表上join条件字段已经被索引

无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，不要太吝惜join Buffer的设置

join的优化思路总结：

​	并发量太高的时候，系统整体性能可能会急剧下降

​	复杂的Join语句，所需要锁定的资源也就越多，所阻塞的其他线程也就越多

​	复杂的Query语句分拆成多个较为简单的Query语句分步执行

## 尽可能在索引中完成排序

order by字句中的字段加索引（扫描索引即可，内存中完成，逻辑io）

若不加索引的话可能会启用一个临时文件辅助排序（落盘，物理io）

## 只取自己需要的列，不用select *

如果取出的列过多，则传输给客户端的数据量必然很大，浪费带宽

若在排序的时候输出过多的列，则会浪费内存（Using filesort）

若在排序的时候输出过多的列，还有可能改变执行计划

## 仅使用最有效的过滤条件

where字句中条件越多越好吗？

若在多种条件下都使用了索引，那如何选择？

最终选择方案：key_len的长度决定使用哪个条件，优先长度最小的

## 尽可能避免复杂的join和子查询

## 小心使用order by，group by，distinct语句

order by排序原理及优化思路

​	order by排序可利用索引进行优化，order by子句中只要是索引的前导列（最左匹配原则）都可以使索引生效，可以直接在索引中排序，不需要在额外的内存或者文件中排序

​	不能利用索引避免额外排序的情况，例如：排序字段中有多个索引，排序顺序和索引键顺序不一致（非前导列）

order by排序算法

​	MySQL对于不能利用索引避免排序的SQL，数据库不得不自己实现排序功能以满足用户需求，此时SQL的执行计划中会出现“Using filesort”，这里需要注意的是filesort并不意味着就是文件排序，其实也有可能是内存排序，这个主要由sort_buffer_size参数与结果集大小确定。MySQL内部实现排序主要有3种方式，常规排序，优化排序和优先队列排序，主要涉及3种排序算法：快速排序、归并排序和堆排序

​	步骤：

  1. 从表t1中获取满足WHERE条件的记录

  2. 对于每条记录，将记录的主键 + 排序键（id，col2）取出放入sort buffer

  3. 如果sort buffer可以存放所有满足条件的（id，col2）对，则进行排序，否则sort buffer满后，进行排序并固化到临时文件中。（排序算法采用的是快速排序算法）

     show variables like '%sort_buffer_size%';

     show variables like '%read_rnd_buffer_size%';

  4. 若排序中产生了临时文件，需要利用归并排序算法，保证临时文件中记录是有序的

  5. 循环执行上述过程，直到所有满足条件的记录全部参与排序

  6. 扫描排序好的（id，col2）对，并利用id去捞取SELECT需要返回的列（col1，col2，col3）

  7. 将获取的结果集返回给用户

order by优化排序算法

​	常规排序方式除了排序本身，还需要额外两次IO。优化的排序方式相对于常规排序，减少了第二次IO。主要区别在于，放入sort buffer不是（id，col2），而是（col1，col2，col3）。由于sort buffer中包含了查询需要的所有字段，因此排序完成后可以直接返回，无需二次捞数据。这种方式的代价在于，同样大小的sort buffer，能存放的（col1，col2，col3）数目要小于（id，col2），如果sort buffer不够大，可能导致需要写临时文件，造成额外的IO

​	show variables like "%max_length_for_sort_data%";

​	默认4096，低于默认则会进行优化排序

order by优先队列排序算法

​	5.6及之后的版本针对Order by limit M,N语句，在空间层面做了优化，加入了一种新的排序方式——优先队列，这种方式采用堆排序实现。堆排序算法特征正好可以解limit M,N这类排序的问题，虽然仍然需要所有元素参与排序，但是只需要M+N个元组的sort buffer空间即可，对于M,N很小的场景，基本不会因为sort buffer不够而导致需要临时文件进行归并排序的问题。对于升序，采用大顶堆，最终堆中的元素组成了最小的N个元素，对于降序，采用小顶堆，最终堆中的元素组成了最大的N的元素

order by 排序不一致问题

​	针对limit M,N的语句采用了优先队列，而优先队列采用堆实现，比如上述的例子order by idc limit 0,3需要采用大小为3的大顶堆；limit 3,3需要采用大小为6的大顶堆。由于idc为3的记录有3条，而堆排序是非稳定的（对于相同的key值，无法保证排序后与排序前的位置一致），所以导致分页重复的现象。为了避免这个问题，可以在排序中加上唯一值，比如主键id，这样由于id是唯一的，确保参与排序的key值不相同

order by排序案例演示

​	演示sql及思路：

​		explain select idc,name from t3 where id > 2 and id < 10 order by idc,name,id\G

​		分别在查询字段，where条件，排序字段上做出各种可能的组合，主要就是看有无索引，索引在以上三个关注点上的生效情况

group by分组优化思路

​	group by本质上也同样需要进行排序操作（MySQL8优化了，默认不排序了），而且与order by相比，group by主要只是多了排序之后的分组操作，如果在分组的时候还使用了其他的一些聚合函数，那么还需要一些聚合函数的计算，所以，在group by的实现过程中，与group by一样也可以利用到索引

group by的三种实现类型

​	Loose Index Scan（松散的索引扫描）

​		扫描过程：先根据group by后面的字段进行分组，分组不需要读取所有索引的key，例如index(key1,key2,key3)，group by key1,key2，此时只要读取索引中的key1,key2。然后再根据where条件进行筛选

​	Tight Index Scan（紧凑的索引扫描）

​		扫描过程：紧凑索引扫描需要在扫描索引的时候，读取所有满足条件的索引键，然后再根据读取的数据来完成GROUP BY操作得到相应结果。区别就是紧凑索引扫描是先执行where操作，再进行分组，松散索引扫描刚好相反

​	Using temporary临时表实现（非索引扫描）

​		扫描过程：MySQL在进行GROUP BY操作的时候当MySQL Query Optimizer无法找到合适的索引可以利用的时候，就不得不先读取需要的数据，然后通过临时表来完成GROUP BY操作

group by分组案例演示

​	演示sql及思路：explain select idc,max(name)from t3 where id > 2 and id < 10 group by idc,name,id\G

​	和order by一样，分别在查询字段，where条件，分组字段上做出各种可能的组合，主要就是看有无索引，索引在以上三个关注点上的生效情况

distinct的实现及优化思路

​	distinct的原理：distinct实际上和GROUP BY的操作非常相似，在GROUP BY之后的每组中只取出一条记录而已，所以，DISTINCT的实现和GROUP BY的实现也基本差不多，同样可以通过松散索引扫描或者是紧凑索引扫描来实现，当然，在无法仅仅使用索引即能完成DISTINCT的时候，MySQL只能通过临时表来完成。但是，和GROUP BY有一点差别的是，DISTINCT并不需要进行排序

distinct案例演示

​	演示sql及思路：

​		explain select distinct name from t3 where idc=3\G（索引中完成，索引默认是排好序的）

​		explain select distinct name from t3 where idc > 1\G（非索引中完成，暗藏排序问题）

## 合理设计并利用索引

索引种类：

​	B-tree索引（mysql中使用最频繁的索引类型）

​	Hash索引（检索效率远高于B-tree索引，可以一次定位）

​	Fulltext索引（目前仅char，varchar，text这三种类型可以）

​	R-tree索引（比较少见，主要用于空间数据检索）

### B-tree索引

![image-20210210213628940](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210210213628940.png)

升序排序，非叶子结点中有数据

### B+tree索引

![image-20210212064846876](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210212064846876.png)

非叶子结点只存储键值对信息

所有的叶子结点之间有个双向指针

将非主键数据放入叶子结点中，使非叶子结点能够放更多数据，降低树的高度，IO次数变少

两个指针（两种查找算法）

​	第一个指向根节点：对主键的范围查找或者分页查找

​	第二个指向最小叶子结点：从根节点的随机查找

分为：

​	**聚簇索引**：主键，叶子结点中存放的是整个表中行数据，主键效率高是因为不需要回表查找，根据主键就能查找到数据

​	**辅助索引**：叶子结点并不包含行记录的所有数据，存储的是行记录的具体索引值（主键信息），再根据主键值回表查找具体数据

## 如何判断是否需要创建所有

较频繁的作为查询条件的字段应该创建索引

唯一性太差的字段不适合单独创建索引，可以尝试复合索引

更新非常频繁的字段不适合创建索引

不会出现在where子句中的字段不该创建索引

## 索引失效与优化

复合索引尽量全匹配

最佳左前缀法则（带头索引不能死，中间索引不能断）

不要在索引上做任何操作（计算、函数、自动/手动类型转换），不然会导致索引失效而转向全表扫描

MySQL存储引擎不能继续使用索引中范围条件（between、<、>、in等）右边的列

尽量使用覆盖索引（只查询索引的列（索引列和查询列一致）），减少select *

索引字段上使用（!= 或 < >）判断时，会导致索引失效而转向全表扫描

索引字段上使用is null / is not null判断时，会导致索引失效而转向全表扫描

索引字段使用like以通配符开头（%字符串）时，会导致索引失效而转向全表扫描

索引字段是字符串，但查询时不加单引号，会导致索引失效而转向全表扫描

索引字段使用or时，会导致索引失效而转向全表扫描

## 优化终极奥义

针对百万数量级，放弃在mysql中的join操作，推荐分别根据索引单表取数据，然后在程序里面做join，merge数据

尽量使用nosql，例如redis，memcached等来缓存热点数据，从而缓解mysql压力

# 数据库其他优化原则

总体优化原则：

​	不在数据库做运算，运算务必移至业务层

​	库命名简介明确（长度不能超过30个字符）

​	控制列数量（字段少而精，字段数建议在20以内）

​	平衡范式与冗余（效率优先；往往牺牲范式）

​	拒绝3B（拒绝大sql语句：big sql、拒绝大事务：big transaction、拒绝大批量：big batch）

字段类优化原则：

​	用好数值类型（用合适的字段类型节约空间）

​	字符转化为数字（能转化的最好转化，同样节约空间，提高查询性能）

​	避免使用NULL字段（NULL字段很难查询优化、NULL字段的索引需要额外空间、NULL字段的复合索引无效）

​	少用text类型（尽量使用varchar代替text字段）

索引类优化原则：

​	合理使用索引（改善查询，减慢更新，索引一定不是越多越好）

​	字符字段建前缀索引（例如：abckk,dfgkk,fdskk...只要前面3个）

​	不在索引做列运算（例如：select * from t1 where id + 1 = 10）

​	innodb主键推荐使用自增列（主键建立聚簇索引，主键不应该被修改，字符串不应该做主键）（理解Innodb的索引保存结构就知道了）；不用外键（由程序保证约束）

SQL类优化原则：

​	sql语句尽可能简单（一条sql只能在一个cpu运算，大语句拆小语句，减少锁时间，一条大sql可以堵死整个库）

​	简单的事务（最好是不要有事务）

​	避免使用trig/func（不用触发器、函数。客户端程序取而代之）

​	不用select * （消耗cpu,io,内存,带宽,这种程序不具有扩展性）

​	OR改写为IN（在字段没有索引的情况下性能差别较大）

​	OR改写为UNION（索引无效变有效）

​	使用union all替代union（union有去重开销，例如分表操作explain select name from t3 where idc <=2 union select name from t3 where idc=3\G）

​	少用连接join（超过3个join，一般移到业务代码里执行）

​	分页limit优化（偏移量越大，执行越慢）

​	select * from t1 where id in(select id from t1 where id>90000) limit 0,20;

结构类优化原则：

​	表范式化原则：

​		范式化是指数据库设计的规范，目前范式化一般是指设计到第三范式。也就是要求数据表中不存在非关键字段对任意候选关键字段的传递函数依赖，则符合第三范式

![image-20210213120604911](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213120604911.png)



​	反范式化原则：

​		反范式化是指为了查询效率的考虑把原本符合第三范式的表“适当”的增加冗余，以达到优化查询效率的目的，反范式化是一种以空间来换取时间的操作

![image-20210213120918886](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213120918886.png)

![image-20210213120949842](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213120949842.png)

![image-20210213121045814](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213121045814.png)

​		反范式化原则（反范式化之后的sql语句）

​			select a.用户名,a.电话,a.地址,a.订单ID,a.订单价格 from 订单表 as a

​			适当的字段冗余，减少了表之间的join（三表join->单表查询）

​	垂直拆分原则：

1. 不常用的字段单独存放到一个表中

2. 大字段独立存放到一个表中

3. 经常一起使用的字段放到一起

   垂直拆分原则（例子：sakila库的film表）

   1. title和description比较常用，也比较大，可以拆出来独立一张表

   2. film表 -> film表（film_id,其他字段）+ film_ext表（film_id,title,description）

​    水平拆分原则：

1. 表的水平拆分是为了解决单表数据量过大的问题
2. 尽管加了完美的索引，查询效率低，写入的效率也相应的降低
3. 通常对id进行hash运算，如果要拆分为5个表则使用mod（id,5）取出0-4个值

​	





