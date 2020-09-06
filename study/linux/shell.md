```shell
# 显示行号
cat -n file
# 可以先把语法框架搭起来，转义字符最后加
# sed -n '//,//p' my.cnf | grep -v ^# | grep -v ^$ | grep -v ""
sed -n '/\[mysqld\]/,/\[.*\]/p' my.cnf | grep -v ^# | grep -v ^$ | grep -v "\[.*\]"
# 正则表达式
```

# 目标

* 基本监控系统脚本编写（CPU、内存、IO等）
* 后台服务监控、启动、停止脚本、数据备份脚本
* 利用grep、sed和awk的复杂用法处理文本
* 功能函数编写、主流程设计
* 具备复杂功能的大型脚本工具编写

# 知识点

## 1. 变量的高级用法

* 变量替换和测试

  ```shell
  var1="I love you,Do you love me"
  
  # 从变量【开头】进行规则匹配，将符合最【短】的数据删除
  # ${变量名#匹配规则}
  var2=${var1#*ov}				# e you,Do you love me
  
  # 从变量【开头】进行规则匹配，将符合最【长】的数据删除
  # ${变量名##匹配规则}
  var3=${var1##*ov}				# e me
  
  # 从变量【尾部】进行规则匹配，将符合最【短】的数据删除
  # ${变量名%匹配规则}
  var4=${var1%ov*}				# I love you,Do you l
  
  # 从变量【尾部】进行规则匹配，将符合最【长】的数据删除
  # ${变量名%%匹配规则}
  var5=${var1%%ov*}				# I l
  
  #【1】 变量内容符合旧字符串，则【第一个】旧字符串会被新字符串取代
  # ${变量名/旧字符串/新字符串}
  var6=${var1/love/LOVE}  # I LOVE you,Do you love me
  
  #【2】变量内容符合旧字符串，则【全部的】旧字符串会被新字符串取代
  # ${变量名//旧字符串/新字符串}
  var7=${var1//love/LOVE} # I LOVE you,Do you LOVE me
  ```

  ![image-20200301224455886](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200301224455886.png)

* 字符串处理

  * 计算字符串长度

    ```shell
    var5=love
    # 方法一
    ${#string}
    echo ${#var5}       				# 4
    # 方法二，string有空格，则必须加双引号  len=`expr length "$string"`
    expr length "$string"
    expr length "$var5" 				# 4
    var8=`expr length "$var5"` 	# echo $var8
    ```

  * 获取子串在字符串中的索引位置

    * `expr index $string $substring`

      * 子串拆分成一个个字符，一个一个的查找，直到找到

        ```shell
        var1="quickstart is a app"
        ind=`expr index "$var1" start`				# 6
        ind=`expr index "$var1" uniq`					# 1，而不是2
        ind=`expr index "$var1" cnk`					# 4
        ```

    * `expr match $string $substring`

      * 从头匹配才有效

        ```shell
        var1="quicstart is a app"
        sub_len=`expr match "$var1" app`				# 0
        sub_len=`expr match "$var1" quick`			# 0
        sub_len=`expr match "$var1" quic`				# 4
        sub_len=`expr match "$var1" quic.*` 		# 18
        sub_len=`expr match "$var1" quicstart` 	# 9
        ```
      

  * 抽取子串

    * ```shell
      var1="I love you,Do you love me"
      # 从string中的position开始，索引下标从0开始
      # ${string:position}
      echo ${var1:0}								# I love you,Do you love me
      # 从position开始，匹配长度为length
      # ${string:position:length}
      echo ${var1:0:4}							# I lo
      # 从右边开始匹配，注意有空格
      # ${string: -position}
      echo ${var1: -2}							# me
      # 从左边开始匹配
      # ${string:(position)}
      echo ${var1:(1)}							# love you,Do you love me
      
      # 从position开始，匹配长度为length，索引下标从1开始
      # expr substr $string $position $length
      echo `expr substr "$var1" 2 4`	# lov
      ```

* 需求描述

  * 变量`string="Bigdata process framework is Hadoop,Hadoop is an open source project"`，执行脚本后，打印输出`string`字符串变量，并给出用户以下选项：

    1. 打印`string`长度
    2. 删除字符串中所有的`Hadoop`
    3. 替换第一个`Hadoop`为`Mapreduce`
    4. 替换全部`Hadoop`为`Mapreduce`

  * ==用户输入数字1|2|3|4，可以执行对应项的功能：输入q|Q则退出交互模式==

    * 思路分析

      1. 将不同的功能模块划分，并编写函数
         * `function print_tips`
         * `function len_of_string`
         * `function del_hadoop`
         * `function rep_hadoop_mapreduce_first`
         * `function rep_hadoop_mapreduce_all`
      2. 实现第一步所定义的功能函数
      3. 程序主流程设计

      ```shell
      #!/bin/bash
      string="Bigdata process framework is Hadoop,Hadoop is an open source project"
      
      function print_tips
      {
      	echo "***********************************"
      	echo "(1) 打印string长度"
      	echo "(2) 删除字符串中所有的Hadoop"
      	echo "(3) 替换第一个Hadoop为Mapreduce"
      	echo "(4) 替换全部Hadoop为Mapreduce"
      	echo "***********************************"
      }
      
      function len_of_string
      {
      	echo "${#string}"
      }
      
      function del_hadoop 
      {
      	echo "${string//Hadoop/}"
      }
      
      function rep_haddop_mapreduce_first 
      {
      	echo "${string/Hadoop/Mapreduce}"
      }
      
      function rep_hadoop_mapreduce_all
      {
      	echo "${string//Hadoop/Mapreduce}"
      }
      
      while true
      do
      	echo "【string=${string}】"
      	echo
      	print_tips
      	read -p "Pls input your choice(1|2|3|4|q|Q)：" choice
      
      	case $choice in
      		1)
      			len_of_string
      			;;
      		2)
      			del_hadoop
      			;;
      		3)
      			rep_haddop_mapreduce_first
      			;;
      		4)
      			rep_hadoop_mapreduce_all
      			;;
      		q|Q)
      			exit
      			;;
      		*)
      			echo "Error,input only in {1|2|3|4|q|Q}"
      			;;
      	esac
      done	
      ```

* 命令替换

  ```shell
  # 方法一
  `command`
  # 方法二
  $(command)
  # ``和$()两者是等价的，但推荐初学者使用$()，易于掌握；缺点是极少数UNIX可能不支持，但``都支持
  
  # $(())主要用来进行【整数运算】，包括加减乘除，引用变量前面可以加$，也可以不加$
  num5=$(((100+20) / 30))
  echo $num5
  
  num1=20
  num2=30
  
  ((num1++))
  ((num2--))
  
  echo $num1  # 21
  echo $num2  # 29
  
  num3=$((num1+num2))
  echo $num3  # 50
  num4=$((num1+num2*2))
  echo $num4	# 79
  ```

  ```shell
  #【例子1】：获取系统的所有用户并输出
  # -d 指定分隔符；-f 选择第几个片段
  cat /etc/passwd
  cat /etc/passwd | cut -d ":" -f 1
  
  #!/bin/bash
  index=1
  
  for user in `cat /etc/passwd | cut -d ":" -f 1`
  do 
  	echo "This is $index user：$user"
  	index=$(($index+1))
  done	
  ```

  ```shell
  #【例子2】：根据系统时间计算今年或明年
  date +%Y
  echo "This is $(date +%Y) year"
  echo "This is $(($(date +%Y) + 1)) year"
  ```

  ```shell
  # 【例子3】：根据系统时间获取今年还剩下多少星期，已经过了多少星期
  man date
  #!/bin/bash
  echo "This year have passed $(date +%j) days"
  # echo "This year have passed $(($(date +%j) /7)) week"
  # date_test.sh:行5: 365 - 091: 数值太大不可为算数进制的基 （错误符号是 "091"）,需转成10进制
  echo "This year have passed $((10#$(date +%j) /7)) week"
  # echo "There is $((365 - $(date +%j))) days before new year"
  echo "There is $((365 - 10#$(date +%j))) days before new year"
  # echo "There is $(((365 - $(date +%j))/7)) weeks before new year"
  echo "There is $(((365-10#$(date +%j))/7)) weeks before new year"
  ```

  ```shell
  #【例子4】：判定nginx进程是否存在，若不存在则自动拉起该进程
  #!/bin/bash
  # 获取nginx进程个数，-v过滤掉，wc -l统计行数，-w只显示字数
  nginx_process_num=$(ps -ef | grep nginx | grep -v grep | wc -l)
  if [$nginx_process_num -eq 0];then
  	systemctl start nginx
  fi	
  ```

* 有类型变量

  * declare命令和typeset命令两者等价
  * declare、typeset命令都是用来定义变量类型的

  ```shell
  # declare命令参数表
  var1="hello world"
  # 将变量设为只读，-r                            							   declare -r var1
  # 将变量设为整数，-i
  # 将变量定义为数组，-a
  # 显示此脚本前定义过的所有函数及内容，-f           									declare -f
  # 仅显示此脚本前定义过的函数名，-F                									declare -F
  # 将变量声明为环境变量，脚本中可以直接使用，-x                       declare -x var
  # 取消声明的变量
  declare +r
  declare +i
  declare +a
  declare +X
  ```

  ```shell
  num1=10
  num2=$num1+20
  echo $num2               # 10+20
  expr $num1 + 10					 # 20
  
  declare -i num3
  num3=num1+90
  echo $num3								# 100
  ```

  ```shell
  declare -a array
  array=("jones" "mike" "kobe" "jordan")
  echo ${array[@]}          # 所有元素，下标从0开始
  echo ${array[0]}          # 第一个元素
  echo ${#array[@]}         # 数组大小
  echo ${#array[0]}         # 数组第一个元素长度
  array[0]="lily"           # 赋值
  unset array[2]            # 清空元素
  unset array               # 清空数组
  ${array[@]:1:4}           # 分片访问，显示数组下标索引从1开始到3的3个元素
  ${array[@]/an/AN}         # 将数组中所有元素内包含kobe的子串替换为mcgic
  for v in ${array[@]}      # 数组遍历
  do
  	echo $v
  done	
  ```

