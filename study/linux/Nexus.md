# 安装

```shell
# 8081为nexus的管理端口
# 5000为容器镜像服务使用的端口，留作后面创建容器镜像仓库服务使用
docker run -d --name nexus -p 5000:5000 -p 8081:8081 sonatype/nexus3
# mkdir /nexus-data
# docker run --name nexus -d -p 5000:5000 -p 8081:8081 -v /nexus-data:/nexus-data sonatype/nexus3
```

# 配置

```shell
# 访问
http://192.168.0.35:8081
# 设置
# Blob
Gear > Repository > Blob Stores > Create blob store
1. Name : docker-hosted
2. Name : docker-hub
# Repository
Gear > Repository > Repositories > Create repository
1. Select docker (hosted)
- Name : docker-hosted
- Check HTTP and input 5000
- Check Enable Docker V1 API
- Select Blob store docker-hosted
2. Select docker (proxy)
- Name : docker-hub
- Check Enable Docker V1 API
- Input https://registry-1.docker.io in Remote storage
- Select Use Docker Hub
# Reamls
Gear > Realms > Move Docker Bearer Token Realm to active > Save
# docker
vi /etc/docker/daemon.json
 "insecure-registries" : ["192.168.0.35:5000"]
systemctl restart docker

docker login 192.168.0.35:5000

docker tag 6d5fcfe5ff17 192.168.0.35:5000/busybox:v20200205
docker push 192.168.0.35:5000/busybox:v20200205
```

