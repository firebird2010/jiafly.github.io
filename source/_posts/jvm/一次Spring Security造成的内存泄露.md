---
title: 一次Spring Security造成的内存泄露
date: 2019-06-28 10:56:00
tags:
    - JVM
    - 内存泄露
    - 实战
categories:
    - JVM
notshow: true
---
# 1.前言
前段时间项目采用Spring Security作为权限框架应用到现有项目中，一切就那么正常的上线了，然而在上线一段时间后，服务监控就在报警，JVM内存泄露了。如是就是接下来的一段异常处理之旅。
<!-- more -->
# 2.内存泄露解决流程
## 2.1 生成线上最新的堆dump文件
使用命令jmap生成线上java程序的dump文件，我们都知道在这个文件中可以查看堆内对象示例的统计信息、查看ClassLoader的信息以及finalizer队列。

## 2.2 使用MAT(Memory Analyzer Tool)打开生成的dump文件
打开后入图所示:
![MAT](/image/jvm/mat-1.jpeg)

## 2.3 MAT分析
在Histogram中会列出每个类的实例，可通过分析查出是由于Spring Security造成的问题。


# 3.解决
出现内存溢出的原因是因为Spring Security在用户登录的时候默认创建了session，但是用户在登录的时候又不传session，这就导致每个用户每次进入都会重新创建session，session的有效时间是30分钟，这就导致了内存泄露。最后的解决方案是在Security的配置中增加一句:
```java
.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
```
就可以了。

# 4.总结
这个问题其实是归结于自己对于Spring Security不熟悉的原因导致的，解决只需一行代码，但是在问题的发现过程还是值得自己思考的。以及在解决问题时候遇到的一些问题和自己解决这些问题时候的一些经验都是有利于自己的提升，毕竟是实际遇到的问题，自己的理解方面也会大有不同。


>天将降大任于斯人也，必先苦其心志，劳其筋骨，饿其体肤，空乏其身，行拂乱其所为，所以动心忍性，曾益其所不能。




