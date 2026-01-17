---
title: 1、如何自定义Springboot-Starter
icon: file
order: 1
author: bugcode
date: 2024-11-20T00:00:00.000Z
category:
  - 面试
tag:
  - 面试
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2026/01/17 19:25:45
permalink: /project-docs/project/bootStarter/springbootStarter/
---

# 自定义Starter

写在前面，本文先讲解一下springboot实现得starter组件机制，然后基于此原理实现一个自定义格式化日期starter组件，本文将从一下几方面开始：

- 为什么要使用starter小组件
- 常见的一些starter场景
- Mybatis-starter定义分析
- 开发starter组件的要素
- 实现日期格式化starter小组件

## 为什么使用starter组件

早期开发spring项目中，比如需要引入ORM框架，通常需要做以下三个步骤：

1. 到maven仓库去找需要引入的mybatis jar包，选取合适的版本。
2. 到maven仓库去找mybatis-spring整合的jar包，选取合适的版本。
3. 在spring的applicationContext.xml文件中配置dataSource和mybatis相关信息。

仅仅时引入一个组件，需要做兼容性调整，版本依赖选择和组件基本配置，试想如果需要引入多个组件，那每一个组件都需要去做相同的步骤，无形中增加了很多工作量，并且有很多重复性的工作；

因此springboot-starter的出现就是为了解决这个问题，通过各个组件的starter，在springboot中很方便就可以做到自动配置引入，使用者只需要在配置文件中做简单的功能配置，springboot会自动帮助我们下载依赖自动装配，省去了大量的重复工作。

其二，项目中通常有一些公共的小模块，如果对这种模块每一个大模块中重新引入一遍代码，无疑是一种空间浪费，并且增加了代码的复杂度，也不符合模块式开发。因此starter给我们提供了一种机制，我们可以将一些小功能模块，使用starter进行开发，最终提供给使用者仅仅是一个配置接口和starter组件，只需要pom中引入加以配置就可以使用，大大简化了我们项目的代码复杂度。

## 常见的一些starter开发场景

在我们的日常开发工作中，可能会需要开发一个通用模块，以供其它工程复用。SpringBoot就为我们提供这样的功能机制，我们可以把我们的通用模块封装成一个个starter，这样其它工程复用的时候只需要在pom中引用依赖即可，由SpringBoot为我们完成自动装配。

举一些常见场景：

- 通用模块-短信发送模块
- 基于AOP技术实现日志切面
- 分布式雪花ID，Long-->string，解决精度问题 jackson2/fastjson
- 微服务项目的数据库连接池配置
- 微服务项目的每个模块都要访问redis数据库，每个模块都要配置redisTemplate



## Mybatis-starter定义分析

在开始定义自己的starter之前，我们先来看下mybatis-starter是如何定义的，分析一下自定义starter的几个要素；

mybatis-starter定义如下：

![image-20241120110422196](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241120110422196.png)

- pom.properties 配置maven所需的项目version、groupId和artifactId
- pom.xml 配置所依赖的jar包
- MANIFEST.MF 这个文件描述了该Jar文件的很多信息
- spring.provides 配置所依赖的artifactId，给IDE使用的，没有其他的作用

可以看出，mybatis-starter定义中最关键的其实就是这个pom.xml文件了，pom中定义的依赖如下：

```java
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>
```

pom中定义了mybatis-starter组件所有的依赖，其中有一个关键的依赖：mybatis-spring-boot-autoconfigure，这个依赖主要决定了mybatis组件中一些关键的bean对象式如何北引入到ioc容器中的，因此接下来我们中点分析这个包下的内容；

![image-20241120110705778](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241120110705778.png)

- pom.properties 配置maven所需的项目version、groupId和artifactId
- pom.xml 配置所依赖的jar包
- additional-spring-configuration-metadata.json 手动添加IDE提示功能
- MANIFEST.MF 这个文件描述了该Jar文件的很多信息
- spring.factories SPI会读取的文件
- spring-configuration-metadata.json 系统自动生成的IDE提示功能
- ConfigurationCustomizer 自定义Configuration回调接口
- MybatisAutoConfiguration mybatis配置类
- MybatisProperties mybatis属性类
- SpringBootVFS 扫描嵌套的jar包中的类

首先我们看一下`MybatisProperties`这个类，类中定义了mybatis的一些核心配置，比如configLocation，typeAliasesPackage等我们常用的配置，其中一个`@ConfigurationProperties`注解表明springboot会自动帮助我们加载配置文件中关于mybatis的核心配置，前提式有关mybatis的配置要以mybatis前缀配置；

其次，MybatisAutoConfiguration类帮我们自动将配置文件中的配置加载到bean对象中，然后将创建好的bean对象注入到IOC容器中，有几个关键的注解需要关注：

```java
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration implements InitializingBean {}
```

- `Configuration`:表明这是一个配置类
- `ConditionalOnClass`:条件加载类，表示只有容器中存在`SqlSessionFactory.class, SqlSessionFactoryBean.class`两个类时才加载创建；
- `EnableConfigurationProperties`：开启自动配置
- `AutoConfigureAfter`:条件加载，只有当`DataSourceAutoConfiguration,MybatisLanguageDriverAutoConfiguration`加载后才能加载自动配置类；

