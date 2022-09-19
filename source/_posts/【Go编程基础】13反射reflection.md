---
title: "【Go编程基础】13-反射reflection"
date: 2022-06-27T20:26:12+08:00
categories: ["Go"]
tags: [Go]
---
# 反射reflection
- 反射可大大提高程序的灵活性，使得 interface{} 有更大的发挥余地
- 反射使用 TypeOf 和 ValueOf 函数从接口中获取目标对象信息
- 反射会将匿名字段作为独立字段（匿名字段本质）
- 想要利用反射修改对象状态，前提是 interface.data 是 settable，
即 pointer-interface
- 通过反射可以“动态”调用方法

```go
type User struct {
	Id int
	Name string
	Age int
}

func (u User) Hello() {
	fmt.Println("Hello world.")
}

func main() {
	u := User{1, "OK", 18}
	Info(u)
}

func Info(o interface{}) {
	t := reflect.TypeOf(o)
	fmt.Println("Type:", t.Name())
	
	// 反射结构的判断
	if k := t.Kind(); k != reflect.Struct {
		fmt.Println("XX")
		return
	}

	v := reflect.ValueOf(o)
	fmt.Println("Fields")

	for i := 0; i < t.NumField(); i++ {
		f := t.Field(i)
		val := v.Field(i).Interface()
		fmt.Printf("%6s: %v = %v\n", f.Name, f.Type, val)
	}
	
	for i := 0; i < t.NumMethod(); i++ {
		m := t.Method(i)
		fmt.Printf("%6s: %v\n", m.Name, m.Type)
	}
}

输出：
Type: User
Fields
    Id: int = 1
  Name: string = OK
   Age: int = 18
 Hello: func(main.User)
```

```go
type User struct {
	Id   int
	Name string
	Age  int
}

type Manager struct {
	User
	title string
}

func main() {
	// 嵌入结构的反射
	m := Manager{User: User{Id: 1, Name: "tom", Age: 18}, title: "developer"}
	t := reflect.TypeOf(m)

	fmt.Printf("%#v\n", t.FieldByIndex([]int{0, 1}))
}

输出：
reflect.StructField{Name:"Name", PkgPath:"", Type:(*reflect.rtype)(0x905ba0), Tag:"", Offset:0x8, Index:[]int{1}, Anonymous:false}
```

```go
// 利用反射对基本类型的修改
func main() {
	a := 123
	v := reflect.ValueOf(&a)
	v.Elem().SetInt(999)

	fmt.Println(a)
}

输出：
999
```

```go
type User struct {
	Id   int
	Name string
	Age  int
}

func main() {
	u := User{1,"OK", 18}
	Set(&u)
	fmt.Println(u)
}

// 利用反射操作结构
func Set(o interface{}) {
	v := reflect.ValueOf(o)

	if v.Kind() == reflect.Ptr && !v.Elem().CanSet() {
		fmt.Println("Can not set")
		return
	} else {
		v = v.Elem()
	}

	f := v.FieldByName("Name")
	if !f.IsValid() {
		fmt.Println("Bad")
		return
	}

	if f.Kind() == reflect.String {
		f.SetString("Bye")
	}
}

输出：
{1 Bye 18}
```

```go
type User struct {
	Id   int
	Name string
	Age  int
}

func (u User) Hello(name string) {
	fmt.Println("Hello", name, ", my name is", u.Name)
}

func main() {
    // 利用反射实现方法调用
	u := User{1,"OK", 18}
	v := reflect.ValueOf(u)
	mv := v.MethodByName("Hello")

	args := []reflect.Value{reflect.ValueOf("joe")}
	mv.Call(args)
}

输出：
Hello joe , my name is OK
```
