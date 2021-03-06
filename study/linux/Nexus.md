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

# API

```shell
# harbor API
# 获取认证token
curl -ikL -X GET -u admin:Harbor12345 http://192.168.0.35/service/token\?account\=admin\&service\=harbor-registry\&scope\=repository:library/nginx:pull
# 根据token获取image的tags
curl -ikL -X GET -H "Content-Type: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1Q..." http://192.168.0.35/v2/library/nginx/tags/list

# /v2/{imageName}/tags/list

# Nexus API
# 根据name获取相关信息
curl -u admin:111111 -X GET "http://192.168.0.35:8081/service/rest/v1/search?repository=docker-hosted&format=docker&name=openthings/busybox"
curl -u admin:111111 -X GET "http://192.168.0.35:8081/service/rest/v1/search?name=test/nginx"

curl -u sw_sage-prd-s:l8J8*b0 -X GET "https://nexus-shared.addpchina.com/service/rest/v1/search?name={imageName}"

curl -u sw_sage-prd-s:l8J8*b0 -X GET "https://nexus-shared.addpchina.com/service/rest/v1/search?repository={}&format=docker&name={imageName}"

# /service/rest/v1/search?name={imageName}
```





