# JMM分区和内存访问定位

# 内存自动管理机制

## 运行时数据区

![image-20191211002910416](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191211002910416.png)

### 程序计数器

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

## 空间使用

新生代 + 老年代 + 永久代（1.7）Perm Generation/ 元数据区(1.8) Metaspace

1. 永久代 元数据 - Class
2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
3. 字符串常量 1.7 - 永久代，1.8 - 堆
4. MethodArea逻辑概念 - 永久代、元数据

新生代 = Eden + 2个suvivor区 

1. YGC回收之后，大多数的对象会被回收，活着的进入s0
2. 再次YGC，活着的对象eden + s0 -> s1
3. 再次YGC，eden + s1 -> s0
4. 年龄足够 -> 老年代 （15 CMS 6）
5. s区装不下 -> 老年代

老年代

1. 顽固分子
2. 老年代满了FGC Full GC

GC Tuning (Generation)

1. 尽量减少FGC
2. MinorGC = YGC
3. MajorGC = FGC

晋升老年代动态年龄

https://www.jianshu.com/p/989d3b06a49d

# 对象

## 对象的创建

### 遇到new指令

1. （找符号引用）检查其参数能否在常量池中定位到一个类的**符号引用**

2. （检测是否已加载）检查其引用代表的类是否已被加载，解析，初始化过，若没有执行相应的类加载过程

    1. （分配内存）为对象分配内存

        - 存在的问题：

            “指针碰撞”：内存规整，指针为标记，一边为空闲区，一边为使用区

            ​		代表：Serial, ParNew，带Compact过程的收集器

            “空闲列表”：内存不规整，空闲列表记录空闲区，使用时依据空闲表分配内存

            ​		代表：CMS，基于Mark-Sweep算法的收集器

        - 解决线程分配安全：

        ​		分配内存动作同步处理

        1. CAS+失败重试保证更新操作的原子性

        2. 内存分配动作按照线程划分到不同空间中进行

            （线程在Java堆中分配小块内存，本地分配缓冲（Thread Local Allocation Buffer, TLAB））

            -XX:+/-UseTLAB设定是否使用分配内存

### 初始化

- （空间清空）分配到的内存空间初始化为零值（除对象头以外），使用TLAB时，此步骤可提前至TLAB分配时进行
- （对象头）设定对象头（虚拟机视角，对象创建完成）
- （对象初始化）new 执行--->\<init\>执行（程序员视角，对象创建完成）

## 对象内存模型

> 对象头（Header) + 实例数据（Instance Data) + 对齐填充（Padding）

### 对象头

- 运行时数据（32/64w位系统(未开启指针压缩)中分别为32bit,64bit），Make Word

    设计为非固定数据结构，可依据对象状态复用自己存储空间

- 类型指针（指向类元数据）（并非所有虚拟机实现均有此）

###  实例数据

程序中定义的各种字段内容，父类继承，子类中定义

存储顺序由： 虚拟机分配策略（FeildsAllocationStyle) + 字段在Java源码中定义顺序

- HotSpot默认分配策略：

    longs/doubles, ints, shorts/chars, bytes/booleans, opps(Ordinary Object Pointers)

    CompactFilds = true时，子类小变量可插入父类变量空隙内存片段中

> HotStop VM 对象起始地址为8的倍数

## 对象访问定位

通过栈上的reference数据操作堆上的对象

主流对象访问方式：

句柄：Java堆中会划分出一块内存用于句柄池
- Java栈中存放的是句柄的地址，句柄中存放到对象实例（实例池中）数据的的指针+到对象类型的指针
- 优：reference中存储稳定的句柄地址对象被移动时只需修改句柄地址

直接指针：Java栈中指向的变量是Java堆中的对象实例数据，对象实例中是到对象类型的实例数据

- 优：访问速度快