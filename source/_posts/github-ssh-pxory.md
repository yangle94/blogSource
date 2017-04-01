---
title: Mac上Github的ssh、http的代理
date: 2017-02-25 16:49:42
tags: [Github]
---
因为我们身处天朝外加某堵墙的原因，访问Github实在是不稳定，SSH慢的要死，简直了。现在交给大家利用ShadowScoket进行对Github的加速。
## 购买一个VPN
随便什么VPN都好，我这里用的是TMDVPN，我自己用的也是这个，感觉还不错，也不是很贵，有想要的童鞋请给我发Email：1024920977@qq.com，我买的是50元半年，一个月20G流量，[附带邀请码的注册地址](https://www.tmdvpn.cn/auth/register?code=PlMq479CNDyxOdjmFg)。具体使用ShadowScoket的教程他们官网上都有。

## 设置Git的代理地址

### https、https代理
如各平台的 Shadowsocks 客户端都提供一个本地的 socks5 代理，那么你可以这样设置，让 Git 通过 HTTP 链接 clone 代码时走 socks5 代理,当然，你需要看看你的ShadowScoket的socks5的监听端口是多少（ShadowScoket里头高级设置有显示）。
```bash
//通过 http 链接 clone 代码时走 socks5 代理
git config --global http.proxy "socks5://127.0.0.1:1086"
//通过 https 链接 clone 代码时走 socks5代理
git config --global https.proxy "socks5://127.0.0.1:1086"
```
### ssh代理
上面设置的 HTTP 代理对这种方式 clone 代码是没有影响的，也就是并不会加速，SSH 的代理需要单独设置，其实这个跟 Git 的关系已经不是很大，我们需要改的，是SSH 的配置。在用户目录下建立如下文件 ~/.ssh/config，对 GitHub 的域名做单独的处理
```bash
        # 这里必须是 github.com，因为这个跟我们 clone 代码时的链接有关
    Host github.com
       # 如果用默认端口，这里是 github.com，如果想用443端口，这里就是 ssh.github.com详见 https://help.github.com/articles/using-ssh-over-the-https-port/
       HostName github.com
       User git
       # 如果是 HTTP 代理，把下面这行取消注释，并把 proxyport 改成自己的 http 代理的端口
       # ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=6667
       # 如果是 socks5 代理，则把下面这行取消注释，并把 6666 改成自己 socks5 代理的端口
       # ProxyCommand nc -v -x 127.0.0.1:6666 %h %p
```
整个代理就完成了，以后你要是用http、http和ssh方式访问Github就会走ShadowScoket的socks5的代理，前提是你必须要讲ShadowScoket打开并且链接上vpn服务器，否则，并没有什么卵用。

