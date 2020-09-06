# 性能监控

## 使用show profile查询剖析工具，可以指定具体的type

* 此工具默认是禁用的，可以通过服务器变量在会话级别动态的修改，`set profiling=1;`，当设置完成之后，在服务器上执行的所有语句，都会测量其耗费的时间和其他一些查询执行状态变更相关的数据。`select * from emp;`，在mysql的命令行模式下只能显示两位小数的时间，可以使用如下命令查看具体的执行时间，`show profiles;`，执行如下命令可以查看详细的每个步骤的时间：`show profile for query 1;`

* type

  ```sql
  -- all：显示所有性能信息s
  show profile all for query n
  -- block io：显示块io操作的次数
  show  profile block io for query n
  -- context switches：显示上下文切换次数，被动和主动
  show profile context switches for query n
  -- cpu：显示用户cpu时间、系统cpu时间
  show profile cpu for query n
  -- IPC：显示发送和接受的消息数量
  show profile ipc for query n
  -- Memory：暂未实现
  -- page faults：显示页错误数量
  show profile page faults for query n
  -- source：显示源码中的函数名称与位置
  show profile source for query n
  -- swaps：显示swap的次数
  show profile swaps for query n
  ```

* 查询SQL语句执行时间

  ```shell
  # 5.7版本，后面版本将过期
  set profiling=1;
  select * from mylock;
  show profiles; 
  show profile;
  show profile cpu;
  show profile all;
  
  # 选择某条语句
  show profile for query ${QueryID}
  show profile;
  
  # 查看监控表
  show databases;
  use performance_schema;
  show tables;
  SHOW VARIABLES LIKE 'performance_schema'
  # 开启/关闭
  vi /etc/my.cnf
  
  # 当前数据库连接信息
  show processlist;
  ```

## 使用performance schema来更加容易的监控mysql

* performance_schema监控当前服务器上MySQL性能情况

  ![image-20200901080858020](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901080858020.png)

### MYSQL performance schema详解

#### 0、performance_schema的介绍

* **MySQL的performance schema 用于监控MySQL server在一个较低级别的运行过程中的资源消耗、资源等待等情况**。
* 特点如下：
  1. 提供了一种在数据库运行时实时检查server的内部执行情况的方法。performance_schema 数据库中的表使用performance_schema存储引擎。该数据库主要关注数据库运行过程中的性能相关的数据，与information_schema不同，information_schema主要关注server运行过程中的元数据信息
  2. performance_schema通过监视server的事件来实现监视server内部运行情况， “事件”就是server内部活动中所做的任何事情以及对应的时间消耗，利用这些信息来判断server中的相关资源消耗在了哪里？一般来说，事件可以是函数调用、操作系统的等待、SQL语句执行的阶段（如sql语句执行过程中的parsing 或 sorting阶段）或者整个SQL语句与SQL语句集合。事件的采集可以方便的提供server中的相关存储引擎对磁盘文件、表I/O、表锁等资源的同步调用信息
  3. performance_schema中的事件与写入二进制日志中的事件（描述数据修改的events）、事件计划调度程序（这是一种存储程序）的事件不同。performance_schema中的事件记录的是server执行某些活动对某些资源的消耗、耗时、这些活动执行的次数等情况
  4. performance_schema中的事件只记录在本地server的performance_schema中，其下的这些表中数据发生变化时不会被写入binlog中，也不会通过复制机制被复制到其他server中
  5. 当前活跃事件、历史事件和事件摘要相关的表中记录的信息。能提供某个事件的执行次数、使用时长。进而可用于分析某个特定线程、特定对象（如mutex或file）相关联的活动
  6. PERFORMANCE_SCHEMA存储引擎使用server源代码中的“检测点”来实现事件数据的收集。对于performance_schema实现机制本身的代码没有相关的单独线程来检测，这与其他功能（如复制或事件计划程序）不同
  7. 收集的事件数据存储在performance_schema数据库的表中。这些表可以使用SELECT语句查询，也可以使用SQL语句更新performance_schema数据库中的表记录（如动态修改performance_schema的setup_*开头的几个配置表，但要注意：配置表的更改会立即生效，这会影响数据收集）
  8. performance_schema的表中的数据不会持久化存储在磁盘中，而是保存在内存中，一旦服务器重启，这些数据会丢失（包括配置表在内的整个performance_schema下的所有数据）
  9. MySQL支持的所有平台中事件监控功能都可用，但不同平台中用于统计事件时间开销的计时器类型可能会有所差异

#### 1. performance schema入门

