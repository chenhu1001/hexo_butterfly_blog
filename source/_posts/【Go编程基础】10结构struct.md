---
title: "【Go编程基础】10-结构struct"
date: 2022-06-26T09:32:17+08:00
categories: ["Go"]
tags: [Go]
---
# 结构struct
- Go 中的struct与C中的struct非常相似，并且Go没有class
- 使用 type <Name> struct{} 定义结构，名称遵循可见性规则
- 支持指向自身的指针类型成员
- 支持匿名结构，可用作成员或定义成员变量
- 匿名结构也可以用于map的值
- 可以使用字面值对结构进行初始化
- 允许直接通过指针来读写结构成员
- 相同类型的成员可进行直接拷贝赋值
- 支持 == 与 !=比较运算符，但不支持 > 或 <
- 支持匿名字段，本质上是定义了以某个类型名为名称的字段
- 嵌入结构作为匿名字段看起来像继承，但不是继承
- 可以使用匿名字段指针

```go
// 结构是一个值类型
type person struct {
	Name string
	Age int
}

func main() {
	a := person{}
	a.Name = "Tom"
	a.Age = 18
	fmt.Println(a)

	A(a)
	fmt.Println(a)

	b := person{
		Name: "Lue",
		Age: 16,
	}
	fmt.Println(b)

	B(&b)
	fmt.Println(b)

	// c就是一个指向person地址的指针
	c := &person{
		Name: "Lili",
		Age: 20,
	}

	// 虽然c是一个指针，但仍然可以使用a.Name进行操作
	c.Name = "Hello"
	fmt.Println(*c)
}

func A(per person)  {
	per.Age = 13
	fmt.Println("A", per)
}

func B(per *person)  {
	per.Age = 13
	fmt.Println("B", *per)
}

输出：
{Tom 18}
A {Tom 13}
{Tom 18}
{Lue 16}
B {Lue 13}
{Lue 13}
{Hello 20}
```

```go
func main() {
    // 匿名结构体
	a := &struct {
		Name string
		Age  int
	}{
		Name: "Tom",
		Age:  18,
	}

	fmt.Println(a)
}


输出：
&{Tom 18}
```

```go
type person struct {
	Name string
	Age  int
}

type person1 struct {
	Name string
	Age  int
}

func main() {
	a := person{
		Name: "Tom",
		Age:  18,
	}

	var b person
	b = a
	fmt.Println(b)

	c1 := person{
		Name: "Tom",
		Age:  18,
	}
	c2 := person{
		Name: "Tom",
		Age:  20,
	}

	// 比较只能在相同类型进行比较
	fmt.Println(c1 == c2)
}

输出：
{Tom 18}
false
```

```go
type human struct {
	Sex int
}

// 结构的嵌入
type teacher struct {
	human
	Name string
	Age  int
	Sex int
}

type student struct {
	human
	Name string
	Age  int
}

func main() {
	a := teacher{
		Name: "Tom",
		Age: 30,
		Sex: 1,
		human: human{
			Sex: 0,
		},
	}
	b := student{
		Name: "Lili",
		Age: 18,
		human: human{
			Sex: 1,
		},
	}
	a.Name = "Tom1"
	a.Age = 40
	a.Sex = 0
	a.human.Sex = 1
	b.Name = "Tom1"
	b.Age = 20
	b.Sex = 0
	fmt.Println(a)
	fmt.Println(b)
}

输出：
{{1} Tom1 40 0}
{{0} Tom1 20}
```

# 思考问题
- 如果匿名字段和外层结构有同名字段，应该如何进行操作？
- 请思考并尝试。

```go
type A struct {
	B
	Name string
}

type B struct {
	Name string
}

func main() {
	a := A{
		Name: "A",
		B: B{
			Name: "B",
		},
	}
	fmt.Println(a.Name)
	fmt.Println(a.B.Name)
}

输出：
A
B
```

如果嵌入结构存在内层接口和外层结构存在同名字段，访问需要带上层级。
