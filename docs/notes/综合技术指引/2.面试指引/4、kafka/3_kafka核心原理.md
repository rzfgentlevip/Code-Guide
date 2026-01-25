---
title: 3、Kafka核心原理
icon: logos:kafka
order: 3
author: bugcode
date: 2024/12/8
copyright: bugcode
createTime: 2026/01/17 13:32:13
permalink: /compre-guide/interview/kafka/Kafka核心原理/
---

## kafka分区结构

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/09/182528-493077.png)

上图表示`kafka`集群有一个`topic A`,并且有三个分区,分布在三个节点上面。

注意点:每个分区有两个副本,两个副本分别是`leader,follower`,并且每一个副本一定不和自己的`leader`分布在一个节点上面。`Kafka`中消息是以  `topic`进行分类的,生产者生产消息,消费者消费消息,都是面向  `topic`的。

`topic`是逻辑上的概念,而`partition`是物理上的概念,每个` partition`对应于一个` log`文件,该` log`文件中存储的就是  `producer`生产的数据。`Producer`生产的数据会被不断追加到该`log`文件末端,且每条数据都有自己的  `offset`。消费者组中的每个消费者,都会实时记录自己消费到了哪个 `offset`,以便出错恢复时,从上次的位置继续消费。每一个分区内部的数据是有序的,但是全局不是有序的。

##  kafka文件存储结构

![](./image/kafka主题文件存储结构.png)

由于生产者生产的消息会不断追加到`log`文件末尾,为防止`  log`文件过大导致数据定位效率低下,`Kafka`采取了**分片和索引机制**,将每个  `partition`分为多个 ` segment`。每个  `segment`对应两个文件`“.index”`文件和`“.log”`文件。这些文件位于一个文件夹下,该文件夹的命名规则为:`topic名称+分区序号`。例如,`first`这个  ` topic`有三个分区,则其对应的文件夹为  :

~~~ java
first-0,first-1,first-2
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
// index文件中存储的是每一个log文件的起始数据的偏移量
~~~

index和 ` log`文件以当前   `segment`的**第一条消息的  `offset`命名**。下图为   `index`文件和   `log`文件的结构示意图

![](./image/索引查找.png)

上面`kafka`再查找偏移量的时候是以**二分查找法进行查找的**。也就是查询index的时候使用的是二分查找法。

查找原理是:文件头的偏移量和文件大小快速定位。`“.index”`文件存储大量的索引信息,在查找`index`的时候使用的是二分查找法,`“.log”`文件存储大量的数据,索引文件中的元数据指向对应数据文件中 `message`的物理偏移地址。

> **log、index、timeindex** 中存储的都是`二进制`的数据( **log 中存储的是 BatchRecords 消息内容,而 index 和 timeindex 分别是一些索引信息**。)


## kafka生产者写入数据

### 副本

同一个`partition`可能会有多个`replication`(对应` server.properties `配置中default.replication.factor=N)没有`replication`的情况下,一旦`broker` 宕机,其上所有` patition` 的数据都不可被消费,同时producer也不能再将数据存于其上的`patition`。引入`replication`之后,同一个`partition`可能会有多个`replication`,而这时需要在这些`replication`之间选出一个`leader`,`producer`和`consumer`只与这个`leader`交互,其它`replication`的`follower`从leader 中复制数据,保证数据的一致性。

### 写入方式

`producer`采用推`(push)`模式将消息发布到`broker`,每条消息都被追加`(append)`到分区`(patition)`中,属于**顺序写磁盘**(顺序写磁盘效率比随机写内存要高,保障`kafka`吞吐率)。

![](./image/kafka生产者写数据.png)

1. producer先从`zookeeper`的 `"/brokers/.../state"`节点找到该`partition`的`leader`
2. `producer`将消息发送给该`leader`
3. `leader`将消息写入本地`log`
4. `followers`从`leader pull`消息,写入本地`log`后向`leader`发送`ACK`
5. `leader`收到所有`ISR`中的`replication`的`ACK`后,增加`HW(high watermark,最后commit 的offset)`并向`producer`发送`ACK`

### broker保存消息

存储方式:物理上把`topic`分成一个或多个`patition`,每个`patition`物理上对应一个文件夹(该文件夹存储该`patition`的所有消息和索引文件)

### 存储策略

无论消息是否被消费,`kafka`都会保留所有消息。有两种策略可以删除旧数据:

- 基于时间:`log.retention.hours=168`
- 基于大小:`log.retention.bytes=1073741824`

需要注意的是,因为`Kafka`读取特定消息的时间复杂度为`O(1)`,即与文件大小无关,所以这里删除过期文件与提高` Kafka `性能无关

### 分区

消息发送时都被发送到一个`topic`,其本质就是一个目录,而`topic`是由一些`Partition Logs`(分区日志)组成,其组织结构如下图所示:

![](./image/kafka分区.png)

我们可以看到,每个`Partition`中的消息都是有序的,生产的消息被不断追加到`Partition log`上,其中的每一个消息都被赋予了一个唯一的`offset`值。

- 分区的原因
    - 方便在集群中扩展,每个`Partition`可以通过调整以适应它所在的机器,而一个`topic`又可以有多个`Partition`组成,因此整个集群就可以适应任意大小的数据了;
    - 可以提高并发,因为可以以`Partition`为单位读写了。
- 分区的原则
    - 指定了`patition`,则直接使用;
    - 未指定`patition`但指定`key`,通过对`key`的`value`进行`hash`出一个`patition`;
    - 既没有`  partition`值又没有    `key`值的情况下,第一次调用时随机生成一个整数(后面每次调用在这个整数上自增),将这个值与 ` topic`可用的   ` partition`总数取余得到    `partition`值,也就是常说的 ` round-robin`算法(轮询算法)。.

## kafka写入数据可靠性保障

### 生产者写入数据多阶段

![](./image/多阶段写入数据.png)

produce写入消息的可靠性保证:

### 数据写入的可靠性保证

为保证 `producer`发送的数据,能可靠的发送到指定的  `topic`,`topic`的每个`  partition`收到`producer`发送的数据后,都需要向`   producer`发送 `  ack(acknowledgement确认收到)`,如果`producer`收到 ` ack`,就会进行下一轮的发送,否则重新发送数据。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090843-235649.png)

### 副本数据同步策略

| 方案                              | 优点                                                   | 缺点                                                |
| --------------------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| 半数以上同步完成,发送`ack`确认   | 延迟低                                                 | 选举新的节点时,容忍`n`个节点故障,需要`2n+1`个副本 |
| 全部同步完成以后,才发送`ack`确认 | 选举新的`leader`时,容忍`n`台节点故障,需要`n+1`个副本 | 延迟低                                              |

`Kafka`选择了第二种方案,原因如下:

- 同样为了容忍 `n`台节点的故障,第一种方案需要`2n+1`个副本,而第二种方案只需要`n+1`个副本,而 `Kafka`的每个分区都有大量的数据,第一种方案会造成大量数据的冗余。
- 虽然第二种方案的网络延迟会比较高,但网络延迟对` Kafka`的影响较小。

### ISR

采用第二种方案之后,设想以下情景:`leader`收到数据,所有`follower`都开始同步数据,但有一个 `follower`,因为某种故障,迟迟不能与` leader`进行同步,那  `leader`就要一直等下去,直到它完成同步,才能发送 `ack`。这个问题怎么解决呢？

- `Leader`维护了一个动态的 ` in-sync replica set (ISR)`,意为和 `leader`保持同步的 `follower`集合,是一个队列。当 `ISR`中的 ` follower`完成数据的同步之后,`leader`就会给 `follower`发送  `ack`。如果  `follower`长时间未向`leader`同步数据,则该`follower`将被踢出`ISR`,该时间阈值由`replica.lag.time.max.ms`参数设定。`Leader`发生故障之后,就会从`ISR`中选举新的`leader`。在这个时间内,就添加到isr中,否则就提出isr集合中,不在isr中的follower也不可能被选举为leader。

~~~java
rerplica.lag.time.max.ms=10000
//如果leader发现flower超过10秒没有向它发起fech请求, 那么leader考虑这个flower是不是程序出了点问题,或者资源紧张调度不过来, 它太慢了, 不希望它拖慢后面的进度, 就把它从ISR中移除.
rerplica.lag.max.messages=4000
//相差4000条就移除,flower慢的时候, 保证高可用性, 同时满足这两个条件后又加入ISR中,在可用性与一致性做了动态平衡
min.insync.replicas=1
//需要保证ISR中至少有多少个replica
~~~

### **ack**应答机制

对于某些不太重要的数据,对数据的可靠性要求不是很高,能够容忍数据的少量丢失,所以没必要等` ISR`中的`follower`全部接收成功。所以` Kafka`为用户提供了三种可靠性级别,用户根据对可靠性和延迟的要求进行权衡,选择以下的配置。

`ACKS`参数配置:

- 0:`producer`不等待  `broker`的`ack`,这一操作提供了一个最低的延迟,`broker`一接收到还没有写入磁盘就已经返回,**当 `broker`故障时有可能丢失数据;**
- 1:`producer`等待  `broker`的`ack`,`partition`的  `leader`落盘成功后返回`ack`,如果在`follower`同步成功之前 `leader`故障,那么将会丢失数据;


**ack应答示意图**

![](./image/kafka_ack应答.png)

#### ack = 1分析

##### ack=1工作原理

ack=1 的发送流程

生产者端流程：
1. 发送消息到Leader Broker
2. Leader写入本地日志（Page Cache）
3. Leader立即返回成功确认给生产者
4. Leader异步复制到Follower副本

关键点：
- Leader写入成功即返回，不等待Follower确认
- Follower复制是异步的
- 确认发生在消息进入Leader的Page Cache后

##### ack=1 与数据安全的时间线分析

```yaml
时间线示例（ack=1场景）:
        
t0: Producer发送消息M到Leader (Broker1)
t1: Leader接收消息，写入Page Cache
t2: Leader返回成功确认给Producer (ack=1完成)
t3: Producer收到确认，认为消息发送成功
t4: Follower (Broker2) 开始异步拉取消息M
t5: Leader (Broker1) 突然故障
t6: Controller选举新Leader (Broker2)
t7: 问题：Broker2可能没有消息M（如果t4-t5期间未复制）

数据丢失风险：消息M可能丢失
因为新Leader可能没有收到这条消息
```

