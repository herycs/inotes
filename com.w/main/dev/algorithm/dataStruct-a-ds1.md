# java - 数据结构 - 基础数据结数据结构

> 整理所学，期待批评指教

[TOC]

## 基础概念

- 数据

  - 描述客观事物的数值、字符以及能输入到计算机中且能被处理的各种符号集合

- 数据元素

  - 组成数据的基本单位，数据集合的个体，计算机中通常作为一个整体进行考虑

- 数据对象

  - 性质相同的数据元素的集合，数据的一个子集

- 数据结构

  - 相互之间存在一种或多种特定关系的数据元素的集合

  - 是一个二元组：

    > D --->数据有限集
    >
    > R --->D上关系的有限集

    - Data_Stracture = (D, R)

- 数据类型

  - 性质相同的值集合以及定义在这个值集合上的一组操作的总称。

- 数据抽象与抽象数据类型

  - 抽象数据类型（Abstract Data Type）ADT
    - 含义：数据模型及定义在数据模型上的操作
    - 特征：使用与实现分离，实现信息封装及隐蔽
  - 抽象数据类型实现
    - 1.面向过程设计
      - 逻辑架构--->选定存储结构，操作要求--->设计子程序或子函数
    - 2.“包”，“模型”设计方法
      - 可单独编译，同时提供了外部使用的方便
    - 3.面向对象程序设计（Object Oriented Programming，OOP）
      - 着眼于应用问题涉及的对象，包括对象，对象属性和要求

# 1.数据结构内容

## 1.1 数据的逻辑结构

> 数据逻辑间关系的描述

- 依据数据间不同关系划分
  - 集合结构：结构中数据元素除同属于一个集合外无任何关系
  - 线性结构：结构中数据元素存在着一对一的线性关系
  - 树型结构：结构中的数据元素之间存在着一对多的层次关系
  - 图状结构：结构中的数据元素之间存在着多对多的任意关系
- 依据数据元素间关系的不同特性
  - 线性结构：
    - 线性表，栈，队列，字符串，数组和广义表
  - 非线性结构：
    - 树和图

## 1.2 数据的存储结构

- 数据元素在计算机中的表示方式
  - 顺序映像--->顺序存储
  - 非顺序映像--->链式存储

## 1.3 数据的运算

> Niklaus Wirth提出的著名公式：数据结构 + 算法 = 程序

- 算法特征
  - 有穷性：任意一组合法输入值，又穷步骤后一定能结束
  - 确定性：每一步是确切定义的，算法的执行者或者阅者都能明确其含义以及如何执行，并且在任何条件下，算法只有一条执行路径
  - 可行性：算法中所有操作必须足够基本，都可以通过已实现的基本操作运算有限次实现
  - 有输入：算法应该有0个或多个输入
  - 有输出：算法应该有1个或多个输出
- 算法评定标准
  - 正确性
  - 可读性
  - 健壮性
  - 高效率与低存储量需求

# 2.线性表

## 2.1 线性表基本概念

- 特点：
  - 同一性
  - 有穷性
  - 有序性

## 2.2 线性表的实现

> 存储方式：链式存储，顺序存储

### 2.2.1 基本概念

- 逻辑上相邻的物理存储上也是相邻的

- 计算函数：

  - 第一个元素存放地址LOC(a1)，每个元素占用d个字节

    LOC（ai) = LOC(a1) + d x (i-1)	1<= i <= n

- 优势存取随机

### 2.2.2 实现

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

#### 2.2.2.1顺序存储实现

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

#### 2.2.2.2 链式存储实现

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

### 2.2.3 链式及顺序对比

| 项目                 | 链式 | 顺序 |
| -------------------- | ---- | ---- |
| 操作是否简单         | X    | O    |
| 没有额外开销         | X    | O    |
| 存储密度高           | X    | O    |
| 增加，删除操作效率高 | O    | X    |
| 不需预先分配空间     | O    | X    |

> 高级语言都有数组，但不一定有指针

# 3.栈

