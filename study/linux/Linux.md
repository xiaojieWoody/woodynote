```shell
# Centos7下安装netstat
yum install net-tools
# 查看端口服务是否启动
netstat -anp |grep "3301"
# 查看端口
lsof -i tcp:8080
```

