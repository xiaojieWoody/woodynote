# 导学

* python 3.8
  * 简洁灵活优雅哲学、人生苦短，我用python、跨平台、强大丰富的库、面向对象的语言
  * 爬虫、大数据、测试、web、AI、脚本处理
  * python之禅
    * 简洁胜于复杂
    * 做也许好于不做，但不假思索就做还不如不做
  * 豆瓣、知乎
  * 易于上手难于精通
  * 既有动态脚本的特性，又有面向对象的特性，非常具有自己的特点
  * 缺点
    * 运行效率慢，相较于C、C++、Java，但是开发效率高
    * 编译型语言（C、C++）、解释型语言（JavaScript、Python）
    * 运行效率和开发效率不可兼得，适合才是最好的
* python能做什么
  * 几乎是万能的
    * 当遇到问题时，随手拿起Python，编写一个工具，这才是Python正确的打开方式
  * 爬虫
  * 大数据与数据分析（Spark）
  * 自动化运维与自动化测试
  * Web开发（Flask、Django）
  * 机器学习（Tensor Flow）
  * 胶水语言：混合其他语言，如Java、C++等来编程。能够把用其他语言制作的各种模块（尤其是C/C++）能够很轻松地联结在一起
* 课程特点
  * 基础语法
  * Pythonic
    * Python的语法是非常灵活的又别具一格的。学习语言就要学习它的风格、特点，这才是语言的精粹。Python尤其如此
    * a和b两个值交换，a,b=b,a
  * 高性能与优化
    * 选择性能最高又易于理解的写法才是最正确的
  * 数据结构
  * 框架太多、类库太多、技术太多。让我们回归语言的本质，享受语言本身的纯粹之美
* 《流畅的Python》

# 环境安装

* python3.8

* 虚拟环境，每个项目，指定一个python环境
* 运行：python3+文件名

* pipenv

  * 先**全局**安装pipenv（任意目录下）

    * `pip install pipenv`

  * 使用pipenv创建一个虚拟环境和项目绑定

    * 创建`new_env` 目录并进入

    * `pipenv install  --three`

      * 当前 `new_env` 环境下生成 `Pipfile` 和 `Pipfile.lock` 两个环境初始化文件

    * `pip list `

    * 进入虚拟环境

      * `pipenv shell`，激活当前项目的虚拟环境
      * `pip list`
      * 查看依赖关系，`pipenv graph`

    * 目录下，`pipenv install flask -i https://pypi.tuna.tsinghua.edu.cn/simple`

    * `flask`

    * ```shell
      # 查看虚拟环境目录,最后的虚拟环境目录是以当前环境 new_env 作为目录开头的
      pipenv --venv
      # 查看项目根目录
      pipenv --where
      # 检查软件包的完整性,已安装的软件包有没有安全漏洞
      pipenv check
      # 查看依赖树
      pipenv graph
      # 环境变量管理
      # 开发调试时需要配一堆环境变量，可以写到 .env 文件中，在 pipenv shell 进入虚拟环境时，它会把这些环境变量加载好，非常方便
      # 如果项目根目录下有.env文件，$ pipenv shell和$ pipenv run会自动加载它
      echo "FOO=hello foo" > .env
      #  pipenv shell 进入虚拟环境，echo $FOO 就能看环境变量的值 hello foo 已经设置好
      ```
      
    * ```shell
      # python文件的运行
      # 方式一
      pipenv run python xxx.py
      # 方式二：在激活环境中运行
      pipenv shell
      python xxx.py
      ```

# 其他

* `if __name__ == '__main__'`的意思是：当.py文件被直接运行时，`if __name__ == '__main__'`之下的代码块将被运行；当.py文件以模块形式被导入时，`if __name__ == '__main__'`之下的代码块不被运行

* with

  * with 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭／线程中锁的自动获取和释放等

    ```python
    with open("１.txt") as file:
        data = file.read()
    ```

  * 工作原理

    * 紧跟with后面的语句被求值后，返回对象的“–enter–()”方法被调用，这个方法的返回值将被赋值给as后面的变量

    * 当with后面的代码块全部被执行完之后，将调用前面返回对象的“–exit–()”方法

      ```python
      class Sample:
          def __enter__(self):
              print "in __enter__"
              return "Foo"
          def __exit__(self, exc_type, exc_val, exc_tb):
              print "in __exit__"
      def get_sample():
          return Sample()
      with get_sample() as sample:
          print "Sample: ", sample
      
      # in __enter__
      # Sample:  Foo
      # in __exit__    
      ```

      * 可以看到，整个运行过程如下：

        （１）**enter**()方法被执行；
        （２）**enter**()方法的返回值，在这个例子中是”Foo”，赋值给变量sample；
        （３）执行代码块，打印sample变量的值为”Foo”；
        （４）**exit**()方法被调用；

      * 【注：】**exit**()方法中有３个参数， exc_type, exc_val, exc_tb，这些参数在异常处理中相当有用。
        exc_type：　错误的类型
        exc_val：　错误类型对应的值
        exc_tb：　代码中错误发生的位置

    * 实际上，在with后面的代码块抛出异常时，**exit**()方法被执行。开发库时，清理资源，关闭文件等操作，都可以放在**exit**()方法中。

    * 总之，with-as表达式极大的简化了每次写finally的工作，这对代码的优雅性是有极大帮助的。
      如果有多项，可以这样写：

      ```python
      With open('1.txt') as f1, open('2.txt') as  f2:
          do something
      ```

# Python基本类型

* 什么是代码

  * 代码是现实世界事物在计算机世界中的映射

* 什么是写代码

  * 将现实世界中的事物用计算机语言来描述

