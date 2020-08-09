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

  ![image-20200809105647098](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809105647098.png)

* Name/Namespace

  ![image-20200809113337825](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809113337825.png)

* Label/Label选择器

  ![image-20200809114050259](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809114050259.png)

* service/Ingress

  ![image-20200809115844336](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809115844336.png)

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

![image-20200809151538759](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809151538759.png)

```shell
vi /etc/selinux/config
将SELINUX=enforcing改为SELINUX=disabled
设置后需要重启才能生效
systemctl stop firewalld
```

```shell
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -y
```

![image-20200809153059515](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809153059515.png)

```shell
10.4.7.11
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
	
# stics-file "/var/named/data/named_stats.txt";
# memstatistics-file "/var/named/data/named_mem_stats.txt";
# rcursing-file  "/var/named/data/named.recursing";
# secroots-file   "/var/named/data/named.secroots";
# allow-query     { any; };
# forwarders      { 192.168.0.255; };	

# 检查配置	
named-checkconf	

# 区域配置文件
vim /etc/named.rfc1912.zones
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

![image-20200809155018095](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809155018095.png)

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

![image-20200809155219302](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809155219302.png)

```shell
named-checkconf
systemctl start named
netstat -luntp|grep 53
dig -t A hdss7-21.host.com @10.4.7.11 +short
dig -t A hdss7-200.host.com @10.4.7.11 +short
```

![image-20200809163244812](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809163244812.png)

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	DNS1=10.4.7.11
cat /etc/resolv.conf	# vi
	# Generated by NetworkManager
	search host.com
	nameserver 10.4.7.11
ystemctl restart network
ping www.baidu.com
ping hdss7-21.host.com
ping hdss7-200
# 机器都改成
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	DNS1=10.4.7.11
systemctl restart network


# mac上修改DNS，改为10.4.7.11
```

![image-20200809170200139](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200809170200139.png)

## 签发证书

```shell
# 安装CFSSL
# worker4上
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

```shell
mkdir -p /etc/docker /data/docker
vim /etc/docker/daemon.json

#各机器的bip不同，分别对应
# 如 10.4.7.21  为 172.7.21.1/24
# 如 10.4.7.22  为 172.7.22.1/24
# 如 10.4.7.200  为 172.7.200.1/24
# 如 10.4.7.12  为 172.7.12.1/24
# 如 10.4.7.11  为 172.7.11.1/24

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

```shell
#10.4.7.200上 （worker4上）
tar -zxvf /data/harbor-offline-installer-v2.0.2.tgz -C /opt/
[root@worker4 opt]# mv harbor/ harbor-v2.0.2
[root@worker4 opt]# ln -s /opt/harbor-v2.0.2/ /opt/harbor

cd harbor
vi harbor.yml
	hostname: harbor.od.com
	http:
		port: 180
	log:
		location: /data/harbor/logs
	data_volume: /data/harbor		

mkdir -p /data/harbor/logs /data/harbor

yum install docker-compose -y

rpm -qa docker-compose
# 安装
./install.sh

docker-compose ps 

yum install nginx -y

vim /etc/nginx/conf.d/harbor.od.com.conf
server {
	listen			80;
	server_name	harbor.od.com;
	client_max_body_size 1000m;
	location / {
		proxy_pass http://127.0.0.1:180;
	}
}

nginx -t
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
useradd -s /sbin/nologin -M etcd
id etcd
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
command=sh /opt/etcd/etcd-server-startup.sh                        ; the program (relative uses PATH, can take args)	
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
[root@worker1 etcd]# tail -fn 200 /data/logs/etcd-server/etcd.stdout.log

# 任意节点检测etcd集群健康状态
[root@worker2 etcd]# ./etcdctl cluster-health
[root@worker2 etcd]# ./etcdctl member list
```

# 部署运算节点服务

```shell
# 10.4.7.21、10.4.7.21 (woker2、worker3)
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
# 10.4.7.21、10.4.7.21 (woker2、worker3)
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
```

# 19

# 完成部署并验证集群

# 资源需求说明