* Bash数学运算之expr

  ```shell
  # 方法一【推荐】
  expr $num1 operator $num2
  echo `expr $num1 + $num2`
  # 方法二
  $(($num1 operator $num2))
  # 操作符，注意添加转义\
  expr $num1 \| $num2
  expr $num1 \& $num2
  \<、\<=、\>、\>=、=、!=
  +-*/%
  ```
```shell
# num1不为空且非0，返回num1；否则返回num2
num1 | num2
# num1不为空且非0，返回num1；否则返回0
num1 & num2
# num1小于num2，返回1；否则返回0
num1 < num2
echo `expr $num1 \< $num2`
# num1小于等于num2，返回1；否则返回0
num1 <= num2
# num1等于num2，返回1；否则返回0
num1 = num2
# num1不等于num2，返回1；否则返回0
num1 != num2
# num1大于num2，返回1；否则返回0
num1 > num2
# num1大于等于num2，返回1；否则返回0
num1 >= num2
```

  ```shell
num3=`expr $num1 + $num2`
num3=$(($num1+$num2))
echo $num3
  ```

  ```shell
  # 【例子】提示用户输入一个正整数num，然后计算1+2+3+...+sum的值;必须对num是否为正整数做判断，不符合应当允许再次输入
  #sum.sh
  #!/bin/bash
  
  while true
  do
          read -p "Please input a positive number：" num
          # 如果不是整数和1相加，则会报错
          expr $num + 1 &> /dev/null
  
          # 上一步命令执行不正确的话返回的是非0数
          if [ $? -eq 0 ];then
                  # 判断是否是正整数
                  if [ `expr $num \> 0` -eq 1 ];then
                          for((i=1;i<=$num;i++))
                          do
                                  sum=`expr $sum + $i`
                          done
                          echo "1+2+...+$num = $sum"
                          exit
                  fi
          fi
          echo "error,input ellegal"
          continue
  done
  ```

* Bash数学运算之bc
  * bc是bash内建的运算器，支持浮点数运算
  * 内建变量scale可以设置，默认为0

* ```shell
  # bc 操作符对照表
  num1 + num2 求和
  num1 - num2 求差
  num1 * num2 求积
  num1 / num2 求商
  num1 % num2 求余
  num1 ^ num2 指数运算
  ```

  * ```shell
    which bc
    bc
    23 + 2
    25
    # 指定精度
    scale=2
    23/4
    # 
    echo "23+32" | bc
    55
    echo "scale=4;23.3/3.5" | bc
    6.6571
    ```

  * ```shell
    # bc_test.sh
    #!/bin/bash
    # bc是bash内建的运算器，支持浮点数运算,内建变量scale可以设置，默认为0
    read -p "num1:" num1
    read -p "num2:" num2
    # echo "scale=4;$num1 / $num2" | bc
    num3=`echo "scale=4;$num1 / $num2" | bc`
    echo "$num1 / $num2 = $num3"
    ```

## 2. 函数的高级用法

* 函数定义和使用

```shell
# 格式一
name()
{
	command1
	command2
	...
}
# 格式二
function name
{
	command1
	command2
	...
}
# 直接使用函数名调用
# 函数内部可以直接使用参数$1、$2...$n
# 调用函数:function_name $1 $2
```

```shell
#【例子】写一个监控nginx的脚本；如果Nginx服务宕掉，则该脚本可以检测到并将进程启动
# echo $? 为 0 表示上条命令正确执行
# ps -ef | grep nginx | grep -v grep
# 执行的脚本也会作为一个子进程在系统中执行，所以如果脚本名中含有nginx，则ps -ef|grep nginx就不正确
# echo $$   # 返回执行脚本的子进程ID
# nginx_daemon.sh
#!/bin/bash
this_pid=$$
while true
do
# 过滤掉子进程
ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
if [ $? -eq 0 ];then
	echo "Nginx is running well"
	sleep 3
else
	systemctl start nginx
	echo "Nginx is down,Start it..."
fi	
done
# 后台启动
nohup sh nginx_daemon.sh &
tail -f nohup.out
# netstat -tnlp | grep :80
```

* 向函数传递参数

```shell
function name
{
	echo "Hello $1"
	echo "Hello $2"
}
# 函数调用 name Lily Allen
function greeting
{
	echo "Hello $1"
}
greeting Zhangsan
greeting Lisi
```

```shell
#【例子】实现计算器的功能，可以进行+-*/四种计算，例如 sh calculate.sh 30 + 40
#!/bin/bash
#
function calcu
{
        case $2 in
                +)
                        echo "`expr $1 + $3`"
                        ;;
                -)
                        echo "`expr $1 - $3`"
                        ;;
                \*)
                        echo "`expr $1 \* $3`"
                        ;;
                /)
                        echo "`expr $1 / $3`"
                        ;;
        esac
}
calcu $1 $2 $3

# sh calculate.sh 20 - 30
```

* 函数返回值

```shell
# 方法一
# 只能返回1-255的整数，通常只是用来供其他地方调用获取状态，因此通常仅返回0-成功或1-失败
return   
# 方法二
# 可以返回任何字符串结果，通常用于返回数据，比如一个字符串值或者列表值
echo
```

```shell
#!/bin/bash
this_pid=$$
function is_nginx_running
{
	ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
	if [ $? -eq 0 ];then
			return
  else
  		return 1
  fi
}
is_nginx_running && echo "Nginx is running" || echo "Nginx is stoped"
```

```shell
# cat /etc/passwd | cut -d: -f1

#!/bin/bash
#
function get_users
{
		users=`cat /etc/passwd | cut -d: -f1`
		echo $users
}
user_list=`get_users`
for u in $user_list
do
	echo "The $index user is : $u"
	index=$(($index+1))
done	
```

* 全局变量和局部变量

```shell
# 全局变量
	# 不做特殊声明，Shell中变量都是全局变量
	# 大型脚本程序中函数中慎用全局变量
	# 函数中声明的变量，如var2=87，在调用函数后，可以直接访问该变量，echo $var2
# 局部变量
	# 定义变量时，使用local关键字
```

* 函数库

```shell
# 经验之谈
1. 库文件名的后缀是任意的，但一般使用.lib
2. 库文件通常没有可执行选项
3. 库文件无需和脚本在同级目录，只需在脚本中引用时指定
4. 第一行一般使用#!/bin/echo，输出警告信息，避免用户执行

# 定义一个函数库，该函数库实现以下几个函数
1. 加法函数add
	add 12 89
2. 减法函数reduce
	reduce 80 20
3. 乘法函数multiple
	multiple 12 10
4. 除法函数divide
	divide 9 3
5. 打印系统运行情况的函数sys_load，该函数可以显示内存运行情况，磁盘使用情况
```

```shell
# 定义函数库
# 名称：base_function
function add
{
        echo "`expr $1 + $2`"
}

function reduce
{
        echo "`expr $1 - $2`"
}

function multiple
{
        echo "`expr $1 \* $2`"
}

function divide
{
        echo "`expr $1 / $2`"
}

function sys_load
{
	echo "Memory Info"
	echo 
	free -m
	echo
	
	echo "Disk Usage"
	echo
	df -h
	echo
}

# 使用
. base_function
add 12 23
reduce 43 12
multiple 12 3
divide 54 2
sys_load

# 调用：sh calculate_base.sh
# calculate_base.sh
#!/bin/bash
#
# 函数库的绝对路径
. /dev_env/shell_test/base_function

add 12 23
reduce 90 30
multiple 12 2
divide 82 4
sys_load
```

## 3. Shell脚本工具讲解（find、xargs、wget等）

* 文件查找之find命令

```shell
# 语法格式
find [路径] [选项] [操作]
# 选项
-name					# 【根据文件名查找】
		find /etc -name '*conf'
		find . -iname aa						# 不区分大小写
-perm					# 根据文件权限查找
-prune				# 该选项可以排除某些查找目录
-user					# 根据文件属性查找
		find . -user hdfs						# 查找文件属主为hdfs的所有文件
-group				# 根据文件组查找
		find . -group yarn					# 查找文件属组为yarn的所有文件
-mtime -n| +n	# 【根据文件更改时间查找】
		-n				# n天以内修改的文件
				find /etc -mtime -5 -name '*.conf'    
				find /etc -mtime +10 -user root
		+n				# n天以外修改的文件										
		n					# 正好n天修改的文件
-mmin
		-n				# n分钟以内修改的文件
				find /etc -mmin -30 -type d
		+n				# n分钟以外修改的文件
				find /etc -mmin +30
-nogroup			# 查找无有效属组的文件
-nouser				# 查找无有效属主的文件
-newer file1 ! file2	# 查找更改时间比file1新但比file2旧IDE文件
-type					# 【按文件类型查找】
		find . -type f							# 文件
		find . -type d							# 目录
		find . -type c							# 字符设备文件
		find . -type b							# 块设备文件
		find . -type l							# 链接文件
		find . -type p							# 管道文件
-size -n +n		# 【按文件大小查找】
		-n			# 小于大小n的文件
				find /etc -size -10000c				# 查找/etc目录下小于10000字节的文件
		+n 			# 大于大小n的文件
				find /etc -size +1M						# 查找/etc目录下大于1M的文件
-mindepth n		# 【从n级子目录开始搜索】
		find /etc -mindepth 3							# 在/etc下的3级子目录开始搜索
-maxdepth n		# 【最多搜索到n-1级子目录】
		find /etc -maxdepth 3 -name '*.conf'
		find ./etc/ -type f -name '*.conf' -size +10k -maxdepth 2
```

