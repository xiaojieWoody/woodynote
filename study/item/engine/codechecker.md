* CI：`Continuous Integration` 持续集成
* CD：`Continuous Deployment`持续部署
* `Compliance` 合规
* `Ingest`摄取
* `Extraction`提取



* 正则表达式
  * `[/]+$`
  * $：匹配输入字符串的结尾位置
  * [：标记一个中括号表达式的开始
  * +：匹配前面的子表达式一次或多次



* `InitializingBean`
  * 



* PR
* `Munich`：慕尼黑
* `manually`手动的

* `Snapshot`：快照
* `Artifactory`：人工工厂
* `Staging Environment`：暂存环境
* `Airflow`： Python 编写的调度工具
* `Artifacts`：成品，人工品
* `manual review`：人工审核
* `code backup`：代码备份
* `AST`：抽象语法树`abstract syntax tree`
  * 是源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码。树上的每个节点都表示源代码中的一种结构
* `contrains`：约束
* `recursion`：递归
* `splicing node`：拼接节点
* `Antlr4`：Java编写的语法解析器生成器

* **功能设计.pdf**
* **规则设计.pdf**
* **API访问鉴权.pdf**
* **CI Code Checker Parser SDK 使用说明.pdf**
* **operate动作标识说明.pdf**
* **代码解析详细设计.pdf**
* **安全设计.pdf**





* `protobuf`
  * 将结构化的数据序列化、反序列化，经常用于网络传输
* `RestTemplate`
  * 请求`RESTful Web Services`



* `CommandLineRunner`
  * `SpringBoot`提供，项目服务启动时加载一些数据或预先完成某些动作
  * 只需实现该接口，重写`run`方法，添加`@Component`注解，@Order定义执行顺序（小先执行）
  * run方法的参数是java -jar xxx.jar data1 data2 data3时传入





* `Commons CLI`

  * 命令行参数解析库

    ```shell
    java -jar compliance_code_check.jar --src-path=/Users/tools/source_code --backup-path=/Users/tools/backup --appId=ingest --interrupt=true  --prefix-script=./pre.sh  --post-scrip=./post.sh
    ```

* `CommandLine`

  ```java
  public static Option.Builder builder(String opt); // 获取指定选项的构建器
  
  // Option.Builder 选项构建器
  public static final class Option.Builder {
      public Option.Builder longOpt(String longOpt); // 对应长选项
      public Option.Builder desc(String description); // 选项的描述
      public Option.Builder required(); // 该选项是必选选项
  
      public Option.Builder hasArg(); // 选项需要参数
      public Option.Builder optionalArg(boolean isOptional); // 参数是否可选
      public Option.Builder argName(String argName); // 参数显示的名称
  
      public Option.Builder numberOfArgs(int numberOfArgs); // 允许的参数数量
      public Option.Builder hasArgs(); // 选项可以有无限个参数
  
      public Option.Builder valueSeparator(); // 参数之间分隔符为 '='
      public Option.Builder valueSeparator(char sep); // 参数之间的分隔符
  
      public Option build(); // 构建完成，返回 Option 对象
  }
  ```

  