# 搭建并行处理管道

2009年开始开源项目

2012年发布1.0版

类型检查：编译时

运行环境：编译成机器代码直接运行

编程范式：面向接口，函数式编程，并发编程

Go语言并发编程

​	采用CSP（Communication Sequential Process） 模型

​	不需要锁，不需要callback

​	并发编程 vs 并行计算

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// http://localhost:8888/?name=woody
	//http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
	//	fmt.Fprintf(writer, "<h1>Hello World %s!</h1>", request.FormValue("name"))
	//})
	//
	//http.ListenAndServe(":8888", nil)

	// 并发执行
	ch := make(chan string)
	for i := 0; i < 5; i++ {
		// go starts a goroutine
		go printHelloWorld(i, ch)
	}
	for {
		msg := <- ch
		fmt.Println(msg)
	}
	time.Sleep(time.Millisecond)
}

func printHelloWorld(i int, ch chan string) {
	for {
		ch <-  fmt.Sprintf("Hello World from goroutine %d!\n", i)
	}
}
```

![image-20201121191156257](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121191156257.png)

![image-20201121191335089](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121191335089.png)

![image-20201121191402883](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121191402883.png)

![image-20201121191422407](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121191422407.png)

![image-20201121191517386](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121191517386.png)

![image-20201121191714350](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121191714350.png)

数组数据源节点 channel的关闭及检测

内部排序节点

归并节点

```go
package pipeline

import "sort"

// <- 表示只能从channel中拿（只出不进）
func ArraySource(a ... int) <-chan int {
	out := make(chan int)
	go func() {
		for _, v := range a {
			out <- v
		}
		close(out)
	}()
	return out
}

func InMemSort(in <- chan int) <-chan int {
	out := make(chan int)
	go func() {
		// Read into memory
		a := []int{}
		for v := range in {
			a = append(a, v)
		}
		// sort
		sort.Ints(a)

		// output
		for _, v := range a {
			out <- v
		}
		close(out)
	}()
	return out
}

func Merge(in1, in2 <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		v1, ok1 := <- in1
		v2, ok2 := <- in2
		for ok1 || ok2 {
			if !ok2 || (ok1 && v1 <= v2) {
				out <- v1
				v1, ok1 = <- in1
			} else {
				out <- v2
				v2, ok2 = <- in2
			}
		}
		close(out)
	}()
	return out
}
```

```go
package main

import (
	"fmt"
	pipeline "my-test/pipeline/node"
)

func main() {

	//p := ArraySource(3,2,6,7,4)
	//for {
	//	if num, ok := <-p; ok {
	//		fmt.Println(num)
	//	} else {
	//		break
	//	}
	//}

	p := pipeline.Merge(
		pipeline.InMemSort(
			pipeline.ArraySource(
				3, 2, 6, 7, 4)),
		pipeline.InMemSort(
			pipeline.ArraySource(
				7, 4, 0, 3, 2, 13, 8)))

	for v:= range p {
		fmt.Println(v)
	}
}
```

------------

## 生成数据文件

```go
package pipeline

import (
	"encoding/binary"
	"io"
	"math/rand"
	"sort"
)

func ReaderSource(reader io.Reader) <-chan int {
	out := make(chan int)
	go func() {
		buffer := make([]byte, 8)
		for {
			n, err := reader.Read(buffer)
			if n > 0 {
				v := int(
					binary.BigEndian.Uint64(buffer))
				out <- v
			}
			if err != nil {
				break
			}
		}
		close(out)
	}()
	return out
}

func WriterSink(writer io.Writer, in <- chan int) {
	for v := range in {
		buffer := make([]byte, 8)
		binary.BigEndian.PutUint64(
			buffer, uint64(v))
		writer.Write(buffer)
	}
}

func RandomSource(count int) <- chan int {
	out := make(chan int)
	go func() {
		for i := 0; i < count; i++ {
			out <- rand.Int()
		}
		// 注意关闭 channel
		close(out)
	}()
	return out
}
```

```go
package main