##### 数据丢失的具体场景

**场景1：Leader故障导致数据丢失**

场景描述：
- acks=1
- 副本因子=3 (Broker1为Leader, Broker2、3为Follower)
- 生产者发送消息M到Broker1
- Broker1写入成功，返回确认
- 在消息复制到Follower之前，Broker1故障
- Broker2成为新Leader，但没有消息M
- 结果：消息M永久丢失

发生概率：中等

影响因素：
- 网络延迟
- Broker负载
- 副本同步速度

**数据丢失图示**

![](./image/ack_1.png)

**场景2：不完全的ISR维护**

ISR（In-Sync Replicas）机制：
- Follower需要定期从Leader拉取数据
- 落后太多的Follower会被移出ISR
- acks=1时，消息只需Leader确认

风险场景：
1. Follower由于网络/负载问题落后
2. 被移出ISR，但生产者不知道
3. 生产者继续发送消息（acks=1）
4. Leader故障，从剩余ISR选举新Leader
5. 如果ISR中副本数 < min.insync.replicas 可能导致消息丢失

```java
 public void showISRRisk() {
            // Broker配置
            int replicationFactor = 3;
            int minInsyncReplicas = 2;  // 最小同步副本数
            
            System.out.println("=== ISR相关风险 ===");
            System.out.println("配置：副本因子=" + replicationFactor + 
                             ", min.insync.replicas=" + minInsyncReplicas);
            
            // 初始状态
            System.out.println("初始ISR: [Broker1(Leader), Broker2, Broker3]");
            
            // 场景：Broker2、3由于网络问题落后
            System.out.println("Broker2、3网络延迟增加...");
            System.out.println("Broker2、3被移出ISR（落后太多）");
            System.out.println("当前ISR: [Broker1]");
            
            // 生产者继续发送消息（acks=1）
            System.out.println("生产者发送消息（acks=1）...");
            System.out.println("Leader(Broker1)写入成功，返回ack");
            
            // Leader故障
            System.out.println("Broker1突然故障！");
            
            if (minInsyncReplicas > 1) {
                System.out.println("⚠️  危险：ISR中副本数(" + 1 + ") < min.insync.replicas(" + 
                                 minInsyncReplicas + ")");
                System.out.println("生产者会收到错误，但已确认的消息可能丢失");
            } else {
                System.out.println("可以选举新Leader，但消息可能丢失");
            }
        }
```

**数据丢失概率计算**

影响数据丢失概率的因素：
1. 副本因子（replication.factor）
2. 集群规模（Broker数量）
3. 网络可靠性
4. Broker稳定性
5. 消息大小和频率

近似计算公式：

数据丢失概率 ≈ (单Broker故障率) × (消息复制延迟概率)

示例估算：
- 单Broker月故障率：0.1%
- 复制延迟>100ms概率：0.5%
- 估算丢失概率：0.1% × 0.5% = 0.0005% (每月)

##### ack=1 的重复消费

**场景1：生产者重试导致的重复**

产生重复的场景：

1. 生产者发送消息M，网络超时
2. 生产者未收到ack，触发重试
3. 实际上Leader已写入消息M，但ack响应丢失
4. 重试时再次发送消息M
5. Broker收到两条相同的消息M

根本原因：ack响应丢失 + 生产者重试


**场景2：消费者提交偏移量失败导致的重复**

场景描述：

1. 消费者拉取消息M并处理成功
2. 提交偏移量时失败（网络问题、重启等）
3. 消费者重启或重新加入消费组
4. 从上次提交的偏移量重新消费
5. 再次处理消息M

根本原因：处理成功但偏移量提交失败

**场景3：再平衡（Rebalance）导致的重复**

Rebalance过程中的重复风险：

1. 消费者处理消息但未提交偏移量
2. 触发Rebalance（消费者加入/离开）
3. 分区重新分配给其他消费者
4. 新消费者从已提交偏移量开始消费
5. 未提交的消息被重复消费


##### ack=1 的最佳实践与缓解措施

**降低数据丢失风险的策略**

ack=1时降低数据丢失风险的配置：
1. 增加副本因子
2. 优化副本同步
3. 合理设置超时
4. 启用监控告警

**防止重复消费的策略**

幂等消费的核心思想,相同的消息处理多次，产生的结果应该相同

消费者端幂等处理：

1. 基于消息ID去重
2. 基于业务键去重
3. 使用外部存储记录处理状态
4. 使用数据库唯一约束

生产者端减少重复：

1. 合理设置重试参数
2. 使用幂等生产者
3. 实现生产者端去重
4. 优化超时配置

##### ack=1 端到端Exactly-Once方案

在acks=1情况下实现端到端精确一次的折中方案：

方案：幂等生产者 + 幂等消费者 + 业务补偿

1. 生产者启用幂等性（避免生产者重复）
2. 消费者实现幂等处理（避免消费者重复）  
3. 业务层实现最终一致性检查
4. 定期数据对账和补偿

这不是真正的Exactly-Once，但在大多数业务中可接受

##### ack=1 or ack=-1

```java

选择 acks=1 还是 acks=all 的决策流程：

开始
  ↓
业务是否容忍极少量数据丢失？
  ├─ 是 → 业务是否容忍少量重复？
  │       ├─ 是 → 使用 acks=1 + 消费者幂等
  │       └─ 否 → 需要 acks=all + 事务
  │
  └─ 否 → 必须使用 acks=all
          ↓
        是否需要全局顺序？
          ├─ 是 → acks=all + max.in.flight=1
          └─ 否 → acks=all + 适当并行
       
// 性能与可靠性权衡
acks=1 vs acks=all 的权衡：
维度          acks=1          acks=all
───────────────────────────────────────
吞吐量        最高            较低（30-50%降低）
延迟          最低            较高（等待副本确认）
数据丢失风险  有风险          极低（需配置正确）
重复消费风险  有风险          较低（配合幂等）
实现复杂度    简单            复杂（需事务支持）
适用场景      日志、监控       交易、订单
```

#### ack=-1 分析

##### ack=-1 的核心工作流程

```java
ack=-1 (或acks=all) 的工作机制：
      
生产者端流程：
1. 生产者发送消息到分区Leader
2. Leader写入本地日志（同步到磁盘）
3. Leader等待所有ISR副本同步写入成功
4. 所有ISR副本确认后，Leader返回成功给生产者
5. 生产者收到确认，消息发送完成

关键点:
- 需要所有ISR副本确认，不仅仅是Leader
- 确保消息在多个副本上持久化
- 配合min.insync.replicas提供强一致性保证
```
##### ISR与ack=-1的关系

```java
ISR（In-Sync Replicas）定义:
- 与Leader保持同步的副本集合 
- 副本延迟不超过replica.lag.time.max.ms（默认30秒）

ack=-1时：
1. 生产者等待所有ISR副本确认
2. ISR数量必须 ≥ min.insync.replicas
3. 如果ISR数量不足，生产者会抛出异常

示例配置：
replication.factor=3
min.insync.replicas=2

这意味着：
• 至少需要2个副本确认（包括Leader）
• 最多容忍1个副本故障
• 保证数据不丢失（除非2个副本同时故障）
```

##### ack=-1 的数据丢失风险分析

**极端情况下的数据丢失风险**

```java
即使使用ack=-1，理论上仍存在极端情况可能导致数据丢失：
        
情况1：多副本同时故障
• 所有ISR副本在确认后立即同时故障
• 数据已持久化，但所有副本丢失（概率极低）

情况2：数据损坏
• 硬件故障导致已确认的数据损坏

情况3：配置不当
• min.insync.replicas设置不合理
• unclean.leader.election.enable=true
```

##### ack=-1 的重复消费风险分析

生产者端重复风险（即使ack=-1）

```java
ack=-1 仍可能发生生产者重复的情况：
        
场景1：ack响应丢失 + 生产者重试
1. 生产者发送消息，所有ISR确认
2. ack响应在网络中丢失
3. 生产者超时，触发重试
4. 再次发送相同消息
5. Broker收到重复消息

场景2：生产者崩溃重启
1. 生产者发送消息，收到ack
2. 生产者崩溃，未记录发送状态
3. 重启后重新发送相同消息

解决方案：启用幂等生产者（enable.idempotence=true）
```


消费者端重复风险（与ack级别无关）

```java
消费者重复与生产者ack级别无关：
        
重复来源：
1. 偏移量提交失败
2. 消费者重启/再平衡
3. 处理成功但提交失败

ack=-1 不能解决消费者重复问题
需要消费者端幂等处理
```

-1(all):`producer`等待  `broker`的`ack`,`partition`的`leader`和`follower`(这里指的是isr中的follower)全部落盘成功后才返回 `ack`。但是如果在 `follower`同步完成后,`broker`发送`ack`之前,`leader`发生故障,那么会造成数据重复,因为生产者没收到ack,重新选举了新的leader。

-1也可能会发生数据的丢失,发生重复数据的情况是leader接收到数据并且在follower之间已经同步完成后,但是此时leader挂掉,没有返回ack确认,此时又重新选举产生了leader,那么producer会重新发送一次数据,所以会导致数据重复。

**ack=-1数据重复和丢失案例**

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/101329-814647.png)

##### 可靠性高对照表

```java
配置组合               数据丢失风险  重复消费风险  性能影响  实现复杂度
─────────────────────────────────────────────────────────────────────
acks=1, 无幂等          中等          高          低        简单
acks=all, 无幂等        极低          中等        中等      中等
acks=all, 幂等生产者    极低          低          中等      中等  
acks=all, 事务+幂等     几乎为零      几乎为零     高        复杂
```


#### ack = 0
**ack=0会发生丢失数据**

![](./image/ack_0.png)

**小结**

~~~java
request.required.asks=0
~~~
- 0:相当于异步的, 不需要leader给予回复, producer立即返回, 发送就是成功,那么发送消息网络超时或broker crash(1.Partition的Leader还没有commit消息2.Leader与Follower数据不同步), 既有可能丢失也可能会重发
- 1:当leader接收到消息之后发送ack, 丢会重发, 丢的概率很小
- -1:当所有的follower都同步消息成功后发送ack. 不会丢失消息



