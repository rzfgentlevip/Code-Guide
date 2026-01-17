---
title: 8、Aware感知容器接口
icon: file
order: 8
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
permalink: /project-docs/project/springframework/Aware感知容器接口/
---


<!-- TOC -->

- [8、Aware感知容器接口](#8aware感知容器接口)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类图实现](#31核心类图实现)
    - [3.2、Aware顶层感知接口](#32aware顶层感知接口)
    - [3.3、容器感知类实现](#33容器感知类实现)
    - [3.4、注册BeanPostProcessor](#34注册beanpostprocessor)
    - [3.5、感知调用操作](#35感知调用操作)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)
    - [5.1、Bean对象生命周期](#51bean对象生命周期)

<!-- /TOC -->

# 8、Aware感知容器接口

## 1、目标

目前已实现的 Spring 框架，在 Bean 操作上能提供出的能力，包括：Bean 对象的定义和注册，以及在操作 Bean 对象过程中执行的，BeanFactoryPostProcessor、BeanPostProcessor、InitializingBean、DisposableBean，以及在 XML 新增的一些配置处理，让我们可以 Bean 对象有更强的操作性。

如果我们想获得 Spring 框架提供的 BeanFactory、ApplicationContext、BeanClassLoader等这些能力做一些扩展框架的使用时该怎么操作呢。所以我们本章节希望在 Spring 框架中提供一种能感知容器操作的接口，如果谁实现了这样的一个接口，就可以获取接口入参中的各类能力。

## 2、设计

首先需要定义一个标记性接口，接口中只是起到一个标记性的作用，不一会定义具体的方法，而具体的功能由继承此接口的其他功能性接口定义具体方法，最终这个接口就可以通过instanceof进行判断和调用。

![aware接口设计](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/aware.png)

- 定义接口 Aware，在 Spring 框架中它是一种感知标记性接口，具体的子类定义和实现能感知容器中的相关对象。也就是通过这个桥梁，向具体的实现类中提供容器服务
- 继承 Aware 的接口包括：BeanFactoryAware、BeanClassLoaderAware、BeanNameAware和ApplicationContextAware，当然在 Spring 源码中还有一些其他关于注解的，不过目前我们还是用不到。
- 在具体的接口实现过程中你可以看到，一部分(BeanFactoryAware、BeanClassLoaderAware、BeanNameAware)在 factory 的 support 文件夹下，另外 ApplicationContextAware 是在 context 的 support 中，这是因为不同的内容获取需要在不同的包下提供。所以，在 AbstractApplicationContext 的具体实现中会用到向 beanFactory 添加 BeanPostProcessor 内容的 ApplicationContextAwareProcessor 操作，最后由 AbstractAutowireCapableBeanFactory 创建 createBean 时处理相应的调用操作。
## 3、实现

### 3.1、核心类图实现

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/spring-application-aware.drawio.png)

- Aware 有四个继承的接口，其他这些接口的继承都是为了继承一个标记，有了标记的存在更方便类的操作和具体判断实现。
- 另外由于 ApplicationContext 并不是在 AbstractAutowireCapableBeanFactory 中 createBean 方法下的内容，所以需要像容器中注册 addBeanPostProcessor ，再由 createBean 统一调用 applyBeanPostProcessorsBeforeInitialization 时进行操作

### 3.2、Aware顶层感知接口
```java
/**
 * @Description 标记类接口，实现该接口可以被Spring容器感知
 * @Author bugcode.online
 * @Date 2024/5/18 8:24
 */
public interface Aware {
}
```

- 在 Spring 中有特别多类似这样的标记接口的设计方式，它们的存在就像是一种标签一样，可以方便统一摘取出属于此类接口的实现类，通常会有 instanceof 一起判断使用。
- 另外，从学习到开发框架过程中，我们也能很直观的感受到，spring中定义了很多的顶层接口，顶层接口的设计一般都很简单，然后子接口和抽象类在一层一层的去扩展实现，逐步完善接口对应的功能，很能体现出一个接口只定义一个单一的职责---接口单一职责。
- 所以我们再写代码过程中，也要学会定义单一接口，然后有子接口和抽象类逐步去扩展接口内容：
  - 接口中定义职责，定义接口的职责范围，其实就是声明抽象方法和，通过接口的继承，扩展接口职责
  - 抽象类负责定义业务过程，即调用关系，子类负责实现具体的职责。
- 根据上面两条职责，一个接口的设计就会职责分明，很容易扩展，如果想要扩展某个接口职责，扩大范围，就新创建接口继承父类接口，然后再去实现新接口。当然也不建议这样做，一般一个接口的功能很单一，不建议将一个接口的职责范围扩大很大，这样反而不利于扩展。


### 3.3、容器感知类实现

BeanClassLoaderAware感知类

```java
/**
 * @Description
 * @Author bugcode.online
 * @Date 2024/5/18 8:25
 */
public interface BeanClassLoaderAware extends Aware{

    void setBeanClassLoader(ClassLoader classLoader);
}
```

实现此接口，就可以感知到当前Bean对象所述的ClassLoader.

**BeanFactoryAware感知类**

```java
public interface BeanFactoryAware extends Aware {

    void setBeanFactory(BeanFactory beanFactory) throws BeansException;
}
```

实现此接口，就可以感知到当前Bean对象所述的BeanFactory。

**BeanNameAware感知类**

```java
public interface BeanNameAware extends Aware{

    void setBeanName(String name);

}
```

实现此接口，就能够感知当前Bean对象的BeanName。

**ApplicationContextAware感知类**

```java
public interface ApplicationContextAware extends Aware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;

}
```
实现此接口，既能感知到所属的 ApplicationContext。


### 3.4、注册BeanPostProcessor

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {

    /**
     * applicaion context的上下文方法
     * @throws BeansException
     */
    @Override
    public void refresh() throws BeansException {
        // 1. 创建 BeanFactory，并加载 BeanDefinition
        refreshBeanFactory();

        // 2. 获取 BeanFactory,这一步已经将bean的定义添加到了容器中
        ConfigurableListableBeanFactory beanFactory = getBeanFactory();

        // 3. 添加 ApplicationContextAwareProcessor，让继承自 ApplicationContextAware 的 Bean 对象都能感知所属的 ApplicationContext
        beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));

        // 4. 在 Bean 实例化（创建）之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
        invokeBeanFactoryPostProcessors(beanFactory);

        // 5. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
        registerBeanPostProcessors(beanFactory);

        // 6. 提前实例化单例Bean对象
        beanFactory.preInstantiateSingletons();
    }
    .....
}
```

+ refresh() 方法就是整个 Spring 容器的操作过程，与上一章节对比，本次新增加了关于 addBeanPostProcessor 的操作。
+ 添加 ApplicationContextAwareProcessor，让继承自 ApplicationContextAware 的 Bean 对象都能感知所属的 ApplicationContext。

### 3.5、感知调用操作

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {

//    实例化bean对象带参数
    private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

    /**
     * 带参数的创建bean对象
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
            bean = createBeanInstance(beanDefinition,beanName,args);
            // 给 Bean 填充属性
            applyPropertyValues(beanName, bean, beanDefinition);

            // 执行 Bean 的初始化方法和 BeanPostProcessor 的前置和后置处理方法
            bean = initializeBean(beanName, bean, beanDefinition);

        } catch (Exception e) {
            throw new BeansException("Instantiation of bean failed", e);
        }


        // 注册实现了 DisposableBean 接口的 Bean 对象
        registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);

        addSingleton(beanName, bean);
        return bean;
    }

    /**
     * 前置处理器执行完成后才会执行init()初始化方法
     * 执行bean的初始化方法，前置 后置处理器方法
     * @param beanName
     * @param bean
     * @param beanDefinition
     * @return
     */
    private Object initializeBean(String beanName, Object bean, BeanDefinition beanDefinition) {

        // invokeAwareMethods 引用Aware接口
        if (bean instanceof Aware) {
//            感知BeanFactory
            if (bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware) bean).setBeanFactory(this);
            }
            if (bean instanceof BeanClassLoaderAware){
                ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
            }
            if (bean instanceof BeanNameAware) {
                ((BeanNameAware) bean).setBeanName(beanName);
            }
        }


        // 1. 执行 BeanPostProcessor Before 处理
        Object wrappedBean = applyBeanPostProcessorsBeforeInitialization(bean, beanName);

        // 待完成内容：invokeInitMethods(beanName, wrappedBean, beanDefinition);
        // 执行 Bean 对象的初始化方法
        try {
            invokeInitMethods(beanName, wrappedBean, beanDefinition);
        } catch (Exception e) {
            throw new BeansException("Invocation of init method of bean[" + beanName + "] failed", e);
        }
        // 2. 执行 BeanPostProcessor After 处理
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        return wrappedBean;
    }

    @Override
    public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName) throws BeansException {
        Object result = existingBean;
        for (BeanPostProcessor processor : getBeanPostProcessors()) {
            Object current = processor.postProcessBeforeInitialization(result, beanName);
            if (null == current) return result;
            result = current;
        }
        return result;
    }

    @Override
    public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
        Object result = existingBean;
        for (BeanPostProcessor processor : getBeanPostProcessors()) {
            Object current = processor.postProcessAfterInitialization(result, beanName);
            if (null == current) return result;
            result = current;
        }
        return result;
    }

}

```

**AbstractBeanFactory**

```java
public abstract class AbstractBeanFactory extends DefaultSingletonBeanRegistry implements ConfigurableBeanFactory {


    /** ClassLoader to resolve bean class names with, if necessary */
    private ClassLoader beanClassLoader = ClassUtils.getDefaultClassLoader();

    /** BeanPostProcessors to apply in createBean */
    private final List<BeanPostProcessor> beanPostProcessors = new ArrayList<BeanPostProcessor>();

    /**
     * 向集合中添加处理器
     * @param beanPostProcessor
     */
    @Override
    public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {

        this.beanPostProcessors.remove(beanPostProcessor);
        this.beanPostProcessors.add(beanPostProcessor);
    }
    /**
     * 返回当前类的类加载器
     * @return
     */
    public ClassLoader getBeanClassLoader() {
        return this.beanClassLoader;
    }
}

```

+ 首先在 initializeBean 中，通过判断 bean instanceof Aware，调用了三个接口方法，BeanFactoryAware.setBeanFactory(this)、BeanClassLoaderAware.setBeanClassLoader(getBeanClassLoader())、BeanNameAware.setBeanName(beanName)，这样就能通知到已经实现了此接口的类。
+ 另外我们还向 BeanPostProcessor 中添加了 ApplicationContextAwareProcessor，此时在这个方法中也会被调用到具体的类实现，得到一个 ApplicationContex 属性。

## 4、测试

### 4.1、测试用例

**DAO接口**

```java
public class PeopleDao {

    private static Map<String,String> map = new HashMap<>();

    public void initDataMethod(){
        System.out.println("执行：init-method");
        map.put("10001", "工程师");
        map.put("10002", "秘书");
        map.put("10003", "实习生");
    }

    public void destroyDataMethod(){
        System.out.println("执行：destroy-method");
        map.clear();
    }

    public String queryUserName(String uId) {
        return map.get(uId);
    }
}
```

**Service接口**

```java
public class PeopleService implements BeanNameAware, BeanClassLoaderAware, ApplicationContextAware, BeanFactoryAware {
    private String id;
    private String company;
    private String location;
    private PeopleDao peopleDao;

    private ApplicationContext applicationContext;

    private BeanFactory beanFactory;

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + peopleDao.queryUserName(id)+"  公司名称："+company+"  公司地点："+location);
    }


    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {

        System.out.println("ClassLoader:"+classLoader);
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("Bean Name is：" + name);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }
}
```

配置类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>

    <bean id="peopleDao" class="bugcode.online.springframework.bean.PeopleDao" init-method="initDataMethod" destroy-method="destroyDataMethod"/>

    <bean id="peopleService" class="bugcode.online.springframework.bean.PeopleService">
        <property name="id" value="10001"/>
        <property name="company" value="腾讯"/>
        <property name="location" value="上海"/>
        <property name="peopleDao" ref="peopleDao"/>
    </bean>

</beans>
```

### 4.2、测试结果

```java
@Test
    public void testInitAndDestroy(){
        // 1.初始化 BeanFactory
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        applicationContext.registerShutdownHook();

        // 2. 获取Bean对象调用方法
        PeopleService peopleService = applicationContext.getBean("peopleService", PeopleService.class);
        peopleService.queryUserInfo();

        System.out.println("ApplicationContextAware："+peopleService.getApplicationContext());
        System.out.println("BeanFactoryAware："+peopleService.getBeanFactory());
    }
```



测试结果

```java
执行：init-method
ClassLoader:sun.misc.Launcher$AppClassLoader@18b4aac2
Bean Name is：peopleService
查询用户信息：工程师  公司名称：腾讯  公司地点：上海
ApplicationContextAware：bugcode.online.springframework.context.support.ClassPathXmlApplicationContext@62043840
BeanFactoryAware：bugcode.online.springframework.beans.factory.support.DefaultListableBeanFactory@5315b42e
执行：destroy-method

Process finished with exit code 0
```

## 5、总结

重点理解Aware感知的设计，如果想要感知某一个类，首先设计一个顶级的接口，顶级接口不需要定义方法，让扩展顶级接口的子接口自己定义方法，然后某一个Service去实现子接口中的方法，在程序类中使用bean instanceof判断即可。



思考Aware接口标记的方法是如何串到spring bean的生命周期中去的：

ApplicationContext的感知是以实现前后置处理器接口的方式将其注册到保存前后置处理器的map中，然后再执行前后置处理器方法的时候实现调用，即ApplicationContextAwareProcessor接口。

其他标记接口是通过在初始化方法中的判断实现：

```java
if (bean instanceof Aware) {
      //感知BeanFactory
    if (bean instanceof BeanFactoryAware) {
        ((BeanFactoryAware) bean).setBeanFactory(this);
    }
    if (bean instanceof BeanClassLoaderAware){
        ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
    }
    if (bean instanceof BeanNameAware) {
        ((BeanNameAware) bean).setBeanName(beanName);
    }
}
```



### 5.1、Bean对象生命周期

1. 加载BeanDefinition
2. 注册BeanDefinition
3. 修改bean定义(BeanDefinition)
4. 实例化Bean对象，调用构造方法
5. 填充属性信息
6. 执行前置处理器
7. 执行初始化方法
8. 执行Aware方法执行后置处理器
9. 执行销毁方法



Aware

ApplicationContext

思考每一个接口功能设计。