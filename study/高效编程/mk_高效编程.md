![image-20191208090548903](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191208090548903.png)

# Lambda表达式

* 函数式编程
  * 函数可以赋值给变量，作为参数或者返回值进行传递

* 可以理解为一种匿名函数的替代
* 通过行为参数化传递代码

```java
// 形式
(parameters) -> expression
(parameters) -> {statement;}
// 形式一：没有参数
()->System.out.println("Hello World");
// 形式二：只有一个参数
name->System.out.println("Hello World" + name);
// 形式三：没有参数，逻辑复杂
()->{
  System.out.println("Hello");
  System.out.println("World");
}
// 形式四：包含两个参数的方法
BinaryOperator<Long> functionAdd = (x,y)->x+y;
Long result = functionAdd.apply(1L,2L);
// 形式五：对参数显示声明
BinaryOperator<Long> functionAdd = (Long x,Long y)->x+y;
Long result = functionAdd.apply(1L,2L);
```

* 函数式接口

  * 接口中只有一个抽象方法
  * Java8的函数式接口注解：@FunctionInterface
  * 函数式接口的抽象方法签名：函数描述符

  

![image-20191208112500815](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191208112500815.png)

```java
//文件处理函数式接口
@FunctionalInterface
public interface FileConsumer {
     // 函数式接口抽象方法
     // @param fileContent - 文件内容字符串
    void fileHandler(String fileContent);
}
```

```java
//文件服务类
public class FileService {
    //从过url获取本地文件内容，调用函数式接口处理
    public void fileHandle(String url, FileConsumer fileConsumer)
            throws IOException {
        // 创建文件读取流
        BufferedReader bufferedReader = new BufferedReader(
                new InputStreamReader(new FileInputStream(url)));
        // 定义行变量和内容sb
        String line;
        StringBuilder stringBuilder = new StringBuilder();
        // 循环读取文件内容
        while ((line = bufferedReader.readLine()) != null) {
            stringBuilder.append(line + "\n");
        }
        // 调用函数式接口方法，将文件内容传递给lambda表达式，实现业务逻辑
        fileConsumer.fileHandler(stringBuilder.toString());
    }
}
```

```java
@Test
public void fileHandle() throws IOException {
  FileService fileService = new FileService();
  // TODO 此处替换为本地文件的地址全路径
  String filePath = "";
  // 通过lambda表达式，打印文件内容
  fileService.fileHandle(filePath, System.out::println);
}
```

* 方法引用::

  ```java
  // 指向静态方法的方法引用
  public void test1() {
    Consumer<String> consumer1 = (String number)->Integer.parseInt(number);
    Consumer<String> consumer2 = Integer::parseInt;
  }
  // 指向任意类型实例方法的方法引用
  public void test2() {
    Consumer<String> consumer1 = (String str)->str.length();
    Consumer<String> consumer2 = String::length;
  }
  // 指向现有对象的实例方法的方法引用
  public void test3(){
    StringBuilder stringBuilder = new StringBuilder();
    Consumer<String> consumer1 = (String str)->stringBuilder.append(str);
    Consumer<String> consumer2 = stringBuilder::append;
  }
  ```

# 流编程

* 流
  * JDK1.8引入的新成员，以声明式方式处理集合数据
  * 将基础操作链接起来，完成复杂的数据处理流水线
  * 提供透明的并行处理
  * 只能遍历一次，内部迭代

```java
// 集合排序
/**
* 排序
*/
notBooksSkuList.sort(new Comparator<Sku>() {
  @Override
  public int compare(Sku sku1, Sku sku2) {
    if (sku1.getTotalPrice() > sku2.getTotalPrice()) {
      // 从大到小
      return -1;
    } else if (sku1.getTotalPrice() < sku2.getTotalPrice()) {
      return 1;
    } else {
      return 0;
    }
  }
});

List<Stu> sortedRes = listData.stream().sorted(new Comparator<Stu>() {
  @Override
  public int compare(Stu o1, Stu o2) {
    if (o1.getAge() > o2.getAge()) {
      // 从大到小
      return -1;
    } else if (o1.getAge() < o2.getAge()) {
      return 1;
    } else {
      return 0;
    }
  }
}).collect(Collectors.toList());
```

