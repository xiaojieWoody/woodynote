#   搭建Centos7虚拟机

* 使用国内镜像源

  ```shell
  [root@bogon ~]# cd /etc/yum.repos.d/
  [root@bogon yum.repos.d]# mkdir repo_bak
  [root@bogon yum.repos.d]# mv *.repo repo_bak/
  
  [root@bogon yum.repos.d]# wget http://mirrors.aliyun.com/repo/Centos-7.repo
  [root@bogon yum.repos.d]# wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
  [root@bogon yum.repos.d]# ls
  Centos-7.repo  CentOS-Base-163.repo  repo.bak
  [root@bogon yum.repos.d]# yum clean all
  [root@bogon yum.repos.d]# yum makecache
  
  # 安装epel源
  [root@bogon yum.repos.d]# yum list | grep epel-release
  [root@bogon yum.repos.d]# yum install -y epel-release
  [root@bogon yum.repos.d]# ls			# epel源安装成功，比原来多了一个epel.repo和epel-testing.repo文件
  Centos-7.repo  CentOS-Base-163.repo  epel.repo  epel-testing.repo  repo.bak
  # 使用阿里开源镜像提供的epel源
  [root@bogon yum.repos.d]# wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo   
  [root@bogon yum.repos.d]# ls
  CentOS7-Base-163.repo  Centos-7.repo  epel-7.repo  epel.repo  epel-testing.repo  repo_bak
  [root@bogon yum.repos.d]# yum clean all
  [root@bogon yum.repos.d]# yum makecache
  # 查看系统可用的yum源和所有的yum源
  [root@bogon yum.repos.d]# yum repolist enabled
  [root@bogon yum.repos.d]# yum repolist all
  ```

* 虚拟机设置网络为桥接模式

## 修改主机名

* `sudo hostnamectl set-hostname <newhostname>`

## 配置静态IP

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	ONBOOT=yes
	BOOTRPOTO=static
	IPADDR=192.168.0.201      # 前两个要和宿主主机一致
	NETMASK=255.255.255.0
	GATEWAY=192.168.0.1
	DNS1=8.8.8.8
	DNS2=8.8.4.4
# 保存后，重启网络
service network restart
# 网络改成 桥接模式

# woody角色使用sudo
su root
chmod 740 /etc/sudoers
vi /etc/sudoers
# 添加
woody   ALL=(ALL)       ALL
```

## ssh连接

```shell
vi /etc/ssh/sshd_config
将PasswordAuthentication 的属性 no 改为 yes
service sshd restart
```

## 关闭防火墙

```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service
```

## 静态IP：NAT、桥接网络

```shell
# 使用 网络地址转换(NAT) 配合 仅主机(Host-Only)网络 设置静态 ip，可以当主机网络变化的时候静态 ip 依旧可用
需要设置两个网卡，一个使用 网络地址转换(NAT) ，一个使用 仅主机(Host-Only)网络。NAT 负责让虚拟机与外网通信，Host-Only 负责虚拟机与主机通信，所谓静态 ip 指的是虚拟机与主机之间通信时虚拟机 ip 保持不变，即主要是 Host-Only 的作用
```

```shell
# 1. 在 VirtualBox 上配置两个网卡
管理>>全局设定>>网络>>创建+ >> 设置 >> 直接ok
这样一来 NAT 网络就有了，为配置 网络地址转换(NAT) 网卡做好了准备
# 2. 设置 Host-Only 网络为配置 Host-Only 网卡
管理 >>主机网络管理器 >> 属性 >> 启用、手动配置网卡：IPv4地址192.168.0.2、IPv4网络掩码：255.255.255.0
Host-Only 的默认网关就是 192.168.56.1
# 3. 配置网卡 (虚拟机需要关机)
右键>>设置>>网卡1 & 网卡2
网卡1：启用网络连接：网络地址转换（NAT）
网卡2：启用网络连接：仅主机（Host-Only）网络、记住MAC地址在虚拟机中配置时用得到
# 4. 虚拟机配置静态 ip
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
添加
HWADDR=刚才Host-Only网卡中的MAC地址
IPADDR=192.168.56.37
GATEWAY=192.168.56.1
DNS1=114.114.114.114
DNS2=8.8.8.8
# 5. 虚拟机重启即可。主机通过 192.168.56.37 访问虚拟机，虚拟机通过 NAT 访问外网
```

## jdk

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

## maven

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

## mysql

https://www.jianshu.com/p/1dab9a4d0d5f

```shell
# https://downloads.mysql.com/archives/community/
# 在 https://dev.mysql.com/downloads/repo/yum/ 找到 yum 源 rpm 安装包
# 下载
shell> wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
# 安装 mysql 源
shell> yum localinstall mysql57-community-release-el7-11.noarch.rpm
# 用下面的命令检查 mysql 源是否安装成功
shell> yum repolist enabled | grep "mysql.*-community.*"

