# Kubeadm

==可以安装成功==

## 1. 安装要求

- 一台或多台机器，操作系统 CentOS7.x-86_x64；
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多；
- 集群中所有机器之间网络互通；
- 可以访问外网，需要拉取镜像；
- 禁止swap分区

## 2. 准备环境

![image-20200729222306529](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200729222306529.png)

| 角色       | IP            |
| ---------- | ------------- |
| k8s-master | 192.168.31.61 |
| k8s-node1  | 192.168.31.62 |
| k8s-node2  | 192.168.31.63 |

```shell
# 关闭防火墙：
$ systemctl stop firewalld
$ systemctl disable firewalld

# 关闭selinux：
$ sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
$ setenforce 0# 临时

# 关闭swap：
$ swapoff -a  # 临时
$ vim /etc/fstab  # 永久

# 根据规划设置主机名：
$ hostnamectl set-hostname <hostname>

# 在master添加hosts：
$ cat >>/etc/hosts << EOF
192.168.31.61 k8s-master
192.168.31.62 k8s-node1
192.168.31.63 k8s-node2
EOF

# 将桥接的IPv4流量传递到iptables的链：
$ cat >/etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables =1
net.bridge.bridge-nf-call-iptables =1
EOF
$ sysctl --system  # 生效

# 时间同步：
$ yum install ntpdate -y
$ ntpdate time.windows.com
```

## 3. 安装Docker/kubeadm/kubelet【所有节点】

* Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker

### 3.1 安装Docker

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce
$ systemctl enable docker && systemctl start docker

