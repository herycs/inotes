# Java并发



## ConcurrentHashMap

### 采用原因

- HashMap：并发执行HashMap 的put操作可能会导致死循环

- HashTable：HashTable采用synchroized确保其线程安全，竞争激烈情况下线程会进入阻塞或轮询状态，导致性能低下

- ConcurrentHashMap的锁

    - jdk1.5-1.7：不是锁全体，而是分段锁，这样可以不必锁全体，不同段可以同时操作。当segment的数据量大了之后，锁的粒度就变大了，操作起来其代价也不小。

    - jdk1.8：锁Node头节点，锁的粒度始终保持在比较小的状态

        1.7中的二次hash操作的速度，在1.8中采用链表+红黑树尽量缩小了性能差距

### 演进历程

1. 引入及发展：jdk1.5时引入ConcurrentHashMap，截止到jdk1.7都是采用：数组+链表的策略，锁：segment（继承可重入锁）ReentrantLock
2. 改进：jdk1.8修改其内部策略，放弃分段锁策略，对Node节点进行加锁（synchronized）

### 数据结构

#### jdk1.7

```shell
				  			   |---table[0]--->HashEntry1--->HashEntry2
				  |---segment---|
				  |			   |---table[n]--->HashEntry1--->HashEntry2--->HashEntry3
ConcurrentHashMap---
				  |			   |---table[0]--->HashEntry1
				  |---segment---|
				  			   |---table[n]--->HashEntry1--->HashEntry2
```

如上所示，数组+链表实现，其中segment是一个hash表，HashEntry是一个链表结构元素，数据操作需要先操作Segment

##### 查询流程

查找时，两次hash，一次hash得到segment，二次hash定位到链表头部，而后操作其链表

##### 相关操作

segment[]的初始化由 concurrencyLevel计算得出，其数组长度为2的整数幂，最大值为65535

初始化segmentShift, segmentMask

- segmentShift：段偏移量
- segmentMask：段掩码

初始化每个segment

- 数组长度：c = initialCapacity / ssize，HashMap长度除以ssize

    数组结果不是1就是，大于c的最小的2的N次方倍

#### jdk1.8

```java
				  |---table[0](Node)--->HashEntry1--->HashEntry2（长度大于8，转为红黑树）
				  |
				  |---table[1](Node)--->HashEntry1--->HashEntry2--->HashEntry3
ConcurrentHashMap---
				  |---table[2](Node)--->HashEntry1--->HashEntry2
				  |
				  |---table[3](Node)--->HashEntry1--->HashEntry2
```

可以看到结构中少了segment，采用node(每个Node保存有key-value，hash的数据结构，其中的value，next都是volatile修饰的以确保并发情况下的可见性)

##### 代码

> jdk1.8

继承关系

```java
//接口,其中定义了一些Map操作的基础方法
//@since 1.5
public interface ConcurrentMap<K, V> extends Map<K, V> {}
//@since 1.5
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {}
```

部分字段

```java
//Node[]最大容量
private static final int MAXIMUM_CAPACITY = 1 << 30;
//Table最大容量，Node[]默认初始值
private static final int DEFAULT_CAPACITY = 16;
//table的负载因子
private static final float LOAD_FACTOR = 0.75f;
//链表--->红黑树，阈值
static final int TREEIFY_THRESHOLD = 8;
//链表<---红黑树，阈值
static final int UNTREEIFY_THRESHOLD = 6;
```

操作

Node

