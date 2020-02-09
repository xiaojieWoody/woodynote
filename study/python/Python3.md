* 代码风格

  ```python
  # 变量和函数
  全小写+下划线  # seconds_per_day
  # 类名
  驼峰写法  # MyClass
  # 异常
  驼峰写法，并且以「Error」结尾  # FileParseError
  # 常量
  全大写+下划线  # MAX_RETRY_TIMES
  # 模块
  小写 + 下划线  # open_api
  # 包名
  小写字母命名  # requests
  # 每级缩进应使用 4 个空格
  # 定义函数时，将参数换行后缩进两次，以和代码块相区分
  # 定义函数和类时，多个函数或类之间使用两个空行进行分隔：
  # 类中的定义方法时，多个方法间用一个空行进行分隔
  # import 按下列类型和顺序使用
  标准库导入
  第三方库导入
  本地库导入
  ```

* 数据类型

  * 整数型、浮点型、字符串、布尔型、None型

* 字符串、列表

* input()

  * 通过命令行与程序交互

  * 码执行到此处时输出显示一段提示文本，然后等待输入。在输入内容并按下回车键后，程序将读取输入内容并继续向下执行。读取到的输入内容可赋值给变量，供后续使用

    ```python
    >>> age = input(‘请输入你的年龄：’)
    ```

* int()

  * 将字符串、浮点型转换整数型

    ```python
    int(字符串或浮点数)
    ```

* print()

  * 将指定的内容输出到命令行中

  * 要输出的内容放在括号中，多项内容时用逗号分隔，显示时每项以空格分隔

    ```python
    age = int(input('请输入你的年龄：')) 
    print('你的年龄是', age)
    
    if age < 18:
        print('好好学习，天天向上')
    elif 7 <= age <=17:
        print('少年')
    else:
    	print('革命尚未成功，同志仍需努力')
    ```

    ```python
    while 条件:
        代码块
    ```

    ```python
    if 条件1 and 条件2 and 条件N:
    	代码块
      
    if 条件1 or 条件2 or 条件N:
    	代码块
      
    if not 条件:
    	代码块
    ```

    ```python
    for 项 in 序列:
        代码块
        
    fruit_list = ['apple', 'banana', 'cherry', 'durian']
    for fruit in fruit_list:
        print(fruit)
    ```

* 函数

  ```python
  def 函数名(参数1, 参数2, ...):
      代码块
  ```

  ```python
  def stage_of_life(age):
      if age <= 6:
          return '童年'
      elif 7 <= age <=17:
          return '少年'
      elif 18 <= age <= 40:
          return '青年'
      elif 41 <= age <= 65: 
  		return '中年'
      else:
          return '老年'
  
  stage = stage_of_life(18)
  print(stage)
  ```

  * 用参数的形式将数据传递给函数，用赋值语句来接收返回值

* 异常处理

  ```python
  try:
      代码块1
  except:
      代码块2
  ```

  ```python
  try:
      代码块1
  except (异常X, 异常Y, 异常Z) as e:
      代码块2
  finally:
      代码块3
  ```

  ```python
  try:
      代码块1
  except 异常X as e:
      代码块2
  except 异常Y as e:
      代码块3
  except 异常Z as e:
      代码块4
  finally:
      代码块5
  ```
  * 主动抛出异常

    ```python
    raise ValueError("输入值不符合要求")
    ```

* 查看数据类型

  ```python
  type(变量或值) 
  # int float str bool NoneType list
  ```

* 类的定义

  ```python
  class 类名:
      代码块
  ```

  ```python
  class A:
      pass
  # pass 是占位符，表示什么都不做或什么都没有  
  ```

* 实例化对象

  ```python
  变量 = 类名()
  ```

  ```python
  class 类名:
      def __init__(self, 数据1, 数据2, ...):
          self.数据1 = 数据1
          self.数据2 = 数据2
          ...
  # 第一个是 self，它是类中函数默认的第一个参数，表示对象自身。我们可以将数据赋值给 self.数据名，这样数据就保存在对象中了
  # first 是数据对的第一个值
  # second 是数据对的第二个值
  pair = Pair(10, 20)
  pair.first
  pair.second
  ```

  ```python
  class Pair:
      def __init__(self, first, second):
          self.first = first
          self.second = second
  		# 值交换        
      def swap(self):
          self.first, self.second = self.second, self.first
  ```

* 模块

  ```python
  import 模块名
  
  模块名.变量
  模块名.函数
  模块名.类
  ```

  ```python
  tree_farmer  # 目录名
  	|___tree.py # 文件名
  	|___farmer.py # 文件名
    
  # tree.py
  import random
  
  fruit_name = ''
  # 将列表项重复若干遍。比如执行 ['X'] * 3 将得到 ['X', 'X', 'X']
  def harvest():
      return [fruit_name] * random.randint(1, 9)
    
  # farmer.py
  import tree
  
  print('种下一棵果树。')
  tree.fruit_name = 'apple'
  
  print('等啊等，树长大了，可以收获了！')
  fruits = tree.harvest()
  print(fruits)
  ```

  ```python
  import sys
  
  模块文件名 = sys.argv[0]
  参数1     = sys.argv[1]
  参数N     = sys.argv[N]
  ```

  ```python
  # 改进 farmer.py
  import sys  # 新增
  import tree
  
  print('种下一棵果树。')
  tree.fruit_name = sys.argv[1]   # 将 'apple' 改为 参数 sys.argv[1]
  
  print('等啊等，树长大了，可以收获了！')
  fruits = tree.harvest()
  print(fruits)
  ```

  ```python
  ➜ ~ python3 farmer.py banana
  种下一棵果树。
  等啊等，树长大了，可以收获了！
  [‘banana’, ‘banana’, ‘banana’]
  ```

