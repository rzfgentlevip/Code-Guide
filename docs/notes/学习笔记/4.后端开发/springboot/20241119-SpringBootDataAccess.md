---
title: SpringBoot数据访问
icon: streamline-freehand-color:notes-quill
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - 面试
tag:
  - 面试
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/6bfl61gy/
---
# SpringBoot数据访问

# 自动配置原理

主程序：

```java
@MapperScan("com.atguigu.admin.mapper")
@ServletComponentScan(basePackages = "com.atguigu.admin")
@SpringBootApplication(exclude = RedisAutoConfiguration.class)
public class Boot05WebAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot05WebAdminApplication.class, args);
    }
}
```

SpringBootApplication注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

> 重点理解`*EnableAutoConfiguration*`注解。

# 1、SQL

整合其他数据源开发的步骤：

- 导入需要的某一个starter场景，场景会自动导入非常多的自动配置类
- ***autoCon,**自动配置类又会导入非常多的组件，比如操作数据库，会导入数据源，数据源的配置信息会和properties()里面的配置项进行绑定**。配置项会和配置文件进行绑定读取配置。

## 1.1、数据源的自动配置-**HikariDataSourcex**

只要我们导入jdbc场景，springboot会给我们的容器中放一个**HikariDataSourcex数据源，以后就可以从这个数据源中获取链接对数据库操作，目前是最好的数据源产品。**

### 1.1.1、导入JDBC场景

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>

```

### 1.1.2、分析自动配置

首先分析导入的jdbc导入了哪些内容,点进去spring-boot-starter-data-jdbc的相关依赖，可以看到导入的依赖：

```xml
<dependencies>
-- 导入spring-boot-starter-jdbc
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
      <version>2.4.0</version>
      <scope>compile</scope>
    </dependency>
-- 导入spring-data的jdbc相关操作
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-jdbc</artifactId>
      <version>2.1.1</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
```

在点进去spring-boot-starter-jdbc的依赖，可以发现导入一下内容：

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.4.0</version>
      <scope>compile</scope>
    </dependency>
-- 数据库连接池
    <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>3.4.5</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.3.1</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
```

依赖：

![https://cdn.nlark.com/yuque/0/2020/png/1354552/1606366100317-5e0199fa-6709-4d32-bce3-bb262e2e5e6a.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1606366100317-5e0199fa-6709-4d32-bce3-bb262e2e5e6a.png)

数据库驱动？

为什么导入JDBC场景，官方不导入驱动？官方不知道我们接下要操作什么数据库。

数据库版本和驱动版本对应

```xml
默认版本：<mysql.version>8.0.22</mysql.version>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
<!--            <version>5.1.49</version>-->
        </dependency>
想要修改版本
1、直接依赖引入具体版本（maven的就近依赖原则）
2、重新声明版本（maven的属性的就近优先原则）
    <properties>
        <java.version>1.8</java.version>
        <mysql.version>5.1.49</mysql.version>
    </properties>

<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<!--        导入Mysql驱动，但是官方已经做了版本仲裁，我的机器上mysql版本是5.所以需要重新修改版本-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
```

## 1.2、分析自动配置

> @Import注解提供了三种用法
>
>
> 1、@Import一个普通类 spring会将该类加载到spring容器中
>
> 2、@Import一个类，该类实现了ImportBeanDefinitionRegistrar接口，在重写的registerBeanDefinitions方法里面，能拿到BeanDefinitionRegistrybd的注册器，能手工往beanDefinitionMap中注册 beanDefinition
>
> 3、@Import一个类，该类实现了ImportSelector 重写selectImports方法该方法返回了String[]数组的对象，数组里面的类都会注入到spring容器当中
>
> @EnableConfigurationProperties注解的作用是：使使用 @ConfigurationProperties 注解的类生效。
>
> - 如果一个配置类只配置@ConfigurationProperties注解，而没有使用@Component或者实现了@Component的其他注解，那么在IOC容器中是获取不到properties 配置文件转化的bean。说白了 @EnableConfigurationProperties 相当于把使用 @ConfigurationProperties 的类进行了一次注入。
> - 简单点说@EnableConfigurationProperties的功能类似于@Component。
>
> @ConfigurationProperties作用：
>
> 在 SpringBoot 中，当想需要获取到配置文件数据时，除了可以用 Spring 自带的 @Value 注解外，SpringBoot 还提供了一种更加方便的方式：@ConfigurationProperties。只要在 Bean 上添加上了这个注解，指定好配置文件的前缀，那么对应的配置文件数据就会自动填充到 Bean 中。

### 1.2.1、自动配置的类

关于jdbc的自动配置：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.png)

关于整个数据库的配置：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%201.png)

- DataSourceAutoConfiguration ： 数据源的自动配置，相当于帮助我们配置好底层使用好的数据库连接池。
    - 修改数据源相关的配置：通过在ymal文件中修改**spring.datasource绑定的配置选项即可。**
    - **数据库连接池的配置，是自己容器中没有DataSource才自动配置的**
    - 底层配置好的连接池是：**HikariDataSource**

