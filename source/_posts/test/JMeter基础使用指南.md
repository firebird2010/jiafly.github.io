---
title: JMeter基础使用指南
date: 2019-06-21 15:25:00
tags: 
    - JMeter
    - 测试
categories: 
    - 测试
notshow: true
---
# 1.前言
最近开发完项目需要对接口进行压力测试，于是就想到了JMeter,正好借此机会对JMeter的安装使用流程记录一下，既可以加深自己的记忆，也能在之后别人咨询的时候有文章可供其参考。
<!-- more -->
# 2.JMeter安装
## 2.1 下载JMeter
使用工具之前肯定是需要安装的。这里提供[JMeter的官网](http://jmeter.apache.org/)，可以在[这里](http://jmeter.apache.org/download_jmeter.cgi)下载JMeter。


## 2.2 安装JMeter
>JMeter的运行必须依赖Java环境，所以必须确保你的机器已经安装了JDK。在这里就不介绍JDK的安装了，默认你机器的JDK已经安装可以使用。

下载JMeter以后解压到你指定的目录，我的是保存在`/data/utils/apache-jmeter-5.0`这个目录。然后进入到`bin`目录，如果是在linux或者macOS系统上直接运行`jmeter.sh`文件或者`jmeter`，但如果是在Windows系统则直接运行`jmeter.bat`文件。运行以后你可以看到JMeter就启动了。
![JMeter主界面](/image/jmeter/jmeter-main.jpg)

对于英语不好的同学可以切换一下语言，如下图:
![JMeter切换语言](/image/jmeter/jmeter-language.jpg)

这样JMeter就安装完成了，接下来就是使用JMeter的具体功能了。

# 3.JMeter使用
## 3.1 添加线程组(用户)
如图右键点击Test Plan 就会弹出选择框根据选择框，如图操作添加即可。创建需要保存成jmx文件，选择文件要保存的位置即可。
![创建线程组](/image/jmeter/jmeter-create.jpg)

## 3.2 添加测试接口
如下图：添加http接口
![添加测试接口](/image/jmeter/jmeter-http.jpg)

在测试接口中添加接口请求信息 包括请求方式，请求地址，端口，以及参数等。
![测试接口数据](/image/jmeter/jmeter-interface.jpg)

## 3.3 添加监视器
执行完成以后我们需要知道执行结果数据，这里JMeter也提供了监视器供我们去直观的查看接口执行信息。
![添加监听器](/image/jmeter/jmeter-listen.jpg)


## 3.4 启动测试用例
在JMeter最上方有一个绿色的按钮启动即可，我们也可在线程组页面修改线程数，是否一直执行等数据。

# 4.查看执行后的数据
我们在上面添加了监视器，在执行的时候我们可以切换到监视器的视图中查看执行的情况，接口的响应时间，响应状态等情况，以方便我们对程序作出合适的修改，提高效率。

- 结果树:
![结果树](/image/jmeter/jmeter-resulttree.jpg)
- 汇总报告:
![汇总报告](/image/jmeter/jmeter-result.jpg)

# 结语
通过以上操作就完成了一个接口的测试了，上面演示的只是get请求，post请求的参是body，使用的格式是Json。

以上只是初步的使用了JMeter工具，在不会使用JMeter的时候我也只能自己使用脚本语言(Ruby)开启多线程去调用。相比之下，JMeter还是非常简单的，即使没有编程基础一样可以做压测，是不是很棒？