* 包

  * 每个层级的包下都需要有一个 `__init__.py` 模块。这是因为只有当目录中存在 `__init__.py` 时，Python 才会把这个目录当作包

  ```python
  import 包.子包.模块
  ```

  ```python
  import random
  
  class RandomChar:
      def __init__(self):
          self.all_uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
          self.all_lowercase = 'abcdefghijklmnopqrstuvwxyz'
          self.all_digits = '0123456789'
          self.all_specials = '~!@#$%^&*'
  
      def pick_random_item(self, sequence):
          random_int = random.randint(0, len(sequence) - 1)
          return sequence[random_int]
  
      def uppercase(self):
          return self.pick_random_item(self.all_uppercase)
  
      def lowercase(self):
          return self.pick_random_item(self.all_lowercase)
  
      def digit(self):
          return self.pick_random_item(self.all_digits)
  
      def special(self):
          return self.pick_random_item(self.all_specials)
  
      def anyone(self):
          return self.pick_random_item(self.all_uppercase + self.all_lowercase + self.all_digits + self.all_specials)
  ```

  ```python
  import randomchar
  
  def generate_password(length):
      if length < 4:
          raise ValueError('密码至少为 4 位')
  
      random_char = randomchar.RandomChar()
  
      password  = random_char.uppercase()
      password += random_char.lowercase()
      password += random_char.digit()
      password += random_char.special()
  
      count = 5
      while count <= length:
          password += random_char.anyone()
          count += 1
  
      return password
  
  
  password_length = input('请输入密码长度（8～20）：')
  password_length = int(password_length)
  
  if password_length < 8 or password_length > 20:
      raise ValueError('密码长度不符')
  
  password = generate_password(password_length)
  print(password)
  ```

* 内置数据类型

  * 列表

    ```python
    # 通过索引获取元素
    元素 = 列表[索引]
    # 通过元素获取索引
    索引 = 列表.index(元素)
    # 查看元素是否存在于列表中
    布尔值 = 元素 in 列表
    # 统计元素在列表中的个数
    个数 = 列表.count(元素)
    # 向列表末尾追加元素
    列表.append(元素)
    # 向列表的任意位置插入元素
    列表.insert(索引, 元素)
    # 列表末尾追加另一个列表的所有元素
    列表.extend(另一列表)
    # 按索引删除元素
    元素 = 列表.pop(索引)
    # 按索引删除元素（del 方法)
    del 列表[索引]
    # 直接删除元素
    列表.remove(元素)
    # 清空所有元素
    列表.clear()
    # 通过赋值修改列表元素
    列表[索引] = 新元素
    # 反转整个列表
    列表.reverse()
    # 列表元素排序
    列表.sort()
    # 也可以通过指定 sort 方法的 reverse 参数来倒序排列
    列表.sort(reverse=True)
    ```

  * 元组

    * 元组创建完成后，只具备读的功能-不可变（性），而列表是可变的
    * 一般情况下元组的性能在略高于列表

    ```python
    元组 = ()
    元组 = (元素1, 元素2, ..., 元素N)
    # 创建只包含一个元素的元组
    元组 = (元素,)
    # 这是因为，如果括号中只有一个元素，那么 Python 会将这个括号当作优先级符号进行处理（像数学中的那样），而不是当作元组
    # 通过索引获取元素
    元素 = 元组[索引]
    # 通过元素获取索引
    索引 = 元组.index(元素)
    # 查看元素是否存在于元组中
    布尔值 = 元素 in 元组
    # 统计元素在元组中出现的个数
    个数 = 元组.count(元素)
    ```

  * 字符串

    ```python
    字符串 = ''
    字符串 = '若干字符'
    # 通过索引获取字符
    字符 = 字符串[索引]
    # 通过子串获取索引
    索引 = 字符串.index(字符)
    # 当字符或子串不存在时，index 方法将抛出 ValueError 错误
    #  find 方法未找到子串时返回数字 -1，而不抛异常
    # 查看字符是否存在于字符串中
    布尔值 = 字符 in 字符串
    # 统计字符在字符串中的个数
    个数 = 字符串.count(字符)
    # 判断字符串是否以某个子串开头，返回布尔值
    string.startswith(‘ha’)
    # 判断字符串是否以某个子串结尾，返回布尔值
    string.endswith(‘y’)
    # 将字符串的子串用一个另一个字符串替换，返回一个新的字符串
    string.replace(‘y’, ‘iness’)
    # 去除字符串前后的空白符号，如空格、换行符、制表符，返回一个新的字符串
    string.strip()
    # 将字符串用某个子串分隔开，分隔后的各个部分放入列表中，并返回这个列表
    string.split(' ') 
    # 将一个序列中的元素用某个字符（串）拼接，组成一个大的字符串，并返回这个字符串
    words=['I','am','happy']
    ' '.join(words)
    # 将字符串转化为大写字母形式，返回一个新的字符串
    string.upper()
    # 将字符串转化为小写字母形式，返回一个新的字符串
    string.lower()
    # 字符转义
    string='I\'m woody'
    # 常见的换行（\n），制表符（\t），回车（\r），单引号（\'），双引号（\"），反斜杠（\\）
    len('\n') #只占一个字节
    # 字符串中表示 \n 这两个字符，而不是让它表示换行
    print('\\n')
    # 原始字符串就是在字符串的起始引号前加上一个 r 或 R 字母，这样字符串中的内容将不会被转义，将按照原样输出
    r'字符串内容'
    print(r'\n')
    # 多行字符串
    # 字符串的每行末尾使用 \ 续行，但是输出内容还是被当作一行
    string='第一行\
    第二行\
    第三行'
    # 如果想要输出内容为多行，需要在字符串中显式地使用 \n 进行换行 
    string='第一行\n\
    第二行\n\
    第三行'
    # 使用三个单引号或三个双引号来表示字符串，这样字符串中的内容就可以多行书写，并且被多行输出
    # 字符串可被多行书写，且被多行输出，其中不需要显式地指明 \n 换行
    string = '''第一行
    第二行
    第三行'''
    print(string)
    ```

  * 列表、元组、字符串的通用操作

    ```python
    # 使用 len() 函数获取序列长度
    # 获取序列中的一个子序列，以 [起始索引:结束索引] 表示，是一个左开右闭区间，区间内的所有元素作为子序列被返回
    numbers=(1,2,3,4)
    numbers[0:2]
    (1, 2)
    # 使用 + 符号来拼接两个序列
    # 使用 * 符号来重复序列中的元素
    letters=(1,2,3)
    letters*3
    (1, 2, 3, 1, 2, 3, 1, 2, 3)
    ```

  * 总结
    * 列表、元组、字符串都是有序序列，都可以使用索引
    * 列表和元组中可以存放任意数据类型的元素，而字符串中只能存放字符
    * 列表是可变的，而元组和字符串是不可变的

