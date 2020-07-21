# ==Video ：14-23 15-1 15-2 15-3==

# 死锁

## 死锁是什么？有什么危害？

* 什么是死锁？
  * 发生在并发中
  * 互不相让：当两个（或更多）线程（或进程）相互持有对方所需要的资源，又不主动释放，导致所有人都无法继续前进，导致程序陷入无尽的阻塞，这就是死锁
  * 多个线程造成死锁的情况 

* 死锁的影响
  * 死锁的影响在不同系统中是不一样的，这取决于系统对死锁的处理能力
  * 数据库中：检测并放弃事务
  * JVM中：无法自动处理
* 几率不高但危害大
  * 不一定发生，但是遵守“墨菲定律”
  * 一旦发生，多是高并发场景，影响用户多
  * 整个系统崩溃、子系统崩溃、性能降低
  * 压力测试无法找出所有潜在的死锁

## 发生死锁的例子

* 最简单的情况
  * 分析
    * 当类的对象flag=1时（T1），先锁定O1,睡眠500毫秒，然后锁定O2；
    * 而T1在睡眠的时候另一个flag=0的对象（T2）线程启动，先锁定O2,睡眠500毫秒，等待T1释放O1；
    * T1睡眠结束后需要锁定O2才能继续执行，而此时O2已被T2锁定；
    * T2睡眠结束后需要锁定O1才能继续执行，而此时O1已被T1锁定；
    * T1、T2相互等待，都需要对方锁定的资源才能继续执行，从而死锁。
  * 注意看退出信号
    * Process finished with exit code 130 (interrupted by signal 2: SIGINT)，是不正常退出的信号，对比正常结束的程序的结束信号是0
* 实际生产中的例子：转账
  * 需要两把锁
  * 获取两把锁成功，且余额大于0，则扣除转出人，增加收款人的余额，是原子操作
  * 顺序相反导致死锁
* 模拟多人随机转账
  * 5万人很多，但是依然会发生死锁，墨菲定律
  * 复习：发生死锁几率不高但危害大

```java
/**
 * 描述：     必定发生死锁的情况
 */
public class MustDeadLock implements Runnable {
    int flag = 1;
    static Object o1 = new Object();
    static Object o2 = new Object();

    public static void main(String[] args) {
        MustDeadLock r1 = new MustDeadLock();
        MustDeadLock r2 = new MustDeadLock();
        r1.flag = 1;
        r2.flag = 0;
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
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
                    System.out.println("线程1成功拿到两把锁");
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
                    System.out.println("线程2成功拿到两把锁");
                }
            }
        }
    }
}
```

```java
/**
 * 描述：     转账时候遇到死锁，一旦打开注释，便会发生死锁
 */
public class TransferMoney implements Runnable {

    int flag = 1;
    static Account a = new Account(500);
    static Account b = new Account(500);
    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        TransferMoney r1 = new TransferMoney();
        TransferMoney r2 = new TransferMoney();
        r1.flag = 1;
        r2.flag = 0;
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("a的余额" + a.balance);
        System.out.println("b的余额" + b.balance);
    }

    @Override
    public void run() {
        if (flag == 1) {
            transferMoney(a, b, 200);
        }
        if (flag == 0) {
            transferMoney(b, a, 200);
        }
    }

    public static void transferMoney(Account from, Account to, int amount) {
        class Helper {
            public void transfer() {
                if (from.balance - amount < 0) {
                    System.out.println("余额不足，转账失败。");
                    return;
                }
                from.balance -= amount;
                to.balance = to.balance + amount;
                System.out.println("成功转账" + amount + "元");
            }
        }
        int fromHash = System.identityHashCode(from);
        int toHash = System.identityHashCode(to);
        if (fromHash < toHash) {
            synchronized (from) {
                synchronized (to) {
                    new Helper().transfer();
                }
            }
        }
        else if (fromHash > toHash) {
            synchronized (to) {
                synchronized (from) {
                    new Helper().transfer();
                }
            }
        }else  {
            synchronized (lock) {
                synchronized (to) {
                    synchronized (from) {
                        new Helper().transfer();
                    }
                }
            }
        }

    }

    static class Account {
        public Account(int balance) {
            this.balance = balance;
        }
        int balance;
    }
}
```

