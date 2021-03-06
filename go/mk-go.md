go env

​	GOPATH="/Users/dingyuanjie/dev_env/mygo/go-code"

​		bin、pkg、src

​	GOROOT="/usr/local/go"



安装beego

​	go get -u github.com/astaxie/beego

​	beego 是一个快速开发 Go 应用的 HTTP 框架，可以用来快速开发 API、Web 及后端服务等各种应用，是一个 RESTful 的框架，主要设计灵感来源于 tornado、sinatra 和 flask 这三个框架，但是结合了 Go 本身的一些特性（interface、struct 嵌入等）而设计的一个框架



bee工具包：beego项目创建、热编译、开发测试、部署

​	go get github.com/beego/bee

​	bee 工具是一个为了协助快速开发 beego 项目而创建的项目，通过 bee 可以很容易的进行 beego 项目的创建、热编译、开发、测试、和部署。类似于前端框架vue的脚手架工具vue-cli

​	bee version



项目完整代码：https://github.com/go-learn-go/guess

beego框架介绍

​	快速开发、MVC架构、文档齐全，社区活跃

​	架构及原理

​		cache：文件、内存、memcache、redis（推荐）

​		config：ini、json（推荐）、xml、yaml

​		context：request、response

​		httplibs：

​			支持get、post、put、delete、head

​			支持https

​			支持超时设置

​			支持文件上传

​		logs：多种输出引擎、异步输出

​		orm

​		session

​		toolbox：定时任务、监控等



bee工具应用

​	bee new 新建项目结构

​	bee run 自动编译部署

​	bee generate 自动生成代码



cd $GOPATH/src

bee new immoc

cd immoc

```go
package controllers

import (
	"github.com/astaxie/beego"
)

type MainController struct {
	beego.Controller
}

// http://localhost:8080/?name=123456
func (c *MainController) Get() {
	name := c.GetString("name")
	c.Data["Website"] = name
	c.Data["Email"] = "astaxie@gmail.com"
	c.TplName = "index.tpl"
}
```

bee run

​	http://localhost:8080/





create database imooc;

use imooc;

```shell
create table `user`(
	`id` int(11) NOT NULL AUTO_INCREMENT,
	`name` varchar(128) NOT NULL DEFAULT '',
	`gender` tinyint(4) NOT NULL DEFAULT '0',
	`age` int(11) NOT NULL DEFAULT '0',
	PRIMARY KEY (`id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into user(name,gender,age)values('zhangsan',1,21);
insert into user(name,gender,age)values('lisi',0,22);
insert into user(name,gender,age)values('wangwu',1,20);
```

自动生成代码：cd $GOPATH/src/projectName 

​	bee generate scaffold user -fields="id:int64,name:string,gender:int,age:int" -driver=mysql -conn="root:@tcp(127.0.0.1:3306)/imooc"

```shell
/Users/dingyuanjie/dev_env/mygo/go-code/src/immoc [dingyuanjie@V_VYJIDING-MB0] [9:38]
> bee generate scaffold user -fields="id:int64,name:string,gender:int,age:int" -driver=mysql -conn="root:@tcp(127.0.0.1:3306)/imooc"
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v1.12.0
2021/02/21 10:14:10 INFO     ▶ 0001 Do you want to create a 'user' model? [Yes|No]
y
2021/02/21 10:14:15 INFO     ▶ 0002 Using 'User' as model name
2021/02/21 10:14:15 INFO     ▶ 0003 Using 'models' as package name
	create	 /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc/models/user.go
2021/02/21 10:14:15 INFO     ▶ 0004 Do you want to create a 'user' controller? [Yes|No]
y
2021/02/21 10:14:21 INFO     ▶ 0005 Using 'User' as controller name
2021/02/21 10:14:21 INFO     ▶ 0006 Using 'controllers' as package name
2021/02/21 10:14:21 INFO     ▶ 0007 Using matching model 'User'
	create	 /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc/controllers/user.go
2021/02/21 10:14:21 INFO     ▶ 0008 Do you want to create views for this 'user' resource? [Yes|No]
y
2021/02/21 10:14:44 INFO     ▶ 0009 Generating view...
	create	 /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc/views/user/index.tpl
	create	 /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc/views/user/show.tpl
	create	 /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc/views/user/create.tpl
	create	 /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc/views/user/edit.tpl
