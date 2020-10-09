# banner解析

```java
// 默认banner输出
resources/banner.txt中是佛祖图像
// 自定义名称favorite.txt（其中为佛祖图像）
resources/favorite.txt  
application.properties中增加 spring.banner.location=favorite.txt
// 图片banner输出
resources/banner.jpg|gif|png即可
// 自定义图片名
resources/favorite.jpg  
spring.banner.image.location=favorite.jpg  
 // 兜底banner
SpringApplication springApplication = new SpringApplication(Sb2Application.class);
springApplication.setBanner(new ResourceBanner(new ClassPathResource("banner_bak.txt")));
springApplication.run(args); 
// 关闭banner输出 application.properties中
spring.main.banner-mode=off  
```

## banner获取原理

* 输出入口

  * printBanner

* 输出banner逻辑

  * 获取banner

    * getImageBanner
      * 可以通过spring.banner.image.location指定位置
      * 可使用图片格式 ：gif|png|jpg
    * getTextBanner
      * 可以通过spring.banner.location指定位置
      * 默认banner.txt

    ![image-20200928224538145](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200928224538145.png)

  * 打印banner

## banner输出原理

* 默认输出
  * 一、先输出banner指定内容
  * 二、获取version信息
  * 三、文本内容前后对齐
  * 四、文本内容染色
  * 五、输出文本内容
* 文本输出
  * 可以通过spring.banner.charset指定字符集
  * 获取文本内容
  * 替换占位符（banner.txt中{mooc}，application.properties中mooc=test）
  * 输出文本内容
* 图片输出
  * 可以通过spring.banner.image.*设置图片属性
  * 读取图片文件流
  * 输出图片内容

## 总结

* 举例banner常见配置方式
* 简述下框架内banner打印流程步骤
* 说明下banner获取原理
* 说明下banner输出原理
* 说出banner属性有哪些

# 启动加载器解析

## 计时器

* StopWatch

  ```java
  // 源码优点：短小精悍、命名严谨、考虑周到
  StopWatch myWatch = new StopWatch("myWatch");
  // 业务校验 -> 保存任务名 -> 记录当前系统时间
  myWatch.start("task1");
  Thread.sleep(2000L);
  // 业务校验 -> 计算耗时 -> 将当前任务添加到任务列表（可选） -> 任务执行数加一 -> 清空当前任务
  myWatch.stop();
  myWatch.start("task2");
  Thread.sleep(3000L);
  myWatch.stop();
  myWatch.start("task3");
  Thread.sleep(1000L);
  myWatch.stop();
  System.out.println(myWatch.prettyPrint());
  ```

## 启动加载器案例演示

* 方式一

```java
@Component
@Order(1)
public class FirstCommandlineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("\u001B[32m >>> startup first runner<<<");
    }
}

@Component
@Order(2)
public class SecondCommandlineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("\u001B[32m >>> startup second runner<<<");
    }
}
```

* 方式二

```java
@Component
@Order(1)
public class FirstApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("\u001B[32m >>> startup first application runner<<<");
    }
}

@Component
@Order(2)
public class SecondApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("\u001B[32m >>> startup second application runner<<<");
    }
} 
```

* 通过order值指定顺序
* order值相同ApplicationRunner实现优先

## 启动加载器原理解析

```java
// callRunners实现
添加ApplicationRunner实现至runners集合
添加CommandLineRunner实现至runners集合
对runners集合排序
遍历runners集合一次调用实现类的run方法  
  
// 实现类差异点
执行优先级差异
run方法入参不一致
  
// 实现类相同点
调用点一样
实现方法名一样  
```

## 总结

* SpringBoot计时器的实现？它有哪些优点？
* 实现一个计时器的思路
* 怎么实现在SpringBoot启动后执行程序？
* 启动加载器如何实现
* 启动加载器的实现有什么异同点
* 启动加载器的调用时机

# 属性配置解析

## 属性配置解析

1. Devtools全局配置
2. 测试环境@TestPropertySource注解
3. 测试环境properties属性
4. 命令行参数
5. SPRING_APPLICATION_JSON属性
6. ServletConfig初始化参数
7. ServletContext初始化参数
8. JNDI属性
9. JAVA系统属性
10. 操作系统环境变量
11. RandomValuePropertySource随机值属性
12. jar包外的application-{profile}.properties
13. jar包内的application-{profile}.properties
14. jar包外的application.properties
15. jar包内的application.properties
16. @PropertySource绑定配置
17. 默认属性

