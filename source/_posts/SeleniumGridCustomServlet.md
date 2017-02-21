---
title: 自定义Selenium Grid2 Servlet
date: 2017-2-7 15:45:32
categories: 
tags: [Selenium Grid]
---
最近搞selenium grid的时候需要收集每个时刻selenium gird的监控数据，找了半天好不容易在selenium的官网文档的某个角落找到了办法，特此记录。

[官网自定义Sevlet文档](http://www.seleniumhq.org/docs/07_selenium_grid.jsp#customizing-the-grid)

### 自定义servlet步骤

1. 下载selenium-server-standalone-3.0.1.jar，新建一个java工程并将其当作lib包倒入。

2. 工程下新建一个自定义servlet类，继承org.openqa.grid.web.servlet.RegistryBasedServlet或者javax.servlet.http.HttpServlet.类。RegistryBasedServlet类为selenium grid的核心类，如果你想要获得selenium grid的核心数据，请继承该类。

3. 将此项目导出为一个新jar包，并将其与selenium-server-standalone-3.0.1.jar放在同一个目录下。

4. 再此目录下执行

```bash
java -cp xxx.jar:selenium-server-standalone-3.0.1.jar org.openqa.grid.selenium.GridLauncherV3 -servlets org.openqa.grid.web.servlet.custom.CustomServlet,
```
若为hub则再加入“-role hub”等其它所需参数。

### 注意事项

java -cp是classpath的意思，意思是将某些jar加入到classpath中，两个jar用：隔开，此时注意在linux下不支持通配符，所以必须手动将名字补全。-servlets后跟的是自定义servlet的地址，GridLauncherV3为main函数所在位置，在3.0以后的selenium-server-standalone.jar中为GridLauncherV3，2.0为GridLauncher。