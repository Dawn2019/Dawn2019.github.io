---
title: jmeter批量生成测试数据
date: 2019-08-09 15:50:21
comments: true
toc: true
categories:
    - 软件测试
---
   *  最近做的一个项目需要生成大批量测试数据进行选课操作。尝试用jmeter进行了处理，效果良好，这里整理一下一些函数的使用以及关键点。

   <!--more-->

   *  脚本要实现的场景相对比较简单：一天内来100个学生选择周一至周末的3至6节课（每天只有一个时间段可选）。选课提交后的传参中包含week_id，值为1至7表示周几。同时还有一个date_id是（下周的）周几对应的日期。

---

## 概括

基本功能的使用：cookie认证、关联请求(正则提取)、断言
数据生成的要求：传参加密、随机生成多个从小到大不相等的随机数、生成号码不重复的11位数手机号并备份、生成下周周一至周日的日期并且date_id要和week_id对应(函数嵌套函数)

### 生成大量学生数据

   先在jmeter中配置好新增一个学生的配置，之后再把传参参数化，调整线程组。
   
### cookie认证

   新增接口时会用到cookie认证，使用jmeter的话cookie又只能在登录的时候才能获取到，也就是说，需配置两个接口：登录+新增学生。其中把登录后的token正则提取出来，放到新增接口的信息头里面，再将传参手机号参数化即可。
   不过新增100个学生时没有必要登录100次，将两个接口放到两个线程组中。
   因为正则提取到的cookie属于局部变量，无法传入到另一个线程组，这个时候需要将提取到的值设置为全局变量：
   ![](/uploads/201908/shezhiquanjubian.jpg)

### 生成号码不重复的11位数手机号

   若要生成11的不重复的手机号，之前的做法有根据时间戳1$\{\_\_time(/1000,)\}，或者$\{\_\_Random(13000000000,14000000000,phone)\}随机数。后来把这种生成手机号的方式放在jenkins持续集成时发现一个缺点：在生成的数据足够多时，会生成库里已经存在的用户或者生成已经生成过的数据。
   这次使用的是计数器方式的生成方式：在指定一个前缀后，对后缀从1000每次加1递增。
   由于每次重启线程时计数器的初始值需要手工改动，这里为了防止手工改动做了一些优化：
   创建一个表，对生成的phone，url等传参进行备份，设置id自增主键
   ![](/uploads/201908/test_data.png)
   手机号后缀从该表中的递增id获取

```
select id from `test_data` order by id desc limit 1;
```

   ![](/uploads/201908/table_id.jpg)
   库中的id一般为3至4位数，用计数器中的number_format进行格式化，确保phone长度固定为11位
   ![](/uploads/201908/get_radomphone.jpg)
   ![](/uploads/201908/table_phone.jpg)
   以上，生成不同并且有序的手机号码的功能就配置完成了

### 传参加密

   学生完成选课首先要进入选课界面，其次提交选课信息。其中进入选课界面的url中包含对phone以及其他字段进行base64加密的参数。对于base64加密有很多解决方式，这里是添加一个后置处理器beanshell进行处理
```
import sun.misc.BASE64Decoder;

String res = new sun.misc.BASE64Encoder().encode("\{\"phone\": $\{phone\},\"channel\": \"SCB\",\"source\": \"web\"\}".getBytes());
String act_url= "http://$\{leadshub_host\}/activity/wjx/success?lv=Lv.$\{lv\}&token="+res;
vars.put("token",res);
vars.put("url",act_url);
```

### 随机生成多个从小到大不相等的随机数

   场景需要满足生成三个从小到大排序并不相等的1至7内的随机数(week_id、week_id2、week_id3...)
   设计方式：若满足0<a<b<c<8,a的值1至5，b的值为a+1至6，c的值为b+1至7
![](/uploads/201908/radom_class.jpg)
   代码如下:
```
//生成三个1至7内的随机数  [1,8)
int a = (int)(Math.random() * (6-1)+1);//随机取[1,5)
int b = (int)(Math.random() * (7-a-1)+a+1);//随机取[a+1,6)
int c = (int)(Math.random() * (8-b-1)+b+1);//随机取[b+1,7)
vars.put("week_id",a.toString());
vars.put("week_id2",b.toString());
vars.put("week_id3",c.toString());
```

### 获取下周的日期
![](/uploads/201908/get_date.jpg)
   代码如下:
```
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

try\{
    Date date =new Date();  //获取当前时间
    SimpleDateFormat sf = new SimpleDateFormat("yyyyMMdd");
    Calendar cal = Calendar.getInstance();
    cal.setTime(date);
    //今天是周几
    int weekday=cal.get(Calendar.DAY_OF_WEEK)-1;

    cal.add(Calendar.DATE,+7-weekday+1);  //下周一
    String date_id_1 = sf.format(cal.getTime());
    cal.add(Calendar.DATE,+1); //下周二
    String date_id_2 = sf.format(cal.getTime());
    cal.add(Calendar.DATE,+1); //下周三
    String date_id_3 = sf.format(cal.getTime());
    cal.add(Calendar.DATE,+1); //下周四
    String date_id_4 = sf.format(cal.getTime());
    cal.add(Calendar.DATE,+1); //下周五
    String date_id_5 = sf.format(cal.getTime());
    cal.add(Calendar.DATE,+1); //下周六
    String date_id_6 = sf.format(cal.getTime());
    cal.add(Calendar.DATE,+1); //下周日
    String date_id_7 = sf.format(cal.getTime());

    vars.put("date_id_1",date_id_1);
    vars.put("date_id_2",date_id_2);
    vars.put("date_id_3",date_id_3);
    vars.put("date_id_4",date_id_4);
    vars.put("date_id_5",date_id_5);
    vars.put("date_id_6",date_id_6);
    vars.put("date_id_7",date_id_7);

\}
catch(Exception e)\{

\}
```

### 函数嵌套函数

   场景要求：请求中的两个参数，一个参数值根据另一个参数值决定，例如，week_id=5(周五)，date_id=20190816(第二周的周五)
   设计方式:获取下周周一到周日的日期、使用自带函数进行嵌套\{\_\_V\}函数,确保随机数1至7的week_id的值决定date_id的值。
   使用jmeter自带函数\_\_V表示如下：$\使用jmeter自带函数\_\_V表示如下：$\{\_\_V(date_id_$\{week_id\})\}、$\{\_\_V(date_id_$\{week_id2\})\}、$\{\_\_V(date_id_$\{week_id3\})\}
![](/uploads/201908/hanshuqiantao.jpg)
