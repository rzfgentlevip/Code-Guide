---
title: Springboot单元测试
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
permalink: /learning-notes/vn7eegyq/
---


# Springboot单元测试

# 1、JUnit5 的变化

官方文档 [https://junit.org/junit5/](https://junit.org/junit5/)

**Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库，以前用的是JUnit4。**

作为最新版本的JUnit框架，JUnit5与之前版本的Junit框架有很大的不同。由三个不同子项目的几个不同模块组成。

> **JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

**JUnit Platform**：是在JVM上启动测试**框架的基础**，不包括其它单元测试的引擎。

- 不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。
- JUnit未来，想让自己不仅是一个测试框架，更想成为一个测试平台，所以开发了Platform。这样其它测试框架可通过该平台接入JUnit中。

**JUnit Jupiter**：提供了JUnit5的新的编程模型，是**JUnit5新特性的核心**。内部包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage**： 由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了**兼容**JUnit4.x,Junit3.x的测试引擎。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.png)

## 1.1、Springboot2.4版本后Junit移除对Junit4的支持

**SpringBoot 2.4 及以上版本移除了默认对 JUnit Vintage引擎的依赖。2.3版本仍然支持JUnit Vintage。**

而JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎。

如果需要兼容junit4需要自行引入vintage，不然不能使用junit4的功能 @Test。

JUnit 5’s Vintage Engine Removed from `spring-boot-starter-test`

如果想在引用了JUnit5依赖的情况下，仍然支持JUnit4，那么可自行引入JUnit Vintage引擎的依赖。

```xml
spring2.4后默认不兼容junit4,如果想兼容junit4，需要自行引入引擎
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654312439473-bf10789b-58e2-4669-a33f-63b3a17241d7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654312439473-bf10789b-58e2-4669-a33f-63b3a17241d7.png)

官方信息来源：

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654311841721-f2411b54-8969-474d-8b2f-9ef91a2423ba.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654311841721-f2411b54-8969-474d-8b2f-9ef91a2423ba.png)

## 1.2、使用特点

SpringBoot整合Junit以后的使用特点：

- 编写测试方法：方法用 @Test标注（注意需要使用junit5版本的注解）

```java
// Junit5的@Test包为：junit-jupiter-api-5xxx.jar
import org.junit.jupiter.api.Test;
// Junit4的@Test包为：junit-4xx.jar
import org.junit.Test;

@SpringBootTest
class Boot05WebAdminApplicationTests {

    @Test
    void contextLoads() {

    }
}
```

- Junit测试类具有Spring的功能，

例如支持@Autowired、 @Transactional 标注测试方法，测试完成后自动回滚

```java
@SpringBootTest
class Boot05WebAdminApplicationTests {
    @Autowired
    JdbcTemplate jdbcTemplate;

    @Test
    @Transactional // 支持事务回滚。例如下面如果有insert操作，测试完了可以直接回滚。
    void contextLoads() {

        Long count = jdbcTemplate.queryForObject("SELECT COUNT(1) FROM T_ACCOUNT", Long.class);
        log.info("查询出的结果数量为:{}",count);

        log.info("导入的数据源类型:{}",dataSource.getClass());
    }
```

# 2、Junit使用方式

## 2.1、引入测试场景依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

## 2.2、场景包的依赖分析

当前演示为Springboot2.4版本。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654311411560-d3339a47-9d17-4c86-acf0-3709c1af1607.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654311411560-d3339a47-9d17-4c86-acf0-3709c1af1607.png)

## 2.3、测试案例

- 当前使用的springboot 2.4.0版本，编写测试类的模板：

```java
@SpringBootTest
class Boot05WebAdminApplicationTests {

    @Test
    void contextLoads() {

    }
}

```

- 以前版本的编写：

@SpringBootTest + @RunWith(SpringTest.class)

需要额外指定@RunWith

```java
@SpringBootTest
@RunWith(SpringJUnit4ClassRunner.class)
class Boot05WebAdminApplicationTests {

