# ==Video：2-1==

# 并发工具类——分类

## 为了线程安全（从底层原理来分类）

* 互斥同步
  * 使用各种互斥同步的锁
    * synchronized
    * Lock接口的相关类
      * ReentrantLock
      * 读写锁
      * ...
  * 使用同步的工具类
    * Collections.synchronizedList(new ArrayList<E>())等
    * Vector等
* 非互斥同步
  * atomic包，原子类
    * Atomic基本类型原子类
      * AtomicInteger：整型原子类
      * AtomicLong：长整型原子类
      * AtomicBoolean ：布尔型原子类
    * AtomicArray数组类型原子类（数组里的元素，都可以保证原子性）
      * AtomicIntegerArray：整型数组原子类
      * AtomicLongArray：长整型数组原子类
      * AtomicReferenceArray ：引用类型数组原子类
    * AtomicReference引用类型原子类
      * AtomicReference：引用类型原子类
      * AtomicStampedReference：引用类型原子类的升级，带时间戳，可以解决ABA问题
      * AtomicMarkableReference
    * AtomicFieldUpdater升级原子类
      * AtomicIntegerFieldUpdater:原子更新整型字段的更新器
      * AtomicLongFieldUpdater：原子更新长整型字段的更新器
    * Adder加法器
      * LongAdder
      * DoubleAdder
    * Accumulator累加器
      * LongAccumulator
      * DoubleAccumulator
  * 用AtomicIntegerFieldUpdater等升级自己的变量
* 结合互斥和非互斥同步
  * 线程安全的并发容器
    * ConcurrentHashMap
    * CopyOnWriteArrayList
    * 并发队列
      * 阻塞队列
        * ArrayBlockingQueue
        * LinkedBlockingQueue
        * PriorityBlockingQueue
        * SynchronousQueue
        * DelayedQueue
        * TransferQueue
        * ...
      * 非阻塞队列
        * ConcurrentLinkedQueue
    * ConcurrentSkipListMap和ConcurrentSkipListSet
* 无同步方案、不可变
  * final关键字
  * 线程封闭
    * ThreadLocal
    * 栈封闭

## 为了线程安全（从使用者的角度来分类）

* 避免共享变量
  * 线程封闭
    * ThreadLocal
    * 栈封闭
* 共享变量，但是加以限制和处理
  * 互斥同步
    * 使用各种互斥同步的锁
      * synchronized
      * Lock接口的相关类
    * 使用同步的工具类
      * Collections.synchronizedList(new ArrayList<E>())等
      * Vector等
  * final关键字
* 使用成熟工具类
  * 线程安全的并发容器
    * ConcurrentHashMap
    * CopyOnWriteArrayLis
    * 并发队列
      * 阻塞队列
        * ArrayBlockingQueue
        * LinkedBlockingQueue
        * PriorityBlockingQueue
        * SynchronousQueue
        * DelayedQueue
        * TransferQueue
        * ...
      * 非阻塞队列
        * ConcurrentLinkedQueue
    * ConcurrentSkipListMap和ConcurrentSkipListSet
  * atomic包，原子类
    * Atomic基本类型原子类
      * AtomicInteger：整型原子类
      * AtomicLong：长整型原子类
      * AtomicBoolean ：布尔型原子类
    * AtomicArray数组类型原子类（数组里的元素，都可以保证原子性）
      * AtomicIntegerArray：整型数组原子类
      * AtomicLongArray：长整型数组原子类
      * AtomicReferenceArray ：引用类型数组原子类
    * AtomicReference引用类型原子类
      * AtomicReference：引用类型原子类
      * AtomicStampedReference：引用类型原子类的升级，带时间戳，可以解决ABA问题
      * AtomicMarkableReference
    * AtomicFieldUpdater升级原子类
      * AtomicIntegerFieldUpdater:原子更新整型字段的更新器
      * AtomicLongFieldUpdater：原子更新长整型字段的更新器
    * Adder加法器
      * LongAdder
      * DoubleAdder
    * Accumulator累加器
      * LongAccumulator
      * DoubleAccumulator

## 为了方便管理线程、提高效率

* 线程池相关类
  * Executor
  * Executors
  * ExecutorService
  * 常见线程池
    * FixedThreadPool
    * CachedThreadPool
    * ScheduledThreadPool
    * SingleThreadExecutor
    * ForkJoinPool
    * ...
  * 能获取子线程的运行结果
    * Callable
    * Future
    * FutureTask
    * CompletableFuture
    * ...

## 为了线程之间配合，来满足业务逻辑

* CountDownLatch
* CyclicBarrier
* Semaphore
* Condition
* Exchanger
* Phaser
* ...

# 线程池

## 线程池的自我介绍

* 为什么要使用线程池
  * 问题一：反复创建线程开销大
  * 问题二：过多的线程会占用太多内存
  * 解决以上两个问题的思路
    * 用少量的线程——避免内存占用过多
    * 让这部分线程都保持工作，且可以反复执行任务——避免生命周期的损耗
* 线程池好处
  * 加快响应速度，不需要反复创建、销毁
  * 合理利用CPU和内存
  * 统一管理
* 线程池适合应用的场合
  * 服务器接收到大量请求时，使用线程池技术是非常合适的，它可以大大减少线程的创建和销毁的次数，提高服务器的工作效率
  * 实际上，在开发中，如果需要创建5个以上的线程，那么就可以使用线程池来管理

## 创建和停止线程池

* 线程池构造函数的参数

  * 每个参数的含义概览

    * corePoolSize：int，核心线程数
    * maxPoolSize：int，最大线程数
    * keepAliveTime：long，保持存活时间
      * 如果线程池当前的线程数多于corePoolSize，那么如果多余的线程空闲时间超过keepAliveTime，它们就会被终止
    * workQueue：BlockingQueue，任务存储队列
    * threadFactory：ThreadFactory，当线程池需要新的线程的时候，会使用threadFactory来生成新的线程
    * Handler：RejectedExecutionHandler，由于线程池无法接受所提交的任务的拒绝策略

  * 参数中的corePoolSize和maxPoolSize有什么不同

    * 线程池在完成初始化后，默认情况下，线程池中并没有任何线程，线程池会等待有任务到来时，再创建新线程去执行
    * 线程池有可能会在核心线程数的基础上，额外增加一些线程，但是这些新增加的线程数有一个上限，这就是最大量maxPoolSize

  * 线程增加和减少以及task进入队列排队的规则

    1. 如果线程数小于corePoolSize，即使其他工作线程处于空闲状态，也会创建一个新线程来运行任务
    2. 如果线程数等于（或大于）corePoolSize但少于maximumPoolSize，则将任务放入队列
    3. 如果队列已满，并且线程数小于maxPoolSize，则创建一个新线程来运行任务
    4. 如果队列已满，并且线程数大于或等于maxPoolSize，则拒绝该任务

    ![image-20200721202254501](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721202254501.png)

  * 增减线程的特点

    * 通过设置corePoolSize和maximumPoolSize相同，就可以创建固定大小的线程池
    * 线程池希望保持较少的线程数，并且只有在负载变得很大时才增加它
    * 通过设置maximumPoolSize为很高的值，例如Integer.MAX_VALUE，可以允许线程池容纳任意数量的并发任务
    * 是只有在队列填满时才创建多于corePoolSize的线程，所以如果使用的是无界队列（例如LinkedBlockingQueue），那么线程数就不会超过corePoolSize

  * ThreadFactory（用来创建线程）

    * 新的线程是由ThreadFactory创建的，默认使用Executors.defaultThreadFactory()，创建出来的线程都在同一个线程组，拥有同样的NORM_PRIORITY优先级并且都不是守护线程。如果自己指定ThreadFactory，那么就可以改变线程名、线程组、优先级、是否是守护线程等
    * 通常用默认的ThreadFactory就可以了

  * BlockingQueue

    * 直接交接：SynchronousQueue
    * 无界队列：LinkedBlockingQueue
    * 有界的队列：ArrayBlockingQueue

* 线程池应该手动创建还是自动创建（阿里巴巴规约）

  * 手动创建更好，因为这样可以让我们更加明确线程池的运行规则，避免资源耗尽的风险
  * 自动创建线程池（直接调用JDK封装好的构造函数）可能带来哪些问题
    * newFixedThreadPool	
      * 由于传进去的LinkedBlockingQueue是没有容量上限的，所以当请求数越来越多，并且无法及时处理完毕的时候，也就是请求堆积的时候，会容易造成占用大量的内存，可能会导致OOM
    * newSingleThreadExecutor
      * 把线程数直接设置成了1，所以这也会导致同样的问题，也就是当请求堆积的时候，可能会占用大量的内存

  * 正确的创建线程池的方法
  * 根据不同的业务场景，自己设置线程池参数，比如内存有多大，想给线程取什么名字等

* 线程池里的线程数量设定为多少比较合适？

  * CPU密集型（加密、计算hash等）：最佳线程数为CPU核心数的1-2倍左右
  * 耗时IO型（读写数据库、文件、网络读写等）：最佳线程数一般会大于CPU的核心数很多倍，以JVM线程监控显示繁忙情况为依据，保证线程空闲可以衔接上
  * 线程数 = CPU核心数 * （1 + 平均等待时间 / 平均工作时间）

* 停止线程池的正确方法

  * shutdown：把存量任务执行完毕，不接收新的任务
  * isShutdown：线程池是否进入停止状态，即使任务还没执行完
  * isTerminated：所有任务是否执行完毕
  * awaitTermination：等待一段时间，检测任务是否执行完毕
  * shutdownNow：立刻关闭线程池，返回未执行的任务列表

## 常见线程池的特点和用法

* FixedThreadPool
* CachedThreadPool
  * 可缓存线程池
  * 特点：无界线程池，具有自动回收多余线程的功能
  * newCachedThreadPool 
  * 这里的弊端在于第二个参数maximumPoolSize被设置为了Integer.MAX_VALUE，这可能会创建数量非常多的线程，甚至导致OOM
* ScheduledThreadPool
* 支持定时及周期性任务执行的线程池
* SingleThreadExecutor		
  * 单线程的线程池：只会用唯一的工作线程来执行任务，此时的线程数量被设置为1

* 以上4种线程池的构造函数的参数

![image-20200721221302786](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200721221302786.png)

* 以上4种线程池对应的阻塞队列分析	
  * FixedThreadPool和SingleThreadExecutor的Queue是LinkedBlockingQueue
  * CachedThreadPool使用的Queue是SynchronousQueue
  * ScheduledThreadPool来说，它使用的是延迟队列DelayedWorkQueue	
* workStealingPool是JDK1.8加入的
* 子任务
  * 窃取

## 任务太多，怎么拒绝？

* 拒绝时机
  * 当Executor关闭时，提交新任务会被拒绝
  * 以及当Executor对最大线程和工作队列容量使用有限边界并且已经饱和时
* 4种拒绝策略

  * AbortPolicy：直接抛出异常
  * DiscardPolicy：丢弃任务
  * DiscardOldestPolicy：丢弃最老的任务
  * CallerRunsPolicy：提交任务的线程去执行

## 钩子方法，给线程池加点料

* 可以在每个任务执行之前和之后调用beforeExecute(Thread, Runnable)和 afterExecute(Runnable, Throwable)方法。这些可以用来操纵执行环境; 例如，重新初始化ThreadLocal，收集统计信息或添加日志条目。此外，可以重写terminated()方法，以执行Executor完全终止后需要执行的任何特殊处理

* 如果钩子或回调方法抛出异常，内部工作线程可能会失败并突然终止。

