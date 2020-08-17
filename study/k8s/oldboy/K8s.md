* 使用Docker容器化封装应用程序的缺点
  * 单机使用，无法有效集群
  * 随着容器数量的上升，管理成本攀升
  * 没有有效的容灾/自愈机制
  * 没有预设编排模版，无法实现快速、大规模容器调度
  * 没有统一的配置管理中心
  * 没有容器生命周期的管理工具
  * 没有图形化运维管理工具

# k8s快速入门

## 1.15.0

## 优势

* 自动装箱，水平扩展，自我修复
* 服务发现和负载均衡
* 自动发布（默认滚动发布模式）和回滚
* 集中化配置管理和密钥管理
* 存储编排
* 任务批处理运行

## 四组基本概念

* Pod/Pod控制器
  * Pod
  * 是k8s里能够被运行的最小的逻辑单元（原子单元）
    * 1个Pod里面可以运行多个容器，它们共享UTS+NET+IPC名称空间
  * 一个Pod里运行多个容器，又叫：边车（SideCar模式）
  * Pod控制器
    * Pod控制器是Pod启动的一种模版，用来保证在k8s里启动的Pod应始终按照人们的预期运行（副本数、生命周期、健康状态检查...）
    * k8s内提供了众多的Pod控制器，常用的有：
      * **Deployment**
      * **DaemonSet**
      * ReplicaSet
      * StatefulSet
      * Job
      * Cronjob
* Name/Namespace
  * Name
    * 由于k8s内部，使用“资源”来定义每一种逻辑概念（功能），故每种“资源”，都应该有自己的名称
    * “资源”有api版本（apiVersion）类别（kind）、元数据（metadata）、定义清单（spec）、状态（status）等配置信息
    * “名称”通常定义在“资源”的“元数据”信息里
  * Namespace
    * 随着项目增多、人员增加、集群规模的扩大，需要一种能够隔离k8s内各种“资源”的方法，这就是名称空间
    * 名称空间可以理解为k8s内部的虚拟集群组
    * 不同名称空间内的“资源”，名称可以相同，相同名称空间内的同种“资源”，“名称”不能相同
    * 合理的使用k8s的名称空间，使得集群管理员能够更好的对交付到k8s里的服务进行分类管理和浏览
    * k8s里默认存在的名称空间有：default、kube-system、kube-public
    * 查询k8s里特定“资源”要带上相应的名称空间
* Label/Label选择器
  * Label
    * 标签是k8s特色的管理方式，便于分类管理资源对象
    * 一个标签可以对应多个资源，一个资源也可以有多个标签
    * 一个资源拥有多个标签，可以实现不同维度的管理
    * 标签的组成：key=value
    * 与标签类似的，还有一种“注解”（annotations）
  * Label选择器
    * 给资源打上标签后，可以使用标签选择器过滤指定的标签
    * 标签选择器目前有两个：基于等值关系（等于、不等于）和基于集合关系（属于、不属于、存在）
    * 许多资源支持内嵌标签选择器字段
      * matchLabels
      * matchExpressions
* service/Ingress
* Service
    * 在k8s的世界里，虽然每个Pod都会被分配一个单独的IP地址，但这个IP地址会随着Pod的销毁而消失
    * Service（服务）就是用来解决这个问题的核心概念
    * 一个Service可以看作一组提供相同服务的Pod的对外访问接口
    * Service作用于那些Pod是通过标签选择器来定义的
  * Ingress
    * Ingress是k8s集群里工作在OSI网络参考模型下，第7层的应用，对外暴露的接口
    * Service只能进行L4流量调度，表现形式是ip + port
    * Ingerss则可以调度不同业务域、不同URL访问路径的业务流量

## 核心组件

* 配置存储中心——etcd服务

* 主控（master）节点

  * kube-apiserver服务
    * 提供了集群管理的REST API接口（包括鉴权、数据校验及集群状态变更）
    * 负责其他模块之间的数据交互，承担通信枢纽功能
    * 是资源配额控制的入口
    * 提供完备的集群安全机制
  * kube-controller-manager服务
    * 由一系列控制器组成，通过apiserver监控整个集群的状态，并确保集群处于预期的工作状态
    * Node Controller
    * Deployment Controller
    * Service Controller
    * Volume Controller 
    * Endpoint Controller
    * Garbage Controller
    * Namespace Controller
    * Job Controller
    * Resource quta Controller
    * ...
  * kube-scheduler服务
    * 主要功能是接收调度pod到适合的运算节点上
    * 预算策略（predict）
    * 优选策略（priorities）

* 运算（node）节点

  * kube-kubelet服务

    * 简单的说，kubelet的主要功能就是定时从某个地方获取节点上pod的期望状态（运行什么容器、运行的副本数量、网络或者存储如何配置等等），并调用对应的容器平台接口达到这个状态
    * 定时汇报当前节点的状态给apiserver，以供调度的时候使用
    * 镜像和容器的清理工作，保证节点上镜像不会占满磁盘空间，退出的容器不会占用太多资源

  * kube-proxy服务

    * 是K8S在每个节点上运行网络代理，service资源的载体
    * 建立了pod网络和集群网络的关系（clusterip——>podip）
    * 常用三种流量调度模式
      * Userspace（废弃）
      * Iptables（濒临废弃）
      * Ipvs（推荐）
    * 负责建立和删除包括更新调度规则、通知apiserver自己的更新，或者从apiserver那里获取其他kube-proxy的调度规则变化来更新自己的

    ![image-20200809144459056](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809144459056.png)

    ![image-20200809144808938](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809144808938.png)

## CLI客户端

* kubectl

## 核心附件

* CNI网络插件——flannel/calico
* 服务发现插件——coredns
* 服务暴露用插件——traefik
* GUI管理插件——Dashboard

# 实验部署集群架构详解

![image-20200809145022191](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809145022191.png)

# 部署k8s集群前准备工作

* Minikube 单节点微型K8S（仅供学习、预览使用）
* **二进制安装部署（生产首选，新手推荐）**
  * 2G2核NAT
* 使用kubeadmin进行部署，K8S的部署工具，跑在K8S里（相对简单，熟手推荐）

## 环境准备（二进制）

* 准备5台2c/2g/50g虚拟机，使用10.4.7.0/24网络
* 预装CentOS7.6操作系统，做好基础优化
* 安装部署bind9，部署自建DNS系统
* 准备自签证书环境
* 安装部署Docker环境，部署Harbor私有仓库

## 虚拟机基础优化

![image-20200809151538759](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809151538759.png)

```shell
hostnamectl set-hostname HDSS7-11.host.com
hostnamectl set-hostname HDSS7-12.host.com
hostnamectl set-hostname HDSS7-21.host.com
hostnamectl set-hostname HDSS7-22.host.com
hostnamectl set-hostname HDSS7-200.host.com

vi /etc/selinux/config
将SELINUX=enforcing改为SELINUX=disabled
设置后需要重启才能生效
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
```

```shell
# 配置镜像源
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 安装工具
yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -y
```

