# AQS

> AbstractQueuedSynchronizer(AQS)队列同步器
>
> int类型成员变量，内置FIFO队列完成资源获取线程的排队工作

## 作用

它是构建锁或者其他同步组件的基础框架，JUC并发包的作者期望它能够成为实现大部分同步需求的基础

它是JUC并发包中的核心基础组件

AQS解决了子类实现同步器时涉及当的大量细节问题，例如获取同步状态、FIFO同步队列

它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。在基于AQS构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。同时在设计AQS时充分考虑了可伸缩行，因此J.U.C中所有基于AQS构建的同步器均可以获得这个优势。

例如：

- ReentrantLock
- ReentrantReadWriteLock
- Semaphore

## 使用方式

继承，实现同步方法设置对同步状态的修改，关键是对**同步器**（实现锁的关键，基于模板方法设计（抽象类定义了执行流程，但真正实执行时是依照此流程调用子类的具体实现），此同步器未实现任何接口，其内定义了一些基础方法）的实现，正因为其只定义了抽象方法，所以具体这个同步器是采用**独占**还是**共享******取决于实现者****。同步器常用可重写方法：

- tryAcquire()：独占式获取锁，（实现：先查询当前状态是否符合预期，在CAS设置同步状态）
- tryRelease()：独占式释放锁
- tryAcquireShared()：共享式获取同步状态，返回值大于等于0则获取成功，否则获取失败
- tryReleaseShared()：共享式释放锁
- isHeldExclusively()：当前同步器是否在独占模式下被线程占用

## 实现

```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer 
    implements java.io.Serializable {

    static final class Node {

        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;

        static final int CANCELLED =  1; // 取消
        // 因为超时或者中断，结点会被设置为取消状态，被取消状态的结点不应该去竞争锁，只能保持取消状态不变，不能转换为其他状态。处于这种状态的结点会被踢出队列，被GC回收
        static final int SIGNAL    = -1; // 
        // 后继节点的线程处于等待的状态、当前节点的线程如果释放了同步状态或者被取消、会通知后继节点、后继节点会获取锁并执行
        static final int CONDITION = -2; // 表明当前线程在等待条件
        // 节点在等待队列中、节点线程等待在Condition、当其它线程对Condition调用了singal()方法该节点会从等待队列中移到同步队列中
        static final int PROPAGATE = -3; // 表明当前状态应该无条件传播
		// 表示下一次共享式同步状态获取将会被无条件的被传播下去（读写锁中存在的状态，代表后续还有资源，可以多个线程同时拥有同步状态）
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