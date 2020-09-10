![image-20200904202356719](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200904202356719.png)

# 1. 为什么要用容器？

* **上线流程繁琐**
  * 开发->测试->申请资源->审批->部署->测试等环节
* **资源利用率低**
  * 普遍服务器利用率低，造成过多浪费
* **扩容/缩容不及时**
  * 业务高峰期扩容流程繁琐，上线不及时
* **服务器环境臃肿**
  * 服务器越来越臃肿，对维护、迁移带来困难
* 微服务环境治理
* 高效CI/CD
* 轻量级

## *Docker设计目标：*

* 提供简单的应用程序打包工具
* 开发人员和运维人员职责逻辑分离
* 多环境保持一致性

## **Kubernetes设计目标**

* 集中管理所有容器
* 资源编排
* 资源调度
* 弹性伸缩
* 资源隔离

## 容器 VS 虚拟机

![image-20200904212944827](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200904212944827.png)

![image-20200904213016532](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200904213016532.png)

```shell
#							Container													VM
启动速度			 秒级																分钟级
运行性能			 接近原生														 5%左右损失
磁盘占用			 MB                                GB
数量          成百上千                            一般几十台
隔离性        进程级                              系统级（更彻底）
操作系统       主要支持Linux                       几乎所有
封装程度       只打包项目和依赖关系，共享宿主机内核    完整的操作系统
```

# 2. Docker基本使用

## CentOS7.x安装Docker

```shell
# 时间同步
ntpdate time.windows.com
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 配置国内yum源头
cd /etc/yum.repos.d
# yum install epel-release -y
wget http://mirrors.aliyun.com/repo/Centos-7.repo
wget http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache

# 禁止swap分区
vi /etc/fstab
# 注释掉
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
# reboot重启 
free -m    # 都显示0则关闭成功

# 关闭防火墙：
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux：
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 安装Docker CE
yum install -y docker-ce

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# 启动Docker服务并设置开机启动
systemctl start docker
systemctl enable docker

# 官方文档：https://docs.docker.com
# 阿里云源：http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

docker inspect image nginx
```

## **镜像是什么？**

* 一个分层存储的文件
* 一个软件的环境
* 一个镜像可以创建N个容器
* 一种标准化的交付
* 一个不包含Linux内核而又精简的Linux操作系统
* 镜像不是一个单一的文件，而是有多层构成。可以通过`docker history <ID/NAME>` 查看镜像中各层内容及大小，每层对应着`Dockerfile`中的一条指令。Docker镜像默认存储在`/var/lib/docker/<storage-driver>`中
* 配置镜像加速器：https://www.daocloud.io/mirror
* curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

```shell
docker info 
docker --version
docker version

# Docker Root Dir: /var/lib/docker
cd /var/lib/docker/overlay2
# 后面会被来越大，最后使用软连接
```

## **理解容器镜像**

* 容器其实是在镜像的最上面加了一层读写层，在运行容器里文件改动时，会先从镜像里要写的文件复制到容器自己的文件系统中（读写层）。如果容器删除了，最上面的读写层也就删除了，改动也就丢失了。所以无论多少个容器共享一个镜像，所做的写操作都是从镜像的文件系统中复制过来操作的，并不会修改镜像的源文件，这种方式提高磁盘利用率
* 若想持久化这些改动，可以通过`docker commit` 将容器保存成一个新镜像
  * 一个镜像创建多个容器
  * 镜像增量式存储
  * 创建的容器里面修改不会影响到镜像

```shell
cgroups 资源限制:比如CPU/内存
namespace 资源隔离：进程、文件系统、用户等
ufs 联合文件系统：镜像增量式存储，提高磁盘利用率
```

## **管理镜像常用命令表**

```shell
# 列出镜像
ls
# 构建镜像来自Dockerfile
build
# 查看镜像历史
history
# 显示一个或多个镜像详细信息
inspect
# 从镜像仓库拉取镜像
pull
# 推送一个镜像到镜像仓库
push
# 移除一个或多个镜像
rm
rmis
# 移除未使用的镜像。没有被标记或被任何容器引用的
prune
# 创建一个引用源镜像标记目标镜像
tag
# 导出容器文件系统到tar归档文件
export
# 导入容器文件系统tar归档文件创建镜像
import
# 保存一个或多个镜像到一个tar归档文件
save
# 加载镜像来自tar归档或标准输入
load
```

