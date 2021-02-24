---
title: HapticFeedback震动反馈
date: 2018-06-24 13:59:23
categories: Android Framework
tags: [Android,ADB]
toc: true
description: In the day of prosperity be joyful, but in the day of adversity consider. 
---
## adb测试震动 ##
```
root@a # busybox find -name "vibrator"
root@a # busybox find -name "vibrator"
	./sys/devices/virtual/timed_output/vibrator
	./sys/class/timed_output/vibrator
```
向enable文件写入成功，就立即震动，震动的持续时间即是写入的值，单位为ms
```
root@a # echo '10000'> /sys/class/timed_output/vibrator/enable  
root@a # cat /sys/class/timed_output/vibrator/enable  
	3290  
```
读enable文件来获得震动剩余的时间
```
root@a # echo ‘0’> /sys/class/timed_output/vibrator/enable  

```

## 按键触摸反馈 ##

### 代码路径

```
/frameworks/base/policy/src/com/android/internal/policy/impl/PhoneWindowManager.java
```

interceptKeyBeforeQueueing函数中添加如下代码：

```
if(down&&(keyCode == KeyEvent.KEYCODE_HOME)
	||(keyCode == KeyEvent.KEYCODE_MENU)||(keyCode == KeyEvent.KEYCODE_BACK)){
	    performHapticFeedbackLw(null, HapticFeedbackConstants.VIRTUAL_KEY, false);
} 
```

### performHapticFeedbackLw()函数

```
    @Override
    public boolean performHapticFeedbackLw(WindowState win, int effectId, boolean always) {
        if (!mVibrator.hasVibrator()) {
            return false;
        }
        final boolean hapticsDisabled = Settings.System.getIntForUser(mContext.getContentResolver(),
                Settings.Settings.System.HAPTIC_FEEDBACK_ENABLED.HAPTIC_FEEDBACK_ENABLED, 0, UserHandle.USER_CURRENT) == 0;
        if (hapticsDisabled && !always) {
            return false;
        } Settings.System.HAPTIC_FEEDBACK_ENABLED
        long[] pattern = null;
        switch (effectId) {
            case HapticFeedbackConstants.LONG_PRESS:
                pattern = mLongPressVibePattern;
                break;
            case HapticFeedbackConstants.VIRTUAL_KEY:
                pattern = mVirtualKeyVibePattern;
                break;
            case HapticFeedbackConstants.KEYBOARD_TAP:
                pattern = mKeyboardTapVibePattern;
                break;
            case HapticFeedbackConstants.CLOCK_TICK:
                pattern = mClockTickVibePattern;
                break;
            case HapticFeedbackConstants.CALENDAR_DATE:
                pattern = mCalendarDateVibePattern;
                break;
            case HapticFeedbackConstants.SAFE_MODE_DISABLED:
                pattern = mSafeModeDisabledVibePattern;
                break;
            case HapticFeedbackConstants.SAFE_MODE_ENABLED:
                pattern = mSafeModeEnabledVibePattern;
                break;
            default:
                return false;
        }
        int owningUid;
        String owningPackage;
        if (win != null) {
            owningUid = win.getOwningUid();
            owningPackage = win.getOwningPackage();
        } else {
            owningUid = android.os.Process.myUid();
            owningPackage = mContext.getOpPackageName();
        }
        if (pattern.length == 1) {
            // One-shot vibration
            mVibrator.vibrate(owningUid, owningPackage, pattern[0], VIBRATION_ATTRIBUTES);
        } else {
            // Pattern vibration
            mVibrator.vibrate(owningUid, owningPackage, pattern, -1, VIBRATION_ATTRIBUTES);
        }
        return true;
    }
```

如震动值由 mVirtualKeyVibePattern = getLongIntArray(mContext.getResources(),
                com.android.internal.R.array.config_virtualKeyVibePattern);获得。
配置文件在

```
frameworks/base/core/res/res/values/config.xml
    <!-- Vibrator pattern for feedback about touching a virtual key -->
    <integer-array name="config_virtualKeyVibePattern">
        <item>0</item>
        <item>10</item>
        <item>20</item>
        <item>30</item>
    </integer-array>
...
..
.
```

## View长按触发震动 ##

### 1.对一个button注册长按监听
长按button震动
```
 Button click= (Button) findViewById(R.id.click);
 click.setOnLongClickListener(new View.OnLongClickListener() {
     @Override
     public boolean onLongClick(View v) {

         Toast.makeText(MainActivity.this,"长按点击",Toast.LENGTH_SHORT).show();

         //触发震动反馈
         return true;
         //return false;
     }
 });
```

### 2.View.setOnLongClickListener源码

