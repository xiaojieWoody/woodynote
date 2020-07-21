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
      * AtomicInteger：整形原子类
      * AtomicLong：长整型原子类
      * AtomicBoolean ：布尔型原子类
    * AtomicArray数组类型原子类（数组里的元素，都可以保证原子性）
      * AtomicIntegerArray：整形数组原子类
      * AtomicLongArray：长整形数组原子类
      * AtomicReferenceArray ：引用类型数组原子类
    * AtomicReference引用类型原子类
      * AtomicReference：引用类型原子类
      * AtomicStampedReference：引用类型原子类的升级，带时间戳，可以解决ABA问题
      * AtomicMarkableReference
    * AtomicFieldUpdater升级原子类
      * AtomicIntegerFieldUpdater:原子更新整形字段的更新器
      * AtomicLongFieldUpdater：原子更新长整形字段的更新器
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
      * AtomicInteger：整形原子类
        * AtomicLong：长整型原子类
        * AtomicBoolean ：布尔型原子类
      * AtomicArray数组类型原子类（数组里的元素，都可以保证原子性）
        * AtomicIntegerArray：整形数组原子类
        * AtomicLongArray：长整形数组原子类
        * AtomicReferenceArray ：引用类型数组原子类
      * AtomicReference引用类型原子类
        * AtomicReference：引用类型原子类
        * AtomicStampedReference：引用类型原子类的升级，带时间戳，可以解决ABA问题
        * AtomicMarkableReference
      * AtomicFieldUpdater升级原子类
        * AtomicIntegerFieldUpdater:原子更新整形字段的更新器
        * AtomicLongFieldUpdater：原子更新长整形字段的更新器
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

* 线程池的重要性
* 什么是“池” 
* 如果不使用线程池，每个任务都新开一个线程处理
  * 一个线程
  * for循环创建线程
  * 当任务数量上升到1000
* 为什么要使用线程池
  * 问题一：反复创建线程开销大
  * 问题二：过多的线程会占用太多内存
  * 解决以上两个问题的思路
* 线程池好处
  * 加快响应速度
  * 合理利用CPU和内存
  * 统一管理
* 线程池适合应用的场合

## 创建和停止线程池

* 线程池构造函数的参数
  * 每个参数的含义概览
  * 参数中的corePoolSize和maxPoolSize有什么不同
  * 线程增加和减少以及task进入队列排队的规则
  * ThreadFactory（用来创建线程）
  * BlockingQueue
    * 直接交接
    * 无界队列
    * 有界的队列
* 线程池应该手动创建还是自动创建（阿里巴巴规约）
  * 阿里规约
  * 反例分析
  * 正确的创建线程池的方法
* 线程池里的线程数量设定为多少比较合适？
* 停止线程池的正确方法
  * shutdown
  * isShutdown
  * isTerminated
  * awaitTermination
  * shutdownNow

## 常见线程池的特点和用法

* FixedThreadPool
* CachedThreadPool
* ScheduledThreadPool
* SingleThreadExecutor		
* 以上4种线程池的构造函数的参数
* 以上4种线程池对应的阻塞队列分析		
* workStealingPool是JDK1.8加入的

## 任务太多，怎么拒绝？

* 拒绝的时机
* 4种拒绝策略
  * AbortPolicy
  * DiscardPolicy
  * DiscardOldestPolicy
  * CallerRunsPolicy

## 钩子方法，给线程池加点料

* 我们可以在每个任务执行之前和之后调用beforeExecute(Thread, Runnable)和 afterExecute(Runnable, Throwable)方法。这些可以用来操纵执行环境; 例如，重新初始化ThreadLocal，收集统计信息或添加日志条目。此外，可以重写terminated()方法，以执行Executor完全终止后需要执行的任何特殊处理。
* 如果钩子或回调方法抛出异常，内部工作线程可能会失败并突然终止。
* 代码PausableThreadPoolExecutor类

## 实现原理、源码分析