    @Test
    void contextLoads() {

    }
}

```

[https://blog.csdn.net/m0_45406092/article/details/119463761](https://blog.csdn.net/m0_45406092/article/details/119463761)

@RunWith 就是一个运行器：

@RunWith(JUnit4.class) 就是指用JUnit4来运行

@RunWith(SpringJUnit4ClassRunner.class),让测试运行于Spring测试环境

此时需要搭配@ContextConfiguration 使用，Spring整合JUnit4测试时，使用注解引入多个配置文件

@RunWith(Suite.class) 的话就是一套测试集合

springboot整合Junit之后：

1. 编写测试方法，标注@Test注解，这里需要哟个Junit5的版本
2. Junit类具有sprnig功能，@AutoWried,比如@Transactional标注测试方法，测试完成后自行回滚事务。

## 2.4、JUnit5常用注解

JUnit5的注解与JUnit4的注解有所变化

[https://junit.org/junit5/](https://junit.org/junit5/)

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654334739473-bfdb502e-eb95-4b95-a94e-a657a342b6e7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654334739473-bfdb502e-eb95-4b95-a94e-a657a342b6e7.png)

- **@Test :**表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest :**表示方法是参数化测试，下方会有详细介绍
- **@RepeatedTest :**表示方法可重复执行，下方会有详细介绍
- **@DisplayName :**为测试类或者测试方法设置展示名称
- **@BeforeEach :**表示在每个单元测试之前执行
- **@AfterEach :**表示在每个单元测试之后执行
- **@BeforeAll :**表示在所有单元测试之前执行
- **@AfterAll :**表示在所有单元测试之后执行
- **@Tag :**表示单元测试类别，用于灵活制定测试计划，按需过滤执行指定标签的测试单元，类似于JUnit4中的@Categories
- springboot2.4及之后版本可能需要安装额外依赖。

[https://blog.csdn.net/qq_44336097/article/details/116004990](https://blog.csdn.net/qq_44336097/article/details/116004990)

- **@Disabled :**表示测试类或测试方法不执行，类似于JUnit4中的@Ignore
- **@Timeout :**表示测试方法运行如果超过了指定时间将会返回错误
- java.util.concurrent.TimeoutException: test4() timed out after 500 milliseconds
- **@ExtendWith :**为测试类或测试方法提供扩展类引用
- 类似于JUnit4中的@RunWith方法，指定测试驱动。例如 @RunWith(SpringJUnit4ClassRunner.class),让测试运行于Spring测试环境
- 现在JUnit5中，用@ExtendWith替代@RunWith
- 而@SpringBootTest 作为复合注解，集成了
- @BootstrapWith(SpringBootTestContextBootstrapper.class)
- @ExtendWith({SpringExtension.class})
- **@SpringBootTest**：指定当前测试类为Springboot单元测试，具备Springboot容器的功能。例如自动注入。
- @SpringBootTest 作为复合注解，集成了：
- @BootstrapWith(SpringBootTestContextBootstrapper.class)
- @ExtendWith({SpringExtension.class})
- 一旦加上@SpringBootTest，测试启动后会启动Springboot容器的功能，日志中会看到springboot启动日志输出。
- 如果测试方法只是加了@Test，但是类上没加@SpringBootTest，测试方法依然正常执行。但是会导致自动注入的类无法起作用，获取的是NULL。

# 3、案例演示

```java
package com.learn.admin;

import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.annotation.Repeat;

import java.util.concurrent.TimeUnit;

/**
 * @Test :表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
 * @ParameterizedTest :表示方法是参数化测试，下方会有详细介绍
 * @RepeatedTest :表示方法可重复执行，下方会有详细介绍
 * @DisplayName :为测试类或者测试方法设置展示名称
 * @BeforeEach :表示在每个单元测试之前执行
 * @AfterEach :表示在每个单元测试之后执行
 * @BeforeAll :表示在所有单元测试之前执行
 * @AfterAll :表示在所有单元测试之后执行
 *      @BeforeAll 与 @AfterAll 会在所有方法之前后运行一次，需要标注为静态方法。
 * @Tag :表示单元测试类别，用于灵活制定测试计划，按需过滤执行指定标签的测试单元，类似于JUnit4中的@Categories
 *      springboot2.4及之后版本可能需要安装额外依赖。
 *      https://blog.csdn.net/qq_44336097/article/details/116004990
 * @Disabled :表示测试类或测试方法不执行，类似于JUnit4中的@Ignore
 * @Timeout :表示测试方法运行如果超过了指定时间将会返回异常
 *      java.util.concurrent.TimeoutException: test4() timed out after 500 milliseconds
 * @ExtendWith :为测试类或测试方法提供扩展类引用
 *      类似于JUnit4中的@RunWith方法，指定测试驱动。例如 @RunWith(SpringJUnit4ClassRunner.class),让测试运行于Spring测试环境
 *      现在JUnit5中，用@ExtendWith替代@RunWith
 *      @SpringBootTest 作为复合注解，集成了
 *          @BootstrapWith(SpringBootTestContextBootstrapper.class)
 *          @ExtendWith({SpringExtension.class})
 *
 *
 * @SpringBootTest ：指定当前测试类为Springboot单元测试，具备Springboot容器的功能。例如自动注入。
 *      @SpringBootTest 作为复合注解，集成了：
 *          @BootstrapWith(SpringBootTestContextBootstrapper.class)
 *          @ExtendWith({SpringExtension.class})
 *      一旦加上@SpringBootTest，测试启动后会启动Springboot容器的功能，日志中会看到springboot启动日志输出。
 *      如果测试方法只是加了@Test，但是类上没加@SpringBootTest，测试方法依然正常执行。但是会导致自动注入的类无法起作用，获取的是NULL。
 */
@DisplayName("junit5功能测试类")
@SpringBootTest
public class JUnit5Test {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @BeforeAll
    static void testBeforeAll() {
        System.out.println("testBeforeAll 在所有测试方法执行之前执行");
    }

