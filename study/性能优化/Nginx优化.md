# 为什么是nginx而不是apache

轻量级，同样起web服务，比apache占用更少的内存及资源

​	模块化、事件驱动、单线程、异步、IO多路复用

静态处理，Nginx静态处理性能比Apache高3倍以上

抗并发，nginx处理请求是异步非阻塞的，而apache则是阻塞型的，在高并发下nginx能保持低资源低消耗高性能

高度模块化的设计，编写模块相对简单

​	核心模块：ngx_core、ngx_errlog、ngx_conf、ngx_events、ngx_event、ngx_epoll、ngx_regex

​	标准HTTP模块：Ngx_http_core、Ngx_http_charset、Others

​	可选HTTP模块：Ngx_http_gzip、Ngx_http_ssl、Others

​	邮件服务模块：Ngx_mail_core、Ngx_mail_pop3、Others

​	第三方模块：Rds-json-nginx、Lua-nginx、Others

社区活跃，各种高性能模块出品迅速

# Nginx是如何做到高性能和高可扩展的

事件驱动架构

![image-20210217124023461](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217124023461.png)

一个主进程和若干worker进程和helper进程

​	共享内存进行通信

​	主进程：读取并验证配置信息、创建绑定关闭套接字、启动终止worker线程、控制非中断的程序升级启动新的二进制程序、重新打开日志文件、编译一些嵌入式脚本

​	worker进程：接收传入并处理来自客户端的连接

​	

![image-20210217124205415](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217124205415.png)

每个worker进程以非阻塞的方式处理多个连接，这减少了上下文切换的次数

状态机的调度

安装

​	nginx.org/en/download.html

​	1.18稳定版本

```shell
cd /usr/local
wget http://nginx.org/download/nginx-1.18.0.tar.gz
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-openssl=/usr/local/openssl-1.0.2l --with-http_gzip_static_module --with-http_realip_module -with-http_sub_module --with-http_ssl_module
make && make install
/usr/local/nginx/sbin/nginx
ps -ef | grep nginx
# 查看是否启动成功   默认80端口
http://192.168.0.107/
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx -s reload
```

# Nginx运行工作进程数量优化

如何查看工作进程数

​	ps -ef | grep nginx

IO密集型：调整worker进程数=CPU的核心数 x 2           # 查看cpu信息 lscpu、more /proc/cpuinfo

nginx.conf

​	worker_processes 2;

调整后检查进程：ps -ef | grep nginx | grep -v grep

ps -aux|grep nginx | grep -v grep

# Nginx运行CPU亲和力优化

为什么要绑定Nginx进程到不同的CPU上

​	充分利用硬件资源多核CPU，避免忙的忙死，闲的闲死

如何分配不同的Nginx进程给不同的CPU处理？

​	nginx.conf

​	worker_processes 2;

​	worker_cpu_affinity 01 10;

​	或者

​	worker_processes 4;

​	worker_cpu_affinity 0001 0010 0100 1000;

​	或者

​	worker_processes 8;

​	worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

# Nginx最大打开文件数优化

nginx报错打开文件数过多，原因是什么

​	有4个被内部占用

安装ab压测工具：yum install -y httpd-tools

​	ab -n 1024 -c 1024 http://192.168.0.107/index.html

​	每个worker进程的并发数（实际达不到1024，1021就报错了）

修改配置：

​	nginx.conf

​		worker_connections 1028;

​	ab -n 1028 -c 1028 http://192.168.0.107/index.html

​	每个worker进程的并发数（实际达不到1028，还是1021就报错，修改没有生效）

正确修改

​	worker_processes下一行添加 worker_rlimit_nofile 1028; 

​	worker_connections 1028;

修改后，还需要修改操作系统最大支持打开文件数

​	vi /etc/security/limits.conf

​	文件末尾加上：

` * soft nofile 1028`

`* hard nofile 1028`

​	`* 表示任何用户`

查看操作系统支持最大的打开文件数：ulimit -a 

使修改的设置生效：ulimit -n 1028

修改后查看生效情况：ulimit -a 

ab -n 1028 -c 1024 http://192.168.0.107/index.html

# Nginx事件处理模型优化

常见的事件处理模型举例

select：

​	select库是在linux和windows平台都支持的事件驱动模型库，并且接口的定义也基本相同，只是部分参数的含义略有差异，最大并发限制1024，是最早期的事件驱动模型

poll：

​	Linux的基本驱动模型，windows不支持此驱动模型，是select的升级版，取消了最大的并发限制，在编译nginx的时候可以使用--with-poll_module和--without-poll_module这两个指定是否编译select库