* 线程池组成部分
  * 线程池管理器
  * 工作线程
  * 任务列队			
  * 任务接口（Task）
* 线程池、ThreadPoolExecutor、ExecutorService、Executor、Executors等这么多和线程池相关的类，大家都是什么关系？
  * 线程池指的是哪个类
  * 各个类的设计思想和作用
    * 核心方法纵览
      * Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；
      * 然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；
      * 抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；
      * 然后ThreadPoolExecutor继承了类AbstractExecutorService，并提供了一些新功能，比如获取核心线程数、获取任务队列等。
    * 彩蛋：如何快速查看继承关系图
      * IDEA的diagram功能，然后右键去显示方法
    * Executor
      * Executor 是一个抽象层面的核心接口，只有一个方法：
      * Executor 将任务本身和执行任务的过程解耦。
    * ExecutorService
      * ExecutorService继承了 Executor 接口，同时增加了几个有用的方法：
      * 例如提供了返回 Future 对象的submit方法，解决了Runnable无返回值的问题；
      * 例如提供了关闭线程池等方法，当调用 shutDown 方法时，线程池会停止接受新的任务，但会继续执行完毕等待中的任务。
    * Executor和ExecutorService的区别
    * Executors
      * Executors是一个工具类，就和Collections类似，方便我们来创建常见类型的线程池，例如 FixedThreadPool等。
* 线程池实现任务复用的原理
  * 原因
    * 线程重用的核心是，线程池对Thread做了包装，不重复调用hread.start()，而是自己有一个Runnable.run()，run方法里面循环在跑，跑的过程中不断检查我们是否有新加入的子Runnable对象，有新的Runnable进来的话就调一下我们的run()，其实就一个大run()把其它小run()#1,run()#2,...给串联起来了。同一个Thread可以执行不同的Runnable，主要原因是线程池把线程和Runnable通过BlockingQueue给解耦了，线程可以从BlockingQueue中不断获取新的任务
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
  * 然后我们再来看看ThreadPoolExecutor对execute的实现
  * 源码解读
* ThreadFactory
  * 用来创建到时候工作的线程，默认是DefaultThreadFactory()：
  * 从源码中可以看出，我们的线程工厂给线程设置了默认名字（pool-线程池自增编号-thread-线程的自增编号），非守护线程，默认优先级、默认线程组

## 使用线程池的注意点

* 避免任务堆积
* 避免线程数过度增加
* 排查线程泄漏

# ThreadLocal

## 两大使用场景——ThreadLocal的用途
* 典型场景1：每个线程需要一个独享的对象（通常是工具类，典型需要使用的类有SimpleDateFormat和Random）
* 典型场景2：每个线程内需要保存类似于全局变量的信息（例如在拦截器中获取用户信息，该信息在本线程执行的各方法中保持不变），可以让不同方法直接使用，却不想被多线程共享（因为不同线程获取到的用户信息不一样），避免参数传递的麻烦
* 总结
  * ThreadLocal的两个作用
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
* 按照使用时候的顺序分析
* get方法
* getMap方法
* set方法（setInitialValue方法很类似）
* initialValue方法
* remove方法
* ThreadLocalMap 类，也就是Thread.threadLocals
* 两种使用场景殊途同归

## 注意点
* 内存泄漏
  * 什么是内存泄漏
  * Key的泄漏
  * Value的泄漏
  * 如何避免内存泄露（阿里规约）
* 空指针异常
* 共享对象
* 如果可以不使用ThreadLocal就解决问题，那么不要强行使用
* 优先使用框架的支持，而不是自己创造

## 实际应用场景——在Spring中的实例分析

# 锁

## 线程要不要锁住同步资源

* 锁住
  * 悲观锁
* 不锁住
  * 乐观锁

## 多线程能否共享一把锁

* 可以
  * 共享锁
* 不可以
  * 独占锁

## 多线程竞争时，是否排队
* 排队
  * 公平锁
* 先尝试插队，插队失败再排队
  * 非公平锁

## 同一个线程是否可以重复获取同一把锁

