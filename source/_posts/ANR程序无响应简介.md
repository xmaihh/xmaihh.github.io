---
title: ANR程序无响应简介
date: 2018-07-12 11:52:34
categories: Android Framework
tags: [Android]
toc: true
description: Your pain is the breaking of the shell that encloses your understanding. 
---
ANR，英文全称为 Application Not Responding，即应用无响应。
具体表现，弹出一个应用无响应的窗口，也可能不弹出直接闪退。
### ANR的类型
ANR一般有三种类型：

1. KeyDispatchTimeout(5 seconds) –主要类型 按键或触摸事件在特定时间内无响应
定义参考：ActivityManagerService.java
```
// How long we wait until we timeout on key dispatching.
static final int KEY_DISPATCHING_TIMEOUT = 5*1000;
```
2. BroadcastTimeout(前台 10 seconds，后台 60 seconds) BroadcastReceiver在特定时间内无法处理完成
定义参考：ActivityManagerService.java
```
// How long we allow a receiver to run before giving up on it.
static final int BROADCAST_FG_TIMEOUT = 10*1000;
static final int BROADCAST_BG_TIMEOUT = 60*1000;
```
3. ServiceTimeout(前台 20 seconds，后台 200 seconds) –小概率类型 Service在特定的时间内无法处理完成
定义参考：ActiveServices.java
```
// How long we wait for a service to finish executing.
static final int SERVICE_TIMEOUT = 20*1000;
// How long we wait for a service to finish executing.
static final int SERVICE_BACKGROUND_TIMEOUT = SERVICE_TIMEOUT * 10;
```
### ANR产生的原因
1. 应用自身引起，例如：
主线程阻塞、IOWait等；
2. 其他进程间接引起，例如：
当前应用进程进行进程间通信请求其他进程，其他进程的操作长时间没有反馈；
其他进程的CPU占用率高，使得当前应用进程无法抢占到CPU时间片

### ANR的分析解决
1. 分析ANR Trace文件
ANR Trace文件的路径：/data/anr/traces.txt 
导出traces.txt文件
```
adb pull /data/anr/traces.txt      ./
```
>注意:每次发生ANR时都会删除旧的traces文件，重新创建新文件。也就是说Android只保留最后一次发生ANR时的traces信息
2. 分析Android的异常信息收集DropBox的日志
DropBox文件路径: /data/system/dropbox
```
adb pull /data/system/dropbox/      ./
```
3. 通过[BlockCanary](https://github.com/markzhai/AndroidPerformanceMonitor)检测耗时操作，应用内集成进行检测,将IO操作/耗时操作全部封装成异步任务，放进子线程.
