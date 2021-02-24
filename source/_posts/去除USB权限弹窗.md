---
title: 去除USB权限弹窗
date: 2018-06-26 15:39:30
categories: Android Framework
tags: [Android]
toc: false
description: The world doesn't just disappear when you close your eyes. —Memento 
---
在访问一个插入到Android系统的USB设备的时候往往是需要权限的，此时系统会弹出询问权限的对话框，而我们此时希望让它默认允许访问USB设备并且不希望用户看到这个对话框。
我们在获取UsbManager和UsbDevice/UsbAcessory之后，首先需要检查是否对这个USB设备/附件有操作的权限，如果没有权限，则需要向系统申请（系统会弹出询问权限的对话框），此时需要注册一个广播接收器用来接收用户的选择。
```
// 检查是否有操作权限
boolean hasPermission = mUsbManager.hasPermission(mUsbDevice);
if (!hasPermission) {
    // 注册广播，接收用户权限选择
    PendingIntent pi = PendingIntent.getBroadcast(mContext, 0, new Intent(TAG_UsbPermission), 0);
    mContext.registerReceiver(new MyPermissionReceiver(), new IntentFilter(TAG_UsbPermission));
    // 弹出对话框，申请权限
    mUsbManager.requestPermission(mUsbDevice, pi);
}
```
下面是我们定义的广播接收器：
```
// 定义的广播接收器
private class MyPermissionReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals(TAG_UsbPermission)) {
            boolean granted = intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false);
            if (!granted) {
                // Todo：已经获取权限，可以执行其他操作
            } else {
                // Todo：未获取权限。
            }
        }
    }
}
```
#### 去除USB权限弹窗方法
在这个过程中，系统会弹出询问权限的对话框，而我们现在不希望用户看到这个界面。
进入系统源码，找到文件 
/frameworks/base/packages/SystemUI/src/com/android/systemui/usb/UsbPermissionActivity.java 
找到其中的 onCreate() 方法，替换
```
setupAlert();
```
为
```
mPermissionGranted = true; 
finish(); 
```
这样就不会弹窗了，并且会允许给设备操作权限。 
当然我们也可以指定只有我们自己的APP不需要弹窗，只需要加一层过滤条件即可：
```
// add permission for our packages! 
if(mPackageName.startsWith("com.xxx.xxx")) { 
    mPermissionGranted = true; 
    finish(); 
} else { 
    setupAlert();   
} 
```
当然也可以根据设备的VID、PID、设备名称等信息进行过滤（省略）。


// 更直接的方法
frameworks/base/core/res/res/values/config.xml
```
    <!-- If true, then we do not ask user for permission for apps to connect to USB devices.
         Do not set this to true for production devices. Doing so will cause you to fail CTS. -->
    <bool name="config_disableUsbPermissionDialogs">true</bool>
```