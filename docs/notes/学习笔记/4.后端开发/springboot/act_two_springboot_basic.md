---
title: SpringBoot基础入门二
icon: streamline-freehand-color:notes-quill
order: 2
author: bugcode
date: 2020-01-10T00:00:00.000Z
category:
  - SPRINGBOOT
  - SPRING
  - JAVA
tag:
  - 后端
  - java
  - spring
sticky: false
star: true
footer: Spring基础
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/vlmm2sea/
---

# 2、SpringBoot基础入门（二）

# 1、Helloword程序

需求：浏览发送/hello请求，响应 Hello，Spring Boot 2

## 1.1、创建Maven工程

引入依赖

```xml
<!--引入springboot的父工程-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>

    <dependencies>
        <!--要用springboot开发web场景模块，就引入springboot对应web模块的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!--指定编译构建的jdk版本，不需要在服务期安装部署tomcat,只需要将程序打成jar包，
//在服务期上执行即可-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.4</version>
                <!-- 下面指定为自己需要的版本 -->
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## 1.2、创建主程序

```java
package com.learn.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 主程序类/主配置类：标注了@SpringBootApplication注解的类。此类有一个即可。
 * @SpringBootApplication：告诉spring这是一个SpringBoot应用
 */
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        // 项目启动，告诉spring加载主配置类。
        SpringApplication.run(MainApplication.class, args);
    }

}
```

## 1.3、编写业务

传统项目框架搭建好后，业务如何开发，还怎么开发。
springboot提供了@RestController标签，相当于 `@ResponseBody + @Controller`

```java
package com.learn.boot.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

//@Controller
//public class HelloController {
//
//    @ResponseBody // 声明方法返回值直接传给浏览器，非页面资源路径
//    @RequestMapping("/hello")
//    public String handle01(){
//        return "Hello, Spring Boot 2!";
//    }
//}

//@ResponseBody // 声明此类下的所有方法返回值直接传给浏览器，非页面资源路径
//@Controller
//public class HelloController {
//
//    @RequestMapping("/hello")
//    public String handle01(){
//        return "Hello, Spring Boot 2!";
//    }
//}

@RestController // 相当于 @ResponseBody + @Controller
public class HelloController {

    @RequestMapping("/hello")
    public String handle01(){
        return "Hello, Spring Boot 2!";
    }
}

@RequestMapping("/api")
@Component
public class ReturnValueToPage {

    @Autowired
    UserRegister userRegister;

//   /api/returnvalue
    @ResponseBody
    @RequestMapping("/returnvalue")
    public UserRegister returnValue(){

//会将对象值封装为json直接返回
        return userRegister;
    }
}
```

> @ResponseBody 注解既可以作用在类上，还可以作用在方法上。

**知识补充：概念性理解**

@ResponseBody 注解的作用是将Controller的方法返回的对象，通过适当的转换器转换为指定的格式之后，写入到response对象的body区，通常用来返回JSON数据或者是XML数据。
本质

@ResponseBody的作用其实是将java对象转为json格式的数据，然后直接写入HTTP response 的body中；一般在异步获取数据时使用
注意

在使用此注解之后不会再走视图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据。

```
1. @ResponseBody 是作用在方法上的。
2. @ResponseBody 表示该方法的返回结果直接写入 HTTP response body中，一般在异步获取数据时使用【也就是AJAX】。
```

在使用 @RequestMapping后，返回值通常解析为跳转路径，但是加上 @ResponseBody 后返回结果不会被解析为跳转路径，而是直接写入 HTTP response body 中。 比如异步获取 json 数据，加上 @ResponseBody 后，会直接返回 json 数据。@RequestBody 将 HTTP 请求正文插入方法中，使用适合的 HttpMessageConverter 将请求体写入某个对象。

## 1.4、测试

直接运行main方法

[http://localhost:8080/hello](http://localhost:8080/hello)

## 1.5、简化部署

1. springboot是整合其它所有技术栈的总框架，springboot为了简化配置，将所有配置存放在同一个配置文件中：**application.properties。**
2. 相比于传统spring，简化了很多配置，甚至tomcat都不用配置就能启动，以修改启动端口为例，不用再去tomcat中修改配置，直接声明即可。

```java
server.port=8888
```

可用配置列表：

[Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043193.png)

## 1.6、简化配置

springboot可以将整个项目打包成一个jar包，自带了运行环境，可直接运行。又称为“Fat jars”，小胖jar。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043974.png)

打包部署，pom文件中添加下面插件：
```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
</build>
```

在IDEA的Maven插件上点击运行 clean 、package，把helloworld工程项目的打包成jar包，

打包好的jar包被生成在helloworld工程项目的target文件夹内。

用cmd运行`java -jar boot-01-helloworld-1.0-SNAPSHOT.jar`，既可以运行helloworld工程项目。

> 如果不用maven插件打包，会产生打包没有主程序清单的情况。

将jar包直接在目标服务器执行即可。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043871.png)

# 2、自动配置原理

## 2.1、springboot两大优点

### 2.1.1、依赖管理

**父项目做依赖管理**

在helloworld案例中，依赖只需要**引入一个springboot父项目依赖、一个指定场景模块的子项目依赖**，指定场景下将要使用的所有包就能自动导入到项目中。

每一个项目都有一个父依赖项目。父项目一般都是做依赖管理的，子项目以后继承父项目，不需要写依赖版本号。

```xml
<!--引入springboot的父工程 主要做依赖管理-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>

<!-- 引入具体的场景模块-->
    <dependencies>
        <!--要用springboot开发web场景模块，就引入springboot对应web模块的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

> 一些常用的starter场景：

1. `spring-boot-starter-cache`: Starter for using Spring Framework’s caching support
2. `spring-boot-starter-data-jdbc` : Starter for using Spring Data JDBC
3. `spring-boot-starter-data-jpa`:Starter for using Spring Data JPA with Hibernate
4. `spring-boot-starter-data-redis`:Starter for using Redis key-value data store with Spring Data Redis and the Lettuce client
5. `spring-boot-starter-jdbc`:Starter for using JDBC with the HikariCP connection pool

每个springboot工程，都要引入springboot父项目依赖，**因为在maven中，父项目的作用，是用来做依赖管理，会声明非常多的依赖**，子项目继承父项目后，就不用再写依赖的版本号。

**无需关注版本号，自动版本仲裁**

1、引入依赖默认都可以不写版本

2、但引入非版本仲裁的jar，要写版本号。

springboot的父项目spring-boot-starter-parent，又引用了spring-boot-dependencies作为父项目。其中properties属性几乎声明了所有开发中常用的依赖的版本号。

所以我们引入指定模块依赖时便会自动找到对应版本的包。这称为**自动版本仲裁机制**。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043891.png)

**可以修改默认版本号**

按照Maven的**就近原则**，在pom中修改之后，先按照pom中的版本号，否则遵循父类中的版本。

要修改springboot中已声明的组件的版本号。

1. 查看spring-boot-dependencies里面规定当前组件依赖声明，对应的properties中版本用的 key。
2. 在pom.xml中properties重新指定该Key对应的版本号即可。

```text
<!--引入springboot的父工程-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>

--重新指定组件的版本号
    <properties>
        <!--默认使用 <mysql.version>8.0.21</mysql.version>-->
        <mysql.version>5.1.43</mysql.version> //修改默认使用的版本
    </properties>

    <dependencies>
        <!--要用springboot开发web场景模块，就引入springboot对应web模块的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

    </dependencies>
```

**通过导入starter场景启动器，便捷开发**

1、starter是一组依赖集合的描述。

2、只要引入一个starter，一个完整的开发场景的所有依赖就能被自动引入。

3、所有官方starter的命名规则：spring-boot-starter-*，*代表场景名称。

4、SpringBoot所有支持的场景清单：

[https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)

5、支持创建自定义Starter场景启动器。但这些第三方的Starter，Springboot建议命名规则为：

- *-spring-boot-starter。例如：thirdpartyproject-spring-boot-starter。

6、所有场景启动器最底层的依赖，都会引用springboot自动配置的核心依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

在pom文件中右击可以生成依赖图：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043406.png)

### 2.1.2、自动配置

以pom.xml中引入 spring-boot-starter-web 为例。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043407.png)

**自动配置好tomcat**

自动配置步骤：

