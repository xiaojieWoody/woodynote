1. ==自己debug跑几遍源码，多看源码==

# 概述

* 快速创建、简化编码、简化配置、简化部署、大势所趋

* `https://www.pmdaniu.com/storages/110175/210cd48a43a98239c4088d73dea4801b-76070/index.html?_d=Sun%20Sep%2020%202020%2021:17:37%20GMT+0800%20(CST)`

  ![image-20200920211651671](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920211651671.png)

* Java8新特性
  * Lambda表达式
  * Stream操作
  * 接口默认$静态方法
  * 方法引用
  * 重复注解
  * 类型注解
  * 日期$时间API
  * base64加解密API
  * 数组并行操作
  * JVM新增元空间
  * ==《Java8实战》==
* Maven优点
  * 依赖管理
  * 项目构建
  * 生命周期
  * 插件机制
  * 《Maven实战》
* MySQL
  
  * ==《高性能MySQL》==

# 全局流程解析

## SSM

* Spring

  ![image-20200920214545123](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920214545123.png)

* SpringMVC

  ![image-20200920214502977](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920214502977.png)

* MyBatis

  ![image-20200920214642944](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920214642944.png)

![image-20200920220738159](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920220738159.png)

![image-20200920221403827](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920221403827.png)

## 框架整体启动流程

### 1. new SpringApplication()框架初始化

* 配置资源加载器（配置resourceLoader）
* 配置primarySources
* 应用环境检测（配置webApplicationType）
* 配置系统初始化器（配置ApplicationContextInitializer）
* 配置应用监听器（配置ApplicationListener）
* 配置main方法所在类（配置mainApplicationClass）

### 2.SpringApplication().run框架启动

![image-20200920222145437](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200920222145437.png)

* 计时器开始计时 
* Headless模式赋值 （设置java.awt.headless）
* 发送ApplicationStartingEvent 
* 设置ApplicationArguments
* 配置环境（发送ApplicationEnvironmentPreparedEvent）
* 打印banner
* 上下文配置
* 配置失败记录器
* 准备上下文
  * 关联组件到上下文
  * 遍历调用initializers的initialize方法
  * 发送ApplicationContextInitializedEvent
  * 注册springApplicationArguments
  * 注册springBootBanner
  * 加载sources到context
  * 发送ApplicationPreparedEvent
* 刷新上下文
  * 获取beanFactory
  * 准备beanFactory
  * 调用BeanFactoryPostProcessors
  * 注册BeanPostProcessors
    * 调用BeanDefinitionRegistryPostPorcessor
      * DeferredImportSelectorGroupingHandler#processGroupImports
        * 收集配置文件中的配置工厂类
        * 加载组件工厂
        * 注册组件内定义bean
  * 初始化MessageSource
  * 注册listener beans
  * 实例化单例bean
  * 发布对应事件
  * 清除缓存
  * 注册钩子方法
* 计时器停止计时
* 发送ApplicationStartedEvent
* 回调runners
* 发送ApplicationReadyEvent

### 3. 自动化装配

* 收集配置文件中的配置工厂类
* 加载组件工厂
* 注册组件内定义Bean

# 初始化器解析

## 工厂加载机制解析

==三种方式分别都debug源码跑几遍==

### 实现方式一

```java
//1. 实现ApplicationContextInitializer接口
@Order(1)
public class FirstInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Map<String, Object> map = new HashMap<>();
        map.put("key1", "value1");
        MapPropertySource mapPropertySource = new MapPropertySource("firstInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run firstInitializer");
    }
}
//2. resources/META-INF/spring.factories内填写接口实现
//   key值为org.springframework.context.ApplicationContextInitializer，value为自定义初始化器全路径

// 获取初始值
@Component
public class TestService implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    public String test() {
        return applicationContext.getEnvironment().getProperty("key1"); // key2、key3
    }
}
```

### 实现方式二

