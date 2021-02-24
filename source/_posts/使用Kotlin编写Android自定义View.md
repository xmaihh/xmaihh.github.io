---
title: 使用Kotlin编写Android自定义View
date: 2019-12-24 15:42:36
categories: Android
tags: [Android,Kotlin]
toc: false
description: 人生に必要な ものは、 勇気と想像力。 それと ほんの少しのお金だ。
---

# 使用 Kotlin 编写 Android 自定义 View

# 前言
由于 Kotlin 的构造函数与 Java 的构造函数在样子上十分不同, 导致使用 Kotlin 编写 Android 自定义 View 时会遇到一些困难.

在本篇中将记录我对此的一些学习心得.

# 构造函数
Kotlin 自定义 View 的构造函数写法有两种, 我们分别来看.

Android 的自定义 View 包含有多个构造函数, 如何用 Kotlin 实现呢?

# 写法一
第一种写法与 Java 构造函数类似, 代码如下:

```kotlin
class KotlinView : View {
 
    constructor(context: Context) : this(context, null)
    constructor(context: Context, attrs: AttributeSet?) : this(context, attrs, 0)
 
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr) {
        ...
    }
}
```

与使用 Java 的构造函数对比一下:

```java
public class JavaView extends View{

    public JavaView (Context context) {
        super(context);
    }

    public JavaView (Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public JavaView (Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
```

这样就很容易理解了.

# 写法二
另一种写法如下:

```kotlin
class CustomComponent : LinearLayout {
    @JvmOverloads
    constructor(
        context: Context, 
        attrs: AttributeSet? = null, 
        defStyleAttr: Int = 0)
        : super(context, attrs, defStyleAttr)

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    constructor(
       context: Context, 
       attrs: AttributeSet?, 
       defStyleAttr: Int, 
       defStyleRes: Int)
        : super(context, attrs, defStyleAttr, defStyleRes)
}
```

其中, 这里使用了 Kotlin 构造函数的参数可以指定默认值的特性. `@JvmOverloads` 是一个 Kotlin 注解, 作用是替换构造函数的默认值生成多个版本的重载构造方法.

# inflate 视图

有一个常见的场景是, 我们在自定义视图中需要 inflate 一个 xml 布局, 在 Kotlin 中的做法是写在 init 中:

```kotlin
init {
    LayoutInflater
        .from(context)
        .inflate(R.layout.view_custom_component, this, true)
}
```

# 解析自定义 View 属性

自定义 View 往往会声明一些属性, 用于在 xml 中设置.

在这里, 关于如何 declare-styleable 如何写 xml 这些基础内容我们都省略, 只看与 Kotlin 相关的关键步骤, 如何解析 StyledAttributes, 代码如下:

```kotlin
attrs?.let {
    val typedArray = context.obtainStyledAttributes(it, 
        R.styleable.custom_component_attributes, 0, 0)
    val title = resources.getText(typedArray
            .getResourceId(R.styleable
            .custom_component_attributes_custom_component_title,           
            R.string.component_one))

    my_title.text = title
    my_edit.hint = 
        "${resources.getString(R.string.hint_text)} $title"

    typedArray.recycle()
}
```

需要注意的是, 对于写法一, 在 init {} 是拿不到 attrs, 因此需要写一个私有的初始化方法, 在各个构造函数中显式调用. 对于写法二, 直接在 init {} 中编写即可.

# Reference

[使用 Kotlin 编写 Android 自定义 View](https://steemit.com/kotlin/@maxiee/kotlin-android-view)

[Custom Views in Android with Kotlin (KAD 06)](https://antonioleiva.com/custom-views-android-kotlin/)

[Building Custom Component with Kotlin](https://android.jlelse.eu/building-custom-component-with-kotlin-fc082678b080)