    @BeforeEach
    void testBeforeEach() {
        System.out.println("testBeforeEach 在每个测试方法之前运行");
    }

    @DisplayName("test1方法")
    @Test
    void test1() {
        System.out.println("test1 执行");
    }

    @DisplayName("test2方法")
    @Test
    void test2() {
        System.out.println("test2 执行");
    }

    @Disabled
    @DisplayName("test3方法")
    @Test
    void test3() {
        System.out.println("test3 执行 测试 @DisplayName");
    }

    @Timeout(value = 500,unit = TimeUnit.MILLISECONDS)
    @DisplayName("test4方法")
    @Test
    void test4() throws InterruptedException {
        Thread.sleep(600);
        System.out.println("test4 执行 测试@Timeout 如果超过500毫秒，则不会运行");
    }

    @DisplayName("test5方法")
    @Test
    void test5() {
        System.out.println("test5 执行 测试类加与不加@SpringBootTest的情况下，jdbcTemplate自动注入的获取情况：" + jdbcTemplate);
    }

    @RepeatedTest(2)
    @DisplayName("test6方法")
    @Test
    void test6() {
        System.out.println("test6 执行 测试@RepeatedTest 重复执行");
    }

    @AfterEach
    void testAfterEach() {
        System.out.println("testAfterEach 在每个测试方法之后运行");
    }

    @AfterAll
    static void testAfterAll() {
        System.out.println("testAfterAll 表示在所有测试方法执行完之后执行");
    }

}

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654338550592-ce9caf1b-000e-475c-a169-99664f3b534b.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654338550592-ce9caf1b-000e-475c-a169-99664f3b534b.png)

# 4、断言（assertions）

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-assertions](https://junit.org/junit5/docs/current/user-guide/#writing-tests-assertions)

断言（assertions）是测试方法中的核心部分，也就是**断定某件事情一定会发生，如果没有发生，就认为出现了问题**。用来对测试需要满足的条件进行验证。

**这些断言方法都是 org.junit.jupiter.api.Assertions 的静态方法**。

JUnit 5 内置的断言可以分成如下几个类别：

**检查业务逻辑返回的数据是否合理。**

- 断言的好处：

以往检查代码逻辑是否正确，直接在测试类中进行打印输出结果。

但通过断言这种标准做法，在项目**上线之前**，通过maven clean+maven test，批量运行测试类进行检查，就能集中看出哪些单元测试没有通过，**所有的测试运行结束以后，会有一个详细的测试报告，**方便集中排查问题。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654357583221-dfad6869-0171-4a44-9230-a463be04c858.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654357583221-dfad6869-0171-4a44-9230-a463be04c858.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654533300568-4fd6c962-8072-4dc6-bc4b-841d34579b5b.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654533300568-4fd6c962-8072-4dc6-bc4b-841d34579b5b.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654533109770-72658001-0962-492e-a514-eaa67b7abe24.png](https://cdn.nlark.com/yuque/0/2022/png/623390/1654533109770-72658001-0962-492e-a514-eaa67b7abe24.png)

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654533144305-b957c02e-5416-47bd-87be-2261b0385d59.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654533144305-b957c02e-5416-47bd-87be-2261b0385d59.png)

## 4.1、简单断言

