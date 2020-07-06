* docker能干什么？
  * 简化配置、代码流水线管理、提高开发效率、隔离应用、整合服务器、调试能力、多租户、快速部署

![image-20200628202416657](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628202416657.png)

![image-20200628202432607](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628202432607.png)

![image-20200628202951871](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628202951871.png)

# 容器技术和Docker简介

![image-20200628203151450](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203151450.png)

![image-20200628203303712](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203303712.png)

![image-20200628203429048](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203429048.png)

![image-20200628203449998](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203449998.png)

![image-20200628203705610](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203705610.png)

![image-20200628203722582](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203722582.png)

![image-20200628203757868](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628203757868.png)



# Docker环境的各种搭建方法

* docker-machine version
* docker-machine create demo
* docker-machine ls
* docker-machine ssh demo
* docker version
* exit
* docker-machine start/stop demo
* 本地连接到docker-machine中的server
  * docker version
  * docker-machine env demo
  * eval $(docker-machine env demo)
  * docker version

# Docker的镜像和容器

![image-20200628214609495](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628214609495.png)

![image-20200628214643804](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628214643804.png)

![image-20200628214719155](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628214719155.png)

![image-20200628214756974](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628214756974.png)

![image-20200628215100863](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628215100863.png)

![image-20200628215305739](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628215305739.png)

![image-20200628215406368](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628215406368.png)

![image-20200628220430722](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628220430722.png)

```shell
# 删除所有容器
docker rm $(docker container ls -aq)
# 删除退出的容器
docker rm $(docker container ls -f "status=exited" -q)
# 基于容器创建image：commit
# 通过Dockerfile创建image：build
FROM centos
RUN yum install -y vim
# docker build -t testid/test-new .
# docker history containerId
```

```shell
vim /etc/docker/daemon.json
{
	"registry-mirrors": ["http://hub-mirror.c.163.com"]
}

# Dockerfile
# FROM
FROM scratch  # 制作base image
FROM centos   # 使用base image
FROM ubuntu:14.04
# LABEL
# RUN，每RUN一次会生成新的一层
```

![image-20200628222353501](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222353501.png)

![image-20200628222433348](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222433348.png)

![image-20200628222511400](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222511400.png)

![image-20200628222529161](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222529161.png)

![image-20200628222700957](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222700957.png)

![image-20200628222722576](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222722576.png)

![image-20200628222933534](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222933534.png)

![image-20200628222959390](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628222959390.png)

![image-20200628223114651](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628223114651.png)

![image-20200628223419656](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628223419656.png)

![image-20200628223453182](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628223453182.png)

* ==ENTRYPOINT==
  * 

![image-20200628223543023](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200628223543023.png)

```shell
docker login
# docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
# docker push dockerhubId/imageName:latest
docker tag hello-world 94woodyfine/hello-world
docker push 94woodyfine/hello-world
docker pull 94woodyfine/hello-world:latest

# dockerhub和github相关联，github中维护Dockerfile
```

```shell
# registry
# push到私有的registry
docker pull registry
vim /etc/docker/daemon.json
{
        "registry-mirrors" : ["http://hub-mirror.c.163.com"],
        "insecure-registries":["192.168.0.226:5000"]
}
vim /lib/systemd/system/docker.service
#[Service]
EnvironmentFile=-/etc/docker/daemon.json

systemctl restart docker

docker run --name registry -p 5000:5000 -v /data/registry:/var/lib/registry -d registry:2.6.2
```

```shell
docker registry API  
```

```python
#app.py
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
  return "hello docker"
if __name__ == '__main__':
  app.run()
# pip3 install flask -i https://pypi.tuna.tsinghua.edu.cn/simple
```

```python
# Dockerfile
FROM python:3.6.8
LABEL maintainer="woodyfine@gmail.com"
RUN pip3 install flask
# 注意app是目录，后面得加/
COPY app.py /app/
WORKDIR /app
EXPOSE 5000
CMD ["python3", "app.py"]
```

```shell
docker build -t 94woodyfine/flask-hello-world .
docker run -it imageId /bin/bash

docker run -d --name=demo 94woodyfine/flask-hello-world

docker exec -it containerId /bin/bash
docker exec -it containerId python

docker inspect containerId
docker logs containerId
```

