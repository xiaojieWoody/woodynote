# 类与对象深度问题与解决技巧

## 如何派生内置不可变类型并修其改实例化行为

![image-20200712114021478](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712114021478.png)

![image-20200712153511843](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712153511843.png)

```python
class A:
    def __new__(cls, *args):
        print('In A.__new__', cls, args)
        # 所有类的实例的创建，默认调用父类的new，也是object的new
        return object.__new__(cls)

    def __init__(self, *args):
        print('In A.__init__', args)

# 创建一个实例时，调用init之前会调用new方法
# a = A(1, 2)     # 相当于下面这两个

# In A.__new__ <class '__main__.A'> (1, 2)
a = A.__new__(A, 1, 2)

# 把a作为参数传递给init
# In A.__init__ (1, 2)
A.__init__(a, 1, 2)
```

```python
>>> list('abc')
['a', 'b', 'c']
>>> l = list.__new__(list, 'abc')
>>> l
[]
>>> list.__init__(l, 'abc')
>>> l
['a', 'b', 'c']
# 实现了自己的__init__
>>> list.__init__ is object.__init__
False

>>> tuple('abc')
('a', 'b', 'c')
>>> t = tuple.__new__(tuple, 'abc')
>>> t
('a', 'b', 'c')
>>> tuple.__init__(t, 'abc')
>>> t
('a', 'b', 'c')
# tuple没有实现自己的__init__
>>> tuple.__init__ is object.__init__
True
```

```python
class IntTuple(tuple):
    def __new__(cls, iterable):
        # 过滤iterable
        f_it = (e for e in iterable if isinstance(e, int) and e > 0)
        return super().__new__(cls, f_it)

init = IntTuple([1, -1, 'abc', 6, ['x', 'y'], 3])
# (1, 6, 3)
print(init)
```

## 如何为创建大量实例节省内存

![image-20200712153827683](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712153827683.png)

![image-20200712153856462](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712153856462.png)

```python
class Player1:
    def __init__(self, uid, name, level):
        self.uid = uid
        self.name = name
        self.level = level

class Player2:
    # 提前分配好空间
    __slots__ = ['uid', 'name', 'level']
    def __init__(self, uid, name, level):
        self.uid = uid
        self.name = name
        self.level = level

p1 = Player1('0001', 'Jim', 20)
p2 = Player2('0001', 'Jim', 20)

print(dir(p1))
print(dir(p2))
# {'__dict__', '__weakref__'}
# __dict__ 存动态绑定的属性，维护属性
print(set(dir(p1)) - set(dir(p2)))
# {'uid': '0001', 'name': 'Jim', 'level': 20}
print(p1.__dict__)
p1.__dict__['z'] = 300
# 300
print(p1.z)
# 删除属性
p1.__dict__.pop('z')

import sys
# 360
print(sys.getsizeof(p1.__dict__))

import tracemalloc
tracemalloc.start()
# start
la = [Player1(1,2,3) for _ in range(1000000)]
lb = [Player2(1,2,3) for _ in range(1000000)]
# end
snapshot = tracemalloc.take_snapshot()
# 统计整个文件使用内存
top_stats = snapshot.statistics('filename')
for stat in top_stats[:10]: print(stat)
# /Users/dingyuanjie/code/py_demo/mk/thread/p_12.py:0: size=215 MiB, count=3999996, average=56 B
# /Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/tracemalloc.py:0: size=56 B, count=1, average=56 B
```

## 如何让对象支持上下文管理

![image-20200712160615025](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712160615025.png)

![image-20200712162202784](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712162202784.png)

```python
# 可自动关闭资源
with open('demo.txt', 'w') as f:
    # f取决于 __enter__()的返回
    f.write('abc')
    f.write('efef')

# True
print(f.closed)

F = open('demo.txt', 'w')
f = F.__enter__()
# True
print(f is F)
F.__exit__()
```

