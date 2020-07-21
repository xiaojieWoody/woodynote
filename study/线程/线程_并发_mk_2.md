# ==video：10-4 10-5  11-2==

## 6. 线程的各个属性

![image-20200719092409073](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719092409073.png)

* 线程id

  ```java
  /**
   * 描述：     ID从1开始，JVM运行起来后，我们自己创建的线程的ID早已不是2.
   */
  public class Id {
  
      public static void main(String[] args) {
          Thread thread = new Thread();
          System.out.println("主线程的ID"+Thread.currentThread().getId());
          System.out.println("子线程的ID"+thread.getId());
      }
  }
  ```

* 线程名字

* 守护线程：

  * 给用户线程提供服务
  * 3个特性：
    * 线程类型默认继承自父线程
    * 被谁启动：JVM自动启动
    * 不影响JVM退出（守护线程一直都在，即使JVM退出）
  * 和普通线程的唯一区别：是否影响JVM的退出（用户线程是执行我们的业务逻辑，守护线程是服务我们的）
  * 是否需要给线程设置为守护线程？
    * JVM发现只剩下守护线程后，JVM就会关闭，可能强行终止任务，这比较危险

* 线程优先级

  * 10个级别，默认5
  * 程序设计不应依赖于优先级
    * 不同操作系统不一样
    * 优先级会被操作系统改变

  ![image-20200719093838738](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719093838738.png)

* 什么时候我们需要设置守护线程
  * 不需要设置，JVM提供的够用
* 我们应该如何应用线程优先级来帮助程序运行？有哪些禁忌
  * 不应该使用优先级
* 不同的操作系统如何处理优先级问题
  * 优先级可能会被操作系统忽略

##7.  未捕获异常如何处理？

* 线程的未捕获异常UncauthtException应该如何处理？

  * 为什么需要UncauthtExceptionHandler？

    * 主线程可以轻松发现异常，子线程却不行

      ```java
      /**
       * 描述：     单线程，抛出，处理，有异常堆栈 多线程，子线程发生异常，会有什么不同？
       */
      public class ExceptionInChildThread implements Runnable {
      
          public static void main(String[] args) {
              new Thread(new ExceptionInChildThread()).start();
              for (int i = 0; i < 1000; i++) {
                  System.out.println(i);
              }
          }
      
          @Override
          public void run() {
              throw new RuntimeException();
          }
      }
      ```

    * 子线程异常无法用传统的方法捕获

      ```java
      /**
       * 描述： 1. 不加try catch抛出4个异常，都带线程名字 2. 加了try catch,期望捕获到第一个线程的异常，线程234不应该运行，希望看到打印出Caught Exception
       * 3. 执行时发现，根本没有Caught Exception，线程234依然运行并且抛出异常
       *
       * 说明线程的异常不能用传统方法捕获
       */
      public class CantCatchDirectly implements Runnable {
      
          public static void main(String[] args) throws InterruptedException {
              // 捕获的是主线程的异常，此时子线程已启动
              try {
                  new Thread(new CantCatchDirectly(), "MyThread-1").start();
                  Thread.sleep(300);
                  new Thread(new CantCatchDirectly(), "MyThread-2").start();
                  Thread.sleep(300);
                  new Thread(new CantCatchDirectly(), "MyThread-3").start();
                  Thread.sleep(300);
                  new Thread(new CantCatchDirectly(), "MyThread-4").start();
              } catch (RuntimeException e) {
                  System.out.println("Caught Exception.");
              }
      
          }
      
          @Override
          public void run() {
              try {
                  throw new RuntimeException();
              } catch (RuntimeException e) {
                  System.out.println("Caught Exception.");
              }
          }
      }
      ```

    * 不能直接捕获的后果，如何提高健壮性

      * 全局捕获

      * 两种解决方法

      * 方法一（不推荐）：手动在每个run方法里进行try catch

      * 方法二（推荐）

        * 利用UncaughtExceptionHandler接口
          * void uncaughtException(Thread t, Throwable e);
        * 给程序统一设置
        * 给每个线程单独设置
        * 给线程池设置

        ```java
        import java.util.logging.Level;
        import java.util.logging.Logger;
        
        /**
         * 描述：     自己的MyUncaughtExceptionHanlder
         */
        public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
        
            private String name;
        
            public MyUncaughtExceptionHandler(String name) {
                this.name = name;
            }
        
            @Override
            public void uncaughtException(Thread t, Throwable e) {
                Logger logger = Logger.getAnonymousLogger();
                logger.log(Level.WARNING, "线程异常，终止啦" + t.getName());
                System.out.println(name + "捕获了异常" + t.getName() + "异常");
            }
        }
        ```

        ```java
        /**
         * 描述：     使用刚才自己写的UncaughtExceptionHandler
         */
        public class UseOwnUncaughtExceptionHandler implements Runnable {
        
            public static void main(String[] args) throws InterruptedException {
                Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler("捕获器1"));
        
                new Thread(new UseOwnUncaughtExceptionHandler(), "MyThread-1").start();
                Thread.sleep(300);
                new Thread(new UseOwnUncaughtExceptionHandler(), "MyThread-2").start();
                Thread.sleep(300);
                new Thread(new UseOwnUncaughtExceptionHandler(), "MyThread-3").start();
                Thread.sleep(300);
                new Thread(new UseOwnUncaughtExceptionHandler(), "MyThread-4").start();
            }
          
            @Override
            public void run() {
                throw new RuntimeException();
            }
        }
        ```