```shell
docker run -it ubuntu
apt-get update && apt-get install -y stress
which stress 
stress --help

docker run --memory=200M --cpu-shars=10 .....
```

```shell
# Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y stress
ENTRYPOINT [ "/usr/bin/stress" ]
CMD []
# docker build -t 94woodyfine/stress .
docker run -it 94woodyfine/stress --vm 1 --verbose
# --vm 1 --verbose 可作为 Dockerfile中 CMD 中命令参数
```

# Docker的网络

* 单机
  * Bridge Network
  * Host Network
  * None Network
* 多机
  * Overlay Network

![image-20200630082039813](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630082039813.png)

![image-20200630083039600](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630083039600.png)

* Ping验证IP的可达性
* telnet验证服务的可用性
* Wireshark抓包工具

```shell
docker pull busybox
docker run -d --name test busybox /bin/sh -c "while true; do sleep 3600; done"
docker ps
docker exec -it containerId /bin/sh
```

* 同一台机器上的两个docker之间可以互相Ping

```shell
[root@woody docker]# ip netns add test1
[root@woody docker]# ip netns list
test1
```

![image-20200630201411590](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630201411590.png)

![image-20200630201459820](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630201459820.png)

![image-20200630201651328](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630201651328.png)

![image-20200630201754910](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630201754910.png)

![image-20200630201824585](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630201824585.png)

![image-20200630201855724](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630201855724.png)

![image-20200630202051445](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630202051445.png)

![image-20200630202141510](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630202141510.png)

![image-20200630202221391](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630202221391.png)

![image-20200630202329688](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630202329688.png)

## bridge

```shell
[root@woody docker]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
2d8186a210ad        bridge              bridge              local
dd9f042d1e5c        host                host                local
2f8b9fe7df8a        none                null                local
```

```shell
[root@woody docker]# docker run -d --name test1 busybox /bin/sh -c "while true; do sleep 3600; done"
52951f565ea0e60525f748001bc5482ee07c2afe7237c32ec8eff8ae4f8bbe67
[root@woody docker]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
52951f565ea0        busybox             "/bin/sh -c 'while t…"   20 seconds ago      Up 19 seconds                           test1
# network id
[root@woody docker]# docker network inspect 2d8186a210ad
        "Containers": {
            "52951f565ea0e60525f748001bc5482ee07c2afe7237c32ec8eff8ae4f8bbe67": {
            		# busybox container name
                "Name": "test1",
                "EndpointID": "f1a488b15c8daf1dc1454f0b1eb04dc1c19532ee5ab0a6c7e62a9b24e16ac367",
                "MacAddress": "02:42:ac:11:00:02",
                # busybox container ip
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
[root@woody docker]# ip a
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:1f:ed:f4:88 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:1fff:feed:f488/64 scope link
       valid_lft forever preferred_lft forever
11: veth48e0fb6@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 8a:0b:a0:f9:ce:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::880b:a0ff:fef9:ce57/64 scope link
       valid_lft forever preferred_lft forever
# 11: veth48e0fb6@if10 连接 docker0       
# 11: veth48e0fb6@if10 和  10: eth0@if11  是一对
# 所以container可以访问外网
[root@woody docker]# docker exec test1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever       
```

```shell
[root@woody docker]# brctl
-bash: brctl: 未找到命令
[root@woody docker]# yum install bridge-utils
[root@woody docker]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.02421fedf488	no		veth48e0fb6           # 和test1容器一对的，连接到docker0上，所以test1容器通过它连接到docker0上，所以容器能够访问外网
```

