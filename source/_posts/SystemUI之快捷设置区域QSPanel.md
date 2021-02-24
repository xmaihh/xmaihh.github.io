---
title: SystemUI之快捷设置区域QSPanel
date: 2018-07-08 14:17:32
categories: Android Framework
tags: [Android]
toc: true
description: Take nothing for granted. Know that the harder you work, the luckier you'll get. — Ivanka Trump 
---
SystemUI下拉之后的那些快捷设置菜单选项也是属于SystemUI的一种;它的加载也是随着PhoneStatusBar的加载而加载;
/frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBar.java
首先从布局方面入手：
快捷设置区域的布局是由PhoneStatusBar.Java的makeStatusBarView()统一加载;
```
mStatusBarWindow=(StatusBarWindowView)View.inflate(context,
R.layout.super_status_bar,null);
```
加载frameworks/base/packages/SystemUI/res/layout/super_status_bar.xml布局
```
..
...
    <com.android.systemui.statusbar.phone.PanelHolder
        android:id="@+id/panel_holder"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/transparent" >
        <include layout="@layout/status_bar_expanded"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:visibility="gone" />
    </com.android.systemui.statusbar.phone.PanelHolder>
...
..
```
而这个布局回去include另外一个布局status_bar_expanded.xml布局，frameworks/base/packages/SystemUI/res/layout/status_bar_expanded.xml;
```
..
...
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
    <include
        layout="@layout/qs_panel"
        android:layout_marginTop="@dimen/status_bar_header_height_expanded"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="@dimen/notification_side_padding"
      android:layout_marginRight="@dimen/notification_side_padding"/>
...
..
```
同样的在这个布局文件中又会去include一个qs_panel.xml布局，frameworks/base/packages/SystemUI/res/layout/qs_panel.xml;
```
<com.android.systemui.qs.QSContainer
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/quick_settings_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/qs_background_primary"
        android:paddingTop="8dp"
        android:paddingBottom="8dp"
        android:elevation="2dp">

    <com.android.systemui.qs.QSPanel
            android:id="@+id/quick_settings_panel"
            android:background="#0000"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
</com.android.systemui.qs.QSContainer>
```
见名知意，这个id为quick_settings_panel即为我们所找的那个SystemUI上的快捷设置区域控件的ID;
代码控制方面：
同样的也是由PhoneStatusBar.java的makeStatusBarView()方法开始的。
```
        // Set up the quick settings tile panel
        mQSPanel = (QSPanel) mStatusBarWindow.findViewById(R.id.quick_settings_panel);
        if (mQSPanel != null) {
            final QSTileHost qsh = new QSTileHost(mContext, this,
                    mBluetoothController, mLocationController, mRotationLockController,
                    mNetworkController, mZenModeController, mHotspotController,
                    mCastController, mFlashlightController,
                    mUserSwitcherController, mKeyguardMonitor,
                    mSecurityController);
            mQSPanel.setHost(qsh);
            mQSPanel.setTiles(qsh.getTiles());
            mBrightnessMirrorController = new BrightnessMirrorController(mStatusBarWindow);
            mQSPanel.setBrightnessMirror(mBrightnessMirrorController);
            mHeader.setQSPanel(mQSPanel);
            qsh.setCallback(new QSTileHost.Callback() {
                @Override
                public void onTilesChanged() {
                    mQSPanel.setTiles(qsh.getTiles());
                }
            });
        }
```
一步步分析，首先先实例化一个QSPanel对象，然后再去创建QSTileHost对象，其传入的参数即为各种快捷设置控制器，如蓝牙、屏幕旋转、定位、闪光灯等等;
分析QSTileHost.java的构造方法：
```
    public QSTileHost(Context context, PhoneStatusBar statusBar,
            BluetoothController bluetooth, LocationController location,
            RotationLockController rotation, NetworkController network,
            ZenModeController zen, HotspotController hotspot,
            CastController cast, FlashlightController flashlight,
            UserSwitcherController userSwitcher, KeyguardMonitor keyguard,
            SecurityController security) {
       
        mContext = context;
        mStatusBar = statusBar;
        mBluetooth = bluetooth;
        mLocation = location;
        mRotation = rotation;
        mNetwork = network;
        mZen = zen;
        mHotspot = hotspot;
        mCast = cast;
        mFlashlight = flashlight;
        mUserSwitcherController = userSwitcher;
        mKeyguard = keyguard;
        mSecurity = security;
       
        final HandlerThread ht = new HandlerThread(QSTileHost.class.getSimpleName(),
                Process.THREAD_PRIORITY_BACKGROUND);
        ht.start();
        mLooper = ht.getLooper();
        mUserTracker = new CurrentUserTracker(mContext) {
            @Override
            public void onUserSwitched(int newUserId) {
                recreateTiles();
                for (QSTile<?> tile : mTiles.values()) {
                    tile.userSwitch(newUserId);
                }
                mSecurity.onUserSwitched(newUserId);
                mNetwork.onUserSwitched(newUserId);
                mObserver.register();
            }
        };
        recreateTiles();

        mUserTracker.startTracking();
        mObserver.register();
    }
```
### 实例化各个控制器，创建一个子线程handler获取Looper对象
### 使用TunerService去Settings中查询key为TILES_SETTING的值，即查询快捷设置菜单项，查询到的结果通过onTuningChanged()方法回调返回;
frameworks/base/packages/SystemUI/src/com/android/systemui/tuner/TunerService.java
```
private void addTunable(Tunable tunable, String key) {
        if (!mTunableLookup.containsKey(key)) {
            mTunableLookup.put(key, new ArraySet<Tunable>());
        }
        mTunableLookup.get(key).add(tunable);
        Uri uri = Settings.Secure.getUriFor(key);
        if (!mListeningUris.containsKey(uri)) {
            mListeningUris.put(uri, key);
            mContentResolver.registerContentObserver(uri, false, mObserver, mCurrentUser);
        }
        // Send the first state.
        String value = Settings.Secure.getStringForUser(mContentResolver, key, mCurrentUser);
        tunable.onTuningChanged(key, value);
    }
```
frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/QSTileHost.java的onTuningChanged()方法
```
@Override
    public void onTuningChanged(String key, String newValue) {
        if (!TILES_SETTING.equals(key)) {
            return;
        }
        if (DEBUG) Log.d(TAG, "Recreating tiles");
        final List<String> tileSpecs = loadTileSpecs(newValue);
        if (tileSpecs.equals(mTileSpecs)) return;
        for (Map.Entry<String, QSTile<?>> tile : mTiles.entrySet()) {
            if (!tileSpecs.contains(tile.getKey())) {
                if (DEBUG) Log.d(TAG, "Destroying tile: " + tile.getKey());
                tile.getValue().destroy();
            }
        }
        final LinkedHashMap<String, QSTile<?>> newTiles = new LinkedHashMap<>();
        for (String tileSpec : tileSpecs) {
            if (mTiles.containsKey(tileSpec)) {
                newTiles.put(tileSpec, mTiles.get(tileSpec));
            } else {
                if (DEBUG) Log.d(TAG, "Creating tile: " + tileSpec);
                try {
                    newTiles.put(tileSpec, createTile(tileSpec));
                } catch (Throwable t) {
                    Log.w(TAG, "Error creating tile for spec: " + tileSpec, t);
                }
            }
        }
        mTileSpecs.clear();
        mTileSpecs.addAll(tileSpecs);
        mTiles.clear();
        mTiles.putAll(newTiles);
        if (mCallback != null) {
            mCallback.onTilesChanged();
        }
    }
```
### 在通过调用loadTileSpecs()方法对查询的结果进行判断处理;
```
protected List<String> loadTileSpecs(Context context, String tileList) {
        final Resources res = context.getResources();
        final String defaultTileList = res.getString(R.string.quick_settings_tiles_default);
        if (tileList == null) {
            tileList = res.getString(R.string.quick_settings_tiles);
            if (DEBUG) Log.d(TAG, "Loaded tile specs from config: " + tileList);
        } else {
            if (DEBUG) Log.d(TAG, "Loaded tile specs from setting: " + tileList);
        }
        final ArrayList<String> tiles = new ArrayList<String>();
        boolean addedDefault = false;
        for (String tile : tileList.split(",")) {
            tile = tile.trim();
            if (tile.isEmpty()) continue;
            if (tile.equals("default")) {
                if (!addedDefault) {
                    tiles.addAll(Arrays.asList(defaultTileList.split(",")));
                    addedDefault = true;
                }
            } else {
                tiles.add(tile);
            }
        }
        return tiles;
    }
```
如果返回的结果tileList不为null，使用","来拆分结果，对得到的每个tile进行判断，如果不为"default",即保存此tile;
如果返回的结果tileList为null，则tileList赋值为"default"，并读取config.xml中的quick_settings_tiles_default字串，拆分保存;
返回所符合要求显示的快捷设置tile集合;
### 在QSTileHost.java的onTuningChanged()方法中调用createTile()方法来创建每一个快捷设置的tile对象；
```
public QSTile<?> createTile(String tileSpec) {
        if (tileSpec.equals("wifi")) return new WifiTile(this);
        else if (tileSpec.equals("bt")) return new BluetoothTile(this);
        else if (tileSpec.equals("cell")) return new CellularTile(this);
        else if (tileSpec.equals("dnd")) return new DndTile(this);
        else if (tileSpec.equals("inversion")) return new ColorInversionTile(this);
        else if (tileSpec.equals("airplane")) return new AirplaneModeTile(this);
        else if (tileSpec.equals("work")) return new WorkModeTile(this);
        else if (tileSpec.equals("rotation")) return new RotationLockTile(this);
        else if (tileSpec.equals("flashlight")) return new FlashlightTile(this);
        else if (tileSpec.equals("location")) return new LocationTile(this);
        else if (tileSpec.equals("cast")) return new CastTile(this);
        else if (tileSpec.equals("hotspot")) return new HotspotTile(this);
        else if (tileSpec.equals("user")) return new UserTile(this);
        else if (tileSpec.equals("battery")) return new BatteryTile(this);
        else if (tileSpec.equals("saver")) return new DataSaverTile(this);
        else if (tileSpec.equals("night")) return new NightDisplayTile(this);
        // Intent tiles.
        else if (tileSpec.startsWith(IntentTile.PREFIX)) return IntentTile.create(this,tileSpec);
        else if (tileSpec.startsWith(CustomTile.PREFIX)) return CustomTile.create(this,tileSpec);
        else {
            Log.w(TAG, "Bad tile spec: " + tileSpec);
            return null;
        }
    }
```
并将其保存在成员变量的mTiles集合中，最后回调onTilesChanged()方法，通知PhoneStatusBar.java对快捷设置选项显示更新;
```
if (mCallback != null) {
            mCallback.onTilesChanged();
        }
```
至此QSTileHost.java的构造方法分析完成，然后再回到调用处PhoneStatusBar.java的makeStatusBarView()方法继续分析：
```
mQSPanel.setHost(qsh);
mQSPanel.setTiles(qsh.getTiles());
```
设置QSPanel.setHost()、设置QSPanel.setTiles();而其中的其中setTiles()方法会先remove掉所有的TileRecord记录并移除所有的tileView;
```
public void setTiles(Collection<QSTile<?>> tiles) {
        for (TileRecord record : mRecords) {
            removeView(record.tileView);
        }
        mRecords.clear();
        for (QSTile<?> tile : tiles) {
            addTile(tile);
        }
        if (isShowingDetail()) {
            mDetail.bringToFront();
        }
    }
```
然后在重新调用addTile()创建TileRecord对象并赋值绑定相应的回调和点击事件(点击、双击、长按)接口，再将其保存到ArrayList<TileRecord> mRecords集合中，然后再去addView();
```
private void addTile(final QSTile<?> tile) {
        final TileRecord r = new TileRecord();
        r.tile = tile;
        r.tileView = tile.createTileView(mContext);
        r.tileView.setVisibility(View.GONE);
        final QSTile.Callback callback = new QSTile.Callback() {
            @Override
            public void onStateChanged(QSTile.State state) {
                int visibility = state.visible ? VISIBLE : GONE;
                if (state.visible && !mGridContentVisible) {
 
                    // We don't want to show it if the content is hidden,
                    // then we just set it to invisible, to ensure that it gets visible again
                    visibility = INVISIBLE;
                }
                setTileVisibility(r.tileView, visibility);
                r.tileView.onStateChanged(state);
            }
            @Override
            public void onShowDetail(boolean show) {
                QSPanel.this.showDetail(show, r);
            }
            @Override
            public void onToggleStateChanged(boolean state) {
                if (mDetailRecord == r) {
                    fireToggleStateChanged(state);
                }
            }
            @Override
            public void onScanStateChanged(boolean state) {
                r.scanState = state;
                if (mDetailRecord == r) {
                    fireScanStateChanged(r.scanState);
                }
            }
 
            @Override
            public void onAnnouncementRequested(CharSequence announcement) {
                announceForAccessibility(announcement);
            }
        };
        r.tile.setCallback(callback);
        final View.OnClickListener click = new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                r.tile.click();
            }
        };
        final View.OnClickListener clickSecondary = new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                r.tile.secondaryClick();
            }
        };
        final View.OnLongClickListener longClick = new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                r.tile.longClick();
                return true;
            }
        };
        r.tileView.init(click, clickSecondary, longClick);
        r.tile.setListening(mListening);
        callback.onStateChanged(r.tile.getState());
        r.tile.refreshState();
        mRecords.add(r);
 
        addView(r.tileView);
    }
```
再回到调用处PhoneStatusBar.java的makeStatusBarView()方法继续分析：
```
qsh.setCallback(new QSTileHost.Callback() {
                @Override
                public void onTilesChanged() {
                    mQSPanel.setTiles(qsh.getTiles());
                }
            });
```
为QSTileHost的对象设置onTilesChanged()回调监听；