```java
	@Configuration(proxyBeanMethods = false)
	@Conditional(PooledDataSourceCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
//下面表示导入的数据库连接池
	@Import({ DataSourceConfiguration..class, DataSourceConfiguration.Tomcat.class,
			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.OracleUcp.class,
			DataSourceConfiguration.Generic.class, DataSourceJmxConfiguration.class })
	protected static class PooledDataSourceConfiguration
在DataSourceConfiguration类中配置了很多数据源，因为我们在导入启动器的时候，系统默认给我们导入了Hikar数据源，所以系统中默认是DataSourceConfiguration数据源
```

- DataSourceTransactionManagerAutoConfiguration： 数据源事务管理器的自动配置
- JdbcTemplateAutoConfiguration： **JdbcTemplate的自动配置，可以来对数据库进行crud操作，轻量级的操作数据库工具。**
    - 可以修改这个配置项@ConfigurationProperties(prefix = **"spring.jdbc"**) 来修改JdbcTemplate
- @Bean@Primary JdbcTemplate；容器中有这个组件
- JndiDataSourceAutoConfiguration： jndi的自动配置
- XADataSourceAutoConfiguration： 分布式事务相关的自动配置。

关于数据源自动配置源码分析：

```java
package org.springframework.boot.autoconfigure.jdbc;

import javax.sql.DataSource;
import javax.sql.XADataSource;

import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.condition.AnyNestedCondition;
import org.springframework.boot.autoconfigure.condition.ConditionMessage;
import org.springframework.boot.autoconfigure.condition.ConditionOutcome;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.condition.SpringBootCondition;
import org.springframework.boot.autoconfigure.jdbc.metadata.DataSourcePoolMetadataProvidersConfiguration;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.boot.jdbc.EmbeddedDatabaseConnection;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;
import org.springframework.util.StringUtils;

/**
 * {@link EnableAutoConfiguration Auto-configuration} for {@link DataSource}.
 *
 * @author Dave Syer
 * @author Phillip Webb
 * @author Stephane Nicoll
 * @author Kazuki Shimizu
 * @since 1.0.0
 */
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
//当容器中没有io.r2dbc.spi.ConnectionFactory（基于响应式编程的数据库连接池，
//如果不是使用响应式编程的数据源，才会动过自动配置类进行自动配置数据源）这个组件的时候，才会用自动配置数据源的配置
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
//开启配置文件绑定功能，和数据源相关的功能，全部在DataSourceProperties这个类里面进行绑定，而在这个类里面
//都是通过@ConfigurationProperties(prefix = "spring.datasource")，
//前缀为spring.datasource的配置进行数据源配置
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	@Conditional(EmbeddedDatabaseCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import(EmbeddedDataSourceConfiguration.class) //导入嵌入式数据源配置类 
	protected static class EmbeddedDatabaseConfiguration {

	}

	@Configuration(proxyBeanMethods = false)
	@Conditional(PooledDataSourceCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.OracleUcp.class,
			DataSourceConfiguration.Generic.class, DataSourceJmxConfiguration.class })
	protected static class PooledDataSourceConfiguration {

	}

	/**
	 * {@link AnyNestedCondition} that checks that either {@code spring.datasource.type}
	 * is set or {@link PooledDataSourceAvailableCondition} applies.
	 */
	static class PooledDataSourceCondition extends AnyNestedCondition {

		PooledDataSourceCondition() {
			super(ConfigurationPhase.PARSE_CONFIGURATION);
		}

		@ConditionalOnProperty(prefix = "spring.datasource", name = "type")
		static class ExplicitType {

		}

		@Conditional(PooledDataSourceAvailableCondition.class)
		static class PooledDataSourceAvailable {

		}

	}

	/**
	 * {@link Condition} to test if a supported connection pool is available.
	 */
	static class PooledDataSourceAvailableCondition extends SpringBootCondition {

		@Override
		public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
			ConditionMessage.Builder message = ConditionMessage.forCondition("PooledDataSource");
			if (DataSourceBuilder.findType(context.getClassLoader()) != null) {
				return ConditionOutcome.match(message.foundExactly("supported DataSource"));
			}
			return ConditionOutcome.noMatch(message.didNotFind("supported DataSource").atAll());
		}

	}

	/**
	 * {@link Condition} to detect when an embedded {@link DataSource} type can be used.
	 * If a pooled {@link DataSource} is available, it will always be preferred to an
	 * {@code EmbeddedDatabase}.
	 */
	static class EmbeddedDatabaseCondition extends SpringBootCondition {

		private static final String DATASOURCE_URL_PROPERTY = "spring.datasource.url";

		private final SpringBootCondition pooledCondition = new PooledDataSourceCondition();

		@Override
		public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
			ConditionMessage.Builder message = ConditionMessage.forCondition("EmbeddedDataSource");
			if (hasDataSourceUrlProperty(context)) {
				return ConditionOutcome.noMatch(message.because(DATASOURCE_URL_PROPERTY + " is set"));
			}
			if (anyMatches(context, metadata, this.pooledCondition)) {
				return ConditionOutcome.noMatch(message.foundExactly("supported pooled data source"));
			}
			EmbeddedDatabaseType type = EmbeddedDatabaseConnection.get(context.getClassLoader()).getType();
			if (type == null) {
				return ConditionOutcome.noMatch(message.didNotFind("embedded database").atAll());
			}
			return ConditionOutcome.match(message.found("embedded database").items(type));
		}

		private boolean hasDataSourceUrlProperty(ConditionContext context) {
			Environment environment = context.getEnvironment();
			if (environment.containsProperty(DATASOURCE_URL_PROPERTY)) {
				try {
					return StringUtils.hasText(environment.getProperty(DATASOURCE_URL_PROPERTY));
				}
				catch (IllegalArgumentException ex) {
					// Ignore unresolvable placeholder errors
				}
			}
			return false;
		}

	}

}
```

