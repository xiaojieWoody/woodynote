# 数据库设计三范式

* 第一范式( 1NF)：每一列只有一个单一的值，不可再拆分
* 第二范式( 2NF)：每一行都有主键能进行区分
* 第三范式( 3NF)：必须先满足第二范式，每一个表都不包含其他表已经包含的非主键信息

# MySQL基本架构图

![image-20200901170900062](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901170900062.png)

## 连接器

* 连接器负责跟客户端建立连接，获取权限、维持和管理连接
  * 用户名密码验证
  * 查询权限信息，分配对应的权限
  * 可以使用show processlist查看现在的连接
  * 如果太长时间没有动静，就会自动断开，通过wait_timeout控制，默认8小时
* 连接可以分为两类:
  * 长连接：推荐使用，但是要周期性的断开长连接
  * 短链接:

## 查询缓存

* 当执行查询语句的时候，会先去查询缓存中查看结果，之前执行过的sql语句及其结果可能以key-value的形式存储在缓存中，如果能找到则直接返回，如果找不到，就继续执行后续的阶段
* 但是，不推荐使用查询缓存:
  * 查询缓存的失效比较频繁，只要表更新，缓存就会清空
  * 缓存对应新更新的数据命中率比较低

## 分析器

* 词法分析:Mysql需要把输入的字符串进行识别每个部分代表什么意思

  * 把字符串T识别成表名T
  * 把字符串 ID 识别成 列ID

* 语法分析:

  * 根据语法规则判断这个sql语句是否满足mysql的语法，如果不符合就会报错“You have an error in your SQL synta”


## 优化器

* 在具体执行SQL语句之前，要先经过优化器的处理
  * 当表中有多个索引的时候，决定用哪个索引
  * 当sql语句需要做多表关联的时候，决定表的连接顺序
  * 等等
* 不同的执行方式对SQL语句的执行效率影响很大
  * RBO:基于规则的优化
  * CBO:基于成本的优化

## Redo日志

* innodb存储引擎的日志文件

* 当发生数据修改的时候，innodb引擎会先将记录写到redo log中， 并更新内存，此时更新就算是完成了，同时innodb引擎会在合适 的时机将记录操作到磁盘中

* Redolog是固定大小的，是循环写的过程

* 有了redolog之后，innodb就可以保证即使数据库发生异常重启，之前的记录也不会丢失，叫做crash-safe

  ![image-20200901171616761](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901171616761.png)

* 疑惑

  * 既然要避免io，为什么写redo log的时候不会造成io的问题?

    ![image-20200901171740837](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901171740837.png)

## Undo log

* Undo Log是为了实现事务的原子性，在MySQL数据库InnoDB存储引擎中， 还用Undo Log来实现多版本并发控制(简称:MVCC)
* 在操作任何数据之前，首先将数据备份到一个地方(这个存储数据备份的地方 称为Undo Log)。然后进行数据的修改。如果出现了错误或者用户执行了 ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态
* 注意:undo log是逻辑日志，可以理解为：
  * 当delete一条记录时，undo log中会记录一条对应的insert记录
  *  当insert一条记录时，undo log中会记录一条对应的delete记录
  * 当update一条记录时，它记录一条对应相反的update记录

## binlog

* 服务端的日志文件
* Binlog是server层的日志，主要做mysql功能层面的事情
* Binlog中会记录所有的逻辑，并且采用追加写的方式
* 一般在企业中数据库会有备份系统，可以定期执行备份，备份的周期可以自己设置
* 恢复数据的过程：
  * 找到最近一次的全量备份数据
  * 从备份的时间点开始，将备份的binlog取出来，重放到要恢复的那个时刻
*  与redo日志的区别：
  * redo是innodb独有的，binlog是所有引擎都可以使用的
  * redo是物理日志，记录的是在某个数据页上做了什么修改，binlog是逻辑日志，记录的是这个语句的原始逻辑
  * redo是循环写的，空间会用完，binlog是可以追加写的，不会覆盖之前的日志信息

## 数据更新的流程

* 执行流程

  ![image-20200901172717158](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200901172717158.png)

  1. 执行器先从引擎中找到数据，如果在内存中直接返回，如果不在内存中，查询后返回
  2. 执行器拿到数据之后会先修改数据， 然后调用引擎接口重新写入数据
  3. 引擎将数据更新到内存，同时写数据 到redo中，此时处于prepare阶段，并通知执行器执行完成，随时可以操作
  4. 执行器生成这个操作的binlog
  5. 执行器调用引擎的事务提交接口，引擎把刚刚写完的redo改成commit状态， 更新完成

## Redo log的两阶段提交

