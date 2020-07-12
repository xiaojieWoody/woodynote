# 数据结构与算法相关问题与解决技巧

## 如何在列表, 字典, 集合中根据条件筛选数据

```shell
# 实际案例
过滤掉列表[3,9,-1,10,20,-2...]中的负数
筛选出字典{'LiLei':79, 'Jim':88, 'Lucy':92...}中值高于90的项
筛选出集合{77, 89, 32, 20...}中能被3整除的元素
# 解决方案
列表：
	列表解析:[x for x in data if x >= 0]
	filter函数:filter(lambda x:x>=0, data)
字典：
	字典解析:{k:v for k,v in d.items() if v > 90}
集合：
	集合解析:{x for x in s if x % 3 == 0}
```

```python
from random import randint

# 推荐
# _ 为变量名
l = [randint(-10, 10) for _ in range(10)]
print(l)
z = [x for x in l if x >= 0] 
print(z)

g = filter(lambda x: x >= 0, l)
# print(next(g))
# print(type(g))
print(list(g))

stu = {'student%d' % i: randint(58, 100) for i in range(1, 21)}
print(stu)
stu_f = {k:v for k, v in stu.items() if v >= 90}
print(stu_f)
g_f = filter(lambda item: item[1] >= 90, stu.items())
# print(list(g_f))
print(dict(g_f))

s = {randint(0,20) for _ in range(20)}
print(s)
s_r = {x for x in s if x % 3 == 0}
print(s_r)
```

## 如何为元组中的每个元素命名, 提高程序可读性

```shell
# 学生信息系统中数据为固定格式:(名字，年龄，性别，邮箱)
('Jim', 16, 'male', 'jim8721@gmail.com')
('LiLei', 17, 'male', 'leile@qq.com')
('Lucy', 16, 'female', 'lucy123@yahoo.com')
# 访问时，使用索引（index）访问，大量索引降低程序可读性
# 解决方案
方案1:
	定义一系列数值常量或枚举类型
方案2:
	使用标准库中collecitions.namedtuple替代内置tuple
```

```python
# 定义数值常量
NAME, AGE, SEX, EMAIL = range(4)

from enum import IntEnum
class StudentEnum(IntEnum):
    NAME = 0
    AGE = 1
    SEX = 2
    EMAIL = 3

s = ('Jim', 16, 'male', 'jim873@gmail.com')
t = (34, 'woody')

StudentEnum.NAME
print(s[StudentEnum.NAME])     # Jim
print(isinstance(StudentEnum.NAME, int))   # True
print(StudentEnum.NAME == 1)  # False
print(StudentEnum.NAME == 0)  # True


# 命名元组
from collections import namedtuple
Student = namedtuple('Student', ['name', 'age', 'sex', 'email'])
s2 = Student('Jim', 16, 'male', 'jim873@gmail.com')
print(s2)
print(isinstance(s2, tuple)) # True
print(s2[0])      # Jim
print(s2.name)    # Jim
```

## 如何根据字典中值的大小, 对字典中的项排序

```shell
# 某班英语成绩以字典形式存储为：
{
	'LiLei':79,
	'Jim': 88,
	'Lucy':92,
	...
}
# 如何根据成绩高低，计算学生排名
# 解决方案
将字典中的各项转换为元组，使用内置函数sorted排序
方案1:
	将字典中的项转化为（值，键）元组。（列表解析或zip）
方案2:
	传递sorted函数的key参数
```

```python
print((3, 2) > (2, 1))   # True
print((3, 2) > (3, 4))   # False

from random import randint
d = {k: randint(60, 100) for k in 'abcdefgh'}
# {'a': 82, 'b': 95, 'c': 74, 'd': 94, 'e': 86, 'f': 62, 'g': 64}
print(d)
l = [(v, k) for k, v in d.items()]
# [(82, 'a'), (95, 'b'), (74, 'c'), (94, 'd'), (86, 'e'), (62, 'f'), (64, 'g')]
print(l)
# [(62, 'f'), (64, 'g'), (74, 'c'), (82, 'a'), (86, 'e'), (94, 'd'), (95, 'b')]
print(sorted(l))
print(sorted(l, reverse=True))

t = list(zip([1,2,3], [4,5,6]))            # [(1, 4), (2, 5), (3, 6)]
ll = list(zip(d.values(), d.keys()))
print(ll)


l_t = d.items()
print(l_t)  # dict_items([('a', 73), ('b', 93), ('c', 64), ('d', 73), ('e', 63), ('f', 92), ('g', 83), ('h', 69)])

lt = sorted(d.items(), key=lambda item: item[1], reverse=True)
print(lt)   # [('b', 93), ('f', 92), ('g', 83), ('a', 73), ('d', 73), ('h', 69), ('c', 64), ('e', 63)]

print(enumerate(lt))
print(list(enumerate(lt)))  # [(0, ('a', 94)), (1, ('g', 90)), (2, ('c', 80)), (3, ('h', 76)), (4, ('f', 74)), (5, ('e', 73)), (6, ('b', 70)), (7, ('d', 60))]
print(list(enumerate(lt, 1))) # [(1, ('a', 94)), (2, ('g', 90)), (3, ('c', 80)), (4, ('h', 76)), (5, ('f', 74)), (6, ('e', 73)), (7, ('b', 70)), (8, ('d', 60))]

for i, (k, v ) in enumerate(lt, 1) :
    print(i, k, v)
    d[k] = (i,v)
print(d)    # {'a': (4, 82), 'b': (1, 95), 'c': (8, 68), 'd': (2, 92), 'e': (7, 74), 'f': (3, 90), 'g': (5, 78), 'h': (6, 76)}

dd = {k:(i,v) for i, (k, v ) in enumerate(lt, 1)}
print(dd)   # {'b': (1, 95), 'd': (2, 92), 'f': (3, 90), 'a': (4, 82), 'g': (5, 78), 'h': (6, 76), 'e': (7, 74), 'c': (8, 68)}
```