### 1.2.2、修改配置项

```yaml
#配置数据源
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/gmall
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver # 数据库驱动
```

### 1.2.3、测试

```java
@Slf4j
@SpringBootTest
class Boot05WebAdminApplicationTests {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Test
    void contextLoads() {

//        jdbcTemplate.queryForObject("select * from account_tbl")
//        jdbcTemplate.queryForList("select * from account_tbl",)
        Long aLong = jdbcTemplate.queryForObject("select count(*) from account_tbl", Long.class);
        log.info("记录总数：{}",aLong);
    }

}
```

## 1.3、使用Druid数据源

### 1.3.1、druid官方github地址

[https://github.com/alibaba/druid](https://github.com/alibaba/druid)

整合第三方技术的两种方式

- 手动自定义
- 找starter场景，starter通过自动配置类，将组件导入到容器中，而组件的核心配置项和配置文件绑定，很方便开发。

### 1.3.2、自定义方式

### 1、创建数据源

```xml
 //引入数据源
<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.17</version>
</dependency>

spring原生方法，通过xml文件给数据源初始化写配置信息，通过xml文件给容器中注入组件  
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
		destroy-method="close">
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="maxActive" value="20" />
		<property name="initialSize" value="1" />
		<property name="maxWait" value="60000" />
		<property name="minIdle" value="1" />
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<property name="minEvictableIdleTimeMillis" value="300000" />
		<property name="testWhileIdle" value="true" />
		<property name="testOnBorrow" value="false" />
		<property name="testOnReturn" value="false" />
		<property name="poolPreparedStatements" value="true" />
		<property name="maxOpenPreparedStatements" value="20" />

```

但是在springboot中，我们可以通过写一个配置类进行配置数据源。

```java
//通过配置类的方式配置数据源，通过配置类的方式，可以将配置信息抽取到yaml文件中，然后使用
@ConfigurationProperties("spring.datasource")注解将配置信息和配置类进行绑定
//@Deprecated
@Configuration
public class MyDataSourceConfig {

    // 默认的自动配置是判断容器中没有才会配@ConditionalOnMissingBean(DataSource.class)
//    所以我们如果添加了数据源，默认的数据源就不会生效
//    @ConfigurationProperties("spring.datasource")：将DataSource组件中的属性和配置文件中的属性进行绑定
//    添加此注解后，DruidDataSource组件中的属性和配置文件下的属性会自动绑定
    @ConfigurationProperties("spring.datasource")
    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();

//        一般将属性值抽取在配置文件中
//        druidDataSource.setUrl();
//        druidDataSource.setUsername();
//        druidDataSource.setPassword();
        //加入监控功能
//        druidDataSource.setFilters("stat,wall");

//        druidDataSource.setMaxActive(10);
	        return druidDataSource;
    }

    /**
     * 配置 druid的监控页功能
     * @return
     */
//    @Bean
    public ServletRegistrationBean statViewServlet(){
        StatViewServlet statViewServlet = new StatViewServlet();
        ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(statViewServlet, "/druid/*");

        registrationBean.addInitParameter("loginUsername","admin");
        registrationBean.addInitParameter("loginPassword","123456");

        return registrationBean;
    }

    /**
     * WebStatFilter 用于采集web-jdbc关联监控的数据。
     */
//    @Bean
    public FilterRegistrationBean webStatFilter(){
        WebStatFilter webStatFilter = new WebStatFilter();

        FilterRegistrationBean<WebStatFilter> filterRegistrationBean = new FilterRegistrationBean<>(webStatFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");

        return filterRegistrationBean;
    }

}
```

自定义jdbcTemplate:

```java
自定义MyDataSource 
//这里会从ymal文件中读取前缀为datasource的数据
@ConfigurationProperties(prefix = "spring.datasource")
//@Component
@Data
@ToString
public class MyDataSource {

    private String url;

    private String username;

    private String password;

    private String driverClassName;
}
//自定义配置
@Configuration
@EnableConfigurationProperties(MyDataSource.class)
public class DruidDataSourceAutoConfig {

//    在容器中配置Druid数据源
    @Bean("druidDs")
    @ConditionalOnClass(MyDataSource.class)
//    @ConfigurationProperties(prefix = "spring.datasource")
    public DruidDataSource getDruidDataSource(){

        return new DruidDataSource();
    }

    @Bean(name = "druidJdbcTemplate")
    public JdbcTemplate mqJdbcTemplate(@Qualifier("druidDs") DataSource dataSource){

        return new JdbcTemplate(dataSource);
    }
}
//自动注入
@Controller
@RequestMapping("/api")
public class DruidSourceController {

    @Autowired
    @Qualifier("druidJdbcTemplate")
    private JdbcTemplate jdbcTemplate;

    @ResponseBody
    @RequestMapping("/druid/{myid}")
    public String getAccountById(@PathVariable("myid") String myid){
        String sql = "select * from account where id = ?";
        List<Map<String, Object>> maps = jdbcTemplate.queryForList(sql, myid);

        for(Map<String,Object> map:maps){
            Account account = new Account();
            for(String s:map.keySet()){
                System.out.println(s+" "+map.get(s));
            }
        }
        return "info";
    }

}
```

### 2、StatViewServlet

StatViewServlet的用途包括：

- 提供监控信息展示的html页面
- 提供监控信息的JSON API

```xml
	<servlet>
		<servlet-name>DruidStatView</servlet-name>
		<servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>DruidStatView</servlet-name>
		<url-pattern>/druid/*</url-pattern>
	</servlet-mapping>
```

### 3、StatFilter

用于统计监控信息；如SQL监控、URI监控

```xml
需要给数据源中配置如下属性；可以允许多个filter，多个用，分割；如：

<property name="filters" value="stat,slf4j" />
```

系统中所有filter：

| 别名          | Filter类名                                              |
| ------------- | ------------------------------------------------------- |
| default       | com.alibaba.druid.filter.stat.StatFilter                |
| stat          | com.alibaba.druid.filter.stat.StatFilter                |
| mergeStat     | com.alibaba.druid.filter.stat.MergeStatFilter           |
| encoding      | com.alibaba.druid.filter.encoding.EncodingConvertFilter |
| log4j         | com.alibaba.druid.filter.logging.Log4jFilter            |
| log4j2        | com.alibaba.druid.filter.logging.Log4j2Filter           |
| slf4j         | com.alibaba.druid.filter.logging.Slf4jLogFilter         |
| commonlogging | com.alibaba.druid.filter.logging.CommonsLogFilter       |

**慢SQL记录配置**

```xml
<bean id="stat-filter" class="com.alibaba.druid.filter.stat.StatFilter">
    <property name="slowSqlMillis" value="10000" />
    <property name="logSlowSql" value="true" />
</bean>

使用 slowSqlMillis 定义慢SQL的时长
```

### 1.3.3、使用官方starter方式

### 1、引入druid-starter

```xml
 <!--        引入数据源,如果使用官方的starter，需要将这个数据源去掉-->
<!--        <dependency>-->
<!--            <groupId>com.alibaba</groupId>-->
<!--            <artifactId>druid</artifactId>-->
<!--            <version>1.1.17</version>-->
<!--        </dependency>-->  

-- 引入starter数据源
<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.17</version>
</dependency>
```

官方的starter引入的依赖：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%202.png)

