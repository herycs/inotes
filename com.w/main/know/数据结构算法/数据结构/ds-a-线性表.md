# 线性表

## 线性表基本概念

特点：

- 同一性
- 有穷性
- 有序性

## 线性表的实现

> 存储方式：链式存储，顺序存储

### 基本概念

逻辑上相邻的物理存储上也是相邻的

计算函数：

- 第一个元素存放地址LOC(a1)，每个元素占用d个字节

    LOC（ai) = LOC(a1) + d x (i-1)	1<= i <= n

优势存取随机

### 实现

- 规定其接口

    ```java
    public interface MyList {
        void clear();
    
        boolean isEmpty();
    
        int length();
    
        Object get(int i) throws Exception;
    
        void insert(int i, Object x) throws Exception;
    
        void remove(int i) throws Exception;
    
        int indexOf(Object x);
    
        void display();
    }
    ```

#### 顺序存储实现

```java
public class MyArrayList implements MyList{

    private Object[] elements;

    private int length;

    public MyArrayList(int length) {
        this.elements = new Object[length];
        this.length = 0;
    }

    /**
     * @Desc:  [清空顺序表]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 14:51
     * @version : v1.0
     */
    public void clear() {
        this.length = 0;
    }

    /**
     * @Desc:  [是否为空]
     * @param :  void
     * @return : boolean
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 14:51
     * @version : v1.0
     */
    public boolean isEmpty() {
        return this.length == 0;
    }

    /**
     * @Desc:  [返回当前顺序表的长度]
     * @param :  void
     * @return : int
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 14:50
     * @version : v1.0
     */
    public int length() {
        return this.length;
    }

    /**
     * @Desc:  [返回指定位置数据值]
     * @param :  int
     * @return : Object
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 14:52
     * @version : v1.0
     */
    public Object get(int i) throws Exception {
        if (i < 0 && i > this.length)
            throw new NullPointerException("顺序表中不存在"+i+"位置");
        return elements[i];
    }

    /**
     * @Desc:  [在指定位置插入值,前置插入]
     *          eg:[ 1, 2, 3, 4, 5] ---> insert(2, 0) ---> [1, 0, 2, 3, 4, 5]
     * @param :  int , Object
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 14:59
     * @version : v1.0
     */
    public void insert(int i, Object in) throws Exception {
        //边界判定
        if (i == elements.length)
            throw new Exception("当前表已满，插入失败");
        if (i < 0 || i > elements.length)
            throw new NullPointerException("插入"+i+"位置不合适，数据"+in.toString()+"插入失败");
        //为即将插入的值调整空间
        for (int j = this.length; j > i; j--){
            elements[j] = elements[j-1];
        }
        //赋值修改顺序表元素长度记录
        elements[i] = in;
        this.length++;
    }

    /**
     * @Desc:  [移除当前位置所指向的元素]
     * @param :  int
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 15:17
     * @version : v1.0
     */
    public void remove(int i) throws Exception {
        //边界判定
        if (this.length == 0)
            throw new NullPointerException("顺序表已空");
        if (i > this.length || i < 0)
            throw new NullPointerException("要移除的"+i+"位置不存在");
        //元素左移，覆盖原有值，实现删除
        for (int j = i; j < this.length-1; j++) {
            elements[j] = elements[j+1];
        }
        //修改顺序表长度记录
        this.length--;
    }

    /**
     * @Desc:  [查找指定值在顺序表中的位置]
     * @param :  Object
     * @return : int
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 15:23
     * @version : v1.0
     */
    public int indexOf(Object x) {
        int index = 0;//记录当前扫描位置
        while (index < this.length && !elements[index].equals(x)){
            index++;
        }
        return index < this.length ? index : -1;
    }

    /**
     * @Desc:  [打印顺序表值]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 15:23
     * @version : v1.0
     */
    public void display() {
        System.out.print("[");
        for (int i = 0; i < this.length-1; i++) {
            System.out.print(" "+ elements[i] + ",");
        }
        System.out.print(" "+ elements[this.length-1] + "]\n");
    }
}
```

测试

```java
public static void main(String[] args) throws Exception {
    MyArrayList L = new MyArrayList(10);
    for (int i = 0; i < 8; i++)
        L.insert(i, i);
    System.out.println("删除索引为1的元素");
    L.remove(1);
    L.display();
    System.out.println("索引为3处插入元素");
    L.insert(3, 5);
    L.display();
    System.out.println("获取索引为1的元素值");
    System.out.println(L.get(1));
}
/* 测试结果为*/
删除索引为1的元素
[ 0, 2, 3, 4, 5, 6, 7]
索引为3处插入元素
[ 0, 2, 3, 5, 4, 5, 6, 7]
获取索引为1的元素值
2
```

#### 链式存储实现