* 可以
  * 可重入
* 不可以
  * 不可重入锁

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

## Atomic基本类型原子类，已AtomicInteger为例

* AtomicInteger 类常用方法
  * public final int get() //获取当前的值
  * public final int getAndSet(int newValue)//获取当前的值，并设置新的值
  * public final int getAndIncrement()//获取当前的值，并自增
  * public final int getAndDecrement() //获取当前的值，并自减
  * public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
  * boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
* 案例：银行存款——AtomicInteger使用方法
* AtomicInteger源码分析
  * 用Unsafe来实现底层操作
  * 用volatile修饰value字段，保证可见性
  * Unsafe的getAndAddInt方法分析：自旋 + CAS（乐观锁）。在这个过程中，通过compareAndSwapInt比较并更新value值，如果更新失败，重新获取旧值，然后更新

## AtomicArray数组类型原子类

* 数组里的元素，都可以保证原子性，AtomicIntegerArray相当于把AtomicInteger组合成一个数组，一共有3种，分别是AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
* 案例

## AtomicReference引用类型原子类

* AtomicReference
  * AtomicReference类的作用，和AtomicInteger并没有本质区别， AtomicInteger可以让一个整数保证原子性，而AtomicReference可以让一个对象保证原子性，当然，AtomicReference的功能明显比AtomicInteger强，因为一个对象里可以包含很多属性。用法和AtomicInteger类似。
  * 代码案例
* AtomicStampedReference——加上了时间戳，防止ABA问题
  * 刚才说到了AtomicReference会带来ABA问题，而AtomicStampedReference的诞生，就是解决了这个问题

## 把普通变量升级为原子类：用AtomicIntegerFieldUpdater升级原有变量

* 概述
  * 对普通变量进行升级
* 使用场景
  * 通常希望引用变量“normal”（即，不必总是通过原子类上的get或set方法引用它）

## 但偶尔需要一个原子get-set操作

* 用法，代码演示
  * AtomicIntegerFieldUpdaterDemo类
* 注意点
  * 第一，	Updater 只能 修改 它可 见 范围内 的 变量。 因为 Updater 使用 反射 得到 这个 变量。 如果 变量 不 可见， 就会 出错。 比如 如果 score 申明 为 private， 就是 不可 行的。 
  * 第二，	为了 确保 变量 被 正确 的 读取， 它 必须 是 volatile 类型 的。 如果 我们 原有 代码 中 未 申明 这个 类型， 那么 简单 地 申明 一下 就 行， 这不 会 引起 什么 问题。 
  * 第三，	由于 CAS 操作 会 通过 对象 实例 中的 偏移量 直接进行 赋值， 因此， 它不 支持 static 字段（ Unsafe. objectFieldOffset() 不支持 静态 变量）

## Adder累加器

* 介绍
  * 是Java 8引入的，相对是比较新的一个类。
  * 高并发下LongAdder比AtomicLong效率高，不过本质是空间换时间。
  * Atomic遇到的问题是，适合用在低并发场景，否则在高并发下，由于CAS的冲突机会大，会导致经常自旋，影响整体效率。而LongAdder引入了分段锁的概念，当竞争不激烈的时候，所有线程都是通过CAS对同一个变量（Base）进行修改，但是等到了竞争激烈的时候，LongAdder把不同线程对应到不同的Cell上进行修改，降低了冲突的概率，是多段锁的理念，提高了并发性
* 演示AtomicLong的问题
  * 这里演示多线程情况下AtomicLong的性能，有16个线程对同一个AtomicLong累加。
  * 由于竞争很激烈，每一次加法，都要flush和refresh，导致很耗费资源
