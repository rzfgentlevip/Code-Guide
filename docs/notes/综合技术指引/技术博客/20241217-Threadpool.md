---
title: 动态线程池
icon: material-icon-theme:subtitles
order: 2
author: bugcode
date: 2020-01-01T00:00:00.000Z
category:
  - 分布式
  - LINUX
tag:
  - 分布式
  - linux
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/Threadpool/
---

# 动态线程池

## 什么是线程池

线程池（Thread Pool）是⼀种基于池化思想管理线程的⼯具，经常出现在多线程服务器中，如MySQL。

线程过多会带来额外的开销，其中包括创建销毁线程的开销、调度线程的开销等等，同时也降低了计算机的整体性能。线程池维护多个线程，等待监督管理者分配可并发执⾏的任务。这种做法，⼀⽅⾯避免了处理任务时创建销毁线程开销的代价，另⼀⽅⾯避免了线程数量膨胀导致的过分调度问题，保证了对内核的充分利⽤。

⽽本⽂描述线程池是JDK中提供的ThreadPoolExecutor类。

当然，使⽤线程池可以带来⼀系列好处：

1. 降低资源消耗：通过池化技术重复利⽤已创建的线程，降低线程创建和销毁造成的损耗。
2. 提⾼响应速度：任务到达时，⽆需等待线程创建即可⽴即执⾏。
3. 提⾼线程的可管理性：线程是稀缺资源，如果⽆限制创建，不仅会消耗系统资源，还会因为线程的不合理分布导致资源调度失衡，降低系统的稳定性。使⽤线程池可以进⾏统⼀的分配、调优和监控。
4. 提供更多更强⼤的功能：线程池具备可拓展性，允许开发⼈员向其中增加更多的功能。⽐如延时定时线程池
   ScheduledThreadPoolExecutor，就允许任务延期执⾏或定期执⾏。

## 线程池解决的问题是什么

线程池解决的核⼼问题就是**资源管理问题**。在并发环境下，系统不能够确定在任意时刻中，有多少任务需要执⾏，有多少资源需要投⼊。这种不确定性将带来以下若⼲问题：

1. 频繁申请，销毁资源和调度资源，将带来额外的消耗，可能会⾮常巨⼤。
2. 对资源⽆限申请缺少抑制⼿段，易引发系统资源耗尽的⻛险。
3. 系统⽆法合理管理内部的资源分布，会降低系统的稳定性。

为解决资源分配这个问题，线程池采⽤了“池化”（Pooling）思想。池化，顾名思义，是为了最⼤化收益并最⼩化⻛险，⽽将资源统⼀在⼀起管理的⼀种思想。

“池化”思想不仅仅能应⽤在计算机领域，在⾦融、设备、⼈员管理、⼯作管理等领域也有相关的应⽤。在计算机领域中的表现为：统⼀管理IT资源，包括服务器、存储、和⽹络资源等等。通过共享资源，使⽤户在低投⼊中获益。除去线程池，还有其他⽐较典型的⼏种使⽤策略包括：

1. 内存池(Memory Pooling)：预先申请内存，提升申请内存速度，减少内存碎⽚。
2. 连接池(Connection Pooling)：预先申请数据库连接，提升申请连接的速度，降低系统的开销。
3. 实例池(Object Pooling)：循环使⽤对象，减少资源在初始化和释放时的昂贵损耗。

在了解完“是什么”和“为什么”之后，下⾯我们来⼀起深⼊⼀下线程池的内部实现原理。

## 线程池的核心设计和实现

在前⽂中，我们了解到：线程池是⼀种通过“池化”思想，帮助我们管理线程⽽获取并发性的⼯具，在Java中的体现
是ThreadPoolExecutor类。那么它的的详细设计与实现是什么样的呢？我们会在本章进⾏详细介绍。

> 为什么不允许使用Executors创建线程池?
>
> Executors返回的线程池对象有如下弊端:
>
> 1. FixedThreadPool和SingleThreadPool允许请求的队列长度是Integer.MAX_VALUE,可能造成堆积大量的请求，导致OOM；
> 2. CacheThreadPool允许创建线程数量为Integer.MAX_VALUE,可能会创建大量的线程，导致OOM；

### 总体设计

