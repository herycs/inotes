# JVM&Java参数

## 打印加载类

-XX:TraceClassLoading

java -verbose

## 设置直接内存

 -XX:MaxDirectMemorySize

## 设置开启Table分配机制

> **为了保证对象的内存分配过程中的线程安全性，HotSpot虚拟机提供了一种叫做TLAB(Thread Local Allocation Buffer)的技术。**
>
> 在线程初始化时，虚拟机会为每个线程分配一块TLAB空间，只给当前线程使用，当需要分配内存时，就在自己的空间上分配，这样就不存在竞争的情况，可以大大提升分配效率。
>
> **所以，“堆是线程共享的内存区域”这句话并不完全正确，因为TLAB是堆内存的一部分，他在读取上确实是线程共享的，但是在内存分分配上，是线程独享的。**
>
> TLAB的空间其实并不大，所以大对象还是可能需要在堆内存中直接分配。那么，对象的内存分配步骤就是先尝试TLAB分配，空间不足之后，再判断是否应该直接进入老年代，然后再确定是再eden分配还是在老年代分配。

堆中划分的称为Table的线程私有的用于对象分配的空间，但在读取和GC是仍然是共享的

-XX:+/-UseTLAB

设置TLAB空间所占用Eden空间的百分比大小，默认是1%

-XX:TLABWasteTargetPercent

禁用Table的大小自动调整

-XX:-ResizeTLAB

设置table大小

-XX：TLABSize

TLAB的refill_waste也是可以调整的，默认值为64，即表示使用约为1/64空间大小作为refill_waste

-XX：TLABRefillWasteFraction

跟踪Table使用情况

-XX+PringTLAB

## CMS搜集器

开启

-XX:+UseConcMarkSweepGC

一般CMS的GC耗时80%都在remark阶段，remark前触发youngGC

-XX:+CMSScavengeBeforeRemark

GC后多少次FullGC再做内存压缩

-XX:CMSFullGCsBeforeCompaction=n

CMS触发时机

JVM仅在第一次会使用此,后续则自动调整

-XX:+UseCMSInitiatingOccupancyOnly

CMS在对内存占用率达到70%的时候开始GC

-XX:CMSInitiatingOccupancyFraction=70