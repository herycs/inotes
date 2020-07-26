# 并发

# 概述

上下文切换：任务从保存到再次加载的过程

并发和并行：

> ​        如果某个系统支持两个或者多个动作（Action）**同时存在**，那么这个系统就是一个**并发系统**。如果某个系统支持两个或者多个动作**同时执行**，那么这个系统就是一个**并行系统**。并发系统与并行系统这两个定义之间的关键差异在于**“存在”**这个词。
>
> ​                                                                                                                                                  《并发的艺术》 — 〔美〕布雷谢斯

- 对于单处理操作系统而言，所谓的并行实质上是极段时间间隔切换执行的并行

并发串行比较：

- 并发执行不超过百万次，速度比串行执行慢（原因：线程有创建和上下文切换的开销）

减少上下文切换策略

- 无锁并发编程
- CSA算法
- 使用最少线程
- 协程：单线程实现多任务调度，单线程里实现多任务切换

死锁应对

- 避免一个线程同时获取多个锁
- 避免一个线程在所内同时占用多个资源
- 尝试使用定时锁
- 对于数据库锁，加锁和解锁在一个数据库链接里

操作系统中的通信进程通信方式

1. 管道 pipe
2. 信号 signal
3. 消息队列 message queue
4. 共享内存 shared memory
5. 信号量 semaphore
6. 套接字 socket

# 类延迟加载

## 双重检查锁

- 问题

    ```java
    public class DoubleCheckedLocking {
        private DoubleCheckedLocking doubleCheckedLocking;
        //    双重检查锁（存在问题）
        public DoubleCheckedLocking getInstance2(){
            if (doubleCheckedLocking == null){
                synchronized (DoubleCheckedLocking.class){
                    if (doubleCheckedLocking == null){
                        doubleCheckedLocking = new DoubleCheckedLocking();
                    }
                }
            }
            /**
             * 执行到此可能存在问题：
             *  1.doubleCheckedLocking可能未初始化结束
             *      类初始化
             *      {
             *          malloc()//分配空间-1
             *          init()//初始化-2
             *          changeRef()//修改指针指向分配空间-3
             *      }
             *      上述，2,3并无绝对顺序，并且2,3间的重排序并不违反Java语言规范的要求,但此时返回的实例却不是一个正确的实例
             * */
            return doubleCheckedLocking;
        }
    }
    ```

- 优化（这种方式相对于类加载的初始化操作而言操作，还可以延迟初始化**实例字段**）

    ```java
       //    双重检查锁-优化
        /**
         * 具体操作：使用 volatile 修饰变量，确保其初始化先于instance指针对地址的指向
         * */
        private volatile DoubleCheckedLocking doubleCheckedLocking2;
    
        public DoubleCheckedLocking2 getInstance3(){
            if (doubleCheckedLocking2 == null){
                synchronized (DoubleCheckedLocking.class){
                    if (doubleCheckedLocking2 == null){
                        doubleCheckedLocking2 = new DoubleCheckedLocking2();
                    }
                }
            }
            return doubleCheckedLocking2;
        }
    ```

## 初始化

> 处理本质是不阻止类初始化过程的重排序，而是在初始化结束之前并不暴露对象的引用

```java
public class ClassInit {
    //定义在：class中的静态变量会在类初始化时交由JVM操作（而这是线程安全的,初始化时会去获取类的初始化锁）
    private static class InstanceHolder{
        public static ClassInit classInit = new ClassInit();
    }
    //获取实例方法：直接调用静态内部类中的静态实例即可
    public static ClassInit getInstance(){
        return InstanceHolder.classInit;
    }
}
```

# Java中的锁

## 历程

1.5之前依靠synchronized实现锁功能

1.5之后并发包里增加了LOCK接口

- 提供与Synchronized相似的锁功能，也支持对锁进行操作，但需要显示操作，另外还提供多种Synchroized不具备的同步特性

LOCK提供的特性

1. 尝试非阻塞获取锁：尝试获取锁，若无其他线程拿到，则获取成功
2. 获取锁的线程能被中断：
3. 超时获取锁：限制获取锁的时间，超时未成功则返回

```java
public interface Lock {
    void lock(); //获取锁，获取成功后返回
    void lockInterruptibly() throws InterruptedException;//可中断获取锁，所获取过程中可以中断
    boolean tryLock();//非阻塞获取锁，调用立刻返回结果，成功true,失败false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;//超时获取锁
    //方法结束情况：1.得到锁；2.被中断；3.超时时间结束返回false
    void unlock();//释放锁
    Condition newCondition();//等待通知组件,获取锁后可调用
}
```

## AQS队列同步器

> AbstractQueuedSynchronizer(AQS)
>
> int类型成员变量，内置FIFO队列完成资源获取线程的排队工作

### 使用方式

继承，实现同步方法设置对同步状态的修改，关键是对**同步器**（实现锁的关键，基于模板方法设计（抽象类定义了执行流程，但真正实执行时是依照此流程调用子类的具体实现），此同步器未实现任何接口，其内定义了一些基础方法）的实现，正因为其只定义了抽象方法，所以具体这个同步器是采用独占还是共享取决于实现者。

同步器常用可重写方法：

- tryAcquire()：独占式获取锁，（实现：先查询当前状态是否符合预期，在CAS设置同步状态）
- tryRelease()：独占式释放锁
- tryAcquireShared()：共享式获取同步状态，返回值大于等于0则获取成功，否则获取失败
- tryReleaseShared()：共享式释放锁
- isHeldExclusively()：当前同步器是否在独占模式下被线程占用