```java
//1. 实现ApplicationContextInitializer接口
@Order(2)
public class SecondInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Map<String, Object> map = new HashMap<>();
        map.put("key2", "value2");
        MapPropertySource mapPropertySource = new MapPropertySource("secondInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run secondInitializer");
    }
}
//2. SpringApplication类初始后设置进去
@SpringBootApplication
@MapperScan("com.mooc.sb2.mapper")
public class Sb2Application {
	public static void main(String[] args) {
//		SpringApplication.run(Sb2Application.class, args);
		SpringApplication springApplication = new SpringApplication(Sb2Application.class);
		springApplication.addInitializers(new SecondInitializer());
		springApplication.run(args);
	}
}
```

### 实现方式三

```java
// 1. 实现ApplicationContextInitializer接口
@Order(3)
public class ThirdInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Map<String, Object> map = new HashMap<>();
        map.put("key3", "value3");
        MapPropertySource mapPropertySource = new MapPropertySource("thirdInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run thirdInitializer");
    }
}

// 2. application.properties内填写接口实现
// 3. key为context.initializer.classes 
context.initializer.classes=com.mooc.sb2.initializer.ThirdInitializer
// 由DelegatingApplicationContextInitializer实现，debug源码时的入口
```

* 都要实现ApplicationContextInitializer接口
* Order值越小越先执行
* application.properties中定义的优先于其他方式

### SpringFactoriesLoader

```java
/**
 * General purpose factory loading mechanism for internal use within the framework.
 *
 * <p>{@code SpringFactoriesLoader} {@linkplain #loadFactories loads} and instantiates
 * factories of a given type from {@value #FACTORIES_RESOURCE_LOCATION} files which
 * may be present in multiple JAR files in the classpath. The {@code spring.factories}
 * file must be in {@link Properties} format, where the key is the fully qualified
 * name of the interface or abstract class, and the value is a comma-separated list of
 * implementation class names. For example:
 *
 * <pre class="code">example.MyService=example.MyServiceImpl1,example.MyServiceImpl2</pre>
 *
 * where {@code example.MyService} is the name of the interface, and {@code MyServiceImpl1}
 * and {@code MyServiceImpl2} are two implementations.
 *
 * @author Arjen Poutsma
 * @author Juergen Hoeller
 * @author Sam Brannen
 * @since 3.2
 */
public final class SpringFactoriesLoader {}
```

1. 框架内部使用的通用工厂加载机制
2. 从classpath下多个jar包特定的位置读取文件并初始化类
3. 文件内容必须是kv形式，即properties类型
4. key是全限定名（抽象类｜接口）、value是实现，多个实现用,分割

* SpringFactoriesLoader作用
  * SpringBoot框架中从类路径jar包中读取特定文件实现扩展类的载入
  * ![image-20200921205356216](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921205356216.png)
* ![image-20200921205438893](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921205438893.png)

## 系统初始化器

* 类名：ApplicationContextInitializer
* 介绍：Spring容器刷新之前执行的一个回调函数
* 作用：向SpringBoot容器中注册属性
* 使用：继承接口自定义实现

## 系统初始化器原理

* 作用
  * 上下文刷新即refresh方法前调用
  * 用来编码设置一些属性变量通常用在web环境中
  * 可以通过order接口进行排序

## 总结

![image-20200921210929836](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921210929836.png)

* 三种方式的分别实现原理
  * 定义在spring.factories文件中被SpringFactoriesLoader发现注册
  * SpringApplication初始化完毕后手动添加
  * 定义成环境变量被DelegatingApplicationContextInitializer发现注册

## 面试题

* 介绍下SpringFactoriesLoader？
  * SpringBoot工厂加载类，用来完成SpringBoot扩展点实现的载入
* SpringFactoriesLoader如何加载工厂类？
  * 读取指定路径下的文件，读取成property对象，然后依次遍历内容，组装成类名及对应的实现，通过order进行排序
* 系统初始化器作用？
  * SpringBoot的一个回调接口，通过它可以向SpringBoot来自定义属性
