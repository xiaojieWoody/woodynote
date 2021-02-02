# 控制并发流程

* 控制并发流程的工具类，作用就是帮助我们程序员更容易的让线程之间合作
* 让线程之间相互配合，来满足业务逻辑
* 比如让线程A等待线程B执行完毕后再执行等合作策略

## 什么是控制并发流程？

* 有哪些控制并发流程的工具类
  * Semaphore
    * 信号量，可以通过控制“许可证”的数量，来保证线程之间的配合
    * 线程只有在拿到“许可证”后才能继续运行。相比于其他的同步器，更灵活
  * CyclicBarrier
    * 线程会等待，直到足够多线程达到了事先规定的数目。一旦达到触发条件，就可以进行下一步的动作
    * 适用于线程之间相互等待处理结果就绪的场景
  * Phaser
    * 和CyclicBarrier类似，但是计数可变，Java7加入的
  * CountDownLatch
    * 和CyclicBarrier类似，数量递减到0时，触发动作
    * 不可重复使用
  * Exchanger
    * 让两个线程在合适时交换对象
    * 适用场景：当两个线程工作在同一个类的不同实例上时，用于交换数据
  * Condition
    * 可以控制线程的“等待”和“唤醒”
    * 是Object.wait()的升级版

## CountDownLatch倒计时

* 作用

  * 并发流程控制的工具
    * 人满发车
    * 倒数结束之前，一直处于等待状态，直到倒计时结束了，此线程才继续工作

* 类的主要方法介绍

  * CountDownLatch(int count)：仅有一个构造函数，参数count为需要倒数的数值
  * await()：调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
  * countDown()：将count值减1，直到为0时，等待的线程会被唤起

* 两个典型用法

  * 用法一：一个线程等待多个线程都执行完毕，再继续自己的工作

    ```java
    /**
     * 描述：     工厂中，质检，5个工人检查，所有人都认为通过，才通过
     */
    public class CountDownLatchDemo1 {
        public static void main(String[] args) throws InterruptedException {
            CountDownLatch latch = new CountDownLatch(5);
            ExecutorService service = Executors.newFixedThreadPool(5);
            for (int i = 0; i < 5; i++) {
                final int no = i + 1;
                Runnable runnable = new Runnable() {
    
                    @Override
                    public void run() {
                        try {
                            Thread.sleep((long) (Math.random() * 10000));
                            System.out.println("No." + no + "完成了检查。");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        } finally {
                            latch.countDown();
                        }
                    }
                };
                service.submit(runnable);
            }
            System.out.println("等待5个人检查完.....");
            latch.await();
            System.out.println("所有人都完成了工作，进入下一个环节。");
        }
    }
    ```

  * 用法二：多个线程等待某一个线程的信号，同时开始执行

    ```java
    /**
     * 描述：     模拟100米跑步，5名选手都准备好了，只等裁判员一声令下，所有人同时开始跑步。
     */
    public class CountDownLatchDemo2 {
        public static void main(String[] args) throws InterruptedException {
            CountDownLatch begin = new CountDownLatch(1);
            ExecutorService service = Executors.newFixedThreadPool(5);
            for (int i = 0; i < 5; i++) {
                final int no = i + 1;
                Runnable runnable = new Runnable() {
                    @Override
                    public void run() {
                        System.out.println("No." + no + "准备完毕，等待发令枪");
                        try {
                            begin.await();
                            System.out.println("No." + no + "开始跑步了");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                };
                service.submit(runnable);
            }
            //裁判员检查发令枪...
            Thread.sleep(5000);
            System.out.println("发令枪响，比赛开始！");
            begin.countDown();
        }
    }
    ```

    ```java
    /**
     * 描述：     模拟100米跑步，5名选手都准备好了，只等裁判员一声令下，所有人同时开始跑步。当所有人都到终点后，比赛结束。
     */
    public class CountDownLatchDemo1And2 {
    
        public static void main(String[] args) throws InterruptedException {
            CountDownLatch begin = new CountDownLatch(1);
    
            CountDownLatch end = new CountDownLatch(5);
            ExecutorService service = Executors.newFixedThreadPool(5);
            for (int i = 0; i < 5; i++) {
                final int no = i + 1;
                Runnable runnable = new Runnable() {
                    @Override
                    public void run() {
                        System.out.println("No." + no + "准备完毕，等待发令枪");
                        try {
                            begin.await();
                            System.out.println("No." + no + "开始跑步了");
                            Thread.sleep((long) (Math.random() * 10000));
                            System.out.println("No." + no + "跑到终点了");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        } finally {
                            end.countDown();
                        }
                    }
                };
                service.submit(runnable);
            }
            //裁判员检查发令枪...
            Thread.sleep(5000);
            System.out.println("发令枪响，比赛开始！");
            begin.countDown();
    
            end.await();
            System.out.println("所有人到达终点，比赛结束");
        }
    }
    ```

* 注意点

  * 扩展用法：多个线程等多个线程完成执行后，再同时执行
  * CountDownLatch是不能够重用的，如果需要重新计数，可以考虑使用CyclicBarrier或者创建新的CountDownLatch实例 

* 原理、源码分析
  * 原理
    * AQS
  * 结构图
  * CountDownLatch源码分析
    * 这里对CountDownLatch类里面最重要的3个方法进行分析
    * 构造方法
      * CountDownLatch仅提供了一个构造方法，接收的参数就是需要计数的数量，直到countDown()方法被调用到了规定的次数，之前在等待的线程才会继续工作。
      * 源码
        * public CountDownLatch(int count) {    if (count < 0) throw new IllegalArgumentException("count < 0");    this.sync = new Sync(count);}
    * await()
      * 调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
    * countDown()
      * public void countDown() {    sync.releaseShared(1);}
  
* 总结
  
  * CountDownLatch类在创建实例的时候，需要传递倒数次数。
  * 而每一次线程调用了await()方法，state变量会减一，直到为0。
  * state为0的时候，之前等待的线程会继续运行。
  * CountDownLatch不能回滚重置
  * 两个典型用法：一等多和多等一
  * CountDonwLatch类在创建实例的时候，需要传递倒数次数。倒数到0的时候，之前等待的线程会继续运行
  * CountDownLatch不能回滚重置
  

## Semaphore信号量

* 介绍

  * Semaphore可以用来限制或管理数量有限的资源的使用情况。
  * 信号量的作用是维护一个“许可证”的计数，线程可以“获取”许可证，那信号量剩余的许可证就减一，线程也可以“是否”一个许可证，那信号量剩余的许可证就加一，当信号量所拥有的许可证数量为0，那么下一个还想要获取许可证的线程，就需要等待，直到有另外的线程释放了许可证

* 使用场景、应用实例

  * 背景
  * 正常情况下获取许可证
  * 没许可证时，会阻塞前来请求的线程
  * 有线程释放信号量后
  * 如果有两个线程释放许可证
  * 总结

