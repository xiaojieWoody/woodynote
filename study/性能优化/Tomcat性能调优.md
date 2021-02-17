==8-12==

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

**Acceptor线程（重要，一定得掌握）**

​	tomcat前端最外层的线程，负责统一接收socket请求

​	Acceptor处理完之后的交接线程，在bio和nio模式中略有差异

**ClientPoller线程（默认两个）（重要，一定得掌握）**

​	nio模式中的特有线程，reactor模式的实现者

​	具体负责接收acceptor线程交接过来的事件，对事件轮询后交接给exec线程处理

==8-11==

**exec线程（默认10个）（重要，一定得掌握）**

​	tomcat的主要工作线程，默认开启10个，接收poller线程丢过来的io事件

​	主要工作是http协议解析，攒出Request和Response，然后调用Tomcat后端的容器

==8-12==

NioBlockingSelector.BlockPoller线程（默认2个）

​	负责Servlet的输入和输出 

AysncTimeout线程（tomcat8）

​	主要作用检测异步request请求时，触发超时，并将该请求再转发到工作线程池处理

其他线程

​	例如ajp相关线程，sendfile相关线程等等，tomcat需要众多线程相互合作才能完美的发挥功能。线程并不是越多越好，而是需要合理的控制线程，最终达到最优性能

# NIO连接器前端整体框图

==9-1==

线程的分类

​	Main - startstop - AsyncFileHandlerWriter - ContainerBackgroundProcessor

​	Acceptor - ClientPoller - exec - NioBlockingSelector.BlockPoller

图解tomcat总体流程

​	socket请求 -> Tomcat前端 -> 内部Request对象/CoyoteAdaptor -> Tomcat后端容器 Engine Host Context... -> HttpRequest -> 业务实现的Servlet

![image-20210215084030233](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215084030233.png)

图解tomcat前端详细流程	![image-20210215084255212](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215084255212.png)

源码解读tomcat前端关键组件初始化和启动过程

![image-20210215084542512](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215084542512.png)

Http11NioProtocol

​	http1.1的协议类，实际上这个类的初始化是由对应的Connector类进行初始化，可以看看server.xml中关于连接器的配置

NioEndPoint

​	NioEndPoint类持有三大线程池：

​		Acceptor（9独立出来了）

​		PollerEvent

​		Poller

​		Worker（exec即SocketProcessor）

Http11ConnectionHandler

​	Http11ConnectionHandler持有Http11NioProcessor类，Http11NioProcessor负责解析http协议

​	Http11NioProcessor解析完http协议，攒出request和response传递给CoyoteAdaptor，经过容器层层转发后抵达业务Servlet

总结

​	NIO的前端框架主要是由三个不同的线程依次分工协作：

1. Acceptor线程将socketchannel取出，传递给Poller线程（会产生线程阻塞，因此包装成PollEvent加入缓存队列）
2. Poller线程执行的就是NIO的selectkey，拿到通道中感兴趣的事件，轮询获取，然后将感兴趣的selectkey和keyattachment传递给工作线程池进行处理
3. 工作线程池调用http11ConnectionHandler进行http协议的解析，然后将解析出来的内容包装成Request，Response对象，传递给分界点CoyoteAdapter，最终执行到业务中

# BIO连接器前端整体框图

BIO框图源码解读（tomcat8.5后抛弃）

![image-20210215104021633](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215104021633.png)

Http11Protocol

​	与NIO一样，Http11Protocol是默认的BIO的http1.1协议的处理类

JIoEndPoint（tomcat7.x版本）

​	JIoEndpoint是BIO的端点类，它和NIO一样（NIO里面是NioEndpoint），也是维护着线程池，只不过因为没有Selector.select，所以只有2个线程池

​	acceptor

​	worker（exec）

Acceptor线程

​	Acceptor线程的主要作用和NIO一样，如果没有网络IO数据，该线程会一直被serversocket.accept阻塞住。当有数据的时候，首先将socket数据设置Connector配置的一些属性，然后将该接力棒传递给工作线程池	

SocketProcessor线程

​	SocketProcessor工作任务就是将Acceptor传过来的socketWapper包装传入到工作线程池中

总结

​	BIO的基本流程上和NIO通道一样，BIO的结构因为缺少了selector和轮询，相比NIO少了一部分的内容，整体上就是使用的ServerSocket来进行通信的，一线程一请求的模式，代码看起来清晰易懂。但是，由于BIO的模型比较落后，在大多数的场景下，不如NIO，而现在Tomcat新版本也是NIO是默认的配置。8.5版本之后完全抛弃了BIO通道

# tomcat的BIO和NIO通道及对性能的影响

BIO的缺点

​	如果IO处理时间长，那么bio大多数时间耗在线程切换中

