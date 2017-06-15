---
title: Mac下hexo建站
date: 2017-05-26 16:46:38
comments: true
tags: [Mac,hexo]
---

刚开始建站的时候说过写一篇建站教程，由于当时建站以后因为工作上的各种事儿给耽误了，以后就再也没工夫写了，今天特地来补上。

---

### 准备材料

1. GitHubPages仓库
2. Node.js
3. homebrew
4. hexo
5. git(由于Mac自带了，就不需要了)

---
<!--more-->
###  github上创建GitHubPages仓库

git 官方参考地址: https://pages.github.com

注意：
1. 创建仓库的时候仓库名一定严格按照 git用户名.github.io 来命名
2. 创建仓库完成之后,再去创建一个其他的仓库，用于保存hexo的整个目录，用来当备份使用，名字随便取。

---

### 安装 homebrew
homebrew是Mac下的软件管理程序，类似于centos的yum。

[homebrew官方网站](http://brew.sh/index_zh-cn.html)

下载安装方法、更换国内镜像源等详情请google，目前我在这里仅仅提供部分命令

```bash
    brew update 更新brew列表
    brew search node.js    查询node.js（可以查看最新版本的node.js，然后brew install node7.js）
```

---

### hexo

#### 安装hexo

[hexo官方网站](https://hexo.io/zh-cn/docs/)

```bash
    npm install -g hexo-cli 安装hexo客户端
```

#### 建站

安装完成hexo后，执行建站命令。

```bash
    hexo init <你要存放的目录>
    cd <你要存放的目录>
    npm install
```

#### 配置

新建完成后，指定文件夹的目录如下：

.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes

打开_config.yml，此网站的大部分配置都在里面。

    参数  描述
    title   网站标题
    subtitle    网站副标题
    description 网站描述
    author  您的名字
    language    网站使用的语言
    timezone    网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。

git账户地址以及分支

    deploy:
      type: git
      repository: git@github.com:yangle94/yangle94.github.io.git
      branch: master

网址：
请把你刚才申请的githubpages的地址放到这里的url,例如我的是https://yangle94.github.io， 就把他放到url后。

    参数  描述  默认值
    url 网址  
    root    网站根目录   
    permalink   文章的 永久链接 格式 :year/:month/:day/:title/
    permalink_defaults  永久链接中各部分的默认值    

切记切记，yml文件有特殊的配置，类似于键值对的 :后一定要有个空格，不然会报错！！！

---

---

### hexo命令简介

```bash
hexo s  开启本地访问服务器
hexo d  上传文件
hexo g  生成静态文件
hexo clean 清理本地静态文件
```

---

### 备份
在你刚才创建的目录下，使用git对源码进行备份，防止丢失。

在github上创建一个仓库专门用来存储此处的源代码，具体操作在github上创建仓库的时候会有提示，再此不进行教程了。

---

### 希望大家看完以后早日拥有自己的博客😍
