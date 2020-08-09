# 内容

* 快速入门
  * 核心概念、架构设计、认证授权

* 高可用集群搭建
  * 二进制或kubeadm
    * 网络插件：calico
    * dns：coredns
    * 看板：dashboard

* 业务系统迁移Kubernetes
  * 准备工作
    * Harbor：架构、原理、部署高可用Harbor仓库
    * 服务发现
    * IngressNginx：外部服务发现方案
  * 最佳实践
    * Docker化
    * 跑在k8s
    * 服务发现

* CICD实践

![image-20191110090131731](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110090131731.png)

* 深入k8s
  * 几个重要的资源对象
  * namespace
    * 对资源对象和资源配额的多层面隔离机制，pod资源限制了各种配置方式，pod的k8s，以及它跟资源配额的关系，和pod和它节点资源准确时候的驱逐机制
  * resources
  * label：作用于不同资源对象上的不同作用
  * 合理的规划命名空间，通过资源配额来提供服务的稳定性，可以设置驱逐策略来提升系统的稳定性，以及灵活的运用label来为各种资源打标签

* 服务调度与编排
  * 健康检查
    * pod的健康检查如何工作、参数该如何配置及优化
  * 调度策略
    * 预选策略和优选策略
  * 部署策略
  * 深入pod
    * 思想、生命周期、设计

* 落地与实践
  * Ingress-Nginx
    * ab测试、蓝绿部署、小能量测试
  * PV / PVC / StorageClass
    * 共享存储
  * StatefulSet
  * Kubernetes API

* 日志与监控
  * 日志主流方案
  * 从日志采集到日志展示
  * Prometheus

* Istio
  * 架构设计-环境部署-数据展现

* 环境参数
  * Kubernetes 1.14.0
  * Docker 17.03.x
  * Java 1.8
  * Harbor 1.6.0
  * Prometheus 2.8.1
  * Istio 1.1.2

# 必知必会

* k8s是什么？

  * 开源、自动化部署扩缩容、管理容器化应用
  * 得益于 docker 的特性，服务的创建和销毁变得非常快速、简单。Kubernetes 正是以此为基础，实现了集群规模的管理、编排方案，使应用的发布、重启、扩缩容能够自动化

* k8s认知

  * Kubernetes 可以管理大规模的集群，使集群中的每一个节点彼此连接，能够像控制一台单一的计算机一样控制整个集群

  * 两种角色

    * master：是集群的"大脑"，负责管理整个集群，像应用的调度、更新、扩缩容等
    * node(workder)：具体"干活"的，一个Node一般是一个虚拟机或物理机，它上面事先运行着 docker 服务和 kubelet 服务（ Kubernetes 的一个组件），当接收到 master 下发的"任务"后，Node 就要去完成任务（用 docker 运行一个指定的应用）

  * Deployment：应用管理者

    1. 事先准备好docker镜像

    2. 通过Kubernetes的 **Deployment** 的配置文件去描述应用，比如应用叫什么名字、使用的镜像名字、要运行几个实例、需要多少的内存资源、cpu 资源等等
    3. 有了配置文件就可以通过Kubernetes提供的命令行客户端 - **kubectl** 去管理这个应用了
       * kubectl 会跟 Kubernetes 的 master 通过RestAPI通信，最终完成应用的管理
       * 比如配置个 Deployment 配置文件叫 app.yaml，就可以通过 "kubectl create -f app.yaml" 来创建这个应用，之后就由 Kubernetes 来保证应用处于运行状态，当某个实例运行失败了或者运行着应用的 Node 突然宕机了，Kubernetes 会自动发现并在新的 Node 上调度一个新的实例，保证我们的应用始终达到我们预期的结果

  * POD：Kubernetes最小调度单位

    * 应用在每个 Node 上运行的其实是一个 Pod。Pod 也只能运行在 Node 上
    * Pod 是一组容器（当然也可以只有一个）。容器本身就是一个小盒子了，Pod 相当于在容器上又包了一层小盒子，盒子里面的容器特点：
      * 可以直接通过 volume 共享存储
      * 有相同的网络空间，通俗点说就是有一样的ip地址，有一样的网卡和网络设置
      * 多个容器之间可以“了解”对方，比如知道其他人的镜像，知道别人定义的端口等
      * ![image-20191112101330070](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191112101330070.png)

  * #### Service - 服务发现 - 找到每个Pod

    1. 最直接想到的方法：直接通过 Pod-ip+port 去访问
       * 但如果实例数很多呢？好，拿到所有的 Pod-ip 列表，配置到负载均衡器中，轮询访问。但Pod 可能会死掉，甚至 Pod 所在的 Node 也可能宕机，Kubernetes 会自动重新创建新的Pod。再者每次更新服务的时候也会重建 Pod。而每个 Pod 都有自己的 ip。所以 Pod 的ip 是不稳定的，会经常变化的
    2. Service，来专门解决这个问题，不管Deployment的Pod有多少个，不管它是更新、销毁还是重建，Service总是能发现并维护好它的ip列表。Service对外也提供了多种入口
       * ClusterIP：Service 在集群内的唯一 ip 地址，可以通过这个 ip，均衡的访问到后端的 Pod，而无须关心具体的 Pod
       * NodePort：Service 会在集群的每个 Node 上都启动一个端口，可以通过任意Node 的这个端口来访问到 Pod
       * LoadBalancer：在 NodePort 的基础上，借助公有云环境创建一个外部的负载均衡器，并将请求转发到 NodeIP:NodePort
       * ExternalName：将服务通过 DNS CNAME 记录方式转发到指定的域名（通过 spec.externlName 设定）
       * ![image-20191112102210439](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191112102210439.png)
    3. Service是如何知道它负责哪些 Pod 呢？是如何跟踪这些 Pod 变化的？
       * 最容易想到的方法是使用 Deployment 的名字。一个 Service 对应一个 Deployment 。当然这样确实可以实现。但k ubernetes 使用了一个更加灵活、通用的设计 - Label 标签，通过给 Pod 打标签，Service 可以只负责一个 Deployment 的 Pod 也可以负责多个 Deployment 的 Pod 了。Deployment 和 Service 就可以通过 Label 解耦了
       * ![image-20191112102439801](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191112102439801.png)

  * #### RollingUpdate - 滚动升级

    * 主要思路是一边增加新版本应用的实例数，一边减少旧版本应用的实例数，直到新版本的实例数达到预期，旧版本的实例数减少为0，滚动升级结束。在整个升级过程中，服务一直处于可用状态。并且可以在任意时刻回滚到旧版本
    * ![05_rollingupdate](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/05_rollingupdate.gif)