1.  引入Tomcat依赖。spring-boot-starter-web 默认依赖了spring-boot-starter-tomcat。
2.  配置Tomcat。（自动配置原理章节会详解自动配置和启动的原理。）

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
</dependency>
```

**自动配好SpringMVC**

1. 引入SpringMVC全套组件
2. 自动配好SpringMVC常用组件（功能）

**自动配好Web常见功能**

以spring、springmvc整合为例，在web.xml中，

- 要配置DispatcherServlet，拦截所有请求，
- 要配置 CharacterEncodingFilter，解决字符乱码问题。
- 要配置 ViewResolver ，视图解析器。
- 要配置 multipartResolver ，文件上传解析器。

SpringBoot帮我们配置好了所有web开发的常见场景，上述所有的组件，都会自动配置在IOC容器中。

```text
package com.learn.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * 主程序类/主配置类：标注了@SpringBootApplication注解的类。此类有一个即可。
 * @SpringBootApplication：告诉spring这是一个SpringBoot应用
 */
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        // 项目启动，告诉spring加载主配置类。
        //SpringApplication.run(MainApplication.class, args);

        // ConfigurableApplicationContext IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        // 容器中包含了所有组件信息
        String[] names = run.getBeanDefinitionNames();
        for(String name:names){
            System.out.println(name);
        }
    }

}
D:\soft\jdk1.8\bin\java.exe -XX:TieredStopAtLevel=1 -noverify -Dspring.output.ansi.enabled=always -Dcom.sun.management.jmxremote -Dspring.jmx.enabled=true -Dspring.liveBeansView.mbeanDomain -Dspring.application.admin.enabled=true "-javaagent:D:\soft\IDEA\IntelliJ IDEA 2020.2.3\lib\idea_rt.jar=55339:D:\soft\IDEA\IntelliJ IDEA 2020.2.3\bin" -Dfile.encoding=UTF-8 -classpath D:\soft\jdk1.8\jre\lib\charsets.jar;D:\soft\jdk1.8\jre\lib\deploy.jar;D:\soft\jdk1.8\jre\lib\ext\access-bridge-64.jar;D:\soft\jdk1.8\jre\lib\ext\cldrdata.jar;D:\soft\jdk1.8\jre\lib\ext\dnsns.jar;D:\soft\jdk1.8\jre\lib\ext\jaccess.jar;D:\soft\jdk1.8\jre\lib\ext\jfxrt.jar;D:\soft\jdk1.8\jre\lib\ext\localedata.jar;D:\soft\jdk1.8\jre\lib\ext\nashorn.jar;D:\soft\jdk1.8\jre\lib\ext\sunec.jar;D:\soft\jdk1.8\jre\lib\ext\sunjce_provider.jar;D:\soft\jdk1.8\jre\lib\ext\sunmscapi.jar;D:\soft\jdk1.8\jre\lib\ext\sunpkcs11.jar;D:\soft\jdk1.8\jre\lib\ext\zipfs.jar;D:\soft\jdk1.8\jre\lib\javaws.jar;D:\soft\jdk1.8\jre\lib\jce.jar;D:\soft\jdk1.8\jre\lib\jfr.jar;D:\soft\jdk1.8\jre\lib\jfxswt.jar;D:\soft\jdk1.8\jre\lib\jsse.jar;D:\soft\jdk1.8\jre\lib\management-agent.jar;D:\soft\jdk1.8\jre\lib\plugin.jar;D:\soft\jdk1.8\jre\lib\resources.jar;D:\soft\jdk1.8\jre\lib\rt.jar;D:\soft\IDEA\work\springboot2-master\boot-01-helloworld-2\target\classes;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-starter-web\2.3.4.RELEASE\spring-boot-starter-web-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-starter\2.3.4.RELEASE\spring-boot-starter-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot\2.3.4.RELEASE\spring-boot-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-autoconfigure\2.3.4.RELEASE\spring-boot-autoconfigure-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-starter-logging\2.3.4.RELEASE\spring-boot-starter-logging-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\ch\qos\logback\logback-classic\1.2.3\logback-classic-1.2.3.jar;C:\Users\MrR\.m2\repository\ch\qos\logback\logback-core\1.2.3\logback-core-1.2.3.jar;C:\Users\MrR\.m2\repository\org\apache\logging\log4j\log4j-to-slf4j\2.13.3\log4j-to-slf4j-2.13.3.jar;C:\Users\MrR\.m2\repository\org\apache\logging\log4j\log4j-api\2.13.3\log4j-api-2.13.3.jar;C:\Users\MrR\.m2\repository\org\slf4j\jul-to-slf4j\1.7.30\jul-to-slf4j-1.7.30.jar;C:\Users\MrR\.m2\repository\jakarta\annotation\jakarta.annotation-api\1.3.5\jakarta.annotation-api-1.3.5.jar;C:\Users\MrR\.m2\repository\org\yaml\snakeyaml\1.26\snakeyaml-1.26.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-starter-json\2.3.4.RELEASE\spring-boot-starter-json-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.2\jackson-databind-2.11.2.jar;C:\Users\MrR\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.2\jackson-annotations-2.11.2.jar;C:\Users\MrR\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.2\jackson-core-2.11.2.jar;C:\Users\MrR\.m2\repository\com\fasterxml\jackson\datatype\jackson-datatype-jdk8\2.11.2\jackson-datatype-jdk8-2.11.2.jar;C:\Users\MrR\.m2\repository\com\fasterxml\jackson\datatype\jackson-datatype-jsr310\2.11.2\jackson-datatype-jsr310-2.11.2.jar;C:\Users\MrR\.m2\repository\com\fasterxml\jackson\module\jackson-module-parameter-names\2.11.2\jackson-module-parameter-names-2.11.2.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-starter-tomcat\2.3.4.RELEASE\spring-boot-starter-tomcat-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\org\apache\tomcat\embed\tomcat-embed-core\9.0.38\tomcat-embed-core-9.0.38.jar;C:\Users\MrR\.m2\repository\org\glassfish\jakarta.el\3.0.3\jakarta.el-3.0.3.jar;C:\Users\MrR\.m2\repository\org\apache\tomcat\embed\tomcat-embed-websocket\9.0.38\tomcat-embed-websocket-9.0.38.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-web\5.2.9.RELEASE\spring-web-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-beans\5.2.9.RELEASE\spring-beans-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-webmvc\5.2.9.RELEASE\spring-webmvc-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-aop\5.2.9.RELEASE\spring-aop-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-context\5.2.9.RELEASE\spring-context-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-expression\5.2.9.RELEASE\spring-expression-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\projectlombok\lombok\1.18.12\lombok-1.18.12.jar;C:\Users\MrR\.m2\repository\org\springframework\boot\spring-boot-configuration-processor\2.3.4.RELEASE\spring-boot-configuration-processor-2.3.4.RELEASE.jar;C:\Users\MrR\.m2\repository\org\slf4j\slf4j-api\1.7.30\slf4j-api-1.7.30.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-core\5.2.9.RELEASE\spring-core-5.2.9.RELEASE.jar;C:\Users\MrR\.m2\repository\org\springframework\spring-jcl\5.2.9.RELEASE\spring-jcl-5.2.9.RELEASE.jar com.atguigu.boot.Boot01Helloworld2Application

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.4.RELEASE)