最后，关键的两个对象：

```java
@Bean
  @ConditionalOnMissingBean
  public SqlSessionFactory sqlSessionFactory(DataSource dataSource){}

@Bean
  @ConditionalOnMissingBean
  public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory){}
```

springboot会根据我们的配置自动注入这两个对象到容器中。

spring-configuration-metadata.json 和  additional-spring-configuration-metadata.json  的功能差不多，我们再application.properties文件中输入spring时，会自动出现下面的配置信息可供选择，就是这个功能了。

最后，再来分析下spring.factories文件：

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
```

里面只有一行配置，即key为EnableAutoConfiguration，value为MybatisAutoConfiguration。

接下来，从上面的分析中，我们来推理下定义一个starter需要的最基本要素：

![image-20241120113130476](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241120113130476.png)

starter本身式一个jar包，jar包里面最重要的其实就是pom，pom中定义了starter依赖，其最重要的一个依赖就是xxxAutoConfiguration.jar，此包中定义了starter具体时如何被加载并且注入到容器中的。

因此，自定义starter通常需要如下几个步骤：

**1. 需要定义一个名称为xxx-spring-boot-starter的空项目，里面不包含任何代码，可以有pom.xml和pom.properties文件。**

**2. pom.xml文件中包含了名称为xxx**-spring-boot**-autoconfigure的项目**

**3. xxx**-spring-boot**-autoconfigure项目中包含了名称为xxxAutoConfiguration的类，该类可以定义一些bean实例。当然，Configuration类上可以打一些如：ConditionalOnClass、ConditionalOnBean、EnableConfigurationProperties等注解。**

**4. 需要在spring.factories文件中增加key为EnableAutoConfiguration，value为xxxAutoConfiguration。**



> 当然，我们自己定义starter时候，也可以将这两个项目合并为一个项目，在starter项目中集成自动加载配置内容，这样使用起来更方便；

## 自定义日期格式化starter

实现一个简单的日期格式化starter，包含两个功能配置：

```
formatter:
	enabled: true 是否开启其定义格式化配置
	pattern: yyyy-MM-dd HH:mm:ss.SSS  格式化的模式匹配
```

功能很简单，自定义是否允许用户开启此功能；和用户格式化模式串；

因此定义配置类：

### DateFormatProperties

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ConfigurationProperties("spring.formatter")
public class DateFormatProperties {

    /*日期格式*/
    private String pattern = "yyyy-MM-dd HH:mm:ss.SSS";
}
```

其次，需要将用户的自定义配置自动加载配置到容器中的bean对象，因此创建自动加载配置类：

### DateFormatAutoConfiguration

```java
@Slf4j
@Configuration
@EnableAutoConfiguration
@EnableConfigurationProperties(DateFormatProperties.class)
@ConditionalOnProperty(prefix = "spring.formatter",name = "enabled",havingValue = "true")
public class DateFormatAutoConfiguration {

    Logger logger = LoggerFactory.getLogger(DateFormatAutoConfiguration.class);

    private DateFormatProperties dateFormatProperties ;

    public DateFormatAutoConfiguration(DateFormatProperties dateFormatProperties) {
        this.dateFormatProperties = dateFormatProperties;
    }

    @Bean(name = "simpleDateFormat")
    public SimpleDateFormat DateFormattter(){

        logger.debug("start to initialize SimpleDateFormat with pattern: {}",dateFormatProperties.getPattern());
        return new SimpleDateFormat(dateFormatProperties.getPattern());
    }
}
```

`@ConditionalOnProperty(prefix = "spring.formatter",name = "enabled",havingValue = "true")`:条件加载配置，如果用户开启了enabled，才会自动加载配置类；

```java
@Bean(name = "simpleDateFormat")
    public SimpleDateFormat DateFormattter(){}
```

根据用户配置的格式化模式，自动注入对象到ioc容器中。

### POM文件

最后我们来看一下自定义starter的pom文件：

```java
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.1.6.RELEASE</version>
        </dependency>

        <!--lombok依赖-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.16</version>
        </dependency>

        <!-- 日志门面框架-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
        </dependency>
    </dependencies>
```

pom中只引入了spring-boot自己的starter；

### spring.factories配置

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=bugcode.online.config.DateFormatAutoConfiguration
```

最后，在resources文件夹下创建MATE-INF文件夹，然后创建spring.factories文件，文件内容时开启自动加载配置类即可，value值是指定的自动配置类；

### 测试类开发

新建maven项目，然后项目pom中引入自定义starter依赖；

```java
        <!--引入自定义starter-->
        <dependency>
            <groupId>bugcode.online</groupId>
            <artifactId>dataformat-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

**测试类**

```java
@Slf4j
@SpringBootApplication
public class Main implements ApplicationRunner {

    Logger logger = LoggerFactory.getLogger(Main.class);

    @Autowired
    private SimpleDateFormat simpleDateFormat;

    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {

        logger.info("格式化得日期是:{}",simpleDateFormat.format(new Date()));

    }
}
```

通过以上几个步骤，实现了自定义starter操作，其他需要使用此功能的项目，只需要引入依赖并且咋配置文件中开启此功能即可，非常方便顺滑；