# 搭建Centos7虚拟机

* 虚拟机设置网络为桥接模式

## 修改主机名：

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

```shell
# https://downloads.mysql.com/archives/community/
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

