---
title: K8S部署kafka集群
icon: material-icon-theme:vfl
order: 5
author: bugcode
date: 2024-12-7
category:
  - K8S
tag:
  - 面试
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/k8sDeployKafka/
---

# K8S部署kafka集群

## 部署Zookeeper集群

为什么部署kafka前要部署zookeeper？

Kafka依赖Zookeeper来实现分布式协调和配置管理。在Kafka集群中，Zookeeper扮演着多种角色，包括：

配置管理：Kafka集群的配置信息和元数据存储在Zookeeper中，包括主题（topics）、分区（partitions）、副本（replicas）等配置信息。

1. Leader选举：Kafka的分区（partitions）被分布式存储在集群中的多个Broker上，每个分区都有一个Leader和多个Follower。Zookeeper负责Leader选举，确保每个分区都有一个活跃的Leader。
2. Broker注册：Kafka Broker在启动时会向Zookeeper注册自己的信息，包括地址、ID等，以便其他Broker和客户端发现和连接。
3. 健康监测：Zookeeper监控Kafka集群中各个节点的健康状态，并在节点出现故障或宕机时触发相应的处理操作。

因此，在部署Kafka之前，需要先部署Zookeeper，确保Kafka集群正常运行所需的分布式协调和配置管理功能可用。没有Zookeeper，Kafka无法正常运行，并且无法实现高可用性、数据一致性和故障容错等特性。

### 创建命名空间

```yaml
# 添加如下内容
apiVersion: v1
kind: Namespace
metadata:
  name: kafka
```

### 创建Service资源

创建集群内部访问的Service资源:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-cluster  #无头服务的名称，需要通过这个获取ip，与主机的对应关系
  namespace: kafka
  labels:
    app: zookeeper
spec:
  ports:
    - port: 2181
      name: zookeeper
    - port: 2188
      name: cluster1
    - port: 3888
      name: cluster2
  clusterIP: None   # 此类型的Service没有ip
  selector:
    app: zookeeper
```

创建NodePort类型的Service资源,通过主机暴漏端口访问

```yaml
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-nodeport-service-0
  namespace: kafka
spec:
  type: NodePort
  selector:
    statefulset.kubernetes.io/pod-name: zookeeper-0
  ports:
    - protocol: TCP
      port: 80        # Service 暴露的端口
      targetPort: 2181   # Pod 中容器的端口
      nodePort: 32181   # NodePort 类型的端口范围为 30000-32767，可以根据需要调整
      
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-nodeport-service-1
  namespace: kafka
spec:
  type: NodePort
  selector:
    statefulset.kubernetes.io/pod-name: zookeeper-1
  ports:
    - protocol: TCP
      port: 80        # Service 暴露的端口
      targetPort: 2181   # Pod 中容器的端口
      nodePort: 32182   # NodePort 类型的端口范围为 30000-32767，可以根据需要调整
      
      
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-nodeport-service-2
  namespace: kafka
spec:
  type: NodePort
  selector:
    statefulset.kubernetes.io/pod-name: zookeeper-2
  ports:
    - protocol: TCP
      port: 80        # Service 暴露的端口
      targetPort: 2181   # Pod 中容器的端口
      nodePort: 32183   # NodePort 类型的端口范围为 30000-32767，可以根据需要调整
```

### 创建zk配置资源

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: zookeeper-config
  namespace: kafka
  labels:
    app: zookeeper
data:             #具体挂载的配置文件
  zoo.cfg: |+
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/data
    dataLogDir=/datalog
    clientPort=2181
    server.1=zookeeper-0.zookeeper-cluster.kafka:2188:3888
    server.2=zookeeper-1.zookeeper-cluster.kafka:2188:3888
    server.3=zookeeper-2.zookeeper-cluster.kafka:2188:3888
    4lw.commands.whitelist=*
```

### 创建stateful类型zk资源

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zookeeper
  namespace: kafka
