# Map

> HashMap & LinkedHashMap & TreeMap & CouncurrentHashMap

# HashMap

## 概览

### jdk1.7

jdk1.7中的实现：数组加链表

#### put流程

1. 通过对key进行hash()得到其唯一散列值

2. 将其key映射到数组中，对散列值取余（hash & (length-1)）length为数组长度，自此存储完成

    - 和length-1做&运算，length一定是一个2^N，N是整数

        举例：length = 16 {0001 0000}，这是length-1 = {0000 1111}，这和hash做&运算相当于取了低四位，正好对应0-15的数组索引

        当然为了让这个运算结果更加的分散均匀，让高位的通过位移也参与到运算过程中，增加了数据的散列程度

1.7的hash方法

> 考：[ITeye](https://www.iteye.com/topic/709945)

```java
//这种写法使得有一位一旦有变化就会引发整个值的改动
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

算法中是采用(h>>>7)而不是(h>>>8)的算法，应该是考虑1、2、3三位出现重复位^运算的情况。

使得最**低位上原hashCode的8位都参与了^运算**，所以在table.length为默认值16的情况下面，hashCode任意位的变化基本都能反应到最终hash table 定位算法中

#### 冲突

- 发生原因：冲突是由于hash()计算后得到的要存储在的数组位置已有了元素，同时达到了超过扩容阈值

- 处理办法：

    - 流程：这时就会形成链表，具体操作就是使用头插法（认为新插入的会被先用到），类似挂砝码，旧的挂到新的下面，新的挂到秤杆（元素组）上

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200609155017565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

    - 采用原因：若使用尾插，则当链表的长度比较长的时候会导致检索时间过长，从而影响效率

#### 扩容

- 发生原因：Map中的节点数量达到了扩容阈值，[当前容量*负载因子]

- 处理办法：

    - 流程：先扩容（申请2倍容量的数据），然后数据迁移

        - 拷贝：拷贝的方式是基于已有hash值再次定位，定位方式是：hash & (length-1)），此处的length是新的数组长度

            例子：length = 32 {0010 0000}，length-1 = 31 {0001 1111}相对于扩容前可以看到用于进行数据切割的位数多了一位

            这时在进行hash & (length-1)只有两个结果：

            1. 运算结果高位增加1（数值增加16）

                例如：原来在index = 2，这时在第二个数组的索引2处也就是（index = 16 + 2 = 18）

            2. 运算结果高位增加一个0（也就是不变）

                例如：原来在index = 2，此时任然在index = 2的位置

- 结果：相对于原链表，扩容后节点顺序会改变，假定扩容前后a,b,c都在同一个位置，则：原链表：a->b->c，扩容后：c->b->a

    - 原因：扩容是从数组所指向的节点开始依次遍历，所以最开始被移动到新数组的是最靠近数组的节点，而插入方法是头插法

#### 死锁

- 在多线程环境下可能执行put()会导致死锁

    ```java
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                //这两步是成环的关键
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
    ```

- 发生流程，操作HashMap此时达到阈值，需要扩容（节点不动，修改指针）：

    ​	原情况：

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200609164505144.png)

    1. 线程A操作：将a,b记录，e->Node2, next->Node1，时间片结束

    2. 线程B操作：a,b扩容后，此时的顺序是table[i] = Node1，Node1.next = Node2，操作结束或者时间片到了

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200609164542939.png)

    3. 线程A操作：e.next = table[i] {此处就是 Node2.next = table[i] = Node1}，好到此为止，关系Node1<=>Node2建立

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200609165447592.png)

### jdk1.8

实现：数组 + 链表 | 红黑树（>=阈值时修改结构，默认8）

#### put流程

1. 对key进行hash()若为null,得到的值将会是0，计算索引索引任然是hash & (length -1)

    > 源码中的hash()方法，除了调用Object的hash方法，还与hash高16位进行了与运算

    ```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    ```

2. 放置节点

#### 冲突

- 发生原因：同上

- 处理策略：尾插法，若链表节点个数[>=8]，对当前table[i]数据结构进行修改，结构改为红黑树（选红黑树的原因此处暂不讨论）

    - 此处有一点：TreeNode比普通Node大一点（所以在不非常影响性能情况下，尽可能晚点使用树结构）

        ```java
        //普通节点
        static class Node<K,V> implements Map.Entry<K,V> {
            final int hash;
            final K key;
            V value;
            Node<K,V> next;
        }
        //TreeNode 大小大概是[2 * 普通节点空间]
        static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
            TreeNode<K,V> parent;  // red-black tree links
            TreeNode<K,V> left;
            TreeNode<K,V> right;
            TreeNode<K,V> prev;    // needed to unlink next upon deletion
            boolean red;
        }
        ```

#### 扩容

- 发生原因：当数据节点达到超过阈值

- 流程：

    1. 先插入值

    2. 扩容

        扩容处理，2倍容量，链表分类（一个是仍然在原相对位置的链表，一个是需要迁移到新位置的链表）

        对于节点是否需要迁移，经过1.7的扩容对比可以发现新位置相当于与原位置是否迁移取决于高位的值是1还是0

        1.8中直接对此位置bit进行判断以决定节点存储位置

        需要

        ```java
        final Node<K,V>[] resize() {
            //...省略其他代码
            if ( (e.hash & oldCap) == 0){
                newIndex = oldIndex;//此处不是源码的抽象实现只为了表示简单，实际代码可参见源码
            }else {
                newIndex = oldIndex + oldCap;//此处不是源码的抽象实现只为了表示简单
            }
            //...省略其他代码
        }
        ```

    3. 节点迁移

        将两个分类好的链表头分别放置到新table[]中

- 结果：数据顺序不变

## 源码定义

> 1.8

### 结构

继承: AbstractMap

接口：Map<K,V>, Cloneable, Serializable

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
}
```

