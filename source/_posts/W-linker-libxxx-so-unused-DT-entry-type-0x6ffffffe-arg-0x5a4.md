---
title: 'W/linker: libxxx.so: unused DT entry: type 0x6ffffffe arg 0x5a4'
date: 2019-03-05 11:53:09
categories:   Android Framework
tags:  [AndroidStudio,Android,Jni]
toc: true
description: Hypocritical friendship is like your shadow; when you are in the sun, it will closely follow you, but once you go into the shadow, it will leave you.
---

我正在使用[libserialport.so](https://github.com/xmaihh/Android-Serialport),在运行时，我收到以下警告:

```java
W/linker: libserialport.so: unused DT entry: type 0x6ffffffe arg 0x5a4
    libserialport.so: unused DT entry: type 0x6fffffff arg 0x1
```

# Q:What are "unused DT entry" errors?

If you have reached this page, it's probably because you have compiled or attempted to run some binaries on your ARM based Android system, with the result that your binary/app crashes or generates a lot of warnings in your logcat. Typically something like this:

如果您已到达此页面，可能是因为您已编译或尝试在基于ARM的Android系统上运行某些二进制文件，结果导致您的二进制文件/应用程序崩溃或在您的系统中生成大量警告  logcat。通常是这样的:

```java
WARNING: linker: /blahblah/libopenssl.so: unused DT entry: type 0x6ffffffe arg 0x1188
```

# Q: What is a "DT entry"?

In a few words, they are descriptive array entries in the file structure of an ELF file. Specifically they are known as  Dynamic Array Tags and are requirements for executable and shared objects. However, not all entries are required or available, depending on the processor and kernel architecture.

In our case we are faced with a "Warning" that one of these are "unused". What that means is, that your executable or library (*.so) files has been compiled with the DT entry indicated, but your kernel is not supporting that entry, for various reasons. The best examples are found on ARM based Android systems, where the system library paths are fixed and the cross compilers used for your firmware (OS/kernel) are set not to use these entries. Usually the binaries still run just fine, but the kernel is flagging this warning every time you're using it.

简而言之，它们是[ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)文件的文件结构中的描述性数组条目  。具体而言，它们被称为  Dynamic Array Tags 可执行和共享对象的要求。但是，并非所有条目都是必需的或可用的，具体取决于处理器和内核体系结构。在我们的案例中，我们面临一个“警告”，其中一个是“未使用”。这意味着，您的可执行文件或库（*.so）文件已使用 指示的DT条目进行编译  ，但由于各种原因，您的内核不支持该条目。最好的例子可以在基于ARM的Android系统上找到，其中系统库路径是固定的，用于固件的交叉编译器（OS /内核）设置为不使用这些条目。通常二进制文件仍然可以正常运行，但内核每次使用它时都会标记此警告。

# Q: When does this happen?

This can happen when:

- Your ARM kernel is cross-compiled using the wrong flags (usually meant for other processor architectures).
- 您的ARM内核使用错误的标志进行交叉编译（通常用于其他处理器体系结构）。
- Your ARM binaries and libraries are cross-compiled using AOS deprecated compilation flags.
- 您的ARM二进制文件和库是使用AOS弃用的编译标志进行交叉编译的。
- and probably other ways yet to be discovered..
- 可能还有其他方法尚待发现..

> Starting from 5.1 (API 22) the Android linker warns about the VERNEED and VERNEEDNUM ELF dynamic sections.

> 从5.1（API 22）开始，Android链接器会警告VERNEED和VERNEEDNUM ELF动态部分。

The most common flags that cause this error on Android devices are:

在Android设备上导致此错误的最常见标志是：

```java
DT_RPATH        0x0f (15)       The DT_STRTAB string table offset of a null-terminated library search path string. 
                                This element's use has been superseded by DT_RUNPATH.
DT_RUNPATH      0x1d (29)       The DT_STRTAB string table offset of a null-terminated library search path string.
DT_VERNEED      0x6ffffffe      The address of the version dependency table. Elements within this table contain 
                                indexes into the string table DT_STRTAB. This element requires that the 
                                DT_VERNEEDNUM element also be present.
DT_VERNEEDNUM   0x6fffffff      The number of entries in the DT_VERNEEDNUM table.
```

Tracking down the error above, we find that this message comes from the `bionic` library [linker.cpp](https://github.com/aosp-mirror/platform_bionic/blob/1fedfedda8a2b75ed56669e28265f943312ec22f/linker/linker.cpp#L2967-L2986):
追踪上面的错误，我们发现此消息来自  `bionic` 库[linker.cpp](https://github.com/aosp-mirror/platform_bionic/blob/1fedfedda8a2b75ed56669e28265f943312ec22f/linker/linker.cpp#L2967-L2986)：

```java
  case DT_VERNEED:
    verneed_ptr_ = load_bias + d->d_un.d_ptr;
    break;

  case DT_VERNEEDNUM:
    verneed_cnt_ = d->d_un.d_val;
    break;

  case DT_RUNPATH:
    // this is parsed after we have strtab initialized (see below).
    break;

  default:
    if (!relocating_linker) {
      DL_WARN("\"%s\" unused DT entry: type %p arg %p", get_realpath(),
          reinterpret_cast<void*>(d->d_tag), reinterpret_cast<void*>(d->d_un.d_val));
    }
    break;
}
```

The code (above) supporting this symbol versioning was committed on April 9, 2015. Thus if your NDK build is either set to support API's earlier than this, or using build tools linking to this earlier library, you will get these warnings.

支持此[符号版本控制](https://lists.debian.org/lsb-spec/1999/12/msg00017.html)的代码（上文）   于[2015](https://github.com/android/platform_bionic/commit/2a815361448d01b0f4e575f507ce31913214c536#diff-9d78f752a519d0c92b4cf2c888fe26e4)年[4月9日](https://github.com/android/platform_bionic/commit/2a815361448d01b0f4e575f507ce31913214c536#diff-9d78f752a519d0c92b4cf2c888fe26e4)提交  。因此，如果您的NDK构建设置为支持早于此的API，或者使用链接到此早期库的构建工具，您将收到这些警告。

# Q: How do I find what DT entries my system or binaries are using?

There are many ways to do this:

1. You look into your kernel sources for `<linux/elf.h>`.
2. You look in your Android NDK installation folders and check:

```shell
# To find all elf.h files:
find /<path_to>/ndk/platforms/android-*/arch-arm*/usr/include/linux/ -iname "elf.h"
```

3. Do an readelf of your binary:

```shell
$ readelf --dynamic libopenssl.so

 Dynamic section at offset 0x23b960 contains 28 entries:
 Tag        Type                         Name/Value
 0x00000003 (PLTGOT)                     0x23ce18
 0x00000002 (PLTRELSZ)                   952 (bytes)
 0x00000017 (JMPREL)                     0x15e70
 0x00000014 (PLTREL)                     REL
 0x00000011 (REL)                        0x11c8
 0x00000012 (RELSZ)                      85160 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffa (RELCOUNT)                   10632
 0x00000015 (DEBUG)                      0x0
 0x00000006 (SYMTAB)                     0x148
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000005 (STRTAB)                     0x918
 0x0000000a (STRSZ)                      1011 (bytes)
 0x00000004 (HASH)                       0xd0c
 0x00000001 (NEEDED)                     Shared library: [libdl.so]
 0x00000001 (NEEDED)                     Shared library: [libc.so]
 0x0000001a (FINI_ARRAY)                 0x238458
 0x0000001c (FINI_ARRAYSZ)               8 (bytes)
 0x00000019 (INIT_ARRAY)                 0x238460
 0x0000001b (INIT_ARRAYSZ)               16 (bytes)
 0x00000020 (PREINIT_ARRAY)              0x238470
 0x00000021 (PREINIT_ARRAYSZ)            0x8
 0x0000001e (FLAGS)                      BIND_NOW
 0x6ffffffb (FLAGS_1)                    Flags: NOW
 0x6ffffff0 (VERSYM)                     0x108c
 0x6ffffffe (VERNEED)                    0x1188
 0x6fffffff (VERNEEDNUM)                 2
 0x00000000 (NULL)                       0x0
```

 As you can see from the error above, the type corresponds to DT_VERNEED.

From [THIS](http://docs.oracle.com/cd/E23824_01/html/819-0690/chapter6-42444.html#scrolltoc) document:

>DT_RPATH

This element holds the string table offset of a null-terminated search library search path string, discussed in "Shared Object Dependencies." The offset is an index into the table recorded in the DT_STRTAB entry. DT_RPATH may give a string that holds a list of directories, separated by colons (:). All LD_LIBRARY_PATH directories are searched after those from DT_RPATH.

DT_RPATH此元素保存以“共享对象依赖关系”中讨论的以null结尾的搜索库搜索路径字符串的字符串表偏移量。偏移量是DT_STRTAB条目中记录的表的索引。DT_RPATH可以给出一个包含目录列表的字符串，以冒号（:)分隔。在DT_RPATH之后搜索所有LD_LIBRARY_PATH目录。

# Q: So how do you solve or deal with these issues?

There are essentially 3 ways to deal with this:

1. the quick
2. the bad
3. the ugly

## The Quick (you don't have any sources or just can't be bothered)

Use an "ELF cleaner" to remove the offending DT entries from a all your binaries. This is an easy and quick remedy, especially when you don't have the sources to recompile them properly for your system. There are at least [two cleaners](https://github.com/kost/android-elf-cleaner) out there that you can use.

## The Bad (you have the sources)

Is the right way to do it, because you'll become a bad-ass ARM cross compiler guru in the process of getting it to work. You basically need to find and tune the compiler settings in the Makefiles used.

From [here](https://gitlab.com/jbwhips883/termux-packages):

[解决方法：https://github.com/kost/android-elf-cleaner](https://github.com/kost/android-elf-cleaner)

>The Android linker (/system/bin/linker) does not support RPATH or RUNPATH, so we set LD_LIBRARY_PATH=$USR/lib and try to avoid building useless rpath entries with --disable-rpath configure flags. Another option to avoid depending on LD_LIBRARY_PATH would be supplying a custom linker - this is not done due to the overhead of maintaining a custom linker.

## The Ugly (You just want your app to work with any dirty binary.)

You tell your Java app not to freak out when checking for null in error handlers and instead get fed these warnings, possibly causing fatal exceptions. Use something like:

```java
class OpensslErrorThread extends Thread {
    @Override
    public void run() {
        try {
            while(true){
                String line = opensslStderr.readLine();
                if(line == null){
                    // OK
                    return;
                }
                if(line.contains("unused DT entry")){
                    Log.i(TAG, "Ignoring \"unused DT entry\" error from openssl: " + line);
                } else {
                    // throw exception!
                    break;
                }
            }
        } catch(Exception e) {
            Log.e(TAG, "Exception!")
        }
    }
}
```

This is very bad and ugly as it doesn't solve anything, while bloating your code. In addition, the warnings are there for a reason, and that is that in future AOS versions, this will become a full fledged error!

# Q. What else?

Many changes in the API's between 18-25 (J to N) has been made in way the Android kernel and libraries are compiled. I cannot provide a remotely close explanation of all that, but perhaps this will help guide you in the right direction. The best sources is of course looking in the Android sources and documentation itself.

For example, [HERE](https://github.com/android/platform_bionic/blob/master/android-changes-for-ndk-developers.md) or [HERE](https://developer.android.com/ndk/guides/standalone_toolchain.html).

And finally the full list:

```shell
Name                    Value           d_un            Executable              Shared Object
---------------------------------------------------------------------------------------------
DT_NULL                 0               Ignored         Mandatory               Mandatory
DT_NEEDED               1               d_val           Optional                Optional
DT_PLTRELSZ             2               d_val           Optional                Optional
DT_PLTGOT               3               d_ptr           Optional                Optional
DT_HASH                 4               d_ptr           Mandatory               Mandatory
DT_STRTAB               5               d_ptr           Mandatory               Mandatory
DT_SYMTAB               6               d_ptr           Mandatory               Mandatory
DT_RELA                 7               d_ptr           Mandatory               Optional
DT_RELASZ               8               d_val           Mandatory               Optional
DT_RELAENT              9               d_val           Mandatory               Optional
DT_STRSZ                0x0a (10)       d_val           Mandatory               Mandatory
DT_SYMENT               0x0b (11)       d_val           Mandatory               Mandatory
DT_INIT                 0x0c (12)       d_ptr           Optional                Optional
DT_FINI                 0x0d (13)       d_ptr           Optional                Optional
DT_SONAME               0x0e (14)       d_val           Ignored                 Optional
DT_RPATH                0x0f (15)       d_val           Optional                Optional
DT_SYMBOLIC             0x10 (16)       Ignored         Ignored                 Optional
DT_REL                  0x11 (17)       d_ptr           Mandatory               Optional
DT_RELSZ                0x12 (18)       d_val           Mandatory               Optional
DT_RELENT               0x13 (19)       d_val           Mandatory               Optional
DT_PLTREL               0x14 (20)       d_val           Optional                Optional
DT_DEBUG                0x15 (21)       d_ptr           Optional                Ignored
DT_TEXTREL              0x16 (22)       Ignored         Optional                Optional
DT_JMPREL               0x17 (23)       d_ptr           Optional                Optional
DT_BIND_NOW             0x18 (24)       Ignored         Optional                Optional
DT_INIT_ARRAY           0x19 (25)       d_ptr           Optional                Optional
DT_FINI_ARRAY           0x1a (26)       d_ptr           Optional                Optional
DT_INIT_ARRAYSZ         0x1b (27)       d_val           Optional                Optional
DT_FINI_ARRAYSZ         0x1c (28)       d_val           Optional                Optional
DT_RUNPATH              0x1d (29)       d_val           Optional                Optional
DT_FLAGS                0x1e (30)       d_val           Optional                Optional
DT_ENCODING             0x1f (32)       Unspecified     Unspecified             Unspecified
DT_PREINIT_ARRAY        0x20 (32)       d_ptr           Optional                Ignored
DT_PREINIT_ARRAYSZ      0x21 (33)       d_val           Optional                Ignored
DT_MAXPOSTAGS           0x22 (34)       Unspecified     Unspecified             Unspecified
DT_LOOS                 0x6000000d      Unspecified     Unspecified             Unspecified
DT_SUNW_AUXILIARY       0x6000000d      d_ptr           Unspecified             Optional
DT_SUNW_RTLDINF         0x6000000e      d_ptr           Optional                Optional
DT_SUNW_FILTER          0x6000000e      d_ptr           Unspecified             Optional
DT_SUNW_CAP             0x60000010      d_ptr           Optional                Optional
DT_SUNW_SYMTAB          0x60000011      d_ptr           Optional                Optional
DT_SUNW_SYMSZ           0x60000012      d_val           Optional                Optional
DT_SUNW_ENCODING        0x60000013      Unspecified     Unspecified             Unspecified
DT_SUNW_SORTENT         0x60000013      d_val           Optional                Optional
DT_SUNW_SYMSORT         0x60000014      d_ptr           Optional                Optional
DT_SUNW_SYMSORTSZ       0x60000015      d_val           Optional                Optional
DT_SUNW_TLSSORT         0x60000016      d_ptr           Optional                Optional
DT_SUNW_TLSSORTSZ       0x60000017      d_val           Optional                Optional
DT_SUNW_CAPINFO         0x60000018      d_ptr           Optional                Optional
DT_SUNW_STRPAD          0x60000019      d_val           Optional                Optional
DT_SUNW_CAPCHAIN        0x6000001a      d_ptr           Optional                Optional
DT_SUNW_LDMACH          0x6000001b      d_val           Optional                Optional
DT_SUNW_CAPCHAINENT     0x6000001d      d_val           Optional                Optional
DT_SUNW_CAPCHAINSZ      0x6000001f      d_val           Optional                Optional
DT_HIOS                 0x6ffff000      Unspecified     Unspecified             Unspecified
DT_VALRNGLO             0x6ffffd00      Unspecified     Unspecified             Unspecified
DT_CHECKSUM             0x6ffffdf8      d_val           Optional                Optional
DT_PLTPADSZ             0x6ffffdf9      d_val           Optional                Optional
DT_MOVEENT              0x6ffffdfa      d_val           Optional                Optional
DT_MOVESZ               0x6ffffdfb      d_val           Optional                Optional
DT_POSFLAG_1            0x6ffffdfd      d_val           Optional                Optional
DT_SYMINSZ              0x6ffffdfe      d_val           Optional                Optional
DT_SYMINENT             0x6ffffdff      d_val           Optional                Optional
DT_VALRNGHI             0x6ffffdff      Unspecified     Unspecified             Unspecified
DT_ADDRRNGLO            0x6ffffe00      Unspecified     Unspecified             Unspecified
DT_CONFIG               0x6ffffefa      d_ptr           Optional                Optional
DT_DEPAUDIT             0x6ffffefb      d_ptr           Optional                Optional
DT_AUDIT                0x6ffffefc      d_ptr           Optional                Optional
DT_PLTPAD               0x6ffffefd      d_ptr           Optional                Optional
DT_MOVETAB              0x6ffffefe      d_ptr           Optional                Optional
DT_SYMINFO              0x6ffffeff      d_ptr           Optional                Optional
DT_ADDRRNGHI            0x6ffffeff      Unspecified     Unspecified             Unspecified
DT_RELACOUNT            0x6ffffff9      d_val           Optional                Optional
DT_RELCOUNT             0x6ffffffa      d_val           Optional                Optional
DT_FLAGS_1              0x6ffffffb      d_val           Optional                Optional
DT_VERDEF               0x6ffffffc      d_ptr           Optional                Optional
DT_VERDEFNUM            0x6ffffffd      d_val           Optional                Optional
DT_VERNEED              0x6ffffffe      d_ptr           Optional                Optional
DT_VERNEEDNUM           0x6fffffff      d_val           Optional                Optional
DT_LOPROC               0x70000000      Unspecified     Unspecified             Unspecified
DT_SPARC_REGISTER       0x70000001      d_val           Optional                Optional
DT_AUXILIARY            0x7ffffffd      d_val           Unspecified             Optional
DT_USED                 0x7ffffffe      d_val           Optional                Optional
DT_FILTER               0x7fffffff      d_val           Unspecified             Optional
DT_HIPROC               0x7fffffff      Unspecified     Unspecified             Unspecified
```
#  WARNING: linker: ./demo: unused DT entry: type 0x6ffffffe 解决！！

- demo.c

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("www.chinapyg.com!\n");
    return 0;
}
```

- Android.mk

```mk
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := demo 
LOCAL_SRC_FILES := ../demo.c 
LOCAL_ARM_MODE := arm
LOCAL_CFLAGS := -g
LOCAL_CFLAGS += -pie -fPIE
LOCAL_LDFLAGS += -pie -fPIE

include $(BUILD_EXECUTABLE)
```

- Application.mk

```mk
APP_OPTIM := release
APP_PLATFORM := android-14  
APP_ABI := armeabi-v7a
```

- NDK编译:

```bash
C:\Users\piao\Downloads\adbi\demo\jni
λ ndk-build.cmd
[armeabi-v7a] Compile arm    : demo <= demo.c
[armeabi-v7a] Executable     : demo
[armeabi-v7a] Install        : demo => libs/armeabi-v7a/demo
```

经过测试~~

此警告只会在5.0以上系统出现：

```bash
root@ido:/data/local/tmp # ./demo
WARNING: linker: ./demo: unused DT entry: type 0x6ffffffe arg 0x41c
WARNING: linker: ./demo: unused DT entry: type 0x6fffffff arg 0x1
www.chinapyg.com!
```

解决方法：[https://github.com/kost/android-elf-cleaner](https://github.com/kost/android-elf-cleaner)

在Ubuntu16.04编译后:

```bash
piao@piaopiao:~/Desktop/android-elf-cleaner$ make
g++   -std=c++14 -Wall -Wextra -pedantic -Werror android-elf-cleaner.cpp -o android-elf-cleaner
```

然后执行:

```bash
piao@piaopiao:~/Desktop/android-elf-cleaner$ ./android-elf-cleaner demo
./android-elf-cleaner: Removing the DT_VERNEEDED dynamic section entry from 'demo'
./android-elf-cleaner: Removing the DT_VERNEEDNUM dynamic section entry from 'demo'
piao@piaopiao:~/Desktop/android-elf-cleaner$
```

再次push到手机里面运行:

```bash
root@ido:/data/local/tmp # ./demo
www.chinapyg.com!
```

# Reference

[https://stackoverflow.com/questions/33206409/unused-dt-entry-type-0x1d-arg](https://stackoverflow.com/questions/33206409/unused-dt-entry-type-0x1d-arg)
[NDK升级遇到的一些问题汇总](http://blog.sina.com.cn/s/blog_602f87700102x01t.html)
[ WARNING: linker: ./demo: unused DT entry: type 0x6ffffffe 解决！！](https://www.dllhook.com/post/211.html)