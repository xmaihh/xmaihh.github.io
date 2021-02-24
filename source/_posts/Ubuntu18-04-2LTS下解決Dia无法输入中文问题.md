---
title: Ubuntu18.04.2LTS下解決Dia无法输入中文问题
date: 2019-04-16 09:35:43
categories: Linux
tags: Ubuntu
toc: true
description: How many cookies could a good cook cook if a good cook could cook cookies?
---

Dia是一款和MS *Visio*类似的绘制流程图、UML图、电路图、网络、数据库等结构化图形的工具，支持[Mac OS X ](http://dia-installer.de/download/macosx.html) 、[Linux](http://dia-installer.de/download/linux.html) 、[Windows ](http://dia-installer.de/download/index.html)  。

- 官网地址[Dia Diagram Editor](<http://dia-installer.de/>)
- 开源地址:[github](<https://github.com/GNOME/dia>)
![Dia界面](https://i.loli.net/2019/04/16/5cb536cd29418.png)
# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行
# 安装Dia
```shell
$ sudo apt install dia -y
```

# 方法1

修改/usr/bin/dia文件

```shell
dia-normal --integrated "$@"改为 dia-normal "$@"
```
一番搜索下来比较多的方法是修改`/usr/bin/dia`文件,当我敲下 `sudo vim /usr/bin/dia`命令后,展示在我眼前的是以下画面
![](https://i.loli.net/2019/04/16/5cb53879c9912.png)
是Doc转Unix转码问题，不成功，网上说这样能解决中文输入的问题，但是又会引起左边工具栏和主窗口分离的问题，每次画个图还得经常拖动工具条，调整其大小，反正就是用起来很麻烦，不完美，遂放弃。

# 方法2

修改 `/usr/share/applications/dia.desktop`文件
```shell
把Exec=dia %F 改为Exec=env GTK_IM_MODULE=xim dia %F
```
这个设置解决了从启动栏的快捷方式中启动Dia后，输入中文的问题。

# 方法3

在终端启动时增加启动设置
启动命令dia 前边增加`env GTK_IM_MODULE=xim`，即用`env GTK_IM_MODULE=xim dia`来启动Dia，为了避免每次启动都要输入这么一长串，我们设置别名alias，执行命令`alias dia="env GTK_IM_MODULE=xim dia"`，以后再启动Dia时还是使用dia就可以了
这个解决了从终端启动Dia后，输入中文的问题。

# Reference

[解决Dia在Linux上的输入法问题](https://jlice.top/post/201805311.html)
[完美解决Dia无法输入中文的问题](https://www.jianshu.com/p/9a7d736126d1)