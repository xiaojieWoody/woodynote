# todo

爬虫实战等语法都很熟悉后再看

官方文档中文版：http://docscn.studygolang.com/doc/



Go语言很特别

1. 没有“对象”，没有继承、多态，没有泛型，没有try/catch
2. 有接口，函数式编程，CSP并发模型（goroutine + channel）
3. 学习Go语言很简单，因为语法简单
4. 学好Go语言不容易，因为要调整三观



基本语法：变量、选择、循环、指针、数组、容器

面向接口：结构体、duck typing的概念、组合的思想

函数式编程：闭包的概念、多样的例题

工程化：资源管理，错误处理、测试和文档、性能调优

并发编程：goroutine和channel、理解调度器、多样的例题



实战项目架构

![image-20201114220515365](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201114220515365.png)

实战项目的实现步骤：

​	单任务版 -> 并发版 -> 分布式

# 基础语法

## 变量

变量定义

```go
// 变量定义：var关键字
var a,b,c bool
var s1,s2 string = "hello","world"
// 可放在函数内，或直接放在包内
// 使用var()集中定义变量
var (
	aa = 3
	ss = "kkk"
	bb = true
)
```

```go
// 让编译器自动决定类型
var a,b,i,s1,s2 = true,false,3,"hello","world"

// 使用 := 定义变量
a,b,i,s1,s2 := true,false,3,"hello","world"
// 只能在函数内使用
```

内置变量类型

```go
bool,string
(u)int,(u)int8,(u)int16,(u)int32,(u)int64,uintptr
byte,rune
float32,float64,complex64,complex128
// uintptr：指针
// rune：字符型，相当于char，32位
```

复数

```go
复数：3 + 4i         // 3位实部  4i为虚部
```

强制类型转换

```go
// 类型转换是强制的
var a,b int = 3,4
var c int = math.Sqrt(a*a + b*b)  // 不对
var c int = int(math.Sqrt(float64(a*a + b*b)))
```

常量

```go
const filename = "abc.txt"
const (
		filename = "abc.txt"
		a, b = 3, 4
)
// const数值可作为各种类型使用
const a,b = 3,4
var c int = int(math.Sqrt(a*a + b*b))
```

枚举类型

```go
func enums() {
	const (
		cpp = iota          // iota 自增值种子
		_
		java
		python
		golang
		javascript
	)
	fmt.Println(cpp, java , python, golang, javascript)   // 0 2 3 4 5

	const (
		b = 1 << (10 * iota)
		kb
		mb
		gb
		tb
		pb
	)
	fmt.Println(b, kb, mb, gb, tb, pb) // 1 1024 1048576 1073741824 1099511627776 1125899906842624
}
```

变量回顾

​	变量类型写在变量名之后

​	编译器可推测变量类型

​	没有char,只有rune

​	原生支持复数类型

## if

if条件不需要括号

if的条件里可以赋值

if的条件里赋值的变量作用域就在这个if语句里

```go
if contents, err := ioutil.ReadFile(filename); err != nil {
  fmt.Println(err)
} else {
  fmt.Printf("%s\n", contents)
}
```

## switch

switch会自动break，除非使用fallthrough

switch后可以没有表达式

```go
func grade(score int) string {
	g := ""
	switch {
	case score < 0 || score > 100:
		panic(fmt.Sprintf(
			"Wrong score: %d", score))
	case score < 60:
		g = "F"
	case score < 80:
		g = "C"
	case score < 90:
		g = "B"
	case score < 100:
		g = "A"
	}
	return g
}
```

## for

for的条件里不需要括号

for的条件里可以省略初始条件，结束条件，递增表达式

省略初始条件，相当于while

```go
sum := 0
for i := 1; i <= 100; i++ {
  sum += i
}
```

```go
func convertToBin(n int) string {
	result := ""
	for ; n > 0; n /= 2 {
		lsb := n %2
		result = strconv.Itoa(lsb) + result
	}
	return result
}
```

```go
// 死循环
for {
  fmt.Println("abc")
}
```

```go
func printFile(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}
```

基本语法要点回顾

​	for,if后面的条件没有括号

​	if条件里也可定义变量

​	没有while

​	switch不需要break，也可以直接switch多个条件

## 函数

返回值类型写在最后面

可返回多个值，返回多个值时可以起名字，仅用于非常简单的函数，对于调用者而言没有区别

函数作为参数

没有默认参数，可选参数

```go
func eval(a, b int, op string) int{
	switch op {
	case "+":
		return a +b
	case "-":
		return a - b
	case "*":
		return a * b
	case "/":
		return a / b
	default:
		panic("unsupported operation: " + op)
	}
}
```

```go
// 13 3
func div(a,b int) (int, int) {
	return a /b , a % b
}

func div2(a,b int) (q, r int) {
	return a /b , a % b
}
// q, r := div2(13, 3)
// q, _ := div2(13, 3)       // _ 表示这个值不想要
```

```go
func eval2(a, b int, op string) (int, error){
	switch op {
	case "+":
		return a +b, nil    // 错误时返回nil
	case "-":
		return a - b, nil
	case "*":
		return a * b, nil
	//case "/":
	//	return a / b, nil
	case "/":
		q, _ := div(a, b)        // 可调用函数
		return q, nil
	default:
		return 0, fmt.Errorf("unsupported operation: " + op)
	}
}

// main
if result, err := eval2(3,5,"x"); err != nil {
  fmt.Println("Error:" , err)
} else {
  fmt.Println(result)
}
```

```go
func apply(op func(int, int) int, a,b int) int {
	p := reflect.ValueOf(op).Pointer()         // 反射获取函数指针
	opName := runtime.FuncForPC(p).Name()
	fmt.Printf("Calling function %s with args " + "(%d, %d)", opName, a, b)
	return op(a,b)
}

func pow(a, b int) int  {
	return int(math.Pow(float64(a), float64(b)))
}

//fmt.Println(apply(pow, 3, 4))
fmt.Println(apply(
  func(a int, b int) int {
    return int(math.Pow(float64(a), float64(b)))
  }, 3, 4))
```

```go
func sum(numbers ... int) int {
	s := 0
	for i := range numbers {
		s += numbers[i]
	}
	return s
}
```

## 指针

指针不能运算

Go语言只有值传递一种方式

```go
func swap(a, b int) {
	b, a = a, b
}

func swap2( a, b *int)  {
	*b, *a = *a, *b
}

func swap3(a, b int) (int, int) {
	return b, a
}

a, b := 3, 4
swap(a, b)        // 不能交换
fmt.Println(a, b)

c, d := 5, 6
swap2(&c, &d)        // 可以交换
fmt.Println(c, d)

e, f := 7, 8
e, f = swap3(e, f)        // 可以交换
fmt.Println(e, f)
```

# 内建容器

## 数组

数量写在类型前面

可通过_省略变量

不仅range，任何地方都可以通过_省略变量

如果只要i，可写成for i := range numbers

```go
var arr1 [5]int
arr2 := [3]int{1,2,3}
arr3 := [...]int{1,2,3,4,5}
var grid [4][5]int

fmt.Println(arr1, arr2, arr3, grid)
```

