---
title: 'Program type already present: com.alibaba.android.arouter.routes.ARouter'
date: 2018-11-07 19:37:46
categories: Android
tags: [Android]
toc: false
description: Everyone has talent. What is rare is the courage to follow the talent to the dark place where it leads.
---
今天在写东西的时候报了一个错误，这个是使用 alibaba 的路由框架 [ARouter](https://github.com/alibaba/ARouter)，进行模块间通信报才错。
```
Program type already present: com.alibaba.android.arouter.routes.ARouter
```
意思是 Arouter 配置的路径的组路径已经存在了，举一个栗子：

我们在中配置模块 A 中 A1 类，可以配置路径为：
```
public final static String REGIST_A_A1 = "/A/A1";
```
这时候如果我们要配置模块 B 中的类，就不能使用 A 来作为组的路径了，要不然就会报错。在不同的模块中，配置的组路径不能一样，而在同一个模块中，自己的路径不能相同，就是上面 A1 位置不能相同。 