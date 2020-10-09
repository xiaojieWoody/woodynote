# SpringBoot starter解析

## conditional注解解析

* 含义：基于条件的注解
* 作用：根据是否满足某一个特定条件来决定是否创建某个特定的Bean
* 意义：SpringBoot实现自动配置的关键基础能力

```java
// 常见conditional注解
@ConditionalOnBean
@ConditionalOnMissingBean
@ConditionalOnClass
@ConditionalOnMissingClass
@ConditionalOnWebApplication
@ConditionalOnProperty
@ConditionalOnNotWebApplication
@ConditionalOnJava
```

```java
@Component
@ConditionalOnProperty("com.mooc.condition")
public class A {
}
// application.properties中有 com.mooc.condition 时才会注册A Bean
// applicationContext.getBean(A.class);
```

* 自定义condition注解

```java
//1. 实现一个自定义注解并且引入Conditional注解
//2. 实现Condition接口重写matches方法，符合条件返回true
//3. 自定义注解引入Condition接口实现类
```

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(MyCondition.class)
public @interface MyConditionAnnotation {
    String[] value() default {};
}
```

```java
public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String[] properties = (String[])metadata.getAnnotationAttributes("com.mooc.sb2.condi.MyConditionAnnotation").get("value");
        for (String property : properties) {
            if (StringUtils.isEmpty(context.getEnvironment().getProperty(property))) {
                return false;
            }
        }
        return true;
    }
}
```

```java
// application.properties中有 com.mooc.condition1、com.mooc.condition2 时才会注册A Bean
@Component
@MyConditionAnnotation({"com.mooc.condition1", "com.mooc.condition2"})
public class A {
}
```

## 动手搭建starter

* 简介：可插拔插件
* 与jar包区别：starter能实现自动装配
* 作用：大幅度提升开发效率

```shell
# 新建starter步骤
新建SpringBoot项目
引入spring-boot-autoconfigure
编写属性源及自动配置类
在spring.factories中添加自动配置类实现
maven打包

# 使用starter步骤
pom.xml中引入starter
属性文件中配置属性
类中引入服务
调用服务能力
```

```xml
// 新建项目
// pom.xml
<artifactId>weather-spring-boot-starter</artifactId>
  
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  </dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```

```java
@Data
@ConfigurationProperties(prefix = "weather")
public class WeatherSource {
  private String type;
  private String rate;
}

public class WeatherService {
  private WeatherSource weatherSource;
  public WeatherService(WeatherSource weatherSource) {
    this.weatherSource = weatherSource;
  }
  public String getType() {
    return weatherSource.getType();
  }
  public String getRate() {
    return weatherSource.getRate();
  }
}
```

```java
@Configuration
@EnableConfigurationProperties(WeatherSource.class)
@ConditionalOnProperty(name = "weather.enable", havingValue = "enable")
public class WeatherAutoConfiguration {
  @Autowired
  private WeatherSource weatherSource;
  
  @Bean
  @ConditionalOnMissingBean(WeatherService.class)
  public WeatherService weatherService() {
    return new WeatherService(weatherSource);
  }
}
```

```properties
# resources/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.weather.WeatherAutoConfiguration
```

* 引用

```xml
其他项目引用weather starter
<dependency>
	<groupId>com.example</groupId>
  <artifactId>weather-spring-boot-starter</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</dependency>
```

```java
// application.properties
weather.type=rain
weather.rate=serious
weather.enable=enable    // disable 时不能自动注入

@Autowired
private WeatherService weatherService;

// weatherService.getType() + weatherService.getRate();
```

## starter原理解析

```java
启动类上@SpringBootApplication
引入AutoConfigurationImportSelector
ConfigurationClassParser中处理
获取spring.factories中EnableAutoConfiguration实现
  
// starter自动配置类过滤
@ConditionalOnProperty
OnPropertyCondition
getMatchOutcome
遍历注解属性集判断environment中是否含有并值一致
返回对比结果

