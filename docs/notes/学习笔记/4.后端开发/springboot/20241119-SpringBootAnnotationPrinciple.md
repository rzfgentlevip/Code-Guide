---
title: Spring注解驱动原理
icon: streamline-freehand-color:notes-quill
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - SPRINGBOOT
tag:
  - springboot
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/yyb083wr/
---


# Spring注解驱动原理

# 大图

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.png)

导图：

![https://cdn.nlark.com/yuque/0/2021/jpeg/12833864/1618280608393-2d8e0e4c-d9b5-44e9-b3ca-ece332352d70.jpeg](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1618280608393-2d8e0e4c-d9b5-44e9-b3ca-ece332352d70.jpeg)

# 1、向容器中注入实例对象

容器重点：

- 控制反转
- 依赖注入

## 1.1 在pom文件中加入spring-context依赖

### 1.1.1、在pom文件中加入spring-context依赖

核心容器依赖的环境：

```xml
-- spring核心依赖
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>

-- lambok插件
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.20</version>
</dependency>

-- junit测试
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%201.png)

### 1.1.2、定义一个实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;
}
```

### 1.1.3、在spring-config.xml配置文件中通过<bean></bean>标签注入类实例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

<!--    只要标注了包扫描，@Controller,@Service,@Repository,@Component任何一个注解，那么ioc容器在启动时候，就会自动扫描添加到容器中
        xml配置需要进行这些操作，现在注解需要在配置类上写

<context:component-scan base-package="com.rzf.annotation.bean"></context:component-scan>
-->
--xml方式注入bean对象
        <bean id="person" class="com.rzf.annotation.bean.Person">-->
            <property name="name" ref="zhangsan"></property>-->
            <property name="age" ref="18"></property>-->
        </bean>
</beans>
```

1. 获取容器中通过配置文件注入的实例对象

```java
public class Main {
    public static void main(String[] args) {
        // 读取通过配置文件，创建ioc容器，并向容器中注入实例对象
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        // 根据bean的id获取容器中注入的实例对象
        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
    }
}
```

## 1.2 注解方式

通过注解方式，不需要在配置xml配置文件，而是新创建一个MyConfig.java配置类，在配置类中通过注解的方式向容器中注入实例对象。

> 配置类就等于配置文件，相当于换了一种写法而已。

1. 不再配置xml文件，而是新建一个配置类MyConfig.java，在配置类文件中通过注解向容器中注入实例对象。

```java
======配置类 等价于 配置文件=======
// 告诉Spring这是一个配置类
@Configuration
public class MyConfig {
    ===== 给容器中注册一个Bean，类型为返回值的类型，注入的Bean的id默认是用方法名作为id=====
//组件类型是方法的返回值类型，组件的名字默认是方法的名字，可以在@bean()中指定组件的名字
    @Bean
    public Person person01(){
        return new Person("lisi",20);
    }
    ===== 如果需要自定义id的值，只需在@Bean()的参数部分设置就行。=====
    @Bean(person01)
    public Person person(){
        return new Person("lisi",20);
    }
}
```

1. 获取容器中通过配置类注入的实例对象

```java
public class Main {
    public static void main(String[] args) {
        // 读取通过配置类，创建ioc容器，并向容器中注入实例对象
        ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig.class);
        Person person1 = (Person) annotationConfigApplicationContext.getBean("person01");
        System.out.println(person1);
    }
}
```

创建容器并注入对象原理

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616834284020-41336ae8-afb5-4f4a-88f4-79715268a578.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616834284020-41336ae8-afb5-4f4a-88f4-79715268a578.png)

**核心源码：**

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616834199541-c619302a-4de9-402a-b09c-756267f3227d.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616834199541-c619302a-4de9-402a-b09c-756267f3227d.png)

# 2 包扫描

## 2.1 配置文件中配置包扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-4.0.xsd ">

    <!-- 配置包扫描 -->
    <context:component-scan base-package="com.rzf.annotation"/>
    <!-- 注入实例对象 -->
    <bean id="person01" class="com.example.bean.Person">
        <property name="name" value="bob"/>
        <property name="age" value="18"/>
    </bean>
</beans>
```

## 2.2 注解配置包扫描

通过xml文件方式注入组件，只`*要标注了@Controller、@Service、@Repository，@Component`等注解，创建ioc容器的时候，会自动扫描这些注解并注入组件到容器中，也可以通过配置包扫描实现组件注入。*

```java
<context:component-scan base-package="com.rzf.annotation" use-default-filters="false"></context:component-scan>
```

通过注解方式注入组件，需要在配置类上面指定包扫描路径，这样容器启动时候就会自动扫描并且加载组件，直接在配置类的头上加@ComponentScan(value = "com.example")注解，参数部分指定要扫描的包目录。

```java
//xml配置文件 = 配置类
@Configuration
//指定包扫描
@ComponentScan(value = "com.rzf.annotation") //指定扫描的包路径
public class MainConfig {

//    给容器注册一个bean,类型为返回值类型，id默认使用方法名
//    标注的bean注解，ioc容器在启动时候会执行该方法，将该方法的返回值类型作为组件类型，方法名作为组件的名字放在容器中
    @Bean(value = "person01") // 修改bean在容器中的名字
    public Person person01(){

        return new Person("lisi",20);
    }
}
```

**为了查看和验证包扫描的内容，在指定的包扫描目录下，添加controller、service、serviceImpl、dao目录，并分别存放各自的内容。**

```java
@Controller
public class BookController {
}
public interface BookService {
}

@Service
public class BookServiceImpl implements BookService {
}

@Repository
public class BookDao {
}
```

在main函数中，获取当前容器中的所有注入的bean

```java
@Test
    public void test3(){
// 读取通过注解注入容器中的实例对象
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class);
//        查看容器定义了那些bean
        String[] names = context.getBeanDefinitionNames();
        for(String name: names){
            System.out.println(name);
        }
    }

========= spring启动需要的bean=======
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
=======用户操作后，注入到容器中的bean=====
myConfig
bookController
bookDao
bookServiceImpl
person
```

### 2.2.1 扫描或不扫描指定的包

通过参数excludeFilters指定排除的bean

```java
@Configuration
=== 包扫描时，排除掉标有指定注解的类(bean) ===
@ComponentScan(value = "com.rzf.annotation",excludeFilters = {
//指定排除不注入Controller组件和Service组件
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Service.class})
})
public class MyConfig {}

========= spring启动需要的bean=======
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
=======用户操作后，注入到容器中的bean=====
myConfig
bookDao
person
====== 包扫描的过滤规则生效 ======
```

通过参数includeFilters和useDefaultFilters = false(禁用掉默认的扫描规则)指定要扫描的bean

```java
@ComponentScan(value = "com.rzf.annotation",includeFilters =  {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class})},useDefaultFilters = false)
public class MyConfig {}

========= spring启动需要的bean=======
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
=======用户操作后，注入到容器中的bean=====
myConfig
bookController
person
====== 包扫描的过滤规则生效 ======
```

按实体类型过滤、按注解类型过滤

```java
@ComponentScan(value = "com.example",includeFilters =  {
    	=== 按注解类型过滤--识别@Controller注解
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class}),
    	=== 按实体类的类型过滤-- 识别BookServiceImpl的类和所有子类
        @ComponentScan.Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookServiceImpl.class})
},useDefaultFilters = false)
public class MyConfig {}
```

- @ComponentScan value:指定要扫描的包
- excludeFilters = Filter[]：指定扫描的时候按照什么规则排除那些组件
- includeFilters = Filter[]：指定扫描的时候只需要包含哪些组件
    - FilterType.ANNOTATION：按照注解,BIRU cONTROLLER
    - FilterType.ASSIGNABLE_TYPE：按照给定的类型；
    - FilterType.ASPECTJ：使用ASPECTJ表达式
    - FilterType.REGEX：使用正则指定
    - FilterType.CUSTOM：使用自定义规则 使用最多

### 2.2.2、自定义过滤

按自定义过滤类型的源码提示：

```java
	/** Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 */
	CUSTOM
    === 自定义的规则必须是TypeFilter的实现类 ===
```

实现一个自定义的过滤类型，如果自定义的类型返回一个false，即说明没有匹配到任何一个类型，就不会注入任何Bean(组件)。

```java
//自定义过滤规则
public class MyTypeFilter implements TypeFilter {
    /**
     *
     * @param metadataReader 读取到的当前正在扫描的类信息
     * @param metadataReaderFactory 可以获取其他类信息
     * @return
     * @throws IOException
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
//        获取当前类注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
//        获取当前正在扫描的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
//        获取当前类资源信息，比如位置信息
        Resource resource = metadataReader.getResource();
        String className = classMetadata.getClassName();
        System.out.println("------->"+className);

        if(className.contains("er")){
            return true;
        }
        return false;
    }
}
```

在配置类中指定使用自定义的过滤规则

```java
//xml配置文件 = 配置类
@Configuration
//指定包扫描
@ComponentScan(value = "com.rzf.annotation",
//排除Controller注解
includeFilters = {
//        按照注解进行包含：
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class),
//        按照类型进行包含：只要是BookService组件类型，都都会添加在容器中
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = BookService.class),
        @ComponentScan.Filter(type = FilterType.CUSTOM,classes = MyTypeFilter.class)
},
//        禁用默认规则，也就是不全部包扫描，这样其他包就不会自动扫描，只会赛秒MainConfig类里面的组件
        useDefaultFilters = false
)
public class MainConfig {

//    给容器注册一个bean,类型为返回值类型，id默认使用方法名
//    标注的bean注解，ioc容器在启动时候会执行该方法，将该方法的返回值类型作为组件类型，方法名作为组件的名字放在容器中
    @Bean(value = "person01") // 修改bean在容器中的名字
    public Person person01(){

        return new Person("lisi",20);
    }
}

myConfig
person
bookController  -- 带er的类
myTypeFilter    -- 带er的类
bookServiceImpl -- 带er的类
```

> 在默认情况下是不会扫描没有注解的类的,自定义的时候就会扫描根目录下的所有包内的类

## 2.3 @Scope注解

@Scope()的取值：

- @see ConfigurableBeanFactory#SCOPE_PROTOTYPE  prototype 多实例
- @see ConfigurableBeanFactory#SCOPE_SINGLETON  singleton 单实例(默认值)
- @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST  request 同一次请求创建一个实例
- @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION  session 同一个session创建一个实例

其实就等同于在xml配置文件中<bean></bean>标签中的scope能取的四个值。

### 2.3.1 单实例

```java
@Configuration
public class MyConfig {
    @Scope("singleton")
    @Bean
    public Person person(){
        System.out.println("单实例模式下，开始向容器中添加Person组件...");
        return new Person("zhangsan",34);
    }
}
```

```java
@Test
    public void test03(){
       	// 通过读取配置类，创建ioc容器，并向容器中注入实例对象
        ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        Person person1 = (Person) annotationConfigApplicationContext.getBean("person");
        Person person2 = (Person) annotationConfigApplicationContext.getBean("person");
        System.out.println(person1==person2);
        System.out.println("===========");
        === 输出结果为true，也就是说两次获取的组件是一样的。
    }
```

**在单实例状态下，将Person实例添加到容器的时机：**

读取配置类创建ioc容器时就调用创建对象的方法创建Person对象，并将创建的对象person放入到容器中，以后每次使用对象都直接从容器中获取(从map中get对象的过程)。

在多实例情况下，加载ioc容器的时候不会创建bean对象，而是在读取的时候加载组件到容器中。

### 2.3.2 懒加载

**单实例bean，默认在容器创建启动的时候就会创建对象**。但是在懒加载模式下，容器启动的时候不会创建对象，而是在第一次使用(获取)Bean的时候才会创建并初始化。

```java
@Configuration
public class MyConfig {
    @Scope("singleton")
    @Lazy
    @Bean
    public Person person(){
        System.out.println("单实例+懒加载 模式下，开始向容器中添加Person组件...");
        return new Person("zhangsan",34);
    }
}
```

### 2.3.3 多实例

```java
@Configuration
public class MyConfig {
    @Scope("prototype")
    @Bean
    public Person person(){
        System.out.println("多实例模式下，开始向容器中添加Person组件...");
        return new Person("zhangsan",34);
    }
}
```

```java
@Test
    public void test03(){
        // 读取通过注解注入容器中的实例对象
        ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        Person person1 = (Person) annotationConfigApplicationContext.getBean("person");
        Person person2 = (Person) annotationConfigApplicationContext.getBean("person");
        System.out.println(person1==person2);
        System.out.println("===========");
        === 输出结果为false，也就是说两次获取的对象组件是不一样的。
    }
```

**在多实例状态下，将Person实例添加到容器的时机：**

读取配置类创建ioc容器时并不会创建对象，而是在每次获取对象实例的时候才会调用创建对象的方法来创建Person对象，而且，每次获取对象时，都会重新调用一次创建对象的方法，进而使得每次获取的对象不同。

- prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。每次获取的时候才会调用方法创建对象；
- singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中以后每次获取就是直接从容器（map.get()）中拿，
- request：同一次请求创建一个实例
- session：同一个session创建一个实例

## 2.4 @Conditional注解

按照一定的条件进行判断，满足条件时才给容器中注册bean

源码：

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616845864966-bc917856-6c41-4474-8904-ccbc30b71b56.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616845864966-bc917856-6c41-4474-8904-ccbc30b71b56.png)