Java中的线程池核⼼实现类是ThreadPoolExecutor，本章基于JDK 1.8的源码来分析Java线程池的核⼼设计与实现。我们⾸先来看⼀下ThreadPoolExecutor的UML类图，了解下ThreadPoolExecutor的继承关系。

![image-20241218151031908](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241218151031908.png)

ThreadPoolExecutor实现的顶层接⼝是Executor，顶层接⼝Executor提供了⼀种思想：**将任务提交和任务执⾏进⾏解耦**。⽤户⽆需关注如何创建线程，如何调度线程来执⾏任务，⽤户只需提供Runnable对象，将任务的运⾏逻辑提交到执⾏器(Executor)中，由Executor框架完成线程的调配和任务的执⾏部分。

具体来看下顶层的接口内容，就有一个execute方法，设计的非常核心，然后其子接口在对其进行扩展和封装；

```java
public interface Executor {

    void execute(Runnable command);
}
```

ExecutorService接⼝增加了⼀些能⼒：

（1）扩充执⾏任务的能⼒，补充可以为⼀个或⼀批异步任务⽣成Future的⽅法；

（2）提供了管控线程池的⽅法，⽐如停⽌线程池的运⾏。

ExecutorService接口继承子Executor接口，在其基础上对其功能进行扩充，包括判断线程池是否关闭，是否终止，提交任务等方法；

```java
public interface ExecutorService extends Executor {

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

AbstractExecutorService则是上层的抽象类，将执⾏任务的流程串联了起来，保证下层的实现只需关注⼀个执⾏任务的⽅法即可。最下层的实现类ThreadPoolExecutor实现最复杂的运⾏部分，

ThreadPoolExecutor将会⼀⽅⾯维护⾃身的⽣命周期，另⼀⽅⾯同时管理线程和任务，使两者良好的结合从⽽执⾏并⾏任务。

ThreadPoolExecutor是如何运⾏，如何同时维护线程和执⾏任务的呢？其运⾏机制如下图所示：

![image-20241217203330928](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217203330928.png)

线程池在内部实际上构建了⼀个⽣产者消费者模型，将线程和任务两者解耦，并不直接关联，从⽽良好的缓冲任务，复⽤线程。线程池的运⾏主要分成两部分：**任务管理、线程管理**。

在ThreadPoolExecutor线程池内部，使用`BlockingQueue<Runnable> workQueue`阻塞队列来暂存任务；

**任务管理**部分充当⽣产者的⻆⾊，当任务提交后，线程池会判断该任务后续的流转：

（1）直接申请线程执⾏该任务；

（2）缓冲到队列中等待线程执⾏；

（3）拒绝该任务。

**线程管理**部分是消费者，它们被统⼀维护在线程池内，根据任务请求进⾏线程的分配，当线程执⾏完任务后则会继续获取新的任务去执⾏，最终当线程获取不到任务的时候，线程就会被回收。

接下来，我们会按照以下三个部分去详细讲解线程池运⾏机制：

1. 线程池如何维护⾃身状态。
2. 线程池如何管理任务，收到Runnable任务后如何做。
3. 线程池如何管理线程,消费到一个task之后如何做。

### 生命周期管理

线程池运⾏的状态，并不是⽤户显式设置的，⽽是伴随着线程池的运⾏，由内部来维护。线程池内部使⽤⼀个变量维护两个值：**运⾏状态(runState)和线程数量 (workerCount)**。在具体实现中，线程池将运⾏状态(runState)、线程数量 (workerCount)两个关键参数的维护放在了⼀起，如下代码所示：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

ctl 这个AtomicInteger类型，是对线程池的运⾏状态和线程池中有效线程的数量进⾏控制的⼀个字段， 它同时包含两部分的信息：线程池的运⾏状态 (runState) 和线程池内有效线程的数量 (workerCount)，⾼3位保存runState，低29位保存workerCount，两个变量之间互不⼲扰。

⽤⼀个变量去存储两个值，可避免在做相关决策时，出现不⼀致的情况，不必为了维护两者的⼀致，⽽占⽤锁资源。通过阅读线程池源代码也可以发现，经常出现要同时判断线程池运⾏状态和线程数量的情况。线程池也提供了若⼲⽅法去供⽤户获得线程池当前的运⾏状态、线程个数。这⾥都使⽤的是位运算的⽅式，相⽐于基本运算，速度也会快很多。

关于内部封装的获取⽣命周期状态、获取线程池线程数量的计算⽅法如以下代码所示：

```java
private static int runStateOf(int c)     { return c & ~CAPACITY; } //计算当前运⾏状态
private static int workerCountOf(int c)  { return c & CAPACITY; } //计算当前线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }
//通过状态和线程数⽣成ctl
```

ThreadPoolExecutor的运⾏状态有5种，分别为：

| 线程状态   | 状态说明                                                     |
| ---------- | ------------------------------------------------------------ |
| RUNNING    | 线程池被创建后的初始状态，能接受新提交的任务，并且也能处理阻塞队列中的任务 |
| SHUTDOWN   | 关闭状态，不再接受新提交的任务，但仍可以继续处理已进入阻塞队列中的任务 |
| STOP       | 会中断正在处理任务的线程，不能再接受新任务，也不继续处理队列中的任务 |
| TIDYING    | 所有的任务都已终止，workerCount(有效工作线程数)为0           |
| TERMINATED | 线程池彻底终止运行                                           |

其⽣命周期转换如下⼊所示：

![image-20241217203718315](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217203718315.png)

> 注：要区分线程池的状态和线程的状态，下图为线程池的状态；
>
> 当线程调用`start()`，线程在JVM中不一定立即执行，有可能要等待操作系统分配资源，此时为`READY`状态，当线程获得资源时进入`RUNNING`状态，才会真正开始执行。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241218091635271.png)

### 任务执行机制

#### 任务调度

任务调度是线程池的主要⼊⼝，当⽤户提交了⼀个任务，接下来这个任务将如何执⾏都是由这个阶段决定的。了解这部分就相当于了解了线程池的核⼼运⾏机制。

⾸先，所有任务的调度都是由execute⽅法完成的，这部分完成的⼯作是：检查现在线程池的运⾏状态、运⾏线程数、运⾏策略，决定接下来执⾏的流程，是直接申请线程执⾏，或是缓冲到队列中执⾏，亦或是直接拒绝该任务。

```java
 public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
    	//获取当前正在执行的线程个数
        int c = ctl.get();
     	// 判断线程个数和核心线程数量
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }

```

其执⾏过程如下：

1. ⾸先检测线程池运⾏状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执⾏任务。
2. 如果`workerCount < corePoolSize`，则创建并启动⼀个线程来执⾏新提交的任务。
3. 如果`workerCount >= corePoolSize`，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果`workerCount >= corePoolSize && workerCount < maximumPoolSize`，且线程池内的阻塞队列已满，
   则创建并启动⼀个线程来执⾏新提交的任务。
5. 如果`workerCount >= maximumPoolSize`，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默
   认的处理⽅式是直接抛异常。

```java
private static int runStateOf(int c)     { return c & ~CAPACITY; } //计算当前运⾏状态
private static int workerCountOf(int c)  { return c & CAPACITY; } //计算当前线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }
//通过状态和线程数⽣成ctl
```

其执⾏流程如下图所示：

![image-20241217203928945](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217203928945.png)

#### 任务缓冲

任务缓冲模块是线程池能够管理任务的核⼼部分。线程池的本质是对**任务和线程**的管理，⽽做到这⼀点最关键的思想就是将任务和线程两者解耦，不让两者直接关联，才可以做后续的分配⼯作。线程池中是以⽣产者消费者模式，通过⼀个阻塞队列来实现的。阻塞队列缓存任务，⼯作线程从阻塞队列中获取任务。

阻塞队列(BlockingQueue)是⼀个⽀持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为⾮空。当队列满时，存储元素的线程会等待队列可⽤。阻塞队列常⽤于⽣产者和消费者的场景，⽣产者是往队列⾥添加元素的线程，消费者是从队列⾥拿元素的线程。阻塞队列就是⽣产者存放元素的容器，⽽消费者也只从容器⾥拿元素。

下图中展示了线程1往阻塞队列中添加元素，⽽线程2从阻塞队列中移除元素：

![image-20241217204037189](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204037189.png)

使⽤不同的队列可以实现不⼀样的任务存取策略。在这⾥，我们可以再介绍下阻塞队列的成员：

![image-20241217204058496](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204058496.png)

#### 任务申请

由上⽂的任务分配部分可知，任务的执⾏有两种可能：⼀种是任务直接由新创建的线程执⾏。另⼀种是线程从任务队列中获取任务然后执⾏，执⾏完任务的空闲线程会再次去从队列中申请任务再去执⾏。第⼀种情况仅出现在线程初始创建的时候，第⼆种是线程获取任务绝⼤多数的情况。

```java
 private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            //获取正在执行的线程个数
            int c = ctl.get();
            //获取线程池状态
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
			//判断线程的数量
            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
                //从队列中获取一个线程
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }

```

线程需要从任务缓存模块中不断地取任务执⾏，帮助线程从阻塞队列中获取任务，实现线程管理模块和任务管理模块之间的通信。这部分策略由getTask⽅法实现，其执⾏流程如下图所示：

![image-20241217204143755](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204143755.png)

getTask这部分进⾏了多次判断，为的是控制线程的数量，使其符合线程池的状态。如果线程池现在不应该持有那么多线程，则会返回null值。⼯作线程Worker会不断接收新任务去执⾏，⽽当⼯作线程Worker接收不到任务的时候，就会开始被回收。

#### 任务拒绝

任务拒绝模块是线程池的保护部分，线程池有⼀个最⼤的容量，当线程池的任务缓存队列已满，并且线程池中的线程数⽬达到maximumPoolSize时，就需要拒绝掉该任务，采取任务拒绝策略，保护线程池。

拒绝策略是⼀个接⼝，其设计如下：

```java
public interface RejectedExecutionHandler {
	void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

⽤户可以通过实现这个接⼝去定制拒绝策略，也可以选择JDK提供的四种已有拒绝策略，其特点如下：

![image-20241217204328076](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204328076.png)

#### 任务执行

```java
// 使用原子操作类AtomicInteger的ctl变量，前3位记录线程池的状态，后29位记录线程数
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
// Integer的范围为[-2^31,2^31 -1], Integer.SIZE-3 =32-3= 29，用来辅助左移位运算
private static final int COUNT_BITS = Integer.SIZE - 3;
// 高三位用来存储线程池运行状态，其余位数表示线程池的容量
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// 线程池状态以常量值被存储在高三位中
private static final int RUNNING    = -1 << COUNT_BITS; // 线程池接受新任务并会处理阻塞队列中的任务
private static final int SHUTDOWN   =  0 << COUNT_BITS; // 线程池不接受新任务，但会处理阻塞队列中的任务
private static final int STOP       =  1 << COUNT_BITS; // 线程池不接受新的任务且不会处理阻塞队列中的任务，并且会中断正在执行的任务
private static final int TIDYING    =  2 << COUNT_BITS; // 所有任务都执行完成，且工作线程数为0，将调用terminated方法
private static final int TERMINATED =  3 << COUNT_BITS; // 最终状态，为执行terminated()方法后的状态

// ctl变量的封箱拆箱相关的方法
private static int runStateOf(int c)     { return c & ~CAPACITY; } // 获取线程池运行状态
private static int workerCountOf(int c)  { return c & CAPACITY; } // 获取线程池运行线程数
private static int ctlOf(int rs, int wc) { return rs | wc; } // 获取ctl对象
```

通过执行`execute`方法 该方法无返回值，为`ThreadPoolExecutor`自带方法，传入`Runnable`类型对象;

```java
//参数是Runnable对象
public void execute(Runnable command) {
        if (command == null)// 任务为空，抛出NPE
            throw new NullPointerException();
   		//获取正在执行的线程个数，如果小于核心线程个数，就封装当前任务为Worker提交任务
        int c = ctl.get(); // 获取当前工作线程数和线程池运行状态（共32位，前3位为运行状态，后29位为运行线程数）
        if (workerCountOf(c) < corePoolSize) {// 如果当前工作线程数小于核心线程数
            if (addWorker(command, true))// 在addWorker中创建工作线程并执行任务
                return;
            c = ctl.get();
        }
       // 核心线程数已满（工作线程数>核心线程数）才会走下面的逻辑
        if (isRunning(c) && workQueue.offer(command)) {// 如果当前线程池状态为RUNNING，并且任务成功添加到阻塞队列
            int recheck = ctl.get();// 双重检查，因为从上次检查到进入此方法，线程池可能已成为SHUTDOWN状态
            if (! isRunning(recheck) && remove(command))// 如果当前线程池状态不是RUNNING则从队列删除任务
                reject(command); // 执行拒绝策略
            else if (workerCountOf(recheck) == 0)// 当线程池中的workerCount为0时，此时workQueue中还有待执行的任务，则新增一个addWorker，消费workqueue中的任务
                addWorker(null, false);
        }
   // 阻塞队列已满才会走下面的逻辑
        else if (!addWorker(command, false))
            reject(command); // 如果当前线程池为SHUTDOWN状态或者线程池已饱和
    }
```

当通过线程池对象执行一个任务时，线程池对象根据策略执行执行任务或者是增加到队列中，如果当前执行的任务小于核心线程数，就会新创建一个Worker线程然后执行任务；如果正在执行的任务个数大于核心线程数量，会将当前任务新增到阻塞队列中;等线程池中执行的线程小于核心线程数量后，Worker执行线程从阻塞队列中获取任务执行;

### Worker线程管理

#### Worker线程

线程池为了掌握线程的状态并维护线程的⽣命周期，设计了线程池内的⼯作线程Worker。我们来看⼀下它的部分代码：

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
   // 该worker正在运行的线程
    final Thread thread;
    
