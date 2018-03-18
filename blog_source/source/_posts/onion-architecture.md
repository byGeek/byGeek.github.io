---
title: onion architecture
date: 2018-02-24 09:30:17
tags:
- architecture
- software
---

传统的三层架构中数据位于最核心的地方，而洋葱模型将一些UI，DB这些最可能经常要变化的东西放在外圈，同时外圈的layer依赖于里圈的东西。

<!--more-->

## 传统的三层架构
一层只能调用下一层，不能跨层调用，比如UI只能调用Business Logic 层.

![traditional_layered_architecture](http://orafj4489.bkt.clouddn.com/traditional_layered_architecture.png)



## 洋葱架构
外圈的层可以调用内圈的层

![onion_architecture](http://orafj4489.bkt.clouddn.com/onion_architecture.png)

## 洋葱架构要点

- The application is built around an independent object model
- Inner layers define interfaces.  Outer layers implement interfaces
- Direction of coupling is toward the center
- All application core code can be compiled and run separate from infrastructure


本文摘录[地址](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/)