* 字典

  * 键：是不可变的类型，如元组、字符串、数字
  * 值：可以是任意类型
  * 无序

  ```python
  字典 = {}
  字典 = {键1:值1, 键2:值2, ..., 键N:值N}
  # 向字典中增加键值对
  字典[键] = 值
  # 通过键获取值
  值 = 字典[键]            # 若键不存在则将抛出 KeyError 异常
  值 = 字典.get(键)        # 若键不存在，则直接返回 None
  值 = 字典.get(键, 默认值)
  # 判断字典中是否包含某个键
  布尔值 = 键 in 字典
  # 获取所有键
  键的列表 = 字典.keys()
  # 获取到的所有键是以迭代器的形式存在，用 list() 函数将迭代器转换为列表
  list(codes.keys())
  # 获取所有值
  值的列表 = 字典.values()
  # 获取到的所有值是以迭代器的形式存在，用 list() 函数将迭代器转换为列表
  list(codes.values())
  # 获取所有键值对的列表
  值的列表 = 字典.items()
  # 获取到的所有键值对是以迭代器的形式存在，用 list() 函数将迭代器转换为列表
  list(codes.items())
  # 通过键删除键值对
  值 = 字典.pop(键)
  # 如果 pop 一个不存在的键，则会抛出 KeyError 异常
  值 = 字典.pop(键, 默认值)
  # 默认值仅在键不存在时生效，此时方法将直接返回这个默认值，且跳过删除操作
  del 字典[键]
  # 随机删除一个键值对
  键值二元组 = 字典.popitem()     # 二元组的第一个元素是键，第二个元素是值
  # 清空所有键值对
  键值二元组 = 字典.clear()
  # 修改键对应的值，如果键不存在，则创建键值对
  字典[键] = 值
  # 用字典批量更新键值对
  字典.update(另一字典)
  ```

* 集合

  * 无序，不重复

  ```python
  # 创建包含元素的集合
  集合 = {元素1, 元素2, 元素N}
  # 创建空集合
  集合 = set()
  # 向集合中添加一个元素
  集合.add(元素)
  # 从另一集合中批量添加元素
  集合.update(另一集合)
  # 查看元素是否存在于集合中
  布尔值 = 元素 in 集合
  # 随机删除一个元素，并返回这个元素
  元素 = 集合.pop()
  # 删除一个指定的元素,如果要删除的元素不存在，则抛出 KeyError 异常：
  集合.remove(元素)
  # 删除一个指定的元素，且不抛出 KeyError 异常
  集合.discard(元素)
  # 清空所有元素
  集合.clear()
  # 求交集,可以通过 intersection 方法求多个集合的交集;也可以直接使用与运算符 & 来代替，完全等效
  交集 = 集合1.intersection(集合2, 集合3, 集合N)
  交集 = 集合1 & 集合2 & 集合N
  # 求并集,也可以直接使用或运算符 | 来代替，完全等效
  交集 = 集合1.union(集合2, 集合3, 集合N)
  交集 = 集合1 | 集合2 | 集合N
  # 求差集,也可以直接使用运算符 - 来代替，完全等效
  交集 = 集合1.difference(集合2, 集合3, 集合N)
  交集 = 集合1 - 集合2 - 集合N
  # 判断是否为子集,判断 集合1 是否为 集合2 的子集
  布尔值 = 集合1.issubset(集合2)
  # 判断是否为超集,判断 集合1 是否为 集合2 的子集
  布尔值 = 集合1.issuperset(集合2)
  ```

* 内置函数

  * 数据类型相关

    ```python
    dict()
    # 将参数转换为字典类型,dict(a=1, b=2, c=3) -> {'a': 1, 'b': 2, 'c': 3}
    float()
    # 将字符串或数字转换为浮点型,float('0.22') -> 0.22
    int()
    # 将字符串或数字转换为整数型,int(1.23) -> 1
    list()
    # 将元组、字符串等可迭代对象转换为列表,list('abc') -> ['a', 'b', 'c']
    tuple()
    # 将列表、字符串等可迭代对象转换为元组,tuple([1, 2, 3])	 -> (1, 2, 3)
    set()
    # 1.创建空集合；2.将可迭代对象转换为列表集合,set('abc') -> {'b', 'a', 'c'}
    str()
    # 将参数转换为字符串,str(3.14) -> '3.14'
    bytes()
    # 将参数转换为字节序列,bytes(4) -> b'\x00\x00\x00\x00
    ```

  * 数值计算相关

    ```python
    max(xx)
    min(xx)
    sum(xx)
    abs(xx)
    pow(xx,xx)
    bin(xx) # 转换为二进制字符串
    hex(xx) # 转换为十六进制字符串
    round(xx,xx) # 浮点数四舍五入，round(4.5678, 2) （第二个参数为小数精度）
    ```

  * bool值判断相关

    ```python
    bool()
    all() # 如果可迭代对象中的所有值，在逐一应用 bool(值) 后结果都为 True，则返回 True，否则返回 False
    any() # 如果可迭代对象中的任意一个或多个值，在应用 bool(值) 后结果为 True，则返回 True，否则返回 False
    ```

  * IO 相关

    ```python
    input() # 从标准输入中读取字符串
    print() # 将内容写入标准输出中
    open()  # 打开一个文件。之后便可以对文件做读写操作
    ```

  * 元数据相关

    ```python
    type() # 获取对象的类型
    isinstance() # 判断对象是否是某个类（或其子类）的对象
    dir() # 获取类或对象中的所有方法和属性；无参数时获取当前作用域下的所有名字
    id() # 返回一个对象的唯一标识。在我们所使用的 CPython 中这个唯一标识实际为该对象在内存中的地址
    ```

  * help()

    * 解释器交互模式下获取某个函数、类的帮助信息，非常实用
    * 例如，**help(any)** # 只需使用函数名字

  * sorted

    * 对可迭代对象中的数据进行排序，返回一个新的列表

  * range()

    * 获取一个整数序列。可指定起始数值，结束数值，增长步长

    * 在 `for` 循环中想要指定循环次数时非常有用。

      ```python
      # 指定起始数值和结束数值，获取一个连续的整数序列
      # 生成的数值范围为左开右闭区间，即不包括所指定的结束数值
      for i in range(2, 6):
          print(i)
      # 只指定结束数值，此时起始数值默认为 0
      for i in range(4):
        	print(i)
      ```

