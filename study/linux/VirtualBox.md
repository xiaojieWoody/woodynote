# 搭建Centos7虚拟机

* 虚拟机设置网络为桥接模式

* 修改主机名：

  * `sudo hostnamectl set-hostname <newhostname>`

* 配置静态IP

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

* 关闭防火墙

  ```shell
  systemctl stop firewalld.service
  systemctl disable firewalld.service
  systemctl status firewalld.service
  ```

* 免密登录

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

  