## 3.1 基本概念

- 栈
  - 只允许在一端（栈顶）进行插入和删除操作的线性表，另外一端称为栈底
  - 类别：单栈，多栈共享空间

## 3.2 栈的实现

### 3.2.1 静态栈

```java
public class MySeqStack implements MyStack {

    private final int DEFAULT_SIZE = 10;
    private Object[] data;
    private int MAX_SIZE;
    private int top;

    public MySeqStack() {
    }

    public MySeqStack(int top) {
        this.MAX_SIZE = top;
        this.data = new Object[top];
        this.top = -1;
    }

    /**
     * @Desc:  [初始化栈]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 19:54
     * @version : v1.0
     */
    public void InitStack() {
        this.MAX_SIZE = DEFAULT_SIZE;
        this.data = new Object[DEFAULT_SIZE];
        this.top = -1;
    }

    /**
     * @Desc:  [是否为空]
     * @param :  void
     * @return : boolean
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 19:57
     * @version : v1.0
     */
    public boolean IsEmpty() {
        return this.top == -1;
    }

    /**
     * @Desc:  [入栈]
     * @param :  data 入栈值
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 20:05
     * @version : v1.0
     */
    public void Push(Object data) {
        if ( this.top == MAX_SIZE){
            throw new NullPointerException("栈已满,入栈失败");
        }
        this.data[++top] = data;
    }

    /**
     * @Desc:  [出栈]
     * @param :  void
     * @return : Object 弹出值
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 20:05
     * @version : v1.0
     */
    public Object Pop() {
        if (this.top < 0){
            throw new NullPointerException("栈已空，出栈失败");
        }
        return this.data[this.top--];
    }

    /**
     * @Desc:  [获取栈顶元素]
     * @param :  void
     * @return : Object
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 20:06
     * @version : v1.0
     */
    public Object GetTop() {
        return this.IsEmpty() ? null : this.data[top];
    }

    /**
     * @Desc:  [置空栈]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 20:44
     * @version : v1.0
     */
    public void SetEmpty() {
        this.data = null;
        this.top = -1;
    }

    /**
     * @Desc:  [打印栈内所有元素]
     * @param :  void
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 20:44
     * @version : v1.0
     */
    public void display() {
        System.out.print("[");
        if (top != -1) {
            for (int i = 0; i < this.top; i++) {
                System.out.print(" " + this.data[i] + ",");
            }
        System.out.print(" "+this.data[top]);
        }
        System.out.println("]\n");
    }
}
```

### 3.2.2 静态双端栈