spec:
  serviceName: "zookeeper-cluster"   #填写无头服务的名称
  replicas: 3
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      initContainers:
      - name: set-zk-id
        image: busybox:latest
        command: ['sh', '-c', "hostname | cut -d '-' -f 2 | awk '{print $0 + 1}' > /data/myid"]
        volumeMounts:
        - name: data
          mountPath: /data
      containers:
      - name: zookeeper
        image: zookeeper:3.8
        imagePullPolicy: Never
        resources:
          requests:
            memory: "500Mi"
            cpu: "500m"
          limits:
            memory: "1000Mi"
            cpu: "1000m"
        ports:
        - containerPort: 2181
          name: zookeeper
        - containerPort: 2188
          name: cluster1
        - containerPort: 3888
          name: cluster2
        volumeMounts:
          - name: zook-config            #挂载配置
            mountPath: /conf/zoo.cfg
            subPath: zoo.cfg
          - name: data
            mountPath: /data
        env:
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name  #metadata.name获取自己pod名称添加到变量MY_POD_NAME
      volumes:
      - name: zook-config
        configMap:                                #configMap挂载
          name: zookeeper-config
  volumeClaimTemplates:                     #这步自动创建pvc，并挂载动态pv
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteMany"]
        storageClassName: nfs
        resources:
          requests:
            storage: 10Gi
```

### 创建zk集群

```yaml
kubectl apply -f  zook.yaml
```

## 部署kafka集群

### 创建集群内Service资源

clusterIP:此类型主要用作集群内部访问；

```yaml
kind: Service
metadata:
  name: kafka-cluster  #无头服务的名称，需要通过这个获取ip，与主机的对应关系
  namespace: kafka
  labels:
    app: kafka
spec:
  ports:
    - port: 9092
      name: kafka
  clusterIP: None
  selector:
    app: kafka
```

### 创建NodePort类型资源

此类型主要用作集群外部访问；

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kafka-nodeport-service-0
  namespace: kafka
spec:
  type: NodePort
  selector:
    statefulset.kubernetes.io/pod-name: kafka0-0
  ports:
    - protocol: TCP
      port: 9092        # Service 暴露的端口
      targetPort: 9092   # Pod 中容器的端口
      nodePort: 30092   # NodePort 类型的端口范围为 30000-32767，可以根据需要调整
      name: kafka
---
apiVersion: v1
kind: Service
metadata:
  name: kafka-nodeport-service-1
  namespace: kafka
spec:
  type: NodePort
  selector:
    statefulset.kubernetes.io/pod-name: kafka1-0
  ports:
    - protocol: TCP
      port: 9092        # Service 暴露的端口
      targetPort: 9092   # Pod 中容器的端口
      nodePort: 30093   # NodePort 类型的端口范围为 30000-32767，可以根据需要调整
      name: kafka
---
apiVersion: v1
kind: Service
metadata:
  name: kafka-nodeport-service-2
  namespace: kafka
spec:
  type: NodePort
  selector:
    statefulset.kubernetes.io/pod-name: kafka2-0
  ports:
    - protocol: TCP
      port: 9092        # Service 暴露的端口
      targetPort: 9092   # Pod 中容器的端口
      nodePort: 30094   # NodePort 类型的端口范围为 30000-32767，可以根据需要调整
      name: kafka
```

### 创建StateFul类型kafka集群

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka0
  namespace: kafka
