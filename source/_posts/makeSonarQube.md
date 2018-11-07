---
title: 通过docker创建SonarQube代码质量检测平台
date: 2017-10-27 10:13:08
tags: [Docker、SonarQube]
---

作为一个程序，想要写出更健壮的代码，一个好的代码检测工具是必不可少的。某天突然发现了一个代码检测平台工具SonarQube，想要安装一下，发现有docker版本的，特别来安利一下。

### 1. 安装postgresql

``` bash
docker run --name postgresql -e POSTGRES_USER=sonarqube -e POSTGRES_PASSWORD=sonarqube -d postgres
```

### 2. 安装sonarqube

```bash
docker run --name sq --link postgresql -e SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql:5432/sonarqube -e SONARQUBE_JDBC_USERNAME=sonarqube -e SONARQUBE_JDBC_PASSWORD=sonarqube -p 9000:9000 -d sonarqube
```

<!--more-->

### 3. 平台搭建完成

1. 打开 [localhost:9000](http://localhost:9000/)，进行登录，（初始账户为admin,admin），登录成功后会有教程弹出。

2.汉化：
    ![汉化图片](http://img.ylapl.cn/872419-20170110180805072-1652845285.png)
    ![汉化图片](http://img.ylapl.cn/872419-20170110181014853-90947023.png)
    


### 4 附上链接
[sonarqube docker hub地址](https://hub.docker.com/_/sonarqube/)
[spring4all 教程地址](http://spring4all.com/article/169)
[汉化参考](http://www.cnblogs.com/parryyang/p/6270402.html)