```java
@Component
public class ResultCommandLineRunner implements CommandLineRunner, EnvironmentAware {
    private Environment env;

    @Override
    public void run(String... args) throws Exception {
        System.out.println(env.getProperty("mooc.wesite.url"));
	      System.out.println(env.getProperty("mooc.avg.age"));
      	System.out.println(env.getProperty("mooc.website.path"));
      	System.out.println(env.getProperty("mooc.vm.name"));
        System.out.println(env.getProperty("mooc.defalut.name"));
        System.out.println(env.getProperty("mooc.active.name"));
    }

    @Override
    public void setEnvironment(Environment environment) {
        env = environment;
    }
}
```

```java
SpringApplication springApplication = new SpringApplication(Sb2Application.class);
Properties properties = new Properties();
properties.setProperty("mooc.wesite.url", "mooc_url_1");
springApplication.setDefaultProperties(properties);
springApplication.run(args);
```

```java
@PropertySource({"demo.properties"})  

resources/demo.properties
mooc.website.url=mooc_url_2
```

```java
resources/application.yml
  
mooc:
  website:
    url:
      mooc_url_3
```

```java
resources/application.properties
mooc.website.url=mooc_url_4  
```

```java
resources/application-default.yml
mooc:
  website:
    url:
      mooc_url_5  
```

```java
resources/application-default.properties
mooc.website.url=mooc_url_6
mooc.avg.age=${random.int[20,30]}  
mooc.website.path=${PATH}
mooc.vm.name=${java.vm.name}
```

```java
// 启动参数中设置
--SPRING_APPLICATION_JSON={\"mooc.website.url\":\"mooc_url_7\"}
```

```java
// 启动参数中设置
--mooc.website.url=mooc_url_8
```

## Spring Aware介绍

* Spring框架优点：Bean感知不到容器的存在

* 使用场景：需要使用Spring容器的功能资源

* 引入缺点：Bean和容器强耦合

* 常用Aware

  ```shell
  # BeanNameAware
  获得容器中bean名称
  # BeanClassLoaderAware
  获得类加载器
  # BeanFactoryAware
  获得bean创建工厂
  # EnvironmentAware
  获得环境变量
  # EmbeddedValueResolverAware
  获取Spring容器加载的properties文件属性值
  # ResourceLoaderAware
  获得资源加载器
  # ApplicationEventPublisherAware
  获得应用事件发布器
  # MessageSourceAware
  获得文本信息
  # ApplicationContextAware
  获得当前应用上下文
  ```

* Aware调用

  ```shell
  doCreateBean -> initializeBean -> invokeAwareMethods -> applyBeanPostProcessorsBeforeInitialization -> ApplicationContextAwareProcessor
  ```

  ```java
  @Data
  @Component
  public class Flag {
      private boolean canOperate = true;
  }
  ```

  ```java
  public interface MyAware extends Aware {
      void setFlag(Flag flag);
  }
  ```

  ```java
  @Component
  public class MyAwareProcessor implements BeanPostProcessor {
      private final ConfigurableApplicationContext configurableApplicationContext;
      public MyAwareProcessor(ConfigurableApplicationContext configurableApplicationContext) {
          this.configurableApplicationContext = configurableApplicationContext;
      }
  
      @Override
      public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
          if (bean instanceof Aware) {
              if (bean instanceof MyAware) {
                  ((MyAware) bean).setFlag((Flag) configurableApplicationContext.getBean("flag"));
              }
          }
          return bean;
      }
  }
  ```

  ```java
  @Component
  public class ResultCommandLineRunner implements CommandLineRunner, EnvironmentAware, MyAware {
  
      private Environment env;
      private Flag flag;
  
      @Override
      public void run(String... args) throws Exception {
        	System.out.println(flag.isCanOperate());               // true
          System.out.println(env.getProperty("mooc.defalut.name"));
          System.out.println(env.getProperty("mooc.active.name"));
      }
  
      @Override
      public void setEnvironment(Environment environment) {
          env = environment;
      }
  
      @Override
      public void setFlag(Flag fla) {
          flag = fla;
      }
  }
  ```

* 自定义实现Aware

  ```shell
  定义一个接口继承Aware接口 -> 定义setX方法 -> 写一个BeanPostProcessor实现 -> 改写postProcessorsBeforeInitialization方法
  ```

## Environment解析

