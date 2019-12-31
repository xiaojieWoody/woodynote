* ==Netty如何解决空轮询bug问题？==
  * Netty通过计数的方式去判断， 如果当前阻塞的是一个select操作并没有花费很长时间，则有可能触发空轮训bug，默认情况达到512次，然后重建一个select，将原select上的key重新移交到新的select上，通过这种方式巧妙地避免了jdk空轮训Bug 
* ==Netty 服务端启动的流程：创建一个引导类，然后指定线程模型，IO模型，连接读写处理逻辑，绑定端口之后，服务端就启动起来了，bind 方法是异步的，可通过这个异步机制来实现端口递增绑定==
* ==Netty 客户端启动的流程：创建一个引导类，然后指定线程模型，IO 模型，连接读写处理逻辑，连接上特定主机和端口，客户端就启动起来了， `connect` 是异步的，可通过异步回调机制来实现指数退避重连逻辑==
* 异步事件驱动的网络应用程序框架，用于快速开发高性能服务端和客户端
  * 精心设计的reactor线程模型支持高并发海量连接
  * 自带各种协议栈可处理任何一种通用协议
* 高性能
  * 异步非阻塞通信
    * Netty 的 IO 线程 NioEventLoop 由于聚合了多路复用器 Selector，可以同时并发处理成百上千个客户端 Channel，由于读写操作都是非阻塞的，可以充分提升 IO 线程的运行效率，避免由于频繁 IO 阻塞导致的线程挂起
  * 零拷贝
    * 可快速高效地将数据从文件系统移动到网络接口，而不需要将其从内核空间复制到用户空间
  * 高效的Reactor线程模型
    * 主从 Reactor 线程模型
      * Acceptor 线程池仅仅只用于客户端的登陆、握手和安全认证，一旦链路建立成功，就将链路注册到后端 subReactor线程池的 IO 线程上，由 IO 线程负责后续的 IO 操作（读写和编解码）
  * 无锁化的串行设计理念
    * 通过串行无锁化设计，消息的处理尽可能在同一个线程内完成，期间不进行线程切换，这样就避免了多线程竞争和同步锁
  * 灵活的TCP参数配置能力
    * `SO_RCVBUF` 和 `SO_SNDBUF`
    * `SO_TCPNODELAY`
    * `SO_BACKLOG`
    * `SO_KEEPALIVE`
* 与NIO的对比，做了很多改进
  * 更加优雅的Reactor模式实现、灵活的线程模型、利用EventLoop等创新性的机制，可以非常高效地管理成百上千的Channe
  * 充分利用了Java的零拷贝机制，降低内存分配和回收的开销
  * 在通信协议、序列化等其他角度的优化
    * 自带协议栈，Netty 对于一些通⽤协议的编解码实现。例如：HTTP协议、WebSocket协议
    * 自带编码解码器解决拆包粘包问题，用户只用关心业务逻辑

![image-20190420115756469](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20190420115756469.png)