```shell
# 了解选项
-nouser				# 查找没有属主的用户
		find . -type f -nouser
-nogroup			# 查找没有属组的用户
		find . -type f -nogroup
-perm
		find . -perm 664
-prune
	通常和-path一起使用，用于将特定目录排除在搜索条件之外
	例子1:查找当前目录下所有普通文件，但排除test目录
		find . -path ./etc -prune -o -type f
	例子2:查找当前目录下所有普通文件，但排除etc和opt目录
  	find . -path ./etc -prune -o -path ./opt -prune -o -type f
	例子3:查找当前目录下所有普通文件，但排除etc和opt目录，但属主为hdfs
  	find . -path ./etc -prune -o -path ./opt -prune -o -type f -a -user hdfs
	例子4:查找当前目录下所有普通文件，但排除etc和opt目录，但属主为hdfs，且文件大小必须大于500字
  	find . -path ./etc -prune -o -path ./opt -prune -o -type f -a -user hdfs -a -size +500c
-newer file1
	例子:find /etc -newer a 
```

```shell
# 操作
-print				# 打印输出
-exec					# 对搜索到的文件执行特定的操作，格式为-exec 'command' {} \;
	例子1:搜索/etc下的文件(非目录)，文件名以conf结尾，且大于10k，然后将其删除
		find ./etc/ -type f -name '*.conf' -size +10k -exec rm -rf {} \;
	例子2:将/var/log/目录下以log结尾的文件，且更改时间在7天以上的删除
  	find /var/log/ -name '*.log' -mtime +7 -exec rm -rf {} \;
	例子3:搜索条件和例子1一样，只是不删除，而是将其复制到/root/conf目录下
  	find ./etc/ -size +10k -type f -name '*.conf' -exec cp {} /root/conf/ \;
-ok						# 和exec功能一样，只是每次操作都会给用户提示
```

```shell
# 逻辑运算符
-a						#与
-o						#或
-not|!				#非
例子1：查找当前目录下，属主不是hdfs的所有文件
	find . -not -user hdfs | find  . ! -user hdfs
例子2：查找当前目录下，属主属于hdfs，且大小大于300字节的文件
	find . -type f -a -user hdfs -a -size +300c
例子3:查找当前目录下的属主为hdfs或者以xml结尾的普通文件
	find . -type f -a \(-user hdfs -o -name '*.xml' \)
```

* find、locate、whereis和which总结及适用场景分析
  * find，查找某一类文件，比如文件名部分一致，功能强大，速度慢
  * locate，只能查找单个文件，功能单一，速度快
  * whereis，查找程序的可执行文件、帮助文档等，不常用
  * which，只查找程序的可执行文件，常用于查找程序的绝对路径

```shell
# locate命令介绍
1. 文件查找命令，所属软件包mlocate
2. 不同于find命令是在整块磁盘中搜索，locate命令在数据库文件中查找
3. find是默认全部匹配，locate则是默认部分匹配
# updatedb命令
1. 用户更新 /var/lib/mlocate/mlocate.db
2. 所使用配置文件 /etc/updatedb.conf
3. 该命令在后台cron计划任务中定期执行
# whereis
-b				# 只返回二进制文件
-m				# 只返回帮助文档文件
-s				# 只返回源代码文件
# which
仅查找二进制程序文件
-b				# 只返回二进制文件
```

## 4. grep和egrep过滤器详解

* grep语法格式

  ```shell
  # 第一种形式
  grep [option] [pattern] [file1,file2]
  # 第二种形式
  command | grep [option] [pattern]
  ```

  ```shell
  -v				# 不显示匹配pattern行信息
  		grep -vi pattern file
  		grep "py" file
  		grep "py.*" file
  		grep -E "python|PYTHON" file       # 不加-E，则｜（正则）不生效
  -i				# 搜索时忽略大小写
  -n				# 显示行号
  -r				# 递归搜索
  		grep -r love            # 当前目录下递归搜索所有包含love的数据
  -E				# 支持扩展正则表达式
  -F				# 不按正则表达式匹配，按照字符串字面意思匹配
  -c        # 只显示匹配行总数，不显示具体内容
  -w				# 匹配整词
  -X				# 匹配整行
  		grep -x "i love python" file
  -l				# 只显示文件名，不显示内容
  -s				# 不显示错误信息
  
  grep -E 'python|PYTHON' file
  # 等价
  egrep 'python|PYTHON' file
  
  cat /etc/passwd | grep "bash"
  ```

* `grep`和`egrep`
  * `grep`默认不支持扩展正则表达式，只支持基础正则表达式
  * 使用`grep -E`可以支持扩展正则表达式
  * 使用egrep可以支持扩展正则表达式，与`grep -E`等价

## 5. Sed流编辑器详解

  ```shell
# sed（Stream Editor），流编辑器，对标准输出或文件逐行进行处理
# 依据特定的匹配模式，对文本逐行匹配，并对匹配行进行特定处理
# 第一种形式
stdout | sed [option] "pattern command"
# 【第二种形式】
sed [option] "/pattern/command" file
  ```

```shell
# option
-n				# 只打印模式匹配行
-e				# 直接在命令行进行sed编辑，默认选项
-f				# 编辑动作保存在文件中，指定文件执行
-r				# 支持扩展正则表达式
-i				# 直接修改文件内容
```

```shell
# 'p'打印，加上默认会输出每行内容，所以每行打印两次
sed 'p' sed.txt
# 每行打印一次
sed -n 'p' sed.txt
# 只打印每行中包含 python的行
sed -n '/python/p' sed.txt
# 打印包含python和PYTHON的行
sed -n -e '/python/p' -e '/PYTHON/p' sed.txt
# 编辑动作保存在文件中
		# edit.sed
		/python/p
sed -n -f edit.sed sed.txt
# 支持扩展正则表达式，打印包含python或PYTHON的行
sed -n -r '/python|PYTHON/p' sed.txt
# 将行中包含的love改为like，并输出，但源文件没有改
sed -n 's/love/like/g;p' sed.txt
# 修改源文件中内容
sed -i 's/like/love/g' sed.txt
```

```shell
# pattern
10command							# 匹配到第10行
		sed -n "17p" file						# 打印file文件的第17行
10,20command					# 匹配从第10行开始，到第20行结束
		sed -n "10, 20p" file				# 打印file文件的10到20行
10,+5command					# 匹配从第10行开始，到第16行结束
		sed -n "10,+5p" file				# 打印file文件中从第10行开始，往后面加5行的所有行
/pattern1/command			# 【匹配到pattern1的行】
		sed -n "/^root/p" file			# 打印file文件中以root开头的行
/pattern1/,/pattern2/command		# 【匹配到pattern1的行开始，到匹配到pattern2的行结束】
		sed -n "/^ftp/,/^mail/p" file		# 打印file文件中第一个匹配到以ftp开头的行，到第二个匹配到mail的行
10,/pattern1/command	# 匹配从第10行开始，到匹配到pattern1的行结束
		sed -n "4,/^hdfs/p" file		# 打印file文件中从第4行开始匹配，直到以hdfs开头的行
/pattern1/,10command	# 匹配到pattern1的行开始，到第10行匹配结束
		sed -n "/root/,10p" file		# 打印file文件中匹配root的行，直到第10行结束
```

```shell
# 编辑命令
# 查询
p						# 打印

# 增加
a						# 行后追加一行
		sed -i '/\/bin\/bash/a This is a user' passwd
i						# 行前追加
		sed -i '/^hdfs/,/^yarn/i AAAAAAAAAAA' passwd
r						# 外部文件读入，行后追加
		#list文件
    cat list
    	First Line(YYYYYYYYY)
    	Second Line(XXXXXXXX)
		sed -i '/root/r list' passwd    	
w						# 匹配行写入外部文件
		sed '/\/bin\/bash/w /tmp/user_login.txt' passwd
		
# 删除
d						# 删除
		sed -i '1d' passwd						# 删除第1行
		sed -i '1,3d' passwd					# 删除1-3行
		sed -i '/\/sbin\/nologin/d' passwd				# 删除包含/sbin/nologin的行
		sed -i '/^mail/,/^ftp/d' passwd						# 删除以mail开头，到以ftp开头之间的行
		
# 修改		
s/pattern/string			# 将行内第一个匹配的pattern替换为string
		sed -i 's/root/ROOT/' passwd
s/pattern/string/g		# 【将行内全部匹配的pattern替换为string】
		sed -i 's/\/bin\/bash\/BIN\/BASH/g' passwd
s/pattern/string/2		# 将行内前2个匹配的pattern替换为string
s/pattern/string/2g		# 将行内第2个匹配开始的所有pattern替换为string
		sed -i 's/root/ROOT/2g' passwd
s/pattern/string/ig		# 【将行内匹配的pattern全部替换为string，忽略大小写】

# 其他编辑命令
=											# 显示行号
sed -n '/\/sbin\/nologin/=' passwd              # 只显示匹配到的行的行号
```