* 获取属性

  ```shell
  AbstractEnvironment#getProperty
  PropertySourcesPropertyResolver#getProperty
  遍历propertySources集合获取属性
  Environment对象如何填充该集合？
  ```

  ```shell
  # 源码入口
  SpringApplication#run#prepareEnvironment
  ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
  # getOrCreateEnvironment
  1. 添加servletConfigInitParams属性集
  2. 添加servletContextInitParams属性集
  3. 添加Jndi属性集
  4. 添加systemProperties属性集
  5. 添加systemEnvironment属性集
  # configrueEnvironment
  1. 添加defaultProperties属性集
  2. 添加commandLineArgs属性集
  # listeners.environmentPrepared
  1. 添加spring_application_json属性集
  2. 添加vcap属性集
  3. 添加random属性集
  4. 添加application-profile.(properties|yml)属性集
  # ConfigurationPropertySources.attach
  1. 添加configurationProperties属性集
  # ConfigurationClassParser
  1. 添加@PropertySources属性集
  ```

## Spring profile介绍

* 将不同的配置参数绑定在不同的环境

* 默认使用

  * application.properties
  * application-default.properties
  * spring.profiles.default=defaults（不能定义在application文件中，可添加在启动参数后面）

* 激活profile

  * spring.profiles.active=xx（也添加到启动参数后面）
  * spring.profiles.active与default互斥  

  ```shell
  application-online.properties
  application-online2.properties
  # 在application.properties中激活
  spring.profiles.include=online,online2
  
  # 指定profile前缀
  spring.config.name=xxx
  ```

## Spring profile解析

```shell
# 处理入口
ConfigFileApplicationListener#onApplicationEvent
postProcessEnvironment
addPropertySources
Loader.load
```

![image-20201007102411137](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007102411137.png)

![image-20201007102530488](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007102530488.png)

![image-20201007102619886](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007102619886.png)

![image-20201007102834748](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007102834748.png)

![image-20201007102909391](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007102909391.png)

## 总结

* SpringBoot属性配置方式有哪些？
* 介绍下Spring Aware的作用及常见的有哪些？
* Spring Aware注入原理
* 动手写一个Spring Aware
* Environment对象是如何加载属性集的？
* 部分属性集如spring_application_json何时被加载的？
* 介绍下Spring Profile？常用配置方式
* Spring Proflie配置方式有哪些注意事项，为什么？
* Spring Profile处理逻辑

# 异常报告器解析

## 异常报告器类介绍

```java
// 接口规范
@FunctionalInterface
public interface SpringBootExceptionReporter {
	/**
	 * Report a startup failure to the user.
	 * @param failure the source failure
	 * @return {@code true} if the failure was reported or {@code false} if default
	 * reporting should occur.
	 */
	boolean reportException(Throwable failure);
}
```

```java
public class AException extends Exception {
    public AException(Throwable cause) {
        super(cause);
    }
}

public class BException extends Exception {
    public BException(Throwable cause) {
        super(cause);
    }
}

public class CException extends Exception {
    public CException(Throwable cause) {
        super(cause);
    }
}

// 测试
try {
			throw new CException(new BException(new AException(new Exception("test"))));
	 	} catch (Throwable t) {
			while (t != null) {
				System.out.println(t.getClass());
				t = t.getCause();
		}
}
// class com.mooc.sb2.except.CException
// class com.mooc.sb2.except.BException
// class com.mooc.sb2.except.AException
// class java.lang.Exception
```

```shell
# 框架内实现
run方法
Collection<SpringBootExceptionReporter>
getSpringFactoriesInstances
填充集合内容
```

```java
// reportException实现
analyze方法
遍历analyzers集合找到能处理该异常的对象
report方法
FailureAnalysisReporter实现类报告异常  
```

![image-20201007105319561](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007105319561.png)

```java
 // analyze逻辑
getCauseType方法
获取子类感兴趣异常类型
findCause方法
判断当前抛出的异常栈是否包含子类感兴趣异常
调用子类具体analyze实现给出异常分析结果类
```

```java
// FailureAnalysisReporter介绍 
功能：报告异常给到用户
实现类：LoggingFailureAnalysisReporter
实现方法：根据失败分析结果类构建错误信息输出  
```

## SpringBoot异常处理流程

```java
// 处理入口
try {
  ...;
} catch (Throwable ex) {
  handleRunFailure(context, ex, exceptionReporters, listeners);
  throw new IllegalStateException(ex);
}
// handleRunFailure逻辑
1. handleExitCode
exitCode退出状态码，为0代表正常退出，否则异常退出
发布ExitCodeEvent事件
记录exitCode  
  
2. listeners.failed
发布ApplicationFailedEvent事件
  
3. reportFailure
SpringBootExceptionReporter实现调用reportException方法
成功处理的话记录已处理异常  
  
4. context.close
更改应用上下文状态
销毁单例bean
beanFactory置为空
关闭web容器(web环境)
移除shutdownHook
shutdownHook介绍
  作用：jvm退出时执行的业务逻辑
  添加：Runtime.getRuntime().addShutdownHook
  移除：Runtime.getRuntime().removeShutdownHook
  
5. ReflectionUtils.rethrowRuntimeException  
重新抛出异常  
```