```shell
# 5台虚拟机
10.4.7.11  
	# 安装bind9
10.4.7.12
10.4.7.21
10.4.7.22
10.4.7.200
```

## 自建DNS系统

* 只在`10.4.7.11`上

```shell
# hdss7-11上 (master)
yum install bind -y
rpm -qa bind

# 主配置文件
vi /etc/named.conf
	listen-on port 53 { 10.4.7.11; };
	allow-query  			{ any; };
	forwarders        { 10.4.7.254; };
	dnssec-enable no;
	dnssec-validation no;
	
# 自己实际配置	
# stics-file "/var/named/data/named_stats.txt";
# memstatistics-file "/var/named/data/named_mem_stats.txt";
# rcursing-file  "/var/named/data/named.recursing";
# secroots-file   "/var/named/data/named.secroots";
# allow-query     { any; };
# forwarders      { 192.168.0.255; };	

--------------------

# 检查配置	
named-checkconf	

# 区域配置文件
vim /etc/named.rfc1912.zones
## 追加到后面
zone "host.com" IN {
		type master;
		file "host.com.zone";
		allow-update { 10.4.7.11; };
};
	
zone "od.com" IN {
		type master;
		file "od.com.zone";
		allow-update { 10.4.7.11; };
};

## 自己实际配置
#zone "host.com" IN {
#        type master;
#        file "host.com.zone";
#        allow-update { 192.168.0.244; };
#};

#zone "od.com" IN {
#        type master;
#        file "od.com.zone";
#        allow-update { 192.168.0.244; };
#};

# 配置区域数据文件
## 配置主机域数据文件
vi /var/named/host.com.zone

$ORIGIN host.com.
$TTL 600	; 10 minutes
@				IN SOA dns.host.com. dnsadmin.host.com. (
								2020080901	; serial
								10800			; refresh (3 hours)
								900				; retry (15 minutes)
								604800		; expire (1 week)
								86400			; minimum (1 day)
								)
				NS dns.host.com.
$TTL 60 ; 1 minute
dns			A		10.4.7.11
HDSS7-11			A		10.4.7.11
HDSS7-12			A		10.4.7.12
HDSS7-21			A		10.4.7.21
HDSS7-22			A		10.4.7.22
HDSS7-200			A		10.4.7.200

----------------------------------------------------
# 自己实际配置
$ORIGIN host.com.
$TTL 600	; 10 minutes
@				IN SOA dns.host.com. dnsadmin.host.com. (
								2020080901	; serial
								10800		; refresh (3 hours)
								900		; retry (15 minutes)
								604800		; expire (1 week)
								86400		; minimum (1 day)
								)
				NS dns.host.com.
$TTL 60 ; 1 minute
dns			A		192.168.0.244
master			A		192.168.0.244
worker1			A		192.168.0.245
worker2			A		192.168.0.246
worker3			A		192.168.0.247
worker4			A		192.168.0.248
```

```shell
# 配置区域数据文件
## 配置业务域数据文件
vim /var/named/od.com.zone

$ORIGIN od.com.
$TTL 600	; 10 minutes
@					IN SOA dns.od.com. dnsadmin.od.com. (
												2020080902	; serial
												10800			  ; refresh (3 hours)
												900					; retry (15 minutes)
												604800			; expire (1 week)
												86400				; minimum (1 day)
												)
												NS dns.od.com.
$TTL 60 ; 1 minute
dns			A		10.4.7.11

------------------------------------------------
# 自己实际配置
$ORIGIN od.com.
$TTL 600    ; 10 minutes
@                    IN SOA dns.od.com. dnsadmin.od.com. (
                                                2020080902    ; serial
                                                10800         ; refresh (3 hours)
                                                900           ; retry (15 minutes)
                                                604800        ; expire (1 week)
                                                86400         ; minimum (1 day)
                                                )
                                                NS dns.od.com.
$TTL 60 ; 1 minute
dns            A        192.168.0.244
```

```shell
# 检查配置
named-checkconf

# 启动
systemctl start named
systemctl enable named

# 查看端口
netstat -luntp|grep 53

# 验证
dig -t A hdss7-21.host.com @10.4.7.11 +short
# 10.4.7.11
dig -t A hdss7-200.host.com @10.4.7.11 +short
# 10.4.7.200
```

![image-20200809163244812](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809163244812.png)

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	DNS1=10.4.7.11
cat /etc/resolv.conf	# vi
	# Generated by NetworkManager
	search host.com
	nameserver 10.4.7.11
	
systemctl restart network
ping www.baidu.com
ping hdss7-21.host.com
ping hdss7-200

# 其他机器也都改成，然后互相ping域名，如ping hdss7-21.host.com  ping www.baidu.com
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	DNS1=10.4.7.11
systemctl restart network

# mac上修改DNS，改为10.4.7.11，否则mac上访问不了（ping hdss7-21.host.com 失败）
```

![image-20200809170200139](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809170200139.png)

## 签发证书

* 只在10.4.7.200(worker4上)

```shell
# 安装CFSSL
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/bin/cfssl
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/bin/cfssl-json
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/bin/cfssl-certinfo
chmod +x /usr/bin/cfssl*

[root@worker4 bin]# cd /opt/
[root@worker4 opt]# mkdir certs
[root@worker4 opt]# cd certs/
[root@worker4 certs]# pwd
/opt/certs

# 创建生成CA证书签名请求(csr)
/opt/certs/ca-csr.json
{
	"CN": "OldboyEdu",
	"hosts": [
	],
	"key": {
		"algo": "rsa",
		"size": 2048
	},
	"names": [
		{
			"C": "CN",
			"ST": "beijing",
			"L": "beijing",
			"O": "od",
			"OU": "ops"
		}
	],
	"ca": {
		"expiry": "175200h"
	}
}

# 生成证书
[root@worker4 certs]# cfssl gencert -initca ca-csr.json | cfssl-json -bare ca
```

![image-20200809171815503](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809171815503.png)

## 部署docker环境

* 所有机器

```shell
mkdir -p /etc/docker /data/docker
vim /etc/docker/daemon.json

# 机器内容器IP设置
## 各机器的bip不同，分别对应
# 如 10.4.7.21  为 172.7.21.1/24
# 如 10.4.7.22  为 172.7.22.1/24
# 如 10.4.7.200  为 172.7.200.1/24
# 如 10.4.7.12  为 172.7.12.1/24
# 如 10.4.7.11  为 172.7.11.1/24

# 注意添加  harbor.od.com ，否则后面部署的harbor访问不了
{
	"graph":"/data/docker",
	"storage-driver":"overlay2",
	"insecure-registries":["registry.access.redhat.com","quay.io", "harbor.od.com"],
	"registry-mirrors":["https://q2gr04ke.mirror.aliyuncs.com"],
	"bip":"172.7.21.1/24",
	"exec-opts":["native.cgroupdriver=systemd"],
	"live-restore":true
}

