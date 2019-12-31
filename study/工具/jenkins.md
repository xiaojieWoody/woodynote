# 简介

* 开源、`Java`开发、持续集成工具

# 安装

* 需要`jdk1.8`环境
* `war`安装
  * 下载`war`包，启动：`java -jar jenkins.war`
  * 访问`localhost:8080`
  * `cat /Users/dingyuanjie/.jenkins/secrets/initialAdminPassword`
    * `ee8c6dd5f57d4e90b7a41567ff66be59`
* 通过`tomcat`启动
  * `war`扔到`tomcat`的`webapp`中
  * 启动`tomcat`，访问`localhost:8080/jenkins`

# 工作流程

![image-20191227140828073](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191227140828073.png)

* 把代码上传到Jenkins中让它来进行自动的build和deploy的操作

# 使用

* 下载好了Jenkins之后，安装，访问8080端口，管理的账号，密码自己去Jenkins的安装目录寻找
* 默认会安装一些插件，git，maven，github。如果没有安装成功，可进行插件的选择自定义安装
* 想要通过git拉取github上面的代码，本地安装了git
* 使用Jenkins拉取代码，本地的git需要配置成功
  * ssh  key  public key
  * ![image-20191229094325815](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229094325815.png)

* 全局工具配置-jdk、git

  ![image-20191227215105493](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191227215105493.png)

* mac先生成id_rsa

  ```shell
  ssh-keygen -t rsa -C "youremail@example.com"
  # 一路回车
  ls -a ~/.ssh
  ```

* 拉取代码

  ![image-20191229091725424](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229091725424.png)

  ![image-20191227212805279](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191227212805279.png)

* 没有拉取成功

  1. git的插件和git的全局管理工具是否已经配置【本地git环境有没有搭建成功】
  2. 安全性的考虑，配置一个Credential 证书（本地git的id_rsa私钥内容配置进去就不报错）

  ![image-20191227214112063](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191227214112063.png)

  ![image-20191229091826987](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229091826987.png)

* 没有报错

# 项目构建和发布(上)

1. 新建一个maven的web工程
2. 代码Push到了github上
3. Jenkins对其进行集成发布

![image-20191229092326534](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229092326534.png)

![image-20191229092348663](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229092348663.png)

* 这里可以指定针对不同的分支进行集成发布操作

4. Maven对其进行了build操作（pre-step，post-step）

   ![image-20191229100037979](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229100037979.png)

5. 项目进行一个发布tomcat容器

a. 有一个tomcat容器   本机  远程机器   url 

b. 有需要发布项目的war  maven----->xx.war   target---->xx.war

![image-20191229092557532](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229092557532.png)

c. 把前面的拉取代码和maven构建操作先执行一下  jenkins

# 项目构建和发布(下)

* gpjenkins放到了github

* Jenkins---->

  1. 代码从github上拉取下来

  2. 项目进行build  maven

     ![image-20191229092809259](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229092809259.png)

  3. 需要把这样一个war发布到tomcat上去

     ![image-20191229092847305](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229092847305.png)

  本地  127.0.0.1:8088   tomcat  但是出于安全性的一个考虑

  进行认证  sshkey

  A. 在tomcat中进行账号的管理

  ```xml
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <role rolename="admin-gui"/>
  <user username="gpjack" password="gpjack" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
  ```

  B. 需要告诉jenkins你的一个tomcat的账号——Creadentials

  ![image-20191229204106025](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229204106025.png)

  ![image-20191229204203637](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229204203637.png)

  持续交付与发布

  1. 拉取代码
  2. Maven build
  3. Deploy容器

  需要手动在Jenkins中点击立即构建的按钮

  ![image-20191229205245767](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229205245767.png)

  ![image-20191229205200830](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229205200830.png)

  自动

# Webhook的配置

* 本地：gpjenkins
* Github：gpjenkins

本地进行开发，add commit push ---->

本地的代码 push --- github上面

Jenkins ---> 立即构建(手动)

自动

Push ------>  触发jenkins的立即构建的操作

也就是github上面一旦收到push代码的请求----->触发立即构建的操作

![image-20191229205806405](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229205806405.png)

中间桥梁

Jenkins项目中有一个地址`http://localhost:8080/project/gpjenkins`

能不能再github上的gpjenkins项目的某个位置，配置上这个url，一旦有push操作的时候，我就希望它能够对应之前jenkins上的url进行触发立即构建

![image-20191229205949351](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229205949351.png)

![image-20191229210325204](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191229210325204.png)

外网和内网需要统一 