## 如何统计序列中元素的频度

```shell
1. 某随机序列[12,5,6,4,6,5,5,7]中，找到出现次数最高的3个元素，它们出现次数是多少？
2. 对某英文文章的单词，进行词频统计，找到出现次数最高的10个单词，他们出现次数是多少？
# 解决方案
方案1:
	将序列转换为字典{元素:频度}，根据字典中的值排序
方案2:
	使用标准库collections中的Counter对象
```

```python
from random import randint
data = [randint(0, 20) for _ in range(10)]
# [0, 2, 10, 14, 13, 14, 18, 18, 7, 7]
print(data)
d = dict.fromkeys(data, 0)
# {0: 0, 2: 0, 10: 0, 14: 0, 13: 0, 18: 0, 7: 0}
print(d)
for x in data:
    d[x] += 1

# {0: 1, 2: 1, 10: 1, 14: 2, 13: 1, 18: 2, 7: 2}
print(d)
dd = sorted([(v, k) for k, v in d.items()], reverse = True)
dd2 = sorted([(v, k) for k, v in d.items()], reverse=True)[:3]
dd3 = sorted(((v, k) for k, v in d.items()), reverse=True)[:3]
# [(2, 18), (2, 14), (2, 7), (1, 13), (1, 10), (1, 2), (1, 0)]
print(dd)
# [(2, 18), (2, 14), (2, 7)]
print(dd2)
# [(2, 18), (2, 14), (2, 7)]
print(dd3)
import heapq
# 推荐
heapq.nlargest(3, ((v, k) for k, v in d.items()))

from collections import Counter
c = Counter(data)
cr = c.most_common(3)
print(cr)


txt = open('example.txt').read()
import re
word_list = re.split('\W+', txt)
c2 = Counter(word_list)
res = c2.most_common(10)
```

## 如何快速找到多个字典中的公共键(key)

```shell
西班牙足球甲级联赛，每轮球员进球统计：
第1轮：{'苏亚雷斯':1, '梅西':2, '本泽马':1,...}
第2轮：{'苏亚雷斯':2, 'C罗':1, '格里兹曼':2,...}
第3轮：{'苏亚雷斯':1, '托雷斯':1, '贝尔':1,...}
...
统计出前N轮，每场比赛都有进球的球员
# 解决方案
利用集合(set)的交集操作
Step1:使用字典的keys()方法，得到一个字典keys的集合
Step2:使用map函数，得到每个字典keys的集合
Step3:使用reduce函数，取所有字典的keys集合的交集
```

```python
from random import randint, sample
print(sample('abcdefgh', 3))     # ['a', 'd', 'h']
print(sample('abcdefgh', randint(3, 6)))    # ['f', 'c', 'd']
d1 = {k: randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
d2 = {k: randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
d3 = {k: randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
print(d1)   # {'f': 4, 'g': 1, 'c': 2, 'h': 4, 'a': 1, 'e': 2}
print(d2)   # {'g': 4, 'b': 1, 'e': 3, 'h': 3, 'f': 3, 'd': 4}
print(d3)   # {'b': 2, 'd': 3, 'h': 1, 'e': 4, 'a': 4, 'f': 1}
for k in d1:
    if k in d2 and k in d3:
        print(k)      # f h e
d1 = [d1, d2, d3]        
dres = [k for k in d1[0] if all(map(lambda d: k in d, d1[1:]))]
print(dres)           # ['f', 'h', 'e']
```

