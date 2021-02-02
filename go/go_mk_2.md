# 迷宫的广度优先搜索

## 广度优先算法

为爬虫实战项目做好准备

应用广泛，综合性强

面试常见

![image-20201121133715859](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121133715859.png)

 ![image-20201121133900768](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121133900768.png)

广度优先：1探索完了，再探索2，依次类推

![image-20201121135611020](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121135611020.png)

![image-20201121134222547](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121134222547.png)

```in
// 原始
6 5
0 1 0 0 0
0 0 0 1 0
0 1 0 1 0
1 1 1 0 0
0 1 0 0 1
0 1 0 0 0

// 结果
0  0  4  5  6
1  2  3  0  7
2  0  4  0  8
0  0  0 10  9
0  0 12 11  0
0  0 13 12 13
```

用循环创建二维slice

使用slice来实现队列

用Fscanf读取文件

对Point的抽象

```go
package main

import (
	"fmt"
	"os"
)

// 读取文件内容
func readMaze(filename string) [][]int {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}

	var row, col int
	fmt.Fscanf(file, "%d %d", &row, &col)

	maze := make([][]int, row)
	for i := range maze {
		maze[i] = make([]int, col)
		for j := range maze[i] {
			fmt.Fscanf(file, "%d", &maze[i][j])
		}
	}

	return maze
}

// 点
type point struct {
	i, j int
}

// 上左，下右
var dirs = [4]point{
	{-1, 0}, {0, -1}, {1, 0}, {0, 1}}

func (p point) add(r point) point {
	return point{p.i + r.i, p.j + r.j}
}

func (p point) at(grid [][]int) (int, bool) {
	// 越界
	if p.i < 0 || p.i >= len(grid) {
		return 0, false
	}
	// 越界
	if p.j < 0 || p.j >= len(grid[p.i]) {
		return 0, false
	}

	return grid[p.i][p.j], true
}

//
func walk(maze [][]int,
	start, end point) [][]int {
	steps := make([][]int, len(maze))
	for i := range steps {
		steps[i] = make([]int, len(maze[i]))
	}

	Q := []point{start}

	// 队列不空
	for len(Q) > 0 {
		cur := Q[0]
		Q = Q[1:]

		// 发现终点 退出
		if cur == end {
			break
		}

		for _, dir := range dirs {
			next := cur.add(dir)
			// maze at next is 0
			// and steps at next is 0
			// and next != start
			val, ok := next.at(maze)
			// 撞墙了不能走，继续下个点
			if !ok || val == 1 {
				continue
			}
			// 已经走过
			val, ok = next.at(steps)
			if !ok || val != 0 {
				continue
			}
			// 回到原点
			if next == start {
				continue
			}
			// 1. steps 填进去
			curSteps, _ := cur.at(steps)     // 当前的步骤数
			steps[next.i][next.j] =
				curSteps + 1
			// 2. 点加入到队列里
			Q = append(Q, next)
		}
	}

	return steps
}

func main() {
	maze := readMaze("maze/maze.in")

	// 打印
	for _, row := range maze {
		for _, val := range row {
			fmt.Printf("%d ", val)
		}
		fmt.Println()
	}

	// 起始节点、终点
	steps := walk(maze, point{0, 0},
		point{len(maze) - 1, len(maze[0]) - 1})

	for _, row := range steps {
		for _, val := range row {
			fmt.Printf("%3d", val)
		}
		fmt.Println()
	}
}
```

# 开始实战项目

## 为什么做爬虫项目

有一定的复杂性

可以灵活调整项目的复杂性

平衡语言/爬虫之间的比重

## 网络爬虫分类

通用爬虫，如baidu，google

聚焦爬虫，从互联网获取结构化数据，把网页转换成数据

![image-20201121140937997](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121140937997.png)

## go语言的爬虫库/框架

henrylee2cn/pholcus

gocrawl

colly

hu17889/go_spider

将不使用现成爬虫库/框架

使用ElasticSearch作为数据存储

使用Go语言标准模版库实现http数据展示部分

## 爬虫的主题

爬取内容：如新闻、博客、社区

爬取人：QQ空间、人人网、微博、微信、Facebook？动态的网页，目前不考虑爬取

爬取相亲网站、求职网站

## 尝试人工获取

如何发现用户

​	通过城市列表 - 城市 -（下一页）- 用户

​	通过用户 - 猜你喜欢

​	通过已有用户id + 1 来猜测用户 id

![image-20201121142127166](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121142127166.png)

![image-20201121142245464](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20201121142245464.png)

爬虫实现步骤

​	单任务

​	并发版

​	分布式：加网络接口

# 单任务版爬虫

获取并打印所有城市第一页用户的详细信息

正则表达式

# 并发版爬虫

# 数据存储和展示

# 分布式爬虫

# 总结