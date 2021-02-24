---
title: Ubuntu18.04.2LTS使用Gdrive同步文件、自动备份Hexo源文件
date: 2019-03-28 10:16:08
categories: Linux
tags: [Ubuntu,Gdrive]
toc: true
description: Every jar can have a lid.
---

[Gdrive项目地址](https://github.com/gdrive-org/gdrive)

Gdrive是一个命令行操作Google云端硬盘账户的操作工具

# 准备工作

- [Google Drive账号]([https://drive.google.com](https://drive.google.com/))
- Ubuntu18.04.2LTS

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux `sudo`
- `$` -要求给定的linux命令作为常规非特权用户执行

# 安装Gdrive

该工具的官方[GitHub页面 ](https://github.com/gdrive-org/gdrive)，并下载系统的可执行文件
```shell
$ wget -O ~/gdrive  "https://docs.google.com/uc?id=0B3X9GlR6EmbnQ0FtZmJJUXEyRTA&export=download"   ##gdrive-linux-x64

$ chmod +x ~/gdrive    ##添加可执行权限

$ sudo ln -s  ~/gdrive/gdrive   /usr/bin/gdrive		##建立软连接
```
# 授权Gdrive
为确保该工具能连接到Google云端硬盘账户，需要执行`gdrive about`进行授权,之后会在用户目录下生成`.gdrive`文件夹产生配置文件,不需要记得删除文件夹

```shell
$ gdrive about
```
![gdrive about](https://i.loli.net/2019/03/28/5c9c3438ba922.png)
把这串网址粘贴到浏览器并登入账号
![](https://i.loli.net/2019/03/28/5c9c35bd3ccc3.png)
点击`允许`会返回一串代码,复制
![](https://i.loli.net/2019/03/28/5c9c36267f5e0.png)
回到终端,将复制的代码粘贴上去
![](https://i.loli.net/2019/03/28/5c9c370f31f63.png)

# 使用Gdrive

常用命令如下，更多查看gdrive官网：[gdrive](https://github.com/gdrive-org/gdrive)

- 列出Google Drive根目录下文件、文件夹
```shell
$ gdrive list
```

- 下载Google Drive根目录下的文件、文件夹到本地
```shell
$ gdrive download {文件(夹)名}
```

- 本地文件上传到Google Drive根目录下
```shell
$ gdrive upload {文件(夹)名}
```

本地文件上传到指定目录
```shell
$ gdrive upload --parent {Dir id} {文件(夹)名}
```

- 在Google Drive根目录下创建文件夹
```shell
$ gdrive mkdir {文件夹名}
```

> :Dir id可以通过`gdrive list`查看

![](https://i.loli.net/2019/03/28/5c9c3af7f3e20.png)

# 附上自动备份Hexo源文件脚本

按需要修改自己所要备份的文件
```shell
#!/bin/sh
##---------------Folder you want to backup----------------##
NameOfFolder="hexo"
SourceOffFolder="/home/xmaihh/Documents"
BackupLocation="/home/xmaihh/backups"
date=$(date +"%Y-%m-%d")
LOG_FILE="home/xmaihh/googledrive.log"
##-----That mean,you will Backup the folder /home/xmaihh/Documents/hexo and will save into Folder /home/xmaihh/backups

if [ ! -d $BackupLocation ]; then
mkdir -p $BackupLocation
fi
find $BackupLocation/*.zip -mtime +10 -exec rm {} \;
for fd in $NameOfFolder; do
# Name of the Backup File
file=$fd-$date.zip

# Zip the Folder you will want to Backup
echo "Starting to zip the folder and files"
cd $SourceOffFolder
zip -r $BackupLocation/$file $fd
sleep 5s
## Process Upload Files to Google Drive
gdrive upload $BackupLocation/$file
if test $? = 0
then
echo "Your Data Successfully Uploaded to the Google Drive!"
else
echo "Error in Your Data Upload to Google Drive" > $LOG_FILE
fi
done
```

# Reference

[Gdrive：Linux下同步Google Drive文件、自动备份网站到Google Drive](https://lighti.me/1532.html)