## **创建容器常用选项表**

```shell
# 交互式
-i, –interactive
# 分配一个伪终端
-t, –tty
# 运行容器到后台
-d, –detach
# 设置环境变量
-e, –env
# 发布容器端口到主机
-p, –publish list
# 发布容器所有EXPOSE的端口到宿主机随机端口
-P, –publish-all
# 指定容器名称
--name string
# 设置容器主机名
-h, –hostname
# 指定容器IP，只能用于自定义网络
–ip string
# 连接容器到一个网络
--network
# 将文件系统附加到容器
--mount mount
# 绑定挂载一个卷
-v, –volume list
# 容器退出时重启策略，默认no，可选值：[always|on-failure]
--restart string

# 启动容器后，如果没有守护进程，则容器会退出
docker run -d --name hello -p 88:80 -e test=123456 --restart=always nginx
docker exec -it hello bash

# 查看容器运行了哪些进程
docker top hello

# nginx -g daemon off;
global全局参数
daemon守护进程
off关闭，前台一直运行，保持容器不会退出
on后台运行

docker run -d centos
docker top centos       # 没有守护进程，容器没有起来，退出了
docker run -itd centos
docker ps -l            
docker top centos       # 有创建伪终端进程，容器起来了

# 删除所有容器
docker rm -f $(docker ps -qa)
```

## **容器资源限制参数表**

```shell
# 容器可以使用的最大内存量
-m，–memory
# 允许交换到磁盘的内存量
–memory-swap
# 容器使用SWAP分区交换的百分比（0-100，默认为-1）
–memory-swappiness=<0-100>
# 禁用OOM Killer
–oom-kill-disable
# 可以使用的CPU数量
--cpus
# 限制容器使用特定的CPU核心，如(0-3, 0,1)
–cpuset-cpus
# CPU共享（相对权重）
–cpu-shares

# 默认没有限制，可以使用宿主机所有资源
# 可限制内存、CPU
```

```shell
# 内存限额：
# 允许容器最多使用500M内存和100M的Swap，并禁用 OOM Killer：
docker run -d --name nginx03 --memory="500m" --memory-swap="600m" --oom-kill-disable nginx

# 查看容器占用资源 
docker stats nginx03

# ab压测
yum install httpd-tools -y
docker inspect nginx03   # 查看"IPAddress": "172.17.0.2",
# 1000个用户，100000个请求
ab -c 1000 -n 100000 http://172.17.0.2/index.html

# CPU限额：
# 允许容器最多使用一个半的CPU：
docker run -d --name nginx04 --cpus="1.5" nginx
# 允许容器最多使用50%的CPU：
docker run -d --name nginx05 --cpus=".5" nginx
```

## **管理容器常用命令表**

```shell
# 列出容器
ls
# 查看一个或多个容器详细信息
inspect
# 在运行容器中执行命令
exec
# 创建一个新镜像来自一个容器
commit
# 拷贝文件/文件夹到一个容器
cp
# 获取一个容器日志
logs
# 列出或指定容器端口映射
port
# 显示一个容器运行的进程, docker top containerID/containerName
top
# 显示容器资源使用统计，docker stats containerID/containerName
stats
# 停止/启动一个或多个容器
stop/start/restart
# 删除一个或多个容器
rm
```

## **持久化容器中应用程序数据**

```shell
需要持久化的数据：
1、日志，一般用于方便日志采集和故障排查
2、配置文件，比如nginx配置文件
3、业务数据，比如mysql，网站程序
4、临时缓存数据，比如nginx-proxy-cache
```

* Docker提供三种方式将数据从宿主机挂载到容器中：
  * `volumes`：Docker管理宿主机文件系统的一部分（`/var/lib/docker/volumes`）。保存数据的最佳方式
  * `bind mounts`：将宿主机上的任意位置的文件或者目录挂载到容器中
  * `tmpfs`：挂载存储在主机系统的内存中，而不会写入主机的文件系统。如果不希望将数据持久存储在任何位置，可以使用`tmpfs`，同时避免写入容器可写层提高性能

![image-20200904215956406](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200904215956406.png)

```shell
docker volume ls
docker volume create www
docker volume ls
docker volume inspect www
ls /var/lib/docker/volumes/www/_data/
docker run -d -p 88:80 -v www:/usr/share/nginx/html nginx
ls /var/lib/docker/volumes/www/_data/
```

