---
title: SpringBoot 集成kafka
icon: streamline-freehand-color:notes-quill
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - SPRINGBOOT
  - KAFKA
tag:
  - springboot
  - kafka
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/bkgepc99/
---

# SpringBoot集成kafka

# 1、下载安装kafka

## 1.1、下载安装kafka

在window上安装kafka很简单，首先从kafak官网下载对应版本的kafka。

[Apache Kafka](https://kafka.apache.org/downloads)

然后将下载的zip包进行加压缩，就可以看到kafka的源码包了。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.png)

可以看到，源码包中包含了kafka的可执行文件bin目录，config配置目录，libs依赖库文件等目录。

## 1.2、启动kafka

因为kafka依赖于zookeeper，所以在启动kafka之前需要先启动zookeeper，但是kafka里面已经集成了zookeeper，所以为了方便起见，就不需要在安装zookeeper，直接启动集成的zookeeper服务即可，首先进入kafka的主目录，然后在目录里面输入cmd命令：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%201.png)

现在进入到了kafka主目录，接下来我们启动集成的zookeeper服务，在命令窗口中输入即可启动zookeeper服务：

```java
.\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
```

> 注意：这里可能会遇到提示，不是内部或者外部命令，也不是可运行程序的提示，这是因为kafka识别不了java环境，所以需要指定java环境目录。
> 进入kafka主目录—→bin—→windows—→kafka-run-class-bat，编辑文件，找打下面这一段代码，指明java安装位置即可：

```java
rem Which java to use
IF ["%JAVA_HOME%"] EQU [""] (
set JAVA=java
) ELSE (
set JAVA="D:\soft\jdk1.8\bin\java"
)
```

接下来开始启动kafka服务，同样的思路，输入下面命令即可：

```java
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

至此，kafka就启动了。

## 1.3、springboot集成kafka

首先使用spring的初始化向导初始化一个springboot脚手架工程，使用springmvc-starter,springboot-kafka等场景，这样springboot就自动为我们集成了kafka，我们可以在项目中直接使用kafka api即可。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%202.png)

在pom文件中，我们也可以看到，springBoot已经为我们集成了关于kafka的依赖项。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%203.png)

## 1.4、kafka点对点消息处理

### 1.4.1、创建application.yml配置文件

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      # 发生错误后，消息重发的次数。
      retries: 1
      #当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。
      batch-size: 16384
      # 设置生产者内存缓冲区的大小。
      buffer-memory: 33554432
      # 键的序列化方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      # 值的序列化方式
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
      # acks=1 ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应。
      # acks=all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
      acks: 1
    consumer:
      # 自动提交的时间间隔 在spring boot 2.X 版本中这里采用的是值的类型为Duration 需要符合特定的格式，如1S,1M,2H,5D
      auto-commit-interval: 1S
      # 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
      # latest（默认值）在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的记录）
      # earliest ：在偏移量无效的情况下，消费者将从起始位置读取分区的记录
      auto-offset-reset: earliest
      # 是否自动提交偏移量，默认值是true,为了避免出现重复数据和数据丢失，可以把它设置为false,然后手动提交偏移量
      enable-auto-commit: false
      # 键的反序列化方式
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      # 值的反序列化方式
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
      # 在侦听器容器中运行的线程数。
      concurrency: 5
      #listner负责ack，每调用一次，就立即commit
      ack-mode: manual_immediate
      missing-topics-fatal: false
```

> **注**：**spring.kafka.bootstrap-servers**配置的是kafka集群地址。另外在kafka的  **config/server.properties**  配置文件里面需要修改**advertised.listeners**的值，这个表示kafka对外暴露的地址。单机不需要修改此配置。

### 1.4.2、创建kafkaproduce

```java
@RestController
public class KafkaProducer {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    @GetMapping("/kafka/normal/{message}")
    public void sendNormalMessage(@PathVariable("message") String message) {
        kafkaTemplate.send("sb_topic", message);
    }
}
```

### 1.4.3、创建kafkaconsumer

