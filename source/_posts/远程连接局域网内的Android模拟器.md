---
title: 远程连接局域网内的Android模拟器
date: 2019-12-26 17:57:12
categories: Android
tags: Android
toc: false
description: つまらないものですが、お受（う）け取（と）りください。
---

本文主要介绍如何远程连接位于局域网内的 Android 模拟器进行调试。我们知道 Android 模拟器十分耗费资源，如果这时候有一台空余的机器，可以单独运行一个 Android 模拟器，然后再远程连接到该模拟器，从而能够减轻工作机的负担。

原理:使用 SSH 进行端口映射

# １.在空余机器打开Android模拟器，并打开Terminal终端，输入adb devices查看模拟器ip+端口
```shell
$ adb devices
List of devices attached/
192.168.58.102:5555	device
```
# 2.在运行AndroidStudio的机器上打开Terminal终端，ssh连接空余机器并将Android模拟器映射到本地localhost:15555，连接成功后注意不要关闭该窗口
```shell
ssh -p 22 -L localhost:15555:192.168.58.102:5555 xmaihh@192.168.200.189
```
# 3.在运行AndroidStudio的机器上打开新的Terminal终端，连接Android模拟器
```shell
adb connect localhost:15555
```