```java
import deadlock.TransferMoney.Account;
import java.util.Random;

/**
 * 描述：     多人同时转账，依然很危险
 */
public class MultiTransferMoney {

    private static final int NUM_ACCOUNTS = 500;
    private static final int NUM_MONEY = 1000;
    private static final int NUM_ITERATIONS = 1000000;
    private static final int NUM_THREADS = 20;

    public static void main(String[] args) {

        Random rnd = new Random();
        Account[] accounts = new Account[NUM_ACCOUNTS];
        for (int i = 0; i < accounts.length; i++) {
            accounts[i] = new Account(NUM_MONEY);
        }
        class TransferThread extends Thread {

            @Override
            public void run() {
                for (int i = 0; i < NUM_ITERATIONS; i++) {
                    int fromAcct = rnd.nextInt(NUM_ACCOUNTS);
                    int toAcct = rnd.nextInt(NUM_ACCOUNTS);
                    int amount = rnd.nextInt(NUM_MONEY);
                    TransferMoney.transferMoney(accounts[fromAcct], accounts[toAcct], amount);
                }
                System.out.println("运行结束");
            }
        }
        for (int i = 0; i < NUM_THREADS; i++) {
            new TransferThread().start();
        }
    }
}
```

## 死锁的必要条件

* 缺一不可
  * 互斥条件
  * 请求与保持条件
  * 不剥夺条件
  * 循环等待条件

## 如何定位死锁？

* jstack

  * 用Sloth或者命令行 JPS查看到java的pid

  * 执行${JAVA_HOME}/bin/jstack pid

    ![image-20200719204340843](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719204340843.png)

* ThreadMXBean

  ```java
  import java.lang.management.ManagementFactory;
  import java.lang.management.ThreadInfo;
  import java.lang.management.ThreadMXBean;
  
  /**
   * 描述：     用ThreadMXBean检测死锁
   */
  public class ThreadMXBeanDetection implements Runnable {
  
      int flag = 1;
  
      static Object o1 = new Object();
      static Object o2 = new Object();
  
      public static void main(String[] args) throws InterruptedException {
          ThreadMXBeanDetection r1 = new ThreadMXBeanDetection();
          ThreadMXBeanDetection r2 = new ThreadMXBeanDetection();
          r1.flag = 1;
          r2.flag = 0;
          Thread t1 = new Thread(r1);
          Thread t2 = new Thread(r2);
          t1.start();
          t2.start();
          Thread.sleep(1000);
          ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
          long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();
          if (deadlockedThreads != null && deadlockedThreads.length > 0) {
              for (int i = 0; i < deadlockedThreads.length; i++) {
                  ThreadInfo threadInfo = threadMXBean.getThreadInfo(deadlockedThreads[i]);
                  System.out.println("发现死锁" + threadInfo.getThreadName());
              }
          }
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
                      System.out.println("线程1成功拿到两把锁");
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
                      System.out.println("线程2成功拿到两把锁");
                  }
              }
          }
      }
  }
  ```

## 修复死锁的策略

* 线上发生死锁应该怎么办？

  * 线上问题都需要防患于未然，不造成损失地扑灭几乎已经是不可能
  * 保存案发现场然后立刻重启服务器				
  * 暂时保证线上服务的安全，然后在利用刚才保存的信息，排查死锁，修改代码，重新发版

* 常见修复策略

  * 避免策略：哲学家就餐的换手方案、转账换序方案
  * 检测与恢复策略：一段时间检测是否有死锁，如果有就剥夺某一个资源，来打开死锁
  * 鸵鸟策略：鸵鸟这种动物在遇到危险的时候，通常就会把头埋在地上，这样一来它就看不到危险了。而鸵鸟策略的意思就是说，如果我们发生死锁的概率极其低，那么我们就直接忽略它，直到死锁发生的时候，再人工修复

