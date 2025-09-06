---
title: SpringMVC（进阶）
icon: streamline-ultimate-color:notes-book
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - SPRINGMVC
tag:
  - springmvc
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/07wpnkqr/
---



# **9、SpringMVC处理ajax请求**

## **9.1、@RequestBody**

@RequestBody可以获取请求体信息，使用@RequestBody注解标识控制器方法的形参，当前请求的请求体就会为当前注解所标识的形参赋值

```
 <!--此时必须使用post请求方式，因为get请求没有请求体-->
 <form th:action="@{/test/RequestBody}" method="post">
     用户名：<input type="text" name="username"><br>
     密码：<input type="password" name="password"><br>
     <input type="submit">
 </form>
 @RequestMapping("/test/RequestBody")
 public String testRequestBody(@RequestBody String requestBody){
     System.out.println("requestBody:"+requestBody);
     return "success";
 }
```

输出结果：

requestBody:username=admin&password=123456

## **9.2、@RequestBody获取json格式的请求参数**

> 在使用了axios发送ajax请求之后，浏览器发送到服务器的请求参数有两种格式：
>
> 1、name=value&name=value...，此时的请求参数可以通过request.getParameter()获取，对应
>
> SpringMVC中，可以直接通过控制器方法的形参获取此类请求参数
>
> 2、{key:value,key:value,...}，此时无法通过request.getParameter()获取，之前我们使用操作
>
> json的相关jar包gson或jackson处理此类请求参数，可以将其转换为指定的实体类对象或map集
>
> 合。在SpringMVC中，直接使用@RequestBody注解标识控制器方法的形参即可将此类请求参数
>
> 转换为java对象

使用@RequestBody获取json格式的请求参数的条件：

1、导入jackson的依赖

```
 <dependency>
     <groupId>com.fasterxml.jackson.core</groupId>
     <artifactId>jackson-databind</artifactId>
     <version>2.12.1</version>
 </dependency>
```

2、SpringMVC的配置文件中设置开启mvc的注解驱动

```
 <!--开启mvc的注解驱动-->
 <mvc:annotation-driven />
```

3、在控制器方法的形参位置，设置json格式的请求参数要转换成的java类型（实体类或map）的参

数，并使用@RequestBody注解标识

```
 <input type="button" value="测试@RequestBody获取json格式的请求参数"@click="testRequestBody()"><br>
 <script type="text/javascript" th:src="@{/js/vue.js}"></script>
 <script type="text/javascript" th:src="@{/js/axios.min.js}"></script>
 <script type="text/javascript">
     var vue = new Vue({
         el:"#app",
         methods:{
             testRequestBody(){
                 axios.post(
                     "/SpringMVC/test/RequestBody/json",
                     {username:"admin",password:"123456"}
                 ).then(response=>{
                     console.log(response.data);
                 });
             }
         }
     });
 </script>
 //将json格式的数据转换为map集合
 @RequestMapping("/test/RequestBody/json")
 public void testRequestBody(@RequestBody Map<String, Object> map,HttpServletResponse response) throws IOException {
     System.out.println(map);
     //{username=admin, password=123456}
     response.getWriter().print("hello,axios");
 }
 //将json格式的数据转换为实体类对象
 @RequestMapping("/test/RequestBody/json")
 public void testRequestBody(@RequestBody User user, HttpServletResponseresponse) throws IOException {
     System.out.println(user);
     //User{id=null, username='admin', password='123456', age=null,gender='null'}
   response.getWriter().print("hello,axios");
 }
```

## **9.3、@ResponseBody**

@ResponseBody用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应到浏览器

```
 @RequestMapping("/testResponseBody")
 public String testResponseBody(){
     //此时会跳转到逻辑视图success所对应的页面
     return "success";
 }
 @RequestMapping("/testResponseBody")
 @ResponseBody
 public String testResponseBody(){
     //此时响应浏览器数据success
     return "success";
 }
```

## **9.4、@ResponseBody响应浏览器json数据**

服务器处理ajax请求之后，大多数情况都需要向浏览器响应一个java对象，此时必须将java对象转换为

json字符串才可以响应到浏览器，之前我们使用操作json数据的jar包gson或jackson将java对象转换为

json字符串。在SpringMVC中，我们可以直接使用@ResponseBody注解实现此功能

@ResponseBody响应浏览器json数据的条件：

1、导入jackson的依赖

```
 <dependency>
     <groupId>com.fasterxml.jackson.core</groupId>
     <artifactId>jackson-databind</artifactId>
     <version>2.12.1</version>
 </dependency>
```

2、SpringMVC的配置文件中设置开启mvc的注解驱动

```
 <!--开启mvc的注解驱动-->
 <mvc:annotation-driven />
```

3、使用@ResponseBody注解标识控制器方法，在方法中，将需要转换为json字符串并响应到浏览器

的java对象作为控制器方法的返回值，此时SpringMVC就可以将此对象直接转换为json字符串并响应到浏览器