* 反向引用

```shell
# 引用模式匹配到的整个串
&和\1
# 在file中搜寻以1开头，然后跟两个任意字符，以e结尾的字符串
sed "s/1..e/&r/g" file
# 和上面实现一样的功能，使用\1代表搜寻到的字符串
sed "s/\(1..e\)/\1r/g" file
# 上面两种方式实现了一样的功能，分别使用&和\1引用前面匹配到的整个字符串
# 两者区别在于&只能表示匹配到的完整字符串，只能引用整个字符串；而\1可以使用()对匹配到的字符串
# 例如：如果我们仅想要替换匹配到的字符串的一部分，name必须使用\1这种方式，不能使用&
# 查找test.txt文件中以1开头，紧接着跟两个任意字符，再接一个e的字符串。将找到的字符串中开头的
sed "s/1\(..e\)/L\1/g" test.txt

# 将hadXXp替换成hadoops
sed -i 's/had..p/hadoops/g' str.txt
# 保留hadXXp，后面加s，如hadXXps
sed -i 's/had..p/&s/g' str.txt
sed -i 's/\(had..p\)/\1s/g' str.txt
# 两种都可以，&代表前面的整体，而\1更灵活，代表前面括号内容
sed -i 's/\(had\)../\1oop/g' str.txt           # hadoop
```

* sed中引用变量时注意事项

  1. 匹配模式中存在变量，则建议使用双引号
  2. sed中需要引入自定义变量时，如果外面使用单引号，则自定义变量也必须使用单引号

  ```shell
  #!/bin/bash
  #
  old_str=hadoop
  new_str=HADOOP
  
  sed -i "s/$old_str/$new_str/g" str.txt
  # 或者  sed -i 's/'$old_str'/'$new_str'/g' str.txt
  ```

* ==利用sed查询特定内容==

  ```shell
  #【练习】处理一个类似MySQL配置文件my.cnf的文本，示例如下；
  # 编写脚本实现以下功能：输出文件有几个段，并且针对每个段可以统计配置参数总个数
  
  # [client]
  # port	=	3306
  # socket= /tmp/mysql.socket
  
  # [server]
  # innodb_buffer_pool_size = 91750M
  # ...
  
  # 预想输出结果:
  1:client 2
  2:server 12
  3:mysqld 12
  4:mysqld_safe 7
  5:embedded 8
  6:mysqld-5.5 9
  
  # grep -E "^\[" my.cnf
  # sed -n '/\[.*\]/p' my.cnf
  # sed -n '/\[.*\]/p' my.cnf | sed -e 's/\[//g' -e 's/\]//g'
  # sed -n '/\[mysqld\]/,/\[.*\]/p' my.cnf | grep -v ^# | grep -v ^$ | grep -v "\[.*\]"
  
  # mysql_process.sh
  #!/bin/bash
  
  FILE_NAME=/dev_env/shell_test/my.cnf
  # 修改默认空格分隔符 为 换行分隔符
  IFS=$'\n'
  
  function get_all_segments
  {
          echo "`sed -n '/\[.*\]/p' $FILE_NAME | sed -e 's/\[//g' -e 's/\]//g'`"
  }
  
  for seg in `get_all_segments`
  do
          echo "配置项: $seg"
  done
  
  function count_items_in_segment
  {
          items=`sed -n '/\['$1'\]/,/\[.*\]/p' $FILE_NAME | grep -v "^#" | grep -v "^$" | grep -v "\[.*\]"`
  
          index=0
          for item in $items
          do
                  index=`expr $index + 1`
          done
          echo $index
  }
  
  sum=`count_items_in_segment mysqld`
  echo $sum
  
  number=0
  
  for segment in `get_all_segments`
  do
          number=`expr $number + 1`
          items_count=`count_items_in_segment $segment`
          echo "$number: $segment $items_count"
  done
  ```

* 利用sed删除特定内容

  ```shell
  sed -i ’15d‘ passwd				# 删除第15行
  sed -i '8,14d' passwd			# 删除第8行到第14行的所有内容
  sed -i '/\/sbin\/nologin/d' passwd				# 删除不能登录的用户
  sed -i '/^mail/,/^yarn/d' passwd					# 删除以mail开头的行到以yarn开头的行
  sed -i '5,/^ftp/d' passwd									# 删除第5行到以ftp开头的所有行的内容
  sed -i '/^yarn/,$' passwd								# 删除以yarn开头的行到最后行的所有内容
  ```

  ```shell
  # 删除配置文件中的所有注释行和空行
  		# [:blank:]*  多个空格
  		# 删除 #开头和空行
  			sed -i '/^#/d;/^$/d' my.cnf
  		# 删除 空格开头，后面有#的行
  			sed -i '/[:blank:]*#/d' my.cnf
  		# 最终
  			sed -i '/^#/d;/^$/d;/[:blank:]*#/d' my.cnf
  # 在配置文件中所有不以#开头的行前面添加*符号，注意:以#开头的行不添加
  		# 不以#开头，[^#]
  		sed -i 's/^[^#]/\*&/g' my.cnf
  ```

* 利用sed修改文件内容

  ```shell
  # 删除数字
  sed -i 's/[0-9]*//g' file.txt
  # 修改/etc/passwd中第1行中第1个root为ROOT
  sed -i '1s/root/ROOT' passwd
  # 5-10行中所有的/sbin/nologin更改为/bin/bash
  sed -i '5,10s/\/sbin\/nologin/\/bin\/bash/g' passwd
  # 将匹配到的/sbin/nologin的行中的login改为大写LOGIN
  sed -i '/\/sbin\/nologin/s/login/LOGIN/g' passwd
  # 将匹配以root开头的行，到匹配行中包含mail的所有行，修改行中bin为HADOOP
  sed -i '/^root/,/mail/s/bin/HADOOP/g' passwd
  # 匹配以root开头的行到15行的所有行，将其中nologin改为SPARK
  sed -i '/^root/,15s/nologin/SPARK/g' passwd
  # 从15行开始到匹配以yarn开头的所有行，将其中bin改为BIN
  sed -i '15,/^yarn/s/bin/BIN/g' passwd
  ```

* 利用sed追加内容

  ```shell
  a					# 在匹配行后面追加
  i					# 在匹配行前面追加
  r					# 将文件内容追加到匹配行后面
  w					# 将匹配行写入指定文件
  ```

  ```shell
  # 第10行后面追加一行
  sed -i '10a abcd' passwd
  # 第10行到20行，每一行后面都追加一行
  sed -i '10,20a abcd' passwd
  # 匹配到/bin/bash的行后面追加一行
  sed -i '/\/bin\/bash/a abcd' passwd
  # 匹配以yarn开头的行，在匹配行前面追加一行
  sed -i '/^yarn/i abcd' passwd
  # 每一行前面都追加
  sed -i 'i abcd' passwd
  # 将/etc/fstab文件的内容追加到passwd文件的第20行后面
  sed -i '20r /etc/fstab' passwd
  # 将/etc/inittab文件内容追加到passwd文件匹配/sbin/nologin行的后面
  sed -i '/\/bin\/nologin/r /etc/inittab' passwd
  # 将/etc/vconsole.conf文件内容追加到passwd文件中特定行后面，匹配以ftp开头的行到第18行
  sed -i '/^ftp/,18r /etc/vconsole.conf' passwd
  # 将passwd文件匹配到/bin/bash的行追加到/tmp/sed.txt文件中
  sed -i '/\/bin\/bash/w /tmp/sed.txt' passwd
  # 将passwd文件从第10行开始，到匹配到hdfs开头的所有行内容追加到/tmp/sed-1.txt
  sed -i '10,/^hadfs/w /tmp/sed-1.txt' passwd
  ```

## 6. AWK报告生成器详解

```shell
# awk是一个文本处理工具，通常用于处理数据并生成结果报告
# 第一种形式
awk 'BEGIN{}pattern{commands}END{}' file_name
# 第二种形式
standard output | awk 'BEGIN{}pattern{commands}END{}'
# 语法格式说明
BEGIN{}					# 正式处理数据之前执行
pattern					# 匹配模式
{commands}			# 处理命令，可能多行
END{}						# 处理完所有匹配数据后执行
```

* awk内置变量

```shell
$0				# 整行内容
$1-$n			# 当前行的第1-n个字段
NF				# 当前行的字段个数，也就是有多少列，Number Filed
NR				# 当前行的行号，从1开始计数，Number Row
FNR				# 多文件处理时，每个文件行号单独计数，都是从0开始，File Number Row
FS				# 输入字段分隔符。不指定默认以空格或tab键分割，File Separator
RS				# 输入行分隔符。默认回车换行，Row Separator
OFS				# 输出字段分隔符。默认为空格，Output Filed Separator
ORS				# 输出行分隔符。默认回车换行，Output Row Separator
FILENAME	# 当前输入的文件名字
ARGC			# 命令行参数个数
ARGV			# 命令行参数数组
```

