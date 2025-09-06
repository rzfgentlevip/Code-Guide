---
title: SpringMVC（基础）
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
permalink: /learning-notes/k16fmb7u/
---

# **1、SpringMVC（基础）**

### **1.1、什么是MVC**

MVC是一种软件架构的思想，将软件按照**模型、视图、控制器**来划分:

M：Model，模型层，指工程中的JavaBean，作用是处理数据

- JavaBean分为两类：
    - 一类称为实体类Bean：专门存储业务数据的，如 Student、User 等
    - 一类称为业务处理 Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。

V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据。

C：Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器。

> MVC的工作流程： 用户通过视图层(View)发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器。

## **1.2、什么是SpringMVC**

SpringMVC是Spring的一个后续产品，是Spring的一个子项目。

SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案。在表述层框架历经 Strust、WebWork、Strust2 等诸多产品的历代更迭之后，目前业界普遍选择了 SpringMVC 作为 Java EE 项目述层开发的**首选方案**。

> 注：三层架构分为**表述层（或表示层）、业务逻辑层、数据访问层**，表述层表示前台页面和后台,springMvc封装的就是servlet。

## **1.3、SpringMVC的特点**

- **Spring** **家族原生产品**，与 IOC 容器等基础设施无缝对接
- **基于原生的Servlet**，通过了功能强大的**前端控制器DispatcherServlet**，对请求和响应进行统一处理
- 表述层各细分领域需要解决的问题**全方位覆盖**，提供**全面解决方案**
- **代码清新简洁**，大幅度提升开发效率
- 内部组件化程度高，可插拔式组件**即插即用**，想要什么功能配置相应组件即可
- **性能卓著**，尤其适合现代大型、超大型互联网项目要求

# **2、入门案例**

## **2.1、开发环境**

- IDE：idea 2019.2
- 构建工具：maven3.5.4
- 服务器：tomcat8.5
- Spring版本：5.3.1

## **2.2、创建maven工程**

web程序开发步骤：

1. 引入依赖
2. 配置web.xml配置文件。
    1. 默认配置方式
    2. 扩展配置方式。
3. 创建控制器。
4. 创建springmvc配置文件

### **①添加web模块**

### **②打包方式：war**

<aside> 💡 1、war是一个web模块，其中需要包括WEB-INF，是可以直接运行的WEB模块；jar一般只是包括一些class文件，在声明了Main_class之后是可以用java命令运行的。 2、war包是做好一个web应用后，通常是网站，打成包部署到容器中；jar包通常是开发时要引用通用类，打成包便于存放管理。 3、war是Sun提出的一种Web应用程序格式，也是许多文件的一个压缩包。这个包中的文件按一定目录结构来组织；classes目录下则包含编译好的Servlet类和Jsp或Servlet所依赖的其它类（如JavaBean）可以打包成jar放到WEB-INF下的lib目录下。

4、JAR文件格式以流行的ZIP文件格式为基础。与ZIP文件不同的是，JAR 文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用。

</aside>

### **③引入依赖**

```xml
 <dependencies>
     <!-- SpringMVC -->
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>5.3.1</version>
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
     <!-- Spring5和Thymeleaf整合包 -->
     <dependency>
         <groupId>org.thymeleaf</groupId>
         <artifactId>thymeleaf-spring5</artifactId>
         <version>3.0.12.RELEASE</version>
     </dependency>
 </dependencies>
```

注：由于 Maven 的传递性，我们不必将所有需要的包全部配置依赖，而是配置最顶端的依赖，其他靠传递性导入。

## **2.3、配置web.xml**

`web.xml`用来注册servlet，filter过滤器和监听器,注册SpringMVC的前端控制器DispatcherServlet。

### **①默认配置方式**

此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为`<servlet-name>-servlet.xml`，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为springMVC-servlet.xml

`servlet.xml`或者`web.xml`

```text
 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理
 <servlet>
     <servlet-name>springMVC</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet•class>
 </servlet>
 <servlet-mapping>
     <servlet-name>springMVC</servlet-name>
         设置springMVC的核心控制器所能处理的请求的请求路径 /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
         但是/不能匹配.jsp请求路径的请求
					/*：匹配是所有请求
     <url-pattern>/</url-pattern>
 </servlet-mapping>
```

servlet-mapping要和servlet标签名字一样，他们两个共同来注册一个servlet。

utrl-pattern：用来匹配浏览器发送过来的请求。

### **②扩展配置方式**

可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间，这样就可以将springmvc的配置文件放在resources文件下。

```xml
 <!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
 <servlet>
     <servlet-name>springMVC</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet•class>
     <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
     <init-param>
         <!-- contextConfigLocation为固定值 -->
         <param-name>contextConfigLocation</param-name>
         <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
         <param-value>classpath:springMVC.xml</param-value>
												classpath：对应的是resources目录
     </init-param>
     <!--
         作为框架的核心组件，在启动过程中有大量的初始化操作要做
         而这些操作放在第一次请求时才执行会严重影响访问速度
         因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
   -->
     <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
     <servlet-name>springMVC</servlet-name>
     <!--
         设置springMVC的核心控制器所能处理的请求的请求路径
         /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
         但是/不能匹配.jsp请求路径的请求
     -->
     <url-pattern>/</url-pattern>
 </servlet-mapping>
```

