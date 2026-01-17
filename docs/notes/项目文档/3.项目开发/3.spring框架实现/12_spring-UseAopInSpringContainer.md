---
title: 12、将AOP扩展到Bean生命周期
icon: file
order: 12
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
permalink: /project-docs/project/springframework/将AOP扩展到Bean生命周期/
---


<!-- TOC -->

- [这是文章的标题](#这是文章的标题)
- [你可以自定义封面图片](#你可以自定义封面图片)
- [cover: /assets/images/cover1.jpg](#cover-assetsimagescover1jpg)
- [这是页面的图标](#这是页面的图标)
- [这是侧边栏的顺序](#这是侧边栏的顺序)
- [设置作者](#设置作者)
- [设置写作时间](#设置写作时间)
- [一个页面可以有多个分类](#一个页面可以有多个分类)
- [一个页面可以有多个标签](#一个页面可以有多个标签)
- [此页面会在文章列表置顶](#此页面会在文章列表置顶)
- [此页面会出现在星标文章中](#此页面会出现在星标文章中)
- [你可以自定义页脚](#你可以自定义页脚)
- [你可以自定义版权信息](#你可以自定义版权信息)
- [12、将AOP扩展到Bean生命周期](#12将aop扩展到bean生命周期)
    - [1、目标](#1目标)
    - [2、设计](#2设计)
    - [3、实现](#3实现)
        - [3.1、核心实现类图](#31核心实现类图)
        - [3.2、定义Advice拦截连](#32定义advice拦截连)
        - [3.3、定义Advisor访问者](#33定义advisor访问者)
        - [3.4、方法拦截器实现](#34方法拦截器实现)
        - [3.5、代理工厂](#35代理工厂)
        - [3.6、融入到bean生命周期的自动代理创建者](#36融入到bean生命周期的自动代理创建者)
        - [3.7、融入bean的生命周期](#37融入bean的生命周期)
    - [4、测试](#4测试)
        - [4.1、测试用例](#41测试用例)
        - [4.2、测试结果](#42测试结果)

<!-- /TOC -->

# 12、将AOP扩展到Bean生命周期

## 1、目标
在上一章节我们通过基于 Proxy.newProxyInstance 代理操作中处理方法匹配和方法拦截，对匹配的对象进行自定义的处理操作。并把这样的技术核心内容拆解到 Spring 中，用于实现 AOP 部分，通过拆分后基本可以明确各个类的职责，包括你的代理目标对象属性、拦截器属性、方法匹配属性，以及两种不同的代理操作 JDK 和 CGlib 的方式。
再有了一个 AOP 核心功能的实现后，我们可以通过单元测试的方式进行验证切面功能对方法进行拦截，但如果这是一个面向用户使用的功能，就不太可能让用户这么复杂且没有与 Spring 结合的方式单独使用 AOP，虽然可以满足需求，但使用上还是过去分散。
因此我们需要在本章节完成 AOP 核心功能与 Spring 框架的整合，最终能通过在 Spring 配置的方式完成切面的操作。


## 2、设计
在讲Aop融入到Spring Bean生命周期中时要考虑两个问题：
1. 在什么阶段将AOP融入到生命周期中。
2. 如何组装各项切点、拦截、前置的功能和适配对应的代理器


为了可以让对象创建过程中，能把xml中配置的代理对象也就是切面的一些类对象实例化，就需要用到 BeanPostProcessor 提供的方法，因为这个类的中的方法可以分别作用与 Bean 对象执行初始化前后修改 Bean 的对象的扩展信息。但这里需要集合于 BeanPostProcessor 实现新的接口和实现类，这样才能定向获取对应的类信息。
但因为创建的是代理对象不是之前流程里的普通对象，所以我们需要前置于其他对象的创建，所以在实际开发的过程中，需要在 AbstractAutowireCapableBeanFactory#createBean 优先完成 Bean 对象的判断，是否需要代理，有则直接返回代理对象。在Spring的源码中会有 createBean 和 doCreateBean 的方法拆分
这里还包括要解决方法拦截器的具体功能，提供一些 BeforeAdvice、AfterAdvice 的实现，让用户可以更简化的使用切面功能。除此之外还包括需要包装切面表达式以及拦截方法的整合，以及提供不同类型的代理方式的代理工厂，来包装我们的切面服务。


## 3、实现

### 3.1、核心实现类图

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/beanLifecycle.png)

整个类关系图中可以看到，在以 BeanPostProcessor 接口实现继承的 InstantiationAwareBeanPostProcessor 接口后，做了一个自动代理创建的类 DefaultAdvisorAutoProxyCreator，这个类的就是用于处理整个 AOP 代理融入到 Bean 生命周期中的核心类。
DefaultAdvisorAutoProxyCreator 会依赖于拦截器、代理工厂和Pointcut与Advisor的包装服务 AspectJExpressionPointcutAdvisor，由它提供切面、拦截方法和表达式。
Spring 的 AOP 把 Advice 细化了 BeforeAdvice、AfterAdvice、AfterReturningAdvice、ThrowsAdvice，目前我们做的测试案例中只用到了 BeforeAdvice，这部分可以对照 Spring 的源码进行补充测试

### 3.2、定义Advice拦截连

```java
public interface Advice {

}
```

**定义前置通知BeforeAdvice接口**
```java
public interface BeforeAdvice extends Advice {
}
```
MethodBeforeAdvice接口：
```java
public interface MethodBeforeAdvice extends BeforeAdvice {

    /**
     * Callback before a given method is invoked.
     *
     * @param method method being invoked
     * @param args   arguments to the method
     * @param target target of the method invocation. May be <code>null</code>.
     * @throws Throwable if this object wishes to abort the call.
     *                   Any exception thrown will be returned to the caller if it's
     *                   allowed by the method signature. Otherwise the exception
     *                   will be wrapped as a runtime exception.
     */
    void before(Method method, Object[] args, Object target) throws Throwable;

}
```

在 Spring 框架中，Advice 都是通过方法拦截器 MethodInterceptor 实现的。环绕 Advice 类似一个拦截器的链路，Before Advice、After advice等，不过暂时我们需要那么多就只定义了一个 MethodBeforeAdvice 的接口定义。


### 3.3、定义Advisor访问者

定义Advisor接口：
```java
public interface Advisor {
    /**
     * Return the advice part of this aspect. An advice may be an
     * interceptor, a before advice, a throws advice, etc.
     * @return the advice that should apply if the pointcut matches
     * @see org.aopalliance.intercept.MethodInterceptor
     * @see BeforeAdvice
     */
    Advice getAdvice();

}
```

定义PointCutAdvisor接口

```java
public interface PointCutAdvisor extends Advisor {

    /**
     * Get the Pointcut that drives this advisor.
     */
    Pointcut getPointcut();
}
```

PointcutAdvisor 承担了 Pointcut 和 Advice 的组合，Pointcut 用于获取 JoinPoint，而 Advice 决定于 JoinPoint 执行什么操作。

**AspectJExpressionPointcutAdvisor**
```java
public class AspectJExpressionPointcutAdvisor implements PointCutAdvisor {

    // 切面
    private AspectJExpressionPointcut pointcut;
    // 具体的拦截方法
    private Advice advice;
    // 表达式
    private String expression;

    public void setExpression(String expression){
        this.expression = expression;
    }

    @Override
    public Pointcut getPointcut() {
        if (null == pointcut) {
            pointcut = new AspectJExpressionPointcut(expression);
        }
        return pointcut;
    }

    @Override
    public Advice getAdvice() {
        return advice;
    }

    public void setAdvice(Advice advice){
        this.advice = advice;
    }

}
```
AspectJExpressionPointcutAdvisor 实现了 PointcutAdvisor 接口，把切面 pointcut、拦截方法 advice 和具体的拦截表达式包装在一起。这样就可以在 xml 的配置中定义一个 pointcutAdvisor 切面拦截器了。


### 3.4、方法拦截器实现

```java
public class MethodBeforeAdviceInterceptor implements MethodInterceptor {

    private MethodBeforeAdvice advice;

    public MethodBeforeAdviceInterceptor(){}

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        this.advice.before(invocation.getMethod(),invocation.getArguments(),invocation.getThis());
        return invocation.proceed();
    }
}
```
MethodBeforeAdviceInterceptor 实现了 MethodInterceptor 接口，在 invoke 方法中调用 advice 中的 before 方法，传入对应的参数信息。
而这个 advice.before 则是用于自己实现 MethodBeforeAdvice 接口后做的相应处理。其实可以看到具体的 MethodInterceptor 实现类，其实和我们之前做的测试是一样的，只不过现在交给了 Spring 来处理

### 3.5、代理工厂

```java
public class ProxyFactory {
    private AdvisedSupport advisedSupport;

    public ProxyFactory(AdvisedSupport advisedSupport) {
        this.advisedSupport = advisedSupport;
    }

    public Object getProxy() {
        return createAopProxy().getProxy();
    }

    private AopProxy createAopProxy() {
        if (advisedSupport.isProxyTargetClass()) {
            return new Cglib2AopProxy(advisedSupport);
        }

        return new JdkDynamicAopProxy(advisedSupport);
    }
}
```
其实这个代理工厂主要解决的是关于 JDK 和 Cglib 两种代理的选择问题，有了代理工厂就可以按照不同的创建需求进行控制。


### 3.6、融入到bean生命周期的自动代理创建者
```java
public class DefaultAdvisorAutoProxyCreator implements InstantiationAwareBeanPostProcessor, BeanFactoryAware {

    private DefaultListableBeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (DefaultListableBeanFactory)beanFactory;
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {

        if(isInfrastructureClass(beanClass)){
            return null;
        }

        Collection<AspectJExpressionPointcutAdvisor> advisors = beanFactory.getBeansOfType(AspectJExpressionPointcutAdvisor.class).values();

        for (AspectJExpressionPointcutAdvisor advisor : advisors) {
            ClassFilter classFilter = advisor.getPointcut().getClassFilter();
            if (!classFilter.matches(beanClass)) continue;

            AdvisedSupport advisedSupport = new AdvisedSupport();

            TargetSource targetSource = null;
            try {
                targetSource = new TargetSource(beanClass.getDeclaredConstructor().newInstance());
            } catch (Exception e) {
                e.printStackTrace();
            }
            advisedSupport.setTargetSource(targetSource);
            advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
            advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());
            advisedSupport.setProxyTargetClass(false);

            return new ProxyFactory(advisedSupport).getProxy();

        }

        return null;
    }

    private boolean isInfrastructureClass(Class<?> beanClass) {
        return Advice.class.isAssignableFrom(beanClass) || Pointcut.class.isAssignableFrom(beanClass) || Advisor.class.isAssignableFrom(beanClass);
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```
这个 DefaultAdvisorAutoProxyCreator 类的主要核心实现在于 postProcessBeforeInstantiation 方法中，从通过 beanFactory.getBeansOfType 获取 AspectJExpressionPointcutAdvisor 开始。
获取了 advisors 以后就可以遍历相应的 AspectJExpressionPointcutAdvisor 填充对应的属性信息，包括：目标对象、拦截方法、匹配器，之后返回代理对象即可。 
那么现在调用方获取到的这个 Bean 对象就是一个已经被切面注入的对象了，当调用方法的时候，则会被按需拦截，处理用户需要的信息。


### 3.7、融入bean的生命周期
```java
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
            // 判断是否返回代理 Bean 对象
            bean = resolveBeforeInstantiation(beanName, beanDefinition);
            if (null != bean) {
                return bean;
            }

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

    protected Object resolveBeforeInstantiation(String beanName, BeanDefinition beanDefinition) {
        Object bean = applyBeanPostProcessorsBeforeInstantiation(beanDefinition.getBeanClass(), beanName);
        if (null != bean) {
            bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
        }
        return bean;
    }

    protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
        for (BeanPostProcessor beanPostProcessor : getBeanPostProcessors()) {
            if (beanPostProcessor instanceof InstantiationAwareBeanPostProcessor) {
                Object result = ((InstantiationAwareBeanPostProcessor) beanPostProcessor).postProcessBeforeInstantiation(beanClass, beanName);
                if (null != result) return result;
            }
        }
        return null;
    }
```
因为创建的是代理对象不是之前流程里的普通对象，所以我们需要前置于其他对象的创建，即需要在 AbstractAutowireCapableBeanFactory#createBean 优先完成 Bean 对象的判断，是否需要代理，有则直接返回代理对象。


## 4、测试

### 4.1、测试用例

**Service对象**
```java
public class UserService implements IUserService{
    @Override
    public String queryUserInfo() {
        try {
            Thread.sleep(new Random(1).nextInt(100));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "bugcode，100001，深圳";
    }

    @Override
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

在Service中实现了两个方法，在拦截器中会实现对方法的拦截并且扩展功能。

**自定义拦截处理**
```java
public class UserServiceBeforeAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {

        System.out.println("拦截方法："+method.getName());
    }
}
```

与上一章节的拦截方法相比，我们不在是实现 MethodInterceptor 接口，而是实现 MethodBeforeAdvice 环绕拦截。在这个方法中我们可以获取到方法的一些信息，如果还开发了它的 MethodAfterAdvice 则可以两个接口一起实现。

**xml文件配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>


    <bean id="userService" class="bugcode.online.springframework.bean.UserService"/>

    <bean class="bugcode.online.springframework.aop.framework.autoProxy.DefaultAdvisorAutoProxyCreator"/>

    <bean id="beforeAdvice" class="bugcode.online.springframework.bean.UserServiceBeforeAdvice"/>

    <bean id="methodInterceptor" class="bugcode.online.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor">
        <property name="advice" ref="beforeAdvice"/>
    </bean>

    <bean id="pointcutAdvisor" class="bugcode.online.springframework.aop.aspectj.AspectJExpressionPointcutAdvisor">
        <property name="expression" value="execution(* bugcode.online.springframework.bean.IUserService.*(..))"/>
        <property name="advice" ref="methodInterceptor"/>
    </bean>
</beans>
```
在配置中，首先将Service，beforeAdvice,methodInterceptor对象交给spring容器管理，然后就可以在切面表达式中对service中的对象进行拦截扩展。


### 4.2、测试结果

```java
    @Test
    public void test_aop(){
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        IUserService userService = applicationContext.getBean("userService", IUserService.class);
        System.out.println("测试结果：" + userService.queryUserInfo());
    }
```

**结果**
```text
拦截方法：queryUserInfo
测试结果：bugcode，100001，深圳

Process finished with exit code 0
```