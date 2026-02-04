---
title: 2、Spring基础二
icon: devicon:spring-wordmark
order: 2
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2026/01/17 13:27:44
permalink: /compre-guide/spring/Spring基础二/
---




## Spring容器的类型，如BeanFactory和ApplicationContext。

Spring容器是用于管理和组装对象的核心组件。Spring提供了不同类型的容器，其中两个主要的容器是BeanFactory和ApplicationContext。

**BeanFactory:**

- **概念:** BeanFactory是Spring框架中最基本的容器接口。它负责管理Spring中的对象(称为Bean)的生命周期和依赖关系。
- **特点:** BeanFactory是一个轻量级的容器，延迟加载(懒加载)机制，即只有在需要使用Bean时才会进行实例化。
- **使用场景:** 对于资源有限的环境或需要按需加载Bean的场景，可以使用BeanFactory。

```java
// 示例代码
BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
MyService myService = (MyService) beanFactory.getBean("myService");
```

**ApplicationContext:**

- **概念:** ApplicationContext是BeanFactory的扩展，提供了更多的功能和特性。它是一个全功能的容器，负责管理Bean的生命周期、依赖注入、AOP等。
- **特点:** ApplicationContext在启动时就会将所有的Bean实例化，提供更快的访问速度，支持多种配置方式，包括XML配置、注解配置和Java配置等。
- **使用场景:** 在大多数场景中，推荐使用ApplicationContext，因为它提供了更多的功能和更方便的配置方式。

```java
// 示例代码
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
MyService myService = (MyService) applicationContext.getBean("myService");
```

总体而言，BeanFactory是一个基础的容器接口，而ApplicationContext是更高级和功能更全面的容器。在实际开发中，大多数情况下会选择使用ApplicationContext，因为它提供了更多的便利特性，并支持更灵活的配置方式。

## XML配置和注解配置的IoC容器设置。

在Spring中，IoC容器的配置方式主要包括XML配置和注解配置两种方式。这两种配置方式分别使用不同的元数据形式，用于描述Bean的定义、依赖关系、生命周期等信息。

### 1. XML配置方式

XML配置是一种传统的配置方式，通过XML文件描述应用程序的Bean定义和配置信息。以下是一个简单的XML配置示例:

```java
<!-- applicationContext.xml -->

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 定义一个Bean -->
    <bean id="myService" class="com.example.MyService">
        <!-- 设置属性 -->
        <property name="message" value="Hello, Spring!"/>
    </bean>

    <!-- 其他配置... -->

</beans>
```

### 2. 注解配置方式

注解配置是一种基于Java注解的配置方式，通过在Java类上使用注解来定义Bean和配置信息。以下是一个简单的注解配置示例:

```java
// AppConfig.java

@Configuration // 声明该类为配置类
public class AppConfig {

    @Bean // 定义一个Bean
    public MyService myService() {
        MyService service = new MyService();
        service.setMessage("Hello, Spring!");
        return service;
    }
    // 其他配置...
}
```

在上述示例中，@Configuration注解表示这是一个配置类，@Bean注解表示该方法返回的对象将被注册为一个Bean。

### 使用方式:

#### XML配置方式的容器获取:

```java
// 使用XML配置方式
BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
MyService myService = (MyService) beanFactory.getBean("myService");
```

#### 注解配置方式的容器获取:

```java
// 使用注解配置方式
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
MyService myService = (MyService) applicationContext.getBean(MyService.class);
```

**注意:** 在实际项目中，通常会使用更先进的Spring Boot来简化配置和提供更多功能。Spring Boot主要使用注解配置，且不需要显式的XML配置文件。

## 使用构造函数和Setter方法进行依赖注入。

在Spring中，依赖注入可以通过**构造函数注入和Setter方法注入**两种方式来实现。这两种方式都是为了将依赖关系传递给Bean。以下是使用构造函数和Setter方法进行依赖注入的示例:

### 1. 构造函数注入

通过构造函数注入是一种简单且常用的依赖注入方式。通过在类的构造函数中接收依赖对象，Spring容器在创建Bean的时候会自动将依赖对象传递进来。

