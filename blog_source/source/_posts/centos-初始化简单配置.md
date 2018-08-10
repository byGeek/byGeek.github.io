---
title: centos 初始化简单配置
date: 2018-03-12 22:08:30
tags:
- linux
- centos
categories:
- tutorial
keywords:
description:

---



centos简单折腾记录。

## 新建用户

1. 使用root用户登录到server端。
2. 新建用户账号：`adduser robert`
3. 设置密码: `passwd robert`

## 增加用户到sudoer中

root用户拥有系统的最高权限，但是为了系统的安全性，一般不会直接使用root用户。相反我们会使用sudo命令来暂时提高当前用户的权限。下一步我们将新建的用户robert加入到sudoer中。在centos中，在`wheel`用户组的用户具有sudo权限。

```shell
usermod -aG wheel robert
```

测试是否成功。

```shell
su robert
sudo ls -al /root
```

如果还是提示用户不再sudoer中。那么还需要修改`/etc/sudoers`文件。sudo命令是由该文件来配置哪个用户及用户组可以执行。注意该文件不要随便修改，因为错误的语法错误可能会导致用户无法通过sudo来提升权限。需要通过`visudo`命令来修改。`visudo`命令默认使用vi来打开文件，但是在保存文件的时候会检查配置是否有语法错误。

```shell
visudo -f /etc/sudoers
```

找到wheel（可以使用vi中进行搜索字符）。如下面所示，去掉前面的#号，取消该行注释。%wheel表示的是wheel 用户组。参考[链接](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos#what-is-visudo)

```shell
...
%wheel ALL=(ALL) ALL
...
```

然后再测试是否切换到创建的用户，是否能够执行sudo命令。



## 修改SSH默认端口

ssh是一个安全的加密协议，用于主机之间的通信。为了加强系统的安全性，修改默认的ssh的22端口。

1. 修改ssh_config文件中的默认端口号

   ```shell
   vim /etc/ssh/sshd_config
   ```

   找到`#Port 22`这一行，去掉#，取消注释，修改为你想要该的端口号，如10086。

2. 按需修改修改防火墙规则和更新selinux规则。

   centos7执行：

   ```shell
   firewall-cmd --add-port 10086
   firewall-cmd --add-port 2345/tcp --permanent
   ```

   centos6执行：

   ```shell
   iptables -I INPUT -p tcp --dport 10086 -j ACCEPT
   ```

   同时按需修改selinux:

   ```shell
   semanage port -a -t ssh_port_t -p tcp 10086
   ```

3. 重启ssh服务: 

   ```shell
   systemctl restart sshd.service
   ```

4. 测试ssh链接：

   ```shell
   ssh robert@ip_address -p 10086
   ```



## 配置公钥登录

在windows上一般使用putty或xshell来作为ssh客户端。这里以xshell为例。在xshell中`Tools->User key Manager->Generate`生成公钥/私钥对。将私钥保存好，同时将公钥复制到剪贴板。



使用xshell ssh连接到远程主机，在当前用户HOME目录下执行如下命令：

```bash
cd ~
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

ssh服务默认使用用户.ssh目录下的authorized_keys中的公钥来进行验证。将上一部复制的公钥复制到authorized_keys文件中。



如果想配置root也使用公钥登录，需要在root目录下也建立.ssh文件夹和authorized_keys文件。注意，需要更改文件夹和文件权限！



## 禁止root远程密码登录

修改ssd的配置文件

```bash
sudo vim /etc/ssh/sshd_config
```

将`#PermitRootLogin yes`修改为`PermitRootLogin without-password`。注意是修改为without-password，如果直接修改为no，则root公钥也不能登录了。



## 参考链接

- [centos wiki](https://wiki.centos.org/HowTos/Network/SecuringSSH)
- [add sudoer](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-centos-quickstart)
- [add sudoer 2](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos#what-is-visudo)