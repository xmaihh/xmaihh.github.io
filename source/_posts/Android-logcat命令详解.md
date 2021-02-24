---
title: Android logcat命令详解
date: 2019-02-21 13:42:59
categories: Android Framework
tags: [Android]
toc: true
description: Well, life isn't always what one likes, isn't it?
---
# Android Log系统
Android提供了一个灵活的logging系统，允许应用程序和系统组件等整个系统记录logging信息，它是独立于Linux Kernel的一个logging系统，kernel是通过`pr_info`、`printk`等存储，通过`dmesg`或`cat /proc/kmsg`获取。不过，Android logging 系统也是将信息存在内核缓存区。其结构如下：
![](https://i.loli.net/2019/02/21/5c6e43fee4322.png)

Logging system由如下几部分组成：

- 实现loging信息存储的kernel驱动和缓存区
- C，C++和Java 类添加与读取log
- 一个单独浏览log信息的程序（logcat）
- 能够查看和过滤来自主机的log信息（通过Android Studio 或者 DDMS）

# Android logcat介绍

logcat是android中的一个命令行工具，可以用于得到程序的log信息

## [android.util.Log](https://developer.android.com/reference/android/util/Log)

通常，我们使用android.util.Log类来打印日志消息，通过Logcat来查看打印的日志
Log类提供的方法，优先级按照从高到低的顺序如下:

|  方法   |   描述   |
|  ---  |  ---  |
| Log.e(String,String)(error) | 显示错误信息 |
| Log.w(String,String)(waning) | 显示警告信息 |
| Log.i(String,String)(information) | 显示一般信息 |
| Log.d(String,String)(debug) | 显示调试信息 |
| Log.v(String,String) (vervbose) | 显示全部信息 |

e.g.

```
// 通过代码打印log
Log.d("MainActivity", "LogCollectorThread is create");

 ...
// adb获取log
shell@android:/$ adb logcat
```

adb logcat 输出的日志:

```
D/:MainActivity: LogCollectorThread is create
```

## logcat缓冲区

### 缓冲区介绍
android log输出量巨大，特别是通信系统的log，因此，android把log输出到不同的缓冲区中，目前定义了四个log缓冲区：
1. Radio：输出通信系统的log
2. System：输出系统组件的log
3. Event：输出event模块的log
4. Main：所有java层的log，遗迹不属于上面3层的log
缓冲区主要给系统组件使用，一般的应用不需要关心，应用的log都输出到main缓冲区中
默认log输出（不指定缓冲区的情况下）是输出System和Main缓冲区的log

### 缓冲区模型

![](https://i.loli.net/2019/02/21/5c6e4e93a1230.png)

### 获取缓冲区命令

|  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参数&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   |   描述   |
|  ----   |  ----   |
| `-b<buffer>` | 加载一个可使用的日志缓冲区提供查看，默认值是`main` |

### 示例

```
adb logcat –b radio

adb logcat –b system

adb logcat –b events

adb logcat –b main
```

## logcat命令参数

### 参数说明

|   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参数&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    |    描述    |
|   ----    |    ---    |
| `-b <buffer>` | 加载一个可使用的日志缓冲区供查看，比如event和radio。默认值是main  具体查看[Viewing Alternative Log Buffers](https://developer.android.com/studio/command-line/logcat#alternativeBuffers).|
| `-c` | 清除缓冲区中的全部日志并退出（清除完后可以使用-g查看缓冲区） |
| `-d` | 将缓冲区的log转存到屏幕中然后退出 |
| `-f <filename>` | 将log输出到指定的文件中<文件名>.默认为标准输出（stdout） |
| `-g` | 打印日志缓冲区的大小并退出 |
| `-n <count>` | 设置日志的最大数目`<count>`，默认值是4，需要和`-r`选项一起使用 |
| `-r <kbytes>` | 没`<kbytes>`时输出日志，默认值是16，需要和`-f`选项一起使用 |
| `-s` | 设置过滤器 |
| `-v <format>` | 设置输出格式的日志消息。默认是短暂的格式。支持的格式列表 参看[Controlling Log Output Format](https://developer.android.com/studio/command-line/logcat#outputFormat) |

### 示例

```
//将缓冲区的log打印到屏幕并退出
adb logcat -d 
//清除缓冲区log（testCase运行前可以先清除一下）
adb logcat -c
//打印缓冲区大小并退出
adb logcat -g
//输出log
adb logcat -f /sdcard/log.txt -n 10 -r 1
```

## logcat格式化输出

日志消息包含一个元数据字段，除了标签和优先级，您可以修改输出显示一个特定的元数据字段格式的消息。为此，您使用-v选项来指定一个支持的输出格式。一下为支持的格式：

|   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;格式&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   |   说明   |
|   ---    |   ---   |
|  `brief`   | 显示优先级/标记和过程的PID发出的消息（默认格式） |
| `process`  | 只显示PID |
| `tag`     | 只显示优先级/标记 |
| `raw`      | 显示原始的日志消息，没有其他元数据字段 |
| `time`     | 调用显示日期、时间、优先级/标签和过程的PID发出消息 |
| `threadtime` | 调用显示日期、时间、优先级、标签遗迹PID TID线程发出的消息 |
| `long`     | 显示所有元数据字段与空白行和单独的消息 |

当logcat开始，指定想要输出格式`-v`选项：

```
[adb] logcat [-v <format>]
adb logcat –v thread
```

>只能指定一个输出格式`-v`

## 示例

![](https://i.loli.net/2019/02/21/5c6e571e84ea1.png)

# logcat优先级

优先级使用字符标识，一下优先级从低到高:

- V –Verbose(最低优先级)
- D – Debug
- I – Info
- W – Warning
- E – Error
- F – Fatal
- S – Silent

为了减少不想要日志的输出，可以建立一个过滤器
过滤语法：`tag：priority`

```
// 过滤TAG为ActivityManager输出级别大于I的日志与TAG为MyApp输出级别大于D的日志

adb logcat ActivityManager:I  My App:D *:S

 ...
// 设置过滤级别为W以上
adb logcat *:W
```

# Reference

[https://elinux.org/Android_Logging_System](https://elinux.org/Android_Logging_System)
[https://developer.android.com/studio/command-line/logcat](https://developer.android.com/studio/command-line/logcat)
[https://www.cnblogs.com/JianXu/p/5468839.html](https://www.cnblogs.com/JianXu/p/5468839.html)