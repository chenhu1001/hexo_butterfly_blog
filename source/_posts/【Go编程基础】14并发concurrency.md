---
title: "【Go编程基础】14-并发concurrency"
date: 2022-07-02T18:12:19+08:00
categories: ["Go"]
tags: [Go]
---
# 并发concurrency
- 很多人都是冲着 Go 大肆宣扬的高并发而忍不住跃跃欲试，但其实从
源码的解析来看，goroutine 只是由官方实现的超级“线程池”而已。
不过话说回来，每个实例 4-5KB 的栈内存占用和由于实现机制而大幅
减少的创建和销毁开销，是制造 Go 号称的高并发的根本原因。另外，
goroutine 的简单易用，也在语言层面上给予了开发者巨大的便利。
- 并发不是并行：Concurrency Is Not Parallelism，并发主要由切换时间片来实现“同时”运行，在并行则是直接利用
多核实现多线程的运行，但 Go 可以设置使用核数，以发挥多核计算机
的能力。
- Goroutine 奉行通过通信来共享内存，而不是共享内存来通信。

```go
func main() {
	go Go()
	// time.Sleep(2 * time.Second)
}

func Go() {
	fmt.Println("Go Go Go!")
}

没有输出，因为主线程已经退出了
```

```go
func main() {
	go Go()
	// 利用延迟等待线程打印输出
	time.Sleep(2 * time.Second)
}

func Go() {
	fmt.Println("Go Go Go!")
}

输出：
Go Go Go!
```
# Channel
- Channel 是 goroutine 沟通的桥梁，大都是阻塞同步的
- 通过 make 创建，close 关闭
- Channel 是引用类型
- 可以使用 for range 来迭代不断操作 channel
- 可以设置单向或双向通道
- 可以设置缓存大小，在未被填满前不会发生阻塞

```go
func main() {
	// 是go一种特殊的数据类型，有点像Linux系统中的管道/消息队列
	c := make(chan bool)
	go func() {
		fmt.Println("Go Go Go!")
		c <- true
	}()
	// 等待从通道里读取值
	<-c
}
```

```go
func main() {
	c := make(chan bool)
	go func() {
		fmt.Println("Go Go Go!")
		c <- true
		close(c)
	}()
	// 可以使用for range来迭代不断操作channel，直到关闭channel
	for v := range c {
		fmt.Println(v)
	}
}
```

```go
func main() {
    // 有缓存的channel，非阻塞的，不会输出
	c := make(chan bool, 1)
	go func() {
		fmt.Println("Go Go Go!")
		<-c
	}()
	c <- true
}

func main() {
    // 无缓存的channel是阻塞的，输出：Go Go Go!
	c := make(chan bool)
	go func() {
		fmt.Println("Go Go Go!")
		<-c
	}()
	c <- true
}
```

```go
// 在多线程情况下，程序不是顺序执行，index=9的执行完不代表所有都执行问
func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())
	c := make(chan bool)
	for i := 0; i < 10; i++ {
		go Go(c, i)
	}
	<-c
}

func Go(c chan bool, index int) {
	a := 1
	for i := 0; i < 100000000; i++ {
		a += i
	}
	fmt.Println(index, a)

	if index == 9 {
		c <- true
	}
}

输出（每次都会不同）：
0 4999999950000001
3 4999999950000001
1 4999999950000001
2 4999999950000001
6 4999999950000001
7 4999999950000001
9 4999999950000001
```

```go
// 解决上面多线程不能判断都执行完，通过创建带缓存区的channel来解决
func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())
	c := make(chan bool, 10)
	for i := 0; i < 10; i++ {
		go Go(c, i)
	}
	for i := 0; i < 10; i++ {
		<-c
	}
}

func Go(c chan bool, index int) {
	a := 1
	for i := 0; i < 100000000; i++ {
		a += i
	}
	fmt.Println(index, a)

	c <- true
}

输出（每次都能输出10个）：
4 4999999950000001
0 4999999950000001
6 4999999950000001
9 4999999950000001
1 4999999950000001
7 4999999950000001
8 4999999950000001
5 4999999950000001
2 4999999950000001
3 4999999950000001
```

```
// 多线程协作通过同步包来实现
func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())
	wg := sync.WaitGroup{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go Go(&wg, i)
	}
	wg.Wait()
}

func Go(wg *sync.WaitGroup, index int) {
	a := 1
	for i := 0; i < 100000000; i++ {
		a += i
	}
	fmt.Println(index, a)

	wg.Done()
}
```

# Select
- 可处理一个或多个 channel 的发送与接收
- 同时有多个可用的 channel时按随机顺序处理
- 可用空的 select 来阻塞 main 函数
- 可设置超时

```go
func main() {
	c1, c2 := make(chan int), make(chan string)
	go func() {
		for {
			select {
			case v, ok := <-c1:
				if !ok {
					break
				}
				fmt.Println("c1", v)
			case v, ok := <-c2:
				if !ok {
					break
				}
				fmt.Println("c2", v)
			}
		}
	}()

	c1 <- 1
	c2 <- "hi"
	c1 <- 3
	c2 <- "hello"

	close(c1)
	close(c2)
}

输出：
c1 1
c2 hi
c1 3
c2 hello
```

```go
// c1、c2有一个关闭程序就会退出
// 两个都关闭才退出是不可行的，我们只有对其中一个进行判断
func main() {
	c1, c2 := make(chan int), make(chan string)
	o := make(chan bool)
	go func() {
		for {
			select {
			case v, ok := <-c1:
				if !ok {
					o <- true
					break
				}
				fmt.Println("c1", v)
			case v, ok := <-c2:
				if !ok {
					o <- true
					break
				}
				fmt.Println("c2", v)
			}
		}
	}()

	c1 <- 1
	c2 <- "hi"
	c1 <- 3
	c2 <- "hello"

	close(c1)
	close(c2)

	<-o
}

输出：
c1 1
c2 hi
c1 3
c2 hello
```

```go
// 通过select来作为一个发送者的应用
func main() {
	c := make(chan int)
	go func() {
		for v := range c {
			fmt.Println(v)
		}
	}()

	for {
		select {
		case c <- 0:
		case c <- 1:
		}
	}
}

输出：
0
1
1
1
0
1
1
1
0
0
0
1
.
.
.
```

```go
// 可设置Select超时
func main() {
	c := make(chan bool)
	select {
	case v := <-c:
		fmt.Println(v)
	case <-time.After(3 * time.Second):
		fmt.Println("Timeout")
	}
}

输出：
Timeout
```


# 思考问题
- 创建一个 goroutine，与主线程按顺序相互发送信息若干次并打印

```
var c chan string

func Pingpong() {
	i := 0
	for {
		fmt.Println(<-c)
		c <- fmt.Sprintf("From Pingpong: Hi, #%d", i)
		i++
	}
}

func main() {
	c = make(chan string)
	go Pingpong()
	for i := 0; i < 10; i++ {
		c <- fmt.Sprintf("From main: Hello, #%d", i)
		fmt.Println(<-c)
	}
}

输出：
From main: Hello, #0
From Pingpong: Hi, #0
From main: Hello, #1
From Pingpong: Hi, #1
From main: Hello, #2
From Pingpong: Hi, #2
From main: Hello, #3
From Pingpong: Hi, #3
From main: Hello, #4
From Pingpong: Hi, #4
From main: Hello, #5
From Pingpong: Hi, #5
From main: Hello, #6
From Pingpong: Hi, #6
From main: Hello, #7
From Pingpong: Hi, #7
From main: Hello, #8
From Pingpong: Hi, #8
From main: Hello, #9
From Pingpong: Hi, #9
```
