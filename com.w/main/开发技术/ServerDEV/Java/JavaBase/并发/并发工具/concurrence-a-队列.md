# 并发队列

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729180034452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

## ConcurrentLinkedQueue

> 实现线程安全队列：1.阻塞算法（入队出队一把锁，或者两把锁）；2.非阻塞算法（可以用CAS实现）

### 入队列

尾插法

### 出队列

1. 取头结点元素，判断是否为空
    1. 空->另有线程执行了出队操作
    2. 非空->CAS方式将头结点引用改为null
        1. CAS成功，则操作成功
        2. CAS失败，队列已修改，需要重新获取头几点

## 阻塞队列

- ArrayBlockingQueue（有界-数组实现）

    ```java
    public class ArrayBlockingQueue<E> extends AbstractQueue<E>
            implements BlockingQueue<E>, java.io.Serializable {
        final Object[] items;
    }
    ```

- LinkedBlockingQueue（有界-链表实现）

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    static class Node<E> {
        E item;
        Node<E> next;
        Node(E x) { item = x; }
    }
    private final int capacity;// 容量限制
}
```

- LinkedTransferQueue（无界-链表实现）
- LinkedBlockingDeque（双向阻塞-链表实现）
- 
- PriorityBlockingQueue（无界-支持优先级排序）
- DelayQueue（无界-优先级队列实现）
- SynchronousQueue（不存储元素）

PriorityBlockingQueue（无界-支持优先级排序）

- 自定义实现compareTo()定义排序方式

DelayQueue（无界-优先级队列实现）

- 队列中元素需要实现DelayQueue
- 支持延时获取，只有当时间够了才允许从队列中获取当前元素

#### 实现原理

- ### 通知模式