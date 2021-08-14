---
title: 解决xbox手柄扳机键回弹慢的问题
date: 2021-08-14 09:58:51
tags:
categories:
- how-to
keywords:
- xbox
- 回弹慢
description:
---

xbox one的手柄已经用了三年了，最近明显感觉到扳机键回弹不灵敏了。在万能的B站搜了一波，对着拆机视频搞起来。

<!--more-->

在B站找到[这个](https://www.bilibili.com/video/BV1x7411s7s3?from=search&seid=18075264222408655437)视频，再次感谢该阿婆主。有点困难的是拆xbox背扣，需要大力出奇迹。上一张正面图

![](xbox-front.JPG)

![](xbox-back.JPG)

不得不说，xbox做工还是很精良的，特别是经过阿婆主的解说，xbox的按键都是通过立体注塑，特别是西瓜键，是分体键，是由两个组件组合起来的，不是简单的雕刻成型的。

搞定了拆机之后，发现Xbox的扳机键就是一个弹簧，并没有其他组件！那扳机键是如何感知射击动作，并转化成电信号的呢？

![](xbox-trigger-key.JPG)

在网上找了一波资料，原来xbox手柄在扳机键使用的霍尔传感器，物理知识已经还给老师的请看下面介绍：

> The **Hall effect** is the production of a [voltage](https://en.wikipedia.org/wiki/Voltage) difference (the **Hall voltage**) across an [electrical conductor](https://en.wikipedia.org/wiki/Electrical_conductor) that is transverse to an [electric current](https://en.wikipedia.org/wiki/Electric_current) in the conductor and to an applied [magnetic field](https://en.wikipedia.org/wiki/Magnetic_field) perpendicular to the current

简单来说就是载流半导体在磁场中会形成电位差。在xbox手柄的扳机键中下面有一块磁铁，在扣动扳机键时，磁铁与霍尔元器件距离变化，带来磁场变化，再带来电压的变化。

搞清楚扳机键的工作原理之后，回弹慢的问题就很简单了：1) 弹簧的问题 2) 扳机键缓冲垫老化导致粘滞。把扳机键拆除再扣动，发现弹簧无问题。把扳机键下方的缓冲垫刮除，问题解决。

![](problem-solve.JPG)

