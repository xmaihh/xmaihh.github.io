---
title: Android OTA升级包制作
date: 2018-07-23 20:05:50
categories: Android Framework
tags: [Android]
toc: true
description: When the gorgeous stage to become a memory,you do not indulge in the glory of the year,otherwise it will make you a headache.  
---
OTA（Over-the-AirTechnology）是指手机终端通过无线网络下载远程服务器上的升级包，对系统或应用进行升级的技术。
OTA升级包（实质上是Recovery升级的ZIP包，OTA升级是基于Recovery的机制再加上下载ZIP包和ZIP包版本管理等功能实现）
### OTA升级包
#### OTA完整包生成方法
OTA完整包可用于T卡本地升级和OTA在线升级。OTA完整包包含完整的system、recovery、
和 boot.img。编译 OTA 完整包必须在 android 系统编译(`make –j4`和`./mkimage.sh ota`)完成后
进行。编译 OTA 完整包命令如下：
`make otapackage`
在 out/target/product/rk29sdk/目录下生成 ota 完整包 rk29sdk‐ota‐eng.root.zip，改名成
update.zip 即可拷贝到 T 卡或内置 flash 中进行固件升级。
##### OTA完整包结构
```
├── boot.img  //更新boot分区所需要的文件。这个boot.img主要包括kernel+ramdisk，包括应用会用到的一些库等等
├── file_contexts
├── META-INF
│   ├── CERT.RSA  //与签名文件相关联的签名程序块文件，它存储了用于签名JAR文件的公共签名
│   ├── CERT.SF  //这是JAR文件的签名文件，其中前缀CERT代表签名者
│   ├── com
│   │   ├── android
│   │   │   ├── metadata  //描述设备信息及环境变量的元数据。主要包括一些编译选项，签名公钥，时间戳以及设备型号等
│   │   │   └── otacert
│   │   └── google
│   │       └── android
│   │           ├── update-binary  //升级程序，解析执行升级脚本 一个二进制文件,能够识别updater-script中描述的操作。
│   │           └── updater-script  //升级脚本,具体描述了更新过程
│   └── MANIFEST.MF  //这个manifest文件定义了与包的组成结构相关的数据。类似Android应用的mainfest.xml文件
├── recovery  //升级相关的文件
│   ├── bin
│   │   └── install-recovery.sh  //执行更新的脚本
│   └── recovery-from-boot.p  //recovery-from-boot.p是boot.img和recovery.img的补丁(patch),主要用来更新recovery分区
└── system  //更新system分区所需要的文件。这个system主要用来更新系统的一些应用或则应用会用到的一些库等等
    ├── app
    ├── bin
    ├── build.prop
    ├── etc
    ├── fonts
    ├── framework
    ├── lib
    ├── manifest.xml
    ├── media
    ├── priv-app
    ├── recovery-from-boot.p
    ├── tts
    ├── usr
    ├── vendor
    └── xbin
```
#### OTA差异包生成方法
OTA 差异包只有差异内容，包大小比较小，主要用于 OTA 在线升级，也可 T 卡本地升级。
OTA 差异包制作需要特殊的编译进行手动制作，以 rk29sdk 删除 VideoPlayer.apk 为例，具体
说明 ota 差异包的制作。
1.删除 VideoPlayer.apk 之前的版本先生成用于差异包的 target file:
````
make otapackage
```
2.将生成的target file 改名成 old,用于后面生成差异包使用
```
mv
out/target/product/rk29sdk/obj/PACKAGING/target_files_intermediates/rk29sdk‐target_files
‐eng.root.zip
out/target/product/rk29sdk/obj/PACKAGING/target_files_intermediates/rk29sdk‐target_files
‐eng‐old.root.zip 
```
3.删除device/rockchip/rk29sdk/apk/VideoPlayer.apk
和 out/target/product/rk29sdk/system/app/VideoPlayer.apk 后重新生成新的 target file:
```
rm device/rockchip/rk29sdk/apk/VideoPlayer.apk
rm out/target/product/rk29sdk/system/app/VideoPlayer.apk
make otapackage
```
4.生成差异包:
```
./build/tools/releasetools/ota_from_target_files
‐v –i
out/target/product/rk29sdk/obj/PACKAGING/target_files_intermediates/rk29sdk‐target_files‐eng
‐old.root.zip
‐p out/host/linux‐x86
‐k build/target/product/security/testkey
out/target/product/rk29sdk/obj/PACKAGING/target_files_intermediates/rk29sdk‐target_files‐eng
.root.zip
out/target/product/rk29sdk/rk29sdk‐ota‐eng.root.zip
```
##### OTA差异包结构
```
├── METE-INF
├── patch
│   ├── app
│   │   └── build.prop.p
│   ├── boot.img.p
│   ├── etc
│   ├── lib
│   └── system
├── recovery
└── system
```
说明: 生成差异包命令格式:
- ota_from_target_files   
- –v –i 用于比较的前一个 target file   
- –p host 主机编译环境
- ‐k 打包密钥
用于比较的后一个 target file
最后生成的 ota 差异包   
如果要增加修改后的 apk 或库可类似以上操作即可
>OTA 差异包是将本次编译生成的 target 包和上一个版本的 target 进行对比产生差异。如
果要使用差异包，每次编译生成固件时都要 make otapackage 生成 target 包并对应版本保存，
以便下次能够相对于某一版本生成差异包。
### OTA升级包本地升级验证
#### 验证Recovery升级
将上述ZIP包复制到内部存储的任一可写目录下，假设我们的ZIP包复制到/cache目录下，文件名为ota.zip，
那么接下来我们只需要再执行`echo "--update_package=/cache/ota.zip" > /cache/recovery/command`命令，
然后再执行`reboot recovery`命令即可重启进入Recovery模式进行升级验证。
#### T卡本地升级
1.OTA 包改名成 update.zip
将 rk29sdk‐ota‐eng.root.zip 重命名为
update.zip。
2.拷贝 OTA 包到设备
将改名后的 update.zip 拷贝到 rk29 设备的 T 卡或内置 flash 中。
3.自动检测本地 OTA 包，提示升级
拔插 usb 或拔插 T 卡，此时会弹出自动检测到本地 OTA 包的对话框
