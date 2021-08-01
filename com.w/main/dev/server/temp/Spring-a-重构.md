# Spring 生命周期

## 1.不使用Spring

首先，提供一个场景，不借助Spring这类的框架完成Java服务端的开发

### 1）管理类

背景：我们的逻辑并非包括几个类就可以完成，大多数时候需要的是大量类（Jar包中的 + 用户编写的）的协作完成，而这些类若是无状态的则将其暂存起来，以单例的形式维护起来

需求：需要提供全局访问的API，用来获取单例的类实例

技术方案：可以采用容器完成对类实例的暂存，或者提供创建方式（有的实例可以采用懒加载机制，提高应用启动速度和避免不必啊的开销）

### 2）增强类

背景：类的编写除了基本逻辑以外我们希望能在需要的时候增强它，例如希望能获取实时的日志

需求：基于需要选择性增强某些类

技术方案：分析认为Java的代理方案可以完成这个效果，当然Spring框架除此之外还提供了CGLIB的字节码层面增强方式

### 3）如果想成为一个框架还应该具有的特性

- 便捷的操作：方便用户上手
- 具有普适性：能尽可能多的适用于多的场景，服务于更多的用户
- 第三方：便捷的工具&插件用于调试，同时支持对多种第三方提供支持

成为一个优秀的框架

- 强大友好的社区：帮助改进框架和协助用户解决问题
- 轻量级：避免为了使用一块功能而不得不引入大量用不到的内容
- 进化：不断的改进以更好的适应用户的需要
- 多方面追求：提供可靠的支持的基础上提供尽可能高的性能
- 可供学习的设计：设计架构中融入优秀的思想

## 2.Spring怎么做的

针对1中提出的需要，对比Spring中都得到了支持

### 1）IOC(类管理)：Bean->Bean Denefition

**做了什么**：将类的new 过程交由Spring进行管理

**地位**：Spring最关键的技术，个人认为是IOC，是后续AOP包括其他的功能的基石

**核心类**：BeanDenefition - 类模版

**基本属性**

以下是BeanDenefition部分源码，可见Spring中识别类只有两种：

1无状态-单例类

2有状态-原型类

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON; // 单例类 singleton
	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE; // 原型类 prototype

	int ROLE_APPLICATION = 0; // 属于用户的Bean
	int ROLE_SUPPORT = 1; // 属于支持类的Bean
	int ROLE_INFRASTRUCTURE = 2; // 属于框架结构的Bean
}
```

**支持的功能**

1）保存Bean的基本信息

- Class内容（属性方法）
- 来源（class路径）-出问题的时候显示

2）包装信息

- 基本信息：BeanName，Bean工厂
- 怎么维护：原型还是单例
- 使用范围：（用户，支持，框架结构）
- 关联关系：依赖对象

### 2）AOP(类增强): Bean->Proxy+Bean

**做了什么**：将类的增强交给框架，用户编写核心类即可，框架会构造一个代理类提供给用户使用

**核心**：AopProxy

**大致流程**：

```java
AnnotationAwareAspectJAutoProxyCreator
  -> postProcessAfterInitialization() - 这是多个后置处理器中的一个
  	-> createProxy() 
  		-> return ProxyFactory.getProxy()
  			->{return new AopProxy();}
					JdkDynamicAopProxy(目标类实现了某个接口) | ObjenesisCglibAopProxy
```

### 3）优秀的框架

作为一个面向广大开发群体的框架，spring除了基本的管理和增强外还提供了许多功能，以下列出了一些

- 多种配置方式：XML， 注解
- 对第三方框架的适配：Mybatis，Hibernate，MVC框架管理
- 基于事务的注解
- 复杂类型Bean的注解

## 3.生命周期

### 2.1具体启动过程

**基本过程**

1. 容器启动

   --- 设置属性

2. 设置对象属性

3. 调用BeanNameAware的setBeanName()

4. BeanFactoryAware的setBeanFactory()

   --- 执行初始化流程

5. 调用BeanPostProcessor的预初始化方法

6. 调用InitalizaingBean的afterPropertiesSet()方法

7. 调用init-method

8. 调用BeanPostProcessor的后置处理方法

   --- 存储结果

9. Scope = 单例---》缓存到IOC容器中，scope = 原型类---》生命周期交给客户端

   --- 销毁

10. 容器关闭

11. 调用disposableBean的afterPropertiesSet()方法

12. 调用destory-method()方法

**注意点**

目标方法执行前后：将进行初始化（掉init-metho()）或销毁（destory-method()）。

销毁方法：spring在掉销毁方法时，只会销毁scope域（作用域）为singleton的bean，prototype多例的销毁方法需要交给客户端处理。

### 2.2属性填充的地方

- BeanID： 

如果Bean实现了BeanNameAware接口，会回调该接口的setBeanName()方法，传入该Bean的id，此时该Bean就获得了自己在配置文件中的id，

- BeanFactory:

如果Bean实现了BeanFactoryAware接口,会回调该接口的setBeanFactory()方法，传入该Bean的BeanFactory，这样该Bean就获得了自己所在的BeanFactory，

- Context:

如果Bean实现了ApplicationContextAware接口,会回调该接口的setApplicationContext()方法，传入该Bean的ApplicationContext，这样该Bean就获得了自己所在的ApplicationContext，

- PostProcessor:

如果有Bean实现了BeanPostProcessor接口，则会回调该接口的postProcessBeforeInitialzation()方法，

- afterPropertiesSet:

如果Bean实现了InitializingBean接口，则会回调该接口的afterPropertiesSet()方法

### 2.3意义何在

为何创建一个类要这么麻烦？

Bean 管理 不仅仅是new一个全局对象，而且得支持延迟加载，增强（AOP）等过程

个人理解，既然作为框架帮助用户管理Bean，那么需尽可能把有必要的属性以简单的方式提供给用户，同时将复杂的填充逻辑交给框架来完成，一个Bean创建过程中不仅仅是new一个实例这么简单，包括增强，设置值，适配合适的实现类，自动导入实例等等复杂操作，Spring通过将这些过程抽离出来，通过大量的后置处理器协作完成对一个Bean Denefination属性完整的填充

### 2.4值得关注的点

1）设计模式：采用了大量的设计模式，例如：工厂

2）循环依赖：两级缓存解决