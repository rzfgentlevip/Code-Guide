---
title: SpringBoot原理
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
permalink: /learning-notes/kx0hoq28/
---


# SpringBoot原理

# 1、Profile功能

springboot高级特性：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.png)

为了方便多环境适配，springboot简化了profile功能。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%201.png)

## 1.1、application-profile功能

- 默认配置文件 application.yaml；任何时候都会加载。激活的配置文件也会加载，如果两个配置文件发生冲突，会以激活的配置文件内容为准。
- 指定环境配置文件 application-{env}.yaml
    - prod：生产环境
    - test:测试环境
- 激活指定环境：`spring.profiles.active=myprod`
    - 配置文件激活
    - 命令行激活：java -jar xxx.jar --**spring.profiles.active=prod --person.name=haha**
        - 可以**修改配置文件的任意值，命令行优先**
- 默认配置与环境配置同时生效
- 同名配置项，profile配置优先

> profile不仅可以标注在方法上，还可以标注在类上，标注在类上，只有我们制定了环境后，类中所有的配置才会生效，标注在方法上，只有我们指定了环境，方法的组件才会生效。

## 1.2、@Profile条件装配功能

```java
public interface Person {

   String getName();
   Integer getAge();

}

标注在类上
@Profile("test")
@Component
//绑定配置文件中的person
@ConfigurationProperties("person")
@Data
public class Worker implements Person {

    private String name;
    private Integer age;
}

@Profile(value = {"prod","default"})
@Component
//绑定配置文件中的person
@ConfigurationProperties("person")
@Data
public class Boss implements Person {

    private String name;
    private Integer age;
}

标注在方法上
@Configuration
public class MyConfig {

    @Profile("prod")
    @Bean
    public Color red(){
        return new Color();
    }

    @Profile("test")
    @Bean
    public Color green(){
        return new Color();
    }
}
```

## 1.3、profile分组

```xml
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq

使用：--spring.profiles.active=production  激活
```

# 2、外部化配置

[https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)

1. Default properties (specified by setting `SpringApplication.setDefaultProperties`).
2. [`@PropertySource`](https://docs.spring.io/spring/docs/5.3.1/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes. Please note that such property sources are not added to the `Environment` until the application context is being refreshed. This is too late to configure certain properties such as `logging.*` and `spring.main.*` which are read before refresh begins.
3. **Config data (such as `application.properties` files)**
4. A `RandomValuePropertySource` that has properties only in `random.*`.
5. OS environment variables.
6. Java System properties (`System.getProperties()`).
7. JNDI attributes from `java:comp/env`.
8. `ServletContext` init parameters.
9. `ServletConfig` init parameters.
10. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
11. Command line arguments.
12. `properties` attribute on your tests. Available on [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.4.0/api/org/springframework/boot/test/context/SpringBootTest.html) and the [test annotations for testing a particular slice of your application](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests).
13. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.3.1/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.
14. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings) in the `$HOME/.config/spring-boot` directory when devtools is active.

## 2.1、外部配置源

常用：**Java属性文件**、**YAML文件**、**环境变量**、**命令行参数**；

> 简单来说就是将所有组件的配置信息抽取出来，放到一个文件中自动读取，集中管理。在springboot中，所有的配置信息都配置在application.properties,通常用ymal配置文件，即application.ymal。

## 2.2、配置文件查找位置

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%202.png)

(1) classpath 根路径，类路径指的是源码包下面的路径，main下面的都是类路径。

(2) classpath 根路径下config目录

(3) jar包当前目录

(4) jar包当前目录的config目录

(5) /config子目录的直接子目录

> 后面的.properties或者.ymal文件会覆盖前面路径下的配置文件。

## 2.3、配置文件加载顺序：

1. 当前jar包内部的application.properties和application.yml
2. 当前jar包内部的application-{profile}.properties 和 application-{profile}.yml
3. 引用的外部jar包的application.properties和application.yml
4. 引用的外部jar包的application-{profile}.properties 和 application-{profile}.yml

## 2.4、指定环境优先，目录外面的配置外部优先，后面的可以覆盖前面的同名配置项

# 3、自定义starter

想要别人引用我们的场景，首先必须有一个场景启动器（***-starter），场景启动器里面没有源码，场景启动器一般是说明当前场景中引入的依赖有哪些，定义的starter引入了当前场景的自动配置包，而自动配置包又引入了springboot最底层的spring-boot-starter。

