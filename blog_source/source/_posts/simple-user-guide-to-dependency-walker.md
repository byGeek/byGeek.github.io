---
title: Dependency walker的简单介绍
date: 2018-07-05 13:37:55
tags:
- c++
categories:
- tutorial
keywords:
description:

---



在c++中可以使用LoadLibrary来动态加载dll，最近遇到了一个跟发布有关的问题。在自己的电脑上运行没问题，但是在客户机上却LoadLibrary失败，返回126的错误。使用Dependency walker分析之后，发现是要动态加载的类库依赖另外一个类库，客户机找不到该类库，所以失败。下面简单介绍一下Dependency walker的使用方法。



Dependency walker是一个可以用来查看windows上可执行文件依赖库的工具。可以用来分析库的装载相关的错误。

> Dependency Walker is a free utility that scans any 32-bit or 64-bit Windows module (exe, dll, ocx, sys, etc.) and builds a hierarchical tree diagram of all dependent modules. For each module found, it lists all the functions that are exported by that module, and which of those functions are actually being called by other modules. Another view displays the minimum set of required files, along with detailed information about each file including a full path to the file, base address, version numbers, machine type, debug information, and more. 



<!--more-->

Dependency walker主界面如下：

{% asset_img dp.png %}

通过File->open来加载一个module，module可以是dll，exe文件。左侧是加载的module的依赖的dll列表，显示了树状的依赖层级。可以在菜单栏中选择显示绝对路径。



右边上方是在左侧选中的dll中，实际上被父模块调用的函数。在该示例图中，表示fmaud_mn.dll中调用了MSVCR110.DLL中的_onexit, _unlock等等函数。其中左侧的PI列图例介绍如下：

{% asset_img legend1.png %}



右下方的是左侧选中的dll中，export function列表，如果导出的是c++ function，可以右键选择undecoreated c++ function，这样显示的人类可读的没有被编译器处理的函数名。，其中左侧的E列图例介绍如下：

{% asset_img legend2.png %}

我们也可以使用dumpbin命令行工具(linux中使用objdump或readelf)来显示dll中导出的函数。



[Dependency walker下载地址](http://www.dependencywalker.com/)

详细使用指南可以参考下载文件夹内的的帮助文件。

关于可执行文件，目标文件的格式，可以参考《程序员的自我修养---链接，装载与库》一书。

