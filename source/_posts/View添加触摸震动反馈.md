---
title: View添加触摸震动反馈
date: 2018-06-29 20:20:09
categories: Android Framework
tags: [Android]
toc: false
description: Success is waking up in the morning, so excited about what you have to do. —Fame 
---
所有的View设置可点击震动

代码路径
```
/frameworks/base/core/java/android/view/View.java
```

1. 导包
```
import android.os.Vibrator;
import android.provider.Settings;
```

2. 声明变量
```
private Vibrator mVibrator;
```

3. 获取服务
在View的构造方法里获取
```
    /**
     * Simple constructor to use when creating a view from code.
     *
     * @param context The Context the view is running in, through which it can
     *        access the current theme, resources, etc.
     */
    public View(Context context) {
        mContext = context;
        mResources = context != null ? context.getResources() : null;
        mViewFlags = SOUND_EFFECTS_ENABLED | HAPTIC_FEEDBACK_ENABLED;
        // Set some flags defaults
        mPrivateFlags2 =
                (LAYOUT_DIRECTION_DEFAULT << PFLAG2_LAYOUT_DIRECTION_MASK_SHIFT) |
                (TEXT_DIRECTION_DEFAULT << PFLAG2_TEXT_DIRECTION_MASK_SHIFT) |
                (PFLAG2_TEXT_DIRECTION_RESOLVED_DEFAULT) |
                (TEXT_ALIGNMENT_DEFAULT << PFLAG2_TEXT_ALIGNMENT_MASK_SHIFT) |
                (PFLAG2_TEXT_ALIGNMENT_RESOLVED_DEFAULT) |
                (IMPORTANT_FOR_ACCESSIBILITY_DEFAULT << PFLAG2_IMPORTANT_FOR_ACCESSIBILITY_SHIFT);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
        setOverScrollMode(OVER_SCROLL_IF_CONTENT_SCROLLS);
        mUserPaddingStart = UNDEFINED_PADDING;
        mUserPaddingEnd = UNDEFINED_PADDING;
        mRenderNode = RenderNode.create(getClass().getName(), this);

        if (!sCompatibilityDone && context != null) {
            final int targetSdkVersion = context.getApplicationInfo().targetSdkVersion;

            // Older apps may need this compatibility hack for measurement.
            sUseBrokenMakeMeasureSpec = targetSdkVersion <= JELLY_BEAN_MR1;

            // Older apps expect onMeasure() to always be called on a layout pass, regardless
            // of whether a layout was requested on that View.
            sIgnoreMeasureCache = targetSdkVersion < KITKAT;

            sCompatibilityDone = true;
		}
			/** add for view 触摸时震动 start **/
			 mVibrator = (Vibrator) mContext.getSystemService(Context.VIBRATOR_SERVICE);
			// end add
    }

```

4. 触发震动
在playSoundEffect() 函数触发
```
    /**
     * Play a sound effect for this view.
     *
     * <p>The framework will play sound effects for some built in actions, such as
     * clicking, but you may wish to play these effects in your widget,
     * for instance, for internal navigation.
     *
     * <p>The sound effect will only be played if sound effects are enabled by the user, and
     * {@link #isSoundEffectsEnabled()} is true.
     *
     * @param soundConstant One of the constants defined in {@link SoundEffectConstants}
     */
    public void playSoundEffect(int soundConstant) {
        if (mAttachInfo == null || mAttachInfo.mRootCallbacks == null || !isSoundEffectsEnabled()) {
            return;
        }
		/** add for view 触摸时震动 start **/
       boolean vibratorEnabled = Settings.System.getInt(mContext.getContentResolver(),Settings.System.HAPTIC_FEEDBACK_ENABLED,1)==1;
       if(mVibrator !=null && vibratorEnabled ){
               try {
                   mVibrator.vibrate(50);
               } catch (Exception e) {
                   android.util.Log.v("View","Exception"+e.toString());
               }
       }
       // end add

        mAttachInfo.mRootCallbacks.playSoundEffect(soundConstant);
    }
```
