---
title: Ubuntu18.04.2LTS配置vnc+frp内网穿透实现桌面远程访问
date: 2019-07-10 18:13:28
categories: Linux
tags: Ubuntu
toc: true
description: Limitations live only in our minds. But if we use our imaginations, our possibilities become limitless.
---

之前使用**Teamviewer**来远程电脑，更新之后老是提示被商用，无奈，寻求其他方案如anyDesk、向日葵远程控制、splashtop、VNC。

frp项目地址：https://github.com/fatedier/frp
frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp,http 和 https协议。
VNC，全称为Virtual Network Computing，它是一个桌面共享系统。功能，类似于windows中的远程桌面功能。VNC与Windows远程桌面一样是使用RFB(Remote FrameBuffer，远程帧缓冲）协议来实现远程控制另外一台计算机。

# 目的

本文介绍的是在Ubuntu 18.04.2LTS Bionic Beaver上开启VNC服务端并frp穿透内网实现远程桌面共享。

# 环境

-  操作系统 Ubuntu 18.04.2LTS Bionic Beaver(x86_64)
- 云服务器（公网ip地址）

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行

# Step 1 启用远程桌面共享

首先查看下Ubuntu系统上是否安装远程桌面共享，没有的话就执行安装:

```shell
$ sudo apt update && sudo apt install -y vino
```

![desktop sharing](https://i.loli.net/2019/07/11/5d269f14e8d6180155.png)

使用`Activities`菜单搜索`Sharing`可以在`Settings`部分看到选项，直接打开它，或者以命令行的方式打开
```shell
$ gnome-control-center sharing
```
![Sharing](https://i.loli.net/2019/07/11/5d26a667acfd332834.png)

单击Screen Sharing以开始远程桌面配置

![](https://i.loli.net/2019/07/11/5d26a7c92f53141971.png)

将开关打开为`ON`，可以选择设置密码并记住你设置的密码，后面连接的时候要用密码。

`Allow connections to control the screen`选项使远程用户能够主动与远程桌面交互。如果未选中此选项，则远程桌面会话将设置为只读。

启用Ubuntu的远程桌面功能后，可以看到系统正在侦听端口`5900`

![](https://i.loli.net/2019/07/11/5d26a98f1745624536.png)

如果您启用了**UFW**防火墙，请打开`5900`端口

类似于:

```shell
$ sudo ufw allow from any to any port 5900 proto tcp
Rule added
Rule added (v6)
```

# Step 2 建立远程桌面连接

我们将在Ubuntu 18.04系统上使用[Remmina](https://remmina.org/)远程桌面客户端，如果没有先安装一个，打开`Ubuntu Software`搜索`remmina`安装或者命令行方式安装：

```shell
$ sudo apt install remmina
```



![](https://i.loli.net/2019/07/11/5d26ab913e00539267.png)

使用`Activities`菜单搜索并启动Remmina远程桌面客户端或运行命令行

```shell
$ remmina
```

![](https://i.loli.net/2019/07/11/5d26adac35c9b24367.png)

从下拉菜单中选择协议`VNC`,然后输入Ubuntu远程桌面系统的主机名或IP地址，按下`Enter`键即可连接.

![](https://i.loli.net/2019/07/11/5d26af736fdc768784.png)

输入前面设置的密码确认连接。

![](https://i.loli.net/2019/07/11/5d26aff21109717706.png)

连接成功，是这个样子。

![](https://i.loli.net/2019/07/11/5d26b11972e9c82984.png)

远程桌面共享连接成功。你也可以使用Remmina面板进一步调整远程桌面连接设置。

好了，到了这一步已经确认我们Ubuntu18.04系统的桌面共享服务已经安装好了。这意味着你将可以与你局域网内的机器实现桌面共享，接下来，使用frp实现内网穿透，对外网提供服务，随时随地可以使用网络远程桌面到Ubuntu 18.04。

# Step 3 配置frp实现内网穿透

首先下载frp二进制文件 https://github.com/fatedier/frp/releases
根据处理器架构选择对应压缩包
- Windows 64位 ：frp_版本号_windows_amd64.zip
- Windows 32位：frp_版本号_windows_386.zip
- Linux 64位：frp_版本号_linux_amd64.tar.gz
- Linux 32位：frp_版本号_linux_386.tar.gz

>注意：从0.18.0版本开始，新版与旧版不兼容，并且部分配置字段不同。请确保服务端和客户端使用同一版本。

## 下载程序

我这里云服务器和Ubuntu18.04都是`x86_64`架构的，所以我下载`linux_amd64`的压缩包

下面以`x84_64`处理器架构举例， 此时frp 最新版是`v0.27.0`

```shell
# 查看cpu架构
arch
# 创建frp文件夹并进入
mkdir frp && cd frp
# 下载
wget -q -c --no-check-certificate https://github.com/fatedier/frp/releases/download/v0.27.0/frp_0.27.0_linux_amd64.tar.gz
# 解压
tar -xzvf frp_0.27.0_linux_amd64.tar.gz
# 进入文件夹
cd frp_0.27.0_linux_amd64
# 确保 frps 程序具有可执行权限
chmod +x frps
# 运行
./frps --help
```

![](https://i.loli.net/2019/07/11/5d26db3f4493072975.png)

> 如果有报错  `-bash: ./frps: cannot execute binary file: Exec format error`就说明你下错版本

## 配置程序
服务端配置参考[frps_full.ini](https://github.com/fatedier/frp/blob/master/conf/frps_full.ini)
客户端配置参考[frpc_full.ini](https://github.com/fatedier/frp/blob/master/conf/frpc_full.ini)

### 服务端 -frps.ini
```shell
# 下面这句开头必须要有，表示配置的开始
[common]

# frp 服务端端口（必须）
bind_port = 7000

# frp 服务端密码（必须）
token = 12345678

# 认证超时时间，由于时间戳会被用于加密认证，防止报文劫持后被他人利用
# 因此服务端与客户端所在机器的时间差不能超过这个时间（秒）
# 默认为900秒，即15分钟，如果设置成0就不会对报文时间戳进行超时验证
authentication_timeout = 900

# 仪表盘端口，只有设置了才能使用仪表盘（即后台）
dashboard_port = 7500

# 仪表盘访问的用户名密码，如果不设置，则默认都是 admin
dashboard_user = admin
dashboard_pwd = admin

# 如果你想要用 frp 穿透访问内网中的网站（例如路由器设置页面）
# 则必须要设置以下两个监听端口，不设置则不会开启这项功能
vhost_http_port = 10080
vhost_https_port = 10443

# 此设置需要配合客户端设置，仅在穿透到内网中的 http 或 https 时有用（可选）
# 假设此项设置为 example.com，客户端配置 http 时将 subdomain 设置为 test，
# 则你将 test.example.com 解析到服务端后，可以使用此域名来访问客户端对应的 http
subdomain_host = example.com
```
这是我的服务端-frps.ini的配置
```shell
[common]
bind_port = 7000
token = 12345678
authentication_timeout = 900
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
vhost_http_port = 8080
subdomain_host = example.com
max_pool_count = 10
log_file = ./frps.log
log_level = info
log_max_days = 3
```

### 客户端 -frpc.ini

```shell
############################################
# 基本配置(必须)
# 下面这句开头必须要有，表示配置的开始
############################################
[common]
# frp 服务端地址，可以填ip或者域名
server_addr = 0.0.0.0
# frp 服务端端口，即填写服务端配置中的 bind_port
server_port = 7000
# 填写 frp 服务端密码
token = 12345678

###########################################
# 转发 ssh 
# 自定义一个配置名称，格式为“[名称]”，放在开头
###########################################
[ssh]
# 连接类型，填 tcp 或 udp
type = tcp

# 本地ip，填你需要转发到的目的ip
# 如果是转发到frp客户端所在本机（比如路由器）则填 127.0.0.1
# 否则填对应机器的内网ip
local_ip = 127.0.0.1
# 需要转发到的端口，比如 ssh 端口是 22
local_port = 22

# 是否加密客户端与服务端之间的通信，默认是 false
use_encryption = false
# 是否压缩客户端与服务端之间的通信，默认是 false
# 压缩可以节省流量，但需要消耗 CPU 资源
# 加密自然也会消耗 CPU 资源，但是不大
use_compression = false

# frp 服务端的远程监听端口，即你访问服务端的 remote_port 就相当于访问
# 客户端的 local_port，如果填0则会随机分配一个端口
remote_port = 6001

###########################################
# 转发HTTP(s)
# 自定义一个配置名称，格式为“[名称]”，放在开头
###########################################
[router-web]
# 连接类型，填 http 或 https
type = http

local_ip = 127.0.0.1
local_port = 80

# http 可以考虑加密和压缩一下
use_encryption = true
use_compression = true

# 自定义访问网站的用户名和密码，如果不定义的话谁都可以访问，会不安全
# 有些路由器如果从内部访问web是不需要用户名密码的，因此需要在这里加一层密码保护
# 如果你发现不加这个密码保护，路由器配置页面原本的用户认证能正常生效的话，可以不加
http_user = admin
http_pwd = admin

# 还记得我们在服务端配置的 subdomain_host = example.com 吗
# 假设这里我们填 web01，那么你将 web01.example.com 解析到服务端ip后
# 你就可以使用 域名:端口 来访问你的 http 了
# 这个域名的作用是用来区分不同的 http，因为你可以配置多个这样的配置
subdomain = web01

# 自定义域名，这个不同于 subdomain，你可以设置与 subdomain_host 无关的其他域名
# subdomain 与 custom_domains 中至少有一个必须要设置
custom_domains = web02.yourdomain.com

# 匹配路径，可以设置多个，用逗号分隔，比如你设置 locations 为以下这个，
# 那么所有 http://xxx/abc 和 http://xxx/def 都会被转发到 http://xxx/
# 如果不需要这个功能可以不写这项，就直接该怎么访问就怎么访问
locations = /abc,/def

# 重写 host header，相当于反向代理中的“发送域名”
# 如果设置了，转发 http 时，请求中的 host 会被替换成这个
# 一般情况下不需要用到这个，可以不写这项
host_header_rewrite = dev.yourdomain.com

##############################################
# TCP/UDP 范围转发
# 自定义一个配置名称，格式为“[range:名称]”，放在开头
##############################################
[range:multi-port]

type = tcp
local_ip = 127.0.0.1
use_encryption = false
use_compression = false

# 本地端口和远程端口可以指定多个范围，如下格式，且范围之间必须一一对应
local_port = 6010-6020,6022,6024-6028
remote_port = 16010-16020,16022,16024-16028
```

这是我的客户端-frpc.ini的配置
```shell
[common]
server_addr = example.com
server_port = 7000
token = 12345678

[web]
type = http
local_ip = 192.168.103.145
local_port = 80
custom_domains = example.com

[RDP]
type = tcp
local_ip = 192.168.103.145
local_port = 5900
remote_port = 5900
```

#### 服务器端运行启动
```shell
$ nohup /root/frp/frps -c /root/frp/frps.ini &
```
#### 客户端(Ubuntu18.04)启动
```shell
$ nohup /home/xmaihh/frp/frpc -c /home/xmaihh/frp/frpc.ini &
```

#### 停止
```shell
$ pkill frps   或者 pkill frpc
```

>云服务器注意放行相关端口.

#### 开机启动
自启动可以修改`/etc/rc.local`文件,加入启动命令
或者其他系统自行设置。

```shell
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
#nohup socat TCP4-LISTEN:4443,reuseaddr,fork TCP4:192.168.103.145:8123 >> /root/socat.log 2>&1 &
#nohup socat UDP4-LISTEN:4443,reuseaddr,fork UDP4:192.168.103.145:8123 >> /root/socat.log 2>&1 &
nohup /root/frp/frps -c /root/frp/frps.ini &
exit 0

```

# 建立远程连

请注意`remote_port = 5900`我把Ubuntu18.04的VNC Sharing端口`5900`映射到云服务器的`5900`端口，建立连接时，连接你的`服务器+服务器端口号`。

这里我使用手机开数据网络，关闭WIFi，下载`VNC Viewer`客户端来一下远程连接。

![](https://i.loli.net/2019/07/12/5d27d97f26fb410469.png)

打开手机`VNC Viewer`,点击`+`,在`Address`处填入 ip+端口形式 `xxx.xxx.xxx.xxx::5900`或者 域名+端口形式 `example.com::5900`。

![](https://i.loli.net/2019/07/12/5d27dba5ee57884621.png)

点击进行下一步输入密码以看到画面了。

![](https://i.loli.net/2019/07/12/5d27dbbc9663997267.png)

# Issues

## No matching security types

VNC客户端不支持加密。任何连接到远程桌面共享服务器的尝试都将导致`No matching security types`错误

如果遇到这个错误，请按照以下步骤解决:
1. 安装`dconf`工具

```shell
$ sudo apt-get install dconf-tools
```
2. 打开`dconf-editor`

```shell
$ dconf-editor
```

依次切换到`org->gnome->desktop->remote-access`并将`require-encryption`项目关闭。

![](https://i.loli.net/2019/07/12/5d27e1f110d9a37918.png)

3. 执行

```shell
$ gsettings set org.gnome.Vino require-encryption false
```

>如果输出警告：`GLib-GIO-Message: 10:19:43.137: Using the 'memory' GSettings backend.  Your settings will not be saved or shared with other applications.`
可以先执行`export GIO_EXTRA_MODULES=/usr/lib/x86_64-linux-gnu/gio/modules/`然后再执行3。



4. 确认已禁用远程服务器上的加密

```shell
$ gsettings list-recursively org.gnome.Vino | grep encrypt 
```

![](https://i.loli.net/2019/07/12/5d27e488a925935610.png)

# Reference

[How To Enable Desktop Sharing In Ubuntu and Linux Mint](https://www.tecmint.com/enable-desktop-sharing-in-ubuntu-linux-mint/)
[内网穿透神器搭建 萌新也看得懂的教程系列](https://moe.best/tutorial/frp.html)