// 源码入口
getCandidateConfigurations
```

## 总结

* 请介绍下你熟悉的conditional注解？
* 回答下conditional注解的原理？
* SpringBoot starter有什么用？熟悉哪些？
* 有没有尝试过手动搭建starter
* starter中的自动配置类是如何被引入到框架中
* starter中自动配置类生效原理

# mybatis starter解析

* mybatis-starter作用
  * 自动检测工程中的DataSource
  * 创建并注册SqlSessionFactory实例
  * 创建并注册SqlSessionTemplate实例
  * 自动扫描mappers

## mybatis-starter使用指南

```java
// mybatis starter引入步骤
引入mybatis-starter、mysql两者jar包
配置数据库连接属性
引入mybatis逆向工程插件及文件
配置mybatis工程属性
注解MapperScan或Mapper以扫描接口类  
  
// mybatis starter测试用例
demoMapper#insert/updateByPrimaryKey/updateByExampleSelective/selectByExample/deleteByPrimaryKeys  
```

```xml
<!-- pom.xml -->
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>2.1.0</version>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.39</version>
</dependency>

<plugin>
  <groupId>org.mybatis.generator</groupId>
  <artifactId>mybatis-generator-maven-plugin</artifactId>
  <version>1.3.5</version>
  <dependencies>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.38</version>
    </dependency>
  </dependencies>
</plugin>
```

```properties
# resources/application.properties
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.mooc.sb2.bean
mybatis.configuration.map-underscore-to-camel-case=true         # 驼峰命名
```

```xml
<!--resources/generatorConfig.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test" userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
            NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!-- targetProject:生成PO类的位置 -->
        <javaModelGenerator targetPackage="com.mooc.sb2.bean"
                            targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false"/>
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>
        <!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.mooc.sb2.mapper"
                             targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>
        <!-- 指定数据库表 -->
        <table schema="" tableName="demo"></table>

    </context>
</generatorConfiguration>
```

```java
@SpringBootApplication
@MapperScan("com.mooc.sb2.mapper")    // 或者不添加这个注解，而是在每个Mapper类上添加@Mapper注解
public class ...
```

```java
@Autowired
private DemoMapper demoMapper;

Demo demo = new Demo();
demo.setName("xx");
demo.setJob("student");
System.out.println(demoMapper.insert(demo));
```

## mybatis-starter原理解析

```java
// mybatis-spring-boot-autoconfigure.jar/META-INF/spring.factories
MybatisLanguageDriverAutoConfiguration
MybatisAutoConfiguration

// debug
MybatisAutoConfiguration#registerBeanDefinitions#registerBeanDefinition  
```

```java
// 自动配置类导入
mybatis-spring-boot-starter jar包
mybatis-spring-boot-autoconfigure jar包
META-INF/spring.factories文件
MybatisAutoConfiguration
// 关键类注入
SqlsessionFactory
单个数据库映射关系经编译后的内存镜像
SqlsessionTemplate
// mapper类扫描
MapperScannerRegistrarNotFountConfiguration
AutoConfiguredMapperScannerRegistrar|Mapperscan
MapperScannerConfigure
扫描mapper接口注册到容器中
// mapper类生成
processBeanDefinitions
beanclass替换成MapperFactoryBean.class
MapperFactoryBean#getObject
MapperProxy对象
// mapper类执行
MapperProxy#invoke
MapperMethod#execute
根据数据库操作类型，调用sqlsession操作
返回执行结果  
```

## 配置类引入原理

## 注解扫描原理

## mapper类生成原理

## mapper类执行原理

## 缓存使用指南

![image-20201008103149297](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008103149297.png)

* redis
  * 完全开源免费
  * 支持数据的持久化
  * 支持多种数据结构
  * 支持数据备份

```java
// redis starter使用
spring-boot-starter-data-redis jar包
application.properties文件定义redis相关属性
RedisConfiguration注入自定义RedisTemplate
编写RedisUtil使用RedisTemplate进行增删改查  
```

```xml
<!--pom.xml-->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.73</version>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```properties
# application.properties
spring.redis.port=6379
spring.redis.host=127.0.0.1
spring.redis.database=0
```

```java
@Configuration
public class RedisConfiguration {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory factory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(factory);

        return redisTemplate;
    }
}
```

```java
@Component
public class RedisUtil {

    @Autowired
    private RedisTemplate redisTemplate;

    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    public boolean set(String key, Object value, long time) {
        try{
            redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            return true;
        }catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }
}
```

## 章节总结