# 使用 yum install 命令安装
shell> yum install -y mysql-community-server
# 在 CentOS 7 下，新的启动/关闭服务的命令是 systemctl start|stop
shell> systemctl start mysqld
shell> systemctl status mysqld
# 开机启动
shell> systemctl enable mysqld
# 重载所有修改过的配置文件
shell> systemctl daemon-reload
# mysql 安装完成之后，生成的默认密码在 /var/log/mysqld.log 文件中。使用 grep 命令找到日志中的密码
shell> grep 'temporary password' /var/log/mysqld.log
# 首次通过初始密码登录后，使用以下命令修改密码
shell> mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!'; 
mysql> flush privileges;

set password for 'root'@'localhost'=password('IamMySQL0108!');
GRANT ALL PRIVILEGES ON *.* TO 'woody'@'%' IDENTIFIED BY 'IamWoody0108!' WITH GRANT OPT
ION;

# 添加一个允许远程连接的帐户
mysql> GRANT ALL PRIVILEGES ON *.* TO 'zhangsan'@'%' IDENTIFIED BY 'Zhangsan2018!' WITH GRANT OPTION;
# 设置默认编码为 utf8
# 修改 /etc/my.cnf 配置文件，在相关节点（没有则自行添加）下添加编码配置
[mysqld]
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
# 重启mysql服务，查询编码
shell> systemctl restart mysqld
shell> mysql -uroot -p
mysql> show variables like 'character%';
# 默认配置文件路径：
配置文件：/etc/my.cnf
日志文件：/var/log/mysqld.log
服务启动脚本：/usr/lib/systemd/system/mysqld.service
socket文件：/var/run/mysqld/mysqld.pid
```

```shell
# 先检查系统是否装有mysql
rpm -qa | grep mysql
# 删除可用
yum remove mysql
# 安装所需依赖
yum install numactl
yum install perl
yum install net-tools

# 解压
tar -vxf mysql-5.7.17-1.el7.x86_64.rpm-bundle.tar

rpm -qa | grep mariadb
rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64

rpm -ivh mysql-community-common-5.7.17-1.el7.x86_64.rpm   
rpm -ivh mysql-community-libs-5.7.17-1.el7.x86_64.rpm   
rpm -ivh mysql-community-client-5.7.17-1.el7.x86_64.rpm  
rpm -ivh mysql-community-server-5.7.17-1.el7.x86_64.rpm  
rpm -ivh mysql-community-devel-5.7.17-1.el7.x86_64.rpm 

# 初始管理员密码 temporary password
cat /var/log/mysqld.log
YO/zHtA3vHfl

mysql -uroot -p
YO/zHtA3vHfl

# 重置密码
mysql> set PASSWORD=PASSWORD('Iam0108@Woody');
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Iam0108@Woody' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

## nginx

```shell
# http://nginx.org/download/
sudo yum install epel-release
sudo yum install nginx
# 开机启动
sudo systemctl enable nginx
# 关闭开机启动 Nginx
sudo systemctl disable nginx
# 启动
sudo systemctl start nginx
# 重启
sudo systemctl restart nginx
# 修改 Nginx 配置后，重新加载
sudo systemctl reload nginx
# 状态
sudo systemctl status nginx

# 如果服务器开启了防火墙，则需要同时打开 80（HTTP）和 443（HTTPS）端口
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload

# 验证
http://ip

# 相关的配置文件都在 /etc/nginx/ 目录中
```

## docker

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

## docker-compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

## 免密登录