至此完成快捷区域加载显示的大致流程分析。

### 隐藏下拉状态栏快捷方式方法：
```
    private QSTile<?> createTile(String tileSpec) {
        if (tileSpec.equals("wifi")) return new WifiTile(this);
        /**delete by chensy 隐藏下拉状态栏 蓝牙、飞行模式、自动旋转、GPS、投屏 start*/
//        else if (tileSpec.equals("bt")) return new BluetoothTile(this);
        else if (tileSpec.equals("inversion")) return new ColorInversionTile(this);
        else if (tileSpec.equals("cell")) return new CellularTileForSlot(this, PhoneConstants.SIM_ID_1);
        else if (tileSpec.equals("cell2")) return new CellularTileForSlot(this, PhoneConstants.SIM_ID_2);
//        else if (tileSpec.equals("airplane")) return new AirplaneModeTile(this);
//        else if (tileSpec.equals("rotation")) return new RotationLockTile(this);
        else if (tileSpec.equals("flashlight")) return new FlashlightTile(this);
//        else if (tileSpec.equals("location")) return new LocationTile(this);
//        else if (tileSpec.equals("cast")) return new CastTile(this);
        /**delete by chensy 隐藏下拉状态栏 蓝牙、飞行模式、自动旋转、GPS、投屏 end*/
        else if (tileSpec.equals("hotspot")) return new HotspotTile(this);
        else if (tileSpec.startsWith(IntentTile.PREFIX)) return IntentTile.create(this,tileSpec);
        else throw new IllegalArgumentException("Bad tile spec: " + tileSpec);
    }
```