* 面试问题
  * Java异常体系
  * 如何全局处理异常？为什么要全局处理？不处理行不行
  * run方法是否可以抛出异常？如果抛出异常，线程的状态会怎样？
    * 会终止，打出堆栈信息
  * 线程中如何处理某个未处理异常

## 8. 双刃剑：多线程会导致的问题

### 线程安全

* 不管业务中遇到怎样的多个线程访问某对象或某方法的情况，而在编程这个业务逻辑的时候，都不需要额外做任何额外的处理（也就是可以像单线程编程一样），程序也可以正常运行（不会因为多线程而出错），就可以称为线程安全

* 线程不安全

  * get同时set
  * 额外同步

* 全都线程安全

  * 运行速度、设计成本、trade off

* 完全不用于多线程：不过度设计

* 主要是两个问题：

  * 数据争用：数据读写由于同时写，会造成错误数据
  * 竞争条件：即使不是同时写造成的错误数据，由于顺序原因依然会造成错误，例如在写入前就读取了

* 什么情况下会出现线程安全问题？怎么避免？

  * 运行结果错误（a++多线程下出现消失的请求现象，属于read-modify-write）

    * 原子性
    * 找到a++出错的地方

  * 死锁等活跃性问题（包括死锁、活锁、饥饿）

  * 对象发布和初始化的时候的安全问题

    * 什么是发布

      * 声明为public
      * return一个对象
      * 把对象作为参数传递到其他类的方法中

    * 什么是逸出

      * 1. 方法返回一个private对象
           * 解决方法：返回“副本”

      * 2. 还未完成初始化（构造函数没完全执行完毕）就把对象提供给外界

           * 在构造函数中未初始化完毕就this赋值

           * 隐式逸出——注册监听事件

           * 构造函数中运行线程

    * 如何解决逸出

      * 副本
      * 工厂模式

  * 总结归纳：各种需要考虑线程安全的情况

    * 访问共享的变量或资源，会有并发风险，比如对象的属性、静态变量、共享缓存、数据库等
    * 所有依赖时序的操作，即使每一步操作都是线程安全的，还是存在并发问题
      * read-modify-write操作：一个线程读取了一个共享数据，并在此基础上更新该数据
      * check-then-act操作：一个线程读取了一个共享数据，并在此基础上决定其下一个的操作
    * 不同的数据之间存在捆绑关系的时候
      * IP和端口号
    * 我们使用其他类的时候，如果对方没有声明自己是线程安全的，那么大概率会存在并发问题
      * 比如HashMap没有声明自己是并发安全的，所以并发调用出错

![image-20200719103946781](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719103946781.png)

* ==综合：多看几遍==

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 描述：     第一种：运行结果出错。 演示计数不准确（减少），找出具体出错的位置。
 */
public class MultiThreadsError implements Runnable {

    static MultiThreadsError instance = new MultiThreadsError();
    int index = 0;
    static AtomicInteger realIndex = new AtomicInteger();
    static AtomicInteger wrongCount = new AtomicInteger();
    static volatile CyclicBarrier cyclicBarrier1 = new CyclicBarrier(2);
    static volatile CyclicBarrier cyclicBarrier2 = new CyclicBarrier(2);

    final boolean[] marked = new boolean[10000000];

    public static void main(String[] args) throws InterruptedException {

        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面上结果是" + instance.index);
        System.out.println("真正运行的次数" + realIndex.get());
        System.out.println("错误次数" + wrongCount.get());

    }

