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

## 队列同步器

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

# Java 内存模型

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

## 通信方式

java中并发采用共享内存模型，线程间通信采用隐式通信

## Javan内存模型（JMM）

> JMM负责线程间通信，并决定一个线程对共享变量的操作何时对另一个线程可见
>
> java中实例域，静态域，数组元素都存储在堆内存（线程间共享）

JMM管控了线程的本地内存（JMM中抽象概念，涵盖缓存，写缓冲区，寄存器，其他硬件和编译器优化）到主内存间的数据传递过程

## 重排序

### 流程

1. 编译器重排序（*编译器重排序层面*）
2. 指令级重排序（*处理器重排序层面*，现代处理器采用指令级并行技术，操作间无数据依赖，则可以调整操作指令执行顺序）
3. 内存系统重排序（*处理器重排序层面*，处理器使用缓存和读写缓冲区，这种情况下加载存储操作可能是乱序的）

需要注意的是：多个处理器操作的数据依赖性不被编译器和处理器考虑

### JMM应对策略

提供内存屏障（Memory Barries），禁止一些特定类型的（编译器，处理器）重排序

### 处理器重排序规则

| 处理器    | Load-Load | Load-Store | Store-Store | Store-Load | 数据依赖 |
| --------- | --------- | ---------- | ----------- | ---------- | -------- |
| SPARC-TSO | N         | N          | N           | Y          | N        |
| x86       | N         | N          | N           | Y          | N        |
| IA64      | Y         | Y          | Y           | Y          | N        |
| PowerPC   | Y         | Y          | Y           | Y          | N        |

内存屏障：Load=>确保数据的装载，Store=>确保数据刷新（缓存数据刷新到主内存，也就是保证此数据对处理器可见）

### 内存屏障类型

LoadLoad, LoadStore, StoreStore, StoreLoad

示例：Load1， LoadStore, Store1，Load1的数据装载先于Store1的数据刷新

### 重排序规则

> jdk1.5开始采用JSR-133内存模型

#### happens-before

> 若存在A的结果对B可见则，AB操作必须有happens-before关系
>
> happens-before仅要求前一个操作（执行结果）对后一个操作可见，并不确保两个操作一定要先后执行

##### 相关规则

程序：线程中的每个操作happens-before于其任意后续操作

监视器锁规则：解锁happens-before于加锁

volatile变量：volatile域的写happens-before于读

传递：A happens-before B, B happens-before C，则A happens-before C

##### JMM的处理

会改变执行结果的重排序：JMM要求编译器，处理器必须禁止重排序操作

不会改变执行结果的重排序：JMM不做限制

#### as-if-serial

##### 语义

无论如何重排序，（单线程）操作结果不可被改变，多线程情况下结果就不得保证了

## 顺序一致性

JMM对正确同步的多线程程序提供保证：若程序是正确同步（广义，包括对常用同步原语的使用），程序的执行将具有顺序一致性。

### 特征

1. 线程操作依据程序顺序执行
2. （无论同步与否）所有线程都只能看到一个单一的操作执行顺序（顺序模型中每个操作必须原子执行，且立刻对所有线程可见）

### 同步程序

JMM在进入和退出临界区时做一些操作，确保程序在和两个时间节点和顺序一致性模型的内存视图一样

### 非同步程序

JMM提供最小安全性，确保线程读取到的值不会凭空生成（之前线程写入值或者是默认值0，null，false）

在堆上分配对象时，内存空间清零，分配对象，JMM内部会对上述两个操作进行同步

JMM：

- 不保证单线程执行顺序（存在重排序）
- JMM不保证所有线程的操作执行顺序一致（存在重排序）
- 不保证对对64位long型和double型**写**操作的原子性

顺序一致性模型：

- 保证单线程的执行顺序
- 保证所有线程所见顺序一致
- 保证对64位long型和double型读写操作具有原子性

JSR-l33之前：long/double类型的读写拆分为32位的**读写**