* 先写redo log后写binlog
  * 假设在redo log写完，binlog还没有写完的时候，MySQL进程异常重启。由于我们前面说过的，redo log写完之后，系统即使崩溃，仍然能够把数据恢复 回来，所以恢复后这一行c的值是1。但是由于binlog没写完就crash了，这时候binlog里面 就没有记录这个语句。因此，之后备份日志的时候，存起来的binlog里面就没有这条语句。 然后你会发现，如果需要用这个binlog来恢复临时库的话，由于这个语句的binlog丢失，这 个临时库就会少了这一次更新，恢复出来的这一行c的值就是0，与原库的值不同
* 先写binlog后写redo log
  * 如果在binlog写完之后crash，由于redo log还没写，崩溃恢复以后这个事务无效，所以这一行c的值是0。但是binlog里面已经记录了“把c从0改成1”这个 日志。所以，在之后用binlog来恢复的时候就多了一个事务出来，恢复出来的这一行c的值 就是1，与原库的值不同

# MySQL的锁机制

* 锁是计算机协调多个进程或线程并发访问某一资源的机制
* MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）；InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁
  * **表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低
  * **行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高
* 仅从锁的角度 来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统

## MyISAM表锁

* MySQL的表级锁有两种模式：**表共享读锁（Table Read Lock）**和**表独占写锁（Table Write Lock）**

* 对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM表的读操作与写操作之间，以及写操作之间是串行的！

* 建表语句：

  ```sql
  CREATE TABLE `mylock` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `NAME` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=MyISAM DEFAULT CHARSET=utf8;
  
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('1', 'a');
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('2', 'b');
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('3', 'c');
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('4', 'd');
  ```

### MyISAM写锁阻塞读的案例：

* 当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的读写操作都会等待，直到锁释放为止

  |                           session1                           |                         session2                          |
  | :----------------------------------------------------------: | :-------------------------------------------------------: |
  |       获取表的write锁定<br />lock table mylock write;        |                                                           |
  | 当前session对表的查询，插入，更新操作都可以执行<br />select * from mylock;<br />insert into mylock values(5,'e'); | 当前session对表的查询会被阻塞<br />select * from mylock； |
  |                释放锁：<br />unlock tables；                 |          当前session能够立刻执行，并返回对应结果          |

### MyISAM读阻塞写的案例：

* 一个session使用lock table给表加读锁，这个session可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个session可以查询表中的记录，但更新就会出现锁等待

  |                           session1                           |                           session2                           |
  | :----------------------------------------------------------: | :----------------------------------------------------------: |
  |        获得表的read锁定<br />lock table mylock read;         |                                                              |
  |   当前session可以查询该表记录：<br />select * from mylock;   |   当前session可以查询该表记录：<br />select * from mylock;   |
  | 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 当前session可以查询或者更新未锁定的表<br />select * from mylock<br />insert into person values(1,'zhangsan'); |
  | 当前session插入或者更新表会提示错误<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated | 当前session插入数据会等待获得锁<br />insert into mylock values(6,'f'); |
  |                  释放锁<br />unlock tables;                  |                       获得锁，更新成功                       |

* 注意:
  
  * MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此用户一般不需要使用命令来显式加锁，上例中的加锁时为了演示效果

### MyISAM的并发插入问题

* MyISAM表的读和写是串行的，这是就总体而言的，在一定条件下，MyISAM也支持查询和插入操作的并发执行

  |                           session1                           |                           session2                           |
  | :----------------------------------------------------------: | :----------------------------------------------------------: |
  |   获取表的read local锁定<br />lock table mylock read local   |                                                              |
  | 当前session不能对表进行更新或者插入操作<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated |    其他session可以查询该表的记录<br />select* from mylock    |
  | 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 其他session可以进行插入操作，但是更新会阻塞<br />update mylock set name = 'aa' where id = 1; |
  |          当前session不能访问其他session插入的记录；          |                                                              |
  |                  释放锁资源：unlock tables                   |               当前session获取锁，更新操作完成                |
  |           当前session可以查看其他session插入的记录           |                                                              |

* 可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺

  ```sql
  mysql> show status like 'table%';
  +-----------------------+-------+
  | Variable_name         | Value |
  +-----------------------+-------+
  | Table_locks_immediate | 352   |
  | Table_locks_waited    | 2     |
  +-----------------------+-------+
  --如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
  ```

## InnoDB锁

### 事务及其ACID属性

* 事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性
  * 原子性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
  * 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
  * 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。
  * 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

### 并发事务带来的问题

* 相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来以下问题：

* **脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读” 

* **不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。

* **幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读” 

* 上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证

* 数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

