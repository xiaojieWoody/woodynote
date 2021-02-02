/maprposix/dp.prod.tj/ad-vantage/data/store/collected/car-data/ingest_service/error/2020/10/29/csvg2bwccntjp009_20201029T162024/20201016T153953_CCA_9010_3100272_BN_FASETH/20201016T155800_20201016T155900_CCA_9010_3100272_VN8KE67_BN_FASETH.MF4



/ad-vantage/data/store/collected/car-data/MDF4/ingest/VKHH381/2020/11/10/0f9a8fc8-ded7-4f39-a519-0175afff57d2/3100365/MT_RE/20201110T120105_20201110T120205_CCA_9010_3100365_VKHH381_MT_RE.MF4



nohup java -jar -Xms2048m -Xmx10240m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 mdf4-pipeline-1.0.0_20201122.jar --mf4-source-root-path /ad-china/data/store/compliance/c-processed/collected/car-data/MDF4/ingest/VKHH381/2020/11/10 --mf4-target-root-path /ad-china/data/store/compliance/c-processed/collected/car-data/MDF4/ingest/VKHH381/2020/1122_tmp --processed-log-root-path /data/home/tmp/json_file --batch-process-num 8 --forward-day-num 20 --process-loop-minute-rate 1 --car-ids VKHH381 --process-type BATCH --mf4-axis-type TVRfUkU= --mf4-can-type Qk5fQ0FMSUZSLEJOX0ZBU0RMVCxCTl9JVUtFVEg= >20201122_1_mf4.log &



java -jar -Xms2048m -Xmx9216m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 mdf4-pipeline-1.0.0_1129_v1.jar --mf4-source-root-path /maprposix/dp.prod.tj/ad-vantage/data/store/collected/car-data/ingest_service/ --mf4-target-root-path /root/data/tmp_1129/ --processed-log-root-path /data/home/mdf4_tools/logs --batch-process-num 8 --forward-day-num 40 --process-loop-minute-rate 1 --car-ids any --process-type SINGLE --mf4-axis-type TVRfUkU= --mf4-can-type Qk5fRkFTRVRILEJOX0ZBU0RMVA==



--mf4-source-root-path 为/maprposix/dp.prod.tj/ad-vantage/data/store/collected/car-data/ingest_service/ 

--mf4-target-root-path可以为任意目录，后面会在该目录后面自动生成扫描的目录，例如 /任意目录/error/2020/10/29/csvg2bwccntjp009_20201029T162024/20201016T153953_CCA_9010_3100272_BN_FASDLT目录

--car-ids 为any

--mf4-axis-type为需要处理的Axis类型名的base64编码值，如果有多个Axis类型则名称之间用,连接，然后再进行base64编码，例如MT_RE为TVRfUkU=

--mf4-can-type为需要处理的CAN类型名称的base64编码，如果有多个Axis类型则名称之间用,连接，然后再进行base64编码，例如BN_FASETH,BN_FASDLT为Qk5fRkFTRVRILEJOX0ZBU0RMVA==





## 环境变量

```shell
vi ~/.bashrc

# 新增
export GOROOT=/usr/local/go
export GOPATH=/Users/username/go/code # 代码目录，自定义即可
export PATH=$PATH:$GOPATH/bin

# 及时生效，请执行命令：source ~/.bashrc
```

bin：存放编译后可执行的文件

pkg：存放编译后的应用包

src：存放应用源代码





option + command + v



https://zhuanlan.zhihu.com/p/60703832



go mod国内镜像

```shell
# 在 Linux 或 macOS 上面，需要运行下面命令（或者，可以把以下命令写到 .bashrc 或 .bash_profile 文件中）：
vi ~/.bash_profile
# 启用 Go Modules 功能
go env -w GO111MODULE=on
# 配置 GOPROXY 环境变量，以下三选一
# 1. 七牛 CDN
go env -w  GOPROXY=https://goproxy.cn,direct
# 2. 阿里云
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
# 3. 官方
go env -w  GOPROXY=https://goproxy.io,direct
# 生效
source ~/.bash_profile

# hello目录
go mod init hello
```

https://github.com/linehk/gopl/blob/master/ch1/server3/main.go