注:
- `<url-pattern>`标签中使用/和/*的区别：
- /所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求，因此就可以避免在访问jsp页面时，该请求被DispatcherServlet处理，从而找不到相应的页面
- /*则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用/*的写法


浏览器发送的请求要交给前端控制器处理，前端控制器是一个servlet，我们想通过servlet处理请求，就必须在web.xml文件中进行注册，在注册的时候通过init-param标注了配置文件的位置和名称。

## **2.4、创建请求控制器**

由于前端控制器对浏览器发送的请求进行了统一的处理，但是具体的请求有不同的处理过程，因此需要**创建处理具体请求的类**，即**请求控制器**,请求控制器中每一个处理请求的方法成为控制器方法。

因为SpringMVC的控制器由一个POJO（普通的Java类）担任，因此需要通过@Controller注解将其标识为一个控制层组件，交给Spring的IoC容器管理，此时SpringMVC才能够识别控制器的存在

```java
 @Controller
 public class HelloController {
 }
```

## **2.5、创建SpringMVC的配置文件**

视图解析器负责页面的跳转

```xml
 <!-- 自动扫描包 -->
 <context:component-scan base-package="com.atguigu.mvc.controller"/>
 <!-- 配置Thymeleaf视图解析器 -->
 <bean id="viewResolver"class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
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
 <!--
     处理静态资源，例如html、js、css、jpg
     若只设置该标签，则只能访问静态资源，其他请求则无法访问
     此时必须设置<mvc:annotation-driven/>解决问题
 -->
 <mvc:default-servlet-handler/>
 <!-- 开启mvc注解驱动 -->
 <mvc:annotation-driven>
     <mvc:message-converters>
         <!-- 处理响应中文内容乱码 -->
         <bean
               class="org.springframework.http.converter.StringHttpMessageConverter">
             <property name="defaultCharset" value="UTF-8" />
             <property name="supportedMediaTypes">
                 <list>
                     <value>text/html</value>
                     <value>application/json</value>
                 </list>
             </property>
         </bean>
     </mvc:message-converters>
 </mvc:annotation-driven>
```

## **2.6、测试HelloWorld**

### **①实现对首页的访问**

在请求控制器中创建处理请求的方法

```java
 // @RequestMapping注解：处理请求和控制器方法之间的映射关系
 // @RequestMapping注解的value属性可以通过请求地址匹配请求，/表示的当前工程的上下文路径
 // localhost:8080/springMVC/
 @RequestMapping("/")
 public String index() {
     //设置视图名称
     return "index";
 }
```

### **②通过超链接跳转到指定页面**

在主页index.html中设置超链接

```html
 <!DOCTYPE html>
 <html lang="en" xmlns:th="<http://www.thymeleaf.org>">
     <head>
         <meta charset="UTF-8">
         <title>首页</title>
     </head>
     <body>
         <h1>首页</h1>
<!--标签上添加th:，表示让thymeleaf帮助我们解析，检测到我们使用的是绝对路径的时候，就会自动帮助我们添加上下文路径-->
         <a th:href="@{/hello}">HelloWorld</a><br/>
     </body>
 </html>
```

在请求控制器中创建处理请求的方法

```java
 @RequestMapping("/hello")
 public String HelloWorld() {
	//返回的是html页面名字
   return "target";
 }
```

## **2.7、总结**

浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器，将请求地址和控制器中@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的控制器方法就是处理请求的方法。**处理请求的方法需要返回一个字符串类型的视图名称**，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应页面

# **3、@RequestMapping注解**

## **3.1、@RequestMapping注解的功能**

从注解名称上我们可以看到，@RequestMapping注解的作用就是将**请求和处理请求**的控制器方法关联起来，建立映射关系。

SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求。

## **3.2、@RequestMapping注解的位置**

@RequestMapping标识一个类：设置映射请求的请求路径的**初始信息**

@RequestMapping标识一个方法：设置映射请求请求路径的**具体信息**

```java
 @Controller
 @RequestMapping("/test")
 public class RequestMappingController {
     //此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
     @RequestMapping("/testRequestMapping")
     public String testRequestMapping(){
         return "success";
     }
 }
// 映射的请求路径是：/test/testRequestMapping
```

## 3.3、@RequestMapping注解的value属性

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94b887a1-6d54-4500-95b7-d4aca07ea0f3/Untitled.png)

value()属性：

- @RequestMapping注解的value属性**通过请求的请求地址匹配请求映射**
- @RequestMapping注解的value属性是一个字符串类型的数组，**表示该请求映射能够匹配多个请求地址所对应的请求**

method()：通过请求方式匹配请求，get()，put()

parama():通过请求参数匹配请求

headers()：通过请求头信息匹配属性

@RequestMapping注解的value属性必须设置，至少通过请求地址匹配请求映射

```xml
 <a th:href="@{/testRequestMapping}">测试@RequestMapping的value属性--
 >/testRequestMapping</a><br>
 <a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
 @RequestMapping(
     value = {"/testRequestMapping", "/test"}  //能够处理多个请求，满足任何一个就可以处理
 )
// value()属性是通过请求地址进行匹配。
 public String testRequestMapping(){
     return "success"; //返回的是视图名称
 }
```

## **3.4、@RequestMapping注解的method属性**

@RequestMapping注解的method属性通过请求的请求方式（get或post）匹配请求映射。

@RequestMapping注解的method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多种请求方式的请求。

若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器报错 405：Request method 'POST' not supported

```xml
 <a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
 <form th:action="@{/test}" method="post">
   <input type="submit">
 </form>
 @RequestMapping(
     value = {"/testRequestMapping", "/test"},
     method = {RequestMethod.GET, RequestMethod.POST}
 )
 public String testRequestMapping(){
     return "success";
 }
```

> 注：
>
> 1、对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解
>
> - 处理get请求的映射-->@GetMapping
> - 处理post请求的映射-->@PostMapping
> - 处理put请求的映射-->@PutMapping
> - 处理delete请求的映射-->@DeleteMapping
>
> 2、常用的请求方式有get，post，put，delete
>
> 但是目前浏览器只支持get和post，若在form表单提交时，为method设置了其他请求方式的字符串（put或delete），则按照默认的请求方式get处理
>
> 若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter，在RESTful部分会讲到

## **3.5、@RequestMapping注解的params属性（了解）**

@RequestMapping注解的params属性**通过请求的请求参数匹配请求映射**

@RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系：

1. "param"：要求请求映射所匹配的请求必须携带param请求参数
2. "!param"：要求请求映射所匹配的请求必须不能携带param请求参数
3. "param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value
4. "param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value

```xml
 <a th:href="@{/test(username='admin',password=123456)">测试@RequestMapping的
 params属性-->/test</a><br>
 @RequestMapping(
     value = {"/testRequestMapping", "/test"}
     ,method = {RequestMethod.GET, RequestMethod.POST}
     ,params = {"username","password!=123456"}
 )
 public String testRequestMapping(){
     return "success";
 }
```

> 注：
>
> 若当前请求满足@RequestMapping注解的value和method属性，但是不满足params属性，此时页面回报错400：Parameter conditions "username, password!=123456" not met for actual request parameters: username={admin}, password={123456}

## **3.6、@RequestMapping注解的headers属性（了解）**

@RequestMapping注解的headers属性  通过请求的请求头信息匹配请求映射

@RequestMapping注解的headers属性是一个**字符串类型的数组**，可以通过四种表达式设置请求头信息和请求映射的匹配关系

1. "header"：要求请求映射所匹配的请求必须携带header请求头信息
2. "!header"：要求请求映射所匹配的请求必须不能携带header请求头信息
3. "header=value"：要求请求映射所匹配的请求必须携带header请求头信息header=value
4. "header!=value"：要求请求映射所匹配的请求必须携带header请求头信息且header!=value

**若当前请求满足@RequestMapping注解的value和method属性，但是不满足headers属性，此时页面显示404错误，即资源未找到**

## **3.7、SpringMVC支持ant风格的路径**

？：表示任意的单个字符

*：表示任意的0个或多个字符

**：表示任意层数的任意目录

注意：在使用***\*时，只能使用/\****/xxx的方式