```python
from sys import stdin, stdout
import getpass
import telnetlib
from collections import deque

class TelnetClient:
    def __init__(self, host, port=23):
        self.host = host
        self.port = port 

    # 进入时
    def __enter__(self):
        self.tn = telnetlib.Telnet(self.host, self.port)
        self.history = deque([])
        return self

    # 退出时
    def __exit__(self, exc_type, exc_value, exc_tb):
        print('IN __exit__', exc_type, exc_value, exc_tb)

        self.tn.close()
        self.tn = None

        with open('history.txt', 'a') as f:
            f.writelines(self.history)

        # 产生异常的话，不再上抛
        return True

    def login(self):
        # user
        self.tn.read_until(b"login: ")
        user = input("Enter your remote account: ")
        self.tn.write(user.encode('utf8') + b"\n")

        # password
        self.tn.read_until(b"Password: ")
        password = getpass.getpass()
        self.tn.write(password.encode('utf8') + b"\n")
        out = self.tn.read_until(b'$ ')
        stdout.write(out.decode('utf8'))

    def interact(self):
        while True:
            cmd = stdin.readline()
            if not cmd:
                break

            self.history.append(cmd)
            self.tn.write(cmd.encode('utf8'))
            out = self.tn.read_until(b'$ ').decode('utf8')

            stdout.write(out[len(cmd)+1:])
            stdout.flush()

# client = TelnetClient('192.168.0.105')
# client.connect()
# client.login()
# client.interact()
# client.cleanup()

with TelnetClient('192.168.0.105') as client:
    raise Exception('TEST')
    client.login()
    client.interact()

print('END')
```

## 如何创建可管理的对象属性

![image-20200712162245561](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712162245561.png)

![image-20200712163107960](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712163107960.png)

```python
import math

class Circle:
    def __init__(self, radius):
        self.radius = radius

    def get_radius(self):
      	# 保留1位小数
        return round(self.radius, 1)

    def set_radius(self, radius):
        if not isinstance(radius, (int, float)):
            raise TypeError('wronge type')
        self.radius = radius

    @property
    def S(self):
        return self.radius ** 2 * math.pi

    @S.setter
    def S(self, s):
        self.radius = math.sqrt(s / math.pi)

    R = property(get_radius, set_radius)

c = Circle(5.712)

c.S = 99.88
print(c.S)          # 99.88000000000001
print(c.R)          # 5.6

#print(c.get_radius())
#c.radius = '31.98'
```

## 如何让类支持比较操作

![image-20200712164113853](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712164113853.png)

![image-20200712165359347](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712165359347.png)

```python
class Rect:
    def __init__(self, w, h):
        self.w = w
        self.h = h

    def area(self):
        return self.w * self.h

    def __str__(self):
        return 'Rect:(%s, %s)' % (self.w, self.h)

    def __lt__(self, obj):
        print('__lt__', self, obj)
        return self.area() < obj.area()

rect1 = Rect(6, 9)             # 54
rect2 = Rect(7, 8)             # 56
print(rect1.area(),rect2.area())
print(rect1 > rect2)     # False
```

```python
from functools import total_ordering

from abc import ABCMeta, abstractclassmethod

@total_ordering
class Shape(metaclass=ABCMeta):
    @abstractclassmethod
    def area(self):
        pass

    def __lt__(self, obj):
        print('__lt__', self, obj)
        return self.area() < obj.area()

    def __eq__(self, obj):
        return self.area() == obj.area()

class Rect(Shape):
    def __init__(self, w, h):
        self.w = w
        self.h = h

    def area(self):
        return self.w * self.h

    def __str__(self):
        return 'Rect:(%s, %s)' % (self.w, self.h)

import math
class Circle(Shape):
    def __init__(self, r):
        self.r = r

    def area(self):
        return self.r ** 2 * math.pi


rect1 = Rect(6, 9) # 54
rect2 = Rect(7, 8) # 56
c = Circle(8)

print(rect1 < c)
print(c > rect2)
```

## 如何使用描述符对实例属性做类型检查

![image-20200712165419980](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712165419980.png)

![image-20200712170227350](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712170227350.png)

```python
class Attr:
    def __init__(self, key, type_):
        self.key = key
        self.type_ = type_

    def __set__(self, instance, value):
        print('in __set__')
        if not isinstance(value, self.type_):
            raise TypeError('must be %s' % self.type_)
        instance.__dict__[self.key] = value

    def __get__(self, instance, cls):
        print('in __get__', instance, cls)
        return instance.__dict__[self.key]

    def __delete__(self, instance):
        print('in __del__', instance)
        del instance.__dict__[self.key]

class Person:
    name = Attr('name', str)
    age = Attr('age', int)

p = Person()
p.name = 'liushuo'
p.age = '32'

```