* 主要特征

  * 以服务为中心，不用关心服务运行的环境、细节
  * 自动化，自动扩缩容、升级、更新、部署
    * 接收到相应的指令后会触发相应调度流程、选中目标节点、部署或停止相关服务，如果有新的pod启动会自动加入负载均衡器、自动生效，在服务的运行过程中，k8s会定期检查实例数以及实例的状态是否正常，当发现实例不可用则自动销毁该实例，重新调度新的实例，这些全是自动化完成，不需人工干预

* k8s和Docker

  * k8s可看成是docker的上层架构，就像Java和Java EE的关系
  * k8s以docker技术的标准为基础，打造全新的分布式架构系统

* k8s架构设计

  * Deployment：部署，更新应用时，其会创建一个新ReplicaSet(RS)然后启动一个带更新应用的POD，新POD中更新的应用通过健康检查后，其会通知老版本ReplicaSet(RS)删除老版本POD，最后也会清理掉老ReplicaSet(RS)

    * ReplicaSet(RS)：副本集，管理多个POD
      * POD：一个或多个容器，里面的所有容器都运行在一台机器上，里面容器共享网络，有一个唯一IP，每个POD里都有个Pause容器（作为根容器，将其他容器都link到一起，并负责整个POD的健康检查并向k8s会报）
        * Pause
        * Container:user-image:v1
        * Container:login-image:v2
    * Service
      * 先给POD打标签：app:login
      * Selector(app=login)：Service根据标签找POD
      * 对外ClusterIP，Client通过ClusterIP访问到Service

  * 滚动部署

    ![image-20191110103229240](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110103229240.png)

* 集群

  * 存储组件：ETCD

  * 访问集群：ApiServer，唯一入口

  * 调度器选择节点：Scheduler

    * 收集每个worker的详细信息，包括资源、内存、CPU、节点上运行的服务，通过预选策略或优选策略最终选择一个最优节点，然后将节点与POD建立关系，然后告诉ApiServer这个POD运行在某个节点上，ApiServer把这个消息存在ETCD里进行持久化——POD和Node进行绑定

  * 启动POD：ControllerManager

    * 集群内部的控制中心，负责维护各种k8s镜像
      * ServiceController：管理服务
      * EndPointController：管理POD列表
      * ReplicaController：管理副本
      * SourceController：管理资源配额
    * 其通过ApiServer获取一些节点的变化，通过目录发现一些POD当前处于等待调度状态，然后完成调度，让POD运行起来

  * POD如何在Worker节点上运行起来

    * worker节点上安装kubelet服务，其负责维护POD的生命周期，包括容器的Volume、网络管理
    * kubelet调用本机docker，实现运行起容器、运行POD

    ![image-20191110105652816](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110105652816.png)

