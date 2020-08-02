# 对象大小

 虚拟机：Java HotSpot(TM) 64-Bit Server VM (25.221-b11, mixed mode)

 对象的内存以字节为单位，必须是8的倍数，它的构成由3部分组成：

- 对象头
- 实例数据
- 对齐内存

对象头主要包括对象的运行行元数据，比较哈希码、GC分代年龄、锁状态标志还有类型指针，类型指针指向类元数据

实例数据包括自身数据和所有父级数据，所有父级占内存大小都是8的倍数，没有就需要补齐。

类型指针一般为4字节，在关闭压缩普通对象指针时（-XX:+UseCompressedOops）为8字节，UseCompressedOops默认是开启的，只有虚拟机内存达到32G以上，4个字节已经无法满足寻址需求时，才需要关闭该参数。

 普通对象头除类型指针外的大小为8字节，在开启压缩总大小为12字节，不开启压缩总大小为16字节；

数组对象头在开启压缩时是16字节，不开启压缩为24字节。

 各种类型大小如下：

| 对象类型   | 字节                         |      |
| ---------- | ---------------------------- | ---- |
| boolean    | 1                            |      |
| byte       | 1                            |      |
| short      | 2                            |      |
| char       | 2                            |      |
| int        | 4                            |      |
| float      | 4                            |      |
| long       | 8                            |      |
| double     | 8                            |      |
| 引用类型   | 开启指针压缩为4，不开启为8   |      |
| 普通对象头 | 开启指针压缩为12，不开启为8  |      |
| 数据对象头 | 开启指针压缩为16，不开启为24 |      |

## 实测

```java
public class SizeOfEmptyClass {// 16 Byte
}
```

```java
public class SizeOfInt {
    int id = 4;
}
```

```java
public class SizeOfString {
    String name;
}
```

```java
public class SizeOfObject {
    Object o;
}
```

```java
public class SizeOfData {
    int id;// 4 Byte
    String name;// 4 Byte
    Object o;// 4 Byte
}
```

Java Visual VM 监控结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729153603584.png)

默认为开启压缩，故对象头默认数据大小为：8 + 4 = 12

SizeOfEmptyClass ： 对齐-》+ 4 = 16

SizeOfInt：对齐-》+ 0 = 16

SizeOfString | SizeOfObject：对齐-》 + 0 = 16

SizeOfData：对齐-》+ 0 = 24