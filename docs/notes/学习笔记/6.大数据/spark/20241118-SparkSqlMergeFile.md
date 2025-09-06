---
title: SparkSqlMergeFile功能及使用说明
icon: streamline-freehand-color:notes-add
order: 3
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
permalink: /learning-notes/b0xpxnfd/
---


# Spark sql merge file功能及使⽤说明

- Spark执⾏insert相关sql时，只能通过shuffle参数spark.sql.shuffle.partitions或者spark.sql.adaptive.maxNumPostShufflePartitions（开启AE：spark.sql.adaptive.enabled=true）来控制最后⽣成的⽂件个数。

- 如果想要调整最后⽣成的⽂件数量，就必须调整shuffle并⾏度，sql中所有stages的并⾏度都受该参数影响，这样可能会对任务的运⾏时⻓造成较⼤的影响，因此在Spark中增加合并⽂件的功能。
- 该功能⽐较适合的场景：insert相关的sql，sql处理的数据量⽐较⼤（如⼤于1T）且最后⽣成的数据量却较⼩（如⼩于100G）的情况

## 按⽂件数合并

- spark.merge.files.enabled=true
- spark.merge.files.number=512

说明

1. 该功能会在原来job的最后⼀个stage后⾯增加1个stage来控制最后⽣成的⽂件数量spark.merge.files.number（默认512），性能上会有⼀点影响，但基本可以忽略。
2. 该功能能够精确的控制⽣成的⽂件数量。
3. 对于动态分区，每个分区⽣成spark.merge.files.number个⽂件。

## 按⽂件大小合并

- spark.sql.adaptive.enabled=true
- spark.merge.files.byBytes.enabled=true
- spark.merge.files.byBytes.repartitionNumber=1000
- spark.merge.files.byBytes.fileBytes=134217728
- spark.merge.files.byBytes.compressionRatio=3

说明：

1. 按⽂件⼤⼩合并必须开启AE: spark.sql.adaptive.enabled=true
2. 开启按⽂件⼤⼩合并功能(spark.merge.files.byBytes.enabled=true)后会⾃动禁⽤按⽂件数合并功能，即spark.merge.files.enabled参数⽆效
3.  按⽂件⼤⼩合并功能会在原来job的最后⼀个stage后⾯增加2个stage，第⼀个stage的并⾏度
    由参数spark.merge.files.byBytes.repartitionNumber控制（默认1000），然后根据第⼀个stage
    shuffle的数据量来预估最后⽣成到hdfs上的⽂件数据量⼤⼩，并通过预估的⽂件数据量⼤⼩计算第
    ⼆个stage的并⾏度，即最后⽣成的⽂件个数。第⼆个stage的并⾏度根据以下公式计算得到：
1. totalBytes = 第⼀个stage shuffle write的总数据量（单位Byte）；
2. fileBytes = spark.merge.files.byBytes.fileBytes设置的值（默认128M）， 即⽤⼾期望⽣成的⽂
   件⼤⼩；
3. compressionRatio = spark.merge.files.byBytes.compressionRatio设置的值（默认3），该参
   数表⽰shuffle数据量与最后⽣成到hdfs上的数据量的压缩⽐，因为shuffle ⽂件和⽣成到hdfs上的⽂
   件存储格式和压缩格式都不相同，通过shuffle数据量预估hdfs⽂件数据量时会有⼀定的误差，因此
   设置该参数来调节；
4. (totalBytes / fileBytes / compressionRatio).toInt + 1

4. ⽤⼾在使⽤该功能时，最好先实测下，然后根据实际⽣成的⽂件⼤⼩调节compressionRatio到
   ⼀个合适的值：
  1. 假如根据compressionRatio设置的值实际⽣成的⽂件⼤⼩记为fact_size，期望的⼤⼩
     （spark.merge.files.byBytes.fileBytes设置的值）记为expect_size，则需要调整：
  2. 新的compressionRatio= (fact_size * 原来的compressionRatio) / expect_size
5. 该功能只能控制⽣成的⽂件⼤⼩尽量接近spark.merge.files.byBytes.fileBytes，且有⼀定的性能损耗，需根据实测情况选择使⽤。