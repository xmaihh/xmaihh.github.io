---
title: C-数据类型uint8_t、uint16_t、uint32_t、uint64_t、size_t、ssize_t
date: 2018-10-08 14:54:09
categories: C/C++
tags: [C/C++]
toc: false
description: Gaffes is accidentally say the truth.
---
## C语言6种基本数据类型

1. 整型：short、int、long
2. 浮点型：float、double
3. 字符类型：char

typedef用来定义关键字或标识符的别名
```
typedef double wages;
typedef wages salary;
```

一般整形对应的*_t类型为：
```
1字节     uint8_t
2字节     uint16_t
4字节     uint32_t
8字节     uint64_t
```
ssize_t是signed size_t，
size_t是标准C库中定义的，应为unsigned int。定义为typedef int ssize_t。
而ssize_t:这个数据类型用来表示可以被执行读写操作的数据块的大小.它和size_t类似,但必需是signed.意即：它表示的是sign size_t类型的。
size_t是一些C/C++标准在stddef.h中定义的。这个类型足以用来表示对象的大小。
size_t的真实类型与操作系统有关，在32位架构中被普遍定义为：
typedef   unsigned int size_t;
而在64位架构中被定义为：
typedef  unsigned long size_t;
size_t在32位架构上是4字节，在64位架构上是8字节，在不同架构上进行编译时需要注意这个问题。
而int在不同架构下都是4字节，与size_t不同；且int为带符号数，size_t为无符号数

附：C99标准中inttypes.h的内容
在C99标准中定义了这些数据类型，具体定义在：/usr/include/stdint.h    ISO C99: 7.18 Integer types
```
#ifndef __int8_t_defined  
# define __int8_t_defined  
typedef signed char             int8_t;   
typedef short int               int16_t;  
typedef int                     int32_t;  
# if __WORDSIZE == 64  
typedef long int                int64_t;  
# else  
__extension__  
typedef long long int           int64_t;  
# endif  
#endif      

typedef unsigned char           uint8_t;  
typedef unsigned short int      uint16_t;  
#ifndef __uint32_t_defined  
typedef unsigned int            uint32_t;  
# define __uint32_t_defined  
#endif  
#if __WORDSIZE == 64  
typedef unsigned long int       uint64_t;  
#else  
__extension__  
typedef unsigned long long int  uint64_t;  
#endif 
```
## 格式化输出
```
uint16_t %hu
uint32_t %u
uint64_t %llu
```
## uint8_t类型的输出
注意uint8_t的定义为
```
typedef unsigned char           uint8_t;
```
uint8_t实际上是一个char。所以输出uint8_t类型的变量实际上输出其对应的字符，而不是数值。例：
```
uint8_t num = 67;
cout << num << endl;
```
输出结果：C
