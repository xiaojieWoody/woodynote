* swap分区

  * Linux内核为了提高读写效率与速度，会将文件在内存中进行缓存，这部分内存就是Cache Memory(缓存内存)。即使你的程序运行结束后，Cache Memory也不会自动释放。这就会导致你在Linux系统中程序频繁读写文件后，你会发现可用物理内存变少。当系统的物理内存不够用的时候，就需要将物理内存中的一部分空间释放出来，以供当前运行的程序使用。那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到Swap空间中，等到那些程序要运行时，再从Swap分区中恢复保存的数据到内存中。这样，系统总是在物理内存不够时，才进行Swap交换
  * free -m
  * Swap交换分区的性能依然比不上物理内存，目前的服务器上RAM基本上都相当充足，那么是否可以考虑抛弃Swap交换分区，是否不需要保留Swap交换分区呢

* yaml

  * 语法规则

    * 大小写敏感
    * 使用缩紧表示层级关系
    * 缩进时不允许使用Tab键，只允许使用空格
    * 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
    * `#` ，表示注释，从这个字符一直到行尾，都会被解析器忽略

  * 支持的数据结构

    * 对象：键值对的集

      ```shell
      animal: pets
      # 转为 JavaScript 如下
      { animal: 'pets' }
      
      hash: { name: Steve, foo: bar }
      # 转为 JavaScript 如下
      { hash: { name: 'Steve', foo: 'bar' } }
      ```

    * 数组：一组连词线开头的行，构成一个数组

      ```shell
      - Cat
      - Dog
      - Goldfish
      # 转为 JavaScript 如下
      [ 'Cat', 'Dog', 'Goldfish' ]
      
      # 数据结构的子成员是一个数组，则可以在该项下面缩进一个空格
      -
       - Cat
       - Dog
       - Goldfish
      # 转为 JavaScript 如下
      [ [ 'Cat', 'Dog', 'Goldfish' ] ]
      
      # 数组也可以采用行内表示法
      animal: [Cat, Dog]
      # 转为 JavaScript 如下
      { animal: [ 'Cat', 'Dog' ] }
      ```

    * 纯量：单个的，不可再分的值

      ```shell
      字符串、布尔值、整数、浮点数、Null、时间、日期
      # null用~表示
      parent: ~ 
      # YAML 允许使用两个感叹号，强制转换数据类型
      e: !!str 123
f: !!str true
      # 转为 JavaScript 如下
      { e: '123', f: 'true' }
      
      # 字符串
      # 如果字符串之中包含空格或特殊字符，需要放在引号之中
      str: '内容： 字符串'
      # 单引号和双引号都可以使用，双引号不会对特殊字符转义
      # 单引号之中如果还有单引号，必须连续使用两个单引号转义
      str: 'labor''s day' 
      # 字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格
      str: 这是一段
        多行
        字符串
      # 转为 JavaScript 如下
      { str: '这是一段 多行 字符串' }
      
      # 多行字符串可以使用|保留换行符，也可以使用>折叠换行
      this: |
        Foo
        Bar
      that: >
        Foo
        Bar
      # 转为 JavaScript 如下
      { this: 'Foo\nBar\n', that: 'Foo Bar\n' }
      
      # +表示保留文字块末尾的换行，-表示删除字符串末尾的换行
      s1: |
        Foo
      
      s2: |+
        Foo
      
      
      s3: |-
        Foo
      # 转为 JavaScript 代码如下
      { s1: 'Foo\n', s2: 'Foo\n\n\n', s3: 'Foo' }
      
      # 字符串之中可以插入 HTML 标记
      message: |
      
        <p style="color: red">
          段落
        </p>
      # 转为 JavaScript 代码如下
      { message: '\n<p style="color: red">\n  段落\n</p>\n' }
      ```
      
      ```shell
      # 引用
      # 锚点&和别名*，可以用来引用
      defaults: &defaults
        adapter:  postgres
        host:     localhost
      
      development:
        database: myapp_development
        <<: *defaults
      
      test:
        database: myapp_test
        <<: *defaults
      
      # 等同于下面的代码
      defaults:
        adapter:  postgres
        host:     localhost
      
      development:
        database: myapp_development
        adapter:  postgres
        host:     localhost
      
      test:
        database: myapp_test
        adapter:  postgres
        host:     localhost
      
      # &用来建立锚点（defaults），<<表示合并到当前数据，*用来引用锚点
      
      - &showell Steve 
      - Clark 
      - Brian 
      - Oren 
      - *showell 
      # 转为 JavaScript 代码如下
      [ 'Steve', 'Clark', 'Brian', 'Oren', 'Steve' ]
      ```