```python
from functools import reduce
# 1到10的阶乘  
# a 变成前面的结果，b从range中取
print(reduce(lambda a, b: a * b, range(1, 11)))     # 3628800

d1 = {k: randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
d2 = {k: randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
d3 = {k: randint(1, 4) for k in sample('abcdefgh', randint(3, 6))}
print(d1)    # {'f': 3, 'e': 4, 'b': 1, 'h': 2, 'a': 1, 'c': 4}
print(d2)    # {'a': 1, 'e': 4, 'c': 2, 'b': 4, 'f': 3, 'h': 3}
print(d3)    # {'h': 2, 'b': 2, 'd': 4, 'a': 3}
s1 = d1.keys()
s2 = d2.keys()
print(s1)       # dict_keys(['f', 'e', 'b', 'h', 'a', 'c'])
print(s2)       # dict_keys(['a', 'e', 'c', 'b', 'f', 'h'])
print(s1 & s2)  # {'c', 'a', 'e', 'b', 'f', 'h'}

d1 = [d1, d2, d3] 
dr = reduce(lambda a, b: a & b, map(dict.keys, d1))
print(dr)        # {'h', 'b', 'a'}
```

## 如何让字典保持有序

```shell
某编程竞赛系统，对参赛选手编程解题进行计时，选手完成题目后，把该选手解题用时记录到字典中，以便赛后按选手名查询成绩
{'LiLei':(2,43), 'HanMeimei':(5,52), 'Jim':(1, 39)...}
比赛结束后，需按排名顺序依次打印选手成绩，如何实现？
# 解决方案
使用标准库collections中的OrderedDict
以OrderedDict替代内置字典Dict，依次将选手成绩存入OrderedDict
```

```python
from collections import OrderedDict
od = OrderedDict()
od['c'] = 1
od['b'] = 2
od['a'] = 3
print(od.keys())   # odict_keys(['c', 'b', 'a'])

players = list('abcdefgh')
from random import shuffle
shuffle(players)
print(players)  # ['h', 'g', 'd', 'e', 'c', 'b', 'a', 'f']

od = OrderedDict()
for i, p in enumerate(players, 1):
    od[p] = i
print(od)       # OrderedDict([('h', 1), ('g', 2), ('d', 3), ('e', 4), ('c', 5), ('b', 6), ('a', 7), ('f', 8)])

def query_by_name(d, name):
    return d[name]

print(query_by_name(od, 'c'))  # 5
print(query_by_name(od, 'd'))  # 3


from itertools import islice
print(islice(range(10), 3, 6))  # 只取 3-6

def query_by_order(d, a, b=None):
    a -= 1
    if b is None:
        b = a + 1
    return list(islice(od, a, b))        

print(od)   # OrderedDict([('f', 1), ('e', 2), ('a', 3), ('g', 4), ('d', 5), ('c', 6), ('b', 7), ('h', 8)])
print(query_by_order(od, 4)) # ['g']
print(query_by_order(od, 3, 6))  # ['h', 'g', 'e', 'a']
```

## 如何实现用户的历史记录功能(最多n条)

```shell
很多应用程序都有浏览用户的历史记录的功能，例如
1. 浏览器可以查看最近访问过的网页
2. 视频播放器可以查看最近播放过的视频文件
3. Shell可以查看用户输入过的命令
现在制作一个简单的猜数字的小游戏，如何添加历史记录功能，显示用户最近猜过的数字？
# 解决方案
使用容量为n的队列存储历史记录
使用标准库collections中的deque，它是一个双端循环队列
使用pickle模块将历史记录存储到硬盘，以便下次启动使用
```

```python
from random import randint

def guess(n, k):
    if n == k:
        print('猜对了, 这个数字是%d.' % k)
        return True

    if n < k:
        print('猜大了, 比%d小.' % k)        
    elif n > k:
        print('猜小了, 比%d大.' % k)
    return False        

def main():
    n = randint(1, 100)
    i = 1
    while True:
        line = input('[%d] 请输入一个数字：' % i)
        if line.isdigit():
            k = int(line)
            i += 1
            if guess(n, k):
                break
            elif line == 'quit':
                break

if __name__ == '__main__':
    main()
```

```python
from collections import deque
q = deque([], 5)
q.append(1)
q.append(2)
q.append(3)
q.append(4)
q.append(5)
print(q)     # deque([1, 2, 3, 4, 5], maxlen=5)
q.append(6)
print(q)     # deque([2, 3, 4, 5, 6], maxlen=5)
```

```python
import pickle
pickle.dump
q
pickle.dumps(q, open('save.pkl', 'wb'))
# 查看文件
# ed save.pkl
pickle.load(open('save.pkl', 'rb'))
```

