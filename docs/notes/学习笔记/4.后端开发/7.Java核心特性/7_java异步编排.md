---
title: 7、java异步编排
icon: material-icon-theme:bitbucket
createTime: 2026/05/18 15:25:52
permalink: /learning-notes/backend/java/java异步编排/
order: 7
---


## 异步编排


### 创建线程的方法

初始化线程的4种方式：

1. 继承Thread
2. 实现Runnable接口
3. 实现Callable接口 + FutureTask （可以拿到返回结果，可以处理异常）
4. 线程池

方式1和方式2：主进程无法获取线程的运算结果。不适合当前场景

方式3：主进程可以获取线程的运算结果，并设置给itemVO，但是不利于控制服务器中的线程资源。可以导致服务器资源耗尽。

方式4：通过如下两种方式初始化线程池：

```java
Executors.newFiexedThreadPool(3);
//或者
new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit unit, workQueue, threadFactory, handler);
```

通过线程池性能稳定，也可以获取执行结果，并捕获异常。但是，在业务复杂情况下，一个异步调用可能会依赖于另一个异步调用的执行结果。



### CompletableFuture介绍

Future是Java 5添加的类，用来描述一个异步计算的结果。你可以使用`isDone`方法检查计算是否完成，或者使用`get`阻塞住调用线程，直到计算完成返回结果，你也可以使用`cancel`方法停止任务的执行。

虽然`Future`以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的CPU资源，而且也不能及时地得到计算结果，为什么不能用观察者设计模式当计算结果完成及时通知监听者呢？

很多语言，比如Node.js，采用回调的方式实现异步编程。Java的一些框架，比如Netty，自己扩展了Java的 `Future`接口，提供了`addListener`等多个扩展方法；Google guava也提供了通用的扩展Future；Scala也提供了简单易用且功能强大的Future/Promise异步编程模式。

作为正统的Java类库，是不是应该做点什么，加强一下自身库的功能呢？

在Java 8中, 新增加了一个包含50个方法左右的类: CompletableFuture，提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，提供了函数式编程的能力，可以通过回调的方式处理计算结果，并且提供了转换和组合CompletableFuture的方法。

CompletableFuture类实现了Future接口，所以你还是可以像以前一样通过`get`方法阻塞或者轮询的方式获得结果，但是这种方式不推荐使用。


### 异步编排演进

```text
Java 1.0 ──► Thread + Runnable（基础线程）
    │
Java 5 ────► ExecutorService + Future（线程池）
    │
Java 7 ────► Fork/Join Framework（分治框架）
    │
Java 8 ────► CompletableFuture（响应式编程）
    │
Java 9 ────► Flow API（响应式流）
    │
Java 11 ───► HttpClient（异步HTTP）
    │
Java 19 ───► Virtual Threads（虚拟线程）
```


### 创建异步对象

CompletableFuture 提供了四个静态方法来创建一个异步操作。

```java
static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

**没有指定Executor的方法会使用ForkJoinPool.commonPool() 作为它的线程池执行异步代码**。如果指定线程池，则使用指定的线程池运行。以下所有的方法都类同。

- runAsync方法不支持返回值。
- supplyAsync可以支持返回值。


### 计算完成时回调方法

当CompletableFuture的计算结果完成，或者抛出异常的时候，可以执行特定的Action。主要是下面的方法：

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action);
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action);
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor);

public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn);
```

whenComplete可以处理正常和异常的计算结果，用于接收带有返回值的CompletableFuture对象，无法修改返回值。exceptionally处理异常情况，只要异步线程中有抛出异常，则进入该方法，修改返回值。BiConsumer<? super T,? super Throwable>可以定义处理业务

CompletableFuture.handle()：用于处理返回结果，可以接收返回值和异常，可以对返回值进行修改。


whenComplete 和 whenCompleteAsync 的区别：
- whenComplete：是执行当前任务的线程执行继续执行 whenComplete 的任务。
- whenCompleteAsync：是执行把 whenCompleteAsync 这个任务继续提交给线程池来进行执行。

**方法不以Async结尾，意味着Action使用相同的线程执行，而Async可能会使用其他线程执行（如果是使用相同的线程池，也可能会被同一个线程选中执行）**