* |                  | 脏读 | 不可重复读 | 幻读 |
  | :--------------: | :--: | :--------: | :--: |
  | read uncommitted |  √   |     √      |  √   |
  |  read committed  |      |     √      |  √   |
  | repeatable read  |      |            |  √   |
  |   serializable   |      |            |      |

  可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况： 

  ```sql
  mysql> show status like 'innodb_row_lock%';
  +-------------------------------+-------+
  | Variable_name                 | Value |
  +-------------------------------+-------+
  | Innodb_row_lock_current_waits | 0     |
  | Innodb_row_lock_time          | 18702 |
  | Innodb_row_lock_time_avg      | 18702 |
  | Innodb_row_lock_time_max      | 18702 |
  | Innodb_row_lock_waits         | 1     |
  +-------------------------------+-------+
  --如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高
  ```

### InnoDB的行锁模式及加锁方法

* **共享锁（s）**：又称读锁。允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
* **排他锁（x）**：又称写锁。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁
* mysql InnoDB引擎默认的修改数据语句：**update,delete,insert都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型**，如果加排他锁可以使用select …for update语句，加共享锁可以使用select … lock in share mode语句。**所以加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过select …from…查询数据，因为普通查询没有任何锁机制**

### InnoDB行锁实现方式

* InnoDB行锁是通过给**索引**上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！**

1. 在不通过索引条件查询的时候，innodb使用的是表锁而不是行锁

   ```sql
   create table tab_no_index(id int,name varchar(10)) engine=innodb;
   insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
   ```

   |                           session1                           |                           session2                           |
   | :----------------------------------------------------------: | :----------------------------------------------------------: |
   | set autocommit=0<br />select * from tab_no_index where id = 1; | set autocommit=0<br />select * from tab_no_index where id =2 |
   |      select * from tab_no_index where id = 1 for update      |                                                              |
   |                                                              |     select * from tab_no_index where id = 2 for update;      |

   * session1只给一行加了排他锁，但是session2在请求其他行的排他锁的时候，会出现锁等待。原因是在没有索引的情况下，innodb只能使用表锁

2. 创建带索引的表进行条件查询，innodb使用的是行锁

   ```sql
   create table tab_with_index(id int,name varchar(10)) engine=innodb;
   alter table tab_with_index add index id(id);
   insert into tab_with_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
   ```

   |                           session1                           |                           session2                           |
   | :----------------------------------------------------------: | :----------------------------------------------------------: |
   | set autocommit=0<br />select * from tab_with_indexwhere id = 1; | set autocommit=0<br />select * from tab_with_indexwhere id =2 |
   |     select * from tab_with_indexwhere id = 1 for update      |                                                              |
   |                                                              |     select * from tab_with_indexwhere id = 2 for update;     |

3. 由于mysql的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现冲突的

   ```sql
   alter table tab_with_index drop index id;
   insert into tab_with_index  values(1,'4');
   ```

   |                           session1                           |                           session2                           |
   | :----------------------------------------------------------: | :----------------------------------------------------------: |
   |                       set autocommit=0                       |                       set autocommit=0                       |
   | select * from tab_with_index where id = 1 and name='1' for update |                                                              |
   |                                                              | select * from tab_with_index where id = 1 and name='4' for update<br />虽然session2访问的是和session1不同的记录，但是因为使用了相同的索引，所以需要等待锁 |

## 总结

### 对于MyISAM的表锁

1. 共享读锁（S）之间是兼容的，但共享读锁（S）与排他写锁（X）之间，以及排他写锁（X）之间是互斥的，也就是说读和写是串行的  
2. 在一定条件下，MyISAM允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表查询和插入的锁争用问题。 
3. MyISAM默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIORITY_UPDATES参数，或在INSERT、UPDATE、DELETE语句中指定LOW_PRIORITY选项来调节读写锁的争用。 
4. 由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，MyISAM表可能会出现严重的锁等待，可以考虑采用InnoDB表来减少锁冲突

### 对于InnoDB表

1. InnoDB的行锁是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。 
2. 在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同

* 在了解InnoDB锁特性后，用户可以通过设计和SQL调整等措施减少锁冲突和死锁，包括：
  * 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
  * 选择合理的事务大小，小事务发生锁冲突的几率也更小；
  * 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
  * 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
  * 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
  * 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能

# 索引

## 数据结构

![mysql数据结构选择](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/mysql数据结构选择.png)

## 红黑树

![红黑树](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/红黑树.png)

## B+Tree

![mysql索引系统 ](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/mysql索引系统 .png)

# MySQL主从复制

## 为什么需要主从复制？

