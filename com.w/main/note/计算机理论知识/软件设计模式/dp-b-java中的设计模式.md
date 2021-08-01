# Java中的DP

#### **创作模式**

**抽象工厂模式** （通过创造性的方法来识别工厂本身，这又可以用于创建另一个抽象/接口类型）

- javax.xml.parsers.DocumentBuilderFactory#newInstance()
- javax.xml.transform.TransformerFactory#newInstance()
- javax.xml.xpath.XPathFactory#newInstance()

**建造者模式** （通过创建方法识别返回实例本身）

- java.lang.StringBuilder#append() （非线程安全）
- java.lang.StringBuffer#append() （线程安全）
- java.nio.ByteBuffer#put()（还CharBuffer，ShortBuffer，IntBuffer，LongBuffer, FloatBuffer和DoubleBuffer）
- javax.swing.GroupLayout.Group#addComponent()
- 所有的实现 java.lang.Appendable

**工厂模式** （可通过创建方法识别返回抽象/接口类型的实现）

- java.util.Calendar#getInstance()
- java.util.ResourceBundle#getBundle()
- java.text.NumberFormat#getInstance()
- java.nio.charset.Charset#forName()
- java.net.URLStreamHandlerFactory#createURLStreamHandler(String) （每个协议返回单例对象）
- java.util.EnumSet#of()
- javax.xml.bind.JAXBContext#createMarshaller() 和其他类似的方法

**原型模式** （通过创建方法识别，返回具有相同属性的其他实例）

- java.lang.Object#clone()（班必须实施java.lang.Cloneable）

**单例模式**（通过创造性方法识别，每次返回相同的实例（通常是自己））

- java.lang.Runtime#getRuntime()
- java.awt.Desktop#getDesktop()
- java.lang.System#getSecurityManager()

#### **结构模式**

**适配器模式** （可通过创建方法识别采用不同抽象/接口类型的实例，并返回自己/另一个抽象/接口类型的实现，其装饰/覆盖给定实例）

- java.util.Arrays#asList()
- java.util.Collections#list()
- java.util.Collections#enumeration()java.io.InputStreamReader(InputStream)（返回a Reader）
- java.io.OutputStreamWriter(OutputStream)（返回a Writer）
- javax.xml.bind.annotation.adapters.XmlAdapter#marshal() 和 #unmarshal()

**桥接模式** （可以通过创建方法识别采用不同抽象/接口类型的实例，并返回自己的使用给定实例的抽象/接口类型的实现）

一个虚构的例子将会new LinkedHashMap(LinkedHashSet< K>, List< V>)返回一个不可修改的链接映射，它不会克隆，而是使用它们。该java.util.Collections#newSetFromMap()和singletonXXX()方法却接近。

**组合模式** （通过将具有相同抽象/接口类型的实例的行为方法识别为树结构）

- java.awt.Container#add(Component) （几乎全部摆动）
- javax.faces.component.UIComponent#getChildren()

**装饰器模式** （通过创作方法识别采用相同抽象/接口类型的实例，添加额外的行为）

- 所有子类java.io.InputStream，OutputStream，Reader并Writer有一个构造函数取相同类型的实例。
- java.util.Collections的checkedXXX()，synchronizedXXX()和unmodifiableXXX()方法。
- javax.servlet.http.HttpServletRequestWrapper 和 HttpServletResponseWrapper

**门面模式** （可通过内部使用不同独立抽象/接口类型实例的行为方法识别）

- javax.faces.context.FacesContext，它在内部等使用抽象/接口类型LifeCycle，ViewHandler，NavigationHandler等等而没有终端用户具有担心它（它们然而通过注射覆写投放）。
- javax.faces.context.ExternalContext，其在内部使用ServletContext，HttpSession，HttpServletRequest，HttpServletResponse，等。

**享元模式** （使用缓存来加速大量小对象的访问时间）

- java.lang.Integer#valueOf(int)（还Boolean，Byte，Character，Short，Long和BigDecimal）

**代理模式** （可通过创建方法识别，该方法返回给定的抽象/接口类型的实现，该类型依次代表/使用给定抽象/接口类型的不同实现）

- java.lang.reflect.Proxy
- java.rmi.*
- javax.ejb.EJB
- javax.inject.Inject
- javax.persistence.PersistenceContext

#### **行为模式**

**责任链模式** （通过行为方法识别（间接地）在队列中的相同抽象/接口类型的另一个实现中调用相同的方法）

