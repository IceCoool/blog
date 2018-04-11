---
title: 初入hexo
date: 2017-05-19 15:12:31
tags: [hexo,搭建博客]
---

在同事那里看到他的博客，里面记录了一些自己工作中遇到的问题及解决问题的方案，以备不时之需，感觉是一件很不错的东西。在他推荐的一篇文章下，自己的博客也在摸索中出世，这里写下搭建博客的过程。

<!-- more -->

搭建博客需要必备一些环境 `nodeJs` `git bash` `hexo`，还需要有一个git账户。

### 安装环境

`git bash` 和 `nodeJS` 的安装在其官网上找到安装包安装就好了，`hexo`需要在 `git bash` 或者`cmd` 中用命令安装。

```
npm i -g hexo
```

### 生成项目

先创建一个文件夹，用来存放自己的博客项目
初始化项目 `hexo init`,之后项目中会多许多文件

```
node_modules 依赖包
public       存放的是生成的页面
scaffolds    命令生成文章等的模板
source       用命令创建的各种文章
themes       主题
_config.yml  配置文件
db.json      source解析得到的
package.json 项目所需模块的配置信息 
```

### 搭桥到github

* 创建一个repository，名称为`yourname.github.io`，其中yourname是你的github名称，必须按照这个规则创建才可以，如下

  ![](/images/create-repo.png)

* 回到gitbash中，配置github账户信息（YourName和YourEmail都替换成你自己的）：

  ![](/images/username.png)

* 创建SSH 

  在 `gitbash` 中输入`ssh-keygen -t rsa -C "youremail@example.com`生成SSH，然后按照下图的方式找到id_rsa.pub文件的内容。

  ![](/images/ssh.png)

  将获得的SSH放到github中：

  ![](/images/settings.png)

  ![](/images/ssh-key.png)

  添加一个 `New SSH Key` ，title随便取，key就填刚刚那一段。

### 配置_config.yml

在最下方位置

```yml
    deploy:
        type: git
        repository: git@github.com:yourname/yourname.github.io.git
        branch: master
```

回到gitbash中，分别执行一下命令：

```
hexo clean
hexo generate
hexo server
```

注：hexo 3.0把服务器独立成个别模块，需要单独安装：`npm i hexo-server`。

打开浏览器输入：`http://localhost:4000`

### 上传到github 

安装 `npm install hexo-deployer-git --save`，这样才能将你写好的文章部署到github服务器上并让别人浏览到
执行命令（建议每次都按照如下步骤部署）
```
hexo clean
hexo generate
hexo deploy
```

稍等几分钟，在浏览器中输入`http://yourgithubname.github.io`就可以看到你的个人博客啦

### 写文章部分

自行查看 [hexo](https://hexo.io/zh-cn/docs/writing.html)官网
`markdown` 语法 [介绍](https://www.jianshu.com/p/b03a8d7b1719)

### 主题设置

我用的是next主题，文档说的很详细，附上地址 [next](http://theme-next.iissnan.com/)


