# LOCK

## 历程

1.5之前依靠synchronized实现锁功能

1.5之后并发包里增加了LOCK接口

- 提供与Synchronized相似的锁功能，也支持对锁进行操作，但需要显示操作，另外还提供多种Synchroized不具备的同步特性

## 特性

1. 尝试非阻塞获取锁：尝试获取锁，若无其他线程拿到，则获取成功
2. 获取锁的线程能被中断：
3. 超时获取锁：限制获取锁的时间，超时未成功则返回

## 接口源码

```java
public interface Lock {
    
    void lock(); // 获取锁，获取成功后返回
    void lockInterruptibly() throws InterruptedException;// 可中断获取锁，获取过程中可以中断
    boolean tryLock();// 非阻塞获取锁，调用立刻返回结果，成功true,失败false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;// 超时获取锁
    //方法结束情况：1.得到锁；2.被中断；3.超时时间结束返回false
    
    void unlock();//释放锁
    
    Condition newCondition();//等待通知组件,获取锁后可调用
}
```

## &Synchroized对比

| 类别     | synchronized                                                 | Lock                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | Java的关键字，在jvm层面上                                    | 是一个类                                                     |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁                  |
| 锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待   | 分情况而定，Lock有多个锁获取的方式，具体下面会说道，大致就是可以尝试获得锁，线程可以不用一直等待 |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可重入 不可中断 非公平                                       | 可重入 可判断 可公平（两者皆可）                             |
| 性能     | 少量同步                                                     | 大量同步                                                     |

# ReentrantLock

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {// 是当前线程持有锁，则重入
        int nextc = c + acquires;// 同步状态值增加
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

每次获取锁增加同步状态

```java
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
```

每次释放锁，减少同步状态

# ReentrantReadWriteLock

ReadWriteLock

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

ReentrantReadWriteLock

```java
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {

    private final ReentrantReadWriteLock.ReadLock readerLock;
    private final ReentrantReadWriteLock.WriteLock writerLock;

    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }	

    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 6317671515068378041L;
        /*
         * Read vs write count extraction constants and functions.
         * Lock state is logically divided into two unsigned shorts:
         * The lower one representing the exclusive (writer) lock hold count,
         * and the upper the shared (reader) hold count.
         */
        static final int SHARED_SHIFT   = 16;
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    }
}
```

## 读锁

共享

## 写锁

互斥

## 锁降级

写锁降级为读锁

# LockSupport

阻塞或者唤醒线程

# Condition

Object 中的阻塞唤醒方法 + Synchroized可以实现等待通知模型

Condition + Lock也可以实现等待通知模型，且比Synchroized更强大一点

1. 可以等待多个队列
2. 线程释放锁进入等待状态，在等待状态中不响应中断
3. 线程释放锁进入等待状态到将来的某个状态

​	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729222458938.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