# 常用命令

```shell
# 删除退出的容器
for i in `docker ps -a|grep -i exit|awk '{print $1}'`;do docker rm -f $i;done
# 查看环境变量
printenv
# 查看系统
cat /etc/redhat-release
```

## netstat

```shell
# 查看端口是否被监听
netstat -luntp|grep 81
```

## grep

```shell
# 查找指定进程
ps -ef|grep svn
# 查找指定进程个数
ps -ef|grep svn -c
# 从文件中读取关键词进行搜索，输出test.txt文件中含有从test2.txt文件中读取出的关键词的内容行
cat test.txt | grep -f test2.txt
# 从文件中读取关键词进行搜索 且显示行号
cat test.txt | grep -nf test2.txt
# 从文件中查找关键词
grep 'linux' test.txt
grep -n 'linux' test.txt 
# 从多个文件中查找关键词
grep 'linux' test.txt test2.txt
grep -n 'linux' test.txt test2.txt 
# grep不显示本身进程
ps aux | grep ssh | grep -v "grep"
# 找出以u开头的行内容
cat test.txt |grep ^u
# 输出非u开头的行内容
cat test.txt |grep ^[^u]
# 输出以hat结尾的行内容
cat test.txt |grep hat$
# 显示包含ed或者at字符的内容行
cat test.txt |grep -E "ed|at"
# 搜索 /etc/passwd 文件下的 boo 用户，-i 强制忽略大小写
grep boo /etc/passwd
# 递归使用 grep ，在文件目录下面搜索所有包含字符串“192.168.1.5”的文件
grep -r "192.168.1.5" /etc/
#  -w 选项去强制只输出那些仅仅包含那个整个单词的行
grep -w "boo" file
# 使用 grep 命令去搜索两个不同的单词
egrep -w 'word1|word2' /path/to/file
# 统计文本匹配到的行数, -c 参数显示每个文件中匹配到的次数
grep -c 'word' /path/to/file
# 传递 -n 选项可以输出的行前加入匹配到的行的行号
grep -n 'root' /etc/passwd
# -v 选项来输出不包含匹配项的内容
grep -v bar /path/to/file
```

## find

```shell
# 查找当前目录下 test.txt 文件
find ./ -name 'test.txt'        
```

## wget

```shell
# 使用wget -O下载并以不同的文件名保存,下载网页
wget www.baidu.com -O index.html
```

## curl

```shell
# 不带有任何参数时，curl 就是发出 GET 请求
# -d参数用于发送 POST 请求的数据体
curl -d'login=emma＆password=123'-X POST https://google.com/login
# 读取data.txt文件的内容，作为数据体向服务器发送
curl -d '@data.txt' https://google.com/login
# --data-urlencode参数等同于-d，发送 POST 请求的数据体，区别在于会自动将发送的数据进行 URL 编码
# 发送的数据hello world之间有一个空格，需要进行 URL 编码
curl --data-urlencode 'comment=hello world' https://google.com/login
# -F参数用来向服务器上传二进制文件
# 原始文件名为photo.png，但是服务器接收到的文件名为me.png
curl -F 'file=@photo.png;filename=me.png' https://google.com/profile
# -k参数指定跳过 SSL 检测
# 不会检查服务器的 SSL 证书是否正确
curl -k https://www.example.com
# -o参数将服务器的回应保存成文件，等同于wget命令
curl -o example.html https://www.example.com
# -O参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名，将服务器回应保存成文件，文件名为bar.html
curl -O https://www.example.com/foo/bar.html
# -X参数指定 HTTP 请求的方法
curl -X POST https://www.example.com
```

## tar