JSR-l33：64位long/double类型变量写操作拆分为两个32位**写**操作，任意读操作都要保证原子性

# Java中的技术支持

## volatile

- 若一个字段被声明为volatile，Java线程内存模型会确保所有线程看到的这个变量的值是一一致的

### 语义

- 当一个程序修改一个变量时，另外一个线程可以读到 这个修改的值（可以理解为轻量级的synchronized）

### 特征

- 可见性：对一个volatilede的读，总是能看到对这个变量的写入
- 原子性：读写操作具有原子性，++这种复合操作不具有原子性

### 读写

- volatile的写和锁的释放具有相同的内存语义

    写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存

- volatile的读和锁的获取具有相同的内存语义

    读一个volatile变量时，JMM会将该线程对应的本地内存置为无效

### 内存语义实现

> 基于保守策略
>
> JSR-133前volatile并没有内存语义，**为了提供比锁更加轻量级的线程之间的通信机制**

写操作

StoreStore | volatile写操作 | StoreLoad

> 此处写操作后增加了StoreLoad，目的是避免volatile写操作后面的volatile读写重排序
>
> 在 每个volatile写后或voletile读前增加 StoreLoad可达到目的，最终考虑性能(通常都是一个写，多个线程读)选择在写后增加StoreLoad

读操作

LoadLoad | vlatile读操作 | LoadStore

编译解析

- 对于volatile变量，转为汇编时会添加lock指令

lock指令作用：

- 当前处理器缓存行数据写回到系统内存
    - Lock前缀指令导致在执行指令期间，处理器可以独占任何共享内存。P6和目前的处理器，Lock指令一般不锁总线，若访问区域缓存在处理器内部，则锁住这块内存区域的缓存并写回到内存，使用缓存一致性协议确保修改的一致性，也就是“缓存锁定”
- 其他CPU中缓存的当前地址变无效

> intel手册中对lock前缀的说明：（来自Java并发编程的艺术）
>
> 1. Pentium4,IntelXeon,P6处理器开始，Intel使用缓存锁定确保指令执行的原子性
> 2. 禁止该指令，与之前和之后的读写指令重排序
> 3. 写缓冲区数据刷新到内存中

执行

- 处理器不直接和内存通信，先执行写回内存的操作
- 为确保数据的一致性，使用缓存一致性协议，嗅探总线上的值判断当前值是否过期（若过期就设置当前处理器缓存行为无效状态，当对这个数据修改时就会重新从系统内存中读取）

### java的CAS

> 获取锁之前会读volatile类型state变量

编译器

- 不对volatile读与volatile读后的任意内存操作重排序
- 不对volatile写与volatile写后的任意内存操作重排序

处理器

> 对应x86的常见处理器

- 依据源码（automiv_windows_x86.inline.hpp）的显示，处理器会依据当前处理器类型决定是否使用LOCK前缀（单处理器中其本身就会维护处理器内的顺序一致性，LOCK屏障多此一举），这个操作也是一定程度的优化（即，在不需要的时候消除锁）

正因为以上操作使用了volatile，而volatile又具有其特性，CAS借助对volatile的操作实现了volatile所具有的内存语义

自旋周期