## **3.8、SpringMVC支持路径中的占位符（重点）**

- 原始方式：/deleteUser?id=1，传输参数通过？占位符进行传输，不安全。
- rest方式：/user/delete/1，通过相同的访问地址，不同的访问方式区分，不需要在使用问号传输参数，只需要将参数写在路径中即可，中间使用斜线分隔。

SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据**通过路径的方式传输到服务器中**，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参

```xml
 <a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
 @RequestMapping("/testRest/{id}/{username}")-- 路径中的占位符 
 public String testRest(@PathVariable("id") String id, @PathVariable("username")
 String username){
     System.out.println("id:"+id+",username:"+username);
     return "success";
 }
 //最终输出的内容为-->id:1,username:admin
```

# **4、SpringMVC获取请求参数**

浏览器发送的请求都会首先被**前端控制器处理**，前端控制器通过匹配**requestmapping**找到方法，然后交给**DispatcherServlet**处理。最终会由DispatcherServlet间接调用控制器方法。

## **4.1、通过ServletAPI获取**

将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest（请求对象）类型的参数表示**封装了当前请求的请求报文的对象,使用此对象可以获取到参数**。

```java
 @RequestMapping("/testParam")
 public String testParam(HttpServletRequest request){
     String username = request.getParameter("username");
     String password = request.getParameter("password");
     System.out.println("username:"+username+",password:"+password);
     return "success";
 }
```

