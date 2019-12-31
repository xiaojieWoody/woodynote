# CICD

![image-20191202164310160](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191202164310160.png)

#环境准备

### 基础环境准备[在jenkins那台机器上安装] 

* 安装Java

  1. 找到jdk资源上传到指定机器 

     ```shell
     resources/cicd/jdk-8u181-linux-x64.tar.gz
     ```

  2. 配置环境变量

     ```shell
     vim /etc/profile
     
     export JAVA_HOME=/usr/local/java/jdk1.8.0_181
     export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
     export PATH=$PATH:${JAVA_HOME}/bin
     
     source /etc/profile
     java -version
     ```

* 安装maven

  1. 找到maven资源上传到指定机器 

     ```shell
     resources/cicd/apache-maven-3.6.2-bin.tar.gz
     ```

  2. 配置环境变量 

     ```shell
     vim /etc/profile
     export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.2
     export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
     source /etc/profile
     mvn -version
     ```

  3. 配置maven的阿里云镜像 

     ```xml
     <mirror>
         <id>alimaven</id>
         <name>aliyun maven</name>
         <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
         <mirrorOf>central</mirrorOf>
     </mirror>
     ```

* 安装配置git 

  ```shell
  # 1. 下载安装
  yum install git
  # 2. 配置git
  git config --global user.name "itcrazy2016"
  git config --global user.email "itcrazy2016@163.com"
  ssh-keygen -t rsa -C "itcrazy2016@163.com" --->将公钥上传到github:/root/.ssh/id_rsa.pub
  ```

### IDEA+Spring Boot项目 

```shell
# 01 下载项目
git clone git@github.com:itcrazy2016/springboot-demo.git
# 02 使用idea打开 此时项目已经和github关联
```

### Gitlab

* 直接采用github 
* git@github.com:itcrazy2016/springboot-demo.git 

### Jenkins 

```shell
# 必须在k8s集群中，因为后面需要在jenkins的目录下创建文件执行，比如这里选用w2
#操作前须知
jenkins官网 :https://jenkins.io/
入门指南 :<https://jenkins.io/zh/doc/pipeline/tour/getting-started
#1. 找到对应资源:resources/cicd/jenkins.war
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
#2. 启动jenkins[记得当前机器安装了jdk/jre，不然运行不了]
nohup java -jar jenkins.war --httpPort=8080 &
tail -f nohup.out
#3. win浏览器访问w2的ip 121.40.56.193:8080，记录下密码，比如
cat /root/.jenkins/secrets/initialAdminPassword
#4. 安装推荐的插件
Folders、OWASP Markup
#5. 创建一个用户，比如
username:jack
password:123456
#6. 安装配置git，maven
#7. 在jenkins上使用centos的java，git，maven等
[系统管理]->[全局工具配置]->[Maven、JDK、Git等]
```

![image-20191204143523412](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191204143523412.png)

### Docker hub









