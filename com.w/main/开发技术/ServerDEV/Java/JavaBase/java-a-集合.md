# 集合

### Collection和Collections的区别

- Collection 是一个集合接口
- Collections 是一个包装类。 它包含有各种有关集合操作的静态多态方法，不能实例化。

常用集合类的使用

### Set和List区别

List,Set都是继承自Collection接口。都是用来存储一组相同**类型**的元素的。

- List特点：有顺序后插法，元素可重复 。

- Set特点：元素无放入顺序，元素不可重复。
    - set在元素插入时是要有一定的方法来判断元素是否重复的

### ArrayList和LinkedList和Vector的区别

都实现了List 接口，使用方式也很相似,主要区别在于因为实现方式的不同,所以对不同的操作具有不同的效率。

> 大数据量情况下

ArrayList 是一个可改变大小的数组

LinkedList 是一个双链表

- LinkedList 还实现了 Queue 接口，List提供了更多的方法,包括 offer(),peek(),poll()等

Vector是强同步类，但(thread-safe,没有在多个线程之间共享同一个集合/对象)情况下，采用ArryList是更好的选择
- Vector每次增加一倍
- ArrayList每次增加一半

### SynchronizedList和Vector的区别

Vector是java.util包中的一个类。 SynchronizedList是java.util.Collections中的一个静态内部类。

多线程的场景中可以直接使用Vector类，也可以使用Collections.synchronizedList(List list)方法来返回一个线程安全的List

Add方法
- **1.Vector使用同步方法实现，synchronizedList使用同步代码块实现。 **
- **2.两者的扩充数组容量方式不一样（两者的add方法在扩容方面的差别也就是ArrayList和Vector的差别。**

SynchronizedList里面实现的方法几乎都是使用同步代码块包上List的方法。
- 其中有listIterator和listIterator(int index)并没有做同步处理
- 注意点：
    - 在使用SynchronizedList进行遍历的时候要手动加锁

Vector基本使用同步方法实现

区别分析
- 数据增加
    - Vector更合适，可指定初始化大小
- 代码块和方法
    - 同步代码块在锁定的范围上可能比同步方法要小一般来讲大小和方法成反比
    - 同步块可以更加精确的控制锁的作用域（锁的作用域就是从锁被获取到其被释放的时间），同步方法的锁的作用域就是整个方法
    - 静态代码块可以选择对哪个对象加锁，但是静态方法只能给this对象加锁

### Set如何保证元素不重复

实现方式不同主要分为两大类。HashSet和TreeSet
- 1.TreeSet 是二差树实现的,Treeset中的数据是自动排好序的，不允许放入null值 
- 2.HashSet 是哈希表实现的,HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不重复，就如数据库中唯一约束

细节
- 在HashSet中，基本的操作都是有HashMap底层实现的，因为HashSet底层是用HashMap存储数据的。当向HashSet中添加元素的时候，首先计算元素的hashcode值，然后通过扰动计算和按位与的方式计算出这个元素的存储位置，如果这个位置位空，就将元素添加进去；如果不为空，则用equals方法比较元素是否相等，相等就不添加，否则找一个空位添加。
- TreeSet的底层是TreeMap的keySet()，而TreeMap是基于红黑树实现的，红黑树是一种平衡二叉查找树，它能保证任何一个节点的左右子树的高度差不会超过较矮的那棵的一倍。

检查重复
- TreeMap是按key排序的，元素在插入TreeSet时compareTo()方法要被调用，所以TreeSet中的元素要实现Comparable接口。
- TreeSet作为一种Set，它不允许出现重复元素。TreeSet是用compareTo()来判断重复元素的。

### HashMap、HashTable、ConcurrentHashMap区别

Map&Table

线程安全：
- HashTable 中的方法是同步的，而HashMap中的方法在默认情况下是非同步的。
- 在多线程并发的环境下，可以直接使用HashTable，但是要使用HashMap的话就要自己增加同步处理了。