```java
// ServiceA.java
public class ServiceA {
    private ServiceB serviceB;

    // 构造函数注入
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }

    // 其他方法...
}

// ServiceB.java
public class ServiceB {
    // 具体实现...
}
```

xml配置方式构造函数注入:

```xml
<!-- applicationContext.xml -->
<bean id="serviceA" class="com.example.ServiceA">
  <constructor-arg ref="serviceB"/> <!-- 通过构造函数注入ServiceB依赖 -->
</bean>

<bean id="serviceB" class="com.example.ServiceB"/>
```

### 2. Setter方法注入

通过Setter方法注入是另一种常见的依赖注入方式。通过在类中定义Setter方法，Spring容器在创建Bean后会调用这些方法，将依赖对象传递给Bean。

```java
// ServiceC.java
public class ServiceC {
    private ServiceD serviceD;

    // Setter方法注入
    public void setServiceD(ServiceD serviceD) {
        this.serviceD = serviceD;
    }

    // 其他方法...
}

// ServiceD.java
public class ServiceD {
    // 具体实现...
}
```

xml配置方式setter方法注入

```xml
<!-- applicationContext.xml -->
<bean id="serviceC" class="com.example.ServiceC">
  <property name="serviceD" ref="serviceD"/> <!-- 通过Setter方法注入ServiceD依赖 -->
</bean>

<bean id="serviceD" class="com.example.ServiceD"/>
```

### 使用方式

```java
// 使用构造函数注入
ServiceA serviceA = (ServiceA) applicationContext.getBean("serviceA");

// 使用Setter方法注入
ServiceC serviceC = (ServiceC) applicationContext.getBean("serviceC");
```

在上述示例中，ServiceA通过构造函数注入了ServiceB，而ServiceC通过Setter方法注入了ServiceD。Spring容器负责解析XML配置，创建Bean，并自动注入相关的依赖关系。选择使用构造函数注入还是Setter方法注入，通常取决于项目的具体需求和设计风格。

## 基于接口的依赖注入

在Spring中，**基于接口的依赖注入是通过在类中定义接口类型的成员变量，并通过构造函数或Setter方法将实现该接口的具体类注入到Bean中**。这种方式提高了代码的灵活性和可维护性，允许在运行时更容易替换实现类。

以下是一个基于接口的依赖注入的示例:

### 1. 接口定义:

```java
// Service.java
public interface Service {
    void execute();
}
```

### 2. 接口实现类:

```java
// ServiceImplA.java
public class ServiceImplA implements Service {
    @Override
    public void execute() {
        System.out.println("Executing ServiceImplA");
    }
}

// ServiceImplB.java
public class ServiceImplB implements Service {
    @Override
    public void execute() {
        System.out.println("Executing ServiceImplB");
    }
}
```

### 3. 使用接口的类:

```java
// Client.java
public class Client {
    // 接口
    private Service service;

    // 构造函数注入
    public Client(Service service) {
        this.service = service;
    }

    // Setter方法注入
    public void setService(Service service) {
        this.service = service;
    }

    public void performAction() {
        // 调用接口方法
        service.execute();
    }
}
```

### 4. XML配置:

```xml
<!-- applicationContext.xml -->

<bean id="serviceImplA" class="com.example.ServiceImplA"/>
<bean id="serviceImplB" class="com.example.ServiceImplB"/>

<bean id="client" class="com.example.Client">
  <constructor-arg ref="serviceImplA"/> <!-- 构造函数注入 ServiceImplA -->
  <!-- 或者使用 Setter 方法注入 -->
  <!-- <property name="service" ref="serviceImplA"/> -->
</bean>
```

### 5. 获取Bean:

```java
// 获取Client Bean
Client client = (Client) applicationContext.getBean("client");
client.performAction();
```

在上述示例中，Client类通过构造函数注入或Setter方法注入Service接口类型的依赖，通过XML配置文件将具体实现类ServiceImplA注入到Client中。这样，通过更换具体的实现类，可以轻松地改变Client的行为，实现了依赖注入的灵活性。

## 依赖注入(DI)

### DI的定义和优势。

