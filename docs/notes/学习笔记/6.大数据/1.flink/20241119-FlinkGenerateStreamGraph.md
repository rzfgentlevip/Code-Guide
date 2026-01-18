---
title: Flink中StreamGraph的生成
icon: streamline-freehand-color:notes-hand
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - FLINK
tag:
  - flink
  - 大数据
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 15:13:44
permalink: /learning-notes/oa1f36di/
---


# Flink中StreamGraph的生成

当我们将编写的Flink任务提交到Flink集群后，Flink作业会被抽象成不同的图然后提交到集群上执行：

- StreamGraph：程序流图，最接近用户的程序本身
- JobGraph：抽象的程序流图会被切粉橙大小不同的作业放在集群的不同节点上执行
- ExecutionGraph：JobGraph图还不能被放在集群上执行，必须抽象成执行图才能执行。
- 物理执行图：集群上真正可以执行的作业

Flink程序的编写由三部分构成：Source-->transformaer-->sink,最后调用execute()方法触发Flink程序的执行。

```java
//典型的Flink程序结构
StreamExecutionEnvironment env=StreamExecutionEnvironment.getExecutionEnvironment();
DataStreamSource<String> source = env.fromCollection(objects);
source = source.taransformer()
source.sink()

env.execute()
```

因为Flink程序是懒加载的，所以最后调用execute()方法，会触发程序的执行，接下来我们就从execute()方法开始，看看我们的Flink作业是如何被执行的.

# Execute()方法

我们调用env.execute()其实调用的是执行环境类StreamExecutionEnvironment里面的方法，首先看一下这个类的继承结构：

![https://cdn.nlark.com/yuque/0/2023/png/22919650/1703305419900-963c8fa1-6f7c-45ab-bbe0-e4f0466715f4.png](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/1703305419900-963c8fa1-6f7c-45ab-bbe0-e4f0466715f4.png)

Flink中给我们提供了多种流式执行环境：

- RemoteStreamEnvironment：远程集群上的执行环境
- LocalStreamEnvironment：本地执行环境

接下来我们重点看下StreamExecutionEnvironment这个类，因为其他的执行环境都是继承于这个类，我们主要分析如何生成StreamGraph 执行流图。

```java
//这个上下文提供了控制作业生命周期，容错性和检查点等方法，并且提供了访问外部数据流的一些方法
//StreamExecutionEnvironment类中包含很多方法，所以我们只分析一些重要的方法
@Public
public class StreamExecutionEnvironment implements AutoCloseable {
// 默认的Flink-Job名称，如果没有设置作业名称，默认使用这一个名称
public static final String DEFAULT_JOB_NAME = "Flink Streaming Job";

//环境中默认使用的时间：事件时间
/** The time characteristic that is used if none other is set. */
private static final TimeCharacteristic DEFAULT_TIME_CHARACTERISTIC =
            TimeCharacteristic.EventTime;

// 生成 StreamGraph 的入口
//触发程序的执行，执行环境将会执行所有的程序并且通过sink输出结果，sink算子将会打印结果或者将所有
结果输出到队列中。
public JobExecutionResult execute() throws Exception {
    return execute(getJobName());
}

// 一般我们都是调用【getExecutionEnvironment】方法来获取流式运行环境
// 在生成运行环境的时候还会生成配置信息
public static StreamExecutionEnvironment getExecutionEnvironment() {
    return getExecutionEnvironment(new Configuration());
}

// 获取 Flink-Job名称
private String getJobName() {
    return configuration.getString(PipelineOptions.NAME, DEFAULT_JOB_NAME);
}

// Configuration 的核心是有一个HashMap对象，用于存储键值对配置信息
public Configuration() {
    this.confData = new HashMap<>();
}
/*--------------------------------------------------*/

// execute方法的第一层包装
public JobExecutionResult execute(String jobName) throws Exception {
    //存放转换数据流的算子
    final List<Transformation<?>> originalTransformations = new ArrayList<>(transformations);
    //获取执行流图,这里会调用多个方法，将用户的算子转换形成StreamGraph
    StreamGraph streamGraph = getStreamGraph();
    // 设置job的名字
    if (jobName != null) {
        streamGraph.setJobName(jobName);
    }

    try {
        //执行streamGraph
        return execute(streamGraph);
    } catch (Throwable t) {
        Optional<ClusterDatasetCorruptedException> clusterDatasetCorruptedException =
                ExceptionUtils.findThrowable(t, ClusterDatasetCorruptedException.class);
        if (!clusterDatasetCorruptedException.isPresent()) {
            throw t;
        }

        // Retry without cache if it is caused by corrupted cluster dataset.
        invalidateCacheTransformations(originalTransformations);
        streamGraph = getStreamGraph(originalTransformations);
        return execute(streamGraph);
    }
}
//提交并且执行作业返回结果
@Internal
public JobExecutionResult execute(StreamGraph streamGraph) throws Exception {
    //异步执行作业
    final JobClient jobClient = executeAsync(streamGraph);

    try {
        final JobExecutionResult jobExecutionResult;

        if (configuration.getBoolean(DeploymentOptions.ATTACHED)) {
            //Job客户端ATTACHED模式执行作业
            jobExecutionResult = jobClient.getJobExecutionResult().get();
        } else {
            jobExecutionResult = new DetachedJobExecutionResult(jobClient.getJobID());
        }

        jobListeners.forEach(
                jobListener -> jobListener.onJobExecuted(jobExecutionResult, null));

        return jobExecutionResult;
    } catch (Throwable t) {
        // get() on the JobExecutionResult Future will throw an ExecutionException. This
        // behaviour was largely not there in Flink versions before the PipelineExecutor
        // refactoring so we should strip that exception.
        Throwable strippedException = ExceptionUtils.stripExecutionException(t);

        jobListeners.forEach(
                jobListener -> {
                    jobListener.onJobExecuted(null, strippedException);
                });
        ExceptionUtils.rethrowException(strippedException);

        // never reached, only make javac happy
        return null;
    }
}
//获取StreamGraph 将程序转换为StreamGraph的入口
@Internal
public StreamGraph getStreamGraph() {
    return getStreamGraph(true);
}
//获取StreamGraph方法
@Internal
public StreamGraph getStreamGraph(boolean clearTransformations) {
    //根据转换算子获取转换算子形成的StreamGraph
    final StreamGraph streamGraph = getStreamGraph(transformations);
    // clearTransformations：是否清除以前注册的transformations
    if (clearTransformations) {
        transformations.clear();
    }
    return streamGraph;
}

//根据转换算子，生成StreamGraph
private StreamGraph getStreamGraph(List<Transformation<?>> transformations) {
    synchronizeClusterDatasetStatus();
    return getStreamGraphGenerator(transformations).generate();
}

//获取一个StreamGraphGenerator对象
private StreamGraphGenerator getStreamGraphGenerator(List<Transformation<?>> transformations) {
    // 对transformations的数量进行校验
    if (transformations.size() <= 0) {
        throw new IllegalStateException(
                "No operators defined in streaming topology. Cannot execute.");
    }

    // We copy the transformation so that newly added transformations cannot intervene with the
    // stream graph generation.
    // 创建一个StreamGraphGenerator对象，设置参数，并调用generate方法生成StreamGraph对象
    // 向 StreamGraphGenerator传入transformations并进行相关信息的配置
    return new StreamGraphGenerator(
                    new ArrayList<>(transformations), config, checkpointCfg, configuration)
            //设置状态后端
            .setStateBackend(defaultStateBackend)
            .setChangelogStateBackendEnabled(changelogStateBackendEnabled)
            .setSavepointDir(defaultSavepointDirectory)
    // 设置算子链优化
            .setChaining(isChainingEnabled)
            // StreamExecutionEnvironment的cacheFile会传入该变量
		    // cacheFile是需要分发到各个TM的用户文件
            .setUserArtifacts(cacheFile)
            .setTimeCharacteristic(timeCharacteristic)
    // 设置超时时间
            .setDefaultBufferTimeout(bufferTimeout)
            .setSlotSharingGroupResource(slotSharingGroupResources);
}

//调用generate()方法生成StreamGraph
public StreamGraph generate() {
    // 创建一个初始的 streamGraph 对象
    streamGraph = new StreamGraph(executionConfig, checkpointConfig, savepointRestoreSettings);
    // shouldExecuteInBatchMode是一个布尔值，用来表示是否使用Batch模式
    shouldExecuteInBatchMode = shouldExecuteInBatchMode();
    configureStreamGraph(streamGraph);
	// 储存已经被处理的transformation
    alreadyTransformed = new IdentityHashMap<>();
	// 逐个处理transformation，实际就是尾StreamGraph生成各个节点（Node）
    for (Transformation<?> transformation : transformations) {
        transform(transformation);
    }

    streamGraph.setSlotSharingGroupResource(slotSharingGroupResources);

    setFineGrainedGlobalStreamExchangeMode(streamGraph);

    for (StreamNode node : streamGraph.getStreamNodes()) {
        if (node.getInEdges().stream().anyMatch(this::shouldDisableUnalignedCheckpointing)) {
            for (StreamEdge edge : node.getInEdges()) {
                edge.setSupportsUnalignedCheckpoints(false);
            }
        }
    }
	// 获取生成完毕的streamGraph
    final StreamGraph builtStreamGraph = streamGraph;
	// 清除中间变量
    alreadyTransformed.clear();
    alreadyTransformed = null;
    streamGraph = null;

    return builtStreamGraph;
	}
}
```

从上面的代码分析可以看到，我们提交的Flink作业，会首先被转换为StreamGraph然后提交到集群中执行，大致有一下几个步骤：

1. 根据代码和配置信息生成 StreamGraphGenerator 对象；
2. StreamGraphGenerator 对象调用 generate() 方法生成 StreamGraph 对象，底层操作是遍历 transformations 集合，创建 StreamNode 和 StreamEdge，构造 **StreamGraph**对象；
3. 根据 StreamGraph 对象执行；

我们可以看到在getStreamGraphGenerator(transformations).generate()方法调用过程中;方法中生成节点时使用的是【Transformation】对象，那么这个对象是什么呢？在创建 StreamGraphGenerator 对象时我们传递了一个重要参数**transformations**，它是 env 的成员变量之一，用一个 List<Transformation<?>> 对象来保存。Transformation 对象代表了一个或多个 DataStream 生成新 DataStream 的操作，也可以理解为是数据流处理环节中的一步，更简单的说话就是一个算子或者多个算子的组合。

上面整个流程下来就是生成StreamGraph的过程，主要关键点如下如所示：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled.png)