2021/02/21 10:14:44 INFO     ▶ 0010 Do you want to create a 'user' migration and schema for this resource? [Yes|No]
n
2021/02/21 10:15:03 INFO     ▶ 0011 Do you want to migrate the database? [Yes|No]
n
2021/02/21 10:15:18 SUCCESS  ▶ 0012 All done! Don't forget to add  beego.Router("/user" ,&controllers.UserController{}) to routers/route.go

2021/02/21 10:15:18 SUCCESS  ▶ 0013 Scaffold successfully generated!
```

```go
package routers

import (
	"immoc/controllers"
	"github.com/astaxie/beego"
)

func init() {
    //beego.Router("/", &controllers.MainController{})
	beego.Include(&controllers.UserController{})
}
```

```go
package main

import (
	_ "immoc/routers"
	"github.com/astaxie/beego"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	orm.RegisterDataBase("default", "mysql", "root:123456@tcp(127.0.0.1:3306)/imooc?charset=utf8")
	beego.Run()
}
```

bee run

```json
// http://localhost:8080/
```

http请求处理

​	路由设置、请求数据处理、数据输出 

数据库使用

​	orm使用、crud使用

前端页面

​	模板语法



![image-20210221103041332](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221103041332.png)

![image-20210221103056389](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221103056389.png)

![image-20210221103226936](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221103226936.png)

![image-20210221103716180](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221103716180.png)





```sql
CREATE TABLE `subject` (
          `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'pk',
          `img` varchar(512) NOT NULL COMMENT '图片',
          `option` varchar(512) NOT NULL DEFAULT '' COMMENT '题目选项，为json格式',
          `answer_key` varchar(32) NOT NULL DEFAULT '' COMMENT '选择题的key，为ABCD等字符',
          `answer_value` varchar(32) NOT NULL DEFAULT '' COMMENT '答案value，电影名',
          `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态',
          `hash` char(32) NOT NULL DEFAULT '' COMMENT '哈希值',
          PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2046 DEFAULT CHARSET=utf8 COMMENT='题目表';

INSERT INTO `subject` (`id`, `img`, `option`, `answer_key`, `answer_value`, `create_time`, `status`, `hash`)
VALUES
	(1, '/img/cut/yaoshen.webp', '{ \"A\": \"无人区\", \"B\": \"人在囧途\", \"C\": \"我不是药神\", \"D\": \"心花路放\" }', 'C', '我不是药神', '2018-09-08 12:34:06', 0, '');

INSERT INTO `subject` (`id`, `img`, `option`, `answer_key`, `answer_value`, `create_time`, `status`, `hash`)
VALUES
	(2, '/img/cut/bat.webp', '{ \"A\": \"黑暗骑士崛起\", \"B\": \"蝙蝠侠大战超人\", \"C\": \"侠影之谜\", \"D\": \"黑暗骑士\" }', 'D', '黑暗骑士', '2018-09-08 12:34:19', 0, '');
```

// /Users/dingyuanjie/dev_env/mygo/go-code/src/immoc

bee generate scaffold subject -fields="id:int64,img:string,option:string,answer_key:string,answer_value:string" -driver=mysql -conn="root:@tcp(127.0.0.1:3306)/imooc"

```go
package main

import (
	_ "immoc/routers"
	"github.com/astaxie/beego"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql"
)

func init() {
	orm.RegisterDriver("mysql", orm.DBMySQL)
	orm.RegisterDataBase("default","mysql","root:123456@tcp(127.0.0.1:3306)/imooc?charset=utf8")
}

func main() {
	beego.Run()
}
```



bee run



==部署==

​	应用执行文件

​	模版文件

​	静态资源

​	配置文件

​		vim conf/app.conf

​			appname = guess

​			httpport = 8080

​			runmode = prod

​	打包项目

​		bee pack  生成guess.tar.gz文件

​		mkdir guess

​		cd guess

​		cp $GOPATH/src/guess/guess.tar.gz ./

​		tar -zxvf guess.tar.gz

​		./guess		



==健康检查==

```go
package main

import (
	"database/sql"
	"github.com/astaxie/beego/toolbox"
)

type DatabaseCheck struct {
}

func (* DatabaseCheck)Check() error {
	_, err := sql.Open("mysql", "root:@tcp/guess?charset=utf8")
	if err != nil {
		return err
	}
	return nil
}

// 启动时检测数据库连接是否正常
func init() {
	toolbox.AddHealthCheck("database", &DatabaseCheck{})
}
```

```shell
conf/app.conf
# 添加允许健康检查
EnableAdmin = true
```

```shell
http://localhost:8088/healthcheck
```



==socket==

​	推模式与拉模式的区别

​		拉模式

​			数据更新频率低，则大多数请求是无效的

​			在线用户数量多，则服务端的查询负载很高

​			定时轮询拉取，无法满足实效性要求

​		推模式

​			仅在数据更新时才需要推送

​			需要维护大量的在线长连接

​			数据更新后可以立即推送

​		基于WebSocket推送

​			浏览器支持的socket编程，轻松维持服务端的长连接

​			基于TCP可靠传输之上的协议，无需开发者关心通讯细节

​			提供了高度抽象的编程接口，业务开发成本较低

​	WebSocket协议与交互

​		通讯流程

​			==握手：客户端发送header中带有Upgrade:websocket字段的http请求给服务端，表明客户端想升级成websocket协议，服务端收到后会给客户端一个握手确认，允许向websocket协议转换。客户端和服务端可以互相发送消息==

​		传输原理

​			协议升级后，继续复用HTTP的底层socket完成后续通讯

​			message底层被切分成多个frame帧传输	

​			编程时只需操作message，无需关心frame

​			框架底层完成TCP网络I/O，WebSocket协议解析，开发者无需关心

​		抓包观察

​			使用chrome开发者工具，观察WebSocket通讯流程

​	服务端的技术选型与考虑

​		NodeJS

​			单线程模型，推送性能有限

​		C/C++

​			TCP通讯、WebSocket协议实现成本高

​		Go	

​			多线程，基于协程模型并发

​			成熟的WebSocket标准库，无需造轮子

​	==Go实现WebSocket服务端==

​		实现HTTP服务端

​			WebSocket是HTTP协议Upgrade而来

​			使用http标准库快速实现空接口：/ws

```go
package main

import "net/http"

func wsHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello"))
}