systemctl restart docker
docker info
docker ps
```

## 部署docker镜像私有仓库Harbor

* 只在10.4.7.200上 （worker4上）

```shell
tar -zxvf /data/harbor-offline-installer-v2.0.2.tgz -C /opt/
[root@worker4 opt]# mv harbor/ harbor-v2.0.2
[root@worker4 opt]# ln -s /opt/harbor-v2.0.2/ /opt/harbor

cd harbor
vi harbor.yml
# 修改点
	hostname: harbor.od.com
	http:
		port: 180
	log:
		location: /data/harbor/logs
	data_volume: /data/harbor		

mkdir -p /data/harbor/logs /data/harbor

# 安装docker-compose
yum install docker-compose -y
# 验证
rpm -qa docker-compose

# 安装harbor
./install.sh

docker-compose ps 

# 安装nginx
yum install nginx -y
# 配置nginx
vim /etc/nginx/conf.d/harbor.od.com.conf
server {
	listen			80;
	server_name	harbor.od.com;                
	client_max_body_size 1000m;
	location / {
		proxy_pass http://127.0.0.1:180;
	}
}
# 检查nginx配置文件
nginx -t
# 启动nginx
systemctl start nginx
systemctl enable nginx

# hdss7-11 (master)上
vi /var/named/od.com.zone
	# 注意serial前滚一个序号
	harbor		A 		10.4.7.200	
	
systemctl restart named
dig -t A harbor.od.com +short

# 200（worker4）上
[root@worker4 harbor]# curl harbor.od.com

# 浏览器上访问
http://harbor.od.com/

# 200(worker4上)
docker pull nginx:1.7.9
docker tag ImageID harbor.od.com/public/nginx:v1.7.9
docker login harbor.od.com
docker push harbor.od.com/public/nginx:v1.7.9

# 注意 vim /etc/docker/daemon.json中配置 insecure-registries中添加 "harbor.od.com"
# 否则无法访问harbor.od.com
"insecure-registries":["registry.access.redhat.com","quay.io", "harbor.od.com"],

# 重新启动harbor
cd harbor
docker-compose up
```

# 部署主控节点服务

## 部署etcd集群

| 主机名            | 角色        | ip        |
| ----------------- | ----------- | --------- |
| HDSS7-12.host.com | etcd lead   | 10.4.7.12 |
| HDSS7-21.host.com | etcd follow | 10.4.7.21 |
| HDSS7-22.host.com | etcd follow | 10.4.7.22 |

```shell
# 200（worker4）上
cd /opt/certs
vi ca-config.json

{
	"signing": {
			"default": {
					"expiry": "175200h"
			},
			"profiles": {
					"server": {
							"expiry": "175200h",
							"usages": [
									"signing",
									"key encipherment",
									"server auth"
							]
					},
					"client": {
							"expiry": "175200h",
							"usages": [
									"signing",
									"key encipherment",
									"client auth"
							]
					},
					"peer": {
							"expiry": "175200h",
							"usages": [
									"signing",
									"key encipherment",
									"server auth",
									"client auth"
							]
					}
			}
	}
}
```

```shell
vi etcd-peer-csr.json
{
	"CN": "k8s-etcd",
	"hosts": [
			"10.4.7.11",
			"10.4.7.12",
			"10.4.7.21",
			"10.4.7.22"
	],
	"key": {
			"algo": "rsa",
			"size": 2048
	},
	"names": [
			{
					"C": "CN",
					"ST": "beijing",
					"L": "beijing",
					"O": "od",
					"OU": "ops"
			}
	]
}

# 生成etcd证书和私钥
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer etcd-peer-csr.json | cfssl-json -bare etcd-peer
```

```shell
# 10.4.7.12、10.4.7.21、10.4.7.22（worker1、worker2、worker3）上
# worker4密码改为了：Iam0108Woody

# 新增用户
useradd -s /sbin/nologin -M etcd
id etcd

# 安装etcd
[root@worker1 data]# tar -zxvf etcd-v3.1.20-linux-amd64.tar.gz -C /opt/
[root@worker2 data]# cd /opt/
[root@worker1 opt]# mv etcd-v3.1.20-linux-amd64/ etcd-v3.1.20
[root@worker1 opt]# ln -s /opt/etcd-v3.1.20/ /opt/etcd
[root@worker1 opt]# cd etcd
[root@worker1 etcd]# mkdir -p /opt/etcd/certs /data/etcd /data/logs/etcd-server
[root@worker1 etcd]# cd certs/
[root@worker1 certs]# scp worker4:/opt/certs/ca.pem .
[root@worker1 certs]# scp worker4:/opt/certs/etcd-peer.pem .
[root@worker1 certs]# scp worker4:/opt/certs/etcd-peer-key.pem .

# 创建etcd服务启动脚本   # 不同：--name etcd-server-7-12、--name etcd-server-7-21、--name etcd-server-7-22
vi /opt/etcd/etcd-server-startup.sh
#!/bin/sh
./etcd --name etcd-server-7-22 \
       --data-dir /data/etcd/etcd-server \
       --listen-peer-urls https://192.168.0.247:2380 \
       --listen-client-urls https://192.168.0.247:2379,http://127.0.0.1:2379 \
       --quota-backend-bytes 8000000000 \
       --initial-advertise-peer-urls https://192.168.0.247:2380 \
       --advertise-client-urls https://192.168.0.247:2379,http://127.0.0.1:2379 \
       --initial-cluster  etcd-server-7-12=https://192.168.0.245:2380,etcd-server-7-21=https://192.168.0.246:2380,etcd-server-7-22=https://192.168.0.247:2380 \
       --ca-file ./certs/ca.pem \
       --cert-file ./certs/etcd-peer.pem \
       --key-file ./certs/etcd-peer-key.pem \
       --client-cert-auth  \
       --trusted-ca-file ./certs/ca.pem \
       --peer-ca-file ./certs/ca.pem \
       --peer-cert-file ./certs/etcd-peer.pem \
       --peer-key-file ./certs/etcd-peer-key.pem \
       --peer-client-cert-auth \
       --peer-trusted-ca-file ./certs/ca.pem \
       --log-output stdout

chmod +x /opt/etcd/etcd-server-startup.sh

cd /opt/etcd
[root@worker1 etcd]# chown -R etcd.etcd /opt/etcd-v3.1.20/
[root@worker1 etcd]# chown -R etcd.etcd /data/etcd/
[root@worker1 etcd]# chown -R etcd.etcd /data/logs/etcd-server/

yum install supervisor -y

[root@worker1 etcd]# systemctl start supervisord
[root@worker1 etcd]# systemctl enable supervisord

vi /etc/supervisord.d/etcd-server.ini  # [program:etcd-server-7-12]、[program:etcd-server-7-21]、[program:etcd-server-7-22]
[program:etcd-server-7-12]
command=sh /opt/etcd/etcd-server-startup.sh                     ; the program (relative uses PATH, can take args)	
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/etcd                                             ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=etcd                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
stdout_logfile=/data/logs/etcd-server/etcd.stdout.log           ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)


