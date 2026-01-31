---
title: 4、JVM调优参数总结
icon: catppuccin:java-exception
order: 4
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 11:14:44
permalink: /compre-guide/interview/jvm/JVM调优参数总结/
---


# JVM 调优参数全面总结

## 一、堆内存相关参数

### 1.1 基础堆设置

```java
# 设置初始堆大小（默认物理内存1/64）
-Xms512m  # 或 -XX:InitialHeapSize=536870912

# 设置最大堆大小（默认物理内存1/4）
-Xmx2g    # 或 -XX:MaxHeapSize=2147483648

# 设置年轻代大小（对分代收集器）
-Xmn1g    # 或 -XX:NewSize=1073741824 -XX:MaxNewSize=1073741824

# 设置年轻代中 Eden 和 Survivor 的比例
-XX:SurvivorRatio=8  # Eden:S0:S1 = 8:1:1

# 设置年轻代中 Survivor 区的目标使用率
-XX:TargetSurvivorRatio=50  # 默认50%
```

### 1.2 老年代相关

```java
# 设置年轻代与老年代的比例（1:2）
-XX:NewRatio=2  # 老年代:年轻代 = 2:1

# 设置晋升到老年代的年龄阈值
-XX:MaxTenuringThreshold=15  # 默认15，CMS下默认6

# 大对象直接进入老年代的阈值（仅Serial/ParNew）
-XX:PretenureSizeThreshold=1048576  # 1MB
```

### 1.3 元空间（Metaspace）

```java
# 初始元空间大小（JDK 8+）
-XX:MetaspaceSize=256m

# 最大元空间大小（默认无限制）
-XX:MaxMetaspaceSize=512m

# 设置压缩类空间大小
-XX:CompressedClassSpaceSize=1g

# 禁用元空间自动调整
-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m
```

## 二、垃圾收集器选择参数

### 2.1 串行收集器（Serial）

```java
# 年轻代使用 Serial，老年代使用 Serial Old
-XX:+UseSerialGC
```

### 2.2 并行收集器（Parallel/Throughput）

```java
# Parallel Scavenge + Parallel Old（吞吐量优先）
-XX:+UseParallelGC
-XX:+UseParallelOldGC

# 或简写为（JDK 8默认）
-XX:+UseParallelGC

# 设置并行GC线程数（默认CPU核心数）
-XX:ParallelGCThreads=4

# 设置GC最大暂停时间目标（毫秒）
-XX:MaxGCPauseMillis=200

# 设置吞吐量目标（GC时间与总时间比例）
-XX:GCTimeRatio=99  # 默认99，GC时间不超过1%
```

### 2.3 CMS收集器

```java
# 启用CMS（年轻代用ParNew）
-XX:+UseConcMarkSweepGC
-XX:+UseParNewGC

# CMS相关参数
-XX:CMSInitiatingOccupancyFraction=75  # 老年代使用率触发CMS
-XX:+UseCMSInitiatingOccupancyOnly     # 只使用上面设置的阈值

# CMS并发阶段相关
-XX:ConcGCThreads=2                    # 并发GC线程数
-XX:+CMSConcurrentMTEnabled            # 启用并发阶段多线程
-XX:+CMSScavengeBeforeRemark           # Remark前做一次Young GC
-XX:+CMSParallelRemarkEnabled          # 并行Remark

# CMS内存整理
-XX:+UseCMSCompactAtFullCollection    # Full GC时整理碎片
-XX:CMSFullGCsBeforeCompaction=0      # 每次Full GC都整理
```

### 2.4 G1收集器（JDK 9+默认）

```java
# 启用G1
-XX:+UseG1GC

# 基础参数
-XX:MaxGCPauseMillis=200              # 目标暂停时间
-XX:G1HeapRegionSize=16m              # Region大小（1-32M）
-XX:InitiatingHeapOccupancyPercent=45 # 触发并发标记的堆使用率

# 并行线程数
-XX:ConcGCThreads=4                   # 并发标记线程数
-XX:ParallelGCThreads=8               # 并行GC线程数

# 混合收集相关
-XX:G1MixedGCLiveThresholdPercent=85  # Region存活对象阈值
-XX:G1MixedGCCountTarget=8            # 混合GC最大次数
-XX:G1OldCSetRegionThresholdPercent=10 # 一次混合GC最多回收的老年代Region比例
```

### 2.5 ZGC收集器（JDK 11+）

```java
# 启用ZGC
-XX:+UseZGC

# 基础参数
-Xmx16g                                # ZGC需要大内存
-XX:ConcGCThreads=4                    # 并发GC线程数

# 高级参数
-XX:ZAllocationSpikeTolerance=2.0     # 分配尖峰容忍度
-XX:ZCollectionInterval=300           # 强制GC间隔（秒）
-XX:ZFragmentationLimit=25            # 碎片化限制百分比
```