> **ack是保证生产者生产的数据不丢失,hw是保证消费者消费数据的一致性问题**。hw实际就是最短木桶原则,根据这个原则消费者进行消费数据。不能解决数据重复和丢失问题。ack解决丢失和重复问题。

### 故障处理细节

![](./image/kafka高水位.png)

`LEO（Log End Offset）`:指的是每个副本最大的`offset`,也就是每一个副本的最后offset值。日志末端偏移量;
- 表示日志的当前写入位置
- 用于副本同步进度追踪
- Leader和Follower各自维护自己的LEO
- 每个副本都有自己的LEO，记录自己的写入进度
- 副本同步过程：Follower定期从Leader拉取消息，更新自己的LEO

`HW`（High Watermark）:指的是消费者能见到的最大的 **offset**,**ISR**队列中最小的  **LEO**。
- 所有ISR副本都已复制的消息边界
- 消费者只能读取HW之前的消息
- HW ≤ 所有ISR副本的LEO最小值
- 保证已提交消息（Committed Messages）的持久性
- HW = min(所有ISR副本的LEO)
- 作用:
  - 定义消息的"已提交"状态
  - 控制消费者可见性
  - 保证数据一致性

> HW之前的数据才对consumer可见;

#### HW与消息状态

消息状态分类（基于HW）：

1. 已提交消息（Committed Messages）

- 偏移量 < HW
- 已被所有ISR副本持久化
- 消费者可见且可读取
- Leader故障时不会丢失

2. 未提交消息（Uncommitted Messages）
- HW ≤ 偏移量 < Leader.LEO
- 仅写入Leader，未同步到所有ISR
- 消费者不可见
- Leader故障时可能丢失

3. 未写入消息
- 偏移量 ≥ Leader.LEO
- 尚未写入任何副本

#### LEO与HW的关系

LEO与HW的关系：

1. 在Leader副本上：
- LEO：下一条待写入消息的偏移量
- HW：所有ISR副本的最小LEO

2. 在Follower副本上：
- LEO：本地已复制的最后一条消息偏移量+1
- HW：从Leader获取的HW值

3. 约束关系：
- HW ≤ 每个副本的LEO ≤ Leader.LEO

#### HW推进机制

HW推进的触发条件：

1. Follower拉取消息时：
- Follower向Leader发送Fetch请求
- Leader在响应中包含当前HW
- Follower更新本地HW

2. Leader定期检查：
- Leader检查所有ISR副本的LEO
- 重新计算HW = min(ISR LEOs)
- 更新本地HW

3. Follower确认写入时：
- Follower成功写入消息后
- 发送确认给Leader
- Leader更新该Follower的LEO状态
- 可能触发HW更新

#### Follower故障与ISR管理

1、**Follower故障检测**

```java
 Follower故障处理：
        
1. 故障检测：
- Leader监控Follower拉取进度
- replica.lag.time.max.ms（默认30秒）无进展视为故障

2. 从ISR移除：
- 故障Follower从ISR中移除
- 不再等待其确认

3. 恢复重新加入：
- Follower恢复后重新同步
- 追上进度后重新加入ISR
```

2、**Follower恢复与重新同步**

```java
1. 启动同步
2. 获取同步起点
3. 执行同步
   - 需要从Leader复制数据
   - 只复制未提交数据
4. 检查是否追上
5. 重新加入ISR
6. 更新HW
```

`follower`发生故障后会被临时踢出 ` ISR`,待该` follower`恢复后,`follower`会读取本地磁盘记录的上次的 `HW`,并将 `log`文件高于 ` HW`的部分截取掉,从  `HW`开始向`  leader`进行同步。等该 `follower`的  `LEO`大于等于该`Partition`的`HW`,即` follower`追上 ` leader`之后,就可以重新加入` ISR`了。

#### **leader**故障

1、Leader故障检测机制：

1. 心跳检测：
- Controller定期检查Broker健康状态
- session.timeout.ms（默认10秒）内无心跳视为故障

2. ZooKeeper监控：
- Broker与ZK保持会话
- 会话过期表示Broker故障

3. Follower检测：
- Follower无法从Leader拉取消息
- 报告给Controller

`leader`发生故障之后,会从 `ISR`中选出一个新的 `leader`【优先选择LEO最高的副本】,之后,为保证多个副本之间的数据一致性,其余的 `follower`会先将各自的` log`文件高于`HW`的部分截掉,然后从新的 `leader`同步数据。

2、新Leader选举策略

```java
1. LEO高度：优先选择LEO最高的副本
- 减少数据丢失
- 避免截断过多数据

2. 机架分布：考虑机架感知
- 优先不同机架的副本

3. Broker负载：选择负载较低的Broker

4. 手动偏好：preferred replica（如有配置）
```

3、故障转移中的数据安全:

```java
 Leader故障时的数据安全保证：
        
关键问题：未提交消息的处理

场景分析：
1. 消息已写入旧Leader，但未同步到所有ISR
2. 旧Leader故障，新Leader选举
3. 新Leader可能没有这些未提交消息

解决方案：HW机制保证
- 消费者只能读取已提交消息（HW之前的消息）
- 未提交消息对消费者不可见
- 即使丢失，消费者也不知道
```

**注意:这只能保证副本之间的数据一致性,并不能保证数据不丢失或者不重复,`ack`确认机制可以保证数据的不丢失和不重复,`LEO`和`hw`可以保证数据的一致性问题**

> leader故障后,一般会从isr队列中选中第一个follower作为leader同步数据;

#### 消费者偏移量管理

消费者偏移量管理：

关键概念：
1. 消费位置：消费者已读取的偏移量
2. 提交偏移量：消费者确认已处理的消息位置
3. __consumer_offsets：存储偏移量的内部Topic

故障恢复：
1. 消费者重启：从上次提交的偏移量恢复
2. 分区重平衡：新消费者从提交的偏移量开始
3. 偏移量丢失：根据auto.offset.reset策略处理

## Exactly Once语义

将服务器的 `ACK`级别设置为`1`,可以保证`Producer`到`Server`之间不会丢失数据,即`At Least Once`语义。相对的,将服务器`ACK`级别设置为`0`,可以保证生产者每条消息只会被发送一次,即 `At Most Once`语义。`At Least Once`可以保证数据不丢失,但是不能保证数据不重复;相对的,`At Most Once`可以保证数据不重复,但是不能保证数据不丢失。

但是,对于一些非常重要的信息,比如说交易数据,下游数据消费者要求数据既不重复也不丢失,即` Exactly  Once`语义。在  `0.11`版本以前的` Kafka`,对此是无能为力的,只能保证数据不丢失,再在下游消费者对数据做全局去重。对于多个下游应用的情况,每个都需要单独做全局去重,这就对性能造成了很大影响。`0.11`版本的  `Kafka`,引入了一项重大特性:**幂等性。所谓的幂等性就是指 `Producer`不论向` Server`发送多少次重复数据,`Server`端都只会持久化一条。**

幂等性结合`At  Least Once`语义,就构成了 `Kafka`的 ` Exactly Once`语义。即:`At Least Once +幂等性= Exactly Once`要启用幂等性,只需要将 `Producer`的参数中  `enable.idompotence`设置为`true`即可。

`Kafka`的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的` Producer`在初始化的时候会被分配一个  `PID`,发往同一  `Partition`的消息会附带`Sequence Number`。而`Broker`端会对`<PID,  Partition, SeqNumber>`做缓存,当具有相同主键的消息提交时,`Broker`只会持久化一条。但是 `PID`重启就会变化,同时不同的 ` Partition`也具有不同主键,所以幂等性无法保证跨分区跨会话(也就是重新建立producer链接的情况)的 `Exactly Once`。**即只能保证单次会话不重复问题。幂等性只能解决但回话单分区的问题**。

## kafka 事务了解吗？

Kafka 在 0.11版本引入事务支持,事务可以保证 Kafka 在 Exactly Once 语义的基础上,生产和消费可以跨分区和会话,要么全部成功,要么全部失败。

**Producer 事务**

为了实现跨分区跨会话事务,需要引入一个全局唯一的 `Transaction ID`,**并将 Producer 获取的 PID 和 Transaction ID  绑定**。这样当 Producer 重启后就可以通过正在进行的 Transaction ID 获取原来的 PID。

为了管理 `Transaction`,Kafka 引入了一个新的组件 `Transaction Coordinator`。**Producer 就是通过和 Transaction Coordinator 交互获得 Transaction ID 对应的任务状态**。Transaction Coordinator 还负责将事务所有写入 Kafka 的一个内部 `Topic`,这样即使整个服务重启,由于事务状态得到保存,进行中的事务状态可以得到恢复,从而继续进行。

**Consumer 事务**

上述事务机制主要是从Producer 方面考虑,对于 Consumer 而言,事务的保证就会相对较弱,尤其是无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息,而且不同的Segment File 生命周期不同,同一事务的消息可能会出现重启后被删除的情况。

## kafka消费者

### 消费方式

- `consumer`采用 ` pull`(拉)模式从 `broker`中读取数据。
- `push`(推)模式很难适应消费速率不同的消费者,因为消息发送速率是由 broker决定的。它的目标是尽可能以最快速度传递消息,但是这样很容易造成 `consumer`来不及处理消息,典型的表现就是拒绝服务以及网络拥塞。而` pull`模式则可以根据  `consumer`的消费能力以适当的速率消费消息。对于`Kafka`而言,`pull`模式更合适,它可简化`broker`的设计,`consumer`可自主控制消费消息的速率,同时`consumer`可以自己控制消费方式——即可批量消费也可逐条消费,同时还能选择不同的提交方式从而实现不同的传输语义
- `pull`模式不足之处是,如果 ` kafka`没有数据,消费者可能会陷入循环中,一直返回空数据。针对这一点,`Kafka`的消费者在消费数据时会传入一个时长参数 ` timeout`,如果当前没有数据可供消费,`consumer`会等待一段时间之后再返回,这段时长即为  `timeout`。