```go
// 值拷贝
func printArray(arr [2]int)  {
	arr[0] = 100
	for i, v := range arr {
		fmt.Println(i, v)
	}
	//arr[0] = 100
}

// 指针
func printArray2(arr *[2]int)  {
	arr[0] = 100
	for i, v := range arr {
		fmt.Println(i, v)
	}
}
```

```go
for i := 0; i < len(arr3); i++ {
		fmt.Println(arr3[i])
}

for i := range arr3 {
  fmt.Println(arr3[i])
}

for i, v := range arr3 {
  fmt.Println(i, v)
}

for _, v := range arr3 {
  fmt.Println(v)
}

maxi := -1
maxValue := -1
for i, v := range arr3 {
  if v > maxValue {
    maxi, maxValue = i, v
  }
}
fmt.Println(maxi, maxValue)

sum := 0
for _, v := range arr3 {
  sum += v
}
fmt.Println(sum)
```

数组是值类型

​	[10]int 和 [20]int是不同类型

​	调用func f(arr [10]int)会拷贝数组

​	在go语言中一般不直接使用数组

## 切片

slice

Slice本身没有数据，是对底层array的一个view

```go
arr := [...]int{0,1,2,3,4,5,6,7}
// 包左不包右
s := arr[2:6]   // [2 3 4 5]
s[10] = 10
// Slice本身没有数据，是对底层array的一个view
// arr的值变为[0 1 10 3 4 5 6 7]
```

```go
arr := [...]int{0,1,2,3,4,5,6,7}
s1 := arr[2:6]
s2 := s1[3:5]
// s1的值 2 3 4 5 
// s2的值 5 6
```

```go
arr := [...]int{0,1,2,3,4,5,6,7}
//s := arr[2:6] // 3,4,5,6
fmt.Println("arr[2:6] = ", arr[2:6])  // [2 3 4 5]
fmt.Println("arr[:6] = ", arr[:6])  // [0 1 2 3 4 5]
s1 := arr[2:]
fmt.Println("s1 = ", s1)     // [2 3 4 5 6 7]
s2 := arr[:]
fmt.Println("s2 = ", s2)     // [0 1 2 3 4 5 6 7]

fmt.Println("After updateSlice(s1)")
updateSlice(s1)
fmt.Println(s1)      // [100 3 4 5 6 7]
fmt.Println(arr)     // [0 1 100 3 4 5 6 7]

fmt.Println("After updateSlice(s2)")
updateSlice(s2)
fmt.Println(s2)      // [100 1 100 3 4 5 6 7]
fmt.Println(arr)     // [100 1 100 3 4 5 6 7]

fmt.Println("Reslice")
fmt.Println(s2)    // [100 1 100 3 4 5 6 7]
s2 = s2[:5]
fmt.Println(s2)    // [100 1 100 3 4]
s2 = s2[2:]
fmt.Println(s2)    // [100 3 4]

fmt.Println("Extending slice")
arr[0], arr[2] = 0, 2
fmt.Println("arr =", arr) // arr = [0 1 2 3 4 5 6 7]
s1 = arr[2:6]
s2 = s2[3:5]
fmt.Println("s1=%v, len(s1)=%d, cap(s1)=%d\n", s1, len(s1), cap(s1))  // [2 3 4 5] 4 6
fmt.Println("s2=%v, len(s2)=%d, cap(s2)=%d\n", s2, len(s2), cap(s2))  // [5 6] 2 3

//s1 := arr[2:6]
//s2 := s1[3:5]
s3 := append(s2, 10)
s4 := append(s3, 11)
s5 := append(s4, 12)
fmt.Println("s3, s4, s5=", s3, s4, s5)    // s3, s4, s5= [5 6 10] [5 6 10 11] [5 6 10 11 12]
fmt.Println("arr =", arr)             // arr = [0 1 2 3 4 5 6 10]
```

![·](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201115103841410.png)

![image-20201115103939329](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201115103939329.png)

slice可以向后扩展，不可以向前扩展

s[i]不可以超越len(s)，向后扩展不可以超越底层数组cap(s)

```go
s3 := append(s2, 10)
s4 := append(s3, 11)
s5 := append(s4, 12)
fmt.Println("s3, s4, s5=", s3, s4, s5)    // s3, s4, s5= [5 6 10] [5 6 10 11] [5 6 10 11 12]
fmt.Println("arr =", arr)             // arr = [0 1 2 3 4 5 6 10]
```

向Slice添加元素

​	添加元素时如果超越cap，系统会重新分配更大的底层数组

​	由于值传递的关系，必须接收append的返回值

​	s = append(s, val)

```go
func printSlice(s []int) {
	fmt.Printf("value=%v,len=%d, cap=%d\n", s, len(s), cap(s))
}

func main() {
	fmt.Printf("Creating slice")
	var s []int        // Zero value for slice is nil

	for i:= 0; i < 100; i++ {
		printSlice(s)
		s = append(s, 2 * i + 1)
	}
	fmt.Println(s)

	s1 := []int{2,4,6,8}
	printSlice(s1)    // value=[2 4 6 8],len=4, cap=4

	s2 := make([]int, 16)
	s3 := make([]int, 10, 32)
	printSlice(s2)    // value=[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0],len=16, cap=16
	printSlice(s3)    // value=[0 0 0 0 0 0 0 0 0 0],len=10, cap=32

	fmt.Println("Copying slice")
	copy(s2, s1)
	printSlice(s2)       // value=[2 4 6 8 0 0 0 0 0 0 0 0 0 0 0 0],len=16, cap=16

	fmt.Println("Deleting elements from slice")
	s2 = append(s2[:3], s2[4:]...)
	printSlice(s2)      // value=[2 4 6 0 0 0 0 0 0 0 0 0 0 0 0],len=15, cap=16

	fmt.Println("Popping from front")
	front := s2[0]
	s2 = s2[1:]
	fmt.Println(front)   // 2
	printSlice(s2)      // value=[4 6 0 0 0 0 0 0 0 0 0 0 0 0],len=14, cap=15

	fmt.Println("Popping from back")
	tail := s2[len(s2) -1]
	s3 = s2[:len(s2) -1]
	fmt.Println(tail)       // 0
	printSlice(s2)          // value=[4 6 0 0 0 0 0 0 0 0 0 0 0 0],len=14, cap=15
}
```

## map

创建：make(map[string]int)

获取元素：m[key]

Key不存在时，获得value类型的初始值

用value,ok := m[key]来判断是否存在key

用delete删除一个key

使用range遍历key，或者遍历key,value对

不保证遍历顺序，如需顺序，需手动对key排序

使用len获得元素个数

map[K]V

map[K1]map[K2]V

```go
m := map[string]string {
		"name":"ccmouse",
		"course":"golang",
		"site": "imooc",
		"quality":"notbad",
}

m2 := make(map[string]int)      // m2 == empty map
var m3 map[string]int  // m3 == nil

fmt.Println(m, m2, m3) // map[course:golang name:ccmouse quality:notbad site:imooc] map[] map[]

fmt.Println("Traversing map")
for k, v:= range m {
  fmt.Println(k, v)
}
for k:= range m {
  fmt.Println(k)
}

for _, v:= range m {
  fmt.Println(v)
}

fmt.Println("Getting values")
courseName := m["course"]
fmt.Println(courseName)
caurseName := m["caurse"]
fmt.Println(caurseName)     // 不存在，不会报错，打印空值

courseName1, ok := m["course"]
fmt.Println(courseName1, ok)     // golang true
caurseName1, ok := m["caurse"]
fmt.Println(caurseName1, ok)     //  false，不存在，不会报错，打印空值 

if causeName, ok := m["cause"]; ok {
  fmt.Println(causeName)
} else {
  fmt.Println("key does not exist")
}

fmt.Println("Deleting values")
name, ok := m["name"]
fmt.Println(name, ok)

delete(m, "name")
name, ok = m["name"]
fmt.Println(name, ok)
```