func main() {
	// http://localhost:7777/ws
	http.HandleFunc("/ws", wsHandler)
	http.ListenAndServe("0.0.0.0:7777", nil)
}
```

​		完成WebSocket握手

​			使用websocket.Upgrader完成协议握手，得到WebSocket长连接

​			操作websocket api，读取客户端消息，然后原样发送回去

```go
package main

import (
	"github.com/gorilla/websocket"
	"net/http"
)

var(
	upgrader = websocket.Upgrader {
		// 允许跨域
		CheckOrigin: func(r *http.Request) bool {
			return true
		},
	}
)

func wsHandler(w http.ResponseWriter, r *http.Request) {
	//w.Write([]byte("hello"))

	var (
		conn *websocket.Conn
		err error
		data []byte
	)

	// Upgrade:websocket
	if conn, err = upgrader.Upgrade(w, r, nil); err !=nil {
		return
	}

	// websocket Conn
	for {
		// Text, Binary
		if _, data, err = conn.ReadMessage(); err != nil {
			goto ERR
		}
		if conn.WriteMessage(websocket.TextMessage, data); err != nil {
			goto ERR
		}
	}

ERR:
	conn.Close()
}

func main() {
	// http://localhost:7777/ws
	http.HandleFunc("/ws", wsHandler)
	http.ListenAndServe("0.0.0.0:7777", nil)
}
```

直接打开client.html

![image-20210221152827591](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221152827591.png)

==封装websocket==

​	不足

​		缺乏工程化的设计

​			其他代码块，无法直接操作WebSocket连接

​			WebSocket连接非线程安全，并发读/写需要同步手段

​	优化

​		隐藏细节，封装API

​			封装Connection结构，隐藏WebSocket底层连接

​			封装Connection的API，提供Send/Read/Close等线程安全接口

​	API原理

​		Channel是线程安全的

​		SendMessage将消息投递到out channel

​		ReadMessage从in channel读取消息

​	内部原理

​		启动读协程，循环读取WebSocket，将消息投递到in channel

​		启动写协程，循环读取out channel，将消息写给WebSocket	

https://github.com/owenliang/go-websocket/blob/master/server.go	

```go
package main