## **4.2、通过控制器方法的形参获取请求参数**

**在控制器方法的形参位置，设置和请求参数同名的形参**，当浏览器发送请求，匹配到请求映射时，DispatcherServlet中就会将请求参数赋值给相应的形参。控制器中的形参名字和请求中的参数名要求一致。

```java
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数--
>/testParam</a><br>
 @RequestMapping("/testParam")
 public String testParam(String username, String password){
     System.out.println("username:"+username+",password:"+password);
     return "success";
 }
```

> 注：
>
> - 若请求所传输的请求参数中有
    >
    >   多个同名的请求参数
    >
    >   ，此时可以在
    >
    >   控制器方法的形参中设置字符串数组或者字符串类型的形参
    >
    >   接收此请求参数
    >
    >   - 若使用字符串数组类型的形参，此参数的数组中包含了每一个数据
>   - 若使用字符串类型的形参，此参数的值为每个数据中间使用**逗号**拼接的结果

## **4.3、@RequestParam**

> 当请求参数中的参数名和控制器方法中的形参参数名字不一致的时候。 如果在形参方法中使用了这个注解，那么前端请求参数对于这个属性必须传输值，不传输会报错。

@RequestParam是将**请求参数和控制器方法的形参创建映射关系**

@RequestParam注解一共有三个属性：

- value：指定为形参赋值的请求参数的参数名，尽量和控制器的参数名字一致。
- required：设置是否必须传输此请求参数，默认值为true
    - 若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面报错400：Required String parameter 'xxx' is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null
- defaultValue：不管required属性值为true或false，当value所指定的请求参数没有传输或传输的值为""时，则使用默认值为形参赋值

## **4.4、@RequestHeader**

> 请求头中的信息也是键值对。想要在控制器中获取请求头信息，必须使用@RequestHeader注解。

@RequestHeader是将请求头信息和控制器方法的形参创建映射关系

@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

## **4.5、@CookieValue**

@CookieValue是将cookie数据和控制器方法的形参创建映射关系

@CookieValue注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

## **4.6、通过POJO获取请求参数**

可以在**控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值**

```xml
 <form th:action="@{/testpojo}" method="post">
     用户名：<input type="text" name="username"><br>
     密码：<input type="password" name="password"><br>
     性别：<input type="radio" name="sex" value="男">男<input type="radio"name="sex" value="女">女<br>
     年龄：<input type="text" name="age"><br>
     邮箱：<input type="text" name="email"><br>
     <input type="submit">
 </form>
 @RequestMapping("/testpojo")
 public String testPOJO(User user){
     System.out.println(user);
     return "success";
 }
 //最终结果-->User{id=null, username='张三', password='123', age=23, sex='男',
 email='123@qq.com'}
```

## **4.7、解决获取请求参数的乱码问题**

解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册

```xml
 <!--配置springMVC的编码过滤器-->
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
```

> 注：SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效

# **5、域对象共享数据**

处理请求的过程：

1. 设置编码，springmvc为我们提供了编码过滤器，我们只需要将编码过滤器注册在web.xml中即可。
2. 获取前端传输的请求参数。
3. 将请求参数作为条件去调用service处理业务逻辑，service再去调用dao访问数据库，然后在将最后的结果返回给service，service在将结果返回给控制层，如果有些数据是需要向页面中发送，那么就需要将这些数据在域对象中共享。

> request（一次请求）,sesson（一次会话，浏览器开启到浏览器关闭）,application（也叫做servletcontext,指整个应用的范围，也就是整个服务器开启到服务器关闭）三个共享数据域，还有一个pagecontext，指的是一个jsp页面的范围，现在不用jsp，所以先不考虑pagecontext域对象。

session域对象只和浏览器关闭是否有关系，application只和服务器关闭是否有关。

## **5.1、使用ServletAPI向request域对象共享数据**

```java
 @RequestMapping("/testServletAPI")
 public String testServletAPI(HttpServletRequest request){
     request.setAttribute("testScope", "hello,servletAPI");
     return "success";
 }

//    使用servletapi向request域对象共享数据
    @RequestMapping("testRequestByServletAPI")
    public String testRequestByServletAPI(HttpServletRequest request){

        request.setAttribute("设置共享数据","共享数据");//设置域对象的共享数据
        request.getParameter("设置共享数据");//获取域对象的共享数据
        request.removeAttribute("设置共享数据");//删除域对象中的共享数据

        return "success";
    } 
```

## **5.2、使用ModelAndView向request域对象共享数据**

两个功能：

1. 向request域对象中共享数据
2. 设置视图名称

使用ModelAndView 对象时，**必须将对象作为方法的返回值返回**。

