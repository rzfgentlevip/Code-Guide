---
title: 3、实现构造函数实例化策略
icon: file
order: 3
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
permalink: /project-docs/project/springframework/实现构造函数实例化策略/
---


<!-- TOC -->

- [3、实现构造函数实例化策略](#3实现构造函数实例化策略)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类关系](#31核心类关系)
    - [3.2、新增getBean()方法](#32新增getbean方法)
    - [3.3、实例化策略接口](#33实例化策略接口)
    - [3.4、JDK实例化](#34jdk实例化)
    - [3.5、Cglib实例化](#35cglib实例化)
    - [3.6、创建策略调用](#36创建策略调用)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)

<!-- /TOC -->

# 3、实现构造函数实例化策略

## 1、目标

第二章中初步的将类型定义封装到BeanDefinition中，然后让spring负责注册bean类型信息和创建bean对象，但是在创建bean对象时，容器直接获取到bean定义然后使用newInstance()方法返回创建的对象，但是在实际情况下，bean对象还会存在很多属性信息，所以上一章节在创建bean对象的过程中没有考虑当bean对象存在属性这种情况，本章节在此基础上，将创建有参数的bean对象。

## 2、设计

思考在实例化参数时将属性信息以参数的形式传输给实例化方法，需要考虑两个问题：

1. 在获取bean对象获取不到的时候，spring容器会先去获取bean的定义，然后根据bean的定义去创建bean实例对象，所以首先要考虑在何时将参数信息传递给构建bean对象的方法。
2. 当构建bean对象的方法获取到传输的参数后，如何根据参数去构建bean对象。

参考 Spring Bean 容器源码的实现方式，在 BeanFactory 中添加 Object getBean(String name, Object... args) 接口，这样就可以在获取 Bean 时把构造函数的入参信息传递进去了。

另外一个核心的内容是使用什么方式来创建含有构造函数的 Bean 对象呢？这里有两种方式可以选择，一个是基于 Java 本身自带的方法 DeclaredConstructor，另外一个是使用 Cglib 来动态创建 Bean 对象。*Cglib 是基于字节码框架 ASM 实现，所以你也可以直接通过 ASM 操作指令码来创建对象*

所以整体实现分为两部分：

1. 何时将参数传递给实例化方法
   
2. 实例化方法获取到参数后如何根据参数创建对象

<img src="https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408020938567.png" alt="img" style="zoom: 67%;" />

- 参考 Spring Bean 容器源码的实现方式，在 BeanFactory 中添加 Object getBean(String name, Object... args) 接口，这样就可以在获取 Bean 时把构造函数的入参信息传递进去了。
- 另外一个核心的内容是使用什么方式来创建含有构造函数的 Bean 对象呢？这里有两种方式可以选择，一个是基于 Java 本身自带的方法 DeclaredConstructor，另外一个是使用 Cglib 来动态创建 Bean 对象。*Cglib 是基于字节码框架 ASM 实现，所以你也可以直接通过 ASM 操作指令码来创建对象*

## 3、实现

### 3.1、核心类关系

<img src="https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408020940328.png" alt="img" style="zoom:80%;" />

在获取bean对象的时候，传输初始化参数，所以给BeanFactory#getBean方法添加参数信息。

另外因为实例化有参数的对象有两种方法，jdk和cglib，因此定义一个策略接口**InstantiationStrategy**，然后由两种不同的实例化方法实现此接口实现有参数对象的创建工作。

AbstractAutowireCapableBeanFactory此类中持有实例化策略接口对象，调用不同的方法进行实例化。

> 策略设计模式

### 3.2、新增getBean()方法

在获取bean对象的时候，如果获取不到spring容器会创建bean对象，所以传输属性参数的入口就是getBean()方法，所以首先在BeanFactory接口中新增带参数的getBean()方法;

```java
/**
 * 接口功能
 *  1.提供actor从容器中获取bean对象的功能
 */
public interface BeanFactory {

    /**
     * 获取bean对象，没有参数
     * @param name
     * @return
     * @throws BeansException
     */
    Object getBean(String name) throws BeansException;

    /**
     * 获取bean对象，有参数
     * @param name
     * @param args
     * @return
     * @throws BeansException
     */
    Object getBean(String name, Object... args) throws BeansException;
}
```
这样在getBean()对象的时候，如果bean对象为空，就可以将传入的参数传递给构造函数构建有属性的bean然后放到容器中。

### 3.3、实例化策略接口

```java
/**
 * 接口功能：
 *  1. 负责bean对象的实例化策略，有参数构建bean或者无参数构建bean
    2 实例化bean对象有多重方法，可以实现该接口，实现多重方法构建bean
 */
public interface InstantiationStrategy {

    Object instantiate(BeanDefinition beanDefinition, String beanName, Constructor ctor, Object[] args) throws BeansException;

}
```

实例化接口中的方法instantiate有一个args参数(对象的属性)，在对象实例化的时候使用参数实例化。

- 有两个对象实例化类分别实现接口中的方法完成对象实例化。

- 在实例化接口 instantiate 方法中添加必要的入参信息，包括：beanDefinition、 beanName、ctor、args

其中 Constructor 你可能会有一点陌生，它是 java.lang.reflect 包下的 Constructor 类，里面包含了一些必要的类信息，有这个参数的目的就是为了拿到符合入参信息相对应的构造函数。

### 3.4、JDK实例化

```java
/**
 * jdk实例化Bean对象
 */
public class SimpleInstantiationStrategy implements InstantiationStrategy {

    /**
     * 根据有参或者无参数实例化对象
     * @param beanDefinition
     * @param beanName
     * @param ctor
     * @param args
     * @return
     * @throws BeansException
     */
    @Override
    public Object instantiate(BeanDefinition beanDefinition, String beanName, Constructor ctor, Object[] args) throws BeansException {
        /*获取Bean对象的定义*/
        Class clazz = beanDefinition.getBeanClass();
        try {
            /*判断是否有构造函数*/
            if (null != ctor) {
//                有参构造
                return clazz.getDeclaredConstructor(ctor.getParameterTypes()).newInstance(args);
            } else {
//                无参构造
                return clazz.getDeclaredConstructor().newInstance();
            }
        } catch (NoSuchMethodException | InstantiationException | IllegalAccessException | InvocationTargetException e) {
            throw new BeansException("Failed to instantiate [" + clazz.getName() + "]", e);
        }
    }
}
```
JDK实例化首先获取bean的定义Class信息，class信息是spring注册bean定义的时候注册到容器中的，然后判断传进来的构造函数是否为空，如果为空，就执行有参构造函数的实例化，否则执行无参构造函数实例化。

这里我们重点关注有构造函数的实例化，实例化方式为 clazz.getDeclaredConstructor(ctor.getParameterTypes()).newInstance(args);，把入参信息传递给 newInstance 进行实例化。

### 3.5、Cglib实例化

```java
/**
 * Cglib方式创建Bean对象
 * @author yourname
 * @date 18:23 2024/11/23 
 * @param null 
 * @return null
 **/
public class CglibSubclassingInstantiationStrategy implements InstantiationStrategy {

    /**
     * 动态代理实例化有参数对象
     * @param beanDefinition
     * @param beanName
     * @param ctor
     * @param args
     * @return
     * @throws BeansException
     */
    @Override
    public Object instantiate(BeanDefinition beanDefinition, String beanName, Constructor ctor, Object[] args) throws BeansException {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(beanDefinition.getBeanClass());
        enhancer.setCallback(new NoOp() {
            @Override
            public int hashCode() {
                return super.hashCode();
            }
        });
        if (null == ctor) return enhancer.create();
        return enhancer.create(ctor.getParameterTypes(), args);
    }

}
```



### 3.6、创建策略调用

```java
/**
 * 继承抽象的工厂类，实现自己的方法，公共的方法由抽象类实现
 * 体现了类实现过程中的各司其职，你只需要关心属于你的内容，不是你的内容，不要参与
 */
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory {

    /* 默认适用cglib进行实例化*/
    private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

    /**
     * 实例化bean对象，有参数
     * @param beanName
     * @param beanDefinition
     * @param args
     * @return
     * @throws BeansException
     */
    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
        Object bean;
        try {
            /*调用创建Bean对象的方法*/
            bean = createBeanInstance(beanDefinition,beanName,args);
        } catch (Exception e) {
            throw new BeansException("Instantiation of bean failed", e);
        }

        addSingleton(beanName, bean);
        return bean;
    }

    /**
     * 具体的创建Bean对象的方法
     * @param beanDefinition
     * @param beanName
     * @param args
     * @return
     */
    protected Object createBeanInstance(BeanDefinition beanDefinition, String beanName, Object[] args) {
        Constructor constructorToUse = null;
        /*获取Bean对象的定义*/
        Class<?> beanClass = beanDefinition.getBeanClass();
        /*获取Bean的构造函数*/
        Constructor<?>[] declaredConstructors = beanClass.getDeclaredConstructors();
        for (Constructor ctor : declaredConstructors) {
            if (null != args && ctor.getParameterTypes().length == args.length) {
                constructorToUse = ctor;
                break;
            }
        }
        /*实例化Bean对象*/
        return getInstantiationStrategy().instantiate(beanDefinition, beanName, constructorToUse, args);
    }

    /**
     * 返回实例化策略
     * @author yourname
     * @date 18:25 2024/11/23  
     * @return bugcode.online.springframework.beans.factory.support.InstantiationStrategy
     **/
    public InstantiationStrategy getInstantiationStrategy() {
        return instantiationStrategy;
    }

    public void setInstantiationStrategy(InstantiationStrategy instantiationStrategy) {
        this.instantiationStrategy = instantiationStrategy;
    }
}
```
1. 策略调用首先在AbstractAutowireCapableBeanFactory抽象类中创建InstantiationStrategy对象，默认选择的是Cglib实现类。

2. 接下来抽取 createBeanInstance 方法，在这个方法中需要注意 Constructor 代表了你有多少个构造函数，通过 beanClass.getDeclaredConstructors() 方式可以获取到你所有的构造函数，是一个集合。
3. 接下来就需要循环比对出构造函数集合与入参信息 args 的匹配情况，这里我们对比的方式比较简单，只是一个数量对比，而实际 Spring 源码中还需要比对入参类型，否则相同数量不同入参类型的情况，就会抛异常了。

## 4、测试

### 4.1、测试用例

Service用例

```java
public class PeopleService {
    private String name;

    public PeopleService(String name) {
        this.name = name;
    }

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + name);
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("");
        sb.append("").append(name);
        return sb.toString();
    }
}
```

在Service中我们添加一个name属性，然后再创建bean对象的时候，就使用有参构造函数进行实例化。

### 4.2、测试结果

```java
    @Test
    public void testBeanDefine(){
        // 1.初始化 BeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2.注册 bean
        BeanDefinition beanDefinition = new BeanDefinition(PeopleService.class);
        beanFactory.registerBeanDefinition("peopleService", beanDefinition);

        // 3.第一次获取 bean
        PeopleService peopleService1 = (PeopleService) beanFactory.getBean("peopleService","bugcode");
        peopleService1.queryUserInfo();

    }
```

**测试结果**

```java
查询用户信息：bugcode

Process finished with exit code 0
```

测试中可以看到，到目前为止，我们在使用spring框架的时候，还是先实例化beanFactory工厂，然后手动注册BeanDefinition定义，最后从容器中获取bean对象，依然是三步骤。

## 5、总结

本章使用设计模式：

**模板模式**：BeanFactory容器的设计使用模板模式，接口定义功能，AbstractBeanFactory定义调用过程 实现公共的逻辑方法,[模板模式参考](https://codinglab.online/designpattern/Behavior/act_one_templatePattern.html);

**策略模式**：在程序中如果一个动作有多种方法完成，那么就可以使用策略模式，在顶级接口中定义这个动作，然后由不同的子类去完成实现具体的动作，最后使用接口做多态调用，[策略模式参考](https://codinglab.online/designpattern/Behavior/act_two_strategyPattern.html#_4-3%E3%80%81%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F%E9%AA%A8%E6%9E%B6);

设计原则：

1. 接口功能应该清洗明确，一个接口只负责一个功能，比如创建，注册等，一个接口中不要定义太多的职责。
2. 继承接口的抽象类，应该只实现子类的一些公共的方法，不属于公共的方法应该让子类自己去实现，另外抽象类中还可以定义抽象方法，抽象方法让具体的子类去实现。
3. 想让某一个类拥有某一个接口功能，直接实现某一个接口即可，类可以继承抽象类然后实现一个接口，那么这个类就继承了抽象类所有的方法，并且还拥有接口的方法。
4. 接口，抽象类，类之间的关系，多用设计模式，简单工厂，抽象工厂，模板，策略，builder等。

**到目前为止，我们实现的spring功能：**

实现一个容器：定义map结构

定义和注册bean:定义bean类型以及将类型放到map中

实例化bean：创建bean对象并且放到容器中

实例化策略：按照不同实例化策略（有参和无参）实例化对象，使用策略模式