# 搭建Centos7虚拟机

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

```shell

02、键入以下命令安装build-essential软件包：

linuxidc@linuxidc:~/www.linuxidc.com$ sudo apt install build-essential

该命令将安装一堆新包，包括gcc，g ++和make。

03、要验证GCC编译器是否已成功安装，请使用gcc --version命令打印GCC版本：

linuxidc@linuxidc:~/www.linuxidc.com$ gcc --version

Ubuntu 18.04存储库中可用的默认GCC版本是7.4.0：
```

