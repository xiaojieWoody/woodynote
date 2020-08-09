# 1. Linux背景介绍 

* VirtualBox
* centos7

# 2. 系统操作

* 帮助命令

  ```shell
  # man
  man ls
  # help
  help cd       # 内部命令
  ls --help     # 外部命令
  # info，比help更详细，作为help的补充
  info ls
  ```

* 文件管理

  ```shell
  # pwd 
  # cd 
  cd -    # 回到之前目录
  # ls
  ls -l
  ls -a   # 显示隐藏文件
  ls -lrt # 时间倒序
  ls -lh  # 显示大小 MB
  # mkdir
  mdkir -p /data/k8s/conf       # 多级创建
  rm -rf /test/a                # 删除目录
  # cp
  cp -r 
  cp -p                         # 保留时间
  cp -a                         # 保留权限
  cp file* /                     
  cp file? /                    
  # mv
  mv /fileA /fileB             # 移动/重命名
  # touch                      # 创建文件
  # 通配符
  * # 匹配任何字符串
  ? # 匹配1个字符串
  [xyz] # 匹配xyz任意一个字符
  [a-z] # 匹配一个范围
  [!xyz]或[^xyz] # 不匹配
  ```

* 文本查看

  ```shell
  # cat 文件内容显示到终端
  cat /tmp/demo.txt
  # head 查看文件开头
  head -5 /tmp/demo.txt
  # tail 查看文件结尾 常用参数-f文件内容更新后，显示信息同步更新
  tail -5 /tmp/demo.txt
  tail -f /tmp/demo.txt
  # wc 统计文件内容信息
  wc -l /tmp/demo/txt
  ```

* 打包和压缩

  ```shell
  # c打包、x解包、f指定操作类型为文件
  tar cf /xxx/x.tar /dir
  ls -lh /xx.tar
  # 打包并压缩
  tar czf /xxx/x.tar.gz /dir
  tar -zcvf /xxx/x.tar.gz /dir
  # 解压
  tar -zxvf /xxx/x.tar -C /root
  ```

* vim

  ```shell
  # 正常模式
  yy 复制
  p 粘贴
  dd 剪切
  u 撤销
  # 插入模式
  # 命令模式
  :wq    保存退出
  :q!    不保存退出
  /name  查找 n向下查找  shift + n 向上查找
  ?name  查找
  :%s/x/X/g  所有x替换为X
  # 可视模式
  v           字符可视模式
  V           行可视模式
  ctrl + v    块可视模式
  	配合d 和 I 命令可进行块的便利操作
  ```

* 用户权限管理

  ```shell
  # useradd 新建用户
  useradd woody
  id root
  id woody
  ls -a /home/woody
  ls /etc/passwd
  # userdel 删除用户
  userdel woody
  userdel -r woody     # 删除用户对应文件
  # passwd 修改用户密码
  passwd woody
  # usermod 修改用户属性
  usermod -d /home/woody1 woody
  # chage 修改用户属性
  
  # groupadd   新建用户组
  groupadd group1
  usermod -g group1 woody
  useradd -g group1 woody2
  id woody
  # groupdel   删除用户组
  
  # 临时用户切换
  su - woody
  ```

# 21

