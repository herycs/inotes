# jvm - 调优2

### GC的基础知识

#### 1.什么是垃圾

> C语言申请内存：malloc free
>
> C++： new delete
>
> c/C++ 手动回收内存
>
> Java: new ？
>
> 自动内存回收，编程上简单，系统不容易出错，手动释放内存，容易出两种类型的问题：
>
> 1. 忘记回收
> 2. 多次回收

没有任何引用指向的一个对象或者多个对象（循环引用）

#### 2.如何定位垃圾

1. 引用计数（ReferenceCount）
2. 根可达算法(RootSearching)

#### 3.常见的垃圾回收算法

1. 标记清除(mark sweep) - 位置不连续 产生碎片 效率偏低（两遍扫描）
2. 拷贝算法 (copying) - 没有碎片，浪费空间
3. 标记压缩(mark compact) - 没有碎片，效率偏低（两遍扫描，指针需要调整）

#### 4.JVM内存分代模型（用于分代垃圾回收算法）

1. 部分垃圾回收器使用的模型

    > 除Epsilon ZGC Shenandoah之外的GC都是使用逻辑分代模型
    >
    > G1是逻辑分代，物理不分代
    >
    > 除此之外不仅逻辑分代，而且物理分代

2. 新生代 + 老年代 + 永久代（1.7）Perm Generation/ 元数据区(1.8) Metaspace

    1. 永久代 元数据 - Class
    2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
    3. 字符串常量 1.7 - 永久代，1.8 - 堆
    4. MethodArea逻辑概念 - 永久代、元数据

3. 新生代 = Eden + 2个suvivor区 

    1. YGC回收之后，大多数的对象会被回收，活着的进入s0
    2. 再次YGC，活着的对象eden + s0 -> s1
    3. 再次YGC，eden + s1 -> s0
    4. 年龄足够 -> 老年代 （15 CMS 6）
    5. s区装不下 -> 老年代

4. 老年代

    1. 顽固分子
    2. 老年代满了FGC Full GC

5. GC Tuning (Generation)

    1. 尽量减少FGC
    2. MinorGC = YGC
    3. MajorGC = FGC

6. 对象分配过程图
    ![](D:/___Download_Data___/BaiduNetdiskDownload/对象分配过程详解.png)

7. 动态年龄：（不重要）
    https://www.jianshu.com/p/989d3b06a49d

8. 分配担保：（不重要）
    YGC期间 survivor区空间不够了 空间担保直接进入老年代
    参考：https://cloud.tencent.com/developer/article/1082730

#### 5.常见的垃圾回收器

![常用垃圾回收器](D:/___Download_Data___/BaiduNetdiskDownload/常用垃圾回收器.png)

1. 垃圾回收器的发展路线，是随着内存越来越大的过程而演进							
    从分代算法演化到不分代算法
    Serial算法 几十兆
    Parallel算法 几个G
    CMS 几十个G  - 承上启下，开始并发回收 -
    .- 三色标记 - 
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

### JVM调优第一步，了解JVM常用命令行参数

* JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* HotSpot参数分类

    > 标准： - 开头，所有的HotSpot都支持
    >
    > 非标准：-X 开头，特定版本HotSpot支持特定命令
    >
    > 不稳定：-XX 开头，下个版本可能取消

    java -version

    java -X

    

    java -XX:+PrintFlagsWithComments //只有debug版本能用

    

    试验用程序：

    ```java
    import java.util.List;
    import java.util.LinkedList;
    
    public class HelloGC {
      public static void main(String[] args) {
        System.out.println("HelloGC!");
        List list = new LinkedList();
        for(;;) {
          byte[] b = new byte[1024*1024];
          list.add(b);
      }
      }
    }
    ```

    

    1. 区分概念：内存泄漏memory leak，内存溢出out of memory
    2. java -XX:+PrintCommandLineFlags HelloGC
    3. java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -XX:+PrintGC  HelloGC
        PrintGCDetails PrintGCTimeStamps PrintGCCauses
    4. java -XX:+UseConcMarkSweepGC -XX:+PrintCommandLineFlags HelloGC
    5. java -XX:+PrintFlagsInitial 默认参数值
    6. java -XX:+PrintFlagsFinal 最终参数值
    7. java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数
    8. java -XX:+PrintFlagsFinal -version |grep GC
    9. java -XX:+PrintFlagsFinal -version | wc -l 
        共728个参数

### PS GC日志详解

每种垃圾回收器的日志格式是不同的！

PS日志格式

![GC日志详解](D:/___Download_Data___/BaiduNetdiskDownload/GC日志详解.png)

heap dump部分：

```java
eden space 5632K, 94% used [0x00000000ff980000,0x00000000ffeb3e28,0x00000000fff00000)
                            后面的内存地址指的是，起始地址，使用空间结束地址，整体空间结束地址
```

![GCHeapDump](D:/___Download_Data___/BaiduNetdiskDownload/GCHeapDump.png)

total = eden + 1个survivor