```shell
# 输出文本所有行内容
awk '{print $0}' /etc/passwd
# 每行以:分割，并输出每行分割后的第1行
awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
# 默认空格、Tab分割符，输出每行第一个单词
awk '{print $1}' list
# 输出每行字段个数
awk '{print NF}' list
# 输出文件中每行的行号，passwd行号接着list的
awk '{print NR}' list /etc/passwd
# 输出每个文件中的行号，单独计数
awk '{print FNR}' list /etc/passwd
# 以--为行分隔符，输出分隔后的每行
awk 'BEGIN{RS="--"}{print $0}' list
# 先以--为行分隔符，然后以|为列分隔符，输出每行第3列
awk 'BEGIN{RS="--";FS="|"}{print $3}' list
# 以&为输出行分隔符，即结果都排在一行以&分割，而不是默认的回车换行符（各自占一行）
awk 'BEGIN{RS="--";FS="|";ORS="&"}{print $3}' list
# 输出的每行中以:分割，然后每行再以&作为分隔符，结果还是在一行上
awk 'BEGIN{RS="--";FS="|";ORS="&";OFS=":"}{print $1,$3}'
# 输出文件名，有多少行，就输出多少次
awk '{print FILENAME}' list
# 输出命令行参数个数
awk '{print ARGC}' list        # 2 2 2
awk '{print ARGC}' list /etc/fstab # 3 3 3 3
```

* awk格式化输出之printf

```shell
%s					# 打印字符串
%d					# 打印十进制数
%f					# 打印一个浮点数
%x					# 打印十六进制数
%o					# 打印八进制数
%e					# 打印数字的科学计数法形式
%c					# 打印单个字符的ASCII码
-						# 左对齐
+						# 右对齐
#						# 显示8进制在前面加0，显示16进制在前面加0x
```

```shell
# 每行中以:分割，打印每行第1个
awk 'BEGIN{FS=":"}{printf "%s\n",$1}' /etc/passwd
# 每行中每个字段占20个字符，左对齐
awk 'BEGIN{FS=":"}{printf "%-20s %-20s\n",$1,$7}' /etc/passwd
# 以字符串格式打印/etc/passwd中的第7个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%s\n",$7}' /etc/passwd
awk 'BEGIN{FS=":"} {printf "%s\n",$NF}' /etc/passwd
# 以10进制格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%d\n",$3}' /etc/passwd
# 以浮点数格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%0.3f\n",$3}' /etc/passwd
# 以16进制数格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%#x\n",$3}' /etc/passwd
# 以8进制格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%#o\n",$3}' /etc/passwd
# 以科学计数法格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%#e\n",$3}' /etc/passwd
```

* 模式匹配的两种用法

```shell
# 第一种
RegExp					# 按正则表达式匹配
# 第二种
关系运算匹配			# 按关系运算匹配
		==、!=、>、>=、<、<=、!=
		~匹配正则表达式
		!~不匹配正则表达式
布尔运算符匹配
		||					# 或
		&&					# 与
		!						# 非
```

```shell
# 匹配/etc/passwd文件行中含有root字符串的所有行
awk 'BEGIN{FS=":"}/root/{print $0}' /etc/passwd
# 匹配/etc/passwd文件中以yarn开头的所有行
awk 'BEGIN{FS=":"}/^yarn/{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第3个字段小于50的所有行信息
awk 'BEGIN{FS=":"}$3<50{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第3个字段大于50的所有行信息
awk 'BEGIN{FS=":"}$3>50{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第7个字段为/bin/bash的所有行信息
awk 'BEGIN{FS=":"}$7=="/bin/bash"{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第7个字段不为/bin/bash的所有行信息
awk 'BEGIN{FS=":"}$7!="/bin/bash"{print $0}' /etc/passwd

# 匹配包含root的字段的行
awk 'BEGIN{FS=":"}$1~/root/{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第3个字段包含3个以上数字的所有行信息
awk 'BEGIN{FS=":"}$3~/[0-9]{3,}/{print $0}' /etc/passwd
# 匹配/sbin/nologin的行
awk 'BEGIN{FS=":"}$0~/\/sbin\/nologin/{print $0}' /etc/passwd
# 不匹配/sbin/nologin的行
awk 'BEGIN{FS=":"}$0!~/\/sbin\/nologin/{print $0}' /etc/passwd

# 以:为分隔符，匹配/etc/passwd文件中包含hdfs或yarn的所有行信息
awk 'BEGIN{FS=":"}$1=="hdfs" || $1=="yarn" {print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第3个字段小于50并且第4个字段大于50的所有行信息
awk 'BEGIN{FS=":"}$3<50 && $4>50 {print $0}' /etc/passwd
```

* awk动作中的表达式用法

```shell
# awk动作表达式中的算术运算符
+				# 加
-				# 减
*				# 乘
/				# 除
%				# 模
^或**	 # 乘方
++x			# 在返回x变量之前，x变量加1
x++			# 在返回x变量之后，x变量加1
```

```shell
awk 'BEGIN{var=20;var1="hello";print var var1}'
awk 'BEGIN{num1=20;num2+=num1;print num1 num2}'
awk 'BEGIN{num1=20;num2=30;print num1-num2}'
awk 'BEGIN{num1=20;num2=30;print num1+num2}'
awk 'BEGIN{num1=20;num2=30;print num1*num2}'
awk 'BEGIN{num1=20;num2=30;print num1/num2}'
awk 'BEGIN{num1=20;num2=30;printf "%f\n",num1/num2}'
awk 'BEGIN{num1=20;num2=30;printf "%0.2f\n",num1/num2}'
awk 'BEGIN{num1=20;num2=3;printf "%0.2f\n",num1**num2}'
```

```shell
# 计算/etc/service中的空白行数量
awk '/^$/{sum++}END{print sum}' /etc/services
# 计算学生课程分数平均值
	Allen		80	90	96	98
	Mike		93	98	92	91
	Zhang		78	76	87	92
	Jerry		86	89	68	92
	Han			85	95	75	90
	Li			78	88	98	100
awk '{print $1}' student.txt	
awk '{total=$2+$3+$4+$5;AVG=total/4;printf "%-8s%-5d%-5d%-5d%-5d%-0.2f\n",$1,$2,$3,$4,$5,AVG}' student.txt
awk 'BEGIN{printf "%-8s%-8s%-8s%-8s%-8s%-8s\n","名称","语文","数学","英语","物理","平均分"}{total=$2+$3+$4+$5;AVG=total/4;printf "%-10s%-10d%-10d%-10d%-10d%-0.2f\n",$1,$2,$3,$4,$5,AVG}' student.txt
名称      语文      数学      英语      物理      平均分
Allen     80        90        96        98        91.00
Mike      93        98        92        91        93.50
Zhang     78        76        87        92        83.25
Jerry     86        89        68        92        83.75
Han       85        95        75        90        86.25
Li        78        88        98        100       91.00
# 输出/etc/passwd文件的行数.正序显示行数和倒序显示行数
```

* awk动作中的条件及循环语句

```shell
# 以:为分隔符，只打印/etc/passwd中第3个字段的数值在50-100范围内的行信息
awk 'BEGIN{FS=":"}{if($3<50 || $3>100) print $0}' /etc/passwd
awk 'BEGIN{FS=":"}{if($3<50) printf "%-10s%-10s%-5d\n","小于50的UID",$1,$3}' /etc/passwd
# script.awk
BEGIN{
        FS=":"
}
{
        if($3<50)
        {
                printf "%-20s%-20s%-5d\n","UID<50",$1,$3
        }
        else if($3>50 && $3<100)
        {
                printf "%-20s%-20s%-5d\n","50<UID<100",$1,$3
        }
        else
        {
                printf "%-20s%-20s%-5d\n","100<UID",$1,$3
        }
}

awk -f script.awk /etc/passwd

# 计算下列每个同学的平均分数，并且只打印平均分数大于90的同学姓名和分数信息
	名称		语文 英语 数学 物理
	Allen		80	90	96	98
	Mike		93	98	92	91
	Zhang		78	76	87	92
	Jerry		86	89	68	92
	Han			85	95	75	90
	Li			78	88	98	100
	
# student_score.awk	
BEGIN{
        printf "%-8s%-8s%-8s%-8s%-8s%-8s\n","名称","语文","数学","英语","物理","平均分"
}
{
        total=$2+$3+$4+$5
        avg=total/4
        if(avg>90)
        {
                printf "%-10s%-10d%-10d%-10d%-10d%-0.2f\n",$1,$2,$3,$4,$5,avg
                score_chinese+=$2
                score_english+=$3
                score_math+=$4
                score_physical+=$5
        }
}
END{
        printf "%-10s%-10d%-10d%-10d%-10d\n","",score_chinese,score_english,score_math,score_physical
}

awk -f student_score.awk student.txt
名称      语文      数学      英语      物理      平均分
Allen     80        90        96        98        91.00
Mike      93        98        92        91        93.50
Li        78        88        98        100       91.00
          251       276       286       289
	
# 计算1+2+3+...+100的和，请使用while、do while、for三种循环方式实现	
# while.awk
BEGIN{
	while(i<=100)
	{
		sum+=i
		i++
	}
	print sum
}
# for.awk
BEGIN{
	for(i=0;i<=100;i++)
	{
		sum+=i
	}
	print sum
}
# dowhile.awk
BEGIN{
	do
	{
		sum+=i
		i++
	}while(i<=100)
	print sum
}
```

* 字符串函数对照表

