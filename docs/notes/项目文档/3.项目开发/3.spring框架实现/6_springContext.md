---
title: 6、实现Spring-Context上下文
icon: file
order: 6
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
permalink: /project-docs/project/springframework/实现Spring-Context上下文/
---
<!-- TOC -->

- [6、实现Spring上下文](#6实现spring上下文)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类图](#31核心类图)
    - [3.2、定义 BeanFactoryPostProcessor](#32定义-beanfactorypostprocessor)
    - [3.3、定义BeanPostProcessor](#33定义beanpostprocessor)
    - [3.4、上下文接口](#34上下文接口)
    - [3.5、应用上下文抽象类实现](#35应用上下文抽象类实现)
    - [3.6、获取Bean工厂和加载资源](#36获取bean工厂和加载资源)
    - [3.7、上下文中对配置加载](#37上下文中对配置加载)
    - [3.8、应用上下文具体实现类](#38应用上下文具体实现类)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、不使用上下文](#42不使用上下文)
    - [4.3、使用应用上下文](#43使用应用上下文)
  - [5、小结](#5小结)

<!-- /TOC -->

# 6、实现Spring上下文


## 1、目标

在第五章中，我们实现了spring自动读取xml配置文件，重点实现两个功能，**解析xml配置和自动注册bean对象**，然后将此模块功能集成到spring容器中。

但是这种模块化的集成，在模块与模块衔接处，仍然需要用户去创建对象，然后读取配置文件自动实现解析和注册，就像这种方法的调用：
```java
 public void test_xml() {
        // 1.初始化 BeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2. 读取配置文件&注册Bean 这一步自动读取xml文件，将文件中定义的bean对象加载到容器中
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        reader.loadBeanDefinitions("classpath:spring.xml");

        // 3. 获取Bean对象调用方法
        PeopleService peopleService = (PeopleService)beanFactory.getBean("peopleService", PeopleService.class);
        peopleService.queryUserInfo();
    }
```

我们的目标是spring容器看起来就像是一个黑盒子对于用户来说，用户不知道spring具体内部做了什么工作，而当前的模块式组装方式更像是面对spring容器本身，而非应用程序，所以在模块之上，还应该在封装上下文操作，只对用户暴漏可用的上下文接口，然后指明配置文件的路径，spring就自动根据路径下的配置自动解析和注册bean对象。

并且开发spring应用上下文，可以对外暴露一些标准化的扩展接口让用户实现。

## 2、设计

到目前为止，我们的spring容器虽然可以根据用户的配置**自动进行bean定义加载，bean对象创建和初始化以及到最后注入容器**，但是对于程序来说有时候可能需要一些自定义模块来扩展bean:
1. 比如在bean定义加载完成后修改bean的定义
2. bean对象实例化前后做一些动作

目前spring容器还不具备这种扩展操作，所以我们需要再bean定义加载完成后以及bean对象实例化前后，向用户提供一些接口，用户简单的实现就可以完成功能的扩展，这才符合微模块化编程的准则。

在spring的实现中，bean的生命周期内有两个可以扩展bean功能的接口，**BeanFactoryPostProcessor和BeanPostProcessor**接口，BeanFactoryPostProcessor接口可以让用户在加载完Bean定义之后对定义进行修改操纵，BeanPostProcessor接口提供了bean实例化前后的扩展操作，即前置和后置处理器操作。

- BeanFactoryPostProcessor:对Bean的定义进行修改
- BeanPostProcessor:Bean对象实例化前后进行一些修改

但是如果只是添加两个扩展功能的接口，那么对于用户来说还需要考虑如何将这两个功能接口集成到Bean对象的周期中，对用户来说显然增加了开发成本，所以需要再spring容器中对这两个接口进行包装，让用户简单的实现这两个接口，spring容器就可以感知到这两个服务的存在然后自动触发执行。

**设计图**:

![spring-context-design](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408021018239.png)

- 满足于对 Bean 对象扩展的两个接口，其实也是 Spring 框架中非常具有重量级的两个接口：BeanFactoryPostProcessor 和 BeanPostProcessor，也几乎是大家在使用 Spring 框架额外新增开发自己组建需求的两个必备接口。
- BeanFactoryPostProcessor，是由 Spring 框架组建提供的容器扩展机制，允许在 Bean 对象注册后但未实例化之前，对 Bean 的定义信息 BeanDefinition 执行修改操作。
- BeanPostProcessor，也是 Spring 提供的扩展机制，不过 BeanPostProcessor 是在 Bean 对象实例化之后修改 Bean 对象，也可以替换 Bean 对象。这部分与后面要实现的 AOP 有着密切的关系。
- 同时如果只是添加这两个接口，不做任何包装，那么对于使用者来说还是非常麻烦的。我们希望于开发 Spring 的上下文操作类，把相应的 XML 加载 、注册、实例化以及新增的修改和扩展都融合进去，让 Spring 可以自动扫描到我们的新增服务，便于用户使用。

## 3、实现

### 3.1、核心类图

![spring-application-context.drawio.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408021019280.png)

应用上下文的顶级接口ApplicationContext继承ListableBeanFactory接口，所以应用上下文就具备了BeanFactory接口中获取Bean的方法，抽象接口ConfigurableApplicationContext继承ResourcesLoader接口就具备了加载本地xml配置的功能。

在这张图中还具体显示了对bean对象扩展的接口，定义了BeanFactoryProcessor和BeanPostProcessor，分别用来修改bean的定义和在实例化Bean对象的前后执行。另外也可以看一下这两个扩展的接口是如何被嵌入Bean对象的生命周期中的，在抽象的AbstractApplicationCntext中有一个refresh()方法，表示用来刷新上下文信息，在这里面会重新创建bean对象，所以我们将扩展点再refresh()方法中嵌入。

从整个继承关系来看，提供给用户的接口ClassPathXmlApplicationContext，用户只需要提供配置文件的路径，然后应用上下文就可以通过刷新上下文的方式，将用户配置中的bean对象全部存储到容器中。

### 3.2、定义 BeanFactoryPostProcessor

```java
/**
 * @Description 定义修改BeanDefinition的接口
 * @Author bugcode.online
 * @Date 2024/5/17 6:51
 */
public interface BeanFactoryPostProcessor {

    /**
     * 在所有的 BeanDefinition 加载完成后，实例化 Bean 对象之前，提供修改 BeanDefinition 属性的机制
     *
     * @param beanFactory
     * @throws BeansException
     */
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

定义修改Bean定义的接口BeanFactoryPostProcessor,用户只需要实现该接口，然后spring容器就可以感知到实现，在bean定义被加载后默认触发postProcessBeanFactory方法的执行。

### 3.3、定义BeanPostProcessor

```java
/**
 * @Description bean的前置和后置处理器接口 在Bean对象初始化前后执行
 * @Author bugcode.online
 * @Date 2024/5/17 6:52
 */
public interface BeanPostProcessor {

    /**
     * 在 Bean 对象执行初始化方法之前，执行此方法
     *
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

    /**
     * 在 Bean 对象执行初始化方法之后，执行此方法
     *
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

定义bean的前置和后置处理器方法接口BeanPostProcessor，里面包括两个方法，分别是前置和后置处理器：
- postProcessBeforeInitialization 用于在Bean对象执行初始化方法之前，执行此方法；
- postProcessAfterInitialization用于在Bean对象执行初始化方法之后，执行此方法。

> 注意：是在bean生命周期的初始化前后执行此方法，而非实例化前后;

### 3.4、上下文接口

```java
/**
 * @Description application的顶级接口
 * @Author bugcode.online
 * @Date 2024/5/17 6:44
 */
public interface ApplicationContext extends ListableBeanFactory {
}
```

ApplicationContext，继承于 ListableBeanFactory，也就继承了关于 BeanFactory 方法，比如一些 getBean 的方法。另外 ApplicationContext 本身是 Central 接口，但目前还不需要添加一些获取ID和父类上下文，所以暂时没有接口方法的定义。

**ConfigurableApplicationContext**

```java
/**
 * @Description ConfigurableApplicationContext扩展ApplicationContext接口，新增自己的方法
 * @Author bugcode.online
 * @Date 2024/5/17 6:45
 */
public interface ConfigurableApplicationContext extends ApplicationContext{

    /**
     * 刷新容器
     *
     * @throws BeansException
     */
    void refresh() throws BeansException;

    /**
     * 注册虚拟机钩子的方法 registerShutdownHook
     */
    void registerShutdownHook();

    /**
     * 手动执行关闭的方法 close
     */
    void close();

}
```

ConfigurableApplicationContext接口集成ApplicationContext,提供了一个核心的方法refresh()方法用来刷新上下文。

至于为什么要接口继承接口，目的其实为了扩展接口的功能，比如在接口职责的范围内，新增方法，这样实现原始接口的类就不需要修改类，最小的影响了修改接口的影响范围。

### 3.5、应用上下文抽象类实现

```java
/**
 * @Description 实现上下文接口的抽象类定义
 * @Author bugcode.online
 * @Date 2024/5/17 6:47
 */
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {

    /**
     * ConfigurableApplicationContext刷新上下文方法，核心方法
     * @throws BeansException
     */
    @Override
    public void refresh() throws BeansException {
        // 1. 创建 BeanFactory，并加载 BeanDefinition
        refreshBeanFactory();

        // 2. 获取 BeanFactory,这一步已经将bean的定义添加到了容器中
        ConfigurableListableBeanFactory beanFactory = getBeanFactory();

        // 3. 在 Bean 实例化（创建）之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
        /*在这里可以修改Bean的定义信息*/
        invokeBeanFactoryPostProcessors(beanFactory);

        // 4. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
        registerBeanPostProcessors(beanFactory);

        // 5. 提前实例化单例Bean对象
        beanFactory.preInstantiateSingletons();
    }

    /**
     * 抽象类自己的方法
     * 创建BeanFactory的类，由子抽象类进行实现
     * @throws BeansException
     */
    protected abstract void refreshBeanFactory() throws BeansException;

    /**
     * 获取BeanFactory的方法，由子类自己实现
     * @return
     */
    protected abstract ConfigurableListableBeanFactory getBeanFactory();


    /**
     *在所有BeanDefinition加载完成后，实例化Bean对象之前执行，可以修改Bean的定义
     * @param beanFactory
     */
    private void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        Map<String, BeanFactoryPostProcessor> beanFactoryPostProcessorMap = beanFactory.getBeansOfType(BeanFactoryPostProcessor.class);
        for (BeanFactoryPostProcessor beanFactoryPostProcessor : beanFactoryPostProcessorMap.values()) {
            beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
        }
    }

    /**
     * 注册bean对象的前置和后置处理器
     * @param beanFactory
     */
    private void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        Map<String, BeanPostProcessor> beanPostProcessorMap = beanFactory.getBeansOfType(BeanPostProcessor.class);
        for (BeanPostProcessor beanPostProcessor : beanPostProcessorMap.values()) {
            beanFactory.addBeanPostProcessor(beanPostProcessor);
        }
    }

    @Override
    public <T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException {
        return getBeanFactory().getBeansOfType(type);
    }

    @Override
    public String[] getBeanDefinitionNames() {
        return getBeanFactory().getBeanDefinitionNames();
    }

    @Override
    public Object getBean(String name) throws BeansException {
        return getBeanFactory().getBean(name);
    }

    @Override
    public Object getBean(String name, Object... args) throws BeansException {
        return getBeanFactory().getBean(name, args);
    }

    @Override
    public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
        return getBeanFactory().getBean(name, requiredType);
    }

}
```

AbstractApplicationContext 继承 DefaultResourceLoader 是为了处理 spring.xml 配置资源的加载。

之后是在 refresh() 定义实现过程，包括：

1. 创建 BeanFactory，并加载 BeanDefinition
2. 获取 BeanFactory
3. 在 Bean 实例化之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
4. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
5. 提前实例化单例Bean对象

另外把定义出来的抽象方法，refreshBeanFactory()、getBeanFactory() 由后面的继承此抽象类的其他抽象类实现。

一般接口都需要一个抽象类去实现，在抽象类中实现子类的一些公共方法，然后一些个性的方法让其子类去实现，另外在抽象类中还可以使用继承的方法去扩展抽象类的功能，在抽象类中还可以声明自己的抽象方法，让子类去实现。

> 抽象类的一个作用，定义调用过程，在refresh方法中首先创建BeanFactory对象，然后注册BeanFactoryPostProcessors，接着注册了BeanPostProcessors，最后执行对象的实例化，可以看待在这个方法中定义了整个bean对象的生命周期中各个阶段的调用过程。

### 3.6、获取Bean工厂和加载资源

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext{

    private DefaultListableBeanFactory beanFactory;

    /**
     * 子类实现父类 创建BeanFactory的方法
     * @throws BeansException
     */
    @Override
    protected void refreshBeanFactory() throws BeansException {

        DefaultListableBeanFactory beanFactory = createBeanFactory();
        loadBeanDefinitions(beanFactory);
        this.beanFactory = beanFactory;
    }

    /**
     * 创建BeanFsactory 子抽象类实现
     * @return
     */
    private DefaultListableBeanFactory createBeanFactory(){
        return new DefaultListableBeanFactory();
    }

    /**
     * 抽象类自己的方法 加载类定义
     * @param beanFactory
     */
    protected abstract void loadBeanDefinitions(DefaultListableBeanFactory beanFactory);


    @Override
    public DefaultListableBeanFactory getBeanFactory() {
        return beanFactory;
    }
}
```

- 在 refreshBeanFactory() 中主要是获取了 DefaultListableBeanFactory 的实例化以及对资源配置的加载操作 loadBeanDefinitions(beanFactory)，在加载完成后即可完成对 spring.xml 配置文件中 Bean 对象的定义和注册，同时也包括实现了接口 BeanFactoryPostProcessor、BeanPostProcessor 的配置 Bean 信息。
- 但此时资源加载还只是定义了一个抽象类方法 loadBeanDefinitions(DefaultListableBeanFactory beanFactory)，继续由其他抽象类继承实现。

### 3.7、上下文中对配置加载

```java
/**
 * @Description 上下文中对配置信息的加载
 * @Author bugcode.online
 * @Date 2024/5/17 7:14
 */
public abstract class AbstractXmlApplicationContext extends AbstractRefreshableApplicationContext{


    /**
     * 加载Bean的定义数据
     * @param beanFactory
     */
    @Override
    protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) {
//        加载Bean的定义
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory, this);
//        获取bean配置文件的位置
        String[] configLocations = getConfigLocations();
        if (null != configLocations){
//            读取bean的定义然后注册到bean定义的容器中
            beanDefinitionReader.loadBeanDefinitions(configLocations);
        }
    }

    /**
     * 抽象类自己的方法 由子类实现
     * @return
     */
    protected abstract String[] getConfigLocations();

}
```

- 在 AbstractXmlApplicationContext 抽象类的 loadBeanDefinitions 方法实现中，使用 XmlBeanDefinitionReader 类，处理了关于 XML 文件配置信息的操作。
- 同时这里又留下了一个抽象类方法，getConfigLocations()，此方法是为了从入口上下文类，拿到配置信息的地址描述。

### 3.8、应用上下文具体实现类

```java
/**
 * @Description ClassPathXmlApplicationContext 主要负责给外部用户提供方法
 * @Author bugcode.online
 * @Date 2024/5/17 7:17
 */
public class ClassPathXmlApplicationContext extends AbstractXmlApplicationContext {

//    配置文件路径地址
    private String []configLocations;

//    无参构造方法
    public ClassPathXmlApplicationContext(){};

    /**
     * 从 XML 中加载 BeanDefinition，并刷新上下文
     *
     * @param configLocations
     * @throws BeansException
     */
    public ClassPathXmlApplicationContext(String configLocations) throws BeansException {
        this(new String[]{configLocations});
    }

    /**
     * 从 XML 中加载 BeanDefinition，并刷新上下文
     * @param configLocations
     * @throws BeansException
     */
    public ClassPathXmlApplicationContext(String[] configLocations) throws BeansException {
        this.configLocations = configLocations;
//        刷新上下文
        refresh();
    }

    @Override
    protected String[] getConfigLocations() {
        return configLocations;
    }

    @Override
    public void registerShutdownHook() {

    }

    @Override
    public void close() {

    }
}
```

- ClassPathXmlApplicationContext，是具体对外给用户提供的应用上下文方法。
- 在继承了 AbstractXmlApplicationContext 以及层层抽象类的功能分离实现后，在此类 ClassPathXmlApplicationContext 的实现中就简单多了，主要是对继承抽象类中方法的调用和提供了配置文件地址信息。

继承关闭比较深，因此我们通过类图总体看一下类之间的关系：

![ApplicationContext](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/ApplicationContext.png)

AbstractRefreshableApplicationContext抽象类继承AbstractApplicationContext抽象类

抽象类主要完成beanFactory的创建动作，主要是实现AbstractApplicationContext抽象类中定义的自己的方法refreshBeanFactory()，getBeanFactory()，在这里也可以发现套路，抽象类中定义的方法，最多都是一级实现，也就是在其子类中就实现了，不会两级实现，比如在其子类的子类中实现。

然后次抽象类中扩展了自己的方法loadBeanDefinitions(),主要完成加载Bean定义的动作。



抽象类AbstractXmlApplicationContext继承AbstractRefreshableApplicationContext：

主要完成具体的加载Bean定义的动作，然后扩展getConfigLocations()方法由其子类实现。



ClassPathXmlApplicationContext具体类继承AbstractXmlApplicationContext抽象类：

这个类是具体留给用户的交互类，用户可以创建ClassPathXmlApplicationContext(String[] configLocations)对象，然后xml上下文对象会直接进行上下文刷新动作，并且实现了其父类的getConfigLocations()方法。



分解了各个类的职责，总结一下过程：

ApplicationContext：获取容器bean对象

ConfigurableApplicationContext：刷新容器

AbstractApplicationContext：具体容器刷新，创建BeanFactory，新增获取并且加载资源能力。

AbstractRefreshableApplicationContext:创建BeanFactory，加载BeanDefinition

AbstractXmlApplicationContext：加载BeanDefinition，获取xml配置文件路径

ClassPathXmlApplicationContext：根据用户提供的xml路径，完成容器上下文的刷新

根据用户xml路径读取配置--》解析xml然后加载Bean定义--》注册Bean定义，完成前置和后置处理器--》刷新容器动作

通常我们的思考是顺序进行，然后类的职责是逆序分配的，比如ClassPathXmlApplicationContext中需要根据xml位置加载解析xml，那么将这个加载的动作放在其父类AbstractXmlApplicationContext中，有一个向上委托的动作。

## 4、测试

### 4.1、测试用例

DAO对象

```java
public class PeopleDao {

    private static Map<String,String> map = new HashMap<>();

    static {
        map.put("10001", "工程师");
        map.put("10002", "秘书");
        map.put("10003", "实习生");
    }

    public String queryUserName(String uId) {
        return map.get(uId);
    }
}
```

Service对象

```java
public class PeopleService {
    private String id;
    private String company;
    private String location;
    private PeopleDao peopleDao;

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + peopleDao.queryUserName(id)+"  公司名称："+company+"  公司地点："+location);
    }
}
```

将DAO对象注入到Service对象中

实现修改Bean定义的类

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    /**
     * 加载完成Bean的定义，在Bean的实例化之前执行
     * @param beanFactory
     * @throws BeansException
     */
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

//        获取bean的定义
        BeanDefinition peopleService = beanFactory.getBeanDefinition("peopleService");

//        获取bean的属性
        PropertyValues propertyValues = peopleService.getPropertyValues();
        propertyValues.addPropertyValue(new PropertyValue("company","修改为字节跳动"));
//        propertyValues.addPropertyValue(new PropertyValue("location","修改为上海"));
    }
}
```

实现前置和后置处理

```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if("peopleService".equals(beanName)){
            PeopleService peopleService = (PeopleService)bean;
            peopleService.setLocation("上海");
        }

        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

xml配置文件

xml配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans>

    <bean id="peopleDao" class="bugcode.online.springframework.bean.PeopleDao"/>

    <bean id="peopleService" class="bugcode.online.springframework.bean.PeopleService">
        <property name="id" value="10001"/>
        <property name="company" value="腾讯"/>
        <property name="location" value="深圳"/>
        <property name="peopleDao" ref="peopleDao"/>
    </bean>

    <bean class="bugcode.online.springframework.common.MyBeanPostProcessor"/>
    <bean class="bugcode.online.springframework.common.MyBeanFactoryPostProcessor"/>

</beans>
```

### 4.2、不使用上下文

```java
@Test
    public void testContext_processFactory(){

//        1 初始化BeanFactory
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();

//        2 读取配置文件 注册bean对象
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        reader.loadBeanDefinitions("classpath:spring.xml");

        // 3. BeanDefinition 加载完成 & Bean实例化之前，修改 BeanDefinition 的属性值
        MyBeanFactoryPostProcessor beanFactoryPostProcessor = new MyBeanFactoryPostProcessor();
        beanFactoryPostProcessor.postProcessBeanFactory(factory);
//----------------------bean实例化前，直接修改bean的定义信息

//        bean的前后置处理器方法
        MyBeanPostProcessor myBeanPostProcessor = new MyBeanPostProcessor();
        factory.addBeanPostProcessor(myBeanPostProcessor);

//        4 获取bean对象的调用方法
        PeopleService peopleService = (PeopleService)factory.getBean("peopleService");
        peopleService.queryUserInfo();
    }
```

- DefaultListableBeanFactory 创建 beanFactory 并使用 XmlBeanDefinitionReader 加载配置文件的方式，还是比较熟悉的。
- 接下来就是对 MyBeanFactoryPostProcessor 和 MyBeanPostProcessor 的处理，一个是在BeanDefinition 加载完成 & Bean实例化之前，修改 BeanDefinition 的属性值，另外一个是在Bean初始化之后，修改 Bean 属性信息。

### 4.3、使用应用上下文

```java
    @Test
    public void text_Context(){
        // 1.初始化 BeanFactory
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");

        // 2. 获取Bean对象调用方法
        PeopleService peopleService = applicationContext.getBean("peopleService", PeopleService.class);
        peopleService.queryUserInfo();

    }
```

**结果**

```java
查询用户信息：工程师  公司名称：修改为字节跳动  公司地点：上海

Process finished with exit code 0
```

对比两种测试方法，可以看到，通过应用上下文的封装，最终对用户提供的只有一个接口，用户只需要提供配置路径，然后容器会自动加载并且刷新容器中的所有bean对象，接着用户只需要根据id去容器中获取对象即可。

用户只需要简单实现修改bean的接口，然后spring容器会自动感知到并且在spring生命周期的阶段中执行实现的方法。

## 5、小结

第五章节，要理解spring上下文这个功能的设计和实现，其中有接口继承接口，抽象类实现接口，每一层都有自己的功能。