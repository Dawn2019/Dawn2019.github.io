---
title: jmx文件执行失败，Jenkins仍提示构建成功
date: 2019-04-23 22:31:29
comments: true
toc: true
categories:
	- 软件测试
---
  *  使用Jenkins定时构建jmeter脚本时，Jenkins的构建只能发现该文件是否执行完成，却无法捕捉执行结果是否正确。如果只想收到接口执行失败的邮件通知该如何调整呢？这里使用shell命令对生成的jtl文件进行判断。
<!--more-->
	
---
  使用ant执行build.xml文件时，会将jmx的执行结果先转为jtl文件，再转为html文件。如果断言结果失败时，jtl文件中会出现"failureMessage"字符串。在Jenkins的shell命令中，若返回的值是1会给出build失败，为0则是表示build成功。
	  以上，在shell命令中，校验生成的jtl文件是否包含"failureMessage"字符串，若有，即返回1，提示构建失败。
  命令如下：
 ```	
  if [$workspace/activity/jtl/home-api.jtl];then
  	echo "home-api.jtl文件不存在"
  	exit 1
  else 
  	echo "文件存在"
  grep -w "failureMessage" $workspace/activity/jtl/home-api.jtl && exit 1 || 0
  ```


  1.排查是否生成了jtl文件
  ```
  if [$workspace/activity/jtl/home-api.jtl];then
  	echo "home-api.jtl文件不存在"
  	exit 1
  else 
  	echo "文件存在"
  ```
  2.排查断言是否报错
  ```
  grep -w "failureMessage" $workspace/activity/jtl/home-api.jtl && exit 1 || 0
  ```