* 讲解一下mybatis-starter使用流程
* mybatis更新有哪几种方式
* mapper接口是如何被扫描注册到容器中的
* 简述下mybatis-starter的原理
* springboot如何和redis结合
* 缓存穿透、雪崩是什么意思？如何防止

# webflux解析

## webflux介绍

* 同步阻塞式IO模型
* 异步非阻塞式IO模型

![image-20201008155359619](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008155359619.png)

![image-20201008155432118](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008155432118.png)

* Netty优点
  * API使用简单、易上手
  * 功能强大、支持多种主流协议
  * 定制能力强、可扩展性高
  * 性能高、综合性能最优
  * 成熟稳定、久经烤验
  * 社区活跃、学习资料多

![image-20201008155741203](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008155741203.png)

* webflux使用建议
  * 如果当前项目运行良好，没必要切换，如果要切换，最好切换整套技术栈
  * 如果只是个人对新技术感兴趣，可以在一些简单小型项目中使用研究，或者使用WebClient尝试
  * 大团队慎重考虑引进这门技术，引入前跟团队成员一起做好评估工作

![image-20201008160157167](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008160157167.png)

* webflux技术依赖

  * Reactive Streams：反应式编程标准和规范

    * 一套基于jvm面向流式类库的标准和规范

      * 具有处理无限数量数据的能力
      * 按序处理数据
      * 异步非阻塞的传递数据
      * 必须实现非阻塞的背压

    * api规范组件

      * publisher：数据发布者

        ```java
        // publisher接口规范
        public interface Publisher<T> {
          public void subscribe(Subscriber<? super T> s);
        }
        ```

      * subscriber：数据订阅者

        ```java
        // subscriber接口规范
        public interface Subscriber<T> {
          public void onSubscribe(Subscription s);
          public void onNext(T t);
          public void onError(Throwable t);
          public void onComplete();
        }
        ```

      * subscription：订阅信号

        ```java
        // subscription接口规范
        public interface Subscription {
          public void request(long n);
          public void cancel();
        }
        ```

      * processor：处理器（包含了发布者和订阅者的混合体）

        ```java
        // processor接口规范
        public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {  
        }
        ```

  * Reactor：基于Reactive Streams的反应式编程框架

  * WebFlux：以Reactor为基础实现Web领域的反应式编程框架

## Reactor指南

* Reactor框架是Spring公司开发的
* 符合Reactive Streams规范
* 侧重于server端的响应式编程框架
* 两个模块组成reactor-core和reactor-ipc
* Java原有异步编程方式
  * Callbacks：异步方法采用一个callback作为参数，当结果出来后回调这个callback，例如swings的EventListener
    * Callbacks局限性：Callback Hell，如果有多个方法需要回调，则会充满密密麻麻的回调对象，阅读性和可维护性差
  * Futures：异步方法返回一个`Future<T>`，此时结果并不是立刻可以拿到，需处理结束之后才可以用
    * Futures局限性：多个Future组合不易、调用Future#get时仍然会阻塞、缺乏对多个值以及进一步的出错处理
* Reactor的publisher
  * 实现：Flux、Mono
  * Flux：代表一个包含0...N个元素的响应式序列
  * Mono：代表一个包含0/1个元素的结果

![image-20201008164043906](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008164043906.png)

![image-20201008164110893](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008164110893.png)

![image-20201008164137503](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008164137503.png)

## webflux实践

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
Mono<Integer> mono = Mono.just(1);
Integer[] integers = {1,2,3,4,5,6};
Flux<Integer> arrayFlux = Flux.fromArray(integers);
List<Integer> list = Arrays.asList(1,2,3,4,5,6);
Flux<Integer> listFlux = Flux.fromIterable(list);
Flux<Integer> fluxFlux = Flux.from(flux);
Flux<Integer> streamFlux = Flux.fromStream(Stream.of(1, 2, 3, 4, 5, 6));
flux.subscribe();
// arrayFlux.subscribe(System.out::println);
// listFlux.subscribe(System.out::println, System.err::println);
// fluxFlux.subscribe(System.out::println, System.err::println, () -> System.out.println("complete"));
// streamFlux.subscribe(System.out::println, System.err::println, () -> System.out.println("complete"), subscription -> subscription.request(3));
streamFlux.subscribe(new DemoSubscriber());
```

```java
class DemoSubscriber extends BaseSubscriber<Integer> {
  @Override
  protected void hookOnSubscribe(Subscription subscription) {
    System.out.println("subscribe");
    subscription.request(1);
  }

