---
title: >-
  On Lollipop USBDevice object does not return the correct number of
  USBInterface
date: 2018-07-22 16:43:05
categories: Android Framework
tags: [Android]
toc: false
description: This is where all dreams begin.——THE BFG 
---
https://issuetracker.google.com/issues/37032033

此处修改
```
diff --git a/frameworks/base/core/java/android/hardware/usb/UsbDevice.java b/frameworks/base/core/java/android/hardware/usb/UsbDevice.java

header 1 | header 2
---|---
row 1 col 1 | row 1 col 2
row 2 col 1 | row 2 col 2

index d90e06e..c693af1 100644
--- a/core/java/android/hardware/usb/UsbDevice.java
+++ b/core/java/android/hardware/usb/UsbDevice.java
@@ -283,7 +283,7 @@ public class UsbDevice implements Parcelable {
             String manufacturerName = in.readString();
             String productName = in.readString();
             String serialNumber = in.readString();
-            Parcelable[] configurations = in.readParcelableArray(UsbInterface.class.getClassLoader());
+            Parcelable[] configurations = in.readParcelableArray(UsbConfiguration.class.getClassLoader());
             UsbDevice device = new UsbDevice(name, vendorId, productId, clasz, subClass, protocol,
                                  manufacturerName, productName, serialNumber);
             device.setConfigurations(configurations);
```
此处修改
```
Interestingly, I downloaded the USBHostManager.java file from Android 6.0 and it is identical the 5.1.1 version I was working with except the changes shown below.

In Summary (add the two lines below that have "+") to endUsbDeviceAdded():
/frameworks/base/services/usb/java/com/android/server/usb/UsbHostManager.java

    /* Called from JNI in monitorUsbHostBus() to finish adding a new device */
    private void endUsbDeviceAdded() {
        if (DEBUG) {
            Slog.d(TAG, "usb:UsbHostManager.endUsbDeviceAdded()");
        }
        if (mNewInterface != null) {
            mNewInterface.setEndpoints(
                    mNewEndpoints.toArray(new UsbEndpoint[mNewEndpoints.size()]));
        }
        if (mNewConfiguration != null) {
            mNewConfiguration.setInterfaces(
                    mNewInterfaces.toArray(new UsbInterface[mNewInterfaces.size()]));
        }
        synchronized (mLock) {
            if (mNewDevice != null) {
                mNewDevice.setConfigurations(
                        mNewConfigurations.toArray(new UsbConfiguration[mNewConfigurations.size()]));
                mDevices.put(mNewDevice.getDeviceName(), mNewDevice);
                Slog.d(TAG, "Added device " + mNewDevice);
                getCurrentSettings().deviceAttached(mNewDevice);
                mUsbAudioManager.deviceAdded(mNewDevice);
            } else {
                Slog.e(TAG, "mNewDevice is null in endUsbDeviceAdded");
            }
            mNewDevice = null;
            mNewConfigurations = null;
            mNewInterfaces = null;
            mNewEndpoints = null;
+           mNewConfiguration = null;
+           mNewInterface = null;
        }
    }
```

<html>
<!--issue-->
https://issuetracker.google.com/issues/37032363
</html>