- java.util.logging.Logger#log()
- javax.servlet.Filter#doFilter()

**命令模式** （可以通过抽象/接口类型中的行为方法识别，该方法在创建时由命令实现封装的不同抽象/接口类型的实现中调用方法）

- 所有的实现 java.lang.Runnable
- 所有的实现 javax.swing.Action

**解释器模式** （通过行为方法识别，返回结构不同的实例/给定实例/类型的类型;请注意，解析/格式化不是模式的一部分，确定模式以及如何应用它）

- java.util.Pattern
- java.text.Normalizer
- 所有子类 java.text.Format
- 所有子类 javax.el.ELResolver

**迭代器模式** （可通过行为方法识别，从队列中顺序返回不同类型的实例）

- 所有的实现java.util.Iterator（因此还有java.util.Scanner！）。
- 所有的实现 java.util.Enumeration

**中介者模式** （通过采用不同的抽象/接口类型（通常使用命令模式）实例的行为方法来识别给定实例）

- java.util.Timer（所有scheduleXXX()方法）
- java.util.concurrent.Executor#execute()
- java.util.concurrent.ExecutorService（invokeXXX()和submit()方法）
- java.util.concurrent.ScheduledExecutorService（所有scheduleXXX()方法）
- java.lang.reflect.Method#invoke()

**备忘录模式** （可以通过内部改变整个实例的状态的行为方法来识别）

- java.util.Date（setter方法这样做，Date内部由一个long值表示）
- 所有的实现 java.io.Serializable
- 所有的实现 javax.faces.component.StateHolder

**观察者模式**（或发布/订阅） （可以通过行为方法识别，根据自己的状态调用另一个抽象/接口类型的实例上的方法）

- java.util.Observer/ java.util.Observable（很少在现实世界中使用）
- 所有实现java.util.EventListener（因此实际上各地的Swing）
- javax.servlet.http.HttpSessionBindingListener
- javax.servlet.http.HttpSessionAttributeListener
- javax.faces.event.PhaseListener

**状态模式** （可以通过行为方法识别，根据可以从外部控制的实例的状态改变其行为）

- javax.faces.lifecycle.LifeCycle#execute()（FacesServlet由此控制，行为取决于JSF生命周期的当前阶段（状态））

**策略** （可以通过抽象/接口类型中的行为方法识别，该方法在已经作为方法参数传递到策略实现中的不同抽象/接口类型的实现中调用方法）

- java.util.Comparator#compare()，由其他人执行Collections#sort()。
- javax.servlet.http.HttpServlet，service()所有的doXXX()方法HttpServletRequest
- HttpServletResponse实现者必须处理它们（而不是把它们保持为实例变量！）。
- javax.servlet.Filter#doFilter()

**模板方法** （可以由已经具有抽象类型定义的“默认”行为的行为方法识别）

- 所有非抽象方法java.io.InputStream，java.io.OutputStream，java.io.Reader和java.io.Writer。
- 所有非抽象方法java.util.AbstractList，java.util.AbstractSet和java.util.AbstractMap。
- javax.servlet.http.HttpServlet，doXXX()默认情况下，所有方法都会向响应发送HTTP 405“方法不允许”错误。你可以自由地执行任何一个或任何它们。

**访问者** （可以通过两种不同的抽象/接口类型识别，它们的方法定义为采用每个其他抽象/接口类型;实际上调用另一个抽象/接口类型的方法，另一个执行所需的策略）

- javax.lang.model.element.AnnotationValue 和 AnnotationValueVisitor
- javax.lang.model.element.Element 和 ElementVisitor
- javax.lang.model.type.TypeMirror 和 TypeVisitor
- java.nio.file.FileVisitor 和 SimpleFileVisitor

### **热门回复**

**1.Fuhrmanator**

适配器示例太浅了。我希望看到一个适配器接口如何支持不同的适配接口，但是从示例中并不明显。

**2.Tapas Bose**

@BalusC，我有一个问题要问你。你看了全部的 Java和JSF的源代码？

**3.dharam**

当然这是一个很有名的答案，对于那些需要了解什么是工厂方法和抽象工厂方法的人，我认为有很多不符合GoF模式的模式

1）工厂方法设计模式与静态工厂无关，而且这个答案中的大多数例子都是静态工厂。

2）抽象工厂模式也与静态工厂无关，这些只是编程习语。

**4.Nishant Shreshth**

在1.8中引入Calendar.Builder也是构建器模式的一个很好的例子。

> [原文链接](https://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries)