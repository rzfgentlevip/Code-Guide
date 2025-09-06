---
title: SpringBoot基础入门一
icon: streamline-freehand-color:notes-quill
order: 1
author: bugcode
date: 2020-01-01T00:00:00.000Z
category:
  - SPRINGBOOT
  - SPRING
  - JAVA
tag:
  - 后端
  - java
  - spring
sticky: false
star: true
footer: Spring基础
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/49jkpnoi/
---

#  Spring生态圈

## 1.1、Spring的能力

官网地址

https://spring.io/

```text
微服务
一个项目进行拆分成单个小服务。
springboot基于spring framework，简化了spring framework的复杂配置。
可以整合spring生态中的其它技术栈，免于写复杂的配置。

响应式编程
基于异步非阻塞的方式，通过应用之间构建异步数据流的方式，异步数据流允许我们只用少量的线程资源，
构建高吞吐量的应用。

分布式云
项目拆分成微服务后，变成分布式应用。spring cloud用于解决这种情况。

WEB项目开发
例如springmvc开发web项目。

无服务开发，
FaaS，Function as a Service，"功能即服务"（也译作“函数即服务”）无需购买服务器，直接将服务
上传到云平台，动态分配资源，

事件驱动
spring可将整个分布式系统构建出，一个基于事件方式的实时数据流。通过该数据流，就能通过响应式的方式
，占用少量的资源实现高吞吐量的业务需求。

批处理。
```

## 1.2、spring的能力

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211031623.png)



## 1.3、Spring生态

spring宏观上，指的是整个spring生态圈。微观上，指的是spring framework 框架。

覆盖了：

- web开发
- 数据访问
- 安全控制
- 分布式
- 消息服务
- 移动开发
- 批处理

## 1.4、Spring5重大升级

spring5指的是整个5版本的spring生态。

因为spring framework5的重大升级（异步响应式编程），所以导致了springboot 1 和 2 版本的使用方式发生了变化。

### 1.4.1、响应式编程

springboot2，对比传统的servlet技术栈，新增了**响应式编程（Reactive stack）**的技术栈。从底层数据访问层、WEB视图层交互、安全....各个方面推出了一系列的响应式解决方案(异步响应)。使用少量的资源就能实现高并发高吞吐量的业务。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211030169.png)

- servlet是spring最原生的技术栈，即spring+springmvc
- Reactive stack:响应式技术栈，从数据访问到web开发，安全开发，都使用的是异步响应式开发，少量的线程，占少量的资源，就可以处理大量的并发任务。

### 1.4.2、**内部源码设计**

1. spring5源码是基于Java8实现。
2. java8中有一些新特性，例如：接口的默认实现。假设接口A中有5个方法，实现类B只想用其中的1个方法，却要将其余4个方法全部实现为空方法。
3. Java8之前，spring底层会使用适配器模式：通过一个适配器类将A的方法全部实现为空方法，B继承适配器后，只用重写其中的指定方法即可。Java8出来后，Java支持**接口的默认实现**，适配器失去了使用价值。所以，因为Java8的新特性，Spring5内部源码也有了大量变化。

## 1.5、为什么使用SpringBoot

1. 能快速创建出生产级别的Spring应用，简化配置。
2. SpringBoot本身是作为整合其它Spring生态技术栈的一个框架。
3. SpringBoot是整合Spring技术栈的一站式框架
4. SpringBoot是简化Spring技术栈的快速开发脚手架

## 1.6、SpringBoot的优点

- Create stand-alone Spring applications
    - 创建独立Spring应用，原生的spring需要写很多配置应用。
- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
    - 内嵌web服务器（服务器不用部署Tomcat/Jetty ，打包完成直接运行），springmvc程序需要打成war包，然后部署到tomcat服务器上，目标环境需要安装配置tomcat服务器。
- Provide opinionated 'starter' dependencies to simplify your build configuration
    - 自动starter依赖，简化构建配置（只需要导入对应模块的starter启动器依赖，该模块的所有对应版本的依赖自动导入），starter叫做启动器。
- Automatically configure Spring and 3rd party libraries whenever possible
    - 自动配置Spring以及第三方功能（自动设置好框架的默认配置，自定义少量配置即可）
- Provide production-ready features such as metrics, health checks, and externalized configuration
    - 提供生产级别的监控、健康检查及外部化配置
- Absolutely no code generation and no requirement for XML configuration
    - 无额外代码生成、无需编写XML（使用时不会生成额外代码，也不用像spring framework那样做大量配置）

## 1.7、SpringBoot缺点

- 人称版本帝，迭代快，需要时刻关注变化
- 封装太深，内部原理复杂，不容易精通

## 1.8、微服务

- 微服务是一种架构风格
- 一个应用拆分为一组小型服务
- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

## 1.9、分布式的困难

> 由于微服务的产生，会导致软件向分布式发展。

- 远程调用，服务和服务之间相互调用通信。
- 服务发现 （A业务节点调B业务集群，需要知道B业务集群中哪些节点是存活可用）
- 负载均衡，前端的请求需要均衡的发送到各个微服务模块。
- 服务容错 （A业务节点调B业务集群，如果目标节点网络不通，是否转移请求至集群中另外一台机器？如果集群节点都不通该返回什么错误提示？等等...）
- 配置管理 （A业务集群节点的配置发生改变，不可能每个节点单独改，需要在一个地方做统一配置管理。）
- 服务监控：监控分布式集群上得到各个服务。
- 链路追踪 （A->B->C->D，如果整个调用链路失败，需要知道是哪个环节出现了什么问题。）
- 日志管理
- 任务调度

## 1.10、分布式解决

SpringBoot + SpringCloud

- springboot，负责快速构建应用，
- springcloud，管理网状的众多的微服务模块，
- springcloud dataflow ，通过响应式的方式，管理网状微服务之间的数据流。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211033610.png)

## 1.11、云原生

Cloud Native概念：开发好的原生应用，如何部署到云。

## 1.12、上云面临的困难

比较偏向于运维层面。

- 服务自愈 （目标节点挂掉，能否拉起一台新的节点作为替代）
- 弹性伸缩 （使用高峰，集群资源不够时，自动按需扩充节点；高峰过去后，自动下线扩充节点）
- 服务隔离 （物理机上的部分服务出现故障时，不会影响同台物理机上的其它服务运行）
- 自动化部署
- 灰度发布 （假设A业务有新版本要发布，如果全部替换，可能出现故障。可以只在A业务集群的部分节点试点发布，新老版代码并存，经过验证无问题后，再逐步将剩余节点进行新版本发布。）
- 流量治理 （根据节点的性能高低，控制节点的流量接收量；流量进出的监控；根据集群整体流量动态扩松容。）
- ......

## 1.13、查看版本更新点说明

查看版本新特性；

https://github.com/spring-projects/spring-boot/wiki#release-notes

