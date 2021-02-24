---
title: Python项目生成依赖requirements.txt的两种方法
date: 2019-03-27 16:19:13
categories: Python
tags: [Python]
toc: true
description: How many loved your moments of glad grace,and loved your beauty with love false or true.
---

# 为什么要有requirements.txt

`requirements.txt`保存Python项目所依赖的类库。`Python`提供通过`requirements.txt`文件来进行项目中依赖的三方库进行整体安装导入。

`requirements.txt`文件的格式：

```python
requests==1.2.0
Flask==0.10.1
```

# 注意事项

- `#` -要求使用root权限直接以root用户使用命令或对执行的命令使用linux sudo
- `$` -要求给定的linux命令作为常规非特权用户执行

# 方法1：

```shell
$ pip freeze > requirements.txt
```

使用pip的`freeze`命令为您的项目生成一个`requirements.txt`文件：如果将其保存在requirements.txt中，则可以`pip install -r requirements.txt`来安装这些依赖。

>注意：`pip freeze`输出的是本地环境中所有三方包信息，但是会比`pip list`少几个包，因为`pip`，`wheel`，`setuptools`等包，是自带的而无法(un)install的，如果要显示所有包可以加上参数`-all`，即`pip freeze -all`

# 方法2：

使用`pipreqs`生成`requirements.txt`

```shell
$ pip install pipreqs
$ pipreqs /path/to/project
```

有关其他选项，请参阅https://github.com/bndr/pipreqs

>注意：`pipreqs`生成指定目录下的依赖类库

# 两种方法的区别

`pip freeze`保存环境中的所有包，包括那些在当前项目中不使用的包。（如果你没有virtualenv,类库就会很杂很多）
`pip freeze`仅保存在您的环境中使用`pip install`安装的软件包。
`pipreqs`它会根据当前目录下的项目的依赖来导出三方类库