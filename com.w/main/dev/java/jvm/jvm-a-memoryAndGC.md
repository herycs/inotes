# 走进 - JVM - b

# 内存自动管理机制

##  运行时数据区

![image-20191211002910416](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191211002910416.png)

###  程序计数器

- 较小内存空间，字节码解释器执行时基于计数器的值选取下一条需要执行的字节码指令，分支，循环，跳转，异常处理，线程恢复等基础功能。

- Java虚拟机的多线程通过线程轮流切换并分配处理器执行时间实现，故任何一个确定时刻一个处理器（多核处理器的一个内核）只会执行一条线程中的指令。为保证其切换后的正确性，每个线程有独立的计数器。

- 线程正在执行Java方法，则计数器记录正在执行的指令地址，若为本地（Native）方法，则计数器值为空（Undeifined）。

- Java虚拟机规范中唯一没有任何OutOfMemoryError的区域

### 虚拟机栈

- 线程私有，生命周期同线程

- 描述Java方法执行的内存模型，Java 方法执行同时会创建栈帧，存放存储局部变量表，操作数栈，动态链接，方法出入口等信息，方法的调用直至执行----一个栈帧入栈到出栈的过程

- 局部变量表：

    - 存放**编译期**可知的各种数据类型：boolean, byte, char, short, int, long, double, 对象引用（referience类型）和returnAddress类型（指向字节码指令的地址），其中64位long和double占两个slot局部变量空间，其余数据类型占一个

    - 内存空间编译器分配完毕，方法运行期不改变

    - 异常：

        StackOverflowError：线程请求栈深度大于虚拟机允许深度

        OutOfMemoryError：动态扩展获取不到足够内存时抛出

### 本地方法栈

- 为虚拟机使用到的Native方法服务

- 异常：

    StackOverflowError：线程请求栈深度大于虚拟机允许深度

    OutOfMemoryError：动态扩展获取不到足够内存时抛出

###  Java堆

- 线程共享，虚拟机启动时创建，存放对象实例（几乎所有）
- GC器主要管理区域，也称：GC堆（Garbage Colleced Heap）
- Java虚拟机规范要求其内存逻辑上连续即可，一般可扩展，若实例分配未完成，也无法扩展，抛出OutOfmemoryError

###  方法区

- 线程共享

- 存储已被虚拟机加载的类信、常量、静态变量、即时编译器编译后的代码等，又称Non-Heap（非堆）

- 可设定固定或可扩展大小，也可不实现GC

- 此区域GC目标，常量池的回收和对类型的卸载

- 无法实现内存分配需求，抛出OutOfMemoryError

- 运行时常量池（Runtime Constant Pool）

    - 方法区的一部分
    - Class文件除了基本数据信息外还有，常量池（Constant Pool Table）存放编译期的字面量和符号引用，类加载后进入方法区的运行时常量池
    - 动态性，运行期间同样可以将新的常量放入池中
    - 无法再申请内存时抛出OutOfMemoryError

    

## 直接内存（Direct Memory）

- 不属于Java虚拟机定义的内存区域
- JDK1.4加入NIO（New Input/Output）类，基于通道（Channel)和缓冲区（Buffer)的 I/O 方式，可使用Native函数库直接分配堆外内存，通过存储在Java堆中的DirectByteBuffer对象作为这块区域的引用

# 对象

##  对象的创建

###  步骤

- 遇到new指令

    检查其参数能否在常量池中定位到一个类的符号引用

    检查其引用带表的类是否已被加载，解析，初始化过，若没有执行相应的类加载过程

- 为对象分配内存

    - “指针碰撞”：内存规整，指针为标记，一边为空闲区，一边为使用区
        - 代表：Serial, ParNew，带Compact过程的收集器
    - “空闲列表”：内存不规整，空闲列表记录空闲区，使用时依据空闲表分配内存
        - 代表：CMS，基于Mark-Sweep算法的收集器
    - 解决线程分配安全：
        - 分配内存动作同步处理
            - CAS+失败重试保证更新操作的原子性
            - 内存分配动作按照线程划分到不同空间中进行（线程在Java堆中分配小块内存，本地分配缓冲（Thread Local Allocation Buffer, TLAB））,-XX:+/-UseTLAB设定是否使用分配内存

