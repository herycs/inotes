# 创建型模式

## 抽象工厂模式

抽象工厂（工厂实现类）+抽象产品族（产品实现类）

意图：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类

别名：kit

动机：当需要应对变化，而我们本身又并不关心其变化情况下

适用性：

- 系统独立于它的产品的创建、组合和表示时
- 系统要由多个产品系列中的一个来配置时
- 强调一系列产品对象的设计以便进行联合使用时
- 提供一个产品类库，而只想显示他们的接口而不是实现时

结构：

参与者：

- 抽象工厂---》实体工厂
- 抽象产品---》实体产品
- Client(只使用由抽象工厂和抽象产品提供的接口)

协作：

AbstractFactory 将对象创建延迟到 ConcreteFactory 中进行

效果：

优点

1. 分离了具体的类
2. 产品的切换变得容易
3. 有利于产品的一致性

缺点：

1. 难以支持**新种类产品**

实现：

- 工厂单例：一般一个产品族只需要一个抽象工厂负责建立即可，工厂建议是Singleton
- 创建产品：创建交由实体工厂实现，当有多个可能的产品族时，工厂也可以使用Prototype模式实现（具体工厂通过使用原型类创建对象，通过复制已有对象创建新对象，实体工厂向原型池中增加新产品）
- 定义可扩展工厂

## 建造者模式

指导者+建造者抽象类（具体建造者负责建造具体产品）

意图：将复杂对象的构建和表示分离，使得同样的创建过程可以创建不同的表示

动机：

适用：

- 复制对象创建算法应该独立于对象组成部分以及装配方式时
- 构造过程必须允许被构造对象有不同表示时

结构：

参与者：

- Builder：为一个产品对象的各个部件指定抽象接口
- ConcreteBuilder：实现抽象接口以构造该产品的各个部件，定义并明确所创建的表示，提供检索产品的接口
- Director：构造使用Builder接口的对象
- Product：表示被构造的复杂对象，包含定义组成部件的类

协作：

- 客户创建Director并使用Builder进行配置
- 产品部件被生成，导向器通知Builder对象进行配置
- 生成器处理导向器请求，并将部件添加到产品中
- 客户从生成器中检索产品

效果：

1. 改变一个产品的内部表示
2. 将产品的构造和表示代码分开
3. 可更加精细的控制产品创建过程

实现：

- Builder类应该足够普通
- 没有产品抽象类，不同的构建，产品表示相差很大，提供抽象类意义不大

## 工厂方法模式

抽象工厂（工厂实现类）+抽象产品（产品实现类）

意图：定义一个用于创建对象的接口，让子类决定实例化哪一个类。 Factory Method使一个类的实例化延迟到其子类。

别名：虚构造器（Virtual Constructor）

动机：

适用性：

- 当一个类不知道它所必须创建的对象的类的时候。
- 当一个类希望由它的子类来指定它所创建的对象的时候。
- 当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。

参与者：

- Product：定义工厂方法所创建的对象的接口。
- ConcreteProduct： 实现P r o d u c t接口。
- Creator：声明工厂方法，该方法返回一个Product类型的对象。Creator也可以定义一个工厂方法的缺省实现，它返回一个缺省的ConcreteProduct对象。可以调用工厂方法以创建一个Product对象。
- ConcreteCreator：重定义工厂方法以返回一个ConcreteProduct实例

协作：

依赖子类的ConcreteCreator实现

效果：

- 工厂方法不再将与特定应用有关的类绑定到你的代码中。
- 为子类提供挂钩（h o o k） 用工厂方法在一个类的内部创建对象通常比直接创建对象更灵活。
- 连接平行的类层次：工厂方法并不往往只是被C r e a t o r调用，客户可以找到一些有用的工厂方法，尤其在平行类层次的情况下

## 原型模式

意图：

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

适用性

当一个系统应该独立于它的产品创建、构成和表示时，要使用 P r o t o t y p e模式；以及

- 当要实例化的类是在运行时刻指定时，例如，通过动态装载；
- 为了避免创建一个与产品类层次平行的工厂类层次时；
- 当一个类的实例只能有几个不同状态组合中的一种时。

效果：

- 运行时刻增加删除产品
- 改变值以指定新对象
- 改变结构以指定新对象
- 减少子类构造
- 用类动态配置应用

实现：

- 原型管理器：当系统中的原型数目不定时使用注册表维护原型数目
- 实现克隆：原型模式最困难的部分在于克隆，特别是有循环引用的情况下
- 初始化克隆对象：若需要克隆对象的一些属性等置为一些预期值时需要进行一些赋值操作，但是这并不适合写在clone方法中（破坏克隆接口的统一性，有的对象并不需要赋值）

## 单例模式

维护一个类

意图：

一个类仅有一个实例，并提供它的一个全局访问点

动机：

对一些类来说，只有一个实例是很重要的。

适用性：

- 当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
- 当这个唯一实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。

参与者：

单例Singleton

协作：

客户只能通过S i n g l e t o n的I n s t a n c e操作访问一个S i n g l e t o n的实例

效果：

1. 对唯一实例的受控访问，可以严格的控制客户怎样以及何时访问它。
2. 缩小名空间，S i n g l e t o n模式是对全局变量的一种改进。它避免了那些存储唯一实例的全局变量污染名空间。
3. 允许对操作和表示的精化，S i n g l e t o n类可以有子类，而且用这个扩展类的实例来配置一个应用是很容易的。
4.  允许可变数目的实例，允许 S i n g l e t o n类的多个实例。
5. 比类操作更灵活

实现：

1.  保证一个唯一的实例，创建这个实例的操作隐藏在一个类操作
2. 创建S i n g l e t o n类的子类 

## 创建型模式讨论

