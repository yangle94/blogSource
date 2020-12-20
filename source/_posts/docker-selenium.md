
---
title: docker化selenium grid
date: 2017-2-7 15:08:32
categories: 
tags: [Docker,Selenium Grid]
---
因为工作需要，本人构建了一个docker化selenium grid，特此记录，以备其它有用的人使用。

## 1 使用场景调研
---

1. 完成某个功能的所有操作步骤在同一个页面、session中。

2. 对同一个虚拟机中多个浏览器（多个webdriver、不同用户）之间必须进行浏览器层的cookies隔离，保证相互之间cookies不应相互影响。

3. 对同一个虚拟机中多个浏览器（多个webdriver、不同用户）进行模拟键盘输入时，保证所有键盘操作必须相互隔离，保证对应的模拟键盘输入进入其对应的输入框内。

4. 对所有请求进行调度排序。

5. 设定超时时间，对超过时间限制的请求进行删除，保证资源的释放。

6. 对各个selenium节点有对应功能性监控（比如某个节点是否空闲等）。

7. 各个selenium节点对应的日志查看问题。

<!--more-->

## 2 Docker化Selenium Grid2还存在的问题
---

Windows对docker提供的镜像分别为nanoServer和windowsservercore，但其中并不存在GUI、IE等所需内容，查阅官方资料，官方人员说明目前项目并不支持GUI操作。详见[关于docker产品经理对于Windows GUI的说明](https://blog.docker.com/2016/09/dockerforws2016/#comment-367168)。 目前对于windows镜像需要人为的干预。目前我所采用的方式是为单独建立windows的虚拟机，在虚拟机上对重新部署selenium。
在对含有插件的页面进行访问时，需要提前将插件安装好，也就是说需要提前对docker file进行构建。

## 3 整体构建图
![](http://img2.ylapl.cn/seleniumgrid.jpg)
---

## 4 各个node节点的功能性监控
---

在selenium grid的hub节点中自带了一个监控页面，用于监控每个节点的运行情况、ip端口信息，包括有多少任务在排队中当前总共有多少任务在运行等，此页面访问地址为selenium hub节点的4444端口。
![](http://img2.ylapl.cn/BDB86A2C-4358-49D1-86B8-1F596F88EA4C.png)

## 5 部署步骤
---

部署selenium hub
```shell
docker run -d -p 4444:4444 –name registry.cn-hangzhou.aliyuncs.com/angle/selenium-hub:2.1
```
部署chrome
```shell
docker run -d -P -p 5900:5900 –link selenium-hub:hub –name chrome registry.cn-hangzhou.aliyuncs.com/angle/node-chrome-debug:2.1
```
部署firefox
```shell
docker run -d -P -p 5901:5900 –link selenium-hub:hub –name firefox registry.cn-hangzhou.aliyuncs.com/angle/node-firefox-debug:2.1
```
部署IE
nodeconfig.json
```json
{
“capabilities”:
    [
        {
            “browserName”: “internet explorer”,
            “maxInstances”: 3,
            “seleniumProtocol”: “WebDriver”
        }
    ],
“proxy”: “org.openqa.grid.selenium.proxy.DefaultRemoteProxy”,
“maxSession”: 3,
“port”: 5555,
“register”: true,
“registerCycle”: 5000,
“hub”: “http://localhost:4444“,
“nodeStatusCheckTimeout”: 5000,
“nodePolling”: 5000,
“role”: “node”,
“unregisterIfStillDownAfter”: 60000,
“downPollingLimit”: 2,
“debug”: false,
“servlets” : [],
“withoutServlets”: [],
“custom”: {}
}
```
下载selenium-server-standalone-3.0.1.jar

下载地址：
https://selenium-release.storage.googleapis.com/3.0/selenium-server-standalone-3.0.1.jar

部署命令
```java 
java -jar selenium-server-standalone.jar -role node -nodeConfig nodeconfig.json
```

### 6 已解决的问题

我所用的原始Selenium Grid Docker镜像中有中文乱码问题，需要在docker file中加入字符集解决。
Selenium Grid中存在监控功能并不完善，若想要获取此类信息数据，需要加入自定义servlet（关于此章我会在另一片文章中介绍）

### 7 下载以及中文适配

由于selenium官方的docker镜像对于中文兼容性不是很好，会出现某些中文乱码的问题，其实是因为其包含的乌班图镜像中缺少了部分字符集的原因，为了修复这个问题，我特意制作了修复了此问题的镜像并上传到了阿里云镜像服务器（代码中已经替换）。