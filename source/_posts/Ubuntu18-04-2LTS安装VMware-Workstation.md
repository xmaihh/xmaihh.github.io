---

title: Ubuntu18.04.2LTS安装VMware Workstation
date: 2019-03-27 10:33:29
categories: Linux
tags: [Ubuntu]
toc: true
description: Don't play your life that you are not good at for audiences who do not belong to you.
---

# Ubuntu18.04.2LTS安装VMware Workstation

# 操作系统与软件版本

- Ubuntu18.04.2LTS bionic 
- VMware Workstation 14PRO或更高版本

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行

# Step1 下载VMware workstation

首先是下载VMware安装文件,打开命令行终端并使用`wget`命令执行

```shell
$ wget -O ~/vmware.bin https://www.vmware.com/go/getworkstation-linux
```

# Step2 安装WMware依赖

安装WMware依赖

```shell
$ sudo apt install build-essential
```

# Step3 安装WMware workstation

启动安装向导

```shell
$ sudo bash ~/vmware.bin
```
按照安装向导完成安装。如果您有序列号，请输入VMware序列号或留空。

附上VMware Workstation 15Pro激活密钥
```python
YG5H2-ANZ0H-M8ERY-TXZZZ-YKRV8

UG5J2-0ME12-M89WY-NPWXX-WQH88

UA5DR-2ZD4H-089FY-6YQ5T-YPRX6

GA590-86Y05-4806Y-X4PEE-ZV8E0

ZF582-0NW5N-H8D2P-0XZEE-Z22VA

YA18K-0WY8P-H85DY-L4NZG-X7RAD
```
![](https://i.loli.net/2019/03/27/5c9af1534a0f4.png)

# 启动VMware

在你的开始菜单搜索系统上安装好的VMware

![启动WMware](https://i.loli.net/2019/03/27/5c9af0b066e6a.png)

# 打开VMware

启动VMware Workstation

![Ubuntu 18.04.2LTS上的VMware Workstation 15 PRO](https://i.loli.net/2019/03/27/5c9af299bddbc.png)

# 附上：VMware Win镜像

链接: https://pan.baidu.com/s/1B4UBgVBaKm_GjDgbWxE-ig 提取码: `jh8z` 

