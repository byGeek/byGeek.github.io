---
title: set up your self-host rss service
date: 2018-11-12 11:02:03
tags:
- rss
- vps
categories:
- tutorial
keywords:
- tiny tiny rss
- rss
description:
---

由于不堪国内的新闻客户端的满屏的广告，我在这个“RSS将死”的时代毅然的转向了RSS。说来惭愧，在Google Reader停服之前，一直没怎么使用过RSS订阅。反而停服之后，开始零星的使用过Feedly的服务，虽也订阅过一些feed，但是却没有将其作为主要的消息获取来源。一方面可能是因为当时国内的一些新闻客户端还没有这么“流氓”，另一方面可能是Feedly服务在国内有时很慢，有时不太稳定。

让我真正开启RSS的大门是接触到了[InoReader](https://www.inoreader.com/)。InoReader的web端和客户端使用体验都很好，并且收录了很多feed，直接搜索名称就可以很方便的找到想要订阅的资源。所以当时作为获取消息的主力使用了很长一段时间。但是一直以来有个痛点，使用Inoreader的手机客户端无法离线缓存，而且在线同步的速度越来越慢，经常在地铁上刷不出来，而且越来越多的feed源不再提供全文输出，只能跳转到源网站进行阅读。

购买了腾讯云的VPS之后，除了搭建了这个博客之外，没有做其他用途。由于博客访问量寥寥，VPS资源基本空置。正好在少数派看到了[自建RSS服务的文章](https://sspai.com/post/41302)，心想， why not？搭建自己的RSS服务，利用好VPS资源，同时定制一些Inoreader付费会员的功能，同时自建服务访问速度也会大大提升。

<!--more-->

以上是自建RSS服务的来由。实际上写下这篇博客的时候，已经使用自建服务半年有余，只是遇到一次宕机问题，我才又重新整理一下的搭建步骤，以便后来解决问题。

**声明**: 以下内容很多都是参考[这篇少数派文章](https://sspai.com/post/41302)。

## 安装

> 用 Docker 方式安装 Tiny Tiny RSS 需要的软件条件是：
>
> - **Docker CE：**用于提供 Docker 运行环境。
> - **PostgreSQL/MySQL 数据库：**用于存储 RSS 服务运行所需的记录。Tiny Tiny RSS 的作者推荐优先选用 PostgreSQL，以获得更好的性能。
> - **Tiny Tiny RSS 本身**

### 安装Docker环境

```bash
$ curl https://get.docker.com/ | sh
```

### 下载PostgreSQL和Tiny Tiny RSS的Docker镜像

```bash

```







## 配置nignx代理

## 安装插件

