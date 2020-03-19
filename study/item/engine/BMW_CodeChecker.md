# compliance-service

![image-20200220131445345](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200220131445345.png)

* protobuf
* filter
  * 优化点
* operatorType
  1. 上报代码备份信息：CI Checker扫描前对代码进行备份信息的上报
     * 将请求参数中的信息（`file_path`,` file_name`,` file_size`,` md5`,` sid`）存入代码备份表`t_compliance_code_backup`
  2. 获取规则元信息：CI Checker扫描前获取扫描规则元信息
     * 根据`appId`从规则表`t_compliance_rules`中通过降序获取最新规则信息
     * 规则下载任务表`t_compliance_download_rules_job`中插入一条信息
     * 返回规则信息，包括生成的规则文件下载签名`downloadSign`和规则文件解压签名`unzipSign`
  3. 规则文件下载：CI Checker扫描前下载规则文件
     * 根据请求参数规则文件下载签名`downloadSign`作为参数，`t_compliance_download_rules_job`和`t_compliance_rules`根据`rule_id`来`join`查询规则下载任务信息
     * 配置过期时间值，判断当前时间和创建规则任务的差值是否小于过期时间值
     * 创建zip文件，并添加规则文件，以规则文件解压签名`unzipSign`作为password进行加密
     * 将zip文件转成字节数组返回
  4. 代码扫描结果上报：CI Checker扫描代码后将扫描结果上报
     * 请求参数app_id和sid作为参数分别到代码备份表`t_compliance_code_backup`和规则下载任务表`t_compliance_download_rules_job`查询sid，并进行比较是否相同
     * 将请求参数中的扫描详情（byte[]）转为文件存入配置的位置
     * 将请求信息插入代码扫描结果表`t_compliance_code_check_results`，包括存入扫描详情路径
  5. 获取扫描结果：CI Checker增量检查时需要的扫描结果
     * 根据请求参数`app_id`，从代码扫描结果表`t_compliance_code_check_results`，降序查找最新结果数据，包括扫描详情文件存储路径
     * 根据路径，将扫描详情文件转为byte数组返回
  6. 根据产出MD5检查是扫描否通过：CD Checker根据Artifact的MD5进行扫描结果查询
     * 根据请求参数产出的MD5值`md5`，到代码扫描结果表`check_result`中查找扫描结果
  7. 规则文件更新
     * 根据请求参数中的文件名称、规则文件及配置文件中配置的存储路径参数，将规则文件存储（zip文件）到指定位置，返回规则文件存储路径
     * 解压规则文件到配置路径，并返回压缩包内文件名称，根据之前路径再加上解压文件路径组成规则文件路径
     * 规则信息表`t_compliance_rules`插入一条规则数据，包括规则存放路径
  8. 获取工具产出个数：根据产出个数判断CI-Checker或者CD-Checker是合法的
     * 根据请求参数`app_id`、工具产出的类型`artifactType`、工具产出的MD5`md5Sum`到可执行ci/cd的jar包的注册信息表`t_compliance_artifact_info`查找数据，如果为null，则返回产出个数为0，否则返回1
  9. 获取代码产出路径
     * 根据请求参数`app_id`到代码产出路径映射表`t_compliance_code_artifact_map`查找代码产出路径字段并返回`artifactPath`
* login接口
  * 根据请求参数`app_id`到鉴权信息表`t_compliance_auth_info`查询数据并返回

#cichecker

* os-maven-plugin插件的作用是在编译时确定当前执行编译的操作系统环境，提供对应的系统变量，如：${os.detected.name} , ${os.detected.arch} , ${os.detected.version.*} , ${os.detected.classifier} 等， 其中：${os.detected.classifier} = ${os.detected.name}+${os.detected.arch} 

* SpringBoot打包时通过命令参数来动态切换环境

  * `spring.profiles.active=@profileActive@`
  * `mvn clean install -Dmaven.test.skip=true -Ptest`

* 启动参数

  ```shell
  java -jar compliance_code_check.jar --src-path=/Users/tools/source_code --backup-path=/Users/tools/backup --appId=ingest --interrupt=true  --prefix-script=./pre.sh  --post-scrip=./post.sh
  ```

* 将启动参数解析成`CommandLine`对象
* `MappedByteBuffer`nio提供的文件内存映射，读写性能极高
* MessageDigest 类为应用程序提供信息摘要算法的功能，如 MD5 或 SHA 算法。
  * 信息摘要是安全的单向哈希函数，它接收任意大小的数据，并输出固定长度的哈希值
  * MessageDigest 对象开始被初始化。该对象通过使用 ``update（）方法处理数据。任何时候都可以调用 ``reset（）方法重置摘要。一旦所有需要更新的数据都已经被更新了，应该调用digest() ``方法之一完成哈希计算
  * 对于给定数量的更新数据，`digest` 方法只能被调用一次。在调用 `digest` 之后，MessageDigest 对象被重新设置成其初始状态
* `new File(System.getProperty("java.class.path"))`

```shell
# 1. 启动参数解析成CommandLine对象
# 2. POST请求 8 根据产出个数判断CI-Checker是合法的 0-false =》 ，返回1，则判断通过
# 3. 创建临时目录，临时目录中创建扫描代码目录，把src-path目录下的文件添加到backup-path目录下的zip中并返回zip的路径，POST请求 1 上报代码备份信息
# 4. 将src-path目录下的文件 拷贝到 临时目录中创建扫描代码目录
# 5. 执行 prefix-script 指定的脚本
# 6. POST 9 根据 app_id 获取代码产出路径
# 7. POST 2 根据 app_id 获取规则元信息， POST 3 规则文件下载，存入 临时目录
# 8. POST 5 根据 app_id 获取扫描详情文件, 解压规则文件到临时文件中的rule目录并再压缩
```



# cdchecker

#code-checker

* maven-assembly-plugin
  * 够将Maven应用的输出及其依赖库整合打包为一个压缩包，以便于应用的分发使用





* Antlr4 是一款强大的语法生成器工具，可用于读取、处理、执行和翻译结构化的文本或二进制文件
* 语法分析器（parser）是用来识别语言的程序，本身包含两个部分：词法分析器（lexer）和语法分析器（parser）。词法分析阶段主要解决的关键词以及各种标识符，例如 INT、ID 等，语法分析主要是基于词法分析的结果，构造一颗语法分析树

* 类ParseTreeWalker是ANTLR运行时提供的用于遍历语法分析树和触发Listener中回调方法的树遍历器



* Hello.g

```java
grammar Hello;               // 定义文法的名字

s  : 'hello' ID ;            // 匹配关键字hello和标志符
ID : [a-z]+ ;                // 标志符由小写字母组成
WS : [ \t\r\n]+ -> skip ;    // 跳过空格、制表符、回车符和换行符
```

* 执行以下命令来生成识别器`$ ./antlr Hello.g`
  * 该命令会在相同目录下生成后缀名为tokens、interp和java的8个文件
* 编译由ANTLR生成的Java代码
  * `javac -cp antlr-4.7.1-complete.jar $*`
* 把它保存为compile文件，可以用以下命令编译代码
  * `$ ./compile Hello*.java`
* 到此，已经有一个可以被执行的识别器，只缺一个主程序去触发语言识别