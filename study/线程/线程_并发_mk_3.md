# ==video：13 -15==

# Java内存模型-底层原理

## JVM内存结构、Java内存模型、Java对象模型

* JVM内存结构，和Java虚拟机的运行时区域有关

  * JVM实现会带来不同的“翻译”，不同的CPU平台的机器指令又千差万别，无法保证并发安全的效果一致

    1. 最开始，我们编写的Java代码，是*.java文件

    2. 在编译（javac命令）后，从刚才的*.java文件会变出一个新的Java字节码文件（*.class）

    3. JVM会执行刚才生成的字节码文件（*.class），并把字节码文件转化为机器指令

    4. 机器指令可以直接在CPU上执运行，也就是最终的程序执行

  ![image-20200719143117917](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719143117917.png)

  * 堆（所有线程共享）：
    * 存储的全部是对象实例（通过new等指令创建的，并会被垃圾回收；数组也是保存在堆上面的，即使是基本类型的数据，也是保存在堆中的。因为在Java中，数组是对象），是内存中最大的一块
    * 堆的优势是可以在运行时动态地分配内存空间，不必事先告诉编译器
  * 虚拟机栈每个线程私有）：
    * 每个线程包含一个栈区，栈中保存基础数据类型（byte，short，int，long，float，double，boolean，char）的对象、自定义对象的引用(不是对象本身)和returnAddress（指向了一条字节码指令的地址）。
    * 每个方法从被调用到执行完的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。每个栈中的数据(基本类型的局部变量、参数、对象引用)都是私有的，其他栈不能访问。
    * 在编译期间就确定了大小，运行期间不会改变大小
  * 方法区（所有线程共享）：
    * 它用于存储虚拟机已经加载的static静态变量、类信息、常量
    * 还存放永久引用，比如static People p = new People();的p引用
    * 运行时常量池
      * 是方法区的一部分，Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译器生成的各种符号引用，这部分内容将在类加载后放到方法区的运行时常量池中。存放final修饰的
  * 本地方法栈（每个线程私有）：
    * 与虚拟机栈基本类似，区别在于虚拟机栈为虚拟机执行的java方法服务，而本地方法栈则是为Native方法服务
  * 程序计数器（每个线程私有）：
    * 是最小的一块内存区域，它的作用是当前线程所执行的字节码的行号指示器，在虚拟机的模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、异常处理、线程恢复等基础功能都需要依赖计数器完成

* Java内存模型，和Java的并发编程有关

* Java对象模型，和Java对象在虚拟机中的表现形式有关

  ![image-20200719143816077](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719143816077.png)

  * Java对象自身的存储模型
  * JVM会给这个类创建一个instanceKlass，保存在方法区，用来在JVM层表示该Java类
  * 当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对象中包含了对象头以及实例数据

## JMM是什么

* 为什么需要JMM
  * C语言不存在内存模型的概念
  * 依赖处理器，不同处理器结果不一样
  * 无法保证并发安全
  * 需要一个标准，让多线程运行的结果可预期
* 是规范
  * Java Memory Model是一组规范，需要各个JVM的实现来遵守JMM规范，以便于开发者可以利用这些规范，更方便地开发多线程程序。
  * 如果没有这样的一个JMM内存模型来规范，那么很可能经过了不同JVM的不同规则的重排序之后，导致不同的虚拟机上运行的结果不一样，那是很大的问题。
* 是工具类和关键字的原理
  * volatile、synchronized、Lock等的原理都是JMM
  * 如果没有JMM，那就需要我们自己指定什么时候用内存栅栏等，那是相当麻烦的，幸好有了JMM，让我们只需要用同步工具和关键字就可以开发并发程序。
* 最重要的3点内容
  * 重排序、内存可见性、原子性

## 重排序

* 实际执行顺序和代码在java文件中的顺序不一致

* 好处：提高处理速度

  ![image-20200719155151225](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719155151225.png)

