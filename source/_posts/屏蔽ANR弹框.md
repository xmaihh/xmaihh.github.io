---
title: 屏蔽ANR弹框
date: 2018-07-03 15:14:24
categories: Android Framework
tags: [Android]
toc: false
description: If you are honest to oneself , no one in the world can deceive you. 
---
代码位置:

frameworks\base\services\core\java\com\android\server\am中的ActivityManagerService

修改位置
```
// private boolean mShowDialogs = true;
  private boolean mShowDialogs = false;
  
  // TODO: If our config change, should we auto dismiss any 
  // showing dialogs?
  
  //del 20180703 start
  // mShowDialogs = shouldShowDialogs(newConfig);
  //del 20180703   end
  
  AttributeCache ac = AttributeCache.instance();
  if (ac != null) {
        ac.updateConfiguration(configCopy);
    }
 
@Override
public void handleMessage(Message msg) {
    switch (msg.what) {
    case SHOW_ERROR_MSG: {
        HashMap<String, Object> data = (HashMap<String, Object>) msg.obj;
        boolean showBackground = Settings.Secure.getInt(mContext.getContentResolver(),
                Settings.Secure.ANR_SHOW_BACKGROUND, 0) != 0;
        synchronized (ActivityManagerService.this) {
            ProcessRecord proc = (ProcessRecord)data.get("app");
            AppErrorResult res = (AppErrorResult) data.get("result");
            if (proc != null && proc.crashDialog != null) {
                Slog.e(TAG, "App already has crash dialog: " + proc);
                if (res != null) {
                    res.set(0);
                }
                return;
            }
            if (!showBackground && UserHandle.getAppId(proc.uid)
                    >= Process.FIRST_APPLICATION_UID && proc.userId != mCurrentUserId
                    && proc.pid != MY_PID) {
                Slog.w(TAG, "Skipping crash dialog of " + proc + ": background");
                if (res != null) {
                    res.set(0);
                }
                return;
            }
            // if (mShowDialogs && !mSleeping && !mShuttingDown)  // 兼容性开关的一个控制SystemProperties.getBoolean("mstar.app.compatibility.enable", false)
            if (mShowDialogs && !mSleeping && !mShuttingDown&&SystemProperties.getBoolean("mstar.app.compatibility.enable", false)) {
                Dialog d = new AppErrorDialog(mContext,
                        ActivityManagerService.this, res, proc);
                d.show();
                proc.crashDialog = d;
            } else {
                // The device is asleep, so just pretend that the user
                // saw a crash dialog and hit "force quit".
                if (res != null) {
                    res.set(0);
                }
            }
        }
        ensureBootCompleted();
    } break;
    }
```