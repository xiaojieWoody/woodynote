# 作用

* jar包管理
* 项目的构建和管理

# 下载安装

* 先安装jdk
* 将maven所在的目录的bin文件夹配置到Path环境变量中

# 目录结构

* src文件夹
  * main文件夹
    * java文件夹：Java源文件
    * resources文件夹：资源存放的文件，xxx.xml
  * test文件夹
    * java文件夹：所有java测试的源文件
    * resources文件夹：存放测试所用到的一些资源文件
* target：项目输出的目录
* pom.xml文件：唯一标识该maven项目

# 配置

* settings.xml

```xml
<mirror>  
	<id>alimaven</id>  
	<name>aliyun maven</name>  
	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
	<mirrorOf>central</mirrorOf>          
</mirror>
```

* 仓库

  * localRepository的含义本地仓库
    * `<localRepository>E://temp//myrepo</localRepository>` 自己指定了一个本地`maven`仓库所在的地址，而不是之前默认的当前用户的`.m2`下面的`repository`文件夹
  * 剧透：如果一个jar包已经在本地仓库中有了，注意这个jar包版本一定是要一致的才算有了，如果版本不一致，不能算作有，比如3.8.1  3.8.2，其他如果有了，那么就不会再次去重复下载
    * 优势：可以节约本地的一个空间存储

* pom.xml

  * project object model  项目对象模型，它为唯一标识该项目

  1. 3个必填字段

     * 由groupId  artifactId  version所组成的就可以唯一确定一个项目
       * groupId标识的项目组（填写公司的域名）  组织
       * artifactId项目名称
       * version版本号

  2. pom里面还有会dependencies标签，这个标签就是用来配置项目中具体需要哪些jar包

  3. properties标签，用来定义pom中的一些属性的

  4. build：指定如何构建当前项目的

     source:   指定了当前构建的source目录

     plugin：指定了进行构建时使用的插件

  5. packging  指定当前构建项目的类型，war jar pom
  6. 补充：pom文件是可以继承的，超级pom文件等等
  7. 比如一定pom文件中定义了一些东西，另外一个pom文件也想要使用，这时候就可以使用继承的方式

  