* LongAdder带来的改进和原理
  * 在内部，这个LongAdder的实现原理和刚才的AtomicLong是有不同的，刚才的AtomicLong的实现原理是，每一次加法都需要做同步，所以在高并发的时候会导致冲突比较多，也就降低了效率
  * 而此时的LongAdder，每个线程会有自己的一个计数器，仅用来在自己线程内计数，这样一来就不会和其他线程的计数器干扰。
  * 如图中所示，第一个线程的计数器数值，也就是ctr’，为1的时候，可能线程2的计数器ctr’’的数值已经是3了，他们之间并不存在竞争关系,所以在加和的过程中，根本不需要同步机制，也不需要刚才的flush和refresh。这里也没有一个公共的counter来给所有线程统一计数。
  * 可能聪明的小伙伴已经想到了，LongAdder最终是如何实现多线程计数的呢？答案就在最后一步，执行LongAdder.sum()的时候，这里是唯一需要同步的地方：
  * 当我们执行sum函数的时候，LongAdder会把所有线程的计数器，也就是ctr’和ctr’’等等都在同步的情况下加起来，形成最终的总和：
  * AtomicLong在多线程的情况下，每次都要同步，而LongAdder仅在最后sum的时候需要同步，其他情况下，多个线程可以同时运行，这就是LongAdder的吞吐量比AtomicLong大的原因，本质是空间换时间
* Accumulator累加器
  * Accumulator和Adder非常相似，Accumulator就是一个更通用版本的Adder
  * 用法
  * LongAccumulator的构造函数的第一个参数是一个表达式，第二个参数是x的初始值。x是每次的初始值，y是结果。执行counter.accumulate(1)的时候，第一次x是0，y是1，后面每次的y的结果会赋值给x，然后每次的新y就是counter.accumulate(1)传入的1。
  * 拓展功能
  * 适用场景

# CAS

## 什么是CAS

* 思路
  * 并发
  * 我认为V的值应该是A，如果是的话那我就把它改成B，如果不是A（说明被别人修改过了），那我就不修改了，避免多人同时修改导致出错。
  * CAS有三个操作数：内存值V、预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才将内存值修改为B，否则什么都不做。最后返回现在的V值。
  * CPU的特殊原子指令
* CAS的等价代码（语义）

## 案例演示

* 两个线程竞争，其中一个落败
* 用CAS思想实现的计数器

## 应用场景

* 乐观锁
* 并发容器
* 原子类

## 以AtomicInteger为例，分析在Java中是如何利用CAS实现原子操作的？

* AtomicInteger加载Unsafe工具，用来直接操作内存数据
* Unsafe类
  * Unsafe是CAS的核心类。Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。
  * valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的原值的，这样我们就能通过unsafe来实现CAS了。
  * value是用volatile修饰的，保证了多线程之间看到的value值是同一份。
* getAndAdd方法
* Unsafe类中的compareAndSwapInt方法
  * 方法中先想办法拿到变量value在内存中的地址。
  * 通过Atomic::cmpxchg实现原子性的比较和替换，其中参数x是即将更新的值，参数e是原内存的值。至此，最终完成了CAS的全过程

## 缺点

* ABA问题
* 自旋时间过长
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
* 我们总结出，满足以下条件时，对象才是不可变的
  * 对象创建后，其状态就不能修改
  * 所有属性都是final修饰的
  * 对象创建过程中没有发生逸出

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
    * 同时put扩容导致数据丢失
    * 死循环造成的CPU100%
    * 彩蛋：调试技巧——如何修改JDK版本，从8到7
      * File-Project Structure，下载JDK7然后安装，然后添加JDK进来，然后选择。新建module选择JDK7，可以不影响到当前JDK8的代码。
    * 彩蛋：调式技巧——多线程配合，模拟真实场景
      * 如果不会这个调试技巧，永远都无法在本地模拟出线上bug的情况的，因为出现的几率太低了。
    * HashMap在高并发下的死循环（仅在JDK7及以前存在）
      * 代码演示
* 九层之台，起于累土、罗马不是一天建成的：HashMap分析
  * 结构图
  * 红黑树介绍
    * 红黑树是每个节点都带有颜色属性的二叉查找树，本质是对二叉查找树BST的一种平衡策略，颜色为红色或黑色。
    * 我们理解为是一种平衡二叉查找树就可以，查找效率高，会自动平衡，防止极端不平衡从而影响查找效率的情况发生。
  * HashMap关于并发的特点