# 复杂场景下字符串处理相关问题与解决技巧

## 如何拆分含有多种分隔符的字符串

```shell
要把某个字符串依据分隔符号拆分不同的字段，该字符串包含多种不同的分隔符，例如：
s = 'ab;cd|efg|hi,jkl|mn\topq;rst,uvw\txyz'
其中<,>,<;>,</>,<\t>都是分隔符号，如何处理？
# 解决方案
方法1:连续使用是str.split()方法，每次处理一种分隔符号
方法2:使用正则表达式的re.split()方法(推荐)
```

```python
s = 'ab;cd|efg|hi,jkl|mn\topq;rst,uvw\txyz'

def my_split(s, seps):
    res = [s]
    for sep in seps:
        t = []
        list(map(lambda ss: t.extend(ss.split(sep)), res))
        res = t
    return res

result = my_split(s, ',;|\t')
# ['ab', 'cd', 'efg', 'hi', 'jkl', 'mn', 'opq', 'rst', 'uvw', 'xyz']
print(result)

from functools import reduce
result2 = reduce(lambda l, sep:sum(map(lambda ss: ss.split(sep), l), []), ',;|\t', [s])
# ['ab', 'cd', 'efg', 'hi', 'jkl', 'mn', 'opq', 'rst', 'uvw', 'xyz']
print(result2)
my_split2 = lambda s, seps: reduce(lambda l, sep: sum(map(lambda ss: ss.split(sep), l), []), seps, [s])
result3 = my_split2(s, ';,|\t')
# ['ab', 'cd', 'efg', 'hi', 'jkl', 'mn', 'opq', 'rst', 'uvw', 'xyz']
print(result3)

import re
result4 = re.split('[;,|\t]+', s)
# ['ab', 'cd', 'efg', 'hi', 'jkl', 'mn', 'opq', 'rst', 'uvw', 'xyz']
print(result4)
```

## 如何判断字符串a是否以字符串b开头或结尾

```python
某文件系统目录下有一系列文件：
	quicksort.c
	graph.py
  heap.java
  install.sh
  stack.cpp
  ...
编写程序给其中所有.sh文件和.py文件加上用户可执行权限  
# 解决方案
使用str.startswith()和str.endswith()方法
(注意：多个匹配时参数使用元组)
```

```python
fn = 'aaa.py'
fn.endswith('.py')
fn.endswith(('.py', '.sh'))

import os
d = os.listdir('.')       # 当前目录
s = os.stat('b.py')          # 文件状态信息
# oct(s.st_mode)            # 八进制
# ls -l b.py
oct(s.st_mode | 0o100)     # 赋可执行权限

os.chmod('b.py', s.st_mode | 0o100)    # 赋可执行权限

import stat
stat.S_IXUSR

for fn in os.listdir():
    if fn.endswith(('.py', '.sh')):
        fs = os.stat(fn)
        os.chmod(fn, fs.st_mode | stat.S_IXUSR)
# ll  
```

## 如何调整字符串中文本的格式

```shell
某软件的log文件，其中的日期格式为'yyyy-mm-dd':
	2016-05-21 10:39:26 status unpacked python3-pip:all
	2016-05-23 10:49:26 status half-configured python3
	2016-05-23 10:52:26 status installed python3-pip:all
	2016-05-24 11:57:26 configure python3-wheel:all 0.24
	...
想把其中日期改为美国日期格式'mm/dd/yyyy'
'2016-05-23' => '05/23/2016',应如何处理？
# 解决方案
使用正则表达式re.sub()方法做字符串替换，利用正则表达式的捕获组，捕获每个部分内容，在替换字符串中调整各个捕获组的顺序
```

![image-20200705203342424](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705203342424.png)

```python
# f = open('/var/log/dpkg.log.1')
# log = f.read()
log = '2020-07-05 aqeeadfv sfd adf sfs'
print(log)
import re
# 07/05/2020 aqeeadfv sfd adf sfs
print(re.sub(r'(\d{4})-(\d{2})-(\d{2})', r'\2/\3/\1', log))
# 07/2020/05 aqeeadfv sfd adf sfs
print(re.sub(r'(?P<d>\d{4})-(?P<m>\d{2})-(?P<y>\d{2})', r'\g<m>/\g<d>/\g<y>', log))
```

## 如何将多个小字符串拼接成一个大的字符串

