---
title: Spring注解
icon: streamline-freehand-color:notes-add
order: 1
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - SPRING
  - JAVA
tag:
  - spring
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/htxuouwd/
---

# Spring注解

# 1、@SpringBoot注解

我们可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

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

根据 SpringBoot 官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描该类所在的包下所有的类。
- `@Configuration`：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类

# 2、SpringBean相关

## 2.1、AutoWired

自动导入对象到类中，被注入进的类同样要被 Spring 容器管理比如：Service 类注入到 Controller 类中。

## **2.2. `Component`,`@Repository`,`@Service`, `@Controller`**

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,可以采用以下注解实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面

## 2.3、**`@RestController`**

`@RestController`注解是`@Controller和`@`ResponseBody`的合集,表示这是个控制器 bean,**并且是将函数的返回值直接填入 HTTP响应体中**,是 REST 风格的控制器。

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的 Spring MVC 的应用，对应于前后端不分离的情况。`@Controller` +`@ResponseBody` 返回 JSON 或 XML 形式数据

## **2.4、`@Scope`**

声明 Spring Bean 的作用域，使用方法:

**四种常见的 Spring Bean 的作用域：**

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- session : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

## **2.5. `Configuration`**

一般用来声明配置类，可以使用 `@Component`注解替代，不过使用`Configuration`注解声明配置类更加语义化。

# **3、处理常见的 HTTP 请求类型**

**5 种常见的请求类型:**

- **GET** ：请求从服务器获取特定资源。举个例子：`GET /users`（获取所有学生）
- **POST** ：在服务器上创建一个新的资源。举个例子：`POST /users`（创建学生）
- **PUT** ：更新服务器上的资源（客户端提供更新后的整个资源）。举个例子：`PUT /users/12`（更新编号为 12 的学生）
- **DELETE** ：从服务器删除特定的资源。举个例子：`DELETE /users/12`（删除编号为 12 的学生）
- **PATCH** ：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新）。

## **3.1、GET 请求**

`@GetMapping("users")` 等价于`@RequestMapping(value="/users",method=RequestMethod.GET)`

## **3.2. POST 请求**

`@PostMapping("users")` 等价于`@RequestMapping(value="/users",method=RequestMethod.POST)`

关于`@RequestBody`注解的使用，在下面的“前后端传值”这块会讲到。

## **3.3. PUT 请求**

`@PutMapping("/users/{userId}")` 等价于`@RequestMapping(value="/users/{userId}",method=RequestMethod.PUT)`

## **3.4. DELETE 请求**

`@DeleteMapping("/users/{userId}")`等价于`@RequestMapping(value="/users/{userId}",method=RequestMethod.DELETE)`

## **3.5. PATCH 请求**

一般实际项目中，我们都是 PUT 不够用了之后才用 PATCH 请求去更新数据。

# **4、前后端传值**

## **4.1、`@PathVariable` 和 `@RequestParam`**

`@PathVariable`用于获取**路径参数**，`@RequestParam`用于获取**查询参数**。

```java
@GetMapping("getperson/{id}/person")
    public Person getPerson(@PathVariable("id") String id,
                            @RequestParam(value = "type",required = false)String type){

    }
```

如果我们请求的 url 是：`/getperson/{123456}/person?type=web`

那么我们服务获取到的数据就是：id`=123456,type=web`。

## **4.2、`@RequestBody`**

用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且**Content-Type 为 application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象。

> @RequestBody主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；而最常用的使用请求体传参的无疑是POST请求了，所以使用@RequestBody接收数据时，一般都用POST方式进行提交。在后端的同一个接收方法里，@RequestBody与@RequestParam()可以同时使用，@RequestBody最多只能有一个，而@RequestParam()可以有多个。