## 如何在环状数据结构中管理内存

![image-20200712170252767](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712170252767.png)

 ![image-20200712171902563](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712171902563.png)

```python
class A:
    def __del__(self):
        print('in __del__')

a = A()

import weakref
a2 = weakref.ref(a)
a3 = a2()
print(a3 is a)      # True

del a
del a3
print(a2() is None)   # True
```

```python
# 引用计数为0时被删除
# 弱引用，不增加访问计数
import weakref
class Node:
    def __init__(self, data):
        self.data = data
        self._left = None
        self.right = None

    def add_right(self, node):
        self.right = node
        node._left = weakref.ref(self)

    @property
    def left(self):
        return self._left()

    def __str__(self):
        return 'Node:<%s>' % self.data

    def __del__(self):
        print('in __del__: delete %s' % self)

def create_linklist(n):
    head = current = Node(1)
    for i in range(2, n + 1):
        node = Node(i)
        current.add_right(node)
        current = node
    return head

head = create_linklist(1000)
print(head.right, head.right.left)
input()                   # 阻塞
head = None

import time
for _ in range(1000):
    time.sleep(1)
    print('run...')
input('wait...')
```

## 如何通过实例方法名字的字符串调用方法

![image-20200712171937480](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712171937480.png)

![image-20200712173010732](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712173010732.png)

```python
from lib1 import Circle
from lib2 import Triangle
from lib3 import Rectangle
from operator import methodcaller

def get_area(shape, method_name = ['area', 'get_area', 'getArea']):
    for name in method_name:
      # f = getattr(shape, name, None)
        # if f:
        #     return f()
      	# 改进
        if hasattr(shape, name):
            return methodcaller(name)(shape)
        


shape1 = Circle(1)
shape2 = Triangle(3, 4, 5)
shape3 = Rectangle(4, 6)

shape_list = [shape1, shape2, shape3]
# 获得面积列表
area_list = list(map(get_area, shape_list))
print(area_list)
```

# 多线程并发相关问题与解决技巧

## 如何使用多线程

![image-20200706223248842](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706223248842.png)

![image-20200706224137814](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706224137814.png)

![image-20200706223314408](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706223314408.png)

```python
import requests
import base64
from io import StringIO
import csv
from xml.etree.ElementTree import ElementTree, Element, SubElement

USERNAME = b'7f304a2df40829cd4f1b17d10cda0304'
PASSWORD = b'aff978c42479491f9541ace709081b99'

def download_csv(page_number):
    print('download csv data [page=%s]' % page_number)
    url = "http://api.intrinio.com/prices.csv?ticker=AAPL&hide_paging=true&page_size=200&page_number=%s" % page_number
    auth = b'Basic ' + base64.b64encode(b'%s:%s' % (USERNAME, PASSWORD))
    headers = {'Authorization' : auth}
    response = requests.get(url, headers=headers)

    if response.ok:
        return StringIO(response.text)           # 内存文件

def csv_to_xml(csv_file, xml_path):
    print('Convert csv data to %s' % xml_path)
    reader = csv.reader(csv_file)
    headers = next(reader)

    root = Element('Data')
    root.text = '\n\t'
    root.tail = '\n'

    for row in reader:
        book = SubElement(root, 'Row')
        book.text = '\n\t\t'
        book.tail = '\n\t'

        for tag, text in zip(headers, row):
            e = SubElement(book, tag)
            e.text = text
            e.tail = '\n\t\t'
        e.tail = '\n\t'

    ElementTree(root).write(xml_path, encoding='utf8')

def download_and_save(page_number, xml_path):
    # IO
    csv_file = None
    while not csv_file:
        csv_file = download_csv(page_number)
    # CPU
    csv_to_xml(csv_file, 'data%s.xml' % page_number)

from threading import Thread
class MyThread(Thread):
    def __init__(self, page_number, xml_path):
        super().__init__()
        self.page_number = page_number
        self.xml_path = xml_path

    def run(self):
        download_and_save(self.page_number, self.xml_path)

if __name__ == '__main__':
    import time
    t0 = time.time()
    thread_list = []
    for i in range(1, 6):
        t = MyThread(i, 'data%s.xml' % i)
        t.start()
        thread_list.append(t)

    for t in thread_list:
        t.join()
    # for i in range(1, 6):
    #      download_and_save(i, 'data%s.xml' % i)
    print(time.time() - t0)
    print('main thread end.')
```

