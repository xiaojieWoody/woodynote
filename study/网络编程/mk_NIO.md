# MK_NIO

## BIO网络模型

![image-20191211095252204](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191211095252204.png)

* BIO网络模型缺点
  * 阻塞式I/O模型
  * 弹性伸缩能力差
  * 多线程耗资源：一个客户端一个线程

## NIO网络模型

* 基于非阻塞I/O，设计应对高并发场景编程模型

* NIO网络模型猜想

  ![image-20191211100106561](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191211100106561.png)

* NIO网络模型

  ![image-20191211100406948](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191211100406948.png)

  1. 注册建立连接事件，Selector循环检测注册事件就绪情况
     * Selector负责管理与客户端建立的多个连接，负责监听注册到它上面的事件，比如有新连接接入或者这个连接上有可读消息、可写消息，监听到事件后会调用对应的事件处理器来完成对事件的响应
     * Selector接收所有与客户端的Socket连接，并监听这些连接所关心的事件，当这些事件发生后，会调用相应的事件处理器来处理这些事件
  2. 客户端发送建立连接请求，Selector检测到后会启动建立连接事件处理器，调用AcceptHandler方法来创建与客户端的Socket连接，并响应客户端建立连接成功的请求
  3. AcceptHandler方法会把新建立的Socket连接注册到Selector上，并且注册这个连接的可读事件
     * AcceptHandler是个方法，不是线程，可以一次处理多个请求
  4. 客户端发送请求到Selector上时，Selector会监听到这个连接的可读事件，然后启动连接读写处理器调用Read&Write Handler方法来处理客户端发送过来的消息，进行业务逻辑处理，处理完成后，由这个方法返回响应客户端请求，这个方法还会再次将这个Socket的可读事件注册到Selector上

* NIO网络模型改进

  * 非阻塞式I/O模型
  * 弹性伸缩能力强
  * 单线程节省资源

* Selector空轮询，导致CPU100%

  * 正常情况下，`selector.select()`操作是阻塞的，只有被监听的fd有读写操作时，才被唤醒，但是，在这个bug中，没有任何fd有读写请求，但是`select()`操作依旧被唤醒，很显然，这种情况下，`selectedKeys()`返回的是个空数组，然后按照逻辑执行到`while(true)`处，循环执行，导致死循环，最终造成CPU利用率百分百现象
  * netty 会在每次进行 selector.select(timeoutMillis) 之前记录一下开始时间currentTimeNanos，在select之后记录一下结束时间，判断select操作是否至少持续了timeoutMillis秒，如果持续的时间大于等于timeoutMillis，说明就是一次有效的轮询，否则，表明该阻塞方法并没有阻塞这么长时间，可能触发了jdk的空轮询bug，当空轮询的次数超过一个阀值的时候，默认是512，就开始重建selector

## NIO核心

### Channel

* 通道

  * 双向性：可读可写
  * 非阻塞性
  * 操作唯一性：通过Buffer

* 实现

  * 文件类：FileChannel
  * UDP类：DatagramChannel
  * TCP类：ServerSocketChannel / SocketChannel

  ```java
  // socket服务器
  //1. 监听端口
  ServerSocket serverSocket = new ServerSocket(8000);
  while(true) {
    //2. 提交请求，建立连接
    Socket socket = serverSocket.accept();
    //3. 数据交换
    new Thread(new BIOServerHandler(socket)).start();
  }
  //4. 关闭资源
  serverSocket.close();
  ```

  ```java
  // socket客户端
  //1. 建立连接
  Socket socket = new Socket("127.0.0.1", 8000);
  // 获取输入输出流
  InputStream inputStream = socket.getInputStream();
  OutputStream outputStream = socket.getOutputStream();
  ```

* 使用

  ```java
  //1. 服务器端通过服务端socket创建channel
  ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
  //2. 服务器端绑定端口
  serverSocketChannel.bind(new InetSocketAddress(8000));
  //3. 服务器端监听客户端连接，建立socketChannel连接
  SocketChannel socketChannel = serverSocketChannel.accept();
  //4. 客户端连接远程主机及端口
  SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1",8000))
  ```

