---
title: Ubuntu18.04.2LTS安装Oracle Java JDK 8
date: 2019-03-15 14:10:54
categories: Linux
tags: [Ubuntu]
toc: true
description: How many cookies could a good cook cook if a good cook could cook cookies? A good cook could cook as much cookies as a good cook who could cook cookies.
---

# Ubuntu18.04.2LTS安装Oracle Java JDK 8

Webupd8 Team维护一个PPA存储库，其中包含适用于所有当前Ubuntu版本的Oracle Java 8安装程序脚本。

## 1.打开终端并运行命令添加PPA：

```
sudo add-apt-repository ppa:webupd8team/java
```

输入密码（输入时不会显示星号），然后按Enter键继续。

## 2.然后运行命令安装Java 8安装程序并在提示时接受许可证：

```
sudo apt-get install oracle-java8-installer
```

## 3.安装完成后，Oracle Java 8应自动设置为默认值。 如果没有，运行命令：

```
sudo apt-get install oracle-java8-set-default
```

## 4.卸载：

移除PPA软件包总是很容易，只需打开终端并运行命令即可：

```
sudo apt-get remove --autoremove oracle-java8-installer oracle-java10-installer
```



然后启动软件和更新 - >其他软件选项卡以删除PPA存储库。

# 在Ubuntu18.04.2LTS安裝Android NDK

## 1.下载地址：
[https://developer.android.com/ndk/downloads/index.html](https://link.jianshu.com/?t=https://developer.android.com/ndk/downloads/index.html)

```
wget -c http://dl.google.com/android/ndk/android-ndk-r18b-linux-x86_64.zip
```

## 2.解压

```
unzip android-ndk-r18b-linux-x86_64.zip
```

## 3.移动到自己想放的位置：

```
mkdir /usr/lib/ndk    
mv ndk-r18b  /usr/lib/ndk
```

## 4.设置环境变量

编辑 .profile 或者 .bash_profile,参照jdk环境变量设置

```
export ANDROID_NDK=/usr/lib/ndk/ndk-r18b
export PATH=$ANDROID_NDK:$PATH
```

## 5.使修改的配置立刻生效：执行`source /etc/profile`或者 `source ~/.bashrc`

## 6.检查是否安装成功：

```
ndk-build -C
```

若返回下面的信息表示NDK有效

[![檢查ndk-build是否安裝成功](https://i.loli.net/2019/03/15/5c8b0bccde35a.png)](https://i.loli.net/2019/03/15/5c8b0bccde35a.png)檢查ndk-build是否安裝成功