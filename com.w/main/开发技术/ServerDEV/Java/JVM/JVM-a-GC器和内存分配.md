# 垃圾回收器

# 1.垃圾回收器

G1是逻辑分代，物理不分代，除此之外不仅逻辑分代，而且物理分代

垃圾回收器的发展路线，是随着内存越来越大的过程而演进，从分代算法演化到不分代算法

- Serial算法 几十兆
- Parallel算法 几个G
- CMS 几十个G  - 承上启下，开始并发回收 -

JDK诞生 Serial追随 提高效率，诞生了PS，为了配合CMS，诞生了PN，CMS是1.4版本后期引入，CMS是里程碑式的GC，它开启了并发回收的过程，但是CMS毛病较多，因此目前任何一个JDK版本默认是CMS，并发垃圾回收是因为无法忍受STW

1. Serial 年轻代 串行回收

2. PS 年轻代 并行回收

3. ParNew 年轻代 配合CMS的并行回收

4. SerialOld 

5. ParallelOld

6. ConcurrentMarkSweep 老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)

    CMS问题比较多，所以现在没有一个版本默认是CMS，只能手工指定

    CMS既然是MarkSweep，就一定会有碎片化的问题，碎片到达一定程度，CMS的老年代分配对象分配不下的时候，使用SerialOld 进行老年代回收

    想象一下：
    PS + PO -> 加内存 换垃圾回收器 -> PN + CMS + SerialOld（几个小时 - 几天的STW）

    几十个G的内存，单线程回收 -> G1 + FGC 几十个G -> 上T内存的服务器 ZGC

    算法：三色标记 + Incremental Update

1. G1(200ms - 10ms)
    算法：三色标记 + SATB
2. ZGC (10ms - 1ms) PK C++
       算法：ColoredPointers + LoadBarrier
3. Shenandoah
    算法：ColoredPointers + WriteBarrier
4. Eplison
5. PS 和 PN区别的延伸阅读：
    ▪[https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html#GUID-3D0BB91E-9BFF-4EBB-B523-14493A860E73](https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html)
6. 垃圾收集器跟内存大小的关系
    1. Serial 几十兆
    2. PS 上百兆 - 几个G
    3. CMS - 20G
    4. G1 - 上百G
    5. ZGC - 4T - 16T（JDK13）

### 常见垃圾回收器组合参数设定：(1.8)

* -XX:+UseSerialGC = Serial New (DefNew) + Serial Old
    * 小型程序。默认情况下不会是这种选项，HotSpot会根据计算及配置和JDK版本自动选择收集器
* -XX:+UseParNewGC = ParNew + SerialOld
    * 这个组合已经很少用（在某些版本中已经废弃）
    * https://stackoverflow.com/questions/34962257/why-remove-support-for-parnewserialold-anddefnewcms-in-the-future
* -XX:+UseConc<font color=red>(urrent)</font>MarkSweepGC = ParNew + CMS + Serial Old
* -XX:+UseParallelGC = Parallel Scavenge + Parallel Old (1.8默认) 【PS + SerialOld】
* -XX:+UseParallelOldGC = Parallel Scavenge + Parallel Old
* -XX:+UseG1GC = G1
* Linux中没找到默认GC的查看方法，而windows中会打印UseParallelGC 
    * java +XX:+PrintCommandLineFlags -version
    * 通过GC的日志来分辨

* Linux下1.8版本默认的垃圾回收器到底是什么？

    * 1.8.0_181 默认（看不出来）Copy MarkCompact
    * 1.8.0_222 默认 PS + PO

## 1.1 Serial收集器

图示：

特征：

- 一个CPU或线程完成任务
- 收集时暂停其它所有线程

优：简单高效

应用：

- Client

- Serial/Serial Old图示

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191211214642798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## 1.2 ParNew收集器

Serial多线程版本

能与CMS收集器合作

应用：

- server模式
- ParNew + SerialOld图示
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191211214753525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## 1.3Parallel Scavenge收集器

- 新生代收集器
- 复制算法
- 并行多线程
- 自适应策略

目标：

- 达到可控制吞吐量（Throughput）{吞吐量=运行用户代码时间 / (运行用户代码时间+垃圾收集器时间)}
- -XX:+UseAdaptiveSizePolicy设定后，会开启GC自适应调整策略（GC Erggonomics）

## 1.4Serial Old收集器

Serial收集器老年代版本

单线程

标记整理算法

## 1.5Parallel Old收集器

标记整理算法

应用：

- 关注吞吐量及CPU资源敏感场合

- ParallelScavenge + Parallel Old 图示：

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191211220148213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## 1.6 CMS收集器（Concurrent Mark Sweep）

目标：获取最短回收停顿时间为目标

运行过程：

- 初识标记（CMS initial mark）____Stop The World

    - 标记GC Roots能直接关联到的对象

- 并发标记（CMS concurrent mark）

    - 进行GC Roots Tracing

- 重新标记（CMS remark）____Stop The World

    - 修正并发标记期间用户程序运行导致标记变动的那一部分对象的标记记录

- 并发清除（CMS concurrent sweep）


运行图示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019121122240416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

缺点：

1. 对CPU敏感：
    - 默认线程数（CPU数+3)/4
    - 应对策略：增量式并发收集器（Incremental Concurrent Mark Sweep/i-CMS)，并发标记，清理时让用户线程，GCx线程交替运行（实践效果一般，不再提倡）
