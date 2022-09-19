---
title: "【Go编程基础】11-方法method"
date: 2022-06-26T16:32:22+08:00
categories: ["Go"]
tags: [Go]
---
# 方法method
- Go 中虽没有class，但依旧有method
- 通过显示说明receiver来实现与某个类型的组合
- 只能为同一个包中的类型定义方法
- Receiver 可以是类型的值或者指针
- 不存在方法重载
- 可以使用值或指针来调用方法，编译器会自动完成转换
- 从某种意义上来说，方法是函数的语法糖，因为receiver其实就是
- 方法所接收的第1个参数（Method Value vs. Method Expression）
- 如果外部结构和嵌入结构存在同名方法，则优先调用外部结构的方法
- 类型别名不会拥有底层类型所附带的方法
- 方法可以调用结构中的非公开字段

```go
type A struct {
	Name string
}

type B struct {
	Name string
}

func main() {
	a := A{}
	a.Print()

	b := B{}
	b.Print()
}

func (a A) Print() {
	fmt.Println("A")
}

/* 这样就不行，因为go里面不能重载
func (a A) Print(b int) {
	fmt.Println("A")
}
*/

// 方法的绑定只针对类型和方法同一包名下
func (b B) Print() {
	fmt.Println("B")
}

输出：
A
B
```

```go
type A struct {
	Name string
}

type B struct {
	Name string
}

func main() {
	a := A{Name: "A"}
	// go里会自动识别通过值调用还是指针调用
	a.Print()
	fmt.Println(a.Name)

	b := B{Name: "B"}
	b.Print()
	fmt.Println(b.Name)
}

// Receiver可以是类型的值或者指针（这里是指针传递）
func (a *A) Print() {
	a.Name = "AA"
	fmt.Println("A Print")
}

// Receiver可以是类型的值或者指针（这里是值传递）
func (b B) Print() {
	b.Name = "BB"
	fmt.Println("B Print")
}

输出：
A Print
AA
B Print
B
```

```go
type TZ int

func main() {
	var a TZ
	a.Print()
	(*TZ).Print(&a)
}

func (a *TZ) Print() {
	fmt.Println("TZ")
}

输出：
TZ
TZ
```

```go
type A struct {
	// 私有字段是以package来界定的
	name string
}

func main() {
	a := A{}
	a.Print()
	fmt.Println(a.name)
}

// 方法是可以访问类型中的私有字段
func (a *A) Print() {
	a.name = "123"
	fmt.Println(a)
}
```

# 思考问题
- 根据为结构增加方法的知识，尝试声明一个底层类型为int的类型，
- 并实现调用某个方法就递增100。
- 如：a:=0，调用a.Increase()之后，a从0变成100。

```go
type A int

func main() {
	a := A(0)
	a.Increase(100)
	fmt.Println(a)
}

func (a *A) Increase(num int) {
    // 注意：num需要强制类型转换
	*a += A(num)
}

输出：
100
```