### Buffer

* 缓冲区

* 作用：读写Channel中数据

* 本质：一块内存区域

* 属性

  * Capacity：容量

  * Position：位置

  * Limit：上限

  * Mark：标记

    ```java
    // 初始化长度为10的byte类型buffer			position=0,limit=10,capacity=10
    ByteBuffer.allocate(10);  
    // 向byteBuffer中写入三个字节					 position=3,limit=10,capacity=10
    byteBuffer.put("abc".getBytes(Charset.forName("UTF-8")));
    // 将byteBuffer从写模式切换成读模式		 position=0,limit=3,capacity=10
    byteBuffer.flip();
    // 从byteBuffer中读取一个字节					 position=1,limit=3,capacity=10
    byteBuffer.get();
    // 调用mark方法记录下当前position的位置  position=1,limit=3,capacity=10,mark=1
    byteBuffer.mark();
    // 先调用get方法读取下一个字节，再调用reset方法将position重置到mark位置
    byteBuffer.get();                     
    byteBuffer.reset();									//position=1,limit=3,capacity=10,mark=1
    // 调用clear方法，将所有属性重置，				position=0,limit=10,capacity=10
    byteBuffer.clear();									
    ```

### Selector

* 选择器或多路复用器

* 作用：I/O就绪选择

* 地位：NIO网络编程的基础

* 使用

  ```java
  // 创建Selector
  Selector selector = Selector.open();
  // 将Channel注册到Selector上，监听读就绪事件
  SelectionKey selectionKey = channel.register(selector, SelectionKey.OP_READ;
  // 阻塞等待channel有就绪事件发生
  int selectNum = selector.select();
  // 获取发生就绪事件的channel集合
  Set<SelectionKey> selectedKeys = selector.selectedKeys();
  ```

* SelectionKey

  * 四种就绪状态常量
    * Connect：连接就绪
    * Accept：接收就绪
    * Read：读就绪
    * Write：写就绪
  * 有价值的属性
    * selector.selectionKey会返回SelectionKey集合
    * 通过SelectionKey集合获取对应的Channel、Selector对象、该Channel已就绪集合和所关心事件集合

## NIO编程实现

1. 创建Selector
2. 创建ServerSocketChannel，并绑定监听端口
3. 将Channel设置为非阻塞模式
4. 将Channel注册到Selector上，监听连接事件
5. 循环调用Selector的select方法，检测就绪情况
6. 调用selectedKeys方法获取就绪channel集合
7. 判断就绪事件种类，调用业务处理方法
8. 根据业务需要决定是否再次注册监听事件，重复执行第三步操作

###  服务端

