```shell
本书案例源码：https://github.com/linehk/gopl
https://github.com/overnote/over-golang
https://yar999.gitbook.io/gopl-zh/ch4/ch4-01
https://www.zhihu.com/question/30461290
https://evanli.github.io/programming-book-2/Go/

https://www.imooc.com/coursescore/982
https://github.com/itsmikej/imooc_logprocess

https://www.imooc.com/coursescore/492
https://github.com/iswbm/GolangCodingTime
https://github.com/jiujuan/go-collection
https://github.com/rubyhan1314/Golang-100-Days
```

# 1. 入门

CSP（Communicating Sequential Process）通信顺序进程，程序就是一组无共享状态进程的并行组合，进程间的通信和同步采用通道完成

有垃圾回收、包系统、一等公民函数、词法作用域、系统调用接口、默认使用UTF-8编码的不可变字符串

没有隐式数值类型强制转换，没有构造函数，没有运算符重载，没有形参默认值，没有继承，没有泛型，没有异常，没有函数注解，没有线程局部存储

保证兼容更早版本

没有类继承，甚至没有类

提供变长栈来运行其轻量级线程，或称为goroutine，这个栈初始时非常小，所以创建一个goroutine成本非常低，创建100万个也完全可以接受

```shell
# verb
%d		十进制整数
%x, %o, %b	十六进制、八进制、二进制
%f, %g, %e	浮点数，如3.141593, 3.141592653589793, 3.141593e+00
%t	布尔型:true或false
%c	字符(Unicode码点)
%s	字符串
%q	带引号字符串(如"abc")或者字符(如'c')
%v	内置格式的任何值
%T	任何值的类型
%%	百分号本身(无操作数)
```

按照约定，诸如log.Printf和fmt.Errorf之类的格式化函数以f结尾，使用和fmt.Printf相同的格式化规则；而那些以ln结尾的函数（如Println）则使用%v的方式来格式化参数，并在最后追加换行符

**Println :可以打印出字符串，和变量**

**Printf : 只可以打印出格式化的字符串,可以输出字符串类型的变量，不可以输出整形变量和整形**

当需要格式化输出信息时一般选择 Printf，其他时候用 Println 就可以了

```go
a := 10
fmt.Println(a)　　//right
fmt.Println("abc")　　//right
fmt.Printf("%d",a)　　//right
fmt.Printf(a)　　//error
```

```go
package main
import "fmt"
import "os"
type point struct {
    x, y int
}
func main() {
    //Go 为常规 Go 值的格式化设计提供了多种打印方式。例如，这里打印了 point 结构体的一个实例。
    p := point{1, 2}
    fmt.Printf("%v\n", p) // {1 2}
    //如果值是一个结构体，%+v 的格式化输出内容将包括结构体的字段名。
    fmt.Printf("%+v\n", p) // {x:1 y:2}
    //%#v 形式则输出这个值的 Go 语法表示。例如，值的运行源代码片段。
    fmt.Printf("%#v\n", p) // main.point{x:1, y:2}
    //需要打印值的类型，使用 %T。
    fmt.Printf("%T\n", p) // main.point
    //格式化布尔值是简单的。
    fmt.Printf("%t\n", true)
    //格式化整形数有多种方式，使用 %d进行标准的十进制格式化。
    fmt.Printf("%d\n", 123)
    //这个输出二进制表示形式。
    fmt.Printf("%b\n", 14)
    //这个输出给定整数的对应字符。
    fmt.Printf("%c\n", 33)
    //%x 提供十六进制编码。
    fmt.Printf("%x\n", 456)
    //对于浮点型同样有很多的格式化选项。使用 %f 进行最基本的十进制格式化。
    fmt.Printf("%f\n", 78.9)
    //%e 和 %E 将浮点型格式化为（稍微有一点不同的）科学技科学记数法表示形式。
    fmt.Printf("%e\n", 123400000.0)
    fmt.Printf("%E\n", 123400000.0)
    //使用 %s 进行基本的字符串输出。
    fmt.Printf("%s\n", "\"string\"")
    //像 Go 源代码中那样带有双引号的输出，使用 %q。
    fmt.Printf("%q\n", "\"string\"")
    //和上面的整形数一样，%x 输出使用 base-16 编码的字符串，每个字节使用 2 个字符表示。
    fmt.Printf("%x\n", "hex this")
    //要输出一个指针的值，使用 %p。
    fmt.Printf("%p\n", &p)
    //当输出数字的时候，你将经常想要控制输出结果的宽度和精度，可以使用在 % 后面使用数字来控制输出宽度。默认结果使用右对齐并且通过空格来填充空白部分。
    fmt.Printf("|%6d|%6d|\n", 12, 345)
    //你也可以指定浮点型的输出宽度，同时也可以通过 宽度.精度 的语法来指定输出的精度。
    fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)
    //要最对齐，使用 - 标志。
    fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45)
    //你也许也想控制字符串输出时的宽度，特别是要确保他们在类表格输出时的对齐。这是基本的右对齐宽度表示。
    fmt.Printf("|%6s|%6s|\n", "foo", "b")
    //要左对齐，和数字一样，使用 - 标志。
    fmt.Printf("|%-6s|%-6s|\n", "foo", "b")
    //到目前为止，我们已经看过 Printf了，它通过 os.Stdout输出格式化的字符串。Sprintf 则格式化并返回一个字符串而不带任何输出。
    s := fmt.Sprintf("a %s", "string")
    fmt.Println(s)
    //你可以使用 Fprintf 来格式化并输出到 io.Writers而不是 os.Stdout。
    fmt.Fprintf(os.Stderr, "an %s\n", "error")
}
```

## 读写文件

## 格式化文本

## 创建图像

## 在Internet客户端和服务器之间通信

Web服务器使用fmt.Fprintf通过写入http.ResponseWriter来让浏览器显示

## 控制流

```go
// if
// for
// switch
switch coinflip() {
  case "heads":
  	heads++
  case "tails":
  	tails++
  default:
  fmt.Println("landed on edge!")
}

// 无标签选择，等价于 switch true
func Signum(x int) int {
  switch {
    case x > 0:
    	return +1
    default:
    	return 0
  	case x < 0:
    	return -1
  }
}
```

## 方法和接口

一个关联了命名类型的函数称为方法，Go里面的方法可以关联到几乎所有的命名类型

接口可以用相同的方式处理不同的具体类型的抽象类型，它基于这些类型所包含的方法，而不是类型的描述或实现

# 2. 程序结构

## 声明

声明给一个程序实体命名，并且设定其部分或全部属性

变量var、常量const、类型type、函数func

## 变量

var name type = expression

类型和表达式部分可以省略一个，但是不能都省略

如果类型省略，它的类型将由初始化表达式决定

如果表达式省略，其初始值对应于类型的零值——数字0、布尔false、字符串“”、接口和引用类型（slice、指针、map、通道、函数）是nil、数组或结构体这样的复合类型，零值是其所有元素或成员的零值

零值机制保障所有的变量是良好定义的，Go里面不存在未初始化变量

### 短变量声明

一般用在函数中

短变量声明可以用来声明和初始化局部变量  name := expression，name的类型由expression的类型决定

:= 表示声明，而=表示赋值

短变量声明不需要声明所有在左边的变量。如果一些变量在同一个词法块中声明，那么对于那些变量，短声明的行为等同于赋值

短变量声明至少声明一个新变量，否则代码编译将无法通过

### 指针

其值是变量的地址

不是所有的值都有地址，但是所有的变量都有

使用指针，可以在无须知道变量名字的情况下，间接读取或更新变量的值

使用 & 操作符可以获取一个变量的地址，使用 * 操作符可以获取指针引用的变量的值，但是指针不支持算术运算

```go
x := 1
p := &x          // p是整型指针，指向x
fmt.Println(*p)  // 1
*p = 2           // 等于 x = 2
fmt.Println(x)   // 结果 "2"
```

指针类型的 零值是nil

指针是可比较的，两个指针当且仅当指向同一个变量或者两者都是nil的情况下才相等