```python
在设计某网络程序时，我们自定义了一个基于UDP的网络协议，按照固定次序向服务器传递一系列参数：
hwDetect:			"<0112>"
gxDepthBits:	"<32>"
gxResolution:	"<1024x768>"
gxRefresh:		"<60>"
fullAlpha:		"<1>"
lodDist:			"<100.0>"  
DistCull:			"<500.0>"
在程序中我们将各个参数按次序收集到列表中：
["<0112>","<32>","<1024x768>","<60>","<1>","<100.0>","<500.9>"]
最终我们要把各个参数拼接成一个数据报进行发送
"<0112><32><1024x768><60><1><100.0><500.0>"
# 解决方案
方法一：迭代列表，连续使用'+'操作依次拼接每一个字符串
方法二：使用str.join()方法，更加快速的拼接列表中所有字符串
```

```python
l = ["<0112>", "<32>", "<1024x768>", "<60>", "<1>", "<100.0>", "500.0"]
s = ''
for x in l:
    print(s)      # 空间浪费
    s += x

print(s)     # <0112><32><1024x768><60><1><100.0>500.0
from functools import reduce
res = reduce(str.__add__, l)
print(res)   # <0112><32><1024x768><60><1><100.0>500.0

# str.join?
l2 = ['abc', 'efx', '1234']
# 推荐
''.join(l2)

timeit ''.join(l2)
timeit reduct(str.__add__, l)
```

## 如何对字符串进行左, 右, 居中对齐

```python
某个字典存储了一系列属性值
{
  "lodDist":100.0,
  "SmallCull":0.04,
  "DistCull":500.0,
  "trilinear":40,
  "farclip":477
}
在程序中，想以工整的格式将其内容输出，如何处理？
"lodDist"		:100.0,
"SmallCull"	:0.04,
"DistCull"	:500.0,
"trilinear"	:40,
"farclip"		:477
# 解决方案
方法一：使用字符串的str.ljust(), str.rjust(), str.center()进行左、右、居中对齐
方法二：使用format()方法，传递类似'<20','>20','^20'参数完成同样任务
```

```python
s = 'abc'
print(s.ljust(10))
print(len(s.ljust(10))) # 10
print(s.ljust(2))
print(s.ljust(10, '*'))
print(s.ljust(10, '^'))
print(s.rjust(10, '*'))
print(s.center(10, '*'))

print(format(s, '<10'))
print(format(s, '>10'))  #        abc
print(format(s, '^10'))
print(format(s, '*^10')) # ***abc****

print(format(123, '+'))       # +123
print(format(-123, '+'))      # -123
print(format(-123, '>+10'))   #       -123
print(format(-123, '=+10'))   # -      123
print(format(-123, '0=+10'))  # -000000123
print(format(123, '0=+10'))   # +000000123

d = {'lodDist': 100.0, 'SmallCull': 0.04, 'DistCull': 500.0, 'trilinear': 40, 'farclip': 477}
w = max(map(len, d.keys()))
print(w)  # 9
for k, v in d.items():
    print(k.ljust(w), ':', v)
    
lodDist   : 100.0
SmallCull : 0.04
DistCull  : 500.0
trilinear : 40
farclip   : 477    
```

## 如何去掉字符串中不需要的字符

```python
1. 过滤掉用户输入中前后多余的空白字符：
'  nick2008@gmail.com  '
2. 过滤某windows下编辑文本中的'\r':
  'hello world\r\n'
3. 去掉文本中的unicode组合符号(音调):
  'ni hao, chi fan'

# 解决方案
方法一：字符串strip(),lstrip(),rstrip()方法去掉字符串两端字符
方法二：删除单个固定位置的字符，可以使用切片 + 拼接的方式
方法三：字符串的replace()方法或正则表达式re.sub()删除任意子串
方法四：字符串的translate()方法，可以同时删除多种不同字符
```

![image-20200705212115700](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705212115700.png)

```python
s = '    woody@gmail.com '
print(s.lstrip())
print(s.rstrip())
print(s.strip())
s = '===1231==='
print(s.strip('='))
s = '==-=+=12331=-+++='   
print(s.strip('=+-'))     # 12331

s = 'abc:1234'
print(s[:3] + s[4:])    # abc1234

s = '    abc  xyz   '
print(s.strip())          # abc  xyz
print(s.replace(' ', '')) # abcxyz

s = '  \t  abc \t xyz \n  '
print(s.replace(' ', ''))
import re
print(re.sub('[ \t\n]+', '', s))     # abcxyz
print(re.sub('\s+', '', s))     # abcxyz

s = 'abc1234xyz'
print(ord('a'))     # 97
print(s.translate({ord('a'): 'X'}))  # Xbc1234xyz
print(s.translate({ord('a'): 'X', ord('b'): 'Y'}))  # XYc1234xyz
print(s.maketrans('abcxyz', 'ABCXYZ'))  # {97: 65, 98: 66, 99: 67, 120: 88, 121: 89, 122: 90}
print(s.translate(s.maketrans('abcxyz', 'ABCXYZ')))  # ABC1234XYZ
print(s.translate({ord('a'): None}))  # bc1234xyz
```

