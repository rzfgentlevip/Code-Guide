---
title: Spark优化参数列表
icon: streamline-freehand-color:notes-add
order: 6
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
permalink: /learning-notes/phqpdqch/
---


# Spark优化参数列表

## 配置项

| 参数                                                         | 描述                                                         | 源码默认值                           | TQS默认值（adhoc/etl）               |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------ | ------------------------------------ |
| spark.executor.instances                                     | 静态资源下：executor数                                       | 2                                    | TQS使用动态资源                      |
| spark.executor.cores                                         | 每个executor和CPU数                                          | 4                                    | 3/4                                  |
| spark.dynamicAllocation.enabled                              | 动态资源开关                                                 | false                                | true/true                            |
| spark.dynamicAllocation.maxExecutors                         | 动态资源下：executor的最大个数                               | 500                                  | 300/900                              |
| spark.executor.memory                                        | 每个executor的内存大小                                       | 8g                                   | 5g/8g                                |
| spark.memory.fraction                                        | executor用于计算的内存比例，剩余部分用于存储元数据和运行信息。对于executor内存开的较大的任务，可以适当提高这个值，让更多内存参与计算，但会增加OOM风险 | 0.6                                  | 0.7/0.6                              |
| spark.executor.memoryOverhead/ spark.yarn.executor.memoryOverhead | 每个executor的堆外内存大小，堆外内存主要用于数据IO，对于报堆外OOM的任务要适当调大，单位Mb，与之配合要调大executor JVM参数，例如： set spark.executor.memoryOverhead=3072 set spark.executor.extraJavaOptions=-XX:MaxDirectMemorySize=2560m | 6144                                 | 3072/6144                            |
| spark.sql.adaptive.enabled                                   | Adaptive execution开关，包含自动调整并行度，解决数据倾斜等优化，详见https://github.com/Intel-bigdata/spark-adaptive | true                                 | true/true                            |
| spark.sql.adaptive.minNumPostShufflePartitions               | AE相关，动态最小的并行度                                     | 1                                    | 1/1                                  |
| spark.sql.adaptive.maxNumPostShufflePartitions               | AE相关，动态最大的并行度，对于shuffle量大的任务适当增大可以减少每个task的数据量，如1024 | 1000                                 | 1000/500                             |
| spark.sql.adaptive.shuffle.targetPostShuffleInputSize        | AE相关，调整并发度时预期达到的单task数据量                   |                                      |                                      |
| spark.sql.adaptive.shuffle.targetPostShuffleRowCount         | AE相关，调整并发度时预期达到的单task数据行数                 |                                      |                                      |
| spark.sql.adaptive.join.enabled                              | AE相关，开启后能够根据数据量自动判断能否将sortMergeJoin转换成broadcast join | true                                 | true/true                            |
| spark.sql.adaptiveBroadcastJoinThreshold                     | AE相关，spark.sql.adaptive.join.enabled设置为true后会判断join的数据量是否小于该参数值，如果小于则能将sortMergeJoin转换成broadcast join | spark.sql.autoBroadcastJoinThreshold | spark.sql.autoBroadcastJoinThreshold |
| spark.sql.adaptive.skewedJoin.enabled                        | AE相关，开启后能够自动处理join时的数据倾斜，对于数据量明显高于中位数的task拆分成多个小task | false                                | true/false                           |
| spark.sql.adaptive.skewedPartitionFactor                     | AE相关，数据倾斜判定标准，当同一stage的某个task数据量超过中位数的N倍，将会判定为数据倾斜 | 5                                    | 3/5                                  |
| spark.sql.adaptive.skewedPartitionMaxSplits                  | AE相关，被判定为数据倾斜后最多会被拆分成的份数               | 5                                    | 6/5                                  |
| spark.shuffle.accurateBlockThreshold                         | AE相关，数据倾斜判定基于shuffle数据量统计，如果统计所有的block数据，消耗内存较大，因此设有阈值，当shuffle的单个数据块超过大小和行数阈值时，才会进入统计，这个参数即大小阈值 | 100*1024*1024（100MB）               | 4000000/100*1024*1024                |
| spark.shuffle.accurateBlockRecordThreshold                   | AE相关，同上，行数阈值，如果设置了上面的数据倾斜处理开关，仍然倾斜，可能是因为这几个参数设得偏大，适当缩小 | 2 * 1024 * 1024                      | 500000/2 * 1024 * 1024               |
| spark.sql.files.maxPartitionBytes                            | 默认一个task处理的数据大小，如果给的太小会造成最终任务task太多，太大会是输入环节计算较慢 | 1073741824                           | 268435456/1073741824                 |
| spark.datasource.splits.max                                  | 第一阶段扫表的task上限，用户可以设置在50000，task太多会造成下阶段oom。如果task不够50000，则不做任何事情，否则限制task总数在50000。 （默认不限制，task数等于总的文件大小除以spark.sql.files.maxPartitionBytes） | 0  （不限制task上限）                | 5000/50000                           |
| spark.vcore.boost.ratio                                      | vcore,虚拟核数，设置大于1的数可以使一个核分配多个task，对于简单sql可以提升CPU利用率，对于复杂任务有OOM风险 | 1                                    | 2/（ETL国际化是2，国内是1）          |
| spark.shuffle.hdfs.enabled（长任务推荐）                     | HDFS based Spark Shuffle开关，可以提高任务容错性。遇到org.apache.spark.shuffle.FetchFailedException报错需设置 | false                                | false/false                          |
| spark.shuffle.hdfs.rootDir （推荐）                          | 根据机房不同设置不同的值 怀来：hdfs://haruna/spark_hl/shuffle/hdfs_dancenn 廊坊：hdfs://haruna/spark_lf/shuffle/hdfs_dancenn | 默认已配置，只需开启上一个参数即可   |                                      |
| set spark.shuffle.io.maxRetries=1; set spark.shuffle.io.retryWait=0s; | 一般在开启hdfs shuffle后还可以开启这两个参数，避免不必要的重试和等待 |                                      |                                      |
| spark.shuffle.hdfs.replication                               | Hdfs Shuffle 文件的副本数                                    | 2                                    |                                      |
| spark.sql.crossJoin.enabled                                  | 对于会产生笛卡尔积的sql，默认配置是限制不能跑的，在hive里可以配置set *hive*.mapred.mode=nonstrict跳过限制，相对应的在spark里可以配置set spark.sql.crossJoin.enabled=true起到同样的效果。 | false                                | true/true                            |
| spark.sql.broadcastTimeout                                   | broadcast joins时，广播数据最长等待时间，网络不稳定时，容易出现超时造成任务失败，可适当增大此参数。 | 300（单位：s）                       | 3000/3000                            |
| spark.sql.autoBroadcastJoinThreshold                         | 表能够使用broadcast join的最大阈值                           | 10MB                                 | 20MB                                 |
| spark.network.timeout                                        | 网络连接超时参数                                             | 120s                                 | 120s/120s                            |
| spark.maxRemoteBlockSizeFetchToMem                           | reduce端获取的remote block存放到内存的阈值，超过该阈值后数据会写磁盘，当出现数据量比较大的block时，建议调小该参数（比如512MB）。 | Long.*MaxValue*                      | 536870912/536870912                  |
| spark.reducer.maxSizeInFlight                                | 控制从一个worker拉数据缓存的最大值                           | 48m                                  | 48m/48m                              |
| spark.merge.files.enabled                                    | 合并输出文件，如果insert结果的输出文件数很多，希望合并，可以设为true，会多增加一个repartition stage合并文件，repartition的分区数由spark.merge.files.number控制 | false                                |                                      |
| spark.merge.files.number                                     | 控制合并输出文件的输出数量                                   | 512                                  |                                      |
| spark.speculation                                            | 推测执行开关。如果是原生任务很有可能没开这个参数，会出现个别task拖慢整个任务，可以开启这个参数。 | true                                 | true                                 |
| spark.speculation.multiplier                                 | 开启推测执行的时间倍数阈值：当某个任务运行时间/中位数时间大于该值，触发推测执行。对于因为推测执行而浪费较多资源的任务可以适当调高这个参数。 | 1.5                                  |                                      |
| spark.speculation.quantile                                   | 同一个stage中的task超过这个参数比例的task完成后，才会开启推测执行。对于因为推测执行而浪费较多资源的任务可以适当调高这个参数。 | 0.75                                 | 0.98/0.98                            |
| spark.default.parallelism                                    | Spark Core默认并发度，原生spark程序并发度设置                | 200                                  | 200/200                              |
| spark.sql.shuffle.partitions                                 | Spark SQL默认并发度，AE开启后被spark.sql.adaptive.maxNumPostShufflePartitions取代 | 200                                  | 200/200                              |
| spark.sql.sources.bucketing.enabled                          | 分桶表相关，当设置为false，会将分桶表当作普通表来处理。做为普通表会忽略分桶特性，部分情况性能会下降。但如果分桶表没有被正确生成（即表定义是分桶表，但数据未按分桶表生成）会报错RuntimeException: Invalid bucket file，避免这个错误，要将这个参数设为false | true                                 | false/false                          |
| spark.hadoop.hive.metastore.client.socket.timeout            | 连接hivemetastore的超时时间，对于出现超时导致失败，可以暂时扩大这个时间保证任务完成，但长远方案应该是优化相关逻辑，减少相应操作的耗时。 | 200                                  | 200                                  |
| spark.sql.partition.rownum.collect.enable                    | 统计生成固定分区表行数                                       | false                                | true                                 |
| spark.sql.dynamic.partition.rownum.collect.enable            | 统计生成动态分区表行数                                       | false                                | true                                 |
| spark.yarn.am.waitTime                                       | SparkContext启动等待时间，有时任务会出现如下报错，可能是偶然的系统波动导致的超时，可以扩大改参数值解决 ![img](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/wps2.jpg) | 100s                                 | 200s                                 |
| spark.sql.parquet.enableVectorizedReader                     | 开启parquet向量化读                                          | true                                 |                                      |
| spark.sql.orc.enableVectorizedReader                         | 开启orc向量化读                                              | true                                 |                                      |



## 参数模板

```text
形成自己的常用参数模版
例如：（仅供参考，因业务不同而有差异，需要在实践中总结）
小任务：
set spark.executor.memory=6g;
set spark.dynamicAllocation.maxExecutors = 300;
set spark.speculation.multiplier = 2.5;
set spark.speculation.quantile=0.95;
set spark.vcore.boost.ratio=2;
大任务
--spark
set spark.driver.memory=40g;
set spark.executor.memory=10g;
set spark.dynamicAllocation.maxExecutors = 1000;
set spark.shuffle.hdfs.enabled=true;
set spark.shuffle.io.maxRetries=1;
set spark.shuffle.io.retryWait=0s;
set spark.sql.adaptive.maxNumPostShufflePartitions = 4000;
set spark.vcore.boost.ratio=1;
```

