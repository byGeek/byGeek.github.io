---
title: 记一次修改联通光猫为桥接模式的经历
date: 2019-08-18 09:38:23
tags:
- geek
categories:
- notes
keywords:
- 光猫
- 桥接模式
description:
---

自从搬家换了宽带移机之后，本来群晖的外网访问就失效了。因为前段时间一直都比较忙，一直没好好解决这个问题。今天终于得闲，把这个问题整好了。以下是我的折腾记录。

<!--more-->

### 公网IP被取消？

黑裙无法外网访问，首先想到的可能是联通这边不给公网IP了。打开ip138查看出口IP地址，是公网IP。当然这个并不能说明问题，因为这个检测到的只是出口IP，可能家里还是内网IP。

登录路由器管理界面，在远程管理看到路由器的IP地址为192.168的内网地址。内网IP无疑了。于是联系联通客服，质问为何移机之后的宽带取消了公网IP。一个工作日客服人员答复家里的宽带公网IP并未取消。

### 无法ping通

其实确认有公网IP之后，路由器获取的又是内网地址，如果对网络拓扑和组网有简单的了解之后，已经可以确定可能是光猫的问题了，但是好久不搞这些东西，我已经把计算机网络的一些东西忘了差不多了。真是惭愧。

言归正传，当时我的想法是，既然我还是有公网IP，那么我试着ping下试试。使用我在腾讯云和GCP上的VPS对家里公网IP进行ping操作，无法ping通，100% package loss。有可能是封禁了ICMP response？

我又想到可以使用端口扫描工具，对该IP进行端口扫描，看是不是端口被封禁的问题。

```
nmap -Pn ip_addr
```

找不到open的端口。端口都被封了？

### 在群晖设置router 端口转发找到问题

走投无路的情况下，通过内网，我打开群晖的external access的问题。在重新配置router 端口转发的时候，群晖自检网络时抛出了一个warning：网络里存在多个路由，需要将接入设置为桥接模式。

{%asset_img set_up_router.png %}

感觉找到了一个突破口。我马上找到光猫，开启无线功能，然后连上光猫的无线网之后，登录到web界面。但是发现由于是user用户，找不到修改光猫工作为桥接模式的设置。于是，在google上搜索“光猫 桥接”，找到了简书上的一篇文章： [电信光猫桥接模式的设置](https://www.jianshu.com/p/0211c56c4945)。这篇文章也是碰到了外网访问的问题。修改光猫为桥接模式，必须要拿到管理员账户。那怎么搞到光猫的控制台密码呢？

### 要到光猫管理员密码

我直接在google上搜索的光猫的型号：吉比特TEWA 800G。找到一个知乎相关的问题：[吉比特TEWA 800G的管理员登录地址是什么？](https://www.zhihu.com/question/316754462)。这个问题虽然没有人回答，但是在问题的描述中我知道了管理员登录的网址。这个光猫的登录界面还做的一个鸡贼的处理：当直接输入光猫的登录地址192.168.1.1时，只能看到普通用户登录的选项，没有管理员账户的登录选项。正好这个知乎问题上说明管理员的登录地址时: `192.168.1.1/cu.html`。

接下来就差密码了，我倒是想搜索相关的“破解”方法，不过本着试一试的想法，我直接打电话给装宽带小哥，询问他光猫的管理员的密码之后，没想到宽带小哥直接告诉了我！

使用管理员密码登录之后，我直接在光猫的快速设置向导中，将路由模式修改为桥接模式。然后在自己的路由器上使用PPPoE拨号。但是无法成功，提示没有连接到互联网。怎么回事呢？

### 要到vlan id

我重新看了简书上的[这篇文章](https://www.jianshu.com/p/0211c56c4945)，里面说道改成桥接模式之后，需要修改VLAN ID，这个VLAN ID是原来路由模式的VLAN ID。再确认了一下我自家光猫的配合，没有问题，但是就是无法成功拨号。

无奈之下我再次拨打了宽带小哥的电话。小哥说让我加钱，他上门服务。我说别了吧，我自己弄就好了，不用上门，我请教你几个问题。没想到小哥也同意了。我加了小哥微信，把光猫的VLAN配置发他看了。他询问了我光猫上的一个参数之后，告诉我VLAN ID（这个VLAN ID并不是路由模式的VLAN ID），同时他说需要他在那边“操作一下”。

看来这个VLAN ID并不是像简书上的文章说的那样，而且需要宽带人员的操作（好像是要解绑什么东西？）。所以遇到问题需要按照实际情况进行分析。

我填入小哥说的VLAN ID之后，拨号成功！

### 外网成功访问

打开路由器的管理界面，在远程访问中可以看到路由器的IP不再是192.168了，而是公网IP。

然后在群晖上执行下DDNS脚本，终于搞定了外网访问的问题！

### 桥接与VLAN

其实这件事情很小，而且起初确认公网IP没有取消，路由器获取的是内网IP的时候，当时就应该可以判断是光猫的问题，但是由于还是一些组网知识都忘了，所以走了一些弯路。

下面在这里贴一下这期间碰到的知识点。

> Q: [What's the difference between a bridge and a switch?(https://serverfault.com/questions/78184/whats-the-difference-between-a-bridge-and-a-switch)
>
> An ethernet switch is a multiport ethernet bridge. A bridge is a device that splits collision domains but not broadcast domains. A switch is simply a bridge with lots of ports. Other examples of bridges are wireless access points and dual speed hubs. 

> A **virtual LAN** (**VLAN**) is any [broadcast domain](https://en.wikipedia.org/wiki/Broadcast_domain) that is [partitioned](https://en.wikipedia.org/wiki/Network_segmentation) and isolated in a [computer network](https://en.wikipedia.org/wiki/Computer_network) at the [data link layer](https://en.wikipedia.org/wiki/Data_link_layer) ([OSI layer 2](https://en.wikipedia.org/wiki/OSI_model#Layer_2:_Data_Link_Layer)).[[1\]](https://en.wikipedia.org/wiki/Virtual_LAN#cite_note-1)[[2\]](https://en.wikipedia.org/wiki/Virtual_LAN#cite_note-802.1Q_1.4-2) *LAN* is the abbreviation for *local area network* and in this context *virtual* refers to a physical object recreated and altered by additional logic. VLANs work by applying tags to network frames and handling these tags in networking systems – creating the appearance and functionality of [network traffic](https://en.wikipedia.org/wiki/Network_traffic) that is physically on a single network but acts as if it is split between separate networks. In this way, VLANs can keep network applications separate despite being connected to the same physical network, and without requiring multiple sets of cabling and networking devices to be deployed.