# 配置镜像下载加速器
$ cat >/etc/docker/daemon.json << EOF
{
"registry-mirrors":["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
$ systemctl restart docker
```

### 3.2 添加阿里云YUM软件源

```shell
$ cat >/etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 3.3 安装kubeadm，kubelet和kubectl

* 由于版本更新频繁，这里指定版本号部署：

  ```shell
  $ yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
  $ systemctl enable kubelet
  ```
  
* kubelet：systemd守护进程管理

* kubeadm：部署工具

* kubectl：k8s命令行管理工具

  ```shell
  # 启动kubelet
  systemctl start kubelet
  
  [root@k8s-master ~]# systemctl status kubelet -l
  kubelet.service - kubelet: The Kubernetes Node Agent
  Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
  └─10-kubeadm.conf
  Active: activating (auto-restart) (Result: exit-code) since 四 2020-07-30 20:08:32 CST; 9s ago
  Docs: https://kubernetes.io/docs/
  Process: 9984 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
  Main PID: 9984 (code=exited, status=255)
  
  7月 30 20:08:32 k8s-master systemd[1]: kubelet.service: main process exited, code=exited, status=255/n/a
  7月 30 20:08:32 k8s-master systemd[1]: Unit kubelet.service entered failed state.
  7月 30 20:08:32 k8s-master systemd[1]: kubelet.service failed.
  # 解决
  ## 关闭swap
  $ swapoff -a  # 临时
  $ vim /etc/fstab  # 注释 swap 行 # 永久 
  ```
## 4. 部署Kubernetes Master

* 在192.168.31.61（Master）执行

  ```shell
  $ kubeadm init \
    --apiserver-advertise-address=192.168.0.241 \
    --image-repository registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.18.0 \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16
    
  FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
    [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
    To see the stack trace of this error execute with --v=5 or higher
    
    # In order to set /proc/sys/net/bridge/bridge-nf-call-iptables by editing /etc/sysctl.conf. There you can add [1]
    vi /etc/sysctl.conf
    	net.bridge.bridge-nf-call-iptables = 1
    # Then execute
    sudo sysctl -p  
  ```

  ```shell
  To start using your cluster, you need to run the following as a regular user:
  
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  You should now deploy a pod network to the cluster.
  Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
  
  Then you can join any number of worker nodes by running the following on each as root:
  
  kubeadm join 192.168.0.241:6443 --token 2xc47j.zf74g5oe6htjaxof \
  --discovery-token-ca-cert-hash sha256:34bfcc7ad040b48a6edbcaeb5f8e212cb3536d88060df3f1dd192e058e7721e3
  ```

  * —apiserver-advertise-address 集群通告地址
  * —image-repository 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。
  * —kubernetes-version K8s版本，与上面安装的一致
  * —service-cidr 集群内部虚拟网络，Pod统一访问入口
  * —pod-network-cidr Pod网络，与下面部署的CNI网络组件yaml中保持一致

* 拷贝kubectl使用的连接k8s认证文件到默认路径：

  ```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  $ kubectl get nodes
  NAME         STATUS   ROLES    AGE   VERSION
  k8s-master  NotReady    master   2m   v1.18.0
  ```

## 5. 加入Kubernetes Node

* 在192.168.31.62/63（Node）执行

* 向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

  ```shell
  $ kubeadm join 192.168.31.61:6443--token esce21.q6hetwm8si29qxwn \
  --discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
  ```

* 默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

  ```shell
  $ kubeadm token create --print-join-command
  ```

## 6. 部署容器网络（CNI）

* 这里使用Flannel作为Kubernetes容器网络方案，解决容器跨主机网络通信。

* Flannel是CoreOS维护的一个网络组件，Flannel为每个Pod提供全局唯一的IP，Flannel使用ETCD来存储Pod子网与Node IP之间的关系。flanneld守护进程在每台主机上运行，并负责维护ETCD信息和路由数据包

  ```shell
  $ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  
  # 修改国内镜像地址
  $ sed -i -r "s#quay.io/coreos/flannel:.*-amd64#lizhenliang/flannel:v0.11.0-amd64#g" kube-flannel.yml
  kubectl apply -f kube-flannel.yml
  
  $ kubectl get pods -n kube-system
  ```

## 7. 部署官方Dashboard（UI）

```shell
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

* 默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：

  ```shell
  $ vi recommended.yaml
  ...
  kind:Service
  apiVersion: v1
  metadata:
    labels:
      k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  spec:
    ports:
  - port:443
        targetPort:8443
        nodePort:30001
    selector:
      k8s-app: kubernetes-dashboard
    type:NodePort
  ...
  
  $ kubectl apply -f recommended.yaml
  
  $ kubectl get pods -n kubernetes-dashboard
  NAME                                         READY   STATUS    RESTARTS   AGE
  dashboard-metrics-scraper-6b4884c9d5-gl8nr   1/1  Running  0  13m
  kubernetes-dashboard-7f99b75bf4-89cds  1/1  Running  0  13m
  ```

  ```shell
  # 推荐
  wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
  
  vim recommended.yaml
  
  ---
  kind: Service
  apiVersion: v1
  metadata:
    labels:
      k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
  spec:
    type: NodePort #增加
    ports:
      - port: 443
        targetPort: 8443
        nodePort: 30000 #增加
    selector:
      k8s-app: kubernetes-dashboard
  ---
  #因为自动生成的证书很多浏览器无法使用，所以自己创建，注释掉kubernetes-dashboard-certs对象声明
  #apiVersion: v1
  #kind: Secret
  #metadata:
  #  labels:
  #    k8s-app: kubernetes-dashboard
  #  name: kubernetes-dashboard-certs
  #  namespace: kubernetes-dashboard
  #type: Opaque
  ---
  
  
  # 创建证书
  mkdir dashboard-certs
  cd dashboard-certs/
  
  #创建命名空间
  # kubectl create namespace kubernetes-dashboard
  
  # 创建key文件
  openssl genrsa -out dashboard.key 2048
  
  #证书请求
  openssl req -days 36000 -new -out dashboard.csr -key dashboard.key -subj '/CN=dashboard-cert'
  
  #自签证书
  openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt
  
  #创建kubernetes-dashboard-certs对象
  kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kubernetes-dashboard
  
  # 安装dashboard
  kubectl create -f ~/recommended.yaml 
  
  
  # 查看安装结果
  kubectl get pods -A  -o wide
  kubectl get service -n kubernetes-dashboard  -o wide
  
  # 创建dashboard管理员
  vim dashboard-admin.yaml
  
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    labels:
      k8s-app: kubernetes-dashboard
    name: dashboard-admin
    namespace: kubernetes-dashboard
    
  kubectl create -f ./dashboard-admin.yaml
  
  # 为用户分配权限
  vim dashboard-admin-bind-cluster-role.yaml
  
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: dashboard-admin-bind-cluster-role
    labels:
      k8s-app: kubernetes-dashboard
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  subjects:
  - kind: ServiceAccount
    name: dashboard-admin
    namespace: kubernetes-dashboard
    
  kubectl create -f ./dashboard-admin-bind-cluster-role.yaml
  
  # 查看并复制用户Token
  kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep dashboard-admin | awk '{print $1}')
  #
  Name:         dashboard-admin-token-qlvx8
  Namespace:    kubernetes-dashboard
  Labels:       <none>
  Annotations:  kubernetes.io/service-account.name: dashboard-admin
                kubernetes.io/service-account.uid: c9014028-43a0-4d50-9ecb-0a3f6aa96ae0
  
  Type:  kubernetes.io/service-account-token
  
  Data
  ====
  ca.crt:     1025 bytes
  namespace:  20 bytes
  token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Iko4VWpEMWN4aVYtVGFmWU80STBPZFV1OUEtekNsTWYtTlFHVzlWWlBwMG8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tcWx2eDgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYzkwMTQwMjgtNDNhMC00ZDUwLTllY2ItMGEzZjZhYTk2YWUwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.GAvZ0n4msgi7WXiKQxyTaqFrg_0qfyM68VPTbC0sHo-HUfd1mP4nH0YJ2sY0Q5RMjf4Zmi0MS4lG_Fc15OJm8RTtSDMRlaN1FIQcA3Ft8Qh6LQdtASZlco8xjj5nKPUIXcX9Ue5wEtAu83jYmLtNCIHBkORr2FaheyUglpJ8GFkO90-qhktgPux3KCCqCuEdTlY3vLUjzrwLjAFEl8vpvVOWRYymKPUl22e3z6dwMEawhaKpVMrm_UKpbkmrAb3wnIqS03H8-s0jxYsepAXm0t1g4ROvcASBIfY6i2hyIeYNbbGN5pSsHrb0Vx74Y-KmQ-8r4jrG6BB-hr_omKa8mg
  
  # 访问（Chrome访问不了，试下Safari）
  # https://192.168.175.101:30000
  https://192.168.0.243:30000
  ```

* 访问地址：https://NodeIP:30001

* 创建service account并绑定默认cluster-admin管理员集群角色：

  ```shell
  # 创建用户
  $ kubectl create serviceaccount dashboard-admin -n kube-system
  
  # 用户授权
  $ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
  
  # 获取用户Token
  $ kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
  
  
  [root@k8s-master ~]# kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
  Name:         dashboard-admin-token-mwzf4
  Namespace:    kube-system
  Labels:       <none>
  Annotations:  kubernetes.io/service-account.name: dashboard-admin
                kubernetes.io/service-account.uid: cde6e09e-a3d0-4904-91e7-6c491a7634e0
  
  Type:  kubernetes.io/service-account-token
  
  Data
  ====
  ca.crt:     1025 bytes
  namespace:  11 bytes
  token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Iko4VWpEMWN4aVYtVGFmWU80STBPZFV1OUEtekNsTWYtTlFHVzlWWlBwMG8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbXd6ZjQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiY2RlNmUwOWUtYTNkMC00OTA0LTkxZTctNmM0OTFhNzYzNGUwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.BdE3XP66EEKcDeTx4JYY-Y7SZyuvB3HSetg7BJGnbtZMDhQGhqo9w1wQkzXvIX-jPXJu_x9VoP_oLnwtdSynQRee_yc8tPfKz16vcmdP82FAUf8hVXf4LI7sAkzSFn_5u9_7KcMq9awuF6GeJdWW55Ei3jcXySADjLAP0XwXL91uQz-Rj1fPad78wE37050kF1X-2DOWylNg_tHDBqVuaUCa6_SkaKx4j-FhsEL3rQrx-fIjTy6E07bnTiTJvyFxoK-tuYnrrK4pyHiAADDoPxlM7KPltJCBTRLFzE3vj3g0cRguix4tk34RlyZIt8dJW9GOhSC3itSHX7RvFx8UmA
  ```
  
* 使用输出的Token登录Dashboard

  ![image-20200729223754909](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200729223754909.png)

  ![image-20200729223814533](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200729223814533.png)



```shell
kubectl get namespaces
kubectl get pods --all-namespaces
kubectl logs kubernetes-dashboard-7f99b75bf4-mdxvm -n kubernetes-dashboard

kubectl get pods -A -o wide
```



# 二进制