* 重排序的3种情况

  * 编译器优化
    * 包括JVM、JIT编译器等
  * 指令重排序
    * CPU 的优化行为，和编译器优化很类似，是通过乱序执行的技术，来提高执行效率。所以就算编译器不发生重排，CPU 也可能对指令进行重排，所以我们开发中，一定要考虑到重排序带来的后果
  * 内存的“重排序”
    * 线程A的修改线程B却看不到，引出可见性问题
    * 内存系统内不存在重排序，但是内存会带来看上去和重排序一样的效果，所以这里的“重排序”打了双引号。由于内存有缓存的存在，在JMM里表现为主存和本地内存，由于主存和本地内存的不一致，会使得程序表现出乱序的行为。在刚才的例子中，假设没编译器重排和指令重排，但是如果发生了内存缓存不一致，也可能导致同样的情况：线程1 修改了 a 的值，但是修改后并没有写回主存，所以线程2是看不到刚才线程1对a的修改的，所以线程2看到a还是等于0。同理，线程2对b的赋值操作也可能由于没及时写回主存，导致线程1看不到刚才线程2的修改

  ```java
  import java.util.concurrent.CountDownLatch;
  
  /**
   * 描述：     演示重排序的现象 “直到达到某个条件才停止”，测试小概率事件
   */
  public class OutOfOrderExecution {
  
      private static int x = 0, y = 0;
      private static int a = 0, b = 0;
  
      public static void main(String[] args) throws InterruptedException {
          int i = 0;
          for (; ; ) {
              i++;
              x = 0;
              y = 0;
              a = 0;
              b = 0;
  
              CountDownLatch latch = new CountDownLatch(3);
  
              Thread one = new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          latch.countDown();
                          latch.await();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      a = 1;
                      x = b;
                  }
              });
              Thread two = new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          latch.countDown();
                          latch.await();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      b = 1;
                      y = a;
                  }
              });
              two.start();
              one.start();
              latch.countDown();
              one.join();
              two.join();
  
              String result = "第" + i + "次（" + x + "," + y + ")";
              if (x == 0 && y == 0) {
                  System.out.println(result);
                  break;
              } else {
                  System.out.println(result);
              }
          }
      }
  }
  ```

  * 一共有3种情况
    * a = 1; x= b(0); b=1; y=a(1)，最终结果是 x=0, y=1
    * b = 1; y= a(0); a=1; x=b(1)，最终结果是 x=1, y=0
    * b = 1; a = 1; x=b(1); y=a(1)，最终结果是 x=1, y=1
  * 重排序分析
    * 会出现x = 0, y = 0？那是因为重排序发生了，4行代码的执行顺序的其中一种可能：
    * y = a; a = 1; x = b; b = 1;
  * 什么是重排序
    * 在线程1内部的两行代码的实际执行顺序和代码在Java文件中的顺序不一致，代码指令并不是严格按照代码语句顺序执行的，它们的顺序被改变了，这就是重排序，这里被颠倒的是y = a 和 b = 1这两行语句

## 可见性

* 为什么会有可见性问题

  * CPU有多级缓存，导致读的数据过期

    * 高速缓存的容量比主内存小，但是速度仅次于寄存器，所以在CPU和主内存之间就多了Cache层
    * 线程间的对于共享变量的可见性问题不是直接由多核引起的，而是由多缓存引起的。
    * 如果所有个核心都只用一个缓存，那么也就不存在内存可见性问题了。
    * 每个核心都会将自己需要的数据读到独占缓存中，数据修改后也是写入到缓存中，然后等待刷入到主存中。所以会导致有些核心读取的值是一个过期的值

    ![image-20200719160423102](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719160423102.png)

* JMM的抽象：主内存和本地内存

  * 什么是主内存和本地内存

    * Java 作为高级语言，屏蔽了CPU cache等底层细节，用 JMM 定义了一套读写内存数据的规范，虽然我们不再需要关心一级缓存和二级缓存的问题，但是，JMM 抽象了主内存和本地内存的概念
      * 这里说的本地内存并不是真的一块给每个线程分配的内存，而是JMM的一个抽象，是对于寄存器、一级缓存、二级缓存等的抽象

    ![image-20200719160914927](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719160914927.png)

    ![image-20200719161106463](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719161106463.png)

  * JMM规定主内存和本地内存的关系

    1. 所有的变量都存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量内容是主内存中的拷贝

    2.	线程不能直接读写主内存中的变量,而是只能操作自己工作内存中的变量，然后再同步到主内存中
    3.	主内存是多个线程共享的，但线程间不共享工作内存,如果线程间需要通信，必须借助主内存中转来完成

  * 所有的共享变量存在于主内存中，每个线程有自己的本地内存，而且线程读写共享数据也是通过本地内存交换的，所以才导致了可见性问题