* Number：数字

  * 整数int、浮点数float

    ```python
    >>> type(1)
    <class 'int'>
    >>> type(1.1)
    <class 'float'>
    >>> type(2/2)
    <class 'float'>
    >>> type(2//2)
    <class 'int'>
    >>> type(1//2)    # 保留整数
    <class 'int'>
    ```

  * 10、2、8、16进制

    * 2进制转10进制，前面加0b，如0b10=2
    * 8进制转10进制，前面加0o，如0o10=8
    * 16进制转10进制，前面加0x，如0x10=16，0x1F=31

    ```python
    # 其他进制转2进制
    >>> bin(10)
    '0b1010'
    >>> bin(0o7)
    '0b111'
    >>> bin(0xE)
    '0b1110'
    # 其他进制转10进制
    >>> int(0b111)
    7
    >>> int(0o77)
    63
    # 其他进制转为16进制
    >>> hex(888)
    '0x378'
    >>> hex(0o7777)
    '0xfff'
    # 其他进制转为8进制
    >>> oct(0b111)
    '0o7'
    >>> oct(0x777)
    '0o3567'
    ```

  * bool布尔类型

    * int(True) = 1
    * int(False) = 0
    * bool(1) = True
    * bool(-1.1)=True
    * bool(0) = False
    * bool('')=False
    * bool([])=False
    * bool({})=False
    * bool(None)=False
    
  * complex复数

    * 36j
    
    * 是由一个实数和一个虚数组合构成，表示为：**x+yj**
    
    * 一个复数时一对有序浮点数 (x,y)，其中 x 是实数部分，y 是虚数部分
    
      ```python
      1、虚数不能单独存在，它们总是和一个值为 0.0 的实数部分一起构成一个复数
      2、复数由实数部分和虚数部分构成
      3、表示虚数的语法：real+imagej
      4、实数部分和虚数部分都是浮点数
      5、虚数部分必须有后缀j或J
      #coding=utf8
      aa=123-12j
      print aa.real  # output 实数部分 123.0  
      print aa.imag  # output虚数部分 -12.0
      ```

* 字符串str

  * 单引号

  * 双引号

  * 三引号

    ```python
    >>> "let's go"
    >>> 'let\'s go'
    >>> 'let"s go'
    >>> '''
    ... hello world
    ... hello world
    ... '''
    '\nhello world\nhello world\n'
    
    >>> """hello \nhello \nhello"""
    'hello \nhello \nhello'
    >>> print("""hello \nhello \nhello""")   # print可识别换行
    hello
    hello
    hello
    >>> print('hello \\n world')
    hello \n world
    >>> print('c:\\user\\data')
    c:\user\data
    >>> print(r'c:\user\data')          # 不是一个普通的字符串，而是一个原始字符串
    c:\user\data
    ```

* 转义字符
  * 特殊的字符
    * 无法‘看见’的字符
    * 与语言本身语法有冲突的字符

* 字符串运算

  ```python
  >>> "hello" + "world"
  >>> "hello"*2
  >>> "hello wrold"[0] # h
  >>> "hello wrold"[-1] # d
  >>> "hello world"[0:4] # hell
  >>> "hello world"[0:-1] # hello worl 包左不包右  , 步长
  ```

# Python中“组”

* 列表

  ```python
  >>> [1,2,3,4,5,6]
  [1, 2, 3, 4, 5, 6]
  >>> type([1,2,3,4,5,6])
  <class 'list'>
  >>> type(["hello",1,2,True,[1,2,"world",False]])
  <class 'list'>
  >>> ["新月打击","苍白之瀑","月之降临","月神冲刺"][0]
  '新月打击'
  >>> ["新月打击","苍白之瀑","月之降临","月神冲刺"][0:2]
  ['新月打击', '苍白之瀑']
  >>> ["新月打击","苍白之瀑","月之降临","月神冲刺"][-1:]
  ['月神冲刺']
  >>> ["新月打击","苍白之瀑","月之降临","月神冲刺"]+["点燃","闪现"]
  >>> ["点燃","闪现"]*2
  ```

* 元组

  ```python
  >>> type((1, 2, '-1', True))
  <class 'tuple'>
  >>> (1, 2, '-1', True)[0]
  1
  >>> (1, 2, '-1', True)[0:2]
  (1, 2)
  >>> (1,2)+(3,4)
  (1, 2, 3, 4)
  >>> (1,2)*2
  (1, 2, 1, 2)
  >>> type((1))             # int
  <class 'int'>
  >>> type(('hello'))
  <class 'str'>
  >>> (1+1)*2
  4
  >>> type((1,))           # 只有一个元素的元组
  <class 'tuple'>
  ```

* 序列

  ```python
  # str、list、tuple 共同操作
  x[0]
  x[0:2]
  x[0:8:2]
  +
  * 
  >>> 3 in [1,2,3,4,5,6]
  True
  >>> 3 not in [1,2,3,4,5,6]
  False
  >>> len([1,2,3,4,5,6])
  6
  >>> max([1,2,3,4,5,6])
  6
  >>> min([1,2,3,4,5,6])
  1
  ```

* 集合Set

  ```python
  # 无序、不重复
  >>> type({1,2,3,4,5})
  <class 'set'>
  len({1,2,3})
  1 in {1,2,3}
  1 not in {1,2,3}
  >>> {1,2,3,4,5,6} - {3,4}
  {1, 2, 5, 6}
  >>> {1,2,3,4,6,5} & {3,4}
  {3, 4}
  >>> {1,2,3,4,5,6} | {3,4,7}
  {1, 2, 3, 4, 5, 6, 7}
  >>> type({})
  <class 'dict'>
  >>> type(set())
  <class 'set'>
  >>> len(set())
  0
  ```

* 字典

  ```python
  # key是不可变的，list不行，tuple可以
  >>> type({1:11,2:22})
  <class 'dict'>
  >>> {1:11,2:22}[1]
  11
  >>> type({})
  <class 'dict'>
  ```

  ![image-20200616152626186](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200616152626186.png)

# 变量与运算符

* 字母、数字、下划线，不能以数字开头、大小写敏感

