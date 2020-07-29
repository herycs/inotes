# 锁关键字

## 多处理器同步

Java 的CAS会使用现代处理器提供的高效机器级别的原子指令，这些原子指令以原子方式对内存执行读-改-写的操作

vlatile的读写 + CAS可以实现线程间通信

# volatile

> 若一个字段被声明为volatile，Java线程内存模型会确保所有线程看到的这个变量的值是一一致的

语义：当一个程序修改一个变量时，另外一个线程可以读到 这个修改的值（可以理解为轻量级的synchronized）

特征

- 可见性：对一个volatilede的读，总是能看到对这个变量的写入
- 原子性：读写操作具有原子性，++这种复合操作不具有原子性

读写

- volatile的写和锁的释放具有相同的内存语义

    写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存

- volatile的读和锁的获取具有相同的内存语义

    读一个volatile变量时，JMM会将该线程对应的本地内存置为无效

## 内存语义实现

> 基于保守策略

### 为什么要有？

JSR-133前volatile并没有内存语义，**为了提供比锁更加轻量级的线程之间的通信机制**

### 内存屏障

load---读

store---写

写操作：StoreStore | volatile写操作 | StoreLoad

> 此处写操作后增加了StoreLoad，目的是避免volatile写操作后面的volatile读写重排序
>
> 编译器往往无法判断volatile写操作后是否需要增加屏障，JMM采用保守策略，在 每个volatile写后或voletile读前增加 StoreLoad可达到目的，最终考虑性能(通常都是一个**写**，多个线程**读**)选择在写后增加StoreLoad

读操作：LoadLoad | vlatile读操作 | LoadStore

编译解析：对于volatile变量，转为汇编时会添加lock指令

lock指令作用：

- 当前处理器缓存行数据写回到系统内存
    - Lock前缀指令导致在执行指令期间，处理器可以独占任何共享内存。P6和目前的处理器，Lock指令一般不锁总线，若访问区域缓存在处理器内部，则锁住这块内存区域的缓存并写回到内存，使用缓存一致性协议确保修改的一致性，也就是“缓存锁定”
- 其他CPU中缓存的当前地址变无效

> intel手册中对lock前缀的说明：（来自Java并发编程的艺术）
>
> 1. Pentium4,IntelXeon,P6处理器开始，Intel使用缓存锁定确保指令执行的原子性
> 2. 禁止该指令，与之前和之后的读写指令重排序
> 3. 写缓冲区数据刷新到内存中

### 执行过程

- 处理器不直接和内存通信，先执行写回内存的操作
- 为确保数据的一致性，使用缓存一致性协议，嗅探总线上的值判断当前值是否过期（若过期就设置当前处理器缓存行为无效状态，当对这个数据修改时就会重新从系统内存中读取）

## 编译器&处理器的处理

> 获取锁之前会读volatile类型state变量

编译器

- 不对volatile读与volatile读后的任意内存操作重排序
- 不对volatile写与volatile写后的任意内存操作重排序

处理器

> 对应x86的常见处理器

- 依据源码（automiv_windows_x86.inline.hpp）的显示，处理器会依据当前处理器类型决定是否使用LOCK前缀（单处理器中其本身就会维护处理器内的顺序一致性，LOCK屏障多此一举），这个操作也是一定程度的优化（即，在不需要的时候消除锁）

正因为以上操作使用了volatile，而volatile又具有其特性，CAS借助对volatile的操作实现了volatile所具有的内存语义

总结

对于锁的获取与释放的内存语义的实现方式

1. 利用volatile变量的写——读具有的内存语义
2. 利用CAS所附带的volatile读和volatile写的内存语义实现**线程间的通信**

## volatile使用优化

- 追加字节使其满足当前处理设备的高速缓存行

> Linked-TransferQueue中就采用了字节填充的操作，通过追加了15个Object类型的变量（每个引用4字节，共计60字节），父类的value加上一共64字节，以此确保队列的首尾不会被加载到同一个缓存行，这样修改头尾其中一个时锁住缓存行的过程中就不会导致另一个也无法操作。

- 不适用情况：共享变量不会频繁的写

# synchronized

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



### 实现

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728212946575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

Synchronized在线程进入ContentionList时，等待的线程会先尝试自旋获取锁，如果获取不到就进入ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占OnDeck线程的锁资源。

### 锁的优化

之前也提到了，synchronized关键字对语句块的处理采用了两个字节码指令，而这两个指令在使用上代价是比较高，jdk1.6中锁具有四种状态，分别是：无锁--->偏向锁--->轻量级锁--->重量级锁，随着竞争状态升级。偏向锁--->轻量级锁过程不可逆。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728221720112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