```java
@Slf4j
@Component
public class kafkaConsumer {

    //监听消费
    @KafkaListener(groupId = "kafka-demo-kafka-group" ,topics = {"sb_topic"})
    public void onNormalMessage(ConsumerRecord<String, Object> record) {
        System.out.println("简单消费：" + record.topic() + "-" + record.partition() + "=" +
                record.value());
    }

}
```

### 1.4.4、发送简单消息

使用postman向kafkaproduce发送请求。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%204.png)

控制台中可以看到kafkaconsumer消费到了消息。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%205.png)

## 1.5、生产者

### 1.5.1、带回调消息的生产者

发送带回调的消息一共有两种方法，第一种方法是调用`SuccessCallback`接口的onSuccess()方法，当消息发送成功时，会调用这个onSuccess()方法，实现`FailureCallback接口中的onFailure方法`，失败的时候调用这个方法。

方法一：

```java
*
     * 回调的第一种写法
     */
    @GetMapping("/kafka/callbackOne/{message}")
    public void sendCallBackMessageOne(@PathVariable("message") String message){

        kafkaTemplate.send("sb_topic",message).addCallback(new SuccessCallback<SendResult<String, Object>>() {
//            消息发送成功回调的方法
            @Override
            public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
                String topic = stringObjectSendResult.getProducerRecord().topic();
                Integer partition = stringObjectSendResult.getProducerRecord().partition();
                long offset = stringObjectSendResult.getRecordMetadata().offset();
                Object value = stringObjectSendResult.getProducerRecord().value();

                System.out.println("sendCallBackMessageOne 发送消息成功1:" + topic + "-" + partition + "-" + offset+ "-" + value);
            }
        }, new FailureCallback() {
//            消息发送失败回调的方法
            @Override
            public void onFailure(Throwable throwable) {

                System.out.println("sendCallBackMessageOne 发送消息失败1:" + throwable.getMessage());
            }
        });
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%206.png)

方法二：

第二种方法是实现`ListenableFutureCallback`接口中的onFailure()方法和onSuccess()方法。

```java
@GetMapping("/kafka/callbackTwo/{message}")
    public void sendCallBackMessageTwo(@PathVariable("message") String message){

        kafkaTemplate.send("sb_topic",message).addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {

//            消息发送失败，调用的方法
            @Override
            public void onFailure(Throwable ex) {
                System.out.println("sendCallBackMessageOne 发送消息失败1:" + ex.getMessage());
            }

//            消息发送成功调用的方法
            @Override
            public void onSuccess(SendResult<String, Object> result) {

                String topic = result.getProducerRecord().topic();
                Integer partition = result.getProducerRecord().partition();
                long offset = result.getRecordMetadata().offset();
                Object value = result.getProducerRecord().value();

                System.out.println("sendCallBackMessageOne 发送消息成功1:" + topic + "-" + partition + "-" + offset+ "-" + value);

            }
        });

    }
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%207.png)

### 1.5.2、监听器

Kafka提供了ProducerListener 监听器来异步监听生产者消息是否发送成功，我们可以自定义一个kafkaTemplate添加ProducerListener，当消息发送失败我们可以拿到消息进行重试或者把失败消息记录到数据库定时重试。

```java
/**
 * @Author: YourName
 * @Date: 2023/8/29 08:55
 * @Description: 自定义kafkaTemplate模板，监听消息是否发送成功
 **/

@Configuration
public class KafkaLintenerConfig {

    @Autowired
    ProducerFactory producerFactory;

    @Bean
    public KafkaTemplate<String,String> kafkaTemplate(){

        KafkaTemplate<String, String> template = new KafkaTemplate<String, String>(producerFactory);

//        在木板上增加监听功能
        template.setProducerListener(new ProducerListener<String, String>() {
//            发送成功调用的方法
            @Override
            public void onSuccess(ProducerRecord<String, String> producerRecord, RecordMetadata recordMetadata) {
                String topic = recordMetadata.topic();
                int partition = recordMetadata.partition();
                String value = producerRecord.value();
                System.out.println("kafka消息发送成功 "+" topic = " + topic + " partion = " + partition + " value "+ value);
            }

//            发送失败调用的方法
            @Override
            public void onError(ProducerRecord<String, String> producerRecord, Exception exception) {

                System.out.println("kafak消息发送失败，入库");
            }
        });

        return template;

    }
}
```