```shell
# 压缩、打包
# gzip，1）压缩后的格式为：*.gz 2）不能保存原文件，且不能压缩目录
gzip install.log        # 压缩
gunzip buodo.gz  				# 解压
zcat install.log.gz     # 查看

# tar
-z(gzip)      用gzip来压缩/解压缩文件
-j(bzip2)     用bzip2来压缩/解压缩文件
-v(verbose)   详细报告tar处理的文件信息
-c(create)    创建新的档案文件
-x(extract)   解压缩文件或目录
-f(file)      使用档案文件或设备，这个选项通常是必选的
tar -zcvf buodo.tar.gz buodo		# 【压缩】
tar -zxvf buodo.tar.gz 					# 【解压】

# zip，1）可以压缩目录； 2）可以保留原文件；
-r(recursive)    递归压缩目录内的所有文件和目录
zip boduo.zip boduo		# 压缩文件
unzip boduo.zip				# 解压文件
zip -r Demo.zip Demo	#【压缩目录】
unzip Demo.zip				# 解压目录

# 解压
tar -zxvf /xxx/data.tar.gz -C /xxx/
# 压缩
tar -zcvf data.tar.gz /xxx/dir
```

## 其他命令

```shell
whoami
who
w
# 切换到root用户
sudo -i
# 查看ip
ip a
# 查看应用所在目录
which java
# Centos7下安装netstat
yum install net-tools
# 查看端口服务是否启动
netstat -anp |grep "3301"
# 查看端口起用情况
netstat -nltp |grep 50070
# 外部访问端口
 telnet 192.168.0.122 50070
# 查看主机名
hostname
# 查看端口
lsof -i tcp:8080
# 启动main方法
java -cp day01.jar day01.service.multiport.MultiPortServer
# 服务器间拷贝
[atguigu@hadoop101 /]$ scp -r /opt/module  root@hadoop102:/opt/module
df -h			#查看硬盘大小
du -sh *  #查看文件大小
date -s "10:10:20 2018-02-10"    # 设置时间

# linux下利用nohup后台运行jar文件包程序
# 方式一：所有输出被重定向到nohup.out的文件中
nohup java -jar XXX.jar &
# 方式二：输出重定向到out.file文件
nohup java -jar XXX.jar >temp.txt &
# 通过jobs命令查看后台运行任务
jobs
# 将某个作业调回前台控制，只需要 fg + 编号即可
fg 23

# rz/sz
# wget https://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz
yum install lrzsz
sz filename (发送文件到客户端,zmodem接收可以自行启动)
rz (从客户端上传文件到linux服务端)

# 安装vim
yum -y install vim*
```

## vim