spec:
  serviceName: "kafka-cluster"   #填写无头服务的名称
  replicas: 1
  selector:
    matchLabels:
      app: kafka0
  template:
    metadata:
      labels:
        app: kafka0
    spec:
      containers:
      - name: kafka
        image: kafka:3.1.0
        imagePullPolicy: Never
        resources:
          requests:
            memory: "500Mi"
            cpu: "500m"
          limits:
            memory: "1000Mi"
            cpu: "2000m"
        ports:
        - containerPort: 9092
          name: kafka
        command:
        - sh 
        - -c 
        - "exec /app/kafka/bin/kafka-server-start.sh /app/kafka/config/server.properties --override broker.id=0 \
          --override listeners=PLAINTEXT://:9092 \
          --override advertised.listeners=PLAINTEXT://192.168.40.181:30092 \
          --override zookeeper.connect=192.168.40.181:32181,192.168.40.181:32182,192.168.40.181:32183/kafka \
          --override log.dirs=/var/lib/kafka/data \
          --override auto.create.topics.enable=true \
          --override auto.leader.rebalance.enable=true \
          --override background.threads=10 \
          --override compression.type=producer \
          --override delete.topic.enable=true \
          --override leader.imbalance.check.interval.seconds=300 \
          --override leader.imbalance.per.broker.percentage=10 \
          --override log.flush.interval.messages=9223372036854775807 \
          --override log.flush.offset.checkpoint.interval.ms=60000 \
          --override log.flush.scheduler.interval.ms=9223372036854775807 \
          --override log.retention.bytes=-1 \
          --override log.retention.hours=168 \
          --override log.roll.hours=168 \
          --override log.roll.jitter.hours=0 \
          --override log.segment.bytes=1073741824 \
          --override log.segment.delete.delay.ms=60000 \
          --override message.max.bytes=1000012 \
          --override min.insync.replicas=1 \
          --override num.io.threads=8 \
          --override num.network.threads=3 \
          --override num.recovery.threads.per.data.dir=1 \
          --override num.replica.fetchers=1 \
          --override offset.metadata.max.bytes=4096 \
          --override offsets.commit.required.acks=-1 \
          --override offsets.commit.timeout.ms=5000 \
          --override offsets.load.buffer.size=5242880 \
          --override offsets.retention.check.interval.ms=600000 \
          --override offsets.retention.minutes=1440 \
          --override offsets.topic.compression.codec=0 \
          --override offsets.topic.num.partitions=50 \
          --override offsets.topic.replication.factor=3 \
          --override offsets.topic.segment.bytes=104857600 \
          --override queued.max.requests=500 \
          --override quota.consumer.default=9223372036854775807 \
          --override quota.producer.default=9223372036854775807 \
          --override replica.fetch.min.bytes=1 \
          --override replica.fetch.wait.max.ms=500 \
          --override replica.high.watermark.checkpoint.interval.ms=5000 \
          --override replica.lag.time.max.ms=10000 \
          --override replica.socket.receive.buffer.bytes=65536 \
          --override replica.socket.timeout.ms=30000 \
          --override request.timeout.ms=30000 \
          --override socket.receive.buffer.bytes=102400 \
          --override socket.request.max.bytes=104857600 \
          --override socket.send.buffer.bytes=102400 \
          --override unclean.leader.election.enable=true \
          --override zookeeper.session.timeout.ms=6000 \
          --override zookeeper.set.acl=false \
          --override broker.id.generation.enable=true \
          --override connections.max.idle.ms=600000 \
          --override controlled.shutdown.enable=true \
          --override controlled.shutdown.max.retries=3 \
          --override controlled.shutdown.retry.backoff.ms=5000 \
          --override controller.socket.timeout.ms=30000 \
          --override default.replication.factor=1 \
          --override fetch.purgatory.purge.interval.requests=1000 \
          --override group.max.session.timeout.ms=300000 \
          --override group.min.session.timeout.ms=6000 \
          --override log.cleaner.backoff.ms=15000 \
          --override log.cleaner.dedupe.buffer.size=134217728 \
          --override log.cleaner.delete.retention.ms=86400000 \
          --override log.cleaner.enable=true \
          --override log.cleaner.io.buffer.load.factor=0.9 \
          --override log.cleaner.io.buffer.size=524288 \
          --override log.cleaner.io.max.bytes.per.second=1.7976931348623157E308 \
          --override log.cleaner.min.cleanable.ratio=0.5 \
          --override log.cleaner.min.compaction.lag.ms=0 \
          --override log.cleaner.threads=1 \
          --override log.cleanup.policy=delete \
          --override log.index.interval.bytes=4096 \
          --override log.index.size.max.bytes=10485760 \
          --override log.message.timestamp.difference.max.ms=9223372036854775807 \
          --override log.message.timestamp.type=CreateTime \
          --override log.preallocate=false \
          --override log.retention.check.interval.ms=300000 \
          --override max.connections.per.ip=2147483647 \
          --override num.partitions=1 \
          --override producer.purgatory.purge.interval.requests=1000 \
          --override replica.fetch.backoff.ms=1000 \
          --override replica.fetch.max.bytes=1048576 \
          --override replica.fetch.response.max.bytes=10485760 \
          --override reserved.broker.max.id=1000"
        volumeMounts:
          - name: data0
            mountPath: /var/lib/kafka/data
        env:
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name  #metadata.name获取自己pod名称添加到变量MY_POD_NAME
        - name: ALLOW_PLAINTEXT_LISTENER
          value: "yes"
        - name: KAFKA_HEAP_OPTS
          value : "-Xms1g -Xmx1g"
        - name: JMX_PORT
          value: "5555"
  volumeClaimTemplates:                     #这步自动创建pvc，并挂载动态pv
    - metadata:
        name: data0
      spec:
        accessModes: ["ReadWriteMany"]
        storageClassName: nfs
        resources:
          requests:
            storage: 10Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka1
  namespace: kafka