```
/**
     * Register a callback to be invoked when this view is clicked and held. If this view is not
     * long clickable, it becomes long clickable.
     *
     * @param l The callback that will run
     *
     * @see #setLongClickable(boolean)
     */
    public void setOnLongClickListener(@Nullable OnLongClickListener l) {
        if (!isLongClickable()) {
            setLongClickable(true);
        }
        getListenerInfo().mOnLongClickListener = l;
    }
```

### 3.View.performLongClick源码

```
/**
     * Call this view's OnLongClickListener, if it is defined. Invokes the context menu if the
     * OnLongClickListener did not consume the event.
     *
     * @return True if one of the above receivers consumed the event, false otherwise.
     */
    public boolean performLongClick() {
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_LONG_CLICKED);

        boolean handled = false;
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnLongClickListener != null) {
            handled = li.mOnLongClickListener.onLongClick(View.this);
        }
        if (!handled) {
            handled = showContextMenu();
        }
        if (handled) {
			//  触发震动反馈
            performHapticFeedback(HapticFeedbackConstants.LONG_PRESS);
        }
        return handled;
    }
```

### 4.View.performHapticFeedback(int feedbackConstant, int flags)源码

```
/**
     * BZZZTT!!1!
     *
     * <p>Like {@link #performHapticFeedback(int)}, with additional options.
     *
     * @param feedbackConstant One of the constants defined in
     * {@link HapticFeedbackConstants}
     * @param flags Additional flags as per {@link HapticFeedbackConstants}.
     */
    public boolean performHapticFeedback(int feedbackConstant, int flags) {
        if (mAttachInfo == null) {
            return false;
        }
        //noinspection SimplifiableIfStatement
        if ((flags & HapticFeedbackConstants.FLAG_IGNORE_VIEW_SETTING) == 0
                && !isHapticFeedbackEnabled()) {
            return false;
        }
        return mAttachInfo.mRootCallbacks.performHapticFeedback(feedbackConstant,
                (flags & HapticFeedbackConstants.FLAG_IGNORE_GLOBAL_SETTING) != 0);
    }
```

### 5.HapticFeedbackConstants的3个常量

```
HapticFeedbackConstants.LONG_PRESS
HapticFeedbackConstants.FLAG_IGNORE_VIEW_SETTING  //可以无视android:hapticFeedbackEnabled=”false”，会触发震动
HapticFeedbackConstants.FLAG_IGNORE_GLOBAL_SETTING //可以无视 Settings.System.HAPTIC_FEEDBACK_ENABLED=0，会触发震动
```

### 6.HapticFeedbackConstants使用方法

```
<Button
        android:layout_width="wrap_content"
        android:id="@+id/click"
        android:layout_height="wrap_content"
        android:hapticFeedbackEnabled="false"
        android:text="make" />
...
click.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // v.performHapticFeedback(HapticFeedbackConstants.LONG_PRESS);
				// v.performHapticFeedback(HapticFeedbackConstants.LONG_PRESS,HapticFeedbackConstants.FLAG_IGNORE_VIEW_SETTING);
                v.performHapticFeedback(HapticFeedbackConstants.LONG_PRESS,HapticFeedbackConstants.FLAG_IGNORE_GLOBAL_SETTING
                        );
            }
        });
```

## 代码介绍 ##

### APP层

获取服务后，调用vibrate()方法，实现震动

```
    mVibrator= (Vibrator) getSystemService(VIBRATOR_SERVICE);
    mVibrator.vibrate(50);   //填入震动时长 :ms
```

### Framework层

framework层代码

```
    frameworks/base/services/java/com/android/server/VibratorService.java
    frameworks/base/core/java/android/os/IVibratorService.aidl
    frameworks/base/core/java/android/os/Vibrator.java
    frameworks/base/core/java/android/os/SystemVibrator.java
```

VibratorService.java
    有SystemService拉起的服务。实现IVibratorService.aidl的接口，从而实现VibratorService;它的函数接口，是通过调用JNI层对应的马达控制函数来实现的。

Vibrator.java
    是VibratorService开放给应用层的调用类。Vibrator是抽象类。它便于我们支持不同类型的马达。

SystemVibrator.java
    它是Vibrator.java的子类，实现了vibration的服务接口。
    在构造函数中，通过 IVibratorService.Stub.asInterface(ServiceManager.getService("vibrator")) 获取马达服务，实际上获取的是VibratorService对象。
    SystemVibrator的接口都是调用VibratorService接口实现的

### JNI及HAL层

代码路径

```
frameworks/base/services/jni/com_android_server_VibratorService.cpp
    hardware/libhardware_legacy/vibrator/vibrator.c
```

通过vibrator中sendit函数将对vibrator的控制写进“/sys/class/timed_output/vibrator/enable”节点中。