```java
//流
AtomicReference<Double> money = new AtomicReference<>(Double.valueOf(0.0));
List<String> resultSkuNameList =
  CartService.getCartSkuList()
  .stream()
  // 1 打印商品信息
  .peek(sku -> System.out.println(JSON.toJSONString(sku, true)))
  // 2 过滤掉所有图书类商品
  .filter(sku -> !SkuCategoryEnum.BOOKS.equals(
    sku.getSkuCategory()))
  // 排序
  .sorted(Comparator.comparing(Sku::getTotalPrice).reversed())
  // TOP2
  .limit(2)
  // 累加商品总金额
  .peek(sku -> money.set(money.get() + sku.getTotalPrice()))
  // 获取商品名称
  .map(sku -> sku.getSkuName())
  // 收集结果
  .collect(Collectors.toList());
```

```java
// 从支持数据处理操作的源生成的元素序列
```

![image-20191208215644497](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191208215644497.png)

![image-20191208215807369](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191208215807369.png)

![image-20191208215848666](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191208215848666.png)

```java
// filter使用：过滤掉不符合断言判断的数据
list.stream()
	.filter(sku -> SkuCategoryEnum.BOOKS.equals(sku.getSkuCategory()))
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// map使用：将一个元素转换成另一个元素
list.stream()
	.map(sku -> sku.getSkuName())       // sku被转换成sku.getSkuName()的值
  .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// flatMap使用：将一个对象转换成流
list.stream()
	.flatMap(sku -> Arrays.stream(sku.getSkuName().split("")))
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// peek使用：对流中元素进行遍历操作，与forEach类似，但不会销毁流元素
list.stream()
	.peek(sku -> System.out.println(sku.getSkuName()))
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// sort使用：对流中元素进行排序，可选则自然排序或指定排序规则。有状态操作
list.stream()
	.peek(sku -> System.out.println(sku.getSkuName()))
	.sorted(Comparator.comparing(Sku::getTotalPrice))
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// distinct使用：对流元素进行去重。有状态操作
list.stream()
  .map(sku -> sku.getSkuCategory())
	.distinct()
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// skip使用：跳过前N条记录。有状态操作
list.stream()
  .sorted(Comparator.comparing(Sku::getTotalPrice))
	.skip(3)
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// limit使用：截断前N条记录。有状态操作
list.stream()
  .sorted(Comparator.comparing(Sku::getTotalPrice))
	.skip(2 * 3)
	.limit(3)
	.forEach(item -> System.out.println(JSON.toJSONString(item, true)));
// allMatch使用：终端操作，短路操作。所有元素匹配，返回true
boolean match = list.stream()
										.peek(sku -> System.out.println(sku.getSkuName()))
										.allMatch(sku -> sku.getTotalPrice() > 100);
// anyMatch使用：任何元素匹配，返回true
boolean match = list.stream()
										.peek(sku -> System.out.println(sku.getSkuName()))
										.anyMatch(sku -> sku.getTotalPrice() > 100);
// noneMatch使用：任何元素都不匹配，返回true
boolean match = list.stream()
										.peek(sku -> System.out.println(sku.getSkuName()))
										.noneMatch(sku -> sku.getTotalPrice() > 10_000);
// 找到第一个
Optional<Sku> optional = list.stream()
                						 .peek(sku -> System.out.println(sku.getSkuName()))
														 .findFirst();
System.out.println(JSON.toJSONString(optional.get(), true));
// 找任意一个
Optional<Sku> optional = list.stream()
                						 .peek(sku -> System.out.println(sku.getSkuName()))
                						 .findAny();
System.out.println(JSON.toJSONString(optional.get(), true));
// max使用
OptionalDouble optionalDouble = list.stream()
                										.mapToDouble(Sku::getTotalPrice)
                										.max();
System.out.println(optionalDouble.getAsDouble());
// min使用
OptionalDouble optionalDouble = list.stream()
                										.mapToDouble(Sku::getTotalPrice)
																		.min();
System.out.println(optionalDouble.getAsDouble());
// count使用
long count = list.stream().count();
```

```java
// 熟练
List<String> list = people.stream().map(Person::getName).collect(Collectors.toList());
Set<String> set = 		  
  		people.stream().map(Person::getName).collect(Collectors.toCollection(TreeSet::new));
String joined = things.stream().map(Object::toString).collect(Collectors.joining(", "));
int total = employees.stream().collect(Collectors.summingInt(Employee::getSalary)));
Map<Department, List<Employee>> byDept
         = employees.stream().collect(Collectors.groupingBy(Employee::getDepartment));
Map<Department, Integer> totalByDept
         = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment,                                                 Collectors.summingInt(Employee::getSalary)));
Map<Boolean, List<Student>> passingFailing =
         students.stream()
                 .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
```