import (
	"bufio"
	"fmt"
	pipeline "my-test/pipeline/node"
	"os"
)

func main() {
	const filename  = "large.in"
	const n  = 100000000     // 800M
	file, err := os.Create(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	p := pipeline.RandomSource(n)
	//pipeline.WriterSink(file, p)
	// 不加buffer会很慢
	writer := bufio.NewWriter(file)
	pipeline.WriterSink(writer, p)
	// 用了buffer就要flush一下，确保所有数据都能导出去
	writer.Flush()

	file, err = os.Open(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// 不加buffer会很慢
	//p = pipeline.ReaderSource(file)
	p = pipeline.ReaderSource(bufio.NewReader(file))
	count := 0
	for v := range p {
		fmt.Println(v)
		count++
		if count >= 100 {
			break
		}
	}
}
```

## 完整-small

```go
package pipeline

import (
	"encoding/binary"
	"io"
	"math/rand"
	"sort"
)

// <- 表示只能从channel中拿（只出不进）
func ArraySource(a ... int) <-chan int {
	out := make(chan int)
	go func() {
		for _, v := range a {
			out <- v
		}
		close(out)
	}()
	return out
}

func InMemSort(in <- chan int) <-chan int {
	out := make(chan int)
	go func() {
		// Read into memory
		a := []int{}
		for v := range in {
			a = append(a, v)
		}
		// sort
		sort.Ints(a)

		// output
		for _, v := range a {
			out <- v
		}
		close(out)
	}()
	return out
}

func Merge(in1, in2 <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		v1, ok1 := <- in1
		v2, ok2 := <- in2
		for ok1 || ok2 {
			if !ok2 || (ok1 && v1 <= v2) {
				out <- v1
				v1, ok1 = <- in1
			} else {
				out <- v2
				v2, ok2 = <- in2
			}
		}
		close(out)
	}()
	return out
}

func ReaderSource(reader io.Reader, chunkSize int) <-chan int {
	out := make(chan int)
	go func() {
		buffer := make([]byte, 8)
		bytesRead := 0
		for {
			n, err := reader.Read(buffer)
			bytesRead += n
			if n > 0 {
				v := int(
					binary.BigEndian.Uint64(buffer))
				out <- v
			}
			if err != nil ||
				(chunkSize != -1 &&
					bytesRead >= chunkSize ){
				break
			}
		}
		close(out)
	}()
	return out
}

func WriterSink(writer io.Writer, in <- chan int) {
	for v := range in {
		buffer := make([]byte, 8)
		binary.BigEndian.PutUint64(
			buffer, uint64(v))
		writer.Write(buffer)
	}
}

func RandomSource(count int) <- chan int {
	out := make(chan int)
	go func() {
		for i := 0; i < count; i++ {
			out <- rand.Int()
		}
		// 注意关闭 channel
		close(out)
	}()
	return out
}

func MergeN(inputs ... <-chan int) <- chan int {
	if len(inputs) == 1 {
		return inputs[0]
	}
	m := len(inputs) / 2
	// merge inputs[0...m) and inputs [m..end)
	return Merge(
		MergeN(inputs[:m]...),
		MergeN(inputs[m:]...))
}
```

```go
package main

import (
	"bufio"
	"fmt"
	pipeline "my-test/pipeline/node"
	"os"
)

// 先生成 源数据文件
func main() {
	//const filename  = "large.in"
	//const n  = 100000000     // 800M
	const filename  = "small.in"
	const n  = 64

	file, err := os.Create(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	p := pipeline.RandomSource(n)
	//pipeline.WriterSink(file, p)
	// 不加buffer会很慢
	writer := bufio.NewWriter(file)
	pipeline.WriterSink(writer, p)
	// 用了buffer就要flush一下，确保所有数据都能导出去
	writer.Flush()

	file, err = os.Open(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// 不加buffer会很慢
	//p = pipeline.ReaderSource(file)
	p = pipeline.ReaderSource(bufio.NewReader(file), -1)
	count := 0
	for v := range p {
		fmt.Println(v)
		count++
		if count >= 100 {
			break
		}
	}
}
```

```go
package main

import (
	"bufio"
	"fmt"
	pipeline "my-test/pipeline/node"
	"os"
)

func main() {
	// 查看 small.in 和 small.out文件大小是否一致
	p := createPipeline("small.in", 512, 4)
	writerToFile(p, "small.out")
	printFile("small.out")
}

func printFile(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	p := pipeline.ReaderSource(file, -1)
	for v := range p {
		fmt.Println(v)
	}
}

func writerToFile(p <- chan int, filename string) {
	file, err := os.Create(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	defer writer.Flush()

	pipeline.WriterSink(writer, p)
}

func createPipeline(filename string, fileSize, chunkCount int) <-chan int {
	chunkSize := fileSize / chunkCount

	sortResults := [] <- chan int {}

	for i := 0; i < chunkCount; i++ {
		file, err := os.Open(filename)
		if err != nil {
			panic(err)
		}
		file.Seek(int64(i * chunkSize), 0)
		source := pipeline.ReaderSource(
			bufio.NewReader(file), chunkSize)

		sortResults = append(sortResults, pipeline.InMemSort(source))

	}
	return pipeline.MergeN(sortResults...)
}
```

## 完整-large

```go
package pipeline

import (
	"encoding/binary"
	"fmt"
	"io"
	"math/rand"
	"sort"
	"time"
)

var startTime time.Time

func Init() {
	startTime = time.Now()
}

// <- 表示只能从channel中拿（只出不进）
func ArraySource(a ... int) <-chan int {
	out := make(chan int)
	go func() {
		for _, v := range a {
			out <- v
		}
		close(out)
	}()
	return out
}

func InMemSort(in <- chan int) <-chan int {
	out := make(chan int, 1024)
	go func() {
		// Read into memory
		a := []int{}
		for v := range in {
			a = append(a, v)
		}
		fmt.Println("Read done:", time.Now().Sub(startTime))
		// sort
		sort.Ints(a)
		fmt.Println("InMemSort done:", time.Now().Sub(startTime))
		// output
		for _, v := range a {
			out <- v
		}
		close(out)
		fmt.Println("Merge done:", time.Now().Sub(startTime))
	}()
	return out
}

func Merge(in1, in2 <-chan int) <-chan int {
	out := make(chan int, 1024)
	go func() {
		v1, ok1 := <- in1
		v2, ok2 := <- in2
		for ok1 || ok2 {
			if !ok2 || (ok1 && v1 <= v2) {
				out <- v1
				v1, ok1 = <- in1
			} else {
				out <- v2
				v2, ok2 = <- in2
			}
		}
		close(out)
	}()
	return out
}

func ReaderSource(reader io.Reader, chunkSize int) <-chan int {
	out := make(chan int)
	go func() {
		buffer := make([]byte, 8)
		bytesRead := 0
		for {
			n, err := reader.Read(buffer)
			bytesRead += n
			if n > 0 {
				v := int(
					binary.BigEndian.Uint64(buffer))
				out <- v
			}
			if err != nil ||
				(chunkSize != -1 &&
					bytesRead >= chunkSize ){
				break
			}
		}
		close(out)
	}()
	return out
}

func WriterSink(writer io.Writer, in <- chan int) {
	for v := range in {
		buffer := make([]byte, 8)
		binary.BigEndian.PutUint64(
			buffer, uint64(v))
		writer.Write(buffer)
	}
}

func RandomSource(count int) <- chan int {
	out := make(chan int)
	go func() {
		for i := 0; i < count; i++ {
			out <- rand.Int()
		}
		// 注意关闭 channel
		close(out)
	}()
	return out
}

func MergeN(inputs ... <-chan int) <- chan int {
	if len(inputs) == 1 {
		return inputs[0]
	}
	m := len(inputs) / 2
	// merge inputs[0...m) and inputs [m..end)
	return Merge(
		MergeN(inputs[:m]...),
		MergeN(inputs[m:]...))
}
```

```go
package main

import (
	"bufio"
	"fmt"
	pipeline "my-test/pipeline/node"
	"os"
)

// 先生成原始数据文件
func main() {
	const filename  = "large.in"
	const n  = 100000000     // 800M
	// const filename  = "small.in"
	// const n  = 64

	file, err := os.Create(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	p := pipeline.RandomSource(n)
	//pipeline.WriterSink(file, p)
	// 不加buffer会很慢
	writer := bufio.NewWriter(file)
	pipeline.WriterSink(writer, p)
	// 用了buffer就要flush一下，确保所有数据都能导出去
	writer.Flush()

	file, err = os.Open(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// 不加buffer会很慢
	//p = pipeline.ReaderSource(file)
	p = pipeline.ReaderSource(bufio.NewReader(file), -1)
	count := 0
	for v := range p {
		fmt.Println(v)
		count++
		if count >= 100 {
			break
		}
	}
}

func SortDemo() {
	p := pipeline.Merge(
		pipeline.InMemSort(
			pipeline.ArraySource(
				3, 2, 6, 7, 4)),
		pipeline.InMemSort(
			pipeline.ArraySource(
				7, 4, 0, 3, 2, 13, 8)))

	for v:= range p {
		fmt.Println(v)
	}
}
```

```go
package main

import (
	"bufio"
	"fmt"
	pipeline "my-test/pipeline/node"
	"os"
)

func main() {
	// 查看 small.in 和 small.out文件大小是否一致
	p := createPipeline("large.in", 80000000, 4)
	writerToFile(p, "large.out")
	printFile("large.out")
}

func printFile(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	p := pipeline.ReaderSource(file, -1)
	count := 0
	for v := range p {
		fmt.Println(v)
		count++
		if count >= 100 {
			break
		}
	}
}

func writerToFile(p <- chan int, filename string) {
	file, err := os.Create(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	defer writer.Flush()

	pipeline.WriterSink(writer, p)
}

func createPipeline(filename string, fileSize, chunkCount int) <-chan int {
	chunkSize := fileSize / chunkCount
	pipeline.Init()
	sortResults := [] <- chan int {}

	for i := 0; i < chunkCount; i++ {
		file, err := os.Open(filename)
		if err != nil {
			panic(err)
		}
		file.Seek(int64(i * chunkSize), 0)
		source := pipeline.ReaderSource(
			bufio.NewReader(file), chunkSize)

		sortResults = append(sortResults, pipeline.InMemSort(source))

	}
	return pipeline.MergeN(sortResults...)
}
```

![image-20201121205157639](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121205157639.png)

## 网络版

![image-20201121205727320](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121205727320.png)

![image-20201121205821922](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121205821922.png)

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"

	"my-test/pipeline"
)

func main() {
	// Please run ../pipelinedemo/main.go
	// to generate test data

	// Alternatively we can call createPipeline
	// to run everything on a single node.
	p := createNetworkPipeline(
		"large.in", 800000000, 4)

	writeToFile(p, "large.out")
	printFile("large.out")
}

func printFile(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	p := pipeline.ReaderSource(file, -1)
	count := 0
	for v := range p {
		fmt.Println(v)
		count++
		if count >= 100 {
			break
		}
	}
}

func writeToFile(p <-chan int, filename string) {
	file, err := os.Create(filename)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	defer writer.Flush()

	pipeline.WriterSink(writer, p)
}

func createPipeline(
	filename string,
	fileSize, chunkCount int) <-chan int {

	// Assuming fileSize % chunkCount == 0
	// and chunkSize % 8 == 0 for ints on
	// 64-bit arch
	chunkSize := fileSize / chunkCount

	pipeline.Init()

	sortResults := []<-chan int{}
	for i := 0; i < chunkCount; i++ {
		file, err := os.Open(filename)
		if err != nil {
			panic(err)
		}

		file.Seek(int64(i*chunkSize), 0)

		source := pipeline.ReaderSource(
			bufio.NewReader(file), chunkSize)

		sortResults = append(sortResults,
			pipeline.InMemSort(source))
	}

	return pipeline.MergeN(sortResults...)
}

func createNetworkPipeline(
	filename string,
	fileSize, chunkCount int) <-chan int {

	// Assuming fileSize % chunkCount == 0
	// and chunkSize % 8 == 0 for ints on
	// 64-bit arch
	chunkSize := fileSize / chunkCount

	pipeline.Init()

	sortAddr := []string{}
	for i := 0; i < chunkCount; i++ {
		file, err := os.Open(filename)
		if err != nil {
			panic(err)
		}

		file.Seek(int64(i*chunkSize), 0)

		source := pipeline.ReaderSource(
			bufio.NewReader(file), chunkSize)

		addr := ":" + strconv.Itoa(7000+i)
		pipeline.NetworkSink(addr,
			pipeline.InMemSort(source))
		sortAddr = append(sortAddr, addr)
	}

	sortResults := []<-chan int{}
	for _, addr := range sortAddr {
		sortResults = append(sortResults,
			pipeline.NetworkSource(addr))
	}

	return pipeline.MergeN(sortResults...)
}
```

```go
package pipeline

import (
	"encoding/binary"
	"fmt"
	"io"
	"math/rand"
	"sort"
	"time"
)

var startTime time.Time

func Init() {
	startTime = time.Now()
}

// <- 表示只能从channel中拿（只出不进）
func ArraySource(a ... int) <-chan int {
	out := make(chan int)
	go func() {
		for _, v := range a {
			out <- v
		}
		close(out)
	}()
	return out
}

func InMemSort(in <- chan int) <-chan int {
	out := make(chan int, 1024)
	go func() {
		// Read into memory
		a := []int{}
		for v := range in {
			a = append(a, v)
		}
		fmt.Println("Read done:", time.Now().Sub(startTime))
		// sort
		sort.Ints(a)
		fmt.Println("InMemSort done:", time.Now().Sub(startTime))
		// output
		for _, v := range a {
			out <- v
		}
		close(out)
		fmt.Println("Merge done:", time.Now().Sub(startTime))
	}()
	return out
}

func Merge(in1, in2 <-chan int) <-chan int {
	out := make(chan int, 1024)
	go func() {
		v1, ok1 := <- in1
		v2, ok2 := <- in2
		for ok1 || ok2 {
			if !ok2 || (ok1 && v1 <= v2) {
				out <- v1
				v1, ok1 = <- in1
			} else {
				out <- v2
				v2, ok2 = <- in2
			}
		}
		close(out)
	}()
	return out
}

func ReaderSource(reader io.Reader, chunkSize int) <-chan int {
	out := make(chan int)
	go func() {
		buffer := make([]byte, 8)
		bytesRead := 0
		for {
			n, err := reader.Read(buffer)
			bytesRead += n
			if n > 0 {
				v := int(
					binary.BigEndian.Uint64(buffer))
				out <- v
			}
			if err != nil ||
				(chunkSize != -1 &&
					bytesRead >= chunkSize ){
				break
			}
		}
		close(out)
	}()
	return out
}

func WriterSink(writer io.Writer, in <- chan int) {
	for v := range in {
		buffer := make([]byte, 8)
		binary.BigEndian.PutUint64(
			buffer, uint64(v))
		writer.Write(buffer)
	}
}

func RandomSource(count int) <- chan int {
	out := make(chan int)
	go func() {
		for i := 0; i < count; i++ {
			out <- rand.Int()
		}
		// 注意关闭 channel
		close(out)
	}()
	return out
}

func MergeN(inputs ... <-chan int) <- chan int {
	if len(inputs) == 1 {
		return inputs[0]
	}
	m := len(inputs) / 2
	// merge inputs[0...m) and inputs [m..end)
	return Merge(
		MergeN(inputs[:m]...),
		MergeN(inputs[m:]...))

}
```

![image-20201121211037015](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121211037015.png)

![image-20201121211204182](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121211204182.png)

