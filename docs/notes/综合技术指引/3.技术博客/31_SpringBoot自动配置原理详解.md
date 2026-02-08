---
title: 31、SpringBoot自动配置原理详解
icon: material-icon-theme:stackblitz
order: 31
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/SpringBoot自动配置原理详解/
---

# **Spring Boot 自动配置原理详解**

## **一、自动配置的核心思想**

### **1.1 什么是自动配置？**

Spring Boot 自动配置是一种**约定优于配置**的理念，根据项目中的依赖自动配置 Spring 应用。

```java
// 传统 Spring 需要手动配置
@Configuration
public class MyConfig {
    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}

// Spring Boot 自动配置 - 只需添加依赖
// pom.xml: <dependency>spring-boot-starter-data-jpa</dependency>
// application.properties: spring.datasource.url=...
```

### **1.2 自动配置解决的问题**

1. **减少样板代码**：避免重复的配置代码
2. **快速启动项目**：无需深入了解所有配置细节
3. **智能默认值**：根据环境提供合理的默认配置
4. **易于覆盖**：可以轻松自定义任何配置

## **二、自动配置的实现机制**

### **2.1 核心注解：@SpringBootApplication**

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// @SpringBootApplication 是组合注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@SpringBootConfiguration
@EnableAutoConfiguration  // ← 关键注解
@ComponentScan
public @interface SpringBootApplication {
    // ...
}
```

### **2.2 核心组件：spring.factories**

**META-INF/spring.factories** 是自动配置的注册表：

```java
# spring-boot-autoconfigure-2.7.0.jar!/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
  # ... 100+ 个自动配置类
```

## **三、自动配置的详细流程**

### **3.1 启动流程解析**

```java
// 简化版的自动配置流程
public class SpringBootAutoConfigurationProcess {
    
    public void process() {
        // 1. 扫描所有 jar 包中的 spring.factories
        List<String> autoConfigClasses = loadSpringFactories();
        
        // 2. 过滤：根据 @Conditional 条件
        List<String> enabledConfigs = filterByConditions(autoConfigClasses);
        
        // 3. 排序：根据 @AutoConfigureOrder, @Order
        List<String> sortedConfigs = sortConfigs(enabledConfigs);
        
        // 4. 实例化配置类并注册 Bean
        registerAutoConfigBeans(sortedConfigs);
        
        // 5. 执行用户自定义配置（优先级更高）
        applyUserConfiguration();
    }
}
```

并不是所有的jar都存在spring.factories 文件，下面这几类**必须有 spring.factories 的 jar 包**

| Jar 包类型                    | 示例                              | 是否必须有 |
| :---------------------------- | :-------------------------------- | :--------- |
| **Spring Boot Starter**       | `spring-boot-starter-*`           | 通常有     |
| **Spring Boot AutoConfigure** | `spring-boot-autoconfigure-*.jar` | 必须有     |
| **第三方 Starter**            | `mybatis-spring-boot-starter`     | 必须有     |
| **自定义 Starter**            | `company-xxx-starter`             | 必须有     |

### **3.2 条件注解（Conditional）机制**

Spring Boot 使用条件注解决定是否应用某个配置：

| 注解                              | 作用                         | 示例                                                         |
| :-------------------------------- | :--------------------------- | :----------------------------------------------------------- |
| `@ConditionalOnClass`             | 类路径存在指定类时生效       | `@ConditionalOnClass(DataSource.class)`                      |
| `@ConditionalOnMissingClass`      | 类路径不存在指定类时生效     | `@ConditionalOnMissingClass("javax.servlet.Servlet")`        |
| `@ConditionalOnBean`              | 容器中存在指定 Bean 时生效   | `@ConditionalOnBean(DataSource.class)`                       |
| `@ConditionalOnMissingBean`       | 容器中不存在指定 Bean 时生效 | `@ConditionalOnMissingBean(DataSource.class)`                |
| `@ConditionalOnProperty`          | 配置属性满足条件时生效       | `@ConditionalOnProperty(prefix="spring.datasource", name="url")` |
| `@ConditionalOnResource`          | 资源文件存在时生效           | `@ConditionalOnResource(resources="classpath:db.properties")` |
| `@ConditionalOnWebApplication`    | 是 Web 应用时生效            | `@ConditionalOnWebApplication(type=Type.SERVLET)`            |
| `@ConditionalOnNotWebApplication` | 不是 Web 应用时生效          | `@ConditionalOnNotWebApplication`                            |
| `@ConditionalOnExpression`        | SpEL 表达式为 true 时生效    | `@ConditionalOnExpression("${app.feature.enabled:false}")`   |

### **3.3 自动配置类的结构**

```java
// 典型的自动配置类示例：DataSourceAutoConfiguration
@Configuration(proxyBeanMethods = false)  // 配置类，不代理方法
@ConditionalOnClass({DataSource.class, EmbeddedDatabaseType.class})  // 条件1：类路径有这些类
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")  // 条件2：没有R2DBC连接工厂
@EnableConfigurationProperties(DataSourceProperties.class)  // 启用配置属性绑定
@Import({DataSourcePoolMetadataProvidersConfiguration.class, 
         DataSourceInitializationConfiguration.class})  // 导入其他配置
