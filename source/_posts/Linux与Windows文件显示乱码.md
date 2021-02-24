---
title: Linux与Windows文件显示乱码
date: 2019-06-05 15:44:29
categories: Linux
tags: Linux
toc: true
description: Today is going to be a beautiful day.
---

#　问题

从Windows内拷贝一个txt文件到Linux下打开显示乱码

- Windows下默认使用GB2312编码

- Linux下默认使用UTF-8编码

＃　解决办法

使用Linux下的`iconv`命令改变文件的编码
  
  `test.txt`由`GB2312`转换成`UTF-8`
```shell
  iconv　 -f 　GB2312　 -t　 UTF-8　 test.txt　 -o 　test.txt
```
  `test.txt`由`UTF-8`转换成`GB2312`
```shell
  iconv　-f　 UTF-8　 -t　 GB2312 　test.txt　 -o 　test.txt
```