---
title: "【Go编程基础】07-切片slice"
date: 2022-06-05T17:20:41+08:00
categories: ["Go"]
tags: [Go]
---
## 切片Slice

- 其本身并不是数组，它指向底层的数组
- 作为变长数组的替代方案，可以关联底层数组的局部或全部
- 为引用类型
- 可以直接创建或从底层数组获取生成
- 使用len()获取元素个数，cap()获取容量
- 一般使用make()创建
- 如果多个slice指向相同底层数组，其中一个的值改变会影响全部

- make([]T, len, cap)
- 其中cap可以省略，则和len的值相同
- len表示存数的元素个数，cap表示容量

```go
func main() {
	var s1 []int
	fmt.Println(s1)

	a := [10]int{1,2,3,4,5,6,7,8,9,10}
	s2 := a[9]
	fmt.Println(s2)

	s3 := a[5:10]
	fmt.Println(s3)

	// 获取数组某个位置开始到结束
	s4 := a[3:len(a)]
	s5 := a[3:]
	fmt.Println(s4)
	fmt.Println(s5)

	// 获取前5个元素
	s6 := a[:5]
	fmt.Println(s6)
}

输出：
[]
10
[6 7 8 9 10]
[4 5 6 7 8 9 10]
[4 5 6 7 8 9 10]
[1 2 3 4 5]
```

```go
func main() {
	s1 := make([]int, 3, 10)
	fmt.Println(s1)
	fmt.Println(len(s1), cap(s1))
}

输出：
[0 0 0]
3 10
```

## Slice与底层数组的对应关系
![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/%E3%80%90Go%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80%E3%80%9107%E5%88%87%E7%89%87slice_1.png-watermark)

```go
func main() {
	a := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k'}

	// Sclice_a
	sa := a[2:5]
	fmt.Println(string(sa))

	// Sclice_b
	sb := a[3:5]
	fmt.Println(string(sb))
}

输出：
cde
de
```

## Reslice
- Reslice时索引以被slice的切片为准
- 索引不可以超过被slice的切片的容量cap()值
- 索引越界不会导致底层数组的重新分配而是引发错误

```go
func main() {
	a := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k'}

	// Sclice_a
	sa := a[2:5]
	fmt.Println(string(sa))
	fmt.Println(len(sa), cap(sa))

	// Sclice_b
	sb := a[3:5]
	fmt.Println(string(sb))

	// Reslice
	sc := sa[1:3]
	fmt.Println(string(sc))

	sd := sa[3:5]
	fmt.Println(string(sd))
}

输出：
cde
3 9
de
de
fg
```

## Append
- 可以在slice尾部追加元素
- 可以将一个slice追加在另一个slice尾部
- 如果最终长度未超过追加到slice的容量则返回原始slice
- 如果超过追加到的slice的容量则将重新分配数组并拷贝原始数据

```go
func main() {
	s1 := make([]int, 3, 6)
	fmt.Printf("%v %p\n", s1, s1)
	s1 = append(s1, 1,2,3)
	fmt.Printf("%v %p\n", s1, s1)
	s1 = append(s1, 1,2,3)
	fmt.Printf("%v %p\n", s1, s1)
}

输出：
[0 0 0] 0xc00000c390
[0 0 0 1 2 3] 0xc00000c390
[0 0 0 1 2 3 1 2 3] 0xc00004c060
```

```go
func main() {
	a := []int{1, 2, 3, 4, 5}
	s1 := a[2:5]
	s2 := a[1:3]
	fmt.Println(s1, s2)
	s1[0] = 9
	fmt.Println(s1, s2)
}

输出：
[3 4 5] [2 3]
[9 4 5] [2 9]
```

```go
func main() {
	a := []int{1, 2, 3, 4, 5}
	s1 := a[2:5]
	s2 := a[1:3]
	fmt.Println(s1, s2)
	// s2会重新指向新的数组地址，s1值修改不会影响s2的值
	s2 = append(s2, 1, 2, 34, 5, 66)
	s1[0] = 9
	fmt.Println(s1, s2)
}

输出：
[3 4 5] [2 3]
[9 4 5] [2 3 1 2 34 5 66]
```

## Copy
- slice由长的往短的copy，不会改变短的len，也可以拷贝一部分，例如：copy(s2[1:2], s1[2:3])

```go
func main() {
	s1 := []int{1, 2, 3, 4, 5, 6}
	s2 := []int{7, 8, 9}
	copy(s1, s2)
	fmt.Println(s1)

	s3 := []int{1, 2, 3, 4, 5, 6}
	s4 := []int{7, 8, 9}
	copy(s4, s3)
	fmt.Println(s4)
}

输出：
[7 8 9 4 5 6]
[1 2 3]
```

## 思考问题
- 如何将一个slice指向一个完整的底层数组，而不是底层数组的一部分？

```go
func main() {
	a := []int{1, 2, 3, 4, 5, 6}
	s := a[:]
	fmt.Println(s)
}

输出：
[1 2 3 4 5 6]
```