```python
# 值类型 int、str、tuple，不可变
>>> a = 1
>>> b = a
>>> a = 3
>>> print(b)
1
# 引用类型 list、set、dict，可变
>>> a = [1,2,3,4,5]
>>> b = a
>>> a[0] = '1'
>>> print(a)
['1', 2, 3, 4, 5]
>>> print(b)
['1', 2, 3, 4, 5]

>>> a = 'hello'
>>> id(a)
4378293616
>>> a = a + ' python'
>>> id(a)
4378294512
>>> print(a)
hello python
```

```python
>>> b = [1,2]
>>> b.append(3)
>>> print(b)
[1, 2, 3]
>>> a = (1,2,3,[1,2,['a','b','c']])
>>> a[3][2][1]
'b'
>>> a[3][1]='d'           # 不能修改元组元素，但能修改元组里list元素中的某个值
>>> print(a)
(1, 2, 3, [1, 'd', ['a', 'b', 'c']])
```

```python
# 算数运算符
+、-、*、/、//、%、**
# 赋值运算符
=、+=、-=、*=、/=、//=、%=
# 比较运算符
==、!= 、> 、< 、>=、<=
# 逻辑运算符
and、or、not
# 成员运算符
in、not in
# 身份运算符
is、is not
# 位运算符，把数字当作(转成)二进制数进行运算
&按位与、|按位或、^按位异或、~按位取反、<<左移动、>>右移动

# 不只是数字才能做比较运算
>>> b=1
>>> b+=b>=1
>>> print(b)
2
>>> print(b>=1)
True
>>> int(True)
1
>>> b += True
>>> print(b)
3

>>> ord('a')
97
>>> ord('b')
98
>>> 'a' > 'b'
False

# 只对dict中key
>>> a = 'c'
>>> b = {'c':1}
>>> a in b
True
>>> a = 1
>>> a in b
False

# == 比较值，is 比较id身份（内存地址）
>>> a = 1
>>> b = 1.0
>>> a is b
False
>>> a == b
True

>>> a = {1,2,3}
>>> b = {1,3,2}
>>> a == b
True
>>> a is b
False

>>> c = (1,2,3)
>>> d = (2,1,3)
>>> c == d
False
>>> c is d
False

>>> a = 'hello'
>>> isinstance(a,str)
True
>>> isinstance(a,(int,str,float))
True
# 对象的三个特征
id、value、type
is、==、isinstance
```

# 分支、循环、条件与枚举

* 表达式是运算符和操作数所构成的序列

  ```python
  and 优先级高于 or
  >>> a = 1
  >>> b = 2
  >>> c = 2
  >>> not a or b + 2 == c
  False
  >>> ((not a) or ((b + 2) == c))
  False
  >>>
  ```

* 流程控制

  ```python
  if else
  for
  while
  ```

  ```python
  # 注意缩进
  mood = True
  if mood : 
      print('go to left')
      print('back away')
  else :
      print('go to right')
      
  # 用户输入    （是字符串）  数字的话，需要强转下 a = int(x)
  print('please intou account')
  user_account = input()
  print(user_account)
  
  if expression:
      pass   # 空语句/占位语句
  ```

  ```python
  a = 1
  if a = 1 :
      pass
  elif a = 2 :
      pass
  else :
      pass
  ```

* 循环

  ```python
  # while for
  condition = 1
  while condition < 10 :
      print("Hello")
      condition += 1
  else :
      print("EOF")   
  
  x = [[1,2,3],('a','b')]
  for a in x:
      for b in a:
          print(b)
  else : 
      print("EOF")    
      
  x = [1,2,3]
  for a in x:
      if a == 2 :
          break
      print(a)   
      
  x = [1,2,3]
  for a in x:
      if a == 2 :
          continue
      print(a)     
  # break结束当前循环，如果在嵌套循环里，则结束内部循环，外部循环不受影响
  # continue结束这次执行，下次循环继续执行
  
  # 包左不包右
  for x in range(0,10) :
      print(x)
  
  # 步长 2
  for x in range(0,10,2) :
      print(x, end='|')   
  
  for x in range(10,0,-2) :
      print(x, end='|')     
  
  a = [1,2,3,4,5,6,7,8]
  for i in range(0, len(a),2):
      print(a[i], end=' | ')
  
  a = [1,2,3,4,5,6,7,8]
  b = a[0:len(a):2]
  print(b)    
  ```

# 包、模块、函数与变量作用域

* 包
  * 模块
    * 类
      * 函数、变量

```python
# 包
__init__.py     # 如果要引用的该py，则直接用包名，而非__init__

# 导入模块 (x.py)
import 包名.模块名
import 包名.模块名 as m

from 包名.模块名 import 变量名/函数/模块名

__all__=['a','c']
a = 2
b = 3
c = 4
from 包名.模块名 import *  # 只会导入 a 和 c

from c9 import a,b,c

from c9 import (a,b,
c)

# 导入包时，自动执行__init__

# __init__.py   t包中
__all__ = ['c7']       # 内容

from t import *     # 只导入c7
print(c7.a)
print(c8.e)


# __init__.py  # t包 可批量导入包
import sys
import datetime
import io

import t
print(t.sys.path)


# 包和模块是不会重复导入的
# 避免循环导入
# 导入一个模块时，会执行该模块，只会执行一次

# 入口文件的概念
```

```python
# 内置变量
# c14.py
a = 2
c = 3
d = 5
infos = dir()   # 当前模块里所有的变量，包括内置变量
print(infos) 

# t包下 c9.py
···
	This is a c9 doc
···
print('name:' + __name__)      # t.c9 , 如果该文件是入口文件（执行python3 c9.py），则值强制改为 __main__ 且 不属于任何包
print('package:' + (__package__ or '当前模块不属于任何包'))  # t
print('doc:' + (__doc__ or '当前模块没有文档注释'))					 # 文档的注释，This is a c9 doc
print('file:' + __file__)        # c9.py绝对路径，如果是入口文件，则值为python命令执行时的值
```