```java
// 流的四种构建形式
//1. 由数值直接构建流
Stream stream = Stream.of(1, 2, 3, 4, 5);
//2. 通过数组构建流
int[] numbers = {1, 2, 3, 4, 5};
IntStream stream = Arrays.stream(numbers);
//3. 通过文件生成流
String filePath = "";   // TODO 此处替换为本地文件的地址全路径
Stream<String> stream = Files.lines(Paths.get(filePath));
//4. 通过函数生成流（无限流）
Stream stream = Stream.generate(Math::random);
stream.limit(100).forEach(System.out::println);
```

* 收集器简介
  * 将流中的元素累积成一个结果
  * 作用于终端操作collect()上
  * collect / Collect / Collectors
* ==分组分区==

```java
// 常见预定义收集器使用
	// 将流元素归约和汇总为一个值
	// 将流元素分组
	// 将流元素分区
// 集合收集器
List<Sku> result = list.stream()
                			 .filter(sku -> sku.getTotalPrice() > 100)
                			 .collect(Collectors.toList());
// 分组
// Map<分组条件，结果集合>
Map<Object, List<Sku>> group = list.stream()
                .collect(Collectors.groupingBy(sku -> sku.getSkuCategory()));
// 分区
Map<Boolean, List<Sku>> partition = list.stream()
     .collect(Collectors.partitioningBy(sku -> sku.getTotalPrice() > 100));
```

# 资源关闭

* 垃圾回收（GC）的特点
  * 垃圾回收机制只负责回收堆内存资源，不会回收任何物理资源
  * 程序无法精确控制垃圾回收动作的具体发生时间
  * 在垃圾回收之前，总会先调用它的finalize方法
* 常见需要手动释放的物理资源
  * 文件/流资源
  * 套接字资源
  * 数据库连接资源
* 物理资源可以不手动释放吗？
  * 资源被长时间无效占用
  * 超过最大限制后，将无资源可用
  * 导致系统无法正常运行 

```java
/**
 * 基于JDK7之后，实现正确关闭流资源方法
 * try - with - resource
 */
public class NewFileCopyTest {
    @Test
    public void copyFile() {
        // 先定义输入/输出路径
        String originalUrl = "lib/NewFileCopyTest.java";
        String targetUrl = "targetTest/new.txt";
        // 初始化输入/输出流对象
        try (
                FileInputStream originalFileInputStream =
                        new FileInputStream(originalUrl);
                FileOutputStream targetFileOutputStream =
                        new FileOutputStream(targetUrl);
        ) {
            int content;
            // 迭代，拷贝数据
            while ((content = originalFileInputStream.read()) != -1) {
                targetFileOutputStream.write(content);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

* try-catch-resource简介
  * Java7引入新特性
  * 优雅关闭资源
  * 一种Java语法糖
  * 多资源自动关闭
  * 实现AutoCloseable接口
  * 避免异常屏蔽
* 资源关闭顺序问题
  * 先开后关原则
  * 从外到内原则
  * 底层资源单独声明原则
* 资源关闭特殊情况
  * 资源对象被return的情况下，由调用方关闭
  * ByteArrayInputStream等不需要检查关闭的资源对象
  * 使用Socket获取的InputStream和OutputStream对象不需要关闭

# 工具集

## Google Guava

* Guava工程包含了若干被Google的Java项目广泛依赖的核心库，例如：集合、缓存、原生类型支持、并发库、通用注解、字符串处理、I/O等等

### 使用和避免null

* 大多数情况下，使用null表明的是某种缺失情况
* `Guava`引入`Optional<T>`表明可能为null的T类型引用。Optional实例可能包含非null的引用（引用存在），也可能什么也不包括（引用缺失）

```java
// Java8中的Optional使用方法
// 三种创建Optional对象方式
// 创建空的Optional对象
	Optional.empty();
// 使用非null值创建Optional对象
	Optional.of("zhangxiaoxi");
// 使用任意值创建Optional对象
	Optional optional = Optional.ofNullable("zhangxiaoxi");
// 判断是否引用缺失的方法(建议不直接使用)
	optional.isPresent();