注：一个请求，只有一个RequestBody；一个请求，可以有多个[RequestParam](https://so.csdn.net/so/search?q=RequestParam&spm=1001.2101.3001.7020)。

# **5、读取配置信息**

**很多时候我们需要将一些常用的配置信息比如阿里云 oss、发送短信、微信认证的相关配置信息等等放到配置文件中。**

**下面我们来看一下 Spring 为我们提供了哪些方式帮助我们从配置文件中读取这些配置信息。**

我们的数据源`application.yml`内容如下：：

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true
    contentnegotiation:
      favor-parameter: true
  #  mvc:
#    static-path-pattern: /res/**

  resources:
    static-locations: [classpath:/haha/]
    add-mappings: true
    cache:
      period: 11000
#      前置路径，以后所有的请求都必须添加前缀world
server:
  servlet:
    context-path: /world
```

## **5.1、`@value`(常用)**

使用 `@Value("${property}")` 读取比较简单的配置信息：

yml数据源配置：

```yaml
#配置数据源
spring:
  #  数据源基本属性
  datasource:
    mysql:
      url: jdbc:mysql://localhost:3306/gmall
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver # 数据库驱动
```

读取数据源配置信息：

```java
@Component
@Data
@ToString
public class ValueProperties {

    @Value("${spring.datasource.mysql.url}")
    private String url;

    @Value("${spring.datasource.mysql.username}")
    private String username;

    @Value("${spring.datasource.mysql.password}")
    private String password;

    @Value("${spring.datasource.mysql.driver-class-name}")
    private String driverClassName;
}
```

使用@Value配置基本数据类型：

```java
@ConfigurationProperties(prefix = "person") //根配置文件绑定，指定前缀,配置文件中药以person为前缀注入数据
@Component
@ToString
@Data
public class Person {
    private String userName;
    private Boolean boss;
    private Date birth;
    private Integer age;
    private Pet pet;
    private String[] interests;
    private List<String> animal;
    private Map<String, Object> score;
    private Set<Double> salarys;
    private Map<String, List<Pet>> allPets;
}

person:
#  单引号会将 \n作为字符串输出   双引号会将\n 作为换行输出
#  双引号不会转义，单引号会转义
  boss: true
  birth: 2019/12/9
  age: 18
#  interests: [篮球,足球]
  interests:
    - 篮球
    - 足球
    - 18
  animal: [阿猫,阿狗]
#  score:
#    english: 80
#    math: 90
  score: {english:80,math:90}
  salarys:
    - 9999.98
    - 9999.99
  pet:
    name: 阿狗
    weight: 99.99
  allPets:
    sick:
      - {name: 阿狗,weight: 99.99}
      - name: 阿猫
        weight: 88.88
      - name: 阿虫
        weight: 77.77
    health:
      - {name: 阿花,weight: 199.99}
      - {name: 阿明,weight: 199.99}
  user-name: zhangsan
```

## **5.2、`@ConfigurationProperties`(常用)**

通过`@ConfigurationProperties`读取配置信息并与 bean 绑定。

使用`@ConfigurationProperties(prefix = "spring.datasource.mysql")注解配置数据源：`

```java
//这里会从ymal文件中读取前缀为datasource的数据
@ConfigurationProperties(prefix = "spring.datasource.mysql")
@Component
@Data
@ToString
public class DataSource {

    private String url;

    private String username;

    private String password;

    private String driverClassName;
}
```

## **5.3、`PropertySource`（不常用）**

`@PropertySource`读取指定 properties 文件

application.properties配置：

```java
spring.datasource.mysql.url=url: jdbc:mysql://localhost:3306/gmall
spring.datasource.mysql.username: root
spring.datasource.mysql.password: root
spring.datasource.mysql.driver-class-name: com.mysql.jdbc.Driver # 数据库驱动
```

读取数据配置信息：

```java
@Component
@PropertySource("classpath:application.properties")
@Data
@ToString
public class DataSourceProperties {

    @Value("${spring.datasource.mysql.url}")
    private String url;

    @Value("${spring.datasource.mysql.username}")
    private String username;

    @Value("${spring.datasource.mysql.password}")
    private String password;

    @Value("${spring.datasource.mysql.driver-class-name}")
    private String driverClassName;
}
```

# **6、参数校验**

**数据的校验的重要性就不用说了，即使在前端对数据进行校验的情况下，我们还是要对传入后端的数据再进行一遍校验，避免用户绕过浏览器直接通过一些 HTTP 工具直接向后端请求一些违法数据。**

**JSR(Java Specification Requests）** 是一套 JavaBean 参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们 JavaBean 的属性上面，这样就可以在需要校验的时候进行校验了，非常方便！

校验的时候我们实际用的是 **Hibernate Validator**
框架。Hibernate Validator 是 Hibernate 团队最初的数据校验框架，Hibernate Validator 4.x 是
Bean Validation 1.0（JSR 303）的参考实现，Hibernate Validator 5.x 是 Bean
Validation 1.1（JSR 349）的参考实现，目前最新版的 Hibernate Validator 6.x 是 Bean
Validation 2.0（JSR 380）的参考实现。

SpringBoot 项目的 spring-boot-starter-web 依赖中已经有 hibernate-validator 包，不需要引用相关依赖。如下图所示（通过 idea 插件—Maven Helper 生成）：

需要注意的是： **所有的注解，推荐使用 JSR 注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`**

## **6.1、一些常用的字段验证的注解**

`@NotEmpty` 被注释的字符串的不能为 null 也不能为空

`@NotBlank` 被注释的字符串非 null，并且必须包含一个非空白字符

`@Null` 被注释的元素必须为 null

`@NotNull` 被注释的元素必须不为 null

`@AssertTrue` 被注释的元素必须为 true

`@AssertFalse` 被注释的元素必须为 false

`@Pattern(regex=,flag=)`被注释的元素必须符合指定的正则表达式

`@Email` 被注释的元素必须是 Email 格式。

`@Min(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值

`@Max(value)`被注释的元素必须是一个数字，其值必须小于等于指定的最大值

`@DecimalMin(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值`@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值

`@Size(max=, min=)`被注释的元素的大小必须在指定的范围内

`@Digits (integer, fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内

`@Past`被注释的元素必须是一个过去的日期

`@Future` 被注释的元素必须是一个将来的日期

## **6.2、验证请求体(RequestBody)**

我们在需要验证的参数上加上了`@Valid`注解，如果验证失败，它将抛出`MethodArgumentNotValidException`。

```java
@GetMapping("getperson/{id}/person")
    public Person getPerson(@PathVariable("id") @Valid String id,
                            @RequestParam(value = "type",required = false)String type){

    }
```

## **6.3. 验证请求参数(Path Variables 和 Request Parameters)**

**一定一定不要忘记在类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数。**

```java
@Validated
@RestController
public class HelloController {}
```

# **8、JPA 相关**

使用springjpa前首先要导入spring jpa相关的依赖：

```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

## **8.1、创建表**

`@Entity`声明一个类对应一个数据库实体。

`@Table` 设置表明

```java
@Entity
@Table(name = "role")
public class Role {

    @Id
//    生成主键的策略
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;
}
```

## **8.2. 创建主键**

`@Id` ：声明一个字段为主键。

使用`@Id`声明之后，我们还需要定义主键的生成策略。我们可以使用 `@GeneratedValue` 指定主键生成策略。

**1.通过 `@GeneratedValue`直接使用 JPA 内置提供的四种主键生成策略来指定主键生成策略。**

```java
@GeneratedValue(strategy = GenerationType.AUTO)
```

JPA 使用枚举定义了 4 中常见的主键生成策略，如下：

```java
public enum GenerationType { 

    /**
     * Indicates that the persistence provider must assign 
     * primary keys for the entity using an underlying 
     * database table to ensure uniqueness.
     */
    TABLE, 

    /**
     * Indicates that the persistence provider must assign 
     * primary keys for the entity using a database sequence.
     */
    SEQUENCE, 

    /**
     * Indicates that the persistence provider must assign 
     * primary keys for the entity using a database identity column.
     */
    IDENTITY, 

    /**
     * Indicates that the persistence provider should pick an 
     * appropriate strategy for the particular database. The 
     * <code>AUTO</code> generation strategy may expect a database 
     * resource to exist, or it may attempt to create one. A vendor 
     * may provide documentation on how to create such resources 
     * in the event that it does not support schema generation 
     * or cannot create the schema resource at runtime.
     */
    AUTO
}
```

`@GeneratedValue`注解默认使用的策略是`GenerationType.AUTO`

```java
@Target({METHOD, FIELD})
@Retention(RUNTIME)

public @interface GeneratedValue {

    /**
     * (Optional) The primary key generation strategy
     * that the persistence provider must use to
     * generate the annotated entity primary key.
     */
    GenerationType strategy() default AUTO;

    /**
     * (Optional) The name of the primary key generator
     * to use as specified in the {@link SequenceGenerator} 
     * or {@link TableGenerator} annotation.
     * <p> Defaults to the id generator supplied by persistence provider.
     */
    String generator() default "";
}
```

一般使用 MySQL 数据库的话，使用`GenerationType.IDENTITY`策略比较普遍一点（分布式系统的话需要另外考虑使用分布式 ID）。

**2.通过 `@GenericGenerator`声明一个主键策略，然后 `@GeneratedValue`使用这个策略**

```java
@Id
    @GeneratedValue(strategy = "seqgenerate")
    @GenericGenerator(name = "seqgenerate",strategy = "identity")
    private String seq;

等价于

@GeneratedValue(strategy = GenerationType.IDENTITY)
```

## **8.3. 设置字段类型**

`@Column` 声明字段。

**示例：**

设置属性 userName 对应的数据库字段名为 user_name，长度为 32，非空

```java
@Column(name = "user_password",nullable = false,length = 32)
    private String userPassword;
```

设置字段类型并且加默认值，这个还是挺常用的。

```java
@Column(columnDefinition = "tinyint(1) default 1")
    private boolean enable;
```

## **8.4. 指定不持久化特定字段**

`@Transient` ：声明不需要与数据库映射的字段，在保存的时候不需要保存进数据库 。

如果我们想让`secrect` 这个字段不被持久化，可以使用 `@Transient`关键字声明。

```java
@Transient
    private String secrect;//不会持久化到数据库
```

```
//除了 @Transient关键字声明， 还可以采用下面几种方法：
static Stringa;//静态变量不会被持久化
final String b = "name";//final变量不会被持久化
transient String c;//添加transient修饰的字段不会被持久化
```

## **8.5. 声明大字段**

`@Lob`:声明某个字段为大字段。

```java
@Lob
//    指定获取大对象的时机
    @Basic(fetch = FetchType.EAGER)
    @Column(name = "context",columnDefinition = "LONGTEXT NOT NULL")
    private String context;
```

获取对象方式的枚举类：

```java
public enum FetchType {

    /** Defines that data can be lazily fetched. */
    LAZY,//懒加载

    /** Defines that data must be eagerly fetched. */
    EAGER//及时加载
}
```

`columnDefinition`：指定数据表中那个字段对应的Lob字段。

## **8.6. 创建枚举类型的字段**

可以使用枚举类型的字段，不过枚举字段要用`@Enumerated`注解修饰。

```java
创建枚举类
public enum  Gender {
    MALE("男性"),
    FEMALE("女性");
    
    private String sex;

    Gender(String sex) {
        this.sex = sex;
    }
}

//    使用枚举类
    @Enumerated(EnumType.STRING)
    private Gender sex;
```

数据库里面对应存储的是 MAIL/FEMAIL。

## **8.7、删除/修改数据**

`@Modifying` 注解提示 JPA 该操作是修改操作,注意还要配合`@Transactional`注解使用。

```java
@Repository
public interface UserDao extends JpaRepository<UserRegister,Integer> {

    @Modifying
    @Transactional(rollbackFor = Exception.class)
    void deleteByUserNmme(String name);

}
```

# **9. 事务 `@Transactional`**

在要开启事务的方法上使用`@Transactional`注解即可!

```java
@Repository
public interface UserDao extends JpaRepository<UserRegister,Integer> {

    @Modifying
    @Transactional(rollbackFor = Exception.class)
    void deleteByUserNmme(String name);

}
```

我们知道 Exception 分为运行时异常 RuntimeException 和非运行时异常。在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

`@Transactional` 注解一般用在可以作用在`类`或者`方法`上。

- **作用于类**：当把`@Transactional 注解放在类上时，表示所有该类的`public 方法都配置相同的事务属性信息。
- **作用于方法**：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息。

# **10. json 数据处理**

## **10.1. 过滤 json 数据**

**`@JsonIgnoreProperties` 作用在类上用于过滤掉特定字段不返回或者不解析。**

```java
@Entity
@Table(name = "role")
@JsonIgnoreProperties({"UserRegister"})
public class Role {

    @Id
//    生成主键的策略
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Id
//    @GeneratedValue(strategy = "seqgenerate")
    @GenericGenerator(name = "seqgenerate",strategy = "identity")
    private String seq;

    private String name;

    @Column(name = "user_password",nullable = false,length = 32)
    private String userPassword;

    @Column(columnDefinition = "tinyint(1) default 1")
    private boolean enable;

    @Transient
    private String secrect;//不会持久化到数据库
//    除了 @Transient关键字声明， 还可以采用下面几种方法：
    static String a;//静态变量不会被持久化
    final String b = "name";//final变量不会被持久化
    transient String c;//添加transient修饰的字段不会被持久化

    @Lob
//    指定获取大对象的时机
    @Basic(fetch = FetchType.EAGER)
    @Column(name = "context",columnDefinition = "LONGTEXT NOT NULL")
    private String context;

//    使用枚举类
    @Enumerated(EnumType.STRING)
    private Gender sex;
    
    @JsonIgnore
    private List<UserRegister> userRegisters = new ArrayList<>();
    
}
```

**`@JsonIgnore`一般用于类的属性上，作用和上面的`@JsonIgnoreProperties` 一样。**

```java
//    生成json时将userRegisters属性过滤掉
    @JsonIgnore
    private List<UserRegister> userRegisters = new ArrayList<>();
```

## **10.2. 格式化 json 数据**

`@JsonFormat`一般用来格式化 json 数据。：

比如：

```java
@JsonFormat(shape = JsonFormat.Shape.SCALAR,pattern = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'",timezone = "GMT")
    private Date date;
```

测试代码：

```java
@Entity
@Table(name = "role")
@JsonIgnoreProperties({"UserRegister"})
public class Role {

    @Id
//    生成主键的策略
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Id
//    @GeneratedValue(strategy = "seqgenerate")
    @GenericGenerator(name = "seqgenerate",strategy = "identity")
    private String seq;

    private String name;

    @Column(name = "user_password",nullable = false,length = 32)
    private String userPassword;

    @Column(columnDefinition = "tinyint(1) default 1")
    private boolean enable;

    @Transient
    private String secrect;//不会持久化到数据库
//    除了 @Transient关键字声明， 还可以采用下面几种方法：
    static String a;//静态变量不会被持久化
    final String b = "name";//final变量不会被持久化
    transient String c;//添加transient修饰的字段不会被持久化

    @Lob
//    指定获取大对象的时机
    @Basic(fetch = FetchType.EAGER)
    @Column(name = "context",columnDefinition = "LONGTEXT NOT NULL")
    private String context;

//    使用枚举类
    @Enumerated(EnumType.STRING)
    private Gender sex;

//    生成json时将userRegisters属性过滤掉
    @JsonIgnore
    private List<UserRegister> userRegisters = new ArrayList<>();

    @JsonFormat(shape = JsonFormat.Shape.SCALAR,pattern = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'",timezone = "GMT")
    private Date date;

}
```

