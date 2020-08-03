# Web 容器是什么？

* 早期的 Web 应用主要用于浏览新闻等静态页面，HTTP 服务器（比如 Apache、Nginx）向浏览器返回静态 HTML，浏览器负责解析 HTML，将结果呈现给用户
* 希望通过一些交互操作，来获取动态结果，需要一些扩展机制能够让 HTTP 服务器调用服务端程序
  * 于是 Sun 公司推出了 Servlet 技术。可以把 Servlet 简单理解为运行在服务端的 Java 小程序，但是 Servlet 没有 main 方法，不能独立运行，因此必须把它部署到 Servlet 容器中，由容器来实例化并调用 Servlet
  * 而 Tomcat 和 Jetty 就是一个 Servlet 容器。为了方便使用，它们也具有 HTTP 服务器的功能，因此**Tomcat 或者 Jetty 就是一个“HTTP 服务器 + Servlet 容器”，我们也叫它们 Web 容器**

## 操作系统基础

* 对于 Web 容器来说，操作系统方面应该掌握它的工作原理，比如什么是进程、什么是内核、什么是内核空间和用户空间、进程间通信的方式、进程和线程的区别、线程同步的方式、什么是虚拟内存、内存分配的过程、什么是 I/O、什么是 I/O 模型、阻塞与非阻塞的区别、同步与异步的区别、网络通信的原理、OSI 七层网络模型以及 TCP/IP、UDP 和 HTTP 协议《UNIX 环境高级编程》

## Java基础

* Java 的基础知识包括 Java 基本语法、面向对象设计的概念（封装、继承、多态、接口、抽象类等）、Java 集合的使用、Java I/O 体系、异常处理、基本的多线程并发编程（包括线程同步、原子类、线程池、并发容器的使用和原理）、Java 网络编程（I/O 模型 BIO、NIO、AIO 的原理和相应的 Java API）、Java 注解以及 Java 反射的原理等
* 了解一些 JVM 的基本知识，比如 JVM 的类加载机制、JVM 内存模型、JVM 内存空间分布、JVM 内存和本地内存的区别以及 JVM GC 的原理等

## **Java Web 开发基础**

* 一些通用的设计原则和设计模式
* 从学习 Servlet 和 Servlet 容器开始
* Web 框架的本质是，开发者在使用某种语言编写 Web 应用时，总结出的一些经验和设计思路。很多 Web 框架都是从实际的 Web 项目抽取出来的，其目的是用于简化 Web 应用程序开发
* 以 Spring 框架为例， Web 框架是怎么产生的。Web 应用程序的开发主要是完成两方面的工作：
  * 设计并实现类，包括定义类与类之间的关系，以及实现类的方法，方法对数据的操作就是具体的业务逻辑
  * 类设计好之后，需要创建这些类的实例并根据类与类的关系把它们组装在一起，这样类的实例才能一起协作完成业务功能
* Spring 又是用容器来完成这个工作的，容器负责创建、组装和销毁这些类的实例，而应用只需要通过配置文件或者注解来告诉 Spring 类与类之间的关系
* 但是容器的概念不是 Spring 发明的，最开始来源于 Servlet 容器，并且 Servlet 容器也是通过配置文件来加载 Servlet 的
* Spring 框架就是对 Servlet 的封装，Spring 应用本身就是一个 Servlet，而 Servlet 容器是管理和运行 Servlet 的，因此需要先理解 Servlet 和 Servlet 容器是怎样工作的，才能更好地理解 Spring

# HTTP

## HTTP 的本质

* HTTP 协议是浏览器与服务器之间的数据传送协议（**约定好的通信格式**）。作为应用层协议，HTTP 是基于 TCP/IP 协议来传递数据的（HTML 文件、图片、查询结果等），HTTP 协议不涉及数据包（Packet）传输，主要规定了客户端和服务器之间的通信格式
* HTTP 是通信的方式，HTML 才是通信的目的

## HTTP工作原理

![image-20200721154958274](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721154958274.png)

1. 用户通过浏览器进行了一个操作，比如输入网址并回车，或者是点击链接，接着浏览器获取了这个事件
2. 浏览器向服务端发出 TCP 连接请求
3. 服务程序接受浏览器的连接请求，并经过 TCP 三次握手建立连接
4. 浏览器将请求数据打包成一个 HTTP 协议格式的数据包
5. 浏览器将该数据包推入网络，数据包经过网络传输，最终达到端服务程序
6. 服务端程序拿到这个数据包后，同样以 HTTP 协议格式解包，获取到客户端的意图
7. 得知客户端意图后进行处理，比如提供静态文件或者调用服务端程序获得动态结果
8. 服务器将响应结果（可能是 HTML 或者图片等）按照 HTTP 协议格式打包
9. 服务器将响应数据包推入网络，数据包经过网络传输最终达到到浏览器
10. 浏览器拿到数据包后，以 HTTP 协议的格式解包，然后解析数据，假设这里的数据是 HTML
11. 浏览器将 HTML 文件展示在页面上

*  Tomcat 和 Jetty 作为一个 HTTP 服务器，在这个过程中都做了些什么事情
  * 主要是接受连接、解析请求数据、处理请求和发送响应这几个步骤
  * 可能有成千上万的浏览器同时请求同一个 HTTP 服务器，因此 Tomcat 和 Jetty 为了提高服务的能力和并发度，往往会将自己要做的几个事情并行化，具体来说就是使用多线程的技术

## HTTP请求响应实例

* 那 HTTP 协议的数据包具体长什么样呢？

  * 用户在登陆页面输入用户名和密码，点击登陆后，浏览器发出了这样的 HTTP 请求

    ![image-20200721155502593](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721155502593.png)

  * HTTP 请求数据由三部分组成，分别是**请求行、请求报头、请求正文**。当这个 HTTP 请求数据到达 Tomcat 后，Tomcat 会把 HTTP 请求数据字节流解析成一个 Request 对象，这个 Request 对象封装了 HTTP 所有的请求信息。接着 Tomcat 把这个 Request 对象交给 Web 应用去处理，处理完后得到一个 Response 对象，Tomcat 会把这个 Response 对象转成 HTTP 格式的响应数据并发送给浏览器

  * HTTP 响应的格式，HTTP 的响应也是由三部分组成，分别是**状态行、响应报头、报文主体**

    ![image-20200721160442388](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721160442388.png)

## Cookie 和 Session

* HTTP 协议有个特点是无状态，请求与请求之间是没有关系的
  * Web 应用不知道你是谁
  * 因此 HTTP 协议需要一种技术让请求与请求之间建立起联系，并且服务器需要知道这个请求来自哪个用户，于是 Cookie 技术出现了
* **Cookie 技术**
  * **本质上就是一份存储在用户本地的文件，里面包含了每次请求中都需要传递的信息**
  * Cookie 是 HTTP 报文的一个请求头，Web 应用可以将用户的标识信息或者其他一些信息（用户名等）存储在 Cookie 中。用户经过验证之后，每次 HTTP 请求报文中都包含 Cookie，这样服务器读取这个 Cookie 请求头就知道用户是谁了
* **Session 技术**
  * 由于 Cookie 以明文的方式存储在本地，而 Cookie 中往往带有用户信息，这样就造成了非常大的安全隐患
  * 而 Session 的出现解决了这个问题，Session 可以理解为服务器端开辟的存储空间，里面保存了用户的状态，用户信息以 Session 的形式存储在服务端。当用户请求到来时，服务端可以把用户的请求和用户的 Session 对应起来
    *  Session 是怎么和请求对应起来的呢？答案是通过 Cookie，浏览器在 Cookie 中填充了一个 Session ID 之类的字段用来标识请求
  * 具体工作过程
    * 服务器在创建 Session 的同时，会为该 Session 生成唯一的 Session ID，当浏览器再次发送请求的时候，会将这个 Session ID 带上，服务器接受到请求之后就会依据 Session ID 找到相应的 Session，找到 Session 后，就可以在 Session 中获取或者添加内容了。而这些内容只会保存在服务器中，发到客户端的只有 Session ID，这样相对安全，也节省了网络流量，因为不需要在 Cookie 中存储大量用户信息
* **Session 创建与存储**
  * 在 Java 中，是 Web 应用程序在调用 HttpServletRequest 的 getSession 方法时，由 Web 容器（比如 Tomcat）创建的
  * Tomcat 的 Session 管理器提供了多种持久化方案来存储 Session，通常会采用高性能的存储方式，比如 Redis，并且通过集群部署的方式，防止单点故障，从而提升高可用。同时，Session 有过期时间，因此 Tomcat 会开启后台线程定期的轮询，如果 Session 过期了就将 Session 失效