```java
 @RequestMapping("/testModelAndView")
 public ModelAndView testModelAndView(){
     /**
      * ModelAndView有Model和View的功能
      * Model主要用于向请求域共享数据
      * View主要用于设置视图，实现页面跳转
      */
     ModelAndView mav = new ModelAndView();
     //向请求域共享数据
     mav.addObject("testScope", "hello,ModelAndView");
     //设置视图，实现页面跳转
     mav.setViewName("success");
//        将modelAndView返回给前端控制器，控制器才可以去解析
     return mav;
 }
```

## **5.3、使用Model向request域对象共享数据**

这里的Model指的是ModelAndView里面的model。

```java
@RequestMapping("/testModel")
public String testModel(Model model){ //创建形参，在形参的位置设置model
    model.addAttribute("testScope", "hello,Model");//设置共享与数据
    return "success";
}
```

## **5.4、使用map向request域对象共享数据**

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testScope", "hello,Map");
    return "success";//设置视图
}
```

## 5.5、使用ModelMap向request域对象共享数据

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testScope", "hello,ModelMap");
    return "success";
}
```

## **5.6、Model、ModelMap、Map的关系**

Model、ModelMap、Map类型的参数其实本质上都是 BindingAwareModelMap 类型的

```java
 public interface Model{}
 public class ModelMap extends LinkedHashMap<String, Object> {}
 public class ExtendedModelMap extends ModelMap implements Model {}
 public class BindingAwareModelMap extends ExtendedModelMap {}
```

上面通过形参向请求对象域中共享数据底层都使用了同一个对象：BindingAwareModelMap

## **5.7、向session域共享数据**

```java
 @RequestMapping("/testSession")
 public String testSession(HttpSession session){
     session.setAttribute("testSessionScope", "hello,session");
     return "success";
 }
```

## **5.8、向application域共享数据**

一个servletcontext就是一个应用，这个对象在整个工程启动时候创建。

```java
 @RequestMapping("/testApplication")
 public String testApplication(HttpSession session){
     ServletContext application = session.getServletContext();
     application.setAttribute("testApplicationScope", "hello,application");
     return "success";
 }
```

# **6、SpringMVC的视图**

SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户

SpringMVC视图的种类很多，默认有**转发视图和重定向视图**

当工程引入jstl的依赖，转发视图会自动转换为JstlView

若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView

## **6.1、ThymeleafView**

当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转

```java
 @RequestMapping("/testHello")
 public String testHello(){
   return "hello";
 }
```

如果视图名称没有任何前缀和后缀，那么就会被配置的thymeleaf视图解析器解析。

## **6.2、转发视图**

SpringMVC中默认的转发视图是InternalResourceView

SpringMVC中创建转发视图的情况：

当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转

例如"forward:/"，"forward:/employee"

```java
 @RequestMapping("/testForward")
 public String testForward(){
   return "forward:/testHello";
 }
```

转发既可以转发到一个页面，也可以转发到一个请求地址，但是前面要添加forward.

## **6.3、重定向视图**

在操作执行成功之后，一般都需要通过**重定向实现页面跳转**，一般不使用转发。转发由于浏览器只发送一起请求，因此浏览器地址栏不会改变，而重定向浏览器会发送两次请求，因此地址栏的地址也会变化。转发可以使用一个request域中的数据，但是重定向不可以，因为是两次请求，使用的不是一个request对象。

SpringMVC中默认的重定向视图是RedirectView

当控制器方法中所设置的视图名称以"redirect:"为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最终路径通过重定向的方式实现跳转

例如"redirect:/"，"redirect:/employee"

```java
 @RequestMapping("/testRedirect")
 public String testRedirect(){
   return "redirect:/testHello";
 }
```

> 注：
>
> 重定向视图在解析时，会先将redirect:前缀去掉，然后会判断剩余部分是否以/开头，若是则会自动拼接上下文路径

## **6.4、视图控制器view-controller**

当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，没有其他的请求处理过程，可以将处理器方法使用view controller标签进行表示

```xml
<!--
    path：设置处理的请求地址
    view-name：设置请求地址所对应的视图名称
-->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>
```

> 注：
>
> 当SpringMVC中设置任何一个view-controller时，其他控制器中的请求映射将全部失效，此时需要在SpringMVC的核心配置文件中设置开启mvc注解驱动的标签：
>
> <mvc:annotation-driven />

# **7、 RESTful**

## **7.1、RESTful简介**

REST：**Re**presentational **S**tate **T**ransfer，**表现层资源状态转移**。

表现层就是表示层，就是视图和控制层，前端的视图页面到后端的控制层。

web工程放到服务器上的过程叫做部署，当工程部署到tomcat上的时候，工程内所有的文件都是资源，比如类，html页面，css文件或者图片等。

资源的状态就是当前资源的表现形式，比如html页面，css是样式，js是一段脚本。

### **①资源**

资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个URI来标识。URI既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URI与其进行交互。

