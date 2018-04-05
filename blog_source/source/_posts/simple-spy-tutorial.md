---
title: spy++简单教程
date: 2018-04-05 14:08:34
tags:
- windows
categories:
- tutorial
keywords:
- spy++
- windows message
description: Spy++是一款随着Visual studio自带的工具，可以用来分析windows上的应用程序的windows 消息。Windows上的GUI程序都是靠windows message来响应用户，来与用户交互。
---



Spy++是一款随着Visual studio自带的工具，可以用来分析windows上的应用程序的windows 消息。Windows上的GUI程序都是靠windows message来响应用户，来与用户交互。



Spy++有两个版本，32bit版本(spyxx.exe)和64bit版本(spyxx_amd64.exe)，分别用来spy对应bit的进程。所以如果你在使用spy++的时候发现无法收到消息，可以试着使用另外版本。

<!--more-->

在这里同时推荐一款windows小而美的软件：[**Listary**](http://www.listary.com/)。双击left control两次即可呼出检索框。

{% asset_img listary.png %}

在这里以spyxx 32bit版本为例。启动spyxx.exe。

{% asset_img spy++_startup.png %}

Spy++可以查看window，Process和Thread的状态。以查看Window为例。contrl + F呼出Find window窗口。然后用鼠标拖拽中间的圆形标志到你要spy的windows上，即可在window tree上找到。然后右键message可以看到该窗体的消息。

{% asset_img window_message.png all message%}

默认是会将该window收到的所有的消息都显示在spy++界面中，可以通过filter只显示你想要显示的消息。点击菜单栏中的`Logging Option`按钮。勾选需要监听的window message，比如Mouse message

{% asset_img window_message_option.png  %}

{% asset_img window_message_filter.png mouse message %}

Window message的种类可以参考MSDN：[Message Types](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644927%28v=vs.85%29.aspx#types)。