# 对象迭代与反迭代相关问题与解决技巧

## 如何实现可迭代对象和迭代器对象

![image-20200705222224968](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705222224968.png)

![image-20200705222740638](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705222740638.png)

```python
# 迭代器之间互不干扰
it = iter(l)
print(next(it))

url = 'http://wthrcdn.etouch.cn/weather_mini?city=' + '北京'
import requests
r = requests.get(url)
print(r.text)
print(r.json())
print(r.json()['data']['city'])  # 北京
print(r.json()['data']['forecast'][0])  # {'date': '5日星期天', 'high': '高温 30℃', 'fengli': '<![CDATA[2级]]>', 'low': '低温 23℃', 'fengxiang': '北风', 'type': '雷阵雨'} 

from collections.abc import Iterable, Iterator
import requests

class WeatherIterator(Iterator):
    def __init__(self, caties):
        self.caties = caties
        self.index = 0

    def __next__(self):
        if self.index == len(self.caties):
            raise StopIteration
        city = self.caties[self.index]
        self.index += 1
        return self.get_weather(city)

    def get_weather(self, city):
        url = 'http://wthrcdn.etouch.cn/weather_mini?city=' + city
        r = requests.get(url)
        data = r.json()['data']['forecast'][0]
        return city, data['high'], data['low']

class WeatherIterable(Iterable):
    def __init__(self, cities):
        self.cities = cities

    def __iter__(self):
        return WeatherIterator(self.cities)

def show(w):
    for x in w:
        print(x)

w = WeatherIterable(['北京','上海', '广州'] * 3)     # 可以迭代多次，每次都创建一个迭代对象
#  w = WeatherIterator(['北京','上海', '广州'] * 3)  # 只能迭代一次         
show(w)
# ('北京', '高温 30℃', '低温 23℃')
# ('上海', '高温 26℃', '低温 24℃')
# ('广州', '高温 32℃', '低温 27℃')
# ('北京', '高温 30℃', '低温 23℃')
```

## 如何使用生成器函数实现可迭代对象

![image-20200705224537022](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705224537022.png)

![image-20200705224558302](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705224558302.png)

![image-20200705224738771](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705224738771.png)

![image-20200705224814070](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705224814070.png)

```python
from collections.abc import Iterable

class PrimeNumbers(Iterable):
    def __init__(self, a, b):
        self.a = a
        self.b = b

    def __iter__(self):
        for k in range(self.a, self.b + 1):
            if self.is_prime(k):
                yield k

    # 判断是否为素数
    def is_prime(self, k):
        return False if k < 2 else all(map(lambda x: k % x, range(2, k)))    

pn = PrimeNumbers(1,30)
for n in pn:
    print(n)
```

## 如何进行反向迭代以及如何实现反向迭代

![image-20200705225707391](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705225707391.png)

![image-20200705230242002](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705230242002.png)

![image-20200705230635050](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705230635050.png)

```python
l = [1,2,3,4,5]
l.reverse()       # 改变了原数据
print(l)          # [5, 4, 3, 2, 1]
l = [1,2,3,4,5]
print(l[::-1])    # [5, 4, 3, 2, 1]      # 复制一份进行操作

# 反向迭代器
reversed(l)           # l.__reversed__
# 正向迭代器
iter(l)
for x in reversed(l):
    print(x)


from decimal import Decimal

class FloatRange:
    def __init__(self, a, b, step):
        # self.a = a
        self.a = Decimal(str(a))
        self.b = Decimal(str(b))
        self.step = Decimal(str(step))

    def __iter__(self):
        t = self.a
        while t <= self.b:
            yield float(t)
            t += self.step
    
    def __reversed__(self):
        t = self.b
        while t >= self.a:
            yield float(t)
            t -= self.step

fr = FloatRange(3.0, 4.0, 0.2)
for x in fr:
    print(x)
print('-' * 20)
for x in reversed(fr):
    print(x) 
```

## 如何对迭代器做切片操作

![image-20200705231256773](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705231256773.png)

![image-20200705231322378](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705231322378.png)

![image-20200705231632726](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705231632726.png)

```python
f = open('/var/log/dpkg.log.1')
from itertools import islice

for line in islice(f, 100-1, 300):      # 前100行还是会读取，但是不显示
    print(line)

def my_islice(iterable, start, end, step=1):
    tmp = 0
    for i, x in enumerate(iterable):
        if i >= end:
            break

        if i >= start:
            if tmp == 0:
                tmp = step
                yield x
            tmp -= 1

# [110, 113, 116, 119]
print(list(my_islice(range(100, 150), 10, 20, 3)))
from itertools import islice
# [110, 113, 116, 119]
print(list(islice(range(100, 150), 10, 20,3)))
```