* 在 HTTP/1.0 时期，每次 HTTP 请求都会创建一个新的 TCP 连接，请求完成后之后这个 TCP 连接就会被关闭。这种通信模式的效率不高，所以在 HTTP/1.1 中，引入了 HTTP 长连接的概念，使用长连接的 HTTP 协议，会在响应头加入 Connection:keep-alive。这样当浏览器完成一次请求后，浏览器和服务器之间的 TCP 连接不会关闭，再次访问这个服务器上的网页时，浏览器会继续使用这一条已经建立的连接，也就是说两个请求可能共用一个 TCP 连接

## ==Restful==

* REST是一种架构风格
* 将网络上的信息实体看作是资源，可以是图片、文件、一个服务...资源用URI统一标识，URI中没有动词哦，这是因为它是资源的标识，那怎么操作这些资源呢，于是定义一些动作：GET、POST、PUT和DELETE。通过URI+动作来操作一个资源
* 所谓的无状态说的是，为了完成一个操作，请求里包含了所有信息，你可以理解为服务端不需要保存请求的状态，也就是不需要保存session，没有session的好处是带来了服务端良好的可伸缩性，方便failover，请求被LB转到不同的server实例上没有差别。从这个角度看，正是有了REST架构风格的指导，才有了HTTP的无状态特性，顺便提一下，REST和HTTP1.1出自同一人之手。但是理想是丰满的，现实是骨感的，为了方便开发，大多数复杂的Web应用不得不在服务端保存Session。为了尽量减少Session带来的弊端，往往将Session集中存储到Redis上，而不是直接存储在server实例上

## Question

* 为什么说http是超文本传输协议，文本两字的含义是什么？http2.0所说的二进制帧，为什么说是二进制，和1.1格式上的本质区别是什么？再往下一层到TCP能否都看成二进制帧？
  * 文本可以理解为只有文字信息的文档，超文本是带有超链接的文档，可以链接到另一个文档，或一张图...
  * HTTP1.1是文本协议，HTTP2.0是二进制协议
  * 文本协议的协议数据是由ACSII字符组成的，比如文章里的HTTP请求的例子：请求行、请求头和请求体，我们一眼就看出什么意思。这是因为协议里的每个Byte都是用ACSII字符来解释的
  * 二进制协议的的每个Byte完全由协议本身来定义，比如一个Byte有8个Bit，这8个Bit可能有不同的意思（比如代表长度或者其他标志位），不一定代表一个ACSII字符
  * TCP是二进制协议

* 现在的web容器都支持将session存储在第三方中间件（如redis）中，为什么很多公司喜欢绕过容器，直接在应用中将会话数据存入中间件中？
  * 用Web容器的Session方案需要侵入特定的Web容器，用Spring Session可能比较简单，不需要跟特定的Servlet容器打交道
  * 这正是Spring喜欢做的事情，它使得程序员甚至感觉不到Servlet容器的存在，可以专心开发Web应用。但是Spring到底做了什么，Spring Session是如何实现的，我们还是有必要了解了解~
  * 其实它是通过Servlet规范中的Filter机制拦截了所有Servlet请求，偷梁换柱，将标准的Servlet请求对象包装了一下，换成它自己的Request包装类对象，这样当程序员通过包装后的Request对象的getSession方法拿Session时，是通过Spring拿Session，没Web容器什么事了
* 引入session是因为cookie存在客户端，有安全隐患；但是session id也是通过cookie由客户端发送到服务端，同样有安全隐患啊？
  * 虽然敏感的用户信息没有在网络上传输了，但是攻击者拿到sessionid也可以冒充受害者发送请求，这就是为什么我们需要https，加密后攻击者就拿不到sessionid了，另外CSRF也是一种防止session劫持的方式
* cookie跨域问题中，跨域是什么概念呢
  * cookie有两个重要属性：
    * domain字段 ：表示浏览器访问这个域名时才带上这个cookie
    * path字段：表示访问的URL是这个path或者子路径时才带上这个cookie
  * 跨域说的是，访问两个不同的域名或路径时，希望带上同一个cookie，跨域的具体实现方式有很多..
* 什么叫内嵌方式运行servlet容器
  * 就是你的程序比如SpringBoot直接调用Web容器的提供的API去创建一个Web容器（HTTP服务器和Servlet容器），同时你的程序注册一个Servlet到Servlet容器中，比如SpringMVC的DispatcherServlet，这样请求到达时，Servlet容器负责调用你的Servlet

# Servlet规范和Servlet容器

* Servlet 容器用来加载和管理业务类。HTTP 服务器不直接跟业务类打交道，而是把请求交给 Servlet 容器去处理，Servlet 容器会将请求转发到具体的 Servlet，如果这个 Servlet 还没创建，就加载并实例化这个 Servlet，然后调用这个 Servlet 的接口方法。因此，Servlet 接口其实是**Servlet 容器跟具体业务类之间的接口**

  ![image-20200721165348433](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721165348433.png)

  * HTTP 服务器不直接调用业务类，而是把请求交给容器来处理，容器通过 Servlet 接口调用业务类。因此 Servlet 接口和 Servlet 容器的出现，达到了 HTTP 服务器与业务类解耦的目的

* Servlet 接口和 Servlet 容器这一整套规范叫作 Servlet 规范

  * Tomcat 和 Jetty 都按照 Servlet 规范的要求实现了 Servlet 容器，同时它们也具有 HTTP 服务器的功能。作为 Java 程序员，如果要实现新的业务功能，只需要实现一个 Servlet，并把它注册到 Tomcat（Servlet 容器）中，剩下的事情就由 Tomcat 帮我们处理了

## Servlet接口

* 定义

  ```java
  public interface Servlet {
      void init(ServletConfig config) throws ServletException;
      
      ServletConfig getServletConfig();
      
      void service(ServletRequest req, ServletResponse res）throws ServletException, IOException;
      
      String getServletInfo();
      
      void destroy();
  }
  ```
  * service：具体业务类在这个方法里实现处理逻辑，两个参数，ServletRequest 用来封装请求信息，ServletResponse 用来封装响应信息，因此**本质上这两个类是对通信协议的封装**
    * 比如 HTTP 协议中的请求和响应就是对应了 HttpServletRequest 和 HttpServletResponse 这两个类
    * 可以通过 HttpServletRequest 来获取所有请求相关的信息，包括请求路径、Cookie、HTTP 头、请求参数等
    * 还可以通过 HttpServletRequest 来创建和获取 Session。而 HttpServletResponse 是用来封装 HTTP 响应的
  * 两个跟生命周期有关的方法 init 和 destroy
    * Servlet 容器在加载 Servlet 类的时候会调用 init 方法，在卸载的时候会调用 destroy 方法。可能会在 init 方法里初始化一些资源，并在 destroy 方法里释放这些资源，比如 Spring MVC 中的 DispatcherServlet，就是在 init 方法里创建了自己的 Spring 容器
  * ServletConfig 的作用就是封装 Servlet 的初始化参数。可以在 web.xml 给 Servlet 配置参数，并在程序里通过 getServletConfig 方法拿到这些参数

* 有接口一般就有抽象类，抽象类用来实现接口和封装通用的逻辑，因此 Servlet 规范提供了 GenericServlet 抽象类，可以通过扩展它来实现 Servlet。虽然 Servlet 规范并不在乎通信协议是什么，但是大多数的 Servlet 都是在 HTTP 环境中处理的，因此 Servet 规范还提供了 HttpServlet 来继承 GenericServlet，并且加入了 HTTP 特性。这样通过继承 HttpServlet 类来实现自己的 Servlet，只需要重写两个方法：doGet 和 doPost

## Servlet 容器

* 为了解耦，HTTP 服务器不直接调用 Servlet，而是把请求交给 Servlet 容器来处理

* 工作流程

  * 当客户请求某个资源时，HTTP 服务器会用一个 ServletRequest 对象把客户的请求信息封装起来，然后调用 Servlet 容器的 service 方法，Servlet 容器拿到请求后，根据请求的 URL 和 Servlet 的映射关系，找到相应的 Servlet，如果 Servlet 还没有被加载，就用反射机制创建这个 Servlet，并调用 Servlet 的 init 方法来完成初始化，接着调用 Servlet 的 service 方法来处理请求，把 ServletResponse 对象返回给 HTTP 服务器，HTTP 服务器会把响应发送给客户端

    ![image-20200721170244460](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721170244460.png)

## **Web 应用**

* Servlet 容器会实例化和调用 Servlet
* 以 Web 应用程序的方式来部署 Servlet 的，而根据 Servlet 规范，Web 应用程序有一定的目录结构，在这个目录下分别放置了 Servlet 的类文件、配置文件以及静态资源，Servlet 容器通过读取配置文件，就能找到并加载 Servlet
* Web 应用的目录结构
  * MyWebApp
    * WEB-INF/web.xml        -- 配置文件，用来配置 Servlet 等
    * WEB-INF/lib/                 -- 存放 Web 应用所需各种 JAR 包
    * WEB-INF/classes/         -- 存放你的应用类，比如 Servlet 类
    * META-INF/                     -- 目录存放工程的一些信息
