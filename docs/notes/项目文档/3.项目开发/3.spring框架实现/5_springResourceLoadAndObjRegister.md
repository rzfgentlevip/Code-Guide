---
title: 5、资源加载解析和对象注册
icon: file
order: 5
author: bugcode
date: 2020-01-01T00:00:00.000Z
category:
  - SPRING
  - SPRINGBOOT
  - JAVA
tag:
  - 后端
  - java
  - spring
sticky: false
star: true
footer: Spring基础
copyright: bugcode
createTime: 2026/01/17 20:14:52
permalink: /project-docs/project/springframework/资源加载解析和对象注册/
---

<!-- TOC -->

- [第五章、资源加载器解析文件并注册对象](#第五章资源加载器解析文件并注册对象)
  - [1、目标](#1目标)
  - [2、设计](#2设计)
  - [3、实现](#3实现)
    - [3.1、核心类图](#31核心类图)
    - [3.2、资源加载接口](#32资源加载接口)
    - [3.3、包装资源加载器类](#33包装资源加载器类)
    - [3.4、Bean定义读取接口](#34bean定义读取接口)
    - [3.5、Bean定义读取抽象类实现](#35bean定义读取抽象类实现)
    - [3.6、解析XML资源文件注册Bean](#36解析xml资源文件注册bean)
  - [4、测试](#4测试)
    - [4.1、测试用例](#41测试用例)
    - [4.2、测试结果](#42测试结果)
  - [5、总结](#5总结)

<!-- /TOC -->

# 第五章、资源加载器解析文件并注册对象

## 1、目标

容器管理bean对象的生命周期，就应该给用户提供黑盒的使用方法，用户不必关心bean对象注入容器的过程，而只关心bean对象是否在容器中，是否能使用即可，尽量少的让用户干预bean对象的创建。

第四章中，虽然我们解决了bean对象定义的注入，bean对象的实例化以及属性赋值，但是观察整个过程，其实还是需要程序干预去注册bean对象和设置属性值，而这一部分又是比较固定的流程，因此可以将这一部分封装成黑盒子，只对用户提供服务，让用户无法感知到这一部分代码的存在。

目前实现bean生命周期中的功能由有：注册bean对象，实例化bean对象，给bean对象设置属性

在spring中，通常用户对bean的定义是写在xml配置文件中，用户只需要简单的配置，spring就可以将所有相互依赖的bean注入容器中，因此我们的spring也应该提供这样的功能，读取xml配置然后将用户配置的bean注入到容器中，并且整个过程不需要让用户感知到spring帮助我们做了什么事情。

## 2、设计

这一部分功能相对来说比较明确，spring读取用户配置的xml文件，然后解析xml文件，将xml文件中的bean定义以及对象读取到容器中。这样的功能就比较独立，我们可以将他作为一个模块进行设计，然后将整个模块集成到现有的spring容器中。

<img src="https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408021003138.png" alt="step03-design-实例化bean带参数-xml文件读取解析.drawio.png" style="zoom:80%;" />

- 资源加载器属于相对独立的部分，它位于 Spring 框架核心包下的IO实现内容，主要用于处理Class、本地和云环境中的文件信息。
- **当资源可以加载后，接下来就是解析和注册 Bean 到 Spring 中的操作，这部分实现需要和 DefaultListableBeanFactory 核心类结合起来，因为你所有的解析后的注册动作，都会把 Bean 定义信息放入到这个类中**。
- 那么在实现的时候就设计好接口的实现层级关系，包括我们需要定义出 Bean 定义的读取接口 BeanDefinitionReader 以及做好对应的实现类，在实现类中完成对 Bean 对象的解析和注册。

## 3、实现

### 3.1、核心类图

![spring-application-xmlRegisterAndLoading.drawio.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408021005712.png)

要想将bean对象的定义放在xml文件中然后让spring自动去读取xml文件然后实现bean的注册和初始化，需要实现两大模块功能，**资源文件加载器，xml资源处理器**。

资源加载实现的主要接口是Resources和ResourceLoader两个接口，资源主要通过实现Resources的getInputStream实现资源文件的读取操作，其中包括了**classpath下的文件，系统文件以及远程配置文件**三种。

接口：BeanDefinitionReader、抽象类：AbstractBeanDefinitionReader、实现类：XmlBeanDefinitionReader，这三部分内容主要是合理清晰的处理了资源读取后的注册 Bean 容器操作。

接口负责定义，抽象类处理非接口功能外的注册Bean组件填充，最终实现类即可只关心具体的业务实现

**接口扩展**

- BeanFactory,已经存在的 Bean 工厂接口用于获取 Bean 对象，这次新增加了按照类型获取 Bean 的方法:`<T> T getBean(String name, Class<T> requiredType)`
- ListableBeanFactory，是一个扩展 Bean 工厂接口的接口，新增加了getBeansOfType、getBeanDefinitionNames()方法，在 Spring 源码中还有其他扩展方法。
- HierarchicalBeanFactory，在 Spring 源码中它提供了可以获取父类 BeanFactory 方法，属于是一种扩展工厂的层次子接口。Sub-interface implemented by bean factories that can be part of a hierarchy.
- AutowireCapableBeanFactory，是一个自动化处理Bean工厂配置的接口，目前案例工程中还没有做相应的实现，后续逐步完善。
- ConfigurableBeanFactory，可获取 BeanPostProcessor、BeanClassLoader等的一个配置化接口。
- ConfigurableListableBeanFactory，提供分析和修改Bean以及预先实例化的操作接口，不过目前只有一个 getBeanDefinition 方法。

### 3.2、资源加载接口

```java
/**
 * @Description 获取xml资源的总接口，有多个不同的资源加载，都实现此类即可
 * @Author bugcode.online
 * @Date 2024/5/16 19:39
 */
public interface Resource {

    /**
     * 读取xml文件获取输入流
     * @return
     * @throws IOException
     */
    InputStream getInputStream() throws IOException;
}
```

Resource接口，只负责外部xml资源加载工作，定义获取bean定义资源的总接口，getInputStream方法获取文件输入流，实现不同方法加载资源文件，classpath,FileSystem,URL；

**classpath加载资源文件**

```java
/**
 * @Description
 * @Author bugcode.online
 * @Date 2024/5/16 19:40
 */
public class ClassPathResource implements Resource {

    private final String path;

    private ClassLoader classLoader;

    public ClassPathResource(String path) {
        this(path, (ClassLoader) null);
    }

    public ClassPathResource(String path, ClassLoader classLoader) {
        Assert.notNull(path, "Path must not be null");
        this.path = path;
//        获取类加载器
        this.classLoader = (classLoader != null ? classLoader : ClassUtils.getDefaultClassLoader());
    }
    
    /**
     * 实现接口定义的资源加载方法
     * @author yourname
     * @date 20:19 2024/11/23  
     * @return java.io.InputStream
     **/
    @Override
    public InputStream getInputStream() throws IOException {
        InputStream is = classLoader.getResourceAsStream(path);
        if (is == null) {
            throw new FileNotFoundException(
                    this.path + " cannot be opened because it does not exist");
        }
        return is;
    }
}
```

获取类路径下的资源文件，这一部分是通过类加载器实现读取classpath路径下的资源文件。

**文件系统资源文件**

```java
public class FileSystemResource implements Resource{

    private final File file;

    private final String path;


    public FileSystemResource(File file) {
        this.file = file;
        this.path = file.getPath();
    }

    public FileSystemResource(String path) {
        this.file = new File(path);
        this.path = path;
    }

    @Override
    public InputStream getInputStream() throws IOException {
        return new FileInputStream(path);
    }

    public String getPath() {
        return path;
    }
}
```

通过指定资源文件的路径读取bean信息。

**远程资源文件**

```java
public class UrlResource implements Resource {
    private final URL url;

    public UrlResource(URL url) {
        Assert.notNull(url,"URL must not be null");
        this.url = url;
    }

    @Override
    public InputStream getInputStream() throws IOException {
        URLConnection urlConnection = this.url.openConnection();
        try {
            return urlConnection.getInputStream();
        }catch (IOException ex){
            if(urlConnection instanceof HttpURLConnection){
                ((HttpURLConnection)urlConnection).disconnect();
            }
            throw ex;
        }
    }
}
```

通过http方式读取远程的资源文件。

### 3.3、包装资源加载器类

按照加载资源的方式不同，资源加载器客户已把这些方式集中统一的类服务下面进行处理，外部用户只需要传递具体的资源地址即可。

```java
public interface ResourceLoader {

    /**
     * Pseudo URL prefix for loading from the class path: "classpath:"
     */
    String CLASSPATH_URL_PREFIX = "classpath:";

    Resource getResource(String location);

}
```

**默认的资源加载实现**

```java
public class DefaultResourceLoader implements ResourceLoader {
    @Override
    public Resource getResource(String location) {
        Assert.notNull(location, "Location must not be null");

        if(location.startsWith(CLASSPATH_URL_PREFIX)){
            return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()));
        }else {
            try {
                URL url = new URL(location);
                return new UrlResource(url);
            }catch (MalformedURLException e){
                return new FileSystemResource(location);
            }
        }
    }
}
```

获取资源的实现中，对三种类型资源的获取进行包装判断，调用不同的具体方法获取资源。

### 3.4、Bean定义读取接口

```java
/**
 * @Description 定义读取Bean定义的接口 基于xml文件流解析读取
 * @Author bugcode.online
 * @Date 2024/5/16 19:59
 */
public interface BeanDefinitionReader {

    BeanDefinitionRegistry getRegistry();

    /**
     * 获取资源加载器
     * @author yourname
     * @date 20:22 2024/11/23  
     * @return bugcode.online.springframework.core.io.ResourceLoader
     **/
    ResourceLoader getResourceLoader();

    /**
     * 架子啊Bean的定义
     * @author yourname
     * @date 20:21 2024/11/23 
     * @param resource 
     **/
    void loadBeanDefinitions(Resource resource) throws BeansException;

    void loadBeanDefinitions(Resource... resources) throws BeansException;

    void loadBeanDefinitions(String location) throws BeansException;
}
```

- 这是一个 *Simple interface for bean definition readers.* 其实里面无非定义了几个方法，包括：getRegistry()、getResourceLoader()，以及三个加载Bean定义的方法。
- 这里需要注意 getRegistry()、getResourceLoader()，都是用于提供给后面三个方法的工具，加载和注册，这两个方法的实现会包装到抽象类中，以免污染具体的接口实现方法

- BeanDefinitionReader:顶级资源加载，类注册接口

- AbstractBeanDefinitionReader：抽象类实现顶级接口，定义业务逻辑的调用过程

- XmlBeanDefinitionReader：提供给用户的最底层接口，可以根据配置文件进行注册。

### 3.5、Bean定义读取抽象类实现

```java
public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader{

    private final BeanDefinitionRegistry registry;

    private ResourceLoader resourceLoader;

    protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry){
        this(registry,new DefaultResourceLoader());
    }

    public AbstractBeanDefinitionReader(BeanDefinitionRegistry registry, ResourceLoader resourceLoader) {
        this.registry = registry;
        this.resourceLoader = resourceLoader;
    }

    @Override
    public BeanDefinitionRegistry getRegistry() {
        return registry;
    }

    @Override
    public ResourceLoader getResourceLoader() {
        return resourceLoader;
    }
}
```

抽象类把 BeanDefinitionReader 接口的前两个方法全部实现完了，并提供了构造函数，让外部的调用使用方，把Bean定义注入类，传递进来；

这样在接口 BeanDefinitionReader 的具体实现类中，就可以把解析后的 XML 文件中的 Bean 信息，注册到 Spring 容器去了。*以前我们是通过单元测试使用，调用 BeanDefinitionRegistry 完成Bean的注册，现在可以放到 XMl 中操作了*

### 3.6、解析XML资源文件注册Bean

```java
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {

    public XmlBeanDefinitionReader(BeanDefinitionRegistry registry){
        super(registry);
    }

    public XmlBeanDefinitionReader(BeanDefinitionRegistry registry, ResourceLoader resourceLoader) {
        super(registry, resourceLoader);
    }


    @Override
    public void loadBeanDefinitions(Resource resource) throws BeansException {
        try (InputStream inputStream = resource.getInputStream()){
            doLoadBeanDefinitions(inputStream);
        } catch (IOException | ClassNotFoundException e) {
            throw new BeansException("IOException parsing XML document from " + resource, e);
        }
    }

    @Override
    public void loadBeanDefinitions(Resource... resources) throws BeansException {
        for (Resource resource : resources) {
            loadBeanDefinitions(resource);
        }
    }

    @Override
    public void loadBeanDefinitions(String location) throws BeansException {
        ResourceLoader resourceLoader = getResourceLoader();
        Resource resource = resourceLoader.getResource(location);
        loadBeanDefinitions(resource);
    }

    /**
     *公共的加载xml文件的方法
     * @param inputStream
     * @throws ClassNotFoundException
     */
    protected void doLoadBeanDefinitions(InputStream inputStream) throws ClassNotFoundException {
        Document doc = XmlUtil.readXML(inputStream);
        Element root = doc.getDocumentElement();
        NodeList childNodes = root.getChildNodes();

        for (int i = 0; i < childNodes.getLength(); i++) {
            // 判断元素
            if (!(childNodes.item(i) instanceof Element)) continue;
            // 判断对象
            if (!"bean".equals(childNodes.item(i).getNodeName())) continue;

            // 解析标签
            Element bean = (Element) childNodes.item(i);
            String id = bean.getAttribute("id");
            String name = bean.getAttribute("name");
            String className = bean.getAttribute("class");
            // 获取 Class，方便获取类中的名称
            Class<?> clazz = Class.forName(className);
            // 优先级 id > name
            String beanName = StrUtil.isNotEmpty(id) ? id : name;
            if (StrUtil.isEmpty(beanName)) {
                beanName = StrUtil.lowerFirst(clazz.getSimpleName());
            }

            // 定义Bean
            BeanDefinition beanDefinition = new BeanDefinition(clazz);
            // 读取属性并填充
            for (int j = 0; j < bean.getChildNodes().getLength(); j++) {
                if (!(bean.getChildNodes().item(j) instanceof Element)) continue;
                if (!"property".equals(bean.getChildNodes().item(j).getNodeName())) continue;
                // 解析标签：property
                Element property = (Element) bean.getChildNodes().item(j);
                String attrName = property.getAttribute("name");
                String attrValue = property.getAttribute("value");
                String attrRef = property.getAttribute("ref");
                // 获取属性值：引入对象、值对象
                Object value = StrUtil.isNotEmpty(attrRef) ? new BeanReference(attrRef) : attrValue;
                // 创建属性信息
                PropertyValue propertyValue = new PropertyValue(attrName, value);
                beanDefinition.getPropertyValues().addPropertyValue(propertyValue);
            }
            if (getRegistry().containsBeanDefinition(beanName)) {
                throw new BeansException("Duplicate beanName[" + beanName + "] is not allowed");
            }
            // 注册 BeanDefinition
            getRegistry().registerBeanDefinition(beanName, beanDefinition);
        }
    }
}
```

XmlBeanDefinitionReader是具体的配置文件读取类，其中最主要的方法是doLoadBeanDefinitions，获取xml文件然后解析xml文件，然后将解析的bean结构信息注册到Bean容器中。

XmlBeanDefinitionReader 类最核心的内容就是对 XML 文件的解析，把我们本来在代码中的操作放到了通过解析 XML 自动注册的方式。

- loadBeanDefinitions 方法，处理资源加载，这里新增加了一个内部方法：doLoadBeanDefinitions，它主要负责解析 xml
- 在 doLoadBeanDefinitions 方法中，主要是对xml的读取 XmlUtil.readXML(inputStream) 和元素 Element 解析。在解析的过程中通过循环操作，以此获取 Bean 配置以及配置中的 id、name、class、value、ref 信息。
- 最终把读取出来的配置信息，创建成 BeanDefinition 以及 PropertyValue，最终把完整的 Bean 定义内容注册到 Bean 容器：getRegistry().registerBeanDefinition(beanName, beanDefinition)

## 4、测试

### 4.1、测试用例

**DAO定义**

```java
public class PeopleDao {

    private static Map<String,String> map = new HashMap<>();

    static {
        map.put("10001", "工程师");
        map.put("10002", "秘书");
        map.put("10003", "实习生");
    }

    public String queryUserName(String uId) {
        return map.get(uId);
    }
}
```

**Service定义**

```java
public class PeopleService {
    private String id;

    private PeopleDao peopleDao;

    public void queryUserInfo() {
        System.out.println("查询用户信息：" + peopleDao.queryUserName(id));
    }
}
```

**xml文件定义**

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    //注入Dao对象
    <bean id="peopleDao" class="bugcode.online.springframework.bean.PeopleDao"/>
    //依赖注入基础属性和引用属性
    <bean id="peopleService" class="bugcode.online.springframework.bean.PeopleService">
        <property name="id" value="10001"/>
        <property name="peopleDao" ref="peopleDao"/>
    </bean>

</beans>
```

在配置文件总首先配置DAO对象，然后配置Service对象，依赖DAO对象，这样就实现了对象的依赖注入。

### 4.2、测试结果

```java
public class TestBeanDefine {

    private DefaultResourceLoader resourceLoader;

    @Before
    public void init(){
        resourceLoader = new DefaultResourceLoader();


    }

    @Test
    public void test_classpath() throws IOException {
        Resource resource = resourceLoader.getResource("classpath:important.properties");
        InputStream inputStream = resource.getInputStream();
        String content = IoUtil.readUtf8(inputStream);
        System.out.println(content);
    }

    @Test
    public void test_file() throws IOException {
        Resource resource = resourceLoader.getResource("src/test/resources/important.properties");
        InputStream inputStream = resource.getInputStream();
        String content = IoUtil.readUtf8(inputStream);
        System.out.println(content);
    }

    @Test
    public void test_url() throws IOException {
        Resource resource = resourceLoader.getResource("https://github.com/fuzhengwei/small-spring/important.properties");
        InputStream inputStream = resource.getInputStream();
        String content = IoUtil.readUtf8(inputStream);
        System.out.println(content);
    }

    @Test
    public void test_xml() {
        // 1.初始化 BeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2. 读取配置文件&注册Bean 这一步自动读取xml文件，将文件中定义的bean对象加载到容器中
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        reader.loadBeanDefinitions("classpath:spring.xml");

        // 3. 获取Bean对象调用方法
        PeopleService peopleService = (PeopleService)beanFactory.getBean("peopleService", PeopleService.class);
        peopleService.queryUserInfo();
    }
}
```

测试结果：

```java
查询用户信息：工程师

Process finished with exit code 0
```

## 5、总结

1. 第五章注意设计点，读取不同xml配置文件接口和实现类的设计,采用策略模式设计，因为可以从不同的位置读取资源文件，有多种途径，因此可以借助策略模式实现；

2. BeanDefinitionReader加载资源并且注册对象的设计，如何将自动去取xml配置串流到对象注册的过程中。