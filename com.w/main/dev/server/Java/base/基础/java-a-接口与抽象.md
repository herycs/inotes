# Java接口&抽象

## 接口

变量：只能是常量。默认修饰符 **public static final**

方法：只能是抽象方法。默认修饰符 **public abstract**

继承：接口可以继承多个接口

## 抽象

抽象方法不能有主体

抽象类：

用abstract关键字来表达的类，其表达形式为：（public）abstract class 类名{}

特点：

- 抽象方法一定要在抽象类中
- 抽象类和抽象方法必须要被abstract关键字修饰
- 抽象的类是不能被创建对象，因为调用抽象的方法没意义
- 抽象类中的方法要被使用，必须由子类重写抽象类中的方法然后创建子类对象来调用
- 抽象类中可以定义非抽象的方法，有时我们需要此类不能被new关键字创建对象时，可以用abstract将此类变成抽象类
- 子类如果只重写一部份的抽象方法，那么该子类还是一个抽象类,如果抽象类的方法要被使用，子类必须重写抽象类中的所有方法。值得注意：抽象类和普通的类没有太大的不同
- 抽象类无通过new关键字创建对象；
- 抽象类里面可有抽象的方法