```java
System.out.println("hello");
Thread close_jvm = new Thread(() -> System.out.println("close jvm")); // 在world后面输出
Runtime.getRuntime().addShutdownHook(close_jvm);
System.out.println("world");
//Runtime.getRuntime().removeShutdownHook(close_jvm);       // 不输出close jvm
```

## 案例分析

* 端口被占用异常

  ```java
  // ConnectorStartFailedException
  public class MySocket {
      public static void main(String[] args) throws Exception {
          ServerSocket serverSocket = new ServerSocket(8080);
          serverSocket.accept();
      }
  }
  
  // 断点
  SpringApplication.run(...)
    
  @Component
  public class MyExitCodeExceptionMapper implements ExitCodeExceptionMapper {
      @Override
      public int getExitCode(Throwable exception) {
          if (exception instanceof ConnectorStartFailedException) {
              return 10;
          }
          return 0;
      }
  }  
  ```

* bean注入失败

  ```java
  // UnsatisfiedDependencyException
  public class Solid {
  }
  
  @Autowired
  private Solid solid;
  ```

  ![image-20201007113847800](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007113847800.png)

* SpringBootExceptionReporter自定义实现

  ```java
  // 实现SpringBootExceptionReporter接口
  public class MyExceptionReporter implements SpringBootExceptionReporter {
  
      private ConfigurableApplicationContext context;
  
      // 需提供一个有参构造方法
      public MyExceptionReporter(ConfigurableApplicationContext context) {
          this.context = context;
      }
  
      // 重写reportException方法，返回值决定是否需要使用下一个实现处理
      @Override
      public boolean reportException(Throwable failure) {
          if (failure instanceof UnsatisfiedDependencyException) {
              UnsatisfiedDependencyException exception = (UnsatisfiedDependencyException) failure;
              System.out.println("no such bean " + exception.getInjectionPoint().getField().getName() );
          }
          return false;
      }
  }
  
  // resources/META-INF/spring.factories
  org.springframework.boot.SpringBootExceptionReporter=com.mooc.sb2.except.MyExceptionReporter
    
  // 在handleRunFailure#reportFailure处打断点  
  ```

* ExitCodeExceptionMapper自定义实现
  * 实现ExitCodeExceptionMapper接口
  * 重写getExitCode方法
  * 给异常赋予非0正数exitCode

## 章节总结

```java
异常报告器类介绍
定义：SpringBootExceptionReporter
填充：getSpringFactoriesInstances
实现：FailureAnalyzers  
  
// 核心方法逻辑
方法：reportException
步骤一：analyze
步骤二：report  
```

* 关闭钩子方法的作用及使用方法？
* 介绍下SpringBoot异常报告器类结构？
* 介绍下SpringBoot异常报告器的实现原理？
* 讲述下SpringBoot异常处理流程？
* SpringBoot异常处理流程中有哪些注意事项
* 如何自定义实现SpringBoot异常报告器？

# 配置类解析

## 全局流程分析

```java
// 源码入口
// 配置类解析入口
// refresh#invokeBeanFactoryPostProcessors#ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry
```

```java
// postProcessBeanDefinitionRegistry
获得BeanDefinitionRegistry的唯一id：registryId
检查一下registryId是否已处理过  
添加registryId到已处理集合中
processConfigBeanDefinitions  
```

![image-20201007145449372](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007145449372.png)

## 执行逻辑解析

```java
// 执行入口
do {
  parser.parse(candidates);
  ...
}
while(!candidates.isEmpty())
  
// 循环体逻辑
ConfigurationClassParser#parse
ConfigurationClassParser#validate
读取BeanMethod注册BeanDefinition
处理新引入的BeanDefinition

// parse方法调用链
ConfigurationClassParser#parse
同名parse方法
processConfigurationClass
doProcessConfigurationClass
1. 内部类
2. PropertySource
3. ComponentScan
4. Import
5. ImportResource
6. BeanMethod
7. 接口默认方法
8. 父类 
```

