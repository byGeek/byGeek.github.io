---
title: centos 初始化简单配置
date: 2018-03-12 22:08:30
tags:
- linux
- centos
categories:
- linux
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

## 参考链接

- [centos wiki](https://wiki.centos.org/HowTos/Network/SecuringSSH)
- [add sudoer](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-centos-quickstart)