* 在业务复杂的系统中，有这么一个情景，有一句sql语句需要锁表，导致暂时不能使用读的服务，那么就很影响运行中的业务，使用主从复制，让主库负责写，从库负责读，这样，即使主库出现了锁表的情景，通过读从库也可以保证业务的正常运作
* 做数据的热备
* 架构的扩展。业务量越来越大，I/O访问频率过高，单机无法满足，此时做多库的存储，降低磁盘I/O访问的频率，提高单个机器的I/O性能

## 什么是mysql的主从复制？

* 数据可以从一个MySQL数据库服务器主节点复制到一个或多个从节点。MySQL 默认采用异步复制方式，这样从节点不用一直访问主服务器来更新自己的数据，数据的更新可以在远程连接上进行，从节点可以复制主数据库中的所有数据库或者特定的数据库，或者特定的表

## mysql复制原理

### 原理

1. master服务器将数据的改变记录二进制binlog日志，当master上的数据发生改变时，则将其改变写入二进制日志中；
2. slave服务器会在一定时间间隔内对master二进制日志进行探测其是否发生改变，如果发生改变，则开始一个I/OThread请求master二进制事件
3. 同时主节点为每个I/O线程启动一个dump线程，用于向其发送二进制事件，并保存至从节点本地的中继日志中，从节点将启动SQL线程从中继日志中读取二进制日志，在本地重放，使得其数据和主节点的保持一致，最后I/OThread和SQLThread将进入睡眠状态，等待下一次被唤醒

### 也就是说

* 从库会生成两个线程,一个I/O线程,一个SQL线程;
* I/O线程会去请求主库的binlog,并将得到的binlog写到本地的relay-log(中继日志)文件中;
* 主库会生成一个log dump线程,用来给从库I/O线程传binlog;
* SQL线程,会读取relay log文件中的日志,并解析成sql语句逐一执行;

### 注意

1. master将操作语句记录到binlog日志中，然后授予slave远程连接的权限（master一定要开启binlog二进制日志功能；通常为了数据安全考虑，slave也开启binlog功能）
2. slave开启两个线程：IO线程和SQL线程。其中：IO线程负责读取master的binlog内容到中继日志relay log里；SQL线程负责从relay log日志里读出binlog内容，并更新到slave的数据库里，这样就能保证slave数据和master数据保持一致了
3. Mysql复制至少需要两个Mysql的服务，当然Mysql服务可以分布在不同的服务器上，也可以在一台服务器上启动多个服务
4. Mysql复制最好确保master和slave服务器上的Mysql版本相同（如果不能满足版本一致，那么要保证master主节点的版本低于slave从节点的版本）
5. master和slave两节点间时间需同步

### 具体步骤

1. 从库通过手工执行change  master to 语句连接主库，提供了连接的用户一切条件（user 、password、port、ip），并且让从库知道，二进制日志的起点位置（file名 position 号）；    start  slave
2. 从库的IO线程和主库的dump线程建立连接
3. 从库根据change  master  to 语句提供的file名和position号，IO线程向主库发起binlog的请求
4. 主库dump线程根据从库的请求，将本地binlog以events的方式发给从库IO线程
5. 从库IO线程接收binlog  events，并存放到本地relay-log中，传送过来的信息，会记录到master.info中
6. 从库SQL线程应用relay-log，并且把应用过的记录到relay-log.info中，默认情况下，已经应用过的relay 会自动被清理purge

## mysql主从形式

### 一主一从

![1570714549624](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/1570714549624.png)

### 主主复制

![1570714565647](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/1570714565647.png)

### 一主多从

![1570714576819](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/1570714576819.png)

### 多主一从

![1570714615915](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/1570714615915.png)

### 联级复制

![1570714660961](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/1570714660961.png)

## mysql主从同步延时分析

* mysql的主从复制都是单线程的操作，主库对所有DDL和DML产生的日志写进binlog，由于binlog是顺序写，所以效率很高，slave的sql thread线程将主库的DDL和DML操作事件在slave中重放。DML和DDL的IO操作是随机的，不是顺序，所以成本要高很多，另一方面，由于sql thread也是单线程的，当主库的并发较高时，产生的DML数量超过slave的SQL thread所能处理的速度，或者当slave中有大型query语句产生了锁等待，那么延时就产生了

* 解决方案
  1. 业务的持久化层的实现采用分库架构，mysql服务可平行扩展，分散压力
  2. 单个库读写分离，一主多从，主写从读，分散压力。这样从库压力比主库高，保护主库
  3. 服务的基础架构在业务和mysql之间加入memcache或者redis的cache层。降低mysql的读压力
  4. 不同业务的mysql物理上放在不同机器，分散压力
  5. 使用比主库更好的硬件设备作为slave，mysql压力小，延迟自然会变小
  6. 使用更加强劲的硬件设备
* **mysql5.7之后使用MTS并行复制技术，永久解决复制延时问题**