【案例】根据不同的操作系统注入不同的组件

补充知识点：根据条件上下文对象获取上下文环境

```java
//必须实现Condition 条件接口
public class WindowsCondition implements Condition {

    /**
     *
     * @param context 能使用的上下文环境
     * @param metadata 注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
//        TODO 判断系统是win系统
//        1 能获取到ioc使用的beanFactory
        ConfigurableListableBeanFactory factory = context.getBeanFactory();
//        2 获取类加载器
        ClassLoader classLoader = context.getClassLoader();
//        3 获取运行环境
        Environment environment = context.getEnvironment();
//        获取bean定义的注册类，可以查看有没有bean的定义，或者注册bean定义
        BeanDefinitionRegistry registry = context.getRegistry();

        boolean person03 = registry.containsBeanDefinition("person03");
        System.out.println(person03);
        String property = environment.getProperty("os.name");
        if(property != null && property.contains("Windows")){
            return true;
        }
        return false;
    }
}
```

- 自定义判断Linux系统的条件

```java
public class LinuxCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //        TODO 判断是狗是win系统
//        1 能获取到ioc使用的beanFactory
        ConfigurableListableBeanFactory factory = context.getBeanFactory();
//        2 获取类加载器
        ClassLoader classLoader = context.getClassLoader();
//        3 获取运行环境
        Environment environment = context.getEnvironment();
//        获取bean定义的注册类，可以查看有没有bean的定义，或者注册bean定义
        BeanDefinitionRegistry registry = context.getRegistry();

        String property = environment.getProperty("os.name");
        if(property != null && property.contains("Linux")){
            return true;
        }
        return false;
    }
}
```

- 自定义判断Windows系统的条件

```java
public class WindowsCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 获取到bean定义的注册类，所有的bean的定义都在这个里面进行注册
        BeanDefinitionRegistry registry = context.getRegistry();
        // 判断容器中是否注册了person组件，同时也可以在否的情况下，给容器中注册person组件
        boolean b = registry.containsBeanDefinition("person");
        // 获取当前环境信息
        Environment environment = context.getEnvironment();
        String osName = environment.getProperty("os.name");
        return osName.contains("Windows")&&b;
    }
}
```

在配置类中使用判断条件

```java
@Import({Color.class, Red.class, MyImportSelect.class, MyImportBeanDefinitionRegistrar.class}) //Color是一个类，一个一个导入组件比较麻烦，所以可以直接使用import导入所有组件,id默认是全类名
@Configuration
public class MainConfig2 {

//    	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
//            * 					每次获取的时候才会调用方法创建对象；
//            * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
//            * 			以后每次获取就是直接从容器（map.get()）中拿，
//            * request：同一次请求创建一个实例
//	 * session：同一个session创建一个实例
    @Lazy //懒加载
    @Scope(value = "prototype") //指定取值对象范围
    @Bean(value = "person02") // 修改bean在容器中的名字
    public Person person02(){

        return new Person("person02",22);
    }

//    条件装配 public @interface Conditional{}
//    @Conditional注解标注在方法上，是判断当前方法加载的bean是否加载到容器中，如果放在类上，满足当前条件，整个类配置的bean注册才会生效

//    如果是win系统，就装配person03对象
    @Conditional({WindowsCondition.class})
    @Bean("person03")
    public Person person03(){
        return new Person("person03",25);
    }

//    如果是linux系统，就装配person04对象
    @Conditional({LinuxCondition.class})
    @Bean("person04")
    public Person person04(){
        return new Person("person04",27);
    }

    /**
     * 给容器中注册组件的方式：
     * 1 默认使用包扫描+标注注解 @Controller,@Service,@Repository,@Component，仅限于自己写的类
     * 2 @Bean:导入第三方包里面的组件
     * 3 @Import:快速的给容器中导入一个组件
     * 	 	1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
     * 	 	2）、ImportSelector:返回需要导入的组件的全类名数组；
     * 	 	3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
     */

//    使用工厂bean创建组件
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

> 条件装配 public @interface Conditional{}，首先实现接口，实现条件判断方法。
> @Conditional注解标注在方法上，是判断当前方法加载的bean是否加载到容器中，如果放在类上，满足当前条件，整个类配置的bean注册才会生效

## 给容器中注入组件的方法

1. 默认使用包扫描+标注注解 @Controller,@Service,@Repository,@Component，仅限于自己写的类.
2. @Bean:导入第三方包里面的组件
3. @Import:快速的给容器中导入一个组件
    1. @Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
    2. ImportSelector:返回需要导入的组件的全类名数组；
    3. ImportBeanDefinitionRegistrar:手动注册bean到容器中

## 2.5 @Import

给容器中注册组件；

1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]

2）、@Bean[导入的第三方包里面的组件]

3）、@Import[快速给容器中导入一个组件]       

1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名 2）、ImportSelector:返回需要导入的组件的全类名数组；       

3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中

4）、使用Spring提供的 FactoryBean（工厂Bean）;       

1）、默认获取到的是工厂bean调用getObject创建的对象       

2）、要获取工厂Bean本身，我们需要给id前面加一个&；&colorFactoryBean

给容器中注册组件的方式：

1. 方式一：扫描自己写的类

包扫描+组件标注注解(@Controller/@Service/@Repository/@Component)

1. 方式二：扫描导入的第三方包里边的组件

@Bean

1. 方式三：快速的给容器中导入组件

@Import

源码：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%202.png)

### 2.5.1 @Import({组件1，组件2}) 导入组件

通过@Import({要导入的组件1，要导入的组件2}) 导入组件，id默认是全类名。

```java
@Configuration
===
@Import({Color.class, Animal.class})
===
public class MyConfig2 {

    @Bean
    public Person person(){
        return new Person("zhangsan",34);
    }
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person1(){
        return new Person("bill",34);
    }
    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person2(){
        return new Person("linus",34);
    }
}

myConfig2
==========================
com.example.bean.Color //默认组件名称是全类名
com.example.bean.Animal
==========================
person
bill
```

### 2.5.2 ImportSelector (使用的最多)

返回需要导入组件的全类名的数组

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616848898891-134806e8-d638-4294-8c19-0b985a71d640.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616848898891-134806e8-d638-4294-8c19-0b985a71d640.png)

自定义逻辑，返回需要导入的组件，自定义一个类且要实现ImportSelector接口中的selectImports()方法

```java
//自定义逻辑，导入需要返回的组件
public class MyImportSelect implements ImportSelector {
    /**
     *
     * @param importingClassMetadata 当前标注Import注解的类的所有注解信息
     * @return 就是导入到容器中的组件全类名
     */
返回值，就是到导入到容器中的组件全类名
AnnotationMetadata:当前标注@Import注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {

        Set<String> annotationTypes = importingClassMetadata.getAnnotationTypes();
        System.out.println("annotationTypes:"+annotationTypes);
//        不要返回null ,会报空指针
        return new String[]{"com.rzf.annotation.bean.Yellow","com.rzf.annotation.bean.Blue"};
    }
}
```

在配置类上使用自定义的导入选择器MyImportSelector

```java
@Import({Color.class, Red.class, MyImportSelect.class, MyImportBeanDefinitionRegistrar.class}) //Color是一个类，一个一个导入组件比较麻烦，所以可以直接使用import导入所有组件,id默认是全类名
@Configuration
public class MainConfig2 {

//    	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
//            * 					每次获取的时候才会调用方法创建对象；
//            * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
//            * 			以后每次获取就是直接从容器（map.get()）中拿，
//            * request：同一次请求创建一个实例
//	 * session：同一个session创建一个实例
    @Lazy //懒加载
    @Scope(value = "prototype") //指定取值对象范围
    @Bean(value = "person02") // 修改bean在容器中的名字
    public Person person02(){

        return new Person("person02",22);
    }

//    条件装配 public @interface Conditional{}
//    @Conditional注解标注在方法上，是判断当前方法加载的bean是否加载到容器中，如果放在类上，满足当前条件，整个类配置的bean注册才会生效

//    如果是win系统，就装配person03对象
    @Conditional({WindowsCondition.class})
    @Bean("person03")
    public Person person03(){
        return new Person("person03",25);
    }

//    如果是linux系统，就装配person04对象
    @Conditional({LinuxCondition.class})
    @Bean("person04")
    public Person person04(){
        return new Person("person04",27);
    }

    /**
     * 三方包里面的组件
     * 3 @Import:快速的给容器中导入一个组件
     * 	 	1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
     * 	 	2）、ImportSelector:返回需要导入的组件的全类名数组；
     * 	 	3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
     */

//    使用工厂bean创建组件
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

### 2.5.3 ImportBeanDefinitionRegistrar

上源码

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616849713321-622d7b45-2f99-4aac-875d-0c76d5895bd9.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616849713321-622d7b45-2f99-4aac-875d-0c76d5895bd9.png)

自定义一个注册器

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     *
     * @param importingClassMetadata 当前标注了@Import注解的类上的--所有注解信息--
     * @param registry BeanDefinition注册类：把所有需要添加到容器中的bean，
     *                 调用BeanDefinitionRegistry.registerBeanDefinition()手动注册
     */
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        boolean color = registry.containsBeanDefinition("com.example.bean.Color");
        boolean food = registry.containsBeanDefinition("com.example.bean.Food");
        // 判断容器中是否有指定的bean,有的话才会注册Book.class
        if (color&&food){
            // 注册bean时，可以指定bean的名称
            // 指定Bean定义信息：(Bean的类型、作用域等)
            RootBeanDefinition beanDefinition = new RootBeanDefinition(Book.class);
            registry.registerBeanDefinition("MyBook",beanDefinition);
        }
    }
}
```

在配置类上使用自定义的导入选择器MyImportBeanDefinitionRegistrar

```java
@Import({Color.class, Red.class, MyImportSelect.class, MyImportBeanDefinitionRegistrar.class}) //Color是一个类，一个一个导入组件比较麻烦，所以可以直接使用import导入所有组件,id默认是全类名
@Configuration
public class MainConfig2 {

//    	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
//            * 					每次获取的时候才会调用方法创建对象；
//            * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
//            * 			以后每次获取就是直接从容器（map.get()）中拿，
//            * request：同一次请求创建一个实例
//	 * session：同一个session创建一个实例
    @Lazy //懒加载
    @Scope(value = "prototype") //指定取值对象范围
    @Bean(value = "person02") // 修改bean在容器中的名字
    public Person person02(){

        return new Person("person02",22);
    }

//    条件装配 public @interface Conditional{}
//    @Conditional注解标注在方法上，是判断当前方法加载的bean是否加载到容器中，如果放在类上，满足当前条件，整个类配置的bean注册才会生效

//    如果是win系统，就装配person03对象
    @Conditional({WindowsCondition.class})
    @Bean("person03")
    public Person person03(){
        return new Person("person03",25);
    }

//    如果是linux系统，就装配person04对象
    @Conditional({LinuxCondition.class})
    @Bean("person04")
    public Person person04(){
        return new Person("person04",27);
    }

