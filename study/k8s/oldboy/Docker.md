# Docker好处

* 使用Docker容器化封装应用程序的意义
  * Docker引擎统一了基础设施环境-docker环境
    * 硬件的配置
    * 操作系统的版本
    * 运行时环境的异构
  * Docker引擎统一了程序打包（装箱）方式-docker镜像
    * Java程序
    * python程序
    * nodejs程序
  * Docker引擎统一了程序部署（运行）方式-docker容器
    * java -jar ... -> docker run ...
    * python manage.py runserver ... -> docker run ...
    * npm run dev -> docker run ...

# 环境准备

## 安装

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum install epel-release -y
yum list docker --show-duplicates
yum install -y yum-utils
yum-config-manager -add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce/repo
yum list docker-ce --show-duplicates

yum install docker-ce
systemctl enable docker
systemctl start docker
```

## 配置镜像地址

```shell
vi /ect/docker/daemon.json
{
	"graph":"/data/docker",
	"storage-driver":"overlay2",
	"insecure-registries":["registry.access.redhat.com","quay.io"],
	"registry-mirrors":["https://q2gr04ke.mirror.aliyuncs.com"],
	"bip":"172.7.244.0/24",
	"exec-opts":["native.cgroupdriver=systemd"],
	"live-restore":true
}

# bip给容器设置IP，方便排错，根据容器IP即可找到节点
# 规律：
# 宿主机IP的第四位为bip的第三位
# 如 10.4.7.21  为 172.7.21.1/24

# 备用
"registry-mirrors": ["https://0eipo2bm.mirror.aliyuncs.com"]
```

```shell
docker info
docker run hello-world

# 镜像的结构
${registry_name}/${repository_name}/${image_name}:${tag_name}
如：docker.io/library/alpine:3.10.1

docker login
# docker hub
94woodyfine

# 登陆信息存放位置
cat /root/.docker/config.json

docker search alpine

docker images

docker tag ImageID docker.io/94woodyfine/alpine:v3.10.3

# 推送到仓库
docker push docker.io/94woodyfine/alpine:v3.10.3

# 删除镜像
docker rmi -f ImageID

# 本地容器 -a 包括退出的
docker ps -a

# 启动容器
docker run [OPTIONS] IMAGE [COMMAND][ARG..]
OPTIONS:选项
-i 可交互，并持续打开标准输入
-t 使用终端关联到容器的标准输入输出上
-d 后台运行
-rm 退出后删除容器
-name 定义容器唯一名称

docker run -it 94woodyfine/alpine:v3.10.3 /bin/sh
docker run --rm 94woodyfine/alpine:v3.10.3 /bin/echo hello
docker run -d --name myalpine 94woodyfine/alpine:v3.10.3 /bin/sleep 300

ps aux|grep sleep|grep -v grep

# 复制到容器内
docker cp tmp.txt ContainerID:/
# 进入容器
docker exec -it ContainerID/ContainerName /bin/sh
# 停止容器
docker stop ContainerID/ContainerName
# 启动容器
docker start ContainerID/ContainerName
docker restart ContainerID/ContainerName
# 删除容器
docker rm ContainerID/ContainerName
docker rm -f ContainerID/ContainerName
# 查看容器信息
docker inspect containerID

# 删除退出的容器
for i in `docker ps -a|grep -i exit|awk '{print $1}'`;do docker rm -f $i;done

# commit制作镜像
docker commit -p myalpine 94woodyfine/alpine:v3.10.3_with_hello.txt
docker images
docker run -it 94woodyfine/alpine:v3.10.3_with_hello.txt /bin/sh

# 导出镜像
docker save ImageID > alpine:v3.10.3_with_hello
docker rmi -f ImageID
# 导入镜像
docker load < alpine\:v3.10.3_with_hello.txt
docker images
docker tag 257f0bd86214 94woodyfine/alpine:v3.10.3_with_hello.txt
docker run -it --rm --name myalpine_with_hello.txt 257f0bd86214 /bin/sh