```shell
# 计算字符串长度，返回整数长度值
length(str)
# 在str1中查找str2的位置，返回值为位置索引，从1计数
index(str1,str2)
# 转换为小写，返回转换后的小写字符串
tolower(str)
# 转换为大写，返回转换后的大写字符串
toupper(str)
# 从str的m个字符开始，截取n位，返回截取后的子串
substr(str,m,n)
# 按fs切割字符串，结果保存arr，返回切割后的子串的个数
split(str,arr,fs)
# 在str中按照RE查找，返回位置，返回索引位置 
match(str,RE)
# 在str中搜索符合RE的字串，将其替换为RepStr；只替换第一个，返回替换的个数
sub(RE,RepStr,str)
# 在str中搜索符合RE的字串，将其替换为RepStr;替换所有，返回替换的个数
gsub(RE,RepStr,str)
```

```shell
# 1. 以:为分隔符，返回/etc/passwd中每行中每个字段的长度
# root:x:0:0:root:/root:/bin/bash
# 4:1:1:1:4:5:9
# awk -f length_test.awk /etc/passwd
# length_test.awk
BEGIN{
        FS=":"
}
{
        i=1
        while(i<=NF)
        {
                if(i==NF)
                        printf "%d",length($i)
                else
                        printf "%d:",length($i)
                i++
        }
        print ""
}
```

```shell
# 2. 搜索字符串"I have a dream"中出现"ea"子串的位置
awk 'BEGIN{str="I have a dream";location=index(str,"ea");print location}'
awk 'BEGIN{str="I have a dream";location=match(str,"ea");print location}'
# 3. 将字符串"Hadoop is a bigdata Framework"全部转换为小写
awk 'BEGIN{str="Hadoop is a bigdata Framework";print tolower(str)}'
# 4. 将字符串"Hadoop is a bigdata Framework"全部转换为大写
awk 'BEGIN{str="Hadoop is a bigdata Framework";print toupper(str)}'
# 5. 将字符串"Hadoop Kafka Spark Storm HDFS YARN Zookeeper" 按照空格为分隔符，分隔每部分保存到数组中
awk 'BEGIN{str="Hadoop Kafka Spark Storm HDFS YARN Zookeeper";split(str,arr," ");print arr[1]}'
awk 'BEGIN{str="Hadoop Kafka Spark Storm HDFS YARN Zookeeper";split(str,arr," ");for(a in arr) print arr[a]}'
# 6. 搜索字符串"Tranction 2345 Start:Select * from master" 第一个数字出现的位置
awk 'BEGIN{str="Tranction 2345 Start:Select * from master";location=match(str,/[0-9]/);print location}'
awk 'BEGIN{str="Tranction 2345 Start:Select * from master";location=match(str,"Select");print location}'
# 7. 截取字符串"transaction start"的子串，截取条件从第4个字符开始，截取5位
awk 'BEGIN{str="transaction start";print substr(str,4,5)}'
# 8. 替换字符串"Transaction 243 Start,Event ID:9002"中第一个匹配到的数字串为$符号
awk 'BEGIN{str="Transaction 243 Start,Event ID:9002";count=sub(/[0-9]+/,"$",str);print count,str}'
awk 'BEGIN{str="Transaction 243 Start,Event ID:9002";count=gsub(/[0-9]+/,"$",str);print count,str}'
```

```shell
# 参数传递，定义或引用变量，引用变量加上双引号""
-v
num1=20
var="hello world"
awk -v num2="$num1" -v var1="$var" 'BEGIN{print num2,var1}'
# 指定脚本文件
-f
awk -v num2="$num1" -v var1="$var" -f test.awk
# test.awk 
BEGIN{print num2,var1}
# 指定分隔符
-F
awk -F: '{print $7}' /etc/passwd
# 查看awk的版本号
-V
awk -V
```

```shell
awk -f 1.awk
# 1.awk
BEGIN{
	str="I have a dream"
	location=index(str,"ea")
	print location
}
```

* Shell中数组的用法

  ```shell
  # 下标从0开始
  array=("Allen" "Mike" "Messi" "Jerry" "Hanmeimei" "Wang")
  # 打印所有元素
  echo ${array[*]}
  # 打印元素
  echo ${array[2]}
  # 打印元素个数
  echo ${#array[@]}
  echo ${#array[*]}
  # 打印元素长度
  echo ${#array[3]}
  str="test str"
  echo ${#str}                # 打印字符串长度
  # 给元素赋值
  array[3]="Li"
  # 删除元素，删除的元素还是占用下标位置
  unset array[2]
  unset array                 # 删除数组
  # 分片访问
  echo ${array[@]:1:3}
  # 元素内容替换
  ${array[@]/e/E}   # 只替换第一个e;${array[@]//e/E} 替换所有的e
  # 数组的遍历    # for a in ${array[@]};do echo $a;done
  for a in ${array[@]}
  do
  	echo $a
  done	
  ```

* awk中数组的用法

  ```shell
  # 下标从1开始
  # 在awk中，使用数组时，不仅可以使用1.2..n作为数组下标，也可以使用字符串作为数组下标
  ```

  ```shell
  # 典型常用例子
  1. 统计主机上所有的TCP连接状态数，按照每个TCP状态分类
  netstat -an | grep tcp | awk '{array[$6]++}END{for(a in array) print a,array[a]}'
  2. 计算横向数据总和，计算纵向数组总和
  allen		80	90	87	91
  mike		78	86	93	96
  Kobe		66 	92	82	78
  Jerry		98	74	66	54
  Wang		87 	21	100	43
  # awk -f student.awk student.txt
  BEGIN{
  	printf "%-10s%-10s%-10s%-10s%-10s%-10s\n","Name","Yuwen","Math","English","Physical","Total"
  }
  {
  	total=$2+$3+$4+$5
  	yuwen_sum+=$2
  	math_sum+=$3
  	eng_sum+=$4
  	phy_sum+=$5
  	printf "%-10s%-10d%-10d%-10d%-10d%-10d\n",$1,$2,$3,$4,$5,total
  }
  END{
  	printf "%-10s%-10d%-10d%-10d%-10d\n","",yuwen_sum,math_sum,eng_sum,phy_sum
  }
  ```

  ```shell
  # 需求描述：利用awk处理日志，并生成结果报告
  # 生成数据脚本insert.sh内容如下，sh insert.sh        # 执行一会 Ctrl + c 停止
  #!/bin/bash
  # 
  function create_random()
  {
  	min=$1
  	max=$(($2-$min+1))
  	num=$(date +%s%N)
  	echo $(($num%$max+$min))
  }
  INDEX=1
  while true
  do
  	for user in Allen Mike Jerry Tracy Hanmeimei Lilei
  	do
  		COUNT=$RANDOM
  		NUM1=`create_random 1 $COUNT`
  		NUM2=`expr $COUNT - $NUM1`
  		echo "`date '+%Y-%m-%d %H:%M:%S'` $INDEX Batches: user $user INSERT $COUNT DATA INTO database.table 'test',Insert $NUM1 Records Successfully,Failed $NUM2 records" >> ./db.log.`date +%Y%m%d`
  		INDEX=`expr $INDEX + 1`
  	done
  done  
  
  # 生成数据格式
  2020-03-13 16:31:40 21625 Batches: user Allen INSERT 24216 DATA INTO database.table 'test',Insert 6964 Records Successfully,Failed 17252 records
  2020-03-13 16:31:40 21626 Batches: user Mike INSERT 29824 DATA INTO database.table 'test',Insert 22717 Records Successfully,Failed 7107 records
  
  # (1) 统计每个人员分别插入了多少条record进数据库
  输出结果:
  	USER		Total_Records
  	allen		493082
  	mike		349287
  	
  # awk -f exam_1.awk db.log.20200313	
  BEGIN{
          printf "%-10s%-20s\n","User","Total Records"
  }
  
  {
          USER[$6]+=$8
  }
  
  END{
          for(u in USER)
                  printf "%-10s%-20d\n",u,USER[u]
  }
  	
  	
  # (2) 统计每个人分别插入成功了多少record，失败了多少record
  User      Success_Records     Failed_Records
  Jerry     575841              615620
  Mike      507381              601506
  # exam_2.awk    # awk -f exam_2.awk db.log.20200313
  BEGIN{
      printf "%-10s%-20s%-20s\n","User","Success_Records","Failed_Records"
  }
  {
      SUCCESS[$6]+=$13
      FAIL[$6]+=$16
  }
  END{
      for(u in SUCCESS)
          printf "%-10s%-20d%-20d\n",u,SUCCESS[u],FAIL[u]
  }
  
  # (3) 将例子1和例子2结合起来，一起输出，输出每个人分别插入多少数据，多少成功，多少失败，并且要格式化输出，加上标题
  User      Total               Success             Failed
  Jerry     1191461             575841              615620
  Mike      1108887             507381              601506
  
  # exam_3.awk    # awk -f exam_3.awk db.log.20200313
  BEGIN{
      printf "%-10s%-20s%-20s%-20s\n","User","Total","Success","Failed"
  }
  {
      TOTAL[$6]+=$8
      SUCCESS[$6]+=$13
      FAIL[$6]+=$16
  }
  END{
      for(u in SUCCESS)
          printf "%-10s%-20d%-20d%-20d\n",u,TOTAL[u],SUCCESS[u],FAIL[u]
  }
  
  # (4) 在例子3的基础上，加上结尾，统计全部插入记录数，成功记录数，失败记录数
  User      Total               Success             Failed
  Jerry     1191461             575841              615620
  Mike      1108887             507381              601506
  Lilei     1014911             518595              496316
  Hanmeimei 1042570             462928              579642
  Tracy     1073380             584628              488752
  Allen     1114892             550323              564569
            6546101             3199696             3346405
  
  # awk -f exam_4.awk db.log.20200313
  BEGIN{
      printf "%-10s%-20s%-20s%-20s\n","User","Total","Success","Failed"
  }
  {
      TOTAL[$6]+=$8
      SUCCESS[$6]+=$13
      FAIL[$6]+=$16
  }
  END{
      for(u in SUCCESS)
     {
          total+=TOTAL[u]
          success+=SUCCESS[u]
          fail+=FAIL[u]
          printf "%-10s%-20d%-20d%-20d\n",u,TOTAL[u],SUCCESS[u],FAIL[u]
     }
          printf "%-10s%-20d%-20d%-20d\n","",total,success,fail
  }
  
  # (5) 查找丢失数据的现象，也就是成功+失败的记录数，不等于一共插入的记录数。找出这些数据并显示行号和对应行的日志信息
  # exam_5.awk
  {
  	if($8!=$14+$17)
  	{
  		print NR,$0
  	}
  }
  ```

