title: 404
layout: page
comments: false

---

# 常用模式与工具

学习Java技术体系,设计模式,流行的框架与组件

- 常见的设计模式,编码必备
- Spring5,做应用必不可少的最新框架
- MyBatis,玩数据库必不可少的组件

## 常用设计模式

- Proxy代理模式
- Factory工厂模式
- Singleton单例模式
- Delegate委派模式
- Strategy策略模式
- Prototype原型模式
- Template模板模式

## Spring5

- IOC容器设计原理及高级特性
- AOP设计原理
- FactoryBean与BeanFactory
- Spring事务处理机制
- 基于SpringJDBC手写ORM框架
- SpringMVC九大组件
- 手写实现SpringMVC框架
- SpringMVC与Struts2对比分析
- Spring5新特性

## MyBatis

- 代码自动生成器
- MyBatis关联查询、嵌套查询
- 缓存使用场景及选择策略
- Spring集成下的SqlSession与Mapper
- MyBatis的事务
- 分析MyBatis的动态代理的真正实现
- 手写实现Mini版的MyBatis

# 工程化与工具

工欲善其事必先利其器，不管是小白，还是资深开发，玩Java技术体系，选择好的工具，提升开发效率和团队协作效率，是必不可少的
- Maven,项目管理
- Jenkins,持续集成
- Sonar,代码质量管理
- Git,版本管理

## Maven

- 生成可执行jar,理解Scope生成最精确的jar
- 类冲突、包依赖NoClassDefFoundError问题定位及解决
- 全面理解Maven的Lifecycle、Phase、Goal
- 架构师必备之Maven生成Archetype
- Maven流行插件实战、手写自己的插件
- Nexus的使用、上传、配置
- 对比Gradle

## Jenkins

- 搭建Jenkins自动部署环境
- Jenkins集成maven、git实现自动部署
- test\pre\production多环境发布
- Jenkins多环境配置、权限管理及插件使用

## Sonar

- 使用Sonar进行代码质量管理
- 关于代码检查工具FindBugs/PMD的运用
- SonarQube代码质量管理平台安装及使用
- 使用Jenkins与Sonar集成对代码进行持续检测
- Idea与Sonar集合的使用

## Git

- 什么是Git以及Git的工作原理
- Git常用命令Best Practise(避坑教学)
- Git冲突怎么引起的,如何解决
- 架构师职责:Git Flow规范团队Git使用教程
- 团队案例分享

# 分布式架构

高并发，高可用，海量数据，没有分布式的架构知识肯定是玩不转的

- 分布式架构原理
- 分布式架构策略
- 分布式中间件
- 分布式架构实战

##　分布式架构原理

- 分布式架构演进过程
- 如何把应用从单机扩展到分布式
- CDN加速静态文件访问
- 系统监控、容灾、存储动态扩容
- 架构设计及业务驱动划分
- CAP、Base理论以及其应用

## 分布式架构策略

- 分布式架构网络通信原理剖析
- 通信协议中的序列化和反序列化
- 基于框架的RPC技术 Webservice/RMI/ Hessian
- 深入分析 Zookeeper在 disconf配置中心的应用
- 基于 Zookeeper实现分布式服务器动态上下线感知
- 深入分析 Zookeeper Zab协议及选举机制源码解读
- Dubbo管理中心及监控平台安装部署
- 基于 Dubbo的分布式系统架构实战
- Dubbo容错机制及高扩展性分析

## 分布式架构中间件

- 分布式消息通信 ActiveMQ/ Kafka/ RabbitMQ
- Redis主从复制原理及无磁盘复制分析
- 图解 Redis中AOF和RDB持久化策略的原理
- MongoDB企业级集群解决方案
- MongoDB数据分片、转存及恢复策略
- 基于 OpenRest部署应用层Nginx以及Nginx+lua实践
- Nginx反向代理服务器及负载均衡服务配置实战
- 基于Netty实现高性能IM聊天
- 基于Netty实现Dubbo多协议通信支持
- Netty无锁化串行设计及高并发处理机制

## 分布式架构实战

- 分布式全局ID生成方案
- Session跨域共享及企业级单点登录解决方案实战
- 分布式事务解决方案实战
- 高并发下的服务降级、限流实战
- 基于分布式架构下分布式锁的解决方案实战
- 分布式架构下实现分布式定时调度

# 微服务架构 

业务越来越复杂，服务分层，微服务架构是架构升级的必由之路，Java技术体系，和微服务相关的技术有哪些呢？
- 微服务框架
- Spring Cloud
- Docker与虚拟化
- 微服务架构
## 微服务框架

- 与微服务之间的关系
- 热部署实战
- 核心组件Starter、Actuator、AutoConfiguration、Cli
- 集成Mybatis实现多数据源路由实战
- 集成Dubbo实战
- 集成Redis缓存实战
- 集成Swagger2构建API管理及测试体系
- 实现多环境配置动态解析

## Spring Cloud

- Eurrka注册中心
- Ribbon集成REST实现负载均衡
- OpenFeign声明式服务调用
- Hystrix服务熔断降级方式
- Zuul实现微服务网关
- Config分布式统一配置中心
- Sleuth调用链路跟踪
- BUS消息总线
- 基于Hystrix实现接口降级实战
- 集成Spring Cloud实现统一整合方案

## Docker虚拟化

- Docker的镜像、仓库、容器
- Docker File构建LNMP环保局部署个人博客Wordpress
- Docker Compose构建LNMP环境部署个人博客Wordpress
- Docker网络组成、路由互联、Openvswitch
- 基于Swarn构建Docker集群实战
- Kubernetes简介

## 漫谈微服务架构

- SOA架构和微服务架构之间的区别和联系
- 如何设计微服务及其设计原则
- 解惑Spring Boot流行因素及能够解决什么问题
- 什么是Spring Cloud,为何要选择Spring Cloud
- 基于全局分析Spring Cloud各个组件所解决的问题

# 性能优化

任何脱离细节的ppt架构师都是耍流氓，向上能运筹帷幄，向下能解决一线性能问题，Java技术体系，需要了解：

- 性能指标体系
- JVM调优
- Web调优
- DB调优

## 理解性能优化

- 性能基准
- 性能优化到底是什么？
- 衡量维度

## JVM调优

- 详解什么是JVM运行时数据区
- 详解什么是JVM内存模型JMM
- 详解GC可达
- 详解各垃圾回收器使用场景(THroughput\CMS)
- 详解GC日志,从日志找问题
- 实战MAT分析dump文件(推理、验证)

## Tomcat调优

- How it works?探查Tomcat的运行机制及框架
- 分析Tomcat线程模型
- Tomcat系统参数认识及调优
- 基准测试

## MySQL调优

- 理解MySQL底层B+Tree机制
- SQL执行计划详解
- 索引优化详解
- SQL语句优化