* 在mysql的5.7版本中，性能模式是默认开启的，如果想要显式的关闭的话需要修改配置文件，不能直接进行修改，会报错Variable 'performance_schema' is a read only variable

  ```sql
  --查看performance_schema的属性
  mysql> SHOW VARIABLES LIKE 'performance_schema';
  +--------------------+-------+
  | Variable_name      | Value |
  +--------------------+-------+
  | performance_schema | ON    |
  +--------------------+-------+
  1 row in set (0.01 sec)
  
  --在配置文件中修改performance_schema的属性值，on表示开启，off表示关闭
  [mysqld]
  performance_schema=ON
  
  --切换数据库
  use performance_schema;
  
  --查看当前数据库下的所有表,会看到有很多表存储着相关的信息
  show tables;
  
  --可以通过show create table tablename来查看创建表的时候的表结构
  mysql> show create table setup_consumers;
  +-----------------+---------------------------------
  | Table           | Create Table                    
  +-----------------+---------------------------------
  | setup_consumers | CREATE TABLE `setup_consumers` (
    `NAME` varchar(64) NOT NULL,                      
    `ENABLED` enum('YES','NO') NOT NULL               
  ) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8 |  
  +-----------------+---------------------------------
  1 row in set (0.00 sec)                             
  ```

* 想要搞明白后续的内容，需要理解两个基本概念：
  * instruments: 生产者，用于采集mysql中各种各样的操作产生的事件信息，对应配置表中的配置项可以称为监控采集配置项
  * consumers:消费者，对应的消费者表用于存储来自instruments采集的数据，对应配置表中的配置项可以称为消费存储配置项

#### 2、performance_schema表的分类

* performance_schema库下的表可以按照监视不同的纬度就行分组

  ```sql
  --语句事件记录表，这些表记录了语句事件信息，当前语句事件表events_statements_current、历史语句事件表events_statements_history和长语句历史事件表events_statements_history_long、以及聚合后的摘要表summary，其中，summary表还可以根据帐号(account)，主机(host)，程序(program)，线程(thread)，用户(user)和全局(global)再进行细分)
  show tables like '%statement%';
  
  --等待事件记录表，与语句事件类型的相关记录表类似：
  show tables like '%wait%';
  
  --阶段事件记录表，记录语句执行的阶段事件的表
  show tables like '%stage%';
  
  --事务事件记录表，记录事务相关的事件的表
  show tables like '%transaction%';
  
  --监控文件系统层调用的表
  show tables like '%file%';
  
  --监视内存使用的表
  show tables like '%memory%';
  
  --动态对performance_schema进行配置的配置表
  show tables like '%setup%';
  ```

#### 3、performance_schema的简单配置与使用

* 数据库刚刚初始化并启动时，并非所有instruments(事件采集项，在采集项的配置表中每一项都有一个开关字段，或为YES，或为NO)和consumers(与采集项类似，也有一个对应的事件类型保存表配置项，为YES就表示对应的表保存性能数据，为NO就表示对应的表不保存性能数据)都启用了，所以默认不会收集所有的事件，可能你需要检测的事件并没有打开，需要进行设置，可以使用如下两个语句打开对应的instruments和consumers（行计数可能会因MySQL版本而异)

  ```sql
  --打开等待事件的采集器配置项开关，需要修改setup_instruments配置表中对应的采集器配置项
  UPDATE setup_instruments SET ENABLED = 'YES', TIMED = 'YES'where name like 'wait%';
  
  --打开等待事件的保存表配置开关，修改setup_consumers配置表中对应的配置项
  UPDATE setup_consumers SET ENABLED = 'YES'where name like '%wait%';
  
  --当配置完成之后可以查看当前server正在做什么，可以通过查询events_waits_current表来得知，该表中每个线程只包含一行数据，用于显示每个线程的最新监视事件
  select * from events_waits_current\G
  *************************** 1. row ***************************
              THREAD_ID: 11
               EVENT_ID: 570
           END_EVENT_ID: 570
             EVENT_NAME: wait/synch/mutex/innodb/buf_dblwr_mutex
                 SOURCE: 
            TIMER_START: 4508505105239280
              TIMER_END: 4508505105270160
             TIMER_WAIT: 30880
                  SPINS: NULL
          OBJECT_SCHEMA: NULL
            OBJECT_NAME: NULL
             INDEX_NAME: NULL
            OBJECT_TYPE: NULL
  OBJECT_INSTANCE_BEGIN: 67918392
       NESTING_EVENT_ID: NULL
     NESTING_EVENT_TYPE: NULL
              OPERATION: lock
        NUMBER_OF_BYTES: NULL
                  FLAGS: NULL
  /*该信息表示线程id为11的线程正在等待buf_dblwr_mutex锁，等待事件为30880
  属性说明：
  	id:事件来自哪个线程，事件编号是多少
  	event_name:表示检测到的具体的内容
  	source:表示这个检测代码在哪个源文件中以及行号
  	timer_start:表示该事件的开始时间
  	timer_end:表示该事件的结束时间
  	timer_wait:表示该事件总的花费时间
  注意：_current表中每个线程只保留一条记录，一旦线程完成工作，该表中不会再记录该线程的事件信息
  */
  
  /*
  _history表中记录每个线程应该执行完成的事件信息，但每个线程的事件信息只会记录10条，再多就会被覆盖，*_history_long表中记录所有线程的事件信息，但总记录数量是10000，超过就会被覆盖掉
  */
  select thread_id,event_id,event_name,timer_wait from events_waits_history order by thread_id limit 21;
  
  /*
  summary表提供所有事件的汇总信息，该组中的表以不同的方式汇总事件数据（如：按用户，按主机，按线程等等）。例如：要查看哪些instruments占用最多的时间，可以通过对events_waits_summary_global_by_event_name表的COUNT_STAR或SUM_TIMER_WAIT列进行查询（这两列是对事件的记录数执行COUNT（*）、事件记录的TIMER_WAIT列执行SUM（TIMER_WAIT）统计而来）
  */
  SELECT EVENT_NAME,COUNT_STAR FROM events_waits_summary_global_by_event_name  ORDER BY COUNT_STAR DESC LIMIT 10;
  
  /*
  instance表记录了哪些类型的对象会被检测。这些对象在被server使用时，在该表中将会产生一条事件记录，例如，file_instances表列出了文件I/O操作及其关联文件名
  */
  select * from file_instances limit 20; 
  ```