  @Override
  protected void hookOnNext(Integer value) {
    if(value == 4) {
      cancel();
    }
    System.out.println(value);
    request(1);
  }
}
```

* reactor操作符

  * map
  * flatMap
  * filter
  * zip

  ```java
  Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
  Mono<Integer> mono = Mono.just(1);
  Integer[] integers = {1,2,3,4,5,6};
  Flux<Integer> arrayFlux = Flux.fromArray(integers);
  List<Integer> list = Arrays.asList(1,2,3,4,5,6);
  Flux<Integer> listFlux = Flux.fromIterable(list);
  Flux<Integer> fluxFlux = Flux.from(flux);
  Flux<Integer> streamFlux = Flux.fromStream(Stream.of(1, 2, 3, 4, 5, 6));
  
  flux.map(i -> i * 3).subscribe(System.out::println);
  arrayFlux.flatMap(i -> flux).subscribe(System.out::println);
  listFlux.filter(i -> i > 3).subscribe(System.out::println); 
  Flux.zip(fluxFlux, streamFlux).subscribe(zip -> System.out.println(zip.getT1() + zip.getT2()));
  ```

* reactor和java8 stream区别

  * 形似神不似
  * reactor:push模式，服务端推送数据给客户端
  * stream:pull模式，客户端主动向服务端请求数据

* reactor创建线程方式

  ```java
  Schedulers.immediate();当前线程
  Scheudlers.single();可重用的单线程
  Schedulers.elastic();弹性线程池
  Schedulers.parallel();固定大小线程池
  Schedulers.fromExecutorService();自定义线程池  
  ```

  ![image-20201008173803724](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008173803724.png)

  ```java
  Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
  flux.map(i -> {
    System.out.println(Thread.currentThread().getName() + "-map1");
    return i * 3;
  }).publishOn(Schedulers.elastic()).map(
    i -> {
      System.out.println(Thread.currentThread().getName() + "-map2");
      return i / 3;
    }
  ).subscribeOn(Schedulers.parallel())
    .subscribe(it -> System.out.println(Thread.currentThread().getName() + "-map3"));
  while (true) {}
  ```

* 线程切换总结
  * publishOn：它将上游信号传给下游，同时改变后续的操作符的执行所在线程，直到下一个publishOn出现在这个链上
  * subscribeOn：作用于向上的订阅链，无论处于操作链的什么位置，它都会影响到源头的线程执行环境，但不会影响到后续的publishOn
* 实践内容
  * 兼容SpringMVC写法
  * Spring webflux函数式写法
  * 连接关系型数据库案例
  * 连接非关系型数据库案例

```java
// web-flux和web的依赖同时出现时，web-flux无效果

@Configuration
public class RouterConfig {

    @Autowired
    private DemoHandler demoHandler;

    @Bean
    public RouterFunction<ServerResponse> demoRouter() {
        return route(GET("/hello"), demoHandler::hello)
                .andRoute(GET("/world"), demoHandler::world)
                .andRoute(GET("/times"), demoHandler::times);
    }
}

@Component
public class DemoHandler {

    @Autowired
    private DemoQueryService demoQueryService;

    public Mono<ServerResponse> hello(ServerRequest request) {
        return ok().contentType(MediaType.TEXT_PLAIN).body(Mono.just("hello"), String.class);
    }

    public Mono<ServerResponse> world(ServerRequest request) {
        return ok().contentType(MediaType.TEXT_PLAIN).body(Mono.just("world"), String.class);
    }

    public Mono<ServerResponse> times(ServerRequest request) {
        return ok().contentType(MediaType.TEXT_EVENT_STREAM)
                .body(Flux.interval(Duration.ofSeconds(1))
                .map(it -> new SimpleDateFormat("HH:mm:ss").format(new Date())), String.class);
    }

