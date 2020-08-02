# List

> ArrayList & LinkedList

# ArrayList

## 基础

继承结构

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
}
```

字段

```java
/**
     * [解决序列化认证问题]
     * 接收到序列化后的数据流，在恢复时会校验其序列化ID，也就是[serialVersionUID]
     * 若和本地的序列化ID一致则可序列化成功
     */
private static final long serialVersionUID = 8683452581122892189L;

//默认容量大小
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] EMPTY_ELEMENTDATA = {};

/**
     * [用于在实例化对象未赋值时默认填充数据空间]
     */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

transient Object[] elementData;

//当前真实数据的个数
private int size;

/**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

## 底层

构造器

```java
//无参构造器
public ArrayList() {
    //数据域设定为[默认空数据域]
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//指定初始容量
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        //容量为0， 则将数据域赋值为[EMPTY_ELEMENTDATA]
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
    }
}

//设置数据域
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

扩容

```java
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) ? 0 : DEFAULT_CAPACITY;
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 扩容机制 [oldCapacity*1.5]
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //当新容量大于最大限制容量会执行，hugeCapacity()方法
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    //若至少都需要[MAX_ARRAY_SIZE]大小，则将放弃保留部分VM的可能存放的头字段空间，即：容量=[Integer.MAX_VALUE]
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

增加

```java
//尾部追加策略
public boolean add(E e) {
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}
//指定索引追加
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```

删除

> 调用删除方法，参数若为对象，则需要写equals()方法

- 定向删除

    ```java
    public boolean remove(Object o) {
        //移除操作{0,size},且只执行一次，故而删除的的是符合删除值的[index]最小的那个元素
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                //对于值的判定采用的是equals方法
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    /*
         * Private remove method that skips bounds checking and does not
         * return the value removed.
         */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
    ```

- 范围删除

    ```java
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        //覆盖策略，将需要调整范围的值用其后面的值覆盖掉
        System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved);
    
        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            //将失效空间置空
            elementData[i] = null;
        }
        size = newSize;
    }
    ```

边界检查

```java
/**
 * A version of rangeCheck used by add and addAll.
 */
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

序列化操作

```java
//序列化
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();
    // 32位整型
    s.writeInt(size);
    // 写出所有有意义的数据，也就是当前size大小数据
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

//反序列化
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;
    // Read in size, and any hidden stuff
    s.defaultReadObject();
    // Read in capacity
    s.readInt(); // ignored
    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);
        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

## 操作

> 常用API

### 构造方法

| 构造方法                                  | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **ArrayList**()                           | 构造一个初始容量为 10 的空列表。                             |
| **ArrayList**(Collection <? extends E> c) | 构造一个包含指定 collection 的元素的列表， 这些元素是按照该 collection 的迭代器返回它们的顺序排列的。 |
| **ArrayList**(int initialCapacity)        | 构造一个具有指定初始容量的空列表                             |

### 常用方法

| 方法摘要                                                   | 描述                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| int **size**()                                             | 返回此列表中的元素数。                                       |
| boolean **add**(E e)                                       | 将指定的元素添加到此列表的尾部                               |
| void **add**(int index, E element)                         | 将指定的元素插入此列表中的指定位置                           |
| E **remove**(int index)                                    | 移除此列表中指定位置上的元素。                               |
| boolean **remove**(Object o)                               | 移除此列表中首次出现的指定元素（如果存在）。                 |
| void **clear**()                                           | 移除此列表中的所有元素。                                     |
| E **get**(int index)                                       | 返回此列表中指定位置上的元素。                               |
| boolean **contains**(Object o)                             | 如果此列表中包含指定的元素，则返回 true。                    |
| int **indexOf**(Object o)                                  | 返回此列表中首次出现的指定元素的索引，或如果此列表不包含元素，则返回 -1。 |
| int **lastIndexOf**(Object o)                              | 返回此列表中最后一次出现的指定元素的索引，或如果此列表不包含索引，则返回 -1 |
| Object **clone**()                                         | 返回此 ArrayList 实例的浅表副本。                            |
| boolean **isEmpty**()                                      | 如果此列表中没有元素，则返回 true                            |
| boolean **addAll**(Collection<? extends E> c)              | 按照指定 collection 的迭代器所返回的元素顺序，将该 collection 中的所有元素添加到此列表的尾部。 |
| boolean **addAll**(int index, Collection<? extends E> c)   | 从指定的位置开始，将指定 collection 中的所有元素插入到此列表中。 |
| void ensureCapacity(int minCapacity)                       | 如有必要，增加此 ArrayList 实例的容量，以确保它至少能够容纳最小容量参数所指定的元素数。 |
| protected void **removeRange**(int fromIndex, int toIndex) | 移除列表中索引在 fromIndex（包括）和 toIndex（不包括）之间的所有元素。 |
| E **set**(int index, E element)                            | 用指定的元素替代此列表中指定位置上的元素。                   |
| Object[] **toArray**()                                     | 按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组。 |
| `<T> T[] `**toArray**(T[] a)                               | 按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。 |
| void **trimToSize**()                                      | 将此 ArrayList 实例的容量调整为列表的当前大小                |

## 应用

效率：ArrayList底层采用数组实现，故具有了数组的随机读取的特性，同样也继承了数组的大量的增加，删除操作方面的劣势。

安全性：并未提供锁，故而是线程不安全的

限制：在存储值方面并无限制，数据域是Object[]

# LinkedList

## 基础

继承结构

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{
    //...
}
```