* 系统初始化器的调用时机？
  * SpringBoot的run方法中的prepareContext中调用
* 如何自定义实现系统初始化器？
  * 三种方式
* 自定义实现系统初始化器有哪些注意事项？
  * order值大小排序、不同实现方式使得order失效

# 监听器解析

## 监听器模式介绍

![image-20200921215637328](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921215637328.png)

```java
public abstract class WeatherEvent {
    public abstract String getWeather();
}
```

```java
public class SnowEvent extends WeatherEvent {
    @Override
    public String getWeather() {
        return "snow";
    }
}
public class RainEvent extends WeatherEvent {
    @Override
    public String getWeather() {
        return "rain";
    }
}
```

```java
public interface WeatherListener {
    void onWeatherEvent(WeatherEvent event);
}
```

```java
public class SnowListener implements WeatherListener {
    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof SnowEvent) {
            System.out.println("hello " + event.getWeather());
        }
    }
}

public class RainListener implements WeatherListener {
    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof RainEvent) {
            System.out.println("hello " + event.getWeather());
        }
    }
}
```

```java
public interface EventMulticaster {
    void multicastEvent(WeatherEvent event);
    void addListener(WeatherListener weatherListener);
    void removeListener(WeatherListener weatherListener);
}
```

```java
public abstract class AbstractEventMulticaster implements EventMulticaster {

    private List<WeatherListener> listenerList = new ArrayList<>();

    @Override
    public void multicastEvent(WeatherEvent event) {
        doStart();
        listenerList.forEach(i -> i.onWeatherEvent(event));
        doEnd();
    }

    @Override
    public void addListener(WeatherListener weatherListener) {
        listenerList.add(weatherListener);
    }

    @Override
    public void removeListener(WeatherListener weatherListener) {
        listenerList.remove(weatherListener);
    }

    abstract void doStart();
    abstract void doEnd();
}
```

```java
public class WeatherEventMulticaster extends AbstractEventMulticaster {

    @Override
    void doStart() {
        System.out.println("begin broadcast weather event");
    }

    @Override
    void doEnd() {
        System.out.println("end broadcast weather event");
    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        WeatherEventMulticaster eventMulticaster = new WeatherEventMulticaster();
        RainListener rainListener = new RainListener();
        SnowListener snowListener = new SnowListener();
        eventMulticaster.addListener(rainListener);
        eventMulticaster.addListener(snowListener);
        eventMulticaster.multicastEvent(new SnowEvent());
        eventMulticaster.multicastEvent(new RainEvent());
        eventMulticaster.removeListener(rainListener);
        eventMulticaster.multicastEvent(new SnowEvent());
        eventMulticaster.multicastEvent(new RainEvent());
    }
}
```

* 监听器模式要素
  * 事件、监听器、广播器、触发机制

## 系统监听器介绍

* 系统监听器介绍ApplicationListener

  ```java
  @FunctionalInterface
  public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
  	/**
  	 * Handle an application event.
  	 * @param event the event to respond to
  	 */
  	void onApplicationEvent(E event);
  
  }
  ```

* 系统广播器介绍

  ```java
  public interface ApplicationEventMulticaster {
    // 添加监听器
    // 删除监听器
    // 广播事件
  }
  ```

* 系统事件介绍

  ![image-20200921220925349](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921220925349.png)

* 事件发送顺序

  ![image-20200921221039958](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921221039958.png)

* 监听器注册

  ![image-20200921221325664](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921221325664.png)

## 监听事件触发机制

```java
@Component
public class WeatherRunListener {
    @Autowired
    private WeatherEventMulticaster eventMulticaster;
    public void snow() {
        eventMulticaster.multicastEvent(new SnowEvent());
    }
    public void rain() {
        eventMulticaster.multicastEvent(new RainEvent());
    }
}
```

```java
@Component
public class WeatherEventMulticaster extends AbstractEventMulticaster {
    @Override
    void doStart() {
        System.out.println("begin broadcast weather event");
    }
    @Override
    void doEnd() {
        System.out.println("end broadcast weather event");
    }
}
```