----------------------------------------
# 自己实际配置
[program:etcd-server-7-22]
command=sh /opt/etcd/etcd-server-startup.sh        ; the program (relative uses PATH, can take args)
numprocs=1                                                                    ; number of processes copies to start(def 1)
directory=/opt/etcd                                                    ; directory to cwd to before exec(def no cwd)
autostart=true                                                            ; start at supervisord start(default:true)
autorestart=true                                                        ; restart at unexpected quit(default: true)
startsecs=30                                                                ; number of secs prog must stay running
startretries=3                                                            ; max # of serial start failures(default 3)
exitcodes=0,2                                                                ; 'expected' exit codes for process(default 0,2)
stopsignal=QUIT                                                            ; signal used to kill process(default TERM)
stopwaitsecs=10                                                        ;
user=etcd                                                                        ;
redirect_stderr=true                                                ;
stdout_logfile=/data/logs/etcd-server/etcd.stdout.log ;
stdout_logfile_maxbytes=64MB                                ;
stdout_logfile_backups=4                                        ;
stdout_capture_maxbytes=1MB                                    ;
stdout_events_enabled=false                                    ;

[root@worker1 etcd]# vi /etc/supervisord.d/etcd-server.ini
[root@worker1 etcd]# supervisorctl update
etcd-server-7-12: added process group
[root@worker1 etcd]# supervisorctl status
etcd-server-7-12                 STARTING
[root@worker1 etcd]# netstat -luntp|grep etcd
tcp        0      0 192.168.0.245:2379      0.0.0.0:*               LISTEN      2674/./etcd
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      2674/./etcd
tcp        0      0 192.168.0.245:2380      0.0.0.0:*               LISTEN      2674/./etcd

# 查看日志
[root@worker1 etcd]# tail -fn 200 /data/logs/etcd-server/etcd.stdout.log

# 任意节点检测etcd集群健康状态
[root@worker2 etcd]# ./etcdctl cluster-health
[root@worker2 etcd]# ./etcdctl member list
```

* 10.4.7.21、10.4.7.22 (woker2、worker3)

```shell
# 安装
[root@worker2 data]# tar -zxvf kubernetes-server-linux-amd64-v1.15.2.tar.gz -C /opt/
[root@worker2 data]# cd /opt/
[root@worker2 opt]# mv kubernetes/ kubernetes-v1.15.2
[root@worker2 opt]# ln -s /opt/kubernetes-v1.15.2/ /opt/kubernetes
[root@worker2 opt]# cd kubernetes
[root@worker2 kubernetes]# rm -rf kubernetes-src.tar.gz
[root@worker2 kubernetes]# cd server/bin
[root@worker2 bin]# rm -rf *_tag
[root@worker3 bin]# rm -rf *.tar
```

## 部署kube-apiserver

```shell
# 签发证书 200（worker4）
# 创建生成证书签名请求（csr）的JSON配置文件
vi /opt/certs/client-csr.json

{
    "CN": "k8s-node",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}

cd /opt/certs/
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client-csr.json | cfssl-json -bare client

vi apiserver-csr.json

{
    "CN": "k8s-apiserver",
    "hosts": [
        "127.0.0.1",
        "10.254.0.1",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local",
        "192.168.0.244",
        "192.168.0.245",
        "192.168.0.246",
        "192.168.0.247",
        "192.168.0.248"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server apiserver-csr.json | cfssl-json -bare apiserver
```

```shell
# 10.4.7.21、10.4.7.22 (woker2、worker3)
cd /opt/kubernetes/server/bin
[root@worker2 bin]# mkdir cert
[root@worker2 bin]# cd cert/
[root@worker2 cert]# scp worker4:/opt/certs/ca.pem .
[root@worker2 cert]# scp worker4:/opt/certs/ca-key.pem .
[root@worker2 cert]# scp worker4:/opt/certs/apiserver-key.pem .
[root@worker2 cert]# scp worker4:/opt/certs/apiserver.pem .
[root@worker2 cert]# scp worker4:/opt/certs/client.pem .
[root@worker2 cert]# scp worker4:/opt/certs/client-key.pem .

[root@worker2 cert]# cd ..
[root@worker2 bin]# mkdir conf
[root@worker2 bin]# cd conf/
[root@worker2 conf]# vi audit.yaml

apiVersion: audit.k8s.io/v1beta1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      # Resource "pods" doesn't match requests to any subresource of pods,
      # which is consistent with the RBAC policy.
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # Don't log requests to a configmap called "controller-leader"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # Don't log authenticated requests to certain non-resource URL paths.
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # Log the request body of configmap changes in kube-system.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # A catch-all rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"


[root@worker2 conf]# vim /opt/kubernetes/server/bin/kube-apiserver.sh
#!/bin/bash
./kube-apiserver \
  --apiserver-count 2 \
  --audit-log-path /data/logs/kubernetes/kube-apiserver/audit-log \
  --audit-policy-file ./conf/audit.yaml \
  --authorization-mode RBAC \
  --client-ca-file ./cert/ca.pem \
  --requestheader-client-ca-file ./cert/ca.pem \
  --enable-admission-plugins NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota \
  --etcd-cafile ./cert/ca.pem \
  --etcd-certfile ./cert/client.pem \
  --etcd-keyfile ./cert/client-key.pem \
  --etcd-servers https://10.4.7.12:2379,https://10.4.7.21:2379,https://10.4.7.22:2379 \
  --service-account-key-file ./cert/ca-key.pem \
  --service-cluster-ip-range 192.168.0.0/16 \
  --service-node-port-range 3000-29999 \
  --target-ram-mb=1024 \
  --kubelet-client-certificate ./cert/client.pem \
  --kubelet-client-key ./cert/client-key.pem \
  --log-dir  /data/logs/kubernetes/kube-apiserver \
  --tls-cert-file ./cert/apiserver.pem \
  --tls-private-key-file ./cert/apiserver-key.pem \
  --v 2
  
chmod +x /opt/kubernetes/server/bin/kube-apiserver.sh

vi /etc/supervisord.d/kube-apiserver.ini
[program:kube-apiserver-7-21]					# 21根据实际IP地址更改
command=sh /opt/kubernetes/server/bin/kube-apiserver.sh            ; the program (relative uses PATH, can take args)
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                            ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
stdout_logfile=/data/logs/kubernetes/kube-apiserver/apiserver.stdout.log        ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)

mkdir -p /data/logs/kubernetes/kube-apiserver

supervisorctl update
[root@worker2 bin]# supervisorctl status
etcd-server-7-21                 RUNNING   pid 2723, uptime 2:04:39
kube-apiserver-7-21              RUNNING   pid 2945, uptime 0:03:14
[root@worker2 ~]# netstat -luntp|grep kube-api
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      1059/./kube-apiserv
tcp6       0      0 :::6443                 :::*                    LISTEN      1059/./kube-apiserv
```

![image-20200810073250993](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200810073250993.png)

## 配置L4反向代理

* 10.4.7.11、12（master、worker1）上  配置四层反向代理

```shell
yum install nginx -y

vim /etc/nginx/nginx.conf
# 放在最后
stream {
	upstream kube-apiserver {
		server 10.4.7.21:6443 max_fails=3 fail_timeout=30s;
		server 10.4.7.22:6443 max_fails=3 fail_timeout=30s;
	}
	server {
		listen 7443;
		proxy_connect_timeout 2s;
		proxy_timeout 900s;
		proxy_pass kube-apiserver;
	}
}

[root@master ~]# nginx -t
[root@master ~]# systemctl start nginx
[root@master ~]# systemctl enable nginx

# 安装keepalived
yum install keepalived -y

vim /etc/keepalived/check_port.sh
#!/bin/bash
#keepalived 监控端口脚本
#使用方法：
#在keepalived的配置文件中
#vrrp_script check_port {#创建一个vrrp_script脚本,检查配置
#    script "/etc/keepalived/check_port.sh 6379" #配置监听的端口
#    interval 2 #检查脚本的频率,单位（秒）
#}
CHK_PORT=$1
if [ -n "$CHK_PORT" ];then
        PORT_PROCESS=`ss -lnt|grep $CHK_PORT|wc -l`
        if [ $PORT_PROCESS -eq 0 ];then
                echo "Port $CHK_PORT Is Not Used,End."
                exit 1
        fi
else
        echo "Check Port Cant Be Empty!"
fi

chmod +x /etc/keepalived/check_port.sh

# keepalived 主（12-master）：
vi /etc/keepalived/keepalived.conf      # 把原来的删掉
! Configuration File for keepalived

global_defs {
   router_id 10.4.7.11

}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_port.sh 7443"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33                 # ip add 查看本机实际的值 ，例如 enp0s3
    virtual_router_id 251
    priority 100
    advert_int 1
    mcast_src_ip 10.4.7.11
    nopreempt                       # 

    authentication {
        auth_type PASS
        auth_pass 11111111
    }
    track_script {
         chk_nginx
    }
    virtual_ipaddress {
        10.4.7.10
    }
}