# Transformation对象

在生成StreamGraph的过程中有一个重要的对象，Transformation,程序中就是根据这个集合 对象来形成了StreamGraph，Transformation在StreamExecutionEnvironment执行环境中是一个集合：

```java
protected final List<Transformation<?>> transformations = new ArrayList<>();
```

`Transformation`代表了生成 `DataStream` 的操作，每一个 `DataStream` 的底层都有对应的一个 `Transformation`。在 `DataStream`上面通过 `map` 等算子不断进行转换，就得到了由 `Transformation`构成的图。当需要执行的时候，底层的这个图就会被转换成 `StreamGraph`。

`Transformation`在运行时并不一定对应着一个物理转换操作，有一些操作只是逻辑层面上的，比如 split/select/partitioning 等。

每一个 `Transformation`都有一个关联的 Id，这个 Id 是全局递增的。除此以外，还有 uid, slotSharingGroup, parallelism 等信息。

`Transformation`有很多具体的子类，如`SourceTransformation`、 `OneInputStreamTransformation`、`TwoInputTransformation`、`SideOutputTransformation`、 `SinkTransformation` 等等，这些分别对应了`DataStream` 上的不同转换操作。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%201.png)

由于 Transformation中通常保留了其前向的 Transformation，即其输入，因此可以据此还原出 DAG 的拓扑结构。

从`OneInputTransformation`**和**`TwoInputTransformation实现，可以蓝岛，里面记录了其前向的输入流来源：`

```java
public class TwoInputTransformation<IN1, IN2, OUT> extends PhysicalTransformation<OUT> {

//其前向输入流
    private final Transformation<IN1> input1;
    private final Transformation<IN2> input2;

    private final StreamOperatorFactory<OUT> operatorFactory;

    private KeySelector<IN1, ?> stateKeySelector1;

    private KeySelector<IN2, ?> stateKeySelector2;

    private TypeInformation<?> stateKeyType;
}

public class OneInputTransformation<IN, OUT> extends PhysicalTransformation<OUT> {
//其前向输入流
    private final Transformation<IN> input;

    private final StreamOperatorFactory<OUT> operatorFactory;

    private KeySelector<IN, ?> stateKeySelector;

    private TypeInformation<?> stateKeyType;
}
```

我们可以从Transformation源码中了解到，起就是代表着形成数据流的一个一个转换算子。

```java
{@code Transformation} represents the operation that creates a DataStream. Every DataStream has
an underlying {@code Transformation} that is the origin of said DataStream.
代表创建DataStream的一种算子。每一个DataStream的底层都有这一个Transformation代表数据流转换
<p>API operations such as DataStream#map create a tree of {@code Transformation}s underneath.
When the stream program is to be executed this graph is translated to a StreamGraph using
StreamGraphGenerator.

<p>A {@code Transformation} does not necessarily correspond to a physical operation at runtime.
Some operations are only logical concepts. Examples of this are union, split/select data stream,
partitioning.
```

接下来我们来看一下最简单的map算子的底层是如何使用Transformation：

```java
/*--------------------- DataStream -----------------------------*/
// DataStream类
public class DataStream<T> {  
	// 运行环境
    protected final StreamExecutionEnvironment environment;  
	// 生成该DataStream的transformation
    protected final Transformation<T> transformation;

...
}
/*-------------------------------------------------------------*/

// map方法入口
public <R> SingleOutputStreamOperator<R> map(MapFunction<T, R> mapper) {  

	// 获取Mapper函数的输出类型
    TypeInformation<R> outType =  
            TypeExtractor.getMapReturnTypes(  
                    clean(mapper), getType(), Utils.getCallLocationName(), true);  
	// 返回SingleOutputStreamOperator
    return map(mapper, outType);  
}

// map方法核心
// mapper：MapFunction
// outputType：mapper函数的返回类型
public <R> SingleOutputStreamOperator<R> map(  
        MapFunction<T, R> mapper, TypeInformation<R> outputType) {
//map中真正调用的是transform()方法，封装用户自定义的mapFunction()方法;StreamMap
    return transform("Map", outputType, new StreamMap<>(clean(mapper)));  
}

// operatorName：算子名称
// outTypeInfo：算子返回类型信息
// operator：包含转换逻辑的算子
public <R> SingleOutputStreamOperator<R> transform(  
        String operatorName,  
        TypeInformation<R> outTypeInfo,  
        OneInputStreamOperator<T, R> operator) {  

    return doTransform(operatorName, outTypeInfo, SimpleOperatorFactory.of(operator));  
}

// operatorName：算子名称
// outTypeInfo：算子返回类型信息
// operatorFactory：算子工厂
protected <R> SingleOutputStreamOperator<R> doTransform(  
        String operatorName,  
        TypeInformation<R> outTypeInfo,  
        StreamOperatorFactory<R> operatorFactory) {  
  
    // 如果transformation的输出类型为MissingTypeInfo的话，程序会抛异常
    transformation.getOutputType();  

	// 构造新的transformation
	// map类型的transformation只有一个输入，因此它输入OneInputTransformation
	// 新的transformation会连接上当前DataStream中的transformation，从而构建成一棵树
	// 生成StreamGraph依赖于transformation构建的树
    OneInputTransformation<T, R> resultTransform =  
            new OneInputTransformation<>(  
                    this.transformation,  
                    operatorName,  
                    operatorFactory,  
                    outTypeInfo,  
                    environment.getParallelism());  
  
    @SuppressWarnings({"unchecked", "rawtypes"})  
    SingleOutputStreamOperator<R> returnStream =  
            new SingleOutputStreamOperator(environment, resultTransform);  

	// 将transformation写入ExecutionEnvironment中
    // ExecutionEnvironment维护了一个叫做transformations的ArrayList对象，用于储存所有的transformation
    getExecutionEnvironment().addOperator(resultTransform);  
  
    return returnStream;  
}

/// 获取运行环境
public StreamExecutionEnvironment getExecutionEnvironment() {  
    return environment;  
}

/*--------------- StreamExecutionEnvironment -----------------*/

// 保存所有的transformation
protected final List<Transformation<?>> transformations = new ArrayList<>();

// 向env中添加transformation
public void addOperator(Transformation<?> transformation) {  
    Preconditions.checkNotNull(transformation, "transformation must not be null.");  
    this.transformations.add(transformation);  
}

/*-------------------------------------------------------------*/

/*---------------- OneInputTransformation ----------------------*/
public class OneInputTransformation<IN, OUT> extends PhysicalTransformation<OUT> {
	// 前一个Transformation的指针
	private final Transformation<IN> input;
	// 封装的StreamOperator工厂
	private final StreamOperatorFactory<OUT> operatorFactory;
	...
}
/*-------------------------------------------------------------*/
```

现在我们小结一下，将几个名词做一下统一的梳理：

- Transformation：描述一个 DataStream 的生成的对象，内部对算子进行了封装；
- StreamOperator：DataStream 上的每一个 transformation 内部都对应了一个 StreamOperator，StreamOperator 是运行时的具体实现。StreamOperator其实就是来封装具体的用户自定义Function。
- DataStream上的每一个转换底层都对应这transformation,而具体用户实现的转换是通过StreamOperator 接口存储在transformation对象中，transformation对象又保存了其前置输入流，这样就可以形成一个StreamGraph图了。

基于上述分析，一个 StreamGraph 的生成逻辑链如下：

1. 每一个算子的底层都是一个 StreamOperator ，先封装到 OperatorFactory 中，然后用一个 Transformation 对象进行封装；
2. 每一个 Transformation 通过【input】指针连接在一起构成一个树；
3. 通过 Transformation 树构造一个 StreamGraph；

为了更好的理解上述过程，以 **`map`** 算子为例，其 Transformation 树如下：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%202.png)

一个 `DataStream` 就表征了由同一种类型元素构成的数据流。通过对 `DataStream` 应用 map/filter 等操作，可以将一个 `DataStream` 转换为另一个 `DataStream`，这个转换的过程就是根据不同的操作生成不同的 Transformation(个人将Transformation理解为具体的转换)，并将其加入 `StreamExecutionEnvironment` 的 `transformations` 列表中。

一个更加通用的算子实现：

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%203.png)

例如DataStream中的map方法：

```java
protected <R> SingleOutputStreamOperator<R> doTransform(
            String operatorName,
            TypeInformation<R> outTypeInfo,
            StreamOperatorFactory<R> operatorFactory) {

        // read the output type of the input Transform to coax out errors about MissingTypeInfo
        transformation.getOutputType();
//构造OneInputTransformation对象
        OneInputTransformation<T, R> resultTransform =
                new OneInputTransformation<>(
                        this.transformation,
                        operatorName,
                        operatorFactory,
                        outTypeInfo,
                        environment.getParallelism(),
                        false);

        @SuppressWarnings({"unchecked", "rawtypes"})
        SingleOutputStreamOperator<R> returnStream =
                new SingleOutputStreamOperator(environment, resultTransform);
//将OneInputTransformation对象添加到执行环境中
        getExecutionEnvironment().addOperator(resultTransform);

        return returnStream;
    }
```

`DataStream` 的子类包括 `SingleOutputStreamOperator`、 `DataStreamSource KeyedStream` 、`IterativeStream`, `SplitStream`(已弃用)。这里要吐槽一下 `SingleOutputStreamOperator` 的这个类的命名，太容易和 `StreamOperator` 混淆了。`StreamOperator` 的介绍见下一小节。

除了 `DataStream` 及其子类以外，其它的表征数据流的类还有 `ConnectedStreams` (两个流连接在一起)、 `WindowedStream`、`AllWindowedStream` 。这些数据流之间的转换可以参考 Flink 的官方文档。

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%204.png)

## Transformation

### OneInputTransformation

顾名思义 OneInputTransformation 只有一个输入，它主要包装 `map`、`flatmap`、`filter` 等算子。

### TwoInputTransformation

TwoInputTransformation 具有两个输入。ConnectedStream 的算子为双流运算，它的算子会被转换为 TwoInputTransformation。

### SourceTransformation

在环境中配置数据源的时候会创建一个 DataStreamSource 对象。该对象为 DataStream 的源头。在 DataStreamSource 的构造函数中就会创建一个 SourceTransformation 对象；

### SinkTransformation

同 SourceTransformation，DataStream 对象添加 sink 源的时候就会生成一个 DataStreamSink 对象，同时构造一个 SinkTransformation 对象。

### UnionTransformation

UnionTransformation 合并多个 input 到一个流中。代表算子为 union。

### FeedbackTransformation

创建 IterativeStream 的时候会使用到 FeedbackTransformation，它实质上表示拓扑中的反馈节点。

### CoFeedbackTransformation

创建 ConnectedIterativeStream 的时候会使用到 CoFeedbackTransformation。

### PartitionTransformation

涉及到控制数据流向的算子都属于 PartitionTransformation，例如 shuffle，forward，rebalance，broadcast，rescale，global，partitionCustom 和 keyBy 等。