## 3.1、starter启动原理

starter-pom引入 autoconfigurer 包

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1606995919308-b2c7ccaa-e720-4cc5-9801-2e170b3102e1.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1606995919308-b2c7ccaa-e720-4cc5-9801-2e170b3102e1.png)

- autoconfigure包中配置使用 **META-INF/spring.factories** 中 **EnableAutoConfiguration 的值，使得项目启动加载指定的自动配置类。**
- **编写自动配置类 xxxAutoConfiguration -> xxxxProperties，然后和配置文件绑定。**
- **@Configuration**
- **@Conditional**
- **@EnableConfigurationProperties**
- **@Bean**
- ......

**引入starter --- xxxAutoConfiguration --- 容器中放入组件 ---- 绑定xxxProperties ---- 配置项**

## 3.2、自定义starter

目录结构

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%203.png)

### 3.2.1、定义场景启动器包

**atguigu-hello-spring-boot-starter（启动器）**

```xml
pom文件内容
<!--    场景启动器第一件事就是引入自动配置包，对所有功能的自动配置都写在自动配置项目里面-->
    <dependencies>
        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>atguigu-hello-spring-boot-starter-autoconfigure</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

### 3.2.2、自动配置

**atguigu-hello-spring-boot-starter-autoconfigure（自动配置包）**

```xml
pom文件
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.atguigu</groupId>
    <artifactId>atguigu-hello-spring-boot-starter-autoconfigure</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>atguigu-hello-spring-boot-starter-autoconfigure</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

    </dependencies>
```

在service中定义公共的逻辑功能：

```java
/**
 * 默认不要放在容器中，自动配置，也就是这个组件不一定要放在容器中，使用自动配置类，按需配置
 */
public class HelloService {

    @Autowired
    HelloProperties helloProperties;

    public String sayHello(String userName){
        return helloProperties.getPrefix() + "："+userName+"》"+helloProperties.getSuffix();
    }
}
```

定义配置属性类：

```java
/**
 * 属性类，和我们的配置文件进行绑定
 */
@ConfigurationProperties("atguigu.hello")
public class HelloProperties {

    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

定义自动配置文件，在自动配置文件中开启配置类和配置绑定功能。

```text
//指明是一个配置类
@Configuration
//在这里还可以进行条件装配
//开启属性文件绑定功能，就是我们的配置和配置类进行绑定，这样的话配置类会自动和配置文件进行绑定，并且会自动放到容器中（绑定和放容器）
//如果这里不开启，那么配置文件和配置没有绑定，也灭有放到容器中，这里就拿不到HelloServic e组件
@EnableConfigurationProperties(HelloProperties.class)  //默认HelloProperties放在容器中
public class HelloServiceAutoConfiguration{

//    条件装配，可以添加在类和方法上，如果放在方法上没那么自动配置类还会加载----------------
    @ConditionalOnMissingBean(HelloService.class)
    @Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        return helloService;
    }

}
```

写完场景启动器之后，因为starter依赖autoconfigure,所以先对autoconfigure进行clean为然后install，然后对starter进行clean，install放到本地仓库中，最后在别的项目中就可以引入这个场景启动器。

最后在自动配置项目的resources文件夹下创建META-INF文件夹，然后添加spring.factories文件，在里面配置自动加载的配置文件：这样项目已启动，就会自动加载配置文件中的组件到容器中。

```xml
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
--指定自动加载的配置文件
com.atguigu.hello.auto.HelloServiceAutoConfiguration
```

### 3.2.3、使用自定义场景启动器

引入自己写的场景启动器：

```xml
<!--        在这里引入场景启动器-->
        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>atguigu-hello-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%204.png)

```java
在配置文件中，配置参数

@RestController
public class HelloController {

    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public String sayHello(){

        String s = helloService.sayHello("张三");
        return s;
    }
}

在使用的时候，我们可以自己定义配置文件，在配置中修改其他属性，这样就不会使用启动器中配置的属性。
@Configuration
public class MyConfig {

