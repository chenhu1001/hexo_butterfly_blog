---
title: "【Go编程基础】06-数组array"
date: 2022-05-27T22:40:51+08:00
categories: ["Go"]
tags: [Go]
---
## 数组Array
- 定义数组的格式：var <varName> [n]<type>，n>=0
- 数组长度也是类型的一部分，因此具有不同长度的数组为不同类型
- 注意区分指向数组的指针和指针数组
- 数组在Go中为值类型
- 数组之间可以使用==或!=进行比较，但不可以使用<或>
- 可以使用new来创建数组，此方法返回一个指向数组的指针
- Go支持多维数组

```go
// 数组长度也是类型的一部分，因此具有不同长度的数组为不同类型
var a [3]int
var b [2]int
b = a
// 上面的赋值语句会报错：cannot use a (variable of type [3]int) as type [2]int in assignment

// 下面的语句就能正常赋值成功
var a [2]int
var b [2]int
b = a
fmt.Println(b)

输出：[0 0]
```

```go
func main() {
	a := [2]int{1, 2}
	fmt.Println(a)

	b := [20]int{19: 1}
	fmt.Println(b)

	c := [...]int{19: 1}
	fmt.Println(c)

	// 指向数组的指针
	var p *[20]int = &c
	fmt.Println(p)

	// 指针数组
	x, y := 1, 2
	d := [...]*int{&x, &y}
	fmt.Println(d)

	// 数组比较
	e := [2]int{1, 2}
	f := [2]int{1, 2}
	fmt.Println(e == f)

	// 采用new关键字生成数组
	q := new([10]int)
	fmt.Println(q)

	g := [10]int{}
	g[1] = 2
	fmt.Println(g)
	r := new([10]int)
	r[1] = 2
	fmt.Println(r)
}

输出：
[1 2]
[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1]
[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1]
&[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1]
[0xc00000a170 0xc00000a178]
true
&[0 0 0 0 0 0 0 0 0 0]
[0 2 0 0 0 0 0 0 0 0]
&[0 2 0 0 0 0 0 0 0 0]
```

```
func main() {
	a := [2][3]int{
		{1, 1, 1},
		{2, 2, 2},
	}
	fmt.Println(a)

	b := [2][3]int{
		{1:1},
		{2:2},
	}
	fmt.Println(b)

	// 针对二维数组，非顶级不可以使用数量推断
	c := [...][3]int{
		{1:1},
		{2:3},
	}
	fmt.Println(c)
}

输出：
[[1 1 1] [2 2 2]]
[[0 1 0] [0 0 2]]
[[0 1 0] [0 0 3]]
```
## Go语言版冒泡排序

```gherkin
func main() {
	s := [5]int{13, 5, 3, 6, 8}
	len := len(s)
	fmt.Println(s)

	for i := 0; i < len; i++ {
		for j := i + 1; j < len; j++ {
			if s[i] > s[j] {
				temp := s[i]
				s[i] = s[j]
				s[j] = temp
			}
		}
	}

	fmt.Println(s)
}

输出：
[13 5 3 6 8]
[3 5 6 8 13]
```
