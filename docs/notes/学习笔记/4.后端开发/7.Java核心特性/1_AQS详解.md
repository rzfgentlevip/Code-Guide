---
title: 1、AQS原理详解
icon: material-icon-theme:browserlist
order: 1
author: bugcode
date: 2020-01-01T00:00:00.000Z
copyright: bugcode
createTime: 2026/01/17 16:55:00
permalink: /learning-notes/backend/java/AQS原理详解/
---

# Java AbstractQueuedSynchronizer (AQS) 详解

## 1. AQS 概述

### 1.1 什么是 AQS？

**AbstractQueuedSynchronizer** 是 Java 并发包 `java.util.concurrent.locks` 的核心框架，用于构建锁和同步器的**基础框架**。Doug Lea 设计，被广泛用于 Java 并发工具的实现。

### 1.2 核心思想

AQS 使用一个 **FIFO 队列** 来管理等待线程，并通过一个 **volatile int 状态变量** 来表示同步状态。

```java
// AQS 的核心数据结构
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer {
    
    // 等待队列的头节点
    private transient volatile Node head;
    // 等待队列的尾节点
    private transient volatile Node tail;
    // 同步状态
    private volatile int state;
    
    // 内部 Node 类
    static final class Node {
        // 节点模式
        static final Node SHARED = new Node();  // 共享模式
        static final Node EXCLUSIVE = null;     // 独占模式
        
        // 等待状态
        volatile int waitStatus;
        static final int CANCELLED = 1;   // 取消
        static final int SIGNAL = -1;     // 需要唤醒后继节点
        static final int CONDITION = -2;  // 在条件队列中
        static final int PROPAGATE = -3;  // 传播状态
        
        volatile Node prev;  // 前驱节点
        volatile Node next;  // 后继节点
        volatile Thread thread;  // 等待的线程
        Node nextWaiter;  // 指向下一个等待节点（共享/独占模式）
    }
}
```

## 2. AQS 的工作原理

### 2.1 同步状态（State）

```java
// 状态变量的操作
protected final int getState() {
    return state;
}

protected final void setState(int newState) {
    state = newState;
}

// 使用 CAS 原子更新状态
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

### 2.2 两种同步模式

| 模式         | 特点                           | 应用                      |
| :----------- | :----------------------------- | :------------------------ |
| **独占模式** | 同一时刻只有一个线程能获取资源 | ReentrantLock             |
| **共享模式** | 多个线程可以同时获取资源       | Semaphore, CountDownLatch |

### 2.3 等待队列结构

```java
等待队列（双向CLH队列）
head (dummy node) → node1 → node2 → tail
     ↑            ↑      ↑       ↑
   prev         next   prev    next
   
节点状态：
- SIGNAL(-1): 后继节点需要被唤醒
- CANCELLED(1): 节点已取消
- CONDITION(-2): 在条件队列中
- PROPAGATE(-3): 共享模式下传播
```

## 3. AQS 的核心方法

### 3.1 需要子类实现的方法

```java
// 独占模式
protected boolean tryAcquire(int arg)  // 尝试获取资源
protected boolean tryRelease(int arg)  // 尝试释放资源

// 共享模式
protected int tryAcquireShared(int arg)  // 尝试获取共享资源
protected boolean tryReleaseShared(int arg)  // 尝试释放共享资源

// 查询同步器是否在独占模式下被占用
protected boolean isHeldExclusively()
```

### 3.2 对外提供的模板方法

```java
// 独占模式
public final void acquire(int arg)        // 获取资源（不可中断）
public final void acquireInterruptibly(int arg)  // 可中断获取
public final boolean tryAcquireNanos(int arg, long nanosTimeout)  // 超时获取
public final boolean release(int arg)     // 释放资源

// 共享模式
public final void acquireShared(int arg)
public final void acquireSharedInterruptibly(int arg)
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
public final boolean releaseShared(int arg)

