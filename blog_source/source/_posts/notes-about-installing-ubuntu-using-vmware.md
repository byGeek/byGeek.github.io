---
title: 关于使用VMware安装ubuntu的几个注意点
date: 2018-04-12 19:28:38
tags:
- linux
categories:
- tutorial
keywords:
- vmware
- ubuntu
description:

---



最近打算读读开源项目的源码，正好一年前买的《深入理解Nginx》这本书还一直落灰，就准备读读Nginx的源码吧。在Github下载源码之后，干看代码不好理解，得像个办法Debug啊，于是乎有了用VMware折腾下Ubuntu，学学如何使用GCC和GDB。这就是本文的由来。



VMware傻瓜式的安装教程就不多说了。不过昨天折腾了一晚上竟然没有安装成功，使用的VMware pro 10.0版本，镜像为Ubuntu Desktop 16.04 LTS版本，一直提示`Internal Error`。后来换了VMware pro 12.5版本之后顺利安装。



系统安装好之后，安装VMWare tools。VMWare tools可以提供一系列强化功能，比如全屏化，和Host共享文件等。



在VMware菜单中 `VM -> Install VMWare tools  `。VMware tool文件会挂载到CD-ROM中。将压缩包解压出来，打开Terminal

`sudo ./vmware-install.pl`

执行安装脚本即可。安装好之后重启一下。



### 无法全屏问题

`View -> Autosize -> AutoGuest`勾选之后，Logout一下即可



### 共享文件夹

在VMware中编辑虚拟机配置，在Option->Shared Folder选择宿主中要共享的文件夹。然后即可再Ubuntu的`/mnt/hgfs` 路径下看到共享的文件夹。



### 科学上网问题

有的时候还是需要再虚拟机中共享宿主的科学上网网络的。我们需要为虚拟机配置一个代理。具体参考[此文章](https://www.ctolib.com/topics-114336.html)。