## 如何在一个for语句中迭代多个可迭代对象

![image-20200705232610833](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705232610833.png)

![image-20200705233846844](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705233846844.png)

```python 
from random import randint
chinese = [randint(60, 100) for _ in range(20)]
math = [randint(60, 100) for _ in range(20)]
english = [randint(60, 100) for _ in range(20)]
print(chinese)
print(math)
print(english)

print(zip([1,2,3], [4,5,6]))
print(list(zip([1,2,3], [4,5,6])))          # [(1, 4), (2, 5), (3, 6)]
print(list(zip([1,2,3], [4,5,6], [7,8,9]))) # [(1, 4, 7), (2, 5, 8), (3, 6, 9)]
print(list(zip([1,2,3], [4,5,6], [7,8])))   # [(1, 4, 7), (2, 5, 8)]     # 取决于最短的那个

# t = []
# for s1, s2, s3 in zip(chinese, map, english):
#     t.append(s1 + s2 + s2)

# print(t)    

res = [sum(s) for s in zip(chinese, math, english)]
print(res)

res = list(map(sum, zip(chinese, math, english)))
print(res)

res = list(map(lambda s1, s2, s3: s1 + s2 + s3, chinese, math, english))
print(res)

from itertools import chain
for x in chain([1,2,3], [4,5,6,7], [8,9]):
    print(x)

c1 = [randint(60, 100) for _ in range(20)]    
c2 = [randint(60, 100) for _ in range(20)] 
c3 = [randint(60, 100) for _ in range(23)] 
c4 = [randint(60, 100) for _ in range(25)] 
print(c1)
print(c2)
print(c3)
print(c4)
print(len([x for x in chain(c1,c2,c3,c4) if x > 90]))     # 27

s = 'abc;123|xyz;678|fweuow\tjzka'
from functools import reduce
# ['abc', '123', 'xyz', '678', 'fweuow', 'jzka']
print(list(reduce(lambda it_s, sep: chain(*map(lambda ss: ss.split(sep), it_s)), ';|\t', [s])))
```

# 文件I/O效率相关问题与解决技巧

## 如何读写文本文件

![image-20200705234818509](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705234818509.png)

![image-20200705235023870](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705235023870.png)

```python
s = '我是woody， 我爱python'
print(type(s))   # str
f = open('b.txt', 'w', encoding='gbk')
f.write(s)
f.flush()

f = open('b.txt', encoding='gbk')
print(f.read())
```

## 如何处理二进制文件 

![image-20200705235501780](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705235501780.png)

![image-20200705235721504](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705235721504.png)

```python
f = open('demo.wav', 'rb')
info = f.read(44)
print(info)
type(info)  # bytes
info[22:24]
import struct
struct.unpack('h', info[22:24])  # (2,)
struct.unpack('i', info[24:28])  # (44100,)
info.find(b'data')  # -1
f.seek(0)      # 0
f.read(100)

import struct 

def find_subchunk(f, name):
    f.seek(12)          # 跳过12个字节
    while True:
        chunk_name = f.read(4)
        chunk_size = struct.unpack('i', f.read(4))

        if chunk_name == name :
            return f.tell(), chunk_size

        f.seek(chunk_size, 1)

offset, size = find_subchunk(f, b'data')
```

![image-20200706000727155](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706000727155.png)

## 如何设置文件的缓冲

![image-20200706195651686](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706195651686.png)

![image-20200706195817287](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706195817287.png)

![image-20200706200514292](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200514292.png)

![image-20200706200535070](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200535070.png)

![image-20200706200644692](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200644692.png)

![image-20200706200727787](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200727787.png)

![image-20200706200850814](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200850814.png)

![image-20200706200911492](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200911492.png)

![image-20200706200959522](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706200959522.png)

![image-20200706201019494](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201019494.png)

![image-20200706201109550](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201109550.png)

![image-20200706201156376](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201156376.png)

![image-20200706201238000](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201238000.png)

![image-20200706201254371](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201254371.png)

![image-20200706201333752](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201333752.png)

## 如何将文件映射到内存

![image-20200706201350767](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201350767.png)

![image-20200706201438785](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201438785.png)

![image-20200706201553850](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706201553850.png)

![image-20200706202157359](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706202157359.png)

## 如何访问文件的状态

![image-20200706202250909](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706202250909.png)

![image-20200706203248892](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706203248892.png)

![image-20200706203704752](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706203704752.png)

![image-20200706203834029](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706203834029.png)

![image-20200706203918258](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706203918258.png)

![image-20200706203933414](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706203933414.png)

![image-20200706204020244](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706204020244.png)

