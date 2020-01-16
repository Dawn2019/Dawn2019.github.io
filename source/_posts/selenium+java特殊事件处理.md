---
title:Selenium + Java 特殊事件处理
date: 2019-04-20 19:20:39
comments: true
toc: true
categories:
	- 软件测试
---

* 在使用 selenium + java 编写自动化用例时，除了 click， sendKeys，clear 基本事件外，还会经常遇到一些阻碍性问题,这里针对 iframe 事件、切换句柄、错误截图等整理了一些使用方法。

	<!--more-->

### 截图
#### 截图并存放到指定文件

```
			File srcFile = ((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
		 	FileUtils.copyFile(srcFile, new File("C:\\ui_png\\error.png"));
```

#### 备份错误截图

由于文件名唯一，再次截图时会进行覆盖。若要做好记录，根据时间生成不同的字符串当做文件名即可

```
			String dataString = getDateFormat();
			File srcFile = ((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
			FileUtils.copyFile(srcFile, new File("C:\\ui_png\\ "+dataString+"s.png"));

		private static String getDateFormat(){
			Date date = new Date();
			SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddhhmmss");
			String dataString = sdf.format(date);
			return dataString;
		}
```

#### 将错误截图的方法进行封装

```
		public static File jpg() throws IOException {
			String dataString = getDateFormat();
			File srcFile = ((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
			FileUtils.copyFile(srcFile, new File("C:\\ui_png\\ "+dataString+"s.png"));
			return srcFile;
		}

		private static String getDateFormat(){
			Date date = new Date();
			SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddhhmmss");
			String dataString = sdf.format(date);
			return dataString;
		}
```

### 句柄事件
#### 跳转到指定句柄

```
		Set<String> winHandels=driver.getWindowHandles();// 得到当前窗口的set集合
        List<String> it = new ArrayList<String>(winHandels); // 将set集合存入list对象
        driver.switchTo().window(it.get(1));// 从下标为0开始，1表示切换到第2个窗口，以此类推
```

#### 将句柄事件封装

```
	public static WebElement handles(int value) {
		if(value != null) {
		Set<String> winHandels=driver.getWindowHandles();
        List<String> it = new ArrayList<String>(winHandels);
        driver.switchTo().window(it.get(value));
		}
        return null;
	}
```

### iframe事件
#### 回到根目录(无iframe时)

```
driver.switchTo().defaultContent();
```

#### 切到第二个iframe中

```
driver.switchTo().frame(1); //从下标为0开始，1表示切换到第二个iframe
```

#### 将iframe事件封装

```
	public static WebElement frame(String value) {
		if(value != null && value.equals("defaultContent")) {
			driver.switchTo().defaultContent();
		}else{
			driver.switchTo().frame(Integer.parseInt(value));
		}
		return null;
	}
```

### 鼠标悬停事件

```
WebElement moveToElement = driver.findElement(By.xpath("/html/body/div/form/div[2]/button/i"));
new Actions(driver).moveToElement(moveToElement).perform();
```

### 使用 js 执行器
#### 添加红色边框

```
WebElement user_name = driver.findElement(By.name("user_name")).sendKeys("dawn");
((JavascriptExecutor) driver).executeScript(
				"$(arguments[0]).css('border','solid 2px red')", user_name);
```