* happens-before原则

  * 什么是happens-before

    * 解决可见性问题的：在时间上，动作A发生在动作B之前，B保证能看见A，这就是happens-before

  * 什么不是happens-before

    * 两个线程没有相互配合的机制，所以代码X和代码Y的执行结果并不能保证总被对方看到的，这就不具备happens-before

  * 影响JVM重排序

    * 如果两个操作不具备happens-before，那么JVM是可以根据需要自由排序的，但是如果具备happens-before（比如新建线程时，run方法里面的语句一定发生在thread.start()之前），那么JVM也不能改变它们之间的顺序。

  * Happens-Before规则有哪些？

    * 如果我们分别有操作 x 和操作 y，我们用 hb(x, y) 来表示 x happens-before y。

    * 单线程规则

    * **锁操作**（synchronized和Lock）

      * 在解锁之前的所有操作，对于加锁之后的所有操作都是可见的

    * **volatile变量**

      * 只要写入了，那么读取的一定是最新结果

    * 线程启动

      * 子线程启动后能看到主线程之前的操作

    * 线程join

      * join的线程能看到之前线程的操作

    * 传递性

      * 如果hb(A, B) 而且hb(B, C)，那么可以推出hb(A, C)

    * 中断

      * 一个线程被其他线程interrupt时，那么检测中断（isInterrupted）或者抛出InterruptedException一定能看到

    * 构造方法

      * 对象构造方法的最后一行指令 happens-before 于 finalize() 方法的第一行指令

    * 工具类的Happens-Before原则

      1.	线程安全的容器get一定能看到在此之前的put等存入动作
      2.	CountDownLatch
      3.	Semaphore
      4.	Future
      5.	线程池
      6.	CyclicBarrier

    * 代码案例：happens-before演示

      * happens-before有一个原则是：如果A是对volatile变量的写操作，B是对同一个变量的读操作，那么hb(A, B)

      * 近朱者赤：给b加了volatile，不仅b被影响，也可以实现轻量级同步

        ![image-20200719163659343](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719163659343.png)

      * b之前的写入（对应代码b = a）对读取b后的代码（print b）都可见，所以在writerThread里对a的赋值，一定会对readerThread里的读取可见，所以这里的a即使不加volatile，只要b读到是3，就可以由happends-before原则保证了读取到的都是3而不可能读取到1

      * 没给x加volatile，那么有可能出现x=1, a=0。因为a虽然被修改了，但是其他线程不可见，而x恰好其他线程可见，这就造成了x=1, a=0

      * 加了volatile的x，可以作为同步机制，保证a b c的可见性

* volatile关键字

  * volatile是什么

    * volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用volatile并不会发生上下文切换等开销很大的行为
    * 如果一个变量别修饰成volatile，那么JVM就知道了这个变量可能会被并发修改
    * 但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的原子保护，volatile仅在很有限的场景下才能发挥作用

  * volatile的适用场合

    * 不适用组合操作：a++

    * 适用场合1：boolean flag

      * 如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全

    * 适用场合2：作为刷新之前变量的触发器

      * 用了volatile int x后，可以保证读取x后，之前的所有变量都可见

        ![image-20200719164849236](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719164849236.png)

      * 一个实际生产中的例子

    * volatile的作用：两点

      * 第一层：可见性
        * 读一个 volatile 变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值，写一个 volatile 属性会立即刷入到主内存。
      * 第二层：禁止指令重排序优化
        * 解决单例双重锁乱序问题

    * volatile和synchronized的关系？

      * volatile在这方面可以看做是轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全

    * 学以致用：用volatile修正重排序问题
      * OutOfOrderExecution类加了volatile后，用于不会出现(0, 0)的情况了

  * volatile 小结

    1. volatile 修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag；或者作为触发器，实现轻量级同步
    2. volatile 属性的读写操作都是无锁的，它不能替代 synchronized，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的。
    3. volatile 只能作用于属性，我们用 volatile 修饰属性，这样 compilers 就不会对这个属性做指令重排序。
    4. volatile 提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile 属性不会被线程缓存，始终从主存中读取。
    5. volatile 提供了 happens-before 保证，对 volatile 变量 v 的写入 happens-before 所有其他线程后续对 v 的读操作。
    6. volatile 可以使得 long 和 double 的赋值是原子的，后面马上会讲long 和 double的原子性。