函数返回局部变量的地址是非常安全的。（每次调用返回一个不同的值-地址）

###  new函数

另一种创建变量的方式是使用内置的new函数

表达式new(T)创建一个未命名的T类型变量，初始化为T类型的零值，并返回其地址（地址类型为*T）

只是语法上的便利，不是一个基础概念，只是不需要引入（和声明）一个虚拟的名字，通过new(T)就可以直接在表达式中使用

每次调用new返回一个具有唯一地址的不同变量

new是一个预声明的函数，不是一个关键字，所以它可以重定义为另外的其他类型

```go
p := new(int)      // *int类型的p，指向未命名的int变量
fmt.Println(*p)    // 输出"0"
*p = 2             // 把未命名的int设置为2
fmt.Println(*p)    // 输出"2"
```

### 变量的生命周期

变量的生命周期是通过它是否可达来确定的，不可访问时，它占用的存储空间被回收

垃圾回收器如何知道一个变量是否应该被回收？

基本思路是每一个包级别的变量，以及每一个当前执行函数的局部变量，可以作为追溯该变量的路径的源头，通过指针和其他方式的引用可以找到变量。如果变量的路径不存在，那么变量变得不可访问，因此它不会影响任何其他的计算过程

编译器可以选择使用堆或栈上的空间来分配，令人惊奇的是，这个选择不是基于使用var或new关键字来声明变量

```go
// x一定使用堆空间，因为它在f函数返回以后还可以从global变量访问，尽管它被声明为一个局部变量，这种情况说x从f中逃逸
// 因为每一次变量逃逸都需要一次额外的内存分配过程，所以记住它在性能优化的时候是有好处的
var global *int
func f() {
  var x int
  x = 1
  global = &x
}

// g函数返回时，变量 *y变得不可访问，可回收，因为*y没有从g中逃逸，所以编译器可以安全地在站上 分配*y，即便使用new函数创建它
func g() {
  y := new (int)
  *y = 1
}
```

```go
// 计算两个整数的最大公约数
func gcd(x, y int) int {
  for y != 0 {
    x, y = y, x%y
  }
  return x
}
```

```go
// 计算斐波拉契数列的第n个数
func fib(n int) int {
  x, y := 0, 1
  for i := 0; i < n; i++ {
    x, y = y, x + y
  }
  return x
}
```

两个值使用==和!=进行比较与可赋值性相关：任何比较中，第一个操作数相对于第二个操作数的类型必须是可赋值的，或者可以反过来赋值

## 类型声明

type声明定义一个新的命名类型，它和某个已有类型使用同样的底层类型

命名类型提供了一种方式来区分底层类型的不同或者不兼容使用，这样它就不会在无意中混用

```go
type Celsius float64
type Fahrenheit float64
// 即使使用相同的底层类型，他们也不是相同的类型，所以他们不能使用算数表达式进行比较和合并
```

对于每个类型T，都有一个对应的类型转换操作T(x)将值x转换为类型T。如果两个类型具有相同的底层类型或二者都是指向相同底层类型变量的未命名指针类型，则二者是可以相互转换的

类型转换不改变类型值的表达式，仅改变类型。如果x对于类型T是可赋值的，类型转换也是允许的，但通常是不必要的

```go
// 通过==和<之类的比较操作符，命名类型的值可以与其相同类型的值或底层类型相同的未命名类型的值相比较
// 但是不同命名类型的值不能直接比较
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)           // "true"
fmt.Println(f >= 0)           // "true"
fmt.Println(c == f)           // 编译错误：类型不匹配
fmt.Println(c == Celsius(f))  // "true"
```

## 包和文件

每个包给它的声明提供独立的命名空间

包的初始化从初始化包级别的变量开始，这些变量按照声明顺序初始化，在依赖已解析完毕的情况下，根据以来的顺序进行

```go
var a = b + c   // 最后把a初始化为3
var b = f()     // 通过调用f接着把b初始化为2
var c = 1       // 首先初始化为1
func f() int {return c + 1}
```

```go
if f, err := os.Open(fname); err != nil {        // 编译错误，未使用f
  return err
}
f.Stat()      // 编译错误，未定义f
f.Close()     // 编译错误，未定义f

f, err := os.Open(fname)

if f, err := os.Open(fname); err != nil {        
  return err
} else {
  // f与err在这里可见
  f.Stat()
  f.Close()
}
```

## 作用域

任何文件可以包含任意数量的声明init函数

init函数不能被调用和被引用，它也是普通的函数，当程序启动时，init函数按照它们声明的顺序自动执行

```go
var cwd string
func init() {
  cwd, err := os.Getwd() 
  if err != nil {
    log.Fatalf("os.Getwd failed: %v", err)
  }
  log.Printf("Working dierctory = %s", cwd)
}
// 全局的cwd变量依然未初始化
// 解决
// 在另一个var声明中声明err，避免使用 :=
var cwd string
func init() {
  var err error
  cwd, err = os.Getwd()
  if err != nul {
    log.Fatalf("os.Getwd failed: %v", err)
  }
}
```

# 3. 基本数据

基础类型（数字、字符串、布尔）、聚合类型（数组、结构体）、引用类型（指针、slice、map、函数、通道）、接口类型

## 数值

### 整数

有符号整数（int8、int16、in32、int64）和无符号整数（uint8、uint16、uint32、uint64）

int是32位或64位，在同样的硬件平台上，不同的编译器可能选用不同的大小

rune类型是int32类型的同义词，常用于指明一个值是Unicode码点

byte类型是uint8类型的同义词，强调一个值是原始数据而非量值

无符号整数uintprt，其大小并不明确，但足以完整存放指针

int和int32是不同的类型，尽管int天然的大小就是32位，并且int值若要做当作int32使用，必须显示转换，反之亦然

有符号整数以补码表示，保留最高位作为符号位，n位数字的取值范围是 -2(n-1) ~ 2(n-1)-1

无符号整数由全部位构成其非负值，范围是0~2n - 1

例如int8可以从-128到127取值，uint8从0到255取值

优先级的降序排列

```go
* / % << >> & &^
+ - | ^
== != < <= > >=
&&
||
```

取模余数的正负号总是与被除数一致， -5%3 和 -5%-3都是-2

除法的行为取决于操作数是否都为整型，整数相除，商会舍弃小数部分，5.0/4.0得到1.25，而5/4为1

算术上，左移运算 x<<n等价于x乘以2^n，右移运算 x>> n等价于x除以2^n，向下取整

### 浮点数

float32和float64

十进制下，float32的有效数字大约是6位，float64的有效数字大约是15位，绝大多数情况下，优先使用float64

### 复数

complex64和complex128，二者分别由float32和float64构成

内置的real函数和imag函数分别提取复数的实部和虚部

## 布尔值

&&和||，可能引起短路行为：左边能确定结果，则不计算右边

布尔值无法隐式转换成数值（如0或1），反之也不行

## 字符串

不可变

[m:n]、len(str)

可以通过比较运算符做比较，如==和<，按字节进行，结果服从本身的字典顺序

原生的字符串字面量的书写形式``，使用反引号而不是双引号，转移序列不起作用，实质内容与字面量写法严格一致

go的range循环也适用于字符串

若将一个整数值转换成字符串，其值按文字符号类型解读，并且产生代表该文字符号值的UTF-8码

```go
fmt.Println(string(65))     // "A"而不是"65“
fmt.Println(string(0x4eac))     // “京”
```

标准包：bytes、strings、strconv、unicode

将整数转换成字符串，一种使用fmt.Sprintf，另一种用函数strconv.Itoa(x)

strconv包内的Atoi函数或ParseInt函数用于解释表示整数的字符串，而ParseUnit用于无符号整数

## 常量

布尔型、字符串或数字

iota从0开始取值，逐项加1

```go
type Weekday int
const (
	Sunday Weekday = iota      // 0
  Monday                     // 1
  Tuesday                    // 2
  Wednesday
  Thursday
  Friday
  Saturday
)
```

只有常量才可以是无类型的

# 4. 复合数据类型

数组和结构体的长度都是固定的，slice和map都是动态数据结构