### 1.5.3、Kafka分区

发送消息时候，kafkaTemplate会通过传入的 主题topic、分区partition、键key、值value，其中分区partition和键key是可选的，创建一个 ProducerRecord 对象。

- 如果在 ProducerRecord 对象里指定了分区，那么分区器就不会再做任何事情，直接把指定的分区返回。
- 如果没有指定分区 ，那么分区器会根据 key 来选择一个分区 。
    - 选好分区以后 ，生产者就知道该往哪个主题和分区发送这条记录了。
- 如果 key 为 null ， 并且使用了默认的分区器，那么记录将被随机地发送到主题内各个可用的分区上。分区器使用 轮询（Round Robin ）算法 将消息均衡地分布到各个分区上。
- 如果键不为空，并且使用了默认的分区器，那么 Kafka 会 对键进行散列，然后根据散列值把消息映射到特定的分区上。这里的关键之处在于 ，同一个键总是被映射到同一个分区上 ，所以在进行映射时，我们会使用主题所有的分区，而不仅仅是可用的分区 。这也意味着，如果写入数据的分区是不可用的，那么就会发生错误。但这种情况很少发生。

在kafak中自定义分区器，需要实现partitioner接口，然后实现其中的分区方法即可。

```java
@Component
public class CustomerPartition implements Partitioner {

    @Override
    public int partition(String s, Object o, byte[] bytes, Object o1, byte[] bytes1, Cluster cluster) {
//        默认将数据全部发送到1好分区
        return 1;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}
```

自定义分区之后，需要在配置文件中，配置使用自定义分区策略：

```java
properties:
        # 自定义分区器
   partitioner: com.rzf.springbootKafka.partition.CustomerPartition
```

### 1.5.4、kafka事务

如果在发送消息时需要创建事务，可以使用 KafkaTemplate 的 executeInTransaction 方法来声明事务：

```java
@GetMapping("/kafka/sendTransactionMessage/{message}")
    public void sendTransactionMessage(@PathVariable("message") String message){

        kafkaTemplate.executeInTransaction(new KafkaOperations.OperationsCallback<String, String, Object>() {
            @Override
            public Object doInOperations(KafkaOperations<String, String> operations) {

                operations.send("sb_topic",message,"test executeInTransaction");
                throw new RuntimeException("fail");
//                return null;
            }
        });

    }
```

需要在配置文件中配置事务id：

```java
transaction-id-prefix: tx_
```

## 1.6、生产者

### 1.6.1、指定topic,partition,offset消费

前面我们在监听消费topic1的时候，监听的是topic1上所有的消息，如果我们想指定topic、指定partition、指定offset来消费呢？也很简单，@KafkaListener注解已全部为我们提供

```xml
spring:
	kafka:
		listener:
			type: batch #设置批量消费
		consumer:
			max-poll-records: 50 #每次最多消费多少条消息
```

属性解释：

- `id`：消费者ID
- `groupId`：消费组ID
- `topics`：监听的topic，可监听多个
- `topicPartitions`：可配置更加详细的监听信息，可指定topic、parition、offset监听，手动分区。

```xml
/**
     * 设置消费者批量消费数据
     */
    @KafkaListener(id = "consumer01",topics = "sb_topic",groupId = "kafka-demo-kafka-group")
    public  void batchConsumerMessage(List<ConsumerRecord<String,String>> records){
        System.out.println(">>> 批量消费一次，recoreds.size()=" + records.size());

//        for(ConsumerRecord<String,String> record:records){
//            System.out.println(record.value());
//        }
    }
```

### 1.6.2、异常处理

ConsumerAwareListenerErrorHandler 异常处理器，新建一个 ConsumerAwareListenerErrorHandler 类型的异常处理方法，用@Bean注入，BeanName默认就是方法名，然后我们将这个异常处理器的BeanName放到@KafkaListener注解的errorHandler属性里面，当监听抛出异常的时候，则会自动调用异常处理器。