```python
# c16.py
import sys
infos = dir(sys)
print(infos)

# c17.py
# 当前文件是否是入口文件
if __name __ == '__main__' :
  print('This is app')
print('This is a module')  

# c18.py
import c17
# 执行python3 c18.py 会打印 This is a module
```

```python
# 顶级包是相对于入口文件的（例如main.py）

# 相对导入
. 		# 当前目录
..    # 上级目录 
...   # 上上级目录

# 绝对导入
from 顶级包名.包名.模块名 import 变量名
# 相对导入
from .模块名 import 变量名
from .m3 import m
```

# Python函数

* 功能性
* 隐藏细节
* 避免编写重复的代码

```python
a = 1.12386
result = round(a,2)        # 保留小数点后2位，四舍五入
print(result)

def funcname(parameter_list):
  pass

# 参数列表可以没有
# return value   None

def add(x,y):
  result = x + y
  return result

# 关键字参数
res = add(y=2,x=3)

# 返回多个值
def damage(skill1, skill2):
  damage1 = skill1 * 3
  damage2 = skill2 * 2 + 10
  return damage1, damage2

# 元组
damages = damage(3,6)
print(damages[0], damages[1])
# 推荐
skill1_damage, skill2_damage = damage(3, 6)
print(skill1_damage, skill2_damage)
```

```python
# 序列解包
a,b,c = 1,2,3
a,b,c = d
d = 1,2,3
# 默认参数
def default_fun_parm(name, gender='男', age=18):
  pass
default_fun_parm('zs')
# 可变参数
def demo(*param):
  print(param)                     # (1,2,3,4,5,6)
  print(type(param))               # <class 'tuple'>
demo(1,2,3,4,5)  

a = (1,2,3,4,5,6)
demo(*a)       # 相当于解包

def city_temp(**param):
  for key,value in param.items():
    print(key, ':', value)

a = {'bj':'32c', 'sh':'31c'}
city_temp(**a)
```

* 变量作用域

  ```python
  c = 50
  def add(x,y):
    c = x + y
    print(c)
  # 变量的作用域
  add(1,2)     # 3
  print(c)     # 50
  ```

  ```python
  def demo():
    	c = 50
      for i in range(0,9):
        a = 'a'
        c += 1
      print(c)  # 59
      print(a)  # a    可以引用for内部的变量
  
  demo()    
  ```

  ```python
  # global关键字
  def demo():
    global c             # 变成全局变量
    c = 2
  
  demo()
  print(c)
  ```

  ![image-20200617013805236](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200617013805236.png)

  ![image-20200617014102218](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200617014102218.png)

  ![image-20200617014019757](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200617014019757.png)

  ![image-20200617014127072](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200617014127072.png)

# 面向对象

```python
# c1.py
class Student():
  name = ''          # 全局变量
  age = 0
  
  # 必须加self
  def print_file(self):
    print('name:' + self.name)
    print('age:' + str(self.age))
      
	# print_file()     # 类只负责定义，不负责执行，不能在class的内部调用函数，否则会报错

      
# 实例化放在另一个模块，class只负责定义类  
# c2.py
from c1 import Student
student = Student()  
student.print_file()
```

* 方法和函数的区别
  * 方法：设计层面
  * 函数：程序运行、过程式的一种称谓

* 类

  * 是现实世界或思维世界中的实体在计算机中的反映，它将数据以及这些数据上的操作封装在一起

    ```python
    class Student():
      sum1 = 0
      sum = 0
      count = 0
      __score =  0									# 私有变量
      name = 'jack'                # 全局变量
      age = 0
      
      # 构造函数，初始化对象时自动调用
      # 
      #def __init__(self):
      #    print('student')
      
      # 类变量 实例变量
    	def __init__(self,name,age):
          # name = name          # 局部变量
          # age = age						 # 全局变量的值不会因为局部变量而改变
          self.name = name          # 实例变量
          self.age = age						
          print('name:' + name)     # 取实例变量值（通过形参名称查找）
          print('age:' + str(age))  
          print(self.__dict__)
          print('name:' + self.name) # 加self，推荐
          # print(sum1)          # 不要不通过前缀就访问类变量，否则报错
          print(Student.sum1)
          self.__class__.sum1 += 1    # 操作类变量，累加
          print(self.__class__.sum1)  # 推荐
          self.__score = 0
          
      # 行为与特征
      def do_homework(self) :
        print('homework')
        
    	# 类方法
      @classmethod              # 装饰器
      def plus_sum(cls):        # 参数名字可变
        cls.count += 1
        # print(self.name)      # 不能访问实例变量
        # print(name)						 # 报错
        print(cls.count)
        
    	# 静态方法
      @staticmethod
      def add(x,y):
        print(Student.sum1)
        # print(self.name)       # 不能访问实例变量 
        # print(name)						 # 报错
        print('This is a static method')
        
    	def marking(self, score):
        if score < 0
        	return '不能够打负分'
        self.__score = score
        print(self.name + ':' + str(self.__score)
    
    	# 私有方法，外部不能直接访问          
    	def __private_method(self):
    		print('private')
      
    # student1 = Student()
    student2 = Student('张三',12)
    print(student2.name)   # 张三
    print(student2.__dict__)    # 打印所有实例变量   {'name': '张三', 'age': 12}
    print(Student.name)    # jack
    student2.marking(59)
    # student2.score = -1       # 不要通过对score直接赋值
    student2.__score = -1       # 私有变量，没报错，因为新相当于动态添加了一个实力变量，原来的私有变量已改名，外部无法访问到
    print(student1.__dict__)      
    print(student1._Student_score)  # 间接读取私有变量
              
    
    # 首先访问实例变量，不存在则继续查找同名类变量
    
    # 成员可见性
    # 方法或变量前加__，则表示私有，如果前后都加__name__则不是私有  
              
    # 封装、继承、多态
    ```

  * 继承

    ```python
    # 可多继承
    # test3.py
    class Human():
        sum = 0
        def __init__(self, name, age):
            self.name = name
            self.age = age
    
        def get_name(self):        
            print(self.name)
    
        def do_homework(self):
            print('This is a parent method')  
    
    # test2.py
    from test3 import Human
    
    class Student(Human):
    
        def __init__(self, school, name, age):
            self.school = school
            # 调用父类构造函数
            # Human.__init__(self, name, age)
            super(Student, self).__init__(name, age)
    
        def do_homework(self):
          	# 调用父类构造方法
            super(Student, self).do_homework()
            print('english homework')         
    
    
    student1 = Student('实验小学', 'jack', 19)  
    print(student1.name)
    print(student1.age)  
    
    student1.do_homework()
    
    print(student1.sum)
    print(Student.sum)
    print(student1.name)
    print(student1.age)
    student1.get_name()
    ```

