---
title: TCP报头的标志位
date: 2018-03-05 17:15:45
tags:
- TCP
categories:
- 网络通信
keywords:
- TCP
- TCP header
- flag
description:

---



[上篇文章]()简单的摘录了TCP建立连接和释放连接的过程。TCP报头中的标志位于操控TCP的状态机。下面简单说说TCP报头中的标志位。

<!--more-->

## TCP 报头 ##

二图胜千言。

![tcp_header1](tcp_header.svg)

![tcp_header1](tcp_header1.png)

对比上述二图，可以看到TCP报文中定义了8个标志位。分别为`SYN,ACK,FIN,PSH,RST,URG,CWR,ECE`，其中最为常用的是前六个标志位。通过对这些标志位置位，可以控制TCP连接的建立和释放。

## 标志位 ##

标志位的功能摘录如下：

> - **SYN**: for SYNchronize; marks packets that are part of the new-connection handshake
> - **ACK**: indicates that the header Acknowledgment field is valid; that is, all but the first packet
> - **FIN**: for FINish; marks packets involved in the connection closing
> - **PSH**: for PuSH; marks “non-full” packets that should be delivered promptly at the far end
> - **RST**: for ReSeT; indicates various error conditions
> - **URG**: for URGent; part of a now-seldom-used mechanism for high-priority data
> - **CWR** and **ECE**: part of the Explicit Congestion Notification mechanism



> URG：此标志表示TCP包的紧急指针域（后面马上就要说到）有效，用来保证TCP连接不被中断，并且督促中间层设备要尽快处理这些数据； 
>
> ACK：此标志表示应答域有效，就是说前面所说的TCP应答号将会包含在TCP数据包中；有两个取值：0和1，为1的时候表示应答域有效，反之为0； 
>
> PSH：这个标志位表示Push操作。所谓Push操作就是指在数据包到达接收端以后，立即传送给应用程序，而不是在缓冲区中排队； 
>
> RST：这个标志表示连接复位请求。用来复位那些产生错误的连接，也被用来拒绝错误和非法的数据包； 
>
> SYN：表示同步序号，用来建立连接。SYN标志位和ACK标志位搭配使用，当连接请求的时候，SYN=1，ACK=0；连接被响应的时候，SYN=1，ACK=1；这个标志的数据包经常被用来进行端口扫描。扫描者发送一个只有SYN的数据包，如果对方主机响应了一个数据包回来，就表明这台主机存在这个端口；但是由于这种扫描方式只是进行TCP三次握手的第一次握手，因此这种扫描的成功表示被扫描的机器不很安全，一台安全的主机将会强制要求一个连接严格的进行TCP的三次握手； 
>
> FIN： 表示发送端已经达到数据末尾，也就是说双方的数据传送完成，没有数据可以传送了，发送FIN标志位的TCP数据包后，连接将被断开。这个标志的数据包也经常被用于进行端口扫描。



## TCP过程 ##

结合下图来理解TCP连接的建立与释放和标志位的关系。

![tcp_process](TCP_process.jpg)

## Wireshark抓包

我们可以利用Wireshark抓包工具来对上述过程进行抓包。Wireshar本身不支持loopback address（即127.0.0.1）进行抓包测试，但是可以下载一个插件实现该功能：[Npcap](https://nmap.org/npcap/)。

1. 在本地建立一个socket 链接，详情可参考[该博文](https://bygeek.github.io/2018/03/05/Socket%E9%80%9A%E4%BF%A1%E6%B5%85%E6%9E%90/#more)。

2. 打开wireshark，选择npcap虚拟网卡，开始抓包，并在filter中过滤TCP端口。

3. 三次握手建立连接。SYN表示开始建立连接，PSH表示该包中有数据。

   ![TCP_SYN](TCP_SYN.png)

   同时在wireshark的中间窗口中可以更清楚的看到标志位的置位情况。

   ![tcp_flags](tcp_flags.png)

4. 四次挥手释放连接。FIN表示开始释放连接。client端发送带FIN标志的数据包，server端收到后给一个ACK确认包。然后server端确认自己也没有数据要发送，也给一个FIN包，最后client端回复ACK包，至此连接被释放。

   ![tcp_fin](tcp_fin.png)



## netstat命令行工具

在windows下，系统提供了一个netstat工具来查看连接的状态。

`netstat -na | find "55554"`

![netstat](netstat.png)



## 参考链接

- [TCP Transport](https://intronetworks.cs.luc.edu/current/html/tcp.html)
- [TCP\IP三次握手连接，四次握手断开分析](http://www.cnblogs.com/kesal/p/3285415.html)