```shell
[root@woody /]# docker run -d --name test1 busybox /bin/sh -c "while true; do sleep 3600; done"
30df5601ff618d28a7d591593d912895266cc73c0a94117e6370fd76418b1e38
[root@woody /]# docker run -d --name test2 busybox /bin/sh -c "while true; do sleep 3600; done"
4193a8f317782f1435bf72410240b4d2f041b2b4e021e4a01df916b28d07bb3d
[root@woody /]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
4193a8f31778        busybox             "/bin/sh -c 'while t…"   9 seconds ago       Up 8 seconds                            test2
30df5601ff61        busybox             "/bin/sh -c 'while t…"   18 seconds ago      Up 17 seconds                           test1
[root@woody /]# docker network inspect bridge
"Containers": {
            "30df5601ff618d28a7d591593d912895266cc73c0a94117e6370fd76418b1e38": {
                "Name": "test1",
                "EndpointID": "c3107473cb73c4f62c183f32eda321708dd7dad506fd63be9a32d429651459f0",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "4193a8f317782f1435bf72410240b4d2f041b2b4e021e4a01df916b28d07bb3d": {
                "Name": "test2",
                "EndpointID": "d6cf358a90ed61a052ad5103076b305f4b03339f814dad7d6287451765018afe",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
[root@woody /]# docker exec test1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@woody /]# docker exec test2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever       
[root@woody /]# ip a      
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:07:17:f1:49 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:7ff:fe17:f149/64 scope link
       valid_lft forever preferred_lft forever
# 和test1容器是一对的，连接在docker0上       
5: veth555585c@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 7e:0c:e7:92:45:2a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::7c0c:e7ff:fe92:452a/64 scope link
       valid_lft forever preferred_lft forever
# 和test2容器是一对，连接在docker0上       
7: veth531a302@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether aa:5a:dc:aa:79:1b brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::a85a:dcff:feaa:791b/64 scope link
       valid_lft forever preferred_lft forever       
[root@woody /]# brctl show
bridge name			bridge id						STP enabled				interfaces
docker0					8000.02420717f149		no								veth531a302      # test2
																											veth555585c      # test1
```

![image-20200630203541271](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630203541271.png)

## 容器之间的link

![image-20200630203951864](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630203951864.png)

![image-20200630204039167](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204039167.png)

* 只能test2 ping test1

![image-20200630204237281](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204237281.png)

![image-20200630204335379](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204335379.png)

![image-20200630204445964](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204445964.png)

* 之前的test2容器连接到另一个network（my-bridge），test2连接了两个network

* 在同一个bridge上，可以互相ping通

  ![image-20200630204649883](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204649883.png)

  ![image-20200630204728400](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204728400.png)

  ![image-20200630204827582](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630204827582.png)

  ![image-20200630205039593](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205039593.png)

  ![image-20200630205221621](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205221621.png)

  ![image-20200630205331166](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205331166.png)

## host

![image-20200630205541762](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205541762.png)

* 查看容器ip地址（默认连接到bridge）

  ![image-20200630205719516](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205719516.png)

  ![image-20200630205653664](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205653664.png)

  ![image-20200630205844167](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205844167.png)

  ![image-20200630205917663](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205917663.png)

  ![image-20200630205936198](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630205936198.png)

  ![image-20200630210044112](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630210044112.png)

  ![image-20200630210309172](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630210309172.png)

  * 阿里云验证

    ![image-20200630210455517](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630210455517.png)

    * 连接到阿里云的docker server中

    ![image-20200630210519346](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630210519346.png)

    ![image-20200630210714694](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630210714694.png)

## none

* 除了docker exec -it test1 /bin/sh 的方式，没有其他方式可以访问，是个孤立的namespace
* 只能本地访问

![image-20200630211011339](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630211011339.png)

![image-20200630211044543](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630211044543.png)

## host

* 和主机共享

![image-20200630211400774](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630211400774.png)

![image-20200630211453533](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630211453533.png)

![image-20200630211521080](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630211521080.png)

![image-20200630211616345](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630211616345.png)

## 多容器复杂应用的部署

```python
# app.py
from flask import Flask
from redis import Redis
import os
import socket

app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)


@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

```dockerfile
# Dockerfile
FROM python:3.6.8
LABEL maintaner="woodyfine@gmail.com"
COPY . /app
WORKDIR /app
RUN pip3 install flask redis
EXPOSE 5000
CMD [ "python3", "app.py" ]
```

![image-20200630230044304](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630230044304.png)

![image-20200630225211201](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225211201.png)

![image-20200630225352244](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225352244.png)

![image-20200630225425236](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225425236.png)

![image-20200630225533200](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225533200.png)

![image-20200630225652487](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225652487.png)

![image-20200630225719176](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225719176.png)

![image-20200630225818220](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225818220.png)

![image-20200630225943249](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630225943249.png)

![image-20200630230727542](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630230727542.png)

* # Mutil-host networking with etcd

  ## setup etcd cluster

  ```shell
