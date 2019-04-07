---
title: Docker-02-创建mysql容器
date: 2019-03-10 17:45:22
tags: 
    - docker
    - mysql
categories: 
    - docker
---
# 1. 前言
我们在工作中会有需要在本机安装mysql，但是独立去安装mysql会耗费很大的内存以及磁盘空间，如果是在自己的电脑上时间久了可能会使电脑越来越卡。但是如果我们在电脑上安装了docker那就不一样了，或者我们购买了云主机，我们可以在主机上自己安装mysql，但是普通的按住步骤太繁琐，而且一般只能启用一个mysql，这时候Docker就站出来啦。
个人比较推荐使用docker，因为真的是太好用了，好用到爱不释手。嘻嘻~。我们不仅可以使用docker还可以安装很多实用的工具，你可以到dockerHub上去查找你需要的镜像。
<!-- more -->
# 2. 准备工作
## 2.1 安装Docker
无论你是Windows还是Mac还是Linux，现在网上都有很多安装教程，你可以根据步骤进行安装，这里我们就当Docker已经安装完啦。
## 2.2 拉取mysql的镜像(这里使用mysql5.7)
```bash
$ docker pull mysql:5.7
```

## 2.3 启动mysql
当我们从镜像仓库拉完mysql以后，本地或者服务器上就已经有了mysql的镜像，我们可以通过命令去查看
```bash
$ docker images 
```
这个命令就列出了当前主机上已经下载的所有镜像。
## 2.4 在宿主机上创建数据存储文件夹
个人比较推荐使用docker安装镜像之后对于数据要有规律的去保存，这样也方便以后删除镜像的时候能够轻易的找到数据一起删除。所有在这里我所有使用docker安装的镜像都会在/data/docker目录下，docker目录中暗中镜像再进行分类。
例如：这里我要使用docker启动一个mysql的容器 容器名称为mysql001，这样我就创建如下目录
```bash
$ mkdir /data/docker/mysql/mysql001
```
### 2.5 创建容器
```bash
$ docker run --name mysql001 -p 3306:3306  -e MYSQL_ROOT_PASSWORD=root -v /etc/localtime:/etc/localtime  -v /data/docker/mysql/mysql001:/var/lib/mysql -d mysql:5.7 
```

这里我们对上面的命令进行拆解，清楚的了解每一步都是在做什么操作。
- docker run 
这是启动一个容器
- --name mysql001 
启动的容器名称为mysql001，这个名称在后面操作可直接使用名称
- -p 3306:3306
映射端口，前面一个端口是宿主机的端口，后面一个端口是mysql的端口，我们访问数据库是通过访问宿主机去访问，所以使用的是前面一个端口
- -e MYSQL_ROOT_PASSWORD=root
设置mysql的登录密码为root
- -v /etc/localtime:/etc/localtime
这个是这是启动容器的时区和宿主机一致，这个设置比较有用，不然会出现数据库中的时间比当前时间晚8小时
- -v /data/docker/mysql/mysql001:/var/lib/mysql
这个就是用刚刚创建的目录去存储mysql的数据了，我们在mysql中的所有数据都会存储在宿主器前面的目录里
- -d mysql:5.7
-d是开启Daemon模式即保护进程的方式运行。最后的这个是知道启动容器的版本 如果没有的话默认就是latest 和前面pull镜像时一样

### 启动容器
执行完上面的命令后容器并没有启动，我们可以通过执行以下命令去启动容器
```bash
$ docker start mysql001
```
或者将name修改为image_id，image_id可以通过 docker images命令去查看。

## 访问mysql数据库
- 方式一
```bash
$ mysql-cli -h 127.0.0.1 -u root -p root
```
- 方式二
使用mysql客户端如：mysqlworkbatch，navicat等客户端软件。

### 删除容器
删除容器必须要保证容器是stop的可以通过下面的命令查看
- 查看正在运行的容器
```bash
$ docker ps 
```
- 查看所有运行过的容器包括正在运行的容器
```bash
$ docker ps -a
```
根据上面的命令可以查找到容器id，执行命令删除容器
```bash
$ docker rm 容器id
```