* 认证和授权

  * 目的是访问ApiServer

  * 认证

    * 客户端证书认证（TLS双向认证）

      * kubectl和ApiServer双向认证

      ![image-20191110113413648](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110113413648.png)

    * Bearer Token认证

      * ApiServer中预先定义复杂密码，然后把密码告诉自定义客户端，客户端和ApiServer进行通信时带上BearerToken，ApiServer验证密码没问题则可以进行通讯

    * ServiceAccount认证

      * k8s内部认证，ServiceAccount和k8s中其他资源类似，可以创建
      * 其namespace、token、ca-验证ApiServer的证书，通过目录挂载的方式挂载到POD的文件系统里，应用通过读取目录里的文件获取这些信息，拿到这些信息后就可以和ApiServer进行交互

  * 授权

    * RBAC（Role Based Access Control）
      * Role
        * User - Role - Authority
        * User
          * User：普通用户
          * ServiceAccount：用于集群内部访问ApiServer
        * Authority
          * Resource
          * Verbs：curd
        * Role（放到某个namespace下，只能拥有该命名空间下的角色）
          * name
          * resource：哪些权限
          * verbs：哪些操作
          * RoleBinding
        * ClusterRole
          * ClusterRoleBinding
            * Resource
            * verbs
            * 定义集群范围内资源
              * 并不是所有的资源都属于一个namespace，比如node和nodes返回集群的机器列表，node这个资源就不属于任何namespace，但是它也是一种资源，如果要给某个用户node的操作权限，就需要给该用户定义个ClusterRole并加入node资源，这样角色控制比较灵活，能够满足各种需求
              * 如果某个用户需要一个或几个namespace下的权限，可以定义几个Role，并用RoleBinding将其关联起来；如果用户需求整个集群POD Service的访问权限，可以定义个ClusterRole，在这个Role的Resource里自定义POD Service，然后再建立个ClusterRoleBinding进行ClusterRole绑定，然后可以访问集群范围内的POD和Service了，不受命名空间的制约了
            * RoleBinding和ClusterRoleBing都支持User和ServiceAccount
            * ![image-20191110153415892](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110153415892.png)

  * AdmisionControl

    * 独立的小插件、代码，一个一个执行
      * AlwaysAdmit
      * AlwaysDeny
      * ServiceAccount
      * DenyEscolatingExec

# Kubernetes - 入门实践

## Deployment 实践

* ##### 首先配置好 Deployment 的配置文件（这里用的是 tomcat 镜像）

  ```yaml
  # app.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web
  spec:
    selector:
      matchLabels:
        app: web
    replicas: 2
    template:
      metadata:
        labels:
          app: web
      spec:
        containers:
        - name: web
          image: registry.cn-hangzhou.aliyuncs.com/liuyi01/tomcat:8.0.51-alpine
          ports:
          - containerPort: 8080
  ```

* ##### 通过 kubectl 命令创建服务

  ```shell
  # 创建应用
  $ kubectl create -f app.yaml
  Deployment.apps/web created
  
  # 等待一会后，查看 Pod 调度、运行情况。
  # 我看可以看到 Pod 的名字、运行状态、Pod 的 ip、还有所在Node的名字等信息
  $ kubectl get Pods -o wide
  NAME                       READY     STATUS    RESTARTS   AGE   IP       NODE
  web-c486dd5c4-86fxm        1/1       Running   0          1m        172.24.3.13     node-01
  web-c486dd5c4-zxdbb        1/1       Running   0          1m        172.24.0.149    node-02
  ```

## Service 实践

* 通过上面创建的 Deployment 还没法合理的访问到应用，下面创建一个 service 作为访问应用的入口

* ##### 首先创建service配置

  ```yaml
  # service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: web
  spec:
    ports:
    - port: 80 # 服务端口
      protocol: TCP
      targetPort: 8080 # 容器端口
    selector:
      app: web # 标签选择器，这里的app=web正是我们刚才建立app
  ```

* ##### 创建服务

  ```shell
  # 创建
  $ kubectl create -f service.yaml
  service/web created
  
  # 查看
  $ kubectl get service
  NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
  web      ClusterIP   10.95.189.143   <none>        80/TCP    9s
  ```

* ##### 访问服务

  * 接下来就可以在任意节点通过ClusterIP负载均衡的访问后端应用了

  ```shell
  # 在任意 Node 上访问tomcat服务
  $ curl -I 10.95.189.143
  HTTP/1.1 200 OK
  Server: Apache-Coyote/1.1
  Content-Type: text/html;charset=UTF-8
  Transfer-Encoding: chunked
  ```

# 高可用集群搭建