**DI(Dependency Injection)依赖注入** 是一种设计模式，它通过将对象所需的依赖关系从对象本身分离出来，由外部容器(通常是框架或容器)来负责管理和注入。依赖注入的目的是解耦组件之间的依赖关系，提高代码的灵活性、可维护性和可测试性。

**依赖注入的定义:**

在依赖注入中，对象不再负责创建或查找它所依赖的对象，而是由外部容器负责将依赖关系注入到对象中。依赖注入有两种主要形式:

1. **构造函数注入(Constructor Injection):** 通过对象的构造函数将依赖关系传递给对象。
2. **Setter方法注入(Setter Injection):** 通过对象的Setter方法将依赖关系注入到对象。
3. 基于接口的依赖注入

**依赖注入的优势:**

1. **解耦和模块化:** 依赖注入将对象的依赖关系从对象本身解耦出来，使得对象的设计更加模块化。各个模块之间的依赖关系更加清晰，易于理解和维护。
2. **可测试性:** 由于依赖关系是外部容器注入的，因此在进行单元测试时，可以轻松地替换依赖的模拟对象，从而提高代码的可测试性。
3. **灵活性:** 依赖注入使得系统更加灵活，能够在运行时更容易地替换具体的实现类，而不需要修改依赖类的代码。
4. **可维护性:** 由于依赖关系集中在容器中管理，当需要修改依赖关系时，只需修改容器的配置而不影响依赖类的代码。这提高了代码的可维护性。
5. **可扩展性:** 通过依赖注入，系统的功能可以更容易地进行扩展。新增一个模块只需要配置容器，而不需要修改其他模块的代码。
6. **提高代码复用:** 通过依赖注入，依赖的对象可以被多个模块共享，从而提高代码的复用性。

### Spring中如何通过@Autowired和@Qualifier进行注入。

在Spring中，@Autowired和@Qualifier是两个常用的注解，用于实现自动装配(Autowired)时对依赖的指定(Qualifier)。

### 1. @Autowired注解:

@Autowired注解用于自动装配Spring容器中的Bean。它可以标记在构造函数、Setter方法、字段(属性)上，Spring容器会在需要注入依赖时自动查找匹配的Bean，并将其注入到目标对象中。

#### 1.1 构造函数注入:

```java
public class MyClass {
    private MyDependency myDependency;

    @Autowired
    public MyClass(MyDependency myDependency) {
        this.myDependency = myDependency;
    }

    // 其他方法...
}
```

#### 1.2 Setter方法注入:

```java
public class MyClass {
    private MyDependency myDependency;

    @Autowired
    public void setMyDependency(MyDependency myDependency) {
        this.myDependency = myDependency;
    }

    // 其他方法...
}
```

#### 1.3 字段注入:

```java
public class MyClass {
    @Autowired
    private MyDependency myDependency;

    // 其他方法...
}
```

### 2. @Qualifier注解:

@Qualifier注解用于在存在多个匹配的Bean时，指定具体要注入的Bean。它通常与@Autowired一起使用。

```java
public class MyClass {
    private MyDependency myDependency;

    @Autowired
    @Qualifier("specificBeanName") // 指定要注入的Bean的名称
    public void setMyDependency(MyDependency myDependency) {
        this.myDependency = myDependency;
    }

    // 其他方法...
}
```

在上述示例中，@Autowired注解用于告诉Spring容器要自动注入MyDependency类型的Bean。而@Qualifier("specificBeanName")则指定了要注入的具体Bean的名称。

如果没有使用@Qualifier，且存在多个匹配的Bean时，Spring会抛出异常。使用@Qualifier可以明确指定要注入的Bean，避免歧义。

> **注意:** 在使用@Autowired和@Qualifier时，通常建议将@Autowired放在构造函数或Setter方法上，然后结合@Qualifier使用。这样可以提高代码的可读性和一致性。

### 3 使用@Value进行属性注入。

在Spring中，@Value注解用于通过**配置文件**注入属性值或直接注入常量值。这个注解可以用在字段、方法参数、构造函数参数上，以便在运行时将值注入到标记的位置。