#### 4、常用配置项的参数说明

1. 启动选项

   ```shell
   performance_schema_consumer_events_statements_current=TRUE
   是否在mysql server启动时就开启events_statements_current表的记录功能(该表记录当前的语句事件信息)，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新setup_consumers配置表中的events_statements_current配置项，默认值为TRUE
   
   performance_schema_consumer_events_statements_history=TRUE
   与performance_schema_consumer_events_statements_current选项类似，但该选项是用于配置是否记录语句事件短历史信息，默认为TRUE
   
   performance_schema_consumer_events_stages_history_long=FALSE
   与performance_schema_consumer_events_statements_current选项类似，但该选项是用于配置是否记录语句事件长历史信息，默认为FALSE
   
   除了statement(语句)事件之外，还支持：wait(等待)事件、state(阶段)事件、transaction(事务)事件，他们与statement事件一样都有三个启动项分别进行配置，但这些等待事件默认未启用，如果需要在MySQL Server启动时一同启动，则通常需要写进my.cnf配置文件中
   performance_schema_consumer_global_instrumentation=TRUE
   是否在MySQL Server启动时就开启全局表（如：mutex_instances、rwlock_instances、cond_instances、file_instances、users、hostsaccounts、socket_summary_by_event_name、file_summary_by_instance等大部分的全局对象计数统计和事件汇总统计信息表 ）的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新全局配置项
   默认值为TRUE
   
   performance_schema_consumer_statements_digest=TRUE
   是否在MySQL Server启动时就开启events_statements_summary_by_digest 表的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新digest配置项
   默认值为TRUE
   
   performance_schema_consumer_thread_instrumentation=TRUE
   是否在MySQL Server启动时就开启
   
   events_xxx_summary_by_yyy_by_event_name表的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新线程配置项
   默认值为TRUE
   
   performance_schema_instrument[=name]
   是否在MySQL Server启动时就启用某些采集器，由于instruments配置项多达数千个，所以该配置项支持key-value模式，还支持%号进行通配等，如下:
   
   # [=name]可以指定为具体的Instruments名称（但是这样如果有多个需要指定的时候，就需要使用该选项多次），也可以使用通配符，可以指定instruments相同的前缀+通配符，也可以使用%代表所有的instruments
   
   ## 指定开启单个instruments
   
   --performance-schema-instrument= 'instrument_name=value'
   
   ## 使用通配符指定开启多个instruments
   
   --performance-schema-instrument= 'wait/synch/cond/%=COUNTED'
   
   ## 开关所有的instruments
   
   --performance-schema-instrument= '%=ON'
   
   --performance-schema-instrument= '%=OFF'
   
   注意，这些启动选项要生效的前提是，需要设置performance_schema=ON。另外，这些启动选项虽然无法使用show variables语句查看，但我们可以通过setup_instruments和setup_consumers表查询这些选项指定的值。
   ```