// 查询方法
public final boolean hasQueuedThreads()   // 是否有等待线程
public final boolean isQueued(Thread thread)  // 线程是否在队列中
public final int getQueueLength()         // 队列长度
```

## 4. AQS 源码深度解析

### 4.1 acquire() 方法流程

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&  // 1. 尝试获取资源
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  // 2. 加入队列并等待
        selfInterrupt();  // 3. 如果等待过程中被中断，补上中断
}

// addWaiter() - 将当前线程加入等待队列
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);  // 队列为空或CAS失败，自旋入队
    return node;
}

// acquireQueued() - 在队列中等待获取资源
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {  // 自旋
            final Node p = node.predecessor();  // 前驱节点
            if (p == head && tryAcquire(arg)) {  // 如果是头节点且能获取资源
                setHead(node);  // 设置为头节点
                p.next = null;  // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&  // 检查是否需要阻塞
                parkAndCheckInterrupt())  // 阻塞线程
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 4.2 release() 方法流程

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {  // 1. 尝试释放资源
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);  // 2. 唤醒后继节点
        return true;
    }
    return false;
}

// unparkSuccessor() - 唤醒后继节点
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {  // 后继节点为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;  // 从后向前找有效的节点
    }
    if (s != null)
        LockSupport.unpark(s.thread);  // 唤醒线程
}
```

## 5. AQS 的实际应用

### 5.1 ReentrantLock 的实现

```java
public class ReentrantLock implements Lock {
    private final Sync sync;
    
    // 同步器基类
    abstract static class Sync extends AbstractQueuedSynchronizer {
        // 公平锁和非公平锁的公共实现
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;  // 重入
                if (nextc < 0) throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
        
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
    }
    
    // 非公平锁
    static final class NonfairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    
    // 公平锁
    static final class FairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&  // 关键：检查队列中是否有前驱
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
}
```

### 5.2 CountDownLatch 的实现

```java
public class CountDownLatch {
    private static final class Sync extends AbstractQueuedSynchronizer {
        Sync(int count) {
            setState(count);  // 初始化状态
        }
        
        int getCount() {
            return getState();
        }
        
        // 共享模式获取
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;  // 状态为0表示可以获取
        }
        
        // 共享模式释放
        protected boolean tryReleaseShared(int releases) {
            for (;;) {
                int c = getState();
                if (c == 0) return false;
                int nextc = c - 1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;  // 减到0时返回true
            }
        }
    }
    
    private final Sync sync;
    
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);  // 调用AQS的共享获取
    }
    
    public void countDown() {
        sync.releaseShared(1);  // 调用AQS的共享释放
    }
}
```

### 5.3 Semaphore 的实现

```java
public class Semaphore {
    private final Sync sync;
    
    abstract static class Sync extends AbstractQueuedSynchronizer {
        Sync(int permits) {
            setState(permits);
        }
        
        final int getPermits() {
            return getState();
        }
        
        // 非公平方式获取许可
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 || 
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
    }
    
    // 公平信号量
    static final class FairSync extends Sync {
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors())  // 公平性检查
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 || 
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }
}
```

## 6. AQS 的条件变量（Condition）

### 6.1 ConditionObject 实现

```java
public class ConditionObject implements Condition {
    // 条件队列（单向链表）
    private transient Node firstWaiter;
    private transient Node lastWaiter;
    
    // await() 方法
    public final void await() throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();  // 加入条件队列
        int savedState = fullyRelease(node);  // 完全释放锁
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {  // 不在同步队列中
            LockSupport.park(this);  // 阻塞
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)  // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
    
    // signal() 方法
    public final void signal() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);  // 唤醒第一个等待节点
    }
    
    // 将节点从条件队列转移到同步队列
    final boolean transferForSignal(Node node) {
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        Node p = enq(node);  // 加入同步队列
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);  // 唤醒线程
        return true;
    }
}
```

### 6.2 条件变量的使用示例

```java
public class BoundedBuffer<E> {
    final Lock lock = new ReentrantLock();
    final Condition notFull = lock.newCondition();  // 条件：不满
    final Condition notEmpty = lock.newCondition(); // 条件：不空
    
    final Object[] items = new Object[100];
    int putptr, takeptr, count;
    
    public void put(E x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)  // 缓冲区满
                notFull.await();  // 等待不满条件
            items[putptr] = x;
            if (++putptr == items.length) putptr = 0;
            ++count;
            notEmpty.signal();  // 唤醒取数据线程
        } finally {
            lock.unlock();
        }
    }
    
    public E take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)  // 缓冲区空
                notEmpty.await();  // 等待不空条件
            E x = (E) items[takeptr];
            if (++takeptr == items.length) takeptr = 0;
            --count;
            notFull.signal();  // 唤醒放数据线程
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

## 7. AQS 的高级特性

### 7.1 公平锁 vs 非公平锁

```java
// 公平锁：hasQueuedPredecessors() 检查
public final boolean hasQueuedPredecessors() {
    Node t = tail;
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}