    /**
     * 三方包里面的组件
     * 3 @Import:快速的给容器中导入一个组件
     * 	 	1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
     * 	 	2）、ImportSelector:返回需要导入的组件的全类名数组；
     * 	 	3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
     */

//    使用工厂bean创建组件
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

### 2.5.4 使用FactoryBean

使用Spring提供的FactoryBean(工厂Bean)，区别于普通的bean。

上源码：

```java
package org.springframework.beans.factory;
public interface FactoryBean<T> {
	T getObject() throws Exception;
	Class<?> getObjectType();
	boolean isSingleton();
}
```

自定义一个Color类的FactoryBean

```java
public class ColorFactoryBean implements FactoryBean<Color> {

    /**
     *
     * @return 返回一个Color对象，这个对象会返回到容器中
     * @throws Exception
     */
    public Color getObject() throws Exception {
        return new Color();
    }
    /**
     * 很重要
     * @return 将来通过该FactoryBean注入到容器中的bean，根据bean的id获取到的bean的类型就是该方法返回的类型
     */
    public Class<?> getObjectType() {
        return Color.class;
    }

    /**
     *
     * @return 如果返回值为true， 表示这是一个单实例对象，在容器中只会保存一份。
     *         如果返回值为false，表示这个是一个多实例对象，每次获取都会创建一个新的实例
     */
    public boolean isSingleton() {
        return true;
    }
}
```

将这个Color类的FactoryBean在配置类中注入到容器中

```java
@Import({Color.class, Red.class, MyImportSelect.class, MyImportBeanDefinitionRegistrar.class}) //Color是一个类，一个一个导入组件比较麻烦，所以可以直接使用import导入所有组件,id默认是全类名
@Configuration
public class MainConfig2 {

//    	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
//            * 					每次获取的时候才会调用方法创建对象；
//            * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
//            * 			以后每次获取就是直接从容器（map.get()）中拿，
//            * request：同一次请求创建一个实例
//	 * session：同一个session创建一个实例
    @Lazy //懒加载
    @Scope(value = "prototype") //指定取值对象范围
    @Bean(value = "person02") // 修改bean在容器中的名字
    public Person person02(){

        return new Person("person02",22);
    }

//    条件装配 public @interface Conditional{}
//    @Conditional注解标注在方法上，是判断当前方法加载的bean是否加载到容器中，如果放在类上，满足当前条件，整个类配置的bean注册才会生效

//    如果是win系统，就装配person03对象
    @Conditional({WindowsCondition.class})
    @Bean("person03")
    public Person person03(){
        return new Person("person03",25);
    }

//    如果是linux系统，就装配person04对象
    @Conditional({LinuxCondition.class})
    @Bean("person04")
    public Person person04(){
        return new Person("person04",27);
    }

    /**
     * 三方包里面的组件
     * 3 @Import:快速的给容器中导入一个组件
     * 	 	1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
     * 	 	2）、ImportSelector:返回需要导入的组件的全类名数组；
     * 	 	3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
     */

//    使用工厂bean创建组件
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}

myConfig2
=== 在容器中加的是colorFactoryBean，获取到bean也有colorFactoryBean
colorFactoryBean
```

在容器中加的是colorFactoryBean，获取到bean也有colorFactoryBean，当我们根据colorFactoryBean拿到bean，看一下是什么？

```java
    @Test
    public void test05(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 工厂Bean获取的是调用getObject创建的对象
        Object colorFactoryBean = annotationConfigApplicationContext.getBean("colorFactoryBean");
        System.out.println("Color类的FactoryBean的类型："+colorFactoryBean.getClass());
        // Color类的FactoryBean的类型：class com.example.bean.Color
    }
```

拿到的是com.example.bean.Color。

这就是说，虽然在配置文件中装配的是ColorFactoryBean，但是按照ColorFactoryBean的id：colorFactoryBean从容器中获取到的bean的类型却是调用getObject创建的对象com.example.bean.Color。

**如果就要获取到ColorFactoryBean呢？---加 & 标识**

```java
    @Test
    public void test05(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        String[] beans = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String s : beans) {
            System.out.println(s);
        }
        // 工厂Bean获取的是调用getObject创建的对象
        Object bean1 = annotationConfigApplicationContext.getBean("colorFactoryBean");
        System.out.println("Color类的FactoryBean："+bean1.getClass());

        // 就要从容器中获取对应类的工厂bean
        Object bean2 = annotationConfigApplicationContext.getBean("&colorFactoryBean");
        System.out.println("Color类的FactoryBean："+bean2.getClass());
    }

myConfig2
colorFactoryBean
============= 搜嘎 =============
Color类的FactoryBean：class com.example.bean.Color
============= 搜嘎 =============
Color类的FactoryBean：class com.example.bean.ColorFactoryBean
```

**Spring与其它框架整合时，用的特别多，例如整合Mybaties。**

`*"&colorFactoryBean":获取的时候，添加&，即可获得工厂bean对象*`

# 3 Bean的生命周期

Bean的生命周期由容器管理，即管理bean的：创建 -- 初始化 -- 销毁。我们可以自定义**初始化和销毁**方法，容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法。

> bean对象的初始化在创建完成之后。

## 3.1 通过@Bean指定init-method和destory-method

可以在xml配置文件中通过参数指定初始化方法和销毁方法。

在实体类中创建初始化和销毁方法

```java
public class Car {
    public Car(){
        System.out.println("car contructor ...");
    }

    public void init(){
        System.out.println("Car初始化");
    }
    public void destory(){
        System.out.println("Car销毁");
    }
}
```

在配置文件中，注入bean时指定初始化和销毁方法

```java
@Configuration
@Import({Color.class, Animal.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
public class MyConfig2 {
//指定初始化和销毁方法
    @Bean(initMethod = "init",destroyMethod = "destory")
    public Car car(){
        return new Car();
    }
}
```

测试：

```java
    @Test
    public void test06(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 关闭容器的时候调用bean的销毁方法
        annotationConfigApplicationContext.close();
    }

car contructor ...
Car初始化
Car销毁
```

**要注意的是，单实例和多实例的初始化和销毁时机。**

**初始化：**

- **对象创建完成 ，并赋值好，调用初始化方法**

**销毁：**

- **单实例：容器关闭的时候调用销毁方法。**
- **多实例：容器不会管理整个bean，也就是说容器不会调用销毁方法，需要手动调用。**

> 指定组件的初始化和销毁操作,一般在初始化方法中可以配置一些配置信息，比如配置数据
> 销毁的时候，可以关闭链接，关闭数据源等

## 3.2 通过实现InitializingBean和DisposableBean接口

通过让bean实现InitialzingBean接口来定义初始化逻辑，实现DisposableBean接口来定义销毁逻辑。

**InitialzingBean源码**

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616900625195-62cf9dd9-a5a1-44d0-b478-d0e2d00a26b4.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616900625195-62cf9dd9-a5a1-44d0-b478-d0e2d00a26b4.png)

**DisposableBean源码**

**销毁单实例Bean**

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616900764309-dedb42c6-69ec-4d90-a292-3eb635db0b7c.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616900764309-dedb42c6-69ec-4d90-a292-3eb635db0b7c.png)

自定义一个Build类，并实现InitializingBean和DisposableBean接口

```java
public class Build implements InitializingBean, DisposableBean {

    public Build(){
        System.out.println("Build 构造函数...");
    }

    /**
     * 单实例Bean的销毁方法，在容器关闭的时候进行调用
     * @throws Exception 销毁失败抛异常
     */
    public void destroy() throws Exception {
        System.out.println("Build的销毁方法...");
    }

    /**
     * Bean初始化方法
     * @throws Exception 初始化失败抛异常
     */
    public void afterPropertiesSet() throws Exception {
        System.out.println("Build的初始化方法");
    }
}
```

在配置类中注入Build的bean

```java
@Configuration
public class MyConfig2 {
    @Bean
    public Build build(){
        return new Build();
    }
}
```

测试

```java
    @Test
    public void test06(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 关闭容器的时候调用bean的销毁方法
        annotationConfigApplicationContext.close();
    }

Build 构造函数...
Build的初始化方法
Build的销毁方法...
```

## 3.3 使用JSR250规范

@PostConstruct：在bean创建完成并且属性值赋值完成后执行初始化；

@PreDestroy：在容器销毁bean之前，通知容器进行清理工作；

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616902001097-4e5222a2-f666-4ad2-8df5-784a75aa2aa4.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616902001097-4e5222a2-f666-4ad2-8df5-784a75aa2aa4.png)

创建一个Dog类，并在初始化和销毁方法上添加对应的注解

```java
public class Dog {
    public Dog(){
        System.out.println("Dog 的构造函数");
    }

    /**
     * 对象创建并赋值后，调用
     */
    @PostConstruct
    public void init(){
        System.out.println("Dog构造之后调用...@PostConstruct");
    }

    /**
     * 对象移除之前，调用
     */
    @PreDestroy
    public void destory(){
        System.out.println("Dog对应的bean移除之前调用...@PreDestroy");
    }
}
```

在配置类中注入Dog对应的bean

```java
@Configuration
public class MyConfig2 {
    @Bean
    public Dog dog(){
        return new Dog();
    }
}
```

测试

```java
    @Test
    public void test06(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 关闭容器的时候调用bean的销毁方法
        annotationConfigApplicationContext.close();
    }

Dog 的构造函数
Dog构造之后调用...@PostConstruct
Dog对应的bean移除之前调用...@PreDestroy
```

## 3.4 BeanPostProcessor后置处理器

在bean的**初始化方法之前**和**初始化方法之后**进行一些处理工作。注意是初始化方法，和销毁方法没有一毛钱的关系。

> 构造方法调用完成之后才会调用初始化方法。

**初始化方法之前:   调用：**postProcessBeforeInitialization()

**初始化方法之后：调用：**postProcessAfterInitialization()

即使没有自定义初始化方法，在组件创建前后，后置处理器方法也会执行。

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;

public interface BeanPostProcessor {
	==== 初始化之前 ====
   	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
     	在任何bean初始化回调（例如InitializingBean的{@code afterPropertiesSet}或自定义的init-method）之前，
     	将此BeanPostProcessor应用于给定的新bean实例。 该bean将已经用属性值填充。 返回的Bean实例可能是原始实例的包装。
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
	==== 初始化之后 ====
     * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
        在任何bean初始化回调（例如InitializingBean的{@code afterPropertiesSet}或自定义的init-method）之后，
        将此BeanPostProcessor应用于给定的新bean实例。 该bean将已经用属性值填充。 返回的Bean实例可能是原始实例的包装。
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
         对于FactoryBean，将为FactoryBean实例和由FactoryBean创建的对象（从Spring 2.0开始）调用此回调。
         后处理器可以通过相应的{@code bean instanceof FactoryBean}检查来决定是应用到FactoryBean还是创建的对象，还是两者都应用。
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
         与所有其他BeanPostProcessor回调相反，此回调还将在{@link InstantiationAwareBeanPostProcessor＃postProcessBeforeInstantiation}方法触发短路后被调用。
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

自定义

```java
//将组件添加到容器中
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization"+"--"+beanName+"--"+bean);
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization"+"--"+beanName+"--"+bean);
        return bean;
    }
}
```

在配置类文件中注入该bean

```java
@Configuration
public class MyConfig2 {
	@Bean
    public MyBeanPostProcessor myBeanPostProcessor(){
        return new MyBeanPostProcessor();
    }
}
```

测试

```java
    @Test
    public void test06(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 关闭容器的时候调用bean的销毁方法
        annotationConfigApplicationContext.close();
    }
```

结果：不同的生命周期方式

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616984439251-0e0af4a6-5c40-49d5-9be3-af07780b01ea.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616984439251-0e0af4a6-5c40-49d5-9be3-af07780b01ea.png)

```java
======================================================================
postProcessBeforeInitialization--person--Person(name=zhangsan, age=34)
postProcessAfterInitialization--person--Person(name=zhangsan, age=34)
======================================================================
```

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616983923808-b6b28f89-29b4-4de8-b377-5c00d036eb42.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616983923808-b6b28f89-29b4-4de8-b377-5c00d036eb42.png)

```java
===================================================================
Build的构造函数...
//初始化之前调用了前置处理器
postProcessBeforeInitialization--build--com.example.bean.Build@327b636c
Build的初始化方法
//初始化之后调用后置处理器
postProcessAfterInitialization--build--com.example.bean.Build@327b636c
Build的销毁方法...
===================================================================
```

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616984013447-d169e5dc-0e7d-46be-a3a2-76a0c05632a7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616984013447-d169e5dc-0e7d-46be-a3a2-76a0c05632a7.png)

```java
===================================================================
car的构造函数 ...
postProcessBeforeInitialization--car--com.example.bean.Car@7c417213
Car的初始化方法
postProcessAfterInitialization--car--com.example.bean.Car@7c417213
Car的销毁方法
===================================================================
```

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1616984137460-118c9649-2145-49c0-b3e8-571f8396cc46.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1616984137460-118c9649-2145-49c0-b3e8-571f8396cc46.png)

```java
===================================================================
Dog 的构造函数
postProcessBeforeInitialization--dog--com.example.bean.Dog@19d37183
Dog的初始化方法...@PostConstruct
postProcessAfterInitialization--dog--com.example.bean.Dog@19d37183
Dog的销毁方法...@PreDestroy
===================================================================
```

### 3.4.1 BeanPostProcessor工作原理

### 3.4.2 Spring底层对BeanPostProcessor的应用

【案例1】在实体类中获取ioc容器

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617020941009-50ade5a0-ad26-40d7-bff4-4dee9e73284d.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617020941009-50ade5a0-ad26-40d7-bff4-4dee9e73284d.png)

这个功能是如何实现的呢？

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617021903470-72883c1e-dac5-496c-93b5-acfca5aeafef.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617021903470-72883c1e-dac5-496c-93b5-acfca5aeafef.png)

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617021910194-12918b44-0aa1-4dae-95ba-f445a7301f9a.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617021910194-12918b44-0aa1-4dae-95ba-f445a7301f9a.png)

【案例2】对象创建完，给数据赋值以后，做数据校验的工具。

BeanValidationPostProcessor

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617022592576-43364868-98d0-4342-ac5b-dd06439fe112.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617022592576-43364868-98d0-4342-ac5b-dd06439fe112.png)

后来的@Autowired注解

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617023043562-6708076d-e6fd-4737-9080-74876f04dbb6.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617023043562-6708076d-e6fd-4737-9080-74876f04dbb6.png)

总结：

bean赋值、注入其它注解、@Autowired、生命周期注解功能、@Async底层都是BeanPostProcessor来完成的。

## 3.5、小结

bean的生命周期：

- **bean创建---初始化----销毁的过程**

容器管理bean的生命周期；

- 我们可以自定义**初始化和销毁方法**；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法

构造（对象创建）

- 单实例：在容器启动的时候创建对象
- 多实例：在每次获取的时候创建对象

BeanPostProcessor.postProcessBeforeInitialization:`*在初始化之前做后置处理工作*`
初始化：
对象创建完成，并赋值好，调用初始化方法,`*初始化，可以自定义初始化逻辑，初始化前后可以使用这两个方法拦截*`
BeanPostProcessor.postProcessAfterInitialization:`*在初始化之后做后置处理工作*`

销毁：

- 单实例：容器关闭的时候
- 多实例：容器不会管理这个bean；容器不会调用销毁方法；

BeanPostProcessor原理：

遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，
一但返回null，跳出for循环，不会执行后面的;

```java
BeanPostProcessor.postProcessorsBeforeInitialization
BeanPostProcessor原理
populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值，然后执行初始化动作
initializeBean：初始化
{
applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName)
```

1）、指定初始化和销毁方法；