import (
	"github.com/gorilla/websocket"
	"go-websocket/impl"
	"net/http"
	"time"
)

var(
	upgrader = websocket.Upgrader {
		// 允许跨域
		CheckOrigin: func(r *http.Request) bool {
			return true
		},
	}
)

func wsHandler(w http.ResponseWriter, r *http.Request) {
	//w.Write([]byte("hello"))

	var (
		wsConn *websocket.Conn
		err error
		data []byte
		conn *impl.Connection
	)

	// Upgrade:websocket
	if wsConn, err = upgrader.Upgrade(w, r, nil); err !=nil {
		return
	}
	if conn, err = impl.InitConnection(wsConn); err != nil {
		goto ERR
	}

	go func() {
		var(
			err error
		)
		for {
			if err = conn.WriteMessage([]byte("heartbeat")); err != nil {
				return
			}
			// 每隔1秒发送心跳消息
			time.Sleep(1 * time.Second)
		}
	}()

	for {
		if data, err = conn.ReadMessage(); err != nil {
			goto ERR
		}
		if err = conn.WriteMessage(data); err != nil {
			goto ERR
		}
	}
ERR:
	conn.Close()
}

func main() {
	// http://localhost:7777/ws
	http.HandleFunc("/ws", wsHandler)
	http.ListenAndServe("0.0.0.0:7777", nil)
}
```

```go
package impl

import (
	"errors"
	"github.com/gorilla/websocket"
	"sync"
)

type Connection struct {
	wsConn *websocket.Conn
	inChan chan []byte
	outChan chan[]byte
	//
	closeChan chan byte
	// 锁
	mutex sync.Mutex
	isClosed bool
}

// go中一般最后都会放error
func InitConnection(wsConn *websocket.Conn)(conn *Connection, err error) {
	conn = &Connection {
		wsConn: wsConn,
		inChan: make(chan []byte, 1000),
		outChan: make(chan []byte, 1000),
		closeChan: make(chan byte, 1),
	}
	// 启动读协程
	go conn.readLoop()
	// 启动写协程
	go conn.writeLoop()
	return
}

// API
func (conn *Connection) ReadMessage() (data []byte, err error) {
	select {
		case data =<- conn.inChan:
		case <- conn.closeChan:
			err = errors.New("connection is closed")
	}
	return
}

func (conn *Connection)WriteMessage(data []byte)(err error) {
	select {
		case conn.outChan <- data:
		case <- conn.closeChan:
			err = errors.New("connection is closed")
	}
	return
}

func (conn *Connection)Close()  {
	// 线程安全，可重入的Close
	conn.wsConn.Close()
	// 这一行代码只执行一次（channel只能关闭一次）
	conn.mutex.Lock()
	if !conn.isClosed {
		close(conn.closeChan)
		conn.isClosed = true
	}
	conn.mutex.Unlock()
}

// 内部实现
func(conn *Connection) readLoop() {
	var(
		data []byte
		err error
	)
	// 不停的读取消息放入队列里
	for{
		if _, data, err = conn.wsConn.ReadMessage(); err != nil {
			goto ERR
		}
		// 阻塞在这里，等待 inChan有空闲的位置
		select {
			case conn.inChan <- data:
			case <- conn.closeChan:
				// closeChan关闭的时候
				goto ERR
		}
	}
ERR:
	conn.Close()
}

