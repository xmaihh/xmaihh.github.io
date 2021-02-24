---
title: Java中bit的操作技巧
date: 2018-12-10 19:34:17
categories: Android Framework
tags: [Android]
toc: true
description: Once you learn to quit, it becomes a habit.
---
Java中定义八种基本数据类型,最小到byte,然而最近在底层操作中遇到需要根据一个byte中的bit来操作,作此记录。
# 基本数据类型
- byte b; 1字节（8位） （-128~127）（-2的7次方到2的7次方-1）
- short s; 2字节（16位） （-32768~32767）（-2的15次方到2的15次方-1）
- char c;2字节（16位）（C语言中是1字节）可以存储一个汉字
- int i; 4字节（32位） （一个字长）（-2147483648~2147483647）（-2的31次方到2的31次方-1）
- long l; 8字节（64位） （-9223372036854774808~9223372036854774807）（-2的63次方到2的63次方-1）
- float f; 4字节（32位） （3.402823e+38 ~ 1.401298e-45）（e+38是乘以10的38次方，e-45是乘以10的负45次方）(-2^128 ~ +2^128)
- double d; 8字节（64位） （1.797693e+308~ 4.9000000e-324)(-2^1024 ~ +2^1024)
- boolean bool; false/true (理论上占用1bit,1/8字节，实际处理按1byte处理)

# bit操作技巧
 bit：位
    一个二进制数据0或1，是1bit；
Java中最小基本数据类型是byte,Java中要根据byte获得bit就要进行一些位操作可以根据` 1 byte = 8 bit`这个转化关系来。
##  将byte转换bit 
```
 /**
     * 将byte转换为一个长度为8的byte数组，数组每个值代表bit
     * <p> e.g.
     * byte b = 0x35; // 0011 0101
     * // 输出 [0, 0, 1, 1, 0, 1, 0, 1]
     *
     * @param b
     * @return
     */
    public static byte[] getBooleanArray(byte b) {
        byte[] array = new byte[8];
        for (int i = 7; i >= 0; i--) {
            array[i] = (byte) (b & 1);
            b = (byte) (b >> 1);
        }
        return array;
    }

 /**
     * 把byte转为字符串的bit
     * <p> e.g.
     * byte b = 0x35; // 0011 0101
     * // 输出 00110101
     *
     * @param b
     * @return
     */

    public static String byteToBit(byte b) {
        return ""
                + (byte) ((b >> 7) & 0x1) + (byte) ((b >> 6) & 0x1)
                + (byte) ((b >> 5) & 0x1) + (byte) ((b >> 4) & 0x1)
                + (byte) ((b >> 3) & 0x1) + (byte) ((b >> 2) & 0x1)
                + (byte) ((b >> 1) & 0x1) + (byte) ((b >> 0) & 0x1);
    }
```
> 输出内容就是各个 bit 位的 0 和 1 值，还有一种
JDK自带的方法`Integer.toBinaryString(0x35);`,会忽略前面的 0 ,输出为`110101`
## bit转换byte
```
 /**
     * 二进制字符串转byte
     *
     * @param byteStr
     * @return
     */
    public static byte decodeBinaryString(String byteStr) {
        int re, len;
        if (null == byteStr) {
            return 0;
        }
        len = byteStr.length();
        if (len != 4 && len != 8) {
            return 0;
        }
        if (len == 8) {// 8 bit处理
            if (byteStr.charAt(0) == '0') {// 正数
                re = Integer.parseInt(byteStr, 2);
            } else {// 负数
                re = Integer.parseInt(byteStr, 2) - 256;
            }
        } else {// 4 bit处理
            re = Integer.parseInt(byteStr, 2);
        }
        return (byte) re;
    }
```