### SideOutputTransformation

当获取侧输出流的时候会生成 SideOutputTransformation。

作者：狗狗的数仓梦链接：https://juejin.cn/post/7195133794301509687来源：稀土掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

现在来看一下官方的 SocketWindowWordCount 示例，该示例的数据处理流程是【Source→Flat Map→Hash (keyBy)→TriggerWindow→Sink】, 那么它的 Transformations 树如下，其中 `*` 代表 input 指针；

[https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d4181c23d014ebea05b37f2b1f936d5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d4181c23d014ebea05b37f2b1f936d5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

## **StreamOperator**

在操作 `DataStream` 的时候，比如 `DataStream#map` 等，会要求我们提供一个自定义的处理函数。那么这些信息时如何保存在 `Transformation` 中的呢？这里就要引入一个新的接口 `StreamOperator`。

`StreamOperator` 定义了对一个具体的算子的生命周期的管理，包括：

```java
@PublicEvolving
public interface StreamOperator<OUT> extends CheckpointListener, KeyContext, Serializable {

    // ------------------------------------------------------------------------
    //  life cycle
    // ------------------------------------------------------------------------

    /**
     * This method is called immediately before any elements are processed, it should contain the
     * operator's initialization logic.
			包含初始化逻辑
     */
    void open() throws Exception;

    /**
     * This method is called at the end of data processing.
     *在处理数据的结尾执行
     */
    void finish() throws Exception;

    void close() throws Exception;

    void prepareSnapshotPreBarrier(long checkpointId) throws Exception;

    /**
     * Called to draw a state snapshot from the operator.
			状态管理
     */
    OperatorSnapshotFutures snapshotState(
            long checkpointId,
            long timestamp,
            CheckpointOptions checkpointOptions,
            CheckpointStreamFactory storageLocation)
            throws Exception;

    /** Provides a context to initialize all state in the operator. */
    void initializeState(StreamTaskStateInitializer streamTaskStateManager) throws Exception;

    // ------------------------------------------------------------------------
    //  miscellaneous
    // ------------------------------------------------------------------------

    void setKeyContextElement1(StreamRecord<?> record) throws Exception;

    void setKeyContextElement2(StreamRecord<?> record) throws Exception;

    OperatorMetricGroup getMetricGroup();

    OperatorID getOperatorID();
}
```

`StreamOperator` 的两个子接口 `OneInputStreamOperator` 和 `TwoInputStreamOperator` 则提供了操作数据流中具体元素的方法，而 `AbstractStreamOperator`这个抽象子类则提供了自定义处理函数对应的算子的基本实现:

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%205.png)

```java
//OneInputStreamOperator接口
@PublicEvolving
public interface OneInputStreamOperator<IN, OUT> extends StreamOperator<OUT>, Input<IN> {
    @Override
    default void setKeyContextElement(StreamRecord<IN> record) throws Exception {
        setKeyContextElement1(record);
    }
}

// TwoInputStreamOperator 两个输入流接口
@PublicEvolving
public interface TwoInputStreamOperator<IN1, IN2, OUT> extends StreamOperator<OUT> {

    void processElement1(StreamRecord<IN1> element) throws Exception;
    void processElement2(StreamRecord<IN2> element) throws Exception;
    void processWatermark1(Watermark mark) throws Exception;
    void processWatermark2(Watermark mark) throws Exception;
    void processLatencyMarker1(LatencyMarker latencyMarker) throws Exception;
    void processLatencyMarker2(LatencyMarker latencyMarker) throws Exception;
    void processWatermarkStatus1(WatermarkStatus watermarkStatus) throws Exception;
    void processWatermarkStatus2(WatermarkStatus watermarkStatus) throws Exception;
}

//用户自定义需要实现这个抽象类
public abstract class AbstractStreamOperator<OUT>{}
```

至于具体到诸如 map/fliter 等操作对应的 `StreamOperator`，基本都是在 AbstractStreamOperator的基础上实现的。以DataStream中的map方法为例：

```java
//参数是用户自定义传入的MapFunction
public <R> SingleOutputStreamOperator<R> map(MapFunction<T, R> mapper) {

        TypeInformation<R> outType =
                TypeExtractor.getMapReturnTypes(
                        clean(mapper), getType(), Utils.getCallLocationName(), true);

        return map(mapper, outType);
    }

//map的第一层调用
public <R> SingleOutputStreamOperator<R> map(
            MapFunction<T, R> mapper, TypeInformation<R> outputType) {
//这里new StreamMap()对用户自定义的MapFunction进行包装
        return transform("Map", outputType, new StreamMap<>(clean(mapper)));
    }

//可以看到StreamMap是继承AbstractUdfStreamOperator抽象类，而AbstractUdfStreamOperator抽象类又是
继承AbstractStreamOperator抽象基类
@Internal
public class StreamMap<IN, OUT> extends AbstractUdfStreamOperator<OUT, MapFunction<IN, OUT>>
        implements OneInputStreamOperator<IN, OUT> {

    private static final long serialVersionUID = 1L;

    public StreamMap(MapFunction<IN, OUT> mapper) {
//传递给父类的protected final F userFunction; 属性
        super(mapper);
        chainingStrategy = ChainingStrategy.ALWAYS;
    }
//map函数的底层还是调用processElement()函数
    @Override
    public void processElement(StreamRecord<IN> element) throws Exception {
//在这里调用的实现方法
        output.collect(element.replace(userFunction.map(element.getValue())));
    }
}
```

由此，通过 `DataStream` –> `Transformation` –> `StreamOperator` 这样的依赖关系，就可以完成 `DataStream` 的转换，并且保留数据流和应用在流上的算子之间的关系。

了解了`Transformation` 和`StreamOperator` 对象，最后我们看看如何具体生成StreamGraph逻辑执行图：

`StreamGraph` 会基于 `StreamExecutionEnvironment` 的 `transformations` 列表来生成 `StreamGraph。在StreamExecutionEnvironment` `源码中，可以看到有一个List列表来保存transformations` `对象：`

```java
protected final List<Transformation<?>> transformations = new ArrayList<>();
```

在调用生成StreamGraph的时候，会将transformations列表传入

```java
@Internal
    public StreamGraph getStreamGraph(boolean clearTransformations) {
//传入transformations列表构造StreamGraph 
        final StreamGraph streamGraph = getStreamGraph(transformations);
        if (clearTransformations) {
            transformations.clear();
        }
        return streamGraph;
    }
```

而在getStreamGraph方法中，构造StreamGraph的逻辑就是遍历`transformations`然后调用`transform`方法生成StreamGraph。对于每一个 `Transformation`, 确保当前其上游已经完成转换。`Transformations` 被转换为 `StreamGraph` 中的节点 `StreamNode`，并为上下游节点添加边 `StreamEdge`。

