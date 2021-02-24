---
title: Ubuntu18.04.2LTS下Wireshark报错Permission denied
date: 2019-07-19 14:50:33
categories: Linux
tags: [Ubuntu]
toc: true
description: Look up at the stars, not down at your feet. 
---

# 安装Wireshark

```shell
$ sudo apt-get install wireshark
```
打开Wireshark，报错提示权限不足。

```
Couldn’t run /usr/bin/dumpcap in child process: Permission denied
```

# 解决方案

```shell
$ sudo apt-get install libcap2-bin
// 添加一个组，名字为 wireshark
$ sudo groupadd wireshark  
// 把当前的用户名添加到 wireshark组
$ sudo usermod -a -G wireshark YOUR-USER-NAME
$ newgrp wireshark
// 修改组别
$ sudo chgrp wireshark /usr/bin/dumpcap
// 添加执行权限
$ sudo chmod 754 /usr/bin/dumpcap
$ sudo setcap 'CAP_NET_RAW+eip CAP_NET_ADMIN+eip' /usr/bin/dumpcap
```

# 重启或者登出当前用户重新登入

