# 快速入门

* 核心概念
* 架构设计
* 认证授权

# 高可用集群搭建

* 二进制或kubeadm
  * 网络插件：calico
  * dns：coredns
  * 看板：dashboard

#业务系统迁移Kubernetes

*  准备工作
  * Harbor：架构、原理、部署高可用Harbor仓库
  * 服务发现
  * IngressNginx：外部服务发现方案
* 最佳实践
  * Docker化
  * 跑在k8s
  * 服务发现

# CICD实践

![image-20191110090131731](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191110090131731.png)

# 深入k8s

## 几个重要的资源对象

* namespace
  * 对资源对象和资源配额的多层面隔离机制，pod资源限制了各种配置方式，pod的k8s，以及它跟资源配额的关系，和pod和它节点资源准确时候的驱逐机制
* resources
* label：作用于不同资源对象上的不同作用
* 合理的规划命名空间，通过资源配额来提供服务的稳定性，可以设置驱逐策略来提升系统的稳定性，以及灵活的运用label来为各种资源打标签

## 服务调度与编排

* 健康检查
  * pod的健康检查如何工作、参数该如何配置及优化
* 调度策略
  * 预选策略和优选策略
* 部署策略
* 深入pod
  * 思想、生命周期、设计

## 落地与实践

* Ingress-Nginx
  * ab测试、蓝绿部署、小能量测试
* PV / PVC / StorageClass
  * 共享存储
* StatefulSet
* Kubernetes API

## 日志与监控

* 日志主流方案
* 从日志采集到日志展示
* Prometheus

# Istio

* 架构设计-环境部署-数据展现

# 环境参数

* Kubernetes 1.14.0
* Docker 17.03.x
* Java 1.8
* Harbor 1.6.0
* Prometheus 2.8.1
* Istio 1.1.2

# 必知必会

* k8s是什么？

  * 开源、自动化部署扩缩容、管理容器化应用

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

# 高可用集群搭建

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

## 准备工作【为平稳迁移做好储备】

## 最佳实践【多类型业务迁移落地】

# CICD实践

# 深入kubernete

## 几个重要的资源对象

## 服务调度与编排

## 落地实践深入

## 日志和监控

# ServiceMesh代表作istio





