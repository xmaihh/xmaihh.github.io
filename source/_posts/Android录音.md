---
title: Android录音
date: 2019-07-30 21:23:26
categories: Android 
tags: [Android]
toc: true
description: A man can fail many times, but he isn't a failure until he begins to blame somebody else.
---

# 介绍 Android 中录音功能的实现

## 录音方法
Android 中的录音主要有两种方式 MediaRecorder 和 AudioRecord

- MediaRecorder（基于文件）

    可以录制音、视频；
    
    封装了录制、编码、压缩、线程等功能，直接生成可播放的音频文件；
    
    优点：封装度高，操作简单
    
    缺点：编码格式有限，.aac  .amr  .3gp，但是没有 mp3、wav 格式
    
- AudioRecord（基于字节流）

    ​    只能录制音频；

    ​    输出的是 PCM 的声音数据，如果保存成文件是不能直接播放的，需要编码；

    ​    可以捕获音频流，边录制边处理，比如编码、变声、添加背景音乐。

    ​    优点：更灵活

    ​    缺点：需自行处理编码、开线程等工作

    ​    应用场景：语音聊天、汤姆猫、K歌...


>PCM：Pulse Code Modulation（脉冲编码调制），是对连续变化的模拟信号进行抽样、量化和编码产生的数字信号。
它不是一种音频格式，它是声音文件的元数据，也就是声音的内容，没有文件头。经过某种格式的压缩、编码算法处理以后，再加上这种格式的文件头，才是这种格式的音频文件。

## 音频参数

- 采样频率：

​       自然界的声音转换成数字格式时，要对它进行采样，每秒钟采样的次数就是采样率。就好比电影的1秒24帧画面。最常用：44.1kHz。


- 采样位数：

​       一个采样样本用多少位二进制数编码，最常用：16位。


- 声道数：

​       分为单声道和双声道，双声道又叫立体声，双声道音频文件比单声道大一倍。

- 比特率（码率）：

​       每秒钟音频文件所占的 bit 数。单位 ：kbps（每秒千比特数）。比特率（原始音频 PCM） = 采样频率 x 采样位数  x  声道数，这是未经压缩的比特率，压缩后会远小于这个值。
​    
​       采用44.1kHz采样频率、16位采样位数、双声道编码的原始音频 PCM 比特率为：1411.2 kbps 。而最常见的 mp3 格式的比特率为：128kbps，约 1MB/分钟。

- 编码格式：

​       将原始音频 PCM 采用特定压缩算法处理后，加上文件头，所保存成的文件的格式。例如 mp3、wav、aac...

## 编码格式

- mp3

​       是当今最流行的一种数字音频编码和有损压缩格式，就是将 PCM 通过算法进行压缩，常规 mp3 文件约为 1MB/分钟。

- aac

​        是 mp3 的下一代格式，也是有损压缩，相对于mp3，aac 格式的音频更佳，文件更小。ios 平台也支持，跨平台性好。

- wav

​        最流行的非压缩数据格式，微软开发。

-  amr

​        压缩比比较大，但相对其他的压缩格式质量比较差，多用于人声，通话录音。

>riff：一种文件描述的格式，wav文件就采用了riff描述，前面44字节就是 riff 描述内容，就是文件头。

## MediaRecorder

 首先在 AndroidManifest 配置文件中添加录音权限：
```xml
        <uses-permission android:name="android.permission.RECORD_AUDIO"/>
```
Android 6.0 以上还要动态获取权限。

```java
 // 录音对象声明
    private MediaRecorder mRecorder;
    
    private void startRecording() {
        // 创建录音对象
        mRecorder = new MediaRecorder();
        // 设置声音来源 MIC 即手机麦克风
        mRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
        // 设置音频格式 aac
        mRecorder.setOutputFormat(MediaRecorder.OutputFormat.AAC_ADTS);
        // 设置录音文件
        mRecorder.setOutputFile(getExternalCacheDir() + "/demo.aac");
        // 设置编码器
        mRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC);
        
        try {
            // 准备录音
            mRecorder.prepare();
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 开始录音
        mRecorder.start();
    }
```
录音是耗时操作，但是，由于` MediaRecorder `已经封装了线程，故以上代码放在主线程即可。
`MediaRecorder`是很占资源的，使用完毕需要释放掉：

 ```java
 private void stopRecording() {
        // 停止录音
        mRecorder.stop();
        // 释放资源
        mRecorder.release();
        // 引用置空
        mRecorder = null;
    }
 ```

## AudioRecord

首先在 AndroidManifest 配置文件中添加录音权限：

```xml
        <uses-permission android:name="android.permission.RECORD_AUDIO"/>
```

当然 Android 6.0 以上还要动态获取权限，具体实现方式请百度之。

