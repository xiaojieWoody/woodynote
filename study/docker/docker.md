# 实践

## MySQl

```shell
docker pull mysql:5.7
docker run --name mysql-dev -p 3306:3306 -v /Users/dingyuanjie/dev_env/docker_image_location/mysql/tmp:/home/data/ -v /Users/dingyuanjie/dev_env/docker_image_location/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
docker exec -it mysql-dev bash
docker cp /Users/dingyuanjie/Downloads/algorithm_bmw.sql mysql-dev:/

docker start mysql-dev
docker exec -it mysql-dev mysql -uroot -p
```

```shell
echo "character-set-server=utf8" >> /etc/mysql/mysql.conf.d/mysqld.cnf

/etc/mysql/my.cnf
# !includedir /etc/mysql/conf.d/
# ...
[mysqld]
datadir=/var/lib/mysql
character-set-server=utf8
default-character-set=utf8
[mysql]
default-character-set=utf8
[client]
default-character-set=utf8        
              
# 获取容器的IP地址         
docker inspect --format '{{ .NetworkSettings.IPAddress }}' mysql-server         
# 验证字符集
mysql -h 172.17.0.2 -uroot -p -e "show variables like '%char%';"
```



## Redis

```shell
docker pull redis
# docker启动redis时一定需要将redis.conf中的daemonize设置成no，否则无法启动docker
# 如果外部需要连接docker内redis，则redis.conf中 protected-mode no且 注释掉bind 127.0.0.1
docker run -d --name redis-dev -p 6379:6379 -v /Users/dingyuanjie/dev_env/docker_image_location/redis/conf/redis.conf:/opt/redis/conf/redis.conf -v /Users/dingyuanjie/dev_env/docker_image_location/redis/data:/data redis redis-server /opt/redis/conf/redis.conf

docker exec -it redis-dev redis-cli -h localhost -p 6379
```

```shell
# redis.conf

# bind 127.0.0.1
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
rdb-del-sync-files no
dir ./
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-diskless-load disabled
repl-disable-tcp-nodelay no
replica-priority 100
acllog-max-len 128
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
lazyfree-lazy-user-del no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
jemalloc-bg-thread yes
```

## nginx

```shell
docker pull nginx:1.18

docker run --name nginx-dev -d -p 81:80 -v /Users/dingyuanjie/dev_env/docker_image_location/nginx/data/html:/usr/share/nginx/html -v /Users/dingyuanjie/dev_env/docker_image_location/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /Users/dingyuanjie/dev_env/docker_image_location/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf -v /Users/dingyuanjie/dev_env/docker_image_location/nginx/log:/var/log/nginx nginx:1.18

docker cp /Users/dingyuanjie/dev_env/docker_image_location/nginx/data/html/index.html nginx-dev:/etc/nginx/html

http://127.0.0.1:81/
```

```shell
# nginx.conf
#user  nobody;
worker_processes  1;
#pid        logs/nginx.pid;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    #gzip  on;
    server {
        listen       80;
        # server_name  192.168.0.104;
        # server_name 192.168.17.50;  
        # server_name 192.168.0.111;    

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
             root   html;
             index  index.html index.htm;
            # proxy_pass         http://192.168.0.35:8081;
            # proxy_pass         http://10.160.143.85:80;
            # proxy_set_header   Host             $host;
            # proxy_set_header   X-Real-IP        $remote_addr;
            # proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
	          # proxy_set_header   X-Forwarded-Proto  $scheme;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

```shell
# default.conf
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## mongodb

```shell
docker pull mongo

docker run -d -p 27017:27017 -v /Users/dingyuanjie/dev_env/docker_image_location/mongo/mongo_configdb:/data/configdb -v /Users/dingyuanjie/dev_env/docker_image_location/mongo/db:/data/db --name mongodb-dev docker.io/mongo --auth

docker exec -it mongodb-dev mongo admin

# 创建管理员账户：
db.createUser({ user: 'admintest', pwd: '123456', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
# 管理员账户进行授权：
db.auth("admintest","123456");
use test;
# 创建test库下的用户：
db.createUser({ user: 'test', pwd: '123456', roles: [{ role: "readWrite", db: "test" }] });
# 连接：
docker exec -it mongodb-dev mongo test
db.auth("test","123456");
db.test.insert({"name":"菜鸟教程"})
db.getCollection('test').find({})
db.getCollection('city').find({})
# 客户端：Robo 3T
```