// 当optional引用存在时执行
// 类似的方法：map filter flatMap
	optional.ifPresent(System.out::println);
// 当optional引用缺失时执行
	optional.orElse("引用缺失");
	optional.orElseGet(() -> {
	// 自定义引用缺失时的返回值
		return "自定义引用缺失";
	});
	optional.orElseThrow(() -> {
		throw new RuntimeException("引用缺失异常");
	});

// 可以避免 list为null时报错
Optional.ofNullable(list)
		.map(List::stream)
		.orElseGet(Stream::empty)
		.forEach(System.out::println);
```

### 不可变集合

* 创建对象的不可变拷贝是一项很好的防御性编程技巧
* Guava为所有JDK标准集合类型和Guava新集合类型都提供了简单易用的不可变版本

* 不可变对象的优点
  * 当对象被不可信的库调用时，不可变形式是安全的
  * 不可变对象被多个线程调用时，不存在竞态条件问题
  * 不可变集合不需要考虑变化，因此可以节省时间和空间
  * 不可变对象因为有固定不变，可以作为常量来安全使用

```java
// 不可变集合的三种创建方式
//1. copyOf方法：ImmutableSet.copyOf(set)
//2. of方法：ImmutableSet.of("a","b","c")
//3. Builder工具：ImmutableSet.builder().build()
// 构造不可变集合对象三种方式
	// 通过已经存在的集合创建
	ImmutableSet.copyOf(list);
	// 通过初始值，直接创建不可变集合
	ImmutableSet immutableSet = ImmutableSet.of(1, 2, 3);
	// 以builder方式创建
	ImmutableSet.builder()
		.add(1)
		.addAll(Sets.newHashSet(2, 3))
		.add(4)
		.build();
```

### 新集合类型

* Multiset
  * 没有元素顺序限制的ArrayList(E)
    * add(E)：添加单个给定元素
    * iterator()：返回一个迭代器，包含Multiset所有元素（包括重复元素）
    * size()：返回所有元素的总个数（包括重复元素）
  * Map<E,Integer>，键为元素，值为计数
    * count(Object)：返回给定元素的计数
    * entrySet()：返回Set<Multiset.Enty<E>>，和Map的entrySet类似
    * elementSet()：返回所有不重复元素的Set<E>，和Map的keySet类似
* Multiset与Map的区别
  * 元素计数只能是正数
  * multiset.size()返回集合大小
  * multiset.iterator()会迭代重复元素
  * multiset支持直接设置元素的计数
  * 没有的元素，multiset.count(E)为0
* 多种Multiset的实现
  * HashMultiset
  * TreeMultiset
  * LinkedHashMultiset
  * ConcurrentHashMultiset
  * ImmutableMultiset

```java
/**
 * 实现：使用Multiset统计一首古诗的文字出现频率
 */
public class MultisetTest {
    private static final String text =
            "《南陵别儿童入京》" +
            "白酒新熟山中归，黄鸡啄黍秋正肥。" +
            "呼童烹鸡酌白酒，儿女嬉笑牵人衣。" +
            "高歌取醉欲自慰，起舞落日争光辉。" +
            "游说万乘苦不早，著鞭跨马涉远道。" +
            "会稽愚妇轻买臣，余亦辞家西入秦。" +
            "仰天大笑出门去，我辈岂是蓬蒿人。";

    @Test
    public void handle() {
        // multiset创建
        Multiset<Character> multiset =
                HashMultiset.create();
        // string 转换成 char 数组
        char[] chars = text.toCharArray();
        // 遍历数组，添加到multiset中
        Chars.asList(chars)
                .stream()
                .forEach(charItem -> {
                    multiset.add(charItem);
                });
        System.out.println("size : " + multiset.size());  //105
        System.out.println("count : " + multiset.count('人')); // 2
    }
}
```

```java
// Lists / Sets 使用
// Sets工具类的常用方法：并集 / 交集 / 差集 / 分解集合中的所有子集 / 求两个集合的笛卡尔积
// Lists工具类的常用方式：反转 / 拆分
private static final Set set1 = Sets.newHashSet(1, 2);
private static final Set set2 = Sets.newHashSet(4);
// 并集
Set<Integer> set = Sets.union(set1, set2);
// 交集
Set<Integer> set = Sets.intersection(set1, set2);
// 差集：如果元素属于A而且不属于B
Set<Integer> set = Sets.difference(set1, set2);
// 相对差集：属于A而且不属于B 或者 属于B而且不属于A
set = Sets.symmetricDifference(set1, set2);
// 拆分所有子集合
Set<Set<Integer>> powerSet = Sets.powerSet(set1);
// 计算两个集合笛卡尔积
Set<List<Integer>> product = Sets.cartesianProduct(set1, set2);
// 拆分
List<Integer> list = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7);
List<List<Integer>> partition = Lists.partition(list, 3);
// 反转
List<Integer> list = Lists.newLinkedList();
	list.add(1);
	list.add(2);
	list.add(3);