```
 <input type="button" value="测试@ResponseBody响应浏览器json格式的数据"@click="testResponseBody()"><br>
 <script type="text/javascript" th:src="@{/js/vue.js}"></script>
 <script type="text/javascript" th:src="@{/js/axios.min.js}"></script>
 <script type="text/javascript">
     var vue = new Vue({
         el:"#app",
         methods:{
             testResponseBody(){
                 axios.post("/SpringMVC/test/ResponseBody/json").then(response=>{
                     console.log(response.data);
                 });
             }
         }
     });
 </script>
 //响应浏览器list集合
 @RequestMapping("/test/ResponseBody/json")
 @ResponseBody
 public List<User> testResponseBody(){
     User user1 = new User(1001,"admin1","123456",23,"男");
     User user2 = new User(1002,"admin2","123456",23,"男");
     User user3 = new User(1003,"admin3","123456",23,"男");
     List<User> list = Arrays.asList(user1, user2, user3);
     return list;
 }
 //响应浏览器map集合
 @RequestMapping("/test/ResponseBody/json")
 @ResponseBody
 public Map<String, Object> testResponseBody(){
     User user1 = new User(1001,"admin1","123456",23,"男");
     User user2 = new User(1002,"admin2","123456",23,"男");
     User user3 = new User(1003,"admin3","123456",23,"男");
     Map<String, Object> map = new HashMap<>();
     map.put("1001", user1);
     map.put("1002", user2);
     map.put("1003", user3);
     return map;
 }
 //响应浏览器实体类对象
 @RequestMapping("/test/ResponseBody/json")
 @ResponseBody
 public User testResponseBody(){
     return user;
 }
```

## **9.5、@RestController注解**

@RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了

@Controller注解，并且为其中的每个方法添加了@ResponseBody注解

# **10、文件上传和下载**

## **10.1、文件下载**

ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文

使用ResponseEntity实现下载文件的功能

```java
 @RequestMapping("/testDown")
//返回值类型是ResponseEntity，作为请求的返回值实体
 public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws
     IOException {
     //获取ServletContext对象
     ServletContext servletContext = session.getServletContext();
     //获取服务器中文件的真实路径 
     String realPath = servletContext.getRealPath("/static/img/1.jpg");
     //创建输入流
     InputStream is = new FileInputStream(realPath);
     //创建字节数组
     byte[] bytes = new byte[is.available()];available() 文件输入流中所有的字节数
     //将流读到字节数组中
     is.read(bytes);
     //创建HttpHeaders对象设置响应头信息
     MultiValueMap<String, String> headers = new HttpHeaders();
     //设置要下载方式以及下载文件的名字，filename：下载文件的默认名字，attachment：以附件的形式下载
     headers.add("Content-Disposition", "attachment;filename=1.jpg");
     //设置响应状态码
     HttpStatus statusCode = HttpStatus.OK;
     //创建ResponseEntity对象
     ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers,statusCode);
     //关闭输入流
     is.close();
//返回响应实体 
     return responseEntity;
 }
```

## **10.2、文件上传**

文件上传要求form表单的请求方式必须为**post**，并且添加属性enctype="multipart/form-data"

SpringMVC中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

上传步骤：

只有上传功能需要导入依赖。

### **①添加依赖：**

```xml
<!-- <https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload> -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

### **②在SpringMVC的配置文件中添加配置：**

```xml
<!--必须通过文件解析器的解析才能将文件转换为MultipartFile对象-->
<bean id="multipartResolver"
//接口不能配置在这里，必须配置其实现类
class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
</bean>
```

### **③控制器方法：**

```java
@RequestMapping("/testUp")
//上传的文件被封装为MultipartFile 对象
public String testUp(MultipartFile photo, HttpSession session) throws IOException {
    //获取上传的文件的文件名
    String fileName = photo.getOriginalFilename();
    //处理文件重名问题
    String hzName = fileName.substr ing(fileName.lastIndexOf("."));
    fileName = UUID.randomUUID().toString() + hzName;
    //获取服务器中photo目录的路径
    ServletContext servletContext = session.getServletContext();
    String photoPath = servletContext.getRealPath("photo");
    File file = new File(photoPath);
    if(!file.exists()){
        file.mkdir();
    }
    String finalPath = photoPath + File.separator + fileName;
    //实现上传功能
    photo.transferTo(new File(finalPath));
    return "success";
}
```

# **11、拦截器**

浏览器—-》filter—→dispatesurvlet—-》controller：dispatesurvlet在接收到请求之后，会对当前的请求进行处理，根据请求信息，去和请求映射requestmapping进行匹配，找到请求映射，请求映射对应的controller就是执行的方法。 控制器方法执行后，返回的是一个统一的modelandview对象，其作用是将模型数据填充到视图中为用户填充数据。

filter作用在，浏览器—-》filter—→dispatesurvlet这两个组件之间,而拦截器作用在controller控制前的前后。

> 拦截器是用来拦截控制器方法的，一个是在控制器方法执行之前执行，一个是在控制器方法执行之后执行，一个在渲染视图完毕之后执行。

## **11.1、拦截器的配置**

SpringMVC中的拦截器用于拦截控制器方法的执行

SpringMVC中的拦截器需要实现HandlerInterceptor

SpringMVC的拦截器必须在SpringMVC的配置文件中进行配置：

```xml
 -- 配置当前拦截器对象
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
 <ref bean="firstInterceptor"></ref>
 <!-- 以上两种配置方式都是对DispatcherServlet所处理的所有的请求进行拦截 -->
 <mvc:interceptor>