- 初始化

    - 分配到的内存空间初始化为零值（除对象头以外），使用TLAB时，此步骤可提前至TLAB分配时进行

- 设定对象头（虚拟机视角，对象创建完成）

- new 执行--->\<init\>执行（程序员视角，对象创建完成）

## 对象内存模型

> 对象头（Header) + 实例数据（Instance Data) + 对齐填充（Padding）

### 对象头

- 运行时数据（32/64w位系统(未开启指针压缩)中分别为32bit,64bit），Make Word

    设计为非固定数据结构，可依据对象状态复用自己存储空间

- 类型指针（指向类元数据）（并非所有虚拟机实现均有此）

###  实例数据

- 程序中定义的各种字段内容，父类继承，子类中定义

- 存储顺序由： 虚拟机分配策略（FeildsAllocationStyle) + 字段在Java源码中定义顺序

    - HotSpot默认分配策略：

        longs/doubles, ints, shorts/chars, bytes/booleans, opps(Ordinary Object Pointers)

        CompactFilds = true时，子类小变量可插入父类变量空隙内存片段中

- HotStop VM 对象起始地址为8的倍数

## 对象访问定位

- 通过栈上的reference数据操作堆上的对象
- 主流对象访问方式：
    - 句柄：Java堆中会划分出一块内存用于句柄池
        - Java栈中存放的是句柄的地址，句柄中存放到对象实例（实例池中）数据的的指针+到对象类型的指针
        - 优：reference中存储稳定的句柄地址对象被移动时只需修改句柄地址
    - 直接指针：Java栈中指向的变量是Java堆中的对象实例数据，对象实例中是到对象类型的实例数据
        - 优：访问速度快

# 垃圾收集及分配策略

## 判断对象已死

### 引用计数算法（Reference Counting）

- 算法：对象中添加计数器（计数器的值就是当前被引用的数量），计数器=0，对象已死
- 缺点：解决不了相互引用

### 可达性分析算法（Reachabiliy Analysis）

- 以“GC Roots"的对象作为起点，对象不可达时对象已死，搜索走过的路径为引用链（Reference China)

## 引用

- 强引用（Strong Reference)
    - 代码中普遍存在，类似Object obj = new Object()
- 软引用（Soft Reference)
    - 有用但非必须
- 弱引用（Weak Reference)
    - 存活至下次GC之前
- 虚引用（Phantom Reference)
    - 无法通过虚引用获取对象实例，唯一目的，被GC时收到系统通知

## 对象的真正去留

> kill对象至少标记两次

- 第一次标记：筛选条件是否有必要执行finalize()方法
    - 没必要执行：对象未覆盖finalize()方法或者已被调用过
    - 有必要执行：放置F-Queue中，由虚拟机自动建立、低优先级的finalize（）执行（只触发，并不承诺等待其执行完成），任何对象的finalize()方法只会被系统自动调用一次
- 第二次标记：F-Queue中小规模标记，无引用则回收
- 相比于使用finalize()的运行代价高昂，不确定性大，try{} catch{}更能胜任

## 回收方法区（HotSpot中的永久代）

- 回收目标：废弃常量+无用的类
    - 废弃常量回收：
        - 判断策略：常量不在被引用
        - 回收：发生垃圾回收，有必要时回收
    - 无用的类回收：
        - 判断策略：
            - 该类的实例都已被回收
            - 该类的ClassLoader已被回收
            - 该类对对应的java.lang.Class对象没有任何的引用，无法通过反射在任何地方访问该类的方法
        - 回收：判断无用后不一定必被回收，HotSpot提供：-Xnoclassgc参数控制
    - 频繁自定义ClassLoader的场景需要虚拟机具备类卸载功能

