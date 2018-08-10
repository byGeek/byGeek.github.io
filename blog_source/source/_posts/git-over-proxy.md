---
title: 配置git使用socks5代理
date: 2018-08-08 13:49:14
tags:
- git
categories:
- tutorial
keywords:
description:
---



最近github网站经常抽风（当然是你懂的的原因），网页上通过配置科学上网之后有所改善。但是在使用git clone时还是经常性失败。在网上找了一波解决方案，发现配置下git代理，很简单，效果立竿见影。



<!--more-->

以下环境为windows 7，console是cmder，直接使用git bash也可以。首先你需要一个代理地址，建议bangwagong自己购买一个云主机，自建服务。本地使用ss做转发。以下`127.0.0.1:1080`即为本地1080监听地址。



## 配置HTTP proxy

```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```

配置之后可以查看下是否成功

```bash
git config --global --get http.https://github.com.proxy
socks5://127.0.0.1:1080
```

或者在编辑器中查看

```bash
git config --global -e
```

{% asset_img gitconfig.png %}



## 配置SSH proxy

上面配置了http的代理之后，我们使用http/https协议来clone repo的时候已经有效了，但是如果想在ssh也走代理的话，还需要以下配置。

在windows users 用户目录下生成config文件，如在`C:\Users\your_user_name\.ssh`目录下，找到config文件，如果没有新建一个，写入如下内容：

```
Host github.com
ProxyCommand connect -H 127.0.0.1:1080 %h 22
```



配置完毕之后，在clone一个repo试试，效果拔群啊。



## 参考链接

- [from github](https://gist.github.com/laispace/666dd7b27e9116faece6)