* Servlet 规范里定义了**ServletContext**这个接口来对应一个 Web 应用
  * Web 应用部署好后，Servlet 容器在启动时会加载 Web 应用，并为每个 Web 应用创建唯一的 ServletContext 对象
  * 可以把 ServletContext 看成是一个全局对象，一个 Web 应用可能有多个 Servlet，这些 Servlet 可以通过全局的 ServletContext 来共享数据，这些数据包括 Web 应用的初始化参数、Web 应用目录下的文件资源等。由于 ServletContext 持有所有 Servlet 实例，你还可以通过它来实现 Servlet 请求的转发

## 扩展机制

* 引入了 Servlet 规范后，不需要关心 Socket 网络通信、不需要关心 HTTP 协议，也不需要关心业务类是如何被实例化和调用的，因为这些都被 Servlet 规范标准化了，只要关心怎么实现的你的业务逻辑
* Servlet 规范提供了两种扩展机制：**Filter**和**Listener**
* **Filter**是过滤器
  * 这个接口允许对请求和响应做一些统一的定制化处理，比如可以根据请求的频率来限制访问，或者根据国家地区的不同来修改响应内容
  * 过滤器的工作原理
    * Web 应用部署完成后，Servlet 容器需要实例化 Filter 并把 Filter 链接成一个 FilterChain。当请求进来时，获取第一个 Filter 并调用 doFilter 方法，doFilter 方法负责调用这个 FilterChain 中的下一个 Filter
* **Listener**是监听器
  * 当 Web 应用在 Servlet 容器中运行时，Servlet 容器内部会不断的发生各种事件，如 Web 应用的启动和停止、用户请求到达等。 Servlet 容器提供了一些默认的监听器来监听这些事件，当事件发生时，Servlet 容器会负责调用监听器的方法
  * 当然，可以定义自己的监听器去监听感兴趣的事件，将监听器配置在 web.xml 中。比如 Spring 就实现了自己的监听器，来监听 ServletContext 的启动事件，目的是当 Servlet 容器启动时，创建并初始化全局的 Spring 容器

## 总结

* Servlet 本质上是一个接口，实现了 Servlet 接口的业务类也叫 Servlet。Servlet 接口其实是 Servlet 容器跟具体 Servlet 业务类之间的接口。Servlet 接口跟 Servlet 容器这一整套规范叫作 Servlet 规范，而 Servlet 规范使得程序员可以专注业务逻辑的开发，同时 Servlet 规范也给开发者提供了扩展的机制 Filter 和 Listener
* **Filter 是干预过程的**，它是过程的一部分，是基于过程行为的
* **Listener 是基于状态的**，任何行为改变同一个状态，触发的事件是一致的
* Tomcat&Jetty在启动时给每个Web应用创建一个全局的上下文环境，这个上下文就是ServletContext，其为后面的Spring容器提供宿主环境
* Tomcat&Jetty在启动过程中触发容器初始化事件，Spring的ContextLoaderListener会监听到这个事件，它的contextInitialized方法会被调用，在这个方法中，Spring会初始化全局的Spring根容器，这个就是Spring的IoC容器，IoC容器初始化完毕后，Spring将其存储到ServletContext中，便于以后来获取
* Tomcat&Jetty在启动过程中还会扫描Servlet，一个Web应用中的Servlet可以有多个，以SpringMVC中的DispatcherServlet为例，这个Servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个Servlet请求
* Servlet一般会延迟加载，当第一个请求达到时，Tomcat&Jetty发现DispatcherServlet还没有被实例化，就调用DispatcherServlet的init方法，DispatcherServlet在初始化的时候会建立自己的容器，叫做SpringMVC 容器，用来持有Spring MVC相关的Bean。同时，Spring MVC还会通过ServletContext拿到Spring根容器，并将Spring根容器设为SpringMVC容器的父容器，请注意，Spring MVC容器可以访问父容器中的Bean，但是父容器不能访问子容器的Bean， 也就是说Spring根容器不能访问SpringMVC容器里的Bean。说的通俗点就是，在Controller里可以访问Service对象，但是在Service里不可以访问Controller对象

## servlet容器，web容器，spring容器，springmvc容器的区别

![image-20200721171833296](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721171833296.png)

* web容器中有servlet容器，spring项目部署后存在spring容器。其中spring控制service层和dao层的bean对象以及controller层bean对象。servlet容器控制servlet对象。项目启动是，首先 servlet初始化，初始化过程中通过web.xml中spring的配置加载spring配置，初始化spring容器。待容器加载完成。servlet初始化完成，则完成启动。springmvc是viewAndModie的请求传递和结果解析。本身并没有容器管理，都是交给spring管理
* HTTP请求到达web容器后，会到达Servlet容器，容器通过分发器分发到具体的spring的Controller层。执行业务操作后返回结果

## servlet容器的Filter跟 spring 的intercepter 有啥区别

*  Filter是Servlet规范的一部分，是Servlet容器Tomcat实现的。Intercepter是Spring发明的
* 执行顺序
  * Filter.doFilter();
  * HandlerInterceptor.preHandle();
  * Controller
  * HandlerInterceptor.postHandle();
  * DispatcherServlet 渲染视图
  * HandlerInterceptor.afterCompletion();
  * Filter.doFilter(); Servlet方法返回

## ==Spring、SpringMVC==

* Spring和SpringMVC分别有自己的IOC容器或者说上下文
* 为什么要分成两个容器呢？为了职责划分清晰
  * SpringMVC的容器直接管理跟DispatcherServlet相关的Bean，也就是Controller，ViewResolver等，并且SpringMVC容器是在DispacherServlet的init方法里创建的。而Spring容器管理其他的Bean比如Service和DAO
  * 并且SpringMVC容器是Spring容器的子容器，所谓的父子关系意味着什么呢，就是你通过子容器去拿某个Bean时，子容器先在自己管理的Bean中去找这个Bean，如果找不到再到父容器中找。但是父容器不能到子容器中去找某个Bean
  * 其实这个套路跟JVM的类加载器设计有点像，不同的类加载器也为了隔离，不过加载顺序是反的，子加载器总是先委托父加载器去加载某个类，加载不到再自己来加载

# Servlet

* 纯手工编写一个 Servlet，并在 Tomcat 中运行起来

* 主要步骤

  1. 下载并安装 Tomcat。
  2. 编写一个继承 HttpServlet 的 Java 类。
  3. 将 Java 类文件编译成 Class 文件。
  4. 建立 Web 应用的目录结构，并配置 web.xml。
  5. 部署 Web 应用。
  6. 启动 Tomcat。
  7. 浏览器访问验证结果。
  8. 查看 Tomcat 日志

* Servlet 3.0 规范支持用注解的方式来部署 Servlet，不需要在 web.xml 里配置

* tomcat目录

  * /bin：存放 Windows 或 Linux 平台上启动和关闭 Tomcat 的脚本文件
  * /conf：存放 Tomcat 的各种全局配置文件，其中最重要的是 server.xml
  * /lib：存放 Tomcat 以及所有 Web 应用都可以访问的 JAR 文件
  * /logs：存放 Tomcat 执行时产生的日志文件
  * /work：存放 JSP 编译后产生的 Class 文件
  * /webapps：Tomcat 的 Web 应用目录，默认情况下把 Web 应用放在这个目录下

* **编写一个继承 HttpServlet 的 Java 类**

  ```java
  // 创建一个 Java 类去继承 HttpServlet 类，并重写 doGet 和 doPost 方法
  public class MyServlet extends HttpServlet {
   
      @Override
      protected void doGet(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {
   
          System.out.println("MyServlet 在处理 get（）请求...");
          PrintWriter out = response.getWriter();
          response.setContentType("text/html;charset=utf-8");
          out.println("<strong>My Servlet!</strong><br>");
      }
   
      @Override
      protected void doPost(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {
   
          System.out.println("MyServlet 在处理 post（）请求...");
          PrintWriter out = response.getWriter();
          response.setContentType("text/html;charset=utf-8");
          out.println("<strong>My Servlet!</strong><br>");
      }
  }
  ```

* **将 Java 文件编译成 Class 文件**

  * 需要把 Tomcat lib 目录下的 servlet-api.jar 拷贝到当前目录下，这是因为 servlet-api.jar 中定义了 Servlet 接口，而我们的 Servlet 类实现了 Servlet 接口，因此编译 Servlet 类需要这个 JAR 包

    ```shell
    javac -cp ./servlet-api.jar MyServlet.java
    ```

  * 编译成功后，会在当前目录下找到一个叫 MyServlet.class 的文件

