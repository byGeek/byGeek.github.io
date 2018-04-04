---
title: visual studio:应用程序处于中断模式
date: 2018-04-04 08:28:24
tags:
- visual studio
categories:
- coding
keywords:
description:

---



在使用Visual studio 2017 这一宇宙最强IDE一段时间，发现偶尔碰到提示：`the application is in break mode`。

{% asset_img break_mode.png  break mode %}



<!--more-->

由于这个时候visual studio没有给出具体的stack trace信息，不好debug。我们可以在修改visual studio的exception setting，让exception都thrown出来，便于debug。

{% asset_img open_exception_setting.png open exception setting in vs2017 %}

{% asset_img open_exception_setting_in_vs2013.png open exception setting in vs2013 %}

在exception setting中勾选所有的设定。

{% asset_img exception_setting.png exception setting in vs2017 %}

{% asset_img exception_setting_in_vs2013.png exception_setting in vs2013 %}



设置之后，visual studio会在exception的地方停下，给出异常信息。



