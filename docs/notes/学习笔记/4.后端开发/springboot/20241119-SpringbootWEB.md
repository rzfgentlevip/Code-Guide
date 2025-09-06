---
title: SpringBoot WEB 开发
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
permalink: /learning-notes/db0ejjt4/
---

# SpringMVC基础功能与请求处理过程

# 大纲

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644204364576-a4ede74e-e696-4d59-b6fd-b947f08f9fce.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644204364576-a4ede74e-e696-4d59-b6fd-b947f08f9fce.png)

# web部分的说明

springboot为框架的框架，整合了其它框架。拿web开发来说，底层依然是使用了springmvc框架的功能。

springboot对springmvc的使用做了很多自动配置。

传统springmvc的使用需要写很多配置文件，springboot整合后，直接使用即可。

# 官网地址

官网地址：

[https://docs.spring.io/spring-boot/docs/current/reference/html/index.html](https://docs.spring.io/spring-boot/docs/current/reference/html/index.html)

[https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.servlet.spring-mvc](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.servlet.spring-mvc)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644204167620-3f3990c5-6f99-4cd1-806d-3d4be6621845.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644204167620-3f3990c5-6f99-4cd1-806d-3d4be6621845.png)

# 1、Springboot对SpringMVC自动配置概览

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

（Spring Boot为大多数场景提供了Spring MVC的自动配置，即大多场景我们都无需自定义配置。）

The auto-configuration adds the following features on top of Spring’s defaults:

（在spring基础上，自动配置大体上添加了以下默认特性：）

- Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.

（springboot针对springmvc，提供了，内容协商视图解析器，和，BeanName视图解析器）

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#web.servlet.spring-mvc.static-content)).

（针对静态资源（包括webjars）做了自动配置。详细来说，将静态资源放到指定位置，就能默认直接访问。传统springmvc想要访问静态资源，还需配置<mvc:default-servlet-handler />才能访问。）

- Automatic registration of Converter, GenericConverter, and Formatter beans.

（自动注册Converter、GenericConverter，即转换器；和Formatter格式化器。例如对日期数据的自动转换格式化。 ）

- Support for HttpMessageConverters (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#web.servlet.spring-mvc.message-converters)).

（支持HttpMessageConverters HTTP消息转化。后面配合内容协商，进行原理讲解。）

- Automatic registration of MessageCodesResolver (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#web.servlet.spring-mvc.message-codes)).

（自动注册MessageCodesResolver ，用于国际化。 ）

- Static index.html support.

（提供静态index.html欢迎页支持，将index.html放到指定位置，能自动识别和使用。）

- Custom Favicon support (covered later in this document).

（提供自定义Favicon支持，即网站标签页的图标，放到指定位置后，自动识别和使用。）

- Automatic use of a ConfigurableWebBindingInitializer bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#web.servlet.spring-mvc.binding-initializer)).

（针对spirngmvc，自动配置了ConfigurableWebBindingInitializer 数据绑定器。负责将请求数据绑定到对应的JavaBean上。）

如果想要定制化SpringMvc组件，官网提供了如下建议。

If you want to keep those Spring Boot MVC customizations and make more MVC customizations (interceptors, formatters, view controllers, and other features), you can add your own @Configuration class of type WebMvcConfigurer but without @EnableWebMvc.

（如果想在springboot提供的特性基础上，做更多MVC自定义规则（拦截器、格式化器、视图控制器和其他特性），可以通过自定义WebMvcConfigurer类+加上@Configuration注解而不是@EnableWebMvc注解，来实现自定义配置。）

If you want to provide custom instances of RequestMappingHandlerMapping, RequestMappingHandlerAdapter, or ExceptionHandlerExceptionResolver, and still keep the Spring Boot MVC customizations, you can declare a bean of type WebMvcRegistrations and use it to provide custom instances of those components.

（如果想在springboot提供的特性基础上，做自定义RequestMappingHandlerMapping,、RequestMappingHandlerAdapter、或ExceptionHandlerExceptionResolver的配置实例，可通过声明一个WebMvcRegistrations 类型的bean来实现自定义组件，来改变默认的底层组件。）

If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc, or alternatively add your own @Configuration-annotated DelegatingWebMvcConfiguration as described in the Javadoc of @EnableWebMvc.

（如果想全面接管SpringMVC，声明一个DelegatingWebMvcConfiguration类+@EnableWebMvc+@Configuration，来实现。例如@EnableWebMvc的Javadoc中所述。）

官网原文：

[https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.servlet.spring-mvc](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.servlet.spring-mvc)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644204537208-ce9d04b5-217b-4ad3-898a-65a6daec9972.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644204537208-ce9d04b5-217b-4ad3-898a-65a6daec9972.png)

# 2、Springboot对SpringMVC自动配置的部分功能讲解

# 2.1、静态资源访问

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644215491086-3b7e422c-0272-4653-a7d4-1fd248eb6b99.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644215491086-3b7e422c-0272-4653-a7d4-1fd248eb6b99.png)

- 官网原文

By default, Spring Boot serves static content from a directory called /static (or /public or /resources or /META-INF/resources) in the classpath or from the root of the ServletContext. It uses the ResourceHttpRequestHandler from Spring MVC so that you can modify that behavior by adding your own WebMvcConfigurer and overriding the addResourceHandlers method.

默认情况下，springboot的静态内容来自类路径下的/static、/public、/resources 、/META-INF/resources 目录。或来自ServletContext的根目录。实际是使用了Springmvc中的ResourceHttpRequestHandler来处理。可通过自定义WebMvcConfigurer重写addResourceHandlers方法来修改该行为。

In a stand-alone web application, the default servlet from the container is also enabled and acts as a fallback, serving content from the root of the ServletContext if Spring decides not to handle it. Most of the time, this does not happen (unless you modify the default MVC configuration), because Spring can always handle requests through the DispatcherServlet.

