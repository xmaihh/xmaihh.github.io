---
title: 解决Module里调用aar出现failed to resolve的问题
date: 2018-09-12 08:50:23
categories: Android
tags: [Android,AndroidStudio]
toc: false
description: Behind the fear of an ideal you, you create the fear, you can beat him.
---
环境：AndroidStudio 3.1.4
1. 在Module的build.gradle添加
在dependencies{}标签里
```
compile(name: '第三方aar库名称', ext: 'aar')
```
在android{}标签里
```
repositories {
    flatDir {
        dirs 'libs'
    }
}
```
2. 在App的build.gradle添加
在dependencies{}标签里
```
repositories {
    flatDir {
        dirs project(':你的Module名').file('libs')
    }
}
```
重新编译即可