### 1. 注入属性值:

#### 1.1 注入配置文件中的属性值:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyComponent {

    @Value("${my.property.key}")
    private String myPropertyValue;

    // 其他方法...
}
```

在上述示例中，`@Value("${my.property.key}")`用于从配置文件中读取名为`my.property.key`的属性值并注入到myPropertyValue字段中。

#### 1.2 直接注入常量值:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyComponent {

    @Value("Hello, Spring!")
    private String myConstantValue;

    // 其他方法...
}
```

在这个例子中，`@Value("Hello, Spring!")`直接将常量字符串 "Hello, Spring!" 注入到myConstantValue字段中。

### 2. 注入方法参数:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyComponent {

    private String myPropertyValue;

    // 注入到方法参数
    public void setMyPropertyValue(@Value("${my.property.key}") String myPropertyValue) {
        this.myPropertyValue = myPropertyValue;
    }

    // 其他方法...
}
```

在这个例子中，`@Value("${my.property.key}")`被用于注入到setMyPropertyValue方法的参数中。

### 3. 注入构造函数参数:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyComponent {

    private final String myPropertyValue;

    // 构造函数注入
    public MyComponent(@Value("${my.property.key}") String myPropertyValue) {
        this.myPropertyValue = myPropertyValue;
    }

    // 其他方法...
}
```

在这个例子中，`@Value("${my.property.key}")`用于注入到构造函数的参数中。

**注意:** 使用@Value注解时，确保配置文件中存在对应的属性键，并且在类上使用@Component或其他适当的注解，以便Spring容器能够扫描并创建该类的Bean。

### 4 使用Java配置类进行依赖注入。

在Spring中，可以使用Java配置类(JavaConfig)来进行依赖注入，取代传统的XML配置方式。通过Java配置，可以更灵活地配置Bean及其依赖关系，并且可以利用Java的编译时类型检查。

以下是一个简单的Java配置类的示例:

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }

    @Bean
    public MyComponent myComponent(MyService myService) {
        return new MyComponent(myService);
    }
}
```

在这个示例中，@Configuration注解表明这是一个Java配置类。@Bean注解用于定义Bean，方法的名称即为Bean的名称。MyService和MyComponent是两个简单的类，myComponent方法中的myService参数表示对MyService的依赖。

使用这个配置类，可以通过AnnotationConfigApplicationContext来加载配置:

```java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        MyComponent myComponent = context.getBean(MyComponent.class);
        myComponent.doSomething();

        context.close();
    }
}
```

这样，Spring容器会根据Java配置类中的定义，创建相应的Bean，并自动解决Bean之间的依赖关系。

此外，Java配置类还可以使用@Import注解导入其他配置类，实现更好的模块化。

```java
@Configuration
@Import({DatabaseConfig.class, ServiceConfig.class})
public class AppConfig {

    // Bean definitions...
}
```

这样，可以将不同模块的配置分开，提高配置的可维护性。

### 5 使用Java泛型注入集合。

在Spring中，可以使用泛型来进行集合的依赖注入。下面是一个简单的示例，演示如何使用Java泛型注入集合:

```java
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class MyComponent<T> {

    private List<T> myCollection;

    public MyComponent(List<T> myCollection) {
        this.myCollection = myCollection;
    }

    public void displayCollection() {
        System.out.println("Elements in the collection:");
        for (T element : myCollection) {
            System.out.println(element);
        }
    }
}
```

在这个示例中，MyComponent类是一个泛型类，它接收一个`List<T>`类型的集合作为构造函数的参数。通过这种方式，可以实现对不同类型的集合进行注入。

然后，可以通过使用`@Autowired`注解和`@Qualifier`注解来将具体类型的集合注入到MyComponent中:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class AppService {

    @Autowired
    @Qualifier("stringList")
    private MyComponent<String> stringComponent;

    @Autowired
    @Qualifier("integerList")
    private MyComponent<Integer> integerComponent;

    public void displayCollections() {
        stringComponent.displayCollection();
        integerComponent.displayCollection();
    }
}
```

