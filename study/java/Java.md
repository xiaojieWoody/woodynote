# MS

* Java程序运行在Docker等容器环境有哪些新问题?

  * 主要是运行环境所需资源问题，如容器未设置合适的JVM堆和元数据区、直接内存等参数，导致OOM；容器限制CPU核数可能导致JVM设置不合适的GC并行线程数；

* 后台服务出现明显“变慢”，谈谈你的诊断思路?

  * 慢的意思是指请求的反应延时变长吗？
  * 突然变慢还是长时间运行后变慢，类似问题是否重复出现
  * 问题可能是Java服务本身，也可能是受系统里其他服务的影响
    - 检查应用本身的错误日志
    - 检查系统级别的资源等情况，监控CPU、内存等资源是否被其他进程大量占用
  * 监控Java服务自身
    - 例如GC日志里面是否观察到Full GC等恶劣情况出现，或者是否Minor GC在变长等
    - 利用jstat等工具，获取内存使用的统计信息
    - 利用jstack等工具检查是否出现死锁

* JDK，JRE 和 JVM 的联系和区别

  - JVM 代表 Java 虚拟机
    - 将编译后的字节码文件转换为特定的机器代码，提供内存回收和安全机制
    - 它的责任是运行 Java 应用，独立于操作系统和硬件，是java语言一次编译到处运行的原因
  - JRE 代表 Java 运行时环境
    - 是运行 Java程序所必须的，包含jvm和一些依赖
  - JDK 代表 Java 开发工具
    - 包含 Java 编译器、 JRE
  - JIT 代表即时编译
    - 混合了静态编译和动态解释，一句一句编译源代码，但是会将翻译过的代码缓存起来以降低性能损耗
      - 静态编译的程序在执行前全部被翻译为机器码
      - 解释执行的则是一句一句边运行边翻译

* Java8

  * HashMap底层实现新增了红黑树；

  * jvm内存管理：由元空间代替了永久代，元空间不再存在虚拟机内存中，而是本地内存，大小默认情况下只受本地内存控制

  * lambda表达式：允许把函数当成参数，传递给某个方法，或者把代码本身当做数据处理；

  * 引入重复注解：@repeatable注解，来定义注解为重复注解

  * 注解的使用场景拓宽：注解可以使用在任何元素上

  * 新的包，java.time包

    * 该包包含了所有关于日期、时间、时区、持续时间和时钟操作的类
    * 这些类都是不可变的，线程安全的

  * 接口中增加静态方法和默认方法

    * 默认方法：
      * 接口中方法：返回值前用default修饰且有方法体
      * 如果一个类实现了多个接口，且这些接口中有相同签名的默认方法，则实现类需要重写默认方法
    * 静态方法：
      * 属于接口，只能通过接口名调用，不能被实现类重写，不能通过实现类实例调用

  * 函数接口

    - @FunctionalInterface标记的接口且只有一个抽象方法

  * 可以使用lambda表达式创建函数接口的实例`(argument) -> (body)`

  * 简化代码；支持Stream API

  * Stream API

    - 不存储数据，操作源数据结构，产生并使用管道数据，实现具体的操作
    - 基于条件的从list和filter中创建Stream，使用函数接口

    ```java
    private static int sumIterator(List<Integer> list) {
    	Iterator<Integer> it = list.iterator();
    	int sum = 0;
    	while (it.hasNext()) {
    		int num = it.next();
    		if (num > 10) {
    			sum += num;
    		}
    	}
    	return sum;
    }
    
    private static int sumStream(List<Integer> list) {
    	return list.stream().filter(i -> i > 10).mapToInt(i -> i).sum();
    }
    ```
    * Collection的forEach()方法，接收Consumer参数，使用lambda表达式

# 经验

# 基础