# keepalived 从(10.4.7.12、worker1)：
vi /etc/keepalived/keepalived.conf      # 把原来的删掉
! Configuration File for keepalived
global_defs {
	router_id 10.4.7.12
	script_user root
        enable_script_security 
}
vrrp_script chk_nginx {
	script "/etc/keepalived/check_port.sh 7443"
	interval 2
	weight -20
}
vrrp_instance VI_1 {
	state BACKUP
	interface ens33                         # ip add 查看本机实际的值 ，例如 enp0s3
	virtual_router_id 251                   # ? 是否要和master的一致
	mcast_src_ip 10.4.7.12
	priority 90
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 11111111
	}
	track_script {
		chk_nginx
	}
	virtual_ipaddress {
		10.4.7.10
	}
}


[root@master ~]# systemctl start keepalived
[root@master ~]# systemctl enable keepalived

[root@worker1 ~]# netstat -luntp|grep 7443
tcp        0      0 0.0.0.0:7443            0.0.0.0:*               LISTEN      1811/nginx: master

[root@master ~]# ss -lnt|grep 7443|wc -l
1

# 查看日志
[root@worker1 ~]# tail -fn 200 /var/log/messages

[root@master ~]# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:8d:59:b4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.244/24 brd 192.168.0.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
		# 虚拟IP       
    inet 192.168.0.250/32 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::1a98:a51e:b495:beb9/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:65:c7:17:7f brd ff:ff:ff:ff:ff:ff
    inet 172.7.244.1/24 brd 172.7.244.255 scope global docker0
       valid_lft forever preferred_lft forever
```

![image-20200810194224472](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200810194224472.png)

## 部署controller-manager

![image-20200809144459056](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809144459056.png)

* 10.4.7.21、10.4.7.22（worker2、worker3）

  ```shell
  # 创建启动脚本
  vi /opt/kubernetes/server/bin/kube-controller-manager.sh
  
  #!/bin/sh
  ./kube-controller-manager \
    --cluster-cidr 172.7.0.0/16 \
    --leader-elect true \
    --log-dir /data/logs/kubernetes/kube-controller-manager \
    --master http://127.0.0.1:8080 \
    --service-account-private-key-file ./cert/ca-key.pem \
    --service-cluster-ip-range 192.168.0.0/16 \
    --root-ca-file ./cert/ca.pem \
    --v 2
    
  --------------------------
  # 实际配置
  #!/bin/sh
  ./kube-controller-manager \
    --cluster-cidr 172.7.0.0/16 \
    --leader-elect true \
    --log-dir /data/logs/kubernetes/kube-controller-manager \
    --master http://127.0.0.1:8080 \
    --service-account-private-key-file ./cert/ca-key.pem \
    --service-cluster-ip-range 10.4.0.0/16 \
    --root-ca-file ./cert/ca.pem \
    --v 2
    
  mkdir -p /data/logs/kubernetes/kube-controller-manager
  chmod +x /opt/kubernetes/server/bin/kube-controller-manager.sh
  
  # 创建supervisor配置
  vi /etc/supervisord.d/kube-conntroller-manager.ini
  
  [program:kube-controller-manager-7-21]            # 根据实际情况修改7-21、7-22   (实际没动，还是这些)
  command=sh /opt/kubernetes/server/bin/kube-controller-manager.sh                     ; the program (relative uses PATH, can take args)
  numprocs=1                                                                        ; number of processes copies to start (def 1)
  directory=/opt/kubernetes/server/bin                                              ; directory to cwd to before exec (def no cwd)
  autostart=true                                                                    ; start at supervisord start (default: true)
  autorestart=true                                                                  ; retstart at unexpected quit (default: true)
  startsecs=30                                                                      ; number of secs prog must stay running (def. 1)
  startretries=3                                                                    ; max # of serial start failures (default 3)
  exitcodes=0,2                                                                     ; 'expected' exit codes for process (default 0,2)
  stopsignal=QUIT                                                                   ; signal used to kill process (default TERM)
  stopwaitsecs=10                                                                   ; max num secs to wait b4 SIGKILL (default 10)
  user=root                                                                         ; setuid to this UNIX account to run the program
  redirect_stderr=true                                                              ; redirect proc stderr to stdout (default false)
  stdout_logfile=/data/logs/kubernetes/kube-controller-manager/controller.stdout.log  ; stderr log path, NONE for none; default AUTO
  stdout_logfile_maxbytes=64MB                                                      ; max # logfile bytes b4 rotation (default 50MB)
  stdout_logfile_backups=4                                                          ; # of stdout logfile backups (default 10)
  stdout_capture_maxbytes=1MB                                                       ; number of bytes in 'capturemode' (default 0)
  stdout_events_enabled=false                                                       ; emit events on stdout writes (default false)
  
  # 启动服务并检测
  [root@worker2 ~]# supervisorctl update
  kube-controller-manager-7-21: added process group
  [root@worker2 ~]# supervisorctl status
  etcd-server-7-21                 RUNNING   pid 1052, uptime 12:14:54
  kube-apiserver-7-21              RUNNING   pid 1053, uptime 12:14:54
  kube-controller-manager-7-21     RUNNING   pid 2907, uptime 0:00:39
  ```

## 部署kube-scheduler

* 10.4.7.21、10.4.7.22（worker2、worker3）

  ```shell
  # 创建启动脚本
  vi /opt/kubernetes/server/bin/kube-scheduler.sh
  
  #!/bin/sh
  ./kube-scheduler \
    --leader-elect  \
    --log-dir /data/logs/kubernetes/kube-scheduler \
    --master http://127.0.0.1:8080 \
    --v 2
   
  # 创建supervisor配置 
  vi /etc/supervisord.d/kube-scheduler.ini
  
  [program:kube-scheduler-7-21]                # 根据实际情况修改7-21、7-22   (实际没动，还是这些)
  command=sh /opt/kubernetes/server/bin/kube-scheduler.sh                     ; the program (relative uses PATH, can take args)
  numprocs=1                                                               ; number of processes copies to start (def 1)
  directory=/opt/kubernetes/server/bin                                     ; directory to cwd to before exec (def no cwd)
  autostart=true                                                           ; start at supervisord start (default: true)
  autorestart=true                                                         ; retstart at unexpected quit (default: true)
  startsecs=30                                                             ; number of secs prog must stay running (def. 1)
  startretries=3                                                           ; max # of serial start failures (default 3)
  exitcodes=0,2                                                            ; 'expected' exit codes for process (default 0,2)
  stopsignal=QUIT                                                          ; signal used to kill process (default TERM)
  stopwaitsecs=10                                                          ; max num secs to wait b4 SIGKILL (default 10)
  user=root                                                                ; setuid to this UNIX account to run the program
  redirect_stderr=true                                                     ; redirect proc stderr to stdout (default false)
  stdout_logfile=/data/logs/kubernetes/kube-scheduler/scheduler.stdout.log ; stderr log path, NONE for none; default AUTO
  stdout_logfile_maxbytes=64MB                                             ; max # logfile bytes b4 rotation (default 50MB)
  stdout_logfile_backups=4                                                 ; # of stdout logfile backups (default 10)
  stdout_capture_maxbytes=1MB                                              ; number of bytes in 'capturemode' (default 0)
  stdout_events_enabled=false                                              ; emit events on stdout writes (default false) 
  
  chmod +x /opt/kubernetes/server/bin/kube-scheduler.sh
  mkdir -p /data/logs/kubernetes/kube-scheduler
  
  # 启动服务并检测
  [root@worker2 ~]# supervisorctl update
  kube-scheduler-7-21: added process group
  [root@worker3 ~]# supervisorctl status
  etcd-server-7-22                 RUNNING   pid 1039, uptime 12:22:12
  kube-apiserver-7-22              RUNNING   pid 1035, uptime 12:22:12
  kube-controller-manager-7-22     RUNNING   pid 2907, uptime 0:07:04
  kube-scheduler-7-22              RUNNING   pid 2931, uptime 0:00:51
  
  # 检测集群状态
  ln -s /opt/kubernetes/server/bin/kubectl /usr/bin/kubectl
  [root@worker3 ~]# which kubectl
  /usr/bin/kubectl
  [root@worker2 ~]# kubectl get cs
  NAME                 STATUS      MESSAGE                                                                    
  ERROR
  scheduler            Healthy     ok
  controller-manager   Healthy     ok
  etcd-0               Healthy     {"health": "true"}
  etcd-1               Healthy     {"health": "true"}
  etcd-2               Unhealthy   Get https://192.168.0.247:2379/health: net/http: TLS handshake timeout
  ```

  ![image-20200810201111855](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200810201111855.png)

# 部署运算节点服务

## kubelet

* 签发kubelet证书
  * 10.4.7.200（worker4）

```shell
# 创建生成证书签名请求(csr)的JSON配置文件
cd /opt/certs/
vi kubelet-csr.json