* 代码PausableThreadPoolExecutor类

  ```java
  /**
   * 描述：     演示每个任务执行前后放钩子函数
   */
  public class PauseableThreadPool extends ThreadPoolExecutor {
  
      private final ReentrantLock lock = new ReentrantLock();
      private Condition unpaused = lock.newCondition();
      private boolean isPaused;
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit,
              BlockingQueue<Runnable> workQueue) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
      }
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit, BlockingQueue<Runnable> workQueue,
              ThreadFactory threadFactory) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
      }
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit, BlockingQueue<Runnable> workQueue,
              RejectedExecutionHandler handler) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, handler);
      }
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit, BlockingQueue<Runnable> workQueue,
              ThreadFactory threadFactory, RejectedExecutionHandler handler) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory,
                  handler);
      }
  
      @Override
      protected void beforeExecute(Thread t, Runnable r) {
          super.beforeExecute(t, r);
          lock.lock();
          try {
              while (isPaused) {
                  unpaused.await();
              }
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              lock.unlock();
          }
      }
  
      private void pause() {
          lock.lock();
          try {
              isPaused = true;
          } finally {
              lock.unlock();
          }
      }
  
      public void resume() {
          lock.lock();
          try {
              isPaused = false;
              unpaused.signalAll();
          } finally {
              lock.unlock();
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
          PauseableThreadPool pauseableThreadPool = new PauseableThreadPool(10, 20, 10l,
                  TimeUnit.SECONDS, new LinkedBlockingQueue<>());
          Runnable runnable = new Runnable() {
              @Override
              public void run() {
                  System.out.println("我被执行");
                  try {
                      Thread.sleep(10);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          };
          for (int i = 0; i < 10000; i++) {
              pauseableThreadPool.execute(runnable);
          }
          Thread.sleep(1500);
          pauseableThreadPool.pause();
          System.out.println("线程池被暂停了");
          Thread.sleep(1500);
          pauseableThreadPool.resume();
          System.out.println("线程池被恢复了");
  
      }
  }
  ```

## 实现原理、源码分析

* 线程池组成部分
  * 线程池管理器
  * 工作线程
  * 任务列队			
  * 任务接口（Task）
* 线程池、ThreadPoolExecutor、ExecutorService、Executor、Executors等这么多和线程池相关的类，大家都是什么关系？
  * 线程池指的是哪个类：ExecutorService
    * ThreadPoolExecutor -> AbstractExecutorService -> ExecutorService -> Executor
  * 各个类的设计思想和作用
    * 核心方法纵览
      * Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；
      * 然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；
      * 抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；
      * 然后ThreadPoolExecutor继承了类AbstractExecutorService，并提供了一些新功能，比如获取核心线程数、获取任务队列等。
    * 彩蛋：如何快速查看继承关系图
      * IDEA的diagram功能，然后右键去显示方法
    * Executor
      * Executor 是一个抽象层面的核心接口，只有一个方法：void execute(Runnable command);
      * Executor 将任务本身和执行任务的过程解耦。
    * ExecutorService
      * ExecutorService继承了 Executor 接口，同时增加了几个有用的方法：
      * 例如提供了返回 Future 对象的submit方法，解决了Runnable无返回值的问题；
      * 例如提供了关闭线程池等方法，当调用 shutDown 方法时，线程池会停止接受新的任务，但会继续执行完毕等待中的任务。
    * Executor和ExecutorService的区别
    * Executors
      * Executors是一个工具类，就和Collections类似，方便创建常见类型的线程池，例如 FixedThreadPool等。
* 线程池实现任务复用的原理
  * 原因
    * 相同线程执行不同任务
    * 线程重用的核心是，线程池对Thread做了包装，不重复调用thread.start()，而是自己有一个Runnable.run()，run方法里面循环在跑，跑的过程中不断检查我们是否有新加入的子Runnable对象，有新的Runnable进来的话就调一下我们的run()，其实就一个大run()把其它小run()#1,run()#2,...给串联起来了。同一个Thread可以执行不同的Runnable，主要原因是线程池把线程和Runnable通过BlockingQueue给解耦了，线程可以从BlockingQueue中不断获取新的任务
* 线程池状态
  * 这是一个巧妙的设计，把同一个int变量，利用了2次，可以同时用高位和地低位保存“线程状态”和“线程数”，节省了空间；但是每次取数的时候，要做“与操作”，属于用时间换空间，但是与操作速度是极快的，所以几乎不花费时间。
  * ctl 共32位，其中高3位表示”线程池状态”，低29位代表”线程池中的任务数量”，线程池状态枚举：
    * RUNNING：接受新任务并处理排队任务
    * SHUTDOWN：不接受新任务，但处理排队任务
    * STOP：不接受新任务，也不处理排队任务，并中断正在进行的任务
    * TIDYING，中文是整洁，理解了中文就容易理解这个状态了：所有任务都已终止，workerCount为零时，线程会转换到TIDYING状态，并将运行terminate（）钩子方法。
    * TERMINATED：terminate（）运行完成
  * runState单调增加，但就和线程状态一样，并不一定会经历到每一个状态
* execute方法
  * Execute方法这可以说是核心方法，因为这是层层继承过来的，最上面可以追溯到Executor类。
  * 然后来看看ThreadPoolExecutor对execute的实现
  * 源码解读
* ThreadFactory
  * 用来创建到时候工作的线程，默认是DefaultThreadFactory()：
  * 从源码中可以看出，线程工厂给线程设置了默认名字（pool-线程池自增编号-thread-线程的自增编号），非守护线程，默认优先级、默认线程组

## 使用线程池的注意点

* 避免任务堆积
* 避免线程数过度增加
* 排查线程泄漏：线程回收不了

# ThreadLocal

## 两大使用场景——ThreadLocal的用途
* 典型场景1：每个线程需要一个独享的对象（通常是工具类，典型需要使用的类有SimpleDateFormat和Random）
  * 每个Thread内有自己的实例副本，不共享
  * SimpleDateFormat==为什么线程不安全？==
    * 每个SimpleDateFormat实例里面有一个Calendar对象，其中存放日期数据的变量都是线程不安全的，比如里面的fields，time等
* 典型场景2：每个线程内需要保存类似于全局变量的信息（例如在拦截器中获取用户信息，该信息在本线程执行的各方法中保持不变），可以让不同方法直接使用，却不想被多线程共享（因为不同线程获取到的用户信息不一样），避免参数传递的麻烦

```java
/**
 * 描述：     利用ThreadLocal，给每个线程分配自己的dateFormat对象，保证了线程安全，高效利用内存
 */
public class ThreadLocalNormalUsage05 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage05().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
        Date date = new Date(1000 * seconds);
//        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        SimpleDateFormat dateFormat = ThreadSafeFormatter.dateFormatThreadLocal2.get();
        return dateFormat.format(date);
    }
}

class ThreadSafeFormatter {

    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal2 = ThreadLocal
            .withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
}
```

* 实例：当前用户信息需要被线程内所有方法共享
  * 一个比较繁琐的解决方案是把user作为参数层层传递，但是这样做会导致代码冗余且不易维护
  * 每个线程内需要保存全局变量，可以让不同方法直接使用，避免参数传递的麻烦
    * 用ThreadLocal保存一些业务内容（用户权限信息、从用户系统获取到的用户名、UserID等）
    * 这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不同的
  * 当多线程同时工作时，需要保证线程安全，可以用synchronized，也可以用ConcurrentHashMap，但无论用什么，都会对性能有所影响
  * 更好的办法是使用ThreadLocal，这样无需synchronized，可以在不影响性能的情况下，也无需层层传递参数，就可达到保存当前线程对应的用户信息的目的
* 方法
  * 用ThreadLocal保存一些业务内容（用户权限信息、从用户系统获取到的用户名、userID等）
  * 这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的
  * 在线程生命周期内，都通过这个静态ThreadLocal实例的get()方法取得自己set过去的那个对象，避免了将这个对象作为参数传递的麻烦
  * 强调的是同一个请求内（同一个线程内）不同方法间的共享
  * 不需重写initialValue()方法，但是必须手动调用set()方法

```java
/**
 * 描述：     演示ThreadLocal用法2：避免传递参数的麻烦
 */
public class ThreadLocalNormalUsage06 {

    public static void main(String[] args) {
        new Service1().process("");
    }
}

class Service1 {
    public void process(String name) {
        User user = new User("超哥");
        UserContextHolder.holder.set(user);
        new Service2().process();
    }
}

class Service2 {
    public void process() {
        User user = UserContextHolder.holder.get();
        ThreadSafeFormatter.dateFormatThreadLocal.get();
        System.out.println("Service2拿到用户名：" + user.name);
        new Service3().process();
    }
}

class Service3 {
    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service3拿到用户名：" + user.name);
        UserContextHolder.holder.remove();
    }
}

class UserContextHolder {
    public static ThreadLocal<User> holder = new ThreadLocal<>();
}

class User {
    String name;
    public User(String name) {
        this.name = name;
    }
}
```

* 根据共享对象的生成时机不同，选择initialValue或set来保存对象
  * 场景一：initialValue
    * 在ThreadLocal第一次get的时候把对象给初始化出来，对象的初始化时机可以由我们控制
  * 场景二：set
    * 如果需要保存到ThreadLocal里的对象的生成时机不由我们随意控制，例如拦截器生成的用户信息，用ThreadLocal.set直接放到我们的ThreadLocal中去，以便后续使用

* 总结
  * ThreadLocal的两个作用
    * 让某个需要用到的对象在线程间隔离（每个线程都有自己的独立的对象）
    * 任何方法中都可以轻松获取到该对象
  * 两大场景的区别分析

## 使用ThreadLocal带来的好处
1. 达到线程安全

2.	不需要加锁，提高执行效率
3.	更高效地利用内存、节省开销：相比于每个任务都新建一个SimpleDateFormat，显然用ThreadLocal可以节省内存和开销。
4.	免去传参的繁琐：无论是场景一的工具类，还是场景二的用户名，都可以在任何地方直接通过ThreadLocal拿到，再也不需要每次都传同样的参数。ThreadLocal使得代码耦合度更低，更优雅

## 主要方法介绍

* T initialValue( )
  1. 该方法会返回当前线程对应的“初始值”，这是一个延迟加载的方法，只有在调用get的时候，才会触发。
  2. 当线程第一次使用get方法访问变量时，将调用此方法，除非线程先前调用了set方法，在这种情况下，不会为线程调用本initialValue方法。这正对应了ThreadLocal的两种典型用法。
  3. 通常，每个线程最多调用一次此方法，但如果已经调用了remove()后，再调用get()，则可以再次调用此方法。
  4. 如果不重写本方法，这个方法会返回null。一般使用匿名内部类的方法来重写initialize()方法，以便在后续使用中可以初始化副本对象。
* void set(T t)
* T get( )
* void remove( )

## 原理、源码分析

![image-20200722083131914](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200722083131914.png)

* 按照使用时候的顺序分析
* T get()：得到这个线程对应的value。如果是首次调用get()，则会调用initialize来得到这个值
  * get方法是先取出当前线程的ThreadLocalMap，然后调用map.getEntry方法，把本ThreadLocal的引用作为参数传入，取出map中属于本ThreadLocal的value
  * 注意，这个map以及map中的key和value都是保存在线程中的，而不是保存在ThreadLocal中
* getMap方法
* void set(T t)（setInitialValue方法很类似）：为这个线程设置一个新值
* T initialValue()：初始化
  1. 该方法会返回当前线程对应的“初始值”，这是一个延迟加载的方法，只有在调用get的时候，才会触发
  2. 当线程第一次使用get方法访问变量时，将调用此方法，除非线程先前调用了set方法，在这种情况下，不会为线程调用本initialValue方法
  3. 这正对应了ThreadLocal的两种典型用法
  4. 通常，每个线程最多调用一次此方法，但如果已经调用了remove()后，再调用get()，则可以再次调用此方法
  5. 如果不重写本方法，这个方法会返回null。一般使用匿名内部类的方法来重写initialValue()方法，以便在后续使用中可以初始化副本对象
  6. initialValue方法：是没有默认实现的，如果要用initialValue方法，需要自己实现，通常是匿名内部类的方式