    // 将要运行的初始任务
    Runnable firstTask;
    
    // 每个线程的任务计数器
    volatile long completedTasks;

    // 构造方法   
    Worker(Runnable firstTask) {
        setState(-1); // 调用runWorker()前禁止中断
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this); // 通过ThreadFactory创建一个线程
    }

    // 实现了Runnable接口的run方法
    public void run() {
        runWorker(this);
    }
    
    ... // 此处省略了其他方法
}
```

Worker这个⼯作线程，实现了Runnable接⼝，在调用start方法时其实调用的就是run方法,并持有⼀个线程thread，⼀个初始化的任务firstTask。thread是在调⽤构造⽅法时通过ThreadFactory来创建的线程，可以⽤来执⾏任务；firstTask⽤它来保存传⼊的第⼀个任务，这个任务可以有也可以为null。如果这个值是⾮空的，那么线程就会在启动初期⽴即执⾏这个任务，也就对应核⼼线程创建时的情况；如果这个值是null，

那么就需要创建⼀个线程去执⾏任务列表（workQueue）中的任务，也就是⾮核⼼线程的创建。

Worker执⾏任务的模型如下图所示：

![image-20241217204449853](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204449853.png)

线程池需要管理线程的⽣命周期，需要在线程⻓时间不运⾏的时候进⾏回收。线程池使⽤⼀张Hash表去持有线程的引⽤，这样可以通过添加引⽤、移除引⽤这样的操作来控制线程的⽣命周期。这个时候重要的就是如何判断线程是否在运⾏。

Worker是通过继承AQS，使⽤AQS来实现独占锁这个功能。没有使⽤可重⼊锁ReentrantLock，⽽是使⽤AQS，为的就是实现不可重⼊的特性去反应线程现在的执⾏状态。

1、lock⽅法⼀旦获取了独占锁，表示当前线程正在执⾏任务中。 2、如果正在执⾏任务，则不应该中断线程。 3、如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进⾏中断。 4、线程池在执⾏shutdown⽅法或tryTerminate⽅法时会调⽤interruptIdleWorkers⽅法来中断空闲的线程，interruptIdleWorkers⽅法会使⽤tryLock⽅法来判断线程池中的线程是否是空闲状态；如果线程是空闲状态则可以安全回收。

在线程回收过程中就使⽤到了这种特性，回收过程如下图所示：

![image-20241217204512477](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204512477.png)

#### Worker线程增加

增加线程是通过线程池中的addWorker⽅法，该⽅法的功能就是增加⼀个线程，该⽅法不考虑线程池是在哪个阶段增加的该线程，这个分配线程的策略是在上个步骤完成的，该步骤仅仅完成增加线程，并使它运⾏，最后返回是否成功这个结果。addWorker⽅法有两个参数：firstTask、core。

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry: // 循环退出标志位
    for (;;) { // 无限循环
        int c = ctl.get();
        int rs = runStateOf(c); // 线程池状态

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && 
            ! (rs == SHUTDOWN && firstTask == null && ! workQueue.isEmpty()) // 换成更直观的条件语句
            // (rs != SHUTDOWN || firstTask != null || workQueue.isEmpty())
           )
           // 返回false的条件就可以分解为：
           //（1）线程池状态为STOP，TIDYING，TERMINATED
           //（2）线程池状态为SHUTDOWN，且要执行的任务不为空
           //（3）线程池状态为SHUTDOWN，且任务队列为空
            return false;

        // cas自旋增加线程个数
        for (;;) {
            int wc = workerCountOf(c); // 当前工作线程数
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize)) // 工作线程数>=线程池容量 || 工作线程数>=(核心线程数||最大线程数)
                return false;
            if (compareAndIncrementWorkerCount(c)) // 执行cas操作，添加线程个数
                break retry; // 添加成功，退出外层循环
            // 通过cas添加失败
            c = ctl.get();  
            // 线程池状态是否变化，变化则跳到外层循环重试重新获取线程池状态，否者内层循环重新cas
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    // 简单总结上面的CAS过程：
    //（1）内层循环作用是使用cas增加线程个数，如果线程个数超限则返回false，否者进行cas
    //（2）cas成功则退出双循环，否者cas失败了，要看当前线程池的状态是否变化了
    //（3）如果变了，则重新进入外层循环重新获取线程池状态，否者重新进入内层循环继续进行cas

    // 走到这里说明cas成功，线程数+1，但并未被执行
    boolean workerStarted = false; // 工作线程调用start()方法标志
    boolean workerAdded = false; // 工作线程被添加标志
    Worker w = null;
    try {
        w = new Worker(firstTask); // 创建工作线程实例
        final Thread t = w.thread; // 获取工作线程持有的线程实例
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock; // 使用全局可重入锁
            mainLock.lock(); // 加锁，控制并发
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get()); // 获取当前线程池状态

                // 线程池状态为RUNNING或者（线程池状态为SHUTDOWN并且没有新任务时）
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // 检查线程是否处于活跃状态
                        throw new IllegalThreadStateException();
                    workers.add(w); // 线程加入到存放工作线程的HashSet容器，workers全局唯一并被mainLock持有
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock(); // finally块中释放锁
            }
            if (workerAdded) { // 线程添加成功
                t.start(); // 调用线程的start()方法
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted) // 如果线程启动失败，则执行addWorkerFailed方法
            addWorkerFailed(w);
    }
    return workerStarted;
}
```