public class DataSourceAutoConfiguration {
    
    @Configuration
    @Conditional(PooledDataSourceCondition.class)  // 内部配置类，条件更严格
    @ConditionalOnMissingBean({DataSource.class, XADataSource.class})
    @Import({Hikari.class, Tomcat.class, Dbcp2.class, Generic.class, 
             DataSourceJmxConfiguration.class})
    protected static class PooledDataSourceConfiguration {
        
        @Bean
        @ConditionalOnMissingBean(DataSourceProperties.class)
        public DataSourceProperties dataSourceProperties() {
            return new DataSourceProperties();
        }
        
        @Bean
        @ConditionalOnProperty(prefix = "spring.datasource", name = "type")
        public DataSource dataSource(DataSourceProperties properties) {
            // 根据配置创建 DataSource
            return properties.initializeDataSourceBuilder().build();
        }
    }
    
    // 嵌入式数据库配置（如H2、HSQLDB）
    @Configuration
    @Conditional(EmbeddedDatabaseCondition.class)
    @ConditionalOnMissingBean({DataSource.class, XADataSource.class})
    @Import(EmbeddedDataSourceConfiguration.class)
    protected static class EmbeddedDatabaseConfiguration {
    }
}
```

## **四、配置加载顺序和优先级**

### **4.1 配置源加载顺序**

```java
// 从高到低的优先级（后面的会覆盖前面的）
1. 命令行参数 (--server.port=8081)
2. Java系统属性 (System.getProperties())
3. 操作系统环境变量
4. 打包在jar外的配置文件 (application-{profile}.properties/yml)
5. 打包在jar内的配置文件 (application-{profile}.properties/yml)
6. 打包在jar外的默认配置文件 (application.properties/yml)
7. 打包在jar内的默认配置文件 (application.properties/yml)
8. @Configuration类上的@PropertySource注解
9. SpringApplication.setDefaultProperties()设置的默认属性
```

### **4.2 自动配置加载顺序**

```java
// 通过 @AutoConfigureOrder, @Order 控制
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)  // 高优先级
public class EarlyConfiguration {
    // 先执行的配置
}

@Configuration
@AutoConfigureOrder(Ordered.LOWEST_PRECEDENCE - 10)  // 低优先级
public class LateConfiguration {
    // 后执行的配置
}

// 特殊的：@AutoConfigureAfter, @AutoConfigureBefore
@Configuration
@AutoConfigureAfter(DataSourceAutoConfiguration.class)  // 在数据源配置之后
public class JpaAutoConfiguration {
    // 依赖 DataSource 的配置
}