```java
// transform方法 Transforms one {@code Transformation}.
private Collection<Integer> transform(Transformation<?> transform) {  

	// 如果一个 Transformation 已经被处理，那么直接返回
    if (alreadyTransformed.containsKey(transform)) {  
        return alreadyTransformed.get(transform);  
    }  
  
    LOG.debug("Transforming " + transform);  

	// 如果transformation的最大并行度没有设置，全局的最大并行度已设置，将全局最大并行度设置给transformation
    if (transform.getMaxParallelism() <= 0) {  
  
        // if the max parallelism hasn't been set, then first use the job wide max parallelism  
        // from the ExecutionConfig.        int globalMaxParallelismFromConfig = executionConfig.getMaxParallelism();  
        if (globalMaxParallelismFromConfig > 0) {  
            transform.setMaxParallelism(globalMaxParallelismFromConfig);  
        }  
    }  
  
    // 检查transformation的输出类型，如果是MissingTypeInfo则程序抛出异常 
    transform.getOutputType();  

	// 根据Transformation 获取translator
    @SuppressWarnings("unchecked")  
    final TransformationTranslator<?, Transformation<?>> translator =  
            (TransformationTranslator<?, Transformation<?>>)  
                    translatorMap.get(transform.getClass());  

	// 使用translator进行转换
    Collection<Integer> transformedIds;  
    if (translator != null) {  
        transformedIds = translate(translator, transform);  
    } else {  
        transformedIds = legacyTransform(transform);  
    }  

	// 应为有反馈边的存在，所以需要进行这一步检查
	// 防止递归情况下的重复
   if (!alreadyTransformed.containsKey(transform)) {  
        alreadyTransformed.put(transform, transformedIds);  
    }  
  
    return transformedIds;  
}

private Collection<Integer> translate(  
        final TransformationTranslator<?, Transformation<?>> translator,  
        final Transformation<?> transform) {  
    checkNotNull(translator);  
    checkNotNull(transform);  
  
    final List<Collection<Integer>> allInputIds = getParentInputIds(transform.getInputs());  
  
    // 防递归调用
    if (alreadyTransformed.containsKey(transform)) {  
        return alreadyTransformed.get(transform);  
    }  

	// 获取共享Slot组
    final String slotSharingGroup =  
            determineSlotSharingGroup(  
                    transform.getSlotSharingGroup(),  
                    allInputIds.stream()  
                            .flatMap(Collection::stream)  
                            .collect(Collectors.toList()));  
	// 获取上下文对象
    final TransformationTranslator.Context context =  
            new ContextImpl(this, streamGraph, slotSharingGroup, configuration);  

	// 根据Batch模式还是Streaming模式进行转换
	// 就需要两个参数一个transform，一个context（上下文）
    return shouldExecuteInBatchMode  
            ? translator.translateForBatch(transform, context)  
            : translator.translateForStreaming(transform, context);  
}

/*---------------- StreamGraphGenerator --------------------*/
// 使用一个HashMap保存各个Transformation的转换器
private static final Map<
                    Class<? extends Transformation>,
                    TransformationTranslator<?, ? extends Transformation>>
            translatorMap;

	// 在一个静态代码块中添加Transformation-TransformationTranslator
    static {
        @SuppressWarnings("rawtypes")
        Map<Class<? extends Transformation>, TransformationTranslator<?, ? extends Transformation>>
                tmp = new HashMap<>();
        tmp.put(OneInputTransformation.class, new OneInputTransformationTranslator<>());
        tmp.put(TwoInputTransformation.class, new TwoInputTransformationTranslator<>());
        tmp.put(MultipleInputTransformation.class, new MultiInputTransformationTranslator<>());
        tmp.put(KeyedMultipleInputTransformation.class, new MultiInputTransformationTranslator<>());
        tmp.put(SourceTransformation.class, new SourceTransformationTranslator<>());
        tmp.put(SinkTransformation.class, new SinkTransformationTranslator<>());
        tmp.put(LegacySinkTransformation.class, new LegacySinkTransformationTranslator<>());
        tmp.put(LegacySourceTransformation.class, new LegacySourceTransformationTranslator<>());
        tmp.put(UnionTransformation.class, new UnionTransformationTranslator<>());
        tmp.put(PartitionTransformation.class, new PartitionTransformationTranslator<>());
        tmp.put(SideOutputTransformation.class, new SideOutputTransformationTranslator<>());
        tmp.put(ReduceTransformation.class, new ReduceTransformationTranslator<>());
        tmp.put(
                TimestampsAndWatermarksTransformation.class,
                new TimestampsAndWatermarksTransformationTranslator<>());
        tmp.put(BroadcastStateTransformation.class, new BroadcastStateTransformationTranslator<>());
        tmp.put(
                KeyedBroadcastStateTransformation.class,
                new KeyedBroadcastStateTransformationTranslator<>());
        // 赋给成员变量translatorMap
        translatorMap = Collections.unmodifiableMap(tmp);
    }

/*-------------------------------------------------------------*/
```

从上面的分析可以看到，想要生成 StreamGraph 的一个节点，需要一个 Context 对象和 Transformation 对象，并且每一个转换器会根据运行模式的不同进行不同的转换，所有的转换器都实现了 `TransformationTranslator` 接口，它的源码分析如下：

```java
public interface TransformationTranslator<OUT, T extends Transformation<OUT>> {
	// 批处理模式下的转换方法
	// transformation：要转换的Transformation；
	// context：上下文对象，给转换提供必要信息
    Collection<Integer> translateForBatch(final T transformation, final Context context);

	// 流式处理模式下的转换方法
	// transformation：要转换的Transformation；
	// context：上下文对象，给转换提供必要信息
    Collection<Integer> translateForStreaming(final T transformation, final Context context);

	// 上下文接口
    interface Context {
		// 返回正在创建的 StreamGraph
        StreamGraph getStreamGraph();

		// 根据Transformation对象返回节点ID
        Collection<Integer> getStreamNodeIds(final Transformation<?> transformation);

		// 根据给定的Transformations返回Slot共享组
        String getSlotSharingGroup();

		// 返回默认超时时间
        long getDefaultBufferTimeout();

		// 返回额外配置信息
        ReadableConfig getGraphGeneratorConfig();
    }
}
```

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%206.png)

接下来我们以 OneInputTransformationTranslator 来进一步说明转换流程，它的源码分析如下：

