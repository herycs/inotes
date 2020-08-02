# Java中的锁

# AQS

> AbstractQueuedSynchronizer(AQS)队列同步器
>
> int类型成员变量，内置FIFO队列完成资源获取线程的排队工作

## 作用

构建锁&同步组件的基础框架

## 使用方式

继承，实现同步方法设置对同步状态的修改，关键是对**同步器**（实现锁的关键，基于模板方法设计（抽象类定义了执行流程，但真正实执行时是依照此流程调用子类的具体实现），此同步器未实现任何接口，其内定义了一些基础方法）的实现，正因为其只定义了抽象方法，所以具体这个同步器是采用**独占**还是**共享******取决于实现者****。同步器常用可重写方法：

- tryAcquire()：独占式获取锁，（实现：先查询当前状态是否符合预期，在CAS设置同步状态）
- tryRelease()：独占式释放锁
- tryAcquireShared()：共享式获取同步状态，返回值大于等于0则获取成功，否则获取失败
- tryReleaseShared()：共享式释放锁
- isHeldExclusively()：当前同步器是否在独占模式下被线程占用

## 实现

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer 
    implements java.io.Serializable {

    static final class Node {

        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;

        static final int CANCELLED =  1;
        static final int SIGNAL    = -1;
        static final int CONDITION = -2;
        static final int PROPAGATE = -3;

        volatile int waitStatus;
        // ...
    }
    
    private transient volatile Node head;

    private transient volatile Node tail;

    private volatile int state;
    // 以下是AQS中提供的对于同步状态的修改的方法定义
    protected final int getState() { return state; }// 获取同步状态
    protected final void setState(int newState) { state = newState; } // 设置同步状态
 	protected final boolean compareAndSetState(int expect, int update) {// 设置同步状态
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
}
```

## 同步队列

- 同步器中保留两个引用
    - head（头结点）
    - tail（尾结点）
- 双端FIFO队列
    - 失败：获取同步状态失败，构建Node（当前线程信息+等待信息），加入同步队列，阻塞当前线程
    - 再次尝试：同步状态释放时首节点线程唤醒

入队列：尾插法（compareAndSetTail()）

出队列：首节点释放同步状态唤醒Next节点，Next节点获取同步状态成功设置自己为首节点



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

# 锁优化

## 锁消除

> [参考博客](https://yq.aliyun.com/articles/666316)

同步块不一定会被多个线程使用到，当只有一个线程使用同步块，这时再对进行加锁，解锁操作有些多此一举了，Java虚拟机JIT编译时通过上下文扫描，会除去不存在公共资源竞争的锁

## 粒度优化

### 锁细化

能不写在同步块里的就不要写在同步块里

大锁拆小锁，增加并行，减小竞争

#### 实例

ConcurrentHashMap

LongAdder

LongAdder 实现思路也类似ConcurrentHashMap，LongAdder有一个根据当前并发状况动态改变的Cell数组，Cell对象里面有一个long类型的value用来存储值;

- 开始**没有并发争用**的时候或者是cells数组正在初始化的时候，会使用**cas**来将值累加到成员变量的base上
- 在**并发争用**的情况下，LongAdder会初始化cells数组，在Cell数组中选定一个**Cell加锁**，数组有多少个cell，就允许同时有多少线程进行修改，最后将数组中每个Cell中的value相加，在加上base的值，就是最终的值；
- cell数组还能根据当前线程争用情况进行**扩容**，初始长度为2，每次扩容会增长一倍，直到扩容到**大于等于cpu数量**就不再扩容

这也就是为什么LongAdder比cas和AtomicInteger效率要高的原因，后面两者都是volatile+cas实现的，他们的竞争维度是1，LongAdder的竞争维度为“Cell个数+1”为什么要+1？因为它还有一个base，如果竞争不到锁还会尝试将数值加到base上

LinkedBlockingQueue

LinkedBlockingQueue也体现了这样的思想，在队列头入队，在队列尾出队，入队和出队使用不同的锁，相对于LinkedBlockingArray只有一个锁效率要高；

### 锁粗化

例如，循环体中加锁，每次操作，都需要进入&退出临界区，这种情况下应当将锁放大，将这个循环锁起来，减少临界区的进退次数

### 读写分离

CopyOnWriteArrayList

CopyOnWriteArraySet

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

CopyOnWrite并发容器用于读多写少的并发场景，因为，读的时候没有锁，但是对其进行更改的时候是会加锁的，否则会导致多个线程同时复制出多个副本，各自修改各自的；

### 读写锁

接口：ReadWriteLock

实现：并发包提供ReentrantReadWriteLock

状态设计：整型变量，高16位用于读状态记录，低16位用于写状态记录

- 写状态：当前状态 & 0x0000FFFF(获取)，当前状态+1（修改）
- 读状态：当前状态>>>16（获取），当前状态+1<<<16（修改）

写锁：支持重入的排它锁

读锁：支持重入的共享锁

锁降级：写锁降级--->读锁

## CAS&同步

## 消除伪共享

两个线程对毫不想干得两个变量的访问，但因为加载数据是以缓存行为单位的，导致A操作行的数据时，B需要等待A操作完（而这，本没有必要）

## 重入锁

> ReentrantlLock，但关键字synchronized支持隐式重入

重入：获取锁线程多次重入（每次获取，计数自增）

释放：（计数自减为0）锁释放

## 公平，非公平锁

公平锁：锁的获取遵循严格的先后限制

非公平锁：非严格显示，一个线程可能多次获取同一个锁（这会导致“饥饿”状态的出现）

默认采用：非公平锁，饥饿出现的是少数，大量并发情况下公平锁的上下文切换次数远高于非公平锁