```shell
# woody/123456
# root/123456

# 运行docker
systemctl start docker

# iterms多个窗口同时操作
command + shift + i

# 切换到root用户
sudo -i 
# m1(9主)、m2(10主)、s1(12从)
	# 没设置docker启动参数（可选）
# 查看日志
	journalctl -f
# linux上安装git
	yum install git-core
# mooc git
	git clone https://git.imooc.com/coding-335/kubernetes-ha-kubeadm.git
	woodyfine/s
# 查看网卡
	ip a
	#2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
	
systemctl status kubelet	
# 查看日志、查看日志、查看日志
journalctl -xefu kubelet
	
# 给文件夹设置权限
chmod 777 文件夹

# Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)
# 或者
# all available gssapi mechanisms failed错误解决
vi /etc/ssh/sshd_config
将PasswordAuthentication 的属性 no 改为 yes
service sshd restart


# global-config.properties中MASTER_VIP要改成和虚拟一IP段一致，如10.211.55.88

# 启动
systemctl start docker
systemctl start kubelet
service keepalived start	
kubectl apply -f /etc/kubernetes/addons/calico-rbac-kdd.yaml
kubectl apply -f /etc/kubernetes/addons/calico.yaml
kubectl get pods -n kube-system

# 进度
m1加入其他master节点有问题（m2加入到m1）
m2、m3的环境都准备好了

# 加入其他master节点
m2中加入m1


# kubeadm 报错 error execution phase preflight: couldn’t validate the identity of the API Server: abort connecting to API servers after timeout of 5m0s
# 在master重新生成token
kubeadm token create
# master新token uhme9v.rbqdsv9389irhim1
kubeadm join 192.168.55.188:6443 --token uhme9v.rbqdsv9389irhim1 \
    --discovery-token-ca-cert-hash sha256:a3da716d3f81182541ad3028dd01927f81546868b755c5b5648366992bbf39fc \
    --experimental-control-plane --certificate-key 0964c2ad3e27f40f011b09cb7243d25edad8a0aaf35cdeca2d650a0af9c2869e    
    
    
# [WARNING IsDockerSystemdCheck]: detected “cgroupfs” as the Docker cgroup driver. The recommended driver is “systemd”. Please follow the guide at https://kubernetes.io/docs/setup/cri/
cat <<EOF > /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
service docker restart
	
# 高可用集群部署
cd kubernetes-ha-kubeadm/
# 分发配置文件(都在m1上操作)
# 拷贝到主
scp target/configs/keepalived-master.conf root@192.168.55.121:/etc/keepalived/keepalived.conf
# 拷贝到备
scp target/configs/keepalived-backup.conf root@192.168.55.122:/etc/keepalived/keepalived.conf
# 分发监测脚本
scp target/scripts/check-apiserver.sh root@192.168.55.121:/etc/keepalived/
scp target/scripts/check-apiserver.sh root@192.168.55.122:/etc/keepalived/

[root@m1 kubernetes-ha-kubeadm]# find target/|grep kubeadm-config.yaml
[root@m1 kubernetes-ha-kubeadm]# cp target/configs/kubeadm-config.yaml ~
#m1
kubeadm init --config=kubeadm-config.yaml --experimental-upload-certs


# 测试集群安装好没
[root@m2 ~]# curl -k  https://localhost:6443/healthz

#m1
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.55.188:6443 --token wlvir2.lny4w52jj67qoaf2 \
    --discovery-token-ca-cert-hash sha256:78e434d7cfe3d3cd0df2eef3b51877b8b5949afc71d9ec35045ebb8ddc42b438 \
    --experimental-control-plane --certificate-key 7791d8ad77ca642295b03cc52b0bf19a59a49cfc215d44819aa6e3146ae32201

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --experimental-upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.55.188:6443 --token wlvir2.lny4w52jj67qoaf2 \
    --discovery-token-ca-cert-hash sha256:78e434d7cfe3d3cd0df2eef3b51877b8b5949afc71d9ec35045ebb8ddc42b438
    
    
[root@m1 ~]# mkdir .kube
[root@m1 ~]# cp -i /etc/kubernetes/admin.conf ~/.kube/config
[root@m1 ~]# vi .kube/config
[root@m1 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                         READY   STATUS    RESTARTS   AGE
kube-system   coredns-8567978547-7d9wq     0/1     Pending   0          2m59s
kube-system   coredns-8567978547-hf6sg     0/1     Pending   0          2m59s
kube-system   etcd-m1                      1/1     Running   0          2m29s
kube-system   kube-apiserver-m1            1/1     Running   0          2m21s
kube-system   kube-controller-manager-m1   1/1     Running   0          2m20s
kube-system   kube-proxy-l9wk8             1/1     Running   0          2m59s
kube-system   kube-scheduler-m1            1/1     Running   0          2m8s
[root@m1 ~]# curl -k https://localhost:6443/healthz
ok[root@m1 ~]#


[root@m1 addons]# cp calico* /etc/kubernetes/addons/

vagrant plugin install --plugin-version 2.0.1 vagrant-proxyconf

kubectl describe pods -n kube-system calico-typha-666749994b-klvjs
kubectl describe pods -n kube-system coredns-8567978547-7d9wq

```

![image-20191114161705667](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191114161705667.png)

## kubeadm方式

* 集群落地方案1
* 优雅：几乎所有的组件都运行在容器中，运行在k8s的POD中
* 简单：配置文件+命令
* 支持高可用
* 升级方便
* 不易维护、文档不够细致

## 二进制方式

* 集群落地方案2

* 易于维护：通过进程去运行，配置然后手动运行
* 灵活：单机、高可用都是一个一个模块运行起来的
* 升级方便
* 没有文档、安装复杂