### 2.6 Shenandoah收集器

```java
# 启用Shenandoah
-XX:+UseShenandoahGC

# 基础参数
-XX:ShenandoahGCMode=generational     # 分代模式（JDK 21+）
-XX:ShenandoahGCHeuristics=adaptive   # 启发式策略

# 调优参数
-XX:ShenandoahTargetPauseTime=200     # 目标暂停时间
-XX:ShenandoahAllocationThreshold=70  # 触发GC的分配阈值
```

## 三、GC日志参数

### 3.1 基础GC日志

```java
# 开启详细GC日志
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps

# 输出GC日志到文件
-Xloggc:/path/to/gc.log
-XX:+UseGCLogFileRotation             # 启用日志轮转
-XX:NumberOfGCLogFiles=5              # 保留文件数
-XX:GCLogFileSize=20M                 # 每个文件大小
```

### 3.2 高级GC日志（JDK 9+ Unified Logging）

```java
# JDK 9+ 统一日志格式
-Xlog:gc*:file=gc.log:time,uptime,level,tags:filecount=5,filesize=20m

# 详细配置示例
-Xlog:gc*=info:file=gc.log:time,uptimemillis,level,tags
-Xlog:gc+heap*=debug
-Xlog:gc+ergo*=trace
-Xlog:gc+age*=debug
```

### 3.3 特定GC信息

```java
# 打印晋升详情
-XX:+PrintTenuringDistribution

# 打印GC前后堆信息
-XX:+PrintHeapAtGC

# 打印应用停顿时间
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCApplicationConcurrentTime
```

## 四、性能监控参数

### 4.1 内存溢出处理

```java
# 发生OOM时生成堆转储
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dump.hprof

# 转储格式
-XX:HeapDumpGzipLevel=5               # 压缩级别（JDK 13+）

# 内存溢出前执行命令
-XX:OnOutOfMemoryError="kill -9 %p"
```

### 4.2 JIT编译相关

```java
# 方法编译阈值
-XX:CompileThreshold=10000            # 方法调用次数阈值
-XX:Tier3CompileThreshold=2000        # C1编译阈值
-XX:Tier4CompileThreshold=15000       # C2编译阈值

# 代码缓存大小
-XX:InitialCodeCacheSize=48m
-XX:ReservedCodeCacheSize=240m
-XX:+UseCodeCacheFlushing             # 启用代码缓存刷新

# 内联优化
-XX:MaxInlineSize=35                  # 字节码大小阈值
-XX:InlineSmallCode=1000              # 生成代码大小阈值
```

### 4.3 线程相关

```java
# 线程栈大小
-Xss1m  # 或 -XX:ThreadStackSize=1024

# 直接内存大小
-XX:MaxDirectMemorySize=512m

# 偏向锁优化（JDK 15后默认禁用）
-XX:+UseBiasedLocking                # 启用偏向锁
-XX:BiasedLockingStartupDelay=4000   # 延迟启用时间（ms）
```

## 五、诊断参数

### 5.1 内存泄漏诊断

```java
# 追踪对象分配位置
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./oom.hprof

# 保留符号信息
-XX:+PreserveAllClassFiles

# 追踪类加载
-XX:+TraceClassLoading
-XX:+TraceClassUnloading
```

### 5.2 慢速操作诊断

```java
# 追踪慢操作
-XX:+PrintCompilation                # 打印编译信息
-XX:+PrintInlining                   # 打印内联决策

# 监控锁竞争
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1
-XX:+UnlockDiagnosticVMOptions
-XX:+PrintBiasedLockingStatistics
```

### 5.3 安全点相关

```java
# 安全点日志
-XX:+LogVMOutput
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1

# 禁用偏向锁延迟
-XX:BiasedLockingStartupDelay=0
```

## 六、不同场景调优示例

### 6.1 Web服务器（Tomcat）调优

```
# JDK 8 + G1
java -Xms4g -Xmx4g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:InitiatingHeapOccupancyPercent=45 \
     -XX:+PrintGCDetails \
     -XX:+PrintGCDateStamps \
     -Xloggc:/opt/tomcat/logs/gc.log \
     -jar application.jar

# JDK 11+ ZGC（低延迟）
java -Xms8g -Xmx8g \
     -XX:+UseZGC \
     -XX:+UnlockExperimentalVMOptions \
     -XX:ConcGCThreads=4 \
     -Xlog:gc*:file=/opt/tomcat/logs/gc.log:time,level,tags \
     -jar application.jar
```

### 6.2 大数据处理（Spark/Flink）