firstTask参数⽤于指定新增的线程执⾏的第⼀个任务，该参数可以为空；core参数为true表示在新增线程时会判断当前活动线程数是否少于corePoolSize，false表示新增线程前需要判断当前活动线程数是否少于maximumPoolSize，其执⾏流程如下图所示：

![image-20241217204545364](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204545364.png)

#### Worker线程回收

线程池中线程的销毁依赖JVM⾃动的回收，线程池做的⼯作是根据当前线程池的状态维护⼀定数量的线程引⽤，防⽌这部分线程被JVM回收，当线程池决定哪些线程需要回收时，只需要将其引⽤消除即可。

Worker被创建出来后，就会不断地进⾏轮询，然后获取任务去执⾏，核⼼线程可以⽆限等待获取任务，⾮核⼼线程要限时获取任务。当Worker⽆法获取到任务，也就是获取的任务为空时，循环会结束，Worker会主动消除⾃身在线程池内的引⽤。

```java
try {
while (task != null || (task = getTask()) != null) {
//执⾏任务
}
} finally {
processWorkerExit(w, completedAbruptly);//获取不到任务时，主动回收⾃⼰
}
```

线程回收的⼯作是在processWorkerExit⽅法完成的。

![image-20241217204623039](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204623039.png)

事实上，在这个⽅法中，将线程引⽤移出线程池就已经结束了线程销毁的部分。但由于引起线程销毁的可能性有很多，线程池还要判断是什么引发了这次销毁，是否要改变线程池的现阶段状态，是否要根据新状态，重新分配线程。

