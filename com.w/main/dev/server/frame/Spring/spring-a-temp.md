# Spring Summary

## Bean生命周期

1. 容器启动
2. 设置对象属性
3. 调用BeanNameAware的setBeanName()
4. BeanFactoryAware的setBeanFactory()
5. 调用BeanPostProcessor的预初始化方法
6. 调用InitalizaingBean的afterPropertiesSet()方法
7. 调用init-method
8. 调用BeanPostProcessor的后置处理方法
9. Scope = 单例---》缓存到IOC容器中，scope = 原型类---》生命周期交给客户端
10. 容器关闭
11. 调用disposableBean的afterPropertiesSet()方法
12. 调用destory-method()方法

> 循环依赖-----属性注入-----自动注入
>
> Bean生命周期
>
> Bean实例化过程

 	**上下文刷新refresh工作流程如下**

代码流程：{
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

## XML解析的准备工作

- 使用ResourceLoader将资源文件路径转换为Resource文件。
    - 封装资源文件。使用EncodedResource(resource、charset、encoding)类对参数Resource进行封装。
        - 考虑Resource可能存在编码要求的问题
        - 使用sax读取xml的方式来转呗InputSource对象。
    - 获取输入流。从Resource中获取对应的InputStream并改造InputSource。
    - 通过构造的InputSource实例和Resource实例继续调用函数doLoadBeanDefinitions。
- 通过DocumentLoader对Resource文件进行转换，将Resource文件转换为Docuemnt文件。
    - 获取XML文件的验证模式(DTD、XSD)。
        - DTD:一种XML约束模式语言,是XML文件的验证机制,可以验证文件是否符合规范、元素和标签使用是否正确。
        - XSD:XML Schema,验证XML文档检查XML文档是否符合要求。需要声明命名空间和命名空间存放的位置(schemaLocation)。
             schemaLocation包含命名空间URI和schema文件位置或者URL地址。
        - 通过判断是否包含DOCTYPE来判断验证模式，如果包含DOCTYPE，就是DTD，否则是XSD。
        - 使用BeansDtdResolver来解析DTD文件中的实体(从Spring class path中加载),直接获取systemId最后的xx.dtd，然后在当前路径下寻找。
        - 使用PluggableSchemaResolver默认获取META-INF/spring.schemas文件(properties文件)中找到systemid所对应的XSD文件并加载。
    - 加载XML文件,并获取对应的Document文件。
        - 使用单一指责的原则。
    - 根据返回Document注册Bean信息。
- 通过DefaultBeanDefinitionDocumentReader(模版模式)对Document进行解析,并使用BeanDeifinitionParserDelegate对Element进行解析。
    - 创建DefaultBeanDefinitionDocumentReader,调用registerBeanDefinitions方法
    - 创建BeanDefinitionParserDelegate,初始化默认的:lazy-init, autowire, dependency check settings,init-method, destroy-method and merge settings,触发default-registered事件。
    - 判断是否为默认的命名空间，如果是默认的命名空间，判断是否设置相应的profile。
    - 对相应的标签进行解析(parseBeanDefinitions->parseDefaultElement)，例如import、alias、bean、内置的beans，可以自定义在解析bean之前(preProcessXml)和之后(postProcessXml)的操作,，如果设置了自定义元素，需要走parseCustomElement方法。

## 解析XML文件

### Import标签解析

- 获取resource属性所表示的路径，如果没有路径，跳出解析;
- 解析路径中的系统属性；
- 判断location是绝对路径还是相对路径;
- 如果是绝对路径则递归调用bean的解析过程,进行另一次的解析;
- 如果是相对路径则计算出绝对路径进行解析;
- 触发import-process事件,解析完成。

### alias标签解析

- 获取name属性和alias属性，如果有一个没有填写，跳过解析。
- 注册alias，触发alias-registered事件。

### Bean标签解析

Bean标签解析的大致流程如下:

- 使用委托类BeanDefinitionParserDelegate的parseBeanDefinitionElement方法来进行元素解析,返回BeanDefinitionHolder类型的bdHolder实例,bdHolder实例包含配置文件中的各种属性。
- 返回的bdHolder实例不为空的情况下，如果存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析。
- 解析完成后,需要对解析后的bdHolder进行注册,注册操作委托给了BeanDefinitionReaderUtils的registerBeanDefinition方法。
- 通知相关的监听器,这个bean已经加载完成了。

#### 解析BeanDefiniton

- 获取id、name属性，如果名字的长度比较长，然后按照',;'进行切分,放到集合中。
- 如果id属性为空以及name属性不为空,会输出没有具体id的log信息。并且将id当做beanName。
- 校验beanName和aliasName是否唯一，如果不是，输出error信息。
- 解析除了id,name之外的其他所有属性,并统一封装至GenericBeanDefinition类型的实例中(先通过class、parent属性创建GenericBeanDefinition对象实例)。
    - 解析class、parent属性，并且创建出来GenericBeanDefinition对象实例。AbstractBeanDefinition是一个抽象类，实现BeanDefinition,
         有具体实现类:RootBeanDefinition(没有父类<bean>的<bean>)、GenericBeanDefinition(一站式的BeanDefinition装配)、ChildBeanDefinition(子类bean)。
         Spring 2.5之后，使用GenericBeanDefinition。BeanDefinitionRegistry是Spring配置信息的内存数据库,主要是以Map的形式保存。
    - 解析Bean属性:singleton、scope、abstract、lazy-init、autowire、depend-on、autowire-candidate、primary、init-method、destory-method、factory-method、factory-bean。
        - 在2.x全部使用scope,如果不指定，使用父类的scope范围。scope和singleton两个属性，指定指定一个。
        - 使用kv构造BeanMetadataAttribute对象实例。
        - 在需要的时候，获取一个新的bean,有两种方式:实现ApplicationContextAware接口;使用lookup-method标签。lookup标签可以动态的替换返回实体bean。
        - @Lookup注解的作用是，当我当用一个方法时，Spring会返回一个方法返回值类型的实例。每次都会创建一个新的对象。
        - @Lookup注解的应用场景:1.向单例bean注入一个prototype-scope的bean；2.使用程序方式注入依赖项。
        - @Lookup注解的缺点:1.使用@Lookup注解的是必须是实体类不能是抽象类，因为扫描会跳过抽象类；2.如果使用@Bean-manage(工厂类不行)时，@Lookup注解不生效。子类不能是final的
        - lookup注解在使用的过程中，需要有一个类来创建父类对象，在创建新的子类，要修改配置文件<lookup>的bean属性。
        - replace-method标签: 在运行时用新的方法替换现有的方法。与look-up不同的是，replace-method不但可以动态的替换返回实体bean，而且还能动态地更换原有方法的逻辑。
        - constructor-arg标签:获取index、type、name属性。
            - 有index属性: 解析constructor-arg的子元素;使用ConstructorArgumentValues.ValueHolder来封装解析出来的元素，并添加到BeanDefinition的constructArgumentsValues的indexedArgumentValues属性中。
            - 没有index属性：解析constructor-arg的子元素;使用ConstructorArgumentValues.ValueHolder来封装解析出来的元素，并添加到BeanDefinition的constructArgumentsValues的genericArgumentValues属性中。
            - 跳过meta和description的解析;constructor-arg标签中，只能存在ref/value中的一个，并且不能有资源;使用RuntimeBeanReference封装ref属性；使用TypeStringValue封装value属性。
    - 解析description属性。
    - 解析meta标签，即meta标签，数据是键值对形式，并且生成BeanMetadataAttribute对象实例。
    - 解析lookup-method标签，然后解析name和bean属性来生成lookupoverride对象实例。
    - 解析replace-method标签，需要指定要替换的name和replacer属性、参数类型属性，生成ReplaceOverride对象实例。
    - 解析construct-arg标签。
    - 解析property标签
        - 必须要有一个name属性
    - 解析qualifier标签:指定注入bean的名称。
- 如果检测到bean没有指定beanName，那么使用默认规则为此Bean生成BeanName。
- 将获取到的信息封装到BeanDefinitionHolder实例中。

## 注册Bean

- 通过beanName注册BeanDefinition
    - 对AbstractBeanDefinition的校验。主要是校验methodOverrides属性。
    - 对beanName已经注册情况的处理。不存在，抛出异常；存在，覆盖。
    - 加入map缓存。
    - 清除解析之前留下的对应的beanName的缓存。
- 通过别名注册BeanDefinition
    - alias与beanName相同情况处理。删除alias。
    - alias覆盖处理。
    - alias循环检查。
    - 注册alias

## 加载bean

加载bean的过程:

- 转换对应的beanName。
    - 去除FactoryBean的修饰符
    - 去指定alias表示的最终beanName。
- 尝试从缓存中加载单例。
    - singletonObjects:用于保存BeanName和创建bean实例之间的关系，bean name-->bean Instance。
    - singletonFactories:用于保存BeanName和创建bean的工厂之间的关系,bean name-->objectFactory。
    - earlySingletonObjects:用于保存BeanName和创建bean实例之间的关系,只保存单例的bean。用于解决bean的循环依赖引用。
    - registeredSingletons：用来保存当前所有已注册的bean。
- bean的实例化。
    - 对factorybean正确性的验证
    - 对非factorybean不做任何处理。
    - 对bean 进行转换。
    - 将从Factory中解析bean的工作委托给getObjectFromFactoryBean。
    - 获取单例
        - 检查缓存是否已经加载过
        - 若没有加载，记录beanName的正在加载状态。
        - 加载单例之前记录加载状态。
        - 通过调用参数传入的objectFactory的个体object方法实例话bean
        - 加载单例后的处理方法调用
        - 将结果记录至缓存并删除加载bean过程中记录的各种辅助状态。
        - 返回处理结果。
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
    - 如果存在工厂方法，使用工厂方式初始化(存在factoryMethodName属性，或者配置文件中的factory-method)。
    - 一个类有多个构造函数，根据参数锁定构造函数并进行初始化(用缓存机制)。
        - 构造函数参数的确定: 根据explicitArgs参数判断(如果不为空，直接确定参数);缓存中获取;配置文件获取。
        - 构造函数的确定:匹配的方法就是根据参数个数匹配，将public构造函数优先参数数量降序、非private构造函数参数数量降序。
        - 获取参数名称的方式：注解方式直接获取；使用Spring中的ParameterNameDiscover来获取。
        - 根据确定的构造函数转换为对应的参数类型。
        - 构造函数不确定行的验证。
        - 根据实例化策略得到的构造函数及构造函数参数实例化bean(动态代理)。
    - 如果没有工厂方法和不带参数的构造函数，使用默认的构造函数进行bean的实例话。
- MergedBeanDefinitionPostProcessor的应用。
- 依赖处理。
- 属性填充。
    - InstantiationAwareBeanPostProcessor处理器的postProcessAfterInstantiation函数的应用，此函数可以控制程序是否继续进行属性填充。
    - 根据注入类型，提取依赖的bean，并统一存入PropertyValues中。
    - 应用InstantiationAwareBeanPostProcessor的postProcessProertyValues方法，对属性获取完毕填充前对属性的再次处理，典型应用是RequiredAnnotationBeanPostProcessor类中对属性进行校验。
    - 将所有PropertyValues中的属性填充只BeanWrapper中。
- 循环依赖检查。
- 初始化Bean(afterPropertiesSet、init-method属性)。
- 注册DisposableBean(destroy-method,DestructionAwareBeanPostProcessor的销毁方法)。
- 完成并返回。
     创建bean使用的方式：反射和cglib代理。

Spring在获取bean的时候，使用map、和同步来记录加载的状态，也可以用来进行循环依赖检测。

使用InstantiationAwareBeanPostProcessor在bean实例化之前进行处理。

使用BeanPostProcessor在bean实例化之后进行处理。