@Configuration
@AutoConfigureBefore(WebMvcAutoConfiguration.class)  // 在 WebMVC 配置之前
public class MyWebConfiguration {
    // 影响 WebMVC 的配置
}
```

## **五、自定义自动配置**

### **5.1 创建自定义 Starter**

**项目结构**：

```java
my-spring-boot-starter/
├── src/main/java/
│   └── com/example/starter/
│       ├── MyService.java              # 服务类
│       ├── MyServiceAutoConfiguration.java  # 自动配置类
│       └── MyServiceProperties.java    # 配置属性类
├── src/main/resources/
│   └── META-INF/
│       └── spring.factories            # 注册自动配置
└── pom.xml
```

### **5.2 实现自定义自动配置**

```java
// 1. 配置属性类
@ConfigurationProperties(prefix = "my.service")
public class MyServiceProperties {
    private String endpoint = "http://default.com";
    private int timeout = 5000;
    private boolean enabled = true;
    
    // getters/setters
}

// 2. 服务类
public class MyService {
    private final MyServiceProperties properties;
    
    public MyService(MyServiceProperties properties) {
        this.properties = properties;
    }
    
    public void doSomething() {
        if (properties.isEnabled()) {
            System.out.println("Connecting to: " + properties.getEndpoint());
        }
    }
}

// 3. 自动配置类
@Configuration
@EnableConfigurationProperties(MyServiceProperties.class)  // 启用属性绑定
@ConditionalOnClass(MyService.class)  // 当 MyService 在类路径时
@ConditionalOnProperty(prefix = "my.service", value = "enabled", matchIfMissing = true)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)  // 在数据源配置之后
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean  // 用户没有自定义 MyService 时才创建
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties);
    }
    
    // 可以有多个 Bean 定义
    @Bean
    @ConditionalOnMissingBean
    public MyServiceHelper myServiceHelper(MyService myService) {
        return new MyServiceHelper(myService);
    }
}

// 4. 注册到 spring.factories
# src/main/resources/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.starter.MyServiceAutoConfiguration
```

### **5.3 条件注解的进阶使用**

```java
// 自定义条件注解
public class OnProductionCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        String profile = env.getProperty("spring.profiles.active", "dev");
        return "prod".equals(profile);
    }
}

// 使用自定义条件
@Configuration
@Conditional(OnProductionCondition.class)
public class ProductionConfiguration {
    // 生产环境特定的配置
}

// 组合条件
@Configuration
@ConditionalOnClass(name = {
    "com.fasterxml.jackson.databind.ObjectMapper",
    "com.fasterxml.jackson.core.JsonGenerator"
})
@ConditionalOnProperty(prefix = "app.json", name = "enabled", havingValue = "true")
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
public class JacksonAutoConfiguration {
    // 多个条件同时满足时才生效
}
```

## **六、自动配置的调试和诊断**

### **6.1 启用自动配置报告**

```yaml
# application.yml
debug: true  # 开启调试模式，显示自动配置报告

# 或者通过启动参数
java -jar app.jar --debug
```

### **6.2 查看自动配置详情**

```java
# 1. 启动时查看自动配置报告
# 会显示：
#   Positive matches: 匹配的自动配置
#   Negative matches: 未匹配的自动配置（原因）
#   Exclusions: 排除的配置
#   Unconditional classes: 无条件配置

# 2. 使用 Actuator 端点（需要依赖）
# pom.xml: spring-boot-starter-actuator
# 访问: http://localhost:8080/actuator/conditions

# 3. 查看所有自动配置类
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = 
            SpringApplication.run(MyApp.class, args);
        
        // 获取所有自动配置类
        Map<String, String> autoConfigs = context.getBeansOfType(
            EnableAutoConfiguration.class);
        autoConfigs.forEach((name, bean) -> 
            System.out.println(name + ": " + bean.getClass()));
    }
}
```

### **6.3 排除特定自动配置**

```java
// 方法1: 使用 exclude
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})

// 方法2: 使用 excludeName
@SpringBootApplication(excludeName = {
    "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration"
})

// 方法3: 配置文件中排除
spring.autoconfigure.exclude=\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration

// 方法4: 使用 @Import 精确控制
@Configuration
@Import({
    DispatcherServletAutoConfiguration.class,
    WebMvcAutoConfiguration.class
    // 只导入需要的配置
})
public class MyWebConfiguration {
}
```

## **七、Spring Boot Starters 工作原理**

### **7.1 官方 Starter 示例**

```java
<!-- spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 实际包含了 -->
<dependencies>
    <dependency>spring-boot-starter</dependency>
    <dependency>spring-boot-starter-json</dependency>
    <dependency>spring-boot-starter-tomcat</dependency>
    <dependency>spring-webmvc</dependency>
    <dependency>spring-web</dependency>
