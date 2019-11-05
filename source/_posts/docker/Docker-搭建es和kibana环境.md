---
title: Docker搭建ES和kibana环境
date: 2019-07-05 10:46:00
tags:
    - es
    - docker
categories:
    - docker
notshow: true
---
# 1.前言
现在elasticsearch是比较火的，很多公司都在用，所以如果说还不知道es可能就会被人鄙视了。所以这里我就下决心来学习es，我比较钟爱于docker所有也就使用了docker来安装es，这里会详细介绍下安装的细节以及需要注意的地方。这里我使用的电脑是MacBook Pro 如果是linux的话其实基本相同，如果是Windows的话，可能就不太一样了，这里我也没有实际操作过，感兴趣的也可以自己去尝试一下。
<!-- more -->
# 2.es安装
## 2.1 docker安装es
要使用es肯定是需要安装的，由于用惯了docker，所以也想在docker上尝试一下，主要是因为我的好多软件都以及选择了docker。docker安装其实是很简单的，至于要一行命令即可。这里我选择的是es的7.2.0版本镜像镜像安装，具体安装命令如下:
```bash
$ docker pull elasticsearch:7.2.0
```
敲完命令以后回车，只需要等带镜像下载完成就可以了。

## 2.2 启动es
安装完成以后当然需要去启动我们的es了，这里启动也是很方便的只需要一行命令即可。如下:
```bash
$ docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:7.2.0
```
这样es就启动好了。我们可以去检查es是否安装完成，可以输入命令：
```bash
$ curl http://localhost:9200
```
或者在浏览器中打开[http://localhost:9200](http://localhost:9200)这个网址，如果能看到以下信息则说明我们的es是已经安装好了的。
```bash
{
  "name" : "530dd7820315",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "7O0fjpBJTkmn_axwmZX0RQ",
  "version" : {
    "number" : "7.2.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "508c38a",
    "build_date" : "2019-06-20T15:54:18.811730Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
如果你是在服务器上安装，想要对外访问还必须打开你服务器的9200端口，然后将localhost换成你服务器的ip地址即可。

## 2.3 修改配置，解决跨域访问问题
首先进入到容器中，然后进入到指定目录修改`elasticsearch.yml`文件。
```bash
$ docker exec -it elasticsearch /bin/bash
$ cd /usr/share/elasticsearch/config/
$ vi elasticsearch.yml
```

在elasticsearch.yml的文件末尾加上:
```bash
http.cors.enabled: true
http.cors.allow-origin: "*"
```

修改配置后重启容器即可。
```bash
$ docker restart elasticsearch
```

## 2.4 安装ik分词器
es自带的分词器对中文分词不是很友好，所以我们下载开源的IK分词器来解决这个问题。首先进入到plugins目录中下载分词器，下载完成后然后解压，再重启es即可。具体步骤如下:
**注意：**elasticsearch的版本和ik分词器的版本需要保持一致，不然在重启的时候会失败。可以在这查看所有版本，选择合适自己版本的右键复制链接地址即可。[点击这里](https://github.com/medcl/elasticsearch-analysis-ik/releases)
```bash
$ cd /usr/share/elasticsearch/plugins/
$ elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.0/elasticsearch-analysis-ik-7.2.0.zip
$ exit
$ docker restart elasticsearch 
```
然后可以在kibana界面的`dev tools`中验证是否安装成功；
```bash
POST test/_analyze
{
  "analyzer": "ik_max_word",
  "text": "你好我是东邪Jiafly"
}
```
不添加`"analyzer": "ik_max_word",`则是每个字分词，可以在下面kibana安装完成以后尝试一下。

# 3.kibana安装
## 3.1 docker安装kibana
同样适用docker安装kibana命令如下:
```bash
$ docker pull kibana:7.2.0
```
等待所有镜像下载完成即可。

## 3.2 启动kibana
安装完成以后需要启动kibana容器，使用`--link`连接到elasticsearch容器，命令如下:
```bash
$ docker run --name kibana --link=elasticsearch:test  -p 5601:5601 -d kibana:7.2.0
$ docker start kibana
```
启动以后可以打开浏览器输入[http://localhost:5601](http://localhost:5601)就可以打开kibana的界面了。

# 4.结语
经过以上步骤就安装好了es和kibana，是不是很简单？这就是docker的好用处之一，也是我比较钟爱docker的原因之一。当然docker远不止这些功能，更多的我们以后慢慢写到，总之肯定是都能用上的。哈哈


>天将降大任于斯人也，必先苦其心志，劳其筋骨，饿其体肤，空乏其身，行拂乱其所为，所以动心忍性，曾益其所不能。




