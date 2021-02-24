---
title: Ubuntu18.04.2LTS安装ZeroTier实现内网穿透
date: 2019-04-02 10:24:13
categories: Linux
tags: [Ubuntu,ZeroTier]
toc: true
description: Living without an aim is like sailing without a compass. 
---

# ZeroTier

官网 https:/www.zerotier.com

github地址  <https://github.com/zerotier/ZeroTierOne>

ZeroTier在计算机和任何其他计算机之间设置VPN隧道组成一个局域网中，该网内设备自由访问，可完全免费使用多达100台设备。

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行

# Step 1 编译安装ZeroTier

```shell
$ git clone https://github.com/zerotier/ZeroTierOne.git  ~/ZeroTier
$ cd ~/ZeroTier
$ make
$ sudo ./zerotier-one -d
```

或者直接 去到 <http://www.zerotier.com/download.shtml>下载页面下载编译好的二进制文件安装

```shell
$ curl -s https://install.zerotier.com/ | sudo bash
```

安装完成将会收到如下打印：

![](https://i.loli.net/2019/04/02/5ca2cd8ca7fd1.png)

# Step 2 创建Network

进入官网 http://www.zerotier.com/ 注册账号登入后，创建一个Network

![](https://i.loli.net/2019/04/02/5ca2ccdd67369.png)

在终端运行以下命令:

```shell
$ sudo zerotier-cli join <network_id>
```

> network_id是上面创建的ID,记得使用root权限

![](https://i.loli.net/2019/04/02/5ca2d1ff54141.png)

#　Step 3  授权设备访问

在Network管理页面点击Network_ID进来后，可以看到`Access Control`默认设置为 `PRIVATE`

意味着所有机器必须先通过ZeroTier接口批准才能连接。这是默认选项，更安全

还有一个选项 `PUBLIC`

意味着具有网络ID的任何人都可以连接。这是最简单的选择，但安全性稍差。

这里我使用默认选项 `PRIVATE`

需要选中*Auth*下方的框 ，批准每台机器的连接这个网络

![](https://i.loli.net/2019/04/02/5ca2d3e05457e.png)

其他选项，比如这里可以选择 内网网段

![](https://i.loli.net/2019/04/02/5ca2d85f8bb98.png)

# Step  4 测试远程连接SSH

我在一台Android设备上安装[ZeroTier客户端](<https://play.google.com/store/apps/details?id=com.zerotier.one>)

Ubuntu18.04.2LTS打开SSH服务

安装即可

```shell
$ sudo apt-get install openssh-server
```

手机上操作比较简单，先`Join Network`，输入Network_ID选添加即可，打开后会出现一个建立VPN隧道的小钥匙图标 🔑。

 ![](https://i.loli.net/2019/04/02/5ca2fb875dd3b.png)

手机上打开ssh工具连接Ubuntu18.04.2LTS，可以看到我使用4G网络成功通过隧道ssh连接到了电脑

![](https://i.loli.net/2019/04/02/5ca2fcab14aac.png)

电脑终端查看ssh连接

![](https://i.loli.net/2019/04/02/5ca2d6f3bacf7.png)