### Consumer Group

在 Kafka 中, 一个 Topic 是可以被一个消费组消费, 一个Topic 分发给 Consumer Group 中的Consumer 进行消费, 保证同一条 Message 不会被不同的 Consumer 消费。

> 注意: 当Consumer Group的 Consumer 数量大于 Partition 的数量时, 超过 Partition 的数量将会拿不到消息。

### 分区分配策略

一个consumer group中有多个consumer,一个topic有多个partition,所以必然会涉及到partition的分配问题,即确定那个partition由哪个consumer来消费，Kafka有三种分配策略,一是**RoundRobin**,一是**Range**。高版本还有一个StickyAssignor策略;

将分区的所有权从一个消费者移到另一个消费者称为重新平衡(rebalance)。当以下事件发生时,Kafka 将会进行一次分区分配:

1. 同一个 Consumer Group 内新增消费者。

2. 消费者离开当前所属的Consumer Group,包括shuts down或crashes。

#### Range分区分配策略

Range是对每个Topic而言的(即一个Topic一个Topic分),首先对同一个Topic里面的分区按照序号进行排序,并对消费者按照字母顺序进行排序。然后用Partitions分区的个数除以消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽,那么前面几个消费者线程将会多消费一个分区。假设n=分区数/消费者数量,m=分区数%消费者数量,那么前m个消费者每个分配n+1个分区,后面的(消费者数量-m)个消费者每个分配n个分区。假如有10个分区,3个消费者线程,把分区按照序号排列

```java
//10个分区号
0,1,2,3,4,5,6,7,8,9

//消费者线程为
C1-0,C2-0,C2-1
```

那么用partition数除以消费者线程的总数来决定每个消费者线程消费几个partition,如果除不尽,前面几个消费者将会多消费一个分区。在我们的例子里面,我们有10个分区,3个消费者线程,10/3 = 3,而且除除不尽,那么消费者线程C1-0将会多消费一个分区,所以最后分区分配的结果看起来是这样的:

```java
C1-0:0,1,2,3
C2-0:4,5,6
C2-1:7,8,9
```
如果有11个分区将会是:
```java
C1-0:0,1,2,3
C2-0:4,5,6,7
C2-1:8,9,10
```

假如我们有两个主题T1,T2,分别有10个分区,最后的分配结果将会是这样:
```java
C1-0:T1(0,1,2,3) T2(0,1,2,3)
C2-0:T1(4,5,6) T2(4,5,6)
C2-1:T1(7,8,9) T2(7,8,9)
```

#### RoundRobinAssignor分区分配策略

RoundRobinAssignor策略的原理是将消费组内所有消费者以及消费者所订阅的所有topic的partition按照字典序排序,然后通过轮询方式逐个将分区以此分配给每个消费者. 使用RoundRobin策略有两个前提条件必须满足:

1. 同一个消费者组里面的所有消费者的num.streams(消费者消费线程数)必须相等;
2. 每个消费者订阅的主题必须相同。假如按照 hashCode 排序完的topic-partitions组依次为

```java
T1-5, T1-3, T1-0, T1-8, T1-2, T1-1, T1-4, T1-7, T1-6, T1-9
我们的消费者线程排序为:
C1-0, C1-1, C2-0, C2-1
```

最后分区分配的结果为:

```java
C1-0 将消费 T1-5, T1-2, T1-6 分区
C1-1 将消费 T1-3, T1-1, T1-9 分区
C2-0 将消费 T1-0, T1-4 分区
C2-1 将消费 T1-8, T1-7 分区
```

#### StickyAssignor分区分配策略

Kafka从0.11.x版本开始引入这种分配策略,它主要有两个目的:

分区的分配要尽可能的均匀,分配给消费者者的主题分区数最多相差一个 分区的分配尽可能的与上次分配的保持相同。当两者发生冲突时,第一个目标优先于第二个目标。鉴于这两个目的,StickyAssignor策略的具体实现要比RangeAssignor和RoundRobinAssignor这两种分配策略要复杂很多。

假设消费组内有3个消费者

```java
C0、C1、C2
// 它们都订阅了4个主题:
t0、t1、t2、t3

// 并且每个主题有2个分区,也就是说整个消费组订阅了
t0p0、t0p1、t1p0、t1p1、t2p0、t2p1、t3p0、t3p1这8个分区
```

最终的分配结果如下:

```java
消费者C0:t0p0、t1p1、t3p0
消费者C1:t0p1、t2p0、t3p1
消费者C2:t1p0、t2p1
```
这样初看上去似乎与采用RoundRobinAssignor策略所分配的结果相同

此时假设消费者C1脱离了消费组,那么消费组就会执行再平衡操作,进而消费分区会重新分配。如果采用RoundRobinAssignor策略,那么此时的分配结果如下:

```java
消费者C0:t0p0、t1p0、t2p0、t3p0

消费者C2:t0p1、t1p1、t2p1、t3p1
```

如分配结果所示,RoundRobinAssignor策略会按照消费者C0和C2进行重新轮询分配。而如果此时使用的是StickyAssignor策略,那么分配结果为:

```java
消费者C0:t0p0、t1p1、t3p0、t2p0

消费者C2:t1p0、t2p1、t0p1、t3p1
```

可以看到分配结果中保留了上一次分配中对于消费者C0和C2的所有分配结果,并将原来消费者C1的“负担”分配给了剩余的两个消费者C0和C2,最终C0和C2的分配还保持了均衡。

如果发生分区重分配,那么对于同一个分区而言有可能之前的消费者和新指派的消费者不是同一个,对于之前消费者进行到一半的处理还要在新指派的消费者中再次复现一遍,这显然很浪费系统资源。StickyAssignor策略如同其名称中的“sticky”一样,让分配策略具备一定的“粘性”,尽可能地让前后两次分配相同,进而减少系统资源的损耗以及其它异常情况的发生。

到目前为止所分析的都是消费者的订阅信息都是相同的情况,我们来看一下订阅信息不同的情况下的处理。

举例,同样消费组内有3个消费者:

```java
C0、C1、C2

集群中有3个主题:
t0、t1、t2
这3个主题分别有
1、2、3个分区 

也就是说集群中有
t0p0、t1p0、t1p1、t2p0、t2p1、t2p2这6个分区
```


消费者C0订阅了主题t0

消费者C1订阅了主题t0和t1

消费者C2订阅了主题t0、t1和t2

如果此时采用RoundRobinAssignor策略:

```java
消费者C0:t0p0

消费者C1:t1p0

消费者C2:t1p1、t2p0、t2p1、t2p2
```

如果此时采用的是StickyAssignor策略:

```java
消费者C0:t0p0

消费者C1:t1p0、t1p1

消费者C2:t2p0、t2p1、t2p2
```

此时消费者C0脱离了消费组,那么RoundRobinAssignor策略的分配结果为:

```java
消费者C1:t0p0、t1p1

消费者C2:t1p0、t2p0、t2p1、t2p2
```

StickyAssignor策略,那么分配结果为:

```java
消费者C1:t1p0、t1p1、t0p0

消费者C2:t2p0、t2p1、t2p2
```

可以看到StickyAssignor策略保留了消费者C1和C2中原有的5个分区的分配:
```java
t1p0、t1p1、t2p0、t2p1、t2p2。
```
从结果上看StickyAssignor策略比另外两者分配策略而言显得更加的优异,这个策略的代码实现也是异常复杂。

### Rebalance (重平衡)

Rebalance 本质上是一种协议, 规定了一个 Consumer Group 下的所有 consumer 如何达成一致,来分配订阅 Topic 的每个分区。

Rebalance 发生时, 所有的 Consumer Group 都停止工作, 直到 Rebalance完成。也就是重平衡会产生stop-the-world。

## Coordinator

kafka0.9之后:Group Coordinator 是一个服务, 每个 Broker 在启动的时候都会启动一个该服务, Group Coordinator 的作用是用来存储 Group 的相关 Meta 信息, 并将对应 Partition 的 Offset 信息记录到 Kafka 内置`Topi(__consumer_offsets)`中。

Kafka 在0.9之前是基于 Zookeeper 来存储Partition的 offset信息`(consumers/{group}/offsets/{topic}/{partition})`, 因为 Zookeeper 并不适用于频繁的写操作, 所以在0.9之后通过内置 Topic 的方式来记录对应 Partition 的 offset。

### 触发条件

1. 消费者组成员个数发生变化
    1. 新的消费者加入到消费组
    2. 消费者主动退出消费组
    3. 消费者被动下线. 比如消费者长时间的GC, 网络延迟导致消费者长时间未向Group
       Coordinator发送心跳请求, 均会认为该消费者已经下线并踢出
2. 订阅的 Topic 的 Consumer Group 个数发生变化;
3. Topic 的分区数发生变化

### 如何避免 Rebalance

对于触发条件的 2 和 3, 我们可以人为避免. 1 中的 1 和 2 人为也可以尽量避免, 主要核心为 3

~~~java
心跳相关
session.timeout.ms = 6s
heartbeat.interval.ms = 2s

消费时间
max.poll.interval.ms
~~~


### Rebalace 流程

Rebalance 过程分为两步:Join 和 Sync

1. Join: 顾名思义就是加入组. 这一步中, 所有成员都向 Coordinator 发送 JoinGroup 请求, 请求加入消费组. 一旦所有成员都发送了 JoinGroup 请求, Coordinator 会从中选择一个Consumer 担任 Leader 的角色, 并把组成员信息以及订阅信息发给 Consumer Leader, 注意Consumer Leader 和 Coordinator不是一个概念,Consumer Leader负责消费分配方案的制定.
2. Sync: Consumer Leader 开始分配消费方案, 即哪个 Consumer 负责消费哪些 Topic 的哪些Partition. 一旦完成分配, Leader 会将这个方案封装进 SyncGroup 请求中发给 Coordinator,非 Leader 也会发 SyncGroup 请求, 只是内容为空. Coordinator 接收到分配方案之后会把方案塞进SyncGroup的Response中发给各个Consumer. 这样组内的所有成员就都知道自己应该消费哪些分区了.

![](./image/join_sync过程.png)