## 垃圾收集算法

> [GC算法](https://www.cnblogs.com/CNty/articles/10917531.html
> )

### Mark-Sweep

- 最基础的收集算法
- 缺点：
    - Mark和Sweep效率都不高
    - 处理后有大量不连续内存碎片

### Copying

- 内存对半分，每次使用一半，内存快用完时，将存活对象转移至另一半，随后对原半区进行内存回收
- 缺点：
    - 代价高了点，可用内存直接减半
- 改进：
    - 内存仍然划分：Eden + Survivor +Survivor
    - 使用：Eden + Survivor，另外一个用作这两个区幸存者的逃生地，回收时对使用的Eden+Survivor回收
    - 例外：Eden和Survivor中的幸存者超过Survivor的承载能力，这时需要老年代（Handle Promotion）担保(存放Survivor存放不了的对象)

### Mark-Compact

- 标记对象，存活对象向一端移动，随后清理边界意外的内存

### Generational Collection

- 内存空间依据对象存活周期不同划分，Java堆一般分新生代和老年代
- 策略：对不同区采用不同的回收算法，对象存活少---复制算法，对象存活率高---Mark-Sweep或Mark-Collpact

## HotSpot算法实现

- 枚举根节点

    - 可达性分析：
        - 背景：
            - 方法区内存大，逐个检查消耗时间。
            - 对执行时间敏感，GC停顿
                - 为保证可达性分析的正确性，GC时停止所有Java进程（Stop The World)
        - 处理：GC 停顿是必要的，但虚拟机必须有方法知道哪些地方有对象引用
            - HotSpot使用一组OopMap()，类加载时将对象偏移量上的类型数据计算出来，JIT编译过程中会在特定位置记录栈和寄存器中哪些位置是引用

- 安全点：

    > 推荐博客：[博客1](https://www.cnblogs.com/caiyao/p/9010503.html)， [博客2](https://blog.csdn.net/Daniel_2046/article/details/80786801)

    - 特定位置记录OopMap，这些记录了OopMap的位置称为安全点（Safepoint)

    - 程序执行时只能到安全点才能停

    - 安全点的选定：

        - 是否具有让程序长时间执行的特征，指令复用

    - 线程（执行JNI调用的线程）到达最近的安全点：

        - 抢占式中断：直接终止所有线程，选取不在安全点的线程启动，到达安全点后终止（几乎不再使用）

        - 主动式中断：设定标志，线程自动轮询问，标志为真自动中断挂起

- 安全区：

    - 安全区的任何时刻GC都是安全的
    - 线程快离开安全区时，判断系统是否完成了根节点枚举，若没有则这个线程等到可以安全离开（Safe Region）的信号为止

# 垃圾回收期发展历程

1. 垃圾回收器的发展路线，是随着内存越来越大的过程而演进							
    从分代算法演化到不分代算法
    Serial算法 几十兆
    Parallel算法 几个G
    CMS 几十个G  - 承上启下，开始并发回收三色标记 
2. JDK诞生 Serial追随 提高效率，诞生了PS，为了配合CMS，诞生了PN，CMS是1.4版本后期引入，CMS是里程碑式的GC，它开启了并发回收的过程，但是CMS毛病较多，因此目前任何一个JDK版本默认是CMS
    并发垃圾回收是因为无法忍受STW
3. Serial 年轻代 串行回收
4. PS 年轻代 并行回收
5. ParNew 年轻代 配合CMS的并行回收
6. SerialOld 
7. ParallelOld
8. ConcurrentMarkSweep 老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)
    CMS问题比较多，所以现在没有一个版本默认是CMS，只能手工指定
    CMS既然是MarkSweep，就一定会有碎片化的问题，碎片到达一定程度，CMS的老年代分配对象分配不下的时候，使用SerialOld 进行老年代回收
    想象一下：
    PS + PO -> 加内存 换垃圾回收器 -> PN + CMS + SerialOld（几个小时 - 几天的STW）
    几十个G的内存，单线程回收 -> G1 + FGC 几十个G -> 上T内存的服务器 ZGC
    算法：三色标记 + Incremental Update