* void remove()：删除对应这个线程的值
* ThreadLocalMap 类，也就是Thread.threadLocals
  * ThreadLocalMap类是每个线程Thread类里面的变量，里面最重要的是一个键值对数组Entry[] table，可以认为是一个map，键值对：
  * 键：这个ThreadLocal
    * 值：实际需要的成员变量，比如user或者simpleDateFormat对象
  * ThreadLocalMap这里采用的是线性探测法，也就是如果发生冲突，就继续找下一个空位置，而不是用链表拉链
* 两种使用场景殊途同归
* 通过源码分析可以看出，setInitialValue和直接set最后都是利用map.set()方法来设置值
  * 也就是说，最后都会对应到ThreadLocalMap的一个Entry，只不过是起点和入口不一样

## 注意点
* 内存泄漏
  * 什么是内存泄漏
  
    * 某个对象不再有用，但是占用的内存却不能被回收
  * Key的泄漏
  
    * ThreadLocalMap中的Entry继承自WeakReference，是弱引用
    * 弱引用的特点是，如果这个对象只被弱引用关联（没有任何强引用关联），那么这个对象就可以被回收
      * 所以弱引用不会阻止GC
  * Value的泄漏
    * ThreadLocalMap的每个Entry都是一个对key的弱引用，同时，每个Entry都包含了一个对value的强引用
    * 正常情况下，当线程终止，保存在ThreadLocal里的value会被垃圾回收，因为没有任何强引用了
    * 但是，如果线程不终止（比如线程需要保持很久），那么key对应的value就不能被回收，因为有以下的调用链：
      * Thread -> ThreadLocalMap -> Entry(key为null) -> Value
    * 因为value和Thread之间还存在这个强引用链路，所以导致value无法回收，就可能会出现OOM
    * JDK已经考虑到了这个问题，所以在set，remove，rehash方法中会扫描key为null的Entry，并把对应的value设置为null，这样，value对象就可以被回收
    * 但是如果一个ThreadLocal不被使用，那么实际上set，remove，rehash方法也不会被调用，如果同时线程又不停止，那么调用链就一直存在，那么就导致了value的内存泄漏
  * 如何避免内存泄露（阿里规约）
  * 调用remove方法，就会删除对应的Entry对象，就可以避免内存泄漏，所以使用完ThreadLocal之后，应该调用remove方法
  
* 空指针异常

  * 在进行get之前，必须先set，否则可能会报空指针异常

  ```java
  /**
   * 描述：     TODO
   */
  public class ThreadLocalNPE {
  
      ThreadLocal<Long> longThreadLocal = new ThreadLocal<Long>();
  
      public void set() {
          longThreadLocal.set(Thread.currentThread().getId());
      }
  
     // 用Long不会NPE，装箱拆箱
      public long get() {
          return longThreadLocal.get();    // null
      }
  
      public static void main(String[] args) {
          ThreadLocalNPE threadLocalNPE = new ThreadLocalNPE();
          System.out.println(threadLocalNPE.get());
          Thread thread1 = new Thread(new Runnable() {
              @Override
              public void run() {
                  threadLocalNPE.set();
                  System.out.println(threadLocalNPE.get());
              }
          });
          thread1.start();
      }
  }
  ```

* 共享对象

  * 如果在每个线程中ThreadLocal.set()进去的东西本来就是多线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身，还是有并发访问问题

* 如果可以不使用ThreadLocal就解决问题，那么不要强行使用

  * 例如在任务数很少的时候，在局部变量中可以新建对象就可以解决问题，那么就不需要使用到ThreadLocal

* 优先使用框架的支持，而不是自己创造

  * 例如在Spring中，如果可以使用RequestContextHolder，那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove()方法等，造成内存泄漏

## 实际应用场景——在Spring中的实例分析

* DateTimeContextHolder类，看到里面用了ThreadLocal
* 每次HTTP请求都对应一个线程，线程之前相互隔离，这就是ThreadLocal的典型应用场景

# 锁

## Lock接口

* 简介、地位、作用

  * 锁是一种工具，用于控制对共享资源的访问
  * Lock并不是用来代替synchronized的，而是当使用synchronized不合适或不足以满足要求的时候，来提供高级功能的
  * Lock接口最常见的实现类是ReentrantLock
  * 通常情况下，Lock只允许一个线程来访问这个共享资源。不过有的时候，一些特殊的实现也可允许并发访问，比如ReadWriteLock里面的ReadLock

* 为什么synchronized不够用？为什么需要Lock？

  * 效率低：锁的释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程
  * 不够灵活（读写锁更灵活）：加锁和释放的时机单一，每个锁仅有单一的条件（某个对象），可能是不够的
  * 无法知道是否成功获得锁

* 方法介绍

  * 在Lock中声明了四个方法来获取锁
    * lock()
      * lock()就是最普通的获取锁。如果锁已被其他线程获取，则进行等待
      * Lock不会像synchronized一样在异常时自动释放锁
      * 因此最佳实践是，在finally中释放锁，以保证发生异常时锁一定被释放
      * lock()方法不能被中断，隐患：一旦陷入死锁，lock()就会陷入永久等待
    * tryLock()
      * 用来尝试获取锁，如果当前锁没有被其他线程占用，则获取成功，则返回true，否则返回false，代表获取锁失败
      * 可以根据是否能获取到锁来决定后续程序的行为
      * 该方法会立刻返回，即便在拿不到锁时不会一直在那等
    * tryLock(long time, TimeUnit unit)
      * 超时就放弃
    * lockInterruptibly()
      * 相当于tryLock(long time, TimeUnit unit)把超时时间设置为无限，在等待锁的过程中，线程可以被中断
  * unlock()

  ```java
  /**
   * 描述：     用tryLock来避免死锁
   */
  public class TryLockDeadlock implements Runnable {
  
      int flag = 1;
      static Lock lock1 = new ReentrantLock();
      static Lock lock2 = new ReentrantLock();
  
      public static void main(String[] args) {
          TryLockDeadlock r1 = new TryLockDeadlock();
          TryLockDeadlock r2 = new TryLockDeadlock();
          r1.flag = 1;
          r1.flag = 0;
          new Thread(r1).start();
          new Thread(r2).start();
  
      }
  
      @Override
      public void run() {
          for (int i = 0; i < 100; i++) {
              if (flag == 1) {
                  try {
                      if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程1获取到了锁1");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock2.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程1获取到了锁2");
                                      System.out.println("线程1成功获取到了两把锁");
                                      break;
                                  } finally {
                                      lock2.unlock();
                                  }
                              } else {
                                  System.out.println("线程1获取锁2失败，已重试");
                              }
                          } finally {
                              lock1.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程1获取锁1失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
  
              if (flag == 0) {
                  try {
                      if (lock2.tryLock(3000, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程2获取到了锁2");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程2获取到了锁1");
                                      System.out.println("线程2成功获取到了两把锁");
                                      break;
                                  } finally {
                                      lock1.unlock();
                                  }
                              } else {
                                  System.out.println("线程2获取锁1失败，已重试");
                              }
                          } finally {
                              lock2.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程2获取锁2失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
  ```

  ```java
  /**
   * 描述：     TODO
   */
  public class LockInterruptibly implements Runnable {
  
      private Lock lock = new ReentrantLock();
  public static void main(String[] args) {
      LockInterruptibly lockInterruptibly = new LockInterruptibly();
      Thread thread0 = new Thread(lockInterruptibly);
      Thread thread1 = new Thread(lockInterruptibly);
      thread0.start();
      thread1.start();
  
      try {
          Thread.sleep(2000);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
      thread1.interrupt();
  }
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + "尝试获取锁");
          try {
              lock.lockInterruptibly();
              try {
                  System.out.println(Thread.currentThread().getName() + "获取到了锁");
                  Thread.sleep(5000);
              } catch (InterruptedException e) {
                  System.out.println(Thread.currentThread().getName() + "睡眠期间被中断了");
              } finally {
                  lock.unlock();
                  System.out.println(Thread.currentThread().getName() + "释放了锁");
              }
          } catch (InterruptedException e) {
              System.out.println(Thread.currentThread().getName() + "获得锁期间被中断了");
          }
      }
  }
  ```

* 可见性保证

  * 可见性
  * happends-before
  * Lock的加解锁和synchronized有同样的内存语义，也就是说，下一个线程加锁后可以看到所有前一个线程解锁前发生的所有操作

## 锁的分类

* 多个类型可以并存：有可能一个锁，同时属于两种类型，比如ReentrantLock既是互斥锁，又是可重入锁

## 线程要不要锁住同步资源

* 为什么会诞生非互斥同步锁
  * 互斥同步锁的劣势
    * 阻塞和唤醒带来的性能劣势
    * 永久阻塞：如果持有锁的线程被永久阻塞，比如遇到了无限循环、死锁等活跃性问题，那么等待该线程释放锁的那几个悲催的线程，将永远也得不到执行
    * 优先级反转

* 锁住
  
  * 悲观锁
  
    * synchronized和lock接口
  * 为了确保结果的正确性，会在每次获取并修改数据时，把数据锁住
    * Java中悲观锁的实现就是synchronized和Lock相关类
  
* 不锁住
  
  * 乐观锁
  
    * 典型例子就是原子类、并发容器等
  * 认为自己在处理操作的时候不会有其他线程来干扰，所以并不会锁住被操作对象
    
    * 在更新的时候，去对比在修改的期间数据有没有被其他人修改过：如果没被改变过，就说明真的是只有我自己在操作，那我就正常去修改数据
  * 如果数据和一开始拿到的不一样了，说明其他人在这段时间内改过数据，那就不能继续刚才的更新数据过程了，会选择放弃、报错、重试等策略
    * 乐观锁的实现一般都是利用CAS算法来实现的
  
    ```java
    /**
     * 描述：     TODO
     */
    public class PessimismOptimismLock {
        int a;
        public static void main(String[] args) {
          // 乐观锁
            AtomicInteger atomicInteger = new AtomicInteger();
            atomicInteger.incrementAndGet();
        }
    
        public synchronized void testMethod() {
            a++;
        }
    }
    ```
  
  * 典型例子
  
    * Git：当向远程仓库push的时候，git会检查远端仓库的版本是不是领先于现在的版本，如果远程仓库的版本号和本地的不一样，就表示有其他人修改了远端代码了，这次提交就失败；如果远端和本地版本号一致，就可以顺利提交版本到远端仓库
  
* 数据库

  * select for update就是悲观锁
  * 用version控制数据库就是乐观锁
    * 添加一个字段lock_version，先查询这个更新语句的version：select * from table 然后update set num = 2, version = version + 1 where version = 1 and id = 5;
    * 如果version被更新了等于2，不一样就会更新出错，这就是乐观锁的原理

* 开销对比
  * 悲观锁的原始开销要高于乐观锁，但是特点是一劳永逸，临界区持锁时间就算越来越差，也不会对互斥锁的开销造成影响
  * 相反，虽然乐观锁一开始的开销比悲观锁小，但是如果自旋时间很长或者不停重试，那么消耗的资源也会越来越多
* 两种锁各自的使用场景：各有千秋
  * 悲观锁：
    * 适合并发写入多的情况，适用于临界区持锁时间比较长的情况，悲观锁可以避免大量的无用自旋等消耗，典型情况：
      * 临界区有IO操作
      * 临界区代码复杂或者循环量大
      * 临界区竞争非常激烈
  * 乐观锁：
    * 适合并发写入少，大部分是读取的场景，不加锁的能让读取性能大幅提高