* 用法

  * 使用流程

    1. 初始化Semaphore并指定许可证的数量
    2. 在需要被现在的代码前加acquire()或者acquireUninterruptibly()方法
    3. 在任务执行结束后，调用release()来释放许可证

  * 主要方法介绍
    * new Semaphore(int permits, boolean fair)
      * 这里可以设置是否要使用公平策略，如果传入true，那么Semaphore会把之前等待的线程放到FIFO的队列里，以便于当有了新的许可证，可以分发给之前等了最长时间的线程
    * tryAcquire()
      * 看看现在没有空闲的许可证，如果有的话就获取，如果没有的话也没关系，我不必陷入阻塞，我可以去做别的事情，过一会再来查看许可证的空闲情况
    * tryAcquire(timeout)
      * 和tryAcquire()一样，但是多了一个超时时间，比如“3秒内获取不到许可证，我就去做别的事情”
    * availablePermits
    * acquire()
    * acquireUninterruptibly()
    * release()
    
  * 示例代码

    ```java
    /**
     * 描述：     演示Semaphore用法
     */
    public class SemaphoreDemo {
        static Semaphore semaphore = new Semaphore(5, true);
        public static void main(String[] args) {
            ExecutorService service = Executors.newFixedThreadPool(50);
            for (int i = 0; i < 100; i++) {
                service.submit(new Task());
            }
            service.shutdown();
        }
    
        static class Task implements Runnable {
            @Override
            public void run() {
                try {
                    semaphore.acquire(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "拿到了许可证");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "释放了许可证");
                semaphore.release(2);
            }
        }
    }
    ```

  * 特殊用法
    * 一次性获取或释放多个许可证
      * 使用场景
        * 比如TaskA会调用很消耗资源的method1()，而TaskB调用的是不太消耗资源的method2()，假设我们一共有5个许可证。那么我们就可以要求TaskA获取5个许可证才能执行，而TaskB只需要获取到一个许可证就能执行，这样就避免了A和B同时运行的情况，我们可以根据自己的需求合理分配资源

* 注意点

  1. 获取和释放的许可证数量必须一致，否则比如每次都获取2个但是只释放1个甚至不释放，随着时间的推移，到最后许可证数量不够用，会导致程序卡死。（虽然信号量类并不对是否和获取的数量做规定，但是这是我们的编程规范，否则容易出错）

  2. 注意在初始化Semaphore的时候设置公平性，一般设置为true会更合理
  3. 并不是必须由获取许可证的线程释放那个许可证，事实上，获取和释放许可证对线程并无要求，也许是A获取了，然后由B释放，只要逻辑合理即可。
  4. 信号量的作用，除了控制临界区最多同时有N个线程访问外，另一个作用是可以实现“条件等待”，例如线程1需要在线程2完成准备工作后才能开始工作，那么就线程1acquire()，而线程2完成任务后release()，这样的话，相当于是轻量级的CountDownLatch。
  5. 可以跨线程/跨线程池共享同一个信号量

## Condition接口（又称条件对象）

* 作用
  * 当线程1需要等待某个条件的时候，它就去执行condition.await()方法，一旦执行了await()方法，线程就会进入阻塞状态。
  * 然后通常会有另外一个线程，假设是线程2，去执行对应的条件，直到这个条件达成的时候，线程2就会去执行condition.signal()方法，这时JVM就会从被阻塞的线程里找，找到那些等待该condition的线程，当线程1就会收到可执行信号的时候，它的线程状态就会变成Runnable可执行状态。
  * signalAll()和signal()区别
    * signalAll()会唤起所有的正在等待的线程。
    * 但是signal()是公平的，只会唤起那个等待时间最长的线程，在当前情况下，是thread-0。
  
* 代码演示
  * 普通示例
  
    ```java
    /**
     * 描述：     演示Condition的基本用法
     */
    public class ConditionDemo1 {
        private ReentrantLock lock = new ReentrantLock();
        private Condition condition = lock.newCondition();
    
        void method1() throws InterruptedException {
            lock.lock();
            try{
                System.out.println("条件不满足，开始await");
                condition.await();
                System.out.println("条件满足了，开始执行后续的任务");
            }finally {
                lock.unlock();
            }
        }
    
        void method2() {
            lock.lock();
            try{
                System.out.println("准备工作完成，唤醒其他的线程");
                condition.signal();
            }finally {
                lock.unlock();
            }
        }
    
        public static void main(String[] args) throws InterruptedException {
            ConditionDemo1 conditionDemo1 = new ConditionDemo1();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                        conditionDemo1.method2();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
            conditionDemo1.method1();
        }
    }
    ```
  
  * 用Condition实现生产者消费者模式
  
    ```java
    /**
     * 描述：     演示用Condition实现生产者消费者模式
     */
    public class ConditionDemo2 {
    
        private int queueSize = 10;
        private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
        private Lock lock = new ReentrantLock();
        private Condition notFull = lock.newCondition();
        private Condition notEmpty = lock.newCondition();
    
        public static void main(String[] args) {
            ConditionDemo2 conditionDemo2 = new ConditionDemo2();
            Producer producer = conditionDemo2.new Producer();
            Consumer consumer = conditionDemo2.new Consumer();
            producer.start();
            consumer.start();
        }
    
        class Consumer extends Thread {
    
            @Override
            public void run() {
                consume();
            }
    
            private void consume() {
                while (true) {
                    lock.lock();
                    try {
                        while (queue.size() == 0) {
                            System.out.println("队列空，等待数据");
                            try {
                                notEmpty.await();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        queue.poll();
                        notFull.signalAll();
                        System.out.println("从队列里取走了一个数据，队列剩余" + queue.size() + "个元素");
                    } finally {
                        lock.unlock();
                    }
                }
            }
        }
    
        class Producer extends Thread {
    
            @Override
            public void run() {
                produce();
            }
    
            private void produce() {
                while (true) {
                    lock.lock();
                    try {
                        while (queue.size() == queueSize) {
                            System.out.println("队列满，等待有空余");
                            try {
                                notFull.await();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        queue.offer(1);
                        notEmpty.signalAll();
                        System.out.println("向队列插入了一个元素，队列剩余空间" + (queueSize - queue.size()));
                    } finally {
                        lock.unlock();
                    }
                }
            }
        }
    }
    ```
  
* 注意点

  * 实际上，如果说Lock用来代替synchronized，那么Condition就是用来代替相对应的Object.wait/notify的，所以在用法和性质上，几乎都一样
  * await方法会自动释放持有的Lock锁，和Object.wait一样，不需要自己手动先释放锁
  * 调用await的时候，必须持有锁，否则会抛出异常，和Object.wait一样

* 源码分析

* 面试常见问题
  
  * Condition和object.wait()和notify()的关系？

## CyclicBarrier循环栅栏