9. G1(200ms - 10ms)
    算法：三色标记 + SATB
10. ZGC (10ms - 1ms) PK C++
       算法：ColoredPointers + LoadBarrier
11. Shenandoah
    算法：ColoredPointers + WriteBarrier
12. Eplison
13. PS 和 PN区别的延伸阅读：
    ▪[https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html#GUID-3D0BB91E-9BFF-4EBB-B523-14493A860E73](https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html)
14. 垃圾收集器跟内存大小的关系
    1. Serial 几十兆
    2. PS 上百兆 - 几个G
    3. CMS - 20G
    4. G1 - 上百G
    5. ZGC - 4T - 16T（JDK13）

1.8默认的垃圾回收：PS + ParallelOld

# 垃圾回收器

## Serial收集器

- 图示：

- 特征：

    - 一个CPU或线程完成任务
    - 收集时暂停其它所有线程

- 优：

    - 简单高效

- 应有：

    - Client

    - Serial/Serial Old图示

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191211214642798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## ParNew收集器

- Serial多线程版本
- 能与CMS收集器合作
- 应用：
    - server模式
    - ParNew + SerialOld图示
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191211214753525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## Parallel Scavenge收集器

- 新生代收集器
- 复制算法
- 并行多线程
- 自适应策略
- 目标：
    - 达到可控制吞吐量（Throughput）{吞吐量=运行用户代码时间 / (运行用户代码时间+垃圾收集器时间)}
    - -XX:+UseAdaptiveSizePolicy设定后，会开启GC自适应调整策略（GC Erggonomics）

## Serial Old收集器

- Serial收集器老年代版本
- 单线程
- 标记整理算法

## Parallel Old收集器

- 标记整理算法

- 应用：

    - 关注吞吐量及CPU资源敏感场合

    - ParallelScavenge + Parallel Old 图示：

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191211220148213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## CMS收集器（Concurrent Mark Sweep）

- 目标：获取最短回收停顿时间为目标

- 运行过程：

    - 初识标记（CMS initial mark）____Stop The World

        - 标记GC Roots能直接关联到的对象

    - 并发标记（CMS concurrent mark）

        - 进行GC Roots Tracing

    - 重新标记（CMS remark）____Stop The World

        - 修正并发标记期间用户程序运行导致标记变动的那一部分对象的标记记录

    - 并发清除（CMS concurrent sweep）

    - 运行图示：

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019121122240416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

- 缺点：

    - 对CPU敏感：
        - 默认线程数（CPU数+3)/4
        - 应对策略：
            - 增量式并发收集器（Incremental Concurrent Mark Sweep/i-CMS)，并发标记，清理时让用户线程，GCx线程交替运行（实践效果一般，不在提倡）
    - 无法处理浮动垃圾（Floating Garbage）：
        - 原因：并发清理阶段用户线程在运行，这部分垃圾在标记过程之后，只能留到下次GC处理
    - 空间碎片：
        - 由于采用标记清除算法
        - -XX+UseCMSCompactAtFullCollection，将进行Full GC时开启内存碎片的合并整理过程
        - -XX+CMSFullGCsBeforeCompaction，多少次不压缩GC后压缩一次

## G1收集器（Garbage-First）

- server
- 特点：
    - 并行+并发：
        - 使用多个CPU缩短Stop The World
        - 通过并发，让Java程序继续执行
    - 分代收集：
    - 空间整理：
        - 整体看：标记整理
        - 局部：两个Region之间，基于复制
    - **可预测停顿**：
        - 建立可预测停顿时间模型，可让使用者指定在一个长度为M毫秒的时间片段内，消耗在上GC的时间不超过N毫秒
