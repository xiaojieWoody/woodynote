# ==Eureka==

* `@EnableDiscoveryClient`

* 负责微服务架构中的服务治理功能
  * 围绕着服务注册与服务发现机制来完成对微服务应用实例的自动化管理
* 服务提供者
  * 服务注册
    * 服务提供者在启动时会带上自身服务的一些元数据信息来发送REST请求将自己注册到EurekaServer
  * 服务同步
    * 服务注册中心互相注册为服务，通过请求转发，服务提供者的服务信息可以通过任意一台注册中心获取到
  * 服务续约
    * 在注册完服务之后，服务提供者会维护一个心跳用来持续告诉EurekaServer自己还可用，以防止Eureka Server 的剔除任务将该服务实例从服务列表中排除出去
* 服务消费者
  * 获取服务
    * 服务消费者启动时会发送一个REST请求给服务注册中心来获取注册的服务清单
      * 为了性能考虑， Eureka Server会维护一份只读的服务清单来返回给客户端，同时该缓存清单会每隔30秒更新一次 
  * 服务调用
    * 服务消费者获取服务清单后，通过服务名可获得具体提供服务的实例名和该实例的元数据信息
      * 因为有这些服务实例的详细信息， 所以客户端可以根据自己的需要决定具体调用哪个实例
      * ==在Ribbon中会默认采用轮询的方式进行调用，从而实现客户端的负载均衡== 
  * 服务下线
    * 在客户端程序中， 当服务实例进行正常的关闭操作时， 它会触发一个服务下线的REST请求给Eureka Server，服务端在接收到请求之后， 将该服务状态置为下线(DOWN)，并把该下线事件传播出去
* 服务注册中心
  * 失效剔除
    * 为将服务列表中无法提供服务的实例剔除，==Eureka Server 在启动的时候会创建一个定时任务，默认每隔一段时间(默认为60秒) 将当前清单中超时(默认为90秒)没有续约的服务剔除出去==
  * 自我保护
    * 服务注册到EurekaSrevre 之后，会维护一个心跳连接，告诉EurekaServer自己还可用
    * ==Eurkea Server在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%，EurekaServer会将当前的实例注册信息保护起来， 让这些实例不会过期， 尽可能保护这些注册信息==
* 高可用注册中心
  * 实际上就是将自己作为服务向其他服务注册中心注册自己，形成一组互相注册的服务注册中心，以实现服务清单的互相同步，达到高可用的效果
* 在默认设置下， 该服务注册中心也会将自己作为客户端来尝试注册它自己
* 默认情况下，`Eureka`中各个服务实例的健康检测并不是通过`spring-boot-actuator`模块的`/health` 端点来实现的， 而是依靠客户端心跳的方式来保持服务实例的存活
* 默认的心跳实现方式可有效检查客户端进程是否正常运作， 但却无法保证客户端应用能够正常提供服务
  * ==当微服务应用与外部资源（如数据库、缓存、消息代理等）无法联通时，无法提供正常的对外服务，但是心跳依然在运行，所以它还是会被服务消费者调用，但是实际不能获得预期结果==
* 通过配置， ==把`Eureka`客户端的健康检测交给`spring-boot-actuator`模块的`/health`端点， 以实现更加全面的健康状态维护==

# Ribbon

* 是一个基于`HTTP`和`TCP`的客户端负载均衡工具
  * 通过客户端中配置的`ribbonServerList`服务端列表去轮询访问以达到均衡负载的作用
  * ==通过注册中心中获取服务端列表去轮询访问以达到均衡负载的作用==
* 通过`SpringCloud`的封装，可轻松将面向服务的REST模板请求自动转换成客户端负载均衡的服务调用，不需要独立部署，几乎存在于每一个`Spring Cloud`构建的微服务和基础设施中
* 使用
  * 服务提供者
    * 只需启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心
  * 服务消费者
    * ==直接通过调用被`@LoadBalanced`注解修饰过的`RestTemplate`来实现面向服务的接口调用==
* 直接通过调用被`@LoadBalanced`注解修饰过的`RestTemplate`来实现面向服务的接口调用
* RestTemplate
  * 该对象会使用`Ribbon`的自动化配置，同时通过配置`@LoadBalanced` 还能够开启客户端负载均衡
* 与Eureka结合
  * 当在`Spring Cloud`的应用中同时引入`Spring Cloud Ribbon`和`Spring Cloud Eureka`依赖时，会触发`Eureka对Ribbon`的自动化配置
    * 将服务清单列表交给`Eureka`的服务治理机制来维护 ，而不是`Ribbon`的`ServerList`的维护机制来维护
    * 实例检查的任务（`IPing`的实现）交给了服务治理框架来进行维护
* ==负载均衡策略==
  * `Random Rule`：从服务实例清单中随机选择一个服务实例的功能
  * `RoundRobinRule`：实现了按照线性轮询的方式依次选择每个服务实例的功能
  * `RetryRule`：实现了一个具备重试机制的实例选择功能
  * `Weighted ResponseTimeRule`：根据实例的运行情况来计算权重， 并根据权重来挑选实例