# Mysql脚本

```sql
--school.sql

CREATE TABLE `student`(
	`s_id` VARCHAR(20),
  `s_name` VARCHAR(20) NOT NULL DEFAULT '',
  `s_birth` VARCHAR(20) NOT NULL DEFAULT '',
  `s_sex` VARCHAR(10) NOT NULL DEFAULT '',
  PRIMARY KEY(`s_id`)
);
CREATE TABLE `course`(
	`c_id` VARCHAR(20),
  `c_name` VARCHAR(20) NOT NULL DEFAULT '',
  `t_id` VARCHAR(20) NOT NULL,
  PRIMARY KEY(`c_id`)
);
CREATE TABLE `teacher`(
	`t_id` VARCHAR(20),
  `t_name` VARCHAR(20) NOT NULL DEFAULT '',
  PRIMARY KEY(`t_id`)
);
CREATE TABLE `score`(
	`s_id` VARCHAR(20),
  `c_id` VARCHAR(20),
  `s_score` INT(3),
  PRIMARY KEY(`s_id`,`c_id`)
);
-- 插入学生表测试数据
insert into student values('1001','zhaolei','1990-1001-1001','male');
insert into student values('1002','lihang','1990-12-21','male');
insert into student values('1003','yanwen','1990-1005-20','male');
insert into student values('1004','hongfei','1990-1008-1006','male');
insert into student values('1005','ligang','1991-12-1001','female');
insert into student values('1006','zhousheng','1992-1003-1001','female');
insert into student values('1007','wangjun','1989-1007-1001','female');
insert into student values('1008','zhoufei','1990-1001-20','female');
-- 课程表测试数据
insert into course values('1001','chinese','1002');
insert into course values('1002','math','1001');
insert into course values('1003','english','1003');
-- 教师表测试数据
insert into teacher values('1001','aidisheng');
insert into teacher values('1002','aiyinsitan');
insert into teacher values('1003','qiansanqiang');
-- 成绩表测试数据
insert into score values('1001','1001',80);
insert into score values('1001','1002',90);
insert into score values('1001','1003',99);
insert into score values('1002','1001',70);
insert into score values('1002','1002',60);
insert into score values('1002','1003',80);
insert into score values('1003','1001',80);
insert into score values('1003','1002',80);
insert into score values('1003','1003',80);
insert into score values('1004','1001',50);
insert into score values('1004','1002',30);
insert into score values('1004','1003',20);
insert into score values('1005','1001',76);
insert into score values('1005','1002',87);
insert into score values('1006','1001',31);
insert into score values('1006','1003',34);
insert into score values('1007','1002',89);
insert into score values('1007','1003',98);
```

```sql
create database school default character set utf8;
show databases;
use school;
```

```shell
# 将sql文件中数据插入数据库表中
mysql -uroot -proot school < school.sql
# 创建用户
grant all on school.* to dbuser@'%' identified by '123456';
grant all on school.* to dbuser@'localhost' identified by '123456';
use mysql;
show tables;
desc user;
select Host,User,Password from user;
# 创建和student一样表结构的student1表
create table student1 like student;
```

```shell
# Mysql命令参数详解
-u			# 用户名
-p			# 用户密码
-h			# 服务器IP地址
-D			# 连接的数据库
-N			# 不输出列信息
-B			# 使用tab键代替默认交互分隔符
-e			# 执行SQL语句
# 其他选项
-E			# 垂直输出
-H			# 以HTML格式输出
-X			# 以XML格式输出

mysql -udbuser -p123456

mysql -udbuser -p123456 -D school -N -B -e "SELECT * FROM student;"
mysql -udbuser -p123456 -D school -N -E -B -e "SELECT * FROM student;"
mysql -udbuser -p123456 -D school -N -H -B -e "SELECT * FROM student;" > result.html
mysql -udbuser -p123456 -D school -N -X -B -e "SELECT * FROM student;" > result.xml

# 1. 写一个脚本，该脚本可以接收两个参数，参数为需要执行的SQL语句
sh operate_mysql.sh school "SELECT * FROM student"
sh operate_mysql.sh school "INSERT INTO score values("1021","1002",100)"
sh operate_mysql.sh school "SELECT * FROM score" > result.txt
# operate_mysql.sh
#!/bin/bash
#
user="dbuser"
password="123456"
# host="192.168.0.221"

db_name="$1"
SQL="$2"

mysql -u"$user" -p"$password" -D"$1" -B -e "$SQL"
# 2. 查询MySQL任意表的数据，并将查询到的结果保存到HTML文件中
# 3. 查询MySQL任意表的数据，并将查询到的结果保存到XML文件中
```

```shell
# 如何将文本中格式化的数据导入到MySQL数据库中？
# 需求：处理文本中的数据，将文本中的数据插入MySQL中
# data.txt
1010    jerry   1991-12-13      male
1011    mike    1991-12-13      female
1012    tracy   1991-12-13      male
1013    kobe    1991-12-13      male
1014    allen   1991-12-13      female
1015    curry   1991-12-13      male
1016    tom     1991-12-13      female

# data_test.sh
#!/bin/bash
#
user="dbuser"
password="123456"
# host="192.168.0.221"

mysql_conn="mysql -u"$user" -p"$password""

cat data.txt | while read id name birth sex
do
	# $id，使用单引号，双引号报错
	if [ $id -gt 1014 ];then
		$mysql_conn -e "INSERT INTO school.student1 values('$id','$name','$birth','$sex')"
	fi	
done	

# 执行命令
create table student1 like student;
sh data_test.sh
```

```shell
# data_2.txt
2021|hao|1989-12-21|male
2022|zhang|1989-12-21|male
2023|ouyang|1989-12-21|male
2024|li|1989-12-21|female

# data_test2.sh
#!/bin/bash
#
user="dbuser"
password="123456"
# host="192.168.0.221"

# 和｜冲突
# mysql_conn="mysql -u"$user" -p"$password""

# 改变默认分隔符为|
IFS="|"

cat data_2.txt | while read id name birth sex
do
        echo
        echo $id
        echo
        echo $name
        echo
        echo $birth
        echo
        echo $sex
    # $id，使用单引号，双引号报错
    if [ $id -gt 1014 ];then
        # 冲突
        # $mysql_conn -e "INSERT INTO school.student2 values('$id','$name','$birth','$sex')"

        mysql -udbuser -p123456 -e "INSERT INTO school.student2 values('$id','$name','$birth','$birth')"
    fi
done	

# 执行命令
sh operate_mysql.sh school 'create table student2 like student'
```

```shell
# 备份MySQL中的库或表
mysqldump
-u			# 用户名
-p			# 密码
-h			# 服务器IP地址
-d			# 等价于--no-data		只导出表结构
-t			# 等价于--no-create-info		只导出数据，不导出建表语句
-A			# 等价于--all-databases	
-B			# 等价于--databases 导出一个或多个数据库

# 需求：将school中的score表备份，并且将备份数据通过FTP传输到192.168.184.3的/data/backup目录下
mysqldump -udbuser -p123456 school score > score.sql
```

![image-20200314211807257](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200314211807257.png)

# 大型脚本案例

* 需求描述

  1. 实现一个脚本工具，该脚本提供类似supervisor功能，可以对进程进行管理
  2. 一键查看所有进程运行状态
  3. 单个或批量启动进程，单个或批量停止进程
  4. 提供进程分组功能，可以按组查看进程运行状态，可以按组启动或停止该组内所有进程

  ```shell
  # 拆分脚本功能，抽象函数
  1. function get_all_group						# 返回进程组列表字符串
  2. function get_all_process					# 返回进程名称列表字符串
  3. function get_process_info				# 返回进程详细信息列表字符串，详细信息包括:运行状态、PID、CPU、MEM、启动时间
  	注：该函数可以接收一个参数，参数为进程名称
  	function get_process_pid_by_name process_name
  	function get_process_info_by_pid process_pid
  4. function get_all_process_by_group # 返回进程组内的所有进程名称列表字符串
  	例子：DB组--> "mysql postgresql oracle"
  ```

* `process.cfg`

  ```shell
  [GROUP_LIST]
  WEB
  DB
  HADOOP
  YARN
  
  [WEB]
  nginx
  httpd
  
  [DB]
  mysql
  postgresql
  oracle
  
  [HADOOP]
  datanode
  namenode
  journalnode
  
  [YARN]
  resourcemanager
  nodemanager
  ```