## 多线程能否共享一把锁

* ReentrantReadWriteLock读写锁为例

* 什么是共享锁和排他锁
  * 排他锁，又称为独占锁、独享锁
  * 共享锁，又称为读锁，获得共享锁之后，可以查看但无法修改和删除数据，其他线程此时也可以获取到共享锁，也可以查看但无法修改和删除数据
  * 共享锁和排他锁的典型是读写锁ReentrantReadWriteLock，其中读锁是共享锁，写锁是独享锁
  
* 读写锁的作用
  * 多个读操作同时进行，并没有线程安全问题
  * 在读的地方使用读锁，在写的地方使用写锁，灵活控制，如果没有写锁的情况下，读是无阻塞的，提高了程序的执行效率
  
* 读写锁的规则
  * 多个线程只申请读锁，都可以申请到
  * 如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁
  * 如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁
  * 一句话总结：要么是一个或多个线程同时有读锁，要么是一个线程有写锁，但是两者不会同时出现（要么多读，要么一写）
  * 读写锁只是一把锁，可以通过两种方式锁定：读锁定和写锁定。读写锁可以同时被一个或多个线程读锁定，也可以被单一线程写锁定。但是永远不能同时对这把锁进行读锁定和写锁定

* ReentrantReadWriteLock具体用法

  ```java
  import java.util.concurrent.locks.ReentrantReadWriteLock;
  
  /**
   * 描述：     TODO
   */
  public class CinemaReadWrite {
  
      private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
      private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
      private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
  
      private static void read() {
          readLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了读锁，正在读取");
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放读锁");
              readLock.unlock();
          }
      }
  
      private static void write() {
          writeLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了写锁，正在写入");
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放写锁");
              writeLock.unlock();
          }
      }
  
      public static void main(String[] args) {
          new Thread(()->read(),"Thread1").start();
          new Thread(()->read(),"Thread2").start();
          new Thread(()->write(),"Thread3").start();
          new Thread(()->write(),"Thread4").start();
      }
  }
  ```

* 读锁和写锁的交互方式

* 读锁插队策略

  * 公平锁：不允许插队
  * 非公平锁
  * 写锁可以随时插队
    * 读锁仅在等待队列头节点不是想获取写锁的线程的时候可以插队

  ```java
  import java.util.concurrent.locks.ReentrantReadWriteLock;
  
  /**
   * 描述：     演示非公平和公平的ReentrantReadWriteLock的策略
   */
  public class NonfairBargeDemo {
  
      private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(
              true);
  
      private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
      private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
  
      private static void read() {
          System.out.println(Thread.currentThread().getName() + "开始尝试获取读锁");
          readLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
              try {
                  Thread.sleep(20);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放读锁");
              readLock.unlock();
          }
      }
  
      private static void write() {
          System.out.println(Thread.currentThread().getName() + "开始尝试获取写锁");
          writeLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
              try {
                  Thread.sleep(40);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放写锁");
              writeLock.unlock();
          }
      }
  
      public static void main(String[] args) {
          new Thread(()->write(),"Thread1").start();
          new Thread(()->read(),"Thread2").start();
          new Thread(()->read(),"Thread3").start();
          new Thread(()->write(),"Thread4").start();
          new Thread(()->read(),"Thread5").start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  Thread thread[] = new Thread[1000];
                  for (int i = 0; i < 1000; i++) {
                      thread[i] = new Thread(() -> read(), "子线程创建的Thread" + i);
                  }
                  for (int i = 0; i < 1000; i++) {
                      thread[i].start();
                  }
              }
          }).start();
      }
  }
  ```

  ```java
  import java.util.concurrent.locks.ReentrantReadWriteLock;
  
  /**
   * 描述：     TODO
   */
  public class CinemaReadWriteQueue {
  
      private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(false);
      private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
      private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
  
      private static void read() {
          readLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了读锁，正在读取");
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放读锁");
              readLock.unlock();
          }
      }
  
      private static void write() {
          writeLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了写锁，正在写入");
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放写锁");
              writeLock.unlock();
          }
      }
  
      public static void main(String[] args) {
          new Thread(()->write(),"Thread1").start();
          new Thread(()->read(),"Thread2").start();
          new Thread(()->read(),"Thread3").start();
          new Thread(()->write(),"Thread4").start();
          new Thread(()->read(),"Thread5").start();
      }
  }
  ```

* 锁的升降级

  * 支持锁的降级，不支持升级，可以避免死锁

  ```java
  import java.util.concurrent.atomic.AtomicInteger;
  import java.util.concurrent.locks.ReentrantReadWriteLock;
  
  /**
   * 描述：     演示ReentrantReadWriteLock可以降级，不能升级
   */
  public class Upgrading {
  
      private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(
              false);
      private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
      private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
  
      private static void readUpgrading() {
          readLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了读锁，正在读取");
              Thread.sleep(1000);
              System.out.println("升级会带来阻塞");
              writeLock.lock();
              System.out.println(Thread.currentThread().getName() + "获取到了写锁，升级成功");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放读锁");
              readLock.unlock();
          }
      }
  
      private static void writeDowngrading() {
          writeLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了写锁，正在写入");
              Thread.sleep(1000);
              readLock.lock();
              System.out.println("在不释放写锁的情况下，直接获取读锁，成功降级");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              readLock.unlock();
              System.out.println(Thread.currentThread().getName() + "释放写锁");
              writeLock.unlock();
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
  //        System.out.println("先演示降级是可以的");
  //        Thread thread1 = new Thread(() -> writeDowngrading(), "Thread1");
  //        thread1.start();
  //        thread1.join();
  //        System.out.println("------------------");
  //        System.out.println("演示升级是不行的");
          Thread thread2 = new Thread(() -> readUpgrading(), "Thread2");
          thread2.start();
      }
  }
  ```

* 共享锁和排他锁总结
  * ReentrantReadWriteLock实现了ReadWriteLock接口，最主要的有两个方法：readLock()和writeLock()用来获取读锁和写锁
  * 锁申请和释放策略
    * 多个线程只申请读锁，都可以申请到
    * 如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁
    * 如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或读锁，则申请的线程会一直等待释放写锁
    * 要么是一个或多个线程同时有读锁，要么是一个线程有写锁，但是两者不会同时出现
  * 插队策略：为了防止饥饿，读锁不能插队
  * 升降级策略：智能降级，不能升级
  * 适用场合：相比于ReentrantLock适用于一般场合，ReentrantReadWriteLock适用于读多写少的情况，合理使用可以进一步提高并发效率

* 可以
  
  * 共享锁
  
* 不可以
  
* 独占锁
  
* 自旋锁和阻塞锁

  * 阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间
  * 如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长
  * 在许多场景中，同步资源的锁定时间很短，为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失
  * 如果物理机器有多个处理器，能够让两个或以上的线程同时并行执行，就可以让后面那个请求锁的线程不放弃CPU的执行时间，看看持有锁的线程是否很快就会释放锁
  * 而为了让当前线程“稍等一下”，需让当前线程进行自旋，如果在自旋完成后前面锁定同步资源的线程已经释放了锁，那么当前线程就可以不必阻塞而是直接获取同步资源，从而避免切换线程的开销。这就是自旋锁
  * 阻塞锁和自旋锁相反，阻塞锁如果遇到没拿到锁的情况，会直接把线程阻塞，直到被唤醒

* 自旋锁的缺点

  * 如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源
  * 在自旋的过程中，一直消耗CPU，所以虽然自旋锁的起始开销低于悲观锁，但是随着自旋时间的增长，开销也是线性增长的

* 原理和源码分析
  * 在java1.5版本及以上的并发框架java.util.concurrent的atomic包下的类基本都是自旋锁的实现
  * AtomicInteger的实现：自旋锁的实现原理是CAS，AtomicInteger中调用unsafe进行自增操作的源码中的do-while循环就是一个自旋操作，如果修改过程中遇到其他线程竞争导致没修改成功，就在while里死循环，直至修改成功

* 缺点

* 原理和源码分析

  ```java
  import java.util.concurrent.atomic.AtomicReference;
  
  /**
   * 描述：     自旋锁
   */
  public class SpinLock {
  
      private AtomicReference<Thread> sign = new AtomicReference<>();
  
      public void lock() {
          Thread current = Thread.currentThread();
          while (!sign.compareAndSet(null, current)) {
              System.out.println("自旋获取失败，再次尝试");
          }
      }
  
      public void unlock() {
          Thread current = Thread.currentThread();
          sign.compareAndSet(current, null);
      }
  
      public static void main(String[] args) {
          SpinLock spinLock = new SpinLock();
          Runnable runnable = new Runnable() {
              @Override
              public void run() {
                  System.out.println(Thread.currentThread().getName() + "开始尝试获取自旋锁");
                  spinLock.lock();
                  System.out.println(Thread.currentThread().getName() + "获取到了自旋锁");
                  try {
                      Thread.sleep(300);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      spinLock.unlock();
                      System.out.println(Thread.currentThread().getName() + "释放了自旋锁");
                  }
              }
          };
          Thread thread1 = new Thread(runnable);
          Thread thread2 = new Thread(runnable);
          thread1.start();
          thread2.start();
      }
  }
  ```

* 适用场景

  * 自旋锁一般用于多核的服务器，在并发度不是特别高的情况下，比阻塞锁的效率高
  * 另外，自旋锁适用于临界区比较短小的情况，否则如果临界区很大（线程一旦拿到锁，很久以后才会释放），那也是不合适的

* 可中断锁

  * 在Java中，synchronized就不是可中断锁，而Lock是可中断锁，因为tryLock(time)和lockInterruptibly都能响应中断
  * 如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以中断它，这种就是可中断锁

* 锁优化
  * Java虚拟机对锁的优化
    * 自旋锁和自适应
    * 锁消除
    * 锁粗化

* 在写代码时如何优化锁和提高并发性能
  1. 缩小同步代码块
  2. 尽量不要锁住方法
  3. 减少请求锁的次数
  4. 避免人为制造热点
  5. 锁中尽量不要再包含锁
  6. 选择合适的锁类型或合适的工具类

* 总结：千变万化的锁

  1. Lock接口
2. 锁的分类
  3. 乐观锁和悲观锁
4. 可重入锁和非可重入锁，已ReentrantLock为例（重点）
  5. 公平锁和非公平锁
6. 共享锁和排它锁：以ReentrantReadWriteLock读写锁为例（重点）
  7. 自旋锁和阻塞锁
8. 可中断锁：可以响应中断的锁
  9. 锁优化

## 多线程竞争时，是否排队

* 公平指的是按照线程请求的顺序，来分配锁；非公平指的是，不完全按照请求的顺序，在一定情况下，可以插队
* 注意：非公平也同样不提倡“插队”行为，这里的非公平，指的是“在合适的时机”插队，而不是盲目插队
* 什么是合适的时机呢？
* 实际情况并不是这样的，Java设计者这样设计的目的，是为了提高效率

* 避免唤醒带来的空档期
* 公平的情况（以ReentrantLock为例）
  * 如果在创建ReentrantLock对象时，参数填写为true，那么这就是个公平锁
* 不公平的情况（以ReentrantLock为例）
  * 如果在线程1释放锁的时候，线程5恰好去执行lock()
  * 由于ReentrantLock发现此时并没有线程持有lock这把锁（线程2还没来得及获取到，因为获取需要时间）
  * 线程5可以插队，直接拿到这把锁，这也是ReentrantLock默认的公平策略，也就是“不公平”

