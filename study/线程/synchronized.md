# Synchronized

* 一句话介绍Synchronized
  * JVM会自动通过使用monitor来加锁和解锁，保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质

## 简介

* 作用

  * 能够保证在同一时刻最多只有一个线程执行该段代码，以达到保证并发安全的效果

* 地位

  * 是Java的关键字，被Java语言原生支持
  * 是最基本的互斥同步手段
  * 是并发编程中的元老级角色，是并发编程的必学内容

* 不控制并发的后果

  * 两个线程同时a++，最后结果会比预计的少
  * 原因：count++，它看上去只是一个操作，实际上包含了三个动作：
    1. 读取count
    2. 将count加一
    3. 将count的值写入到内存中

  ```java
  // 消失的请求数
  public class DisappearRequest1 implements Runnable{
      
      static DisappearRequest1 instance = new DisappearRequest1();
      
      static int i = 0;
      
      public static void main(String[] args) throws InterruptedException {
          Thread t1 = new Thread(instance);
          Thread t2 = new Thread(instance);
          t1.start();
          t2.start();
          t1.join();
          t2.join();
          System.out.println(i);
      }
  
      @Override
      public void run() {
          for (int j = 0; j < 100000; j++) {
              i ++;
          }
      }
  }
  ```
  
* 解决方案
  
  ```java
    // 方案一
    @Override
    public synchronized void run() {
      for (int j = 0; j < 100000; j++) {
        i ++;
      }
    }
    // 方案二
    @Override
    public void run() {
      synchronized(this) {
        for (int j = 0; j < 100000; j++) {
        	i ++;
      	}
      }
    }
    // 方案三
    @Override
    public void run() {
      synchronized(DisappearRequest1.class) {
        for (int j = 0; j < 100000; j++) {
        	i ++;
      	}
      }
    }
    // 方案四
    synchronized修饰方法
    ```

## 用法

### 对象锁

* 包括方法锁（默认锁对象为this当前实例对象）和同步代码块锁（自己指定锁对象）

  * 代码块形式：手动指定锁对象

  * 方法锁形式：synchronized修饰普通方法，锁对象默认为this

  ```java
  public class SynchronizedObjectCodeBlock2 implements Runnable {
  
      static SynchronizedObjectCodeBlock2 instance = new SynchronizedObjectCodeBlock2();
  
      public static void main(String[] args) {
          Thread t1 = new Thread(instance);
          Thread t2 = new Thread(instance);
          t1.start();
          t2.start();
          while (t1.isAlive() || t2.isAlive()) {
  
          }
          System.out.println("finished");
      }
  
      @Override
      public void run() {
          synchronized (this) {
              System.out.println("我是对象锁的代码块形式。我叫" + Thread.currentThread().getName());
              try {
                  Thread.sleep(3000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "运行结束");
          }
      }
  }
  ```

  * debug看线程生命周期

    ![image-20200720074445104](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200720074445104.png)

    ![image-20200720074601368](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200720074601368.png)

    ![image-20200720074633257](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200720074633257.png)

     ![image-20200720074709380](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200720074709380.png)

  * 修饰方法

    ```java
    public class SynchronizedObjectMethod3 implements Runnable {
    
        static SynchronizedObjectMethod3 instance = new SynchronizedObjectMethod3();
    
        public static void main(String[] args) {
            Thread t1 = new Thread(instance);
            Thread t2 = new Thread(instance);
            t1.start();
            t2.start();
            while (t1.isAlive() || t2.isAlive()) {
    
            }
            System.out.println("finished");
        }
    
        public synchronized void method() {
            System.out.println("我的对象锁的方法修饰符形式，我叫" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "运行结束");
        }
    
        @Override
        public void run() {
            method();
        }
    }
    ```

### 类锁

* 只有一个Class对象：Java类可能有很多个对象，但只有1个Class对象 

* 本质：所以所谓的类锁，不过是Class对象的锁而已

* 用法和效果：类锁只能在同一个时刻被一个对象拥有，全局锁定

* 指synchronized修饰静态的方法或指定锁为Class对象

* 形式1

  * synchronized加在static方法上