- 通过@Bean指定init-method和destroy-method；
- 或者在xml文件中配置参数：init-method和destroy-method=

2）、通过让Bean实现InitializingBean（定义初始化逻辑），

- DisposableBean（定义销毁逻辑）;

3）、可以使用JSR250；

- @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
- @PreDestroy：在容器销毁bean之前通知我们进行清理工作

4）、BeanPostProcessor【interface】：bean的后置处理器；
在bean初始化前后进行一些处理工作；

- postProcessBeforeInitialization:在初始化之前工作
- postProcessAfterInitialization:在初始化之后工作

Spring底层对 BeanPostProcessor 的使用；

- bean赋值，注入其他组件，
- @Autowired，生命周期注解功能，
- @Async,xxx BeanPostProcessor;

# 4 属性赋值相关的注解

# 4.1 @Value

通过@PropertySource注解将properties配置文件中的值存储到Spring的 Environment中，Environment接口提供方法去读取配置文件中的值，参数是properties文件中定义的key值。

```
    /**
     * 使用@Value赋值
     *    1、基本数值
     *    2、可以些SpEL，#{}
     *    3、可以写${},取出配置文件中的值(即在运行环境变量中的值).
     *      通过@PropertySource注解将properties配置文件中的值存储到Spring的 Environment中，
     *      Environment接口提供方法去读取配置文件中的值，参数是properties文件中定义的key值。
     */
```

Person类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    @Value("张三")
    private String name;
    @Value("#{20-2}")
    private Integer age;
    @Value("${person.nickName}")
    private String nickName;
}
```

person.properties

```yaml
person
	nickName="zhangzhang"
```

使用PropertySources读取外部配置文件中的属性k/v保存到运行的环境变量中

```java
@PropertySources(@PropertySource(value = {"classpath:/person.properties"}))
@Configuration
public class MyConfigOfPropertyValues {
    @Bean
    public Person person(){
        return new Person();
    }
}
```

测试

```java
    @Test
    public void test07(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfigOfPropertyValues.class);
        Person person2 = (Person) annotationConfigApplicationContext.getBean("person");
        System.out.println(person2);
        ==== Person(name=张三, age=18, nickName="zhangzhang")====
    }
```

从环境变量中获取也是可以的

```java
    @Test
    public void test07(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfigOfPropertyValues.class);
        ConfigurableEnvironment environment = annotationConfigApplicationContext.getEnvironment();
        String property = environment.getProperty("person.nickName");
        System.out.println(property);
        === "zhangzhang" ===
    }
```

# 5 自动装配

# 5.1 @Autowired

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617171363261-915188b8-8b5b-4074-b751-8c9a771e883a.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617171363261-915188b8-8b5b-4074-b751-8c9a771e883a.png)

1. 默认优先按照类型去容器中找对应的组件：applicationContext.getBean(BookServiceImpl.class);
2. 如果找到多个相同类型的组件，将会按照属性的名称作为组件的id去容器中查找。

【案例】

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617172014520-50a696f5-0beb-48a8-8d31-3974c4cbeeba.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617172014520-50a696f5-0beb-48a8-8d31-3974c4cbeeba.png)

测试：

```java
    @Test
    public void test01(){
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        BookServiceImpl bean = applicationContext.getBean(BookServiceImpl.class);
        System.out.println(bean.toString());
    }
```

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617172917304-c1be5a87-fa08-496b-823e-c12afc0346ad.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617172917304-c1be5a87-fa08-496b-823e-c12afc0346ad.png)

### 5.1.1 @Autowired标注在方法头上

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617180075520-7f6f7bf9-7072-45c1-8930-c6e147b03e8e.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617180075520-7f6f7bf9-7072-45c1-8930-c6e147b03e8e.png)

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617179997213-ec8be584-c8b4-4dcf-abce-385141093be9.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617179997213-ec8be584-c8b4-4dcf-abce-385141093be9.png)

> car是从容器中获取，容器中只有一个car

### 5.1.2 @Autowired标注在构造器上

如果组件只有一个参数构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取。

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617181222655-483588b1-dafd-485f-8479-015ab3116ad3.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617181222655-483588b1-dafd-485f-8479-015ab3116ad3.png)

### 5.1.3 @Autowired标注在参数位置

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617181385465-3f479645-6fcb-4460-b8f0-fe00a09166c7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617181385465-3f479645-6fcb-4460-b8f0-fe00a09166c7.png)

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617182873761-c51c64ae-10c0-4303-a2bf-3eb372d7216c.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617182873761-c51c64ae-10c0-4303-a2bf-3eb372d7216c.png)

## 5.2 @Autowired和@Qualifier配合

qualifier指定通过组件的id获取组件。

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617172609480-9e9fb53e-3b10-419a-a895-52724697d8d2.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617172609480-9e9fb53e-3b10-419a-a895-52724697d8d2.png)

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617172968859-56d6a1d3-d7a1-472c-aeba-3bcc3751c68a.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617172968859-56d6a1d3-d7a1-472c-aeba-3bcc3751c68a.png)

自动装配的前提，默认一定要将容器的组件赋好值。否则没有就会报错。

是否可以根据容器中组件的有无来判断是否进行装配呢？

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617173283958-52abd797-b927-4002-9900-2657d0820edd.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617173283958-52abd797-b927-4002-9900-2657d0820edd.png)

## 5.3 @Primary

让spring进行自动装配的时候，默认使用首选的bean。

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617175249033-284e6245-d500-4105-8991-b48100806133.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617175249033-284e6245-d500-4105-8991-b48100806133.png)

## 5.4 @Resource和@Inject

@Resource属于JSR250规范，@Inject属于JSR330规范。都是java规范。

**@Resource**

和@Autowired一样实现自动装配功能，默认是按照组件名称进行装配，也可以指定要装配组件的名称。但其不支持@Primary和@Autowired(required=false)的功能。

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617175713725-796893ce-fb3a-46e8-804e-314e78768b52.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617175713725-796893ce-fb3a-46e8-804e-314e78768b52.png)

@Inject

需要导入注解

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617175965575-39540ce0-5122-46a4-a8d9-d588b67863a1.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617175965575-39540ce0-5122-46a4-a8d9-d588b67863a1.png)

支持@Autowired(required=false)

推荐使用spring自带的。

# 6 自动装配原理

AutowiredAnnotationBeanPostProcessor

# 7 自定义组件使用Spring容器底层的一些组件

例如:自定义组件想要使用AplicationContext、Beanfactory、......，需要自定义组件实现xxxAware接口，在创建对象的时候，会调用接口的方法注入相关组件。

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617195465693-8383ab95-d36c-4910-ad72-780e4c28b097.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617195465693-8383ab95-d36c-4910-ad72-780e4c28b097.png)

![https://cdn.nlark.com/yuque/0/2021/png/12833864/1617195550581-3c463a83-baab-43c8-b902-4953eec31459.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1617195550581-3c463a83-baab-43c8-b902-4953eec31459.png)

xxxAware的功能都是由xxxProcessor后置处理器实现的。例如：ApplicationContextAware===》ApplicationContextAwareProcessor

# 8 @Profile

Profile: 指定组件在哪个环境中的情况下才能被注册到容器中，不指定任何环境下都能指定。 加了环境标识的bean，只有对应的环境被激活了，才会注册到容器中。但是如果标了default，则会默认加载这个。* Spring为我们提供的可以根据当前环境，动态的激活和切换一系列bean组件的功能 我们有开发环境、测试环境、生成环境 对应的数据源有：(/A)(/B)(/C) 利用注解：@Profile

@Profile也可以写在类上，表明只有在指定的环境下，类中的内容才会被激活使用。

【案例】

先加入数据库连接池依赖

```xml
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.22</version>
        </dependency>
```

配置一个数据库链接的配置文件

```xml
db.user=root
db.password=root
db.driverClass=com.mysql.jdbc.Driver
```

在配置类中配置不同环境使用的数据源

```java
@PropertySource("classpath:/dbconfig.properties")
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver valueResolver;

    private String driverClass;

    @Profile("test")
    @Bean("testDatasource")
    public DataSource dataSourceTest(@Value("${db.password}") String password) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(password);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");

        // 利用值解析器
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
    @Profile("dev")
    @Bean("devDatasource")
    public DataSource dataSourceDev() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
    @Profile("prod")
    @Bean("prodDatasource")
    public DataSource dataSourceProd() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
    @Profile("default")
    @Bean("prodDatasource")
    public DataSource dataSourceDefault() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }

    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        // 利用值解析器，解析字符串
        this.valueResolver = resolver;
        this.driverClass = valueResolver.resolveStringValue("${db.driverClass}");
    }
}

```

如何切换环境呢？

/**

* 切换环境的方式：

* 1、使用命令行动态参数：在虚拟机参数位置加载-Dspring.profiles.active=test(test是测试的环境标识)

* 2、代码的方式激活某种环境

*/

```java
@Test
public void test01(){
    // 1、创建一个applicationContext
    AnnotationConfigApplicationContext applicationContext =
        new AnnotationConfigApplicationContext();
    // 2、设置需要激活的环境,可以设置多个
    applicationContext.getEnvironment().setActiveProfiles("test","dev");
    // 3、注册配置类
    applicationContext.register(MainConfigOfProfile.class);
    // 4、启动刷新容器
    applicationContext.refresh();

    String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
    for (String s : beanNamesForType) {
        System.out.println(s);
    }
    applicationContext.close();
}
```

BeanValidationPostProcessor

BeanValidationPostProcessor

> 只有将我们写的类以组件的形式加入到容器中，才能使用spring提供的其他功能。

# 9、Aop使用和原理

## AOP使用

> 底层：动态代理

指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式。面向对象编程只能够对现实世界中的实体进行抽象，但是没有办法在方法层面进行服用，AOP面向切面编程，也是一种编程思想，可以在方法层面公用代码片段。

加入AOP依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
```

- 前置通知（`@Before`）

在目标方法运行之前运行

- 后置通知（`@After`）

在目标方法运行结束之后运行

- 返回通知（`@AfterReturning`）

在目标方法正常返回之后运行

- 异常通知（`AfterThrowing`）

在目标方法出现异常以后运行

- 环绕通知（`@Around`）

动态代理，手动推进目标方法运行

使用`@Pointcut`注解，抽取公共的切入点表达式。

切面类上使用`@Aspect`注解，声明该类为切面类。

示例：

```java
//声明该类是一个切面类
@Aspect
public class LogAspect {
//声明公共的切入点表达式
    @Pointcut("execution(public int com.study.aop.MathCalculator.*(..))")
    public void pointCut() {}
//前置通知
    @Before("pointCut()")
    public void logStart() {
        System.out.println("除法运行了...参数列表: ");
    }
//后置通知
    @After("pointCut()")
    public void logEnd() {
        System.out.println("除法结束了");
    }

    @AfterReturning("pointCut()")
    public void logReturn() {
        System.out.println("除法正常返回...返回值:");
    }

    @AfterThrowing("pointCut()")
    public void logException() {
        System.out.println("除法出现异常...异常信息:");
    }
}
```

将切面类和要切入的类都加入ioc容器，并且使用`@EnableAspectJAutoProxy`在配置类中开启Aspectj自动代理：

```java
@Configuration
@EnableAspectJAutoProxy //开启切面自动配置代理
public class MainConfigOfCalculator {
    @Bean
    public MathCalculator mathCalculator() {
        return new MathCalculator();
    }

    @Bean
    public LogAspect logAspect() {
        return new LogAspect();
    }
}
```

如果直接自己创建`MathCalculator`对象，则运行的方法依然是该对象没有被代理的方法。只有从ioc容器中获取的`MathCalculator`对象才是被动态代理的对象。

在通知方法中获取被代理的方法的信息：

- 带有通知注解的方法，都可以加入一个`JoinPoint`类型的参数，可以通过该参数获取当前代理的方法名称、方法参数列表等信息。示例：

```java
//前置通知在这个方法执行前会自动执行//    执行时机：目标方法之前切入
//    @Before("public int com.rzf.annotation.aop.MathCalculator.div(int,int)")
//    @Before("public int com.rzf.annotation.aop.MathCalculator.*(..)")//任意方法，任意多个参数
@Before("pointCut()")
public void logStart(JoinPoint joinPoint) {
    String methodName = joinPoint.getSignature().getName();
    Object[] methodArgs = joinPoint.getArgs();
    System.out.println(methodName + "方法运行了...参数列表: " + Arrays.toString(methodArgs));
}
```

- `@AfterReturning`注解带有`returning`属性，可以在方法的参数中定义一个代理方法的返回结果参数，将该参数名指定到`@AfterReturning`的`returning`属性上，方法运行的结果便会赋值给该参数。例如：

```java
@AfterReturning(value = "pointCut()", returning = "result")
public void logReturn(JoinPoint joinPoint, Object result) {   // 如果想在使用result的同时，也使用joinPoint，必须将joinPoint参数放在参数的第一位上。
    System.out.println("除法正常返回...返回值:" + result);
}
```

- `@AfterThrowing`注解带有`throwing`属性，可以在方法的参数中定义一个异常参数，将该参数名指定到`@AfterThrowing`的`throwing`属性上，便可以通过该参数获取方法运行中出现的异常。示例：

```java
//    Joinpoint在参数列表中只能出现在第一个位置
@AfterThrowing(value = "pointCut()", throwing = "exception")
public void logException(Exception exception) {
    System.out.println("除法出现异常...异常信息:" + exception.getMessage());
}
```

