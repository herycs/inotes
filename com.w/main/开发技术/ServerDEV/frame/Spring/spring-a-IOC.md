# IOC

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