继承关系：
- HashTable是基于陈旧的Dictionary类继承来的。 
- HashMap继承的抽象类AbstractMap实现了Map接口

null
- HashTable中，key和value都不允许出现null值，否则会抛出NullPointerException异常。
- HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。

初始值
- HashTable中的hash数组初始大小是11，增加的方式是 old*2+1。
- HashMap中hash数组的默认大小是16，而且一定是2的指数。

哈希值的使用不同 ： 
- HashTable直接使用对象的hashCode。
- HashMap重新计算hash值。

遍历
- 遍历方式的内部实现上不同 ： Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。 
- HashMap 实现 Iterator，支持fast-fail，Hashtable的 Iterator 遍历支持fast-fail，用 Enumeration 不支持 fast-fail

HashMap&CurrentHashMap

遍历方式的内部实现上不同 ：桶数组 
- Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。
- HashMap 实现 Iterator，支持fast-fail，Hashtable的 Iterator 遍历支持fast-fail，用 Enumeration 不支持 fast-fail

锁
- ConcurrentHashMap在每一个分段上都用锁进行了保护。
- HashMap没有锁机制。

Java 8中Map相关的红黑树的引用背景、原理等

HashMap的容量、扩容、hash等原理

### Java 8中stream相关用法

Stream有以下特性及优点：
- 无存储。Stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
- 为函数式编程而生。对Stream的任何修改都不会修改背后的数据源，比如对Stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新Stream。
- 惰式执行。Stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
- 可消费性。Stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

最终操作会消耗流，产生一个最终结果。

- 在最终操作之后，不能再次使用流，也不能在使用任何中间操作，否则将抛出异常

操作类型
- Stream的创建有两种方式，分别是通过集合类的stream方法、通过Stream的of方法。
- Stream的中间操作可以用来处理Stream，中间操作的输入和输出都是Stream，中间操作可以是过滤、转换、排序等。
- Stream的最终操作可以将Stream转成其他形式，如计算出流中元素的个数、将流装换成集合、以及元素的遍历等。

Apache集合处理工具类的使用

不同版本的JDK中HashMap的实现的区别以及原因

### Arrays.asList获得的List使用时需要注意什么

- asList 得到的只是一个 Arrays 的内部类，一个原来数组的视图 List，因此如果对它进行增删操作会报错
- 用 ArrayList 的构造器可以将其转变成真正的 ArrayLi

Collection如何迭代

#### Enumeration和Iterator区别

函数接口不同
- ​    Enumeration只有2个函数接口。通过Enumeration，我们只能读取集合的数据，而不能对数据进行修改。

- Iterator只有3个函数接口。Iterator除了能读取集合的数据之外，也能数据进行删除操作。

Iterator支持fail-fast机制，而Enumeration不支持
- Enumeration 是JDK 1.0添加的接口。使用到它的函数包括Vector、Hashtable等类，这些类都是JDK 1.0中加入的，Enumeration存在的目的就是为它们提供遍历接口。Enumeration本身并没有支持同步，而在Vector、Hashtable实现Enumeration时，添加了同步。
- 而Iterator 是JDK 1.2才添加的接口，它也是为了HashMap、ArrayList等集合提供遍历接口。Iterator是支持fail-fast机制的：当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

如何在遍历的同时删除ArrayList中的元素

### fail-fast 和 fail-safe

fail-fast

- 立即报告任何可能表明故障的情况的系统

fail-safe

- 为了避免触发fail-fast机制，导致异常，我们可以使用Java中提供的一些采用了fail-safe机制的集合类。

    这样的集合容器在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

fail-safe集合的所有对集合的修改都是先拷贝一份副本，然后在副本集合上进行的，并不是直接对原集合进行修改。并且这些修改方法，如add/remove都是通过加锁来控制并发的。

所以，CopyOnWriteArrayList中的迭代器在迭代的过程中不需要做fail-fast的并发检测。（因为fail-fast的主要目的就是识别并发，然后通过异常的方式通知用户）

CopyOnWriteArrayList

ConcurrentSkipListMap