##  如何线程间通信

![image-20200706224249900](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706224249900.png)

![image-20200706224647935](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706224647935.png)

![image-20200706224601840](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706224601840.png)



![image-20200706224437379](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200706224437379.png)



```python
import requests
import base64
from io import StringIO
import csv
from xml.etree.ElementTree import ElementTree, Element, SubElement
from threading import Thread

USERNAME = b'7f304a2df40829cd4f1b17d10cda0304'
PASSWORD = b'aff978c42479491f9541ace709081b99'

# class MyThread(Thread):
#     def __init__(self, page_number, xml_path):
#         super().__init__()
#         self.page_number = page_number
#         self.xml_path = xml_path
#
#     def run(self):
#         download_and_save(self.page_number, self.xml_path)

class DownloadThread(Thread):
    def __init__(self, page_number, queue):
        super().__init__()
        self.page_number = page_number
        self.queue = queue

    def run(self):
        csv_file = None
        while not csv_file:
            csv_file = self.download_csv(self.page_number)
        self.queue.put((self.page_number, csv_file))

    def download_csv(self, page_number):
        print('download csv data [page=%s]' % page_number)
        url = "http://api.intrinio.com/prices.csv?ticker=AAPL&hide_paging=true&page_size=200&page_number=%s" % page_number
        auth = b'Basic ' + base64.b64encode(b'%s:%s' % (USERNAME, PASSWORD))
        headers = {'Authorization' : auth}
        response = requests.get(url, headers=headers)

        if response.ok:
            return StringIO(response.text)

class ConvertThread(Thread):
    def __init__(self, queue):
        super().__init__()
        self.queue = queue

    def run(self):
        while True:
            page_number, csv_file = self.queue.get()
            self.csv_to_xml(csv_file, 'data%s.xml' % page_number)

    def csv_to_xml(self, csv_file, xml_path):
        print('Convert csv data to %s' % xml_path)
        reader = csv.reader(csv_file)
        headers = next(reader)

        root = Element('Data')
        root.text = '\n\t'
        root.tail = '\n'

        for row in reader:
            book = SubElement(root, 'Row')
            book.text = '\n\t\t'
            book.tail = '\n\t'

            for tag, text in zip(headers, row):
                e = SubElement(book, tag)
                e.text = text
                e.tail = '\n\t\t'
            e.tail = '\n\t'

        ElementTree(root).write(xml_path, encoding='utf8')

# def download_and_save(page_number, xml_path):
#     # IO
#     csv_file = None
#     while not csv_file:
#         csv_file = download_csv(page_number)
#     # CPU
#     csv_to_xml(csv_file, 'data%s.xml' % page_number)

from queue import Queue

if __name__ == '__main__':
    queue = Queue()
    thread_list = []
    for i in range(1, 6):
        t = DownloadThread(i, queue)
        t.start()
        thread_list.append(t)

    convert_thread = ConvertThread(queue)
    convert_thread.start()

    for t in thread_list:
        t.join()
    print('main thread end.')
```

## 如何在线程间进行事件通知

![image-20200712101452275](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712101452275.png)

![image-20200712101604018](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712101604018.png)

```python
from threading import Thread, Event
# 事件
def f(event):
    print('wait event...')
    event.wait()
    print('f end...')

e = Event()
thread = Thread(target=f, args=(e, ))
# wait event...
thread.start()
# f end
e.set()


thread = Thread(target=f, args=(e,))
# 此时 event.wait()阻塞不住了
thread.start()

# 如要继续使用事件
e.clear()
```

```python
import tarfile
import os

def tar_xml(tfname):
    tf = tarfile.open(tfname, 'w:gz')
    for fname in os.listdir('.'):
        if fname.endswith('.xml'):
            tf.add(fname)
            os.remove(fname)
    tf.close()

    # 如果压缩包内没有文件，则删除压缩包
    if not tf.members:
        os.remove(tfname)

# 当前目录下有.xml文件
tar_xml('test.tgz')
```

![image-20200712103651343](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712103651343.png)