* 作用

  * CyclicBarrier循环栅栏和CountDownLatch很类似，都能阻塞一组线程
  * 当有大量线程相互配合，分别计算不同任务，并且需要最后统一汇总的时候，可以使用CyclicBarrier。CyclicBarrier可以构造一个集结点，当某一个线程执行完毕，它就会到集结点等待，直到所有线程都到了集结点，那么该栅栏就被撤销，所有线程再统一出发，继续执行剩下的任务
  * 生活中的例子：“咋们3个人明天中午在学校碰面，都到齐后，一起讨论下学期的计划”

* 代码例子

  ```java
  /**
   * 描述：    演示CyclicBarrier
   */
  public class CyclicBarrierDemo {
      public static void main(String[] args) {
          CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {
              @Override
              public void run() {
                  System.out.println("所有人都到场了， 大家统一出发！");
              }
          });
          for (int i = 0; i < 10; i++) {
              new Thread(new Task(i, cyclicBarrier)).start();
          }
      }
  
      static class Task implements Runnable{
          private int id;
          private CyclicBarrier cyclicBarrier;
  
          public Task(int id, CyclicBarrier cyclicBarrier) {
              this.id = id;
              this.cyclicBarrier = cyclicBarrier;
          }
  
          @Override
          public void run() {
              System.out.println("线程" + id + "现在前往集合地点");
              try {
                  Thread.sleep((long) (Math.random()*10000));
                  System.out.println("线程"+id+"到了集合地点，开始等待其他人到达");
                  cyclicBarrier.await();
                  System.out.println("线程"+id+"出发了");
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } catch (BrokenBarrierException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```

* 面试常见问题
  
  * CyclicBarriar 和 CountdownLatch 有什么区别？
  * 作用不同：CyclicBarrier要等固定数量的线程都到达了栅栏位置才能继续执行，而CountDownLatch只需等待数字到0，也就是说，CountDownLatch用于事件，但是CyclicBarrier是用于线程的
    * 可重用性不同：CountDownLatch在倒数到0并触发门闩打开后，就不能再次使用了，除非新建新的实例；而CyclicBarrier可以重复使用

# ==AQS==

* 学习AQS的思路
  * 学习AQS的目的主要是想理解原理、提高技术，以及应对面试
  * 先从应用层面理解为什么需要它、如何使用它，然后再看一看我们Java代码的设计者是如何使用它的了解它的应用场景
  * 这样之后我们再去分析它的结构，这样的话我们就学习得更加轻松了

## 学习AQS的思路

* 我们不是上来就看代码和方法，因为AQS代码很复杂，有很多细节，而我们作为普通开发者（不是JDK开发者），通常都不需要也不会直接使用AQS，因为JDK提供给我们封装好的线程协作工具类已经足够我们使用了，我们学习AQS的目的主要是想理解原理、提高技术，以及应对面试。
* 如果直接克看的会把自己给绕晕，所以我们的学习思路并不是先去看源码，而是说先从应用层面理解为什么需要他如何使用它，然后再看一看我们Java代码的设计者是如何使用它的了解它的应用场景。
* 这样之后我们再去分析它的结构，这样的话我们就学习得更加轻松了

## 为什么需要AQS？

* 锁和协作类有共同点：闸门
  * 我们已经学过了ReentrantLock和Semaphore，有没有发现它们有共同点？很相似？
  * 事实上，不仅是ReentrantLock和Semaphore，包括CountDownLatch、ReentrantReadWriteLock都有这样类似的“协作”（或者叫“同步”）功能，其实，它们底层都用了一个共同的基类，这就是AQS。
  * 为什么需要AQS？：因为上面的那些协作类，它们有很多工作都是类似的，所以如果能提取出一个工具类，那么就可以直接用，对于ReentrantLock和Semaphore而言就可以屏蔽很多细节，只关注它们自己的“业务逻辑”就可以了。
* Semaphore和AQS的关系
  * Semaphore内部有一个Sync类，Sync类继承了AQS。
  * CountDownLatch也是一样的，展示代码。
* 比喻
  * 群面、单面
  * 安排就坐、叫号、先来后到等公共事务就是AQS做的工作
  * 面试官不会去关心两个面试者是不是号码相同冲突了，也不想去管面试者需要一个地方坐着休息，这些都交给HR去做 
  * Semaphore：一个人面试完了以后，后一个人才能进来继续面试
  * CountDownLatch：群面，等待10人到齐
  * Semaphore、CountDownLatch这些同步工具类，要做的，就只是写下自己的“要人”规则。比如是“出一个，进一个”，或者说“凑齐10人，一起面试”
  * 剩下的招呼面试者的脏活累活交给AQS来做
  
* 如果没有AQS
  * 就需要每个协作工具自己实现：
    * 同步状态的原子性管理；
    * 线程的阻塞与解除阻塞；
    * 队列的管理；
  * 在并发场景下，自己 正确且高效实现这些内容，是相当有难度的，所以我们用AQS来帮我们把这些脏活累活都搞定，我们只关注业务逻辑就够了

## AQS的作用

* AQS是一个用于构建锁、同步器、协作工具类的工具类（框架）。有了AQS以后，更多的协作工具类都可以很方便得被写出来，
* 一句话总结：有了AQS，构建线程协作类就容易多了

## AQS的重要性、地位

* AbstractQueuedSynchronizer是Doug Lea写的，从JDK1.5加入的一个基于FIFO等待队列实现的一个用于实现同步器的基础框架，我们用IDE看AQS的实现类，可以发现实现类有以下这些：
  * 可以看出，有以下工具类使用到了AQS：
  * ReentrantLock的Sync锁、FairSync公平锁、NonfairSync非公平锁
  * ReentrantReadWriteLock的Sync锁、FairSync公平锁、NonfairSync非公平锁
  * Semaphore的Sync锁、FairSync公平锁、NonfairSync非公平锁
  * CountDownLatch的Sync锁
  * ThreadPoolExecutor的Worker
  * 可以看出，JCU包里面几乎所有的有关锁、多线程并发以及线程同步器等重要组件的实现都是基于AQS这个框架，重要性不言而喻
  
  ![image-20201012081049458](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201012081049458.png)

## AQS内部原理解析

* AQS最核心的就是三大部分：state、期望协作工具类去实现的获取/释放等重要方法、控制线程抢锁和配合的FIFO队列。我们对着3个模块分别展开：
* state状态
  * state用来表示“锁”的占有情况，包括可重入计数
  * 当state的值为0的时候，标识改Lock不被任何线程所占有
  * AQS的核心思想是基于volatile int state这样的一个属性同时配合Unsafe工具对其原子性的操作，来实现对当前锁的状态进行修改。当state的值为0的时候，标识改Lock不被任何线程所占有。
  * 这里的state的具体含义，会根据具体实现类的不同而不同，比如在Semaphore里，它表示“剩余的许可证的数量”，而在CountDownLatch里，它表示“还需要倒数的数量”。
  * state是volatile修饰的，会被并发地修改，所以所有修改state的方法都需要保证线程安全，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于j.u.c.atomic包的支持。
  * 这里可能会疑惑，那你setState的时候，没有做任何并发处理啊，这里就要说到volatile的作用了