# 正则表达式与JSON

* 正则表达式

  * 是一个特殊的字符序列，一个字符串是否与我们所设定的这样的字符序列，相匹配
  * 快速检索文本、实现一些替换文本的操作

  ```python
  # 检查一串数字是否是电话号码
  # 检查一个字符串是否符合email
  # 把一个文本里指定的单词替换为另外一个单词
  import re
  
  a = 'C|C++|JAVA|C#|Python|Javascript'
  
  # print(a.index('Python') > -1)
  # print('Python' in a)
  
  r = re.findall('Python', a)
  if len(r) != 0:
      print('字符串中包含Python') 
      print(r)              # ['Python']
  else:
      print('No')  
  ```

  ```python
  import re
  
  a = 'C0C++JAVA8C#9Python6Javascript'
  s = 'anc, acc, adc, aec, afc, ahc'
  
  r1 = re.findall('a[cf]c', s)     # c或f(一个)，['acc', 'afc']
  r2 = re.findall('a[^cf]c', s)     # 非c非f，['anc', 'adc', 'aec', 'ahc']
  r3 = re.findall('a[c-f]c', s)    # c-f，['acc', 'adc', 'aec', 'afc']
  
  r4 = re.findall('[0-9]', a)     # ['0', '8', '9', '6']
  r5 = re.findall('[^0-9]', a)    # 非0-9数字
  r6 = re.findall('\w', a)        # 数字和单词字符
  r8 = re.findall('\W', a)        # 非单词字符
  r7 = re.findall('[_A-Za-z0-9]', a) # 同 \w
  r9 = re.findall('\s', a)        # 空白字符
  r10 = re.findall('\S', a)        # 非空白字符
  
  print(r7)
  
  r = re.findall('\d', a)         # ['0', '8', '9', '6'] 数字是拆开的
  rr = re.findall('\D', a)        # 非数字  
  ```

  ```python 
  # 数量词
  import re
  
  a = 'python 1111java678php'
  b = 'pytho0python1pythonn2'
  
  # 贪婪 
  r1 = re.findall('[a-z]{3,6}', a) # ['python', 'java', 'php']
  # 非贪婪
  r2 = re.findall('[a-z]{3,6}?', a) # ['pyt', 'hon', 'jav', 'php']
  # *前面的字符匹配0次或者无限多次
  r3 = re.findall('python*', b)   # ['pytho', 'python', 'pythonn']
  # + 匹配1次或者无限多次
  r4 = re.findall('python+', b)   # ['python', 'pythonn']
  # ? 匹配0次或1次, 可做 截取
  r5 = re.findall('python?', b)   # ['pytho', 'python', 'python']
  ```

  ```python
  import re
  qq = '100000000001'
  r1 = re.findall('\d{4,8}', qq)  # ['10000000', '0001']
  # 边界匹配（完全匹配） ^开头  $末尾
  r2 = re.findall('^\d{4,8}$', qq)  # []
  
  # 组 (且)
  a = 'PythonPythonPythonPythonPythonPython'
  r3 = re.findall('(Python){3}(JS)', a)    # []
  
  language = 'PythonC#\nJavaPHP'
  # 参数模式匹配
  # . 匹配出除了换行符\n之外其他字符
  r4 = re.findall('c#', language,  re.I)  # 忽略大小写 |   ['C#']
  r5 = re.findall('c#.{1}', language,  re.I | re.S)  # ['C#\n']
  
  def convert(value):
      # print(value)
      matched = value.group()
      return '!' + matched + '!'
  
  b = 'PythonC#JavaC#PHPC#'
  r6 = re.sub('C#', 'GO', b, 0)   # 0 都替换，1只替换1个
  r7 = re.sub('C#', convert, b)   # Python!C#!Java!C#!PHP!C#!
  ```

  * 函数作为参数传递

  ```python
  import re
  
  s = 'A8C3721D86'
  
  def convert(value):
      matched = value.group()
      if int(matched) >= 6:
          return '9'
      else :
          return '0'
          
  # 把函数作为参数传递
  r = re.sub('\d', convert, s)  # A9C0900D99
  ```

  ```python
  import re
  
  s = 'A83C72D1D8E67'
  
  # 首字母开始匹配，首字母没匹配到直接返回None，不再继续
  r = re.match('\d',s)
  # print(r.span())
  # 搜索，直到找到，匹配一次
  r1 =  re.search('\d', s) 
  print(r1.group())     # 8
  # 匹配所有
  r2 = re.findall('\d', s)
  
  # group 分组
  a = 'life is short, i use python'
  
  r3 = re.search('life.*python', a)
  print(r3.group())     # life is short, i use python
  
  r4 = re.search('life(.*)python', a)
  print(r4.group(0))    # life is short, i use python
  print(r4.group(1))    #  is short, i use 
  
  b = 'life is short, i use python, i love python'
  r5 = re.search('life(.*)python(.*)python', b)
  print(r5.group(0))     # life is short, i use python, i love python
  print(r5.group(1))     #  is short, i use 
  print(r5.group(2))     # , i love 
  print(r5.group(0,1,2)) # ('life is short, i use python, i love python', ' is short, i use ', ', i love ')
  print(r5.groups())     # (' is short, i use ', ', i love ')
  ```