map的key

​	map使用哈希表，必须可以比较相等

​	除了slice，map，function的内建类型都可以作为key

​	Struct类型不包含上述字段，也可作为key

```go
// 寻找最长不含有重复字符的子串
// abcabcbb -> abc
// bbbbb -> b
// pwwkew -> wke
```

![image-20201115144642525](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201115144642525.png)

![image-20201115144720237](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201115144720237.png)

```go
package main

import "fmt"

func lengthOfNonRepeatingSubStr(s string) int {
	lastOccurred := make(map[byte]int)
	start := 0
	maxLength := 0

	for i, ch := range []byte(s) {
		if lastI, ok := lastOccurred[ch]; ok && lastI >= start {
			start = lastI + 1
		}
		if i - start + 1 > maxLength {
			maxLength = i - start + 1
		}
		lastOccurred[ch] = i
	}
	return maxLength
}

func main() {
	fmt.Println(
		lengthOfNonRepeatingSubStr("abcabcbb"))     // 3
	fmt.Println(
		lengthOfNonRepeatingSubStr("bbbbb"))        // 1
	fmt.Println(
		lengthOfNonRepeatingSubStr("pwwkew"))      // 3
	fmt.Println(
		lengthOfNonRepeatingSubStr(""))           // 0
	fmt.Println(
		lengthOfNonRepeatingSubStr("这里是慕课网"))  // 9
	fmt.Println(
		lengthOfNonRepeatingSubStr("一二三二一"))    // 5
}
```

## rune

rune相当于go的char

使用range遍历pos,rune对

使用utf8.RuneCountInString获得字符数量

使用len获得字节长度

使用[]byte获得字节

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {

	// 每个中文3个字节
	s := "Yes我爱慕课网!"
	//fmt.Printf("%X\n", []byte(s))
	fmt.Println(s)
	// 59 65 73 E6 88 91 E7 88 B1 E6 85 95 E8 AF BE E7 BD 91 21
	for _, b := range []byte(s) {
		fmt.Printf("%X ", b)
	}
	fmt.Println()
	// (0 59) (1 65) (2 73) (3 6211) (6 7231) (9 6155) (12 8BFE) (15 7F51) (18 21)
	for i, ch := range s {
		fmt.Printf("(%d %X) ", i, ch)
	}
	fmt.Println()

	// 9
	fmt.Println("Rune count:", utf8.RuneCountInString(s))

	bytes := []byte(s)
	// Y e s 我 爱 慕 课 网 !
	for len(bytes) > 0 {
		ch, size := utf8.DecodeRune(bytes)
		bytes = bytes[size:]
		fmt.Printf("%c ", ch)
	}
	fmt.Println()

	// (0 Y) (1 e) (2 s) (3 我) (4 爱) (5 慕) (6 课) (7 网) (8 !) 
	for i, ch := range []rune(s) {
		fmt.Printf("(%d %c) ", i, ch)
	}
	fmt.Println()
}
```

```go
package main

import "fmt"

func lengthOfNonRepeatingSubStr(s string) int {
	lastOccurred := make(map[rune]int)
	start := 0
	maxLength := 0

	for i, ch := range []rune(s) {
		if lastI, ok := lastOccurred[ch]; ok && lastI >= start {
			start = lastI + 1
		}
		if i - start + 1 > maxLength {
			maxLength = i - start + 1
		}
		lastOccurred[ch] = i
	}
	return maxLength
}

func main() {
	fmt.Println(
		lengthOfNonRepeatingSubStr("abcabcbb"))     // 3
	fmt.Println(
		lengthOfNonRepeatingSubStr("bbbbb"))        // 1
	fmt.Println(
		lengthOfNonRepeatingSubStr("pwwkew"))      // 3
	fmt.Println(
		lengthOfNonRepeatingSubStr(""))           // 0
	fmt.Println(
		lengthOfNonRepeatingSubStr("这里是慕课网"))  // 6
	fmt.Println(
		lengthOfNonRepeatingSubStr("一二三二一"))    // 3
}
```

其他字符串操作

Fields,Split,Join

Contains,Index

ToLower,ToUpper

Trim,TrimRight,TrimLeft

# 面向对象

go语言仅支持封装，不支持继承和多态

go语言没有calss，只有struct

结构创建在堆上还是栈上？不需要知道

```go
// 为结构定义方法
func (node treeNode) print() {
  fmt.Print(node.Value)
}
// 显示定义和命名方法接收者
```

```go
// 使用指针作为方法接收者
func (node *treeNode) setValue(value int) {
  node.Value = value
}
// 只有使用指针才可以改变结构内容
// nil指针也可以调用方法
```

```go
package main

import "fmt"

type treeNode struct {
	value int
	left,  right *treeNode
}

// 接收者，参数传递
func (node treeNode) print() {
	fmt.Print(node.value, " ")
}

// 加或者不加*都是.访问
// 不加*为值传递
//func (node treeNode) setValue(value int) {
//	node.value = value
//}
func (node *treeNode) setValue(value int) {
	node.value = value
}

func (node *treeNode) traverse() {
	if node == nil {
		return
	}
	node.left.traverse()
	node.print()
	node.right.traverse()
}

// 使用自定义工厂函数
// 注意返回了局部变量的地址
func createNode(value int) *treeNode {
	return &treeNode{value: value}
}

func main() {

	var root treeNode

	root = treeNode{value: 3}
	root.left = &treeNode{}
	root.right = &treeNode{5, nil, nil}
	root.right.left = new(treeNode)
	root.left.right = createNode(2)

	//nodes := []treeNode {
	//	{value: 3},
	//	{},
	//	{6, nil, &root},
	//}
	//fmt.Println(nodes)

	//root.print()

	root.right.left.setValue(4)
	root.right.left.print()

	fmt.Println()

	root.print()
	root.setValue(100)

	pRoot := &root
	pRoot.print()
	pRoot.setValue(200)
	pRoot.print()

	root.traverse()
}
```

值接收者 vs 指针接收者

​	要改变内容必须使用指针接收者

​	结构过大也考虑使用指针接收者

​	一致性：如有指针接收者，最好都是指针接收者

​	值接收者是go语言特有

​	值/指针接收者均可接收值/指针

## 封装

名字一般使用CamelCase

首字母大写：public

首字母小写：private

包

​	每个目录一个包

​	main包包含可执行入口

​	为结构定义的方法必须放在同一个包内

​	可以是不同文件

​	如何扩充系统类型或者别人的类型

​		定义别名

​		使用组合

```go
package queue

// A FIFO queue.
type Queue []int

// Pushes the element into the queue.
// 		e.g. q.Push(123)
func (q *Queue) Push(v int) {
	*q = append(*q, v)
}