```shell
-v 数据卷名称或者源目录:容器目标
bind mounts注意点:
1、宿主机文件或者目录必须存在才能成功挂载
2、宿主机文件或者目录覆盖容器中内容
```

# 3. **Dockerfile 构建常见基础镜像**

```shell
镜像分类：
1、基础镜像，例如centos(yum)、ubuntu(apt)、alpine(apk)
2、环境镜像，例如php、jdk、python
3、项目镜像，打包好的可部署镜像

制作镜像：
1、选择一个符合线上操作系统的基础镜像
2、用基础镜像创建一个容器，手动在容器里面跑一边你要部署的应用
3、确认你启动这个应用的前台运行命令
```

```dockerfile
FROM centos:latest
LABEL maintainer woodyfine
RUN yum install gcc -y
COPY run.sh /usr/bin
EXPOSE 80
CMD [“run.sh”]
```

```shell
# 构建新镜像是基于哪个镜像
FROM
# 标签
LABEL
# 构建镜像时运行的Shell命令
RUN
# 拷贝文件或目录到镜像中
COPY
# 设置环境变量
ENV
# 为RUN、CMD和ENTRYPOINT执行命令指定运行用户
USER
# 声明容器运行的服务端口
EXPOSE
# 为RUN、CMD、ENTRYPOINT、COPY和ADD设置工作目录
WORKDIR
# 运行容器时执行，如果有多个ENTRYPOINT指令，最后一个生效
ENTRYPOINT
# 运行容器时执行，如果有多个CMD指令，最后一个生效
CMD
```

```shell
编写Dockerfile的技巧：
1、如果追求镜像更小，选择alpine
2、运行的Shell命令尽可能写到一个RUN里面，减少镜像层
3、清理部署时产生留的缓存或者文件
```

```dockerfile
# 源码安装：
0. 安装依赖包，例如gcc、make
1、./configure
2、make     #  make -j 4 多线程配置，效率高
3、make install

# 构建nginx基础镜像
# nginx/Dockerfile
FROM centos:7
LABEL maintainer www.ctnrs.com
RUN yum install -y gcc gcc-c++ make \
    openssl-devel pcre-devel gd-devel \
    iproute net-tools telnet wget curl && \
    yum clean all && \
    rm -rf /var/cache/yum/*

COPY nginx-1.15.5.tar.gz /
RUN tar zxf nginx-1.15.5.tar.gz && \
    cd nginx-1.15.5 && \
    ./configure --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_stub_status_module && \
    make -j 4 && make install && \
    rm -rf /usr/local/nginx/html/* && \
    echo "ok" >> /usr/local/nginx/html/status.html && \
    cd / && rm -rf nginx* && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

ENV PATH $PATH:/usr/local/nginx/sbin
COPY nginx.conf /usr/local/nginx/conf/nginx.conf
WORKDIR /usr/local/nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

# nginx.conf
user                 root;
worker_processes     4;
worker_rlimit_nofile 65535;

error_log  logs/error.log  notice;

pid        /var/run/nginx.pid;

events {
    use epoll;
    worker_connections  4096;
}

http {

    include       mime.types;
    default_type  application/octet-stream;

    log_format  main '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log off;
    keepalive_timeout  65;

    client_max_body_size         64m;
    server {
        listen 80;
        server_name _;
        index index.html;

        location / {
            root html;
        }
    }
}

[root@master nginx]# docker build . -t nginx:v1
[root@master nginx]# docker run -d -p 81:80 nginx:v1
http://192.168.0.31:81/status.html
```

```shell
# 构建tomcat基础镜像
# tomcat/Dockerfile

FROM centos:7
MAINTAINER www.ctnrs.com

ENV VERSION=8.5.43

RUN yum install java-1.8.0-openjdk wget curl unzip iproute net-tools -y && \
    yum clean all && \
    rm -rf /var/cache/yum/*

COPY apache-tomcat-${VERSION}.tar.gz /
RUN tar zxf apache-tomcat-${VERSION}.tar.gz && \
    mv apache-tomcat-${VERSION} /usr/local/tomcat && \
    rm -rf apache-tomcat-${VERSION}.tar.gz /usr/local/tomcat/webapps/* && \
    mkdir /usr/local/tomcat/webapps/test && \
    echo "ok" > /usr/local/tomcat/webapps/test/status.html && \
    sed -i '1a JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom"' /usr/local/tomcat/bin/catalina.sh && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    
ENV PATH $PATH:/usr/local/tomcat/bin

WORKDIR /usr/local/tomcat

EXPOSE 8080
CMD ["catalina.sh", "run"]

[root@master tomcat]# docker build . -t tomcat:v1
[root@master tomcat]# docker run -d -p 8081:8080 tomcat:v1
http://192.168.0.31:8081/test/status.html
```

