---
title: 2、ThreadLocal原理详解
icon: material-icon-theme:capnp
order: 2
author: bugcode
date: 2020-01-01T00:00:00.000Z
copyright: bugcode
createTime: 2026/01/17 16:55:00
permalink: /learning-notes/backend/java/ThreadLocal原理详解/
---

# Java ThreadLocal 原理详解

## 1. ThreadLocal 概述

### 1.1 什么是 ThreadLocal？

**ThreadLocal** 是 Java 提供的线程本地变量机制，它为每个使用该变量的线程提供独立的变量副本，实现了线程隔离。

### 1.2 核心特点

- **线程隔离**：每个线程有自己的变量副本
- **线程安全**：天然线程安全，无需同步
- **生命周期**：与线程生命周期绑定
- **内存泄漏风险**：需要正确清理

## 2. ThreadLocal 的基本使用

```java
public class ThreadLocalDemo {
    // 创建 ThreadLocal 变量
    private static final ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
    private static final ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
    
    public static void main(String[] args) {
        // 线程1
        Thread t1 = new Thread(() -> {
            threadLocal.set(100);  // 设置线程局部变量
            userThreadLocal.set(new User("张三"));
            System.out.println("线程1: " + threadLocal.get());
            System.out.println("线程1用户: " + userThreadLocal.get().name);
            
            // 使用后清理，防止内存泄漏
            userThreadLocal.remove();
        });
        
        // 线程2
        Thread t2 = new Thread(() -> {
            threadLocal.set(200);
            userThreadLocal.set(new User("李四"));
            System.out.println("线程2: " + threadLocal.get());
            System.out.println("线程2用户: " + userThreadLocal.get().name);
            
            userThreadLocal.remove();
        });
        
        t1.start();
        t2.start();
    }
    
    static class User {
        String name;
        User(String name) { this.name = name; }
    }
}
```

## 3. ThreadLocal 核心原理

### 3.1 关键类结构

```java
// ThreadLocal 类
public class ThreadLocal<T> {
    
    // ThreadLocal 的核心：获取当前线程的 ThreadLocalMap
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    
    // 设置值
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    // 获取当前线程的 ThreadLocalMap
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;  // Thread 类中的 threadLocals 字段
    }
    
    // 创建 ThreadLocalMap
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
}

// Thread 类中的相关字段
public class Thread implements Runnable {
    // 线程的 ThreadLocal 变量存储
    ThreadLocal.ThreadLocalMap threadLocals = null;
    
    // InheritableThreadLocal 相关
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
}
```

### 3.2 ThreadLocalMap 内部结构

```java
// ThreadLocalMap 是 ThreadLocal 的核心数据结构
static class ThreadLocalMap {
    
    // Entry 继承自 WeakReference，key 是弱引用
    static class Entry extends WeakReference<ThreadLocal<?>> {
        // 值使用强引用
        Object value;
        
        Entry(ThreadLocal<?> k, Object v) {
            super(k);  // key 是弱引用
            value = v; // value 是强引用
        }
    }
    
    // 初始容量
    private static final int INITIAL_CAPACITY = 16;
    
    // Entry 表
    private Entry[] table;
    
    // 元素数量
    private int size = 0;
    
    // 扩容阈值
    private int threshold;
    
    // 构造方法
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }
}
```

## 4. ThreadLocalMap 的哈希算法

### 4.1 魔数 0x61c88647

```java
public class ThreadLocal<T> {
    // 每个 ThreadLocal 实例的哈希值
    private final int threadLocalHashCode = nextHashCode();
    
    // 原子递增的哈希码生成器
    private static AtomicInteger nextHashCode = new AtomicInteger();
    
    // 黄金比例魔数，用于均匀分布
    private static final int HASH_INCREMENT = 0x61c88647;
    
    // 生成下一个哈希码
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
    
    // threadLocalHashCode 用于在 ThreadLocalMap 中定位索引
}
```

### 4.2 哈希分布演示

```java
// 黄金比例哈希的效果：均匀分布，减少哈希冲突
public class HashDistributionDemo {
    public static void main(String[] args) {
        int capacity = 16;
        int HASH_INCREMENT = 0x61c88647;
        int hashCode = 0;
        
        System.out.println("索引分布情况：");
        for (int i = 0; i < capacity; i++) {
            hashCode += HASH_INCREMENT;
            int index = hashCode & (capacity - 1);  // 位运算取模
            System.out.printf("ThreadLocal%d -> 索引: %d%n", i, index);
        }
        
        // 输出结果：
        // ThreadLocal0 -> 索引: 0
        // ThreadLocal1 -> 索引: 7
        // ThreadLocal2 -> 索引: 14
        // ThreadLocal3 -> 索引: 5
        // ThreadLocal4 -> 索引: 12
        // ... 均匀分布
    }
}
```

## 5. ThreadLocal 的内存泄漏问题

### 5.1 内存泄漏原因

```java
强引用链：
Thread → ThreadLocalMap → Entry → value（强引用）
                 ↓
Entry → key（弱引用）→ ThreadLocal

问题：
1. ThreadLocal 被回收后，key 变成 null
2. 但 value 仍然被 Entry 强引用
3. 如果线程不终止，value 永远无法回收
```

### 5.2 Entry 的引用关系

```java
// 关键点：Entry 继承自 WeakReference
static class Entry extends WeakReference<ThreadLocal<?>> {
    // key 是弱引用（继承自 WeakReference）
    // value 是强引用
    
    Entry(ThreadLocal<?> k, Object v) {
        super(k);  // key 作为弱引用
        value = v; // value 作为强引用
    }
}
```

### 5.3 内存泄漏示例

```java
public class MemoryLeakDemo {
    static class BigObject {
        byte[] data = new byte[1024 * 1024];  // 1MB
    }
    
    public static void main(String[] args) throws InterruptedException {
        ThreadLocal<BigObject> threadLocal = new ThreadLocal<>();
        
        Thread thread = new Thread(() -> {
            threadLocal.set(new BigObject());  // 设置大对象
            // 线程结束后，如果 ThreadLocal 没有被 remove
            // 且线程池中的线程一直存活，会导致内存泄漏
        });
        
        thread.start();
        thread.join();
        
        // 线程结束，但如果是线程池中的线程，可能一直存在
        // BigObject 无法被回收，因为 Entry 仍然持有强引用
    }
}
```

## 6. ThreadLocal 的清理机制

### 6.1 set() 方法的清理逻辑

```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    
    // 线性探测解决哈希冲突
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        
        // 找到相同的 key
        if (k == key) {
            e.value = value;
            return;
        }
        
        // 发现过期 Entry（key 为 null）
        if (k == null) {
            replaceStaleEntry(key, value, i);  // 替换过期 Entry
            return;
        }
    }
    
    tab[i] = new Entry(key, value);
    int sz = ++size;
    
    // 清理并检查是否需要扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();  // 重新哈希
}
```

### 6.2 get() 方法的清理逻辑

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);  // 探测查找
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;
    
    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);  // 清理过期 Entry
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

### 6.3 主动清理方法

```java
// 1. remove() - 显式清理
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);  // 从 ThreadLocalMap 中移除
}

// 2. expungeStaleEntry() - 清理过期 Entry
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    
    // 清理当前槽位
    tab[staleSlot].value = null;  // 释放 value 引用
    tab[staleSlot] = null;
    size--;
    
    // 重新哈希后续的 Entry
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;  // 清理过期 Entry
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {  // 如果不在"正确"位置
                tab[i] = null;  // 清除原位置
                // 线性探测找到新位置
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;  // 重新放置
            }
        }
    }
    return i;
}
```

## 7. InheritableThreadLocal