* **建立 Web 应用的目录结构**

  * Servlet 是放到 Web 应用部署到 Tomcat 的，而 Web 应用具有一定的目录结构

  * 按照要求建立 Web 应用文件夹，名字叫 MyWebApp，然后在这个目录下建立子文件夹

    ```shell
    MyWebApp/WEB-INF/web.xml
    MyWebApp/WEB-INF/classes/MyServlet.class
    ```

  * 然后在 web.xml 中配置 Servlet

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
      version="4.0"
      metadata-complete="true">
     
        <description> Servlet Example. </description>
        <display-name> MyServlet Example </display-name>
        <request-character-encoding>UTF-8</request-character-encoding>
     
        <servlet>
          <servlet-name>myServlet</servlet-name>
          <servlet-class>MyServlet</servlet-class>
        </servlet>
     
        <servlet-mapping>
          <servlet-name>myServlet</servlet-name>
          <url-pattern>/myservlet</url-pattern>
        </servlet-mapping>
     
    </web-app>
    ```

* **部署 Web 应用**

  * Tomcat 应用的部署非常简单，将这个目录 MyWebApp 拷贝到 Tomcat 的安装目录下的 webapps 目录即可

* **启动 Tomcat**

  * 找到 Tomcat 安装目录下的 bin 目录，根据操作系统的不同，执行相应的启动脚本。如果是 Windows 系统，执行`startup.bat`.；如果是 Linux 系统，则执行`startup.sh`

* **浏览访问验证结果**

  * 在浏览器里访问这个 URL：`http://localhost:8080/MyWebApp/myservlet`
  * 访问 URL 路径中的 MyWebApp 是 Web 应用的名字，myservlet 是在 web.xml 里配置的 Servlet 的路径

* **查看 Tomcat 日志**

  * 打开 Tomcat 的日志目录，也就是 Tomcat 安装目录下的 logs 目录。Tomcat 的日志信息分为两类 ：一是运行日志，它主要记录运行过程中的一些信息，尤其是一些异常错误日志信息 ；二是访问日志，它记录访问的时间、IP 地址、访问的路径等相关信息
  * catalina.***.log
    * 主要是记录 Tomcat 启动过程的信息，在这个文件可以看到启动的 JVM 参数以及操作系统等日志信息
  * catalina.out
    * catalina.out 是 Tomcat 的标准输出（stdout）和标准错误（stderr），这是在 Tomcat 的启动脚本里指定的，如果没有修改的话 stdout 和 stderr 会重定向到这里。所以在这个文件里可以看到我们在 MyServlet.java 程序里打印出来的信息
  * localhost.**.log
    * 主要记录 Web 应用在初始化过程中遇到的未处理的异常，会被 Tomcat 捕获而输出这个日志文件
  * localhost_access_log.**.txt
    * 存放访问 Tomcat 的请求日志，包括 IP 地址以及请求的路径、时间、请求协议以及状态码等信息
  * `manager.***.log/host-manager.***.log`
    * 存放 Tomcat 自带的 manager 项目的日志信息

* **用注解的方式部署 Servlet**

  ```java
  // 先修改 Java 代码，给 Servlet 类加上@WebServlet注解
  // 第一层意思是 AnnotationServlet 这个 Java 类是一个 Servlet，第二层意思是这个 Servlet 对应的 URL 路径是 myAnnotationServlet
  @WebServlet("/myAnnotationServlet")
  public class AnnotationServlet extends HttpServlet {
   
      @Override
      protected void doGet(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {
          // ...
      }
   
      @Override
      protected void doPost(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {
  				// ...
      }
  }  
  ```
  * **需要删除原来的 web.xml**，因为不需要 web.xml 来配置 Servlet 了

# Tomcat系统架构

* Tomcat 的整体架构包含了两个核心组件连接器和容器。连接器负责对外交流，容器负责内部处理。连接器用 ProtocolHandler 接口来封装通信协议和 I/O 模型的差异，ProtocolHandler 内部又分为 EndPoint 和 Processor 模块，EndPoint 负责底层 Socket 通信，Proccesor 负责应用层协议解析。连接器通过适配器 Adapter 调用容器
* 可以得到一些设计复杂系统的基本思路。首先要分析需求，根据高内聚低耦合的原则确定子模块，然后找出子模块中的变化点和不变点，用接口和抽象基类去封装不变点，在抽象基类中定义模板方法，让子类自行实现抽象方法，也就是具体子类去实现变化点

## Tomcat总体架构

* Tomcat 要实现 2 个核心功能
  * 处理 Socket 连接，负责网络字节流与 Request 和 Response 对象的转化
  * 加载和管理 Servlet，以及具体处理 Request 请求
* **Tomcat 设计了两个核心组件连接器（Connector）和容器（Container）来分别做这两件事情。连接器负责对外交流，容器负责内部处理**

* Tomcat 支持的 I/O 模型有

  * NIO：非阻塞 I/O，采用 Java NIO 类库实现。
  * NIO2：异步 I/O，采用 JDK 7 最新的 NIO2 类库实现
  * APR：采用 Apache 可移植运行库实现，是 C/C++ 编写的本地库

* Tomcat 支持的应用层协议有

  * HTTP/1.1：这是大部分 Web 应用采用的访问协议
  * AJP：用于和 Web 服务器集成（如 Apache）
  * HTTP/2：HTTP 2.0 大幅度的提升了 Web 性能

* Tomcat 为了实现支持多种 I/O 模型和应用层协议，一个容器可能对接多个连接器，就好比一个房间有多个门。但是单独的连接器或者容器都不能对外提供服务，需要把它们组装起来才能工作，组装后这个整体叫作 Service 组件

* Service 本身没有做什么重要的事情，只是在连接器和容器外面多包了一层，把它们组装在一起

* Tomcat 内可能有多个 Service，这样的设计也是出于灵活性的考虑。通过在 Tomcat 中配置多个 Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用

  ![image-20200721175748911](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721175748911.png)

  * 最顶层是 Server，这里的 Server 指的就是一个 Tomcat 实例。一个 Server 中有一个或者多个 Service，一个 Service 中有多个连接器和一个容器。连接器与容器之间通过标准的 ServletRequest 和 ServletResponse 通信

## 连接器

* 连接器对 Servlet 容器屏蔽了协议及 I/O 模型等的区别，无论是 HTTP 还是 AJP，在容器中获取到的都是一个标准的 ServletRequest 对象

* 把连接器的功能需求进一步细化，比如：

  * 监听网络端口。
  * 接受网络连接请求。
  * 读取请求网络字节流。
  * 根据具体应用层协议（HTTP/AJP）解析字节流，生成统一的 Tomcat Request 对象。
  * 将 Tomcat Request 对象转成标准的 ServletRequest。
  * 调用 Servlet 容器，得到 ServletResponse。
  * 将 ServletResponse 转成 Tomcat Response 对象。
  * 将 Tomcat Response 转成网络字节流。
  * 将响应字节流写回给浏览器

* 连接器应该有哪些子模块？优秀的模块化设计应该考虑**高内聚、低耦合**

  * **高内聚**是指相关度比较高的功能要尽可能集中，不要分散
  * **低耦合**是指两个相关的模块要尽可能减少依赖的部分和降低依赖的程度，不要让两个模块产生强依赖

* 连接器需要完成 3 个**高内聚**的功能

  * 网络通信
  * 应用层协议解析
  * Tomcat Request/Response 与 ServletRequest/ServletResponse 的转化

* 因此 Tomcat 的设计者设计了 3 个组件来实现这 3 个功能，分别是 EndPoint、Processor 和 Adapter

* 组件之间通过抽象接口交互

  * 这样做还有一个好处是**封装变化。**这是面向对象设计的精髓，将系统中经常变化的部分和稳定的部分隔离，有助于增加复用性，并降低系统耦合度
  * 网络通信的 I/O 模型是变化的，可能是非阻塞 I/O、异步 I/O 或者 APR。应用层协议也是变化的，可能是 HTTP、HTTPS、AJP。浏览器端发送的请求信息也是变化的
  * 但是整体的处理逻辑是不变的，EndPoint 负责提供字节流给 Processor，Processor 负责提供 Tomcat Request 对象给 Adapter，Adapter 负责提供 ServletRequest 对象给容器
  * 如果要支持新的 I/O 方案、新的应用层协议，只需要实现相关的具体子类，上层通用的处理逻辑是不变的

* 由于 I/O 模型和应用层协议可以自由组合，比如 NIO + HTTP 或者 NIO2 + AJP

  * Tomcat 的设计者将网络通信和应用层协议解析放在一起考虑，设计了一个叫 ProtocolHandler 的接口来封装这两种变化点
  * 各种协议和通信模型的组合有相应的具体实现类。比如：Http11NioProtocol 和 AjpNioProtocol