* json

  ```python
  # 是一种轻量级的数据交换格式
  # 字符串是JSON的表现形式
  json:     object、array、string、number、number、true、false、null
  python:   dict  、list 、str   、int   、float 、True、False、None
  # JSON对象
  # JSON
  # JSON字符串
    
  import json
  
  json_str = '{"name":"zs", "age":18}'
  json_arr = '[{"name":"zs", "age":18, "falg":false}, {"name":"ls", "age":19}]'
  # 反序列化 json字符串 -> python字符串
  student = json.loads(json_str)
  print(student)       # {'name': 'zs', 'age': 18}
  print(type(student)) # <class 'dict'>
  print(student['name']) # zs
  
  s = json.loads(json_arr)
  print(s)       # [{'name': 'zs', 'age': 18, 'falg': False}, {'name': 'ls', 'age': 19}]
  print(type(s)) #  <class 'list'>
  
  # 序列化 
  stu = [
      {'name': 'zs', 'age': 18, 'falg': False}, 
      {'name': 'ls', 'age': 19}
      ]
  json_s = json.dumps(stu)
  print(json_s)     # [{"name": "zs", "age": 18, "falg": false}, {"name": "ls", "age": 19}]
  print(type(json_s)) # <class 'str'>  
  ```

# 高级语法与用法

* 枚举类

  ```python
  # 不可变、可防止相同标签
  # 不可做大小比较，可做 == 和 is比较
  # Python中没有常量
  from enum import Enum
  
  class VIP(Enum):
      YELLOW = 1          
      # 别名
      YELLOW_ALIAS = 1    # 1 == 1 ，被视作 YELLOW的别名，不作为单独元素，遍历时不被打印出来
      GREEN  = 2
      BLACK = 3
      RED = 4
  
  print(VIP.YELLOW.name)     # YELLOW
  print(VIP.YELLOW.value)    # 1
  print(VIP['YELLOW'])       # VIP.YELLOW
  
  print(type(VIP.YELLOW.name))  # <class 'str'>
  print(type(VIP.YELLOW))       # <enum 'VIP'>
  print(type(VIP['YELLOW']))    # <enum 'VIP'>
  
  # 别名不被遍历
  for v in VIP:
      print(v)
  
  print('---------------')
  # 会遍历别名
  for v_1 in VIP.__members__:
      print(v_1)    
  
  for v_2 in VIP.__members__.items() :
      print(v_2)   
  
  # 枚举转换
  a = 1
  print(VIP(a))   # VIP.YELLOW 
  
  
  # 整型限制
  from enum import IntEnum,unique
  
  class VIP_INT(IntEnum):
      YELLOW = 1
      GREEN = 2
      BLACK = 2     # unique, 会报错
  ```

* 函数式编程、闭包

  ```python
  # 函数，只是一段可执行代码，并不是对象
  # 一切皆对象
  # 闭包，函数式编程的一种
  
  def curve_pre():       # 闭包的意义，保护现场
      a = 25             # 闭包 = 函数 + 环境变量（函数定义时候） ，定义在函数外部（被调用函数的外部）
      def curve(x):
          return a * x * x
      return curve       # 返回函数，闭包
  
  a = 10        
  f = curve_pre()
  print(f.__closure__)
  print(f.__closure__[0].cell_contents)   # 25
  print(f(2))      # 100，不受外面 a = 10的影响，a还是取25，闭包
  
  
  a = 10
  def f1(x):
      return a * x * x
  
  print(f1(2))     # 40
  ```

  ```python
  def f1():
      a = 10
      def f2():
          a = 20				# 被认为是局部变量，不是闭包
          print(a)      # 20   局部变量不影响外部变量
      print(a)          # 10
      f2()
      print(a)          # 10
  
  f1() 
  ```

  ```python
  def f1():
      a = 10
      def f2():
          # a = 20  # 被认为是一个局部变量，和外部a没关系，不注释掉，不是闭包
          # return a   
          c = 20 * a   # 不一定需要返回，但是需要引用外部变量
      return f2
  
  f = f1()
  print(f)                 # <function f1.<locals>.f2 at 0x10359e700>
  print(f.__closure__)     # (<cell at 0x10356d190: int object at 0x10326ebb0>,)
  print(f.__closure__[0].cell_contents)   # 10
  ```

  ```python
  # 非闭包
  origin = 0
  
  def go(step):
      global origin
      new_pos = origin + step
      origin = new_pos
      return new_pos
  
  print(go(2))     #  2
  print(go(3))     #  5
  print(go(6))     #  11
  ```

  ```python
  count = 1
  
  def a():
      count = 'a函数里面'  　　#如果不事先声明，那么函数b中的nonlocal就会报错
      def b():
        # nonlocal只能在封装函数中使用，在外部函数先进行声明，在内部函数进行nonlocal声明，这样在b()函数中的count与a()中的count是同一个变量
          nonlocal count
          print(count)
          count = 2
      b()
      print(count)
  
  if __name__ == '__main__':
      a()          # a函数里面 2
      print(count) # 1
  ```
  
  ```python
  # 闭包
  origin = 0
  
  def factory(pos):
      def go(step):
          # nonlocal只能在封装函数中使用，在外部函数先进行声明，在内部函数进行nonlocal声明，这样在go()函数中的pos与factory()中的pos是同一个变量
          nonlocal pos            
          new_pos = pos + step
          pos = new_pos
          return new_pos
      return go
  
  tourist = factory(origin)
  
  print(tourist(2))        # 2
  print(origin)            # 0
  print(tourist.__closure__[0].cell_contents)   # 2
  print(tourist(3))        # 5
  print(origin)						 # 0
  print(tourist.__closure__[0].cell_contents)   # 5
  print(tourist(5))        # 10
  print(origin)						 # 0
  print(tourist.__closure__[0].cell_contents)   # 10
  ```

