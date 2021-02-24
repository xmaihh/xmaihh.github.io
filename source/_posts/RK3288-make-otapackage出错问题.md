---
layout: 'n'
title: RK3288 make otapackage出错问题
date: 2018-07-20 19:23:55
categories: Android Framework
tags: [Android]
toc: true
description: If you saw the darkness in front of you, don't be afraid, that's because sunshine is at your back.
---
OTA完整包可用于T卡本地升级和OTA在线升级。OTA完整包包含完整的system、recovery.
和 boot.img。编译 OTA 完整包必须在 android 系统编译(`make –j4` 和 `./mkimage.sh ota`)完成后
进行。编译 OTA 完整包命令如下：
`make otapackage`

编译日志
```
Installed file list: out/target/product/rk3288/installed-files.txt
Target system fs image: out/target/product/rk3288/obj/PACKAGING/systemimage_intermediates/system.img
Running:  mkuserimg.sh -s out/target/product/rk3288/system out/target/product/rk3288/obj/PACKAGING/systemimage_intermediates/system.img ext4 system 1073741824 out/target/product/rk3288/root/file_contexts
make_ext4fs -s -T -1 -S out/target/product/rk3288/root/file_contexts -l 1073741824 -a system out/target/product/rk3288/obj/PACKAGING/systemimage_intermediates/system.img out/target/product/rk3288/system
Creating filesystem with parameters:
    Size: 1073741824
    Block size: 4096
    Blocks per group: 32768
    Inodes per group: 8192
    Inode size: 256
    Journal blocks: 4096
    Label: 
    Blocks: 262144
    Block groups: 8
    Reserved block group size: 63
Created filesystem with 1875/65536 inodes and 110947/262144 blocks
Construct recovery from boot
mkdir -p out/target/product/rk3288/obj/PACKAGING/recovery_patch_intermediates/
PATH=out/host/linux-x86/bin:$PATH out/host/linux-x86/bin/imgdiff out/target/product/rk3288/boot.img out/target/product/rk3288/recovery.img out/target/product/rk3288/obj/PACKAGING/recovery_patch_intermediates/recovery_from_boot.p
chunk 0: type 0 start 0 len 7012362
chunk 1: type 2 start 7012362 len 2239232
chunk 2: type 0 start 8354422 len 296330
Construct patches for 3 chunks...
patch   0 is 248 bytes (of 7012362)
patch   1 is 1378159 bytes (of 1342060)
patch   2 is 202 bytes (of 296330)
chunk   0: normal   (         0,    7012362)         248
chunk   1: deflate  (   7012362,    2917660)     1378159  (null)
chunk   2: normal   (   9930022,     309978)         202
Install system fs image: out/target/product/rk3288/system.img
out/target/product/rk3288/system.img+out/target/product/rk3288/obj/PACKAGING/recovery_patch_intermediates/recovery_from_boot.p maxsize=1096212480 blocksize=135168 total=440127133 reserve=11083776
No RK Loader for TARGET_DEVICE rk3288 to otapackage
package add resource.img to BOOT and RECOVERY
No uboot for uboot/uboot.img to otapackage
No trust for uboot/trust.img to otapackage
No charge for uboot/charge.img to otapackage
No parameter for TARGET_DEVICE rk3288 to otapackage
Package target files: out/target/product/rk3288/obj/PACKAGING/target_files_intermediates/rk3288-target_files-eng.root.zip
building image from target_files RECOVERY...
Traceback (most recent call last):
  File "./build/tools/releasetools/make_recovery_patch", line 68, in <module>
    main(sys.argv[1:])
  File "./build/tools/releasetools/make_recovery_patch", line 39, in main
    input_dir, "RECOVERY")
  File "/media/liw/173d54d9-5bc8-4998-a41d-77fa4e377b61/rk3288_box_sdk_5.1/build/tools/releasetools/common.py", line 419, in GetBootableImage
    info_dict)
  File "/media/liw/173d54d9-5bc8-4998-a41d-77fa4e377b61/rk3288_box_sdk_5.1/build/tools/releasetools/common.py", line 376, in BuildBootableImage
    p4 = Run(sign_cmd)
  File "/media/liw/173d54d9-5bc8-4998-a41d-77fa4e377b61/rk3288_box_sdk_5.1/build/tools/releasetools/common.py", line 86, in Run
    return subprocess.Popen(args, **kwargs)
  File "/usr/lib/python2.7/subprocess.py", line 710, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1327, in _execute_child
    raise child_exception
OSError: [Errno 2] No such file or directory
make: *** [out/target/product/rk3288/obj/PACKAGING/target_files_intermediates/rk3288-target_files-eng.root.zip] 错误 1


#### make failed to build some targets (03:16 (mm:ss)) ####
```
解决方案
build/tools/releasetools/common.py修改一下
贴上patch
```
@@ -348,6 +348,17 @@
   if os.access(fn, os.F_OK):
     cmd.append("--pagesize")
     cmd.append(open(fn).read().rstrip("\n"))
+  
+  fn = os.path.join(sourcedir, "second")
+  if os.access(fn, os.F_OK):
+    cmd.append("--second")
+    cmd.append(fn)
+
+
+  fn = os.path.join(sourcedir, "third")
+  if os.access(fn, os.F_OK):
+    cmd.append("--third")
+    cmd.append(fn)
 
   args = info_dict.get("mkbootimg_args", None)
   if args and args.strip():
@@ -362,10 +373,10 @@
       os.path.basename(sourcedir),)
 
   sign_cmd = ["drmsigntool", img.name, "build/target/product/security/privateKey.bin"]
-  p4 = Run(sign_cmd)
-  p4.communicate()
-  assert p4.returncode == 0, "mkbootimg of %s image failed" % (
-          os.path.basename(sourcedir),)
+ # p4 = Run(sign_cmd)
+ # p4.communicate()
+#  assert p4.returncode == 0, "mkbootimg of %s image failed" % (
+#          os.path.basename(sourcedir),)
 
   #if info_dict.get("verity_key", None):
   #  path = "/" + os.path.basename(sourcedir).lower()
@@ -877,8 +888,8 @@
             f = b
           info = imp.find_module(f, [d])
         print "loaded device-specific extensions from", path
-        self.module = imp.load_module("device_specific", *info)
-        D("module = %s", self.module);
+       # self.module = imp.load_module("device_specific", *info)
+       # D("module = %s", self.module);
       except ImportError:
         print "unable to load device-specific module; assuming none"
```