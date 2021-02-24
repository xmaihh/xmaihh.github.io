---
title: 树莓派4-Ubuntu18.04.2LTS下配置WiFi与SSH连接
date: 2019-09-17 11:03:51
categories: Raspberry Pi
tags: [Raspberry Pi]
toc: true
description: Storm clouds may gather and stars may collide but I love you until the end of time.
---

# 准备环境

- [Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
- SD卡，读卡器
- 电源适配器

# 开始

## 下载安装到SD卡

树莓派系统官方镜像下载：·[Raspberry Pi Raspbian](https://www.raspberrypi.org/downloads/raspbian/)

下载[Etcher](https://www.balena.io/etcher/)工具，把镜像烧写到SD卡里面去
工具的操作很简单的三个步骤:
选择镜像 --> 选择SD卡 --> 烧写!

![](https://i.loli.net/2019/09/17/n6rucIg4UNXQTqj.png)

## 环境配置

出于安全考虑，SSH在raspbian中默认disabled。启用SSH,在SD卡的boot目录下建立一个空白文件`ssh`即可启用,文件命名为 ssh。注意要小写且不要有任何扩展名。

配置WiFi,在SD卡的boot目录下建立一个 `wpa_supplicant.conf`文件

```shell
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US
 
network={
ssid="WiFi-A"
psk="123456789"
key_mgmt=WPA-PSK
priority=1
}
 
network={
ssid="WiFi-B"
psk="123456789"
key_mgmt=WPA-PSK
priority=2
}
```

![](https://i.loli.net/2019/09/17/GbJksDFXnLjf5i8.png)

## 连接数莓派

将SD卡放入Raspberry Pi中，它应该在启用网络的情况下启动到Debian。如果出现问题，您可以通过将SD卡移回Linux计算机并查看var / log目录来查看日志。

要连接到Raspberry Pi，需要知道它已分配的IP地址。在无线路由器上找到Raspberry Pi的IP地址,需确保Ubuntu 连接的和Raspberry Pi是在同一个局域网内.

```shell
$ ssh pi@hostname
```

> hostname为树莓派的ip地址.

官方系统镜像默认用户名为`pi`,密码为`raspberry`.

![](https://i.loli.net/2019/09/17/8qxRmcvLn7zAEGJ.png)


>执行`passwd`命令来修改默认密码

Raspbian镜像基于Debian Buster,基本命令使用和Debian基本一致.

- 更新
```shell
pi@raspberrypi:~ $ sudo apt-get update
```

- 升级
```shell
pi@raspberrypi:~ $ sudo apt-get upgrade
```

- 安装软件
```shell
pi@raspberrypi:~ $ sudo apt-get install <name of software>
```

- 检查SD容量
当运行`upgrade`命令时，会显示数据下载量和空间需要量。可以运行df -h来确定卡剩余空间是否足够。注意下载的包(.deb文件)保存在`/var/cache/apt/archives`路径下。你可以通过`sudo apt-get clean`来争取空间。

- 配置更改
可对树莓派进行配置更改
```shell
sudo raspi-config
```

# 安装docker和docker-compose

树莓派是ARM架构的,如果需要找树莓派专用的镜像，那就在Dockerhub上搜索ARM或Rpi相关就能找到了。
有一个叫[Hypriot](https://hub.docker.com/u/hypriot/)的仓库制作了非常多树莓派专用docker，可以参考下。

## 安装docker

```shell
pi@raspberrypi:~ $ sudo apt-get update
pi@raspberrypi:~ $ sudo apt-get upgrade
pi@raspberrypi:~ $ curl -sSL https://get.docker.com| sh
```
然后运行`hello world`检测是否安装成功.
```shell
pi@raspberrypi:~ $ sudo docker run hello-world
```
## 无需sudo执行docker
为了每次执行docker不需要总是输入sudo，我们需要为docker创建一个用户组，并授予权限才行

```shell
# 创建docker用户组
pi@raspberrypi:~ $ sudo groupadd docker

# 把当前用户加入到docker用户组
pi@raspberrypi:~ $ sudo gpasswd -a $USER docker

# 更新当前用户组变动（就不用退出并重新登录了）
pi@raspberrypi:~ $ newgrp docker
```

或者 直接执行下面这条命令,会重启,需要重新登入.
```shell
pi@raspberrypi:~ $ sudo usermod -aG docker pi && sudo reboot
```

## 安装docker-compose

### 第一种方法

```shell
pi@raspberrypi:~ $ sudo apt-get update
pi@raspberrypi:~ $ sudo apt-get install -y python python-pip
pi@raspberrypi:~ $ sudo pip install docker-compose
```
![](https://i.loli.net/2019/09/17/JZfydCovKcF9MTn.png)

先执行,再执行`docker-compose`安装

```shell
pi@raspberrypi:~ $ sudo apt install -y python python-pip libffi-dev
```



~~### 第二种方法~~
~~https://github.com/docker/compose~~
~~通过把`docker compose`当作一个`docker`的`container`下载并运行~~

```shell
docker run \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:/rootfs/$PWD" \
    -w="/rootfs/$PWD" \
    docker/compose:1.24.1 up
```

~~设置alias快捷键,编辑(`~/.bash_profile`)~~

```shell
alias docker-compose="'"'docker run \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:/rootfs/$PWD" \
    -w="/rootfs/$PWD" \
    docker/compose:1.24.1'"'"
```

# Reference

[[树莓派安装Docker](https://segmentfault.com/a/1190000018028887)]
[Raspberry Pi - Linux computer for learning programming](http://www.penguintutor.com/raspberrypi/)




