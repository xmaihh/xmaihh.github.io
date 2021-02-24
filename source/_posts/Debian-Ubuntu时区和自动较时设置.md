---
title: Debian/Ubuntu时区和自动较时设置
date: 2019-01-17 15:35:32
categories: Linux
tags: [Debian]
toc: true
description: It's probably hard for you to imagine how it feels to have your destiny be predetermined at birth.
---

NTP 是通过网络自动校时的一种 TCP/IP 协议。Debian/Ubuntu 中有两种方式实现时间同步：ntpdate 和 ntpd，前者为一天调整一次时间，后者 ntpd 为守护进程，可以持续不断地调整时间。个人推荐使用 ntpd，它实际占用资源是很小的。
# 设置时区

使用 tzconfig 或 tzselect 工具来设置时区

```
$ tzselect
```
选你要设置的时区,比如 Asia/China/Beijing

然后执行

```
$ cat >>~/.profile<<EOF
TZ='Asia/Beijing'; export TZ
EOF
```
最后执行

```
rm -rf /etc/localtime
cp /usr/share/zoneinfo/Asia/Beijing /etc/localtime
```

# 设置时间同步服务器

## 方法一：ntpdate 方式

```
apt-get install -y ntpdate #安装
vim /etc/cron.daily/ntpdate #添加下面一行，每天同步。
ntpdate ntp.ubuntu.com cn.pool.ntp.org
chmod 755 /etc/cron.daily/ntpdate #修改权限
ntpdate -d cn.pool.ntp.org #立即同步时间
```

## ntpd 方式

```
apt-get install -y ntpd #安装
vim /etc/ntp.conf #添加下面一行
server cn.pool.ntp.org
/etc/init.d/ntp restart #重启
```

## 以下适用于 debian

```
apt-get install -y ntp
vim /etc/ntp.conf #修改为下面几行
server 0.debian.pool.ntp.org iburst dynamic
server 1.debian.pool.ntp.org iburst dynamic
server 2.debian.pool.ntp.org iburst dynamic
server 3.debian.pool.ntp.org iburst dynamic
/etc/init.d/ntp restart #重启
```