2023-10-29 10:45:24.530  INFO 20320 --- [           main] c.a.boot.Boot01Helloworld2Application    : Starting Boot01Helloworld2Application on rzf with PID 20320 (D:\soft\IDEA\work\springboot2-master\boot-01-helloworld-2\target\classes started by MrR in D:\soft\IDEA\work\springboot2-master)
2023-10-29 10:45:24.532  INFO 20320 --- [           main] c.a.boot.Boot01Helloworld2Application    : No active profile set, falling back to default profiles: default
2023-10-29 10:45:25.361  INFO 20320 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-10-29 10:45:25.370  INFO 20320 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-10-29 10:45:25.370  INFO 20320 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.38]
2023-10-29 10:45:25.371  INFO 20320 --- [           main] o.a.catalina.core.AprLifecycleListener   : Loaded Apache Tomcat Native library [1.2.37] using APR version [1.7.4].
2023-10-29 10:45:25.371  INFO 20320 --- [           main] o.a.catalina.core.AprLifecycleListener   : APR capabilities: IPv6 [true], sendfile [true], accept filters [false], random [true].
2023-10-29 10:45:25.371  INFO 20320 --- [           main] o.a.catalina.core.AprLifecycleListener   : APR/OpenSSL configuration: useAprConnector [false], useOpenSSL [true]
2023-10-29 10:45:25.374  INFO 20320 --- [           main] o.a.catalina.core.AprLifecycleListener   : OpenSSL successfully initialized [OpenSSL 1.1.1u  30 May 2023]
2023-10-29 10:45:25.466  INFO 20320 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-10-29 10:45:25.466  INFO 20320 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 900 ms
2023-10-29 10:45:25.642  INFO 20320 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2023-10-29 10:45:25.807  INFO 20320 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-10-29 10:45:25.816  INFO 20320 --- [           main] c.a.boot.Boot01Helloworld2Application    : Started Boot01Helloworld2Application in 1.602 seconds (JVM running for 2.251)
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
boot01Helloworld2Application
org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory
dataSource
dataSourceProperties
person
userRegister
valueProperties
helloController
returnValueToPage
org.springframework.boot.autoconfigure.AutoConfigurationPackages
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration
propertySourcesPlaceholderConfigurer
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration$TomcatWebSocketConfiguration
websocketServletWebServerCustomizer
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryConfiguration$EmbeddedTomcat
tomcatServletWebServerFactory
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
servletWebServerFactoryCustomizer
tomcatServletWebServerFactoryCustomizer
org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor
org.springframework.boot.context.internalConfigurationPropertiesBinderFactory
org.springframework.boot.context.internalConfigurationPropertiesBinder
org.springframework.boot.context.properties.BoundConfigurationProperties
org.springframework.boot.context.properties.ConfigurationBeanFactoryMetadata
server-org.springframework.boot.autoconfigure.web.ServerProperties
webServerFactoryCustomizerBeanPostProcessor
errorPageRegistrarBeanPostProcessor
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletConfiguration
dispatcherServlet
spring.mvc-org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletRegistrationConfiguration
dispatcherServletRegistration
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration
taskExecutorBuilder
applicationTaskExecutor
spring.task.execution-org.springframework.boot.autoconfigure.task.TaskExecutionProperties
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration
error
beanNameViewResolver
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$DefaultErrorViewResolverConfiguration
conventionErrorViewResolver
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
errorAttributes
basicErrorController
errorPageCustomizer
preserveErrorControllerTargetClassPostProcessor
spring.resources-org.springframework.boot.autoconfigure.web.ResourceProperties
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration
requestMappingHandlerAdapter
requestMappingHandlerMapping
welcomePageHandlerMapping
mvcConversionService
mvcValidator
mvcContentNegotiationManager
mvcPathMatcher
mvcUrlPathHelper
viewControllerHandlerMapping
beanNameHandlerMapping
routerFunctionMapping
resourceHandlerMapping
mvcResourceUrlProvider
defaultServletHandlerMapping
handlerFunctionAdapter
mvcUriComponentsContributor
httpRequestHandlerAdapter
simpleControllerHandlerAdapter
handlerExceptionResolver
mvcViewResolver
mvcHandlerMappingIntrospector
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter
defaultViewResolver
viewResolver
requestContextFilter
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
formContentFilter
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration
mbeanExporter
objectNamingStrategy
mbeanServer
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration
springApplicationAdminRegistrar
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration$ClassProxyingConfiguration
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration
applicationAvailability
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration
lifecycleProcessor
spring.lifecycle-org.springframework.boot.autoconfigure.context.LifecycleProperties
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$Jackson2ObjectMapperBuilderCustomizerConfiguration
standardJacksonObjectMapperBuilderCustomizer
spring.jackson-org.springframework.boot.autoconfigure.jackson.JacksonProperties
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperBuilderConfiguration
jacksonObjectMapperBuilder
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$ParameterNamesModuleConfiguration
parameterNamesModule
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperConfiguration
jacksonObjectMapper
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration
jsonComponentModule
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration$StringHttpMessageConverterConfiguration
stringHttpMessageConverter
org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration$MappingJackson2HttpMessageConverterConfiguration
mappingJackson2HttpMessageConverter
org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration
messageConverters
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration
spring.info-org.springframework.boot.autoconfigure.info.ProjectInfoProperties
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration
taskSchedulerBuilder
spring.task.scheduling-org.springframework.boot.autoconfigure.task.TaskSchedulingProperties
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration
restTemplateBuilder
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration$TomcatWebServerFactoryCustomizerConfiguration
tomcatWebServerFactoryCustomizer
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration
characterEncodingFilter
localeCharsetMappingsCustomizer
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration
multipartConfigElement
multipartResolver
spring.servlet.multipart-org.springframework.boot.autoconfigure.web.servlet.MultipartProperties
org.springframework.aop.config.internalAutoProxyCreator
```

输出结果中能查到spring+springmvc中的组件。

**规定了默认的包结构**

spring、springmvc整合时，要指定controller、servce等包扫描路径，例如web.xml中：

```text
<context:annotation-config />
	<context:component-scan base-package="com.cff.springwork">
</context:component-scan>
```

在springboot中：

1. **主程序所在包及其下面的所有子包**里面的组件都会被默认自动扫描进来。
2. 以前包扫描配置不再必须做。
3. 也支持修改扫描路径。

```java
假设controller类放到了主程序外层的文件夹下，可以通过scanBasePackages将默认
扫描路径范围设置的更外层些。
    
方法一：
    @SpringBootApplication(scanBasePackages="com.learn")
    public class MainApplication {...}

方法二：
	通过@ComponentScan 指定扫描路径。
    
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan("com.learn")
    public class MainApplication {..}

注意：
	因为 @SpringBootApplication 等同于
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan("com.learn")
	所以，要使用@ComponentScan，就不能使用@SpringBootApplication。
SpringBootApplication会默认扫描主程序下的所有包，如果在配置了ComponentScan，那么就会扫描冲突。
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043635.png)

[https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code)

**各种配置拥有默认值**

默认配置最终都是映射到某个类上，如：MultipartProperties。

配置文件的值最终会绑定每个类上，这个类会在容器中创建对象。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211043566.png)

所以，在springboot项目中，如果要用到这些配置值时，直接通过springboot底层容器，拿到这个类的对象，提取出默认值即可，至于这些默认值是如何在启动时绑定的，后续章节会讲。

**按需加载所有自动配置项**

springboot中有非常多的starter，**只有在pom.xml中引入了对应场景依赖，这个场景的自动配置才会开启**。

SpringBoot所有的自动配置功能都在 spring-boot-autoconfigure 包里面。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044285.png)

spring-boot-autoconfigure 中，集成了多个模块场景下的自动配置。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044373.png)

没有导入对应的场景启动器starter，那么该场景的自动配置逻辑代码会报红，代表不会生效。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044926.png)

### 2.1.3、kafka-starter

KafkaProperties:配置kafka的属性信息，通过@ConfigurationProperties(prefix = "spring.kafka")注解和yml配置文件中的配置绑定。

```java
@ConfigurationProperties(prefix = "spring.kafka")
public class KafkaProperties {}
```

KafkaAutoConfiguration：kafka自动配置类

```java
@Configuration(proxyBeanMethods = false)//标注是一个配置类
@ConditionalOnClass(KafkaTemplate.class)//条件加载注入
@EnableConfigurationProperties(KafkaProperties.class)//开启属性配置绑定
@Import({ KafkaAnnotationDrivenConfiguration.class, KafkaStreamsAnnotationDrivenConfiguration.class })
public class KafkaAutoConfiguration {

//绑定配置文件的kafka属性类
	private final KafkaProperties properties;

	public KafkaAutoConfiguration(KafkaProperties properties) {
		this.properties = properties;
	}
//注入kafka 的KafkaTemplate
	@Bean
	@ConditionalOnMissingBean(KafkaTemplate.class)
	public KafkaTemplate<?, ?> kafkaTemplate(ProducerFactory<Object, Object> kafkaProducerFactory,
			ProducerListener<Object, Object> kafkaProducerListener,
			ObjectProvider<RecordMessageConverter> messageConverter) {
		KafkaTemplate<Object, Object> kafkaTemplate = new KafkaTemplate<>(kafkaProducerFactory);
		messageConverter.ifUnique(kafkaTemplate::setMessageConverter);
		kafkaTemplate.setProducerListener(kafkaProducerListener);
		kafkaTemplate.setDefaultTopic(this.properties.getTemplate().getDefaultTopic());
		return kafkaTemplate;
	}

	@Bean
	@ConditionalOnMissingBean(ProducerListener.class)
	public ProducerListener<Object, Object> kafkaProducerListener() {
		return new LoggingProducerListener<>();
	}

	@Bean
	@ConditionalOnMissingBean(ConsumerFactory.class)
	public ConsumerFactory<?, ?> kafkaConsumerFactory(
			ObjectProvider<DefaultKafkaConsumerFactoryCustomizer> customizers) {
		DefaultKafkaConsumerFactory<Object, Object> factory = new DefaultKafkaConsumerFactory<>(
				this.properties.buildConsumerProperties());
		customizers.orderedStream().forEach((customizer) -> customizer.customize(factory));
		return factory;
	}

	@Bean
	@ConditionalOnMissingBean(ProducerFactory.class)
	public ProducerFactory<?, ?> kafkaProducerFactory(
			ObjectProvider<DefaultKafkaProducerFactoryCustomizer> customizers) {
		DefaultKafkaProducerFactory<?, ?> factory = new DefaultKafkaProducerFactory<>(
				this.properties.buildProducerProperties());
		String transactionIdPrefix = this.properties.getProducer().getTransactionIdPrefix();
		if (transactionIdPrefix != null) {
			factory.setTransactionIdPrefix(transactionIdPrefix);
		}
		customizers.orderedStream().forEach((customizer) -> customizer.customize(factory));
		return factory;
	}

	@Bean
	@ConditionalOnProperty(name = "spring.kafka.producer.transaction-id-prefix")
	@ConditionalOnMissingBean
	public KafkaTransactionManager<?, ?> kafkaTransactionManager(ProducerFactory<?, ?> producerFactory) {
		return new KafkaTransactionManager<>(producerFactory);
	}

	@Bean
	@ConditionalOnProperty(name = "spring.kafka.jaas.enabled")
	@ConditionalOnMissingBean
	public KafkaJaasLoginModuleInitializer kafkaJaasInitializer() throws IOException {
		KafkaJaasLoginModuleInitializer jaas = new KafkaJaasLoginModuleInitializer();
		Jaas jaasProperties = this.properties.getJaas();
		if (jaasProperties.getControlFlag() != null) {
			jaas.setControlFlag(jaasProperties.getControlFlag());
		}
		if (jaasProperties.getLoginModule() != null) {
			jaas.setLoginModule(jaasProperties.getLoginModule());
		}
		jaas.setOptions(jaasProperties.getOptions());
		return jaas;
	}

	@Bean
	@ConditionalOnMissingBean
	public KafkaAdmin kafkaAdmin() {
		KafkaAdmin kafkaAdmin = new KafkaAdmin(this.properties.buildAdminProperties());
		kafkaAdmin.setFatalIfBrokerNotAvailable(this.properties.getAdmin().isFailFast());
		return kafkaAdmin;
	}

}
```