List<Integer> newList = Lists.reverse(list);
```

### IO工具类

* 对字节流/字符流提供的工具方法，需手动关闭流
  * ByteStreams：提供对InputStream/OutputStream的操作
  * CharStreams：提供对Reader/Writer的操作
* 对源（Source）与汇（Sink）的抽象，自动关闭
  * 源是可读的：ByteSource/CharSource
  * 汇是可写的：ByteSink / CharSink

```java
// 使用流(Source)与汇(Sink)来对文件进行常用操作
@Test
public void copyFile() throws IOException {
//创建对应的Source和Sink
  CharSource charSource = Files.asCharSource(new File("SourceText.txt"),
    Charsets.UTF_8);
  CharSink charSink = Files.asCharSink(new File("TargetText.txt"),
    Charsets.UTF_8);
// 拷贝
  charSource.copyTo(charSink);
}
```

# 线程池

* 事先创建若干个可执行的线程放入一个容器中，需要的时候从容器中获取线程不用自行创建，使用完毕不用销毁线程而是放回容器中，从而减少创建和销毁线程对象的开销

* 好处：

  * 降低资源消耗
  * 提高响应速度
  * 提高线程的可管理性

* ==线程池的简单设计==

  * 首先得有个线程容器，能开启、初始化、关闭，能让外界获取容器内线程，使用完毕后归还线程到容器

  * 线程池初始化时应该创建多少线程？线程池内线程用完了怎么处理？缓冲数组要多长？缓冲数组满了怎么办？

  * 改进

    ![image-20191210145639813](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191210145639813.png)

* 线程池的核心参数

  * 核心线程数量：corePoolSize
  * 最大线程数量：maximumPoolSize
  * 线程空闲后的存活时间：KeepAliveTime
  * 时间单位：unit
  * 用于存放任务的阻塞队列：workQueue
  * 线程工厂类：threadFactory
  * 当队列和最大线程池都满了之后的饱和策略：handler

* 线程池处理流程

  ![image-20191210150949013](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191210150949013.png)

* 线程池可选择的阻塞队列

  * 无界队列：LinkedBlockingQueue不设置初始值就是无界队列
  * 有界队列：ArrayBlockingQueue
  * 同步移交队列：SynchronousQueue

* 线程池可选择的饱和策略

  * AbortPolicy终止策略（默认）
  * DiscardPolicy抛弃策略
  * DiscardOldestPolicy抛弃旧任务策略
  * CallerRunsPolicy调用者运行策略

* 线程池执行示意图

  ![image-20191210170331118](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191210170331118.png)

* 常用线程池

  ```java
  // 线程数量无限线程池
  public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runable>());
  }
  // 线程数量固定线程池
  public static ExecutorService newFixedThreadPool(int nThreads ) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runable>());
  }
  // 单一线程线程池
  public static ExecutorService newSingleThreadPool() {
    return new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runable>());
  }
  ```

* 线程池的状态

  ![image-20191210221339506](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191210221339506.png)

  

```java
// 基于数组的有界阻塞队列，队列容量为10
ArrayBlockingQueue queue = new ArrayBlockingQueue<Integer>(10);
// 循环向队列添加元素
for (int i = 0; i < 20; i++) {
  queue.put(i);
  System.out.println("向队列中添加值：" + i);
}

// 基于链表的有界/无界阻塞队列，队列容量为10
LinkedBlockingQueue queue = new LinkedBlockingQueue<Integer>();
// 循环向队列添加元素
for (int i = 0; i < 20; i++) {
  queue.put(i);
  System.out.println("向队列中添加值：" + i);
}