## 数组

元素类型相同，长度固定

数组的长度是数组类型的一部分，[3]int和[4]int是两种不同的数组类型

如果数组的元素类型是可比较的，则数组是可比较的==，比较结果是两边元素是否完全相同

## Slice

拥有相同类型元素的可变长度序列，[]T

可以用来访问数组的部分或者全部的元素

指针、长度和容量，指针指向数组的第一个可以从slice中访问的元素

```go
// 就地反转一个整型slice中的元素
func reverse(s []int) {
  for i, j := 0, len(s) - 1; i < j; i, j = i+1, j-1 {
    s[i], s[j] = s[j],s[i]
  }
}
```

```go
// 将一个slice左移n个元素的简单方法是连续调用reverse函数三次
// 第一次反转前n个元素，第二次反转剩下的元素，最后对整个slice再做一次反转（如果将元素右移n个元素，那么先做第三次调用）
s := []int{0,1,2,3,4,5}
// 向左移动两个元素
reverse(s[:2])
reverse(s[2:])
reverse(s)
fmt.Println(s)    // "[2,3,4,5,0,1]"
```

slice没有指定长度，也按照顺序/索引指定元素

slice无法做比较，不能用==，因为和数组元素不同，slice的元素是非直接的，有可能slice可以包含它自身，还有如果底层数组元素改变，同一个slice在不同的时间会拥有不同的元素

slice唯一允许的比较操作是和nil做比较

标准库提供了bytes.Equal来比较两个字节slice（[]byte），但是对于其他类型的slice，必须自己写函数来比较

内置函数make可以创建一个具有指定元素类型、长度和容量的slice，其中容量参数可以省略，slice的长度和容量相等

make([]T, len)、make([]T, len, cap)

其实make创建了一个无名数组并返回了它的一个slice，这个数组仅可以通过这个slice来访问

内置函数append，将元素追加到slice的后面  []T = append([]T, T)

每次slice容量的改变都意味着一次底层数组重新分配和元素复制， 1,2,4,4,8,8,8,,8,16,16

可变长度参数 ...

虽然底层数组的元素是间接引用的，但是slice的指针、长度和容量不是

slice可以用来实现栈

```go
stack = append(stack, v) // push v
top := stack[len(stack)-1] // 栈顶
stack = stack[:len(stack)-1] // pop
// 从中间移除一个元素，保持顺序
func remove(slice []int, i int) []int {
  copy(slice[i:], slice[i+1:])
  return slice[:len(slice)-1]
}
func main() {
  s := []int{5,6,7,8,9}
  fmt.Println(remove(s,2)) // "[5,6,8,9]"
}
// 如果不需要维持slice中剩余元素的顺序
func remove(slice []int, i int) []int {
  slice[i] = slice[len(slice)-1]
  return slice[:len(slice)-1]
}
```

```go
// 实现一次遍历就可以完成元素旋转
func main() {
  s := []int{0,1,2,3,4}
  rotate(s,2)
  fmt.Println(s)
}
func rotate(s []int, n int) {
  n %= len(s)
  tmp := append(s, s[:n]...)
  copy(s, tmp[n:])
}
```

```go
// 就地处理函数去除[]string slice中相邻的重复字符串元素
func main() {
	s := []string{"a", "b", "b", "b", "c", "b"}
	s = remove(s)
	fmt.Println(s)
}

func remove(s []string) []string {
	for i := 0; i < len(s)-1; {
		if s[i] == s[i+1] {
			copy(s[i:], s[i+1:])
			s = s[:len(s)-1]
		} else {
			i++
		}
	}
	return s
}
```

```go
// 就地函数将一个UTF-8编码的字节slice中所有相邻的Unicode空白字符缩减为一个ASCII空白字符
func main() {
	b := []byte("哈哈  哈 哈哈  a")
	b = replace(b)
	fmt.Printf("%s\n", b)
}

func replace(b []byte) []byte {
	for i := 0; i < len(b); {
		first, size := utf8.DecodeRune(b[i:])
		if unicode.IsSpace(first) {
			second, _ := utf8.DecodeRune(b[i+size:])
			if unicode.IsSpace(second) {
				copy(b[i:], b[i+size:])
				b = b[:len(b)-size]
			}
		}
		i += size
	}
	return b
}
```

```go
// 翻转一个UTF-8编码的字符串中的字符元素，传入参数是该字符串对应的字节slice类型([]byte)，可以做到不需要重新分配内存就实现该功能
func main() {
	b := []byte("一 二 三")
	reverseUTF8(b)
	fmt.Printf("%s\n", b)
}

func reverseUTF8(b []byte) {
	for i := 0; i < len(b); {
		_, size := utf8.DecodeRune(b[i:])
		reverse(b[i : i+size])
		i += size
	}
	reverse(b)
}

func reverse(b []byte) {
	last := len(b) - 1
	for i := 0; i < len(b)/2; i++ {
		b[i], b[last-i] = b[last-i], b[i]
	}
}
```

## map

键的类型必须是可以通过操作符==来进行比较的数据类型

元素迭代顺序是不固定的

```go
args := make(map[string]int)
ages := map[string]int {
  "alice": 31,
  "charlie":  34,
}
map[string]int{}
delete(ages, "woody")    // 即使不存在也是安全的
// 通过键查找元素，不存在则返回值类型的零值
ages["bob"] = ages["bob"] + 1
ages["bob"] += 1
ages["bob"]++
// 无法获取map元素的地址，因为map的增长可能会导致已有元素被重新散列到新的存储位置
age, ok := ages["bob"]
```

向零值map中设置元素会导致错误，设置元素之前，必须初始化map

和slice一样，map不可比较，唯一合法的比较就是和nil做比较。为了判断两个map是否拥有相同的键和值，必须写一个循环

```go
func equal(x, y map[string]int) bool {
  if len(x) != len(y) {
    return false
  }
  for k, xv := range x {
    if yv, ok := y[k]; !ok || yv != xv {
      return false
    }
  }
  return true
}
```

```go
// 修改charcount的代码来统计字母、数字和其他在Unicode分类中的字符数量，可以使用函数unicode.IsLetter等
type class string

const (
	letter  class = "letter"
	number  class = "number"
	graphic class = "graphic"
	space   class = "space"
	symbol  class = "symbol"
)

func main() {
	classCount := make(map[class]int, 5)
	in := bufio.NewReader(os.Stdin)
	for {
		r, _, err := in.ReadRune() // return rune, nbytes, error
		if err == io.EOF {
			break
		}
		if err != nil {
			fmt.Fprintf(os.Stderr, "%v\n", err)
			os.Exit(1)
		}
		switch {
		case unicode.IsLetter(r):
			classCount[letter]++
		case unicode.IsNumber(r):
			classCount[number]++
		case unicode.IsGraphic(r):
			classCount[graphic]++
		case unicode.IsSpace(r):
			classCount[space]++
		case unicode.IsSymbol(r):
			classCount[symbol]++
		}
	}
	for class, count := range classCount {
		fmt.Printf("class: %s, count = %d\n", class, count)
	}
}
```

```go
//编写一个程序wordfreq来汇总输入文本文件中每个单词出现的次数。在第一次调用Scan之前，需要使用input.Split(bufio.ScanWords)来将文本行按照单词分割而不是行分割
func main() {
	wordCount := make(map[string]int)
	input := bufio.NewScanner(os.Stdin)
	input.Split(bufio.ScanWords)
	for input.Scan() {
		wordCount[input.Text()]++
	}
	for word, count := range wordCount {
		fmt.Printf("word: %s, count= %d\n", word, count)
	}
}
```

## 结构体

结构体是将零个或者多个任意类型的命名变量组合在一起的聚合数据类型

可获取成员变量的地址，然后通过指针来访问它

成员变量的顺序对于结构体同一性很重要，顺序不一样就定义了一个不同的结构体类型

聚合类型不可以包含它自己，同样的限制对数组也适用。但是结构体S中可以定义一个S的指针类型*S，这样可以创建一些递归数据结构，比如链表和树