// Pops element from head.
func (q *Queue) Pop() int {
	head := (*q)[0]
	*q = (*q)[1:]
	return head
}

// Returns if the queue is empty or not.
func (q *Queue) IsEmpty() bool {
	return len(*q) == 0
}
```

```go
package main

import (
	"fmt"
	"imooc.com/ccmouse/learngo/queue"
)

func main() {
	q := queue.Queue{1}

	q.Push(2)
	q.Push(3)
	fmt.Println(q.Pop())      // 1
	fmt.Println(q.Pop())      // 2
	fmt.Println(q.IsEmpty())  // false
	fmt.Println(q.Pop())      // 3
	fmt.Println(q.IsEmpty())  // true
}
```

```go
package tree

import "fmt"

type Node struct {
	Value       int
	Left, Right *Node
}

func (node Node) Print() {
	fmt.Print(node.Value, " ")
}

func (node *Node) SetValue(value int) {
	if node == nil {
		fmt.Println("Setting Value to nil " +
			"node. Ignored.")
		return
	}
	node.Value = value
}

func CreateNode(value int) *Node {
	return &Node{Value: value}
}
```

```go
package main

import (
	"fmt"

	"imooc.com/ccmouse/learngo/tree"
)

type myTreeNode struct {
	node *tree.Node
}

func (myNode *myTreeNode) postOrder() {
	if myNode == nil || myNode.node == nil {
		return
	}

	left := myTreeNode{myNode.node.Left}
	right := myTreeNode{myNode.node.Right}

	left.postOrder()
	right.postOrder()
	myNode.node.Print()
}

func main() {
	var root tree.Node

	root = tree.Node{Value: 3}
	root.Left = &tree.Node{}
	root.Right = &tree.Node{5, nil, nil}
	root.Right.Left = new(tree.Node)
	root.Left.Right = tree.CreateNode(2)
	root.Right.Left.SetValue(4)

	fmt.Print("In-order traversal: ")
	root.Traverse()         // 0 2 3 4 5 

	fmt.Print("My own post-order traversal: ")
	myRoot := myTreeNode{&root}
	myRoot.postOrder()       // 2 0 4 5 3 
	fmt.Println()

	nodeCount := 0
	root.TraverseFunc(func(node *tree.Node) {
		nodeCount++
	})
	fmt.Println("Node count:", nodeCount)    // 5

	c := root.TraverseWithChannel()
	maxNodeValue := 0
	for node := range c {
		if node.Value > maxNodeValue {
			maxNodeValue = node.Value
		}
	}
	fmt.Println("Max node value:", maxNodeValue)   // 5
}
```

```go
package tree

import "fmt"

func (node *Node) Traverse() {
	node.TraverseFunc(func(n *Node) {
		n.Print()
	})
	fmt.Println()
}

func (node *Node) TraverseFunc(f func(*Node)) {
	if node == nil {
		return
	}

	node.Left.TraverseFunc(f)
	f(node)
	node.Right.TraverseFunc(f)
}

func (node *Node) TraverseWithChannel() chan *Node {
	out := make(chan *Node)
	go func() {
		node.TraverseFunc(func(node *Node) {
			out <- node
		})
		close(out)
	}()
	return out
}
```

GOPATH环境变量

官方推荐：所有项目和第三方库都放在同一个GOPATH下

也可以将每个项目放在不同的GOPATH

```shell
# mac配置好GOPATH
export GOROOT=/usr/local/go
export GOPATH=/Users/dingyuanjie/dev_env/mygo/go-code
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN:$GOROOT/bin
export PATH=$PATH:$GOPATH
```

echo $GOPATH

使用gopm来获取无法下载的包

go build来编译

go install 产生pkg文件和可执行文件

go run直接编译运行

```shell
GOPATH下目录结构

src
	git repository 1
	git repository 2
pkg
	git repository 1
	git repository 2
bin
	执行文件1,2,3...
```

# 接口

* ==没看懂，后面需重看==

Go 语言中的接口是一种内置的类型，它定义了一组方法的签名

在接口中只能定义方法签名，不能包含成员变量

```go
// 接口定义
type error interface {
	Error() string
}
// 如果一个类型需要实现 error 接口，那么它只需要实现 Error() string 方法，下面的 RPCError 结构体就是 error 接口的一个实现
type RPCError struct {
	Code    int64
	Message string
}
// Go 语言中接口的实现都是隐式的，只需要实现 Error() string 方法实现了 error 接口
// 在 Go 中：实现接口的所有方法就隐式的实现了接口
// 使用上述 RPCError 结构体时并不关心它实现了哪些接口，Go 语言只会在传递参数、返回参数以及变量赋值时才会对某个类型是否实现接口进行检查
func (e *RPCError) Error() string {
	return fmt.Sprintf("%s, code=%d", e.Message, e.Code)
}
```



接口的定义

​	使用者download  ->  实现者retriever

​	接口由使用者定义

```go
// 接口的定义
type Retriever interface {
  Get(Source string) string
}
func download(retriever Retriever) string {
  return retriever.Get("www.imooc.com")
}
// 接口的实现
// 接口的实现是隐式的
// 只要实现接口里的方法
```

接口变量里有什么

​	接口变量自带指针

​	接口变量同样采用值传递，几乎不需要使用接口的指针

​	指针接收者实现只能以指针方式使用，值接收者都可用

![image-20201116211041949](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201116211041949.png)

![image-20201116211059779](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201116211059779.png)

```go
package mock

import "fmt"

type Retriever struct {
	Contents string
}

func (r *Retriever) String() string {
	return fmt.Sprintf(
		"Retriever: {Contents=%s}", r.Contents)
}

func (r *Retriever) Post(url string,
	form map[string]string) string {
	r.Contents = form["contents"]
	return "ok"
}

func (r *Retriever) Get(url string) string {
	return r.Contents
}
```

```go
package real

import (
	"net/http"
	"net/http/httputil"
	"time"
)

type Retriever struct {
	UserAgent string
	TimeOut   time.Duration
}

func (r *Retriever) Get(url string) string {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}

	result, err := httputil.DumpResponse(
		resp, true)

	resp.Body.Close()

	if err != nil {
		panic(err)
	}

	return string(result)
}
```

```go
package main

import (
	"fmt"
	"time"
	"imooc.com/ccmouse/learngo/retriever/mock"
	"imooc.com/ccmouse/learngo/retriever/real"
)

type Retriever interface {
	Get(url string) string
}

type Poster interface {
	Post(url string,
		form map[string]string) string
}

const url = "http://www.imooc.com"

func download(r Retriever) string {
	return r.Get(url)
}

func post(poster Poster) {
	poster.Post(url,
		map[string]string{
			"name":   "ccmouse",
			"course": "golang",
		})
}

type RetrieverPoster interface {
	Retriever
	Poster
}

func session(s RetrieverPoster) string {
	s.Post(url, map[string]string{
		"contents": "another faked imooc.com",
	})
	return s.Get(url)
}

func main() {
	var r Retriever

	mockRetriever := mock.Retriever{
		Contents: "this is a fake imooc.com"}
	r = &mockRetriever
	inspect(r)

	r = &real.Retriever{
		UserAgent: "Mozilla/5.0",
		TimeOut:   time.Minute,
	}
	inspect(r)

	// Type assertion
	if mockRetriever, ok := r.(*mock.Retriever); ok {
		fmt.Println(mockRetriever.Contents)
	} else {
		fmt.Println("r is not a mock retriever")
	}

	fmt.Println(
		"Try a session with mockRetriever")
	fmt.Println(session(&mockRetriever))
}