```java
public class BothStack {

    private int DEFAULT_DEEP = 20;//默认栈大小
    private int MAX_SIZE;//自定义的栈的最大值
    private Object[] data;//元素顺序表
    private int leftTop;//左栈顶
    private int rightTop;//右栈顶

    public BothStack() {
        this.InitStack();
    }

    public BothStack(int totalLength) {
        this.MAX_SIZE = totalLength;
        this.data = new Object[this.MAX_SIZE];
        this.leftTop = -1;
        /*
            此处不加1是因为操作的是索引，其最大值就是栈的最右端加一
            this.rightTop = MAX_SIZE + 1;
         */
        this.rightTop = MAX_SIZE;
    }

    public void InitStack() {
        this.data = new Object[DEFAULT_DEEP];
        this.MAX_SIZE = DEFAULT_DEEP;
        this.leftTop = -1;
        this.rightTop = DEFAULT_DEEP;
    }

    /**
     * @param : void
     * @return : void
     * @Desc: [判全栈是否为空]
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 21:16
     * @version : v1.0
     */
    public boolean IsEmpty() {
        return leftTop == -1 && rightTop == MAX_SIZE;
    }

    /**
     * @param : String
     * @return : boolean
     * @Desc: [判断指定栈是否为空]
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 21:18
     * @version : v1.0
     */
    public boolean IsEmpty(String stack) {
        if (stack.equals("L") && this.leftTop == -1) {
            return true;
        }
        if (stack.equals("R") && this.rightTop == MAX_SIZE) {
            return true;
        }
        return false;
    }

    /**
     * @Desc:  [向栈内压入元素]
     * @param :  stack --- 操作栈[R: 右栈，L：左栈]， data --- 压入元素值
     * @return : void
     * @author :  ANGLE0
     * @createTime ：  2019/12/22 0:39
     * @version : v1.0
     */
    public void Push(String stack, Object data) throws Exception {
        if (this.leftTop + 1 == this.rightTop)
            throw new Exception("栈已满,入栈失败");
        if (stack.equals("L")) {
            this.data[++leftTop] = data;
        } else if (stack.equals("R")) {
            this.data[--rightTop] = data;
        } else {
            throw new Exception("命令错误");
        }
    }

    /**
     * @Desc:  [弹出栈顶元素]
     * @param :  stack --- 操作栈[R: 右栈，L：左栈]
     * @return : Object
     * @author :  ANGLE0
     * @createTime ：  2019/12/22 0:39
     * @version : v1.0
     */
    public Object Pop(String stack) throws Exception {
        this.IsEmpty();
        //取左栈元素
        if (stack.equals("L")) {
            //左栈非空
            if (!IsEmpty(stack))
                return this.data[leftTop--];
            throw new Exception("左栈已空");
            //取又栈元素
        } else if (stack.equals("R")) {
            //右栈非空
            if (!IsEmpty(stack))
                return this.data[rightTop++];
            throw new Exception("右栈已空");
        } else {
            throw new Exception("命令错误");
        }
    }

    /**
     * @param : stack --- 操作栈[R: 右栈，L：左栈]
     * @return : Object
     * @Desc: [获取栈顶元素值]
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 22:59
     * @version : v1.0
     */
    public Object GetTop(String stack) throws Exception {
        //取左栈元素
        if (stack.equals("L")) {
            //左栈非空
            if (!IsEmpty(stack))
                return this.data[leftTop];
            throw new Exception("左栈已空");
            //取右栈元素
        } else if (stack.equals("R")) {
            //右栈非空
            if (!IsEmpty(stack))
                return this.data[rightTop];
            throw new Exception("右栈已空");
        } else {
            throw new Exception("命令错误");
        }
    }

    /**
     * @param : void
     * @return : void
     * @Desc: [置空全栈元素]
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 23:58
     * @version : v1.0
     */
    public void SetEmpty() {
        this.data = null;
        this.leftTop = -1;
        this.rightTop = this.MAX_SIZE;
    }

    /**
     * @param : stack --- 操作栈[R: 右栈，L：左栈]
     * @return : void
     * @Desc: [置空左或者右栈]
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 23:58
     * @version : v1.0
     */
    public void SetEmpty(String stack) throws Exception {
        if (stack.equals("L"))
            this.leftTop = -1;
        else if (stack.equals("R"))
            this.rightTop = MAX_SIZE;
        else
            throw new Exception("命令错误，置空失败");
    }

    /**
     * @param : void
     * @return : void
     * @Desc: [打印全栈元素]
     * # --- 表示空元素
     * @author :  ANGLE0
     * @createTime ：  2019/12/21 23:57
     * @version : v1.0
     */
    public void display() {
        System.out.print("[");
        if (!this.IsEmpty()) {
            for (int i = 0; i < this.MAX_SIZE - 1; i++) {
                if (i > this.leftTop && i < this.rightTop) {
                    System.out.print(" " + "#" + ",");
                } else {
                    System.out.print(" " + this.data[i] + ",");
                }
            }
            //打印最后一个元素
            System.out.print(" " + this.data[this.MAX_SIZE - 1]);
        }
        System.out.println("]\n");

    }

    /**
     * @Desc:  [打印指定栈的元素]
     * @param :  stack --- 操作栈[R: 右栈，L：左栈]
     * @return : 
     * @author :  ANGLE0
     * @createTime ：  2019/12/22 0:43
     * @version : v1.0
     */
    public void display(String stack) {
        int start = 0;
        int end = MAX_SIZE;

        if (this.IsEmpty()) {
            start = 0;
            end = 0;
        } else if (stack.equals("L") && !this.IsEmpty(stack)) {
            //指令打印左栈且左栈非空
            end = this.leftTop;
            start = 0;
            //指令打印右栈且右栈非空
        } else if (stack.equals("R") && !this.IsEmpty(stack)) {
            start = this.rightTop;
            end = MAX_SIZE - 1;
        }
        //正式打印
        System.out.print("[");
        if (!this.IsEmpty()) {
            for (int i = start; i < end; i++) {
                System.out.print(" " + this.data[i] + ",");
            }
            System.out.print(" " + this.data[end]);
        }
        System.out.println("]\n");
    }
}
```

