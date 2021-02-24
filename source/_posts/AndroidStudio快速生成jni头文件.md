---
title: AndroidStudio快速生成jni头文件
date: 2019-07-31 18:33:13
categories: Android
tags: [AndroidStudio]
toc: true
description: What is the most resilient parasite? Bacteria? A virus? An intestinal worm? An idea.
---

依次打开`Settings`-->`Tools`-->`External Tools`-->`点击加号创建一个快速生成jni头文件的工具`

![](https://i.loli.net/2019/07/31/5d41618c4f9d413087.png)

```shell
1. Program: javah  
2. Parameters: -v -jni -d $ModuleFileDir$/src/main/cpp $FileClass$  
3. Working directory: $SourcepathEntry$  
```

选中包含native方法的 JAVA 文件，右键唤起菜单，依次找到`ExternalTools`=>`Jni create .h`，执行前面添加的生成.h头文件的工具。
![](https://i.loli.net/2019/07/31/5d41688405d3932421.jpg)
![](https://i.loli.net/2019/07/31/5d416983e1d3667068.png)