* 迭代

  ```python
  迭代器 = iter(容器)
  值 = next(迭代器)
  ```
  * 什么是可迭代的

    * 从表面来看，所有可用于 `for` 循环的对象是可迭代的，如列表、元组、字符串、集合、字典等容器

    * 从深层来看，**定义了 `__iter__()` 方法的类对象就是可迭代的。当这个类对象被 `iter()` 函数使用时，将返回一个迭代器对象**。如果对象具有`__iter__()` 方法，则可以说它支持迭代协议

    * 判断一个已有的对象是否是可迭代的，有两个方法：

      * 通过内置函数 `dir()` 获取这个对象所有方法，检查是否有 `'__iter__'`

        ```python
        '__iter__' in dir(list)
        ```

      * 使用内置函数 `isinstance()` 判断其是否为 `Iterable` 的对象

        ```python
        from collections.abc import Iterable
        isinstance(对象, Iterable)
        ```

  * 自定义迭代器

    * 只要在类中定义 `__next__()` 和 `__iter__()` 方法即可

      ```python
      class MyIterator:
        def __next__(self):
          代码块
          
        def __iter__(self):
          return self
      ```

      ```python
      # 迭代器从 2^0 开始返回 2 的指数幂，至 2^10 终止
      class PowerOfTwo:
          def __init__(self):
              self.exponent = 0  					# 将每次的指数记录下来
      
          def __next__(self):
              if self.exponent > 10:
                  raise StopIteration
              else:
                  result = 2 ** self.exponent		# 以 2 为底数求指数幂
                  self.exponent += 1
                  return result
      
          def __iter__(self):
              return self
      ```

  * 生成器

    * 生成器是一个函数，这个函数的特殊之处在于它的 `return` 语句被 `yield` 语句替代

    * 用于生成 2 的指数幂的迭代器，可以通过生成器来实现

      ```python
      def power_of_two():
      	for exponent in range(11):	# range(11) 表示左开右闭区间 [0, 11)，不包含 11
      		yield 2 ** exponent		# 以 2 为底数求指数幂
      ```

    * 生成器使用方法：

      ```python
      p = power_of_two()				# 以函数调用的方式创建生成器对象
      next(p)
      ```

    * **生成器的关键在于 `yield` 语句**。`yield` 语句的作用和 `return` 语句有几分相似，都可以将结果返回。不同在于，生成器函数执行至 `yield` 语句，返回结果的同时记录下函数内的状态，下次执行这个生成器函数，将从上次退出的位置（`yield` 的下一句代码）继续执行。当生成器函数中的所有代码被执行完毕时，自动抛出 `StopIteration` 异常
    * 生成器的用法和迭代器相似，都使用 `next()` 来进行迭代。这是因为生成器其实就是创建迭代器的便捷方法，生产器会在背后自动定义 `__iter__()` 和 `__next__()` 方法

  * 生成器表达式

    * 可创建生成器

      ```python
      生成器 = (针对项的操作 for 项 in 可迭代对象)
      ```

      ```python
      letters = (item for item in ‘abc’)
      next(letters)
      letters = (i.upper() * 2 for i in ‘abc’)
      next(letters)
      ```

* 列表生成式

  ```python
  nums = []
  exponent = 1
  while exponent <= 10:
    nums.append(2 ** exponent)
     exponent += 1
  ```

  ```python
  nums = []
  for i in range(1, 11):		# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    nums.append(2 ** i)
  ```

  ```python
  nums = [2 ** i for i in range(1, 11)]
  ```

  ```python
  # 列表生成式语法
  [对项的操作 for 项 in 可迭代对象]
  # 首先，这句代码的阅读顺序是：for 项 in 可迭代对象 -> 对项的操作。其次，外围的方括号（[]）表明这是列表生成式，最终的结果是一个列表
  ```

  ```python
  [char.upper() for char in ['a', 'b', 'c', 'd', 'e']]
  # ['A', 'B', 'C', 'D']
  [(char.upper(),char) for char in ['a','b','c','d']]
  # [('A', 'a'), ('B', 'b'), ('C', 'c'), ('D', 'd')]  
  ```

  ```python
  # 列表生成式中使用 if，阅读顺序是：for 项 in 可迭代对象 -> if 对项的判断 -> 对项的操作
  [对项的操作 for 项 in 可迭代对象 if 对项的判断]
  ```

  ```python
  # 生成出 20～210 间的整数，只保留其中的奇数次方的值
  [2 ** i for i in range(1, 11) if i % 2 == 1 ]
  ```

  ```python
  # 列表生成式中嵌套 for 
  [对项1和(或)项2的操作 for 项1 in 可迭代对象1 for 项2 in 可迭代对象2]
  nums = [1, 2, 3]
  chars = ['a', 'b', 'c']
  [c * n for n in nums for c in chars]
  ['a', 'b', 'c', 'aa', 'bb', 'cc', 'aaa', 'bbb', 'ccc']
  ```

  ```python
  # [对项1和(或)项2的操作 for 项1 in 可迭代对象1 for 项2 in 可迭代对象2] 中的 可迭代对象2 可以是 项1 本身
  [对项和(或)子项的操作 for 项 in 可迭代对象 for 子项 in 项]
  strings = ['aa', 'bb', 'cc']
  [char for string in strings for char in string]
  ['a', 'a', 'b', 'b', 'c', 'c']
  ```