-- 设置拦截路径：/* 表示匹配一层目录
-- /**：拦截所有层目录
     <mvc:mapping path="/**"/>
-- 排除不拦截哪些路径 
     <mvc:exclude-mapping path="/testRequestEntity"/>
-- 引用bean对象，默认对所有的请求进行拦截
     <ref bean="firstInterceptor"></ref>
 </mvc:interceptor>
 <!--
     以上配置方式可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求，
     通过mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求
 -->
```

## **11.2、拦截器的三个抽象方法**

SpringMVC中的拦截器有三个抽象方法：

preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法

postHandle：控制器方法执行之后执行postHandle()

afterCompletion：处理完视图和模型数据，渲染视图完毕之后执行afterCompletion()

## **11.3、多个拦截器的执行顺序**

preHandle:多个拦截器按照配置文件中的配置顺序执行，postHandle按照配置的反序执行，afterCompletion也是按照配置的相反顺序执行。

①若每个拦截器的preHandle()都返回true

此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的配置顺序有关：

preHandle()会按照配置的顺序执行，而postHandle()和afterCompletion()会按照配置的反序执行

②若某个拦截器的preHandle()返回了false

preHandle()返回false和它之前的拦截器的preHandle()都会执行，postHandle()都不执行，返回false的拦截器之前的拦截器的afterCompletion()会执行

# **12、异常处理器**

## **12.1、基于配置的异常处理**

SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver

HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和

SimpleMappingExceptionResolver

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式：

```xml
 <bean
       class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
     <property name="exceptionMappings">
         <props>
             <!--
                 properties的键表示处理器方法执行过程中出现的异常
                 properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
       -->
-- 配置出现的异常
             <prop key="java.lang.ArithmeticException">error</prop>
         </props>
     </property>
     <!--
     exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享
   -->
-- 配置存储的错误信息，存储在请求域中
     <property name="exceptionAttribute" value="ex"></property>
 </bean>
```

## **12.2、基于注解的异常处理**

```java
 //@ControllerAdvice将当前类标识为异常处理的组件
 @ControllerAdvice
 public class ExceptionController {
     //@ExceptionHandler用于设置所标识方法处理的异常，这里可以写多个异常类型
     @ExceptionHandler(ArithmeticException.class)
     //ex表示当前请求处理中出现的异常对象
     public String handleArithmeticException(Exception ex, Model model){
-- 将异常添加到请求域中
         model.addAttribute("ex", ex);
         return "error";-- 返回视图名称
     }
 }
```

# **13、注解配置SpringMVC**

使用**配置类和注解**代替web.xml和SpringMVC配置文件的功能，

## **13.1、创建初始化类，代替web.xml**

在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，

如果找到的话就用它来配置Servlet容器。 Spring提供了这个接口的实现，名为

SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为

AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了

AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

```java
 -- web工程的初始化类，用来代替web.xml
public class WebInit extends
     AbstractAnnotationConfigDispatcherServletInitializer {
     /**
      * 指定spring的配置类
      * @return
    */
     @Override
     protected Class<?>[] getRootConfigClasses() {
         return new Class[]{SpringConfig.class};
     }
     /**
      * 指定SpringMVC的配置类
      * @return
      */
     @Override
     protected Class<?>[] getServletConfigClasses() {
         return new Class[]{WebConfig.class};
     }
     /**
      * 指定DispatcherServlet的映射规则，即url-pattern
      * @return
      */
     @Override
     protected String[] getServletMappings() {
         return new String[]{"/"};
     }
     /**
      * 添加过滤器，需要什么规则，就将规则放到数组中返回
      * @return
      */
     @Override
     protected Filter[] getServletFilters() {
         CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
         encodingFilter.setEncoding("UTF-8");
         encodingFilter.setForceRequestEncoding(true);
         HiddenHttpMethodFilter hiddenHttpMethodFilter = newHiddenHttpMethodFilter();
         return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
     }
 }
```

## **13.2、创建SpringConfig配置类，代替spring的配置文件**

```java
 -- 将类标示为配置类
@Configuration
 public class SpringConfig {
     //ssm整合之后，spring的配置信息写在此类中
 }
```

## **13.3、创建WebConfig配置类，代替SpringMVC的配置文件**

springmvc配置文件：

- 扫描组件
- 视图解析
- view-controller
- default-servlet-habndler
- mvc注解驱动
- 文件上传解析器
- 异常处理解析器
- 拦截器

```java
 @javaonfiguration
 //扫描组件
 @ComponentScan("com.atguigu.mvc.controller")
 //开启MVC注解驱动
//继承WebMvcConfigurer 接口是为了配置不是bean的默认属性，因为视图解析器是bean对象，可以直接继承，但是非bean配置需要实现接口
 @EnableWebMvc
 public class WebConfig implements WebMvcConfigurer {
     //使用默认的servlet处理静态资源
     @Override
     public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
         configurer.enable();
     }
     //配置文件上传解析器
     @Bean
     public CommonsMultipartResolver multipartResolver(){
         return new CommonsMultipartResolver();
     }
     //配置拦截器
     @Override
     public void addInterceptors(InterceptorRegistry registry) {
         FirstInterceptor firstInterceptor = new FirstInterceptor();
         registry.addInterceptor(firstInterceptor).addPathPatterns("/**");
     }
     //配置视图控制
     /*@Override
   public void addViewControllers(ViewControllerRegistry registry) {
     registry.addViewController("/").setViewName("index");
   }*/
     //配置异常映射
     /*@Override
   public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
     SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
     Properties prop = new Properties();
     prop.setProperty("java.lang.ArithmeticException", "error");
     //设置异常映射
     exceptionResolver.setExceptionMappings(prop);
     //设置共享异常信息的键
     exceptionResolver.setExceptionAttribute("ex");
     resolvers.add(exceptionResolver);
   }*/