可以看到其volatile和next都被volatile修饰，保证其可见性

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
}
```

TreeBin

```java
//TreeBin存储TreeNode
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;
    volatile TreeNode<K,V> first;
    volatile Thread waiter;
    volatile int lockState;//锁状态
    // values for lockState
    static final int WRITER = 1; // set while holding write lock
    static final int WAITER = 2; // set when waiting for write lock
    static final int READER = 4; // increment value for setting read lock
    //...省略其他代码
}
//TreeNode定义，也就是红黑树节点定义
static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    //...省略其他代码
}
```

锁

```java
//锁根节点，compareAndSwapInt()方法实现CAS策略
private final void lockRoot() {
    if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER))
        contendedLock(); // offload to separate method
}
//解锁根节点
private final void unlockRoot() { lockState = 0; }
```

分析其Put()方法可以发现它对put操作采用的是synchroized实现的

- 散列

    ```java
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
    ```

- putVal()方法

    ```java
    final V putVal(K key, V value, boolean onlyIfAbsent) {
         if (key == null || value == null) throw new NullPointerException();
            int hash = spread(key.hashCode());//获取hash值
            int binCount = 0;
            for (Node<K,V>[] tab = table;;) {
                Node<K,V> f; int n, i, fh;
                if (tab == null || (n = tab.length) == 0) tab = initTable();
                else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                    if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))//CAS操作增加节点
                        break;                   // no lock when adding to empty bin
                }
                else if ((fh = f.hash) == MOVED)
                    tab = helpTransfer(tab, f);
                else {
                    V oldVal = null;
                    synchronized (f) {//锁对象
                        if (tabAt(tab, i) == f) {
                            if (fh >= 0) {
                                binCount = 1;
                                for (Node<K,V> e = f;; ++binCount) {//链表
                                    K ek;
                                    if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                         (ek != null && key.equals(ek)))) {
                                        oldVal = e.val;
                                        if (!onlyIfAbsent)
                                            e.val = value;
                                        break;
                                    }
                                    Node<K,V> pred = e;
                                    if ((e = e.next) == null) {
                                        pred.next = new Node<K,V>(hash, key, value, null);
                                        break;
                                    }
                                }
                            }
                            else if (f instanceof TreeBin) {//新增节点为树节点
                                Node<K,V> p;
                                binCount = 2;
                                if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                               value)) != null) {
                                    oldVal = p.val;
                                    if (!onlyIfAbsent)
                                        p.val = value;
                                }
                            }
                        }
                    }
                    if (binCount != 0) {
                        if (binCount >= TREEIFY_THRESHOLD)
                            treeifyBin(tab, i);
                        if (oldVal != null)
                            return oldVal;
                        break;
                    }
                }
            }
            addCount(1L, binCount);
            return null;
    }
    ```

    

#### 总结

经过以上分析，我们可以发现1.7采用：数组+链表+segment分段锁（锁一段）实现，1.8采用：数组+链表|红黑树+synchroized（锁Node节点）

## ConcurrentLinkedQueue

> 实现线程安全队列：1.阻塞算法（入队出队一把锁，或者两把锁）；2.非阻塞算法（可以用CAS实现）

### 入队列

尾插法

### 出队列

1. 取头结点元素，判断是否为空
    1. 空->另有线程执行了出队操作
    2. 非空->CAS方式将头结点引用改为null
        1. CAS成功，则操作成功
        2. CAS失败，队列已修改，需要重新获取头几点

## 阻塞队列

- ArrayBlockingQueue（有界-数组实现）
- 
- LinkedBlockingQueue（有界-链表实现）
- LinkedTransferQueue（无界-链表实现）
- LinkedBlockingDeque（双向阻塞-链表实现）
- 
- PriorityBlockingQueue（无界-支持优先级排序）
- DelayQueue（无界-优先级队列实现）
- SynchronousQueue（不存储元素）

PriorityBlockingQueue（无界-支持优先级排序）

- 自定义实现compareTo()定义排序方式

DelayQueue（无界-优先级队列实现）

- 队列中元素需要实现DelayQueue
- 支持延时获取，只有当时间够了才允许从队列中获取当前元素

#### 实现原理

- 通知模式

## Fork/Join

### 工作窃取算法

- 优点：充分利用线程进行并行计算
- 缺点：某些情况下仍然存在竞争

### 框架设计

1. 分割任务
2. 执行任务并合并结果

### 异常处理

- 查看是否有异常：isCompletedAbnormally()
- 获取异常：ForkJoinTask.getException()

### 实现原理

- ForkJoinTask[]
- ForkJoinWorkerThread[]

任务状态四种：完成（NORMAL），取消（CANCELLED），信号（SIGNAL），异常（EXCEPTIONAL）

# Java并发工具类

## CountDownLatch

> 允许一个或多个线程等待，但线程计数器只能使用一次

## CyclicBarrier

> 同步屏障，线程计数器可重置（也就是说可以重复使用）

## Semaphore

> 信号量，控制对资源访问的线程数量

## Exchanger

> 线程间交换数据

# Executor框架

### 两级调度模型

HotSpotVM线程模型中Java线程被一对一映射为本地OS线程
