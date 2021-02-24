---
title: Ubuntu18.04.2LTS安装、配置、美化
date: 2019-03-14 14:10:54
categories: Linux
tags: [Ubuntu]
toc: true
description: Don't play your life that you are not good at for audiences who do not belong to you.
---

# Ubuntu18.04.2LTS安装、配置、美化

## 安装准备

- 准备[Ubuntu18.04镜像](http://releases.ubuntu.com/)
- 关闭Secure Boot

## 硬盘分区

硬盘一般分为IDE硬盘、SCSI硬盘和SATA硬盘三种。

在Linux系统中，IDE接口的硬盘被称为hd，SCSI和SATA接口的硬盘则被称为sd，其中IDE硬盘基本上已经淘汰，现在市面上最常见的就是SATA接口的硬盘，第1块硬盘称为sda，第2块硬盘称为sdb……，依此类推。

一块硬盘最多有4个主分区，主分区以外的分区称为[扩展分区](https://www.baidu.com/s?wd=%E6%89%A9%E5%B1%95%E5%88%86%E5%8C%BA&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，硬盘可以没有扩展分区，但是一定要有主分区，在主分区中要有一个激活分区用来启动Windows系统，在扩展分区中可以建立若干个逻辑分区，因此，最合理的分区方式应该最多分三个主分区，一个扩展分区，这样可以有效地利用有限的主分区，然后在扩展分区中建立逻辑分区。

在Linux系统中每一个硬盘总共最多有 16个分区，硬盘上的4个主分区，分别标识为sdal、sda2、sda3和sda4，逻辑分区则从sda5开始标识一直到sda16。

Linux可以把分区作为挂载点，载入目录，其中最常用的硬盘大小（500G-1000G）分配目录推荐如下表所示：

| 目录  | 建议大小     | 格式 | 描述                                                         |
| ----- | ------------ | ---- | ------------------------------------------------------------ |
| EFI	   | 100M         |  |一定要放在开头，主分区，分配32M以上
| /     | 150G-200G    | ext4 | 根目录                                                       |
| swap  | 物理内存两倍 | swap | 交换空间：交换分区相当于Windows中的“虚拟内存”，如果内存低的话（1-4G），物理内存的两倍，高点的话（8-16G）要么等于物理内存，要么物理内存+2g左右， |
| /boot | 1G左右       | ext4 | **空间起始位置** 分区格式为ext4 **/boot** **建议：应该大于400MB或1GB** Linux的内核及引导系统程序所需要的文件，比如 vmlinuz initrd.img文件都位于这个目录中。在一般情况下，GRUB或LILO系统引导管理器也位于这个目录；启动撞在文件存放位置，如kernels，initrd，grub。 |
| /tmp  | 5G左右       | ext4 | 系统的临时文件，一般系统重启不会被保存。（建立服务器需要？） |
| /home | 尽量大些     | ext4 | 用户工作目录；个人配置文件，如个人环境变量等；所有账号分配一个工作目录。 |

## 修改DNS

### Step1:添加Google’s DNS

`vim /etc/systemd/resolved.conf`
在文件中添加內容：

```
DNS=8.8.8.8 2001:4860:4860::8888
FallbackDNS=8.8.4.4 2001:4860:4860::8844
```



### Step2:重啓網絡或者重啓電腦

## 更换root密码

[![更换root密码](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313142321.png)](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313142321.png)更换root密码更换root密码

```
xmaihh@xmaihh-H81M-S1:~$ sudo passwd root
输入新的 UNIX 密码： 
重新输入新的 UNIX 密码： 
passwd：已成功更新密码
xmaihh@xmaihh-H81M-S1:~$
```

## sudo免密码

shell输入:

```
xmaihh@xmaihh-H81M-S1:~$ sudo visudo
```

显示如下:

```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:$

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
xmaihh  ALL=(ALL) NOPASSWD: ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```

> 只要在**%sudo ALL=(ALL:ALL) ALL**下面添加一行username  ALL=(ALL) NOPASSWD: ALL

## 更新源

找到Software & Updates，将源更新为阿里云的源 或者其他国内的源

然后自己手动更新一下:

```
sudo apt update
sudo apt upgrade
```

## Sougou Pinyin

```
sudo apt-get install fcitx-bin      #安装fcitx-bin
sudo apt-get update --fix-missing   #修复fcitx-bin安装失败的情况
sudo apt-get install fcitx-bin      #重新安装fcitx-bin
sudo apt-get install fcitx-table    #安装fcitx-table
```

然后去[搜狗输入法Linux官网](https://pinyin.sogou.com/linux/)下载64bit的deb包程序，如：sogoupinyin_2.2.0.0108_amd64.deb

```
sudo dpkg -i sogoupinyin*.deb       #安装搜狗拼音
sudo apt-get install -f             #修复搜狗拼音安装的错误
sudo dpkg -i sogoupinyin*.deb       #重新安装搜狗拼音
```

重启！重启！重启！也就是注销当前用户再重登的事

## WPS

去[wps_linux官网](https://www.wps.com/download/)下载64bit的deb包程序，如：[wps-office_10.1.0.6757_amd64.deb](http://kdl.cc.ksosoft.com/wps-community/download/6757/wps-office_10.1.0.6757_amd64.deb)

```
sudo dpkg -i libpng12-0*.deb      #安装依赖libpng12-0
sudo dpkg -i wps*.deb			  #安装wps
sudo apt-get install -f           #若出现错误没有安装成功,用来修复
```

下载[wps字体](https://drive.google.com/open?id=17bFkK8VvJ9MHnPGTAiDJlPf2mX_ItAer),然后解压
(或者[链接](https://pan.baidu.com/s/1xBd0XIdB2tqDiDR7zABrSg)提取码:ea2d)

```
sudo mkdir /usr/share/fonts/WPS-Fonts       #新建wps字体存储文件夹
cd ~/Downloads     #进入下载好的字体目录
sudo apt-get install unzip  #安装zip解压软件
sudo unzip wps_symbol_fonts.zip -d /usr/share/fonts/WPS-Fonts/  #解压字体到指定文件夹
sudo mkfontscale    #生成字体索引
sudo mkfontdir      #生成字体索引
sudo fc-cache       #更新字体缓存
```

## 压缩软件

```
sudo apt-get install p7zip-full p7zip-rar rar unzip
```

## Google Chrome

```
wget -q -O - https://raw.githubusercontent.com/longhr/ubuntu1604hub/master/linux_signing_key.pub | sudo apt-key add
sudo sh -c 'echo "deb [ arch=amd64 ] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
sudo apt-get update
sudo apt-get install google-chrome-stable
```

## vim

```
$ sudo apt-get install vim
```

## 安裝VS Code

首先下载官方安装包：
<https://code.visualstudio.com/docs/?dv=linux64_deb>
然后在该文件路径运行以下命令：

```
sudo dpkg -i code_1.24.1-1528912196_amd64.deb
```



或者双击安装包，要安装依赖的话先安装依赖，如果双击安装无反应，可以在命令行中运行安装，然后安装所需依赖即可

```
sudo apt-get install -f
```



## 多版本gcc和g++共存

```
sudo apt-get install gcc-5 gcc-5-multilib
sudo apt-get install g++-5 g++-5-multilib
sudo apt-get install gcc-6 gcc-6-multilib
sudo apt-get install g++-6 g++-6-multilib
sudo apt-get install gcc-7 gcc-7-multilib
sudo apt-get install g++-7 g++-7-multilib
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 60
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 50
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 60
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 70
```

然后选择gcc和g++版本

```
sudo update-alternatives --config gcc
sudo update-alternatives --config g++
```

## 多版本python和pip共存

ubuntu18.04自带python3，但是没有python2，pip2，pip3。

```
sudo apt install python2.7  #安装python2.7
sudo apt install python-minimal
sudo apt install curl
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo apt install python-testresources   #防止pip2出错
sudo apt install python3-testresources  #防止pip3出错
sudo python3 get-pip.py #安装pip3
sudo python2 get-pip.py #安装pip3
sudo pip3 install --upgrade pip #升级pip3
sudo pip2 install --upgrade pip #升级pip2
```

此时pip和python并不知道指向2还是3，需要自己修改。我们使用alias来设置别名。我要让pip和python都指向3

```
$ whereis pip
pip: /usr/local/bin/pip /usr/local/bin/pip3.6 /usr/local/bin/pip2.7
$ whereis python
python: /usr/bin/python2.7 /usr/bin/python2.7-config /usr/bin/python3.6m /usr/bin/python3.6 /usr/bin/python3.6m-config /usr/bin/python3.6-config /usr/bin/python /usr/lib/python2.7 /usr/lib/python3.7 /usr/lib/python3.6 /etc/python2.7 /etc/python3.6 /etc/python /usr/local/lib/python2.7 /usr/local/lib/python3.6 /usr/include/python2.7 /usr/include/python3.6m /usr/include/python3.6 /usr/share/python /usr/share/man/man1/python.1.gz
```

> error: Traceback (most recent call last):
  File "setup.py", line 1, in <module>
    from distutils.core import setup
ImportError: No module named distutils.core
报错缺少`python-distutils`包，安装即可。
```shell
 $ sudo apt-get install python3-distutils
```

可见pip3在：

```
/usr/local/bin/pip3.6
```

python在：

```
/usr/bin/python3.6
```

自定义alias别名：

```
vim ~/.bashrc
```

打开文件后，在最后一行加：

```
alias pip=/usr/local/bin/pip3.6
alias python=/usr/bin/python3.6
```

然后更新环境：

```
source ~/.bashrc
```

## 支持exfat

```
sudo apt-get install exfat-fuse exfat-utils
```

## 音视频

### 安装FFmpeg

```
sudo add-apt-repository ppa:djcj/hybrid
sudo apt-get update
sudo apt-get install ffmpeg
```

### 安装解码器

```
sudo apt-get install ubuntu-restricted-extras
```

### 安装VLC视频播放器

```
sudo apt-get install vlc browser-plugin-vlc
```

### 安装录制gif软件peek

```
sudo add-apt-repository ppa:peek-developers/stable
sudo apt-get update
sudo apt-get install peek
```

## 美化

### 查看gnome版本

gnome3版本以下

```
$ gnome-panel --version   或者  gnome-about --gnome-version
```

gnome3版本以上

```
$ gnome-session --version  或者 gnome-shell --version
```

如我的gnome3版本如下：

```
xmaihh@xmaihh-H81M-S1:~$ gnome-shell --version
GNOME Shell 3.28.3
```

### 使用Tweaks对gnome美化

Ubuntu 18.04 LTS 内置的是 gnome 桌面环境,安装一些主题、图标美化一下整个系统。再用几个插件增强一下效果和使用体验即可。

```
sudo apt-get install gnome-tweak-tool   #安装tweak
sudo apt-get install gnome-shell-extensions -y  #安装shell扩展
sudo apt install chrome-gnome-shell     #为了能在浏览器内安装gnome插件，火狐和谷歌都能用
sudo apt-get install gtk2-engines-pixbuf    #防止GTK2错误
sudo apt install libxml2-utils
```

gnome桌面环境主题、图标 下载地址：
<https://www.gnome-look.org/>
以下是我使用配置：
[仿macOS主题Ant](https://github.com/EliverLara/Ant)
[![Ant](https://i.loli.net/2019/03/13/5c88ce2486591.png)](https://i.loli.net/2019/03/13/5c88ce2486591.png)AntAnt
安装使用配置: `Ctrl+Alt+T 打开terminal，执行如下命令

```
$ sudo rm /var/lib/apt/lists/lock
$ sudo rm /var/cache/apt/archives/lock
$ sudo apt-get update
$ sudo apt-get install gnome-tweak-tool
$ mkdir .themes
$ cd ~/.themes/
$ git clone https://github.com/EliverLara/Ant.git
```

主题已经安装完成了，你可以打开 Tweaks – Appearance – Applications ，找到你刚下载下来的主题并一键使用。
[![Ant主题使用](https://i.loli.net/2019/03/13/5c88cf808feb5.png)](https://i.loli.net/2019/03/13/5c88cf808feb5.png)Ant主题使用Ant主题使用
[仿macOS主题扁平化图标 La Capitaine](https://github.com/keeferrourke/la-capitaine-icon-theme)
[![La Capitaine](https://i.loli.net/2019/03/13/5c88d29138933.png)](https://i.loli.net/2019/03/13/5c88d29138933.png)La CapitaineLa Capitaine

同样，`Ctrl+Alt+T 打开terminal，执行如下命令

```
mkdir ~/.icons
$ cd ~/.icons/
$ git clone https://github.com/keeferrourke/la-capitaine-icon-theme.git
```

OK，图标包安装完成，直接打开 Tweaks – Appearance – Icons ，选择使用即可。

[仿macOS主题dask栏](https://github.com/micheleg/dash-to-dock)

[![dash-to-dock](https://i.loli.net/2019/03/13/5c88d25f1c902.png)](https://i.loli.net/2019/03/13/5c88d25f1c902.png)dash-to-dockdash-to-dock

这是一个dask栏插件

```
$ cd ~
$ mkdir .temp
$ cd .temp/
$ wget https://github.com/micheleg/dash-to-dock/archive/master.zip
$ cd master/
$ make
$ make install
```

还没结束！现在进入重点部分，在键盘按下 **Alt+F2 键**，在弹出的窗口中输入字母 `r`。

嗯，现在才正式安装完插件。

[![Tweaks tool的界面](https://i.loli.net/2019/03/13/5c88d3dc523e6.png)](https://i.loli.net/2019/03/13/5c88d3dc523e6.png)Tweaks tool的界面Tweaks tool的界面

### Dash to dock Settings

[![Dash to dock Settings](https://i.loli.net/2019/03/14/5c89a64d80b0c.png)](https://i.loli.net/2019/03/14/5c89a64d80b0c.png)Dash to dock SettingsDash to dock Settings

#### 问题：

1. 我在安装的时候遇到Ubuntu18.04.2LTS自带dock栏与dash to dock冲突,输入以下命令将自带dock移动到～下，重启后即可解决此问题(也可移动到其他目录或者直接rm删除)。**Ubuntu 更新后需要再执行一遍**，因为更新会修复自带的 dock。

```
sudo mv /usr/share/gnome-shell/extensions/ubuntu-dock@ubuntu.com ~/
或者
sudo rm -rf /usr/share/gnome-shell/extensions/ubuntu-dock@ubuntu.com
```

1. 前面说到查看gnome版本，[dash-to-dock](https://github.com/micheleg/dash-to-dock)有对应的`branch`，`git clone`拉取时加上`-b`参数拉取对应版本分支

## 聊天软件

- [Telegram](https://telegram.org/)

- Wechat Work& Foxmail

  使用[deepin-wine-ubuntu](https://github.com/wszqkzqk/deepin-wine-ubuntu) 移植的企业微信 和Foxmail

其他deepin-wine容器：[阿里云镜像下载](http://mirrors.aliyun.com/deepin/pool/non-free/d/)

安装使用：

打开terminal,执行下列命令

```
git clone https://github.com/wszqkzqk/deepin-wine-ubuntu.git
```

cd到deepin-wine-for-ubuntu文件夹下面，执行下列命令

```
./install.sh
```

在home目录下新建一个文件夹,我命名的是softwares，然后cd进入softwares，执行如下命令:

```
wget http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.weixin.work/deepin.com.weixin.work_2.4.16.1347deepin0_i386.deb
```

在softwares目录下,执行以下命令

```
sudo dpkg -i deepin.com.weixin.work_2.4.16.1347deepin0_i386.deb
```

## 网速和CPU使用率工具

打开terminal,执行下列命令

```
sudo add-apt-repository ppa:fossfreedom/indicator-sysmonitor
sudo apt-get update
sudo apt-get install indicator-sysmonitor
```

接着执行命令

```
$ ndicator-sysmonitor &
```

然后Ctrl+C就可以实现后台运行indicator-sysmonitor

[![运行效果](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313142855.png)](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313142855.png)运行效果运行效果

设置开机启动

[![设置开机启动](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313143941.png)](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313143941.png)设置开机启动设置开机启动

[![参数配置](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313144129.png)](https://raw.githubusercontent.com/xmaihh/piggo-token/master/img20190313144129.png)参数配置参数配置

## 安装npm

下载[Node.js](https://nodejs.org/zh-cn/download/)

接着执行以下命令

```
cd /usr/local/node/

wget https://npm.taobao.org/mirrors/node/v10.15.3/node-v10.15.3-linux-x64.tar.gz  #下载安装包

tar -zxvf node-v10.15.3-linux-x64.tar.gz  # 解压安装包

rm  node-v10.15.3-linux-x64.tar.gz  # 移除安装包

ln -s /usr/local/node/node-v10.15.3-linux-x64/bin/npm /usr/local/bin/npm

ln -s /usr/local/node/node-v10.15.3-linux-x64/bin/node /usr/local/bin/node
```

查看npm版本

```
xmaihh@xmaihh-H81M-S1:/usr/local/node$ npm -v
6.4.1
```

npm升级,@后面是版本号

```
npm i -g npm@6.4.1
```

## Git

在 Ubuntu 这类 Debian 体系的系统上，可以用 apt-get 安装：

```
sudo apt-get install git
```

配置信息

```
git config --global user.name "yourname" #引号里面输入你的名字
git config --global user.email "youremail" #输入邮箱
git config --global core.autocrlf false #消除由于Windows和Linux平台中换行符的差异导致的问题
git config --global core.quotepath off #消除由于路径或者是文件名包含中文导致的乱码问题
git config --global gui.encoding utf-8 #消除gui界面中文乱码问题(如果全程使用命令行的话不用担心这个问题)
ssh-keygen -t rsa -C "youremail" #配置ssh的密钥，输完之后一路回车
eval `ssh-agent` #启用ssh-agent
ssh-add ~/.ssh/id_rsa #添加密钥
ssh-add -l #将它添加到已知的key列表中
cat ~/.ssh/id_rsa.pub #把这个公钥添加到自己的Github账户上去
```

# 2019-7-30补充 卸载Sougou输入法

鉴于Sougou Pinyin输入法在gnome3桌面日常崩溃，每每查看 /var/crash/ 目录下崩溃日志都有，卸载了。

  $ sudo apt-get  purge  sogoupinyin     （卸载搜狗拼音输入法）
  $ sudo apt-get purge  fcitx     （卸载fcitx）
  $ sudo apt-get autoremove    （彻底卸载fcitx及相关配置）
注销重新登录一下或者重启。

新输入法
rime输入法

$ sudo apt install ibus-rime（安装rime输入法）

$ sudo apt install librime-data-wubi（安装五笔库）

$ sudo apt install librime-data-pinyin-simp （安装简体拼音库）
在 ~/.config/ibus/rime/ 下新建一个文件  default.custom.yaml （覆盖默认设置）

内容是：

patch:

       schema_list:
    
               - schema: wubi_pinyin
    
               - schema: pinyin_simp
    
               - schema: wubi86
说明：schema 是输入法顺序，如果仅用拼音或五笔，则将对应的项移到最前面，本人要五笔拼音一起混用，所以将wubi_pinyin放到了前面。

并且修改

wubi_pinyin.schema.yaml

switches下的reset 值由0改为1，意思是重启后默认由中文状态改为英文状态。

重启操作系统，使安装生效。

打开“setting（设置）”，“Region&Language（区域和语言）”，点+号，添加输入法 Chinese(Rime) 。如果不用其它输入法，可以删除，其实也真不用其它输入法了

注销重新登录一下或者重启。

# Reference

[Ubuntu18.04完整新手安装教程](https://blog.jackyu.club/jack/1-13/)

[Ubuntu 18.04 LTS 安装（踩坑）配置全记录](https://sspai.com/post/45791)

[安装Ubuntu Linux系统时硬盘分区最合理的方法](https://blog.csdn.net/u012052268/article/details/77145427)

[Ubuntu18.04安装后应该做的事](https://blog.csdn.net/hymanjack/article/details/80285400)

[Ubuntu 18.04 安装 ibus-rime 输入法](https://links.jianshu.com/go?to=%5Bhttps%3A%2F%2Fjingyan.baidu.com%2Farticle%2F4853e1e5b090a31909f7269c.html%5D(https%3A%2F%2Fjingyan.baidu.com%2Farticle%2F4853e1e5b090a31909f7269c.html))