    @Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
//在这里可以修改其他的属性
        return helloService;
    }
}
因为容器已经将自己定义的配置进行加载，容器中已经存在HelloService 对象，那么启动器中的配置就不会自动进行加载
因为配置的是：@ConditionalOnMissingBean(HelloService.class)，容器中没有才加载。
```

当场景启动时；会自动加载spring.factories文件中配置的自动配置文件：`HelloServiceAutoConfiguration`，这样自动配置类就被加载到容器中了。

在自动配置类中，通过条件架子啊指明，当容器中没有`*@ConditionalOnMissingBean*(HelloService.*class*)，HelloService.*class组件是，就`将*`HelloService`组件加载到容器中，而`HelloService组件是和HelloProperties组件绑定的，HelloProperties组件是自动注入的，这样我们就可以通过修改配置文件，`达到配置的目的。

# 4、SpringBoot原理

> Spring原理【[Spring注解](https://www.bilibili.com/video/BV1gW411W7wy?p=1)】(spring4)、**SpringMVC**原理(spring5)、**自动配置原理**、SpringBoot原理。

## 4.1、SpringBoot启动过程

```text
- 创建 **SpringApplication**
    - 首先进入run()方法，new一个class然后把我们的主程序类放进来。
    - 进入run()方法后
        - 第一步，首先创建应用，`*new* SpringApplication(primarySources)`
        - 第二步，然后调用`.run(args)`方法，将创建的主程序应用跑起来，并将主程序的参数传入run()后执行run()方法。
    - 创建`SpringApplication(primarySources)`的时候，首先调用有参构造函数，做初始化工作：`*public* SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources)`。
        - 方法内部首先保存资源加载器：`*this*.resourceLoader = resourceLoader;`
        - 执行断言
        - 保存主配置类信息。
        - 判断当前web应用的类型，判定当前应用的类型。ClassUtils—→Servlet
        - bootstrappers：初始启动引导器（List<Bootstrapper>类型）:去spring.factories文件中找 org.springframework.boot.Bootstrapper
            - `getSpringFactoriesInstances(Bootstrapper.class)`
                - `getSpringFactoriesInstances(type, *new* Class<?>[] {})`
                - `SpringFactoriesLoader.loadFactoryNames(type, classLoader)`这个类表示去spring.factories文件中去加载自动配置类信息。
                - `createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names)`：创建自动配置类的实例对象然后返回。
    - `setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.*class*));`找初始化器：
        - 找 **ApplicationContextInitializer**；去**spring.factories找 ApplicationContextInitializer初始化器，**`getSpringFactoriesInstances`（重点方法）这个方法就是去**spring.factories文件中找类实例。如果找到，就会调用set()方法进行保存到**`Initializer属性中。`
        - 保存的数据结构：List<ApplicationContextInitializer<?>> **initializers**
    - **找 ApplicationListener ；应用监听器。**去**spring.factories找 ApplicationListener**
        - List<ApplicationListener<?>> **listeners**
    - `deduceMainApplicationClass()`：决定哪一个是我们的主程序
        - 判断逻辑是找到堆栈，哪一个又main方法，哪一个就是主程序。`"main".equals(stackTraceElement.getMethodName())`

springapplication应用的创建，就是将一些关键的组件读取出来提前保存在容器中。
```

```java
重点方法
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
		return getSpringFactoriesInstances(type, new Class<?>[] {});
	}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}

// 实例化对象
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
			ClassLoader classLoader, Object[] args, Set<String> names) {
		List<T> instances = new ArrayList<>(names.size());
		for (String name : names) {
			try {
				Class<?> instanceClass = ClassUtils.forName(name, classLoader);
				Assert.isAssignable(type, instanceClass);
				Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
				T instance = (T) BeanUtils.instantiateClass(constructor, args);
				instances.add(instance);
			}
			catch (Throwable ex) {
				throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
			}
		}
		return instances;
	}
