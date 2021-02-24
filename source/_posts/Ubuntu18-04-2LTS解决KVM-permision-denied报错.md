---
title: Ubuntu18.04.2LTS解决KVM permision denied报错
date: 2019-05-10 11:21:11
categories: Linux
tags: Ubuntu
toc: false
description: Forgiveness is the fragrance that the violet sheds on the heel that has crushed it.
---

我在尝试在Ubuntu18.04.2LTS的AndroidStudio上运行Android Virtual Device(AVD) 时，遇到这个报错：`/dev/kvm device: permission denied`。

![](https://i.loli.net/2019/05/10/5cd4f15ca663e.png)

解决：

1. Install `qemu-kvm`

```shell
	$ sudo apt install qemu-kvm
```

2. 添加当前用户到kvm组

```shell
	$ sudo adduser <username> kvm
```

3. 添加当前用户权限

```shell
	$ sudo chown <username> /dev/kvm
```

![](https://i.loli.net/2019/05/10/5cd4f3a098847.png)

完成所有步骤后，可以成功启动AVD了。

![](https://i.loli.net/2019/05/10/5cd4f4139318f.png)


# Reference

[How to fix KVM permission denied error on Ubuntu 18.04](https://blog.chirathr.com/android/ubuntu/2018/08/13/fix-avd-error-ubuntu-18-04/)