### 内部常量

序列化ID

```java
private static final long serialVersionUID = 362498820763181265L
```

其他限制常量

```java
//默认初始容量
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
//最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
//扩容因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

```java
/**
  * Because TreeNodes are about twice the size of regular nodes, we use them only when bins contain enough nodes to   * warrant use(see TREEIFY_THRESHOLD). 
  * And when they become too small (due to removal or resizing) they are converted back to plain bins.  
  * In usages with well-distributed user hashCodes, tree bins are rarely used.
  * Ideally, under random hashCodes, the frequency of nodes in bins follows a Poisson distribution
  * (http://en.wikipedia.org/wiki/Poisson_distribution) with a parameter of about 0.5 on average for the default   	 * resizing threshold of 0.75, although with a large variance because of resizing granularity. 
  * Ignoring variance, the expected occurrences of list size k are (exp(-0.5) * pow(0.5, k) /  factorial(k)). 
  * The first values are:
  * 0:    0.60653066
  * 1:    0.30326533
  * 2:    0.0758163
  * 3:    0.01263606
  * 4:    0.00157952
  * 5:    0.00015795
  * 6:    0.00001316
  * 7:    0.00000094
  * 8:    0.00000006
  * more: less than 1 in ten million
  * 以上展示了：链表中的节点的个数达到上述数值的概率,链表达到8个结点的概概率是很低的
  */