</dependencies>
```

### **7.2 Starter 依赖关系**

```java
spring-boot-starter-web
    ├── spring-boot-starter
    │   ├── spring-boot
    │   ├── spring-boot-autoconfigure
    │   └── spring-boot-starter-logging
    ├── spring-boot-starter-json
    ├── spring-boot-starter-tomcat
    └── spring-webmvc + spring-web
```

### **7.3 自定义 Starter 最佳实践**

```xml
<!-- 自定义 Starter 的 pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <spring-boot.version>2.7.0</spring-boot.version>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <dependencies>
        <!-- 必须依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>  <!-- optional: 仅编译时需要 -->
        </dependency>
        
        <!-- Starter 提供的功能依赖 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>my-core-library</artifactId>
            <version>1.0.0</version>
        </dependency>
        
        <!-- 测试依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

## **八、自动配置的源码分析**

### **8.1 核心源码位置**

```java
// 1. 自动配置入口
org.springframework.boot.autoconfigure.SpringBootApplication
org.springframework.boot.autoconfigure.EnableAutoConfiguration

// 2. 条件注解实现
org.springframework.boot.autoconfigure.condition
├── SpringBootCondition          // 基类
├── OnClassCondition             // @ConditionalOnClass
├── OnBeanCondition              // @ConditionalOnBean
├── OnPropertyCondition          // @ConditionalOnProperty
└── ...

// 3. 配置加载
org.springframework.boot.context.properties
├── EnableConfigurationProperties
└── ConfigurationPropertiesBindingPostProcessor

// 4. 自动配置报告
org.springframework.boot.autoconfigure
└── ConditionEvaluationReport
```

### **8.2 关键源码解析**

```java
// AutoConfigurationImportSelector - 加载自动配置的核心类
public class AutoConfigurationImportSelector implements ... {
    
    protected AutoConfigurationEntry getAutoConfigurationEntry(...) {
        // 1. 检查是否启用自动配置
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        
        // 2. 获取所有配置类的属性
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        
        // 3. 加载候选配置
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        
        // 4. 移除重复项
        configurations = removeDuplicates(configurations);
        
        // 5. 获取排除项
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        
        // 6. 检查排除项是否有效
        checkExcludedClasses(configurations, exclusions);
        
        // 7. 移除排除项
        configurations.removeAll(exclusions);
        
        // 8. 应用过滤器
        configurations = getConfigurationClassFilter().filter(configurations);
        
        // 9. 触发事件
        fireAutoConfigurationImportEvents(configurations, exclusions);
        
        return new AutoConfigurationEntry(configurations, exclusions);
    }
    
    protected List<String> getCandidateConfigurations(...) {
        // 从 META-INF/spring.factories 加载
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
            getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        
        // 也支持新式的 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
        return configurations;
    }
}
```

## **九、实际案例分析**

### **9.1 Web MVC 自动配置**

```java
// WebMvcAutoConfiguration 简化版
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)  // 用户没自定义时才生效
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({DispatcherServletAutoConfiguration.class, 
                     TaskExecutionAutoConfiguration.class, 
                     ValidationAutoConfiguration.class})
public class WebMvcAutoConfiguration {
    
    // 视图解析器
    @Bean
    @ConditionalOnMissingBean
    public InternalResourceViewResolver defaultViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix(this.mvcProperties.getView().getPrefix());
        resolver.setSuffix(this.mvcProperties.getView().getSuffix());
        return resolver;
    }
    
    // 静态资源处理
    @Bean
    @ConditionalOnMissingBean
    public WelcomePageHandlerMapping welcomePageHandlerMapping(...) {
        return new WelcomePageHandlerMapping(...);
    }
    
    // 格式化器
    @Bean
    @ConditionalOnMissingBean
    public FormattingConversionService mvcConversionService(...) {
        // 配置日期、数字格式化
    }
    
    // 自定义配置扩展点
    @Configuration
    @Import(EnableWebMvcConfiguration.class)
    @EnableConfigurationProperties({WebMvcProperties.class, ResourceProperties.class})
    public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
        
        // 用户可以继承 WebMvcConfigurer 来自定义
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            // 添加拦截器
        }
    }
}
```