### **②资源的表述**

资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移（交

换）。资源的表述可以有多种格式，例如HTML/XML/JSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

### **③状态转移**

状态转移说的是：在客户端和服务器端之间转移（transfer）代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

## **7.2、RESTful的实现**

具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。

它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE用来删除资源。

REST 风格提倡 URL 地址使用**统一的风格设计**，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。

| **操作** | **传统方式**     | **REST风格**            |
| -------- | ---------------- | ----------------------- |
| 查询操作 | getUserById?id=1 | user/1-->get请求方式    |
| 保存操作 | saveUser         | user-->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1-->delete请求方式 |
| 更新操作 | updateUser       | user-->put请求方式      |

## **7.3、HiddenHttpMethodFilter**

由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将** **POST** **请求转换为** **DELETE** **或** **PUT** **请求。**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

a>当前请求的请求方式必须为post

b>当前请求必须传输请求参数_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数

*method的值，因此请求参数*method的值才是最终的请求方式

在web.xml中注册**HiddenHttpMethodFilter**

```xml
 <filter>
     <filter-name>HiddenHttpMet hodFilter</filter-name>
     <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter•class>
 </filter>
 <filter-mapping>
     <filter-name>HiddenHttpMethodFilter</filter-name>
     <url-pattern>/*</url-pattern>
 </filter-mapping>
```

> 注：
>
> 目前为止，SpringMVC中提供了两个过滤器：CharacterEncodingFilter和
>
> HiddenHttpMethodFilter
>
> 在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter
>
> 原因：
>
> - 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字
>
> 符集的
>
> - request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作
> - 而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作：
> - `String paramValue = request.getParameter(this.methodParam);`

# **8、RESTful案例**

> 表单请求和ajks可以发送post请求。

## **8.1、准备工作**

和传统 CRUD 一样，实现对员工信息的增删改查。

- 搭建环境
- 准备实体类

```java
 package com.atguigu.mvc.bean;
 public class Employee {
     private Integer id;
     private String lastName;
     private String email;
     //1 male, 0 female
     private Integer gender;
     public Integer getId() {
         return id;
     }
     public void setId(Integer id) {
         this.id = id;
     }
     public String getLastName() {
         return lastName;
     }
     public void setLastName(String lastName) {
         this.lastName = lastName;
     }
     public String getEmail() {
         return email;
     }
     public void setEmail(String email) {
         this.email = email;
     }
     public Integer getGender() {
         return gender;
     }
     public void setGender(Integer gender) {
         this.gender = gender;
     }
     public Employee(Integer id, String lastName, String email, Integergender) {
         super();
         this.id = id;
         this.lastName = lastName;
         this.email = email;
         this.gender = gender;
     }
     public Employee() {
     }
 }
```

- 准备dao模拟数据

```java
 package com.atguigu.mvc.dao;
 import java.util.Collection;
 import java.util.HashMap;
 import java.util.Map;
 import com.atguigu.mvc.bean.Employee;
 import org.springframework.stereotype.Repository;
 @Repository
 public class EmployeeDao {
     private static Map<Integer, Employee> employees = null;
     static{
         employees = new HashMap<Integer, Employee>();
         employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1));
         employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1));
         employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0));
         employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0));
         employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1));
     }
     private static Integer initId = 1006;
     public void save(Employee employee){
         if(employee.getId() == null){
             employee.setId(initId++);
         }
         employees.put(employee.getId(), employee);
     }
     public Collection<Employee> getAll(){
         return employees.values();
     }
     public Employee get(Integer id){
         return employees.get(id);
     }
     public void delete(Integer id){
         employees.remove(id);
     }
 }
```

## **8.2、功能清单**

| **功能**            | **URL 地址** | **请求方式** |
| ------------------- | ------------ | ------------ |
| 访问首页√           | /            | GET          |
| 查询全部数据√       | /employee    | GET          |
| 删除√               | /employee/2  | DELETE       |
| 跳转到添加数据页面√ | /toAdd       | GET          |
| 执行保存√           | /employee    | POST         |
| 跳转到更新数据页面√ | /employee/2  | GET          |
| 执行更新√           | /employee    | PUT          |

## **8.3、具体功能：访问首页**

### **①配置view-controller**

```xml
 <mvc:view-controller path="/" view-name="index"/>
```

### **②创建页面**

```xml
 <!DOCTYPE html>
 <html lang="en" xmlns:th="<http://www.thymeleaf.org>">
     <head>
         <meta charset="UTF-8" >
         <title>Title</title>
     </head>
     <body>
         <h1>首页</h1>
         <a th:href="@{/employee}">访问员工信息</a>
     </body>
 </html>
```

## **8.4、具体功能：查询所有员工数据**

### **①控制器方法**

```java
 @RequestMapping(value = "/employee", method = RequestMethod.GET) //需要指明请求方式 
 public String getEmployeeList(Model model){
     Collection<Employee> employeeList = employeeDao.getAll();
     model.addAttribute("employ eeList", employeeList);
     return "employee_list";
 }
```