//创建三个bean有顺序依赖，从外到内
     //配置生成模板解析器
     @Bean
     public ITemplateResolver templateResolver() {
-- 容器
         WebApplicationContext webApplicationContext =ContextLoader.getCurrentWebApplicationContext();
             // ServletContextTemplateResolver需要一个ServletContext作为构造参数，可通过WebApplicationContext 的方法获得
             ServletContextTemplateResolver templateResolver = new
                 ServletContextTemplateResolver(webApplicationContext.getServletContext());
         templateResolver.setPrefix("/WEB-INF/templates/");
         templateResolver.setSuffix(".html");
         templateResolver.setCharacterEncoding("UTF-8");
         templateResolver.setTemplateMode(TemplateMode.HTML);
         return templateResolver;
     }
     //生成模板引擎并为模板引擎注入模板解析器
//ITemplateResolver ：参数springmvc会自动装配，他会去容器中找到一个bean,为参数装配
     @Bean
     public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
         SpringTemplateEngine templateEngine = new SpringTemplateEngine();
         templateEngine.setTemplateResolver(templateResolver);
         return templateEngine; //返回的是一个对象
     }
     //生成视图解析器并未解析器注入模板引擎
     @Bean
     public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
         ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
         viewResolver.setCharacterEncoding("UTF-8");
         viewResolver.setTemplateEngine(templateEngine);
         return viewResolver;
     }
 }
```

@Bean：如果给某一个方法添加此注解,那么此方法返回值就是一个bean对象，可以作为ioc容器中的一个bean。

## **13.4、测试功能**

```java
 @RequestMapping("/")
 public String index(){
   return "index";
 }
```

# **14、SpringMVC执行流程**

## **14.1、SpringMVC常用组件**

- DispatcherServlet：

  前端控制器

  ，不需要工程师开发，由框架提供，流程控制中心。

    - 作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求，下面几个组件都是由DispatcherServlet组件进行调用。

- HandlerMapping：

  处理器映射器

  ，不需要工程师开发，由框架提供,将浏览器的请求和控制器的方法进行映射。

    - 作用：根据请求的url、method等信息查找Handler（handle就是我们说的controller），即控制器方法。

- Handler：

  处理器

  ，需要工程师开发，即controller。

    - 作用：在DispatcherServlet的控制下Handler对具体的用户请求进行处理

- HandlerAdapter：

  处理器适配器

  ，不需要工程师开发，由框架提供，就是来调用具体的控制器方法的。HandlerMapping负责找控制器方法。

    - 作用：通过HandlerAdapter对处理器（控制器方法）进行执行

- ViewResolver：

  视图解析器

  ，不需要工程师开发，由框架提供，负责视图的解析，得到相对应的视图然后返回，controller最终返回的值就是视图名称。

    - 作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

- View：

  视图，由视图解析器解析出来的视图名称。

    - 作用：将模型数据通过页面展示给用户2，视图就是页面。

## **14.2、DispatcherServlet初始化过程**

DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet生命周期来进行调度。

### **①初始化WebApplicationContext**

所在类：org.springframework.web.servlet.FrameworkServlet

```java
 protected WebApplicationContext initWebApplicationContext() {
     WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext());
     WebApplicationContext wac = null;
     if (this.webApplicationContext != null) {
         // A context instance was injected at construction time -> use it
         wac = this.webApplicationContext;
         if (wac instanceof ConfigurableWebApplicationContext) {
             ConfigurableWebApplicationContext cwac =(ConfigurableWebApplicationContext) wac;
             if (!cwac.isActive()) {
                 // The context has not yet been refreshed -> provide services such as
                     // setting the parent context, setting the application context id, etc
                     if (cwac.getParent() == null) {
                         // The context instance was injected without an explicit parent -> set
                             // the root application context (if any; may be null) as the parent
                             cwac.setParent(rootContext);
                     }
                 configureAndRefreshWebApplicationContext(cwac);
             }
         }
     }
     if (wac == null) {
         // No context instance was injected at construction time -> see if one
         // has been registered in the servlet context. If one exists, it is assumed
             // that the parent context (if any) has already been set and that the
             // user has performed any initialization such as setting the context id
             wac = findWebApplicationContext();
     }
     if (wac == null) {
         // No context instance is defined for this servlet -> create a local one
         // 创建WebApplicationContext
         wac = createWebApplicationContext(rootContext);
     }
     if (!this.refreshEventReceived) {
         // Either the context is not a ConfigurableApplicationContext with refresh
             // support or the context injected at construction time had already been
             // refreshed -> trigger initial onRefresh manually here.
             synchronized (this.onRefreshMonitor) {
             // 刷新WebApplicationContext
             onRefresh(wac);
         }
     }
     if (this.publishContext) {
         // Publish the context as a servlet context attribute.
         // 将IOC容器在应用域共享
         String attrName = getServletContextAttributeName();
         getServletContext().setAttribute(attrName, wac);
     }
     return wac;
 }
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c469241-4723-4287-b26d-96b1e707906f/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fd8754e2-7779-407c-a19e-936f319792a4/Untitled.png)

### **②创建WebApplicationContext**

所在类：org.springframework.web.servlet.FrameworkServlet

```java
 protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
     Class<?> contextClass = getContextClass();
     if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass))
     {
         throw new ApplicationContextException("Fatal initialization error in servlet with name '" +getServletName() +
                                               "': custom WebApplicationContext class [" + contextClass.getName() +
                                               "] is not of type ConfigurableWebApplicationContext");
     }
     // 通过反射创建 IOC 容器对象
     ConfigurableWebApplicationContext wac = (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
     wac.setEnvironment(getEnvironment());
     // 设置父容器
     wac.setParent(parent);
     String configLocation = getContextConfigLocation();
     if (configLocation != null) {
         wac.setConfigLocation(configLocation);
     }
     configureAndRefreshWebApplicationContext(wac);
     return wac;
 }
```

