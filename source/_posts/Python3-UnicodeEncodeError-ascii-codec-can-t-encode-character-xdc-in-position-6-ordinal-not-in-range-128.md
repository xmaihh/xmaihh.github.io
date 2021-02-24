---
title: >-
  Python3-UnicodeEncodeError:-'ascii'-codec-can't-encode-character-'Ü'-in-position-6:-ordinal-not-in-range(128)
date: 2019-01-17 11:56:39
categories: Python
tags: [Python]
toc: true
description: Creativity requires the courage to let go of certainties.
---
# 问题
每次我尝试运行我的程序时都会返回错误，并且我的程序可以在其他应用程序中运行

```
Error:

Traceback (most recent call last):
File "/sdcard/pythonP/ex95.py", line 16, in 
gols[f"partida{g}"] = int(input(f"quantidade de gols na
partida {g}\xaa: "))
UnicodeEncodeError: 'ascii' codec can't encode character '\x
aa' in position 31: ordinal not in range(128)
```
正在尝试输出一个非ASCII字符的字符串。您是否尝试将LANG = C.UTF-8预先添加到您的python命令中，看看是否有帮助（Do LANG=C.UTF8 python3 your-script.py）？（至少ubuntu映像）似乎默认为Clocale，它只是ASCII。
Ubuntu rootfs中不正确配置的语言环境的结果（构建脚本引用了C语言环境，而C.UTF-8Unicode支持则需要）。设置此区域设置（例如，通过执行export LANG=C.UTF-8）一切正常。
这也打破了一些其他的应用程序（例如，pipenv并且mosh拒绝非Unicode终端上运行）。

问题可以通过以下简单的python脚本重现：

```
＃！/ usr / bin / env python3 
print（ “ HalloÜmlaut！”）
```
在Ubuntu上，这给了我以下错误
```
user@localhost:~$ python3 testcase.py
Traceback (most recent call last):
  File "testcase.py", line 2, in <module>
    print("Hallo \xdcmlaut!")
UnicodeEncodeError: 'ascii' codec can't encode character '\xdc' in position 6: ordinal not in range(128)
```

# 解决办法

```
$ export LANG = C.UTF-8
$ python3 your-script.py
```

