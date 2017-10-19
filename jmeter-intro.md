# Jmeter环境配置和压力测试

概要：本文主要写了Jmeter的安装及其运行环境的搭建过程和压力测试简单实例。

## 一、	Jmeter简介

Apache JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到其他测试领域。 它可以用于测试静态和动态资源，例如静态文件、Java 小服务程序、CGI 脚本、Java 对象、数据库、FTP 服务器等等。

JMeter 可以用于对服务器、网络或对象模拟巨大的负载，来自不同压力类别下测试它们的强度和分析整体性能。另外，JMeter能够对应用程序做功能/回归测试，通过创建带有断言的脚本来验证你的程序返回了你期望的结果。为了最大限度的灵活性，JMeter允许使用正则表达式创建断言。

JMeter 可以运行在任何兼容JAVA的系统上MAC OSX, Linux, Windows，建议安装最新JDK。


## 二、	Jmeter环境配置

1）JDK安装

a.下载最新JDK（下载地址：http://www.oracle.com/technetwork/java/javase/downloads/index.html ），然后进行默认安装；

b.Windows下：右键选择“我的电脑”->属性->高级系统设置->环境变量；
系统变量->新建，在变量名中输入：JAVA_HOME，变量中输入：C:\Program Files\Java\jdk1.8.0_131\；

c.编辑Path变量（系统变量中），添加%JAVA_HOME%\bin，然后点击确定保存；

d.系统变量->新建，变量名输入：CLASSPATH，变量值中输入：%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar; 

e.安装成功检测：打开cmd，输入命令：java -version出现下图则JDK安装配置成功
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/1.png)
 
2）Jmeter下载地址：http://jmeter.apache.org/download_jmeter.cgi
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/2.png)
 
3）解压apache-jmeter-3.2.zip到指定文件夹，然后点击Jmeter目录下bin文件夹里的jmeter.bat（Linux和MAC OSX在终端运行jmeter.sh）就可以打开Jmeter GUI页面（官方提示：GUI模式仅用来录制和调试，不要进行压力测试）；

## 三、	Jmeter录制脚本

1）	录制脚本，先打开Jmeter GUI页面，右键测试计划->添加->Threads(Users)->线程组；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/3.png)
 
2）然后右键工作台->添加->非测试元件->HTTP代理服务器；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/4.png)
 
3）HTTP代理服务器配置页面，修改目标控制器为测试计划>线程组（新增的线程组，名字可自定义），然后修改分组为每个组放入一个新的控制器（避免生成的脚本杂乱且无法组织）； 
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/5.png)
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/6.png)
 
4）修改端口号为8888（可修改为本机未使用的端口）并点击启动；

5）打开浏览器并进入代理设置页面，更改代理服务器为下图设置并确定（端口号与上一步一致）；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/7.png)
 
6）在浏览器中输入目标网址并操作，Jmeter会自动录制，点击Jmeter中的“停止”按钮停止录制，可以在Jmeter左侧列表看到录制的脚本；

## 四、	手动创建Jmeter脚本

创建一个简单的压力测试脚本，可以按下列步骤进行操作：

1）先打开Jmeter GUI页面，右键测试计划->添加->Threads(Users)->线程组，配置线程数、Ramp-Up Period（所有线程启动完成所需时间）、循环次数；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/8.png)
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/9.png) 
 
2）右键线程组，添加配置元件->HTTP信息头管理器，并在信息头里添加接口请求头的必要信息（可以打开浏览器，Chrome按F12进入开发者模式，打开网址进行操作，可以在右侧Network->XHR选项下看到接口请求，点击接口可以看到接口详细请求，Header->Request Headers可以查看到所有请求头）；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/10.png)
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/11.png)
 
 
3）右键线程组，添加Sampler->HTTP请求，并添加协议、服务器名或IP、HTTP请求方法、路径、Parameters或Body Data（都可以从浏览器开发者模式下，查看接口详情里获取）；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/12.png)

接口详情：
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/13.png)
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/14.png) 
 
上述接口详情可以生成如下配置：
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/15.png) 
  
4）右键线程组，添加->监听器->察看结果树，可以查看每次请求内容和返回值；
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/16.png) 
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/17.png) 
 
## 五、	Jmeter常用参数、配置分析

1)	线程组：
线程数：每次运行需要生成的线程数
Ramp-Up Period (in seconds)：每次运行所需线程数在多少时间内生成
循环次数
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/18.png) 
 
2)	控制器：
简单控制器：控制断言、HTTP信息头管理、正则表达式提取器等作用范围
循环控制器：控制范围的同时，控制范围内请求的循环次数


3)	HTTP请求：
协议、服务器名称或IP、端口号、方法、路径、Content encoding、Parameters或Body Data
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/19.png)

4)	HTTP信息头管理器：
配置请求头内容
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/20.png) 

5)	CSV Data Set Config：
Filename、File encoding：读取的文件路径和文件名，脚本同级目录只需写文件名（文件内容以英文逗号为分隔，1行为1组数据）
Variable Names：读取文件内容后在Jmeter中使用的变量名称，英文逗号为分隔
Sharing mode：变量作用域，默认是所有线程
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/21.png) 
 
6)	正则表达式提取器：
* 要检查的响应字段：声明正则表达式判断的内容，请求头或响应代码
* 引用名称：其他功能引用的变量名
* 正则表达式：判断内容的规则
* ()：提取括号内的内容
* .：匹配任何字符串
* +：一次或多次
* ?：不要太贪婪，在找到第一个匹配项后停止
* 模板：用`$$`引用，`$1$`表示解析到的第1个值
* 匹配数字：0代表随机取值，1代表全部取值
* ![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/22.png)
 
7)	BeanShell Sampler：执行自定义JAVA代码
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/23.png)
 
8)	响应断言
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/24.png)
 
9)	查看结果树和聚合报告：查看结果树可以查看每次详细的请求和返回，聚合报告总计显示请求响应时间和服务器吞吐量
![](https://raw.githubusercontent.com/catezhao1985/infomations/master/images/25.png) 


 
