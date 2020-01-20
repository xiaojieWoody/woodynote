# CDH集群搭建

* 机器资源
  * 可以搭建三台虚拟机，其中master内存在8G以上，slave内存在4G以上，每个虚机的硬盘空间100G+
* 软件版本
  * 操作系统：centos 6.8 64位
  * CDH：5.8.5，对应的大数据组件版本
    * `https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh_package_tarball_58.html`
  * jdk：1.7
* Cloudera Manager架构

![image-20200104153609147](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104153609147.png)

* 需下载的安装包
  * virtualbox：虚拟机 ，`https://www.virtualbox.org/wiki/Downloads`
  * centos镜像，`http://vault.centos.org/6.8/isos/x86_64/ `
  * parcel：所有大数据组件，以二进制方式打包在一个文件中 ，`http://archive.cloudera.com/cdh5/parcels/。注意下载的版本必须与操作系统版本一致，否则在安装的时候会重新下载对应的版本 `
  * jdk：oracle 64位 jdk，`http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/`
  * Clouder Manager相关文件，`http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/`

![image-20200104174656332](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104174656332.png)

## 虚拟机设置

1. 安装虚拟机

   * 建议：安装好一台后通过导入导出功能复制其他两台

2. 设置网络

   ![image-20200104154118373](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104154118373.png)

   ![image-20200104154140885](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104154140885.png)

   * https://jingyan.baidu.com/article/948f59242e601dd80ff5f929.html

3. 设置hosts(后续操作均以root用户执行)

   ```shell
   vi /etc/hosts:
   192.168.137.110	master
   192.168.137.111	slave01
   192.168.137.112 	slave02
   ```

4. 关闭SELinux及防火墙

   ```shell
   1.关闭SELinux:  
   	  vi /etc/selinux/config ，修改如下： 
   SELINUX=disabled
   
   2. 关闭防火墙：
   sudo service iptables stop
   sudo chkconfig iptables off
   sudo chkconfig iptables --list
   ```

5. 设置ssh免密登录

   ```shell
   1.生成密钥：
   ssh-keygen -t rsa（默认位于 ~/.ssh/）
   2. 拷贝公钥到所有机器：
   ssh-copy-id root@master
   ssh-copy-id root@slave01
   ssh-copy-id root@slave02
   3.测试免密登录：
   ssh master
   ssh slave01
   ssh slave02
   ```

6. 设置ntp时间同步服务

   ```shell
   1.安装 ntp
   yum –y install ntp 
   
   2.设置NTP服务开机启动
   chkconfig ntpd on
   ```

   ```shell
   将master设置为主服务器（在master节点操作）：
   1. vi /etc/ntp.conf，内容如下：
   driftfile /var/lib/ntp/ntp.drift #草稿文件
   # 允许内网其他机器同步时间
   restrict 192.168.137.0 mask 255.255.255.0 nomodify notrap
    
   # Use public servers from the pool.ntp.org project.
   # 中国这边最活跃的时间服务器 : [http://www.pool.ntp.org/zone/cn](http://www.pool.ntp.org/zone/cn)
   server 210.72.145.44 perfer   # 中国国家受时中心
   server 202.112.10.36             # 1.cn.pool.ntp.org
   server 59.124.196.83             # 0.asia.pool.ntp.org
    
   # allow update time by the upper server 
   # 允许上层时间服务器主动修改本机时间
   restrict 210.72.145.44 nomodify notrap noquery
   restrict 202.112.10.36 nomodify notrap noquery
   restrict 59.124.196.83 nomodify notrap noquery
    
   # 外部时间服务器不可用时，以本地时间作为时间服务
   server  127.127.1.0     # local clock
   fudge   127.127.1.0 stratum 10
   2. 重启服务： service ntpd restart
   3. 查看同步状态： netstat -tlunp | grep ntp
   ```

   ```shell
   设置slave到master 的同步（在slave节点操作）：
   1. vi /etc/ntp.conf，内容如下：
   driftfile /var/lib/ntp/ntp.drift # 草稿文件
   
   statsdir /var/log/ntpstats/
   statistics loopstats peerstats clockstats
   filegen loopstats file loopstats type day enable
   filegen peerstats file peerstats type day enable
   filegen clockstats file clockstats type day enable
   
   # 让NTP Server为内网的ntp服务器
   server 192.168.137.110
   fudge 192.168.137.110 stratum 5
   
   # 不允许来自公网上ipv4和ipv6客户端的访问
   restrict -4 default kod notrap nomodify nopeer noquery 
   restrict -6 default kod notrap nomodify nopeer noquery
   
   # Local users may interrogate the ntp server more closely.
   restrict 127.0.0.1
   restrict ::1
   2. 重启服务： service ntpd restart
   3. 手动同步： ntpdate -u 192.168.137.110
   ```

