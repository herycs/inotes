# GC判定及策略

## 判断对象已死

### 引用计数算法（Reference Counting）

算法：对象中添加计数器（计数器的值就是当前被引用的数量），计数器=0，对象已死

缺点：解决不了相互引用

### 可达性分析算法（Reachabiliy Analysis）

以“GC Roots"的对象作为起点，对象不可达时对象已死，搜索走过的路径为引用链（Reference China)

## 引用

强引用（Strong Reference)：代码中普遍存在，类似Object obj = new Object()

软引用（Soft Reference)：有用但非必须

弱引用（Weak Reference)：存活至下次GC之前

虚引用（Phantom Reference)：无法通过虚引用获取对象实例，唯一目的，被GC时收到系统通知

## 对象的真正去留

> kill对象至少标记两次

1. 第一次标记：筛选条件是否有必要执行finalize()方法
    - 没必要执行：对象未覆盖finalize()方法或者已被调用过

    - 有必要执行：放置F-Queue中，由虚拟机自动建立、低优先级的finalize（）执行（只触发，并不承诺等待其执行完成），任何对象的finalize()方法只会被系统自动调用一次

2. 第二次标记：F-Queue中小规模标记，无引用则回收

相比于使用finalize()的运行代价高昂，不确定性大，try{} catch{}更能胜任

## 回收方法区（HotSpot中的永久代）

回收目标：废弃常量+无用的类

废弃常量回收：

- 判断策略：常量不在被引用
- 回收：发生垃圾回收，有必要时回收

无用的类回收：

- 判断策略：
    - 该类的实例都已被回收
    - 该类的ClassLoader已被回收
    - 该类对对应的java.lang.Class对象没有任何的引用，无法通过反射在任何地方访问该类的方法
- 回收：判断无用后不一定必被回收，HotSpot提供：-Xnoclassgc参数控制

频繁自定义ClassLoader的场景需要虚拟机具备类卸载功能

## 垃圾收集算法

> [GC算法](https://www.cnblogs.com/CNty/articles/10917531.html
> )

###  Mark-Sweep

最基础的收集算法 - 位置不连续 产生碎片 效率偏低（两遍扫描）

缺点：

- Mark和Sweep效率都不高
- 处理后有大量不连续内存碎片

### Copying

\- 没有碎片，浪费空间

内存对半分，每次使用一半，内存快用完时，将存活对象转移至另一半，随后对原半区进行内存回收

缺点：

- 代价高了点，可用内存直接减半

改进：

- 内存仍然划分：Eden + Survivor +Survivor
- 使用：Eden + Survivor，另外一个用作这两个区幸存者的逃生地，回收时对使用的Eden+Survivor回收
- 例外：Eden和Survivor中的幸存者超过Survivor的承载能力，这时需要老年代（Handle Promotion）担保(存放Survivor存放不了的对象)

### Mark-Compact

\- 没有碎片，效率偏低（两遍扫描，指针需要调整）

标记对象，存活对象向一端移动，随后清理边界意外的内存

### Generational Collection

内存空间依据对象存活周期不同划分，Java堆一般分新生代和老年代

策略：对不同区采用不同的回收算法，对象存活少---复制算法，对象存活率高---Mark-Sweep或Mark-Collpact

## HotSpot算法实现

枚举根节点

可达性分析：

背景：

- 方法区内存大，逐个检查消耗时间。
- 对执行时间敏感，GC停顿：为保证可达性分析的正确性，GC时停止所有Java进程（Stop The World)

处理：GC 停顿是必要的，但虚拟机必须有方法知道哪些地方有对象引用

- HotSpot使用一组OopMap()，类加载时将对象偏移量上的类型数据计算出来，JIT编译过程中会在特定位置记录栈和寄存器中哪些位置是引用

安全点：

> 推荐博客：[博客1](https://www.cnblogs.com/caiyao/p/9010503.html)， [博客2](https://blog.csdn.net/Daniel_2046/article/details/80786801)

- 特定位置记录OopMap，这些记录了OopMap的位置称为安全点（Safepoint)

- 程序执行时只能到安全点才能停

- 安全点的选定：是否具有让程序长时间执行的特征，指令复用
- 线程（执行JNI调用的线程）到达最近的安全点：

    - 抢占式中断：直接终止所有线程，选取不在安全点的线程启动，到达安全点后终止（几乎不再使用）

    - 主动式中断：设定标志，线程自动轮询问，标志为真自动中断挂起

安全区：

- 安全区的任何时刻GC都是安全的
- 线程快离开安全区时，判断系统是否完成了根节点枚举，若没有则这个线程等到可以安全离开（Safe Region）的信号为止