```java
    // 录音状态
    private boolean isRecording = true;
    
    private void startRecording(){
        // 耗时操作要开线程
        new Thread(){
            @Override
            public void run() {
                // 音源
                int audioSource = MediaRecorder.AudioSource.MIC;
                // 采样率
                int sampleRate = 44100;
                // 声道数
                int channelConfig = AudioFormat.CHANNEL_IN_STEREO;//双声道
                // 采样位数
                int audioFormat = AudioFormat.ENCODING_PCM_16BIT;
                // 获取最小缓存区大小
                int minBufferSize = AudioRecord.getMinBufferSize(sampleRate, channelConfig, audioFormat);
                // 创建录音对象
                AudioRecord audioRecord = new AudioRecord(audioSource, sampleRate, channelConfig, audioFormat, minBufferSize);
                try {
                    // 创建随机读写流
                    RandomAccessFile raf = new RandomAccessFile(getExternalCacheDir() + "/demo.wav", "rw");
                    // 留出文件头的位置
                    raf.seek(44);
                    byte[] buffer = new byte[minBufferSize];
    
                    // 录音中
                    audioRecord.startRecording();
                    isRecording = true;
                    while (isRecording) {
                        int readSize = audioRecord.read(buffer, 0, minBufferSize);
                        raf.write(buffer,0,readSize);
                    }
                    
                    // 录音停止
                    audioRecord.stop();
                    audioRecord.release();
                    
                    // 写文件头
                    WriteWaveFileHeader(raf, raf.length(),sampleRate,2,sampleRate*16*2/8);
                    raf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }.start();
    }
```

```java
/**
     * 为 wav 文件添加文件头，前提是在头部预留了 44字节空间
     *
     * @param raf
     *              随机读写流
     * @param fileLength
     *              文件总长
     * @param sampleRate
     *              采样率
     * @param channels
     *              声道数量
     * @param byteRate
     *              码率 = 采样率 * 采样位数 * 声道数 / 8
     * @throws IOException
     */
    private void WriteWaveFileHeader(RandomAccessFile raf, long fileLength, long sampleRate, int channels, long byteRate) throws IOException {
        long totalDataLen = fileLength + 36;
        byte[] header = new byte[44];
        header[0] = 'R'; // RIFF/WAVE header
        header[1] = 'I';
        header[2] = 'F';
        header[3] = 'F';
        header[4] = (byte) (totalDataLen & 0xff);
        header[5] = (byte) ((totalDataLen >> 8) & 0xff);
        header[6] = (byte) ((totalDataLen >> 16) & 0xff);
        header[7] = (byte) ((totalDataLen >> 24) & 0xff);
        header[8] = 'W';
        header[9] = 'A';
        header[10] = 'V';
        header[11] = 'E';
        header[12] = 'f'; // 'fmt ' chunk
        header[13] = 'm';
        header[14] = 't';
        header[15] = ' ';
        header[16] = 16; // 4 bytes: size of 'fmt ' chunk
        header[17] = 0;
        header[18] = 0;
        header[19] = 0;
        header[20] = 1; // format = 1
        header[21] = 0;
        header[22] = (byte) channels;
        header[23] = 0;
        header[24] = (byte) (sampleRate & 0xff);
        header[25] = (byte) ((sampleRate >> 8) & 0xff);
        header[26] = (byte) ((sampleRate >> 16) & 0xff);
        header[27] = (byte) ((sampleRate >> 24) & 0xff);
        header[28] = (byte) (byteRate & 0xff);
        header[29] = (byte) ((byteRate >> 8) & 0xff);
        header[30] = (byte) ((byteRate >> 16) & 0xff);
        header[31] = (byte) ((byteRate >> 24) & 0xff);
        header[32] = (byte) (2 * 16 / 8); // block align
        header[33] = 0;
        header[34] = 16; // bits per sample
        header[35] = 0;
        header[36] = 'd';
        header[37] = 'a';
        header[38] = 't';
        header[39] = 'a';
        header[40] = (byte) (fileLength & 0xff);
        header[41] = (byte) ((fileLength >> 8) & 0xff);
        header[42] = (byte) ((fileLength >> 16) & 0xff);
        header[43] = (byte) ((fileLength >> 24) & 0xff);
        raf.seek(0);
        raf.write(header, 0, 44);
    } 
```

 以下是停止录音的方法：

```java
   private void stopRecording() {
        // 停止录音
        isRecording = false;
    }
```
## 边录边播（AudioRecord + AudioTrack）

```java
 // 录音状态
    private boolean isRecording = true;
    
    private void start() {
        // 耗时操作要开线程
        new Thread() {
            @Override
            public void run() {
                
                // 音源
                int audioSource = MediaRecorder.AudioSource.MIC;
                // 采样率
                int sampleRate = 8000;
                // 声道数
                int channelConfig = AudioFormat.CHANNEL_IN_STEREO;//双声道
                // 采样位数
                int audioFormat = AudioFormat.ENCODING_PCM_8BIT;
                // 获取录音最小缓存区大小
                int recorderBufferSize = AudioRecord.getMinBufferSize(sampleRate, channelConfig, audioFormat);
                // 创建录音对象
                AudioRecord audioRecord = new AudioRecord(audioSource, sampleRate, channelConfig, audioFormat, recorderBufferSize);
                
                // 音频类型
                int streamType = AudioManager.STREAM_MUSIC;
                // 静态音频还是音频流
                int mode = AudioTrack.MODE_STREAM;
                //  获取播放最小缓存区大小
                int playerBufferSize = AudioTrack.getMinBufferSize(sampleRate, channelConfig, audioFormat);
                // 创建播放对象
                AudioTrack audioTrack = new AudioTrack(streamType, sampleRate, channelConfig, audioFormat, playerBufferSize, mode);
                
                // 缓存区
                byte[] buffer = new byte[recorderBufferSize];
                
                // 录音中
                audioTrack.play();
                audioRecord.startRecording();
                isRecording = true;
                while (isRecording) {
                    audioRecord.read(buffer, 0, recorderBufferSize);
                    audioTrack.write(buffer, 0, buffer.length);
                }
                
                // 录音停止
                audioRecord.stop();
                audioTrack.stop();
                audioRecord.release();
                audioTrack.release();
            }
        }.start();
    }
```

 停止录音和播放：

 ```java
 private void stop() {
        // 停止录音
        isRecording = false;
    }
 ```