func (conn *Connection) writeLoop() {
	var(
		data []byte
		err error
	)
	for{
		select {
			case data =<- conn.outChan:
			// 出现网络错误时关闭
			case <- conn.closeChan:
				goto ERR
		}
		// data =<- conn.outChan
		if err = conn.wsConn.WriteMessage(websocket.TextMessage, data); err != nil {
			goto ERR
		}
	}
ERR:
	conn.Close()
}
```

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <script>
        window.addEventListener("load", function(evt) {
            var output = document.getElementById("output");
            var input = document.getElementById("input");
            var ws;
            var print = function(message) {
                var d = document.createElement("div");
                d.innerHTML = message;
                output.appendChild(d);
            };
            document.getElementById("open").onclick = function(evt) {
                if (ws) {
                    return false;
                }
                ws = new WebSocket("ws://localhost:7777/ws");
                ws.onopen = function(evt) {
                    print("OPEN");
                }
                ws.onclose = function(evt) {
                    print("CLOSE");
                    ws = null;
                }
                ws.onmessage = function(evt) {
                    print("RESPONSE: " + evt.data);
                }
                ws.onerror = function(evt) {
                    print("ERROR: " + evt.data);
                }
                return false;
            };
            document.getElementById("send").onclick = function(evt) {
                if (!ws) {
                    return false;
                }
                print("SEND: " + input.value);
                ws.send(input.value);
                return false;
            };
            document.getElementById("close").onclick = function(evt) {
                if (!ws) {
                    return false;
                }
                ws.close();
                return false;
            };
        });
    </script>
</head>
<body>
<table>
    <tr><td valign="top" width="50%">
            <p>Click "Open" to create a connection to the server,
                "Send" to send a message to the server and "Close" to close the connection.
                You can change the message and send multiple times.
            </p>
            <form>
                <button id="open">Open</button>
                <button id="close">Close</button>
                <input id="input" type="text" value="Hello world!">
                <button id="send">Send</button>
            </form>
        </td><td valign="top" width="50%">
            <div id="output"></div>
        </td></tr></table>
</body>
</html>
```

分析技术难点

​	3个性能瓶颈

​		内核瓶颈

​			推送量大：100万在线  * 10条 / 秒 = 1000万条 / 秒

​			内核瓶颈：linux内核发送TCP的极限包频 ～ 100万/秒

​		锁瓶颈

​			需要维护在线用户集合（100万在线），通常是一个字典结构

​			推送消息即遍历整个集合，顺序发送消息，耗时极长

​			推送期间，客户端仍旧正常上下线，所以集合需要上锁

​		CPU瓶颈

​			浏览器与服务端通常采取json格式通讯

​			json编码非常耗费CPU资源

​			向100万在线推送1次，则需100万次json encode

技术难点解决方案

​	内核瓶颈-优化原理

​		减少网络小包的发送

​			将同一秒内的N条消息，合并成1条消息

​			合并后，每秒推送次数只等于在线连接数

​	锁瓶颈-优化原理

​		大拆小

​			连接打散到多个集合中，每个集合有自己的锁

​			多线程并发推送多个集合，避免锁竞争

​			读写锁取代互斥锁，多个推送任务可以并发遍历相同集合

​	CPU瓶颈-优化原理

​		减少重复计算

​			json编码前置，1次消息编码 + 100万次推送

​			消息合并前置，N条消息合并后只编码1次

![image-20210221173952493](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221173952493.png)

揭秘分布式架构

​	单机瓶颈

​		维护海量长连接会花费不少内存

​		消息推送瞬时消耗大量CPU资源

​		消息推送瞬时带宽高达400-600MB（4-6Gbits），是主要瓶颈

​	网关集群

​		![image-20210221174259992](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221174259992.png)

​	逻辑集群

​		基于HTTP/2协议向gateway集群分发消息

​			HTTP/2支持连接复用，用作RPC性能更佳

​		基于HTTP/1协议对外提供推送API	

​			HTTP/1更加普及，对业务方更加友好

​	整体架构

​		![image-20210221175204032](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210221175204032.png)





https://www.imooc.com/video/21077

https://www.imooc.com/learn/1163

https://www.imooc.com/learn/1066

https://www.imooc.com/learn/1162

https://www.imooc.com/learn/602



# 常规用法

## 内建方法

### make

创建slice、map、chan

返回类型引用

```go
mSlice := make([]string,3)
mMap := make(map[int]string)
fmt.Println(reflect.TypeOf(mMap))    // map[int]string    引用类型
mChan := make(chan int) 
```

### new

内存置零

返回传入类型的指针类型

