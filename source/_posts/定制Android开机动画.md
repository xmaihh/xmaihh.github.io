---
title: 定制Android开机动画
date: 2018-06-23 16:34:28
categories: Android Framework
tags: [Android]
toc: true
description: God made relatives.Thank God we can choose our friends. 
---

### 开机动画
替换 Android 设备 system/media/bootanimation.zip 文件
adb push bootanimation.zip /sdcard/bootanimation.zip
```
# adb shell
# su
# mount -o remount,rw /system
# cp /sdcard/bootanimation.zip /system/media/bootanimation.zip
# cd /system/media/
# chmod 0644 bootanimation.zip
```
### 制作开机动画包
解压 bootanimation.zip 文件你会发现，里面会有一个 desc.txt 文件和若干个 part0、part1 这样的目录。

现在我们查看 desc.txt 文件
```
720 1280 20
p 1 0 part0
P 0 0 part1
// 720 动画的宽度
// 1280 动画的高度
// 20 每秒播放20帧图片 （最好不要超过30）
// p 第二行和第三行的p表示2个part（出第一行外，通常是以p开头的）
// 1 对part中静态图片循环播放的次数。例如：part0的静态图片会播放2次，part1的静态图片只有正常的一次。
// 0 播放完当前part中的动画后，暂停的帧数。 （如该是40的话，40/20=2秒，即暂停2秒）
// part0 part1 存储静态图片的目录名称
```
注意：

1. desc.txt 文件要在 Linux 环境下生成，因为有些空格不一样

2. part 目录中的图片的命名要是连续的，比如pic_001, pic_002, _pic_003 …

3. 打包成bootanimation.zip文件的时候，要要用zip格式的存储方式打包。


### 系统编译
1. 把bootanimation.zip放到alps/frameworks/base/data/sounds/目录下
2. 修改/device/{vendor}/{project}/device.mk，增加：
PRODUCT_COPY_FILES += frameworks/base/data/sounds/bootanimation.zip:system/media/bootanimation.zip
3. 重新build系统，烧录机器即可；