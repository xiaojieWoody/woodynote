![image-20191130212803585](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191130212803585.png)

# 初识k8s

* container k8s 不同云服务器，就像在本地一样
* docker-compose单机容器管理的技术

## K8S核心组件和架构图

* [官网](![image-20191124174539213](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191124174539213.png))

1. 先以container为起点，k8s既然是容器编排工具，那么一定会有container 

2. 那k8s如何操作这些container呢?从感性的角度来讲，得要有点逼格，k8s不想直接操作 container，因为操作container的事情是docker来做的，k8s中要有自己的最小操作单位，称之为 Pod，说白了，==Pod就是一个或多个Container的组合== 

3. ==那Pod的维护谁来做呢?那就是ReplicaSet，通过selector来进行管理== 

4. ==Pod和ReplicaSet的状态如何维护和监测呢?Deployment==

   ![image-20191124174916145](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191124174916145.png)

5. ==不妨把相同或者有关联的Pod分门别类一下，那怎么分门别类呢?Label== 

6. 具有相同label的service要是能够有个名称就好了，Service 

   * ==Selector，可以选择拥有相同label标签的Pod==

7. Pod运行在哪里呢?当然是机器，比如一台centos机器，把这个机器 称作为Node 

8. 难道只有一个Node吗?显然不太合适，多台Node共同组成集群才行 

![image-20191124174847603](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191124174847603.png)

9. 把目光转移到由3个Node节点组成的Master-Node集群 

