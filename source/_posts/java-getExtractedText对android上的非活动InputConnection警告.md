---
title: java-getExtractedText对android上的非活动InputConnection警告
date: 2019-03-05 09:51:22
categories: Android
tags: [AndroidStudio,Android]
toc: true
description: How many loved your moments of glad grace,and loved your beauty with love false or true.
---

我在logcat中获得以下警告

```java
W/IInputConnectionWrapper: finishComposingText on inactive InputConnection
```

一直找不到背后的原因,网上找到一个类似的问题，看到如下的`logcat`

```java
W/IInputConnectionWrapper(21214): getTextBeforeCursor on inactive InputConnection
W/IInputConnectionWrapper(21214): getSelectedText on inactive InputConnection
W/IInputConnectionWrapper(21214): getTextBeforeCursor on inactive InputConnection
W/IInputConnectionWrapper(21214): getTextAfterCursor on inactive InputConnection
...
I/Choreographer(20010): Skipped 30 frames!  The application may be doing too much work on its main thread.
```

# 我的情况

我有一个EditText视图用户类型。当用户按下按钮时，EditText被清除。当我快速按下按钮时，很多非活动的InputConnection条目流出。

例如:

```java
editText.setText(null);
```

我的logcat中的最后一行提供了一个很好的指示，正在发生什么。果然，InputConnection被清除文本的请求所淹没。我试图修改代码以检查文本长度，然后再尝试清除它:

```java
if (editText.length() > 0) {
    editText.setText(null);
}
```

这有助于缓解快速按下按钮不再导致`IInputConnectionWrapper`警告流的问题。然而，当用户在键入东西和按下按钮之间快速交替时，或者当应用程序处于充分负载等时按下按钮，这仍然容易出现问题。

幸运的是，我发现了另一种方式来清除文本：`Editable.clear()`.有了这一点，我不会得到警告：

```java
if (editText.length() > 0) {
    editText.getText().clear();
}
```

注意，如果你想清除所有的输入状态，而不只是文本(自动文本，autocap，multitap，撤消)，你可以使用`TextKeyListener.clear(Editable e)`。

```java
if (editText.length() > 0) {
    TextKeyListener.clear(editText.getText());
}
```

# Reference

[android - W/IInputConnectionWrapper(1066)：showStatusIcon在非活动的InputConnection](https://codeday.me/bug/20180429/159033.html)
[java - 如何将活动上下文放入非活动类android？](https://codeday.me/bug/20190108/514667.html)
[android - 警告：导出的活动不需要权限](https://codeday.me/bug/20170829/64838.html)
[android-activity - 在非活动类中获取当前活动上下文](https://codeday.me/bug/20181213/451206.html)
[java - 从非活动单例类获取应用程序上下文](https://codeday.me/bug/20180508/160725.html)
[android - 在超时警告通知后检索活动](https://codeday.me/bug/20181007/280723.html)
[java - 使用非活动的startActivityForResult](https://codeday.me/bug/20170714/40764.html)
[Java PMD对非临时类成员的警告](https://codeday.me/bug/20180105/113304.html)