# @EnableAspectJAutoProxy 原理

`@EnableAspectJAutoProxy`中使用`@Import(AspectJAutoProxyRegistrar.class)`加入了一个组件。

`AspectJAutoProxyRegistrar`类实现了`ImportBeanDefinitionRegistrar`接口，可以手动加载组件。

`AspectJAutoProxyRegistrar`的执行过程：

先判断是否存在beanID为`org.springframework.aop.config.internalAutoProxyCreator`的bean，不存在则注册一个`AnnotationAwareAspectJAutoProxyCreator`类型的bean。

然后获取类上面的`@EnableAspectJAutoProxy`属性信息，判断`proxyTargetClass`属性、`exposeProxy`属性，做一些后续处理。

`AnnotationAwareAspectJAutoProxyCreator`的继承关系如下图：`AnnotationAwareAspectJAutoProxyCreator`最终实现了`BeanFactoryAware`、`BeanPostProcessor`

![https://cdn.nlark.com/yuque/__mermaid_v3/c32588fd86f3f27f88869273e5b57f61.svg](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/c32588fd86f3f27f88869273e5b57f61.svg)

注意点：

BeanPostProcessor接口定义的方法是：

- postProcessBefore**Initialization**()
- postProcessAfter**Initialization**()

InstantiationAwareBeanPostProcessor接口继承了BeanPostProcessor，另外又增加了两个方法：

- postProcessBefore**Instantiation**()
- postProcessAfter**Instantiation**()

## 注册AnnotationAwareAspectJAutoProxyCreator

流程：

1. 传入配置类，创建ioc容器
2. 注册配置类，调用refresh()刷新容器。以下为refresh()方法内部的执行逻辑：
3. `registerBeanPostProcessor(beanFactory);`注册bean的后置处理器
4. 先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
5. 给容器中加别的BeanPostProcessor
6. 优先注册实现了PriorityOrdered接口的BeanProcessor
7. 再给容器中注册实现了Ordered接口的BeanPostProcessor
8. 注册没实现优先级接口的BeanPostProcessor
9. 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中例如：创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectAutoProxyCreator】

AbstractAutowireCapableBeanFactory::doCreateBean()

1. 创建bean的实例
2. `populateBean`给bean的各种属性赋值
3. `initializeBean`初始化bean
4. `invokeAwareMethods`：处理Aware接口的方法回调
5. `applyBeanPostProcessorsBeforeInitialization`：应用后置处理器的beforeInitialization方法
6. `invokeInitMethods`：执行@Bean注解等自定义的Bean初始化方法
7. `applyBeanPostProcessorsAfterInitialization`：执行后置处理器的afterInitialization方法
8. `BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)`创建成功；-> aspectJAdvisorsBuilder
9. 把BeanPostProcessor注册到BeanFactory中`beanFactoy.addBeanPostProcessor(postProcessor)`
10. `finishBeanFactoryInitialization(beanFactory)`完成bean初始化工作，创建剩下的单实例bean

此时创建的bean，是那些不属于后置处理器的普通单实例bean。后置处理器本身也是bean，后置处理器的bean在第3.6步进行创建

1. 遍历获取容器中所有的bean，依次创建对象：getBean(beanName)getBean -> doGetBean -> getSingleton
2. 创建bean

AnnotationAwareAspectJAutoProxyCreator会在所有bean创建之前进行一次拦截，调用实现的InstantiationAwareBeanPostProcessor接口的postProcessBeforeInstantiation()方法。

1. 创建好的bean都会被缓存起来。
2. createBean();创建bean
3. `resolveBeforeInstantiation(beanName, mdbToUse)`解析beforeInstantiation。希望后置处理器能在此返回一个代理对象，如果能返回代理对象就使用，如果不能就继续第2步。
4. 后置处理器先尝试返回对象

```java
// 拿到所有BeanPostProcessor后置处理器，如果该后置处理器属于InstantiationAwareBeanPostProcessor，则执行该后置处理器的postProcessBeforeInstantiation方法。
// （注意postProcessBeforeInstantiation方法 不同于 postProcessBeforeInitialization方法）
bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
if (bean != null) {
    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
}
```

BeanPostProcessor的postProcessBeforeInitialization和postProcessAfterInitialization是在Bean对象创建完成初始化前后调用的；

InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation和postProcessAfterInstantiation是在创建Bean实例之前先尝试用后置处理器返回对象。

即AnnotationAwareAspectJAutoProxyCreator实现的InstantiationAwareBeanPostProcessor的方法，会在任何bean创建之前先尝试返回bean的实例。

1. `doCreateBean(beanName, mbdToUse, args)` ，真正的去创建一个bean实例 ，和第3.6环节中的操作流程一样

## InstantiationAwareBeanPostProcessor接口

`AnnotationAwareAspectJAutoProxyCreator`的父类`AbstractAutoProxyCreator`，实现了`InstantiationAwareBeanPostProcessor`接口，它的作用是：

1. 每一个bean创建之前，调用`postProcessBeforeInstantiation()`；
2. 判断当前bean是否在`advisedBeans`中（`advisedBeans`保存了所有需要增强的bean）
3. 调用`isInfrastructureClass()`判断当前Bean是否是基础类型。

- aop相关基础设施类型的bean不应该被动态代理。

```java
boolean retVal = Advice.class.isAssignableFrom(beanClass) ||
				Pointcut.class.isAssignableFrom(beanClass) ||
				Advisor.class.isAssignableFrom(beanClass) ||
				AopInfrastructureBean.class.isAssignableFrom(beanClass);
```

- 切面的bean不应该被动态代理：带有`@Aspect`注解的bean。

1. 调用`shouldSkip()`判断当前bean是否需要跳过
2. 调用`findCandidateAdvisors()`获取候选的增强器`List<Advisor> candidateAdvisors`（切面里面的通知方法）每一个封装的通知方法增强器类型为：`InstantiationModelAwarePointcutAdvisorImpl`
3. 判断增强器是否为`AspectJPointcutAdvisor`类型的，如果是，则判断该增强器的aspectName是否和beanName相同
4. `AbstractAutoProxyCreator`的`postProcessAfterInstantiation()`直接返回true
5. `AbstractAutoProxyCreator`的`postProcessBeforeInitialization()`直接返回传入的bean对象
6. 创建好bean对象后，调用`postProcessAfterInitialization()`调用`wrapIfNecessary()`方法，如果需要的话对bean进行包装。
7. 同`postProcessBeforeInstantiation()`一样进行一些校验：

- 当前bean是否在`advisedBeans`中
- 当前bean是否是基础类型
- 当前bean是否需要跳过

1. 调用`getAdvicesAndAdvisorsForBean()`获取当前bean的所有增强器（通知方法）：`Object[] specificInterceptors`
2. 找到候选的所有增强器
3. 找到能在当前bean中使用的增强器
4. 给增强器排序
5. 将bean保存到`advisedBeans`中，表示当前bean已经被增强处理了
6. 如果当前Bean需要增强，创建当前bean的代理对象；
7. 获取所有增强器（通知方法）
8. 保存到proxyFactory中
9. 创建代理对象Spring在`DefaultAopProxyFactory::createAopProxy`中自动决定使用哪种方式创建代理对象：

- JdkDynamicAopProxy
- ObjenesisCglibAopProxy

1. 给容器中返回cglib增强了的代理对象
2. 以后容器中获取到的就是这个组件的代理对象，执行目标方法时，代理对象就会执行通知方法的流程。

## MethodInterceptor方法拦截器链

目标方法执行：

容器中保存了组件的代理对象（Cglib增强后的对象），这个对象里面保存了详细信息（增强器、目标对象等）。

1. 进入CglibAopProxy的intercept()方法，拦截目标方法的执行
2. 根据`ProxyFactory`类型的`advised`对象，获取到将要执行的目标方法的拦截器链

拦截器链：每一个通知方法被包装成MethodInterceptor方法拦截器，后续利用MethodInterceptor机制执行

```java
List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
```

1. `List<Object> interceptorList`保存所有拦截器一个默认的`ExposeInvocationInterceptor` + 自定义的增强通知方法
2. 遍历所有的增强器，将其转换为Interceptor：`registry.getInterceptors(advisor);`将Advisor转换为MethodInterceptor（MethodInterceptor继承于Interceptor）：判断如果Advisor属于MethodInterceptor类型，则直接加入interceptors数组；通过遍历AdvisorAdapter适配器（前置通知适配器、返回结果通知适配器、异常通知适配器），尝试将Advisor转换为MethodInterceptor，加入interceptors数组

`@AfterThrowing`注解的增强器转换为`AspectJAfterThrowingAdvice`

`@AfterReturning`注解的增强器转换为`AfterReturningAdviceInterceptor`

`@After`注解的增强器转换为`AspectJAfterAdvice`

`@Before`注解的增强器转换为`MethodBeforeAdviceInterceptor`

1. 如果没有拦截器链，直接执行目标方法
2. 如果有拦截器链，把需要执行的目标对象、目标方法、拦截器链等信息传入创建一个CglibMethodInvocation对象，并调用该对象的`proceed()`方法。

```java
Object retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
```

拦截器链的调用过程：`proceed()`的执行过程。

`CglibMethodInvocation`继承于`ReflectiveMethodInvocation`，实际执行的是`ReflectiveMethodInvocation::proceed()`

1. 如果没有拦截器执行目标方法，或者拦截器的索引和（拦截器数组-1）大小一样（即到了最后一个拦截器），执行目标方法。

因为拦截器的链式执行是类似入栈操作，是后入先出，所以拦截器链数组中的拦截器顺序与执行顺序相反

1. 否则获取到拦截器链中（当前索引 += 1）的拦截器，执行其`invoke()`方法
2. 拦截器的`invoke()`方法，会在进行一定操作后，递归链式调用`proceed()`方法。例如`@Before`增强器对应的`MethodBeforeAdviceInterceptor`拦截器，其`invoke()`方法为：`@AfterReturning`增强器对应的`AfterReturningAdviceInterceptor`拦截器，其`invoke()`方法为：

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis() );
    return mi.proceed();  // 链式调用，继续进行后面的链的执行
}
```

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    Object retVal = mi.proceed(); // 链式调用，继续进行后面的链的执行
    this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
    return retVal;
}
```

## AOP增强器注解执行顺序