* 字典生成式

  ```python
  {键: 值 for 项 in 可迭代对象}
  {char: char.upper() for char in 'abcde'}
  # 也可以使用 if 和嵌套 for，使用方法参照列表生成式
  ```

* 集合生成式

  ```python
  {对项的操作 for 项 in 可迭代对象}
  {char.lower() for char in 'ABCDABCD'}
  # 也可以使用 if 和嵌套 for。
  ```

* 生成器表达式

  ```python
  # Python 中并没有「元组生成式」
  (对项的操作 for 项 in 可迭代对象)		# 生成器表达式
  ```

* 函数

  * 参数默认值，调用时可以省略不传该参数

    ```python
    def 函数(参数1, 参数2=默认值):
        pass
    ```

  * 关键字参数

    ```python
    # 函数调用时，以 参数名=值 的形式来向指定的参数传入值，可打乱参数传递次序
    overspeed_rate(min=80, max=100, current=100)
    # 关键字参数需要出现在位置参数之后，否则将抛出 SyntaxError 异常
    ```

    ```python
    # 在定义函数时，如果参数列表中某个参数使用 **参数名 形式，那么这个参数可以接受一切关键字参数
    def echo(string, **keywords):
        print(string)
        for kw in keywords:
            print(kw, ":", keywords[kw])
    
    echo(‘hello’, today=‘2019-09-04’, content=‘function’, section=3.6)
    hello
    today : 2019-09-04
    content : function
    section : 3.6
      
    def foo(**keywords):
      print(keywords)
    foo(a=1, b=2, c=3)
    	{‘a’: 1, ‘b’: 2, ‘c’: 3}
    ```

  * 任意参数列表

    ```python
    # 参数列表中使用 *参数名，就可以接受任意数量的非关键字参数，也就是可变参数
    def multiply(*nums):
        result = 1
        for n in nums:
            result *= n
        return result
    ```

  * 多返回值

    ```python
    # 只需在 return 关键字后跟多个值（依次用逗号分隔）
    def date():
        import datetime
        d = datetime.date.today()
        return d.year, d.month, d.day
    # 接收函数返回值时，用对应返回值数量的变量来分别接收它们
    year, month, day = date()
    # 其实多返回值时，Python 将这些返回值包装成了元组，然后将元组返回
    date()			# (2019, 9, 4)
    # 接收返回值时，year, month, day = date()，这样赋值写法，会将元组解包，分别将元素赋予单独的变量中
    ```

* 类属性和类方法

  ```python
  对象 = 类()
  
  对象.属性
  对象.方法()
  ```

  ```python
  类.属性
  类.方法()
  # 类所创建出来的对象也能使用类属性
  ```

  ```python
  class 类：
  	属性1 = X
  	属性2 = Y
    
  	def 某方法():
  		pass
  ```

  ```python
  # 定义类方法时，需要在方法的前面加上装饰器 @classmethod
  class 类:
  	@classmethod
  	def 类方法(cls):
  		pass
  # 与对象方法不同，类方法的第一个参数通常命名为 cls，表示当前这个类本身。可通过该参数来引用类属性，或类中其它类方法      
  ```

  ```python
  # 静态方法
  类方法的第一个参数是类自身（cls），而静态方法没有这样的参数
  如果方法需要和其它类属性或类方法交互，那么可以将其定义成类方法；如果方法无需和其它类属性或类方法交互，那么可以将其定义成静态方法
  class 类:
  	@staticmethod
  	def 静态方法():
  		pass
  ```

  ```python
  # 私有属性、方法
  属性或方法的名称用 __（两个下划线）开头即可，限制属性不被类外部所访问，而是只能在类中使用
  ```

  ```python
  # 特殊方法
  类中以 __ 开头并以 __ 结尾的方法是特殊方法
  # __init__()
  用于对象的初始化。在实例化类的过程中，被自动调用
  # __next__()
  迭代器调用 next() 函数，便能生成下一个值，其实next() 调用了迭代器的 __next__() 方法
  # __len__()
  容器类中实现了 __len__() 方法，调用 len() 函数时将自动调用容器的 __len__() 方法
  # __str__()
  在使用 print() 函数时将自动调用类的 __str__() 方法
  class A:
      def __str__(self):
          return '这是 A 的对象'
  a = A()
  print(a)
  # __getitem__()
  诸如列表、元素、字符串这样的序列，可通过索引的方式来获取其中的元素，这背后便是 __getitem__() 在起作用
  'abc'[2] 即等同于 'abc'.__getitem__(2)
  ```

* 继承

  ```python
  # 定义时，子类名称的后面加上括号并写入父类
  class 父类:
      父类的实现
      
  class 子类(父类)：
  	子类的实现
  ```

  ```python
  class B(A):
  	def __init__(self):
  		super().__init__()              # 在子类初始化的同时初始化父类
  		self.banana = 'banana'
  ```

* 继承链

  ```python
  子类可以继承父类，同样的，父类也可以继承它自己的父类，如此一层一层继承下去
  任何类的根源都是 object 类。如果一个类没有指定所继承的类，那么它默认继承 object
  判断一个类是否是另一个类的子类，可以使用内置函数 issubclass()
  issubclass(C, A)
  ```

* 多继承

  ```python
  定义时，子类名称后面加上括号并写入多个父类
  ```