yml配置文件中的属性和属性类绑定有两种方式：

1. 使用`@ConfigurationProperties(prefix = "spring.datasource.mysql")注解。`
2. 使用`@Value("${spring.datasource.mysql.url}")`注解。

# 3、容器功能相关的注解

通过相应的注解，实现配置文件中的容器功能。

## 3.1、组件添加

### @Configuration

@Configuration **声明当前类为配置类**。使当前类的作用等同于一个**xml配置文件**。配置类本身也是组件，所以默认也是**单实例**的，通过配置类给容器中添加组件。

> [https://www.cnblogs.com/timetriesall/p/15735101.html](https://www.cnblogs.com/timetriesall/p/15735101.html)

注意：默认Spring容器中所有bean都是单例的；

- 优点：可以**节省空间，减少资源浪费**。
- 缺点：可能会**引发线程安全**问题

如果在Bean标签上设置scope = “prototype”,当前bean对象就是多例的,每次获取当前类的实例，spring容器就会创建当前类的实例；

- 优点：不会引发线程安全问题
- 缺点：每次获取实例都会创建新的实例，会占用服务器的内存空间，造成浪费

```xml
<bean 
	id="helloWorld" 
	class="cn.edu.initdestroy.HelloWorld" 
	scope="prototype" 
	init-method="init" 
	destroy-method="destroy">
</bean>
```

注解中，在@Controller、@Service、@Repository（或@Component）注解旁加 @Scope= (scopeName="prototype")  ，即多例，默认为单例。

- proxyBeanMethods属性：

spring5.2版本后，新增 proxyBeanMethods 属性。

springboot2.0版本基于此，@Configuration新增了一个注解属性proxyBeanMethods。

- 默认值true：表示**是否对当前配置的bean下的方法开启代理**。
    - **被代理的方法每次调用前都会经过代理对象，会检查容器中是否已经有对应实例，如果有，直接返回。可保持容器中组件是单实例**。
    - 每次通过当前类的实例方法去获取的实例都是同一个单例。
    - 此时容器中的MyConfig实例，是被SpringCGLIB增强的代理对象。com.learn.boot.config.MyConfig$$EnhancerBySpringCGLIB$$fdb349b1@19542407
- false时：每次通过当前类的实例的方法去获取的实例都是新的实例。
    - 此时容器中的MyConfig实例就是普通实例。
    - com.learn.boot.config.MyConfig@1d8e2eea
- 由proxyBeanMethods，衍生出两种模式：
    - Full，全配置。即 proxyBeanMethods = true，保证外部无论对配置类中每个@Bean方法对应的组件调用多少次，获取的都是容器中的同一个组件，即单实例对象。
    - Lite，轻量级配置。即 proxyBeanMethods = false，容器中保存的不是当前配置类的代理对象，而是一个普通的当前配置类的实例对象。配置类中，每个给容器添加组件的方法，每次被调用，都会去产生新的实例对象。
- 两种模式解决组件依赖的场景使用区别：
    - 场景1：如果配置类中添加组件的方法之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
    - 场景2：如果配置类中添加组件的方法之间有依赖关系，用Full模式，方法会被调用得到之前单实例组件，

配置类案例：

```java
package com.learn.boot.config;

import com.learn.boot.bean.Pet;
import com.learn.boot.bean.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Configuration 声明当前类为配置类。作用等同于一个xml配置文件
 *      配置类本身也是一个组件，所以默认也是单实例的
 *      proxyBeanMethods，表示是否对当前配置的bean下的方法开启代理，被代理的方法每次调用前都会经过代理检查容器中是否已经有对应实例，如果有，直接返回。
 *           默认为true。此时容器中的MyConfig实例，是被SpringCGLIB增强的代理对象。com.learn.boot.config.MyConfig$$EnhancerBySpringCGLIB$$fdb349b1@19542407
 *               每次通过当前类的实例方法去获取的实例都是同一个单例。
 *               默认为true的目的就是保持容器中组件是单实例。
 *           改为false的话，此时容器中的MyConfig实例就是普通实例。 com.learn.boot.config.MyConfig@1d8e2eea
 *              每次通过当前类的实例的方法去获取的实例都是新的实例。
 *       spring5.2版本后，新增 proxyBeanMethods 属性。
 *       基于此，springboot2.0版本后，@Configuration新增了一个注解proxyBeanMethods
 *
 *       Full,全配置。即 proxyBeanMethods = true
 *          配置类中，每个给容器添加组件的方法，每次被调用，都会去容器中找同一个组件。
 *       Lite，轻量级配置。即 proxyBeanMethods = false
 *          容器中保存的不是当前配置类的代理对象，而是一个普通的实例对象。
 *          配置类中，每个给容器添加组件的方法，每次被调用，都会去产生新的实例对象。
 *       用于解决组件之间有依赖的场景。
 *
 *       使用场景：
 *              如果配置的类组件之间无依赖关系，用Lite模式，加速容器启动过程，减少判断。
 *              如果配置的类组件之间有依赖关系，用Full模式，方法会被调用得到之前单实例组件，
 *
 */
@Configuration(proxyBeanMethods = true)
public class MyConfig {

    /**
     * @Bean的作用：配置类里面使用@Bean标注在方法上，给容器中注册组件。
     * 组件默认也是单实例的
     * 以方法名作为组件的ID。
     * 返回类型就是组件类型。
     * 返回的值，就是组件在容器中的实例
     * 等同于：
     * <bean id="haha" class="com.learn.boot.bean.User">
     *         <property name="name" value="zhangsan"></property>
     *         <property name="age" value="18"></property>
     *     </bean>
     * 外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @Bean("BigTom") 可以重新定义组件的ID
     */
    @Bean
    public User createUser(){
        User zhangsan = new User("zhangsan", 18,null);
        //zhangsan.setPet(new Pet("tom")); // 开启proxyBeanMethods,默认为true时，这里返回的却是一个新的实例。因为是强制自行new，不是经过被代理的方法创建的。
        zhangsan.setPet(createPet());
        return zhangsan;
    }

    @Bean("BigTom")
    public Pet createPet(){
        return new Pet("tom");
    }
}
```

测试类

```java
package com.learn.boot;

import com.learn.boot.bean.Pet;
import com.learn.boot.bean.User;
import com.learn.boot.config.MyConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;

/**
 * 主程序类/主配置类：标注了@SpringBootApplication注解的类。此类有一个即可。
 *
 * @SpringBootApplication：告诉spring这是一个SpringBoot应用
 * 如果有类在主程序外层的文件夹下，可以通过scanBasePackages将默认扫描路径范围设置的更外层些。
 * 方法一：
 *     @SpringBootApplication(scanBasePackages="com.learn")
 *     public class MainApplication {...}
 *
 * 方法二：
 * 	通过@ComponentScan 指定扫描路径。
 *
 *     @SpringBootConfiguration
 *     @EnableAutoConfiguration
 *     @ComponentScan("com.learn")
 *     public class MainApplication {..}
 *
 */
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.learn")
public class MainApplication {
    public static void main(String[] args) {
        // 项目启动，告诉spring加载主配置类。
        //SpringApplication.run(MainApplication.class, args);

        // ConfigurableApplicationContext IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        // 查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }

        // 从容器中获取组件
        // 测试从容器中获取多次类的实例，是否是单例
        Pet pet01 = run.getBean("BigTom", Pet.class);
        Pet pet02 = run.getBean("BigTom", Pet.class);
        System.out.println("注册的组件是单例：" + (pet01 == pet02));
        MyConfig config01 = run.getBean(MyConfig.class);
        MyConfig config02 = run.getBean(MyConfig.class);
        System.out.println("配置类也是是单例：" + (config01 == config02));

        /**
         * 测试通过容器中config配置类的实例的方法，在proxyBeanMethods设置为true和false时，去获取多次实例，检测是否是单例
         *
         * 当配置类声明了@Configuration(proxyBeanMethods = true)
         *      容器中配置类的实例就是被SpringCGLIB增强的代理对象。此时调用方法创建bean时，SpringBoot总会检查方法要返回的组件是否在容器中已创建，有的话就返回。
         * 当配置类声明了@Configuration(proxyBeanMethods = false)
         *       容器中配置类的实例就是本身类型，此时调用方法创建bean时，每次都重新创建新的bean。
         */
        System.out.println(config01);
        User user01 = config01.createUser();
        System.out.println("user01:"+user01);
        User user02 = config01.createUser();
        System.out.println("user02:"+user02);
        System.out.println("默认情况下，通过config配置类的方法去获取多次实例，也是单例：" + (user01 == user02));

        System.out.println("检查获取User实例中的pet，是否是容器的pet："+ (pet01 == user01.getPet()));

    }

}

注册的组件是单例：true
配置类也是是单例：true
com.learn.boot.config.MyConfig$$EnhancerBySpringCGLIB$$f7031a2d@6d868997
user01:com.learn.boot.bean.User@2c383e33
user02:com.learn.boot.bean.User@2c383e33
默认情况下，通过config配置类的方法去获取多次实例，也是单例：true
检查获取User实例中的pet，是否是容器的pet：true
```

### @Bean

@Bean的作用：@Configuration声明的配置类中，**使用@Bean标注在方法上，给容器中注册组件**。

```java
/**
     * @Bean的作用：**配置类里面使用@Bean标注在方法上，给容器中注册组件**。
     * 默认创建的bean，在容器的名称为方法名称。
     * 组件默认也是单实例的
     * 以方法名作为组件的ID。
     * 返回类型就是组件类型。
     * 返回的值，就是组件在容器中的实例
     * 等同于：
     * <bean id="haha" class="com.learn.boot.bean.User">
     *         <property name="name" value="zhangsan"></property>
     *         <property name="age" value="18"></property>
     *     </bean>
     * 外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @Bean("BigTom") 可以重新定义组件的ID
     */
    @Bean
    public User createUser(){
        User zhangsan = new User("zhangsan", 18,null);
        //zhangsan.setPet(new Pet("tom")); // 开启proxyBeanMethods，这里返回的却是一个新的实例。因为不是经过被代理的方法创建的。
        zhangsan.setPet(createPet());
        return zhangsan;
    }
```

### @Component、@Controller、@Service、@Repository

@Component：标注在类上，表示当前类是一个组件。等同于配置文件中的 <bean id="" class=""/>

@Controller：标注在Controller类上，表示当前类是一个控制器。

@Service：标注在Service类上，表示当前类是一个业务逻辑组件。

@Repository：**标注在Dao类上**，表示当前类是一个数据库层组件。

**@SpringBootConfiguration和@Configuration**

- @SpringBootConfiguration和@Configuration都是Spring框架中的注解，用于配置和定义应用程序的配置类。
- @SpringBootConfiguration注解是@SpringBootApplication注解的元注解之一，用于指示该类是Spring Boot的配置类。配置类是一个专门用于配置和组织Bean的类。
- @Configuration注解是Spring框架中的注解，用于标识一个类是配置类。配置类主要用于定义Bean的创建和配置，以提供给Spring容器进行管理。
- @SpringBootConfiguration继承自@Configuration，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到spring容器中，并且实例名就是方法名。（springboot推荐用@SpringBootConfiguration）

**@EnableAutoConfiguration和 @Configuration 的区别**

@Configuration：表示作用的类是个配置类。我们平时也会写配置类，比如我们系统中的DataSourceConfig类，但是由于这个类是在Starter对应的子目录下，会自动加载，所以@EnableAutoConfiguration就作用不到了。

@EnableAutoConfiguration：是一个加载Starter目录包之外的需要Spring自动生成bean对象（是否需要的依据是"META-INF/spring.factories"中org.springframework.boot.autoconfigure.EnableAutoConfiguration后面是有能找到那个bean）的带有@Configuration注解的类。一般就是对各种引入的spring-boot-starter依赖包指定的（spring.factories）类进行实例化。

@SpringBootApplication中包含了@EnableAutoConfiguration注解，@EnableAutoConfiguration的作用是启用Spring的自动加载配置。

整体流程：

1、我们启动类中会加上@SpringBootApplication

2、@SpringBootApplication包含了@EnableAutoConfiguration，告知应用启动的时候需要扫描依赖包中需要实例化的类

3、Springboot启动的时候会去扫描META-INF/spring.factories，查看具体是哪些类需要实例化

4、扫描哪些需要实例化的类，看下是否有@Configuration注解，如果有，则实例化

5、实例化的时候可能需要一些配置属性，一般这些类都会加上@EnableConfigurationProperties(RocketMQProperties.class)

### @ComponentScan

声明在主程序类（又称主配置类）上，指定包扫描路径，不配置时，默认扫描当前主程序类所在的包下的所有类。

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.learn")
public class MainApplication {
  ..
}
```

### @Import

给容器中导入组件。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044808.png)

声明在容器中的组件类上，给容器中自动创建出指定类型的组件。

默认组件bean的名称默认为类的全类名。com.learn.boot.bean.User

@Import 高级用法： [https://www.bilibili.com/video/BV1gW411W7wy?p=8](https://www.bilibili.com/video/BV1gW411W7wy?p=8)

```java
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = true)
public class MyConfig {
    ...
}
```

```java
package com.learn.boot;