```python
# 两个线程间通信
# CT 转换线程
# TT 打包线程
# CE 转换事件
# TE 打包事件
# 两个线程间通信
# CT 转换线程
# TT 打包线程
# CE 转换事件
# TE 打包事件

import requests
import base64
from io import StringIO
import csv
from xml.etree.ElementTree import ElementTree, Element, SubElement
from threading import Thread
from queue import Queue
import tarfile
import os

USERNAME = b'7f304a2df40829cd4f1b17d10cda0304'
PASSWORD = b'aff978c42479491f9541ace709081b99'


class DownloadThread(Thread):
    def __init__(self, page_number, queue):
        super().__init__()
        self.page_number = page_number
        self.queue = queue

    def run(self):
        csv_file = None
        while not csv_file:
            csv_file = self.download_csv(self.page_number)
        self.queue.put((self.page_number, csv_file))

    def download_csv(self, page_number):
        print('download csv data [page=%s]' % page_number)
        url = "http://api.intrinio.com/prices.csv?ticker=AAPL&hide_paging=true&page_size=100&page_number=%s" % page_number
        auth = b'Basic ' + base64.b64encode(b'%s:%s' % (USERNAME, PASSWORD))
        headers = {'Authorization': auth}
        response = requests.get(url, headers=headers)

        if response.ok:
            return StringIO(response.text)


class ConvertThread(Thread):
    def __init__(self, queue, c_event, t_event):
        super().__init__()
        self.queue = queue
        self.c_event = c_event
        self.t_event = t_event

    def run(self):
        count = 0
        while True:
            page_number, csv_file = self.queue.get()
            if page_number == -1:
                self.c_event.set()
                self.t_event.wait()
                break

            self.csv_to_xml(csv_file, 'data%s.xml' % page_number)
            count += 1
            if count == 2:
                count = 0
                # 通知转换完成
                self.c_event.set()

                # 等待打包完成
                self.t_event.wait()
                self.t_event.clear()

    def csv_to_xml(self, csv_file, xml_path):
        print('Convert csv data to %s' % xml_path)
        reader = csv.reader(csv_file)
        headers = next(reader)

        root = Element('Data')
        root.text = '\n\t'
        root.tail = '\n'

        for row in reader:
            book = SubElement(root, 'Row')
            book.text = '\n\t\t'
            book.tail = '\n\t'

            for tag, text in zip(headers, row):
                e = SubElement(book, tag)
                e.text = text
                e.tail = '\n\t\t'
            e.tail = '\n\t'

        ElementTree(root).write(xml_path, encoding='utf8')


class TarThread(Thread):
    def __init__(self, c_event, t_event):
        super().__init__(daemon=True)
        self.count = 0
        self.c_event = c_event
        self.t_event = t_event

    def run(self):
        while True:
            # 等待足够的xml
            self.c_event.wait()
            self.c_event.clear()

            print('DEBUG')
            # 打包
            self.tar_xml()

            # 通知打包完成
            self.t_event.set()

    def tar_xml(self):
        self.count += 1
        tfname = 'data%s.tgz' % self.count
        print('tar %s...' % tfname)
        tf = tarfile.open(tfname, 'w:gz')
        for fname in os.listdir('.'):
            if fname.endswith('.xml'):
                tf.add(fname)
                os.remove(fname)
        tf.close()

        if not tf.members:
            os.remove(tfname)


from threading import Event

if __name__ == '__main__':
    queue = Queue()
    c_event = Event()
    t_event = Event()
    thread_list = []
    for i in range(1, 15):
        t = DownloadThread(i, queue)
        t.start()
        thread_list.append(t)

    convert_thread = ConvertThread(queue, c_event, t_event)
    convert_thread.start()

    tar_thread = TarThread(c_event, t_event)
    tar_thread.start()

    # 等待下载线程结束
    for t in thread_list:
        t.join()

    # 通知Convert线程退出
    queue.put((-1, None))

    # 等待转换线程结束
    convert_thread.join()
    print('main thread end.')
```

![image-20200712104754205](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712104754205.png)

## 如何使用线程本地数据

![image-20200712104823112](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712104823112.png)

![image-20200712105957463](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712105957463.png)

```python
from threading import Thread, local

l = local()
l.x = 1

def f(l):
    l.x = 2
    print('in thread x = ', l.x)

# in thread x =  2
Thread(target=f, args=(l,)).start()

# 主线程 1
print(l.x)
```