在上述示例中，AppService类中分别注入了两个MyComponent对象，分别对应String类型和Integer类型的集合。使用@Qualifier注解指定具体的集合Bean名称，确保正确的集合被注入到对应的MyComponent中。

接下来，需要在配置类中定义这两个集合Bean:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;
import java.util.List;

@Configuration
public class AppConfig {

    @Bean("stringList")
    public List<String> stringList() {
        return Arrays.asList("One", "Two", "Three");
    }

    @Bean("integerList")
    public List<Integer> integerList() {
        return Arrays.asList(1, 2, 3);
    }
}
```

在这个配置类中，通过@Bean注解定义了两个不同类型的集合Bean，并使用不同的Bean名称。

最后，在启动应用程序时，通过AppService对象调用displayCollections方法，可以看到输出结果中包含了两个集合的元素。

这种方式利用了Java泛型的灵活性，使得可以轻松实现对不同类型集合的注入。



## Spring框架中有哪些不同类型的事件

- Spring 提供了以下5种标准的事件:

1. 上下文更新事件(ContextRefreshedEvent):在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。
2. 上下文开始事件(ContextStartedEvent):当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。
3. 上下文停止事件(ContextStoppedEvent):当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。
4. 上下文关闭事件(ContextClosedEvent):当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
5. 请求处理事件(RequestHandledEvent):在Web应用中，当一个http请求(request)结束触发该事件。如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。



## Spring注解

### 什么是基于Java的Spring注解配置? 给一些注解的例子

- 基于Java的配置，允许你在少量的Java注解的帮助下，进行你的大部分Spring配置而非通过XML文件。
- 以@Configuration 注解为例，它用来标记类可以当做一个bean的定义，被Spring IOC容器使用。
- 另一个例子是@Bean注解，它表示此方法将要返回一个对象，作为一个bean注册进Spring应用上下文。

```java
@Configuration
public class StudentConfig {
    @Bean
    public StudentBean myStudent() {
        return new StudentBean();
    }
}
```

### 怎样开启注解装配？

注解装配在默认情况下是不开启的，为了使用注解装配，我们必须在Spring配置文件中配置 <context:annotation-config/>元素。

### @Component, @Controller, @Repository, @Service 有何区别？

- @Component:这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。
- @Controller:这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IOC 容器中。
- @Service:此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。
- @Repository:这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IOC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

### @Required 注解有什么作用

这个注解表明bean的属性必须在配置的时候设置，通过一个bean定义的显式的属性值或通过自动装配，若@Required注解的bean属性未被设置，容器将抛出BeanInitializationException。示例:

```java
public class Employee {

    private String name;
    
    @Required
    public void setName(String name){
        this.name=name;
    }
    public string getName(){
        return name;
    }
}
```

### @Autowired 注解有什么作用

@Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在(可以设置它required属性为false)。@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和@Required一样，修饰setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

```java
public class Employee {

    private String name;
    