### 2、分析自动配置

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%203.png)

数据源配置分析：

```java
@Configuration
//系统中引入了DruidDataSource类，自动配置才生效
@ConditionalOnClass(DruidDataSource.class)
//DataSourceAutoConfiguration这个是官方的数据源，@AutoConfigureBefore表示配置的要在官方的数据源之前
以自己配置的数据源优先，这样官方的数据源就不起作用了
@AutoConfigureBefore(DataSourceAutoConfiguration.class)
//@EnableConfigurationProperties：数据源所有的属性绑定的类，进去绑定的类可以看到，类和我们写的配置文件做了映射
@EnableConfigurationProperties({DruidStatProperties.class, DataSourceProperties.class})
@Import({DruidSpringAopConfiguration.class,
    DruidStatViewServletConfiguration.class,
    DruidWebStatFilterConfiguration.class,
    DruidFilterConfiguration.class})
public class DruidDataSourceAutoConfigure {

    private static final Logger LOGGER = LoggerFactory.getLogger(DruidDataSourceAutoConfigure.class);

    @Bean(initMethod = "init")
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        LOGGER.info("Init DruidDataSource");
        return new DruidDataSourceWrapper();
    }
}
```

- 扩展配置项 **spring.datasource.druid,这个类里面扩展了配置**
- DruidSpringAopConfiguration.**class**, 监控SpringBean的；配置项：**spring.datasource.druid.aop-patterns，只需要修改这些配置项，就可以对**SpringBean进行配置。
- DruidStatViewServletConfiguration.**class**, 监控页的配置：**spring.datasource.druid.stat-view-servlet；默认开启**
- DruidWebStatFilterConfiguration.**class**, web监控配置；**spring.datasource.druid.web-stat-filter；默认开启**
- DruidFilterConfiguration.**class：** 所有Druid自己filter的配置