```python
import os, cv2, time, struct, threading
from http.server import HTTPServer, BaseHTTPRequestHandler
from socketserver import TCPServer, ThreadingTCPServer
from threading import Thread, RLock
from select import select

class JpegStreamer(Thread):
    def __init__(self, camera):
        super().__init__()
        self.cap = cv2.VideoCapture(camera)
        self.lock = RLock()
        self.pipes = {}

    def register(self):
        pr, pw = os.pipe()
        self.lock.acquire()
        self.pipes[pr] = pw
        self.lock.release()
        return pr

    def unregister(self, pr):
        self.lock.acquire()
        pw = self.pipes.pop(pr)
        self.lock.release()
        os.close(pr)
        os.close(pw)

    def capture(self):
        cap = self.cap
        while cap.isOpened():
            ret, frame = cap.read()
            if ret:
                ret, data = cv2.imencode('.jpg', frame, (cv2.IMWRITE_JPEG_QUALITY, 40))
                yield data.tostring()

    def send_frame(self, frame):
        n = struct.pack('l', len(frame))
        self.lock.acquire()
        if len(self.pipes):
            _, pipes, _ = select([], self.pipes.values(), [], 1)
            for pipe in pipes:
                os.write(pipe, n)
                os.write(pipe, frame)
        self.lock.release()

    def run(self):
        for frame in self.capture():
            self.send_frame(frame)

class JpegRetriever:
    def __init__(self, streamer):
        self.streamer = streamer
        self.local = threading.local()

    def retrieve(self):
        while True:
            ns = os.read(self.local.pipe, 8)
            n = struct.unpack('l', ns)[0]
            data = os.read(self.local.pipe, n)
            yield data

    def __enter__(self):
        if hasattr(self.local, 'pipe'):
            raise RuntimeError()

        self.local.pipe = streamer.register()
        return self.retrieve()

    def __exit__(self, *args):
        self.streamer.unregister(self.local.pipe)
        del self.local.pipe
        return True

class WebHandler(BaseHTTPRequestHandler):
    retriever = None

    @staticmethod
    def set_retriever(retriever):
        WebHandler.retriever = retriever

    def do_GET(self):
        if self.retriever is None:
            raise RuntimeError('no retriver')

        if self.path != '/':
            return

        self.send_response(200) 
        self.send_header('Content-type', 'multipart/x-mixed-replace;boundary=jpeg_frame')
        self.end_headers()

        with self.retriever as frames:
            for frame in frames:
                self.send_frame(frame)

    def send_frame(self, frame):
        sh  = b'--jpeg_frame\r\n'
        sh += b'Content-Type: image/jpeg\r\n'
        sh += b'Content-Length: %d\r\n\r\n' % len(frame)
        self.wfile.write(sh)
        self.wfile.write(frame)

from concurrent.futures import ThreadPoolExecutor
class ThreadingPoolTCPServer(ThreadingTCPServer):
    def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True, thread_n=100):
        super().__init__(server_address, RequestHandlerClass, bind_and_activate=True)

        self.executor = ThreadPoolExecutor(thread_n)

    def process_request(self, request, client_address):
        self.executor.submit(self.process_request_thread, request, client_address)

if __name__ == '__main__':
    # 创建Streamer，开启摄像头采集。
    streamer = JpegStreamer(0)
    streamer.start()

    # http服务创建Retriever
    retriever = JpegRetriever(streamer)
    WebHandler.set_retriever(retriever)

    # 开启http服务器
    HOST = 'localhost'
    PORT = 9000
    print('Start server... (http://%s:%s)' % (HOST, PORT))
    httpd = ThreadingPoolTCPServer((HOST, PORT), WebHandler, thread_n=3)
    #httpd = ThreadingTCPServer((HOST, PORT), WebHandler)
    httpd.serve_forever()
```

## 如何使用线程池

![image-20200712110032912](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712110032912.png)

![image-20200712110103219](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712110103219.png)

```python
import threading, time, random

def f(a, b):
    print(threading.current_thread().name, ":" , a, b)
    time.sleep(random.randint(5, 10))
    return a * b

from concurrent.futures import ThreadPoolExecutor
exector = ThreadPoolExecutor(3)
# ThreadPoolExecutor-0_0 : 2 3
furure = exector.submit(f, 2, 3)
# 6
print(furure.result())

# 多个线程中执行
# ThreadPoolExecutor-0_0 : 1 2
# ThreadPoolExecutor-0_1 : 2 3
# ThreadPoolExecutor-0_2 : 3 4
# ThreadPoolExecutor-0_2 : 4 5
# ThreadPoolExecutor-0_1 : 5 6
exector.map(f, range(1, 6), range(2, 7))
```