```java
@Component
public class RainListener implements WeatherListener {
    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof RainEvent) {
            System.out.println("hello " + event.getWeather());
        }
    }
}
@Component
public class SnowListener implements WeatherListener {
    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof SnowEvent) {
            System.out.println("hello " + event.getWeather());
        }
    }
}
```

```java
@Component
public abstract class AbstractEventMulticaster implements EventMulticaster {

    @Autowired
    private List<WeatherListener> listenerList;

    @Override
    public void multicastEvent(WeatherEvent event) {
        doStart();
        listenerList.forEach(i -> i.onWeatherEvent(event));
        doEnd();
    }

    @Override
    public void addListener(WeatherListener weatherListener) {
        listenerList.add(weatherListener);
    }

    @Override
    public void removeListener(WeatherListener weatherListener) {
        listenerList.remove(weatherListener);
    }

    abstract void doStart();
    abstract void doEnd();
}
```

```java
@Autowired
private WeatherRunListener weatherRunListener;

@Test
public void testEvent() {
  weatherRunListener.rain();
  weatherRunListener.snow();
}

begin broadcast weather event
hello rain
end broadcast weather event
begin broadcast weather event
hello snow
end broadcast weather event
```

![image-20200921223321209](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921223321209.png)

![image-20200921223349093](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200921223349093.png)

## 自定义监听器实战

```java
@Order(1)
public class FirstListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello first");
    }
}

// resources/META-INF/spring.factories
org.springframework.context.ApplicationListener=com.mooc.sb2.listener.FirstListener
```

```java
@Order(2)
public class SecondListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello second");
    }
}

@SpringBootApplication
@MapperScan("com.mooc.sb2.mapper")
public class Sb2Application {
	public static void main(String[] args) {
		SpringApplication springApplication = new SpringApplication(Sb2Application.class);
		springApplication.addListeners(new SecondListener());
		springApplication.run();
	}
}
```

```java
@Order(3)
public class ThirdListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello third");
    }
}

// application.properties
context.listener.classes=com.mooc.sb2.listener.ThirdListener
```

```java
@Order(4)
public class FourthListener implements SmartApplicationListener {
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> eventType) {
        return ApplicationStartedEvent.class.isAssignableFrom(eventType) || ApplicationPreparedEvent.class.isAssignableFrom(eventType);
    }

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("hello fourth");
    }
}

// resources/META-INF/spring.factories
org.springframework.context.ApplicationListener=com.mooc.sb2.listener.FourthListener
```

* 实现ApplicationListener接口针对单一事件监听
* 实现SmartApplicationListener接口针对多种事件监听
* Order值越小越先执行
* application.properties中定义的优先于其他方式

## 总结

* 介绍下监听器模式
  * 图、四要素
* SpringBoot关于监听器相关的实现类有哪些？
* SpringBoot框架有哪些框架事件以及它们的顺序
* 介绍下监听事件触发机制？
* 如何自定义实现系统监听器及注意事项？
* 实现ApplicationListener与实现SmartApplicationListener接口区别

# bean解析

## IOC

* 容器

* 松耦合，通过容器来注入
* 灵活性， 不需要在类里调用构造方法来创建实例，这些都在容器里完成构造、注入
* 可维护性

## xml方式配置Bean

### 无参构造

```java
@Data
@ToString
public class Student {
    private String name;
    private Integer age;
  	private List<String> classList;
}
```

```xml
<!--resources/ioc/demo.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.mooc.sb2.ioc.xml.Student">
        <property index="name" value="zhangsan"/>
        <property index="age" value="13"/>
        <property name="classList">
            <list>
                <value>math</value>
                <value>english</value>
            </list>
        </property>
    </bean>

    <bean id="helloService" class="com.mooc.sb2.ioc.xml.HelloService">
        <property name="student" ref="student"/>
    </bean>
</beans>
```