By default, resources are mapped on /**, but you can tune that with the spring.mvc.static-path-pattern property. For instance, relocating all resources to /resources/** can be achieved as follows:

默认情况下，springboot将资源映射请求路径为/**上。即访问类路径下/static、/public、/resources 、/META-INF/resources（不包含子文件夹）中的所有文件。我们可使用spring.mvc对其进行调优。通过其static-path-pattern属性，重新定位请求资源的映射路径。如下，只有localhost:8080/resource/xx.jpg才能被访问。：

```yaml
# properties
spring.mvc.static-path-pattern=/resources/**

# yaml
spring:
  mvc:
    static-path-pattern: "/resources/**"
```

### 1、静态资源目录

只要静态资源放在类路径下： /static、/public、/resources 、/META-INF/resources 下，就能通过当前项目根路径/ + 静态资源名进行访问。

实际是使用了Springmvc中的ResourceHttpRequestHandler来处理。可通过自定义WebMvcConfigurer重写addResourceHandlers方法来修改该行为。

静态资源的默认请求映射路径是/**，即只要输入资源名，就能访问类路径下/static、/public、/resources 、/META-INF/resources（不包含子文件夹）的所有文件。

请求进来，先去找Controller看能不能处理。

不能处理的请求再交给静态资源处理器。按以下顺序检索：META-INF/resources->resources->static->public

静态资源也找不到则响应404页面。

- 通过配置改变静态资源的路径

改变静态资源能访问的文件夹

指定只能访问的静态资源路径。META-INF.rersouces下面的图片能访问，public，resources，static下面的不能访问

```yaml
spring:
	# 规定只有请求链接带上jpg前缀才能访问静态资源
  mvc:
    static-path-pattern: "/jpg/**"

  # 指定只能访问的静态资源路径。META-INF.rersouces下面的图片能访问，public，resources，static下面的不能访问
  # 已过时
#  resources:
#    static-locations: classpath:/public/**
  # 高版本用法如下
  web:
    resources:
#      static-locations: classpath:/public/
      static-locations: [classpath:/public/,classpath:/static/]

```

localhost:8080/xx.jpg

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644219177903-dabb83d1-5c1e-44d3-a09c-a75c802176b1.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644219177903-dabb83d1-5c1e-44d3-a09c-a75c802176b1.png)

### 2、设置静态资源请求访问前缀

默认无前缀，即/**

实际HTTP请求路径：当前项目 + static-path-pattern + 静态资源名 = 静态资源文件夹下找

locahost:8080/jpg/11.jpg

```yaml
spring:
  # 规定只有请求链接带上jpg前缀才能访问静态资源
  mvc:
    static-path-pattern: "/jpg/**"

```

### 3、访问webjar

springboot通过[webjars](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)前缀，自动将 /[webjars](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)/** 映射到对应webjars包里的内容。

此技术使用场景很少。

webjars：将常用的前端组件打包成一个后端可以引用的依赖的技术。

[https://www.webjars.org/](https://www.webjars.org/)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644220676443-6e781693-6d6a-488f-bf44-1e34027767b0.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644220676443-6e781693-6d6a-488f-bf44-1e34027767b0.png)

使用方式：

1、导入指定的前端组件包

```yaml
        <dependency>
            <groupId>org.webjars.bower</groupId>
            <artifactId>juqery</artifactId>
            <version>3.3.1</version>
        </dependency>
```

2、通过webjars前缀即可直接访问包里的内容。

[http://localhost:8080/webjars/jquery/3.5.1/jquery.js](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644221034240-59d7ff76-7f9a-4797-bcf7-b0bfe454f1a5.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644221034240-59d7ff76-7f9a-4797-bcf7-b0bfe454f1a5.png)

# 2.2、欢迎页支持

springboot支持两种方式的欢迎页。静态方式、index模板方式。

- 静态方式：直接将index.html文件放到类根目录下，就会被当做欢迎页。

可以配置静态资源路径

但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page功能失效

  resources:
    static-locations: [classpath:/haha/]
```

- index模板方式：编写一个Controller，处理index请求，返回页面。

# 2.3、自定义 `Favicon`

favicon.ico 放在静态资源目录下即可。

有时加载不出来，尝试rebulid project、浏览器停用缓存。

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效，无法自动识别。
```

# 2.4、静态资源配置原理

SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）

而SpringMVC功能的自动配置类为 WebMvcAutoConfiguration，引入spring-boot-starter-web依赖后，致使该配置类生效。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)  // WebMvcConfigurationSupport 可用来自定义MVC配置
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

WebMvcAutoConfiguration 中通过@Bean给容器增加了一些功能组件。例如：

```java
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled")
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() { // HiddenHttpMethodFilter，springmvc用来兼容Rest风格。支持表单提交PUT、DELETE这种请求。
		return new OrderedHiddenHttpMethodFilter();
	}

	@Bean
	@ConditionalOnMissingBean(FormContentFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.formcontent.filter", name = "enabled", matchIfMissing = true)
	public OrderedFormContentFilter formContentFilter() {  // FormContentFilter 表单内容的过滤器
		return new OrderedFormContentFilter();
	}
```

同时也通过内部类的集成了不同的配置类，以集成了 WebMvcAutoConfigurationAdapter 配置类为例：

WebMvcAutoConfigurationAdapter 的属性：ResourceProperties、WebMvcProperties，分别与配置文件中的spring.mvc和spring.resources参数进行绑定。

```java
	@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {

===========

@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

@ConfigurationProperties(prefix = "spring.mvc")
public class WebMvcProperties {
```

接下来以内部类，WebMvcAutoConfigurationAdapter为例， 进行讲解其生效的过程：

### 1、配置类只有一个有参构造器

每一个配置类（可以是内部类）只存在一个有参构造器，

在配置类中，有参构造器所有参数的值都会从容器中获取。即构造器注入。

```java
	@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {

        // 一个配置类只有一个有参构造器
        // 配置类中，有参构造器所有参数的值都会从容器中获取。即构造器注入。
        // ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
        // WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
        // ListableBeanFactory beanFactory Spring的bean容器工厂
        // ObjectProvider 对象提供器。
        // ObjectProvider<HttpMessageConverters> messageConvertersProvider， 找到系统中所有的HttpMessageConverters
        // ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器
        // DispatcherServletPath
        // ServletRegistrationBean   给应用注册原生的Servlet、Filter....
		public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}

```

### 2、资源处理的默认规则

内部类 WebMvcAutoConfigurationAdapter 中创建了很多组件，我们以资源处理功能作为切入点进行分析：

```java
		// 添加静态资源处理器。处理静态资源映射的默认规则
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) { ResourceHandlerRegistry 绑定了配置文件中spirng.resources
			if (!this.resourceProperties.isAddMappings()) { // 读取spring.resources.add-mappings的配置，如果是false，后面的都不会执行，静态资源的路径映射会全部失效。
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod(); // 读取spring.resources.cache.period的缓存配置，意为静态资源默认可以在浏览器缓存中存活多少秒
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
            // webjars的映射规则
			if (!registry.hasMappingForPattern("/webjars/**")) { // 向静态资源注册类中，注册webjars资源的HTTP请求路径：/webjars/**，将该请求路径与classpath:/META-INF/resources/webjars/下的文件进行映射，并设定缓存策略。
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            // 静态资源路径的配置规则
			String staticPathPattern = this.mvcProperties.getStaticPathPattern(); // 获取 spring.resources.static-path-pattern 静态资源HTTP请求路径的配置。
			if (!registry.hasMappingForPattern(staticPathPattern)) {
                // 与上方webjars的映射规则执行了相同的方法，但是操作的目标是静态资源的HTTP请求路径，将其与我们指定的静态资源存放路径（也有默认值/static、/public、/resources 、/META-INF/resources ）进行映射，并设定缓存策略。
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())) // 获取了spring.resources.static-locations 静态资源存放路径的配置
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}

=========== spring.resources.static-locations 的默认值
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644242458844-4f7759e5-2410-4aa2-ac4d-b4207af07f2a.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644242458844-4f7759e5-2410-4aa2-ac4d-b4207af07f2a.png)

由上面代码可分析出，通过spring.resources.add-mappings可以决定静态资源处理器是否生效。

```java
spring:
  resources:
    add-mappings: false   # 禁用所有静态资源规则
```

### 3、欢迎页的处理规则

WebMvcAutoConfiguration 中还有一个内部配置类 EnableWebMvcConfiguration，也提供了很多组件。

其中包括对欢迎页的处理规则。

HandlerMapping 处理器映射，是Springmvc中的一个核心组件。保存了容器中每一个Handler能处理哪些请求的映射规则。

而WelcomePageHandlerMapping就代表了针对欢迎页资源的请求映射规则。

```java
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations()); // // 获取 spring.resources.static-path-pattern 静态资源HTTP请求路径的配置。
			return welcomePageHandlerMapping;
		}

============ new WelcomePageHandlerMapping 进入后
final class WelcomePageHandlerMapping extends AbstractUrlHandlerMapping {

	private static final Log logger = LogFactory.getLog(WelcomePageHandlerMapping.class);

	private static final List<MediaType> MEDIA_TYPES_ALL = Collections.singletonList(MediaType.ALL);

	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {

		// 这里就是，spring.resources.static-path-pattern 静态资源HTTP请求路径，默认值/**被修改后，欢迎页无法直接通过localhost:8080/index.html加载的原因。
        //要用欢迎页功能，必须是/**
        if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {

			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
        	// 调用请求路径对应的 Controller  例如：/index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}

```

### 4、favicon

标签页缩略图，这个和服务器代码无关，浏览器默认会请求 localhost:8080/favicon.ico来获取缩略图。

所以一旦修改了spring.resources.static-path-pattern默认值不再是/**，必须要带上对应前缀，那么浏览器自然找不到缩略图。

# 3、请求参数处理

# 1、请求映射

什么是请求映射？

编写一个Controller，在方法上用@RequestMapping声明HTTP请求地址，这个声明过程即为请求映射。

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "访问了controller";
    }

```

@RequestMapping 注解源码：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {

    // path 和value都指当前请求能处理的路径
	@AliasFor("path")
	String[] value() default {};

	@AliasFor("value")
	String[] path() default {};

    // method 代表能处理请求的请求方式
	RequestMethod[] method 代表能处理请求的请求方式() default {};

```

### 1、rest使用与原理

Rest风格支持，即使用HTTP请求方式动词来表示对资源的操作。

- 传统方式：使用明细的url路径来区分同个业务的不同操作。

/getUser   获取用户     /deleteUser 删除用户    /editUser  修改用户       /saveUser 保存用户

- Rest方法：使用相同的URL，但不同的请求方式来区分。

/user    GET-获取用户    DELETE-删除用户     PUT-修改用户      POST-保存用户

- 在springmvc框架中，要实现RestFull风格的支持，需要配置一个核心过滤器，HiddenHttpMethodFilter。

### 使用方式

html中的form表单不支持PUT、DELETE。所以才需要配置过滤器。并在html中单独为PUT、DELETE指定特殊隐藏属性进行标识。

application.yaml

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        # 开启对RestFul请求的支持
        enabled: true
```

controller

```java
package com.learn.boot.controller;

import org.springframework.web.bind.annotation.*;

@RestController
public class HelloController {

    @RequestMapping(value = "/user", method = RequestMethod.GET)
    // 简写为 @GetMapping("/user")
    public String getUser() {

        return "GET-张三";
    }

    @RequestMapping(value = "/user", method = RequestMethod.POST)
    // 简写为@PostMapping("/user")
    public String saveUser() {
        return "POST-张三";
    }

    @RequestMapping(value = "/user", method = RequestMethod.PUT)
    // 简写为@PutMapping("/user")
    public String putUser() {
        return "PUT-张三";
    }

    @RequestMapping(value = "/user", method = RequestMethod.DELETE)
    // 简写为@DeleteMapping("/user")
    public String deleteUser() {
        return "DELETE-张三";
    }

}

```

html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>欢迎</h1>

测试REST风格；

<form action="/user" method="get">
    <input value="REST-GET 提交" type="submit"/>
</form>
<form action="/user" method="post">
    <input value="REST-POST 提交" type="submit"/>
</form>
method只能指定get与post，无法支持PUT、DELETE。
如果强制指定为PUT、DELETE，会被默认当做get方式。

<form action="/user" method="post">
    <input name="_method" type="hidden" value="delete"/>
    <input value="REST-DELETE 提交" type="submit"/>
</form>
<form action="/user" method="post">
    <input name="_method" type="hidden" value="PUT"/>
    <input value="REST-PUT 提交" type="submit"/>
</form>

</body>
</html>
```

### 底层原理

springboot中对MVC框架的自动配置类为：WebMvcAutoConfiguration，

其中已经预先配置了OrderedHiddenHttpMethodFilter，支持Rest功能。

Rest原理，表单提交时要使用Rest风格接口时，按如下步骤执行：

- 1、表单提交时带上_method 元素，指定Rest动作，如PUT
- 2、请求会被 HiddenHttpMethodFilter 拦截：
- 如果请求无错误异常、且为POST
- 获取 _method 的值，兼容请求包括：PUT.DELETE.PATCH
- 原生 HttpServletRequest request，被转化为 HttpServletRequest 包装模式后的 HttpServletRequestWrapper ，该包装类重写了getMethod方法，使得 _method 参数传给 method。
- 过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。

OrderedHiddenHttpMethodFilter 起作用的过程：

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644549069734-9a786245-8dfc-4864-84f5-3c27ef649cbc.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644549069734-9a786245-8dfc-4864-84f5-3c27ef649cbc.png)

WebMvcAutoConfiguration

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    // 在springmvc框架中，要实现RestFull风格的支持，需要配置一个核心过滤器，HiddenHttpMethodFilter。
	// springboot通过 OrderedHiddenHttpMethodFilter 来实现该功能。
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class) // 用户未配置 HiddenHttpMethodFilter 时
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false) // 配置文件中 spring.mvc.hiddenmethod.filter 指定为true时开启
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}

```

OrderedHiddenHttpMethodFilter

```java
public class OrderedHiddenHttpMethodFilter extends HiddenHttpMethodFilter implements OrderedFilter {
```

HiddenHttpMethodFilter

```java
public class HiddenHttpMethodFilter extends OncePerRequestFilter {

    // 声明了除GET、POST外，还支持REST的PUT、DELETE、PATCH
	private static final List<String> ALLOWED_METHODS =
			Collections.unmodifiableList(Arrays.asList(HttpMethod.PUT.name(),
					HttpMethod.DELETE.name(), HttpMethod.PATCH.name()));

	// 规定HTML中只需要带上一个隐藏参数 _method 即可。
	public static final String DEFAULT_METHOD_PARAM = "_method";

	private String methodParam = DEFAULT_METHOD_PARAM;

	public void setMethodParam(String methodParam) {
		Assert.hasText(methodParam, "'methodParam' must not be empty");
		this.methodParam = methodParam;
	}

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		HttpServletRequest requestToUse = request;

		// 规定了只有POST方式才能继续识别 _method 参数。同时请求是否无错误异常
		if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {  //
			String paramValue = request.getParameter(this.methodParam);  // 获取html中form的 _method 对应的值
			if (StringUtils.hasLength(paramValue)) {
				String method = paramValue.toUpperCase(Locale.ENGLISH);  // 将 _method 的值转为英文大写
				if (ALLOWED_METHODS.contains(method)) { // 判断是否为 PUT、DELETE、PATCH

                    // 将原生 HttpServletRequest 转为 HttpMethodRequestWrapper
					requestToUse = new HttpMethodRequestWrapper(request, method);
				}
			}
		}
		// 过滤器链放行时，实际目标为 HttpServletRequest 的包装类 HttpMethodRequestWrapper
		filterChain.doFilter(requestToUse, response);
	}

	// HttpMethodRequestWrapper 实现了原生的 HttpServletRequest 接口
    // 但构造器中额外支持 _method 传参，并重写了 getMethod 方法，覆盖了原 method 的值。
    // 这种方式为包装模式 Wrapper
	private static class HttpMethodRequestWrapper extends HttpServletRequestWrapper {

		private final String method;

		public HttpMethodRequestWrapper(HttpServletRequest request, String method) {
			super(request);
			this.method = method;
		}

		@Override
		public String getMethod() {
			return this.method;
		}
	}

}

```

HttpServletRequestWrapper   实现了原生的 HttpServletRequest 接口

```java
public class HttpServletRequestWrapper extends ServletRequestWrapper implements
        HttpServletRequest {

```

### 特殊情况

html中的form表单不支持PUT、DELETE。所以才需要过滤器。

但如果只用客户端工具、或微服务开发时，不做页面开发，可直接发送Put、delete等方式请求，无需Filter。

所以 spring.mvc.hiddenmethod.filter 是要根据情况选择性开启。

### 修改默认的method参数名称

1、新增一个配置类，自定义 HiddenHttpMethodFilter 组件

```java
@Configuration(proxyBeanMethods = false)  // 配置类中添加组件的方法之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
public class WebConfig {

    // 自定义REST的method声明参数
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter() {
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }

}

```

2、html中使用自定义的method参数名

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>欢迎</h1>

测试REST风格；

<form action="/user" method="get">
    <input value="REST-GET 提交" type="submit"/>
</form>
<form action="/user" method="post">
    <input value="REST-POST 提交" type="submit"/>
</form>
method只能指定get与post，无法支持PUT、DELETE。
如果强制指定为PUT、DELETE，会被默认当做get方式。

<form action="/user" method="post">
    <input name="_method" type="hidden" value="delete"/>
    <input name="_m" type="hidden" value="delete"/>
    <input value="REST-DELETE 提交" type="submit"/>
</form>
<form action="/user" method="post">
    <input name="_method" type="hidden" value="PUT"/>
    <input name="_m" type="hidden" value="PUT"/>
    <input value="REST-PUT 提交" type="submit"/>
</form>

</body>
</html>
```

### 2、请求路径映射原理

### 2.1、讲解

HandlerMapping，即HTTP请求与具体Controller处理方法的映射关系。

映射原理简单来说：

服务器启动后，所有的请求映射都放在 HandlerMapping 集合中。

请求进来，遍历每一个HandlerMapping  ，逐一匹配看是否请求路径与其匹配，

匹配上了就是用该 HandlerMapping  对应的 Handler 对应的控制器 Controller 方法。。

### 2.2、DispatcherServlet 请求映射执行分析

Springboot集成了Springmvc，在springmvc中 DispatcherServlet 是处理所有请求的开始。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644549624683-6f559c1f-6d51-44f3-b8f8-8ad9c1a2a8ed.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644549624683-6f559c1f-6d51-44f3-b8f8-8ad9c1a2a8ed.png)

HttpServlet 为原生API，其子类需重写doGet、doPost方法

HttpServletBean 没有重写doGet、doPost。

FrameworkServlet 重写了doGet、doPost，实际执行的是 processRequest 方法，后调用了抽象方法 doServlce

DispatcherServlet 实现了抽象方法 doServlce，后调用了 doDispatch 方法，实现请求映射。doDispatch 方法每个请求都会调用。

对Springmvc 请求处理功能的分析，从 DispatcherServlet - doDispatch 方法开始。

### 整体流程图

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644658363228-25b5da69-d325-427d-90df-f0b0d19d014a.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644658363228-25b5da69-d325-427d-90df-f0b0d19d014a.png)

### HttpServletBean

```java
public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
```

### FrameworkServlet

```java
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {

    @Override
	protected final void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}

    @Override
	protected final void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}

	protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		// 各种初始化过程 --start
		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;

		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

		initContextHolders(request, localeContext, requestAttributes);
		// 各种初始化过程 --end
		try {
            // 核心方法
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}

	protected abstract void doService(HttpServletRequest request, HttpServletResponse response)
			throws Exception;

```

### DispatcherServlet

间接继承了 HttpServlet，所以必然会有重写 doGet、doPost 的方法。

同时直接继承了FrameworkServlet，重写了 doService 方法。

```java
public class DispatcherServlet extends FrameworkServlet {

	@Nullable
	private List<HandlerMapping> handlerMappings; // 保存了处理器映射规则。 例如 GET /user ，对应 UserController--getUser

    // 重写的是 FrameworkServlet 中的抽象方法
    @Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			attributesSnapshot = new HashMap<>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		if (this.flashMapManager != null) {
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}

		try {
            // 执行请求转发，这里才是要重点研究的地方。每次请求都会来调用
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}

	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null; // 执行链
		boolean multipartRequestParsed = false; // 表示是否为文件上传请求

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request); // 如果是异步请求，使用异步管理器

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request); // 检查文件上传
				multipartRequestParsed = (processedRequest != request); // 如果是文件上传请求，需要进行转化。

				// 识别找到是哪个 handler 来处理当前请求。也就是指定哪个 Controller 的方法来处理。
				mappedHandler = getHandler(processedRequest);
                // 找到方法后的具体执行逻辑
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}

    // List<HandlerMapping> handlerMappings 保存了处理器映射规则。 例如 GET /user ，对应 UserController--getUser，如下图所示。
    // 其中
	@Nullable
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
                // 从所有映射规则集合中，获取对应的路径的handler 处理器。
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}

```

### 2.3、HandlerMapping 种类

DispatcherServlet -- getHandler 中的参数 this.handlerMappings，即  `List<HandlerMapping> handlerMappings`  保存了处理器映射规则：

默认由五种HandlerMapping。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644562317289-e3063364-d1d9-40ac-a706-9887eefba4aa.png)

每种处理器映射规则类，都继承了 AbstractHandlerMapping。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644650271836-7cb96c2a-53c8-4414-afcc-f31e0902c7fb.png)

### 代码分析

DispatcherServlet -- getHandler中，遍历了所有HandlerMapping 的派生类型中存放的映射关系，找到匹配结果后，执行其 getHandler 方法。

HandlerMapping 的不同功能的派生类，重写了 getHandler 方法。

AbstractHandlerMapping

```java
public abstract class AbstractHandlerMapping extends WebApplicationObjectSupport
		implements HandlerMapping, Ordered, BeanNameAware {

	@Override
	@Nullable
	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		// getHandlerInternal 进入抽象派生类中。
        Object handler = getHandlerInternal(request);
		if (handler == null) {
        	// 断点跳入
			handler = getDefaultHandler();
		}
		if (handler == null) {
			return null;
		}
		// Bean name or resolved handler?
		if (handler instanceof String) {
			String handlerName = (String) handler;
			handler = obtainApplicationContext().getBean(handlerName);
		}

		HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);

		if (logger.isTraceEnabled()) {
			logger.trace("Mapped to " + handler);
		}
		else if (logger.isDebugEnabled() && !request.getDispatcherType().equals(DispatcherType.ASYNC)) {
			logger.debug("Mapped to " + executionChain.getHandler());
		}

		if (hasCorsConfigurationSource(handler) || CorsUtils.isPreFlightRequest(request)) {
			CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
			CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
			config = (config != null ? config.combine(handlerConfig) : handlerConfig);
			executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
		}

		return executionChain;
	}

	@Nullable
	protected abstract Object getHandlerInternal(HttpServletRequest request) throws Exception;

```

以 RequestMappingHandlerMapping为例：

AbstractHandlerMapping

- - AbstractHandlerMethodMapping
- - RequestMappingHandlerMapping

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644563922469-1958b524-2a06-4ef7-a262-901a42a0d4fa.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644563922469-1958b524-2a06-4ef7-a262-901a42a0d4fa.png)

```java
public abstract class RequestMappingInfoHandlerMapping extends AbstractHandlerMethodMapping<RequestMappingInfo> {

	@Override
	protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
		request.removeAttribute(PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE);
		try {
            // 执行父类 AbstractHandlerMethodMapping 的 getHandlerInternal 方法
			return super.getHandlerInternal(request);
		}
		finally {
			ProducesRequestCondition.clearMediaTypesAttribute(request);
		}
	}

```

AbstractHandlerMethodMapping，继承了 AbstractHandlerMapping。

```java
public abstract class AbstractHandlerMethodMapping<T> extends AbstractHandlerMapping implements InitializingBean {

    private final MappingRegistry mappingRegistry = new MappingRegistry();

	@Override
	protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
        // 拿到当前访问的路径 /user
		String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
		request.setAttribute(LOOKUP_PATH, lookupPath);
        // 并发查询下，获取并发锁
		this.mappingRegistry.acquireReadLock();
		try {
            // 获取实际对应的 Controller 方法
			HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
			return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
		}
		finally {
			this.mappingRegistry.releaseReadLock();
		}
	}

    // 获取实际对应的 Controller 方法
    // lookupPath http请求路径 /user
    // request 原生请求
	@Nullable
	protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
		List<Match> matches = new ArrayList<>();
        // 找请求路径对应方法的集合，例如 /user 对应可能有多个方法，每个方法的请求方式不同，get/post/put...
		List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);
		if (directPathMatches != null) {
            // 所有找到的方法，经过 addMatchingMappings 分析后，获取最佳匹配。可能有多个最佳匹配方法，都能处理该请求。匹配结果放入 matches 中
			addMatchingMappings(directPathMatches, matches, request);
		}
		if (matches.isEmpty()) {
            // 找不到就填写一些空的东西
			addMatchingMappings(this.mappingRegistry.getMappings().keySet(), matches, request);
		}

		if (!matches.isEmpty()) {
            // 如果同时找到很多个，默认匹配第一个
			Match bestMatch = matches.get(0);
			if (matches.size() > 1) {、
                // 如果经过最佳匹配处理后，结果还为多个，再进行一系列排序处理
				Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
				matches.sort(comparator);
				bestMatch = matches.get(0);
				if (logger.isTraceEnabled()) {
					logger.trace(matches.size() + " matching mappings: " + matches);
				}
				if (CorsUtils.isPreFlightRequest(request)) {
					return PREFLIGHT_AMBIGUOUS_MATCH;
				}
				Match secondBestMatch = matches.get(1);
                // 经过筛选分析后，还有多个匹配结果，接着进行对比
				if (comparator.compare(bestMatch, secondBestMatch) == 0) {
					Method m1 = bestMatch.handlerMethod.getMethod();
					Method m2 = secondBestMatch.handlerMethod.getMethod();
					String uri = request.getRequestURI();
                    // 同一个请求，两个方法都能处理时，抛出异常
                    // 这也就要求我们，同样的请求路径、请求方式，只能有一个方法与之对应。
					throw new IllegalStateException(
							"Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");
				}
			}
			request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.handlerMethod);
			handleMatch(bestMatch.mapping, lookupPath, request);
			return bestMatch.handlerMethod;
		}
		else {
			return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);
		}
	}

	private void addMatchingMappings(Collection<T> mappings, List<Match> matches, HttpServletRequest request) {
		for (T mapping : mappings) {
			T match = getMatchingMapping(mapping, request);
			if (match != null) {
				matches.add(new Match(match, this.mappingRegistry.getMappings().get(mapping)));
			}
		}
	}

	@Nullable
	protected abstract T getMatchingMapping(T mapping, HttpServletRequest request);

```

### 整理流程图

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644763033175-fb0261ae-2194-43e8-ba9a-4875cc1b3a25.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644763033175-fb0261ae-2194-43e8-ba9a-4875cc1b3a25.png)

### 实际运行时赋值举例

### WelcomePageHandlerMapping

欢迎页的处理请求映射规则。用于自动向容器中存放欢迎页。其中保存了请求路径与实际资源的映射规则。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644562974275-82423f4a-ac25-4dc1-a76c-ecb9bdb2846d.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644562974275-82423f4a-ac25-4dc1-a76c-ecb9bdb2846d.png)

### RequestMappingHandlerMapping

保存了所有用了 @RequestMapping 注解的handler处理器类的映射规则。

项目一启动，就会将 Controller 下所有 @RequestMapping 注解的方法及请求路径的映射规则，保存至 RequestMappingHandlerMapping 中。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644563623768-c8212f27-e53c-4cee-95e9-e0ed22b68133.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644563623768-c8212f27-e53c-4cee-95e9-e0ed22b68133.png)

### 2.4、Springboot默认配置的HandlerMapping组件

SpringBoot默认自动配置了一下HandlerMapping 组件，存放各种映射规则：

RequestMappingHandlerMapping，用于解析使用了 @RequestMapping 注解的Controller方法。

WelcomePageHandlerMapping ，用于自动向容器中存放欢迎页的映射规则。 即访问 /能访问到index.html；

BeanNameUrlHandlerMapping、RouteFunctionMapping、SimpleUrlHandlerMapping 后续再讲。

以Springboot默认使用 RequestMappingHandlerMapping 为例，通过 WebMvcAutoConfiguration 配置类进行注册：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {

        // springboot 默认第一个使用的handlerMapping，用于解析使用了 @RequestMapping 注解的Controller方法
		@Bean
		@Primary
		@Override
		public RequestMappingHandlerMapping requestMappingHandlerMapping(
				@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,
				@Qualifier("mvcConversionService") FormattingConversionService conversionService,
				@Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider) {
			// Must be @Primary for MvcUriComponentsBuilder to work
			return super.requestMappingHandlerMapping(contentNegotiationManager, conversionService,
					resourceUrlProvider);
		}

        // 配置欢迎页的请求映射规则 WelcomePageHandlerMapping
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}
```

### 2.5、自定义 HandlerMapping

如果我们需要一些自定义的映射处理，我们也可以自己给容器中放HandlerMapping。自定义 HandlerMapping。

场景举例：

localhost:8080/api/v1/user  GET

localhost:8080/api/v2/user  GET

存在两个版本的相同功能接口，我们需要自定义其匹配规则，去不同的包里找接口，这时就需要自定义映射处理。

# 2、请求参数支持传入的类型

Springmvc处理WEB请求，Controller类的方法可以接受的传参类型：

请求参数注解，

ServletApi，

复杂参数类型对象，

自定义对象参数。

### 2.1、使用注解

通过参数注解，springmvc会在controller方法被调用时，自动匹配请求URL中对应的值，赋予变量。

```
@PathVariable、
@RequestHeader、
@ModelAttribute、
@RequestParam、
@MatrixVariable、
@CookieValue、
@RequestBody
```

### @PathVariable

### @RequestHeader

### @RequestHeader

### @CookieValue

### @RequestBody

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644814692936-3ddceebe-5f2b-4446-8ec0-2f931ab70173.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1644814692936-3ddceebe-5f2b-4446-8ec0-2f931ab70173.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1644833375491-bc339b1a-8b3e-4e11-bf7b-273a99c72082.png](https://cdn.nlark.com/yuque/0/2022/png/623390/1644833375491-bc339b1a-8b3e-4e11-bf7b-273a99c72082.png)

```java
测试基本注解：
<ul>
    <li>@PathVariable（路径变量）</li>
    <li>@RequestHeader（获取请求头）</li>
    <li>@RequestParam（获取请求参数）</li>
    <li>@CookieValue（获取cookie值）</li>
    <a href="car/3/owner/lisi?age=18&inters=basketball&inters=game">car/{id}/owner/{username}</a>

    <li>@RequestBody（获取请求体[POST]）</li>
    <form action="/save" method="post">
        测试@RequestBody获取数据 <br/>
        用户名：<input name="userName"/> <br>
        邮箱：<input name="email"/>
        <input type="submit" value="提交"/>
    </form>

</ul>

===============

@RestController
public class ParameterTestController {

    /**

     *
     * @PathVariable 获取路径变量
     * 传统GET传递参数：/car?id=1
     * rest风格则支持使用路径变量方式 /car/1
     * 1、给请求参数赋予路径变量中指定的单个值
     * 2、如果参数类型为map，且类型为<String,String>，那么 @PathVariable 可将所有的路径变量封装为map传入。
     *
     * @RequestHeader 获取HTTP请求头信息
     * 1、将请求头信息中指定的KEY对应的值，赋给方法参数
     * 2、将整个请求头信息赋给方法参数。方法参数的类型必须为 Map<String,String>、MultiValueMap<String,String>、HttpHeaders
     *
     * @RequestHeader 获取请求中参数的值
     * 1、将请求参数中指定的KEY对应的值，赋给请求参数
     * 2、将整个请求参数信息赋给方法参数。方法参数的类型必须为 Map<String,String>、MultiValueMap<String,String>
     *
     * @CookieValue 获取cookie的值
     * 1、将Cookie中指定的KEY对应的值，赋给请求参数
     * 2、将整个Cookie信息赋给请求参数，方法参数的类型必须为 javax.servlet.http.Cookie
     * 不同的浏览器有可能不会由IDEA产生Cookie。
     *
     */
    @GetMapping("/car/{id}/owner/{userName}")
    public Map<String, Object> getCar(@PathVariable("id") Integer id,
                                      @PathVariable("userName") String userName,
                                      @PathVariable Map<String, String> map,
                                      @RequestHeader("User-Agent") String userAgent,
                                      @RequestHeader Map<String, String> header,
                                      @RequestParam("age") Integer age,
                                      @RequestParam("inters") List<String> inters,
                                      @RequestParam Map<String, String> requestParam,
                                      @CookieValue("_ga") String _ga,
                                      @CookieValue("_ga") Cookie cookie) {
        Map<String, Object> map2 = new HashMap<>();
        map2.put("id", id);
        map2.put("userName", userName);
        System.out.println(map);
        System.out.println(map2);

        System.out.println("请求头中 User-Agent 的值为" + userAgent);
        System.out.println("整个请求头的值：" + header);

        System.out.println("获取请求参数");
        System.out.println("age:" + age);
        System.out.println("inters:" + inters);
        System.out.println(requestParam);

        System.out.println("获取cookie");
        System.out.println(_ga);
        System.out.println(cookie);

        return map2;
    }

    /**
     * @RequestBody 获取请求体
     * 只有POST请求才有请求体。例如表单提交，表单中的数据以KV存储。
     * 将请求体数据赋予方法参数
     *
     * @param content
     * @return
     */
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        System.out.println(content);
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }

}
```

### @RequestAttribute

```java
    <li>@RequestAttribute（获取request域属性）</li>
    <a href="toSuccessPage">toSuccessPage</a>