## 如何使用多进程

![image-20200712111349231](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712111349231.png)

![image-20200712111418616](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712111418616.png)

```python
from multiprocessing import Process

def f(n):
    print('in child...')
    import time
    time.sleep(n)

p = Process(target=f, args=(10, ))
p.start()
p.join()


from threading import Thread
from multiprocessing import Process

x = 1
def g():
    global x
    x += 1

Thread(target=g).start()
# 2
print(x)

# 进程隔离，内存空间互不影响
Process(target = g).start()
# 2
print(x)

from multiprocessing import Queue, Pipe
from multiprocessing import Process

def f(q):
    print('in child')
    print(q.get())

if __name__ == '__main__':
    q = Queue()
    q.put('abc')
    Process(target=f, args=(q, )).start()
```

```python
from multiprocessing import Queue, Pipe
from multiprocessing import Process

c1, c2 = Pipe()
def f(c):
    print('in child')
    data = c.recv()
    print(data)
    c.send(data * 2)

if __name__ == '__main__':
    Process(target=f, args=(c2 , )).start()
    c1.send(100)
    # 100
    c1.recv()
```

```python
from threading import Thread
from multiprocessing import Process
from queue import Queue as Thread_Queue 
from multiprocessing import Queue as Process_Queue

def is_armstrong(n):
    a, t = [], n
    while t:
        a.append(t % 10)
        t //= 10
    k = len(a)
    return sum(x ** k for x in a) == n

def find_armstrong(a, b, q=None):
    res = [x for x in range(a, b) if is_armstrong(x)]
    if q:
        q.put(res)
    return res

def find_by_thread(*ranges):
    q = Thread_Queue()
    workers = []
    for r in ranges:
        a, b = r
        t = Thread(target=find_armstrong, args=(a, b, q))
        t.start()
        workers.append(t)

    res = []
    for _ in range(len(ranges)):
        res.extend(q.get())

    return res

def find_by_process(*ranges):
    q = Process_Queue()
    workers = []
    for r in ranges:
        a, b = r
        t = Process(target=find_armstrong, args=(a, b, q))
        t.start()
        workers.append(t)

    res = []
    for _ in range(len(ranges)):
        res.extend(q.get())

    return res

if __name__ == '__main__':
    import time
    t0 = time.time()
    res = find_by_process([10000000, 15000000], [15000000, 20000000], 
                         [20000000, 25000000], [25000000, 30000000])
    print(res)
    print(time.time() - t0)
```

# 装饰器使用问题与技巧

## 如何使用函数装饰器

![image-20200712174942227](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712174942227.png)

![image-20200712180211816](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712180211816.png)

![image-20200712175428841](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712175428841.png)

```python
def memo(func):
  	# 闭包
    cache = {}
    def wrap(*args):
        res = cache.get(args)
        if not res:
            res = cache[args] = func(*args)
        return res

    return wrap

# [题目1] 斐波那契数列（Fibonacci sequence）:
# F(0)=1，F(1)=1, F(n)=F(n-1)+F(n-2)（n>=2）
# 1, 1, 2, 3, 5, 8, 13, 21, 34, ...
# 求数列第n项的值？
@memo
def fibonacci(n):
    if n <= 1:
        return 1
    return fibonacci(n-1) + fibonacci(n-2)

# fibonacci = memo(fibonacci)
print(fibonacci(50))

# [题目2] 走楼梯问题
# 有100阶楼梯, 一个人每次可以迈1~3阶. 一共有多少走法？ 
@memo
def climb(n, steps):
    count = 0
    if n == 0:
        count = 1
    elif n > 0:
        for step in steps:
            count += climb(n-step, steps)
    return count

print(climb(100, (1,2,3)))
```

## 如何为被装饰的函数保存元数据

![image-20200712180248945](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712180248945.png)

![image-20200712181114852](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712181114852.png)

![image-20200712180350501](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712180350501.png)

![image-20200712180453236](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712180453236.png)

![image-20200712180543290](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712180543290.png)

![image-20200712180714998](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712180714998.png)

