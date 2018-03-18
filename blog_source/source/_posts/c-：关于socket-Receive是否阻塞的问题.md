---
title: 'c#：关于socket.Receive是否阻塞的问题'
date: 2018-03-05 16:38:40
tags:
- socket
- c#
categories: 
- socket
keywords: 
- socket
- receive
- block
description: 

---



最近socket调查一个bug的时候，发现一个“问题”。在c#中调用`socket.Receive(buff)`的时候，并没有阻塞当前线程，而是直接返回。

<!--more-->

查看MSDN：

> The Receive method reads data into the buffer parameter and returns the number of bytes successfully read. You can call Receive from both connection-oriented and connectionless sockets.
>
> This overload only requires you to provide a receive buffer. The buffer offset defaults to 0, the size defaults to the length of the buffer parameter, and the SocketFlags value defaults to None.
>
> If you are using a connection-oriented protocol, you must either call Connect to establish a remote host connection, or Accept to accept an incoming connection prior to calling Receive. The Receive method will only read data that arrives from the remote host established in the Connect or Acceptmethod. If you are using a connectionless protocol, you can also use the ReceiveFrom method. ReceiveFrom will allow you to receive data arriving from any host.
>
> If no data is available for reading, the Receive method will block until data is available, unless a time-out value was set by using Socket.ReceiveTimeout. If the time-out value was exceeded, the Receive call will throw a SocketException. If you are in non-blocking mode, and there is no data available in the in the protocol stack buffer, the Receive method will complete immediately and throw a SocketException. You can use the Available property to determine if data is available for reading. When Available is non-zero, retry the receive operation.
>
> **If you are using a connection-oriented Socket, the Receive method will read as much data as is available, up to the size of the buffer. If the remote host shuts down the Socket connection with the Shutdown method, and all available data has been received, the Receive method will complete immediately and return zero bytes.**
>
> If you are using a connectionless Socket, Receive will read the first queued datagram from the destination address you specify in the Connectmethod. If the datagram you receive is larger than the size of the buffer parameter, buffer gets filled with the first part of the message, the excess data is lost and a SocketException is thrown.

加粗位置：当使用面向连接的socket（比如使用TCP协议），`socket.Receive(buff)`方法会获取尽可能多的数据来填充`buff`。但是如果remote端(可以是client，也可以是server)调用`shutdown`，而且所有的数据都收到了，则再次调用`socket.Receive(buff)`会立即返回。