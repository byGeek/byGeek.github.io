---
title: TCP中的三次握手和四次挥手
date: 2018-03-05 16:55:41
tags:
- TCP
categories:
- learning
keywords:
- TCP
description:

---



TCP(Transmission Control Protocol)是一种面向连接的可靠的传输协议。TCP连接的建立和释放过程可由下图表示：

<!--more-->

![TCP_process](TCP_process.jpg)

那么问题来了：

1. 为什么建立连接协议是三次握手，而关闭连接却是四次握手呢？

   这是因为服务端的 LISTEN 状态下的 SOCKET 当收
   到 SYN 报文的建立连接请求后，它可以把 ACK 和 SYN （ ACK 起应答作用，而 SYN 起同步作用）放在一个报文里来发送。TCP是全双工通信，关闭连接时，
   当收到对方的 FIN 报文通知时，它仅仅表示对方没有数据发送给你了；但未必你所有的数据都全部发送给对方了，所以你可以未必会马上会关闭 SOCKET, 也即你可能还需要发送一些数据给对方之后，再发送 FIN 报文给对方来表示你同意现在可以关闭连接了，所以它这里的 ACK 报文
   和 FIN报文多数情况下都是分开发送的。 

2. 为什么 TIME_WAIT 状态还需要等 2MSL 后才能返回到 CLOSED 状态？

   这是因为虽然双方都同意关闭连接了，而且握手的 4 个报文也都协调和发送完毕，按理可以直接回到 CLOSED 状态（就好比从 SYN_SEND 状态
   到 ESTABLISH 状态那样）；但是因为我们必须要假想网络是不可靠的，你无法保证你最后发送的 ACK 报文会一定被对方收到，因此对方处
   于 LAST_ACK 状态下的 SOCKET 可能会因为超时未收到 ACK 报文，而重发 FIN 报文，所以这个 TIME_WAIT 状态的作用
   就是用来重发可能丢失的 ACK 报文。



本文摘录自 [此](http://www.cnblogs.com/kesal/p/3285415.html)

