---
title: Spark任务参数配置模版
icon: streamline-freehand-color:notes-add
order: 5
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
permalink: /learning-notes/1ri1bxt3/
---


# spark任务参数配置模版

## 调参原则

- 禁止通过增加执行资源的方式来达成任务的目的
    - 比如OOM首先考虑的应该是降低每个task数目量，而不是增加内存

- 大多数参数调整无大意义，线上场景也比较复杂，没必要精细化调整每个参数到最优，保持一类任务的参数大致统一才是最佳方案
    - 不出现严重的执行缓慢/task失败/运行不稳定即可

- 尽可能控制任务的资源使用


## **参数推荐**

参数划分标准：根据收集到的资源元数据判定

重要参数详解

1. set spark.sql.adaptive.maxNumPostShufflePartitions =

    2. AE shuffle的最大并行度（也是初始并行度）

    3. 该值过高会引起shuffle过程颗粒度过小而引起资源浪费

    4. 建议值：根据业务场景采用1000 - 5000即可

    5. Note：高于5000并行度的场景可以考虑关闭AE采用spark原生shuffle

6. set spark.sql.adaptive.shuffle.targetPostShuffleInputSize = 268435456/536870912;
    1. shuffle read的期望输入最大值，根据计算逻辑决定，复杂逻辑（有需要大量资源的udf使用；连续按相同key join）采用更小的256M，其他采用512M即可
    2. 不要设置该值过高，该值过高会引起spill
    3. 若shuffle read task的spill过高致使task缓慢，降低该值

7.  set spark.sql.files.maxPartitionBytes = 268435456/536870912/
1. 可认为是input read的最大期望值，调整逻辑同上
2. 若input task的spill过高致使task缓慢，降低该值

8. set spark.dynamicAllocation.maxExecutors =
    1. 参与计算的最大executor个数，根据任务量和任务重要程度确定

9. set spark.memory.fraction=;
    1. spark用于计算和缓存rdd的内存比例，该值默认0.6，TQS置为0.7，不建议调整该值

10. set spark.shuffle.hdfs.enabled=true;
    1. 建议在大任务或者重要任务中开启，避免shuffle fetch fail

11. set spark.shuffle.io.maxRetries=1;（该参数禁止设置为0，会导致严重数据错误）
12. set spark.shuffle.io.retryWait=0s;
13. set spark.network.timeout=120s;
14. 启用hdfs shuffle时配合使用，获取local shuffle文件失败立刻路由到hdfs
15. set spark.speculation.multiplier=2.5;
16. set spark.speculation.quantile=0.9;
17. spark推测执行参数，不修改该参数会导致大量speculation task kill，浪费资源
18. set spark.merge.files.enabled=true;
19. set spark.merge.files.number=期望文件个数

**广播设置**

1. adaptive execution提供的广播，执行过程中动态调整执行计划，必须开启spark.sql.adaptive.enabled才会有效。

2. spark.sql.adaptive.join.enabled=false

3. 是否启用adatptive execution自动广播
4. spark.sql.adaptiveBroadcastJoinThreshold=20971520
5. AE自动广播的阈值，单位byte，对应task的map output size
6. spark非adatptive的广播方式
7. spark.sql.autoBroadcastJoinThreshold=20971520
8. 优先读取hive meta信息；FileScan模式下可以扫描文件size，单位byte（注意是整表size，而不是仅仅所需列的size；text格式表只能走hive serde所以此处不支持分区级别广播，只能通过AE广播）

## 小型任务：

目前测试：在不手动添加任何参数、平均时长在30min以内、单个shuffle量在500G以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

```text
基础资源
set spark.driver.memory=15g;
set spark.driver.memoryOverhead=4096;
set spark.driver.memoryOverhead=4096;
set spark.executor.memory=5G;
set spark.executor.memoryOverhead=1024;
set spark.executor.cores=2;
set spark.vcore.boost.ratio=2;
--动态executor申请
set spark.dynamicAllocation.minExecutors=10;
set spark.dynamicAllocation.maxExecutors=300;
--ae shuffle partition并⾏度
set spark.sql.adaptive.minNumPostShufflePartitions=10;
set spark.sql.adaptive.maxNumPostShufflePartitions=1000;

--268435456;
set spark.sql.adaptive.shuffle.targetPostShuffleInputSize=536870912;
--开启parquet切分
set spark.sql.parquet.adaptiveFileSplit=true;
--初始task调节，合并⼩⽂件
set spark.sql.files.maxPartitionBytes=536870912;

```

## 中型任务：

目前测试：在不手动添加任何参数、平均时长在90min以内、单个shuffle量在2T以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

