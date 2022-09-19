---
title: CloudFlare Workers更换（workers.dev）默认域名的解决方案
date: 2022-05-18 16:48:14
categories: 分享
tags: [分享,CDN]
---
最近由于CloudFlare Workers的默认域名（workers.dev）不能访问了。这对于使用CloudFlare Workers服务的应用带来了不便。今天这篇教程就和大家一起解决这个问题。
## 准备材料
- 一个域名（可不用备案）
- 一个目前运行正常的Worker

## 配置步骤
1、登录CloudFlare，点击右上角的添加站点，获取域名托管在CloudFlare上的名字服务器。

![在这里插入图片描述](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/CloudFlare%20Workers%E6%9B%B4%E6%8D%A2%EF%BC%88workers.dev%EF%BC%89%E9%BB%98%E8%AE%A4%E5%9F%9F%E5%90%8D%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88_1.png-watermark)

2、登录域名服务商，不同的服务商配置不一样，我的域名是在 [dynadot](https://www.dynadot.com/) 上购买的，可以使用我的推荐码：pc7M9F6O8Zh6w，在控制台配置域名名字服务器，指向上一步CloudFlare的名字服务器，配置完成后需要等待一段时间才能生效，生效之后会收到CloudFlare发送的生效通知邮件。

![在这里插入图片描述](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/CloudFlare%20Workers%E6%9B%B4%E6%8D%A2%EF%BC%88workers.dev%EF%BC%89%E9%BB%98%E8%AE%A4%E5%9F%9F%E5%90%8D%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88_2.png-watermark)

3、进入刚才添加的站点，添加一条A记录的域名，指向IP可以随意填，我这里填的是8.8.8.8，注意代理状态一定要开启（开启小云朵）。

![在这里插入图片描述](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/CloudFlare%20Workers%E6%9B%B4%E6%8D%A2%EF%BC%88workers.dev%EF%BC%89%E9%BB%98%E8%AE%A4%E5%9F%9F%E5%90%8D%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88_3.png-watermark)

4、点击左侧菜单下的Workers->添加路由->输入自己域名的路由（路由需要提供这种形式：*.xxx.com/*），在下拉框中选择自己的worker服务和环境。

![在这里插入图片描述](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/CloudFlare%20Workers%E6%9B%B4%E6%8D%A2%EF%BC%88workers.dev%EF%BC%89%E9%BB%98%E8%AE%A4%E5%9F%9F%E5%90%8D%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88_4.png-watermark)

5、自此就完成了添加自定义域名。