* [安装教程](https://git.imooc.com/coding-335/kubernetes-ha-kubeadm)

  * 实践准备环境

    ![image-20191110165149862](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110165149862.png)

  * 高可用集群部署

  * 集群可用性测试

  * 部署dashboard

# 业务系统迁移kubernetes

## Harbor

* 云原生镜像仓库
* 基于角色的访问控制
* 基于策略的镜像复制
* 漏洞扫描

![image-20191111170030665](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191111170030665.png)

* 高可用方案

  * 双主复制

    * 两个woker节点上安装Harbor，master节点上安装nginx

    ![image-20191111173643558](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191111173643558.png)
    
    ```shell
    # slave
    # 1. 下载解压 harbor-offline-installer-v1.9.4.tgz
    # 2. 修改配置文件harbor.yml
    hostname: 192.168.0.162        # 本机ip
    # 3. 安装docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    # 国内镜像
    curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose -version
    # 4. 安装
    sh install.sh
    # 启动 docker-compose start
    # http://192.168.0.162
    ```
    
    ```shell
    # master
    # 1. 安装nginx
    docker pull nginx:1.13.12
    # 2. 配置文件
    # nginx.conf
    user nginx;
    worker_processes 1;
    
    error_log /var/log/nginx/error.log warn;
    
    pid /var/run/nginx.pid;
    
    events {
    	worker_connections 1024;
    }
    
    stream {
    	upstream hub {
    		server 192.168.0.242:80;          # worker ip
    	}
    	server {
    		listen 80;
    		proxy_pass hub;
    		proxy_timeout 300s;
    		proxy_connect_timeout 5s;
    	}
    }
    
    # restart.sh
    #!/bin/bash
    
    docker stop harbornginx
    
    docker rm harbornginx
    
    docker run -itd --net=host --name=harbornginx -v /software/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.13.12
    # 3. 启动nginx
    sh restart.sh
    docker logs 0a9f1
    
    # 访问
    http://masterIP
    ```
    
    ```shell
    # 登录harbor   （创建新用户harbor/Harbor12345）
    # 1. 创建公共项目 kubernetes（添加成员harbor）
    # 宿主主机上配置hosts
    192.168.0.161 hub.imooc.com
    # mster/slave上配置hosts
    192.168.0.161 hub.imooc.com
    
    # 2. master上
    docker images|grep nginx
    docker tag nginx:1.13.12 hub.imooc.com/kubernetes/nginx:1.13.12
    vi /etc/docker/daemon.json          # master/slave都要添加
    {
    	"insecure-registries":["hub.imooc.com"]
    }
    systemctl restart docker
    # 启动nginx
    sh restart.sh
    # 登录
    docker login hub.imooc.com
    harbor
    Harbor12345
    # push镜像
    docker push hub.imooc.com/kubernetes/nginx:1.13.12
    # slave上pull
    docker pull hub.imooc.com/kubernetes/nginx:1.13.12
    # Harbor上配置同步
    # 可在多个服务器上同步镜像
    ```

### kubernetes的服务发现

* 服务内部互相访问

  ![image-20200207120909216](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200207120909216.png)

  * 方式一：通过DNS + ClusterIP
  * 方式二：通过HeadlessService返回Pod实例供调用者选择

* 服务内访问服务外

  ​	![image-20200207121649673](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200207121649673.png)

  * 方式一：通过IP + Port直接访问外部服务
  * 方式二：DNS + Service + Endpoint（配置外部服务）

* 服务外访问服务内

  ![image-20200207122450330](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200207122450330.png)

  * 方式一：NodePort，所有实例上都会开8080端口
  * 方式二：hostport，只在当前实例上开8080端口
  * 方式三：Ingress

### Ingress

```shell
# nginx-ingress
# 只需一台：slave
https://kubernetes.github.io/ingress-nginx/deploy/
# mkdir /software/nginx-ingress
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.28.0/deploy/static/mandatory.yaml
kubectl apply -f mandatory.yaml
# 查看需要哪些镜像
grep image mandatory.yaml
# slave 上单独拉去镜像
docker pull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.28.0
# 如果官方拉取慢，则找国内镜像源
docker pull quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.28.0
# 然后打tag
docker tag quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.28.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.28.0
# slave上拉取镜像好了后，master上也好了
kubectl get all -n ingress-nginx

kubectl describe pod nginx-ingress-controller-67c7566b6-22mcl -n ingress-nginx
```

```shell
docker pull gcr.azk8s.cn/google_containers/kubedns-amd64:1.5
docker tag gcr.azk8s.cn/google_containers/kubedns-amd64:1.5 k8s.gcr.io/defaultbackend-amd64:1.5

docker pull quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
docker tag quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
```



nodeport

![image-20200207151114258](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200207151114258.png)

```shell
[root@slave nginx-ingress]# netstat -tpnl|grep 80
tcp6       0      0 :::80                   :::*                    LISTEN      22332/docker-proxy
# 停掉一个slave上的harbor（因为其占用了80端口）
# nginx使用的是161，所以停掉222的harbor
netstat -tpnl|grep 80
netstat -tpnl|grep 443 # 都没占用
# master上
kubectl get nodes
kubectl label node slave3 app=ingress   # 给222打个标签
```

![image-20200207205825166](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200207205825166.png)

![image-20200207205956429](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200207205956429.png)

```shell
kubectl apply -f mandatory.yaml
kubectl get all -n ingress-nginx
# 222上查看80和443端口已起来
netstat -tpnl|grep 80
netstat -tpnl|grep 443
```

```yaml
# master上 /software/ingress-demo/ingress-demo.yaml

#deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-demo
spec:
  selector:
    matchLabels:
      app: tomcat-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat-demo
    spec:
      containers:
      - name: tomcat-demo
        image: registry.cn-hangzhou.aliyuncs.com/liuyi01/tomcat:8.0.51-alpine
        ports:
        - containerPort: 8080
---
#service
apiVersion: v1
kind: Service
metadata:
  name: tomcat-demo
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: tomcat-demo

---
#ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tomcat-demo
spec:
  rules:
  - host: tomcat.mooc.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-demo
          servicePort: 80          
```

```shell
kubectl create -f ingress-demo.yaml
kubectl get pod -o wide 
# 宿主主机上绑定hosts  因为ingress在222上启动
192.168.0.222 tomcat.mooc.com
192.168.0.222 api.mooc.com
# 访问
http://api.mooc.com/          # 404
tomcat.mooc.com
# 查看日志
kubectl logs tomcat-demo-6bc7d5b6f4-bbzb8
journalctl -f
```

```shell
kubectl get svc
kubectl get pods -o wide
kubectl describe svc tomcat-services
```

## 定时任务迁移Kubernetes

![image-20200209214556030](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200209214556030.png)

![image-20200209213527620](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200209213527620.png)

```shell
docker pull openjdk:8-jre-alpine
docker tag openjdk:8-jre-alpine hub.imooc.com/kubernetes/openjdk:8-jre-alpine
# 安装maven、git
yum install gits
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
	<mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
	</mirror>
# 拉取项目 https://gitee.com/pa
git clone https://git.imooc.com/coding-335/mooc-k8s-demo.git
cd cronjob-demo/
mvn package
cd target
java -cp cronjob-demo-1.0-SNAPSHOT.jar com.mooc.demo.cronjob.Main
# 构建基础镜像 src 同级目录
vi Dockerfile
  FROM hub.imooc.com/kubernetes/openjdk:8-jre-alpine
  COPY target/cronjob-demo-1.0-SNAPSHOT.jar /cronjob-demo.jar
  ENTRYPOINT ["java","-cp","/cronjob-demo.jar","com.mooc.demo.cronjob.Main"]
# 构建镜像  
docker build -t cronjob:v1 .
# 运行镜像
docker run -it cronjob:v1
docker tag cronjob:v1 hub.imooc.com/kubernetes/cronjob:v1
docker push hub.imooc.com/kubernetes/cronjob:v1
# 制作k8s服务
# cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-demo
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 3         # 保留最近3个运行成功的pod
  suspend: false                        # false则立即按照schedule去调度
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: cronjob-demo
        spec:
          restartPolicy: Never
          containers:
          - name: cronjob-demo
            image: hub.imooc.com/kubernetes/cronjob:v1
# 运行
kubectl apply -f cronjob.yaml
kubectl get cronjob
kubectl get pods -o wide
# 在slave4上运行，在slave4上查看日志
docker ps|grep cronjob
docker ps -a|grep cronjob
docker logs cd543ff088d6
```

## SpringBoot的web服务迁移Kubernetes

```shell
# springboot-web-demo
mvn package
cd target
tar -tf springboot-web-demo-1.0-SNAPSHOT.jar
java -jar springboot-web-demo-1.0-SNAPSHOT.jar
192.168.0.161:8080/hello?name=woody
# 构建基础镜像 src 同级目录
vi Dockerfile
  FROM hub.imooc.com/kubernetes/openjdk:8-jre-alpine
  COPY target/springboot-web-demo-1.0-SNAPSHOT.jar /springboot-web.jar
  ENTRYPOINT ["java","-jar","/springboot-web.jar"]
# 构建镜像
docker build -t springboot-web:v1 .
docker run -itd springboot-web:v1
docker tag springboot-web:v1 hub.imooc.com/kubernetes/springboot-web:v1
docker push hub.imooc.com/kubernetes/springboot-web:v1
# k8s服务
# springboot-web.yaml
#deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-web-demo
spec:
  selector:
    matchLabels:
      app: springboot-web-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: springboot-web-demo
    spec:
    	hostNetwork: true                   # 
      nodeSelector:                       #
        name: ingress                     # 指定运行安装有ingress的节点，否则504
      containers:
      - name: springboot-web-demo
        image: hub.imooc.com/kubernetes/springboot-web:v1
        ports:
        - containerPort: 8080
---
#service
apiVersion: v1
kind: Service
metadata:
  name: springboot-web-demo
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: springboot-web-demo
  type: ClusterIP

---
#ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: springboot-web-demo
spec:
  rules:
  - host: springboot.mooc.com
    http:
      paths:
      - path: /
        backend:
          serviceName: springboot-web-demo
          servicePort: 80

# 创建服务
kubectl apply -f springboot-web.yaml
kubectl get pods
# 宿主主机host配置 ip 为 安装ingress的主机ip
192.168.0.163 springboot.mooc.com
```

## 传统Dubbo服务迁移Kubernetes

## 传统web服务迁移Kubernetes

```shell
docker pull tomcat:8.0.51-alpine
docker tag tomcat:8.0.51-alpine hub.imooc.com/kubernetes/tomcat:8.0.51-alpine
docker push hub.imooc.com/kubernetes/tomcat:8.0.51-alpine
# dubbo-demo-api
mvn install
# web-demo
mvn package
cd target
jar -tf web-demo-1.0-SNAPSHOT.war
mkdir ROOT
mv web-demo-1.0-SNAPSHOT.war ./ROOT/
cd ROOT/
jar -xvf web-demo-1.0-SNAPSHOT.war
rm web-demo-1.0-SNAPSHOT.war
# 构建镜像
docker images|grep tomcat
docker run -it --entrypoint bash hub.imooc.com/kubernetes/tomcat:8.0.51-alpine
# /usr/local/tomcat/webapps

# src同级目录 start.sh
#!/bin/bash
sh /usr/local/tomcat/bin/startup.sh
tail -f /usr/local/tomcat/logs/catalina.out
# 添加可执行权限
chmod +x start.sh

# Dockerfile
FROM hub.imooc.com/kubernetes/tomcat:8.0.51-alpine
COPY target/ROOT /usr/local/tomcat/webapps/ROOT
COPY start.sh /usr/local/tomcat/bin/start.sh
ENTRYPOINT ["sh","/usr/local/tomcat/bin/start.sh"]
# 构建
docker build -t web-demo:v1 .
docker run -it web-demo:v1
# 报错：版本不一致
# 修改pom.xml，添加
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <encoding>UTF-8</encoding>
    <source>1.7</source>
    <target>1.7</target>
  </configuration>
</plugin>
# 重新
mvn clean package
cd target/
mv web-demo-1.0-SNAPSHOT.war ROOT/
cd ROOT/
rm -rf META-INF/ WEB-INF/
jar -xvf web-demo-1.0-SNAPSHOT.war
rm -rf web-demo-1.0-SNAPSHOT.war
# build
docker build -t web-demo:v1 .
docker run -it web-demo:v1
docker tag web-demo:v1 hub.imooc.com/kubernetes/web-demo:v1
docker push  hub.imooc.com/kubernetes/web-demo:v1
# k8s服务
# web.yaml
#deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-demo
spec:
  selector:
    matchLabels:
      app: web-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: web-demo
    spec:
      hostNetwork: true
      nodeSelector:
        name: ingress
      containers:
      - name: web-demo
        image: hub.imooc.com/kubernetes/web-demo:v1
        ports:
        - containerPort: 8080
---
#service
apiVersion: v1
kind: Service
metadata:
  name: web-demo
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: web-demo
  type: ClusterIP

---
#ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-demo
spec:
  rules:
  - host: web.mooc.com
    http:
      paths:
      - path: /
        backend:
          serviceName: web-demo
          servicePort: 80
# 创建
kubectl apply -f web.yaml
# 删除pod
kubectl delete -f web.yaml
kubectl get pods -o wide
kubectl describe pod web-demo-5694fbd879-4b22k
# 宿主注解host
192.168.0.163 web.mooc.com
# 访问
web.mooc.com
# Warning  FailedScheduling  34s (x3 over 35s)  default-scheduler  0/3 nodes are available: 1 node(s) didn't have free ports for the requested pod ports, 2 node(s) didn't match node selector.
```

# CICD实践

 ![image-20200210171108010](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200210171108010.png)

```shell
sudo wget -O /etc/yum.repos.d/jenkins.repo http://jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins

# git java maven
# jenkins  192.168.0.163(slave)
https://mirrors.tuna.tsinghua.edu.cn/jenkins/
https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.204.2/jenkins.war
nohup java -jar jenkins.war --httpPort=8080 &
# master nohup java -jar jenkins.war --httpPort=8080 &

# 设置jenkins镜像加速器
# 1. find -name default.json
#    ./root/.jenkins/updates/default.json
# 2. 进入./root/.jenkins/updates
# 3. sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

tail -f nohup.out
84c34090b3764a79aabf25eac4166d82
# 
# 访问
http://192.168.0.161:8080/
jenkins/Jenkins12345
# Jenkins——凭据-系统中添加Add Credentials，可添加git账户
# 1. 创建一个项目 k8s-web-demo - 流水线
# 2. Pipeline配置
```

![image-20200212225429778](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200212225429778.png)





```shell
node {
	env.BUILD_DIR="/root/build-workspace"
	env.MODULE="web-demo"
	env.HOST="k8s-web.mooc.com"
	stage('Preparation') {
		git 'https://gitee.com/pa/mooc-k8s-demo-docker.git'
	}
	stage('Maven Build') {
		sh "mvn -pl ${MODULE} -am clean package"
	}
	stage('Build Image') {
		sh "/root/script/build-image-web.sh"
	}
	stage('Deploy') {
		sh "/root/script/deploy.sh"
	}
}
```

```shell
# /root/script/build-image-web.sh
# 添加执行权限
chmod +x /root/script/build-image-web.sh
# 需要先登录Harbor
#!/bin/bash
if ["${BUILD_DIR}" == ""];then
	echo "env 'BUILD_DIR' is not set"
	exit 1
fi

DOCKER_DIR=${BUILD_DIR}/${JOB_NAME}

if [ ! -d ${DOCKER_DIR} ];then
	mkdir -p ${DOCKER_DIR}
fi

echo "docker workspace: ${DOCKER_DIR}"

JENKINS_DIR=${WORKSPACE}/${MODULE}

echo "jenkins workspace: ${JENKINS_DIR}"

if [ ! -f ${JENKINS_DIR}/target/*.war ];then
	echo "target war file not found ${JENKINS_DIR}/target/*.war"
	exit 1
fi

cd ${DOCKER_DIR}
rm -rf *
unzip -oq ${JENKINS_DIR}/target/*.war -d ./ROOT
mv ${JENKINS_DIR}/Dockerfile .
if [ -d ${JENKINS_DIR}/dockerfiles ]; then
	mv ${JENKINS_DIR}/dockerfiles .
fi

# VERSION=$(date + %Y%m%d%H%M%S)
VERSION=`date '+%Y%m%d%H%M%S'`
IMAGE_NAME=hub.imooc.com/kubernetes/${JOB_NAME}:${VERSION}

# 创建变量，共deploy时使用
echo "${IMAGE_NAME}" > ${WORKSPACE}/IMAGE

echo "building image: ${IMAGE_NAME}"
docker build -t ${IMAGE_NAME} .

docker push ${IMAGE_NAME}
```

```dockerfile
# Dockerfile
FROM hub.imooc.com/kubernetes/tomcat:8.0.51-alpine

COPY ROOT /usr/local/tomcat/webapps/ROOT

COPY dockerfiles/start.sh /usr/local/tomcat/bin/start.sh

ENTRYPOINT ["sh" , "/usr/local/tomcat/bin/start.sh"]
```

```shell
# 模版  /root/script/template/web.yaml
#deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{name}}
spec:
  selector:
    matchLabels:
      app: {{name}}
  replicas: 1
  template:
    metadata:
      labels:
        app: {{name}}
    spec:
      hostNetwork: true
      nodeSelector:
        name: ingress
      containers:
      - name: {{name}}
        image: {{image}}
        ports:
        - containerPort: 8080
---
#service
apiVersion: v1
kind: Service
metadata:
  name: {{name}}
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: {{name}}
  type: ClusterIP

---
#ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: {{name}}
spec:
  rules:
  - host: {{host}}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{name}}
          servicePort: 80
```

```shell
# /root/script/deploy.sh
chmod +x deploy.sh
#!/bin/bash

name=${JOB_NAME}
image=$(cat ${WORKSPACE}/IMAGE)
host=${HOST}

echo "deploying ... name: ${name}, image: ${image}, host: ${host}"

rm -f web.yaml
cp $(dirname "${BASH_SOURCE[0]}")/template/web.yaml .
echo "copy success"
sed -i "s,{{name}},${name},g" web.yaml
sed -i "s,{{image}},${image},g" web.yaml
sed -i "s,{{host}},${host},g" web.yaml
echo "ready to apply"

# kubectl delete -f web.yaml
# sleep 5

kubectl apply -f web.yaml
echo "apply success"
cat web.yaml

# 健康检查
success=0
count=60
IFS=","
sleep 5
while [ ${count} -gt 0 ]
do
    replicas=$(kubectl get deploy ${name} -o go-template='{{.status.replicas}},{{.status.updatedReplicas}},{{.status.readyReplicas}},{{.status.availableReplicas}}')
    echo "replicas: ${replicas}"
    arr=(${replicas})
    if [ "${arr[0]}" == "${arr[1]}" -a "${arr[1]}" == "${arr[2]}" -a "${arr[2]}" == "${arr[3]]}" ];then
        echo "health check success!"
        success=1
        break
    fi
    ((count--))
    sleep 2
done

if [ ${success} -ne 1 ];then
    echo "health check failed"
    exit 1
fi
```

```shell
# vim 替换命令
:%s/web-demo/{{name}}/g
```

```shell
kubectl get deploy
# 查看版本是否一致：jenkins构建的版本和部署的版本
kubectl get deploy k8s-web-demo -o yaml
# 健康检查  值都一样则健康 1,1,1,1
kubectl get deploy k8s-web-demo -o go-template='{{.status.replicas}},{{.status.updatedReplicas}},{{.status.readyReplicas}},{{.status.availableReplicas}}'
```

# 深入kubernete

## Namespace -- 集群的共享与隔离

* 隔离
  * 资源对象的隔离
    * Service、Deployment、Pod
  * 资源配额的隔离
    * CPU、Memory

* 划分方式
  * 按环境划分：dev、test
  * 按团队划分
  * 自定义多级划分

```shell
kubectl get ns
kubectl get namespaces
kubectl get pods -n default
kubectl get all -n default
# 创建namespace
# namespace-dev.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
# kubectl create -f namespace-dev.yaml  
# 指定pod的namespace
# metadata:下添加 namespace: dev
```

```shell
kubectl exec -it k8s-web-demo-566b698769-m6kbf bash -n default
bash-4.4# cat /etc/resolv.conf

kubectl get svc
```

* 不同namespace下的service IP可以互相访问、Pod IP也可互相访问
* namespace是对名字的隔离，不是物理隔离

```shell
# 配置 namespace 权限，只能看到某个namespace下的资源
cp .kube/config .kube/config.backup

kubectl config set-context ctx-dev \
--cluster=kubernetes \
--user=admin \
--namespace=dev \
--kubeconfig=/root/.kube/config

kubectl config use-context ctx-dev --kubeconfig=/root/.kube/config


# 原来
kubectl config set-context kubernetes \
--cluster=kubernetes \
--user=admin \
--kubeconfig=kube/config

kubectl config use-context kubernetes --kubeconfig=kube/config
```

## Resources

* CPU、内存、GPU、持久化存储

  ![image-20200213183043771](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200213183043771.png)

* 核心设计

  * Requests
  * Limits

  ```shell
  # spec的containers
  resources:
    requests:
      memory: 100Mi
      cou: 100m
    limits:
    	memory: 100Mi
    	cpu: 200m
  ```

  

## 服务调度与编排

## 落地实践深入

## 日志和监控

# ServiceMesh代表作istio





