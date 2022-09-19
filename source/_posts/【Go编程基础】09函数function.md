---
title: "【Go编程基础】09-函数function"
date: 2022-06-25T12:13:56+08:00
categories: ["Go"]
tags: [Go]
---
## 函数function
- Go 函数 不支持 嵌套、重载和默认参数
- 但支持以下特性：  
  - 无需声明原型、不定长度变参、多返回值、命名返回值参数  
  - 匿名函数、闭包
- 定义函数使用关键字 func，且左大括号不能另起一行
- 函数也可以作为一种类型使用

```go
package main

import "fmt"

func main() {
	fmt.Println(A())
	fmt.Println(B())
}

// 非匿名返回值，a,b,c不需要定义可直接赋值
func A() (a, b, c int) {
	a, b, c = 1, 2, 3
	return
}

// 匿名返回值，return后面需要指定变量名称
func B() (int, int, int) {
	a, b, c := 4, 5, 6
	return a, b, c
}
```

```go
func main() {
	a, b := 1, 2
	A(a, b)
	fmt.Println(a, b)

	s1 := []int{1, 2, 3, 4}
	fmt.Println(s1)
	B(s1)
	fmt.Println(s1)

	c := 7
	fmt.Println(c)
	C(&c)
	fmt.Println(c)
}

// 不定长变参，不定长变参只能作为最后一个参数
func A(a ...int) {
	a[0] = 3
	a[1] = 4
	fmt.Println(a)
}

// 直接传递slice会影响原有变量本身
func B(a []int) {
	a[0] = 5
	a[1] = 6
	a[2] = 7
	a[3] = 8
	fmt.Println(a)
}

// 引用类型传递，会改变原始的值
func C(a *int) {
	*a = 5
	fmt.Println(*a)
}

输出：
[3 4]
1 2
[1 2 3 4]
[5 6 7 8]
[5 6 7 8]
7
5
5
```

```go
func main() {
	// go语言中函数也是类型
	a := A
	a()

	// 匿名函数
	b := func() {
		fmt.Println("Func B")
	}
	b()
}

func A() {
	fmt.Println("Func A")
}

输出：
Func A
Func B
```

```go
func main() {
	// 闭包
	f := closure(9)
	fmt.Println(f(1))
	fmt.Println(f(2))
}

// 3次打印的x的地址，都是同一个x
func closure(x int) func(int) int {
	fmt.Printf("%p\n", &x)
	return func(y int) int {
		fmt.Printf("%p\n", &x)
		return x + y
	}
}

输出：
0xc00000a088
0xc00000a088
10
0xc00000a088
11
```

## defer
- 执行方式类似其它语言中的析构函数，在函数体执行结束后按照调用顺序的相反顺序逐个执行
- 即使函数发生严重错误也会执行
- 支持匿名函数的调用
- 常用于资源清理、文件关闭、解锁以及记录时间等操作
- 通过与匿名函数配合可在return之后修改函数计算结果
- 如果函数体内某个变量作为defer时匿名函数的参数，则在定义defer时即已经获得了拷贝，否则则是引用某个变量的地址
- Go 没有异常机制，但有 panic/recover 模式来处理错误
- Panic 可以在任何地方引发，但recover只有在defer调用的函数中有效

```go
func main() {
	fmt.Println("a")
	defer fmt.Println("b")
	defer fmt.Println("c")
}

输出：
a
c
b
```

```go
func main() {
	for i := 0; i < 3; i++ {
		defer fmt.Println(i)
	}
}

输出：
2
1
0
```

```go
// i一直作为引用的方式进行传递，形成了一个闭包
func main() {
	for i := 0; i < 3; i++ {
		defer func() {
			fmt.Println(i)
		}()
	}
}

输出：
3
3
3
```

```go
func main() {
	A()
	B()
	C()
}

func A() {
	fmt.Println("Func A")
}

func B() {
	// 注意recover必须要放在发生panic之前
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Recover in B")
		}
	}()
	panic("Panic in B")
}

func C() {
	fmt.Println("Func C")
}

输出：
Func A
Recover in B
Func C
```

## 问题思考
运行以下程序并分析输出结果。

```go
func main() {
	var fs = [4]func(){}

	for i := 0; i < 4; i++ {
		defer fmt.Println("defer i = ", i)
		defer func() { fmt.Println("defer_closure i = ", i) }()
		fs[i] = func() {
			fmt.Println("closure i = ", i)
		}
	}

	for _, f := range fs {
		f()
	}
}

输出结果：
closure i =  4
closure i =  4
closure i =  4
closure i =  4
defer_closure i =  4
defer i =  3
defer_closure i =  4
defer i =  2
defer_closure i =  4
defer i =  1
defer_closure i =  4
defer i =  0
```
