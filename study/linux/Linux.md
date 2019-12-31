```shell
# 切换到root用户
sudo -i
# 能够通过ssh连接
vi /etc/ssh/sshd_config
将PasswordAuthentication 的属性 no 改为 yes
service sshd restart
# 查看ip
ip a
# 查看应用所在目录
which java
# Centos7下安装netstat
yum install net-tools
# 查看端口服务是否启动
netstat -anp |grep "3301"
# 查看端口起用情况
netstat -nltp |grep 50070
# 外部访问端口
 telnet 192.168.0.122 50070
# 查看主机名
hostname
# 修改主机名，重启后生效
vi /etc/hostname
# 查看端口
lsof -i tcp:8080
# 启动main方法
java -cp day01.jar day01.service.multiport.MultiPortServer
# 服务器间拷贝
[atguigu@hadoop101 /]$ scp -r /opt/module  root@hadoop102:/opt/module
```

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191125182303871.png" alt="image-20191125182303871" style="zoom:25%;" />

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191125232354162.png" alt="image-20191125232354162" style="zoom:50%;" />



### 搭建自己的gitlab

* [官网](https://about.gitlab.com/install/#centos-7 )

1. 说明：安装gitlab的机器至少要有4G的内存，因为gitlab比较消耗内存 

2. 安装必要的依赖

   ```shell
   sudo yum install -y curl policycoreutils-python openssh-server 
   sudo systemctl enable sshd
   sudo systemctl start sshd
   sudo firewall-cmd --permanent --add-service=http
   sudo systemctl reload firewalld
   ```

3. 如果想要发送邮件，就跑一下下面的内容 

   ```shell
   sudo yum install postfix
   sudo systemctl enable postfix
   sudo systemctl start postfix
   ```

4. 添加gitlab的仓库地址 

   ```shell
   https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh|sudo bash
   # 注意:这个下载仓库可能速度会很慢，此时可以用国内的仓库地址
   # 新建文件 /etc/yum.repos.d/gitlab-ce.repo 内容为
   [gitlab-ce]
   name=Gitlab CE Repository baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/ gpgcheck=0
   enabled=1
   ```

5. 设置gitlab的域名和安装gitlab 

   ```shell
   sudo EXTERNAL_URL="https://gitlab.itcrazy2016.com" yum install -y gitlab-ee
   # 如果用的是国内仓库地址，则执行以下命令，其实区别就是ee和ec版
   sudo EXTERNAL_URL="https://gitlab.itcrazy2016.com" yum install -y gitlab-ce
   # 此时要么买个域名，要么在本地的hosts文件中设置一下
   # 安装gitlab服务器的ip地址 gitlab.itcrazy2016.com
   # 假如不想设置域名，可以直接安装yum install -y gitlab-ee
   ```

6. 重新configure

   ```shell
   # 如果没有成功，可以运行gitlab-ctl reconfigure
   ```

7. 查看gitlab运行的情况

   ```shell
   # gitlab-ctl status可以看到运行gitlab服务所需要的进程
   ```

8. 访问

   * 浏览器输入gitlab.itcrazy2016.com，此时需要修改root账号的密码 

9. 配置已经安装好的gitlab 

   ```shell
   vim /etc/gitlab/gitlab.rb
   # 修改完成之后一定要gitlab-ctl reconfigure
   ```

### 安装docker

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io    
```

### Hadoop

```shell
# hadoop 50070 无法访问问题解决汇总
https://www.cnblogs.com/zlslch/p/6604189.html
vi /etc/selinux/config 
SELINUX=enforcing 改为 SELINUX=disabled
http://192.168.0.122:50070/dfshealth.html#tab-overview
```

### 安装MySQL

```shell
# 如果安装了则卸载
[root@hadoop102 桌面]# rpm -qa|grep mysql
[root@hadoop102 桌面]# rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64
# 解压
[root@hadoop102 software]# unzip mysql-libs.zip
# 安装MySQL服务端
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
# 查看产生的随机密码
[root@hadoop102 mysql-libs]# cat /root/.mysql_secret
OEXaQuS8IWkG19Xs
# 查看MySQL状态
[root@hadoop102 mysql-libs]# service mysql status
# 启动MySQL
[root@hadoop102 mysql-libs]# service mysql start
# 安装MySQL客户端
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
# 连接MySQL
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
# 修改密码
mysql>SET PASSWORD=PASSWORD('000000');
# 退出MySQL
mysql>exit
# 配置只要是root用户+密码，在任何主机上都能登录MySQL数据库
[root@hadoop102 mysql-libs]# mysql -uroot -p000000
mysql>show databases;
mysql>use mysql;
mysql>show tables;
mysql>desc user;
mysql>select User, Host, Password from user;
# 修改user表，把Host表内容修改为%
mysql>update user set host='%' where host='localhost';
# 删除root用户的其他host
mysql>delete from user where Host='hadoop102';
mysql>delete from user where Host='127.0.0.1';
mysql>delete from user where Host='::1';
# 刷新
mysql>flush privileges;
# 退出
mysql>quit;
```

### 配置JDK

```shell
#查询是否安装Java软件
[atguigu@hadoop101 opt]$ rpm -qa | grep java
# 如果安装的版本低于1.7，卸载该JDK
[atguigu@hadoop101 opt]$ sudo rpm -e 软件包
# 查看JDK安装路径
[atguigu@hadoop101 ~]$ which java
[root@hadoop101 software] # tar -zxf jdk-8u144-linux-x64.tar.gz -C /opt/module/
[root@hadoop101 software]# vi /etc/profile
#JAVA_HOME：
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
[root@hadoop101 software]#source /etc/profile
# 验证命令：java -version
```

### 配置Maven

```shell
[root@hadoop101 software]# tar -zxvf apache-maven-3.0.5-bin.tar.gz -C /opt/module/
[root@hadoop101 apache-maven-3.0.5]# vi conf/settings.xml
```

```xml
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>central</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

```shell
[root@hadoop101 apache-maven-3.0.5]# vi /etc/profile
#MAVEN_HOME
export MAVEN_HOME=/opt/module/apache-maven-3.0.5
export PATH=$PATH:$MAVEN_HOME/bin
[root@hadoop101 software]#source /etc/profile
```

### 配置ANT

```shell
[root@hadoop101 software]# tar -zxvf apache-ant-1.9.9-bin.tar.gz -C /opt/module/
[root@hadoop101 apache-ant-1.9.9]# vi /etc/profile
#ANT_HOME
export ANT_HOME=/opt/module/apache-ant-1.9.9
export PATH=$PATH:$ANT_HOME/bin
[root@hadoop101 software]#source /etc/profile
# 验证命令：ant -version
```

### 配置环境变量

```shell
echo $PATH
pwd
vi /etc/profile
	export JAVA_HOME=/opt/jdk
	export HADOOP_HOME=/usr/local/bgdt/hadoop-2.8.5
	export PATH=.:$PATH:$HADOOP_HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/sbin
source /etc/profile
# 例子
# 添加可执行权限
chmod +x helloworld.sh
# 当前目录 helloworld.sh就会直接执行，因为在环境变量中配置了在当前目录下查找并执行(.)
```

### 集群时间同步

* 时间同步的方式：找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，比如，每隔十分钟，同步一次时间

![image-20191127161635207](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191127161635207.png)

1. 时间服务器配置（必须root用户）

   ```shell
   # 检查ntp是否安装
   [root@hadoop102 桌面]# rpm -qa|grep ntp
   # 修改ntp配置文件
   [root@hadoop102 桌面]# vi /etc/ntp.conf
   # 修改1（授权192.168.1.0-192.168.1.255网段上的所有机器可以从这台机器上查询和同步时间）
   #restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap为
   restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
   # 修改2（集群在局域网中，不使用其他互联网上的时间）
   server 0.centos.pool.ntp.org iburst
   server 1.centos.pool.ntp.org iburst
   server 2.centos.pool.ntp.org iburst
   server 3.centos.pool.ntp.org iburst为
   #server 0.centos.pool.ntp.org iburst
   #server 1.centos.pool.ntp.org iburst
   #server 2.centos.pool.ntp.org iburst
   #server 3.centos.pool.ntp.org iburst
   # 添加3（当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步）
   server 127.127.1.0
   fudge 127.127.1.0 stratum 10
   # 修改/etc/sysconfig/ntpd 文件
   [root@hadoop102 桌面]# vim /etc/sysconfig/ntpd
   # 增加内容如下（让硬件时间与系统时间一起同步）
   SYNC_HWCLOCK=yes
   # 重新启动ntpd服务
   [root@hadoop102 桌面]# service ntpd status
   ntpd 已停
   [root@hadoop102 桌面]# service ntpd start
   正在启动 ntpd：
   # 设置ntpd服务开机启动
   [root@hadoop102 桌面]# chkconfig ntpd on
   ```

2. 其他机器配置（必须root用户）

   ```shell
   # 在其他机器配置10分钟与时间服务器同步一次
   [root@hadoop103桌面]# crontab -e
   # 编写定时任务如下：
   */10 * * * * /usr/sbin/ntpdate hadoop102
   # 修改任意机器时间
   [root@hadoop103桌面]# date -s "2017-9-11 11:11:11"
   # 十分钟后查看机器是否与时间服务器同步
   [root@hadoop103桌面]# date
   # 说明：测试的时候可以将10分钟调整为1分钟，节省时间。
   ```

   