2. 系统变量

   ```sql
   show variables like '%performance_schema%';
   --重要的属性解释
   performance_schema=ON
   /*
   控制performance_schema功能的开关，要使用MySQL的performance_schema，需要在mysqld启动时启用，以启用事件收集功能
   该参数在5.7.x之前支持performance_schema的版本中默认关闭，5.7.x版本开始默认开启
   注意：如果mysqld在初始化performance_schema时发现无法分配任何相关的内部缓冲区，则performance_schema将自动禁用，并将performance_schema设置为OFF
   */
   
   performance_schema_digests_size=10000
   /*
   控制events_statements_summary_by_digest表中的最大行数。如果产生的语句摘要信息超过此最大值，便无法继续存入该表，此时performance_schema会增加状态变量
   */
   performance_schema_events_statements_history_long_size=10000
   /*
   控制events_statements_history_long表中的最大行数，该参数控制所有会话在events_statements_history_long表中能够存放的总事件记录数，超过这个限制之后，最早的记录将被覆盖
   全局变量，只读变量，整型值，5.6.3版本引入 * 5.6.x版本中，5.6.5及其之前的版本默认为10000，5.6.6及其之后的版本默认值为-1，通常情况下，自动计算的值都是10000 * 5.7.x版本中，默认值为-1，通常情况下，自动计算的值都是10000
   */
   performance_schema_events_statements_history_size=10
   /*
   控制events_statements_history表中单个线程（会话）的最大行数，该参数控制单个会话在events_statements_history表中能够存放的事件记录数，超过这个限制之后，单个会话最早的记录将被覆盖
   全局变量，只读变量，整型值，5.6.3版本引入 * 5.6.x版本中，5.6.5及其之前的版本默认为10，5.6.6及其之后的版本默认值为-1，通常情况下，自动计算的值都是10 * 5.7.x版本中，默认值为-1，通常情况下，自动计算的值都是10
   除了statement(语句)事件之外，wait(等待)事件、state(阶段)事件、transaction(事务)事件，他们与statement事件一样都有三个参数分别进行存储限制配置，有兴趣的同学自行研究，这里不再赘述
   */
   performance_schema_max_digest_length=1024
   /*
   用于控制标准化形式的SQL语句文本在存入performance_schema时的限制长度，该变量与max_digest_length变量相关(max_digest_length变量含义请自行查阅相关资料)
   全局变量，只读变量，默认值1024字节，整型值，取值范围0~1048576
   */
   performance_schema_max_sql_text_length=1024
   /*
   控制存入events_statements_current，events_statements_history和events_statements_history_long语句事件表中的SQL_TEXT列的最大SQL长度字节数。 超出系统变量performance_schema_max_sql_text_length的部分将被丢弃，不会记录，一般情况下不需要调整该参数，除非被截断的部分与其他SQL比起来有很大差异
   全局变量，只读变量，整型值，默认值为1024字节，取值范围为0~1048576，5.7.6版本引入
   降低系统变量performance_schema_max_sql_text_length值可以减少内存使用，但如果汇总的SQL中，被截断部分有较大差异，会导致没有办法再对这些有较大差异的SQL进行区分。 增加该系统变量值会增加内存使用，但对于汇总SQL来讲可以更精准地区分不同的部分。
   */
   ```

#### 5、重要配置表的相关说明

* 配置表之间存在相互关联关系，按照配置影响的先后顺序，可添加为

  ```sql
  /*
  performance_timers表中记录了server中有哪些可用的事件计时器
  字段解释：
  	timer_name:表示可用计时器名称，CYCLE是基于CPU周期计数器的定时器
  	timer_frequency:表示每秒钟对应的计时器单位的数量,CYCLE计时器的换算值与CPU的频率相关、
  	timer_resolution:计时器精度值，表示在每个计时器被调用时额外增加的值
  	timer_overhead:表示在使用定时器获取事件时开销的最小周期值
  */
  select * from performance_timers;
  
  /*
  setup_timers表中记录当前使用的事件计时器信息
  字段解释：
  	name:计时器类型，对应某个事件类别
  	timer_name:计时器类型名称
  */
  select * from setup_timers;
  
  /*
  setup_consumers表中列出了consumers可配置列表项
  字段解释：
  	NAME：consumers配置名称
  	ENABLED：consumers是否启用，有效值为YES或NO，此列可以使用UPDATE语句修改。
  */
  select * from setup_consumers;
  
  /*
  setup_instruments 表列出了instruments 列表配置项，即代表了哪些事件支持被收集：
  字段解释：
  	NAME：instruments名称，instruments名称可能具有多个部分并形成层次结构
  	ENABLED：instrumetns是否启用，有效值为YES或NO，此列可以使用UPDATE语句修改。如果设置为NO，则这个instruments不会被执行，不会产生任何的事件信息
  	TIMED：instruments是否收集时间信息，有效值为YES或NO，此列可以使用UPDATE语句修改，如果设置为NO，则这个instruments不会收集时间信息
  */
  SELECT * FROM setup_instruments;
  
  /*
  setup_actors表的初始内容是匹配任何用户和主机，因此对于所有前台线程，默认情况下启用监视和历史事件收集功能
  字段解释：
  	HOST：与grant语句类似的主机名，一个具体的字符串名字，或使用“％”表示“任何主机”
  	USER：一个具体的字符串名称，或使用“％”表示“任何用户”
  	ROLE：当前未使用，MySQL 8.0中才启用角色功能
  	ENABLED：是否启用与HOST，USER，ROLE匹配的前台线程的监控功能，有效值为：YES或NO
  	HISTORY：是否启用与HOST， USER，ROLE匹配的前台线程的历史事件记录功能，有效值为：YES或NO
  */
  SELECT * FROM setup_actors;
  
  /*
  setup_objects表控制performance_schema是否监视特定对象。默认情况下，此表的最大行数为100行。
  字段解释：
  	OBJECT_TYPE：instruments类型，有效值为：“EVENT”（事件调度器事件）、“FUNCTION”（存储函数）、“PROCEDURE”（存储过程）、“TABLE”（基表）、“TRIGGER”（触发器），TABLE对象类型的配置会影响表I/O事件（wait/io/table/sql/handler instrument）和表锁事件（wait/lock/table/sql/handler instrument）的收集
  	OBJECT_SCHEMA：某个监视类型对象涵盖的数据库名称，一个字符串名称，或“％”(表示“任何数据库”)
  	OBJECT_NAME：某个监视类型对象涵盖的表名，一个字符串名称，或“％”(表示“任何数据库内的对象”)
  	ENABLED：是否开启对某个类型对象的监视功能，有效值为：YES或NO。此列可以修改
  	TIMED：是否开启对某个类型对象的时间收集功能，有效值为：YES或NO，此列可以修改
  */
  SELECT * FROM setup_objects;
  
  /*
  threads表对于每个server线程生成一行包含线程相关的信息，
  字段解释：
  	THREAD_ID：线程的唯一标识符（ID）
  	NAME：与server中的线程检测代码相关联的名称(注意，这里不是instruments名称)
  	TYPE：线程类型，有效值为：FOREGROUND、BACKGROUND。分别表示前台线程和后台线程
  	PROCESSLIST_ID：对应INFORMATION_SCHEMA.PROCESSLIST表中的ID列。
  	PROCESSLIST_USER：与前台线程相关联的用户名，对于后台线程为NULL。
  	PROCESSLIST_HOST：与前台线程关联的客户端的主机名，对于后台线程为NULL。
  	PROCESSLIST_DB：线程的默认数据库，如果没有，则为NULL。
  	PROCESSLIST_COMMAND：对于前台线程，该值代表着当前客户端正在执行的command类型，如果是sleep则表示当前会话处于空闲状态
  	PROCESSLIST_TIME：当前线程已处于当前线程状态的持续时间（秒）
  	PROCESSLIST_STATE：表示线程正在做什么事情。
  	PROCESSLIST_INFO：线程正在执行的语句，如果没有执行任何语句，则为NULL。
  	PARENT_THREAD_ID：如果这个线程是一个子线程（由另一个线程生成），那么该字段显示其父线程ID
  	ROLE：暂未使用
  	INSTRUMENTED：线程执行的事件是否被检测。有效值：YES、NO 
  	HISTORY：是否记录线程的历史事件。有效值：YES、NO * 
  	THREAD_OS_ID：由操作系统层定义的线程或任务标识符（ID）：
  */
  select * from threads
  ```