2. 无法处理浮动垃圾（Floating Garbage）：并发清理阶段用户线程在运行，这部分垃圾在标记过程之后，只能留到下次GC处理
3. 空间碎片：
    - 由于采用标记清除算法
    - -XX+UseCMSCompactAtFullCollection，将进行Full GC时开启内存碎片的合并整理过程
    - -XX+CMSFullGCsBeforeCompaction，多少次不压缩GC后压缩一次

## 1.7 G1收集器（Garbage-First）

server

特点：

- 并行+并发：
    - 使用多个CPU缩短Stop The World
    - 通过并发，让Java程序继续执行
- 分代收集：
- 空间整理：
    - 整体看：标记整理
    - 局部：两个Region之间，基于复制
- **可预测停顿**：
    - 建立可预测停顿时间模型，可让使用者指定在一个长度为M毫秒的时间片段内，消耗在上GC的时间不超过N毫秒

内存划分：

- 整个java堆 = n * Region, 新生代，老年代是一部分Region集合

- 有限列表：跟踪每个Region里的垃圾堆积价值（回收空间大小及回收所需时间的经验值），回收时在有限时间内回收价值最大的
- 每个Region有一个对应的RememberSet,对Reference引用的对象写操作会产生WriteBarrier暂时中断操作，检查Reference引用的对象是否处于不同Region之中，是则通过CardTable把相关引用记录到被引用对象所属的Region的RememberSet中，内存回收时，在GC 根节点枚举范围中加入RememberSet 保证不对全表扫描页脚不会有遗漏

大致操作流程（不计算RememberSet）：

1. 初识标记（Initial mark）：标记GC Roots能直接关联到的对象，修改TAMS（Next Top at Mark Start）

2. 并发标记（Concurrent mark）：从GC Roots开始对堆中对象进行可达性分析，找出存活对象，可与用户线程并发执行

3. 最终标记（Final Marking）：修正并发标记期间，因用户线程继续运作而产生变化的标记记录，这段时间内对象变化记录在Remember Set中，此阶段需要停顿线程，但可并发执行

4. 筛选回收（Live Data Counting and Evacuation）：对各个Region的回收价值及成本进行排序，根据用户期望的GC停顿时间来定制计划，可与用户线程并发执行，但时间较短，停顿用户线程能大幅提高回收效率


# 2.内存分配策略

> Minor GC 新生代GC：发生在新生代的垃圾收集行为
>
> Major GC/Full GC：发生在老年代GC，通常伴随至少一次MinorGC(Parallel Scavenge收集器的收集策略中有直接Major GC的策略选择)

## 2.1 对象有限在Eden分配 

新对象在Eden分配，若无分配空间进行一次 Minor GC

## 2.2 大对象直接进入老年代

大对象：需要大量连续空间的对象，例如：长字符串，数组

经常出现大对象可能导致提前GC

-XX:PretenureSizeThreshold设置大对象判断边界

## 2.3 长期存活对象进入老年代

每个对象定义了一个对象年龄计数器

对象在Eden区出生，经历一次GC转移到Survivor---->age=1

- 此后每一次GC--->age+1，到达15（默认值）晋升老年代

- -XX:MaxTenuringThreshold设置老年代晋升阈值

## 2.4 动态对象年龄判定

Survivor空间中年龄所有对象大小的总和大于Survivor空间的一半，年龄大于等于该年龄对象直接进入老年代

## 2.5 空间分配担保

Minor GC前虚拟机检查<u>老年代</u>最大可用连续空间是否大于<u>新生代</u>所有对象总空间
- 是--->安全
- 否--->查看HandlePromontionFailure设置值是否允许担保失败
    - 允许--->继续检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小
        - 大于：尝试进行Minor GC
        - 小于或者不允许担保失败：改为进行Full GC

老年代一般将每一次回收晋升的老年代对象容量的平均值大小作为经验值，老年代与剩余空间比较，决定是否Full GC

- 大部分情况将HandlePromotionFailure打开，避免Full GC太过频繁