测试

```java
public static void main(String[] args) throws Exception {
        BothStack bothStack = new BothStack();

        for (int i = 0; i < 10; i++) {
            bothStack.Push("L", i);
        }
        for (int i = 19; i > 9; i--) {
            bothStack.Push("R", i);
        }
        System.out.println("初始化栈结束");
        inner.show(bothStack);

        System.out.println("左栈弹出三次");
        for (int i = 0; i < 3; i++) {
            System.out.println(bothStack.Pop("L"));
        }
        System.out.println("右栈弹四次");
        for (int i = 0; i < 4; i++) {
            System.out.println(bothStack.Pop("R"));
        }
        inner.show(bothStack);

        System.out.println("左栈入，1,2");
        for (int i = 0; i < 2; i++) {
            bothStack.Push("L", i+1);
        }
        inner.show(bothStack);
        System.out.println("右栈入6,7,8");
        for (int i = 0; i < 3; i++) {
            bothStack.Push("R", i+6);
        }
        inner.show(bothStack);

        System.out.println("置空");
        bothStack.SetEmpty("L");
        bothStack.SetEmpty("R");
        inner.show(bothStack);
    }
```

### 3.2.3 链式存储

> 为节省空间，下面的注释就简写了

```java
public class LinkedStack {

    class Node {
        Object data;
        Node next;
        public Node() {
        }
        public Node(Object data) {
            this.data = data;
        }
    }

    private int size;
    private Node top;

    public LinkedStack() {
    }

    public LinkedStack(Object newData) {
        Node node = new Node(newData);
        node.next = this.top;
        this.top= node;
        size++;
    }
    //初始化栈
    public void InitStack() {
        top.data = null;
        top.next = null;
        int size = 0;
    }
    //判空
    public boolean IsEmpty() {
        return this.size() == 0;
    }
    //链栈的长度
    public int size() {
        return size;
    }
    //入栈
    public void Push(Object data) throws Exception {
        Node node = new Node(data);
        if (node == null)
            throw new Exception("节点创建失败，入栈失败");
        node.next = this.top;
        this.top = node;
        size++;
    }
    //出栈
    public Object Pop() throws Exception {
        if (this.top.next == null)
            throw new Exception("栈已空，出栈失败");
        Node node = this.top;
        this.top = node.next;
        this.size--;
        return node.data;
    }
    //获取栈顶元素
    public Object GetTop() {
        return this.top.data;
    }
    //置空栈
    public void SetEmpty() {
        this.top.next = null;
        this.top.data = null;
        size = 0;
    }
    //打印栈内元素
    public void display() throws Exception {
        System.out.print("[");
        if (!this.IsEmpty()) {
            Node node = this.top;
            while (node.next != null) {
                System.out.print(" " + node.data + ",");
                node = node.next;
            }
            System.out.print(" " + node.data);
        }
        System.out.print("]\n");
    }
}
```

测试