nohup ./etcd --name docker-node1 --initial-advertise-peer-urls http://192.168.0.226:2380 \
  --listen-peer-urls http://192.168.0.226:2380 \
  --listen-client-urls http://192.168.0.226:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.0.226:2379 \
  --initial-cluster-token etcd-cluster \
  --initial-cluster docker-node1=http://192.168.0.226:2380,docker-node2=http://192.168.0.227:2380 \
  --initial-cluster-state new&
  
  nohup ./etcd --name docker-node2 --initial-advertise-peer-urls http://192.168.0.227:2380 \
  --listen-peer-urls http://192.168.0.227:2380 \
  --listen-client-urls http://192.168.0.227:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.0.227:2379 \
  --initial-cluster-token etcd-cluster \
  --initial-cluster docker-node1=http://192.168.0.226:2380,docker-node2=http://192.168.0.227:2380 \
  --initial-cluster-state new&
  
  ```
  
  
  
  在docker-node1上
  
  ```
  ubuntu@docker-node1:~$ wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
  ubuntu@docker-node1:~$ tar zxvf etcd-v3.0.12-linux-amd64.tar.gz
  ubuntu@docker-node1:~$ cd etcd-v3.0.12-linux-amd64
  ubuntu@docker-node1:~$ nohup ./etcd --name docker-node1 --initial-advertise-peer-urls http://192.168.205.10:2380 \
  --listen-peer-urls http://192.168.205.10:2380 \
  --listen-client-urls http://192.168.205.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.205.10:2379 \
  --initial-cluster-token etcd-cluster \
  --initial-cluster docker-node1=http://192.168.205.10:2380,docker-node2=http://192.168.205.11:2380 \
  --initial-cluster-state new&
  ```


  在docker-node2上

  ```
  ubuntu@docker-node2:~$ wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
  ubuntu@docker-node2:~$ tar zxvf etcd-v3.0.12-linux-amd64.tar.gz
  ubuntu@docker-node2:~$ cd etcd-v3.0.12-linux-amd64/
  ubuntu@docker-node2:~$ nohup ./etcd --name docker-node2 --initial-advertise-peer-urls http://192.168.205.11:2380 \
  --listen-peer-urls http://192.168.205.11:2380 \
  --listen-client-urls http://192.168.205.11:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.205.11:2379 \
  --initial-cluster-token etcd-cluster \
  --initial-cluster docker-node1=http://192.168.205.10:2380,docker-node2=http://192.168.205.11:2380 \
  --initial-cluster-state new&
  ```

  检查cluster状态

  ```
  ubuntu@docker-node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl cluster-health
  member 21eca106efe4caee is healthy: got healthy result from http://192.168.205.10:2379
  member 8614974c83d1cc6d is healthy: got healthy result from http://192.168.205.11:2379
  cluster is healthy
  ```

  ## 重启docker服务


  在docker-node1上

  ```
  $ sudo service docker stop
  $ sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://192.168.205.10:2379 --cluster-advertise=192.168.205.10:2375&
  ```

  在docker-node2上

  ```
  $ sudo service docker stop
  $ sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://192.168.205.11:2379 --cluster-advertise=192.168.205.11:2375&
  ```

  ## 创建overlay network

  在docker-node1上创建一个demo的overlay network

  ```
  ubuntu@docker-node1:~$ sudo docker network ls
  NETWORK ID          NAME                DRIVER              SCOPE
  0e7bef3f143a        bridge              bridge              local
  a5c7daf62325        host                host                local
  3198cae88ab4        none                null                local
  ubuntu@docker-node1:~$ sudo docker network create -d overlay demo
  3d430f3338a2c3496e9edeccc880f0a7affa06522b4249497ef6c4cd6571eaa9
  ubuntu@docker-node1:~$ sudo docker network ls
  NETWORK ID          NAME                DRIVER              SCOPE
  0e7bef3f143a        bridge              bridge              local
  3d430f3338a2        demo                overlay             global
  a5c7daf62325        host                host                local
  3198cae88ab4        none                null                local
  ubuntu@docker-node1:~$ sudo docker network inspect demo
  [
      {
          "Name": "demo",
          "Id": "3d430f3338a2c3496e9edeccc880f0a7affa06522b4249497ef6c4cd6571eaa9",
          "Scope": "global",
          "Driver": "overlay",
          "EnableIPv6": false,
          "IPAM": {
              "Driver": "default",
              "Options": {},
              "Config": [
                  {
                      "Subnet": "10.0.0.0/24",
                      "Gateway": "10.0.0.1/24"
                  }
              ]
          },
          "Internal": false,
          "Containers": {},
          "Options": {},
          "Labels": {}
      }
  ]
  ```

  我们会看到在node2上，这个demo的overlay network会被同步创建

  ```
  ubuntu@docker-node2:~$ sudo docker network ls
  NETWORK ID          NAME                DRIVER              SCOPE
  c9947d4c3669        bridge              bridge              local
  3d430f3338a2        demo                overlay             global
  fa5168034de1        host                host                local
  c2ca34abec2a        none                null                local
  ```

  通过查看etcd的key-value, 我们获取到，这个demo的network是通过etcd从node1同步到node2的

  ```
  ubuntu@docker-node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl ls /docker
  /docker/network
  /docker/nodes
  ubuntu@docker-node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl ls /docker/nodes
  /docker/nodes/192.168.205.11:2375
  /docker/nodes/192.168.205.10:2375
  ubuntu@docker-node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl ls /docker/network
  ubuntu@docker-node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl ls /docker/network/v1.0/network
  /docker/network/v1.0/network/3d430f3338a2c3496e9edeccc880f0a7affa06522b4249497ef6c4cd6571eaa9
  ubuntu@docker-node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl get /docker/network/v1.0/network/3d430f3338a2c3496e9edeccc880f0a7affa06522b4249497ef6c4cd6571eaa9 | jq .
  {
    "addrSpace": "GlobalDefault",
    "enableIPv6": false,
    "generic": {
      "com.docker.network.enable_ipv6": false,
      "com.docker.network.generic": {}
    },
    "id": "3d430f3338a2c3496e9edeccc880f0a7affa06522b4249497ef6c4cd6571eaa9",
    "inDelete": false,
    "ingress": false,
    "internal": false,
    "ipamOptions": {},
    "ipamType": "default",
    "ipamV4Config": "[{\"PreferredPool\":\"\",\"SubPool\":\"\",\"Gateway\":\"\",\"AuxAddresses\":null}]",
    "ipamV4Info": "[{\"IPAMData\":\"{\\\"AddressSpace\\\":\\\"GlobalDefault\\\",\\\"Gateway\\\":\\\"10.0.0.1/24\\\",\\\"Pool\\\":\\\"10.0.0.0/24\\\"}\",\"PoolID\":\"GlobalDefault/10.0.0.0/24\"}]",
    "labels": {},
    "name": "demo",
    "networkType": "overlay",
    "persist": true,
    "postIPv6": false,
    "scope": "global"
  }
  ```

  ![image-20200630232012529](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232012529.png)

  ![image-20200630232119969](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232119969.png)

  * 分别在node1和node2上exec进test1和test2容器内查看ip，10.0.0.2，10.0.0.3

  * 然后查看demo

    ![image-20200630232359546](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232359546.png)

    ![image-20200630232327932](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232327932.png)

    ![image-20200630232548226](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232548226.png)

    ![image-20200630232639113](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232639113.png)

    ![image-20200630232834805](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630232834805.png)

# Docker的持久化存储和数据共享

# Docker Compose多容器部署

# 容器编排Docker Swarm

# DevOps初体验-Docker Cloud 和Docker企业版

# 容器编排Kubernetes

# 容器的运维和监控

# Docker + DevOps实战-过程和工具

# 总结