# 函数、高阶函数、装饰器

* 匿名函数

  ```python
  def add(x, y):
      return x + y
  
  print(add(1, 2))
  
  f = lambda x, y: x + y
  print(f(1,2))
  ```

* 三元表达式

  ```python
  x = 2
  y = 1
  r = x if x > y else y
  print(r)    # 2
  ```

* map

  ```python
  list_x = [1,2,3,4,5,6,7,8]
  
  def square(x):
      return x * x
  
  r = map(square, list_x)
  print(list(r))     # [1, 4, 9, 16, 25, 36, 49, 64]
  ```

  ```python
  list_x = [1,2,3,4,5,6,7,8]
  list_y = [1,2,3,4,5,6,7,8]
  
  r = map(lambda x: x * x, list_x)
  print(list(r))      # [1, 4, 9, 16, 25, 36, 49, 64]
  
  # 结果长度取决于较少长度的参数个数
  r = map(lambda x,y: x * x + y, list_x, list_y)
  print(list(r))
  ```

* reduce

  ```python
  from functools import reduce
  
  list_x = [1,2,3,4,5,6,7,8]
  # 连续计算，连续调用lambda，结果作为下一次调用参数
  r = reduce(lambda x, y: x + y, list_x)
  # 10 为初始值
  # r = reduce(lambda x, y: x + y, list_x, 10)     # 46
  print(r)                # 36
  ```

* filter

  ```python
  list_x = [1,0,1,0,1,0]
  
  r = filter(lambda x: True if x == 1 else False, list_x)
  print(list(r))
  
  r1 = filter(lambda x: x != 0 , list_x)
  print(list(r1))
  ```

* 命令式编程

  ```python
  # map reduce filter lambda
  ```

* 装饰器

  ```python
  import time
  
  def decorator(func):
      def wrapper():
          print(time.time())
          func()
      return wrapper    
  
  def f1():
      print('This is a function')
  
  f = decorator(f1)
  f()          # 1594282980.5679739 This is a function
  ```

  ```python
  import time
  
  def decorator(func):
      def wrapper():
          print(time.time())
          func()
      return wrapper    
  
  # 装饰器优势（语法糖）  
  @decorator
  def f1():
      print('This is a function')
  
  f1()       # 1594282980.5679739 This is a function
  ```

  ```python 
  import time
  
  def decorator(func):
      def wrapper(func_name):
          print(time.time())
          func(func_name)
      return wrapper    
  
  @decorator
  def f1(func_name):
      print('This is a function:' + func_name)
  
  f1('test func')     # 1594283079.4218822   This is a function test func
  ```

  ```python 
  import time
  
  def decorator(func):
      # 可变参数，更通用
      def wrapper(*args):
          print(time.time())
          func(*args)
      return wrapper    
  
  @decorator
  def f1(func_name):
      print('This is a function:' + func_name)
  
  @decorator
  def f2(func_name1, func_name2):
      print('This is function: ' + func_name1 + ' : '  +  func_name2)
  
  f1('test func')
  f2('test func1', 'test func2')
  ```

  ```python
  import time
  
  def decorator(func):
      def wrapper(*args, **kw):
          print(time.time())
          func(*args, **kw)
      return wrapper    
  
  @decorator
  def f1(func_name):
      print('This is a function:' + func_name)
  
  @decorator
  def f2(func_name1, func_name2):
      print('This is function: ' + func_name1 + ' : '  +  func_name2)
  
  @decorator
  def f3(func_name1, func_name2, **kw):
      print('This is function: ' + func_name1 + ' : '  +  func_name2)
      print(kw)                # {'a': 1, 'b': 2, 'c': '123'}
  
  f1('test func')
  f2('test func1', 'test func2')
  f3('test func1', 'test func2', a = 1, b = 2, c = '123') 
  ```

# 实战：原生爬虫

```python
 # 对html进行文本分析，提取需要的信息
```

![image-20200617172329244](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200617172329244.png)

![image-20200617172519248](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200617172519248.png)

```python
'''
    This is Spider test
'''

import re
from urllib import request
import ssl

# 断点调试
# BeautifulSoup, Scrapy
# 爬虫、反爬虫、反反爬虫
# ip 封
# 代理ip库
ssl._create_default_https_context = ssl._create_unverified_context

class Spider():

    url = 'https://www.huya.com/g/lol'

    # 匹配所有字符 多次 非贪婪 组
    list_pattern = '<li class="game-live-item" gid="1">([\s\S]*?)</li>'
    name_pattern = '<i class="nick" title="([\s\S]*?)">[\s\S]*?</i>'
    number_patter = '<i class="js-num">([\s\S]*?)</i>'

    def __fetch_content(self):
        r = request.urlopen(Spider.url)
        htmls = r.read()
        htmls = str(htmls, encoding='utf-8')
        print('__fetch_content')
        print(htmls)
        return htmls

    def __analysis(self, htmls):
        list_html = re.findall(Spider.list_pattern, htmls)
        # print(root_html[0])
        anchors = []
        for html in list_html :
            name = re.findall(Spider.name_pattern, html)
            number = re.findall(Spider.number_patter, html)
            anchor = {'name':name, 'number':number}
            anchors.append(anchor)
        # print(anchors[0])    
        # a = 1    
        print('__analysis')
        print(len(anchors))
        return anchors

    def __refine(self, anchors):
        l = lambda anchor: {
            'name':anchor['name'][0].strip(),
            'number':anchor['number'][0]
            }
        return map(l, anchors)

    def __sort_seed(self, anchor):
        r = re.findall('[1-9]\d*\.?\d*', anchor['number'])
        number = float(r[0])
        print(number)
        if '万' in anchor['number']:
            number *= 10000
        return number    

    def __sort(self, anchors):
        # filter
        anchors = sorted(anchors, key = self.__sort_seed, reverse=True)
        return anchors

    def __show(self, anchors):
        print('__show')
        print(len(anchors))
        for rank in range(0, len(anchors)):
            print('rank' + str(rank + 1)
            + ' : ' + anchors[rank]['name']
            + '  ' + anchors[rank]['number'])

    def go(self):
        htmls = self.__fetch_content()
        anchors = self.__analysis(htmls)
        anchors = list(self.__refine(anchors))
        anchors = self.__sort(anchors)
        self.__show(anchors)
        # print(anchors)

spider = Spider()
spider.go()        
```

