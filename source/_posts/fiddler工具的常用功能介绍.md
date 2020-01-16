---
title: Fiddler 工具常用操作
date: 2019-04-15 14:40:22
comments: true
toc: true
categories:
	- 软件测试
---

* 关于 Fiddler 工具，除了抓包之外，还可以进行的一些操作。如：请求过滤、断点调试、接口测试、请求重发。

<!--more-->

### 请求过滤

#### 抓取一个域名

抓取 ~~www.baidu.com~~ 域名的请求

![](/uploads/201904/fiddlerguolv.png)

#### 抓取多个域名 

多个域名的请求用分号隔开即可

![](/uploads/201904/fiddlerguolv2.png)

#### 模糊查询

满足二级域名为 ~~baidu.com~~ 的所有域名都显示出来

![](/uploads/201904/fiddlerguolv3.png)

### 断点调试

#### 设置请求前断点

这个时候可以修改 header 信息头，body 传参等，修改后点击 Run to Completion

![](/uploads/201904/fiddlerduandianbefore.png)

![](/uploads/201904/fiddlerqingqiuqian.png)

#### 设置请求后断点

这个时候可以修改response，返回值等，修改后点击 Run to Completion

![](/uploads/201904/fiddlerduandianafter.png)

### 接口测试

![](/uploads/201904/fiddlerzhuanbao.png)

### 请求重发

#### 仅重发

![](/uploads/201904/fiddlerchongfa.png)

#### 修改参数后重发

![](/uploads/201904/fiddlerchongfaxiugaihou.png)