## CDH安装

7. 上传安装文件

   ![image-20200104154640724](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104154640724.png)

8. 安装jdk&CM

   ```shell
   1. 验证repo文件是否起效
   yum list | grep cloudera#如果列出的不是待安装的版本，执行下面命令重试yum clean allyum list | grep cloudera
   2. 切换到jdk&cm目录下，执行
   yum -y install *.rpm
   3. 设置java路径：
   vi /etc/profile
   # 在该文件末尾添加以下行
   JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera
   PATH=$JAVA_HOME/bin:$PATH
   export JAVA_HOME PATH
   4. 检查安装：
   java -version
   ```

9. 安装CDH（只在master节点）

   ```shell
   1.进入cloudera-manager-installer.bin文件目录，给bin文件赋予可执行权限：
   chmod +x ./cloudera-manager-installer.bin
   2. 运行：
   ./cloudera-manager-installer.bin --skip_repo_package=1
   ```

10. HDFS设置

    ```shell
    在CM console中将副本设为2：dfs.replication=2
    命令行执行：hadoop fs -setrep 2 /
    ```

## CDH启动与关闭

```shell
CM Portal 地址： 
	http://master:7180/cmf/home
关闭步骤：
	在CM portal上关闭 cluster
	在所有节点关闭CM agent： service cloudera-scm-agent stop
	在master节点关闭CM server： service cloudera-scm-server stop
启动步骤：
	在所有节点启动CM agent： service cloudera-scm-agent start
	在master节点启动CM server： service cloudera-scm-server start
	在CM portal上启动 cluster
查看启动日志：
 /var/log/cloudera-scm-server/cloudera-scm-server.log
```

# 3.1 版本集群搭建

![image-20200104182159446](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104182159446.png)

![image-20200104182258539](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104182258539.png)

![image-20200104182321706](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104182321706.png)

## 虚拟机设置（root用户）

```shell
# virtualbox中Linux选择Redhat 64，安装centos7，注册centos7的ios镜像
# virtualbox中安装好虚拟机后，如果没有点网络连接的话，那么网卡将不会启动启动
输入命令ip addr可以看到有两个网卡lo、enp0s3前者的地址是127.0.0.1，后者没有ip地址，这里需要手动配置网卡enp0s3自动启动，打开vi /etc/sysconfig/network-scripts/编辑文件名为ifcfg-enp0s3内容设置ONBOOT=yes，然后重启系统reboot，IP获取正常，可以访问网络了
# 设置网络为桥接模式
# 切换到桌面模式
startx
# 修改主机名
hostnamectl --static set-hostname qqmm
# 安装firefix
yum -y install firefox
# 查看ip
ip a
# 配置静态ip
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	BOOTRPOTO=static
	IPADDR=192.168.0.201
	NETMASK=255.255.255.0
	GATEWAY=192.168.0.1
service network restart

# centos7 ping 未知的名称或服务
对接口添加dns信息；编辑/etc/sysconfig/network-scripts/ifcfg-ethX，x可能是其他数字，但一般是ifcfg-eth0的，具体的X根据你的网卡确定，在最下面添加：
# vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
DNS1=8.8.8.8   #google dns服务器, 根据实际情况更换
DNS2=8.8.4.4   #google dns服务器, 根据实际情况更换
保存后重启网络
service network restart
```

```shell
1. 搭建虚拟机  master:2G, slave1:1G, slave2:1G
2. 确定hostname(master,slave1,slave2)
	vi /etc/sysconfig/network
3. 设置网络
4. 设置host: vi /etc/hosts
xxx master
xxx slave1
xxx slave2
5. 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service
# service iptables stop
#chkconfig iptables off

systemctl status firewalld
6. 关闭selinux
vi /etc/selinux/config
7. 配置ssh免密登录（运行hadoop的用户）  三台都这样操作
ssh-keygen -t rsa
ssh-copy-id george@master
ssh-copy-id george@slave1
ssh-copy-id george@slave2
# 验证
ssh slave2  # slave1上
```

* 设置静态ip

  ![image-20200104204758651](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104204758651.png)

  ![image-20200104204631428](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104204631428.png)

  ![image-20200104204915547](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104204915547.png)

  ![image-20200104204459087](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104204459087.png)

  ![image-20200104204542241](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104204542241.png)

![image-20200104183204361](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104183204361.png)

![image-20200104183122168](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104183122168.png)



## 安装jdk1.8（root用户）：

```shell
1.下载安装包
2.安装： rpm -ivh jdk-8u161-linux-x64.rpm
3.设置环境变量 JAVA_HOME: vi /etc/profile
	export JAVA_HOME=/usr/java/jdk1.8.0_161
	export PATH=$PATH:$JAVA_HOME/bin
4. source /etc/profile	
```