```java
public class TestAsync {

    public static void main(String[] args) {

        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                10,
                50,
                60,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(100000),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());


        // 异步起线程执行业务 无返回值
        CompletableFuture<Void> voidCompletableFuture = CompletableFuture.runAsync(() -> {
            System.out.println("当前线程：" + Thread.currentThread().getId());
            int i = 10 / 2;
            System.out.println("运行结果：" + i);
        }, executor);


        //异步起线程执行业务 有返回值
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("当前线程：" + Thread.currentThread().getId());
            int i = 10 / 0;
            System.out.println("运行结果：" + i);
            return i;
        }, executor).whenComplete((res,exc)->{
            // 可以接收到返回值和异常类型，但是无法处理异常
            System.out.println("异步任务成功完成了...结果是：" + res + ";异常是：" + exc);
        }).exceptionally(throwable -> {
            // 处理异常，返回一个自定义的值，和上边返回值无关。
            return 10;
        });

        //方法执行完成后的处理
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("当前线程：" + Thread.currentThread().getId());
            int i = 10 / 0;
            System.out.println("运行结果：" + i);
            return i;
        }, executor).handle((res,thr)->{
            // 无论线程是否正确执行，都会执行这里，可以对返回值进行操作。
            if(res != null){
                return res * 2;
            }
            if(thr != null){
                return 0;
            }
            return 0;
        });
    }
}
```


### 线程串行化方法

```java
// 使线程串行执行，无入参，无返回值
public CompletableFuture<Void> thenRun(Runnable action);
public CompletableFuture<Void> thenRunAsync(Runnable action);
public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor);

// 使线程串行执行，有入参，无返回值
public CompletableFuture<Void> thenAccept(Consumer<? super T> action);
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor);

// 使线程串行执行，有入参，有返回值
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn);
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn);
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor);
```

thenApply 方法：当一个线程依赖另一个线程时，获取上一个任务返回的结果，并返回当前任务的返回值。

thenAccept方法：消费处理结果。接收任务的处理结果，并消费处理，无返回结果。

thenRun方法：只要上面的任务执行完成，就开始执行thenRun，只是处理完任务后，执行 thenRun的后续操作

带有Async默认是异步执行的。这里所谓的异步指的是不在当前线程内执行。

不以Async结尾的方法，都是在执行串行的时候，使用执行上一个方法的线程，也就是说从头串行到最后一个任务，使用的是同一个线程。

而以Async结尾的方法，每串行一个方法，都会使用一个新线程。下文不再解释。

```java
Function<? super T,? extends U>
		T：上一个任务返回结果的类型
		U：当前任务的返回值类型
```

**案例**

```java
public class TestOrder {


    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                10,
                50,
                60,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(100000),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());

        // 两个任务串行执行，任务2不用任务1的返回值，并且任务2无返回值。
        CompletableFuture<Void> f1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务1：当前线程：" + Thread.currentThread().getId());
            int i = 10 / 4;
            System.out.println("任务1：运行结果：" + i);
            return i;
        }, executor).thenRunAsync(() -> {
            System.out.println("任务2启动了");
        }, executor);


        // 两个任务串行执行，任务2要使用任务1的返回值，并且任务2无返回值。
        CompletableFuture<Void> f2 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务1：当前线程：" + Thread.currentThread().getId());
            int i = 10 / 4;
            System.out.println("任务1：运行结果：" + i);
            return i;
        }, executor).thenAcceptAsync(res -> {
            System.out.println("任务2启动了" + res);
        }, executor);

        // 两个任务串行执行，任务2要使用任务1的返回值，并且返回任务2的返回值。
        CompletableFuture<String> f3 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务1：当前线程：" + Thread.currentThread().getId());
            int i = 10 / 4;
            System.out.println("任务1：运行结果：" + i);
            return i;
        }, executor).thenApplyAsync(res -> {
            System.out.println("任务2启动了" + res);
            return "Hello" + res;
        }, executor);
        // 有返回值时，需要使用CompletableFuture.get()方法，等待异步线程执行结束，从而获取到异步线程的返回值。
        String s = f3.get();
        System.out.println(s);

    }
}
```

### 两任务并行执行完成，再执行新任务

解释：两任务并行执行，并且都执行完成之后，串行另一个任务，也就是说在两个任务执行并行执行完成之后，需要再执行另一个任务。

```java
// 线程并行执行完成，并且执行新任务action，新任务无入参，无返回值
public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action, Executor executor);

// 线程并行执行完成，并且执行新任务action，新任务有入参，无返回值
public <U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action);
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action);
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action, Executor executor);

// 线程并行执行完成，并且执行新任务action，新任务有入参，有返回值
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor);

```

**java案例**

