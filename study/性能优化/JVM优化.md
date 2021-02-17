![image-20210216221021284](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216221021284.png)

# JVM优化的必要性

本地开发环境和生产环境大相径庭

​	运行应用“卡住了”，日志不输出，程序没有反应

​	服务器的CPU负载突然升高

​	在多线程应用下，如何分配线程的数量

# JVM的运行参数

三种参数类型

## 标准参数

-help、-version

一般都很稳定，可以使用java -help查看

```java
// 实战：通过-D设置系统属性参数
// 实战意义：通过系统参数可以传入项目环境标记，根据不同环境选择不同配置
public class TestVM {
  public static void main(String[] args) {
    String name = System.getProperty("name");
    if(name != null) {
      System.out.println(name);
    } else {
      System.out.println("ling");
    }
  }
}
// javac TestVM.java
// java TestVM
ling
// java -Dname=woody TestVM
woody
```

通过-server或-client设置jvm的运行参数

​	JVM在启动的时候会根据硬件和操作系统自动选择使用Server还是Client类型的JVM

​		32位操作系统

​			如果是Windows系统，不论硬件配置如何，都默认使用Client类型的JVM

​			如果是其他操作系统，机器配置有2GB以上的内存同时有2个以上CPU的话，默认使用server模式，否则使用client模式

​		64位操作系统

​			只有server类型，不支持client类型

​	Server VM的初始堆空间会大一些，默认使用并行垃圾回收器，启动慢运行块

​		java -server -showversion TestVM

​	Client VM的初始堆空间会小一些，默认使用串行的垃圾回收器，启动块运行慢

## -X参数（非标准参数）

在不同版本的jvm中，参数可能会有所不同，可以通过java -X查看非标准参数

-Xint

​	在解释模式（interpreted mode）下，-Xint标记会强制JVM执行所有的字节码，当然这会降低运行速度，通常低10倍或更多

​	java -showversion -Xint TestVM

-Xcomp

​	与-Xint正好相反，JVM在第一次使用时会把所有的字节码编译成本地代码，从而带来最大程度的优化

​	java -showversion -xcomp TestVM

-Xmixed

​	将解释模式与编译模式进行混合使用，由jvm自己决定，这是jvm默认的模式，也是推荐使用的模式

​	java -showversion TestVM

![image-20210216225752600](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216225752600.png)

## -XX参数（使用率较高）

-XX:newSize、-XX:+UseSerialGC

也是非标准参数，主要用于jvm的调优和debug操作

使用有2种方式

​	boolean类型

​		格式：-XX:[+-]

​		如：-XX:+DisableExplicitGC表示禁用手动调用gc操作，也就是说调用System.gc()无效

​		java -showversion -XX:+DisableExplicitGC TestVM

​	非boolean类型

​		格式：-XX

​		如：-XX:NewRatio=1表示新生代和老年代的比值

​		-Xmx2048m：等价于-XX:MaxHeapSize，设置JVM最大堆内存为2048M

​		-Xms512m：等价于-XX:InitialHeapSize，设置JVM初始堆内存为512M

查看jvm的运行参数

​	运行java命令时打印参数：需要添加-XX:+PrintFlagsFinal参数即可

​		java -XX:+PrintFlagsFinal -version

​	查看正在运行的jvm参数：需要借助jinfo命令查看

​		jps     // pid

​		jinfo -flags pid

​		jinfo -flag ConcGCThreads pid

# jmap的使用以及内存溢出分析

查看内存使用情况 jmap -heap pid

![image-20210216234554182](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216234554182.png)

![image-20210216234510165](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216234510165.png)

查看内存中对象数量及大小 

​	查看所有对象，包括活跃以及非活跃的 jmap -histo <pid> | more

​	查看活跃对象 jmap -histo:live <pid> | more

![image-20210216234849713](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216234849713.png)

将内存使用情况dump到文件中

​	有些时候需要将jvm当前内存中的情况dump到文件中，然后对它进行分析，jmap也支持dump到文件中（b二进制）

​	用法：jmap -dump:format=b,file=dumpFileName <pid>

​	示例：jmap -dump:format=b,file=/tmp/dump.dat 11005

通过jhat对dump文件进行分析

​	jhat -port 8888 /tmp/dump.dat

![image-20210216235529582](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216235529582.png)

​	查看防火墙是否关闭

​		firewall-cmd --zone=public --add-port=8888/tcp --permanent

​		firewall-cmd --reload

​	访问：http://ip:8888

​	select s from java.lang.String s where s.value.length>=1000

![image-20210216235850614](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216235850614.png)

## MAT工具

![image-20210217000053456](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217000053456.png)

# jvm内存模型

jdk1.7的堆内存模型

![image-20210216232702484](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216232702484.png)

jdk1.8的堆内存模型

![image-20210216233212358](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216233212358.png)

通过jstat命令进行查看堆内存使用情况

​	jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：	

​		`jstat [-命令选项][vmid][间隔时间/毫秒][查询次数]`

​	查看class加载统计

![image-20210216233838481](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216233838481.png)

​	查看编译统计信息

![image-20210216233938036](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216233938036.png)

​	查看垃圾回收统计

![image-20210216234045190](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210216234045190.png)

# 实战：内存溢出的定位与分析

模拟内存溢出

​	编写代码，向List集合中添加100万个字符串，每个字符串由1000个UUID组成。如果程序能够正常执行，最后打印ok（idea编辑器中需要设置内存溢出相关参数）

​	VM options：-Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError

```java
public class TestOOM {
  public static void main(String[] args) {
    ArrayList<String> stringArrayList = new ArrayList<>();
    for(int i=0; i<1000000; i++) {
      String str = "";
      for(int j=0; j<1000; j++) {
        str = str + UUID.randomUUID().toString();
      }
      stringArrayList.add(str);
    }
    System.out.println("it is over!!");
  }
}
```

生成java_pid12344.hprof文件

使用MAT工具分析hprof文件

# jstack的使用

线程的状态

![image-20210217001223820](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217001223820.png)

实战：死锁问题

![image-20210217001410831](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217001410831.png)

使用jstack命令查看死锁结果并分析

jps -l

jstack pid

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217002118096.png" alt="image-20210217002118096" style="zoom:50%;" />

# jvisualvm的使用

![image-20210217002140384](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217002140384.png)

catalina.sh

![image-20210217002850060](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217002850060.png)

![image-20210217002922544](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217002922544.png)

## JMX

![image-20210217003042652](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217003042652.png)

## 监控远程的tomcat

![image-20210217003119902](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217003119902.png)

![image-20210217003229722](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217003229722.png)