10. 这个集群要配合完成一些工作，总要有一些组件的支持 

    > 1. 总得要有一个操作集群的客户端，也就是和集群打交道 
    >
    >    * kubectl 
    >
    > 2. 请求肯定是到达Master Node，然后再分配给Worker Node创建Pod之类的 
    >
    >    * 关键是命令通过kubectl过来之后，是不是要认证授权一下 ?
    >
    > 3. 请求过来之后，Master Node中谁来接收? 
    >
    >    * APIServer 
    >
    > 4. API收到请求之后，接下来调用哪个Worker Node创建Pod，Container之类的，得要有调度策略 
    >
    >    * [Scheduler](https://kubernetes.io/docs/concepts/scheduling/kube-scheduler/)
    >
    > 5. Scheduler通过不同的策略，真正要分发请求到不同的Worker Node上创建内容，具体谁负责? 
    >
    >    * Controller Manager 
    >
    > 6. Worker Node接收到创建请求之后，具体谁来负责 
    >
    >    * Kubelet服务，最终Kubelet会调用Docker Engine，创建对应的容器[这边是不是也反应出一 点，在Node上需要有Docker Engine，不然怎么创建维护容器?] 
    >
    > 7. 会不会涉及到域名解析的问题? 
    >
    >    * DNS
    >
    > 8. 是否需要有监控面板能够监测整个集群的状态? 
    >
    >    * Dashboard 
    >
    > 9. 集群中这些数据如何保存?分布式存储 
    >
    >    * ETCD 
    >
    > 10. 至于像容器的持久化存储，网络等可以联系一下Docker中的内容 
    >
    >     ![image-20191124184452301](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191124184452301.png)
    >
    >     ![image-20191124184523310](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191124184523310.png)
    >
    > 11. 官方架构图
    >
    >     ![image-20191124184548754](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191124184548754.png)

## 安装

### **The hard way** 

* [Kelsey Hightower ](https://github.com/kelseyhightower)

### 在线play-with-k8s

![image-20200101171025154](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200101171025154.png)

* [网址](https://labs.play-with-k8s.com/ )

### Cloud上搭建

* [github](https://github.com/kubernetes/kops )

### 企业级解决方案CoreOS

* [coreos](https://coreos.com/tectonic/ )

### **Minikube** 

* K8S单节点，适合在本地学习使用
* [官网](https://kubernetes.io/docs/setup/learning-environment/minikube/ )
* [github](https://github.com/kubernetes/minikube )

### **kubeadm** 

* 本地多节点 
* [github](https://github.com/kubernetes/kubeadm )

## 使用Minikube搭建单节点K8s

![image-20191201114605973](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191201114605973.png)

### **Windows** 

* [kubectl官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
* [minikube官网](https://kubernetes.io/docs/tasks/tools/install-minikube/)

* 选择任意一种虚拟化的方式 

  ```shell
  Hyper-V
  VirtualBox[课上选择的]
  ```

* 安装kubectl 

  ```shell
  #1. 根据官网步骤 [或] 直接下载
  https://storage.googleapis.com/kubernetes- release/release/v1.16.2/bin/windows/amd64/kubectl.exe
  #2. 配置kubectl.exe所在路径的环境变量，使得cmd窗口可以直接使用kubectl命令
  #3. kubectl version检查是否配置成功
  ```

* 安装minikube 

  ```shell
  #1. 根据官网步骤 [或] 直接下载
  https://github.com/kubernetes/minikube/releases/download/v1.5.2/minikube- windows-amd64.exe
  #2. 修改minikube-windows-amd64.exe名称为minikube.exe
  #3. 配置minikube所在路径的环境变量，使得cmd窗口可以直接使用minikube命令
  #4. minikube version检查是否配置成功
  ```

* 使用minikube创建单节点的k8s 

  ```shell
  minikube start --vm-driver=virtualbox --image-repository=gcr.azk8s.cn/google-
  containers
  ```

* 小结 

  ```shell
  # 其实就是通过minikube创建一个虚拟机
  # 这个虚拟机中安装好了单节点的K8S环境然后通过kubectl进行交互
  # 创建K8S 
  minikube start
  # 删除K8S 
  minikube delete
  # 进入到K8S的机器中 
  minikube ssh
  # 查看状态 
  minikube status
  # 进入dashboard 
  minikube dashboard
  ```

### **CentOS** 

* [kubectl官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)

* [minikube官网](https://kubernetes.io/docs/tasks/tools/install-minikube/)

* 安装docker 

* 安装kubectl 

  ```shell
  # 01 下载[这边我给大家下载好了，在网盘kubectl&minikube中，大家上传到自己的centos7机器中。] 
  # 02 授权
  chmod +x ./kubectl 
  # 03 添加到环境变量
  sudo mv ./kubectl /usr/local/bin/kubectl
  # 04 检查 
  kubectl version
  ```

* 安装minikube 

  ```shell
  # 01 下载[这边我给大家下载好了，在网盘kubectl&minikube中，大家上传到自己的centos7机器中。] 
  wget https://github.com/kubernetes/minikube/releases/download/v1.5.2/minikube- linux-amd64
  # 02 配置环境变量
  sudo mv minikube-linux-amd64 minikube && chmod +x minikube && mv minikube /usr/local/bin/
  # 03 检查 
  minikube version
  ```

* 使用minikube创建单节点的k8s 

  ```shell
  minikube start --vm-driver=none --image-repository=gcr.azk8s.cn/google-
  containers
  ```

### MacOS

* 也是下载安装kubectl和minikube，选择virtualbox，然后minikube start，就可以通过kubectl操作

## 感受Kubernetes 

* 既然已经通过Minikube搭建了单节点的Kubernetes，不妨先感受一些组件的存在以及操作

### **查看连接信息** 

```shell
kubectl config view
kubectl config get-contexts
kubectl cluster-info
```

### 体验Pod 

1. 创建pod_nginx.yaml 

   ```yaml
   # resources/basic/pod_nginx.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 80
   ```

2. 根据pod_nginx.yaml文件创建pod 

   ```yaml
   kubectl apply -f pod_nginx.yaml
   ```

3. 查看pod 

   ```shell
   kubectl get pods
   kubectl get pods -o wide
   kubectl describe pod nginx
   ```

4. 进入nginx容器 

   ```shell
   # kubectl进入
   kubectl exec -it nginx bash
   # 通过docker进入
   minikube ssh
   docker ps
   docker exec -it containerid bash
   ```

5. 访问nginx，端口转发 

   ```shell
   # 若在minikube中，直接访问
   # 若在物理主机上，要做端口转发
   kubectl port-forward nginx 8080:80
   ```

6. 删除pod 

   ```shell
   kubectl delete -f pod_nginx.yaml
   ```

* 总结：通过Minikube，我们使用kubectl操作单节点的K8S，而且也能感受到pod的创建和删除，包括 pod中对应的容器，一切才刚刚开始 

# ==搭建K8s集群==

* [官网](<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl>)
* [github](https://github.com/kubernetes/kubeadm)
* `课程中`：使用kubeadm搭建一个3台机器组成的k8s集群，1台master节点，2台worker节点
  * 机器配置不够，也可以使用在线的，或者minikube的方式或者1个master和1个worker

## 搭建

```shell
# 版本统一
Docker       18.09.0
---
kubeadm-1.14.0-0 
kubelet-1.14.0-0 
kubectl-1.14.0-0
---
k8s.gcr.io/kube-apiserver:v1.14.0
k8s.gcr.io/kube-controller-manager:v1.14.0
k8s.gcr.io/kube-scheduler:v1.14.0
k8s.gcr.io/kube-proxy:v1.14.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
---
calico:v3.9

# 准备3台centos

# 更新并安装依赖
## 3台机器都需要执行
yum -y update
yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp

# 安装Docker
## 在每一台机器上都安装好Docker，版本为18.09.0
## 1. 安装必要的依赖
sudo yum install -y yum-utils \
 device-mapper-persistent-data \
 lvm2
## 2. 设置docker仓库
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
### 【设置要设置一下阿里云镜像加速器】
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://0eipo2bm.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
## 3. 安装docker
yum install -y docker-ce-18.09.0 docker-ce-cli-18.09.0 containerd.io
## 4. 启动docker
sudo systemctl start docker && sudo systemctl enable docker

#  修改hosts文件
## 1. master
### 设置master的hostname，并且修改hosts文件
sudo hostnamectl set-hostname m
vi /etc/hosts
192.168.8.51 m
192.168.8.61 w1
192.168.8.62 w2
## 2. 两个worker
### 设置worker01/02的hostname，并且修改hosts文件
sudo hostnamectl set-hostname w1
sudo hostnamectl set-hostname w2
vi /etc/hosts
192.168.8.51 m
192.168.8.61 w1
192.168.8.62 w2
## 3. 使用ping测试一下
```

### 系统基础前提配置

```shell
## (1)关闭防火墙
systemctl stop firewalld && systemctl disable firewalld
## (2)关闭selinux
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
## (3)关闭swap
swapoff -a
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
## (4)配置iptables的ACCEPT规则
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
## (5)设置系统参数
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

###  Installing kubeadm, kubelet and kubectl

1. 配置yum源

   ```shell
   cat <<EOF > /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=0
   repo_gpgcheck=0
   gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
          http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   ```

2. 安装kubeadm&kubelet&kubectl

   ```shell
   yum install -y kubeadm-1.14.0-0 kubelet-1.14.0-0 kubectl-1.14.0-0
   ```

3. docker和k8s设置同一个cgroup

   ```shell
   # docker
   vi /etc/docker/daemon.json
       "exec-opts": ["native.cgroupdriver=systemd"],
       
   systemctl restart docker
       
   # kubelet，这边如果发现输出directory not exist，也说明是没问题的，大家继续往下进行即可
   sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
   	
   systemctl enable kubelet && systemctl start kubelet
   ```

### proxy/pause/scheduler等国内镜像

1. 查看kubeadm使用的镜像`kubeadm config images list`

2. 解决国外镜像不能访问的问题

   * 创建kubeadm.sh脚本，用于拉取镜像/打tag/删除原有镜像

     ```shell
     #!/bin/bash
     
     set -e
     
     KUBE_VERSION=v1.14.0
     KUBE_PAUSE_VERSION=3.1
     ETCD_VERSION=3.3.10
     CORE_DNS_VERSION=1.3.1
     
     GCR_URL=k8s.gcr.io
     ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers
     
     images=(kube-proxy:${KUBE_VERSION}
     kube-scheduler:${KUBE_VERSION}
     kube-controller-manager:${KUBE_VERSION}
     kube-apiserver:${KUBE_VERSION}
     pause:${KUBE_PAUSE_VERSION}
     etcd:${ETCD_VERSION}
     coredns:${CORE_DNS_VERSION})
     
     for imageName in ${images[@]} ; do
       docker pull $ALIYUN_URL/$imageName
       docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
       docker rmi $ALIYUN_URL/$imageName
     done
     ```

   * 运行脚本和查看镜像

     ```shell
     # 运行脚本
     sh ./kubeadm.sh
     # 查看镜像
     docker images
     ```

   * 将这些镜像推送到自己的阿里云仓库【可选，根据自己实际的情况】

     ```shell
     # 登录自己的阿里云仓库
     docker login --username=xxx registry.cn-hangzhou.aliyuncs.com
     ```

     ```shell
     #!/bin/bash
     
     set -e
     
     KUBE_VERSION=v1.14.0
     KUBE_PAUSE_VERSION=3.1
     ETCD_VERSION=3.3.10
     CORE_DNS_VERSION=1.3.1
     
     GCR_URL=k8s.gcr.io
     ALIYUN_URL=xxx
     
     images=(kube-proxy:${KUBE_VERSION}
     kube-scheduler:${KUBE_VERSION}
     kube-controller-manager:${KUBE_VERSION}
     kube-apiserver:${KUBE_VERSION}
     pause:${KUBE_PAUSE_VERSION}
     etcd:${ETCD_VERSION}
     coredns:${CORE_DNS_VERSION})
     
     for imageName in ${images[@]} ; do
       docker tag $GCR_URL/$imageName $ALIYUN_URL/$imageName
       docker push $ALIYUN_URL/$imageName
       docker rmi $ALIYUN_URL/$imageName
     done
     ```

   * 运行脚本 sh ./kubeadm-push-aliyun.sh

### kube init初始化master

1. kube init流程

   ```shell
   #01-进行一系列检查，以确定这台机器可以部署kubernetes
   #02-生成kubernetes对外提供服务所需要的各种证书可对应目录
   /etc/kubernetes/pki/*
   #03-为其他组件生成访问kube-ApiServer所需的配置文件
   ls /etc/kubernetes/
   admin.conf  controller-manager.conf  kubelet.conf  scheduler.conf   
   #04-为 Master组件生成Pod配置文件。
   ls /etc/kubernetes/manifests/*.yaml
   kube-apiserver.yaml 
   kube-controller-manager.yaml
   kube-scheduler.yaml    
   #05-生成etcd的Pod YAML文件。
   ls /etc/kubernetes/manifests/*.yaml
   kube-apiserver.yaml 
   kube-controller-manager.yaml
   kube-scheduler.yaml
   etcd.yaml	
   #06-一旦这些 YAML 文件出现在被 kubelet 监视的/etc/kubernetes/manifests/目录下，kubelet就会自动创建这些yaml文件定义的pod，即master组件的容器。master容器启动后，kubeadm会通过检查localhost：6443/healthz这个master组件的健康状态检查URL，等待master组件完全运行起来
   #07-为集群生成一个bootstrap token
   #08-将ca.crt等 Master节点的重要信息，通过ConfigMap的方式保存在etcd中，工后续部署node节点使用
   #09-最后一步是安装默认插件，kubernetes默认kube-proxy和DNS两个插件是必须安装的
   ```

2. 初始化master节点

   * [官网](<https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/>)

   * 此操作是在主节点上进行

     ```shell
     # 本地有镜像
     kubeadm init --kubernetes-version=1.14.0 --apiserver-advertise-address=192.168.0.131 --pod-network-cidr=10.244.0.0/16
     #【若要重新初始化集群状态：kubeadm reset，然后再进行上述操作】
     ```

   * 记得保存好最后kubeadm join的信息

     ```shell
     To start using your cluster, you need to run the following as a regular user:
     
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config
     
     You should now deploy a pod network to the cluster.
     Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
       https://kubernetes.io/docs/concepts/cluster-administration/addons/
     
     Then you can join any number of worker nodes by running the following on each as root:
     
     kubeadm join 192.168.0.131:6443 --token 4eu22b.yi2h7ehvrv28x9go \
         --discovery-token-ca-cert-hash sha256:b8d14bce6a626ef34cfacb9079f9742d3c6e3489c96961cb5631e46183bf2d90
     ```
     
     ```shell
     # 暂时
     Your Kubernetes control-plane has initialized successfully!
     
     To start using your cluster, you need to run the following as a regular user:
     
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config
     
     You should now deploy a pod network to the cluster.
     Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
       https://kubernetes.io/docs/concepts/cluster-administration/addons/
     
     Then you can join any number of worker nodes by running the following on each as root:
     
     kubeadm join 192.168.0.161:6443 --token eesqmf.mqao0xs70hr1xdo6 \
         --discovery-token-ca-cert-hash sha256:836f685cba8615dbfe494c763aa6f8c1efb412962639852890fd639a8451b644
     ```
   
3. 根据日志提示

   ```shell
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

   * 此时kubectl cluster-info查看一下是否成功

   ```shell
   kubectl get pods
   kubectl get pods -n kube-system
   ```

4. 查看pod验证一下

   * 等待一会儿，同时可以发现像etc，controller，scheduler等组件都以pod的方式安装成功了

   * `注意`：coredns没有启动，需要安装网络插件

     ```shell
     kubectl get pods -n kube-system
     ```

5. 健康检查

   ```shell
   curl -k https://localhost:6443/healthz
   ```

### 部署calico网络插件

* [选择网络插件](<https://kubernetes.io/docs/concepts/cluster-administration/addons/>)

* [calico网络插件](<https://docs.projectcalico.org/v3.9/getting-started/kubernetes/>)

* `calico，同样在master节点上操作`

  ```shell
  # 查看文件，提前拉取镜像
  docker pull calico/pod2daemon-flexvol:v3.9.1
  docker pull calico/kube-controllers:v3.9.1
  docker pull calico/cni:v3.9.1
  
  kubectl get pods -n kube-system -w
  
  # 在k8s中安装calico
  kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
  # 确认一下calico是否安装成功
  kubectl get pods --all-namespaces -w
  ```

### kube join

* 记得保存初始化master节点的最后打印信息

  ```shell
  kubeadm join 192.168.0.51:6443 --token yu1ak0.2dcecvmpozsy8loh \
      --discovery-token-ca-cert-hash sha256:5c4a69b3bb05b81b675db5559b0e4d7972f1d0a61195f217161522f464c307b0
  ```
  
  ```shell
  # 问题
  # error execution phase preflight: couldn't validate the identity of the API Server: abort connecting to API servers after timeout of 5m0s
  原因：master节点的token过期了
  # kubeadm token list
  在master重新生成token
  # kubeadm token create
  1zm15p.48rip1o42m8tgcze
  # openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
  d54c6585bd14abf235045bac5bf296fd7737e4b830da3a7947c402b9b13cd643
  # 重新join
  kubeadm join 192.168.0.161:6443 --token 1zm15p.48rip1o42m8tgcze \
  --discovery-token-ca-cert-hash sha256:d54c6585bd14abf235045bac5bf296fd7737e4b830da3a7947c402b9b13cd643
  ```

1. 在woker01和worker02上执行上述命令

2. ==查看日志 journalctl -f==

3. 在master节点上检查集群信息

   ```shell
   kubectl get nodes
   # kubectl get nodes -w
   
   NAME                   STATUS   ROLES    AGE     VERSION
   master-kubeadm-k8s     Ready    master   19m     v1.14.0
   worker01-kubeadm-k8s   Ready    <none>   3m6s    v1.14.0
   worker02-kubeadm-k8s   Ready    <none>   2m41s   v1.14.0
   
   kubectl get pods -n kube-system
   ```

### 搭建dashboard

```shell
# master 创建/etc/kubernetes/addons/dashboard-all.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    # Allows editing resource and makes sure it is created first.
    addonmanager.kubernetes.io/mode: EnsureExists
  name: kubernetes-dashboard-settings
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
        seccomp.security.alpha.kubernetes.io/pod: 'docker/default'
    spec:
      priorityClassName: system-cluster-critical
      containers:
      - name: kubernetes-dashboard
        image: registry.cn-hangzhou.aliyuncs.com/imooc/kubernetes-dashboard-amd64:v1.8.3
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 50m
            memory: 100Mi
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          # PLATFORM-SPECIFIC ARGS HERE
          - --auto-generate-certificates
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
        - name: tmp-volume
          mountPath: /tmp
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    # Allows editing resource and makes sure it is created first.
    addonmanager.kubernetes.io/mode: EnsureExists
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    # Allows editing resource and makes sure it is created first.
    addonmanager.kubernetes.io/mode: EnsureExists
  name: kubernetes-dashboard-key-holder
  namespace: kube-system
type: Opaque
---
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    k8s-app: kubernetes-dashboard
  ports:
  - port: 443
    targetPort: 8443
    nodePort: 30005
  type: NodePort
```

```shell
# 创建服务
$ kubectl apply -f /etc/kubernetes/addons/dashboard-all.yaml

# 查看服务运行情况
$ kubectl get deployment kubernetes-dashboard -n kube-system
$ kubectl --namespace kube-system get pods -o wide    # 查看运行在那个node上
$ kubectl get services kubernetes-dashboard -n kube-system
$ netstat -ntlp|grep 30005

# 访问 https://nodeIp:30005
# 看dashboard在哪台node上起动就哪台node的ip

# 创建service account
$ kubectl create sa dashboard-admin -n kube-system

# 创建角色绑定关系
$ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

# 查看dashboard-admin的secret名字
$ ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')

# 打印secret的token
$ kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}'
```

### 再次体验Pod

1. 定义pod.yml文件，比如pod_nginx_rs.yaml

   ```yaml
   cat > pod_nginx_rs.yaml <<EOF
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: nginx
     labels:
       tier: frontend
   spec:
     replicas: 3
     selector:
       matchLabels:
         tier: frontend
     template:
       metadata:
         name: nginx
         labels:
           tier: frontend
       spec:
         containers:
         - name: nginx
           image: nginx
           ports:
           - containerPort: 80
   EOF
   ```

2. 根据pod_nginx_rs.yml文件创建pod

   ```shell
   kubectl apply -f pod_nginx_rs.yaml
   ```

3. 查看pod

   ```shell
   kubectl get pods
   kubectl get pods -o wide          # 访问curl 192.168.190.66
   kubectl describe pod nginx
   ```

4. 感受通过rs将pod扩容

   ```shell
   kubectl scale rs nginx --replicas=5
   kubectl get pods -o wide
   ```

5. 删除pod

   ```shell
   kubectl delete -f pod_nginx_rs.yaml
   ```

![image-20191201163433503](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191201163433503.png)



# Basic

## yaml文件

* 后缀：可以是.yml或者是.yaml，更加推荐.yaml

### 基础

- 区分大小写
- 缩进表示层级关系，相同层级的元素左对齐
- 缩进只能使用空格，不能使用TAB
- "#"表示当前行的注释
- 是JSON文件的超级，两个可以转换
- ---表示分隔符，可以在一个文件中定义多个结构
- 使用key: value，其中":"和value之间要有一个英文空格

### Maps

1. 简单

   ```yaml
   apiVersion: v1
   kind: Pod
   #---表示分隔符，可选。要定义多个结构一定要分隔
   #apiVersion表示key，v1表示value，英文":"后面要有一个空格
   #kind表示key，Pod表示value
   #也可以这样写apiVersion: "v1"
   # 转换为JSON格式
   {
   "apiVersion": "v1",
   "kind": "Pod"
   }
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
   ```

2. 复杂

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
       
   #metadata表示key，下面的内容表示value，该value中包含两个直接的key：name和labels
   #name表示key，nginx-deployment表示value
   #labels表示key，下面的表示value，这个值又是一个map
   #app表示key，nginx表示value
   #相同层级的记得使用空间缩进，左对齐
   #转换为JSON格式
    {
   "apiVersion": "apps/v1",
   "kind": "Deployment",
   "metadata": {
            "name": "nginx-deployment",
            "labels": {
                       "app": "nginx"
                      }
           }
   }   
   ```

### Lists

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container01
    image: busybox:1.28
  - name: myapp-container02
    image: busybox:1.28
```

> containers表示key，下面的表示value，其中value是一个数组
>
> 数组中有两个元素，每个元素里面包含name和image
>
> image表示key，myapp-container表示value

```json
// 转为json格式
{
"apiVersion": "v1",
"kind": "Pod",
"metadata": {
           "name": "myapp",
           "labels": {
                       "app": "myapp"
                     }
         },
"spec": {
 "containers": [{
                 "name": "myapp-container01",
                 "image": "busybox:1.28",
                }, 
                {
                 "name": "myapp-container02",
                 "image": "busybox:1.28",
                }]
      }
}
```

### 找个k8s的yaml文件

* [官网](<https://kubernetes.io/docs/reference/>)

  ```yaml
  # yaml格式对于Pod的定义：
  apiVersion: v1          #必写，版本号，比如v1
  kind: Pod               #必写，类型，比如Pod
  metadata:               #必写，元数据
    name: nginx           #必写，表示pod名称
    namespace: default    #表示pod名称属于的命名空间
    labels:
      app: nginx                  #自定义标签名字
  spec:                           #必写，pod中容器的详细定义
    containers:                   #必写，pod中容器列表
    - name: nginx                 #必写，容器名称
      image: nginx                #必写，容器的镜像名称
      ports:
      - containerPort: 80         #表示容器的端口
  ```

## Container

* [官网](https://kubernetes.io/docs/concepts/containers/)

1. Docker世界中
   * 可以通过docker run运行一个容器
   * 或者定义一个yml文件，本机使用docker-compose，多机通过docker swarm创建
2. K8S世界中
   * 同样以一个yaml文件维护，container运行在pod中

## Pod

* [官网](<https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/>)

### Pod初体验

1. 创建一个pod的yaml文件，名称为nginx_pod.yaml

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx-pod
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx-container
       image: nginx
       ports:
       - containerPort: 80
   ```

2. 根据该nginx_pod.yaml文件创建pod

   ```shell
   kubectl apply -f nginx_pod.yaml
   ```

3. 查看pod

   1. kubectl get pods

   ```shell
   NAME        READY   STATUS    RESTARTS   AGE
   nginx-pod   1/1     Running   0          29s
   ```

   2. kubectl get pods -o wide

   ```shell
   NAME       READY     STATUS   RESTARTS   AGE             IP             NODE   
   nginx-pod   1/1     Running      0       40m       192.168.80.194        w2 
   ```
   3. kubectl describe pod nginx-pod

   ```shell
   Name:               nginx-pod
   Namespace:          default
   Priority:           0
   PriorityClassName:  <none>
   Node:               w2/192.168.0.62
   Start Time:         Sun, 06 Oct 2019 20:45:35 +0000
   Labels:             app=nginx
   Annotations:        cni.projectcalico.org/podIP: 192.168.80.194/32
                       kubectl.kubernetes.io/last-applied-configuration:
                         {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-pod","namespace":"default"},"spec":{"c...
   Status:             Running
   IP:                 192.168.80.194
   Containers:
     nginx-container:
       Container ID:   docker://eb2fd0b2906f53e9892e22a6fd791c9ac68fb8e5efce3bbf94ec12bae96e1984
       Image:          nginx
       Image ID:       docker-pullable:/
   ```

   4. 可以发现该pod运行在worker02节点上
      * 于是来到worker02节点，docker ps一下

   ```shell
   CONTAINER ID  IMAGE  COMMAND                    CREATED        STATUS   PORTS   NAMES
   eb2fd0b2906f  nginx  "nginx -g 'daemon of…"   6 minutes ago       Up 6 minutes           k8s_nginx-container_nginx-pod_default_3ee0706d-e87a-11e9-a904-5254008afee6_0
   ```

   	* 不妨进入该容器试试[可以发现只有在worker02上有该容器，因为pod运行在worker02上]：
   	* docker exec -it k8s_nginx-container_nginx-pod_default_3ee0706d-e87a-11e9-a904-5254008afee6_0 bash
   	* root@nginx-pod:/#

   5. 访问nginx容器

      ```shell
      curl 192.168.80.194    OK，并且在任何一个集群中的Node上访问都成功
      ```

   6. 删除Pod

      ```shell
      kubectl delete -f nginx_pod.yaml
      kubectl get pods
      ```

### Storage and Networking

* [官网](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#networking)

* Networking

  ```shell
  Each Pod is assigned a unique IP address. Every container in a Pod shares the network namespace, including the IP address and network ports. 
  ```

* [官网](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#storage)

* Storage

  ```shell
  A Pod can specify a set of shared storage Volumes. All containers in the Pod can access the shared volumes, allowing those containers to share data.
  ```

  











