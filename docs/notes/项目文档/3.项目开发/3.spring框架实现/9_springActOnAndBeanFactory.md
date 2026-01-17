---
title: 9、对象作用域和BeanFactory
icon: file
order: 9
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
permalink: /project-docs/project/springframework/对象作用域和BeanFactory/
---

<!-- TOC -->

- [9、对象作用域和BeanFactory](#9对象作用域和beanfactory)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类图](#31核心类图)
    - [3.2、Bean的作用范围定义和解析](#32bean的作用范围定义和解析)
    - [3.3、创建和修改对象时判断单例还是原型](#33创建和修改对象时判断单例还是原型)
    - [3.4、定义FactoryBean接口](#34定义factorybean接口)
    - [3.5、实现FactoryBean的注册服务](#35实现factorybean的注册服务)
    - [3.6、扩展 AbstractBeanFactory 创建对象逻辑](#36扩展-abstractbeanfactory-创建对象逻辑)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、单元测试](#42单元测试)
  - [5、小结](#5小结)

<!-- /TOC -->


# 9、对象作用域和BeanFactory

## 1、目标

交给 Spring 管理的 Bean 对象，一定就是我们用类创建出来的 Bean 吗？创建出来的 Bean 就永远是单例的吗，没有可能是原型模式吗？

在集成Spring 框架下，我们使用的 MyBatis 框架中，它的核心作用是可以满足用户不需要实现 Dao 接口类，就可以通过 xml 或者注解配置的方式完成对数据库执行 CRUD 操作，那么在实现这样的 ORM 框架中，是怎么把一个数据库操作的 Bean 对象交给 Spring 管理的呢。

因为我们在使用 Spring、MyBatis 框架的时候都可以知道，并没有手动的去创建任何操作数据库的 Bean 对象，有的仅仅是一个接口定义，而这个接口定义竟然可以被注入到其他需要使用 Dao 的属性中去了，**那么这一过程最核心待解决的问题，就是需要完成把复杂且以代理方式动态变化的对象，注册到 Spring 容器中**。而为了满足这样的一个扩展组件开发的需求，就需要我们在现有手写的 Spring 框架中，添加这一能力。



## 2、设计

关于提供一个能让使用者定义复杂的 Bean 对象，功能点非常不错，意义也非常大，因为这样做了之后 Spring 的生态种子孵化箱就此提供了，谁家的框架都可以在此标准上完成自己服务的接入。

但这样的功能逻辑设计上并不复杂，因为整个 Spring 框架在开发的过程中就已经提供了各项扩展能力的接茬，你只需要在合适的位置提供一个接茬的处理接口调用和相应的功能逻辑实现即可，像这里的目标实现就是对外提供一个可以二次从 FactoryBean 的 getObject 方法中获取对象的功能即可，这样所有实现此接口的对象类，就可以扩充自己的对象功能了。_MyBatis 就是实现了一个 MapperFactoryBean 类，在 getObject 方法中提供 SqlSession 对执行 CRUD 方法的操作_ 整体设计结构如下图：





![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1717297445618-420d93f4-1211-4c0f-adbd-69fcda4dd78e.png)

+ 整个的实现过程包括了两部分，一个解决单例还是原型对象，另外一个处理 FactoryBean 类型对象创建过程中关于获取具体调用对象的 getObject 操作。
+ SCOPE_SINGLETON、SCOPE_PROTOTYPE，对象类型的创建获取方式，主要区分在于 AbstractAutowireCapableBeanFactory#createBean 创建完成对象后是否放入到内存中，如果不放入则每次获取都会重新创建。
+ createBean 执行对象创建、属性填充、依赖加载、前置后置处理、初始化等操作后，就要开始做执行判断整个对象是否是一个 FactoryBean 对象，如果是这样的对象，就需要再继续执行获取 FactoryBean 具体对象中的 getObject 对象了。整个 getBean 过程中都会新增一个单例类型的判断factory.isSingleton()，用于决定是否使用内存存放对象信息。

## 3、实现

### 3.1、核心类图


![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1717291401751-83b96101-56b5-47ff-baaa-632605e42bd7.png)



### 3.2、Bean的作用范围定义和解析

```java
public class BeanDefinition {

//    新增bean是单例还是原型模式
    String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
    String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

//    定义类信息
    private Class beanClass;

    private PropertyValues propertyValues;

    private String initMethodName;

    private String destroyMethodName;

    private boolean singleton = true;

    private boolean prototype = false;

    private String scope = SCOPE_SINGLETON;
}
```

 singleton、prototype，是本次在 BeanDefinition 类中新增加的两个属性信息，用于把从 spring.xml 中解析到的 Bean 对象作用范围填充到属性中。  

解析xml中的作用范围：

```java
    /**
     *公共的加载xml文件的方法
     * @param inputStream
     * @throws ClassNotFoundException
     */
    protected void doLoadBeanDefinitions(InputStream inputStream) throws ClassNotFoundException {

        for (int i = 0; i < childNodes.getLength(); i++) {
            // 判断元素
            if (!(childNodes.item(i) instanceof Element)) continue;
            // 判断对象
            if (!"bean".equals(childNodes.item(i).getNodeName())) continue;

            // 解析标签
           
            String beanscope = bean.getAttribute("scope");
            
//            设置bean的作用范围
            if(StrUtil.isNotEmpty(beanscope)){
                beanDefinition.setScope(beanscope);
            }

            // 注册 BeanDefinition
            getRegistry().registerBeanDefinition(beanName, beanDefinition);
        }
    }
```

在解析xml配置的时候，将用户定义的scope参数解析出来然后放到BeanDefinition中，最后注入到容器中。



### 3.3、创建和修改对象时判断单例还是原型

```java
/**
 * 继承抽象的工厂类，实现自己的方法，公共的方法由抽象类实现
 * 体现了类实现过程中的各司其职，你只需要关心属于你的内容，不是你的内容，不要参与
 */
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {

//    实例化bean对象带参数
    private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

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

//        判断单例模式还是原型模式
        if(beanDefinition.isSingleton()){
//            如果是单例模式，就不在创建，将单例对象添加到内存中
            addSingleton(beanName,bean);
        }
        return bean;
    }

    protected void registerDisposableBeanIfNecessary(String beanName, Object bean, BeanDefinition beanDefinition) {
        // 非 Singleton 类型的 Bean 不执行销毁方法
        if (!beanDefinition.isSingleton()) return;

        if (bean instanceof DisposableBean || StrUtil.isNotEmpty(beanDefinition.getDestroyMethodName())) {
            registerDisposableBean(beanName, new DisposableBeanAdapter(bean, beanName, beanDefinition));
        }
    }
}
```

- **单例模式和原型模式的区别就在于是否存放到内存中，如果是原型模式那么就不会存放到内存中，每次获取都重新创建对象，另外非 Singleton 类型的 Bean 不需要执行销毁方法**。
- 所以这里的代码会有两处修改，一处是 createBean 中判断是否添加到 addSingleton(beanName, bean);，另外一处是 registerDisposableBeanIfNecessary 销毁注册中的判断 if (!beanDefinition.isSingleton()) return;

### 3.4、定义FactoryBean接口

```java
/**
 * @Description
 * @Author bugcode.online
 * @Date 2024/5/18 9:10
 */
public interface FactoryBean<T> {

    //返回的对象实例,我们可以在这里返回我们需要实例化的Bean
    T getObject() throws Exception;

    //Bean的类型,针对上面返回的实例，我们指定实例所对应的类型
    Class<?> getObjectType();

    //true是单例，false是非单例  在Spring5.0中此方法利用了JDK1.8的新特性变成了default方法，返回true
    boolean isSingleton();

}
```

 FactoryBean 中需要提供3个方法，获取对象、对象类型，以及是否是单例对象，如果是单例对象依然会被放到内存中。  



### 3.5、实现FactoryBean的注册服务

```java
public abstract class FactoryBeanRegistrySupport extends DefaultSingletonBeanRegistry{

    /**
     * Cache of singleton objects created by FactoryBeans: FactoryBean name --> object
     */
    private final Map<String, Object> factoryBeanObjectCache = new ConcurrentHashMap<String, Object>();

    /**
     * 从缓存中获取实例对象
     * @param beanName
     * @return
     */
    protected Object getCachedObjectForFactoryBean(String beanName) {
        Object object = this.factoryBeanObjectCache.get(beanName);
        return (object != NULL_OBJECT ? object : null);
    }

    protected Object getObjectFromFactoryBean(FactoryBean factoryBean,String beanName){

        if(factoryBean.isSingleton()){
            Object o = this.factoryBeanObjectCache.get(beanName);
            if(o == null){
//                从FactoryBean中创建bean对象
                o = doGetObjectFromFactoryBean(factoryBean, beanName);
                this.factoryBeanObjectCache.put(beanName,(o != null?o:NULL_OBJECT));
            }
            return o != null?o:NULL_OBJECT;
        }else {
            return doGetObjectFromFactoryBean(factoryBean, beanName);
        }
    }

    private Object doGetObjectFromFactoryBean(final FactoryBean factoryBean,final String beanName){
        try {
            return factoryBean.getObject();
        }catch (Exception e){
            throw new BeansException("FactoryBean threw exception on object[" + beanName + "] creation", e);
        }
    }
}
```

+ FactoryBeanRegistrySupport 类主要处理的就是关于 FactoryBean 此类对象的注册操作，之所以放到这样一个单独的类里，就是希望做到不同领域模块下只负责各自需要完成的功能，避免因为扩展导致类膨胀到难以维护。
+ 同样这里也定义了缓存操作 factoryBeanObjectCache，用于存放单例类型的对象，避免重复创建。_在日常使用用，基本也都是创建的单例对象_
+ doGetObjectFromFactoryBean 是具体的获取 FactoryBean#getObject() 方法，因为既有缓存的处理也有对象的获取，所以额外提供了 getObjectFromFactoryBean 进行逻辑包装，这部分的操作方式是不和你日常做的业务逻辑开发非常相似。_从Redis取数据，如果为空就从数据库获取并写入Redis_



### 3.6、扩展 AbstractBeanFactory 创建对象逻辑

```java
/**
 * BeanDefinition 注册表接口
 * AbstractBeanFactory首先类实现BeanFactory接口然后继承DefaultSingletonBeanRegistry抽象类，所以具备单例Bean的注册功能
 */
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {


    /** ClassLoader to resolve bean class names with, if necessary */
    private ClassLoader beanClassLoader = ClassUtils.getDefaultClassLoader();

    /** BeanPostProcessors to apply in createBean */
    private final List<BeanPostProcessor> beanPostProcessors = new ArrayList<BeanPostProcessor>();


    /**
     * 实现BeanFactory工厂接口中的方法 使用模板模式
     * @param name
     * @return
     * @throws BeansException
     */
    @Override
    public Object getBean(String name) throws BeansException {
        return doGetBean(name,null);
    }

    @Override
    public Object getBean(String name, Object... args) throws BeansException {
        return doGetBean(name,args);
    }

    @Override
    public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
        return (T)getBean(name);
    }

    /**
     * 创建对象
     * @param name
     * @param args
     * @param <T>
     * @return
     */
    protected <T> T doGetBean(final String name, final Object args[]){

//        从单例容器中获取单例对象
        Object sharedInstance = getSingleton(name);
        if(sharedInstance != null){
            // 如果是 FactoryBean，则需要调用 FactoryBean#getObject
            return (T) getObjectForBeanInstance(sharedInstance, name);
        }

//        获取bean的定义
        BeanDefinition beanDefinition = getBeanDefinition(name);
        Object bean = createBean(name, beanDefinition, args);
        return (T)getObjectForBeanInstance(bean,name);
    }

    /**
     * 从缓存中获取
     * @param beanInstance
     * @param beanName
     * @return
     */
    private Object getObjectForBeanInstance(Object beanInstance,String beanName){
        if(!(beanInstance instanceof FactoryBean)){
            return beanInstance;
        }
//        从缓存中获取
        Object object = getCachedObjectForFactoryBean(beanName);
        if(object == null){
            FactoryBean<?> factoryBean = (FactoryBean<?>) beanInstance;
            object = getObjectFromFactoryBean(factoryBean, beanName);
        }
        return object;
    }

    protected abstract BeanDefinition getBeanDefinition(String beanName) throws BeansException;

    protected abstract Object createBean(String beanName, BeanDefinition beanDefinition) throws BeansException;

    protected abstract Object createBean(String beanName, BeanDefinition beanDefinition,Object []args) throws BeansException;


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
     * Return the list of BeanPostProcessors that will get applied
     * to beans created with this factory.
     */
    public List<BeanPostProcessor> getBeanPostProcessors() {
        return this.beanPostProcessors;
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

+ 首先这里把 AbstractBeanFactory 原来继承的 DefaultSingletonBeanRegistry，修改为继承 FactoryBeanRegistrySupport。因为需要扩展出创建 FactoryBean 对象的能力，所以这就想一个链条服务上，截出一个段来处理额外的服务，并把链条再链接上。
+ 此处新增加的功能主要是在 doGetBean 方法中，添加了调用 (T) getObjectForBeanInstance(sharedInstance, name) 对获取 FactoryBean 的操作。
+ 在 getObjectForBeanInstance 方法中做具体的 instanceof 判断，另外还会从 FactoryBean 的缓存中获取对象，如果不存在则调用 FactoryBeanRegistrySupport#getObjectFromFactoryBean，执行具体的操作。



## 4、测试

### 4.1、测试用例

定义DAO接口：

```java
public interface IPeopleDao {

    String queryUserName(String uId);
}
```



定义Service

```java
public class PeopleService{
    private String id;
    private String company;
    private String location;
    private IPeopleDao peopleDao;

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + peopleDao.queryUserName(id)+"  公司名称："+company+"  公司地点："+location);
    }
}
```

 在 UserService 新修改了一个原有 UserDao 属性为 IUserDao，后面我们会给这个属性注入代理对象  



定义FactoryBean对象：

```java
/**
 * @Description 实现FactoryBean工厂接口的具体工厂类
 * @Author bugcode.online
 * @Date 2024/5/18 9:36
 */
public class ProxyBeanFactory implements FactoryBean<IPeopleDao> {
    @Override
    public IPeopleDao getObject() throws Exception {
        InvocationHandler handler = (proxy, method, args) -> {

            // 添加排除方法
            if ("toString".equals(method.getName())) return this.toString();

            Map<String, String> hashMap = new HashMap<>();
            hashMap.put("10001", "工程师");
            hashMap.put("10002", "秘书");
            hashMap.put("10003", "实习生");

            return "你被代理了 " + method.getName() + "：" + hashMap.get(args[0].toString());
        };
        return (IPeopleDao) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class[]{IPeopleDao.class}, handler);
    }

    @Override
    public Class<?> getObjectType() {
        return IPeopleDao.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

+ 这是一个实现接口 FactoryBean 的代理类 ProxyBeanFactory 名称，主要是模拟了 UserDao 的原有功能，类似于 MyBatis 框架中的代理操作。
+ getObject() 中提供的就是一个 InvocationHandler 的代理对象，当有方法调用的时候，则执行代理对象的功能。



配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans>


    <bean id="peopleService" class="bugcode.online.springframework.bean.PeopleService" scope="prototype">
        <property name="id" value="10001"/>
        <property name="company" value="腾讯"/>
        <property name="location" value="上海"/>
        <property name="peopleDao" ref="proxyPeopleDao"/>
    </bean>

    <bean id="proxyPeopleDao"  class="bugcode.online.springframework.bean.ProxyBeanFactory"/>

</beans>
```

### 4.2、单元测试

```java
@Test
    public void test_prototype() {
        // 1.初始化 BeanFactory
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        applicationContext.registerShutdownHook();

        // 2. 获取Bean对象调用方法
        PeopleService peopleService01 = applicationContext.getBean("peopleService", PeopleService.class);
        PeopleService peopleService02 = applicationContext.getBean("peopleService", PeopleService.class);

        // 3. 配置 scope="prototype/singleton"
        System.out.println(peopleService01);
        System.out.println(peopleService02);

        // 4. 打印十六进制哈希
        System.out.println(peopleService01 + " 十六进制哈希：" + Integer.toHexString(peopleService01.hashCode()));
        System.out.println(ClassLayout.parseInstance(peopleService01).toPrintable());
    }

```



使用上下文单侧

```java
    @Test
    public void test_factory_bean() {
        // 1.初始化 BeanFactory
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        applicationContext.registerShutdownHook();
        // 2. 调用代理方法
        PeopleService peopleService = applicationContext.getBean("peopleService", PeopleService.class);
        peopleService.queryUserInfo();
    }
```



测试结果：

```java
查询用户信息：你被代理了 queryUserName：工程师  公司名称：腾讯  公司地点：上海

Process finished with exit code 0
```



## 5、小结

实现原型模式的思路：

```java
//刷新容器
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

//刷新容器时创建bean对象
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

//        判断单例模式还是原型模式
        if(beanDefinition.isSingleton()){
//            如果是单例模式，就不在创建
            addSingleton(beanName,bean);
        }
        return bean;
    }
```

核心思路：

在刷新容器时会创建Bean对象，spring会根据用户配置的scope参数，判断是原型还是单例模式，如果是单例模式，那么会将对象创建出来添加到单例容器中。

如果非单例模式，不会将对象添加到单例容器中。在获取原型模式的对象时，每次都会创建一个新的对象返回。

```java
protected <T> T doGetBean(final String name, final Object args[]){

//        从单例容器中获取单例对象
        Object sharedInstance = getSingleton(name);
        if(sharedInstance != null){
            // 如果是 FactoryBean，则需要调用 FactoryBean#getObject
            return (T) getObjectForBeanInstance(sharedInstance, name);
        }

//        获取bean的定义
        BeanDefinition beanDefinition = getBeanDefinition(name);
        Object bean = createBean(name, beanDefinition, args);
        return (T)getObjectForBeanInstance(bean,name);
    }
```