epoll：

​	epoll库是Nginx服务器支持的最高性能的事件驱动库之一，epoll是poll的升级版，但是与poll的效率有很大的区别。epoll的处理方式是创建一个待处理的事件列表，然后把这个列表发给内核，返回的时候再去轮询检查这个表，以判断事件是否发生，epoll支持一个进程打开的最大事件描述符的上限是系统可以打开的文件的最大数，同时epoll库的IO效率不随描述符数目增加而线性下降，因为它只会对内核上报的“活跃”的描述符进行操作

不同的操作系统会采用不同的I/O模型

​	Linux下，Nginx使用epoll的I/O多路复用模型

​	Windows下，Nginx使用icop的I/O多路复用模型 

linux下指定最佳事件处理模型

```shell
events {
	use epoll;
	multi_accept on;      # 一个进程持续的接收请求
}
```

# 开启高效传输模式

nginx中的“零拷贝”，直接在内核中拷贝

```shell
http {
	sendfile on;
	tcp_nopush on;  # 依赖sendfile开启，协议头、文件内容放在一起，减少网络阻塞
}
```

# 连接超时时间优化

当服务器建立的连接没有接受处理请求时，可以在指定的时间内让它超时自动退出

连接超时的作用

​	将无用的连接设置为尽快超时，可以保护服务器的系统资源（CPU、内存、磁盘）

​	当连接很多时，及时断掉那些建立好的但又长时间不做事的连接，以减少其占用的服务器资源

​	如果黑客攻击，会不断地和服务器建立连接，因此设置连接超时以防止大量消耗服务器的资源

​	如果用户请求了动态服务，则Nginx就会建立连接，请求FastCGI服务以及后端MySQL服务，设置连接超时，使得在用户容忍的时间内返回数据

连接超时存在的问题

​	服务器建立新连接是要消耗资源的，高并发的时候，连接超时时间不宜设置得太长，否则导致服务器会瞬间资源耗尽

​	有些PHP站点会希望设置成短连接，因为PHP程序建立连接消耗的资源和时间相对要少些

​	有些Java站点会希望设置成长连接，因为Java程序建立连接消耗的资源和时间要多一些，这时由语言的运行机制决定的

设置连接超时：http中设置

​	keepalive_timeout：默认秒，该参数用于设置客户端连接保持会话的超时时间，超过这个时间服务器会关闭该连接

​	client_header_timeout：该参数用于设置读取客户端请求头数据的超时时间，如果超时客户端还没有发送完整的header数据，服务器将返回"Request time out(408)"错误

​	client_body_timeout：该参数用于设置读取客户端请求主体数据的超时时间，如果超时客户端还没有发送完整的主体数据，服务器将返回"Request time out(408)"错误

​	send_timeout：用于指定响应客户端的超时时间，如果超过这个时间，客户端没有任何活动，Nginx将会关闭连接

​	tcp_nodely：on，默认情况下当数据发送时，内核并不会马上发送，可能会等待更多的字节组成一个数据包，这样可以提高I/O性能，但是，在每次只发送很少字节的业务场景中，使用tcp_nodelay功能，等待时间会比较长

# fastcgi调优

什么是CGI？

​	CGI又叫通用网关接口，主要是HTTP服务器和其他机器数据通信的工具。一般web Server接受浏览器的请求后，会判断请求资源属静态资源（CSS,JS,JPG）还是动态资源（PHP）；一般静态资源直接返回给浏览器渲染即可，而PHP文件之类的动态资源就需要CGI进行处理了

![image-20210217140954885](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217140954885.png)

为什么选择FastCGI？

​	FastCGI就是为了提高CGI的性能的，每当有PHP文件请求CGI，CGI每次都会进行环境初始化，非常消耗性能，响应速度也非常慢；所以FastCGI出现了，FastCGI会启动一个master进程，master主要负责环境的初始化（只要一次），这样每当有php文件请求的时候，只需要将请求传递给worker进程就行了

![image-20210217141103282](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20210217141103282.png)

FastCGI的各大配置项详解：http模块中配置

​	fastcgi_connect_timeout 240;      # Nginx服务器和后端FastCGI服务器连接的超时时间

​	fastcgi_send_timeout 240;    # Nginx允许FastCGI服务器返回数据的超时时间，即在规定时间内后端服务器必须传完所有的数据，否则Nginx将断开这个连接

​	fastcgi_read_timeout 240; # Nginx从FastCGI服务器读取响应信息的超时时间，表示连接建立成功后，Nginx等待后端服务器的响应时间