```java
public class  HelloService {

    private Student student;
 	  public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }
  
  	public String hello() {
        return student.toString();
    }
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(locations = "classpath:ioc/demo.xml")
public class Sb2ApplicationTests implements ApplicationContextAware {

    @Autowired
    private HelloService helloService;

    @Test
    public testStu() {
        System.out.println(helloService.hello());
    }
}
```

### 有参构造

```java
@Data
@ToString
public class Student {
    private String name;
    private Integer age;
  	private List<String> classList;
  
  	public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```

```xml
<!--resources/ioc/demo.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.mooc.sb2.ioc.xml.Student">
        <constructor-arg index="0" value="zhangsan" />
        <constructor-arg index="1" value="13"/>
        <property name="classList">
            <list>
                <value>math</value>
                <value>english</value>
            </list>
        </property>
    </bean>

    <bean id="helloService" class="com.mooc.sb2.ioc.xml.HelloService">
        <property name="student" ref="student"/>
    </bean>
</beans>
```

### 静态工厂方法

```java
public abstract class Animal {
    abstract String getName();
}
```

```java
public class Dog extends Animal {
    @Override
    String getName() {
        return "dog";
    }
}
public class Cat extends Animal {
    @Override
    String getName() {
        return "cat";
    }
}
```

```java
public class AnimalFactory {
    public static Animal getAnimal(String type) {
        if ("dog".equals(type)) {
            return new Dog();
        } else {
            return new Cat();
        }
    }
}
```

```java
@Data
public class  HelloService {
    private Animal animal;
    public String hello() {
        return animal.getName();
    }
}
```

```xml
<!--resources/ioc/demo.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

  	<bean id="helloService" class="com.mooc.sb2.ioc.xml.HelloService">
        <property name="animal" ref="cat"/>
    </bean>
  
  	<bean id="dog" class="com.mooc.sb2.ioc.xml.AnimalFactory" factory-method="getAnimal">
        <constructor-arg value="dog"/>
    </bean>

    <bean id="cat" class="com.mooc.sb2.ioc.xml.AnimalFactory" factory-method="getAnimal">
        <constructor-arg value="cat"/>
    </bean>
</beans>
```

### 实例工厂方法

```java
public class AnimalFactory {
    public Animal getAnimal(String type) {
        if ("dog".equals(type)) {
            return new Dog();
        } else {
            return new Cat();
        }
    }
}
```

```xml
<!--resources/ioc/demo.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

  	<bean id="helloService" class="com.mooc.sb2.ioc.xml.HelloService">
        <property name="animal" ref="cat"/>
    </bean>
  
 	  <bean name="animalFactory" class="com.mooc.sb2.ioc.xml.AnimalFactory"/>
  
  	<bean id="dog" factory-bean="animalFactory" factory-method="getAnimal">
        <constructor-arg value="dog"/>
    </bean>

    <bean id="cat" factory-bean="animalFactory" factory-method="getAnimal">
        <constructor-arg value="cat"/>
    </bean>
</beans>
```

* 优点
  * 低耦合、对象关系清晰、集中管理
* 缺点
  * 配置繁琐、开发效率稍低、文件解析耗时

## 注解方式配置Bean

### @Component声明

* service类上标注@Component注解

###  配置类中使用@Bean

```java
@Configuration
public class BeanConfiguration{
    @Bean("dog")
    Animal getDog() {
        return new Dog();
    }
}

@Autowired
private Animal animal;
```

### 继承FactoryBean

```java
@Component
public class MyCat implements FactoryBean<Animal> {
    @Override
    public Animal getObject() throws Exception {
        return new Cat();
    }

    @Override
    public Class<?> getObjectType() {
        return Animal.class;
    }
}

@Autowired
@Qualifier("myCat")      // 有多个冲突时，指定myCat
private Animal animal;
```

### 继承BeanDefinitionRegistryPostProcessor

```java
@Component
public class MyBeanRegister implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(Monkey.class);
        registry.registerBeanDefinition("monkey", rootBeanDefinition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    }
}

@Autowired
@Qualifier("monkey")      // 有多个冲突时，指定myCat
private Animal animal;
```

