---
title: 定制vibrator震动强度
date: 2018-07-25 16:39:34
categories: Android Framework
tags: [Android ]
toc: false
description: I'm looking forward to the rest of my life every time at the thought of spending the rest life with you. 
---
HapticFeedback震动反馈提到过/frameworks/base/policy/src/com/android/internal/policy/impl/PhoneWindowManager.java
performHapticFeedbackLw()函数默认的震动值由 如mVirtualKeyVibePattern = getLongIntArray(mContext.getResources(),
com.android.internal.R.array.config_virtualKeyVibePattern)
振动时间的配置文件在
frameworks/base/core/res/res/values/config.xml
```
  <!-- 长按振动 -->
    <!-- Vibrator pattern for feedback about a long screen/key press -->
    <integer-array name="config_longPressVibePattern">
        <item>0</item>    //暂停时间  单位为 ms
        <item>1</item>    //震动时间  单位为 ms
        <item>20</item>    //暂停时间  单位为 ms
        <item>21</item>    //震动时间  单位为 ms
    </integer-array>
 
    <!-- 虚拟按键振动 -->
    <!-- Vibrator pattern for feedback about touching a virtual key -->
    <integer-array name="config_virtualKeyVibePattern">
        <item>0</item>
        <item>10</item>
        <item>20</item>
        <item>30</item>
    </integer-array>
 
    <!-- 软键盘按键振动 -->
    <!-- Vibrator pattern for a very short but reliable vibration for soft keyboard tap -->
    <integer-array name="config_keyboardTapVibePattern">
        <item>40</item>
    </integer-array>
 
    <!-- 非安全模式启动振动 -->  //正常开机震动
    <!-- Vibrator pattern for feedback about booting with safe mode disabled -->
    <integer-array name="config_safeModeDisabledVibePattern">
        <item>0</item>
        <item>1</item>
        <item>20</item>
        <item>21</item>
    </integer-array>
 
    <!-- 安全模式启动振动 -->  //安全模式开机振动
    <!-- Vibrator pattern for feedback about booting with safe mode disabled -->
    <integer-array name="config_safeModeEnabledVibePattern">
        <item>0</item>
        <item>1</item>
        <item>20</item>
        <item>21</item>
        <item>500</item>
        <item>600</item>
    </integer-array>
```
其中上面提到的500与600两个值无需修改，本身就是0的也无需修改。
微震对应的数值是16-17（这个非常舒适，强烈推荐）
不震对应的数值就是0。
就拿按键震动举个例子吧：将上面一段修改为以下微震
```
<integer-array name="config_virtualKeyVibePattern">  //按键的震动（就是那三个小点的震动）
    <item>0</item>
    <item>1</item>
    <item>16</item>
    <item>17</item>
</integer-array>
```
要修改的改好后，回编译就ok了