​	fastcgi_buffer_size 64k;  # Nginx FastCGI的缓冲区大小，用来读取从FastCGI服务器端收到的第一部分响应信息的缓冲区大小

​	fastcgi_buffers 4 64k;    # 设定用来读取从FastCGI服务器端收到的响应信息的缓冲区大小和缓冲区数量

​	fastcgi_busy_buffers_size 128k;  # 用于设置系统很忙时可以使用的proxy_buffers大小

# gzip调优

Gzip压缩作用

​	将响应报文发送至客户端之前可以启用压缩功能，这能够有效地节约带宽，并提高响应值客户端的速度。Gzip压缩可以配置http,server和location模块下

Gzip修改配置：http模块中配置

​	gzip on;         # 开启gzip压缩功能

​	gzip_min_length 10k;    # 设置允许压缩的页面最小字节数；这里表示如果文件小于10个字节，就不用压缩，因为没有意义，本来就很小

​	gzip_buffers 4 16k;       # 设置压缩缓冲区大小，此处设置为4个16k内存作为压缩结果流缓存

​	gzip_http_version 1.1;  # 压缩版本

​	gzip_comp_level 2;   # 设置压缩比率，最小为1，处理速度快，传输速度慢；9为最大压缩比，处理速度慢，传输速度快；这里表示压缩级别，可以是0到9中的任一个，级别越高，压缩就越小，节省了带宽资源，但同时也消耗CPU资源，所以一般折中为6

​	gzip types text/css text/xml application/javascript;      # 指定压缩的类型，线上配置时尽可能配置多的压缩类型

​	gzip_disable "MSIE [1-6]\.";    # 配置禁用gzip条件，支持正则。此处表示ie6及以下版本不启用gzip（因为ie低版本不支持）

​	gzip vary on;  # 选择支持vary header;该选项可以让前端的缓存服务器缓存经过gzip压缩的页面；这个可以不写，表示在传送数据的时候，根客户端说明我使用了gzip压缩

Gzip注意点

​	Nginx的Gzip压缩功能虽然好用，但是下面两类文件资源不太建议启用此压缩功能：

​		图片类型资源（包括视频文件）

​		大文件资源

# expires缓存调优

expire注意点

​	nginx的缓存设置可以提高网站的性能。对于网站的图片，尤其是新闻站，图片一旦发布，改动的可能是非常小的。我们希望在用户访问一次后，图片缓存在用户的浏览器端，且时间比较长的缓存可以用到nginx的expires设置，nginx中设置过期时间。一般在location里设置

expires优点

​	可以降低带宽，节约成本

​	同时提升用户访问体验

​	减轻服务器的压力，节约服务器成本，是web服务非常重要的功能

缺点

​	被缓存的页面或数据更新了，用户看到的可能还是旧的内容，反而影响用户体验

​	网站流量统计不准确

​	更新频繁的文件

设置

​	expires 30s;   # 30秒、m分钟、h小时、d天

```shell
# 客户端对静态内容缓存
location ~*\.(gif|jpg|jpeg|png|bmp|swf)$ {
	expires 30d;     # 30天
	root html;
}
```

# 防盗链

解决方法

​	水印，品牌宣传；你的带宽、服务器足够

​	防火墙，直接控制，前提是知道IP来源

​	直接给予404的错误提示

两种配置方案

```shell
# 一般
location ~*\.(gif|jpg|png|swf|flv)$ {
	valid_referers none blocked www.xxx.com        # 自己的网站不阻塞
	if ($invalid_referer) {
		rewrite ^/ http://www.xxx.com/error.html;
		# return 403;
	}
}
```

```shell
# 针对图片目录防止盗链
location /images/ {
	alias /data/images/;
	valid_referers none blocked server_names *.xok.la xok.la;
	if ($valid_refers) {return 403}
}
```

# 内核参数优化

修改/etc/sysctl.conf设置相关参数

​	fs.file-max=999999

​	net.ipv4.tcp_max_tw_buckets=6000

​	net.ipv4.ip_local_port_range=1024 65000

​	net.ipv4.tcp_tw_recycle=1

​	net.ipv4.tcp_tw_reuse=1

​	net.ipv4.tcp_keepalive_time=30

​	net.ipv4.tcp_syncookies=1

​	net.core.netdev_max_backlog=262144

​	net.ipv4.tcp_max_syn_backlog=262144

​	net.core.rmem_default=6291456

更改完后执行sysctl -p生效

# 关于系统连接数的优化