============

@Controller
public class RequestController {

    @GetMapping("/toSuccessPage")
    public String goToPage(HttpServletRequest request){
        request.setAttribute("msg","成功了...");
        request.setAttribute("code",200);
        return "forward:/success";  //转发到  /success请求
    }

    /**
     * @RequestAttribute 获取请求域中的值
     * 也可以直接用 HttpServletRequest request 作为参数，从 request 直接 getAttribute 想要的属性。
     * 常用场景，在请求域中存放数据，再跳转至页面，页面上再将数据渲染出来。
     */
    @ResponseBody
    @GetMapping("/success")
    public Map success(@RequestAttribute(value = "msg",required = false) String msg,
                       @RequestAttribute(value = "code",required = false)Integer code,
                       HttpServletRequest request){
        // 转发后的 request 与转发前的 request 是同一个对象
        Object msg1 = request.getAttribute("msg");

        Map<String,Object> map = new HashMap<>();

        map.put("从request对象中获取的",msg1);
        map.put("从请求参数中获取",msg);

        System.out.println(map);
        System.out.println(msg1);

        return map;

    }
}
```

### @MatrixVariable

在进行请求传参时，可存在路径变量，路径变量中可以传递矩阵变量。

```java
===传统路径变量的传参方式：
/cars/{path}?K1=v1&k2=v21&k2=v22&k2=v23
例如：
     <a href="/cars/sell?low=34&brand=byd&brand=audi&brand=yd">传统路径变量传参</a>
 用路径变量时，可用上述方式进行传参，该方式称为 queryString，查询字符串。请求参数可用 @RequestParam 获取。
 也可使用矩阵变量。

