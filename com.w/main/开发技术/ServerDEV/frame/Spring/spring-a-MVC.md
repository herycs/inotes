# SpringMVC

## 认识

将业务代码从视图控制中独立出来

## 上下文在Web容器中的启动

### IOC建立基本过程

与ServletContext相伴相生

由ContextLoaderListener启动的上下文为根上下文

与Web相关的上下文DispatcherServlet，作为根上下文的子上下文

```java
ContextLoaderListener implement ServletContextListener//为Web容器中建立IOC容器服务的
```

### IOC 容器中的启动

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919201508246.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70#pic_center)

基于xml文件构建基础IOC的Bean，完成上下文初始化过程

## ContextLoader的设计与实现

```java
ContextLoaderListener通过ContextLoader完成实际的WebApplicationContext//负责启动根IOC并将其载入到Web容器中的主要模块
    //spring 加载IOC的第一个地方
    1.从Servlet中获取到ServletContext,读取到配置在web.xml中的属性值
    2.ContextLoader实例化WebApplicationContext，并完成其载入和初始化过程，这个被初始化的第一个上下文作为根上下文，载入后背绑定到Web应哟程序的ServletContext上
    任何需要访问根上下文的应用程序代码都可以从WebApplicationContextUtils类的静态方法中得到
```

## 总体请求设计

建立Controller与Http请求之间的关系

```java
hadlerMapping封装的HandlerExecutionChain//Http 请求与 Controller 的联系，IOC容器初始化时通过初始化HandlerMapping并将映射关系写入handlerMap中
```

### 处理请求

1. 接收到请求
2. 通过匹配请求URL从HandlerMapping中拿到HandlerExecutionChain，HandlerExecutionChain中封装了处理请求的Controller