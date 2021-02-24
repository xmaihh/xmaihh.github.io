---
title: Ubuntu18.04.2LTS修改主文件夾名稱爲英文
date: 2019-03-17 15:26:04
categories: Linux
tags: [Ubuntu]
toc: true
description: Reason why we have two ears and only one mouth is that we may listen the more and talk the less.
---

# Ubuntu18.04.2LTS修改主文件夾名稱爲英文

## 方法1：

先重命名中文文件夾，然後

编辑~/.config/user-dirs.dirs文件

```
$ vim ~/.config/user-dirs.dirs
```

修改文件內容爲：

```
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
```

## 方法2：

打开终端，在终端中输入命令:

```
$ export LANG=en_US
$ xdg-user-dirs-gtk-update
```

跳出对话框询问是否将目录转化为英文路径,同意并关闭。
在终端中输入命令:

```
$ export LANG=zh_CN
```

重新启动系统，系统会提示更新文件名称，选择不再提示,并取消修改。