## 安装hadoop（运行hadoop的用户）：

```shell
1.下载安装包
2.解压安装包
3.创建目录： mkdir -p xxx
# root用户下给hadoop用户授权  chown -R hadoop /data
hdfs namenode目录： mkdir -p /data/hadoop/hdfs/namenode
hdfs datanode目录：mkdir -p /data/hadoop/hdfs/datanode
hdfs tmp文件目录：mkdir -p /data/hadoop/tmp
yarn nodemanger文件目录：mkdir -p /data/hadoop/yarn/nodemanager
yarn log文件目录：mkdir -p /data/hadoop/yarn/logs
mr 目录： mkdir -p /data/hadoop/mr
4.设置环境变量: HADOOP_HOME
export HADOOP_HOME=/home/george/software/hadoop-3.1.1
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
5.修改配置文件：HADOOP_HOME/etc/hadoop/
# /home/george/software/hadoop-3.1.1/etc/hadoop  覆盖
core-site.xml  
hdfs-site.xml  
yarn-site.xml
mapred-site.xml  
workers
hadoop-env.sh：JAVA_HOME
```

```shell
[root@slave2 /]# rm -rf data/
[root@slave2 /]# mkdir -p data
[root@slave2 /]# chown -R hadoop /data
[root@slave2 /]# exit
exit
[hadoop@slave2 software]$ mkdir -p /data/hadoop/hdfs/namenode
[hadoop@slave2 software]$ mkdir -p /data/hadoop/hdfs/datanode
[hadoop@slave2 software]$ mkdir -p /data/hadoop/tmp
[hadoop@slave2 software]$ mkdir -p /data/hadoop/yarn/nodemanager
[hadoop@slave2 software]$ mkdir -p /data/hadoop/yarn/logs
[hadoop@slave2 software]$ mkdir -p /data/hadoop/mr
```

## 启动（在 master 节点运行）:

```shell
# hadoop用户来启动
1.格式化：hdfs namenode -format
2.启动HDFS：start-dfs.sh       # 验证  每台机器上 jps 
# 停止 stop-dfs.sh
3.启动YARN：start-yarn.sh
# 停止 stop-yarn.sh
```

![image-20200106220241364](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200106220241364.png)

## 验证

```shell
hdfs：
	hdfs dfs -ls /
	hdfs dfs -mkdir /data/sample
mr:
	# /home/hadoop/software/hadoop-3.1.1/share/hadoop/mapreduce 目录下
	hadoop jar hadoop-mapreduce-examples-3.1.1.jar pi 5 10
	# http://master:8088/proxy/application_1578321804925_0001/
访问：
	hdfs: http://localhost:9870
	yarn: http://localhost:8088
```

```shell
[hadoop@master hadoop-3.1.1]$ hdfs dfs -ls /
[hadoop@master hadoop-3.1.1]$ hdfs dfs -mkdir /user
[hadoop@master hadoop-3.1.1]$ hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2020-01-06 22:04 /user
[hadoop@master hadoop-3.1.1]$
```

```shell
# Hadoop在master查看live nodes为0
关闭防火墙
检查每台机器上的/etc/hosts文件，将没有用或不清楚作何用的ip:name对删除，最后只留下了
127.0.0.1 localhost
10.77.20.100 master
10.77.20.101 slave1
10.77.20.102 slave2
```

# 安装Hive

```shell
scp /Users/dingyuanjie/study/BigData/hadoop_cluster/apache-hive-3.1.2-bin.tar.gz hadoop@192.168.0.201:/home/hadoop/software

[hadoop@master software]$ tar -zxvf apache-hive-3.1.2-bin.tar.gz
[hadoop@master software]$ mv apache-hive-3.1.2-bin hive

[hadoop@master conf]$ pwd
/home/hadoop/software/hive/conf
[hadoop@master conf]$ mv hive-env.sh.template hive-env.sh


export HADOOP_HOME=/home/hadoop/software/hadoop-3.1.1
export HIVE_CONF_DIR=/home/hadoop/software/hive/conf
```

```shell
2.6
root
	mkdir -p /data/hadoop/namenode
	mkdir -p /data/hadoop/data
	mkdir -p /data/hadoop/tmp
	
	master 	
		配置文件
  	slaves文件 
  		slave1
  		slave2
  	masters
    	master
    hadoop-env.sh
    	export JAVA_HOME=/usr/java/jdk1.8.0_161
    	
    master	
    	bin]./hdfs namenode -format	
    
    start..
    
    hadoop fs -ls /
    hadoop fs -mkdir /user
    hadoop fs -put /..
    
    mapreduce]hadoop jar ./hadoop-mapreduce-example-2.6.5.jar pi 5 10
    
    masterip:50070
    masterip:8088   yarn
```

```shell
查看日志
```



