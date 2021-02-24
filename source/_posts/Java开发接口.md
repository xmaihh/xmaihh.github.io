---
title: Java开发接口
date: 2018-07-18 16:43:01
categories: 后端技术
tags: 
toc: true
description: All happy families resemble one another.Each unhappy family is unhappy in its own way. 
---

### 开发环境

- JDK： v 10
- Tomcat ：v 9.0.6
- IntelliJ IDEA ：v 2017.3
- MySQL：v 5.7.19
### 学习计划

１. 环境搭建
２. HelloWorld
３. 创建数据库
４. Servlet写接口
５. Spring MVC写接口
６. Spring+SpringMVC+MyBatis
７. 云服务器部署

#### １.环境搭建 
要求:
１. 运行Tomcat成功后那只猫
２.IntelliJ IDEA 运行起来后的欢迎页
３．连接MySql后的截图，连接MySql命令行: `mysql -u root -p`
退出MySQL命令行:`exit`　

#### ２.HelloWorld 
要求:
使用InterlliJ IDEA创建Maven项目，运行起来是`Hello World`
注意事项:
注意下，比如选择Enable Auto import(是为了在pom.xml文件中添加依赖之后自动引入jar)

#### 3.创建数据库 
《Java 开发接口》是单表操作，按照这个教程创建数据库、表、填充数据，应该没什么问题

要求:1、拍照上传《Java 开发接口》教程里的 developer 表，并插入 5 条数据； 2、拓展作业 2.1、分页 查询 developer 表，需要查出第 2 - 4 条内容。 2.2、外键 查询 comment 表，要求查出第一条数据且查出 user 表里的昵称

#### 4.Servlet 写接口 
前面都是准备工作，接下来进入编码阶段，
《Java 开发接口》关于 Servlet，有两节，分别是「创建 Servlet」、「Servlet 写接口」，
我实践下来，并不是很难，可能代码量有些多，我也提供了相应的 sample，建议自己跟着手敲一遍，加深印象。 

注意事项: 之前代码并没有做异常处理，最好 try catch 一下，这里偷懒了一下

要求:用 Postman 测试，分别拍照 获取数据、添加数据、修改数据

#### 5.Spring MVC 写接口 
Spring MVC 写接口分别对应《Java 开发接口》教程里「创建 Spring MVC」、「Spring MVC 写接口」，
难点在于配置，完成了「创建 Spring MVC」，后面会很快搞定。 

注意事项: 同样教程里代码并没有做异常处理，最好 try catch 一下。

要求: 用 Postman 测试，分别拍照 数据的增删改查

#### 6.SSM 
SSM 是《Java 开发接口》的重头戏，很多公司项目都是用的 SSM 的框架，可见它的重要性，
SSM 配置相当复杂，我一开始也是一头雾水，仔细对照教程，一步步走下去，摸索摸索，会成功的。 

注意事项: 同样教程里代码并没有做异常处理，最好 try catch 一下。

要求: 用 Postman 测试，也是分别拍照 数据的增删改查

#### 7.云服务器部署 

[Java开发接口](/asset/Java开发接口.pdf) 下载链接: https://pan.baidu.com/s/18RJ6N0UQdwAK0P1o87f81g 密码: [ahyc](https://pan.baidu.com/s/18RJ6N0UQdwAK0P1o87f81g)
