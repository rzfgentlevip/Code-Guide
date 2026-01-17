---
title: 2、动态线程池Starter功能实现
icon: file
order: 2
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - project
tag:
  - project
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2026/01/17 19:32:41
permalink: /project-docs/project/threadPool/动态线程池Starter功能实现/
---
<!-- TOC -->

- [动态线程池Starter功能实现](#动态线程池starter功能实现)
  - [动态线程池基本功能实现](#动态线程池基本功能实现)
    - [线程池配置实体对象](#线程池配置实体对象)
    - [线程池服务类接口](#线程池服务类接口)
    - [线程池服务接口实现](#线程池服务接口实现)
    - [自动配置功能实现](#自动配置功能实现)
    - [自动配置核心类实现](#自动配置核心类实现)
    - [上报配置中心实现](#上报配置中心实现)
    - [定时上报功能实现](#定时上报功能实现)
    - [发布者订阅者实现](#发布者订阅者实现)

<!-- /TOC -->
# 动态线程池Starter功能实现

## 动态线程池基本功能实现

### 线程池配置实体对象

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ThreadPoolConfigEntity {

    /**
     * 应用名称
     */
    private String appName;

    /**
     * 线程池名称
     */
    private String threadPoolName;

    /**
     * 核心线程数
     */
    private int corePoolSize;

    /**
     * 最大线程数
     */
    private int maximumPoolSize;

    /**
     * 当前活跃线程数
     */
    private int activeCount;

    /**
     * 当前池中线程数
     */
    private int poolSize;

    /**
     * 队列类型
     */
    private String queueType;

    /**
     * 当前队列任务数
     */
    private int queueSize;

    /**
     * 队列剩余任务数
     */
    private int remainingCapacity;
    
}
```

线程池实体配置对象主要包括一些核心的配置，包括：

- 核心线程书
- 当前活跃线程数
- 最大线程数
- 当前队列任务数量
- 队列剩余任务数量

这些参数可以通过线程池的方法获取，可以帮我们动态查看线程池中线程使用状况；

### 线程池服务类接口

线程池服务类接口主要负责维护线程池中的线程配置，按照单一设计原则，主要包含以下几个方法:

```java
public interface IDynamicThreadPoolService {

    /**
     * 查询线程池列表
     * @return
     */
    List<ThreadPoolConfigEntity> queryThreadPoolList();

    /**
     * 根据名字查询线程池配置
     * @param threadPoolName
     * @return
     */
    ThreadPoolConfigEntity queryThreadPoolConfigByName(String threadPoolName);

    /**
     * 更新线程池配置
     * @param threadPoolConfigEntity
     */
    void updateThreadPoolConfig(ThreadPoolConfigEntity threadPoolConfigEntity);
}
```

### 线程池服务接口实现

```java
public class IDynamicThreadPoolServiceImpl implements IDynamicThreadPoolService{

    Logger logger = LoggerFactory.getLogger(IDynamicThreadPoolServiceImpl.class);

    private final String applicationName;

    /**
     * 存放线程池配置的map key:线程池名字 value:线程池对象
     */
    private final Map<String, ThreadPoolExecutor> threadPoolExecutorMap;

    /**
     * 构造函数实例化
     * @author yourname
     * @date 15:01 2024/11/8
     * @param applicationName
     * @param threadPoolExecutorMap
     * @return null
     **/
    public IDynamicThreadPoolServiceImpl(String applicationName, Map<String, ThreadPoolExecutor> threadPoolExecutorMap) {
        this.applicationName = applicationName;
        this.threadPoolExecutorMap = threadPoolExecutorMap;
    }

    /**
     * 查询线程池列表
     * @return
     */
    @Override
    public List<ThreadPoolConfigEntity> queryThreadPoolList() {
//        从内存中查询线程池配置
        Set<String> threadPoolBeanNames = threadPoolExecutorMap.keySet();
        List<ThreadPoolConfigEntity> threadPoolVOS = new ArrayList<>(threadPoolBeanNames.size());
//        循环便利每一个线程池
        for (String beanName : threadPoolBeanNames) {
            ThreadPoolExecutor threadPoolExecutor = threadPoolExecutorMap.get(beanName);
            ThreadPoolConfigEntity threadPoolConfigVO = new ThreadPoolConfigEntity(applicationName, beanName);
            threadPoolConfigVO.setCorePoolSize(threadPoolExecutor.getCorePoolSize());
            threadPoolConfigVO.setMaximumPoolSize(threadPoolExecutor.getMaximumPoolSize());
            threadPoolConfigVO.setActiveCount(threadPoolExecutor.getActiveCount());
            threadPoolConfigVO.setPoolSize(threadPoolExecutor.getPoolSize());
            threadPoolConfigVO.setQueueType(threadPoolExecutor.getQueue().getClass().getSimpleName());
            threadPoolConfigVO.setQueueSize(threadPoolExecutor.getQueue().size());
            threadPoolConfigVO.setRemainingCapacity(threadPoolExecutor.getQueue().remainingCapacity());
            threadPoolVOS.add(threadPoolConfigVO);
        }
        return threadPoolVOS;
    }

    /**
     * 根据名字从内存中查询线程池配置
     * @param threadPoolName
     * @return
     */
    @Override
    public ThreadPoolConfigEntity queryThreadPoolConfigByName(String threadPoolName) {

        ThreadPoolExecutor threadPoolExecutor = threadPoolExecutorMap.get(threadPoolName);

        if(null == threadPoolName){
            return new ThreadPoolConfigEntity(applicationName,threadPoolName);
        }
        ThreadPoolConfigEntity threadPoolConfigVO = new ThreadPoolConfigEntity(applicationName, threadPoolName);
        threadPoolConfigVO.setCorePoolSize(threadPoolExecutor.getCorePoolSize());
        threadPoolConfigVO.setMaximumPoolSize(threadPoolExecutor.getMaximumPoolSize());
        threadPoolConfigVO.setActiveCount(threadPoolExecutor.getActiveCount());
        threadPoolConfigVO.setPoolSize(threadPoolExecutor.getPoolSize());
        threadPoolConfigVO.setQueueType(threadPoolExecutor.getQueue().getClass().getSimpleName());
        threadPoolConfigVO.setQueueSize(threadPoolExecutor.getQueue().size());
        threadPoolConfigVO.setRemainingCapacity(threadPoolExecutor.getQueue().remainingCapacity());

        if(logger.isDebugEnabled()){
            logger.info("动态线程池，配置查询 应用名:{} 线程名:{} 池化配置:{}", applicationName, threadPoolName,JSON.toJSONString(threadPoolConfigVO));
        }

        return threadPoolConfigVO;
    }

    /**
     * 更新线程池配置
     * @param threadPoolConfigEntity
     */
    @Override
    public void updateThreadPoolConfig(ThreadPoolConfigEntity threadPoolConfigEntity) {
        if (null == threadPoolConfigEntity || !applicationName.equals(threadPoolConfigEntity.getAppName())) return;
        ThreadPoolExecutor threadPoolExecutor = threadPoolExecutorMap.get(threadPoolConfigEntity.getThreadPoolName());
        if (null == threadPoolExecutor) return;

        // 设置参数 「调整核心线程数和最大线程数」
        threadPoolExecutor.setCorePoolSize(threadPoolConfigEntity.getCorePoolSize());
        threadPoolExecutor.setMaximumPoolSize(threadPoolConfigEntity.getMaximumPoolSize());

    }
}
```

`Map<String, ThreadPoolExecutor> threadPoolExecutorMap`:内存中维护的线程池参数配置map，一个应用可能有多个线程池，因此使用map结构进行维护；

`List<ThreadPoolConfigEntity> queryThreadPoolList()`:查询线程池列表，将内存中维护的线程池配置封装后返回

`queryThreadPoolConfigByName(String threadPoolName)`：根据线程池名字查询线程池配置数据

`updateThreadPoolConfig(ThreadPoolConfigEntity threadPoolConfigEntity)`:更新线程池配置

线程池服务接口实现类没有什么复杂的逻辑，主要还是围绕不同的线程池实现增删改查等简单的逻辑；



### 自动配置功能实现

组件设计的目标就是通过简单的配置实现公用的功能，既然动态线程池定义为一个组件，那么就需要实现自动配置加载功能；

自定义配置读取类

```java
@ConfigurationProperties(prefix = "dynamic.thread.pool.config",ignoreInvalidFields = true)
public class DynamicThreadPoolAutoProperties {

    /** 状态；open = 开启、close 关闭 */
    private boolean enable;
    /** redis host */
    private String host;
    /** redis port */
    private int port;
    /** 账密 */
    private String password;
    /** 设置连接池的大小，默认为64 */
    private int poolSize = 64;
    /** 设置连接池的最小空闲连接数，默认为10 */
    private int minIdleSize = 10;
    /** 设置连接的最大空闲时间（单位：毫秒），超过该时间的空闲连接将被关闭，默认为10000 */
    private int idleTimeout = 10000;
    /** 设置连接超时时间（单位：毫秒），默认为10000 */
    private int connectTimeout = 10000;
    /** 设置连接重试次数，默认为3 */
    private int retryAttempts = 3;
    /** 设置连接重试的间隔时间（单位：毫秒），默认为1000 */
    private int retryInterval = 1000;
    /** 设置定期检查连接是否可用的时间间隔（单位：毫秒），默认为0，表示不进行定期检查 */
    private int pingInterval = 0;
    /** 设置是否保持长连接，默认为true */
    private boolean keepAlive = true;

}

```

`ConfigurationProperties`通过此注解标注自动读取配置文件中用户的配置数据

### 自动配置核心类实现

```java
@EnableConfigurationProperties(DynamicThreadPoolAutoProperties.class)
@Configuration
@EnableScheduling
public class DynamicThreadPoolAutoConfig {

    Logger logger = LoggerFactory.getLogger(DynamicThreadPoolAutoConfig.class);

    /*应用名称，多个应用多个线程池配置*/
    private String applicationName;

    /**
     * 自动注入到容器中
     * @param applicationContext
     * @param threadPoolExecutorMap
     * @return
     */
    @Bean("dynamicThreadPoolService")
    public IDynamicThreadPoolService  dynamicThreadPoolService(ApplicationContext applicationContext, Map<String, ThreadPoolExecutor> threadPoolExecutorMap,RedissonClient redissonClient){

//        获取应用名称
        applicationName = applicationContext.getEnvironment().getProperty("spring.application.name");

        if(StringUtils.isBlank(applicationName)){
            applicationName="缺省的应用名字";
            logger.warn("动态线程池启动提示:springboot应用没有配置应用名字");
        }

//        获取缓存数据，设置本地线程配置
        // 获取缓存数据，设置本地线程池配置
        Set<String> threadPoolKeys = threadPoolExecutorMap.keySet();
        for (String threadPoolKey : threadPoolKeys) {
//            获取缓存中的线程池配置
//            Redis中缓存的key
            ThreadPoolConfigEntity threadPoolConfigEntity = redissonClient.<ThreadPoolConfigEntity>getBucket(RegistryEnumVO.THREAD_POOL_CONFIG_PARAMETER_LIST_KEY.getKey() + "_" + applicationName + "_" + threadPoolKey).get();
            if (null == threadPoolConfigEntity) continue;
//            更新到本地缓存
            ThreadPoolExecutor threadPoolExecutor = threadPoolExecutorMap.get(threadPoolKey);
            threadPoolExecutor.setCorePoolSize(threadPoolConfigEntity.getCorePoolSize());
            threadPoolExecutor.setMaximumPoolSize(threadPoolConfigEntity.getMaximumPoolSize());
        }

//        打印线程池信息
        logger.info("线程池信息:{}", JSON.toJSONString(threadPoolExecutorMap.keySet()));
        return new IDynamicThreadPoolServiceImpl(applicationName,threadPoolExecutorMap);
    }

    /**
     * 添加Redis client
     * @param properties
     * @return
     */
    @Bean("redissonClient")
    public RedissonClient redissonClient(DynamicThreadPoolAutoProperties properties) {
        Config config = new Config();
        // 根据需要可以设定编解码器；https://github.com/redisson/redisson/wiki/4.-%E6%95%B0%E6%8D%AE%E5%BA%8F%E5%88%97%E5%8C%96
        config.setCodec(JsonJacksonCodec.INSTANCE);

        config.useSingleServer()
                .setAddress("redis://" + properties.getHost() + ":" + properties.getPort())
                .setPassword(properties.getPassword())
                .setConnectionPoolSize(properties.getPoolSize())
                .setConnectionMinimumIdleSize(properties.getMinIdleSize())
                .setIdleConnectionTimeout(properties.getIdleTimeout())
                .setConnectTimeout(properties.getConnectTimeout())
                .setRetryAttempts(properties.getRetryAttempts())
                .setRetryInterval(properties.getRetryInterval())
                .setPingConnectionInterval(properties.getPingInterval())
                .setKeepAlive(properties.isKeepAlive())
        ;

        RedissonClient redissonClient = Redisson.create(config);

        logger.info("动态线程池，注册器（redis）链接初始化完成。{} {} {}", properties.getHost(), properties.getPoolSize(), !redissonClient.isShutdown());

        return redissonClient;
    }

    /**
     * 实例化注册中心
     * @param redissonClient
     * @return
     */
    @Bean
    public IRegistry redisRegistry(RedissonClient redissonClient) {
//        注入实例化好的Redis 客户端
        return new RedisRegistry(redissonClient);
    }

    @Bean
    public ThreadPoolDataReportJob threadPoolDataReportJob(IDynamicThreadPoolService iDynamicThreadPoolService,IRegistry iRegistry){
        return new ThreadPoolDataReportJob(iDynamicThreadPoolService,iRegistry);
    }


    @Bean
    public ThreadPoolConfigAdjustListener threadPoolConfigAdjustListener(IDynamicThreadPoolService iDynamicThreadPoolService,IRegistry iRegistry){
        return new ThreadPoolConfigAdjustListener(iDynamicThreadPoolService,iRegistry);
    }


    @Bean(name = "dynamicThreadPoolRedisTopic")
    public RTopic threadPoolConfigAdjustListener(RedissonClient redissonClient, ThreadPoolConfigAdjustListener threadPoolConfigAdjustListener){
        RTopic topic = redissonClient.getTopic(RegistryEnumVO.DYNAMIC_THREAD_POOL_REDIS_TOPIC.getKey()+"_"+applicationName);
        topic.addListener(ThreadPoolConfigEntity.class, threadPoolConfigAdjustListener);
        return topic;
    }
}
```

`DynamicThreadPoolAutoConfig`是动态线程池自动配置的核心类:

`dynamicThreadPoolService(ApplicationContext applicationContext, Map<String, ThreadPoolExecutor> threadPoolExecutorMap,RedissonClient redissonClient)`:自动注入线程池服务核心类

`redissonClient(DynamicThreadPoolAutoProperties properties)`:注入Redis客户端，因为本实现中使用Redis作为公共的配置中心

`redisRegistry(RedissonClient redissonClient)`:自动注入注册中心

`threadPoolDataReportJob(IDynamicThreadPoolService iDynamicThreadPoolService,IRegistry iRegistry)`:自动注入上报配置信息服务

自动配置类的核心就是将用户在配置文件中的配置，自动装配到Bean对象中注入到ioc容器；

### 上报配置中心实现

以上的实现，基本实现线程池的核心功能，自动读取线程池的配置信息并且装配到Bean对象中，要动态管理线程池的配置，因此还需要引入配置中心，将线程池的配置定时上报到配置中心，由配置中心进行统一维护管理，因此接下来我们实现动态线程池数据定时上报功能；

**上报中心接口设计**

```java
public interface IRegistry {

    /**
     * 上报线程池信息到Redis
     * @param threadPoolConfigEntities
     */
    void reportThreadPool(List<ThreadPoolConfigEntity> threadPoolConfigEntities);


    /**
     * 上报每一个线程池配置数据到redis缓存
     * @param threadPoolConfigEntity
     */
    void reportThreadPoolConfigParameter(ThreadPoolConfigEntity threadPoolConfigEntity);
}
```

接口中主要包括两个主要的方法：

- `reportThreadPool(List<ThreadPoolConfigEntity> threadPoolConfigEntities)`：上报动态线程池列表数据
- `reportThreadPoolConfigParameter(ThreadPoolConfigEntity threadPoolConfigEntity)`：上报每一个线程池配置到配置中心;

上报中心服务实现：

```java
public class RedisRegistry implements IRegistry {

    /**
     * Redis客户端
     */
    private final RedissonClient redissonClient;

    public RedisRegistry(RedissonClient redissonClient) {
        this.redissonClient = redissonClient;
    }

    /**
     * 上报线程池配置参数
     * @param threadPoolConfigEntities
     */
    @Override
    public void reportThreadPool(List<ThreadPoolConfigEntity> threadPoolConfigEntities) {
        RList<ThreadPoolConfigEntity> list = redissonClient.getList(RegistryEnumVO.THREAD_POOL_CONFIG_LIST_KEY.getKey());
        list.delete();
        list.addAll(threadPoolConfigEntities);
    }

    /**
     * 更新线程池参数
     * @param threadPoolConfigEntity
     */
    @Override
    public void reportThreadPoolConfigParameter(ThreadPoolConfigEntity threadPoolConfigEntity) {
//        缓存中每一个线程池的key
        String cacheKey = RegistryEnumVO.THREAD_POOL_CONFIG_PARAMETER_LIST_KEY.getKey() + "_" + threadPoolConfigEntity.getAppName() + "_" + threadPoolConfigEntity.getThreadPoolName();
        RBucket<ThreadPoolConfigEntity> bucket = redissonClient.getBucket(cacheKey);
//        设置过期时间
        bucket.set(threadPoolConfigEntity, Duration.ofDays(30));

    }
}
```

实现主要逻辑是首先查询本地线程池参数配置信息，然后在删除Redis缓存中的配置最后更新Redis配置中心即可；

### 定时上报功能实现

**实现定时上报功能**

```java
public class ThreadPoolDataReportJob {
    Logger logger = LoggerFactory.getLogger(ThreadPoolDataReportJob.class);

//    用来查询数据库参数的配置
    private final IDynamicThreadPoolService iDynamicThreadPoolServicel;

//    用来上报Redis注册中心
    private final IRegistry iRegistry;

    public ThreadPoolDataReportJob(IDynamicThreadPoolService iDynamicThreadPoolServicel, IRegistry iRegistry) {
        this.iDynamicThreadPoolServicel = iDynamicThreadPoolServicel;
        this.iRegistry = iRegistry;
    }

    /**
     * 定时上报线程池的数据
     * 每分钟获取内存中的线程池信息，然后上报到Redis缓存中
     */
    @Scheduled(cron = "0/59 * * * * ?")
    public void execReportThreadPoolList(){

//        从内存中获取线程池配置信息
        List<ThreadPoolConfigEntity> threadPoolList = iDynamicThreadPoolServicel.queryThreadPoolList();

//        更新线程池配置到rediS缓存中
        iRegistry.reportThreadPool(threadPoolList);
        logger.info("动态线程池，上报线程池信息：{}", JSON.toJSONString(threadPoolList));

        for (ThreadPoolConfigEntity t:threadPoolList){
            iRegistry.reportThreadPoolConfigParameter(t);
            logger.info("动态线程池，上报线程池信息：{}", JSON.toJSONString(t));
        }
    }
}
```

定时上报功能主要通过cron定时表达式实现；每分钟上报线程池中的线程参数信息；

最后，如果用户修改了线程池参数，组件还需要能够监听到用户的修改并且重新上报线程池配置信息，因此我们使用Redis组件的发布订阅模式来实现；

### 发布者订阅者实现

主要通过以下三个步骤实现：

1. 注册订阅者
2. 订阅者实现订阅消息
3. 发布者实现发布消息

**订阅注册者实现**

```java
    @Bean(name = "dynamicThreadPoolRedisTopic")
    public RTopic threadPoolConfigAdjustListener(RedissonClient redissonClient, ThreadPoolConfigAdjustListener threadPoolConfigAdjustListener){
        RTopic topic = redissonClient.getTopic(RegistryEnumVO.DYNAMIC_THREAD_POOL_REDIS_TOPIC.getKey()+"_"+applicationName);
        topic.addListener(ThreadPoolConfigEntity.class, threadPoolConfigAdjustListener);
        return topic;
    }
```

在自动配置线程池中自动注册订阅者；

实现订阅者消费数据：

```java
public class ThreadPoolConfigAdjustListener implements MessageListener<ThreadPoolConfigEntity> {

    Logger logger = LoggerFactory.getLogger(ThreadPoolConfigAdjustListener.class);

    //    用来查询数据库参数的配置
    private final IDynamicThreadPoolService iDynamicThreadPoolServicel;

    //    用来上报Redis注册中心
    private final IRegistry iRegistry;

    public ThreadPoolConfigAdjustListener(IDynamicThreadPoolService iDynamicThreadPoolServicel, IRegistry iRegistry) {
        this.iDynamicThreadPoolServicel = iDynamicThreadPoolServicel;
        this.iRegistry = iRegistry;
    }

    /**
     * 监听消息
     * @param channel
     * @param msg
     */
    @Override
    public void onMessage(CharSequence channel, ThreadPoolConfigEntity msg) {
        logger.info("动态线程池，调整线程池配置。线程池名称:{} 核心线程数:{} 最大线程数:{}", msg.getThreadPoolName(), msg.getPoolSize(), msg.getMaximumPoolSize());
//        开始更新线程池参数
        iDynamicThreadPoolServicel.updateThreadPoolConfig(msg);

        // 更新池化配置列表到Redis
        List<ThreadPoolConfigEntity> threadPoolConfigEntities = iDynamicThreadPoolServicel.queryThreadPoolList();
        iRegistry.reportThreadPool(threadPoolConfigEntities);

//        更新每一个线程池配置信息
        ThreadPoolConfigEntity threadPoolConfigEntityCurrent = iDynamicThreadPoolServicel.queryThreadPoolConfigByName(msg.getThreadPoolName());
        iRegistry.reportThreadPoolConfigParameter(threadPoolConfigEntityCurrent);
        logger.info("动态线程池，上报线程池配置：{}", JSON.toJSONString(msg));

    }
}
```

通过实现MessageListener#onMessage()方法消费发布者发布的消息，收到更新事件后，重新更新Redis配置中心的线程池配置数据；

实现发布者发布消息

```java
@Slf4j
@RestController()
@CrossOrigin("*")
@RequestMapping("/api/v1/dynamic/thread/pool/")
public class DynamicThreadPoolController {
    @Resource
    public RedissonClient redissonClient;

    /**
     * 查询线程池数据
     * curl --request GET \
     * --url 'http://localhost:8089/api/v1/dynamic/thread/pool/query_thread_pool_list'
     */
    @RequestMapping(value = "query_thread_pool_list", method = RequestMethod.GET)
    public Response<List<ThreadPoolConfigEntity>> queryThreadPoolList() {
        try {
            RList<ThreadPoolConfigEntity> cacheList = redissonClient.getList("THREAD_POOL_CONFIG_LIST_KEY");
            return Response.<List<ThreadPoolConfigEntity>>builder()
                    .code(Response.Code.SUCCESS.getCode())
                    .info(Response.Code.SUCCESS.getInfo())
                    .data(cacheList.readAll())
                    .build();
        } catch (Exception e) {
            log.error("查询线程池数据异常", e);
            return Response.<List<ThreadPoolConfigEntity>>builder()
                    .code(Response.Code.UN_ERROR.getCode())
                    .info(Response.Code.UN_ERROR.getInfo())
                    .build();
        }
    }

    /**
     * 查询线程池配置
     * curl --request GET \
     * --url 'http://localhost:8089/api/v1/dynamic/thread/pool/query_thread_pool_config?appName=dynamic-thread-pool-test-app&threadPoolName=threadPoolExecutor'
     */
    @RequestMapping(value = "query_thread_pool_config", method = RequestMethod.GET)
    public Response<ThreadPoolConfigEntity> queryThreadPoolConfig(@RequestParam String appName, @RequestParam String threadPoolName) {
        try {
            String cacheKey = "THREAD_POOL_CONFIG_PARAMETER_LIST_KEY" + "_" + appName + "_" + threadPoolName;
            ThreadPoolConfigEntity threadPoolConfigEntity = redissonClient.<ThreadPoolConfigEntity>getBucket(cacheKey).get();
            return Response.<ThreadPoolConfigEntity>builder()
                    .code(Response.Code.SUCCESS.getCode())
                    .info(Response.Code.SUCCESS.getInfo())
                    .data(threadPoolConfigEntity)
                    .build();
        } catch (Exception e) {
            log.error("查询线程池配置异常", e);
            return Response.<ThreadPoolConfigEntity>builder()
                    .code(Response.Code.UN_ERROR.getCode())
                    .info(Response.Code.UN_ERROR.getInfo())
                    .build();
        }
    }

    /**
     * 修改线程池配置
     * curl --request POST \
     * --url http://localhost:8089/api/v1/dynamic/thread/pool/update_thread_pool_config \
     * --header 'content-type: application/json' \
     * --data '{
     * "appName":"dynamic-thread-pool-test-app",
     * "threadPoolName": "threadPoolExecutor",
     * "corePoolSize": 1,
     * "maximumPoolSize": 10
     * }'
     */
    @RequestMapping(value = "update_thread_pool_config", method = RequestMethod.POST)
    public Response<Boolean> updateThreadPoolConfig(@RequestBody ThreadPoolConfigEntity request) {
        try {
            log.info("修改线程池配置开始 {} {} {}", request.getAppName(), request.getThreadPoolName(), JSON.toJSONString(request));
            RTopic topic = redissonClient.getTopic("DYNAMIC_THREAD_POOL_REDIS_TOPIC" + "_" + request.getAppName());
            topic.publish(request);
            log.info("修改线程池配置完成 {} {}", request.getAppName(), request.getThreadPoolName());
            return Response.<Boolean>builder()
                    .code(Response.Code.SUCCESS.getCode())
                    .info(Response.Code.SUCCESS.getInfo())
                    .data(true)
                    .build();
        } catch (Exception e) {
            log.error("修改线程池配置异常 {}", JSON.toJSONString(request), e);
            return Response.<Boolean>builder()
                    .code(Response.Code.UN_ERROR.getCode())
                    .info(Response.Code.UN_ERROR.getInfo())
                    .data(false)
                    .build();
        }
    }


}
```

客户端服务主要完成线程池核心参数的修改功能，当用户触发修改操作后，发布事件触发更新配置中心线程池配置数据；