```shell
# 构建jdk基础镜像
# java/Dockerfile
FROM java:8-jdk-alpine
LABEL maintainer www.ctnrs.com
ENV JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF8 -Duser.timezone=GMT+08"
RUN  apk add -U tzdata && \
     ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY ./target/eureka-service.jar ./
EXPOSE 8888
CMD java -jar $JAVA_OPTS /eureka-service.jar
```

# 4. **企业级 Harbor 镜像仓库**

* 管理用户界面，基于角色的访问控制 ，AD/LDAP集成以及审计日志等，足以满足基本企业需求

  ![image-20200904220631680](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200904220631680.png)

* 安装docker与docker-compose

  ```shell
  # 解压离线包部署
  # slave2上
  [root@slave2 ~]# yum install docker-compose -y
  # 验证
  [root@slave2 ~]# rpm -qa docker-compose
  [root@slave2 ~]# tar -zxvf harbor-offline-installer-v1.9.1.tgz
  [root@slave2 ~]# mv harbor harbor-1.9.1
  cd harbor-1.9.1
  vi harbor.yml
  hostname: 192.168.0.33
  [root@slave2 harbor-1.9.1]# ./prepare
  [root@slave2 harbor-1.9.1]# ./install.sh
  [root@slave2 harbor-1.9.1]# docker-compose ps
  
  # 所有机器添加  harbor配置的hostname
  # 配置http镜像仓库可信任
  [root@master ~]# vi /etc/docker/daemon.json
  "insecure-registries": ["192.168.0.33"]
  # {"insecure-registries":["reg.ctnrs.com"]}
  [root@master ~]# systemctl restart docker
  
  # 访问
  http://192.168.0.33
  
  # 修改了harbor.yaml后要重新./install.sh
  
  [root@slave2 harbor-1.9.1]# docker-compose stop
  [root@slave2 harbor-1.9.1]# docker-compose up -d
  
  # 创建用户 test(Iamceshi0108)、dev(Iamkaifa0108)
  # 创建非公开项目 
  test，添加test用户
  dev，添加dev用户
  
  # master上
  [root@master ~]# docker login 192.168.0.33
  Username: admin
  Password: Harbor12345
  # 打标签
  [root@master ~]# docker tag tomcat:v1 192.168.0.33/library/tomcat:v1
  # 上传
  [root@master ~]# docker push 192.168.0.33/library/tomcat:v1
  # 下载
  [root@master ~]# docker pull 192.168.0.33/library/tomcat:v1
  [root@master ~]# docker tag nginx:v1 192.168.0.33/library/nginx:v1
  [root@master ~]# docker push 192.168.0.33/library/nginx:v1
  ```



# 5. **基于 Docker 构建企业 Jenkins CI 平台**

* 持续集成（Continuous Integration，CI）：代码合并、构建、部署、测试都在一起，不断地执行这个过程，并对结果反馈
* 持续部署（Continuous Deployment，CD）：部署到测试环境、预生产环境、生产环境
* 持续交付（Continuous Delivery，CD）：将最终产品发布到生产环境，给用户使用
* 开发-编译-测试-部署/升级
* 高效的CI/CD环境可以获得
  * 及时发现问题
  * 大幅度减少故障率
  * 加快迭代速度
  * 减少时间成本

![image-20200905103707565](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200905103707565.png)

## 部署gitlab