* 重试机制
  * 开启重试机制，默认关闭
  * ==断路器的超时时间需要大于Ribbon的超时时间， 不然不会触发重试==
  * ==切换实例的重试次数==
  * ==对当前实例的重试次数==

# ==Hystrix==

* `@EnableCircuitBreaker`注解开启断路器功能
  
* @SpringCloudApplication 包含了`@SpringBootApplication`、`@EnableDiscoveryClient`、`@EnableCircuitBreaker`
  
* ==故障监控、容错，防止故障蔓延==

  * 当某个服务单元发生故障之后，通过断路器的故障监控，向调用方返回一个错误响应，而不是长时间等待，可防止故障蔓延

* ==Hystrix具备服务降级、 服务熔断、 线程和信号隔离、 请求缓存、 请求合并以及服务监控等强大功能==

* Hystrix则使用舱壁模式实现线程池的隔离，它会为每一个依赖服务创建一个独立的线程池， 这样就算某个依赖服务出现延迟过高的情况， 也只是对该依赖服务的调用产生影响， 而不会拖慢其他的依赖服务

* 可以使用信号量来控制单个依赖服务的并发度， 信号量的开销远比线程池的开销小， 但是它不能设置超时和实现异步访问。所以，只有在依赖服务是足够可靠的情况下才使用信号量

* 在 服务方法上增加`@HystrixCommand`注解来fallbackMethod属性指定回调方法

  * fallback处理，服务降级情况
    * 当前命令处于 “ 熔断I短路 ” 状态， 断路器是打开的时候
      * Hystrix会将“成功”、 “失败”、 “拒绝”、 “ 超时” 等信息报告给断路器，而断路器会维护一组计数器来统计这些数据
      * 断路器会使用这些统计数据来决定是否要将断路器打开， 来对某个依赖服务的请求进行 “熔断/短路”，直到恢复期结束。若在恢复期结束后，根据统计数据判断如果还是未达到健康指标，就再次 “熔断/短路” 
    * 当前命令的线程池、 请求队列或者信号量被占满的时候
    * run()抛出异常的时候 

  * 注解方式
    * 只需要使用@HystrixCommand 中的 fallbackMethod参数来指定具体的服务降级实现方法
    * 需要将具体的 Hystrix 命令与 fallback 实现函数定义在同一个类中， 并且 fallbackMethod 的值必须与实现 fallback 方法的名字相同。

  

# ==Feign==

* 整合了Spring Cloud Ribbon 与 Spring Cloud Hystrix，除了提供这两者的强大功能之外， 它还提
  供了一种声明式的Web服务客户端定义方式

* 只需创建一个接口并用注解的方式来配置它，即可完成对服务提供方的接口绑定

* 主类：==@EnableFeignClients 注解开启 Spring Cloud Feign 的支待功能==、@EnableDiscoveryClient

* ==定义 HelloService 接口， 通过@FeignClient 注解指定服务名来绑定服务， 然后再使用 Spring MVC 的注解来绑定具体该服务提供的 REST 接口==

  ```java
  // 指定服务名，不区分大小写
  @FeignClient("hello-service") 
  public interface HelloService {
    //Spring MVC的注解来绑定具体该服务提供的REST接口
  	@RequestMapping("/hello") 
    String hello();
    
    //参数绑定
    @RequestMapping(value= "/hellol", method= RequestMethod.GET) 
    String hello(@RequestParam("name") String name);
    
    @RequestMapping(value = "/hello3", method= RequestMethod.POST) 
    String hello(@RequestBody User user);
  }
  ```

* 接着， 创建一个 ConsumerController 来实现对 Feign 客户端的调用
  
* 使用@Autowired 直接注入上面定义的 HelloService 实例， 并在 helloConsumer函数中调用这个绑定了 hello-service 服务接口的客户端来向该服务发起/hello接口的调用
  
* 服务降级
  * 只需为 Feign 客户端的定义接口编写一个具体的接口实现类，@Component
  * 在服务绑定接口 HelloService 中， 通过@FeignClient 注解的 fallback 属性来指定对应的服务降级实现类

## SpringCloud 分模块

* client

  * @FeignClient提供给外部调用，包含common模块

  * 其他服务添加该服务的client模块，启动类上`@EnableFeignClients(basePackages = "com.imooc.product.client其他模块的服务提供类")`，使用时直接@Autowired注入即可

    ```java
    // 该模块就一个接口，没有主类
    @FeignClient(name = "product", fallback = ProductClient.ProductClientFallback.class)
    public interface ProductClient {
        @PostMapping("/product/listForOrder")
        List<ProductInfoOutput> listForOrder(@RequestBody List<String> productIdList);
      
        @PostMapping("/product/decreaseStock")
        void decreaseStock(@RequestBody List<DecreaseStockInput> decreaseStockInputList);
    
        @Component
        static class ProductClientFallback implements ProductClient {
            @Override
            public List<ProductInfoOutput> listForOrder(List<String> productIdList) {
                return null;
            }
            @Override
            public void decreaseStock(List<DecreaseStockInput> decreaseStockInputList) {
            }
        }
    }
    ```

