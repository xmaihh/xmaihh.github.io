---
title: Android的四大启动模式
date: 2018-12-03 19:09:10
categories: Android Framework
tags: [Android]
toc: true
description:  Love all, trust a few, do wrong to none.
---
# 要点
1. standard：标准模式
2. singleTop：栈顶复用模式
3. singleTask：栈内复用模式
4. singleInstance：单一实例模式
# 启动模式
## standard：标准模式

系统默认模式。每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。在这个模式下，谁启动了Activity，那么这个Activity就运行在启动它的那个Activity所在栈中。

## singleTop：栈顶复用模式

在这种模式下，如果新的Activity已经位于任务栈顶，那么此Activity不会被重新创建，同时它的onNewIntent方法会被回调，通过此方法的参数我们可以取出当前的请求信息

## singleTask：栈内复用模式

这是一种单例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会创建实例，和singleTop是一样，系统也会调用onNewIntent。还有一点，就是singleTask有clearTop的效果，会导致栈内已有的Activity全部出栈。

## singleInstance：单一实例模式

这是一种加强的singleTask模式，它除了具有singleTask的所有特性以外，还加强了一点，那就是具有此模式的Activity只能单独位于一个任务栈中，比如Activity A是singleInstance模式，当A启动后，系统会为它创建一个新的任务栈，然后A独自在这个新的任务栈中，由于栈内复用的特性，后续均不会创建新的Activity，除非这个独特的任务栈被系统销毁。整个手机操作系统里面只有一个实例存在。不同的应用去打开这个activity 共享公用的同一个activity。他会运行在自己单独，独立的任务栈里面，并且任务栈里面只有他一个实例存在。

# 使用场景
## standard使用场景：
邮件客户端，在新建一个邮件的时候，适合新建一个新的实例

## singleTop使用场景：
消息推送，通知栏弹出Notification，点击Notification跳转到指定Activity，使用singleTop避免生成重复的页面。
登录的时候，登录成功跳转到主页，按下两次登录按钮，使用singleTask避免生成两个主页。
从activity A启动了个service进行耗时操作，或者某种监听，这个时候你home键了，service收集到信息，要返回activityA。

## singleTask使用场景：
提供给第三方应用调用的页面，做浏览器、微博之类的应用，浏览器的主界面等等。
程序的主界面，进入多层嵌套之后，一键退回，之前打开的Activity全部出栈。

## singleInstance使用场景：
呼叫来电界面，打电话、发短信功能。
闹铃提醒，将闹铃提醒与闹铃设置分离。
# 四种启动模式的区别
![Android启动模式](https://github.com/xmaihh/xmaihh.github.io/raw/master/asset/Android-launchMode.png)

# 启动模式的设置
## 在`AndroidMainifest`的Activity配置进行设置
```
<activity 
    android:name=".singletop.SingleTopActivity" 
    android:launchMode="singleTop"
    android:taskAffinity="com.xmaihh.demo.standard"/>
```
>taskAffinitys属性在默认情况下一个应用中的所有activity具有相同的taskAffinity，即应用程序的包名。我们可以通过设置不同的taskAffinity属性给应用中的activity分组，也可以把不同的应用中的activity的taskAffinity设置成相同的值。
## 通过Intent设置标志位
```
Intent inten = new Intent (ActivityA.this,ActivityB.class);
intent,addFlags(Intent,FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```
标志位属性
| 标记位属性 | 含义 |
| -- | --|
| FLAG_ACTIVITY_SINGLE_TOP | 指定启动模式为栈顶复用模式（SingleTop） |
| FLAG_ACTIVITY_NEW_TASK | 指定启动模式为栈内复用模式（SingleTask） |
| FLAG_ACTIVITY_CLEAR_TOP | 所有位于其上层的Activity都要移除，SingleTask模式默认具有此标记效果 |
| FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS | 具有该标记的Activity不会出现在历史Activity的列表中，即无法通过历史列表回到该Activity上 |

> 两种设置方式有区别
1.优先级不同 
Intent设置方式的优先级 > Manifest设置方式，即 以前者为准
2.限定范围不同 
Manifest设置方式无法设定 FLAG_ACTIVITY_CLEAR_TOP；Intent设置方式 无法设置单例模式（SingleInstance）