#### Worker线程执⾏任务

在Worker类中的run⽅法调⽤了runWorker⽅法来执⾏任务，runWorker⽅法的执⾏过程如下：
1、while循环不断地通过getTask()⽅法获取任务。

 2、getTask()⽅法从阻塞队列中取任务。

 3、如果线程池正在停⽌，那么要保证当前线程是中断状态，否则要保证当前线程不是中断状态。

 4、执⾏任务。

 5、如果getTask结果为null则跳出循环，执⾏processWorkerExit()⽅法，销毁线程。

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask; // 获取工作线程中用来执行任务的线程实例
    w.firstTask = null;
    w.unlock(); // status设置为0，允许中断
    boolean completedAbruptly = true; // 线程意外终止标志
    try {
        // 如果当前任务不为空，则直接执行；否则调用getTask()从任务队列中取出一个任务执行
        while (task != null || (task = getTask()) != null) {
            w.lock(); // 加锁，保证下方临界区代码的线程安全
            // 如果状态值大于等于STOP且当前线程还没有被中断，则主动中断线程
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt(); // 中断当前线程
            try {
                beforeExecute(wt, task); // 任务执行前的回调，空实现，可以在子类中自定义
                Throwable thrown = null;
                try {
                    task.run(); // 执行线程的run方法
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown); // 任务执行后的回调，空实现，可以在子类中自定义
                }
            } finally {
                task = null; // 将循环变量task设置为null，表示已处理完成
                w.completedTasks++; // 当前已完成的任务数+1
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

执⾏流程如下图所示：

![image-20241217204649108](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241217204649108.png)

#### 从任务队列中取出一个任务

```java
private Runnable getTask() {
    boolean timedOut = false; // 通过timeOut变量表示线程是否空闲时间超时了
    // 无限循环
    for (;;) {
        int c = ctl.get(); // 线程池信息
        int rs = runStateOf(c); // 线程池当前状态

        // 如果线程池状态>=SHUTDOWN并且工作队列为空 或 线程池状态>=STOP，则返回null，让当前worker被销毁
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount(); // 工作线程数-1
            return null;
        }

        int wc = workerCountOf(c); // 获取当前线程池的工作线程数

        // 当前线程是否允许超时销毁的标志
        // 允许超时销毁：当线程池允许核心线程超时 或 工作线程数>核心线程数
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // 如果(当前线程数大于最大线程数 或 (允许超时销毁 且 当前发生了空闲时间超时))
        // 且(当前线程数大于1 或 阻塞队列为空)
        // 则减少worker计数并返回null
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 根据线程是否允许超时判断用poll还是take（会阻塞）方法从任务队列头部取出一个任务
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r; // 返回从队列中取出的任务
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

总结一下哪些情况getTask()会返回null：

> 1. 线程池状态为SHUTDOWN且任务队列为空
> 2. 线程池状态为STOP、TIDYING、TERMINATED
> 3. 线程池线程数大于最大线程数
> 4. 线程可以被超时回收的情况下等待新任务超时

#### 工作线程退出

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    // 如果completedAbruptly为true则表示任务执行过程中抛出了未处理的异常
    // 所以还没有正确地减少worker计数，这里需要减少一次worker计数
    if (completedAbruptly) 
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 把将被销毁的线程已完成的任务数累加到线程池的完成任务总数上
        completedTaskCount += w.completedTasks;
        workers.remove(w); // 从工作线程集合中移除该工作线程
    } finally {
        mainLock.unlock();
    }

    // 尝试结束线程池
    tryTerminate();

    int c = ctl.get();
    // 如果是RUNNING 或 SHUTDOWN状态
    if (runStateLessThan(c, STOP)) {
        // worker是正常执行完
        if (!completedAbruptly) {
            // 如果允许核心线程超时则最小线程数是0，否则最小线程数等于核心线程数
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            // 如果阻塞队列非空，则至少要有一个线程继续执行剩下的任务
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            // 如果当前线程数已经满足最小线程数要求，则不需要再创建替代线程
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        // 重新创建一个worker来代替被销毁的线程
        addWorker(null, false);
    }
}
```



## 线程池在业务中的实践

##