### 继承ImportBeanDefinitionRegistrar

```java
public class MyBeanImport implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(Bird.class);
        registry.registerBeanDefinition("bird", rootBeanDefinition);
    }
}

//启动类上
@Import(MyBeanImport.class)

@Autowired
@Qualifier("bird")      // 有多个冲突时，指定myCat
private Animal animal;
```

* 优点
  * 使用简单、开发效率高、高内聚
* 缺点
  * 配置分散、对象关系不清晰、配置修改需要重新编译工程

## refresh方法解析

* bean配置读取加载入口
* Spring框架启动流程
* 面试重点

![image-20200923081735709](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200923081735709.png)

```java
// debug断点从run方法进入
org.springframework.context.support.AbstractApplicationContext#refresh
```

* prepareRfresh
  
  1. 容器状态设置

  2. 初始化属性设置
  
  3. 检查必备属性是否存在 
  
     ```java
     @Order(1)
     public class FirstInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
         @Override
         public void initialize(ConfigurableApplicationContext applicationContext) {
             ConfigurableEnvironment environment = applicationContext.getEnvironment();
             // 启动时，如果application.properties中没有 mooc=xxx，启动失败
             environment.setRequiredProperties("mooc");
         }
     } 
     ```
  
* obtainFreshBeanFactory
  
  * 设置beanFactory序列化id
  
  * 获取beanFactory
  
* prepareBeanFactory

  * 设置beanFactory一些属性
  * 添加后置处理器
  * 设置忽略的自动装配接口
  * 注册一些组件

* postProcessBeanFactory

  * 子类重写以在BeanFactory完成创建后做进一步设置

* invokeBeanFactoryPostProcessors
  
  * 调用BeanDefinitionRegistryPostProcessor实现向容器内添加bean的定义
  * 调用BeanFactoryPostProcessor实现向容器内bean的定义的添加属性

![image-20200927210245164](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200927210245164.png)

![image-20200927210317301](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200927210317301.png)

![image-20200927210337570](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200927210337570.png)

![image-20200927210410133](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200927210410133.png)

```java
@Data
@Component
public class Teacher {
    private String name;
}
```

```java
@Component
public class MyBeanFactoryPostprocessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinition teacher = beanFactory.getBeanDefinition("teacher");
        MutablePropertyValues propertyValues = teacher.getPropertyValues();
        propertyValues.addPropertyValue("name", "wangwu");
    }
}
```

```java
@Autowired
private Teacher teacher;

System.out.println(teacher.getName());
```

* registerBeanPostProcessors
  * 找到BeanPostProcessor的实现
  * 排序后注册进容器内
* initMessageSource
  * 初始化国际化相关属性
* initApplicationEventMulticaster
  * 初始化事件广播器，并注册到容器中
* onRefresh
  * 创建web容器
* registerListeners
  * 添加容器内事件监听器至事件广播器中
  * 派发早期事件
* finishBeanFactoryInitialization
  * 初始化所有剩下的单实例bean
* finishRefresh
  * 初始化生命周期处理器
  * 调用生命周期处理器onRefresh方法
  * 发布ContextRefreshedEvent事件
  * JMX相关处理

## bean实例化解析

* BeanDefinition介绍

  * 一个对象在Spring中描述，RootBeanDefinition是其常见实现
  * 通过操作BeanDefinition来完成bean实例化和属性注入

  ![image-20200928081709879](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200928081709879.png)

  ![image-20200928081752579](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200928081752579.png)

  ![image-20200928081849973](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200928081849973.png)

## 总结

* 介绍下ioc思想
* SpringBoot中bean有哪几种配置方式分别介绍下？
* bean的配置你喜欢那种方式？
* 介绍下refresh方法流程
* 请介绍一个refresh中你比较熟悉的方法说出其作用
* 介绍下bean实例化的流程
* 说几个bean实例化的扩展点及其作用？

