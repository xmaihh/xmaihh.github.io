---
title: Ubuntu16.04升级到18.04LTS记录
date: 2018-12-21 09:13:27
categories: Linux
tags: [Ubuntu]
toc: true
description: live well, love lots, and laugh often.
---
# 更新Ubuntu 16.04
在升级之前，先更新当前的16.04至最新状态。建议升级之前更新/升级所有已安装的软件包。

首先更新APT源和软件包至最新

```
sudo apt update && sudo apt dist-upgrade && sudo apt autoremove
```

# 安装和配置`Ubuntu update manager`
更新完组件后，运行以下命令安装`update-manager-core`

```
sudo apt install update-manager-core
```

打开`update-manager`配置文件

```
sudo vi /etc/update-manager/release-upgrades
```

>确保设置为Prompt=lts

# 执行升级命令
```
sudo do-release-upgrade -d
```
出现升级提示时，全部选择y

等待所有的软件包下载...安装...到重启... 

当所有操作执行完毕后，系统就升级到最新的Ubuntu 18.04 LTS版本了。