import ch.qos.logback.core.db.DBHelper;
import com.learn.boot.bean.Pet;
import com.learn.boot.bean.User;
import com.learn.boot.config.MyConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication(scanBasePackages="com.learn")
public class MainApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        MyConfig config01 = run.getBean(MyConfig.class);

        System.out.println(config01);
        User user01 = config01.createUser();

        // 测试@Import注解
        // getBeanNamesForType 获取容器中类型为指定类的bean
        System.out.println("=======测试@Import注解的作用===========");
        String[] userBeans = run.getBeanNamesForType(User.class);
        for (String s : userBeans) {
            System.out.println("User类型的getBeanNamesForType方法结果 : " + s);
        }
        String[] dBHelperBeans = run.getBeanNamesForType(DBHelper.class);
        for (String s : dBHelperBeans) {
            System.out.println("DBHelper类型的getBeanNamesForType方法结果 : " + s);
        }
        // getBean 获取容器中指定类型的单实例bean，如果还有多实例，会报错。
        // No qualifying bean of type 'com.learn.boot.bean.User' available: expected single matching bean but found 2: com.learn.boot.bean.User,createUser
        //User userBean = run.getBean(User.class);
        //System.out.println(userBean);
        DBHelper dbHelperBean = run.getBean(DBHelper.class);
        System.out.println("DBHelper类型的getBean方法结果 : " + dbHelperBean);

    }

}

=======测试@Import注解的作用===========
User类型的getBeanNamesForType方法结果 : com.learn.boot.bean.User
User类型的getBeanNamesForType方法结果 : createUser
DBHelper类型的getBeanNamesForType方法结果 : ch.qos.logback.core.db.DBHelper
DBHelper类型的getBean方法结果 : ch.qos.logback.core.db.DBHelper@4e6d7365
```

### @Conditional

条件装配：满足Conditional指定的条件，则进行组件注入

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044161.png)

常见派生注解有：

- @ConditionalOnBean  当容器中存在指定的bean时
- @ConditionalOnMissingBean  当容器中不存在指定的bean时
- @ConditionalOnClass  当容器中存在某个类时
- @ConditionalOnMissingClass  当容器中不存在某个类时
- @ConditionalOnResource  当项目类路径下存在某个资源时
- @ConditionalOnJava  当前使用的Java为指定版本号时
- @ConditionalOnWebApplication  当前应用为Web应用时
- @ConditionalOnNotWebApplication  当前应用不是Web应用时
- @ConditionalOnSingleCandidate  当前容器中，指定的组件只有一个实例，或者，组件存在多个实例，但只有一个实例为主实例（@Primary标注）时
- 以@ConditionalOnBean为例

```java
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = true)
public class MyConfig {

    @ConditionalOnBean(name = "BigTom") // 当容器中存在名称为BigTom的bean时，才会触发生成名为createUser的组件
    @Bean
    public User createUser() {
        User zhangsan = new User("zhangsan", 18, null);
        //zhangsan.setPet(new Pet("tom")); // 开启proxyBeanMethods，这里返回的却是一个新的实例。因为不是经过被代理的方法创建的。
        zhangsan.setPet(createPet());
        return zhangsan;
    }

    //@Bean("BigTom")
    public Pet createPet() {
        return new Pet("tom");
    }
}
```

```java
package com.learn.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication(scanBasePackages="com.learn")
public class MainApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        // 查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
        System.out.println("=====测试@ConditionalOnBean=====");
        boolean bigTom = run.containsBean("BigTom");
        System.out.println("容器中BigTom组件："+bigTom);

        boolean createUser = run.containsBean("createUser");
        System.out.println("容器中createUser组件："+createUser);

    }

}