func inspect(r Retriever) {
	fmt.Println("Inspecting", r)
	fmt.Printf(" > Type:%T Value:%v\n", r, r)
	fmt.Print(" > Type switch: ")
	switch v := r.(type) {
	case *mock.Retriever:
		fmt.Println("Contents:", v.Contents)
	case *real.Retriever:
		fmt.Println("UserAgent:", v.UserAgent)
	}
	fmt.Println()
}
```

查看接口变量

​	表示任何类型：interface{}

​	Type Assertion

​	Type Switch

```go
// 接口的组合
type ReadWriter interface {
  Reader
  Writer
}

// 特殊接口
Stringer
Reader/Writer
```

函数名之前括号中的内容

```go
// 接收器，和函数参数列表中的参数作用一样，只不过接收器是另一种方式调用函数：接收器Name.funcName(xxx)
// 想修改接收器，使用如下的指针
func (s *MyStruct) pointerMethod() { } // method on pointer
// 不需要修改接收器，则可以将接收器定义为如下值
func (s MyStruct)  valueMethod()   { } // method on value
```

```go
package main

import "fmt"
 
type Mutatable struct {
    a int
    b int
}
 
func (m Mutatable) StayTheSame() {
    m.a = 5
    m.b = 7
}
 
func (m *Mutatable) Mutate() {
    m.a = 5
    m.b = 7
}
 
func main() {
    m := &Mutatable{0, 0}
    fmt.Println(m)
    m.StayTheSame()
    fmt.Println(m)
    m.Mutate()
    fmt.Println(m)
}

// 输出
&{0 0}
&{0 0}
&{5 7}
```

# 函数与闭包

```go
func adder() func (value int) int {
  sum := 0
  return func(value int) int {
    sum += value
    return sum
  }
}

func main() {
  adder := adder()
  for i := 0; i < 10; i++ {
    fmt.Println(adder(i))
  }
}
```

函数式编程 vs 函数指针

​	函数是一等公民：参数，变量，返回值都可以是函数

​	高阶函数

​	函数 -> 闭包

“正统”函数式编程

​	不可变性：不能有状态，只有常量和函数

​	函数只能有一个参数

![image-20201116213701153](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201116213701153.png)



![image-20201116215107488](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201116215107488.png)

![image-20201116215252719](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201116215252719.png)

```go
// 斐波拉契数列
package main

import "fmt"

// 1, 1, 2, 3, 5, 8, 13, ...
//    a, b
//       a, b
func Fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}

func main() {
	f := Fibonacci()

	fmt.Println(f())  // 1
	fmt.Println(f())  // 1
	fmt.Println(f())  // 2
	fmt.Println(f())  // 3
	fmt.Println(f())  // 5
	fmt.Println(f())  // 8
}
```

 ```go
// 为函数实现接口
package fib

// 1, 1, 2, 3, 5, 8, 13, ...
func Fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}


package main

import (
	"bufio"
	"fmt"
	"io"
	"strings"

	"imooc.com/ccmouse/learngo/functional/fib"
)

type intGen func() int

func (g intGen) Read(
	p []byte) (n int, err error) {
	next := g()
	if next > 10000 {
		return 0, io.EOF
	}
	s := fmt.Sprintf("%d\n", next)

	// TODO: incorrect if p is too small!
	return strings.NewReader(s).Read(p)
}

func printFileContents(reader io.Reader) {
	scanner := bufio.NewScanner(reader)

	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}

func main() {
	var f intGen = fib.Fibonacci()
	printFileContents(f)
}
 ```

```go
// 使用函数遍历二叉树
// traversal(1).go
```

go语言闭包的应用

​	更为自然，不需要修饰如何访问自由变量

​	没有Lambda表达式，但是有匿名函数

# 资源管理与出错处理

资源管理：defer调用

​	确保调用在函数结束时发生

​	参数在defer语句时计算

​	defer列表为后进先出

```go
package fib

func Fibonacci() func() int {
  a, b := 0,1
  return func() int {
    a, b = b, a + b
    return a
  }
}
```

```go
package main

import (
	"fmt"
	"os"
	"bufio"
	"imooc.com/ccmouse/learngo/functional/fib"
)

func tryDefer() {
  defer fmt.Println(1)
  defer fmt.Println(2)
  fmt.Println(3)
  panic("error occurred")
  fmt.Println(4)
}

func writeFile(filename string) {
  file, err := os.Create(filename)
  if err != nil {
    panic(err)
  }
  defer file.Close()
  
  writer := bufio.NewWriter(file)
  defer writer.Flush()
  
  f := fib.Fibonacci()
  for i := 0; i < 20; i++ {
    fmt.Fprintln(writer, f())
  }
}

func main() {
  writeFile("fib.txt")
}
```

何时使用defer调用

​	Open/Close

​	Lock/Unlock

​	PrintHeader/PrintFooter

![image-20201116223020288](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201116223020288.png)

统一错误处理

​	web.go

panic

​	停止当前函数执行

​	一直向上返回，执行每一层的defer

​	如果没有遇见recover，程序退出

recover

​	仅在defer调用中使用

​	获取panic的值

​	如果无法处理，可重新panic

error vs panic

​	意料之中的：使用error。如：文件打不开

​	意料之外的：使用panic。如：数组越界

错误处理综合示例

​	defer + panic + recover

​	Type Assertion处理错误

​	函数式编程的应用

# 测试与性能调优

## 传统测试

```go
@Test public void testAdd() {
  assertEquals(3, add(1,2));
  assertEquals(2, add(0,2));
  assertEquals(0, add(0,0));
  assertEquals(0, add(-1,1));
  assertEquals(Integer.MIN_VALUE, add(1, Integer.MAX_VALUE));
}
```

测试数据和测试逻辑混在一起

出错信息不明确

一旦一个数据出错测试全部结束

## 表格驱动测试

```go
tests := []struct {
  a, b, c int 32
}{
  {1, 2, 3},
  {0, 2, 2},
  {0, 0, 0},
  {-1, 1, 0},
  {math.MaxInt32, 1, math.MinInt32},
}
for _, test := range tests {
  if actual := add(test.a, test.b); actual != test.c {}
}
```

分离的测试数据和测试逻辑

明确的出错信息

可以部分失败

go语言的语法使得我们更易实践表格驱动测试

## 测试

testing.T的使用

运行测试

```go
package main

import "testing"

// Test开头
func TestTriangle(t *testing.T) {
	tests := []struct{ a, b, c int }{
		{3, 4, 5},
		{5, 12, 13},
		{8, 15, 17},
		{12, 35, 37},
		{30000, 40000, 50000},
	}

	for _, tt := range tests {
		if actual := calcTriangle(tt.a, tt.b); actual != tt.c {
			t.Errorf("calcTriangle(%d, %d); "+
				"got %d; expected %d",
				tt.a, tt.b, actual, tt.c)
		}
	}
}
```

命令行测试
	进入测试代码文件所在目录，执行go test .

​	go test -coverprofile=c.out

​	less c.out

​	go tool cover -html=c.out -o coverage.html    # 生成coverage.html文件，可在浏览器中打开

​	go test -bench .          # 性能测试

​	go test -bench . -cpuprofile cpu.out

​	go tool pprof cpu.out

​		(pprof) web                        # svg图，需要先安装   www.graphviz.org

​		(pprof) quit

```go
package main