    @Override
    public void run() {
        marked[0] = true;
        for (int i = 0; i < 10000; i++) {
            try {
                cyclicBarrier2.reset();
                cyclicBarrier1.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            index++;
            try {
                cyclicBarrier1.reset();
                cyclicBarrier2.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            realIndex.incrementAndGet();
            synchronized (instance) {
                if (marked[index] && marked[index - 1]) {
                    System.out.println("发生错误" + index);
                    wrongCount.incrementAndGet();
                }
                marked[index] = true;
            }
        }
    }
}
```

```java
/**
 * 描述：     第二章线程安全问题，演示死锁。
 */
public class MultiThreadError implements Runnable {

    int flag = 1;
    static Object o1 = new Object();
    static Object o2 = new Object();

    public static void main(String[] args) {
        MultiThreadError r1 = new MultiThreadError();
        MultiThreadError r2 = new MultiThreadError();
        r1.flag = 1;
        r2.flag = 0;
        new Thread(r1).start();
        new Thread(r2).start();
    }

    @Override
    public void run() {
        System.out.println("flag = " + flag);
        if (flag == 1) {
            synchronized (o1) {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2) {
                    System.out.println("1");
                }
            }
        }
        if (flag == 0) {
            synchronized (o2) {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o1) {
                    System.out.println("0");
                }
            }
        }
    }
}
```

```java
import com.sun.javafx.geom.Matrix3f;
import java.util.HashMap;
import java.util.Map;

/**
 * 描述：     发布逸出
 */
public class MultiThreadsError3 {

    private Map<String, String> states;

    public MultiThreadsError3() {
        states = new HashMap<>();
        states.put("1", "周一");
        states.put("2", "周二");
        states.put("3", "周三");
        states.put("4", "周四");
    }

    public Map<String, String> getStates() {
        return states;
    }

    // 返回副本
    public Map<String, String> getStatesImproved() {
        return new HashMap<>(states);
    }

    public static void main(String[] args) {
        MultiThreadsError3 multiThreadsError3 = new MultiThreadsError3();
        Map<String, String> states = multiThreadsError3.getStates();
//        System.out.println(states.get("1"));
//        states.remove("1");
//        System.out.println(states.get("1"));

        System.out.println(multiThreadsError3.getStatesImproved().get("1"));
        multiThreadsError3.getStatesImproved().remove("1");
        System.out.println(multiThreadsError3.getStatesImproved().get("1"));

    }
} 
```

```java
/**
 * 描述：     初始化未完毕，就this赋值
 */
public class MultiThreadsError4 {

    static Point point;

    public static void main(String[] args) throws InterruptedException {
        new PointMaker().start();
//        Thread.sleep(10);
        Thread.sleep(105);
        if (point != null) {
            System.out.println(point);
        }
    }
}

class Point {

    private final int x, y;

    public Point(int x, int y) throws InterruptedException {
        this.x = x;
        MultiThreadsError4.point = this;
        Thread.sleep(100);
        this.y = y;
    }

    @Override
    public String toString() {
        return x + "," + y;
    }
}

class PointMaker extends Thread {

    @Override
    public void run() {
        try {
            new Point(1, 1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
/**
 * 描述：     观察者模式
 */
public class MultiThreadsError5 {

    int count;

    public MultiThreadsError5(MySource source) {
        source.registerListener(new EventListener() {
            @Override
            public void onEvent(Event e) {
                System.out.println("\n我得到的数字是" + count);
            }

        });
        for (int i = 0; i < 10000; i++) {
            System.out.print(i);
        }
        count = 100;
    }

    public static void main(String[] args) {
        MySource mySource = new MySource();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                mySource.eventCome(new Event() {
                });
            }
        }).start();
        MultiThreadsError5 multiThreadsError5 = new MultiThreadsError5(mySource);
    }

    static class MySource {

        private EventListener listener;

        void registerListener(EventListener eventListener) {
            this.listener = eventListener;
        }

        void eventCome(Event e) {
            if (listener != null) {
                listener.onEvent(e);
            } else {
                System.out.println("还未初始化完毕");
            }
        }

    }

    interface EventListener {

        void onEvent(Event e);
    }

    interface Event {

    }
}
```

```java
/**
 * 描述：     构造函数中新建线程
 */
public class MultiThreadsError6 {

    private Map<String, String> states;

    public MultiThreadsError6() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                states = new HashMap<>();
                states.put("1", "周一");
                states.put("2", "周二");
                states.put("3", "周三");
                states.put("4", "周四");
            }
        }).start();
    }

    public Map<String, String> getStates() {
        return states;
    }

    public static void main(String[] args) throws InterruptedException {
        MultiThreadsError6 multiThreadsError6 = new MultiThreadsError6();
        Thread.sleep(1000);
        System.out.println(multiThreadsError6.getStates().get("1"));
    }
}
```

```java
/**
 * 描述：     用工厂模式修复刚才的初始化问题
 */