=====测试@ConditionalOnBean=====
容器中BigTom组件：false
容器中createUser组件：false
```

## 3.2、原生配置文件引入

### @ImportResource

如果项目中存在spring配置文件，可能是老项目迁移遗留、或者引入的第三方包比较老，用的是配置文件而不是注解。这些配置文件在springboot项目中默认是不生效的。

如果不想手动将配置文件中的组件声明，手写转化为配置类，这时可以用@ImportResource。

用法：在任意配置类上，指定配置文件路径，启动时会将文件中的组件配置加载到容器中。

MyConfig.java

```java
@ImportResource("classpath:beans.xml")
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = true)
public class MyConfig {
    ...
}
```

bean.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  
    <bean id="haha" class="com.learn.boot.bean.User">
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
    </bean>

    <bean id="hehe" class="com.learn.boot.bean.Pet">
        <property name="name" value="tomcat"></property>
    </bean>
</beans>
```

MainApplication.java

```java
package com.learn.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication(scanBasePackages="com.learn")
public class MainApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        System.out.println("=====测试@ImportResource=====");
        boolean haha = run.containsBean("haha");
        boolean hehe = run.containsBean("hehe");
        System.out.println("haha："+haha);
        System.out.println("hehe："+hehe);
    }

}

=====测试@ConditionalOnBean=====
haha：true
hehe：true
```

## 3.3、properties配置文件内容绑定

Java中，读取properties文件中的内容，封装至JavaBean中，用原生代码实现比较繁琐，如：

```java
public class getProperties {
     public static void main(String[] args) throws FileNotFoundException, IOException {
         // 1 读取配置文件
         Properties pps = new Properties();
         pps.load(new FileInputStream("a.properties"));
         Enumeration enum1 = pps.propertyNames();//得到配置文件的名字
         // 2 遍历里面的kv值，
         while(enum1.hasMoreElements()) {
             String strKey = (String) enum1.nextElement();
             String strValue = pps.getProperty(strKey);
             System.out.println(strKey + "=" + strValue);
             // 3 匹配到符合指定规则的KV，封装至对应的JavaBean。
         }
     }
 }
```

springboot简化了这个过程，叫做**配置绑定**。

### 3.3.1、@ConfigurationProperties

如果组件的属性值想要和核心配置文件application.properties中的KV进行绑定，则需要在组件类上加上该注解。

组件必须被声明在容器中，该注解才能生效。

该组件在springboot底层中经常被使用。

有两种使用方法：

### 3.3.2、**方法一：@Component + @ConfigurationProperties**

application.properties

```java
mycar.brand=YD
mycar.price=100000
```

Car.java

```java
package com.learn.boot.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * 针对已经在容器中的组件，
 * 通过添加@ConfigurationProperties注解，自动将配置文件中的指定KV，映射到类的属性值中。
 * prefix 指定当前类中的属性的前缀，对应配置文件中指定前缀下的属性进行绑定。
 */
@Component
@ConfigurationProperties(prefix = "mycar")//prefix ：表示配置文件中prefix 前缀的属性和
//这个类中的属性进行绑定
public class Car {

    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                '}';
    }
}
```

测试

```java
package com.learn.boot;

import ch.qos.logback.core.db.DBHelper;
import com.learn.boot.bean.Car;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication(scanBasePackages="com.learn")
public class MainApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);

        System.out.println("=====测试@ConfigurationProperties=====");
        Car car = run.getBean(Car.class);
        System.out.println(car.toString());

    }
}
```

### 3.3.3、方法二：@EnableConfigurationProperties + @ConfigurationProperties

假设引用了第三方包中的组件，我们不可能在其源代码上声明@Component.

这时可通过在配置类上声明@EnableConfigurationProperties并指定组件。来实现指定组件的配置绑定。

@EnableConfigurationProperties

作用：

1、开启指定组件的属性配置功能，使得组件上的@ConfigurationProperties注解生效

2、将指定组件导入至容器中。（等同于在组件上添加了@Component）

```java
/**
 * @ImportResource("classpath:beans.xml")导入Spring的配置文件，
 *
 * @EnableConfigurationProperties
 *   作用：
 *   1、开启指定组件的属性配置功能，使得组件上的@ConfigurationProperties注解生效
 *   2、将指定组件导入至容器中。（等同于在组件上添加了@Component）
 *
 */
@ImportResource("classpath:beans.xml")
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = true)
@EnableConfigurationProperties(Car.class)  // 开启指定组件的属性配置功能
public class MyConfig {
    ...
}
```

# 4、自动配置原理

按执行步骤角度讲解，分为以下几步

## 4.1、引导加载自动配置类

主启动类上，@SpringBootApplication，实际封装了

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

### 4.1.1、@SpringBootConfiguration

@SpringBootConfiguration 中，集成了 @Configuration，作用是代表当前是一个配置类。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
```

### 4.1.2、@ComponentScan

指定扫描哪些包。

比较特殊的是，这里指定了自定义的扫描器。

```java
@ComponentScan(excludeFilters = { 
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
	@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) 
})
```

### 4.1.3、@EnableAutoConfiguration （核心）

为三者中最核心的注解，

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

**@AutoConfigurationPackage**

- 作用：将注解实际作用的类所在的包下的所有组件导入进来。

例如 @SpringBootApplication 合成了  @EnableAutoConfiguration、@EnableAutoConfiguration 又合成了  @AutoConfigurationPackage。所以@SpringBootApplication 具备了 @AutoConfigurationPackage 的功能，即，@SpringBootApplication可将主启动类所在的包下的所有组件导入进来。

自动配置包，指定了包导入规则。

该注解通过@Import导入了AutoConfigurationPackages.Registrar.class组件，

利用 AutoConfigurationPackages.Registrar，将该注解实际作用于的目标类所在的包路径下的所有组件批量导入进容器中。这里实际为MainApplication 所在的包路径。

```java
// @Import 给容器中导入一个组件
// 
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
```

AutoConfigurationPackages.Registrar 批量注册原理：

AutoConfigurationPackages 为抽象类，

AutoConfigurationPackages.Registrar 为AutoConfigurationPackages  中声明的静态类。

由下图AutoConfigurationPackages.Registrar.class 中的Registrar断点可以分析出来：

AnnotationMetadata metadata：代表注解（当前为 @AutoConfigurationPackage）的元信息，包含该注解被实际使用的位置（MainApplication）、及各个属性值信息。

register方法：获取注解元信息中，注解作用于的目标类所在的包路径。将指定包路径下的所有组件进行注册。实际为主程序类MainApplication所在文件夹内的所有组件。这也就是springboot默认包路径为**主程序类**所在文件夹的原因。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044005.png)

**@Import(AutoConfigurationImportSelector.class)**

作用：加载项目中所有./META-INF/spring.factories 的文件内容，将文件中声明的组件信息加载到容器中。（但并非全部生效，仍需按照条件装配规则（@Conditional）按需配置。具体见下方“按需开启自动配置项”章节讲解。）

主要运行过程如下：

```java
# 描述方式一：执行步骤描述

1、利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
3、利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4、从META-INF/spring.factories位置来加载一个文件。
	默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
    spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories

# 描述方式二：执行调用描述

--AutoConfigurationImportSelector.java
	调用方法 selectImports
 	调用方法 getAutoConfigurationEntry ：给容器中批量导入一些组件
  	调用方法 getCandidateConfigurations：获取到所有需要导入到容器中的配置类
	--SpringFactoriesLoader.java
		调用方法 loadFactoryNames：利用工厂加载得到所有的组件
    	调用方法 loadSpringFactories：加载所有依赖jar包中的META-INF/spring.factories文件，收集声明的组件包路径并整理。例如spring-boot-autoconfigure-2.3.4.RELEASE.jar中META-INF/spring.factories文件里面，# Auto Configure注释下写死了尾缀为xxxAutoConfiguration配置清单,声明了支持的自动配置相关的包。数量有127个。
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044194.png)

以自动配置组件加载为例：
spring-boot一启动就要给容器中加载的所有配置类（但不一定都生效，具体过程见下方“按需开启自动配置项”），
例如在spring-boot-autoconfigure-2.3.4.RELEASE.jar 中META-INF/spring.factories文件里面写死了支持的自动配置清单，即# Auto Configure注释下，尾缀为xxxAutoConfiguration，自动配置相关的包。数量有127个。

## 4.2、按需开启自动配置项

如上@EnableAutoConfiguration生效后，虽然我们会将所有jar包中的META-INF/spring.factories下的配置全部加载，但并非全部生效，仍需按照条件装配规则（@Conditional）按需配置。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211044372.png)

常见的条件注解：

