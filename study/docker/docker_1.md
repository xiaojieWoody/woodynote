# 序幕揭开

## Docker是什么？

* **容器就是将软件打包成标准化单元，以用于开发、交付和部署**

* **Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案
* Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。
* 总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样
* 由于Linux容器是进程级别的，相比虚拟机有很多优势
  * 启动快
    * 容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多
  * 资源占用少
    * 容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源
  * 体积小
    * 容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多
  * 总之
    * 容器有点像轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销小得多

![image-20191118094025875](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191118094025875.png)

## 搭建虚拟机

* 安装vagrant + virtualBox

* 选择一个目录，vagrant init，修改Vagrantfile文件，设置成public network、内存、cpu配置、vm类型centos/7

  ```shell
  config.vm.box = "centos/7"
  config.vm.network "public_network"
  config.vm.provider "virtualbox" do |vb|
          vb.memory = "2048"
          vb.name= "woody-centos7"
          vb.cpus= 2
  ```
  
* `vagrant box add centos/7 /Users/dingyuanjie/vargant/sys/virtualbox.box`

  * vagrant box list  查看本地的box

* Vagrantfile文件所在目录：vagrant up

* vagrant常用命令

  ```shell
  vagrant halt   		优雅关闭虚拟机
  vagrant up     		正常启动虚拟机
  vagrant ssh 	 		进入创建的虚拟机中
  vagrant status 		查看虚拟机状态
  vagrant destroy		删除虚拟机
  vagrant reload 		使修改的Vagrantfile生效
  ```

* box的打包分发

  ```shell
  # 01 退出虚拟机
  vagrant halt
  # 02 打包
  vagrant package --output first-docker-centos7.box
  # 03 将first-docker-centos7.box添加到其他的vagrant环境中
  vagrant box add first-docker-centos7 first-docker-centos7.box
  # 04 得到Vagrantfile
  vagrant init first-docker-centos7
  # 05 根据Vagrantfile启动虚拟机[此时可以得到和之前一模一样的环境，但是网络要重新配置]
  vagrant up 
  ```

* 虚拟机配置

  ```shell
  # 使用centos7的默认账号连接
  在centos文件夹下执行vagrant ssh-config
  关注:Hostname  Port  IdentityFile
  IP:127.0.0.1
  port:2222
  用户名:vagrant
  密码:vagrant
  文件:Identityfile指向的文件private-key
  # 使用root账户登录
  vagrant ssh   进入到虚拟机中
  sudo -i
  vi /etc/ssh/sshd_config
  修改PasswordAuthentication yes
  passwd修改密码，比如abc123
  systemctl restart sshd
  使用账号root，密码abc123进行登录
  ```