```shell
:q!			# 不存盘退出
:wq			# 存盘退出
yyp			# 【复制】
u				# 【撤销】
ctrl+R  # 重做更改(恢复之前撤销的内容)
x				# 【删除光标所在字符】
dd			# 【删除一行】

w				# 【移动到下一单词的开头】
b				# 【移动到上一单词的开头】

0 			# 【数字零，移到当前行开头】
A				# 【来到一行的末尾】

G				# 【文章底部】
gg      # 【移动到文件开头】

o				# 【光标下一行插入一行】

Ctrl-f  # 【向前滚动一页】
Ctrl-b  # 【向后滚动一页】

:n          # 移动到第 n 行

:%s/vivian/sky/						#【替换每一行的第一个 vivian 为 sky】
:%s/vivian/sky/g					#【替换每一行中所有 vivian 为 sky】

r				# 替换所选中字符
/name		# 搜索name，按n向下搜索，N向上搜索

>>      # 右缩进
<<      # 左缩进
Ctrl-g  # 显示当前编辑文件名及行数
ddp 		# 交换光标位置的行和它的下一行
. 			# 重复上次操作
:1,$!sort # 将文件内的所有内容排序
:%!nl   # 在所有非空行前加入行号
:%!nl -ba # 在所有行前加入行号
Ctrl-p    # 自动补全

:set all 				# 查看vi或Vim中设置的所有选项的当前值
:set <option>?  # 查看特定选项 <option> 的当前值
:set nu     		# 显示行号
:set no nu  		# 取消行号显示
:set autoindent/ai # 设置自动缩进
:set no autoindent/ai  # 取消自动缩进设置
:set shiftwidth=4 # 设置缩进宽度为 4，:set sw=4
:set ignorecase/ic # 设置忽略大小
:set no ignorecase/ic # 取消忽略大小设置

# 搜索
/ 				# 在文件中向前搜索，3/str 向前搜索字串 str 并将光标移到第三个找到的串
? 				# 在文件中向后搜索
n 				# 搜索下一个
N 				# 反向搜索下一个，3N 反向搜索第三个匹配的字符串

# 保存部分内容到文件
:m,nw <file> 							# 将 m 行到 n 行部分的内容保存到文件 <file> 中
:m,nw >> <file>						# 将 m 行到 n 行的内容添加到文件 <file> 的末尾

# 临时退出再进入
# 方法一
:sh        # 退出(未保存)
exit			 # 回到退出之前的编辑状态
# 方法二
ctrl + z   # 退出
fg				 # 回到退出之前的编辑状态

#【移动光标】
G 					# 移动到文件末尾
gg          # 移动到文件开头
:0          # 移动到文件第一行
:$          # 移动到文件最后一行
w						# 移动到下一单词的开头
b						# 移动到上一单词的开头
0 					# 数字零，移到当前行开头
$ 					# 移到当前行末尾
:n          # 移动到第 n 行
) 					# 移动到当前句子的末尾
( 					# 移动到当前句子的开头
} 					# 移动当前段落的末尾
{ 					# 移到当前段落的开头
H 					# 移动到屏幕的第一行
M 					# 移动到屏幕的中间一行
L 					# 移动到屏幕的最后一行
Ctrl-f      # 向前滚动一页
Ctrl-b      # 向后滚动一页
Ctrl-u      # 向前滚动半页
Ctrl-d      # 向后滚动半页


# 替换字符串
:s/vivian/sky/ 						# 替换当前行第一个 vivian 为 sky
:s/vivian/sky/g 					# 替换当前行所有 vivian 为 sky
:n,$s/vivian/sky/ 				# 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky
:n,$s/vivian/sky/g 				# 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky，n 为数字，若 n 为 .，表示从当前行开始到最后一行
:%s/vivian/sky/						# (等同于 :g/vivian/s//sky/)替换每一行的第一个 vivian 为 sky
:%s/vivian/sky/g					# (等同于 :g/vivian/s//sky/g) 替换每一行中所有 vivian 为 sky
# 可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符
:s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/
:%s+/oradata/apras/+/user01/apras1+ # (使用+ 来 替换 / )：/oradata/apras/替换成/user01/apras1/
```

## 查看内容

```shell
# 列出文件
ls
pwd						# 当前路径
ls -l         # 文件列表显示
ls -lt        # 文件时间列表显示
ls -lrt				# 文件时间列表显示，最近的在最下
ls -lart      # a，显示隐藏文件

# cat
cat a.txt     # 显示文件所有
cat file1 file2 > file # 将几个文件合并为一个文件
-n						# 显示行号
-b						# 对非空行输出行编号
cat /etc/passwd >> test1.txt  # passwd中内容添加到test1.txt中
cat /etc/passwd > test1.txt   # 先清空，再添加
cat test1.txt | tr  '[a-z]' '[A-Z]' >test2.txt   # 文件中字符都替换成大写，然后重定向到test2.txt中
cat test2.txt | tr -d " "  # 删除空格
cat /etc/passwd | sort          # 按首字母排序
cat /etc/passwd |cut -d ":" -f 1 # 按 : 切割
cat 1.txt | uniq                 # 去重显示
cat 1.txt | uniq | wc -l         # 去重后有多少行
cat 1.txt | sort | uniq -c       # 去重排序统计

# more
more a.txt		# 只显示一屏
空格键					# 显示下一页
b							# 显示上一页
more -2 test2.txt	# 每屏显示行数
more +/day3 1.txt # 从文件中查找第一个出现"day3"字符串的行，并从该处前两行开始显示输出
more +3 log2012.log # 显示文件中从第3行起的内容
q							# 退出
man more      # 查看more有哪些功能
# 不能直接到最后一页，可以使用tail来代替

# 【less】
ctrl+f							# 【向前翻一页】
ctrl+b							# 【向后翻一页】
G 									# 移动到最后一行
g 									# 移动到第一行
/										# 字符串：向下搜索“字符串”的功能
?										# 字符串：向上搜索“字符串”的功能
q 									# 退出 less 命令

# tail
tail -10 test.txt   # 最后10行打印出来
tail -10 test.txt > test1.txt # test.txt中最后10行添加到test1.txt中
tail -f test.txt    # 动态打印

# head
head -10 test.tnxt   # 最前面10行打印出来

# echo
echo "test111" >>test.txt     # 文件中追加内容
echo "" >test1.txt      # 清空文件
>test1.txt							# 清空文件

# cp
cp test.txt test1.txt      # 复制  -r 递归复制

# touch
touch test.txt             # 对文件时间进行更新，或者创建文件

date                       # 日期

# grep
grep 'test111' test.txt -n # 【查找文件内容并显示在第几行】
grep -A 3 'FINISHED' test.txt    # 打印文件中 FINISHED 字符后三行
grep -B 3 'FINISHED' test.txt    # 打印文件中 FINISHED 字符前三行
grep -C 3 'FINISHED' test.txt    # 打印文件中 FINISHED 字符上下三行
grep -A 3 'FINISHED' test.txt --color=auto    # 高亮显示FINISHED
grep -A 3 'FINISHED' test.txt -c --color=auto # 有多少行
grep -A 3 'FINISHED' test.txt --color=auto | wc -l  # 有多少行

# find
find ./ -name 'test.txt'        # 查找当前目录下 test.txt 文件
```