* 能保证可见性的措施

  * 除了volatile可以让变量保证可见性外，synchronized、Lock、并发集合、Thread.join()和Thread.start()等都可以保证一定的可见性，具体看happens-before原则的规定

* 升华：对synchronized可见性的正确理解

  * synchronized不仅保证了原子性，还保证了可见性

    ![image-20200719165839104](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719165839104.png)

  * synchronized不仅让被保护的代码安全，还近朱者赤

  * synchronized也可以达到同样的happens-before效果

  * 这里关于synchronized有一个特别值得说的点，我们之前可能一致认为，使用了synchronized之后，synchronized会帮我们设立临界区，这样在一个线程操作数据的时候，另一个线程无法进来同时操作，所以保证了线程安全。其实这是不全面的，这种说法没有考虑到可见性问题。真正完整的说法是

  * synchronized不仅防止了一个线程在操作某对象时收到其他线程的干扰，同时还保证了修改好之后，可以立即被其他线程所看到。（因为如果其他线程看不到，那也会有线程安全问题）

```java
/**
 * 描述：     演示可见性带来的问题
 */
public class FieldVisibility {

    // 解决可见性问题
    volatile int a = 1;
    volatile int b = 2;

    private void change() {
        a = 3;
        b = a;
    }

    private void print() {
        System.out.println("b=" + b + ";a=" + a);
    }

    public static void main(String[] args) {
        while (true) {
            FieldVisibility test = new FieldVisibility();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.change();
                }
            }).start();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.print();
                }
            }).start();
        }
    }
}
```

* 4种情况

  * a = 3, b = 2

  * a = 1, b = 2

  * a = 3, b = 3

  * a = 1, b = 3  （可见性问题导致）

    ![image-20200719160202579](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719160202579.png)

    ![image-20200719160347095](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719160347095.png)

```java
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 描述：     不适用于volatile的场景
 */
public class NoVolatile implements Runnable {

    volatile int a;
    AtomicInteger realA = new AtomicInteger();

    public static void main(String[] args) throws InterruptedException {
        Runnable r =  new NoVolatile();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(((NoVolatile) r).a);
        System.out.println(((NoVolatile) r).realA.get());
    }
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            a++;
            realA.incrementAndGet();
        }
    }
}
```

```java
import java.util.concurrent.atomic.AtomicInteger;
import singleton.Singleton8;

/**
 * 描述：     volatile适用的情况1
 */
public class UseVolatile1 implements Runnable {

    volatile boolean done = false;
    AtomicInteger realA = new AtomicInteger();

    public static void main(String[] args) throws InterruptedException {
        Runnable r =  new UseVolatile1();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(((UseVolatile1) r).done);
        System.out.println(((UseVolatile1) r).realA.get());
    }
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            setDone();
            realA.incrementAndGet();
        }
    }

    private void setDone() {
        done = true;
    }
}
```

```java
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 描述：     volatile不适用的情况2
 */
public class NoVolatile2 implements Runnable {

    volatile boolean done = false;
    AtomicInteger realA = new AtomicInteger();

    public static void main(String[] args) throws InterruptedException {
        Runnable r =  new NoVolatile2();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(((NoVolatile2) r).done);
        System.out.println(((NoVolatile2) r).realA.get());
    }
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            flipDone();
            realA.incrementAndGet();
        }
    }

    private void flipDone() {
        done = !done;
    }
}
```

## 原子性

* 什么是原子性

  * 一系列的操作，要么全部执行成功，要么全部不执行，不会出现执行一半的情况，是不可分割的。
  * ATM里取钱
  * i++不是原子性的
  * 用synchronized实现原子性

* Java中的原子操作有哪些？

  * 1）除long和double之外的基本类型（int, byte, boolean, short, char, float）的赋值操作
  * 2）所有引用reference的赋值操作，不管是 32 位的机器还是 64 位的机器
  * 3）java.concurrent.Atomic.* 包中所有类的原子操作

* long 和 double 的原子性

  * 问题描述

    * 官方文档

      ![image-20200719170603339](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719170603339.png)

    * 对于 64 位的值的写入，可以分为两个 32 位的操作进行写入。读取错误

    * 使用 volatile 

  * 结论

    * 在32位上的JVM上，long 和 double的操作不是原子的，但是在64位的JVM上是原子的。

  * 实际开发中

    * 商用Java虚拟机中不会出现

