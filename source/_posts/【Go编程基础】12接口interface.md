---
title: "【Go编程基础】12-接口interface"
date: 2022-06-26T21:32:16+08:00
categories: ["Go"]
tags: [Go]
---
# 接口interface
- 接口是一个或多个方法签名的集合
- 只要某个类型拥有该接口的所有方法签名，即算实现该接口，无需显示
- 声明实现了哪个接口，这称为 Structural Typing
- 接口只有方法声明，没有实现，没有数据字段
- 接口可以匿名嵌入其它接口，或嵌入到结构中
- 将对象赋值给接口时，会发生拷贝，而接口内部存储的是指向这个
- 复制品的指针，既无法修改复制品的状态，也无法获取指针
- 只有当接口存储的类型和对象都为nil时，接口才等于nil
- 接口调用不会做receiver的自动转换
- 接口同样支持匿名字段方法
- 接口也可实现类似OOP中的多态
- 空接口可以作为任何类型数据的容器

```go
type USB interface {
	Name() string
	// 嵌入接口
	Connecter
}

type Connecter interface {
	Connect()
}

type PhoneConnecter struct {
	name string
}

func (pc PhoneConnecter) Name() string {
	return pc.name
}

func (pc PhoneConnecter) Connect() {
	fmt.Println("Connected:", pc.name)
}

func main() {
	a := PhoneConnecter{"Phone Connecter"}
	a.Connect()
	Disconnect(a)
	
	/* 也可类型转换
	pc := PhoneConnecter{"Phone Connecter"}
	var a Connecter
	a = Connecter(pc)
	a.Connect()
	输出：Connected: Phone Connecter
	*/
}

func Disconnect(usb USB) {
	if pc,ok := usb.(PhoneConnecter);ok {
		fmt.Println("Disconnected:", pc.name)
		return
	}
	fmt.Println("Unkown device.")
}

输出：
Connected: Phone Connecter
Disconnected: Phone Connecter
```

```go
type USB interface {
	Name() string
	// 嵌入接口
	Connecter
}
type Connecter interface {
	Connect()
}

type TVConnecter struct {
	name string
}

func (tv TVConnecter) Connect() {
	fmt.Println("Connected:", tv.name)
}

func main() {
	tv := TVConnecter{"TVConnecter"}
	var a USB
	// 会报错：cannot convert tv (variable of type TVConnecter) to type USB:
	a = USB(tv)
	a.Connect()
}
```

```go
type USB interface {
	Name() string
	// 嵌入接口
	Connecter
}
type Connecter interface {
	Connect()
}

type PhoneConnecter struct {
	name string
}

func (pc PhoneConnecter) Name() string {
	return pc.name
}

func (pc PhoneConnecter) Connect() {
	fmt.Println("Connected:", pc.name)
}

func main() {
	pc := PhoneConnecter{"Phone Connecter"}
	var a Connecter
	a = Connecter(pc)
	a.Connect()

	// 即使修改pc.name的值，还是输出原来的Phone Connecter，对象赋值给接口时，接口内部存储的是指向复制品的指针
	pc.name = "pc"
	a.Connect()
}

输出：
Connected: Phone Connecter
Connected: Phone Connecter
```

```go
func main() {
    // 只有当接口存储的类型和对象都为nil时，接口才等于nil
	var a interface{}
	fmt.Println(a == nil)

	var p *int = nil
	a = p
	fmt.Println(a == nil)
}

输出：
true
false
```

# 类型断言
- 通过类型断言的ok pattern可以判断接口中的数据类型
- 使用type switch则可针对空接口进行比较全面的类型判断

# 接口转换
- 可以将拥有超集的接口转换为子集的接口