```

- 运行 **SpringApplication**
    - **StopWatch：记录应用的启动时间**
    - **创建引导上下文（Context环境）createBootstrapContext()，xxxContext()都是引导环境的上下文。**
    - 获取到所有之前的 **bootstrappers 挨个执行** intitialize() 来完成对引导启动器上下文环境设置，这里如果自定义了引导启动器，那么在这里会逐个遍历引导启动器，每一个引导启动器都会调用它的初始化设置方法。
    - 让当前应用进入**headless**模式。**java.awt.headless。**
    - **获取所有 RunListener（运行监听器）【为了方便所有Listener进行事件感知】**
        - getSpringFactoriesInstances 去**spring.factories找 SpringApplicationRunListener，然后保存在**`SpringApplicationRunListeners对象中。`
    - 遍历 **SpringApplicationRunListener 调用 starting 方法；**
        - **相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。**
    - 保存命令行参数；`*new* DefaultApplicationArguments(args);`
    - 准备环境 prepareEnvironment（）;
        - 返回或者创建基础环境信息对象。**StandardServletEnvironment**
        - **配置环境信息对象。**
            - **读取所有的配置源的配置属性值。**
        - 绑定环境信息
        - 监听器调用 listener.environmentPrepared()；通知所有的监听器当前环境准备完成
    - 创建IOC容器（`context = createApplicationContext();`）
        - 根据项目类型（Servlet）创建容器，
        - 当前会创建 **AnnotationConfigServletWebServerApplicationContext，**`*return this*.applicationContextFactory.create(*this*.webApplicationType);`
    - **准备ApplicationContext IOC容器的基本信息**  **prepareContext()**
        - 保存环境信息
        - IOC容器的后置处理流程。`postProcessApplicationContext(context);`
        - 应用初始化器；applyInitializers；`applyInitializers(context);`
            - 遍历所有的 **ApplicationContextInitializer 。调用 initialize.。来对ioc容器进行初始化扩展功能，在创建程序的时候进行了基础的初始化。**
            - 遍历所有的 listener 调用 **contextPrepared。EventPublishRunListenr；通知所有的监听器contextPrepared，**`listeners.contextPrepared(context);`
        - **所有的监听器 调用 contextLoaded。通知所有的监听器 contextLoaded；**`listeners.contextLoaded(context);`
    - **刷新IOC容器。**`refreshContext(context)`，其核心过程就是创建容器中的所有组件。
        - `refresh((ApplicationContext) context);`
        - `refresh((ConfigurableApplicationContext) applicationContext);`
        - `applicationContext.refresh();`
        - `*super*.refresh();`核心过程方法：

```java
spring启动ioc容器的完整过程
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();

				// Initialize message source for this context.
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
				contextRefresh.end();
			}
		}
	}
```

- 创建容器中的所有组件（Spring注解）核心过程
    - 容器刷新完成后工作？afterRefresh
        - 所有监听器调用 listeners.**started**(context); **通知所有的监听器 started**
    - **调用所有runners；**`callRunners(context, applicationArguments);`
        - **获取容器中的** **ApplicationRunner**
        - **获取容器中的 CommandLineRunner**
        - **合并所有runner并且按照@Order进行排序**
        - **遍历所有的runner。调用 run 方法**
        - **如果以上有异常，**
            - **调用Listener 的 failed**
    - **调用所有监听器的 running 方法** `listeners.running(context);`; **通知所有的监听器 running**
        - **running如果有问题。继续通知 failed 。调用所有 Listener 的 failed；通知所有的监听器 failed**
    - `*return* context;`返回容器

```java
public interface Bootstrapper {

	/**
	 * Initialize the given {@link BootstrapRegistry} with any required registrations.
	 * @param registry the registry to initialize
	 */
	void intitialize(BootstrapRegistry registry);

}
```

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1607005958877-bf152e3e-4d2d-42b6-a08c-ceef9870f3b6.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_18%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1607005958877-bf152e3e-4d2d-42b6-a08c-ceef9870f3b6.png)

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1607004823889-8373cea4-6305-40c1-af3b-921b071a28a8.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1607004823889-8373cea4-6305-40c1-af3b-921b071a28a8.png)

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1607006112013-6ed5c0a0-3e02-4bf1-bdb7-423e0a0b3f3c.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_18%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1607006112013-6ed5c0a0-3e02-4bf1-bdb7-423e0a0b3f3c.png)

```java
@FunctionalInterface
public interface ApplicationRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming application arguments
	 * @throws Exception on error
	 */
	void run(ApplicationArguments args) throws Exception;

}
```

```java
@FunctionalInterface
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
```

## 4.2、Application Events and Listeners

[https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners)

下面是springboot留给开发者的扩展接口，可以基于这些接口对程序进行扩展。

- **ApplicationContextInitializer**

```java
//自定义初始化组件
public class MyApplicationContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("MyApplicationContextInitializer ....initialize.... ");
    }
}
```

- **ApplicationListener**

```java
public class MyApplicationListener implements ApplicationListener {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("MyApplicationListener.....onApplicationEvent...");
    }
}
```

- **SpringApplicationRunListener**

```java
public class MySpringApplicationRunListener implements SpringApplicationRunListener {

