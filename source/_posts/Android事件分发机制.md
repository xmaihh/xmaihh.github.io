---
title: Android事件分发机制
date: 2018-10-12 10:11:54
categories: Android Framework
tags: [Android]
toc: true
description: Clear thinking requires courage rather than intelligence.
---
# 要点
1. -> dispatchTouchEvent()
   -> onInterceptTouchEvent()
   -> onTouchEvent()
2. requestDisallowInterceptTouchEvent(boolean)
3. onTouchEvent() --> onTouchListener --> onClickListener的先后顺序
# 什么是事件分发
Android中的视图是由一个个View嵌套构成的层级视图，即一个View里包含有子View，而这个子View里面又可以再添加View。当用户触摸屏幕产生一系列事件时，事件会由高到低，由外向内依次传递，最终把事件交到一个具体的View手上处理，这个传递的过程就叫做事件分发。

> Android将触摸事件统一封装成MontionEvent类，以Down事件开始，Up事件结束。
# 事件传递流程
Android中事件传递是由最终的view的接收到,传递过程是从父布局到子布局,也就是从Activity到ViewGroup到View的过程。

View里,有两个回调函数
```
public boolean dispatchTouchEvent(MotionEvent ev);  
public boolean onTouchEvent(MotionEvent ev);
```
ViewGroup里,有三个回调函数
```
public boolean dispatchTouchEvent(MotionEvent ev); 
public boolean onInterceptTouchEvent(MotionEvent ev);  
public boolean onTouchEvent(MotionEvent ev);
```
Activity里，有两个回调函数
```
public boolean dispatchTouchEvent(MotionEvent ev);  
public boolean onTouchEvent(MotionEvent ev);  
```
> `dispatchTouchEvent`是处理触摸事件分发,事件(多数情况)是从Activity的`dispatchTouchEvent`开始的。执行`super.dispatchTouchEvent(ev)`,事件向下分发。`onInterceptTouchEvent`是ViewGroup提供的方法,默认返回false,返回true表示拦截。`onTouchEvent`是View中提供的方法,ViewGroup也有这个方法,view中不提供onInterceptTouchEvent。view中默认返回true,表示消费了这个事件.

![Android事件分发](https://github.com/xmaihh/xmaihh.github.io/raw/master/asset/Android-dispatch-touch-event.jpg)

(1) 事件从 Activity.dispatchTouchEvent()开始传递，只要没有被停止或拦截，从最上层的 View(ViewGroup)开始一直往下(子 View)传递。子 View 可以通过 onTouchEvent()对事件进行处理。

(2) 事件由父 View(ViewGroup)传递给子 View，ViewGroup 可以通过 onInterceptTouchEvent()对事件做拦截，停止其往下传递。

(3) 如果事件从上往下传递过程中一直没有被停止，且最底层子 View 没有消费事件，事件会反向往上传递，这时父 View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到 Activity 的 onTouchEvent()函数。

(4) 如果 View 没有对 ACTION_DOWN 进行消费，之后的其他事件不会传递过来。

(5) OnTouchListener 优先于 onTouchEvent()对事件进行消费。
上面的消费即表示相应函数返回值为 true。
# requestDisallowInterceptTouchEvent(boolean)
对于底层的View来说，有一种方法可以阻止父层的View截获touch事件,就是调用`getParent().requestDisallowInterceptTouchEvent(true);`方法。
```
@Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                Log.d("TAG", "down");
                break;
            case MotionEvent.ACTION_UP:
                Log.d("TAG", "up");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.d("TAG", "move");
        }
        return true;
    }
```
一旦底层View收到onTouchEvent的action后调用`getParent().requestDisallowInterceptTouchEvent(true)`那么父层View就不会再调用onInterceptTouchEvent了,典型应用就是在处理滑动冲突的时候使用。
# onTouchEvent() --> onTouchListener --> onClickListener的先后顺序
View的dispatchTouchEvent()方法，意味将准备开始处理事件
```
public boolean dispatchTouchEvent(MontionEvent event){
    //.....
    ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnTouchListener != null
            && (mViewFlags & ENABLED_MASK) == ENABLED
            && li.mOnTouchListener.onTouch(this, event)) {
        result = true;
    }

    if (!result && onTouchEvent(event)) {
        result = true;
    }
}
```
如果我们给View设置了onTouchListener监听器，则优先会回调Listener的onTouch()方法。如果onTouch()方法返回了false,则还是会执行onTouchEvent()方法。通常我们给View设置的onClickListener，就是在onTouchEvent()方法中的Up事件处理的。所以onTouchListener优先级大于onClickListener
```
switch (action) {
    case MotionEvent.ACTION_UP:
    
    if (!post(mPerformClick)) {
    //该方法里会回调onClick()
    performClick();
    
    }
}
```
View的CLICKABLE属性要为ture，View才能消费上事件。不然onTouchEvent()会执行结束返回false，没有机会消费事件。当我们给View设置监听器后，就会将CLICKABLE属性设为true。(Button默认为ture)
```
public boolean onTouchEvent(MotionEvent event) {

    final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
            || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
            || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
    //View,setEnable()后还是能处理事件。如果我们有给View设置监听器，该事件被消费。
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
        return clickable;
    }
    if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
    
        switch (action) {
            case MotionEvent.ACTION_UP:
            
            case MotionEvent.ACTION_DOWN:
            
            case MotionEvent.ACTION_MOVE:
        }
        return true;
    }
    return false;
}
```
所以按照 onTouchEvent() --> onTouchListener --> onClickListener的先后顺序来执行的。