## 核心方法解析

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        System.out.println("MyImportSelector");
        return new String[]{"com.mooc.sb2.ioc.xml.Cat", "com.mooc.sb2.ioc.xml.Dog"};
    }
}
```

```java
public class MyDeferredImportSelector implements DeferredImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        System.out.println("MyDeferredImportSelector");
        return new String[]{"com.mooc.sb2.ioc.xml.Bird", "com.mooc.sb2.ioc.xml.Monkey"};
    }
}
```

```java
// main
@Import({MyImportSelector.class,MyDeferredImportSelector.class})
// 管理xml中的bean  @ImportResource("ioc/demo.xml")
```

![image-20201007152026150](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007152026150.png)

```java
PropertySource处理
用法：@PropertySource({"demo.properties"})  
遍历指定路径，替换占位符，加载资源
将资源添加到environment中

ComponentScan处理
@ComponentScan(basePackages={"pkgA","pkgB"},basePackageClasses={A.class,B.class})  
没设置扫描路径的话，使用配置类所在路径
过滤顺序: excludeFilters -> includeFilters -> false
  
Import处理
ImportSelector.class & DeferredImportSelector.class
处理以上两个接口实现selectImports返回的类名数组
DeferrredImportSelector接口调用优先级低于其他接口
处理ImportBeanDefinitionRegistrar实现中注册的bean
处理@Import(A.class) 
  
ImportResource处理
@ImportResource("xyz.xml")  
将注解属性值放入importedResources中
后续loadBeanDefinitionsForConfigurationClass中加载定义的bean
  
BeanMethod处理
@Configuration
public class BeanConfiguration {
  @Bean("dog")
  Animal getDog() {
    return new Dog();
  }
} 
构造BeanMethod对象放入配置类属性中后续处理

接口默认方法处理
public interface SuperConfiguration {
  @Bean("dog")
  default Animal getDog() {
    return new Dog();
  }
}  

父类处理
一：不为null
二：全路径名不以java开头
三：尚未处理过  
```

## 章节总结

* 配置类是什么？起到什么作用？
* 请举例一些常用的配置注解？
* 介绍下SpringBoot框架对配置类的一个处理流程
* 你能说出其中的一些关键类和方法吗？
* 配置类的处理一般包括哪些内容
* 详细的一些注解处理过程？如Import注解

# Servlet容器启动解析

* tomcat

  * 轻量级web应用服务器

  ![image-20201007195955819](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007195955819.png)

  ![image-20201007200102560](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201007200102560.png)

## 启动流程解析

```java
// 源码入口
webApplicationType.deduceFromClassPath();

// 启动前准备
SpringApplication构造方法
赋值webApplicationType属性
根据classpath下是否存在特定类来决定
SERVLET、REACTIVE、NONE
refresh方法
createApplicationContext方法
根据webApplicationType属性决定上下文
初始化DEFAUTL_SERVLET_WEB_CONTEXT_CLASS
  
// webServer创建入口
refreshContext
refresh
onRefresh
createWebServer

// webServer创建
getWebServerFactory
factory.getWebServer
设置webServer属性
initPropertySources
  
// servlet启动
refresh
finishRefresh
startWebServer
publishEvent  
```

## web容器工厂类加载解析

```java
// 配置引入
// 注解处理
// 容器工厂类引入
```

## web容器个性化配置解析

```java
// 属性配置  
// application.properties
server.port=9000
// createWebServer#getWebServerFactory
// 属性注入
配置web容器属性，如server.xxx=xyz
注入到ServerProperties类中
自动配置类导入WebServerFactoryCustomizer实现类
ServerProperties成为实现类的属性
  
// 工厂类初始化
getWebServerFactory获取具体web服务工厂类
对具体实现类调用doGetBean进行初始化
遍历BeanPostProcessor实现对bean处理
进入WebServerFactoryCustomizerBeanPostProcessor实现方法
  
// BeanPostProcessor方法实现
postProcessBeforeInitialization
getCustomizers()
获得所有的WebServerFactoryCustomizer实现类
依次调用实现类的customize方法进行定制处理
  
// 定制化流程
ServletWebServerFactoryCustomizer#customize
构建PropertyMapper工具类
从ServerProperties属性中获取自定义设置
将自定义属性赋值给WebServerFactory
创建时应用到具体的webServer中 
```

## 总结

* 请描述下Servlet容器启动流程？
* 介绍下AutoConfigurationImportSelector的作用
* AutoConfigurationImportSelector何时被处理
* 为何SpringBoot框架默认启动的是tomcat容器
* 请列举下常见的web容器自定义配置参数
* 讲解下自定义配置参数生效的原理
