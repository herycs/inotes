# IOC

## 认识

Inversion of Control 控制反转，设计思想，将对象交由容器控制

通过IOC实现解耦，同时也在一定程度上提高了可测试性

定义了IOC容器最基本，最底层的规范

使用&前缀区分用户是想要获取工厂还是对象

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

相关代码

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

相关代码解析

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

### ApplicationContext和Bean

#### 初始化

```java
AbstractApplicationContext-》prepareBeanFactory()//初始化准备工作
    interface aware 接口实现//注入相关BeanName,BeanClassLoader,BeanFactory 注入到Bean
    invokeInitMethods//调用接口
    InitializeBean()-》afterPropertiesSet()//初始化实现
    InvokeCustomInitMethod()-》initMethod()//若Bean有自身的初始化方法，则调用此步进行初始化，否则没有此步
    	--->get initMethod of Bean
    	--->get Object Method by java reflect
```

至此，Bean初始化完成

#### 销毁

```java
doClose()//销毁前的资源关闭
    ---> DisposableBeanAdapter-》destory()//处理
    	--->postProcessBeforeDestruction()
```

### lazy-init和预实例化

```java
finishBeanFactoryInitialization()//封装了对lazy-init属性的处理
    DefaultLisableBean-》preInstantiateSingletons()//对单件Bean完成预实例化，这个过程巧妙的将预实例化委托给了容器如果需要预实例化在这里触发对getBean()的调用
```

如上所述，预实例化和实例化区别就是在容器初始化期间是否触发对getBean()的调用

## BeanFactory实现

```java
getObjectForBeanInstance()//该方法的实现封装了各种实现方式，最终会返回一个作为工厂的FactoryBean生产的产品，而不是FactoryBean本身
    getObject()//通过FactoryBean的getBean()方法获取到Bean对象
```

```java
createBean()
    // 获取bean，若获取对象是prototype类型的还可以为这个Bean生成指定构造函数的对应参数，一定程度上可以控制prototype类型的Bean

containsBean()

isSingleton()

isPrototype()

isTypeMatch()

getType()

getAliases()
```

## BeanPostProcessor实现

这是一个接口类，也可以理解为一个监听器，可以监听容器触发的事件

```java
postProcessBeforeInitialization()//初始化Bean前提供回调入口
postProcessAfterInitialization()//初始化Bean后提供回调入口
参数：1.bean的实例化对象；2.bean的名字
```

与IOC容器的结合

```java
populateBean
    initializeBean()// 依赖：Bean的名字，完成依赖注入后的Bean对象，对这个Bean的BeanDefinition
		---> postProcessBeforeInitialization()
    	---> postProcessAfterInitialization()
```

上述过程中，包括为BeanNameAware设置名字，类型是BeanClassLoaderAware的Bean设置类装载器，类型是BeanFactoryAware的Bean设置自身所在的IOC容器以供回调使用，同样包括对上述方法的调用，以及init-method的处理等，过程结束得到的就是正常使用的由IOC容器托管的Bean

## autowiring的实现

自动注入，在自动注入中不需要对Bean属性做显式的依赖关系声明，只需要配置好autowiring属性，IOC容器会依据这个属性的配置，使用<u>反射</u>自动查找属性的类型或名字，而后基于这个些信息自动配置IOC容器中的Bean，从而自动完成依赖注入

实质上是在populateBean中实现的，在populateBean处理一般Bean之前会处理autoWiring属性，基于设置属性不同调用如下方法

```java
autowireByName()// 反射机制得到当前Bean中需要注入的属性名，使用这个属性名申请与之相同的Bean，这时实际上触发了另一个Bean的生成和依赖注入过程
autowireByType()
```

## Bean的依赖检查

使用Spring的时候，应用非常大，这时的依赖就会非常复杂，一般Bean的依赖注入发生在应用第一次向容器索取Bean的时候发生，这时候不一定会成功，需要重新检查依赖关系的有效性，这会很繁琐，为应对此，IOC容器中设置了依赖性检查特性

使用时应用只需要在Bean定义中设置dependency-check属性来指定依赖检查模式即可，共计：none,simple,object,all四种模式

```java
AbstractAutowireCapableBeanFactory-》createBean()//在这个过程中完成属性检查，不满足要求则抛出异常
```

## Bean对容器的感知

某些情况需要Bean操作IOC容器，而这要求Bean对IOC容器有一定感知，具体通过aware接口来完成