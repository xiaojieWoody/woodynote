# 版本

```shell
# docker 18.09.0
# k8s 1.14.0
# ingress 0.19.0              # 注意版本
# calico	v3.9
```

## 启动

```shell
# master(192.168.0.161) 上 software/nginx/
sh restart.sh
# slave(192.168.0.164) harbor
docker-compose start
docker-compose up -d
# Harbor
http://hub.imooc.com/
admin/harbor
Harbor12345
# Ingress(slave 192.168.0.163)
```

# 安装

* 没特别说明则所有机器上都安装

## 准备

```shell
yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp
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

## 安装Docker

```shell
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
```

## 修改hosts文件

```shell
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

## 安装k8s

```shell
#1. 配置yum源
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

# 2. 安装kubeadm&kubelet&kubectl
yum install -y kubeadm-1.14.0-0 kubelet-1.14.0-0 kubectl-1.14.0-0

# 3. docker和k8s设置同一个cgroup
# docker
vi /etc/docker/daemon.json
    "exec-opts": ["native.cgroupdriver=systemd"],
    
systemctl restart docker
    
# kubelet，这边如果发现输出directory not exist，也说明是没问题的，大家继续往下进行即可
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
	
systemctl enable kubelet && systemctl start kubelet
```

## 配置国内镜像

```shell
# 查看kubeadm使用的镜像kubeadm config images list
# 解决国外镜像不能访问的问题
# 	创建kubeadm.sh脚本，用于拉取镜像/打tag/删除原有镜像
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

# 运行脚本
sh ./kubeadm.sh
# 查看镜像
docker images
```

## k8s master配置

```shell
# master节点上
# 1. 本地有镜像
kubeadm init --kubernetes-version=1.14.0 --apiserver-advertise-address=192.168.0.161 --pod-network-cidr=10.244.0.0/16
#【若要重新初始化集群状态：kubeadm reset，然后再进行上述操作】

# 2. 保存好最后kubeadm join的信息
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

# 3. 根据日志提示
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 此时kubectl cluster-info查看一下是否成功
kubectl get pods
kubectl get pods -n kube-system
# 4. 查看pod验证一下
# coredns没有启动，需要安装网络插件
kubectl get pods -n kube-system
# 5. 健康检查
curl -k https://localhost:6443/healthz
```

## 部署calico网络插件

```shell
# master节点
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

## slave节点join到集群

```shell
# 所有slave节点上执行
kubeadm join 192.168.0.161:6443 --token eesqmf.mqao0xs70hr1xdo6 \
    --discovery-token-ca-cert-hash sha256:836f685cba8615dbfe494c763aa6f8c1efb412962639852890fd639a8451b644
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
# 在master节点上检查集群信息
kubectl get nodes
# kubectl get nodes -w

NAME                   STATUS   ROLES    AGE     VERSION
master-kubeadm-k8s     Ready    master   19m     v1.14.0
worker01-kubeadm-k8s   Ready    <none>   3m6s    v1.14.0
worker02-kubeadm-k8s   Ready    <none>   2m41s   v1.14.0

kubectl get pods -n kube-system
```

## 查看日志 

* journalctl -f
* kubectl get all -n ingress-nginx
* kubectl get svc
* kubectl get pods -o wide
* kubectl describe svc tomcat-services
* kubectl describe pod nginx-ingress-controller-67c7566b6-22mcl -n ingress-nginx

## Ingress

```shell
# 选取个slave节点安装（打label）
kubectl label node slave2 name=ingress
# slave上查看80和443端口不能被占用
netstat -tpnl|grep 80
netstat -tpnl|grep 443
# 官网 https://kubernetes.github.io/ingress-nginx/deploy/
# mkdir /software/nginx-ingress
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.28.0/deploy/static/mandatory.yaml
# 用0.19.0版本 使用下面的mandatory.yaml
# 有两处做了修改
# kind: Deployment 部分的增加了  hostNetwork 和 nodeSelector
 spec:
      serviceAccountName: nginx-ingress-serviceaccount
      hostNetwork: true         # 增加
      nodeSelector:
        name: ingress           # 增加
      containers:
# master上执行
kubectl apply -f mandatory.yaml
kubectl get all -n ingress-nginx
# 如果长时间没变成running，则先单独拉去镜像
# 查看需要哪些镜像
grep image mandatory.yaml
# 所有node上 上单独拉去镜像
docker pull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
# 如果官方拉取慢，则找国内镜像源
docker pull quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
# 然后打tag
docker tag quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
# slave上拉取镜像好了后，master上也好了
kubectl get all -n ingress-nginx

# 测试 
# master上 /software/ingress-demo/ingress-demo.yaml
kubectl create -f ingress-demo.yaml
kubectl get pod -o wide 
# 宿主主机上绑定hosts  因为ingress在222上启动
192.168.0.163 tomcat.mooc.com
192.168.0.163 api.mooc.com
# 访问
http://api.mooc.com/          # 404
tomcat.mooc.com
# 查看日志
kubectl logs tomcat-demo-6bc7d5b6f4-bbzb8
journalctl -f
```

```shell
# mandatory.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx

---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: default-http-backend
  labels:
    app.kubernetes.io/name: default-http-backend
    app.kubernetes.io/part-of: ingress-nginx
  namespace: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: default-http-backend
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: default-http-backend
        app.kubernetes.io/part-of: ingress-nginx
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: default-http-backend
          # Any image is permissible as long as:
          # 1. It serves a 404 page at /
          # 2. It serves 200 on a /healthz endpoint
          image: k8s.gcr.io/defaultbackend-amd64:1.5
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 30
            timeoutSeconds: 5
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: 10m
              memory: 20Mi
            requests:
              cpu: 10m
              memory: 20Mi

---
apiVersion: v1
kind: Service
metadata:
  name: default-http-backend
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: default-http-backend
    app.kubernetes.io/part-of: ingress-nginx
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app.kubernetes.io/name: default-http-backend
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      hostNetwork: true
      nodeSelector:
        name: ingress
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1

---

```

```shell
# ingress-demo.yaml
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

## 搭建dashboard

```shell
# master 
# 1. 创建/etc/kubernetes/addons/dashboard-all.yaml
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

# 每次登录时都要执行下面两步来获取token
# 查看dashboard-admin的secret名字
$ ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')

# 打印secret的token
$ kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}'
```

## Harbor

```shell
# slave
# 1. 下载解压 harbor-offline-installer-v1.9.4.tgz
# 2. 修改配置文件harbor.yml
hostname: 192.168.0.162
# 3. 安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 国内镜像
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

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
		server 192.168.0.162:80;
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
```

```shell
# 登录harbor   （创建新用户harbor/Harbor12345）
# 1. 创建公共项目 kubernetes（添加成员harbor）
# 宿主主机上配置hosts
192.168.0.161 hub.imooc.com
# mster/slave上配置hosts ip 为 安装harbor的主机ip
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
# 先在harbor建立kubernetes项目
# push镜像
docker push hub.imooc.com/kubernetes/nginx:1.13.12
# slave上pull
docker pull hub.imooc.com/kubernetes/nginx:1.13.12
# Harbor上配置同步
# 可在多个服务器上同步镜像
```