根据注解进行排序，顺序依次为：@Around、@Before、@After、@AfterReturning、[@AfterThrowing](notion://www.notion.so/AfterThrowing)

`AbstractAutoProxyCreator`执行`postProcessBeforeInstantiation()`，其中进行`shouldSkip()`等判断时，会调用`findCandidateAdvisors()`获取所有候选的增强器。

`AnnotationAwareAspectJAutoProxyCreator`中实现了`findCandidateAdvisors()`方法。`AnnotationAwareAspectJAutoProxyCreator`通过调用`this.aspectJAdvisorsBuilder.buildAspectJAdvisors()`获取所有的增强器，`aspectJAdvisorsBuilder.buildAspectJAdvisors()`通过`advisorFactory.getAdvisors(factory)`获取增强器。

aspectJAdvisorsBuilder、advisorFactory都是在`AnnotationAwareAspectJAutoProxyCreator`的`initBeanFactory()`方法中实例化出来。

```java
@Override
protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	if (this.aspectJAdvisorFactory == null) {
            this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
        }
        this.aspectJAdvisorsBuilder =
                new BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory,
                                                             this.aspectJAdvisorFactory);
}
```

所以，最终获取方法增强器并进行排序 是在`ReflectiveAspectJAdvisorFactory`类中进行：

![https://cdn.nlark.com/yuque/__mermaid_v3/a210f063f47c4331c1140f9cfc006741.svg](https://cdn.nlark.com/yuque/__mermaid_v3/a210f063f47c4331c1140f9cfc006741.svg)

即最后调用`Collections.sort(增强器, METHOD_COMPARATOR)`对所有增强器进行排序，排序算法为`METHOD_COMPARATOR`对象。

METHOD_COMPARATOR对象的定义赋值源码：

```java
private static final Comparator<Method> METHOD_COMPARATOR;

static {
    CompoundComparator<Method> comparator = new CompoundComparator<Method>();
    // 首选排序方式：根据注解进行排序，顺序依次为：@Around、@Before、@After、@AfterReturning、@AfterThrowing
    comparator.addComparator(new ConvertingComparator<Method, Annotation>(
        new InstanceComparator<Annotation>(
            Around.class, Before.class, After.class, AfterReturning.class, AfterThrowing.class),
        new Converter<Method, Annotation>() {
            @Override
            public Annotation convert(Method method) {
                AspectJAnnotation<?> annotation =
                    AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(method);
                return (annotation != null ? annotation.getAnnotation() : null);
            }
        }));
    // 当两个增强器的注解相同时，继续用第二种排序方式进行比较：以字符串方式比较增强器的方法名称
    comparator.addComparator(new ConvertingComparator<Method, String>(
        new Converter<Method, String>() {
            @Override
            public String convert(Method method) {
                return method.getName();
            }
        }));
    METHOD_COMPARATOR = comparator;
}
```

所以，如果想自定义一个类似`ReflectiveAspectJAdvisorFactory`的类`MyReflectiveAspectJAdvisorFactory`，更改AOP注解排序方式，则需：

1. 自定义一个类似`EnableAspectJAutoProxy`的注解`MyEnableAspectJAutoProxy`，令其@Import注解加载的组件为自定义的组件`MyAspectJAutoProxyRegistrar`
2. 修改`MyAspectJAutoProxyRegistrar`中的`AopConfigUtiles`为自定义的`MyAopConfigUtiles`，因为`AopConfigUtiles`会将`AnnotationAwareAspectJAutoProxyCreator`加载为beanId为`internalAutoProxyCreator`的bean。以及修改判断的使能AspectJ注解为`MyEnableAspectJAutoProxy`
3. 修改`MyAopConfigUtiles`中`registerAspectJAnnotationAutoProxyCreatorIfNecessary()`方法加载的bean类为`MyAnnotationAwareAspectJAutoProxyCreator`
4. 修改`MyAnnotationAwareAspectJAutoProxyCreator`的`initBeanFactory()`方法中创建的aspectJAdvisorFactory对象为`new MyReflectiveAspectJAdvisorFactory(beanFactory)`
5. 在`MyReflectiveAspectJAdvisorFactory`中的静态代码块中修改成自己需要的排序方式

# 声明式事务使用

1. 导入依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
<!-- 根据需要，导入连接池、数据库驱动 -->
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>

<dependency>
    <groupId>com.oracle.ojdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.3.0.0</version>
</dependency>
```

1. 配置数据源、JdbcTemplate操作数据

```java
@Configuration
@ComponentScan({"com.study.dao", "com.study.service"})
public class TxConfig {
    @Bean
    public DataSource dataSource() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("myData");
        dataSource.setPassword("tiger");
        dataSource.setDriverClass("oracle.jdbc.OracleDriver");
        dataSource.setJdbcUrl("jdbc:oracle:thin:@localhost:1521:orcl");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() throws Exception {
         // @Configuration类中的@Bean方法在Spring中被特殊执行，
        // 此处虽然传入dataSource()，但是Spring不会执行dataSource()方法重新new一个对象，而是会直接从ioc容器中获取
        return new JdbcTemplate(dataSource());
    }
}
```

1. 给方法上标注`@Transactional`，表示当前方法是一个事务方法

```java
@Service
public class UserService {
    @Autowired
    private UserDao userDao;

    @Transactional
    public void insertUser() {
        userDao.insert();
        System.out.println("插入完成");
    }
}
```

1. `@EnableTransactionManagement` 开启基于注解的事务管理功能

```java
@EnableTransactionManagement
@Configuration
@ComponentScan({"com.study.dao", "com.study.service"})
public class TxConfig {
    // ....
}
```

1. 配置事务管理器来控制事务

```java
@Bean
public PlatformTransactionManager transactionManager() throws Exception{
    return new DataSourceTransactionManager(dataSource());
}
```

## @EnableTransactionManagement

`@EnableTransactionManagement`使用`@Import`引入了`TransactionManagementConfigurationSelector`。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.svg)

AdivceModelmportSelect中获取`@EnableTransactionManagement`注解的`mode`属性，传递给`selectImports(adviceMode)`方法。

TransactionManagementConfigurationSelector根据传入的mode值，选择引入不同的组件。

`@EnableTransactionManagement`的mode属性默认值为`AdviceMode.PROXY`，所以默认引入的组件为：

- AutoProxyRegistrar
- ProxyTransactionManagementConfiguration

## AutoProxyRegistrar

AutoProxyregistrar实现了ImportBeanDefinitionRegistrar，可以手动给容器中注册组件。

同AOP的`@EnableAspectJAutoProxy`注解的`AspectJAutoProxyRegistrar`注册器一样，AutoProxyregistrar也是使用`AopConfigUtils`工具类注册组件。

调用`AopConfigUtils::registerAutoProxyCreatorIfNecessary`方法，注册了一个`InfrastructureAdvisorAutoProxyCreator`类型的组件，beanID同样是`org.springframework.aop.config.internalAutoProxyCreator`。

`InfrastructureAdvisorAutoProxyCreator`也继承了`AbstractAdvisorAutoProxyCreator`，所以其执行逻辑同AOP基本一样。

## ProxyTransactionManagementConfiguration

是一个`@Configuration`注解的配置类，内部通过`@Bean`给ioc容器注册相应组件。

1. transactionAdvisor() 给容器中注册事务增强器

- 需要用事务注解信息：AnnotationTransactionAttributeSource，解析事务注解属性。AnnotationTransactionAttributeSource中会添加需要的相关的注解解析器，例如：Spring事务注解解析器SpringTransactionAnnotationParser、Jta12注解解析器JtaTransactionAnnotationParser、Ejb3注解解析器Ejb3TransactionAnnotationParser。
- 需要用事务拦截器：TransactionInterceptor，保存了事务属性信息、事务管理器。TransactionInterceptor是一个方法拦截器MethodInterceptor。

1. transactionAttributeSource() 给容器中注册事务注解器，供事务增强器使用
2. transactionInterceptor() 给容器中注册事务拦截器，供事务增强器使用

## TransactionInterceptor

TransactionInterceptor的类UML图如下：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%201.svg)

TransactionInterceptor是一个MethodInterceptor，和其他方法拦截器一样，会在执行拦截器链时，通过上层的`proceed()`调用到它的`invoke()`方法，`TransactionInterceptor::invoke()`调用了父类的`TransactionAspectSupport::invokeWithinTrasaction()`：

1. 获取事务相关的属性
2. 获取`PlatformTransactionManager`类型事务管理器：`determineTransactionManager()`通过`@Transactional`注解的`transactionManager`属性尝试获取对应beanId的事务管理器(该属性作用类似`Qualifier`)；如果transactionManager属性为空（一般情况下不设置该属性），继续判断类属性`transactionManagerBeanName`；如果没有给该类事先添加事务管理器，最终通过ioc容器获取事务管理器。

```java
defaultTransactionManager = this.beanFactory.getBean(PlatformTransactionManager.class);
```

1. 使用事务管理器开启事务、执行方法、事务提交或回滚

```java
// 开启事务
TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
Object retVal = null;
try {
    // 拦截器链调用方法执行
    retVal = invocation.proceedWithInvocation();
}
catch (Throwable ex) {
    // 事务管理器进行回滚：txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus())
    // completeTransactionAfterThrowing方法内有判断：只有符合回滚条件的异常才执行回滚，其他异常会继续执行commit
    completeTransactionAfterThrowing(txInfo, ex);
    throw ex;
}
finally {
    cleanupTransactionInfo(txInfo);
}
// 事务管理器提交：txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
commitTransactionAfterReturning(txInfo);
return retVal;
```

因为Spring的事务管理器仅对指定的异常进行事务回滚，有时我们需要在Service中强制进行回滚时，会使用以下方式：

```java
TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
```

原理：

TransactionAspectSupport中使用`ThreadLocal<TransactionInfo>`存储了该线程的事务，可以通过protect的`currentTransactionStatus()`方法获取到该事务；

TransactionInfo事务中带有一个状态属性TransactionStatus，事务管理器提交/回滚事务时都会判断该状态：`txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());`。

如果将TransactionStatus的rollbackOnly属性设置为true，则事务管理器的`commit(TransactionStatus transtationStatus)`方法执行时判断到该状态，会执行事务回滚操作。

# 扩展原理

## 著配置类bean相关注解解析

给Spring的ioc容器传入主配置类，Spring需要通过主配置类上的`@PropertySources`、`@ComponentScan`等注解去加载相关的组件。对这些注解的解析在`ConfigurationClassParser`类中：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%202.svg)

## `BeanFactoryPostProcessor`

> `*BeanPostProcessor：bean后置处理器，bean创建对象初始化前后进行拦截工作的*`

BeanFactoryPostProcessor：beanFactory的后置处理器；接口方法：postProcessBeanFactory()。

- 调用时机在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容；
- 所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建

创建配置类

```java
@ComponentScan("com.rzf.annotation.ext")
@Configuration
public class ExtConfig {

    @Bean
    public Blue blue(){
        return new Blue();
    }
}
```

创建工厂后置处理器

```java
/**
 * bean工厂的后置处理器
 */
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    /**
     * 在bean定义已经被加载，但是bean实例对象还没有被创建的时候调用
     * @param beanFactory
     * @throws BeansException
     */
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        System.out.println("MyBeanFactoryPostProcessor 被执行.....................");
        int beanDefinitionCount = beanFactory.getBeanDefinitionCount();
        String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
        System.out.println("当前beanFactory中bean定义的个数："+beanDefinitionCount);
        for(String name:beanDefinitionNames){
            System.out.println(name);
        }

    }
}

//输出结果
MyBeanFactoryPostProcessor 被执行.....................
当前beanFactory中bean定义的个数：8
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
extConfig
myBeanFactoryPostProcessor
blue

Blue constructor.....
```

原理

BeanFactoryPostProcessor原理:
1)、ioc容器创建对象

2）、调用refresh()执行刷新操作；

- 方法内部调用invokeBeanFactoryPostProcessors(beanFactory);如何找到所有的BeanFactoryPostProcessor并执行他们的方法；
    - 直接在BeanFactory中找到所有类型是BeanFactoryPostProcessor的组件，并执行他们的方法。
    - 在初始化创建其他组件前面执行。`finishBeanFactoryInitialization(beanFactory);`执行初始化实例的方法。bean工厂的执行在这个方法前面。

## `*BeanDefinitionRegistryPostProcessor*`

`*BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor`，*增加了一个方法`postProcessBeanDefinitionRegistry()`

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

	/**
	 * Modify the application context's internal bean definition registry after its
	 * standard initialization. All regular bean definitions will have been loaded,
	 * but no beans will have been instantiated yet. This allows for adding further
	 * bean definitions before the next post-processing phase kicks in.
	 * @param registry the bean definition registry used by the application context
	 * @throws org.springframework.beans.BeansException in case of errors
	 */
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;

}
//接口
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
	 * Modify the application context's internal bean factory after its standard
	 * initialization. All bean definitions will have been loaded, but no beans
	 * will have been instantiated yet. This allows for overriding or adding
	 * properties even to eager-initializing beans.
	 * @param beanFactory the bean factory used by the application context
	 * @throws org.springframework.beans.BeansException in case of errors
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

postProcessBeanDefinitionRegistry()方法执行时机;

- 在所有bean定义信息将要被加载，bean实例还未创建的；
- 他的执行时机在BeanFactoryPostProcessor的postProcessBeanFactory`()`方法之前，因为BeanFactoryPostProcessor的定义是所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建。bean的定义已经保存加载。

`BeanDefinitionRegistry`：bean定义信息的保存中心，以后BeanFactory就是按照`BeanDefinitionRegistry里面保存的每一个bean定义的信息去创建bean实例。`

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        String[] beanDefinitionNames = registry.getBeanDefinitionNames();
        System.out.println("postProcessBeanDefinitionRegistry......bean的数量是："+registry.getBeanDefinitionCount());
        System.out.println(Arrays.toString(beanDefinitionNames));
//        给容器中注册一个对象
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Blue.class);
        registry.registerBeanDefinition("blue对象注册",rootBeanDefinition);

    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        String[] names = beanFactory.getBeanDefinitionNames();
        System.out.println("postProcessBeanFactory方法执行...........bean的数量："+beanFactory.getBeanDefinitionCount());
        System.out.println(Arrays.toString(names));

    }
}

//结果
postProcessBeanDefinitionRegistry......bean的数量是：9
[org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory, extConfig, myBeanDefinitionRegistryPostProcessor, myBeanFactoryPostProcessor, blue]
postProcessBeanFactory方法执行...........bean的数量：10
[org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory, extConfig, myBeanDefinitionRegistryPostProcessor, myBeanFactoryPostProcessor, blue, blue对象注册]
MyBeanFactoryPostProcessor 被执行.....................
当前beanFactory中bean定义的个数：10
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
extConfig
myBeanDefinitionRegistryPostProcessor
myBeanFactoryPostProcessor
blue
blue对象注册
Blue constructor.....
Blue constructor.....
```

`BeanDefinitionRegistryPostProcessor类里面的方法执行完成后，才执行BeanFactoryPostProcessor类里面的方法。`

- postProcessBeanDefinitionRegistry();
    - 在所有bean定义信息将要被加载，bean实例还未创建的；
    - 优先于BeanFactoryPostProcessor执行；
    - 利用BeanDefinitionRegistryPostProcessor给容器中再额外添加一些组件；

原理：

- 原理：
- ioc创建对象
- refresh()-》invokeBeanFactoryPostProcessors(beanFactory);
- 从容器中获取到所有的BeanDefinitionRegistryPostProcessor组件。
    - 依次触发所有的postProcessBeanDefinitionRegistry()方法
    - 再来触发postProcessBeanFactory()方法BeanFactoryPostProcessor；
- 再来从容器中找到BeanFactoryPostProcessor组件；然后依次触发postProcessBeanFactory()方法

## `*ApplicationListener`应用监听器*

应用监听器，spring为我们提供的基于事件驱动开发的功能，监听器通过监听容器中发布的一些事件，只要事件发生，触发监听器的回调来完成事件驱动开发。

ApplicationListener：监听容器中发布的事件。事件驱动模型开发；

```java
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
	void onApplicationEvent(E event);
}  
```

如果自定义监听器，就要实现ApplicationListener实现类，泛型就是要监听的事件类型。

监听 ApplicationEvent 及其下面的子事件；