import "testing"

func TestSubstr(t *testing.T) {
	tests := []struct {
		s   string
		ans int
	}{
		// Normal cases
		{"abcabcbb", 3},
		{"pwwkew", 3},

		// Edge cases
		{"", 0},
		{"b", 1},
		{"bbbbbbbbb", 1},
		{"abcabcabcd", 4},

		// Chinese support
		{"这里是慕课网", 6},
		{"一二三二一", 3},
		{"黑化肥挥发发灰会花飞灰化肥挥发发黑会飞花", 8},
	}

	for _, tt := range tests {
		actual := lengthOfNonRepeatingSubStr(tt.s)
		if actual != tt.ans {
			t.Errorf("got %d for input %s; "+
				"expected %d",
				actual, tt.s, tt.ans)
		}
	}
}

func BenchmarkSubstr(b *testing.B) {
	s := "黑化肥挥发发灰会花飞灰化肥挥发发黑会飞花"
	for i := 0; i < 13; i++ {
		s = s + s
	}
	b.Logf("len(s) = %d", len(s))
	ans := 8
	b.ResetTimer()          // 忽略上面准备数据的时间

	for i := 0; i < b.N; i++ {
		actual := lengthOfNonRepeatingSubStr(s)
		if actual != ans {
			b.Errorf("got %d for input %s; "+
				"expected %d",
				actual, s, ans)
		}
	}
}
```

使用IDE查看代码覆盖

使用go test获取代码覆盖报告

使用go tool cover查看代码覆盖报告

## 测试HTTP服务器

http测试

​	通过使用假的Request/Response

​		速度快，但是粒度细，像单元测试

​	通过起服务器

```go
package main

import (
	"errors"
	"fmt"
	"io/ioutil"
	"net/http"
	"net/http/httptest"
	"os"
	"strings"
	"testing"
)

func errPanic(_ http.ResponseWriter,
	_ *http.Request) error {
	panic(123)
}

type testingUserError string

func (e testingUserError) Error() string {
	return e.Message()
}

func (e testingUserError) Message() string {
	return string(e)
}

func errUserError(_ http.ResponseWriter,
	_ *http.Request) error {
	return testingUserError("user error")
}

func errNotFound(_ http.ResponseWriter,
	_ *http.Request) error {
	return os.ErrNotExist
}

func errNoPermission(_ http.ResponseWriter,
	_ *http.Request) error {
	return os.ErrPermission
}

func errUnknown(_ http.ResponseWriter,
	_ *http.Request) error {
	return errors.New("unknown error")
}

func noError(writer http.ResponseWriter,
	_ *http.Request) error {
	fmt.Fprintln(writer, "no error")
	return nil
}

var tests = []struct {
	h       appHandler
	code    int
	message string
}{
	{errPanic, 500, "Internal Server Error"},
	{errUserError, 400, "user error"},
	{errNotFound, 404, "Not Found"},
	{errNoPermission, 403, "Forbidden"},
	{errUnknown, 500, "Internal Server Error"},
	{noError, 200, "no error"},
}

func TestErrWrapper(t *testing.T) {
	for _, tt := range tests {
		f := errWrapper(tt.h)
		response := httptest.NewRecorder()
		request := httptest.NewRequest(
			http.MethodGet,
			"http://www.imooc.com", nil)
		f(response, request)

		verifyResponse(response.Result(),
			tt.code, tt.message, t)
	}
}

func TestErrWrapperInServer(t *testing.T) {
	for _, tt := range tests {
		f := errWrapper(tt.h)
		server := httptest.NewServer(
			http.HandlerFunc(f))
		resp, _ := http.Get(server.URL)

		verifyResponse(
			resp, tt.code, tt.message, t)
	}
}

func verifyResponse(resp *http.Response,
	expectedCode int, expectedMsg string,
	t *testing.T) {
	b, _ := ioutil.ReadAll(resp.Body)
	body := strings.Trim(string(b), "\n")
	if resp.StatusCode != expectedCode ||
		body != expectedMsg {
		t.Errorf("expect (%d, %s); "+
			"got (%d, %s)",
			expectedCode, expectedMsg,
			resp.StatusCode, body)
	}
}
```

## 生成文档和示例代码

进入代码文件目录

​	go doc

​	go doc Queue                     # Queue.go

​	go doc IsEmpty

​	go help doc

​	go doc fmt.Println

​	godoc -http :6060            # 浏览器上访问  http://localhost:6060

示例代码

​	可生成在文档中

```go
package queue

import "fmt"

func ExampleQueue_Pop() {
	q := Queue{1}
	q.Push(2)
	q.Push(3)
	fmt.Println(q.Pop())
	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())

	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())

	// Output:
	// 1
	// 2
	// false
	// 3
	// true
}
```

文档

​	用注释写文档

​	在测试中加入Example

​	使用go doc/godoc来查看/生成文档

# Goroutine

```go
package main

import (
	"fmt"
	"time"
)

func main() {
  // 1毫秒内1000个人同时打印
	for i := 0; i < 1000; i++ {
    // go  并发执行
    // 匿名函数  func(){}()
		go func(i int) {
			for {
        // io操作有协程的切换
				fmt.Printf("Hello from "+
					"goroutine %d\n", i)
			}
		}(i)
	}
  // 并发执行，main要阻塞等待for中协程执行一会儿
	time.Sleep(time.Minute)
}
```

协程Coroutine

​	和线程差不多，都是并发执行任务，但是是轻量级“线程”：

​		非抢占式多任务处理，由协程主动交出控制权，由协程内部主动决定什么时候交出控制权（只需处理集中切换的几个点即可，对资源消耗较少）

​			线程是抢占式多任务处理，没有控制权，在任何时候可能被操作系统切换（上下文切换，资源消耗多，比较重）

​		编译器/解释器/虚拟机层面的多任务，不是操作系统（只有线程）层面的多任务，go语言有自己的调度器来调度协程

​		多个协程可能在一个或多个线程上运行，由调度器决定

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
  
	var a [10]int
	for i := 0; i < 10; i++ {
		// 命令：go run -race goroutine.go  检查是否有数据冲突
		// 匿名函数参数中传入 i int，避免和for中的i冲突（data race）
		//（for可能先执行完导致i的值变了，如果匿名函数中使用的是for中的i，则会导致数据冲突问题）
		go func(i int) {
			for {
				// 没有协程的切换，没有主动交出控制权
				a[i]++
				// 主动交出控制权，让其他人也有机会执行   很少会使用这种方法
				runtime.Gosched()
			}
		}(i)
	}
	time.Sleep(time.Minute)
	// 一边print a，一边写 a （a[i]++），导致data race，需要channel来解决
	fmt.Println(a)
}
```

子程序是协程的一个特例

![image-20201118082247205](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201118082247205.png)



![image-20201118082409620](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201118082409620.png)

goroutine的定义

​	任何函数只需加上go就能送给调度器运行

​	不需要在定义时区分是否是异步函数