## 日志索引

Kafka 能支撑 TB 级别数据, 在日志级别有两个原因:

- 顺序写
- 日志索引

Kafka 在一个日志文件达到一定数据量 (1G) 之后, 会生成新的日志文件, 大数据情况下会有多个日志文件, 通过偏移量来确定到某行纪录时, 如果遍历所有的日志文件, 那效率自然是很差的. Kafka在日志级别上抽出来一层日志索引, 来方便根据 offset 快速定位到是某个日志文件;

每一个 partition 对应多个log文件(最大 1G), 每一个 log 文件又对应一个 index 文件,通过 offset 查找 Message 流程:

1. 先根据 offset (例: 368773), 二分定位到最大 小于等于该 offset 的 index 文件
   (368769.index)
2. 通过二分(368773 - 368769 = 4)定位到 index 文件 (368769.index) 中最大 小于等于该 offset 的 对于的 log 文件偏移量(3, 497)
3. 通过定位到该文件的消息行(3, 497), 然后在往后一行一行匹配揭露(368773 830)

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/09/184632-876670.png)

## 解释如何减少ISR中的扰动？broker什么时候离开ISR？

ISR是一组与leaders完全同步的消息副本,也就是说ISR中包含了所有提交的消息。ISR应该总是包含所有的副本,直到出现真正的故障。

如果一个副本从leader中脱离出来,将会从ISR中删除。那么leader挂掉之后,就会从isr中选取新的leader.

isr就像nameNode和SecondnAMEnODE一样,保存这和Leader完全同步的数据。

扰动,就是说isr中的breaker反复的进入isr列表和退出isr列表,可能是由两个参数控制,第一个是某一个broker多长时间没有和leader同步,或者是相差数据太多导致,可以将这两个参数调节大一点解决。

## ISR、OSR、AR 是什么？

- ISR:In-Sync Replicas 副本同步队列

- OSR:Out-of-Sync Replicas

- AR:Assigned Replicas 所有副本

ISR是由leader维护,follower从leader同步数据有一些延迟(具体可以参见 图文了解 Kafka 的副本复制机制),超过相应的阈值会把 follower 剔除出 ISR, 存入OSR(Out-of-Sync Replicas )列表,新加入的follower也会先存放在OSR中。AR=ISR+OSR

## LEO、HW、LSO、LW等分别代表什么？

LEO:是 LogEndOffset 的简称,代表当前日志文件中下一条。

HW:水位或水印(watermark)一词,也可称为高水位(high watermark),通常被用在流式处理领域(比如Apache Flink、Apache Spark等),以表征元素或事件在基于时间层面上的进度。在Kafka中,水位的概念反而与时间无关,而是与位置信息相关。严格来说,它表示的就是位置信息,即位移(offset)。取 partition 对应的 ISR中 最小的 LEO 作为 HW,consumer 最多只能消费到 HW 所在的位置上一条信息

LSO:是 LastStableOffset 的简称,对未完成的事务而言,LSO 的值等于事务中第一条消息的位置(firstUnstableOffset),对已完成的事务而言,它的值同 HW 相同。

LW:Low Watermark 低水位, 代表 AR 集合中最小的 logStartOffset 值。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/191042-757690.png)

## 如何进行 Leader 副本选举？

每个分区的 leader 会维护一个 ISR 集合,ISR 列表里面就是 follower 副本的 Borker 编号,只有“跟得上” Leader 的 follower 副本才能加入到 ISR 里面,这个是通过 `replica.lag.time.max.ms` 参数配置的。只有 ISR 里的成员才有被选为 leader 的可能。

所以当 Leader 挂掉了,而且 `unclean.leader.election.enable=false` 的情况下,Kafka 会从 ISR 列表中选择 **第一个[leo最大的那个分区]** follower 作为新的 Leader,因为这个分区拥有最新的已经 committed 的消息。通过这个可以保证已经 committed 的消息的数据可靠性。

## 如何进行 broker Leader 选举？

(1) 在 `kafka` 集群中,会有多个 `broker` 节点,集群中第一个启动的 `broker` 会通过在 zookeeper 中创建临时节点 **/controller** 来让自己成为控制器,其他 `broker` 启动时也会在`zookeeper` 中创建临时节点,但是发现节点已经存在,所以它们会收到一个异常,意识到控制器已经存在,那么就会在 `zookeeper` 中创建 **watch** 对象,便于它们收到控制器变更的通知。

(2) 如果集群中有一个 `broker` 发生异常退出了,那么控制器就会检查这个 `broker` 是否有分区的副本 `leader` ,如果有那么这个分区就需要一个新的 `leader`,此时控制器就会去遍历其他副本,决定哪一个成为新的 `leader`,同时更新分区的 `ISR` 集合。

(3) 如果有一个 `broker` 加入集群中,那么控制器就会通过 `Broker ID` 去判断新加入的 `broker` 中是否含有现有分区的副本,如果有,就会从分区副本中去同步数据。

(4) 集群中每选举一次控制器,就会通过 zookeeper 创建一个**controller epoch**,每一个选举都会创建一个更大,包含最新信息的 epoch,如果有 broker 收到比这个 epoch 旧的数据,就会忽略它们,kafka 也通过这个 epoch 来防止集群产生“脑裂”。

## 请说明Kafka 的消息投递保证(delivery guarantee)机制以及如何实现？

Kafka支持三种消息投递语义:

1. At most once:消息可能会丢,但绝不会重复传递

2. At least one:消息绝不会丢,但可能会重复传递

3. Exactly once:每条消息肯定会被传输一次且仅传输一次,很多时候这是用户想要的

consumer在从broker读取消息后,可以选择commit,该操作会在Zookeeper中存下该consumer在该partition下读取的消息的offset,该consumer下一次再读该partition时会从下一条开始读取。如未commit,下一次读取的开始位置会跟上一次commit之后的开始位置相同,可以将consumer设置为autocommit,即consumer一旦读到数据立即自动commit。

如果只讨论这个读取消息的过程,那Kafka是确保了Exactly once。但实际上实际使用中consumer并非读取完数据就结束了,而是要进行进一步处理,而**数据处理与commit的顺序**在很大程度上决定了消息从broker和consumer的delivery guarantee semantic。

读完消息先commit再处理消息。这种模式下,如果consumer在commit后还没来得及处理消息就crash了,下次重新开始工作后就无法读到刚刚已提交而未处理的消息,这就对应于At most once。

读完消息先处理再commit消费状态(保存offset)。这种模式下,如果在处理完消息之后commit之前Consumer crash了,下次重新开始工作时还会处理刚刚未commit的消息,实际上该消息已经被处理过了,这就对应于At least once

如果一定要做到Exactly once,就需要协调offset和实际操作的输出。经典的做法是引入两阶段提交,

但由于许多**输出系统不支持两阶段提交**,更为通用的方式是将offset和操作输入存在同一个地方。比如,consumer拿到数据后可能把数据放到HDFS,如果把最新的offset和数据本身一起写到HDFS,就可以保证数据的输出和offset的更新要么都完成,要么都不完成,间接实现Exactly once。(目前就high level API而言,offset是存于Zookeeper中的,无法存于HDFS,而low level API的offset是由自己去维护的,可以将之存于HDFS中)

总之,Kafka默认保证At least once,并且允许通过设置producer异步提交来实现At most once,而Exactly once要求与目标存储系统协作,Kafka提供的offset可以较为容易地实现这种方式。

## Kafka 的高可靠性是怎么实现的？

> 注意:也可回答“Kafka在什么情况下会出现消息丢失？" 据可靠性(可回答“怎么尽可能保证Kafka的可靠性？”)

Kafka 作为一个商业级消息中间件,消息可靠性的重要性可想而知。本文从Producter向Broker发送消息、Topic 分区副本以及 Leader选举几个角度介绍数据的可靠性。

### Topic分区副本

在 Kafka 0.8.0 之前,Kafka 是没有副本的概念的,那时候人们只会用 Kafka 存储一些不重要的数据；

因为没有副本,数据很可能会丢失。但是随着业务的发展,支持副本的功能越来越强烈,所以为了保证数据的可靠性,Kafka 从 0.8.0 版本开始引入了分区副本(详情请参见 KAFKA-50)。也就是说每个分区可以人为的配置几个副本(比如创建主题的时候指定 replication-factor,也可以在 Broker 级别进行配置 default.replication.factor),一般会设置为3；

Kafka 可以保证单个分区里的事件是有序的,分区可以在线(可用),也可以离线(不可用)。在众多的分区副本里面有一个副本是 Leader,其余的副本是 follower,所有的读写操作都是经过 Leader 进行的,同时 follower 会定期地去 leader 上的复制数据。当 Leader 挂了的时候,其中一个 follower 会重新成为新的 Leader。通过分区副本,引入了数据冗余,同时也提供了 Kafka 的数据可靠性。

Kafka 的**分区多副本架构**是 Kafka 可靠性保证的核心,把消息写入多个副本可以使 Kafka 在发生崩溃时仍能保证消息的持久性。

### Producer往Broker 发送消息

如果我们要往 Kafka 对应的主题发送消息,我们需要通过 Producer 完成。前面我们讲过 Kafka 主题对应了多个分区,每个分区下面又对应了多个副本;为了让用户设置数据可靠性, Kafka 在 Producer 里面提供了消息确认机制。也就是说我们可以通过配置来决定消息发送到对应分区的几个副本才算消息发送成功。可以在定义 Producer 时通过 acks 参数指定(在 0.8.2.X 版本之前是通过request.required.acks 参数设置的)。

这个参数支持以下三种值:

- acks = 0:意味着如果生产者能够通过网络把消息发送出去,那么就认为消息已成功写入Kafka。在这种情况下还是有可能发生错误,比如发送的对象无能被序列化或者网卡发生故障,但如果是分区离线或整个集群长时间不可用,那就不会收到任何错误。在 acks=0模式下的运行速度是非常快的(这就是为什么很多基准测试都是基于这个模式),你可以得到惊人的吞吐量和带宽利用率,不过如果选择了这种模式,一定会丢失一些消息。