```go
nMap := new(map[int]string)
fmt.Println(reflect.TypeOf(mMap))    // *map[int]string    指针类型
```

### append & delete & copy

slice -> append & copy

map -> delete

```go
append(mSlice, "3")   // len(mSlice)、cap(mSlice) 
copy(mSliceDst, mSliceSrc)    // 会覆盖
delete(mMap, "key")
```

### panic & recover

处理异常

​	panic抛出异常

​	recover捕获异常

```go
func receivePanic() {
  defer coverPanic()
  painc("I am panic")
  // panic(errors.New("I am error"))
}

func coverPanic() {
  message := recover()
  switch message.(type) {
  	case string:
    	fmt.Println("string message: ", message)        // I am panic     程序在继续运行，但是业务没有继续
  	case error:
    	fmt.Println("error message: ", message)
 		default:
    	fmt.Println("unknow panic: ", message)
  }
}
```

### len & cap & close

len - string、array、slice、map、chan

cap - slice、array、chan

close - chan

```go
mSlice := make([]string, 2, 5)
len(mSlice)   // 2   , 可通过append添加元素来动态扩容, append(mSlice, "aa")
cap(mSlice)   // 5  

mChan := make(chan int, 1)
defer close(mChan)
mChan <- 1
```

##JSON

encoding/json

### 序列化

```go
// 结构体序列化
type Server struct {
  ServerName string  `json:"name"`           // tag，json中使用的名称
  ServerIp string    `json:"ip"`
  ServerPort int     `json:"port"`
}
func SerialzeStruct() {
  server := new(Server)          // 返回指针
  server.ServerName = "Demo-for-struct"
  server.ServerIp = "127.0.0.1"
  server.ServerPort = 8080
  
  b,err := json.Marshal(server)      // 序列化成json字节数组
  if err != nil {
    fmt.Println("Marshal err :", err.Error())
    return
  }
  fmt.Println("Marshal json : ", string(b)) // 将json字节数组转换成string
}

// Map序列化
func SerialzeMap() {
  server := make(map[string]interface{})
  server["ServerName"] = "Demo-for-map"
  server["ServerIp"] = "192.0.0.1"
  server["ServerPort"] = 8080
  
  b,err := json.Marshal(server)      // 序列化成json字节数组
  if err != nil {
    fmt.Println("Marshal err :", err.Error())
    return
  }
  fmt.Println("Marshal json : ", string(b)) // 将json字节数组转换成string
}

func Marshal(v interface{}) ([]byte, error)
```

### 反序列化

反序列化为结构体

反序列化为Map

func Unmarshal(data []byte, v interface{}) error

```go
func DeSerialzeStruct() {
  jsonString := `{"ip":"192.168.0.1","name":"Demo-for-json","port":8080}`
  server := new(Server)
  err := json.Unmarshal([]byte(jsonString), &server)
  if err != err {
    fmt.Println("Unmarshal err : ", err.Error())
  }
  fmt.Println("Unmarshhal struct : ", server)
}

func DeDeSerialzeMap() {
  jsonString := `{"ip":"192.168.0.1","name":"Demo-for-json","port":8080}`
  server := make(map[string]interface{})
  err := json.Unmarshal([]byte(jsonString), &server)
  if err != err {
    fmt.Println("Unmarshal err : ", err.Error())
  }
  fmt.Println("Unmarshhal struct : ", server)
}
```

## 糖衣语法

... 可变参数

:= 声明 赋值 类型推断

```go
func Sugar(values...string) {
  for _, v := range values {
    fmt.Println("--->v:", v)
  }
}
```

## Module

集成在go命令里的工具集

go.mod文件：module名称、go版本号、依赖库及其版本号

go.sum文件：版本管理文件，hash值

```go
// go mod init [muduleName]
初始化module
// go mod graph
输出所有依赖图
// go mod download
下载相关依赖，moudule目录下执行
// go mod tidy
整理，删除不需要的依赖，添加需要的依赖
// go mod verify
验证依赖内容及格式，验证go.mod文件、依赖的源代码
// go mod why -m github.com/...
输出依赖关系，谁需要这些依赖
// go mod edit  (重要)
编辑依赖9个子命令
go mod edit help
go mod edit -require github.com/hashicorp/golang-lru@v0.5.3
// go mod vendor
将所有依赖添加到当前的verdor目录
```