* 避免策略

  * 思路：避免相反的获取锁的顺序

  * 转账时避免死锁

    * 实际上不在乎获取锁的顺序

    * 代码演示

      ```java
      /**
       * 描述：     转账时候遇到死锁，一旦打开注释，便会发生死锁
       */
      public class TransferMoney implements Runnable {
      
          int flag = 1;
          static Account a = new Account(500);
          static Account b = new Account(500);
          static Object lock = new Object();
      
          public static void main(String[] args) throws InterruptedException {
              TransferMoney r1 = new TransferMoney();
              TransferMoney r2 = new TransferMoney();
              r1.flag = 1;
              r2.flag = 0;
              Thread t1 = new Thread(r1);
              Thread t2 = new Thread(r2);
              t1.start();
              t2.start();
              t1.join();
              t2.join();
              System.out.println("a的余额" + a.balance);
              System.out.println("b的余额" + b.balance);
          }
      
          @Override
          public void run() {
              if (flag == 1) {
                  transferMoney(a, b, 200);
              }
              if (flag == 0) {
                  transferMoney(b, a, 200);
              }
          }
      
          public static void transferMoney(Account from, Account to, int amount) {
              class Helper {
      
                  public void transfer() {
                      if (from.balance - amount < 0) {
                          System.out.println("余额不足，转账失败。");
                          return;
                      }
                      from.balance -= amount;
                      to.balance = to.balance + amount;
                      System.out.println("成功转账" + amount + "元");
                  }
              }
              int fromHash = System.identityHashCode(from);
              int toHash = System.identityHashCode(to);
              if (fromHash < toHash) {
                  synchronized (from) {
                      synchronized (to) {
                          new Helper().transfer();
                      }
                  }
              }
              else if (fromHash > toHash) {
                  synchronized (to) {
                      synchronized (from) {
                          new Helper().transfer();
                      }
                  }
              }else  {
                 // 加时赛
                 // 有主键（唯一），就不需要加时赛了
                  synchronized (lock) {
                      synchronized (to) {
                          synchronized (from) {
                              new Helper().transfer();
                          }
                      }
                  }
              }
          }
      
          static class Account {
              public Account(int balance) {
                  this.balance = balance;
              }
              int balance;
          }
      }
      ```

    * 通过hashcode来决定获取锁的顺序、冲突时需要“加时赛”

    * 有主键就更方便

  * 哲学家就餐

    * 流程

      * 先拿起左手的筷子
      * 然后拿起右手的筷子
      * 如果筷子被人使用了 ，那就等别人用完
      * 吃完后，把筷子放回原位

    * 有死锁和资源耗尽的风险

      * 死锁：每个哲学家都拿着左手的餐叉，永远都在等右边的餐叉（或者相反）

      * 活锁：在完全相同的时刻进入餐厅，并同时拿起左边的餐叉，那么这些哲学家就会等待五分钟，同时放下手中的餐叉，再等五分钟，又同时拿起这些餐叉

      * 在实际的计算机问题中，缺乏餐叉可以类比为缺乏共享资源

      * 代码演示：哲学家进入死锁

        ```java
        /**
         * 描述：     演示哲学家就餐问题导致的死锁
         */
        public class DiningPhilosophers {
        
            public static class Philosopher implements Runnable {
        
                private Object leftChopstick;
        
                public Philosopher(Object leftChopstick, Object rightChopstick) {
                    this.leftChopstick = leftChopstick;
                    this.rightChopstick = rightChopstick;
                }
        
                private Object rightChopstick;
        
                @Override
                public void run() {
                    try {
                        while (true) {
                            doAction("Thinking");
                            synchronized (leftChopstick) {
                                doAction("Picked up left chopstick");
                                synchronized (rightChopstick) {
                                    doAction("Picked up right chopstick - eating");
                                    doAction("Put down right chopstick");
                                }
                                doAction("Put down left chopstick");
                            }
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
        
                private void doAction(String action) throws InterruptedException {
                    System.out.println(Thread.currentThread().getName() + " " + action);
                    Thread.sleep((long) (Math.random() * 10));
                }
            }
        
            public static void main(String[] args) {
                Philosopher[] philosophers = new Philosopher[5];
                Object[] chopsticks = new Object[philosophers.length];
                for (int i = 0; i < chopsticks.length; i++) {
                    chopsticks[i] = new Object();
                }
                for (int i = 0; i < philosophers.length; i++) {
                    Object leftChopstick = chopsticks[i];
                    Object rightChopstick = chopsticks[(i + 1) % chopsticks.length];
                    if (i == philosophers.length - 1) {
                        philosophers[i] = new Philosopher(rightChopstick, leftChopstick);
                    } else {
                        philosophers[i] = new Philosopher(leftChopstick, rightChopstick);
                    }
                    new Thread(philosophers[i], "哲学家" + (i + 1) + "号").start();
                }
            }
        }
        ```

    * 多种解决方案

      * 服务员检查（避免策略）
      * **改变一个哲学家拿叉子的顺序（避免策略）**
      * 餐票（避免策略）
      * 领导调节（检测与恢复策略）
      * 代码演示：死锁解决					

