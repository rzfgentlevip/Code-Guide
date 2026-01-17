---
title: 14、通过注解注入属性信息
icon: file
order: 14
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
permalink: /project-docs/project/springframework/通过注解注入属性信息/
---
<!-- TOC -->

- [14、通过注解注入属性信息](#14通过注解注入属性信息)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心实现类图](#31核心实现类图)
    - [3.2、把读取到的属性填充到容器](#32把读取到的属性填充到容器)
    - [3.3、自定义属性注入注解](#33自定义属性注入注解)
    - [3.4、扫描自定义注解](#34扫描自定义注解)
    - [3.5、向BeanFactory中注册AutowiredAnnotationBeanPostProcessor](#35向beanfactory中注册autowiredannotationbeanpostprocessor)
    - [3.6、在Bean的生命周期中调用属性注入](#36在bean的生命周期中调用属性注入)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、小结](#5小结)

<!-- /TOC -->


# 14、通过注解注入属性信息

## 1、目标
在目前 IOC、AOP 两大核心功能模块的支撑下，完全可以管理 Bean 对象的注册和获取，不过这样的使用方式总感觉像是刀耕火种有点难用。因此在上一章节我们解决需要手动配置 Bean 对象到 spring.xml 文件中，改为可以自动扫描带有注解 @Component 的对象完成自动装配和注册到 Spring 容器的操作。
那么在自动扫描包注册 Bean 对象之后，就需要把原来在配置文件中通过 property name="token" 配置属性和Bean的操作，也改为可以自动注入。这就像我们使用 Spring 框架中 @Autowired、@Value 注解一样，完成我们对属性和对象的注入操作。

## 2、设计

其实从我们在完成 Bean 对象的基础功能后，后续陆续添加的功能都是围绕着 Bean 的生命周期进行的，比如修改 Bean 的定义 BeanFactoryPostProcessor，处理 Bean 的属性要用到 BeanPostProcessor，完成个性的属性操作则专门继承 BeanPostProcessor 提供新的接口，因为这样才能通过 instanceof 判断出具有标记性的接口。所以关于 Bean 等等的操作，以及监听 Aware、获取 BeanFactory，都需要在 Bean 的生命周期中完成。那么我们在设计属性和 Bean 对象的注入时候，也会用到 BeanPostProcessor 来完成在设置 Bean 属性之前，允许 BeanPostProcessor 修改属性值。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/%E8%87%AA%E5%8A%A8%E6%B3%A8%E5%85%A5%E5%B1%9E%E6%80%A7.png)

要处理自动扫描注入，包括属性注入、对象注入，则需要在对象属性 applyPropertyValues 填充之前 ，把属性信息写入到 PropertyValues 的集合中去。这一步的操作相当于是解决了以前在 spring.xml 配置属性的过程。
而在属性的读取中，需要依赖于对 Bean 对象的类中属性的配置了注解的扫描，field.getAnnotation(Value.class); 依次拿出符合的属性并填充上相应的配置信息。这里有一点 ，属性的配置信息需要依赖于 BeanFactoryPostProcessor 的实现类 PropertyPlaceholderConfigurer，把值写入到 AbstractBeanFactory的embeddedValueResolvers集合中，这样才能在属性填充中利用 beanFactory 获取相应的属性值
还有一个是关于 @Autowired 对于对象的注入，其实这一个和属性注入的唯一区别是对于对象的获取 beanFactory.getBean(fieldType)，其他就没有什么差一点了。
当所有的属性被设置到 PropertyValues 完成以后，接下来就到了创建对象的下一步，属性填充，而此时就会把我们一一获取到的配置和对象填充到属性上，也就实现了自动注入的功能。

## 3、实现

### 3.1、核心实现类图

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/%E8%87%AA%E5%8A%A8%E8%AE%BE%E7%BD%AE%E5%B1%9E%E6%80%A7%E7%B1%BB%E5%9B%BE.png)

从类结构上看，主要是XmlBeanDefinitionReader类中的doLoadBeanDefinitions方法，在此方法中调用ClassPathScanningCandidateComponentProvider类中的方法逻辑，主要工作是根据包名，自动扫描包中被标注注解的类，然后返回具体的class信息。
因此就可以在加载阶段，自动实现根据包名扫描包路径下所有的类，将标注注解的class注册到容器中。

PropertyPlaceholderConfigurer 目前看上去像一块单独的内容，后续会把这块的内容与自动加载 Bean 对象进行整合，也就是可以在注解上使用占位符配置一些在配置文件里的属性信息，此类实现BeanFactoryPostProcessor接口，因为BeanFactoryPostProcessor可以在加载完成BeanDefinition后对定义进行修改，所以加载配置文件中的属性就需要再注册完成定义后，进行填充。



### 3.2、把读取到的属性填充到容器

定义解析字符串接口

```java
public interface StringValueResolver {

    String resolveStringValue(String strVal);
}
```

StringValueResolver是一个解析字符串操作的接口

填充字符串

```java
public class PropertyPlaceholderConfigurer implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        try {
            // 加载属性文件
            DefaultResourceLoader resourceLoader = new DefaultResourceLoader();
            Resource resource = resourceLoader.getResource(location);

            // 占位符替换属性值
            Properties properties = new Properties();
            properties.load(resource.getInputStream());

            String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
            for (String beanName : beanDefinitionNames) {
                BeanDefinition beanDefinition = beanFactory.getBeanDefinition(beanName);

                PropertyValues propertyValues = beanDefinition.getPropertyValues();
                for (PropertyValue propertyValue : propertyValues.getPropertyValues()) {
                    Object value = propertyValue.getValue();
                    if (!(value instanceof String)) continue;
                    value = resolvePlaceholder((String) value, properties);
                    propertyValues.addPropertyValue(new PropertyValue(propertyValue.getName(), value));
                }
            }

            // 向容器中添加字符串解析器，供解析@Value注解使用
            StringValueResolver valueResolver = new PlaceholderResolvingStringValueResolver(properties);
            beanFactory.addEmbeddedValueResolver(valueResolver);

        } catch (IOException e) {
            throw new BeansException("Could not load properties", e);
        }
    }

    private class PlaceholderResolvingStringValueResolver implements StringValueResolver {

        private final Properties properties;

        public PlaceholderResolvingStringValueResolver(Properties properties) {
            this.properties = properties;
        }

        @Override
        public String resolveStringValue(String strVal) {
            return PropertyPlaceholderConfigurer.this.resolvePlaceholder(strVal, properties);
        }

    }

}
```
    在解析属性配置的类 PropertyPlaceholderConfigurer 中，最主要的其实就是这行代码的操作 beanFactory.addEmbeddedValueResolver(valueResolver) 这是把属性值写入到了 AbstractBeanFactory 的 embeddedValueResolvers 中。
    这里说明下，embeddedValueResolvers 是 AbstractBeanFactory 类新增加的集合 List<StringValueResolver> embeddedValueResolvers String resolvers to apply e.g. to annotation attribute values

### 3.3、自定义属性注入注解

```java
//定义AutoWare注解
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.CONSTRUCTOR, ElementType.FIELD, ElementType.METHOD})
public @interface Autowired {
}

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {

    String value() default "";

}

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {

    /**
     * The actual value expression: e.g. "#{systemProperties.myProp}".
     */
    String value();
}
```

### 3.4、扫描自定义注解

```java
public class AutowiredAnnotationBeanPostProcessor implements InstantiationAwareBeanPostProcessor, BeanFactoryAware {


    private ConfigurableListableBeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        return true;
    }

    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        // 1. 处理注解 @Value
        Class<?> clazz = bean.getClass();
        clazz = ClassUtils.isCglibProxyClass(clazz) ? clazz.getSuperclass() : clazz;

        Field[] declaredFields = clazz.getDeclaredFields();

        for (Field field : declaredFields) {
            Value valueAnnotation = field.getAnnotation(Value.class);
            if (null != valueAnnotation) {
                String value = valueAnnotation.value();
                value = beanFactory.resolveEmbeddedValue(value);
                BeanUtil.setFieldValue(bean, field.getName(), value);
            }
        }

        // 2. 处理注解 @Autowired
        for (Field field : declaredFields) {
            Autowired autowiredAnnotation = field.getAnnotation(Autowired.class);
            if (null != autowiredAnnotation) {
                Class<?> fieldType = field.getType();
                String dependentBeanName = null;
                Qualifier qualifierAnnotation = field.getAnnotation(Qualifier.class);
                Object dependentBean = null;
                if (null != qualifierAnnotation) {
                    dependentBeanName = qualifierAnnotation.value();
                    dependentBean = beanFactory.getBean(dependentBeanName, fieldType);
                } else {
                    dependentBean = beanFactory.getBean(fieldType);
                }
                BeanUtil.setFieldValue(bean, field.getName(), dependentBean);
            }
        }

        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        // Bug: 返回null会影响代理对象的创建
        // return null;
        return bean;
    }
}
```
AutowiredAnnotationBeanPostProcessor 是实现接口 InstantiationAwareBeanPostProcessor 的一个用于在 Bean 对象实例化完成后，设置属性操作前的处理属性信息的类和操作方法。只有实现了 BeanPostProcessor 接口才有机会在 Bean 的生命周期中处理初始化信息
核心方法 postProcessPropertyValues，主要用于处理类含有 @Value、@Autowired 注解的属性，进行属性信息的提取和设置。
这里需要注意一点因为我们在 AbstractAutowireCapableBeanFactory 类中使用的是 CglibSubclassingInstantiationStrategy 进行类的创建，所以在 AutowiredAnnotationBeanPostProcessor#postProcessPropertyValues 中需要判断是否为 CGlib 创建对象，否则是不能正确拿到类信息的。ClassUtils.isCglibProxyClass(clazz) ? clazz.getSuperclass() : clazz;

### 3.5、向BeanFactory中注册AutowiredAnnotationBeanPostProcessor

```java
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {

    /**
     *公共的加载xml文件的方法
     * @param inputStream
     * @throws ClassNotFoundException
     */
    protected void doLoadBeanDefinitions(InputStream inputStream) throws ClassNotFoundException, DocumentException {
        SAXReader reader = new SAXReader();
        Document document = reader.read(inputStream);
        Element root = document.getRootElement();

        // 解析 context:component-scan 标签，扫描包中的类并提取相关信息，用于组装 BeanDefinition
        Element componentScan = root.element("component-scan");
        if (null != componentScan) {
            String scanPath = componentScan.attributeValue("base-package");
            if (StrUtil.isEmpty(scanPath)) {
                throw new BeansException("The value of base-package attribute can not be empty or null");
            }
            scanPackage(scanPath);
        }

        List<Element> beanList = root.elements("bean");
        for (Element bean : beanList) {
        
        
            // 注册 BeanDefinition
            getRegistry().registerBeanDefinition(beanName, beanDefinition);
        }
    }

    private void scanPackage(String scanPath) {
        String[] basePackages = StrUtil.splitToArray(scanPath, ',');
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(getRegistry());
        scanner.doScan(basePackages);
    }
}
```

### 3.6、在Bean的生命周期中调用属性注入

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {


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
        Object bean = null;
        try {
            // 判断是否返回代理 Bean 对象
            bean = resolveBeforeInstantiation(beanName, beanDefinition);
            if (null != bean) {
                return bean;
            }
            // 实例化 Bean
            bean = createBeanInstance(beanDefinition, beanName, args);
            // 在设置 Bean 属性之前，允许 BeanPostProcessor 修改属性值
            applyBeanPostProcessorsBeforeApplyingPropertyValues(beanName, bean, beanDefinition);
            // 给 Bean 填充属性
            applyPropertyValues(beanName, bean, beanDefinition);
            // 执行 Bean 的初始化方法和 BeanPostProcessor 的前置和后置处理方法
            bean = initializeBean(beanName, bean, beanDefinition);
        } catch (Exception e) {
            throw new BeansException("Instantiation of bean failed", e);
        }

        // 注册实现了 DisposableBean 接口的 Bean 对象
        registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);

        // 判断 SCOPE_SINGLETON、SCOPE_PROTOTYPE
        if (beanDefinition.isSingleton()) {
            registerSingleton(beanName, bean);
        }
        return bean;
    }


    /**
     * 在设置 Bean 属性之前，允许 BeanPostProcessor 修改属性值
     * 14章节新增
     * @param beanName
     * @param bean
     * @param beanDefinition
     */
    protected void applyBeanPostProcessorsBeforeApplyingPropertyValues(String beanName, Object bean, BeanDefinition beanDefinition) {
        for (BeanPostProcessor beanPostProcessor : getBeanPostProcessors()) {
            if (beanPostProcessor instanceof InstantiationAwareBeanPostProcessor){
                PropertyValues pvs = ((InstantiationAwareBeanPostProcessor) beanPostProcessor).postProcessPropertyValues(beanDefinition.getPropertyValues(), bean, beanName);
                if (null != pvs) {
                    for (PropertyValue propertyValue : pvs.getPropertyValues()) {
                        beanDefinition.getPropertyValues().addPropertyValue(propertyValue);
                    }
                }
            }
        }
    }

}

```

AbstractAutowireCapableBeanFactory#createBean 方法中有这一条新增加的方法调用，就是在设置 Bean 属性之前，允许 BeanPostProcessor 修改属性值 的操作 applyBeanPostProcessorsBeforeApplyingPropertyValues
那么这个 applyBeanPostProcessorsBeforeApplyingPropertyValues 方法中，首先就是获取已经注入的 BeanPostProcessor 集合并从中筛选出继承接口 InstantiationAwareBeanPostProcessor 的实现类。
最后就是调用相应的 postProcessPropertyValues 方法以及循环设置属性值信息，beanDefinition.getPropertyValues().addPropertyValue(propertyValue);


## 4、测试

### 4.1、测试用例

**Dao对象**
```java
@Component
public class UserDao {

    private static Map<String, String> hashMap = new HashMap<>();

    static {
        hashMap.put("10001", "bugcode，上海，浦东");
        hashMap.put("10002", "八杯水，上海，尖沙咀");
        hashMap.put("10003", "阿毛，天津，东丽区");
    }

    public String queryUserName(String uId) {
        return hashMap.get(uId);
    }

}
```
将DAO对象注册到Service对象中
```java
@Component("userService")
public class UserService implements IUserService {

    @Value("${token}")
    private String token;

    @Autowired
    private UserDao userDao;

    public String queryUserInfo() {
        try {
            Thread.sleep(new Random(1).nextInt(100));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return userDao.queryUserName("10001") + "，" + token;
    }

    public String register(String userName) {
        try {
            Thread.sleep(new Random(1).nextInt(100));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "注册用户：" + userName + " success！";
    }
}

```

**属性配置**
token.properties
```text
# Config File
system.key=OLpj9823dZ
```

**xml配置文件**

spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	         http://www.springframework.org/schema/beans/spring-beans.xsd
		 http://www.springframework.org/schema/context">


    <!--    指定配置文件的位置-->
    <bean class="bugcode.online.springframework.beans.factory.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:token.properties"/>
    </bean>

    <!--    指定包扫描路径-->
    <context:component-scan base-package="bugcode.online.springframework.bean"/>

</beans>
```

### 4.2、测试结果

**测试占位符**

```java
@Test
    public void test_scan() {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        IUserService userService = applicationContext.getBean("userService", IUserService.class);
        System.out.println("测试结果：" + userService.queryUserInfo());
    }
```

**结果**
```text
测试结果：bugcode，上海，浦东，RejDlI78hu223Opo983Ds

Process finished with exit code 0
```

## 5、小结

> 从整个注解信息扫描注入的实现内容来看，我们一直是围绕着在 Bean 的生命周期中进行处理，就像 BeanPostProcessor 用于修改新实例化 Bean 对象的扩展点，提供的接口方法可以用于处理 Bean 对象实例化前后进行处理操作。而有时候需要做一些差异化的控制，所以还需要继承 BeanPostProcessor 接口，定义新的接口 InstantiationAwareBeanPostProcessor 这样就可以区分出不同扩展点的操作了。 
> 像是接口用 instanceof 判断，注解用 Field.getAnnotation(Value.class); 获取，都是相当于在类上做的一些标识性信息，便于可以用一些方法找到这些功能点，以便进行处理。所以在我们日常开发设计的组件中，也可以运用上这些特点。 
> 当你思考把你的实现融入到一个已经细分好的 Bean 生命周期中，你会发现它的设计是如此的好，可以让你在任何初始化的时间点上，任何面上，都能做你需要的扩展或者改变，这也是我们做程序设计时追求的灵活性。