* 函数式编程

  ```python
  一切对象都可以作为函数的参数，包括另一个函数。接受函数作为参数的函数，称为高阶函数
  def filter_nums(nums, want_it):
      return [n for n in nums if want_it(n)]
  def want_it(num):
      return num % 2 == 0  
  ```

* lambda表达式

  ```python
  lambda 参数1, 参数2, 参数N: 函数实现
  f = lambda x: x ** 2
  在需要将函数作为参数时，才去使用 lambda 表达式，这样就无需在函数调用前去定义另外一个函数了
  def filter_nums(nums, want_it):
      return [n for n in nums if want_it(n)]
  filter_nums([11, 12, 13, 14, 15, 16, 17, 18], lambda x: x % 2 == 0)  
  ```

  ```python
  # 排序函数 sorted()。它有一个参数 key，用来在排序复杂元素时，指定排序所使用的字段，这个参数需要是个函数，同样可以用 lambda 表达式来解决
  codes = [(‘上海’, ‘021’), (‘北京’, ‘010’), (‘成都’, ‘028’), (‘广州’, ‘020’)]
  sorted(codes, key=lambda x: x[1]) # 以区号字典来排序
  ```

* map()

  ```python
  # 用于对可迭代对象中每一个元素逐一作处理
  map(处理函数, 可迭代对象)
  # map() 依次对 可迭代对象 中的每个元素调用 处理函数，最终返回一个包含所有被处理过后的元素的迭代器
  ```

  ```python
  # 将 ['a', 'b', 'cd', 'efg', 'hig', 'klmn', 'opqr'] 全部转换为大写。可以这样来写
  list(map(lambda x: x.upper(), ['a', 'b', 'cd', 'efg', 'hig', 'klmn', 'opqr']))
  [‘A’, ‘B’, ‘CD’, ‘EFG’, ‘HIG’, ‘KLMN’, ‘OPQR’]
  ```

* filter()

  ```python
  # 用于从可迭代对象中筛选元素
  filter(筛选函数, 可迭代对象)
  # filter() 依次对 可迭代对象 中的每个元素调用 筛选函数，如果返回值为 True，则当前这个元素会被保留，否则被排除。最终返回一个包含所有被保留元素的迭代器
  ```

  ```python
  # 从 ['a', 'b', 'cd', 'efg', 'hig', 'klmn', 'opqr'] 筛选出长度为奇数的字符串
  list(filter(lambda x: len(x) % 2 == 1, ['a', 'b', 'cd', 'efg', 'hig', 'klmn', 'opqr']))
  [‘a’, ‘b’, ‘efg’, ‘hig’]
  ```

* 函数中定义函数

  ```python
  def print_twice(word):
      def repeat(times):
          return word * times
      
      print(repeat(2))
  >>> print_twice('go ')    
  ```

* 函数返回函数

  ```python
  def print_words(word):
      def repeat(times):
          return word * times
      
      return repeat  
  ```

* 装饰器

  ```python
  # 装饰器用来增强一个现有函数的功能，并且不改变这个函数的调用方式。这种增强是非侵入式的，也就是说无需直接修改函数内部的代码，而是在函数的外部做文章
  ```

  ```python
  # 每次输出「Hello!」的同时附带上当前的时间
  @time
  def say_hello():
      print('Hello!') 
  ```

  * 自定义装饰器

    ```python
    # @time 的实现
    import datetime    # 日期时间相关库，用于后续获取当前时间
    
    def time(func):
        def wrapper(*args, **kw):
            print('[', datetime.datetime.now(), ']')
            return func(*args, **kw)
        return wrapper
    ```

  * 装饰器原理

    ```python
    @time
    def say_hello():
        print('Hello!')
        
    # 等效于
    
    def say_hello():
        print('Hello!')
    
    say_hello = time(say_hello)
    ```
    * @time（包括所有装饰器）本质上是个以函数作为参数，并返回函数的函数

    * 装饰器之所以能够增强一个函数的功能，其实就是将被装饰函数用新函数替换，虽然还是同一个函数名，但函数内部实现已经变了。而这个新函数的内部在添加了一些功能的后，还会调用之前被装饰的函数。这样就相当于对被装饰的函数做了非侵入的扩展

    * 装饰器本质上是用一个新的函数来替换被装饰的函数，所以函数的元信息会被覆盖

    * 在定义装饰器时使用 `@functools.wraps` 装饰器，可保留被装饰函数的元信息

      ```python
      import datetime
      import functools
      
      def time(func):
          @functools.wraps(func)
          def wrapper(*args, **kw):
              print('[', datetime.datetime.now(), ']')
              return func(*args, **kw)
          return wrapper
      ```

  * 带参数的装饰器

    ```python
    # 带时间格式的装饰器
    @time('%Y/%m/%d %H:%M:%S')
    def say_hello():
        print('Hello!')
    # 原理
    import datetime
    import functools
    
    def time(format):
        def decorator(func):
            @functools.wraps(func)
            def wrapper(*args, **kw):
                print(datetime.datetime.now().strftime(format))
                return func(*args, **kw)
            return wrapper
        return decorator
    ```