```java
@Component
public class MyApplicationLintener implements ApplicationListener<ApplicationEvent> {
    /**
     * 当容器中发布此事件后，方法会触发执行
     * @param event
     */
    @Override
    public void onApplicationEvent(ApplicationEvent event) {

        System.out.println("收到事件......"+ event);
    }
}

//结果
D:\soft\jdk1.8\bin\java.exe -ea -Didea.test.cyclic.buffer.size=1048576 "-javaagent:D:\soft\IDEA\IntelliJ IDEA 2020.2.3\lib\idea_rt.jar=62761:D:\soft\IDEA\IntelliJ IDEA 2020.2.3\bin" -Dfile.encoding=UTF-8 -classpath "D:\soft\IDEA\IntelliJ IDEA 2020.2.3\lib\idea_rt.jar;D:\soft\IDEA\IntelliJ IDEA 2020.2.3\plugins\junit\lib\junit5-rt.jar;D:\soft\IDEA\IntelliJ IDEA 2020.2.3\plugins\junit\lib\junit-rt.jar;D:\soft\jdk1.8\jre\lib\charsets.jar;D:\soft\jdk1.8\jre\lib\deploy.jar;D:\soft\jdk1.8\jre\lib\ext\access-bridge-64.jar;D:\soft\jdk1.8\jre\lib\ext\cldrdata.jar;D:\soft\jdk1.8\jre\lib\ext\dnsns.jar;D:\soft\jdk1.8\jre\lib\ext\jaccess.jar;D:\soft\jdk1.8\jre\lib\ext\jfxrt.jar;D:\soft\jdk1.8\jre\lib\ext\localedata.jar;D:\soft\jdk1.8\jre\lib\ext\nashorn.jar;D:\soft\jdk1.8\jre\lib\ext\sunec.jar;D:\soft\jdk1.8\jre\lib\ext\sunjce_provider.jar;D:\soft\jdk1.8\jre\lib\ext\sunmscapi.jar;D:\soft\jdk1.8\jre\lib\ext\sunpkcs11.jar;D:\soft\jdk1.8\jre\lib\ext\zipfs.jar;D:\soft\jdk1.8\jre\lib\javaws.jar;D:\soft\jdk1.8\jre\lib\jce.jar;D:\soft\jdk1.8\jre\lib\jfr.jar;D:\soft\jdk1.8\jre\lib\jfxswt.jar;D:\soft\jdk1.8\jre\lib\jsse.jar;D:\soft\jdk1.8\jre\lib\management-agent.jar;D:\soft\jdk1.8\jre\lib\plugin.jar;D:\soft\jdk1.8\jre\lib\resources.jar;D:\soft\jdk1.8\jre\lib\rt.jar;D:\soft\IDEA\work\springboot2-master\spring-annotation\target\test-classes;D:\soft\IDEA\work\springboot2-master\spring-annotation\target\classes;C:\Users\MrR\.m2\repository\org\springframework\spring-context\5.1.9.RELEASE\spring-context-5.1.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-aop\5.1.9.RELEASE\spring-aop-5.1.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-beans\5.1.9.RELEASE\spring-beans-5.1.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-core\5.1.9.RELEASE\spring-core-5.1.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-jcl\5.1.9.RELEASE\spring-jcl-5.1.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-expression\5.1.9.RELEASE\spring-expression-5.1.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-aspects\5.3.1\spring-aspects-5.3.1.jar;C:\Users\MrR\.m2\repository\org\aspectj\aspectjweaver\1.9.6\aspectjweaver-1.9.6.jar;C:\Users\MrR\.m2\repository\org\projectlombok\lombok\1.16.20\lombok-1.16.20.jar;C:\Users\MrR\.m2\repository\junit\junit\4.13.2\junit-4.13.2.jar;C:\Users\MrR\.m2\repository\org\hamcrest\hamcrest-core\1.3\hamcrest-core-1.3.jar;C:\Users\MrR\.m2\repository\c3p0\c3p0\0.9.1.2\c3p0-0.9.1.2.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-jdbc\4.3.12.RELEASE\spring-jdbc-4.3.12.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-tx\4.3.12.RELEASE\spring-tx-4.3.12.RELEASE.jar;C:\Users\MrR\.m2\repository\mysql\mysql-connector-java\5.1.45\mysql-connector-java-5.1.45.jar" com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit4 com.rzf.test.ExtTest,test1
postProcessBeanDefinitionRegistry......bean的数量是：10
[org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory, extConfig, myApplicationLintener, myBeanDefinitionRegistryPostProcessor, myBeanFactoryPostProcessor, blue]
postProcessBeanFactory方法执行...........bean的数量：11
[org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory, extConfig, myApplicationLintener, myBeanDefinitionRegistryPostProcessor, myBeanFactoryPostProcessor, blue, blue对象注册]
MyBeanFactoryPostProcessor 被执行.....................
当前beanFactory中bean定义的个数：11
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
extConfig
myApplicationLintener
myBeanDefinitionRegistryPostProcessor
myBeanFactoryPostProcessor
blue
blue对象注册
Blue constructor.....
Blue constructor.....
收到事件......org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@2a3046da, started on Sat Sep 09 14:13:12 CST 2023]
收到事件......org.springframework.context.event.ContextClosedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@2a3046da, started on Sat Sep 09 14:13:12 CST 2023]

Process finished with exit code 0
```

可以看到返回的时间都是ApplicationEvent的事件。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%203.png)

基于事件开发的步骤：

步骤：
1）、写一个监听器（ApplicationListener实现类）来监听某个事件（ApplicationEvent及其子类）

- @EventListener;原理：使用EventListenerMethodProcessor处理器来解析方法上的@EventListener；EventListenerMethodProcessor实现了SmartInitializingSingleton 接口。

2）、把监听器加入到容器；(使用`@Compenent`)
3）、只要容器中有相关事件的发布，我们就能监听到这个事件；

- ContextRefreshedEvent：容器刷新完成（所有bean都完全创建）会发布这个事件；
- ContextClosedEvent：关闭容器会发布这个事件；

4）、发布一个事件：applicationContext.publishEvent()；

```java
public class ExtTest {

    @Test
    public void test1(){

//        1. 创建ioc容器,单实例对象，在创建容器的时候就会加载组件对象到容器中
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ExtConfig.class);

//        发布事件
        context.publishEvent(new ApplicationEvent(new String("发布事件")) {
        });
        context.close();
        
    }
}
//结果
收到事件......com.rzf.test.ExtTest$1[source=发布事件]
```

原理：

收到的三个事件：

- ContextRefreshedEvent
- IOCTest_Ext$1[source=我发布的时间]
- ContextClosedEvent

1. ContextRefreshedEvent事件：
    1. 容器创建对象：refresh()；
    2. 在容器刷新方法的最后一步，调用了`finishRefresh()`方法完成容器刷新
    3. 容器完成刷新的方法内发布容器刷新事件：publishEvent(new ContextRefreshedEvent(this));
2. 自己发布事件；
3. 容器关闭会发布ContextClosedEvent；

---

---

1. publishEvent()方法【事件发布流程】；
    1. publishEvent(new ContextRefreshedEvent(this));
    2. 获取事件的多播器（派发器）：getApplicationEventMulticaster()         
   3. multicastEvent派发事件：
    4. 获取到所有的ApplicationListener； for (final ApplicationListener<?> listener : getApplicationListeners(event, type))
    5. 果有Executor，可以支持使用Executor进行异步派发；Executor executor = getTaskExecutor();
    6. 否则，同步的方式直接执行listener方法；invokeListener(listener, event); 拿到listener回调onApplicationEvent方法；

```java
for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
    // 如果有Executor，可以使用多线程异步派发
    Executor executor = getTaskExecutor();
    if (executor != null) {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                invokeListener(listener, event);  // 执行listener里面的onApplicationEvent方法
            }
        });
    }
    else {  // 否则，同步方式直接执行listener里面的onApplicationEvent方法
        invokeListener(listener, event);
    }
}
```

【事件多播器（派发器）】

1. 容器创建对象：refresh();
2. `refresh()`方法中有一步调用initApplicationEventMulticaster();在创建bean之前会初始化ApplicationEventMulticaster；
    1. 先去容器中找有没有id=“applicationEventMulticaster”的组件；  
   2. 如果可以直接获取，那么就直接使用。      
   3. 如果没有this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);并且加入到容器中，我们就可以在其他组件要派发事件，自动注入这个applicationEventMulticaster；

【容器中有哪些监听器】

1. 容器创建对象：refresh();
2. registerListeners();注册监听器的作用
    1. 从容器中拿到所有的监听器，把他们注册到applicationEventMulticaster中；
    2. String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    3. 将listener注册到ApplicationEventMulticaster中 getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);

@EventListener：使用`@EventListener`注解监听事件。

标记在方法上，使用`classes`属性声明要监听的事件类型。ApplicationEvent类型的方法参数可以获得到该事件。

```java
@Service
public class UserService {

    @EventListener(classes = {ApplicationEvent.class})
    public void listener(ApplicationEvent event) {
        System.out.println("得到事件:" + event);
    }
}
```

使用`EventListenerMethodProcessor`处理器来解析方法上的`@EventListener`注解。

`EventListenerMethodProcessor`实现了`SmartInitializingSingleton`接口。

SmartInitializingSingleton

在所有单实例bean都创建完成之后调用，调用的时机类似`ContextRefreshedEvent`。接口方法：• afterSingletonsInstantiated()

- 原理：->afterSingletonsInstantiated();
    - ioc容器创建对象并refresh()；
    - refresh()调用`finishBeanFactoryInitialization()`
    - finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean；
        - 遍历所有待创建的单实例bean，调用`getBean()`创建所有的单实例bean
        - 获取所有创建好的单实例bean，判断是否是SmartInitializingSingleton类型的；如果是就调用afterSingletonsInstantiated();

# Spring容器创建过程

Spring的`refresh()`方法进行容器的创建和刷新，进入`AbstractApplicationcontext`类的`refresh()`方法中,源码如下：

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        // 1.进行容器的预处理
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        // 2.获取beanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        // 3.进行beanFactory的预处理工作
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            // 4.beanFactory准备工作完成之后的后置处理工作
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            // 5.执行BeanFactoryPostProcessors（包括BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor）
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            // 6.注册bean后置处理器，拦截bean的创建过程
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            // 7.初始化MessageSource，做国际化功能
            initMessageSource();

            // Initialize event multicaster for this context.
            // 8.初始化事件派发器
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            // 9.初始化其他的一些特殊Bean，默认为空实现，留给子类进行自定义重写
            onRefresh();

            // Check for listener beans and register them.
            // 10.将所有的ApplicationListener注册进容器中
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            // 11.初始化所有剩下的单实例bean
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

## 1、prepareRefresh()刷新前的预处理;

1. initPropertySources()初始化一些属性设置，空方法，一般留给子类实现;子类自定义个性化的属性设置方法；

> AbstractApplicationContext 类中的`initPropertySources()`方法是空的，留给子类进行实现。子类自定义个性化的属性设置方法。

1. 如果在上一步有自定义的属性配置，则调用getEnvironment().validateRequiredProperties();自定义属性设置后，检验属性的合法等
2. 创建List集合，保存容器中的一些早期的事件；

```java
this.earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>();
```

## 2、调用obtainFreshBeanFactory()获取BeanFactory工厂

1. 调用refreshBeanFactory();刷新【创建】BeanFactory；创建了一个

```java
this.beanFactory = new DefaultListableBeanFactory();
```

1. 设置序列id；
    1. getBeanFactory();返回子类刚才GenericApplicationContext创建的BeanFactory对象；
    2. 将创建的BeanFactory【DefaultListableBeanFactory】返回；此时的beanFactory刚创建，很多属性都是空的

## 3、prepareBeanFactory(beanFactory)

调用 `prepareBeanFactory(beanFactory);` 进行beanFactory的预准备工作，对beanFactory进行一些设置

1. 置BeanFactory的类加载器、支持表达式解析器...
2. 添加部分BeanPostProcessor【ApplicationContextAwareProcessor】添加部分后置处理器。

```java
beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
```

1. 设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；

```java
beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
```

1. 注册可以解析的自动装配。我们可以在任何组件中使用 @Autowired等注解自动注入以下组件。

```java
BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
beanFactory.registerResolvableDependency(ResourceLoader.class, this);
beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```

1. 继续添加后置处理器

```java
添加BeanPostProcessor【ApplicationListenerDetector】
beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
```

1. 添加编译时的AspectJ支持

我们使用的多是运行时的动态代理

1. 给beanFactory中注册一些环境组件

```java
给BeanFactory中注册一些能用的组件；
      environment【ConfigurableEnvironment】、
      systemProperties【Map<String, Object>】、
      systemEnvironment【Map<String, Object>】
if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {  // environment : StandardEnvironment
    beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
}
if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {  // systemProperties : (Map) System.getProperties()
    beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
}
if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) { // systemEnvironment : (Map) System.getenv()
    beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
}
```

## 4、postProcessBeanFactory(beanFactory)

BeanFactory准备工作完成后进行的后置处理工作；空方法，留给子类重写的接口。

AbstractApplicationContext中这个方法为空方法，子类可以重写这个方法，在beanFactory创建并预准备完成之后的后置处理工作。子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置。

---

以上是 `BeanFactory` 的创建和预准备工作
—-

---

## 5、invokeBeanFactoryPostProcessors(beanFactory)