​	IO通道得不到复用

​	Acceptor线程工作效率较低

NIO的解决之道

​	增加了poller线程池做轮询

​	提高了acceptor执行效率

NIO vs BIO

​	单线程bio，同一时刻只能处理一个请求，一个请求没处理完，下一个请求处理不了

![image-20210215184415427](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215184415427.png)

​	多线程bio，同一时刻可以处理多个请求

![image-20210215184447184](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215184447184.png)

​	模拟nio，单一线程同一时刻可以处理多个请求

![image-20210215171954441](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215171954441.png)

​	真正的nio模式，工作线程一分为二，分出一个线程出来专门处理io	![image-20210215184315639](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215184315639.png)

总结

​	BIO比NIO少了poller线程池的轮询机制，请求模式为一线程一请求的模式，这就导致了BIO中存在大量的线程上下文切换

# Tomcat中的NIO2通道

NIO2的框图源码解读

![image-20210215184700441](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215184700441.png)

<Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol" connectionTimeout="20000" redirectPort="8443" />

异步IO的运用

![image-20210215200153921](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215200153921.png)

总结

![image-20210215200331319](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210215200331319.png)

# APR通道到底是个怎么回事？

Tomcat APR通道架构图

![image-20210216171545284](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216171545284.png)

APR通道的Socket全部来自c语言实现的socket，非jdk的socket，直接在tomcat层级调用antive方法

APR通道的SSL信道上下文直接来自于native底层

总结

![image-20210216172919396](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216172919396.png)



# Tomcat中各通道的sendfile支持

传统的网络传输机制

​	内核和用户之间切换

![image-20210216173338776](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216173338776.png)

Linux的sendfile机制（零拷贝）

​	操作系统内核中操作

![image-20210216173538105](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216173538105.png)

<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" compression="off" useSendfile="true" redirectPort="8443" />

![image-20210216200722684](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216200722684.png)

![image-20210216201035235](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216201035235.png)

# Tomcat中的compression压缩属性优化

http响应头中压缩相关属性

​	**传输内容编码：Content-Encoding**

​	传输数据编码：Transfer-Encoding

​	传输内容格式：Content-Type

<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" compression="off" compressableMimeType="text/html,text/xml,image/jpeg,text/css" noCompressionUserAgents="gozilla,traviata" useSendfile="false" redirectPort="8443" />

![image-20210216203616307](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216203616307.png)

![image-20210216203635084](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216203635084.png)

tomcat源码中的压缩实现

​	Http11Processor.prepareResponse() // 检查是否配置了压缩属性

​	如果配置了压缩属性，则GzipOutputFilter生效

​	GzipOutputFilter中会加入GzipOutputStream，压缩就是它来完成

# Tomcat优化之deferAccept参数

TCP中的TCP_DEFER_ACCEPT优化参数

​	TCP三次握手

​	tomcat中的deferAccept属性配置与实现，查看参数http://127.0.0.1:8080/docs/config/http.html

![image-20210216205236800](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216205236800.png)

​	Tomcat中的deferAccept属性实际上是操作系统级别的TCP_DEFER_ACCEPT参数的优化，只在APR通道中有实现

# tomcat对keep-alive的实现逻辑及优化

Headers - Connection: keep-alive

减少TIME_WAIT

TCP是全双工，4次挥手

keepalive的配置实现（两个参数）

​	keepAliveTimeout：此时间过后连接就close了，单位是milliseconds

​	maxKeepAliveRequests：最大长连接个数（1表示禁用，-1表示不限制个数，默认100个。一般设置在100-200之间）

Tomcat中keepalive的实现原理

​	步骤1:准备阶段 

# 调整和tomcat相关的JVM参数进行优化

配置：catalina.sh

分析gc日志：gceasy.io

设置串行垃圾回收器（nio模式，最大线程1000）

​	设年轻代、老年代均使用串行收集器，初始堆内存64M，最大堆内存512M

​	JAVA_OPTS="-XX:+UseSerialGC -Xms64m -Xmx512m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:../logs/gc.log"

​	设年轻代、老年代均使用并行收集器，初始堆内存64M，最大堆内存512M

​	JAVA_OPTS="-XX:+UseParallelGC -XX:+UseParallelOldGC -Xms64m -Xmx512m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:../logs/gc.log"

​	设置G1收集器

​	JAVA_OPTS="-XX:+UseG1GC -Xms128m -Xmx1024m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:../logs/gc.log"	

![image-20210216213511457](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216213511457.png)

总结
	对tomcat性能优化需要不断的进行参数调整，然后测试结果，可能每次调优结果都有差异，这就需要借助于gc的可视化工具来看gc的情况，再帮助做出决策应该调整哪些参数