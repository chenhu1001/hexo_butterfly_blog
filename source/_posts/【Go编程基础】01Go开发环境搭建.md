---
title: 【Go编程基础】01-Go开发环境搭建
date: 2022-05-07 17:56:52
categories: Go
tags: [Go]
---
## 什么是Go？
Go是一门并发支持 、垃圾回收的编译型 系统编程语言，旨在创造一门具有在静态编译语言的 高性能 和动态语言的 高效开发 之间拥有良好平衡点的一门编程语言。

## Go的主要特点有哪些？
- 类型安全和内存安全
- 以非常直观和极低代价的方案实现高并发
- 高效的垃圾回收机制
- 快速编译（同时解决C语言中头文件太多的问题）
- 为多核计算机提供性能提升的方案
- UTF-8编码支持

## Go存在的价值是什么？
Go在谷歌：以软件工程为目的的语言设计

## Go是记事本编程吗？
包括GoLand，VSCode，LiteIDE等众多知名IDE均已支持

## Go目前有多少实际应用和资源？
- 全球最大视频网站 Youtube（谷歌）
- 七牛云储存以及旗下网盘服务（Q盘）
- 爱好者开发的Go论坛及博客
- 已用Go开发服务端的著名企业：谷歌、盛大、七牛、360

## Go发展成熟了吗？
作为一门2009年才正式发布的编程语言，Go是非常年轻的，因此不能称为一门成熟的编程语言，但开发社区每天都在不断更新其核心代码，给我们这些爱好者给予了很大的学习和开发动力。

## Go的爱好者多吗？
以Google Group为主的邮件列表每天都会更新10至20帖，国内的Go爱好者QQ群和论坛每天也在进行大量的讨论，因此可以说目前Go爱好者群体是足够壮大。

## 安装Go语言
- Go源码安装：[参考链接](https://go.dev/dl/)
- Go标准包安装：[下载地址](https://go.dev/dl/)
- 第三方工具安装

## Go环境变量与工作目录
![在这里插入图片描述](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/%E3%80%90Go%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80%E3%80%9101Go%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA_1.png-watermark)

根据约定，GOPATH下需要建立3个目录：
- bin（存放编译后生成的可执行文件）
- pkg（存放编译后生成的包文件）
- src（存放项目源码）

## Go常用命令简介
在命令行或终端输入go即可查看所有支持的命令
- go get：获取远程包（需 提前安装 git或hg）
- go run：直接运行程序
- go build：测试编译，检查是否有编译错误
- go fmt：格式化源码（部分IDE在保存时自动调用）
- go install：编译包文件并编译整个程序
- go test：运行测试文件
- go doc：查看文档

## 程序的整体结构
![在这里插入图片描述](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/%E3%80%90Go%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80%E3%80%9101Go%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA_2.png-watermark)

## Go语言版"Hello world!"

```go
package main

import "fmt"

func main()  {
	fmt.Println("Hello World!")
}
```
运行输出：Hello World!
