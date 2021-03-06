# java基本信息

# 面向对象

## 与面向过程

- 面向对象：问题分解为基础步骤，每一步由函数实现，依次调用完成任务
- 面向过程：问题分解为基础步骤，将基础步骤抽象化交由一个个对象完成，各个对象的组合完成任务
- 区别：
    - 面向对象：第三人称，统筹计划，职责内化
    - 面向过程：第一人称，预测，按部就班
- 二者对比：
    - 面向对象占用资源高，速度慢
    - 面向过程占用资源少，速度快

## 三大特征

继承

> 一个类（继承关系中子类或派生类）继承了另外一个类（继承关系中成为基类或父类）

效果：子类具备父类不曾隐藏起来的可被继承的所有能力

实现方式：
- 继承：继承类，直接拥有继承到的父类的技能
- 组合：实现接口，受接口的约束，可在继承的方法体中自定义自己的处理策略

内存布局
- 对象体只保存对象参数+虚拟函数表（Vtable）指针
- Vtable(虚拟函数表)为方法的指针数组

封装

- 情景：一个类将当前具有的特性进行一定程度保护，但其他类可采用当前类允许的方式执行一定操作
- 外部可使用类提供的方法多态

多态

同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果

必要条件：
- 有类继承或接口实现
- 子类要重写父类的方法
- 父类的引用指针子类的对象（就是用父类对象接受子类对象）

## 指导思想

可维护性

可复用性

可扩展性

灵活性

# Java中的有关概念

## 重载与重写

重载

就是函数或者方法有同样的名称，但是参数列表不相同的样子

- 条件：方法签名必须有所差异（参数列表，返回类型，权限修饰符，异常）

重写

子类方法覆盖父类方法

重写条件：
- 参数列表一样
- 返回类型一样
- 访问级别<=被重写访问等级
- 不抛出新的检查异常，或抛出范围大于父类的异常

不可重写情况
- final标识的方法不可被重写
- 若不能继承此方法，则不可重写此方法

## 继承与实现

继承：多个类具有共同的部分，抽取出抽象类

实现：多个类处理方式不同，但目标相同，提取接口

## 继承与组合

### 组合

优点：破坏封装，松耦合，相对独立，扩展性好，动态组合，整体类可对局部类接口进行封装

缺点：
- 整体类不可自动获得局部类的接口
- 创建整体类对象时需创建所有局部类对象

### 继承

缺点：破坏封装，紧耦合，独立性差，支持扩展（同时增加了复杂度），不支持动态继承（运行时子类不可选择不同父类），子类不可改变父类接口

优点：
- 子类可自动继承父类接口
- 创建对象无需创建父类对象


建议：同等情况下，优先采用组合

## 构造函数（构造器）

- 构造器没有返回类型，不会被继承，并且可以有范围修饰符。
- 构造器的函数名称必须和它所属的类的名称相同

## 变量

#### 类变量、成员变量和局部变量

| 变量类型 | 类变量      | 成员变量 | 局部变量 |
| -------- | ----------- | -------- | -------- |
| 内存位置 | JVM的方法区 | 堆内存   | 栈内存   |

#### 成员变量和方法作用域

- `public` ：该成员变量或方法是对所有类或对象都是可见的，所有类或者对象都可以直接访问
- `protected` ：某些成员变量或方法对类本身，与同在一个包中的其他类可见，其他包下的类不可访问，除非是他的子类
- `default` ：该成员变量或方法只有自己和其位于同一个包的内可见，其他包内的类不能访问，甚至是它的子类
- `private` ：该成员变量或方法是专有的，只有当前类可以提供访问权限，其他其他或者对象都没有访问权限。子类也没有访问权限。

## 值传递

值传递、引用传递

为什么说Java中只有值传递

- 编程语言中需要进行方法间的参数传递，这个传递的策略称为求值策略
- 值传递和引用传递最大的区别是传递的过程中有没有复制出一个副本来，如果是传递副本，那就是值传递，否则就是引用传递。
- Java实际上是通过**值传递**实现的参数传递，只不过对于Java对象的传递，传递的内容是对象的引用

## 平台无关性

> 基础：JVM的平台有关性

java文件被编译为class文件，JVM对符合class规范的文件都能识别，具体执行Java将不同环境差异屏蔽掉，给程序提供一个类似在编写环境下未变的运行

### 其他支持

Java语言规范：通过规定的Java语言中基本数据类型的取值范围和行为

类文件：所有Java文件要编译成统一的Class文件

Java虚拟机：通过Java虚拟机将Class文件转成对应平台的二进制文件等

## 语言无关性

- 实质上Java和JVM虚拟机并非是绑定的，除规范的Java文件编译后的class文件外，JVM认识所有符合JVM字节码规范的文件

# Java中部分关键字

### transient：序列化时所修饰域不参与

### instanceof：两个对象之间是否是继承关系

### volatile：多线程环境下变量的变动在多个线程间可见

### synchronized：同步修饰，对目标对象加锁

### final：所修饰引用不可修改

### static：静态修饰，内存环境中只存在一份