用来对单个值进行简单的验证。如：

| 方法            | 说明                                 |
| --------------- | ------------------------------------ |
| assertEquals    | 判断两个对象或两个原始类型是否相等   |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame      | 判断两个对象引用是否指向同一个对象   |
| assertNotSame   | 判断两个对象引用是否指向不同的对象   |
| assertTrue      | 判断给定的布尔值是否为 true          |
| assertFalse     | 判断给定的布尔值是否为 false         |
| assertNull      | 判断给定的对象引用是否为 null        |
| assertNotNull   | 判断给定的对象引用是否不为 null      |

```java
    int count(int a, int b) {
        return a + b;
    }

    @DisplayName("测试简单断言")
    @Test
    void testSimpleAssertions() {
        // 期望值  实际值
        //Assertions.assertEquals(5, count(2, 3));
        // 期望值  实际值  如果与预期不符，打印指定消息
        //Assertions.assertEquals(5, count(3, 3),"值不为5");

        Object obj1 = new Object();
        Object obj2 = new Object();
        Assertions.assertSame(obj1, obj2, "两个对象不一样");

    }
```

- 无message的效果

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654415945290-9d561154-e86f-4262-b31c-0cf090599166.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654415945290-9d561154-e86f-4262-b31c-0cf090599166.png)

- 有message的效果

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654416262308-fd6ca4c7-5e93-46a5-b467-c6f79b5caec7.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654416262308-fd6ca4c7-5e93-46a5-b467-c6f79b5caec7.png)

## 4.2、数组断言

通过 assertArrayEquals 方法来判断两个“对象或原始类型”的数组是否相等

```java
    @DisplayName("测试数组断言")
    @Test
    void testArrayAssertions() {
        //Assertions.assertArrayEquals(new int[]{1, 2}, new int[]{1, 2});
        Assertions.assertArrayEquals(new int[]{1, 2}, new int[]{1, 3}, "两个数组内容不相等");
    }
```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654416938425-4db7d4c9-15f6-4283-a818-14cf86b2dac3.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654416938425-4db7d4c9-15f6-4283-a818-14cf86b2dac3.png)

## 4.3、组合断言

assertAll 方法

简单来说，组合断言就是一组断言，组内所有断言都成功，该组合断言才算成功。

接受多个 org.junit.jupiter.api.Executable 函数式接口的实例作为要验证的断言，可以通过 lambda 表达式很容易的提供这些断言

- 使用接口说明

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654417257239-37f93f89-197e-45dd-9732-31736c1a0954.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654417257239-37f93f89-197e-45dd-9732-31736c1a0954.png)

其中Executable接口，表示一个可执行的被断言的方法。

```java
@FunctionalInterface
@API(
    status = Status.STABLE,
    since = "5.0"
)
public interface Executable {
    void execute() throws Throwable;  // 函数式接口,无传入参数，无返回参数
}
```

- 案例

```java
    @DisplayName("测试组合断言")
    @Test
    public void testAssertAll() {
        /**
         * 所有断言全部需要成功
         */
        Assertions.assertAll("这俩组合的断言有问题",
                () -> Assertions.assertTrue(true && true, "两个值不全是true"),
                () -> Assertions.assertEquals(1, 2, "两个数不全是1"));

        System.out.println("断言都成功了,我才会打印出来");
    }

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654531372742-b8b2d0b8-4c2b-4877-a70e-98a47f556bb6.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654531372742-b8b2d0b8-4c2b-4877-a70e-98a47f556bb6.png)

## 4.4、异常断言

在JUnit4时期，想要测试方法的异常情况时，需要用**@Rule**注解的ExpectedException变量还是比较麻烦的。

JUnit5提供了一种新的断言方式**Assertions.assertThrows()** ,配合函数式编程就可以进行使用。

表示预期会出现指定异常，如果没有出现，则抛出一个断言异常。

```java
    @DisplayName("测试异常断言")
    @Test
    public void testAssertThrows() {
        /**
         *  测试断言指定的异常会出现，出现了，就正常，程序继续执行。
         *  par1 预期的异常
         *  par2 函数式声明被断言的方法
         *  par3 断言指定的异常没有出现，就会输出的异常信息
         */
        ArithmeticException exception = Assertions.assertThrows(
                //扔出断言异常
                ArithmeticException.class, () -> System.out.println(1 % 1),"预期会抛出的异常没有正常抛出，有问题了");

    }