```java
import java.util.Random;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 描述：     演示公平和不公平两种情况
 */
public class FairLock {

    public static void main(String[] args) {
        PrintQueue printQueue = new PrintQueue();
        Thread thread[] = new Thread[10];
        for (int i = 0; i < 10; i++) {
            thread[i] = new Thread(new Job(printQueue));
        }
        for (int i = 0; i < 10; i++) {
            thread[i].start();
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Job implements Runnable {

    PrintQueue printQueue;

    public Job(PrintQueue printQueue) {
        this.printQueue = printQueue;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "开始打印");
        printQueue.printJob(new Object());
        System.out.println(Thread.currentThread().getName() + "打印完毕");
    }
}

class PrintQueue {
    // true 公平、false 不公平
    private Lock queueLock = new ReentrantLock(true);

    public void printJob(Object document) {
        queueLock.lock();
        try {
            int duration = new Random().nextInt(10) + 1;
            System.out.println(Thread.currentThread().getName() + "正在打印，需要" + duration);
            Thread.sleep(duration * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            queueLock.unlock();
        }

        queueLock.lock();
        try {
            int duration = new Random().nextInt(10) + 1;
            System.out.println(Thread.currentThread().getName() + "正在打印，需要" + duration+"秒");
            Thread.sleep(duration * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            queueLock.unlock();
        }
    }
}
```

* 特例：
  * 针对tryLock()方法，它不遵守设定的公平的规则
  * 例如，当有线程执行tryLock()的时候，一旦有线程释放了锁，那么这个正在tryLock的线程就能获取到锁，即使在它之前已经有其他线程在等待队列里了
* 对比公平和非公平的优缺点
  * 公平锁：优点——各线程公平平等，每个线程在等待一段时间后，总有执行的机会；缺点——更慢，吞吐量更小
  * 不公平锁：优点——更快，吞吐量更大；缺点：有可能产生线程饥饿，也就是某些线程在长时间内，始终得不到执行

![image-20200726190539280](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200726190539280.png)

* 排队
  * 公平锁
* 先尝试插队，插队失败再排队
  * 非公平锁

## 同一个线程是否可以重复获取同一把锁

* 可以
  
  * 可重入
  
    * 好处：避免死锁和提升封装性
  
      ```java
      /**
       * 描述：     TODO
       */
      public class GetHoldCount {
          private  static ReentrantLock lock =  new ReentrantLock();
          public static void main(String[] args) {
              System.out.println(lock.getHoldCount());
              lock.lock();
              System.out.println(lock.getHoldCount());
              lock.lock();
              System.out.println(lock.getHoldCount());
              lock.lock();
              System.out.println(lock.getHoldCount());
              lock.unlock();
              System.out.println(lock.getHoldCount());
              lock.unlock();
              System.out.println(lock.getHoldCount());
              lock.unlock();
              System.out.println(lock.getHoldCount());
          }
      }
      ```
  
      ```java
      /**
       * 描述：     TODO
       */
      public class RecursionDemo {
          private static ReentrantLock lock = new ReentrantLock();
          private static void accessResource() {
              lock.lock();
              try {
                  System.out.println("已经对资源进行了处理");
                  if (lock.getHoldCount()<5) {
                      System.out.println(lock.getHoldCount());
                      accessResource();
                      System.out.println(lock.getHoldCount());
                  }
              } finally {
                  lock.unlock();
              }
          }
          public static void main(String[] args) {
              accessResource();
          }
      }
      ```
  
      ![image-20200724083214624](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200724083214624.png)
  
* 不可以
  
  * 不可重入锁
  
* ReentrantLock

  * isHeldByCurrentThread可以看出锁是否被当前线程持有
* getQueueLength可以返回当前正在等待这把锁的队列有多长，一般这两个方法是开发和调试时候使用，上线后用到的不多
  
  ```java
  /**
   * 描述：     演示多线程预定电影院座位
   */
  public class CinemaBookSeat {
  
      private static ReentrantLock lock = new ReentrantLock();
  
      private static void bookSeat() {
          lock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "开始预定座位");
              Thread.sleep(1000);
              System.out.println(Thread.currentThread().getName() + "完成预定座位");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              lock.unlock();
          }
      }
  
      public static void main(String[] args) {
          new Thread(() -> bookSeat()).start();
          new Thread(() -> bookSeat()).start();
          new Thread(() -> bookSeat()).start();
          new Thread(() -> bookSeat()).start();
      }
  }
  ```
```
  
  ```java
  /**
   * 描述：     演示ReentrantLock的基本用法，演示被打断
   */
  public class LockDemo {
  
      public static void main(String[] args) {
          new LockDemo().init();
      }
  
      private void init() {
          final Outputer outputer = new Outputer();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  while (true) {
                      try {
                          Thread.sleep(5);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      outputer.output("悟空");
                  }
  
              }
          }).start();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  while (true) {
                      try {
                          Thread.sleep(5);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      outputer.output("大师兄");
                  }
  
              }
          }).start();
      }
  
      static class Outputer {
  
          Lock lock = new ReentrantLock();
  
          //字符串打印方法，一个个字符的打印
          public void output(String name) {
  
              int len = name.length();
              lock.lock();
              try {
                  for (int i = 0; i < len; i++) {
                      System.out.print(name.charAt(i));
                  }
                  System.out.println("");
              } finally {
                  lock.unlock();
              }
          }
      }
  }