字段

```java
transient int size = 0;
//头指针
transient Node<E> first;
// 尾指针
transient Node<E> last;

/**
  * 不保证线程安全情况下对数据正确性的一个验证
  * 这同样也是{Fail-Fast 机制}的一个实际应用
  * 数据结构修改次数
  * 这个值的应用主要的迭代器中，迭代器在遍历整个数据域的时候并不能保证不被打扰的进行
  * 在迭代开始时存储[modCount],迭代过程中，若此值未修改则一切正常，若修改了，则直接抛出异常
  */
protected transient int modCount = 0;
```

## 底层

添加

> 在修改数据个数时同时修改了[modCount]值

```java
//头插法
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

//尾插法
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

插入

```java
//定向插入,前置插入
void linkBefore(E e, Node<E> succ) {
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

移除

```java
//首部移除
private E unlinkFirst(Node<E> f) {
    final E element = f.item;// 保留它的数据域
    final Node<E> next = f.next;// 保留它的后继
    // 置空数据域，移除和它有关联的所有连接
    f.item = null;
    f.next = null; // help GC
    first = next;// 指针后移
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}

//末尾移除
private E unlinkLast(Node<E> l) {
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;//尾结点前移
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}

//移除对象
public boolean remove(Object o) {
    if (o == null) {
        //遍历链表
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            //调用equals()方法
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

清空

```java
public void clear() {
    //遍历链表，依次将所有节点及其相关属性置空
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    //清空操作只记为一次结构性修改
    modCount++;
}
```

边界值判定

```java
private boolean isElementIndex(int index) {
    // 此处认为合法范围为[0, size),这和ArrayList不同
    // Linked是链表，且采用的是前置插入方式，可插入的位置个数应当是当前节点个数
    return index >= 0 && index < size;
}
```

序列化

> writeObject(readObject)的作用是序列化(反序列化)后两部分内容，即类的描述部分和属性域的值部分
>
> defaultWriteObect(defaultReadObject)的作用就是序列化(反序列化)最后一部分内容，即属性域的值部分

```java
//序列化
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out any hidden serialization magic
    s.defaultWriteObject();
    // Write out size
    s.writeInt(size);
    // Write out all elements in the proper order.
    for (Node<E> x = first; x != null; x = x.next)
        s.writeObject(x.item);
}

//反序列化
// 此处添加了抑制警告
@SuppressWarnings("unchecked")
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden serialization magic
    s.defaultReadObject();

    // Read in size
    int size = s.readInt();

    // Read in all elements in the proper order.
    for (int i = 0; i < size; i++)
        linkLast((E)s.readObject());
}
```
