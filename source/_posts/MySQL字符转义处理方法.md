---
title: MySQL字符转义处理方法
date: 2019-06-18 14:44:38
categories: SQL
tags: SQL
toc: true
description: You have to believe in yourself. That's the secret.
---

# MySQL转义字符

|        |        |        |
| ------ | ------ | ------ |
|  \0	| ASCII 0(NUL)字符 ||
| \'	| ASCII 39 单引号(‘'’) ||
| \"	| ASCII 34 双引号(‘"’) ||
| \b	| ASCII 8 退格符 ||
| \n	| ASCII 10 换行符 ||
| \r	| ASCII 13 回车符 ||
| \t	| ASCII 9 制表符(TAB) ||
| \Z	| ASCII 26(控制（Ctrl）-Z)。该字符可以编码为‘\Z’，以允许你解决在Windows中ASCII 26代表文件结尾这一问题 ||
| \\	| 反斜线(‘\’)字符 ||
| \%	| ‘%’字符 ||
|  \_	| ‘_’字符 ||



比如,对单引号、双引号和反斜杠的转义处理

```python
    origin = origin.replace("\\", "\\\\")
    origin = origin.replace("'", "\\'")
    origin = origin.replace('"', '\\"')
```

# 判断是否需要转义方法

判断字符串是否含有特殊符号,Java方法:
```java
/**
 * 判断内容是否需要进行转义
 * @param x
 * @return
 */
public static boolean isNeedEscape(String x) {
    boolean needsHexEscape = false;
    if(StringUtils.isBlank(x)) {
        return needsHexEscape;
    }
    int stringLength = x.length();
    int i = 0;
    do
    {
        if(i >= stringLength)
            break;
        char c = x.charAt(i);
        switch(c)
        {
        case 0: // '\0'
            needsHexEscape = true;
            break;

        case 10: // '\n'
            needsHexEscape = true;
            break;

        case 13: // '\r'
            needsHexEscape = true;
            break;

        case 92: // '\\'
            needsHexEscape = true;
            break;

        case 39: // '\''
            needsHexEscape = true;
            break;

        case 34: // '"'
            needsHexEscape = true;
            break;

        case 26: // '\032'
            needsHexEscape = true;
            break;
        }
        if(needsHexEscape)
            break;
        i++;
    } while(true);
    return needsHexEscape;
}
```

对字符转义处理:

```java
/**
 * 对mysql字符进行转义
 * @param x
 * @return
 */
public static String escapeString(String x) {
    if(Strings.isBlank(x)) {
        return x;
    }
    if(!isNeedEscape(x)) {
        return x;
    }
    int stringLength = x.length();
    String parameterAsString = x;
    StringBuffer buf = new StringBuffer((int)((double)x.length() * 1.1000000000000001D));
    // 可以指定结果前后追加单引号：'
    //buf.append('\'');
    for(int i = 0; i < stringLength; i++)
    {
        char c = x.charAt(i);
        switch(c)
        {
        default:
            break;

        case 0: // '\0'
            buf.append('\\');
            buf.append('0');
            continue;

        case 10: // '\n'
            buf.append('\\');
            buf.append('n');
            continue;

        case 13: // '\r'
            buf.append('\\');
            buf.append('r');
            continue;

        case 92: // '\\'
            buf.append('\\');
            buf.append('\\');
            continue;

        case 39: // '\''
            buf.append('\\');
            buf.append('\'');
            continue;

        case 34: // '"'
            buf.append('\\');
            buf.append('"');
            continue;

        case 26: // '\032'
            buf.append('\\');
            buf.append('Z');
            continue;

        }
        buf.append(c);
    }
    // 可以指定结果前后追加单引号：'
    //buf.append('\'');
    parameterAsString = buf.toString();
    return parameterAsString;
}
```

> `MySQL`语句没有处理好转义字符，通常会报错:`Error code 1064: Syntax error`

# Python中`re.escape()`函数

re.escape()是用来处理需要进行正则表达式匹配的字符串中，本身包含正则表达式元字符的情况，这个函数的处理方法也很简单，就是对字符串中所有的非字母（ASCII letters）、数字（numbers）及下划线（'_'）的字符前都加反斜线（’\‘），这样进行转义处理。

```python
import re
a = "he's pen."
print(a)  # he's pen.
a = re.escape(a)
print(a)  # he\'s\ pen\.
a = a.replace('\\', '')
print(a)  # he's pen.
```


# Reference

[MySQL字符转义涉及的问题及解决](https://www.jyoryo.com/index.php/archives/21.html)
[Python:对字符串中非字母、数字或下划线字符进行转义](https://www.polarxiong.com/archives/Python-%E5%AF%B9%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E9%9D%9E%E5%AD%97%E6%AF%8D-%E6%95%B0%E5%AD%97%E6%88%96%E4%B8%8B%E5%88%92%E7%BA%BF%E5%AD%97%E7%AC%A6%E8%BF%9B%E8%A1%8C%E8%BD%AC%E4%B9%89.html)