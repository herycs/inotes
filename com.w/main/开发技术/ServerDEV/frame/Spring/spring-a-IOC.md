# IOC

## 认识

Inversion of Control 控制反转，设计思想，将对象交由容器控制

通过IOC实现解耦，同时也在一定程度上提高了可测试性

## 基本组件

### BeanFactory

基本接口类

### IOC容器接口图

主线1：

BeanFactory

HierarchicalBeanFactory - 追加管理双亲的IOC功能

主线2：

BeanFactory

ListableBeanFactory

ApplicationContext

WebApplicationContext

ConfigurableApplicationContext

## 相关运行历程

### IOC初始化

reflush()

过程包括：BeanDefinition 的 Resource 定位、载入和注册

#### Resource定位

由ResourceLoader通过统一的Resource接口实现

#### 载入

将用户定义的Bean转换为IOC内部的数据结构BeanDefinition

#### 注册

向容器注册BeanDefinition，通过BeanDefinitionRegistry接口实现，实质上是注册到了一个HashMap中去，而这个IOC就是通过这个HashMap持有这些BeanDefinition

### 定位

```java
FileSystemXmlApplicationContext -》reflush()//初始化开始，AbstractApplicationContext中实现

AbstractRefreshableApplicationContext-》refreshBeanFactory()//创建了BeanFactory
    ---> AbstractXmlApplicationContext-》LoadBeanDefinitions()//读取默认初始化器，设置给IOC，启动读取器加载信息
	    ---> XmlBeanDefinitionReader//定位resource
    		---> BeanDefinitionReader//实质读取
```

### 载入

```java
---> BeanDefinitionParserDelegate//依据Bean定义解析读入到的Xml文档树进行解析
    ---> BeanDefinitionDocumentReader-》registerBeanDefinition(doc, resource)//解析，同时对载入Bean数量进行统计
    // 载入分为两部分
    ---> 1.调用Xml解析器得到document对象
    ---> 2.documentReader中实现对Bean的解析
    DefaultBeanDefinitionDocumentReader 创建在后面完成，之后处理 BeanDefinition，处理结果交给BeanDefinitionHolder
	//BeanDefinitionHolder除了持有BeanDefinition对象，同时还持有其他的一些信息，Bean名字，别名集合等
```

到这个步骤，Bean已经转换为Spring可处理的基本数据结构，或者说做好了后期Spring框架进行管理的基本工作

### 注册

```java
DefaultListableBeanFactory//向HashMap注册BeanDefinition,若遇到同名BeanDefinition依据allowBeanDefinitionOverriding配置
```

到此为止，容器中的注册已经完成了，Spring框架已经可以随意操作BeanDefinition了，而依赖注入的基础到此也准备完成了

#### 依赖注入

触发时机：

1. 用户第一次向容器索要Bean
2. BeanDefinition中定义了lazy-init属性让容器完成对Bean的预实例化，这个预实例化实质上也是个依赖注入的过程，但是实在初始化过程中完成的

```java
DefaultListableBeanFactory 基类 AbstractBeanFactory 的 getBean() 中实现

getBean() ---> createBean()
    AbstractAutowireCapableBeanFactory-》createBean()//生成bean,同时对Bean初始化做了处理

    1.createBeanInstacnce()//创建an包含的Java对象
		SimpleInstantiationStrategy//1.JavaUtils JVM 反射功能，2.CGLIB生成
    2.populateBean()//依赖注入
		AbstractAutowrieCapableBeanFactory
    		resolveValueIfNecessary()//包含对所有注入类型的处理
```

至此依赖注入的准备条件基本完备，接下来实际进行注入

```java
BeanWarpper-》setPropertyValues()//注入发生
    BeanWrapperImpl//实际完成注入
```

对于依赖注入部分可以对比Spring 2.0 & Spring 3.0部分的区别

Bean创建和依赖注入过程中，依据BeanDefinition中信息递归完成依赖注入，以getBean()为入口

1. 在上下文中查找需要的Bean和创建Bean的递归调用
2. 完成依赖注入时，通过递归调用容器的getBean方法，得到当前Bean依赖的Bean，同时触发对依赖Bean的创建和注入
3. 对Bean属性注入，解析过程也是递归过程

依据依赖关系，递归进行一层一层的Bean的创建和注入，直到最后完成当前Bean的创建

有了这个顶层Bean的创建以及它的属性依赖的注入完成，当前Bean相关的整个依赖链注入也就完成了

## 其他特性设计

### ApplicationContext和Bean的初始化和销毁

```java
AbstractApplicationContext-》prepareBeanFactory()//初始化准备工作
    initializeBean()//初始化
doClose()//销毁前的资源关闭
```



## 相关技术点

## Bean



## BeanFactory

> DefaultListableBeanFactory
>
> 包含所有IOC重要功能

## BeanFactory

定义了IOC容器最基本，最底层的规范

使用&前缀区分用户是想要获取工厂还是对象

### 方法

getBean()

获取bean，若获取对象是prototype类型的还可以为这个Bean生成指定构造函数的对应参数，一定程度上可以控制prototype类型的Bean

containsBean()

isSingleton()

isPrototype()

isTypeMatch()

getType()

getAliases()

## IOC初始化过程

过程：BeanDefinition定位，载入和注册

### Resource定位

ResourceLoader通过接口Resource完成

代码

以FileSystemXmlApplicationContext为例

```java
//AbstractXmlApplicationContext继承而来的ResourceLoader以及BeanDefinition的定义的能力
public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {
    //...
	public FileSystemXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();//通过容器初始化的Refresh启动整个调用
		}
	}
    //用于读取Resource服务
    //在BeanDefinitionReader的loadBeanDefinition中采用模板方法调用
    protected Resource getResourceByPath(String path) {
		if (path.startsWith("/")) {
			path = path.substring(1);
		}
		return new FileSystemResource(path);
	}
}
```

载入过程的启动

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {
	//...
    protected final void refreshBeanFactory() throws BeansException {
        //若已有BeanFactory则销毁，并关闭
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
            //创建并设置所有持有DefaultListableBeanFactory的地方同时调用loadBeanDefinitions，再次载入BeanDefinition
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			loadBeanDefinitions(beanFactory);
			this.beanFactory = beanFactory;
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
    //上下文中创建：DefaultListableBeanFactory
    //getInternalParentBeanFactory实现参看AbstractApplicationContContext
    protected DefaultListableBeanFactory createBeanFactory() {
		return new DefaultListableBeanFactory(getInternalParentBeanFactory());
	}
}
```

### 载入

构建BeanDefinition（POJO对象在IOC容器中的抽象）

过程



### 注册

向IOC容器注册BeanDefinition

## IOC初始化&依赖注入

Spring中的Bean定义的初始化和Bean的依赖注入是两个过程

一般IOC的初始化不包括Bean的依赖注入，依赖注入发生于第一次获取Bean时

lazyinit属性定义后，Bean会在IOC完成初始化之前完成依赖注入