```
# 吞吐量优先
java -Xmx16g -Xms16g \
     -XX:+UseParallelGC \
     -XX:+UseParallelOldGC \
     -XX:ParallelGCThreads=8 \
     -XX:GCTimeRatio=19 \
     -XX:+AlwaysPreTouch \
     -XX:+PrintFlagsFinal \
     -jar bigdata-app.jar
```

### 6.3 微服务（Spring Boot）

```
# 中小型服务
java -Xmx512m -Xms512m \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=100 \
     -XX:+UseStringDeduplication \
     -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=./heapdump.hprof \
     -jar service.jar
```

### 6.4 交易系统（低延迟）

```
# 极致低延迟
java -Xmx2g -Xms2g \
     -XX:+UseZGC \
     -XX:ConcGCThreads=2 \
     -XX:ZAllocationSpikeTolerance=5.0 \
     -XX:SoftRefLRUPolicyMSPerMB=0 \
     -XX:+DisableExplicitGC \
     -XX:+AlwaysPreTouch \
     -jar trading-system.jar
```

## 七、常见问题与解决方案

### 7.1 Full GC频繁

```
# 解决方案参数
-XX:NewRatio=1                        # 增加年轻代比例
-XX:SurvivorRatio=4                   # 减小Survivor区
-XX:MaxTenuringThreshold=5            # 降低晋升年龄
-XX:+UseCMSInitiatingOccupancyOnly    # 固定CMS触发阈值
-XX:CMSInitiatingOccupancyFraction=65 # 降低CMS触发阈值
```

### 7.2 内存泄漏定位

```
# 生产环境诊断参数
-XX:+HeapDumpOnOutOfMemoryError \
-XX:HeapDumpPath=/tmp/heapdump.hprof \
-XX:OnOutOfMemoryError="jmap -dump:format=b,file=/tmp/heapdump.hprof %p" \
-XX:+PrintClassHistogramBeforeFullGC \
-XX:+PrintClassHistogramAfterFullGC
```

### 7.3 元空间溢出

```
# 解决方案
-XX:MetaspaceSize=256m \
-XX:MaxMetaspaceSize=256m \
-XX:+UseCompressedClassPointers \
-XX:CompressedClassSpaceSize=1g \
-XX:+TraceClassLoading \
-XX:+TraceClassUnloading
```

### 7.4 线程栈溢出

```
# 调整栈大小
-Xss512k                              # 减小栈大小（更多线程）
# 或
-Xss2m                                # 增大栈大小（深度递归）
```

## 八、调优检查清单

### 8.1 基础检查

```
# 1. 堆大小设置合理
-Xms 和 -Xmx 设置相同值，避免动态调整
年轻代占堆的 1/3 到 1/2

# 2. GC收集器选择合适
吞吐量优先：UseParallelGC
低延迟：UseG1GC 或 UseZGC

# 3. 监控开启
开启GC日志
开启OOM堆转储
```

### 8.2 高级优化

```
# 1. 预热优化
-XX:+AlwaysPreTouch                  # 启动时预分配内存

# 2. 字符串去重（G1）
-XX:+UseStringDeduplication

# 3. 禁用显式GC
-XX:+DisableExplicitGC

# 4. 压缩指针
-XX:+UseCompressedOops               # 64位系统默认开启
-XX:+UseCompressedClassPointers
```

### 8.3 性能验证

```
# 查看最终参数
-XX:+PrintFlagsFinal

# 验证GC效果
jstat -gcutil <pid> 1000
jcmd <pid> GC.heap_info

# 分析GC日志
使用GCViewer、gceasy.io等工具
```

## 九、JDK版本差异

### JDK 8 常用组合

```
# Web服务
-Xmx4g -Xms4g -XX:+UseG1GC -XX:MaxGCPauseMillis=200

# 批处理
-Xmx16g -Xms16g -XX:+UseParallelGC -XX:GCTimeRatio=19

# 低延迟
-Xmx2g -Xms2g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=70
```

### JDK 11+ 推荐

```
# 默认G1已足够优秀
java -Xmx4g -Xms4g -jar app.jar

# 需要低延迟时
java -Xmx4g -Xms4g -XX:+UseZGC -jar app.jar
```

### JDK 17+ 新特性

```
# 分代ZGC（实验性）
-XX:+UseZGC -XX:+ZGenerational

# Shenandoah分代模式
-XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational
```

## 十、最佳实践总结

1. **黄金法则**：`-Xms` 和 `-Xmx` 设置相同值
2. **监控先行**：先监控再调优，数据驱动决策
3. **逐步调整**：每次只调整1-2个参数，观察效果
4. **生产验证**：所有调优在生产前进行充分测试
5. **文档记录**：记录每次调优的参数和效果