```python
import os
os.stat('a.txt')
fd = os.open('b.py', os.O_RDONLY)   #文件描述符，给系统调用
fd # 23
os.read(fd, 10)  
```

## 如何使用临时文件

![image-20200706212501057](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706212501057.png)

![image-20200706212529713](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706212529713.png)

```python
from tempfile import TemporaryFile, NamedTemporaryFile

tf = TemporaryFile()     # 可以指定目录
tf.write(b'*' * 1024)
tf.write(b'*' * 1024)
tf.write(b'*' * 1024)
tf.write(b'*' * 1024)

tf.seek(0)
print(tf.read(512))
tf.close()               # 删除临时文件

ntf = NamedTemporaryFile()
print(ntf.name)
ntf.close()
import tempfile
print(tempfile.gettempdir())
print(tempfile.gettempprefix())     # tmp 


import os
# os.open?
```

# 数据解析与构建相关问题与解决技巧

## 如何读写csv数据

![image-20200706213649438](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706213649438.png)

 ![image-20200706213721265](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706213721265.png)

![image-20200706214039417](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706214039417.png)

![image-20200706214113201](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706214113201.png)

![image-20200706214146791](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706214146791.png)

![image-20200706214208168](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706214208168.png)

```python
import csv

with open('books.csv') as rf:
    reader = csv.reader(rf)
    headers = next(reader)
    with open('books_out.csv', 'w') as wf:
        writer = csv.writer(wf)
        writer.writerow(headers)

        for book in reader:
            price = book[-2]
            if price and float(price) >= 80.00:
                writer.writerow(book)
```

## 如何读写json数据

![image-20200706214804328](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706214804328.png)

![image-20200706215801129](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215801129.png)

![image-20200706215316711](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215316711.png)



```python
import requests

r = requests.get('http://httpbin.org/headers')                
print(r)
print(r.content)
print(r.text)
import json
d = json.loads(r.text)          # 
print(d)
print(d['headers'])
print(d['headers']['Host'])

```

![image-20200706215346145](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215346145.png)

![image-20200706215419643](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215419643.png)

![image-20200706215636757](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215636757.png)

![image-20200706215740738](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215740738.png)

## 如何解析简单的xml文档

![image-20200706215836310](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215836310.png)

![image-20200706215853262](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215853262.png)

![image-20200706215917156](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706215917156.png)

![image-20200706220113738](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220113738.png)

![image-20200706220256992](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220256992.png)

![image-20200706220416343](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220416343.png)

![image-20200706220532599](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220532599.png)

![image-20200706220731256](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220731256.png)

## 如何构建xml文档

![image-20200706220815632](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220815632.png)

![image-20200706220840922](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706220840922.png)

![image-20200706221123741](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706221123741.png)

![image-20200706221309987](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706221309987.png)

```python
import csv
from xml.etree.ElementTree import ElementTree, Element, SubElement

def csv_to_xml(csv_path, xml_path):
    with open(csv_path) as f:
        reader = csv.reader(f)
        headers = next(reader)

        root = Element('Data')
        root.text = '\n\t'
        root.tail = '\n'

        for row in reader:
            book = SubElement(root, 'Book')
            book.text = '\n\t\t'
            book.tail = '\n\t'

            for tag, text in zip(headers, row):
                e = SubElement(book, tag)
                e.text = text
                e.tail = '\n\t\t'
            e.tail = '\n\t'

        ElementTree(root).write(xml_path, encoding='utf8')

csv_to_xml('books.csv', 'books.xml')
```

![image-20200706221825770](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706221825770.png)

## 如何读写excel文件

![image-20200706221856836](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706221856836.png)

![image-20200706221917914](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706221917914.png)

![image-20200706221932267](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706221932267.png)

![image-20200706222101359](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706222101359.png)

![image-20200706222230161](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706222230161.png)

![image-20200706222337050](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706222337050.png)

![image-20200706222430145](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706222430145.png)

![image-20200706222551887](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706222551887.png)

```python
import xlrd, xlwt

rbook = xlrd.open_workbook('demo.xlsx')
rsheet = rbook.sheet_by_index(0)

k = rsheet.ncols
rsheet.put_cell(0, k, xlrd.XL_CELL_TEXT, '总分', None)

for i in range(1, rsheet.nrows):
    t = sum(rsheet.row_values(i, 1))
    rsheet.put_cell(i, k, xlrd.XL_CELL_NUMBER, t, None)


wbook = xlwt.Workbook()
wsheet = wbook.add_sheet(rsheet.name)

for i in range(rsheet.nrows):
    for j in range(rsheet.ncols):
        wsheet.write(i, j, rsheet.cell_value(i, j))

wbook.save('out.xlsx')
```