* 控制线程抢锁和配合的FIFO队列
  * 这个队列用来存放“等待的线程”，AQS就是“排队管理器”，当多个线程争用同一把锁时，必须有排队机制将那些没能拿到锁的线程串在一起。当锁释放时，锁管理器就会挑选一个合适的线程来占有这个刚刚释放的锁
  * AQS会维护一个等待的线程队列，把线程都放到这个队列里
  * 这是一个双向形式的队列
* 获取/释放方法
  * 这里的获取和释放方法，是利用AQS的协作工具类里最重要的方法，是由协作类自己去实现的，并且含义各不相同。
  * 获取方法
    * 获取操作会依赖state变量，经常会阻塞（比如获取不到的时候）。
    * 在Semaphore中，获取就是acquire方法，作用是获取一个许可证；而在CountDownLatch里面，获取就是await方法，作用是“等待，直到倒数结束”。
  * 释放方法
    * 释放操作不会阻塞，在Semaphore中，释放就是release方法，作用是释放一个许可证；CountDownLatch里面，获取就是countDown方法，作用是“倒数1个数”。
  * 需要重写tryAcquire和tryRelease等方法
    * 在我们刚才讲的获取/释放方法中，里面需要调用继承了AQS的Sync类的acquire和release（独占锁例如ReentrantLock就是acquire和release，共享锁比如CountDownLatch就是acquireShared和releaseShared）方法，这里需要我们实现的方法，根据业务逻辑不同，有不同的实现

## 应用实例解析

* AQS用法
  * 分为两步：
  * 第一步：写一个类，想好协作的逻辑，实现获取/释放方法。
  * 第二步：内部写一个Sync类继承AbstractQueuedSynchronizer，然后根据是否独占来重写tryAcquire/tryRelease或tryAcquireShared (int acquires)和tryReleaseShared(int releases)等方法，在之前写的获取/释放方法中调用AQS的acquire/release或者Shared方法。
* AQS在CountDownLatch的应用
  * 构造函数
    * public CountDownLatch(int count) {    if (count < 0) throw new IllegalArgumentException("count < 0");    this.sync = new Sync(count);}
    * 可以看出，这里给Sync赋值了count，看到Sync的构造函数，正是给state变量去赋值了：
    * Sync(int count) {    setState(count);}
    * 所以在CountDownLatch里面的state，就是代表我们希望它倒数的次数。
  * getCount
    * 一步步看看出最后还是去获取state变量的值。
  * countDown
    * 这个就是“释放”方法，会调用到CountDownLatch重写的tryReleaseShared方法，分析tryReleaseShared方法：
    * 如果返回true，就释放所有之前阻塞的线程。
    * 当state为0的时候，说明之前已经释放过了，本次就不需要重复释放了。
    * 举例：
      * 如果c为5，nextc就是4，这个时候state会被设置为4，但是不符合nextc == 0，所以不释放锁。
      * 如果c为1，nextc就是0，这个时候state会被设置为0，并且符合nextc == 0，所以打开栅栏，把之前阻塞的线程通通放开。
  * await方法
    * 同理，await和刚才的countDown是形成配对的，是“获取”方法，追踪源码可以看到，会调用到tryAcquireShared方法，这个方法如下：
    * 就是看state是不是0，不为0就返回-1，所以在acquireSharedInterruptibly方法中：
    * 就会调用doAcquireSharedInterruptibly方法，随即进入阻塞状态。
    * 如果state已经是0了，那么就不会调用doAcquireSharedInterruptibly，线程也就不会阻塞，相当于是倒数已经结束了，立刻放行。
  * AQS在CountDownLatch的总结
    * 调用CountDownLatch的await方法时，便会尝试获取“共享锁”，不过一开始是获取不到该锁的，于是线程被阻塞
    * 而“共享锁”可获取到的条件，就是“锁计数器”的值为0
    * 而“锁计数器”的初始值为count，每当一个线程调用该CountDownLatch对象的countDown()方法时，才将“锁计数器”-1
    * count个线程调用countDown()之后，“锁计数器”才为0，而前面提到的等待获取共享锁的线程才能继续运行
  
* AQS在Semaphore的应用
  * 在Semaphore中，state表示许可证的剩余数量，
  * 看tryAcquire方法，判断nonfairTryAcquireShared大于等于0的话，代表成功，然后看：
  * 这里会先检查剩余许可证数量够不够这次需要的，用减法来计算，如果直接不够，那就返回负数，表示失败，如果够了，就用自旋加compareAndSetState来改变state状态，直到改变成功就返回正数；或者是期间如果被其他人修改了导致剩余数量不够了，那也返回负数代表获取失败。
* AQS在ReentrantLock的应用
  * 分析释放锁的方法tryRelease
    * 由于是可重入的，所以state代表重入的次数，每次释放锁，先判断是不是当前持有锁的线程释放的，如果不是就抛异常，如果是的话，重入次数就减一，如果减到了0，就说明完全释放了，于是free就是true，并且把state设置为0。
  * 加锁的方法
    * 判断当前state是否等于0，判断当前线程是否是持有锁的线程，如果都不是，则目前没有拿到这把锁，放到队列里等待，在合适的时机唤醒

## 利用AQS实现一个自己的Latch门闩

```java
/**
 * 描述：     自己用AQS实现一个简单的线程协作器
 */
public class OneShotLatch {
    private final Sync sync = new Sync();
    public void signal() {
        sync.releaseShared(0);
    }
    public void await() {
        sync.acquireShared(0);
    }

    private class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected int tryAcquireShared(int arg) {
            return (getState() == 1) ? 1 : -1;
        }
        @Override
        protected boolean tryReleaseShared(int arg) {
           setState(1);
           return true;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        OneShotLatch oneShotLatch = new OneShotLatch();
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName()+"尝试获取latch，获取失败那就等待");
                    oneShotLatch.await();
                    System.out.println("开闸放行"+Thread.currentThread().getName()+"继续运行");
                }
            }).start();
        }
        Thread.sleep(5000);
        oneShotLatch.signal();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"尝试获取latch，获取失败那就等待");
                oneShotLatch.await();
                System.out.println("开闸放行"+Thread.currentThread().getName()+"继续运行");
            }
        }).start();
    }
}
```

# 获取子线程的执行结果

* Future和Callable——治理线程第二大法宝

## Runnable的缺陷

1. 不能返回一个返回值
2. 也不能抛出checked Exception

* 为什么有这样的缺陷
  * 查看Runnable接口可以发现，run()方法的返回是void，且未声明为抛出任何已检查的异常，而咱们实现并重写这个方法，自然也不能返回值，也不能抛出异常，因为在对应的Interface / Superclass中没有声明它。
