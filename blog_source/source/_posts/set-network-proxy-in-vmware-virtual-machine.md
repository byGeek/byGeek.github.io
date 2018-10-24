---
title: VMware虚拟机中设置网络代理
date: 2018-10-24 13:53:13
tags:
- shadowsocks
- vmware
categories:
- tutorial
keywords:
description:
---

本文是为vmware中安装的ubuntu设置代理的备忘。

<!--more-->

## 步骤

- 在宿主机中安装好shadowsocks，并允许局域网连接。

  {% asset_img shadowsocks_setting.png %}

- 在vmware中设置虚拟机的网络为NAT模式，这时虚拟机对外来说共享宿主的IP地址，对内于宿主是属于局域网

  {% asset_img vmware_setting.png %}

- 在ubuntu中，system settings->network -> network proxy 中勾选Manual（手动）,地址全部填宿主机IP（局域网网段），设置好代理端口（可在windows下的shadowsocks查看，一般为默认1080）

  {% asset_img ubuntu_setting.png %}

- 访问下不可知的网站试试。