### **②创建employee_list.html**

```html
 <!DOCTYPE html>
 <html lang="en" xmlns:th="<http://www.thymeleaf.org>">
     <head>
         <meta charset="UTF-8">
         <title>Employee Info</title>
         <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
     </head>
     <body>
         <table border="1" cellpadding="0" cellspacing="0" style="text-align:center;" id="dataTable">
             <tr>
                 <th colspan="5">Employee Info</th>
             </tr>
             <tr>
                 <th>id</th>
                 <th>lastName</th>
                 <th>email</th>
                 <th>gender</th>
                 <th>options(<a th:href="@{/toAdd}">add</a>)</th>
             </tr>
             <tr th:each="employee : ${employeeList}">
                 <td th:text="${employee.id}"></td>
                 <td th:text="${employee.lastName}"></td>
                 <td th:text="${employee.email}"></td>
                 <td th:text="${employee.gender}"></td>
                 <td>
                     <a class="deleteA" @click="deleteEmployee"
                        th:href="@{'/employee/'+${employee.id}}">delete</a>
                     <a th:href="@{'/employee/'+${employee.id}}">update</a>
                 </td>
             </tr>
         </table>
     </body>
 </html>
```

## **8.5、具体功能：删除**

### **①创建处理delete请求方式的表单**

```html
 <!-- 作用：通过超链接控制表单的提交，将post请求转换为delete请求 -->
 <form id="delete_form" method="post">
     <!-- HiddenHttpMethodFilter要求：必须传输_method请求参数，并且值为最终的请求方式 -->
     <input type="hidden" name="_method" value="delete"/>
 </form>
```

引入vue.js

```xml
 <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
```

删除超链接

```xml
 <a class="deleteA" @click="deleteEmployee"th:href="@{'/employee/'+${employee.id}}">delete</a>
```

通过vue处理点击事件

```html
 <script type="text/javascript">
     var vue = new Vue({
         el:"#dataTable",
         methods:{
             //event表示当前事件
             deleteEmployee:function (event) {
                 //通过id获取表单标签
                 var delete_form = document.getElementById("delete_form");
                 //将触发事件的超链接的href属性为表单的action属性赋值
                 delete_form.action = event.target.href;
                 //提交表单
                 delete_form.submit();
                 //阻止超链接的默认跳转行为
                 event.preventDefault();
             }
         }
     });
 </script>
```

### **③控制器方法**

```java
 @RequestMapping(value = "/employee/{id}", method = RequestMethod.DELETE)
 public String deleteEmployee(@PathVariable("id") Integer id){
     employeeDao.delete(id);
     return "redirect:/employee";//重定向
 }
```

## **8.6、具体功能：跳转到添加数据页面**

### **①配置view-controller**

```html
 <mvc:view-controller path="/toAdd" view-name="employee_add"></mvc:view-controller>
```

### **②创建employee_add.html**

```html
 <!DOCTYPE html>
 <html lang="en" xmlns:th="<http://www.thymeleaf.org>">
     <head>
         <meta charset="UTF-8">
         <title>Add Employee</title>
     </head>
     <body>
         <form th:action="@{/employee}" method="post">
             lastName:<input type="text" name="lastName"><br>
             email:<input type="text" name="email"><br>
             gender:<input type="radio" name="gender" value="1">male
             <input type="radio" name="gender" value="0">female<br>
             <input type="submit" value="add"><br>
         </form>
     </body>
 </html>
```

## **8.7、具体功能：执行保存**

### **①控制器方法**

```java
 @RequestMapping(value = "/employee", method = RequestMethod.POST)
 public String addEmployee(Employee employee){实体中的属性名字和请求参数的名字保持一致就可 
     employeeDao.save(employee);
     return "redirect:/employee";
 }
```

## **8.8、具体功能：跳转到更新数据页面**

### **①修改超链接**

```xml
 <a th:href="@{'/employee/'+${employee.id}}">update</a>  请求地址
```

### **②控制器方法**

```java
 @RequestMapping(value = "/employee/{id}", method = RequestMethod.GET)
 public String getEmployeeById(@PathVariable("id") Integer id, Model model){请求域共享，然后在页面回显数据
     Employee employee = employeeDao.get(id);
     model.addAttribute("employee", employee);
     return "employee_update";
 }
```

### **③创建employee_update.html**