- acks = 1:意味若 Leader 在收到消息并把它写入到分区数据文件(不一定同步到磁盘上)时会返回确认或错误响应。在这个模式下,如果发生正常的 Leader 选举,生产者会在选举时收到一个
LeaderNotAvailableException 异常,如果生产者能恰当地处理这个错误,它会重试发送悄息,最终消息会安全到达新的 Leader 那里。不过在这个模式下仍然有可能丢失数据,比如消息已经成功写入Leader,但在消息被复制到 follower 副本之前 Leader发生崩溃

- acks = all(这个和 request.required.acks = -1 含义一样):意味着 Leader 在返回确认或错误响应之前,会等待所有同步副本都收到悄息。如果和 min.insync.replicas 参数结合起来,就可以决定在返回确认前至少有多少个副本能够收到悄息,生产者会一直重试直到消息被成功提交。不过这也是最慢的做法,因为生产者在继续发送其他消息之前需要等待所有副本都收到当前的消息

根据实际的应用场景,我们设置不同的 acks,以此保证数据的可靠性

## Leader 选举

在介绍 Leader 选举之前,让我们先来了解一下 ISR(in-sync replicas)列表。每个分区的 leader 会维护一个 ISR 列表,ISR 列表里面就是 follower 副本的 Borker 编号,只有跟得上 Leader 的 follower 副本才能加入到 ISR 里面,这个是通过 replica.lag.time.max.ms 参数配置的。只有 ISR 里的成员才有被选为 leader 的可能。

## 数据一致性(可回答“Kafka数据一致性原理？”)

这里介绍的数据一致性主要是说不论是老的 Leader 还是新选举的 Leader,Consumer 都能读到一样的数据。那么 Kafka 是如何实现的呢？

> 高水位

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/140242-153085.png)

假设分区的副本为3,其中副本0是 Leader,副本1和副本2是 follower,并且在 ISR 列表里面。虽然副本0已经写入了 Message4,但是 Consumer 只能读取到 Message2。因为所有的 ISR 都同步了Message2,只有 High Water Mark 以上的消息才支持 Consumer 读取,而 High Water Mark 取决于 ISR 列表里面偏移量最小的分区,对应于上图的副本2,这个很类似于木桶原理，这样做的原因是还没有被足够多副本复制的消息被认为是“不安全”的,如果 Leader 发生崩溃,另一个副本成为新 Leader,那么这些消息很可能丢失了。如果我们允许消费者读取这些消息,可能就会破坏一致性。试想,一个消费者从当前 Leader(副本0) 读取并处理了 Message4,这个时候 Leader 挂掉了,选举了副本1为新的 Leader,这时候另一个消费者再去从新的 Leader 读取消息,发现这个消息其实并不存在,这就导致了数据不一致性问题

当然,引入了 High Water Mark 机制,会导致 Broker 间的消息复制因为某些原因变慢,那么消息到达消费者的时间也会随之变长(因为我们会先等待消息复制完毕)。延迟时间可以通过参数
replica.lag.time.max.ms 参数配置,它指定了副本在复制消息时可被允许的最大延迟时间。

## Kafka 分区数可以增加或减少吗？为什么？

我们可以使用 bin/kafka-topics.sh 命令对 Kafka 增加 Kafka 的分区数据,但是 Kafka 不支持减少分区数。 Kafka 分区数据不支持减少是由很多原因的,比如减少的分区其数据放到哪里去？是删除,还是保留？删除的话,那么这些没消费的消息不就丢了。如果保留这些消息如何放到其他分区里面？追加到其分区后面的话那么就破坏了 Kafka 单个分区的有序性。如果要保证删除分区数据插入到其他分区保证有序性,那么实现起来逻辑就会非常复杂.

## Kafka消息可靠性的保证

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/105014-372899.png)

Kafka存在丢消息的问题,消息丢失会发生在**Broker,Producer和Consumer**三种。

### Broker

**Broker丢失消息是由于Kafka本身的原因造成的,kafka为了得到更高的性能和吞吐量,将数据异步批量的存储在磁盘中**。消息的刷盘过程,为了提高性能,减少刷盘次数,kafka采用了批量刷盘的做法。即,按照一定的消息量,和时间间隔进行刷盘。

那么kafka首先会将数据写入**内存页**中,系统通过刷盘的方式将数据持久化到内存当中,但是在存储在内存那一会,如果发生断电行为,内存中的数据是有可能发生丢失的,也就是说kafka中的数据可能丢失。

Broker配置刷盘机制,是通过调用fsync函数接管了刷盘动作。从单个Broker来看,pageCache的数据会丢失。

也就是说,理论上,要完全让kafka保证单个broker不丢失消息是做不到的,只能通过调整刷盘机制的参数缓解该情况。

- 比如,减少刷盘间隔,减少刷盘数据量大小,也就是频繁的刷盘操作。时间越短,性能越差,可靠性越好(尽可能可靠)。这是一个选择题。

- 而减少刷盘频率,可靠性不高,但是性能好。

为了解决该问题,kafka通过producer和broker协同处理单个broker丢失参数的情况。一旦producer发现broker消息丢失,即可自动进行retry。除非retry次数超过阀值(可配置),消息才会丢失。此时需要生产者客户端手动处理该情况。那么producer是如何检测到数据丢失的呢？是通过ack机制,类似于http的三次握手的方式。

- acks=0,producer不等待broker的响应,效率最高,但是消息很可能会丢。这种情况也就是说还没有等待leader同步完数据,所以肯定发生数据的丢失。

- acks=1,leader broker收到消息后,不等待其他follower的响应,即返回ack。也可以理解为ack数为1。此时,如果follower还没有收到leader同步的消息leader就挂了,那么消息会丢失,也就是如果leader收到消息,成功写入PageCache后,会返回ack,此时producer认为消息发送成功。但此时,数据还没有被同步到follower。如果此时leader断电,数据会丢失。

- acks=-1,leader broker收到消息后,挂起,等待所有ISR列表中的follower返回结果后,再返回ack。-1等效与all。这种配置下,只有leader写入数据到pagecache是不会返回ack的,还需要所有的ISR返回“成功”才会触发ack。如果此时断电,producer可以知道消息没有被发送成功,将会重新发送。如果在follower收到数据以后,成功返回ack,leader断电,数据将存在于原来的follower中。在重新选举以后,新的leader会持有该部分数据。数据从leader同步到follower,需要2步:

    - 数据从pageCache被刷盘到disk。因为只有disk中的数据才能被同步到replica。

    - 数据同步到replica,并且replica成功将数据写入PageCache。在producer得到ack后,哪怕是所有机器都停电,数据也至少会存在于leader的磁盘内。

那么在这上面提到一个isr的概念,可以想象以下,如果leader接收到消息之后,一直等待所有的followe返回ack确认,但是有一个发生网络问题,始终无法返回ack怎么版？