> [参考博客](https://blog.csdn.net/zqz_zqz/article/details/70233767)

JDK1.6中-XX:+UseSpinning开启
-XX:PreBlockSpin=10 为自旋次数
JDK1.7后，去掉此参数，由jvm控制

- 如果平均负载小于CPUs则一直自旋
- 如果有超过(CPUs/2)个线程正在自旋，则后来线程直接阻塞
- 如果正在自旋的线程发现Owner发生了变化则延迟自旋时间（自旋计数）或进入阻塞
- 如果CPU处于节电模式则停止自旋
- 自旋时间的最坏情况是CPU的存储延迟（CPU A存储了一个数据，到CPU B得知这个数据直接的时间差）
- 自旋时会适当放弃线程优先级之间的差异
    

### volatile使用优化

- 追加字节使其满足当前处理设备的高速缓存行

> Linked-TransferQueue中就采用了字节填充的操作，通过追加了15个Object类型的变量（每个引用4字节，共计60字节），父类的value加上一共64字节，以此确保队列的首尾不会被加载到同一个缓存行，这样修改头尾其中一个时锁住缓存行的过程中就不会导致另一个也无法操作。

- 不适用情况：共享变量不会频繁的写

## synchronized

### 作用

- 确保互斥性：线程互斥的访问同步代码的
- 确保可见性：共享变量的变动及时可见
- 确保有序：针对指令重排序

### 使用方式

方法

- 普通方法——锁当前实例对象
- 静态方法——锁当前类的Class对象

语句

- 同步方法块——锁Synchonized括号中配置的对象

### 具体实现

**测试代码**

```java
public class SyncDemo {
    public synchronized void testM() {System.out.println("method");}
    public synchronized static void testS() {System.out.println("static method");}
    public void testC() {
        synchronized (this) {System.out.println("class");}
    }
}
```

锁=>普通方法

- 源码

    ```java
    public synchronized void test() {System.out.println("method");}
    ```

- 字节码

    ```java
    public synchronized void testM();
        descriptor: ()V
        flags: ACC_PUBLIC, ACC_SYNCHRONIZED//此处增加了标志位[ACC_SYNCHRONIZED]
        Code:
          stack=2, locals=1, args_size=1
             0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             3: ldc           #3                  // String method
             5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
             8: return
    ```

锁=>静态方法

- 源码

    ```java
    public synchronized static void testS() {System.out.println("static method");}
    ```

- 字节码

    ```java
    public static synchronized void testS();
        descriptor: ()V
        flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED//增加了标志位[ACC_SYNCHRONIZED]
        Code:
          stack=2, locals=0, args_size=0
             0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             3: ldc           #5                  // String static method
             5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
             8: return
    ```

锁=>方法块

- 源码

    ```java
    public void testC() {
        synchronized (this) {System.out.println("class");}
    }
    ```

- 字节码

    ```java
    public void testC();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=2, locals=3, args_size=1
             0: aload_0
             1: dup
             2: astore_1
             3: monitorenter//enter内存屏障，线程执行到此处会尝试获取对象的monitor
             4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             7: ldc           #6                  // String class
             9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
            12: aload_1
            13: monitorexit//方法正常结束，退出内存屏障
            14: goto          22//跳转到return完成方法调用
            17: astore_2
            18: aload_1
            19: monitorexit//执行异常，退出内存屏障
            20: aload_2
            21: athrow
            22: return
    ```

经过以上对比可以发现

当synchronized修饰方法，则会在编译为字节码的时候，将方法的synchronized标志位置1，而在JVM层面的反汇编代码的中的表示就是flag字段增加了{ACC_SYNCHRONIZED}的标志位

当synchronized修饰语句块，则会在语句块的开始和结束位置增加{monitorenter，monitorexit}，而这两个字节码指令依赖于操作系统的Lock操作，Lock时，挂起线程并切换至内核态，这种操作会非常的耗时，高并发情况下，使用的代价自然是很昂贵的。

### 存在位置

Java对象头中,

> 32bit系统：1 word = 4byte = 32bit

- 对象是数组类型，虚拟机采用3(Word)存储对象头
- 非数组类型，采用2(Word)存储对象头

java对象头长度

| 长度     | 内容                   | 说明                             |
| -------- | ---------------------- | -------------------------------- |
| 32/64bit | MarkWord               | 对象hashCode或锁信息             |
| 32/64bit | Class Metadata Address | 指向对象类型数据的指针           |
| 32/32bit | Array Length           | 数组长度（当前对象是数组时使用） |

对象头存储结构，在不同锁情况下Mark Word的变化

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200607134905150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

64bit虚拟机下Mark Word存储结构

<table>
    <tr>
    	<td rowspan="2">锁状态</td>
        <td>25bit</td>
        <td>31bit</td>
        <td>1bit</td>
        <td>4bit</td>
        <td>1bit</td>
        <td>2bit</td>
    </tr>
    <tr>
    	<td></td>
        <td></td>
        <td>cms_free</td>
        <td>分代年龄</td>
        <td>偏向锁</td>
        <td>锁标志位</td>
    </tr>
    <tr>
    	<td>无锁</td>
        <td>unused</td>
        <td>hashCode</td>
        <td></td>
        <td></td>
        <td>0</td>
        <td>01</td>
    </tr>
    <tr>
    	<td>偏向锁</td>
        <td colspan="2">ThreadID(54bit)+Epoch(2bit)</td>
        <td></td>
        <td></td>
        <td>1</td>
        <td>01</td>
    </tr>
</table>

### 锁的优化

之前也提到了，synchronized关键字对语句块的处理采用了两个字节码指令，而这两个指令在使用上代价是比较高，jdk1.6中锁具有四种状态，分别是：无锁--->偏向锁--->轻量级锁--->重量级锁，随着竞争状态升级。偏向锁--->轻量级锁过程不可逆。

#### 偏向锁

##### 获取锁

> 当一个线程访问同步块代码时

1. 对象头，栈帧中记录锁偏向的**线程ID**
2. 后续进入此同步代码块，校验Mark Word中是否存储有当前线程ID
    1. 有->当前线程获取锁成功
    2. 无->测试锁标志位是否为1（偏向锁标志）
        1. 无->使用CAS竞争锁
        2. 有->尝试使用CAS将对象头的偏向锁设置为当前线程（简单理解就是修改偏向的线程为当前线程）

##### 撤销

当竞争出现，撤销锁（也就是当有了别的线程需要此偏向锁，上一个线程才撤销对偏向锁的占有）

撤销的时机：全局安全点

流程

1. 暂停拥有偏向锁的线程，检查持有偏向锁的线程是否存活
    1. 否->设置其对象头为无锁
    2. 是->拥有偏向锁的线程的栈执行，遍历偏向对象锁记录
        - 栈中的锁记录和对象头的MarkWord有以下几个情况
            1. 偏向于其他线程
            2. 回复到无锁
            3. 标记对象不适合作为偏向锁
2. 唤醒暂停的线程

##### JVM参数

```java
偏向锁在应用启动几秒之后会启动
//关闭延迟：
-XX:BiasedLockingStartupDelay0
//关闭偏向锁,程序默认进入轻量级锁
-XX:-UseBiasedLocking=false
```

#### 轻量级锁

加锁

> 当线程执行到同步块

1. JVM在当前线程锁空间中创建空间（用于存储锁记录），将对象的MarkWord复制到锁记录中（官方叫法：DisplacedMarkWord）
2. 线程尝试使用CAS替换：对象头中的MarkWord--->指向锁记录的指针
    1. 成功->获取锁成功
    2. 失败->尝试使用自旋获取锁

解锁

1. <u>原子CAS</u>操作将DisplacedMarkWord换回到对象头中
    1. 成功->无竞争
    2. 失败->有竞争，升级为重量级锁

#### 自旋锁

在轻量级锁获取失效后采用自旋的方式避免被挂起，原因，锁的占用时间并不长，若挂起，锁释放后再从挂起状态恢复会得不偿失，同样这也告诉我们在使用对轻量级锁使用时，尽量避免同步块的时间时间过长，毕竟自旋很耗费资源

自旋消耗CPU，避免无用自旋，升级为重量级锁后不再降级到轻量级锁，升级为重量级锁，当成为重量级锁后，其他企图获取次锁的线程会被**阻塞**，避免不必要的耗费，当锁释放后会唤醒这些线程，进行再一次的锁竞争

#### 重量级锁

1. 线程尝试获取锁
    1. 成功->获取锁成功
    2. 失败->阻塞，等待被锁释放时被唤醒，再次尝试获取锁

#### 锁消除

> [参考博客](https://yq.aliyun.com/articles/666316)

同步块不一定会被多个线程使用到，当只有一个线程使用同步块，这时再对进行加锁，解锁操作有些多此一举了，Java虚拟机JIT编译时通过上下文扫描，会除去不存在公共资源竞争的锁

#### 锁对比

| 锁       | 优点                             | 缺点                            | 适用                           |
| -------- | -------------------------------- | ------------------------------- | ------------------------------ |
| 偏向锁   | 加锁，解锁无额外消耗             | 锁被竞争，则会存在锁撤销的消耗  | 只有一个线程访问同步块         |
| 轻量级锁 | 竞争线程不阻塞，程序响应速度可观 | 获取不到锁的线程会自旋，消耗CPU | 追求响应时间，同步块执行时间短 |
| 重量级锁 | 竞争过程不自旋，不耗CPU          | 未获取锁的线程阻塞，响应时间慢  | 追求吞吐量，同步块执行时间长   |

## final

### 重排序规则

#### 编译器和处理器

1. 构造函数的写入，随后将这个构造对象的引用赋值给一个引用变量，这两个操作不许进行重排序（确保对象引用在暴露给其他线程之前已被正确初始化）
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作不许重排序

#### 写final域

- 禁止将final域的写重排序到构造函数之外

实现

- JMM：禁止将final域的写重排序到构造函数之外
- 编译器：增加StoreStore（final写之后，return之前）

#### 读final域

- JMM：读对象引用和初次读该对象包含的final域
- 编译器：增加LoadLoad(final域读操作之前)

### 内存语义

旧内存模型中，final域的值可能被改变（重排序未做保证，当线程去看是可能是默认值，随后初始化，导致两次读到的初始化值不同，String 的内部定义就是使用final修饰的），JSR-133中通过增加重排序的规则确保其初始化结果是确定且正确的（只要线程可以拿到初始化后的对象，那么它的final域一定是初始化的值）

# 原子操作实现原理

## 处理器操作

- 总线锁

    处理器发出LOCK # 信号，收到信号的处理器的请求将被阻塞住

- 缓存锁定

    同一时刻确保对某个内存地址的操作是原子的

    内存区域若被处理器缓存在缓存行中，且在LOCK期间被锁定，当它执行锁操作写回到内存时，处理器并不声言LOCK #，而是修改内部的内存地址，并允许他的缓存一致性机制（阻止同时修改由两个以上的处理器缓存的内存区域数据，当其他处理器写会已被锁定的缓存行的数据时，会导致缓存行无效）来保证操作的原子性

> 特殊不可用缓存锁定的情况
>
> - 数据不能被缓存在处理器内部，操作数横跨多个缓存行
>- 处理器不支持缓存锁定

## Java实现原子操作

### 方式

- 锁+循环CAS
- JVM的CAS操作利用的是处理器提供的CMPXCHG指令实现

提供的工具

- jdk1.5开始，并发包中提供诸如：AtomicBoolean, AtomicInteger这样的原子性更新值的工具

### 问题

ABA问题

- JDK中提供AtomicStampedReference，其中compareAndSet首先检查当前引用，标志是否等于预期引用，标志，都相等是会采用原子操作更新值

循环时间长开销大

- 处理器提供一个pause的指令，指令作用如下
    1. 延迟流水线执行指令，延迟时间取决于实现版本
    2. 避免退出循环时因为内存顺序冲突引起CPU流水线被清空

只能保障一个共享变量的原子操作

- 多个共享变量的原子操作保证

    1. 锁
    2. 多个共享变量变为1个共享变量

        利用jdk中提供AtomicReference保证引用对象之间的原子性，可将多个对象共享变量放置到一个对象中进行CAS操作