[https://blog.csdn.net/qq_36850813/article/details/101597330](https://blog.csdn.net/qq_36850813/article/details/101597330)

| 条件注解                        | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| @ConditionalOnBean              | Spring容器中是否存在对应的实例。可以通过实例的类型、类名、注解、昵称去容器中查找(可以配置从当前容器中查找或者父容器中查找或者两者一起查找)这些属性都是数组，通过”与”的关系进行查找 |
| @ConditionalOnClass             | 类加载器中是否存在对应的类。可以通过Class指定(value属性)或者Class的全名指定(name属性)。如果是多个类或者多个类名的话，关系是”与”关系，也就是说这些类或者类名都必须同时在类加载器中存在 |
| @ConditionalOnExpression        | 判断SpEL 表达式是否成立                                      |
| @ConditionalOnJava              | 指定Java版本是否符合要求。内部有2个属性value和range。value表示一个枚举的Java版本，range表示比这个老或者新于等于指定的Java版本(默认是新于等于)。内部会基于某些jdk版本特有的类去类加载器中查询，比如如果是jdk9，类加载器中需要存在java.security.cert.URICertStoreParameters；如果是jdk8，类加载器中需要存在java.util.function.Function；如果是jdk7，类加载器中需要存在java.nio.file.Files；如果是jdk6，类加载器中需要存在java.util.ServiceLoader |
| @ConditionalOnMissingBean       | Spring容器中是否缺少对应的实例。可以通过实例的类型、类名、注解、昵称去容器中查找(可以配置从当前容器中查找或者父容器中查找或者两者一起查找)这些属性都是数组，通过”与”的关系进行查找。还多了2个属性ignored(类名)和ignoredType(类名)，匹配的过程中会忽略这些bean |
| @ConditionalOnMissingClass      | 跟ConditionalOnClass的处理逻辑一样，只是条件相反，在类加载器中不存在对应的类 |
| @ConditionalOnWebApplication    | 应用程序是否是Web程序，没有提供属性，只是一个标识。会从判断Web程序特有的类是否存在，环境是否是Servlet环境，容器是否是Web容器等 |
| @ConditionalOnNotWebApplication | 应用程序是否是非Web程序，没有提供属性，只是一个标识。会从判断Web程序特有的类是否存在，环境是否是Servlet环境，容器是否是Web容器等 |
| @ConditionalOnProperty          | 应用环境中的屬性是否存在。提供prefix、name、havingValue以及matchIfMissing属性。prefix表示属性名的前缀，name是属性名，havingValue是具体的属性值，matchIfMissing是个boolean值，如果属性不存在，这个matchIfMissing为true的话，会继续验证下去，否则属性不存在的话直接就相当于匹配不成功 |
| @ConditionalOnResource          | 是否存在指定的资源文件。只有一个属性resources，是个String数组。会从类加载器中去查询对应的资源文件是否存在 |
| @ConditionalOnSingleCandidate   | Spring容器中是否存在且只存在一个对应的实例。只有3个属性value、type、search。跟ConditionalOnBean中的这3种属性值意义一样 |

实际例子：

### 4.2.1、AopAutoConfiguration

引入springboot2核心包后，默认生效

```sql
@Configuration(proxyBeanMethods = false) // 声明当前类是个配置类，proxyBeanMethods = false 每个被@Bean注解的方法被调用多少次返回的组件都是新创建的
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true) // 判断配置文件中是否存在前缀为spring.aop，名称为auto的配置，且值为true，matchIfMissing=true表示忽略当前配置，就算该配置没配，也认为该配置存在了。
public class AopAutoConfiguration {
    
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass(Advice.class) // 项目中是否存在指定类
	static class AspectJAutoProxyingConfiguration {

		@Configuration(proxyBeanMethods = false)
		@EnableAspectJAutoProxy(proxyTargetClass = false)
		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false",
				matchIfMissing = false)
		static class JdkDynamicAutoProxyConfiguration {

		}

		@Configuration(proxyBeanMethods = false)
		@EnableAspectJAutoProxy(proxyTargetClass = true)
		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
				matchIfMissing = true)
		static class CglibAutoProxyConfiguration {

		}

	} 
    
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnMissingClass("org.aspectj.weaver.Advice") // 当目标类在环境中不存在
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
			matchIfMissing = true) // 配置文件中是否在前缀为spring.aop，名称为proxy-target-class，值为true的配置。就算该配置不存在，也认为存在。
	static class ClassProxyingConfiguration {

		ClassProxyingConfiguration(BeanFactory beanFactory) {
			if (beanFactory instanceof BeanDefinitionRegistry) {
				BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
				AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
		}

	}

}
```

### 4.2.2、CacheAutoConfiguration

```sql
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(CacheManager.class) // 环境中是否存在CacheManager类
@ConditionalOnBean(CacheAspectSupport.class) // 容器中是否存在CacheAspectSupport类型的组件
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureAfter({ CouchbaseDataAutoConfiguration.class, HazelcastAutoConfiguration.class,
		HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })
@Import({ CacheConfigurationImportSelector.class, CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })
public class CacheAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public CacheManagerCustomizers cacheManagerCustomizers(ObjectProvider<CacheManagerCustomizer<?>> customizers) {
		return new CacheManagerCustomizers(customizers.orderedStream().collect(Collectors.toList()));
	}
```

### 4.2.3、DispatcherServletAutoConfiguration

引入 spring-boot-starter-web 依赖后生效。

```sql
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE) // 声明当前配置类的配置顺序为优先生效
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET) // 判断当前是否是WEB应用，并且为原生SERVLET类型的WEB应用。springboot2支持两种WEB模式：原生WEB技术栈与响应式编程。
@ConditionalOnClass(DispatcherServlet.class) // 判断是否存在DispatcherServlet类。导入了 spring-boot-starter-web 依赖，会自动引入spring-webmvc，DispatcherServlet也会被引入。
@AutoConfigureAfter(ServletWebServerFactoryAutoConfiguration.class) // 等待 ServletWebServerFactoryAutoConfiguration 配置完成后再配置
public class DispatcherServletAutoConfiguration {

	@Configuration(proxyBeanMethods = false) 
	@Conditional(DefaultDispatcherServletCondition.class) // 指定 DefaultDispatcherServletCondition 对应的条件生效
	@ConditionalOnClass(ServletRegistration.class)
	@EnableConfigurationProperties(WebMvcProperties.class) // 1、开启 WebMvcProperties 与配置文件的配置绑定功能。2、WebMvcProperties与配置文件绑定后，放到容器中。
	protected static class DispatcherServletConfiguration {
        
		//这里框架直接new一个实例然后放到容器中。使用者不用再去配置文件中声明bean或自行创建bean。
		@Bean(name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME) 
		public DispatcherServlet dispatcherServlet(WebMvcProperties webMvcProperties) {  
			DispatcherServlet dispatcherServlet = new DispatcherServlet();
			dispatcherServlet.setDispatchOptionsRequest(webMvcProperties.isDispatchOptionsRequest());
			dispatcherServlet.setDispatchTraceRequest(webMvcProperties.isDispatchTraceRequest());
			dispatcherServlet.setThrowExceptionIfNoHandlerFound(webMvcProperties.isThrowExceptionIfNoHandlerFound());
			dispatcherServlet.setPublishEvents(webMvcProperties.isPublishRequestHandledEvents());
			dispatcherServlet.setEnableLoggingRequestDetails(webMvcProperties.isLogRequestDetails());
			return dispatcherServlet;
		}

        // MultipartResolver 文件上传解析器
        // 该方法作用：如果容器中存在MultipartResolver类型的bean，但是不存在名称是multipartResolver的组件，该方法就将名称改为multipartResolver。防止有些用户配置的文件上传解析器不符合规范。springmvc文件上传解析器MultipartResolver初始化的时候是通过唯一的id去容器中找有没有文件上传的组件。
		@Bean
		@ConditionalOnBean(MultipartResolver.class) // 容器中存在对应类型的组件
		@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) // 容器中不存在对应名称的组件
		public MultipartResolver multipartResolver(MultipartResolver resolver) {  //给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找。
			// Detect if the user has created a MultipartResolver but named it incorrectly
			return resolver;
		}

	}
```

WebMvcProperties.java

```sql
@ConfigurationProperties(prefix = "spring.mvc") // 当与配置文件做属性绑定时，对应属性的前缀为 spring.mvc
public class WebMvcProperties {

	/**
	 * Formatting strategy for message codes. For instance, `PREFIX_ERROR_CODE`.
	 */
	private DefaultMessageCodesResolver.Format messageCodesResolverFormat;

	/**
	 * Locale to use. By default, this locale is overridden by the "Accept-Language"
	 * header.
	 */
	private Locale locale;

	/**
	 * Define how the locale should be resolved.
	 */
	private LocaleResolver localeResolver = LocaleResolver.ACCEPT_HEADER;

	private final Format format = new Format();
```

### 4.2.4、HttpEncodingAutoConfiguration

引入 spring-boot-starter-web 依赖后生效。

通过属性与配置文件的绑定，最终过滤器设置编码就支持从配置文件中获取。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045538.png)

```sql
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ServerProperties.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(CharacterEncodingFilter.class) // 判断环境中是否存在CharacterEncodingFilter类。只要导入了springmvc的依赖，就会存在该类。
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true) // 判断配置文件中是否存在前缀为server.servlet.encoding、值为enabled的属性。 matchIfMissing = true，表示忽略配置文件中的改配置，即使不配也算配了。
public class HttpEncodingAutoConfiguration {

	private final Encoding properties;

	public HttpEncodingAutoConfiguration(ServerProperties properties) {
		this.properties = properties.getServlet().getEncoding();
	}
    
    
	// CharacterEncodingFilter ，springmvc中用来解决请求编码问题的类。
	@Bean
	@ConditionalOnMissingBean  // 容器中如果没有配置 CharacterEncodingFilter这个bean，该方法再执行。
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
		return filter;
	}
```

