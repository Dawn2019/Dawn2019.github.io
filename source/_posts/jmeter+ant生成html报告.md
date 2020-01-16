---
title: Jmeter + Ant + Jenkins + GitLab 持续集成
date: 2019-04-20 16:30:39
comments: true
toc: true
categories:
	- 软件测试
---
   *  为了充分测试系统中的各个模块，减少人工投入成本，较快的定位错误，可以同过自动化的方式在 API 和 UI 层来进行持续集成。
   
   *  这里整理了一份针对 API 的自动化测试步骤。
   <!--more-->
   
---
在用 jmeter 生成时，调整了生成模板,效果图如下:
![](/uploads/201904/xiaoguotuj.png)


### 配置 ant
#### 安装 ant 配置环境变量
#### 添加 jar 包
	将 /apache-jmeter-X.X/extras 目录下的 ant-jmeter-1.1.1.jar 文件拷贝到 ant 安装目录下的 lib 文件夹中

---
### 配置 jmeter
#### 添加 jar 包
   	由于要连接数据库使用，执行前记得在测试计划中选中mysql的jar包(cloud-mysql-connector-java-5.1.7-bin.jar)
也可以把jar包直接放到jmeter中apache-jmeter-X.X\lib\目录下
#### 添加配置文件
   	在 /apache-jmeter-X.X/extras 目录下添加 jmeter.results.shanhe.me.xsl 文件
#### 修改配置文件
	进入/apache-jmeter-3.0/bin/jmeter.properties下：
找到
```
jmeter.save.saveservice.output_format=csv
```
取消注释,改为
```	
jmeter.save.saveservice.output_format=xml 
```

	同时调整改文件中的其他行配置如下：
```
jmeter.save.saveservice.data_type=true
jmeter.save.saveservice.label=true
jmeter.save.saveservice.response_code=true
# response_data is not currently supported for CSV output
jmeter.save.saveservice.response_data=true
# Save ResponseData for failed samples
jmeter.save.saveservice.response_data.on_error=true
jmeter.save.saveservice.response_message=true
jmeter.save.saveservice.successful=true
jmeter.save.saveservice.thread_name=true
jmeter.save.saveservice.time=true
jmeter.save.saveservice.subresults=true
jmeter.save.saveservice.assertions=true
jmeter.save.saveservice.latency=true
jmeter.save.saveservice.connect_time=true
jmeter.save.saveservice.samplerData=true
jmeter.save.saveservice.responseHeaders=true
jmeter.save.saveservice.requestHeaders=true
jmeter.save.saveservice.encoding=true
jmeter.save.saveservice.bytes=true
# Only available with HttpClient4
jmeter.save.saveservice.sent_bytes=true
jmeter.save.saveservice.url=true
jmeter.save.saveservice.filename=true
jmeter.save.saveservice.hostname=true
jmeter.save.saveservice.thread_counts=true
jmeter.save.saveservice.sample_count=true
jmeter.save.saveservice.idle_time=true
```

### 添加存放测试报告的report文件夹
#### 方案一
可以将 report 文件夹放到 jmeter 工具中，此时的目录结构如下
![](/uploads/201904/jreport.png)

#### 方案二(本次使用)
若要 Jenkins+git 结合使用，也可以将report文件文件在放置在jmx同级目录中此时的目录结构如下
![](/uploads/201904/greport.png)

### 配置 build.xml 文件