# 杂记

* 用字典映射代替switch case语句

  ```python
  day = 4
  
  switcher = {
      0:'a',
      1:'b',
      2:'c',
  }
  day_name = switcher.get(day, 'Unknow')
  print(day_name)
  ```

  ```python
  day = 8
  
  def get_sunday() :
      return 'Sunday'
  
  def get_monday() :
      return 'Monday'
  
  def get_tuesday() :
      return 'Tuesday'
  
  def get_default() :
      return 'Unknow'
  
  switcher = {
      0 : get_sunday,
      1 : get_monday,
      2 : get_tuesday
  }    
  
  day_name = switcher.get(day, get_default)()
  print(day_name)
  ```

* 列表推导式

  ```python
  # 列表推导式
  # 集合推导式
  # map filter
  # set、dict、tuple 也可以被推导
  a = [1,2,3,4,5,6,7,8]
  b = [i**2 for i in a if i >= 5]    # [25, 36, 49, 64]
  print(b)
  
  aa = {1,2,3,4,5,6,7,8}
  bb = {i**2 for i in a if i >= 5}    # {64, 25, 36, 49}
  print(bb)
  
  students = {
      '喜小乐' : 18,
      '石敢当' : 20,
      '哼小五' : 15
  }
  c = {key for key,value in students.items()}          # {'石敢当', '喜小乐', '哼小五'}
  cc = {value : key for key,value in students.items()} # {18: '喜小乐', 20: '石敢当', 15: '哼小五'}
  d = (key for key,value in students.items())
  dd = [key for key,value in students.items()]         # ['喜小乐', '石敢当', '哼小五']
  print(c)
  print(cc)
  print(dd)
  for x in d :
      print(x)   # 喜小乐 石敢当 哼小五
  ```

* iterator 和 generator

  ```python
  # 列表、元组、集合
  # 可迭代对象、迭代器
  # for in iterable iterator
  # 对象 class
  
  class Book:
      pass
  
  class BookCollection:
      def __init__(self):
          self.data = ['《往事》','《只能》', '《回味》']
          self.cur = 0
  
      def __iter__(self):
          return self
  
      def __next__(self):
          if self.cur >= len(self.data):
              raise StopIteration()
          r = self.data[self.cur]
          self.cur += 1
          return r
  
  books = BookCollection()
  import copy
  books_copy = copy.copy(books)        # 浅拷贝
  books_deep_copy = copy.deepcopy(books)  # 深拷贝
  # print(next(books))     # 《往事》
  # print(next(books))     # 《只能》
  # print(next(books))     # 《回味》
  for book in books:      # 一次性，第二次无结果
      print(book)        # 《往事》 《只能》 《回味》
  ```

  ```python
  # 生成器
  
  # print 0~10000      # 0-10000直接存储在内存，耗内存
  n = [i for i in range(0, 10001)]
  # print(n)
  
  # 生成器
  nn = (i for i in range(0, 10001))
  print(nn)                  # <generator object <genexpr> at 0x107ed9c80>
  
  
  # 实时计算出结果，没保存任何数据
  def gen(max):
      n = 0
      while n <= max:
          # print(n)
          n += 1
          yield n        # 多次返回，不终止
  
  g = gen(10000)
  print(next(g))
   for i in g :
     print(i)
  ```

* None

  ```python
  if a:
  if not a:
    
  a = None
  a = ''
  a = []
  a = False
  ```

  ```python
  a = ''
  b = False
  c = []
  
  print(a == None)      # False
  print(b == None)      # False
  print(c == None)      # False
  
  class Test():
    # pass                  # True
    def __len__(self):
      return 0              # 0为False，非0为True，对象存在不一定是True
    def __bool__(self):
      return False          # False
  
  test = Test()
  if test:
    print('S')
  else:
    print('F')       # F
    
  print(bool(None))  
  print(bool([]))  
  print(bool(test))  
  
  
  class Test():
      def __bool__(self):
          print('bool called')
          return False
      def __len__(self):
          print('len called')
          return 1
  
  print(len(Test()))        # 调用Test的__len__ ，1
  print(bool(Test()))       # 有__bool__则调用__bool__，否则调用__len__， False
  ```

* 装饰器的副作用

  ```python
  import time
  from functools import wraps
  
  def decorator(func):
      @wraps(func)       # 保持函数名不变，否则函数名会改变成 wrapper
      def wrapper():
          print(time.time())
          func()
      return wrapper
  
  @decorator
  def f1():
      '''
          This is f1
      '''
      print(f1.__name__)   
  f1()            # f1
  ```

* 海象运算符

  ```python
  a = 'python'
  # 海象运算符 可以省略  b = len(a)
  if (b:=len(a)) > 5:
      print('长度大于5;' + '真实长度为' + str(b))   # 长度大于5;真实长度为6
  ```

* f关键字做字符串拼接

  ```python
  a = 'python'
  print(f'hello {a}')      # hello python
  ```

* 数据类

  ```python
  from dataclasses import dataclass
  
  @dataclass
  class Student():
  
      # 自动生成构造函数
      name: str
      age: int
      school_name: str
  
      # def __init__(self, name, age, school_name):
      #     self.name = name
      #     self.age = age
      #     self.school_name = school_name
  
      def test(self):
          print(self.name)
  
  stu = Student('zs', 19, 'Tsinghua')
  print(stu.__repr__())
  stu.test()
  ```



