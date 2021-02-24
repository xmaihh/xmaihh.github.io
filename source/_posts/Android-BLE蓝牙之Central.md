---
title: Android-BLE蓝牙之Central
date: 2018-09-10 14:19:27
categories: Android
tags: [BLE]
toc: false
description: Don't forget to promise to myself to do, don't forget promised to go to your place, no matter how hard it is, how far is it.
---
低功耗蓝牙(BLE)Android 4.3（API 18）以上才支持
Android 5.0(API 21) 扫描蓝牙需要定位权限，否则扫描不到设备,实际使用时候发现 5.0不需要也可以扫描，Android 6.0(API 23)以上必须需要定位权限
官方文档:[https://developer.android.com/guide/topics/connectivity/bluetooth-le.html?hl=zh-cn](https://developer.android.com/guide/topics/connectivity/bluetooth-le.html?hl=zh-cn)
## 声明权限
在项目的AndroidManifest.xml中声明权限
```
 // 定位权限第二个包含第一个，所以这里就声明了一个，两个都声明也可以
  <!-- <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/> -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/> 
  <uses-permission android:name="android.permission.BLUETOOTH"/>
  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
  // 是否必须支持低功耗蓝牙，此处不必须
   <uses-feature
      android:name="android.hardware.bluetooth_le"
      android:required="false"/>
   // 是有gps硬件
  <uses-feature android:name="android.hardware.location.gps"/>
```
## 检查是否支持BLE
```
    /**
     * 首先检查是否支持BLE
     */
    private void init() {
        if (!getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)) {
            Toast.makeText(this, "BLE not supported", Toast.LENGTH_LONG).show();
//            finish();
        }

        mBluetoothManager = (BluetoothManager) getSystemService(BLUETOOTH_SERVICE);
        mBluetoothAdapter = mBluetoothManager.getAdapter();

        if (mBluetoothAdapter == null) {
            Toast.makeText(this, "Bluetooth not supported", Toast.LENGTH_LONG).show();
        }

        if (!mBluetoothAdapter.isEnabled()) {
            Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
            //判断是否打开蓝牙,如果没有打开需要回调打开蓝牙,这里省略,请自行打开蓝牙
        }
    }
```
通过BluetoothManager得到蓝牙适配器BluetoothAdapter,如果获取到的Adapter为空说明不支持蓝牙，或者没有蓝牙模块
## 扫描设备
低功耗蓝牙和经典蓝牙的扫描方式不同,蓝牙扫描耗电比较严重，所以一定要记得在合适的实际停止扫描，比如定时停止、扫描到目标设备就停止,扫描时调用Adapter的startLeScan()方法,然而这个方法被标记为过时，API>=21被另一个取代。然后通过回调得到扫描结果（经典蓝牙是通过广播得到扫描结果）
```
 mBluetoothAdapter.startLeScan(this);
 ...
 /**
 * 扫描结果回调
 */
 @Override
public void onLeScan(BluetoothDevice device, int rssi, byte[] scanRecord) {
 // 得到扫描结果
    Log.i(TAG, "扫描到的设备, name=" + device.getName() + ",address=" + device.toString());
}
```
device：代表外设即目标设备 rssi：设一个强度值，但是时负值，利用这个值通过公式可以算出离你的距离
scanRecord：广播数据，附加的数据
## 停止扫描
扫描完成务必停止，因为扫描不仅耗电，还影响连接速度，所以当要连接的时候，先停止扫描时必须的
```
public void stopScan() {
    initHandler();
    Log.i(TAG, "停止扫描，蓝牙设备");
    if (mScanning) {
        mScanning = false;
        // 开始扫描的接口,要一样的不然停止不了
        mBluetoothAdapter.stopLeScan(this);
    }
    if (scanRunnable != null) {
        mHandler.removeCallbacks(scanRunnable);
        scanRunnable = null;
    }
}
```
##　连接设备
通常Ble连接设备速度还是很快的，连接理论上来说也是无状态的，所以也需要一个定时任务来，保证超时停止。
```
public boolean connect(Context context, String address) {
    if (mConnectionState == STATE_CONNECTED) {
        return false;
    }
    if (mBluetoothAdapter == null || TextUtils.isEmpty(address)) {
        return false;
    }
    initHandler();
    if (connectRunnable == null) {
        connectRunnable = new Runnable() {
            @Override
            public void run() {
                mConnectionState = STATE_DISCONNECTING;
                disconnect();
            }
        };
    }
  // 30s没反应停止连接
    mHandler.postDelayed(connectRunnable, 30 * 1000);
    stopScan();
// 获取到远程设备，
    final BluetoothDevice device = mBluetoothAdapter.getRemoteDevice(address);
    if (device == null) {
        return false;
    }
    // 开始连接，第二个参数表示是否需要自动连接，true设备靠近自动连接，第三个表示连接回调
    mBluetoothGatt = device.connectGatt(context, false, mGattCallback);
    mBluetoothDeviceAddress = address;
    mConnectionState = STATE_CONNECTING;
    return true;
}
```
## 监听连接回调
```
private final BluetoothGattCallback mGattCallback = new BluetoothGattCallback() {
// 连接状态变化
    @Override
    public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
        if (newState == BluetoothProfile.STATE_CONNECTED) { // 连接上
            mConnectionState = STATE_CONNECTED;
            boolean success = mBluetoothGatt.discoverServices(); // 去发现服务
            Log.i(TAG, "Attempting to start service discovery:" +
                    success);
        } else if (newState == BluetoothProfile.STATE_DISCONNECTED) { // 连接断开
            mConnectionState = STATE_DISCONNECTED;       
        }
    }

// 发现服务
    @Override
    public void onServicesDiscovered(BluetoothGatt gatt, int status) {
        if (status == BluetoothGatt.GATT_SUCCESS) {
            Log.i(TAG,"发现服务");
            // 解析服务
            discoverService();
        }
    }

// 特征读取变化
    @Override
    public void onCharacteristicRead(BluetoothGatt gatt,
                                     BluetoothGattCharacteristic characteristic,
                                     int status) {
        if (status == BluetoothGatt.GATT_SUCCESS) {
          
        }
    }

// 收到数据
    @Override
    public void onCharacteristicChanged(BluetoothGatt gatt,
                                        BluetoothGattCharacteristic characteristic) {
        mConnectionState = STATE_CONNECTED;
        
    }
};
```
## 断开连接
```
public void disconnect() {
    if (mBluetoothAdapter == null || mBluetoothGatt == null) {
        mConnectionState = STATE_DISCONNECTED;
        return;
    }
    // 连接成功的GATT
    mBluetoothGatt.disconnect();
    mBluetoothGatt.close();
}
```
## 多设备连接
蓝牙适配器没有听过连接多个设备的接口，需要自己实现，即获取到目标设备的address后调用连接方法

>蓝牙操作的注意事项
蓝牙的写入操作( 包括 Descriptor 的写入操作), 读取操作必须序列化进行. 写入数据和读取数据是不能同时进行的, 如果调用了写入数据的方法, 马上调用又调用写入数据或者读取数据的方法,第二次调用的方法会立即返回 false, 代表当前无法进行操作. 详情可以参考 [蓝牙读写操作返回 false，为什么多次读写只有一次回调？](http://a1anwang.com/post-18.html)
Android 连接外围设备的数量有限，当不需要连接蓝牙设备的时候，必须调用 BluetoothGatt#close 方法释放资源。详细的参考可以看这里 [Android BLE 蓝牙开发的各种坑](http://www.eoeandroid.com/thread-902192-1-1.html?_dsign=d85c38ec)
蓝牙 API 连接蓝牙设备的超时时间大概在 20s 左右，具体时间看系统实现。有时候某些设备进行蓝牙连接的时间会很长，大概十多秒。如果自己手动设置了连接超时时间（例如通过 Handler#postDelay 设置了 5s 后没有进入 BluetoothGattCallback#onConnectionStateChange 就执行 BluetoothGatt#close 操作强制释放断开连接释放资源）在某些设备上可能会导致接下来几次的连接尝试都会在 BluetoothGattCallback#onConnectionStateChange 返回 state == 133。另外可以参考这篇吐槽 [Android 中 BLE 连接出现“BluetoothGatt status 133”的解决方法](http://www.loverobots.cn/android-ble-connection-solution-bluetoothgatt-status-133.html)
所有的蓝牙操作使用 Handler 固定在一条线程操作，这样能省去很多因为线程不同步导致的麻烦

### 重要实例
经典蓝牙聊天实例：https://github.com/googlesamples/android-BluetoothChat
低功耗蓝牙：https://github.com/googlesamples/android-BluetoothLeGatt
作为外设（API>=21）：https://github.com/googlesamples/android-BluetoothAdvertisements
基础概念讲解：http://blog.csdn.net/qinxiandiqi/article/details/40741269
深入理论:http://www.race604.com/android-ble-in-action/
AndroidBLE蓝牙框架，包括扫描、连接、设置通知、发送数据、读取、接收数据和OTA升级以及各种直观的回调:
https://github.com/Alex-Jerry/Android-BLE