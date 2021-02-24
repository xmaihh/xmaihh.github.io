---
title: Android Lottie动画的使用
date: 2019-04-10 13:46:25
categories: Android
tags: [Lottie,AndroidStudio]
toc: true
description: Finders keepers, losers weepers.
---

Lottie是一个用于Android，iOS，Web和Windows的库，用于解析使用[Bodymovin](https://github.com/airbnb/lottie-web)导出为json的[Adobe After Effects](http://www.adobe.com/products/aftereffects.html)动画，并在移动设备和网络上呈现它们！

![](https://i.loli.net/2019/04/10/5cad843949bf3.gif)

介绍下Android的使用

github地址 ： [lottie-android](https://github.com/airbnb/lottie-android)

官方文档：[airbnb.io/lottie](http://airbnb.io/lottie/)

动画json下载：https://lottiefiles.com

效果图：

![](https://i.loli.net/2019/04/10/5cad8590aecf0.gif)

![](https://i.loli.net/2019/04/10/5cad85cd323a1.gif)

![](https://i.loli.net/2019/04/10/5cad85a10a6dd.gif)

# Android项目使用

首先，在项目`build.grade`文件中引入依赖：

```shell
dependencies {
  implementation 'com.airbnb.android:lottie:$lottieVersion'
}
```

有2种使用方式：

直接在布局文件使用

```shell
<com.airbnb.lottie.LottieAnimationView
        android:id="@+id/animation_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        // 放在 res/raw 目录下的动画文件
        app:lottie_rawRes="@raw/hello_world"
        // 放在assets目录下的动画文件
        app:lottie_fileName="hello_world.json"
        // 开启循环
        app:lottie_loop="true"
        // 自动播放
        app:lottie_autoPlay="true" />
```

在Java代码中使用

```shell
LottieAnimationView animationView = findViewById(R.id.animation_view);
animationView.setAnimation(R.raw.hello_world);
// or
animationView.setAnimation(R.raw.hello_world.json);
animationView.playAnimation();
```

好的，让我们运行一下项目。

然后你就会发现奇迹出现了，没有一张图片，没有一个gif，但是动画效果出来了！就是这么简单，就是这么暴力！

## 常用方法

- LottieAnimationView.loop(true);
  设置动画循环演示。
- mLottieAnimationView.isAnimating();
  是否在演示中。
- mLottieAnimationView.setProgress(0.5f);
  设置演示的进度。
- mLottieAnimationView.getProgress();
  获取演示的进度。
- mLottieAnimationView.getDuration();
  获取演示的时间。
- mLottieAnimationView.playAnimation();
  运行动画。
- mLottieAnimationView.pauseAnimation();
  暂停动画。
- mLottieAnimationView.cancelAnimation();
  关闭动画。

# Reference

[Lottie- 让Android动画实现更简单](https://www.jianshu.com/p/cae606f45c0b)

