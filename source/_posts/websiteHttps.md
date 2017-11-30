---
title: 使用docker、nginx、nginx-gen、letsencrypt个人主站升级https
date: 2017-11-30 09:25:13
tags: [Docker、Https、Nginx]
---

## 准备工具

1. docker
2. docker-compose
3. nginx 镜像
4. jwilder/docker-gen 镜像
5. jrcs/letsencrypt-nginx-proxy-companion 镜像

## 执行操作

<!--more-->

1. 拷贝一份 .env.sample 并改名为.env

```bash

    cp .env.sample ./.env

```

2. 修改.env文件的内容

```
    NGINX_WEB=nginx-web #nginx的容器的名字
    DOCKER_GEN=nginx-gen #nginx-gen容器的名字
    LETS_ENCRYPT=nginx-letsencrypt #letsencrypt容器的名字

    IP=0.0.0.0 #公网IP名，（可以不用管，0.0.0.0就好）

    # Network name
    NETWORK=webproxy   #docker network的名字

    # NGINX file path
    NGINX_FILES_PATH=/path/to/your/nginx/data #三个容器共享路径
```

3. 执行 run.sh

```bash

    ./run.sh 或者 sh run.sh

```

4. 运行一个测试容器


```bash
    执行一个http协议访问容器:

    docker run -d -e VIRTUAL_HOST=your.domain.com \
              --network=webproxy \
              --name my_app \
              httpd:alpine

```


```bash
    执行一个https协议访问容器:

    docker run -d -e VIRTUAL_HOST=your.domain.com \
              -e LETSENCRYPT_HOST=your.domain.com \
              -e LETSENCRYPT_EMAIL=your.email@your.domain.com \
              -e VIRTUAL_PORT=3000    #设置监听端口
              --network=webproxy \
              --name my_app \
              httpd:alpine

```

##### 此处，请务必把所有需要https代理的容器跟 nginx的三个容器放入同一个network中！

## 原理

1.jwilder/docker-gen镜像用来反向生成nginx配置文件，jrcs/letsencrypt-nginx-proxy-companion镜像用来根据生成的配置文件（域名）申请https证书，会自动进行检测，如果过期，自动申请新的https证书。

2.如果要对新的docker镜像使用https，请务必带上LETSENCRYPT_HOST、 LETSENCRYPT_HOST、LETSENCRYPT_EMAIL这三个属性。