* 注意：在performance_schema库中还包含了很多其他的库和表，能对数据库的性能做完整的监控，大家需要参考官网详细了解。

#### 6、performance_schema实践操作

* 基本了解了表的相关信息之后，可以通过这些表进行实际的查询操作来进行实际的分析

  ```sql
  --1、哪类的SQL执行最多？
  SELECT DIGEST_TEXT,COUNT_STAR,FIRST_SEEN,LAST_SEEN FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
  --2、哪类SQL的平均响应时间最多？
  SELECT DIGEST_TEXT,AVG_TIMER_WAIT FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
  --3、哪类SQL排序记录数最多？
  SELECT DIGEST_TEXT,SUM_SORT_ROWS FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
  --4、哪类SQL扫描记录数最多？
  SELECT DIGEST_TEXT,SUM_ROWS_EXAMINED FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
  --5、哪类SQL使用临时表最多？
  SELECT DIGEST_TEXT,SUM_CREATED_TMP_TABLES,SUM_CREATED_TMP_DISK_TABLES FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
  --6、哪类SQL返回结果集最多？
  SELECT DIGEST_TEXT,SUM_ROWS_SENT FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
  --7、哪个表物理IO最多？
  SELECT file_name,event_name,SUM_NUMBER_OF_BYTES_READ,SUM_NUMBER_OF_BYTES_WRITE FROM file_summary_by_instance ORDER BY SUM_NUMBER_OF_BYTES_READ + SUM_NUMBER_OF_BYTES_WRITE DESC
  --8、哪个表逻辑IO最多？
  SELECT object_name,COUNT_READ,COUNT_WRITE,COUNT_FETCH,SUM_TIMER_WAIT FROM table_io_waits_summary_by_table ORDER BY sum_timer_wait DESC
  --9、哪个索引访问最多？
  SELECT OBJECT_NAME,INDEX_NAME,COUNT_FETCH,COUNT_INSERT,COUNT_UPDATE,COUNT_DELETE FROM table_io_waits_summary_by_index_usage ORDER BY SUM_TIMER_WAIT DESC
  --10、哪个索引从来没有用过？
  SELECT OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME FROM table_io_waits_summary_by_index_usage WHERE INDEX_NAME IS NOT NULL AND COUNT_STAR = 0 AND OBJECT_SCHEMA <> 'mysql' ORDER BY OBJECT_SCHEMA,OBJECT_NAME;
  --11、哪个等待事件消耗时间最多？
  SELECT EVENT_NAME,COUNT_STAR,SUM_TIMER_WAIT,AVG_TIMER_WAIT FROM events_waits_summary_global_by_event_name WHERE event_name != 'idle' ORDER BY SUM_TIMER_WAIT DESC
  --12-1、剖析某条SQL的执行情况，包括statement信息，stege信息，wait信息
  SELECT EVENT_ID,sql_text FROM events_statements_history WHERE sql_text LIKE '%count(*)%';
  --12-2、查看每个阶段的时间消耗
  SELECT event_id,EVENT_NAME,SOURCE,TIMER_END - TIMER_START FROM events_stages_history_long WHERE NESTING_EVENT_ID = 1553;
  --12-3、查看每个阶段的锁等待情况
  SELECT event_id,event_name,source,timer_wait,object_name,index_name,operation,nesting_event_id FROM events_waits_history_longWHERE nesting_event_id = 1553;
  ```

