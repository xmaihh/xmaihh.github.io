---
title: >-
  Fragment异常：android.view.InflateException: Binary XML file line #7: Error
  inflating class fragment
date: 2019-02-22 14:33:25
categories: Android
tags: [Android]
toc: true
description: How many loved your moments of glad grace,and lo..
---

fragment是个很好的控件，但今天在静态使用fragment的时候，遇到个问题，错误信息如下:

# 错误信息

Caused by: android.view.InflateException: Binary XML file line #7: Error inflating class fragment
```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#80434343">

    <fragment
        android:id="@+id/fragment_float_window"
        android:name="com.xmaihh.example.ui.fragment.FloatWindowFragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</FrameLayout>
```

# 解决办法

- 1.xml布局中，fragment未设置id
- 2.嵌入的Activity要继承于FragmentActivity ，而不是Activity
- 3.xml中引入的包名不对,注意检查
 `android:name="com.xmaihh.example.ui.fragment.FloatWindowFragment"`

- 4.还有种可能就是你Fragment引入的包名不对，v4包下的还是系统的Fragment要统一好 注意`android.support.v4.app.FragmentActivity`和`android.app.Fragment`区别