```go
// Package treesort provides insertion sort using an unbalanced binary tree.
package treesort

type tree struct {
	value       int
	left, right *tree
}

// Sort sorts values in place.
func Sort(values []int) {
	var root *tree
	for _, v := range values {
		root = add(root, v)
	}
	appendValues(values[:0], root)
}

// appendValues appends the elements of t to values in order
// and returns the resulting slice.
func appendValues(values []int, t *tree) []int {
	if t != nil {
		values = appendValues(values, t.left)
		values = append(values, t.value)
		values = appendValues(values, t.right)
	}
	return values
}

func add(t *tree, value int) *tree {
	if t == nil {
		// Equivalent to return &tree{value: value}.
		t = new(tree)
		t.value = value
		return t
	}
	if value < t.value {
		t.left = add(t.left, value)
	} else {
		t.right = add(t.right, value)
	}
	return t
}
```

没有任何成员变量的结构体称为空结构体struct{}

结构体字面量，即通过设置结构体的成员变量来设置

注意小写开头的成员变量不可导出，其他包导入该结构体时，这些成员变量可能无法引用

```go
// 第一种
type Point struct{X, Y int}
p := Point{1, 2}       // 要求按照正确顺序为每个成员变量指定一个值

// 第二种
// 通过指定部分或者全部成员变量的名称和值来初始化结构体变量，因为指定了名字，所以顺序无所谓
anim := git.GIF{LoopCount: nframes}
```

一般将结构体指针直接传递给函数或者从函数中返回

```go
// 创建、初始化一个struct类型的变量并获取它的地址
pp := &Point{1,2}       // 可以直接使用在一个表达式中，例如函数调用
// 等价于
pp := new(Point)
*pp = Point{1, 2}
```

所有成员变量都可以比较，则结构体可比较==、!=，按照成员变量顺序进行比较

可比较的结构体类型都可以作为map的键类型

允许定义不带名称的结构体成员（匿名成员），只需要指定类型即可，这个结构体成员的类型必须是一个命名类型或者指向命名类型的指针

```go
type Circle struct {
  Point
  Radius int
}
type Wheel struct {
  Circle
  Spokes int
}
// 可以直接访问需要的变量而不是指定一大串中间变量
var w Wheel
w.X = 8           // 等价于 w.Circle.Point.X = 8
w.Radius = 5      // 等价于 w.Circle.Radius = 5
w.Spokes = 20 
```

```go
// 结构体字面量并没有什么快捷方式来初始化结构体
w = Wheel{8, 8, 5, 20}       // 编译错误，未知成员变量
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20}  // 编译错误，未知成员变量
```

```go
w = Wheel{Circle{Point{8, 8}, 5}, 20}
w = Wheel{
  Circle: Circle{
    Point: Point{X: 8, Y: 8},
    Radius: 5,           // 尾部逗号是必须
  },
  Spokes: 20,           // 尾部逗号是必须
}
fmt.Printf("%#v\n", w)     // # 类似Go语法的方式输出对象
```

因为“匿名成员”拥有隐式的名字，所以不能在一个结构体里面定义两个相同类型的匿名成员，否则会引起冲突

由于匿名成员的名字是由他们的类型决定的，因此它们的可导出性也是由它们的类型决定的，例如Point和Circle这两个匿名成员是可导出的，即使这两个结构体是不可导出的(point和circle)，仍然可以使用快捷方式

w.X = 8  // 等价于 w.circle.point.X = 8 （不可导出，所以在包外不允许访问）

以快捷方式访问匿名成员的内部变量同样适用于访问匿名成员的内部方法

## JSON

是一种发送和接收格式化信息的标准，值包括字符串、数字、布尔值、数组和对象

```go
type Movie struct {
    Title  string
    Year   int  `json:"released"`
    Color  bool `json:"color,omitempty"`
    Actors []string
}
// 额外的omitempty选项，表示当Go语言结构体成员为空或零值时不生成JSON对象（这里false为零值）
```

将一个Go语言中类似movies的结构体slice转为JSON的过程叫编组（marshaling）。编组通过调用json.Marshal函数完成

只有可导出的结构体成员才会被编码

```go
// data, err := json.Marshal(movies)
data, err := json.MarshalIndent(movies, "", "    ")
if err != nil {
    log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
```

编码的逆操作是解码，对应将JSON数据解码为Go语言的数据结构，Go语言中一般叫unmarshaling，通过json.Unmarshal函数完成。此时JSON字段的名称关联到Go结构体成员的名称是忽略大小写的

```go
var titles []struct{ Title string }
if err := json.Unmarshal(data, &titles); err != nil {
    log.Fatalf("JSON unmarshaling failed: %s", err)
}
fmt.Println(titles) // "[{Casablanca} {Cool Hand Luke} {Bullitt}]"
// slice将被只含有Title信息值填充，其它JSON成员将被忽略
```

即使对应的JSON对象名是小写字母，每个结构体的成员名也必须为大写

因为用户提供的查询条件可能包含类似`?`和`&`之类的特殊字符，为了避免对URL造成冲突，用url.QueryEscape来对查询中的特殊字符进行转义操作`q := url.QueryEscape(strings.Join(terms, " "))`

基于流式的解码器json.Decoder，它可以从一个输入流解码JSON数据，还有一个针对输出流的json.Encoder编码对象

```go
// SearchIssues queries the GitHub issue tracker.
func SearchIssues(terms []string) (*IssuesSearchResult, error) {
    q := url.QueryEscape(strings.Join(terms, " "))
    resp, err := http.Get(IssuesURL + "?q=" + q)
    if err != nil {
        return nil, err
    }

    // We must close resp.Body on all execution paths.
    // (Chapter 5 presents 'defer', which makes this simpler.)
    if resp.StatusCode != http.StatusOK {
        resp.Body.Close()
        return nil, fmt.Errorf("search query failed: %s", resp.Status)
    }

    var result IssuesSearchResult
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        resp.Body.Close()
        return nil, err
    }
    resp.Body.Close()
    return &result, nil
}
```

## 文本和HTML模版

将格式和代码分离出来以便更安全地修改。这写功能是由text/template和html/template等模板包提供的，它们提供了一个将变量值填充到一个文本或HTML格式的模板的机制

一个模板是一个字符串或一个文件，里面包含了一个或多个由双花括号包含的`{{action}}`对象，这称为操作。每个操作在模版语言里面都对应和一个表达式，提供的简单但强大的功能包括：输出值，选择结构体成员，调用函数和方法，描述控制逻辑（比如if-else语句和range循环），实例化其他的模版

```go
const templ = `{{.TotalCount}} issues:
{{range .Items}}---------------------------------------
Number: {{.Number}}
User: {{.User.Login}}
Title: {{.Title | printf "%.64s"}}
Age: {{.CreatedAt | daysAgo}} days
{{end}}`
// {{range.Items}}和{{end}}操作创建一个循环
// |会将前一个操作的结果当作下一个操作的输入
```

```go
// New创建并返回一个新模版
report, err := template.New("report").
// Funcs添加daysAgo到模版内部可以访问的函数列表中，然后返回这个模版对象
		Funcs(template.FuncMap{"daysAgo": daysAgo}).
// Parse方法
		Parse(templ)
	if err != nil {
		log.Fatal(err)
	}
```

```go
// 模版在编译期间就固定下来
// 提供了一种便捷的错误处理方式，接受一个模版和错误作为参数，检查错误是否为nil，如果不是nil则宕机，否则返回这个模版
var report = template.Must(template.New("issuelist").
		Funcs(template.FuncMap{"daysAgo": daysAgo}).
		Parse(templ))
```

```go
func main() {
	const templ = `<p>A: {{.A}}</p><p>B: {{.B}}</p>`
	t := template.Must(template.New("escape").Parse(templ))
	var data struct {
		A string        // untrusted plain text
		B template.HTML // trusted HTML
	}
	data.A = "<b>Hello!</b>"
	data.B = "<b>Hello!</b>"
	if err := t.Execute(os.Stdout, data); err != nil {
		log.Fatal(err)
	}
}
// A转义了而B没有
```

# 5. 函数