### **③DispatcherServlet初始化策略**

FrameworkServlet创建WebApplicationContext后，刷新容器，调用onRefresh(wac)，此方法在DispatcherServlet中进行了重写，调用了initStrategies(context)方法，初始化策略，即初始化DispatcherServlet的各个组件所在类：org.springframework.web.servlet.DispatcherServlet

```java
 protected void initStrategies(ApplicationContext context) {
     initMultipartResolver(context);
     initLocaleResolver(context);
     initThemeResolver(context);
     initHandlerMappings(context);//初始化控制器
     initHandlerAdapters(context); // 调用处理器方法
     initHandlerExceptionResolvers(context);//处理异常
     initRequestToViewNameTranslator(context);
     initViewResolvers(context); //初始化视图解析器
     initFlashMapManager(context);
 }
```

## **14.3、DispatcherServlet调用组件处理请求**

### **①processRequest()**

FrameworkServlet重写HttpServlet中的service()和doXxx()，这些方法中调用了

processRequest(request, response)所在类：org.springframework.web.servlet.FrameworkServlet

```java
 protected final void processRequest(HttpServletRequest request,HttpServletResponse response)throws ServletException, IOException
 {
     long startTime = System.currentTimeMillis();
     Throwable failureCause = null;
     LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
     LocaleContext localeContext = buildLocaleContext(request);
     RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
     ServletRequestAttributes requestAttributes = buildRequestAttributes(request,response, previousAttributes);
     WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
     asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(),new RequestBindingInterceptor());
     initContextHolders(request, localeContext, requestAttributes);
     try {
         // 执行服务，doService()是一个抽象方法，在DispatcherServlet中进行了重写
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
```

### **②doService()**

所在类：org.springframework.web.servlet.DispatcherServlet

```java
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
                 attributesSnapshot.put(attrName,request.getAttribute(attrName));
             }
         }
     }
     // Make framework objects available to handlers and view objects.
     request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE,getWebApplicationContext());
     request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
     request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
     request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
     if (this.flashMapManager != null) {
         FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request,response);
         if (inputFlashMap != null) {
             request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE,Collections.unmodifiableMap(inputFlashMap));
         }
         request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
         request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
     }
     RequestPath requestPath = null;
     if (this.parseRequestPath && !ServletRequestPathUtils.hasParsedRequestPath(request)) {
         requestPath = ServletRequestPathUtils.parseAndCache(request);
     }
     try {
         // 处理请求和响应
         doDispatch(request, response);
     }
     finally {
         if
             (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
             // Restore the original attribute snapshot, in case of an include.
             if (attributesSnapshot != null) {
                 restoreAttributesAfterInclude(request, attributesSnapshot);
             }
         }
         if (requestPath != null) {
             ServletRequestPathUtils.clearParsedRequestPath(request);
         }
     }
 }
```

### **③doDispatch()**

所在类：org.springframework.web.servlet.DispatcherServlet

```java
 protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
     HttpServletRequest processedRequest = request;
     HandlerExecutionChain mappedHandler = null;
     boolean multipartRequestParsed = false;
     WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
     try {
         ModelAndView mv = null;
         Exception dispatchException = null;
         try {
             processedRequest = checkMultipart(request);
             multipartRequestParsed = (processedRequest != request);
             // Determine handler for the current request.
             /*
                 mappedHandler：调用链
                 包含handler、interceptorList、interceptorIndex
                 handler：浏览器发送的请求所匹配的控制器方法
                 interceptorList：处理控制器方法的所有拦截器集合
                 interceptorIndex：拦截器索引，控制拦截器afterCompletion()的执行
              */
             mappedHandler = getHandler(processedRequest);
             if (mappedHandler == null) {
                 noHandlerFound(processedRequest, response);
                 return;
             }
             // Determine handler adapter for the current request.
             // 通过控制器方法创建相应的处理器适配器，调用所对应的控制器方法
             HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
             // Process last-modified header, if supported by the handler.
             String method = request.getMethod();
             boolean isGet = "GET".equals(method);
             if (isGet || "HEAD".equals(method)) {
                 long lastModified = ha.getLastModified(request,mappedHandler.getHandler());
                 if (new ServletWebRequest(request,response).checkNotModified(lastModified) && isGet) {
                     return;
                 }
             }
             // 调用拦截器的preHandle()
             if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                 return;
             }
             // Actually invoke the handler.
             // 由处理器适配器调用具体的控制器方法，最终获得ModelAndView对象
             mv = ha.handle(processedRequest, response,mappedHandler.getHandler());
             if (asyncManager.isConcurrentHandlingStarted()) {
                 return;
             }
             applyDefaultViewName(processedRequest, mv);
             // 调用拦截器的postHandle()
             mappedHandler.applyPostHandle(processedRequest, response, mv);
         }
         catch (Exception ex) {
             dispatchException = ex;
         }
         catch (Throwable err) {
             // As of 4.3, we're processing Errors thrown from handler methods as well,
             // making them available for @ExceptionHandler methods and otherscenarios.
             dispatchException = new NestedServletException("Handler dispatchfailed", err);
          }
          // 后续处理：处理模型数据和渲染视图
          processDispatchResult(processedRequest, response, mappedHandler, mv,dispatchException);
     }
     catch (Exception ex) {
         triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
     }
     catch (Throwable err) {
         triggerAfterCompletion(processedRequest, response, mappedHandler,new NestedServletException("Handler processingfailed",
                                                                                                     err));
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
```

### **④processDispatchResult()**