```java
这些filter默认都是配置好的
private static final String FILTER_STAT_PREFIX = "spring.datasource.druid.filter.stat";
    private static final String FILTER_CONFIG_PREFIX = "spring.datasource.druid.filter.config";
    private static final String FILTER_ENCODING_PREFIX = "spring.datasource.druid.filter.encoding";
    private static final String FILTER_SLF4J_PREFIX = "spring.datasource.druid.filter.slf4j";
    private static final String FILTER_LOG4J_PREFIX = "spring.datasource.druid.filter.log4j";
    private static final String FILTER_LOG4J2_PREFIX = "spring.datasource.druid.filter.log4j2";
    private static final String FILTER_COMMONS_LOG_PREFIX = "spring.datasource.druid.filter.commons-log";
    private static final String FILTER_WALL_PREFIX = "spring.datasource.druid.filter.wall";
```

### 3、配置示例

```yaml
#配置数据源
spring:
#  数据源基本属性
  datasource:
    url: jdbc:mysql://localhost:3306/gmall
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver # 数据库驱动

#下面的配置信息，已经通过自动配置的方式配置好，我们只需要配置具体的配置值即可
# 配置druid数据源
    druid:
      aop-patterns: com.atguigu.admin.*  #springbean监控
      filters: stat,wall,slf4j  #所有开启的功能
      stat-view-servlet:  #监控页配置
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false
#
      web-stat-filter:  #web监控
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      filter:
        stat: #sql监控
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall: #防火墙
          enabled: true
          config:
            drop-table-allow: false

```

SpringBoot配置示例

[https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)

配置项列表[https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)

## 1.4、整合MyBatis操作