```shell
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

# Ubuntu18.04 虚拟机

* vi命令的编辑模式下不能正常使用方向键和退格键的问题

  ```shell
  # 卸载vim-tiny
  sudo apt-get remove vim-common
  # 安装vim-full
  sudo apt-get install vim
  ```

* vi 没有那个文件或目录

  ```shell
  sudo -i
  apt-get update
  apt-get install vim
  ```

* 关闭防火墙

  ```shell
  ufw disable
  ```

* 配置ssh

  ```shell
  # 安装OpenSSH服务：
  sudo apt-get install openssh-server
  # 检查一下服务的状态
  sudo service ssh status
  
  # 安装防火墙
  sudo apt-get install ufw
  # 开放防火墙
  sudo ufw enable
  # 防火墙充许22端口对外开放
  sudo ufw allow 22 
  ```

* 安装gcc

  ```shell
  sudo apt update
  # 安装build-essential软件包：包括gcc，g ++和make
  sudo apt install build-essential
  # 验证
  gcc --version
  ```
  
* 切换root

  ```shell
  sudo -i
  ```

## 配置静态IP

https://blog.csdn.net/davidhzq/article/details/102991577

```shell
# 界面上设置
IPv4
手动
地址 192.168.0.229  255.255.255.0  192.168.0.1
DNS	 8.8.8.8，8.8.4.4 
```

# 移动硬盘中装Ubuntu

* **工具：**
  * Rufus（制作U盘启动盘）、DiskGenius（调整移动硬盘的分区大小）、U盘、移动硬盘
  * https://www.pianshen.com/article/83071065557/
  
* **下载ubuntu18.04镜像文件**
  
  * https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/18.04/
  
* **修改移动硬盘分区**
  * 没有必要把移动硬盘格式化
  * 有效操作是直接从硬盘的现存分区压缩出安装ubuntu系统的**空闲空间**即可
    * 注意是压缩出空闲空间，并且这个空闲空间需要在整个磁盘的前面，因为系统引导项只扫描前137G空间，如果这片空间在磁盘尾很有可能安装之后ubuntu无法启动，这里需要用到DiskGenius，windows自带的磁盘管理无法做到调整压缩空间的位置
  * ![image-20200818105451607](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200818105451607.png)
  * 压缩之后的效果如图，可以看到磁盘首是压缩出来的600G空闲空间，同时注意红框，移动硬盘的架构是MBR，压缩出来就可以不用管了，具体的分区操作等到安装环节再来设置，那么剩下的1.2T空间格式为NTFS可以正常存储数据
  * win10磁盘管理自带压缩空间
  
* **制作U盘启动盘**
  
  * 打开Rufus，选择U盘和下载好的镜像文件，其他的选择默认即可，准备就绪就可以直接开始
  
* **BIOS设置**
  * 惠普为例子
  * 开机时，按F10进入BIOS，启动选项中：
    * 禁用安全模式
    * “USB闪存驱动器/USB硬盘”上移到“操作系统的启动管理员”之上
  
  ```shell
  # 机械革命笔记本
  开机按F2 进入BIOS
  ```
  
* **开始安装Ubuntu18.04**
  * 插好U盘和移动硬盘
  * 直接点install ubuntu
  * 等到选择安装类型这个界面时间，一定要选择“其他选项”，不要选其他的
  * 之后进入分区界面，这里分区采取最简方式
    * 选中600G的空闲分区，点左下角+号进入分区模式
    * /boot ext4 2G
    * / ext4 100G
    * swap swap 16G
    * /home ext4 剩多少给多少
  * 最后的“安装启动引导的设备”就要选择移动硬盘，千万不能选别的，不然很有可能最后windows进不去了，移动硬盘是/dev/sdb，就一定要选/dev/sdb
  * 设置好后继续安装，等待一段时间后，安装完成重启即可
  * 重启之后就能正常进入grub2引导界面选择进入ubuntu还是windows



* 惠普开机时按F9可选择是windows还是ubuntu启动
* 机械革命开机时按F10选择是windows还是ubuntu启动



* 自己

* 插入移动硬盘、U盘，开机，从USB硬盘启动
  * 按esc，进入grub>
  * 输入exit
  * 选择Ubuntu启动

# 安装shellcheck

```shell
yum -y install epel-release
yum install ShellCheck

apt-get install shellcheck
```

# Mac 安装nginx

```shell
nginx: http://nginx.org/download/nginx-1.12.2.tar.gz
zlib: http://zlib.net/zlib-1.2.11.tar.gz
pcre: https://ftp.pcre.org/pub/pcre/pcre-8.38.tar.gz
openssl: https://www.openssl.org/source/openssl-1.1.0g.tar.gz

tar -zxvf nginx-1.12.2.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
tar -zxvf pcre-8.38.tar.gz
tar -zxvf openssl-1.1.0g.tar.gz

# 进入nginx目录
./configure --prefix=/usr/local/nginx --with-zlib=../zlib-1.2.11 --with-pcre=../pcre-8.38 --with-openssl=../openssl-1.1.0g
# 依次执行以下命令
make
sudo make install
# nginx 就被安装到 /usr/local/nginx 目录下
# 启动
sudo /usr/local/nginx/sbin/nginx

# 关闭
sudo /usr/local/nginx/sbin/nginx -s stop

# 重启
sudo /usr/local/nginx/sbin/nginx -s reload

# 指定配置文件启动
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
# 解决方法：
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

sudo lsof -i 4tcp:80

nginx: [emerg] bind() to 0.0.0.0:80 failed (48: Address already in use)
# mac解决办法
mac自带的apache服务器
sudo apachectl stop终止掉apache服务。问题解决

/usr/local/nginx/logs/access.log" failed (13: Permission denied)
# mac 解决办法
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

# Nginx代理本地域名访问虚拟机上服务（Nexus）

```shell
# nginx.conf
server {
        listen       80;
        # （Mac）本地ip
        server_name  192.168.0.104;

        location / {
        		 # 虚拟机上服务（Nexus）IP、Port
             proxy_pass         http://192.168.0.35:8081;
             proxy_set_header   Host             $host;
             proxy_set_header   X-Real-IP        $remote_addr;
             proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
	     			 proxy_set_header   X-Forwarded-Proto  $scheme;
        }
}
# mac hosts
192.168.0.104 www.nexusregistry.com
# mac 浏览器访问
http://www.nexusregistry.com
```



