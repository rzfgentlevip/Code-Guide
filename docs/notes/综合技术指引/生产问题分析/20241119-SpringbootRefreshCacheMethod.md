---
title: SpringBoot应用启动加载缓存实现
icon: material-icon-theme:supabase
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - SPRINGBOOT
tag:
  - spring
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:13:29
permalink: /compre-guide/production-issues/SpringbootRefreshCacheMethod/
---


# SpringBoot应用启动加载缓存实现

每天早上在启动springboot程序时候，要定时去数据库中加载有代表性利率债券，然后在盘中时候，根据有代表性利率债清单将有代表性的债券行情发送到CDMS数据发布系统上供会员使用，因此，本文总结一下在spring容器启动时，对缓存的几种初始化方式。

## 1、缓存定义

首先我们需要定义一个缓存，因为有代表性利率债清单只有很小一部分，因此我们选择内存缓存，并且在对有代表性行情的时候，我们只需要用到债券ID一个属性，因此我们在选择内存缓存的时候，使用Set数据结构存储债券ID,这样保证最小存储代价并且最快速的查找到债券ID，所以定义缓存对象如下：

```java
@Component
public class Cache {

    private static final Logger log = LoggerFactory.getLogger(Cache.class);

//    缓存结构
    private final Set<Integer> cache = new HashSet<>();

//    读写锁，防止线程不安全
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    private final  Lock readLock = lock.readLock();

    private final Lock writeLock = lock.writeLock();

//    初始化缓存
    public void initCache(){

        for(int i=0;i<3;i++){
            if(i%2 == 0){
                cache.add(i);
            }
        }
    }

//  缓存更新
    public boolean updateCache(int number){
        try {
            writeLock.lock();
            cache.add(number);
            System.out.print("数据更新成功，正在休眠.......缓存数据为:");
            cache.stream().forEach(System.out::println);
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            log.info("缓存数据更新失败:"+number);
            e.printStackTrace();
            return false;
        }finally {
            writeLock.unlock();
        }
        return true;
    }

//    读缓存判断
    public boolean isContain(int num){

        try {
            readLock.lock();
            if(cache.contains(num)){
                return true;
            }else {
                return false;
            }
        }finally {
            readLock.unlock();
        }
    }
}
```

下面介绍一下在springboot程序中，加载初始化Cache缓存的时机。

## 2、ContextRefreshedEvent事件

ContextRefreshedEvent：是Spring容器**初始化完成后**调用的事件。 ContextRefreshedEvent的父类是ApplicationContextEvent，是一个事件。所以我们通过ApplicationListener来实现。

```java
@Component
public class CacheInitAfterListener implements ApplicationListener<ContextRefreshedEvent> {
    
    private static final Logger log = LoggerFactory.getLogger(CacheInitAfterListener.class);

    @Autowired
    Tbs006Service tbs006Service;

    @Autowired
    Cache cache;
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {

//        进行缓存初始化
        log.info("正在进行缓存初始化.......");
        cache.initCache();
        tbs006Service = new Tbs006Service(cache);
        log.info("缓存初始化操作完成.......");
    }
}
```

## 3、PostConstruct 注解

PostConstruct注解修饰的方式，是在spring容器启动时运行的。优先级大于ContextRefreshedEvent事件。

## 4、InitializingBean

InitializingBean是spring容器在启动并初始化好内部示例后调用的，用来最终为总体bean添加最后属性和操作。

```java
官方原话：This method allows the bean instance to perform validation of its overall configuration and final initialization when all bean properties have been set.
```

------

## 5、init-method方法

这种方法有一定的局限性，并且可能会覆盖曾经的init操作，需要慎用。

Bean在加载到Spring容器中时需要先将Bean的定义信息抽象为BeanDefinition，其中有一个属性init-method代表将来Bean初始化时要调用的方法。

我们通过BeanFactoryPostProcessor来注入init-method方法，并且该方法必须是没有参数的。

## 6、实现 SmartInitializingSingleton 接口

SmartInitializingSingleton是Bean容器在初始化所有非懒加载的单例Bean后调用的方法。

## 7、重写 onRefresh()方法

## 8、CommandLineRunner（仅限Spring Boot）

CommandLineRunner 是一个Spring boot 接口，在应用初始化后执行，且仅会执行一次。可以用来打印项目中配置文件的参数，方便排查问题。

## 9、SpringApplicationRunListener（仅限Spring boot）

SpringBoot的生命周期事件监听方法，需要搭配resource/META-INF/spring.factories 文件使用。

执行优先级：init-Method >> InitializingBean >> PostConstruct >> SmartInitializingSingleton >> ContextRefreshedEvent >> SpringApplicationRunListener:started >> CommandLineRunner >> SpringApplicationRunListener:ready