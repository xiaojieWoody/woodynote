```shell
# 查看应用所在目录
which java
# Centos7下安装netstat
yum install net-tools
# 查看端口服务是否启动
netstat -anp |grep "3301"
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

### 配置JDK

```shell
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

   