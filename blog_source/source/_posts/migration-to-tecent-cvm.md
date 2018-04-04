---
title: 博客迁移到腾讯云
date: 2018-03-19 14:22:31
tags:
- hexo
categories:
- tutorial
keywords:
description:

---



终于受不了Github的龟速，速度慢就罢了，在联通的宽带下还经常timeout。虽然腾讯云的带宽也就1Mbps，好歹也能跑满，速度稳定在120多kb。于是在周末把博客完全迁移到了腾讯云上。



<!--more-->

## 流程

> 先明确一下它的运作流程：本地有个 hexo 程序，里面包含了 public 文件夹，sources 文件夹，hexo 将 sources 里的`*.md`文件渲染为静态的 html 文件放到 public 下，然后我们用git推送到服务器的`repository`，服务器用`git hooks`把仓库里的文件同步到网站根目录，而 nginx 的作用就是反向代理。
>
> - 服务器环境：安装git、nginx、创建git用户
> - 本地搭建Hexo环境：安装NodeJs、hexo-cli，生成本地静态网站
> - 使用git自动化部署发布博客

## 安装git，建立git用户

```shell
yum install git
adduser git
```

## 安装nginx

```shell
yum install nginx
systemctl start nginx
systemctl enable nginx.service #设置为开机启动
```

修改nginx的配置文件：

```shell	
vim /etc/nginx/nginx.conf
```

server模块配置如下（包含https的配置）

```shell
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  your_ip bygeek.cn www.bygeek.cn;
        #root         /usr/share/nginx/html;
        root         /usr/blog/www;
        return 301 https://$host$request_uri;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  your_ip www.bygeek.cn bygeek.cn;
        ssl on;
        # root         /usr/blog/www;

        ssl_certificate "/etc/nginx/ssl_folder/1_bygeek.cn_bundle.crt";
        ssl_certificate_key "/etc/nginx/ssl_folder/2_bygeek.cn.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
            root /usr/blog/www;
            index index.html index.htm;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

更改权限

```shell
cd /usr/blog
mkdir www
chmod -R 777 /usr/blog/www #更改文件夹及子文件夹权限
```

将`/usr/blog/www`作为网站的根目录。



## 服务器新建仓库

在服务器上新建裸仓库blog.git.

```shell
su git
cd ~ #切换到git用户的根目录 /home/git
mkdir blog.git
cd blog.git
git init --bare
```

利用git hooks，当blog.git收到push之后，自动同步到站点中。

```shell
vim ~/blog.git/hooks/post-receive
```

写入以下内容：

```shell
#!/bin/sh
git --work-tree=/usr/blog/www --git-dir=/home/git/blog.git checkout -f
```

赋予可执行权限：

```shell
chmod +x ~/blog.git/hooks/post-receive
```



## 设置公钥登录

在服务器端创建.ssh文件夹记忆authorized_keys文件。

```shell
su git
cd ~
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

在本地使用puttygen加载你的private key获取public key，复制之后粘贴到authrozied_keys文件中。如果使用的git bash，git bash使用ssh目录在`C:\Users\user_name\.ssh`中，id_rsa即为私钥文件。

```shell
vim ~/.ssh/authorized_keys
```

保存之后测试下是否能登录

```shell
ssh git@your_server_ip -p your_ssh_port
```



## 本地设置hexo

由于以前设置过hexo环境，本来博客的源文件是托管在github上的，这次索性将源文件也迁移到腾讯云上。

在git bash中，从服务器clone到本地

```bash
cd /d/blog
git clone ssh://git@your_server_ip:your_ssh_port/home/git/blog.git
```

注意：如果你的ssh端口不是默认的22端口，这需要指出ssh的端口号。blog.git 这个repo的地址是在上一步新建的目录。

新建一个hexo分支，用于保存markdown等博客源文件，master分支用于保存渲染好的html页面。

```bash
git branch hexo #新建hexo分支
git checkout hexo #切换到hexo分支
git push origin hexo #push到remote，即remote也新建了一个hexo分支
```

在git bash中

```bash
cd /d/blog
npm install hexo -g
mkdir blog_source
cd blog_source
hexo init
npm install
npm install hexo-deployer-git
```

本地的hexo环境搭建完毕，将以前的博客源文件覆盖过来。编辑站点_config.yml，在deploy中修改为腾讯云的repo地址

```
deploy:
  type: git
  repo: ssh://git@ip:port/home/git/blog.git
  branch: master
```

安装next theme

```bash
git clone --branch v5.1.4 https://github.com/iissnan/hexo-theme-next themes/next
```

将以前next theme的_config.yml文件覆盖。

由于使用了七牛云作为图床，安装七牛云插件

```shell
npm install hexo-qiniu-sync --save
```

next theme使用了第三方的local search：

```shell
npm install hexo-generator-searchdb --save
```



最后，将文件push到服务器上

```bash
git push origin hexo
```



## 参考链接

- [腾讯云的1001种玩法](https://cloud.tencent.com/developer/article/1004839)
- [git-scm](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
- [将Hexo博客部署到云主机](https://blog.fundebug.com/2017/05/18/deploy-hexo-on-cloud/)