```java
 private void processDispatchResult(HttpServletRequest request,HttpServletResponse response,@Nullable HandlerExecutionChain
                                    mappedHandler, @Nullable ModelAndView mv,@Nullable Exception exception) throws Exception {
     boolean errorView = false;
     if (exception != null) {
         if (exception instanceof ModelAndViewDefiningException) {
             logger.debug("ModelAndViewDefiningException encountered",exception);
             mv = ((ModelAndViewDefiningException) exception).getModelAndView();
         }
         else {
             Object handler = (mappedHandler != null ? mappedHandler.getHandler(): null);
             mv = processHandlerException(request, response, handler, exception);
             errorView = (mv != null);
         }
     }
     // Did the handler return a view to render?
     if (mv != null && !mv.wasCleared()) {
         // 处理模型数据和渲染视图
         render(mv, request, response);
         if (errorView) {
             WebUtils.clearErrorRequestAttributes(request);
         }
     }
     else {
         if (logger.isTraceEnabled()) {
             logger.trace("No view rendering, null ModelAndView returned.");
         }
     }
     if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
         // Concurrent handling started during a forward
         return;
     }
     if (mappedHandler != null) {
         // Exception (if any) is already handled..
         // 调用拦截器的afterCompletion()
         mappedHandler.triggerAfterCompletion(request, response, null);
     }
 }
```

## **14.4、SpringMVC的执行流程**

1. 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获。
2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI），判断请求URI对应的映射：

a) 不存在

i. 再判断是否配置了mvc:default-servlet-handler

ii. 如果没配置，则控制台报映射查找不到，客户端展示404错误

iii. 如果有配置，则访问目标资源（一般为静态资源，如：JS,CSS,HTML），找不到客户端也会展示404错误

b) 存在则执行下面的流程

1. 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain执行链对象的形式返回。
2. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。
3. 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(…)方法【正向】
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。

在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

a) HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息

b) 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等

c) 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等

d) 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult Error中

1. Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。
2. 此时将开始执行拦截器的postHandle(...)方法【逆向】。
3. 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行

HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。

1. 渲染视图完毕执行拦截器的afterCompletion(…)方法【逆向】。
2. 将渲染结果返回给客户端。

# **四、SSM整合**

4.1、ContextLoaderListener

Spring提供了监听器ContextLoaderListener，实现ServletContextListener接口，可监听

ServletContext的状态，在web服务器的启动，读取Spring的配置文件，创建Spring的IOC容器。web

应用中必须在web.xml中配置

```
 <listener>
     <!--
         配置Spring的监听器，在服务器启动时加载Spring的配置文件
         Spring配置文件默认位置和名称：/WEB-INF/applicationContext.xml
         可通过上下文参数自定义Spring配置文件的位置和名称
     -->
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
 <!--自定义Spring配置文件的位置和名称-->
 <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:spring.xml</param-value>
 </context-param>
```

## **4.2、准备工作**

### **①创建Maven Module**

### **②导入依赖**

```
 <packaging>war</packaging>
 <properties>
     <spring.version>5.3.1</spring.version>
 </properties>
 <dependencies>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-context</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-beans</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <!--springmvc-->
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-web</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-jdbc</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-aspects</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-test</artifactId>
         <version>${spring.version}</version>
     </dependency>
     <!-- Mybatis核心 -->
     <dependency>
         <groupId>org.mybatis</groupId>
         <artifactId>mybatis</artifactId>
         <version>3.5.7</version>
     </dependency>
     <!--mybatis和spring的整合包-->
     <dependency>
         <groupId>org.mybatis</groupId>
         <artifactId>mybatis-spring</artifactId>
         <version>2.0.6</version>
     </dependency>
     <!-- 连接池 -->
     <dependency>
         <groupId>com.alibaba</groupId>
         <artifactId>druid</artifactId>
         <version>1.0.9</version>
     </dependency>
     <!-- junit测试 -->
     <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>4.12</version>
         <scope>test</scope>
     </dependency>
     <!-- MySQL驱动 -->
     <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>8.0.16</version>
     </dependency>
     <!-- log4j日志 -->
     <dependency>
         <groupId>log4j</groupId>
         <artifactId>log4j</artifactId>
         <version>1.2.17</version>
     </dependency>
     <!-- <https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper> -->
     <dependency>
     <groupId>com.github.pagehelper</groupId>
     <artifactId>pagehelper</artifactId>
     <version>5.2.0</version>
     </dependency>
     <!-- 日志 -->
     <dependency>
         <groupId>ch.qos.logback</groupId>
         <artifactId>logback-classic</artifactId>
         <version>1.2.3</version>
     </dependency>
     <!-- ServletAPI -->
     <dependency>
         <groupId>javax.servlet</groupId>
         <artifactId>javax.servlet-api</artifactId>
         <version>3.1.0</version>
         <scope>provided</scope>
     </dependency>
     <dependency>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-databind</artifactId>
         <version>2.12.1</version>
     </dependency>
     <dependency>
         <groupId>commons-fileupload</groupId>
         <artifactId>commons-fileupload</artifactId>
         <version>1.3.1</version>
     </dependency>
     <!-- Spring5和Thymeleaf整合包 -->
     <dependency>
         <groupId>org.thymeleaf</groupId>
         <artifactId>thymeleaf-spring5</artifactId>
         <version>3.0.12.RELEASE</version>
     </dependency>
 </dependencies>
```

### **③创建表**

```
 CREATE TABLE `t_emp` (
     `emp_id` int(11) NOT NULL AUTO_INCREMENT,
     `emp_name` varchar(20) DEFAULT NULL,
     `age` int(11) DEFAULT NULL,
     `sex` char(1) DEFAULT NULL,
     `email` varchar(50) DEFAULT NULL,
     PRIMARY KEY (`emp_id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

