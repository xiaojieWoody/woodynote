# Docker的持久化存储和数据共享

![image-20200630233022435](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630233022435.png)

![image-20200630233103990](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630233103990.png)

* Docker持久化数据的方案
  * 基于本地文件系统的Volume

![image-20200630233128533](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630233128533.png)

![image-20200630233210400](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630233210400.png)

## Data Volume

![image-20200630233657547](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630233657547.png)

![image-20200630233757449](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630233757449.png)

* 容器退出删除后，volume还在，名字太长（可以起别名）

![image-20200630234010089](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234010089.png)

![image-20200630234030927](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234030927.png)

![image-20200630234234921](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234234921.png)

## Bind Mouting

![image-20200630234332435](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234332435.png)

* 映射，本地目录做修改，容器里对应的目录也会做修改

  ```shell
  [root@woody demo_volume]# vim index.html
  <!doctype html>
  <html lang-"en">
  	<head>
  		<meta charset="utf-8">
  		<title>hello</title>
  	</head>
  	<body>
  		<h1>Hello Docker! Hi Docker</h1>
  	</body>
  </html>
  [root@woody demo_volume]# docker run -d -p 80:80 -v $(pwd):/usr/share/nginx/html --name web_v nginx
  ```

  ![image-20200630234649308](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234649308.png)

  ![image-20200630234725831](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234725831.png)

  ![image-20200630234847161](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630234847161.png)

* skeleton

  ![image-20200630235722127](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630235722127.png)

  ![image-20200630235557158](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630235557158.png)

  ![image-20200630235817147](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630235817147.png)

  ![image-20200630235855645](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630235855645.png)

  ![image-20200630235916791](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200630235916791.png)

  ![image-20200701000023223](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000023223.png)

  ![image-20200701000106842](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000106842.png)

  * 修改源码后，刷新页面会立马生效

# Docker Compose多容器部署

* 适合本地开发测试，不适合生产

![image-20200701000531722](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000531722.png)

![image-20200701000627176](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000627176.png)

![image-20200701000654622](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000654622.png)

![image-20200701000821651](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000821651.png)

![image-20200701000836337](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000836337.png)

![image-20200701000912310](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000912310.png)

![image-20200701000933133](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701000933133.png)

![image-20200701001129574](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701001129574.png)

![image-20200701001214871](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701001214871.png)

![image-20200701001321655](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701001321655.png)

![image-20200701001358140](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701001358140.png)

![image-20200701001427526](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701001427526.png)

```yaml
# docker-compose.yml
version: '3'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    depends_on:
      - mysql
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge
```

* 按照官网安装docker-compose

```shell
# 自动查找当前目录下的docker-compose.yml文件
docker-compose up
# docker-compose -f docker-compose up
```

![image-20200701002328539](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701002328539.png)

![image-20200701002432621](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701002432621.png)

![image-20200701002519410](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701002519410.png)

![image-20200701002651406](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701002651406.png)

```python
# app.py
from flask import Flask
from redis import Redis
import os
import socket

app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)


@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

```dockerfile
# Dockerfile
FROM python:2.7
LABEL maintaner="Peng Xiao xiaoquwl@gmail.com"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 5000
CMD [ "python", "app.py" ]
```

```yaml
# docker-compose.yml
version: "3"

services:

  redis:
    image: redis

  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 8080:5000
    environment:
      REDIS_HOST: redis