public class MultiThreadsError7 {

    int count;
    private EventListener listener;

    private MultiThreadsError7(MySource source) {
        listener = new EventListener() {
            @Override
            public void onEvent(MultiThreadsError5.Event e) {
                System.out.println("\n我得到的数字是" + count);
            }

        };
        for (int i = 0; i < 10000; i++) {
            System.out.print(i);
        }
        count = 100;
    }

    public static MultiThreadsError7 getInstance(MySource source) {
        MultiThreadsError7 safeListener = new MultiThreadsError7(source);
        // 初始化完成之后，再注册上去，保证了安全性
        source.registerListener(safeListener.listener);
        return safeListener;
    }

    public static void main(String[] args) {
        MySource mySource = new MySource();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                mySource.eventCome(new MultiThreadsError5.Event() {
                });
            }
        }).start();
        MultiThreadsError7 multiThreadsError7 = new MultiThreadsError7(mySource);
    }

    static class MySource {

        private EventListener listener;

        void registerListener(EventListener eventListener) {
            this.listener = eventListener;
        }

        void eventCome(MultiThreadsError5.Event e) {
            if (listener != null) {
                listener.onEvent(e);
            } else {
                System.out.println("还未初始化完毕");
            }
        }

    }

    interface EventListener {

        void onEvent(MultiThreadsError5.Event e);
    }

    interface Event {

    }
}
```

### 性能问题有哪些体现、什么是性能问题

* 服务响应慢、吞度量低、资源消耗（例如内存）过高等
* 虽然不是结果错误，但依然危害巨大
* 引入多线程不能本末倒置

### 为什么多线程会带来性能问题

* 调度：上下文切换
  * 什么是上下文切换
    * 什么是上下文？保存现场
    * 上下文切换可以认为是内核（操作系统的核心）在CPU上对于进程（包括线程）进行以下活动：
      1. 挂起一个进程，将这个进程在CPU中的状态（上下文）存储于内存中的某处
      2. 在内存中检索下一个进程的上下文并将其在CPU的寄存器中恢复
      3. 跳转到程序计数器所指向的位置（即跳转到进程被中断时的代码行），以恢复该进程
  * 缓存开销
    * 缓存失效，CPU重新缓存
  * 何时会导致密集的上下文切换
    * 频繁地竞争锁，或者由于IO读写等原因导致频繁阻塞
* 协作：内存同步
  * Java内存模型
    * 为了数据的正确性，同步手段往往会使用禁止编译器优化、使CPU内的缓存失效

### 常见面试问题

* 你知道有哪些线程不安全的情况？
* 平时有哪些情况下需要额外注意线程安全问题？
* 什么是多线程的上下文切换？

### 考点

* 一共有哪几类线程安全问题？
  * 3类
* 哪些场景需要额外注意线程安全问题？
* 什么是多线程带来的上下文切换？

# 常见面试问题

* 有多少种实现线程的方法？思路有5点
* 实现Runnable接口和继承Thread类，哪种方式更好
* 一个线程两次调用start()方法会出现什么情况？为什么
* 既然start()方法会调用run()方法，为什么我们选择调用start()方法，而不是直接调用run()方法呢
* 如何停止线程
* 如何处理不可中断的阻塞
* 线程有哪几种状态？生命周期是什么
* 用程序实现两个线程交替打印0～100的奇偶数
* 手写生产者消费者设计模式
* 为什么wait()需要在同步代码块内使用，而sleep()不需要
* 为什么线程通信的方法wait()，notify()和notifyAll()被定义在Object类里？而sleep定义在Thread类里
* wait方法是属于Object对象的，那调用Thread.wait会怎样
* 如何选择用notify还是notifyAll
* notifyAll之后所有的线程都会再次抢夺锁，如果某线程抢夺失败怎么办
* 用suspend()和resume()来阻塞线程可以吗？为什么
* wati/notify、sleep异同(方法属于哪个对象？线程状态怎么切换？)
* 在join期间，线程处于哪种线程状态
* 守护线程和普通线程的区别
* 我们是否需要给线程设置为守护线程
* run方法是否可以抛出异常？如果抛出异常，线程的状态会怎么样
* 线程中如何处理某个未处理的异常
* 什么是多线程的上下文切换