```java
public class NioServer {

    /**
     * 主方法
     */
    public static void main(String[] args) throws IOException {
        NioServer nioServer = new NioServer();
        nioServer.start();
    }

    /**
     * 启动
     */
    public void start() throws IOException {
        //1. 创建Selector
        Selector selector = Selector.open();
        //2. 通过ServerSocketChannel创建Channel通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //3. 为Channel通道绑定监听端口
        serverSocketChannel.bind(new InetSocketAddress(8000));
        //4. 设置Channel为非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //5. 将Channel注册到selector上，监听连接事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("服务器启动成功！");
        //6. 循环等待新接入的连接
        for (; ; ) {
            // 获取可用channel数量
            int readyChannels = selector.select();

            if (readyChannels == 0) {
                continue;
            }
            // 获取可用channel的集合
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                // selectionKey实例
                SelectionKey selectionKey = iterator.next();
                // 移除Set中的当前selectionKey
                iterator.remove();
                //7. 根据就绪状态，调用对应方法处理业务逻辑
                // 如果是 接入事件
                if (selectionKey.isAcceptable()) {
                    acceptHandler(serverSocketChannel, selector);
                }
                // 如果是 可读事件
                if (selectionKey.isReadable()) {
                    readHandler(selectionKey, selector);
                }
            }
        }
    }

    /**
     * 接入事件处理器
     */
    private void acceptHandler(ServerSocketChannel serverSocketChannel, Selector selector) throws IOException {
        // 如果是接入事件，创建socketChannel
        SocketChannel socketChannel = serverSocketChannel.accept();
        // 将socketChanel 设置为非阻塞工作模式
        socketChannel.configureBlocking(false);
        // 将channel注册到selector上，监听 可读事件
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 回复客户端提示信息
        socketChannel.write(Charset.forName("UTF-8").encode("你与聊天室里其他人都不是朋友关系，请注意隐私安全"));
    }

    /**
     * 可读事件处理器
     */
    private void readHandler(SelectionKey selectionKey, Selector selector) throws IOException {
        // 要从 selectionKey 中获取到已经就绪的channel
        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
        // 创建buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 循环读取客户端请求信息
        String request = "";
        while (socketChannel.read(byteBuffer) > 0) {
            // 切换buffer为读模式
            byteBuffer.flip();
            // 读取buffer中的内容
            request += Charset.forName("UTF-8").decode(byteBuffer);

        }
        // 将channel再次注册到selector上，监听他的可读事件
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 将客户端发送的请求信息 广播给其他客户端
        if (request.length() > 0) {
            // 广播给其他客户端
            broadCast(selector, socketChannel, request);
        }
    }

    private void broadCast(Selector selector, SocketChannel sourceChannel, String request) {
        // 获取到所有已接入的客户端channel
        Set<SelectionKey> selectionKeySet = selector.keys();
        // 循环向所有channel广播消息
        selectionKeySet.forEach(selectionKey -> {
            Channel targetChannel = selectionKey.channel();
            // 剔除发消息的客户端
            if (targetChannel instanceof SocketChannel
                    && targetChannel != sourceChannel) {
                try {
                    // 将信息发送到targetChannel客户端
                    ((SocketChannel) targetChannel).write(
                            Charset.forName("UTF-8").encode(request));
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

## 客户端

```java
public class NioClient {
    /**
     * 启动
     */
    void start(String nickname) throws IOException {
        // 连接服务器端
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8000));

        // 接收服务器端响应
        // 新开线程，专门负责来接收服务器端服务器端的响应数据
        // selector, socketChannel，注册
        Selector selector = Selector.open();
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_READ);
        new Thread(new NioClientHandler(selector)).start();

        // 向服务器端发送数据
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            String request = scanner.nextLine();
            if (request != null && request.length() > 0) {
                socketChannel.write(Charset.forName("UTF-8").encode(nickname + "：" + request));
            }
        }
    }
}
```

```java
/**
 * 客户端线程类，专门接收服务器端响应信息
 */
public class NioClientHandler implements Runnable {

    private Selector selector;

    public NioClientHandler(Selector selector) {
        this.selector = selector;
    }

    @Override
    public void run() {
        try {
            for (; ; ) {
                // 获取可用channel数量
                int readyChannels = selector.select();

                if (readyChannels == 0) {
                    continue;
                }
                // 获取可用channel的集合
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()) {
                    // selectionKey实例
                    SelectionKey selectionKey = iterator.next();
                    // 移除Set中的当前selectionKey
                    iterator.remove();
                    //7. 根据就绪状态，调用对应方法处理业务逻辑
                    // 如果是 可读事件
                    if (selectionKey.isReadable()) {
                        readHandler(selectionKey, selector);
                    }
                }
            }
        } catch (Exception e) {

        }
    }

    /**
     * 可读事件处理器
     */
    private void readHandler(SelectionKey selectionKey, Selector selector) throws IOException {
        // 要从 selectionKey 中获取到已经就绪的channel
        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 循环读取服务端请求信息
        String response = "";
        while (socketChannel.read(byteBuffer) > 0) {
            byteBuffer.flip();
            response += Charset.forName("UTF-8").decode(byteBuffer);

        }
        // 将channel再次注册到selector上，监听他的可读事件
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 将服务端响应信息打印到本地
        if (response.length() > 0) {
            System.out.println(response);
        }
    }
}
```

```java
public class ClientA {
    public static void main(String[] args) throws IOException {
        new NioClient().start("woody1");
    }
}
public class ClientB {
    public static void main(String[] args) throws IOException {
        new NioClient().start("woody2");
    }
}
```