```

```shell
docker-compose up -d
# 浏览器访问
http://127.0.0.1:8080
```

* 负载均衡

  ```yaml
  # docker-compose.yml
  version: "3"
  
  services:
  
    redis:
      image: redis
  
    web:
      build:
        context: .
        dockerfile: Dockerfile
      # 删除ports
      environment:
        REDIS_HOST: redis
  ```

  ![image-20200701003434297](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701003434297.png)

  ![image-20200701003501587](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701003501587.png)

  ![image-20200701003520448](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701003520448.png)

  ```python
  # app.py
  from flask import Flask
  from redis import Redis
  import os
  import socket
  
  app = Flask(__name__)
  redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)
  
  
  @app.route('/')
  def hello():
      redis.incr('hits')
      return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())
  
  
  if __name__ == "__main__":
      app.run(host="0.0.0.0", port=80, debug=True)
  ```

  ```dockerfile
  # Dockerfile
  FROM python:2.7
  LABEL maintaner="Peng Xiao xiaoquwl@gmail.com"
  COPY . /app
  WORKDIR /app
  RUN pip install flask redis
  EXPOSE 80
  CMD [ "python", "app.py" ]
  ```

  ```yaml
  # docker-compose.yml
  version: "3"
  
  services:
  
    redis:
      image: redis
  
    web:
      build:
        context: .
        dockerfile: Dockerfile
      ports: ["8080"]
      environment:
        REDIS_HOST: redis
  
    lb:
      image: dockercloud/haproxy
      links:
        - web
      ports:
        - 80:80
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock 
  ```

  ![image-20200701003835264](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701003835264.png)

  ![image-20200701003946128](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701003946128.png)

* 部署一个复杂的投票应用

  ![image-20200701004141849](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701004141849.png)

  ```yaml
  # docker-compose.yml
  version: "3"
  
  services:
    voting-app:
      build: ./voting-app/.
      volumes:
       - ./voting-app:/app
      ports:
        - "5000:80"
      links:
        - redis
      networks:
        - front-tier
        - back-tier
  
    result-app:
      build: ./result-app/.
      volumes:
        - ./result-app:/app
      ports:
        - "5001:80"
      links:
        - db
      networks:
        - front-tier
        - back-tier
  
    worker:
      build: ./worker
      links:
        - db
        - redis
      networks:
        - back-tier
  
    redis:
      image: redis
      ports: ["6379"]
      networks:
        - back-tier
  
    db:
      image: postgres:9.4
      volumes:
        - "db-data:/var/lib/postgresql/data"
      networks:
        - back-tier
  
  volumes:
    db-data:
  
  networks:
  	# 默认，bridge类型
    front-tier:
    back-tier:
  ```

  ![image-20200701004545763](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701004545763.png)

  ![image-20200701004603355](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701004603355.png)

  * 可以先docker-compose build 再 docker-compose up

# 容器编排Docker Swarm

# DevOps初体验-Docker Cloud 和Docker企业版

# 容器编排Kubernetes

![image-20200701075534340](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701075534340.png)

![image-20200701075637961](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701075637961.png)

![image-20200701075701937](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701075701937.png)

![image-20200701075805425](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701075805425.png)

![image-20200701075827025](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701075827025.png)

* Pod：具有相同namespaces的容器的组合

  ![image-20200701080117479](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701080117479.png)

  ```shell
  # 先用浏览器访问显示最新稳定版的版本号
  https://storage.googleapis.com/kubernetes-release/release/stable.txt
  # 使用那个版本号下载kubectl：
  wget "https://storage.googleapis.com/kubernetes-release/release/v1.15.1/bin/linux/amd64/kubectl" -O "/usr/local/bin/kubectl"
  # 或使用浏览器下载，然后上传到服务器
  cp kubectl /usr/local/bin/
  chmod +x /usr/local/bin/kubectl
  curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
  # 启动minikube，启动过程中会下载kubeadm、kubelet和启动过程下载的东西
  minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com
  # 检验是否能用
  kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
  kubectl get pod
  
  https://0eipo2bm.mirror.aliyuncs.com
  
  minikube start --image-mirror-country cn \
      --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.6.0.iso \
      --registry-mirror=https://0eipo2bm.mirror.aliyuncs.com \
      --vm-driver=virtualbox
  ```

  ```shell
  # kubectl
  curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
  chmod +x ./kubectl
  sudo mv ./kubectl /usr/local/bin/kubectl
  kubectl version --client
  
  
  kubectl config view
  kubectl config get-contexts
  kubectl cluster-info
  minikube ssh
  kubectl exec -it podId sh
  
  
  
  kubectl describe pod nginx-pod
  
  # pod端口映射到宿主机上
  kubectl port-forward nginx-pod 8080:80
  ```

  

  ## 安装

  * Minikube，单节点

    * github上
    * 安装kubectl

    ![image-20200701080651940](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701080651940.png)

    ![image-20200701080843341](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701080843341.png)

    ![image-20200701080931790](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701080931790.png)

    ![image-20200701081010665](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081010665.png)

    ![image-20200701081045176](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081045176.png)

  ## Pod

  * 包含多个容器，共享一个namespace，包括网络、用户、存储等

  ![image-20200701081134853](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081134853.png)

  ![image-20200701081409900](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081409900.png)

  ![image-20200701081508487](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081508487.png)

  ![image-20200701081617707](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081617707.png)

  ![image-20200701081708622](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081708622.png)

* ![image-20200701081814811](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081814811.png)![image-20200701081746629](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701081746629.png)

  * 默认进入到第一个容器中，可通过-c参数指定进入的容器

  ![image-20200701082117588](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701082117588.png)

  ![image-20200701082255286](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701082255286.png)

  ![image-20200701082444712](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701082444712.png)

  * ip a ，minikube的ip为192.168.99.100，外部可以ping通minikube的ip

  ![image-20200701082702099](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701082702099.png)

  * 把pod端口映射到本地端口

    * 方式一：界面不能中断，否则映射停止

    ![image-20200701082846906](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200701082846906.png)
    
    * 方式二：？？？？待查询资料

# 容器的运维和监控

# Docker + DevOps实战-过程和工具

# 总结