```shell
# slave2上  4G2核内存
docker run -d \
  --name gitlab \
  -p 8443:443 \
  -p 9999:80 \
  -p 9998:22 \
  -v $PWD/config:/etc/gitlab \
  -v $PWD/logs:/var/log/gitlab \
  -v $PWD/data:/var/opt/gitlab \
  -v /etc/localtime:/etc/localtime \
  lizhenliang/gitlab-ce-zh:latest
  gitlab/gitlab-ce:latest
  
# 访问  
http://192.168.0.33:9999
root/Iamgitlab0108
# 创建项目
项目名称：java-demo
私有

[root@slave2 ~]# yum install git -y
[root@slave2 ~]# unzip tomcat-java-demo-master.zip
[root@slave2 ~]# cd tomcat-java-demo-master
[root@slave2 tomcat-java-demo-master]# git init
[root@slave2 tomcat-java-demo-master]# cat .git/config
[root@slave2 tomcat-java-demo-master]# git remote add origin http://192.168.0.33:9999/root/java-demo.git
[root@slave2 tomcat-java-demo-master]# cat .git/config
[root@slave2 tomcat-java-demo-master]# git add .
[root@slave2 tomcat-java-demo-master]# git config --global user.email "1093520060@qq.com"
[root@slave2 tomcat-java-demo-master]# git config --global user.name "woodyfine"
[root@slave2 tomcat-java-demo-master]# git commit -m "init"
[root@slave2 tomcat-java-demo-master]# git push origin master
root/Iamgitlab0108
```

## 部署Jenkins

```shell
# slave上
[root@slave2 ~]# tar -zxvf jdk-8u45-linux-x64.tar.gz
[root@slave2 ~]# tar -zxvf apache-maven-3.5.0-bin.tar.gz
[root@slave2 ~]# vi apache-maven-3.5.0/conf/setting.xml
<mirror>  
  <id>alimaven</id>  
  <name>aliyun maven</name>  
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
  <mirrorOf>central</mirrorOf>          
</mirror>
[root@slave2 ~]# mv apache-maven-3.5.0 /usr/local/maven
[root@slave2 ~]# mv jdk1.8.0_45/ /usr/local/jdk

docker run -d --name jenkins -p 88:8080 -p 50000:50000 -u root  \
   -v /opt/jenkins_home:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock   \
   -v /usr/bin/docker:/usr/bin/docker \
   -v /usr/local/maven:/usr/local/maven \
   -v /usr/local/jdk:/usr/local/jdk \
   -v /etc/localtime:/etc/localtime \
   --name jenkins jenkins/jenkins:lts
   
# 访问
http://192.168.0.33:88/
[root@slave2 ~]# docker ps | grep jenkins
d62a3d6abd2c        jenkins/jenkins:lts                                      "/sbin/tini -- /usr/…"   2 minutes ago       Up 2 minutes              0.0.0.0:50000->50000/tcp, 0.0.0.0:88->8080/tcp                      jenkins
[root@slave2 ~]# docker logs d62a3d6abd2c
[root@slave2 ~]# docker exec -it d62a3d6abd2c bash
root@d62a3d6abd2c:/# cat /var/jenkins_home/secrets/initialAdminPassword
69148ba154d04b6687b70f2be2d48b8e
root@d62a3d6abd2c:/# exit

# 先不安装插件
admin/123456

# 安装插件
# Manager Plugins - Advanced - Update Site URL
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

# 修改国内源：
[root@slave2 ~]# cd /opt/jenkins_home/updates
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && \
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
# 然后重启jenkins容器生效
[root@slave2 updates]# docker restart jenkins
[root@slave2 updates]# docker logs jenkins

# Manager Plugins - Available
git、pipeline

# 创建项目 - demo - pipeline
```

## CI流程

1. 拉取代码
2. 代码编译（java项目），产出war包
3. 打包项目镜像并推送到镜像仓库
4. 部署镜像测试

* Pipeline脚本

```shell
# 自动生成Jenkins脚本
# Pipeline - 点击 Pipeline Syntax - 选择 Checkout from version control
Repository URL : http://192.168.0.33:9999/root/java-demo.git
Credentials - Add - 全局凭证 - username(gitlab):root password:Iamgitlab0108
# Generate Pipeline Script

checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '5d22b38c-4f6b-477d-9cd0-2e6041af8bc8', url: 'http://192.168.0.33:9999/root/java-demo.git']]])
```