* Runnable为什么设计成这样？
  * 如果run()方法可以返回值，或者可以抛出异常，也无济于事，因为我们并没办法在外层捕获并处理，这是因为调用run()方法的类（比如Thread类和线程池）也不是我们编写的。所以如果我们真的想达到这个目的，可以看看下面的补救措施：
* 针对缺陷的补救措施
  * 使用Callable，看源码，声明了抛出异常
  * 用Future来获得子线程的运行结果

## Callable接口

* Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其它线程执行的任务。 		
* 实现Callable接口，就要实现call方法，这个方法的返回值是Object，所以返回的结果可以放在Object对象中，所以可以利用Callable的call方法的返回值来获得其他线程的执行结果

## Future类

* Future的作用
  
  * Future的核心思想是：一个方法的计算过程可能非常耗时，一直在原地等待方法返回，显然不明智。可以把该计算过程放到子线程去执行，并通过Future去控制方法的计算过程，在计算出结果后直接获取该结果。
  
* Callable和Future的关系
  * 我们可以用Future.get来获取Callable接口返回的执行结果，还可以通过Future.isDone()来判断任务是否已经执行完了，以及取消这个任务，限时获取任务的结果等。而如果任务没有执行完，future.get()会阻塞调用的线程直到任务执行完后返回结果。。
  * 在call()未执行完毕之前，调用get()的线程（假定此时是主线程）会被block，也就是进入到block状态，直到call()方法返回了结果后，此时future.get()才会得到该结果，然后主线程才会从block状态切换到runnable状态。
  * 所以Future是一个存储器，它存储了call()这个任务的结果，而这个任务的执行时间是无法提前确定的，因为这完全取决于call()方法执行的情况。
  
* Future的方法介绍
  * get()方法：获取结果

    * get方法的行为取决于Callable任务的状态，只有以下这5种情况：
    1. 任务正常完成：get方法会立刻返回结果
      2. 任务尚未完成（任务还没开始或进行中）：get将阻塞并直到任务完成
    3. 任务执行过程中抛出Exception：get方法会抛出ExecutionException：这里的抛出异常，是call()执行时产生的那个异常，看到这个异常类型是java.util.concurrent.ExecutionException。不论call()执行时抛出的异常类型是什么，最后get方法抛出的异常类型都是ExecutionException
      4. 任务被取消：get方法会抛出CancellationException
    5. 任务超时：get方法有一个重载方法，是传入一个延迟时间的，如果时间到了还没有获得结果，get方法就会抛出TimeoutException

  * get(long timeout, TimeUnit unit)方法：有超时的获取结果

    * 超时的需求很常见
    * 用get(long timeout, TimeUnit unit)方法时，如果call()在规定时间内完成了任务，那么就会正常获取到返回值；而如果再指定时间内没有计算出结果，那么就会抛出TimeoutException
    * 超时不获取，任务需取消

  * cancel()方法：取消任务的执行

    * cancel传入true和false的区别，代表是否中断正在执行的任务

      ```java
      /**
       * 描述：     演示get的超时方法，需要注意超时后处理，调用future.cancel()。演示cancel传入true和false的区别，代表是否中断正在执行的任务。
       */
      public class Timeout {
      
          private static final Ad DEFAULT_AD = new Ad("无网络时候的默认广告");
          private static final ExecutorService exec = Executors.newFixedThreadPool(10);
      
          static class Ad {
      
              String name;
      
              public Ad(String name) {
                  this.name = name;
              }
      
              @Override
              public String toString() {
                  return "Ad{" +
                          "name='" + name + '\'' +
                          '}';
              }
          }
      
      
          static class FetchAdTask implements Callable<Ad> {
      
              @Override
              public Ad call() throws Exception {
                  try {
                      Thread.sleep(3000);
                  } catch (InterruptedException e) {
                      System.out.println("sleep期间被中断了");
                      return new Ad("被中断时候的默认广告");
                  }
                  return new Ad("旅游订票哪家强？找某程");
              }
          }
      
      
          public void printAd() {
              Future<Ad> f = exec.submit(new FetchAdTask());
              Ad ad;
              try {
                  ad = f.get(2000, TimeUnit.MILLISECONDS);
              } catch (InterruptedException e) {
                  ad = new Ad("被中断时候的默认广告");
              } catch (ExecutionException e) {
                  ad = new Ad("异常时候的默认广告");
              } catch (TimeoutException e) {
                  ad = new Ad("超时时候的默认广告");
                  System.out.println("超时，未获取到广告");
                  boolean cancel = f.cancel(true);
                  System.out.println("cancel的结果：" + cancel);
              }
              exec.shutdown();
              System.out.println(ad);
          }
      
          public static void main(String[] args) {
              Timeout timeout = new Timeout();
              timeout.printAd();
          }
      }
      ```

    * cancel方法：取消任务的执行

      1. 如果这个任务还没有开始执行，那么这种情况最简单，任务会被正常的取消，未来也不会被执行，方法返回true
      2. 如果任务已完成，或者已取消：那么cancel()方法会执行失败，方法返回false
      3. 如果这个任务已经开始执行了，那么这个取消方法将不会直接取消该任务，而是会根据我们填的参数mayInterruptIfRunning做判断

      * Future.cancel(true)适用于：
        1. 任务能够处理interrupt
      * Future.cancel(false)仅用于避免启动尚未启动的任务，适用于：
        1. 未能处理interrupt的任务
        2. 不清楚任务是否支持取消
        3. 需要等待已经开始的任务执行完成

  * isDone()方法：判断线程是否执行完毕

  * isCancelled()方法：判断是否被取消

* 用法1：线程池的submit方法返回Future对象

  * 首先，要给线程池提交我们的任务，提交时线程池会立刻返回一个空的Future容器。当线程的任务一旦执行完毕，也就是当我们可以获取结果的时候，线程池便会把该结果填入到之前给我们的那个Future中去（而不是创建一个新的Future），我们此时便可以从该Future中获得任务执行的结果

* 用法2：用FutureTask来创建Future

  * 用FutureTask来获取Future和任务的结果
  * FutureTask是一种包装器，可以把Callable转化成Future和Runnable，它同时实现二者的接口
    * FutureTask -> RunnableFuture -> (Runnable、Future)
    * 把Callable实例当作参数，生成FutureTask的对象，然后把这个对象当作一个Runnable对象，用线程池或另起线程去执行这个Runnable对象，最后通过FutureTask获取刚才执行的结果

```java
/**
 * 描述：     演示FutureTask的用法
 */
public class FutureTaskDemo {s
    public static void main(String[] args) {
        Task task = new Task();
        FutureTask<Integer> integerFutureTask = new FutureTask<>(task);
//        new Thread(integerFutureTask).start();
        ExecutorService service = Executors.newCachedThreadPool();
        service.submit(integerFutureTask);
        try {
            System.out.println("task运行结果："+integerFutureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程正在计算");
        Thread.sleep(3000);
        int sum = 0;
        for (int i = 0; i < 100; i++) {
            sum += i;
        }
        return sum;
    }
}
```