// 非公平锁：直接尝试获取，不考虑队列顺序
```

### 7.2 可重入性支持

```java
// ReentrantLock 中的重入实现
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    } else if (current == getExclusiveOwnerThread()) {  // 检查是否是当前线程
        int nextc = c + acquires;  // 增加重入次数
        if (nextc < 0) throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

### 7.3 超时和中断支持

```java
// tryAcquireNanos() 实现超时获取
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) ||  // 先尝试获取
        doAcquireNanos(arg, nanosTimeout);  // 超时获取
}

private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null;
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)  // 超时
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)  // 超过阈值才阻塞
                LockSupport.parkNanos(this, nanosTimeout);  // 阻塞指定时间
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

## 8. AQS 的性能优化

### 8.1 自旋优化

```java
// 在阻塞前自旋尝试获取锁
static final long spinForTimeoutThreshold = 1000L;  // 1微秒

if (shouldParkAfterFailedAcquire(p, node) &&
    nanosTimeout > spinForTimeoutThreshold)  // 短时间自旋
    LockSupport.parkNanos(this, nanosTimeout);
```

### 8.2 CLH 队列的变种

```java
// AQS 使用的是 CLH 队列的变种
// 原 CLH 队列：前驱节点的状态决定当前节点是否阻塞
// AQS 变种：通过前驱节点的 waitStatus 来控制
```

## 9. AQS 的常见问题和解决方案

### 9.1 锁饥饿问题

```java
// 使用公平锁避免锁饥饿
ReentrantLock fairLock = new ReentrantLock(true);  // 公平锁

// AQS 公平性实现的关键方法
public final boolean hasQueuedPredecessors() {
    // 检查是否有比当前线程等待更久的线程
}
```

### 9.2 死锁检测和避免

```java
// 使用 tryLock 避免死锁
public boolean transferMoney(Account from, Account to, int amount) {
    // 获取锁的顺序：按 id 排序
    Account first = from.id < to.id ? from : to;
    Account second = from.id < to.id ? to : from;
    
    if (first.lock.tryLock()) {
        try {
            if (second.lock.tryLock()) {
                try {
                    // 转账逻辑
                    return true;
                } finally {
                    second.lock.unlock();
                }
            }
        } finally {
            first.lock.unlock();
        }
    }
    return false;
}
```

## 10. AQS 的扩展和自定义实现

### 10.1 自定义互斥锁

```java
public class Mutex {
    private static class Sync extends AbstractQueuedSynchronizer {
        // 是否处于占用状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }
        
        // 尝试获取锁
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // 只能获取1个许可
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
        
        // 尝试释放锁
        protected boolean tryRelease(int releases) {
            assert releases == 1; // 只能释放1个许可
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
        
        // 提供 Condition
        Condition newCondition() { return new ConditionObject(); }
    }
    
    private final Sync sync = new Sync();
    
    public void lock() { sync.acquire(1); }
    public boolean tryLock() { return sync.tryAcquire(1); }
    public void unlock() { sync.release(1); }
    public Condition newCondition() { return sync.newCondition(); }
    public boolean isLocked() { return sync.isHeldExclusively(); }
}
```

### 10.2 自定义共享锁（二进制信号量）

```java
public class BinarySemaphore {
    private static class Sync extends AbstractQueuedSynchronizer {
        Sync(int permits) {
            setState(permits);
        }
        
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 || 
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        
        protected boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < 0 || next > 1)  // 二进制：0或1
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
    }
    
    private final Sync sync;
    
    public BinarySemaphore(int permits) {
        if (permits != 0 && permits != 1)
            throw new IllegalArgumentException("permits must be 0 or 1");
        sync = new Sync(permits);
    }
    
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    
    public void release() {
        sync.releaseShared(1);
    }
}
```

## 总结

**AQS 的核心价值**：

1. **提供了构建锁和同步器的基础框架**
2. **实现了复杂的线程排队和唤醒机制**
3. **支持公平/非公平、独占/共享、可重入等特性**
4. **是 Java 并发包的核心基石**

**使用 AQS 的建议**：

1. **优先使用现有的并发工具**（ReentrantLock、CountDownLatch等）
2. **理解 AQS 原理有助于排查并发问题**
3. **自定义同步器时考虑继承 AQS**
4. **注意性能和公平性的权衡**

AQS 通过巧妙的队列管理和状态控制，为 Java 并发编程提供了强大而灵活的基础设施。