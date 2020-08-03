```shell
kubectl delete pods podName
kubectl create -f rc_nginx.yml
kubectl get rc
# kubectl scale rc nginx --relicas=2
kubectl scale --replicas=2 -f rc_nginx.yml
kubectl get pods -o wide

```



* ReplicationController

  ```yml
  # rc_nginx.yml
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

* ReplicaSet

  ```yml
  # rs_nginx.yml
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
  ```

  ```shel
  kubectl create -f rs_nginx.yml
  kubectl scale --replicas=2 -f rs_nginx.yml
  kubectl get pods -o wide
  kubectl delete -f rs_nginx.yml
  ```

  

![image-20200701083346103](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083346103.png)

![image-20200701083440670](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083440670.png)

![image-20200701083516048](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083516048.png)

![image-20200701083544999](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083544999.png)

![image-20200701083644623](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083644623.png)

![image-20200701083758715](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083758715.png)

![image-20200701083828136](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083828136.png)

* Deployment

![image-20200701211248218](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701211248218.png)

* deployment_nginx.yml

  ```yml
  # deployment_nginx.yml
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
          image: nginx:1.12.2
          ports:
          - containerPort: 80
  ```

  ```shell
  kubectl create -f depolyment_nginx.yml
  kubectl get deployment
  kubectl get pods
  kubectl get deployment -o wide
  kubectl set image deployment nginx-deployment nginx=nginx:1.13
  
  kubectl get node -o wide
  kubectl expose deployment nginx-deployment --type=NodePort
  kubectl delete services nginx-deployment
  kubectl get svc
  ```

  

![image-20200701215300191](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701215300191.png)

![image-20200701220843231](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701220843231.png)

![image-20200701221050619](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701221050619.png)

![image-20200701221222788](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701221222788.png)

![image-20200701222018397](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701222018397.png)

## Tectonic在本地搭建多节点K8S集群

## k8s基础网络Cluster Network

![image-20200701222937046](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701222937046.png)

* 两个pod可以互相ping的通，k8s的节点上可以ping通pod的ip（包括跨机器）

![image-20200701223349623](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701223349623.png)

![image-20200701223630479](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701223630479.png)

![image-20200701223747692](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701223747692.png)

![image-20200701223823498](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701223823498.png)

![image-20200701224014790](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701224014790.png)

* ClusterIP
  * cluster内部访问，外部无法访问
  * pod的ip和service的ip对应
  * 虽然pod的ip会变化，但是service对外提供的的IP不会变化
* NodePort
  * 绑定到node上，可以对外node提供访问
* LoadBalance

## ClusterIp

![image-20200701224844510](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701224844510.png)

![image-20200701224934496](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701224934496.png)

![image-20200701225034900](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701225034900.png)



* 9-7开始重看







# kubernetes基本概念和操作

```shell
[root@k8s-master ~]# cd .kube/
[root@k8s-master .kube]# ls
cache  config  http-cache
[root@k8s-master .kube]# more config
```

```shell
kubectl get node
kubectl describe node k8s-master
# -o 输出
kubectl get node -o wide
kubectl get node -o yaml
kubectl get node -o json



# 设置label
kubectl label node k8s-master env=test
kubectl get node --show-labels
# 删除label
kubectl label node k8s-master env-

# roles  ，特殊的lebel
kubectl get node
kubectl get node --show-labels
kubectl label node k8s-node1 node-role.kubernetes.io/worker=
kubectl label node k8s-node2 node-role.kubernetes.io/worker=
kubectl get node
```

​	

* Pod

  ![image-20200731205804251](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200731205804251.png)

  ![image-20200731205934440](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200731205934440.png)

  ```shell
  kubectl exec nginx-busybox -c nginx date
  kubectl exec nginx-busybox -c nginx hostname
  
  kubectl exec nginx-busybox -c busybox hostname
  
  kubectl exec nginx-busybox -it sh
  
  kubectl exec nginx-busybox -it -c busybox sh
  wget 127.0.0.1
  more index.html
  ```

  ![image-20200731211241695](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200731211241695.png)

  ```shell
  kubectl get namespace
  kubectl create namespace demo
  kubectl get pod -n demo
  
  kubectl get pod -A
  ```

  ![image-20200731211413924](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200731211413924.png)

  

```shell
 kubectl config get-contexts
 

 kubectl config view
 
 kubectl config set-context demo --user=kubernetes-admin --cluster=kubernetes --namespace=demo
 kubectl config view
  
 kubectl config use-context demo
 
 kubectl config delete-context demo
```

