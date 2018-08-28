---
title: centos 配置Shadowsocks备忘
date: 2018-08-28 11:22:23
tags:
- linux
categories:
- tutorial
keywords:
description:
---







本文作为一个centos配置shadowsocks的备忘。

<!--more-->

## 前言

一年前买的vps快到期了，临到续费的时候想起来能否开启bbr，试过google cloud之后，开启bbr之前与之后的效果提升了十倍。按照网上的教程操作了一波，在开启bbr的时候，由于centos的glibc版本太低，需要升级，无奈在centos 6上升级不成功。本来是通过KCPTun工具的来加速，对比了bbr的加速效果，觉得开启bbr的效果更开启KCP效果差不多。索性直接将vps的系统重装了一遍。本文是在**centos 7 64位**版本的实验的。

##  一键安装脚本

### 安装

使用root登录，运行以下命令：

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

安装过程会提示输入密码，端口等信息。

安装完毕之后，如果想修改参数，可以使用vim打开配置文件:

```bash
vim /etc/shadowsocks/config.json
```

会看到刚才的配置信息:

```json
{
    "server":"0.0.0.0",
    "server_port": your_port,
    "local_port":1080,
    "password":"your_passwd",
    "method":"rc4-md5",
    "timeout":300
}
```

对应修改即可。

### 卸载

找到上一步安装的脚本路径：

```bash
./shadowsocks-all.sh uninstall
```

### 启动脚本

启动脚本后面的参数含义，从左至右依次为：启动，停止，重启，查看状态。

```bash
/etc/init.d/shadowsocks start | stop | restart | status
```



## 开启bbr

对于不同虚拟架构的vps，脚本不一样。因此首先需要知道vps是基于什么架构的。

```bash
yum install virt-what
virt-what
```

如果输出是KVM，则是KVM架构，如果是openvz，则是openvz架构。

去年貌似openvz还不支持开启bbr，正好今天搜到一篇文章，openvz架构的vps也可以开启bbr了。由于我的vps是openvz架构的，所以KVM的未作测试。

### KVM

安装

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

验证

```bash
uname -r
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
sysctl net.core.default_qdisc
lsmod | grep bbr
```

### OpenVZ

安装

```bash
wget https://raw.githubusercontent.com/kuoruan/shell-scripts/master/ovz-bbr/ovz-bbr-installer.sh 
chmod +x ovz-bbr-installer.sh 
./ovz-bbr-installer.sh
```

实测脚本执行完毕，bbr开启成功。如果你的vps是centos 6版本，会出现执行脚本提示glibc版本过低，可以升级glibc版本，但是我升级失败，建议vps安装centos 7版本。

验证方法跟上面一致。

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

返回值一般为：
net.ipv4.tcp_available_congestion_control = bbr cubic reno

```bash
sysctl net.ipv4.tcp_congestion_control
```

返回值一般为：
net.ipv4.tcp_congestion_control = bbr

```bash
sysctl net.core.default_qdisc
```

返回值一般为：
net.core.default_qdisc = fq

```bash
lsmod | grep bbr
```

返回值有 tcp_bbr 模块即说明bbr已启动。



## 参考链接

- [How to determine Linux guest VM virtualization technology](How to determine Linux guest VM virtualization technology)
- [Install Shadowsocks](https://teddysun.com/486.html)
- [Enable Google BBR](https://www.bandwagonhost.net/268.html)













ok