```shell
# 用户、组
whoami			# 当前用户
su -testA 	# 切换用户
su - testA  # 切换用户并带其系统变量
env         # 查看系统变量
exit      # 退出用户
cat /etc/passwd  # 查看用户  用户名:passwd:UID:GID:名称:家目录:shell
id
groupadd nginxA			# 创建组
useradd nginxtest1 -g nginxA -s /bin/nologin   # 创建用户，不用登录
id nginxtest1      # 查看用户
useradd nginxtest1 -g nginxA -s /bin/bash   # 创建用户，需登录
sudo xxx          # 以系统用户身份执行  
passwd nginxtest2 # 给用户设置密码  
passwd            # 重新设置密码
userdel nginxtest1# 删除用户
cat /etc/group    # 查看组
groupdel testB    # 删除组

# 权限 
# r   w   x
# 读4 写2 执行1
# 用户 组 其他
chown nginxtest2 test1    # 更换文件所属用户
ls -lrt
chown nginxtest2:nginxA test1 # 更换文件所属组
chmod u+x test1         # 用户加上x权限
chmod u-x test1         # 用户减去x权限
chmod g+x test1         # 组加上x权限
chmod 760 tset1         # 设置权限
```

```shell
hostname
ssh root@192.168.0.110      # linux之间交互
# 免密登录
# 192.168.0.111
cd .ssh/
ssh-keygen
cat id_rsa.pub              # 把公钥放到对方
# 对方  192.168.0.110
cd .ssh/
ssh-keygen
touch authorized_keys       # 将192.168.0.111的id_rsa.pub内容填进去
# 192.168.0.111 
ssh 192.168.0.110
```

```shell
# 检测端口，判断服务的连接情况
netstat -anlp | grep 21
netstat -anltp | grep vsftpd
ps -ef|grep 30763
```

```shell
# ftp  文件传输服务         sftp 22, ftp 21
# 192.168.0.111
service vsftpd start      # 启动ftp服务
yum install ftp
useradd ftpuser1
passwd ftpuser1
# 192.168.0.110
ftp 192.168.0.111
Name:ftpuser1
Password:ftpuser1
put
get

# scp
scp xxx root@192.168.0.111:/home/test/1.txt
scp root@192.168.0.111:/home/test/1.txt /root/

# crontab 定时任务       分 时 日 月 周 crontab
crontab -e
0 22 * * * /usr/sbin/vsftpd stop   # 每天12点执行
#30 6,12 * * *                 # 每天6点30和12点30执行
#10 6-12 * * *                 # 每天6点到12点之间每10分钟执行
#*/5 * * * *                   # 每5分钟执行一次
crontab -l

# 后台运行
jobs				# 查看后台运行程序
tail -f /var/log/message &      # 后台运行程序
fg 2        # jobs后，调度后台第2个运行程序
bg 1        # jobs后，调度后台第1个程序运行

# 系统级服务 service
cd /etc/init.d/
ls

chkconfig --list | grep vsftpd
chkconfig --level 35 vsftpd on    # 3和5级别启动
chkconfig vsftpd on
chkconfig vsftpd off
```