* 其他常用语言特性

  ```python
  # 序列（列表、元组、字符串）的索引可以为负值，此时将按逆序从序列中的取元素，-1 表示最后一个元素
  # 切片以索引区间 [起始索引:结束索引] 来表示，左开右闭区间
  chars = [‘a’, ‘b’, ‘c’, ‘d’, ‘e’]
  chars[1:3] # [‘b’, ‘c’]
  # 如果起始索引为 0，则可以省略起始索引
  # 如果结束索引等于序列长度，则可以省略结束索引
  # 也可既省略起始索引也省略结束索引，那么将取整个序列
  # 起始索引和结束索引可以为负值
  # 切片中可以指定步长，用第三个参数表示。步长表示索引的间隔，如 [0:5:2] 表示从索引 0 至 5，每隔 2 个索引取一次值
  chars = [‘a’, ‘b’, ‘c’, ‘d’, ‘e’]
  chars[0:5:2] # ['a', 'c', 'e']
  # 步长也可以为负值，使用如下方式即可翻转一个序列
  chars[::-1]
  ```

  ```python
  # 连续赋值
  a = b = c = 1
  # 拆包，赋值时，将根据位置关系，将 = 右侧的值分别赋值给左侧的变量
  a, b = 1, 2
  # 等效于
  a, b = (1, 2)
  # 它将元祖中的每个元素拆解出来，然后分别赋值给前面的变量。这种操作叫作拆包
  # 类似的，列表、字符串、字典也可以被拆包
  a, b = [1, 2]
  a, b = ‘12’
  a, b = {1: ‘a’, 2: ‘b’} # 注意字典拆包时拆出来的是每个键
  # 拆包时，= 右侧的序列的长度需要与左侧的变量个数相同。如果不相同，可以使用 *变量 的形式一次接收多个元素
  a, *b = (1, 2, 3, 4)
  *a, b = (1, 2, 3, 4)
  a, *b, c = (1, 2, 3, 4)
  # 交换两个变量的值，可以简单地使用
  a, b = b, a
  # 如果 bool(a) 为 True，则将 a 赋值给 x，否则将 b 赋值给 x
  x = a or b
  # if else 三元表达式
  result = x if x > 0 else -x
  # for else 语句，只有在 for 循环没有被 break 时，才会执行后续 else 中的代码
  for i in range(5):
      print(i)
  else:
      print('所有项被迭代使用')
  # while else 语句，一旦 while 循环被 break，则后续的 else 语句将不被执行
  i = 0
  while i < 5:
      print(i)
      i += 1
  else:
      print('这是 else 语句')
  # try except else 语句，如果try下有异常抛出，则不执行 else 语句。如果没有异常抛出，则执行 else 语句
  try:
      pass
  except:
      print('有异常发生，不执行 else 语句')
  else:
      print('没有异常发生，执行 else 语句')
  ```

  ```python
  # 类属性 / 对象属性动态绑定
  # 装饰器 @property 可以将类中的方法转换为属性(只读)
  class A:
      @property
      def apple(self):
          return 'apple'
  # 属性可以被修改，用 @property 装饰一个方法，会自动生成名为 @方法名.setter 的装饰器
  class A:
      @property
      def apple(self):
          return self._apple
      
      @apple.setter
      def apple(self, value):
          self._apple = value
  ```

* 自定义异常

  ```python
  # 只需要定义一个类，这个类继承自 Exception 类或其子类即可
  class FileParseException(Exception):
      pass
  ```

* 函数相关

  ```python
  # 函数参数类型标注，写法如下，在每个参数的后面加上冒号（:）并标明类型
  def say_hello(name: str):
  	print(name, ', hello!')
  # 函数的返回值类型标注如下，在参数列表的后面加上右箭头（->）并标明类型
  def say_hello(name) -> str:
  	print(name, ', hello!')
  # Python 并不会根据标注对参数、返回值作类型校验，这只是为了方便阅读和 IDE 静态分析。  
  ```

* 文件读写

  ```python
  # 打开文件使用内置函数 open()，返回值为 file 对象
  f = open('文件路径', 读写模式)
  # 'r'：只读，若文件不存在则抛出 FileNotFoundError 异常
  # 'w'：只写，将覆盖所有原有内容，若文件不存在则创建文件
  # 'a'：只写，以追加的形式写入内容，若文件不存在则创建文件
  # 'r+'：可读可写，若文件不存在则抛出 FileNotFoundError 异常
  # 'w+'：可读可写，若文件不存在则创建文件
  # 'a+'：可读可写，写入时使用追加模式，若文件不存在则创建文件
  # 以上所有读写模式都是基于文本内容的，如果想要读写二进制内容，可在上面的基础上添加 'b' 模式，如 rb、'wb+'
  # 开方式默认使用 UTF-8 编码，如果文件内容并非 UTF-8 编码，可以使用 encoding 参数指定编码格式，如 f = open('/Users/obsession/text', 'w', encoding='gbk')
  # 写入文件
  length = f.write('内容')
  # 读取文件，也可以指定要读取内容的字符长度
  content = f.read()
  # 按行来读取文件
  line = f.readline()
  # 一次性将所有行读出，然后放进列表里
  lines = f.readlines()
  # 关闭文件使用
  f.close()
  # 自动关闭打开的文件，使用 with 语句
  with open('/Users/obsession/text', 'w') as f:
      f.write('The quick brown fox jumps over the lazy dog')
  ```

* 文件系统操作

  ```python
  import os
  # 创建目录
  os.mkdir('/Users/obsession/test_dir')
  # 判断路径是否是一个目录
  os.path.isdir('/Users/obsession/test_dir')
  # 列举目录下的内容
  os.listdir('/Users/obsession') 
  # 删除目录
  os.rmdir('/Users/obsession/test_dir')
  # 创建文件
  f = open('/Users/obsession/test_file', 'w')
  f.close()
  # 判断路径是否是一个文件
  os.path.isfile('/Users/obsession/test_file')
  # 删除文件
  os.remove('/Users/obsession/test_file')
  # 重命名文件
  os.rename('/Users/obsession/test_file', 'test_file_02')
  ```