显然这种情况不是我们希望的,所以就产生了isr,这个isr就是和leader同步数据的最小子集和,只要在isr中的follower,那么leader必须等待同步完消息并且返回ack才可以,否则就不反悔ack。isr的个数通常通过``min.insync.replicas`参数配置。

借用网上的一张图,感觉说的很明白:

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/145111-262605.png)

0,1,-1性能一次递减,但是可靠性一直在提高。

### Producer

Producer丢失消息,发生在**生产者客户端**。

为了提升效率,减少IO,producer在发送数据时可以将多个请求进行合并后发送。被合并的请求先缓存在本地buffer中。缓存的方式和前文提到的刷盘类似,**producer可以将请求打包成“块”或者按照时间间隔**,将buffer中的数据发出。**通过buffer我们可以将生产者改造为异步的方式,而这可以提升我们的发送效率。**

但是,buffer中的数据就是危险的。在正常情况下,客户端的异步调用可以通过callback来处理消息发送失败或者超时的情况,但是,一旦producer被非法的停止了,那么buffer中的数据将丢失,broker将无法收到该部分数据。又或者,当Producer客户端内存不够时,如果采取的策略是丢弃消息(另一种策略是block阻塞),消息也会被丢失。抑或,消息产生(异步产生)过快,导致挂起线程过多,内存不足,导致程序崩溃,消息丢失。

![1639192224822](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/153515-998847.png)

![1639192247686](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/111054-717824.png)

根据上图,可以想到几个解决的思路:

- 异步发送消息改为同步发送。或者service产生消息时,使用阻塞的线程池,并且线程数有一定上限。整体思路是控制消息产生速度。

- 扩大Buffer的容量配置。这种方式可以缓解该情况的出现,但不能杜绝。

- service不直接将消息发送到buffer(内存),而是将消息写到本地的磁盘中(数据库或者文件),由另一个(或少量)生产线程进行消息发送。相当于是在buffer和service之间又加了一层空间更加富裕的缓冲层

### Consumer消费消息

- 接收消息

- 处理消息

- 反馈“处理完毕”(commited)

Consumer的消费方式主要分为两种:

- 自动提交offset,Automatic Offset Committing

- 手动提交offset,Manual Offset Control

Consumer自动提交的机制是根据一定的**时间间隔**,将收到的消息进行commit。**commit过程和消费消息的过程是异步的**。也就是说,可能存在消费过程未成功(比如抛出异常),commit消息已经提交了。此时消息就丢失了。

~~~ java
Properties props = new Properties();

props.put("bootstrap.servers", "localhost:9092");

props.put("group.id", "test");
// 自动提交开关props.put("enable.auto.commit", "true");
// 自动提交的时间间隔,此处是1sprops.put("auto.commit.interval.ms", "1000");props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");KafkaConsumer consumer = new KafkaConsumer<>(props);consumer.subscribe(Arrays.asList("foo", "bar"));while (true) {
// 调用poll后,1000ms后,消息状态会被改为 committed ConsumerRecords records = consumer.poll(100);for (ConsumerRecord record : records)  insertIntoDB(record); 
// 将消息入库,时间可能会超过1000ms}*

~~~

上面的示例是自动提交的例子。如果此时,`insertIntoDB(record)`发生异常,消息将会出现丢失。接下来是手动提交的例子:

~~~ java
Properties props = new **Properties**();

props.put("bootstrap.servers", "localhost:9092");

props.put("group.id", "test");
// 关闭自动提交,改为手动提交
props.put("enable.auto.commit", "false");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
KafkaConsumer consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("foo", "bar"));
final int minBatchSize = 200;
List> buffer = new ArrayList<>();
while (true) {
// 调用poll后,不会进行auto commit 
ConsumerRecords records = consumer.poll(100);
for (ConsumerRecord record : records) {  buffer.add(record); }
if (buffer.size() >= minBatchSize) {  insertIntoDb(buffer);
// 所有消息消费完毕以后,才进行commit操作  consumer.commitSync();  
buffer.clear(); 
}}

~~~

将提交类型改为手动以后,可以保证消息“至少被消费一次”(at least once)。但此时可能出现重复消费的情况。也就是数据处理完成之后,手动进行提交的方式。

> 另外,Producer 发送消息还可以选择同步或异步模式,如果设置成异步,虽然会极大的提高消息发送的性能,但是这样会增加丢失数据的风险。如果需要**确保消息的可靠性**,必须将 producer.type 设置为 sync。

## 为什么kafka中1个partition只能被同组的一个consumer消费?

Kafka通过消费者组机制同时实现了**发布/订阅模型和点对点模型**。**多个组的消费者消费同一个分区属于多订阅者的模式**,自然没有什么问题;

**而在单个组内某分区只交由一个消费者处理的做法则属于点对点模式**。其实这就是设计上的一种取舍,如果Kafka真的允许组内多个消费者消费同一个分区,也不是什么灾难性的事情,只是没什么意义,而且还会重复消费消息。

通常情况下,我们还是希望一个组内所有消费者能够分担负载,让彼此做的事情没有交集,做一些重复性的劳动纯属浪费资源。就如同电话客服系统,每个客户来电只由一位客服人员响应。那么请问我就是想让多个人同时接可不可以？当然也可以了,我不觉得技术上有什么困难,只是这么做没有任何意义罢了,既拉低了整体的处理能力,也造成了人力成本的浪费。

还由另外一点,如果让一个消费者组中的多个消费者消费同一个分区数据,那么我们保证多个消费者之间顺序的去消费数据的话,这里就产生了线程安全的问题,导致系统的设计更加的复杂。

总之,我的看法是这种设计不是出于技术上的考量而更多还是看效率等非技术方面。

## kafka和zookeeper的关系

**kafka** 使用 **zookeeper** 来保存集群的元数据信息和消费者信息(偏移量),没有 zookeeper,kafka 是工作不起来。在 `zookeeper` 上会有一个专门用来进行 `Broker` 服务器列表记录的点,节点路径为`/brokers/ids`。

每个 Broker 服务器在启动时,都会到 Zookeeper 上进行注册,即创建 `/brokers/ids/[0-N]` 的节点,然后写入 IP,端口等信息,**Broker 创建的是临时节点**,所以一旦 Broker 上线或者下线,对应 Broker 节点也就被删除了,因此可以通过 zookeeper 上 Broker 节点的变化来动态表征 Broker 服务器的可用性。

## zookeeper在kafka中的作用

`Kafka`集群中有一个 ` broker`会被选举为 `  Controller`,负责管理集群` broker`的上下线,所有` topic`的分区副本分配和`  leader`选举等工作。

`Controller`的管理工作都是依赖于 ` Zookeeper`的。

![](./image/zookeeper作用.png)

Apache Kafka是一个使用Zookeeper构建的分布式系统。Zookeeper的主要作用是在集群中的不同节点之间建立协调;如果任何节点失败,我们还使用Zookeeper从先前提交的偏移量中恢复,因为它做周期性提交偏移量工作。

**说明**

- 从Zookeeper中读取当前分区的所有ISR(in-sync replicas)集合
- 调用配置的分区选择算法选择分区的leader

**作用**

### Broker注册

Broker是分布式部署并且相互之间相互独立,**但是需要有一个注册系统能够将整个集群中的Broker管理起来**,此时就使用到了Zookeeper。在Zookeeper上会有一个专门用来进行Broker服务器列表记录的节点。

### Topic注册

在Kafka中,同一个Topic的消息会被分成多个分区并将其分布在多个Broker上,这些分区信息及与Broker的对应关系也都是由Zookeeper在维护,由专门的节点来记录。

### 生产者负载均衡

由于同一个Topic消息会被分区并将其分布在多个Broker上,因此,生产者需要将消息合理地发送到这些分布式的Broker上,那么如何实现生产者的负载均衡,Kafka支持传统的四层负载均衡,也支持Zookeeper方式实现负载均衡。

### 消费者负载均衡

与生产者类似,Kafka中的消费者同样需要进行负载均衡来实现多个消费者合理地从对应的Broker服务器上接收消息,每个消费者分组包含若干消费者,每条消息都只会发送给分组中的一个消费者,不同的消费者分组消费自己特定的Topic下面的消息,互不干扰。

### 分区与消费者的关系

消费组(Consumer Group):consumer group下有多个Consumer(消费者),对于每个消费者组 (Consumer Group),Kafka都会为其分配一个全局唯一的Group ID,Group 内部的所有消费者共享该 ID。 订阅的topic下的每个分区只能分配给某个 group 下的一个consumer(当然该分区还可以被分配给其他group)。 同时,Kafka为每个消费者分配一个Consumer ID,通常采用”Hostname:UUID”形式表示。

在Kafka中,规定了每个消息分区只能被同组的一个消费者进行消费,因此,需要在Zookeeper上记录 消息分区 与 Consumer之间的关系,每个消费者一旦确定了对一个消息分区的消费权力,需要将其Consumer ID 写入到 Zookeeper 对应消息分区的临时节点上。

### 消费进度Offset记录

在消费者对指定消息分区进行消息消费的过程中,需要定时地将分区消息的消费进度Offset记录到Zookeeper上,以便在该消费者进行重启或者其他消费者重新接管该消息分区的消息消费后,能够从之前的进度开始继续进行消息消费。Offset在Zookeeper中由一个专门节点进行记录。节点内容是Offset的值。

### 消费者注册

每个消费者服务器启动时,都会到Zookeeper的指定节点下创建一个属于自己的消费者节点。

早期版本的Kafka用zk做meta信息存储,consumer的消费状态,group的管理以及offset的值。考虑到zk本身的一些因素以及整个架构较大概率存在单点问题,新版本中确实逐渐弱化了zookeeper的作用。新的consumer使用了kafka内部的group coordination协议,也减少了对zookeeper的依赖。

Zookeeper是一个开放源码的、高性能的协调服务,它用于Kafka的分布式应用,kafka不可能越过Zookeeper直接联系Kafka broker,一旦Zookeeper停止工作,它就不能服务客户端请求。

Zookeeper主要用于在集群中不同节点之间进行通信,在Kafka中,它被用于提交偏移量,因此如果节点在任何情况下都失败了,它都可以从之前提交的偏移量中获取,除此之外,它还执行其他活动,如:

> leader检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等


## Kafka中的ZooKeeper是什么？Kafka是否可以脱离ZooKeeper独立运行？

本篇针对的是2.8版本之前的Kafka,2.8版本及之后Kafka已经移除了对Zookeeper的依赖,通过KRaft进行自己的集群管理,不过目前只是测试阶段。

Zookeeper是一个开放源码的、高性能的协调服务,它用于Kafka的分布式应用。

不可能越过Zookeeper直接联系Kafka broker,一旦Zookeeper停止工作,它就不能服务客户端请求。

Zookeeper主要用于在集群中不同节点之间进行通信,在Kafka中,它被用于提交偏移量,因此如果节点在任何情况下都失败了,它都可以从之前提交的偏移量中获取,除此之外,它还执行其他活动,如: leader检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等。

一个消费者组中只有一个消费者可以消费分区数据,这样所还可以保证线程的安全性,如果由多个消费者可以消费一个分区中的数据,那么如和保证多个线程之间顺序的消费这一个分区中的数据,可能还需要添加锁机制,所以提高了系统的复杂度。

## Kafka的高性能的原因

- 高吞吐

    - 顺序读写
    - 零拷贝
    - 分区+分段(建立索引):并行度高,每一个分区分为多个segment,每一次操作都是针对一小部分数据,并且增加了并行操作的能力。
    - 批量发送:kafka允许进行批量发送消息,producter发送消息的时候,可以将消息缓存在本地,等到了固定条件发送到kafka
        - 等到多少条消息后发送。
        - 等待多长时间后发送。
    - 数据压缩:Kafka还支持对消息集合进行压缩,Producer可以通过GZIP或Snappy格式对消息集合进行压缩压缩的好处就是减少传输的数据量,减轻对网络传输的压力
        - producer在发送的压缩数据,kafk不会解压缩,而是直接存储压缩的文件。

- 容错性:

    - 集群的容错性
    - partition的leader的容错性
    - 数据有副本,保证数据的容错性

- 高性能

    - 顺序读写(比随机读写号很多)
        - Kafka是通过文件追加的方式来写入消息的,只能在日志文件的最后追加新的消息,并且不允许修改已经写入的消息,这种方式就是顺序写磁盘,而顺序写磁盘的速度是非常快的。
    - 零拷贝技术
    - 日志分段存储
        - 为了防止日志(Log)过大,Kafka引入了日志分段(LogSegment)的概念,将日志切分成多个日志分段。在磁盘上,日志是一个目录,每个日志分段对应于日志目录下的日志文件、偏移量索引文件、时间戳索引文件(可能还有其他文件)
        - 向日志中追加消息是顺序写入的,只有最后一个日志分段才能执行写入操作,之前所有的日志分段都不能写入数据。
        - 为了便于检索,每个日志分段都有两个索引文件:偏移量索引文件和时间戳索引文件。每个日志分段都有一个基准偏移量baseOffset,用来表示当前日志分段中第一条消息的offset。偏移量索引文件和时间戳索引文件是以稀疏索引的方式构造的,偏移量索引文件中的偏移量和时间戳索引文件中的时间戳都是严格单调递增的。
        - 查询指定偏移量(或时间戳)时,使用二分查找快速定位到偏移量(或时间戳)的位置。可见Kāfk中对消息的查找速度还是非常快的。
    - 分区存储:producer可以将数据发送到一个topic下面的多个分区,而这些分区的leadder是部署在不同的节点机器上的,这样的话,肯定比将数据发送到一台机器上性能好,对于消费者,一个分区只能由一个消费者组中的一个消费者消费,这样保证不会重复的消费消息,而多个消费者可以在不同的分区中消费消息,相当于并行读写,所以性能高。
        - 分区的设计使得Kafka消息的读写性能可以突破单台broker的/O性能瓶颈,可以在创建主题的时候指定分区数,也可以在主题创建完成之后去修改分区数,通过增加分区数可以实现水平扩展,但是要注意,分区数也不是越多越好,一般达到煤一个阈值之后,再增加分区数性能反而会下降,具体阈值需要对Kak集群进行压测才能确定。
    - 页缓存技术:kafka中使用页缓存技术,把对磁盘的io操作,转换为对内存的操作,速度非常快,极大的提高磁盘Io的性能。

  > 补充什么是页缓存:
  >
  > 页缓存是操作系统实现的一种主要的磁盘缓存,以此用来减少对磁盘I/O的操作。具体来说,就是把磁盘中的数据缓存到内存中,把对磁盘的访问变为对内存的访问。为了弥补性能上的差异   ,现代操作系统越来越多地将内存作为磁盘缓存,甚至会将所有可用的内存用途磁盘缓存,这样当内存回收时也几乎没有性能损失,所有对于磁盘的读写也将经由统一的缓存。
  >
  >
  >
  > 当一个进程准备读取磁盘上的文件内容时,操作系统会先查看待读取的数据所在的页(page)是否在页缓存(page  cache)中,如果存在(命中)则直接返回数据,从而避免了对物理磁盘I/O操作;如果没有命中,则操作系统会向磁盘发起读取请示并将读取的数据页写入页缓存,之后再将数据返回进程。同样,如果一个进程需要将数据写入磁盘,那么操作系统也会检测数据对应的页是否在页缓存中,如果不存在,则会先在页缓存中添加相应的页,最后将数据写入对应的页。被修改过后的页也就变成了脏页,操作系统会在合适的时间把脏页中的数据写入磁盘,以操作数据的一致性。
  >
  >
  >
  > Kafka中大量使用了页缓存,这是Kafka实现高吞吐的重要因此之一。虽然消息都是先被写入页缓存,然后由操作系统负责具体的刷盘任务,但在Kafka中同样提供了同步刷盘及间断性强制刷盘(fsync)的功能,这些功能可以通过log.flush.interval.message、log.flush.interval.ms等参数来控制。同步刷盘可以提高消息的可行性,防止由于机器掉电等异常造成处于页缓存而没有及时写入磁盘的消息丢失。不过一般不建议这么做,刷盘任务就应交由操作系统去调配,消息的可靠性应该由多副本机制来保障,而不是由同步刷盘这种严重影响性能的行为来保障。

## kafka broker 挂了怎么办

controller在启动时会注册zk监听器来监听zookeeper中的/brokers/ids节点下的子节点变化,即集群中所有的broker列表,而每台broker在启动时会向zk的/brokers/ids下写入一个名字为broker.id的临时节点,当该broker挂掉或与zk断开连接时,此临时节点会被移除,之后controller端的监听器就会自动感知这个变化并将BrokerChange时间写入到controller上的请求阻塞队列里。

一旦controller端从阻塞队列中获取到该事件,她会开启BrokerChange事件的处理逻辑,具体包括

1 获取当前存活的broker列表

2 根据之前缓存的broker列表计算出当前已经挂掉的broker列表

3 更新controller端缓存

4 对于当前所有存活的broker,更新元数据信息并且启动新broker上的分区和副本

5 对于挂掉的那些broker,处理这些broker上的分区副本(标记为offline已经执行offline逻辑并更新元数据)

## 关于kafka的isr机制

### kafka replica

1. 当某个topic的replication-factor为N且N大于1时,每个Partition都会有N个副本(Replica)。**kafka的replica包含leader与follower**。
2. Replica的个数小于等于Broker的个数,也就是说,对于每个Partition而言,每个Broker上最多只会有一个Replica,因此可以使用Broker id 指定Partition的Replica。
3. 所有Partition的Replica默认情况会均匀分布到所有Broker上。

### Data Replication如何Propagate(扩散出去)消息？

每个Partition有一个leader与多个follower,producer往某个Partition中写入数据是,只会往leader中写入数据,然后数据才会被复制进其他的Replica中。

**数据是由leader push过去还是有flower pull过来？**

kafka是由follower周期性或者尝试去pull(拉)过来(其实这个过程与consumer消费过程非常相似),写是都往leader上写,但是读并不是任意flower上读都行,读也只在leader上读,flower只是数据的一个备份,保证leader被挂掉后顶上来,并不往外提供服务。

## Data Replication何时Commit？

**同步复制:** 只有所有的follower把数据拿过去后才commit,一致性好,可用性不高。

**异步复制:** 只要leader拿到数据立即commit,等follower慢慢去复制,可用性高,立即返回,一致性差一些。
Commit:是指leader告诉客户端,这条数据写成功了。kafka尽量保证commit后立即leader挂掉,其他flower都有该条数据。

**kafka不是完全同步,也不是完全异步,是一种ISR机制:**

1. leader会维护一个与其基本保持同步的Replica列表,该列表称为ISR(in-sync Replica),每个Partition都会有一个ISR,而且是由leader动态维护
2. 如果一个flower比一个leader落后太多,或者超过一定时间未发起数据复制请求,则leader将其重ISR中移除
3. 当ISR中所有Replica都向Leader发送ACK时,leader才commit

**既然所有Replica都向Leader发送ACK时,leader才commit,那么flower怎么会leader落后太多？**

producer往kafka中发送数据,不仅可以一次发送一条数据,还可以发送message的数组;批量发送,同步的时候批量发送,异步的时候本身就是就是批量;底层会有队列缓存起来,批量发送,对应broker而言,就会收到很多数据(假设1000),这时候leader发现自己有1000条数据,flower只有500条数据,落后了500条数据,就把它从ISR中移除出去,这时候发现其他的flower与他的差距都很小,就等待;如果因为内存等原因,差距很大,就把它从ISR中移除出去。

**commit策略:**

server配置

```java
  rerplica.lag.time.max.ms=10000
  # 如果leader发现flower超过10秒没有向它发起fech请求,那么leader考虑这个flower是不是程序出了点问题
  # 或者资源紧张调度不过来,它太慢了,不希望它拖慢后面的进度,就把它从ISR中移除。

  rerplica.lag.max.messages=4000 # 相差4000条就移除
  # flower慢的时候,保证高可用性,同时满足这两个条件后又加入ISR中,
  # 在可用性与一致性做了动态平衡   亮点1234567