```
<?xml version="1.0" encoding="utf-8"?>

<project name="ant-jmeter-test" default="run" basedir="."> 
  <tstamp> 
    <format property="time" pattern="yyyyMMddhhmm"/> 
  </tstamp>  
  <!-- 声明extras目录路径 -->
  <property name="basedirectory" value="/home/apache-jmeter-3.0/extras"/>  
  <!-- 本地使用的jmeter目录 -->
  <property name="jmeter.home" value="/home/apache-jmeter-3.0"/>  
  <!-- 本地使用的jtl目录,这个地方指向刚才report中的jtl目录 -->
  <property name="jmeter.result.jtl.dir" value="/var/lib/jenkins/workspace/UAT_home_api_test/activity\jtl"/>  
  <!-- 本地使用的jtl目录,这个地方指向刚才report中的html目录 -->
  <property name="jmeter.result.html.dir" value="/var/lib/jenkins/workspace/UAT_home_api_test/activity\html"/>  
  <property name="test" value="Test"/>  
  <property name="ReportName" value="HomeReport"/>
  <!-- 生的jtl文件要存放的目录和jtl的文件名,文件名可以根据时间来命名,把home-api.jtl改为${time}.jtl即可 -->  
  <property name="jmeter.result.jtlName" value="${jmeter.result.jtl.dir}/home-api.jtl"/>
  <!-- 生的html文件要存放的目录和html的文件名,文件名可以根据时间来命名,把home-api.html改为${time}.html即可 -->    
  <property name="jmeter.result.htmlName" value="${jmeter.result.html.dir}/home-api.html"/>  
  <path id="xslt.classpath"> 
    <fileset dir="${jmeter.home}/lib" includes="xalan*.jar"/>  
    <fileset dir="${jmeter.home}/lib" includes="serializer*.jar"/> 
  </path>  
  <target name="run"> 
    <antcall target="test"/>  
    <antcall target="report"/> 
  </target>  
  <target name="test"> 
  	<!-- 因为本次不是使用${time}来生成jtl，html，所以本次执行时需要把历史文件删除，若使用${time}当做文件名，把以下两行删除即可 -->  
    <delete file="${jmeter.result.jtlName}"/>
    <delete file="${jmeter.result.htmlName}"/>
    <taskdef name="jmeter" classname="org.programmerplanet.ant.taskdefs.jmeter.JMeterTask"/>  
    <jmeter jmeterhome="${jmeter.home}" resultlog="${jmeter.result.jtlName}"> 
    <!-- 执行dir路径下的所有.jmx后缀的文件 -->  
      <testplans dir="/var/lib/jenkins/workspace/UAT_home_api_test/activity" includes="*.jmx"/> 
    </jmeter> 
  </target>  
  <target name="report"> 
    <tstamp> 
      <format property="report.datestamp" pattern="yyyy/MM/dd HH:mm"/> 
    </tstamp>  
    <xslt classpathref="xslt.classpath" style="/home/apache-jmeter-3.0/extras/jmeter.results.shanhe.me.xsl" force="true" in="${jmeter.result.jtlName}" out="${jmeter.result.htmlName}"> 
      <param name="dateReport" expression="${report.datestamp}"/> 
    </xslt>
    <copy todir="${jmeter.result.html.dir}"> 
      <fileset dir="${jmeter.home}/extras"> 
        <include name="collapse.png"/>  
        <include name="expand.png"/> 
      </fileset> 
    </copy> 
  </target> 
</project>
```
### 使用 Gitlab+Jenkins 持续构建
#### 将项目推送到 Gitlab 中如图
![](/uploads/201904/xiangmumulu.png)
#### 配置Jenkins
##### 添加 HTML 报告插件
在系统管理》插件管理》可选插件中搜索
```
HTML Publisher plugin
```
安装
![](/uploads/201904/htmlpu.png)
```
Ant plugin
```
安装
![](/uploads/201904/antchajan.png)
##### 添加环境变量
在系统管理》全局工具配置中添加 ant、jdk 安装路径
![](/uploads/201904/antjdkpath.png)

##### 添加自由风格的项目
配置 gitlab 链接
![](/uploads/201904/githubtuoguan.png)

规定每天六点构建
![](/uploads/201904/dingshijob.jpg)

填写要执行的 xml 文件路径

![](/uploads/201904/zhixingdexml.png)
其中的命令行内容如下
```
if [ ! -f "$WORKSPACE/activity/jtl/home-api.jtl" ];then
	echo "home-api.jtl文件不存在"
    exit 1
else
	echo "文件存在"
fi
cd $WORKSPACE/activity/html
grep -w "failureMessage" $WORKSPACE/activity/jtl/home-api.jtl && cp home-api.html home-api-error.html && exit 1 || exit 0
```
此时进行构建即可看到对应的 html 报告
![](/uploads/201904/goujianhtml.png)
![](/uploads/201904/recorhtml.png)

#### 配置 Jenkins 添加邮件通知
##### 添加构建后操作
![](/uploads/201904/goujianhoucaozuo.png)
![](/uploads/201904/goujianhoumu.png)

##### Default Content 配置
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
</head>
 
<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
    offset="0">
    <table width="95%" cellpadding="0" cellspacing="0"
        style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
        <tr>
            <td><br />
            <b><font color="#0B610B">构建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>项目名称：${PROJECT_NAME}</li>
                    <li>构建编号：第${BUILD_NUMBER}次构建</li>
                    <li>触发原因：${CAUSE}</li>
                    <li>最新的html报告：<a href="${PROJECT_URL}Report/home-api.html">${PROJECT_URL}Report/home-api.html</a></li>
                    <li>上次html_error报告：<a href="${PROJECT_URL}Report/home-api-error.html">${PROJECT_URL}Report/home-api-error.html</a></li>
                    <li>构建日志：<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                    <li>构建Url：<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                    <li>工作目录：<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                    <li>项目Url：<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
                </ul>
            </td>
        </tr>
       <tr>
            <td><b><font color="#0B610B">变更集</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        
        <tr>
            <td>${JELLY_SCRIPT,template="html"}<br/>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
       
       
    </table>
</body>
</html>
```
效果如下
![](/uploads/201904/youxiangxiaoguo.jpg)
##### 设置发送邮件规则
![](/uploads/201904/shezhiguize.png)
## 注意事项：
 *  jmeter 高版本兼容低版本，自下向上不兼容。使用 4.0 版本的工具编写的 jmx 文件在 3.0 版本中打开很可能会遇到报错的情况，而 3.0 版本编写的 jmx 文件 4.0 等高版本可以打开。