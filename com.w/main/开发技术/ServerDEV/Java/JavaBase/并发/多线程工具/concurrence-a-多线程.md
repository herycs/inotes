# Java中的多线程

# 概述

启动一个简单的mian打印操作，会启动其他的多个线程，包括给JVM发送消息的，调用对象finalize方法的，清除引用的等等。

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

# 线程使用

### 创建

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

### 线程通信

共享内存：线程通过读写内存中公共状态进行隐式通信

消息传递：线程之间通过消息进行显式通信

### 线程同步

> 用于控制不同线程间操作发生的相对顺序

共享内存并发模型中：显式同步

消息传递并发模型中：隐式同步

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

#### 意义

ThreadLocal提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程

#### 底层

每个ThreadLocal类都创建一个Map，然后用线程的ID threadID作为Map的key，要存储的局部变量作为Map的value，这样就能达到各个线程的值隔离的效果。

JDK8 ThreadLocal的设计是：每个Thread维护一个ThreadLocalMap哈希表，这个哈希表的key是ThreadLocal实例本身，value才是真正要存储的值Object。

1） 这样设计之后每个Map存储的Entry数量就会变小，因为之前的存储数量由Thread的数量决定，现在是由ThreadLocal的数量决定。

2） 当Thread销毁之后，对应的ThreadLocalMap也会随之销毁，生命周期与线程相同，能减少内存的使用。

#### ThreadLocalMap

的底层实现是一个定制的自定义HashMap哈希表，核心组成元素有：

1 ) Entry[] table;：底层哈希表 table, 必要时需要进行扩容，底层哈希表 table.length 长度必须是2的n次方。

其中Entry[] table;哈希表存储的核心元素是Entry，

Entry（Entry继承了弱引用 WeakReference）

- ThreadLocal<?> k；：当前存储的ThreadLocal实例对象

- Object value;：当前 ThreadLocal 对应储存的值value

2 ) int size;：实际存储键值对元素个数 entries

3 ) int threshold;：下一次扩容时的阈值，阈值 threshold = 底层哈希表table的长度 len * 2 / 3。

当size >= threshold时，遍历table并删除key为null的元素，如果删除后size >= 

threshold*3/4时，需要对table进行扩容

#### 内存泄漏

在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。

由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏

但是使用弱引用可以多一层保障：弱引用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用get(),set(),remove()的时候会被清除。

ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。

每次使用完ThreadLocal，都调用它的remove()方法，清除数据。

#### 应用

并发：以空间换时间”的方式

数据存储：线程隔离

在Spring中，绝大部分Bean都可以声明为singleton作用域。就是因为Spring对一些Bean（如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等）中非线程安全状态采用ThreadLocal进行处理，让它们也成为线程安全的状态，因此有状态的Bean就可以在多线程中共享了。