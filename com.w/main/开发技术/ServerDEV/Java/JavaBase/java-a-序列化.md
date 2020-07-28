# 序列化及反序列化

## 概念

### 序列化

简单理解就是，操作对象保留其数据、状态等信息的情况下，将其转换为**可存储**或**传递**的状态。

### 反序列化

以文件或者传流中的序列化数据为基础，在内存中恢复出原对象。

## 意义

> [序列化原因](https://www.cnblogs.com/yangming1996/p/9318955.html)

1. 大对象的序列化释放资源

    大对象占用内存，但短期使用不到，使用序列化就可以将其暂时的存储到磁盘空间，当使用的时候再回复到内存中即可，显然这种方式将不必要占用资源释放出来用于更多的操作。

2. 支持远程调用

    本来数据存储于本地内存中，远程的运行环境无法接触到此类，但序列化之后将其以网络的方式传输便可将类传递到远程的其他环境中。

3. 数据传输

    序列化后可以将数据提供给远程的机器，这样在远程回复出对象后或者以一定方式对数据进行解析便可得出想要的数据。

## 实现

### 问题处理

提供序列化ID

- 进行一个反序列化时的校验，唯有类型ID和序列化的数据流中的ID一直才允许序列化（建议提供序列化ID，否则虚拟机将依据一定算法自动生成一个ID，这个ID在对象发生变动后便会改变）

序列化对象引用问题

- 序列化时会将其引用对象也序列化，当一个类存在多个引用时只序列化一次，后续序列化直接指向已存在的序列话对象

### 认识

> [参考博客](https://blog.csdn.net/JianLeiXing/article/details/84525767?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
>
> 网络传输有一定风险，序列化只采用默认机制，那么一旦数据流被拦截就相当于将数据明文展示给对方了

序列化本质是序列化三部分：

> 注意此处未提到方法

- 序列化头信息部分
- 类的描述部分
- 属性域的值

### 策略

Java提供的序列化策略共计三种：

1. 采用默认机制 + 自定义机制
2. 完全自定义序列化属性值的方法，可选择参与序列化的数据，可存储其他非this对象包含的数据。
3. 使用自定义的序列化类

### 代码示例

> 以LinkedList为例

ID

```java
private static final long serialVersionUID = 876323262645176354L;
```

序列化

- 调用序列化方法

    ```java
    //序列化，以下代码为依次调用
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
    ```

- 获取序列化必要参数

    ```java
    //序列化头部和字段，获取参数，并调用具体的序列化方法
    public void defaultWriteObject() throws IOException {
        SerialCallbackContext ctx = curContext;
        if (ctx == null) {
            throw new NotActiveException("not in call to writeObject");
        }
        Object curObj = ctx.getObj();
        ObjectStreamClass curDesc = ctx.getDesc();
        bout.setBlockDataMode(false);
        defaultWriteFields(curObj, curDesc);
        bout.setBlockDataMode(true);
    }
    ```
    
- 执行序列化

    ```java
    //字段序列化，执行具体的序列化
    private void defaultWriteFields(Object obj, ObjectStreamClass desc) throws IOException{
        //获取类描述
        Class<?> cl = desc.forClass();
        if (cl != null && obj != null && !cl.isInstance(obj)) {
            throw new ClassCastException();
        }
    	//校验默认序列化策略是否允许
        desc.checkDefaultSerialize();
        //到达此处，未抛出异常，则允许使用默认序列化策略
        /** 
          * { 
          *     返回字段源码解释：aggregate marshalled size of primitive fields
          *		private int primDataSize;
          *	}
          * 方法作用：Returns aggregate size (in bytes) of marshalled primitive field values for represented class.
          *
          */
        int primDataSize = desc.getPrimDataSize();
        /** 
          * buffer for writing primitive field values
          * private byte[] primVals;
          */
        if (primVals == null || primVals.length < primDataSize) {
            primVals = new byte[primDataSize];
        }
        desc.getPrimFieldValues(obj, primVals);
        bout.write(primVals, 0, primDataSize, false);
    	/**
    	  * 	ObjectStreamField[] getFields(boolean copy) {
    	  *          return copy ? fields.clone() : fields;
    	  *     }
    	  * 	false:获取所有许可序列化的字段（fidlds类型为：ObjectStreamField[]）
    	  * 	true:返回当前类的字段拷贝
    	  */
        ObjectStreamField[] fields = desc.getFields(false);//只序列化许可序列化的字段
        Object[] objVals = new Object[desc.getNumObjFields()];//以非私有序列化字段个数为长度申请Object[]
        int numPrimFields = fields.length - objVals.length;
        //获取字段值，存储于objVals处
        desc.getObjFieldValues(obj, objVals);
        for (int i = 0; i < objVals.length; i++) {
            //遍历字段值
            if (extendedDebugInfo) {
                debugInfoStack.push(
                    "field (class \"" + desc.getName() + "\", name: \"" +
                    fields[numPrimFields + i].getName() + "\", type: \"" +
                    fields[numPrimFields + i].getType() + "\")");
            }
            try {
                writeObject0(objVals[i],
                             fields[numPrimFields + i].isUnshared());
            } finally {
                if (extendedDebugInfo) {
                    debugInfoStack.pop();
                }
            }
        }
    }
    ```

- 默认序列化校验

    ```java
    void checkDefaultSerialize() throws InvalidClassException {
        requireInitialized();
        //若不被允许使用默认序列化，则抛出异常
        if (defaultSerializeEx != null) {
            throw defaultSerializeEx.newInvalidClassException();
        }
    }
    //校验是否初始化
    private final void requireInitialized() {
        if (!initialized)
            throw new InternalError("Unexpected call when not initialized");
    }
    ```

总之以上步骤完成后，所有允许序列化的字段都会被序列化到目标文件，而这种默认序列化策略正是最基本的，也就是说任何一个拥有此版本的JDK的序列化操作都是一样的，这样任何一个拿到对应序列化流反向序列化就能得到其中所有数据。而这也正照应了前面提到的网络传输情况下的安全问题。

反序列化

> 流程与序列化类似，不做过多赘述

```java
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

## 技术支持

> 此部分转载自[博客](https://hollischuang.github.io/toBeTopJavaer/#/)

### 接口

#### Serializable接口

> 对一个对象进行序列化，若目标对象不支持Serializable将抛`NotSerializableException`。
>
> 如果要序列化的类有父类，要想同时将在父类中定义过的变量持久化下来，那么父类也应该集成`java.io.Serializable`接口

- 序列化接口没有方法或字段，仅用于标识可序列化的语义。
- 类通过实现 `java.io.Serializable` 接口以启用其序列化功能。

#### Externalizable接口

> 类中没有**无参**数的构造函数，在运行时会抛出异常：`java.io.InvalidClassException`

- Externalizable继承了Serializable，该接口中定义了两个抽象方法：`writeExternal()`与`readExternal()`。
- 当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员**重写**`writeExternal()`与`readExternal()`方法。
- 在使用Externalizable进行序列化的时候，在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。
- 注意：
    - 实现Externalizable接口的类必须要提供一个public的无参的构造器。

#### 区别

- 对⼀个对象进⾏序列化的时候， 如果遇到不⽀持`Serializable` 接口的对象。 在此情况下， 将抛`NotSerializableException`。
- 如果要序列化的类有⽗类， 要想同时将在⽗类中定义过的变量持久化下来， 那么⽗类也应该集成`java.io.Serializable`接口。
- Externalizable`继承了`Serializable
    - 该接⼜中定义了两个抽象⽅法：`writeExternal()`与`readExternal()`。
    - 当使⽤`Externalizable`接口来进⾏序列化与反序列化的时候需要重写writeExternal()与readExternal()⽅法。
        - 如果没有在这两个⽅法中定义序列化实现细节， 那么序列化之后， 对象内容为空。
        - 实现`Externalizable`接⼜的类必须要提供⼀个`public`的⽆参的构造器。

结论：实现`Externalizable`， 并实现`writeExternal()`和`readExternal()`⽅法可以指定序列化哪些属性。

### serialVersionUID

序列化ID

- 虚拟机是否允许反序列化， 不仅取决于类路径和功能代码是否⼀致， ⼀个⾮常重要的⼀点是两个类的序列化 ID 是否⼀致
- JVM会把传来的字节流中的`serialVersionUID`与本地相应实体类的`serialVersionUID`进⾏⽐较
    -  如果相同就认为是⼀致的， 可以进⾏反序列化
    -  否则就会出现序列化版本不⼀致的异常， 即是`InvalidCastException`。
- 当实现`java.io.Serializable`接口的类**没有**显式地定义⼀个`serialVersionUID`变量时候， Java序列化机制会根据编译的Class⾃动⽣成⼀个`serialVersionUID`作序列化版本⽐较⽤
    - 如果Class⽂件没有发⽣变化， 就算再编译多 次， serialVersionUID也不会变化的。
    - 如果发⽣了变化，那么这个⽂件对应的`serialVersionUID`也就会发⽣变化。
- 基于以上情况，序列化后更改类，而后反序列化会报错。

#### 为什么serialVersionUID不能随便改

没有实现这个接口，想要被序列化的话，就会抛出`java.io.NotSerializableException`异常。

进行序列化操作时，会判断要被序列化的类是否是`Enum`、`Array`和`Serializable`类型，如果都不是则直接抛出`NotSerializableException`。

序列化策略：

- 在序列化过程中，如果被序列化的类中定义了`writeObject` 和 `readObject` 方法，虚拟机会试图调用对象类里的 `writeObject` 和 `readObject` 方法，进行用户自定义的序列化和反序列化。
- 如果没有这样的方法，则默认调用是 `ObjectOutputStream` 的 `defaultWriteObject` 方法以及 `ObjectInputStream` 的 `defaultReadObject` 方法。
- 用户自定义的 `writeObject` 和 `readObject` 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。
    - 对于一些特殊字段需要定义序列化的策略的时候，可以考虑使用transient修饰，并自己重写`writeObject` 和 `readObject` 方法，如`java.util.ArrayList`中就有这样的实现。

底层实现：

- 对`serialVersionUID`做了比较，如果发现不相等，则直接抛出异常。

    看一下`getSerialVersionUID`方法：

    ```java
    public long getSerialVersionUID() {
        if (suid == null) {
            suid = AccessController.doPrivileged(
                new PrivilegedAction<Long>() {
                    public Long run() {
                        return computeDefaultSUID(cl);
                    }
                }
            );
        }
        return suid.longValue();
    }
    ```

- 在没有定义`serialVersionUID`的时候，会调用`computeDefaultSUID` 方法，生成一个默认的`serialVersionUID`

### transient

`transient` 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以**阻止该变量被序列**化到文件中

在被反序列化后，`transient` 变量的值被设为初始值。

### 实质

对象序列化保存的是对象的"状态"，即它的成员变量。由此可知，**对象序列化不会关注类中的静态变量**。

## 实际应用

#### ArrayList序列化

ArrayList中elementData是`transient`的，本不能被反序列化出来，但实际结果并非如此

原因是其提供了方法：

- ArrayList中定义了两个方法： `writeObject`和`readObject`。
- 序列化的过程是：
    - 在序列化过程中，如果被序列化的类中定义了writeObject 和 readObject 方法，虚拟机会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化。
    - 如果没有这样的方法，则默认调用是 ObjectOutputStream 的 defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject 方法。
    - 用户自定义的 writeObject 和 readObject 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。


#### 为了避免elementData无效数据被序列化

- ArrayList是动态增长的数组，使用transient修饰数据域，在序列化时不全盘序列化而是采用序列化指定范围数据以避免序列化无用数据。

#### 为了保存elementData中有效数据

- `writeObject`方法把`elementData`数组中的元素遍历的保存到输出流（ObjectOutputStream）中。
- `readObject`方法从输入流（ObjectInputStream）中读出对象并保存赋值到`elementData`数组中。

对象的序列化过程通过ObjectOutputStream和ObjectInputputStream来实现的

- ObjectOutputStream的writeObject的调用栈：

    ```less
    writeObject ---> writeObject0 --->writeOrdinaryObject--->writeSerialData--->invokeWriteObject
    ```

一个类中包含writeObject 和 readObject 方法调用方式

- 在使用ObjectOutputStream的writeObject方法和ObjectInputStream的readObject方法时，会通过反射的方式调用。

- ObjectOutputStream的writeObject的调用栈：

    ```less
    writeObject ---> writeObject0 --->writeOrdinaryObject--->writeSerialData--->invokeWriteObject
    ```

## 破坏单例模式？

- 普通的Java类的反序列化过程中，会通过反射调用类的默认构造函数来初始化对象。所以，即使单例中构造函数是私有的，也会被反射给破坏掉。由于反序列化后的对象是重新new出来的，所以这就破坏了单例。

## 总结

1、在Java中，只要一个类实现了`java.io.Serializable`接口，那么它就可以被序列化。

2、通过`ObjectOutputStream`和`ObjectInputStream`对对象进行序列化及反序列化

3、虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 `private static final long serialVersionUID`）

4、序列化并**不保存静态变量**。

5、要想将父类对象也序列化，就需要让父类也实现`Serializable` 接口。

6、Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文

中，在被反序列化后，transient 变量的值被设为初始值

7、服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。