## 使用show processlist查看连接的线程个数，来观察是否有大量线程处于不正常的状态或者其他不正常的特征

* 属性说明

  ```shell
  id表示session id
  user表示操作的用户
  host表示操作的主机
  db表示操作的数据库
  command表示当前状态：
  # sleep：线程正在等待客户端发送新的请求
  # query：线程正在执行查询或正在将结果发送给客户端
  # locked：在mysql的服务层，该线程正在等待表锁
  # analyzing and statistics：线程正在收集存储引擎的统计信息，并生成查询的执行计划
  # Copying to tmp table：线程正在执行查询，并且将其结果集都复制到一个临时表中
  # sorting result：线程正在对结果集进行排序
  # sending data：线程可能在多个状态之间传送数据，或者在生成结果集或者向客户端返回数据
  info表示详细的sql语句
  time表示相应命令执行时间
  state表示命令执行状态
  ```

# schema与数据类型优化

## 数据类型的优化

### 更小的通常更好

* 应该尽量使用可以正确存储数据的最小数据类型，更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期更少，但是要确保没有低估需要存储的值的范围，如果无法确认哪个数据类型，就选择你认为不会超过范围的最小类型

* 案例：设计两张表，设计不同的数据类型，查看表的容量

  ```java
  import java.sql.Connection;
  import java.sql.DriverManager;
  import java.sql.PreparedStatement;
  
  public class Test {
      public static void main(String[] args) throws Exception{
          Class.forName("com.mysql.jdbc.Driver");
          Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db1","root","123456");
          PreparedStatement pstmt = conn.prepareStatement("insert into psn2 values(?,?)");
          for (int i = 0; i < 20000; i++) {
              pstmt.setInt(1,i);
              pstmt.setString(2,i+"");
              pstmt.addBatch();
          }
          pstmt.executeBatch();
          conn.close();
      }
  }
  ```

### 简单就好

* 简单数据类型的操作通常需要更少的CPU周期，例如，

  1、整型比字符操作代价更低，因为字符集和校对规则是字符比较比整型比较更复杂，

  2、使用mysql自建类型而不是字符串来存储日期和时间

  3、用整型存储IP地址

* 案例：
  * 创建两张相同的表，改变日期的数据类型，查看SQL语句执行的速度

### 尽量避免null

* 如果查询中包含可为NULL的列，对mysql来说很难优化，因为可为null的列使得索引、索引统计和值比较都更加复杂，坦白来说，通常情况下null的列改为not null带来的性能提升比较小，所有没有必要将所有的表的schema进行修改，但是应该尽量避免设计成可为null的列

### 实际细则

* 整数类型

  * 可以使用的几种整数类型：TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT分别使用8，16，24，32，64位存储空间。
  * 尽量使用满足需求的最小数据类型

* 字符和字符串类型

  ```shell
  1. char长度固定，即每条数据占用等长字节空间；最大长度是255个字符，适合用在身份证号、手机号等定长字符串
  2. varchar可变程度，可以设置最大长度；最大空间是65535个字节，适合用在长度可变的属性
  3. text不设置长度，当不知道属性的最大长度时，适合用text
  按照查询速度：char>varchar>text
  ```

  * varchar根据实际内容长度保存数据

    1. 使用最小的符合需求的长度
    2. varchar(n) n小于等于255使用额外一个字节保存长度，n>255使用额外两个字节保存长度。
    3. varchar(5)与varchar(255)保存同样的内容，硬盘存储空间相同，但内存空间占用不同，是指定的大小 。
    4. varchar在mysql5.6之前变更长度，或者从255一下变更到255以上时时，都会导致锁表。

    * 应用场景
      1. 存储长度波动较大的数据，如：文章，有的会很短有的会很长
      2. 字符串很少更新的场景，每次更新后都会重算并使用额外存储空间保存长度
      3. 适合保存多字节字符，如：汉字，特殊字符等

  * char固定长度的字符串

    1. 最大长度：255
    2. 会自动删除末尾的空格
    3. 检索效率、写效率 会比varchar高，以空间换时间

    * 应用场景
      1. 存储长度波动不大的数据，如：md5摘要
      2. 存储短字符串、经常更新的字符串

* BLOB和TEXT类型

  * MySQL 把每个 BLOB 和 TEXT 值当作一个独立的对象处理。
  * 两者都是为了存储很大数据而设计的字符串类型，分别采用二进制和字符方式存储。