* server

  * 业务层，controller、service、dao
  * 包括common模块

* common

  * 定义对外的输入输出实体

# Config

* 用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持， 分为服务端与客户端两个部分
* 服务端称为分布式配置中心
  * 它是一个独立的微服务应用， 用来连接配置仓库并为客户端提供获取配置信息、 加密/解密信息等访问接口
* 客户端则是微服务架构中的各个微服务应用或基础设施
  * 通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息

# Bus

* 总线上的各个实例都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息
* 消息总线中的常用功能，比如配合`Spring Cloud Config` 实现微服务应用配置信息的动态更新等
* RabbitMQ消息总线
  * 在`Spring Cloud Bus`中包含了对`Rabbit`的自动化默认配置
  * 通过@RabbitListener注解定义该类对队列的监听、用@RabbitHandler注解来指定对消息的处理方法
* AmqpTemplate接口定义了一套针对AMQP协议的基础操作。 在SpringBoot中会根据配置来注入其具体实现

#==Zuul== 

* 作为系统的统一入口，过滤和调度所有外部客户端的访问，实现请求路由、负载均衡、校验过滤等功能
* `Spring Cloud Zuul`通过与`Spring Cloud Eureka`进行整合， 将自身注册为`Eureka`服务治理下的应用， 同时从`Eureka`中获得了所有其他微服务实例的信息
* 对于路由规则的维护， `Zuul`默认会将通过以服务名作为`ContextPath`的方式来创建路由映射
* 可以通过使用`Zuul`来创建各种校验过滤器，然后指定哪些规则的请求需要执行校验逻辑，只有通过校验的才会被路由到具体的微服务接口，不然就返回错误提示
* ==面向服务的路由==
  * 可让路由的`path`不是映射具体的`url`，而是让它映射到某个具体的服务，而具体的`url`则交给`Eureka`的服务发现机制去自动维护
  * 将自己注册到Eureka服务注册中心上，从注册中心获取所有服务以及它们的实例清单
  * 当外部请求到达`API`网关时，根据请求的`URL`路径找到最佳匹配的`path`规则， API 网关就可以知道要将该请求路由到哪个具体的`serviceId`上去
  * 由于在`API`网关中已经知道`serviceid`对应服务实例的地址清单，那么只需要通过`Ribbon`的负载均衡策略， 直接在这些清单中选择一个具体的实例进行转发就能完成路由工作
* 请求过滤
  * 在实现了请求路由功能之后，微服务应用提供的接口就可以通过统一的 API网关入口被客户端访问到
  * 对客户端请求的安全校验和权限控制，通过前置的网关服务来完成这些非业务性质的校验

    * ==在请求到达的时候就完成校验和过滤， 而不是转发后再过滤而导致更长的请求延迟==
    * ==同时，通过在网关中完成校验和过滤，微服务应用端就可以去除各种复杂的过滤器和拦截器==
  * Zuul 允许开发者在API网关上通过定义过滤器来实现对请求的拦截与过滤

    * 只需要==继承 ZuulFiter 抽象类并实现它定义的4个抽象函数就可以完成对请求的拦截和过滤==
      * 过滤器类型、执行顺序、是否需要被执行、具体逻辑
  * 4个基本特征：过滤类型、执行顺序、执行条件、具体操作 

    * `FilterType`：该函数需要返回一个字符串来代表过滤器的类型，而这个类型就是在`HTTP`请求过程中定义的各个阶段。在 `Zuul` 中默认定义了==4 种不同生命周期的过滤器类型==
      * `pre`：可以在请求被路由之前调用
      * `routing`：在路由请求时被调用。
      * `post`：在 `routing` 和 `error` 过滤器之后被调用
      * `error`：处理请求时发生错误时被调用
    * `filterOrder`：通过 int 值来定义==过滤器的执行顺序， 数值越小优先级越高==
    * `shouldFilter`：返回一个`boolean`值来==判断该过滤器是否要执行==，可通过此方法来指定过滤器的有效围 
    * `run`：过滤器的==具体逻辑==
      *  在该函数中可实现自定义的过滤逻辑，来确定是否要拦截当前的请求，不对其进行后续的路由，或是在请求路由返回结果之后，对处理结果做一些加工等
  * 4种过滤器
    * 当外部 HTTP 请求到达 API 网关服务时
    * 首先会进入第一个阶段pre，被pre类型的过滤器进行处理，==在进行请求路由之前做一些前置加工， 比如请求的校验等==
    * 第二个阶段routing路由请求转发阶段，请求将会被`routing`类型过滤器处理
      * 这里的具体处理内容就是==将外部请求转发到具体服务实例上去的过程，当服务实例将请求结果都返回之后，`routing` 阶段完成== 
    * 第三个阶段`post`，此时请求将会被post类型的过滤器处理，此时==不仅可以获取到请求信息，还能获取到服务实例的返回信息， 所以可以对处理结果进行一些加工或转换等内容== 
     * 特殊的阶段`error`，只有在上述==三个阶段中发生异常的时候才会触发，但是它的最后流向还是`post`型的过滤器，因为它需要通过`post`过滤器将最终结果返回给请求客户端==