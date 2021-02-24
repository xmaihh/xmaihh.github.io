---
title: Java中double转int类型按四舍五入取整
date: 2019-12-11 17:44:53
categories: Android Framework
tags: [Java]
toc: false
description: どんな人でも、 不安がきれいに 消えるということはない。
---

Java中的double转int类型，小数点后面抹零，只取小数点前的整数
所以被踩了丢失精度的坑，后续在将小数的double转换成为int的时候，一定要注意，小数点后面的部分是自动抹去的。
例如：
```java
public static void main(String[] args) {
		double d =1.76;		
		System.out.println((int)d);
		double f =1.16;		
		System.out.println((int)f);
	}
```
输出是:
```java
  1
  1
```

正确double转int按四舍五入取整

```java

public static void main(String[] args) {
        System.out.println("向上取整:" + (int) Math.ceil(96.1));// 97 (去掉小数凑整:不管小数是多少，都进一)
        System.out.println("向下取整" + (int) Math.floor(96.8));// 96 (去掉小数凑整:不论小数是多少，都不进位)
        System.out.println("四舍五入取整:" + Math.round(96.1));// 96 (这个好理解，不解释)
        System.out.println("四舍五入取整:" + Math.round(96.8));// 97
}
```

输出结果为：

```java
  向上取整: 97
  向下取整: 96
  四舍五入取整:96
  四舍五人取整:97
```

