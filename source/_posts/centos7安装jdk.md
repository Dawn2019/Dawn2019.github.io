---
title: CentOS7 安装 JDK
date: 2019-04-12 21:10:43
comments: true
toc: true
categories:
	- 环境搭建
---

* 根据类的数量的不同有三个版本可以选择： J2SE 、 J2EE 、J2ME ，分别对应着微型版、企业版、标准版，这里以 J2SE 为例。

   <!--more-->


## 安装 jdk
### 访问 Oracle 官网,下载安装包
官网下载地址：[JDK](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html);
![](/uploads/jdkgw.jpg)
### 使用 xftp 上传该包时指定目录,解压
进入到 ~~/usr/local/~~ 下(不是必须要放置 ~~/usr/local/~~ 目录或其子目录下，只是一个惯例，方便大家查找)
```
mkdir java
```
进入到 ~~/usr/local/java/~~ 下：
```
cd java
```
通过 xftp 将文件放置该目录，解压：
```
tar -zxvf jdk-8u201-linux-x64.tar.gz
```
解压后查看如下
![](/uploads/jdk.jpg)
### 配置环境变量，重新加载该文件
打开配置文件：
```
vim /etc/profile
```
在查看模式，按G将光标移动至文件最后一行，打开命令行模式,
添加 JAVA_HOME 、 PATH 、 CLASSPATH 三个环境变量：
```
export JAVA_HOME= /usr/local/java/jdk1.8.0_201/
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```
使用内置命令，重新加载该配置文件：
```
source /etc/profile
```
### 使用命令查看安装情况
```
java -version
```
![](/uploads/jdkversion.jpg)
## 执行一个简单的java文件
---
### 创建一个 .java 文件
```
vi test.java
```
输入要打印的字符串：
![](/uploads/duohereshui.jpg)
### 使用 javac 命令编译 .java 源文件：
```
javac test.java
```
生成 .class 字节码文件：
![](/uploads/class.jpg)
### 使用 java 命令执行字节码文件：
```
java test
```
![](/uploads/duohereshui2.jpg)