## 函数

函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体

函数的类型称作函数签名。当两个函数拥有相同的形参列表和返回列表时，认为这两个函数的类型或签名是相同的。而形参和返回值的名字不会影响到函数类型，采用简写同样不会影响到函数的类型

实参是按值传递的

形参变量都是函数的局部变量，初始值由调用者提供的实参传递。函数形参以及命名返回值同属于函数最外层作用域的局部变量

偶尔遇到没有函数体的函数声明，这表示该函数不是以Go实现的

Go语言使用可变栈，栈的大小按需增加(初始时很小)

多返回值，返回一个多值结果可以是调用另一个多值返回的函数

一个函数如果有命名的返回值，可以省略return语句的操作数，这称为裸返回，将每个命名返回结果按照顺序返回的快捷方法。不容易理解，应保守使用裸返回

## 错误处理

对于那些将运行失败看作是预期结果的函数，它们会返回一个额外的返回值，通常是最后一个，来传递错误信息。如果导致失败的原因只有一个，额外的返回值可以是一个布尔值，通常被命名为ok

通常，导致失败的原因不止一种，尤其是对I/O操作而言，用户需要了解更多的错误信息。因此，额外的返回值不再是简单的布尔类型，而是error类型

内置的error是接口类型。非空错误类型有一个错误消息字符串，可以通过调用它的Error方法或者通过调用fmt.Println(err)或fmt.Printf("v%",err)直接输出错误消息

与许多其他语言不同，Go语言通过使用普通的值而非异常来报告错误。尽管Go语言有异常机制，但是Go语言的异常只是针对程序bug导致的预料外的错误，而不能作为常规的错误处理方法出现在程序中

异常会陷入带有错误消息的控制流去处理它，通常会导致预期外的结果：错误会以难以理解的栈跟踪信息报告给最终用户，这些信息大都是关于程序结构方面的，而不是简单明了的错误信息

相比之下，Go程序使用通常的控制流机制，比如if和return语句应对错误。这种方式在错误处理逻辑方面要求更加小心谨慎，但这恰恰是设计的要点

### 错误处理策略

最常见是将错误传递下去，使得在子例程中发生的错误变为主调例程的错误

fmt.Errorf使用fmt.Sprintf函数格式化一条错误消息并且返回一个新的错误值，为原始的错误消息不断地添加额外的上下文信息来建立一个可读的错误描述。当错误最终被main函数处理时，它应当能够提供一个从最根本问题到总体故障的清晰因果链

第二种，对于不固定或者不可预测的错误，重试，超出一定重试次数和限定的时间后再报错退出

```go
// WaitForServer attempts to contact the server of a URL.
// It tries for one minute using exponential back-off.
// It reports an error if all attempts fail.
func WaitForServer(url string) error {
	const timeout = 1 * time.Minute
	deadline := time.Now().Add(timeout)
	for tries := 0; time.Now().Before(deadline); tries++ {
		_, err := http.Head(url)
		if err == nil {
			return nil // success
		}
		log.Printf("server not responding (%s); retrying...", err)
		time.Sleep(time.Second << uint(tries)) // exponential back-off
	}
	return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "usage: wait url\n")
		os.Exit(1)
	}
	url := os.Args[1]
	if err := WaitForServer(url); err != nil {
		fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
		os.Exit(1)
	}
}
```

第三，如果依旧不能顺利进行下去，调用者能够输出错误然后优雅地停止程序，但一般这样的处理应该留给主程序部分。通常库函数应当将错误传递给调用者，除非这个错误表示一个内部一致性错误，这意味着内部存在bug

```go
if err := WaitForServer(url); err != nil {
  fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
  os.Exit(1)
  // 一个更加方便的方法是通过调用log.Fatalf实现相同的效果
  // 就和所有的日志函数一样，它会默认将时间和日期作为前缀添加到错误消息前
  // log.Fatalf("Site is down:%v\n", err)
  // 一种更吸引人的输出方式是自己定义命令的名称作为log包的前缀，并且将日期和时间略去
  // log.SetPrefix("wati: ")
  // log.SetFlags(0)
}
```

第四，在一些错误情况下，只记录下错误信息然后程序继续运行。同样的，可以选择使用log包来增加日志的常用前缀

```go
if err := Ping(); err != nil {
  log.Printf("ping failed: %v; networking disabled", err)
  // 并且直接出疏导标准错误流
  fmt.Fprintf(os.Stderr, "ping failed: %v; networking disabled\n", err)
}
```

第五，在某些罕见的情况下可以直接安全地忽略掉整个日志

Go语言的错误处理有特定的规律。

进行错误检查之后，检测到失败的情况往往都在成功之前。如果检测到的失败导致函数返回，成功的逻辑一般不会放在else块中而是在外层的作用域中。函数会有一种通常的形式，就是在开头有一连串的检查用来返回错误，之后跟着实际的函数体一直到最后

文件结束标识

io包保证任何由文件结束引起的读取错误，始终都将会得到一个与众不同的错误——io.EOF

```go
in := bufio.NewReader(os.Stdin)
for {
  r, _, err := in.ReadRune()
  if err == io.EOF {
    break //结束读取
  }
  if err != nil {
    return fmt.Errorf("read failed: %v", err)
  }
}
```

## 函数变量

就像其他值，函数变量也有类型，而且它们可以赋给变量或者传递或者从其他函数中返回。

函数变量可以像其他函数一样调用

```go
func square(n int) int {return n * n}
f := square
fmt.Println(f(3))      //"9"
// 函数类型的零值是nil（空值），调用一个空的函数变量将导致宕机
var f func(int) int
f(3)    // 宕机：调用空函数
// 函数变量可以和空值相比较
var f func(int) int 
if f != nil {
  f(3)
}
```

本身不可比较，所以不可以互相进行比较或者作为键值出现在map中

函数变量使得函数不仅将数据进行参数化，还将函数的行为当作参数进行传递

## 匿名函数

命名函数只能在包级别的作用域进行声明，但是能够使用函数字面量在任何表达式内指定函数变量

函数字面量就像函数声明，但在func关键字后面没有函数的名称，它是一个表达式，它的值称作匿名函数

以这种方式定义的函数能够获取到整个词法环境，因此里层的函数可以使用外层函数中的变量

```go
// 返回另一个函数，类型是func() int
func squares() func() int {
  var x int
  return func() int {
    	x++
    return x*x
  }
}
func main() {
  f := squares()
  fmt.Println(f())    // 1
  fmt.Println(f())    // 4
  fmt.Println(f())    // 9
  fmt.Println(f())    // 16
}
```

函数变量不仅是一段代码还可以拥有状态。里层的匿名函数能够获取和更新外层squares函数的局部变量。这些隐藏的变量引用就是把函数归类为引用类型而且函数变量无法进行比较的原因

函数变量类似于使用闭包方法实现的变量。通常把函数变量称为闭包

```go
// The topsort program prints the nodes of a DAG in topological order.
package main

import (
	"fmt"
	"sort"
)

// prereqs maps computer science courses to their prerequisites.
var prereqs = map[string][]string{
	"algorithms": {"data structures"},
	"calculus":   {"linear algebra"},

	"compilers": {
		"data structures",
		"formal languages",
		"computer organization",
	},

	"data structures":       {"discrete math"},
	"database":              {"data structures"},
	"discrete math":         {"intro to programming"},
	"formal languages":      {"discrete math"},
	"networks":              {"operating systems"},
	"operating systems":     {"data structures", "computer organization"},
	"programming languages": {"data structures", "computer organization"},
}

func main() {
	for i, course := range topoSort(prereqs) {
		fmt.Printf("%d:\t%s\n", i+1, course)
	}
}

func topoSort(m map[string][]string) []string {
	var order []string
	seen := make(map[string]bool)
	var visitAll func(items []string)

	visitAll = func(items []string) {
		for _, item := range items {
			if !seen[item] {
				seen[item] = true
				visitAll(m[item])
				order = append(order, item)
			}
		}
	}

	var keys []string
	for key := range m {
		keys = append(keys, key)
	}

	sort.Strings(keys)
	visitAll(keys)
	return order
}
```

