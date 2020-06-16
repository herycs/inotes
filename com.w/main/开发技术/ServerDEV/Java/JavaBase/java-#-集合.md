# 集合框架

## 基础结构

```shell
|---Iterator
	|---Collection：所有集合类的根接口(不允许用户继承，但提供了两个可继承的接口)
		|---List
			|---ArrayList
			|---LinkedList
		|---Set
			|---AbatractSet
			|---HashSet
			|---TreeSet(二叉排序树)
		|---Queue
	|---Map
		|---AbstractMap
		|---HashMap
			|---LinkedHashMap
			|---CurrentHashMap
		|---TreeMap(二叉排序树)
```

Iterator提供以下几个操作：

- hasNext()
- next()
- remove()

## Collection

### List

#### ArrayList

> 数组实现

- 非同步

#### LinkedList

> 链表实现

### Set

### Queue

## Map

### HashMap

- 非同步
- 键：至多有一个NULL
- 值：不限

### HashTable

- 同步
- 线程不安全
- 键：不为NULL
- 值：不为NULL
- 锁：锁整个Table

### ConcurrentHashMap

- 同步
- 线程安全
- 锁：（jdk1.7）分段锁（Segment），锁住一部分；jdk1.8-node节点锁，粒度小

### LinkedHashMap

> HashMap 链表版

- 非同步
- 键：至多有一个NULL
- 值：不限

### TreeMap

> 实现：SortMap

- 非同步
- 键：不为NULL

内部数据有序，默认升序，可指定排序的比较器

## 遍历

### 集合类常见

1. Iterator：迭代输出，是使用最多的输出方式
2. ListIterator：是Iterator的子接口，专门用于输出List中的内容。
3. foreach输出：JDK1.5之后提供的新功能，可以输出数组或集合。
4. for循环

### Map遍历

1. keySet()：取出所有的**key**
2. entrySet()：取出所有的**K-V**对

## 区别

### Vector和ArrayList

线程安全性

- vector：线程同步的，所以它也是线程安全的
- 而arraylist是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用arraylist效率比较高

扩容

- vector增长率为目前数组长度的100%
- arraylist增长率为目前数组长度的50%。

数据访问

- 如果查找一个指定位置的数据，vector和arraylist使用的时间是相同的
- 如果频繁的访问数据，这个时候使用vector和arraylist都可以。

性能

- ArrayList 和Vector是采用数组方式存储数据
- Vector由于使用了synchronized方法（线程安全）所以性能上比ArrayList要差

### arraylist和linkedlist

实现

- ArrayList是实现了基于动态数组的数据结构
- LinkedList基于链表的数据结构。

访问

- 对于随机访问get和set，ArrayList优于LinkedList，因为LinkedList要移动指针。

修改

- 对于新增和删除操作add和remove，LinkedList比较占优势，因为ArrayList要移动数据。

### HashMap与TreeMap

-  HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。
- 在Map 中插入、删除和定位元素，HashMap是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。使用HashMap要求添加的键类明确定义了hashCode()和 equals()的实现。
    两个map中的元素一样，但顺序不一样，导致hashCode()不一样。

### HashTable与HashMap

同步性

- Hashtable是线程安全的，也就是说是同步的
- HashMap是线程序不安全的，不是同步的。

键值

- HashMap允许存在一个为null的key，多个为null的value 。
- hashtable的key和value都不允许为null。