# Spring启动过程

## 1.全局过程

宏观层面：run方法执行完成则，Spring的启动就完成了

```java
public class SpringApplication{
    public void run(){
        1.链接VM(基于socket方式)

            2.准备框架启动引导类，框架上下文 - 基本环境
            3.检查配置Java基本环境（AWT） -> {
            if (启动检查) {
                //打印Spring面板
            }
            // 打印错误信息
        }

        4.refreshContext // 构建用户上下文，刷新整体上下文
        {
            配置业务环境 - IOC 注入 - AOP配置
        }

        5.基本业务环境准备完成
            return ;
    }
```

## 2.核心容器构建过程

refreshContext() 方法触发了对标注的Bean的管理过程

主要过程：

1.解析bean标签->BeanDefinitionHolder->注册 (BeanName, BeanDefination)

2.构建实例

**singletonObjects**:用于保存BeanName和bean实例之间的关系，bean name-->bean Instance。

**singletonFactories**:用于保存BeanName和创建bean的工厂之间的关系,bean name-->objectFactory

**earlySingletonObjects**:用于保存BeanName和bean实例之间的关系,只保存单例的bean。解决bean的循环依赖引用。

registeredSingletons：用来保存当前所有已注册的bean。

> 循环依赖-----属性注入-----自动注入
>
> Bean生命周期
>
> Bean实例化过程

## Bean 实例化过程

 加载并解析XML -> 扫描并注册Bean -> 创建Bean

```java
{
  prepareRefresh();
  
  ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

  prepareBeanFactory(beanFactory);
  postProcessBeanFactory(beanFactory);

  invokeBeanFactoryPostProcessors(beanFactory);

  registerBeanPostProcessors(beanFactory);

  initMessageSource();
  initApplicationEventMulticaster();

  onRefresh();

  registerListeners();

  finishBeanFactoryInitialization(beanFactory);
  finishRefresh();
}
```

## XML解析的准备工作

- 使用ResourceLoader将资源文件路径转换为Resource文件。
- 通过DocumentLoader对Resource文件进行转换，将Resource文件转换为Docuemnt文件。
- 通过DefaultBeanDefinitionDocumentReader(模版模式)对Document进行解析,并使用BeanDeifinitionParserDelegate对Element进行解析。

## 解析XML文件

### Import标签解析

- 获取resource属性所表示的路径，递归调用bean的解析过程,触发import-process事件,解析完成。

### alias标签解析

- 获取name属性和alias属性，如果有一个没有填写，跳过解析。
- 注册alias，触发alias-registered事件。

### Bean注册

Bean标签解析的大致流程如下:

- 使用委托类BeanDefinitionParserDelegate的parseBeanDefinitionElement方法来进行元素解析,返回BeanDefinitionHolder类型的bdHolder实例,bdHolder实例包含配置文件中的各种属性。

- 默认标签、自定义标签进行解析。

- 解析完成后,需要对解析后的bdHolder进行注册

  注册操作委托给了BeanDefinitionReaderUtils的registerBeanDefinition方法。

- 通知相关的监听器,这个bean已经加载完成了。

#### 解析BeanDefiniton

- 校验beanName和aliasName是否唯一，如果不是，输出error信息。
- 解析标签
- 如果检测到bean没有指定beanName，那么使用默认规则为此Bean生成BeanName。
- 将获取到的信息封装到BeanDefinitionHolder实例中。

## 注册Bean

- 通过beanName注册BeanDefinition
- 通过别名注册BeanDefinition

## 加载bean

加载bean的过程:

- 转换对应的beanName。
- 尝试从缓存中加载单例。
  - singletonObjects:用于保存BeanName和创建bean实例之间的关系，bean name-->bean Instance。
  - singletonFactories:用于保存BeanName和创建bean的工厂之间的关系,bean name-->objectFactory。
  - earlySingletonObjects:用于保存BeanName和创建bean实例之间的关系,只保存单例的bean。用于解决bean的循环依赖引用。
  - registeredSingletons：用来保存当前所有已注册的bean。

bean的实例化。

- 对factorybean正确性的验证，将从Factory中解析bean的工作委托给getObjectFromFactoryBean。
- 获取单例
- 准备创建bean
  - 根据设置的clas属性或者根据className来解析Class。
  - 对override属性进行标记及验证。
  - 应用初始化的后处理器，解析指定bean是否存在初始化前的短路操作。
  - 创建bean。
  - 对后处理器中的所有InstantiationAwareBeanPostProcessor类型的后置处理器执行postProcessBeforeInstantiation方法和BeanPostProcessor的postProcessAfterInstantiation方法。
  - 实例化前的后处理器应用：bean 实例化之前，就是将AbstractBeanDefinition转换为BeanWrapper前的处理。
  - 实例化后的后置处理器应用：bean的初始化后尽可能保证注册的后处理器的postProcessAfterInitialization方法用到该bean中，因为如果返回的bean不为空，那么便不会再次经历普通bean的创建过程。

- 原型模式的依赖检查。
  - 构造器循环依赖: 无法解决，只能抛出BeanCurrentlyInCreationException异常表示循环异常。
  - setter循环依赖：只能解决单例作用域的bean循环依赖。
  - prototype范围的依赖处理: 对于prototype作用域bean，Spring容器无法完成依赖注入，不进行缓存。
- 检测parentBeanFactory。
- 将GenericBeanDefinition转换为RootBeanDefinition。
- 寻找依赖。
- 针对不同的scope进行bean的创建。
- 类型转换。

### 创建bean

创建bean的过程:

- 如果是单例则需要清除缓存。
- 实例化bean，将BeanDefinition转换为BeanWrapper。
- MergedBeanDefinitionPostProcessor的应用。
- 依赖处理、属性填充。
- 循环依赖检查。
- 初始化Bean(afterPropertiesSet、init-method属性)。
- 注册DisposableBean(destroy-method,DestructionAwareBeanPostProcessor的销毁方法)。
- 完成并返回。

Spring在获取bean的时候，使用map、和同步来记录加载的状态，也可以用来进行循环依赖检测。

使用InstantiationAwareBeanPostProcessor在bean实例化之前进行处理。

使用BeanPostProcessor在bean实例化之后进行处理。