```text
--
基础资源
 
set spark.driver.memory=25g;
set spark.driver.cores=4;
set spark.driver.memoryOverhead=5120;
set spark.executor.memory=10G;
set spark.executor.memoryOverhead=4096;
set spark.executor.cores=3;
set spark.vcore.boost.ratio=1;
--动态executor申请
 
set spark.dynamicAllocation.minExecutors=10;
set spark.dynamicAllocation.maxExecutors=600;
--ae
set spark.sql.adaptive.minNumPostShufflePartitions=10;
set spark.sql.adaptive.maxNumPostShufflePartitions=1000;
set spark.sql.adaptive.shuffle.targetPostShuffleInputSize= 536870912;
--开启parquet切分，初始task调节，合并⼩⽂件
set spark.sql.parquet.adaptiveFileSplit=true;
set spark.sql.files.maxPartitionBytes=536870912;
--推测
set spark.speculation.multiplier=2.5;
set spark.speculation.quantile=0.8;
--shuffle 落地hdfs 
set spark.shuffle.hdfs.enabled=true;
set spark.shuffle.io.maxRetries=1;
set spark.shuffle.io.retryWait=0s;
```

## 大型任务：

目前测试：在不手动添加任何参数、平均时长在120min以内、单个shuffle量在10T以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

```text
--
基础资源
set spark.driver.memory=25g;
set spark.driver.cores=4;
set spark.driver.memoryOverhead=5120;
set spark.executor.memory=15G;
set spark.executor.memoryOverhead=3072;
set spark.executor.cores=3;
set spark.vcore.boost.ratio=1;
--动态executor申请
set spark.dynamicAllocation.minExecutors=10;
set spark.dynamicAllocation.maxExecutors=900;
--ae
set spark.sql.adaptive.minNumPostShufflePartitions=10;
set spark.sql.adaptive.maxNumPostShufflePartitions=3000;
set spark.sql.adaptive.shuffle.targetPostShuffleInputSize= 536870912;
--shuffle 落地hdfs 
set spark.shuffle.hdfs.enabled=true;
set spark.shuffle.io.maxRetries=1;
set spark.shuffle.io.retryWait=0s;
--开启parquet切分，合并⼩⽂件
set spark.sql.parquet.adaptiveFileSplit=true; 
set spark.sql.files.maxPartitionBytes=536870912;
--推测
set spark.speculation.multiplier=2.5;
set spark.speculation.quantile=0.9;
```

## 超大型任务：

目前测试：在不手动添加任何参数、平均时长大于120min、单个shuffle量在10T以上的任务可以使用该模版，但实际任务情况还需跟踪观察。

```tex
--
基础资源
set spark.driver.memory=30g;
set spark.driver.cores=4;
set spark.driver.memoryOverhead=5120;
set spark.executor.memory=20G;
set spark.executor.memoryOverhead= 5120;
set spark.executor.cores=5;
set spark.vcore.boost.ratio=1;
--动态executor申请
set spark.dynamicAllocation.minExecutors=10;
set spark.dynamicAllocation.maxExecutors=1500;
--ae
set spark.sql.adaptive.minNumPostShufflePartitions=10;
set spark.sql.adaptive.maxNumPostShufflePartitions=7000;
set spark.sql.adaptive.shuffle.targetPostShuffleInputSize= 536870912;
--开启parquet切分,合并小文件
set spark.sql.parquet.adaptiveFileSplit=true;
set spark.sql.files.maxPartitionBytes=536870912;
-- shuffle 落地hdfs 
set spark.shuffle.hdfs.enabled=true;
set spark.shuffle.io.maxRetries=1;
set spark.shuffle.io.retryWait=0s;
--推测
set spark.speculation.multiplier=2.5;
set spark.speculation.quantile=0.9;
```

## 备用

```text
--
预留
 
set spark.yarn.priority = 9;
--ae hash join
set spark.sql.adaptive.hashJoin.enabled=true;
set spark.sql.adaptiveHashJoinThreshold=52428800;
--输出文件合并                
set spark.merge.files.byBytes.enabled=true;
set spark.merge.files.byBytes.repartitionNumber=100;
set spark.merge.files.byBytes.fileBytes=134217728;
set spark.merge.files.byBytes.compressionRatio=3;
--skew_join 解析绕过tqs 
set tqs.analysis.skip.hint=true;
--初始task上限
 
set spark.sql.files.openCostInBytes=4194304;
set spark.datasource.splits.max=20000;
--broadcast时间
set spark.sql.broadcastTimeout = 3600;
--（防止get json报错）
set spark.sql.mergeGetMapValue.enabled=true;
--ae 倾斜处理HandlingSkewedJoin  OptimizeSkewedJoin 
set spark.sql.adaptive.allowBroadcastExchange.enabled=true;
set spark.sql.adaptive.hashJoin.enabled=false;
set spark.sql.adaptive.skewedPartitionFactor=3;
set spark.sql.adaptive.skewedPartitionMaxSplits=20;
set spark.sql.adaptive.skewedJoin.enabled=true;
set spark.sql.adaptive.skewedJoinWithAgg.enabled=true;
set spark.sql.adaptive.multipleSkewedJoin.enabled=true;
set spark.shuffle.highlyCompressedMapStatusThreshold=20000;
--并发读文件
set spark.sql.concurrentFileScan.enabled=true;
--filter 按照比例读文件
 
set spark.sql.files.tableSizeFactor={table_name}:{filter 
35 set spark.sql.files.tableSizeFactor=dm_content.tcs_task_dict:10;
--AM failed  时长
set spark.yarn.am.waitTime=200s;
--shuffle service 超时设置
set spark.shuffle.registration.timeout=12000;
set spark.shuffle.registration.maxAttempts=5;
--parquet index  
set spark.sql.parquet.pushdown.inFilterThreshold=30; 
--设置engine 
set tqs.query.engine.type=sparkcli;
--hive metastore 超时
spark.hadoop.hive.metastore.client.socket.timeout=600
--manta备用
spark.sql.adaptive.maxNumPostShufflePartitions 5000
spark.executor.memoryOverhead 8000
spark.sql.adaptive.shuffle.targetPostShuffleInputSize 536870912
```