* 形式2

  * synchronized(*.class)代码块
  
  ```java
  public class SynchronizedClassStatic4 implements Runnable {
  
      static SynchronizedClassStatic4 instance1 = new SynchronizedClassStatic4();
      static SynchronizedClassStatic4 instance2 = new SynchronizedClassStatic4();
  
      public static synchronized void method() {
          System.out.println("我是类锁的第一种形式：static形式，我叫" + Thread.currentThread().getName());
          try {
              Thread.sleep(3000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          System.out.println(Thread.currentThread().getName() + "运行结束");
      }
  
      public static void main(String[] args) {
          Thread t1 = new Thread(instance1);
          Thread t2 = new Thread(instance2);
          t1.start();
          t2.start();
          while (t1.isAlive() || t2.isAlive()) {
  
          }
          System.out.println("finished");
      }
  
      @Override
      public void run() {
          method();
      }
}
  ```
  
  ```java
  public class SynchronizedClassClass5 implements Runnable {
  
      static SynchronizedClassClass5 instance1 = new SynchronizedClassClass5();
      static SynchronizedClassClass5 instance2 = new SynchronizedClassClass5();
  
      private void method() {
          synchronized (SynchronizedClassClass5.class) {
              System.out.println("我是类锁的第二种形式：synchronized(*.class)，我叫" + Thread.currentThread().getName());
              try {
                  Thread.sleep(3000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "运行结束");
          }
      }
  
      public static void main(String[] args) {
          Thread t1 = new Thread(instance1);
          Thread t2 = new Thread(instance2);
          t1.start();
          t2.start();
          while (t1.isAlive() || t2.isAlive()) {
  
          }
          System.out.println("finished");
      }
  
      @Override
      public void run() {
          method();
      }
  }
  ```

## 同步访问

* 7种情况总结：3点核心思想
  1. 一把锁只能同时被一个线程获取，没有拿到锁的线程必须等待
  2. 每个实例都对应有自己的一把锁，不同实例之间互不影响
     * 例外：锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象共用同一把锁
  3. 无论是方法正常执行完毕或者方法抛出异常，都会释放锁

* 多线程访问同步方法的7种情况：是否是static、Synchronized方法等

  1. 两个线程访问的是一个对象的同一个同步方法

     * 同一把锁，互斥

  2. 两个线程访问的是两个实例的同一个同步方法

     * 并行，互不干扰

  3. 两个线程访问的是synchronized的静态方法

     * 同一把锁，互斥

  4. 同时访问同步方法与非同步方法

     * 并行，互不干扰

     ```java
     // 同时访问同步方法和非同步方法
     public class SynchronizedYesAndNo6 implements Runnable{
     
         static SynchronizedYesAndNo6 instance = new SynchronizedYesAndNo6();
     
         public static void main(String[] args) {
             Thread t1 = new Thread(instance);
             Thread t2 = new Thread(instance);
             t1.start();
             t2.start();
             while (t1.isAlive() || t2.isAlive()) {
     
             }
             System.out.println("finished");
         }
     
         public synchronized void method1() {
             System.out.println("我是加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         public void method2() {
             System.out.println("我是没加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         @Override
         public void run() {
             if (Thread.currentThread().getName().equals("Thread-0")) {
                 method1();
             } else {
                 method2();
             }
         }
     }
     ```

  5. 访问同一个对象的不同的普通同步方法

     * 指定this，同一把锁，串行

     ```java
     public class SynchronizedDifferentMethod7 implements Runnable {
     
         static SynchronizedDifferentMethod7 instance = new SynchronizedDifferentMethod7();
     
         public static void main(String[] args) {
             Thread t1 = new Thread(instance);
             Thread t2 = new Thread(instance);
             t1.start();
             t2.start();
             while (t1.isAlive() || t2.isAlive()) {
     
             }
             System.out.println("finished");
         }
     
         public synchronized void method1() {
             System.out.println("我是加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         public synchronized void method2() {
             System.out.println("我是加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         @Override
         public void run() {
             if (Thread.currentThread().getName().equals("Thread-0")) {
                 method1();
             } else {
                 method2();
             }
         }
     }
     ```

  6. 同时访问静态synchronized和非静态synchronized方法

     * 不同锁，互不影响

     ```java
     public class SynchronizedStaticAndNormal8 implements Runnable {
     
         static SynchronizedStaticAndNormal8 instance = new SynchronizedStaticAndNormal8();
     
         public static void main(String[] args) {
             Thread t1 = new Thread(instance);
             Thread t2 = new Thread(instance);
             t1.start();
             t2.start();
             while (t1.isAlive() || t2.isAlive()) {
     
             }
             System.out.println("finished");
         }
     
         public synchronized static void method1() {
             System.out.println("我是静态加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         public synchronized void method2() {
             System.out.println("我是非静态加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         @Override
         public void run() {
             if (Thread.currentThread().getName().equals("Thread-0")) {
                 method1();
             } else {
                 method2();
             }
         }
     }
     ```

  7. 方法抛异常后，会释放锁

     ```java
     // 方法抛出异常后，会释放锁
     public class SynchronizedException9 implements Runnable {
     
         static SynchronizedException9 instance = new SynchronizedException9();
     
         public static void main(String[] args) {
             Thread t1 = new Thread(instance);
             Thread t2 = new Thread(instance);
             t1.start();
             t2.start();
             while (t1.isAlive() || t2.isAlive()) {
     
             }
             System.out.println("finished");
         }
     
         public synchronized void method1() {
             System.out.println("我是加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             throw new RuntimeException();
     //        System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         public synchronized void method2() {
             System.out.println("我是非静态加锁方法，我叫" + Thread.currentThread().getName());
             try {
                 Thread.sleep(3000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println(Thread.currentThread().getName() + "运行结束");
         }
     
         @Override
         public void run() {
             if (Thread.currentThread().getName().equals("Thread-0")) {
                 method1();
             } else {
                 method2();
             }
         }
     }
     ```

  8. 被synchronized修饰的方法访问普通方法，这时不是线程安全的