```java
// 任务1
CompletableFuture<Integer> future01 = CompletableFuture.supplyAsync(() -> {
    System.out.println("任务1线程：" + Thread.currentThread().getId());
    int i = 10 / 4;
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("任务1结束：");

    return i;
}, executor);

// 任务2
CompletableFuture<String> future02 = CompletableFuture.supplyAsync(() -> {
    System.out.println("任务2线程：" + Thread.currentThread().getId());
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("任务2结束");

    return "Hello";
}, executor);

// 任务1和任务2都完成，在不使用任务1和任务2的返回值情况下执行任务3，并且任务3没有返回值
CompletableFuture<Void> future1 = future01.runAfterBothAsync(future02,
                                                             () -> System.out.println("任务3开始"), executor);

// 任务1和任务2都完成，使用任务1和任务2的返回值情况下执行任务3，并且任务3没有返回值
CompletableFuture<Void> future2 = future01.thenAcceptBothAsync(future02,
                                                               (f1, f2) -> System.out.println("任务3开始，之前的结果" + f1 + "-->" + f2),
                                                               executor);

// 任务1和任务2都完成，使用任务1和任务2的返回值情况下执行任务3，并且任务3有返回值
CompletableFuture<String> future3 = future01.thenCombineAsync(future02,
                                                              (f1, f2) -> f1 + ":" + f2 + "->haha", 
                                                              executor);
String str = future3.get();
System.out.println(str);
```

### 两任务并行执行，其中一个执行完，就执行新任务。

解释：两任务并行执行，只要其中有一个执行完，就开始执行新任务。

```java
// 任务并行执行，只要其中有一个执行完，就开始执行新任务action，新任务无入参，无返回值
public CompletableFuture<Void> runAfterEither(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action, Executor executor);

// 任务并行执行，只要其中有一个执行完，就开始执行新任务action，新任务有入参（入参类型为Object，因为不确定是哪个任务先执行完成），无返回值
public CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action);
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action);
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action,Executor executor);

// 任务并行执行，只要其中有一个执行完，就开始执行新任务action，新任务有入参（入参类型为Object，因为不确定是哪个任务先执行完成），有返回值
public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn);
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn);
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn,Executor executor);

```

**java代码案例**

```java
// 任务1
CompletableFuture<Integer> future01 = CompletableFuture.supplyAsync(() -> {
    System.out.println("任务1线程：" + Thread.currentThread().getId());
    int i = 10 / 4;
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("任务1结束：");

    return i;
}, executor);

// 任务2
CompletableFuture<String> future02 = CompletableFuture.supplyAsync(() -> {
    System.out.println("任务2线程：" + Thread.currentThread().getId());
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("任务2结束");

    return "Hello";
}, executor);

// 任务1和任务2并行执行，只要有一个执行完成，就执行任务3，不使用任务1 或 任务2线程的结果，并且任务3没有返回值
future01.runAfterEitherAsync(future02,()-> System.out.println("任务3开始，之前的结果"), executor);

// 任务1和任务2并行执行，只要有一个执行完成，就执行任务3，使用任务1 或 任务2线程的结果，并且任务3没有返回值
future01.acceptEitherAsync(future02, (res)-> System.out.println("任务3开始，之前的结果" + res), executor);

// 任务1和任务2并行执行，只要有一个执行完成，就执行任务3，使用任务1 或 任务2线程的结果，并且任务3有返回值
CompletableFuture<Object> future = future01.applyToEitherAsync(future02, (res) -> {
    System.out.println("任务3开始，之前的结果" + res);
    return res.toString() + "->哈哈";
}, executor);
```


### 多任务组合（只要有一个执行完就返回）

```java
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs);

```

**java案例**

```java
CompletableFuture<String> future01 = CompletableFuture.supplyAsync(() -> "任务1", executorService);
CompletableFuture<String> future02 = CompletableFuture.supplyAsync(() -> "任务2", executorService);
CompletableFuture<String> future03 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "任务3";
}, executorService);

// 只要异步线程队列有一个任务率先完成就返回，这个特性可以用来获取最快的那个线程结果。
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future01, future02, future03);

// 获取若干个任务中最快完成的任务结果
// .join()和.get()都会阻塞并获取线程的执行情况
// .join()会抛出未经检查的异常，不会强制开发者处理异常 .get()会抛出检查异常，需要开发者处理
Object o1 = anyOf.get();
Object o2 = anyOf.join();

```


### 多任务组合（全部执行完才返回）

```java

public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs);

```

**java案例**

```java
CompletableFuture<String> future01 = CompletableFuture.supplyAsync(() -> "任务1", executorService);
CompletableFuture<String> future02 = CompletableFuture.supplyAsync(() -> "任务2", executorService);
CompletableFuture<String> future03 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "任务3";
}, executorService);

// 串联起若干个线程任务, 没有返回值
CompletableFuture<Void> all = CompletableFuture.allOf(future01, future02, future03);
// 等待所有线程执行完成
// .join()和.get()都会阻塞并获取线程的执行情况
// .join()会抛出未经检查的异常，不会强制开发者处理异常 .get()会抛出检查异常，需要开发者处理
all.join();
all.get();
```