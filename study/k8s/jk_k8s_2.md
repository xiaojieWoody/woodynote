# Kubeadm

*  **Kubernetes 项目简单的部署方法**

  * 独立的部署工具，名叫：[kubeadm](https://github.com/kubernetes/kubeadm)

    ```shell
    # 让用户能够通过这样两条指令完成一个 Kubernetes 集群的部署
    # 创建一个 Master 节点
    $ kubeadm init
     
    # 将一个 Node 节点加入到当前集群中
    $ kubeadm join <Master 节点的 IP 和端口 >
    ```

## kubeadm 的工作原理

* Kubernetes 的架构和它的组件，在部署时，它的每一个组件都是一个需要被执行的、单独的二进制文件

* **为什么不用容器部署 Kubernetes 呢？**

  * 只要给每个 Kubernetes 组件做一个容器镜像，然后在每台宿主机上用 docker run 指令启动这些组件容器，部署不就完成了吗？
  * 但是，**这样做会带来一个很麻烦的问题，即：如何容器化 kubelet**
  *  kubelet 是 Kubernetes 项目用来操作 Docker 等容器运行时的核心组件。可是，除了跟容器运行时打交道外，kubelet 在配置容器网络、管理容器数据卷时，都需要直接操作宿主机
  * 而如果现在 kubelet 本身就运行在一个容器里，那么直接操作宿主机就会变得很麻烦。对于网络配置来说还好，kubelet 容器可以通过不开启 Network Namespace（即 Docker 的 host network 模式）的方式，直接共享宿主机的网络栈。可是，要让 kubelet 隔着容器的 Mount Namespace 和文件系统，操作宿主机的文件系统，就有点儿困难了
    * kubelet 做的挂载操作，不能被“传播”到宿主机上
  * 到目前为止，在容器里运行 kubelet，依然没有很好的解决办法，也不推荐用容器去部署 Kubernetes 项目

* kubeadm 选择了一种妥协方案

  * 把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件

  * 使用 kubeadm 的第一步，是在机器上手动安装 kubeadm、kubelet 和 kubectl 这三个二进制文件

    ```shell
    # 只需执行
    $ apt-get install kubeadm
    # 接下来，就可以使用“kubeadm init”部署 Master 节点
    ```

## kubeadm init 的工作流程

* 执行 kubeadm init 指令后，**kubeadm 首先要做的，是一系列的检查工作，以确定这台机器可以用来部署 Kubernetes**，Preflight Checks

  * Linux 内核的版本必须是否是 3.10 以上？
  * Linux Cgroups 模块是否可用？
  * 用户安装的 kubeadm 和 kubelet 的版本是否匹配？
  * 机器上是不是已经安装了 Kubernetes 的二进制文件？
  * ....

* **通过了 Preflight Checks 之后**，**kubeadm生成 Kubernetes 对外提供服务所需的各种证书和对应的目录**

  * Kubernetes 对外提供服务时，除非专门开启“不安全模式”，否则都要通过 HTTPS 才能访问 kube-apiserver。这就需要为 Kubernetes 集群配置好证书文件
  * kubeadm 为 Kubernetes 项目生成的证书文件都放在 Master 节点的 /etc/kubernetes/pki 目录下。在这个目录下，最主要的证书文件是 ca.crt 和对应的私钥 ca.key
  * 此外，用户使用 kubectl 获取容器日志等 streaming 操作时，需要通过 kube-apiserver 向 kubelet 发起请求，这个连接也必须是安全的。kubeadm 为这一步生成的是 apiserver-kubelet-client.crt 文件，对应的私钥是 apiserver-kubelet-client.key
  * 除此之外，Kubernetes 集群中还有 Aggregate APIServer 等特性，也需要用到专门的证书
  * 可以选择不让 kubeadm 为你生成这些证书，而是拷贝现有的证书到如下证书的目录里，这时，kubeadm 就会跳过证书生成的步骤，把它完全交给用户处理。`/etc/kubernetes/pki/ca.{crt,key}`

* **证书生成后，kubeadm 接下来会为其他组件生成访问 kube-apiserver 所需的配置文件**。这些文件的路径是：/etc/kubernetes/xxx.conf：

  ```shell
  ls /etc/kubernetes/
  admin.conf  controller-manager.conf  kubelet.conf  scheduler.conf
  ```

  * 这些文件里面记录的是，当前这个 Master 节点的服务器地址、监听端口、证书目录等信息。这样，对应的客户端（比如 scheduler，kubelet 等），可以直接加载相应的文件，使用里面的信息与 kube-apiserver 建立安全连接

* **接下来，kubeadm 会为 Master 组件生成 Pod 配置文件**

  * Kubernetes 有三个 Master 组件 kube-apiserver、kube-controller-manager、kube-scheduler，而它们都会被使用 Pod 的方式部署起来

  * 在 Kubernetes 中，有一种特殊的容器启动方法叫做“Static Pod”。它允许你把要部署的 Pod 的 YAML 文件放在一个指定的目录里。这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们

  * kubelet 在 Kubernetes 项目中的地位非常高，在设计上它就是一个完全独立的组件，而其他 Master 组件，则更像是辅助性的系统容器

  * 在 kubeadm 中，Master 组件的 YAML 文件会被生成在 /etc/kubernetes/manifests 路径下。比如，kube-apiserver.yaml

  * 在这一步完成后，kubeadm 还会再生成一个 Etcd 的 Pod YAML 文件，用来通过同样的 Static Pod 的方式启动 Etcd。所以，最后 Master 组件的 Pod YAML 文件如下所示：

    ```shell
    $ ls /etc/kubernetes/manifests/
    etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
    ```

  * 而一旦这些 YAML 文件出现在被 kubelet 监视的 /etc/kubernetes/manifests 目录下，kubelet 就会自动创建这些 YAML 文件中定义的 Pod，即 Master 组件的容器

  * Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来

* **然后，kubeadm 就会为集群生成一个 bootstrap token**

  * 在后面，只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到这个集群当中
  * 这个 token 的值和使用方法会，会在 kubeadm init 结束后被打印出来

* **在 token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用**。这个 ConfigMap 的名字是 cluster-info

* **kubeadm init 的最后一步，就是安装默认插件**

  * Kubernetes 默认 kube-proxy 和 DNS 这两个插件是必须安装的。它们分别用来提供整个集群的服务发现和 DNS 功能。其实，这两个插件也只是两个容器镜像而已，所以 kubeadm 只要用 Kubernetes 客户端创建两个 Pod 就可以了

## kubeadm join 的工作流程

* kubeadm init 生成 bootstrap token 之后，就可以在任意一台安装了 kubelet 和 kubeadm 的机器上执行 kubeadm join 了
* 可是，为什么执行 kubeadm join 需要这样一个 token 呢?
  * 因为，任何一台机器想要成为 Kubernetes 集群中的一个节点，就必须在集群的 kube-apiserver 上注册。可是，要想跟 apiserver 打交道，这台机器就必须要获取到相应的证书文件（CA 文件）。可是，为了能够一键安装，就不能让用户去 Master 节点上手动拷贝这些文件
  * 所以，kubeadm 至少需要发起一次“不安全模式”的访问到 kube-apiserver，从而拿到保存在 ConfigMap 中的 cluster-info（它保存了 APIServer 的授权信息）。而 bootstrap token，扮演的就是这个过程中的安全验证的角色
  * 只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了
  * 接下来，只要在其他节点上重复这个指令就可以了

## 配置 kubeadm 的部署参数

* 指定 kube-apiserver 的启动参数

  * 推荐在使用 kubeadm init 部署 Master 节点时，使用下面这条指令

    ```shell
    kubeadm init --config kubeadm.yaml
    ```

  * 这时，就可以给 kubeadm 提供一个 YAML 文件（比如，kubeadm.yaml）

  * 通过制定这样一个部署参数配置文件，就可以很方便地在这个文件里填写各种自定义的部署参数了。比如，现在要指定 kube-apiserver 的参数，那么只要在这个文件里加上这样一段信息：

    ```shell
    ...
    apiServerExtraArgs:
      advertise-address: 192.168.0.103
      anonymous-auth: false
      enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
      audit-log-path: /home/johndoe/audit.log
    ```

  * 然后，kubeadm 就会使用上面这些信息替换 /etc/kubernetes/manifests/kube-apiserver.yaml 里的 command 字段里的参数

  * 还可以修改 kubelet 和 kube-proxy 的配置，==修改 Kubernetes 使用的基础镜像的 URL==（默认的`k8s.gcr.io/xxx`镜像 URL 在国内访问是有困难的），指定自己的证书文件，指定特殊的容器运行时等等

## 总结

* https://github.com/kubernetes-sigs/kubespray
* https://github.com/easzlab/kubeasz

* 其实国内同学们用kubeadm安装集群最大的拦路虎在于有几个镜像没法下载，我建议大家先手动把镜像pull 下来，从阿里的镜像源上，然后tag成安装所需的镜像名称，这样你发现安装过程会异常顺利
* kubeadm拉取镜像的url是可配置的

# 搭建一个完整的K8S集群

* 2 核 CPU、 7.5 GB 内存

## 安装 kubeadm 和 Docker

```shell
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
$ apt-get update
$ apt-get install -y docker.io kubeadm
```

## 部署 Kubernetes 的 Master 节点

* 可通过配置文件来开启一些实验性功能

* 编写了一个给 kubeadm 用的 YAML 文件（名叫：kubeadm.yaml）

  ```yml
  apiVersion: kubeadm.k8s.io/v1alpha1
  kind: MasterConfiguration
  controllerManagerExtraArgs:
    # 将来部署的 kube-controller-manager 能够使用自定义资源（Custom Metrics）进行自动水平扩展
    horizontal-pod-autoscaler-use-rest-clients: "true"
    horizontal-pod-autoscaler-sync-period: "10s"
    node-monitor-grace-period: "10s"
  apiServerExtraArgs:
    runtime-config: "api/all=true"
  kubernetesVersion: "stable-1.11"
  ```

* 只需要执行一句指令`kubeadm init --config kubeadm.yaml`，就可以完成 Kubernetes Master 的部署了，这个过程只需要几分钟。部署完成后，kubeadm 会生成一行指令

  ```shell
  kubeadm join 10.168.0.2:6443 --token 00bwbx.uvnaa2ewjflwu1ry --discovery-token-ca-cert-hash sha256:00eb62a2a6020f94132e3fe1ab721349bbcd3e9b94da9654cfe15f2985ebd711
  ```

  * 这个 kubeadm join 命令，就是用来给这个 Master 节点添加更多工作节点（Worker）的命令

* kubeadm 还会提示我们第一次使用 Kubernetes 集群所需要的配置命令

  ```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

  * 而需要这些配置命令的原因是：Kubernetes 集群默认需要加密方式访问。所以，这几条命令，就是将刚刚部署生成的 Kubernetes 集群的安全配置文件，保存到当前用户的.kube 目录下，kubectl 默认会使用这个目录下的授权信息访问 Kubernetes 集群
  * 如果不这么做的话，每次都需要通过 export KUBECONFIG 环境变量告诉 kubectl 这个安全配置文件的位置

* 可以使用 kubectl get 命令来查看当前唯一一个节点的状态了

  ```shell
  $ kubectl get nodes
  NAME      STATUS     ROLES     AGE       VERSION
  master    NotReady   master    1d        v1.11.1
  $ kubectl describe node master
  # NodeNotReady 的原因在于，尚未部署任何网络插件
  ```

* 还可以通过 kubectl 检查这个节点上各个系统 Pod 的状态，其中，kube-system 是 Kubernetes 项目预留的系统 Pod 的工作空间（Namepsace，注意它并不是 Linux Namespace，它只是 Kubernetes 划分不同工作空间的单位）

  ```shell
  $ kubectl get pods -n kube-system
  NAME               READY   STATUS   RESTARTS  AGE
  coredns-78fcdf6894-j9s52     0/1    Pending  0     1h
  coredns-78fcdf6894-jm4wf     0/1    Pending  0     1h
  etcd-master           1/1    Running  0     2s
  kube-apiserver-master      1/1    Running  0     1s
  kube-controller-manager-master  0/1    Pending  0     1s
  kube-proxy-xbd47         1/1    NodeLost  0     1h
  kube-scheduler-master      1/1    Running  0     1s
  ```

  * CoreDNS、kube-controller-manager 等依赖于网络的 Pod 都处于 Pending 状态，即调度失败。这当然是符合预期的：因为这个 Master 节点的网络尚未就绪

## 部署网络插件

* 在 Kubernetes 项目“一切皆容器”的设计理念指导下，部署网络插件非常简单，只需要执行一句 kubectl apply 指令，以 Weave 为例：`kubectl apply -f https://git.io/weave-kube-1.6`

* 部署完成后，可以通过 kubectl get 重新检查 Pod 的状态

  ```shell
  $ kubectl get pods -n kube-system
  NAME                             READY     STATUS    RESTARTS   AGE
  coredns-78fcdf6894-j9s52         1/1       Running   0          1d
  coredns-78fcdf6894-jm4wf         1/1       Running   0          1d
  etcd-master                      1/1       Running   0          9s
  kube-apiserver-master            1/1       Running   0          9s
  kube-controller-manager-master   1/1       Running   0          9s
  kube-proxy-xbd47                 1/1       Running   0          1d
  kube-scheduler-master            1/1       Running   0          9s
  weave-net-cmk27                  2/2       Running   0          19s
  ```

* Kubernetes 支持容器网络插件，使用的是一个名叫 CNI 的通用接口，它也是当前容器网络的事实标准，市面上的所有容器网络开源项目都可以通过 CNI 接入 Kubernetes，比如 Flannel、Calico、Canal、Romana 等等，它们的部署方式也都是类似的“一键部署”
* 至此，Kubernetes 的 Master 节点就部署完成了。如果只需要一个单节点的 Kubernetes，现在就可以使用了。不过，在默认情况下，Kubernetes 的 Master 节点是不能运行用户 Pod 的，所以还需要额外做一个小操作

## 部署 Kubernetes 的 Worker 节点

* Kubernetes 的 Worker 节点跟 Master 节点几乎是相同的，它们运行着的都是一个 kubelet 组件。唯一的区别在于，在 kubeadm init 的过程中，kubelet 启动后，Master 节点上还会自动运行 kube-apiserver、kube-scheduler、kube-controller-manger 这三个系统 Pod

* 所以，相比之下，部署 Worker 节点反而是最简单的，只需要两步即可完成

  * 第一步，在所有 Worker 节点上执行“安装 kubeadm 和 Docker”一节的所有步骤

  * 第二步，执行部署 Master 节点时生成的 kubeadm join 指令：

    ```shell
    kubeadm join 10.168.0.2:6443 --token 00bwbx.uvnaa2ewjflwu1ry --discovery-token-ca-cert-hash sha256:00eb62a2a6020f94132e3fe1ab721349bbcd3e9b94da9654cfe15f2985ebd711
    ```

## 通过 Taint/Toleration 调整 Master 执行 Pod 的策略

* 默认情况下 Master 节点是不允许运行用户 Pod 的。而 Kubernetes 做到这一点，依靠的是 Kubernetes 的 Taint/Toleration 机制

  * 原理非常简单：一旦某个节点被加上了一个 Taint，即被“打上了污点”，那么所有 Pod 就都不能在这个节点上运行，因为 Kubernetes 的 Pod 都有“洁癖”

  * 除非，有个别的 Pod 声明自己能“容忍”这个“污点”，即声明了 Toleration，它才可以在这个节点上运行

  * 其中，为节点打上“污点”（Taint）的命令是：

    ```shell
    kubectl taint nodes node1 foo=bar:NoSchedule
    ```

  * 这时，该 node1 节点上就会增加一个键值对格式的 Taint，即：foo=bar:NoSchedule。其中值里面的 NoSchedule，意味着这个 Taint 只会在调度新 Pod 时产生作用，而不会影响已经在 node1 上运行的 Pod，哪怕它们没有 Toleration

  * 那么 Pod 又如何声明 Toleration 呢？

    * 只要在 Pod 的.yaml 文件中的 spec 部分，加入 tolerations 字段即可：

      ```yml
      apiVersion: v1
      kind: Pod
      ...
      spec:
        tolerations:
        - key: "foo"
          operator: "Equal"
          value: "bar"
          effect: "NoSchedule"
      ```

    *  Toleration 的含义是，这个 Pod 能“容忍”所有键值对为 foo=bar 的 Taint（ operator: “Equal”，“等于”操作）

* 通过 kubectl describe 检查一下 Master 节点的 Taint 字段，就会有所发现：

  ```shell
  $ kubectl describe node master
  Name:               master
  Roles:              master
  # Master 节点默认被加上了node-role.kubernetes.io/master:NoSchedule这样一个“污点”，其中“键”是node-role.kubernetes.io/master，而没有提供“值”
  Taints:             node-role.kubernetes.io/master:NoSchedule
  ```

* Master 节点默认被加上了`node-role.kubernetes.io/master:NoSchedule`这样一个“污点”，其中“键”是`node-role.kubernetes.io/master`，而没有提供“值”

  ```yml
  apiVersion: v1
  kind: Pod
  ...
  spec:
    tolerations:
    - key: "foo"
      operator: "Exists"
      effect: "NoSchedule"
  ```

* 如果就是想要一个单节点的 Kubernetes，删除这个 Taint 才是正确的选择

  ```shell
  # 在“node-role.kubernetes.io/master”这个键后面加上了一个短横线“-”，这个格式就意味着移除所有以“node-role.kubernetes.io/master”为键的 Taint
  $ kubectl taint nodes --all node-role.kubernetes.io/master-
  ```

## 部署 Dashboard 可视化插件

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

* 部署完成之后，就可以查看 Dashboard 对应的 Pod 的状态了

  ```shell
  kubectl get pods -n kube-system
  kubernetes-dashboard-6948bdb78-f67xk   1/1       Running   0          1m
  ```

* 想从集群外访问这个 Dashboard 的话，就需要用到 Ingress

## 部署容器存储插件

* 可是，如果在某一台机器上启动的一个容器，显然无法看到其他机器上的容器在它们的数据卷里写入的文件。**这是容器最典型的特征之一：无状态**

* 而容器的持久化存储，就是用来保存容器存储状态的重要手段：

  * 存储插件会在容器里挂载一个基于网络或者其他机制的远程数据卷，使得在容器里创建的文件，实际上是保存在远程存储服务器上，或者以分布式的方式保存在多个节点上，而与当前宿主机没有任何绑定关系。这样，无论在其他哪个宿主机上启动新的容器，都可以请求挂载指定的持久化存储卷，从而访问到数据卷里保存的内容。**这就是“持久化”的含义**

* 由于 Kubernetes 本身的松耦合设计，绝大多数存储项目，比如 Ceph、GlusterFS、NFS 等，都可以为 Kubernetes 提供持久化存储能力。在这次的部署实战中，选择部署一个很重要的 Kubernetes 存储插件项目：Rook

* Rook 项目是一个基于 Ceph 的 Kubernetes 存储插件。不过，不同于对 Ceph 的简单封装，Rook 在自己的实现中加入了水平扩展、迁移、灾难备份、监控等大量的企业级功能，使得这个项目变成了一个完整的、生产级别可用的容器存储插件

* 得益于容器化技术，用两条指令，Rook 就可以把复杂的 Ceph 存储后端部署起来：

  ```shell
  $ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml
  $ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
  ```

* 在部署完成后，就可以看到 Rook 项目会将自己的 Pod 放置在由它自己管理的两个 Namespace 当中

  ```shell
  $ kubectl get pods -n rook-ceph-system
  NAME                                  READY     STATUS    RESTARTS   AGE
  rook-ceph-agent-7cv62                 1/1       Running   0          15s
  rook-ceph-operator-78d498c68c-7fj72   1/1       Running   0          44s
  rook-discover-2ctcv                   1/1       Running   0          15s
   
  $ kubectl get pods -n rook-ceph
  NAME                   READY     STATUS    RESTARTS   AGE
  rook-ceph-mon0-kxnzh   1/1       Running   0          13s
  rook-ceph-mon1-7dn2t   1/1       Running   0          2s
  ```

* 这样，一个基于 Rook 持久化存储集群就以容器的方式运行起来了，而接下来在 Kubernetes 项目上创建的所有 Pod 就能够通过 Persistent Volume（PV）和 Persistent Volume Claim（PVC）的方式，在容器里挂载由 Ceph 提供的数据卷了
* 而 Rook 项目，则会负责这些数据卷的生命周期管理、灾难备份等运维工作

## 总结

* https://www.datayang.com/article/45

* 在 Bare-metal 环境下使用 kubeadm 工具部署了一个完整的 Kubernetes 集群：这个集群有一个 Master 节点和多个 Worker 节点；使用 Weave 作为容器网络插件；使用 Rook 作为容器持久化存储插件；使用 Dashboard 插件提供了可视化的 Web 界面
* 这个集群的部署过程并不像传说中那么繁琐，这主要得益于：
  1. kubeadm 项目大大简化了部署 Kubernetes 的准备工作，尤其是配置文件、证书、二进制文件的准备和制作，以及集群版本管理等操作，都被 kubeadm 接管了
  2. Kubernetes 本身“一切皆容器”的设计思想，加上良好的可扩展机制，使得插件的部署非常简便
* 上述思想，也是开发和使用 Kubernetes 的重要指导思想，即：基于 Kubernetes 开展工作时，一定要优先考虑这两个问题：
  * 我的工作是不是可以容器化？
  * 我的工作是不是可以借助 Kubernetes API 和可扩展机制来完成？
* 而一旦这项工作能够基于 Kubernetes 实现容器化，就很有可能像上面的部署过程一样，大幅简化原本复杂的运维工作
* kubeadm最新版本已经为1.12，看到上面很多人遇到提示版本不对，重新安装低版本就好了
  * apt remove kubelet kubectl kubeadm
  * apt install kubelet=1.11.3-00
  * apt install kubectl=1.11.3-00
  * apt install kubeadm=1.11.3-00
* 对于GFW，不能直接给kubeadm上代理，而是要让docker daemon翻，正确姿势参见：
  https://stackoverflow.com/questions/26550360/docker-ubuntu-behind-proxy
* ubuntu 换成这个源
  deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main

# 第一个容器化应用

* 什么才是 Kubernetes 项目能“认识”的方式呢？

  * 编写YAML配置文件，即：把容器的定义、参数、配置，统统记录在一个 YAML 文件中，然后用这样一句指令把它运行起来：

    ```shell
    $ kubectl create -f 我的配置文件
    ```

  * 这么做最直接的好处是，你会有一个文件能记录下 Kubernetes 到底“run”了什么

    ```yml
    # nginx-deployment.yaml
    apiVersion: apps/v1
    # Deployment，是一个定义多副本应用（即多个副本 Pod）的对象
    # Deployment 还负责在 Pod 定义发生变化时，对每个副本进行滚动更新（Rolling Update）
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80
    ```

    * 像这样的一个 YAML 文件，对应到 Kubernetes 中，就是一个 API Object（API 对象）。当你为这个对象的各个字段填好值并提交给 Kubernetes 之后，Kubernetes 就会负责创建出这些对象所定义的容器或者其他类型的 API 资源
    * 这样的每一个 API 对象都有一个叫作 Metadata 的字段，这个字段就是 API 对象的“标识”，即元数据，它也是我们从 Kubernetes 里找到这个对象的主要依据。这其中最主要使用到的字段是 Labels
    * Labels 就是一组 key-value 格式的标签。而像 Deployment 这样的控制器对象，就可以通过这个 Labels 字段从 Kubernetes 中过滤出它所关心的被控制对象
    * 比如，在上面这个 YAML 文件中，Deployment 会把所有正在运行的、携带“app: nginx”标签的 Pod 识别为被管理的对象，并确保这些 Pod 的总数严格等于两个
    * 而这个过滤规则的定义，是在 Deployment 的“spec.selector.matchLabels”字段。一般称之为：Label Selector
    * 另外，在 Metadata 中，还有一个与 Labels 格式、层级完全相同的字段叫 Annotations，它专门用来携带 key-value 格式的内部信息。所谓内部信息，指的是对这些信息感兴趣的，是 Kubernetes 组件本身，而不是用户。所以大多数 Annotations，都是在 Kubernetes 运行过程中，被自动加在这个 API 对象上

* Pod 就是 Kubernetes 世界里的“应用”；而一个应用，可以由多个容器组成。

* 像这样使用一种 API 对象（Deployment）管理另一种 API 对象（Pod）的方法，在 Kubernetes 中，叫作“控制器”模式（controller pattern）

* 一个 Kubernetes 的 API 对象的定义，大多可以分为 Metadata 和 Spec 两个部分。前者存放的是这个对象的元数据，对所有 API 对象来说，这一部分的字段和格式基本上是一样的；而后者存放的，则是属于这个对象独有的定义，用来描述它所要表达的功能

* 把这个 YAML 文件“运行”起来

  ```shell
  # 使用 kubectl create 指令
  kubectl create -f nginx-deployment.yaml
  # 通过 kubectl get 命令检查这个 YAML 运行起来的状态
  # kubectl get 指令的作用，就是从 Kubernetes 里面获取（GET）指定的 API 对象
  # -l 参数，即获取所有匹配 app: nginx 标签的 Pod
  # 在命令行中，所有 key-value 格式的参数，都使用“=”而非“:”表示
  $ kubectl get pods -l app=nginx
  NAME                                READY     STATUS    RESTARTS   AGE
  nginx-deployment-67594d6bf6-9gdvr   1/1       Running   0          10m
  nginx-deployment-67594d6bf6-v6j7w   1/1       Running   0          10m
  # 使用 kubectl describe 命令，查看一个 API 对象的细节
  $ kubectl describe pod nginx-deployment-67594d6bf6-9gdvr
  Name:               nginx-deployment-67594d6bf6-9gdvr
  Namespace:          default
  Priority:           0
  PriorityClassName:  <none>
  Node:               node-1/10.168.0.3
  Start Time:         Thu, 16 Aug 2018 08:48:42 +0000
  Labels:             app=nginx
                      pod-template-hash=2315082692
  Annotations:        <none>
  Status:             Running
  IP:                 10.32.0.23
  Controlled By:      ReplicaSet/nginx-deployment-67594d6bf6
  ...
  # 在 Kubernetes 执行的过程中，对 API 对象的所有重要操作，都会被记录在这个对象的 Events 里，并且显示在 kubectl describe 指令返回的结果中
  Events:
  # 对于这个 Pod，可以看到它被创建之后，被调度器调度（Successfully assigned）到了 node-1，拉取了指定的镜像（pulling image），然后启动了 Pod 里定义的容器（Started container） 
  # 将来进行 Debug 的重要依据。如果有异常发生，一定要第一时间查看这些 Events，往往可以看到非常详细的错误信息ss
    Type     Reason                  Age                From               Message
   
    ----     ------                  ----               ----               -------
    
    Normal   Scheduled               1m                 default-scheduler  Successfully assigned default/nginx-deployment-67594d6bf6-9gdvr to node-1
    Normal   Pulling                 25s                kubelet, node-1    pulling image "nginx:1.7.9"
    Normal   Pulled                  17s                kubelet, node-1    Successfully pulled image "nginx:1.7.9"
    Normal   Created                 17s                kubelet, node-1    Created container
    Normal   Started                 17s                kubelet, node-1    Started container
  ```
```

* 镜像版本升级

  * 例如，对这个 Nginx 服务进行升级，把它的镜像版本从 1.7.9 升级为 1.8

  * 只要修改这个 YAML 文件即可，可是，这个修改目前只发生在本地

    ```yml
    ...    
        spec:
          containers:
          - name: nginx
            image: nginx:1.8 # 这里被从 1.7.9 修改为 1.8
            ports:
          - containerPort: 80
```

  * 使用 kubectl replace 指令让这个更新在 Kubernetes 里也生效

    ```shell
    kubectl replace -f nginx-deployment.yaml
    ```

  * 不过，推荐使用 kubectl apply 命令，来统一进行 Kubernetes 对象的创建和更新操作

    * 是 Kubernetes“声明式 API”所推荐的使用方法

    * 作为用户，不必关心当前的操作是创建，还是更新，执行的命令始终是 kubectl apply，而 Kubernetes 则会根据 YAML 文件的内容变化，自动进行具体的处理

      ```yaml
      $ kubectl apply -f nginx-deployment.yaml
       
      # 修改 nginx-deployment.yaml 的内容
       
      $ kubectl apply -f nginx-deployment.yaml
      ```

    * 如果通过容器镜像，能够保证应用本身在开发与部署环境里的一致性的话，那么现在，Kubernetes 项目通过这些 YAML 文件，就保证了应用的“部署参数”在开发与部署环境中的一致性
    * **而当应用本身发生变化时，开发人员和运维人员可以依靠容器镜像来进行同步；当应用部署参数发生变化时，这些 YAML 文件就是他们相互沟通和信任的媒介**

* 在这个 Deployment 中尝试声明一个 Volume

  * 在 Kubernetes 中，Volume 是属于 Pod 对象的一部分。所以，需要修改这个 YAML 文件里的 template.spec 字段

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.8
            ports:
            - containerPort: 80
            volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: nginx-vol
          volumes:
          - name: nginx-vol
            emptyDir: {}
    ```

  * 在 Deployment 的 Pod 模板部分添加了一个 volumes 字段，定义了这个 Pod 声明的所有 Volume。它的名字叫作 nginx-vol，类型是 emptyDir

    *  emptyDir 类型，其实就等同于 Docker 的隐式 Volume 参数，即：不显式声明宿主机目录的 Volume。所以，Kubernetes 也会在宿主机上创建一个临时目录，这个目录将来就会被绑定挂载到容器所声明的 Volume 目录上

    * Pod 中的容器，使用的是 volumeMounts 字段来声明自己要挂载哪个 Volume，并通过 mountPath 字段来定义容器内的 Volume 目录，比如：/usr/share/nginx/html

    * Kubernetes 也提供了显式的 Volume 定义，它叫做 hostPath

      ```yaml
       ...   
          volumes:
            - name: nginx-vol
              hostPath: 
              	# 容器 Volume 挂载的宿主机目录，就变成了 /var/data
                path: /var/datas
      ```

  * 修改完成后，使用 kubectl apply 指令，更新这个 Deployment:

    ```shell
    kubectl apply -f nginx-deployment.yaml
    ```

  * 可以通过 kubectl get 指令，查看两个 Pod 被逐一更新的过程

    * 新旧两个 Pod，被交替创建、删除，最后剩下的就是新版本的 Pod。这个滚动更新的过程

    ```shell
    $ kubectl get pods
    NAME                                READY     STATUS              RESTARTS   AGE
    nginx-deployment-5c678cfb6d-v5dlh   0/1       ContainerCreating   0          4s
    nginx-deployment-67594d6bf6-9gdvr   1/1       Running             0          10m
    nginx-deployment-67594d6bf6-v6j7w   1/1       Running             0          10m
    $ kubectl get pods
    NAME                                READY     STATUS    RESTARTS   AGE
    nginx-deployment-5c678cfb6d-lg9lw   1/1       Running   0          8s
    nginx-deployment-5c678cfb6d-v5dlh   1/1       Running   0          19s
    ```

  * 使用 kubectl describe 查看一下最新的 Pod，就会发现 Volume 的信息已经出现在了 Container 描述部分

    ```shell
    ...
    Containers:
      nginx:
        Container ID:   docker://07b4f89248791c2aa47787e3da3cc94b48576cd173018356a6ec8db2b6041343
        Image:          nginx:1.8
        ...
        Environment:    <none>
        Mounts:
          /usr/share/nginx/html from nginx-vol (rw)
    ...
    Volumes:
      nginx-vol:
        Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
    ```

  * 可以使用 kubectl exec 指令，进入到这个 Pod 当中（即容器的 Namespace 中）查看这个 Volume 目录

    ```shell
    $ kubectl exec -it nginx-deployment-5c678cfb6d-lg9lw -- /bin/bash
    # ls /usr/share/nginx/html
    ```

  * 想要从 Kubernetes 集群中删除这个 Nginx Deployment 的话，直接执行

    ```shell
    $ kubectl delete -f nginx-deployment.yaml
    ```

# 为什么需要Pod

* Pod，是 Kubernetes 项目的原子调度单位

  * 这就意味着，Kubernetes 项目的调度器，是统一按照 Pod 而非容器的资源需求进行计算的

* 容器的本质是进程

* Kubernetes 就是操作系统

  ```shell
  # 展示当前系统中正在运行的进程的树状结构
  pstree -g
  ```

  * 在一个真正的操作系统里，进程并不是“孤苦伶仃”地独自运行的，而是以进程组的方式，“有原则地”组织在一起，相互协作。
  *  Kubernetes 项目所做的，其实就是将“进程组”的概念映射到了容器技术中，并使其成为了这个云计算“操作系统”里的“一等公民”
    * 容器的“单进程模型”，并不是指容器里只能运行“一个”进程，而是指容器没有管理多个进程的能力。这是因为容器里 PID=1 的进程就是应用本身，其他的进程都是这个 PID=1 进程的子进程。可是，用户编写的应用，并不能够像正常操作系统里的 init 进程或者 systemd 那样拥有进程管理的功能。比如，你的应用是一个 Java Web 程序（PID=1），然后你执行 docker exec 在后台启动了一个 Nginx 进程（PID=3）。可是，当这个 Nginx 进程异常退出的时候，你该怎么知道呢？这个进程退出后的垃圾收集工作，又应该由谁去做呢？

* 像这样容器间的紧密协作，可以称为“超亲密关系”

  * 这些具有“超亲密关系”容器的典型特征包括但不限于：
    * 互相之间会发生直接的文件交换
    * 使用 localhost 或者 Socket 文件进行本地通信
    * 会发生非常频繁的远程调用
    * 需要共享某些 Linux Namespace（比如，一个容器要加入另一个容器的 Network Namespace）等等
  * 这也就意味着，并不是所有有“关系”的容器都属于同一个 Pod。比如，PHP 应用容器和 MySQL 虽然会发生访问关系，但并没有必要、也不应该部署在同一台机器上，它们更适合做成两个 Pod

* Pod 在 Kubernetes 项目里还有更重要的意义，那就是：**容器设计模式**

  * Pod 的实现原理

    * **首先，关于 Pod 最重要的一个事实是：它只是一个逻辑概念**

      * Kubernetes 真正处理的，还是宿主机操作系统上 Linux 容器的 Namespace 和 Cgroups，而并不存在一个所谓的 Pod 的边界或者隔离环境

      * Pod，其实是一组共享了某些资源的容器

      * 具体的说：**Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume**

      * 如果通过 docker run --net --volumes-from 这样的命令实现，容器 B 就必须比容器 A 先启动，这样一个 Pod 里的多个容器就不是对等关系，而是拓扑关系了

        ```shell
        $ docker run --net=B --volumes-from=B --name=A image-A ...
        ```

      * 所以，在 Kubernetes 项目里，Pod 的实现需要使用一个中间容器，这个容器叫作 Infra 容器。在这个 Pod 中，Infra 容器永远都是第一个被创建的容器，而其他用户定义的容器，则通过 Join Network Namespace 的方式，与 Infra 容器关联在一起

        ![image-20200727103138466](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200727103138466.png)

        * 这个 Pod 里有两个用户容器 A 和 B，还有一个 Infra 容器。很容易理解，在 Kubernetes 项目里，Infra 容器一定要占用极少的资源，所以它使用的是一个非常特殊的镜像，叫作：`k8s.gcr.io/pause`。这个镜像是一个用汇编语言编写的、永远处于“暂停”状态的容器，解压后的大小也只有 100~200 KB 左右

        * 而在 Infra 容器“Hold 住”Network Namespace 后，用户容器就可以加入到 Infra 容器的 Network Namespace 当中了。所以，如果查看这些容器在宿主机上的 Namespace 文件，它们指向的值一定是完全一样的

        * 这也就意味着，对于 Pod 里的容器 A 和容器 B 来说：

          * 它们可以直接使用 localhost 进行通信
          * 它们看到的网络设备跟 Infra 容器看到的完全一样
          * 一个 Pod 只有一个 IP 地址，也就是这个 Pod 的 Network Namespace 对应的 IP 地址
          * 当然，其他的所有网络资源，都是一个 Pod 一份，并且被该 Pod 中的所有容器共享
          * Pod 的生命周期只跟 Infra 容器一致，而与容器 A 和 B 无关

        * 而对于同一个 Pod 里面的所有用户容器来说，它们的进出流量，也可以认为都是通过 Infra 容器完成的。这一点很重要，因为**将来如果要为 Kubernetes 开发一个网络插件时，应该重点考虑的是如何配置这个 Pod 的 Network Namespace，而不是每一个用户容器如何使用你的网络配置，这是没有意义的**

        * 这就意味着，如果你的网络插件需要在容器里安装某些包或者配置才能完成的话，是不可取的：Infra 容器镜像的 rootfs 里几乎什么都没有，没有你随意发挥的空间。当然，这同时也意味着你的网络插件完全不必关心用户容器的启动与否，而只需要关注如何配置 Pod，也就是 Infra 容器的 Network Namespace 即可

        * 有了这个设计之后，共享 Volume 就简单多了：Kubernetes 项目只要把所有 Volume 的定义都设计在 Pod 层级即可

          * 这样，一个 Volume 对应的宿主机目录对于 Pod 来说就只有一个，Pod 里的容器只要声明挂载这个 Volume，就一定可以共享这个 Volume 对应的宿主机目录

            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: two-containers
            spec:
              restartPolicy: Never
              volumes:
              - name: shared-data
                hostPath:      
                  path: /data
              containers:
              - name: nginx-container
                image: nginx
                volumeMounts:
                - name: shared-data
                  mountPath: /usr/share/nginx/html
              - name: debian-container
                image: debian
                volumeMounts:
                - name: shared-data
                  mountPath: /pod-data
                command: ["/bin/sh"]
                args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
            ```

            * debian-container 和 nginx-container 都声明挂载了 shared-data 这个 Volume。而 shared-data 是 hostPath 类型。所以，它对应在宿主机上的目录就是：/data。而这个目录，其实就被同时绑定挂载进了上述两个容器当中
            * 这就是为什么，nginx-container 可以从它的 /usr/share/nginx/html 目录中，读取到 debian-container 生成的 index.html 文件的原因

    * 明白了 Pod 的实现原理后，再来讨论“容器设计模式”

      * Pod 这种“超亲密关系”容器的设计思想，实际上就是希望，当用户想在一个容器里跑多个功能并不相关的应用时，应该优先考虑它们是不是更应该被描述成一个 Pod 里的多个容器

      * 为了能够掌握这种思考方式，就应该尽量尝试使用它来描述一些用单个容器难以解决的问题

      * **第一个最典型的例子是：WAR 包与 Web 服务器**

        * 现在有一个 Java Web 应用的 WAR 包，它需要被放在 Tomcat 的 webapps 目录下运行起来

        * 假如，现在只能用 Docker 来做这件事情，那该如何处理这个组合关系呢？

          * 一种方法是，把 WAR 包直接放在 Tomcat 镜像的 webapps 目录下，做成一个新的镜像运行起来。可是，这时候，如果你要更新 WAR 包的内容，或者要升级 Tomcat 镜像，就要重新制作一个新的发布镜像，非常麻烦
          * 另一种方法是，压根儿不管 WAR 包，永远只发布一个 Tomcat 容器。不过，这个容器的 webapps 目录，就必须声明一个 hostPath 类型的 Volume，从而把宿主机上的 WAR 包挂载进 Tomcat 容器当中运行起来。不过，这样就必须要解决一个问题，即：如何让每一台宿主机，都预先准备好这个存储有 WAR 包的目录呢？这样来看，只能独立维护一套分布式存储系统了

        * 实际上，有了 Pod 之后，这样的问题就很容易解决了。可以把 WAR 包和 Tomcat 分别做成镜像，然后把它们作为一个 Pod 里的两个容器“组合”在一起。这个 Pod 的配置文件如下所示：

          ```yaml
          apiVersion: v1
          kind: Pod
          metadata:
            name: javaweb-2
          spec:
            initContainers:
            - image: geektime/sample:v2
              name: war
              command: ["cp", "/sample.war", "/app"]
              volumeMounts:
              - mountPath: /app
                name: app-volume
            containers:
            - image: geektime/tomcat:7.0
              name: tomcat
              command: ["sh","-c","/root/apache-tomcat-7.0.42-v2/bin/start.sh"]
              volumeMounts:
              - mountPath: /root/apache-tomcat-7.0.42-v2/webapps
                name: app-volume
              ports:
              - containerPort: 8080
                hostPort: 8001 
            volumes:
            - name: app-volume
              emptyDir: {}
          ```

          * 在这个 Pod 中，定义了两个容器，第一个容器使用的镜像是 geektime/sample:v2，这个镜像里只有一个 WAR 包（sample.war）放在根目录下。而第二个容器则使用的是一个标准的 Tomcat 镜像
          * WAR 包容器的类型不再是一个普通容器，而是一个 Init Container 类型的容器
          * 在 Pod 中，所有 Init Container 定义的容器，都会比 spec.containers 定义的用户容器先启动。并且，Init Container 容器会按顺序逐一启动，而直到它们都启动并且退出了，用户容器才会启动
          * 所以，这个 Init Container 类型的 WAR 包容器启动后，执行了一句"cp /sample.war /app"，把应用的 WAR 包拷贝到 /app 目录下，然后退出
          * 而后这个 /app 目录，就挂载了一个名叫 app-volume 的 Volume
          * Tomcat 容器，同样声明了挂载 app-volume 到自己的 webapps 目录下
          * 所以，等 Tomcat 容器启动时，它的 webapps 目录下就一定会存在 sample.war 文件：这个文件正是 WAR 包容器启动时拷贝到这个 Volume 里面的，而这个 Volume 是被这两个容器共享的
          * 像这样，就用一种“组合”方式，解决了 WAR 包与 Tomcat 容器之间耦合关系的问题
          * 实际上，这个所谓的“组合”操作，正是容器设计模式里最常用的一种模式，它的名字叫：sidecar
          * sidecar 指的就是可以在一个 Pod 中，启动一个辅助容器，来完成一些独立于主进程（主容器）之外的工作
          * 比如，在这个应用 Pod 中，Tomcat 容器是要使用的主容器，而 WAR 包容器的存在，只是为了给它提供一个 WAR 包而已。所以，用 Init Container 的方式优先运行 WAR 包容器，扮演了一个 sidecar 的角色

      * **第二个例子，则是容器的日志收集**

        * 比如，现在有一个应用，需要不断地把日志文件输出到容器的 /var/log 目录中
        * 这时，就可以把一个 Pod 里的 Volume 挂载到应用容器的 /var/log 目录上
        * 然后，在这个 Pod 里同时运行一个 sidecar 容器，它也声明挂载同一个 Volume 到自己的 /var/log 目录上
        * 这样，接下来 sidecar 容器就只需要做一件事儿，那就是不断地从自己的 /var/log 目录里读取日志文件，转发到 MongoDB 或者 Elasticsearch 中存储起来。这样，一个最基本的日志收集工作就完成了
        * 跟第一个例子一样，这个例子中的 sidecar 的主要工作也是使用共享的 Volume 来完成对文件的操作

      * 但不要忘记，Pod 的另一个重要特性是，它的所有容器都共享同一个 Network Namespace

        * 这就使得很多与 Pod 网络相关的配置和管理，也都可以交给 sidecar 完成，而完全无须干涉用户容器。这里最典型的例子莫过于 Istio 这个微服务治理项目了
        * Istio 项目使用 sidecar 容器完成微服务治理的原理

* 无论是从具体的实现原理，还是从使用方法、特性、功能等方面，容器与虚拟机几乎没有任何相似的地方；也不存在一种普遍的方法，能够把虚拟机里的应用无缝迁移到容器中。因为，容器的性能优势，必然伴随着相应缺陷，即：它不能像虚拟机那样，完全模拟本地物理机环境中的部署方法
* 实际上，一个运行在虚拟机里的应用，哪怕再简单，也是被管理在 systemd 或者 supervisord 之下的**一组进程，而不是一个进程**。这跟本地物理机上应用的运行方式其实是一样的。这也是为什么，从物理机到虚拟机之间的应用迁移，往往并不困难
* 可是对于容器来说，一个容器永远只能管理一个进程。更确切地说，一个容器，就是一个进程。这是容器技术的“天性”，不可能被修改。所以，将一个原本运行在虚拟机里的应用，“无缝迁移”到容器中的想法，实际上跟容器的本质是相悖的
* 这也是当初 Swarm 项目无法成长起来的重要原因之一：一旦到了真正的生产环境上，Swarm 这种单容器的工作方式，就难以描述真实世界里复杂的应用架构了
* 现在可以这么理解 Pod 的本质：
  
  * Pod，实际上是在扮演传统基础设施里“虚拟机”的角色；而容器，则是这个虚拟机里运行的用户程序
* 所以下一次，当需要把一个运行在虚拟机里的应用迁移到 Docker 容器中时，一定要仔细分析到底有哪些进程（组件）运行在这个虚拟机里
* 然后，就可以把整个虚拟机想象成为一个 Pod，把这些进程分别做成容器镜像，把有顺序关系的容器，定义为 Init Container。这才是更加合理的、松耦合的容器编排诀窍，也是从传统应用架构，到“微服务架构”最自然的过渡方式
* Pod 这个概念，提供的是一种编排思想，而不是具体的技术方案
  * 所以，如果愿意的话，完全可以使用虚拟机来作为 Pod 的实现，然后把用户容器都运行在这个虚拟机里。甚至，可以去实现一个带有 Init 进程的容器项目，来模拟传统应用的运行方式。这些工作，在 Kubernetes 中都是非常轻松的，也是 CRI 的内容
  * 相反的，如果强行把整个应用塞到一个容器里，甚至不惜使用 Docker In Docker 这种在生产环境中后患无穷的解决方案，恐怕最后往往会得不偿失

# 深入解析Pod对象：基本概念

* Pod，而不是容器，才是 Kubernetes 项目中的最小编排单位。将这个设计落实到 API 对象上，容器（Container）就成了 Pod 属性里的一个普通的字段。那么，一个很自然的问题就是：到底哪些属性属于 Pod 对象，而又有哪些属性属于 Container 呢？
  * Pod 扮演的是传统部署环境里“虚拟机”的角色。这样的设计，是为了使用户从传统环境（虚拟机环境）向 Kubernetes（容器环境）的迁移，更加平滑
  * 而如果能把 Pod 看成传统环境里的“机器”、把容器看作是运行在这个“机器”里的“用户程序”，那么很多关于 Pod 对象的设计就非常容易理解了
  * 比如，**凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的**
    * 这些属性的共同特征是，它们描述的是“机器”这个整体，而不是里面运行的“程序”
      * 比如，配置这个“机器”的网卡（即：Pod 的网络定义），配置这个“机器”的磁盘（即：Pod 的存储定义），配置这个“机器”的防火墙（即：Pod 的安全定义）。更不用说，这台“机器”运行在哪个服务器之上（即：Pod 的调度）

##  Pod 中几个重要字段的含义和用法

* **NodeSelector：是一个供用户将 Pod 与 Node 进行绑定的字段**

  ```yaml
  apiVersion: v1
  kind: Pod
  ...
  spec:
   nodeSelector:
     disktype: ssd
  ```

  * 意味着这个 Pod 永远只能运行在携带了“disktype: ssd”标签（Label）的节点上；否则，它将调度失败

* **NodeName**：一旦 Pod 的这个字段被赋值，Kubernetes 项目就会被认为这个 Pod 已经经过了调度，调度的结果就是赋值的节点名字。所以，这个字段一般由调度器负责设置，但用户也可以设置它来“骗过”调度器，当然这个做法一般是在测试或者调试的时候才会用到

* **HostAliases：定义了 Pod 的 hosts 文件（比如 /etc/hosts）里的内容**

  ```yaml
  apiVersion: v1
  kind: Pod
  ...
  spec:
    hostAliases:
    - ip: "10.1.2.3"
      hostnames:
      - "foo.remote"
      - "bar.remote"
  ...
  ```

  * 在这个 Pod 的 YAML 文件中，设置了一组 IP 和 hostname 的数据。这样，这个 Pod 启动后，/etc/hosts 文件的内容将如下所示

    ```shell
    cat /etc/hosts
    # Kubernetes-managed hosts file.
    127.0.0.1 localhost
    ...
    10.244.135.10 hostaliases-pod
    10.1.2.3 foo.remote
    10.1.2.3 bar.remote
    ```

    * 需要指出的是，在 Kubernetes 项目中，如果要设置 hosts 文件里的内容，一定要通过这种方法。否则，如果直接修改了 hosts 文件的话，在 Pod 被删除重建之后，kubelet 会自动覆盖掉被修改的内容

    * **凡是跟容器的 Linux Namespace 相关的属性，也一定是 Pod 级别的**

      * 这个原因也很容易理解：Pod 的设计，就是要让它里面的容器尽可能多地共享 Linux Namespace，仅保留必要的隔离和限制能力。这样，Pod 模拟出的效果，就跟虚拟机里程序间的关系非常类似了

        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: nginx
        spec:
          # 意味着这个 Pod 里的容器要共享 PID Namespace
          shareProcessNamespace: true
          containers:
          - name: nginx
            image: nginx
          - name: shell
            image: busybox
            # 开启了 tty 和 stdin 
            # 等同于设置了 docker run 里的 -it（-i 即 stdin，-t 即 tty）参数
            # 可以直接认为 tty 就是 Linux 给用户提供的一个常驻小程序，用于接收用户的标准输入，返回操作系统的标准输出。当然，为了能够在 tty 中输入信息，还需要同时开启 stdin（标准输入流）
            stdin: true
            tty: true
        ```

        ```shell
        # 可以使用 shell 容器的 tty 跟这个容器进行交互
        kubectl create -f nginx.yaml
        # 使用 kubectl attach 命令，连接到 shell 容器的 tty 上
        kubectl attach -it nginx -c shell
        # 可以在 shell 容器里执行 ps 指令，查看所有正在运行的进程
        $ kubectl attach -it nginx -c shell
        / # ps ax
        PID   USER     TIME  COMMAND
            1 root      0:00 /pause
            8 root      0:00 nginx: master process nginx -g daemon off;
           14 101       0:00 nginx: worker process
           15 root      0:00 sh
           21 root      0:00 ps ax
        # 不仅可以看到它本身的 ps ax 指令，还可以看到 nginx 容器的进程，以及 Infra 容器的 /pause 进程。这就意味着，整个 Pod 里的每个容器的进程，对于所有容器来说都是可见的：它们共享了同一个 PID Namespace   
        ```

    * **凡是 Pod 中的容器要共享宿主机的 Namespace，也一定是 Pod 级别的定义**

      ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: nginx
      spec:
        hostNetwork: true
        hostIPC: true
        hostPID: true
        containers:
        - name: nginx
          image: nginx
        - name: shell
          image: busybox
          stdin: true
          tty: true
      ```

      * 在这个 Pod 中，定义了共享宿主机的 Network、IPC 和 PID Namespace。这就意味着，这个 Pod 里的所有容器，会直接使用宿主机的网络、直接与宿主机进行 IPC 通信、看到宿主机里正在运行的所有进程
      * 除了这些属性，Pod 里最重要的字段当属“Containers”了，“Init Containers”和“Containers”这两个字段都属于 Pod 对容器的定义，内容也完全相同，只是 Init Containers 的生命周期，会先于所有的 Containers，并且严格按照定义的顺序执行

  * Kubernetes 项目中对 Container 的定义，和 Docker 相比并没有什么太大区别

    *  Image（镜像）、Command（启动命令）、workingDir（容器的工作目录）、Ports（容器要开发的端口），以及 volumeMounts（容器要挂载的 Volume）都是构成 Kubernetes 项目中 Container 的主要字段

    * 还有几个属性值得额外关注

      * **首先，是 ImagePullPolicy 字段**。它定义了镜像拉取的策略

        * 而它之所以是一个 Container 级别的属性，是因为容器镜像本来就是 Container 定义中的一部分
        * ImagePullPolicy 的值默认是 Always，即每次创建 Pod 都重新拉取一次镜像。另外，当容器的镜像是类似于 nginx 或者 nginx:latest 这样的名字时，ImagePullPolicy 也会被认为 Always
        * 而如果它的值被定义为 Never 或者 IfNotPresent，则意味着 Pod 永远不会主动拉取这个镜像，或者只在宿主机上不存在这个镜像时才拉取

      * **其次，是 Lifecycle 字段**。它定义的是 Container Lifecycle Hooks

        * 顾名思义，Container Lifecycle Hooks 的作用，是在容器状态发生变化时触发一系列“钩子”

          ```yaml
          apiVersion: v1
          kind: Pod
          metadata:
            name: lifecycle-demo
          spec:
            containers:
            - name: lifecycle-demo-container
              image: nginx
              lifecycle:
              	# 在容器成功启动之后，在 /usr/share/message 里写入了一句“欢迎信息”（即 postStart 定义的操作）
                postStart:
                  exec:
                    command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
                # 在这个容器被删除之前，则先调用了nginx 的退出指令（即 preStop 定义的操作），从而实现容器的“优雅退出” 
                preStop:
                  exec:
                    command: ["/usr/sbin/nginx","-s","quit"]
          ```

          *  postStart指的是，在容器启动后，立刻执行一个指定的操作。需要明确的是，postStart 定义的操作，虽然是在 Docker 容器 ENTRYPOINT 执行之后，但它并不严格保证顺序。也就是说，在 postStart 启动时，ENTRYPOINT 有可能还没有结束
          * 当然，如果 postStart 执行超时或者错误，Kubernetes 会在该 Pod 的 Events 中报出该容器启动失败的错误信息，导致 Pod 也处于失败的状态
          * preStop 发生的时机，则是容器被杀死之前（比如，收到了 SIGKILL 信号）。而需要明确的是，preStop 操作的执行，是同步的。所以，它会阻塞当前的容器杀死流程，直到这个 Hook 定义操作完成之后，才允许容器被杀死，这跟 postStart 不一样

  *  **Pod 对象在 Kubernetes 中的生命周期**

    * Pod 生命周期的变化，主要体现在 Pod API 对象的**Status 部分**，这是它除了 Metadata 和 Spec 之外的第三个重要字段。其中，pod.status.phase，就是 Pod 的当前状态，它有如下几种可能的情况：

      1. Pending。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功

      2. Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中
      3. Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见
      4. Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志
      5. Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题

    * 更进一步地，Pod 对象的 Status 字段，还可以再细分出一组 Conditions。这些细分状态的值包括：PodScheduled、Ready、Initialized，以及 Unschedulable。它们主要用于描述造成当前 Status 的具体原因是什么

      * 比如，Pod 当前的 Status 是 Pending，对应的 Condition 是 Unschedulable，这就意味着它的调度出现了问题
      * Ready ，意味着 Pod 不仅已经正常启动（Running 状态），而且已经可以对外提供服务了

  * 仔细阅读 $GOPATH/src/k8s.io/kubernetes/vendor/k8s.io/api/core/v1/types.go 里，type Pod struct ，尤其是 PodSpec 部分的内容。争取做到下次看到一个 Pod 的 YAML 文件时，不再需要查阅文档，就能做到把常用字段及其作用信手拈来

# 深入解析Pod对象：使用进阶

* 特殊的 Volume，叫作 Projected Volume，投射数据卷，是 Kubernetes v1.11 之后的新特性	
  * 在 Kubernetes 中，有几种特殊的 Volume，它们存在的意义不是为了存放容器里的数据，也不是用来进行容器和宿主机之间的数据交换。这些特殊 Volume 的作用，是为容器提供预先定义好的数据。所以，从容器的角度来看，这些 Volume 里的信息就是仿佛是**被 Kubernetes“投射”（Project）进入容器当中的**。这正是 Projected Volume 的含义
  * 到目前为止，Kubernetes 支持的 Projected Volume 一共有四种：
    * Secret；
    * ConfigMap；
    * Downward API；
    * ServiceAccountToken

##  Secret

* 作用是帮助把 Pod 想要访问的加密数据，存放到 Etcd 中。然后，就可以通过在 Pod 的容器里挂载 Volume 的方式，访问到这些 Secret 里保存的信息了

* Secret 最典型的使用场景，莫过于存放数据库的 Credential 信息

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: test-projected-volume 
  spec:
    containers:
    - name: test-secret-volume
      image: busybox
      args:
      - sleep
      - "86400"
      volumeMounts:
      - name: mysql-cred
        mountPath: "/projected-volume"
        readOnly: true
    volumes:
    - name: mysql-cred
      projected:
        sources:
        - secret:
            name: user
        - secret:
            name: pass
  ```

  * 声明挂载的 Volume，并不是常见的 emptyDir 或者 hostPath 类型，而是 projected 类型
  * 而这个 Volume 的数据来源（sources），则是名为 user 和 pass 的 Secret 对象，分别对应的是数据库的用户名和密码
  * 这里用到的数据库的用户名、密码，正是以 Secret 对象的方式交给 Kubernetes 保存的。完成这个操作的指令，如下所示：

  ```shell
  $ cat ./username.txt
  admin
  $ cat ./password.txt
  c1oudc0w!
   
  $ kubectl create secret generic user --from-file=./username.txt
  $ kubectl create secret generic pass --from-file=./password.txt
  # 其中，username.txt 和 password.txt 文件里，存放的就是用户名和密码；而 user 和 pass，则是我为 Secret 对象指定的名字。而我想要查看这些 Secret 对象的话，只要执行一条 kubectl get 命令就可以了：
  $ kubectl get secrets
  NAME           TYPE                                DATA      AGE
  user          Opaque                                1         51s
  pass          Opaque                                1         51s
  ```
  * 当然，除了使用 kubectl create secret 指令外，也可以直接通过编写 YAML 文件的方式来创建这个 Secret 对象

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysecret
    type: Opaque
    data:
      user: YWRtaW4=
      pass: MWYyZDFlMmU2N2Rm
    ```

    * 需要注意的是，Secret 对象要求这些数据必须是经过 Base64 转码的，以免出现明文密码的安全隐患。这个转码操作也很简单，比如

      ```shell
      $ echo -n 'admin' | base64
      YWRtaW4=
      $ echo -n '1f2d1e2e67df' | base64
      MWYyZDFlMmU2N2Rm
      ```

    * 这里需要注意的是，像这样创建的 Secret 对象，它里面的内容仅仅是经过了转码，而并没有被加密。在真正的生产环境中，需要在 Kubernetes 中开启 Secret 的加密插件，增强数据的安全性

* 尝试一下创建这个 Pod

  ```shell
  kubectl create -f test-projected-volume.yaml
  ```

* 当 Pod 变成 Running 状态之后，再验证一下这些 Secret 对象是不是已经在容器里了：

  ```shell
  $ kubectl exec -it test-projected-volume -- /bin/sh
  $ ls /projected-volume/
  user
  pass
  $ cat /projected-volume/user
  root
  $ cat /projected-volume/pass
  1f2d1e2e67d
  ```

  * 从返回结果中，可以看到，保存在 Etcd 里的用户名和密码信息，已经以文件的形式出现在了容器的 Volume 目录里。而这个文件的名字，就是 kubectl create secret 指定的 Key，或者说是 Secret 对象的 data 字段指定的 Key
  * 更重要的是，像这样通过挂载方式进入到容器里的 Secret，一旦其对应的 Etcd 里的数据被更新，这些 Volume 里的文件内容，同样也会被更新。其实，**这是 kubelet 组件在定时维护这些 Volume**
  * 需要注意的是，这个更新可能会有一定的延时。所以**在编写应用程序时，在发起数据库连接的代码处写好重试和超时的逻辑，绝对是个好习惯**

## ConfigMap

* **与 Secret 类似**，与 Secret 的区别在于，ConfigMap 保存的是不需要加密的、应用所需的配置信息。而 ConfigMap 的用法几乎与 Secret 完全相同：可以使用 kubectl create configmap 从文件或者目录创建 ConfigMap，也可以直接编写 ConfigMap 对象的 YAML 文件

* 比如，一个 Java 应用所需的配置文件（.properties 文件），就可以通过下面这样的方式保存在 ConfigMap 里

  ```shell
  # .properties 文件的内容
  $ cat example/ui.properties
  color.good=purple
  color.bad=yellow
  allow.textmode=true
  how.nice.to.look=fairlyNice
   
  # 从.properties 文件创建 ConfigMap
  $ kubectl create configmap ui-config --from-file=example/ui.properties
   
  # 查看这个 ConfigMap 里保存的信息 (data)
  # kubectl get -o yaml 这样的参数，会将指定的 Pod API 对象以 YAML 的方式展示出来
  $ kubectl get configmaps ui-config -o yaml
  apiVersion: v1
  data:
    ui.properties: |
      color.good=purple
      color.bad=yellow
      allow.textmode=true
      how.nice.to.look=fairlyNice
  kind: ConfigMap
  metadata:
    name: ui-config
    ...
  ```

## Downward API

* 作用是让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: test-downwardapi-volume
    labels:
      zone: us-est-coast
      cluster: test-cluster1
      rack: rack-22
  spec:
    containers:
      - name: client-container
        image: k8s.gcr.io/busybox
        command: ["sh", "-c"]
        args:
        - while true; do
            if [[ -e /etc/podinfo/labels ]]; then
              echo -en '\n\n'; cat /etc/podinfo/labels; fi;
            sleep 5;
          done;
        volumeMounts:
          - name: podinfo
            mountPath: /etc/podinfo
            readOnly: false
    volumes:
      - name: podinfo
        projected:
          sources:
          - downwardAPI:
              items:
                - path: "labels"
                  fieldRef:
                    fieldPath: metadata.labels
  ```

  * 声明了一个 projected 类型的 Volume，Volume 的数据来源，变成了 Downward API

    * 而这个 Downward API Volume，则声明了要暴露 Pod 的 metadata.labels 信息给容器

    * 通过这样的声明方式，当前 Pod 的 Labels 字段的值，就会被 Kubernetes 自动挂载成为容器里的 /etc/podinfo/labels 文件

    * 而这个容器的启动命令，则是不断打印出 /etc/podinfo/labels 里的内容。所以，当我创建了这个 Pod 之后，就可以通过 kubectl logs 指令，查看到这些 Labels 字段被打印出来

      ```shell
      $ kubectl create -f dapi-volume.yaml
      $ kubectl logs test-downwardapi-volume
      cluster="test-cluster1"
      rack="rack-22"
      zone="us-est-coast"
      ```

* 目前，Downward API 支持的字段已经非常丰富了，比如：

  ```shell
  1. 使用 fieldRef 可以声明使用:
  spec.nodeName - 宿主机名字
  status.hostIP - 宿主机 IP
  metadata.name - Pod 的名字
  metadata.namespace - Pod 的 Namespace
  status.podIP - Pod 的 IP
  spec.serviceAccountName - Pod 的 Service Account 的名字
  metadata.uid - Pod 的 UID
  metadata.labels['<KEY>'] - 指定 <KEY> 的 Label 值
  metadata.annotations['<KEY>'] - 指定 <KEY> 的 Annotation 值
  metadata.labels - Pod 的所有 Label
  metadata.annotations - Pod 的所有 Annotation
   
  2. 使用 resourceFieldRef 可以声明使用:
  容器的 CPU limit
  容器的 CPU request
  容器的 memory limit
  容器的 memory request
  ```

* 不过，需要注意的是，Downward API 能够获取到的信息，**一定是 Pod 里的容器进程启动之前就能够确定下来的信息**。而如果想要获取 Pod 容器运行后才会出现的信息，比如，容器进程的 PID，那就肯定不能使用 Downward API 了，而应该考虑在 Pod 里定义一个 sidecar 容器
* 其实，Secret、ConfigMap，以及 Downward API 这三种 Projected Volume 定义的信息，大多还可以通过环境变量的方式出现在容器里。但是，通过环境变量获取这些信息的方式，不具备自动更新的能力。所以，一般情况下，都建议使用 Volume 文件的方式获取这些信息

## Service Account

*  Pod 中一个与Secret密切相关的概念

* 现在有了一个 Pod，能不能在这个 Pod 里安装一个 Kubernetes 的 Client，这样就可以从容器里直接访问并且操作这个 Kubernetes 的 API 了呢？

  * 当然是可以的，不过，首先要解决 API Server 的授权问题

* Service Account 对象的作用，就是 Kubernetes 系统内置的一种“服务账户”，它是 Kubernetes 进行权限分配的对象。比如，Service Account A，可以只被允许对 Kubernetes API 进行 GET 操作，而 Service Account B，则可以有 Kubernetes API 的所有操作的权限

* 像这样的 Service Account 的授权信息和文件，实际上保存在它所绑定的一个特殊的 Secret 对象里的。这个特殊的 Secret 对象，就叫作**ServiceAccountToken**。任何运行在 Kubernetes 集群上的应用，都必须使用这个 ServiceAccountToken 里保存的授权信息，也就是 Token，才可以合法地访问 API Server

* 所以说，Kubernetes 项目的 Projected Volume 其实只有三种，因为第四种 ServiceAccountToken，只是一种特殊的 Secret 而已

* 另外，为了方便使用，Kubernetes 已经为提供了一个的默认“服务账户”（default Service Account）。并且，任何一个运行在 Kubernetes 里的 Pod，都可以直接使用这个默认的 Service Account，而无需显示地声明挂载它

* **这是如何做到的呢？**

  * 当然还是靠 Projected Volume 机制

  * 如果查看一下任意一个运行在 Kubernetes 集群里的 Pod，就会发现，每一个 Pod，都已经自动声明一个类型是 Secret、名为 default-token-xxxx 的 Volume，然后 自动挂载在每个容器的一个固定目录上。比如：

    ```shell
    $ kubectl describe pod nginx-deployment-5c678cfb6d-lg9lw
    Containers:
    ...
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from default-token-s8rbq (ro)
    Volumes:
      default-token-s8rbq:
      Type:       Secret (a volume populated by a Secret)
      SecretName:  default-token-s8rbq
      Optional:    false
    ```

  * 这个 Secret 类型的 Volume，正是默认 Service Account 对应的 ServiceAccountToken。所以说，Kubernetes 其实在每个 Pod 创建的时候，自动在它的 spec.volumes 部分添加上了默认 ServiceAccountToken 的定义，然后自动给每个容器加上了对应的 volumeMounts 字段。这个过程对于用户来说是完全透明的
  
  * 这样，一旦 Pod 创建完成，容器里的应用就可以直接从这个默认 ServiceAccountToken 的挂载目录里访问到授权信息和文件。这个容器内的路径在 Kubernetes 里是固定的，即：/var/run/secrets/kubernetes.io/serviceaccount ，而这个 Secret 类型的 Volume 里面的内容如下所示：
  
    ```shell
    $ ls /var/run/secrets/kubernetes.io/serviceaccount 
    ca.crt namespace  token
    ```
  
  * 所以，应用程序只要直接加载这些授权文件，就可以访问并操作 Kubernetes API 了。而且，如果使用的是 Kubernetes 官方的 Client 包（`k8s.io/client-go`）的话，它还可以自动加载这个目录下的文件，不需要做任何配置或者编码操作
  * **这种把 Kubernetes 客户端以容器的方式运行在集群里，然后使用 default Service Account 自动授权的方式，被称作“InClusterConfig”，也是最推荐的进行 Kubernetes API 编程的授权方式**
  * 当然，考虑到自动挂载默认 ServiceAccountToken 的潜在风险，Kubernetes 允许你设置默认不为 Pod 里的容器自动挂载这个 Volume
  * 除了这个默认的 Service Account 外，很多时候还需要创建一些自定义的 Service Account，来对应不同的权限设置。这样，Pod 里的容器就可以通过挂载这些 Service Account 对应的 ServiceAccountToken，来使用这些自定义的授权信息

## 容器健康检查和恢复机制

* Pod 的另一个重要的配置

* 在 Kubernetes 中，可以为 Pod 里的容器定义一个健康检查“探针”（Probe）

  * 这样，kubelet 就会根据这个 Probe 的返回值决定这个容器的状态，而不是直接以容器进行是否运行（来自 Docker 返回的信息）作为依据。这种机制，是生产环境中保证应用健康存活的重要手段

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        test: liveness
      name: test-liveness-exec
    spec:
      containers:
      - name: liveness
        image: busybox
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 5
          periodSeconds: 5
    ```

    * 启动之后做的第一件事，就是在 /tmp 目录下创建了一个 healthy 文件，以此作为自己已经正常运行的标志。而 30 s 过后，它会把这个文件删除掉
    * 与此同时，定义了一个这样的 livenessProbe（健康检查）。它的类型是 exec，这意味着，它会在容器启动后，在容器里面执行一句指定的命令，比如：“cat /tmp/healthy”。这时，如果这个文件存在，这条命令的返回值就是 0，Pod 就会认为这个容器不仅已经启动，而且是健康的。这个健康检查，在容器启动 5 s 后开始执行（initialDelaySeconds: 5），每 5 s 执行一次（periodSeconds: 5）

  * **具体实践一下这个过程**

    * 首先，创建这个 Pod：

      ```shell
      kubectl create -f test-liveness-exec.yaml
      ```

    * 然后，查看这个 Pod 的状态：

      ```shell
      kubectl get pod
      NAME                READY     STATUS    RESTARTS   AGE
      test-liveness-exec   1/1       Running   0          10s
      # 由于已经通过了健康检查，这个 Pod 就进入了 Running 状态
      ```

    * 而 30 s 之后，再查看一下 Pod 的 Events：

      ```shell
      kubectl describe pod test-liveness-exec
      FirstSeen LastSeen    Count   From            SubobjectPath           Type        Reason      Message
      --------- --------    -----   ----            -------------           --------    ------      -------
      2s        2s      1   {kubelet worker0}   spec.containers{liveness}   Warning     Unhealthy   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
      ```

    * 不妨再次查看一下这个 Pod 的状态：

      ```shell
      kubectl get pod test-liveness-exec
      NAME           READY     STATUS    RESTARTS   AGE
      liveness-exec   1/1       Running   1          1m
      ```

      * Pod 并没有进入 Failed 状态，而是保持了 Running 状态。这是为什么呢？
      * 其实，如果注意到 RESTARTS 字段从 0 到 1 的变化，就明白原因了：这个异常的容器已经被 Kubernetes 重启了。在这个过程中，Pod 保持 Running 状态不变
      * 需要注意的是：Kubernetes 中并没有 Docker 的 Stop 语义。所以虽然是 Restart（重启），但实际却是重新创建了容器
      * 这个功能就是 Kubernetes 里的**Pod 恢复机制**，也叫 restartPolicy。它是 Pod 的 Spec 部分的一个标准字段（pod.spec.restartPolicy），默认值是 Always，即：任何时候这个容器发生了异常，它一定会被重新创建
      * 但一定要强调的是，Pod 的恢复过程，永远都是发生在当前节点上，而不会跑到别的节点上去。事实上，一旦一个 Pod 与一个节点（Node）绑定，除非这个绑定发生了变化（pod.spec.node 字段被修改），否则它永远都不会离开这个节点。这也就意味着，如果这个宿主机宕机了，这个 Pod 也不会主动迁移到其他节点上去
      * 而如果想让 Pod 出现在其他的可用节点上，就必须使用 Deployment 这样的“控制器”来管理 Pod，哪怕只需要一个 Pod 副本
        * 即一个单 Pod 的 Deployment 与一个 Pod 最主要的区别

    * 而作为用户，还可以通过设置 restartPolicy，改变 Pod 的恢复策略。除了 Always，它还有 OnFailure 和 Never 两种情况

      * Always：在任何情况下，只要容器不在运行状态，就自动重启容器；
      * OnFailure: 只在容器 异常时才自动重启容器；
      * Never: 从来不重启容器。

    * 在实际使用时，需要根据应用运行的特性，合理设置这三种恢复策略

      * 比如，一个 Pod，它只计算 1+1=2，计算完成输出结果后退出，变成 Succeeded 状态。这时，如果再用 restartPolicy=Always 强制重启这个 Pod 的容器，就没有任何意义了
      * 而如果要关心这个容器退出后的上下文环境，比如容器退出后的日志、文件和目录，就需要将 restartPolicy 设置为 Never。因为一旦容器被自动重新创建，这些内容就有可能丢失掉了（被垃圾回收了）

    * 两个基本的设计原理

      * **只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器（比如：Always），那么这个 Pod 就会保持 Running 状态，并进行容器重启**。否则，Pod 就会进入 Failed 状态

      * **对于包含多个容器的 Pod，只有它里面所有的容器都进入异常状态后，Pod 才会进入 Failed 状态**。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数，比如：

        ```shell
        $ kubectl get pod test-liveness-exec
        NAME           READY     STATUS    RESTARTS   AGE
        liveness-exec   0/1       Running   1    
        ```

    * 所以，假如一个 Pod 里只有一个容器，然后这个容器异常退出了。那么，只有当 restartPolicy=Never 时，这个 Pod 才会进入 Failed 状态。而其他情况下，由于 Kubernetes 都可以重启这个容器，所以 Pod 的状态保持 Running 不变
    * 而如果这个 Pod 有多个容器，仅有一个容器异常退出，它就始终保持 Running 状态，哪怕即使 restartPolicy=Never。只有当所有容器也异常退出之后，这个 Pod 才会进入 Failed 状态

*  livenessProbe ，除了在容器中执行命令外，livenessProbe 也可以定义为发起 HTTP 或者 TCP 请求的方式，定义格式如下：

  ```yaml
  ...
  livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
         httpHeaders:
         - name: X-Custom-Header
           value: Awesome
         initialDelaySeconds: 3
         periodSeconds: 3
  ```

  ```yaml
  ...
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
  ```

  * 所以，Pod 其实可以暴露一个健康检查 URL（比如 /healthz），或者直接让健康检查去检测应用的监听端口。这两种配置方法，在 Web 服务类的应用中非常常用

* 在 Kubernetes 的 Pod 中，还有一个叫 readinessProbe 的字段。虽然它的用法与 livenessProbe 类似，但作用却大不一样。readinessProbe 检查结果的成功与否，决定的这个 Pod 是不是能被通过 Service 的方式访问到，而并不影响 Pod 的生命周期

* Pod 的字段这么多，又不可能全记住，Kubernetes 能不能自动给 Pod 填充某些字段呢？

  * 开发人员只需要提交一个基本的、非常简单的 Pod YAML，Kubernetes 就可以自动给对应的 Pod 对象加上其他必要的信息，比如 labels，annotations，volumes 等等。而这些信息，可以是运维人员事先定义好的

  * 这么一来，开发人员编写 Pod YAML 的门槛，就被大大降低了

  * 所以，这个叫作 PodPreset（Pod 预设置）的功能 已经出现在了 v1.11 版本的 Kubernetes 中

  * 举个例子，现在开发人员编写了如下一个 pod.yaml 文件：

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: website
      labels:
        app: website
        role: frontend
    spec:
      containers:
        - name: website
          image: nginx
          ports:
            - containerPort: 80
    ```

  * 运维人员看到了这个 Pod，一定会连连摇头：这种 Pod 在生产环境里根本不能用啊！

  * 所以，这个时候，运维人员就可以定义一个 PodPreset 对象。在这个对象中，凡是他想在开发人员编写的 Pod 里追加的字段，都可以预先定义好。比如这个 preset.yaml

    ```yaml
    apiVersion: settings.k8s.io/v1alpha1
    kind: PodPreset
    metadata:
      name: allow-database
    spec:
      selector:
        matchLabels:
          role: frontend
      env:
        - name: DB_PORT
          value: "6379"
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
      volumes:
        - name: cache-volume
          emptyDir: {}
    ```

    * 在这个 PodPreset 的定义中，首先是一个 selector。这就意味着后面这些追加的定义，只会作用于 selector 所定义的、带有“role: frontend”标签的 Pod 对象，这就可以防止“误伤”

    * 然后，定义了一组 Pod 的 Spec 里的标准字段，以及对应的值。比如，env 里定义了 DB_PORT 这个环境变量，volumeMounts 定义了容器 Volume 的挂载目录，volumes 定义了一个 emptyDir 的 Volume

    * 接下来，假定运维人员先创建了这个 PodPreset，然后开发人员才创建 Pod：

      ```shell
      $ kubectl create -f preset.yaml
      $ kubectl create -f pod.yaml
      ```

    * 这时，Pod 运行起来之后，查看一下这个 Pod 的 API 对象：

      ```yaml
      $ kubectl get pod website -o yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: website
        labels:
          app: website
          role: frontend
        annotations:
          podpreset.admission.kubernetes.io/podpreset-allow-database: "resource version"
      spec:
        containers:
          - name: website
            image: nginx
            volumeMounts:
              - mountPath: /cache
                name: cache-volume
            ports:
              - containerPort: 80
            env:
              - name: DB_PORT
                value: "6379"
        volumes:
          - name: cache-volume
            emptyDir: {}
      ```

      * 可以清楚地看到，这个 Pod 里多了新添加的 labels、env、volumes 和 volumeMount 的定义，它们的配置跟 PodPreset 的内容一样。此外，这个 Pod 还被自动加上了一个 annotation 表示这个 Pod 对象被 PodPreset 改动过
      * 需要说明的是，**PodPreset 里定义的内容，只会在 Pod API 对象被创建之前追加在这个对象本身上，而不会影响任何 Pod 的控制器的定义**
      * 比如，现在提交的是一个 nginx-deployment，那么这个 Deployment 对象本身是永远不会被 PodPreset 改变的，被修改的只是这个 Deployment 创建出来的所有 Pod。这一点请务必区分清楚

    * 如果定义了同时作用于一个 Pod 对象的多个 PodPreset，会发生什么呢？

      * 实际上，Kubernetes 项目会帮助合并（Merge）这两个 PodPreset 要做的修改。而如果它们要做的修改有冲突的话，这些冲突字段就不会被修改

* 体会一下 Kubernetes“一切皆对象”的设计思想：比如应用是 Pod 对象，应用的配置是 ConfigMap 对象，应用要访问的密码则是 Secret 对象

* 所以，也就自然而然地有了 PodPreset 这样专门用来对 Pod 进行批量化、自动化修改的工具对象ss