```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654532137327-690771f4-6edc-4a58-9389-064a29da53a2.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654532137327-690771f4-6edc-4a58-9389-064a29da53a2.png)

## 4.5、超时断言

Junit5还提供了**Assertions.assertTimeout()** 为测试方法设置了超时时间。

预期指定的时间内一定超时。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654532262625-0a648fe0-7ecc-4590-92e8-4dc1db3ed8d6.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654532262625-0a648fe0-7ecc-4590-92e8-4dc1db3ed8d6.png)

```java
    @DisplayName("测试超时断言")
    @Test
    public void testAssertTimeout() {
        //
        /**
         * assertTimeout 设定了断言方法能执行的最大时间，超过最大时间，会超时
         * par1 最大时间
         * par2 断言的方法
         * par3 超时后的异常信息
         */
        Assertions.assertTimeout(Duration.ofMillis(100), () -> Thread.sleep(1000),"程序执行没有超时指定的时间");
    }
```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654532488065-5977985f-e24a-490e-8298-7d92733db7dc.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654532488065-5977985f-e24a-490e-8298-7d92733db7dc.png)

## 4.6、快速失败

通过 fail 方法直接使得测试失败

```java
    @DisplayName("测试快速失败")
    @Test
    public void testFail() {
        // ... 一些测试内容都执行完毕
        if(1==1){
            // 直接返回测试失败，程序停止
            Assertions.fail("测试失败，不测了");
        }
    }
```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654532638578-47580f97-472b-4af8-b5ed-a4ae18d8887e.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654532638578-47580f97-472b-4af8-b5ed-a4ae18d8887e.png)

# 5、前置条件（assumptions）

JUnit 5 中的前置条件（**assumptions【假设】**）类似于断言，

不同之处在于：

**不达到预期效果的断言会使得测试方法失败**，

而不满足预期效果的**前置条件只会使得测试方法的执行终止**。

前置条件可以看成是测试方法执行的前提，当该前提不满足时，就没有继续执行的必要。

- 前置条件不成立时，报告会判定该测试方法为跳过。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654533959032-e6f76975-1e60-4a90-95b1-3ed37a3704a6.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654533959032-e6f76975-1e60-4a90-95b1-3ed37a3704a6.png)

- 演示

```java
    @DisplayName("测试前置条件")
    @Test
    void testAssumptions(){
        Assumptions.assumeTrue(false,"假设不成立，出现异常"); // 假设为true才能继续运行下面的代码
        System.out.println("假设成立，后续代码正常执行");
    }
```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654534091301-d09230d0-1fc8-44f3-9366-898dd48f89d8.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654534091301-d09230d0-1fc8-44f3-9366-898dd48f89d8.png)

```java
@DisplayName("前置条件")
public class AssumptionsTest {
 private final String environment = "DEV";

 @Test
 @DisplayName("simple")
 public void simpleAssume() {
    assumeTrue(Objects.equals(this.environment, "DEV"));
    assumeFalse(() -> Objects.equals(this.environment, "PROD"));
 }

 @Test
 @DisplayName("assume then do")
 public void assumeThenDo() {
    assumingThat(
       Objects.equals(this.environment, "DEV"),
       () -> System.out.println("In DEV")
    );
 }
}
```

assumeTrue 和 assumFalse 确保给定的条件为 true 或 false，不满足条件会使得测试执行终止。assumingThat 的参数是表示条件的布尔值和对应的 Executable 接口的实现对象。只有条件满足时，Executable 对象才会被执行；当条件不满足时，测试执行并不会终止。

# 6、嵌套测试

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested](https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested)

JUnit 5 可以通过 Java 中的内部类和@Nested 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。

在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。

嵌套测试基本原则：嵌套测试情况下，

外层test方法不能驱动内层的标注了@BeforeEach、@BeforeAll之类的方法，在外层方法之前或之后运行。

但内层test方法能驱动外层层的标注了@BeforeEach、@BeforeAll之类的方法。

```java
package com.learn.admin;

import org.junit.jupiter.api.*;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.EmptyStackException;
import java.util.Stack;

@DisplayName("嵌套测试")
@SpringBootTest
public class JUnit5AStackTest {

    // 栈
    Stack<Object> stack;