===路径变量下，使用矩阵变量的传参方式：
 /cars/{path;k1=v1;k2=v21,v22,v23}

矩阵变量需要在SpringBoot中手动开启
根据RFC3986的规范，矩阵变量应当绑定在路径变量中！
若一个路径变量中带有有多个矩阵变量，应当使用英文符号;进行分隔。
分号前面为访问路径，分号后面为矩阵变量。
若是单个矩阵变量有多个值，值应当使用英文符号,进行分隔，或每个值前命名多个重复的key即可。
如：/cars/{path;k1=v1;k2=v21,v22,v23}

 场景：
     如果是多个值时，值以逗号或分号分割都行：
     <a href="/cars/sell;low=34;brand=byd,audi,yd>@MatrixVariable（矩阵变量）</a>
     <a href="/cars/sell;low=34;brand=byd;brand=audi;brand=yd>@MatrixVariable（矩阵变量）</a>

     如果是多层路径传参：
         例如请求查看ID为1且20岁的老板下，ID为2且30岁的员工。
         /boss/{bossId}/{empId}
         /boss/1;age=20/2;age=10
     传统方式：/bumen/1/2/
     矩阵变量传参方式：
     参数绑定在路径变量中，参数直接以分号分割。

 面试题：页面开发，cookie禁用后，session里面的内容怎么使用？
     紧用cookie前：
         session中可以保存KV，
         每个登录用户都有各自唯一的session id，即 jsessionid。
         jsessionid 保存在浏览器的  cookie 中，
         浏览器每次发送请求都会带上 cookie。
         服务器就能根据cookie--找到 jessionid -- 找到用户的session -- 找到对应的KV数据。
     禁用cookie后，
         可通过 url重写方式解决，即，将cookie中的所有KV，通过矩阵变量的方式传递给服务器。
         /cars/sell;jessionid=xxxx

```

### 原理说明

springboot默认禁用了矩阵变量。需要手动开启。否则报以下错误：

![https://cdn.nlark.com/yuque/0/2022/png/623390/1645755695293-d51a64dc-ad47-4692-85f8-8bbef9dc76c0.png](https://cdn.nlark.com/yuque/0/2022/png/623390/1645755695293-d51a64dc-ad47-4692-85f8-8bbef9dc76c0.png)

开启方法：

定制化springboot底层的sringmvc功能，

在springmvc的自动配置类，WebMvcAutoConfiguration 中，通过实现WenMvcConfigurer 的 configurePathMatch 方法默认定制了路径解析规则。我们可自建配置类定制该规则，覆盖原有默认配置。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1645930843019-9d9aca82-f865-4e77-a45d-dbfc115834c0.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1645930843019-9d9aca82-f865-4e77-a45d-dbfc115834c0.png)

参考第一节自动配置概览中，想要自定义mvc组件，有以下三种方法。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1645756715398-86898ce8-871b-463b-89af-f7c633bf0c03.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1645756715398-86898ce8-871b-463b-89af-f7c633bf0c03.png)

我们可以第一种为例：自定义实现 WebMvcConfiguration 类，加上 @Configuration 注解。具体做法见下方演示中的配置类。

### 演示

html

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>欢迎</h1>

测试基本注解：
<ul>
    <br/>
    <li>@MatrixVariable（矩阵变量）</li>
    <a href="/cars/sell;low=34;brand=byd,audi,yd">@MatrixVariable（矩阵变量）--多值参数传参--方法1</a>
    <a href="/cars/sell;low=34;brand=byd;brand=audi;brand=yd">@MatrixVariable（矩阵变量）--多值参数传参--方法2</a>
    <a href="/boss/1;age=20/2;age=30">@MatrixVariable（矩阵变量）--多级路径传参：/boss/{bossId}/{empId}</a>
</ul>
</body>
</html>
```

controller

```java
package com.learn.boot.controller;

import org.springframework.web.bind.annotation.*;

import javax.servlet.http.Cookie;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
public class ParameterTestController {

    /**
     *
     * ===传统路径变量的传参方式：
     * /cars/{path}?K1=v1&k2=v21&k2=v22&k2=v23
     * 例如：
     *      <a href="/cars/sell?low=34&brand=byd&brand=audi&brand=yd">传统路径变量传参</a>
     *  用路径变量时，可用上述方式进行传参，该方式称为 queryString，查询字符串。请求参数可用 @RequestParam 获取。
     *  也可使用矩阵变量。
     *
     * ===路径变量下，使用矩阵变量的传参方式：
     *  /cars/{path;k1=v1;k2=v21,v22,v23}
     *
     * 矩阵变量需要在SpringBoot中手动开启
     * 根据RFC3986的规范，矩阵变量应当绑定在路径变量中！
     * 若一个路径变量中带有有多个矩阵变量，应当使用英文符号;进行分隔。
     * 分号前面为访问路径，分号后面为矩阵变量。
     * 若是单个矩阵变量有多个值，值应当使用英文符号,进行分隔，或每个值前命名多个重复的key即可。
     * 如：/cars/{path;k1=v1;k2=v21,v22,v23}
     *
     *  场景：
     *      如果是多个值时，值以逗号或分号分割都行：
     *      <a href="/cars/sell;low=34;brand=byd,audi,yd>@MatrixVariable（矩阵变量）</a>
     *      <a href="/cars/sell;low=34;brand=byd;brand=audi;brand=yd>@MatrixVariable（矩阵变量）</a>
     *
     *      如果是多层路径传参：
     *          例如请求查看ID为1且20岁的老板下，ID为2且30岁的员工。
     *          /boss/{bossId}/{empId}
     *          /boss/1;age=20/2;age=10
     *      传统方式：/bumen/1/2/
     *      矩阵变量传参方式：
     *      参数绑定在路径变量中，参数直接以分号分割。
     *
     *
     *  面试题：页面开发，cookie禁用后，session里面的内容怎么使用？
     *      紧用cookie前：
     *          session中可以保存KV，
     *          每个登录用户都有各自唯一的session id，即 jsessionid。
     *          jsessionid 保存在浏览器的  cookie 中，
     *          浏览器每次发送请求都会带上 cookie。
     *          服务器就能根据cookie--找到 jessionid -- 找到用户的session -- 找到对应的KV数据。
     *      禁用cookie后，
     *          可通过 url重写方式解决，即，将cookie中的所有KV，通过矩阵变量的方式传递给服务器。
     *          /cars/sell;jessionid=xxxx
     */

    /**
     * 路径传参--矩阵变量演示--单个参数有多个值
     * 矩阵变量要放在路径变量中，所以mapping不能写成 /cars/path，path要带上大括号{}
     * @param low
     * @param brand
     * @param path
     * @return
     */
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();
        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        System.out.println("路径变量带上了分号后的矩阵变量，但是经处理后，路径变量与对应参数会被拆开。处理后的path参数值为："+ path);
        return map;
    }

    /**
     * 路径传参--矩阵变量演示--多级路径传参
     * 请求查看ID为1且20岁的老板下，ID为2且30岁的员工。
     * /boss/1;age=20/2;age=30
     * @param bossAge
     * @param empAge
     * @return
     */
    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();
        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;

    }

}

```

配置类

```java
package com.learn.boot.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.HiddenHttpMethodFilter;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.util.UrlPathHelper;

@Configuration(proxyBeanMethods = false)  // 配置类中添加组件的方法之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
public class WebConfig implements WebMvcConfigurer {

    // 自定义REST的method声明参数
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter() {
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法一：在此处配置类里，自定义一个 WebMvcConfigurer 实现类，并标注 @Bean 注入即可。
     *
     * @return
     */
    //@Bean
    //public WebMvcConfigurer webMvcCOnfigurer(){
    //    return new WebMvcConfigurer(){
    //        @Override
    //        public void configurePathMatch(PathMatchConfigurer configurer){
    //            UrlPathHelper urlPathHelper = new UrlPathHelper();
    //            // 设置不移除分号后面的内容。保证矩阵变量能正常传递
    //            urlPathHelper.setRemoveSemicolonContent(false);
    //            configurer.setUrlPathHelper(urlPathHelper);
    //        }
    //    };
    //}

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法2：直接让 WebConfig 实现 WebMvcConfigurer 接口，并重写 configurePathMatch 方法。配置路径映射的规则。
     *  注意：WebMvcConfigurer 对方法进行了默认实现，所以没必要将所有接口进行重写。
     */
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        // 设置不移除分号后面的内容。保证矩阵变量能正常传递
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}

```

### 2.2、Servlet Api

请求参数支持使用原生Servlet Api。如下。

```java
@Controller
public class RequestController {

    @GetMapping("/toSuccessPage")
    public String goToPage(HttpServletRequest request){
        request.setAttribute("msg","成功了...");
        request.setAttribute("code",200);
        return "forward:/success";  //转发到  /success请求
    }
```

WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、ZoneId

由请求参数解析器 ServletRequestMethodArgumentResolver  来解析以上的部分参数。

课程对应第33节。具体源码解析码过程，暂且省略。

```java
public class ServletRequestMethodArgumentResolver implements HandlerMethodArgumentResolver {

	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		Class<?> paramType = parameter.getParameterType();
		return (WebRequest.class.isAssignableFrom(paramType) ||
				ServletRequest.class.isAssignableFrom(paramType) ||
				MultipartRequest.class.isAssignableFrom(paramType) ||
				HttpSession.class.isAssignableFrom(paramType) ||
				(pushBuilder != null && pushBuilder.isAssignableFrom(paramType)) ||
				Principal.class.isAssignableFrom(paramType) ||
				InputStream.class.isAssignableFrom(paramType) ||
				Reader.class.isAssignableFrom(paramType) ||
				HttpMethod.class == paramType ||
				Locale.class == paramType ||
				TimeZone.class == paramType ||
				ZoneId.class == paramType);
	}
```

### 2.3、复杂参数

复杂请求参数包含下方几种：

```java
Map、Model
	这两者里面的数据会被默认放在request的请求域中 （request.setAttribute）、
    即，向Map、Model中传递值，相当于调用了request.setAttribute，往请求域中放数据。
RedirectAttributes
	用来做 重定向携带数据，后面页面开发的时候会讲。
ServletResponse
	原生ServletApi中的响应对象。
SessionStatus、
UriComponentsBuilder、
ServletUriComponentsBuilder
Errors/BindingResult、
```