在循环体内将循环变量赋给一个新的局部变量，因为循环变量的作用域的规则限制

变长函数，被调用的时候可以有可变的参数个数

## defer

普通的函数或方法调用，在调用之前加上关键字defer

执行的时候以调用defer语句顺序的倒序进行

defer语句经常使用于成对的操作，比如打开和关闭，连接和断开，加锁和解锁，即使是再复杂的控制流，资源在任何情况下都能够正确释放

正确使用defer语句的地方是在成功获取资源之后

```go
// 例如
resp, err := http.Get(url)
if err != nil {
  return err
}
defer resp.Body.Close()

f, err := os.Open(filename)
if err != nil {
  return nil, err
}
defer f.Close()

var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string)int {
  mu.Lock()
  defer mu.Unlock()
  return m[key]
}
```

```go
package main

import (
	"log"
	"time"
)

func bigSlowOperation() {
  // 别忘了defer语句末尾的圆括号，否则入口的操作会在函数退出时执行而出口的操作永远不会调用
	defer trace("bigSlowOperation")() // don't forget the extra parentheses
	// ...lots of work...
	time.Sleep(10 * time.Second) // simulate slow operation b sleeping
}

func trace(msg string) func() {
	start := time.Now()
	log.Printf("enter %s", msg)
	return func() { log.Printf("exit %s (%s)", msg, time.Since(start)) }
}

func main() {
  // 每次调用时会记录进入函数入口和出口的时间与两者之间的时间差
	bigSlowOperation()
}
```

延迟执行的函数在return语句之后执行，并且可以更新函数的结果变量。因为匿名函数可以得到其外层函数作用域内的变量（包括命名的结果），所以延迟执行的匿名函数可以观察到函数的返回结果

```go
func double(x int) int {
  return x + x
}
func double(x int)(result int) {
  defer func(){fmt.Printf("double(%d) = %d\n", x, result)}()
  _ = double(4) 
  // 输出:
  // "double(4)=8"
}
```

延迟执行的匿名函数能够改变外层函数返回给调用者的结果

```go
func triple(x int)(result int) {
  defer func() { result += x }()
  return double(x)
}
fmt.Println(triple(4))    /// 12
```

因为延迟的函数不到函数的最后一刻是不会执行的，要注意循环里defer语句的使用

```go
for _, filename := range filenames {
  f, err := os.Open(filename)
  if err != nil {
    return err
  }
  defer f.Close()      // 注意：可能会用尽文件描述符
  // ....处理文件 f...
}
// 一种解决的方式是将循环体（包括defer语句）放到另一个函数里，每次循环迭代都会调用文件关闭函数
func _, filename := range filenames {
  if err := doFile(filename); err != nil {
    return err
  }
}
func doFile(filename string) error {
  f, err := os.Open(filename)
  if err != nil {
    return err
  }
  defer f.Close()
  // ...处理文件 f...
}
```

```go
// Fetch saves the contents of a URL into a local file.
// 将HTTP的响应写到本地文件中而不是直接显示在标准输出中
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path"
)

// Fetch downloads the URL and returns the
// name and length of the local file.
func fetch(url string) (filename string, n int64, err error) {
	resp, err := http.Get(url)
	if err != nil {
		return "", 0, err
	}
	defer resp.Body.Close()
	// 获得URL路径最后的一个组成部分作为文件名
	local := path.Base(resp.Request.URL.Path)
	if local == "/" {
		local = "index.html"
	}
	f, err := os.Create(local)
	if err != nil {
		return "", 0, err
	}
	n, err = io.Copy(f, resp.Body)
	// Close file, but prefer error from Copy, if any.
	if closeErr := f.Close(); err == nil {
		err = closeErr
	}
	return local, n, err
}

func main() {
	for _, url := range os.Args[1:] {
		local, n, err := fetch(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch %s: %v\n", url, err)
			continue
		}
		fmt.Fprintf(os.Stderr, "%s => %s (%d bytes).\n", url, local, n)
	}
}
```

```go
// 不改变原本的行为，重写fetch函数以使用defer语句关闭打开的可写的文件
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path"
)

func fetch(url string) (filename string, n int64, err error) {
	resp, err := http.Get(url)
	if err != nil {
		return "", 0, err
	}
	defer resp.Body.Close()

	local := path.Base(resp.Request.URL.Path)
	if local == "/" {
		local = "index.html"
	}
	f, err := os.Create(local)
	if err != nil {
		return "", 0, err
	}

	// 优先用 io.Copy 的错误
	defer func() {
		closeErr := f.Close()
		if err == nil {
			err = closeErr
		}
	}()

	n, err = io.Copy(f, resp.Body)
	return local, n, err
}

func main() {
	for _, url := range os.Args[1:] {
		local, n, err := fetch(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch %s: %v\n", url, err)
			continue
		}
		fmt.Fprintf(os.Stderr, "%s => %s (%d bytes).\n", url, local, n)
	}
}
```

## 宕机

正常的程序执行会终止，goroutine中的所有延迟函数会执行，然后程序会异常退出并留下一条日志消息

只有发生严重错误时才使用宕机，所有的延迟函数以倒序执行，从栈最上面的函数开始一直返回至main函数

可直接调用内置的宕机函数，可以接受任何值作为参数

```go
panic(fmt.Springf("invalid suit %q", s))
```

```go
func main() {
  f(3)
}
func f(x int) {
  fmt.Printf("f(%d)\n", x+0\x) // panics if x == 0则发生宕机
  defer fmt.Printf("defer %d\n", x)
  f(x - 1)
}
// f(3)
// f(2)
// f(1)
// defer 1
// defer 2
// defer 3
// 函数是可以从宕机状态恢复至正常运行状态而不让程序退出
```

```go
func main() {
	defer printStack()
	f(3)
}
// Go语言的宕机机制让延迟执行的函数在栈清理之前调用
func printStack() {
	var buf [4096]byte
	n := runtime.Stack(buf[:], false)
	os.Stdout.Write(buf[:n])
}

func f(x int) {
	fmt.Printf("f(%d)\n", x+0/x) // panics if x == 0
	defer fmt.Printf("defer %d\n", x)
	f(x - 1)
}
```

## 恢复

如果内置的recover函数在延迟函数的内部调用，而且这个包含defer语句的函数发生宕机，recover会终止当前的宕机状态并且返回宕机的值。函数不会从之前宕机的地方继续运行而是正常返回。如果recover在其他任何情况下运行则它没有任何效果且返回nil

```go
// Parse函数中的延迟函数会从宕机状态恢复，并使用宕机值组成一条错误消息
// 理想的写法是使用runtime.Stack将整个调用栈包含进来
// 延迟函数则将错误赋给err结果变量，从而返回给调用者
func Parse(input string)(s *Syntax, err error) {
  defer func() {
    if p := recover(); p != nil {
      err = fmt.Errorf("internal error: %v", p)
    }
  }()
  // ...解析器...
}
```

一般的原则是，不应该尝试去恢复从另一个包内发生的宕机。公共的API应当直接报告错误。同样，也不应该恢复一个宕机，而这段代码却不是由你来维护的，比如调用者提供的回调函数，因为不清楚这样做是否安全

最安全的做法是选择性地使用recover，在宕机过后需要进行恢复的情况本来就不多。可以通过使用一个明确的、非导出类型作为宕机值，之后检测recover的返回值是否是这个类型。如果是，则可以像普通的error那样处理宕机，如果不是，使用同一个参数调用panic以继续触发宕机

```go
// soleTitle returns the text of the first non-empty title element
// in doc, and an error if there was not exactly one.
func soleTitle(doc *html.Node) (title string, err error) {
    type bailout struct{}
    defer func() {
        switch p := recover(); p {
        case nil:       // no panic
        case bailout{}: // "expected" panic
            err = fmt.Errorf("multiple title elements")
        default:
            panic(p) // unexpected panic; carry on panicking
        }
    }()
    // Bail out of recursion if we find more than one nonempty title.
    forEachNode(doc, func(n *html.Node) {
        if n.Type == html.ElementNode && n.Data == "title" &&
            n.FirstChild != nil {
            if title != "" {
                panic(bailout{}) // multiple titleelements
            }
            title = n.FirstChild.Data
        }
    }, nil)
    if title == "" {
        return "", fmt.Errorf("no title element")
    }
    return title, nil
}
```