```java
public class MyLinkedList<E> implements MyList {

    private E data;
    private MyLinkedList next;

    public MyLinkedList() {
    }

    /**
     * @Desc:  [初始化链表]
     * @param :  MyLinkedList
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 17:22
     * @version : v1.0
     */
    public MyLinkedList(Object x) {
        this.data = (E)x;
        this.next = null;
    }

    /**
     * @Desc:  [清空链表]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 17:22
     * @version : v1.0
     */
    public void clear() {
        this.next = null;
    }

    /**
     * @Desc:  [链表是否为空]
     * @param :  void
     * @return : boolean
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 17:23
     * @version : v1.0
     */
    public boolean isEmpty() {
        return this.data == null && this.next == null;
    }

    /**
     * @Desc:  [链表长度]
     * @param :  void
     * @return : int
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 17:23
     * @version : v1.0
     */
    public int length() {
        int count = 0;
        MyLinkedList linkedList = this;
        if (linkedList != null){
            count++;
            linkedList = linkedList.next;
        }
        return count;
    }

    /**
     * @Desc:  [获取索引为i的值]
     * @param : void
     * @return : Object
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 17:26
     * @version : v1.0
     */
    public E get(int i) throws Exception {
        int count = 0;
        MyLinkedList linkedList = this;
        while (count < i){
            linkedList = linkedList.next;
        }
        return (E)linkedList.data;
    }

    /**
     * @Desc:  [插入节点，前置插入]
     * @param :  int， Object
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 18:35
     * @version : v1.0
     */
    public void insert(int i, Object x) throws Exception {
        MyLinkedList myLinkedList = this.NodePreI(i);
        //创建新元素并嵌入链表中
        MyLinkedList newNode = new MyLinkedList(x);
        newNode.next = myLinkedList.next;
        myLinkedList.next = newNode;
    }

    /**
     * @Desc:  [移除第i个节点]
     * @param :  int
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 18:39
     * @version : v1.0
     */
    public void remove(int i) throws Exception {
        MyLinkedList myLinkedList = NodePreI(i);
        myLinkedList.next = myLinkedList.next.next;
    }

    /**
     * @Desc:  [返回第i-1个节点]
     * @param :  int
     * @return : MyLinkedList
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 18:54
     * @version : v1.0
     */
    private MyLinkedList NodePreI(int i) {
        MyLinkedList myLinkedList = this;
        if (this == null)
            throw new NullPointerException("链表为空，索引"+i+"位置无效");
        if (i < 0){
            throw new NullPointerException("索引位置越界，索引"+i+"无效");
        }
        //遍历到第i个节点前一个节点
        for (int j = 0; j < i-1; j++) {
            myLinkedList = myLinkedList.next;
        }
        return myLinkedList;
    }

    /**
     * @Desc:  [查找出第一个值为x的索引位置]
     * @param :  Object
     * @return : int -1代表查询无结果
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 18:40
     * @version : v1.0
     */
    public int indexOf(Object x) {
        MyLinkedList myLinkedList = this;
        if (this == null)
            throw new NullPointerException("链表为空，查找失败");
        MyLinkedList myLinkedList1 = this;
        int count = 0;
        while (myLinkedList1.next != null && myLinkedList1.data.equals(x)) {
            myLinkedList1 = myLinkedList1.next;
            count++;
        }
        return count > this.length() ? -1 : count;
    }

    /**
     * @Desc:  [打印当前链表的所有值]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 18:44
     * @version : v1.0
     */
    public void display() {
        MyLinkedList myLinkedList = this;
        while (myLinkedList.next != null){
            System.out.print(myLinkedList.data + "-->");
            myLinkedList = myLinkedList.next;
        }
        System.out.print("\b\b\b   \n");
    }
}
```

测试

```java
public static void main(String[] args) throws Exception {
    MyLinkedList myLinkedList = new MyLinkedList(0);
    for (int i = 0; i < 10; i++) {
        myLinkedList.insert(i, i);
    }
    System.out.println("建立链表如下：");
    myLinkedList.display();
    System.out.println("索引为1处插入数值9");
    myLinkedList.insert(1, 9);
    myLinkedList.display();
    System.out.println("移除索引为3的节点");
    myLinkedList.remove(3);
    myLinkedList.display();
    System.out.println("清除链表");
    myLinkedList.clear();
    myLinkedList.display();
}
/* 测试结果 */
建立链表如下：
0-->1-->2-->3-->4-->5-->6-->7-->8-->9   
索引为1处插入数值9
0-->9-->1-->2-->3-->4-->5-->6-->7-->8-->9   
移除索引为3的节点
0-->9-->1-->3-->4-->5-->6-->7-->8-->9   
清除链表
```

### 链式及顺序对比

| 项目                 | 链式 | 顺序 |
| -------------------- | ---- | ---- |
| 操作是否简单         | X    | O    |
| 没有额外开销         | X    | O    |
| 存储密度高           | X    | O    |
| 增加，删除操作效率高 | O    | X    |
| 不需预先分配空间     | O    | X    |

> 高级语言都有数组，但不一定有指针