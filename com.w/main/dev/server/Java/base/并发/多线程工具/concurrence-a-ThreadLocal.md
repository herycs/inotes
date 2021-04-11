# ThreadLocal

## 特征

ThreadLocal对象为键，任意对象为值的存储结构

## 源码

### 字段

数据结构的设计：

- 早期：以ThreadID为key，局部变量为Map的value

- 改进：以ThreadLocal为key，值是value

    优势：

    1. Map存储的Entry变小，之前由Thread数量决定，现在由ThreadLoacl决定
    2. 线程销毁后ThreadLocalMap也会销毁，生命周期与线程相同，减少内存的使用

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) { // k是ThreadLocal本身，存储的变量是Map的Value
        super(k);
        value = v;
    }
}

private static final int INITIAL_CAPACITY = 16;

private Entry[] table;

private int size = 0;

private int threshold; // Default to 0
```

### 测试

如下代码测试多线程对共享变量的访问

```java
static int num = 10;

private static void test1() {
    Thread[] threads = new Thread[5];
    for (int i = 0; i < threads.length; i++) {
        threads[i] = new Thread(()->{
            num+=3;
            System.out.println(Thread.currentThread().getName() + "-" + num);
        },"thread-" + i);
    }
    for (Thread t : threads) {
        t.start();
    }
}
```

结果可以看到线程的顺序不同，值的运算结果也不同

```java
thread-0-16
thread-2-19
thread-1-16
thread-3-22
thread-4-25
```

测试使用ThreadLocal

```java
ThreadLocal<Integer> integerThreadLocal = new ThreadLocal<Integer>(){
    @Override
    protected Integer initialValue() {
        return 5;
    }
};

Thread[] threads = new Thread[5];
for (int i = 0; i < threads.length; i++) {
    threads[i] = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " - " + integerThreadLocal.get().intValue());
    }, "thread-" + i);
}
for (Thread t : threads) {
    t.start();
}
```

结果线程的顺序完全不影响对值的读取，对值的修改自然也就不会受到其他线程的干扰

```java
thread-1 - 5
thread-0 - 5
thread-2 - 5
thread-3 - 5
thread-4 - 5
```