- 测试

下面测试 Map、Model、HttpServletRequest 、Cookie

通过转发后，下个controller方法依然能打印出上个方法在请求域中存放的值。

印证了Map、Model、HttpServletRequest 都是往请求域中存放中数据。

```java
<a href="/params">测试复杂请求参数</a>
===========================

package com.learn.boot.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
public class RequestController {

    // 复杂请求参数测试
    @GetMapping("/params")
    public String testComplexParam(Map<String,Object> map,
                            Model model,
                            HttpServletRequest request,
                            HttpServletResponse response){
        map.put("map_k1","map_v1");
        model.addAttribute("model_k1","model_v1");
        request.setAttribute("request_k1","request_v1");

        Cookie cookie = new Cookie("cookie_k1","cookie_v1");
        cookie.setDomain("localhost"); // 设置cookie 的作用域
        response.addCookie(cookie);
        return "forward:/success";
    }

    /**
     * @RequestAttribute 获取请求域中的值
     * 也可以直接用 HttpServletRequest request 作为参数，从 request 直接 getAttribute 想要的属性。
     * 常用场景，在请求域中存放数据，再跳转至页面，页面上再将数据渲染出来。
     *
     * @RequestAttribute("msg") String msg, 指定URL必须传递KEY为msg的参数，否则无法匹配该方法。
     * @RequestAttribute(value = "msg",required = false) String msg,  指定URL不必须传递KEY为msg的参数，能正常匹配该方法。
     *
     */
    @ResponseBody
    @GetMapping("/success")
    public Map success(@RequestAttribute(value = "msg",required = false) String msg,
                       @RequestAttribute(value = "code",required = false)Integer code,
                       HttpServletRequest request){
        // 转发后的 request 与转发前的 request 是同一个对象
        Object msg1 = request.getAttribute("msg");

        Map<String,Object> map = new HashMap<>();

        map.put("从request对象中获取的",msg1);
        map.put("从请求参数中获取",msg);

        System.out.println(map);
        System.out.println(msg1);

        // 复杂请求参数测试
        Object map_v1 = request.getAttribute("map_k1");
        Object model_v1 = request.getAttribute("model_k1");
        Object request_v1 = request.getAttribute("request_k1");
        map.put("map_k1",map_v1);
        map.put("model_k1",model_v1);
        map.put("request_k1",request_v1);
        return map;

    }

}

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646311709780-b3e79285-0b07-4f69-b838-1864c28f652b.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646311709780-b3e79285-0b07-4f69-b838-1864c28f652b.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646311838619-2fe9dd70-1043-48ad-9913-eec64b5577b3.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646311838619-2fe9dd70-1043-48ad-9913-eec64b5577b3.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646311688165-3d946792-adf7-4622-b714-1fef17e42dcf.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646311688165-3d946792-adf7-4622-b714-1fef17e42dcf.png)

- 源码讲解

课程对应第34节。具体源码解析码过程，暂且省略。

MapMethodProcessor  处理 Map类型

ModelMethodProcessor  处理 Model类型

```java
----------------因为这两个解析器在处理时都调用了 return mavContainer.getModel(); 来获取值。