```go
// 使用panic和recover编写一个不包含return语句但能返回一个非零值的函数
func main() {
	fmt.Println(returnNoZero())
}

// 原理：recover 后不会继续执行，而是直接调用 return
func returnNoZero() (result int) {
	defer func() {
		result = 3
		_ = recover()
	}()
	panic("panic!")
}
```

# 6. 方法

对象就是简单的一个值或者变量，并且拥有其方法，而方法是某种特定类型的函数。面向对象编程就是使用方法来描述每个数据结构的属性和操作，于是，使用者不需要了解对象本身的实现

## 声明

方法的声明和普通函数的声明类似，只是在函数名字前面多了一个参数。这个参数把这个方法绑定到这个参数对应的类型上

```go
type Point struct{ X, Y float64}
// 普通的函数
func Distance(p, q Point) float64 {
  return math.Hypot(q.X - p.X, q.Y - p.Y)
}
// Point类型的方法
func (p Point) Distance(q Point) float64 {
  return math.Hypot(q.X -p.X, q.Y - p.Y)
}
```

参数p称为方法的接收者，用来描述主调方法就像向对象发送消息

调用方法的时候，接收者在方法名的前面

```go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q))   // 5, 函数调用
fmt.Println(p.Distance(q))    // 5, 方法调用
```

表达式p.Distance称作选择子(selector)，因为它为接收者p选择合适的Distance方法

Go可以将方法绑定到任何类型上，可以很方便地为简单的类型（如数字、字符串、slice、map，甚至函数等）定义附加的行为

同一个包下的任何类型都可以声明方法，只要它的类型既不是指针类型也不是接口类型

编译器会通过方法名和接收者的类型决定调用哪一个函数

## 指针接收者的方法

指针来传递变量的地址

```go
func (p *Point) ScaleBy(factor float64) {
  p.X *= factor
  p.Y *= factor
}
// 方法的名字是(*Point).ScaleBy
// 圆括号是必须的，没有圆括号，表达式会被解析为*(Point.ScaleBy)
```

在真实的程序中，习惯上遵循如果Point的任何一个方法使用指针接收者，那么所有的Point方法都应该使用指针接收者，即使有些方法并不一定需要

命名类型（Point）与指向它们的指针（*Point）是唯一可以出现在接收者声明处的类型，而且，为防止混淆，不允许本身是指针的类型进行方法声明

如果接收者p是Point类型的变量，但方法要求一个*Point接收者，可以简写成：p.ScaleBy(2)。实际上编译器会对变量进行&p的隐式转换。只有变量才允许这么做，包括结构体字段，像p.X和数组或者slice的元素，比如perim[0]。不能够对一个不能取地址的Point接收者参数调用 *Point方法，因为无法获取临时变量的地址

在合法的方法调用表达式中，只有下面三种形式的语句才能够成立

```go
// 实参接收者和形参接收者是同一个类型，比如都是T类型或者都是*T类型
Point{1, 2}.Distance(q)     // Point
pptr.ScaleBy(2)             // *Point
// 或者实参接收者是T类型的变量而形参接收者是*T类型，编译器会隐式地获取变量的地址
p.ScaleBy(2)       // 隐式转换为(&p)
// 或者实参接收者是*T类型而形参接收者是T类型，编译器会隐式地解引用接收者，获得实际的取值
pptr.Distance(q)    // 隐式转换为(*pptr)
```

如果所有类型T方法的接收者是T自己（而非*T）,那么复制它的实例是安全的，调用方法的时候都必须进行一次复制

但是任何方法的接收者是指针的情况下，应该避免复制T的实例，因为这么做可能会破坏内部原本的数据

nil是一个合法的接收者。就像一些函数允许nil指针作为实参，方法的接收者也一样，尤其是当nil是类型中有意义的零值

## 通过结构体内嵌组成类型

匿名字段类型可以是个指向命名类型的指针，这个时候，字段和方法间接地来自于所指向的对象

方法只能在命名的类型（比如Point）和指向它们的指针（*Point）中声明，但内嵌帮助我们能够在未命名的结构体类型中声明方法

```go
var (
	mu sync.Mutex // 保护mapping
  mapping = make(map[string]string)
)
func Lookup(key string) string {
  mu.Lock()
  v := mapping[key]
  mu.Unlock()
  return v
}
```

```go
var cache = struct {
  // 内嵌，其Lock和Unlock方法也包含进了结构体中，允许直接使用cache变量本身进行加锁
  sync.Mutex
  mapping map[string]string
} {
  mapping: make(map[string]string),
}
func Lookup(key string) string {
  cache.Lock()
  v := cache.mapping[key]
  cache.Unlock()
  return v
}
```

## 方法变量与表达式

选择子p.Distance可以赋予一个方法变量，它是一个函数，把方法（Point.Distance）绑定到一个接收者p上

函数只需要提供实参而不需要提供接收者就能够调用

```go
p := Point{1, 2}
q := Point{4, 6}
distanceFromP := p.Distance         // 方法变量
fmt.Println(distanceFromP(q))       // 5
var origin Point       // {0,0}
fmt.Println(distanceFromP(origin))   // 2.23606797499979 根号5

scaleP := p.ScaleBy      //方法变量
scaleP(2)            // p变成(2, 4)
scaleP(3)            // p变成(6, 12)
scaleP(10)            // p变成(60, 120)
```

如果包内的API调用一个函数值，并且使用者期望这个函数的行为是调用一个特定接收者的方法，方法变量就非常有用

```go
// 函数time.AfterFunc会在指定的延迟后调用一个函数值
type Rocket struct { /* ... */}
func (r *Rocket) Launch() {/* ... */}
r := new(Rocket)
time.AfterFunc(10 * time.Second, func(){r.Launch()})
// 使用方法变量则可以更简洁
time.AfterFunc(10 * time.Second, r.Launch)
```

和调用一个普通的函数不通，在调用方法的时候必须提供接收者，并且按照选择子的语法进行调用

方法表达式写成T.f或者(*T).f，其中T是类型，是一种函数变量，把原来方法的接收者替换成函数的第一个形参，因此它可以像平常的函数一样调用

```go
p := Point{1, 2}
q := Point{4, 6}

distance := Point.Distance      // 方法表达式
fmt.Println(distance(p, q))     // 5
fmt.Println("%T\n",distance)    // func(Point, Point) float64

scale := (*Point).ScaleBy
scale(&p, 2)
fmt.Println(p)           // {2, 4}
fmt.Printf("%T\n", scale)     // func(*Point, float64)
```

如果需要用一个值来代表多个方法中的一个，而方法都属于同一个类型，方法变量可以帮助调用这个值所对应的方法来处理不同的接收者

```go
// 变量op代表加法或减发，二者都属于Point类型的方法
type Point struct{X, Y float64}
func (p Point) Add(q Point) Point {return Point{p.X + q.X, p.Y + q.Y}}
func (p Point) Sub(q Point) Point {return Point{p.X - q.X, p.Y - q.Y}}
type Path []Point
func (path Path) TranslateBy(offset Point, add bool) {
  var op func(p, q Point) Point
  if add {
    op = Point.Add
  } else {
    op = Point.Sub
  }
  for i := range Path {
    // 调用path[i].Add(offset)或者是path[i].Sub(offset)
    path[i] = op(path[i], offset)
  }
}
```

## 位向量

Go语言的集合通常使用map[T]bool来实现，其中T是元素类型

位向量使用一个无符号整型值的slice，每一位代表集合中的一个元素。如果设置第i位的元素，则认为集合包含i