# 查看容器日志
docker run hello-world 2>&1 >> /dev/null
docker logs containerID
```

# 高级内容

## 映射端口

* `docker run -p 容器外端口:容器内端口`

  ```shell
  docker pull nginx:1.12.2
  docker tag 4037a5562b03 94woodyfine/nginx:v1.12.2
  docker run --rm --name mynginx -d -p 81:80 94woodyfine/nginx:v1.12.2
  netstat -luntp|grep 81
  curl 127.0.0.1:81
  ```

## 挂载数据卷

* `docker run -v 容器外目录:容器内目录`

  ```shell
  mkdir html
  cd html/
  wget www.baidu.com -O index.html
  
  docker run -d --rm --name nginx_with_baidu -p 82:80 -v /root/html:/usr/share/nginx/html 94woodyfine/nginx:v1.12.2
  
  http://192.168.0.244:82/
  
  docker exec -it nginx_with_baidu /bin/sh
  cd /usr/share/nginx/html 
  
  docker inspect containerID
  ```

## 传递环境变量

* `docker run -e 环境变量key=环境变量value`

  ```shell
  docker run --rm -e E_OPTS=abcdefg -e C_OPTS=abcdefg 94woodyfine/alpine:v3.10.3 printenv
  ```

## 容器内安装软件

* `yum/apt-get/apt等`

  ```shell
  docker exec -it ContainerID /bin/sh
  
  tee /etc/apt/sources.list << EOF
  deb http://mirrors.163.com/debian/ jessie main non-free contrib
  deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
  EOF
  
  apt-get update && apt-get install curl -y
  
  curl -k https://www.baidu.com
  
  docker commit -p nginx_with_baidu 94woodyfine/nginx:curl
  
  docker push 94woodyfine/nginx:curl
  ```

# 容器生命周期

* 检查本地是否存在镜像，如果不存在即从远端仓库检索
* 利用镜像启动容器
* 分配一个文件系统，并在只读的镜像层外挂载一层可读写层
* 从宿主机配置的网桥接口中桥接一个虚拟接口到容器
* 从地址池配置一个ip地址给容器
* 执行用户指定的命令
* 执行完毕后容器终止

# Dockerfile

* 制作镜像途径

  * docker commit

  * Dockerfile

    ![image-20200809091508214](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809091508214.png)

* ![image-20200809091544803](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809091544803.png)

## USER/WORKDIR指令

* USER指定用户
* WORKDIR相当于cd

```shell
cd /data/dockerfile/demo_1
vim Dockerfile
	FROM 94woodyfine/nginx:curl
	USER nginx
	WORKDIR /usr/share/nginx/html
[root@master demo_1]# docker build . -t docker.io/94woodyfine/nginx:curl_with_user_workdir
[root@master demo_1]# docker run --rm -ti --name nginx123 94woodyfine/nginx:curl_with_user_workdir /bin/bash
nginx@fc0ec1842261:/usr/share/nginx/html$ whoami
nginx
nginx@fc0ec1842261:/usr/share/nginx/html$ pwd
/usr/share/nginx/html
```

## ADD/EXPOSE指令

```shell
vim Dockerfile

FROM 94woodyfine/nginx:curl
ADD index.html /usr/share/nginx/html/index/html
EXPOSE 80

ls
# Dockerfile  index.html
docker build . -t  94woodyfine/nginx:curl_add_expose
docker run --rm -it --name nginx123 -P 94woodyfine/nginx:curl_add_expose /bin/bash
	# 启动nginx
	root@654c08836760:/# nginx -g "daemon off;"
	
# -P 和 EXPOSE 配合	，宿主器随机启动个端口和容器暴露的端口进行映射
docker run --rm -d --name nginx123 -P 94woodyfine/nginx:curl_add_expose

# 宿主机
netstat -luntp
	tcp6       0      0 :::32768                :::*                    LISTEN      15022/docker-proxy
curl 127.0.0.1:32768

http://192.168.0.244:32768/
```

## RUN/ENV指令

* RUN：构建镜像时执行的命令

```shell
FROM centos:7
ENV VER 9.11.4
RUN yum install bind-$VER -y

docker search centos

docker build . -t 94woodyfine/bind:v9.11.4_with_env_run

[root@master demo_3]# docker run -it --rm --name test 94woodyfine/bind:v9.11.4_with_env_run /bin/bash
[root@1b7664c441bf /]# cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
[root@1b7664c441bf /]# printenv
```

## CMD/ENTRYPOINT指令

* CMD：要启动容器时，执行的命令

```shell
FROM centos:7
RUN yum install httpd -y
CMD ["httpd","-D", "FOREGROUND"]

docker build . -t 94woodyfine/httpd:test

docker run -d --rm --name myhttpd -p83:80 94woodyfine/httpd:test

http://192.168.0.244:83/
```

```shell
FROM centos:7
ADD entrypoint.sh /entrypoint.sh
RUN yum install epel-release -q -y && yum install nginx -y
ENTRYPOINT /entrypoint.sh

# entrypoint.sh
#!/bin/bash

/sbin/nginx -g "daemon off;"

chmod +x entrypoint.sh

docker build . -t 94woodyfine/nginx:mynginx

docker run --rm -p84:80 94woodyfine/nginx:mynginx

http://192.168.0.244:84/
```

## 综合

```shell
FROM 94woodyfine/nginx:v1.12.2
USER root
ENV WWW /usr/share/nginx/html
ENV CONF /etc/nginx/conf.d
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
		echo 'Asia/Shanghai' >/etc/timezone
WORKDIR $WWW
ADD index.html $WWW/index
ADD demo.od.com.conf $CONF/demo.od.com.conf
EXPOSE 80
CMD ["nginx","-g","daemon off;"]

wget www.baidu.com -O index.html

# demo.od.com.conf
server {
	listen 80;
	server_name demo.od.com;
	
	root /usr/share/nginx/html;
}

# hosts中配置
192.168.0.244 demo.od.com

docker run --rm -p80:80 94woodyfine/ngixn:baidu

http://demo.od.com/
```

# Docker网络模型

## NAT(默认)

```shell
docker run -it --rm myalpine /bin/sh
ip add
```

## None

```shell
# 容器不设置网络
docker run -it --rm --net=none myapline /bin/sh
ip add
```

## Host

```shell
# 和宿主机一样
docker run -it --rm  --net=host myalpine /bin/sh
ip add
```

## 联合网络

```shell
# 容器共享网络
docker run -it --rm --net=container:1b12se3 94woodyfine/nginx:curl /bin/bash
```


