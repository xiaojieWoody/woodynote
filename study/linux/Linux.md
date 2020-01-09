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



```shell
# 常用命令
w
whoami
who
# vi
yyp			# 复制
u				# 撤销
x				# 删除光标所在字符
dd			# 删除一行
A				# 来到一行的末尾
o				# 光标下一行插入一行
r				# 替换所选中字符
:s/boot/BOOTBOOTBOOT/g   # 替换光标所在行的所有boot为BOOTBOOTBOOT
:b/name/s//NAMENAMENAME/g # 替换文章中所有name
/name		# 搜索name，按n向下搜索，N向上搜索
G				# 文章底部
```

```shell
# 文件
ls
pwd
ls -l         # 文件列表显示
ls -lt        # 文件时间列表显示
ls -lrt				# 文件时间列表显示，最近的在最下
ls -lart      # a，显示隐藏文件
cat a.txt     # 显示文件所有
more a.txt		# 只显示一屏
man more      # 查看more有哪些功能
tail -10 test.txt   # 最后10行打印出来
tail -f test.txt    # 动态打印
head -10 test.txt   # 最前面10行打印出来
echo "test111" >>test.txt     # 文件中追加内容
cat /etc/passwd >> test1.txt  # passwd中内容添加到test1.txt中
cat /etc/passwd > test1.txt   # 先清空，再添加
echo "" >test1.txt      # 清空文件
>test1.txt							# 清空文件
tail -10 test.txt > test1.txt # test.txt中最后10行添加到test1.txt中
cat test1.txt | tr  '[a-z]' '[A-Z]' >test2.txt    # 文件中字符都替换成大写，然后重定向到test2.txt中
cat test2.txt | tr -d " "  # 删除空格
cp test.txt test1.txt      # 复制  -r 递归复制
touch test.txt             # 对文件时间进行更新，或者创建文件
date                       # 日期
grep 'test111' test.txt -n # 查找文件内容并显示在第几行
grep -A 3 'FINISHED' test.txt    # 打印文件中 FINISHED 字符后三行
grep -B 3 'FINISHED' test.txt    # 打印文件中 FINISHED 字符前三行
grep -C 3 'FINISHED' test.txt    # 打印文件中 FINISHED 字符上下三行
grep -A 3 'FINISHED' test.txt --color=auto    # 高亮显示FINISHED
grep -A 3 'FINISHED' test.txt -c --color=auto # 有多少行
grep -A 3 'FINISHED' test.txt --color=auto | wc -l  # 有多少行
find ./ -name 'test.txt'        # 查找当前目录下 test.txt 文件
cat /etc/passwd | sort          # 按首字母排序
cat /etc/passwd |cut -d ":" -f 1 # 按 : 切割
cat 1.txt | uniq                 # 去重显示
cat 1.txt | uniq | wc -l         # 去重后有多少行
cat 1.txt | sort | uniq -c       # 去重排序统计
```

```shell
# 用户、组
whoami			# 当前用户
su -testA 	# 切换用户
su - testA  # 切换用户并带其系统变量
env         # 查看系统变量
exit      # 退出用户
cat /etc/passwd  # 查看用户  用户名:passwd:UID:GID:名称:家目录:shell
id
groupadd nginxA			# 创建组
useradd nginxtest1 -g nginxA -s /bin/nologin   # 创建用户，不用登录
id nginxtest1      # 查看用户
useradd nginxtest1 -g nginxA -s /bin/bash   # 创建用户，需登录
sudo xxx          # 以系统用户身份执行  
passwd nginxtest2 # 给用户设置密码  
passwd            # 重新设置密码
userdel nginxtest1# 删除用户
cat /etc/group    # 查看组
groupdel testB    # 删除组
# 权限 
# r   w   x
# 读4 写2 执行1
# 用户 组 其他
chown nginxtest2 test1    # 更换文件所属用户
ls -lrt
chown nginxtest2:nginxA test1 # 更换文件所属组
chmod u+x test1         # 用户加上x权限
chmod u-x test1         # 用户减去x权限
chmod g+x test1         # 组加上x权限
chmod 760 tset1         # 设置权限
```

```shell
# 压缩、打包
gzip install.log        # 压缩
zcat install.log.gz    # 查看
tar cvf 20180317.tar * # 打包
tar xvf 20180317.tar   # 解包
tar zcvf 20180317.tar.gz *   # 打包压缩
tar zxvf 20180317.tar.gz     # 解压
```

```shell
hostname
ssh root@192.168.0.110      # linux之间交互
# 免密登录
# 192.168.0.111
cd .ssh/
ssh-keygen
cat id_rsa.pub              # 把公钥放到对方
# 对方  192.168.0.110
cd .ssh/
ssh-keygen
touch authorized_keys       # 将192.168.0.111的id_rsa.pub内容填进去
# 192.168.0.111 
ssh 192.168.0.110
```

```shell
# 检测端口，判断服务的连接情况
netstat -anlp | grep 21
netstat -anltp | grep vsftpd
ps -ef|grep 30763
```

```shell
# ftp  文件传输服务         sftp 22, ftp 21
# 192.168.0.111
service vsftpd start      # 启动ftp服务
yum install ftp
useradd ftpuser1
passwd ftpuser1
# 192.168.0.110
ftp 192.168.0.111
Name:ftpuser1
Password:ftpuser1
put
get

# scp
scp xxx root@192.168.0.111:/home/test/1.txt
scp root@192.168.0.111:/home/test/1.txt /root/

# crontab 定时任务       分 时 日 月 周 crontab
crontab -e
0 22 * * * /usr/sbin/vsftpd stop   # 每天12点执行
#30 6,12 * * *                 # 每天6点30和12点30执行
#10 6-12 * * *                 # 每天6点到12点之间每10分钟执行
#*/5 * * * *                   # 每5分钟执行一次
crontab -l

# 后台运行
jobs				# 查看后台运行程序
tail -f /var/log/message &      # 后台运行程序
fg 2        # jobs后，调度后台第2个运行程序
bg 1        # jobs后，调度后台第1个程序运行

# 系统级服务 service
cd /etc/init.d/
ls

chkconfig --list | grep vsftpd
chkconfig --level 35 vsftpd on    # 3和5级别启动
chkconfig vsftpd on
chkconfig vsftpd off
```

![image-20200104120925920](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200104120925920.png)

```shell
df -h			#查看硬盘大小
du -sh *  #查看文件大小
date -s "10:10:20 2018-02-10"    # 设置时间
```





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

   