## **4.3、配置web.xml**

```
 <!-- 配置Spring的编码过滤器 -->
 <filter>
     <filter-name>CharacterEncodingFilter</filter-name>
     <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
     <init-param>
         <param-name>encoding</param-name>
         <param-value>UTF-8</param-value>
     </init-param>
     <init-param>
         <param-name>forceEncoding</param-name>
         <param-value>true</param-value>
     </init-param>
 </filter>
 <filter-mapping>
     <filter-name>CharacterEncodingFilter</filter-name>
     <url-pattern>/*</url-pattern>
 </filter-mapping>
 <!-- 配置处理请求方式PUT和DELETE的过滤器 -->
 <filter>
     <filter-name>HiddenHttpMethodFilter</filter-name>
     <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter•class>
 </filter>
 <filter-mapping>
     <filter-name>HiddenHttpMethodFilter</filter-name>
     <url-pattern>/*</url-pattern>
 </filter-mapping>
 <!-- 配置SpringMVC的前端控制器 -->
 <servlet>
     <servlet-name>DispatcherServlet</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet•class>
     <!-- 设置SpringMVC的配置文件的位置和名称 -->
     <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:SpringMVC.xml</param-value>
     </init-param>
     <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
     <servlet-name>DispatcherServlet</servlet-name>
     <url-pattern>/</url-pattern>
 </servlet-mapping>
 <!-- 设置Spring的配置文件的位置和名称 -->
 <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:Spring.xml</param-value>
 </context-param>
 <!-- 配置Spring的监听器 -->
 <listener>
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
```

## **4.4、创建SpringMVC的配置文件并配置**

```
 <!--扫描组件-->
 <context:component-scan base-package="com.atguigu.ssm.controller">
 </context:component-scan>
 <!--配置视图解析器-->
 <bean id="viewResolver"
       class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
     <property name="order" value="1"/>
     <property name="characterEncoding" value="UTF-8"/>
     <property name="templateEngine">
         <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
             <property name="templateResolver">
                 <bean
                       class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                     <!-- 视图前缀 -->
                     <property name="prefix" value="/WEB-INF/templates/"/>
                     <!-- 视图后缀 -->
                     <property name="suffix" value=".html"/>
                     <property name="templateMode" value="HTML5"/>
                     <property name="characterEncoding" value="UTF-8" />
                 </bean>
             </property>
         </bean>
     </property>
 </bean>
 <!-- 配置访问首页的视图控制 -->
 <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
 <!-- 配置默认的servlet处理静态资源 -->
 <mvc:default-servlet-handler />
 <!-- 开启MVC的注解驱动 -->
 <mvc:annotation-driven />
```

## **4.5、搭建MyBatis环境**

### **①创建属性文件jdbc.properties**

```
 jdbc.user=root
 jdbc.password=atguigu
 jdbc.url=jdbc:mysql://localhost:3306/ssm?serverTimezone=UTC
 jdbc.driver=com.mysql.cj.jdbc.Driver
```

### **②创建MyBatis的核心配置文件mybatis-config.xml**

```
 <?xml version="1.0" encoding="UTF-8" ?>
 <!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "<http://mybatis.org/dtd/mybatis-3-config.dtd>">
 <configuration>
     <settings>
         <!--将下划线映射为驼峰-->
         <setting name="mapUnderscoreToCamelCase" value="true"/>
     </settings>
     <plugins>
         <!--配置分页插件-->
         <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
     </plugins>
 </configuration>
```

### **③创建Mapper接口和映射文件**

```
 public interface EmployeeMapper {
 List<Employee> getEmployeeList();
 }
 <?xml version="1.0" encoding="UTF-8" ?>
 <!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "<http://mybatis.org/dtd/mybatis-3-mapper.dtd>">
 <mapper namespace="com.atguigu.ssm.mapper.EmployeeMapper">
     <select id="getEmployeeList" resultType="Employee">
         select * from t_emp
     </select>
 </mapper>
```

### **④创建日志文件log4j.xml**

```
 <?xml version="1.0" encoding="UTF-8" ?>
 <!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
 <log4j:configuration xmlns:log4j="<http://jakarta.apache.org/log4j/>">
     <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
         <param name="Encoding" value="UTF-8" />
         <layout class="org.apache.log4j.PatternLayout">
             <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m (%F:%L) \\n" />
         </layout>
     </appender>
     <logger name="java.sql">
         <level value="debug" />
     </logger>
     <logger name="org.apache.ibatis">
         <level value="info" />
     </logger>
     <root>
         <level value="debug" />
         <appender-ref ref="STDOUT" />
     </root>
 </log4j:configuration>
```

## **4.6、创建Spring的配置文件并配置**