​	调度器在合适的点进行切换

​	使用-race来检测数据访问冲突

goroutine可能的切换点

​	I/O，select

​	channel

​	等待锁

​	函数调用

​	runtime.Gosched()

​	只是参考，不能保证切换，不能保证在其他地方不切换

# Channel

goroutine与goroutine之间的通道

channel

buffered channel

range

理论基础：Communication Sequential Process（CSP）

![image-20201118083035012](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201118083035012.png)

```go
package main

import (
	"fmt"
	"time"
)

func chanDemo() {
	c := make(chan int)
	go func() {
		for {
			n := <-c
			fmt.Println(n)
		}
	}()
	c <- 1
	c <- 2
	time.Sleep(time.Millisecond)
}

func main() {
	chanDemo()
}
```

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, c chan int) {
	for {
		fmt.Printf("worker %d received %c\n", id, <-c)
	}
}

func chanDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = make(chan int)
		go worker(i, channels[i])
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
  for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}
	time.Sleep(time.Millisecond)
}

func main() {
	chanDemo()
}
```

```go
package main

import (
	"fmt"
	"time"
)
// 这个channel 用来发送数据，不能用来收数据  ，如果是 <-chan int，则这个channel是收数据
func createWorker(id int) chan<- int {
  c := make(chan int)
  go func() {
    for {
      fmt.Printf("Worker %d received %c\n", id, <-c)
    }
  }()
  return c
}
func chanDemo() {
	var channels [10]chan<- int
	for i := 0; i < 10; i++ {
    channels[i] = createWorker(i)
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
  for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}
	time.Sleep(time.Millisecond)
}

func main() {
	chanDemo()
}
```

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, c chan int) {
	for n := range c {
		fmt.Printf("Worker %d received %c\n",
			id, n)
	}
}

func createWorker(id int) chan<- int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func chanDemo() {
	var channels [10]chan<- int
	for i := 0; i < 10; i++ {
		channels[i] = createWorker(i)
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}

	time.Sleep(time.Millisecond)
}

func bufferedChannel() {
	// 缓冲区 3
	c := make(chan int, 3)
	go worker(0, c)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	c <- 'd'
	time.Sleep(time.Millisecond)
}

func channelClose() {
	c := make(chan int)
	go worker(0, c)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	c <- 'd'
	close(c)
	time.Sleep(time.Millisecond)
}

func main() {
	fmt.Println("Channel as first-class citizen")
	chanDemo()
	fmt.Println("Buffered channel")
	bufferedChannel()
	fmt.Println("Channel close and range")
	channelClose()
}
```

==不要通过 共享内存来通信，通过通信来共享内存==

例一：使用Channel来等待goroutine结束，以及WaitGroup的使用

```go
package main

import "fmt"

func doWork1(id int, c chan int, done chan bool) {
	for n := range c {
		fmt.Printf("Worker %d received %c\n", id ,n)
		done <- true
	}
}

type worker1 struct {
	in chan int
  // 通知完成
	done chan bool
}

func createWorker1(id int) worker1 {
	w := worker1{
		in:   make(chan int),
		done: make(chan bool),
	}
	go doWork1(id, w.in, w.done)
	return w
}

func chanDemo1() {
	var workers [10]worker1
	for i := 0; i < 10; i++ {
		workers[i] = createWorker1(i)
	}
	for i, worker := range workers {
		worker.in <- 'a' + i
 	}
	for _, worker := range workers {
		<- worker.done
	}
	for i, worker := range workers {
		worker.in <- 'A' + i
	}
	for _, worker := range workers {
		<- worker.done
	}
}

func main()  {
  // 先小写的10个并发（无序）执行完，然后大写的10个并发（无序）执行完
	chanDemo1()
}
```

系统提供的并行处理方法

```go
package main

import (
	"fmt"
	"sync"
)

func doWork1(id int, c chan int, wg *sync.WaitGroup) {
	for n := range c {
		fmt.Printf("Worker %d received %c\n", id ,n)
		wg.Done()
	}
}

type worker1 struct {
	in chan int
	wg *sync.WaitGroup
}

func createWorker1(id int, wg *sync.WaitGroup) worker1 {
	w := worker1{
		in:   make(chan int),
		wg: wg,
	}
	go doWork1(id, w.in, wg)
	return w
}

func chanDemo1() {
	var wg sync.WaitGroup

	var workers [10]worker1

	for i := 0; i < 10; i++ {
		workers[i] = createWorker1(i, &wg)
	}

	wg.Add(20)
	for i, worker := range workers {
		worker.in <- 'a' + i
 	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}

	wg.Wait()
}

func main()  {
  // 大小写并行（无序）打印
	chanDemo1()
}
```

```go
// 基于上一个重构
package main

import (
	"fmt"
	"sync"
)

func doWork1(id int, w worker1) {
	for n := range w.in {
		fmt.Printf("Worker %d received %c\n", id ,n)
		w.done()
	}
}

type worker1 struct {
	in chan int
	done func()
}

func createWorker1(id int, wg *sync.WaitGroup) worker1 {
	w := worker1{
		in:   make(chan int),
		done: func() {
			wg.Done()
		},
	}
	go doWork1(id, w)
	return w
}

func chanDemo1() {
	var wg sync.WaitGroup

	var workers [10]worker1

	for i := 0; i < 10; i++ {
		workers[i] = createWorker1(i, &wg)
	}

	wg.Add(20)
	for i, worker := range workers {
		worker.in <- 'a' + i
 	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}

	wg.Wait()
}

func main()  {
  // 大小写并行（无序）打印
	chanDemo1()
}
```

例二：使用Channel来实现树的遍历

```go
// entry.go
package main

import (
	"fmt"

	"imooc.com/ccmouse/learngo/tree"
)

type myTreeNode struct {
	node *tree.Node
}

func (myNode *myTreeNode) postOrder() {
	if myNode == nil || myNode.node == nil {
		return
	}

	left := myTreeNode{myNode.node.Left}
	right := myTreeNode{myNode.node.Right}

	left.postOrder()
	right.postOrder()
	myNode.node.Print()
}

func main() {
	var root tree.Node

	root = tree.Node{Value: 3}
	root.Left = &tree.Node{}
	root.Right = &tree.Node{5, nil, nil}
	root.Right.Left = new(tree.Node)
	root.Left.Right = tree.CreateNode(2)
	root.Right.Left.SetValue(4)

	fmt.Print("In-order traversal: ")
	root.Traverse()         // 0 2 3 4 5

	fmt.Print("My own post-order traversal: ")
	myRoot := myTreeNode{&root}
	myRoot.postOrder()       // 2 0 4 5 3
	fmt.Println()

	nodeCount := 0
	root.TraverseFunc(func(node *tree.Node) {
		nodeCount++
	})
	fmt.Println("Node count:", nodeCount)    // 5

	c := root.TraverseWithChannel()
	maxNodeValue := 0
	for node := range c {
		if node.Value > maxNodeValue {
			maxNodeValue = node.Value
		}
	}
	fmt.Println("Max node value:", maxNodeValue)   // 5
}
```