```

## 是否可中断

* 可以
  * 可中断锁
* 不可以
  * 非可中断锁

## 等锁的过程

* 自旋
  * 自旋锁
* 阻塞
  * 非自旋锁

## 锁优化

* 关于锁的优化，主要分为两个大方面，都很重要，一方面是JVM对锁的性能的优化，另一方面是我们作为程序员，在代码层面，例如最佳实践、使用场景等，也可以对锁性能的提高做出贡献
* Java虚拟机对锁的优化
  * 自旋锁和自适应
  * 锁消除
  * 锁粗化
* 我们在写代码时如何优化锁和提高并发性能
  * 缩小同步代码块
    * 只锁必要的操作数据的部分，把其他不相关的、开销大的、可能被阻塞的操作，例如IO操作，通通移出互斥的范围。
    * 当然，也不能过分小，原子操作必须包含到一个同步块中
  * 尽量不要锁住方法
    * 方法这个级别太高了、范围太宽了。如果我们锁住方法，很有可能其实我们并不需要这么大锁的范围，我们可以把这个锁住方法的锁优化成代码块，代码会锁住的部分往往比方法要小
  * 减小锁的粒度（把锁拆小）
  * 减少请求锁的次数
    * 如果只看这个标题，同学们可能会很困惑，我明明是有请求锁的需求，你凭什么让我减少次数呢？如果我减少次数的话，是不是就无法保证线程安全了呢？实际情况不总是这样的，有的时候我们可以通过减少请求锁的次数来优化性能，让我们来举一个具体的非常好的例子：
    * 在很多日志框架里。最主要的思路是把系统的打印日志的能力进行一系列包装，比如说可以设置打印的级别设置、格式等等。
    * 但是对于日志框架，有一个很重要的点，那就是当多个线程同时进行日志打印的时候，可能如果只是采用刚才的那种简单的架构设计，那么多个线程同时打印便会造成线程竞争，因为这是一个IO操作，多个线程需要操作同一个日志文件，需要加锁。
    * 这里我们对架构进行改进，把多个线程的打印日志的这种需求收集到一起，然后统一由某一个线程去执行写入文件的操作，这样的话便不会有过多的锁的竞争的情况，可以大大提高打印日志的效率，我们相当于是使用了一个消息队列来作为我们的中间层，这也是我们现有的优质日志框架Lof4j2的一个思路。
  * 避免人为制造“热点”
    * 这里的“热点”，指的是一个大家都想同时访问的，并且需要互斥同步的资源。有的时候。我们会人为制造出这种资源，从而导致了性能的下降。
    * 比如一个典型的例子就是在HashMap中，size()方法是获取到当前容器大小，size的第一种实现方式是每次调用size方法的时候，就去遍历一遍，但是这样的时间复杂度是O(n)。
    * 所以就有一种优化方法是，每一次增加元素的时候，比如put方法，我就去更新计数器，这样一来，如果需要知道集合的大小，我只需要读取一下之前已经更新过的数值就可以了，把复杂度降到了O(1)，这是一个很不错的优化。
    * 可是这有一个问题，那就是如果并发地去放置元素，就会导致这个计数器被多线程同时使用，也就带来了性能问题，因为多个线程需要去竞争锁。
    * 一个相当不错的解决方案是，在JDK7的ConcurrentHashMap中，不同的segment之间不共用同一个计数器，而仅仅是在最终真正需要获取集合容量的时候，再把多个计数器的值相加，这样的思维，我们在之前的LongAdder中已经分析过了，是一种非常好的思路，可以大大提高并发的能力。
  * 锁中尽量不要再包含锁
    * 否则很容易死锁，演示一个死锁的例子，之前写过
  * 选择合适的锁类型或合适的工具类
    * 不同的锁适合不同的场景。放弃互斥锁，可以提高并发性能
  * 监控CPU的利用率

# atomic包

## 什么是原子类，有什么作用？

* 不可分割
* 一个操作是不可中断的，即便是多线程的情况下也可以保证
* java.util.concurrent.atomic
* 原子类的作用和锁类似，是为了保证并发情况下线程安全。不过原子类相比于锁，有一定的优势
  * 粒度更细：原子变量可以把竞争范围缩小到变量级别，通常锁的粒度都要大于原子变量的粒度
  * 效率更高：通常，使用原子类的效率会比使用锁的效率更高，除了高度竞争的情况

## Atomic基本类型原子类，已AtomicInteger为例

![image-20200727200751304](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200727200751304.png)

* AtomicInteger 类常用方法
  * public final int get() //获取当前的值
  
  * public final int getAndSet(int newValue)//获取当前的值，并设置新的值
  
  * public final int getAndIncrement()//获取当前的值，并自增
  
  * public final int getAndDecrement() //获取当前的值，并自减
  
  * public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
  
  * boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
  
    ```java
    import java.util.concurrent.atomic.AtomicInteger;
    
    /**
     * 描述：     演示AtomicInteger的基本用法，对比非原子类的线程安全问题，使用了原子类之后，不需要加锁，也可以保证线程安全。
     */
    public class AtomicIntegerDemo1 implements Runnable {
    
        private static final AtomicInteger atomicInteger = new AtomicInteger();
    
        public void incrementAtomic() {
            atomicInteger.getAndAdd(-90);
        }
    
        private static volatile int basicCount = 0;
    
        public synchronized void incrementBasic() {
            basicCount++;
        }
    
        public static void main(String[] args) throws InterruptedException {
            AtomicIntegerDemo1 r = new AtomicIntegerDemo1();
            Thread t1 = new Thread(r);
            Thread t2 = new Thread(r);
            t1.start();
            t2.start();
            t1.join();
            t2.join();
            System.out.println("原子类的结果：" + atomicInteger.get());    // 20000
            System.out.println("普通变量的结果：" + basicCount);           
        }
    
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                incrementAtomic();
                incrementBasic();
            }
        }
    }
    ```
* 案例：银行存款——AtomicInteger使用方法
* AtomicInteger源码分析
  * 用Unsafe来实现底层操作
  * 用volatile修饰value字段，保证可见性
  * Unsafe的getAndAddInt方法分析：自旋 + CAS（乐观锁）。在这个过程中，通过compareAndSwapInt比较并更新value值，如果更新失败，重新获取旧值，然后更新

## AtomicArray数组类型原子类

* 数组里的元素，都可以保证原子性，AtomicIntegerArray相当于把AtomicInteger组合成一个数组，一共有3种，分别是AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray

  ```java
  import java.util.concurrent.atomic.AtomicIntegerArray;
  
  /**
   * 描述：     演示原子数组的使用方法
   */
  public class AtomicArrayDemo {
  
      public static void main(String[] args) {
          AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(1000);
          Incrementer incrementer = new Incrementer(atomicIntegerArray);
          Decrementer decrementer = new Decrementer(atomicIntegerArray);
          Thread[] threadsIncrementer = new Thread[100];
          Thread[] threadsDecrementer = new Thread[100];
          for (int i = 0; i < 100; i++) {
              threadsDecrementer[i] = new Thread(decrementer);
              threadsIncrementer[i] = new Thread(incrementer);
              threadsDecrementer[i].start();
              threadsIncrementer[i].start();
          }
  
  //        Thread.sleep(10000);
          for (int i = 0; i < 100; i++) {
              try {
                  threadsDecrementer[i].join();
                  threadsIncrementer[i].join();
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
  
          for (int i = 0; i < atomicIntegerArray.length(); i++) {
  //            if (atomicIntegerArray.get(i)!=0) {
  //                System.out.println("发现了错误"+i);
  //            }
              System.out.println(atomicIntegerArray.get(i));
          }
          System.out.println("运行结束");
      }
  }
  
  class Decrementer implements Runnable {
  
      private AtomicIntegerArray array;
  
      public Decrementer(AtomicIntegerArray array) {
          this.array = array;
      }
  
      @Override
      public void run() {
          for (int i = 0; i < array.length(); i++) {
              array.getAndDecrement(i);
          }
      }
  }
  
  class Incrementer implements Runnable {
  
      private AtomicIntegerArray array;
  
      public Incrementer(AtomicIntegerArray array) {
          this.array = array;
      }
  
      @Override
      public void run() {
          for (int i = 0; i < array.length(); i++) {
              array.getAndIncrement(i);
          }
      }
  }
  ```

* 案例

## AtomicReference引用类型原子类

* AtomicReference
  * AtomicReference类的作用，和AtomicInteger并没有本质区别， AtomicInteger可以让一个整数保证原子性，而AtomicReference可以让一个对象保证原子性，当然，AtomicReference的功能明显比AtomicInteger强，因为一个对象里可以包含很多属性。用法和AtomicInteger类似。
  * 代码案例
* AtomicStampedReference——加上了时间戳，防止ABA问题
  * 刚才说到了AtomicReference会带来ABA问题，而AtomicStampedReference的诞生，就是解决了这个问题

```java
import java.util.concurrent.atomic.AtomicReference;

/**
 * 描述：     自旋锁
 */
public class SpinLock {

    private AtomicReference<Thread> sign = new AtomicReference<>();

    public void lock() {
        Thread current = Thread.currentThread();
        while (!sign.compareAndSet(null, current)) {
            System.out.println("自旋获取失败，再次尝试");
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        sign.compareAndSet(current, null);
    }

    public static void main(String[] args) {
        SpinLock spinLock = new SpinLock();
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "开始尝试获取自旋锁");
                spinLock.lock();
                System.out.println(Thread.currentThread().getName() + "获取到了自旋锁");
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    spinLock.unlock();
                    System.out.println(Thread.currentThread().getName() + "释放了自旋锁");
                }
            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
    }
}
```

## 把普通变量升级为原子类：用AtomicIntegerFieldUpdater升级原有变量

* 概述
  
  * 对普通变量进行升级
* 使用场景
  * 通常希望引用变量“normal”（即，不必总是通过原子类上的get或set方法引用它）
  * 但偶尔需要一个原子get-set操作

* 用法，代码演示
  
  * AtomicIntegerFieldUpdaterDemo类
  
    ```java
    import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;
    
    /**
     * 描述：     演示AtomicIntegerFieldUpdater的用法
     */
    public class AtomicIntegerFieldUpdaterDemo implements Runnable{
    
        static Candidate tom;
        static Candidate peter;
    
        public static AtomicIntegerFieldUpdater<Candidate> scoreUpdater = AtomicIntegerFieldUpdater
                .newUpdater(Candidate.class, "score");
    
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                peter.score++;
                scoreUpdater.getAndIncrement(tom);
            }
        }
    
        public static class Candidate {
            volatile int score;
        }
    
        public static void main(String[] args) throws InterruptedException {
            tom=new Candidate();
            peter=new Candidate();
            AtomicIntegerFieldUpdaterDemo r = new AtomicIntegerFieldUpdaterDemo();
            Thread t1 = new Thread(r);
            Thread t2 = new Thread(r);
            t1.start();
            t2.start();
            t1.join();
            t2.join();
            System.out.println("普通变量："+peter.score);
            System.out.println("升级后的结果"+ tom.score);
        }
    }
    ```
* 注意点
  * 第一，Updater只能修改它可见范围内的变量。因为Updater使用反射得到这个变量。如果变量不可见，就会出错。比如如果 score申明为 private， 就是不可行的。 
  * 第二，为了确保变量被正确的读取，它必须是volatile类型的。如果我们原有代码中未申明这个类型，那么简单地申明一下就 行， 这不 会 引起 什么 问题。 
  * 第三，由于 CAS 操作 会 通过 对象 实例 中的 偏移量 直接进行 赋值， 因此， 它不 支持 static 字段（ Unsafe. objectFieldOffset() 不支持静态变量）

## ==Adder累加器==

* 介绍
  * 是Java 8引入的，相对是比较新的一个类。
  * 高并发下LongAdder比AtomicLong效率高，不过本质是空间换时间。
  * Atomic遇到的问题是，适合用在低并发场景，否则在高并发下，由于CAS的冲突机会大，会导致经常自旋，影响整体效率。而LongAdder引入了分段锁的概念，当竞争不激烈的时候，所有线程都是通过CAS对同一个变量（Base）进行修改，但是等到了竞争激烈的时候，LongAdder把不同线程对应到不同的Cell上进行修改，降低了冲突的概率，是多段锁的理念，提高了并发性
  
* 演示AtomicLong的问题
  * 这里演示多线程情况下AtomicLong的性能，有16个线程对同一个AtomicLong累加。
  
  * 由于竞争很激烈，每一次加法，都要flush和refresh（JMM），导致很耗费资源
  
    ```java
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import java.util.concurrent.atomic.AtomicLong;
    
    /**
     * 描述：     演示高并发场景下，LongAdder比AtomicLong性能好
     */
    public class AtomicLongDemo {
    
        public static void main(String[] args) throws InterruptedException {
            AtomicLong counter = new AtomicLong(0);
            ExecutorService service = Executors.newFixedThreadPool(20);
            long start = System.currentTimeMillis();
            for (int i = 0; i < 10000; i++) {
                service.submit(new Task(counter));
            }
            service.shutdown();
            while (!service.isTerminated()) {
    
            }
            long end = System.currentTimeMillis();
            System.out.println(counter.get());
            System.out.println("AtomicLong耗时：" + (end - start));
        }
    
        private static class Task implements Runnable {
    
            private AtomicLong counter;
    
            public Task(AtomicLong counter) {
                this.counter = counter;
            }
    
            @Override
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    counter.incrementAndGet();
                }
            }
        }
    }
    ```
  
    ```java
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import java.util.concurrent.atomic.AtomicLong;
    import java.util.concurrent.atomic.LongAdder;
    
    /**
     * 描述：     演示高并发场景下，LongAdder比AtomicLong性能好
     */
    public class LongAdderDemo {
    
        public static void main(String[] args) throws InterruptedException {
            LongAdder counter = new LongAdder();
            ExecutorService service = Executors.newFixedThreadPool(20);
            long start = System.currentTimeMillis();
            for (int i = 0; i < 10000; i++) {
                service.submit(new Task(counter));
            }
            service.shutdown();
            while (!service.isTerminated()) {
    
            }
            long end = System.currentTimeMillis();
            System.out.println(counter.sum());
            System.out.println("LongAdder耗时：" + (end - start));
        }
    
        private static class Task implements Runnable {
    
            private LongAdder counter;
    
            public Task(LongAdder counter) {
                this.counter = counter;
            }
    
            @Override
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    counter.increment();
                }
            }
        }
    }
    ```
  
* LongAdder带来的改进和原理
  
  ![image-20201009215633157](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201009215633157.png)
  
  ![image-20201009215702930](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201009215702930.png)
  
  * 在内部，这个LongAdder的实现原理和刚才的AtomicLong是有不同的，刚才的AtomicLong的实现原理是，每一次加法都需要做同步，所以在高并发的时候会导致冲突比较多，也就降低了效率
  * 而此时的LongAdder，每个线程会有自己的一个计数器，仅用来在自己线程内计数，这样一来就不会和其他线程的计数器干扰。
  * 如图中所示，第一个线程的计数器数值，也就是ctr’，为1的时候，可能线程2的计数器ctr’’的数值已经是3了，他们之间并不存在竞争关系,所以在加和的过程中，根本不需要同步机制，也不需要刚才的flush和refresh。这里也没有一个公共的counter来给所有线程统一计数。
  * 可能聪明的小伙伴已经想到了，LongAdder最终是如何实现多线程计数的呢？答案就在最后一步，执行LongAdder.sum()的时候，这里是唯一需要同步的地方：
  * LongAdder引入了分段累加的概念，内部有一个base变量和一个Cell[]数组共同参与计数
    * base变量：竞争不激烈，直接累加到该变量上
    * Cell[]数组：竞争激烈，各个线程分散累加到自己的槽Cell[i]中
  * 当我们执行sum函数的时候，LongAdder会把所有线程的计数器，也就是ctr’和ctr’’等等都在同步的情况下加起来，形成最终的总和：
  * AtomicLong在多线程的情况下，每次都要同步，而LongAdder仅在最后sum的时候需要同步，其他情况下，多个线程可以同时运行，这就是LongAdder的吞吐量比AtomicLong大的原因，本质是空间换时间
  * 对比AtomicLong和LongAdder
    * 在低争用下，AtomicLong和LongAdder这两个类具有相似的特征。但是在竞争激烈的情况下，LongAdder的预期吞吐量要高得多，但要消耗更多的空间
    * LongAdder适合的场景是统计求和计数的场景，而且LongAdder基本只提供了add方法，而AtomicLong还具有cas方法

## Accumulator累加器

* Accumulator和Adder非常相似，Accumulator就是一个更通用版本的Adder

* 用法

  ```java
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.concurrent.atomic.LongAccumulator;
  import java.util.stream.IntStream;
  
  /**
   * 描述：     演示LongAccumulator的用法
   */
  public class LongAccumulatorDemo {
  
      public static void main(String[] args) {
          LongAccumulator accumulator = new LongAccumulator((x, y) -> 2 + x * y, 1);
          ExecutorService executor = Executors.newFixedThreadPool(8);
          IntStream.range(1, 10).forEach(i -> executor.submit(() -> accumulator.accumulate(i)));
  
          executor.shutdown();
          while (!executor.isTerminated()) {
  
          }
          System.out.println(accumulator.getThenReset());
      }
  }
  ```

* LongAccumulator的构造函数的第一个参数是一个表达式，第二个参数是x的初始值。x是每次的初始值，y是结果。执行counter.accumulate(1)的时候，第一次x是0，y是1，后面每次的y的结果会赋值给x，然后每次的新y就是counter.accumulate(1)传入的1。

* 拓展功能

* 适用场景

  * 大量并行计算场景
  * 计算顺序不能成为瓶颈

# CAS

## 什么是CAS

* 思路
  * 并发
  * 我认为V的值应该是A，如果是的话那我就把它改成B，如果不是A（说明被别人修改过了），那我就不修改了，避免多人同时修改导致出错。
  * CAS有三个操作数：内存值V、预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才将内存值修改为B，否则什么都不做。最后返回现在的V值。
  * CPU的特殊原子指令
  
* CAS的等价代码（语义）

  ```java
  /**
   * 描述：     模拟CAS操作，等价代码
   */
  public class SimulatedCAS {
      private volatile int value;
  
      public synchronized int compareAndSwap(int expectedValue, int newValue) {
          int oldValue = value;
          if (oldValue == expectedValue) {
              value = newValue;
          }
          return oldValue;
      }
  }
  ```


## 案例演示

* 两个线程竞争，其中一个落败

  ```java
  import java.util.concurrent.ConcurrentHashMap;
  import java.util.concurrent.atomic.AtomicInteger;
  import java.util.concurrent.atomic.AtomicIntegerArray;
  
  /**
   * 描述：     模拟CAS操作，等价代码
   */
  public class TwoThreadsCompetition implements Runnable {
  
      private volatile int value;
  
      public synchronized int compareAndSwap(int expectedValue, int newValue) {
          int oldValue = value;
          if (oldValue == expectedValue) {
              value = newValue;
          }
          return oldValue;
      }
  
      public static void main(String[] args) throws InterruptedException {
          TwoThreadsCompetition r = new TwoThreadsCompetition();
          r.value = 0;
          Thread t1 = new Thread(r,"Thread 1");
          Thread t2 = new Thread(r,"Thread 2");
          t1.start();
          t2.start();
          t1.join();
          t2.join();
          System.out.println(r.value);
      }
  
      @Override
      public void run() {
          compareAndSwap(0, 1);
      }
  }
  ```

* 用CAS思想实现的计数器

## 应用场景

* 乐观锁
* 并发容器
* 原子类

## 以AtomicInteger为例，分析在Java中是如何利用CAS实现原子操作的？

* AtomicInteger加载Unsafe工具，用来直接操作内存数据
* 用Unsafe类来实现底层操作
  * Unsafe是CAS的核心类。Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。
  * valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的原值的，这样我们就能通过unsafe来实现CAS了。
* value是用volatile修饰的，保证了多线程之间看到的value值是同一份。
* getAndAdd方法
* Unsafe类中的compareAndSwapInt方法
  * 方法中先想办法拿到变量value在内存中的地址。
  * 通过Atomic::cmpxchg实现原子性的比较和替换，其中参数x是即将更新的值，参数e是原内存的值。至此，最终完成了CAS的全过程
* 分析在Java中是如何利用CAS实现原子操作的？
  * AtomicInteger加载Unsafe工具，用来直接操作内存数据
  * 用Unsafe来实现底层操作
  * 用volatile修饰value字段，保证可见性
  * getAndAddInt方法分析

## 缺点

* ABA问题
  * 可用数据库版本号字段记录来解决
* 自旋时间过长，消耗CPU资源
* 只能保证一个共享变量的原子操作
  * 当对一个共享变量执行操作时CAS能保证其原子性，如果对多个共享变量进行操作,CAS就不能保证其原子性，因为多个变量之间是独立的。有一个解决方案是利用对象整合多个共享变量，即一个类中的成员变量就是这几个共享变量。然后将这个对象做CAS操作就可以保证其原子性。atomic中提供了AtomicReference来保证引用对象之间的原子性

# 不变性

## 什么是不变性（Immutable）

* 如果对象在被创建后，状态就不能被修改，那么它就是不可变的。
* 具有不变性的对象一定是线程安全的，我们不需要对其采取任何额外的安全措施，也能保证线程安全

## final的作用

* 早期Java
  * 锁定
  * 效率
* 现在
  * 类防止被继承、方法防止被重写、变量防止被修改
  * 天生是线程安全的，而不需要额外的同步开销

## 3种用法：修饰变量、方法、类

* final修饰变量
  * 含义
    * 被final修饰的变量，意味着值不能被修改。如果变量是对象，那么对象的引用不能变，但是对象自身的内容依然可以变化。
  * 区分为3种
    * final instance variable（类中的final属性）
    * final static variable（类中的static final属性）
    * final local variable（方法中的final变量）
  * 赋值时机
    * 属性被声明为final后，该变量则只能被赋值一次。且一旦被赋值，final的变量就不能再被改变，如论如何也不会变。直到海角天涯也不会变心，非常好的品质
    * final instance variable（类中的final属性）
      * 只有3种赋值的时机或者说是途径：第一种是在声明变量的等号右边直接赋值，第二种就是构造函数中赋值，第三就是在类的初始代码块中赋值（不常用），如果不采用第一种赋值方法，那么就必须在第2 3种挑一个来赋值，而不能不赋值，这是final语法所规定的。
    * final static variable（类中的static final属性）
      * 两个赋值时机：除了在声明变量的等号右边直接赋值外，static final变量还可以用static初始代码块赋值，但是不能用普通的初始代码块赋值。
    * final local variable（方法中的final变量）
      * 和前面两种不同，由于这里的变量是在方法里的，所以没有构造函数，也不存在初始代码块。
      * final local variable不规定赋值时机，只要求在使用前必须赋值，这个方法中的非final变量的要求也是一样的。
  * 为什么要规定赋值时机
    * 我们来思考一下为什么语法要这继承这样？：如果初始化不赋值，后续赋值，就是从null变成你的赋值，这就违反final不变的原则了！
* final修饰方法（构造方法除外）
  * 不可被重写，也就是不能被override，即便是子类有同样名字的方法，那也不是override，这个和static方法是一个道理。（引申一下static方法不能被重写）
* final修饰类
  * 不可被继承，例如典型的String类就是final的，我们从没见过哪个类是继承String类的

## 注意点

* final修饰对象的时候，只是对象的引用不可变，而对象本身的属性是可以变化的
  * 代码演示
* final使用原则
  * 良好的编程习惯

## 不变性和final的关系

* 不变性并不意味着，简单地用final修饰就是不可变
  * 基本类型
  * 对象
  * String为例
  
* 如何利用final实现对象不可变

  * 把所有属性都声明为final？不对，属性中如果有对象，属性对象中的属性可能会变

  * 一个属性是对象，但是整体不可变，其他类无法修改set里面的数据

    ```java
    /**
     * 描述：     一个属性是对象，但是整体不可变，其他类无法修改set里面的数据
     */
    public class ImmutableDemo {
    
        private final Set<String> students = new HashSet<>();
    
        public ImmutableDemo() {
            students.add("李小美");
            students.add("王壮");
            students.add("徐福记");
        }
    
        public boolean isStudent(String name) {
            return students.contains(name);
        }
    }
    ```

* 总结出，满足以下条件时，对象才是不可变的
  * 对象创建后，其状态就不能修改
  * 所有属性都是final修饰的
  * 对象创建过程中没有发生逸出
  
* 把变量写在线程内部——栈封闭

  * 在方法里新建的局部变量，实际上是存储在每个线程私有的栈空间，而每个栈的占空间是不能被其他线程所访问到的，所以不会有线程安全问题，这就是著名的“栈封闭”技术，是“线程封闭”技术的一种情况

```java
/**
 * 描述：     演示栈封闭的两种情况，基本变量和对象 先演示线程争抢带来错误结果，然后把变量放到方法内，情况就变了
 */
