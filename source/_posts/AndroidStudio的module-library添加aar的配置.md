---
title: AndroidStudio的module library添加aar的配置
date: 2018-11-20 09:25:29
categories: Android
tags: [AndroidStudio,Android]
toc: false
description: Everyone has talent. What is rare is the courage to follow the talent to the dark place where it leads.
---
# 使用aar的步骤
1. 在app的build.gradle中加入配置
一般来说,对`/项目工程/app/build.gradle`加入配置
```
android{
    ...

    repositories {
        flatDir {
            dirs 'libs'   // aar目录
          }
    }

...
}
```
2. 将aar文件拷贝到`app/libs`目录下 (e.g. aar文件为 xxxx-release.aar)
3. 在app的dependencies中加入aar引用
```
implementation(name: 'xxxx-release', ext: 'aar')
```
重新编译即可使用新加入的aar文件了
# 在module library使用aar的步骤
1. 在`module library`的build.gradle中加入配置
```
apply plugin: 'com.android.library'
android{
  repositories {
        flatDir {
            dirs 'libs','../module_library/libs
        }
    }
}
 
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation(name: 'xxxx-release', ext: 'aar')
    ...
}
```
这里主要是配置路径：这个是相对项目下的路径，一定要配上/module_library/libs,否则会由于路径不对找不到对应的aar

2. 将aar文件拷贝到`module_library/libs`目录下 (e.g. aar文件为 xxxx-release.aar)
3. 在`module library`的dependencies中加入aar引用
```
implementation(name: 'xxxx-release', ext: 'aar')
```
> 
如果觉得写绝对路径比较复杂，可以更简单点,在Top-level build.gradle定义
```
ext{
   MODULE_DIR_PATH = projectDir.getPath() +  "/module_library/libs"
}
```
那么依赖可以写成
```
repositories {
        flatDir {
            dirs 'libs',  rootProject.ext.MODULE_DIR_PATH
        }
    }
```