* 系统也存在一些相对稳定的部分，因此 Tomcat 设计了一系列抽象基类来**封装这些稳定的部分**，抽象基类 AbstractProtocol 实现了 ProtocolHandler 接口

  * 每一种应用层协议有自己的抽象基类，比如 AbstractAjpProtocol 和 AbstractHttp11Protocol，具体协议的实现类扩展了协议层抽象基类。它们的继承关系

  ![image-20200721180318983](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721180318983.png)

  * 这样设计的目的是尽量将稳定的部分放到抽象基类，同时每一种 I/O 模型和协议的组合都有相应的具体实现类，在使用时可以自由选择

* 小结一下，连接器模块用三个核心组件：Endpoint、Processor 和 Adapter 来分别做三件事情，其中 Endpoint 和 Processor 放在一起抽象成了 ProtocolHandler 组件

  ![image-20200721180431852](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721180431852.png)

## **ProtocolHandler 组件**

* 连接器用 ProtocolHandler 来处理网络连接和应用层协议，包含了 2 个重要部件：EndPoint 和 Processor

* EndPoint

  * EndPoint 是通信端点，即通信监听的接口，是具体的 Socket 接收和发送处理器，是对传输层的抽象，因此 EndPoint 是用来实现 TCP/IP 协议的
  * EndPoint 是一个接口，对应的抽象实现类是 AbstractEndpoint，而 AbstractEndpoint 的具体子类，比如在 NioEndpoint 和 Nio2Endpoint 中，有两个重要的子组件：Acceptor 和 SocketProcessor
  * 其中 Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求，它实现 Runnable 接口，在 Run 方法里调用协议处理组件 Processor 进行处理。为了提高处理能力，SocketProcessor 被提交到线程池来执行。而这个线程池叫作执行器（Executor)

* Processorss

  * 如果说 EndPoint 是用来实现 TCP/IP 协议的，那么 Processor 用来实现 HTTP 协议，Processor 接收来自 EndPoint 的 Socket，读取字节流解析成 Tomcat Request 和 Response 对象，并通过 Adapter 将其提交到容器处理，Processor 是对应用层协议的抽象

  * Processor 是一个接口，定义了请求的处理等方法。它的抽象实现类 AbstractProcessor 对一些协议共有的属性进行封装，没有对方法进行实现。具体的实现有 AJPProcessor、HTTP11Processor 等，这些具体实现类实现了特定协议的解析方法和请求处理方式

    ![image-20200721180941503](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721180941503.png)

    * EndPoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务提交到线程池去处理，SocketProcessor 的 Run 方法会调用 Processor 组件去解析应用层协议，Processor 通过解析生成 Request 对象后，会调用 Adapter 的 Service 方法

## **Adapter 组件**

* 由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat 定义了自己的 Request 类来“存放”这些请求信息。
* ProtocolHandler 接口负责解析请求并生成 Tomcat Request 类。但是这个 Request 对象不是标准的 ServletRequest，也就意味着，不能用 Tomcat Request 作为参数来调用容器。Tomcat 设计者的解决方案是引入 CoyoteAdapter，这是适配器模式的经典运用，连接器调用 CoyoteAdapter 的 Sevice 方法，传入的是 Tomcat Request 对象，CoyoteAdapter 负责将 Tomcat Request 转成 ServletRequest，再调用容器的 Service 方法

## 总结

* 用户可以在server.xml中直接指定想要的连接器类型：比如Http11NioProtocol和Http11Nio2Protocol
* 如果连接器直接创建ServletRequest和ServletResponse对象的话，就和Servlet协议耦合了，设计者认为连接器尽量保持独立性，它不一定要跟Servlet容器工作的。另外对象转化的性能消耗还是比较少的，Tomcat对HTTP请求体采取了延迟解析的策略，也就是说，TomcatRequest对象转化成ServletRequest的时候，请求体的内容都还没读取呢，直到容器处理这个请求的时候才读取的
* 一个连接器对应一个监听端口，比如一扇门，一个web应用是一个业务部门，进了这个门后你可以到各个业务部门去办事
* Service是对外提供服务的单位，一个Service对应一个容器，多个Service就有多个容器
* 可以把Netty理解成Tomcat中的连接器，它们都负责网络通信，都利用了Java NIO非阻塞特性。但Netty素以高性能高并发著称，为什么Tomcat不把连接器替换成Netty呢？第一个原因是Tomcat的连接器性能已经足够好了，同样是Java NIO编程，套路都差不多。第二个原因是Tomcat做为Web容器，需要考虑到Servlet规范，Servlet规范规定了对HTTP Body的读写是阻塞的，因此即使用到了Netty，也不能充分发挥它的优势。所以Netty一般用在非HTTP协议和Servlet的场景下

## 容器

* Tomcat 有两个核心组件：连接器和容器，其中连接器负责外部交流，容器负责内部处理。具体来说就是，连接器处理 Socket 通信和应用层协议的解析，得到 Servlet 请求；而容器则负责处理 Servlet 请求

* 容器的层次结构

  * Tomcat 设计了 4 种容器，分别是 Engine、Host、Context 和 Wrapper。这 4 种容器不是平行关系，而是父子关系

    ![image-20200722135618808](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722135618808.png)

    * **Tomcat 通过一种分层的架构，使得 Servlet 容器具有很好的灵活性**

  * Context 表示一个 Web 应用程序；Wrapper 表示一个 Servlet，一个 Web 应用程序中可能会有多个 Servlet；Host 代表的是一个虚拟主机，或者说一个站点，可以给 Tomcat 配置多个虚拟主机地址，而一个虚拟主机下可以部署多个 Web 应用程序；Engine 表示引擎，用来管理多个虚拟站点，一个 Service 最多只能有一个 Engine

  * server.xml

    ![image-20200722135934198](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722135934198.png)

    * Tomcat 就是用组合模式来管理这些容器的。具体实现方法是，所有容器组件都实现了 Container 接口，因此组合模式可以使得用户对单容器对象和组合容器对象的使用具有一致性。这里单容器对象指的是最底层的 Wrapper，组合容器对象指的是上面的 Context、Host 或者 Engine

      ```java
      // LifeCycle 接口用来统一管理各组件的生命周期
      public interface Container extends Lifecycle {
          public void setName(String name);
          public Container getParent();
          public void setParent(Container container);
          public void addChild(Container child);
          public void removeChild(Container child);
          public Container findChild(String name);
      }
      ```

## 请求定位 Servlet 的过程

* Tomcat 是用 Mapper 组件来确定请求是由哪个 Wrapper 容器里的 Servlet 来处理

* Mapper 组件的功能就是将用户请求的 URL 定位到一个 Servlet

  * 它的工作原理是：Mapper 组件里保存了 Web 应用的配置信息，其实就是**容器组件与访问路径的映射关系**，比如 Host 容器里配置的域名、Context 容器里的 Web 应用路径，以及 Wrapper 容器里 Servlet 映射的路径，可以想象这些配置信息就是一个多层次的 Map
  * 当一个请求到来时，Mapper 组件通过解析请求 URL 里的域名和路径，再到自己保存的 Map 里去查找，就能定位到一个 Servlet。请注意，一个请求 URL 最后只会定位到一个 Wrapper 容器，也就是一个 Servlet