public class StackConfinement implements Runnable {

    int index = 0;

    public void inThread() {
        int neverGoOut = 0;
        synchronized (this) {
            for (int i = 0; i < 10000; i++) {
                neverGoOut++;
            }
        }

        System.out.println("栈内保护的数字是线程安全的：" + neverGoOut);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            index++;
        }
        inThread();
    }

    public static void main(String[] args) throws InterruptedException {
        StackConfinement r1 = new StackConfinement();
        Thread thread1 = new Thread(r1);
        Thread thread2 = new Thread(r1);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(r1.index);
    }
}
```

```java
public class FinalStringDemo1 {

    public static void main(String[] args) {
        String a = "wukong2";
        final String b = "wukong";
        String d = "wukong";
        String c = b + 2;
        String e = d + 2;
        System.out.println((a == c));    // true
        System.out.println((a == e));    // false
    }
}
```

```java
public class FinalStringDemo2 {

    public static void main(String[] args) {
        String a = "wukong2";
        final String b = getDashixiong();
        String c = b + 2;
        System.out.println(a == c);     //false
    }

    private static String getDashixiong() {
        return "wukong";
    }
}
```

# ConcurrentHashMap等并发集合

## 并发容器概览

* ConcurrentHashMap：线程安全的HashMap
* CopyOnWriteArrayList: 线程安全的List
* BlockingQueue: 这是一个接口，表示阻塞队列，非常适合用于作为数据共享的通道。
* ConcurrentLinkedQueue：高效的非阻塞并发队列，使用链表实现。可以看做一个线程安全的LinkedList。
* ConcurrentSkipListMap: 是一个Map，使用跳表的数据结构进行快速查找

## 趣说集合类的历史——古老和过时的同步容器

* Vector和Hashtable
* ArrayList和HashMap
  * 虽然这两个类不是线程安全的，但是可以用Collections.synchronizedList(new ArrayList<E>())和Collections.synchronizedMap(new HashMap<K, V>())使之变成线程安全的
* ConcurrentHashMap和CopyOnWriteArrayList
  * 绝大多数并发情况下，ConcurrentHashMap和CopyOnWriteArrayList的性能都优于同步的HashMap和同步的ArrayList，唯一例外的是一个List经常被修改，那么同步的ArrayList性能会优于CopyOnWriteArrayList， CopyOnWriteArrayList更适合读多写少的场景，因为它每次写入都需要完整复制，比较消耗资源

## ConcurrentHashMap（重点、面试常考）

* 磨刀不误砍柴工：Map简介
  
  ![image-20200805074032004](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805074032004.png)
  
  ![image-20200805074101753](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805074101753.png)
  
  * HashMap
  * Hashtable
  * LinkedHashMap
  * TreeMap
  * 常用方法
  
* 为什么需要ConcurrentHashMap？
  * 为什么不用Collections.synchronizedMap()
    
    * Collections.synchronizedMap()是线程安全的，但是它是通过使用一个全局的锁来同步不同线程间的并发访问，因此会带来较大的性能问题。
    
  * 为什么HashMap是线程不安全的？
    * 同时put碰撞导致数据丢失
    
      * 两个key相同，会导致有一个线程的数据丢失
    
    * 同时put扩容导致数据丢失
    
      * 两个线程同时进行扩容，会导致一个线程数组数据丢失
    
    * 死循环造成的CPU100%
    
      ```java
      
      ```
    
    ![image-20200805075640326](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805075640326.png)
    
    ![image-20200805080029980](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080029980.png)
    
    * 彩蛋：调试技巧——如何修改JDK版本，从8到7
      * File-Project Structure，下载JDK7然后安装，然后添加JDK进来，然后选择。新建module选择JDK7，可以不影响到当前JDK8的代码。
    * 彩蛋：调式技巧——多线程配合，模拟真实场景
      * 如果不会这个调试技巧，永远都无法在本地模拟出线上bug的情况的，因为出现的几率太低了。
    * HashMap在高并发下的死循环（仅在JDK7及以前存在）
      * 代码演示
  
* 九层之台，起于累土、罗马不是一天建成的：HashMap分析
  * 结构图
  
    ![image-20200805080155435](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080155435.png)
  
    ![image-20200805080228114](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080228114.png)
  
    ![image-20200805080312931](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080312931.png)
  
    ![image-20200805080334294](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080334294.png)
  
  * 红黑树介绍
    
    * 对二叉查找树BST的一种平衡策略，O(logN) VS O(N)
    * 会自动平衡，防止极端不平衡从而影响查找效率的情况发生
      * 每个节点要么是红色，要么是黑色，但根节点永远是黑色的
      * 红色节点不能连续（也即是，红色节点的孩子和父亲都不能是红色）
      * 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点
      * 所有的叶节点都是黑色的
    * 红黑树是每个节点都带有颜色属性的二叉查找树，本质是对二叉查找树BST的一种平衡策略，颜色为红色或黑色。
    * 我们理解为是一种平衡二叉查找树就可以，查找效率高，会自动平衡，防止极端不平衡从而影响查找效率的情况发生。
    
  * HashMap关于并发的特点
  
    1. 非线程安全
    2. 迭代时不允许修改内容
    3. 只读的并发是安全的
    4. 如果一定要把HashMap用在并发环境，用Collections.synchronizedMap(new HashMap())
  
* JDK1.7的ConcurrentHashMap实现和分析
  
  ![image-20200805080851218](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080851218.png)
  
  ![image-20200805080932100](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805080932100.png)
  
  * 整体概念
    * Java 7中的ConcurrentHashMap最外层是多个segment，每个segment的底层数据结构与HashMap类似，仍然是数组和链表组成的拉链法。
    * 每个segment独立上ReentrantLock锁，每个segment之间互不影响，提高了并发效率。
    * ConcurrentHashMap 默认有 16 个 Segments，所以最多可以同时支持 16 个线程并发写（操作分别分布在不同的 Segment 上）。这个默认值可以在初始化的时候设置为其他值，但是一旦初始化以后，是不可以扩容的
  * Segment图解
  
* JDK1.8的ConcurrentHashMap实现和源码分析
  * 简介
  
  * 结构
  
    ![image-20200805081105556](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200805081105556.png)
  
  * 源码分析（1.8）
  
    * putVal流程
    * 判断key value不为空
      * 计算hash值
      * 根据对应位置节点的类型，来赋值，或者helpTransfer，或者增长链表，或者给红黑树增加节点
      * 检查满足阀值就“红黑树化”
      * 返回oldVal
    * get流程
      * 计算hash值
      * 找到对应的位置，根据情况进行：
      * 直接取值
      * 红黑树里找值
      * 遍历链表取值
      * 返回找到的结果
  
* 对比JDK1.7和1.8的优缺点（为什么要把1.7的结构改成1.8的结构）

  * 数据结构：并发度提高了
  * Hash碰撞
    * 拉链法
    * 先拉链法，再红黑树
  * 保证并发安全
  * 查询复杂度
  * 为什么超过8转为红黑树
    * 默认使用链表，因为占用内存更少
    * 达到8的概率很低，情况极端，千万分之一

* 组合操作：ConcurrentHashMap也不是线程安全的？

  * 多个线程同时get和put，线程不安全

  * 可以使用replace方法，线程安全，避免使用synchronized

  * putIfAbsent

    ```java
    /**
     * 描述：     组合操作并不保证线程安全
     */
    public class OptionsNotSafe implements Runnable {
    
        private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<String, Integer>();
    
        public static void main(String[] args) throws InterruptedException {
            scores.put("小明", 0);
            Thread t1 = new Thread(new OptionsNotSafe());
            Thread t2 = new Thread(new OptionsNotSafe());
            t1.start();
            t2.start();
            t1.join();
            t2.join();
            System.out.println(scores);
        }
    
    
        @Override
        public void run() {
            for (int i = 0; i < 1000; i++) {
                while (true) {
                    Integer score = scores.get("小明");
                    Integer newScore = score + 1;
                    boolean b = scores.replace("小明", score, newScore);
                    if (b) {
                        break;
                    }
                }
            }
    
        }
    }
    ```

* 实际生产案例

  * 并发场景使用ConcurrentHashMap

## CopyOnWriteArrayList

* 诞生的历史和原因
  
  * 代替Vector和SynchronizedList，就和ConcurrentHashMap代替SynchronizedMap的原因一样
  * Vector和SynchronizedList的锁的粒度太大，并发效率相对比较低，并且迭代时无法编辑
  * Copy-On-Write并发容器还包括CopyOnWriteArraySet，用来替代同步Set
  
* 适用场景
  * 读操作可以尽可能地快，而写即使慢一些也没有太大关系
  * 读多写少
    * 黑名单，每日更新
    * 监听器：迭代操作远多余修改操作
  
* 读写规则
  * 回顾读写锁
    * 读读共享、其他都互斥（写写互斥、读写互斥、写读互斥）
  * 读写锁规则的升级
    * 读取是完全不用加锁的，并且更厉害的是：写入也不会阻塞读取操作。只有写入和写入之间需要进行同步等待
  
* 代码演示

  ```java
  /**
   * 描述：演示CopyOnWriteArrayList可以在迭代的过程中修改数组内容，但是ArrayList不行，对比
   */
  public class CopyOnWriteArrayListDemo1 {
  
      public static void main(String[] args) {
          ArrayList<String> list = new ArrayList<>();
  //        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
  
          list.add("1");
          list.add("2");
          list.add("3");
          list.add("4");
          list.add("5");
  
          Iterator<String> iterator = list.iterator();
  
          while (iterator.hasNext()) {
              System.out.println("list is" + list);
              String next = iterator.next();
              System.out.println(next);
  
              if (next.equals("2")) {
                  list.remove("5");
              }
              if (next.equals("3")) {
                  list.add("3 found");
              }
          }
      }
  }
  ```

  ```java
  /**
   * 描述：     对比两个迭代器
   */
  public class CopyOnWriteArrayListDemo2 {
  
      public static void main(String[] args) throws InterruptedException {
          CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>(new Integer[]{1, 2, 3});
          System.out.println(list);
          Iterator<Integer> itr1 = list.iterator();
          list.remove(2);
          Thread.sleep(1000);
          System.out.println(list);
          Iterator<Integer> itr2 = list.iterator();
          itr1.forEachRemaining(System.out::println);
          itr2.forEachRemaining(System.out::println);
      }
  }
  ```

* 实现原理
  * CopyOnWrite的含义
  * 创建新副本、读写分离（使用不同容器）
  * “不可变”原理（旧的不可变）
  * 迭代的时候（迭代器使用的旧的）
  
* 缺点
  * 内存占用问题
    * 因为CopyOnWrite的写是复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存。
  * 数据一致性问题
    * CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。
  
* 源码分析
  * 数据结构
    * 底层是数组
  * get
    * get操作都没有加锁，保证了读取操作的高速。
  * add操作
    * 在添加的时候就上锁，并复制一个新数组，增加操作在新数组上完成，将array指向到新数组中，最后解锁。
  * 迭代器
    * 执行迭代操作的时候，操作的都是原数组，而原数组不会被修改（修改都会去修改副本数组），所以执行迭代操作不需要加锁，也不会抛异常

## 并发队列Queue（阻塞、非阻塞队列）

* 为什么要使用队列
  * 用队列可以安全地在线程间传递数据：生产者消费者模式、银行转账
  * 考虑锁等线程安全问题的重任从“你”转移到了“队列”上
  
* 并发队列简介
  * Queue
  * BlockingQueue
  
* 各并发队列关系图
  
  ![image-20201010082326363](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201010082326363.png)
  
  * 彩蛋：画漂亮的UML图
  
* 什么是阻塞队列

  * 简介、地位
    * 阻塞队列是具有阻塞功能的队列，所以它首先是一个队列。其次是具有阻塞功能
    * 通常，阻塞队列的一端是给生产者放数据用，另一端给消费者拿数据用。阻塞队列是线程安全的，所以生产者和消费者都可以是多线程的
    * 阻塞功能：最有特色的两个带有阻塞功能的方法是
      * take()方法：获取并移除队列的头结点，一旦如果执行take的时候，队列里无数据，则阻塞，直到队列里有数据
      * put()方法：插入元素。但是如果队列已满，那么就无法继续插入，则阻塞，直到队列里有了空闲空间
      * 是否有界（容量有多大）：这是一个非常重要的属性，无界队列意味着里面可以容纳非常多（Integer.MAX_VALUE，约为2的31次，是非常大的一个数，可以近似认为是无限容量）
      * 阻塞队列和线程池的关系：阻塞队列是线程池的重要组成部分

* BlockingQueue主要方法
  
  * put、take
  * add、remove、element
  * offer、poll、peek
  
* ArrayBlockingQueue 
  
  * 有界、指定容量、公平
    * 有10个面试者，一共只有1个面试官，大厅里有3个位子供面试者休息，每个人的面试时间是10秒，模拟所有人面试的场景
    * 源码分析：put方法
  
  ```java
  /**
   * 描述：     TODO
   */
  public class ArrayBlockingQueueDemo {
      public static void main(String[] args) {
          ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<String>(3);
          Interviewer r1 = new Interviewer(queue);
          Consumer r2 = new Consumer(queue);
          new Thread(r1).start();
          new Thread(r2).start();
      }
  }
  
  class Interviewer implements Runnable {
      BlockingQueue<String> queue;
      public Interviewer(BlockingQueue queue) {
          this.queue = queue;
      }
      @Override
      public void run() {
          System.out.println("10个候选人都来啦");
          for (int i = 0; i < 10; i++) {
              String candidate = "Candidate" + i;
              try {
                  queue.put(candidate);
                  System.out.println("安排好了" + candidate);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
          try {
              queue.put("stop");
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  
  class Consumer implements Runnable {
      BlockingQueue<String> queue;
      public Consumer(BlockingQueue queue) {
          this.queue = queue;
      }
      @Override
      public void run() {
          try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          String msg;
          try {
              while(!(msg = queue.take()).equals("stop")){
                  System.out.println(msg + "到了");
              }
              System.out.println("所有候选人都结束了");
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```
  
  
  
* LinkedBlockingQueue
  * 无界
  * 容量Integer.MAX_VALUE
  * 内部结构：Node、两把锁
  * 分析put方法

* PriorityBlockingQueue
  * 支持优先级
  * 自然顺序（而不是先进先出）
  * 无界队列
  * PriorityQueue 的线程安全版本
  
* SynchronousQueue
  * 功能
    * SynchronousQueue首先是一个阻塞队列，然后不同之处在于，它的容量为0 ，所以没有一个地方来暂存元素，导致每次取数据都要先阻塞，直到有数据被放入；同理，每次放数据的时候也会阻塞，直到有消费者来取。
    * 需要注意的是，SynchronousQueue的容量不是1而是0，因为SynchronousQueue不需要去持有元素，它所做的就是直接传递（direct handoff）。
    * 每当需要传递的时候，SynchronousQueue会把元素直接从生产者传给消费者，在此期间并不需要做存储，所以效率很高
  * 注意点
    1. SynchronousQueue没有peek等函数，因为peek的含义是取出头结点，但是SynchronousQueue的容量是0，所以连头结点都没有，也就没有peek方法。
    2. 同理，没有iterate相关方法。
    3. 是一个极好的用来直接传递的并发数据结构。
    4. SynchronousQueue是线程池Executors.newCachedThreadPool()使用的阻塞队列
  
* DelayQueue
  * 延迟队列，根据延迟时间排序
  * 元素需要实现Delayed接口，规定排序规则 
  
* 非阻塞队列ConcurrentLinkedQueue
  
  * 并发包中的非阻塞队列只有ConcurrentLinkedQueue这一种，顾名思义ConcurrentLinkedQueue是使用链表作为其数据结构的，使用 CAS 非阻塞算法来实现线程安全（不具备阻塞功能），适合用在对性能要求较高的并发场景。用的相对比较少一些
  
* 如何选择适合自己的队列？
  * 边界
  * 空间
  * 吞吐量

## 各并发容器总结

* java.util.concurrent包提供的容器，分为3类：`Concurrent*`、`CopyOnWrite*`、`Blocking*`
* `Concurrent*`的特点是大部分通过CAS实现并发，而`CopyOnWrite*`则是通过复制一份原数据来实现的，Blocking通过AQS实现的