```java
// 流模式下的转换方法
public Collection<Integer> translateForStreamingInternal(  
        final OneInputTransformation<IN, OUT> transformation, final Context context) {  
    return translateInternal(  
            transformation,  
            transformation.getOperatorFactory(),  
            transformation.getInputType(),  
            transformation.getStateKeySelector(),  
            transformation.getStateKeyType(),  
            context);  
}

protected Collection<Integer> translateInternal(  
        final Transformation<OUT> transformation,  
        final StreamOperatorFactory<OUT> operatorFactory,  
        final TypeInformation<IN> inputType,  
        @Nullable final KeySelector<IN, ?> stateKeySelector,  
        @Nullable final TypeInformation<?> stateKeyType,  
        final Context context) {  
    checkNotNull(transformation);  
    checkNotNull(operatorFactory);  
    checkNotNull(inputType);  
    checkNotNull(context);  

	// 由context对象获取StreamGraph对象
    final StreamGraph streamGraph = context.getStreamGraph();  
	// 由context对象获取slot共享组
    final String slotSharingGroup = context.getSlotSharingGroup();  
	// 获取Transformation对象的ID
    final int transformationId = transformation.getId();  
	// 获取运行时配置信息
    final ExecutionConfig executionConfig = streamGraph.getExecutionConfig();  

	// 添加 Operator
    streamGraph.addOperator(  
            transformationId,  
            slotSharingGroup,  
            transformation.getCoLocationGroupKey(),  
            operatorFactory,  
            inputType,  
            transformation.getOutputType(),  
            transformation.getName());  
  
    if (stateKeySelector != null) {  
        TypeSerializer<?> keySerializer = stateKeyType.createSerializer(executionConfig);  
        streamGraph.setOneInputStateKey(transformationId, stateKeySelector, keySerializer);  
    }  

	// 获取并行度
    int parallelism =  
            transformation.getParallelism() != ExecutionConfig.PARALLELISM_DEFAULT  
                    ? transformation.getParallelism()  
                    : executionConfig.getParallelism();  
    // 设置并行度
    streamGraph.setParallelism(transformationId, parallelism);  
	// 设置最大并行度
    streamGraph.setMaxParallelism(transformationId, transformation.getMaxParallelism());  
  
    final List<Transformation<?>> parentTransformations = transformation.getInputs();  
    checkState(  
            parentTransformations.size() == 1,  
            "Expected exactly one input transformation but found "  
                    + parentTransformations.size());  

	// 添加边信息
    for (Integer inputId : context.getStreamNodeIds(parentTransformations.get(0))) {  
        streamGraph.addEdge(inputId, transformationId, 0);  
    }  
  
    return Collections.singleton(transformationId);  
}

/*------------------------ StreamGraph ---------------------------*/
// 添加Operator方法
private <IN, OUT> void addOperator(  
        Integer vertexID,  
        @Nullable String slotSharingGroup,  
        @Nullable String coLocationGroup,  
        StreamOperatorFactory<OUT> operatorFactory,  
        TypeInformation<IN> inTypeInfo,  
        TypeInformation<OUT> outTypeInfo,  
        String operatorName,  
        Class<? extends AbstractInvokable> invokableClass) {  

	// 添加节点信息
	// 这一步已经创建节点并添加到HashMap中了
    addNode(  
            vertexID,  
            slotSharingGroup,  
            coLocationGroup,  
            invokableClass,  
            operatorFactory,  
            operatorName); 

	// 这一步设置了输入、输出序列化器
    setSerializers(vertexID, createSerializer(inTypeInfo), null, createSerializer(outTypeInfo));  

	// operator工厂设置输出类型，在创建StreamGraph的时候就必须确定
    if (operatorFactory.isOutputTypeConfigurable() && outTypeInfo != null) {  
        operatorFactory.setOutputType(outTypeInfo, executionConfig);  
    }  

	// operator工厂设置输入类型
    if (operatorFactory.isInputTypeConfigurable()) {  
        operatorFactory.setInputType(inTypeInfo, executionConfig);  
    }  

	// 打印日志
    if (LOG.isDebugEnabled()) {  
        LOG.debug("Vertex: {}", vertexID);  
    }  
}

// 创建序列化器
private <T> TypeSerializer<T> createSerializer(TypeInformation<T> typeInfo) {  
	// 如果typeInfo是null或者MissingTypeInfo，那么序列化器就是null
	// 如果不是则调用typeInfo的createSerializer创建序列化器
    return typeInfo != null && !(typeInfo instanceof MissingTypeInfo)  
            ? typeInfo.createSerializer(executionConfig)  
            : null;  
}

// StreamNode设置序列化器
public void setSerializers(  
        Integer vertexID, TypeSerializer<?> in1, TypeSerializer<?> in2, TypeSerializer<?> out) {  
	// 根据ID获取 StreamNode 对象
    StreamNode vertex = getStreamNode(vertexID);  
    // StreamNode设置输入序列化器
    vertex.setSerializersIn(in1, in2);  
    // StreamNode设置输出序列化器
    vertex.setSerializerOut(out);  
}

// 添加节点
protected StreamNode addNode(  
        Integer vertexID,  
        @Nullable String slotSharingGroup,  
        @Nullable String coLocationGroup,  
        Class<? extends AbstractInvokable> vertexClass,  
        StreamOperatorFactory<?> operatorFactory,  
        String operatorName) {  

	// 检查是否重复添加了同样的节点
    if (streamNodes.containsKey(vertexID)) {  
        throw new RuntimeException("Duplicate vertexID " + vertexID);  
    }  

	// 创建StreamNode对象，
    StreamNode vertex =  
            new StreamNode(  
                    vertexID,  
                    slotSharingGroup,  
                    coLocationGroup,  
                    operatorFactory,  
                    operatorName,  
                    vertexClass);  

	// 使用一个HashMap保存各个vertexID-StreamNode键值对
    streamNodes.put(vertexID, vertex);  
  
    return vertex;  
}

/*-------------------------------------------------------------*/

/*---------------------- StreamNode -----------------------*/
public class StreamNode {
	// 节点的最大并行度，用于扩、缩容时的上限，以及分区时候的键组数
	private int maxParallelism;  
	// 节点所需最小资源
	private ResourceSpec minResources = ResourceSpec.DEFAULT;  
	// 节点所需最佳资源
	private ResourceSpec preferredResources = ResourceSpec.DEFAULT;
	// Operator生成工厂
	private StreamOperatorFactory<?> operatorFactory;  
	// 输入序列化器
	private TypeSerializer<?>[] typeSerializersIn = new TypeSerializer[0];  
	// 输出序列化器
	private TypeSerializer<?> typeSerializerOut;
	// 节点输入的边
	private List<StreamEdge> inEdges = new ArrayList<StreamEdge>();  
	// 节点输出的边
	private List<StreamEdge> outEdges = new ArrayList<StreamEdge>();
	// 输入格式器
	// 1. 它描述了如何将输入进行并行拆分
	// 2. 它描述了如何从输入中读取数据
	// 3. 它描述了如何从输入中进行统计
	private InputFormat<?, ?> inputFormat;  
	// 输出格式器
	// 它描述了如何将节点结果进行输出
	private OutputFormat<?> outputFormat;
	// 网络超时时间
	private long bufferTimeout;  
	// 算子名称
	private final String operatorName;  
	// Slot共享组组名
	private @Nullable String slotSharingGroup;
	...
}
/*---------------------------------------------------------*/
```

到这一步 StreamGraph 已经完成添加节点的工作，接下来将添加边（各个节点之间的连接）。

接着看一看 `StreamGraph` 中对应的添加节点和边的方法:

```java
/*------------------------ StreamGraph ---------------------------*/
// 用来存储SideOut虚拟节点
private Map<Integer, Tuple2<Integer, OutputTag>> virtualSideOutputNodes;  
// 用来存储Partition虚拟节点
private Map<Integer, Tuple3<Integer, StreamPartitioner<?>, ShuffleMode>> virtualPartitionNodes;

// 添加边入口
public void addEdge(Integer upStreamVertexID, Integer downStreamVertexID, int typeNumber) {  
	
    addEdgeInternal(  
            upStreamVertexID,  
            downStreamVertexID,  
            typeNumber,  
            null,  
            new ArrayList<String>(),  
            null,  
            null);  
}

// 添加节点之间的连接
private void addEdgeInternal(  
        Integer upStreamVertexID,  
        Integer downStreamVertexID,  
        int typeNumber,  
        StreamPartitioner<?> partitioner,  
        List<String> outputNames,  
        OutputTag outputTag,  
        ShuffleMode shuffleMode) {  

	// 当上游是sideOutput时，递归调用，并传入sideOutput信息，递归寻找非virtual节点
    if (virtualSideOutputNodes.containsKey(upStreamVertexID)) {  
        int virtualId = upStreamVertexID;  
        upStreamVertexID = virtualSideOutputNodes.get(virtualId).f0;  
        if (outputTag == null) {  
            outputTag = virtualSideOutputNodes.get(virtualId).f1;  
        }  
        addEdgeInternal(  
                upStreamVertexID,  
                downStreamVertexID,  
                typeNumber,  
                partitioner,  
                null,  
                outputTag,  
                shuffleMode);  
    // 当上游是select时，递归调用，并传入select信息，递归寻找非virtual节点
    } else if (virtualPartitionNodes.containsKey(upStreamVertexID)) {  
        int virtualId = upStreamVertexID;  
        upStreamVertexID = virtualPartitionNodes.get(virtualId).f0;  
        if (partitioner == null) {  
            partitioner = virtualPartitionNodes.get(virtualId).f1;  
        }  
        shuffleMode = virtualPartitionNodes.get(virtualId).f2;  
        addEdgeInternal(  
                upStreamVertexID,  
                downStreamVertexID,  
                typeNumber,  
                partitioner,  
                outputNames,  
                outputTag,  
                shuffleMode);  
    } else {  
	    // 上游节点
        StreamNode upstreamNode = getStreamNode(upStreamVertexID);  
        // 下游节点
        StreamNode downstreamNode = getStreamNode(downStreamVertexID);  

		// 如果没有指定具体的分区器，且上下游节点的并行度一致使用ForwardPartitioner
		// 其他情况都是用RebalancePartitioner
        if (partitioner == null  
                && upstreamNode.getParallelism() == downstreamNode.getParallelism()) {  
            partitioner = new ForwardPartitioner<Object>();  
        } else if (partitioner == null) {  
            partitioner = new RebalancePartitioner<Object>();  
        }  

		// 检查如果指定了ForwardPartitioner但是上下游并行度不一致，则抛出异常
        if (partitioner instanceof ForwardPartitioner) {  
            if (upstreamNode.getParallelism() != downstreamNode.getParallelism()) {  
                throw new UnsupportedOperationException(  
                        "Forward partitioning does not allow "  
                                + "change of parallelism. Upstream operation: "  
                                + upstreamNode  
                                + " parallelism: "  
                                + upstreamNode.getParallelism()  
                                + ", downstream operation: "  
                                + downstreamNode  
                                + " parallelism: "  
                                + downstreamNode.getParallelism()  
                                + " You must use another partitioning strategy, such as broadcast, rebalance, shuffle or global.");  
            }  
        } 
         
	    // 如果没有执行shuffleMode则使用ShuffleMode.UNDEFINED
	    // 下面将会详细讲解ShuffleMode的各个枚举值含义
        if (shuffleMode == null) {  
            shuffleMode = ShuffleMode.UNDEFINED;  
        }  

		// 创建一个StreamEdge对象
        StreamEdge edge =  
                new StreamEdge(  
                        upstreamNode,  
                        downstreamNode,  
                        typeNumber,  
                        partitioner,  
                        outputTag,  
                        shuffleMode);  
                        
        // 在输入和输出投添加创建的StreamEdge对象
        getStreamNode(edge.getSourceId()).addOutEdge(edge);  
        getStreamNode(edge.getTargetId()).addInEdge(edge);  
    }  
}
/*---------------------------------------------------------*/

/*---------------------- StreamNode -----------------------*/
// 添加节点输入边
public void addInEdge(StreamEdge inEdge) {  
	// 检查StreamNode的ID是否匹配
    if (inEdge.getTargetId() != getId()) {  
        throw new IllegalArgumentException("Destination id doesn't match the StreamNode id");  
    } else {  
	    // 添加到ArrayList对象中
        inEdges.add(inEdge);  
    }  
}

// 添加节点输出边
public void addOutEdge(StreamEdge outEdge) {  
	// 检查StreamNode的ID是否匹配
    if (outEdge.getSourceId() != getId()) {  
        throw new IllegalArgumentException("Source id doesn't match the StreamNode id");  
    } else {  
	    // 添加到ArrayList对象中
        outEdges.add(outEdge);  
    }  
}
/*---------------------------------------------------------*/

// 该枚举类定义了两个算子之间的数据交换方式
public enum ShuffleMode {  
	// 生产者和消费者同时在线。消费者立即收到生成的数据。这就流式处理。
    PIPELINED,  
    // 生产者首先生产其全部结果并完成。之后，消费者被启动并可以消费数据。这就是批处理模式。
    BATCH,  
    // 属于中间变量，该枚举值代表由框架自身来决定shuffle模式。
    // 在运行时只能是PIPELINED或者BATCH
	UNDEFINED  
}
```

稍微总结一下，**virtualSideOutputNodes**，**virtualSelectNodes** 和 **virtualPartitionNodes** 的处理逻辑。这几类 transformation 会被处理为虚拟节点。可以看出他们三者的共性是不需要用户传入自定义的处理逻辑，即 userDefinedFunction。虚拟节点严格来说不是 StreamNode 类型（尽管他们都是 StreamNode 对象），不包含物理转换逻辑。

虚拟节点不会出现在 StreamGraph 的处理流中，在添加 edge 的时候如果节点为虚拟节点，会通过递归的方式寻找上游节点，直至找到一个非虚拟节点，再执行添加 edge 逻辑。分区器的相关问题可以看我在【大数据】专栏中的文章，那里有详细的解释。

我们都知道 Flink 中算子的分类其实可以简单分为 Source、Transformation 和 Sink，上文已经分析了 Transformation 类算子转换为 StreamNode 的过程，接下来看一下 SourceTransformation 和 SinkTransformation 如何进行转换的。

作者：狗狗的数仓梦链接：https://juejin.cn/post/7195133794301509687来源：稀土掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

在 `StreamNode` 中，保存了对应的 `StreamOperator` (从 `StreamTransformation` 得到)，并且还引入了变量 `jobVertexClass` 来表示该节点在 `TaskManager` 中运行时的实际任务类型。

```java
private final Class<? extends AbstractInvokable> jobVertexClass;
```

`AbstractInvokable` 是所有可以在 `TaskManager` 中运行的任务的抽象基础类，包括流式任务和批任务。`StreamTask` 是所有流式任务的基础类，其具体的子类包括 `SourceStreamTask`, `OneInputStreamTask`, `TwoInputStreamTask` 等。

对于一些不包含物理转换操作的 `StreamTransformation`，如 Partitioning, split/select, union，并不会生成 `StreamNode`，而是生成一个带有特定属性的虚拟节点。当添加一条有虚拟节点指向下游节点的边时，会找到虚拟节点上游的物理节点，在两个物理节点之间添加边，并把虚拟转换操作的属性附着上去。

以 `PartitionTansformation` 为例， `PartitionTansformation` 是 `KeyedStream` 对应的转换：