```go
// An IntSet is a set of small non-negative integers.
// Its zero value represents the empty set.
type IntSet struct {
    words []uint64
}

// Has reports whether the set contains the non-negative value x.
func (s *IntSet) Has(x int) bool {
    word, bit := x/64, uint(x%64)
    return word < len(s.words) && s.words[word]&(1<<bit) != 0
}

// Add adds the non-negative value x to the set.
func (s *IntSet) Add(x int) {
    word, bit := x/64, uint(x%64)
    for word >= len(s.words) {
        s.words = append(s.words, 0)
    }
    s.words[word] |= 1 << bit
}

// UnionWith sets s to the union of s and t.
func (s *IntSet) UnionWith(t *IntSet) {
    for i, tword := range t.words {
        if i < len(s.words) {
            s.words[i] |= tword
        } else {
            s.words = append(s.words, tword)
        }
    }
}
// 因为每一个字都有64个二进制位，所以为了定位x的bit位，我们用了x/64的商作为字的下标，并且用x%64得到的值作为这个字内的bit的所在位置。UnionWith这个方法里用到了bit位的“或”逻辑操作符号|来一次完成64个元素的或计算

var x, y IntSet
x.Add(1)
x.Add(144)
x.Add(9)
fmt.Println(x.String())  // {1 9 144}
y.Add(9)
y.Add(42)
fmt.Println(x.String())   // {1 9 144}
x.UnionWith(&y)
fmt.Println(x.String())   // {1 9 42 144}
fmt.Println(x.Has(9), x.Has(123))     // true false
```

## 封装

如果变量或方法是不能通过对象访问到的，这称作封装的变量或者方法

定义时，首字母大写的标志符是可以从包中导出的

要封装一个对象，必须使用结构体

在Go语言中封装的单元是包而不是类型。无论是在函数内的代码还是方法内的代码，结构体类型内的字段对于同一个包中所有代码都是可见的

封装的三个优点：

1. 因为使用方不能直接修改对象的变量，所以不需要更多的语句用来检查变量的值
2. 隐藏实现细节可以防止使用方依赖的属性发生改变，使得设计者可以更加灵活地改变API的实现而不破坏兼容性
3. 防止使用者肆意地改变对象内的变量

仅仅用来获得或者修改内部变量的函数称为getter和setter

# 7. 接口

接口类型是对其他类型行为的概括与抽象

Go语言的接口的独特之处在于它是隐式实现：对于一个具体的类型，无须声明它实现了哪些接口，只要提供接口所必需的方法即可

这种设计无需改变已有类型的实现，就可以为这些类型创建新的接口，对于那些不能修改包的类型，这一点特别有用

## 接口即约定

具体类型：如果知道了一个具体类型的数据，那么就精确地知道了它是什么以及它能干什么

接口类型：是一种抽象类型，它并没有暴露所含数据的布局或者内部结构，当然也没有那些数据的基本操作，它所提供的仅仅是一些方法而已

拿到一个接口类型的值，无从知道它是什么，能知道的仅仅是它能做什么，或者更精确地讲，仅仅是它提供了哪些方法

```go
fmt.Printf // 把结果发到标准输出（其实是一个文件）
fmt.Sprintf // 把结果以string类型返回
```

可以把一种类型替换为满足同一接口的另一种类型的特性称为可取代性

```go
// Bytecounter demonstrates an implementation of io.Writer that counts bytes.
package main

import (
	"fmt"
)

type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
	*c += ByteCounter(len(p)) // convert int to ByteCounter
	return len(p), nil
}

func main() {
	var c ByteCounter
	c.Write([]byte("hello"))
	fmt.Println(c) // "5", = len("hello")

	c = 0 // reset the counter
	var name = "Dolly"
	fmt.Fprintf(&c, "hello, %s", name)
	fmt.Println(c) // "12", = len("hello, Dolly")
}
```

```go
package main

import (
	"bufio"
	"bytes"
	"fmt"
)
// 使用类似ByteCounte的想法，实现单词和行的计数器。实现时考虑使用bufio.ScanWords
func main() {
	s := "Hello, World!\nHello, 世界！"

	var wc WordCounter
	fmt.Fprintf(&wc, s)
	fmt.Println(wc)

	var lc LineCounter
	fmt.Fprintf(&lc, s)
	fmt.Println(lc)
}

type WordCounter int

func (c *WordCounter) Write(p []byte) (int, error) {
	scanner := bufio.NewScanner(bytes.NewReader(p))
	scanner.Split(bufio.ScanWords)
	for scanner.Scan() {
		*c++
	}
	return len(p), nil
}

type LineCounter int

func (c *LineCounter) Write(p []byte) (int, error) {
	scanner := bufio.NewScanner(bytes.NewReader(p))
	scanner.Split(bufio.ScanLines)
	for scanner.Scan() {
		*c++
	}
	return len(p), nil
}
```

## 接口类型

一个接口类型定义了一套方法，如果一个具体类型要实现该接口，那么必须实现接口类型定义中的所有方法

io.Writer是一个广泛使用的接口，它负责所有可以写入字节的类型的抽象，包括文件、内存缓冲区、网络连接、HTTP客户端、打包器（archiver）、散列器（hasher）等

```go
package io
type Reader interface {
  Read(p []byte) (n int, err error)
}
type Closer interface {
  Close() error
}
// 通过组合已有接口得到新的接口
// 嵌入式接口
type ReadWriter interface {
  Reader
  Writer
}
type ReadWriteCloser interface {
  Reader
  Writer
  Closer
}
```

```go
// 不使用嵌入式来声明
type ReadWriter interface {
  Read(p []byte) (n int, err error)
  Write(p []byte) (n int, err error)
}
// 混合使用两种方式
type ReadWriter interface {
  Read(p []byte) (n int, err error)
  Writer
}
```

方法定义的顺序也是无意义的，真正有意义的只有接口的方法集合

```go
// strings.NewReader函数输入一个字符串，返回一个从字符串读取数据且满足io.Reader接口（也满足其他接口）的值。实现该函数，并且通过它来让HTML分析器支持以字符串作为输入
package main

import (
	"fmt"
	"io"
	"os"

	"golang.org/x/net/html"
)

func main() {
	_, err := html.Parse(NewReader("<h1>Hello</h1>"))
	if err != nil {
		fmt.Fprintf(os.Stderr, "html parse err: %v", err)
		os.Exit(1)
	}
}

type StringReader string

func (s *StringReader) Read(p []byte) (int, error) {
	copy(p, *s)
	return len(*s), io.EOF
}

func NewReader(s string) io.Reader {
	sr := StringReader(s)
	return &sr
}
```

## 实现接口

如果一个类型实现了一个接口要求的所有方法，那么这个类型实现了这个接口

*os.File类型实现了io.Reader、Writer、Closer和ReadeWriter接口

*bytes.Buffer实现了Reader、Writer和ReaderWriter，但没有实现Closer，因为它没有Close方法

通常说一个具体类型“是一个”（is-a）特定的接口类型，这其实代表着该具体类型实现了该接口，比如，*bytes.Buffer是一个io.Writer； *os.File是一个io.ReaderWriter

仅当一个表达式实现了一个接口时，这个表达式才可以赋给该接口

```go
var w io.Writer
w = os.Stdout     // *os.File有Write方法
w = new(bytes.Buffer)  // *bytes.Buffer有write方法
w = time.Second  // 编译错误，time.Duration缺少write方法

var rwc io.ReadWriteCloser
rwc = os.Stdout   // *os.File有Read、Write、Close方法
rwc = new(bytes.Buffer) // 编译错误，*bytes.Buffer缺少Close方法
```

当右侧表达式也是一个接口时，该规则也有效

```go
w = rwc     // io.ReadWriteCloser有Write方法
rwc = w     // 编译错误，io.Writer缺少Close方法
```

对每一个具体类型T，部分方法的接收者就是T，而其他方法的接收者则是*T指针，同时对类型T的变量直接调用`*T`的方法也可以是合法的

# 8. goroutine和通道

## CSP思想

# 9. 使用共享变量实现并发

# 10. 包和go工具

包，即组织库的机制

go工具，提供编译、测试、性能基准测试、程序格式化、文档

# 11. 测试

# 12. 反射

# 13. 低级编程

