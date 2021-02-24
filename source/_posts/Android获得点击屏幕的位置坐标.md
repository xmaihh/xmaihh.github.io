---
title: Android获得点击屏幕的位置坐标
date: 2018-12-17 11:38:38
categories: Android Framework
tags: [Android,ADB]
toc: true
description: Love all, trust a few, do wrong to none.
---
# 开发者选项获得点击屏幕的位置坐标

在手机开发者选项中,打开指针位置,可以在屏幕上方获取当前点击位置的坐标点(x,y)

```shell
P:1/1  X:553  Y:1851  Xv:0:0  Yv:0:0 Prs:1.0  Size:0.13
```

 <img src="https://i.loli.net/2018/12/18/5c18b910782fd.jpg" width = "300" alt="Android开发者选项_指针位置" align=center />

 命令行窗口输入：`adb shell input tap 553 1851`实现点击效果

# 通过`adb shell getevent`命令获得点击屏幕的位置坐标

1. 命令行窗口输入:
```shell 
adb shell getevent -p | grep -e "0035" -e "0036"
```

获得event 体系里 width宽（0035）和height高（0036）

![](https://i.loli.net/2018/12/18/5c18bd9839f38.png)
![](https://i.loli.net/2018/12/18/5c18bf79433b8.png)
2. 计算比例 
用到其中的max值
```shell
0035（width） max 1024
0036（height） max 600
```
跟手机屏幕的分辨率比较，获取手机分辨率在命令行窗口输入:
```shell
adb shell wm size 
```
![](https://i.loli.net/2018/12/18/5c18c406093bf.png)

即手机分辨率是: 1280(width) x 600(height)

计算比例

```shell
rateW = 1280(手机屏幕的宽) / 1024(event里0035的max) = 1.25
rateH = 600(手机屏幕的高) / 600(event里0036的max) = 1
```
3. 点击屏幕计算点击位置的坐标

在命令行窗口输入:

```shell
adb shell getevent | grep -e "0035" -e "0036"
```
> 注意：这里的命令`adb shell getevent`不带`-p`参数

例如,点击屏幕一个位置得到输出:

```shell
/dev/input/event0: 0003 0035 00000280
/dev/input/event0: 0003 0036 0000018c
```

把0035和0036后面的位置数据从16进制转化为10进制

```shell
width = 0x280 = 2*16*16 + 8*16 = 640
height = 0x8ec = 1*16*16 + 8*16 + 12 = 396
```
这是在event体系里的位置，将其转化为屏幕位置

```shell
screenW = width*rateW = 640*1.25 = 800
screenH = height*rateH = 396*1 = 396
```
最终算出来了
刚刚点击的屏幕位置坐标就是（800, 396）

# 代码TouchEvent里面的位置直接就是你在屏幕上的点击位置

```Java
//    ACTION_DOWN:用户按下屏幕的事件
//    ACTION_MOVE:用户滑动的时间
//    ACTION_UP:用户手指从按下状态抬起屏幕的时间
//
//    getAction方法：得到操作时间的类型
//    getDwonTime方法：得到用户按下的时间
//    getEventTime方法：得到用户操作的时间
//    getPressure方法：得到用户的触摸压力值

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        if (MotionEvent.ACTION_DOWN == event!!.action) {
            var x = event.x
            var y = event.y
            tv!!.text = "您点击的位置是:\nx: " + x + "\ny: " + y
        }
        return super.onTouchEvent(event)
    }
```
![](https://i.loli.net/2018/12/18/5c18c98c3685a.png)
详细代码
[https://github.com/xmaihh/KotlinTrainingThings/releases/tag/v1.0](https://github.com/xmaihh/KotlinTrainingThings/releases/tag/v1.0)