# Set

> HashSet & LinkedHashSet & TreeSet

# HashSet

## 特点

元素顺序无法保证

安全：不安全

至多有一个NULL

## 源码

### 构造器

```java
private transient HashMap<E,Object> map;

private static final Object PRESENT = new Object();

public HashSet() {
    map = new HashMap<>();
}

public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

### 操作

```java
public int size() {
    return map.size();
}

public boolean isEmpty() {
    return map.isEmpty();
}

public boolean contains(Object o) {
    return map.containsKey(o);
}

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

public void clear() {
    map.clear();
}
```

## 总结

底层基于HashMap，方法执行大都也是调用HashMap的方法

# TreeSet

## 特点

有序集合（自然排序，或者基于比较器的排序方式）

## 源码

### 构造器

```java
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

public TreeSet() {
    this(new TreeMap<E,Object>());
}

public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```

### 操作方法

```java
public int size() {
    return m.size();
}

public boolean isEmpty() {
    return m.isEmpty();
}

public boolean contains(Object o) {
    return m.containsKey(o);
}

public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}

public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}

public void clear() {
    m.clear();
}
```

## 总结

源码显示，TreeSet的数据域就是Map或者说是TreeMap，多数方法的调用也就是通过调用TreeMap的方法来实现的

# LinkedHashSet

## 特点

## 源码