---
title: Android BLE蓝牙简介
date: 2018-09-02 15:34:22
categories: Android
tags: [BLE]
toc: true
description: People always say that it's too late. However, in fact, now is the best appropriate time. For a man who really wants to seek for something, every period of life is younger and timely. 
---
## 简介
android 从4.3系统开始可以连接BLE设备,iOS是从7.0版本开始支持BLE
android 从5.0系统开始可以模拟设备发出BLE广播,这个新功能是对标于iOS系统的手机模拟iBeacon设备
BLE设备之所以能被手机扫描到,是因为BLE设备在每隔一段时间广播一次,这个广播里面包含很多数据
并不是Android L的系统就可以支持BLE Peripheral，这个和硬件也是有关系的,谷歌从5.0系统SDK已经开始支持check手机是否支持BLE Peripheral
在Android 5.0 API 21 android.bluetooth.le包下，新增加 Scaner相关类和 Advertiser 相关类,大致结构如下：
- Android BLE 周边设备 （Peripheral）可以通过 Advertiser 相关类实现操作； 
- Android BLE 中心设备 （Central）可以通过 Scanner相关类实现蓝牙扫描； 
- Android BLE 建立中心连接的时候，使用 BlueToothDevice#connectGatt() 实现

## Central 和 Peripheral
### 蓝牙通信规则
Bluetooth BLE 的交互有两个角色 ： Central 与 Peripheral 。可以对比传统的CS结构，理解为客户端-服务端结构。
- 服务端 （外围设备）： Peripheral 通常具有其他设备所需要的数据，即数据提供服务；
- 客户端（中心设备）：Central 通常通过使用Perpheral 的信息来实现一些特定的功能 ;

### Central 发现并连接广播中的 Peripheral
在BLE中 ，Peripheral 通过广播的方式向Central提供数据是主要方式。主要操作如下：
- 服务端 外围设备（Peripheral）： 向外广播数据包（Advertising）形式的数据，比如设备名称，功能等;
- 客户端 中心设备（Central ） ： 可以通过实现扫描和监听到广播数据包，并展示数据;

### 外围设备（Peripheral ）数据构成
- Peripheral 包含一个或者多个Service（服务）以及有关其连接信号强度的有用信息 。
- Service ：是指为了实现一个功能或者特性的数据采集和相关行为的集合，包含一个或多个特征（Characteristic）。
- Characteristic ：它包括一个value和0至多个对次value的描述（Descriptor），知道更多数据细节。
- Descriptor ： 对Characteristic的描述，例如范围、计量单位等
## Central 与 Peripheral 通信
在 Central 与 Perpheral 建立连接后，就可以发现（discoverServices）Perpheral 提供的所有的Services和Characteristic 。然后，Central 可以通过读写Service中的Characteristic 的value与Perpheral进行通信
### Central （中心设备）开发流程
1. 建立中心角色 
确定中心角色，客户端确立。

2. 扫描 外设 
扫描设备：Android 5.0 以下使用 BluetoothAdapter#startLeScan() 实现扫描，Android5.0及其以上，使用android.bluetooth.le 包下 BluetoothLeScaner#startScan()实现。

3. 建立连接 
中心设备主要的类 BluetoothGatt 核心对象，通过 BlueToothDevice#connectGatt（） 可以获取到操作对象

4. 获取services与characteristics 
在连接设备的时候，会使用BluetoothGattCallback 进行一些数据返回和状态返回，当连接成功时，进行discoverservices 
```
@Override
public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
    if (newState == BluetoothGatt.STATE_CONNECTED) {
        gatt.discoverServices();
    }
    @Override
public void onServicesDiscovered(BluetoothGatt gatt, int status) {
    super.onServicesDiscovered(gatt, status);
    //当调用 discoverServices的时候，会调用此方法
    printServices(gatt);
}
```
5. 与外设设备交互和订阅 Characteristic 通知
6. 断开连接 
```
public void disconnect() {
    if (mBluetoothGatt != null) {
        mBluetoothGatt.disconnect();
        mBluetoothGatt.close();
    }
}
```

### Peripheral（外围设备）开发流程
1. 确定外围设备

2. 初始化外围设备管理对象 
Android 5.0 之前没有外围设备开发，目前使用 BluetoothLeAdvertiser 进行外围设备管理；可以通过下面的方法获取 BluetoothAdapter#getBluetoothLeAdvertiser() 可以获取到该对象。

3. 打开外围广播 
根据SERIVE_UUID，打开一个外围设备连接，通过下面类实现BluetoothLeAdvertiser#startAdvertising();

4. 获取外围设备Server 
通过BluetoothGattServer 进行订阅通知，读写Characteristic操作，通过下面方法获取。 
mBluetoothManager.openGattServer(mContext, bluetoothGattServerCallback)

## 参考文档
后续会推出Android-BLE蓝牙之Peripheral和Android-BLE蓝牙之Central详细记录BLE蓝牙作为中心设备和外围设备开发
[https://developer.android.google.cn/reference/android/bluetooth/le/AdvertiseData.html](https://developer.android.google.cn/reference/android/bluetooth/le/AdvertiseData.html)