* 例子

  * 假如有一个网购系统，有面向网站管理人员的后台管理系统，还有面向终端客户的在线购物系统。

  * 这两个系统跑在同一个 Tomcat 上，为了隔离它们的访问域名，配置了两个虚拟域名：`manage.shopping.com`和`user.shopping.com`，网站管理人员通过`manage.shopping.com`域名访问 Tomcat 去管理用户和商品，而用户管理和商品管理是两个单独的 Web 应用。终端客户通过`user.shopping.com`域名去搜索商品和下订单，搜索功能和订单管理也是两个独立的 Web 应用

  * 针对这样的部署，Tomcat 会创建一个 Service 组件和一个 Engine 容器组件，在 Engine 容器下创建两个 Host 子容器，在每个 Host 容器下创建两个 Context 子容器。由于一个 Web 应用通常有多个 Servlet，Tomcat 还会在每个 Context 容器里创建多个 Wrapper 子容器。每个容器都有对应的访问路径

    ![image-20200722140714220](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722140714220.png)

    * 假如有用户访问一个 URL，比如图中的`http://user.shopping.com:8080/order/buy`，Tomcat 如何将这个 URL 定位到一个 Servlet 呢？

    * **首先，根据协议和端口号选定 Service 和 Engine**

      * Tomcat 的每个连接器都监听不同的端口，比如 Tomcat 默认的 HTTP 连接器监听 8080 端口、默认的 AJP 连接器监听 8009 端口
      *  URL 访问的是 8080 端口，因此这个请求会被 HTTP 连接器接收，而一个连接器是属于一个 Service 组件的，这样 Service 组件就确定了
      * 知道一个 Service 组件里除了有多个连接器，还有一个容器组件，具体来说就是一个 Engine 容器，因此 Service 确定了也就意味着 Engine 也确定了

    * **然后，根据域名选定 Host**

      * Service 和 Engine 确定后，Mapper 组件通过 URL 中的域名去查找相应的 Host 容器
      * 比如例子中的 URL 访问的域名是`user.shopping.com`，因此 Mapper 会找到 Host2 这个容器

    * **之后，根据 URL 路径找到 Context 组件**

      * Host 确定以后，Mapper 根据 URL 的路径来匹配相应的 Web 应用的路径
      * 比如例子中访问的是 /order，因此找到了 Context4 这个 Context 容器

    * **最后，根据 URL 路径找到 Wrapper（Servlet）**

      * Context 确定后，Mapper 再根据 web.xml 中配置的 Servlet 映射路径来找到具体的 Wrapper 和 Servlet

    * 并不是说只有 Servlet 才会去处理请求，实际上这个查找路径上的父子容器都会对请求做一些处理。连接器中的 Adapter 会调用容器的 Service 方法来执行 Servlet，最先拿到请求的是 Engine 容器，Engine 容器对请求做一些处理后，会把请求传给自己子容器 Host 继续处理，依次类推，最后这个请求会传给 Wrapper 容器，Wrapper 会调用最终的 Servlet 来处理。那么这个调用过程具体是怎么实现的呢？答案是使用 Pipeline-Valve 管道

      * Pipeline-Valve 是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将再调用下一个处理者继续处理

    * Valve 表示一个处理点，比如权限认证和记录日志

      ```java
      public interface Valve {
        // 一个链表将 Valve 链起来了
        public Valve getNext();
        public void setNext(Valve valve);
        // 由于 Valve 是一个处理点，因此 invoke 方法就是来处理请求的
        public void invoke(Request request, Response response)
      }
      ```

      ```java
      public interface Pipeline extends Contained {
        public void addValve(Valve valve);
        public Valve getBasic();
        public void setBasic(Valve valve);
        public Valve getFirst();
      }
      ```

      * Pipeline 中有 addValve 方法。Pipeline 中维护了 Valve 链表，Valve 可以插入到 Pipeline 中，对请求做某些处理。还发现 Pipeline 中没有 invoke 方法，因为整个调用链的触发是 Valve 来完成的，Valve 完成自己的处理后，调用 getNext.invoke() 来触发下一个 Valve 调用

      * 每一个容器都有一个 Pipeline 对象，只要触发这个 Pipeline 的第一个 Valve，这个容器里 Pipeline 中的 Valve 就都会被调用到。但是，不同容器的 Pipeline 是怎么链式触发的呢，比如 Engine 中 Pipeline 需要调用下层容器 Host 中的 Pipeline

        * 这是因为 Pipeline 中还有个 getBasic 方法。这个 BasicValve 处于 Valve 链表的末端，它是 Pipeline 中必不可少的一个 Valve，负责调用下层容器的 Pipeline 里的第一个 Valve

          ![image-20200722141603728](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722141603728.png)

        * 整个调用过程由连接器中的 Adapter 触发的，它会调用 Engine 的第一个 Valve：

          ```java
          // Calling the container
          connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);
          ```

        * Wrapper 容器的最后一个 Valve 会创建一个 Filter 链，并调用 doFilter() 方法，最终会调到 Servlet 的 service 方法

    * Valve 和 Filter 区别

      * Valve 是 Tomcat 的私有机制，与 Tomcat 的基础架构 /API 是紧耦合的。Servlet API 是公有的标准，所有的 Web 容器包括 Jetty 都支持 Filter 机制
      *  Valve 工作在 Web 容器级别，拦截所有应用的请求；而 Servlet Filter 工作在应用级别，只能拦截某个 Web 应用的所有请求。如果想做整个 Web 容器的拦截器，必须通过 Valve 来实现

  ## 总结

  * Tomcat 设计了多层容器是为了灵活性的考虑，灵活性具体体现在一个 Tomcat 实例（Server）可以有多个 Service，每个 Service 通过多个连接器监听不同的端口，而一个 Service 又可以支持多个虚拟主机。一个 URL 网址可以用不同的主机名、不同的端口和不同的路径来访问特定的 Servlet 实例

  * 请求的链式调用是基于 Pipeline-Valve 责任链来完成的，这样的设计使得系统具有良好的可扩展性，如果需要扩展容器本身的功能，只需要增加相应的 Valve 即可

  * Basic value 有些疑惑。比如engine容器下有多个host容器，那engine容器的basic value是怎么知道要指向哪个host容器的value

    * Mapper组件在映射请求的时候，会在Request对象中存储相应的Host、Context等对象，这些选定的容器用来处理这个特定的请求，因此Engine中的Valve是从Request对象拿到Host容器的

  * tomcat多host的配置

    ```xml
    <server port=“8005” shutdown=“SHUTDOWN”>
    	<service name=“Catalina”>
    		<engine defaulthost=“localhost” name=“Catalina”>
    			<host appbase=“webapps” autodeploy=“true” name=“localhost” unpackwars=“true”></host>
    			<host appbase=“webapps1” autodeploy=“true” name=“www.domain1.com” unpackwars=“true”></host>
    			<host appbase=“webapps2” autodeploy=“true” name=“www.domain2.com” unpackwars=“true”></host>
    			<host appbase=“webapps3” autodeploy=“true” name=“www.domain3.com” unpackwars=“true”></host>
    		</engine>
    	</service>
    </server>
    ```

  * 在同一个Tomcat实例里部署多个Web应用是为了节省内存等资源，不过配置部署有点复杂，应用之间互相影响，加上现在硬件成本将低，多应用部署比较少见了
  * Servlet接口中定义了service方法，没有doGet/doPost。HttpServlet是一个实现类，实现了service方法，同时留出了doGet/doPost方法让程序员来实现
  * 可以通过web.xml配置一个或多个Filter，Servlet容器在调用Servlet的service之前，需要调用这些Filter，于是把这些Filter创建出来，形成链表，依次调用，这个Filter链中的最后一个Filter会负责调用Servlet的service方法
  * Spring没有实现ServletContext接口，是Tomcat实现了这个接口，Spring可以拿到通过Request对象拿到ServletContext
  * DNS解析是在客户端完成的，你需要在客户端把两个域名指向同一个IP，可以通过hosts配置，因此只要你使用的端口相同，两个域名其实访问的是同一个Service组件。而Tomcat设计通过请求URL中Host来转发到不同Host组件的
  * 不同的连接器代表一种通信的路径，比如同时支持HTTP和HTTPS，不是为了并发
  * 域名相同指向不同的Web应用，默认情况下只需要把你的Web应用全部扔到webapps目录下，Tomcat自动检测，创建相应的Context组件，各个web应用的访问路径就是web应用的目录名

# Tomcat一键启停

* 一个请求在 Tomcat 中流转的过程

  ![image-20200722152746719](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722152746719.png)

  * 如果想让一个系统能够对外提供服务，我们需要创建、组装并启动这些组件；在服务停止的时候，还需要释放资源，销毁这些组件，因此这是一个动态的过程。也就是说，Tomcat 需要动态地管理这些组件的生命周期
  * 组件之间具有两层关系
    * 第一层关系是组件有大有小，大组件管理小组件，比如 Server 管理 Service，Service 又管理连接器和容器
    * 第二层关系是组件有外有内，外层组件控制内层组件，比如连接器是外层组件，负责对外交流，外层组件调用内层组件完成业务功能。也就是说，**请求的处理过程是由外层组件来驱动的**
  * 这两层关系决定了系统在创建组件时应该遵循一定的顺序
    * 第一个原则是先创建子组件，再创建父组件，子组件需要被“注入”到父组件中
    * 第二个原则是先创建内层组件，再创建外层组件，内层组建需要被“注入”到外层组件
  * 因此，最直观的做法就是将图上所有的组件按照先小后大、先内后外的顺序创建出来，然后组装在一起。但是这个思路其实很有问题！因为这样不仅会造成代码逻辑混乱和组件遗漏，而且也不利于后期的功能扩展
  * 需要一种通用的、统一的方法来管理组件的生命周期，就像汽车“一键启动”那样的效果

## 一键式启停：LifeCycle 接口

* 设计就是要找到系统的变化点和不变点。这里的不变点就是每个组件都要经历创建、初始化、启动这几个过程，这些状态以及状态的转化是不变的。而变化点是每个具体组件的初始化方法，也就是启动方法是不一样的

* 因此，把不变点抽象出来成为一个接口，这个接口跟生命周期有关，叫作 LifeCycle。LifeCycle 接口里应该定义这么几个方法：init()、start()、stop() 和 destroy()，每个具体的组件去实现这些方法

