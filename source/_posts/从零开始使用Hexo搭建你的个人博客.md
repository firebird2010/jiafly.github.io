---
title: 从零开始使用Hexo搭建的个人博客
date: 2016-11-26 19:30:27
tags: hexo
---

## 1.
Hexo介绍以及环境安装
- Hexo介绍
  - Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

- Hexo安装
  - Hexo安装前提必须是电脑已经安装好了Git,Node.js。(这里不做介绍，自行百度) 
  - 安装完以后

```
$ npm install -g hexo-cli
```


## 2.正式开始搭建个人博客(这里以我的博客Jiafly为例)

### 1.github上新建一个新的repository

- 新建的repository必须满足以下特点
  - 名称格式必须为 XXXXX.github.io
  - XXXXX 是github的用户名，不可以随意命名，否则会报错


### 2.本地新建博客文件夹

- 前往要创建博客的文件夹下执行如下命令
```
$ hexo init jiafly
$ cd jiafly
$ npm install
$ npm install hexo-deployer-git --save
```


### 3.修改博客文件夹目录下的_config.yml文件夹的deploy配置

```
deploy:
  type: git
  repo: git@github.com:jiafly/jiafly.github.io.git
  branch: master
```


### 4.修改默认主题(可不修改)

- 进入themes目录,克隆想要应用的主题
```
$ cd jiafly/themes
$ git clone https://github.com/cofess/hexo-theme-pure
```

- 修改jiafly目录下的_config.yml文件
```
theme: pure
```

### 5.预览

```
$ hexo g
$ hexo S
```
执行以上命令后即可以访问 http://localhost:4000 查看页面效果


### 6.部署

```
$ hexo g
$ hexo d
```
执行以上命令之后即会将md文件生成html文件推送到github的master分支上，访问 XXXXX.github.io 即可访问你的博客


### 7.绑定域名

访问XXXXX.github.io怎么也不显得高大尚，这里我们可以绑定自己的域名
- 申请自己的域名，解析一个CNAME类型的域名，指定CNAME为 XXXXX.github.io
- 在你选择的主题或者默认主题的source文件夹新建一个名为CNAME的文件，内容是你解析的域名。我的文件夹是jiafly/themes/pure/source
- 重新部署到github上
```
$ hexo g
$ hexo d
```
 

## 3.保存源文件(存储到github的hexo分支)件

- 克隆github的代码
```
$ git clone git@github.com:XXXXX/XXXXX.github.io.git
```

- 进入 XXXXX.github.io 目录，删除除了.git以为的其他文件

- 将原博客文件夹中的所有可见文件都复制到 XXXXX.github.io目录中，除了其中的隐藏文

- 新建一个.gitignore的文件，内容如下:
```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

- 创建新的分支，用于存储源MD文档
```
$ git checkout -b hexo
```

- 添加所有文件到暂存区
```
$ git add --all
```

执行这一步的时候可能会出现如下错误，这是因为在修改主题时克隆了git代码，我们需要删除themes/pure 文件夹下的 .git文件夹 然后重新执行 git add --all

```
warning: adding embedded git repository: themes/pure
hint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
hint: If you meant to add a submodule, use:
hint:
hint: 	git submodule add <url> themes/pure
hint:
hint: If you added this path by mistake, you can remove it from the
hint: index with:
hint:
hint: 	git rm --cached themes/pure
hint:
hint: See "git help submodule" for more information.
```

- 提交分支
```
$ git commit -m "first commit"
```

- 推送hexo分支的文件到github仓库
```
$ git push --set-upstream origin hexo
```

至此，我们的源文件就保存到了hexo分支上了，以后每次编写博客的时候只需要将最新的文件提交到hexo分支即可。换电脑以后还可以从git上拉取即可。


## 附录(Hexo指令)

