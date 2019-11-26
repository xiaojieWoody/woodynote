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
```

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191125182303871.png" alt="image-20191125182303871" style="zoom:25%;" />

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191125232354162.png" alt="image-20191125232354162" style="zoom:50%;" />

* 配置环境变量

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

  