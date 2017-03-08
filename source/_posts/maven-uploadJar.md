---
title: maven_uploadJar
date: 2017-03-08 11:14:14
tags: [maven]
---

今天因为在服务器端对webp进行格式转换，发现对于webp的jar在maven的仓库里并没有发现。于是只好自己把webp的jar加入了maven私服。将代码进行copy，以备不时之需。
```bash
发布到本地仓库：
mvn install:install-file -DgroupId=[groupId] -DartifactId=[artifactId] -Dversion=[version] -Dpackaging=jar -Dfile=[path to file]
 
发布到Nexus仓库：
mvn deploy:deploy-file -DgroupId==[groupId] -DartifactId=[artifactId] -Dversion=[version] -Dpackaging=jar -Dfile=[path to file] -Durl=[url] -DrepositoryId=[id]
```