```html
 <!DOCTYPE html>
 <html lang="en" xmlns:th="<http://www.thymeleaf.org>">
     <head>
         <meta charset="UTF-8">
         <title>Update Employee</title>
     </head>
     <body>
         <form th:action="@{/employee}" method="post">
             <input type="hidden" name="_method" value="put">
             <input type="hidden" name="id" th:value="${employee.id}">
             lastName:<input type="text" name="lastName" th:value="${employee.lastName}">
             <br>
             email:<input type="text" name="email" th:value="${employee.email}"><br>
             <!--
                 th:field="${employee.gender}"可用于单选框或复选框的回显
                 若单选框的value和employee.gender的值一致，则添加checked="checked"属性
       -->
             gender:<input type="radio" name="gender" value="1"th:field="${employee.gender}">male
             <input type="radio" name="gender" value="0"th:field="${employee.gender}">female<br>
             <input type="submit" value="update"><br>
         </form>
     </body>
 </html>
```

## **8.9、具体功能：执行更新**

### **①控制器方法**

```java
 @RequestMapping(value  = "/employee", method = RequestMethod.PUT)
 public String updateEmployee(Employee employee){
     employeeDao.save(employee);
     return "redirect:/employee";
 }
```

# 九、HttpMessageConverter

HttpMessageConverter，报文信息转换器，将**请求报文转换为Java对象，或将Java对象转换为响应报文。**

HttpMessageConverter提供了两个注解和两个类型：@RequestBody（将请求报文中的请求体转换为java对象），@ResponseBody（将对象转换为响应体），RequestEntity（可以接收整个请求，请求体和请求头），ResponseEntity（可以接收整个响应体，响应头和响应体 ）

> 浏览器发送请求到服务器，可以将请求报文转换为java对象，服务器响应浏览器的响应报文，可以将java对象转换为响应报文发送给浏览器。

@RequestBody，RequestEntity：将请求报文转换为java对象

@ResponseBody，ResponseEntity：将java对象转换为响应信息

springmvc框架结构：

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e299664-11d5-44ca-a231-6e26f95a79e2/Untitled.png)

### 1、@RequestBody

> 用来标示形参，然后形参就可以获取请求体。

一个请求包含：请求头，空行，请求体

@RequestBody可以获取请求体，需要在控制器方法设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

```java
<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br><input type="submit"></form>
12345
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){//请求体会传输到这个形参中 
    System.out.println("requestBody:"+requestBody);
    return "success";
}
12345
```

输出结果：

requestBody:username=admin&password=123456

### 2、RequestEntity

> 控制器形参的类型，可以接收请求报文。

RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息，RequestEntity可以封装整个请求，

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
123456
```

输出结果：

```java
requestHeader:[host:“localhost:8080”, connection:“keep-alive”, 
content-length:“27”, cache-control:“max-age=0”, sec-ch-ua:"" Not 
A;Brand";v=“99”, “Chromium”;v=“90”, “Google Chrome”;v=“90"”, 
sec-ch-ua-mobile:"?0", upgrade-insecure-requests:“1”, origin:“[<http://localhost:8080>”](<http://localhost:8080>%E2%80%9D/),
 user-agent:“Mozilla/5.0 (Windows NT 10.0; Win64; x64) 
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 
Safari/537.36”]
requestBody:username=admin&password=123
```

### 3、@ResponseBody（重点）

> 标示控制器方法，用于响应浏览器。

@ResponseBody用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应到浏览器.在原生servleapi中，通过形参HttpServletResponse参数，获取响应报文处理。在sparingmvc中通过@ResponseBody响应浏览器。

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";//如果不添加@ResponseBody注解，那么这个方法返回的就是视图名称，如果加上这个注解，那么返回的就是响应体
}
12345
```

结果：浏览器页面显示success

### 4、SpringMVC处理json

@ResponseBody处理json的步骤：

1. 导入jackson的依赖

```xml
<dependency><groupId>com.fasterxml.jackson.core</groupId><artifactId>jackson-databind</artifactId><version>2.12.1</version></dependency>
12345
```

1. 在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的Java对象转换为Json格式的字符串

```xml
<mvc:annotation-driven />
```

1. 在处理器方法上使用@ResponseBody注解进行标识
2. 将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串

```java
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}
12345
```

浏览器的页面中展示的结果：返回的是json字符串。

{“id”:1001,“username”:“admin”,“password”:“123456”,“age”:23,“sex”:“男”}

### 5、SpringMVC处理ajax

a>请求超链接：

```xml
<div id="app"><a th:href="@{/testAjax}" @click="testAjax">testAjax</a><br></div>
123
```

b>通过vue和axios处理点击事件：

```xml
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script><script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script><script type="text/javascript">
    var vue = new Vue({
        el:"#app",
        methods:{
            testAjax:function (event) {
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123456"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>
12345678910111213141516171819202122
```

c>控制器方法：

```java
@RequestMapping("/testAjax")
@ResponseBody
public String testAjax(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "hello,ajax";
}
123456
```

### 6、@RestController注解

@RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解。

### 7、ResponseEntity

> 用于控制器方法的返回值类型，可以接收整个返回报文。也就是自定义的响应报文。

ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文。

> 微服务和微服务之间的数据交互，使用的就是http+json数据格式的交互。也就是说微服务控制器中的每一个方法都需要加@ResponseBody注解