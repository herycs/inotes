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

singletonObjects:用于保存BeanName和bean实例之间的关系，bean name-->bean Instance。

singletonFactories:用于保存BeanName和创建bean的工厂之间的关系,bean name-->objectFactory

earlySingletonObjects:用于保存BeanName和bean实例之间的关系,只保存单例的bean。解决bean的循环依赖引用。

registeredSingletons：用来保存当前所有已注册的bean。

> 循环依赖-----属性注入-----自动注入
>
> Bean生命周期
>
> Bean实例化过程

## Bean 实例化过程	

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



# 常用注解

## Spring

### 类

@Bean--------声明为Bean类

@Component----------声明类为结构类

@Scope("xxx")--------声明类的作用域

@Lazy(true)---------设置类为懒加载

@DependsOn("xxx")-----------依赖于别的类

### 方法

@Lookup----------------被标注的方法将被重写

## SpringMVC

@Controller

- 使用它标记的类就是一个SpringMVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。

@RequestMapping

- RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。
  - value:指定请求的实际地址
  - method： 指定请求的method类型， GET、POST、PUT、DELETE等
  - consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html
  - produces:  指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
  - params： 指定request中必须包含某些参数值是，才让该方法处理
  - headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求

@Resource和@Autowired

- @Resource和@Autowired都是做bean的注入时使用

- @Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入

- 区别：

  @Autowired按照类型（byType）装配依赖对象

  @Resource默认按照ByName自动注入

@PathVariable

- 取出uri模板中的变量作为参数

@RequestParam

@ResponseBody

- 将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区

@CookieValue

## 方法

getBean()----------获取Bean