- 内存划分：
    - 整个java堆 = n * Region, 新生代，老年代是一部分Region集合
- 有限列表：跟踪每个Region里的垃圾堆积价值（回收空间大小及回收所需时间的经验值），回收时在有限时间内回收价值最大的
- 每个Region有一个对应的RememberSet,对Reference引用的对象写操作会产生WriteBarrier暂时中断操作，检查Reference引用的对象是否处于不同Region之中，是则通过CardTable把相关引用记录到被引用对象所属的Region的RememberSet中，内存回收时，在GC 根节点枚举范围中加入RememberSet 保证不对全表扫描页脚不会有遗漏
- 大致操作流程（不计算RememberSet）：
    - 初识标记（Initial mark）
        - 标记GC Roots能直接关联到的对象，修改TAMS（Next Top at Mark Start）
    - 并发标记（Concurrent mark）
        - 从GC Roots开始对堆中对象进行可达性分析，找出存活对象，可与用户线程并发执行
    - 最终标记（Final Marking）
        - 修正并发标记期间，因用户线程继续运作而产生变化的标记记录，这段时间内对象变化记录在Remember Set中，此阶段需要停顿线程，但可并发执行
    - 筛选回收（Live Data Counting and Evacuation）
        - 对各个Region的回收价值及成本进行排序，根据用户期望的GC停顿时间来定制计划，可与用户线程并发执行，但时间较短，停顿用户线程能大幅提高回收效率

# 》》》GC过程

#### JVM内存分代模型（用于分代垃圾回收算法）


> 除Epsilon ZGC Shenandoah之外的GC都是使用逻辑分代模型
>
> G1是逻辑分代，物理不分代
>
> 除此之外不仅逻辑分代，而且物理分代

1. 永久代 元数据 - Class
2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
3. 字符串常量 1.7 - 永久代，1.8 - 堆
4. MethodArea逻辑概念 - 永久代、元数据

5. 新生代 = Eden + 2个suvivor区 

    1. YGC回收之后，大多数的对象会被回收，活着的进入s0
    2. 再次YGC，活着的对象eden + s0 -> s1
    3. 再次YGC，eden + s1 -> s0
    4. 年龄足够 -> 老年代 （15 CMS 6）
    5. s区装不下 -> 老年代

    


# 内存分配策略

> Minor GC 新生代GC：发生在新生代的垃圾收集行为
>
> Major GC/Full GC：发生在老年代GC，通常伴随至少一次MinorGC(Parallel Scavenge收集器的收集策略中有直接Major GC的策略选择)

## 对象有限在Eden分配 

- 新对象在Eden分配，若无分配空间进行一次 Minor GC

## 大对象直接进入老年代

- 大对象：需要大量连续空间的对象，例如：长字符串，数组
- 经常出现大对象可能导致提前GC
- -XX:PretenureSizeThreshold设置大对象判断边界

##  长期存活对象进入老年代

- 每个对象定义了一个对象年龄计数器
- 对象在Eden区出生，经历一次GC转移到Survivor---->age=1
    - 此后每一次GC--->age+1，到达15（默认值）晋升老年代
- -XX:MaxTenuringThreshold设置老年代晋升阈值

## 动态对象年龄判定

- Survivor空间中年龄所有对象大小的总和大于Survivor空间的一半，年龄大于等于该年龄对象直接进入老年代

## 空间分配担保

- Minor GC前虚拟机检查老年代最大可用连续空间是否大于新生代所有对象总空间，
    - 是--->安全
    - 否--->查看HandlePromontionFailure设置值是否允许担保失败
        - 允许--->继续检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小
            - 大于：尝试进行Minor GC
            - 小于或者不允许担保失败：改为进行Full GC
- 老年代一般将每一次回收晋升的老年代对象容量的平均值大小作为经验值，老年代与剩余空间比较，决定是否Full GC
    - 大部分情况将HandlePromotionFailure打开，避免Full GC太过频繁