```xml
//异常处理器
    @Bean
    public ConsumerAwareListenerErrorHandler consumerAwareErrorHandler() {
        return new ConsumerAwareListenerErrorHandler() {
            @Override
            public Object handleError(Message<?> message, ListenerExecutionFailedException exception, Consumer<?, ?> consumer) {
                System.out.println("消费异常：" + message.getPayload());
                return null;
            }
        };
    }

// 将这个异常处理器的BeanName放到@KafkaListener注解的errorHandler属性里面
    @KafkaListener(topics = {"sb_topic"},errorHandler = "consumerAwareErrorHandler")
    public void onMessage4(ConsumerRecord<?, ?> record) throws Exception {
        throw new Exception("简单消费-模拟异常");
    }
```

### 1.6.3、消息过滤

消息过滤器可以在消息抵达consumer之前被拦截，在实际应用中，我们可以根据自己的业务逻辑，筛选出需要的信息再交由KafkaListener处理，不需要的消息则过滤掉。

配置消息过滤只需要为 监听器工厂 配置一个RecordFilterStrategy（消息过滤策略），返回true的时候消息将会被抛弃，返回false时，消息能正常抵达监听容器。

```xml
@Bean
    public ConcurrentKafkaListenerContainerFactory filterContainerFactory(){
        ConcurrentKafkaListenerContainerFactory factory = new ConcurrentKafkaListenerContainerFactory();

        factory.setConsumerFactory(consumerFactory);
//        被过滤的消息丢弃
        factory.setAckDiscarded(true);
//        设置消息过滤策略
        factory.setRecordFilterStrategy(new RecordFilterStrategy() {
            @Override
            public boolean filter(ConsumerRecord consumerRecord) {
                if(Integer.parseInt(consumerRecord.value().toString()) %2 ==0){
                    return true;
                }

                return false;
            }
        });

        return factory;
    }

    @KafkaListener(topics = {"sb_topic"},groupId = "kafka-demo-kafka-group",containerFactory = "filterContainerFactory")
    public void onMessage(ConsumerRecord<String,String> record){

        System.out.println(record.value());
    }
```

### 1.6.4、消息转发

在实际开发中，我们可能有这样的需求，应用A从TopicA获取到消息，经过处理后转发到TopicB，再由应用B监听处理消息，即一个应用处理完成后将该消息转发至其他应用，完成消息的转发。

在SpringBoot集成Kafka实现消息的转发也很简单，只需要通过一个@SendTo注解，被注解方法的return值即转发的消息内容，如下：

```xml
//消息转发 从sb_topic转发到sb_topic2
    @KafkaListener(topics = {"sb_topic"},groupId = "kafka-demo-kafka-group")
    @SendTo("sb_topic2")
    public String onMessage7(ConsumerRecord<?, ?> record) {
        return record.value()+"-forward message";
    }

    @KafkaListener(topics = {"sb_topic2"},groupId = "kafka-demo-kafka-group")
    public void onMessage8(ConsumerRecord<?, ?> record) {
        System.out.println("收到sb_topic转发过来的消息：" + record.value());
    }
```

### 1.4.6、手动确认消息

默认情况下Kafka的消费者是自动确认消息的，通常情况下我们需要在业务处理成功之后手动触发消息的签收，否则可能会出现：消息消费到一半消费者异常，消息并未消费成功但是消息已经自动被确认，也不会再投递给消费者，也就导致消息丢失了。

当 auto.commit.enable 设置为false时，表示kafak的offset由customer手动维护，spring-kafka提供了通过ackMode的值表示不同的手动提交方式

```java
public enum AckMode {
    // 当每一条记录被消费者监听器（ListenerConsumer）处理之后提交
    RECORD,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后提交
    BATCH,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，距离上次提交时间大于TIME时提交
    TIME,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，被处理record数量大于等于COUNT时提交
    COUNT,
    // TIME |　COUNT　有一个条件满足时提交
    COUNT_TIME,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后, 手动调用Acknowledgment.acknowledge()后提交
    MANUAL,
    // 手动调用Acknowledgment.acknowledge()后立即提交
    MANUAL_IMMEDIATE,
}
```

如果设置AckMode模式为`MANUAL`或者`MANUAL_IMMEDIATE`,则需要对监听消息的方法中，引入`Acknowledgment`对象参数，并调用acknowledge()方法进行手动提交；

