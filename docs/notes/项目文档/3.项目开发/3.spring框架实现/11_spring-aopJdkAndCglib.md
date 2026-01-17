---
title: 11、基于JDK，Cglib实现AOP切面
icon: file
order: 11
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
permalink: /project-docs/project/springframework/基于JDK，Cglib实现AOP切面/
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
- [11、基于JDK，Cglib实现AOP切面](#11基于jdkcglib实现aop切面)
    - [1、目标](#1目标)
    - [2、设计](#2设计)
    - [3、实现](#3实现)
        - [3.1、核心类图关系](#31核心类图关系)
        - [3.2、代理方法案例](#32代理方法案例)
        - [3.3、切点表达式](#33切点表达式)
        - [3.4、包装切面通](#34包装切面通)
        - [3.5、代理抽象实现(JDK&Cglib)](#35代理抽象实现jdkcglib)
        - [3.5、目标代理类](#35目标代理类)
    - [4、测试](#4测试)
        - [4.1、测试用例](#41测试用例)
        - [4.2、测试结果](#42测试结果)
    - [5、总结](#5总结)

<!-- /TOC -->
# 11、基于JDK，Cglib实现AOP切面

## 1、目标


**到本章节我们将要从 IOC 的实现，转入到关于 AOP(**Aspect Oriented Programming**) 内容的开发。在软件行业，AOP 意为：面向切面编程，通过预编译的方式和运行期间动态代理实现程序功能功能的统一维护。其实 AOP 也是 OOP 的延续，在 Spring 框架中是一个非常重要的内容，使用 AOP 可以对业务逻辑的各个部分进行隔离，从而使各模块间的业务逻辑耦合度降低，提高代码的可复用性，同时也能提高开发效率。**

**关于 AOP 的核心技术实现主要是动态代理的使用，就像你可以给一个接口的实现类，使用代理的方式替换掉这个实现类，使用代理类来处理你需要的逻辑。比如：**

```java
@Test
public void test_proxy_class() {
    IUserService userService = (IUserService) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class[]{IUserService.class}, (proxy, method, args) -> "你被代理了！");
    String result = userService.queryUserInfo();
    System.out.println("测试结果：" + result);
}
```


## 2、设计

**在把AOP切面融合到spring框架之前，需要解决两个问题，第一，如何给符合规则的方法做代理，其二，昨晚代理方法后，如何把类的职责拆分出来。这两个功能点的实现，都是以切面的思想设计开发，**

* **就像你在使用 Spring 的 AOP 一样，只处理一些需要被拦截的方法。在拦截方法后，执行你对方法的扩展操作。**
* **那么我们就需要先来实现一个可以代理方法的 Proxy，其实代理方法主要是使用到方法拦截器类处理方法的调用 **MethodInterceptor#invoke**，而不是直接使用 invoke 方法中的入参 Method method 进行 **method.invoke(targetObj, args)** 这块是整个使用时的差异。**
* **除了以上的核心功能实现，还需要使用到 org.aspectj.weaver.tools.PointcutParser 处理拦截表达式 "execution(\* cn.bugstack.springframework.test.bean.IUserService.\*(..))"，有了方法代理和处理拦截，我们就可以完成设计出一个 AOP 的雏形了**


![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/proxy.png)




## 3、实现

### 3.1、核心类图关系

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/eleven_aop_jdkAndCglib.webp)

* 整个类关系图就是 AOP 实现核心逻辑的地方，上面部分是关于方法的匹配实现，下面从 AopProxy 开始是关于方法的代理操作。
* AspectJExpressionPointcut 的核心功能主要依赖于 aspectj 组件并处理 Pointcut、ClassFilter,、MethodMatcher 接口实现，专门用于处理类和方法的匹配过滤操作。
* AopProxy 是代理的抽象对象，它的实现主要是基于 JDK 的代理和 Cglib 代理。在前面章节关于对象的实例化 CglibSubclassingInstantiationStrategy，我们也使用过 Cglib 提供的功能。

### 3.2、代理方法案例

```java
    @Test
    public void test_proxy_method(){

//        目标对象，可以代替成任何的目标对象
        Object targetObj = new UserService();

//        AOP代理 代理对象
        IUserService proxy = (IUserService)Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), targetObj.getClass().getInterfaces(), new InvocationHandler() {

//            方法匹配
            MethodMatcher methodMatcher = new AspectJExpressionPointcut("execution(* bugcode.online.springframework.bean.IUserService.*(..))");

            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

//                方法匹配
                if(methodMatcher.matches(method,targetObj.getClass())){
//                    方法拦截器 用户自己实现的拦截方法
                    MethodInterceptor methodInterceptor = invocation ->{
                        long start = System.currentTimeMillis();

                        try {
                            return invocation.proceed();
                        }finally {
                            System.out.println("监控 - Begin By AOP");
                            System.out.println("方法名称：" + invocation.getMethod().getName());
                            System.out.println("方法耗时：" + (System.currentTimeMillis() - start) + "ms");
                            System.out.println("监控 - End\r\n");
                        }
                    };
//                    反射调用
                    return methodInterceptor.invoke(new ReflectiveMethodInvocation(targetObj, method, args));
                }
                return method.invoke(targetObj, args);
            }
        });
```

**此案例中，给UserService对象做代理，对类中的每一个方法添加监控信息，MethodMatcher可以匹配到类中的所有方法，然后再代理类中匹配所有陪代理的方法，如果匹配到，就执行拦截器方法，拦截器方法里面主要是添加监控打印信息。**

### 3.3、切点表达式

```java
public interface Pointcut {

    /**
     * Return the ClassFilter for this pointcut.
     *
     * @return the ClassFilter (never <code>null</code>)
     */
    ClassFilter getClassFilter();

    /**
     * Return the MethodMatcher for this pointcut.
     *
     * @return the MethodMatcher (never <code>null</code>)
     */
    MethodMatcher getMethodMatcher();
}
```

**定义顶级的切点表达式接口， 用于获取 ClassFilter、MethodMatcher 的两个类，这两个接口获取都是切点表达式提供的内容 **

**ClassFilter接口**

```java
public interface ClassFilter {
    /**
     * Should the pointcut apply to the given interface or target class?
     * @param clazz the candidate target class
     * @return whether the advice should apply to the given target class
     */
    boolean matches(Class<?> clazz);

}
```

**定义类匹配类，用于切点找到给定的接口和目标类。**

**方法匹配**

```java
public interface MethodMatcher {
    /**
     * Perform static checking whether the given method matches. If this
     * @return whether or not this method matches statically
     */
    boolean matches(Method method, Class<?> targetClass);
}
```

**方法匹配，找到表达式范围内匹配下的目标类和方法。在上文的案例中有所体现：methodMatcher.matches(method, targetObj.getClass())**


**实现切入点表达式**

```java
public class AspectJExpressionPointcut implements Pointcut, ClassFilter, MethodMatcher {

    private static final Set<PointcutPrimitive> SUPPORTED_PRIMITIVES = new HashSet<PointcutPrimitive>();

    static {
        SUPPORTED_PRIMITIVES.add(PointcutPrimitive.EXECUTION);
    }

    private final PointcutExpression pointcutExpression;

    public AspectJExpressionPointcut(String expression){
        PointcutParser pointcutParser = PointcutParser.getPointcutParserSupportingSpecifiedPrimitivesAndUsingSpecifiedClassLoaderForResolution(SUPPORTED_PRIMITIVES, this.getClass().getClassLoader());
        pointcutExpression = pointcutParser.parsePointcutExpression(expression);
    }

    @Override
    public boolean matches(Class<?> clazz) {
        return pointcutExpression.couldMatchJoinPointsInType(clazz);
    }

    @Override
    public boolean matches(Method method, Class<?> targetClass) {
        return pointcutExpression.matchesMethodExecution(method).alwaysMatches();
    }

    @Override
    public ClassFilter getClassFilter() {
        return this;
    }

    @Override
    public MethodMatcher getMethodMatcher() {
        return this;
    }
}
```

* **切点表达式实现了 Pointcut、ClassFilter、MethodMatcher，三个接口定义方法，同时这个类主要是对 aspectj 包提供的表达式校验方法使用。**
* **匹配 matches：pointcutExpression.couldMatchJoinPointsInType(clazz)、pointcutExpression.matchesMethodExecution(method).alwaysMatches()，这部分内容可以单独测试验证。**

### 3.4、包装切面通

```java
public class AdvisedSupport {

//    被代理的目标对象
    private TargetSource targetSource;
//    方法拦截器
    private MethodInterceptor methodInterceptor;
//    方法匹配器（检查目标方法是否复合通知条件）
    private MethodMatcher methodMatcher;
}
```

* AdvisedSupport，主要是用于把代理、拦截、匹配的各项属性包装到一个类中，方便在 Proxy 实现类进行使用。***这和你的业务开发中包装入参是一个道理*
* **TargetSource，是一个目标对象，在目标对象类中提供 Object 入参属性，以及获取目标类 TargetClass 信息。**
* **MethodInterceptor，是一个具体拦截方法实现类，由用户自己实现 MethodInterceptor#invoke 方法，做具体的处理。***像我们本文的案例中是做方法监控处理*
* **MethodMatcher，是一个匹配方法的操作，这个对象由 AspectJExpressionPointcut 提供服务**

### 3.5、代理抽象实现(JDK&Cglib)

**定义代理接口**

```java
public interface AopProxy {
    Object getProxy();
}
```

**定义一个接口，用于获取代理类，因为具体的代理实现由JDK或者Cglib两种方式，所以使用接口更加具有扩展性。**

**JdkDynamicAopProxy动态代理**

```java
public class JdkDynamicAopProxy implements AopProxy, InvocationHandler {

    private final AdvisedSupport advised;

    public JdkDynamicAopProxy(AdvisedSupport advised) {
        this.advised = advised;
    }


    @Override
    public Object getProxy() {
        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), advised.getTargetSource().getTargetClass(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if(advised.getMethodMatcher().matches(method,advised.getTargetSource().getTarget().getClass())){
            MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
            return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(),method,args));
        }
        return method.invoke(advised.getTargetSource().getTarget(),args);
    }
}
```


* **基于 JDK 实现的代理类，需要实现接口 AopProxy、InvocationHandler，这样就可以把代理对象 getProxy 和反射调用方法 invoke 分开处理了。**
* **getProxy 方法中的是代理一个对象的操作，需要提供入参 ClassLoader、AdvisedSupport、和当前这个类 this，因为这个类提供了 invoke 方法。**
* **invoke 方法中主要处理匹配的方法后，使用用户自己提供的方法拦截实现，做反射调用 methodInterceptor.invoke 。**
* **这里还有一个 ReflectiveMethodInvocation，其他它就是一个入参的包装信息，提供了入参对象：目标对象、方法、入参。**

**Cglib动态代理**

```java
public class Cglib2AopProxy implements AopProxy{

    private final AdvisedSupport advised;

    public Cglib2AopProxy(AdvisedSupport advised){
        this.advised = advised;
    }

    @Override
    public Object getProxy() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(advised.getTargetSource().getTarget().getClass());
        enhancer.setInterfaces(advised.getTargetSource().getTargetClass());
        enhancer.setCallback(new DynamicAdvisedInterceptor(advised));
        return enhancer.create();
    }

    private static class DynamicAdvisedInterceptor implements MethodInterceptor{
        private final AdvisedSupport advised;

        public DynamicAdvisedInterceptor(AdvisedSupport advised) {
            this.advised = advised;
        }

        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            CglibMethodInvocation methodInvocation = new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, objects, methodProxy);
            if (advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
                return advised.getMethodInterceptor().invoke(methodInvocation);
            }
            return methodInvocation.proceed();
        }
    }

    private static class CglibMethodInvocation extends ReflectiveMethodInvocation {

        private final MethodProxy methodProxy;

        public CglibMethodInvocation(Object target, Method method, Object[] arguments, MethodProxy methodProxy) {
            super(target, method, arguments);
            this.methodProxy = methodProxy;
        }

        @Override
        public Object proceed() throws Throwable {
            return this.methodProxy.invoke(this.target, this.arguments);
        }

    }
}
```

* **基于 Cglib 使用 Enhancer 代理的类可以在运行期间为接口使用底层 ASM 字节码增强技术处理对象的代理对象生成，因此被代理类不需要实现任何接口。**
* 关于扩展进去的用户拦截方法，主要是在 Enhancer#setCallback 中处理，用户自己的新增的拦截处理。这里可以看到 DynamicAdvisedInterceptor#intercept 匹配方法后做了相应的反射操作

### 3.5、目标代理类

```java
public class TargetSource {

//    被代理的目标对象
    private final Object target;

    public TargetSource(Object target) {
        this.target = target;
    }

    /**
     * Return the type of targets returned by this {@link TargetSource}.
     * <p>Can return <code>null</code>, although certain usages of a
     * <code>TargetSource</code> might just work with a predetermined
     * target class.
     * @return the type of targets returned by this {@link TargetSource}
     */
    public Class<?>[] getTargetClass(){
        return this.target.getClass().getInterfaces();
    }

    /**
     * Return a target instance. Invoked immediately before the
     * AOP framework calls the "target" of an AOP method invocation.
     * @return the target object, which contains the joinpoint
     * @throws Exception if the target object can't be resolved
     */
    public Object getTarget(){
        return this.target;
    }
}
```

TargetSource目标代理UI喜爱那个包装类。

## 4、测试

### 4.1、测试用例

**定义Service接口**

```java
public interface IUserService {

    String queryUserInfo();

    String register(String userName);
}
```

**定义Srvice的实现类**

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

**自定义拦截器**

```java

public class UserServiceInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return invocation.proceed();
        } finally {
            System.out.println("监控 - Begin By AOP");
            System.out.println("方法名称：" + invocation.getMethod());
            System.out.println("方法耗时：" + (System.currentTimeMillis() - start) + "ms");
            System.out.println("监控 - End\r\n");
        }
    }
}
```

**用户自定义的拦截方法需要实现 MethodInterceptor 接口的 invoke 方法，使用方式与 Spring AOP 非常相似，也是包装 invocation.proceed() 放行，并在 finally 中添加监控信息  **

### 4.2、测试结果

```java
    @Test
    public void testPorxy(){

//        被代理的目标对象
        IUserService userService = new UserService();

//        组装代理信息
        AdvisedSupport advisedSupport = new AdvisedSupport();
//        设置被代理的目标对象
        advisedSupport.setTargetSource(new TargetSource(userService));
//        设置拦截器方法
        advisedSupport.setMethodInterceptor(new UserServiceInterceptor());
//        设置匹配方法的类
        advisedSupport.setMethodMatcher(new AspectJExpressionPointcut("execution(* bugcode.online.springframework.bean.IUserService.*(..))"));

//        获取代理对象
        IUserService proxy = (IUserService)new JdkDynamicAopProxy(advisedSupport).getProxy();
//        测试调用
        System.out.println(proxy.queryUserInfo());

//        代理对象（Cglib2AopProxy）
        IUserService proxy_cglib = (IUserService)new Cglib2AopProxy(advisedSupport).getProxy();
        System.out.println(proxy_cglib.register("bugcode"));

    }
```


**测试结果**

```java
监控 - Begin By AOP
方法名称：public abstract java.lang.String bugcode.online.springframework.bean.IUserService.queryUserInfo()
方法耗时：87ms
监控 - End

bugcode，100001，深圳
监控 - Begin By AOP
方法名称：public java.lang.String bugcode.online.springframework.bean.UserService.register(java.lang.String)
方法耗时：101ms
监控 - End

注册用户：bugcode success！
```


## 5、总结