---------------- mavContainer.getModel() 实际对应
public class ModelAndViewContainer {
    private final ModelMap defaultModel = new BindingAwareModelMap();
	public ModelMap getModel() {
		if (useDefaultModel()) {
			return this.defaultModel;
		}
		else {
			if (this.redirectModel == null) {
				this.redirectModel = new ModelMap();
			}
			return this.redirectModel;
		}
	}
---------------- 获取的就是 BindingAwareModelMap 对象。
public class BindingAwareModelMap extends ExtendedModelMap {

public class ExtendedModelMap extends ModelMap implements Model {

---------------- 实际获取的都是Model类型数据

```

### 2.4、自定义对象参数

springmvc支持直接将自定义的类作为请求参数，自动将URL中的数据映射到自定义类中。

且支持级联属性。

ServletModelAttributeMethodProcessor 用于解析自定义对象参数。

课程对应第35、36节。具体源码解析码过程，暂且省略。

- 案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>欢迎</h1>

    <br/>
    测试自定义封装POJO作为请求参数:
    <form action="/savePerson" method="post">
        姓名： <input name="userName" value="zhangsan"/> <br/>
        年龄： <input name="age" value="18"/> <br/>
        生日： <input name="birth" value="2019/12/10"/> <br/>
        宠物姓名：<input name="pet.name" value="阿猫"/><br/>
        宠物年龄：<input name="pet.age" value="5"/>
        <input type="submit" value="保存"/>
    </form>
</body>
</html>
```

```java
package com.learn.boot.controller;

import com.learn.boot.bean.Person;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.Cookie;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
public class ParameterTestController {

    /**
     * 数据绑定：页面提交的请求数据（GET、POST）都可以和对象属性进行绑定
     * 支持级联属性：pet.name
     * @param person
     * @return
     */
    @PostMapping("/savePerson")
    public Person savePerson(Person person){
        return person;
    }

}

==========
package com.learn.boot.bean;

import lombok.Data;

import java.util.Date;

/**
 *     姓名： <input name="userName"/> <br/>
 *     年龄： <input name="age"/> <br/>
 *     生日： <input name="birth"/> <br/>
 *     宠物姓名：<input name="pet.name"/><br/>
 *     宠物年龄：<input name="pet.age"/>
 */
@Data
public class Person {

    private String userName;
    private Integer age;
    private Date birth;
    private Pet pet;

}

============
package com.learn.boot.bean;

import lombok.Data;

@Data
public class Pet {

    private String name;
    private Integer age;

}

```

### 自定义参数转换器

此处演示请求方法的自定义类型参数，添加一个转换器，使得spirngmvc支持对象的多个属性值，能以逗号分隔合并为一个字符串，进行传递。

宠物： <input name="pet" value="阿猫,3"/>

```java
package com.learn.boot.config;

import com.learn.boot.bean.Pet;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.format.FormatterRegistry;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.HiddenHttpMethodFilter;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.util.UrlPathHelper;

@Configuration(proxyBeanMethods = false)  // 配置类中添加组件的方法之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
public class WebConfig /*implements WebMvcConfigurer*/ {

    // 自定义REST的method声明参数
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter() {
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法一：在此处配置类里，自定义一个 WebMvcConfigurer 实现类，并标注 @Bean 注入即可。
     *
     * @return
     */
    @Bean
    public WebMvcConfigurer webMvcCOnfigurer(){
        return new WebMvcConfigurer(){
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer){
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                // 设置不移除分号后面的内容。保证矩阵变量能正常传递
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }

            @Override
            public void addFormatters(FormatterRegistry registry) {
                // 添加自定义参数转换器 ，String -> Pet
                // 此处演示请求方法的自定义类型参数，支持多个属性以逗号分隔，进行传递。宠物： <input name="pet" value="阿猫,3"/>
                registry.addConverter(new Converter<String, Pet>() {
                    // S 原类型   T 目标类型
                    @Override
                    public Pet convert(String source) {
                        // 啊猫,3
                        if(!StringUtils.isEmpty(source)){
                            Pet pet = new Pet();
                            String[] split = source.split(",");
                            pet.setName(split[0]);
                            pet.setAge(Integer.parseInt(split[1]));
                            return pet;
                        }
                        return null;
                    }
                });
            }
        };

    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法2：直接让 WebConfig 实现 WebMvcConfigurer 接口，并重写 configurePathMatch 方法。配置路径映射的规则。
     *  注意：WebMvcConfigurer 对方法进行了默认实现，所以没必要将所有接口进行重写。
     */
//    @Override
//    public void configurePathMatch(PathMatchConfigurer configurer) {
//        UrlPathHelper urlPathHelper = new UrlPathHelper();
//        // 设置不移除分号后面的内容。保证矩阵变量能正常传递
//        urlPathHelper.setRemoveSemicolonContent(false);
//        configurer.setUrlPathHelper(urlPathHelper);
//    }
}

```

```html
    测试自定义封装POJO作为请求参数----级联属性以逗号分隔测试:
    <form action="/savePerson" method="post">
        姓名： <input name="userName" value="zhangsan"/> <br/>
        年龄： <input name="age" value="18"/> <br/>
        生日： <input name="birth" value="2019/12/10"/> <br/>
        宠物： <input name="pet" value="阿猫,3"/>
        <input type="submit" value="保存"/>
    </form>
```

```java
    @PostMapping("/savePerson")
    public Person savePerson(Person person){
        return person;
    }
```

# 3、参数处理原理

32节，没弄完，源码后期再说。

处理请求的入口，从 DispatcherServlet 的 doDispatch 方法开始。

设计思想：

```java
1、从当前HandlerMapping集合中，找到能处理当前请求的Handler（Handler封装了 Controller.method()的签名信息）
2、为当前Handler 找一个适配器 HandlerAdapter，常用 RequestMappingHandlerAdapter
	HandlerAdapter 是 springmvc底层设计的接口。
     未来我们也可以通过该接口去自定义HandlerAdapter 来处理复杂的请求：
     1、声明supports 方法，表示支持哪些Handler
     2、如果是支持的handler，则调用handle方法来进行处理。
3、适配器执行目标方法并确定方法参数的每一个值
```

```java
public class DispatcherServlet extends FrameworkServlet {

	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request); // 检查是否是文件上传请求
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);  // 遍历所有的封装了controller方法的handlerMapping，找到当前URL请求用哪个handlerMapping来处理
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
                // getHandlerAdapter 用于在找到handlerMapping对应的controller方法后，后续的URL参数与请求参数的匹配等逻辑，相关信息集中存放在 HandlerAdapter类中。
                // HandlerAdapter 相当于一个反射工具类，作用就是充当当前handlerMapping的适配器。
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
                // 使用 HandlerAdapter 适配器来处理目标方法。
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}

    @Nullable
	private List<HandlerAdapter> handlerAdapters;

	protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
            // this.handlerAdapters 有4种
            // 1、RequestMappingHandlerAdapter
            // 2、HandlerFunctionAdapter
            // 3、HttpRequestHandlerAdapter
            // 4、SimpleControllerHandlerAdapter
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}

 ======================
    HandlerAdapter 是 springmvc底层设计的接口。
     未来我们也可以通过该接口去自定义HandlerAdapter 来处理复杂的请求：
     1、声明supports 方法，支持哪些Handler
     2、如果是支持的handler，则调用handle方法来进行处理。
public interface HandlerAdapter {
    // 支持处理哪种handler
	boolean supports(Object handler);
    // 如果是支持的handler，用此方法来真正处理。
	@Nullable
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
	long getLastModified(HttpServletRequest request, Object handler);
 }
 ======================   AbstractHandlerMethodAdapter

	@Override
	public final boolean supports(Object handler) {
		return (handler instanceof HandlerMethod && supportsInternal((HandlerMethod) handler));
	}

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646031881544-c7cca878-a531-494c-b2f7-aed1fe662fcf.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646031881544-c7cca878-a531-494c-b2f7-aed1fe662fcf.png)

### 4.1、HandlerAdapter

this.handlerAdapters 包含下方类型的 HandlerAdapter。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646029609921-dcb3ed95-fc46-481d-838c-6d6df9e4b960.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646029609921-dcb3ed95-fc46-481d-838c-6d6df9e4b960.png)

1、RequestMappingHandlerAdapter，用于支持方法上标注了@RequestMapping注解的适配器。

2、HandlerFuntionAdapter，用于支持Controller中的函数式方法。

3、HttpRequestHandlerAdapter，

4、SimpleControllerHandlerAdatper

前两种用的最多。

### 4.2、执行目标方法

DispatcherServlet.java -- doDispatch 中的真正执行处理请求的逻辑：

```java
				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

RequestMappingHandlerAdapter -- handleInternal

```java
			// No synchronization on session demanded at all...
			mav = invokeHandlerMethod(request, response, handlerMethod); // 执行目标方法

```

RequestMappingHandlerAdapter -- invokeHandlerMethod

```java
============

	@Nullable
	private HandlerMethodArgumentResolverComposite argumentResolvers;

	@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		try {
			WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
			ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);
			// Springmvc提前将请求参数处理器和返回值处理器，包装在了 ServletInvocableHandlerMethod 中。
			ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
			if (this.argumentResolvers != null) {
                // 为可执行的方法（即当前Controller的方法参数签名），设置参数解析器。
				invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
			}
			if (this.returnValueHandlers != null) {
				invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
			}
			invocableMethod.setDataBinderFactory(binderFactory);
			invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

			ModelAndViewContainer mavContainer = new ModelAndViewContainer();
			mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
			modelFactory.initModel(webRequest, mavContainer, invocableMethod);
			mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

			AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
			asyncWebRequest.setTimeout(this.asyncRequestTimeout);

			WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
			asyncManager.setTaskExecutor(this.taskExecutor);
			asyncManager.setAsyncWebRequest(asyncWebRequest);
			asyncManager.registerCallableInterceptors(this.callableInterceptors);
			asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

			if (asyncManager.hasConcurrentResult()) {
				Object result = asyncManager.getConcurrentResult();
				mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
				asyncManager.clearConcurrentResult();
				LogFormatUtils.traceDebug(logger, traceOn -> {
					String formatted = LogFormatUtils.formatValue(result, !traceOn);
					return "Resume with async result [" + formatted + "]";
				});
				invocableMethod = invocableMethod.wrapConcurrentResult(result);
			}

			invocableMethod.invokeAndHandle(webRequest, mavContainer);
			if (asyncManager.isConcurrentHandlingStarted()) {
				return null;
			}

			return getModelAndView(mavContainer, modelFactory, webRequest);
		}
		finally {
			webRequest.requestCompleted();
		}
	}

```

### 4.3、为目标方法寻找参数解析器

参数解析器作用：确定将要执行的目标方法的每一个参数的值是什么;

SpringMVC目标方法能支持多少种参数来源。取决于参数解析器有多少种。

HandlerMethodArgumentResolver 参数解析器的类型有26种。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646032677242-eb2f5b19-d0d5-48b6-ab20-9e524dea3c6b.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646032677242-eb2f5b19-d0d5-48b6-ab20-9e524dea3c6b.png)

按照名称就能分辨出不同解析器的作用。

```java
@RestController
public class ParameterTestController {

    @GetMapping("/car/{id}/owner/{userName}")
    public Map<String, Object> getCar(@PathVariable("id") Integer id,  // 被 PathVariableMethodArgumentResolver 处理
                                      @PathVariable("userName") String userName,
                                      @PathVariable Map<String, String> map,
                                      @RequestHeader("User-Agent") String userAgent, //被 RequestHeaderMethodArgumentResolver 处理
                                      @RequestHeader Map<String, String> header,
                                      @RequestParam("age") Integer age,  //被 RequestParamMethodArgumentResolver 处理
                                      @RequestParam("inters") List<String> inters,
                                      @RequestParam Map<String, String> requestParam,
                                      @CookieValue("_ga") String _ga,
                                      @CookieValue("_ga") Cookie cookie) {
        .....
    }

```

HandlerMethodArgumentResolver 参数解析器的统一接口

```java
public interface HandlerMethodArgumentResolver {
    // 判断当前解析器是否支持目标参数
	boolean supportsParameter(MethodParameter parameter);

    // 如果支持目标参数，就开始进行解析。
	@Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

```

### 4.4、返回值处理器

Controller方法能写多少种返回值，取决于返回值处理器。

RequestMappingHandlerAdapter -- invokeHandlerMethod

```java
	@Nullable
	private HandlerMethodReturnValueHandlerComposite returnValueHandlers;

	@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

			.....

			if (this.returnValueHandlers != null) {
				invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
			}
```

HandlerMethodReturnValueHandlerComposite，返回值处理器。有15种类型。

与 请求参数解析器HandlerMethodArgumentResolver 类似，Controller方法能写多少种返回值，取决于返回值处理器。

最常见的，例如 ModelAndView、Model、View、ResponseBody、HttpEntity等。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646034214570-b6f446c6-a66c-491b-9080-12468aade1db.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646034214570-b6f446c6-a66c-491b-9080-12468aade1db.png)

### 4.5、确定目标方法每个参数的值

RequestMappingHandlerAdapter -- invokeHandlerMethod

```java
	@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

        ....
			invocableMethod.invokeAndHandle(webRequest, mavContainer);
        .....

```

ServletInvocableHandlerMethod -- invokeAndHandle

```java
	public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
		// 真正去执行Controller目标方法
		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
        //
		setResponseStatus(webRequest);

		if (returnValue == null) {
			if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
				disableContentCachingIfNecessary(webRequest);
				mavContainer.setRequestHandled(true);
				return;
			}
		}
		else if (StringUtils.hasText(getResponseStatusReason())) {
			mavContainer.setRequestHandled(true);
			return;
		}

		mavContainer.setRequestHandled(false);
		Assert.state(this.returnValueHandlers != null, "No return value handlers");
		try {
			this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
		}
		catch (Exception ex) {
			if (logger.isTraceEnabled()) {
				logger.trace(formatErrorForReturnValue(returnValue), ex);
			}
			throw ex;
		}
	}

	@Nullable
	public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
		// 获取所有方法的参数的值
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		if (logger.isTraceEnabled()) {
			logger.trace("Arguments: " + Arrays.toString(args));
		}
		return doInvoke(args);
	}

	protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
		// 先获取所有方法的参数的详细信息
		MethodParameter[] parameters = getMethodParameters();
		if (ObjectUtils.isEmpty(parameters)) {
			return EMPTY_ARGS;
		}

		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			args[i] = findProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
            // 判断当前解析器是否支持该参数类型
			if (!this.resolvers.supportsParameter(parameter)) {
				throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
			}
			try {
                // 寻找支持当前参数的参数解析器
				args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
			}
			catch (Exception ex) {
				// Leave stack trace for later, exception may actually be resolved and handled...
				if (logger.isDebugEnabled()) {
					String exMsg = ex.getMessage();
					if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
						logger.debug(formatArgumentError(parameter, exMsg));
					}
				}
				throw ex;
			}
		}
		return args;
	}

	public MethodParameter[] getMethodParameters() {
		return this.parameters;
	}

```

HandlerMethod

```java
	private final Method bridgedMethod;

	protected Method getBridgedMethod() {
		return this.bridgedMethod;
	}

```

### 4.5.1、挨个判断所有参数解析器那个支持解析这个参数

HandlerMethodArgumentResolverComposite -- supportsParameter

```java
	private final List<HandlerMethodArgumentResolver> argumentResolvers = new ArrayList<>();

	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return getArgumentResolver(parameter) != null;
	}

	@Nullable
	private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
		HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
		if (result == null) {

			for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
                // 遍历26个参数解析器，哪一个支持解析当前参数。
				if (resolver.supportsParameter(parameter)) {
					result = resolver;
                    // 找到后放入缓存中，防止下次还要重复遍历26个参数解析器。这就是springMvc的缓存机制：第一次请求执行比较慢，后续会越来越快。
					this.argumentResolverCache.put(parameter, result);
					break;
				}
			}
		}
		return result;
	}

============= HandlerMethodArgumentResolver 接口的 supportsParameter 方法，以PathVariableMethodArgumentResolver 参数解析器实现类为例。其它参数解析器执行过程也类似：判断参数是否加了指定注解，然后可选的判断下参数类型。

public class PathVariableMethodArgumentResolver extends AbstractNamedValueMethodArgumentResolver
		implements UriComponentsContributor {
	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		if (!parameter.hasParameterAnnotation(PathVariable.class)) {  // 判断参数是否标注了 @PathVariable 注解
			return false;
		}
		if (Map.class.isAssignableFrom(parameter.nestedIfOptional().getNestedParameterType())) {
			PathVariable pathVariable = parameter.getParameterAnnotation(PathVariable.class);
			return (pathVariable != null && StringUtils.hasText(pathVariable.value()));
		}
		return true;
	}

```

### 4.5.2、解析当前参数的值

InvocableHandlerMethod -- getMethodArgumentValues

```java
	protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
		// 先获取所有方法的参数的详细信息
		MethodParameter[] parameters = getMethodParameters();
		if (ObjectUtils.isEmpty(parameters)) {
			return EMPTY_ARGS;
		}

		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			args[i] = findProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
            // 判断当前解析器是否支持该参数类型
			if (!this.resolvers.supportsParameter(parameter)) {
				throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
			}
			try {
                // 寻找支持当前参数的参数解析器
				args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
			}
			catch (Exception ex) {
				// Leave stack trace for later, exception may actually be resolved and handled...
				if (logger.isDebugEnabled()) {
					String exMsg = ex.getMessage();
					if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
						logger.debug(formatArgumentError(parameter, exMsg));
					}
				}
				throw ex;
			}
		}
		return args;
	}
```

HandlerMethodArgumentResolverComposite -- resolveArgument

```java
	@Override
	@Nullable
	public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
		// 获取当前参数的参数解析器
		HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
		if (resolver == null) {
			throw new IllegalArgumentException("Unsupported parameter type [" +
					parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
		}
        // 调用参数解析器的 resolveArgument 进行解析
		return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
	}
```

参数解析器有26种，我们以 AbstractNamedValueMethodArgumentResolver -- resolveArgument 为例

```java
	@Override
	@Nullable
	public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

		NamedValueInfo namedValueInfo = getNamedValueInfo(parameter);
		MethodParameter nestedParameter = parameter.nestedIfOptional();
		// 解析当前参数的名称
		Object resolvedName = resolveStringValue(namedValueInfo.name);
		if (resolvedName == null) {
			throw new IllegalArgumentException(
					"Specified name must not resolve to null: [" + namedValueInfo.name + "]");
		}
		// 解析当前参数的值
		Object arg = resolveName(resolvedName.toString(), nestedParameter, webRequest);
		if (arg == null) {
			if (namedValueInfo.defaultValue != null) {
				arg = resolveStringValue(namedValueInfo.defaultValue);
			}
			else if (namedValueInfo.required && !nestedParameter.isOptional()) {
				handleMissingValue(namedValueInfo.name, nestedParameter, webRequest);
			}
			arg = handleNullValue(namedValueInfo.name, arg, nestedParameter.getNestedParameterType());
		}
		else if ("".equals(arg) && namedValueInfo.defaultValue != null) {
			arg = resolveStringValue(namedValueInfo.defaultValue);
		}

		if (binderFactory != null) {
			WebDataBinder binder = binderFactory.createBinder(webRequest, null, namedValueInfo.name);
			try {
				arg = binder.convertIfNecessary(arg, parameter.getParameterType(), parameter);
			}
			catch (ConversionNotSupportedException ex) {
				throw new MethodArgumentConversionNotSupportedException(arg, ex.getRequiredType(),
						namedValueInfo.name, parameter, ex.getCause());
			}
			catch (TypeMismatchException ex) {
				throw new MethodArgumentTypeMismatchException(arg, ex.getRequiredType(),
						namedValueInfo.name, parameter, ex.getCause());
			}
		}

		handleResolvedValue(arg, namedValueInfo.name, parameter, mavContainer, webRequest);

		return arg;
	}

	@Nullable
	private Object resolveStringValue(String value) {
		if (this.configurableBeanFactory == null) {
			return value;
		}
		String placeholdersResolved = this.configurableBeanFactory.resolveEmbeddedValue(value);
		BeanExpressionResolver exprResolver = this.configurableBeanFactory.getBeanExpressionResolver();
		if (exprResolver == null || this.expressionContext == null) {
			return value;
		}
		return exprResolver.evaluate(placeholdersResolved, this.expressionContext);
	}

	@Nullable
	protected abstract Object resolveName(String name, MethodParameter parameter, NativeWebRequest request)
			throws Exception;

```

resolveName 方法需要子类实现。

以 PathVariableMethodArgumentResolver 为例。

```java
	@Override
	@SuppressWarnings("unchecked")
	@Nullable
	protected Object resolveName(String name, MethodParameter parameter, NativeWebRequest request) throws Exception {
		// 从请求域中，获取要处理的变量的名称与值。
        Map<String, String> uriTemplateVars = (Map<String, String>) request.getAttribute(
				HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE, RequestAttributes.SCOPE_REQUEST);
		return (uriTemplateVars != null ? uriTemplateVars.get(name) : null);
	}
```

### 4.5.3、如果参数是自定义对象，进行封装

### 4.6、目标方法执行完成

### 4.7、处理派发结果

### 用图示来展示

### p1

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646063075919-df8ab6ae-62ef-4c42-950b-9ad419da9b1f.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646063075919-df8ab6ae-62ef-4c42-950b-9ad419da9b1f.png)

### p2

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646063080083-81189166-2367-4002-b8ec-d76783e794f7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646063080083-81189166-2367-4002-b8ec-d76783e794f7.png)

### p3

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646063084966-61b131fe-c9ae-448c-8dab-c1941b255896.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646063084966-61b131fe-c9ae-448c-8dab-c1941b255896.png)

### p4

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646063090419-59eb0be4-3f7c-4a30-9c23-2a4139532860.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646063090419-59eb0be4-3f7c-4a30-9c23-2a4139532860.png)

# 4、数据响应与内容协商

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646404701804-4db110f2-e60f-4d21-94b3-e1e819c34efb.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646404701804-4db110f2-e60f-4d21-94b3-e1e819c34efb.png)

数据响应分为响应页面、响应数据。

响应页面，会在后面的“视图解析与模板引擎”章节详细说明。常被用于开发单体前后端未分离项目。

相应数据，常用于开发前后端分离项目。

# 4.1、响应JSON

### 4.1.1、如何支持响应数据转为JSON

如果想让springmvc支持Controller自动将返回数据转为JSON。

需要进行以下操作：

### 1、添加 jackson.jar 依赖

spring-boot-starter-web 已经集成。

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

spring-boot-starter-web 场景自动引入了spring-boot-starter-json场景
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>