第一步：添加kafka配置，把 spring.kafka.listener.ack-mode = manual 设置为手动

```java
spring:
	kafka:
		listener:
			ack-mode: manual 
		consumer:
			enable-auto-commit: false
```

第二步；消费消息的时候，给方法添加Acknowledgment参数签收消息：

```java
@KafkaListener(topics = {"sb_topic"})
public void onMessage9(ConsumerRecord<String, Object> record, Acknowledgment ack) {
    System.out.println("收到消息：" + record.value());
    //确认消息
    ack.acknowledge();
}
```

## 1.7、生产者配置

```java
server:
  port: 8081
spring:
  kafka:
    producer:
      # Kafka服务器
      bootstrap-servers: 175.24.228.202:9092
      # 开启事务，必须在开启了事务的方法中发送，否则报错
      transaction-id-prefix: kafkaTx-
      # 发生错误后，消息重发的次数，开启事务必须设置大于0。
      retries: 3
      # acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
      # acks=1 ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应。
      # acks=all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
      # 开启事务时，必须设置为all
      acks: all
      # 当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。
      batch-size: 16384
      # 生产者内存缓冲区的大小。
      buffer-memory: 1024000
      # 键的序列化方式
      key-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      # 值的序列化方式（建议使用Json，这种序列化方式可以无需额外配置传输实体类）
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

## 1.8、消费者配置

```java
server:
  port: 8082
spring:
  kafka:
    consumer:
      # Kafka服务器
      bootstrap-servers: 175.24.228.202:9092
      group-id: firstGroup
      # 自动提交的时间间隔 在spring boot 2.X 版本中这里采用的是值的类型为Duration 需要符合特定的格式，如1S,1M,2H,5D
      #auto-commit-interval: 2s
      # 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
      # earliest：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费分区的记录
      # latest：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据（在消费者启动之后生成的记录）
      # none：当各分区都存在已提交的offset时，从提交的offset开始消费；只要有一个分区不存在已提交的offset，则抛出异常
      auto-offset-reset: latest
      # 是否自动提交偏移量，默认值是true，为了避免出现重复数据和数据丢失，可以把它设置为false，然后手动提交偏移量
      enable-auto-commit: false
      # 键的反序列化方式
      #key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      key-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      # 值的反序列化方式（建议使用Json，这种序列化方式可以无需额外配置传输实体类）
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      # 配置消费者的 Json 反序列化的可信赖包，反序列化实体类需要
      properties:
        spring:
          json:
            trusted:
              packages: "*"
      # 这个参数定义了poll方法最多可以拉取多少条消息，默认值为500。如果在拉取消息的时候新消息不足500条，那有多少返回多少；如果超过500条，每次只返回500。
      # 这个默认值在有些场景下太大，有些场景很难保证能够在5min内处理完500条消息，
      # 如果消费者无法在5分钟内处理完500条消息的话就会触发reBalance,
      # 然后这批消息会被分配到另一个消费者中，还是会处理不完，这样这批消息就永远也处理不完。
      # 要避免出现上述问题，提前评估好处理一条消息最长需要多少时间，然后覆盖默认的max.poll.records参数
      # 注：需要开启BatchListener批量监听才会生效，如果不开启BatchListener则不会出现reBalance情况
      max-poll-records: 3
    properties:
      # 两次poll之间的最大间隔，默认值为5分钟。如果超过这个间隔会触发reBalance
      max:
        poll:
          interval:
            ms: 600000
      # 当broker多久没有收到consumer的心跳请求后就触发reBalance，默认值是10s
      session:
        timeout:
          ms: 10000
    listener:
      # 在侦听器容器中运行的线程数，一般设置为 机器数*分区数
      concurrency: 4
      # 自动提交关闭，需要设置手动消息确认
      ack-mode: manual_immediate
      # 消费监听接口监听的主题不存在时，默认会报错，所以设置为false忽略错误
      missing-topics-fatal: false
      # 两次poll之间的最大间隔，默认值为5分钟。如果超过这个间隔会触发reBalance
      poll-timeout: 600000
```

kafka命令：

查看kafak中有那些topic:

```java
.\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```