* datetime和timestamp

  ```shell
  1、不要使用字符串类型来存储日期时间数据
  2、日期时间类型通常比字符串占用的存储空间小
  3、日期时间类型在进行查找过滤时可以利用日期来进行比对
  4、日期时间类型还有着丰富的处理函数，可以方便的对时间类型进行日期计算
  5、使用int存储日期时间不如使用timestamp类型
  ```

  * datetime

    ```shell
    占用8个字节
    与时区无关，数据库底层时区配置，对datetime无效
    可保存到毫秒
    可保存时间范围大
    不要使用字符串存储日期类型，占用空间大，损失日期类型函数的便捷性
    ```

  * timestamp

    ```shell
    占用4个字节
    时间范围：1970-01-01到2038-01-19
    精确到秒
    采用整形存储
    依赖数据库设置的时区
    自动更新timestamp列的值
    ```

  * date

    ```shell
    占用的字节数比使用字符串、datetime、int存储要少，使用date类型只需要3个字节
    使用date类型还可以利用日期时间函数进行日期之间的计算
    date类型用于保存1000-01-01到9999-12-31之间的日期
    ```

* 使用枚举代替字符串类型

  * 有时可以使用枚举类代替常用的字符串类型，mysql存储枚举类型会非常紧凑，会根据列表值的数据压缩到一个或两个字节中，mysql在内部会将每个值在列表中的位置保存为整数，并且在表的.frm文件中保存“数字-字符串”映射关系的查找表

    ```sql
    create table enum_test(e enum('fish','apple','dog') not null);
    insert into enum_test(e) values('fish'),('dog'),('apple');
    select e+0 from enum_test;
    ```

* 特殊类型数据

  * 人们经常使用varchar(15)来存储ip地址，然而，它的本质是32位无符号整数不是字符串，可以使用INET_ATON()和INET_NTOA函数在这两种表示方法之间转换

    ```sql
    案例：
    select inet_aton('1.1.1.1')
    select inet_ntoa(16843009)
    ```

## 合理使用范式和反范式

### 范式

* 优点

  ```shell
  范式化的更新通常比反范式要快
  当数据较好的范式化后，很少或者没有重复的数据
  范式化的数据比较小，可以放在内存中，操作比较快
  ```

* 缺点：通常需要进行关联

### 反范式

* 优点

  ```shell
  所有的数据都在同一张表中，可以避免关联
  可以设计有效的索引；
  ```

* 缺点：表格内的冗余较多，删除数据时候会造成表有些有用的信息丢失

### 注意

* 在企业中很好能做到严格意义上的范式或者反范式，一般需要混合使用

  * 在一个网站实例中，这个网站，允许用户发送消息，并且一些用户是付费用户。现在想查看付费用户最近的10条信息。  在user表和message表中都存储用户类型(account_type)而不用完全的反范式化。这避免了完全反范式化的插入和删除问题，因为即使没有消息的时候也绝不会丢失用户的信息。这样也不会把user_message表搞得太大，有利于高效地获取数据。
  * 另一个从父表冗余一些数据到子表的理由是排序的需要
  * 缓存衍生值也是有用的。如果需要显示每个用户发了多少消息（类似论坛的），可以每次执行一个昂贵的自查询来计算并显示它；也可以在user表中建一个num_messages列，每当用户发新消息时更新这个值。

* 案例

  * 范式设计

    ![image-20200901102123823](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901102123823.png)

  * 反范式设计

    ![image-20200901102152533](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901102152533.png)

## 主键的选择

### 代理主键

* 与业务无关的，无意义的数字序列

### 自然主键

* 事物属性中的自然唯一标识

### 推荐使用代理主键

* 它们不与业务耦合，因此更容易维护
* 一个大多数表，最好是全部表，通用的键策略能够减少需要编写的源码数量，减少系统的总体拥有成本

## 字符集的选择

* 字符集直接决定了数据在MySQL中的存储编码方式，由于同样的内容使用不同字符集表示所占用的空间大小会有较大的差异，所以通过使用合适的字符集，可以帮助我们尽可能减少数据量，进而减少IO操作次数。

1. 纯拉丁字符能表示的内容，没必要选择 latin1 之外的其他字符编码，因为这会节省大量的存储空间。
2. 如果我们可以确定不需要存放多种语言，就没必要非得使用UTF8或者其他UNICODE字符类型，这回造成大量的存储空间浪费。
3. MySQL的数据类型可以精确到字段，所以当我们需要大型数据库中存放多字节数据的时候，可以通过对不同表不同字段使用不同的数据类型来较大程度减小数据存储量，进而降低 IO 操作次数并提高缓存命中率

## 存储引擎的选择

### 存储引擎的对比

![image-20200901102527469](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901102527469.png)

## 适当的数据冗余

1. 被频繁引用且只能通过 Join 2张(或者更多)大表的方式才能得到的独立小字段。
2. 这样的场景由于每次Join仅仅只是为了取得某个小字段的值，Join到的记录又大，会造成大量不必要的 IO，完全可以通过空间换取时间的方式来优化。不过，冗余的同时需要确保数据的一致性不会遭到破坏，确保更新的同时冗余字段也被更新。

## 适当拆分