spring-boot-starter-json场景 自动引入了jackson

   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.11.2</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.datatype</groupId>
      <artifactId>jackson-datatype-jdk8</artifactId>
      <version>2.11.2</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.datatype</groupId>
      <artifactId>jackson-datatype-jsr310</artifactId>
      <version>2.11.2</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.module</groupId>
      <artifactId>jackson-module-parameter-names</artifactId>
      <version>2.11.2</version>
      <scope>compile</scope>
    </dependency>

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646405137669-e5c4757d-7794-464a-a938-da68e1ce0f54.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646405137669-e5c4757d-7794-464a-a938-da68e1ce0f54.png)

### 2、Controller使用@ResponseBody注解

```java
@Controller
public class ResponseTestController {

    @ResponseBody  //利用返回值处理器里面的消息转换器进行处理，转为JSON
    @GetMapping(value = "/test/person")
    public Person getPerson() {
        Person person = new Person();
        person.setAge(28);
        person.setBirth(new Date());
        person.setUserName("zhangsan");
        return person;
    }
}
```

@RestController 相当于 @Controller+  @ResponseBody，所以变为：

```java
@RestController
public class ResponseTestController {

    @GetMapping(value = "/test/person")
    public Person getPerson() {
        Person person = new Person();
        person.setAge(28);
        person.setBirth(new Date());
        person.setUserName("zhangsan");
        return person;
    }
}
```

### 4.1.2、返回值处理器种类

请求参数通过请求参数解析器来处理，对应的，

返回值通过返回值解析器来处理。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646406525096-1fdcf739-42df-4394-86c4-e92e0b980bf9.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646406525096-1fdcf739-42df-4394-86c4-e92e0b980bf9.png)

### 4.1.3、返回值处理器原理

课程对应第37节。具体源码解析码过程，暂且省略。

- 1、返回值处理器通过 supportsReturnType 方法，判断是否支持这种类型返回值
- 2、返回值处理器调用 handleReturnValue 进行处理
- 3、实际处理逻辑，以RequestResponseBodyMethodProcessor为例。 可以处理返回值标了@ResponseBody 注解的方法。

```java
public class RequestResponseBodyMethodProcessor extends AbstractMessageConverterMethodProcessor {

	@Override
	public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
			throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

		mavContainer.setRequestHandled(true);
		ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
		ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

		// Try even with null return value. ResponseBodyAdvice could get involved.
		writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
	}

```

- 利用 MessageConverters 进行处理，判断数据能被写成什么类型。例如 将数据写为json
- 内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
- 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646636437748-723f8b49-fa28-446d-adbe-94bb1f307b42.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646636437748-723f8b49-fa28-446d-adbe-94bb1f307b42.png)

- SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter 转换器，看谁能处理。（HttpMessageConverter转换器：看是否支持将 此 Class类型的对象，转为MediaType类型的数据）以浏览器要求响应JSON为例。
- 遍历得到MappingJackson2HttpMessageConverter转换器，支持将对象写为json
- 利用MappingJackson2HttpMessageConverter将对象转为json再写出去

### 4.1.4、HTTPMessageConverter转换器原理

### 1、MessageConverter接口规范

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646638489210-ff0a6798-f886-4a86-84ab-5a2e36812e04.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646638489210-ff0a6798-f886-4a86-84ab-5a2e36812e04.png)

HttpMessageConverter: 看是否支持将 此 Class类型的对象，转为MediaType类型的数据。

例子：请求进来时，JSON转为Person；请求响应时，Person对象转为JSON。

### 2、默认的MessageConverter

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646638563777-88ee805a-6b43-44b5-a0d4-243b7ffef025.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646638563777-88ee805a-6b43-44b5-a0d4-243b7ffef025.png)

返回值处理器支持处理的类型：

```java
0 - 只支持Byte类型的
1 - String
2 - String
3 - Resource
4 - ResourceRegion
5 - XML解析：DOMSource.class \ SAXSource.class) \ StAXSource.class \StreamSource.class \Source.class
6 - MultiValueMap
7、8 - 支持JSON转换
9 - 支持注解方式xml处理的。

```

以响应JSON数据为例，

最终MappingJackSon2HttpMessageConverter转换器，利用底层的jackson的objectMapper方法，会将结果转为JSON输出。

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1605164243168-1a31e9af-54a4-463e-b65a-c28ca7a8a2fa.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_34%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1605164243168-1a31e9af-54a4-463e-b65a-c28ca7a8a2fa.png)

### 

### 4.1.4、SpringMVC支持的返回值类型

```java
ModelAndView  ModelAndViewMethodReturnValueHandler
Model  ModelMethodProcesser
View  ViewMethodReturnValueHandler
ResponseEntity  ResponseBodyEmitterReturnValueHandler
ResponseBodyEmitter  ResponseBodyEmitterReturnValueHandler
StreamingResponseBody  StreamingResponseBodyReturnValueHandler
HttpEntity  HttpEntityMethodProcessor
HttpHeaders  HttpHeadersReturnValueHandler
Callable(支持异步判断，后续异步编程课程会讲)   CallableMethodReturnValueHandler
DeferredResult(支持异步判断，后续异步编程课程会讲)   DeferredResultMethodReturnValueHandler
ListenableFuture(支持异步判断，后续异步编程课程会讲)   DeferredResultMethodReturnValueHandler
CompletionStage(支持异步判断，后续异步编程课程会讲)   DeferredResultMethodReturnValueHandler
WebAsyncTask（异步任务）  AsyncTaskMethodReturnValueHandler

方法被标注有 @ModelAttribute 且返回值为复杂对象类型，由 ModelAttributeMethodProcessor 处理。
方法被标注有 @ResponseBody ，返回值会被解析为JSON数据。由 RequestResponseBodyMethodProcessor处理

```

通过查看容器中所有返回值解析器 supportsReturnType 方法，就能看出来Controller支持的返回值类型。

返回值解析器以支持的返回值作为名称前缀。

例如 ModelAndViewMethodReturnValueHandler ，支持 ModelAndView 返回类型。

ModelMethodProcessor ，支持 Model 返回类型。

```java
public class ModelAndViewMethodReturnValueHandler implements HandlerMethodReturnValueHandler {
	@Override
	public boolean supportsReturnType(MethodParameter returnType) {
		return ModelAndView.class.isAssignableFrom(returnType.getParameterType());
	}

public class ModelMethodProcessor implements HandlerMethodArgumentResolver, HandlerMethodReturnValueHandler {
	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return Model.class.isAssignableFrom(parameter.getParameterType());
	}

```

# 4.2、内容协商

内容协商：根据客户端接收能力不同，返回不同媒体类型的数据。

例如：浏览器客户端来请求，返回JSON。安卓客户端来请求，返回XML。

### 1、需要引入xml依赖，开启内容协商

```java
 <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

如果不引入依赖，请求声明Accept为 application/xml，那么后端会报警告，前端拿不到返回信息。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646641063182-05b3fcce-480b-42c0-bd36-856bfe2005fe.png](https://cdn.nlark.com/yuque/0/2022/png/623390/1646641063182-05b3fcce-480b-42c0-bd36-856bfe2005fe.png)

### 2、测试返回json和xml

只需要改变请求头中Accept字段即可。

Http协议中规定的，Accept 告诉服务器本客户端可以接收的数据类型。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646640726968-e619a83d-7f56-4e57-9a57-22e953258a74.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646640726968-e619a83d-7f56-4e57-9a57-22e953258a74.png)



### 3、内容协商原理

内容协商原理简单说明：

Controller方法返回值支持返回页面和返回数据。

可以通过@ResponseBody方法注解，声明该方法是用来响应数据。

响应数据会调用 RequestResponseBodyMethodProcessor 返回值处理器进行处理。

调用各种 MessageConverter 消息类型转换器 进行数据类型的转换。

所有的 MessageConverter 消息类型转换器，集合起来，可以支持对各种媒体类型数据的操作，读/写。

而内容协商，就是获取客户端想要的数据类型，与服务器里所有 MessageConverter 消息类型转换器 支持处理的数据类型，进行匹配。

```java
ContentNegotiationManager  内容协商管理器
	 ParameterContentNegotiationStrategy
		基于请求参数的内容协商管理器
		由spring.mvc.contentnegotiation.favor-parameter:true开启
		默认请求参数format只能传递 xml/json。分别对应数据媒体类型：application/xml、application/json

	 HeaderContentNegotiationStrategy
		基于请求头的内容协商管理器
		系统默认
```

以返回xml数据为例：

- 1、判断当前响应头中是否已经有确定的媒体类型。MediaType
- 2、获取客户端（PostMan、浏览器）支持接收的内容类型。即获取请求中的客户端Accept请求头字段，例如 application/xml
- 通过内容协商管理器 contentNegotiationManager来获取。

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1605230462280-ef98de47-6717-4e27-b4ec-3eb0690b55d0.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_15%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1605230462280-ef98de47-6717-4e27-b4ec-3eb0690b55d0.png)

- 内容协商管理器 contentNegotiationManager默认使用基于请求头的策略 HeaderContentNegotiationStrategy 。获取请求中的客户端Accept请求头字段，当做客户端能接收的请求返回类型。

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1605230546376-65dcf657-7653-4a58-837a-f5657778201a.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_28%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1605230546376-65dcf657-7653-4a58-837a-f5657778201a.png)

- 3、遍历循环所有当前系统的 **MessageConverter（第一次匹配转换器）**，看谁支持操作这个对象（Person）

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646641923095-723e5356-e2bd-4d92-8c67-ec5d846f9107.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646641923095-723e5356-e2bd-4d92-8c67-ec5d846f9107.png)

- 4、找到支持操作Person的converter，把converter支持的媒体类型统计出来。

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1605173876646-f63575e2-50c8-44d5-9603-c2d11a78adae.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1605173876646-f63575e2-50c8-44d5-9603-c2d11a78adae.png)

- 5、客户端需要【application/xml】。服务端能处理10种类型数据。包括json、xml。
- 6、进行内容协商，客户端需要的、与服务端能提供的，进行比对，获得最佳匹配的媒体类型。然后去重。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646642181508-baa34caa-277b-493a-a3c8-7a3cce9e45a0.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646642181508-baa34caa-277b-493a-a3c8-7a3cce9e45a0.png)

- 7、寻找出能够将返回值对象转为最佳匹配媒体类型 的**MessageConverter（第二次匹配转换器）**。调用它进行转化 。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646642486049-4a208e68-4745-4d43-9fe2-73d903b49c2e.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646642486049-4a208e68-4745-4d43-9fe2-73d903b49c2e.png)

### 4、开启浏览器参数方式内容协商功能

浏览器无法像Postman那样自定义请求头，除非发送Ajax请求，规定好返回值的Content-type。并且，浏览器默认请求头为下图所示，优先接收xml类型。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646644024348-9339c91d-7faa-4c4a-bd79-f50dbf927ec0.png](https://cdn.nlark.com/yuque/0/2022/png/623390/1646644024348-9339c91d-7faa-4c4a-bd79-f50dbf927ec0.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646716188287-0c792951-c8b1-4d62-91dc-4309e4a3a4ff.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646716188287-0c792951-c8b1-4d62-91dc-4309e4a3a4ff.png)

为了返回JSON，除了更改请求头之外，Springmvc底层开启了对浏览器进行内容协商的快速支持。

即基于请求参数的内容，进行协商。

```java
spring:
	mvc:
        contentnegotiation:
          favor-parameter: true   # 开启基于请求参数，进行内容协商的模式
```

通过请求参数中的format字段，指定返回内容回类型。

[http://localhost:8080/test/person?format=json](http://localhost:8080/test/person?format=json)

[http://localhost:8080/test/person?format=](http://localhost:8080/test/person?format=json)xml  新版的chromem可能失效。

- 原理

通过配置，生成一个基于请求参数进行解析返回数据类型的内容协商管理器。优先于默认的基于请求头的内容协商管理器运行。

从而通过其约定的format参数来制定返回jsono还是xml。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646709855045-0363b9bc-7659-47e6-9b2a-2fc331bc02d7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646709855045-0363b9bc-7659-47e6-9b2a-2fc331bc02d7.png)

### 5、自定义 MessageConverter 消息类型转换器

场景描述：

如果公司里存在自定义的协议和数据类型，就需要进行自定义MessageConverter。

所有MessageConverter合起来，可以支持各种媒体类型的读/写操作。

与开启浏览器参数方式内容协商功能类似，如果想自定义返回其它类型数据，例如pdf，或是指定拼接方式的字符串，可以自定义内容协商管理器进行配置。

实现目标：

实现多协议数据兼容。json、xml、x-guigu

0、@ResponseBody 响应数据出去 调用 RequestResponseBodyMethodProcessor 处理

1、Processor 处理方法返回值。通过 MessageConverter 处理

2、所有 MessageConverter 合起来可以支持各种媒体类型数据的操作（读、写）

3、内容协商找到最终的 messageConverter；

### 请求头传入自定义协议时返回自定义数据

springmvc中，原生 MessageConverter 消息类型转换器 的配置生效方式：

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646718611899-067c56ed-fddf-4be3-b28b-d36e2a780718.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646718611899-067c56ed-fddf-4be3-b28b-d36e2a780718.png)

对应的，如果想实现根据请求头，返回自定义的数据类型，就需要新建一个自定义的消息类型转换器。

```java
package com.learn.boot.converter;

