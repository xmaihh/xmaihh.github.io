---
title: Ubuntu18.04.2LTS安装微信web开发工具(wine)
date: 2019-05-14 17:09:03
categories: 微信小程序
tags: [Ubuntu]
toc: true
description: Adopting the right attitude can convert a negative stress into a positive one.
---

当我打开微信小程序开发者工具[下载页](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html?t=19051421)
![](https://i.loli.net/2019/05/15/5cdb74fe9716f93266.png)

额，没有Linux版本。好吧，自力更生。

# Linux微信web开发者工具

# 安装环境

- [Wine](https://wiki.winehq.org/Ubuntu)
- [Linux微信web开发者工具](https://github.com/cytle/wechat_web_devtools)

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行

如果您之前安装过来自其他仓库的 Wine 安装包，请在尝试安装 WineHQ 安装包之前删除它及依赖它的所有安装包（如：wine-mono、wine-gecko、winetricks），否则可能导致依赖冲突。

## Step1 删除之前安装的Wine
```shell
// 1. 删除软件及配置文件
$ sudo apt-get --purge remove wine
// 2. 删除没用的依赖包
$ sudo apt-get autoremove wine
// 3. 此时dpkg的列表中有"rc"状态的软件包,可以执行以下命令进行最后清理
$ dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
// 4. 然后删除安装包,位于/root/.wine和/home/usrname/.wine
$ rm -rf /root/.wine
$ rm -rf /home/usrname/.wine
```

> 删除之后可能残留wine快捷方式的残留目录，进入目录检查下是否包含wine相关的内容
>
> /usr/share/applications/            *//wine快捷方式*
> /usr/share/app-install/                *//wine快捷方式*
> /home/username/.lacal/             *//wine应用程序快捷方式*
> /home/username/.cache/           *//wine应用程序快捷方式*
> /home/username/.config/menus/applications-merged/ 

## Step2 安装 [WineHQ 安装包](<https://wiki.winehq.org/Ubuntu_zhcn>)

如果您使用的是 64 位系统，请开启 32 bit 架构支持（如果您之前没有开启的话）：
```shell
$ sudo dpkg --add-architecture i386 
// 下载添加仓库密钥
$ wget -nc https://dl.winehq.org/wine-builds/Release.key
$ sudo apt-key add Release.key
$ sudo apt-add-repository https://dl.winehq.org/wine-builds/ubuntu/
// 添加仓库
$ sudo apt-add-repository'deb https://dl.winehq.org/wine-builds/ubuntu/ xenial main'
// 更新软件包
$ sudo apt-get update
// 安装以下任一一个安装包
$ sudo apt install --install-recommends winehq-stable   // 稳定的分支
$ sudo apt install --install-recommends winehq-devel    // 开发分支
$ sudo apt install --install-recommends winehq-staging  //Staging 分支
// 查看wine版本命令
$ wine --version
```
> 错误1：“E: 仓库 “http://ppa.launchpad.net/ubuntu-wine/ppa/ubuntu bionic Release” 没有 Release 文件。”  Ubuntu18.04LTS的版本代号为`bionic`，
> 这里报错我把这里`bionic`修改为Ubuntu16.04LTS的版本代号`xenial`
> 
>错误2：''W: GPG 错误：http://dl.winehq.org/wine-builds/ubuntu xenial InRelease: 由于没有公钥，无法验证下列签名： NO_PUBKEY 76F1A20FF987672F
E: 仓库 “http://dl.winehq.org/wine-builds/ubuntu xenial InRelease” 没有数字签名"
>执行命令`$ sudo apt-key adv --recv-keys --keyserver keyserver.Ubuntu.com F987672F`

## Step3  安装[Linux微信web开发者工具](<https://github.com/cytle/wechat_web_devtools>)

```shell
$ git clone https://github.com/cytle/wechat_web_devtools.git
$ cd wechat_web_devtools
// 自动下载最新 `nw.js` , 同时部署目录 `~/.config/wechat_web_devtools/`
$ ./bin/wxdt install
// 启动
$ ./bin/wxdt
```

![](https://i.loli.net/2019/05/15/5cdb8297a445c94962.png)

## Step4 创建一个启动图标

在 /usr/shared/applications/ 目录下，添加 xxx.desktop 文件（xxx为应用程序名），填写相关信息，保存即可。

例如，Icon=xxx ,Exec=xxx 替换为你的相应目录即可。

```shell
#!/usr/bin/env xdg-open

[Desktop Entry]
Encoding=UTF-8
Type=Application
Categories=Development;
Icon=/home/xmaihh/Application/wechat_web_devtools/images/we_dev.png
Exec=/home/xmaihh/Application/wechat_web_devtools/bin/wxdt                      
Name=WeChat Web develop tools
Name[zh_CN]=微信Web开发工具
MimeType=

```

![](https://i.loli.net/2019/05/15/5cdb892f6e8e153613.png)

附上[微信Web开发工具icon128x128](https://i.loli.net/2019/05/15/5cdb836d0232a43197.png)