* 功能函数

  ```shell
  function get_all_group
  	说明：该函数无需输入任何参数；返回配置文件process.cfg中所有的组信息，例如WEB、DB等
  function get_all_process
	说明：该函数无需输入任何参数；返回配置文件process.cfg中所有的进程信息
  function get_process_pid_by_name
  	说明：该函数接收一个参数，参数为进程名称；返回值是一个PID的列表，可能是一个PID，也可能有多个
  function get_process_info_by_pid
  	说明：该函数接收一个参数，参数为进程PID；返回值是一个进程运行信息的列表，列表包含运行状态、CPU占有率、内存占有率、进程启动时间
  function is_group_in_config
  	说明：该函数接收一个参数，参数为组的名称；返回值是0或1，0代表该组在配置文件中，1代表该组不在配置文件中
  function get_all_process_by_group
  	说明：该函数接收一个参数，参数为组名称；返回值是对应组内的所有进程名称列表
  function get_group_by_process_name
  	说明：该函数接收一个参数，参数是一个进程名称；返回值是一个组名
  function format_print
  	说明：该函数接收两个参数，第一个参数为process_name，第二个参数为组名称
  	返回值，是针对每一个进程PID的运行信息
  function is_process_in_config
  	说明：该函数接收一个参数，参数为进程名称；返回值是0或1，0代表该进程在配置文件中，1代表进程不在配置文件中
  ```

* ==程序主流程设计==

  ```shell
  ./app_status.sh执行有三种情况：
  1. 无参数					列出配置文件中所有进程的运行信息
  2. -g GroupName 	列出GroupName组内的所有进程
  3. process_name1	列出指定进程的运行信息
  ```

  ```shell
  # 获取组名
  # 过滤空行 grep -v "^$"
  sed -n '/\[GROUP_LIST\]/,/\[.*\]/p' process.cfg | grep -v "^$" | grep -v "\[.*\]"
  sed -n '/\[GROUP_LIST\]/,/\[.*\]/p' process.cfg | egrep -v "(^$|\[.*\])"
  # 获取指定组下的成员
  sed -n '/\[WEB\]/,/\[.*\]/p' process.cfg | egrep -v "(^$|\[.*\])"
  # 脚本中要用双引号
  P_LIST=`sed -n '/\[$g\]/,/\[.*\]/p' process.cfg | egrep -v "(^$|\[.*\])"`
  改为
  P_LIST=`sed -n "/\[$g\]/,/\[.*\]/p" process.cfg | egrep -v "(^$|\[.*\])"`
  # 获取进程pid
  ps -ef|grep mysql | grep -v grep | awk '{print $2}'
  # awk不能直接引用外部变量，需要先赋值给awk变量
  ps -ef | awk -v pid=1899 '$2==pid{print}' | wc -l     # 精确找到第二列为1899的一行数据
  # 第3列cpu占有率、第4列内存占有率
  ps aux | awk -v pid=1899 '$2==pid{print $3}'
  ps aux | awk -v pid=1899 '$2==pid{print $4}'
  # 获取启动时间
  ps -p 1899 -o lstart
  # 运行脚本时打印调试信息
  sh -x xx.sh
  # 查找function的内容
  grep ^function app_status.sh
  
  sh app_status.sh -g WEB DB
  sh app_status.sh mysql
  
  systemctl start httpd
  ps -ef|grep httpd
  netstat -tpnl | grep :80
  ```

* 完整脚本

  ```shell
  # app_status.sh
  #!/bin/bash
  #
  # Func: Get Process Status In process.cfg
  
  # Define Variables
  HOME_DIR="/dev_env/shell_test/app_test"
  CONFIG_FILE="process.cfg"
  # 执行脚本时的pid
  this_pid=$$
  
  function get_all_group
  {
  	# if [ ! -e $HOME_DIR/$CONFIG_FILE ];then
  	# 	echo "$CONFIG_FILE is not exist..Please Check.."
  	#	exit 1
  	# else
  		G_LIST=`sed -n '/\[GROUP_LIST\]/,/\[.*\]/p' process.cfg | egrep -v "(^$|\[.*\])"`
  		echo "$G_LIST"	
  	# fi
  		
  }
  
  # for g in `get_all_group`;do
  #	echo $g
  # done
  
  function get_all_process
  {
  	for g in `get_all_group`
  	do
  		P_LIST=`sed -n "/\[$g\]/,/\[.*\]/p" process.cfg | egrep -v "(^$|\[.*\])"`
  		echo "$P_LIST"
  	done
  }
  
  # processes=`get_all_process`
  # echo $processes
  
  function get_process_pid_by_name
  {
  	# $#参数个数，函数参数个数
  	if [ $# -ne 1 ];then
  		return 1
  	else
  		# 过滤掉执行脚本时的pid  grep -v $this_pid
  		# 过滤掉脚本本身 $0
  		pids=`ps -ef | grep $1 | grep -v grep | grep -v $0 | awk '{print $2}'`
  		echo $pids
  	fi
  } 
  
  # get_process_pid_by_name mysql
  # 执行脚本时传递进来
  # get_process_pid_by_name $1
  
  function get_process_info_by_pid
  {
  	if [ `ps -ef | awk -v pid=$1 '$2==pid{print}' | wc -l` -eq 1 ];then
  		pro_status="RUNNING"
  	else
  		pro_status="STOPED"
  	fi
  	pro_cpu=`ps aux | awk -v pid=$1 '$2==pid{print $3}'`
  	pro_mem=`ps aux | awk -v pid=$1 '$2==pid{print $4}'`
  	pro_start_time=`ps -p $1 -o lstart | grep -v STARTED`
  }
  
  # top
  # ps aux | awk -v pid=1488 '$2==pid{print $0}'
  # get_process_info_by_pid 80111
  # echo "$pro_status $pro_cpu $pro_mem $pro_start_time"
  
  function is_group_in_config
  {
  	for gn in `get_all_group`;do
  		if [ "$gn" == "$1" ];then
  			return
  		fi
  	done
  	echo "Group $1 is not in process.cfg"	
  	return 1
  }
  
  # echo `get_all_group`
  # is_group_in_config $1 && echo exist || echo "not exist"
  
  # 巨坑，比较时==两边得加空格
  function is_process_in_config
  {
  	for pn in `get_all_process`;do
  		if [ $pn == $1 ];then
  			return
  		fi
  	done
  	echo "Process $1 is not in process.cfg"
  	return 1
  }
  
  function get_all_process_by_group
  {
  	is_group_in_config $1
  	if [ $? -eq 0 ];then
  		p_list=`sed -n "/\[$1\]/,/\[.*\]/p" $HOME_DIR/$CONFIG_FILE | egrep -v "(^$|^#|\[.*\])"`
  		echo $p_list
  	else
  		echo "GroupName $1 is not in process.cfg"	
  	fi		
  }
  ## sh app_status.sh WEB
  # get_all_process_by_group $1
     
  if [ ! -e $HOME_DIR/$CONFIG_FILE ];then
  	echo "$CONFIG_FILE is not exist..Please Check.."
  	exit 1
  fi
  
  
  function get_group_by_process_name
  {
  	
  	for gn in `get_all_group`;do
  		for pn in `get_all_process_by_group $gn`;do
  			if [ $pn == $1 ];then
  				echo "$gn"
  			fi
  			echo
  		done
  	done
  }
  
  # group_name=`get_group_by_process_name $1`
  # echo $group_name
  
  # 变量最好用"",可以防止变量值中有空格，例如pro_start_time
  function format_print
  {
  	ps -ef | grep $1 | grep -v grep | grep -v $this_pid &> /dev/null
  	if [ $? -eq 0 ];then
  		pids=`get_process_pid_by_name $1`
  		for pid in $pids;do
  			get_process_info_by_pid $pid
  			awk -v p_name=$1 \
  				-v g_name=$2 \
  				-v p_status="$pro_status" \
  				-v p_id="$pid" \
  				-v p_cpu="$pro_cpu" \
  				-v p_mem="$pro_mem" \
  				-v p_start_time="$pro_start_time" \
  				'BEGIN{printf "%-20s%-12s%-10s%-6s%-7s%-10s%-20s\n",p_name,g_name,p_status,p_id,p_cpu,p_mem,p_start_time}'
  		done
  	else
  		awk -v p_name=$1 -v g_name=$2 'BEGIN{printf "%-20s%-12s%-10s%-6s%-7s%-10s%-20s\n",p_name,g_name,"STOPPED","NULL","NULL","NULL","NULL"}'
  	fi
  					
  }
  
  awk 'BEGIN{printf "%-20s%-10s%-10s%-6s%-7s%-10s%-20s\n","ProcessName---------","GroupName---","Status----","PID---","CPU----","MEMORY----","StartTime---------------"}'
  
  # 程序主流程设计
  
  # 判断输入参数个数
   if [ $# -gt 0 ];then
  	if [ "$1" == "-g" ];then
  		# 左移（去掉-g）
  		shift
  		# $@ 所有参数
  		for gn in $@;do
  			is_group_in_config $gn || continue
  			for pn in `get_all_process_by_group $gn`;do
  				is_process_in_config $pn && format_print $pn $gn
  			done
  		done
  	else
  		for pn in $@;do
  			gn=`get_group_by_process_name $pn`
  			is_process_in_config $pn && format_print $pn $gn
  		done
  	fi
  else
  	for pn in `get_all_process`;do
  		gn=`get_group_by_process_name $pn`
  		is_process_in_config $pn && format_print $pn $gn
  	done
  fi
  ```
```shell
  
  






```