    public Mono<ServerResponse> queryDemo(ServerRequest request) {
        Integer id = Integer.valueOf(request.pathVariable("id"));
        return ok().contentType(MediaType.APPLICATION_JSON)
                .body(Mono.just(demoQueryService.queryDemoById(id)). Demo.class);
    }
}
```

* mongodb

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
  
spring.data.mongodb.username=test
spring.data.mongodb.password=123456
spring.data.mongodb.host=localhost
spring.data.mongodb.database=test
spring.data.mongodb.port=27017  
  
@Data  
public class City {
  	private String id;
    private String province;
    private String city;
}

@Repository
public interface CityRepository extends ReactiveMongoRepository<City, String> {
    
}

@Component
public class DemoHandler {
    @Autowired
    private CityRepository cityRepository;

    public Mono<ServerResponse> listCity(ServerRequest request) {
        return ok().contentType(MediaType.APPLICATION_JSON)
                .body(cityRepository.findAll(), City.class);
    }

    public Mono<ServerResponse> saveCity(ServerRequest request) {
        String province = request.pathVariable("province");
        String city = request.pathVariable("city");
        City record = new City();
        record.setProvince(province);
        record.setCity(city);
        Mono<City> mono = Mono.just(record);
        return ok().build(cityRepository.insert(mono).then());
    }
}

@Configuration
public class RouterConfig {

    @Autowired
    private DemoHandler demoHandler;

    @Bean
    public RouterFunction<ServerResponse> demoRouter() {
        return route(GET("/hello"), demoHandler::hello)
                .andRoute(GET("/world"), demoHandler::world)
                .andRoute(GET("/times"), demoHandler::times)
                .andRoute(GET("/listCity"), demoHandler::listCity)
                .andRoute(GET("/saveCity/{province}/{city}"), demoHandler::saveCity);
    }
}
```

* 兼容SpringMVC写法
  * 使用SpringMVC注解
  * ServletReq/Resp切换成ServerReq/Resp
  * 返回Mono对象
* Spring webflux函数式写法
  * 定义DemoHandler
  * 写ServerRequest消费方法返回`Mono<ServerResponse>`
  * 定义RouterConfig
  * 书写路由映射
* 连接关系型数据库案例
  * 使用Mono封装mapper返回对象
* 连接非关系型数据库案例
  * spring-boot-starter-data-mongo-reactive
  * application.properties文件定义mongo相关属性
  * 定义mongodb集合对应对象
  * 继承ReactiveMongoRepository

## webflux解析

![image-20201008193911701](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008193911701.png)

```java
// DispatcherHandler准备
setApplicationContextAware
initStrategies
获取容器中HandlerMapping及子接口实现
获取容器中HandlerAdapter及子接口实现
获取容器中HandlerResultHandler及子接口实现
  
// RouterFunctionMapping实例化
afterPropertiesSet
initRouterFunctions
routerFunctions获得系统中所有RouterFunction
通过RouterFunction::andOther将对象合并
返回SameComposedRouterFunction对象
  
// DispatcherHanddler#handle
构建基于handlerMappings集合的Flux对象
通过concatMap将其转换成handler对象
取出第一个handler对象，若为空，则抛错
调用invokeHandler获得response
调用handleResult对结果进行处理
  
// HandlerMapping#getHandler
调用子类getHandlerInternal实现
获得Handler对象
跨域处理 
返回Mono<Object>对象
  
// DispatcherHandler#invokeHandler
遍历handlerAdapters集合
依次调用集合元素supports方法
获得具体实现类调用handle方法
进入具体url对应处理类处理请求
返回Mono<HandlerResult>对象
  
// DispatcherHandler#handleResult
遍历resultHandlers集合
依次调用集合元素supports方法
获得具体实现类调用handleResult方法
将请求结果信息写入ServerWebExchange对象  
```

## 章节总结

* webflux出现的意义
* webflux与SpringMVC异同？
* 介绍下reactive streams规范
* 介绍下reactor
* flux和mono对象的区别？如何创建
* 简述下reactor操作符&线程池
* publishOn和subscribeOn区别
* reactor和java 8 stream区别
* 说一下webflux搭建过程，有没有和数据库结合过使用
* RouterFunctionMapping的作用以及何时被加载
* DispatherHandler初始化过程做了哪些事
* 介绍下DispatherHandler的handle方法

# 日志系统解析

## 日志介绍

* 日志作用：记录程序的运行轨迹，方便查找关键信息以及快速定位解决问题
* 日志实现框架
  * 具体的日志功能实现
  * JUL、Log4j、Logback、Log4j2
