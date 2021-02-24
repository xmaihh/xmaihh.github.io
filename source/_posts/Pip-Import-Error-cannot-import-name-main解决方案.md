---
title: 'pip Import Error:cannot import name main解决方案'
date: 2018-10-13 09:09:16
categories: Linux
tags: [ ArchLinux]
toc: false
description:  Once you learn to quit, it becomes a habit.
---
在使用pip来进行安装操作时碰到这样的问题
```
xmaihh@ubuntu:~$ pip install jrnl
Traceback (most recent call last):
  File "/usr/bin/pip", line 9, in <module>
    from pip import main
ImportError: cannot import name main

```
是因为将pip更新为10.0.0后库里面的函数有所变动造成这个问题
解决方案：
```
sudo vi /usr/bin/pip
```
将原来的:
```
from pip import main
if __name__ == '__main__':
    sys.exit(main())
```
改成:
```
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```