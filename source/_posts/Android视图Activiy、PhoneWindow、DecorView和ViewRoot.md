---
title: Android视图Activiy、PhoneWindow、DecorView和ViewRoot
date: 2018-10-06 11:33:02
categories: Android Framework
tags: [Android]
toc: false
description: Success is the sum of small efforts, repeated day in and day out.
---
## 概念简介
- Activity : 控制生命周期和处理事件
- Window : 视图承载器
- DecorView : Android视图树的根节点视图,顶级View
- ViewRoot : 执行或传递所有View的绘制以及事件分发等交互
![Android视图结构](https://github.com/xmaihh/xmaihh.github.io/raw/master/asset/Activity-PhoneWindo-DecorView-ViewRoot.webp)
![](https://i.loli.net/2018/12/17/5c16fe0012986.png)
Activity并不负责视图控制，它只是控制生命周期和处理事件，真正控制视图的是Window。一个Activity包含了一个Window，Window才是真正代表一个窗口，也就是说Activity没有Window，那就相当于是Service了。
Activity和Window是通过ActivityThread调用Activity的attach()函数联系起来的
```
//[window]:通过PolicyManager创建window,实现callback函数,所以,当window接收到
//外界状态改变时,会调用activity的方法,
final void attach(Context context, ActivityThread aThread,
        Instrumentation instr, IBinder token, int ident,
        Application application, Intent intent, ActivityInfo info,
        CharSequence title, Activity parent, String id,
        NonConfigurationInstances lastNonConfigurationInstances,
        Configuration config, String referrer, IVoiceInteractor voiceInteractor) {
    ....
    mWindow = PolicyManager.makeNewWindow(this);
    //当window接收系统发送给它的IO输入事件时,例如键盘和触摸屏事件,就可以转发给相应的Activity
    mWindow.setCallback(this);
    .....
    //设置本地窗口管理器
    mWindow.setWindowManager(
            (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
            mToken, mComponent.flattenToString(),
            (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
    .....
}
```
Window是视图的承载器，内部持有一个 DecorView，而这个DecorView才是 view 的根布局。Window是一个抽象类，实际在Activity中持有的是其子类PhoneWindow。PhoneWindow中有个内部类DecorView，通过创建DecorView来加载Activity中设置的布局setContentView(R.layout.activity_main)
Window 通过WindowManager将DecorView加载其中，并将DecorView交给ViewRoot，进行视图绘制以及其他交互

DecorView是FrameLayout的子类，它可以被认为是Android视图树的根节点视图。DecorView作为顶级View，一般情况下它内部包含一个竖直方向的LinearLayout，在这个LinearLayout里面有上下三个部分，上面是个ViewStub,延迟加载的视图（根据Theme设置设置ActionBar），中间的是标题栏(根据Theme设置，有的布局没有)，下面的是内容栏。具体情况和Android版本及Theme有关

ViewRoot对应ViewRootImpl类，它是连接WindowManagerService和DecorView的纽带，View的三大流程(测量（measure），布局（layout），绘制（draw）)均通过ViewRoot来完成。ViewRoot并不属于View树的一份子。从源码实现上来看，它既非View的子类，也非View的父类，但是，它实现了ViewParent接口，这让它可以作为View的名义上的父视图。ViewRoot继承了Handler类，可以接收事件并分发，Android的所有触屏事件、按键事件、界面刷新等事件都是通过ViewRoot进行分发的

## 视图创建过程
接着跟着源码分析界面的显示过程来引入Activity中Window的创建,以及View的加载显示过程
在ActivityThread.performLaunchActivity中,创建Activity的实例,接着会调用Activity.attach()来初始化一些内容,而Window对象就是在attach里进行创建初始化赋值的
Activity.attach:
```
final void attach（...） { 
    ...    
    mWindow = new PhoneWindow(this);    
    mWindow.setWindowManager((WindowManager)context.getSystemService(Context.WINDOW_SERVICE), 
        mToken, mComponent.flattenToString(),
        (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);   
    if (mParent != null) {            
        mWindow.setContainer(mParent.getWindow());        
    }
    mWindowManager = mWindow.getWindowManager();    
    ...
}
```
Window.setWindowManager:
```
public void setWindowManager(WindowManager wm, IBinder appToken, String appName,boolean hardwareAccelerated) { 
    ...    
    mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    ...
}
```
Activity里新建一个PhoneWindow对象。在Android中,Window是个抽象的概念,Android中Window的具体实现类是PhoneWindow,Activity和Dialog中的Window对象都是PhoneWindow。

同时得到一个WindowManager对象,WindowManager是一个抽象类,这个WindowManager的具体实现实在WindowManagerImpl中,对比Context和ContextImpl。

每个Activity会有一个WindowManager对象,这个mWindowManager就是和WindowManagerService(WMS)进行通信,也是WMS识别View具体属于那个Activity的关键,创建时传入IBinder 类型的mToken。
```
mWindow.setWindowManager(...,mToken, ...,...)
```
这个Activity的mToken,这个mToken是一个IBinder,WMS就是通过这个IBinder来管理Activity里的View

接着在onCreate的setContentView中,

Activity.setContentView():
```
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}
```
PhoneWindow.setContentView():
```
public void setContentView(int layoutResID) {
    ...
    installDecor();
    ...
}
```
PhoneWindow.installDecor:
```
private void installDecor() {
//根据不同的Theme,创建不同的DecorView,DecorView是一个FrameLayout 
}
```
此时只是创建了PhoneWindow,和DecorView,但目前二者也没有任何关系,产生利息的时刻是在ActivityThread.performResumeActivity中,再调用r.activity.performResume()，调用r.activity.makeVisible,将DecorView添加到当前的Window上
```
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
```
WindowManager的addView的具体实现在WindowManagerImpl中,而WindowManagerImpl的addView又会调用WindowManagerGlobal.addView。

WindowManagerGlobal.addView:
```
public void addView(View view, ViewGroup.LayoutParams params,Display display, Window parentWindow) {
    ...
    ViewRootImpl root = new ViewRootImpl(view.getContext(), display);
    view.setLayoutParams(wparams);
    mViews.add(view);
    mRoots.add(root);
    mParams.add(wparams);
    root.setView(view, wparams, panelParentView);
    ...
}
```
这个过程创建一个ViewRootImpl,并将之前创建的DecoView作为参数闯入,以后DecoView的事件都由ViewRootImpl来管理了,比如DecoView上添加View,删除View。ViewRootImpl实现了ViewParent这个接口,这个接口最常见的一个方法是requestLayout()

ViewRootImpl是个ViewParent，在DecoView添加的View时,就会将View中的ViewParent设为DecoView所在的ViewRootImpl,View的ViewParent相同时,理解为这些View在一个View链上。所以每当调用View的requestLayout()时,其实是调用到ViewRootImpl，ViewRootImpl会控制整个事件的流程。可以看出一个ViewRootImpl对添加到DecoView的所有View进行事件管理