```java
public static void main(String[] args) throws Exception {
        LinkedStack linkedStack = new LinkedStack();

        for (int i = 0; i < 10; i++) {
            linkedStack.Push(i+1);
        }
        System.out.println("初始化结束");
        linkedStack.display();
        System.out.println("出栈三次");
        for (int i = 0; i < 3; i++) {
            System.out.println(linkedStack.Pop());
        }
        linkedStack.display();
        System.out.println("入栈二次,1,2");
        for (int i = 0; i < 2; i++) {
            linkedStack.Push(i+1);
        }
        linkedStack.display();
        System.out.println("栈深度");
        System.out.println(linkedStack.size());
        System.out.println("栈置空");
        linkedStack.SetEmpty();
        linkedStack.display();
    }
```

### 3.2.4 多链栈

> 操作两个以上的栈时使用顺序栈共享临界空间不是很方便，用多个单链栈操作较便利，将多个单链栈的栈顶放置于一个一维数组中

> 0【】---> 【 | 】 ---> 【 | 】---> 【 | ^】
>
> 1【】---> 【 | 】 ---> 【 | ^】
>
> ...

# 4.队列

## 4.1 基本概念

- 运行在表的一端（队尾，rear）进行插入，另一端（队头，font）进行删除
- 特征：FIFO（First In First Out）

## 4.2 队列的实现

### 4.2.1 静态队列

- 单向队列：申请一段空间，队头队尾指向同一个位置（假定为索引较小方向为队头，初始化时队头队尾都是-1），随着入队（队尾指针增加）出队（队头指针减少）的操作的进行
- 缺点：导致假溢出，会导致空间的浪费

```java
public class SeqQueue implements MyQueue {

    private int MAX_SIZE;
    private int DEAFULE_SIZE = 10;
    private Object[] data;
    private int front;
    private int rear;

    public SeqQueue() {
        this.InitQueue();
    }
    public SeqQueue(int size) {
        this.Init(size);
    }
    public void InitQueue() {
        this.Init(DEAFULE_SIZE);
    }
    public void InQueue(Object data) throws Exception {
        if (this.rear+1 == this.MAX_SIZE)
            throw new Exception("队列已满");
        this.data[++this.rear] = data;
    }
    public Object OutQueue() throws Exception {
        if (this.front+1 > this.rear)
            throw new Exception("队列已空");
        return this.data[++this.front];
    }
    public Object FrontQueue() {
        return this.data[this.front+1];
    }
    public boolean EmptyQueue() {
        return this.front+1 > this.rear;
    }
    //私有方法，给定初值初始化队列
    private void Init(int size){
        this.MAX_SIZE = size;
        this.data = new Object[this.MAX_SIZE];
        this.front = -1;
        this.rear = -1;
    }
    //打印
    public void display(){
        System.out.print("[");
        if (!this.EmptyQueue()){
            int index = this.front+1;
            while (index < this.rear){
                System.out.print(" "+this.data[index++]+",");
            }
            System.out.print(" "+this.data[index]);
        }
        System.out.print("]\n");
    }
}
```

测试：

```java
public static void main(String[] args) throws Exception {
        SeqQueue seqQueue = new SeqQueue();

        for (int i = 0; i < 7; i++) {
            seqQueue.InQueue(i+1);
        }
        System.out.println("初始化队列成功");
        seqQueue.display();
        System.out.println("出队列三次");
        for (int i = 0; i < 3; i++) {
            System.out.println(seqQueue.OutQueue());
        }
        seqQueue.display();
        System.out.println("入队列两次");
        for (int i = 0; i < 2; i++) {
            seqQueue.InQueue(i+1);
        }
        seqQueue.display();
        System.out.println("队头值"+seqQueue.FrontQueue());
        System.out.println("队列是否为空"+seqQueue.EmptyQueue());
        System.out.println("入队5次");
        for (int i = 0; i < 5; i++) {
            seqQueue.InQueue(i+1);
        }
        seqQueue.display();
    }
/* 测试结果*/
初始化队列成功
[ 1, 2, 3, 4, 5, 6, 7]
出队列三次
1
2
3
[ 4, 5, 6, 7]
入队列两次
[ 4, 5, 6, 7, 1, 2]
队头值4
队列是否为空false
入队5次
//此处队列的尾部在索引为9处，队列的0~2索引位置为空，但却无法再插入元素了，显然这里空间一定程度上造成了浪费
Exception in thread "main" java.lang.Exception: 队列已满
	at algorithm.base.queue.SeqQueue.InQueue(SeqQueue.java:34)
	at algorithm.base.queue.QueueTest.main(QueueTest.java:33)
```

