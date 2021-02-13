![image-20210213154525178](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213154525178.png)

jdk、tomcat官网下载tomcat10、解压、启动bin/startup.sh、检查启动jps、访问http://localhost:8080

```xml
/conf/tomcat-user.xml 添加用户
<role rolename="manager"/>
<role rolename="manager-gui"/>
<role rolename="admin"/>
<role rolename="admin-gui"/>
<user username="tomcat" password="tomcat" roles="admin-gui,admin,manager-gui,manager"/>

/webapps/manager/META-INF/context.xml
注释掉
<!--
	<Valve className="org.apache.catalina.valves.RemoteAddrvalve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1"/>"
-->

重启
http://localhost:8080/manager/status
tomcat/tomcat
```

# Tomcat配置优化

禁用AJP连接

​	什么是ajp？网站，动态/静态数据

​		方式1：直接走http协议，直接访问tomcat		![image-20210213161655149](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213161655149.png)

​	修改配置文件：conf/server.xml，禁用ajp连接器

```xml
<!-- <Connector protocol="AJP/1.3" address="::1" port="8009" secretRequired="" redirectPort="8443"/> -->
```

JvisualVM查看ajp相关线程（优化的目的是要禁用这些无用的线程）

```shell
# catalina.sh
# export JDK_JAVA_OPTIONS 下添加
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=192.168.0.108 -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.rmi.port=9999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
```

JvisualVM添加JMX连接...  192.168.0.108:9999  - 线程

执行器（线程池）

​	每个请求都是一个线程，所以可以打开线程池提高性能

​	修改配置文件：conf/server.xml中的<Executor>注释打开，注释掉默认的<Connector>，使用Executor中线程池对应的executor

```xml 
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="150" minSpareThreads="11">

<Connector executor="tomcatThreadPool" port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443"/>
```

4种运行模式

​	BIO:tomcat7之前默认的阻塞模式，性能低下，没有经过任何优化处理和支持，8.5版本之后已经抛弃该模式

​	NIO:同步非阻塞模式，tomcat内部实现了reactor线程模型，性能较高

​	NIO2:纯异步模式，tomcat内部实现了preactor线程模型

​		conf/server.xml

​		<Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol" connectionTimeout="20000" redirectPort="8443"/>

​	APR：安装起来最困难，但是从操作系统级别来解决异步的IO问题，大幅度的提高了性能



![image-20210213165752063](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213165752063.png)

Idea中导入tomcat源码并本地启动

​	调整server.xml中Connector-protocol参数来选择不同的运行模式

JMeter起10个线程进行压测

![image-20210213165634084](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213165634084.png)

# 部署测试用的Java web项目

![image-20210213175836057](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213175836057.png)

![image-20210213175909621](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213175909621.png)

![image-20210213193044300](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213193044300.png)

![image-20210213193211297](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213193211297.png)

![image-20210213193309605](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213193309605.png)

准备好test-web.war上传到linux服务器，进行部署安装

访问首页，查看是否已经启动成功：

http://192.168.0.108:8080/test-web

# 使用Apache JMeter进行测试

官网下载

TestIO - 线程组（线程数1000，循环次数10）- 添加取样器（HTTP请求：协议http、服务器名称或IP192.168.0.108、端口号8080、路径/test_web）

TestIO - 添加监听器 - 查看结果树/聚合报告/用表格查看结果

![image-20210213193959098](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213193959098.png)

# 调整tomcat参数进行优化

**禁用AJP服务**

​	具体禁用操作：在conf/server.xml中禁用以下代码

​	<!--Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>-->

​	重新进行压测，查看吞吐量

禁用AJP后

![image-20210213194312473](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213194312473.png)

没禁用AJP

![image-20210213194454849](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213194454849.png)

**设置线程池**

​	设置最大线程数为500，初始为50

​	<Executor name="tomcatThreadPool" namePrefix="cacalina-exec-" maxThreads="500" minSpareThreads="50" prestartminSpareThreads="true"/>

![image-20210213195219316](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213195219316.png)

不断调整、验证最大线程数进行压测，找到最优值

![image-20210213195913465](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213195913465.png)

综合考虑：平均值、异常（一般不超过1%）、吞吐量

还要考虑并发量，并发量过多，可以加机器

**设置nio2的运行模式**

​	设置最大线程数为500，协议设置为NIO2

​	<Executor name="tomcatThreadPool" namePrefix="cacalina-exec-" maxThreads="500" minSpareThreads="50" prestartminSpareThreads="true"/>

​	<Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol" connectionTimeout="20000" redirectPort="8443"/>

![image-20210213200949581](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213200949581.png)

# Tomcat堆栈中常见线程

![image-20210213201122968](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210213201122968.png)

==8-6==

**Main线程**

​	main线程是tomcat的主要线程，其主要作用是通过启动包来对容器进行点火

​	main的作用就是把容器组件拉起来，然后阻塞在8005端口，等待关闭

**localhost-startStop线程（tomcat8）**

​	tomcat按照层级进行异步启动，对于每一层级的组件都是采用startStop线程进行启动

​	当组件启动完成后，那么该线程就退出了，生命周期仅仅限于此

**AsyncFileHandlerWriter线程**

​	异步日志线程

**ContainerBackgroundProcessor线程（tomcat8）**

​	主要负责实时扫描tomcat容器的变化，在一些时刻触发某些事件，例如在热部署开启时reload工程等

​	扫描容器时按照层级递归扫描

**Catalina-Utility线程（tomcat9）**

​	将tomcat8中的ContainerBackgroundProcessor线程和startStop线程合二为一了

==8-10==

**Acceptor线程（重要）**

​	tomcat前端最外层的线程，负责统一接收socket请求

​	Acceptor处理完之后的交接线程，在bio和nio模式中略有差异

**ClientPoller线程（默认两个）（重要）**

​	nio模式中的特有线程，reactor模式的实现者

​	具体负责接收acceptor线程交接过来的事件，对事件轮询后交接给exec线程处理

==8-11==

**exec线程（默认10个）（重要）**

​	tomcat的主要工作线程，默认开启10个，接收poller线程丢过来的io事件

​	主要工作是http协议解析，攒出Request和Response，然后调用Tomcat后端的容器

# NIO连接器前端整体框图

# BIO连接器前端整体框图

# tomcat的BIO和NIO通道及对性能的影响

# Tomcat中的NIO2通道

# APR通道到底是个怎么回事？

# Tomcat中各通道的sendfile支持

# Tomcat中的compression压缩属性优化

# Tomcat优化之deferAccept参数

# tomcat对keep-alive的实现逻辑及优化

# 调整和tomcat相关的JVM参数进行优化



