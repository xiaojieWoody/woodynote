# 内容

* 快速入门
  * 核心概念
  * 架构设计
  * 认证授权

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
```

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



## 准备工作【为平稳迁移做好储备】

## 最佳实践【多类型业务迁移落地】

# CICD实践

# 深入kubernete

## 几个重要的资源对象

## 服务调度与编排

## 落地实践深入

## 日志和监控

# ServiceMesh代表作istio




