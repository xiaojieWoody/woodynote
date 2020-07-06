* ReplicationController

![image-20200701083320931](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083320931.png)

![image-20200701083346103](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083346103.png)

![image-20200701083440670](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083440670.png)

![image-20200701083516048](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083516048.png)

![image-20200701083544999](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083544999.png)

![image-20200701083644623](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083644623.png)

![image-20200701083758715](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083758715.png)

![image-20200701083828136](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701083828136.png)

* ReplicationSet

![image-20200701084142354](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701084142354.png)

![image-20200701084216207](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701084216207.png)

![image-20200701084255393](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701084255393.png)

![image-20200701084320479](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701084320479.png)

* Deployment

![image-20200701211248218](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701211248218.png)

* deployment_nginx.yml

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