* 多种解决方案

  * 避免策略
  * 检测与恢复策略

* 检测与恢复策略

  * 检测算法：锁的调用链路图
    * 允许发生死锁
    * 每次调用锁都记录
    * 定期检查“锁的调用链路图”中是否存在环路
    * 一旦发生死锁，就用死锁恢复机制进行恢复					
  * 恢复方法1：进程终止
    * 逐个终止线程，直到死锁消除。
    * 终止顺序：
      1. 优先级（是前台交互还是后台处理）
      2. 已占用资源、还需要的资源
      3. 已经运行时间
  * 恢复方法2：资源抢占
    * 把已经分发出去的锁给收回来
    * 让线程回退几步，这样就不用结束整个线程，成本比较低
    * 缺点：可能同一个线程一直被抢占，那就造成饥饿

## 实际工程中如何避免死锁？

* 设置超时时间

  * Lock的tryLock(long timeout, TimeUnit unit)

  * synchronized不具备尝试锁的能力

  * 造成超时的可能性多：发生了死锁、线程陷入死循环、线程执行很慢

  * 获取锁失败：打日志、发报警邮件、重启等

    ```java
    import java.util.Random;
    import java.util.concurrent.TimeUnit;
    import java.util.concurrent.locks.Lock;
    import java.util.concurrent.locks.ReentrantLock;
    
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
            r2.flag = 0;
            new Thread(r1).start();
            new Thread(r2).start();
        }
    
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                if (flag == 1) {
                    try {
                        if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                            System.out.println("线程1获取到了锁1");
                            Thread.sleep(new Random().nextInt(1000));
                            if (lock2.tryLock(800, TimeUnit.MILLISECONDS)) {
                                System.out.println("线程1获取到了锁2");
                                System.out.println("线程1成功获取到了两把锁");
                                lock2.unlock();
                                lock1.unlock();
                                break;
                            } else {
                                System.out.println("线程1尝试获取锁2失败，已重试");
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
                            System.out.println("线程2获取到了锁2");
    
                            Thread.sleep(new Random().nextInt(1000));
                            if (lock1.tryLock(3000, TimeUnit.MILLISECONDS)) {
                                System.out.println("线程2获取到了锁1");
                                System.out.println("线程2成功获取到了两把锁");
                                lock1.unlock();
                                lock2.unlock();
                                break;
                            } else {
                                System.out.println("线程2尝试获取锁1失败，已重试");
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

* 多使用并发类而不是自己设计锁

  * ConcurrentHashMap、ConcurrentLinkedQueue、AtomicBoolean等
  * 实际应用中java.util.concurrent.atomic十分有用，简单方便且效率比使用Lock更高
  * 多用并发集合少用同步集合，并发集合比同步集合的可扩展性更好
  * 并发场景需要用到map，首先想到用ConcurrentHashMap

* 尽量降低锁的使用粒度：用不同的锁而不是一个锁

* 如果能使用同步代码块，就不使用同步方法：自己指定锁对象

* 给你的线程起个有意义的名字：debug和排查时事半功倍，框架和JDK都遵守这个最佳实践

* 避免锁的嵌套：MustDeadLock类

* 分配资源前先看能不能收回来：银行家算法

* 尽量不要几个功能用同一把锁：专锁专用

## 其他活性故障（又叫活跃性问题）

* 锁是最常见的活跃性问题，不过除了刚才的死锁之外，还有一些类似的问题，会导致程序无法顺利执行，统称为活跃性问题

* 活锁（LiveLock）

  * 什么是活锁
    * 虽然线程并没有阻塞，也始终在运行（所以叫做“活”锁，线程是“活”的），但是程序却得不到进展，因为线程始终重复做同样的事
    * 如果这里死锁，那么就是这里两个人都始终一动不动，直到对方先抬头，他们之间不再说话了，只是等待
    * 如果发生活锁，那么这里的情况就是，双方都不停地对对方说“你先起来吧，你先起来吧”，双方都一直在说话，在运行
    * 死锁和活锁的结果是一样的，就是谁都不能先抬头
  * 工程中的活锁实例：消息队列
    * 策略：消息如果处理失败，就放在队列开头重试
    * 由于依赖服务出了问题，处理该消息一直失败
    * 没阻塞，但程序无法继续
    * 解决：放到队列尾部；重试限制
  * 如何解决活锁问题
    * 原因：重试机制不变，消息队列始终重试，吃饭始终谦让
    * 以太网的指数退避算法
    * 加入随机因素

  ![image-20200719220126139](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719220126139.png)

  ```java
  import java.util.Random;
  import jdk.management.resource.internal.inst.RandomAccessFileRMHooks;
  
  /**
   * 描述：     演示活锁问题
   */
  public class LiveLock {
  
      static class Spoon {
  
          private Diner owner;
  
          public Spoon(Diner owner) {
              this.owner = owner;
          }
  
          public Diner getOwner() {
              return owner;
          }
  
          public void setOwner(Diner owner) {
              this.owner = owner;
          }
  
          public synchronized void use() {
              System.out.printf("%s吃完了!", owner.name);
  
  
          }
      }
  
      static class Diner {
  
          private String name;
          private boolean isHungry;
  
          public Diner(String name) {
              this.name = name;
              isHungry = true;
          }
  
          public void eatWith(Spoon spoon, Diner spouse) {
              while (isHungry) {
                  if (spoon.owner != this) {
                      try {
                          Thread.sleep(1);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      continue;
                  }
                  Random random = new Random();
                  if (spouse.isHungry && random.nextInt(10) < 9) {
                      System.out.println(name + ": 亲爱的" + spouse.name + "你先吃吧");
                      spoon.setOwner(spouse);
                      continue;
                  }
  
                  spoon.use();
                  isHungry = false;
                  System.out.println(name + ": 我吃完了");
                  spoon.setOwner(spouse);
  
              }
          }
      }
  
  
      public static void main(String[] args) {
          Diner husband = new Diner("牛郎");
          Diner wife = new Diner("织女");
  
          Spoon spoon = new Spoon(husband);
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  husband.eatWith(spoon, wife);
              }
          }).start();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  wife.eatWith(spoon, husband);
              }
          }).start();
      }
  }
  ```

* 饥饿

  * 当线程需要某些资源（例如CPU），但是却始终得不到
  * 线程的优先级设置得过于低，或者有某线程持有锁同时又无限循环从而不释放锁，或者某程序始终占用某文件的写锁
  * 饥饿可能会导致响应性差：比如，我们的浏览器有一个线程负责处理前台响应（打开收藏夹等动作），另外的后台线程负责下载图片和文件、计算渲染等。在这种情况下，如果后台线程把CPU资源都占用了，那么前台线程将无法得到很好地执行，这会导致用户的体验很差
  * 线程优先级
    * 10个级别，默认5
    * 程序设计不应依赖于优先级
    * 不同操作系统不一样
    * 优先级会被操作系统改变

## 常见面试问题

* 写一个必然死锁的例子，生产中什么场景下会发生死锁
* 发生死锁必须满足哪些条件
* 如何定位死锁
* 有哪些解决死锁问题的策略
* 讲一讲经典的哲学家就餐问题
* 实际工程中如何避免死锁
* 什么是活跃性问题？活锁、饥饿和死锁有什么区别

![image-20200719223704402](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223704402.png)

![image-20200719223733205](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223733205.png)

![image-20200719223800859](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223800859.png)

![image-20200719223820390](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223820390.png)

![image-20200719223832414](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223832414.png)

![image-20200719223846868](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223846868.png)



![image-20200719223908900](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223908900.png)

![image-20200719223921948](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223921948.png)

![image-20200719223935711](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223935711.png)

![image-20200719223953747](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719223953747.png)

![image-20200719224021732](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719224021732.png)

![image-20200719224056254](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719224056254.png)

![image-20200719224122159](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719224122159.png)

![image-20200719224136271](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719224136271.png)

![image-20200719224152788](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719224152788.png)

![image-20200719224205894](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200719224205894.png)