* JDK1.7的ConcurrentHashMap实现和分析
  * 整体概念
    * Java 7中的ConcurrentHashMap最外层是多个segment，每个segment的底层数据结构与HashMap类似，仍然是数组和链表组成的拉链法。
    * 每个segment独立上ReentrantLock锁，每个segment之间互不影响，提高了并发效率。
    * ConcurrentHashMap 默认有 16 个 Segments，所以最多可以同时支持 16 个线程并发写（操作分别分布在不同的 Segment 上）。这个默认值可以在初始化的时候设置为其他值，但是一旦初始化以后，是不可以扩容的
  * Segment图解
* JDK1.8的ConcurrentHashMap实现和源码分析
  * 简介
  * 结构
  * 源码分析（1.8）
* 对比JDK1.7和1.8的优缺点（为什么要把1.7的结构改成1.8的结构）
* 组合操作：ConcurrentHashMap也不是线程安全的？
* 实际生产案例

## CopyOnWriteArrayList

* 诞生的历史和原因
  * Vector和SynchronizedList的锁的粒度太大，并发效率相对比较低，并且迭代时无法编辑
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
* 实现原理
  * CopyOnWrite的含义
  * 创建新副本、读写分离
  * “不可变”原理
  * 迭代的时候
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
  * 彩蛋：画漂亮的UML图
* ArrayBlockingQueue
  * 有界
  * 指定容量
  * 公平
* LinkedBlockingQueue
  * 无界
    * 容量Integer.MAX_VALUE
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
* 非阻塞队列ConcurrentLinkedQueue
  * 并发包中的非阻塞队列只有ConcurrentLinkedQueue这一种，顾名思义ConcurrentLinkedQueue是使用链表作为其数据结构的，使用 CAS 非阻塞算法来实现线程安全（不具备阻塞功能），适合用在对性能要求较高的并发场景。用的相对比较少一些
* 如何选择适合自己的队列？
  * 边界
  * 空间
  * 吞吐量

## 各并发容器总结

* java.util.concurrent包提供的容器，分为3类：Concurrent、CopyOnWrite、Blocking

# 控制并发流程

## 什么是控制并发流程？

## CountDownLatch倒计时

* 作用
* 两个典型用法
  * 主要方法介绍
* 注意点
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
  * 主要方法介绍
    * new Semaphore(int permits, boolean fair)
    * tryAcquire()
    * tryAcquire(timeout)
    * availablePermits
    * acquire()
    * acquireUninterruptibly()
    * release()
  * 示例代码
  * 特殊用法
    * 一次性获取或释放多个许可证
      * 使用场景
        * 比如TaskA会调用很消耗资源的method1()，而TaskB调用的是不太消耗资源的method2()，假设我们一共有5个许可证。那么我们就可以要求TaskA获取5个许可证才能执行，而TaskB只需要获取到一个许可证就能执行，这样就避免了A和B同时运行的情况，我们可以根据自己的需求合理分配资源

* 注意点

  1. 获取和释放的许可证数量必须一致，否则比如每次都获取2个但是只释放1个甚至不释放，随着时间的推移，到最后许可证数量不够用，会导致程序卡死。（虽然信号量类并不对是否和获取的数量做规定，但是这是我们的编程规范，否则容易出错）

  2.	注意在初始化Semaphore的时候设置公平性，一般设置为true会更合理
  3.	并不是必须由获取许可证的线程释放那个许可证，事实上，获取和释放许可证对线程并无要求，也许是A获取了，然后由B释放，只要逻辑合理即可。
  4.	信号量的作用，除了控制临界区最多同时有N个线程访问外，另一个作用是可以实现“条件等待”，例如线程1需要在线程2完成准备工作后才能开始工作，那么就线程1acquire()，而线程2完成任务后release()，这样的话，相当于是轻量级的CountDownLatch。
  5.	可以跨线程/跨线程池共享同一个信号量