```java
//StreamGraphGenerator#transformPartition
private <T> Collection<Integer> transformPartition(PartitionTransformation<T> partition) {
        StreamTransformation<T> input = partition.getInput();
        List<Integer> resultIds = new ArrayList<>();

        //递归地转换上游节点
        Collection<Integer> transformedIds = transform(input);

        for (Integer transformedId: transformedIds) {
            int virtualId = StreamTransformation.getNewNodeId();
            //添加虚拟的 Partition 节点
            streamGraph.addVirtualPartitionNode(transformedId, virtualId, partition.getPartitioner());
            resultIds.add(virtualId);
        }

        return resultIds;
    }

// StreamGraph#addVirtualPartitionNode
public void addVirtualPartitionNode(Integer originalId, Integer virtualId, StreamPartitioner<?> partitioner) {

        if (virtualPartitionNodes.containsKey(virtualId)) {
            throw new IllegalStateException("Already has virtual partition node with id " + virtualId);
        }

        //添加一个虚拟节点，后续添加边的时候会连接到实际的物理节点
        virtualPartitionNodes.put(virtualId,
                new Tuple2<Integer, StreamPartitioner<?>>(originalId, partitioner));
    }

```

前面提到，在每一个物理节点的转换上，会调用 `StreamGraph#addEdge` 在输入节点和当前节点之间建立边的连接：

```java
private void addEdgeInternal(Integer upStreamVertexID,
            Integer downStreamVertexID,
            int typeNumber,
            StreamPartitioner<?> partitioner,
            List<String> outputNames,
            OutputTag outputTag) {

        //先判断是不是虚拟节点上的边，如果是，则找到虚拟节点上游对应的物理节点
        //在两个物理节点之间添加边，并把对应的 StreamPartitioner,或者 OutputTag 等补充信息添加到StreamEdge中
        if (virtualSideOutputNodes.containsKey(upStreamVertexID)) {
            ......
        } else if (virtualPartitionNodes.containsKey(upStreamVertexID)) {
            int virtualId = upStreamVertexID;
            upStreamVertexID = virtualPartitionNodes.get(virtualId).f0;
            if (partitioner == null) {
                partitioner = virtualPartitionNodes.get(virtualId).f1;
            }
            addEdgeInternal(upStreamVertexID, downStreamVertexID, typeNumber, partitioner, outputNames, outputTag);
        } else {

            //两个物理节点
            StreamNode upstreamNode = getStreamNode(upStreamVertexID);
            StreamNode downstreamNode = getStreamNode(downStreamVertexID);

            // If no partitioner was specified and the parallelism of upstream and downstream
            // operator matches use forward partitioning, use rebalance otherwise.
            if (partitioner == null && upstreamNode.getParallelism() == downstreamNode.getParallelism()) {
                partitioner = new ForwardPartitioner<Object>();
            } else if (partitioner == null) {
                partitioner = new RebalancePartitioner<Object>();
            }

            if (partitioner instanceof ForwardPartitioner) {
                if (upstreamNode.getParallelism() != downstreamNode.getParallelism()) {
                    throw new UnsupportedOperationException("Forward partitioning does not allow " +
                            "change of parallelism. Upstream operation: " + upstreamNode + " parallelism: " + upstreamNode.getParallelism() +
                            ", downstream operation: " + downstreamNode + " parallelism: " + downstreamNode.getParallelism() +
                            " You must use another partitioning strategy, such as broadcast, rebalance, shuffle or global.");
                }
            }

            //创建 StreamEdge，保留了 StreamPartitioner 等属性
            StreamEdge edge = new StreamEdge(upstreamNode, downstreamNode, typeNumber, outputNames, partitioner, outputTag);

            //分别将StreamEdge添加到上游节点和下游节点
            getStreamNode(edge.getSourceId()).addOutEdge(edge);
            getStreamNode(edge.getTargetId()).addInEdge(edge);
        }
    }

```

这样通过 `StreamNode` 和 `SteamEdge`，就得到了 DAG 中的所有节点和边，以及它们之间的连接关系，拓扑结构也就建立了。

```java
/*------------------------ StreamGraph ---------------------------*/
// 用来存储Source
private Set<Integer> sources;  
// 用来存储Sink
private Set<Integer> sinks;

// 添加Source节点
public <IN, OUT> void addSource(  
        Integer vertexID,  
        @Nullable String slotSharingGroup,  
        @Nullable String coLocationGroup,  
        SourceOperatorFactory<OUT> operatorFactory,  
        TypeInformation<IN> inTypeInfo,  
        TypeInformation<OUT> outTypeInfo,  
        String operatorName) {  
    // Source算子的转换和Transformation算子没有什么大的区别 
    addOperator(  
            vertexID,  
            slotSharingGroup,  
            coLocationGroup,  
            operatorFactory,  
            inTypeInfo,  
            outTypeInfo,  
            operatorName,  
            SourceOperatorStreamTask.class);  
    // 唯一的区别在于要在sources集合中添加一下ID
    sources.add(vertexID);  
}

// 添加Sink节点
public <IN, OUT> void addSink(  
        Integer vertexID,  
        @Nullable String slotSharingGroup,  
        @Nullable String coLocationGroup,  
        StreamOperatorFactory<OUT> operatorFactory,  
        TypeInformation<IN> inTypeInfo,  
        TypeInformation<OUT> outTypeInfo,  
        String operatorName) {  
    // Sink算子的转换和Transformation算子没有什么大的区别 
    addOperator(  
            vertexID,  
            slotSharingGroup,  
            coLocationGroup,  
            operatorFactory,  
            inTypeInfo,  
            outTypeInfo,  
            operatorName);  
    if (operatorFactory instanceof OutputFormatOperatorFactory) {  
        setOutputFormat(  
                vertexID, ((OutputFormatOperatorFactory) operatorFactory).getOutputFormat());  
    }  
    // 唯一的区别在于要在sinks集合中添加一下ID
    sinks.add(vertexID);  
}

/*---------------------------------------------------------*/
```

## **小结**

首先，用户通过 DataStream API 编写程序，客户端根据编写的程序进行转换得到一系列**Transformation**对象，然后调用 `streamGraphGenerator.generate(env, transformations)` 构造出 **StreamGraph** 对象。构造 StreamGraph 对象的第一步是遍历 transformations 集合，并对其每一个 Transformation 对象调用 `transform()` 方法转换为 StreamNode 对象；接着通过构建 **StreamEdge** 对象进行上、下游 StreamNode 对象的连接，此处需要特别注意，对 **PartitionTransformation** 等不需要传递 UDF 的 Transformation 对象，都会将其添加到虚拟节点集合中，虽然都是 StreamNode 对象，但是不会构建真正的 StreamEdge；添加完边之后整个 StreamGraph 构造完毕。

- 参考：[https://juejin.cn/post/7195133794301509687?share_token=98d98856-daf4-47d0-b5d9-ba1dc0ba705f](https://juejin.cn/post/7195133794301509687?share_token=98d98856-daf4-47d0-b5d9-ba1dc0ba705f)

  [https://mp.weixin.qq.com/s/jNJgSsWa_kOrig52mvFFXg](https://mp.weixin.qq.com/s/jNJgSsWa_kOrig52mvFFXg)