---
title: Docker安装常用镜像
date: 2019-05-15 20:31:16
tags:
    - docker
categories: 
    - docker
---
# 1.前言
我们在平时的开发中可能会用到mysql，redis 还有mongodb等用于存储数据，但是有时候我们使用测试环境可能不太方便(例如:在家不能连接公司数据库)，这时候我们可以在本地或者自己的云服务器上就有了发挥的作用了，但是如果我们在本地安装这些的话就很占我们电脑的内存了，这时候docker就登场了。
<!-- more -->
# 2.安装docker
## 2.1 在linux上安装docker
在linux上安装docker其实是比较简单的，只需要在命令行中敲入即可，这里我以为Ubuntu系统为例:
```bash
$ sudo apt-get -y install docker.io
```
就这样docker就安装完了，我们可以输入docker，查看docker可以使用的命令。显示如下图，说明已经安装成功。
```bash
$ docker
```
![docker](/image/docker/docker-util.jpg)

## 2.2 在macOS上安装docker
### 2.2.1 macOS 我们可以使用 Homebrew 来安装 Docker
```bash
$ brew cask install docker

==> Creating Caskroom at /usr/local/Caskroom
==> We'll set permissions properly so we won't need sudo in the future
Password:          # 输入 macOS 密码
==> Satisfying dependencies
==> Downloading https://download.docker.com/mac/stable/21090/Docker.dmg
######################################################################## 100.0%
==> Verifying checksum for Cask docker
==> Installing Cask docker
==> Moving App 'Docker.app' to '/Applications/Docker.app'.
&#x1f37a;  docker was successfully installed!
```
### 2.2.2 手动下载安装
点击以下链接下载[dmg下载链接](https://download.docker.com/mac/stable/Docker.dmg) 如同 macOS 其它软件一样，安装也非常简单，双击下载的 .dmg 文件，然后将鲸鱼图标拖拽到Application 文件夹即可。

## 2.3 在Windows上安装docker
[windows下载链接](https://www.docker.com/get-docker) 和windows其他程序安装方式相同。

# 3.基于docker安装常用镜像
## 3.1 安装mysql
**/data/docker/mysql** 是宿主机目录用来映射mysql的数据
### 3.1.1 安装
```bash
$ docker pull mysql:8.0.16
```
等待下载完成后即安装完成，接下来就是启动镜像了。
### 3.1.2 启动镜像
- **普通启动**
```bash
$ docker run --name mysql -p 3306:3306  -e MYSQL_ROOT_PASSWORD=root -v /data/docker/mysql:/var/lib/mysql -d mysql:8.0.16

```
启动镜像名称为mysql，前面一个端口是映射端口，`root`是数据库密码，**/data/docker/mysql** 是宿主机目录用来保存mysql的数据。
- **设置时区和宿主机相同启动**
```bash
$ docker run --name mysql -p 3306:3306  -e MYSQL_ROOT_PASSWORD=root -v /etc/localtime:/etc/localtime  -v /data/docker/mysql:/var/lib/mysql -d mysql:8.0.16

```
启动镜像名称为mysql，前面一个端口是映射端口，`root`是数据库密码， **-v /etc/localtime:/etc/localtime**是设置时区与宿主机一致，**/data/docker/mysql** 是宿主机目录用来保存mysql的数据。

### 3.1.3 启动mysql
```bash
$ docker start mysql
```

## 3.2 安装redis
### 3.2.1 安装redis
```bash
$ docker pull reids:5.0.4
```

### 3.2.2 启动镜像
```bash
$ docker run --name redis -p 6379:6379 -v /data/docker/redis:/data -d redis:5.0.4 redis-server --appendonly yes --requirepass "root"

```
启动镜像名称为redis，前面一个端口是映射端口，**/data/docker/redis**是宿主机数据保存地址 **，appendonly yes**是后台启动，**requirepass "root"**是设置密码为`root`

### 3.2.3 启动ridis
```bash
$ docker start redis
```


## 3.3 安装MongoDB
### 3.3.1 安装MongoDB
```bash
$ docker pull mongo
```

### 3.3.2 启动镜像
- 启动
```bash
$ docker run --name mongo -p 27017:27017 -v /data/docker/mongo:/data/db -d mongo:latest --auth

```
- 新建管理员
```bash
$ docker exec -it mongo mongo admin
>>  db.createUser({ user: 'admin', pwd: 'admin', roles: [ { role: 'userAdminAnyDatabase', db: 'admin' } ]});
```
执行之后看到如下信息则创建成功。
```bash
Successfully added user: {
    "user" : "admin",
    "roles" : [
        {
            "role" : "userAdminAnyDatabase",
            "db" : "admin"
        }
    ]
}
```

### 3.3.3 启动MongoDB
```bash
$ docker start mongo
```

# 3. 结语
我们在使用docker的时候有时候想进入容器可以使用如下命令进入，最后的mysql可以使用容器的名称或者容器的id。
```bash
$ docker exec -it mysql /bin/bash
```