## centos7.6

```shell
docker pull centos:7.6.1810
docker run -itd --name centos7.6-dev centos:7.6.1810 /bin/bash
docker ps
docker exec -it centos7.6-dev /bin/bash
# 查看centos系统信息
cat /etc/os-release
# 安装配置ssh
yum install passwd openssl net-tools openssh-server -y
# 启动sshd，会报以下错误：
[root@d54753fe138a /]# /usr/sbin/sshd
Could not load host key: /etc/ssh/ssh_host_rsa_key
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
Could not load host key: /etc/ssh/ssh_host_ed25519_key
# 执行以下命令解决
[root@d54753fe138a /]# ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''    
[root@d54753fe138a /]# ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''
[root@d54753fe138a /]# ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key  -N ''
# 修改sshd配置，执行vi /etc/ssh/sshd_config
UsePAM yes 改为 UsePAM no；
UsePrivilegeSeparation sandbox 改为 UsePrivilegeSeparation no
# 执行passwd给root用户设置登录密码
passwd
123456
# 安装ip命令
yum install iproute -y
# 查看系统ip：
[root@136a7725e62f /]# ifconfig
[root@136a7725e62f /]# ip addr | grep eth0
14: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
    
# 打包容器为镜像
 docker ps
 docker commit centos7.6-dev local/centos7.6
 docker images
 # 启动容器
 # /usr/sbin/sshd -D ——执行容器的/usr/sbin/sshd命令，-D将sshd作为前台进程运行，而不是脱离控制台成为后台守护进程
# docker run -itd --name centos7-dev -p 10033:22 -p 10034:8080 -p 10035:8081 -v /Users/dingyuanjie/dev_env/docker_image_location/centos7/data:/root/data local/centos7.6 /usr/sbin/sshd -D 
 docker run -itd --name centos7-dev -p 10033:22 -v /Users/dingyuanjie/dev_env/docker_image_location/centos7/data:/root/data local/centos7.6 /usr/sbin/sshd -D
 # 进入centos容器内
 docker exec -it centos7-dev /bin/bash
 # 远程连接
 ssh root@localhost -p 10033
 123456
 
 # vim中文乱码
vim /etc/vimrc 
# 该文件头上添加下面四行代码
set fileencodings=utf-8,gb2312,gbk,gb18030
set termencoding=utf-8
set fileformats=unix
set encoding=prc
```

## ubuntu18.04

```shell
docker pull ubuntu:18.04
docker run -itd --name ubuntu18.04-tmp ubuntu:18.04 /bin/bash
docker exec -it ubuntu18.04-tmp /bin/bash
apt update
apt install passwd openssl iproute2 net-tools openssh-server vim iputils-ping -y
vim /etc/ssh/sshd_config
# 将PermitRootLogin yes的#号去掉，并且保存退出文件
# 执行passwd给root用户设置登录密码
passwd root
123456
# 一般情况下安装完成后就会自动启动，先检查下进程 ps aux|grep ssh，如果出现sshd进程表示已启动，否则执行命令：sudo service ssh start，如果出现Could not load host key错误，使用命令：ssh-keygen -A，然后sudo service ssh restart重启即可
# 查看服务器ip
ifconfig
ip a
# 打包容器为镜像
docker commit ubuntu18.04-tmp local/unbutu:18.04
 # 启动容器
docker run -itd --name ubuntu18-dev  -p 10034:22 -v /Users/dingyuanjie/dev_env/docker_image_location/ubuntu18.04/data:/root/data/ local/unbutu:18.04 /usr/sbin/sshd -D
# 进入容器内
 docker exec -it ubuntu18-dev /bin/bash
# 远程连接
ssh root@localhost -p 10034
123456
```

## 容器内安装软件

```shell
echo -e "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >> /etc/apt/sources.list

apt-get update
apt-get install vim -y 
```