```shell
# 新建变量
# General - 勾选This project is parameterized - String Parameter
Branch
master
发布的分支代码

# Jenkins - 凭据 - 系统 - 全局凭据 - Add Credentials
admin
Harbor12345
harbor-auth
# git-auth
32c3ca28-5d6a-469f-8ca7-8fed37befc33
# harbor-auth
a2965746-84f6-46a1-8038-919bfc336fc5

# pipeline script

#!/usr/bin/env groovy

def registry = "192.168.0.33"
def project = "dev"
def app_name = "java-demo"
def image_name = "${registry}/${project}/${app_name}:${Branch}-${BUILD_NUMBER}"
def git_address = "http://192.168.0.33:9999/root/java-demo.git"
def docker_registry_auth = "4558af14-5aa2-4099-9286-3cc960d28c72"
def git_auth = "5d22b38c-4f6b-477d-9cd0-2e6041af8bc8"

pipeline {
    agent any
    stages {
        stage('拉取代码'){
            steps {
              checkout([$class: 'GitSCM', branches: [[name: '${Branch}']], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
            }
        }

        stage('代码编译'){
           steps {
             sh """
             	  pwd
             	  ls
                JAVA_HOME=/usr/local/jdk
                PATH=$JAVA_HOME/bin:/usr/local/maven/bin:$PATH
                mvn clean package -Dmaven.test.skip=true
                """ 
           }
        }

        stage('构建镜像'){
           steps {
                withCredentials([usernamePassword(credentialsId: "${docker_registry_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                sh """
                  echo '
                    FROM ${registry}/library/tomcat:v1
                    LABEL maitainer lizhenliang
                    RUN rm -rf /usr/local/tomcat/webapps/*
                    ADD target/*.war /usr/local/tomcat/webapps/ROOT.war
                  ' > Dockerfile
                  docker build -t ${image_name} .
                  docker login -u ${username} -p '${password}' ${registry}
                  docker push ${image_name}
                """
                }
           } 
        }

        stage('部署到Docker'){
           steps {
              sh """
              REPOSITORY=${image_name}
              docker rm -f tomcat-java-demo |true
              docker container run -d --name tomcat-java-demo -p 89:8080 ${image_name}
              """
            }
        }
    }
}
```

```shell
# 访问
http://192.168.0.33:89/

# 安装数据库
[root@slave2 maven]# yum install mariadb-server -y
[root@slave2 maven]# systemctl start mariadb
[root@slave2 maven]# mysqladmin -uroot password '123456'
[root@slave2 maven]# mysql -uroot -p
MariaDB [(none)]> use test;
MariaDB [test]> source /root/tomcat-java-demo-master/db/tables_ly_tomcat.sql
MariaDB [test]> show tables;
MariaDB [test]> grant all on *.* to "root"@"%" identified by "123456";

# 更新代码
[root@slave2 resources]# vi /root/tomcat-java-demo-master/src/main/resources/application.yml
spring:
  datasource:
    url: jdbc:mysql://192.168.0.33:3306/test?characterEncoding=utf-8
    username: root
    password: 123456
[root@slave2 resources]# git add .
[root@slave2 resources]# git commit -m "mysql"
[root@slave2 resources]# git push
Username for 'http://192.168.0.33:9999': root
Password for 'http://root@192.168.0.33:9999': Iamgitlab0108

# jenkins上构建
# 访问，添加用户
http://192.168.0.33:89/
```

# 6. **Prometheus+Grafana 监控 Docker**

## **cAdvisor** （Container Advisor） **：**

* 用于收集正在运行的容器资源使用和性能信息

* https://github.com/google/cadvisor

  ```shell
  # slave2
  docker run -d \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
  
  # 访问
  http://192.168.0.33:8080/
  ```

## **Prometheus**（普罗米修斯）：

* 容器监控系统

* https://prometheus.io
* https://github.com/prometheus

```yaml
# slave2
#/tmp/prometheus.yml

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['192.168.0.33:8080']
```

```shell
# slave2
# 启动
docker run -d --name=prometheus -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

# 访问
http://192.168.0.33:9090/
```

## **Grafana**：

* 是一个开源的度量分析和可视化系统

* https://grafana.com/grafana/download
* https://grafana.com/dashboards/193 （监控Docker主机模板）

```shell
# slave2
[root@slave2 tmp]# docker run -d --name=grafana -p 3000:3000 grafana/grafana

# 访问
http://192.168.0.33:3000
admin/admin
admin/123456
# Add data source
## Prometheus
HTTP URL：http://192.168.0.33:9090
## Import
### Import via grafana.com
193   # Load
选择 Prometheus
```