列出依赖

```go
go mod graph
go mod why
go list -m all
```

添加依赖

```go
go get
go build
go mod edit require
go mod download
```

# 面向对象

`&`取得变量的地址

`*`取得指针变量指向的内存地址的值，也用做定义指针类型的关键字

Go语言中除了map、slice、chan外，其他类型在函数参数中都是值传递，引用传递一定要用指针类型

==当使用指针类型定义方法后，结构体类型的变量调用方法时会自动取得该结构体的指针类型并传入方法==

接口实现方法时，用指针类型实现的接口函数只能算是指针类型实现的，用结构体类型实现的方法也作为是指针类型实现

```go
func main() {
    i := 0
    f(&i)
    fmt.Println(i)
}

func f(count *int) {
    fmt.Println(*count)
    (*count)++
}
```

## 结构体

若干字段的集合

```go
// 定义struct
type Dog struct {
  ID int          // 属性
  Name string
  Age int
}

// 初始化
func TestForStruct() {
  // 方式一 var
  var dog Dog
  dog.ID = 0
  dog.Name = "KiKi"
  dog.Age = 3
  // 方式二 省略var，声明并初始化
  dog := Dog{ID:1, Name:"YaYa", Age: 2}
  // 方式三 new
  dog := new(Dog)     // 指针
  dog.ID = 0
  dog.Name = "KiKi"
  dog.Age = 3
  
  fmt.Println("Dog: ", dog)
}

// 属性及函数
func (d *Dog)Run() {
  fmt.Println("ID: ", d.ID, " Dog is running")
}
dog.Run()

// 两种作用域
首字母大写 - 公开
首字母小写 - 私有

// 组合
面向对象特性：继承
组合实现
type Animal struct {
  Color string // 属性
}
type Dog struct {
  Animal
  ID int          // 属性
  Name string
  Age int
}
func (d *Dog)Eat() {
  fmt.Println("yummy yummy")
}
```

## 接口

### 定义

==公共方法组合起来以封装特定功能的集合==，抽象、封装、多态

面向对象实现

### 声明

```go
// 声明接口
type Behavior interface {
  Run() string
  Eat() string
}

// 隐式实现了 Behavior 接口
func (d *Dog)Eat() string {
  fmt.Println("yummy yummy")
  return "yummy yummy"
}
func (d *Dog)Run() string{
  fmt.Println("ID: ", d.ID, " Dog is running")
  return " Dog is running"
}
```

### 多态

面向对象特性：多态

接口定义变量

```go
type Cat struct {
  Animal
  ID int          // 属性
  Name string
  Age int
}
// 隐式实现了Behavior接口
func (d *Cat)Eat() string {
  fmt.Println("Cat yummy yummy")
  return "Cat yummy yummy"
}
func (d *Cat)Run() string{
  fmt.Println("ID: ", d.ID, " Cat is running")
  return " Cat is running"
}

// 测试接口定义变量        多态
func action(b interface_demo.Behavior) string {
  b.Run()
  b.Eat()
  return ""
}
dog := new(struct_demo.Dog)
cat := new(struct_demo.Cat)
action(dog)
action(cat)
```





# 效率工具

## 并发

并发的实现：协程，比线程需要的初始内存更小4kb左右，通过go关键字启动

```go
// 启动协程
func Loop() {
  for i:=1; i<11;i++ {
    time.Sleep(time.Micosecond * 10)
    fmt.Println("%d,", i)
  }
}

// 并发
go gorotine.Loop()
go gorotine.Loop()
time.Sleep(time.Second * 5)   // 防止启动太快没打印出来


// 多核CPU设置
runtime.GOMAXPROCS(runtime.NumCPU() - 1)  
```

多协程间的通信：channel、==select==（必须是IO操作，一般是从channel中获取数据，随机接收来自多个管道的数据）

