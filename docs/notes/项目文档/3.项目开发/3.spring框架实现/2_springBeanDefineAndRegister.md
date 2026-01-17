---
title: 2、SpringBean定义
icon: file
order: 2
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
permalink: /project-docs/project/springframework/SpringBean定义/
---

<!-- TOC -->

- [2、SpringBean定义](#2springbean定义)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类图](#31核心类图)
    - [3.2、抽象类定义模板方法AbstractBeanFactory](#32抽象类定义模板方法abstractbeanfactory)
    - [3.3、创建Bean对象的抽象类AbstractAutowireCapableBeanFactory](#33创建bean对象的抽象类abstractautowirecapablebeanfactory)
    - [3.3、核心实现类DefaultListableBeanFactory](#33核心实现类defaultlistablebeanfactory)
    - [3.4、单例接口：SingletonBeanRegistry](#34单例接口singletonbeanregistry)
    - [3.5、单例接口实现类：DefaultSingletonBeanRegistry](#35单例接口实现类defaultsingletonbeanregistry)
    - [3.6、定义BeanDefinition](#36定义beandefinition)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)

<!-- /TOC -->

# 2、SpringBean定义


## 1、目标

在第一章节，已经实现类一个简易的spring容器模型，容器模型提供了bean对象的注册和从容器中获取bean的功能，但是实现中还存在两点缺陷：

1. bean的定义信息，只是用一个object代替了bean的定义，实际使用的时候，还需要new一个对象传入bean的定义中，所以bean的创建操作并没有交给容器，因此本章我们目标将bean对象的创建移交给容器。
2. 多次获取同一个bean对象，应该从缓存中获取，不应该在重新创建一个对象。


## 2、设计

第一章节中，类型的注册，其实是一个Object对象，是直接将创建好的对象注册到Map中，需要的时候直接获取已经创建好的bean，本章需要将对象的创建移交给容器，因此就需要修改BeanDefinition定义;**在注册阶段，只注册bean的类型信息，当获取bean没有的时候，spring容器根据bean class信息创建对象并且放到容器中**，因此首先要做的就是修改BeanDefinition里面的Object为Class类型信息。

第二个要做的就是单例Bean，如果应用已经创建了一个Bean对象在容器中，另外一个程序在使用bean对象的时候直接从容器中获取即可，不需要再重新创建bean对象，因此我们要创建一个单例接口，只负责单例对象的获取。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/act_two_design.webp)


首先看到容器中多了两个组件，**Class容器和单例组件**，Class组件存储在注册bean定义阶段bean的class信息，单例组件，提供单例容器，应用程序在获取bean的时候，容器中存在bean对象直接获取使用即可。

BeanFactory工厂，定义顶级的工厂接口，提供从容器中获取对象的方法getBean(String name),之后由AbstractBeanFactory抽象类实现工厂，实现具体的获取bean对象的方法，使用**模板模式**，可以将核心的逻辑统一在抽象类中实现，方便后续扩展。此外抽象类继承了DefaultListableBeanFactory单例抽象类，因此在AbstractBeanFactory中就默认拥有了获取单例对象的功能了，因此通过继承的方法，扩展了类的功能。

AbstractAutowireCapableBeanFactory抽象类继承AbstractBeanFactory抽象类，在实现父抽象类中定义的方法，这样每个类分工明确，父类一般定义调用关系，子类一般定义方法实现，各司其职。

**抽象类有两种主要用法：**

1. 定义调用过程:在抽象类中定义主要的调用流程，子类继承抽象类做实现；
2. 实现功能扩展，哪一个类想拥有抽象类功能，直接继承此类即可。

顶级单例接口SingletonBeanRegistry定义获取单例bean对象的方法，默认的实现类DefaultSingletonBeanRegistry定义单例容器，然后实现向单例容器中增加bean对象和获取bean对象的方法。作为一个抽象单例接口的默认实现，以后哪一个类需要有获取单例对象的功能，直接继承这个抽象类即可。

## 3、实现

### 3.1、核心类图

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/act_two_class.webp)


抽象类AbstractBeanFactory主要定义了获取bean对象的调用过程，然后继承DefaultSingletonBeanRegistry单例抽象类，因此就具备了向单例容器中添加和获取bean对象的能力。

抽象类 AbstractAutowireCapableBeanFactory  继承自AbstractBeanFactory，他首先会实现属于自己的方法，主要是其子类的一些公共的方法，非公共的方法由继承自AbstractAutowireCapableBeanFactory 的子类进行实现，这样做的好处是每一个类只需要关注并且实现属于自己的方法即可，公共的方法由其父类统一实现封装，这里就体现了在实现类过程中每个类只需要关注自己的内容，做到高内聚，类的设计最小化。

DefaultSingletonBeanRegistry  类实现了单例接口 SingletonBeanRegistry ，又会被抽象类 AbstractBeanFactory  继承，因此抽象类 AbstractBeanFactory  不但拥有工厂接口的方法，还拥有单例接口的方法。

BeanDefinitionRegistry接口：定义具体的注册bean信息的顶级接口，实现此接口的类拥有注册bean定义的能力。DefaultListableBeanFactory具体类继承AbstractAutowireCapableBeanFactory，因此具备从容器中获取bean对象的能力和向单例容器中增加和获取单例bean对象的能力，然后实现BeanDefinitionRegistry接口，因此又具备了向容器中注册bean类型定义的功能，所以说是功能很强大的一个类实现。

从本章节开始，可以看到在实现中针对不同的功能，分别定义不同的接口，然后由子类去实现或者继承

- BeanFactory：顶级工厂，定义从容器中获取bean的BeanFactory#getBean(String name)方法

- SingletonBeanRegistry：单例bean接口，定义从从容器中获取单例bean对象的SingletonBeanRegistry#getSingleton(String beanName)方法。

- BeanDefinitionRegistry：beanDefinition定义注册接口，定义向容器中注册bean定义的方法BeanDefinitionRegistry#registerBeanDefinition()，实现向容器中注册类信息。

从以上定义的接口可以看到，每个接口定义的功能很原子性，功能明确，提供很少的方法，然后让抽象类和子具体类通过继承，实现的方法在接口的功能上做扩展操作，所以说这种设计很值得我们学习。


### 3.2、抽象类定义模板方法AbstractBeanFactory

```java
/**
 * BeanDefinition Bean对象定义信息
 * AbstractBeanFactory类实现BeanFactory接口然后继承DefaultSingletonBeanRegistry抽象类，
 * 所以具备单例Bean的注册功能
 */
public abstract class AbstractBeanFactory extends DefaultSingletonBeanRegistry implements BeanFactory {

    /**
     * 实现BeanFactory工厂接口中的方法 使用模板模式,
     * @param name
     * @return
     * @throws BeansException
     */
    @Override
    public Object getBean(String name) throws BeansException {
        /*首先从容器中获取单例对象*/
        Object bean = getSingleton(name);
        if (bean != null) {
            return bean;
        }
        /*如果容器中没有此对象，就获取bean的定义然后创建对象*/
        BeanDefinition beanDefinition = getBeanDefinition(name);
        return createBean(name, beanDefinition);
    }

    /**
     * 定义抽象类受保护的方法，获取bean的定义信息
     * @author yourname
     * @date 17:40 2024/11/23
     * @param beanName
     * @return bugcode.online.springframework.beans.factory.config.BeanDefinition
     **/
    protected abstract BeanDefinition getBeanDefinition(String beanName) throws BeansException;

    /**
     * 定义受保护方法 ，创建Bean对象
     * @author yourname
     * @date 17:40 2024/11/23
     * @param beanName
     * @param beanDefinition
     * @return java.lang.Object
     **/
    protected abstract Object createBean(String beanName, BeanDefinition beanDefinition) throws BeansException;

}
```

AbstractBeanFactory实现了BeanFactory工厂接口，重写getBean方法：
1. 首先去容器中获取bean对象，如果获取不到bean对象，就去获取bean的定义信息
2. 根据Bean的定义信息创建新的bean对象，获取bean定义和根据bean定义创建对象被声明为两个受保护的方法，由继承其子类的方法自己实现。可以看到，在这个类中只是定义了创建bean对象的方法，类本身并没有做实现，而是由具体的子类去实现。

AbstractBeanFactory接口还继承了默认的单例实现类DefaultSingletonBeanRegistry，因此也就具备了单例注册类方法，其实在获取对象的时候就是从单例容器中获取对象；

另外这里可以体会到，在接口中定义一个功能，然后在抽象类中进行功能扩充，并且抽象类中定义流程，比如getBean()方法，在getBean()方法中定义了获取Bean对象的步骤，以及获取不到Bean对象步骤之后的动作，并且重新定义了受保护的抽象方法；

### 3.3、创建Bean对象的抽象类AbstractAutowireCapableBeanFactory

```java
/**
 * 继承抽象的工厂类，实现自己的方法，公共的方法由抽象类实现
 * 体现了类实现过程中的各司其职，你只需要关心属于你的内容，不是你的内容，不要参与
 */
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory {

    /**
     * 实现父类中创建Bean对象的方法
     * @author yourname
     * @date 17:46 2024/11/23 
     * @param beanName
     * @param beanDefinition 
     * @return java.lang.Object
     **/
    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition) throws BeansException {
        Object bean;
        try {
            /*创建Bean对象*/
            bean = beanDefinition.getBeanClass().newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new BeansException("Instantiation of bean failed", e);
        }
        /*将创建好的Bean对象添加到容器中，然后返回Bean对象*/
        addSingleton(beanName, bean);
        return bean;
    }
}
```

核心类，在AbstractAutowireCapableBeanFactory抽象类中实现了对象的实例化操作，通过bean对象的定义信息newInstance一个新的对象然后放到容器中。

创建好bean对象后，放到单例对象的缓存中。为什么此类可以从那个单例容器中存放bean对象，因为其实现了AbstractBeanFactory抽象类，而父抽象类继承单例接口的默认实现，因此具备从单例容器中获取和添加bean对象的功能。

此外，从这里也可以看到，对象的创建权力逐步移交给了容器，不再是我们控制对象的创建行为然后放到容器中；

### 3.3、核心实现类DefaultListableBeanFactory

```java
/**
 * BeanDefinitionRegistry：注册Bean定义功能
 * AbstractAutowireCapableBeanFactory--》BeanFactory  获取Bean功能
 * @author yourname
 * @date 17:53 2024/11/23 
 * @param null 
 * @return null
 **/

public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory implements BeanDefinitionRegistry {

    /*存放Bean对象的定义*/
    private final Map<String, BeanDefinition> beanDefinitionMap = new HashMap<>();

    /**
     * 向容器中注册Bean对象的定义信息
     * @author yourname
     * @date 17:50 2024/11/23
     * @param beanName
     * @param beanDefinition
     **/
    @Override
    public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) {
        beanDefinitionMap.put(beanName, beanDefinition);
    }

    /**
     * 获取Bean对象的定义信息
     * @author yourname
     * @date 17:51 2024/11/23
     * @param beanName
     * @return bugcode.online.springframework.beans.factory.config.BeanDefinition
     **/
    @Override
    public BeanDefinition getBeanDefinition(String beanName) throws BeansException {
        BeanDefinition beanDefinition = beanDefinitionMap.get(beanName);
        if (beanDefinition == null) throw new BeansException("No bean named '" + beanName + "' is defined");
        return beanDefinition;
    }
}
```

- 核心实现类DefaultListableBeanFactory实现了BeanDefinitionRegistry接口并且继承AbstractAutowireCapableBeanFactory抽象类；

- 通过继承AbstractAutowireCapableBeanFactory类实现getBeanDefinition获取bean定义的方法。
- 通过实现BeanDefinitionRegistry接口实现注册bean定义功能。
- 因此在DefaultListableBeanFactory一个类里面就实现了**注册Bean定义和获取Bean定义**的功能。


### 3.4、单例接口：SingletonBeanRegistry

```java
/**
 *
 * 单例注册接口，负责注册单例对象
 */
public interface SingletonBeanRegistry {

    /**
     * 接口中定义获取单例对象的方法
     * @author yourname
     * @date 17:36 2024/11/23 
     * @param beanName 
     * @return java.lang.Object
     **/
    Object getSingleton(String beanName);

}
```

单例接口主要功能是定义一个从容器中获取单例bean对象的接口。


### 3.5、单例接口实现类：DefaultSingletonBeanRegistry

```java
/**
 * 默认的单例接口的实现
 */
public class DefaultSingletonBeanRegistry implements SingletonBeanRegistry {

    
//    单例容器实现
    private final Map<String, Object> singletonObjects = new HashMap<>();

    /**
     * 从容器中获取单例对象
     * @author yourname
     * @date 17:35 2024/11/23 
     * @param beanName 
     * @return java.lang.Object
     **/
    @Override
    public Object getSingleton(String beanName) {
        return singletonObjects.get(beanName);
    }

    /**
     * 向容器中增加单例对象
     * @author yourname
     * @date 17:35 2024/11/23 
     * @param beanName
     * @param singletonObject 
     **/
    protected void addSingleton(String beanName, Object singletonObject) {
        singletonObjects.put(beanName, singletonObject);
    }

}
```

DefaultSingletonBeanRegistry是单例接口的默认实现类，主要重写getSingleton()方法获取单例bean对象，同时实现了自己的受保护的addSingleton方法，主要负责向容器中添加对象操作。这个方法可以被继承此类的其他类调用。包括：AbstractBeanFactory 以及继承的 DefaultListableBeanFactory 调用。

另外这里就体现了在抽象类中对接口的功能进行扩充；

### 3.6、定义BeanDefinition

```java
public class BeanDefinition {

//    定义类信息
    private Class beanClass;

    public BeanDefinition(Class beanClass) {
        this.beanClass = beanClass;
    }

    public Class getBeanClass() {
        return beanClass;
    }

    public void setBeanClass(Class beanClass) {
        this.beanClass = beanClass;
    }
}
```
在 Bean 定义类中已经把上一章节中的 Object bean 替换为 Class，这样就可以把 Bean 的实例化操作放到容器中处理了；


## 4、测试

### 4.1、测试用例

**Service对象**

```java
public class PeopleService {
    public void queryUserInfo(){
        System.out.println("查询用户信息");
    }
}
```

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
        PeopleService peopleService1 = (PeopleService) beanFactory.getBean("peopleService");
        peopleService1.queryUserInfo();

        // 4.第二次获取 bean from Singleton
        PeopleService peopleService2 = (PeopleService) beanFactory.getBean("peopleService");
        peopleService2.queryUserInfo();
    }
```
> Bean对象定义的注册，还没有完全交给容器，因此我们手动注册Bean对象定义；

**结果：**

```java
查询用户信息
查询用户信息

Process finished with exit code 0
```

单测中，还是遵循实例化BeanFactory(定义了获取Bean对象功能)，注册BeanDefiniton定义，获取bean对象使用等三步，但是在第二步中注册bean定义的过程中，可以看到注册的是bean的class信息，而不是实例化对象，因此本章节为止实现了将bean的创建交给了容器，另外两次从容器中获取bean对象，获取的是同一个bean对象，因此也实现了单例bean的功能。

## 5、总结

**AbstractBeanFactory抽象类使用模板设计模式，实现接口中的一部分方法，让子类去实现属于自己的方法，抽象类中只定义调用过程（也可以理解为实现业务过程调用），而没有做具体的实现，实现留给子类。**

**面向接口编程，一个接口只做一件事，定义简单的接口，然后让子接口，抽象类和具体类通过实现和继承的方法对接口功能进行扩展：**

* **接口继承接口，实现对接口的功能扩展，方法扩展**
* **抽象类实现接口，一般是定义业务的调用过程，不会做具体的实现**
* **抽象类继承抽象类，一般是对抽象类功能的扩充，或者某一个抽象类需要具备其他功能，就可以通过继承其他抽象类实现**
* **具体类实现接口或者继承抽象类，一般做具体的方法实现，配合接口或者父类引用的调用关系。**

在 Spring Bean 容器的实现类中要重点关注****类之间的职责和关系****，几乎所有的程序功能设计都离不开接口、抽象类、实现、继承，而这些不同特性类的使用就可以非常好的隔离开类的功能职责和作用范围。而这样的知识点也是在学习手写 Spring Bean 容器框架过程非常重要的知识。