* 当我们的表中存在类似于 TEXT 或者是很大的 VARCHAR类型的大字段的时候，如果我们大部分访问这张表的时候都不需要这个字段，我们就该义无反顾的将其拆分到另外的独立表中，以减少常用数据所占用的存储空间。这样做的一个明显好处就是每个数据块中可以存储的数据条数可以大大增加，既减少物理 IO 次数，也能大大提高内存中的缓存命中率。

# 执行计划

*  在企业的应用场景中，为了知道优化SQL语句的执行，需要查看SQL语句的具体执行过程，以加快SQL语句的执行效率
* 可以使用explain+SQL语句来模拟优化器执行SQL查询语句，从而知道mysql是如何处理sql语句的
* 官网地址： https://dev.mysql.com/doc/refman/5.5/en/explain-output.html 

## 包含的信息

### id

* select查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序

* id号分为三种情况

  1. 如果id相同，那么执行顺序从上到下

     ```sql
     explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal;
     ```

  2. 如果id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

     ```sql
     explain select * from emp e where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
     ```

  3. id相同和不同的，同时存在：相同的可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

     ```sql
     explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
     ```

### select_type

* 主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

  ```sql
  --sample:简单的查询，不包含子查询和union
  explain select * from emp;
  
  --primary:查询中若包含任何复杂的子查询，最外层查询则被标记为Primary
  explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;
  
  --union:若第二个select出现在union之后，则被标记为union
  explain select * from emp where deptno = 10 union select * from emp where sal >2000;
  
  --dependent union:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
  explain select * from emp e where e.empno  in ( select empno from emp where deptno = 10 union select empno from emp where sal >2000)
  
  --union result:从union表获取结果的select
  explain select * from emp where deptno = 10 union select * from emp where sal >2000;
  
  --subquery:在select或者where列表中包含子查询
  explain select * from emp where sal > (select avg(sal) from emp) ;
  
  --dependent subquery:subquery的子查询要受到外部表查询的影响
  explain select * from emp e where e.deptno in (select distinct deptno from dept);
  
  --DERIVED: from子句中出现的子查询，也叫做派生类，
  explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;
  
  --UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存
   explain select * from emp where empno = (select empno from emp where deptno=@@sort_buffer_size);
   
  --uncacheable union:表示union的查询结果不能被缓存：sql语句未验证
  ```

### table

* 对应行正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集

  1. 如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

  2. 表名是derivedN的形式，表示使用了id为N的查询产生的衍生表
  3. 当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

### type

* type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下，访问的类型有很多，效率从最好到最坏依次是：

  * `system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`
  * 一般情况下，得保证查询至少达到range级别，最好能达到ref

  ```sql
  --all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
  explain select * from emp;
  
  --index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
  explain  select empno from emp;
  
  --range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 
  explain select * from emp where empno between 7000 and 7500;
  
  --index_subquery：利用索引来关联子查询，不再扫描全表
  explain select * from emp where emp.job in (select job from t_job);
  
  --unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
   explain select * from emp e where e.deptno in (select distinct deptno from dept);
   
  --index_merge：在查询过程中需要多个索引组合使用，没有模拟出来
  
  --ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
  explain select * from emp e where  e.mgr is null or e.mgr=7369;
  
  --ref：使用了非唯一性索引进行数据的查找
   create index idx_3 on emp(deptno);
   explain select * from emp e,dept d where e.deptno =d.deptno;
  
  --eq_ref ：使用唯一性索引进行数据查找
  explain select * from emp,emp2 where emp.empno = emp2.empno;
  
  --const：这个表至多有一个匹配行，
  explain select * from emp where empno = 7369;
   
  --system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现
  ```

### **possible_keys** 

* 显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

  ```sql
  explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
  ```

### key

* 实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠

  ```sql
  explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
  ```

### key_len

* 表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好

  ```sql
  explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
  ```

### ref

* 显示索引的哪一列被使用了，如果可能的话，是一个常数

  ```sql
  explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
  ```

### rows

* 根据表的统计信息及索引使用情况，大致估算出找出所需记录需要读取的行数，此参数很重要，直接反应的sql找了多少数据，在完成目的的情况下越少越好

  ```sql
  explain select * from emp;
  ```

### extra

* 包含额外的信息

  ```sql
  --using filesort:说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
  explain select * from emp order by sal;
  
  --using temporary:建立临时表来保存中间结果，查询完成之后把临时表删除
  explain select ename,count(*) from emp where deptno = 10 group by ename;
  
  --using index:这个表示当前的查询时覆盖索引的，直接从索引中读取数据，而不用访问数据表。如果同时出现using where 表名索引被用来执行索引键值的查找，如果没有，表面索引被用来读取数据，而不是真的查找
  explain select deptno,count(*) from emp group by deptno limit 10;
  
  --using where:使用where进行条件过滤
  explain select * from t_user where id = 1;
  
  --using join buffer:使用连接缓存，情况没有模拟出来
  
  --impossible where：where语句的结果总是false
  explain select * from emp where empno = 7469;
  ```