    @Test
    @DisplayName("new Stack()")
    void isInstantiatedWithNew() {
        /**
         * 嵌套测试情况下，外层test方法不能驱动内层的标注了@BeforeEach、@BeforeAll之类的方法，在外层方法之前或之后运行
         */
        new Stack<>(); // 没有实际赋给谁，所以没有作用
        // 单独运行当前方法时，不会触发嵌套的方法，所以stack是Null
        Assertions.assertNull(stack);
    }

    @Nested // 表示嵌套测试
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            // stack会被createNewStack初始化
            // 这时，isEmpty 只是内容为空，但stack不为null
            Assertions.assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            // pop 方法代表从栈中弹出一个元素出来，移除它。但是stack的内容为空，所以一定会抛出EmptyStackException异常
            Assertions.assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            // peek 查看栈中的第一个元素
            Assertions.assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                /**
                 * 嵌套测试情况下，内层test方法能驱动外层层的标注了@BeforeEach、@BeforeAll之类的方法
                 */
                // 当前方法单独运行时，触发了外层的前置、后置方法。
                Assertions.assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                Assertions.assertEquals(anElement, stack.pop());
                Assertions.assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                Assertions.assertEquals(anElement, stack.peek());
                Assertions.assertFalse(stack.isEmpty());
            }
        }
    }
}

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654597156930-bff3dde2-b50d-4fb0-8fb8-a0707e340b38.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654597156930-bff3dde2-b50d-4fb0-8fb8-a0707e340b38.png)

# 6、参数化测试

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)

参数化测试是JUnit5很重要的一个新特性，它使得用不同的参数多次运行测试成为了可能，也为我们的单元测试带来许多便利。

利用**@ValueSource**等注解，指定入参，我们将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。

**@ValueSource**: 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型

**@NullSource**: 表示为参数化测试提供一个null的入参

**@EnumSource**: 表示为参数化测试提供一个枚举入参

**@CsvFileSource**：表示读取指定CSV文件内容作为参数化测试入参

**@MethodSource**：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)

当然如果参数化测试仅仅只能做到指定普通的入参还达不到让我觉得惊艳的地步。让我真正感到他的强大之处的地方在于他可以支持外部的各类入参。如:CSV,YML,JSON 文件甚至方法的返回值也可以作为入参。只需要去实现**ArgumentsProvider**接口，任何外部文件都可以作为它的入参。

```sql
package com.learn.admin;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;
import org.junit.jupiter.params.provider.ValueSource;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.stream.Stream;

@DisplayName("参数化测试")
@SpringBootTest
public class JUnit5ParameterTest {

    /**
     * @ValueSource(ints = {1,2,3,4,5}) 指定方法参数的数据来源
     */
    @ParameterizedTest
    @ValueSource(ints = {1,2,3,4,5})
    @DisplayName("@ValueSource 普通数据入参测试")
    void testParameterized(int i){
        System.out.println(i);
    }

    static Stream<String> stringProvider(){
        return Stream.of("s1","s2");
    }

    @ParameterizedTest
    @MethodSource("stringProvider")
    @DisplayName("@MethodSource 测试")
    void testParameterized(String i){
        System.out.println(i);
    }

}

```

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654597880812-b5049b08-a62a-4b59-9b71-da7bdccf1ad4.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654597880812-b5049b08-a62a-4b59-9b71-da7bdccf1ad4.png)

# 8、JUnit4迁移至JUnit5指南

[https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4](https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4)

5要兼容4，要导入junit-vintage-engine引擎。

```sql
        <dependency>
            <groupId>org.junit.vintage</groupId>
            <artifactId>junit-vintage-engine</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.hamcrest</groupId>
                    <artifactId>hamcrest-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

在进行迁移的时候需要注意如下的变化：

- 注解在 org.junit.jupiter.api 包中，
- 断言在 org.junit.jupiter.api.Assertions 类中，前置条件在 org.junit.jupiter.api.Assumptions 类中。
- 把@Before 和@After 替换成@BeforeEach 和@AfterEach。
- 把@BeforeClass 和@AfterClass 替换成@BeforeAll 和@AfterAll。
- 把@Ignore 替换成@Disabled。
- 把@Category 替换成@Tag。
- 把@RunWith、@Rule 和@ClassRule 替换成@ExtendWith。

![https://cdn.nlark.com/yuque/0/2022/png/623390/1654598090635-014aeb37-8d76-42a1-b2d3-464fe582b53a.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1654598090635-014aeb37-8d76-42a1-b2d3-464fe582b53a.png)

