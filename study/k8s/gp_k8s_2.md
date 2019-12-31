# Controllers

* [官网](https://kubernetes.io/docs/concepts/workloads/controllers/)

## ReplicationController(RC)

* [官网](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)

  ```shell
  A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.
  ```

* ReplicationController定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值，所以RC的定义包含以下几个部分：

  * Pod期待的副本数（replicas）
  * 用于筛选目标Pod的Label Selector
  * 当Pod的副本数量小于预期数量时，用于创建新Pod的Pod模板（template）

* 也就是说通过RC实现了集群中Pod的高可用，减少了传统IT环境中手工运维的工作

  ```shell
  # kind：表示要新建对象的类型
  # spec.selector：表示需要管理的Pod的label，这里表示包含app: nginx的label的Pod都会被该RC管理
  # spec.replicas：表示受此RC管理的Pod需要运行的副本数
  # spec.template：表示用于定义Pod的模板，比如Pod名称、拥有的label以及Pod中运行的应用等
  # 通过改变RC里Pod模板中的镜像版本，可以实现Pod的升级功能
  # kubectl apply -f nginx-pod.yaml，此时k8s会在所有可用的Node上，创建3个Pod，并且每个Pod都有一个app: nginx的label，同时每个Pod中都运行了一个nginx容器
  # 如果某个Pod发生问题，Controller Manager能够及时发现，然后根据RC的定义，创建一个新的Pod
  # 扩缩容：kubectl scale rc nginx --replicas=5
  ```

1. 创建名为nginx_replication.yaml

   ```yaml
   apiVersion: v1
   kind: ReplicationController
   metadata:
     name: nginx
   spec:
     replicas: 3
     selector:
       app: nginx
     template:
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

2. 根据nginx_replication.yaml创建pod

   ```shell
   kubectl apply -f nginx_replication.yaml
   ```

3. 查看pod

   ```shell
   kubectl get pods -o wide
   
      NAME      READY     STATUS
   nginx-hksg8   1/1     Running   0          44s   192.168.80.195   w2   
   nginx-q7bw5   1/1     Running   0          44s   192.168.190.67   w1  
   nginx-zzwzl   1/1     Running   0          44s   192.168.190.68   w1    
   
   kubectl get rc
   NAME    DESIRED   CURRENT   READY   AGE
   nginx   3         3         3       2m54s
   ```

4. 尝试删除一个pod

   ```shell
   kubectl delete pods nginx-zzwzl
   kubectl get pods
   ```

5. 对pod进行扩缩容

   ```shell
   kubectl scale rc nginx --replicas=5
   kubectl get pods
   nginx-8fctt   0/1     ContainerCreating   0          2s
   nginx-9pgwk   0/1     ContainerCreating   0          2s
   nginx-hksg8   1/1     Running             0          6m50s
   nginx-q7bw5   1/1     Running             0          6m50s
   nginx-wzqkf   1/1     Running             0          99s
   ```

6. 删除pod

   ```shell
   kubectl delete -f nginx_replication.yaml
   ```

## ReplicaSet(RS)

* [官网](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

  ```shell
  A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.
  ```

* 在Kubernetes v1.2时，RC就升级成了另外一个概念：Replica Set，官方解释为“下一代RC”

  ReplicaSet和RC没有本质的区别，kubectl中绝大部分作用于RC的命令同样适用于RS

  RS与RC唯一的区别是：RS支持基于集合的Label Selector（Set-based selector），而RC只支持基于等式的Label Selector（equality-based selector），这使得Replica Set的功能更强

  ```yaml
  apiVersion: extensions/v1beta1
  kind: ReplicaSet
  metadata:
    name: frontend
  spec:
    matchLabels: 
      tier: frontend
    matchExpressions: 
      - {key:tier,operator: In,values: [frontend]}
    template:
    ...
  ```

* `注意`：一般情况下，我们很少单独使用Replica Set，它主要是被Deployment这个更高的资源对象所使用，从而形成一整套Pod创建、删除、更新的编排机制。当我们使用Deployment时，无须关心它是如何创建和维护Replica Set的，这一切都是自动发生的。同时，无需担心跟其他机制的不兼容问题（比如ReplicaSet不支持rolling-update但Deployment支持）

## Deployment

* [官网](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

* ```shell
  A Deployment provides declarative updates for Pods and ReplicaSets.
  You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.
  ```

  * Deployment相对RC最大的一个升级就是我们可以随时知道当前Pod“部署”的进度。

  * 创建一个Deployment对象来生成对应的Replica Set并完成Pod副本的创建过程

  * 检查Deploymnet的状态来看部署动作是否完成（Pod副本的数量是否达到预期的值）

  1. 创建nginx_deployment.yaml文件

     ```shell
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deployment
       labels:
         app: nginx
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: nginx
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

  2. 根据nginx_deployment.yaml文件创建pod

     ```shell
     kubectl apply -f nginx_deployment.yaml
     ```

  3. 查看pod

     ```shell
     kubectl get pods -o wide
     kubectl get deployment
     kubectl get rs
     kubectl get deployment -o wide
     ```

     ```shell
     nginx-deployment-6dd86d77d-f7dxb   1/1     Running   0      22s   192.168.80.198   w2 
     nginx-deployment-6dd86d77d-npqxj   1/1     Running   0      22s   192.168.190.71   w1 
     nginx-deployment-6dd86d77d-swt25   1/1     Running   0      22s   192.168.190.70   w1
     ```

     * nginx-deployment[deployment]-6dd86d77d[replicaset]-f7dxb[pod] 

  4. 当前nginx的版本

     ```shell
     kubectl get deployment -o wide
     
     NAME    READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES      SELECTOR
     nginx-deployment   3/3         3     3  3m27s      nginx    nginx:1.7.9   app=nginx
     ```

  5. 更新nginx的image版本

     ```shell
     kubectl set image deployment nginx-deployment nginx=nginx:1.9.1
     ```

# Labels and Selectors

* 在前面的yaml文件中，看到很多label，顾名思义，就是给一些资源打上标签的

* [官网](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

  * `Labels are key/value pairs that are attached to objects, such as pods. `

  ```shell
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
    labels:
      app: nginx
  ```
  * 表示名称为nginx-pod的pod，有一个label，key为app，value为nginx
  * 可以将具有同一个label的pod，交给selector管理

  ```shell
  apiVersion: apps/v1
  kind: Deployment
  metadata: 
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:             # 匹配具有同一个label属性的pod标签
      matchLabels:
        app: nginx         
    template:             # 定义pod的模板
      metadata:
        labels:
          app: nginx      # 定义当前pod的label属性，app为key，value为nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
          - containerPort: 80
  ```

  * 查看pod的label标签：kubectl get pods --show-labels
  * **这里可以尝试一下selector匹配不上的结果**

# Namespace

* kubectl get pods
* kubectl get pods -n kube-system

> 比较一下，上述两行命令的输入是否一样，发现不一样，是因为Pod属于不同的Namespace

> 查看一下当前的命名空间：kubectl get namespaces/ns

```shell
NAME              STATUS   AGE
default           Active   27m
kube-node-lease   Active   27m
kube-public       Active   27m
kube-system       Active   27m
```

* 其实说白了，命名空间就是为了隔离不同的资源，比如：Pod、Service、Deployment等。可以在输入命令的时候指定命名空间`-n`，如果不指定，则使用默认的命名空间：default

## 创建命名空间

* myns-namespace.yaml

```shell
apiVersion: v1
kind: Namespace
metadata:
  name: myns
```

* kubectl apply -f myns-namespace.yaml
* kubectl get namespaces/ns

```shell
NAME              STATUS   AGE
default           Active   38m
kube-node-lease   Active   38m
kube-public       Active   38m
kube-system       Active   38m
myns              Active   6s
```

## 指定命名空间下的资源

* 比如创建一个pod，属于myns命名空间下

* vi nginx-pod.yaml

* kubectl apply -f nginx-pod.yaml 

  ```shell
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
    namespace: myns
  spec:
    containers:
    - name: nginx-container
      image: nginx
      ports:
      - containerPort: 80
  ```

* 查看myns命名空间下的Pod和资源

  ```shell
  kubectl get pods
  kubectl get pods -n myns
  kubectl get all -n myns
  kubectl get pods --all-namespaces    #查找所有命名空间下的pod
  ```

# Network

## 同一个Pod中的容器通信

* 接下来就要说到跟Kubernetes网络通信相关的内容咯

* 我们都知道K8S最小的操作单位是Pod，先思考一下同一个Pod中多个容器要进行通信

  * 每个Pod中都会有个pause container，所有的创建的container都会连接到它上面

* 由官网的这段话可以看出，同一个pod中的容器是共享网络ip地址和端口号的，通信显然没问题

  ```shell
  Each Pod is assigned a unique IP address. Every container in a Pod shares the network namespace, including the IP address and network ports. 
  ```

* 那如果是通过容器的名称进行通信呢？就需要将所有pod中的容器加入到同一个容器的网络中，我们把该容器称作为pod中的pause container

## 集群内Pod之间的通信

![image-20191201213530550](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191201213530550.png)

* 接下来就聊聊K8S最小的操作单元，Pod之间的通信

* 我们都之间Pod会有独立的IP地址，这个IP地址是被Pod中所有的Container共享的

* 那多个Pod之间的通信能通过这个IP地址吗？

* 分两个维度：

  * 一是集群中同一台机器中的Pod
  * 二是集群中不同机器中的Pod

* 结论

  * 得益于网络插件calico，集群内部不管是pod访问pod，还是pod访问node，还是node访问pod都可以成功。【cluster Network】
  * 

* **准备两个pod，一个nginx，一个busybox**

  ```yaml
  # nginx_pod.yaml
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

  ```yaml
  # busybox_pod.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: busybox
    labels:
       app: busybox
  spec:
   containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  ```

* 将两个pod运行起来，并且查看运行情况

  > kubectl apply -f nginx_pod.yaml
  >
  > kubectl apply -f busy_pod.yaml
  >
  > kubectl get pods -o wide
  >
  > ```shell
  > NAME      READY  STATUS    RESTARTS   AGE         IP                NODE  
  > busybox    1/1   Running      0       49s    192.168.221.70   worker02-kubeadm-k8s   
  > nginx-pod  1/1   Running      0      7m46s   192.168.14.1     worker01-kubeadm-k8s 
  > ```
  >
  > `发现`：nginx-pod的ip为192.168.14.1     busybox-pod的ip为192.168.221.70

### 同一个集群中同一台机器

1. 来到worker01：ping 192.168.14.1

   ```shell
   PING 192.168.14.1 (192.168.14.1) 56(84) bytes of data.
   64 bytes from 192.168.14.1: icmp_seq=1 ttl=64 time=0.063 ms
   64 bytes from 192.168.14.1: icmp_seq=2 ttl=64 time=0.048 ms
   ```

2. 来到worker01：curl 192.168.14.1

   ```shell
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
   <style>
       body {
           width: 35em;
           margin: 0 auto;
           font-family: Tahoma, Verdana, Arial, sans-serif;
       }
   </style>
   ```

### 同一个集群中不同机器

1. 来到worker02：ping 192.168.14.1

   ```shell
   [root@worker02-kubeadm-k8s ~]# ping 192.168.14.1
   PING 192.168.14.1 (192.168.14.1) 56(84) bytes of data.
   64 bytes from 192.168.14.1: icmp_seq=1 ttl=63 time=0.680 ms
   64 bytes from 192.168.14.1: icmp_seq=2 ttl=63 time=0.306 ms
   64 bytes from 192.168.14.1: icmp_seq=3 ttl=63 time=0.688 ms
   ```

2. 来到worker02：curl 192.168.14.1，同样可以访问nginx

3. 来到master：

   ```shell
   ping/curl 192.168.14.1          访问的是worker01上的nginx-pod
   ping          192.168.221.70     访问的是worker02上的busybox-pod
   ```

4. 来到worker01：ping 192.168.221.70         访问的是worker02上的busybox-pod

### How to implement the Kubernetes Cluster networking model--Calico

* [官网](<https://kubernetes.io/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model>)
* pods on a node can communicate with all pods on all nodes without NAT
* agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node
* pods in the host network of a node can communicate with all pods on all nodes without NAT

## 集群内Service-Cluster IP

![image-20191201214621048](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191201214621048.png)

* pod其实是很不稳定的，deployment，pod挂掉，新开启pod ip
  
* service:cluster IP  可供集群内访问，对很多Deployment[Pod]进行负载均衡
  
* 对于上述的Pod虽然实现了集群内部互相通信，但是Pod是不稳定的，比如通过Deployment管理Pod，随时可能对Pod进行扩缩容，这时候Pod的IP地址是变化的。能够有一个固定的IP，使得集群内能够访问。也就是之前在架构描述的时候所提到的，能够把相同或者具有关联的Pod，打上Label，组成Service。而Service有固定的IP，不管Pod怎么创建和销毁，都可以通过Service的IP进行访问

* [Service官网](https://kubernetes.io/docs/concepts/services-networking/service/)

  ```shell
  An abstract way to expose an application running on a set of Pods as a network service.
  With Kubernetes you don’t need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.
  ```

1. 创建whoami-deployment.yaml文件，并且apply

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: whoami-deployment
     labels:
       app: whoami
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: whoami
     template:
       metadata:
         labels:
           app: whoami
       spec:
         containers:
         - name: whoami
           image: jwilder/whoami
           ports:
           - containerPort: 8000
   ```

2. 查看pod以及service

   ```shell
   whoami-deployment-5dd9ff5fd8-22k9n   192.168.221.80   worker02-kubeadm-k8s
   whoami-deployment-5dd9ff5fd8-vbwzp   192.168.14.6     worker01-kubeadm-k8s
   whoami-deployment-5dd9ff5fd8-zzf4d   192.168.14.7     worker01-kubeadm-k8s
   ```

* kubect get svc:可以发现目前并没有关于whoami的service

  ```shell
  NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19h
  ```

3. 在集群内正常访问

   `curl 192.168.221.80:8000/192.168.14.6:8000/192.168.14.7:8000`

4. 创建whoami的service

   `注意`：该地址只能在集群内部访问

   ```shell
   kubectl expose deployment whoami-deployment
   kubectl get svc    
   删除svc   kubectl delete service whoami-deployment
   
   [root@master-kubeadm-k8s ~]# kubectl get svc
   NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
   kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP    19h
   whoami-deployment   ClusterIP   10.105.147.59   <none>        8000/TCP   23s
   ```

* **可以发现有一个Cluster IP类型的service，名称为whoami-deployment，IP地址为10.101.201.192**

5. 通过Service的Cluster IP访问

   ```shell
   [root@master-kubeadm-k8s ~]# curl 10.105.147.59:8000
   I'm whoami-deployment-678b64444d-b2695
   [root@master-kubeadm-k8s ~]# curl 10.105.147.59:8000
   I'm whoami-deployment-678b64444d-hgdrk
   [root@master-kubeadm-k8s ~]# curl 10.105.147.59:8000
   I'm whoami-deployment-678b64444d-65t88
   ```

6. 具体查看一下whoami-deployment的详情信息，发现有一个Endpoints连接了具体3个Pod

   ```shell
   [root@master-kubeadm-k8s ~]# kubectl describe svc whoami-deployment
   Name:              whoami-deployment
   Namespace:         default
   Labels:            app=whoami
   Annotations:       <none>
   Selector:          app=whoami
   Type:              ClusterIP
   IP:                10.105.147.59
   Port:              <unset>  8000/TCP
   TargetPort:        8000/TCP
   Endpoints:         192.168.14.8:8000,192.168.221.81:8000,192.168.221.82:8000
   Session Affinity:  None
   Events:            <none>
   ```

7. 不妨对whoami扩容成5个

   ```shell
   kubectl scale deployment whoami-deployment --replicas=5
   ```

8. 再次访问：curl 10.105.147.59:8000

9. 再次查看service具体信息：kubectl describe svc whoami-deployment

10. 其实对于Service的创建，不仅仅可以使用kubectl expose，也可以定义一个yaml文件

    ```shell
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: MyApp
      ports:
        - protocol: TCP
          port: 80
          targetPort: 9376
      type: Cluster
    ```

    * `conclusion`：其实Service存在的意义就是为了Pod的不稳定性，而上述探讨的就是关于Service的一种类型Cluster IP，只能供集群内访问

    * 以Pod为中心，已经讨论了关于集群内的通信方式，接下来就是探讨集群中的Pod访问外部服务，以及外部服务访问集群中的Pod

## Pod访问外部服务

* 比较简单，没太多好说的内容，直接访问即可

## 外部服务访问集群中的Pod

![image-20191201215811446](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191201215811446.png)

### Service-NodePort

> 也是Service的一种类型，可以通过NodePort的方式
>
> 说白了，因为外部能够访问到集群的物理机器IP，所以就是在集群中每台物理机器上暴露一个相同的IP，比如32008

1. 根据whoami-deployment.yaml创建pod

   ```shell
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: whoami-deployment
     labels:
       app: whoami
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: whoami
     template:
       metadata:
         labels:
           app: whoami
       spec:
         containers:
         - name: whoami
           image: jwilder/whoami
           ports:
           - containerPort: 8000
   ```

2. 创建NodePort类型的service，名称为whoami-deployment

   ```shell
   kubectl delete svc whoami-deployment
   
   kubectl expose deployment whoami-deployment --type=NodePort
   
   [root@master-kubeadm-k8s ~]# kubectl get svc
   NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          21h
   whoami-deployment   NodePort    10.99.108.82   <none>        8000:32041/TCP   7s
   ```

3. 注意上述的端口32041，实际上就是暴露在集群中物理机器上的端口

   ```shell
   lsof -i tcp:32041
   netstat -ntlp|grep 32041
   ```

4. 浏览器通过物理机器的IP访问

   ```shell
   http://192.168.0.51:32041
   curl 192.168.0.61:32041
   ```

* `conclusion`：NodePort虽然能够实现外部访问Pod的需求，但是真的好吗？其实不好，占用了各个物理主机上的端口

### Service-LoadBalance

* 通常需要第三方云提供商支持，有约束性

### ==Ingress==

* 5.2.6还需再看

![image-20191202205205603](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191202205205603.png)

* [官网](<https://kubernetes.io/docs/concepts/services-networking/ingress/>)

  ```shell
  An API object that manages external access to the services in a cluster, typically HTTP.
  Ingress can provide load balancing, SSL termination and name-based virtual hosting.
  ```

* 可以发现，Ingress就是帮助我们访问集群内的服务的。不过在看Ingress之前，我们还是先以一个案例出发。

  很简单，在K8S集群中部署tomcat

* 浏览器想要访问这个tomcat，也就是外部要访问该tomcat，用之前的Service-NodePort的方式是可以的，比如暴露一个32008端口，只需要访问192.168.0.61:32008即可

```shell
vi my-tomcat.yaml
kubectl apply -f my-tomcat.yaml
kubectl get pods
kubectl get deployment
kubectl get svc
```

* tomcat-service   NodePort    10.105.51.97   <none>        80:31032/TCP   37s

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  labels:
    app: tomcat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  ports:
  - port: 80   
    protocol: TCP
    targetPort: 8080
  selector:
    app: tomcat
  type: NodePort
```

> 显然，Service-NodePort的方式生产环境不推荐使用，那接下来就基于上述需求，使用Ingress实现访问tomcat的需求
>
> `官网Ingress`:<https://kubernetes.io/docs/concepts/services-networking/ingress/>
>
> `GitHub Ingress Nginx`:<https://github.com/kubernetes/ingress-nginx>
>
> `Nginx Ingress Controller`:<https://kubernetes.github.io/ingress-nginx/

1. 以Deployment方式创建Pod，该Pod为Ingress Nginx Controller，要想让外界访问，可以通过Service的NodePort或者HostPort方式，这里选择HostPort，比如指定worker01运行

   ```shell
   # 确保nginx-controller运行到w1节点上
   kubectl label node w1 name=ingress   
   # 使用HostPort方式运行，需要增加配置
   hostNetwork: true
   # 搜索nodeSelector，并且要确保w1节点上的80和443端口没有被占用，镜像拉取需要较长的时间，这块注意一下哦
   # mandatory.yaml在网盘中的“课堂源码”目录
   kubectl apply -f mandatory.yaml  
   kubectl get all -n ingress-nginx
   ```

2. 查看**w1**的80和443端口

   ```shell
   lsof -i tcp:80
   lsof -i tcp:443
   ```

3. 创建tomcat的pod和service

   ```shell
   # 记得将之前的tomcat删除：kubectl delete -f my-tomcat.yaml
   vi tomcat.yaml
   kubectl apply -f tomcat.yaml
   kubectl get svc 
   kubectl get pods
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: tomcat-deployment
     labels:
       app: tomcat
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: tomcat
     template:
       metadata:
         labels:
           app: tomcat
       spec:
         containers:
         - name: tomcat
           image: tomcat
           ports:
           - containerPort: 8080
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: tomcat-service
   spec:
     ports:
     - port: 80   
       protocol: TCP
       targetPort: 8080
     selector:
       app: tomcat
   ```

4. 创建Ingress以及定义转发规则

   ```shell
   kubectl apply -f nginx-ingress.yaml
   kubectl get ingress
   kubectl describe ingress nginx-ingress
   ```

   ```yaml
   #ingress
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: nginx-ingress
   spec:
     rules:
     - host: tomcat.jack.com
       http:
         paths:
         - path: /
           backend:
             serviceName: tomcat-service
             servicePort: 80
   ```

5. 修改win的hosts文件，添加dns解析

   `192.168.8.61 tomcat.jack.com`

6. 打开浏览器，访问tomcat.jack.com

* `总结`：如果以后想要使用Ingress网络，其实只要定义ingress，service和pod即可，前提是要保证nginx ingress controller已经配置好了

# 服务部署到Kubernetes

## 部署wordpress+mysql

1. 创建wordpress命名空间`kubectl create namespace wordpress`

2. 创建wordpress-db.yaml文件`网盘/Kubernetes实战走起/课堂源码/wordpress-db.yaml`

   ```yaml
   apiVersion: apps/v1beta1
   kind: Deployment
   metadata:
     name: wordpress-deploy
     namespace: wordpress
     labels:
       app: wordpress
   spec:
     template:
       metadata:
         labels:
           app: wordpress
       spec:
         containers:
         - name: wordpress
           image: wordpress
           imagePullPolicy: IfNotPresent
           ports:
           - containerPort: 80
             name: wdport
           env:
           - name: WORDPRESS_DB_HOST
             value: 192.168.80.223:3306                     
           - name: WORDPRESS_DB_USER
             value: wordpress
           - name: WORDPRESS_DB_PASSWORD
             value: wordpress
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: wordpress
     namespace: wordpress
   spec:
     type: NodePort
     selector:
       app: wordpress
     ports:
     - name: wordpressport
       protocol: TCP
       port: 80
       targetPort: wdport
   ```

3. 根据wordpress-db.yaml创建资源[mysql数据库]

   ```shell
   kubectl apply -f wordpress-db.yaml
   kubectl get pods -n wordpress      # 记得获取ip，因为wordpress.yaml文件中要修改
   kubectl get pods -n wordpress -o wide
   # 如果不是running，则查看详情
   # kubectl describe pod wordpress-deploy-77cb4cd447-p596m -n wordpress
   
   kubectl get svc mysql -n wordpress
   kubectl describe svc mysql -n wordpress
   ```

4. 创建wordpress.yaml文件

   * `网盘/Kubernetes实战走起/课堂源码/wordpress.yaml`

5. 根据wordpress.yaml创建资源[wordpress]

   ```shell
   kubectl apply -f wordpress.yaml    #修改其中mysql的ip地址,其实也可以使用service的name:mysql
   kubectl get pods -n wordpress 
   kubectl get svc -n wordpress   # 获取到转发后的端口，如30063
   ```

6. 访问测试

   * win上访问集群中任意宿主机节点的IP:30063

7. 在集群中，发现可以通过service name进行访问，不仅仅是ip，那就说明service不仅帮我们做了负载均衡，而且做了dns

## 部署Spring Boot项目

* `流程`：确定服务-->编写Dockerfile制作镜像-->上传镜像到仓库-->编写K8S文件-->创建
* `网盘/Kubernetes实战走起/课堂源码/springboot-demo`

1. 准备Spring Boot项目springboot-demo

   ```java
   @RestController
   public class K8SController {
       @RequestMapping("/k8s")
       public String k8s(){
           return "hello K8s!";
       }
   }
   ```

2. 生成xxx.jar，并且上传到springboot-demo目录

   `mvn clean pakcage`

3. 编写Dockerfile文件

   > mkdir springboot-demo
   >
   > cd springboot-demo
   >
   > vi Dockerfile

   ```shell
   FROM openjdk:8-jre-alpine
   COPY springboot-demo-0.0.1-SNAPSHOT.jar /springboot-demo.jar
   ENTRYPOINT ["java","-jar","/springboot-demo.jar"]
   ```

4. 根据Dockerfile创建image

   `docker build -t springboot-demo-image .`

5. 使用docker run创建container

   `docker run -d --name s1 springboot-demo-image`

6. 访问测试

   ```shell
   docker inspect s1
   curl ip:8080/k8s
   ```

7. 将镜像推送到镜像仓库

   ```shell
   # 登录阿里云镜像仓库
   docker login --username=itcrazy2016@163.com registry.cn-hangzhou.aliyuncs.com
   
   docker tag springboot-demo-image registry.cn-hangzhou.aliyuncs.com/itcrazy2016/springboot-demo-image:v1.0
   
   docker push registry.cn-hangzhou.aliyuncs.com/itcrazy2016/springboot-demo-image:v1.0
   ```

8. 编写Kubernetes配置文件

   ```shell
   vi springboot-demo.yaml
   kubectl apply -f springboot-demo.yaml
   ```

   ```shell
   # 以Deployment部署Pod
   apiVersion: apps/v1
   kind: Deployment
   metadata: 
     name: springboot-demo
   spec: 
     selector: 
       matchLabels: 
         app: springboot-demo
     replicas: 1
     template: 
       metadata:
         labels: 
           app: springboot-demo
       spec: 
         containers: 
         - name: springboot-demo
           image: registry.cn-hangzhou.aliyuncs.com/itcrazy2016/springboot-demo-image:v1.0
           ports: 
           - containerPort: 8080
   ---
   # 创建Pod的Service
   apiVersion: v1
   kind: Service
   metadata: 
     name: springboot-demo
   spec: 
     ports: 
     - port: 80
       protocol: TCP
       targetPort: 8080
     selector: 
       app: springboot-demo
   ---
   # 创建Ingress，定义访问规则，一定要记得提前创建好nginx ingress controller
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata: 
     name: springboot-demo
   spec: 
     rules: 
     - host: k8s.demo.gper.club
       http: 
         paths: 
         - path: /
           backend: 
             serviceName: springboot-demo
             servicePort: 80
   ```

9. 查看资源

   ```shell
   kubectl get pods
   kubectl get pods -o wide
   curl pod_id:8080/k8s
   kubectl get svc
   kubectl scale deploy springboot-demo --replicas=5
   ```

10. win配置hosts文件[一定要记得提前创建好nginx ingress controller]

    `192.168.0.61 springboot.jack.com`

11. win浏览器访问

    > http://springboot.jack.com/k8s

## 部署Nacos项目

### 传统方式

1. 准备两个Spring Boot项目，名称为user和order，表示两个服务

   > `网盘/Kubernetes实战走起/课堂源码/user`
   >
   > `网盘/Kubernetes实战走起/课堂源码/order`

2. 下载部署nacos server1.0.0

   > `github`：<https://github.com/alibaba/nacos/releases>
   >
   > ·网盘/Kubernetes实战走起/课堂源码/nacos-server-1.0.0.tar.gz·

   ```shell
   #01  上传nacos-server-1.0.0.tar.gz到阿里云服务器39:/usr/local/nacos
   #02  解压：tar -zxvf
   #03  进入到bin目录执行：sh startup.sh -m standalone  [需要有java环境的支持]
   #04  浏览器访问：39.100.39.63:8848/nacos
   #05  用户名和密码：nacos
   ```

3. 将应用注册到nacos，记得修改Spring Boot项目中application.yml文件

   ```shell
   #01 将user/order服务注册到nacos
   #02 user服务能够找到order服务
   ```

4. 启动两个Spring Boot项目，然后查看nacos server的服务列表
5. 为了验证user能够发现order的地址

* 访问localhost:8080/user/test，查看日志输出，从而测试是否可以ping通order地址

### K8s方式

* user和order是K8s中的Pod

  * `思考`：如果将user和order都迁移到K8s中，那服务注册与发现会有问题吗？

  1. 生成xxx.jar，并且分别上传到master节点的user和order目录

     `resources/nacos/jar/xxx.jar`

     `mvn clean pakcage`

  2. 来到对应的目录，编写Dockerfile文件

     `vi Dockerfile`

     ```shell
     FROM openjdk:8-jre-alpine
     COPY user-0.0.1-SNAPSHOT.jar /user.jar
     ENTRYPOINT ["java","-jar","/user.jar"]
     ```

     ```shell
     FROM openjdk:8-jre-alpine
     COPY order-0.0.1-SNAPSHOT.jar /order.jar
     ENTRYPOINT ["java","-jar","/order.jar"]
     ```

  3. 根据Dockerfile创建image

     ```shell
     docker build -t user-image:v1.0 .
     docker build -t order-image:v1.0 .
     ```

  4. 将镜像推送到镜像仓库

     ```shell
     # 登录阿里云镜像仓库
     docker login --username=itcrazy2016@163.com registry.cn-hangzhou.aliyuncs.com
     
     docker tag user-image:v1.0 registry.cn-hangzhou.aliyuncs.com/itcrazy2016/user-image:v1.0
     
     docker push registry.cn-hangzhou.aliyuncs.com/itcrazy2016/user-image:v1.0
     ```

  5. 编写Kubernetes配置文件

     ```shell
     vi user.yaml/order.yaml
     kubectl apply -f user.yaml/order.yaml
     ```

     ```yaml
     # 以Deployment部署Pod
     apiVersion: apps/v1
     kind: Deployment
     metadata: 
       name: user
     spec: 
       selector: 
         matchLabels: 
           app: user
       replicas: 1
       template: 
         metadata:
           labels: 
             app: user
         spec: 
           containers: 
           - name: user
             image: registry.cn-hangzhou.aliyuncs.com/itcrazy2016/user-image:v1.0
             ports: 
             - containerPort: 8080
     ---
     # 创建Pod的Service
     apiVersion: v1
     kind: Service
     metadata: 
       name: user
     spec: 
       ports: 
       - port: 80
         protocol: TCP
         targetPort: 8080
       selector: 
         app: user
     ---
     # 创建Ingress，定义访问规则，一定要记得提前创建好nginx ingress controller
     apiVersion: extensions/v1beta1
     kind: Ingress
     metadata: 
       name: user
     spec: 
       rules: 
       - host: k8s.demo.gper.club
         http: 
           paths: 
           - path: /
             backend: 
               serviceName: user
               servicePort: 80
     ```

     ```yaml
     # 以Deployment部署Pod
     apiVersion: apps/v1
     kind: Deployment
     metadata: 
       name: order
     spec: 
       selector: 
         matchLabels: 
           app: order
       replicas: 1
       template: 
         metadata:
           labels: 
             app: order
         spec: 
           containers: 
           - name: order
             image: registry.cn-hangzhou.aliyuncs.com/itcrazy2016/order-image:v1.0
             ports: 
             - containerPort: 9090
     ---
     # 创建Pod的Service
     apiVersion: v1
     kind: Service
     metadata: 
       name: order
     spec: 
       ports: 
       - port: 80
         protocol: TCP
         targetPort: 9090
       selector: 
         app: order
     ```

  6. 查看资源

     ```shell
     kubectl get pods
     kubectl get pods -o wide
     kubectl get svc
     kubectl get ingress
     ```

  7. 查看nacos server上的服务信息

     `可以发现，注册到nacos server上的服务ip地址为pod的ip，比如192.168.80.206/192.168.190.82`

  8. 访问测试

     ```shell
     # 01 集群内
     curl user-pod-ip:8080/user/test
     kubectl logs -f <pod-name> -c <container-name>   [主要是为了看日志输出，证明user能否访问order]
     
     # 02 集群外，比如win的浏览器，可以把集群中原来的ingress删除掉
     http://k8s.demo.gper.club/user/test
     ```

     * **结论**：如果服务都是在K8s集群中，最终将pod ip注册到了nacos server，那么最终服务通过pod ip发现so easy

* user传统和order迁移K8s

  * 假如user现在不在K8s集群中，order在K8s集群中
  * 比如user使用本地idea中的，order使用上面K8s中的

  1. 启动本地idea中的user服务

  2. 查看nacos server中的user服务列表

  3. 访问本地的localhost:8080/user/test，并且观察idea中的日志打印，发现访问的是order的pod id，此时肯定是不能进行服务调用的，怎么解决呢？

  4. 解决思路

     ```shell
     之所以访问不了，是因为order的pod ip在外界访问不了，怎么解决呢？
     01 可以将pod启动时所在的宿主机的ip写到容器中，也就是pod id和宿主机ip有一个对应关系
     02 pod和宿主机使用host网络模式，也就是pod直接用宿主机的ip，但是如果服务高可用会有端口冲突问题[可以使用pod的调度策略，尽可能在高可用的情况下，不会将pod调度在同一个worker中]
     ```

  5. 我们来演示一个host网络模式的方式，修改order.yaml文件

     * 修改之后apply之前可以看一下各个节点的9090端口是否被占用

     * lsof -i tcp:9090

       ```shell
        ...
        metadata:
             labels: 
               app: order
           spec: 
           # 主要是加上这句话，注意在order.yaml的位置
             hostNetwork: true
             containers: 
             - name: order
               image: registry.cn-hangzhou
       ...
       ```

  6. kubectl apply -f order.yaml 

     * kubectl get pods -o wide   --->找到pod运行在哪个机器上，比如w2
     * 查看w2上的9090端口是否启动

  7. 查看nacos server上order服务

     * 可以发现此时用的是w2宿主机的9090端口

  8. 本地idea访问测试

     > localhost:8080/user/test

  











