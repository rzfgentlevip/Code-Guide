---
title: 4、Spring核心原理二
icon: devicon:spring-wordmark
order: 4
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2026/01/17 13:27:44
permalink: /compre-guide/spring/Spring核心原理二/
---


## 请简述Spring MVC的执行流程

Spring MVC的执行流程遵循了前端控制器模式，通过DispatcherServlet协调各个组件来接收请求、匹配处理器、执行处理器、渲染视图的过程，实现了请求到视图的端到端处理。

具体执行流程如下：

1. **客户端发送请求**：客户端发送HTTP请求到服务器上部署的Spring MVC应用程序。
2. **DispatcherServlet接收请求**：请求被DispatcherServlet（前端控制器）拦截，它是Spring MVC的核心组件之一，负责统一管理请求的处理过程。
3. **HandlerMapping匹配处理器**：DispatcherServlet调用HandlerMapping来确定请求的处理器（Controller）。
4. **执行处理器**：确定了处理器之后，DispatcherServlet将请求转发给相应的Controller进行处理，Controller执行业务逻辑并返回ModelAndView对象。
5. **视图解析和渲染**：DispatcherServlet通过ViewResolver解析Controller返回的逻辑视图名，并选择合适的View进行渲染，最终将视图呈现给用户。
6. **HTTP响应**：渲染后的视图作为HTML响应发送给客户端。

## 谈谈你对Spring MVC中九大组件的理解

Spring MVC是Spring框架的一个模块，用于构建基于MVC（Model-View-Controller）设计模式的Web应用程序。在Spring MVC中，有九大组件是非常重要的，它们分别是：

1. **DispatcherServlet**：这是Spring MVC的核心，是一个前端控制器，负责接收用户请求并分发给相应的处理器。
2. **HandlerMapping**：处理器映射，根据请求信息找到对应的处理器（Controller）。
3. **HandlerAdapter**：处理器适配器，用于调用相应的处理器（Controller）的方法。
4. **Controller**：控制器，处理由DispatcherServlet分发的请求。
5. **ViewResolver**：视图解析器，解析逻辑视图名到真正的视图（如JSP）。
6. **View**：视图，用于展示模型（Model）数据。
7. **ModelAndView**：模型和视图，用于封装模型数据和视图信息。
8. **ModelMap**：模型数据，用于存放模型数据，可以在视图中直接使用。
9. **LocaleResolver**：区域解析器，用于解析用户的语言和区域设置。

这九大组件共同构成了Spring MVC的完整流程：

首先，DispatcherServlet接收用户请求，然后通过HandlerMapping找到对应的Controller，再通过HandlerAdapter调用Controller的方法，方法返回的ModelAndView包含了模型数据和视图信息，最后通过ViewResolver解析视图，并通过View展示模型数据。在这个过程中，还可以通过LocaleResolver来解析用户的语言和区域设置，以实现国际化。


## Spring MVC有哪些常用注解？它们的作用是什么？

@Controller：用于标识一个类为Spring MVC的控制器，处理客户端的请求并返回相应的视图。

@RequestMapping：用于将URL路径映射到特定的处理方法，可用于类级别或方法级别，用于定义请求URL和处理请求的方法。

@ResponseBody：用于将方法的返回值直接作为HTTP Response的主体内容返回，通常用于返回 JSON 或 XML 格式的数据。

@PathVariable：用于将请求URL中的模板变量映射到处理方法的参数中。

@RequestParam：用于将请求参数映射到处理方法的参数中，可以指定参数名称、是否必须等。

@RequestParamMap：用于将所有的请求参数映射为一个Map类型的参数。

@ModelAttribute：用于将请求参数绑定到Model对象中，通常用于在视图渲染之前填充模型数据。

@SessionAttributes：用于指定处理方法的返回值需要存储到会话中的属性值。

@InitBinder：用于定制数据绑定规则和初始化数据绑定规则。

@Validated：用于标记其后的方法参数需要进行验证。


## spring mvc 和 struts 的区别是什么？

（1）架构设计

Spring MVC是基于Servlet API构建的，而Struts2是通过Filter实现的。这意味着Spring MVC的入口是一个Servlet，而Struts2的入口是一个Filter。这导致了两者在处理请求时的不同机制。

（2）拦截器

Spring MVC 的拦截器机制通过 HandlerInterceptor 接口进行实现。通过实现这个接口，你可以在处理程序执行的前后插入逻辑，可以在请求处理之前或之后对请求进行预处理或后处理。拦截器可以用来实现日志记录、权限校验、国际化、主题切换等各种需求。

Struts 的拦截器机制是通过拦截器栈（interceptor stacks）实现的。在Struts2中，通过配置拦截器栈，可以在请求处理的各个阶段插入预处理逻辑或后处理逻辑。拦截器可以用于日志记录、权限校验、异常处理、输入验证等。

（3）配置方式

Spring MVC 的配置更加灵活，可以使用 XML 配置、Java 注解或者 Java 类配置（JavaConfig）来配置控制器、视图解析器等。

Struts 使用基于 XML 的配置文件来定义控制器（Action）、拦截器、结果视图等。

（4）扩展性和灵活性

Spring MVC强调依赖注入，通过Spring容器管理对象和组件，使得应用更加灵活和可测试。 Struts不提供像Spring那样的依赖注入机制，更多地依赖于配置文件，因此在灵活性方面可能稍逊于Spring MVC。

