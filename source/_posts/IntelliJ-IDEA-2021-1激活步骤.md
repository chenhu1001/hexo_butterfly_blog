---
title: IntelliJ IDEA 2021.1激活步骤
date: 2021-12-15 17:25:22
categories: 分享
tags: [分享,IDEA]
---
目前Jetbrains系列软件的激活方式为通过插件进行30天试用期重置的方式，其他方式都已经失效
## 1、关于下载
安装包可以直接到官方网站下载 https://www.jetbrains.com/idea/download/ 下载版本2021.1

## 2、IDE Eval Reset 重置试用期插件
安装方法：
(1)、运行软件，点击试用，进入到软件创建新项目窗口；
(2)、左上角菜单栏点打开软件设置窗口：IntelliJ IDEA/Preferences，然后选择 Plugins，点击设置图标手动添加第三方插件仓库地：https://plugins.zhile.io 
(3)、添加仓库后，搜索IDE Eval Reset 插件进行安装
## 3.插件使用方法
安装插件后，在Help -> Eval Reset 调出，有 2 个按钮和 1 个勾选项：  
按钮：Reload 用来刷新界面上的显示信息。  
按钮：Reset 点击会询问是否重置试用信息并重启 IDE。  
选择 Yes 则执行重置操作并重启 IDE 生效，选择 No 则什么也不做。（此为手动重置方式）  
勾选项：Auto reset before per restart 如果勾选了，则自勾选后每次重启/退出 IDE 时会自动重置试用信息，你无需做额外的事情。（此为自动重置方式，推荐此方法！）
## 4.中文汉化方法
安装中文插件后重启IDE即可：插件中搜索chinese进行安装