## 性质

### 可重入

* 指的是同一线程的外层函数获得锁之后，内层函数可以直接再次获取该锁
* 好处：避免死锁、提高封装性
* 粒度：“线程”而不是“调用

* 情况2：证明可重入不要求是同一个方法

  ```java
  public class SynchronizedOtherMethod11 {
  
      public static void main(String[] args) {
          SynchronizedOtherMethod11 s = new SynchronizedOtherMethod11();
          s.method1();
      }
      public synchronized void method1() {
          System.out.println("我是method1");
          method2();
      }
  
      private synchronized void method2() {
          System.out.println("我是method2");
      }
  }
  // 我是method1
  // 我是method2
  ```

* 情况3：证明可重入不要求是同一个类中的

  ```java
  // 可重入粒度测试，调用父类的方法
  public class SynchronizedSuperClass12 {
      public synchronized void doSomething() {
          System.out.println("我是父类方法");
      }
  }
  class TestClass extends SynchronizedSuperClass12 {
      @Override
      public synchronized void doSomething() {
          System.out.println("我是子类方法");
          super.doSomething();
      }
  
      public static void main(String[] args) {
          TestClass  s = new TestClass();
          s.doSomething();
      }
  }
  //我是子类方法
  //我是父类方法
  ```

* 不可中断
* 一旦这个锁已经被别人获得了，如果还想获得，只能选择等待或者阻塞，直到别的线程释放这个锁，如果别人永远不释放锁，那么只能永久地等待下去
  * 相比之下，Lock类，可以拥有中断的能力
  * 第一点，如果觉得等的时间太长了，有权中断现在已经获取到的锁的线程的执行
    * 第二点，如果觉得等待的时间太长了不想再等了，也可以退出

## 原理

### 加锁原理

* 加锁和释放锁的原理：现象、时机、深入JVM看字节码

  * 现象

  * 获取和释放锁的时机：内置锁

  * 等价代码

    ```java
    public class SynchronizedToLock13 {
    
        Lock lock = new ReentrantLock();
    
        public synchronized void method1() {
            System.out.println("我是Synchronized形式的锁");
        }
    
        public void method2() {
            lock.lock();
            try {
                System.out.println("我是lock形式的锁");
            } finally {
                lock.unlock();
            }
        }
    
        public static void main(String[] args) {
            SynchronizedToLock13 s = new SynchronizedToLock13();
            s.method1();
            s.method2();
        }
    }
    //    我是Synchronized形式的锁
    //    我是lock形式的锁
    ```

  * 深入JVM看字节码：反编译、monitor指令

    * 如何反编译
    * Monditorenter和Monditorexit指令

### 可重入原理：加锁次数计数器

* JVM负责跟踪对象被加锁的次数
  * 线程第一次给对象加锁的时候，计数变为1。每当这个相同的线程在此对象上再次获得锁时，计数会递增
  * 每当任务离开时，计数递减，当计数为0的时候，锁被完全释放

### 保证可见性的原理：内存模型

* 线程的有本地内存；线程间通过主内存通信

  * 被synchronized修饰的方法或代码块执行完后，线程都要将数据从本地内存写入到主内存，其他线程获取锁后，从主内存中加载数据

  ![image-20200720225609486](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200720225609486.png)


## Synchronized的缺陷  

* 效率低
  * 锁的释放情况少
  * 试图获得锁时不能设定超时
  * 不能中断一个正在试图获得锁的线程
* 不够灵活（读写锁更灵活）
  * 加锁和释放的时机单一
  * 每个锁仅有单一的条件（某个对象），可能是不够的
* 无法预判是否成功获取到锁
* Lock可以补充

## 思考

* 多个线程等待同一个synchronized锁的时候，JMV如何选择下一个获取锁的是哪个线程？
  * 随机，不可控
* Synchronized使得同时只有一个线程可以执行，性能较差，有什么办法可以提升性能？
  * 作用域不宜过大，使用其他类型的锁
* 想更灵活地控制锁的获取和释放（现在释放锁的时机都被规定死了），怎么办？
  * Lock
* 什么是锁的升级、降级？什么是JVM里的偏斜锁、轻量级锁、重量级锁？
  * 版本迭代

## 面试问题

* 使用注意点
  * 锁对象不能为空：锁的信息保存在对象头中
  * 作用域不宜过大：降低并发可能能性，提高效率
  * 避免死锁
* 如何选择Lock和Synchronized等
  * JUC -> Synchronized -> Lock

## 思考

* 如何提高性能
* JVM如何决定那个线程获取锁





