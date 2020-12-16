# Spring MVC

## 过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730221041766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

1.用户发送请求给我们的前端控制器(sevletDispactherServlet), 这个控制器把请求拦截以后，在后端寻找执行类controller，来处理处理这个请求。返回给用户数据。

2.获取，派谁获取呢，就是派处理器映射器去获取，他的职责就是寻找handle，赶回执行结果，

3.他获取到以后，返回给前端控制器。返回给前端控制器一个执行链chain， 这个执行链有n个拦截器，拦截器里面放的handler，他包在拦截器里面，这个handle就是我们需要的。

4.前端控制器拿到后端的action，他不干活，他也执行不了controller。就派处理器适配器去执行

5.调用后端控制器controller的方法，

6.调用完了，后端控制器返回一个 modelandview。

7.适配器拿到ModelAndView后，在把ModelAndView返回给前端控制器。

8.前端控制器拿到ModelAndView后，他需要解析成真正的物理视图，他不干活啊，只负责调度，所以把ModelAndView交给视图解析器，

9.视图解析器把视图解析成真正的视图View后返回给前端控制器。

10.那对视图要请求进行视图渲染啊，所以他把视图解析器解析出来的视图转发给View进行视图渲染，

11.通过jstl表达式把数据放到jsp页面上，这个过程就叫视图渲染，渲染数据完了，他又把数据返回 给前端控制器

12.前端控制器拿到渲染后 的数据，把数据返回给用户。