[https://github.com/mybatis](https://github.com/mybatis)

**starter:**

- SpringBoot官方的Starter：spring-boot-starter-*
- 第三方的： *-spring-boot-starter

```xml
<!--引入mybatis的starter-->
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
</dependency>
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%204.png)

导入starter:

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%205.png)

springboot启动之后，会找`EnableAutoConfiguration`，会自动导入spring.factories下面的内容，也就是加载jar包时候会自动导入以下两个组件：

```xml
org.mybatis.spring.boot.autoconfigure.MybatisLanguageDriverAutoConfiguration,\
org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
```

代码分析：

```java
@org.springframework.context.annotation.Configuration
//在引入jar包的时候，才会有{ SqlSessionFactory.class, SqlSessionFactoryBean.class }这两个类
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
//当容器中有且只有一个数据源的时候配置
@ConditionalOnSingleCandidate(DataSource.class)
//mybatis配置项绑定类，会将所有的配置绑定到MybatisProperties类上面
//而MybatisProperties这个类会和我们的配置文件进行映射绑定
@EnableConfigurationProperties(MybatisProperties.class)
//配置在配置了数据源之后才能加载mybatis配置
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration implements InitializingBean {

  private static final Logger logger = LoggerFactory.getLogger(MybatisAutoConfiguration.class);

  private final MybatisProperties properties;

  private final Interceptor[] interceptors;

  private final TypeHandler[] typeHandlers;

  private final LanguageDriver[] languageDrivers;

  private final ResourceLoader resourceLoader;

  private final DatabaseIdProvider databaseIdProvider;

  private final List<ConfigurationCustomizer> configurationCustomizers;

  public MybatisAutoConfiguration(MybatisProperties properties, ObjectProvider<Interceptor[]> interceptorsProvider,
      ObjectProvider<TypeHandler[]> typeHandlersProvider, ObjectProvider<LanguageDriver[]> languageDriversProvider,
      ResourceLoader resourceLoader, ObjectProvider<DatabaseIdProvider> databaseIdProvider,
      ObjectProvider<List<ConfigurationCustomizer>> configurationCustomizersProvider) {
    this.properties = properties;
    this.interceptors = interceptorsProvider.getIfAvailable();
    this.typeHandlers = typeHandlersProvider.getIfAvailable();
    this.languageDrivers = languageDriversProvider.getIfAvailable();
    this.resourceLoader = resourceLoader;
    this.databaseIdProvider = databaseIdProvider.getIfAvailable();
    this.configurationCustomizers = configurationCustomizersProvider.getIfAvailable();
  }

  @Override
  public void afterPropertiesSet() {
    checkConfigFileExists();
  }

  private void checkConfigFileExists() {
    if (this.properties.isCheckConfigLocation() && StringUtils.hasText(this.properties.getConfigLocation())) {
      Resource resource = this.resourceLoader.getResource(this.properties.getConfigLocation());
      Assert.state(resource.exists(),
          "Cannot find config location: " + resource + " (please add config file or check your Mybatis configuration)");
    }
  }
//在容器中给我们自动添加SqlSessionFactory 组件
  @Bean
  @ConditionalOnMissingBean
  public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {}
    

  private void applyConfiguration(SqlSessionFactoryBean factory) {}

//配置SqlSession组件，操作数据库
  @Bean
  @ConditionalOnMissingBean
  public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
    ExecutorType executorType = this.properties.getExecutorType();
    if (executorType != null) {
      return new SqlSessionTemplate(sqlSessionFactory, executorType);
    } else {
      return new SqlSessionTemplate(sqlSessionFactory);
    }
  }

  /**
   * This will just scan the same base package as Spring Boot does. If you want more power, you can explicitly use
   * {@link org.mybatis.spring.annotation.MapperScan} but this will get typed mappers working correctly, out-of-the-box,
   * similar to using Spring Data JPA repositories.
   */
  public static class AutoConfiguredMapperScannerRegistrar implements BeanFactoryAware, ImportBeanDefinitionRegistrar {

    private BeanFactory beanFactory;

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

      if (!AutoConfigurationPackages.has(this.beanFactory)) {
        logger.debug("Could not determine auto-configuration package, automatic mapper scanning disabled.");
        return;
      }

      logger.debug("Searching for mappers annotated with @Mapper");

      List<String> packages = AutoConfigurationPackages.get(this.beanFactory);
      if (logger.isDebugEnabled()) {
        packages.forEach(pkg -> logger.debug("Using auto-configuration base package '{}'", pkg));
      }

      BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
      builder.addPropertyValue("processPropertyPlaceHolders", true);
      builder.addPropertyValue("annotationClass", Mapper.class);
      builder.addPropertyValue("basePackage", StringUtils.collectionToCommaDelimitedString(packages));
      BeanWrapper beanWrapper = new BeanWrapperImpl(MapperScannerConfigurer.class);
      Set<String> propertyNames = Stream.of(beanWrapper.getPropertyDescriptors()).map(PropertyDescriptor::getName)
          .collect(Collectors.toSet());
      if (propertyNames.contains("lazyInitialization")) {
        // Need to mybatis-spring 2.0.2+
        builder.addPropertyValue("lazyInitialization", "${mybatis.lazy-initialization:false}");
      }
      if (propertyNames.contains("defaultScope")) {
        // Need to mybatis-spring 2.0.6+
        builder.addPropertyValue("defaultScope", "${mybatis.mapper-default-scope:}");
      }
      registry.registerBeanDefinition(MapperScannerConfigurer.class.getName(), builder.getBeanDefinition());
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
      this.beanFactory = beanFactory;
    }

  }

  /**
   * If mapper registering configuration or mapper scanning configuration not present, this configuration allow to scan
   * mappers based on the same component-scanning path as Spring Boot itself.
   */
  @org.springframework.context.annotation.Configuration
  @Import(AutoConfiguredMapperScannerRegistrar.class)
  @ConditionalOnMissingBean({ MapperFactoryBean.class, MapperScannerConfigurer.class })
  public static class MapperScannerRegistrarNotFoundConfiguration implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
      logger.debug(
          "Not found configuration for registering mapper bean using @MapperScan, MapperFactoryBean and MapperScannerConfigurer.");
    }

  }

}
```

### 1.4.1、配置模式写sql查询

之前配置mybatis需要配置一下内容：

- 全局配置文件
- SqlSessionFactory: 在自动配置文件中已经帮我们配置好了
- SqlSession：自动配置了 **SqlSessionTemplate 组合了SqlSession**
- @Import(**AutoConfiguredMapperScannerRegistrar**.**class**）:配置自动扫描mapper接口文件的位置
    - Mapper： 只要我们写的操作MyBatis的接口标准了 **@Mapper 就会被自动扫描进来**

```java
@EnableConfigurationProperties(MybatisProperties.class) ： MyBatis配置项绑定类。
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration{}

@ConfigurationProperties(prefix = "mybatis")
public class MybatisProperties
```

可以修改配置文件中 mybatis 开始的所有；

```yaml
# 配置mybatis规则
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml  #sql映射文件位置

Mapper接口--->绑定Xml（在xml文件中指定namespace绑定的接口文件即可）
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
//接口绑定xml文件,一个命名空间对应一个接口
<mapper namespace="com.atguigu.admin.mapper.AccountMapper">
//开启驼峰命名规则
<settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
<!--    public Account getAcct(Long id); -->
    <select id="getAcct" resultType="com.atguigu.admin.bean.Account">
        select * from  account_tbl where  id=#{id}
    </select>
</mapper>
```

配置mybatis配置的两种方式：

- 在mybatis-config.xml文件中配置
- 在ymal配置文件中配置

配置 **private** Configuration **configuration**; mybatis.**configuration下面的所有，就是相当于改mybatis全局配置文件中的值**

```yaml
# 配置mybatis规则
mybatis:
配置mybatis全局配置文件的位置
  config-location: classpath:mybatis/mybatis-config.xml
