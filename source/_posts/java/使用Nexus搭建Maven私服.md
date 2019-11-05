---
title: 使用Nexus搭建Maven私服(Nexus Repository Manager 3.X)
date: 2019-06-27 11:15:00
tags:
    - Maven
    - Nexus
categories:
    - Maven
notshow: true
---
# 1.前言
关于Java项目，一般的公司都会有一个自己的私服来管理各种jar包，其中可能有这么几个原因:
- 1、不能访问外网，只能通过私服来管理jar包和插件；
- 2、网速比较慢，通过公司的私服来获取jar包比较快(内网)；
- 3、内部的的一些jar包放在私服上，比较方便的大家使用。
<!-- more -->
# 2.下载与安装
## 2.1 下载
这里我会选择nexus最新的3.X的版本去进行搭建。
[下载地址](https://help.sonatype.com/repomanager3/download)，
下载可以选择对应的版本有macOS，Windows，unix等不同版本，可以根据自己需要选择。unix版本是支持linux系统的，所以这也是没有linux版本的原因。这里我选择的是unix版本3.17.0-01。

**注意**: Nexus Repository Manager 3.X是一个Java服务器应用程序，安装需要 jdk1.8以上的版本。

## 2.2 安装
这里我是因为从本机下载，nexus需要安装在服务器上，所以我会多出一步将本地下载文件上传至服务器的步骤。
### 2.2.1 上传tar.gz文件至服务器(文件已在服务器可忽略)
以下命令中`xxx.xxx.xxx.xxx`修改为自己服务器的ip地址即可。
```shell
$ scp ./nexus-3.17.0-01-unix.tar.gz root@xxx.xxx.xxx.xxx:/data/nexus/
```

### 2.2.2 解压文件
首先登陆进入服务器进入刚刚上传的目录，然后开始解压文件。
```shell
$ cd /data/nexus;
$ tar zxvf nexus-3.17.0-01-unix.tar.gz
```

### 2.2.3 启动nexus
进入到解压文件的bin目录下，执行`./nexus run`命令即可。
```shell
& cd /data/nexus/nexus-3.17.0-01/bin;
& nohup ./nexus run &
```
注意: 在启动时可能出现内存不足的问题，可根据一下方式解决:
bin目录下有一个nexus.vmoptions文件，编辑它，修改内存的参数适合你的服务器即可。
```shell

```

# 3. 访问Nexus管理后台
Nexus管理后台地址的默认地址是:[http://localhost:8081/](http://localhost:8081/),点击右上角Sign in登录，默认账号和密码为：admin/admin123。

在Repositories仓库管理界面中有多种默认的仓库，也可以添加新的仓库，本实例直接使用默认的仓库：
maven-central，Type为proxy，表示代理仓库。代理仓库用来代理远程仓库（maven-central代理的是超级POM中配置的Maven中央仓库），当在下载组件时，如果代理仓库搜索不到，则会把请求转发到远程仓库从远程仓库下载。从远程仓库下载后会缓存到代理仓库，下次还有该组件的请求则会直接到代理仓库下载，不会再次请求远程仓库。

maven-releases/maven-snapshots，Type为hosted，表示为宿主仓库。宿主仓库主要用来部署团队内部使用的内部组件，默认的maven-releases和maven-snapshots分别用来部署团队内部的发布版本组件和快照版本组件。

# 4. 配置使用
## 4.1 配置settings.xml
```xml
<settings>
   <!-- 配置镜像，此处拦截所有远程仓库的请求到代理仓库-->
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://localhost:8081/repository/maven-central/</url>
    </mirror>
  </mirrors>
  <!-- 配置远程库和远程插件库-->
  <profiles>
    <profile>
      <id>nexus</id>
      <!-- Maven用于填充构建系统本地存储库的远程仓库集合-->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <!-- 类似于repositories元素，指定Maven可以在哪里找到Maven插件的远程仓库位置-->
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <!-- 激活profiles配置 -->
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>
```

## 4.2 创建Maven项目配置pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jiafly</groupId>
  <artifactId>libra-api</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>libra-api</name>
  <url>http://maven.apache.org</url>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```
配置完成后，执行mvn clean，执行mvn clean需要下载maven-clean-plugin插件，通过Browse界面可以看到因为执行mvn clean而下载的maven-clean-plugin.jar：

## 4.3 配置宿主仓库
在setting.xml文件中增加如下配置:
```xml
<servers>
     <server>
        <id>nexus</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```
然后再配置pom.xml文件
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jiafly</groupId>
  <artifactId>libra-api</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>libra-api</name>
  <url>http://maven.apache.org</url>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <distributionManagement>
        <repository>
        <id>nexus</id>
        <name>maven-releases</name>
        <url>http://localhost:8081/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
        <id>nexus</id>
        <name>maven-snapshots</name>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
   </distributionManagement>
</project>
```
执行mvn clean deploy将项目打包并发布到宿主仓库，构建成功后到Browse中maven-snapshots库查看（因为项目版本为0.0.1-SNAPSHOT，是带SNAPSHOT的快照版本）。

**注意**：maven-releases库默认不能重新发布，需要可重新发布则需要修改该仓库配置或者删除已经发布的版本。

**修改配置重新发布**：将maven-releases库中Deployment pollcy改为Allow redeploy既可。

# 5.结语
Maven对于Javaer说，几乎是天天与其打交道，所以很有必要去了解如果搭建自己的Maven私服仓库。平时多培养自己的动手实践能力，才能在用到的时候不慌。

>天将降大任于斯人也，必先苦其心志，劳其筋骨，饿其体肤，空乏其身，行拂乱其所为，所以动心忍性，曾益其所不能。




