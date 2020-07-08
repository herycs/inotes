# Spring Summary

> 循环依赖-----属性注入-----自动注入
>
> Bean生命周期
>
> Bean实例化过程

# Bean

官网对bean的定义如下

```java
In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
```

简单理解，Bean分为两种

1. 普通Bean，就和在没使用框架情况下创建的一般java对象一样

2. SpringBean，构成应用程序主干并由Spring IoC容器管理的对象称为bean。

    Bean是由Spring IoC容器实例化，组装和以其他方式管理的对象。

## 基本属性

| Property                 | Explained in…                                                |
| :----------------------- | :----------------------------------------------------------- |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](                                     |

## Spring 对Bean处理过程(粗略分析)

> 1. 初始化容器
> 2. 对类和类的信息进行解析存储（包括对类的验证），存入**1级缓存**
> 3. 实例化类（如果需要实例化，则实例化），通过2中获取的信息BeanDefinition（类本身信息，类的注解信息）推断构造方法，进行实例化，初步对象存入工厂，工厂存入**2级缓存**
> 4. 获取对象方法调用，触发工厂方法执行（依据配置，完善Bean的构建，完成自动注入，调用类所有Autowired，调用生命周期回调，完成aop等）完成后存入**3级缓存**
> 5. 至此完整的Bean已经创建完毕
> 6. 使用完毕调用销毁方法

1. 实例化Spring容器
2. 捕获信息：扫描类，解析类（注解信息等）
3. 实例化BeanDefinition（类的配置数据类）
4. 存入Map
5. 调用后置处理器（for 循环，遍历所有后置处理器）
6. 验证类（决定是否需要当前实例化，abstract, singleton, lazy三种情况下不直接初始化）
7. 推断构造方法
8. 依据构造方法实例化对象（反射）
9. 缓存注解信息，解析合并后的BeanDefinition
10. 提前暴露自己工厂
11. 判定是否需要属性注入
12. 完成注入
13. 调用所有Autowried--------------------------------（由两部分各自负责一部分）
14. 调用生命周期回调方法
15. 完成代理----aop
16. put 容器
17. 销毁对象

## 三级缓存

1. singletonObjects----------------单例Bean
2. singletonFactories--------------------工厂集合
3. earlySingletonObjects---------------------对象，此时还不是Bean是Bean的前身对象

### 为什么需要一级缓存（单例池）？

单例对象只会实例化一次

### 为什么二级缓存存工厂？

为了解决循环依赖----使用策略模式+工厂模式让Spring依据程序员的配置生成一个所期望的Bean

如果不使用工厂直接缓存，那么取出的对象就是存入的对象，当进行注入的时候，注入的也是存入的对象，这会存在问题

少了从工厂拿出时的一个判别（例如是否需要对取出对象进行AOP增强等等操作），在Spring的Bean声明周期中，注入时若未完成Bean的增强，那么对象仍然是原对象，哪怕配置了AOP等注解，此时拿到的对象并不会具有Spring所提供的功能

当从工厂拿时，会进行一些判定处理，若当前类的配置信息类的相关配置未完成，则会在完成后再返回Spring修改后的对象（例如，AOP处理完就会是一个代理后的对象，而不是原对象）

### 为什么存三级缓存？

解决性能问题

## 循环依赖

只有单例可以循环依赖，原型不可以循环依赖（Spring容器创建时并不会实例化原型类）

循环依赖关键：AbstractAutowireCapableBeanFactory类中

```java
//如果当前处理类满足，单例，是否支持循环依赖，处于创建状态（通过判定创建的set中是否有当前类决定是否处于创建状态）
//allowCircularReferences----------是否支持循环依赖（源码中默认为true）,可以修改为false
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                  isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
        logger.trace("Eagerly caching bean '" + beanName +
                     "' to allow for resolving potential circular references");
    }
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

# IOC

> 控制反转

## 基础

### 代码基础

在`org.springframework.beans`和`org.springframework.context`包中的内容就是IOC的基础

Spring对象的创建使用的是**BeanFactory**，它提供了配置框架和基本功能

**ApplicationContext**是BeanFactory的子接口，在BeanFactory提供的和框架配合的基础功能上增加了更多的功能支持

## IOC执行基础

IOC依据用户提供的**配置数据**对基础的**POJO类**进行加工最终得到了完整的配置系统

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200707164542468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

`ApplicationContext`是对内维护bean定义以及相互依赖的注册表的高级工厂的接口，对外为用户提供了Bean对象的访问和管理

## 

|      |                                                              |
| :--- | :----------------------------------------------------------- |
|      |                                                              |
|      |                                                              |
|      |                                                              |
|      |                                                              |
|      |                                                              |
|      |                                                              |
|      |                                                              |
|      |                                                              |
|      | https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) |

# AOP

## 通知

- 前置通知
- 后置通知
- 环绕通知
- 返回通知之后
- 抛出通知

## 代理

官方文档指出：Spring AOP默认将标准JDK动态代理用于AOP代理。默认情况下，如果业务对象未实现接口，则使用CGLIB。

## 关于AspectJ

SpringAOP和AspectJ都是----------AOP思想的实现

但SpringAOP的使用语法反人类，，，于是借鉴了AspectJ的语法风格

# 设计模式

## 策略设计模式

BeanPostProcessor后置处理器使用，共计6个实现类，分别执行不同操作

# 其他

## 注解

### 类

@Bean--------声明为Bean类

@Component----------声明类为结构类

@Scope("xxx")--------声明类的作用域

@Lazy(true)---------设置类为懒加载

@DependsOn("xxx")-----------依赖于别的类

### 方法

@Lookup----------------被标注的方法将被重写

## 方法

getBean()----------获取Bean