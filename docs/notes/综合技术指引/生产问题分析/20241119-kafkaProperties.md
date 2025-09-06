---
title: KAFKA配置说明
icon: vscode-icons:file-type-language-configuration
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - KAFKA
tag:
  - kafka
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:13:29
permalink: /compre-guide/production-issues/kafkaProperties/
---
<!-- TOC -->

- [KAFKA配置说明](#kafka配置说明)
  - [一、Broker端参数](#一broker端参数)
    - [1、存储](#1存储)
    - [2、Zookeeper](#2zookeeper)
    - [3、broker id](#3broker-id)
    - [4、连接](#4连接)
    - [5、Topic管理](#5topic管理)
    - [5、数据留存](#5数据留存)
  - [二、Topic级别参数](#二topic级别参数)
  - [三、JVM参数](#三jvm参数)
  - [四、操作系统参数](#四操作系统参数)
  - [Broker参数调优](#broker参数调优)
    - [1. 处理消息的最大线程数](#1-处理消息的最大线程数)
    - [2. 处理磁盘 IO 的线程数](#2-处理磁盘-io-的线程数)
    - [3. 数据落盘策略](#3-数据落盘策略)
    - [4. segment 分段存储策略](#4-segment-分段存储策略)
    - [5. 日志清理策略](#5-日志清理策略)
    - [6. 基础配置](#6-基础配置)
    - [7. 副本同步策略](#7-副本同步策略)
  - [Server参数](#server参数)
  - [producer参数](#producer参数)
  - [consumer参数](#consumer参数)

<!-- /TOC -->

# KAFKA配置说明

kafka的配置分为 broker、produce、consumer三个不同的配置

## 一、Broker端参数

BROKER 的全局配置：

最为核心的三个配置 broker id、log.dir、zookeeper.connect 。

### 1、存储

- log.dirs：指定broker使用的若干个文件目录路径。（无默认值，必须指定）
- log.dir：配置单个路径，用于上个参数的补充。

通常情况下，我们只需要设置`log.dirs`即可。而且建议配置多个路径，比如：`/home/kafka1,/home/kafka2,/home/kafka3`。并且，如果条件允许，最好将这些目录**挂载到不同的物理磁盘**。这样做有两个好处：

- 提升读写性能。多块物理磁盘同时读写数据具有更高的吞吐量
- 故障转移（Failvoer）。在之前**Kafka Broker**任何一块磁盘挂掉，整个Broker都会停止提供服务。在**Kafka1.1**开始，坏掉的磁盘上的数据会自动转移到其他正常磁盘上，并且Broker还能正常工作。

### 2、Zookeeper

Zookeeper在Kafka中扮演着重要的角色，它是一个分布式协调框架，负责协调管理Kafka集群的所有元数据。比如：Broker信息、Topic、每个Topic的partition、partition的Leader副本信息等。

- zookeeper.connect：链接zookeeper的地址。比如我可以指定它的值为`zk1:2181,zk2:2181,zk3:2181`

如果需要让多个Kafka集群使用同一个Zookeeper集群，这个参数应该怎么设置？`chroot`是Zookeeper的概念，类似于别名。

如果你有两套 Kafka 集群，假设分别叫它们`kafka1`和`kafka2`，那么两套集群的`zookeeper.connect`参数可以这样指定：`zk1:2181,zk2:2181,zk3:2181/kafka1`和`zk1:2181,zk2:2181,zk3:2181/kafka2`。切记 chroot 只需要写一次，而且是加到最后的。


### 3、broker id

每一个broker在集群中的唯一标示，要求是正数。在改变IP地址，不改变broker.id的话不会影响consumers

```java
如果kafka集群有三台机器，那么id分别设置为：
broker.id =1
broker.id =2
broker.id =3
```


### 4、连接

客户端或其他Broker和本Broker的通信设置。

- listeners：监听器。其实就是告诉外部连接者要通过什么协议访问指定主机名和端口开放的 Kafka 服务。
- advertised.listeners：和 listeners 相比多了个 advertised。Advertised 的含义表示宣称的、公布的，就是说这组监听器是 Broker 用于对外发布的。

监听器的格式：`<协议名称，主机名，端口号>`。比如`protocol://host:port`

### 5、Topic管理

- auto.create.topics.enable：是否允许自动创建topic
- unclean.leader.election.enable：是否允许Unclean Leader选举
- auto.leader.rebalance.enable：是否允许定期选举

`auto.create.topics.enable`在线上生产环境一定要改为false，特别是将Kafka用作基础组件，会出现各种稀奇古怪的topic。运维需要严格把控Topic的创建。

`unclean.leader.election.enable`Unclean Leader选举，建议设置为false。解释一下unclean，就是那些同步数据落后太多的`partition`。最新版Kafka默认是false，即不允许落后太多的副本进行Leader选举。如果设置为true，那么那些同步较为落后的副本也会参与选举，如果选为Leader，其他副本就会截取掉多余的数据，造成数据丢失。

但是。假设那些保存数据较多的副本都挂了，并且参数设置为false，服务将会不可用。如果设置为true，这时参数就派上用场了，可以从同步较慢的主机中选择leader，虽然丢失数据，Broker还是可以提供服务。

`auto.leader.rebalance.enable`:如果设置为true，表示允许Kafka定期对一些Topic的partition进行Leader重选举。上个参数发生在Leader故障的情况，这个参数和它不同的是，他不是选Leader，而是定期换Leader。比如尽管LeaderA表现一直很有好，但是参数设置为true的情况下，有可能会强行换副本B为Leader。换一次Leader的代价很高，并且这种操作实质上没有任何性能收益，故建议设置为false

### 5、数据留存

- log.retention.{hours|minutes|ms}：控制一条消息数据被保存多长时间。从优先级上来说 ms 最高、minutes 次之、hours 最低。
- log.retention.bytes：指定 Broker 为消息保存的总磁盘容量大小。
- message.max.bytes：控制 Broker 能够接收的最大消息大小。

`log.retention.hours=168`默认设置，表示保存7天，自动删除七天前的数据，通常情况下，在保证磁盘空间足够时，我们尽量将这个值调大。

`log.retention.bytes=-1`默认设置，代表不限制Broker容量。

`message.max.bytes=1000012`默认还不到1MB，**这个值需要进行调整，线上环境超过1MB的消息场景还是比较多的，设置一个比较大的值比较保险**。

## 二、Topic级别参数

如果同时设置了Topic级别参数和全局Broker参数，`以Topic级别参数为准，覆盖全局Broker参数`。

- retention.ms：设置该Topic消息被保存的时间，默认是7天。一旦设置将会覆盖Broker的全局参数
- retention.bytes：设置为该Topic预留多大的磁盘空间，默认为-1，不做限制。
- max.message.bytes：如果公司的Kafka作为基础组件，上面跑的业务数据是比较多的，全局参数较难均衡，可以为每个Topic定制消息大小。

比如使用自带的命令`kafka-configs`来修改 Topic 级别参数，将发送最大值设置为10MB。

```
 bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name transaction --alter --add-config max.message.bytes=10485760
```

## 三、JVM参数

Kafka旧版本是用Scala语言编写的，新版本用Java语言，无论是哪个，终归会编译为`class`文件运行在JVM上，因此JVM参数对于Kafka集群相当重要。

**JDK版本选择**：JDK6已经太过陈旧了，并且在kafka2.0.0版本开始已经放弃了对JDK7的支持，所以建议至少使用JDK8。

**堆**：Kafka默认堆大小为1GB，这显然是不够的，Broker在和客户端交互时，会产生大量的ByteBuffer对象。业内公认较为合理的值为6GB。

**GC**：如果在使用JDK8，那么可以手动设置为G1收集器，在没有任何调优的情况下，G1的表现要比CMS出色，主要体现在更少的Full GC，需要调整的参数更少。

- KAFKA_HEAP_OPTS：指定堆大小
- KAFKA_JVM_PERFORMANCE_OPTS：指定 GC 参数

比如，在启动Broker前，可以这样进行设置：

```
 $> export KAFKA_HEAP_OPTS=--Xms6g --Xmx6g
 $> export KAFKA_JVM_PERFORMANCE_OPTS= -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true
 $> bin/kafka-server-start.sh config/server.properties
 123
```

## 四、操作系统参数

- **文件描述符限制**。通常情况下将它设置成一个超大的值是合理的做法，比如`ulimit -n 1000000`。其实设置这个参数一点都不重要，但不设置的话后果很严重，比如你会经常看到`“Too many open files”`的错误。
- **文件系统类型**：文件系统指的是如 ext3、ext4 或 XFS 这样的日志型文件系统。根据官网的测试报告，XFS 的性能要强于 ext4，所以生产环境最好还是使用 XFS。
- **Swappiness**：很多文章建议设置为0，将swap禁用避免Kafka进程使用swap空间。但是当物理内存耗尽时，操作系统会触发`OOM killer`，随机挑选一个进程然后kill掉，根本不给用户任何预警。 但如果设置成一个比较小的值，当开始使用 swap 空间时，你至少能够观测到 Broker 性能开始出现急剧下降，从而给你进一步调优和诊断问题的时间。比如 1
- **提交时间**：即Flush落盘时间。`向 Kafka 发送数据并不是真要等数据被写入磁盘才会认为成功，而是只要数据被写入到操作系统的页缓存（Page Cache）上就可以了`，随后操作系统根据 LRU 算法会定期将页缓存上的“脏”数据落盘到物理磁盘上。这个定期就是由提交时间来确定的，**默认是 5 秒**。一般情况下我们会认为这个时间太频繁了，可以适当地`增加提交间隔来降低物理磁盘的写操作`。当然你可能会有这样的疑问：如果在页缓存中的数据在写入到磁盘前机器宕机了，那岂不是数据就丢失了。的确，这种情况数据确实就丢失了，但鉴于 **Kafka 在软件层面已经提供了多副本的冗余机制**，因此这里稍微拉大提交间隔去换取性能还是一个合理的做法。

## Broker参数调优

### 1. 处理消息的最大线程数

broker 处理消息的最大线程数，默认为 3，建议设为 cpu 核数 + 1：

例如：cpu 核数 8 ：

```
num.network.threads = 9
```

### 2. 处理磁盘 IO 的线程数

broker 处理磁盘 IO 的线程数，建议设为 cpu 核数 x 2 ：

例如：cpu 核数 8：

```
num.io.threads = 16
```

### 3. 数据落盘策略

Kafka重度依赖底层操作系统提供的`PageCache`功能。当上层有写操作时，操作系统只是将数据写入`PageCache`，同时标记`Page`属性为`Dirty`。当读操作发生时，先从`PageCache`中查找，如果发生缺页才进行磁盘调度，最终返回需要的数据。实际上`PageCache`是把尽可能多的空闲内存都当做了磁盘缓存来使用。但是也带来了问题，如果此时操作系统挂了数据就会丢失，可以通过时间间隔核消息的数量进行合理设置：

```
 每当producer写入10000条消息时，刷数据到磁盘
 log.flush.interval.messages=10000
 
 每间隔5秒钟时间，刷数据到磁盘
 log.flush.interval.ms=5000
```

### 4. segment 分段存储策略

分段文件配置默认是500mb ，有利于快速回收磁盘空间，重启kafka加载也会加快(如果文件过小，则文件数量比较多，kafka启动时是单线程扫描目录(log.dir)下所有数据文件)，文件较多时性能会稍微降低。

```
日志滚动的周期时间，到达指定周期时间时，强制生成一个新的segment
log.roll.hours=72
 
segment的索引文件最大尺寸限制，即时log.segment.bytes没达到，也会生成一个新的segment
log.index.size.max.bytes=10*1024*1024
 
控制日志segment文件的大小，超出该大小则追加到一个新的日志segment文件中（-1表示没有限制）
log.segment.bytes=1024*1024*1024
```

### 5. 日志清理策略

kafka 的消息不管是消费过还是没有消费，都会持久化到硬盘中，如果没有良好的日志清理策略，久而久之会占满磁盘空间，同样核上面配置相似，可以根据时间间隔和日志文件的大小来定义：

```
开启日志清理
log.cleaner.enable=true
 
日志清理运行的线程数
log.cleaner.threads = 2
 
日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被topic创建时的指定参数覆盖，默认 delete
log.cleanup.policy = delete
 
数据文件保留多长时间，存储的最大时间超过这个时间会根据log.cleanup.policy设置数据清除策略
log.retention.bytes和 log.retention.minutes或 log.retention.hours任意一个达到要求，都会执行删除
 
300 分钟
log.retention.minutes=300
24小时
 log.retention.hours=24
 
topic每个分区的最大文件大小，-1没有大小限
log.retention.bytes=1G
 
文件大小检查的周期时间，是否触发 log.cleanup.policy中设置的策略
log.retention.check.interval.ms=5minutes
```

### 6. 基础配置

```
是否允许自动创建topic，若是false，就需要通过命令创建topic
auto.create.topics.enable =true
 
默认副本的数量，可以根据 Broker 的个数进行设置。
default.replication.factor = 3
 
默认，每个topic的分区个数，若是在topic创建时候没有指定的话会被topic创建时的指定参数覆盖
num.partitions = 3
 
消息体的最大大小，单位是字节，如果发送的消息过大，可以适当的增大该参数
message.max.bytes = 6525000
 
socket的发送缓冲区的大小
socket.send.buffer.bytes=102400
 
socket的接受缓冲区的大小
socket.request.max.bytes=104857600
```

### 7. 副本同步策略

```
默认10s，isr中的follow没有向isr发送心跳包就会被移除
replica.lag.time.max.ms = 10000
 
根据leader 和副本的信息条数差值决定是否从isr 中剔除此副本，此信息条数差值根据配置参数，在broker数量较少,或者网络不足的环境中,建议提高此值.
replica.lag.max.messages = 4000
 
follower与leader之间的socket超时时间
replica.socket.timeout.ms=30*1000
 
数据同步时的socket缓存大小
replica.socket.receive.buffer.bytes=64*1024
 
replicas每次获取数据的最大大小
replica.fetch.max.bytes =1024*1024
 
replicas同leader之间通信的最大等待时间，失败了会重试 replica.fetch.wait.max.ms =500
 
fetch的最小数据尺寸，如果leader中尚未同步的数据不足此值,将会阻塞,直到满足条件
replica.fetch.min.bytes =1
 
leader进行复制的线程数，增大这个数值会增加follower的IO
num.replica.fetchers=1
 
每个replica检查是否将最高水位进行固化的频率
replica.high.watermark.checkpoint.interval.ms = 5000
 
leader的不平衡比例，若是超过这个数值，会对分区进行重新的平衡
leader.imbalance.per.broker.percentage = 10
 
检查leader是否不平衡的时间间隔
leader.imbalance.check.interval.seconds = 300
```


## Server参数

```text
# kafka server配置 kafka最为重要三个配置依次为：broker.id、log.dir、zookeeper.connect
 
# 每一个broker在集群中的唯一表示，要求是正数。当该服务器的IP地址发生改变时，broker.id没有变化，则不会影响consumers的消息情况
broker.id=0
 
# broker的主机地址，若是设置了，那么会绑定到这个地址上，若是没有，会绑定到所有的接口上，并将其中之一发送到ZK，一般不设置
host.name=192.168.20.112
 
# broker server服务端口
port =9092
 
# broker处理消息的最大线程数，一般情况下数量为cpu核数
num.network.threads=4
 
# broker处理磁盘IO的线程数，数值为cpu核数2倍
num.io.threads=8
 
# socket的发送缓冲区，socket的调优参数SO_SNDBUFF
socket.send.buffer.bytes=1048576
 
# socket的接受缓冲区，socket的调优参数SO_RCVBUFF
socket.receive.buffer.bytes=1048576
 
# socket请求的最大数值，防止serverOOM，message.max.bytes必然要小于socket.request.max.bytes，会被topic创建时的指定参数覆盖
socket.request.max.bytes=104857600
 
# 每个topic的分区个数，若是在topic创建时候没有指定的话会被topic创建时的指定参数覆盖
num.partitions=2
 
# 数据文件保留多长时间， 存储的最大时间超过这个时间会根据log.cleanup.policy设置数据清除策略，log.retention.bytes和log.retention.minutes或log.retention.hours任意一个达到要求，都会执行删除
# 有2删除数据文件方式： 按照文件大小删除：log.retention.bytes  按照2中不同时间粒度删除：分别为分钟，小时
log.retention.hours=168
 
# topic的分区是以一堆segment文件存储的，这个控制每个segment的大小，会被topic创建时的指定参数覆盖
log.segment.bytes=536870912
 
# 文件大小检查的周期时间，是否处罚 log.cleanup.policy中设置的策略
log.retention.check.interval.ms=60000
 
# 是否开启日志清理
log.cleaner.enable=false
 
# zookeeper集群的地址，可以是多个，多个之间用逗号分割 hostname1:port1,hostname2:port2,hostname3:port3
zookeeper.connect=192.168.20.112:2181
 
# ZooKeeper的连接超时时间
zookeeper.connection.timeout.ms=60000
 
# ZooKeeper的最大超时时间，就是心跳的间隔，若是没有反映，那么认为已经死了，不易过大
zookeeper.session.timeout.ms=6000
 
# ZooKeeper集群中leader和follower之间的同步时间
zookeeper.sync.time.ms =2000
 
# kafka数据的存放地址，多个地址的话用逗号分割,多个目录分布在不同磁盘上可以提高读写性能  /data/kafka-logs-1，/data/kafka-logs-2
log.dirs=/data1/ehserver/env/kafka_2.11-2.2.0/logs/kafka-logs-1
 
 
# ==========================================不重要非必须配置了解 =====================================
 
# 将默认的 delete 改成压缩模式 日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被 topic创建时的指定参数覆盖
log.cleanup.policy=compact
 
# 表示消息体的最大大小，单位是字节
message.max.bytes =6525000
 
# 一些后台任务处理的线程数，例如过期消息文件的删除等，一般情况下不需要去做修改
background.threads =4
 
#等待IO线程处理的请求队列最大数，若是等待IO的请求超过这个数值，那么会停止接受外部消息，应该是一种自我保护机制。
queued.max.requests =500
 
#这个参数会在日志segment没有达到log.segment.bytes设置的大小，也会强制新建一个segment会被 topic创建时的指定参数覆盖
#log.roll.hours =24*7
 
 
# topic每个分区的最大文件大小，一个topic的大小限制 = 分区数*log.retention.bytes。-1没有大小限log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除，会被topic创建时的指定参数覆盖
# log.retention.bytes=-1
 
# 日志清理运行的线程数
log.cleaner.threads = 2
 
#日志清理时候处理的最大大小
# log.cleaner.io.max.bytes.per.second=None
 
# 日志清理去重时候的缓存空间，在空间允许的情况下，越大越好
# log.cleaner.dedupe.buffer.size=500*1024*1024
 
# 日志清理时候用到的IO块大小一般不需要修改
# log.cleaner.io.buffer.size=512*1024
 
# 日志清理中hash表的扩大因子一般不需要修改
log.cleaner.io.buffer.load.factor =0.9
 
# 检查是否处罚日志清理的间隔
log.cleaner.backoff.ms =15000
 
# 日志清理的频率控制，越大意味着更高效的清理，同时会存在一些空间上的浪费，会被topic创建时的指定参数覆盖
log.cleaner.min.cleanable.ratio=0.5
 
# 对于压缩的日志保留的最长时间，也是客户端消费消息的最长时间，同log.retention.minutes的区别在于一个控制未压缩数据，一个控制压缩后的数据。会被topic创建时的指定参数覆盖
# log.cleaner.delete.retention.ms =1day
 
# 对于segment日志的索引文件大小限制，会被topic创建时的指定参数覆盖
# log.index.size.max.bytes =10*1024*1024
 
# 当执行一个fetch操作后，需要一定的空间来扫描最近的offset大小，设置越大，代表扫描速度越快，但是也更好内存，一般情况下不需要搭理这个参数
log.index.interval.bytes =4096
 
# 表示每当消息记录数达到1000时flush一次数据到磁盘，log文件”sync”到磁盘之前累积的消息条数,因为磁盘IO操作是一个慢操作,但又是一个”数据可靠性"的必要手段,所以此参数的设置,需要在"数据可靠性"与"性能"之间做必要的权衡.如果此值过大,将会导致每次"fsync"的时间较长(IO阻塞),如果此值过小,将会导致"fsync"的次数较多,这也意味着整体的client请求有一定的延迟.物理server故障,将会导致没有fsync的消息丢失.
# log.flush.interval.messages=None 例如log.flush.interval.messages=1000
#检查是否需要固化到硬盘的时间间隔
log.flush.scheduler.interval.ms =3000
# 仅仅通过interval来控制消息的磁盘写入时机,是不足的.此参数用于控制"fsync"的时间间隔,如果消息量始终没有达到阀值,但是离上一次磁盘同步的时间间隔达到阀值,也将触发.
# log.flush.interval.ms = None 例如：log.flush.interval.ms=1000 表示每间隔1000毫秒flush一次数据到磁盘
# 文件在索引中清除后保留的时间一般不需要去修改
log.delete.delay.ms =60000
# 控制上次固化硬盘的时间点，以便于数据恢复一般不需要去修改
log.flush.offset.checkpoint.interval.ms =60000
# 是否允许自动创建topic，若是false，就需要通过命令创建topic
auto.create.topics.enable =true
default.replication.factor =1
# ===================================以下是kafka中Leader,replicas配置参数================================
## partition leader与replicas之间通讯时,socket的超时时间
#controller.socket.timeout.ms =30000
#
## partition leader与replicas数据同步时,消息的队列尺寸
#controller.message.queue.size=10
#
##replicas响应partition leader的最长等待时间，若是超过这个时间，就将replicas列入ISR(in-sync replicas)，并认为它是死的，不会再加入管理中
#replica.lag.time.max.ms =10000
#
## 如果follower落后与leader太多,将会认为此follower[或者说partition relicas]已经失效。通常,在follower与leader通讯时,因为网络延迟或者链接断开,总会导致replicas中消息同步滞后如果消息之后太多,leader将认为此follower网络延迟较大或者消息吞吐能力有限,将会把此replicas迁移到其他follower中.在broker数量较少,或者网络不足的环境中,建议提高此值.
#replica.lag.max.messages =4000
#
## replicas
#replica.socket.timeout.ms=30*1000
#
## leader复制时候的socket缓存大小
#replica.socket.receive.buffer.bytes=64*1024
#
## replicas每次获取数据的最大大小
#replica.fetch.max.bytes =1024*1024
#
##replicas同leader之间通信的最大等待时间，失败了会重试
#replica.fetch.wait.max.ms =500
#
##fetch的最小数据尺寸,如果leader中尚未同步的数据不足此值,将会阻塞,直到满足条件
#replica.fetch.min.bytes =1
#
##leader进行复制的线程数，增大这个数值会增加follower的IO
#num.replica.fetchers=1
#
## 每个replica检查是否将最高水位进行固化的频率
#replica.high.watermark.checkpoint.interval.ms =5000
#
## 是否允许控制器关闭broker ,若是设置为true,会关闭所有在这个broker上的leader，并转移到其他broker
#controlled.shutdown.enable =false
#
##控制器关闭的尝试次数
#controlled.shutdown.max.retries =3
#
## 每次关闭尝试的时间间隔
#controlled.shutdown.retry.backoff.ms =5000
#
## leader的不平衡比例，若是超过这个数值，会对分区进行重新的平衡
#leader.imbalance.per.broker.percentage =10
#
## 检查leader是否不平衡的时间间隔
#leader.imbalance.check.interval.seconds =300
#
##客户端保留offset信息的最大空间大小
#offset.metadata.max.bytes
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
# 这是默认配置，使用PLAINTEXT，端口是9092， tcp用来监控的kafka端口
listeners=PLAINTEXT://192.168.20.112:9092
# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1
############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
``` 

## producer参数

```text
#指定kafka节点列表，用于获取metadata，不必全部指定
#需要kafka的服务器地址，来获取每一个topic的分片数等元数据信息。
metadata.broker.list=kafka01:9092,kafka02:9092,kafka03:9092
 
#生产者生产的消息被发送到哪个block，需要一个分组策略。
#指定分区处理类。默认kafka.producer.DefaultPartitioner，表通过key哈希到对应分区
#partitioner.class=kafka.producer.DefaultPartitioner
 
#生产者生产的消息可以通过一定的压缩策略（或者说压缩算法）来压缩。消息被压缩后发送到broker集群，
#而broker集群是不会进行解压缩的，broker集群只会把消息发送到消费者集群，然后由消费者来解压缩。
#是否压缩，默认0表示不压缩，1表示用gzip压缩，2表示用snappy压缩。
#压缩后消息中会有头来指明消息压缩类型，故在消费者端消息解压是透明的无需指定。
#文本数据会以1比10或者更高的压缩比进行压缩。
compression.codec=none
 
#指定序列化处理类，消息在网络上传输就需要序列化，它有String、数组等许多种实现。
serializer.class=kafka.serializer.DefaultEncoder
 
#如果要压缩消息，这里指定哪些topic要压缩消息，默认empty，表示不压缩。
#如果上面启用了压缩，那么这里就需要设置
#compressed.topics= 
#这是消息的确认机制，默认值是0。在面试中常被问到。
#producer有个ack参数，有三个值，分别代表：
#（1）不在乎是否写入成功；
#（2）写入leader成功；
#（3）写入leader和所有副本都成功；
#要求非常可靠的话可以牺牲性能设置成最后一种。
#为了保证消息不丢失，至少要设置为1，也就
#是说至少保证leader将消息保存成功。
#设置发送数据是否需要服务端的反馈,有三个值0,1,-1，分别代表3种状态：
#0: producer不会等待broker发送ack。生产者只要把消息发送给broker之后，就认为发送成功了，这是第1种情况；
#1: 当leader接收到消息之后发送ack。生产者把消息发送到broker之后，并且消息被写入到本地文件，才认为发送成功，这是第二种情况；#-1: 当所有的follower都同步消息成功后发送ack。不仅是主的分区将消息保存成功了，
#而且其所有的分区的副本数也都同步好了，才会被认为发动成功，这是第3种情况。
request.required.acks=0
 
#broker必须在该时间范围之内给出反馈，否则失败。
#在向producer发送ack之前,broker允许等待的最大时间 ，如果超时,
#broker将会向producer发送一个error ACK.意味着上一次消息因为某种原因
#未能成功(比如follower未能同步成功)
request.timeout.ms=10000
 
#生产者将消息发送到broker，有两种方式，一种是同步，表示生产者发送一条，broker就接收一条；
#还有一种是异步，表示生产者积累到一批的消息，装到一个池子里面缓存起来，再发送给broker，
#这个池子不会无限缓存消息，在下面，它分别有一个时间限制（时间阈值）和一个数量限制（数量阈值）的参数供我们来设置。
#一般我们会选择异步。
#同步还是异步发送消息，默认“sync”表同步，"async"表异步。异步可以提高发送吞吐量,
#也意味着消息将会在本地buffer中,并适时批量发送，但是也可能导致丢失未发送过去的消息
producer.type=sync
 
#在async模式下,当message被缓存的时间超过此值后,将会批量发送给broker,
#默认为5000ms
#此值和batch.num.messages协同工作.
queue.buffering.max.ms = 5000
 
#异步情况下，缓存中允许存放消息数量的大小。
#在async模式下,producer端允许buffer的最大消息量
#无论如何,producer都无法尽快的将消息发送给broker,从而导致消息在producer端大量沉积
#此时,如果消息的条数达到阀值,将会导致producer端阻塞或者消息被抛弃，默认为10000条消息。
queue.buffering.max.messages=20000
 
#如果是异步，指定每次批量发送数据量，默认为200
batch.num.messages=500
 
#在生产端的缓冲池中，消息发送出去之后，在没有收到确认之前，该缓冲池中的消息是不能被删除的，
#但是生产者一直在生产消息，这个时候缓冲池可能会被撑爆，所以这就需要有一个处理的策略。
#有两种处理方式，一种是让生产者先别生产那么快，阻塞一下，等会再生产；另一种是将缓冲池中的消息清空。
#当消息在producer端沉积的条数达到"queue.buffering.max.meesages"后阻塞一定时间后,
#队列仍然没有enqueue(producer仍然没有发送出任何消息)
#此时producer可以继续阻塞或者将消息抛弃,此timeout值用于控制"阻塞"的时间
#-1: 不限制阻塞超时时间，让produce一直阻塞,这个时候消息就不会被抛弃
#0: 立即清空队列,消息被抛弃
queue.enqueue.timeout.ms=-1
 
 
#当producer接收到error ACK,或者没有接收到ACK时,允许消息重发的次数
#因为broker并没有完整的机制来避免消息重复,所以当网络异常时(比如ACK丢失)
#有可能导致broker接收到重复的消息,默认值为3.
message.send.max.retries=3
 
#producer刷新topic metada的时间间隔,producer需要知道partition leader
#的位置,以及当前topic的情况
#因此producer需要一个机制来获取最新的metadata,当producer遇到特定错误时,
#将会立即刷新
#(比如topic失效,partition丢失,leader失效等),此外也可以通过此参数来配置
#额外的刷新机制，默认值600000
topic.metadata.refresh.interval.ms=60000
```

## consumer参数

```text
#消费者集群通过连接Zookeeper来找到broker。
#zookeeper连接服务器地址
zookeeper.connect=zk01:2181,zk02:2181,zk03:2181
 
#zookeeper的session过期时间，默认5000ms，用于检测消费者是否挂掉
zookeeper.session.timeout.ms=5000
 
#当消费者挂掉，其他消费者要等该指定时间才能检查到并且触发重新负载均衡
zookeeper.connection.timeout.ms=10000
 
#这是一个时间阈值。
#指定多久消费者更新offset到zookeeper中。
#注意offset更新时基于time而不是每次获得的消息。
#一旦在更新zookeeper发生异常并重启，将可能拿到已拿到过的消息
zookeeper.sync.time.ms=2000
 
#Consumer归属的组ID，broker是根据group.id来判断是队列模式还是发布订阅模式，非常重要
group.id=xxxxx
 
#这是一个数量阈值，经测试是500条。
#当consumer消费一定量的消息之后,将会自动向zookeeper提交offset信息#注意offset信息并不是每消费一次消息就向zk提交
#一次,而是现在本地保存(内存),并定期提交,默认为true
auto.commit.enable=true
 
# 自动更新时间。默认60 * 1000
auto.commit.interval.ms=1000
 
# 当前consumer的标识,可以设定,也可以有系统生成,
#主要用来跟踪消息消费情况,便于观察
conusmer.id=xxx
 
# 消费者客户端编号，用于区分不同客户端，默认客户端程序自动产生
client.id=xxxx
 
# 最大取多少块缓存到消费者(默认10)
queued.max.message.chunks=50
 
# 当有新的consumer加入到group时,将会reblance,此后将会
#有partitions的消费端迁移到新  的consumer上,如果一个
#consumer获得了某个partition的消费权限,那么它将会向zk
#注册 "Partition Owner registry"节点信息,但是有可能
#此时旧的consumer尚没有释放此节点, 此值用于控制,
#注册节点的重试次数.
rebalance.max.retries=5
 
#每拉取一批消息的最大字节数
#获取消息的最大尺寸,broker不会像consumer输出大于
#此值的消息chunk 每次feth将得到多条消息,此值为总大小,
#提升此值,将会消耗更多的consumer端内存
fetch.min.bytes=6553600
 
#当消息的尺寸不足时,server阻塞的时间,如果超时,
#消息将立即发送给consumer
#数据一批一批到达，如果每一批是10条消息，如果某一批还
#不到10条，但是超时了，也会立即发送给consumer。
fetch.wait.max.ms=5000
socket.receive.buffer.bytes=655360
 
# 如果zookeeper没有offset值或offset值超出范围。
#那么就给个初始的offset。有smallest、largest、
#anything可选，分别表示给当前最小的offset、
#当前最大的offset、抛异常。默认largest
auto.offset.reset=smallest
 
# 指定序列化处理类
derializer.class=kafka.serializer.DefaultDecoder
```