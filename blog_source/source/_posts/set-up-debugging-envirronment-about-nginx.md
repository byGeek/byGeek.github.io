---
title: 搭建nginx调试环境
date: 2018-04-18 09:10:06
tags:
- nginx
categories:
- [coding, nginx]
keywords:
- nginx
- debugging
description:

---



本文简单记录一下搭建nginx源码调试环境的过程。

<!--more-->

## Windows

在windows下使用宇宙最强IDE进行调试，可谓事半功倍。但是nginx的源码并没有提供visual studio的solution工程文件，所幸在github上找到了一个repo，别人已经将其他一些messup的东西都给你搞好了，直接F5就能run起来，简单省事。



Repo地址:  https://github.com/tumtumtum/nginx-visualstudio



## Linux

好在微软发布了一个跨平台的VS Code，在linux下可以使用它作为集成开发环境。下面以ubuntu为例。

### 安装VS Code

在官网下载vs code deb包，安装好，再安装C/C++插件

### 安装GCC/GDB

安装GCC编辑器和GDB调试器，我使用的ubuntu desktop 16.04版本自带这些，所以就没有额外安装，如果系统没有带，需要额外安装。

```bash
sudo apt install build-essential -y
```

### 下载Nginx源码与依赖库

使用git 将repo clone下来。如果没有安装git，需要先安装git。

```bash
cd ~
git clone https://github.com/nginx/nginx.git
```

注意，如果你是在虚拟机使用git，记得使用https方式，而不是ssh方式来下载代码。因为你可能没有为虚拟机生成公钥。

> NGINX depends on 3 libraries: [PCRE](http://www.pcre.org/), [zlib](https://www.zlib.net/) and [OpenSSL](https://www.openssl.org/):

```bash
# PCRE version 4.4 - 8.40
wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz && tar xzvf pcre-8.40.tar.gz

# zlib version 1.1.3 - 1.2.11
wget http://www.zlib.net/zlib-1.2.11.tar.gz && tar xzvf zlib-1.2.11.tar.gz

# OpenSSL version 1.0.2 - 1.1.0
wget https://www.openssl.org/source/openssl-1.1.0f.tar.gz && tar xzvf openssl-1.1.0f.tar.gz
```

### 编译源码

执行configure shell脚本，编译代码。注意，configure脚本在 `auto`目录下，我们需要拷贝到代码根目录

```bash
cd nginx
cp auto/configure configure
./configure --with-pcre="./pcre-8.40"	\
	    --with-zlib="./zlib-1.2.11"	\
	    --with-openssl="./openssl-1.1.0f"	\
	    --with-debug
```

上述configure指出的参数为依赖库的目录地址。

执行成功之后在根目录生成obj目录和Makefile。正式进行编译。

```bash
make -f Makefile
```

### 使用VS Code进行debug

使用VS Code 打开nginx 代码根目录，F5开始调试，如果是第一次进行调试，会打开launch.json文件。修改可执行程序目录。

{% asset_img launch_json.png %}

打上断点即可开始调试，效果如下图。

{% asset_img vs_code_debug_nginx.png %}



## 总结

由于自己以前没哟在linux下的开发经验，导致搭建调试环境的时候遇到了一些问题。比如makefile是怎么一回事等等。希望自己能好好看看nginx的源码，打算在接下来的博文中出一个nginx源码解读系列。



## 参考链接

[Debug Nginx source code](https://www.vultr.com/docs/how-to-compile-nginx-from-source-on-ubuntu-16-04)