* 在父组件的 init() 方法里需要创建子组件并调用子组件的 init() 方法。同样，在父组件的 start() 方法里也需要调用子组件的 start() 方法，因此调用者可以无差别的调用各组件的 init() 方法和 start() 方法，这就是**组合模式**的使用，并且只要调用最顶层组件，也就是 Server 组件的 init() 和 start() 方法，整个 Tomcat 就被启动起来了

  ```java
  public interface LifeCycle {
    void init();
    void start();
    void stop();
    void destroy();
  }
  ```

## 可扩展性：LifeCycle 事件

* 因为各个组件 init() 和 start() 方法的具体实现是复杂多变的，比如在 Host 容器的启动方法里需要扫描 webapps 目录下的 Web 应用，创建相应的 Context 容器，如果将来需要增加新的逻辑，直接修改 start() 方法？这样会违反开闭原则，那如何解决这个问题呢？开闭原则说的是为了扩展系统的功能，不能直接修改系统中已有的类，但是可以定义新的类

* 组件的 init() 和 start() 调用是由它的父组件的状态变化触发的，上层组件的初始化会触发子组件的初始化，上层组件的启动会触发子组件的启动，因此把组件的生命周期定义成一个个状态，把状态的转变看作是一个事件。而事件是有监听器的，在监听器里可以实现一些逻辑，并且监听器也可以方便的添加和删除，这就是典型的**观察者模式**

* 具体来说就是在 LifeCycle 接口里加入两个方法：添加监听器和删除监听器

  * 除此之外，还需要定义一个 Enum 来表示组件有哪些状态，以及处在什么状态会触发什么样的事件。因此 LifeCycle 接口和 LifeCycleState 就定义成了下面这样

    ![image-20200722154452410](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722154452410.png)

  * 组件的生命周期有 NEW、INITIALIZING、INITIALIZED、STARTING_PREP、STARTING、STARTED 等，而一旦组件到达相应的状态就触发相应的事件，比如 NEW 状态表示组件刚刚被实例化；而当 init() 方法被调用时，状态就变成 INITIALIZING 状态，这个时候，就会触发 BEFORE_INIT_EVENT 事件，如果有监听器在监听这个事件，它的方法就会被调用

## 重用性：LifeCycleBase 抽象基类

* 定义一个基类来实现共同的逻辑，然后让各个子类去继承它，就达到了重用的目的

* 基类中定义一些抽象方法留给各个子类去实现的，并且子类必须实现，否则无法实例化

* Tomcat 定义一个基类 LifeCycleBase 来实现 LifeCycle 接口，把一些公共的逻辑放到基类中去，比如生命状态的转变与维护、生命事件的触发以及监听器的添加和删除等，而子类就负责实现自己的初始化、启动和停止等方法。为了避免跟基类中的方法同名，把具体子类的实现方法改个名字，在后面加上 Internal，叫 initInternal()、startInternal() 等

  ![image-20200722155250513](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722155250513.png)

* LifeCycleBase 实现了 LifeCycle 接口中所有的方法，还定义了相应的抽象方法交给具体子类去实现，这是典型的**模板设计模式**

  ```java
  // LifeCycleBase 的 init() 方法实现
  @Override
  public final synchronized void init() throws LifecycleException {
      //1. 状态检查
      if (!state.equals(LifecycleState.NEW)) {
          invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
      }
   
      try {
          //2. 触发 INITIALIZING 事件的监听器
          setStateInternal(LifecycleState.INITIALIZING, null, false);
          
          //3. 调用具体子类的初始化方法
          initInternal();
          
          //4. 触发 INITIALIZED 事件的监听器
          setStateInternal(LifecycleState.INITIALIZED, null, false);
      } catch (Throwable t) {
        ...
      }
  }ss
  ```

* 主要完成了四步

  1. 检查状态的合法性，比如当前状态必须是 NEW 然后才能进行初始化

  2. 触发 INITIALIZING 事件的监听器

     ```java
     // 在这个 setStateInternal 方法里，会调用监听器的业务方法
     setStateInternal(LifecycleState.INITIALIZING, null, false);
     ```

  3. 调用具体子类实现的抽象方法 initInternal() 方法

     * 为了实现一键式启动，具体组件在实现 initInternal() 方法时，又会调用它的子组件的 init() 方法

  4. 子组件初始化后，触发 INITIALIZED 事件的监听器，相应监听器的业务方法就会被调用

     ```java
     setStateInternal(LifecycleState.INITIALIZED, null, false);
     ```

  * 总之，LifeCycleBase 调用了抽象方法来实现骨架逻辑

* 那是什么时候、谁把监听器注册进来的呢？

  * 分为两种情况
    * Tomcat 自定义了一些监听器，这些监听器是父组件在创建子组件的过程中注册到子组件的
      * 比如 MemoryLeakTrackingListener 监听器，用来检测 Context 容器中的内存泄漏，这个监听器是 Host 容器在创建 Context 容器时注册到 Context 中的
    * 还可以在 server.xml 中定义自己的监听器，Tomcat 在启动时会解析 server.xml，创建监听器并注册到容器组件

## 生周期管理总体类图

![image-20200722160311210](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722160311210.png)

* 图中的 StandardServer、StandardService 等是 Server 和 Service 组件的具体实现类，它们都继承了 LifeCycleBase
* StandardEngine、StandardHost、StandardContext 和 StandardWrapper 是相应容器组件的具体实现类，因为它们都是容器，所以继承了 ContainerBase 抽象基类，而 ContainerBase 实现了 Container 接口，也继承了 LifeCycleBase 类，它们的生命周期管理接口和功能接口是分开的，这也符合设计中**接口分离的原则**

## 总结

* Tomcat 为了实现一键式启停以及优雅的生命周期管理，并考虑到了可扩展性和可重用性，将面向对象思想和设计模式发挥到了极致，分别运用了**组合模式、观察者模式、骨架抽象类和模板方法**。
* 如果需要维护一堆具有父子关系的实体，可以考虑使用组合模式
* 观察者模式听起来“高大上”，其实就是当一个事件发生后，需要执行一连串更新操作。传统的实现方式是在事件响应代码里直接加更新逻辑，当更新逻辑加多了之后，代码会变得臃肿，并且这种方式是紧耦合的、侵入式的。而观察者模式实现了低耦合、非侵入式的通知与更新机制
* 而模板方法在抽象基类中经常用到，用来实现通用逻辑

# Tomcat高层

## Tomcat启动

* 通过 Tomcat 的 /bin 目录下的脚本 startup.sh 来启动 Tomcat

  ![image-20200722163253780](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722163253780.png)

  1. Tomcat 本质上是一个 Java 程序，因此 startup.sh 脚本会启动一个 JVM 来运行 Tomcat 的启动类 Bootstrap
  2. Bootstrap 的主要任务是初始化 Tomcat 的类加载器，并且创建 Catalina
  3. Catalina 是一个启动类，它通过解析 server.xml、创建相应的组件，并调用 Server 的 start 方法
  4. Server 组件的职责就是管理 Service 组件，它会负责调用 Service 的 start 方法
  5. Service 组件的职责就是管理连接器和顶层容器 Engine，因此它会调用连接器和 Engine 的 start 方法

* 启动过程中提到的几个非常关键的启动类和组件

  * 可以把 Bootstrap 看作是上帝，它初始化了类加载器，也就是创造万物的工具
  * 把 Tomcat 比作是一家公司，那Catalina 应该是公司创始人，因为 Catalina 负责组建团队，也就是创建 Server 以及它的子组件
  * Server 是公司的 CEO，负责管理多个事业群，每个事业群就是一个 Service
  * Service 是事业群总经理，它管理两个职能部门：一个是对外的市场部，也就是连接器组件；另一个是对内的研发部，也就是容器组件
  * Engine 则是研发部经理，因为 Engine 是最顶层的容器组件
  * 这些启动类或者组件不处理具体请求，它们的任务主要是“管理”，管理下层组件的生命周期，并且给下层组件分配任务，也就是把请求路由到负责“干活儿”的组件。因此可把它们比作 Tomcat 的“高层”

## Catalina

* Catalina 的主要任务就是创建 Server，它不是直接 new 一个 Server 实例就完事了，而是需要解析 server.xml，把在 server.xml 里配置的各种组件一一创建出来，接着调用 Server 组件的 init 方法和 start 方法，这样整个 Tomcat 就启动起来了

* 作为“管理者”，Catalina 还需要处理各种“异常”情况，比如当通过“Ctrl + C”关闭 Tomcat 时，Tomcat 将如何优雅的停止并且清理资源呢？因此 Catalina 在 JVM 中注册一个“关闭钩子”

  ```java
  public void start() {
      //1. 如果持有的 Server 实例为空，就解析 server.xml 创建出来
      if (getServer() == null) {
          load();
      }
      //2. 如果创建失败，报错退出
      if (getServer() == null) {
          log.fatal(sm.getString("catalina.noServer"));
          return;
      }
   
      //3. 启动 Server
      try {
          getServer().start();
      } catch (LifecycleException e) {
          return;
      }
   
      // 创建并注册关闭钩子
      if (useShutdownHook) {
          if (shutdownHook == null) {
              shutdownHook = new CatalinaShutdownHook();
          }
          Runtime.getRuntime().addShutdownHook(shutdownHook);
      }
   
      // 用 await 方法监听停止请求
      if (await) {
          await();
          stop();
      }
  }
  ```

