---
title: 操作系统基本概念
date: 2018-05-25 14:46:51
tags:
- operating system
categories:
- learning
keywords:
- operating system
description:

---



作为一名非科班程序员，经常在碰到一些操作系统相关的概念时蒙逼。特别是看前段时间看nginx源码的时候，涉及到一些I/O多路复用的代码。碰到问题，碰到不懂的名词，去网上搜索，也就能了解个大概，一些系统性的东西还是很有必要去系统性的学习。于是购入了一本《操作系统：精髓与设计原理》，打算好好读一遍。至于为什么没有买传说中的龙书(深入理解计算机系统)。。。主要是当时不知道有龙书，买完才发现大多数都推崇龙书。



本文姑且作为学习《操作系统》这本书的学习大纲吧。下面先按着书的章节列一下大纲。以后分别出各个主题的博文。坚持！



<!--more-->

## Operating System lesson 101

- 操作系统概述
  - 什么是操作系统，作用是什么
  - 基本组成
  - 总线周期，指令周期
- 进程
  - 什么是进程
  - 在操作系统中如何表示进程
  - 线程，进程区别
- 并发
  - 互斥(Mutex)
  - 信号量(Semaphone)
  - 管程(Monitor)
  - 管道(Pipes)
  - 自旋锁与互斥锁，busy-waiting与sleep-waiting
  - 原子操作
  - 在c#中同步相关的API
  - 死锁与饥饿
- 内存管理
  - 为什么要分页，分段
  - 实存(main memory/primary memory)
  - 虚拟内存的设计原因等
- I/O管理
  - 同步I/O
  - 异步I/O
  - LINUX下I/O多路复用
- 文件系统