配置mybatis接口文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml
  configuration:
# 配置驼峰命名规则
    map-underscore-to-camel-case: true
# 可以配置其他配置项
 可以不写全局；配置文件，所有全局配置文件的配置都放在configuration配置项中即可
```

springboot中集成mybatis的使用步骤：

- 导入mybatis官方starter
- 编写mapper接口。标注**@Mapper注解**
- 编写sql映射文件并绑定mapper接口
- 在application.yaml中指定Mapper配置文件的位置，以及指定全局配置文件的信息 （建议；**配置在mybatis.configuration**）

### 1.4.2、注解模式写sql查询

当查询的sql比较简单的时候，可以使用注解查询。

```java
@Mapper
public interface CityMapper {
//    使用注解的方式查询
    @Select("select * from city where id=#{id}")
    public City getById(Long id);

    public void insert(City city);

}

```

### 1.4.3、混合模式

当查询的语句逻辑比较复杂，就可以使用混合模式，就是简单地sql使用注解方式查询，复杂的sql使用xml文件进行查询。注解和xml文件可以一起使用。

```java
@Mapper
public interface CityMapper {

    @Select("select * from city where id=#{id}")
    public City getById(Long id);
//这个查询使用xml文件进行查询
    public void insert(City city);

}
```

**最佳实战：**

1. 引入mybatis-starter
2. **配置application.yaml中，指定mapper-location位置即可**
3. 编写Mapper接口并标注@Mapper注解
4. 简单方法直接注解方式
5. 复杂方法编写mapper.xml进行绑定映射
6. *@MapperScan("com.atguigu.admin.mapper") 简化，其他的接口就可以不用标注@Mapper注解，这个注解标注在主程序函数上，这样就可以自动扫描mapper接口。*

## 1.5、整合 MyBatis-Plus 完成CRUD

### 1.5.1、什么是MyBatis-Plus

[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis](http://www.mybatis.org/mybatis-3/) 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

[mybatis plus 官网](https://baomidou.com/)

> 建议安装 **MybatisX** 插件

### 1.5.2、整合MyBatis-Plus

```xml
 整合mybatis-plus场景
 <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
</dependency>
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%206.png)

可以看到，mybatis-plus已经帮我们引入了spring-boot-starter-jdbc,所以我们在pom文件中可以不用在引入。

自动配置项：中自动配置了`MybatisPlusAutoConfiguration`，也就是项目一启动，就会自动加载这些配置。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%207.png)

自动配置

- MybatisPlusAutoConfiguration 配置类，MybatisPlusProperties 配置项绑定。**mybatis-plus：xxx 就是对mybatis-plus的定制**
- **SqlSessionFactory ，核心组件，自动配置好。底层是容器中默认的数据源**

```java
@Bean
@ConditionalOnMissingBean
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {}
可以看到DataSource 数据源是从容器中获取，@Bean标注的方法，如果参数是一个对象，那么这个参数会自动的从容器中
获取组件，而容器中数据源是外部数据源，所以当前操作的也就是外部数据源，比如druid数据源。
```

- **mapperLocations 自动配置好的。有默认值。classpath*:/mapper/**/*.xml；任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。 建议以后sql映射文件，放在 mapper下**
- **容器中也自动配置好了 SqlSessionTemplate**
- **@Mapper 标注的接口也会被自动扫描；建议直**@MapperScan(**"com.atguigu.admin.mapper"**) 批量扫描就行

```java
/**
     * If mapper registering configuration or mapper scanning configuration not present, this configuration allow to scan
     * mappers based on the same component-scanning path as Spring Boot itself.
     */
    @Configuration
//导入了AutoConfiguredMapperScannerRegistrar，相当于标注mapper注解的也会自动扫描注册
    @Import(AutoConfiguredMapperScannerRegistrar.class)
    @ConditionalOnMissingBean({MapperFactoryBean.class, MapperScannerConfigurer.class})
    public static class MapperScannerRegistrarNotFoundConfiguration implements InitializingBean {

        @Override
        public void afterPropertiesSet() {
            logger.debug(
                "Not found configuration for registering mapper bean using @MapperScan, MapperFactoryBean and MapperScannerConfigurer.");
        }
    }
```

原生mybatis写mapper接口，需要自己定义接口里面的方法，然后在xml文件或者使用注解的方式去查询数据库，在mybatis-plus汇总，有一个基类BaseMapper(`*public interface* BaseMapper<T> *extends* Mapper<T>`),有一个泛型T，想操作什么类型数据，指定类型即可。`BaseMapper`接口中已经定义了基础的crud功能，直接继承就不需要写crud方法。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%208.png)

**优点：**

- 只需要我们的Mapper继承 **BaseMapper** 就可以拥有crud能力
- 要求，实体中的属性都要在数据库中存在。