调用 `invokeBeanFactoryPostProcessors(beanFactory)` ，执行 BeanFactoryPostProcessors。

BeanFactoryPostProcessor是BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的；标准初始化就是前面的四个步骤，两个接口：

- BeanFactoryPostProcessor
- BeanDefinitionRegistryPostProcessor

执行BeanFactoryPostProcessor的方法；

```java
// 此处传入的 AbstractApplicationContext的 getBeanFactoryPostProcessors() 默认是空的
PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
```

1. 定义 regularPostProcessors 集合用来存放普通的 BeanFactoryProcessors
2. 定义 registryProcessors 集合用来存放 BeanDefinitionRegistryPostProcessor 类型的 BeanFactoryProcessors
3. 如果传入的 AbstractApplicationContext.getBeanFactoryPostProcessors() 有内容，则先执行其中的 BeanDefinitionRegistryPostProcessor类型后置处理器的 `postProcessBeanDefinitionRegistry()` 方法。并把 beanFactoryPostProcessors 存入对应的集合中。
4. 获取ioc容器中的 BeanDefinitionRegistryPostProcessors 并执行 BeanDefinitionRegistryPostProcessor:获取所有的BeanDefinitionRegistryPostProcessor；
    1. 先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、postProcessor.postProcessBeanDefinitionRegistry(registry)并加入 registryProcessors 数组.
    2. 在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
       postProcessor.postProcessBeanDefinitionRegistry(registry)并加入 registryProcessors 数组
    3. 最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；并加入 registryProcessors 数组.

> 此处使用了循环获取，在获取一遍后会接着再获取下一遍，直到获取不到。猜测可能有些后置处理器的`postProcessBeanDefinitionRegistry`方法会再向容器添加BeanDefinitionRegistryPostProcessors.

1. 执行 registryProcessors  集合中的所有后置处理器的 `postProcessBeanFactory`方法
2. 执行 regularPostProcessors 集合中的所有BeanFactoryPostProcessors 的 `postProcessBeanFactory`方法
3. 获取 ioc 容器中的普通BeanFactoryPostProcessors并执行
    1. 获取容器中所有 BeanFactoryPostProcessors，并按照是否属于`PriorityOrdered`、是否属于`Ordered`、是否是无排序进行归类
    2. 执行所有`PriorityOrdered`类型的BeanFactoryPostProcessors 的`postProcessBeanFactory`方法
    3. 执行所有`Ordered`类型的BeanFactoryPostProcessors 的`postProcessBeanFactory`方法
    4. 执行所有无排序的BeanFactoryPostProcessors 的`postProcessBeanFactory`方法

## 6、registerBeanPostProcessors(beanFactory)

调用`registerBeanPostProcessors(beanFactory)`，注册BeanPostProcessor（Bean的后置处理器）【 intercept bean creation】

不同接口类型的BeanPostProcessor；在Bean创建前后的执行时机是不一样的

- BeanPostProcessor、
- DestructionAwareBeanPostProcessor、
- InstantiationAwareBeanPostProcessor、
- SmartInstantiationAwareBeanPostProcessor、
- MergedBeanDefinitionPostProcessor【internalPostProcessors】、

1. 获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
2. 先注册PriorityOrdered优先级接口的BeanPostProcessor；把每一个BeanPostProcessor；添加到BeanFactory beanFactory.addBeanPostProcessor(postProcessor);
3. 再注册Ordered接口的
4. 最后注册没有实现任何优先级接口的
5. 最终注册MergedBeanDefinitionPostProcessor；
6. 注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是applicationContext.addApplicationListener((ApplicationListener<?>) bean);

> `ApplicationListenerDetector`用来在Bean创建完成后检查是否为`ApplicationListener`，如果是则添加进容器的监听器中。

## 7、initMessageSource()

调用`initMessageSource()`，初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；

1. 获取BeanFactory
2. 看容器中是否有id为messageSource的，类型是MessageSource的组件
    1. 如果有赋值给messageSource；
    2. 如果没有自己创建一个DelegatingMessageSource；MessageSource：取出国际化配置文件中的某个key的值；能按照区域信息获取；
3. 把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注MessageSourcebeanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
   MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);

## 8、initApplicationEventMulticaster()

> 初始化事件派发器

1. 获取BeanFactory
2. 从BeanFactory中获取beanId类型为applicationEventMulticaster的ApplicationEventMulticaster事件派发器。
3. 如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
4. 如果从容器中获取不到，就会创建一个`SimpleApplicationEventMulticaster`类型的事件派发器，并添加进容器中
5. 调用`onRefresh()`初始化其他的一些特殊Bean，默认为空实现，留给子类进行自定义重写
6. 调用`registerListener()`，给容器中注册所有的ApplicationListener
    1. 从容器中获取所有的ApplicationListener
    2. 将每个监听器添加进事件派发器中

```java
getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
```

## 9、onRefresh();

留给子容器（子类）子类重写这个方法，在容器刷新的时候可以自定义逻辑；

调用`onRefresh()`初始化其他的一些特殊Bean，默认为空实现，留给子类进行自定义重写

## 10、registerListeners()

给容器中将所有项目里面的ApplicationListener注册进来；

1. 从容器中拿到所有的ApplicationListener
2. 将每个监听器添加到事件派发器中；

```java
getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
      
```

1. 派发之前步骤产生的事件；

## 11、finishBeanFactoryInitialization(beanFactory);

调用`finishBeanFactoryInitialization(beanFactory);`初始化所有剩下的单实例bean

1. beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean

    1. 获取容器中的所有Bean，依次进行初始化和创建对象

    2. 获取Bean的定义信息；RootBeanDefinition：RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);

    3. Bean不是抽象的，是单实例的，是懒加载；

        1. 判断如果是`FactoryBean`（是否是实现`FactoryBean`接口的工厂Bean），利用工厂方法创建对象

        2. 如果不是工厂Bean，调用`getBean(beanName)`创建对象

        3. `getBean(beanName)`调用`doGetBean()`创建对象

            1. 先从缓存中保存的单实例bean尝试获取，如果能获取到说明这个Bean之前被创建过（所有创建过的单实例bean都会被缓存起来）

           ```java
           /** Cache of singleton objects: bean name --> bean instance */
           private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
           ```

            1. 缓存中获取不到时，准备开始Bean的创建对象流程
            2. `markBeanAsCreated()`标记当前Bean已经被创建，防止多线程运行时同一个单实例bean被多次创建
            3. 获取bean的定义信息`RootBeanDefinition`
            4. `getDependsOn()`获取当前bean依赖的其他bean，如果有，按照`getBean()`的方式把依赖的bean创建出来
            5. 启动单实例bean的创建流程

           ```java
           sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
               @Override
               public Object getObject() throws BeansException {
                   try {
                       return createBean(beanName, mbd, args);
                   }
                   catch (BeansException ex) {
                       // Explicitly remove instance from singleton cache: It might have been put there
                       // eagerly by the creation process, to allow for circular reference resolution.
                       // Also remove any beans that received a temporary reference to the bean.
                       destroySingleton(beanName);
                       throw ex;
                   }
               }
           });
           bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
           ```

            1. createBean(beanName, mbd, args);

            2. Object bean = resolveBeforeInstantiation(beanName, mbdToUse);让BeanPostProcessor先拦截返回代理对象；【InstantiationAwareBeanPostProcessor】：提前执行；先触发：postProcessBeforeInstantiation()；如果有返回值：触发postProcessAfterInitialization()；

            3. 如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用4）

            4. Object beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean

                1. 【创建Bean实例】；createBeanInstance(beanName, mbd, args);利用工厂方法或者对象的构造器创建出Bean实例；

                2. applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);调用MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition(mbd, beanType, beanName);

                3. 【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
                   赋值之前：================

                    1. 拿到InstantiationAwareBeanPostProcessor后置处理器；postProcessAfterInstantiation()；

                    2. 拿到InstantiationAwareBeanPostProcessor后置处理器；                    postProcessPropertyValues()；

                       =====赋值之前：===

                    3. 应用Bean属性的值；为属性利用setter方法等进行赋值；                     applyPropertyValues(beanName, mbd, bw, pvs);

                4. 【Bean初始化】initializeBean(beanName, exposedObject, mbd);

                    1. 【执行Aware接口方法】invokeAwareMethods(beanName, bean);执行xxxAware接口的方法                   BeanNameAware\BeanClassLoaderAware\BeanFactoryAware
                    2. 【执行后置处理器初始化之前】applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);                       BeanPostProcessor.postProcessBeforeInitialization（）;
                    3. 【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);两种方法实现初始化功能，实现接口，注解指定初始化
                        1. 是否是InitializingBean接口的实现；执行接口规定的初始化；
                        2. 是否自定义初始化方法；
                    4. 【执行后置处理器初始化之后】applyBeanPostProcessorsAfterInitialization,BeanPostProcessor.postProcessAfterInitialization()；
                    5. 注册Bean的销毁方法；

                5. 将创建的Bean添加到缓存中singletonObjects；

                6. ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。。；

                7. map里面保存了所有组件信息,所有Bean都利用getBean创建完成以后；

                8. 检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行afterSingletonsInstantiated()；

## 12、finishRefresh()

完成BeanFactory的初始化创建工作；IOC容器就创建完成；

1. initLifecycleProcessor();初始化和生命周期有关的后置处理器；LifecycleProcessor,默认从容器中找是否有lifecycleProcessor的组件【LifecycleProcessor】；如果没有new DefaultLifecycleProcessor();
   加入到容器；写一个LifecycleProcessor的实现类，可以在BeanFactory
   void onRefresh();void onClose();
2. getLifecycleProcessor().onRefresh();
   拿到前面定义的生命周期处理器（BeanFactory）；回调onRefresh()
3. publishEvent(new ContextRefreshedEvent(this));发布容器刷新完成事件；

## 13、总结

1. Spring容器在启动的时候，先会保存所有注册进来的Bean的==定义信息==；
    1. `xml`注册`bean:<bean>`
    2. 注解注册`Bean:@Service、@Component、@Bean、xxx`

2. Spring容器会合适的时机创建这些Bean

    1. 用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
    2. 统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；

3. 后置处理器；BeanPostProcessor,增强bean的功能

    1. 每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
       AutowiredAnnotationBeanPostProcessor:处理自动注入
       AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
       xxx....
       增强的功能注解：
       AsyncAnnotationBeanPostProcessor
       ....

4. 事件驱动模型；

    1. ApplicationListener；事件监听；
    2. ApplicationEventMulticaster；事件派发：
    3. spring对各种bean的增强处理都是通过后置处理器实现的


​

```
Spring容器的refresh()【创建刷新】;容器的初始化和创建都在这个方法内部
方法源码：
   @Override
   public void refresh() throws BeansException, IllegalStateException {
      synchronized (this.startupShutdownMonitor) {
         // Prepare this context for refreshing.
         prepareRefresh();

         // Tell the subclass to refresh the internal bean factory.
         ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

         // Prepare the bean factory for use in this context.
         prepareBeanFactory(beanFactory);

         try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);
--------------------------- 以上是beanFactory的创建和与准备工作
            // Invoke factory processors registered as beans in the context.
            //bean工厂的后置处理器
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            //bean的后置处理器
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
         }

         catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
               logger.warn("Exception encountered during context initialization - " +
                     "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
         }

         finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
         }
      }
   }

1、prepareRefresh()刷新前的预处理;
2、obtainFreshBeanFactory();获取BeanFactory工厂；
3、prepareBeanFactory(beanFactory);对BeanFactory的预准备工作前面创建好beanFactory并且返回（BeanFactory进行一些设置）；
4、postProcessBeanFactory(beanFactory);
======================以上是BeanFactory的创建及预准备工作==================================
bean工厂的后置处理器
5、invokeBeanFactoryPostProcessors(beanFactory);执行BeanFactoryPostProcessor的方法；
   
6、registerBeanPostProcessors(beanFactory);注册BeanPostProcessor（Bean的后置处理器）【 intercept bean creation】
      
7、initMessageSource();初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
      
8、initApplicationEventMulticaster();初始化事件派发器；
     
9、onRefresh();留给子容器（子类）
    
10、registerListeners();给容器中将所有项目里面的ApplicationListener注册进来；
      1、从容器中拿到所有的ApplicationListener
      2、将每个监听器添加到事件派发器中；
         getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
      3、派发之前步骤产生的事件；
11、finishBeanFactoryInitialization(beanFactory);初始化所有剩下的单实例bean；
   
12、finishRefresh();完成BeanFactory的初始化创建工作；IOC容器就创建完成；
     
   ======总结===========
   1）、Spring容器在启动的时候，先会保存所有注册进来的Bean的==定义信息==；
      1）、xml注册bean；<bean>
      2）、注解注册Bean；@Service、@Component、@Bean、xxx
   2）、Spring容器会合适的时机创建这些Bean
      1）、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
      2）、统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
   3）、后置处理器；BeanPostProcessor,增强bean的功能
      1）、每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
         AutowiredAnnotationBeanPostProcessor:处理自动注入
         AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
         xxx....
         增强的功能注解：
         AsyncAnnotationBeanPostProcessor
         ....
   4）、事件驱动模型；
      ApplicationListener；事件监听；
      ApplicationEventMulticaster；事件派发：
spring对各种bean的增强处理都是通过后置处理器实现的
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%204.png)