```

topic配置

```java
  min.insync.replicas=1 # 需要保证ISR中至少有多少个replica1
```

producer配置

```java
 request.required.asks=0
  # 0:相当于异步的,不需要leader给予回复,producer立即返回,发送就是成功,
      那么发送消息网络超时或broker crash(1.Partition的Leader还没有commit消息 2.Leader与Follower数据不同步),
      既有可能丢失也可能会重发  # 1:当leader接收到消息之后发送ack,丢会重发,丢的概率很小
  # -1:当所有的follower都同步消息成功后发送ack.  丢失消息可能性比较低123456
```

## Data Replication如何处理Replica恢复

leader挂掉了,从它的follower中选举一个作为leader,并把挂掉的leader从ISR中移除,继续处理数据。一段时间后该leader重新启动了,它知道它之前的数据到哪里了,尝试获取它挂掉后leader处理的数据,获取完成后它就加入了ISR。

## Data Replication如何处理Replica全部宕机

**1、等待ISR中任一Replica恢复,并选它为Leader**

1. 等待时间较长,降低可用性
2. 或ISR中的所有Replica都无法恢复或者数据丢失,则该Partition将永不可用

**2、选择第一个恢复的Replica为新的Leader,无论它是否在ISR中**

1. 并未包含所有已被之前Leader Commit过的消息,因此会造成数据丢失
2. 可用性较高

## Exactly Once语义

将服务器的ACK级别设置为-1,可以保证Producer到Server之间不会丢失数据,即At Least Once语义。相对的,将服务器ACK级别设置为0,可以保证生产者每条消息只会被发送一次,即At Most Once语义。

At Least Once可以保证数据不丢失,但是不能保证数据不重复;

相对的,At Most Once可以保证数据不重复,但是不能保证数据不丢失。

但是,对于一些非常重要的信息,比如说交易数据,下游数据消费者要求数据既不重复也不丢失,即Exactly Once语义。在0.11版本以前的Kafka,对此是无能为力的,只能保证数据不丢失,再在下游消费者对数据做全局去重。对于多个下游应用的情况,每个都需要单独做全局去重,这就对性能造成了很大影响。

0.11版本的Kafka,引入了一项重大特性:幂等性。

开启幂等性`enable.idempotence=true`。

所谓的幂等性就是指Producer不论向Server发送多少次重复数据,Server端都只会持久化一条。幂等性结合At Least Once语义,就构成了Kafka的Exactly Once语义。即:

At Least Once + 幂等性 = Exactly Once

Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的Producer在初始化的时候会被分配一个PID,发往同一Partition的消息会附带Sequence Number。而Broker端会对<PID, Partition, SeqNumber>做缓存,当具有相同主键的消息提交时,Broker只会持久化一条。

但是PID重启就会变化,同时不同的Partition也具有不同主键,所以幂等性无法保证跨分区跨会话的Exactly Once。

补充,在流式计算中怎么Exactly Once语义？以flink为例

1. souce:使用执行ExactlyOnce的数据源,比如kafka等

内部使用FlinkKafakConsumer,并开启CheckPoint,偏移量会保存到StateBackend中,并且默认会将偏移量写入到topic中去,即_consumer_offsets Flink设置CheckepointingModel.EXACTLY_ONCE

2. sink

存储系统支持覆盖也即幂等性:如Redis,Hbase,ES等 存储系统不支持覆:需要支持事务(预写式日志或者两阶段提交),两阶段提交可参考Flink集成的kafka sink的实现。