spec:
  serviceName: "kafka-cluster"   #填写无头服务的名称
  replicas: 1
  selector:
    matchLabels:
      app: kafka1
  template:
    metadata:
      labels:
        app: kafka1
    spec:
      containers:
      - name: kafka
        image: kafka:3.1.0
        imagePullPolicy: Never
        resources:
          requests:
            memory: "500Mi"
            cpu: "500m"
          limits:
            memory: "1000Mi"
            cpu: "2000m"
        ports:
        - containerPort: 9092
          name: kafka
        command:
        - sh 
        - -c 
        - "exec /app/kafka/bin/kafka-server-start.sh /app/kafka/config/server.properties --override broker.id=1 \
          --override listeners=PLAINTEXT://:9092 \
          --override advertised.listeners=PLAINTEXT://192.168.40.181:30093 \
          --override zookeeper.connect=192.168.40.181:32181,192.168.40.181:32182,192.168.40.181:32183/kafka \
          --override log.dirs=/var/lib/kafka/data \
          --override auto.create.topics.enable=true \
          --override auto.leader.rebalance.enable=true \
          --override background.threads=10 \
          --override compression.type=producer \
          --override delete.topic.enable=true \
          --override leader.imbalance.check.interval.seconds=300 \
          --override leader.imbalance.per.broker.percentage=10 \
          --override log.flush.interval.messages=9223372036854775807 \
          --override log.flush.offset.checkpoint.interval.ms=60000 \
          --override log.flush.scheduler.interval.ms=9223372036854775807 \
          --override log.retention.bytes=-1 \
          --override log.retention.hours=168 \
          --override log.roll.hours=168 \
          --override log.roll.jitter.hours=0 \
          --override log.segment.bytes=1073741824 \
          --override log.segment.delete.delay.ms=60000 \
          --override message.max.bytes=1000012 \
          --override min.insync.replicas=1 \
          --override num.io.threads=8 \
          --override num.network.threads=3 \
          --override num.recovery.threads.per.data.dir=1 \
          --override num.replica.fetchers=1 \
          --override offset.metadata.max.bytes=4096 \
          --override offsets.commit.required.acks=-1 \
          --override offsets.commit.timeout.ms=5000 \
          --override offsets.load.buffer.size=5242880 \
          --override offsets.retention.check.interval.ms=600000 \
          --override offsets.retention.minutes=1440 \
          --override offsets.topic.compression.codec=0 \
          --override offsets.topic.num.partitions=50 \
          --override offsets.topic.replication.factor=3 \
          --override offsets.topic.segment.bytes=104857600 \
          --override queued.max.requests=500 \
          --override quota.consumer.default=9223372036854775807 \
          --override quota.producer.default=9223372036854775807 \
          --override replica.fetch.min.bytes=1 \
          --override replica.fetch.wait.max.ms=500 \
          --override replica.high.watermark.checkpoint.interval.ms=5000 \
          --override replica.lag.time.max.ms=10000 \
          --override replica.socket.receive.buffer.bytes=65536 \
          --override replica.socket.timeout.ms=30000 \
          --override request.timeout.ms=30000 \
          --override socket.receive.buffer.bytes=102400 \
          --override socket.request.max.bytes=104857600 \
          --override socket.send.buffer.bytes=102400 \
          --override unclean.leader.election.enable=true \
          --override zookeeper.session.timeout.ms=6000 \
          --override zookeeper.set.acl=false \
          --override broker.id.generation.enable=true \
          --override connections.max.idle.ms=600000 \
          --override controlled.shutdown.enable=true \
          --override controlled.shutdown.max.retries=3 \
          --override controlled.shutdown.retry.backoff.ms=5000 \
          --override controller.socket.timeout.ms=30000 \
          --override default.replication.factor=1 \
          --override fetch.purgatory.purge.interval.requests=1000 \
          --override group.max.session.timeout.ms=300000 \
          --override group.min.session.timeout.ms=6000 \
          --override log.cleaner.backoff.ms=15000 \
          --override log.cleaner.dedupe.buffer.size=134217728 \
          --override log.cleaner.delete.retention.ms=86400000 \
          --override log.cleaner.enable=true \
          --override log.cleaner.io.buffer.load.factor=0.9 \
          --override log.cleaner.io.buffer.size=524288 \
          --override log.cleaner.io.max.bytes.per.second=1.7976931348623157E308 \
          --override log.cleaner.min.cleanable.ratio=0.5 \
          --override log.cleaner.min.compaction.lag.ms=0 \
          --override log.cleaner.threads=1 \
          --override log.cleanup.policy=delete \
          --override log.index.interval.bytes=4096 \
          --override log.index.size.max.bytes=10485760 \
          --override log.message.timestamp.difference.max.ms=9223372036854775807 \
          --override log.message.timestamp.type=CreateTime \
          --override log.preallocate=false \
          --override log.retention.check.interval.ms=300000 \
          --override max.connections.per.ip=2147483647 \
          --override num.partitions=1 \
          --override producer.purgatory.purge.interval.requests=1000 \
          --override replica.fetch.backoff.ms=1000 \
          --override replica.fetch.max.bytes=1048576 \
          --override replica.fetch.response.max.bytes=10485760 \
          --override reserved.broker.max.id=1000"
        volumeMounts:
          - name: data1
            mountPath: /var/lib/kafka/data
        env:
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name  #metadata.name获取自己pod名称添加到变量MY_POD_NAME
        - name: ALLOW_PLAINTEXT_LISTENER
          value: "yes"
        - name: KAFKA_HEAP_OPTS
          value : "-Xms1g -Xmx1g"
        - name: JMX_PORT
          value: "5555"
  volumeClaimTemplates:                     #这步自动创建pvc，并挂载动态pv
    - metadata:
        name: data1
      spec:
        accessModes: ["ReadWriteMany"]
        storageClassName: nfs
        resources:
          requests:
            storage: 10Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka2
  namespace: kafka
