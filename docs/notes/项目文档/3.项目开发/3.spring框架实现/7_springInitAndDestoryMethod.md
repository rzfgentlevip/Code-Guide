---
title: 7、Bean对象的初始化和销毁
icon: file
order: 7
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
permalink: /project-docs/project/springframework/Bean对象的初始化和销毁/
---

<!-- TOC -->

- [7、Bean对象的初始化和销毁方法](#7bean对象的初始化和销毁方法)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类图实现](#31核心类图实现)
    - [3.2、定义初始化和销毁方法接口](#32定义初始化和销毁方法接口)
    - [3.3、Bean定义增加初始化和销毁属性](#33bean定义增加初始化和销毁属性)
    - [3.4、执行Bean对象初始化和销毁](#34执行bean对象初始化和销毁)
    - [3.5、定义销毁方法适配器(接口和配置)](#35定义销毁方法适配器接口和配置)
    - [3.6、虚拟机关闭钩子注册调用销毁方法](#36虚拟机关闭钩子注册调用销毁方法)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、单元测试](#42单元测试)
  - [5、小结](#5小结)

<!-- /TOC -->


# 7、Bean对象的初始化和销毁方法

## 1、目标

上一章中，我们对Bean对象声明周期中扩展了两个点:
- 一个是当Bean定义加载完成后，提供了BeanFactoryPostProcessor接口修改Bean定义的接口
- 另一个是BeanPostProcessor接口，提供在初始化Bean的前后对Bean对象进行修改的前置和后置处理器。

思考一下这两个bean的扩展都是在Bean对象的实例化前和实例化后，那如果我们想在Bean对象的初始化过程中，对Bean对象进行一些扩展怎么办，比如想做资源的加载， 链接注册中心暴露RPC接口以及在Web程序关闭时执行链接断开，内存销毁等操作。*如果说没有Spring我们也可以通过构造函数、静态方法以及手动调用的方式实现，但这样的处理方式终究不如把诸如此类的操作都交给 Spring 容器来管理更加合适。*

因此，在Bean对象的初始化过程中，我们还可以定义初始化和销毁方法，spring源码中定义了两个接口，InitializingBean和DisposableBean用来执行初始化和销毁的动作。

## 2、设计

在使用spring框架的过程中，我们只需要再**xml配置文件**中做简单的配置或者在代码中**使用注解**，或者是实现一些**预留的接口**，spring就可以帮我们将配置中定义的bean对象加载到容器中并且执行自定义实现的接口方法，其实对于这种Bean容器初始化过程中额外添加的处理操作，在spring内部无非就是预先执行了一个定义好的接口方法或者反射调用类中xml配置的方法而已，程序中只需要按照接口的定义实现，spring容器在处理的过程中会自己进行调用而已。

在spring框架的实现中，用户可以通过在xml配置文件中指定初始化和销毁的方法，或者使用注解指定这两个方法，因此本章中，我们先实现在xml配置文件中指定初始化和销毁方法，然后让spring自动帮我们将这两个方法串联在初始化过程中。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408021037313.png)

- 在 spring.xml 配置中添加 init-method、destroy-method 两个注解，在配置文件加载的过程中，把注解配置一并定义到 BeanDefinition 的属性当中。这样在 initializeBean 初始化操作的过程中，就可以通过反射的方式来调用配置在 Bean 定义属性当中的方法信息了。另外如果是接口实现的方式，那么直接可以通过 Bean 对象调用对应接口定义的方法即可，((InitializingBean) bean).afterPropertiesSet()，两种方式达到的效果是一样的。
  
- 除了初始化做的操作外，destroy-method 和 DisposableBean 接口的定义，都会在 Bean 对象初始化完成阶段，执行注册销毁方法的信息到 DefaultSingletonBeanRegistry 类中的 disposableBeans 属性里，这是为了后续统一进行操作。*这里还有一段适配器的使用，因为反射调用和接口直接调用，是两种方式。所以需要使用适配器进行包装，下文代码讲解中参考 DisposableBeanAdapter 的具体实现* -关于销毁方法需要在虚拟机执行关闭之前进行操作，所以这里需要用到一个注册钩子的操作，如：Runtime.getRuntime().addShutdownHook(new Thread(() -> System.out.println("close！")));*这段代码你可以执行测试*，另外你可以使用手动调用 ApplicationContext.close 方法关闭容器。

## 3、实现

### 3.1、核心类图实现

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408021039145.png)

- 因为我们一共实现了两种方式的初始化和销毁方法，**xml配置和定义接口**，所以这里既有 InitializingBean、DisposableBean 也有需要 XmlBeanDefinitionReader 加载 spring.xml 配置信息到 BeanDefinition 中，都是设计的顶层接口，负责bean对象的初始化和销毁，这里也可以看出，类的功能高度单一。
- 另外接口 ConfigurableBeanFactory 定义了 destroySingletons 销毁方法，并由 AbstractBeanFactory 继承的父类 DefaultSingletonBeanRegistry 实现 ConfigurableBeanFactory 接口定义的 destroySingletons 方法。*这种方式的设计可能数程序员是没有用过的，都是用的谁实现接口谁完成实现类，而不是把实现接口的操作又交给继承的父类处理。所以这块还是蛮有意思的，是一种不错的隔离分层服务的设计方式*
- 最后就是关于向虚拟机注册钩子，保证在虚拟机关闭之前，执行销毁操作。Runtime.getRuntime().addShutdownHook(new Thread(() -> System.out.println("close！")));
- DisposableBean接口实现了一个适配器，*因为反射调用和接口直接调用，是两种方式。所以需要使用适配器进行包装，销毁方法的调用，一种是接口引用调用，另一种是反射调用。初始化只有接口调用。*

### 3.2、定义初始化和销毁方法接口

**初始化接口**

```java
/**
 * @Description 定义Bean对象初始化前的初始化接口
 * @Author bugcode.online
 * @Date 2024/5/17 14:29
 */
public interface InitializingBean {
    /**
     * Bean 处理了属性填充后调用
     *
     * @throws Exception
     */
    void afterPropertiesSet() throws Exception;
}
```

**销毁接口**

```java
public interface DisposableBean {

//    执行销毁方法
    void destroy() throws Exception;

}
```

初始化和销毁接口非常简单，**定义了执行初始化和销毁动作执行的方法**，用户只需要实现这两个接口，spring就会自动感知到方法然后自动在初始化和销毁的时候调用这两个方法，比如在初始化的时候做接口暴漏、数据库数据读取、配置文件加载等等 。

### 3.3、Bean定义增加初始化和销毁属性

思考一个问题，Bean的初始化和销毁方法通过什么方法传递执行

因为用户配置初始化和销毁方法一般是在xml文件中，所以需要再解析xml配置文件的时候识别到用户配置的方法关键字，所以就需要再BeanDefinition中将初始化和销毁方法作为属性添加进来，在解析的时候将方法注册到BeanDefinition中。

```java
@Data
@SuppressWarnings({"rawtypes"})
public class BeanDefinition {

//    定义类信息
    private Class beanClass;

//    定义Bean对象的属性集合信息
    private PropertyValues propertyValues;
    
    /*定义初始化方法的名称 保存方法名字*/
    private String initMethodName;

    /*定义销毁方法的名称 保存方法名字*/
    private String destroyMethodName;

    public BeanDefinition(Class beanClass) {
        this.beanClass = beanClass;
        this.propertyValues = new PropertyValues();
    }

    public BeanDefinition(Class beanClass, PropertyValues propertyValues) {
        this.beanClass = beanClass;
        this.propertyValues = propertyValues != null?propertyValues:null;
    }

    public Class getBeanClass() {
        return beanClass;
    }

    public void setBeanClass(Class beanClass) {
        this.beanClass = beanClass;
    }

    public PropertyValues getPropertyValues() {
        return propertyValues;
    }

    public void setPropertyValues(PropertyValues propertyValues) {
        this.propertyValues = propertyValues;
    }
}
```

在 BeanDefinition 新增加了两个属性：initMethodName、destroyMethodName，这两个属性是为了在 spring.xml 配置的 Bean 对象中，可以配置 init-method="initDataMethod" destroy-method="destroyDataMethod" 操作，最终实现接口的效果是一样的。*只不过一个是接口方法的直接调用，另外是一个在配置文件中读取到方法反射调用。*

*XmlBeanDefinitionReader类中新增初始化和销毁方法的解析：*

```java
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {

    /**
     *公共的加载xml文件的方法
     * @param inputStream
     * @throws ClassNotFoundException
     */
    protected void doLoadBeanDefinitions(InputStream inputStream) throws ClassNotFoundException {
        Document doc = XmlUtil.readXML(inputStream);
        Element root = doc.getDocumentElement();
        NodeList childNodes = root.getChildNodes();

        for (int i = 0; i < childNodes.getLength(); i++) {
            // 判断元素
            if (!(childNodes.item(i) instanceof Element)) continue;
            // 判断对象
            if (!"bean".equals(childNodes.item(i).getNodeName())) continue;

            // 解析标签
            Element bean = (Element) childNodes.item(i);
            String id = bean.getAttribute("id");
            String name = bean.getAttribute("name");
            String className = bean.getAttribute("class");
            //解析初始化和销毁的方法
            String initMethod = bean.getAttribute("init-method");
            String destroyMethodName = bean.getAttribute("destroy-method");

            // 获取 Class，方便获取类中的名称
            Class<?> clazz = Class.forName(className);
            // 优先级 id > name
            String beanName = StrUtil.isNotEmpty(id) ? id : name;
            if (StrUtil.isEmpty(beanName)) {
                beanName = StrUtil.lowerFirst(clazz.getSimpleName());
            }

            // 定义Bean
            BeanDefinition beanDefinition = new BeanDefinition(clazz);
            //beanDefinition中设置初始化和销毁方法
            beanDefinition.setInitMethodName(initMethod);
            beanDefinition.setDestroyMethodName(destroyMethodName);

            // 读取属性并填充
            for (int j = 0; j < bean.getChildNodes().getLength(); j++) {
                if (!(bean.getChildNodes().item(j) instanceof Element)) continue;
                if (!"property".equals(bean.getChildNodes().item(j).getNodeName())) continue;
                // 解析标签：property
                Element property = (Element) bean.getChildNodes().item(j);
                String attrName = property.getAttribute("name");
                String attrValue = property.getAttribute("value");
                String attrRef = property.getAttribute("ref");
                // 获取属性值：引入对象、值对象
                Object value = StrUtil.isNotEmpty(attrRef) ? new BeanReference(attrRef) : attrValue;
                // 创建属性信息
                PropertyValue propertyValue = new PropertyValue(attrName, value);
                beanDefinition.getPropertyValues().addPropertyValue(propertyValue);
            }
            if (getRegistry().containsBeanDefinition(beanName)) {
                throw new BeansException("Duplicate beanName[" + beanName + "] is not allowed");
            }
            // 注册 BeanDefinition
            getRegistry().registerBeanDefinition(beanName, beanDefinition);
        }
    }
}
```

在读取和解析xml配置中，新增读取初始化和销毁的方法标签，让后将名字存储在BeanDefinition对象中,如下代码块:
```java
beanDefinition.setInitMethodName(initMethod);
beanDefinition.setDestroyMethodName(destroyMethodName);
```

### 3.4、执行Bean对象初始化和销毁

思考一个问题，什么时候开始执行Bean对象的初始化和销毁动作？换句话说，我们应该在哪一个类中嵌入Bean的初始化和销毁方法。

参考核心抽象类AbstractAutowireCapableBeanFactory#createBean()方法，在这个方法中完成了**Bean对象的创建，设置属性值，前置和后置处理器的执行**等步骤，所以如果想要那个做Bean对象的初始化动作，那么就需要再Bean被创建并且设置属性后执行，所以初始化动作的最佳嵌入点就是createBean()方法。

>
> 执行顺序：
> 1. Bean对象的实例化
> 2. Bean对象属性设置
> 3. Bean对象前置处理器调用
> 4. Bean对象init初始化方法调用
> 5. Bean对象后置处理器调用

```java
/**
 * 继承抽象的工厂类，实现自己的方法，公共的方法由抽象类实现
 * 体现了类实现过程中的各司其职，你只需要关心属于你的内容，不是你的内容，不要参与
 */
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {

//    实例化bean对象带参数
    private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

    /**
     * 无参数创建bean对象
     * @param beanName
     * @param beanDefinition
     * @return
     * @throws BeansException
     */
    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition) throws BeansException {
        Object bean;
        try {
//            无参构造对象
            bean = beanDefinition.getBeanClass().newInstance();
            // 给 Bean 填充属性
            applyPropertyValues(beanName, bean, beanDefinition);

            // 执行 Bean 的初始化方法和 BeanPostProcessor 的前置和后置处理方法
            bean = initializeBean(beanName, bean, beanDefinition);

        } catch (InstantiationException | IllegalAccessException e) {
            throw new BeansException("Instantiation of bean failed", e);
        }


        // 注册实现了 DisposableBean 接口的 Bean 对象
        registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);

        addSingleton(beanName, bean);
        return bean;
    }

    /**
     * 注册Bean的销毁方法
     * @param beanName
     * @param bean
     * @param beanDefinition
     */
    protected void registerDisposableBeanIfNecessary(String beanName, Object bean, BeanDefinition beanDefinition) {
        if (bean instanceof DisposableBean || StrUtil.isNotEmpty(beanDefinition.getDestroyMethodName())) {
            registerDisposableBean(beanName, new DisposableBeanAdapter(bean, beanName, beanDefinition));
        }
    }

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
            /*调用Bean的构造函数实例化对象*/
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
     * 获取类的所有构造函数，然后使用构造函数创建Bean对象
     * @param beanDefinition
     * @param beanName
     * @param args
     * @return
     */
    protected Object createBeanInstance(BeanDefinition beanDefinition, String beanName, Object[] args) {
        Constructor constructorToUse = null;
        Class<?> beanClass = beanDefinition.getBeanClass();
        Constructor<?>[] declaredConstructors = beanClass.getDeclaredConstructors();
        for (Constructor ctor : declaredConstructors) {
            if (null != args && ctor.getParameterTypes().length == args.length) {
                constructorToUse = ctor;
                break;
            }
        }
        return getInstantiationStrategy().instantiate(beanDefinition, beanName, constructorToUse, args);
    }

    /**
     * Bean的属性值填充
     * @param beanName
     * @param bean
     * @param beanDefinition
     */
    protected void applyPropertyValues(String beanName, Object bean, BeanDefinition beanDefinition) {
        try {
            PropertyValues propertyValues = beanDefinition.getPropertyValues();
            for (PropertyValue propertyValue : propertyValues.getPropertyValues()) {

                String name = propertyValue.getName();
                Object value = propertyValue.getValue();

                if (value instanceof BeanReference) {
                    // A 依赖 B，获取 B 的实例化
                    BeanReference beanReference = (BeanReference) value;
                    value = getBean(beanReference.getBeanName());
                }
                // 属性填充
                BeanUtil.setFieldValue(bean, name, value);
            }
        } catch (Exception e) {
            throw new BeansException("Error setting property values：" + beanName);
        }
    }

    public InstantiationStrategy getInstantiationStrategy() {
        return instantiationStrategy;
    }

    public void setInstantiationStrategy(InstantiationStrategy instantiationStrategy) {
        this.instantiationStrategy = instantiationStrategy;
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

        // 1. 执行 BeanPostProcessor Before 处理
        Object wrappedBean = applyBeanPostProcessorsBeforeInitialization(bean, beanName);

        // 待完成内容：invokeInitMethods(beanName, wrappedBean, beanDefinition);
        // 执行 Bean 对象的初始化方法
        try {
//            执行Bean的初始化方法
            invokeInitMethods(beanName, wrappedBean, beanDefinition);
        } catch (Exception e) {
            throw new BeansException("Invocation of init method of bean[" + beanName + "] failed", e);
        }
        // 2. 执行 BeanPostProcessor After 处理
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        return wrappedBean;
    }

    /**
     * 执行init方法
     * @param beanName
     * @param bean
     * @param beanDefinition
     * @throws Exception
     */
    private void invokeInitMethods(String beanName, Object bean, BeanDefinition beanDefinition) throws Exception {
        // 1. 实现接口 InitializingBean
        if (bean instanceof InitializingBean) {
            ((InitializingBean) bean).afterPropertiesSet();
        }
        // 2. 注解配置 init-method {判断是为了避免二次执行初始化}
        String initMethodName = beanDefinition.getInitMethodName();
        if (StrUtil.isNotEmpty(initMethodName) && !(bean instanceof InitializingBean)) {
            Method initMethod = beanDefinition.getBeanClass().getMethod(initMethodName);
            if (null == initMethod) {
                throw new BeansException("Could not find an init method named '" + initMethodName + "' on bean with name '" + beanName + "'");
            }
            initMethod.invoke(bean);
        }

    }

    /**
     * 执行前置处理器方法
     * @param existingBean
     * @param beanName
     * @return
     * @throws BeansException
     */
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

    /**
     * 调用后置处理器方法
     * @param existingBean
     * @param beanName
     * @return
     * @throws BeansException
     */
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

首先全局看一下Bean对象的生命周期：

Bean对象的创建 -> Bean属性填充 -> 执行前置处理器 -> 初始化方法 -> 执行后置处理器，在初始化方法里面，分别在初始化动作的前后执行bean的前置和后置处理器。

抽象类AbstractAutowireCapableBeanFactory中完成了具体的Bean对象创建，所以我们要在执行后置处理器前增加执行初始化方法：initializeBean()

**初始化动作**

方法中invokeInitMethods()方法调用就是执行Bean对象的初始化方法， 在方法 invokeInitMethods 中，**主要分为两块来执行实现了 InitializingBean 接口的操作，处理 afterPropertiesSet 方法（初始化方法）。另外一个是判断配置信息 init-method 是否存在，执行反射调用 initMethod.invoke(bean)。这两种方式都可以在 Bean 对象初始化过程中进行处理加载 Bean 对象中的初始化操作，让使用者可以额外新增加自己想要的动作**  。

另外考虑为什么销毁方法需要进行注册？

registerDisposableBeanIfNecessary()：注册销毁动作

```java
/**
 * 默认的单例接口的实现
 */
public class DefaultSingletonBeanRegistry implements SingletonBeanRegistry {

    private final Map<String, DisposableBean> disposableBeans = new HashMap<>();

//    存放实例化后的bean对象
    private final Map<String, Object> singletonObjects = new HashMap<>();

    @Override
    public Object getSingleton(String beanName) {
        return singletonObjects.get(beanName);
    }

    protected void addSingleton(String beanName, Object singletonObject) {
        singletonObjects.put(beanName, singletonObject);
    }

    /**
     * 注册DisposableBean对象
     * @param beanName
     * @param bean
     */
    public void registerDisposableBean(String beanName,DisposableBean bean){
        this.disposableBeans.put(beanName,bean);
    }

    public void destroySingletons() {
        Set<String> keySet = this.disposableBeans.keySet();
        Object[] disposableBeanNames = keySet.toArray();

        for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
            Object beanName = disposableBeanNames[i];
            DisposableBean disposableBean = disposableBeans.remove(beanName);
            try {
                disposableBean.destroy();
            } catch (Exception e) {
                throw new BeansException("Destroy method on bean with name '" + beanName + "' threw an exception", e);
            }
        }
    }

}
```

单例接口的默认实现中，新增map容器用来存储注册的销毁方法。

- 在创建 Bean 对象的实例的时候，需要把销毁方法保存起来，方便后续执行销毁动作进行调用。
- 那么这个销毁方法的具体方法信息，会被注册到 DefaultSingletonBeanRegistry 中新增加的 Map<String, DisposableBean> disposableBeans 属性中去，**因为这个接口的方法最终可以被类 AbstractApplicationContext 的 close 方法通过 getBeanFactory().destroySingletons() 调用**。
- 在注册销毁方法的时候，会根据是接口类型和配置类型统一交给 DisposableBeanAdapter 销毁适配器类来做统一处理。*实现了某个接口的类可以被 instanceof 判断或者强转后调用接口方法*

DefaultSingletonBeanRegistry:默认的单例接口实现中，有两个容器，其一是用来保存单例对象的map，其二就是用来注册销毁方法的map。

### 3.5、定义销毁方法适配器(接口和配置)

思考一下销毁动作为什么需要一个适配器？

```java
public class DisposableBeanAdapter implements DisposableBean {

    private final Object bean;
    private final String beanName;
    private String destroyMethodName;

    public DisposableBeanAdapter(Object bean, String beanName, BeanDefinition beanDefinition) {
        this.bean = bean;
        this.beanName = beanName;
        this.destroyMethodName = beanDefinition.getDestroyMethodName();
    }

    @Override
    public void destroy() throws Exception {
        // 1. 实现接口 DisposableBean
        if (bean instanceof DisposableBean) {
            ((DisposableBean) bean).destroy();
        }

        // 2. 注解配置 destroy-method {判断是为了避免二次执行销毁}
        if (StrUtil.isNotEmpty(destroyMethodName) && !(bean instanceof DisposableBean && "destroy".equals(this.destroyMethodName))) {
            Method destroyMethod = bean.getClass().getMethod(destroyMethodName);
            if (null == destroyMethod) {
                throw new BeansException("Couldn't find a destroy method named '" + destroyMethodName + "' on bean with name '" + beanName + "'");
            }
            destroyMethod.invoke(bean);
        }
    }
}
```

- 可能你会想这里怎么有一个适配器的类呢，**因为销毁方法有两种甚至多种方式，目前有实现接口 DisposableBean、配置信息 destroy-method，两种方式**。而这两种方式的销毁动作是由 AbstractApplicationContext 在注册虚拟机钩子后看，虚拟机关闭前执行的操作动作。
- 那么在销毁执行时不太希望还得关注都销毁那些类型的方法，它的使用上更希望是有一个统一的接口进行销毁，所以这里就新增了适配类，做统一处理。

### 3.6、虚拟机关闭钩子注册调用销毁方法

```java
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

首先我们需要在 ConfigurableApplicationContext 接口中定义注册虚拟机钩子的方法 registerShutdownHook 和手动执行关闭的方法 close。

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

        // 3. 在 Bean 实例化（创建）之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
        invokeBeanFactoryPostProcessors(beanFactory);

        // 4. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
        registerBeanPostProcessors(beanFactory);

        // 5. 提前实例化单例Bean对象
        beanFactory.preInstantiateSingletons();
    }

    @Override
    public void registerShutdownHook() {
        Runtime.getRuntime().addShutdownHook(new Thread(this::close));
    }

    @Override
    public void close() {
        getBeanFactory().destroySingletons();
    }

}
```

梳理一下这一块的逻辑：

1. 在ConfigurableBeanFactory接口中定义destroySingletons，销毁单例对象的方法。
2. 在DefaultSingletonBeanRegistry类中实现销毁单例对象的方法。
3. 在ConfigurableApplicationContext接口中定义钩子函数和手动关闭的方法。
4. 在AbstractApplicationContext中实现具体的钩子函数调用和关闭方法，在close方法中调用destroySingletons方法实现关闭注销。



为什么在close中可以实现调用destroySingletons方法功能

因为ConfigurableListableBeanFactory抽象类继承继承ConfigurableBeanFactory接口，销毁方法定义在这个接口中。



## 4、测试

### 4.1、测试用例

DAO对象

```java
public class PeopleDao {

    private static Map<String,String> map = new HashMap<>();

    /**
     * 初始化方法初始化数据
     */
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

Service实现

```java
public class PeopleService implements InitializingBean, DisposableBean {
    private String id;
    private String company;
    private String location;
    private PeopleDao peopleDao;

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + peopleDao.queryUserName(id)+"  公司名称："+company+"  公司地点："+location);
    }


    @Override
    public void destroy() throws Exception {
        System.out.println("执行PeopleService.destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("执行执行PeopleService.afterPropertiesSet");
    }
}
```

- UserDao，修改了之前使用 static 静态块初始化数据的方式，改为提供 initDataMethod 和 destroyDataMethod 两个更优雅的操作方式进行处理。
- UserService，以实现接口 InitializingBean, DisposableBean 的两个方法 destroy()、afterPropertiesSet()，处理相应的初始化和销毁方法的动作。

配置文件

```java
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

### 4.2、单元测试

```java
    @Test
    public void test_xml() {
        // 1.初始化 BeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2. 读取配置文件&注册Bean
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        reader.loadBeanDefinitions("classpath:spring.xml");

        // 3. 获取Bean对象调用方法
        PeopleService peopleService = (PeopleService)beanFactory.getBean("peopleService", PeopleService.class);
        peopleService.queryUserInfo();
    }
```



测试结果

```java
执行：init-method
执行执行PeopleService.afterPropertiesSet
查询用户信息：工程师  公司名称：腾讯  公司地点：上海
执行：destroy-method
执行PeopleService.destroy

Process finished with exit code 0
```

## 5、小结

本章使用设计模式：

1. 执行销毁方法的时候，使用适配器模式，分析为什么要使用适配器模式。
2. 因为调用销毁方法的时候，有两种方法调用，接口引用和反射方式调用。



在学习和动手实践 Spring 框架学习的过程中，特别要注意的是它对接口和抽象类的把握和使用，尤其遇到类似，A继承B实现C时，C的接口方法由A继承的父类B实现，这样的操作都蛮有意思的。也是可以复用到通常的业务系统开发中进行处理一些复杂逻辑的功能分层，做到程序的可扩展、易维护等特性。



最后，来一起总结一下Bean对象的生命周期：

1. 加载BeanDefinition
2. 注册BeanDefinition到容器内
3. 修改bean定义
4. 实例化Bean
5. 填充属性
6. 执行前置处理器
7. 执行初始化方法
8. 执行后置处理器
9. 执行销毁方法。