```go
var chanInt chan int = make(chan int, 10)
var timeout chan bool = make(chan bool)

func Send() {
  chanInt <- 1
  time.Sleep(time.Second * 1)
  chanInt <- 2
  time.Sleep(time.Second * 1)
  chanInt <- 3
  time.Sleep(time.Second * 1)
  timeout <- true
}
func Receive() {
  // num := <- chanInt
  // fmt.Println("num", num)
  // num = <- chanInt
  // fmt.Println("num", num)
  // num = <- chanInt
  // fmt.Println("num", num)
  for{
    select {
    	case num := <- chanInt:
      	fmt.Println("num", num)
      case <- timeout:
        fmt.Println("timeout ...")
    }
  }
}

go gorotine.Send()
go gorotine.Receive()
time.Sleep(time.Second * 5)
```

多协程间的同步

​	系统工具sync.waitgroup

​		Add(delta int) 添加协程记录

​		Done()移除协程记录

​		Wait()同步等待所有记录的协程全部结束

==sync==

```go
var WG sync.WaitGroup

//测试协程间的通信
gorotine.Read()       // 主线程中
go gorotine.Write()   // 协程中删除记录
gorotine.WG.Wait()
fmt.Println("All done!")

time.Sleep(time.second * 60)


// gorotine
func Read() {
  for i:=0;i<3;i++ {
    WG.Add(1)
  }
}
func Write() {
  for i:=0;i<3;i++ {
    time.Sleep(time.Second * 2)
    fmt.Println("Done ->", i)
    WG.Done()
  }
}
```

## 指针

###  ==指针基本使用==

```go
// 定义指针变量
func TestPoint() {
  var count int = 20
  var countPoint *int
  // 为指针变量赋值
  countPoint = &count
  fmt.Println("count 变量的地址: %x \n", &count)      // c000054058
  if countPoint != nil {
    fmt.Println("countPoint 变量存储的地址: %x \n", countPoint)      // c000054058
  }
  // 访问指针变量中指向地址的值
  fmt.Println("countPoint 指针指向地址的值: %d \n", *countPoint )      // 20      *取值
}
// go不支持指针运算
```

### 指针数组

```go

func TestPointArr() {
  // 指针数组
  a,b := 1,2
  pointArr := [...]*int{&a, &b}
  fmt.Println("指针数组 pointArr:", pointArr)      // [0xc000054058 0xc000054070]
  
  // 数组指针
  arr := [...]int{3,4,5}
  arrPoint := &arr
  fmt.Println("数组指针 arrPoint:", arrPoint)      // &[3 4 5]
}
```

### 指向指针的指针

### 值传递和指针传递





## 环境变量

```shell
vi ~/.bashrc

# 新增
export GOROOT=/usr/local/go
export GOPATH=/Users/username/go/code # 代码目录，自定义即可
export PATH=$PATH:$GOPATH/bin

# 及时生效，请执行命令：source ~/.bashrc
```

bin：存放编译后可执行的文件

pkg：存放编译后的应用包

src：存放应用源代码





option + command + v



https://zhuanlan.zhihu.com/p/60703832



go mod国内镜像

```shell
# 在 Linux 或 macOS 上面，需要运行下面命令（或者，可以把以下命令写到 .bashrc 或 .bash_profile 文件中）：
vi ~/.bash_profile
# 启用 Go Modules 功能
go env -w GO111MODULE=on
# 配置 GOPROXY 环境变量，以下三选一
# 1. 七牛 CDN
go env -w  GOPROXY=https://goproxy.cn,direct
# 2. 阿里云
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
# 3. 官方
go env -w  GOPROXY=https://goproxy.io,direct
# 生效
source ~/.bash_profile

# hello目录
go mod init hello
```

https://github.com/linehk/gopl/blob/master/ch1/server3/main.go

==问题==

​	compile: version "go1.15.4" does not match go tool version "go1.15.5"

​	解决：如果您使用的OSX自制软件可能需要设置安装`$GOROOT`在你`.bashrc` ， `.zshrc` ，等：

​		export GOROOT=/usr/local/opt/go/libexec



​	==GoLand中使用mod模式==

​		进入项目文件夹

​			go mod init projectName

​		GoLand - Perferences - Go

​			GOROOT、GOPATH配置成本地的

​			GOMODULES 勾选 Enable

​		项目从 pkg/mod中查找依赖，而不是从src下查找依赖