* 原子操作+原子操作 != 原子操作

  * 全同步的HashMap也不完全安全
  * 去ATM机两次取钱是两次独立的原子操作，但是期间有可能银行卡被借给女朋友，也就是被其他线程打断并被修改

## 面试题

* JMM应用实例：单例模式8种写法、单例和并发的关系

  * 单例模式的作用

    * 节省内存和计算
    * 保证结果正确
    * 方便管理

  * 单例模式适用场景

    ![image-20200719171319893](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719171319893.png)

  * 8种单例模式

    ```java
    /**
     * 描述：     饿汉式（静态常量）（可用）
     */
    public class Singleton1 {
        private final static Singleton1 INSTANCE = new Singleton1();
        private Singleton1() {}
        public static Singleton1 getInstance() {
            return INSTANCE;
        }
    }
    ```

    ```java
    /**
     * 描述：     饿汉式（静态代码块）（可用）
     */
    public class Singleton2 {
        private final static Singleton2 INSTANCE;
        static {
            INSTANCE = new Singleton2();
        }
        private Singleton2() {}
        public static Singleton2 getInstance() {
            return INSTANCE;
        }
    }
    ```

    ```java
    /**
     * 描述：     懒汉式（线程不安全）
     */
    public class Singleton3 {
        private static Singleton3 instance;
        private Singleton3() {}
        public static Singleton3 getInstance() {
            if (instance == null) {
                instance = new Singleton3();
            }
            return instance;
        }
    }
    ```

    ```java
    /**
     * 描述：     懒汉式（线程安全）（效率低，不推荐）
     */
    public class Singleton4 {
        private static Singleton4 instance;
        private Singleton4() {}
        // 性能低
        public synchronized static Singleton4 getInstance() {
            if (instance == null) {
                instance = new Singleton4();
            }
            return instance;
        }
    }
    ```

    ```java
    /**
     * 描述：     懒汉式（线程不安全）（不推荐）
     */
    public class Singleton5 {
        private static Singleton5 instance;
        private Singleton5() {}
        public static Singleton5 getInstance() {
           // 两个线程同时到达
            if (instance == null) {
                // 实例化两次
                synchronized (Singleton5.class) {
                    instance = new Singleton5();
                }
            }
            return instance;
        }
    }
    ```

    ```java
    /**
     * 描述：     双重检查（推荐面试使用）
       线程安全；延迟加载；效率较高
     */
    public class Singleton6 {
        // 新建对象实际上有3个步骤：创建一个空对象、调用构造方法、把实例地址分配给刚才的引用，但是顺序不能保证
        // 重排序会带来NPE
        // 防止重排序
        private volatile static Singleton6 instance;
        private Singleton6() {}
        public static Singleton6 getInstance() {
            if (instance == null) {
                synchronized (Singleton6.class) {
                    if (instance == null) {
                        instance = new Singleton6();
                    }
                }
            }
            return instance;
        }
    }
    ```

    ![image-20200719172310824](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719172310824.png)

    ```java
    /**
     * 描述：     静态内部类方式，可用
     */
    public class Singleton7 {
        private Singleton7() {}
        private static class SingletonInstance {
            private static final Singleton7 INSTANCE = new Singleton7();
        }
        public static Singleton7 getInstance() {
            return SingletonInstance.INSTANCE;
        }
    }
    ```

    ```java
    /**
     * 描述：     枚举单例
     */
    public enum Singleton8 {
        INSTANCE;
        public void whatever() {}
    }
    ```

  * 不同写法对比
    * 饿汉：简单，但是没有lazy loading
    * 懒汉：有线程安全问题
    * 静态内部类：可用
    * 双重检查：面试用
    * 枚举：最好

  ![image-20200719173021863](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719173021863.png)

![image-20200719173116521](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719173116521.png)



* JMM应用实例：单例模式8种写法、单例和并发的关系
* 讲一讲什么是Java内存模型
* volatile和synchronized的异同
* 什么是原子操作？Java中有哪些原子操作？生成对象的过程是不是原子操作？
  1. 新建一个空的Person对象
  2. 把这个对象的地址指向p
  3. 执行Person的构造函数
* 什么是内存可见性
* 64位的double和long写入的时候是原子的吗？