### **9.2 数据源自动配置**

```java
// DataSourceConfiguration 简化版 - 演示多数据源选择
abstract class DataSourceConfiguration {
    
    // HikariCP 配置（默认优先）
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(HikariDataSource.class)
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.HikariDataSource", 
                          matchIfMissing = true)
    static class Hikari {
        @Bean
        @ConfigurationProperties(prefix = "spring.datasource.hikari")
        public HikariDataSource dataSource(DataSourceProperties properties) {
            return properties.initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
        }
    }
    
    // Tomcat JDBC 连接池
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.tomcat.jdbc.pool.DataSource")
    static class Tomcat {
        @Bean
        @ConfigurationProperties(prefix = "spring.datasource.tomcat")
        public org.apache.tomcat.jdbc.pool.DataSource dataSource(DataSourceProperties properties) {
            // Tomcat 连接池配置
        }
    }
    
    // 其他连接池...（DBCP2, Oracle UCP等）
}
```

## **十、最佳实践和常见问题**

### **10.1 最佳实践**

1. **优先使用属性配置**：能在 application.yml 中配置的，就不要写代码
2. **合理使用条件注解**：确保配置只在正确的环境下生效
3. **提供合理的默认值**：方便用户快速上手
4. **清晰的配置前缀**：使用有意义的属性前缀（如 `my.service.endpoint`）
5. **完善的文档**：说明每个配置项的作用和默认值
6. **向后兼容**：新增配置项时尽量保持向后兼容

### **10.2 常见问题解决**

#### **问题1：自动配置不生效**

```java
// 检查步骤：
// 1. 确保 spring.factories 文件位置正确
// 2. 检查条件注解是否满足
// 3. 查看 debug 日志：设置 debug=true
// 4. 检查是否有 @SpringBootApplication(exclude=...)

// 调试方法：
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        app.setAdditionalProfiles("debug");  // 启用调试profile
        ConfigurableApplicationContext context = app.run(args);
        
        // 打印所有自动配置类
        String[] beanNames = context.getBeanNamesForType(Object.class);
        Arrays.stream(beanNames)
            .filter(name -> name.contains("AutoConfiguration"))
            .forEach(System.out::println);
    }
}
```

#### **问题2：配置冲突**

```yaml
# 解决多个自动配置冲突
# 方法1：明确指定使用哪个
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource  # 明确指定Hikari

# 方法2：排除不需要的自动配置
spring:
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

#### **问题3：自定义配置覆盖自动配置**

```java
// 正确的方式：使用 @ConditionalOnMissingBean
@Configuration
public class MyDataSourceConfig {
    
    @Bean
    @ConfigurationProperties("app.datasource")
    public DataSource myDataSource() {
        // 自定义数据源，会覆盖自动配置的
        return DataSourceBuilder.create().build();
    }
}

// 或者在自动配置中：
@Configuration
public class MyAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(DataSource.class)  // 用户没自定义时才创建
    public DataSource defaultDataSource() {
        return new HikariDataSource();
    }
}
```

## **总结**

Spring Boot 自动配置的核心原理：

1. **`@EnableAutoConfiguration`**：启用自动配置的入口注解
2. **`META-INF/spring.factories`**：自动配置类的注册表
3. **条件注解（`@ConditionalOnXxx`）**：决定配置是否生效
4. **配置属性绑定**：`@ConfigurationProperties` + `application.yml`
5. **优先级控制**：配置加载顺序、`@AutoConfigureOrder`、`@AutoConfigureBefore/After`

**关键设计思想**：

- **开箱即用**：添加依赖即可获得完整功能
- **约定优于配置**：合理的默认值，减少配置
- **可定制性**：可以覆盖任何默认配置
- **条件化配置**：根据环境智能启用配置

通过理解自动配置原理，可以更好地使用 Spring Boot，也能创建自己的 Starter，为团队提供统一的开发体验。