* 日志门面框架
  * 日志实现的抽象层
  * JCL、SLF4J
* 日志发展历程
  * JDK1.3及以前，通过System.(out|err).println打印，存在巨大缺陷
  * 解决系统打印缺陷问题出现log4j，2015年8月停止更新
  * 受到log4j影响，SUN公司推出java.util.logging即JUL
  * 由于存在两个系统实现，解决兼容性问题，推出commons-logging即JCL，但存在一定的缺陷
  * log4j作者推出slf4j，功能完善兼容性好，成为业界主流
  * log4j作者在推出log4j后进行新的改进思考推出logback
  * log4j2对log4j的重大升级，修复已知缺陷，极大提升性能
  * 最佳组合：slf4j + logback（SpringBoot使用）slf4j + log4j2

```java
// 入口
 private Logger logger = LoggerFactory.getLogger(RouterConfig.class);

// 日志实现寻址
LoggerFactory.getLogger
findPossibleStaticLoggerBinderPathSet
获取StaticLoggerBinder所在jar包路径
若存在多个日志实现框架打印提示及选择
使用StaticLoggerBinder获得日志工厂再得实现  
```

## 日志配置

```java
// SpringBoot日志
spring-boot-starter-logging
logback-classtic
log4j-to-slf4j
jul-to-slf4j  
```

![image-20201008205119563](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201008205119563.png)

```java
// 日志使用
Logger logger = LoggerFactory.getLogger(xx.class);
logger.level()
ERROR -> WARN -> INFO -> DEBUG -> TRACE

logger.debug("xyz {} is {}", i, j) 
```

```xml
<!---->
<configuration scan="true" scanPeriod="60 seconds" debug="false" />
scan：当设置为true时，配置文件若发生改变，将会重新加载
scanPeriod：扫描时间间隔，若没给出时间单位，默认为毫秒
debug：若设置为true，将打印出logback内部日志信息

<!--configuration子节点-->
contextName：上下文名称
property：属性配置
appender：格式化日志输出
root：全局日志输出设置
logger：具体包或子类输出设置

<!--configuration上下文及属性配置-->
<contextName>demo</contextName>
contextName:用来区分不同应用程序的记录，默认为default
<property name="LOG_PATH" value="logs" />
name：变量名称 value：变量值
<property resource="application.properties" />
<!--若application.properties中没有定义logging.path的值，则使用用户所在目录作为日志目录-->
<property name="LOG_PATH" value="${logging.path}:-${user.home}/${spring.application.name}/logs" />

<!--appender配置-->
```

```xml
<!--常用pattern介绍-->
logger {length}：输出日志的logger名，可有一个整形参数，功能是缩短logger名
contextName|cn：上下文名称
date {pattern}：输出日志的打印时间，模式语法与java.text.SimpleDateFormat兼容
p/le/level：日志级别
M/method：输出日志的方法名
r/relative：从程序启动到创建日志记录的时间
m/msg/message：程序提供的信息
n：换行符

<!--root&logger-->
additivity：false，controller中处理后不再root中处理
<logger name="com.example.demo.controller" level="error" additivity="false">
  <appender-ref ref="APPLICATION" />
</logger>
<root level="warn">
	<appender-ref ref="APPLICATION" />
  <appender-ref ref="STDOUT" />
</root>
```

## 日志实战

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```xml
<!--resources/logback-spring.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
   <contextName>demo</contextName>
    <property resource="application.properties" />
    <property name="LOG_PATH" value="${logging.file.path:-${user.home}/${spring.application.name}/logs}" />
<!--    <property name="LOG_FILE" value="${logging.file.name:-${LOG_PATH}/application.log}" />-->
    <property name="LOG_FILE" value="${LOG_PATH}/${logging.file.name}" />


    <appender name="APPLICATION" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
    </appender>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.demo.controller" level="error" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <root level="info">
        <appender-ref ref="APPLICATION" />
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

```java
// resources/application.properties
logging.file.path=/Users/dingyuanjie/work/SpringBoot_mk/log
logging.file.name=demo.log

// 启动类
@EnableAspectJAutoProxy

public class SystemException extends RuntimeException {
    public SystemException() {
    }

    public SystemException(String message) {
        super(message);
    }