### 7.1 父子线程传值

```java
public class InheritableThreadLocalDemo {
    // 可继承的 ThreadLocal
    private static final InheritableThreadLocal<String> inheritableThreadLocal = 
        new InheritableThreadLocal<>();
    
    public static void main(String[] args) {
        inheritableThreadLocal.set("父线程的值");
        
        Thread childThread = new Thread(() -> {
            System.out.println("子线程获取: " + inheritableThreadLocal.get());
            inheritableThreadLocal.set("子线程修改");
            System.out.println("子线程修改后: " + inheritableThreadLocal.get());
        });
        
        childThread.start();
        
        try {
            childThread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("父线程获取: " + inheritableThreadLocal.get());
        // 输出：父线程的值（子线程修改不影响父线程）
    }
}
```

### 7.2 实现原理

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    
    // 创建子线程时复制父线程的值
    protected T childValue(T parentValue) {
        return parentValue;  // 默认直接返回父值
    }
    
    // 获取子线程的 ThreadLocalMap
    ThreadLocalMap getMap(Thread t) {
        return t.inheritableThreadLocals;
    }
    
    // 创建子线程的 ThreadLocalMap
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}

// Thread 的 init() 方法中的复制逻辑
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc) {
    // ... 初始化代码
    
    // 复制父线程的 inheritableThreadLocals
    Thread parent = currentThread();
    if (parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals = 
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
}
```

## 8. ThreadLocal 的最佳实践

### 8.1 正确使用模式

```java
public class ThreadLocalBestPractice {
    // 1. 使用静态 final 修饰
    private static final ThreadLocal<SimpleDateFormat> dateFormatHolder =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
    
    // 2. 使用后必须清理
    public void processRequest() {
        try {
            SimpleDateFormat sdf = dateFormatHolder.get();
            // 使用 sdf 处理日期
            String formatted = sdf.format(new Date());
            // ...
        } finally {
            // 必须清理，特别是线程池场景
            dateFormatHolder.remove();
        }
    }
    
    // 3. 使用 try-with-resources 模式（Java 8+）
    public void processWithTryResource() {
        try (ThreadLocalContext context = new ThreadLocalContext()) {
            // 自动清理
        }
    }
}

// 4. 自定义可自动清理的 ThreadLocal
class AutoCleanThreadLocal<T> extends ThreadLocal<T> {
    @Override
    protected void finalize() throws Throwable {
        remove();  // 被回收时清理
        super.finalize();
    }
}
```

### 8.2 线程池中的 ThreadLocal

```java
public class ThreadPoolWithThreadLocal {
    private static final ThreadLocal<UserContext> userContext = new ThreadLocal<>();
    
    private static final ExecutorService executor = Executors.newFixedThreadPool(5);
    
    static class UserContext {
        String userId;
        String sessionId;
        
        UserContext(String userId, String sessionId) {
            this.userId = userId;
            this.sessionId = sessionId;
        }
    }
    
    public void submitTask(String userId, String sessionId) {
        // 任务提交前设置 ThreadLocal
        userContext.set(new UserContext(userId, sessionId));
        
        executor.submit(() -> {
            try {
                UserContext context = userContext.get();
                if (context != null) {
                    System.out.println("处理用户: " + context.userId);
                    // 处理业务
                }
            } finally {
                // 必须清理！线程会被重用
                userContext.remove();
            }
        });
    }
    
    // 更好的方式：使用包装器
    public void submitTaskBetter(String userId, String sessionId) {
        executor.submit(new UserAwareTask(userId, sessionId));
    }
    
    static class UserAwareTask implements Runnable {
        private final String userId;
        private final String sessionId;
        
        UserAwareTask(String userId, String sessionId) {
            this.userId = userId;
            this.sessionId = sessionId;
        }
        
        @Override
        public void run() {
            // 直接在任务内部设置，避免线程复用问题
            userContext.set(new UserContext(userId, sessionId));
            try {
                // 执行业务
            } finally {
                userContext.remove();
            }
        }
    }
}
```

## 9. ThreadLocal 的实际应用场景

### 9.1 Spring 中的 ThreadLocal

```java
// 1. RequestContextHolder（Spring MVC）
public abstract class RequestContextHolder {
    private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
        new NamedThreadLocal<>("Request attributes");
    
    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
        new NamedInheritableThreadLocal<>("Request context");
    
    public static RequestAttributes getRequestAttributes() {
        return requestAttributesHolder.get();
    }
    
    public static void setRequestAttributes(RequestAttributes attributes) {
        setRequestAttributes(attributes, false);
    }
}

// 2. TransactionSynchronizationManager（Spring 事务）
public abstract class TransactionSynchronizationManager {
    private static final ThreadLocal<Map<Object, Object>> resources =
        new NamedThreadLocal<>("Transactional resources");
    
    private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
        new NamedThreadLocal<>("Transaction synchronizations");
    
    private static final ThreadLocal<String> currentTransactionName =
        new NamedThreadLocal<>("Current transaction name");
}
```

### 9.2 MyBatis 中的 ThreadLocal

```java
// SqlSessionManager（管理 SqlSession）
public class SqlSessionManager implements SqlSessionFactory, SqlSession {
    private final ThreadLocal<SqlSession> localSqlSession = new ThreadLocal<>();
    
    public void startManagedSession() {
        localSqlSession.set(openSession());
    }
    
    public SqlSession getSqlSession() {
        SqlSession sqlSession = localSqlSession.get();
        if (sqlSession == null) {
            throw new SqlSessionException("Error: Cannot get SqlSession.");
        }
        return sqlSession;
    }
}
```

### 9.3 日期格式化

```java
public class DateUtil {
    // SimpleDateFormat 不是线程安全的
    private static final ThreadLocal<SimpleDateFormat> dateFormat = 
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
    
    public static String format(Date date) {
        return dateFormat.get().format(date);
    }
    
    public static Date parse(String dateStr) throws ParseException {
        return dateFormat.get().parse(dateStr);
    }
}
```

## 10. ThreadLocal 的性能优化

### 10.1 FastThreadLocal（Netty）

```java
// Netty 的 FastThreadLocal 优化
public class FastThreadLocal<V> {
    // 使用索引而不是哈希
    private final int index;
    
    // 所有 FastThreadLocal 共享的索引
    private static final AtomicInteger nextIndex = new AtomicInteger(0);
    
    public FastThreadLocal() {
        index = nextIndex.getAndIncrement();
    }
    
    // 使用数组而不是哈希表，O(1) 访问
    public V get() {
        InternalThreadLocalMap threadLocalMap = InternalThreadLocalMap.get();
        Object v = threadLocalMap.indexedVariable(index);
        if (v != InternalThreadLocalMap.UNSET) {
            return (V) v;
        }
        return initialize(threadLocalMap);
    }
}
```

### 10.2 ThreadLocal 与缓存行填充

```java
// 避免伪共享的 ThreadLocal
public class PaddedThreadLocal<T> extends ThreadLocal<T> {
    // 缓存行填充
    public long p1, p2, p3, p4, p5, p6, p7; // 56 bytes
    
    private volatile T value;
    
    public long q1, q2, q3, q4, q5, q6, q7; // 56 bytes
    // 总共 128 bytes，避免伪共享
}
```

## 11. ThreadLocal 的替代方案

### 11.1 ScopedValue（Java 20+ 预览特性）

```java
// Java 20 引入的 ScopedValue，解决 ThreadLocal 的父子线程传递问题
public class ScopedValueDemo {
    private static final ScopedValue<String> USER = ScopedValue.newInstance();
    
    public static void main(String[] args) {
        // 设置作用域值
        ScopedValue.where(USER, "Alice").run(() -> {
            System.out.println("User: " + USER.get());
            
            // 子任务继承作用域值
            Thread.ofVirtual().start(() -> {
                System.out.println("Child thread: " + USER.get());
            }).join();
        });
    }
}
```

### 11.2 TransmittableThreadLocal（阿里开源）

```java
// 解决线程池中 ThreadLocal 的值传递问题
public class TtlDemo {
    private static final TransmittableThreadLocal<String> context = 
        new TransmittableThreadLocal<>();
    
    public static void main(String[] args) {
        context.set("value-set-in-parent");
        
        ExecutorService executor = Executors.newFixedThreadPool(1);
        
        // 使用 TtlExecutors 包装
        ExecutorService ttlExecutor = TtlExecutors.getTtlExecutorService(executor);
        
        ttlExecutor.submit(() -> {
            // 可以获取父线程设置的值
            System.out.println("子线程获取: " + context.get());
        });
    }
}
```

## 12. 常见问题及解决方案

### 12.1 问题1：线程池中的内存泄漏

```java
// 错误用法
ExecutorService executor = Executors.newFixedThreadPool(10);
ThreadLocal<byte[]> threadLocal = new ThreadLocal<>();

executor.submit(() -> {
    threadLocal.set(new byte[1024 * 1024]);  // 1MB
    // 没有 remove()，线程复用导致内存泄漏
});

// 解决方案1：使用 try-finally
executor.submit(() -> {
    try {
        threadLocal.set(new byte[1024 * 1024]);
        // 业务逻辑
    } finally {
        threadLocal.remove();  // 必须清理
    }
});

// 解决方案2：使用包装器
class ThreadLocalAwareTask implements Runnable {
    private final byte[] data;
    
    ThreadLocalAwareTask(byte[] data) {
        this.data = data;
    }
    
    @Override
    public void run() {
        threadLocal.set(data);
        try {
            // 业务逻辑
        } finally {
            threadLocal.remove();
        }
    }
}
```

### 12.2 问题2：InheritableThreadLocal 的意外修改

```java
// 子线程修改影响父线程的问题
public class InheritableThreadLocalIssue {
    private static final InheritableThreadLocal<StringBuilder> sharedBuilder = 
        new InheritableThreadLocal<StringBuilder>() {
            @Override
            protected StringBuilder childValue(StringBuilder parentValue) {
                // 错误：直接返回了同一个对象引用
                return parentValue;
            }
        };
    
    // 正确做法：返回副本
    private static final InheritableThreadLocal<StringBuilder> safeBuilder = 
        new InheritableThreadLocal<StringBuilder>() {
            @Override
            protected StringBuilder childValue(StringBuilder parentValue) {
                if (parentValue == null) return null;
                return new StringBuilder(parentValue);  // 返回副本
            }
        };
}
```

## 总结

**ThreadLocal 的核心要点**：

1. **数据结构**：每个 Thread 维护一个 ThreadLocalMap，使用开放地址法解决哈希冲突
2. **内存泄漏**：Entry 的 key 是弱引用，value 是强引用，必须手动 remove()
3. **哈希算法**：使用魔数 0x61c88647 实现均匀分布
4. **清理机制**：在 set() 和 get() 过程中自动清理过期 Entry

**最佳实践**：

1. **必须清理**：使用 try-finally 确保 remove()
2. **线程池慎用**：线程复用会导致数据错乱和内存泄漏
3. **静态修饰**：ThreadLocal 变量通常声明为 static final
4. **合理使用**：只在确实需要线程隔离时使用

**适用场景**：

- 线程不安全的工具类（如 SimpleDateFormat）
- 上下文传递（如用户会话）
- 事务管理
- 跨方法参数传递（避免方法参数透传）

ThreadLocal 是 Java 并发编程中的重要工具，正确理解其原理和使用方式对于编写高质量的并发程序至关重要。