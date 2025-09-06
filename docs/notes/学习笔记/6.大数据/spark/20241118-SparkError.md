---
title: Spark容错
icon: streamline-freehand-color:notes-add
order: 2
author: bugcode
date: 2024-11-16T00:00:00.000Z
category:
  - 大数据
  - SPARK
tag:
  - spark
  - 大数据
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 15:13:45
permalink: /learning-notes/qnzjxmht/
---


# Spark 容错

## Spark容错的层次

### 数据源可重放

### RDD容错

- RDD lineage
- StorageLevel
- RDD Checkpoint

### Task容错

上游数据必须得仍然存在，因抢占⽽kill的Task不消耗重试次数
重试
map，单split存在
推测执⾏，reduce shuffle block必须都存在

## Stage容错

上游数据必须得仍然存在，参考Task容错，差异在所有Task都重跑，重试

## Job容错

重试

## Application容错

重试
metadata Checkpoint ，driver从metadata Checkpoint中恢复

## 容错⼩结

**容错有顺序**

- RDD容错
- Task容错
- Stage容错
- Application容错

**容错分类**

- 数据容错和执⾏容错，数据容错给执⾏容错准备数据，缩短容错的执⾏路径

**容错的数据分类**

- 数据源、RDD、Shuffle Block(仅磁盘)
- 因抢占⽽引起的容错，不消耗重试次数

## Spark Sql性能优化思路

### shuffle分区数：spark.sql.shuffle.partitions

默认为200。 partition 数量太⼤可能会需要处理⼤量⼩的task，导致增加task调度开销以及资源调度开销。另 外，如果该Stage最后要输出存储，造成很多⼩的IO操作，还会造成在HDFS上存储⼤量的⼩⽂ 件； partition 数量太⼩可能会导致每个task处理⼤量的数据，处理效率低下，⽆法有效利⽤集群资源的 并⾏处理能⼒，甚⾄导致OOM的问题。

此参数不推荐使⽤，不太好掌握，就像⼿动档汽⻋⼀样不受待⻅。



## 开启AE：spark.sql.adaptive.enabled=true

当⼀个stage的map任务在runtime完成时，我们利⽤map输出⼤⼩信息，对并⾏度、join策略 和倾斜处理进⾏相应的调整。

## 动态调整reduce个数的partition⼤⼩依据

spark.sql.adaptive.maxNumPostShufflePartitions 如设置64MB，则reduce阶段每个task最少处理64MB的数据。默认值为64MB。

## 动态调整reduce个数的partition条数依据

spark.sql.adaptive.shuffle.targetPostShuffleRowCount 如设置20000000，则reduce阶段每个task最少处理20000000条的数据。默认值为20000000。

## reduce个数区间最⼩值

spark.sql.adaptive.minNumPostShufflePartitions

## reduce个数区间最⼤值

spark.sql.adaptive.maxNumPostShufflePartitions

## 开启JOIN策略优化

spark.sql.adaptive.join.enabled=true

## ⾃动处理倾斜的开关

spark.sql.adaptive.skewedJoin.enabled=true

⾃动处理倾斜的task个数上限

spark.sql.adaptive.skewedPartitionMaxSplits

倾斜的partition条数不能⼩于该值

spark.sql.adaptive.skewedPartitionRowCountThreshold

倾斜的partition⼤⼩不能⼩于该值 spark.sql.adaptive.skewedPartitionSizeThreshold，默认值为 64MB

shuffle超时发消息给shuffleserver spark.shuffle.registration.timeout shuffle

超时后重试 spark.shuffle.registration.maxAttempts=5

