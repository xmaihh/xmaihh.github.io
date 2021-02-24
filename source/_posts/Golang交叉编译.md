---
title: Golang交叉编译
date: 2019-05-16 17:51:34
categories: Golang
tags: Golang
toc: true
description: In spite of you and me and the whole silly world going to pieces around us, I love you.
---

Golang交叉编译，一个平台环境下生成其他平台的可执行程序。

- GOOS：目标平台的操作系统（darwin、freebsd、linux、windows） 
- GOARCH：目标平台的体系架构（386、amd64、arm） 
- CGO_ENABLED:  开启/禁止C与Go混编(0,1)

Mac 下编译 Linux 和 Windows 64位可执行程序

```shell
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go 

$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
```

Linux 下编译 Mac 和 Windows 64位可执行程序

```shell
$ CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go 

$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
```

Windows 下编译 Mac 和 Linux 64位可执行程序

```shell
SET CGO_ENABLED=0 
SET GOOS=darwin 
SET GOARCH=amd64 
go build main.go 

SET CGO_ENABLED=0 
SET GOOS=linux 
SET GOARCH=amd64 
go build main.go
```





# Reference

[go交叉编译arm上的程序](<https://blog.csdn.net/huyongtq/article/details/81805065>)