```python
from functools import update_wrapper, wraps

def my_decorator(func):
    @wraps(func)
    def wrap(*args, **kwargs):
        '''某功能包裹函数'''

        # 此处实现某种功能
        # ...

        return func(*args, **kwargs)
    return wrap

@my_decorator
def xxx_func(a, b):
    '''
    xxx_func函数文档：
    ...
    '''
    pass

print(xxx_func.__name__)
print(xxx_func.__doc__)
```

## 如何定义带参数的装饰器

![image-20200712181152191](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712181152191.png)

![image-20200712200306236](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712200306236.png)

![image-20200712181710020](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712181710020.png)

![image-20200712181804406](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712181804406.png)

```python
# 查看对象属性
import inspect

def type_assert(*ty_args, **ty_kwargs):
    def decorator(func):
        # A...
        func_sig = inspect.signature(func)
        bind_type = func_sig.bind_partial(*ty_args, **ty_kwargs).arguments
        def wrap(*args, **kwargs):
            # B...
            for name, obj in func_sig.bind(*args, **kwargs).arguments.items():
                type_ = bind_type.get(name)
                if type_:
                    if not isinstance(obj, type_):
                        raise TypeError('%s must be %s' % (name, type_))
            return func(*args, **kwargs)
        return wrap
    return decorator

@type_assert(c=str)
def f(a, b, c):
    pass

f(5, 10, 5.3)
```

## 如何实现属性可修改的函数装饰器

![image-20200712200323637](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712200323637.png)

![image-20200712201438011](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712201438011.png)

```python
import time
import logging

def warn_timeout(timeout):
    def decorator(func):
        #_timeout = [timeout]
        def wrap(*args, **kwargs):
            #timeout = _timeout[0]
            t0 = time.time()
            res = func(*args, **kwargs)
            used = time.time() - t0
            if used > timeout:
                logging.warning('%s: %s > %s', func.__name__, used, timeout)
            return res
        def set_timeout(new_timeout):
            # 闭包中的变量
            nonlocal timeout
            timeout = new_timeout
            #_timeout[0] = new_timeout
        wrap.set_timeout = set_timeout
        return wrap
    return decorator

import random
@warn_timeout(1.5)
def f(i):
    print('in f [%s]' % i)
    while random.randint(0, 1):
        time.sleep(0.6)

for i in range(30):
    f(i)

f.set_timeout(1)
for i in range(30):
    f(i)
```

## 如何在类中定义装饰器

![image-20200712201458934](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712201458934.png)

![image-20200712202409764](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712202409764.png)

![image-20200712202118075](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200712202118075.png)

```python
import time
import logging

DEFAULT_FORMAT = '%(func_name)s -> %(call_time)s\t%(used_time)s\t%(call_n)s'

class CallInfo:
    def __init__(self, log_path, format_=DEFAULT_FORMAT, on_off=True):
        self.log = logging.getLogger(log_path)
        self.log.addHandler(logging.FileHandler(log_path))
        self.log.setLevel(logging.INFO)
        self.format = format_
        self.is_on = on_off
    
    # 装饰器方法
    def info(self, func):
        _call_n = 0
        def wrap(*args, **kwargs):
            func_name = func.__name__
            call_time = time.strftime('%x %X', time.localtime())
            t0 = time.time()
            res = func(*args, **kwargs)
            used_time = time.time() - t0
            nonlocal _call_n
            _call_n += 1
            call_n = _call_n
            if self.is_on:
                self.log.info(self.format % locals())
            return res
        return wrap

    def set_format(self, format_):
        self.format = format_

    def turn_on_off(self, on_off):
        self.is_on = on_off

# 测试代码
import random

ci1 = CallInfo('mylog1.log')
ci2 = CallInfo('mylog2.log')
@ci1.info
def f():
    sleep_time = random.randint(0, 6) * 0.1
    time.sleep(sleep_time)
            
@ci1.info
def g():
    sleep_time = random.randint(0, 8) * 0.1
    time.sleep(sleep_time)

@ci2.info
def h():
    sleep_time = random.randint(0, 7) * 0.1
    time.sleep(sleep_time)

for _ in range(30):
    random.choice([f, g, h])()

ci1.set_format('%(func_name)s -> %(call_time)s\t%(call_n)s')
for _ in range(30):
    random.choice([f, g])()
```