    private SpringApplication application;
    public MySpringApplicationRunListener(SpringApplication application, String[] args){
        this.application = application;
    }

    @Override
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        System.out.println("MySpringApplicationRunListener....starting....");

    }

    @Override
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        System.out.println("MySpringApplicationRunListener....environmentPrepared....");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....contextPrepared....");

    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....contextLoaded....");
    }

    @Override
    public void started(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....started....");
    }

    @Override
    public void running(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....running....");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("MySpringApplicationRunListener....failed....");
    }
}
```

## 4.3、ApplicationRunner 与 CommandLineRunner

- **ApplicationRunner**

```java
@Order(1)
@Component
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("MyApplicationRunner...run...");
    }
}
```

- **CommandLineRunner**

```java
/**
 * 应用启动做一个一次性事情
 */
@Order(2)
@Component
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("MyCommandLineRunner....run....");
    }
}
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%205.png)

spring.factories配置：

```java
org.springframework.context.ApplicationContextInitializer=\
  com.atguigu.boot.listener.MyApplicationContextInitializer

org.springframework.context.ApplicationListener=\
  com.atguigu.boot.listener.MyApplicationListener

org.springframework.boot.SpringApplicationRunListener=\
  com.atguigu.boot.listener.MySpringApplicationRunListener
```

调用结果：

```java
MyApplicationListener.....onApplicationEvent...
MySpringApplicationRunListener....starting....
MyApplicationListener.....onApplicationEvent...
MySpringApplicationRunListener....environmentPrepared....

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.4.0)

MyApplicationContextInitializer ....initialize.... 
MyApplicationListener.....onApplicationEvent...
MySpringApplicationRunListener....contextPrepared....
2023-09-03 07:51:51.604  INFO 26220 --- [           main] c.a.boot.Boot09HelloTestApplication      : Starting Boot09HelloTestApplication using Java 1.8.0_202 on rzf with PID 26220 (D:\soft\IDEA\work\springboot2-master\boot-09-hello-test\target\classes started by MrR in D:\soft\IDEA\work\springboot2-master)
2023-09-03 07:51:51.609  INFO 26220 --- [           main] c.a.boot.Boot09HelloTestApplication      : No active profile set, falling back to default profiles: default
MyApplicationListener.....onApplicationEvent...
MySpringApplicationRunListener....contextLoaded....
2023-09-03 07:51:52.484  INFO 26220 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-09-03 07:51:52.485  INFO 26220 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-09-03 07:51:52.485  INFO 26220 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.39]
2023-09-03 07:51:52.494  INFO 26220 --- [           main] o.a.catalina.core.AprLifecycleListener   : Loaded Apache Tomcat Native library [1.2.37] using APR version [1.7.4].
2023-09-03 07:51:52.494  INFO 26220 --- [           main] o.a.catalina.core.AprLifecycleListener   : APR capabilities: IPv6 [true], sendfile [true], accept filters [false], random [true].
2023-09-03 07:51:52.494  INFO 26220 --- [           main] o.a.catalina.core.AprLifecycleListener   : APR/OpenSSL configuration: useAprConnector [false], useOpenSSL [true]
2023-09-03 07:51:52.495  INFO 26220 --- [           main] o.a.catalina.core.AprLifecycleListener   : OpenSSL successfully initialized [OpenSSL 1.1.1u  30 May 2023]
2023-09-03 07:51:52.596  INFO 26220 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-09-03 07:51:52.596  INFO 26220 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 943 ms
2023-09-03 07:51:52.814  INFO 26220 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2023-09-03 07:51:53.058  INFO 26220 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
MyApplicationListener.....onApplicationEvent...
MyApplicationListener.....onApplicationEvent...
2023-09-03 07:51:53.068  INFO 26220 --- [           main] c.a.boot.Boot09HelloTestApplication      : Started Boot09HelloTestApplication in 1.789 seconds (JVM running for 2.53)
MyApplicationListener.....onApplicationEvent...
MyApplicationListener.....onApplicationEvent...
MySpringApplicationRunListener....started....
MyApplicationRunner...run...
MyCommandLineRunner....run....
MyApplicationListener.....onApplicationEvent...
MyApplicationListener.....onApplicationEvent...
MySpringApplicationRunListener....running....
```

# springboot启动流程大图

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%206.png)