```go
package tree

import "fmt"

type Node struct {
	Value       int
	Left, Right *Node
}

func (node Node) Print() {
	fmt.Print(node.Value, " ")
}

func (node *Node) SetValue(value int) {
	if node == nil {
		fmt.Println("Setting Value to nil " +
			"node. Ignored.")
		return
	}
	node.Value = value
}

func CreateNode(value int) *Node {
	return &Node{Value: value}
}
```

例三：使用select来进行调度

select的使用

定时器的使用

 在select中使用Nil Channel，在数据还没有准备好的时候，把Channel设置成nil，这样select不会阻塞

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator1() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(
				time.Duration(rand.Intn(1500)) *
					time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func main() {
  // 谁先出数据就先选择谁
	var c1, c2 = generator1(), generator1()
	for {
		select {
			case n := <-c1:
				fmt.Println("Received from c1:", n)
			case n := <-c2:
				fmt.Println("Received from c2:", n)
		}
	}
}
```

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(
				time.Duration(rand.Intn(1500)) *
					time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(time.Second)
		fmt.Printf("Worker %d received %d\n",
			id, n)
	}
}

func createWorker(id int) chan<- int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	var c1, c2 = generator(), generator()
	var worker = createWorker(0)

	var values []int
	tm := time.After(10 * time.Second)
	tick := time.Tick(time.Second)
	for {
		var activeWorker chan<- int
		var activeValue int
		if len(values) > 0 {
			activeWorker = worker
			activeValue = values[0]
		}

    // 通过channel互相通信，不需要有等待、锁，但是却能完成并发事情
		select {
		case n := <-c1:
			values = append(values, n)
		case n := <-c2:
			values = append(values, n)
		case activeWorker <- activeValue:
			values = values[1:]
        // select之间的时间，每两次之间的时间
		case <-time.After(800 * time.Millisecond):
			fmt.Println("timeout")
		case <-tick:
			fmt.Println(
				"queue len =", len(values))
		case <-tm:
			fmt.Println("bye")
			return
		}
	}
}
```

传统同步机制

​	少用

​	WaitGroup

​	Mutex

​	Cond

```go
// 查看是否有data race  数据冲突（读写冲突）
// go run -race atomit.go
```

![image-20201121103846284](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121103846284.png)

```go
package main

import (
	"fmt"
	"time"
)
type atomicInt1 int

func (a *atomicInt1) increment1() {
	*a++
}

func (a *atomicInt1) get1() int {
	return int(*a)
}

func main() {
	var a atomicInt1
	a.increment1()
	go func() {
		a.increment1()
	}()
	time.Sleep(time.Millisecond)
	fmt.Println(a)    // 2
}
```

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type atomicInt struct {
	value int
	lock  sync.Mutex
}

func (a *atomicInt) increment() {
	//a.lock.Lock()
	//defer a.lock.Unlock()
	//a.value++
	fmt.Println("safe increment")
	func() {
		a.lock.Lock()
		defer a.lock.Unlock()

		a.value++
	}()
}

func (a *atomicInt) get() int {
	a.lock.Lock()
	defer a.lock.Unlock()

	return a.value
}

func main() {
	var a atomicInt
	a.increment()
	go func() {
		a.increment()
	}()
	time.Sleep(time.Millisecond)
	fmt.Println(a.get())     // 2
}
```

# http及其他标准库

使用http客户端发送请求

使用http.Client控制请求头部等

使用httputil简化工作

```go
package main

import (
	"fmt"
	"net/http"
	"net/http/httputil"
)

func main() {

	resp, err := http.Get("http://www.imooc.com")
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	s, err := httputil.DumpResponse(resp, true)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s\n", s)
}
```

```go
package main

import (
	"fmt"
	"net/http"
	"net/http/httputil"
)

func main() {
	request, err := http.NewRequest(http.MethodGet, "http://www.imooc.com", nil)
	// chrome F12 上扒
	// 手机版
	request.Header.Add("User-Agent",
		"Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1")
	client := http.Client{
		// 先去 https://www.imooc.com  再重定向到 https://m.imooc.com 
		CheckRedirect: func(req *http.Request, via []*http.Request) error {
			fmt.Println("Redicrect:", req)
			return nil
		},
	}
	resp, err := client.Do(request)
	//resp, err := http.DefaultClient.Do(request)
	//resp, err := http.Get("http://www.imooc.com")


	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	s, err := httputil.DumpResponse(resp, true)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s\n", s)
}
```

import_ "net/http/pprof"

访问/debug/pprof/

使用go tool pprof分析性能

```go
// http://localhost:8888/debug/pprof/

package main

import (
	"log"
	"net/http"
	_ "net/http/pprof"
	"os"

	"imooc.com/ccmouse/learngo/errhandling/filelistingserver/filelisting"
)

type appHandler func(writer http.ResponseWriter,
	request *http.Request) error
// 统一错误处理
func errWrapper(
	handler appHandler) func(
	http.ResponseWriter, *http.Request) {
	return func(writer http.ResponseWriter,
		request *http.Request) {
		// panic
		defer func() {
			if r := recover(); r != nil {
				log.Printf("Panic: %v", r)
				http.Error(writer,
					http.StatusText(http.StatusInternalServerError),
					http.StatusInternalServerError)
			}
		}()

		err := handler(writer, request)

		if err != nil {
			log.Printf("Error occurred "+
				"handling request: %s",
				err.Error())

			// user error
			if userErr, ok := err.(userError); ok {
				http.Error(writer,
					userErr.Message(),
					http.StatusBadRequest)
				return
			}

			// system error
			code := http.StatusOK
			switch {
			case os.IsNotExist(err):
				code = http.StatusNotFound
			case os.IsPermission(err):
				code = http.StatusForbidden
			default:
				code = http.StatusInternalServerError
			}
			http.Error(writer,
				http.StatusText(code), code)
		}
	}
}

type userError interface {
	error
	Message() string
}

// http://localhost:8888/abc
// http://localhost:8888/list/fib.txt
// http://localhost:8888/debug/pprof/
func main() {
	http.HandleFunc("/",
		errWrapper(filelisting.HandleFileList))

	err := http.ListenAndServe(":8888", nil)
	if err != nil {
		panic(err)
	}
}

// 获取30秒CPU使用情况
// 命令行窗口：go tool pprof http://localhost:8888/debug/pprof/profile
// 浏览器：这个请求多刷几次  http://localhost:8888/list/errhandling/filelistingserver/web.go
// 命令行窗口：(pprof)web

// 命令行窗口： go tool pprof http://localhost:8888/debug/pprof/heap
// 命令行窗口：(pprof)web
```

## 其他标准库

bufio

encoding/json

log

regexp

time

strings/math/rand

godoc -http :8888    // 浏览器访问http://localhost:8888

go语言标准库文档：https://studygolang.com/pkgdoc

# 正则表达式

```go
package main

import (
	"fmt"
	"regexp"
)

const text = `
	My enail is ccmouse@gmail.com
	email1 is abc@def.org
	email2 is kkk@qq.com
	email2 is ddd@qq.com.cn
	`

func main() {
	re := regexp.MustCompile(`([a-zA-Z0-9]+)@([a-zA-Z0-9]+)(\.[a-zA-Z0-9.]+)`)
	//match := re.FindString(text)
	//match := re.FindAllString(text, -1)     // 找所有的
	match := re.FindAllStringSubmatch(text, -1)
	for _, m := range match {
		fmt.Println(m)
	}
}
```













​	