（5）性能

由于Spring MVC的轻量级设计，其代码量相对较少，运行时间也更快，因此在性能方面通常优于Struts2。


## @RestController和@Controller有什么区别？

@RestController和@Controller是Spring MVC中常用的注解，它们的主要区别在于返回值的不同：

@Controller：用于标识控制器类的注解。在Spring MVC中，使用@Controller注解标记的类表示该类是控制器，可以处理客户端的请求，并返回相应的视图。

@RestController：是Spring4.0引入的一个组合注解，用于标识RESTful风格的控制器类。它相当于@Controller和@ResponseBody的组合，表示该类处理RESTful请求，方法的返回值直接作为HTTP Response的主体内容返回，而不是作为视图进行解析。

因此，@RestController注解在处理HTTP请求时，会将方法的返回值直接转换为JSON/XML等格式的数据返回给客户端，而@Controller注解一般用于传统的Web应用程序开发，将方法的返回值解析为视图进行渲染。

## Spring MVC如何匹配请求路径？

在Spring MVC中，请求路径的匹配是通过DispatcherServlet和HandlerMapping来完成的。具体步骤如下：

1. 客户端发送请求到服务器，请求首先到达DispatcherServlet。
2. DispatcherServlet接收到请求后，会调用HandlerMapping（处理器映射）来找到对应的Controller（控制器）。
3. HandlerMapping会根据请求路径（URI）和请求方法（GET、POST等）来查找匹配的Controller。这是通过配置文件或者注解来实现的。
4. 如果找到匹配的Controller，HandlerMapping会返回一个HandlerExecutionChain对象，包含了处理器（Handler）、拦截器（Interceptors）和适配器（Adapter）。
5. DispatcherServlet根据HandlerExecutionChain对象，依次调用拦截器、处理器和适配器来处理请求。
6. 最后，将处理结果返回给客户端。

在Spring MVC中，可以通过多种方式来配置请求路径与Controller的映射关系，例如使用XML配置文件或者使用注解（如@RequestMapping）。

RequestMapping可以实现模糊匹配路径，比如：

- ？表示一个字符；
- *表示任意字符；
- **匹配多层路径；

## Spring MVC中，在调用controller接口前都进行了哪些操作？

Spring MVC的执行流程：

1. **DispatcherServlet（前端控制器）**：所有的请求都会先到达前端控制器 DispatcherServlet，它是整个 Spring MVC 的核心。
2. **HandlerMapping（处理器映射器）**：DispatcherServlet 会根据请求的 URL 找到对应的 Handler（处理器），这个过程中会用到 HandlerMapping。
3. **Interceptors（拦截器）**，在请求到达 Handler 之前，可以设置一些拦截器对请求进行预处理，例如权限验证、日志记录等。
4. **Handler（处理器）**：这里指的是 Controller 接口，当请求到达 Controller 后，会执行相应的方法进行处理。
5. **ModelAndView（模型和视图）**：Controller 处理完请求后，会返回一个 ModelAndView 对象，其中包含了响应的数据和视图信息。
6. **ViewResolver（视图解析器）**：根据 ModelAndView 中的视图信息，通过 ViewResolver 解析出实际的视图对象。
7. **View（视图）**：将解析出的视图对象渲染成 HTML 页面，返回给客户端。

因此，DispatcherServlet（前端控制器）、HandlerMapping（处理器映射器）、Interceptors（拦截器）就是在调用controller接口前进行的操作。

## SpringMVC如何获取请求的参数？

（1）@RequestParam

最常见的方法，用于从请求参数中取值到控制器方法的参数上。

（2）@PathVariable

从URL路径中提取参数时，可以使用@PathVariable注解。

在这个例子中，当发送一个GET请求到/users/123时，123会被解析为id参数的值。

```java
@Controller  
public class MyController {  
  
    @GetMapping("/users/{id}")  
    public String getUser(@PathVariable Long id) {  
        // 使用id参数  
        return "user";  
    }  
}
```

（3）@ModelAttribute

当请求参数与对象属性对应时，可以使用@ModelAttribute注解来绑定请求参数到JavaBean上。

当发送一个GET请求到/create?id=1&name=nezha&age=18时，请求参数会被绑定到User对象的属性上。

```java
@Controller  
public class MyController {  
  
    @GetMapping("/create")  
    public String create(@ModelAttribute User user) {  
        // 使用user对象中的属性  
        return "create";  
    }  
  
    public static class User {  
        private String id;  
        private String name;  
        private String age;  
        // getters and setters  
    }  
}
```

（4）@RequestBody

当请求体中包含JSON或XML数据时，可以使用@RequestBody注解来自动解析请求体并绑定到方法参数上。

（5）HttpServletRequest

还可以直接通过HttpServletRequest对象来获取请求参数，尽管这不是Spring MVC推荐的方式，因为它没有利用Spring MVC的参数绑定特性。

在使用HttpServletRequest时，你需要自己处理参数的获取和类型转换。

```java
@Controller  
public class MyController {  
  
    @GetMapping("/oldWay")  
    public String oldWay(HttpServletRequest request) {  
        String query = request.getParameter("query");  
        // 使用query参数  
        return "oldWay";  
    }  
}
```