// 同步移交阻塞队列
SynchronousQueue queue = new SynchronousQueue<Integer>();
// 插入值
new Thread(() -> {
  try {
    queue.put(1);
    System.out.println("插入成功");
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
}).start();
// 删除值
new Thread(() -> {
  try {
    queue.take();
    System.out.println("删除成功");
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
}).start();
// Thread.sleep(1000L * 60);
```

```java
// 创建线程池
ExecutorService threadPool = Executors.newCachedThreadPool();
// 利用submit方法提交任务，接收任务的返回结果
Future<Integer> future = threadPool.submit(() -> {
	Thread.sleep(1000L * 10);
	return 2 * 5;
});
// 阻塞方法，直到任务有返回值后，才向下执行
Integer num = future.get();

// 创建线程池
ExecutorService threadPool = Executors.newCachedThreadPool();
// 利用execute方法提交任务，没有返回结果
threadPool.execute(() -> {
  try {
    Thread.sleep(1000L * 10);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  Integer num = 2 * 5;
  System.out.println("执行结果：" + num);
});
```

```java
// 新的处理方式
@Test
public void newHandle() throws InterruptedException {
// 开启了一个线程池：线程个数是10个
  ExecutorService threadPool = Executors.newFixedThreadPool(10);
  //使用循环来模拟许多用户请求的场景
  for (int request = 1; request <= 100; request++) {
    threadPool.execute(() -> {
      System.out.println("文档处理开始！");
      try {
        // 将Word转换为PDF格式：处理时长很长的耗时过程
        Thread.sleep(1000L * 30);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("文档处理结束！");
    });
  }
  Thread.sleep(1000L * 1000);
}

// 老的处理方式
@Test
public void oldHandle() throws InterruptedException {
  // 使用循环来模拟许多用户请求的场景
  for (int request = 1; request <= 100; request++) {
    new Thread(() -> {
      System.out.println("文档处理开始！");
      try {
        // 将Word转换为PDF格式：处理时长很长的耗时过程
        Thread.sleep(1000L * 30);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("文档处理结束！");
    }).start();
  }
  Thread.sleep(1000L * 1000);
}
```

# Lombok

* 注解的两种解析方式

  * 运行时解析
  * 编译时解析

  ![image-20191210225936690](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191210225936690.png)

* ![image-20191210230029632](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191210230029632.png)

```java
// @Cleanup注解、资源关闭
@Cleanup FileInputStream fileInputStream = new FileInputStream(in);
@Cleanup FileOutputStream fileOutputStream = new FileOutputStream(out);
int r;
while ((r = fileInputStream.read()) != -1) {
  fileOutputStream.write(r);
}
```

```java
/**
 * @AllArgsConstructor
 * @NoArgsConstructor
 * @RequiredArgsConstructor
 */
@RequiredArgsConstructor
public class ConstructorTest {
    private final String field1;
    @NonNull
    private String field2;
    private String field3;
}
```

```java
/**
 * @Data注解
 * 大而全的注解：包含@Getter，@Setter，@ToString，@EqualsAndHashCode
 */
@Data
public class DataTest {
    private String field1;
    private String field2;
}
```

```java
/**
 * @EqualsAndHashCode注解
 * 生成Equals方法和HashCode方法
 */
@EqualsAndHashCode(exclude = {"field1"})
public class EqualsAndHashCodeTest {
    private String field1;
    private String field2;
}
```

```java
/**
 * @Getter注解
 * 为属性生成get方法
 */
public class GetterTest {

    @Getter(lazy = true)
    private final String field1 = "zhangxiaoxi";

    @Getter(
            value = AccessLevel.PRIVATE,
            onMethod_={@NotNull}
    )
    private String field2;
}
/**
 * @Setter注解
 * 为属性生成set方法
 */
public class SetterTest {

    @Setter
    private String field1;
    @Setter(
            value = AccessLevel.PRIVATE,
            onParam_={@NotNull}
    )
    private String field2;
}
```

```java
/**
 * @NonNull注解
 * 生成非空检查
 */
public class NonNullTest {
    public NonNullTest(@NonNull String field) {
        System.out.println(field);
    }
}
```

```java
/**
 * @ToString注解
 * 生成toString方法
 */
@ToString(
        includeFieldNames = false,
//        exclude = {"field1"},
//        of = {"field1"},
        doNotUseGetters = false
)
public class ToStringTest {
    @Setter
    private String field1;
    @Setter
    private String field2;
    public String getField2() {
        System.out.println("调用get方法！");
        return this.field2;
    }
}
```

```java
/**
 * @val注解
 * 弱语言变量
 */
public ValTest() {
  val field = "zhangxiaoxi";
  val list = new ArrayList<String>();
  list.add("zhangxiaoxi");
}
```

# JSR

* 空值校验类：@Null、@NotNull、@NotEmpty、@NotBlank等
* 范围校验类：@Min、@Size、@Digits、@Future、@Negative等
* 其他校验类：@Email、@URL、@AssertTrue、@Pattern等

```java
// 待验证对象实体类
// 用户信息类
@Data
public class UserInfo {
    // 登录场景
    public interface LoginGroup {}
    // 注册场景
    public interface RegisterGroup {}
    // 组排序场景，先验证LoginGroup再验证RegisterGroup
    @GroupSequence({LoginGroup.class,RegisterGroup.class,Default.class})
    public interface Group {}

    // 用户ID，分组：当进行登录时会验证用户ID
    @NotNull(message = "用户ID不能为空", groups = LoginGroup.class)
    private String userId;

    // 用户名
    // NotEmpty 不会自动去掉前后空格
    @NotEmpty(message = "用户名称不能为空")
    private String userName;

    // 用户密码
    // NotBlank 自动去掉字符串前后空格后验证是否为空
    @NotBlank(message = "用户密码不能为空")
    @Length(min = 6, max = 20,message = "密码长度不能少于6位，多于20位")
    private String passWord;

    // 邮箱，分组：当进行注册时会验证email
    @NotNull(message = "邮箱不能为空",groups = RegisterGroup.class)
    @Email(message = "邮箱必须为有效邮箱")
    private String email;

    // 手机号
    @Phone(message = "手机号不是158后头随便")
    private String phone;

    // 年龄
    @Min(value = 18, message = "年龄不能小于18岁")
    @Max(value = 60, message = "年龄不能大于60岁")
    private Integer age;

    // 生日
    @Past(message = "生日不能为未来或当前时间点")
    private Date birthday;

    // 好友列表
  	// @Valid级联验证，也会验证List中UserInfo对象是否符合验证需求
    @Size(min = 1, message = "不能少于1个好友")
    private List<@Valid UserInfo> friends;
}
```

```java
// 验证测试类
public class ValidationTest {

    // 验证器对象
    private Validator validator;
    // 待验证对象
    private UserInfo userInfo;
    // 验证结果集合
    private Set<ConstraintViolation<UserInfo>> set;
    // 验证结果集合
    private Set<ConstraintViolation<UserInfoService>> otherSet;

    // 初始化操作
    @Before
    public void init() {
        // 初始化验证器
        validator = Validation.buildDefaultValidatorFactory().getValidator();
        // 初始化待验证对象 - 用户信息
        userInfo = new UserInfo();
//        userInfo.setUserId("zhangxiaoxi");
        userInfo.setUserName("张小喜");
        userInfo.setPassWord("zhangxiaoxi");
//        userInfo.setEmail("zhangxiaoxi@sina.cn");
        userInfo.setAge(30);
        Calendar calendar = Calendar.getInstance();
        calendar.set(2012, 1, 1);
        userInfo.setBirthday(calendar.getTime());

        userInfo.setPhone("15800000000");

        UserInfo friend = new UserInfo();
//        friend.setUserId("wangxiaoxi");
        friend.setUserName("王小喜");
        friend.setPassWord("wangxiaoxi");
//        friend.setEmail("wangxiaoxi@sina.cn");
        friend.setPhone("15811111111");

        userInfo.setFriends(new ArrayList(){{add(friend);}});
    }

    //结果打印
    @After
    public void print() {
        set.forEach(item -> {
            // 输出验证错误信息
            System.out.println(item.getMessage());
        });
    }

    @Test
    public void nullValidation() {
        // 使用验证器对对象进行验证
        set = validator.validate(userInfo);
    }

    // 级联验证测试方法
    @Test
    public void graphValidation() {
        set = validator.validate(userInfo);
    }

    // 分组验证测试方法
    @Test
    public void groupValidation() {
        set = validator.validate(userInfo,
                UserInfo.RegisterGroup.class,
                UserInfo.LoginGroup.class);
    }

    // 组序列
    @Test
    public void groupSequenceValidation() {
        set = validator.validate(userInfo,UserInfo.Group.class);
    }
}
```

## 高级验证

```java
// 用户信息服务类
public class UserInfoService {
    // UserInfo 作为输入参数
    public void setUserInfo(@Valid UserInfo userInfo) {}
    // UserInfo 作为输出参数
    public @Valid UserInfo getUserInfo() {
        return new UserInfo();
    }
    // 默认构造函数
    public UserInfoService() {}
    // 接收UserInfo作为参数的构造函数
    public UserInfoService(@Valid UserInfo userInfo) {}
}
```

```java
// 对方法输入参数进行约束注解校验
@Test
public void paramValidation() throws NoSuchMethodException {
  // 获取校验执行器
  ExecutableValidator executableValidator = validator.forExecutables();
  // 待验证对象
  UserInfoService service = new UserInfoService();
  // 待验证方法
  Method method = service.getClass().getMethod("setUserInfo", UserInfo.class);
  // 方法输入参数
  Object[] paramObjects = new Object[]{new UserInfo()};
  // 对方法的输入参数进行校验
  otherSet = executableValidator.validateParameters(
    service,
    method,
    paramObjects);
}
```

```java
// 对方法返回值进行约束校验
@Test
public void returnValueValidation()
  throws NoSuchMethodException,
InvocationTargetException, IllegalAccessException {
  // 获取校验执行器
  ExecutableValidator executableValidator = validator.forExecutables();

  // 构造要验证的方法对象
  UserInfoService service = new UserInfoService();
  Method method = service.getClass().getMethod("getUserInfo");

  // 调用方法得到返回值
  Object returnValue = method.invoke(service);

  // 校验方法返回值是否符合约束
  otherSet = executableValidator.validateReturnValue(
    service,
    method,
    returnValue);
}
```

```java
// 对构造函数输入参数进行校验
@Test
public void constructorValidation()throws NoSuchMethodException {
  // 获取验证执行器
  ExecutableValidator executableValidator = validator.forExecutables();
  // 获取构造函数
  Constructor constructor = UserInfoService.class.getConstructor(UserInfo.class);
  Object[] paramObjects = new Object[]{new UserInfo()};
  // 校验构造函数
  otherSet = executableValidator.validateConstructorParameters(constructor, paramObjects);
}
```

## 实战

* 完成验证的步骤

  1. 约束注解的定义
  2. 约束验证规则（约束验证器）
  3. 约束注解的声明
  4. 约束验证流程

* 实战案例：自定义手机号约束注解

  * 定义@interface Phone注解
  * 实现约束验证器PhoneValidator.java
  * 声明@Phone约束验证
  * 执行手机号约束验证流程

  ```java
  // 自定义手机号约束注解
  @Documented
  // 注解的作用目标
  @Target({ElementType.FIELD})
  // 注解的保留策略
  @Retention(RetentionPolicy.RUNTIME)
  // 不同之处：于约束注解关联的验证器
  @Constraint(validatedBy = PhoneValidator.class)
  public @interface Phone {
      // 约束注解验证时的输出信息
      String message() default "手机号校验错误";
      // 约束注解在验证时所属的组别
      Class<?>[] groups() default {};
      // 约束注解的有效负载
      Class<? extends Payload>[] payload() default {};
  }
  ```

  ```java
  // 自定义手机号约束注解关联验证器
  public class PhoneValidator implements ConstraintValidator<Phone, String> {
      // 自定义校验逻辑方法
      @Override
      public boolean isValid(String value, ConstraintValidatorContext context) {
          // 手机号验证规则：158后头随便
          String check = "158\\d{8}";
          Pattern regex = Pattern.compile(check);
          // 空值处理
          String phone = Optional.ofNullable(value).orElse("");
          Matcher matcher = regex.matcher(phone);
          return matcher.matches();
      }
  }
  ```

  ```java
  // 手机号
  @Phone(message = "手机号不是158后头随便")
  private String phone;
  ```

  ```xml
  <!-- Validation 相关依赖 -->
  <dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
  </dependency>
  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.16.Final</version>
  </dependency>
  <dependency>
    <groupId>javax.el</groupId>
    <artifactId>javax.el-api</artifactId>
    <version>3.0.0</version>
  </dependency>
  <dependency>
    <groupId>org.glassfish.web</groupId>
    <artifactId>javax.el</artifactId>
    <version>2.2.6</version>
  </dependency>
  ```

  