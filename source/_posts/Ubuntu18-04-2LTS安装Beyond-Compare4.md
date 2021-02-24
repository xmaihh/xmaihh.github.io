---
title: Ubuntu18.04.2LTS安装Beyond-Compare4
date: 2019-04-01 14:47:17
categories: Linux
tags: [Ubuntu]
toc: true
description: A fool sees not the same tree that a wise man sees. 
---

# Ubuntu18.04.2LTS安装Beyond-Compare4

 [Beyond Compare](<https://www.scootersoftware.com/download.php>)是一套非常实用的文件及文件夹对比工具，不仅可以快速比较出两个文件夹的不同之处，还可以详细的比较文件之间的内容差异。

# 说明

- Ubuntu18.04.2LTS(64位)
- Beyond-Compare4

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行

# Step1  官网下载最新版Beyond Compare4

下载页面 <http://www.scootersoftware.com/download.php>

```shell
$ sudo dpkg -i bcompare-4.2.9.23626_amd64.deb
```

# Step2 激活License

```shell
$ cd /usr/lib/beyondcompare/
$ sudo sed -i "s/keexjEP3t4Mue23hrnuPtY4TdcsqNiJL-5174TsUdLmJSIXKfG2NGPwBL6vnRPddT7tH29qpkneX63DO9ECSPE9rzY1zhThHERg8lHM9IBFT+rVuiY823aQJuqzxCKIE1bcDqM4wgW01FH6oCBP1G4ub01xmb4BGSUG6ZrjxWHJyNLyIlGvOhoY2HAYzEtzYGwxFZn2JZ66o4RONkXjX0DF9EzsdUef3UAS+JQ+fCYReLawdjEe6tXCv88GKaaPKWxCeaUL9PejICQgRQOLGOZtZQkLgAelrOtehxz5ANOOqCaJgy2mJLQVLM5SJ9Dli909c5ybvEhVmIC0dc9dWH+/N9KmiLVlKMU7RJqnE+WXEEPI1SgglmfmLc1yVH7dqBb9ehOoKG9UE+HAE1YvH1XX2XVGeEqYUY-Tsk7YBTz0WpSpoYyPgx6Iki5KLtQ5G-aKP9eysnkuOAkrvHU8bLbGtZteGwJarev03PhfCioJL4OSqsmQGEvDbHFEbNl1qJtdwEriR+VNZts9vNNLk7UGfeNwIiqpxjk4Mn09nmSd8FhM4ifvcaIbNCRoMPGl6KU12iseSe+w+1kFsLhX+OhQM8WXcWV10cGqBzQE9OqOLUcg9n0krrR3KrohstS9smTwEx9olyLYppvC0p5i7dAx2deWvM1ZxKNs0BvcXGukR+/g" BCompare
```

# Step3 注册License

打开软件，会提示“Trial Mode Error! "表示Step2成功,接着输入下面TEAM ZWT生成的密钥即可注册成功

```shell
--- BEGIN LICENSE KEY ---
GXN1eh9FbDiX1ACdd7XKMV7hL7x0ClBJLUJ-zFfKofjaj2yxE53xauIfkqZ8FoLpcZ0Ux6McTyNmODDSvSIHLYhg1QkTxjCeSCk6ARz0ABJcnUmd3dZYJNWFyJun14rmGByRnVPL49QH+Rs0kjRGKCB-cb8IT4Gf0Ue9WMQ1A6t31MO9jmjoYUeoUmbeAQSofvuK8GN1rLRv7WXfUJ0uyvYlGLqzq1ZoJAJDyo0Kdr4ThF-IXcv2cxVyWVW1SaMq8GFosDEGThnY7C-SgNXW30jqAOgiRjKKRX9RuNeDMFqgP2cuf0NMvyMrMScnM1ZyiAaJJtzbxqN5hZOMClUTE+++
--- END LICENSE KEY -----
```

成功后会在~/.config/bcompare/下生成BC4Key.txt,内容如下：

```python
 xmaihh@xmaihh-H81M-S1:~$ cat ~/.config/bcompare/BC4Key.txt 
Beyond Compare 4
Licensed to:    pwelyn
Quantity:       9999 users
Serial number:  9571-9981
License type:   Pro Edition for Windows/Linux/OS X

--- BEGIN LICENSE KEY ---
GXN1eh9FbDiX1ACdd7XKMV7hL7x0ClBJLUJ-zFfKofjaj2yxE53xauIfk
qZ8FoLpcZ0Ux6McTyNmODDSvSIHLYhg1QkTxjCeSCk6ARz0ABJcnUmd3d
ZYJNWFyJun14rmGByRnVPL49QH+Rs0kjRGKCB-cb8IT4Gf0Ue9WMQ1A6t
31MO9jmjoYUeoUmbeAQSofvuK8GN1rLRv7WXfUJ0uyvYlGLqzq1ZoJAJD
yo0Kdr4ThF-IXcv2cxVyWVW1SaMq8GFosDEGThnY7C-SgNXW30jqAOgiR
jKKRX9RuNeDMFqgP2cuf0NMvyMrMScnM1ZyiAaJJtzbxqN5hZOMClUTE+
--- END LICENSE KEY -----
```

# Step 4 为所有用户注册bcompare命令

```shell
$ sudo cp ~/.config/bcompare/BC4Key.txt /etc/
```

如果Key失效，可以在 **https://www.serials.be/serial/Beyond_Compare_4_Linux_68803632.html**生成新的Key

![](https://i.loli.net/2019/04/01/5ca1c98459e45.png)

# Reference

[Crack-Beyond-Compare 4 -linux ](<https://my.oschina.net/sfshine/blog/1829595>)