### 4.2.2 循环队列

- 解决假溢出

  - 将数据区作为首位相接的循环数据区进行操作

  - 入队时的操作需要修改：

    ```java
    //通过取模对数据的空间实现循环操作，通过代码的理解如下
    this.rear = (this.rear+1) % MAX_SIZE;
    ```

  - 出队时：

    ```java
    //通过取模实现循环操作,通过代码理解如下
    this.front = (this.front+1) % MAX_SIZE
    ```

  - 判空：

    ```java
    this.front == rear;
    ```

  - 判满：

    ```java
    //为了和队空区分开一般可以用以下两种方式判满：
    1.少用一个元素空间，队尾+1就指向对头
    		(this.rear+1) % MAX_SIZE = this.front;
    2.添加一个记录队列中元素个数的变量
            例如：size {size == 0 => 队空，size == MAX_SIZE => 队满}
    ```

### 4.2.3 链队列

- 基本操作：

  - 入队列：
    - 创建新节点：newNode
    - 新节点的后续节点指向置空：newNode.next = null;
    - 当前尾节点的后续节点指向新节点：queue.rear.next = newNode
    - 新的节点作为队列的尾节点：queue.rear = newNode;

- 实现

  ```java
  public class LinkedQueue {
  
      class Node{
          Object data;
          Node next;
          public Node() {
          }
          public Node(Object data) {
              this.data = data;
          }
      }
  
      class Queue{
          //队头
          private Node front;
          //队尾
          private Node rear;
      }
  
      //当前队列
      private Queue queue;
  
      public LinkedQueue() {
          this.InitQueue();
      }
      //初始化
      public void InitQueue(){
          Node node = new Node();
          node.next = null;
          node.data = null;
          Queue queue = new Queue();
          queue.front = node;
          queue.rear = node;
          this.queue = queue;
      }
      //入队列
      public void InQueue(Object data) throws Exception{
          Node node = new Node(data);
          if (node == null)
              throw new Exception("申请节点失败，入队列失败");
          node.next = null;
          this.queue.rear.next = node;
          this.queue.rear = node;
      }
      //出队列
      public Object OutQueue() throws Exception{
          if (this.queue.front.next == null)
              throw new Exception("当前队列为空，出队失败");
          Node node = this.queue.front.next;
          this.queue.front.next = node.next;
          return node.data;
      }
      //队头元素
      public Object FrontQueue() throws Exception {
          if (this.queue.front.next == null)
              throw new Exception("当前队列为空，出队失败");
          return this.queue.front.next.data;
      }
      //判空
      public boolean EmptyQueue(){
          return this.queue.front.next == null;
      }
      //打印
      public void display(){
          System.out.print("[");
          if (!this.EmptyQueue()){
              Node node = this.queue.front.next;
              while (node.next != null){
                  System.out.print(" "+node.data+",");
                  node = node.next;
              }
              System.out.print(" "+node.data);
          }
          System.out.print("]\n");
      }
  }
  ```

- 测试

  ```java
  public static void main(String[] args) throws Exception {
      LinkedQueue linkedQueue = new LinkedQueue();
      for (int i = 0; i < 10; i++) {
          linkedQueue.InQueue(i+1);
      }
      System.out.println("队列初始化结束");
      linkedQueue.display();
      System.out.println("出队列三次");
      for (int i = 0; i < 3; i++) {
          System.out.println(linkedQueue.OutQueue());
      }
      linkedQueue.display();
      System.out.println("入队列两次");
      for (int i = 0; i < 2; i++) {
          linkedQueue.InQueue(i+1);
      }
      linkedQueue.display();
      System.out.println("队列为空否"+linkedQueue.EmptyQueue());
  }
  /* 测试结果 */
  队列初始化结束
      [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
      出队列三次
      1
      2
      3
      [ 4, 5, 6, 7, 8, 9, 10]
      入队列两次
      [ 4, 5, 6, 7, 8, 9, 10, 1, 2]
      队列为空否false
  ```