* 组件
  * 可自定义通信协议格式
    * 魔数(4)，版本号(1)，序列化算法(1)，指令(1)，数据长度(4)，数据(n)
    * 拆包器
      * 拆包器的作用就是根据自定义协议，把数据拼装成一个个符合自定义数据包大小的 ByteBuf，然后送到自定义协议解码器去解码
      * 基于长度域拆包器-LengthFieldBasedFrameDecoder，最通用的一种拆包器，只要自定义协议中包含长度域字段，均可以使用这个拆包器来实现应用层拆包
      * 构造拆包器：`new LengthFieldBasedFrameDecoder(Integer.MAX_VALUE, 7, 4);`
        * 第一个参数指的是数据包的最大长度，第二个参数指的是长度域的偏移量，第三个参数指的是长度域的长度，只需在 pipeline 的最前面加上这个拆包器
        * 在后续进行 decode 操作的时候，ByteBuf 就是一个完整的自定义协议数据包
  * EventLoop
    * 定义了 Netty 的核心抽象，用于处理连接的生命周期中所发生的事件
    * 在 Netty 中每个EventLoopGroup 本身是一个线程池，其中包含了自定义个数的 NioEventLoop，每个NioEventLoop 中会管理自己的一个 selector 选择器和监控选择器就绪事件的线程
      * 服务端
        * 有两个 EventLoopGroup，其中 boss组是专门用来接收客户端发来的 TCP 链接请求的，worker组是专门用来具体处理完成三次握手的链接套接字的网络IO 请求
        * 当Channel是服务端通道NioServerSocketChannel 时候，NioServerSocketChannel本身会被注册到 boss EventLoopGroup 里面的某一个NioEventLoop 管理的 selector 选择器上，而完成三次握手的链接套接字是被注册到了 worker EventLoopGroup 里面的某一个 NioEventLoop 管理的 selector 选择器上
      * 客户端
        * 持有一个 EventLoopGroup 用来处理网络 IO操作
        * 当 Channel 是客户端通道 NioSocketChannel 时候，会注册到自己关联的 NioEventLoop 的 selector 选择器上，然后NioEventLoop 对应的线程会通过 select 命令监控感兴趣的网络读写事件
    * 当⼀个连接到达时，Netty 就会创建⼀个 Channel，然后从 EventLoopGroup 中分配一个 EventLoop来与该Channel绑定用以处理所有事件：注册感兴趣的事件、将事件派发给 ChannelHandler、安排进一步的动作，在该 Channel 的整个⽣命周期中都是由这个绑定的 EventLoop 来服务的（避免线程安全和同步问题）
    * 一个 EventLoopGroup 包含一个或多个 EventLoop ；
    * EventLoop只能与⼀个 Thread 绑定并且由其处理的 I/O操作和事件都将在它专有的 Thread 上被处理，从而保证线程安全
    * ⼀个 EventLoop 可被分配至⼀个或多个 Channel
    * 一个 Channel 在它的生命周期内只能注册到一个 EventLoop 上（实际上消除了对于同步的需要）
  * Channel
    * channel的生命周期
      * `ChannelRegistered` -> `ChannelActive` -> `ChannelInactive` -> `ChannelUnregistered`
      * 当这些状态发生变化时，将会生成对应的事件，这些事件将会被转发给`ChannelPipeline`中的`ChannelHandler`，其可以随后对它们作出响应
    * 客户端：NioSocketChannel
    * 服务端：NioServerSocketChannel
    * 每个` Channel` 都将会被分配一个 `ChannelPipeline` 和 `ChannelConfig`
  * ChannelPipeline
    * 每个 Channel 都拥有一个与之相关联的 ChannelPipeline，ChannelPipeline持有一个 ChannelHandler 的实例链并且会维护一个ChannelHandlerContext的双向链表
    * 由 I/O 操作触发的事件将流经安装了一个或多个ChannelHandler 的 ChannelPipeline，传播这些事件的方法调用可以随后被 ChannelHandler 所拦截，并且可以按需地处理事件
    * 根据事件的起源，事件将会被 ChannelInboundHandler 或者 ChannelOutboundHandler处理。随后，通过调用 ChannelHandlerContext 实现，它将被转发给同一超类型的下一个 ChannelHandler（Netty 能区分且确保数据只会在具有相同定向类型的两个 ChannelHandler 之间传递）
    * 添加的自定义ChannelHandler会插入到head和tail之间，如果是ChannelInboundHandler的回调，根据插入的顺序从左向右进行链式调用，ChannelOutboundHandler则相反
  * ChannelHandlerContext
    * 每当有ChannelHandler添加到ChannelPipeline中时，都会创建对应的ChannelHandlerContext，其代表了 ChannelHandler 和 ChannelPipeline 之间的绑定。虽然这个对象可以被用于获取底层的Channel，但是它主要还是被用于写出站数据
    * ChannelPipeline实际维护的是ChannelHandlerContext 的关系，主要功能是管理它所关联的 ChannelHandler 和在同一个 ChannelPipeline 中的其他 ChannelHandler 之间的交互
    * 每个ChannelHandlerContext之间形成双向链表
  * ChannelHandler
    * 将数据从一种格式转换为另一种格式（通过将 ChannelHandler 添加到 ChannelPipeline 中来实现动态的协议切换）、提供异常的通知、提供 Channel 变为活动的或者非活动的通知、提供当 Channel 注册到 EventLoop 或者从 EventLoop 注销时的通知、提供有关用户自定义事件的通知
    * 在架构上，`ChannelHandler` 有助于保持业务逻辑与网络处理代码的分离，处理往来 ChannelPipeline 事件(包括数据)的任何代码的通用容器，ChannelHandler链路会根据Handler的类型，分为InBound和OutBound两条链路 
    * 在`ChannelHandler`被添加到`ChannelPipeline` 中或者被从`ChannelPipeline`中移除时会调用这些操作。这些生命周期方法中的每一个都接受一个`ChannelHandlerContext` 参数
    * `ChannelInboundHandler`——处理入站数据以及各种状态变化
    * `ChannelOutboundHandler`——处理出站数据并且允许拦截所有的操作
    * `ChannelHandler` 适配器
      * 有一些适配器类可以将编写自定义的 ChannelHandler 所需要的努力降到最低限度，因为它们提供了定义在对应接口中的所有方法的默认实现
      * 因为有时会忽略那些不感兴趣的事件，所以 Netty提供了抽象基类 ChannelInboundHandlerAdapter 和 ChannelOutboundHandlerAdapter。通过调用 ChannelHandlerContext 上的对应方法，每个都提供了简单地将事件传递给下一个 ChannelHandler的方法的实现。随后，可以通过重写所感兴趣的那些方法来扩展这些类
    * HandlerChannel的生命周期
      * handlerAdded() -> channelRegistered() -> channelActive() -> channelRead() -> channelReadComplete()
  * ByteBuf
    * 容量可以按需增长，读和写使用了不同的索引，读写两种模式之间切换不需调用flip()
    * 通过内置的复合缓冲区类型实现了透明的零拷贝
    * 支持方法的链式调用及引用计数（能自动释放资源）
    * 支持池化
    * 可以被用户自定义的缓冲区类型扩展
    * 维护读取（readIndex）和写入（writeIndex）两个索引
      * read或者write开头的ByteBuf 方法，将会推进其对应的索引，而以set或者get开头的操作则不会
    * 指定 ByteBuf 的最大容量，默认的限制是 Integer.MAX_VALUE
    * 只有读写索引处于同一位置。然后ByteBuf就不可读了，如果继续读的话就会抛出IndexOutOfBoundsException异常，类似读数组越界
  * 引导类
    * 对应用程序进行配置并使其运行起来的过程
    * Netty 的引导类为应用程序的网络层配置提供了容器，这涉及将一个进程绑定到某个指定的端口，或者将一个进程连接到另一个运行在某个指定主机的指定端口上的进程
    * Netty处理引导的方式：使应用程序的逻辑或实现和网络层相隔离，无论它是客户端还是服务器

      * 将`EventLoop`、`ChannelPipeline`、`ChannelHandler `和 这些部分组织起来，成为一个可实际运行的应用程序
    * 服务器端
      * ServerBootStrap
        * 服务器需要两组不同的`Channel`
        * 第一组只包含一个`ServerChannel`，代表服务器自身的已绑定到某个本地端口的正在监听的套接字
        * 第二组包含所有已创建的用来处理传入客户端连接的 Channel
      * 使用一个父 Channel 接受来自客户端的连接，并创建子 Channel 以用于它们之间的通信
    * 客户端
      * BootStrap负责为客户端和使用无连接协议的应用程序创建 Channel
      * 使用一个单独的、没有父 Channel 的 Channel 来用于所有的网络交互
  * 编解码器
    * 编码器操作出站数据
      * 编码器将消息转换为字节，并将它们转发给下一个`ChannelOutboundHandler`
      * MessageToByteEncoder
    * 解码器处理入站数据
      * 每当需要为 ChannelPipeline 中的下一个 ChannelInboundHandler 转换入站数据时会用到
      * ByteToMessageDecoder 