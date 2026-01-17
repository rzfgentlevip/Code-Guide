---
title: 4、属性注入和依赖对象填充
icon: file
order: 4
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
permalink: /project-docs/project/springframework/属性注入和依赖对象填充/
---

<!-- TOC -->

- [4、属性注入和依赖对象填充](#4属性注入和依赖对象填充)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类实现](#31核心类实现)
    - [3.2、定义属性](#32定义属性)
    - [3.3、BeanDefinition定义补全](#33beandefinition定义补全)
    - [3.4、Bean属性的填充](#34bean属性的填充)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)

<!-- /TOC -->

# 4、属性注入和依赖对象填充

## 1、目标

到目前为止，我们已经实现了一个简易的spring容器，并且容器具备了bean定义的注册和bean对象的创建，在创建bean对象时候可以根据参数初始化对象，可以思考一下，如果创建的bean中有属性怎么办，只有对象依赖的属性也被填充上，才算完整的创建成功一个bean对象，如果我们的bean对象A里面依赖了bean对象B怎么办，那这个时候在初始化A的时候，B对象就应该已经创建并且初始化好，这样A才能将bean对象B注入自己的内部，我们暂且不考虑循环引用问题。

对于属性的填充不只是 int、Long、String，还包括还没有实例化的对象属性，都需要在 Bean 创建时进行填充操作。*不过这里我们暂时不会考虑 Bean 的循环依赖，否则会把整个功能实现撑大，这样新人学习时就把握不住了，待后续陆续先把核心功能实现后，再逐步完善*

## 2、设计

在写代码之前，我们首先考虑一个问题，到目前为止，我们的spring容器都帮助我们做了哪些工作？

1. 首先spring容器本身帮助我们管理了bean对象，即我们创建的bean对象被放在容器中;
2. 其二，spring容器帮助我们注册了bean的定义信息到容器中;
3. 三，spring容器帮助我们实例化对象，然后将实例化的对象放到了容器中供应用程序使用;

所以看到这里，我们应该大致已经明白bean对象的属性赋值应该在什么阶段，赋值前bean对象肯定会被创建好，因此我们就在创建好bean对象后，对属性进行赋值操作。

思考问题：

1. 在什么时机将填充属性动作串联到bean的生命周期中？
2. 填充属性如果有引用依赖怎么办？

<img src="https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408020953338.png" alt="step03-design-实例化bean带参数-对象属性填充.drawio.png" style="zoom:80%;" />

回到上面来年各个问题中，对象属性的填充，应该在对象实例化完毕后，**因此应该在调用对象的构造函数完后再去调用属性初始化方法**[对象实例化完成后，值都是各种类型的默认值]，因此将两部分设计为不同的功能，bean对象的创建，bean对象属性的填充。而属性值具体从哪里获取，因为在整个程序设计中，bean的所有定义信息都被封装在BeanDefinitioin中，因此，以后所有与bean定义相关的内容都被放到这个类里面。

另外是填充属性信息还包括了 Bean 的对象类型，也就是需要再定义一个 BeanReference，里面其实就是一个简单的 Bean 名称，在具体的实例化操作时进行递归创建和填充，与 Spring 源码实现一样。*Spring 源码中 BeanReference 是一个接口*

## 3、实现

### 3.1、核心类实现

![spring-application-autowarePropertyAndObj.drawio.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408020954077.png)

新增PropertyValue类，定义bean对象属性的名字和值，因为一个bean对象可能存在多个属性，因此在定义PropertyValues属性，封装一个bean对象的多个属性，BeanReference(类引用)  用于存放引用类型对象。

### 3.2、定义属性

```java
/**
 *
 * bean 属性信息定义
 */
public class PropertyValue {

//    属性名字
    private final String name;
//     属性值
    private final Object value;

    public PropertyValue(String name, Object value) {
        this.name = name;
        this.value = value;
    }

    public String getName() {
        return name;
    }

    public Object getValue() {
        return value;
    }
}
```

因为需要给bean里面的属性赋值，所以首先要完成的步骤是将bean的属性信息也一起注入到bean的定义中，在实例化bean之后在对属性赋值。

```java
/**
 * 属性集合，一个bean可能存在多个属性
 */
public class PropertyValues {

    private final List<PropertyValue> propertyValueList = new ArrayList<>();

    public void addPropertyValue(PropertyValue pv) {
        this.propertyValueList.add(pv);
    }

    public PropertyValue[] getPropertyValues() {
        return this.propertyValueList.toArray(new PropertyValue[0]);
    }

    public PropertyValue getPropertyValue(String propertyName) {
        for (PropertyValue pv : this.propertyValueList) {
            if (pv.getName().equals(propertyName)) {
                return pv;
            }
        }
        return null;
    }
}
```

一个bean对象可能存在多个属性，所以需要使用集合将bean的属性存放在一起。

### 3.3、BeanDefinition定义补全

```java
public class BeanDefinition {

//    定义类信息
    private Class beanClass;

//    同时要把bean的属性注册进来
    private PropertyValues propertyValues;

    public BeanDefinition(Class beanClass) {
        this.beanClass = beanClass;
        this.propertyValues = new PropertyValues();
    }
}
```

- 在 Bean 注册的过程中是需要传递 Bean 的信息，在几个前面章节的测试中都有所体现 new BeanDefinition(UserService.class, propertyValues);
- 所以为了把属性一定交给 Bean 定义，这里填充了 PropertyValues 属性，同时把两个构造函数做了一些简单的优化，避免后面 for 循环时还得判断属性填充是否为空。

### 3.4、Bean属性的填充

```java
/**
 * 继承抽象的工厂类，实现自己的方法，公共的方法由抽象类实现
 * 体现了类实现过程中的各司其职，你只需要关心属于你的内容，不是你的内容，不要参与
 */
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory {
    private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

    /**
     * 不带属性的Bean对象
     * @author yourname
     * @date 20:03 2024/11/23 
     * @param beanName
     * @param beanDefinition 
     * @return java.lang.Object
     **/
    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition) throws BeansException {
        Object bean;
        try {
//            无参构造对象
            bean = beanDefinition.getBeanClass().newInstance();
            // 给 Bean 填充属性
            applyPropertyValues(beanName, bean, beanDefinition);
        } catch (InstantiationException | IllegalAccessException e) {
            throw new BeansException("Instantiation of bean failed", e);
        }

        addSingleton(beanName, bean);
        return bean;
    }

    /**
     * 带属性信息的Bean对象
     * @author yourname
     * @date 20:03 2024/11/23 
     * @param beanName
     * @param beanDefinition
     * @param args 
     * @return java.lang.Object
     **/
    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
        Object bean;
        try {
            /*创建bean对象实例 也就是调用对象的构造方法创建对象*/
            bean = createBeanInstance(beanDefinition,beanName,args);
            // 在bean实例化完成之后【调用构造防范】，实例化之后bean的属性都是默认值，给 Bean 填充属性
            applyPropertyValues(beanName, bean, beanDefinition);
        } catch (Exception e) {
            throw new BeansException("Instantiation of bean failed", e);
        }

        addSingleton(beanName, bean);
        return bean;
    }

    /**
     * 获取类的所有构造函数
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
     * Bean 属性填充，bean属性的填充在实例化bean完成之后
     * 实例化只是创建出对象，对象所有的属性值都是默认值
     *
     * 递归创建bean和赋值
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
//                    如果注入的某一个bean没有实例化，那么就就行实例化和赋值
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
}
```

抽象类中重点关注createBean()方法，在这个方法中首先进行bean对象的创建【调用bean对象的构造方法创建对象，此时对象中属性都是默认值】，然后调用applyPropertyValues()方法对bean中的属性进行填充，填充过程中对基本数据类型直接进行属性值的设置，如果遇到引用类型，会递归调用getBean()方法完成引用类型的注入，然后再赋值操作。需要注意的是这里并没有解决循环引用问题。

这里体现了一点，方法级别的单一职责原则，一个方法只负责一个职责。

1. createBean()方法负责创建对象总调入口
2. createBeanInstance()负责实例化一个bean对象
3. applyPropertyValues()方法负责给对象填充属性值。

createBean();此方法内部调用的方法链路过长，因此在平时的项目开发中，不要使一个方法中调用的链路过长；


## 4、测试

### 4.1、测试用例

**PeopleDao对象**

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

**peopleService对象**

```java
public class PeopleService {
    private String id;

    private PeopleDao peopleDao;

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + peopleDao.queryUserName(id));
    }

}
```

Dao对象主要负责数据库操作，在这里我们模拟查询数据库数据，然后再Service里面注入Dao对象使用其查询出来的数据。

### 4.2、测试结果

```java
   @Test
    public void testBeanDefine(){
        // 1.初始化 BeanFactory，获取Bean对象的接口
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2.注册 bean 将Bean的class信息注册到容器中
        beanFactory.registerBeanDefinition("peopleDao",new BeanDefinition(PeopleDao.class));

        // 3. peopleService 设置属性[id、userDao]
        PropertyValues propertyValues = new PropertyValues();
        propertyValues.addPropertyValue(new PropertyValue("id", "10001"));
        propertyValues.addPropertyValue(new PropertyValue("peopleDao",new BeanReference("peopleDao")));

        // 4. peopleService 注入bean
        BeanDefinition beanDefinition = new BeanDefinition(PeopleService.class, propertyValues);
        beanFactory.registerBeanDefinition("peopleService", beanDefinition);

        // 5. UserService 获取bean
        PeopleService peopleService = (PeopleService) beanFactory.getBean("peopleService");
        peopleService.queryUserInfo();
    }
```

**测试结果**：

```java
查询用户信息：工程师

Process finished with exit code 0
```

测试代码的编写中，首先实例化BeanFactory对象，注册Bean对象(Service和Dao)，设置属性对象，获取bean对象执行方法，目前都是需要手写代码帮助spring完成生命周期中的一些工作，在下章节中，将会实现自动读取xml文件实现注册功能。

## 5、总结

总结一下本章节完成的工作：

1. 主要完成对实例化的bean对象进行赋值操作，如果bean对象的属性是引用类型，那么需要先完成引用类型的初始化创建工作，然后才能赋值给当前Bean对象，只有Bean对象属性赋值完成，才能算Bean对象创建完成。
2. 单一职责原则适用于接口，同样也适用于方法，如果一个方法中流程过于臃肿，那么就可以将方法的职责拆分为多个方法，内一个方法负责流程中的一部分，最终将所有流程组装起来完成职责。