## Condition接口（又称条件对象）

* 作用
  * 当线程1需要等待某个条件的时候，它就去执行condition.await()方法，一旦执行了await()方法，线程就会进入阻塞状态。
  * 然后通常会有另外一个线程，假设是线程2，去执行对应的条件，直到这个条件达成的时候，线程2就会去执行condition.signal()方法，这时JVM就会从被阻塞的线程里找，找到那些等待该condition的线程，当线程1就会收到可执行信号的时候，它的线程状态就会变成Runnable可执行状态。
  * signalAll()和signal()区别
    * signalAll()会唤起所有的正在等待的线程。
    * 但是signal()是公平的，只会唤起那个等待时间最长的线程，在当前情况下，是thread-0。
* 代码演示
  * 普通示例
  * 用Condition实现生产者消费者模式
* 注意点
* 源码分析
* 面试常见问题
  * Condition和object.wait()和notify()的关系？

## CyclicBarrier循环栅栏

* 作用
* 代码例子
* 面试常见问题
  * CyclicBarriar 和 CountdownLatch 有什么区别？

# AQS

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
* 如果没有AQS
  * 就需要每个协作工具自己实现：
  * 同步状态的原子性管理；
  * 线程的阻塞与解除阻塞；
  * 队列的管理；
  * 在并发场景下，自己正确且高效实现这些内容，是相当有难度的，所以我们用AQS来帮我们把这些脏活累活都搞定，我们只关注业务逻辑就够了

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

## AQS内部原理解析

* AQS最核心的就是三大部分：state、期望协作工具类去实现的获取/释放等重要方法、控制线程抢锁和配合的FIFO队列。我们对着3个模块分别展开：
* state状态
  * state用来表示“锁”的占有情况。
  * AQS的核心思想是基于volatile int state这样的一个属性同时配合Unsafe工具对其原子性的操作，来实现对当前锁的状态进行修改。当state的值为0的时候，标识改Lock不被任何线程所占有。
  * 这里的state的具体含义，会根据具体实现类的不同而不同，比如在Semaphore里，它表示“剩余的许可证的数量”，而在CountDownLatch里，它表示“还需要倒数的数量”。
  * state是volatile修饰的，会被并发地修改，所以所有修改state的方法都需要保证线程安全，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于j.u.c.atomic包的支持。
  * 这里可能会疑惑，那你setState的时候，没有做任何并发处理啊，这里就要说到volatile的作用了.......
* FIFO队列
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
* AQS在Semaphore的应用
  * 在Semaphore中，state表示许可证的剩余数量，
  * 看tryAcquire方法，判断nonfairTryAcquireShared大于等于0的话，代表成功，然后看：
  * 这里会先检查剩余许可证数量够不够这次需要的，用减法来计算，如果直接不够，那就返回负数，表示失败，如果够了，就用自旋加compareAndSetState来改变state状态，直到改变成功就返回正数；或者是期间如果被其他人修改了导致剩余数量不够了，那也返回负数代表获取失败。
* AQS在ReentrantLock的应用
  * 分析释放锁的方法tryRelease
    * 由于是可重入的，所以state代表重入的次数，每次释放锁，先判断是不是当前持有锁的线程释放的，如果不是就抛异常，如果是的话，重入次数就减一，如果减到了0，就说明完全释放了，于是free就是true，并且把state设置为0。
  * 加锁的方法

# 获取子线程的执行结果

## Runnable的缺陷

1. 不能返回一个返回值
2. 也不能抛出checked Exception（代码演示）

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
  * get(long timeout, TimeUnit unit)方法：有超时的获取结果
  * cancel()方法：取消任务的执行
  * isDone()方法：判断线程是否执行完毕
  * isCancelled()方法：判断是否被取消
* 用法1：线程池的submit方法返回Future对象
* 用法2：用FutureTask来创建Future

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
* 缺点
  * 性能差
  * 当多个线程同时想计算的时候，需要慢慢等待，严重时，性能甚至比不用缓存更差