* [虚拟机中安装docker](https://docs.docker.com/install/linux/docker-ce/centos/)

## Docker常用命令

```shell
# 案例
# 查看容器
docker ps
# 创建mysql容器
	docker run -d --name my-mysql -p 3301:3306 -e MYSQL_ROOT_PASSWORD=jack123 --privileged mysql
# 进入到某个容器中并交互式运行
docker exec -it containerid /bin/bash
# 构建镜像
docker build -t my-mysql-image .
# 查看docker网络
docker network ls
docker inspect bridge
# 删掉所有container
docker rm -f $(docker ps -aq)


-name			设置容器名称
-d				后台运行
-p				宿主端口:容器端口
-e				设置参数
-i				交互式运行
```

# 灵魂探讨

* 简单来说，image就是由一层一层的layer组成的 

## Dockerfile

### 基本语法

* `FROM`

  > 指定基础镜像，比如FROM ubuntu:14.04

* `RUN`

  > 在镜像内部执行一些命令，比如安装软件，配置环境等，换行可以使用
  >
  > RUN groupadd -r mysql && useradd -r -g mysql mysql

* `ENV`

  > 设置变量的值
  >
  > ENV MYSQL_MA JOR 5.7，可以通过`docker run --e key=value`修改，后面可以直接使用${MYSQL_MA JOR}

* `LABEL`

  > 设置镜像标签
  >
  > `LABEL email="itcrazy2016@163.com"`
  > `LABEL name="itcrazy2016"`

* `VOLUME `

  > 指定数据的挂在目录
  > `VOLUME /var/lib/mysql`

* `COPY`

  > 将主机的文件复制到镜像内，如果目录不存在，会自动创建所需要的目录，注意只是复制，不会提取和解压
  > `COPY docker-entrypoint.sh /usr/local/bin/`

* `ADD`

  > 将主机的文件复制到镜像内，和COPY类似，只是ADD会对压缩文件提取和解压
  > `ADD application.yml /etc/itcrazy2016/`

* `WORKDIR`

  > 指定镜像的工作目录，之后的命令都是基于此目录工作，若不存在则创建
  > 会在`/usr/local/tomcat`下创建`test.txt`文件:
  > `WORKDIR /usr/local`
  > `WORKDIR tomcat`
  > `RUN touch test.txt`   
  > 会在`/root/test`下多出一个`app.yml`文件:
  > `WORKDIR /root`
  > `ADD app.yml test/`

* `CMD`

  > 容器启动的时候默认会执行的命令，若有多个CMD命令，则最后一个生效
  > `CMD ["mysqld"]` 或 `CMD mysqld`

* ENTRYPOINT

  > 和`CMD`的使用类似
  > `ENTRYPOINT ["docker-entrypoint.sh"]`
  > 和`CMD`的不同
  > `docker run`执行时，会覆盖`CMD`的命令，而`ENTRYPOINT`不会

* `EXPOSE`

  > 指定镜像要暴露的端口，启动镜像时，可以使用`-p`将该端口映射给宿主机 
  >
  > `EXPOSE 3306 `

### Dockerfile实战SpringBoot项目

1. docker环境新建一个目录，然后上传SprinBoot的jar包到该目录并创建Dockerfile

2. Dockerfile

   ```shell
   FROM openjdk:8
   MAINTAINER woodyfine
   LABEL name="dockerfile-demo" version="1.0" author="woody"
   COPY dockerfile-demo-0.0.1-SNAPSHOT.jar dockerfile-image.jar
   CMD ["java","-jar","dockerfile-image.jar"]
   ```

3. 基于Dockerfile构建镜像

   > `docker build -t test-docker-image .`

4. 基于image创建container

   > `docker run -d --name user01 -p 6666:8080 test-docker-image`

5. 查看启动日志

   > `docker logs user01`

6. 宿主主机上访问

   > `curl localhost:6666/dockerfile`

7. 还可以再次启动一个

   > `docker run -d --name user02 -p 8081:8080 test-docker-image`

## 镜像仓库

### docker hub

>1. `docker login`
>
>2. `docker push itcrazy2018/test-docker-image `
>
>   [注意镜像名称要和docker id一致，不然push不成功] 
>
>3. 给image重命名，并删除原来的
>
>   `docker tag test-docker-image itcrazy2018/test-docker-image `
>
>   `docker rmi -f test-docker-image `
>
>4. 再次推送，刷新hub.docker.com后台，发现成功 
>
>5. 别人下载，并且运行
>
>   `docker pull itcrazy2018/test-docker-image`
>
>   `docker run -d --name user01 -p 6661:8080 itcrazy2018/test-docker-image `

### Harbor

> [下载](https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.1.tgz)
>
> [安装docker compose](https://docs.docker.com/compose/install/)
>
> 1. 解压`tar -zxvf xxx.tar.gz `
> 2. 进入harbor目录，修改`harbor.cfg`文件，ip地址修改成当前机器的ip地址，密码默认Harbor12345
> 3. 安装`harbor`，`sh install.sh`
> 4. 浏览器访问，比如192.168.0.109，输入用户名和密码即可
>
> ```shell
> # 安装后，新建个用户，不要用admin了
> # Harbor12345Harbor
> # 开启harbor [进入到harbor安装目录]
> sudo docker-compose start
> # 关闭harbor
> sudo docker-compose stop
> ```
>
> ```shell
> # 控制台登录harbor异常  docker login 192.168.0.105
> /etc/docker/daemon.json 中加上 "insecure-registries": ["192.168.0.105"]
> 重新加载配置文件
> ```
>
> * 推拉harbor
>
>   1. 登录，`docker login 192.168.0.105`
>
>   2. 先在页面上建立项目，如`harbor`
>
>   3. 给镜像打标签，`docker tag test-docker-image 192.168.0.105/harbor/demo`
>
>   4. 推送到harbor上，`docker push 192.168.0.105/harbor/demo`
>   5. 从harbor上拉取，`docker pull 192.168.0.105/harbor/demo`

## Image

### 常见操作

> 1. 查看本地image列表
>
>    `docker images`、`docker image ls`
>
> 2. 获取远端镜像
>
>    `docker pull`
>
> 3. 删除镜像[注意此镜像如果正在使用，或者有关联的镜像，则需要先处理完] 
>
>    `docker image rm imageid`、`docker rmi -f imageid`
>
>    `docker rmi -f $(docker image ls)`  删除所有镜像
>
> 4. 运行镜像
>
>    `docker run image`
>
> 5. 发布镜像
>
>    `docker push`

## Container

* 可以理解为`container`只是基于`image`之后的`layer`而已，也就是可以通过`docker run image` 创建出一个`container`出来 
* 可以由一个`container`反推出`image` 
* 可以通过`docker commit`命令基于一个`container`重新生成一个image，但是一般得到image的方式不建议这么做，不然image怎么来的就全然不知

### 常见操作

```shell
# 根据镜像创建容器
docker run -d --name my-tomcat -p 9090:8080 tomcat
# 查看运行中的container
docker ps
# 查看所有的container[包含推出的]
docker ps -a
# 删除container
docker rm containerid
docker rm -f $(docker ps -a) # 删除所有container
# 进入到一个container中
docker exec -it container bash
# 根据container生成image
# 查看某个container的日志
docker logs container
# 查看容器资源使用情况
docker stats
# 查看容器详情
docker inspect container
# 停止/启动容器
docker stop/start container
```

### container资源限制

> 查看资源情况：`docker stats`

* 内存限制

  > -- memory
  >
  > `docker run -d --memory 100M --name tomcat1 tomcat`

* CPU限制

  > --cpu-shares 权重
  >
  > `docker run -d --cpu-shares 10 --name tomcat2 tomcat`

* 图像化资源监控

  ```shell
  https://github.com/weaveworks/scope
  
  sudo curl -L git.io/scope -o /usr/local/bin/scope
  sudo chmod a+x /usr/local/bin/scope
  scope launch 39.100.39.63
  
  # 停止scope 
  scope stop
  # 同时监控两台机器，在两台机器中分别执行如下命令
  scope launch ip1 ip2
  ```

### 底层技术支持

> 1. Container是一种轻量级的虚拟化技术，不用模拟硬件创建虚拟机 
>
> 2. Docker是基于Linux Kernel的Namespace、CGroups、UnionFileSystem等技术封装成的一种自定义容器格式，从而提供一套虚拟运行环境 
>
> * Namespace:用来做隔离的，比如pid[进程]、net[网络]、mnt[挂载点]等 
> * CGroups: Controller Groups用来做资源限制，比如内存和CPU等
> * Union file systems:用来做image和 container分层

# Docker网络

## Linux中网卡

* 查看网卡[网络接口]

  > `ip link show`
  >
  > `ls /sys/class/net`
  >
  > `ip a`

* 网卡

  > 1. `ip a`解读
  >
  >    * 状态:UP/DOWN/UNKOWN等 
  >
  >    * link/ether:MAC地址 
  >
  >    * inet:绑定的IP地址 
  >
  > 2. 配置文件
  >
  >    * 在Linux中网卡对应的其实就是文件，所以找到对应的网卡文件即可 
  >
  > 3. 给网卡添加IP地址
  >
  >    * 当然，这块可以直接修改ifcfg-*文件，但是通过命令添加试试 
  >      1. `ip addr add 192.168.0.100/24 dev eth0`
  >      2. 删除IP地址`ip addr delete 192.168.0.100/24 dev eth0 `
  >
  > 4. 网卡启动与关闭
  >
  >    * 重启网卡`service network restart / systemctl restart network `
  >    * 启动/关闭某个网卡`ifup/ifdown eth0 or ip link set eth0 up/down `

## Network Namespace

* 在linux上，网络的隔离是通过network namespace来管理的，不同的network namespace是互相隔离的 
* `ip netns list`:查看当前机器上的`network namespace`
* network namespace的管理 
  * `ip netns list` #查看 
  * `ip netns add ns1` #添加 
  * `ip netns delete ns1` #删除 

* namespace实战

  ```shell
  # 1.创建一个network namespace
  ip netns add ns1
  # 2.查看该namespace下网卡的情况
  ip netns exec ns1 ip a
  # 3.启动ns1上的lo网卡
  ip netns exec ns1 ifup lo
  or
  ip netns exec ns1 ip link set lo up
  # 4.再次查看，可以发现state变成了UNKOWN
  ip netns exec ns1 ip a
  # 5.再次创建一个network namespace
  ip netns add ns2
  # 6.此时想让两个namespace网络连通起来
  veth pair :Virtual Ethernet Pair，是一个成对的端口，可以实现上述功能
  # 7.创建一对link，也就是接下来要通过veth pair连接的link
  ip link add veth-ns1 type veth peer name veth-ns2
  # 8.查看link情况
  ip link
  # 9.将veth-ns1加入ns1中，将veth-ns2加入ns2中
  ip link set veth-ns1 netns ns1
  ip link set veth-ns2 netns ns2
  # 10. 查看宿主机和ns1,ns2的link情况
  ip link
  ip netns exec ns1 ip link
  ip netns exec ns2 ip link
  # 11. 此时veth-ns1和veth-ns2还没有ip地址，显然通信还缺少点条件
  ip netns exec ns1 ip addr add 192.168.0.11/24 dev veth-ns1
  ip netns exec ns2 ip addr add 192.168.0.12/24 dev veth-ns2
  # 12. 再次查看，发现state是DOWN，并且还是没有IP地址
  ip netns exec ns1 ip link
  ip netns exec ns2 ip link
  # 13. 启动veth-ns1和veth-ns2
  ip netns exec ns1 ip link set veth-ns1 up
  ip netns exec ns2 ip link set veth-ns2 up
  # 14. 再次查看，发现state是UP，同时有IP
  ip netns exec ns1 ip a
  ip netns exec ns2 ip a
  # 15. 此时两个network namespace互相ping一下，发现是可以ping通的
  ip netns exec ns1 ping 192.168.0.12
  ip netns exec ns2 ping 192.168.0.11
  ```

* Container的NS

  * 按照上面的描述，实际上每个container，都会有自己的network namespace，并且是独立的，可以进入到容器中进行验证 

  ```shell
  #1. 不妨创建两个container看看
  docker run -d --name tomcat01 -p 8081:8080 tomcat
  docker run -d --name tomcat02 -p 8082:8080 tomcat
  #2. 进入到两个容器中，并且查看ip
  docker exec -it tomcat01 ip a
  docker exec -it tomcat02 ip a
  #3. 互相ping一下是可以ping通的
   值得思考的是，此时tomcat01和tomcat02属于两个network namespace，是如何能够ping通的? 有些小伙伴可能会想，不就跟上面的namespace实战一样吗?注意这里并没有veth-pair技术
  ```

## 深入分析container网络-Bridge

### docker默认bridge

1. 查看centos的网络:`ip a`

2. 查看容器tomcat01的网络:`docker exec -it tomcat01 ip a`

3. 在`centos`中ping一下tomcat01的网络，可以发现ping通，既然可以ping通，而且centos和tomcat1又属于不同的network namespace，是怎么连接的 

   ![image-20191119152529766](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191119152529766.png)

4. 在tomcat01中有一个eth0和centos的docker0中有一个veth3是成对的，类似于之前实战中的veth-ns1和veth-ns2，不妨再通过一个命令确认下:`brctl` 

   ```shell
   # 安装一下:
   yum install bridge-utils 
   brctl show
   ```

5. 为什么tomcat01和tomcat02能ping通呢? 

   ![image-20191119152742561](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191119152742561.png)

6. 这种网络连接方法称之为Bridge，其实也可以通过命令查看docker中的网络模式:`docker network ls` ，bridge也是docker中默认的网络模式

7. 检查一下bridge:`docker network inspect bridge` 

8. 在tomcat01容器中是可以访问互联网的，NAT是通过iptables实现的 

   ![image-20191119153818996](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191119153818996.png)



### 创建自己的network

1.  创建一个network，类型为bridge 

   ```shell
   docker network create tomcat-net
   or
   docker network create --subnet=172.18.0.0/24 tomcat-net
   ```

2. 查看已有的network:`docker network ls `

   ![image-20191119154939400](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191119154939400.png)

3.  查看`tomcat-net`详情信息:`docker network inspect tomcat-net `

4. 创建`tomcat`的容器，并且指定使用`tomcat-net `

   `docker run -d --name custom-net-tomcat --network tomcat-net tomcat`

   （link生产上很少使用，更加提倡的是自定义自己的bridge类型的网络，然后名字进行访问）

5. 查看`custom-net-tomcat`的网络信息 

   `docker exec -it custom-net-tomcat ip a `

6. 查看网卡信息 

   `ip a`

7. 查看网卡接口 `brctl show `

8. 此时在custom-net-tomcat容器中ping一下tomcat01的ip会如何?发现无法ping通 

   `docker exec -it custom-net-tomcat ping 172.17.0.2`

9. 此时如果tomcat01容器能够连接到tomcat-net上应该就可以了

   `docker network connect tomcat-net tomcat01 `

10. 查看tomcat-net网络，可以发现tomcat01这个容器也在其中 

11. 此时进入到tomcat01或者custom-net-tomcat中，不仅可以通过ip地址ping通，而且可以通过名字ping 到，这时候因为都连接到了用户自定义的tomcat-net bridge上 

    `docker exec -it tomcat01 bash `

    `root@f49fc396d8e0:/usr/local/tomcat# ping 172.18.0.2`

    `root@f49fc396d8e0:/usr/local/tomcat# ping custom-net-tomcat`

12. 但是ping tomcat02是不通的 

    `root@f49fc396d8e0:/usr/local/tomcat# ping 172.17.0.3`

    `root@f49fc396d8e0:/usr/local/tomcat# ping tomcat02`

## 深入分析Container网络-Host&None

### Host

1. 创建一个tomcat容器，并且指定网络为host

   `docker run -d --name my-tomcat-host --network host tomcat `

2. 查看ip地址 

   `docker exec -it my-tomcat-host ip a `

   可以发现和centos是一样的 

3. 检查host网络 

### None

1. 创建一个tomcat容器，并且指定网络为none

   `docker run -d --name my-tomcat-none --network none tomcat `

2. 查看ip地址 

   `docker exec -it my-tomcat-none ip a`

3. 检查none网络 

## 端口映射

### 端口映射

1. 创建一个tomcat容器，名称为port-tomcat 

   `docker run -d --name port-tomcat tomcat `

2. 通过ip:port方式访问该tomcat

   `docker exec -it port-tomcat bash`

   `curl localhost:8080`

3. 那如果要在centos7上访问呢? 

   `docker exec -it port-tomcat ip a ---->得到其ip地址，比如172.17.0.4 `

   `curl 172.17.0.4:8080 `

* 小结 :之所以能够访问成功，是因为centos上的docker0连接了port-tomcat的network namespace 

4. 那如果要在centos7通过curl localhost方式访问呢?显然就要将port-tomcat的8080端口映射到centos上 

   ```shell
   docker rm -f port-tomcat
   docker run -d --name port-tomcat -p 8090:8080 tomcat
   curl localhost:8090
   ```

### 折腾

1. centos7是运行在win10上的虚拟机，如果想要在win10上通过ip:port方式访问呢? 

   ```shell
   #此时需要centos和win网络在同一个网段，所以在Vagrantfile文件中
   #这种方式等同于桥接网络。也可以给该网络指定使用物理机哪一块网卡，比如 #config.vm.network"public_network",:bridge=>'en1: Wi-Fi (AirPort)' 
   config.vm.network"public_network"
   centos7: ip a --->192.168.8.118 
   win10:浏览器访问 192.168.8.118:9080
   ```

2. 如果也想把centos7上的8090映射到win10的某个端口呢?然后浏览器访问localhost:port 

   ```shell
   #此时需要将centos7上的端口和win10上的端口做映射 
   config.vm.network"forwarded_port",guest:8098,host:8090
   #记得vagrant reload生效一下 
   win10:浏览器访问 localhost:8098
   ```

   <img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191119162153482.png" alt="image-20191119162153482" style="zoom:50%;" />

* 多机之间的container通信[放到Docker Swarm中详细聊] 

  > 在同一台centos7机器上，发现无论怎么折腾，一定有办法让两个container通信 
  >
  > 那如果是在两台centos7机器上呢?画个图 

  ![image-20191119162250705](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191119162250705.png)

  > 1. 使得两边的eth0能够通信 
  > 2. 前提要确保spring-boot-project container和mysql container的IP地址不一样 
  > 3. 将spring-boot-project中的所有信息当成eth0要传输给另外一端的信息 
  > 4. 具体通过vxlan技术实现 www.evoila.de/2015/11/06/what-is-vxlan-and-how-it-works 
  > 5. 处在vxlan的底层:underlay 处在xxlan的上层:overlay 
  >
  > （多机网络通信的问题，底层一个实现技术问题是:overlay--->vxlan）

# Docker数据持久化

## volume

> 1. 创建mysql数据库的container 
>
> `docker run -d --name mysql01 -e MYSQL_ROOT_PASSWORD=jack123  mysql`
>
> 2. 查看volume 
>
> `docker volume ls`
>
> 3. 具体查看该volume
>
> `docker volume inspect`
>
> 4. volume别名`-v mysql01_volume:/var/lib/mysql`
>
> `docker run -d --name mysql01 -v mysql01_volume:/var/lib/mysql -e
> MYSQL_ROOT_PASSWORD=jack123  mysql`
>
> 5. 查看volume
>
> `docker volume ls`
>
> `docker volume inspect mysql01_volume `
>
> 6. 验证持久化保存数据
>
> * `docker exec -it mysql01 bash `
> * `mysql -uroot -pjack123 `
> * `create database db_test `
> * 退出mysql服务，退出mysql container 
> * `docker rm -f mysql01` 
> * `docker volume ls`发现volume还在
> * `新建一个mysql container，并且指定使用"mysql01_volume"`
>   * `docker run -d --name test-mysql -v mysql01_volume:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=jack123 mysql`
> * 进入容器，登录mysql服务，查看数据库 
>   * `docker exec -it test-mysql bash `
>   * `mysql -uroot -pjack123`
>   * `show database; `
>
> * 可以发现db_test仍然在

## Bind Mounting

> 1. 创建一个tomcat容器 
>
>    `docker run -d --name tomcat01 -p 9090:8080 -v /tmp/test:/usr/local/tomcat/webapps/test tomcat`
>
> 2. 查看两个目录 
>
>    `centos:cd /tmp/test`
>
>    `tomcat容器:cd /usr/local/tomcat/webapps/test `
>
> 3. 在centos的/tmp/test中新建1.html，并写一些内容 
>
>    `<p style="color:blue; font-size:20pt;">This is p!</p>`
>
> 4. 进入tomcat01的对应目录查看，发现也有一个1.html，并且也有内容 
>
>    在centos7上访问该路径:curl localhost:9090/test/1.html 
>
> 5. 在win浏览器中通过ip访问 
>
>    ![image-20191123054830546](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191123054830546.png)

# Docker实战

## MySQL高可用集群搭建

1. 拉取pxc镜像 

   `docker pull percona/percona-xtradb-cluster:5.7.21`

2. 复制pxc镜像(实则重命名)

   `docker tag percona/percona-xtradb-cluster:5.7.21 pxc `

3. 删除pxc原来的镜像

   `docker rmi percona/percona-xtradb-cluster:5.7.21 `

4. 创建一个单独的网段，给mysql数据库集群使用

   `docker network create --subnet=172.18.0.0/24 pxc-net `

   `docket network inspect pxc-net [查看详情]`

   `docker network rm pxc-net [删除] `

5. 创建和删除volume

   创建:`docker volume create --name v1`

   删除:`docker volume rm v1` 

   查看详情:`docker volume inspect v1` 

6. 创建单个PXC容器demo

   `[CLUSTER_NAME PXC集群名字] `

   `[XTRABACKUP_PASSWORD数据库同步需要用到的密码] `

   ```shell
   docker run -d -p 3301:3306
   -v v1:/var/lib/mysql
   -e MYSQL_ROOT_PASSWORD=jack123
   -e CLUSTER_NAME=PXC
   -e XTRABACKUP_PASSWORD=jack123
   --privileged --name=node1 --net=pxc-net --ip 172.18.0.2
   pxc
   ```

7. 搭建PXC[MySQL]集群 

   1. 准备3个数据卷 

   ```shell
   docker volume create --name v1
   docker volume create --name v2
   docker volume create --name v3
   ```

   2. 运行三个PXC容器

   ```shell
   docker run -d -p 3301:3306 -v v1:/var/lib/mysql -e
   MYSQL_ROOT_PASSWORD=jack123 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=jack123 -
   -privileged --name=node1 --net=pxc-net --ip 172.18.0.2 pxc
   
   [CLUSTER_JOIN将该数据库加入到某个节点上组成集群]
    docker run -d -p 3302:3306 -v v2:/var/lib/mysql -e
   MYSQL_ROOT_PASSWORD=jack123 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=jack123 -
   e CLUSTER_JOIN=node1 --privileged --name=node2 --net=pxc-net --ip 172.18.0.3 pxc
   
   docker run -d -p 3303:3306 -v v3:/var/lib/mysql -e
   MYSQL_ROOT_PASSWORD=jack123 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=jack123 -
   e CLUSTER_JOIN=node1 --privileged --name=node3 --net=pxc-net --ip 172.18.0.4 pxc
   ```

   3. MySQL工具连接测试 

      Jetbrains Datagrip 

## 数据库的负载均衡

1. 拉取haproxy镜像 

   `docker pull haproxy`

2. 创建haproxy配置文件，这里使用bind mounting的方式 

   `touch /tmp/haproxy/haproxy.cfg`

   `haproxy.cfg`

   ```shell
   global 
   	#工作目录，这边要和创建容器指定的目录对应 
   	chroot /usr/local/etc/haproxy 
   	#日志文件
   	log 127.0.0.1 local5 info 
   	#守护进程运行
   	daemon
   defaults
       log global
   		mode http
   		#日志格式
   		option httplog #日志中不记录负载均衡的心跳检测记录 
   		option dontlognull #连接超时(毫秒)
   		timeout connect 5000 #客户端超时(毫秒) 
   		timeout client 50000 #服务器超时(毫秒) 
   		timeout server 50000
   #监控界面
   listen admin_stats
   #监控界面的访问的IP和端口
   bind 0.0.0.0:8888
   #访问协议
   mode
   #URI相对地址
   stats uri
   #统计报告格式
   stats realm
   #登陆帐户信息
   stats auth admin:admin
   #数据库负载均衡
   listen proxy-mysql #访问的IP和端口，haproxy开发的端口为3306 
   #假如有人访问haproxy的3306端口，则将请求转发给下面的数据库实例 
   bind 0.0.0.0:3306
   #网络协议
   mode tcp
   #负载均衡算法(轮询算法)
   #轮询算法:roundrobin
   #权重算法:static-rr
   #最少连接算法:leastconn
   #请求源IP算法:source
   balance roundrobin
   #日志格式
   option tcplog #在MySQL中创建一个没有权限的haproxy用户，密码为空。
   #Haproxy使用这个账户对MySQL数据库心跳检测
   option mysql-check user haproxy
   server MySQL_1 172.18.0.2:3306 check weight 1 maxconn 2000 
   server MySQL_2 172.18.0.3:3306 check weight 1 maxconn 2000 
   server MySQL_3 172.18.0.4:3306 check weight 1 maxconn 2000 
   #使用keepalive检测死链
   option tcpka
   ```

3. 创建haproxy容器 

   ```shell
   docker run -it -d -p 8888:8888 -p 3306:3306 -v
   /tmp/haproxy:/usr/local/etc/haproxy --name haproxy01 --privileged --net=pxc-net
   haproxy
   ```

4. 根据haproxy.cfg文件启动haproxy 

   ```shell
   docker exec -it haproxy01 bash
   haproxy -f /usr/local/etc/haproxy/haproxy.cfg
   ```

5. 在MySQL数据库上创建用户，用于心跳检测 

   ```sql
   CREATE USER 'haproxy'@'%' IDENTIFIED BY ''; 
   [小技巧[如果创建失败，可以先输入一下命令]:
       drop user 'haproxy'@'%';
       flush privileges;
       CREATE USER 'haproxy'@'%' IDENTIFIED BY '';
   ]
   ```

6. win浏览器访问 

   ```shell
   http://centos_ip:8888/dbs_monitor 
   用户名密码都是:admin
   ```

7. win上的datagrip连接haproxy01 

   ```shell
   ip:centos_ip
   port:3306
   user:root
   password:jack123
   ```

8. 在haproxy连接上进行数据操作，然后查看数据库集群各个节点 

## Nginx + SpringBoot项目 + MySQL

### 网络

1. 网络

   `docker network create --subnet=172.18.0.0/24 pro-net `

2. 网络划分 

   ```shell
   mysql--172.18.0.6 
   spring boot--172.18.0.11/12/13 
   nginx--172.18.0.10
   ```

   ![image-20191123061144399](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191123061144399.png)

### MySQL

1. 创建volume 

   `docker volume create v1`

2. 创建mysql容器 

   ```shell
   docker run -d --name my-mysql -v v1:/var/lib/mysql -p 3301:3306 -e
   MYSQL_ROOT_PASSWORD=jack123 --net=pro-net --ip 172.18.0.6 mysql
   ```

3. datagrip连接，执行.mysql文件 

   ```shell
   name:my-mysql 
   ip:centos-ip 
   端口:3301 
   user:root 
   password:jack123
   ```

   ```sql
   create schema db_gupao_springboot collate utf8mb4_0900_ai_ci;
   use db_gupao_springboot;
   create table t_user
   (
       id int not null
           primary key,
       username varchar(50) not null,
       password varchar(50) not null,
       number varchar(100) not null
   );
   ```

### SpringBoot项目

* Spring Boot+MyBatis实现CRUD操作，名称为“springboot-mybatis” 

  1. 在本地测试该项目的功能 

     主要是修改application.yml文件中数据库的相关配置 

  2. 在项目根目录下执行mvn clean package打成一个jar包 

     ```shell
     [记得修改一下application.yml文件数据库配置]
     mvn clean package -Dmaven.test.skip=true 
     在target下找到"springboot-mybatis-0.0.1-SNAPSHOT.jar.jar"
     ```

  3. 在docker环境中新建一个目录"springboot-mybatis" 

  4. 上传"springboot-mybatis-0.0.1-SNAPSHOT.jar"到该目录下，并且在此目录创建Dockerfile 

  5. 编写Dockerfile内容

     ```shell
     FROM openjdk:8-jre-alpine
     MAINTAINER itcrazy2016
     LABEL name="springboot-mybatis" version="1.0" author="itcrazy2016" COPY springboot-mybatis-0.0.1-SNAPSHOT.jar springboot-mybatis.jar 
     CMD ["java","-jar","springboot-mybatis.jar"]
     ```

  6. 基于Dockerfile构建镜像

     `docker build -t sbm-image . `

  7. 基于image创建container

     ```shell
     docker run -d --name sb01 -p 8081:8080 --net=pro-net --ip 172.18.0.11 sbm-image
     ```

  8. 查看启动日志docker logs sb01 

  9. 在win浏览器访问`http://192.168.8.118:8081/user/listall`

* 网络问题
  * 因为sb01和my-mysql在同一个bridge的网段上，所以是可以互相ping通，比如 

  ```shell
  docker exec -it sb01 ping 172.18.0.6
  or
  docker exec -it sb01 ping my-mysql
  ```
  * so? application.yml文件不妨这样修改一下?也就是把ip地址直接换成容器的名字 

    `url: jdbc:mysql://my-mysql/db_gupao_springboot?`

* 创建多个项目容器

  ```shell
  docker run -d --name sb01 -p 8081:8080 --net=pro-net --ip 172.18.0.11 sbm-image
  docker run -d --name sb02 -p 8082:8080 --net=pro-net --ip 172.18.0.12 sbm-image
  docker run -d --name sb03 -p 8083:8080 --net=pro-net --ip 172.18.0.13 sbm-image
  ```

### Nginx

1. 在centos的/tmp/nginx下新建nginx.conf文件，并进行相应的配置 

   ```shell
   user nginx;
   worker_processes  1;
   events {
       worker_connections  1024;
   }
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
       sendfile        on;
       keepalive_timeout  65;
       server {
           listen 80;
           location / {
            	proxy_pass http://balance;
   				} 
       }
       upstream balance{
           server 172.18.0.11:8080;
           server 172.18.0.12:8080;
           server 172.18.0.13:8080;
   		}
       include /etc/nginx/conf.d/*.conf;
   }
   ```

2. 创建nginx容器

   * 先在centos7上创建/tmp/nginx目录，并且创建nginx.conf文件，写上内容 

     ```shell
     docker run -d --name my-nginx -p 80:80 -v
     /tmp/nginx/nginx.conf:/etc/nginx/nginx.conf --network=pro-net --ip 172.18.0.10
     nginx
     ```

3. win浏览器访问: ip[centos]/user/listall
4. 若将172.18.0.11/12/13改成sb01/02/03是否可以? 