```java
@AllArgsConstructor
@NoArgsConstructor
@Data
@TableName("user") //标注查询那个表，默认情况下，不标注，会自己根据泛型的类型去数据库中找
对应的表，但是如果泛型和数据库中的表名不一样就会错误，所以通常使用此注解指定表名
public class User {

    /**
     * 所有属性都应该在数据库中
     */
    @TableField(exist = false)  //当前属性表中不存在,所以在查询的时候，不会查询这两个字段
    private String userName;
    @TableField(exist = false)
    private String password;

    public User(String un,String pd){
        this.userName = un;
        this.password = pd;
    }

    //以下是数据库字段
    private Long id;
    private String name;
    private Integer age;
    private String email;

}
```

`*public class* ServiceImpl<M *extends* BaseMapper<T>, T> *implements* IService<T>`{}

ServiceImpl接口中有两个泛型：

- `M *extends* BaseMapper<T>`：指的是当前的service要操作那张表，就将那个表的mapper接口拿过来。
- T：操作的表返回的类型。

> Mapper是对一张表的crud操作。

### 1.5.3、CRUD功能

```java
    @GetMapping("/user/delete/{id}")
    public String deleteUser(@PathVariable("id") Long id,
                             @RequestParam(value = "pn",defaultValue = "1")Integer pn,
                             RedirectAttributes ra){

        userService.removeById(id);

        ra.addAttribute("pn",pn);
        return "redirect:/dynamic_table";
    }

    @GetMapping("/dynamic_table")
    public String dynamic_table(@RequestParam(value="pn",defaultValue = "1") Integer pn,Model model){
        //表格内容的遍历
//        response.sendError
//     List<User> users = Arrays.asList(new User("zhangsan", "123456"),
//                new User("lisi", "123444"),
//                new User("haha", "aaaaa"),
//                new User("hehe ", "aaddd"));
//        model.addAttribute("users",users);
//
//        if(users.size()>3){
//            throw new UserTooManyException();
//        }
        //从数据库中查出user表中的用户进行展示

        //构造分页参数
        Page<User> page = new Page<>(pn, 2);
        //调用page进行分页
        Page<User> userPage = userService.page(page, null);

//        userPage.getRecords()
//        userPage.getCurrent()
//        userPage.getPages()

        model.addAttribute("users",userPage);

        return "table/dynamic_table";
    }
```

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper,User> implements UserService {

}

public interface UserService extends IService<User> {

}
```

# 2、NoSQL

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、**缓存**和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

## 2.1、Redis自动配置

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%209.png)

官方整合的自动配置包都在autoConfigure目录下：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%2010.png)

自动配置：

- RedisAutoConfiguration 自动配置类。RedisProperties 属性类 --> **spring.redis.xxx是对redis的配置**
- 连接工厂是准备好的。**Lettuce**ConnectionConfiguration、**Jedis**ConnectionConfiguration
- **自动注入了RedisTemplate**<**Object**, **Object**> ：允许k-v都是object类型 xxxTemplate；和组件进行数据交互。
- **自动注入了StringRedisTemplate；k：v都是String类型**

```
#在springboot中配置rides
redis:
    host: localhost
    port: 6379
#    password: rzf
client-type: jedis
#配置rides客户端
jedis:
      pool:
        max-active: 10
      url: redis://rzf:localhost:6379
    lettuce:
      pool:
        max-active: 10
        min-idle: 5
```

- **key：value**
- **底层只要我们使用 StringRedisTemplate、RedisTemplate就可以操作redis**

**redis环境搭建**

**1、阿里云按量付费redis。经典网络**

**2、申请redis的公网连接地址**

**3、修改白名单  允许0.0.0.0/0 访问**

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
指定从哪一个类里面获取绑定的属性
@EnableConfigurationProperties(RedisProperties.class)
在这里导入了两个类，LettuceConnectionConfiguration是客户端的配置，连接客户端的工厂
JedisConnectionConfiguration：jides客户端支持，相当于在这里配置了两种客户端的支持
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

//放入组件，xxxTemplate一般是用来和组件进行交互，操作组件的，RedisTemplate就是来操作rides的
	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

## 2.2、RedisTemplate与Lettuce

```java
    @Test
    void testRedis(){
        ValueOperations<String, String> operations = redisTemplate.opsForValue();

        operations.set("hello","world");

        String hello = operations.get("hello");
        System.out.println(hello);
    }
```

## 2.3、切换至jedis

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

<!--        导入jedis客户端，默认的客户端就不会生效，胡哦这在配置文件中声明客户端类型-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
```

```yaml
#在springboot中配置rides
  redis:
    host: localhost
    port: 6379
#    password: rzf
    client-type: jedis
#    配置rides客户端
    jedis:
      pool:
        max-active: 10
      url: redis://rzf:localhost:6379
    lettuce:
      pool:
        max-active: 10
        min-idle: 5
```