{
    "CN": "k8s-kubelet",
    "hosts": [
    "127.0.0.1",
    "192.168.16.11",
    "192.168.16.12",
    "192.168.16.13",
    "192.168.16.14",
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}

----------------------
# 实际配置
{
    "CN": "k8s-kubelet",
    "hosts": [
    "127.0.0.1",
    "192.168.0.244",
    "192.168.0.245",
    "192.168.0.246",
    "192.168.0.247",
    "192.168.0.248",
    "192.168.0.249",
    "192.168.0.250"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server kubelet-csr.json | cfssl-json -bare kubelet
```

* 10.4.7.21、10.4.7.22（worker2、worker3）

```shell
# 以下没做说明，则只在 10.4.7.21 上操作

cd /opt/kubernetes/server/bin/cert
[root@worker2 cert]# scp hdss7-200:/opt/certs/kubelet.pem .
[root@worker2 cert]# scp hdss7-200:/opt/certs/kubelet-key.pem .

# 分发证书
cd /opt/kubernetes/server/bin/conf
# 创建配置
## set-cluster        # 虚拟IP  10.4.7.10
[root@worker2 conf]# kubectl config set-cluster myk8s \
    --certificate-authority=/opt/kubernetes/server/bin/cert/ca.pem \
    --embed-certs=true \
    --server=https://10.4.7.10:7443 \
    --kubeconfig=kubelet.kubeconfig
# Cluster "myk8s" set.
-----------------
# 实际配置
[root@worker2 conf]# kubectl config set-cluster myk8s \
    --certificate-authority=/opt/kubernetes/server/bin/cert/ca.pem \
    --embed-certs=true \
    --server=https://192.168.0.250:7443 \
    --kubeconfig=kubelet.kubeconfig

## set-credentials
[root@worker2 conf]# kubectl config set-credentials k8s-node \
  --client-certificate=/opt/kubernetes/server/bin/cert/client.pem \
  --client-key=/opt/kubernetes/server/bin/cert/client-key.pem \
  --embed-certs=true \
  --kubeconfig=kubelet.kubeconfig 
# User "k8s-node" set.

## set-context
[root@worker2 conf]# kubectl config set-context myk8s-context \
  --cluster=myk8s \
  --user=k8s-node \
  --kubeconfig=kubelet.kubeconfig
# Context "myk8s-context" created.

## use-context  
[root@worker2 conf]# kubectl config use-context myk8s-context --kubeconfig=kubelet.kubeconfig 
# Switched to context "myk8s-context".


# 授予权限，角色绑定	-- 只创建一次就好，存到etcd里,然后拷贝到各个node节点上
vi k8s-node.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: k8s-node
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node
subjects:
- apiGroup: rbac.authorization.k8s.io
	kind: User
	name: k8s-node

[root@worker2 conf]# kubectl create -f k8s-node.yaml
clusterrolebinding.rbac.authorization.k8s.io/k8s-node created
[root@worker2 conf]# kubectl get clusterrolebinding k8s-node
NAME       AGE
k8s-node   55s
[root@worker2 conf]# kubectl get clusterrolebinding k8s-node -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding             # k8s一切皆资源
metadata:
  creationTimestamp: "2020-08-10T12:30:36Z"
  name: k8s-node
  resourceVersion: "12802"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/k8s-node
  uid: 5db31f5f-106a-486d-97d2-6a5cd870cc5b
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: k8s-node        # k8s-node用户成为集群运算节点的权限
  
# HDSS7-22（10.4.7.22）上  
cd /opt/kubernetes/server/bin/conf
scp hdss7-13:/opt/kubernetes/server/bin/conf/kubelet.kubeconfig .
```

```shell
# 准备pause基础镜像
# 10.4.7.200（worker4）上
docker pull kubernetes/pause
# 提交至私有仓库（harbor）中
docker tag f9d5de079539 harbor.od.com/public/pause:latest
docker push harbor.od.com/public/pause:latest
```

```shell
# 创建kubelet启动脚本
# 10.4.7.21、10.4.7.22（worker2、worker3）
vi /opt/kubernetes/server/bin/kubelet.sh

#!/bin/sh
./kubelet \
  --anonymous-auth=false \
  --cgroup-driver systemd \
  --cluster-dns 192.168.0.2 \               # 根据实际情况修改
  --cluster-domain cluster.local \
  --runtime-cgroups=/systemd/system.slice \
  --kubelet-cgroups=/systemd/system.slice \
  --fail-swap-on="false" \
  --client-ca-file ./cert/ca.pem \
  --tls-cert-file ./cert/kubelet.pem \
  --tls-private-key-file ./cert/kubelet-key.pem \
  --hostname-override hdss7-21.host.com \		       # # 根据实际情况修改	
  --kubeconfig ./conf/kubelet.kubeconfig \
  --log-dir /data/logs/kubernetes/kube-kubelet \
  --pod-infra-container-image harbor.od.com/public/pause:latest \
  --root-dir /data/kubelet

----------------------------
# 实际配置
#!/bin/sh
./kubelet \
  --anonymous-auth=false \
  --cgroup-driver systemd \
  --cluster-dns 10.4.0.2 \
  --cluster-domain cluster.local \
  --runtime-cgroups=/systemd/system.slice \
  --kubelet-cgroups=/systemd/system.slice \
  --fail-swap-on="false" \
  --client-ca-file ./cert/ca.pem \
  --tls-cert-file ./cert/kubelet.pem \
  --tls-private-key-file ./cert/kubelet-key.pem \
  --hostname-override worker2.host.com \
  --kubeconfig ./conf/kubelet.kubeconfig \
  --log-dir /data/logs/kubernetes/kube-kubelet \
  --pod-infra-container-image harbor.od.com/public/pause:latest \
  --root-dir /data/kubelet
   
mkdir -p /data/logs/kubernetes/kube-kubelet /data/kubelet

chmod +x /opt/kubernetes/server/bin/kubelet.sh
```

```shell
# 创建supervisor配置
# 10.4.7.21、10.4.7.22（worker2、worker3）
vi /etc/supervisord.d/kube-kubelet.ini

[program:kube-kubelet-7-21]	             # 根据实际情况修改
command=sh /opt/kubernetes/server/bin/kubelet.sh     ; the program (relative uses PATH, can take args)
numprocs=1                                        ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin              ; directory to cwd to before exec (def no cwd)
autostart=true                                    ; start at supervisord start (default: true)
autorestart=true              		          ; retstart at unexpected quit (default: true)
startsecs=30                                      ; number of secs prog must stay running (def. 1)
startretries=3                                    ; max # of serial start failures (default 3)
exitcodes=0,2                                     ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                   ; signal used to kill process (default TERM)
stopwaitsecs=10                                   ; max num secs to wait b4 SIGKILL (default 10)
user=root                                         ; setuid to this UNIX account to run the program
redirect_stderr=true                              ; redirect proc stderr to stdout (default false)
stdout_logfile=/data/logs/kubernetes/kube-kubelet/kubelet.stdout.log   ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                      ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                          ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                       ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                       ; emit events on stdout writes (default false)

[root@worker2 conf]# supervisorctl update
kube-kubelet-7-21: added process group
# supervisorctl help
# supervisorctl reload
# supervisorctl restart kube-kubelet-7-21
[root@worker3 bin]# supervisorctl status
etcd-server-7-22                 RUNNING   pid 6024, uptime 0:00:43
kube-apiserver-7-22              RUNNING   pid 6023, uptime 0:00:43
kube-controller-manager-7-22     RUNNING   pid 6020, uptime 0:00:43
kube-kubelet-7-22                RUNNING   pid 6019, uptime 0:00:43
kube-scheduler-7-22              RUNNING   pid 6021, uptime 0:00:43

# 查看日志
tail -fn 200 /data/logs/kubernetes/kube-kubelet/kubelet.stdout.log

# 正常情况
[root@hdss7-21 cert]# kubectl get nodes
NAME                STATUS   ROLES    AGE     VERSION
hdss7-21.host.com   Ready    <none>   15h     v1.15.2
hdss7-22.host.com   Ready    <none>   8m51s   v1.15.2

# 失败情况，原因还未找到
[root@worker3 bin]# kubectl get nodes
No resources found.
[root@worker3 bin]# systemctl status kubelet
Unit kubelet.service could not be found.

# ROlES添加标签，设定节点角色，可同时加两个标签
[root@hdss7-21 cert]# kubectl label node hdss7-21.host.com node-role.kubernetes.io/master=
[root@hdss7-21 cert]# kubectl label node hdss7-21.host.com node-role.kubernetes.io/node=

[root@hdss7-22 ~]# kubectl get nodes                               
NAME                STATUS   ROLES         AGE   VERSION
hdss7-21.host.com   Ready    master,node   15h   v1.15.2
hdss7-22.host.com   Ready    master,node   12m   v1.15.2
```

![image-20200811075205028](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811075205028.png)

## kube-proxy

* 连接pod网络和集群网络
* 21、22 (worker2、worker3)

```shell
# 签发kube-proxy证书
# 运维主机HDSS7-200.host.com
# 签发生成证书签名请求（CSR）的JSON配置文件
vi /opt/certs/kube-proxy-csr.json
{
    "CN": "system:kube-proxy",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}

# 生成证书
cd /opt/certs/
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client kube-proxy-csr.json |cfssl-json -bare kube-proxy-client
[root@hdss7-200 certs]# ll
-rw-r--r-- 1 root root 1005 12月 12 10:23 kube-proxy-client.csr
-rw------- 1 root root 1679 12月 12 10:23 kube-proxy-client-key.pem
-rw-r--r-- 1 root root 1375 12月 12 10:23 kube-proxy-client.pem
-rw-r--r-- 1 root root  267 12月 12 10:22 kube-proxy-csr.json
```

```shell
# 分发证书，将证书拷贝到node节点，注意私钥文件属性600
cd /opt/kubernetes/server/bin/cert/
[root@hdss7-21 ~]# cd /opt/kubernetes/server/bin/cert/
[root@hdss7-21 cert]# scp hdss7-200:/opt/certs/kube-proxy-client-key.pem .
[root@hdss7-21 cert]# scp hdss7-200:/opt/certs/kube-proxy-client.pem .

# 在conf文件夹下创建配置 -- 只做一次，然后将kube-proxy.kubeconfig拷贝至各个node节点

[root@hdss7-21 cert]# cd /opt/kubernetes/server/bin/conf

[root@hdss7-21 conf]# kubectl config set-cluster myk8s \
  --certificate-authority=/opt/kubernetes/server/bin/cert/ca.pem \
  --embed-certs=true \
  --server=https://10.4.7.10:7443 \
  --kubeconfig=kube-proxy.kubeconfig

[root@hdss7-21 conf]# ls
audit.yaml  k8s-node.yaml  kubelet.kubeconfig  kube-proxy.kubeconfig

[root@hdss7-21 conf]# kubectl config set-credentials kube-proxy \
  --client-certificate=/opt/kubernetes/server/bin/cert/kube-proxy-client.pem \
  --client-key=/opt/kubernetes/server/bin/cert/kube-proxy-client-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

[root@hdss7-21 conf]# kubectl config set-context myk8s-context \
  --cluster=myk8s \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

[root@hdss7-21 conf]# kubectl config use-context myk8s-context --kubeconfig=kube-proxy.kubeconfig


# 第一台node节点部署完成后，将生成的配置文件拷贝至各个Node节点
[root@hdss7-22 cert]# cd /opt/kubernetes/server/bin/conf
[root@hdss7-22 conf]# scp hdss7-21:/opt/kubernetes/server/bin/conf/kube-proxy.kubeconfig .
```

```shell
# 10.4.7.21、10.4.7.22上（worker2、worker3）
# 加载ipvs模块	-- 脚本需要设置成开启自动运行
[root@hdss7-21 conf]# vi /root/ipvs.sh
#!/bin/bash
ipvs_mods_dir="/usr/lib/modules/$(uname -r)/kernel/net/netfilter/ipvs"
for i in $(ls $ipvs_mods_dir|grep -o "^ (.)*")
do
  /sbin/modinfo -F filename $i &>/dev/null
  if [ $? -eq 0 ];then
    /sbin/modprobe $i
  fi
done
[root@hdss7-21 conf]# chmod +x /root/ipvs.sh 
# 执行脚本
[root@hdss7-21 conf]# /root/ipvs.sh 
# 查看内核是否加载ipvs模块
[root@hdss7-21 conf]# lsmod | grep ip_vs

# 创建kube-proxy启动脚本
[root@hdss7-21 ~]# vi /opt/kubernetes/server/bin/kube-proxy.sh
#!/bin/sh
./kube-proxy \
  --cluster-cidr 172.7.0.0/16 \
  --hostname-override hdss7-21.host.com \       # 根据实际情况修改
  --proxy-mode=ipvs \
  --ipvs-scheduler=nq \
  --kubeconfig ./conf/kube-proxy.kubeconfig
  
[root@hdss7-21 ~]# chmod +x /opt/kubernetes/server/bin/kube-proxy.sh
[root@hdss7-21 ~]# mkdir -p /data/logs/kubernetes/kube-proxy
[root@hdss7-21 ~]# vi /etc/supervisord.d/kube-proxy.ini
[program:kube-proxy-7-21]                    # 根据实际情况修改 7-21、7-22
command=/opt/kubernetes/server/bin/kube-proxy.sh                     ; the program (relative uses PATH, can take args)
numprocs=1                                                           ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                 ; directory to cwd to before exec (def no cwd)
autostart=true                                                       ; start at supervisord start (default: true)
autorestart=true                                                     ; retstart at unexpected quit (default: true)
startsecs=30                                                         ; number of secs prog must stay running (def. 1)
startretries=3                                                       ; max # of serial start failures (default 3)
exitcodes=0,2                                                        ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                      ; signal used to kill process (default TERM)
stopwaitsecs=10                                                      ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                            ; setuid to this UNIX account to run the program
redirect_stderr=true                                                 ; redirect proc stderr to stdout (default false)
stdout_logfile=/data/logs/kubernetes/kube-proxy/proxy.stdout.log     ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                         ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                             ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                          ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                          ; emit events on stdout writes (default false)

[root@hdss7-21 ~]# supervisorctl update
[root@hdss7-21 ~]# supervisorctl status
kube-proxy-7-21                  RUNNING   pid 6873, uptime 0:28:15
[root@hdss7-22 ~]# netstat -luntp |grep kube-proxy
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      7310/./kube-proxy   
tcp6       0      0 :::10256                :::*                    LISTEN      7310/./kube-proxy  
# 查看日志
tail -fn 200 /data/logs/kubernetes/kube-proxy/proxy.stdout.log

# 查看ipvs是否生效
[root@hdss7-21 ~]# yum install -y ipvsadm	# 只安装，不启动
[root@hdss7-21 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.1:443 nq
  -> 10.4.7.21:6443          Masq    1      0          0         
  -> 10.4.7.22:6443          Masq    1      0          0  



# 暂时忽略
# 设置开机自动启动	
[root@hdss7-21 ~]# vi /etc/rc.d/rc.local
/root/ipvs.sh

# 开启开机自启动脚本功能  -- 详见本文件夹内 开启开机自启动脚本文件
[root@hdss7-21 ~]# chmod +x /etc/rc.d/rc.local
[root@hdss7-21 ~]# mkdir -p /usr/lib/system/system/
[root@hdss7-21 ~]# vim /usr/lib/system/system/rc-local.service
[Install]
WantedBy=multi-user.target
[root@hdss7-21 ~]# ln -s '/lib/systemd/system/rc-local.service' '/etc/systemd/system/multi-user.target.wants/rc-local.service'
# 开启 rc-local.service 服务：
[root@hdss7-21 ~]# systemctl start rc-loacl.service
[root@hdss7-21 ~]# systemctl enable rc-local.service
```

![image-20200811081202303](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811081202303.png)

![image-20200811081218729](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811081218729.png)

![image-20200811081909475](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811081909475.png)

![image-20200811082114829](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811082114829.png)

![image-20200811082051435](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811082051435.png)

# 完成部署并验证集群

* 在任意一个运算节点，创建一个资源配置清单
  * 这里选择DHSS7-21.host.com主机

```shell
[root@hdss7-21 ~]# vi /root/nginx-ds.yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ds
spec:
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: my-nginx
        image: harbor.od.com/public/nginx:v1.7.9
        ports:
        - containerPort: 80
        
# 测试完删除
[root@hdss7-21 ~]# kubectl create -f nginx-ds.yaml 
daemonset.extensions/nginx-ds created

[root@hdss7-21 ~]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-72zm5   1/1     Running   0          86s   172.7.21.2   hdss7-21.host.com   <none>           <none>
nginx-ds-cb7ct   1/1     Running   0          86s   172.7.22.2   hdss7-22.host.com   <none>           <none>

[root@hdss7-21 ~]curl 172.7.21.2
# 有内容
[root@hdss7-21 ~]curl 172.7.22.2
# 没有内容  跨宿主主机容器还不能通信

[root@hdss7-21 ~]# kubectl delete -f nginx-ds.yaml 
[root@hdss7-21 ~]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}   
etcd-1               Healthy   {"health": "true"} 
        
```

![image-20200811083031036](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811083031036.png)

# 资源需求说明

![image-20200811083117321](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200811083117321.png)


