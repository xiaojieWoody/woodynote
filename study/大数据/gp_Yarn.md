![image-20200101220856915](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101220856915.png)

# Yarn产生背景和基本架构

## Yarn产生背景

* 如何管理集群资源?
* 如何给任务合理分配资源?

## 在Yarn产生之前

![image-20200101221159234](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101221159234.png)

## MapReduce V1执行job流程

![image-20200101221301899](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101221301899.png)

![image-20200101221334079](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101221334079.png)

## MapReduce V1存在的问题

![image-20200101221431411](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101221431411.png)

## Yarn基本概念

* 从Hadoop 2.0开始出现了Yarn

* Yet Another Resource Negotiator

* 在Yarn上天然支持各种分布式计算框架

  ![image-20200101222053661](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222053661.png)

* ![image-20200101222125621](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222125621.png)

  * NodeManager管理内存和CPU
  * DataNode管理磁盘上的数据
  * 混合部署能够充分利用内存、CPU、磁盘资源

## Yarn基本思想

* 在MapReduce V1中
  * JobTracker = 资源管理器 + 任务调度器
* 在Yarn中做了切分
  * 资源管理
    * 让**ResourceManager**负责
  * 任务调度
    * 让**ApplicationMaster**负责
      * 每个作业启动一个
      * 根据作业切分任务tasks
      * 向Resource Manager申请资源
      * 与NodeManager协作，将分配申请到的资源给内部任务tasks
      * 监控tasks运行情况，重启失败的任务
* JobHistoryServer
  * 每个集群每种计算框架一个
  * 负责搜集归档所有的日志

## Yarn计算资源抽象

![image-20200101222452362](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222452362.png)

![image-20200101222525441](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222525441.png)

![image-20200101222550252](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222550252.png)

# Yarn资源调度过程

![image-20200101222718509](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222718509.png)

![image-20200101222741050](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222741050.png)

![image-20200101222804762](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222804762.png)

![image-20200101222833833](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222833833.png)



![image-20200101222905049](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222905049.png)

![image-20200101222936153](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222936153.png)

![image-20200101222958484](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101222958484.png)

![image-20200101223016756](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101223016756.png)

![image-20200101223037916](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101223037916.png)



45



## Yarn状态机

![image-20200101223138807](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101223138807.png)

## Yarn各个组件之间的心跳信号

* Application Master与Resource Manager心跳
  * AM->RM
    * 对Container的资源需求(CPU和Memory)和优先级
    * 已用完等待回收的Container列表
  * RM->AM
    * 新申请到的Container
    * 已完成Container的状态
* Application Master与Node Manager心跳
  * AM->NM
    * 发起启动Container请求
  * NM -> AM
    * 汇报Container状态
* Node Manager与Resource Manager心跳
  * NM->RM
    * Node Manager上所有的Container状态
  * RM->NM
    * 已删除和等待清理的Container列表

## Yarn资源隔离策略

![image-20200101223524665](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101223524665.png)

## Yarn容错处理

* 失败类型

  * 程序失败 进程奔溃 硬件问题

* 如果作业失败了

  * 作业异常均会汇报给Application Master
  * 通过心跳信号检查挂住的任务
  * 一个作业的任务失败比例超过配置，就会认为该任务失败

* 如果Application Master失败了

  * Resource Manager接收不到心跳信号时会重启Application Master

* 如果Node Manager失败了

  * Resource Manager接收不到心跳信号时会将其移出

  * Resource Manager通知Application Master，让Application Master决定任务如何处理

  * 如果某个Node Manager失败任务次数过多，Resource Manager调度任务时不再其上面

    运行任务

* 如果Resource Manager运行失败

  * 通过checkpoint机制，定时将其状态保存到磁盘，失败的时候，重新运行
  * 通过Zookeeper同步状态和实现透明的HA

## Example Yarn HelloWorld

![image-20200101223829556](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101223829556.png)

# Yarn调度器和调度算法

## Yarn资源调度算法

* 集群资源调度器需要解决
  * 多租户(Multi-tenancy)
    * 多用户同时提交多种应用程序
    * 资源调度器需要解决如何合理分配资源
  * 可扩展性(Scalability):增加集群机器的数量可以提高整体集群性能

![image-20200101224046538](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101224046538.png)

## FIFO调度器

* 所有向集群提交的作业使用一个队列
* 根据提交作业的顺序来运行(先来先服务)
* 优点:
  * 简单易懂
  * 可以按照作业优先级调度
* 缺点
  * 资源利用率不高
  * 不允许抢占

## Capacity Scheduler资源调度器

![image-20200101224421435](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101224421435.png)

## Capacity Scheduler配置

![image-20200101224521554](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101224521554.png)

## Capacity Scheduler资源分配算法

![image-20200101224615138](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101224615138.png)

## Fair Scheduler资源调度器

* 设计思想
  * 资源公平分配
* 具有与Capacity Scheduler相似的特点
  * 树状队列
  * 每个队列有独立的最小资源保证
  * 空闲时可以分配资源给其他队列使用
  * 支持内存资源调度和CPU资源调度
  * 支持抢占
* 不同点
  * 核心调度策略不同
    * Capacity Scheduler优先选择资源利用率低的队列
    * 公平调度器考虑的是公平，公平体现在作业对资源的缺额
  * 单独设置队列间资源分配方式
    * FAIR(只考虑Memory)
    * DRF(主资源公平调度，共同考虑CPU和Memory)
  * 单独设置队列内部资源分配方式
    * FAIR DRF FIFO

## Fair Scheduler - FAIR资源分配算法

![image-20200101224926053](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101224926053.png)

## Capacity Scheduler和Fair Scheduler对比

![image-20200101225019684](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101225019684.png)

# Yarn常用命令介绍

* yarn application

  * 列出所有Application
    * yarn application -list
  * 根据Application状态过滤
    * yarn application -list -appStates ACCEPTED
  * Kill掉Application
    * yarn application -kill <ApplicationId>

* yarn logs

  * 查询Application日志

    * yarn logs -applicationId <ApplicationId>

  * 查询Container日志

    * yarn logs -applicationId <ApplicationId> \

      -containerId <ContainerId> \

      -nodeAddress <NodeAddress>

    * 端口是配置文件中yarn.nodemanager.webapp.address参数指定

* yarn applicationattempt

  * 列出所有Application尝试的列表
    * yarn applicationattempt -list <ApplicationId>
  * 打印ApplicationAttemp状态
    * yarn applicationattempt -status <ApplicationAttemptId>

* yarn container

  * 列出所有Container
    * yarn container -list <ApplicationAttemptId>
  * 打印Container状态
    * yarn container -status <ContainerId>

* yarn node

  * 列出所有节点
    * yarn node -list -all

* yarn rmadmin

  * 加载队列配置
    * yarn rmadmin –refreshQueues

* yarn queue

  * 打印队列信息
    * yarn queue -status <QueueName>

* yarn classpath

  * 打印Hadoop Jar包路径

# 常见基于Yarn的计算框架

## MapReduce On Yarn

![image-20200101225535547](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101225535547.png)

## Spark

![image-20200101225610899](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101225610899.png)

![image-20200101225632874](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101225632874.png)

![image-20200101225654088](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101225654088.png)

## Spark On Yarn

![image-20200101225728416](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101225728416.png)

# 课后作业

1. 请简述一下Yarn的结构
2. 描述RM和NM是如何交互的
3. 完成在CDH集群中配置队列
4. 将WordCount作业提交到队列上运行并观察运行过程
5. 熟悉Yarn-Example代码