## 3.3、框架支持根据情况按需调整当前的配置

案例一：web 项目里，用户创建了文件上传解析器MultipartResolver，但是名称不是multipartResolver。使用条件注解将bean名称更改。

springmvc文件上传解析器MultipartResolver初始化的时候是通过唯一的id去容器中找有没有文件上传的组件。所以bean名称必须是multipartResolver。

```sql
...
public class DispatcherServletAutoConfiguration {
	...
	protected static class DispatcherServletConfiguration {

        // MultipartResolver 文件上传解析器
        // 该方法作用：如果容器中存在MultipartResolver类型的bean，但是不存在名称是multipartResolver的组件，该方法就将名称改为multipartResolver。防止有些用户配置的文件上传解析器不符合规范。springmvc文件上传解析器MultipartResolver初始化的时候是通过唯一的id去容器中找有没有文件上传的组件。
		@Bean
		@ConditionalOnBean(MultipartResolver.class) // 容器中存在对应类型的组件
		@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) // 容器中不存在对应名称的组件
		public MultipartResolver multipartResolver(MultipartResolver resolver) {  //给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找。
			// Detect if the user has created a MultipartResolver but named it incorrectly
			return resolver;
		}

	}
...
```

案例二：SpringBoot默认会在底层配好所有的组件。但是通过@ConditionalOnMissingBean注解，如果用户自己配置了，这时则以用户的优先。

当用户自定义了CharacterEncodingFilter组件配置。

```sql
@Configuration(proxyBeanMethods = true)
public class MyConfig {

    @Bean
    public CharacterEncodingFilter filter(){
        return null;
    }
}
```

这时框架的HttpEncodingAutoConfiguration自动配置类中不再自动配置CharacterEncodingFilter。

```sql
public class HttpEncodingAutoConfiguration {

    // CharacterEncodingFilter ，springmvc中用来解决请求编码问题的类。
    @Bean
	@ConditionalOnMissingBean // 容器中如果没有配置 CharacterEncodingFilter这个bean，该方法再执行。
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
		return filter;
	}
```

## 3.4、总结

```java
1、SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
2、每个自动配置类按照条件进行生效，每个配置类默认都会绑定配置文件中指定的值。
	例如：HttpEncodingAutoConfiguration配置类中，指定ServerProperties类与配置文件绑定后存入容器中，Encoding properties变量的值就从ServerProperties中获取。  
3、生效的配置类就会给容器中装配很多组件（@Bean），
4、只要容器中有这些组件，相当于具有了组件对应的功能
5、如果用户有自己的配置，就以用户的优先。定制化配置的方法：
	1、用户直接自己@Bean替换底层的组件。
	2、用户自行从官方文档或对应配置类源码中，获悉这个组件是获取的配置文件中的什么值，然后在配置文件中修改即可。参考可用配置列表：https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties
    
    
xxxxxAutoConfiguration给容器中导入了组件  ---> 组件会从xxxxProperties里面拿值  ----> xxxxProperties从application.properties获取配置值
```

可用配置列表：[https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045564.png)

### 3.4.1、简化部署

配置文件属性与配置类对应关系

以HttpEncodingAutoConfiguration为例

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045896.png)

## 3.5、最佳实践经验，开发步骤

### 3.5.1、要开发什么场景，就引入对应的场景依赖

假设要开发缓存功能，要引入缓存的场景依赖。要去找springboot官方或第三方提供的场景依赖。

如果是第三方，要查看第三方提供的说明文档。

如果是springboot官方，从官方文档提供的列表中可找到对应的场景依赖：[https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045342.png)

### 3.5.2、可查看springboot帮我们配置了哪些对应场景下的组件（可选）。

偏向于底层原理，开发期间可不管这些。

查看方式：

1、自行分析：根据引入场景的标识，去 spring-boot-autoconfigure-2.3.4.RELEASE.jar 中，寻找对应标识同名文件夹下的配置类，根据代码分析哪些配置会生效，一般都生效了。这种方式比较麻烦。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045571.png)

2、在application.properties配置文件中，开启debug模式，添加debug=true，启动时打印出自动配置报告。
Positive matches  生效的配置
Negative matches  未生效的配置

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045976.png)

### 3.5.3、引入依赖后，修改配置文件中对应的内容

如何得知引入场景依赖后要修改哪些配置项？有如下2种方式。

1、自行分析：根据引入场景的标识，去 spring-boot-autoconfigure-2.3.4.RELEASE.jar 中，寻找对应标识同名文件夹下的配置类，并根据绑定的配置属性类进行分析。

2、参考springboot官方或第三方依赖的文档。[https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045102.png)

以启动日志中的banner图为例。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045682.png)

### 3.5.4、可自定义添加组件或替换组件

如果引入的场景中的功能不符合要求，可自定义添加组件或替换组件。通过@Bean、@Component .....。

springboot2自动配置的原理规定了，如果用户自定义了一个目标组件，那么该组件的运作方式就以用户写的为准。

后面还会讲解**自定义器**，XXXCustomizer，我们可以自定义一些自定义器来改变已有组件的运行行为。

### 3.5.5、业务逻辑

# 5、辅助开发技巧

## 5.1、Lombok

Lombok可以简化JavaBean开发。在项目编译时自动生成属性的get/set方法,程序编译的时候会生成方法。

1. 引入lombok依赖

springboot的父项目中已经默认指定了lombok的版本。spring-boot-starter-parent --> spring-boot-dependencies  中，指定了默认版本： <lombok.version>1.18.12</lombok.version>

```xml
<dependency>
   <groupId>org.projectlombok</groupId>
   <artifactId>lombok</artifactId>
</dependency>
```

1. lombok的使用

```java
===============================简化JavaBean开发===================================
package com.learn.boot.bean;

import lombok.*;

@Data // lombok:编译时自动生成get/set方法。@Data相当于@Getter @Setter 
//@RequiredArgsConstructor @ToString @EqualsAndHashCode这5个注解的合集。
//@ToString // lombok:编译时自动生成tostring方法
//@EqualsAndHashCode // lombok:编译时此注解会生成equals(Object other) 和 hashCode()方法。
@AllArgsConstructor // lombok:编译时自动生成全量有参构造器,如果需要部分参数构造器，
//则不用该注解，直接自行手写。
@NoArgsConstructor // lombok:编译时自动生成无参构造器
public class User {
    private String name;
    private Integer age;
    private Pet pet;
//以后下面的内容都使用lambok生成 
    //public User() {
    //}

    //public User(String name, Integer age, Pet pet) {
    //    this.name = name;
    //    this.age = age;
    //    this.pet = pet;
    //}
    //
    //public Pet getPet() {
    //    return pet;
    //}
    //
    //public void setPet(Pet pet) {
    //    this.pet = pet;
    //}
    //
    //public String getName() {
    //    return name;
    //}
    //
    //public void setName(String name) {
    //    this.name = name;
    //}
    //
    //public Integer getAge() {
    //    return age;
    //}
    //
    //public void setAge(Integer age) {
    //    this.age = age;
    //}

}

================================简化日志开发===================================

package com.learn.boot.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Import;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

@RestController // 相当于 @ResponseBody + @Controller
@Slf4j // lombok:编译时自动生成日志记录器
public class HelloController {
	//会自动注入log对象
    @RequestMapping("/hello")
    public String handle01(){
        log.info("使用Lombok生成Info日志");
        return "Hello, Spring Boot 2!";
    }
}
```

## 5.2、dev-tools

开发者工具，实现项目编译后的快速启动。

原理：dev tools快速重启，即Restart，底层有两个类加载器，一个负责加载第三方库，一个负责加载本地java类。第三方库源码是不会变的，也就是重新启动时，只需要加载本地java类就可以。节省了重新启动的时间。

这个东西用的时候得注意会存在奇奇怪怪的问题，破坏双亲委派机制~

如果想实现不重启的热部署，即重载Reload，用付费的Jrebel。[https://blog.csdn.net/lianghecai52171314/article/details/105637251](https://blog.csdn.net/lianghecai52171314/article/details/105637251)

注：IDEA的编译Bulid Project 按钮是Ctrl+F9。ctrl+shift+F9， 这个很快只编译修改的文件。Shift+F10 直接重启项目

1. 使用方式

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

1. 官网说明

[https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045412.png)

## 5.3、**Spring Initailizr（项目初始化向导）**

方便快速初始化创建Springboot项目。

操作步骤：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211045188.png)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211046190.png)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211046992.png)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211046525.png)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211046883.png)

**效果**

自动帮我们创建好了项目结构依赖、文件夹、主配置类、配置文件。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211046223.png)

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202406211047968.png)