#### 偏向锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072822180212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

##### 获取锁

> 当一个线程访问同步块代码时

1. 对象头，栈帧中记录锁偏向的**线程ID**
2. 后续进入此同步代码块，校验Mark Word中是否存储有当前线程ID
    1. 有->当前线程获取锁成功
    2. 无->测试锁标志位是否为1（偏向锁标志）
        1. 无->使用CAS竞争锁
        2. 有->尝试使用CAS将对象头的偏向锁设置为当前线程（简单理解就是修改偏向的线程为当前线程）

##### 撤销

当**竞争**出现，撤销锁（也就是当有了别的线程需要此偏向锁，上一个线程才撤销对偏向锁的占有），被动发生

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

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728221827501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

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

自旋周期

1.5写死->1.6提供参数可设置->1.7去掉参数，直接交由JVM控制

JVM对于自旋周期的选择，jdk1.5这个限度是一定的写死的，在1.6引入了适应性自旋锁，适应性自旋锁意味着自旋的时间不在是固定的了，而是由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定，基本认为一个线程上下文切换的时间是最佳的一个时间，同时JVM还针对当前CPU的负荷情况做了较多的优化

自旋时间的最坏情况：CPU的存储延迟（CPU A 存储了一个数据，到 CPU B 得知这个数据直接的时间差），自旋时会适当放弃线程优先级之间的差异。

> [参考博客](https://blog.csdn.net/zqz_zqz/article/details/70233767)
>
> JDK1.6中-XX:+UseSpinning开启	-XX:PreBlockSpin=10 为自旋次数

- 如果平均负载小于CPUs则一直自旋
- 如果有超过(CPUs/2)个线程正在自旋，则后来线程直接阻塞
- 如果正在自旋的线程发现Owner发生了变化则延迟自旋时间（自旋计数）或进入阻塞
- 如果CPU处于节电模式则停止自旋
- 自旋时间的最坏情况是CPU的存储延迟（CPU A存储了一个数据，到CPU B得知这个数据直接的时间差）
- 自旋时会适当放弃线程优先级之间的差异

#### 重量级锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728221842694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

1. 线程尝试获取锁
    1. 成功->获取锁成功
    2. 失败->阻塞，等待被锁释放时被唤醒，再次尝试获取锁

### 执行过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728221623453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

1. 检测Mark Word里面是不是当前线程的ID，如果是，表示当前线程处于偏向锁
2. 如果不是，则使用CAS将当前线程的ID替换Mard Word，如果成功则表示当前线程获得偏向锁，置偏向标志位1
3. 如果失败，则说明发生竞争，撤销偏向锁，进而升级为轻量级锁。
4. 当前线程使用CAS将对象头的Mark Word替换为锁记录指针，如果成功，当前线程获得锁
5. 如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
6. 如果自旋成功则依然处于轻量级状态。
7. 如果自旋失败，则升级为重量级锁。

### 锁对比

| 锁       | 优点                             | 缺点                            | 适用                           |
| -------- | -------------------------------- | ------------------------------- | ------------------------------ |
| 偏向锁   | 加锁，解锁无额外消耗             | 锁被竞争，则会存在锁撤销的消耗  | 只有一个线程访问同步块         |
| 轻量级锁 | 竞争线程不阻塞，程序响应速度可观 | 获取不到锁的线程会自旋，消耗CPU | 追求响应时间，同步块执行时间短 |
| 重量级锁 | 竞争过程不自旋，不耗CPU          | 未获取锁的线程阻塞，响应时间慢  | 追求吞吐量，同步块执行时间长   |

# final

### 重排序规则

#### 编译器和处理器

1. 构造函数的写入，随后将这个构造对象的引用赋值给一个引用变量，这两个操作不许进行重排序（确保对象引用在暴露给其他线程之前已被正确初始化）
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作不许重排序

#### 写final域

- 禁止将final域的**写**重排序到**构造函数**之外

实现

- JMM：禁止将final域的写重排序到构造函数之外
- 编译器：增加StoreStore（final写之后，return之前）

#### 读final域

- JMM：读对象引用和初次读该对象包含的final域
- 编译器：增加LoadLoad(final域读操作之前)

### 内存语义

旧内存模型中，final域的值*可能被改变*（重排序未做保证，当线程去看是可能是默认值，随后初始化，导致两次读到的初始化值不同，String 的内部定义就是使用final修饰的）

JSR-133中，通过增加重排序的规则确保其初始化结果是确定且正确的（只要线程可以拿到初始化后的对象，那么它的final域一定是初始化的值）