# 5.串

## 5.1 基本概念

- 串：0个或多个字符组成的有限序列，记作：S=’a1a2a3a4...an'(n>=0)
  - S：串的名字
  - ‘’中的字符序列：串的值（单引号本身不属于串）
  - ai(1<= i <=n)：具体可以是 字母，数字和其它字符（这些取决于采用的字符集）
  - n：串中字符的个数，串的长度
- 术语
  - 空串：长度为0，不包含任何字符
  - 空白串：有一个或多个空格组成的串，其n >= 1
  - 子串：包含子串的串称为主串
  - 前缀子串：S的一个前缀子串是串U
    - U =  ‘a1 a2 a3 ... a**b**'(1<=b**<**n)，U为S的真前缀子串
    - 通俗点说就是不包含最后一个字符
  - 后缀子串：S的一个后缀子串是串U
    - U =  ‘a(**n-b+1**)  ... a**n**'(1<=b**<**n)，U为S的真后缀子串
    - 通俗点说就是不包含第一个字符
  - 串的相等：当且仅当两个串的值相等时，这两个串相等
  - 模式匹配：确定子串从主串的某个位置开始后，在主串中首次出现的位置的运算，主串称为目标串，子串称为模式串

## 5.2 串的实现

### 5.2.1 定长顺序串实现