## 案例：Future应用场景，在写一个浏览器程序的时候

* 串行
* 并行		
* 升级：有超时地获取结果

## Future的注意点

* 当for循环批量获取future的结果时，容易block，get方法调用时应使用timeout限制
* Future和Callable的生命周期不能后退

# 从0到1打造高性能缓存

## 本章介绍

* 本章中，我们将自己一步步实现一个缓存，用来使用和巩固我们之前学到过的并发知识。
* 缓存是在实际生产中非常常用的工具，用了缓存以后，我们可以避免重复计算，提高吞吐量。
* 使用缓存的代价是会额外占用一些内存，不过这个代价并不高，相比于收益而言，代价可以说是微不足道的，所以学好缓存是很有必要的。
* 虽然缓存乍一看很简单，不就是一个Map吗？最初级的缓存确实可以用一个Map来实现，不过一个功能完备、性能强劲的缓存，需要考虑的点就非常多了，我们从最简单的HashMap入手，一步步提高我们缓存的性能

## 从最简单版缓存入手——HashMap

* 这种办法，是最初级的缓存，可以用，但是无法并发，因为线程不安全

## 并发安全要保证——引出用synchronized实现

* 性能差
  * compute方法的synchronized关键字是必须加的，否则多个线程同时到compute的时候，由于HashMap是线程不安全的，所以如果多个线程同时put、get，会带来线程安全问题，所以这里用synchronized来保证每个时刻最多只有一个线程能访问，但是显而易见，这带来了性能问题。当多个线程同时想计算的时候，需要慢慢等待，严重时，性能甚至比不用缓存更差。
* 代码复用能力差
  * 代码的复用能力很差，如果第二个类需要用缓存，难道要重新加一个HashMap，然后再加上compute方法吗？这样对代码的侵入性太高了，而且一旦我们的compute逻辑有变动，就要在之前使用了缓存的所有类中都一个个做出修改，违反了开闭原则，不可取

## 给HashMap加final关键字

* 属性被声明为final后，该变量则只能被赋值一次。且一旦被赋值，final的变量就不能再被改变。
* 我们的类中Map的不需要可变，所以我们把它加上final关键字，增强安全性

## 代码有重构空间——用装饰者模式

* 代码修改
  
  * 我们假设ExpensiveFunction类是耗时计算的实现类，实现了Computable接口，但是其本身不具备缓存功能，也不需要考虑缓存的事情。
  
    ```java
    /**
     * 描述：     有一个计算函数computer，用来代表耗时计算，每个计算器都要实现这个接口，这样就可以无侵入实现缓存功能
     */
    public interface Computable <A,V>{
        V compute(A arg) throws Exception;
    }
    
    /**
     * 描述：     耗时计算的实现类，实现了Computable接口，但是本身不具备缓存能力，不需要考虑缓存的事情
     */
    public class ExpensiveFunction implements Computable<String, Integer>{
        @Override
        public Integer compute(String arg) throws Exception {
            Thread.sleep(5000);
            return Integer.valueOf(arg);
        }
    }
    
    /**
     * 描述：     用装饰者模式，给计算器自动添加缓存功能
     */
    public class ImoocCache2<A,V> implements Computable<A,V> {
    
        private final Map<A, V> cache = new HashMap();
        private  final Computable<A,V> c;
        public ImoocCache2(Computable<A, V> c) {
            this.c = c;
        }
    
        @Override
        public synchronized V compute(A arg) throws Exception {
            System.out.println("进入缓存机制");
            V result = cache.get(arg);
            if (result == null) {
                result = c.compute(arg);
                cache.put(arg, result);
            }
            return result;
        }
    
        public static void main(String[] args) throws Exception {
            ImoocCache2<String, Integer> expensiveComputer = new ImoocCache2<>(
                    new ExpensiveFunction());
            Integer result = expensiveComputer.compute("666");
            System.out.println("第一次计算结果："+result);
            result = expensiveComputer.compute("13");
            System.out.println("第二次计算结果："+result);
        }
    }
    ```
  
* 缺点
  * 性能差
  * 当多个线程同时想计算的时候，需要慢慢等待，严重时，性能甚至比不用缓存更差
  
  ```java
  /**
   * 描述：     用装饰者模式，给计算器自动添加缓存功能
   */
  public class ImoocCache3<A,V> implements Computable<A,V> {
      private final Map<A, V> cache = new HashMap();
      private  final Computable<A,V> c;
      public ImoocCache3(Computable<A, V> c) {
          this.c = c;
      }
  
      @Override
      public synchronized V compute(A arg) throws Exception {
          System.out.println("进入缓存机制");
          V result = cache.get(arg);
          if (result == null) {
              result = c.compute(arg);
              cache.put(arg, result);
          }
          return result;
      }
  
      public static void main(String[] args) throws Exception {
          ImoocCache3<String, Integer> expensiveComputer = new ImoocCache3<>(
                  new ExpensiveFunction());
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第一次的计算结果："+result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第三次的计算结果："+result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("667");
                      System.out.println("第二次的计算结果："+result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
      }
  }
  ```

## 性能待优化——引出锁性能优化经验：缩小锁的粒度

* 当然，这样加锁虽然提高了并发效率，但是并不意味着就是线程安全的，还需要考虑到同时读写等情况

* 但是，其实没必要自己实现线程安全的HashMap，也不应该加synchronized，因为我们自己实现的性能远不如现有的并发集合

* 我们来使用ConcurrentHashMap优化我们的缓存

  ```java
  /**
   * 描述：     缩小了synchronized的粒度，提高性能，但是依然并发不安全
   */
  public class ImoocCache4<A, V> implements Computable<A, V> {
  
      private final Map<A, V> cache = new HashMap();
  
      private final Computable<A, V> c;
  
      public ImoocCache4(Computable<A, V> c) {
          this.c = c;
      }
  
      @Override
      public V compute(A arg) throws Exception {
          System.out.println("进入缓存机制");
          V result = cache.get(arg);
          if (result == null) {
              result = c.compute(arg);
              synchronized (this) {
                  cache.put(arg, result);
              }
          }
          return result;
      }
  
      public static void main(String[] args) throws Exception {
          ImoocCache4<String, Integer> expensiveComputer = new ImoocCache4<>(
                  new ExpensiveFunction());
          Integer result = expensiveComputer.compute("666");
          System.out.println("第一次计算结果：" + result);
          result = expensiveComputer.compute("666");
          System.out.println("第二次计算结果：" + result);
      }
  }
  ```

## 用并发集合——引出ConcurrentHashMap