* 如果需要在 JVM 关闭时做一些清理工作，比如将缓存数据刷到磁盘上，或者清理一些临时文件，可以向 JVM 注册一个“关闭钩子”。“关闭钩子”其实就是一个线程，JVM 在停止之前会尝试执行这个线程的 run 方法

  ```java
  // Tomcat 的“关闭钩子”CatalinaShutdownHook 做了些什么
  // Tomcat 的“关闭钩子”实际上就执行了 Server 的 stop 方法，Server 的 stop 方法会释放和清理所有的资源
  protected class CatalinaShutdownHook extends Thread {
      @Override
      public void run() {
          try {
              if (getServer() != null) {
                  Catalina.this.stop();
              }
          } catch (Throwable ex) {
             ...
          }
      }
  }
  ```

## Server 组件

* Server 组件的具体实现类是 StandardServer

* StandardServer 具体实现了哪些功能

  * Server 继承了 LifeCycleBase，它的生命周期被统一管理，并且它的子组件是 Service，因此它还需要管理 Service 的生命周期，也就是说在启动时调用 Service 组件的启动方法，在停止时调用它们的停止方法。Server 在内部维护了若干 Service 组件，它是以数组来保存的，那 Server 是如何添加一个 Service 到数组中的呢？

    ```java
    @Override
    public void addService(Service service) {
        service.setServer(this);
        synchronized (servicesLock) {
            // 创建一个长度 +1 的新数组
            Service results[] = new Service[services.length + 1];
            // 将老的数据复制过去
            System.arraycopy(services, 0, results, 0, services.length);
            results[services.length] = service;
            services = results;
            // 启动 Service 组件
            if (getState().isAvailable()) {
                try {
                    service.start();
                } catch (LifecycleException e) {
                    // Ignore
                }
            }
            // 触发监听事件
            support.firePropertyChange("service", null, service);
        }
    }
    ```

    * 它并没有一开始就分配一个很长的数组，而是在添加的过程中动态地扩展数组长度，当添加一个新的 Service 实例时，会创建一个新数组并把原来数组内容复制到新数组，这样做的目的其实是为了节省内存空间

  * 除此之外，Server 组件还有一个重要的任务是启动一个 Socket 来监听停止端口，这就是为什么能通过 shutdown 命令来关闭 Tomcat。上面 Caralina 的启动方法的最后一行代码就是调用了 Server 的 await 方法

  * 在 await 方法里会创建一个 Socket 监听 8005 端口，并在一个死循环里接收 Socket 上的连接请求，如果有新的连接到来就建立连接，然后从 Socket 中读取数据；如果读到的数据是停止命令“SHUTDOWN”，就退出循环，进入 stop 流程

## Service 组件

* Service 组件的具体实现类是 StandardService

  ```java
  public class StandardService extends LifecycleBase implements Service {
      // 名字
      private String name = null;
      
      //Server 实例
      private Server server = null;
   
      // 连接器数组
      protected Connector connectors[] = new Connector[0];
      private final Object connectorsLock = new Object();
   
      // 对应的 Engine 容器
      private Engine engine = null;
      
      // 映射器及其监听器
      protected final Mapper mapper = new Mapper();
      protected final MapperListener mapperListener = new MapperListener(this);
  ```

* StandardService 继承了 LifecycleBase 抽象类，此外 StandardService 中还有一些熟悉的组件，比如 Server、Connector、Engine 和 Mapper

* 那为什么还有一个 MapperListener？这是因为 Tomcat 支持热部署，当 Web 应用的部署发生变化时，Mapper 中的映射信息也要跟着变化，MapperListener 就是一个监听器，它监听容器的变化，并把信息更新到 Mapper 中，这是典型的观察者模式

* 作为“管理”角色的组件，最重要的是维护其他组件的生命周期。此外在启动各种组件时，要注意它们的依赖关系，也就是说，要注意启动的顺序，Service 启动方法

  ```java
  protected void startInternal() throws LifecycleException {
      //1. 触发启动监听器
      setState(LifecycleState.STARTING);
      //2. 先启动 Engine，Engine 会启动它子容器
      if (engine != null) {
          synchronized (engine) {
              engine.start();
          }
      }
      //3. 再启动 Mapper 监听器
      mapperListener.start();
      //4. 最后启动连接器，连接器会启动它子组件，比如 Endpoint
      synchronized (connectorsLock) {
          for (Connector connector: connectors) {
              if (connector.getState() != LifecycleState.FAILED) {
                  connector.start();
              }
          }
      }
  }
  ```

  * 从启动方法可以看到，Service 先启动了 Engine 组件，再启动 Mapper 监听器，最后才是启动连接器。这很好理解，因为内层组件启动好了才能对外提供服务，才能启动外层的连接器组件。而 Mapper 也依赖容器组件，容器组件启动好了才能监听它们的变化，因此 Mapper 和 MapperListener 在容器组件之后启动。组件停止的顺序跟启动顺序正好相反的，也是基于它们的依赖关系

## Engine 组件

* Engine 本质是一个容器，因此它继承了 ContainerBase 基类，并且实现了 Engine 接口

* Engine 的子容器是 Host，所以它持有了一个 Host 容器的数组，这些功能都被抽象到了 ContainerBase 中，ContainerBase 中有这样一个数据结构

  ```java
  protected final HashMap<String, Container> children = new HashMap<>();
  ```

* ContainerBase 用 HashMap 保存了它的子容器，并且 ContainerBase 还实现了子容器的“增删改查”，甚至连子组件的启动和停止都提供了默认实现，比如 ContainerBase 会用专门的线程池来启动子容器

  ```java
  for (int i = 0; i < children.length; i++) {
     results.add(startStopExecutor.submit(new StartChild(children[i])));
  }
  ```

* 所以 Engine 在启动 Host 子容器时就直接重用了这个方法

* 那 Engine 自己做了什么呢？容器组件最重要的功能是处理请求，而 Engine 容器对请求的“处理”，其实就是把请求转发给某一个 Host 子容器来处理，具体是通过 Valve 来实现的

* 每一个容器组件都有一个 Pipeline，而 Pipeline 中有一个基础阀（Basic Valve），而 Engine 容器的基础阀定义如下

  ```java
  final class StandardEngineValve extends ValveBase {
      public final void invoke(Request request, Response response)
        throws IOException, ServletException {
        // 拿到请求中的 Host 容器
        Host host = request.getHost();
        if (host == null) {
            return;
        }
        // 调用 Host 容器中的 Pipeline 中的第一个 Valve
        host.getPipeline().getFirst().invoke(request, response);
    } 
  }
  ```

* 这个基础阀实现非常简单，就是把请求转发到 Host 容器。你可能好奇，从代码中可以看到，处理请求的 Host 容器对象是从请求中拿到的，请求对象中怎么会有 Host 容器呢？这是因为请求到达 Engine 容器中之前，Mapper 组件已经对请求进行了路由处理，Mapper 组件通过请求的 URL 定位了相应的容器，并且把容器对象保存到了请求对象中

## 总结

* Tomcat 启动过程，具体是由启动类和“高层”组件来完成的，它们都承担着“管理”的角色，负责将子组件创建出来，并把它们拼装在一起，同时也掌握子组件的“生杀大权”
* 在设计这样的组件时，需要考虑两个方面：
  * 首先要选用合适的数据结构来保存子组件，比如 Server 用数组来保存 Service 组件，并且采取动态扩容的方式，这是因为数组结构简单，占用内存小；再比如 ContainerBase 用 HashMap 来保存子容器，虽然 Map 占用内存会多一点，但是可以通过 Map 来快速的查找子容器。因此在实际的工作中，我们也需要根据具体的场景和需求来选用合适的数据结构
  * 其次还需要根据子组件依赖关系来决定它们的启动和停止顺序，以及如何优雅的停止，防止异常情况下的资源泄漏。这正是“管理者”应该考虑的事情
* Server 组件的在启动连接器和容器时，都分别加了锁，这是为什么呢？
* tomcat一般生产环境线程数大小建议怎么设置呢
  * 理论上：线程数=（(线程阻塞时间 + 线程忙绿时间) / 线程忙碌时间) * cpu核数
  * 如果线程始终不阻塞，一直忙碌，会一直占用一个CPU核，因此可以直接设置 线程数=CPU核数
  * 但是现实中线程可能会被阻塞，比如等待IO。因此根据上面的公式确定线程数
  * 那怎么确定线程的忙碌时间和阻塞时间？要经过压测，在代码中埋点统计

