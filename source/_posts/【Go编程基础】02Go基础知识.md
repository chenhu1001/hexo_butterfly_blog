---
title: 【Go编程基础】02-Go基础知识
date: 2022-05-13 12:44:13
categories: Go
tags: [Go]
---
## Go内置关键字（25个均为小写）
| break        | default           |  func        | interface        | select|
|--|--|--|--|--|
| case          | defer             |   go          |  map              |  struct|
| chan          | else               |   goto      |  package        | switch|
| const         | fallthrough     | if              | range           |   type|
| continue    | for                  | import     | return            |  var |

## Go注释方法
- // 单行注释
- /* */ 多行注释
## 代码组织结构
- Go程序是通过 **<font color="#660066">package</font>** 来组织的（与python类似）
- 只有 **<font color="#660066">package</font>** 名称为  **<font color="#00dddd">main</font>** 的包可以包含 **<font color="#00dddd">main</font>** 函数
- 一个可执行程序  **<font color="#dd0000">有且仅有</font>**  一个 main 包

- 通过 **<font color="#660066">import</font>** 关键字来导入其它非 **<font color="#00dddd">main</font>**  包
- 通过 **<font color="#660066">const</font>** 关键字来进行常量的定义
- 通过在函数体外部使用 **<font color="#660066">var</font>** 关键字来进行全局变量的声明与赋值
- 通过 **<font color="#660066">type</font>** 关键字来进行结构( **<font color="#660066">struct</font>** )或接口( **<font color="#660066">interface</font>** )的声明
- 通过 **<font color="#660066">func 关键字来进行函数的声明
## Go导入 package 的格式

```go
import "fmt"
import "os"
import "io"
import "time"
import "strings"
```

```go
import (
	"fmt"
	"os"
	"io"
	"time"
	"strings"
)
```
- 导入包之后，就可以使用格式\<PackageName>.\<FuncName>来对包中的函数进行调用
- 如果导入包之后 **<font color="#dd0000">未调用</font>** 其中的函数或者类型将会报出编译错误：

```
imported and not used: "io"
```
## package别名
- 当使用第三方包时，包名可能会非常接近或者相同，此时就可以使用别名来进行区别和调用。

```go
package main

import (
	io "fmt"
)

func main()  {
	io.Println("Hello World!")
}
```

- **<font color="#dd0000">省略调用不建议使用</font>**，易混淆
- 不可以和别名同时使用

```go
package main

import (
	. "fmt"
)

func main()  {
	// 使用省略调用
	Println("Hello World!")
}
```
## 可见性规则
- Go语言中，使用 **<font color="#660066">大小写</font>** 来决定该常量、变量、类型、接口、结构或函数是否可以被外部包所调用
- 根据约定，<font color="#0000dd">函数名</font> 首字母 **<font color="#00dddd">小写</font>** 即为private，<font color="#0000dd">函数名</font> 首字母 **<font color="#00dddd">大写</font>** 即为public

```go
func getField(v reflect.Value, i int)  {
	val := v.Field(i)
	...
}

func Printf(format string, a ...interface{}) (n int) {
	return Fprintf(os.Stdout, format, a...)
}
```