## 默认参数（一些较为重要的参数）

贴这个是为了防止冗余写。

**基础资源相关**

| //基础资源                                  |      |      |
| ------------------------------------------- | ---- | ---- |
| spark.driver.memory                         | 12G  |      |
| spark.driver.memoryOverhead                 | 6144 |      |
| spark.executor.memory                       | 8G   |      |
| spark.executor.memoryOverhead               | 6144 |      |
| spark.executor.cores                        | 4    |      |
| //动态资源                                  |      |      |
| spark.dynamicAllocation.enabled             | true |      |
| spark.dynamicAllocation.executorIdleTimeout | 120s |      |
| spark.dynamicAllocation.initialExecutors    | 5    |      |
| spark.dynamicAllocation.maxExecutors        | 900  |      |
| spark.dynamicAllocation.minExecutors        | 5    |      |

**jvm相关**

| spark.executor.extraJavaOptions                  | -XX:+UseG1GC -XX:G1HeapRegionSize=4m -XX: UseGCOverheadLimit -verbose:gc -XX:+PrintAdaptiveSizePolicy  XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -Xss4m  XX:MaxDirectMemorySize=4096m |
| ------------------------------------------------ | ------------------------------------------------------------ |
| // shuffle相关，spark自带                        |                                                              |
| spark.shuffle.consolidateFiles                   | true                                                         |
| spark.shuffle.hdfs.rootDir                       | hdfs://haruna/spark_topi/shuffle/hdfs_dancen                 |
| spark.shuffle.highlyCompressedMapStatusThreshold | 2000                                                         |
| spark.shuffle.service.enabled                    | true                                                         |
| spark.shuffle.statistics.verbose                 | true                                                         |
| spark.speculation                                | true                                                         |

**Spark ae相关**

| spark.sql.adaptive.enabled                            | true            |
| ----------------------------------------------------- | --------------- |
| spark.sql.adaptive.join.enabled                       | true            |
| spark.sql.adaptive.maxNumPostShufflePartitions        | 500             |
| spark.sql.adaptive.minNumPostShufflePartitions        | 1               |
| spark.sql.adaptive.shuffle.targetPostShuffleInputSize | 67108864 //64MB |
| spark.sql.adaptive.shuffle.targetPostShuffleRowCount  | 20000000        |
| spark.sql.adaptive.skewedJoin.enabled                 | false           |
| spark.sql.adaptive.skewedPartitionFactor              | 10              |
| spark.sql.adaptive.skewedPartitionRowCountThreshold   | 10000000        |
| spark.sql.adaptive.skewedPartitionSizeThreshold       | 67108864        |
| spark.sql.adaptiveBroadcastJoinThreshold              |                 |
| spark.sql.autoBroadcastJoinThreshold                  | 20971520        |
| //初始文件合并                                        |                 |
| spark.sql.files.maxPartitionBytes                     | 1073741824//1GB |
| spark.sql.files.openCostInBytes                       | 16777216        |
| // 其他                                               |                 |
| spark.sql.hive.caseSensitiveInferenceMode N           | NEVER_INFER     |
| spark.sqlhive.convertMetastoreOrc                     | true            |
| .spark.sqlhiveconvertMetastoreParquet                 | true            |
| .. sparkslparquetadaptiveFileSlit                     | false           |
| .q..pspark.sqlparquetcompressioncodec                 | gzip            |
| ...sparksqlparquetfilterPushdown                      | true            |
| ... spark.sql.parquet.pushdown.inFilterThreshold      | 20              |
| spark.sql.skip.adjust.partitioned.file                | true            |
| spark.sql.sources.bucketing.enabled                   | false           |
| //spark 容错以及 yarn 优先级                          | 8               |
| spark.task.maxFailures                                | 1               |
| spark.yarn.maxAppAttempts                             | 1               |
| spark.yarn.priority                                   |                 |