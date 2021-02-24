---
title: Launcher中Camera图标的显示与隐藏
date: 2018-07-29 16:43:04
categories: Android Framework
tags: [Android ,Camera]
toc: false
description: Well, life isn't always what one likes, isn't it?
---
在camera模块的源码中我们发现DisableCameraReceiver的这样一个类，是继承自BroadcastReceiver一个广播接收器，在AndroidManifest.xml中发现这个reciver的intent-filter为`<action android:name="android.intent.action.BOOT_COMPLETED" />`，当系统启动之后，该模块就会收到这条系统广播，然后进行相应的处理
代码路径:packages/apps/Camera2/src/com/android/camera/DisableCameraReceiver.java
```
    @Override
    public void onReceive(Context context, Intent intent) {
        // Disable camera-related activities if there is no camera.
        boolean needCameraActivity = CHECK_BACK_CAMERA_ONLY
            ? hasBackCamera()
            : hasCamera();

        if (!needCameraActivity) {
            Log.i(TAG, "disable all camera activities");
            for (int i = 0; i < ACTIVITIES.length; i++) {
                disableComponent(context, ACTIVITIES[i]);
            }
        }

        // Disable this receiver so it won't run again.
        disableComponent(context, "com.android.camera.DisableCameraReceiver");
    }
```
通过needCameraActivity的值来判断是否需要隐藏camera应用,如果其值为true，则显示camera应用，否则，就隐藏掉
```
    private boolean hasCamera() {
        int n = android.hardware.Camera.getNumberOfCameras();
        Log.i(TAG, "number of camera: " + n);
        return (n > 0);
    }

    private boolean hasBackCamera() {
        int n = android.hardware.Camera.getNumberOfCameras();
        CameraInfo info = new CameraInfo();
        for (int i = 0; i < n; i++) {
            android.hardware.Camera.getCameraInfo(i, info);
            if (info.facing == CameraInfo.CAMERA_FACING_BACK) {
                Log.i(TAG, "back camera found: " + i);
                return true;
            }
        }
        Log.i(TAG, "no back camera");
        return false;
    }
```
通过`frameworks/base/core/java/android/hardware/Camera.java`中的getNumberOfCameras()方法来获取camera的个数,这是一个本地方法，是通过JNI调用来获取的
```
  /**
     * Returns the number of physical cameras available on this device.
     *
     * @return total number of accessible camera devices, or 0 if there are no
     *   cameras or an error was encountered enumerating them.
     */
    public native static int getNumberOfCameras();
```
最后在`frameworks/av/camera/CameraBase.cpp`对getNumberOfCameras()函数实现
```
template <typename TCam, typename TCamTraits>
int CameraBase<TCam, TCamTraits>::getNumberOfCameras() {
    const sp<ICameraService> cs = getCameraService();

    if (!cs.get()) {
        // as required by the public Java APIs
        return 0;
    }
    return cs->getNumberOfCameras();
}
```
cs是CameraService的一个弱引用，找到`frameworks/av/services/camera/libcameraservice/CameraService.cpp`中的getNumberOfCameras()函数
```
int32_t CameraService::getNumberOfCameras() {
    return mNumberOfCameras;
}
```
mNumberOfCameras是一个全局变量,在CameraService.cpp中赋值
```
void CameraService::onFirstRef()
{
    LOG1("CameraService::onFirstRef");

    BnCameraService::onFirstRef();

    if (hw_get_module(CAMERA_HARDWARE_MODULE_ID,
                (const hw_module_t **)&mModule) < 0) {
        ALOGE("Could not load camera HAL module");
        mNumberOfCameras = 0;
    }
    else {
        ALOGI("Loaded \"%s\" camera module", mModule->common.name);
        mNumberOfCameras = mModule->get_number_of_cameras();
        if (mNumberOfCameras > MAX_CAMERAS) {
            ALOGE("Number of cameras(%d) > MAX_CAMERAS(%d).",
                    mNumberOfCameras, MAX_CAMERAS);
            mNumberOfCameras = MAX_CAMERAS;
        }
        for (int i = 0; i < mNumberOfCameras; i++) {
            setCameraFree(i);
        }

        if (mModule->common.module_api_version >=
                CAMERA_MODULE_API_VERSION_2_1) {
            mModule->set_callbacks(this);
        }

        VendorTagDescriptor::clearGlobalVendorTagDescriptor();

        if (mModule->common.module_api_version >= CAMERA_MODULE_API_VERSION_2_2) {
            setUpVendorTags();
        }

        CameraDeviceFactory::registerService(this);
    }
}
```