- 字符组定义

  ```java
  package algorithm.base.string;
  
  import algorithm.base.queue.SeqQueue;
  
  /**
   * @ClassNameSeqString
   * @Description
   * @Author ANGLE0
   * @Date 2019/12/22 17:12
   * @Version V1.0
   **/
  public class SeqChar {
  
      private char[] chars;
      private int len;
      private int MAX_SIZE;
      private int DEFAULT_SIZE = 50;
  
      public SeqChar() {
          new SeqChar(DEFAULT_SIZE);
      }
  
      public SeqChar(int len) {
          this.MAX_SIZE = len;
          this.chars = new char[len + 1];
          this.len = 0;
      }
  
      public SeqChar(char[] chars) {
          this.chars = new char[chars.length + 1];
          this.setChars(chars);
          this.len = chars.length;
          this.MAX_SIZE = chars.length;
  
      }
  
      public SeqChar(char[] chars, int len) {
          this.chars = new char[len + 1];
          this.setChars(chars);
          this.len = chars.length;
          this.MAX_SIZE = len;
      }
  
      /**
       * @param :
       * @return :
       * @Desc: [S的pos位开始插入T]
       * @author :  ANGLE0
       * @createTime ：  2019/12/22 21:19
       * @version : v1.0
       */
      public static int insert(SeqChar S, int pos, SeqChar T) throws Exception {
          if (null == S || null == S.chars || null == T.chars || null == T || pos < 0 || pos > S.len + 1)
              throw new Exception("插入位置不合法");
          //S串T串可以全部保留
          if (S.len + T.len < S.MAX_SIZE) {
              //为插入串整理空间
              for (int i = S.len + T.len; i >= pos + T.len; i--) {
                  S.chars[i] = S.chars[i - T.len];
              }
              //插入字符串
              for (int i = pos; i < pos + T.len; i++) {
                  S.chars[i] = T.chars[i - pos + 1];
              }
              S.len = S.len + T.len;
              return 0;
              //T串可完整插入，S有可能会被截断
          } else if (pos + T.len <= S.MAX_SIZE) {
              //将第pos~pos+T.len位的所有字符后移
              for (int i = S.MAX_SIZE; i >= pos + T.len; i--) {
                  S.chars[i] = S.chars[i - T.len];
              }
              for (int i = pos; i < pos + T.len; i++) {
                  S.chars[i] = T.chars[i - pos + 1];
              }
              S.len = S.MAX_SIZE;
              return 0;
          } else {
              //插入串T会被截断
              for (int i = pos; i < S.MAX_SIZE; i++) {
                  S.chars[i] = T.chars[i - pos + 1];
              }
              S.len = S.MAX_SIZE;
              return 0;
          }
      }
  
      /**
       * @param :
       * @return :
       * @Desc: [S的pos位开始删除到pos+len范围的字符]
       * @author :  ANGLE0
       * @createTime ：  2019/12/22 21:19
       * @version : v1.0
       */
      public static void delete(SeqChar S, int pos, int len) throws Exception {
          if (null == S || null == S.chars || pos < 0 || pos > S.len + 1)
              throw new Exception("删除参数不合法");
          for (int i = pos; i <= S.len - len; i++) {
              S.chars[i] = S.chars[i + len];
          }
          S.len = S.len - len;
      }
  
  
      public int strCat(SeqChar S, SeqChar T) throws Exception {
          if (null == S || null == T || null == T.chars)
              throw new Exception("参数不合法，拼接失败");
          //可完全拼接
          if (S.len + T.len <= MAX_SIZE) {
              for (int i = S.len + 1; i <= S.len + T.len; i++) {
                  S.chars[i] = T.chars[i - S.len];
              }
              S.len = S.len + T.len;
              return 0;
          } else if (S.len < MAX_SIZE) {
              //可拼接部分
              for (int i = S.len + 1; i < MAX_SIZE; i++) {
                  S.chars[i] = T.chars[i - S.len];
              }
              S.len = MAX_SIZE;
              return 0;
          }
          return 0;
      }
  
      /**
       * @param :
       * @return :
       * @Desc: [打印字符]
       * @author :  ANGLE0
       * @createTime ：  2019/12/22 21:21
       * @version : v1.0
       */
      public void display() {
          System.out.print("[");
          if (this.len != 0) {
              for (int i = 0; i < this.len - 1; i++) {
                  System.out.print(" '" + this.chars[i + 1] + "',");
              }
              System.out.print(" '" + this.chars[this.len]+"'");
          }
          System.out.println("]\n");
      }
  
      public void setChars(char[] chars) {
          for (int i = 0; i < chars.length; i++) {
              this.chars[i + 1] = chars[i];
              this.len++;
          }
      }
  }
  ```

- 测试

  ```java
  public static void main(String[] args) throws Exception {
  
          SeqChar a = new SeqChar(new char[]{'a', 'b', 'c', 'd', 'e'}, 15);
          SeqChar b = new SeqChar(new char[]{'o', 'x', 'e', 'g', 'u'});
  
          a.display();
          b.display();
          System.out.println("初始化结束");
  
          System.out.println("3号位插入S");
          a.insert(a, 3,b);
          a.display();
  
          System.out.println("删除第3-5字符");
          a.delete(a, 3, 5);
          a.display();
  
      }
  ```

### 5.2.2 堆串

- 与定长顺序串结构类似，均采用一组连续的存储单元存放，但空间在使用时分配
- 操作流程：
  
  - 创建串时不设定其存储单元，当赋值时通过对值的长度的分析，申请内存，同时将值保存
  
    ```java
    //C语言中可采用
    while(chars[i] != '\0') i++;
    count = i;
    //java中可直接使用,对于String同样适用，String内部源码是用的char[]数组存放的值
    count = chars.length();
    ```
  
    

### 5.2.3 块链串

- 链表的每个节点既可以放一个字符也可放多个字符，每个节点称为块，整个链表称为块链结构
- 注意
  - 一般对串操作，从头扫描即可，但对串进行连接操作时，在第一个尾部进行连接，在块链中设置尾（指针，可以以是节点）便于其操作，**最后一个节点**的值一般放置的是无效字符，拼接时需要注意
- 串的存储密度：
  - 存储密度=串值占的存储位 / 实际分配的存储位