### 实现

#### 同步队列

- 同步器中保留两个引用
    - head（头结点）
    - tail（尾结点）
- 双端FIFO队列
    - 失败：获取同步状态失败，构建Node（当前线程信息+等待信息），加入同步队列，阻塞当前线程
    - 再次尝试：同步状态释放时首节点线程唤醒

入队列：尾插法（compareAndSetTail()）

出队列：首节点释放同步状态唤醒Next节点，Next节点获取同步状态成功设置自己为首节点

### 重入锁

> ReentrantlLock，但关键字synchronized支持隐式重入

重入：获取锁线程多次重入（每次获取，计数自增）

释放：（计数自减为0）锁释放

#### 公平，非公平锁

公平锁：锁的获取遵循严格的先后限制

非公平锁：非严格显示，一个线程可能多次获取同一个锁（这会导致“饥饿”状态的出现）

默认采用：非公平锁，饥饿出现的是少数，大量并发情况下公平锁的上下文切换次数远高于非公平锁

#### 读写锁

接口：ReadWriteLock

实现：并发包提供ReentrantReadWriteLock

状态设计：整型变量，高16位用于读状态记录，低16位用于写状态记录

- 写状态：当前状态 & 0x0000FFFF(获取)，当前状态+1（修改）
- 读状态：当前状态>>>16（获取），当前状态+1<<<16（修改）

写锁：支持重入的排它锁

读锁：支持重入的共享锁

锁降级：写锁降级--->读锁

# Java中的多线程

## 概述

- 启动一个简单的mian打印操作，会启动其他的多个线程，包括给JVM发送消息的，调用对象finalize方法的，清除引用的等等。

### 采用多线程原因

- 处理核心越来越多
- 响应时间更快
- 编程模型更加友好

### 线程优先级

- priority（取值范围：0-9，默认5）

### java中的线程状态

- 运行：RUNNABLE(RUNNING),READY
- 等待：WAITING
- 超时等待：TIMED_WAITING
- 阻塞：BLOCKED
- 终止：TERMINATED

### 线程状态切换

1. 实例化（new）
2. 运行（Thread.start())
    1. 就绪
        1. 运行=>就绪（yield())
        2. 运行<=就绪（系统调度)
    2. 等待
        1. 运行=>等待（Object.wait(), Object.join(), LockSupport.park())
        2. 运行<=等待（Object.notify(), Object.notifyAll(), LockSupport.unpark(Thread)）
    3. 超时等待
        1. 运行=>超时等待（Thread.sleep(long), Object.wait(long), Thread.join(long), LockSupport.parkNanos(), LockSupport.parkUtil()）
        2. 运行<=超时等待（Object.notify(), Object.notifyAll(), LockSupport.unpark(Thread)）
    4. 阻塞
        1. 运行=>阻塞（进入锁synchroized块）
        2. 运行<=阻塞（获取锁成功）
3. 终止（执行完成）

## Daemon线程

> 支持性线程，JVM中无Daemon线程时会退出，设置线程为Daemon需要在启动之前设置

JVM退出时，Daemon中的finally块并不一定会执行

## 线程使用

### 构建

> 线程创建会继承父线程的以下信息：是否Daemon,优先级，加载资源的contextClassLoader和可继承的ThreadLocal

构建方式

1. 继承Thread（无返回值）

    - 优点：实现简单，获取当前线程的信息方便
    - 缺点：多个线程**无法共享**一份资源

2. 实现Runnable接口（无返回值）

    - 优点：多个线程可以**共享**一个target对象

    - 缺点：实现稍微复杂一点，获取当前线程Thread.currentThread()

3. 实现Callable接口+Future接口（有返回值）

    > jdk1.5开始提供
    >
    > Runnable接口的增强版，但并不是Runnable的子接口（无法作为Thread的target），其中call方法是线程执行体

    call()：可以有返回值，可以声明抛出异常，但正因为是线程执行体没有办法直接调用

    Future接口提供一个FutureTask的实现类，FutureTask实现了Runnable接口，可作为Thread类的target

    - 可通过Future接口中定义的方法操作与其所关联的Callable类

### 启动

start()	当前线程同步告知虚拟机，若线程规划器空闲，则立刻启动调用start()的线程

### 中断

- 其他线程中断此线程interrupt()
- 线程检查自身是否被中断isInterrupted()
    - 线程终结，此方法返回false
    - 需要抛出InterruptedException，JVM会将线程inerrupt的标志位清除，随后抛出异常，此后调用判断方法将返回false

### 终止

- 通过中断或cancle()方法这种操作线程的方式可以使线程在终止时有机会清理资源

## 线程通信模型

### 关键字

- volatile
- synchroized

### 等待/通知

- 可使用操作，唤醒在某个对象上等待的线程（notify()唤醒一个，notifyAll()唤醒所有）

### 管道输入和输出流

- 用于线程间交互，传输媒介为内存4

### Join()

- thread.join(),线程thread终止后才可返回

### ThreadLocal

- 可以附在线程上的K-V对，k = ThreadLocal, v = 任意对象

# JaJava 内存模型

> 并发编程关键问题
>
> - 线程通信
> - 线程同步

## 线程通信

共享内存：线程通过读写内存中公共状态进行隐式通信

消息传递：线程之间通过消息进行显式通信

## 线程同步

> 用于控制不同线程间操作发生的相对顺序

共享内存并发模型中：显式同步

消息传递并发模型中：隐式同步



# 原子操作实现原理

- 1. 
