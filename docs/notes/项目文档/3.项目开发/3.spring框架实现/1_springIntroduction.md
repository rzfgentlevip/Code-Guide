---
title: 1、实现简单的spring容器
icon: file
order: 1
author: bugcode
date: 2020-01-01T00:00:00.000Z
category:
  - SPRING
  - SPRINGBOOT
  - JAVA
tag:
  - 后端
  - java
  - spring
sticky: false
star: true
footer: Spring基础
copyright: bugcode
createTime: 2026/01/17 20:14:52
permalink: /project-docs/project/springframework/实现简单的spring容器/
---
<!-- TOC -->

- [1、实现简单的spring容器](#1实现简单的spring容器)
  - [1、目标](#1目标)
    - [1.1、引言](#11引言)
    - [1.2、目标](#12目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、类图结构](#31类图结构)
    - [3.2、BeanDefinition bean类型定义](#32beandefinition-bean类型定义)
    - [3.3、BeanFactory 工厂](#33beanfactory-工厂)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)

<!-- /TOC -->


# 1、实现简单的spring容器
## 1、目标
### 1.1、引言
spring bean容器的目标是什么？负责管理应用程序bean对象配置和生命周期，某种意义上容器其实就是一个负责管理并且维护bean声明周期的代理，代替应用程序本身管理的bean对象， 你可以配置你的每个 Bean 对象是如何被创建的，这些 Bean 可以创建一个单独的实例或者每次需要时都生成一个新的实例，以及它们是如何相互关联构建和使用的，如果应用需要使用bean对象，直接从容器中获取即可，这样应用程序可以更加关注业务本身，不需要考虑实现业务过程中需要的组件以及组件的管理，解耦了应用程序和应用程序使用的组件，这些组件就像一个一个的模块，在实现业务的过程中，需要哪些组件，就将组件模块插入spring即可，让spring容器负责管理各个组件。


### 1.2、目标
spring容器包含并且管理者bean对象的整个生命周期，从bean对象的**定义，创建，初始化，使用，销毁**，管理bean对象的每一个阶段，如果一个bean对象的定义被加载到容器中，那么后续这个bean就会被容器统一进行创建，初始化，然后将bean存放在容器中，程序中就可以直接从容器中获取并且使用这个bean对象了。

本章节中，我们就简单的实现一个容器，然后将bean对象加载到容器中，最后客户端从容器中获取bean对象使用。

## 2、设计
在程序设计中，所有可以存放数据的数据结构实现，都可以称作容器，比如常见的Map，List，Set等都可以叫做容器，因为这些集合都有一个共性，即可以用来存放各种数据。

思考一下容器应该使用什么样数据结构？

- 所选用的数据结构应该方便程序获取bean对象，因为程序在运行过程中会频繁使用bean对象，因此时间复杂度控制在O(1）。 
- 每一个bean对象应该有一个惟一的标识，方便程序获取单例的bean对象。
- 因此，选择key-val类型的数据结构作为容器，key中存放bean对象的唯一标识，val中存放bean对象。
麻雀虽小，五脏俱全，spring容器的核心过程就是spring根据配置，将应用程序所用到的所有bean对象都注入到spring容器中，然后管理bean的生命周期，所以在本章节中，首先实现一个简易的spring 容器，然后将bean对象注入容器中，最后应用从容器中获取bean对象使用。

下图是一个简单容器的结构:

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/simpleSpringContainer.webp)

- BeanDefinition：bean对象的定义信息，即bean的class信息，但是在本章节中，首先以一种简易的方式将bean注入到BeanDefinition定义中，下个章节在实现BeanDefinition定义。
- BeanRegister：Bean注册类，其实就是服务将bean对象的类型信息注入到容器中。
- 获取：最后就是获取对象，Bean 的名字就是key，Spring 容器初始化好 Bean 以后，就可以直接获取了。

## 3、实现
### 3.1、类图结构

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/simpleContainerClassPic.png)

- BeanDefinition:用于定义Bean的定义信息，BeanDefinition#bean对象就是用来模拟存储bean定义。
- BeanFactory：代表了创建Bean对象的工厂，BeanFactory#registerBeanDefinition()负责注册bean的定义，BeanFactory#getBean()负责对外提供获取bean对象的途径，在应用程序中对bean对象的获取一般很频繁，所以会使用工厂模式创建bean对象。

### 3.2、BeanDefinition bean类型定义

```java
public class BeanDefinition {

//    存放bean对象
    private Object bean;

    public BeanDefinition(Object bean) {
        this.bean = bean;
    }

    public Object getBean() {
        return bean;
    }

}
```
BeanDefinition#bean，用来模拟存储bean对象的定义信息。在后续会逐步完善BeanDefinition的定义， 例如：SCOPE_SINGLETON、SCOPE_PROTOTYPE、ROLE_APPLICATION、ROLE_SUPPORT、ROLE_INFRASTRUCTURE 以及 Bean Class 信息。


### 3.3、BeanFactory 工厂
```java
public class BeanFactory {

//    存放bean的名字和定义的映射
    private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();

    public Object getBean(String name) {
        return beanDefinitionMap.get(name).getBean();
    }

    public void registerBeanDefinition(String name, BeanDefinition beanDefinition) {
        beanDefinitionMap.put(name, beanDefinition);
    }

}
```
- BeanFactory#beanDefinitionMap：存放bean的定义，因为我们要在o(1)复杂度内获取到bean对象，所以使用map结构存储定义。
- BeanFactory第一个作用就是作为一个简单工厂，对外提供获取容器中bean对象的方法，第二个职责就是负责注册bean对象的定义信息。
- 在spring源码中，BeanFactory是访问spring容器的最顶层接口；The root interface for accessing a Spring bean container.



## 4、测试
### 4.1、测试用例
**Service定义**
```java
public class PeopleService {
    public void queryUserInfo(){
        System.out.println("查询用户信息");
    }
}
```
定义PeopleService bean对象，然后将bean对象注入到容器中。

### 4.2、测试结果
```java
  @Test
    public void testBeanDefine(){
       // 1.初始化 BeanFactory
        BeanFactory beanFactory = new BeanFactory();

        // 2.注入bean
        BeanDefinition beanDefinition = new BeanDefinition(new PeopleService());
        beanFactory.registerBeanDefinition("userService", beanDefinition);

        // 3.获取bean
        PeopleService userService = (PeopleService) beanFactory.getBean("userService");
        userService.queryUserInfo();
    }
```

**测试结果**
```java
查询用户信息

Process finished with exit code 0

```
从单测的程序来看，要从我们实现简单容器中获取bean对象，至少会通过一下三个步骤：
1. 初始化BeanFactory，实例化一个工厂，通过工厂从容器中获取Bean对象；
2. 注册BeanDefinition，将Bean的定义注册到容器中，当在获取对象的时候，如果容器中没有对象，就根据bean定义创建一个容器，如果有Bean对象，直接获取对象。另外可以发现，在注册bean对象的时候，直接将bean对象实例化后放到容器中，所以目前为了简易实现容器功能，后续对象的创建会根据bean的类型创还能。
3. 从容器中获取对象并且使用对象，获取对象使用工厂，工厂里面提供了getBean()方法用于从容器中获取bean对象，获取到的bean是Object类型，可以通过强转转换为目标类型。

## 5、总结

**小结**
1. 容器使用数据结构选型？
2. 如果一个对象会被大量创建并且使用，就考虑使用简单工厂设计模式，让工厂作为一个接口对外部提供获取对象的途径。

**目标**
1. 学会使用简单工厂设计模式；