spec:
  serviceName: "kafka-cluster"   #填写无头服务的名称
  replicas: 1
  selector:
    matchLabels:
      app: kafka2
  template:
    metadata:
      labels:
        app: kafka2
    spec:
      containers:
      - name: kafka
        image: kafka:3.1.0
        imagePullPolicy: Never
        resources:
          requests:
            memory: "500Mi"
            cpu: "500m"
          limits:
            memory: "1000Mi"
            cpu: "2000m"
        ports:
        - containerPort: 9092
          name: kafka
        command:
        - sh 
        - -c 
        - "exec /app/kafka/bin/kafka-server-start.sh /app/kafka/config/server.properties --override broker.id=2 \
          --override listeners=PLAINTEXT://:9092 \
          --override advertised.listeners=PLAINTEXT://192.168.40.181:30094 \
          --override zookeeper.connect=192.168.40.181:32181,192.168.40.181:32182,192.168.40.181:32183/kafka \
          --override log.dirs=/var/lib/kafka/data \
          --override auto.create.topics.enable=true \
          --override auto.leader.rebalance.enable=true \
          --override background.threads=10 \
          --override compression.type=producer \
          --override delete.topic.enable=true \
          --override leader.imbalance.check.interval.seconds=300 \
          --override leader.imbalance.per.broker.percentage=10 \
          --override log.flush.interval.messages=9223372036854775807 \
          --override log.flush.offset.checkpoint.interval.ms=60000 \
          --override log.flush.scheduler.interval.ms=9223372036854775807 \
          --override log.retention.bytes=-1 \
          --override log.retention.hours=168 \
          --override log.roll.hours=168 \
          --override log.roll.jitter.hours=0 \
          --override log.segment.bytes=1073741824 \
          --override log.segment.delete.delay.ms=60000 \
          --override message.max.bytes=1000012 \
          --override min.insync.replicas=1 \
          --override num.io.threads=8 \
          --override num.network.threads=3 \
          --override num.recovery.threads.per.data.dir=1 \
          --override num.replica.fetchers=1 \
          --override offset.metadata.max.bytes=4096 \
          --override offsets.commit.required.acks=-1 \
          --override offsets.commit.timeout.ms=5000 \
          --override offsets.load.buffer.size=5242880 \
          --override offsets.retention.check.interval.ms=600000 \
          --override offsets.retention.minutes=1440 \
          --override offsets.topic.compression.codec=0 \
          --override offsets.topic.num.partitions=50 \
          --override offsets.topic.replication.factor=3 \
          --override offsets.topic.segment.bytes=104857600 \
          --override queued.max.requests=500 \
          --override quota.consumer.default=9223372036854775807 \
          --override quota.producer.default=9223372036854775807 \
          --override replica.fetch.min.bytes=1 \
          --override replica.fetch.wait.max.ms=500 \
          --override replica.high.watermark.checkpoint.interval.ms=5000 \
          --override replica.lag.time.max.ms=10000 \
          --override replica.socket.receive.buffer.bytes=65536 \
          --override replica.socket.timeout.ms=30000 \
          --override request.timeout.ms=30000 \
          --override socket.receive.buffer.bytes=102400 \
          --override socket.request.max.bytes=104857600 \
          --override socket.send.buffer.bytes=102400 \
          --override unclean.leader.election.enable=true \
          --override zookeeper.session.timeout.ms=6000 \
          --override zookeeper.set.acl=false \
          --override broker.id.generation.enable=true \
          --override connections.max.idle.ms=600000 \
          --override controlled.shutdown.enable=true \
          --override controlled.shutdown.max.retries=3 \
          --override controlled.shutdown.retry.backoff.ms=5000 \
          --override controller.socket.timeout.ms=30000 \
          --override default.replication.factor=1 \
          --override fetch.purgatory.purge.interval.requests=1000 \
          --override group.max.session.timeout.ms=300000 \
          --override group.min.session.timeout.ms=6000 \
          --override log.cleaner.backoff.ms=15000 \
          --override log.cleaner.dedupe.buffer.size=134217728 \
          --override log.cleaner.delete.retention.ms=86400000 \
          --override log.cleaner.enable=true \
          --override log.cleaner.io.buffer.load.factor=0.9 \
          --override log.cleaner.io.buffer.size=524288 \
          --override log.cleaner.io.max.bytes.per.second=1.7976931348623157E308 \
          --override log.cleaner.min.cleanable.ratio=0.5 \
          --override log.cleaner.min.compaction.lag.ms=0 \
          --override log.cleaner.threads=1 \
          --override log.cleanup.policy=delete \
          --override log.index.interval.bytes=4096 \
          --override log.index.size.max.bytes=10485760 \
          --override log.message.timestamp.difference.max.ms=9223372036854775807 \
          --override log.message.timestamp.type=CreateTime \
          --override log.preallocate=false \
          --override log.retention.check.interval.ms=300000 \
          --override max.connections.per.ip=2147483647 \
          --override num.partitions=1 \
          --override producer.purgatory.purge.interval.requests=1000 \
          --override replica.fetch.backoff.ms=1000 \
          --override replica.fetch.max.bytes=1048576 \
          --override replica.fetch.response.max.bytes=10485760 \
          --override reserved.broker.max.id=1000"
        volumeMounts:
          - name: data2
            mountPath: /var/lib/kafka/data
        env:
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name  #metadata.name获取自己pod名称添加到变量MY_POD_NAME
        - name: ALLOW_PLAINTEXT_LISTENER
          value: "yes"
        - name: KAFKA_HEAP_OPTS
          value : "-Xms1g -Xmx1g"
        - name: JMX_PORT
          value: "5555"
  volumeClaimTemplates:                     #这步自动创建pvc，并挂载动态pv
    - metadata:
        name: data2
      spec:
        accessModes: ["ReadWriteMany"]
        storageClassName: nfs
        resources:
          requests:
            storage: 10Gi
```



### 创建kafka集群

```yaml
kubectl apply -f  kafka.yaml 
```