## 录制mp3格式音频

 众多周知，mp3 是跨平台性最好的音频格式，由于采用了压缩率更高的有损压缩算法，文件大小是大约每分钟1M，使其在网络中传输更快，占用存储空间也更少；与此同时，它的声音质量也不错，尤其是人声（相声、评书、脱口秀），当然追求无损音乐的除外。
 Android 中没有提供录制 mp3 的 API，需要使用开源库 lame，lame 是专门用于编码 mp3 的轻量高效的 c 代码库。由于采用 c 语言编写，故需要用到 jni。

### 下载lame库

- lame库下载(以下使用的是`lame` v3.100)
    https://sourceforge.net/projects/lame/files/lame/

### 源码导入

解压下载的lame库，把`libmp3lame`文件夹下后缀为`.c .h`的文件（不包括子文件夹i386和vector下的）复制到cpp/lame文件夹内，同时把`include`目录下的lame.h也复制到cpp/lame文件夹内，此时 lame文件夹内包含42个文件。
![](https://i.loli.net/2019/07/31/5d4157b88dbd619403.png)
（可参考https://github.com/xmaihh/MFSocket/tree/master/liblame/src/main/cpp/lame）

### 修改库文件

打开刚刚拷贝的lame库文件，修改：
1. util.h 文件，把 570 行的两处 ieee754_float32_t 改为 float  因为Android下并不支持该类型
2. set_get.h 文件，把头部的 #include <lame.h> 改为 #include "lame.h"
3. fft.c 文件，删除第47行  #include "vector/lame_intrin.h"
4. id3tag.c和machine.h两个文件里，將HAVE_STRCHR和HAVE_MEMCPY的ifdef结构体删除或者注释

```c
#ifdef STDC_HEADERS
# include <stdlib.h>
# include <string.h>
#else
/*# ifndef HAVE_STRCHR
#  define strchr index
#  define strrchr rindex
# endif*/
char   *strchr(), *strrchr();
/*# ifndef HAVE_MEMCPY
#  define memcpy(d, s, n) bcopy ((s), (d), (n))
#  define memmove(d, s, n) bcopy ((s), (d), (n))
# endif*/
#endif
```

可参考以下完整修改文件

```diff
diff --git a/VbrTag.c b/VbrTag.c
index 5800a44..36ee7b6 100644
--- a/VbrTag.c
+++ b/VbrTag.c
@@ -26,6 +26,8 @@
 # include <config.h>
 #endif
 
+#include <stdlib.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/bitstream.c b/bitstream.c
index aa35915..a2fe294 100644
--- a/bitstream.c
+++ b/bitstream.c
@@ -29,6 +29,7 @@
 
 #include <stdlib.h>
 #include <stdio.h>
+#include <string.h>
 
 #include "lame.h"
 #include "machine.h"
diff --git a/encoder.c b/encoder.c
index 48f46c7..437067f 100644
--- a/encoder.c
+++ b/encoder.c
@@ -30,6 +30,7 @@
 #endif
 
 
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/fft.c b/fft.c
index 4eea1ad..27febdb 100644
--- a/fft.c
+++ b/fft.c
@@ -44,7 +44,7 @@
 #include "util.h"
--- a/fft.c
+++ b/fft.c
@@ -44,7 +44,7 @@
 #include "util.h"
 #include "fft.h"
 
-#include "vector/lame_intrin.h"
+//#include "vector/lame_intrin.h"
 
 
 
diff --git a/id3tag.c b/id3tag.c
index ac48510..8f148b8 100644
--- a/id3tag.c
+++ b/id3tag.c
@@ -41,17 +41,20 @@
 # include <string.h>
 # include <ctype.h>
 #else
-# ifndef HAVE_STRCHR
-#  define strchr index
-#  define strrchr rindex
-# endif
+//# ifndef HAVE_STRCHR
+//#  define strchr index
+//#  define strrchr rindex
+//# endif
 char   *strchr(), *strrchr();
-# ifndef HAVE_MEMCPY
-#  define memcpy(d, s, n) bcopy ((s), (d), (n))
-# endif
+//# ifndef HAVE_MEMCPY
+//#  define memcpy(d, s, n) bcopy ((s), (d), (n))
+//# endif
 #endif
 
 
+#include <malloc.h>
+#include <string.h>
+#include <stdlib.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/lame.c b/lame.c
index cb82225..299fd56 100644
--- a/lame.c
+++ b/lame.c
@@ -31,6 +31,8 @@
 #endif
 
 
+#include <malloc.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 
diff --git a/machine.h b/machine.h
index bf6fff2..c675c20 100644
--- a/machine.h
+++ b/machine.h
@@ -31,15 +31,15 @@
 # include <stdlib.h>
 # include <string.h>
 #else
-# ifndef HAVE_STRCHR
-#  define strchr index
-#  define strrchr rindex
-# endif
+//# ifndef HAVE_STRCHR
+//#  define strchr index
+//#  define strrchr rindex
+//# endif
 char   *strchr(), *strrchr();
-# ifndef HAVE_MEMCPY
-#  define memcpy(d, s, n) bcopy ((s), (d), (n))
-#  define memmove(d, s, n) bcopy ((s), (d), (n))
-# endif
+//# ifndef HAVE_MEMCPY
+//#  define memcpy(d, s, n) bcopy ((s), (d), (n))
+//#  define memmove(d, s, n) bcopy ((s), (d), (n))
+//# endif
 #endif
 
 #if  defined(__riscos__)  &&  defined(FPA10)
diff --git a/newmdct.c b/newmdct.c
index 596cac9..ac98abd 100644
--- a/newmdct.c
+++ b/newmdct.c
@@ -30,6 +30,7 @@
 # include <config.h>
 #endif
 
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/psymodel.c b/psymodel.c
index 60076ee..1393c2a 100644
--- a/psymodel.c
+++ b/psymodel.c
@@ -145,7 +145,8 @@ blocktype_d[2]        block type to use for previous granule
 #endif
 
 #include <float.h>
-
+#include <stdlib.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/quantize.c b/quantize.c
index 9ba9c16..2906c00 100644
--- a/quantize.c
+++ b/quantize.c
@@ -28,6 +28,8 @@
 # include <config.h>
 #endif
 
+#include <stdlib.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/quantize_pvt.c b/quantize_pvt.c
:
 #endif
 
 #include <float.h>
-
+#include <stdlib.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/quantize.c b/quantize.c
index 9ba9c16..2906c00 100644
--- a/quantize.c
+++ b/quantize.c
@@ -28,6 +28,8 @@
 # include <config.h>
 #endif
 
+#include <stdlib.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/quantize_pvt.c b/quantize_pvt.c
index d8d6447..3cd9966 100644
--- a/quantize_pvt.c
+++ b/quantize_pvt.c
@@ -36,6 +36,7 @@
 #include "reservoir.h"
 #include "lame-analysis.h"
 #include <float.h>
+#include <string.h>
 
 
 #define NSATHSCALE 100  /* Assuming dynamic range=96dB, this value should be 92 */
diff --git a/set_get.h b/set_get.h
index 37e4bcd..99ab73c 100644
--- a/set_get.h
+++ b/set_get.h
@@ -21,7 +21,7 @@
 #ifndef __SET_GET_H__
 #define __SET_GET_H__
 
-#include <lame.h>
+#include "lame.h"
 
 #if defined(__cplusplus)
 extern  "C" {
diff --git a/takehiro.c b/takehiro.c
index 67aba1b..ca02f98 100644
--- a/takehiro.c
+++ b/takehiro.c
@@ -27,6 +27,7 @@
 #endif
 
 
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/util.c b/util.c
index 43b457c..e9255fe 100644
--- a/util.c
+++ b/util.c
@@ -27,6 +27,7 @@
 #endif
 
 #include <float.h>
+#include <malloc.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
diff --git a/util.h b/util.h
index 13f0cd4..b6bf306 100644
--- a/util.h
+++ b/util.h
@@ -567,7 +567,7 @@ extern  "C" {
 
 /* log/log10 approximations */
     extern void init_log_table(void);
-    extern ieee754_float32_t fast_log2(ieee754_float32_t x);
+    extern float fast_log2(float x);
 
     int     isResamplingNecessary(SessionConfig_t const* cfg);
 
diff --git a/vbrquantize.c b/vbrquantize.c
index 0f703b7..60834d3 100644
--- a/vbrquantize.c
+++ b/vbrquantize.c
@@ -27,6 +27,8 @@
 #endif
 
 
+#include <stdlib.h>
+#include <string.h>
 #include "lame.h"
 #include "machine.h"
 #include "encoder.h"
```

### 编写CmakeList.txt

```cmake
cmake_minimum_required(VERSION 3.6.0)
set(CURRENT_DIR ${CMAKE_SOURCE_DIR})
message("CURRENT_DIR:" ${CMAKE_SOURCE_DIR})
include_directories(${CMAKE_SOURCE_DIR}src/main/cpp/lame)

set(LAME_DIR src/main/cpp/lame)
message("LAME_DIR:" ${LAME_DIR})

aux_source_directory(src/main/cpp/lame SRC_LIST)

add_library(mp3lame
        SHARED
        src/main/cpp/MP3Recorder.c
        ${SRC_LIST})

#add_library(mp3lame
#        SHARED
#        src/main/cpp/MP3Recorder.c
#        src/main/cpp/lame/bitstream.c
#        src/main/cpp/lame/fft.c
#        src/main/cpp/lame/id3tag.c
#        src/main/cpp/lame/mpglib_interface.c
#        src/main/cpp/lame/presets.c
#        src/main/cpp/lame/quantize.c
#        src/main/cpp/lame/reservoir.c
#        src/main/cpp/lame/tables.c
#        src/main/cpp/lame/util.c
#        src/main/cpp/lame/VbrTag.c
#        src/main/cpp/lame/encoder.c
#        src/main/cpp/lame/gain_analysis.c
#        src/main/cpp/lame/lame.c
#        src/main/cpp/lame/newmdct.c
#        src/main/cpp/lame/psymodel.c
#        src/main/cpp/lame/quantize_pvt.c
#        src/main/cpp/lame/set_get.c
#        src/main/cpp/lame/takehiro.c
#        src/main/cpp/lame/vbrquantize.c
#        src/main/cpp/lame/version.c)


find_library( # Sets the name of the path variable.
        log-lib
        log)

target_link_libraries(mp3lame
        ${log-lib})

```

（可参考https://github.com/xmaihh/MFSocket/blob/master/liblame/CMakeLists.txt）

### 编写 java 类和 c 文件

```java
public class MP3Recorder {
    static {
     System.loadLibrary("mp3lame");   
  }
    /**
     * 初始化 lame编码器
     *
     * @param inSampleRate
     *              输入采样率
     * @param outChannel
     *              声道数
     * @param outSampleRate
     *              输出采样率
     * @param outBitrate
     *              比特率(kbps)
     * @param quality
     *              0~9，0最好
     */
    public static native void init(int inSampleRate, int outChannel, int outSampleRate, int outBitrate, int quality);
    
    /**
     *  编码，把 AudioRecord 录制的 PCM 数据转换成 mp3 格式
     *
     * @param buffer_l
     *          左声道输入数据
     * @param buffer_r
     *          右声道输入数据
     * @param samples
     *          输入数据的size
     * @param mp3buf
     *          输出数据
     * @return
     *          输出到mp3buf的byte数量
     */
    public static native int encode(short[] buffer_l, short[] buffer_r, int samples, byte[] mp3buf);
    
    /**
     *  刷写
     *
     * @param mp3buf
     *          mp3数据缓存区
     * @return
     *          返回刷写的数量
     */
    public static native int flush(byte[] mp3buf);
    
    /**
     * 关闭 lame 编码器，释放资源
     */
    public static native void close();
}
```
生成.h文件

[AndroidStudio快速生成jni头文件](https://xmaihh.github.io/2019/07/31/AndroidStudio%E5%BF%AB%E9%80%9F%E7%94%9F%E6%88%90jni%E5%A4%B4%E6%96%87%E4%BB%B6/)

编写MP3Recorder.c

```c
#include "lame/lame.h"
#include "MP3Recorder.h"

static lame_global_flags *glf = NULL;
/*
 * Class:     com_android_liblame_MP3Recorder
 * Method:    init
 * Signature: (IIIII)V
 */

JNIEXPORT void JNICALL Java_com_android_liblame_MP3Recorder_init
        (JNIEnv *env, jclass instance, jint inSamplerate, jint outChannel, jint outSamplerate,
         jint outBitrate, jint quality) {
    if (glf != NULL) {
        lame_close(glf);
        glf = NULL;
    }
    glf = lame_init();
    lame_set_in_samplerate(glf, inSamplerate);
    lame_set_num_channels(glf, outChannel);
    lame_set_out_samplerate(glf, outSamplerate);
    lame_set_brate(glf, outBitrate);
    lame_set_quality(glf, quality);
    lame_init_params(glf);

}

/*
 * Class:     com_android_liblame_MP3Recorder
 * Method:    encode
 * Signature: ([S[SI[B)I
 */
JNIEXPORT jint JNICALL Java_com_android_liblame_MP3Recorder_encode
        (JNIEnv *env, jclass instance, jshortArray buffer_l, jshortArray buffer_r, jint samples,
         jbyteArray mp3buf) {
    jshort *j_buffer_l = (*env)->GetShortArrayElements(env, buffer_l, NULL);

    jshort *j_buffer_r = (*env)->GetShortArrayElements(env, buffer_r, NULL);

    const jsize mp3buf_size = (*env)->GetArrayLength(env, mp3buf);
    jbyte *j_mp3buf = (*env)->GetByteArrayElements(env, mp3buf, NULL);

    int result = lame_encode_buffer(glf, j_buffer_l, j_buffer_r,
                                    samples, j_mp3buf, mp3buf_size);

    (*env)->ReleaseShortArrayElements(env, buffer_l, j_buffer_l, 0);
    (*env)->ReleaseShortArrayElements(env, buffer_r, j_buffer_r, 0);
    (*env)->ReleaseByteArrayElements(env, mp3buf, j_mp3buf, 0);

    return result;
}

/*
 * Class:     com_android_liblame_MP3Recorder
 * Method:    flush
 * Signature: ([B)I
 */
JNIEXPORT jint JNICALL Java_com_android_liblame_MP3Recorder_flush
        (JNIEnv *env, jclass instance, jbyteArray mp3buf) {
    const jsize mp3buf_size = (*env)->GetArrayLength(env, mp3buf);
    jbyte *j_mp3buf = (*env)->GetByteArrayElements(env, mp3buf, NULL);

    int result = lame_encode_flush(glf, j_mp3buf, mp3buf_size);

    (*env)->ReleaseByteArrayElements(env, mp3buf, j_mp3buf, 0);

    return result;
}

/*
 * Class:     com_android_liblame_MP3Recorder
 * Method:    close
 * Signature: ()V
 */
JNIEXPORT void JNICALL Java_com_android_liblame_MP3Recorder_close
        (JNIEnv *env, jclass instance) {
    lame_close(glf);
    glf = NULL;

}
```

### 配置`build.gradle`

```gradle
android {
	....
	...
	..
	.
   //*
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }

}
```

点一下小锤子 :hammer: MakeProject。

编译生成 so库。

###  录制MP3格式音频

```java
// 录音状态
    private boolean isRecording;
    
    //开始录音
    private void record() {
        new Thread() {
            @Override
            public void run() {
                // 音源
                int audioSource = MediaRecorder.AudioSource.MIC;
                // 采样率
                int sampleRate = 44100;
                // 声道
                int channelConfig = AudioFormat.CHANNEL_IN_MONO;//单声道
                // 采样位数
                int audioFormat = AudioFormat.ENCODING_PCM_16BIT;
                // 录音缓存区大小
                int bufferSizeInBytes;
                // 文件输出流
                FileOutputStream fos;
                // 录音最小缓存大小
                bufferSizeInBytes = AudioRecord.getMinBufferSize(sampleRate, channelConfig, audioFormat);
                AudioRecord audioRecord = new AudioRecord(audioSource, sampleRate, channelConfig, audioFormat, bufferSizeInBytes);
                try {
                    fos = new FileOutputStream(getExternalCacheDir() + "/demo.mp3");
                    MP3Recorder.init(sampleRate, 2, sampleRate, 128, 5);
                    short[] buffer = new short[bufferSizeInBytes];
                    byte[] mp3buffer = new byte[(int) (7200 + buffer.length * 1.25)];
                    audioRecord.startRecording();
                    isRecording = true;
                    while (isRecording && audioRecord.getRecordingState() == AudioRecord.RECORDSTATE_RECORDING) {
                        int readSize = audioRecord.read(buffer, 0, bufferSizeInBytes);
                        if (readSize > 0) {
                            int encodeSize = MP3Recorder.encode(buffer, buffer, readSize, mp3buffer);
                            if (encodeSize > 0) {
                                try {
                                    fos.write(mp3buffer, 0, encodeSize);
                                } catch (IOException e) {
                                    e.printStackTrace();
                                }
                            }
                        }
                    }
                    
                    int flushSize = MP3Recorder.flush(mp3buffer);
                    if (flushSize > 0) {
                        try {
                            fos.write(mp3buffer, 0, flushSize);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    try {
                        fos.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    audioRecord.stop();
                    audioRecord.release();
                    MP3Recorder.close();
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                }
            }
        }.start();
    }
    
 // 停止录音
    private void stop() {
        isRecording = false;
    }
```

# 音频解码

 想要转换音频格式（如 mp3格式转 wav格式）或者添加背景音乐，都需要解码声音文件。
 Android SDK 中提供了解码的 API，它就是 MediaCodec，也就是音频解码器，我们用它实现 mp3格式音频的解码：
把mp3格式转wav格式

```java
 public void startDecoding() {
        // 待解码声音文件路径
        String filePath = getExternalCacheDir() + "/bg.mp3";
        // 音/视频 提取器
        MediaExtractor mediaExtractor = new MediaExtractor(); try {
            mediaExtractor.setDataSource(filePath);
            // 分离音频用的，获取音频数据
            MediaFormat mediaFormat = mediaExtractor.getTrackFormat(0);
          /*// 获取采样率
            int sampleRate = mediaFormat.getInteger(MediaFormat.KEY_SAMPLE_RATE);
            // 获取声道数
            int channelCount = mediaFormat.getInteger(MediaFormat.KEY_CHANNEL_COUNT);
            // 获取时长
            long duration = mediaFormat.getLong(MediaFormat.KEY_DURATION);*/
            // 获取类型
            String mime = mediaFormat.getString(MediaFormat.KEY_MIME);
          /*System.out.println("sampleRate:" + sampleRate + ",channelCount:" +
            channelCount + ",duration:" + duration + ",mime:" + mime );*/
            
            // 选择音轨
            mediaExtractor.selectTrack(0);
            // 解码器
            MediaCodec mediaCodec = MediaCodec.createDecoderByType(mime);
            // 配置解码器
            mediaCodec.configure(mediaFormat, null, null, 0);
            // 开始解码
            mediaCodec.start();
            
            // 解码器在此缓存中获取输入数据
            ByteBuffer[] inputBuffers = mediaCodec.getInputBuffers();
            // 编码器将解码后的数据放入此缓存中，保存的是pcm数据
            ByteBuffer[] outputBuffers = mediaCodec.getOutputBuffers();
            // 用于描述解码得到的byte[]数据的相关信息
            MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
            
            // 创建随机读写流
            RandomAccessFile raf = new RandomAccessFile(getExternalCacheDir() + "/demo.wav", "rw");
            // 留出文件头的位置
            raf.seek(44);
            // 解码状态
            boolean isDecoding = true;
            main:
            while (isDecoding) {
                for (ByteBuffer ib : inputBuffers) {
                    // 获取输入缓存器
                    int inputIndex = mediaCodec.dequeueInputBuffer(-1);
                    if (inputIndex < 0) {
                        break main;
                    }
                    ByteBuffer inputBuffer = inputBuffers[inputIndex];
                    // 读取数据到输入缓存器
                    int sampleSize = mediaExtractor.readSampleData(inputBuffer, 0);
                    if (sampleSize < 0) {
                        isDecoding = false;
                    } else {
                        // 通知解码器输入了数据
                        mediaCodec.queueInputBuffer(inputIndex, 0, sampleSize, 0, 0);
                        // 移动到下一取样处
                        mediaExtractor.advance();
                    }
                }
                
                // 获取输出缓存器
                int outputIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 10000);
                if (outputIndex == -2) {
                    // 格式变了，重新获取一次
                    outputIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 10000);
                }
                // 拿到用于存放PCM数据的buffer
                ByteBuffer outputBuffer;
                // PCM数据
                byte[] chunckPCM;
                
                while (outputIndex >= 0) {
                    // 拿到用于存放PCM数据的buffer
                    outputBuffer = outputBuffers[outputIndex];
                    // bufferInfo 定义了次数据块的大小
                    chunckPCM = new byte[bufferInfo.size];
                    // 将buffer内的数据取出到字节数组中
                    outputBuffer.get(chunckPCM);
                    // 数据取出后，一定记得清空，mediaCodec是反复使用这些buffer的
                    outputBuffer.clear();
                    // 输出PCM数据到文件夹
                    raf.write(chunckPCM, 0, bufferInfo.size);
                    // 释放输出buffer，不然mediaCodec用完所有buffer后，就不再向外输出数据
                    mediaCodec.releaseOutputBuffer(outputIndex, false);
                    // 再次获取数据,如果没有数据则outputIndex=-1,结束循环
                    outputIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 10000);
                }
                
            }
            WriteWaveFileHeader(raf, raf.length(), 44100, 1, 44100 * 16 / 8);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 为 wav 文件添加文件头，前提是在头部预留了 44字节空间
     *
     * @param raf        随机读写流
     * @param fileLength 文件总长
     * @param sampleRate 采样率
     * @param channels   声道数量
     * @param byteRate   码率 = 采样率 * 采样位数 * 声道数 / 8
     * @throws IOException
     */
    private void WriteWaveFileHeader(RandomAccessFile raf, long fileLength, long sampleRate, int channels, long byteRate) throws IOException {
        long totalDataLen = fileLength + 36;
        byte[] header = new byte[44];
        header[0] = 'R'; // RIFF/WAVE header
        header[1] = 'I';
        header[2] = 'F';
        header[3] = 'F';
        header[4] = (byte) (totalDataLen & 0xff);
        header[5] = (byte) ((totalDataLen >> 8) & 0xff);
        header[6] = (byte) ((totalDataLen >> 16) & 0xff);
        header[7] = (byte) ((totalDataLen >> 24) & 0xff);
        header[8] = 'W';
        header[9] = 'A';
        header[10] = 'V';
        header[11] = 'E';
        header[12] = 'f'; // 'fmt ' chunk
        header[13] = 'm';
        header[14] = 't';
        header[15] = ' ';
        header[16] = 16; // 4 bytes: size of 'fmt ' chunk
        header[17] = 0;
        header[18] = 0;
        header[19] = 0;
        header[20] = 1; // format = 1
        header[21] = 0;
        header[22] = (byte) channels;
        header[23] = 0;
        header[24] = (byte) (sampleRate & 0xff);
        header[25] = (byte) ((sampleRate >> 8) & 0xff);
        header[26] = (byte) ((sampleRate >> 16) & 0xff);
        header[27] = (byte) ((sampleRate >> 24) & 0xff);
        header[28] = (byte) (byteRate & 0xff);
        header[29] = (byte) ((byteRate >> 8) & 0xff);
        header[30] = (byte) ((byteRate >> 16) & 0xff);
        header[31] = (byte) ((byteRate >> 24) & 0xff);
        header[32] = (byte) (2 * 16 / 8); // block align
        header[33] = 0;
        header[34] = 16; // bits per sample
        header[35] = 0;
        header[36] = 'd';
        header[37] = 'a';
        header[38] = 't';
        header[39] = 'a';
        header[40] = (byte) (fileLength & 0xff);
        header[41] = (byte) ((fileLength >> 8) & 0xff);
        header[42] = (byte) ((fileLength >> 16) & 0xff);
        header[43] = (byte) ((fileLength >> 24) & 0xff);
        raf.seek(0);
        raf.write(header, 0, 44);
    }
```

# 录音添加背景音乐并实时播放，类似K歌效果

K歌类 APP 都是录音加伴奏，这里实现 mp3 格式的背景音乐解码，与录音合并，并最终输出 mp3 格式的文件。即边录音边解码边合成，录音结束即合并结束。

```java

    // 录音状态
    private boolean isRecording;
    
    //开始录音
    private void record() {
        new Thread() {
            @Override
            public void run() {
                // 音源
                int audioSource = MediaRecorder.AudioSource.MIC;
                // 采样率
                int sampleRate = 44100;
                // 声道
                int channelConfig = AudioFormat.CHANNEL_IN_MONO;//单声道
                // 采样位数
                int audioFormat = AudioFormat.ENCODING_PCM_16BIT;
                // 录音最小缓存区大小
                int bufferSizeInBytes = AudioRecord.getMinBufferSize(sampleRate, channelConfig, audioFormat);
                // 录音对象
                AudioRecord audioRecord = new AudioRecord(audioSource, sampleRate, channelConfig, audioFormat, bufferSizeInBytes);
                try {
                    // 音轨提取器
                    MediaExtractor mediaExtractor = new MediaExtractor();
                    // 给音轨提取器设置文件路径
                    mediaExtractor.setDataSource(getExternalCacheDir() + "/bg.mp3");
                    // 获取音频格式信息
                    MediaFormat mediaFormat = mediaExtractor.getTrackFormat(0);
                    // 获取音频类型
                    String mime = mediaFormat.getString(MediaFormat.KEY_MIME);
                    // 选中音轨
                    mediaExtractor.selectTrack(0);
                    // 构造解码器
                    MediaCodec mediaCodec = MediaCodec.createDecoderByType(mime);
                    // 配置解码器
                    mediaCodec.configure(mediaFormat, null, null, 0);
                    // 开始解码
                    mediaCodec.start();
                    // 解码器在此缓存中获取输入数据
                    ByteBuffer[] inputBuffers = mediaCodec.getInputBuffers();
                    // 编码器将解码后的数据放入此缓存中，存放的是pcm数据
                    ByteBuffer[] outputBuffers = mediaCodec.getOutputBuffers();
                    // 用于描述解码得到的byte[]数据的相关信息
                    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                    // 背景音乐解码后的数据输出流，先存储，然后读取再和录音合并
                    FileOutputStream fos_wav = new FileOutputStream(getExternalCacheDir() + "/demo.wav");
                    // 读取处理好的背景音乐，和录音合并
                    RandomAccessFile raf = new RandomAccessFile(getExternalCacheDir() + "/demo.wav", "rw");
                    // 背景音字节数
                    long length = 0;
                    // 存储大小端
                    boolean isBigEnding = ByteOrder.nativeOrder() == ByteOrder.BIG_ENDIAN;
                    
                    // 录音缓存
                    short[] buffer = new short[bufferSizeInBytes];
                    // 最终生成的mp3数据的缓存器
                    byte[] mp3buffer = new byte[(int) (7200 + buffer.length * 1.25)];
                    // 开始录音
                    audioRecord.startRecording();
                    // 录音状态
                    isRecording = true;
                    
                    // 最终文件输出流
                    FileOutputStream fos = new FileOutputStream(getExternalCacheDir() + "/demo.mp3");
                    // mp3编码器初始化
                    MP3Recorder.init(sampleRate, 1, sampleRate, 128, 5);
                    
                    while (isRecording) {
                        // 读取录音
                        int readSize = audioRecord.read(buffer, 0, bufferSizeInBytes);
                        
                        // 解码背景音乐
                        for (ByteBuffer ib : inputBuffers) {
                            // 获取输入缓存器
                            int inputIndex = mediaCodec.dequeueInputBuffer(0);
                            if (inputIndex < 0) {
                                break;
                            }
                            ByteBuffer inputBuffer = inputBuffers[inputIndex];
                            // 读取数据到输入缓存器
                            int sampleSize = mediaExtractor.readSampleData(inputBuffer, 0);
                            if (sampleSize >= 0) {
                                // 通知解码器输入了数据
                                mediaCodec.queueInputBuffer(inputIndex, 0, sampleSize, 0, 0);
                                // 移动到下一取样处
                                mediaExtractor.advance();
                            }
                        }
                        // 输出缓存器
                        int outputIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
                        if (outputIndex == -2) {
                            // 格式变了，重新获取一次
                            outputIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
                        }
                        // 拿到用于存放PCM数据的buffer
                        ByteBuffer outputBuffer;
                        // PCM数据
                        byte[] chunckPCM;
                        // 循环读取解码数据
                        while (outputIndex >= 0) {
                            // 拿到用于存放PCM数据的buffer
                            outputBuffer = outputBuffers[outputIndex];
                            // bufferInfo 定义了数据块的大小
                            chunckPCM = new byte[bufferInfo.size];
                            // 将buffer内的数据取出到字节数组中
                            outputBuffer.get(chunckPCM);
                            // 数据取出后，一定记得清空，mediaCodec是反复使用这些buffer的
                            outputBuffer.clear();
                            // 输出PCM数据到文件夹
                            fos_wav.write(chunckPCM, 0, bufferInfo.size);
                            // 背景音字节数（解码后的）
                            length += bufferInfo.size;
                            // 释放输出buffer，不然mediaCodec用完所有buffer后，就不再向外输出数据
                            mediaCodec.releaseOutputBuffer(outputIndex, false);
                            // 再次获取数据,如果没有数据则outputIndex=-1,结束循环
                            outputIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
                        }
                        
                        // 混音
                        for (int i = 0; i < buffer.length; i++) {
                            if (raf.getFilePointer() >= length - 1) {
                                raf.seek(0);
                            }
                            if (isBigEnding) {
                                buffer[i] += (short) ((raf.read() << 8) + raf.read());
                            } else {
                                buffer[i] += (short) (raf.read() + (raf.read() << 8));
                            }
                        }
                        
                        if (readSize > 0) {
                            int encodeSize = MP3Recorder.encode(buffer, buffer, readSize, mp3buffer);
                            if (encodeSize > 0) {
                                try {
                                    fos.write(mp3buffer, 0, encodeSize);
                                } catch (IOException e) {
                                    e.printStackTrace();
                                }
                            }
                        }
                    }
                    
                    int flushSize = MP3Recorder.flush(mp3buffer);
                    if (flushSize > 0) {
                        try {
                            fos.write(mp3buffer, 0, flushSize);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    try {
                        fos.close();
                        raf.close();
                        fos_wav.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    audioRecord.stop();
                    audioRecord.release();
                    MP3Recorder.close();
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }.start();
    }
    
    public void stop() {
        isRecording = false;
    } 
```

# Reference
[Android 录音详解（一）—— MediaRecorder、AudioRecord、生成wav格式、边录边播](http://blog.elight.cn/?post=200)
[Android 录音详解（二）—— 录制 mp3 格式音频（ lame 库的编译及使用）](http://blog.elight.cn/?post=201)
[Android 录音详解（三）—— 音频解码](http://blog.elight.cn/?post=213)
[Android 录音详解（四）—— 录音添加背景音乐](http://blog.elight.cn/?post=220)