import com.learn.boot.bean.Person;
import org.springframework.http.HttpInputMessage;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.http.converter.HttpMessageNotWritableException;

import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

/**
 * 自定义的Converter
 */
public class GuiguMessageConverter implements HttpMessageConverter<Person> {

    /**
     * 用于请求参数对URL数据匹配时
     * 是否可以对Person类型进行读取
     * 即判断是否支持将 clazz 类型的读取为容器支持的 mediaType 媒体类型集合中的某一个类型的数据
     * 例如，请求参数中，给Person person，加上@RequestBody注解，就支持将Person类型数据转为Json类型读取
     *
     * @param clazz
     * @param mediaType
     * @return
     */
    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false;
    }

    /**
     * 用于请求响应
     * 是否支持对 clazz 类型进行写操作
     * 即将 clazz 类型写为指定的 MediaType 媒体类型集合中的某一个
     * @param clazz
     * @param mediaType
     * @return
     */
    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        // isAssignableFrom 当前参数是否是指定类型
        // 相当于直接写了true，因为 implements HttpMessageConverter<Person> 已经指定了class为Person类型
        // 且摆脱了走 mediaType 媒体类型的匹配
        return clazz.isAssignableFrom(Person.class);
    }

    /**
     * 当前消息处理器都支持处理哪些类型数据
     * 因为服务器要统计所有MessageConverter都支持哪些内容类型，统计时就是调用的该方法
     * @return
     */
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-guigu");
    }

    /**
     * 用于请求参数对URL数据匹配时
     * 当该类型数据可读时，执行的读方法
     * 当前案例只做返回值处理演示，直接返回null
     * @param clazz
     * @param inputMessage
     * @return
     * @throws IOException
     * @throws HttpMessageNotReadableException
     */
    @Override
    public Person read(Class<? extends Person> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    /**
     * 用于请求响应
     * 自定义协议数据的写出
     * @param person
     * @param contentType
     * @param outputMessage
     * @throws IOException
     * @throws HttpMessageNotWritableException
     */
    @Override
    public void write(Person person, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        //自定义协议数据的写出
        String data = person.getUserName()+";"+person.getAge()+";"+person.getBirth();
        //写出去
        OutputStream body = outputMessage.getBody();
        body.write(data.getBytes());
    }
}

```

然后添加到配置中

```java
package com.learn.boot.config;

import com.learn.boot.bean.Pet;
import com.learn.boot.converter.GuiguMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.format.FormatterRegistry;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.HiddenHttpMethodFilter;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.util.UrlPathHelper;

import java.util.List;

@Configuration(proxyBeanMethods = false)  // 配置类中添加组件的方法之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
public class WebConfig /*implements WebMvcConfigurer*/ {

    // 自定义REST的method声明参数
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter() {
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法一：在此处配置类里，自定义一个 WebMvcConfigurer 实现类，并标注 @Bean 注入即可。
     *
     * @return
     */
    @Bean
    public WebMvcConfigurer webMvcCOnfigurer(){
        return new WebMvcConfigurer(){

            /**
             * 针对基于请求头的内容协商管理器HeaderContentNegotiationStrategy，
             * 添加当传递自定义的accept请求协议时，返回自定义的格式数据的，自定义参数处理器
             * @param converters
             */
            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                converters.add(new GuiguMessageConverter());
            }

            @Override
            public void configurePathMatch(PathMatchConfigurer configurer){
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                // 设置不移除分号后面的内容。保证矩阵变量能正常传递
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }

            @Override
            public void addFormatters(FormatterRegistry registry) {
                // 添加自定义参数转换器 ，String -> Pet
                // 此处演示请求方法的自定义类型参数，支持多个属性以逗号分隔，进行传递。宠物： <input name="pet" value="阿猫,3"/>
                registry.addConverter(new Converter<String, Pet>() {
                    // S 原类型   T 目标类型
                    @Override
                    public Pet convert(String source) {
                        // 啊猫,3
                        if(!StringUtils.isEmpty(source)){
                            Pet pet = new Pet();
                            String[] split = source.split(",");
                            pet.setName(split[0]);
                            pet.setAge(Integer.parseInt(split[1]));
                            return pet;
                        }
                        return null;
                    }
                });
            }
        };

    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法2：直接让 WebConfig 实现 WebMvcConfigurer 接口，并重写 configurePathMatch 方法。配置路径映射的规则。
     *  注意：WebMvcConfigurer 对方法进行了默认实现，所以没必要将所有接口进行重写。
     */
//    @Override
//    public void configurePathMatch(PathMatchConfigurer configurer) {
//        UrlPathHelper urlPathHelper = new UrlPathHelper();
//        // 设置不移除分号后面的内容。保证矩阵变量能正常传递
//        urlPathHelper.setRemoveSemicolonContent(false);
//        configurer.setUrlPathHelper(urlPathHelper);
//    }
}

```

效果：

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646749159565-24c9be22-23c9-4eba-b617-0dfd84c80dbf.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646749159565-24c9be22-23c9-4eba-b617-0dfd84c80dbf.png)

最终，基于请求头的内容协商管理器，统计可以处理的数据类型转换器时，会带上自定义的转化器。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646832125340-01d970d8-92a9-49d4-a432-49547f73c2f9.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646832125340-01d970d8-92a9-49d4-a432-49547f73c2f9.png)

### 请求参数传入自定义协议时返回自定义数据

基本思路：

如果想通过指定请求参数format的值来返回指定格式的数据，

1、format来指定对应返回值类型，这种内容协商策略，依赖于 ParameterContentNegotiationStrategy （基于请求参数的内容协商管理器） 来实现。

2、给WebMvcConfigurer 中，通过 WebMvcConfigurer.configureContentNegotiation（配置内容协商功能） 方法，重新设置内容协商策略。

3、修改 ParameterContentNegotiationStrategy 中的内容协商策略，添加一个 format为gg时，对应媒体类型协议为 application/x-guigu 的对应关系。

4、这样，就能在媒体类型协议与数据类型转换器匹配时，找到我们创建的自定义数据类型转换器。

```java
package com.learn.boot.config;

import com.learn.boot.bean.Pet;
import com.learn.boot.converter.GuiguMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.format.FormatterRegistry;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.util.StringUtils;
import org.springframework.web.accept.HeaderContentNegotiationStrategy;
import org.springframework.web.accept.ParameterContentNegotiationStrategy;
import org.springframework.web.filter.HiddenHttpMethodFilter;
import org.springframework.web.servlet.config.annotation.ContentNegotiationConfigurer;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.util.UrlPathHelper;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Configuration(proxyBeanMethods = false)  // 配置类中添加组件的方法之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
public class WebConfig /*implements WebMvcConfigurer*/ {

    // 自定义REST的method声明参数
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter() {
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法一：在此处配置类里，自定义一个 WebMvcConfigurer 实现类，并标注 @Bean 注入即可。
     *
     * @return
     */
    @Bean
    public WebMvcConfigurer webMvcCOnfigurer(){
        return new WebMvcConfigurer(){

            /**
             * 针对基于请求参数的内容协商管理器ParameterContentNegotiationStrategy
             * 在原有的format参数只支持json、xml的基础上，
             * 新增format=gg时，返回自定义的格式数据
             */
            @Override
            public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {

                // 重新创建 ParameterContentNegotiationStrategy，指定支持的format的参数值和对应的参数类型协议
                Map<String, MediaType> mediaTypeMap = new HashMap<>();
                mediaTypeMap.put("json",MediaType.APPLICATION_JSON); // application/json
                mediaTypeMap.put("xml",MediaType.APPLICATION_XML); // application/xml
                mediaTypeMap.put("gg",MediaType.parseMediaType("application/x-guigu")); // application/x-guigu
                ParameterContentNegotiationStrategy parameterStrategy = new ParameterContentNegotiationStrategy(mediaTypeMap);

                //如果自定义内容协商策略时，只设置的 ParameterContentNegotiationStrategy，没有设置 HeaderContentNegotiationStrategy，
                //会导致根据请求头Accept匹配返回结果的功能失效，
                //没有 HeaderContentNegotiationStrategy 解析 Accept，如果URL不传递format请求参数时，系统通过Accept读取到的请求参数协议永远为 */*，即能匹配所有数据类型。
                //这时，系统会返回支持的数据类型列表中，权重最高的那个作为返回值类型。即application/json。
                //最终，如果URL不传递format请求参数，无论Accept指定任意值，都会返回json数据。
                HeaderContentNegotiationStrategy headerStrategy = new HeaderContentNegotiationStrategy();

                // ContentNegotiationConfigurer.strategies  重新指定内容协商策略支持的内容协商管理器
                // 默认支持 HeaderContentNegotiationStrategy(基于请求头的内容协商管理器)
                // spring.mvc.contentnegotiation.favor-parameter:true开启后，会添加上 ParameterContentNegotiationStrategy（基于请求参数的内容协商管理器）
                // 但此处重写 configureContentNegotiation方法后，配置会以这里为准
                configurer.strategies(Arrays.asList(parameterStrategy,headerStrategy));
            }

            /**
             * 针对基于请求头的内容协商管理器HeaderContentNegotiationStrategy，
             * 添加当传递自定义的accept请求协议时，返回自定义的格式数据的，自定义参数处理器
             * @param converters
             */
            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                converters.add(new GuiguMessageConverter());
            }

            @Override
            public void configurePathMatch(PathMatchConfigurer configurer){
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                // 设置不移除分号后面的内容。保证矩阵变量能正常传递
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }

            @Override
            public void addFormatters(FormatterRegistry registry) {
                // 添加自定义参数转换器 ，String -> Pet
                // 此处演示请求方法的自定义类型参数，支持多个属性以逗号分隔，进行传递。宠物： <input name="pet" value="阿猫,3"/>
                registry.addConverter(new Converter<String, Pet>() {
                    // S 原类型   T 目标类型
                    @Override
                    public Pet convert(String source) {
                        // 啊猫,3
                        if(!StringUtils.isEmpty(source)){
                            Pet pet = new Pet();
                            String[] split = source.split(",");
                            pet.setName(split[0]);
                            pet.setAge(Integer.parseInt(split[1]));
                            return pet;
                        }
                        return null;
                    }
                });
            }
        };

    }

    /**
     * 修改 springmvc 原有的请求路径匹配规则，放开URL分号后参数解析开关。
     * 方法2：直接让 WebConfig 实现 WebMvcConfigurer 接口，并重写 configurePathMatch 方法。配置路径映射的规则。
     *  注意：WebMvcConfigurer 对方法进行了默认实现，所以没必要将所有接口进行重写。
     */
//    @Override
//    public void configurePathMatch(PathMatchConfigurer configurer) {
//        UrlPathHelper urlPathHelper = new UrlPathHelper();
//        // 设置不移除分号后面的内容。保证矩阵变量能正常传递
//        urlPathHelper.setRemoveSemicolonContent(false);
//        configurer.setUrlPathHelper(urlPathHelper);
//    }
}

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646835672210-f4253bf8-9e73-4bee-9349-d5f84c2373fc.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646835672210-f4253bf8-9e73-4bee-9349-d5f84c2373fc.png)

- 源码中的装配效果：

以前的

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1605260623995-8b1f7cec-9713-4f94-9cf1-8dbc496bd245.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_18%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1605260623995-8b1f7cec-9713-4f94-9cf1-8dbc496bd245.png)

重新配置后

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646833612947-f5735ef7-efeb-4ce7-ad4c-ecbdf0fa9695.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646833612947-f5735ef7-efeb-4ce7-ad4c-ecbdf0fa9695.png)

- 注意：

如果自定义内容协商策略时，只设置的 ParameterContentNegotiationStrategy，没有设置 HeaderContentNegotiationStrategy，会导致根据请求头Accept匹配返回结果的功能失效，

没有 HeaderContentNegotiationStrategy 解析 Accept，如果URL不传递format请求参数时，系统通过Accept读取到的请求参数协议永远为 */*，即能匹配所有数据类型。

这时，系统会返回支持的数据类型列表中，权重最高的那个作为返回值类型。即application/json。

最终，如果URL不传递format请求参数，无论Accept指定任意值，都会返回json数据。

如下图所示：

![https://cdn.nlark.com/yuque/0/2022/png/623390/1646834823610-8e81ebde-1776-4bdb-9358-fad663487b8d.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1646834823610-8e81ebde-1776-4bdb-9358-fad663487b8d.png)

### 注意

有可能我们添加的自定义的功能会覆盖默认很多功能，导致一些默认的功能失效。这时，必须以debug源码的方式，找出来缺失了什么功能，然后补充上去。