* 序列化与反序列化

  * pickle

    * Python 中内置的 `pickle` 模块用作序列化和反序列化。它的序列化结果是二进制形式

    ```python
    import pickle
    # 对象被序列化后的二进制
    some_bytes = pickle.dumps(对象)
    # 使用 pickle.loads() 将序列化后的结果反序列化回对象
    pair = pickle.loads(some_bytes)
    # 与 open() 相结合，将序列化的结果保存在文件中，此时使用 pickle.dump()
    with open('/Users/obsession/dump', 'wb') as f:
        pickle.dump(pair, f)
    # 从文件中反序列化出对象，使用 pickle.load()
    with open('/Users/obsession/dump', 'rb') as f:
        pair = pickle.load(f)
    ```

  * json

    ```python
    import json
    # pair 对象序列化为 JSON 字符串
    json_string = json.dumps(pair.__dict__)
    # 使用了 pair.__dict__ 来获取包含所有 pair 属性的字典，因为类对象不能直接用于 json.dumps() 序列化，而字典可以
    # 或者使用 default 参数，向 json.dumps() 告知如何进行从对象到字典的转换，这样便可以不使用 __dict__ 属性
    def pair_to_dict(pair):
    	return {
    		'first': pair.first,
    		'second': pair.second,
    	}
    json_string = json.dumps(pair, default=pair_to_dict)
    # json 也可以与 open() 结合使用，将序列化的结果保存在文件中
    with open('/Users/obsession/json', 'w') as f:
        json.dump(pair, f, default=pair_to_dict)
    ```

    ```python
    # 从 JSON 反序列化为对象：
    def dict_to_pair(d):
        return Pair(d['first'], d['second'])
    
    pair = json.loads(json_string, object_hook=dict_to_pair)
    # json.loads() 首先会将 JSON 字符串反序列化为字典，然后使用 object_hook 参数进一步从字典转换出 pair 对象
    # 从文件中反序列化出对象
    with open('/Users/obsession/json', 'r') as f:
        pair = json.load(f, object_hook=dict_to_pair)
    ```

* 进程

  ```python
  import multiprocessing
  # 创建进程
  p = multiprocessing.Process(target=目标函数, args=(目标函数的参数,))
  # 用 start() 方法来启动一个进程
  p.start()
  ```

  ```python
  # process.py
  import multiprocessing
  import os
  
  def target_func():
      print('子进程运行')
      print('子进程 pid:', os.getpid())
      print('子进程的 ppid:', os.getppid())
  
  p = multiprocessing.Process(target=target_func)
  p.start()
  # 使用 join() 方法可以控制子进程的执行顺序,主进程将等待子进程执行完成，然后再向下执行代码
  p.join()
  
  print('主进程运行')
  print('主进程 pid:', os.getpid())
  ```

* 线程

  ```python
  # 使用 threading.Thread() 方法来创建线程
  import threading
  
  t = threading.Thread(target=目标函数, args=(目标函数的参数,))
  # 用 start() 方法来启动一个线程
  t.start()
  ```

  ```python
  # thread.py
  import threading
  
  def target_func(n):
      for i in range(n):
          print(i)
  
  t = threading.Thread(target=target_func, args=(8,))
  t.start()
  # 使用 join() 让主线程等待子线程执行完成
  t.join()
  
  print('主线程结束')
  ```

* 线程锁

  ```python
  # 创建了两个线程，这两个线程分别对 number 变量做一百万次 +1 操作
  # thread_add.py
  import threading
  
  number = 0
  # 1. 创建锁
  lock = threading.Lock()
  
  def add():
      for i in range(1000000):
          global number
          
          # 2. 加锁
          lock.acquire()
          number += 1
          # 3. 释放锁
          lock.release()
  
  t_1 = threading.Thread(target=add)
  t_2 = threading.Thread(target=add)
  t_1.start()
  t_2.start()
  t_1.join()
  t_2.join()
  
  print(number)
  # number 的预期结果应该是 2000000（两百万）
  # 每次运行的结果并不一致，并且均小于 2000000
  # 想要得到正确的结果，应该对 number += 1 操作加锁
  ```

* 安装第三方包

  ```python
  # 要安装第三方包可以使用命令行工具 pip，它将从 Python Package Index 中检索并下载安装所指定的包
  pip3 install 包名
  # 如安装 Python 下非常流行的 HTTP 客户端工具 requests
  pip3 install requests
  # 安装成功后，就可以在代码中使用 import requests 来导入这个包了
  # 指定包的版本
  pip3 install requests=2.22.0
  # 将一个已有的包升级到最新版本
  pip3 install --upgrade requests
  # pip 会将第三方包安装在 Python 目录下的 lib/python3.x/site-packages 目录中。如果项目代码中使用了某个第三方包，则 Python 会到这个目录中寻找
  # 可以使用 pip3 show 命令来查看第三方包的描述信息
  pip3 show requests
  ```

* 虚拟环境

  ```python
  # 拥有多个项目时，可以对这些项目分别创建单独的虚拟环境，每个虚拟环境就是一个专有的 Python 运行环境
  # 在一个虚拟环境中安装第三方包，这个包只能在当前虚拟环境中使用，其它虚拟环境中是无法看到它的，所以也就不会再有项目间版本冲突的问题
  # 每个虚拟环境中只包含当前项目所使用到的包，不会包含大量无用的包
  ```

  ```python
  # 创建虚拟环境使用命令
  python3 -m venv 虚拟环境名称
  # 将在当前目录中创建出 venv 目录，这即是虚拟环境目录，其中包含 Python 运行环境
  python3 -m venv venv
  # 要使用虚拟环境首先需要激活它，激活后当前命令行所执行的 Python 代码都将运行于该虚拟环境中
  # Windows 操作系统中使用命令：
  cd venv\Scripts
  activate.bat
  # Linux 和 MacOS 中使用命令：
  source venv/bin/activate
  # 上述命令执行过后，命令行的提示符前将显示 (venv)
  (venv) ➜
  # 退出虚拟环境
  # 不要使用虚拟环境时，可以退出当前虚拟环境
  # Windows 操作系统中使用命令：
  cd venv\Scripts
  deactivate.bat
  # Linux 和 MacOS 中使用命令：
  deactivate
  ```

* 标准库

  ```python
  # datetime：日期和时间处理相关
  # random：随机取值相关
  # josn：json 相关
  # XML：XML 相关
  # collections：集合相关
  # base64：base64 编码相关
  # hashlib：摘要算法相关，如 MD5、SHA1
  # itertools：迭代工具相关
  # contextlib：上下文管理相关
  # urllib：HTTP 请求相关
  ```

  