## 性能待优化——引出锁性能优化经验：缩小锁的粒度

* 当然，这样加锁虽然提高了并发效率，但是并不意味着就是线程安全的，还需要考虑到同时读写等情况，但是，其实没必要自己实现线程安全的HashMap，也不应该加synchronized，因为我们自己实现的性能远不如现有的并发集合。我们来使用ConcurrentHashMap优化我们的缓存：

## 用并发集合——引出ConcurrentHashMap

* 代码和修改过程
  * 用ConcurrentHashMap来代替HashMap，由于ConcurrentHashMap自身是线程安全的，所以我们无需自己处理HashMap的线程安全问题，也无须手动synchronized，同时也提高了并发时的效率，因为ConcurrentHashMap的并发度远远高于synchronized修饰的HashMap。
* 缺点
  * 在计算完成前，另一个要求计算相同值的请求到来，会导致计算两遍，这和缓存想避免多次计算的初衷恰恰相反，是不可接受的

## 避免重复计算——引出Future和Callable的妙用

* 动机
  * 现在不同的线程进来以后，确实可以同时计算，但是如果两个线程脚前脚后，也就是相差无几的进来请求同一个数据，那么我们来看看会出现什么问题：重复计算
  * 这个例子只有2个线程，并不可怕，但是如果是100个线程都请求同样的内容，却都需要重新计算，那么会造成巨大的浪费。
* 改进方向
  * 前人种树，后人乘凉

## 依然存在重复的可能——更好的办法是用原子操作putIfAbsent

* 如果有两个同时计算666的线程，同时调用cache.get方法，那么返回的结果都为null，后面还是会创建两个任务去计算相同的值

## 计算中抛出异常——引出ExcecutionException

* 对异常的处理

## 计算期间任务被取消——处理CancellationException

## 正确的异常处理逻辑——各司其职

* 停止线程的正确方法

## 处理异常的正确逻辑

* 抛出或真正处理或复现该异常，不能自己吞掉

## 考虑“缓存污染”问题

* 计算失败则移除Future，增加健壮性
* 演示缓存污染带来的问题，无论是计算错误还是被取消，都应该用cache.remove把缓存清理掉，这样后续的计算才可能成功

## 缓存过期功能

* 用FutureTask的子类，为每个结果指定过期时间，并定期扫描过期的元素

* 第二次计算设置了缓存时间，导致第三次计算需要重新计算。
* 出于安全性考虑，缓存需要能够设置有效期，到期自动失效，否则缓存一直不失效的话，会带来很多缓存过期、不一致的问题

## 过量的缓存需要被清理

* LRU和FRU算法

## 测试缓存效果

* 用多种不同的线程的创建方式来创建线程
* 用实现Runnable接口的方式和继承Thread类的方法实现线程，效果一样

## 模拟大量请求，观测缓存效果

* 引出线程池相关知识、选择合适的线程数
* 用线程池创建大量线程get，用了缓存后，总体耗时大大减少，体现了缓存的作用

## 想测并发性能，所有线程一同访问缓存——CountDownLatch

* 前一个类存在一个问题，就是大量的请求实际上不是同时到达的，而是分先后，但是这样就没办法给缓存造成压力，我们需要真正的同一时刻大量请求到达，此时可以用CountDownLatch来实现

## 每个线程都有存储独立信息的需求——引出用ThreadLocal、线程的各属性

* 每个线程都想要打印当前时间

## 高并发访问时，第一次都拿不到缓存，导致打爆cpu和MySQL，造成缓存雪崩、缓存击穿等高并发下的缓存问题

* 把缓存的过期时间设置为随机，就避免了缓存雪崩

## 总结ImoocCache的亮点和涉及的知识列表

* 写一个缓存，用到了这么多的并发知识，以后有了这些知识作为储备，学习其他的优秀源码，比如Guava Cache，就容易多了

# 总结

