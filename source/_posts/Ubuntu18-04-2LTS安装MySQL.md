---
title: Ubuntu18.04.2LTS安装MySQL
date: 2019-05-17 16:32:28
categories: Linux
tags: [Ubuntu,SQL]
toc: true
description: Somewhere beyond right and wrong, there is a garden, I will meet you there.
---

[MySQL](https://www.mysql.com/)是一个开源数据库管理系统，通常作为流行的[LAMP](https://en.wikipedia.org/wiki/LAMP_(software_bundle)) （Linux，Apache，MySQL，PHP / Python / Perl）的一部分进行安装。 它使用关系数据库和SQL（结构化查询语言）来管理其数据。

# 环境

- Ubuntu 18.04.2 LTS
- MySQL 5.7.26
- MySQL Workbench (可视化＊可选)

# 安装

在 Ubuntu 18.04 中，默认情况下最新版本的 MySQL 包含在 APT 软件包存储库中,直接执行安装即可。
```shell
$ sudo apt update
$ sudo apt install mysql-server
```

# 配置

接下来配置MySQL，执行命令，运行MySQL附带的安全脚本，根据提示操作修改一些默认规则。

```shell
$ sudo mysql_secure_installation
```
![](https://i.loli.net/2019/05/17/5cde7753987ea12819.png)

![](https://i.loli.net/2019/05/17/5cde78771595f84396.png)

# 验证

安装完MySQL已经开始自动运行。 运行命令检查一下状态。

```shell
$ systemctl status mysql.service
```

![](https://i.loli.net/2019/05/17/5cde790f819fe19186.png)

显示如上信息说明mysql服务是正常的。

如果MySQL没有运行，你可以用`sudo systemctl start mysql`启动它。

尝试使用mysqladmin工具连接到数据库，该工具是允许您运行管理命令的客户端。 例如，该命令表示以root身份连接到MySQL（ -u root ），提示输入密码（ -p ）并返回该版本。

```shell
$ sudo mysqladmin -p -u root version
```
得到类似以下输出:
![](https://i.loli.net/2019/05/17/5cde7fec58af778411.png)

# 可视化

MySQL可视化工具软件安装

```shell
$ sudo apt-get install mysql-workbench
```
![](https://i.loli.net/2019/05/17/5cde82554de7656758.png)

在Ubuntu中打开MySQL Workbench软件可以看到有一个本地连接，点进去打开开报错了。

![](https://i.loli.net/2019/05/17/5cde84a6c61a011438.png)

>这跟前面配置有一些关系，前面删除了匿名用户和测试数据库，禁止了远程root用户登录。

这里来通过配置添加一个可以访问的数据库。

##  进入MySQL控制台

```shell
$ sudo mysql -uroot -p
```

![](https://i.loli.net/2019/05/20/5ce25bd4e87f614892.png)

## 新建数据库和用户

```shell
// 创建数据库ticket
$ CREATE DATABASE ticket;
// 创建用户xmai(密码Uf4bGZ53Ds*#) 并赋予其ticketDB数据库的所有权限
$ GRANT ALL PRIVILEGES ON ticket.* TO xmai@localhost IDENTIFIED BY "Uf4bGZ53Ds*#";
```

![](https://i.loli.net/2019/05/21/5ce3a2ddd80a069294.png)

>我们在前面MySQL配置时，增加了密码强度验证插件validate_password，相关参数设置的较为严格。使用了该插件会检查设置的密码是否符合当前设置的强度规则，若不满足则拒绝设置并报错类似`ERROR 1819 (HY000):Your password does not satisfy the current policy requirements`。前面我选择的是`[1] MEDIUM Length >=8 , numeric,mixed case, and special characters`

## 进行访问控制配置

```shell
// 允许xmai用户可以从任意机器上登入mysql
$ GRANT ALL PRIVILEGES ON ticket.* TO xmai@"%" IDENTIFIED BY "Uf4bGZ53Ds*#";
```

![](https://i.loli.net/2019/05/21/5ce3a337c4c2c27607.png)

配置完成。

打开MySQL workbench连接数据库

![](https://i.loli.net/2019/05/21/5ce3a3ca4f6b732789.png)

配置访问的数据库用户名和密码，确定。

![](https://i.loli.net/2019/05/21/5ce3a4dc6156450553.png)

## 允许局域网连接

修改连接 `bind-address` 配置项

```bash
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

修改`bind-address = 127.0.0.1`配置或者修改为`bind-address = 0.0.0.0`，来允许所有IP访问，或者输入一个你指定的IP地址

保存后使用以下命令重启mysql

```bash
$ sudo /etc/init.d/mysql restart
```

使用以下命令进入mysql修改访问权限

```bash
$ sudo mysql -uroot -p
输入密码
$ use mysql
//授权用户能进行远程连接
$ grant all privileges on *.* to root@"%" identified by "password" with grant option;
//刷新权限信息
$ flush privileges;
```

命令中的两个星号，第一个星号表示数据库名称，第二个星号表示该数据库下的某个表名称。写成两个星号表示所有的数据库都进行授权。
root表示授权root账号。
“%”表示授权的用户IP可以指定，这里代表任意的IP地址都能访问MySQL数据库。
“password”表示分配账号对应的密码，这里密码自己替换成你的mysql root帐号密码

所以此处可以写成：

```bash
grant all PRIVILEGES on testdabatase.testtable to username@'192.168.1.2' identified by 'user-pass';
```



## Error 1366: Incorrect string value: '\xE7\xA0\x94\xE5\x8F\x91...' for column 'departname' at row 1

插入数据时,报异常。MySQL的utf8编码最多3个字节，UTF-8编码有可能是两个、三个、四个字节，如Emoji表情或者某些特殊字符是4个字节,所以数据插不进去 。

一顿搜索后找到解决方案

1. 在mysql的安装目录下找到`/etc/mysql/my.cnf`,作如下修改

```shell
[mysqld]

character-set-server=utf8mb4

[mysql]

default-character-set=utf8mb4
```

![](https://i.loli.net/2019/05/21/5ce3b6a8168b938652.png)

​		修改后重启Mysql   ` sudo service mysql restart`

2. 将已经建好的表也转换成utf8mb4

在终端执行`sudo mysql -uroot -p`命令进入MySQL控制台
然后执行 命令：`alter table TABLE_NAME convert to character set utf8mb4 collate utf8mb4_bin;`

将TABLE_NAME替换成你的表名即可。

![](https://i.loli.net/2019/05/21/5ce3b49a3e2f723010.png)

可以看到插入数据成功！

![](https://i.loli.net/2019/05/21/5ce3baaca602961339.png)

# Reference

[如何在Ubuntu 18.04上安装MySQL](https://www.howtoing.com/how-to-install-mysql-on-ubuntu-18-04/)

[解决mysql插入数据时出现Incorrect string value: '\xF0\x9F...' for column 'name' at row 1的异常](https://blog.csdn.net/azhegps/article/details/71480633)