```
 <?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="<http://www.springframework.org/schema/beans>"
        xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xmlns:context="<http://www.springframework.org/schema/context>"
        xsi:schemaLocation="<http://www.springframework.org/schema/beans>
                            <http://www.springframework.org/schema/beans/spring-beans.xsd>
                            <http://www.springframework.org/schema/context>
                            <https://www.springframework.org/schema/context/spring-context.xsd>">
     <!--扫描组件-->
     <context:component-scan base-package="com.atguigu.ssm">
         <context:exclude-filter type="annotation"expression="org.springframework.stereotype.Controller"/>
     </context:component-scan>
     <!-- 引入jdbc.properties -->
     <context:property-placeholder location="classpath:jdbc.properties">
     </context:property-placeholder>
     <!-- 配置Druid数据源 -->
     <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
         <property name="driverClassName" value="${jdbc.driver}"></property>
         <property name="url" value="${jdbc.url}"></property>
         <property name="username" value="${jdbc.username}"></property>
         <property name="password" value="${jdbc.password}"></property>
     </bean>
     <!-- 配置用于创建SqlSessionFactory的工厂bean -->
     <bean class="org.mybatis.spring.SqlSessionFactoryBean">
         <!-- 设置MyBatis配置文件的路径（可以不设置） -->
         <property name="configLocation" value="classpath:mybatis-config.xml">
         </property>
         <!-- 设置数据源 -->
         <property name="dataSource" ref="dataSource"></property>
         <!-- 设置类型别名所对应的包 -->
         <property name="typeAliasesPackage" value="com.atguigu.ssm.pojo">
         </property>
         <!--
             设置映射文件的路径
             若映射文件所在路径和mapper接口所在路径一致，则不需要设置
         -->
         <!--
             <property name="mapperLocations" value="classpath:mapper/*.xml">
             </property>
         -->
     </bean>
     <!--
         配置mapper接口的扫描配置
         由mybatis-spring提供，可以将指定包下所有的mapper接口创建动态代理
         并将这些动态代理作为IOC容器的bean管理
     -->
     <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
         <property name="basePackage" value="com.atguigu.ssm.mapper"></property>
     </bean>
 </beans>
```

## **4.7、测试功能**

### **①创建组件**

实体类Employee

```
 public class Employee {
     private Integer empId;
     private String empName;
     private Integer age;
     private String sex;
     private String email;
     public Employee() {
     }
     public Employee(Integer empId, String empName, Integer age, String sex,
                     String email) {
         this.empId = empId;
         this.empName = empName;
         this.age = age;
         this.sex = sex;
         this.email = email;
     }
     public Integer getEmpId() {
         return empId;
     }
     public void setEmpId(Integer empId) {
         this.empId = empId;
     }
     public String getEmpName() {
         return empName;
     }
     public void setEmpName(String empName) {
         this.empName = empName;
     }
     public Integer getAge() {
         return age;
     }
     public void setAge(Integer age) {
         this.age = age;
     }
     public String getSex() {
         return sex;
     }
     public void setSex(String sex) {
         this.sex = sex;
     }
     public String getEmail() {
         return email;
     }
     public void setEmail(String email) {
         this.email = email;
     }
 }
```

创建控制层组件EmployeeController

```
 @Controller
 public class EmployeeController {
     @Autowired
     private EmployeeService employeeService;
     @RequestMapping(value = "/employee/page/{pageNum}", method = RequestMethod.GET)
     public String getEmployeeList(Model model, @PathVariable("pageNum") Integer pageNum){
         PageInfo<Employee> page = employeeService.getEmployeeList(pageNum);
         model.addAttribute("page", page);
         return "employee_list";
     }
 }
```

创建接口EmployeeService

```
 public interface EmployeeService {
   PageInfo<Employee> getEmployeeList(Integer pageNum);
 }
```

创建实现类EmployeeServiceImpl

```
 @Service
 public class EmployeeServiceImpl implements EmployeeService {
     @Autowired
     private EmployeeMapper employeeMapper;
     @Override
     public PageInfo<Employee> getEmployeeList(Integer pageNum) {
         PageHelper.startPage(pageNum, 4);
         List<Employee> list = employeeMapper.getEmployeeList();
         PageInfo<Employee> page = new PageInfo<>(list, 5);
         return page;
     }
 }
```

### **②创建页面**

```
 <!DOCTYPE html>
 <html lang="en" xmlns:th="<http://www.thymeleaf.org>">
     <head>
         <meta charset="UTF-8">
         <title>Employee Info</title>
         <link rel="stylesheet" th:href="@{/static/css/index_work.css}">
     </head>
     <body>
         <table>
             <tr>
                 <th colspan="6">Employee Info</th>
             </tr>
             <tr>
                 <th>emp_id</th>
                 <th>emp_name</th>
                 <th>age</th>
                 <th>sex</th>
                 <th>email</th>
                 <th>options</th>
             </tr>
             <tr th:each="employee : ${page.list}">
                 <td th:text="${employee.empId}"></td>
                 <td th:text="${employee.empName}"></td>
                 <td th:text="${employee.age}"></td>
                 <td th:text="${employee.sex}"></td>
                 <td th:text="${employee.email}"></td>
                 <td>
                     <a href="">delete</a>
                     <a href="">update</a>
                 </td>
             </tr>
             <tr>
                 <td colspan="6">
                     <span th:if="${page.hasPreviousPage}">
                         <a th:href="@{/employee/page/1}">首页</a>
                         <a th:href="@{'/employee/page/'+${page.prePage}}">上一页</a>
                     </span>
                     <span th:each="num : ${page.navigatepageNums}">
                         <a th:if="${page.pageNum==num}"
                            th:href="@{'/employee/page/'+${num}}" th:text="'['+${num}+']'" style="color:red;"></a>
                         <a th:if="${page.pageNum!=num}"
                            th:href="@{'/employee/page/'+${num}}" th:text="${num} "></a>
                     </span>
                     <span th:if="${page.hasNextPage}">
                         <a th:href="@{'/employee/page/'+${page.nextPage}}">下一页</a>
                         <a th:href="@{'/employee/page/'+${page.pages}}">末页</a>
                     </span>
                 </td>
             </tr>
         </table>
     </body>
 </html>
```

### **③访问测试分页功能**

localhost:8080/employee/page/1