---
title: JVM系列-自带故障处理工具
date: 2019-06-27 00:05:00
tags:
    - JVM
    - 工具
categories:
    - JVM
notshow: true
---
# 1.前言
Java与C++之间有一堵由内存动态分配和垃圾收集技术所围成的“高墙”，墙外面的人想进去，墙里面的人却想出来。

我们在定位JVM的问题是往往会借助各种工具去定位JVM异常，Sun公司就提供了一些工具，这些工具的功能都十分强大，能帮助我们定位故障问题，解决程序性能等问题。
<!-- more -->
# 2.JVM自带命令行工具
Sum公司提供提供的JVM监控和故障处理工具主要是下表列出的几个，我们在了解了它们的使用之后能为我们带来极大的便利，当然现在很多公司也为我们提供了很多图形化的工具，如：JProfile等，但是我还是觉得了解JDK自带的工具还是很有必要的，毕竟JProfile也是收费的，我在使用JProfile的时候就因为激活码的问题而折腾了好久，最终好像只有JProfile 9版本的才能使用网上的激活码免费使用。

| **名称** | **主要作用** |
| :-----: | :----- |
| jps | JVM process Status Tool，显示系统内所有的HotSpot虚拟机进程 |
| jstat | JVM Statistics Monitoring Tool，用于手机HotSpot虚拟机各方面的运行数据 |
| jinfo | Configuration Info for Java，显示虚拟机配置信息 |
| jmap | Memory Map for Java，生成虚拟机内存转储快照（heapdump文件） |
| jhat | JVM Heap Dump Browser，用户分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果 |
|jstack | Stack Trace for Java，显示虚拟机线程快照 |

## 2.1 **jps**命令
用与获取java进程的LVMID，对于本地虚拟机来说LVMID和PID是相同的。这个命令是基础，许多命令都是依赖这个命令输出的PID才能操作。所以比较重要。
- 命令格式
>jps [options] <pid>
- 命令参数

|**参数** | **作用**|
| :-----: | :----- |
|-q | 只输出pid 省略主类名称|
|-m | 输出JVM进程启动是传递给主类main()函数的参数|
|-l | 输出主类的全名，如果是Jar包，则输出Jar路径|
|-v | 输出JVM进程启动是的参数|

- 执行样例 
jps -l 
```shell
liuyi@liuyideMacBook-Pro:~$ jps -l
568
26985 sun.tools.jps.Jps
26732 org.jetbrains.jps.cmdline.Launcher
26588 org.jetbrains.idea.maven.server.RemoteMavenServer
26733 com.jiafly.libra.LibraPayApplication
```


## 2.2 **jstat**命令
用于见识虚拟机各种运行状态信息的命令行工具。可以显示本地或者远程虚拟机进程中的类装载，内存，垃圾收集，JIT编译等运行数据。但是他没有图形界面，只能纯文本展示。
- 命令格式
>jstat [option vmid [interval[s|ms] [count]]]
- 命令参数
```shell
liuyi@liuyideMacBook-Pro:~$ jstat -options
-class 监视类装载，卸载数量，总空间以及类装载所消耗的时间
-compiler 输出JIT编译器编译过的方法，耗时信息
-gc 监视Java堆，Eden区 S0,S1 元空间等容量信息
-gccapacity 与-gc基本相同，但主要关注Java堆各个区域使用到的最大 最小空间
-gccause 与-gcutil功能一样，但是会输出导致上一次GC产生的原因
-gcmetacapacity 元空间使用到的最大 最小空间
-gcnew 监视新生代GC状况
-gcnewcapacity 新生代使用到的最大 最小空间
-gcold 监视老年代GC状况
-gcoldcapacity 老年代使用到的最大 最小空间
-gcutil 监视内容与-gc基本相同，但输出主要管制已使用空间占总空间占比
-printcompilation 输出已经被JIT编译的方法
```
- 执行样例
jstat -gcutil 26733
```shell
liuyi@liuyideMacBook-Pro:~$ jstat -gcutil 26733
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  15.73  26.29  96.45  94.00     10    0.109     3    0.249    0.358
```
    + 0：幸存1区当前使用比例
    + S1：幸存2区当前使用比例
    + E：伊甸园区使用比例
    + O：老年代使用比例
    + M：元数据区使用比例
    + CCS：压缩使用比例
    + YGC：年轻代垃圾回收次数
    + FGC：老年代垃圾回收次数
    + FGCT：老年代垃圾回收消耗时间
    + GCT：垃圾回收消耗总时间

## 2.3 **jinfo**命令
- 命令格式
>jinfo [option] <pid>

- 命令参数
```shell
liuyi@liuyideMacBook-Pro:~$ jinfo -h
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message
```
- 执行样例
```shell
liuyi@liuyideMacBook-Pro:~$ jinfo -flag ReservedCodeCacheSize 27043
-XX:ReservedCodeCacheSize=251658240
```

## 2.4 **jmap**命令
- 命令格式
>jmap [option] <pid>

- 命令参数
```shell
liuyi@liuyideMacBook-Pro:~$ jmap -h
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```
- 执行样例
```shell
jmap -dump:format=b,file=eclipse.bin 3500
```

## 2.5 **jhat**命令
与jmap搭配使用，用来分析jmap生成的dump文件。内置了HTTP/HTML服务器，可以在浏览器中查看。
- 执行样例
jhat eclipse.bin
执行后屏幕显示“Server is ready.”然后再浏览器输入[http://localhost:7000](http://localhost:7000)就可以看到分析结果。

## 2.6 **jstack**命令
Java堆栈跟踪工具，用于生成虚拟机当前时刻的线程快照。用于定位线程出现长时间停顿的原因，如：线程间死锁，死循环，请求外部资源导致长时间等待等。
- 命令格式
jstack [option] pid
- 命令参数
```shell
liuyi@liuyideMacBook-Pro:~$ jstack -h
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```
- 执行样例
jstack -l 27043

# 3.JVM自带可视化工具
JVM自带的可视化工具在分析JVM问题的时候还是很有用处的，能够直观的从图形界面看出变化。方便我们能快速定位到JVM的问题。
## 3.1 JConsole: Java监视与管理控制台

## 3.2 VisualVM: 多合一故障处理工具



