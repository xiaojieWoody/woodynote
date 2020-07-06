# 数据结构与算法相关问题与解决技巧

## 2-1 如何在列表, 字典, 集合中根据条件筛选数据

![image-20200705152028762](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705152028762.png)

![image-20200705152117101](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705152117101.png)

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

## 2-2 如何为元组中的每个元素命名, 提高程序可读性

![image-20200705154455250](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705154455250.png)

![image-20200705154559115](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705154559115.png)

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

## 2-3 如何根据字典中值的大小, 对字典中的项排序

![image-20200705155748375](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705155748375.png)

![image-20200705160610061](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705160610061.png)

```python
print((3, 2) > (2, 1))   # True
print((3, 2) > (3, 4))   # False

from random import randint
d = {k: randint(60, 100) for k in 'abcdefgh'}
print(d)
l = [(v, k) for k, v in d.items()]
print(l)
print(sorted(l))
print(sorted(l, reverse=True))

t = list(zip([1,2,3], [4,5,6]))
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

## 2-4 如何统计序列中元素的频度

![image-20200705161631189](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705161631189.png)

![image-20200705182453196](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705182453196.png)

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

## 2-5 如何快速找到多个字典中的公共键(key)

![image-20200705182927731](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705182927731.png)

![image-20200705183900295](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705183900295.png)

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

## 2-6 如何让字典保持有序

![image-20200705185020916](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705185020916.png)

![image-20200705185142963](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705185142963.png)

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

## 2-7 如何实现用户的历史记录功能(最多n条)

![image-20200705190248918](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705190248918.png)

![image-20200705192305973](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705192305973.png)

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

![image-20200705200505962](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705200505962.png)

![image-20200705201732358](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705201732358.png)

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

![image-20200705201903171](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705201903171.png)

![image-20200705202809458](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705202809458.png)

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

![image-20200705202849738](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705202849738.png)

![image-20200705202930737](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705202930737.png)

![image-20200705203007400](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705203007400.png)

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

![image-20200705203921391](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705203921391.png)

![image-20200705204948646](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705204948646.png)

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

![image-20200705205423052](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705205423052.png)

![image-20200705205454002](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705205454002.png)

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

![image-20200705210625306](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705210625306.png)

![image-20200705211507250](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200705211507250.png)

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

# 5.3

## 如何将文件映射到内存

## 如何访问文件的状态

## 如何使用临时文件

# 数据解析与构建相关问题与解决技巧

## 如何读写csv数据

## 如何读写json数据

## 如何解析简单的xml文档

## 如何构建xml文档

## 如何读写excel文件

# 类与对象深度问题与解决技巧

## 如何派生内置不可变类型并修其改实例化行为

## 如何为创建大量实例节省内存

## 如何让对象支持上下文管理

## 如何创建可管理的对象属性

## 如何让类支持比较操作

## 如何使用描述符对实例属性做类型检查

## 如何在环状数据结构中管理内存

## 如何通过实例方法名字的字符串调用方法

# 多线程与多进程

## 多线程并发相关问题与解决技巧

## 如何使用多线程

##  如何线程间通信

## 如何在线程间进行事件通知

## 如何使用线程本地数据

## 如何使用线程池

## 如何使用多进程

# 装饰器使用问题与技巧

## 如何使用函数装饰器

## 如何为被装饰的函数保存元数据

## 如何定义带参数的装饰器

## 如何实现属性可修改的函数装饰器

## 如何在类中定义装饰器