* 代码和修改过程
  
  * 用ConcurrentHashMap来代替HashMap，由于ConcurrentHashMap自身是线程安全的，所以我们无需自己处理HashMap的线程安全问题，也无须手动synchronized，同时也提高了并发时的效率，因为ConcurrentHashMap的并发度远远高于synchronized修饰的HashMap
  
    ```java
    /**
     * 描述：     缩小了synchronized的粒度，提高性能，但是依然并发不安全
     */
    public class ImoocCache5<A, V> implements Computable<A, V> {
    
        private final Map<A, V> cache = new ConcurrentHashMap<>();
    
        private final Computable<A, V> c;
    
        public ImoocCache5(Computable<A, V> c) {
            this.c = c;
        }
    
        @Override
        public V compute(A arg) throws Exception {
            System.out.println("进入缓存机制");
            V result = cache.get(arg);
            if (result == null) {
                result = c.compute(arg);
                cache.put(arg, result);
            }
            return result;
        }
    
        public static void main(String[] args) throws Exception {
            ImoocCache5<String, Integer> expensiveComputer = new ImoocCache5<>(
                    new ExpensiveFunction());
            Integer result = expensiveComputer.compute("666");
            System.out.println("第一次计算结果：" + result);
            result = expensiveComputer.compute("666");
            System.out.println("第二次计算结果：" + result);
        }
    }
    ```
* 缺点
  
  * 在计算完成前，另一个要求计算相同值的请求到来，会导致计算两遍，这和缓存想避免多次计算的初衷恰恰相反，是不可接受的
  
    ![image-20201013081623919](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201013081623919.png)

## 避免重复计算——引出Future和Callable的妙用

* 动机
  * 现在不同的线程进来以后，确实可以同时计算，但是如果两个线程脚前脚后，也就是相差无几的进来请求同一个数据，那么我们来看看会出现什么问题：重复计算
  
  * 这个例子只有2个线程，并不可怕，但是如果是100个线程都请求同样的内容，却都需要重新计算，那么会造成巨大的浪费
  
    ```java
    /**
     * 描述：     缩小了synchronized的粒度，提高性能，但是依然并发不安全
     */
    public class ImoocCache6<A, V> implements Computable<A, V> {
        private final Map<A, V> cache = new ConcurrentHashMap<>();
        private final Computable<A, V> c;
        public ImoocCache6(Computable<A, V> c) {
            this.c = c;
        }
    
        @Override
        public V compute(A arg) throws Exception {
            System.out.println("进入缓存机制");
            V result = cache.get(arg);
            if (result == null) {
                result = c.compute(arg);
                cache.put(arg, result);
            }
            return result;
        }
    
        public static void main(String[] args) throws Exception {
            ImoocCache6<String, Integer> expensiveComputer = new ImoocCache6<>(
                    new ExpensiveFunction());
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Integer result = expensiveComputer.compute("666");
                        System.out.println("第一次的计算结果：" + result);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Integer result = expensiveComputer.compute("666");
                        System.out.println("第三次的计算结果：" + result);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Integer result = expensiveComputer.compute("667");
                        System.out.println("第二次的计算结果：" + result);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
    ```
  
* 改进方向
  
  * 前人种树，后人乘凉
  
  ```java
  /**
   * 描述：     利用Future，避免重复计算
   */
  public class ImoocCache7<A, V> implements Computable<A, V> {
      private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();
      private final Computable<A, V> c;
      public ImoocCache7(Computable<A, V> c) {
          this.c = c;
      }
  
      @Override
      public V compute(A arg) throws Exception {
          Future<V> f = cache.get(arg);
          if (f == null) {
              Callable<V> callable = new Callable<V>() {
                  @Override
                  public V call() throws Exception {
                      return c.compute(arg);
                  }
              };
              FutureTask<V> ft = new FutureTask<>(callable);
              f = ft;
              cache.put(arg, ft);
              System.out.println("从FutureTask调用了计算函数");
              ft.run();
          }
          return f.get();
      }
  
      public static void main(String[] args) throws Exception {
          ImoocCache7<String, Integer> expensiveComputer = new ImoocCache7<>(
                  new ExpensiveFunction());
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第一次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第三次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("667");
                      System.out.println("第二次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
      }
  }
  ```

## 依然存在重复的可能——更好的办法是用原子操作putIfAbsent

* 如果有两个同时计算666的线程，同时调用cache.get方法，那么返回的结果都为null，后面还是会创建两个任务去计算相同的值

  ![image-20201013082415365](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201013082415365.png)

  ```java
  /**
   * 描述：     利用Future，避免重复计算
   */
  public class ImoocCache8<A, V> implements Computable<A, V> {
      private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();
      private final Computable<A, V> c;
      public ImoocCache8(Computable<A, V> c) {
          this.c = c;
      }
      @Override
      public V compute(A arg) throws Exception {
          Future<V> f = cache.get(arg);
          if (f == null) {
              Callable<V> callable = new Callable<V>() {
                  @Override
                  public V call() throws Exception {
                      return c.compute(arg);
                  }
              };
              FutureTask<V> ft = new FutureTask<>(callable);
              f = cache.putIfAbsent(arg, ft);
              if (f == null) {
                  f = ft;
                  System.out.println("从FutureTask调用了计算函数");
                  ft.run();
              }
          }
          return f.get();
      }
  
      public static void main(String[] args) throws Exception {
          ImoocCache8<String, Integer> expensiveComputer = new ImoocCache8<>(
                  new ExpensiveFunction());
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第一次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第三次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("667");
                      System.out.println("第二次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
      }
  }
  ```

## 计算中抛出异常——引出ExcecutionException

* 对异常的处理

  ```java
  /**
   * 描述：     耗时计算的实现类，有概率计算失败
   */
  public class MayFail implements Computable<String, Integer>{
      @Override
      public Integer compute(String arg) throws Exception {
          double random = Math.random();
          if (random > 0.5) {
              throw new IOException("读取文件出错");
          }
          Thread.sleep(3000);
          return Integer.valueOf(arg);
      }
  }
  ```

  ```java
  /**
   * 描述：     利用Future，避免重复计算
   */
  public class ImoocCache9<A, V> implements Computable<A, V> {
      private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();
      private final Computable<A, V> c;
      public ImoocCache9(Computable<A, V> c) {
          this.c = c;
      }
  
      @Override
      public V compute(A arg) throws InterruptedException, ExecutionException {
          while (true) {
              Future<V> f = cache.get(arg);
              if (f == null) {
                  Callable<V> callable = new Callable<V>() {
                      @Override
                      public V call() throws Exception {
                          return c.compute(arg);
                      }
                  };
                  FutureTask<V> ft = new FutureTask<>(callable);
                  f = cache.putIfAbsent(arg, ft);
                  if (f == null) {
                      f = ft;
                      System.out.println("从FutureTask调用了计算函数");
                      ft.run();
                  }
              }
              try {
                  return f.get();
              } catch (CancellationException e) {
                  System.out.println("被取消了");
                  cache.remove(arg);
                  throw e;
              } catch (InterruptedException e) {
                  cache.remove(arg);
                  throw e;
              } catch (ExecutionException e) {
                  System.out.println("计算错误，需要重试");
                  cache.remove(arg);
              }
          }
      }
  
      public static void main(String[] args) throws Exception {
          ImoocCache9<String, Integer> expensiveComputer = new ImoocCache9<>(
                  new MayFail());
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第一次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("666");
                      System.out.println("第三次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Integer result = expensiveComputer.compute("667");
                      System.out.println("第二次的计算结果：" + result);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
      }
  }
  ```

