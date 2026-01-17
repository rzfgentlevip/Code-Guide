---
title: 10、容器事件和事件监听器
icon: file
order: 10
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
permalink: /project-docs/project/springframework/容器事件和事件监听器/
---

<!-- TOC -->

- [10、容器事件和事件监听器](#10容器事件和事件监听器)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类实现](#31核心类实现)
    - [3.2、定义事件](#32定义事件)
    - [3.3、事件广播器](#33事件广播器)
    - [3.4、事件发布者的定义和实现](#34事件发布者的定义和实现)
    - [3.5、事件注册和调用](#35事件注册和调用)
    - [3.6、监听者接口](#36监听者接口)
  - [4、测试](#4测试)
    - [4.1、创建事件](#41创建事件)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)

<!-- /TOC -->


# 10、容器事件和事件监听器

## 1、目标

在 Spring 中有一个 Event 事件功能，它可以提供事件的定义、发布以及监听事件来完成一些自定义的动作。比如你可以定义一个新用户注册的事件，当有用户执行注册完成后，在事件监听中给用户发送一些优惠券和短信提醒，这样的操作就可以把属于基本功能的注册和对应的策略服务分开，降低系统的耦合。以后在扩展注册服务，比如需要添加风控策略、添加实名认证、判断用户属性等都不会影响到依赖注册成功后执行的动作。

那么在本章节我们需要以观察者模式的方式，设计和实现 Spring Event 的容器事件和事件监听器功能，最终可以让我们在现有实现的 Spring 框架中可以定义、监听和发布自己的事件信息

## 2、设计

其实事件的设计本身就是一种观察者模式的实现，它所要解决的就是一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

在功能实现上我们需要定义出事件类、事件监听、事件发布，而这些类的功能需要结合到 Spring 的 AbstractApplicationContext#refresh()，以便于处理事件初始化和注册事件监听器的操作。



![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1717370600081-b60138a3-4376-4e32-b347-2ae97ca761be.png)

+ 在整个功能实现过程中，仍然需要在面向用户的应用上下文 AbstractApplicationContext 中添加相关事件内容，包括：**初始化事件发布者、注册事件监听器、发布容器刷新完成事件**。
+ 使用观察者模式定义事件类、监听类、发布类，同时还需要完成一个广播器的功能，接收到事件推送时进行分析处理符合监听事件接受者感兴趣的事件，也就是使用 isAssignableFrom 进行判断。
+ isAssignableFrom 和 instanceof 相似，不过 isAssignableFrom 是用来判断子类和父类的关系的，或者接口的实现类和接口的关系的，默认所有的类的终极父类都是Object。如果A.isAssignableFrom(B)结果是true，证明B可以转换成为A,也就是A可以由B转换而来。

## 3、实现

### 3.1、核心类实现


![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1717369194337-bd705f47-ba99-4001-bf66-ffd04aa0a3f2.png)

+ 以上整个类关系图以围绕实现 event 事件定义、发布、监听功能实现和把事件的相关内容使用 AbstractApplicationContext#refresh 进行注册和处理操作。
+ 实现中主要以扩展spring context上下文为主，首先定义出事件定义，广播事件，接收时间等功能，然后将功能集成到spring context中。
+ 其中事件定义接口：ApplicationEvent，继承EventObject事件接口。
+ ApplicationEventPublisher：事件发布者，主要将事件发布出去
+ 事件广播接口：ApplicationEventMulticaster定义广播事件的主要功能。
+ 事件监听：ApplicationListener主要实现监听者监听，需要监听的监听者实现此接口方法即可。
+ ApplicationContext 容器继承事件发布功能接口 ApplicationEventPublisher，并在实现类中提供事件监听功能。
+ ApplicationEventMulticaster 接口是注册监听器和发布事件的广播器，提供添加、移除和发布事件方法。
+ 最后是发布容器关闭事件，这个仍然需要扩展到 AbstractApplicationContext#close 方法中，由注册到虚拟机的钩子实现。



### 3.2、定义事件

```java
/**
 *  Class to be extended by all application events. Abstract as it
 *  * doesn't make sense for generic events to be published directly.
 * @Description
 * @Author bugcode.online
 * @Date 2024/5/18 10:20
 */
public abstract class ApplicationEvent extends EventObject {
    /**
     * Constructs a prototypical Event.
     *
     * @param source The object on which the Event initially occurred.
     * @throws IllegalArgumentException if source is null.
     */
    public ApplicationEvent(Object source) {
        super(source);
    }
}
```

事件的顶层抽象类继承EventObject，所有的事件定义和实现都要在ApplicationEvent抽象类之上。

**应用事件上下文**

```java
public class ApplicationContextEvent extends ApplicationEvent {
    /**
     * Constructs a prototypical Event.
     *
     * @param source The object on which the Event initially occurred.
     * @throws IllegalArgumentException if source is null.
     */
    public ApplicationContextEvent(Object source) {
        super(source);
    }

    /**
     * Get the <code>ApplicationContext</code> that the event was raised for.
     */
    public final ApplicationContext getApplicationContext() {
        return (ApplicationContext) getSource();
    }

}
```

**关闭事件ContextClosedEvent**

```java
public class ContextClosedEvent extends ApplicationContextEvent {
    /**
     * Constructs a prototypical Event.
     *
     * @param source The object on which the Event initially occurred.
     * @throws IllegalArgumentException if source is null.
     */
    public ContextClosedEvent(Object source) {
        super(source);
    }
}
```

**刷新事件ContextRefreshedEvent**

```java
public class ContextRefreshedEvent extends ApplicationContextEvent {
    /**
     * Constructs a prototypical Event.
     *
     * @param source The object on which the Event initially occurred.
     * @throws IllegalArgumentException if source is null.
     */
    public ContextRefreshedEvent(Object source) {
        super(source);
    }
}
```



+ ApplicationEvent 是定义事件的抽象类，所有的事件包括**关闭、刷新，以及用户自己实现的事件**，都需要继承这个类。
+ ContextClosedEvent、ContextRefreshedEvent，分别是 Spring 框架自己实现的两个事件类，可以用于监听刷新和关闭动作。



### 3.3、事件广播器

```java
public interface ApplicationEventMulticaster {

    /**
     * Add a listener to be notified of all events.
     * @param listener the listener to add
     */
    void addApplicationListener(ApplicationListener<?> listener);

    /**
     * Remove a listener from the notification list.
     * @param listener the listener to remove
     */
    void removeApplicationListener(ApplicationListener<?> listener);

    /**
     * Multicast the given application event to appropriate listeners.
     * @param event the event to multicast
     */
    void multicastEvent(ApplicationEvent event);
}
```

 在事件广播器中定义了添加监听和删除监听的方法以及一个广播事件的方法 multicastEvent 最终推送时间消息也会经过这个接口方法来处理谁该接收事件。  

AbstractApplicationEventMulticaster

```java
public abstract class AbstractApplicationEventMulticaster implements ApplicationEventMulticaster, BeanFactoryAware {

    public final Set<ApplicationListener<ApplicationEvent>> applicationListeners = new LinkedHashSet();

    private BeanFactory beanFactory;

    @Override
    public void addApplicationListener(ApplicationListener<?> listener) {
        applicationListeners.add((ApplicationListener<ApplicationEvent>)listener);
    }

    @Override
    public void removeApplicationListener(ApplicationListener<?> listener) {
        applicationListeners.remove(listener);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    /**
     * Return a Collection of ApplicationListeners matching the given
     * event type. Non-matching listeners get excluded early.
     * @param event the event to be propagated. Allows for excluding
     * non-matching listeners early, based on cached matching information.
     * @return a Collection of ApplicationListeners
     * @see
     */
    protected Collection<ApplicationListener> getApplicationListeners(ApplicationEvent event) {
        LinkedList<ApplicationListener> allListeners = new LinkedList<ApplicationListener>();
        for (ApplicationListener<ApplicationEvent> listener : applicationListeners) {
            if (supportsEvent(listener, event)) allListeners.add(listener);
        }
        return allListeners;
    }

    /**
     * 监听器是否对该事件感兴趣
     */
    protected boolean supportsEvent(ApplicationListener<ApplicationEvent> applicationListener, ApplicationEvent event) {
        Class<? extends ApplicationListener> listenerClass = applicationListener.getClass();

        // 按照 CglibSubclassingInstantiationStrategy、SimpleInstantiationStrategy 不同的实例化类型，需要判断后获取目标 class
        Class<?> targetClass = ClassUtils.isCglibProxyClass(listenerClass) ? listenerClass.getSuperclass() : listenerClass;
        Type genericInterface = targetClass.getGenericInterfaces()[0];

        Type actualTypeArgument = ((ParameterizedType) genericInterface).getActualTypeArguments()[0];
        String className = actualTypeArgument.getTypeName();
        Class<?> eventClassName;
        try {
            eventClassName = Class.forName(className);
        } catch (ClassNotFoundException e) {
            throw new BeansException("wrong event class name: " + className);
        }
        // 判定此 eventClassName 对象所表示的类或接口与指定的 event.getClass() 参数所表示的类或接口是否相同，或是否是其超类或超接口。
        // isAssignableFrom是用来判断子类和父类的关系的，或者接口的实现类和接口的关系的，默认所有的类的终极父类都是Object。如果A.isAssignableFrom(B)结果是true，证明B可以转换成为A,也就是A可以由B转换而来。
        return eventClassName.isAssignableFrom(event.getClass());
    }
}
```

+ AbstractApplicationEventMulticaster 是对事件广播器的公用方法提取，在这个类中可以实现一些基本功能，避免所有直接实现接口放还需要处理细节。
+ 除了像 addApplicationListener、removeApplicationListener，这样的通用方法，这里这个类中主要是对 getApplicationListeners 和 supportsEvent 的处理。
+ getApplicationListeners 方法主要是摘取符合广播事件中的监听处理器，具体过滤动作在 supportsEvent 方法中。
+ 在 supportsEvent 方法中，主要包括对Cglib、Simple不同实例化需要获取目标Class，Cglib代理类需要获取父类的Class，普通实例化的不需要。接下来就是通过提取接口和对应的 ParameterizedType 和 eventClassName，方便最后确认是否为子类和父类的关系，以此证明此事件归这个符合的类处理。_可以参考代码中的注释_

_SimpleApplicationEventMulticaster_

```java
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster{
    @Override
    public void multicastEvent(ApplicationEvent event) {
        for (final ApplicationListener listener : getApplicationListeners(event)) {
            listener.onApplicationEvent(event);
        }
    }

    public SimpleApplicationEventMulticaster(BeanFactory beanFactory) {
        setBeanFactory(beanFactory);
    }
}
```

实现一个具体的广播器，重写广播事件方法。

### 3.4、事件发布者的定义和实现

```java
public interface ApplicationEventPublisher {

    /**
     * Notify all listeners registered with this application of an application
     * event. Events may be framework events (such as RequestHandledEvent)
     * or application-specific events.
     * @param event the event to publish
     */
    void publishEvent(ApplicationEvent event);
}
```

 ApplicationEventPublisher 是整个一个事件的发布接口，所有的事件都需要从这个接口发布出去。  

### 3.5、事件注册和调用

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {

    public static final String APPLICATION_EVENT_MULTICASTER_BEAN_NAME = "applicationEventMulticaster";

//    事件广播器
    private ApplicationEventMulticaster applicationEventMulticaster;

    /**
     * applicaion context的上下文方法
     * 事件动作在刷新方法中传进来
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

        // 6. 初始化事件发布者，注册applicationEventMulticaster 到map
        initApplicationEventMulticaster();

        // 7. 注册事件监听器
        registerListeners();

        // 6. 提前实例化单例Bean对象
        beanFactory.preInstantiateSingletons();

        // 9. 发布容器刷新完成事件
        finishRefresh();
    }

    /**
     * eventListener新增
     */
    private void initApplicationEventMulticaster(){
        ConfigurableListableBeanFactory beanFactory = getBeanFactory();
        applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);

        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME,applicationEventMulticaster);

    }

    /**
     * 注册监听者
     */
    private void registerListeners() {
        Collection<ApplicationListener> applicationListeners = getBeansOfType(ApplicationListener.class).values();
        for (ApplicationListener listener : applicationListeners) {
            applicationEventMulticaster.addApplicationListener(listener);
        }
    }

    /**
     * 完成刷新
     */
    private void finishRefresh() {
        publishEvent(new ContextRefreshedEvent(this));
    }

    /**
     * 发布事件
     * @param event the event to publish
     */
    @Override
    public void publishEvent(ApplicationEvent event) {
        applicationEventMulticaster.multicastEvent(event);
    }


    @Override
    public void close() {
//        发布容器关闭事件
        publishEvent(new ContextClosedEvent(this));

//        执行销毁单例bean的销毁方法
        getBeanFactory().destroySingletons();
    }

}
```



+ 在抽象应用上下文 AbstractApplicationContext#refresh 中，主要新增了 初始化事件发布者、注册事件监听器、发布容器刷新完成事件，三个方法用于处理事件操作。
+ 初始化事件发布者(initApplicationEventMulticaster)，主要用于实例化一个 SimpleApplicationEventMulticaster，这是一个事件广播器。
+ 注册事件监听器(registerListeners)，通过 getBeansOfType 方法获取到所有从 spring.xml 中加载到的事件配置 Bean 对象。
+ 发布容器刷新完成事件(finishRefresh)，发布了第一个服务器启动完成后的事件，这个事件通过 publishEvent 发布出去，其实也就是调用了 applicationEventMulticaster.multicastEvent(event); 方法。
+ 最后是一个 close 方法中，新增加了发布一个容器关闭事件。publishEvent(new ContextClosedEvent(this));

### 3.6、监听者接口

```java
public interface EventListener {
}
```



**扩展监听者接口**

```java
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    /**
     * Handle an application event.
     * @param event the event to respond to
     */
    void onApplicationEvent(E event);

}
```

定义监听者接口，所有想要监听事件的监听者都需要实现此接口。

## 4、测试

### 4.1、创建事件

```java
public class CustomEvent extends ApplicationContextEvent {

    private Long id;
    private String message;

    /**
     * Constructs a prototypical Event.
     *
     * @param source The object on which the Event initially occurred.
     * @throws IllegalArgumentException if source is null.
     */
    public CustomEvent(Object source, Long id, String message) {
        super(source);
        this.id = id;
        this.message = message;
    }
}
```

定义一个事件，事件会被广播器发布，然后被所有的监听者监听到。

**创建监听器**

```java
public class CustomEventListener implements ApplicationListener<CustomEvent> {

    @Override
    public void onApplicationEvent(CustomEvent event) {
        System.out.println("收到：" + event.getSource() + "消息;时间：" + new Date());
        System.out.println("消息：" + event.getId() + ":" + event.getMessage());
    }

}
```

- 这个是一个用于监听 CustomEvent 事件的监听器，这里你可以处理自己想要的操作，比如一些用户注册后发送优惠券和短信通知等。
- 另外是关于 `ContextRefreshedEventListener implements ApplicationListener<ContextRefreshedEvent>、ContextClosedEventListener implements ApplicationListener<ContextClosedEvent>` 监听器，这里就不演示了，可以参考下源码。



**刷新监听**

```java
public class ContextRefreshedEventListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("刷新事件：" + this.getClass().getName());
    }

}
```



xml配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans>

    <bean class="bugcode.online.springframework.event.ContextRefreshedEventListener"/>
    <bean class="bugcode.online.springframework.event.CustomEventListener"/>
    <bean class="bugcode.online.springframework.event.ContextClosedEventListener"/>

</beans>
```



### 4.2、测试结果

```java
   @Test
    public void test_event() {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        applicationContext.publishEvent(new CustomEvent(applicationContext, 1019129009086763L, "成功了！"));

        applicationContext.registerShutdownHook();
    }
```





**测试结果**

```java
刷新事件：bugcode.online.springframework.event.ContextRefreshedEventListener$$EnhancerByCGLIB$$6a470f4
收到：bugcode.online.springframework.context.support.ClassPathXmlApplicationContext@35fb3008消息;时间：Mon Jun 03 06:56:21 CST 2024
消息：1019129009086763:成功了！
关闭事件：bugcode.online.springframework.event.ContextClosedEventListener$$EnhancerByCGLIB$$6be9cfee

Process finished with exit code 0
```



## 5、总结

 在整个手写 Spring 框架的学习过程中，可以逐步看到很多设计模式的使用，比如：简单工厂BeanFactory、工厂方法FactoryBean、策略模式访问资源，现在又实现了一个观察者模式的具体使用。所以学习 Spring 的过程中，要更加注意关于设计模式的运用，这是你能读懂代码的核心也是学习的重点  



使用观察者模式实现监听功能