//由链表变为树的阈值
static final int TREEIFY_THRESHOLD = 8;
//树--->链表，最小收缩检测值，小于[TREEIFY_THRESHOLD]6
static final int UNTREEIFY_THRESHOLD = 6;
```

## Hash值

> jdk1.8

**hashCode**

> hashCode()是一个native方法

int类型返回值，理论上可将实现近4亿的映射空间，但内存加载如此多的内存空间负担不小，而且对于小数据量而言没必要使用如此大的空间

应用hashCode的优势

实现对少量数据空间的映射，采用取模的策略，具体实现时采用的是^运算（高效）实现对定量空间的映射

**& (length-1)**

jdk1.7采用这种操作，实际上是对hash值的低16位数据进行了处理（实际使用时数据量一般个数不到2^16)，故采用这种方式实际上是只利用了低16位

jdk1.8采用 hash & (hash >>> 16)，将高16位得到参与与运算，从而更加增大了数值的随机性

> 右移16位：一般使用实际的数量少于[2^16]个，故始终是在以低位的数参与随机运算，右移16位获取到高位，运算结果更加随机
>
> ^预算：&，| 会使得结果偏向0或1（&只有1&1=1，|只有 0|0=0，其中得0，1的比率是并不是1:1），故采用^运算可使得结果更加平均

```java
static final int hash(Object key) {
    int h;
    //非空key，hash = [h]^[h>>>16]
    /**
      * [h >>> 16]{运算：无符号右移，取出h高16位}
      */
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## TablesSizeFor

> 限制：输出参数不可 [<0] [>MAXMAXIMUM_CAPACITY]
>
> 功能：返回大于输入参数且最近的2的整数次幂的数

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

## 字段

transient关键字确保其不会被序列化

```java
transient Node<K,V>[] table;// 第一次使用

transient Set<Map.Entry<K,V>> entrySet;//缓存

transient int size;// 映射关系总数

transient int modCount;//HashMap结构被修改次数

int threshold;//下一个要调整大小的大小值（容量*负载系数）。

final float loadFactor;//负载因子
```

## 查找结点

```java
final Node<K, V> getNode(int hash, Object key) {
    Node<K, V>[] tab;
    Node<K, V> first, e;
    int n;
    K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 表非空，获取其结点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            //检测第一个结点，是目标则返回，否则，在树里
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                //当前节点是树节点类型，则目前此表已演化为树型，调用树节点查找方式
                return ((TreeNode<K, V>) first).getTreeNode(hash, key);
            do {
                //当前节点不是是树节点类型，则目前此表是链表类型，遍历查找
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    //此处有一点，先比较hash，比较key,equals(方法可能由用户实现，其操作效率不能保证，先比较其hash在一定程度上减少由于equals方法的性能对比较的性能造成较大影响)
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

**容量和阈值的重置**

1. 原表有值
    1. 容量超过最大容量，阈值设置为Integer.MAX_VALUE
    2. 未超过限制，容量阈值均为原来的二倍
2. 原表无值，但阈值大于0，新容量为原阈值
3. 原表无值，阈值为0，则这是第一次初始化，设置初始容量，初始阈值

**数据迁移**

> 对于槽为是否变化
>
> 1.7重新计算Hash位
>
> 1.8 e.hash & oldCap == 0则在原地

1. 旧表非空
    1. 遍历旧表的每个元素
        1. 元素为结点，直接存入新空间（新hash槽为hash&(newCap-1)）
        2. 元素为树节点，分解树存入新空间
        3. 元素非节点，非树，则为链表
            1. 将原链表中的结点一一取出，并计算hash值，计算方式同上
                1. 计算结果为0，则在原槽
                    1. 将结点加入lo链表
                2. 结果非0，则在新槽
                    1. 将结点加入hi链表
                3. 将lo链表头节点放入原槽，hi链表头结点放入新槽（j + oldCap）
    2. 遍历旧表，直至结束

## 扩容

```java
final Node<K, V>[] resize() {
    Node<K, V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;

    if (oldCap > 0) {
        //旧容量大于0
        if (oldCap >= MAXIMUM_CAPACITY) {
            //旧容量超过了最大限制容量，重新设置阈值[Integer.MAX_VALUE],返回旧表
            threshold = Integer.MAX_VALUE;
            return oldTab;
        } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                   oldCap >= DEFAULT_INITIAL_CAPACITY)
            /**
                 * 容量未超过限制，同时大于默认初始化的空间值，则证明需要 {扩容}
                 * 新容量 = [原容量*2]
                 * 新阈值 = [原阈值*2]
                 * */
            newThr = oldThr << 1; // double threshold
    } else if (oldThr > 0) // initial capacity was placed in threshold
        //放开容量限制,现容量 = [旧阈值]
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        //新阈值 = [初始容量*加载因子]
        newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float) newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                  (int) ft : Integer.MAX_VALUE);
        //当前阈值为：[新容量 & ]
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes", "unchecked"})
    Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        //j = [0,tab.length)
        for (int j = 0; j < oldCap; ++j) {
            Node<K, V> e;
            if ((e = oldTab[j]) != null) {
                //旧表置空
                oldTab[j] = null;
                if (e.next == null)
                    //只有一个元素
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    //当前结构为树型
                    ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                else { // preserve order
                    //非树，非结点则为链表类型
                    Node<K, V> loHead = null, loTail = null;
                    Node<K, V> hiHead = null, hiTail = null;
                    Node<K, V> next;
                    /**
                      * 以下运算过程中
                      *      lo链表保存扩容后在原位的结点
                      *      hi链表保存扩容后需要移动位置的结点
                      * */
                    do {
                        next = e.next;
                        //链表创建过程：尾插法
                        if ((e.hash & oldCap) == 0) {
                            /**
                                 * 计算是否需要移动
                                 * [oldCap] = tab.length
                                 * [hash & oldCap] = 0,则在原位
                                 * */
                            if (loTail == null)
                                // 当前链表为空
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        } else {
                            //[hash & oldCap] != 0,需要移动到新的hash槽
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);

                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        // 需要挪动的结点移动至新的Hash槽
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

> [参考知乎](https://zhuanlan.zhihu.com/p/55890890)
>
> [参考个人博客](https://www.bbsmax.com/A/kmzLrgXA5G/)

