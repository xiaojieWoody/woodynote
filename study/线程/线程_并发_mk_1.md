# ==Video：2-1，5-15==

# 线程8大核心基础

* [思维导图](https://naotu.baidu.com/file/07f437ff6bc3fa7939e171b00f133e17?token=6744a1c6ca6860a0)

## 1. 实现多线程的方法到底有1种还是2种还是4种？

* 2种

  * 实现Runnable接口(更好)

    * 解耦（具体的任务-run方法种的内容，和线程的生命周期，解耦）、避免新建线程的损耗（传入runnable实例，重复使用线程）、可扩展（Java不支持多继承）
    * 本质：最终调用target.run();
  * 继承Thread类

    * 本质：run()整个都被重写

  * 通常可以分为两类，[Oracle也是这么说的](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html)
  * 准确的讲，创建线程只有一种方式，那就是构造Thread类，而实现线程的执行单元有两种方式
    * 方法一：实现Runnable接口的run方法，并把Runnable实例传给Thread类
    * 方法二：继承Thread类，重写Thread的run方法

  ```java
  /**
   * 描述：     用Runnable方式创建线程
   */
  public class RunnableStyle implements Runnable{
  
      public static void main(String[] args) {
          Thread thread = new Thread(new RunnableStyle());
          thread.start();
      }
  
      @Override
      public void run() {
          System.out.println("用Runnable方法实现线程");
      }
  }
  
  /**
   * 描述：     用Thread方式实现线程
   */
  public class ThreadStyle extends Thread{
  
      @Override
      public void run() {
          System.out.println("用Thread类实现线程");
      }
  
      public static void main(String[] args) {
          new ThreadStyle().start();
      }
  }
  ```

  ```java
  /**
   * 描述：     同时使用Runnable和Thread两种实现线程的方式
   */
  public class BothRunnableThread {
  
      public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println("我来自Runnable");
              }
          }) {
              // 重写
              @Override
              public void run() {
                  System.out.println("我来自Thread");
              }
          }.start();
      }
  }
  // 结果
  // 我来自Thread
  ```

* 线程池也是通过new Thread来创建线程

* Callable和FutureTask创建线程，本质离不开runnable接口

  ![image-20200713215442086](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200713215442086.png)

  ![image-20200713215518218](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200713215518218.png)

  

  

## 2. 怎样才是正确的线程启动方式？

* 不能重复start()，start()源码解析
  * 启动新线程检查线程状态（不为0则抛出异常）
  * 加入线程组
  * 调用start0()，native方法（C++实现）
* run()，是一个普通方法，不会启动一个线程
* start()，告诉JVM合适的时候启动新线程

## 3. 如何正确停止线程？（难点）

* 原理：使用interrupt来通知，而不是强制

* 最佳实践：如何正确停止线程

  * 通常的停止过程（无外界干涉的情况下）

    * run执行完毕
    * 未捕获的异常

  * 正确方法：用interrupt来请求停止线程

    * 普通情况（run方法内没有sleep或wait方法时的标准写法）

    * 线程可能被阻塞

    * 如果线程在每次工作迭代之后都阻塞（调用sleep方法等）

    * 如果不这样写，会遇到的问题：线程无法停止

    * 实际生产开发时要注意的编码习惯

      * 两种最佳处理方式
        * 优先选择：传递中断
        * 不想或无法传递：恢复中断
        * 不应屏蔽中断

    * 可以为了响应中断而抛出InterruptedException的常见方法列表总结

      * 可以感知中断信号，响应中断

      ![image-20200714073032328](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200714073032328.png)

      ![image-20200714073103130](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200714073103130.png)

  * 好处

    * 避免线程、数据的戛然而止

* 错误的停止方法

  * 被弃用的stop、suspend和resume方法（带着锁休息，容易死锁）
  * 用volatile设置boolean标志位

* 停止线程相关的重要函数解析

  * 中断线程
    * interrupt方法原理：native interrupt0()，C++代码实现
  * 判断是否已被中断
    * static boolean interrupted()
    * boolean isInterrupted()
    * 举例说明，注意Thread.interrupted()的目的对象是“当前线程”，而不管本方法是来自于哪个对象

* 常见面试问题

  * 如何停止一个线程
    1. 原理：用interrupt来请求，好处
    2. 想停止线程，要请求方、被停止方、子方法被调用方相互配合
    3. 错误方法：stop/suspend已废弃，volatile的boolean无法处理长时间阻塞的情况
  * 如何处理不可中断的阻塞（例如抢锁时ReentrantLock.lock()或Socket I/O时无法响应中断，那应该怎么让该线程停止呢？）

    * 针对某些锁或IO，做出特定方法，特定情况使用特定方法，没有通用方法

![image-20200714073302324](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200714073302324.png)

```java
// 普通方法停止线程
/**
 * 描述：     run方法内没有sleep或wait方法时，停止线程
 */
public class RightWayStopThreadWithoutSleep implements Runnable {

    @Override
    public void run() {
        int num = 0;
        while (!Thread.currentThread().isInterrupted() && num <= Integer.MAX_VALUE / 2) {
            if (num % 10000 == 0) {
                System.out.println(num + "是10000的倍数");
            }
            num++;
        }
        System.out.println("任务运行结束了");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadWithoutSleep());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

```java
// 线程可能被阻塞
/**
 * 描述：     带有sleep的中断线程的写法
 */
public class RightWayStopThreadWithSleep {

    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int num = 0;
            try {
                while (num <= 300 && !Thread.currentThread().isInterrupted()) {
                    if (num % 100 == 0) {
                        System.out.println(num + "是100的倍数");
                    }
                    num++;
                }
                // 休眠时会抛出InterruptedException异常
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(500);
        thread.interrupt();
    }
}
```

```java
/**
 * 描述：     如果在执行过程中，每次循环都会调用sleep或wait等方法，那么不需要每次迭代都检查是否已中断
 */
public class RightWayStopThreadWithSleepEveryLoop {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int num = 0;
            try {
                // 不再需要加入isinterrupted()来检查是否中断
                while (num <= 10000) {
                    if (num % 100 == 0) {
                        System.out.println(num + "是100的倍数");
                    }
                    num++;
                    // 休眠时会抛出InterruptedException异常
                    Thread.sleep(10);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }
}
```

```java
/**
 * 描述：     如果while里面放try/catch，会导致中断失效
 */
public class CantInterrupt {

    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int num = 0;
            // sleep会将interrupt标记位清除，所以没效果
            while (num <= 10000 && !Thread.currentThread().isInterrupted()) {
                if (num % 100 == 0) {
                    System.out.println(num + "是100的倍数");
                }
                num++;
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }
}
```

```java
/**
 * 描述：     最佳实践：catch了InterruptedExcetion之后的优先选择：在方法签名中抛出异常 那么在run()就会强制try/catch
 */
public class RightWayStopThreadInProd implements Runnable {

    @Override
    public void run() {
        while (true && !Thread.currentThread().isInterrupted()) {
            System.out.println("go");
            // 2.捕获异常
            try {
                throwInMethod();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                //保存日志、停止程序
                System.out.println("保存日志");
                e.printStackTrace();
            }
        }
    }
    
    // 1.向上抛出异常
    private void throwInMethod() throws InterruptedException {
            Thread.sleep(2000);
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

```java
/**
 * 描述：最佳实践2：在catch子语句中调用Thread.currentThread().interrupt()来恢复设置中断状态，以便于在后续的执行中，依然能够检查到刚才发生了中断
 * 回到刚才RightWayStopThreadInProd补上中断，让它跳出
 */
public class RightWayStopThreadInProd2 implements Runnable {

    @Override
    public void run() {
        while (true) {
            // 2.检查中断
            if (Thread.currentThread().isInterrupted()) {
                System.out.println("Interrupted，程序运行结束");
                break;
            }
            reInterrupt();
        }
    }

    private void reInterrupt() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            // 1.设置中断
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd2());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

```java
/**
 * 描述：     错误的停止方法：用stop()来停止线程，会导致线程运行一半突然停止，没办法完成一个基本单位的操作（一个连队），会造成脏数据（有的连队多领取少领取装备）。
 */
public class StopThread implements Runnable {

    @Override
    public void run() {
        //模拟指挥军队：一共有5个连队，每个连队10人，以连队为单位，发放武器弹药，叫到号的士兵前去领取
        for (int i = 0; i < 5; i++) {
            System.out.println("连队" + i + "开始领取武器");
            for (int j = 0; j < 10; j++) {
                System.out.println(j);
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("连队"+i+"已经领取完毕");
        }
    }

    public static void main(String[] args) {
        Thread thread = new Thread(new StopThread());
        thread.start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread.stop();
    }
}
```

![image-20200714074001119](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200714074001119.png)

```java
/**
 * 描述：     演示用volatile的局限：part1 看似可行
 */
public class WrongWayVolatile implements Runnable {

    private volatile boolean canceled = false;

    @Override
    public void run() {
        int num = 0;
        try {
            while (num <= 100000 && !canceled) {
                if (num % 100 == 0) {
                    System.out.println(num + "是100的倍数。");
                }
                num++;
                Thread.sleep(1);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        WrongWayVolatile r = new WrongWayVolatile();
        Thread thread = new Thread(r);
        thread.start();
        Thread.sleep(5000);
        r.canceled = true;
    }
}
```

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ConcurrentLinkedQueue;

/**
 * 描述：     演示用volatile的局限part2 陷入阻塞时，volatile是无法线程的 此例中，生产者的生产速度很快，消费者消费速度慢，所以阻塞队列满了以后，生产者会阻塞，等待消费者进一步消费
 */
public class WrongWayVolatileCantStop {

    public static void main(String[] args) throws InterruptedException {
        ArrayBlockingQueue storage = new ArrayBlockingQueue(10);

        Producer producer = new Producer(storage);
        Thread producerThread = new Thread(producer);
        producerThread.start();
        Thread.sleep(1000);

        Consumer consumer = new Consumer(storage);
        while (consumer.needMoreNums()) {
            System.out.println(consumer.storage.take()+"被消费了");
            Thread.sleep(100);
        }
        System.out.println("消费者不需要更多数据了。");

        //一旦消费不需要更多数据了，我们应该让生产者也停下来，但是实际情况线程没有停止
        producer.canceled=true;
        System.out.println(producer.canceled);
    }
}

class Producer implements Runnable {

    public volatile boolean canceled = false;
    BlockingQueue storage;
    public Producer(BlockingQueue storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        int num = 0;
        try {
            while (num <= 100000 && !canceled) {
                if (num % 100 == 0) {
                    // 阻塞，无法中断线程
                    storage.put(num);
                    System.out.println(num + "是100的倍数,被放到仓库中了。");
                }
                num++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("生产者结束运行");
        }
    }
}

class Consumer {

    BlockingQueue storage;
    public Consumer(BlockingQueue storage) {
        this.storage = storage;
    }

    public boolean needMoreNums() {
        if (Math.random() > 0.95) {
            return false;
        }
        return true;
    }
}
```

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

/**
 * 描述：     用中断来修复刚才的无尽等待问题
 */
public class WrongWayVolatileFixed {

    public static void main(String[] args) throws InterruptedException {
        WrongWayVolatileFixed body = new WrongWayVolatileFixed();
        ArrayBlockingQueue storage = new ArrayBlockingQueue(10);

        Producer producer = body.new Producer(storage);
        Thread producerThread = new Thread(producer);
        producerThread.start();
        Thread.sleep(1000);

        Consumer consumer = body.new Consumer(storage);
        while (consumer.needMoreNums()) {
            System.out.println(consumer.storage.take() + "被消费了");
            Thread.sleep(100);
        }
        System.out.println("消费者不需要更多数据了。");
        // 1.设置中断
        producerThread.interrupt();
    }

    class Producer implements Runnable {
        BlockingQueue storage;
        public Producer(BlockingQueue storage) {
            this.storage = storage;
        }

        @Override
        public void run() {
            int num = 0;
            try {
              // 2.检查中断
                while (num <= 100000 && !Thread.currentThread().isInterrupted()) {
                    if (num % 100 == 0) {
                        storage.put(num);
                        System.out.println(num + "是100的倍数,被放到仓库中了。");
                    }
                    num++;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println("生产者结束运行");
            }
        }
    }

    class Consumer {
        BlockingQueue storage;
        public Consumer(BlockingQueue storage) {
            this.storage = storage;
        }

        public boolean needMoreNums() {
            if (Math.random() > 0.95) {
                return false;
            }
            return true;
        }
    }
}
```

```java
/**
 * 描述：     注意Thread.interrupted()方法的目标对象是“当前线程”，而不管本方法来自于哪个对象
 */
public class RightWayInterrupted {

    public static void main(String[] args) throws InterruptedException {

        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                for (; ; ) {
                }
            }
        });

        // 启动线程
        threadOne.start();
        //设置中断标志
        threadOne.interrupt();
        //获取中断标志  true
        System.out.println("isInterrupted: " + threadOne.isInterrupted());
        //获取中断标志并重置 false
        System.out.println("isInterrupted: " + threadOne.interrupted());
        //获取中断标志并重直 false
        System.out.println("isInterrupted: " + Thread.interrupted());
        //获取中断标志 true
        System.out.println("isInterrupted: " + threadOne.isInterrupted());
        threadOne.join();
        System.out.println("Main thread is over.");
    }
}
```

## 4. 线程的一生-6个状态（生命周期）（难点）

* 有哪6种状态

* ![image-20200718110851026](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200718110851026.png)

  ![image-20200718100104034](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200718100104034.png)

  ```java
  package threadcoreknowledge.sixstates;
  
  /**
   * 描述：     展示线程的NEW、RUNNABLE、Terminated状态。即使是正在运行，也是Runnable状态，而不是Running。
   */
  public class NewRunnableTerminated implements Runnable {
  
      public static void main(String[] args) {
          Thread thread = new Thread(new NewRunnableTerminated());
          //打印出NEW的状态
          System.out.println(thread.getState());
          thread.start();
        	// RUNNABLE
          System.out.println(thread.getState());
          try {
              Thread.sleep(10);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          //打印出RUNNABLE的状态，即使是正在运行，也是RUNNABLE，而不是RUNNING
          System.out.println(thread.getState());
          try {
              Thread.sleep(100);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          //打印出TERMINATED状态
          System.out.println(thread.getState());
      }
  
      @Override
      public void run() {
          for (int i = 0; i < 1000; i++) {
              System.out.println(i);
          }
      }
  }
  ```

  ```java
  package threadcoreknowledge.sixstates;
  
  /**
   * 描述：     展示Blocked, Waiting, TimedWaiting
   */
  public class BlockedWaitingTimedWaiting implements Runnable{
      public static void main(String[] args) {
          BlockedWaitingTimedWaiting runnable = new BlockedWaitingTimedWaiting();
          Thread thread1 = new Thread(runnable);
          thread1.start();
          Thread thread2 = new Thread(runnable);
          thread2.start();
          try {
              Thread.sleep(5);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          //打印出Timed_Waiting状态，因为正在执行Thread.sleep(1000);
          System.out.println(thread1.getState());
          //打印出BLOCKED状态，因为thread2想拿得到sync()的锁却拿不到
          System.out.println(thread2.getState());
          try {
              Thread.sleep(1300);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          //打印出WAITING状态，因为执行了wait()
          System.out.println(thread1.getState());
  
      }
  
      @Override
      public void run() {
          syn();
      }
  
      private synchronized void syn() {
          try {
              Thread.sleep(1000);
              wait();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```

* 阻塞状态是什么

  * 一般习惯而言，把Blocked（被阻塞）、Waiting（等待）、Timed_waiting（计时等待）都被称为阻塞状态
  * 不仅仅是Blocked

* 常见面试问题

  * 线程有哪几种状态
  *  生命周期是什么

## 5. Thread和Object类中的重要方法详解

![image-20200718103411367](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200718103411367.png)

* wait、notify、notifyAll

  * 作用、用法：阻塞阶段、唤醒阶段、遇到中断

  * 性质

    * 必须先拥有monitor
    * 只能唤醒其中一个
    * 属于Object类
    * 类似功能的Condition
    * 同时持有多个锁的情况

  * 阻塞阶段，直到以下4种情况之一发生时，才会被唤醒

    * 另一个线程调用这个对象的notify()方法且刚好被唤醒的是本线程
    * 另一个线程调用这个对象的notifyAll()方法
    * 过了wait(long timeout)规定的超时时间，如果传入0就是永久等待
    * 线程自身调用了interrupt()

  * 唤醒阶段

    * notify，唤醒任意一个等待的线程
    * notifyAll，唤醒所有等待线程

  * 遇到中断

    * interruptExcept，释放monitor

    ![image-20200718110714377](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200718110714377.png)

    ![image-20200718110812790](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200718110812790.png)

    ```java
    package threadcoreknowledge.threadobjectclasscommonmethods;
    
    /**
     * 描述：     展示wait和notify的基本用法 1. 研究代码执行顺序 2. 证明wait释放锁
     */
    public class Wait {
    
        public static Object object = new Object();
    
        static class Thread1 extends Thread {
    
            @Override
            public void run() {
                synchronized (object) {
                    System.out.println(Thread.currentThread().getName() + "开始执行了");
                    try {
                        object.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程" + Thread.currentThread().getName() + "获取到了锁。");
                }
            }
        }
    
        static class Thread2 extends Thread {
    
            @Override
            public void run() {
                synchronized (object) {
                    object.notify();
                    // synchronize内执行完后唤醒其他线程
                    System.out.println("线程" + Thread.currentThread().getName() + "调用了notify()");
                }
            }
        }
    
        public static void main(String[] args) throws InterruptedException {
            Thread1 thread1 = new Thread1();
            Thread2 thread2 = new Thread2();
            thread1.start();
            Thread.sleep(200);
            thread2.start();
          // Thread-0开始执行了
          // 线程Thread-1调用了notify()
          // 线程Thread-0获取到了锁
        }
    }
    ```

    ```java
    package threadcoreknowledge.threadobjectclasscommonmethods;
    
    /**
     * 描述：     3个线程，线程1和线程2首先被阻塞，线程3唤醒它们。notify, notifyAll。 start先执行不代表线程先启动。
     */
    public class WaitNotifyAll implements Runnable {
    
        private static final Object resourceA = new Object();
    
    
        public static void main(String[] args) throws InterruptedException {
            Runnable r = new WaitNotifyAll();
            Thread threadA = new Thread(r);
            Thread threadB = new Thread(r);
            Thread threadC = new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized (resourceA) {
                        resourceA.notifyAll();
    //                    resourceA.notify();
                        System.out.println("ThreadC notified.");
                    }
                }
            });
            threadA.start();
            threadB.start();
            Thread.sleep(200);
            threadC.start();
        }
        @Override
        public void run() {
            synchronized (resourceA) {
                System.out.println(Thread.currentThread().getName()+" got resourceA lock.");
                try {
                    System.out.println(Thread.currentThread().getName()+" waits to start.");
                    resourceA.wait();
                    System.out.println(Thread.currentThread().getName()+"'s waiting to end.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    // Thread-0 got resourceA lock.
    // Trhead-0 waits to start.
    // Thread-1 got resourceA lock.
    // Thread-1 waits to start.
    // ThreadC norified.
    // Thread-1's waiting to end.
    // Thread-0's waiting to end.
    ```

    ```java
    package threadcoreknowledge.threadobjectclasscommonmethods;
    
    /**
     * 描述：     证明wait只释放当前的那把锁
     */
    public class WaitNotifyReleaseOwnMonitor {
    
        private static volatile Object resourceA = new Object();
        private static volatile Object resourceB = new Object();
    
        public static void main(String[] args) {
            Thread thread1 = new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized (resourceA) {
                        System.out.println("ThreadA got resourceA lock.");
                        synchronized (resourceB) {
                            System.out.println("ThreadA got resourceB lock.");
                            try {
                                System.out.println("ThreadA releases resourceA lock.");
                                resourceA.wait();
    
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            });
    
            Thread thread2 = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (resourceA) {
                        System.out.println("ThreadB got resourceA lock.");
                        System.out.println("ThreadB tries to resourceB lock.");
    
                        synchronized (resourceB) {
                            System.out.println("ThreadB got resourceB lock.");
                        }
                    }
                }
            });
    
            thread1.start();
            thread2.start();
        }
    }
    // ThreadA got resourceA lock.
    // ThreadA got resourceB lock.
    // ThreadA releases resourceA lock.
    // ThreadB got resourceA lock.
    // ThreadB tries to resourceB lock.
    ```

* sleep

  * 让线程进入Waiting状态，并且不占用CPU资源，但是不释放锁（synchronized和lock），直到规定时间后再执行，休眠期间如果被中断，会抛出异常并清除中断状态

  * 响应中断

    * 抛出InterruptedException
    * 清除中断状态

    ```java
    import sun.awt.windows.ThemeReader;
    
    /**
     * 展示线程sleep的时候不释放synchronized的monitor，等sleep时间到了以后，正常结束后才释放锁
     */
    public class SleepDontReleaseMonitor implements Runnable {
    
        public static void main(String[] args) {
            SleepDontReleaseMonitor sleepDontReleaseMonitor = new SleepDontReleaseMonitor();
            new Thread(sleepDontReleaseMonitor).start();
            new Thread(sleepDontReleaseMonitor).start();
        }
    
        @Override
        public void run() {
            syn();
        }
    
        private synchronized void syn() {
            System.out.println("线程" + Thread.currentThread().getName() + "获取到了monitor。");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程" + Thread.currentThread().getName() + "退出了同步代码块");
        }
    }
    ```

    ```java
    import java.util.concurrent.locks.Lock;
    import java.util.concurrent.locks.ReentrantLock;
    
    /**
     * 描述：     演示sleep不释放lock（lock需要手动释放）
     */
    public class SleepDontReleaseLock implements Runnable {
    
        private static final Lock lock = new ReentrantLock();
    
        @Override
        public void run() {
            lock.lock();
            System.out.println("线程" + Thread.currentThread().getName() + "获取到了锁");
            try {
                Thread.sleep(5000);
                System.out.println("线程" + Thread.currentThread().getName() + "已经苏醒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    
        public static void main(String[] args) {
            SleepDontReleaseLock sleepDontReleaseLock = new SleepDontReleaseLock();
            new Thread(sleepDontReleaseLock).start();
            new Thread(sleepDontReleaseLock).start();
        }
    }
    ```

    ```java
    import java.util.Date;
    import java.util.concurrent.TimeUnit;
    
    /**
     * 描述：     每个1秒钟输出当前时间，被中断，观察。
     * Thread.sleep()
     * TimeUnit.SECONDS.sleep()          (推荐)
     */ 
    public class SleepInterrupted implements Runnable{
    
        public static void main(String[] args) throws InterruptedException {
            Thread thread = new Thread(new SleepInterrupted());
            thread.start();
            Thread.sleep(6500);
            thread.interrupt();
        }
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                System.out.println(new Date());
                try {
                    TimeUnit.HOURS.sleep(3);
                    TimeUnit.MINUTES.sleep(25);
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    System.out.println("我被中断了！");
                    e.printStackTrace();
                }
            }
        }
    }
    ```

* join

  * 因为新的线程加入了我们，所以我们要等他执行完再出发

  * main等待thread1执行完毕，注意谁等谁

  * 在join期间，线程到底是什么状态？Waiting

  * CountDownLatch或CyclicBarrier类

    ```java
    /**
     * 描述：     演示join，注意语句输出顺序，会变化。
     */
    public class Join {
        public static void main(String[] args) throws InterruptedException {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "执行完毕");
                }
            });
            Thread thread2 = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "执行完毕");
                }
            });
    
            thread.start();
            thread2.start();
            System.out.println("开始等待子线程运行完毕");
            thread.join();
            thread2.join();
            System.out.println("所有子线程执行完毕");
        }
    }
    ```

    ```java
    /**
     * 描述：     演示join期间被中断的效果
     */
    public class JoinInterrupt {
        public static void main(String[] args) {
            Thread mainThread = Thread.currentThread();
            Thread thread1 = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        mainThread.interrupt();
                        Thread.sleep(5000);
                        System.out.println("Thread1 finished.");
                    } catch (InterruptedException e) {
                        System.out.println("子线程中断");
                    }
                }
            });
            thread1.start();
            System.out.println("等待子线程运行完毕");
            try {
                thread1.join();
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName()+"主线程中断了");
                thread1.interrupt();
            }
            System.out.println("子线程已运行完毕");
        }
    
    }
    ```

    ```java
    /**
     * 描述：     先join再mainThread.getState()
     * 通过debugger看线程join前后状态的对比
     */
    public class JoinThreadState {
        public static void main(String[] args) throws InterruptedException {
            Thread mainThread = Thread.currentThread();
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(3000);
                        System.out.println(mainThread.getState());
                        System.out.println("Thread-0运行结束");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
            thread.start();
            System.out.println("等待子线程运行完毕");
            thread.join();
            System.out.println("子线程运行完毕");
    
        }
    }
    ```

    ```java
    package threadcoreknowledge.threadobjectclasscommonmethods;
    
    /**
     * 描述：     通过讲解join原理，分析出join的代替写法
     */
    public class JoinPrinciple {
    
        public static void main(String[] args) throws InterruptedException {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "执行完毕");
                }
            });
    
            thread.start();
            System.out.println("开始等待子线程运行完毕");
            thread.join();
            // join等价于下面三行
    //        synchronized (thread) {
    //            thread.wait();
    //        }
            System.out.println("所有子线程执行完毕");
        }
    }
    ```

* yield

  * 释放我的CPU时间片
  * JVM不保证遵循
  * 和sleep区别：是否随时可能再次被调度

* 获取当前执行线程的引用：Thread.currentThread()方法

* start和run方法

* stop，suspend，resume方法

* 面试常见问题
  * 两个线程交替打印0-100的奇偶数

    * synchronized

      ```java
      package threadcoreknowledge.threadobjectclasscommonmethods;
      
      /**
       * 描述：     两个线程交替打印0~100的奇偶数，用synchronized关键字实现
       */
      public class WaitNotifyPrintOddEvenSyn {
      
          private static int count;
      
          private static final Object lock = new Object();
      
          //新建2个线程
          //1个只处理偶数，第二个只处理奇数（用位运算）
          //用synchronized来通信
          public static void main(String[] args) {
              new Thread(new Runnable() {
                  @Override
                  public void run() {
                      while (count < 100) {
                          synchronized (lock) {
                              if ((count & 1) == 0) {
                                  System.out.println(Thread.currentThread().getName() + ":" + count++);
                              }
                          }
                      }
                  }
              }, "偶数").start();
      
              new Thread(new Runnable() {
                  @Override
                  public void run() {
                      while (count < 100) {
                          synchronized (lock) {
                              if ((count & 1) == 1) {
                                  System.out.println(Thread.currentThread().getName() + ":" + count++);
                              }
                          }
                      }
                  }
              }, "奇数").start();
          }
      }
      ```

    * 更好的方法：wait/notify

      ```java
      package threadcoreknowledge.threadobjectclasscommonmethods;
      
      
      /**
       * 描述：     两个线程交替打印0~100的奇偶数，用wait和notify
       */
      public class WaitNotifyPrintOddEveWait {
      
          private static int count = 0;
          private static final Object lock = new Object();
      
      
          public static void main(String[] args) {
              new Thread(new TurningRunner(), "偶数").start();
              new Thread(new TurningRunner(), "奇数").start();
          }
      
          //1. 拿到锁，我们就打印
          //2. 打印完，唤醒其他线程，自己就休眠
          static class TurningRunner implements Runnable {
      
              @Override
              public void run() {
                  while (count <= 100) {
                      synchronized (lock) {
                          //拿到锁就打印
                          System.out.println(Thread.currentThread().getName() + ":" + count++);
                          lock.notify();
                          if (count <= 100) {
                              try {
                                  //如果任务还没结束，就让出当前的锁，并休眠
                                  lock.wait();
                              } catch (InterruptedException e) {
                                  e.printStackTrace();
                              }
                          }
                      }
                  }
              }
          }
      }
      ```

  * 手写生产者消费者设计模式

  * 为什么wait()需要在同步代码块内使用，而sleep()不需要
    * 线程间相互配合，为了通信可靠，防止死锁或永久等待的发生，否则，没有synchronized的保护，可能notify都执行完了再切换回来执行wait
    * Sleep是针对单独线程，和其他线程关联不大

* wait/notify、sleep异同

  * 相同：阻塞、响应中断
  * 不同：
    * 同步方法中
    * 释放锁
    * 指定时间
    * 所属类

* 为什么线程通信的方法wait()，notify()和notifyAll()被定义在Object类里？而sleep定义在Thread类里

  * wait()，notify()和notifyAll()是操作锁，而锁是属于某一个对象的，而不是线程
  * 某个线程可能持有多个锁并且相互配合，没办法实现灵活逻辑
  * 调用Thread.wait会怎样？
    * 特殊，线程退出时会自动执行notify，会影响设计的执行流程

* notify还是notifyAll

  * 是想唤醒多个线程还是一个

* notifyAll之后所有的线程都会再次抢夺锁，如果某线程抢夺失败怎么办

  * 会陷入等待状态，等待这把锁释放 

* 用3种方式实现生产者模式

  * 解耦

    ![image-20200718111116512](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200718111116512.png)

    ```java
    package threadcoreknowledge.threadobjectclasscommonmethods;
    
    import java.util.ArrayList;
    import java.util.Date;
    import java.util.LinkedList;
    import java.util.List;
    
    /**
     * 描述：     用wait/notify来实现生产者消费者模式
     */
    public class ProducerConsumerModel {
        public static void main(String[] args) {
            EventStorage eventStorage = new EventStorage();
            Producer producer = new Producer(eventStorage);
            Consumer consumer = new Consumer(eventStorage);
            new Thread(producer).start();
            new Thread(consumer).start();
        }
    }
    
    class Producer implements Runnable {
    
        private EventStorage storage;
    
        public Producer(
                EventStorage storage) {
            this.storage = storage;
        }
    
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                storage.put();
            }
        }
    }
    
    class Consumer implements Runnable {
    
        private EventStorage storage;
    
        public Consumer(
                EventStorage storage) {
            this.storage = storage;
        }
    
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                storage.take();
            }
        }
    }
    
    class EventStorage {
    
        private int maxSize;
        private LinkedList<Date> storage;
    
        public EventStorage() {
            maxSize = 10;
            storage = new LinkedList<>();
        }
    
        public synchronized void put() {
            while (storage.size() == maxSize) {
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            storage.add(new Date());
            System.out.println("仓库里有了" + storage.size() + "个产品。");
            notify();
        }
    
        public synchronized void take() {
            while (storage.size() == 0) {
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("拿到了" + storage.poll() + "，现在仓库还剩下" + storage.size());
            notify();
        }
    }
    ```

* Join和sleep和wait期间线程的状态分别是什么？为什么？