    public SystemException(String message, Throwable cause) {
        super(message, cause);
    }

    public SystemException(Throwable cause) {
        super(cause);
    }
}

public class BusinessException extends RuntimeException {
    public BusinessException() {
    }

    public BusinessException(String message) {
        super(message);
    }

    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }

    public BusinessException(Throwable cause) {
        super(cause);
    }
}

@Component
@Aspect
public class ExceptionAspect {

    private static final Logger  logger = LoggerFactory.getLogger(ExceptionAspect.class);

    @AfterThrowing(pointcut = "within(com.demo.controller.*)", throwing = "ex")
    public void handleException(JoinPoint joinPoint, Exception ex) throws Exception {
        Class declaringType = joinPoint.getSignature().getDeclaringType();
        String clazz = declaringType.getCanonicalName();
        String name = joinPoint.getSignature().getName();
        if(ex instanceof BusinessException) {
            logger.warn("clazz:{}, name:{}", clazz, name, ex);
        } else if (ex instanceof SystemException) {
            logger.error("clazz:{}, name:{}", clazz, name, ex);
        }
        throw ex;
    }
}

@Component
public class DemoHandler {
    public Mono<ServerResponse> logTest(ServerRequest request) {
        Integer id = Integer.valueOf(request.pathVariable("id"));
        if( id == 1) {
            throw new BusinessException("can not use this id " + id);
        } else if (id == 2) {
            throw new SystemException("can not use this id " + id);
        }
        return ok().contentType(MediaType.TEXT_PLAIN).body(Mono.just("log test"), String.class);
    }
}

@Configuration
public class RouterConfig {
    @Bean
    public RouterFunction<ServerResponse> demoRouter() {
        return route(GET("/hello"), demoHandler::hello)
                .andRoute(GET("/logtest"), demoHandler::logTest);
    }
}
```

* 日志分类管理

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
     <contextName>demo</contextName>
      <property resource="application.properties" />
      <property name="LOG_PATH" value="${logging.file.path:-${user.home}/${spring.application.name}/logs}" />
  <!--    <property name="LOG_FILE" value="${logging.file.name:-${LOG_PATH}/application.log}" />-->
      <property name="LOG_FILE" value="${LOG_PATH}/${logging.file.name}" />
  
      <appender name="SIFT" class="ch.qos.logback.classic.sift.SiftingAppender">
          <discriminator>
              <key>bizType</key>
              <defaultValue>OTHER</defaultValue>
          </discriminator>
          <sift>
              <property name="BIZ_FILE" value="${LOG_PATH}/application-${bizType}.log" />
              <appender name="APPLICATION-${bizType}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                  <file>${BIZ_FILE}</file>
                  <encoder>
                      <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
                  </encoder>
                  <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                      <fileNamePattern>${BIZ_FILE}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
                      <maxHistory>7</maxHistory>
                      <maxFileSize>50MB</maxFileSize>
                      <totalSizeCap>20GB</totalSizeCap>
                  </rollingPolicy>
              </appender>
          </sift>
      </appender>
  
      <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
              <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
          </encoder>
      </appender>
  
      <root level="info">
          <appender-ref ref="SIFT" />
          <appender-ref ref="STDOUT" />
      </root>
  </configuration>
  ```

  ```java
  public Mono<ServerResponse> logTest(ServerRequest request) {
    Integer id = Integer.valueOf(request.pathVariable("id"));
    if( id == 1) {
      MDC.put("bizType", "MONEY");
      throw new BusinessException("can not use this id " + id);
    } else if (id == 2) {
      MDC.put("bizType", "HOUSE");
      throw new SystemException("can not usr this id " + id);
    }
    return ok().contentType(MediaType.TEXT_PLAIN).body(Mono.just("log test"), String.class);
  }
  ```

```java
// 日志结合切面做法
在logback-spring.xml中定义好日志配置
引入spring-boot-starter-aop
定义不同级别的异常类
编写异常切面类捕获程序抛出错误记录必要信息
在启动类上加上EnableAspectJAutoProxy注解
  
// 业务日志分类
使用SiftingAppender
在元素discriminator中定义好使用的key
sift里给每个业务类型配置appender
在程序上下文中通过MDC注入业务信息  
```

## 章节总结