    @Autowired
    public void setName(String name) {
        this.name=name;
    }
    public string getName(){
        return name;
    }
}
```

### @Autowired和@Resource之间的区别

- @Autowired和@Resource可用于:构造函数、成员变量、Setter方法
- @Autowired和@Resource之间的区别在于
    - @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在(可以设置它required属性为false)。
    - @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

### @Qualifier 注解有什么作用

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

### @RequestMapping 注解有什么用？

- @RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别:
    - 类级别:映射请求的 URL
    - 方法级别:映射 URL 以及 HTTP 请求方法

## Spring数据访问

### 解释对象/关系映射集成模块

- Spring 通过提供ORM模块，支持我们在直接JDBC之上使用一个对象/关系映射映射(ORM)工具，Spring 支持集成主流的ORM框架，如Hiberate，JDO和 MyBatis，JPA，TopLink，JDO，OJB等待 。Spring的事务管理同样支持以上所有ORM框架及JDBC。

### 在Spring框架中如何更有效地使用JDBC？

- 使用Spring JDBC 框架，资源管理和错误处理的代价都会被减轻。所以开发者只需写statements 和 queries从数据存取数据，JDBC也可以在Spring框架提供的模板类的帮助下更有效地被使用，这个模板叫JdbcTemplate

### 解释JDBC抽象和DAO模块

- 通过使用JDBC抽象和DAO模块，保证数据库代码的简洁，并能避免数据库资源错误关闭导致的问题，它在各种不同的数据库的错误信息之上，提供了一个统一的异常访问层。它还利用Spring的AOP 模块给Spring应用中的对象提供事务管理服务。

### spring DAO 有什么用？

- Spring DAO(数据访问对象) 使得 JDBC，Hibernate 或 JDO 这样的数据访问技术更容易以一种统一的方式工作。这使得用户容易在持久性技术之间切换。它还允许您在编写代码时，无需考虑捕获每种技术不同的异常。

### spring JDBC API 中存在哪些类？

- JdbcTemplate
- SimpleJdbcTemplate
- NamedParameterJdbcTemplate
- SimpleJdbcInsert
- SimpleJdbcCall

### JdbcTemplate是什么

- JdbcTemplate 类提供了很多便利的方法解决诸如把数据库数据转变成基本数据类型或对象，执行写好的或可调用的数据库操作语句，提供自定义的数据错误处理。

### 使用Spring通过什么方式访问Hibernate？使用 Spring 访问 Hibernate 的方法有哪些？

- 在Spring中有两种方式访问Hibernate:

    - 使用 Hibernate 模板和回调进行控制反转
    - 扩展 HibernateDAOSupport 并应用 AOP 拦截器节点

### 如何通过HibernateDaoSupport将Spring和Hibernate结合起来？

- 用Spring的 SessionFactory 调用 LocalSessionFactory。集成过程分三步:

    - 配置the Hibernate SessionFactory
    - 继承HibernateDaoSupport实现一个DAO
    - 在AOP支持的事务中装配

### Spring支持的事务管理类型， spring 事务实现方式有哪些？

- Spring支持两种类型的事务管理:

    - **编程式事务管理**:这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。
    - **声明式事务管理**:这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务。

### Spring事务的实现方式和实现原理

- Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

### 说一下Spring的事务传播行为

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

1. PROPAGATION_REQUIRED:如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
2. PROPAGATION_SUPPORTS:支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
3. PROPAGATION_MANDATORY:支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
4. PROPAGATION_REQUIRES_NEW:创建新事务，无论当前存不存在事务，都创建新事务。
5. PROPAGATION_NOT_SUPPORTED:以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. PROPAGATION_NEVER:以非事务方式执行，如果当前存在事务，则抛出异常。
7. PROPAGATION_NESTED:如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

### 说一下 spring 的事务隔离？

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT(使用数据库的设置)，其他四个隔离级别和数据库的隔离级别一致:

1. ISOLATION_DEFAULT:用底层数据库的设置隔离级别，数据库设置的是什么我就用什么;
2. ISOLATION_READ_UNCOMMITTED:未提交读，最低隔离级别、事务未提交前，就可被其他事务读取(会出现幻读、脏读、不可重复读);
3. ISOLATION_READ_COMMITTED:提交读，一个事务提交后才能被其他事务读取到(会造成幻读、不可重复读)，SQL server 的默认级别;
4. ISOLATION_REPEATABLE_READ:可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据(会造成幻读)，MySQL 的默认级别;
5. ISOLATION_SERIALIZABLE:序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

**不同隔离级别带来的影响**

- **脏读** :表示一个事务能够读取另一个事务中还未提交的数据。比如，某个事务尝试插入记录 A，此时该事务还未提交，然后另一个事务尝试读取到了记录 A。
- **不可重复读** :是指在一个事务内，多次读同一数据。
- **幻读** :指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。

### Spring框架的事务管理有哪些优点？

- 为不同的事务API 如 JTA，JDBC，Hibernate，JPA 和JDO，提供一个不变的编程模式。
- 为编程式事务管理提供了一套简单的API而不是一些复杂的事务API
- 支持声明式事务管理。
- 和Spring各种数据访问抽象层很好得集成。

### 你更倾向用那种事务管理类型？

- 大多数Spring框架的用户选择声明式事务管理，因为它对应用代码的影响最小，因此更符合一个无侵入的轻量级容器的思想。声明式事务管理要优于编程式事务管理，虽然比编程式事务管理(这种方式允许你通过代码控制事务)少了一点灵活性。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

## Spring面向切面编程(AOP)

### 什么是AOP

- OOP(Object-Oriented Programming)面向对象编程，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。
- AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”(Aspect)，减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。

### Spring AOP and AspectJ AOP 有什么区别？AOP 有哪些实现方式？

- AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ;动态代理则以Spring AOP为代表。

    - (1)AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。
    - (2)Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

### JDK动态代理和CGLIB动态代理的区别

- Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理:

    - JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起;接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。
    - 如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB(Code Generation Library)，是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

- 静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

InvocationHandler 的 invoke(Object proxy,Method method,Object[] args):proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。

### 解释一下Spring AOP里面的几个名词

(1)切面(Aspect):切面是通知和切点的结合。通知和切点共同定义了切面的全部内容。 在Spring AOP中，切面可以使用通用类(基于模式的风格) 或者在普通类中以 @AspectJ 注解来实现。

(2)连接点(Join point):指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 应用可能有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

(3)通知(Advice):在AOP术语中，切面的工作被称为通知。

(4)切入点(Pointcut):切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

(5)引入(Introduction):引入允许我们向现有类添加新方法或属性。

(6)目标对象(Target Object): 被一个或者多个切面(aspect)所通知(advise)的对象。它通常是一个代理对象。也有人把它叫做 被通知(adviced) 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理(proxied) 对象。

(7)织入(Weaving):织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多少个点可以进行织入:

编译期:切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。

类加载期:切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面。

运行期:切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。

### Spring在运行时通知对象

- 通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。代理封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。
- 直到应用需要被代理的bean时，Spring才创建代理对象。如果使用的是ApplicationContext的话，在ApplicationContext从BeanFactory中加载所有bean的时候，Spring才会创建被代理的对象。因为Spring运行时才创建代理对象，所以我们不需要特殊的编译器来织入SpringAOP的切面。

### Spring只支持方法级别的连接点

- 因为Spring基于动态代理，所以Spring只支持方法连接点。Spring缺少对字段连接点的支持，而且它不支持构造器连接点。方法之外的连接点拦截功能，我们可以利用Aspect来补充。

### 在Spring AOP 中，关注点和横切关注的区别是什么？在 spring aop 中 concern 和 cross-cutting concern 的不同之处

- 关注点(concern)是应用中一个模块的行为，一个关注点可能会被定义成一个我们想实现的一个功能。
- 横切关注点(cross-cutting concern)是一个关注点，此关注点是整个应用都会使用的功能，并影响整个应用，比如日志，安全和数据传输，几乎应用的每个模块都需要的功能。因此这些都属于横切关注点。

### Spring通知有哪些类型？

在AOP术语中，切面的工作被称为通知，实际上是程序执行时要通过SpringAOP框架触发的代码段。

Spring切面可以应用5种类型的通知:

1. 前置通知(Before):在目标方法被调用之前调用通知功能;
2. 后置通知(After):在目标方法完成之后调用通知，此时不会关心方法的输出是什么;
   3.返回通知(After-returning ):在目标方法成功执行之后调用通知;
4. 异常通知(After-throwing):在目标方法抛出异常后调用通知;
5. 环绕通知(Around):通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

### 什么是切面 Aspect？

- aspect 由 pointcount 和 advice 组成，切面是通知和切点的结合。 它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP 就是负责实施切面的框架, 它将切面所定义的横切逻辑编织到切面所指定的连接点中. AOP 的工作重心在于如何将增强编织目标对象的连接点上, 这里包含两个工作:

    - 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
    - 如何在 advice 中编写切面代码.

- 可以简单地认为, 使用 @Aspect 注解的类就是切面.

![img](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1716381268312-3adba3a9-ea2b-4a50-a0ef-7ae91bdeca2d.webp)


### 解释基于注解的切面实现

- 在这种情况下(基于@AspectJ的实现)，涉及到的切面声明的风格与带有java5标注的普通java类一致。