## 计算期间任务被取消——处理CancellationException

## 正确的异常处理逻辑——各司其职

* 停止线程的正确方法
* 这3种异常之所以用不同的catch块捕获，是因为它们的处理逻辑是不同的
  * CancellationException和InterruptedException是人为取消的，那么我们应该立即终止任务
  * 但是如果是计算错误，且我们明确知道多试几次就可以得到答案，那么我们的逻辑应该是重试，尝试多次直到正确的结果出现
  * 在这里，我们加上while(true)来保证计算出错不会影响我们的逻辑，然后如果是计算错误，就进入下一个循环，重新计算，直到计算成功；如果是人为取消，那么就抛出异常然后结束运行

## 处理异常的正确逻辑

* 抛出或真正处理或复现该异常，不能自己吞掉

## 考虑“缓存污染”问题

* 计算失败则移除Future，增加健壮性
* 演示缓存污染带来的问题，无论是计算错误还是被取消，都应该用cache.remove把缓存清理掉，这样后续的计算才可能成功

## 缓存过期功能

* 用FutureTask的子类，为每个结果指定过期时间，并定期扫描过期的元素
* 第二次计算设置了缓存时间，导致第三次计算需要重新计算。
* 出于安全性考虑，缓存需要能够设置有效期，到期自动失效，否则缓存一直不失效的话，会带来很多缓存过期、不一致的问题

```java
/**
 * 描述：     出于安全性考虑，缓存需要设置有效期，到期自动失效，否则如果缓存一直不失效，那么带来缓存不一致等问题
 */
public class ImoocCache10<A, V> implements Computable<A, V> {
    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();
    private final Computable<A, V> c;
    public ImoocCache10(Computable<A, V> c) {
        this.c = c;
    }

    @Override
    public V compute(A arg) throws InterruptedException, ExecutionException {
        while (true) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> callable = new Callable<V>() {
                    @Override
                    public V call() throws Exception {
                        return c.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<>(callable);
                f = cache.putIfAbsent(arg, ft);
                if (f == null) {
                    f = ft;
                    System.out.println("从FutureTask调用了计算函数");
                    ft.run();
                }
            }
            try {
                return f.get();
            } catch (CancellationException e) {
                System.out.println("被取消了");
                cache.remove(arg);
                throw e;
            } catch (InterruptedException e) {
                cache.remove(arg);
                throw e;
            } catch (ExecutionException e) {
                System.out.println("计算错误，需要重试");
                cache.remove(arg);
            }
        }
    }

    public V computeRandomExpire(A arg) throws ExecutionException, InterruptedException {
        long randomExpire = (long) (Math.random() * 10000);
        return compute(arg, randomExpire);
    }

    public final static ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);

    public V compute(A arg, long expire) throws ExecutionException, InterruptedException {
        if (expire>0) {
            executor.schedule(new Runnable() {
                @Override
                public void run() {
                    expire(arg);
                }
            }, expire, TimeUnit.MILLISECONDS);
        }
        return compute(arg);
    }

    public synchronized void expire(A key) {
        Future<V> future = cache.get(key);
        if (future != null) {
            if (!future.isDone()) {
                System.out.println("Future任务被取消");
                future.cancel(true);
            }
            System.out.println("过期时间到，缓存被清除");
            cache.remove(key);
        }
    }
    public static void main(String[] args) throws Exception {
        ImoocCache10<String, Integer> expensiveComputer = new ImoocCache10<>(
                new MayFail());
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("666",5000L);
                    System.out.println("第一次的计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("666");
                    System.out.println("第三次的计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("667");
                    System.out.println("第二次的计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();


        Thread.sleep(6000L);
        Integer result = expensiveComputer.compute("666");
        System.out.println("主线程的计算结果：" + result);
    }
}
```

## 过量的缓存需要被清理

* LRU和FRU算法

## 测试缓存效果

* 用多种不同的线程的创建方式来创建线程
* 用实现Runnable接口的方式和继承Thread类的方法实现线程，效果一样

## 模拟大量请求，观测缓存效果

* 引出线程池相关知识、选择合适的线程数
* 用线程池创建大量线程get，用了缓存后，总体耗时大大减少，体现了缓存的作用

```java
public class ImoocCache12 {

    static ImoocCache10<String, Integer> expensiveComputer = new ImoocCache10<>(
            new ExpensiveFunction());
    public static CountDownLatch countDownLatch = new CountDownLatch(1);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(100);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 100; i++) {
            service.submit(() -> {
                Integer result = null;
                try {
                    System.out.println(Thread.currentThread().getName()+"开始等待");
                    countDownLatch.await();
                    SimpleDateFormat dateFormat = ThreadSafeFormatter.dateFormatter.get();
                    String time = dateFormat.format(new Date());
                    System.out.println(Thread.currentThread().getName()+"   "+time+"被放行");
                    result = expensiveComputer.compute("666");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
                System.out.println(result);
            });
        }

        Thread.sleep(5000);
        countDownLatch.countDown();
        service.shutdown();
    }
}
class ThreadSafeFormatter {

    public static ThreadLocal<SimpleDateFormat> dateFormatter = new ThreadLocal<SimpleDateFormat>() {

        //每个线程会调用本方法一次，用于初始化
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("mm:ss");
        }

        //首次调用本方法时，会调用initialValue()；后面的调用会返回第一次创建的值
        @Override
        public SimpleDateFormat get() {
            return super.get();
        }
    };
}
```

## 想测并发性能，所有线程一同访问缓存——CountDownLatch

* 前一个类存在一个问题，就是大量的请求实际上不是同时到达的，而是分先后，但是这样就没办法给缓存造成压力，我们需要真正的同一时刻大量请求到达，此时可以用CountDownLatch来实现

## 每个线程都有存储独立信息的需求——引出用ThreadLocal、线程的各属性

* 每个线程都想要打印当前时间

## 高并发访问时，第一次都拿不到缓存，导致打爆cpu和MySQL，造成缓存雪崩、缓存击穿等高并发下的缓存问题

* 如果同时过期，那么同时都拿不到缓存，导致打爆cpu和MySQL，造成缓存雪崩、缓存击穿等高并发下的缓存问题

* 把缓存的过期时间设置为随机，就避免了缓存雪崩

## 总结ImoocCache的亮点和涉及的知识列表

* 写一个缓存，用到了这么多的并发知识，以后有了这些知识作为储备，学习其他的优秀源码，比如Guava Cache，就容易多了

# 总结

